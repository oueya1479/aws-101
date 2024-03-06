![transfer](https://github.com/oueya1479/aws-101/assets/147911523/7d79ad48-5a9a-45a2-a6f2-71c6e20fdc1c)

<small>Author: heony</small>

### 목차

- [AWS Transfer Family](#aws-transfer-family)

# AWS Transfer Family

AWS Transfer Family는 스토리지 서비스 간에 파일을 전송하고 스토리지 서비스에서 파일을 전송할 수 있는 **보안 전송 서비스**입니다. 

> S3 또는 EFS를 통해 안팎으로 데이터를 전송해야 하지만, S3 APIs는 사용하고 싶지 않을 때 사용되기도 합니다.

AWS Transfer Family는 다음 프로토콜을 통하여 데이터 전송을 지원합니다.

- AWS Transfer for FTP(File Transfer Protocol)
- AWS Transfer for FTPS(File Transfer Protocol Secure)
- AWS Transfer for SFTP(Secure Shell File Transfer Protocol)