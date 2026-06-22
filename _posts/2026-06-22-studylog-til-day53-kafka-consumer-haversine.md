---
title: "[내일배움캠프 TIL, Day 53] Kafka 코레오그래피 아웃박스 패턴 & Haversine 위치 기반 검색"
excerpt: "Saga 코레오그래피 전략과 Kafka Outbox 패턴 개념 정리, review.events.v1 Consumer 구현, 하버사인 공식 기반 거리 검색 추가"

categories:
  - Studylog
tags:
  - [TIL, Saga, kafka, haversine]

permalink: /studylog/til-day53-kafka-consumer-haversine/

toc: true
toc_sticky: true

date: 2026-06-22
last_modified_at: 2026-06-22
---

## 1. 오늘 학습 키워드

* Saga 패턴 — 코레오그래피(Choreography) 전략
* Kafka Outbox 패턴
* Kafka Consumer 구현 (`@KafkaListener`, Poison Pill 처리, 의미적 유효성 검증)
* Redis 랭킹 동기화 (`TransactionSynchronizationManager.afterCommit()`)
* Haversine 공식 — QueryDSL `Expressions.numberTemplate()`으로 DB에서 거리 계산

---

## 2. 개념 정리

### ✅ Saga 패턴 — 코레오그래피(Choreography) 전략

MSA에서 여러 서비스에 걸친 분산 트랜잭션을 처리하는 패턴이다.
단일 서비스 내에서는 DB 트랜잭션으로 원자성을 보장할 수 있지만, 서비스가 분리된 환경에서는 하나의 트랜잭션으로 묶을 수 없기 때문에 Saga로 대신한다.

Saga의 구현 전략은 두 가지다.

| 전략 | 방식 | 특징 |
|---|---|---|
| **오케스트레이션(Orchestration)** | 중앙 조율자(Orchestrator)가 각 서비스에 명령 | 흐름 파악이 쉽지만 Orchestrator에 로직이 집중됨 |
| **코레오그래피(Choreography)** | 각 서비스가 이벤트를 발행하고, 다른 서비스가 구독해 반응 | 느슨한 결합, 중앙 조율자 불필요, 흐름 파악이 어려울 수 있음 |

오늘 프로젝트에 적용한 방식은 **코레오그래피**다.
Review Service가 리뷰 생성/수정/삭제 시 `review.events.v1` 토픽에 이벤트를 발행하고, Store Service가 이를 구독해 자체 평점과 Redis 랭킹을 갱신한다. 두 서비스는 Kafka 토픽을 통해서만 연결되고 서로를 직접 호출하지 않는다.

**보상 트랜잭션(Compensating Transaction)**
코레오그래피에서 중간 단계가 실패하면 이미 커밋된 앞 단계를 되돌리는 보상 이벤트를 발행해야 한다.  
보상 트랜잭션은 **서비스 간 분산 DB 트랜잭션**을 대상으로 한다. 예를 들어 Store Service가 평점을 갱신(DB 커밋)했는데 이후 연쇄된 다른 서비스의 처리가 실패하면, Store Service에서 평점을 원복하는 보상 이벤트를 발행하는 방식이다.

> Redis 동기화 실패처럼 인프라 레벨의 부분 실패는 Saga 보상 트랜잭션의 대상이 아니다. `afterCommit()` + 예외 처리로 별도 관리한다.

---

### ✅ Kafka Outbox 패턴

**문제:** `@Transactional` 메서드 안에서 DB 저장 + Kafka 발행을 함께 하면 원자성이 깨질 수 있다.
- DB 커밋 성공 → Kafka 발행 실패: 이벤트 유실
- Kafka 발행 성공 → DB 롤백: 이벤트는 나갔지만 실제 데이터는 저장 안 됨

**해결 — Outbox 패턴:**

```
1. 비즈니스 로직 실행 + Outbox 테이블에 이벤트 레코드 저장 (같은 트랜잭션)
2. 별도 프로세스(Relay)가 Outbox 테이블을 폴링
3. 미발행 이벤트를 Kafka로 발행
4. 발행 성공 시 Outbox 레코드를 processed 처리
```

DB 저장과 이벤트 기록이 하나의 트랜잭션이므로 둘 다 성공하거나 둘 다 실패한다. Kafka 발행은 Relay가 재시도하므로 **최소 한 번(at-least-once)** 전달이 보장된다.

> 오늘 프로젝트에서는 Consumer(Store Service) 쪽을 구현했지만, Producer(Review Service)에 Outbox 패턴을 적용하면 이벤트 유실 없이 안정적으로 발행할 수 있다.

---

