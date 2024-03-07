![cloudfront](https://github.com/oueya1479/aws-101/assets/147911523/208a7fec-775a-4b60-a665-c4729935addd)

<small>Author: heony</small>

### 목차

- [CloudFront](#cloudfront)
  - [CloudFront - Origins](#cloudfront---origins)
  - [CloudFront vs S3 Cross Region Replication](#cloudfront-vs-s3-cross-region-replication)
  - [ALB or EC2 as an origin](#alb-or-ec2-as-an-origin)
    - [CloudFront Get Restriction](#cloudfront-get-restriction)
  - [Pricing](#pricing)
  - [Cache Invalidations](#cache-invalidations)

# CloudFront

![image](https://d1.awsstatic.com/products/cloudfront/product-page-diagram_CloudFront_HIW.475cd71e52ebbb9acbe55fd1b242c75ebb619a2e.png)

Content Delivery Network(CDN)은 웹사이트의 컨텐츠를 서로 다른 엣지 로케이션에 미리 캐싱하여 읽기 성능을 높일 수 있습니다. S3를 정적 웹 호스팅하여 단순히 읽기 기능이 가능한 데이터가 있을 때, 이를 CloudFront로 감싸면 **가장 가까운 지역의 엣지 로케이션에 캐싱된 데이터**를 가져오게 되어 더 빠릅니다.

전세계의 216개 포인트가 엣지 로케이션으로 구성되어 있으며 DDos 방어 등의 보안이 설정되어 있습니다.

> 가령 호주에 S3를 제작했다고 하더라도 전세계에 퍼져있는 엣지 로케이션을 통해 가장 가까이에 있는 엣지포인트로부터 트래픽을 전달받을 수 있습니다.

## CloudFront - Origins

**S3 bucket**
  - CloudFront를 통해 파일을 분산하고 캐싱할 수 있습니다.
  - Origin Access Control(OAC)를 통해 CloudFront만 접근가능할 수 있도록 설정 가능합니다.

**Custom Origin (HTTP)**
  - Application Load Balancer
  - EC2 Instance
  - S3 website
  - Any HTTP 요청 등

```
기존 방식 : Client ===== S3
CDN : Client ===== [CloudFront Edge Location] ===== S3
```

## CloudFront vs S3 Cross Region Replication

**CloudFront**
- Global Edge network
- 엣지 로케이션에 캐싱합니다.
- 전세계를 대상으로 한 정적 컨텐츠를 사용할 때 좋습니다.

**S3 Cross Region Replication**
- 엣지 로케이션에 캐싱되지 않고 단순히 읽기 전용을 늘립니다.
- 일부 리전을 대상으로 동적 컨텐츠를 낮은 지연시간으로 지원하려고 할 때 사용합니다.
- 다른 리전으로의 버킷을 복제하여 고가용성, 안정성을 추구합니다.

## ALB or EC2 as an origin

**EC2 as an origin**

- Edge Location을 통해 EC2 인스턴스에 접근. 엣지 로케이션에 모든 공용 IP가 EC2에 접근할 수 있도록 **보안 그룹**을 설정해야합니다.

```
User ===== Edge Location ===== EC2 (public)
```

**ALB**

- EC2 인스턴스를 프라이빗으로 설정해도 되며 대신에 ALB에 연결되어 있는 상태여야 합니다. Edge Location을 통해 **ALB 보안 그룹**에 연결하면 사용할 수 있습니다.

```
User ===== Edge Location ===== ALB ===== EC2 (private or public)
```

### CloudFront Get Restriction

> [문서 정보](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/georestrictions.html)

**지역에 따른 배포 제한을 설정할 수 있습니다.** 접근 가능한 국가 및 접근 불가능한 국가를 설정할 수 있으며 사용자의 IP를 확인하여 감지할 수 있습니다.

## Pricing

엣지 로케이션은 전 세계에 고루 분산되어있기 때문에 지역마다 요금표가 다릅니다. 대륙이나 지리 위치에 따라 가격 정책이 다르기 때문입니다.

더 많은 데이터가 전송될수록 더 낮은 요금으로 책정됩니다.

> 비용 절감을 위해 전 세계 엣지 로케이션 수를 줄이는 방법이 있습니다.

1. Price Class All: 전체 지역을 대상
2. Price Class 200: 대부분의 지역이지만 비싼 지역은 제외
3. Price Class 100: 대부분의 저렴한 지역으로 제공

## Cache Invalidations

최대한 빨리 새 컨텐츠를 받고 싶을때 전체 또는 일부의 캐시를 강제로 새로고침하여 캐시에 있는 모든 TTL을 제거할 수 있으며, 이는 CloudFront Cache Invalidation을 실행해야합니다.

> TTL은 Time To Live으로 도메인을 설정할 때 시간을 지정할 수 있습니다. 페이지가 한번 로드된 이후 추가적으로 로드하게 되면 더 빨리 데이터를 가져오게 되는데 이는 TTL 시간동안 컨텐츠가 캐싱되어 있기 때문입니다.