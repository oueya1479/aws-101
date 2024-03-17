![cloudwatch](https://github.com/oueya1479/aws-101/assets/147911523/7f0245f0-f51a-4464-82b7-68a310ec9261)

<small>Author: heony</small>

### 목차

- [Amazon CloudWatch](#amazon-cloudwatch)
- [Amazon CloudWatch Metrics](#amazon-cloudwatch-metrics)
- [Amazon CloudWatch Logs](#amazon-cloudwatch-logs)
  - [CloudWatch Logs Insights](#cloudwatch-logs-insights)
  - [CloudWatch Logs Subscriptions](#cloudwatch-logs-subscriptions)
  - [CloudWatch Logs Live Tail](#cloudwatch-logs-live-tail)
  - [CloudWatch Logs for EC2](#cloudwatch-logs-for-ec2)
- [Amazon CloudWatch Alarms](#amazon-cloudwatch-alarms)
  - [Composite Alarms](#composite-alarms)
- [Amazon CloudWatch EventBridge](#amazon-cloudwatch-eventbridge)
  - [Event Bus](#event-bus)
  - [Schema Registry](#schema-registry)
  - [Resource-based Policy](#resource-based-policy)
- [Amazon CloudWatch Insights](#amazon-cloudwatch-insights)
  - [1. CloudWatch Container Insights](#1-cloudwatch-container-insights)
  - [2. ClooudWatch Lambda Insights](#2-clooudwatch-lambda-insights)
  - [3. CloudWatch Contributor Insights](#3-cloudwatch-contributor-insights)
  - [4. CloudWatch Application Insights](#4-cloudwatch-application-insights)

# Amazon CloudWatch

CloudWatch는 AWS의 대부분의 서비스에 대한 모니터링 시스템을 제공합니다. 용도에 따라 CloudWatch Metrics, Alarms, Logs, EventBridge 및 Insights 등으로 나뉘게 됩니다.

해당 문서에서는 이 다섯 가지 서비스들을 다룹니다.
- `Amazon CloudWatch Metrics`: AWS 리소스 및 애플리케이션에서 발생하는 다양한 지표를 수집하고 모니터링할 수 있도록 해주는 서비스입니다. **시간에 따른 데이터의 시각화를 생성할 수 있습니다.**
- `Amazon CloudWatch Logs`: 애플리케이션과 AWS 리소스의 로그 파일을 수집, 모니터링, 분석할 수 있는 서비스입니다. **로그 데이터에 대한 실시간 모니터링과 저장이 가능합니다.**
- `Amazon CloudWatch Alarms`: 설정한 규칙에 따라 지표의 임계값을 넘었을 때 알람을 발생시키는 서비스입니다. **자동화된 작업 실행이나 알림을 통해 사용자에게 상태 변화를 알려줍니다.**
- `Amazon CloudWatch EventBridge`: AWS 서비스, 이벤트 패턴, 애플리케이션에서 발생하는 다양한 이벤트를 감지하여 자동으로 반응하도록 해주는 서버리스 이벤트 서비스입니다. 다양한 대상에 이벤트를 라우팅할 수 있습니다.
- `Amazon CloudWatch Insights`: CloudWatch Logs에서 수집된 로그 데이터를 분석하여 인사이트를 제공하는 서비스입니다. **복잡한 쿼리를 실행하여 시스템의 성능 문제를 진단하고 해결하는 데 도움을 줍니다.**

# Amazon CloudWatch Metrics

AWS에서 일어나는 모든 서비스들에 대해 지표를 제공합니다. `CPUUtilization`, `NetworkIn`과 같이 특정 지표에 대해 모니터링 할 수 있으며, **이러한 지표는 미리 설정된 템플릿을 사용하여 가져올 수 있습니다.** 지표는 timestamps를 포함하고 있습니다.

지표를 토대로 Dashboard를 제작할 수 있습니다. 또한, 미리 지정되어있는 variable 말고도, 사용자가 직접 필요한 데이터 지표(예를 들면 RAM 사용률 등)를 Custom 하여 지표를 제작할 수도 있습니다.

이러한 데이터는 Metric 뿐만 아니라 다른 서비스를 사용해서 더 효과적인 분석 아키텍처를 구성할 수도 있습니다. 이는 *near-real-time delivery*로 낮은 지연속도로 제공가능 합니다.

```bash
CloudWatch -> Kinesis Data Firehose -> S3 -> Athena 등
```

# Amazon CloudWatch Logs

AWS에서 애플리케이션 로그를 저장할 수 있는 완벽한 곳입니다.

> 완벽한 곳이라고 이야기한 이유는, S3, RDS 및 Elasticache 등 다양한 곳에 로그를 저장할 수 있음에도 더 뛰어난 로그 관리 서비스라는 것을 의미합니다.

이러한 로그 시스템을 구축하기 위해서는 먼저, 로그 그룹을 지정해야 합니다. 보통 애플리케이션 이름으로 지정합니다. 또한 로그 스트림을 지정해야 하는데요, 이는 애플리케이션 또는 로그 파일, 컨테이너 등을 지정하는 단계입니다. 마지막으로는 로그의 만료시간을 지정합니다. 하루 또는 10년, 더 나아가 삭제하지 않는 옵션도 존재합니다.

이러한 로그는 CloudWatch Logs 말고도 다른 서비스로 보내질 수 있습니다.
- Amazon S3
- Kinesis Data Stream
- Kinesis Data Firehose
- AWS Lambda
- OpenSearch 등

위의 서비스들로 로그들이 이동하는데, 그러면 로그의 원천은 어디서 발생할 수 있을까요?
- SDK, CloudWatch Logs Agent
- Elastic Beanstalk
- ECS
- AWS Lambda
- VPC Flow Logs
- API Gateway
- Route53 등

AWS의 대부분의 서비스에서 Logs를 발생시킬 수 있습니다.

## CloudWatch Logs Insights

![image](https://media.amazonwebservices.com/blog/2018/ci_dlg_qo_ec2_1.png)

쿼리를 사용하여 Log를 분석할 수도 있습니다. CloudWatch Logs에 저장되어 있는 데이터를 가지고 찾거나 분석할 수 있습니다. 집계 통계를 계산하거나 또는 이벤트를 정렬하거나 이벤트 개수를 제한하는 등의 작업이 가능합니다.

이는 실시간이 아닌 쿼리엔진이기 때문에 과거의 데이터에만 접근할 수 있습니다.

## CloudWatch Logs Subscriptions

S3를 사용하여 CloudWatch Logs에 있는 데이터를 모두 내보낼 수 있습니다. 그러나 이는 장시간에 걸쳐 진행될 수 있습니다. 최대 12시간 정도라고 하니, Log 데이터의 빠른 분석과 접근을 위해서는 이러한 과정이 불편할 수 있습니다.

> 이떄 사용되는 API는 CreateExportTask 입니다.

이러한 불편함을 해소하기 위해, 미리 설정해두어야 하는 것이 있습니다. 바로 실시간 스트림을 사용하여 그 데이터를 Kinesis Data Streams 또는 Lambda로 보내어 전송 대상으로 전달하려는 로그 이벤트의 종류를 지정할 수 있습니다.

또한 Subscription Filter를 통해 다양한 Region에서 들어오는 모든 Logs를 `Kinesis Data Streams`에서 받아들이고, 로그데이터를 다른 목적지로 전송시킬 수 있습니다.

```bash
CloudWatch Logs -- Subscription Filter -- |
CloudWatch Logs -- Subscription Filter --  >>  Kinesis Data Streams
CloudWatch Logs -- Subscription Filter -- |
```

## CloudWatch Logs Live Tail

![image](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/06/15/Figure-1-%E2%80%94-Live-Tail-console-1024x540.png)

Log Stream과 Log Filter를 지정하여 Stream으로부터 발생되는 Log 중 Filter의 해당하는 데이터만 실시간으로 가져와 확인할 수 있습니다. 로그를 쉽게 디버깅할 수 있습니다.

## CloudWatch Logs for EC2

기본적으로 EC2에는 CloudWatch Logs를 발생시킬 수 있는 어떤 장치도 없기 때문에, 만약 CloudWatch Logs의 상호 연결이 필요하다면 `CloudWatch agent`를 고려해야 하빈다. EC2에 Agent를 다운로드 받아 IAM 권한을 올바르게 부여하면 Logs가 EC2 상에 작동합니다.

> On-Premise에서도 Agent를 받아 실행시킬 수 있습니다.

Agent를 통해 다양한 지표를 얻을 수 있습니다.

- CPU (active, guest, idel, system, user, steal)
- Disk metrics (free, used, total), Disk IO (writes, reads, bytes, iops)
- RAM (free, inactive, used, total, cached)
- Netstat (number of TCP and UDP connections, net packets, bytes)
- Processes (total, dead, bloqued, idel, running, sleep)
- Swap Space (free, used, used %)

# Amazon CloudWatch Alarms

CloudWatch Alarms는 Metrics로부터 나오는 알림을 트리거하는데 사용됩니다. 다양한 옵션에서 선택이 가능합니다. 또한 옵션(%, max, min 등)에 의해 작동하는 알람의 상태는 다음 3가지의 상태로 출력됩니다.

1. OK: 상태가 좋음
2. INSUFFICIENT_DATA: 데이터가 부족
3. ALARM: 경보 발생

Target으로는 대표적으로 EC2, Auto Scaling, SNS 등이 있습니다. 모두 경보가 발생했을 때 이를 사용자에게 알리기 위한 방법이나, 혹은 자동으로 스케일링을 처리할 수 있도록 만들어진 서비스가 목표 지점이 됩니다. 특히, *Free tier의 사용이 어느 정도 초과될 지경에 이를 경우 자동으로 메일이 오게 되는데요*, **이러한 시스템이 바로 CloudWatch Alarms와 Amazon SNS를 활용한 예시라고 볼 수 있겠습니다.**

## Composite Alarms

![image](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/04/27/Step3-1.png)

복합알람이라고 불리는 이 기능은, 하나의 알람이 아닌 다수의 알람을 AND, OR 등의 연산자로 묶어 경보를 일으키는 서비스입니다. 위의 사진에서와 같이 `No CPU Credits Left`, `WS1_CPUUtilization`의 알람을 하나로 묶어 또 다른 경보를 생성하는 사진입니다.

둘 중의 하나만 경보가 울리거나 혹은, 두 개 다 경보가 울릴 경우에만 Amazon SNS를 트리거하는 등 다양한 시나리오로 제작할 수 있습니다.

# Amazon CloudWatch EventBridge

EventBridge를 통해 이벤트 패턴에 의한, 또는 스케줄러와 같은 역할을 수행할 수 있습니다.

1. Cron jobs(예정된 스크립트): 이 기능을 통해 매일 12시마다 혹은, 한 시간마다 작업해야 하는 스케줄러 역할을 수행할 수 있습니다. EventBridge와 Lambda의 합작품으로 이루어질 수 있습니다.
2. Event Pattern(특정 이벤트 발생): 규칙을 정하여 특정 이벤트가 발생했을 때 무언가를 트리거시킬 수 있습니다. 예를 들어 루트 계정에 로그인 했을 때 이를 Amazon SNS와 연결시켜 이메일 알림이 전송되도록 설정할 수 있는 것입니다.

## Event Bus

AWS Services들은 `Default Event Bus`에 의해 이러한 Event의 트리거와 Source - Target 연결이 가능합니다. AWS Services 제외하고도 AWS SaaS Partners 에서도 이러한 Event를 실행시킬 수 있는데요, 이 때 사용되는 것이 바로 `Partner Event Bus` 입니다. Custom Apps와 연결된 Event Bus는 `Custom Event Bus`입니다.

각 상황별에 맞게 어떤 Event Bus를 사용하는지 알아두어야 합니다. 

리소스 기반 정책을 사용하여 다른 AWS 계정의 리소스를 가져올 수도 있습니다.

## Schema Registry

이벤트가 어떻게 생겼는지 알고 계신가요? 이벤트는 보통 JSON형식으로 만들어집니다. EventBridge는 각 이벤트 버스와 스키마를 분석할 수 있는 기능을 가지고 있스빈다. 데이터의 정형화 방식을 파악하며, 스키마에 대한 버저닝이 가능합니다.

## Resource-based Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "events:PutEvents",
            "Principal": { "AWS": "111122223333" },
            "Resource": "arn:aws:events:us-east-1:123456789012:event-bus/central-event-bus"
        }
    ]
}
```
위와 같이 정책을 지정하면 다른 AWS 계정으로부터 Lambda Function에 의해 호출된 `PutEvents`를 실행시킬 수 있습니다. 이는 EventBridge Bus에 의해 이벤트 트리거가 중앙화될 수 있음을 의미하빈다.

# Amazon CloudWatch Insights

Logs에 추가적인 Insights를 부여하여 각 옵션에 특화된 기능으로 로그를 분석할 수 있습니다. Container, Lambda, Contributor 및 Application의 Log를 분석하여 문제점 해결을 도와주는 역할을 수행합니다.

## 1. CloudWatch Container Insights
   - Amazon Elastic Container Service
   - Amazon Elastic Kubernetes Services
   - Kubernetes platforms on EC2
   - Fargate 등

컨테이너와 연결되는 서비스를 사용했을 때 지표와 로그를 수집 및 분석할 수 있습니다. EKS 이거나 k8s의 경우 EC2에서는 Container용 Agent를 설치해야 이 기능을 온전히 이용할 수 있습니다.

## 2. ClooudWatch Lambda Insights

Lambda로부터 발생하는 애플리케이션의 트러블슈팅 및 모니터링이 가능합니다. 서버리스 애플리케이션의 세부 모니터링이 필요한 경우 사용합니다.

## 3. CloudWatch Contributor Insights

Contributor 데이터를 표시하는 시계열 데이터를 생성하고, 로그를 분석하는 서비스입니다. 이를 통해 시스템 성능에 영향을 미치는 대상을 파악할 수 있습니다. **VPC 로그 및 DNS 로그를 통해 불량 호스트를 식별해낼 수 있고, 사용량이 가장 많은 사용자를 식별해낼 수도 있습니다.**

```bash
VPC Flow Logs -> CloudWatch Logs -> CloudWatch Contributor Insights 순으로 이동
```

이러한 데이터는 모두 CloudWatch Logs에 의존하며, 추가적으로 어떤 기여자(Contributor) 데이터를 수집&분석 해야 하는 경우 이 옵션을 사용해야합니다.

## 4. CloudWatch Application Insights

애플리케이션의 잠재적인 문제와 진행 중인 문제를 분리할 수 있도록 자동화된 대시보드를 제공합니다. 애플리케이션에 문제가 발생하는 경우 해당 옵션은 자동으로 대시보드를 생성하여 서비스의 문제점들을 보여줍니다.

**AWS의 다른 리소스나 서비스를 사용하는 애플리케이션에 문제가 발생하면 해당 옵션을 사용하여 확인해야합니다.**