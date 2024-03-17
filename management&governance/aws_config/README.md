![config](https://github.com/oueya1479/aws-101/assets/147911523/e00ec1f3-ac71-4680-a737-cb3088669df8)

<small>Author: heony</small>

### 목차

- [AWS Config](#aws-config)
  - [Remediations](#remediations)
  - [Notification](#notification)

# AWS Config

AWS Config는 AWS 리소스 내에 대한 감사와 규정 준수 여부를 기록할 수 있도록 해주는 서비스입니다.

> 보안 그룹에 제한되지 않은 ssh 접근이 있는지, 버킷에 공용 액세스가 있는지 등의 규정 준수 여부를 파악하고 이를 알림으로 전달할 수 있습니다.

이러한 규칙은 직접 제작할 수도 있으며, AWS에서 관리하는 Config 규칙을 사용할 수도 있습니다. 예를 들면, EBS 디스크가 gp2 유형인지 평가하거나 혹은 EC2의 인스턴스가 t2.micro인지 확인하는 Config 규정을 만들 수 있습니다.

**어떤 동작을 미리 예방할 수는 없습니다. 하지만 구성의 개요와 리소스의 규정 준수 여부는 Config를 통해 확인할 수 있습니다.**

## Remediations

AWS Config에서 규정을 준수하지 않는 리소스가 확인이 된다면 이를 모니터링하여 교정할 수 있습니다. `SSM Document`를 트리거하여 규정을 준수하지 않는 리소스에 대하여 교정할 수 있는 코드를 실행시키고, 이는 어떤 오류에 의해 수행되지 않을 시 최대 5번 재수행시킬 수 있습니다.

## Notification

EventBridge와 SNS를 통합하여 규정을 준수하지 않는 리소스에 대한 알림을 설정할 수 있습니다. 특정 이벤트에만 적용되게 필터링하고 싶다면 SNS 필터링을 사용하여 SNS 주제를 필터링할 수 있습니다.