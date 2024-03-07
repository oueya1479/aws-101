![elb](https://github.com/oueya1479/aws-101/assets/147911523/2994d2e1-14fc-4fa3-9495-20d04ce104fd)

<small>Author: heony</small>

### 목차

- [Elastic Load Balancing](#elastic-load-balancing)
    - [네트워크 트래픽을 분산하여 애플리케이션 확장성 개선](#네트워크-트래픽을-분산하여-애플리케이션-확장성-개선)
  - [Health Checks](#health-checks)
  - [Security Group](#security-group)
- [Application Load Balancer (ALB)](#application-load-balancer-alb)
  - [대상 그룹](#대상-그룹)
- [Network Load Balancer (NLB)](#network-load-balancer-nlb)
  - [대상 그룹](#대상-그룹-1)
- [Gateway Load Balancer (GWLB)](#gateway-load-balancer-gwlb)
  - [대상 그룹](#대상-그룹-2)
- [Sticky Sessions (Session Affinity)](#sticky-sessions-session-affinity)
  - [쿠키는 무엇인가요?](#쿠키는-무엇인가요)
- [Cross-Zone Load Balancing](#cross-zone-load-balancing)
- [SSL/TLS](#ssltls)
  - [SNI (Server Name Indication)](#sni-server-name-indication)

# Elastic Load Balancing

### 네트워크 트래픽을 분산하여 애플리케이션 확장성 개선

Elastic Load Balancing(이하 ELB)는 서버 혹은 서버셋으로 트래픽을 백엔드나 다운스트림으로 전달하는 역할을 합니다. 예를 들어 EC2가 3개 있다고 할 때 ELB는 이를 계산하여 부하를 지속적으로 연결된 3개의 인스턴스로 분산시킵니다.

AWS가 관리하며 유지 관리 및 고가용성을 제공하기 때문에 `managed load balancer` 라고도 불립니다. 총 3가지의 load balancer를 제공합니다.

<del>Classic Load Balancer: 이제는 AWS에서 공식적으로 지원하지 않습니다.</del>
1. Application Load Balancer
2. Network Load Balancer
3. Gateway Load Balancer

일부 로드 밸런서들은 네트워크에 개인적 접근이 가능하고 액세스가 간편하기 때문에 적절한 사용이 필요합니다.

## Health Checks

ELB는 Health checks를 필수로 가지고 있습니다. 예를 들어 HTTP protocol에 4567포트를 사용하는 EC2가 있다고 가정해봅시다. Endpoint를 `/health`라고 지정했을 때 ELB는 지속적으로 요청을 보내 답변을 확인합니다. 만약 `200`의 상태를 가지고 있지 않다면 이는 `unhealthy` 상태로 판단합니다.

## Security Group

보안 그룹은 EC2에서 학습했습니다. VPC단에서 트래픽을 담당하는 NACL이 아닌, 인스턴스단에서 트래픽을 관리하는 `보안 그룹`이 있었습니다. 이는 포트에 대한 범위를 직접적으로 제공하여 인스턴스 인바운드 및 아웃바운드에 대한 규칙을 세울 수 있었습니다.

ELB에서도 Security Group을 지정해야합니다. 특히 권장하는 설정이 있습니다.

**User ----- ELB ----- EC2**

위와 같이 로드벨런서 타겟의 모든 EC2는 ELB에 의존하기 때문에

- ELB : 443, 80 port 제공
- EC2 : 80 port 제공 (ELB로부터 들어오는 포트 오픈)

이와 같은 사용이 권장됩니다. ELB는 `80(HTTP)`, `443(HTTPS)`포트를 개방하여 EC2로 접근할 수 있는 두 가지 방법을 제시해야만 합니다.

> 추가적으로 보안이 요청되는 것이 있습니다. 바로, ELB ----- EC2간 보안 그룹을 또 지정할 수 있는데요, EC2는 ELB에 의해 요청을 받으므로, EC2의 Security Group의 80번 포트는 0.0.0.0이 아닌 ELB에서 수신되는 값으로만 설정해야 합니다.

---

# Application Load Balancer (ALB)

7계층의 로드벨런서입니다. HTTP와 HTTPS로부터 수신되는 트래픽을 라우팅할 수 있습니다.

```bash
URL 기반 : example/users & example.com/posts
hostname 기반 : one.example.com & other.example.com
query string, header 기반 : example.com/users?id=123&order=false
```

포트 매핑 기능이 있어 Docker를 사용할 수 있는 ECS 서비스에 적절합니다.

## 대상 그룹

대상 그룹이란, ELB로 관리받을 수 있는 어떤 집단을 의미합니다. 예를 들어 10개의 EC2가 있지만 그 중에 5개만 ELB로 관리하고 싶다고 하면 대상 그룹은 해당 EC2 다섯 개가 되는 것입니다.

오토 스케일링 그룹으로 관리될 수 있는 EC2 인스턴스나 ECS tasks, Lambda function, IP Addresses 등이 될 수 있습니다.

---

# Network Load Balancer (NLB)

4계층의 로드밸런서입니다. HTTP를 다루는 ALB보다 더 하위계층이며 TCP와 UDP 트래픽을 처리합니다. 성능은 ALB보다도 매우 높습니다. 초당 수백만건을 처리할 수 있으며 지연 시간도 짧습니다. 무려 4배 정도의 차이입니다.

**가용 영역별로 하나의 고정 IP를 갖습니다. 탄력적 IP 주소를 각 AZ에 할당할 수도 있습니다.** 여러 개의 고정 IP를 가진 애플리케이션을 선택해야할 때 좋은 선택입니다.

> Free tier에 포함되지 않으니 유료 사용을 조심해주세요.

> 만약 EC2의 상태가 unhealthy로 출력된다면 EC2의 security group을 확인하세요. NLB의 경우는 보안 그룹이 따로 존재하지 않습니다. 따라서 EC2의 보안 그룹 영향을 받기 때문에 확인이 필요합니다.

## 대상 그룹

NLB의 대상 그룹으로는 EC2 instances, IP Addresses(private IPs), ALB 등이 있습니다. ALB의 경우 고정 IP를 가질 수 있는 NLB의 특성을 이용할 수도 있고, HTTP의 처리 또한 가능해지기 때문에 대상 그룹을 ALB으로도 지정할 수 있습니다.

`Health Checks`로는 TCP, HTTP, HTTPS 프로토콜을 지원합니다.

---

# Gateway Load Balancer (GWLB)

가장 최근에 제작된 로드밸런서입니다. 배포 및 확장에 이용되며 모든 트래픽이 방화벽을 통과하게 하거나 침입 방지 시스템에도 이용됩니다. 다른 로드밸런서와 비슷하지만, 모든 네트워크 트래픽을 분석하는 시스템을 갖추고 있습니다.

예를 들면 Application으로 도달하기 전 GWLB를 통해 트래픽을 받고, 이를 `Target Group`인 써드파티 보안 가상 Application을 통하여 검증을 실시합니다. 이렇게 반환된 트래픽을 다시 GWLB가 건네받아 최종 목적지인 Application에 전달하는 형태입니다.

> GWLB는 모든 로드밸런서보다 낮은 3계층입니다.

> 6081번 포트의 GENEVE 프로토콜을 사용하면 바로 GWLB가 됩니다.

## 대상 그룹

EC2 instances, IP Addresses(private IPs)를 지원합니다.

---

# Sticky Sessions (Session Affinity)

sticky는 접착과 관련된 단어입니다. 즉 ELB를 사용할 때 우리는 하나의 EC2에 접근하게 됩니다. 이 서버 안에서 여러 세션 또는 데이터가 저장될 수 있습니다. 그러나 ELB는 부하분산을 지원하기 때문에 새로 고침을 하거나, 서버에 다시 재요청을 하게 될 때 **다른 서버의 리소스로 보내질 수 있습니다.** 즉 기존에 있던 데이터가 ELB 때문에 다 날라갈 수도 있다는 것입니다.

이러한 것을 방지하기 위해 Sticky Sessions의 기능이 개발되었습니다. 이는 쿠키의 활용으로 가능하게 되었는데 사용자의 로그인과 같이 세션 데이터를 잃지 않기 위해 동일한 백엔드 서버로 이동하게 됩니다.

## 쿠키는 무엇인가요?

Application-based Cookies는 애플리케이션에서 생성되는 것으로 모든 사용자의 속성을 포함할 수 있습니다. 각 대상 그룹별고 개별적으로 저장됩니다. 애플리케이션에서 기간을 정할 수 있습니다.

Duration-based Cookies는 로드밸런서에서 생성됩니다.

---

# Cross-Zone Load Balancing

편하게 `Cross-Zone`을 교차 영역이라고 부르겠습니다. 교차 영역 로드밸런서는 트래픽의 부하분산을 어떻게 시킬것인지 방법론에 대한 이야기입니다. 실제 기능으로도 작동하고요.

![image](https://docs.aws.amazon.com/images/elasticloadbalancing/latest/userguide/images/cross_zone_load_balancing_enabled.png)

- With Cross Zone Load Balancing : 각 로드밸런서가 각기 다른 가용영역에 있는 EC2를 탐지하여 모두 고르게 분산시킵니다.

> 예를 들면, AZ1에 인스턴스 두 개, AZ2에 인스턴스 여덟 개가 있다고 하고, 각각 AZ에 하나의 ELB가 있다고 가정해봅시다. **트래픽은 각 ELB에 50씩 전달** 되며 모든 인스턴스는 가중치 10의 트래픽을 전달받습니다. 즉, 각 AZ에 ELB가 있다면 교차 영역에 있는 모든 인스턴스는 가중치를 똑같이 받는다는 의미입니다.

> 교차 영역 로드밸런싱의 경우 `ALB`에서 기본적으로 켜져있지만, `NLB`는 기본적으로 꺼져있습니다. 만약 `NLB`에서도 사용하고 싶을 경우 추가적으로 켜야합니다.

![image](https://docs.aws.amazon.com/images/elasticloadbalancing/latest/userguide/images/cross_zone_load_balancing_disabled.png)

- Without Cross Zone Load Balancing : 각 로드밸런서는 똑같은 가중치의 수치를 받지만, 같은 가용 영역에 있는 인스턴스들은 그 ELB의 가중치를 나눕니다.

> 위의 상황과 같다고 하면 각 ELB는 50씩의 가중치를 받고, AZ1에는 각각 25씩(25 * 2 = 50), AZ2에는 각각 6.25씩(6.25 * 8 = 50) 가중치를 받게 됩니다.

물론 정답은 없습니다. 어느 서비스를 어떤 상황에서 써야하는지 구분만 잘 하면 됩니다.

---

# SSL/TLS

**SSL 인증서는 클라이언트와 서버 사이의 데이터 이동 사이에 암호화하는 기술**을 의미합니다. 여기서는 로드 밸런서를 다루고 있으니 클라이언트와 로드 밸런서 사이의 암호화 기술을 의미하겠네요. SSL 인증서에는 만료 날짜가 있어서 주기적으로 인증 상태를 유지해야 합니다.

> SSL은 Secure Sockets Layer으로 보안 소켓 계층을 의미합니다.

> TLS는 Transport Layer Security으로 SSL보다 향상된 더욱 안전한 버전입니다. *TLS 보안 인증서 또한 SSL로 언급한다고 합니다 (구분해서 사용하면 좋지만 통상적으로).*

```
Users ---------- ELB ---------- EC2 Instance
       HTTPS통신        HTTP통산
```

- 위와 같은 상황에서 Load Balancer는 X.509 인증서를 사용합니다.
- ACM(AWS Certification Manager)을 사용하여 인증서를 관리할 수 있습니다.
- 직접 가지고 있는 인증서를 업로드할 수도 있습니다.
- HTTP Listener
  - 기본 인증서를 정해야 합니다.
  - 다중 도메인을 지원하기 위해 다른 인증서를 추가할 수도 있습니다.
  - `SNI(Server Name Indication)`를 사용하여 접속할 호스트의 이름을 알릴 수 있습니다.

## SNI (Server Name Indication)

SNI는 다중 SSL 인증서를 하나의 웹 서버가 지원할 수 있도록 처리하는 기술입니다. 클라이언트가 호스트네임을 지정하게 되면 SNI는 해당 호스트네임에 해당하는 서버를 지원해줍니다.

**단일 로드 밸런서 뒤에서 각각 자체 인증서를 가지고 있는 다수의 보안 애플리케이션을 실행할 수 있습니다.**

> [AWS 블로그](https://aws.amazon.com/ko/blogs/korea/application-load-balancers-now-support-multiple-tls-certificates-with-smart-selection-using-sni/)를 참조하세요.

