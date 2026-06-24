---
title: "[내일배움캠프 TIL, Day 55] Redis 캐싱 & 랭킹 구현"
excerpt: "Cache-Aside 패턴으로 매장 목록 캐싱, Redis Sorted Set으로 인기 매장 랭킹 구현, 그리고 포스트맨 테스트 중 발견한 버그 수정"

categories:
  - Studylog
tags:
  - [TIL, Redis]

permalink: /studylog/til-dayday55-redis-cache-ranking/

toc: true
toc_sticky: true

date: 2026-06-24
last_modified_at: 2026-06-24
---

## 1. 오늘 학습 키워드

* Redis Cache-Aside 패턴
* Redis Sorted Set (ZADD / ZREVRANGE)
* 캐시 무효화 전략 (`afterCommit`, SCAN)
* Kafka 이벤트 기반 Redis 갱신
* JPA flush 타이밍 & Partial Index (soft delete)

---

## 2. Redis 개념 정리

### Cache-Aside 패턴이란?

애플리케이션이 캐시를 직접 관리하는 가장 일반적인 캐시 전략이다.

```
[조회 요청]
  └─ 캐시 히트  → 캐시에서 바로 반환 (DB 미조회)
  └─ 캐시 미스  → DB 조회 → 결과를 캐시에 저장(TTL 설정) → 반환
```

- **장점**: DB 부하 감소, 읽기 성능 향상
- **단점**: 첫 요청(cold start)은 항상 DB를 탄다. 캐시와 DB 사이에 일시적인 불일치(stale) 가능성이 있다.
- **무효화 시점**: 데이터가 변경될 때 캐시를 삭제해야 다음 조회 시 최신 데이터를 반영할 수 있다.

### Redis String vs Sorted Set

| 자료구조 | 사용 목적 | 주요 명령 |
|----------|-----------|-----------|
| String | 단일 값 또는 직렬화된 객체 저장 (캐싱) | GET / SET / DEL |
| Sorted Set | score 기반 순위 데이터 관리 | ZADD / ZREVRANGE / ZREM |

**Sorted Set**은 각 member(storeId)마다 score(averageRating)를 함께 저장한다. ZADD는 멱등(idempotent) — 같은 member를 다시 ZADD하면 score가 덮어써진다.

### SCAN vs KEYS

- `KEYS pattern`: 매칭되는 모든 키를 한 번에 반환 → **운영 환경에서 Redis를 블로킹**할 수 있어 위험
- `SCAN cursor COUNT n`: 커서 기반으로 조금씩 탐색 → 블로킹 없이 안전하게 키 순회 가능

캐시 전체 무효화(`evictAll`)에는 반드시 SCAN을 사용해야 한다.

### afterCommit()이 중요한 이유

`@TransactionalEventListener(phase = AFTER_COMMIT)` 또는 `afterCommit()` 콜백은 **DB 트랜잭션이 완전히 커밋된 뒤** Redis를 갱신한다.

- 커밋 전에 Redis를 갱신하면: DB 롤백 시 Redis에는 잘못된 데이터가 남는다.
- 커밋 후에 갱신하면: DB와 Redis의 불일치 가능성을 최소화할 수 있다.

---

## 3. 이번 프로젝트에 적용한 것들

### ✅ 매장 목록 검색 캐싱 (Cache-Aside)

캐시 추상화 인터페이스 `StoreListCacheRepository`를 application 레이어에 두고, Redis 구현체 `StoreListRedisRepository`를 infra 레이어에 분리했다. 
애플리케이션 레이어가 Redis에 직접 의존하지 않으므로 나중에 구현체를 교체해도 비즈니스 로직은 변경이 없다.

**캐시 키 설계**

```
store:list:{categoryIds}:{sido}:{sigungu}:{keyword}:{amenities}:{status}:{sort}:{page}:{size}
```

- 조건 미지정 필드는 `ALL`로 치환해 키 충돌 방지
- 편의시설 목록은 enum 이름을 **정렬 후 결합** → 요청 파라미터 순서가 달라도 동일 캐시 키 보장
- TTL: 300초 (5분)
- OWNER 요청은 캐시 바이패스: 본인 매장만 조회하므로 캐시 효과가 낮고 다른 OWNER 데이터와 격리 필요

**캐시 무효화 시점**

