---
title: "[내일배움캠프 TIL, Day 59] Kafka Outbox 패턴 & 분산 락 구현"
excerpt: "매장 등록 이벤트를 Outbox 패턴으로 비동기 발행하고, Redisson 분산 락으로 다중 인스턴스 환경의 중복 발행을 막은 과정"

categories:
  - Studylog
tags:
  - [TIL, Kafka, Redis]

permalink: /studylog/til-day59-kafka-outbox-publisher/

toc: true
toc_sticky: true

date: 2026-06-30
last_modified_at: 2026-06-30
---

## 1. 오늘 학습 키워드

* Kafka Outbox 패턴 (Transactional Outbox)
* Saga 코레오그래피(Choreography)
* Redisson 분산 락 (RLock)
* Kafka Producer 설정 (acks, idempotence, retries)
* JSONB 컬럼 매핑 (`@JdbcTypeCode`)

---

## 2. 개념 정리

### Outbox 패턴이란?

서비스 간 통신에서 "DB 저장"과 "메시지 발행"을 하나의 트랜잭션처럼 안전하게 묶기 위한 패턴이다.

```
[일반적인 방식 - 문제 있음]
DB 저장 → Kafka 발행
  └─ Kafka가 죽으면? DB는 저장됐는데 이벤트는 유실됨 (데이터 불일치)

[Outbox 패턴]
DB 저장 + Outbox 테이블 저장 (같은 트랜잭션)
  └─ 매장 저장 실패 시 Outbox도 같이 롤백 (원자성 보장)
  └─ 별도 스케줄러(Publisher)가 Outbox의 PENDING 행을 읽어 Kafka로 발행
  └─ Kafka가 죽어도 매장 등록 자체는 성공, 이벤트는 PENDING으로 남아 복구 후 재발행
```

핵심은 "비즈니스 트랜잭션"과 "메시지 발행"을 분리하는 것이다. API 응답은 Kafka 발행을 기다리지 않고, Outbox 저장이 끝나는 즉시 반환된다.

### Saga 코레오그래피 — 발행 측 설계에서 고민했던 지점

오케스트레이션/코레오그래피 개념 자체는 이전 프로젝트와 리뷰 컨슈머 구현 때 정리했던 내용이라, 오늘은 **"발행하는 쪽(Producer)을 설계할 때 실제로 무엇을 신경 써야 하는가"**에 집중해서 적용해봤다.

```
[store] 매장 생성 → STORE_CREATED 이벤트 발행 (여기서 끝, 누가 듣는지 모름)
        ↓
[waiting] STORE_CREATED 구독 → 웨이팅 설정 자동 초기화 (자기 할 일만 수행)
```

컨슈머만 구현해봤을 땐 "이미 정해진 이벤트 포맷을 보고 내가 뭘 할지"만 고민하면 됐는데, 발행 측을 설계하니 고민의 종류가 달랐다.

**1. payload는 누가 결정하는가**

설계 단계에서 `STORE_CREATED`의 payload를 `storeId`만 보낼지, 매장의 다른 정보(이름, 카테고리 등)까지 같이 보낼지를 두고 협의가 필요했다. "기본값은 누가 소유하는가"가 기준이 됐는데, 웨이팅 기본 설정값은 waiting 도메인이 소유하는 게 맞다고 보고 **payload는 storeId만, 나머지는 구독자가 알아서 처리**하는 방향(옵션 A)으로 정했다. 이게 코레오그래피의 핵심과 맞닿아 있다 — 발행자는 "사실"만 알리고, 그 사실을 가지고 무엇을 할지는 전적으로 구독자의 책임이다.

**2. 발행자는 구독자의 실패를 책임지지 않는다**

Outbox 패턴 자체가 "내 트랜잭션은 내가 책임지고, 구독자가 그 이벤트로 뭘 하다가 실패하는지는 내 알 바 아니다"라는 코레오그래피의 태도를 그대로 반영한다. store는 Kafka에 발행만 성공하면 책임이 끝나고, waiting이 그 이벤트를 받아 멱등하게 처리하는지는 waiting의 책임 영역이다. 실제로 설계 문서에도 "멱등 처리는 waiting 담당, store는 발행까지만"이라고 명확히 선을 그어뒀다.

### 분산 락이 필요한 이유

서비스가 여러 인스턴스로 떠 있으면(스케일 아웃), `@Scheduled` 메서드는 인스턴스마다 독립적으로 실행된다.

```
인스턴스 A 스케줄러: PENDING 1건 조회 → 발행 시도
인스턴스 B 스케줄러: 같은 PENDING 1건 조회 → 발행 시도 (동시에!)
        ↓
같은 이벤트가 Kafka에 중복 발행됨
```

일반 Java 락(`synchronized`)은 같은 JVM 안에서만 유효하다. 인스턴스마다 별도 프로세스라서 메모리 공유가 안 되므로, Redis 같은 외부 공유 저장소를 "공용 칠판" 삼아 락을 거는 분산 락이 필요하다.

```java
RLock lock = redissonClient.getLock(LOCK_KEY);
if (!lock.tryLock(0, 5, TimeUnit.SECONDS)) {
    return; // 다른 인스턴스가 이미 처리 중 → 양보
}
```

### Kafka Producer 핵심 설정

| 설정 | 의미 |
|------|------|
| `acks: all` | 모든 ISR(동기화된 복제본)이 응답해야 발행 성공 처리. 누락되면 안 되는 이벤트에 사용 |
| `enable.idempotence: true` | 멱등성 프로듀서. 재시도로 인한 Kafka 레벨 중복 발행 방지 |
| `retries: 10` | 전송 실패 시 클라이언트가 즉시 자동 재시도하는 횟수 |