### ✅ Redis 랭킹 동기화 — `afterCommit()` 패턴
**문제:** `@Transactional` 메서드 안에서 Redis를 직접 갱신하면, DB 롤백 시 Redis만 변경된 상태가 남아 데이터 불일치가 발생한다.
**해결 — `TransactionSynchronizationManager.afterCommit()`:**
DB 커밋이 확정된 이후에만 Redis 작업을 실행하도록 콜백을 등록한다.

```java
TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
    @Override
    public void afterCommit() {
        // DB 커밋 완료 후 Redis 갱신 — 롤백 시 실행되지 않음
        storeRankingRepository.updateScore(storeId, averageRating);
    }
});
```

- DB 롤백 시 afterCommit()은 호출되지 않으므로 Redis 불일치가 발생하지 않는다.
- isSynchronizationActive() 확인으로 트랜잭션 컨텍스트가 없는 경우(테스트 등)도 안전하게 처리한다.
- Redis 작업 실패는 서비스 중단 사유가 아니므로 try-catch로 감싸고 warn 로그만 남긴다.

---

### ✅ Haversine 공식

지구는 구(球)이기 때문에 두 좌표 사이의 직선 거리(유클리드 거리)를 그대로 쓰면 오차가 생긴다.
Haversine 공식은 **구면 위 두 점의 최단 거리(대원 거리, Great-circle distance)**를 계산한다.

```
d = 2R × arcsin( sqrt( sin²((lat2-lat1)/2) + cos(lat1)×cos(lat2)×sin²((lng2-lng1)/2) ) )
```

전개하면 아래와 같이 표현할 수 있고, SQL 함수(`acos`, `cos`, `sin`, `radians`)로 직접 쓸 수 있는 형태다:

```
d = 6371 × acos(
      cos(lat1°) × cos(lat2°) × cos(lng2° - lng1°)
      + sin(lat1°) × sin(lat2°)
    )
```

- `R = 6371` → 지구 평균 반지름(km)
- `lat1`, `lng1` → 기준 좌표 (사용자 입력)
- `lat2`, `lng2` → 각 매장 좌표 (DB 컬럼)
- 결과 단위: **km**

**QueryDSL에서 적용하는 방법**
DB가 지원하는 수학 함수를 QueryDSL에서 직접 사용할 때는 `Expressions.numberTemplate()`을 쓴다.
JPA JPQL은 `acos`, `radians` 같은 함수를 기본 지원하지 않으므로 native SQL 표현식으로 넘겨야 한다.

```java
NumberTemplate<Double> distance = Expressions.numberTemplate(Double.class,
    "6371 * acos(cos(radians({0})) * cos(radians({1})) * cos(radians({2}) - radians({3})) + sin(radians({0})) * sin(radians({1})))",
    lat, store.address.latitude, store.address.longitude, lng
);
return distance.loe(radiusKm); // 반경 이내 필터
```

**주의:** Haversine은 지구가 완전한 구임을 가정하므로 오차가 최대 0.5% 정도 발생할 수 있다. 정밀 지도 서비스가 아닌 이상 실용적으로 충분하다.

---

## 3. 학습하며 겪었던 문제점 & 에러

### 🤔 `@Transactional` 내 Redis 직접 갱신 → DB 롤백 시 불일치 발생

DB 트랜잭션 안에서 Redis를 갱신하면, DB 롤백 시 Redis만 바뀐 상태가 남는다.
`TransactionSynchronizationManager.afterCommit()` 콜백을 사용해 DB 커밋이 완료된 후에만 Redis를 갱신하도록 처리.

### 🤔 `valueOf(null)` → `NullPointerException` (not `IllegalArgumentException`)

`Enum.valueOf(null)`은 IAE가 아닌 NPE를 던진다.
`ReviewEventType.from()` 메서드에서 `NullPointerException`도 함께 catch해야 안전하게 처리 가능.

```java
public static Optional<ReviewEventType> from(String value) {
    try {
        return Optional.of(valueOf(value));
    } catch (IllegalArgumentException | NullPointerException e) {
        return Optional.empty();
    }
}
```

### 🤔 기본형 `int` → JSON 필드 누락 시 `0`으로 역직렬화

`schemaVersion`을 `int`로 선언하면 JSON에 해당 키가 없어도 `0`으로 들어와 유효한 값처럼 통과된다.
Wrapper 타입 `Integer`로 변경해 `null` 여부로 누락을 구분.

### 🤔 REVIEW_DELETED 엣지케이스 — `averageRating` 명세 불명확

마지막 리뷰 삭제 시 `averageRating`이 `0`인지 `null`인지 Review Service 명세가 불명확.
`averageRating` 값 대신 `reviewCount == 0` 여부로 분기해 명세 불확실성 흡수.

---

#내일배움캠프 #단기Java #TIL #SagaChoreography #Kafka #Outbox패턴
