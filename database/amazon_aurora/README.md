![aurora](https://github.com/oueya1479/aws-101/assets/147911523/1f7bc9c2-eb7e-4f39-a5df-53926c79c495)

<small>Author: heony</small>

### 목차

- [Amazon Aurora](#amazon-aurora)
  - [High Availability and Read Scaling](#high-availability-and-read-scaling)
  - [Aurora DB Cluster](#aurora-db-cluster)
  - [Features of Aurora](#features-of-aurora)
- [Backups](#backups)
  - [RDS Backup](#rds-backup)
  - [Aurora Backups](#aurora-backups)
  - [Restore options](#restore-options)
  - [Aurora Database Cloning](#aurora-database-cloning)
  - [RDS Aurora Security](#rds-aurora-security)

# Amazon Aurora

AWS 고유의 기술로 MySQL 및 PostgreSQL 호환성과 고성능 및 고가용성을 제공하는 서비스입니다. MySQL과 PostgreSQL 두 RDS만 지원합니다. 실제로는 RDS에 비해 더 비싸지만, 상용 데이터베이스 비교시 1/10의 비용을 제공합니다.

- 10GB으로 시작하여 자동으로 128TB까지 스토리지를 자동으로 확장합니다.
- 읽기 전용 복제본을 기존 5개(RDS)에 비해 15개까지 확장할 수 있습니다.
- 장애 조치가 즉각적으로 이루어집니다.
- 다중 AZ의 속도도 빨라집니다.
- 클라우드 네이티브이므로 가용성 또한 좋습니다.
- 스케일링이 효율적이라 가격이 조금 더 비싸더라도, 그보다 더 높은 효율성을 제공합니다.

## High Availability and Read Scaling

기본 데이터베이스 인스턴스에 데이터가 기록되면 Aurora에서 가용 영역의 데이터를 6개의 스토리지 노드에 동기적으로 복제합니다. 이는 총 3개의 가용 영역에 걸쳐있습니다.

- 쓰기에는 4개만 필요
- 읽기에는 3개만 필요

Aurora 또한 RDS의 Multi-AZ와 같이 마스터가 작동하지 않으면 30초 안에 장애조치가 되며, 총 15개의 복제본을 둘 수 있습니다.

> 기본적으로 마스터 인스턴스가 하나인 것은 변함없습니다.

## Aurora DB Cluster

![image](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/images/AuroraArch001.png)

Aurora DB cluster는 하나 이상의 데이터베이스 인스턴스와 이 인스턴스의 데이터를 관리하는 `클러스터 볼륨`으로 구성됩니다. 이는 다중 가용 영역을 아우르는 가상 데이터베이스 스토리지 볼륨으로 **각 가용 영역에는 클러스터 데이터의 사본이 있습니다.**

- 기본 DB 인스턴스: 읽기 및 쓰기 작업 지원, 클러스터 볼륨의 모든 데이터 수정을 실행
- Aurora 복제본: 읽기 작업만 지원, 최대 15개 복제본 지원, 기본 DB 인스턴스를 사용할 수 없는 경우 장애 조치 즉각적

## Features of Aurora
1. **Auto scaling**: 애플리케이션의 요구 사항에 맞게 I/O를 자동으로 조정합니다. 10GB에서 128TB까지 자동으로 확장될 수 있습니다.
2. **Custom Endpoints**: 사용자 지정 엔드포인트를 통해 여러 데이터베이스 인스턴스에서 워크로드를 분산하며 로드밸런싱을 할 수 있습니다. **여기서 Endpoint를 통하여 메모리 용량이 더 크거나 작은 인스턴스 유형을 선택하여 복제본을 프로비저닝할 수 있습니다.**
3. **Serverless**: 실제 사용량에 따라 오토 스케일링을 제공하기 때문에 용량 계획을 세울 필요가 없습니다(비용 효율적).
4. **Multi-Master**: 모든 Aurora의 복제본을 Writer로 만들 수 있습니다(마스터가 여러개).
5. **Global Aurora**: 하나의 Aurora 데이터베이스가 여러 AWS 리전에 걸쳐 있는 `Aurora Global Database`를 사용하면 더 빠른 로컬 읽기 및 빠른 재해 복구가 가능합니다. 글로벌 데이터베이스의 데이터를 리전 간 복제하는 데 평균 1초 미만이 소요됩니다.
6. **Aurora Machine Learning**: SageMaker, Comprehend 머신 러닝 서비스가 Aurora와 통합되어 있습니다. 사기 탐지, 광고 타겟팅, 감정 분석 등 Aurora에서 사용할 수 있고 사용자가 머신러닝을 이해하고 있지 않아도 됩니다.

---

# Backups

## RDS Backup

- 자동 데이터베이스 백업

5분마다 트랜잭션 로그가 백업됩니다. 이는 언제라도 5분 전으로 돌아갈 수 있는 것을 의미합니다. 백업 보존 기간은 1 ~ 35일입니다.

- 수동 데이터베이스 스냅샷

원하는 기간 동안 유지할 수 있습니다. 저동 백업의 경우 만료 기간이 존재하지만, 수동 기간의 경우 원하는 기간을 설정할 수 있습니다.

> 스냅샷을 찍어서 보관하는 것은, 가끔 사용하는 데이터베이스의 요금을 줄일 수 있는 방법을 제공합니다.

## Aurora Backups

- 자동 데이터베이스 백업

RDS와 마찬가지로 1 ~ 35일 동안 데이터를 저장할 수 있으며, 이는 비활성화 할 수 없습니다. 시접 복구 기능이 있어 어느 시점으로든 복구가 가능합니다.

- 수동 데이터베이스 스냅샷

원하는 기간 동안 유지할 수 있습니다. 저동 백업의 경우 만료 기간이 존재하지만, 수동 기간의 경우 원하는 기간을 설정할 수 있습니다.

## Restore options

1. 새로운 데이터베이스로 복원할 수 있습니다.
2. S3에서 MySQL를 복원할 수 있으며, RDS에는 S3 백업파일을 복원하는 옵션이 있습니다.
3. S3로부터 Aurora를 복원할 수 있으며, Onpremise 데이터베이스에서 Aurora cluster으로. Percona XtraBackup의 사용이 필수적입니다.

## Aurora Database Cloning

프로덕션 데이터베이스가 있는 상태에서 테스트용으로 데이터베이스를 구축하고 싶다면 `Aurora를 Cloning`하면 됩니다. 스냅샷을 찍고 복원하는것보다 훨씬 빠르며 이는 `copy-on-write protocol`을 사용하기 때문입니다. 비용 효율적이고 프로덕션 데이터베이스에 영향을 주지않고 복제하는데 매우 유용합니다.

> **Copy On Write(COW)란?**
>
> 리소스가 복제되었을 때 수정되지 않은 경우에는 새로운 리소스를 복제하는 것이 아닌, 단순히 기존의 지점을 가리키는 것을 의미합니다. 즉, X = 'abc'라고 했을 때, Y = X 수식으로 Y가 X의 값으로 대입되는 경우 Y 또한 동일한 메모리 인덱스인 'abc'를 가리키는 것을 의미합니다.

## RDS Aurora Security

저장된 데이터베이스에 암호화를 적용할 수 있습니다. 이 때 KMS를 사용할 수 있으며, 모든 복제본에 암호화가 적용됩니다.

> 암호화가 되어있지 않은 기존 데이터베이스의 경우 읽기 전용 복제본도 암호화를 진행할 수 없습니다.

- 저장데이터 암호화는 기본적으로 전송 중 데이터 암호화기능을 가지고 있기 때문에 TLS 루트 인증서만 있으면 암호화가 됩니다.
- IAM 역할을 사용하여 인스턴스에 접근할 수 있습니다.
- 보안 그룹을 사용하여 데이터베이스의 네트워크를 차단할 수 있습니다.
- 관리형 서비스가 있기 때문에 SSH 액세스가 없습니다.
- 감사 로그가 있어서 시간에 따른 쿼리 로그를 확인할 수 있습니다(활성화시켜야 함).