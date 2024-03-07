![route53](https://github.com/oueya1479/aws-101/assets/147911523/7c1d675e-ad73-4d85-ab82-7ff24b0c1129)

<small>Author: heony</small>

### 목차

- [Amazon Route 53](#amazon-route-53)
  - [DNS Works](#dns-works)
  - [Route 53](#route-53)
    - [Hosted Zones](#hosted-zones)
  - [TTL](#ttl)
  - [CNAME vs Alias](#cname-vs-alias)
    - [Alias](#alias)
  - [Routing Policies](#routing-policies)
    - [Simple](#simple)
    - [Weighted](#weighted)
    - [Latency-based](#latency-based)
    - [Health Checks](#health-checks)
    - [Failover](#failover)
    - [Geolocation](#geolocation)
    - [Geoproximity Routing Policy](#geoproximity-routing-policy)
    - [IP-based](#ip-based)
    - [Multi-Value](#multi-value)

# Amazon Route 53

> DNS 트래픽이 사용하는 표준 포트 번호는 53 이라고 합니다.

![image](https://d1.awsstatic.com/Route53/product-page-diagram_Amazon-Route-53_HIW%402x.4c2af00405a0825f83fca113352b480b19d9210e.png)

Route 53은 IP를 도메인으로 만들어주는 DNS 웹 서비스 입니다. IP주소를 도메인으로 관리하거나 레코드 및 정책 간에 복잡한 라우팅 관계를 생성하고 시각화할 수 있습니다.

Route 53을 이해하기 위해서는 몇 가지 키워드를 보아야 합니다.

- Domain Registrar: Route 53, 가비아 등
- DNS Records: A, AAAA, CNAME, NS
- Zone File: DNS 레코드를 포함
- Name Server: resolves DNS 쿼리
- Top Level Domain: .com, .us, .in
- Second Level Domain: 차상위 도메인

```bash
Ex) http://api.www.example.com.

. : root
.com : TLD (Top Level Domain)
example.com : SLD (Second Level Domain)
www.example.com : Sub Domain
api.www.example.com : Domain Name
http : protocol
```

http://api.www.example.com. : FQDN (Fully Qualified Domain Name)

## DNS Works

![image](https://d1.awsstatic.com/Route53/how-route-53-routes-traffic.8d313c7da075c3c7303aaef32e89b5d0b7885e7c.png)

1. 사용자가 www.example.com을 컴퓨터에 입력합니다.
2. www.example.com은 DNS resolver로 이동합니다. resolver는 이름을 변환하는 기능을 제공하며 DNS 서버와 사용자(클라이언트)간 중개 역할을 합니다.
3. www.example.com을 DNS 서버에 요청합니다. / .com TLD를 위한 네임 서버로 이동시킵니다.
4. www.example.com을 .com TLD 서버에 요청합니다. / Route 53 네임 서버로 이동시킵니다.
5. www.example.com을 Route 53 서버에 요청합니다.
6. 192.0.2.44 라는 IP 주소를 반환합니다.
7. 해당 IP를 사용자에게 알려줍니다.
8. http://www.example.com을 요청합니다.
9. 해당 웹 사이트의 데이터가 올바르게 출력됩니다.

## Route 53

레코드를 통해 특정 도메인으로 라우팅하는 방법을 정의합니다. **Route 53에서 DNS 지원하는 레코드가 있습니다.**

- A - IPv4와 매핑
- AAAA - IPv6와 매핑
- CNAME - 호스트 이름을 다른 호스트 이름과 매핑
- NS - 호스팅 존의 네임서버. DNS 쿼리에 응답할 수 있음. 트래픽이 도메인으로 라우팅되는 정책을 설정할 수 있음

### Hosted Zones

- Public Host Zone: 인터넷에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 포함되어 있습니다.
프라이빗 호스트 존: 가상 프라이빗 클라우드만이 URL을 리졸브할 수 있음

- Private Host Zone: 가상 프라이빗 클라우드만이 URL을 리졸브할 수 있으며, VPC에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 포함되어 있습니다.

> 월에 50센트, 일년에 대략 12달러

## TTL

TTL(Time To Live)은 클라이언트가 Amazon Route 53에 DNS Request를 요청함으로써 반환되는 ip를 Cache로 저장하는 기술을 의미합니다. 기존에는 example.com이라는 요청을 응답하기 위해서는 Route 53에게 계속 요청을 시도해야 합니다. 그러나 반환되는 ip값을 저장해두면 가지고 있는 정보만으로도 목적지를 찾을 수 있습니다.

- High TTL: 24시간
- Low TTL: 60초

## CNAME vs Alias

도메인을 다른 호스트 네임으로 지정할 수 있는 방법이 있습니다. 바로 CNAME과 Alias인데요, `CNAME`은 다른 호스트 네임으로 변경할 수 있지만, Non root domain만 가능합니다. 따라서 anything.com과 같은 루트 도메인은 불가능합니다.

`Alias`의 경우 AWS Resource로 제공되기 때문에 Root domain까지 지원해줍니다.

- CNAME : app.mydomain.com -> blabla.anything.com 으로 호스트 네임을 지정 (다른 호스트 네임을 지정할 수 있음) / **NON ROOT DOMAIN 만 가능**

- Alias : app.mydomain.com -> blabla.amazonaws.com 으로 호스트 네임을 지정 (다른 AWS Resource를 지정할 수 있음) / **ROOT DOMAIN, NON ROOT DOMAIN 다 가능**

### Alias

- AWS Resource에 매핑
- DNS 확장 기능 제공
- CNAME과 달리 DNS namespace의 상위 노드로 사용 가능
- 별칭 레코드 타입은 A / AAAA 가능
- TTL 설정 불가 (Route 53이 기본으로 정해줌)

ELB, CloudFront, API Gateway, Beanstalk, S3 Website, VPC Interface Endpoints, Route 53 Record 등 많은 서비스에서 사용 가능합니다.

**하지만 EC2 DNS 이름을 위한 Alias는 불가능**합니다.

> elb 주소가 elb-example.aws 이고, 본인이 구매한 도메인이 example.com이라고 가정해봅시다. example.com을 입력하면 elb-example.aws로 이동시키는 기능은 어떻게 구현할 수 있을까요?

> CNAME은 Root domain의 직접적 연결을 금하기 때문에 ().example.com 과 같이 괄호 안에 어떤 내용이 필수적으로 들어가야만 합니다. 따라서 이 상황을 해결하기 위해서는 Alias로 지정 -> A 레코드 타입 선택 -> elb-example.aws 선택하면 됩니다.

## Routing Policies

DNS 관점의 Routing으로 생각해야합니다.

- Simple
- Weighted
- Failover
- Latency-based
- Geolocation
- Multi-Value Answer
- Geoproximity

### Simple

일반적으로 트래픽을 단일 리소스로 보내는 방식입니다.

1. Single Value -> request : example.com / response : 11.22.33.44
2. Multiple Value -> request : example.com / response 11.22.33.44, 55.66.77.88, 99.11.22.33

### Weighted

특정 리소스를 가중치를 할당받아 요청을 보낼 수 있습니다.

Weight가 70인 EC2에게는 전체 트래픽(100) 중 70% 가량 담당하며, 가중치 0은 트래픽을 받지 않습니다. 만약, 모든 리소스의 가중치가 0일 경우 모두 동등해집니다.

### Latency-based

지연 시간 기반으로 측정되어 트래픽이 이동됩니다.

페이지를 불러올 때에 가장 시간이 적게 소요되는 서버로부터 트래픽을 전달받습니다. 실제로 만들기 위해서는 IP만으로는 어느 지역인지 특정지을 수 없기 때문에 Region을 꼭 선택해야합니다.

### Health Checks

1. Monitor and Endpoint
- 15개의 글로벌 환경에서 health checkers들이 request를 보내고, 2XX, 3XX code가 반환되면 허용됩니다.
- 텍스트 기반 응답일 경우 처음 5,120 바이트를 확인하여 해당 텍스트가 존재하는지 확인합니다.

2. Calculated Health Checks
여러 개의 상태 확인 결과를 하나로 합쳐주는 기능입니다.

- AND, OR, NOT만 가능
- 하위 상태 확인을 256개까지 모니터링할 수 있다.
- 상태 확인인이 실패하는 일 없이 웹사이트를 관리하고 유지해야 할 때 사용

3. Private Hosted Zones
Route 53 health checkers는 VPC 외부에 있습니다. private endpoints에 액세스할 수 없으며, CloudWatch Metric과 Alarm을 통하여 Health Check를 진행합니다.

### Failover

Health check에도 연결이 되어있어야 하며, primary / secondary 둘을 지정하여 primary에서 장애가 발생했을 때 secondary로 트래픽을 보냅니다.

### Geolocation

사용자의 실제 위치를 기반으로 가장 정확한 위치가 선택되어 그 ip로 트래픽이 전송됩니다.

> Location은 Africa, Asia, Europe, Oceania, United States 등으로 지정이 가능합니다.
> Default는 기본 위치로 이동합니다(EC2 서버가 존재하는 곳)

### Geoproximity Routing Policy

지리적 위치 편향값이라고 불리며, 특정 리소스에 더 많은 트래픽을 보내려면 편향값을 증가시키고, 줄이고 싶으면 편향 값을 줄이면 됩니다.

> 미국 앙 끝단에(동·서 지방) 서버가 있을 때에는 미국을 반으로 나누어 왼쪽은 west 서버, 오른쪽은 east 서버에 접근되겠지만, Geoproximity 정책을 통해 west 서버를 편향값 0, east 서버를 편향값 50이라고 지정하면, 더 많은 사람들이 east 서버를 이용하게 될 것입니다.

### IP-based

ip 기반 라우팅, 클라이언트 ip 주소를 기반으로 정책을 지정합니다. CIDR을 통해 나눌 수 있습니다.

| location | CIDR blocks |
|------------|----------------|
| location-1 | 203.0.113.0/24 |
| location-2 |  200.5.4.0/24  |

| Record Name | Value | IP-based |
|------------|----------|------------|
| example.com | 1.2.3.4 | location-1 |
| example.com | 5.6.7.8 | location-2 |

이 때, ip `203.0.113.56` 는 `location-1`에 ip `200.5.4.100` 는 `location-2`에 접근하게 될 것입니다.

### Multi-Value

트래픽을 다중 리소스로 라우팅할 때 사용합니다.

Simple 정책은 Health check가 안 되기 때문에 Multi-Value를 사용해야 하며, 상태 확인과 연결되면 다중 값 정책에서 반환되는 유일한 리소스는 정상 상태 확인과 관련이 있습니다.

> ELB와 유사하지만 그것을 대체할 수 없습니다.

| Name | Type | Value | TTL | Set ID | Health Check |
|-----------------|----------|-----------|----|------|---|
| www.example.com | A Record | 192.0.2.2 | 60 | Web1 | A |
| www.example.com | A Record | 198.51.100.2 | 60 | Web2 | B |
| www.example.com | A Record | 203.0.113.2 | 60 | Web3 | C |