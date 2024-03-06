![datasync](https://github.com/oueya1479/aws-101/assets/147911523/01df40aa-4fc4-49d0-9422-c80afdf3b4c1)

<small>Author: heony</small>

### 목차

- [AWS Datasync](#aws-datasync)
  - [AWS DataSync NFS / SMB to AWS](#aws-datasync-nfs--smb-to-aws)
  - [Transfer between AWS storage services](#transfer-between-aws-storage-services)

# AWS Datasync

AWS DataSync는 AWS로의 데이터 마이그레이션을 간소화 & 가속화하고, 데이터를 동기화하며 대용량의 데이터를 한 곳에서 다른 곳으로 옮길 수 있습니다.

> 네트워크를 통해 데이터를 AWS 스토리지 서비스로 빠르게 이동시킬 수 있으며, 데이터를 내부 또는 외부로 이동시킬 수 있습니다.

- `On-premises / other cloud` -> AWS (에이전트가 필요)
- `AWS` -> AWS (에이전트가 필요하지 않음)

**동기화 가능한 서비스 목록**
- Amazon S3
- Amazon EFS
- Amazon FSx

매 주, 매 시간 등 설정할 수 있으며 파일권한과 메타데이터 저장 기능이 있음
에이전트 하나의 태스크가 초당 10Gb까지 사용 가능하며, 대역폭에 제한을 걸 수도 있다.

## AWS DataSync NFS / SMB to AWS

`NFS(Network File System)`은 네트워크 파일 시스템으로, Linux 기반 네트워크 아키텍처의 용도입니다. 파일 및 디렉터리를 공유하며, 클라이언트가 서버와 통신 가능합니다.

`SMB(Server Message Block)`은 서버 메시지 블록으로, Windows 기반 아키텍처의 용도입니다. 파일 및 인쇄 서비스, 스토리지 디바이스, 가상 머신 스토리지를 비롯하여 광범위한 네트워크 리소스를 공유하며, 서버 및 다른 클라이언트와 통신할 수 있습니다.

네트워크 용량이 따라주지 못할 때 Snowcone을 사용하며, on-premise에서 snowcone을 활용하여 데이터를 동기화할 수 있습니다.

## Transfer between AWS storage services

서로 다른 AWS 스토리지간 메타데이터 및 데이터를 동기화할 수 있으며, 지속적이지 않고 일정에 따라 움직입니다. **메타데이터와 파일 권한은 보존됩니다.**