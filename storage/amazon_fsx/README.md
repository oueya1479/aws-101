![fsx](https://github.com/oueya1479/aws-101/assets/147911523/59f25984-a71a-4294-95e9-96773bfff62f)

<small>Author: heony</small>

### 목차

- [Amazon FSx](#amazon-fsx)
  - [Amazon FSx for Windows File Server](#amazon-fsx-for-windows-file-server)
  - [Amazon FSx for Lustre](#amazon-fsx-for-lustre)
    - [Amazon FSx deploy option](#amazon-fsx-deploy-option)
  - [Amazon FSx for NetApp ONTAP](#amazon-fsx-for-netapp-ontap)
  - [Amazon FSx for OpenZFS](#amazon-fsx-for-openzfs)

# Amazon FSx

FSx는 File System의 줄임말입니다. 완전 관리형 서비스로 타사 고성능 파일 시스템을 실행시킵니다. 가령 RDS에서 MySQL나 Postgre를 실행하는 것과 마찬가지입니다. 이 문서에서 소개할 FSx 서비스는 다음과 같습니다.

- Amazon FSx for Windows File Server
- Amazon FSx for Lustre
- Amazon FSx for NetApp ONTAP
- Amazon FSx for OpenZFS

## Amazon FSx for Windows File Server

완전 관리형 Windows 파일 시스템으로 `SMB`, `NTFS` 프로토콜을 지원합니다. 사용자 보안을 추가할 수 있고 ACL로 할당량을 추가할 수도 있습니다.

- 겉보기에는 Windows에서만 사용 가능할 것 같지만 **Linux EC2에서도 사용 가능합니다.**
- Microsoft 분산 파일 시스템인 DFS 기능을 이용하여 파일 시스템을 그룹화할 수 있습니다.
- 성능으로는 초당 수십 GB 및 수백만 IOPS / 수백 PB의 데이터까지 확장될 수 있습니다.
- 스토리지 옵션으로는 `SSD`, `HDD`를 지원합니다.
- 프라이빗 연결로 온프라미스 인프라에서 사용가능합니다.
- 재해복구 목적으로 S3에 매일 백업됩니다.

## Amazon FSx for Lustre

> Lustre는 Linux와 cluster의 합성어입니다.

Lustre는 원래 분산 파일 시스템으로 대형 연산에 쓰였습니다. 동영상 처리나 금융 모델링 등 확장성이 상당히 높은곳에 쓰일 수 있고, 짧은 지연시간을 감당할 수 있습니다. 

보통 일반적인 파일 공유 및 Windows 환경에서의 파일 서버는 `Amazon FSx for Windows File Server`을 사용하여 대용량 데이터 처리나 고성능 컴퓨팅환경이 필요하다면 `Amazon FSx for Lustre`을 고려할 수 있습니다.

- 스토리지 옵션으로 `SSD`, `HDD`를 지원합니다.
- S3와 무결절성 통합이 가능하며 FSx로 S3를 파일 시스템처럼 읽어들일 수 있습니다. 또한 FSx의 연산 출력값을 다시 S3에 쓸 수 있습니다.
- VPN과 직접 연결을 통해 온프레미스 서버에서 사용할 수 있습니다.

### Amazon FSx deploy option

**Scratch File System**
  - 일시적인 데이터 저장이나 임시 작업에 사용됩니다.
  - 데이터가 복제되지 않습니다.
  - 최적화로 초과 버스트를 사용가능하며 속도를 빠르게 만들 수 있습니다.
  - 복제가 없어 비용을 최적화할 수 있습니다.

**Persistent File System**
  - 데이터의 장기 보존이 필요한 경우에 사용됩니다.
  - 동일한 가용 영역에 데이터가 복제됩니다.
  - 서버가 오작동했을 때 쉽게 파일이 대체될 수 있습니다.
  - 민감한 데이터의 장기 처리 및 스토리지를 예로 들 수 있습니다.

## Amazon FSx for NetApp ONTAP

`NFS`, `SMB`, `iSCSI` 프로토콜과 호환이 가능합니다. 이를 활용하여 온프레미스의 `ONTAP`이나 `NAS`에서 실행 중인 워크로드를 AWS로 옮길 수 있습니다.

다양한 운영체제에서 사용 가능합니다.

- Linux
- Windows
- MacOS
- VMware Cloud on AWS 등

스토리지는 자동으로 축소될 수 있으며, 복제와 스냅샷 기능또한 제공합니다. 데이터 압축이나 중복제거도 가능합니다. 자정 시간 복제 기능은 새 워크로드를 테스트할 때 유용합니다.

## Amazon FSx for OpenZFS

여러 버전에서의 NFS 프로토콜과 호환이 가능합니다. 주로 ZFS에서 사용하는 것을 내부적으로 AWS에 옮길 때 사용합니다.

- Linux
- Windows
- MacOS
- VMware Cloud on AWS 등

스냅샷, 압축을 지원하고 비용이 적지만, 데이터 중복제거 기능은 없습니다. ONTAP처럼 자정 시간 복제 기능이 탑재되어 있습니다.