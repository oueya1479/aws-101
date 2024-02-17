![auto](https://github.com/oueya1479/aws-101/assets/147911523/18c399d0-c011-4791-a726-8c7a347cc663)

<small>Author: heony</small>

### 목차

- [Amazon EC2 Auto Scaling](#amazon-ec2-auto-scaling)
  - [스케일링 정책](#스케일링-정책)
    - [Target Tracking Scaling](#target-tracking-scaling)
    - [Simple / Step Scaling](#simple--step-scaling)
    - [Scheduled Action](#scheduled-action)
    - [Predictive Scaling](#predictive-scaling)
    - [지표 (Metric)](#지표-metric)
  - [Scaling Cooldowns란?](#scaling-cooldowns란)
    - [번외) 트래픽 초과해보기](#번외-트래픽-초과해보기)

# Amazon EC2 Auto Scaling

Auto Scaling은 로드밸런서로부터 트래픽을 확인하여 애플리케이션 및 웹사이트의 서버를 자동으로 조정하는 서비스입니다. 이는 빠른 속도로 감지하고, 서버를 자동으로 실행시키거나 종료하는데요, 이러한 Auto Scaling은 다음과 같은 목표를 가지고 작동합니다.

- 트래픽 증가에 의한 **스케일 아웃 (EC2 인스턴스 추가)**
- 트래픽 감소에 의한 **스케일 인 (EC2 인스턴스 제거)**
- 작동 중인 EC2 인스턴스를 최소화 및 최대화
- 자동으로 **새로운 인스턴스를 로드 밸런서에 등록**
- 작동되지 않는 EC2 인스턴스를 제거한 후 재생성

이러한 Auto Scaling이 일어날 때에 ELB는 새로운 인스턴스를 감지해야 합니다. 만약, 어떤 그룹도 지정되어있지 않다면 ELB는 새로운 인스턴스가 추가되어도 이것이 Auto Scaling 전략에 의한 인스턴스인지, 또는 사용자가 직접 제작한 인스턴스인지 알지 못합니다.

![image](https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/images/as-basic-diagram.png)

따라서, Auto Scaling이 일어나는 그룹을 지정해주어야 하는데요, <u>**이것이 바로 Auto Scaling Group(ASG) 입니다.**</u> ASG를 지정하면 해당 그룹 내에서 최소 및 최대 인스턴스를 지정하고 트래픽에 따른 인스턴스를 개편할 수 있습니다.

## 스케일링 정책

### Target Tracking Scaling

대상 추적 스케일링 정책은 지표(Metric) 값을 기반으로 ASG의 용량을 자동으로 스케일링합니다. 따라서 이 때에는 `CloudWatch Alarams` 서비스가 필수적으로 사용되어야 합니다. 예를 들어 2개의 인스턴스에서 애플리케이션이 실행되고 있고, 트래픽에 변경이 있을 경우 CPU 사용량을 유지하기 위해 스케일 인 / 아웃 전략을 사용할 수 있는 것입니다.

> Example : 평균 ASG CPU를 40%으로 유지하고 싶어요.

### Simple / Step Scaling

단계별 및 단순 스케일링은 Alarms을 기반으로 ASG 용량을 사전 정의된 양으로 조정됩니다. Alarm 임계값이 위반될 때 스케일 인 / 아웃 전략을 사용할 수 있습니다.

> Example : CPU > 70%가 될 때 트리거(인스턴스 2개 추가) 하고 싶어요.

> Example : CPU < 30%가 될 때 트리거(인스턴스 제거) 하고 싶어요.

### Scheduled Action

Scheduled Action은 미리 알 수 있는 정보에 의해 사전에 인스턴스 예약을 신청할 수 있습니다. 예를 들어, 오전 10시에 수강신청이 있어서 미리 인스턴스를 여러 대 준비해놓아야 할 경우 이 서비스를 사용할 수 있습니다.

이는 트래픽이 감지되어 자동으로 늘리거나 줄이는 옵션이 아닌, 사전에 어떤 이벤트가 일어날 것임을 알고 미리 스케줄링을 예약하는 것입니다. 마치 예약 메시지와 같은 의미입니다.

> Example : 오전 10시에 3대 증가시킬래요.

### Predictive Scaling

Predictive Scaling 전략은 머신러닝에 의해 트래픽이 많은 시간에 자동으로 사전에 늘리거나 줄이는 전략입니다. `Scheduled Action`과는 다르게 미리 지정해놓지 않아도 머신러닝이 자동으로 판단하여 스케일링을 마칩니다.

### 지표 (Metric)

ASG가 자동으로 스케일링하는 것에는 어떤 지표(Metric)에 의존할 수 밖에 없습니다. 따라서 AWS에서는 여러 지표를 제공하는데 대표적인 것들을 소개합니다.

- CPUUtilization: CPU utilization 평균 사용를
- RequestCountPerTarget: 각 EC2 인스턴스당 요청 수
- Average Network In / Out: 네트워크 인터페이스에서 단일 인스턴스가 받거나 보낸 평균 바이트 수
- Any Custom Metric: 커스텀 지표

---

## Scaling Cooldowns란?

ASG는 인스턴스를 추가하거나 제거할 때 Simple Scaling 정책에 의해 시작된 추가 스케일링 활동이 시작되기 전에 Cooldown 기간이 끝날때까지 기다립니다. 이 기간의 목적은 **ASG가 안정되도록 하고 이전의 스케일링 활동의 효과가 가시화되기 전에 추가 인스턴스가 시작되거나 해지되는 것을 방지**하기 위함입니다.

---

### 번외) 트래픽 초과해보기
```bash
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```