- 매장 데이터 변경(생성/수정/삭제/상태변경) → DB 커밋 완료 후 `evictAll`
- Kafka로 평점 변경 이벤트 수신 시에도 `evictAll`
- 무효화에 `SCAN` 명령 사용 (KEYS 대신)

**장애 대응**

Redis 조회/저장 실패 시 예외를 전파하지 않고 DB fallback으로 처리 → 가용성 우선 설계

**Jackson 직렬화 이슈**

`PageImpl`은 기본적으로 역직렬화가 불가능하다. 내부 `CachedPage` DTO를 만들어 `content` + `totalElements`만 저장하는 방식으로 해결했다.

---

### ✅ 인기 매장 랭킹 (Redis Sorted Set)

```
StoreRankingRepository (interface, domain 레이어)
└── StoreRankingRedisRepository (infra 레이어, Redis ZSet)
```

| 메서드 | 동작 |
|--------|------|
| `updateScore(storeId, averageRating)` | ZADD — score 중복 시 덮어씀 (멱등) |
| `remove(storeId)` | 리뷰 전체 삭제(reviewCount=0) 시 랭킹에서 제거 |
| `getTopRanking(size)` | `reverseRange`로 averageRating 내림차순 상위 N개 |

Kafka `review.events.v1` 이벤트 수신 → DB 평점 갱신 → DB 커밋 후 → Redis ZSet score 갱신 + 캐시 무효화

**랭킹 순서 보존 문제**

Redis ZSet에서 받은 storeId 순서를 DB 조회 후에도 유지해야 했다. `Map<UUID, Store>`로 조회한 뒤 rankedIds 순서대로 재정렬해 반환하는 방식으로 해결했다.

---

### ✅ 전체 흐름 요약

```
[리뷰 작성]
  └─ review-service → Kafka(review.events.v1)
       └─ store-service Consumer
            ├─ DB: store.averageRating, reviewCount 갱신
            ├─ afterCommit → Redis ZSet(store:ranking) score 갱신
            └─ afterCommit → Redis String(store:list:*) 전체 무효화

[매장 목록 조회]
  └─ StoreService.searchStores()
       ├─ 캐시 히트 → Redis에서 바로 반환
       └─ 캐시 미스 → DB 조회 → Redis 저장 (TTL 300s) → 반환

[랭킹 조회]
  └─ StoreRatingService.getRanking(size)
       ├─ Redis ZSet reverseRange(0, size-1)
       └─ storeId 목록으로 DB 조회 → Redis 순서 보존해 반환
```

---

## 4. 학습하며 겪었던 문제점 & 에러

### 🔴 이미지 슬롯 교체 시 Unique Key 충돌

**문제**: 이미지 수정 시 기존 슬롯을 soft delete(`deletedAt` 세팅)한 뒤 새 이미지를 같은 `displayOrder`로 넣으면 `(store_id, display_order)` Unique Constraint 위반이 발생했다. 
JPA 플러시 시점에 delete와 update가 같은 트랜잭션 안에서 처리되면서 UK 충돌이 나는 구조적 문제였다.

**수정**:
- `@UniqueConstraint` 제거 → DB 레벨 Partial Index(`deleted_at IS NULL` 조건)로 교체 → soft delete된 행은 유일성 검사 대상에서 제외
- 슬롯 교체 시 기존 이미지 soft delete 후 즉시 `flush`(`evictImageSlot`)해 insert 전 슬롯을 비움


### 🟠 `PERMANENTLY_CLOSED` 매장 Kafka 재시도 루프

**문제**: `findActiveOrThrow()` 사용 시 삭제/비활성 매장 이벤트가 들어오면 예외 발생 → Kafka가 계속 재시도하는 무한 루프 발생

**수정**: `findById()`로 교체 후 soft delete / 비활성 상태 체크 → 조건에 해당하면 조용히 skip

### 🟠 `selectDistinct` + `ORDER BY` SQL 에러

**문제**: 편의시설 필터가 없을 때도 `selectDistinct` 적용 → Haversine 거리 정렬 식이 SELECT 절에 없어 SQL 표준 위반 에러 (H2/PostgreSQL 공통)

**수정**: 편의시설 필터 존재 여부에 따라 `selectDistinct` / `select` 분기 처리


---

#내일배움캠프 #단기Java #TIL #Kok #Redis
