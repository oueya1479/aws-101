![rds](https://github.com/oueya1479/aws-101/assets/147911523/c1ec6530-75fd-4a77-a19c-e6d74ac26ccd)

### 목차

- [RDS - Storage Auto Scaling](#rds---storage-auto-scaling)
  - [Read Replicas for read scalability](#read-replicas-for-read-scalability)
  - [Multi AZ](#multi-az)
  - [Single AZ 에서 Multi AZ으로](#single-az-에서-multi-az으로)
  - [RDS Proxy](#rds-proxy)
- [RDS Custom](#rds-custom)

# RDS - Storage Auto Scaling

RDS는 Amazon Relational Database의 약자입니다. EC2에서도 간단히 데이터베이스 서버를 만들어서 운영할 수도 있지만, RDS를 사용하면 여러가지 이점을 볼 수 있습니다.

- 간편한 관리 : 인프라를 프로비저닝하거나 소프트웨어를 유지 관리할 필요가 없습니다. 여기서 프로비저닝이란 인프라를 생성하고 설정하는 프로세스로서 '인프라를 프로비저닝할 필요가 없다'라는 의미는 직접 다양한 리소스에 대해 액세스를 관리할 필요가 없다는 것입니다.
- 고가용성 : 다중 AZ 배포가 가능합니다.
- 다양한 엔진 옵션 : 선택한 관계형 데이터베이스 엔진을 선택하여 배포할 수 있습니다. 

## Read Replicas for read scalability

읽기 전용 복제본이라고도 불리는 `Read Replicas for read scalability`는 데이터베이스의 읽기 성능과 내구성을 높일 수 있습니다. 읽기가 많아지는 상황을 고려하여 탄력적으로 **scale out**하여 읽기 중심의 데이터베이스 워크로드를 처리할 수 있습니다.

> **✏️ use case**
> 
> production application을 운영하고 있는 팀이 있다고 가정. reporting 팀이 나타나서 검증한다고 하면 트래픽이 늘어나므로 이 떄, 읽기 전용 복제본을 만들어 reporting 팀에게 전달. 기존에 있는 데이터베이스 트래픽은 유지.

> **네트워크 비용**
> 
> 동일한 리전일 때에는 비용발생하지 않으며 다른 리전일 경우 비용 발생

## Multi AZ

![image](https://d1.awsstatic.com/product-page-diagram_MAZ_HIW%402xa.245de181144d709479981ab02a5318165b7ed8a9.png)

다중 AZ는 최대 3개의 가용 영역에 데이터베이스를 배포하여, 고가용성과 내구성을 강화시킨 서비스입니다. 다중 AZ를 통하여 자동으로 프라이머리 데이터베이스 DB 인스턴스를 생성하고 동시에 다른 AZ의 인스턴스에 데이터를 복제합니다.
읽기 전용 복제본(Read Replicas for read scalability)와 비슷하게 보일 수 있지만 용도가 조금 다릅니다.

다중 AZ 배포는 `고가용성`이 주 목적입니다. 하나의 데이터베이스 서버가 오류나도, 다른 곳에서 복구가 가능하다는 특징을 가지고 있지만, 읽기 전용 복저본의 경우 `확장성`이 주요 목적입니다.

> Primary writer --- Standby instance
>
> 네트워크가 손실되거나 장애가 발생할 때 Standby instance가 Primary로 승격되는 것입니다. Standby는 scaling 목적이 아닌, 대기 목적입니다. 누구도 쓰거나 읽을 수 없으며 단순히 조치를 위한 instance입니다. 추가적으로, **읽기 전용 복제본을 Multi AZ로 설정할 수도 있습니다.**

## Single AZ 에서 Multi AZ으로

> [blog](https://repost.aws/ko/knowledge-center/rds-convert-single-az-multi-az)를 참조하세요.

단일 AZ 인스턴스를 다중 AZ로 변경하여도 인스턴스에 가동 중단이 발생하지 않습니다. 내부적으로 DB snapshot을 찍고, Standby 데이터베이스로 복원한 이후 이를 동기화시키는 과정이 소요됩니다. 

새 볼륨은 즉시 사용 가능하지만 일부 성능문제가 발생할 수 있어 DB 인스턴스가 백그라운드에서 데이터를 계속 로드하는 과정이 필요합니다.

## RDS Proxy

프록시는 네트워크 용어로 사용될 때 중계서버 혹은 프록시서버로 쓰입니다. 예를 들면, '무신사' 앱의 경우 각 쇼핑몰에 대해 프록시 역할을 잘 수행한다고 볼 수 있습니다.

```bash
                      ----- A 쇼핑몰
사용자(우리) ----- 무신사 ----- B 쇼핑몰
                      ----- C 쇼핑몰
```

우리는 각 쇼핑몰에 접근할 필요 없이 프록시 앱을 통하여 모든 쇼핑몰에 접근할 수 있으며, 구매 혹은 배송이 이루어질 경우 각 쇼핑몰에서는 '무신사'앱에서 결제가 이루어졌다고 출력될 것입니다.

이와 마찬가지로, 우리는 어떤 서버에 접근할 때 프록시 서버를 사용하게 되면, 목적지 서버에는 프록시 서버의 기록으로 인식하게 됩니다. 이러한 시스템의 장점은 먼저 **프록시에 캐싱을 저장**할 수 있어, 목적지 서버까지 도달하지 않아도 캐싱된 데이터로 빠르게 접근할 수 있습니다. 또한, 프록시 서버에 보안을 적용하여 **원하지 않는 목적지 서버의 접근을 차단**할 수도 있습니다.

완전 관리형 RDS Database 도 프록시를 사용할 수 있습니다. 이는 애플리케이션이 데이터베이스 연결을 풀링하고 공유하도록 허용하여 확장 기능을 향상시킬 수 있습니다.

> 애플리케이션 연결을 유지하면서 예비 DB 인스턴스에 자동으로 연결하여 장애에 대한 복원력을 높일 수 있으며, IAM 혹은 Secrets Manager에 자격 증명을 안전하게 저장합니다.

RDS Proxy를 사용하면 데이터베이스 인스턴스에 일일이 접근하게 할 필요 없이 Proxy만 사용하면 됩니다.

1. 인스턴스에 연결이 많은 경우 CPU RAM등 부담을 줄여 효율성 향상
2. 개방된 연결과 시간초과를 최소화

완전한 서버리스로 오토 스케일링이 가능해 용량을 관리할 필요가 없으며, 장애조치가 발생하면 대기 인스턴스로 실행되며 프록시 덕분에 장애 조치 시간을 66%까지 줄일 수 있습니다. 이는 장애 조치를 각자 처리하는 것이 아닌 프록시에 연결하는 것으로 해결이 되는 것입니다.

**Other**

- MySQL, PostgreSQL, MariaDB 적용 및 Aurora 적용
- 코드 변경없이 연결만 하면 됨
- 퍼블릭 액세스가 불가능. VPC 내에서만 가능
- Lambda 함수를 사용하면 여러 개가 instance에 접근할 수 있기 때문에 proxy가 이를 대처할 수 있음

---

# RDS Custom

RDS Custom은 데이터베이스 관리 작업 및 운영을 자동화합니다. 데이터베이스 관리자가 데이터베이스 환경 및 운영 체제에 액세스하고 사용자를 지정할 수 있습니다.

```bash
1. Managed Oracle
2. Microsoft SQL
# 위의 운영체제에서만 사용 가능
```

보통 EC2로도 데이터베이스를 사용할 수 있지만, AWS의 데이터베이스 완전 관리형이 필요할 경우 Amazon RDS를 사용하는 것이 효율적입니다. 여기에서 종속 애플리케이션 및 운영체제에 대한 관리 권한까지 필요할 경우에는 `Amazon RDS Custom`이 나은 선택입니다.