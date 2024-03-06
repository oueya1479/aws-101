![elasticache](https://github.com/oueya1479/aws-101/assets/147911523/4c41d907-6a37-463f-ae63-1316d28f7b9c)

<small>Author: heony</small>

### 목차
- [ElastiChche](#elastichche)
  - [Architecture](#architecture)
  - [User Session Store](#user-session-store)
  - [Redis vs Memcached](#redis-vs-memcached)
    - [Redis use case](#redis-use-case)
  - [ElasticCache - security](#elasticcache---security)

# ElastiChche

RDS와는 달리 인메모리 데이터베이스를 지원하는 ElastiCache는 Redis와 Memcached와 호환됩니다. 몇가지 특징으로는 다음과 같이 정리될 수 있습니다.

- Redis와 같이 `인메모리 데이터베이스`를 지원합니다(데이터베이스의 로드를 줄여준다).
- 매번 데이터베이스를 쿼리하지 않아도 캐시를 사용하여 쿼리의 결과를 검색할 수 있습니다.
- 애플리케이션을 상태 비저장형으로 저장시킬 수 있습니다(패치, 최적화, 설정, 백업 등).
- 캐시를 Query 할 수 있도록 애플리케이션 코드를 약간 변경해야한다.

## Architecture

![image](https://res.cloudinary.com/practicaldev/image/fetch/s--IuU6r5CW--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9k7e7avoxfjlcd6mkmfo.jpg)

> [해당 블로그](https://dev.to/kelvinskell/an-overview-of-elasticache-caching-strategies-3h3d)를 참조했습니다.

Lazy Laoding(지연 로딩)은 필요한 경우에만 데이터를 캐시에 로드하는 캐싱 전략입니다.

1. Cache HIT: elasticache에서 데이터를 가져옴
2. Cache MISS: 캐시에 데이터가 없는 경우
3. Database reading(Read from DB): 데이터를 가져옴
4. Writing into Cache(Write to cache): 가져온 데이터를 캐시에 저장

2번의 캐시미스의 경우 요청한 데이터를 사용할 수 없거나 만료되었을 때 발생합니다. 또는 `ElastiCache`안에 데이터가 존재하지 않는 경우에도 **캐시 미스**가 발생하게 됩니다.

## User Session Store
- Write Session: 사용자 세션을 저장하는 것, 애플리케이션을 상태 비저장으로 하기 위하여.
- Retrieve Session: ElastiCache로부터 데이터를 가져옴

## Redis vs Memcached

ElastiCache는 `Redis`와 `Memcached` 두 가지 인 메모리 데이터 스토어 엔진을 제공합니다. 각각은 장단점이 있으며 사용 사례 및 요구 사항에 따라 선택해야 합니다.

**Redis**

- 다중 AZ, 읽기 복제본이 있음
- AOF 지속성을 이용한 데이터 내구성
- 백업 및 복원 기능

**Memcached**

- 데이터 샤딩, 분산 캐싱을 위한 주요 목적에 초점
- 고가용성이 없고 복제가 일어나지 않으며 백업 및 복원도 없음
- 여러 인스턴스가 샤딩을 통해 작용
- 분산되어 있는 그저 순수한 캐시

### Redis use case

Redis Sorted Sets는 실시간 리더보드를 만들 수 있도록 도와줍니다.

## ElasticCache - security

AWS Identity and Access Management(IAM)은 API 수준의 보안만 제공합니다. 이는 AWS 리소스에 대한 액세스 및 권한을 관리하는 것입니다. ElastiCache는 자체 보안으로 `Redis AUTH`를 제공합니다.

Redis AUTH를 사용하여 비밀번호와 토큰을 설정할 수 있습니다. 데이터 전송 시에 SSL 암호화를 지원하여 데이터의 안전한 전송을 보장합니다.

Memcached는 SASL 기반의 승인을 지원합니다.