**`retries`(Kafka 클라이언트 레벨)와 Outbox의 `retryCount`(애플리케이션 레벨)는 다른 계층의 재시도**다. `retries`는 찰나의 네트워크 흔들림에 밀리초 단위로 즉시 재시도하는 것이고, `retryCount`는 그래도 최종 실패했을 때 분 단위 간격으로 스케줄러가 다시 시도하는 것이다. 둘 다 있어야 각각 다른 장애 시나리오를 막아준다.

---

## 3. 이번 프로젝트에 적용한 것들

### ✅ StoreOutboxEvent 엔티티

```
StoreOutboxEvent
├── status: PENDING / PUBLISHED / FAILED
├── retryCount: 실패 시 증가, MAX_RETRY_COUNT 초과 시 FAILED로 전환
├── published() / failed(reason) / retry()
└── @JdbcTypeCode(SqlTypes.JSON) - payload를 JSONB로 저장
```

`payload` 컬럼을 JSONB로 매핑할 때 `columnDefinition = "jsonb"`만으로는 부족했다. Hibernate가 INSERT 시 이 정보를 안 쓰고 표준 VARCHAR로 바인딩하려다 PostgreSQL이 타입 불일치로 거부했다. `@JdbcTypeCode(SqlTypes.JSON)` 어노테이션을 추가해야 실제 JSONB 캐스팅이 적용된다.

### ✅ StoreEventEnvelope / StoreEventFactory

Kafka로 나가는 메시지는 다른 서비스가 읽는 외부 규격이라, DTO(일반 클래스)로 명확히 정의했다.

```java
StoreEventEnvelope<StoreCreatedEvent> envelope = storeEventFactory.createStoreCreatedEnvelope(eventId, store);
```

`eventType`은 도메인에서는 enum(`StoreEventType`)이지만, Envelope(JSON 직렬화 대상)에서는 `.name()`으로 String 변환해서 저장한다. 

### ✅ StoreOutboxPublisher

```
publishPendingEvents()  - PENDING 이벤트 폴링 → 발행 (5초 주기)
retryFailedEvents()     - FAILED 이벤트 재시도 (1분 주기)
        ↓ (분산 락으로 보호)
processSingleEvent()    - 이벤트별 독립 트랜잭션(PROPAGATION_REQUIRES_NEW)
        ↓
publish()               - 실제 Kafka 발행, 성공 시 published(), 실패 시 failed()
```

이벤트별로 독립 트랜잭션을 쓴 이유는, 100건을 한 트랜잭션으로 묶으면 1건 실패 시 나머지 99건의 커밋까지 같이 롤백되기 때문이다. `PROPAGATION_REQUIRES_NEW`로 분리하면 한 건의 실패가 나머지에 영향을 주지 않는다.

### ✅ 동시성 검증 방법

처음엔 인스턴스 2개를 직접 띄워서 로그로 확인하려 했는데, 두 인스턴스의 스케줄러 주기가 우연히 어긋나서 "한쪽만 영원히 성공"하는 것처럼 보이는 착시가 있었다. 정확히는 **번갈아가며 락을 잡고 푸는 것**이 정상 동작이고, 동시에 경쟁하는 순간을 봐야 진짜 검증이 된다.

확실한 검증을 위해 `CountDownLatch`로 여러 스레드가 정확히 같은 순간에 `tryLock()`을 호출하도록 강제하는 통합 테스트를 작성했다.

```java
@Test
void only_one_thread_acquires_lock_when_competing_simultaneously() {
    // CountDownLatch로 N개 스레드를 동시에 출발시켜 락 경쟁 강제
    // → 단 1개 스레드만 성공해야 함
    assertThat(successCount.get()).isEqualTo(1);
}
```

이 테스트는 Spring 컨텍스트 전체를 띄우지 않고 `Redisson.create()`로 직접 연결해서, Kafka Consumer/JPA 초기화 같은 불필요한 의존성 없이 순수하게 락 동작만 검증하도록 했다.

---

## 4. 학습하며 겪었던 문제점 & 에러

### 🔴 JSONB 컬럼 INSERT 타입 불일치

**문제**: `columnDefinition = "jsonb"`로 NeonDB에 JSONB 컬럼이 만들어졌는데, Hibernate가 INSERT 시 String을 VARCHAR로 바인딩하려다 `column "payload" is of type jsonb but expression is of type character varying` 에러 발생

**수정**: `@JdbcTypeCode(SqlTypes.JSON)` 어노테이션 추가로 해결

### 🟠 단위 테스트에서 LocalDateTime 직렬화 실패

**문제**: 테스트용 `ObjectMapper`를 `new ObjectMapper()`로 직접 생성하니, 운영 환경(Spring Boot 자동 구성)과 달리 `JavaTimeModule`이 없어서 `LocalDateTime` 직렬화 시 `InvalidDefinitionException` 발생

**수정**: `new ObjectMapper().registerModule(new JavaTimeModule())`로 모듈 명시적 등록

### 🟠 통합 테스트에 `@SpringBootTest`를 썼다가 노이즈 발생

**문제**: 분산 락만 검증하면 되는데 `@SpringBootTest`로 전체 애플리케이션을 띄우니, Kafka Consumer 연결·H2가 PostgreSQL 전용 타입(JSONB, ENUM)을 이해 못 해 테이블 생성 실패·실제 스케줄러가 백그라운드에서 동작하며 우리가 만든 테스트와 뒤섞이는 문제가 발생

**수정**: Spring 컨텍스트를 띄우지 않고 `RedissonClient`를 코드로 직접 생성(`Redisson.create(config)`)해서 순수하게 락 로직만 분리 검증

---

#내일배움캠프 #단기Java #TIL #Kok #Kafka #Redis
