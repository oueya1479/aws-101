![global](https://github.com/oueya1479/aws-101/assets/147911523/2975d295-efd3-46f2-91c6-bb78c2533bcb)

<small>Author: heony</small>

- [Global Accelerator](#global-accelerator)
  - [Unicast IP vs Anycast IP](#unicast-ip-vs-anycast-ip)
  - [Global Accelerator vs CloudFront](#global-accelerator-vs-cloudfront)

# Global Accelerator

![image](https://d1.awsstatic.com/product-page-diagram_AWS-Global-Accelerator%402x.dd86ff5885ab5035037ad065d54120f8c44183fa.png)

글로벌 애플리케이션을 구축했다고 가정하고, 하나의 지역에만 로드밸런서를 설정했다고 해봅시다.
사용자들이 애플리케이션에 접근할 때에는 공용 인터넷을 사용하지만 라우터를 거치는 시간 동안 지연이 일어날 수도 있습니다(여러 개의 홉을 거치므로). 이는 연결이 끊어지거나 지연시간이 발생할 수도 있습니다.

지연 시간을 최소화하기위해 AWS Network를 빨리 접근하는게 좋습니다. 이때 Global Accelerator를 사용합니다.

**애플리케이션의 가용성과 성능 및 보안을 개선하고, DDos 공격으로부터 애플리케이션을 보호**할 수 있습니다.

- Elastic IP, EC2 instances, ALB, NLB, public or private
- 지속적인 퍼포먼스
- 장애 복구
- DDos 보안 등 다양한 기능

## Unicast IP vs Anycast IP

> - Unicast IP: 하나의 IP만 가지는 것, 클라이언트가 두 개의 서버와 통신할 때 각각 아이피로 그냥 접근합니다.
> - Anycast IP: 모든 서버가 동일한 IP를 가지고 있는 것, 자신과 가장 가까운 곳으로 서버를 사용합니다.

위의 개념을 통하여 Global Accelerator가 작동할 수 있습니다.

## Global Accelerator vs CloudFront
둘 다 `AWS Global Network`이며 DDoS 방어를 할 수 있습니다.

**CloudFront**
- 이미지나 비디오처럼 캐싱 가능한 정적 데이터 및 동적 내용의 성능을 향상시킵니다.
- 엣지 로케이션으로부터 제공됩니다.

**Global Accelerator**
- TCP나 UDP상의 애플리케이션 성능을 향상시킵니다.
- 패킷은 하나 이상의 리전에서 실행되는 애플리케이션으로 프록시됩니다. 캐싱은 일어나지 않습니다.
- 글로벌하게 고정 IP를 요구하는 HTTP를 사용할 때도 유용합니다.
- 결정적이고 신속한 리전 장애 복구가 가능합니다.