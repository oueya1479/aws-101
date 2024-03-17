![cloudtrail](https://github.com/oueya1479/aws-101/assets/147911523/e4d09a18-53f9-48aa-918e-10fe6615d64a)

<small>Author: heony</small>

### 목차

- [AWS CloudTrail](#aws-cloudtrail)
  - [CloudTrail Events](#cloudtrail-events)
  - [Cloud Events Retention](#cloud-events-retention)

# AWS CloudTrail

**CloudTrail은 AWS 계정의 거버넌스, 컴플라이언스, 감사를 실현하는 방법입니다.** 모든 이벤트와 API는 바로 CloudTrail이 수집하기 때문에, 어떤 문제가 발생하거나 사용자를 추적해야 하는 경우 CloudTrail을 사용할 수 있습니다.

> 가령 어떤 사용자가 EC2를 무단으로 종료한 경우, 이 사용자의 신원을 밝혀야 한다면 AWS CloudTrail을 사용할 수 있습니다.

SDK, CLI, Console, IAM Users & Roles로부터 추적되는 모든 데이터를 CloudTrail Console에서 확인할 수 있으며, **이는 CloudWatch Logs 혹은 S3에 이동하여 로그를 저장할 수도 있습니다.**

## CloudTrail Events

CloudTrail에서 수집하는 이벤트의 종류는 3가지로 나눌 수 있습니다.

**1. Management Events**

이는 AWS 계정의 리소스로부터 생성된 운영 이벤트입니다. 예를 들어 보안 구성(IAM)이나 EC2 라우팅 데이터에 대한 권한 수정, 로깅 설정 등의 이벤트입니다. 이는 기본으로 구성되어 있어 따로 설정하지 않아도 로그를 관리할 수 있습니다.

Read Event와 Write Event로 나눌 수 있는데, 리소스를 수정하지 않고 단순히 데이터를 불러오는 것이 `Read Event`, 무언가 리소스를 수정하거나 생성하는 것이 `Write Event`입니다.

**2. Data Events**

S3에 GetObject, DeleteObject, PutObject가 수행되는 것은 정말 빈번하게 일어날 수 있습니다. 이러한 세부적인 데이터 사항은 기본적으로 로깅되지 않습니다. 이는 따로 설정한 경우 확인할 수 있지만, 객체 수준의 활동은 기본적으로 로깅지원이 되지 않음을 이해해야 합니다.

**3. CloudTrail Insights**

부정확한 리소스 프로비저닝, 서비스 한도 도달, IAM 액션의 폭증 등 비정상적인 패턴을 감지하는 역할을 수행합니다. <u>**이는 정상적인 관리 활동을 분석하여 기준선을 만들고, 올바른 형태의 이벤트인지 계속 분석하기 때문에**</u>, 비정상적인 패턴에 대한 분석을 실행할 수 있는 것입니다.

## Cloud Events Retention

기본값으로 90일 동안 보관되며 그 기간이 지나면 삭제됩니다. 하지만 감사를 목적으로 하여 이벤트를 더 오랜 기간동안 보관할 수 있으며, S3에 전송 및 Athena 사용을 통한 분석이 이루어져야 합니다.