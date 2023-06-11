---
title: Redis와 캐싱에 대해
subTitle: Redis로 NestJS에서 Database Cahe 적용하기
tech: Redis
category: Redis 사용하기
tags:
	- Redis
	- NestJS
	- Cache
date: 2022-01-02
---

프로젝트에서 레디스는 session, rateLimit 스토리지로 이미 사용하고 있지만
최근 게시글 랭킹 시스템을 개선하기 위해서 조금 더 깊이 레디스를 알 필요가 있다고 생각했고
Redis를 활용한 데이터베이스 캐싱을 적용한 과정을 공유하고자 합니다



# Redis 란
---

`In-Memory Data Structure Store`

데이터베이스, 캐시, 메시지 브로커로 사용되는 인 메모리 데이터 구조 저장소입니다



# 지원하는 데이터 자료구조(Collection)
---

Redis는 Mem cached와 다르게 여러 가지 데이터 자료구조를 지원합니다.
간단하게 주로 사용되는 것들을 나열해 보겠습니다

## Strings

일반적인 Key-value입니다.
문자열뿐만 아니라 모든 종류의 데이터를 저장할 수 있습니다
Key에 할당 가능한 데이터의 크기는 512mb입니다

## List

String의 집합으로 새로운 요소를 `앞, 뒤`로 삽입하는것이 매우 빠릅니다

## Set

