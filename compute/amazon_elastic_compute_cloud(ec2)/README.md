![ec2](https://github.com/oueya1479/aws-101/assets/147911523/1a15c2a2-d5b2-4486-ab00-8f0a9461be2e)

<small>Update: 2024. 02. 09</small>

<small>Author: heony</small>

### 목차

- [Amazon Elastic Compute Cloud (EC2)](#amazon-elastic-compute-cloud-ec2)
    - [거의 모든 워크로드에 적합한 안전하고 크기 조정 가능한 컴퓨팅 용량](#거의-모든-워크로드에-적합한-안전하고-크기-조정-가능한-컴퓨팅-용량)
- [EC2 접근하기](#ec2-접근하기)
  - [💡Tip](#tip)
- [EC2 유형](#ec2-유형)
  - [인스턴스 패밀리](#인스턴스-패밀리)
    - [범용](#범용)
    - [컴퓨팅 최적화](#컴퓨팅-최적화)
    - [메모리 최적화](#메모리-최적화)
    - [가속화된 컴퓨팅](#가속화된-컴퓨팅)
    - [스토리지 최적화](#스토리지-최적화)
    - [세대 및 추가기능](#세대-및-추가기능)
    - [인스턴스 크기](#인스턴스-크기)
    - [종합](#종합)
- [인스턴스 구입 옵션](#인스턴스-구입-옵션)
  - [1. On Demand (온디맨드 인스턴스)](#1-on-demand-온디맨드-인스턴스)
  - [2. Reserved Instance (예약 인스턴스)](#2-reserved-instance-예약-인스턴스)
  - [3. Saving Plans (절감형 플랜)](#3-saving-plans-절감형-플랜)
  - [4. Spot Instance (스팟 인스턴스)](#4-spot-instance-스팟-인스턴스)
  - [5. Dedicated Host (전용 호스트)](#5-dedicated-host-전용-호스트)
  - [6. Dedicated Instance (전용 인스턴스)](#6-dedicated-instance-전용-인스턴스)
- [Private vs Public vs Elastic IP](#private-vs-public-vs-elastic-ip)
- [Placement Groups](#placement-groups)
  - [Cluster](#cluster)
  - [Partition](#partition)
  - [Spread](#spread)
- [Elastic Network Interfaces (ENI)](#elastic-network-interfaces-eni)
- [EC2 Hibernate](#ec2-hibernate)

# Amazon Elastic Compute Cloud (EC2)

### 거의 모든 워크로드에 적합한 안전하고 크기 조정 가능한 컴퓨팅 용량

Amazon Elastic `C`ompute `C`loud 즉, EC2라고 불리는 서비스는 AWS의 대표적인 클라우드 서비스입니다.

---

# EC2 접근하기

**가장 먼저 개인 키를 생성한 폴더로 접근해서 키에대한 접근 권한을 변경해주어야 합니다.** 그냥 접근하게 되면 권한에 대해 부인되면서 접속이 되지 않습니다. 저는 /Users/heony/key 라는 곳에 key를 저장해두었고, 다음은 key에 명령을 내리는 방법입니다.

```bash
heony@gimdongheon-ui-MacBookPro ~ % cd /Users/heony/key
heony@gimdongheon-ui-MacBookPro key % chmod 700 i-am-key.pem
heony@gimdongheon-ui-MacBookPro key %
```

cd로 해당 디렉토리에 접근해서 권한을 부여해도 되고, 다음과 같이 부여해도 됩니다.

```bash
heony@gimdongheon-ui-MacBookPro ~ % chmod 700 /Users/heony/key/i-am-key.pem
heony@gimdongheon-ui-MacBookPro ~ %
```

## 💡Tip

> 자신에게 맞는 권한 추가하기. 이 외에도 여러가지 방식이 존재합니다.
- chmod 000 test.pem : 사용자, 그룹, 다른사용자의 모든 권한을 제거한다.
- chmod 777 test.pem : 사용자, 그룹, 다른사용자의 모든 권한을 추가한다.
- chmod 700 test.pem : 사용자에게만 모든 권한을 준다.
- chmod 744 test.pem : 사용자에게는 모든 권한을 주고, 그룹, 다른 사용자에게는 읽기 권한만 준다.

key에 대해 권한이 부여되었다면 인스턴스의 퍼블릭 IPv4주소와 함께 키 페어를 입력하면 됩니다. yes를 입력하는 것도 잊지 마세요.

```bash
// 형식은 다음과 같다.
ssh -i [key의 위치와 key파일 이름] [사용자이름]@[퍼블릭IPv4주소]
```

```bash
// 여기서 사용자이름은 OS마다 다르다.
// ubuntu를 OS로 선택했기 때문에 사용자 이름을 ubuntu로 입력하면 된다.
// 다음과 같이 입력하면 된다.
ssh -i /Users/heony/key/i-am-key.pem ubuntu@
ec2-3-34-125-16.ap-northeast-2.compute.amazonaws.com
```

```bash
heony@gimdongheon-ui-MacBookPro ~ % ssh -i /Users/heony/key/i-am-key.pem ubuntu@
ec2-3-34-125-16.ap-northeast-2.compute.amazonaws.com
The authenticity of host 'ec2-3-34-125-16.ap-northeast-2.compute.amazonaws.com (3.34.125.16)' cant be established.
ED25519 key fingerprint is SHA256:ckZpe/xCWPOsuF3SJSpfDsxDuPr/yuS0NZT604NHINA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

우리는 접속하는 방법이 ssh -i [key] [사용자이름]@[퍼블릭IPv4]라는 것을 알았습니다.

여기서 사용자 이름은 OS가 ubuntu이었기 때문에 ubuntu로 지정했지만 만약 다른 OS를 선택하셨다면 다음을 참고해주세요.

- Amazon Linux 2 또는 Amazon Linux AMI의 경우 사용자 이름은 ec2-user입니다.
- CentOS AMI의 경우 사용자 이름은 centos 또는 ec2-user입니다.
- Debian AMI의 경우 사용자 이름은 admin입니다.
- Fedora AMI의 경우 사용자 이름은 fedora 또는 ec2-user입니다.
- RHEL AMI의 경우 사용자 이름은 ec2-user 또는 root입니다.
- SUSE AMI의 경우 사용자 이름은 ec2-user 또는 root입니다.
- Ubuntu AMI의 경우 사용자 이름은 ubuntu입니다.
- Oracle AMI의 경우 사용자 이름은 ec2-user입니다.
- Bitnami AMI의 경우 사용자 이름은 bitnami입니다.
- 그렇지 않은 경우 AMI 공급자에게 문의하세요.

---

# EC2 유형

EC2를 생성하기 위해서는 OS이미지 및 인스턴스 유형에 대해서 선택해야 합니다. 알다시피 `t2.micro`는 AWS에서 지원하는 프리 티어 인스턴스 유형입니다.

첫 번째로 알아볼 것은 인스턴스 패밀리입니다. `t2`의 의미는 무엇인지, 또한 `micro`라는 이름이 붙여진 이유에 대해서 알아보도록 하겠습니다.

두 번째로 알아볼 것은 인스턴스 구입 옵션입니다. 요구 사항에 따라 비용을 최적화 하는 방법을 소개합니다.

t2.micro는 다음과 같은 이름 규칙으로 지어져있습니다.

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xADeY8sF1XIYEOKgLgb-yg.png)

## 인스턴스 패밀리

### 범용

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*psWLHZAjChtRCLOs.png)

**어느 한쪽으로도 치우쳐 있지 않은 AWS EC2 인스턴스 유형입니다.** AWS에 따르면 균형 있는 컴퓨팅, 메모리 및 네트워크 리소스를 제공한다고 합니다. 머신 러닝이나 딥러닝, 또한 특수한 상황에 놓여있지 않는 모든 컴퓨팅 서비스를 범용이라고 칭합니다.

웹 서버 및 코드 리포지토리와 같이 이러한 리소스를 동등한 비율로 사용하는 애플리케이션에 적합합니다.

범용에는 총 4가지 유형이 존재합니다.

- Mac : macOS
- M : 범용 (vCPU 1개, 4GB 메모리)
- T : 버스트가 가능한 CPU
- A : ARM 기반

### 컴퓨팅 최적화

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*xxB6TTLOiYPaHIDA.png)

**컴퓨팅 최적화는 고성능의 프로세서를 활용하는 컴퓨팅 집약적 애플리케이션에 적합합니다.** 배치 처리 워크로드, 미디어 트랜스코딩, 고성능 웹 서버 및 고성능 컴퓨팅, 과학적 모델링, 전용 게임 서버, 광고 서버 엔진 등의 다양한 컴퓨팅 집약 애플리케이션에 적합합니다.

컴퓨팅 최적화군은 다행히 유형이 C 하나입니다.

Computing의 약자인 C가 유형이 되었습니다.
- C : 컴퓨팅 최적화

### 메모리 최적화

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*G93osm9hZpSbczUy.png)

**메모리 최적화 인스턴스는 메모리에서 대규모 데이터 세트를 처리하는 워크로드를 위한 빠른 성능을 제공하기 위해 설계되었습니다.** 많은 프로그램을 돌리거나 대규모 데이터를 처리하는데 유리합니다.

메모리 최적화 인스턴스는 3가지 유형이 존재합니다.

> 아쉽게도 메모리 최적화의 유형은 M이 아니네요

RAM을 뜻하는 단어가 유형이 되었습니다 (R)
- R : 초대형 메모리
- X : 랜덤 액세스 메모리(RAM)
- z : 고주파수 메모리

### 가속화된 컴퓨팅

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*4eP0VKvQ7Pz7fpoW.png)

가속화된 컴퓨팅 인스턴스는 **하드웨어 액셀러레이터 또는 코프로세서를 사용**하여 부동 소수점 수 계산이나 그래픽 처리, 데이터 패턴 일치 등의 기능을 CPU에서 실행되는 소프트웨어보다 훨씬 효율적으로 수행합니다.

즉, CPU의 기능을 보충하기 위해 사용되는 컴퓨터 프로세서를 보완하며 고성능 컴퓨팅을 제공합니다.

가속화된 컴퓨팅에서는 3개의 유형이 존재합니다.

- P : 프리미엄 GPU
- G : GPU
- F : FPGA(Field programmable gate array: 프로그래머블 논리 요소와 프로그래밍 가능 반도체)

### 스토리지 최적화

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*QRWEPWoSDSvu5yCx.png)

스토리지 최적화 인스턴스는 로컬 스토리지에서 **매우 큰 데이터 세트에 대해 많은 순차적 읽기 및 쓰기 액세스를 요구**하는 워크로드들 위해 설계되었습니다.

이러한 인스턴스는 애플리케이션에 대해 대기 시간이 짧은, 수만 단위의 무작위 IOPS를 지원하도록 최적화되었습니다.

대용량 데이터 집합에 대해 높은 I/O 성능을 제공하고, 디스크 처리량이나 저렴한 스토리지가 필요한 애플리케이션에 대해 최적화되어 있습니다.

스토리지 최적화 인스턴스는 3가지 유형이 존재합니다.

- D : (Dense Storage) 고밀도 스토리지
- H : HDD
- I : NVMe(Non-Volatile Memory Express)

### 세대 및 추가기능

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*3lordpaNkkwPK-xU.png)

t2.micro에서는 살펴볼 수 없었지만 다음과 같은 이미지를 보면 추가기능이 붙은 것을 확인할 수 있습니다.

위의 사진을 분석하면 x이므로 메모리 최적화 인스턴스이며(그 안의 내용은 당연히 중요하지만, 꼭 외울 필요는 없습니다. 그저 x가 메모리 최적화와 관련된 인스턴스 유형임을 기억하면 좋습니다.) 2이므로 2세대라는 것을 알 수 있습니다.

그러나 iedn이라는 추가 기능이 붙어있는데 이는 아직 살펴보지 않아서 잘 모르는 것 같습니다. 우리는 추가기능으로 어떤 것이 존재하는지에 대해 살펴보도록 하겠습니다.

- a : AMD 프로세서
- b : 블록 스토리지 최적화
- d : 인스턴스 스토어 볼륨(휘발성)
- e : 추가 스토리지 또는 메모리
- g : AWS Graviton 프로세서
- i : 인텔 프로세서
- n : 네트워크 최적화
- z : 고주파

즉 위의 사진은 (메모리 최적화 2세대 인스턴스) + 인텔 프로세서(i) + 추가 스토리지(e) + 인스턴스 스토어 볼륨(d) + 네트워크 최적화(n) 기능이 포함된 것을 알 수 있습니다.

### 인스턴스 크기

![image](https://miro.medium.com/v2/resize:fit:710/format:webp/0*FOdcs-GN1TU77Z-L.png)

다음은 t4g에 속해있는 인스턴스 크기를 가져와보았습니다.

`large`부터는 일정한 특징을 가지고 있습니다. 바로 vCPU의 개수가 2씩 곱해지며, 메모리의 크기도 2씩 곱해진다는 특징입니다. nano부터 2xlarge에 이르기까지 전체적인 특징을 한번 이해해보는 것이 좋지만, 굳이 외우시지 않으셔도 됩니다.

그러나 모든 규칙이 위의 사진과 같이 성립되는 것은 아닙니다. medium이 1vCPU와 2GiB 메모리의 성능을 가질 수도 있습니다. 즉, 각 인스턴스 유형마다 특징이 다르기 때문에 어느 하나로 답을 내릴 수 있는 것은 아닙니다.

그러나 xlarge는 large의 두배의 성능, nxlarge(여기서 n은 자연수)는 xlarge 의 n배의 성능을 가지고 있다라는 것만 이해하시면 도움이 될 것 같습니다.

그냥 자신의 컴퓨터가 어느 정도 사양에 속하는지에 대해 한 번 생각해보고, 이 정도의 인스턴스 크기면 어느 정도의 성능이겠다 정도로만 이해하시면 될 것 같습니다. + 가장 좋은 방법은 해당 유형의 크기를 참조하는 것이 제일 좋습니다.

### 종합

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*mPF8MSf1v1_ABSvD.png)

t2.micro는 어떠한 추가기능이 없는 범용 인스턴스이며 2세대입니다. 또한 micro이며 t2에서의 micro는 1개의 vCPU와 1GiB의 메모리 성능을 가지고 있습니다.

---

# 인스턴스 구입 옵션

흔히 체크카드를 신청하기 전에 우리는 어떤 은행을 선택할 것인가, 어떤 체크카드를 선택할 것인가에 대해 고민하게 됩니다. 왜 고민하시나요? 체크카드를 교통카드로 많이 사용하는 사람이 있고, 또는 편의점 및 백화점에 많이 사용하는 사람이 있듯이 각자의 소비패턴에 맞는 것을 골라야 하기 때문입니다.

인스턴스 또한 마찬가지입니다. **사용자의 요구 사항에 맞게 비용을 최적화할 수 있는 구입 옵션을 선택해야합니다.**

종류는 다음과 같습니다.

- 온디맨드 인스턴스 — 초 단위로 지불
- 예약 인스턴스 — 1년 또는 3년 기간 동안 인스턴스 구성을 약정하여 EC2 비용을 절감
- 절감형 플랜— 1년 또는 3년 기간 동안 시간당 USD로 사용량을 약정하여 EC2 비용을 절감
- 스팟 인스턴스 — 미사용 EC2 인스턴스를 요청하여 EC2 비용을 대폭 절감
- 전용 호스트 — 인스턴스 실행을 전담하는 실제 호스트 비용을 지불하여 비용을 절감
- 전용 인스턴스 — 인스턴스 비용을 시간 단위로 지불
  
## 1. On Demand (온디맨드 인스턴스)

사용한 만큼 지불합니다. 리눅스나 윈도우를 사용하게 되면 1분이 지나가는 시점부터 초단위로 지불되기 시작합니다. 그 외의 다른 운영체제에서는 시간당 비용이 청구됩니다.

가장 중요한 부분은 사용한 만큼만 처리된다는 점입니다. 그렇기 때문에 미리 예약을 하여 인스턴스를 제공받는 것 보다 비쌀 수 밖에 없습니다. (실제로 비싼건 아니지만 다른 옵션들에 비해 상대적으로)

따라서 보통 애플리케이션의 동작을 예측할 수 없을 때 사용합니다.

## 2. Reserved Instance (예약 인스턴스)

1년 또는 3년동안 미리 예약을 할 수 있습니다. 이 때 예약하는 것은 인스턴스 타입, 리전(Region), 테넌시(Tenancy), 운영체제(OS) 입니다.

EC2에서 예약하는 모든 것들은 1년 혹은 3년입니다.

당연히 3년을 예약할 경우 할인 폭이 가장 넓어지며, 1년을 예약하면 할인 폭이 조금 좁아집니다. 결제 방식에 따라서도 할인 폭이 결정되는데 전체 선결제 > 부분 선결제 > 선결제x 순으로 정해집니다. 예약 인스턴스를 구매하거나 또한 판매도 가능합니다 (Instance Marketplace).

인스턴스 유형과 운영체제 및 테넌시를 변경할 수 있는 상품도 존재하는데 그것이 바로 Convertible Reserved Instance 입니다. 그만큼 할인 폭도 좁아집니다.

그래봤자 Reserved Instance는 최대 72%할인, Convertible Reserved Instance는 최대 66%할인입니다.

## 3. Saving Plans (절감형 플랜)

예약 인스턴스와 함께 1년 또는 3년으로 예약할 수 있습니다. 보통 시간 당 10$을 약정하며, 초과하는 사용량에 대해서는 On demand 가격으로 청구됩니다.

Reserved Instance보다 여유롭습니다. 따라서 Instance Size 및 운영체제, 테넌시 등을 전환할 수 있습니다.

## 4. Spot Instance (스팟 인스턴스)

돈이 없을 때에는 가장 탁월한 선택인 Spot Instance입니다. 예약 인스턴스에서는 가장 큰 혜택을 봐도 72% 할인이 되지만, 스팟 인스턴스에서는 무려 90% 가량 할인을 받을 수 있습니다.

그 이유는 인스턴스가 언제든 중단될 수 있다는 점 때문인데요, 여러분이 지불하고자 하는 가격보다 스팟 가격이 더 높아지게 되면 인스턴스가 중단됩니다.

따라서 배치 작업, 데이터 분석, 이미지 처리, 분산된 워크로드 등 다양하고 유동적인 상황에서 쓰이면 좋은 반면 데이터베이스와 같이 중요한 일에 대해서는 절대 사용하면 안 됩니다.

## 5. Dedicated Host (전용 호스트)

EC2 인스턴스 용량을 가진 물리적 서버입니다. 규정 준수 요구 사항이 있거나 기존의 서버 결합 소프트웨어 라이선스를 사용해야 할 경우에 사용됩니다. 실제 물리적 서버를 갖추었기 때문에 가장 비싸며, 소켓, 코어 및 VM 라이선스 당 비용이 청구됩니다.

## 6. Dedicated Instance (전용 인스턴스)

여러분의 전용 하드웨어에서 실행되는 인스턴스이며 물리적 서버와는 다릅니다. 같은 계정의 다른 인스턴스와 하드웨어를 공유하기도 하며, 인스턴스 배치는 제어할 수 없습니다.

사용자 하드웨어에 고유한 인스턴스를 갖는 것이며 전용 호스트는 물리적 서버 자체에 접근하여 저수준 하드웨어에 대한 가시성을 제공합니다.

---

# Private vs Public vs Elastic IP

<img src="https://github.com/oueya1479/aws-101/assets/147911523/c03e425f-3638-4247-89e9-05d4ac9e469b">

- private : 사설 IP 주소
- public : 공인 IP 주소
- Elastic IP : 탄력적 IP 주소

```
탄력적 IP 주소가 뭔가요?
기본적으로 EC2는 서버가 재시동되면 IP주소가 바뀌게 됩니다. 따라서 만약 'ssh'로 ec2를 접근할 때 수동으로 접근하고 있었거나, 어떤 IP를 미리 코드상에 지정해놓았을 때 지속적으로 바꾸어야 하는 문제가 생기기 때문에 이를 방지하고자 고정 IP인 Elastic IP를 사용합니다. 
```

그러나 Elastic IP의 사용을 추천하지 않습니다. `LoadBalancer`와 `AutoScaling`을 통해 충분히 IP가 바뀌어도 추적할 수 있도록 만들어야 합니다. **고정적으로 IP가 들어가는 곳**이 있다면 그 부분을 변경하는 것을 추천드립니다.

---

# Placement Groups

EC2 인스턴스가 AWS 인프라에 배치되는 방식을 제어하고 싶을 때 사용합니다.

- Cluster : 인스턴스를 가용 영역 안에 서로 근접하게 패깅하는 것
- Partition : 인스턴스를 논리적 파티션에 분산하여 한 파티션에 있는 인스턴스 그룹이 다른 파티션의 인스턴스 그룹과 기본 하드웨어롤 공유하지 않도록 하는 것
- Spread : 소규모의 신스턴스 그룹을 다른 기본 하드웨어로 분산하는 것

## Cluster  

![image](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-cluster.png)

장점 : 아주 좋은 네트워크 성능을 제공합니다. 대략 10Gbps의 대역폭입니다.

단점 : 만약 rack이 실패한다면 모든 인스턴스는 동시에 실패하게 됩니다. 위험 수준이 높습니다.

> 좋은 속도를 가지고 있기 때문에 매우 빨리 완료되어야 하는 빅데이터 작업을 수행할 수 있으며 실패를 겪을 위험을 감수하면서 빠른 지연속도를 제공해주어야 할 때 사용합니다.

## Partition

![image](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-partition.png)

장점 : 같은 파티션 내 최대 수백 개의 EC2 인스턴스를 배치할 수 있으며, 하드웨어를 전체 공유하지 않으므로 하나의 파티션이 종료된다고 해서 다른 파티션이 종료되지 않습니다.

단점 : 비용이 가장 많이 듭니다.

> 파티션은 총 7개만 존재할 수 있으며, 같은 리전에 여러 개의 가용영역을 사용하는 방식입니다. HDSK, HBase, Kfaka와 같은 서비스에서 활용합니다.

## Spread

![image](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-spread.png)

장점 : 가용영역에 분산시킬 수 있습니다. 실패의 가능성을 줄이고 인스턴스도 다른 물리적인 하드웨어와 분리됩니다.

단점 : 가용영역당 7개의 인스턴스로 제한됩니다. 즉, 많은 트래픽이 요구되는 곳에서는 제한될 수 있습니다.

> 가용성을 극대화하고 위험을 줄여야 하는 서비스에 대해 많이 사용됩니다.

---

# Elastic Network Interfaces (ENI)

`탄력적 네트워크 인터페이스`를 ENI라고 합니다. EC2 인스턴스가 네트워크에 액세스할 수 있게 해줍니다. VPC에서 가상 네트워크 카드를 나타내는 논리적 네트워킹 구성 요소입니다.

다음 속성이 포함될 수 있습니다.
- VPC의 IPv4 주소 범위 중 기본 프라이빗 IPv4 주소
- VPC의 IPv6 주소 범위 중 기본 IPv6 주소
- VPC의 IPv4 주소 범위 중 하나 이상의 보조 프라이빗 IPv4 주소
- 프라이빗 IPv4 주소당 한 개의 탄력적 IP 주소(IPv4)
- 한 개의 퍼블릭 IPv4 주소
- 한 개 이상의 IPv6 주소
- 하나 이상의 보안 그룹
- MAC 주소
- 원본/대상 확인 플래그
- 설명

> 생각보다 이해하기 어려운 서비스입니다. 이 [AWS Blog](https://aws.amazon.com/ko/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/)를 참조하여 읽어보는 것을 추천드립니다. 또한 VPC 파트를 먼저 이해하는 것 또한 추천드립니다.

---

# EC2 Hibernate

> [AWS Blog](https://aws.amazon.com/ko/blogs/korea/new-hibernate-your-ec2-instances/)를 참조하면 좋습니다.

EC2를 절전모드로 사용합니다. 기존에 EC2에 Stop명령을 내리면 단순히 다음 시작 전까지 데이터를 디스크(EBS)에 저장하여 대기 상태로 변경됩니다. Terminate는 EC2에 대한 사용을 아예 중지시켜 `EBS Volume 자동 삭제 기능`을 켜놓았을 경우 EBS마저 삭제됩니다.

시작할 때에는 OS의 시작으로 EC2 유저 데이터 스크립트를 실행하고, 그 이후 OS가 완전히 실행되는 과정을 거치게 됩니다.

![image](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/hibernation-flow.png)

그러나 절전모드를 사용한다면 EC2 RAM에 저장되어 있던 데이터가 EBS Volume으로 이동하며 인스턴스는 정지상태로 변경됩니다. 이후 다시 시작될 때 EBS에 있었던 RAM 데이터가 EC2로 이동하여 더욱 빠른 시작을 가능하게 합니다.

> 마치 Redis에서 사용하는 방식과 비슷한 것 같습니다. 따라서 EBS에는 EC2의 RAM 데이터를 저장할 수 있는 여유 공간이 필요로 합니다.

**하지만 다음 주요 사항을 확인해야 합니다.**

1. 인스턴스 유형 : Hibernate 기능은 모든 인스턴스 유형에서 사용할 수는 없습니다.
2. Root volume 크기 : Hibernate가 성공적으로 설정되려면 루트 볼륨에 인스턴스 RAM 용량과 같은 크기의 여유 공간이 있어야 합니다.
3. 저장 기간 : 최대 60일까지 사용할 수 있습니다.