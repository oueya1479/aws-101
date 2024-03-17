![vpc](https://github.com/oueya1479/aws-101/assets/147911523/51fb0019-5c8f-42d1-afa4-15f70bd6a042)

<small>Author: heony</small>

### 목차

- [VPC](#vpc)
  - [CIDR](#cidr)
  - [Subnet](#subnet)
  - [Internet Gateway (IGW)](#internet-gateway-igw)
    - [Bastion Hosts](#bastion-hosts)
  - [NAT Gateway](#nat-gateway)
  - [NACL](#nacl)
    - [Seciruty Groups \& NACL](#seciruty-groups--nacl)
    - [Ephemeral Ports](#ephemeral-ports)
  - [VPC Peering](#vpc-peering)
  - [VPC Endpoints (AWS PrivateLink)](#vpc-endpoints-aws-privatelink)
    - [Types of Endpoints](#types-of-endpoints)
  - [VPC Flow Logs](#vpc-flow-logs)
  - [Site-to-Site VPN](#site-to-site-vpn)
  - [Direct Connect (DX)](#direct-connect-dx)
  - [Transit Gateway](#transit-gateway)
    - [Site-to-Site VPN ECMP](#site-to-site-vpn-ecmp)
  - [Network Protection](#network-protection)
- [IPv6](#ipv6)
  - [Egress-only Internet Gateway](#egress-only-internet-gateway)

# VPC

> AWS 계정을 생성하면 Default VPC가 적용됩니다. 해당 VPC는 172.31.0.0/16으로 총 `65,536`개의 호스트를 제작할 수 있습니다. Subnet 역시 3개로 분리되어 있는데 각각 `172.31.0.0/20`, `172.31.16.0/20`, `172.31.32.0/20` 입니다.
>
> 위 내용이 이해하기 어렵다면 VPC 문서를 읽어보아야 합니다.

VPC는 Virtual Private Cloud으로 각 리전 당 5개정도 제한으로 두고 있습니다. [CIDR](#cidr)을 최소 /28 (16개의 주소)와 최대 /16(65,536개의 주소)로 지정할 수 있습니다.

## CIDR

Classless Inter-Domain Routing으로, 아이피 주소의 할당을 위한 방법입니다.

> WW.XX.YY.ZZ/32 는 하나의 IP만을 나타냅니다.
> 0.0.0.0/0 은 모든 IP를 나타냅니다.
> 192.168.0.0/26 은 192.168.0.0. ~ 192.168.0.63 까지 총 64개의 아이피를 나타냅니다.

- Base IP: 범위안에 포함된 IP를 의미합니다.
  - ex) 10.0.0.0, 192.168.0.0 ...
- Subnet Mask: IP에서 변경 가능한 비트의 개수를 정의합니다.
  - ex) /0, /24, /32 등

```md
`192.168.0.0/32` -> 192.168.0.0
`192.168.0.0/31` -> 192.168.0.0 ~ 192.168.0.1
`192.168.0.0/30` -> 192.168.0.0 ~ 192.168.0.3
`192.168.0.0/29` -> 192.168.0.0 ~ 192.168.0.7
`192.168.0.0/16` -> 192.168.0.0 ~ 192.168.255.255
`192.168.0.0/0` -> 0.0.0.0 ~ 255.255.255.255
```

> 자세한 내용으로 테스트해보려면 [여기](https://www.ipaddressguide.com/cidr)를 클릭하세요.

이러한 방식으로 CIDR은 이루어져있습니다. 이를 통해 네트워크의 범위에 따라 아이피 할당을 다르게 할 수 있습니다.

- 10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8) 는 범위가 큰 네트워크에서 사용합니다.
- 172.16.0.0 ~ 172.31.255.255 (172.16.0.0/12) 는 AWS의 디폴트 범위입니다.
- 192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16) 는 보통의 집에 해당하는 네트워크에서 사용합니다.

## Subnet

IPv4 주소의 부분 범위입니다. 이 범위 내에서 AWS는 IP 주소 다섯 개를 예약하는데, 각 서브넷마다 IP 주소 처음 네 개와 마지막 한 개를 예약합니다. 이 IP는 사용도 되지 않으며 EC2 인스턴스에 IP로 할당되지도 않습니다.

> 각 디폴트 서브넷 접속해서 보면 4,096개가 사용 가능해야 하지만, 4,091개만 사용 가능한 것을 보실 수 있습니다.

CIDR가 10.0.0.0/24라고 가정했을 때
1. 10.0.0.0 - 네트워크 주소입니다.
2. 10.0.0.1 - AWS VPC 라우터로 예약됩니다.
3. 10.0.0.2 - Amazon-privded DNS로 예약됩니다.
4. 10.0.0.3 - 미래에 사용될 AWS 서비스로 예약됩니다.
5. 10.0.0.255 - VPC에서 브로드캐스트를 지원하지 않기에 예약은 되지만 사용되지는 않습니다.

💡 따라서 29개의 주소가 필요하다면 10.0.0.0/27 이라고 가정 했을 때 32개의 주소를 사용할 수 있을 것 같지만 사실 32 - 5 = 27개만 사용가능하기 때문에 **/26을 사용해야 합니다.**

## Internet Gateway (IGW)

![image](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/subnet-association.png)

인터넷 게이트웨이는 인터넷 액세스를 위한 서비스입니다. VPC와는 별개로 생성해야 하며, VPC는 오직 하나의 IGW로 연결됩니다.

예를 들어 VPC 내부에 public, private subnet이 각 한 개씩 있다고 가정하고, Internet Gateway가 VPC와 연결되어 있다면 이는 Router에 의해 public subnet이 인터넷을 사용할 수 있도록 만들 수 있습니다.

이제 위의 사진이 이해가 가시나요? 몇 가지 특징점들을 보면 다음과 같이 설명할 수 있습니다.

1. Region안에 VPC가 구축되어 있습니다.
2. VPC는 Public subnet과 Private subnet이 구분되어 있습니다. 각각의 CIDR는 모릅니다.
3. 인터넷 연결을 위해 Internet gateway를 VPC에 설치했습니다.
   - 단순히 Internet gateway를 VPC에 설치하는 것만으로 EC2가 인터넷을 사용할 수 있는건 아닙니다.
4. Public route table에서 0.0.0.0/0으로 Target을 IGW로 설정해주어야 합니다.

### Bastion Hosts

Private subnet에 있는 EC2가 인터넷 액세스가 필요한 경우가 있습니다. 이 상황에서는 `Bastion Hosts`를 사용할 수 있습니다. 중요한 점은 **Bastion Hosts는 public subnet에 위치한다는 것입니다.**

즉 internet과 같은 public access를 받을 수 있는 EC2이자 private subnet에 접근할 수 있는 서버가 바로 Bastion Hosts인 셈입니다.

몇 가지 조건이 있습니다.
1. **Bastion Hosts는 모든 트래픽을 허용해서는 안 됩니다.** 만약 모든 트래픽을 받아들일 수 있다면, private subnet에 존재하는 ec2에게도 전체 접근할 수 있는 상황이 벌어집니다.
2. private subnet의 ec2에게는 ssh로 접근할 수 있어야 하빈다.
3. private subnet의 ec2들은 ssh 즉, 22번 포트를 열어야 합니다.

## NAT Gateway

![image](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/public-nat-gateway-diagram.png)

NAT이란 Network Address Translation 즉, 네트워크 주소 변환 서비스입니다. 보통 VPC 내부에는 public subnet과 private subnet 이 존재합니다. private subnet은 인터넷 게이트웨이(IGW)가 직접적으로 연결되어 있지 않아 인터넷 사용이 어렵습니다.

private subnet의 인스턴스가 VPC 외부의 서비스를 연결할 수 있으면서도 외부 서비스에서 이 인스턴스와의 연결을 시작할 수 없도록 `NAT Gateway`가 사용됩니다.

위의 사진에서 삼지창 처럼 되어있는 아이콘이 바로 NAT Gateway입니다. **Public subnet에서 활동하여 Private에서의 인터넷 연결(⋂ 모양)을 도와줍니다.**

## NACL

> NACL, 나클이라고 부르곤 합니다.

NACL이란 Network Acess Control List 즉, 네트워크의 액세스 제어 목록입니다. 서브넷으로부터 특정한 IP를 차단하는데 좋으며, 방화벽의 역할을 합니다. 서브넷당 하나의 NACL을 가지고 있으며 새로운 서브넷에는 Default NACL이 할당됩니다.

몇 가지 규칙이 존재합니다.
- 1 ~ 32,766개의 우선순위를 가질 수 있습니다.
- 만약 # 100 ALLOW(10.0.0.10/32) 와 # 200 DENY(10.0.0.10/32)라고 지정하게 되면 100의 우선순위를 가진 규칙에 먼저 도달합니다.

자세한 내용은 아래에서 사진과 함께 다룹니다.

### Seciruty Groups & NACL

![image](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/network-acl.png)

Security Groups은 EC2를 한번이라도 사용해보신 분이라면 익숙할 것이라 생각합니다. 데이터베이스나 혹은 서버를 열기위해 아무리 서버를 호스팅해도, 시간초과에 걸렸더라면 모두 Security Groups의 문제였으니까요.

Security Groups에서는 포트를 개폐할 수 있는 장치가 있습니다. 예를 들어, 스프링 서버로 8080포트를 열었다면, Security Groups에서도 8080(inbound)를 열어야 작동하기 때문입니다.

**그런데 왜 8080포트를 열면 outbound를 설정하지 않아도 어떻게 사용자의 요청에 대답할 수 있는 것일까요?**

Security Groups는 상태 저장(Stateful) 상태를 가지고 있습니다. 즉, 어떤 요청이 들어오게 되었을 때 그 요청을 저장하고, 이후 요청에 대한 처리가 종료되면 저장했던 상태를 통하여 outbound로 내보낼 수 있는 것입니다.

NACL은 이와 반대로 상태 비저장입니다. 보안 그룹처럼 Inbound, Outbound의 상태를 지정할 수 있어 트래픽에 대한 제어를 할 수 있으나, 상태 비저장 상태이므로 Inbound의 트래픽 따로, 내부에서 요청된 데이터를 외부로 내보내는 Outbound의 트래픽이 따로 움직입니다. **이러한 점으로 상태 비저장이라고 불리는 것입니다.**

### Ephemeral Ports

Ephemeral는 '일시적인' 이라는 뜻입니다. 우리가 어떤 서비스를 사용하게 되면 클라이언트로서 참여하게 될 텐데, 우리는 어떻게 포트를 개폐할 수 있을까요?

각 OS마다 다른 규칙을 가지고 있는데요, 예를 들면 IANA & MS WIndows 에서는 `49152 ~ 65535`이고, Many Linux Kernels에서는 `32768 ~ 60999`를 임시 포트로 사용하고 있습니다.

```bash
Client(11.22.33.44) ---> Web Server(55.66.77.88)

Src.IP: 11.22.33.44
Src.Port: 50105 # 임시포트로 열림
Dest.IP: 55.66.77.88
Dest.Port: 443
Payload...

Web Server(55.66.77.88) ---> Client(11.22.33.44)

Payload...
Dest.IP: 11.22.33.44
Dest.Port: 50105 # 임시포트가 재사용됨
Src.IP: 55.66.77.88
Src.Port: 443
```

## VPC Peering

서로 다른 두 VPC 간 상호 연결을 위해 VPC Peering을 진행할 수 있습니다. 예를 들어 VPC-A와 VPC-B 간 연결을 위해서 VPC Peering을 진행하면 서로 다른 두 네트워크가 하나의 VPC 안에서 움직이는 것 처럼 만들 수 있습니다.

> 위의 상황에서 VPC-B가 VPC-C와 VPC Peering을 하면 VPC-A에서도 VPC-C에 접근할 수 있을까요?
> **답은 '아니다'입니다.** 서로 연결하기 위해서는 꼭 직접 VPC Peering을 진행해야만 합니다. 즉, VPC-A와 VPC-C가 연결되어 있어야만 VPC Peering을 온전히 사용할 수 있습니다.

또한 VPC Peering을 연결하더라도 라우팅테이블에 대한 규칙을 수정해주어야만 상호간 연결이 작동합니다.

## VPC Endpoints (AWS PrivateLink)

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*p90DUmYzTOw4EMrIdv7pcg.png)

<small>해당 이미지는 [여기](https://awstip.com/aws-vpc-endpoint-gateway-using-terraform-82c641ac90ca)에서 확인할 수 있습니다.</small>

Private Subnet에서 AWS 서비스를 사용하기 위한 방법으로는 크게 두 가지로 볼 수 있습니다.

1. Private Subnet -> **NAT Gateway** -> **Internet Gateway(IGW)** -> Internet -> Amazon S3

그러나 이 방법은 중간에 허브를 많이 두어야 합니다. 또한 비용도 그만큼 청구될 것입니다. 이러한 불편함을 위해 VPC Endpoints 서비스가 탄생했습니다.

2. Private subnet -> **VPC Endpoint** -> Amazon S3

이는 AWS의 서비스에서만 가능합니다. 즉 Amazon S3는 AWS가 지원하는 서비스이므로 Endpoint에서 접근할 수 있게 되는 것이며, 이 때 사용되는 연결 통로는 AWS PrivateLink라고 볼 수 있겠습니다.

### Types of Endpoints

엔드포인트는 2가지 종류로 나누어볼 수 있습니다.

1. Interface Endpoints
  - PrivateLink에 의해 움직이며, AWS에서 지원하는 대부분의 서비스를 이용할 수 있습니다.
  - 가격은 시간당 + 데이터 처리당 으로청구됩니다.
2. Gateway Endpoints
  - 라우팅테이블의 Target으로서만 사용되어야 합니다.
  - Target으로 S3와 DynamoDB만 사용 가능합니다.
  - 무료입니다.

## VPC Flow Logs

IP 트래픽이 VPC에 접근하는 로그들에 대하여 확인할 수 있습니다.

- VPC 플로우 로그
- 서브넷 플로우 로그
- ENI 플로우 로그

이렇게 3가지에 대한 로그 기록을 확인할 수 있으며, 이를 통해 모니터링 및 트러블슈팅이 가능합니다.

```bash
# 다음과 같은 형식으로 로그가 작성됩니다.
2 123456789010 eni-1235b8ca123456789 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK

2 - version
123456789010 - account-id
eni-1235b8ca123456789 - interface-id
172.31.16.139 - srcaddr : IP추적
172.31.16.21 - dstaddr : IP추적
20641 - srcport : 포트 추적
22 - dstport : 포트 추적
6 - protocol
20 - packets
4249 - bytes
1418530010 - start
1418530070 - end
ACCEPT - action
OK - log-status
```

`ACTION` 필드를 통해 Security groups 혹은 NACL의 상태를 확인할 수 있습니다.

**Imcoming Requests**
- Inbound REJECT일 경우 NACL 혹은 SG 이슈
- Inbound ACCEPT, Outbound REJECT일 경우 NACL 이슈
  - SG의 경우는 상태저장(Stateful)이기 때문에 Inbound에 대하여 Outbound를 지정하지 않아도 되기 때문입니다. 따라서 이 경우에는 NACL의 Outbound에서 생기는 이슈입니다.

**Outgoing Requests**
- Outbound REJECT일 경우 NACL 혹은 SG 이휴
- Outbound ACCEPT, Inbound REJECT일 경우 NACL 이슈
  - 위와 설명은 동일합니다.

또한 이러한 정보는 CloudWatch Logs, Amazon S3 bucket 등 Target을 지정하여 다른 서비스와 함께 접목되어 사용될 수 있습니다.

## Site-to-Site VPN

![image](https://docs.aws.amazon.com/images/vpn/latest/s2svpn/images/cgw-high-level.png)

AWS간 VPC 연결을 위해 VPC Peering 서비스를 사용하는 것과 달리 **On-premises network와 AWS의 Region 안에 VPC와 연결해야 하는 경우도 존재합니다.** 이는, `Site-to-Site VPN`을 사용하여 해결할 수 있습니다.

Site-to-Site VPN는 두 개 이상의 네트워크 위치간의 안전한 연결을 생성하는 데 사용되는 가상 사설망 기술입니다. 한 지점(Site)의 네트워크에서 다른 지점의 네트워크로 데이터를 안전하게 전송할 수 있다는 의미를 가지고 있습니다.

먼저 위의 사진에서 Virtual private gateway(이하 VGW)와 Customer gateway(CGW)를 이해해야 합니다.

**VGW**
- AWS단에 있는 VPN 연결자입니다.
- Site-to-Site VPN연결을 생성하기 원하는 VPC로부터 생성됩니다.

**CGW**
- VPN연결의 Customer단 소프트웨어 애플리케이션 또는 물리적 장치입니다.

온프레미스의 CGW Device를 이용하여 연결을 시도할 수 있습니다. 여기서 Public IP 주소를 가지고 있다면 VGW와 직접적으로 연결할 수 있지만, 만약 IP가 NAT 디바이스에 감춰져 있는 경우(즉, NAT-T가 켜져있는 경우)에는 NAT 디바이스의 Public IP를 활용하여 연결하면 됩니다.

## Direct Connect (DX)

Site-to-Site 와는 조금 다르게, VPC간 전용 Private 연결을 가능하게 합니다. 큰 데이터 세트를 처리해야할 때 또는 비용을 절감해야 하는 경우 사용합니다. 또한, 실시간 데이터 피드를 사용하는 경우에도 유용합니다.

Direct Connect Gateway의 경우 다중 리전의 VPC를 사용할때 유용합니다. **각 VPC의 VGW가 Direct Connect Gateway와 연결되어 다중 리전의 VPC와 연결될 수 있습니다.**

> 속도가 빠른 만큼 단점도 존재합니다. 설치 기간이 무려 한 달 정도 걸리기 때문에, **만약 일주일 내에 빠르게 데이터를 전송해야 하는 일이 있다고 하면 Direct Connect를 사용할 수 없을 것입니다.**

기본적으로 암호화되지 않지만 애초에 Private한 연결이기 때문에 괜찮습니다. 암호화가 필요한 경우 VPN을 활용하여 추가적으로 암호화를 구현할 수 있지만 이는 *조금 어렵다고 합니다.*

## Transit Gateway

![image](https://docs.aws.amazon.com/images/whitepapers/latest/aws-vpc-connectivity-options/images/aws-transit-gateway.png)

Transit Gateway는 너무 많은 VPC 연결이 전체 지도를 보기 어렵게 만든다는 것부터 시작된 아이디어입니다. VPC Peering과 Site-to-Site VPC 및 Direct Connect 등 다양한 VPC가 서로 연결되면 한 눈에 알아보기 힘든 거미줄처럼 서로 연결되게 됩니다.

이러한 문제점을 해결하기 위해 `Transit Gateway`가 등장했고, 이는 각 VPC와 서로 연결되어 시작 및 목표지점에 대한 연결을 도와줍니다. Site-to-Site 및 Direct connect의 요청에 맞게 서로를 연결하여 네트워크 문제 해결에 도움을 줄 수 있습니다.

모든 트래픽 제어를 관리하여 VPN 연결과 함께 작동하고, *IP 멀티캐스트를 지원합니다.*

### Site-to-Site VPN ECMP

![image](https://docs.aws.amazon.com/images/wellarchitected/latest/hybrid-networking-lens/images/termination-transit-gateway.png)

Site-to-Site VPN 기술임에도 불구하고 Transit Gateway 목차 뒤에 설명하는 이유는 **TGW에서만 가능한 기술이기 때문입니다.**

ECMP는 Equal-Cost Multi-Path routing 이며, On-Premises에서 VPN 서버와 Site-to-Site로 연결되었을 때 속도와 처리량을 더욱 빠르게 만들어줄 수 있는 서비스입니다. Site-to-Site로 연결시 두개의 터널당 VPN의 속도는 1.25 Gbps입니다. **VPC에 직접 연결 시에는 하나의 터널이 두개 생성됩니다.**

TGW에 연결한다면 그 connection의 수를 늘릴 수 있으며 터널도 각각 사용될 수 있습니다. connection만 3개로 생성하더라도 6개의 터널로 구축되며 이는 7.5 Gbps의 속도를 낼 수 있게 됩니다.

## Network Protection

AWS 네트워크를 보호하는 데에는 여러 가지 서비스가 존재한다는 것을 알게되었습니다.
1. Network Access Control List(NACLs)
2. Amazon VPC security groups
3. AWS WAF
4. AWS Shield
5. AWS Firewall Manager 등

전체 VPC를 보호하는 방법으로는 `AWS Network Firewall`을 사용하면 됩니다.

VPC 내부의 Layer3부터 Layer7에서의 모든 방향 및 트래픽을 관리하고 보호할 수 있습니다. VPC to VPC traffic, 인터넷으로의 Outbound 및 Inbound 등 VPC의 모든 입출력과 관련된 곳에서 보호할 수 있습니다.

---

# IPv6

IPv4는 43억개의 주소를 나타낼 수 있습니다. 그러나 생각보다 빠른 고갈로 인해 새로운 주소 체계가 필요하게 되었으며 이는 IPv6의 탄생이 되었습니다.

Format은 x.x.x.x.x.x.x.x 형태이며 각 x에는 0000 부터 ffff까지의 수가 들어갈 수 있습니다.
- `2001:db8:3333:4444:5555:6666:7777:8888`
- `2001:db8:3333:4444:cccc:dddd:eeee:ffff`

위와 같은 형태를 가지고 있습니다.

**IPv4의 경우 VPC와 subnet에서는 이를 비활성화할 수 없습니다.** 따라서 EC2 Instance를 서브넷에서 시작할 수 없는 경우에는 IPv6 때문이 아닌, IPv4의 이용 가능한 CIDR를 의심해야 합니다.

IPv6의 경우 그 공간이 매우 크기때문에 보통 IPv4에서 지원가능한 공간이 부족하여 나타나는 문제로 의심해볼 수 있겠습니다.

## Egress-only Internet Gateway

Egress-only Internet Gateway는 IPv6용 Internet Gateway입니다.