![image](https://user-images.githubusercontent.com/55491354/193414646-71399e51-d46f-4e11-82e2-d25155114f2d.png)
List와 다르게 중복된 데이터를 넣을 수 없는 string 집합입니다
SET 자료구조 간 `집합 연산`을 매우 빠른시간내에 처리 할 수 있습니다

## Sorted-set

![image](https://user-images.githubusercontent.com/55491354/193414674-3d017366-1c56-45b3-a7fc-57c81359fc56.png)
Set 자료구조의 특성은 그대로 가지면서 추가로 `score`를 저장해서
저장된 값들의 순서를 관리합니다
Score가 같다면 사전 순으로 정렬됩니다

## Hashes

![image](https://user-images.githubusercontent.com/55491354/193414703-947ae300-7c5f-4e10-b40c-a2a9564c51f8.png)
데이터베이스와 유사하게 하나의 `key`에 여러 개의 서브키, 데이터를 저장할 수 있습니다

## Hyperloglog

해당 집합 내 원소의 개수를 추정하는 방법입니다
매우 큰 집합의 근사치 카운트를 구할 때 사용합니다(1% 미만 오차)

## Bitmap

0, 1 상태를 가지는 Key들을 효율적으로 관리할 수 있습니다

## Geospatial

![image](https://user-images.githubusercontent.com/55491354/193414751-e1a195c2-52ec-48d7-9532-9a4f893e0f23.png)
경도와 위도를 저장합니다

> 추가적으로 사용가능한 명령어는
>
> [https://redis.io/commands](https://redis.io/commands) 공식문서에서 확인할 수 있었습니다.



# Cache가 해결하는 문제
---

캐시는 고속 데이터 스토리지 계층으로
A 데이터를 캐시 스토리지에 저장한 뒤 이후에 A 데이터에 대한 요청이 있으면
데이터의 기본 스토리지에 엑세스 하는 것 보다 더 `빠르게` 요청을 처리할 수 있습니다
하지만 캐시로 저장된 데이터는 `실시간 업데이트가 불가능`하기 때문에
실시간 변경이 필요하지 않고
데이터를 새로 고침하는 주기가 일정한(랭킹, 연산 결과의 저장(팩토리얼)) 환경에서 주로 사용합니다



# Cache 적용 패턴
---

## Inline Cache

![image](https://user-images.githubusercontent.com/55491354/193414978-6f5484b4-fbcd-4c01-a560-f9b69eff171c.png)
데이터를 조회할 때 반드시 캐시에서 조회하고
일정 주기로 시작되는 배치 작업이 캐시 스토리지를 업데이트하는 방식입니다

### 고려사항

- 모든 데이터를 캐시 스토리지에 적재합니다
- 추가적인 배치 작업 설계가 필요합니다
- 배치작업이 원활하게 이루어지지 않을 경우 캐시 스토리지와 데이터베이스 간 정합성이 크게 깨집니다

## Cache Aside (Look aside Cache)

![image](https://user-images.githubusercontent.com/55491354/193415015-eefca746-b6b7-4970-84c9-cc779567d2b3.png)
일반적으로 가장 많이 사용되는 패턴입니다
데이터를 조회할 때 캐시에서 조회한 뒤 데이터가 있다면 바로 응답하고
없다면 데이터베이스에서 데이터를 조회한 뒤 데이터를 캐시 스토리지에 저장한 뒤 응답하는 방식입니다.

### 고려사항

- 최초접근 또는 캐시 데이터가 만료되었을 경우 데이터베이스에 접근합니다

## Write back

![image](https://user-images.githubusercontent.com/55491354/193415105-edcfc6e6-8834-4f76-a956-7f912ba6e7d1.png)
데이터를 조회, 수정할 때 모든 상호작용을 캐시 스토리지에서 진행합니다
일정 주기마다 캐시 스토리지에 있는 데이터를 데이터베이스에 저장합니다

### 추가사항

INSERT를 따로 N회 하는 것 보다
N개의 데이터를 한꺼번에 INSERT 하는 것이 퍼포먼스가 좋다는 검증 결과에 근거한 방식입니다

### 고려사항

- 캐시 스토리지가 다운되면 저장되지 않은 데이터는 모두 사라집니다
- 캐시 스토리지가 다운된 후 다시 서비스를 시작하기 위해서 추가 작업이 필요합니다



# 레디스도 동시성 문제가 발생할 수 있다
---

레디스는 단일 스레드이고 단일 명령을 실행할 때는 원자성을 보장하지만
여러 클라이언트가 동일한 데이터를 상대로 명령을 실행 할 경우
일반적인 RDBMS와 마찬가지로 **갱신손실**이 발생할 수 있습니다
트랜잭션을 사용해서 데이터의 고립성을 지켜야 합니다

> 레디스에서 잠금, 트랜잭션이 필요한 이유 [스택 오버 플로우 링크](https://stackoverflow.com/questions/30004364/redis-race-condition-and-single-threaded)

> 레디스 트랜잭션 공식 문서 [https://redis.io/topics/transactions](https://redis.io/topics/transactions)



# NestJS에서 캐싱 적용하기
---

위 사항들을 모두 숙지한 상태에서 NestJS를 통해 캐싱을 구현해보도록 하겠습니다

NestJS경우 여러 가지 캐시 저장소(Redis, Memcache, In-Memory)를 지원하고
각 기능을 추상화 래핑 하는 `cache-manager` 모듈이 존재합니다
이 모듈은 다양한 캐시 저장소에 대해 통합된 추상화 인터페이스(퍼사드 패턴, 어댑터 패턴)로 제공하기때문에
도중에 스토리지를 변경하더라도 쉽게 대응할 수 있습니다

하지만 각 캐시 저장소의 세부 기능을 이용할 수 없는 단점이 있습니다
진행하는 프로젝트에서 해당 모듈이 제공하는 기능이 부족함이 없기 때문에 그대로 적용해 보겠습니다.

## CacheManager.Module

```typescript
...
@Module({
  imports: [
CacheModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => {
        // Redis config
        const REDIS_HOST = configService.get<string>('REDIS_HOST', 'localhost');
        const REDIS_PORT = configService.get<number>('REDIS_PORT', 6379);
        return { store: redisStore, host: REDIS_HOST, port: REDIS_PORT };
      },
    }),

...
export:[CacheManagerService]
```

`ConfigModule` 을 사용하기 위해 registerAsync로 환경변수를 가져왔습니다
이후 다른 모듈에서 서비스를 사용하기 위해 `CacheManagerService`를 export합니다

## CacheManager.Service

```tsx
@Injectable()
export class CacheManagerService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  /**
   * 트랜드 포스트 데이터를 캐시에 저장합니다
   * @param requestData:{ ScopeData: searchScope; cursor: number; postListData: MostLikedResponseDto; ttl: number }
   * @returns
   */
  public setTrendPost(requestData: {
    ScopeData: searchScope;
    cursor: number;
    postListData: MostLikedResponseDto;
    ttl: number;
  }) {
    const KEY_NAME = `TrendPost${requestData.ScopeData}:${requestData.cursor}`;
    return this.cacheManager.set(KEY_NAME, requestData.postListData, { ttl: requestData.ttl });
  }

  /**
   * 캐시에 저장된 트랜드 포스트 데이터를 조회합니다
   * @param requestData:{ ScopeData: searchScope; cursor: number }
   * @returns
   */
  public getTrendPost(requestData: { ScopeData: searchScope; cursor: number }) {
    const KEY_NAME = `TrendPost${requestData.ScopeData}:${requestData.cursor}`;
    return this.cacheManager.get(KEY_NAME);
  }
}
```

각 서비스에서 필요한 형태대로 서비스 메서드를 작성합니다
위 예제에서는 인기 게시글 조회 결과 전체를 캐시 매니저를 통해 저장하고 가져오는 서비스 메서드를 구현했습니다

## Post.Controller 에 위 캐시 매니저 서비스 적용

```typescript
...
const cacheResult = await this.CacheManagerService.getTrendPost({ ScopeData: searchScopeData, cursor: cursor });

      if (!!cacheResult) {
        // 캐시가 존재할 경우
        // 응답 리턴
        return ResponseUtility.create(false, 'ok', cacheResult);
      } else {

        // 캐시가 없을 경우 데이터 베이스에 질의
        const getPostsResult = await this.postsService.getMostLiked(getPostsRequestDto);

        // 캐시 저장
        await this.CacheManagerService.setTrendPost({ ScopeData: searchScopeData, cursor: cursor, postListData: getPostsResult, ttl: 5000 });

        // 응답 리턴
        return ResponseUtility.create(false, 'ok', getPostsResult);
...
```

`CacheManager.Service`를 Post 트랜드 리스트를 보여주는 서비스 로직에 적용 했습니다



# 적용 결과
---

![image](https://user-images.githubusercontent.com/55491354/193415324-86fbb025-859e-48f7-962c-0e2f8dc9d694.png)

데이터베이스에 직접 액세스하는 첫번째 요청보다 더 빠르게 사용자에게 데이터를 제공할 수 있었습니다



# 더 읽어보면 좋은 글
---

글을 작성하면서 추가로 리서치한 페이지들 입니다

`Reids Persistence`
레디스는 인메모리 스토리지지만 데이터 스냅샷을 저장해서 불러오는 영속성 옵션을 제공합니다
[https://redis.io/topics/persistence](https://redis.io/topics/persistence)

`Redis Cluster`
레디스는 고가용성을 위한 클러스터링, 샤딩 기능을 제공합니다
[https://redis.io/topics/cluster-tutorial](https://redis.io/topics/cluster-tutorial)

`Redis Pub/Sub`
Publish로 메시지를 보내고 Subscribe으로 메시지를 받는형태의 메시징 페러다임을 구현할 수 있습니다
[https://redis.io/topics/pubsub](https://redis.io/topics/pubsub)

`Redis Stream`
Kafka와 유사항 메시징 큐 시스템을 구현할 수 있습니다
Pub/Sub와 달리 소비자의 개념이 있고 하나의 메시지가 여러 소비자에게 전달되지 않습니다
[https://redis.io/topics/streams-intro](https://redis.io/topics/streams-intro)

NestJS에서는 대표적으로 두가지의 Redis Client 모듈이 존재합니다
지금 포스트에서는 공식 Node-Redis 모듈을 사용하지만 아래와 같은 문제점이 있을 가능성이 높습니다
[https://ably.com/blog/migrating-from-node-redis-to-ioredis](https://ably.com/blog/migrating-from-node-redis-to-ioredis)



# Reference
---

우아한 레디스
[https://www.youtube.com/watch?v=mPB2CZiAkKM&t=1364s](https://www.youtube.com/watch?v=mPB2CZiAkKM&t=1364s)

레디스 카프카 레빗엠큐 차이점에 대해
[https://www.youtube.com/watch?v=H_DaPyUOeTo](https://www.youtube.com/watch?v=H_DaPyUOeTo)

캐시 정책에 대해
[https://sowhat4.tistory.com/64](https://sowhat4.tistory.com/64)
[https://brunch.co.kr/@springboot/151](https://brunch.co.kr/@springboot/151)

NestJS CacheModule연동
[https://medium.com/zigbang/nestjs의-module과-cachemodule을-활용한-redis-연동-2166a771196](https://medium.com/zigbang/nestjs의-module과-cachemodule을-활용한-redis-연동-2166a771196)
