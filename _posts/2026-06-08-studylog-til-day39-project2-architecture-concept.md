---
title: "[내일배움캠프 TIL, Day 43] 2차 프로젝트 최종 회고2 : 파이널 전 개념 정리"
excerpt: "심화학습 주차, 개념정리 과제 : 아키텍처 & 서비스 간 통신 개념 정리"

categories:
  - Studylog
tags:
  - [TIL, Project2, Monolithic, MSA, SAGA]

permalink: /studylog/til-day39-project2-architecture-concept/

toc: true
toc_sticky: true

date: 2026-06-08
last_modified_at: 2026-06-08
---

# 아키텍처 & 서비스 간 통신 개념 정리

> 정리 주제: 모놀리식 아키텍처 / 마이크로서비스 아키텍처(MSA) / 서비스 간 통신 방법  
> 이론 중심의 개념 정리 및 지난 프로젝트(물류 시스템 개발 프로젝트) 실제 의사결정 과정과 적용 사례를 함께 기록

---

## 1. 모놀리식 아키텍처 (Monolithic Architecture)

### 개념

모든 기능(주문, 결제, 배송 등)이 **하나의 코드베이스, 하나의 배포 단위**로 구성되는 전통적인 방식이다.  
모든 모듈이 단일 프로세스 안에서 실행되고, 하나의 DB를 공유한다.

```
┌──────────────────────────────────────┐
│            단일 애플리케이션              │
│  ┌────────┐  ┌────────┐  ┌────────┐  │
│  │  주문   │  │  결제   │  │  배송   │  │
│  └────────┘  └────────┘  └────────┘  │
│                 단일 DB               │
└──────────────────────────────────────┘
```

### 장단점

**장점**

- 초기 개발 속도가 빠름 : 모듈 간 통신이 메서드 호출로 끝남
- 배포가 단순함 : JAR 파일 하나만 빌드·배포
- 트랜잭션 처리가 쉬움 : `@Transactional` 하나로 원자성 보장

**단점**

- 규모가 커질수록 유지보수 어려움 : 작은 수정에도 전체 재배포 필요
- 장애가 전체에 영향 (ex.결제 모듈 버그 → 주문·배송 전체 중단)
- 확장이 비효율적 : 특정 기능만 부하가 높아도 전체를 스케일아웃해야 함

### 프로젝트 적용 사례

물류 플랫폼은 주문·재고·배송·허브 도메인이 하나의 흐름 안에 긴밀하게 연결되어 있다.  
모놀리식 구조였다면 특정 도메인에 부하가 집중되거나 장애가 발생할 때 전체 시스템이 중단되는 구조였을 것이다.  
이 리스크를 도메인 단위로 분리하고, 각 서비스가 독립적으로 장애를 격리·확장할 수 있도록 MSA를 채택했다.  

---

## 2. 마이크로서비스 아키텍처 (MSA, Microservice Architecture)

### 개념

애플리케이션을 비즈니스 기능 단위의 **독립적인 소규모 서비스**로 분리하는 아키텍처다.  
각 서비스는 독립된 프로세스로 실행되고, 자체 DB를 가지며, 네트워크를 통해 서로 통신한다.

```
┌─────────────────────────────────────────────────────┐
│                    API Gateway (8080)               │
└──────────────────────────┬──────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   user-service      order-service    company-service
     (19091)           (19094)           (19092)
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    hub-service    delivery-service  operations-service
      (19093)          (19095)           (19096)
```

### 장단점

**장점**

- 독립 배포(ex.order-service만 수정해도 다른 서비스에 영향 없음)
- 장애 격리(ex.company-service가 다운돼도 이미 처리된 주문 조회는 가능)
- 선택적 스케일아웃(ex.주문량이 폭증할 때 order-service만 인스턴스를 늘릴 수 있음)

**단점**

- 분산 트랜잭션이 복잡 : 여러 서비스에 걸친 원자성 보장에 Saga 패턴 등 별도 설계 필요
- 운영 인프라 증가 : Eureka(서비스 디스커버리), Config Server, API Gateway 등 추가 구성 필요
- 네트워크 오버헤드 : 모듈 간 통신이 메서드 호출에서 HTTP 통신으로 바뀌며 지연 발생

### 프로젝트 적용 사례

**서비스 분리**

6개의 독립 서비스로 분리하고, 각 서비스는 전용 PostgreSQL DB를 보유한다. (order-db, company-db, hub-db 등)

**DDD 4계층 구조**

order-service는 도메인 주도 설계(DDD)를 적용해 각 계층이 단방향 의존만 허용한다.

```
order-service/
├── presentation/   ← HTTP 요청/응답 처리 (Controller, DTO)
├── application/    ← 비즈니스 유스케이스 조율 (Service, Port 인터페이스)
├── domain/         ← 핵심 비즈니스 규칙 (Order, CompanyOrder 도메인 객체)
└── infrastructure/ ← 외부 연동 구현체 (JPA Repository, FeignClient Adapter)
```

application 계층은 `CompanyPort` 인터페이스만 알고, 실제 FeignClient 구현체(`CompanyAdapter`)는 infrastructure에 숨겨진다.  
덕분에 외부 서비스 호출 방식이 바뀌어도 domain·application 코드는 수정이 없다.  
이 계층 간 의존 방향은 **ArchUnit 테스트**로 빌드 시점에 자동 검증한다.

---

## 3. 서비스 간 통신 방법

MSA에서 각 서비스는 독립된 프로세스이므로, 협력하려면 네트워크를 통해 통신해야 한다.  
크게 **동기(Synchronous)** 방식과 **비동기(Asynchronous)** 방식으로 나뉜다.

---

### 동기 통신

요청을 보낸 서비스가 **응답이 올 때까지 기다린다.**
대표 방식은 REST(HTTP)이며, Spring에서는 **FeignClient**로 선언적으로 사용할 수 있다.

```
order-service ──요청──▶ company-service
              ◀──응답── (응답을 받은 후 다음 단계 진행)
```

- 흐름이 직관적이고 구현이 간단하다.
- 하위 서비스가 느리거나 다운되면 호출한 서비스의 스레드가 묶이고 연쇄 장애로 이어질 수 있다.
- 여러 서비스를 순서대로 호출할 때, **중간에 실패하면 이전 단계의 처리를 직접 되돌려야 한다.**

---

### 비동기 통신

요청을 보낸 서비스가 **응답을 기다리지 않는다.**
메시지를 브로커(Kafka, RabbitMQ 등)에 발행(Publish)하면, 소비자(Consumer) 서비스가 독립적으로 처리한다.

```
order-service ──이벤트 발행──▶ [Kafka Topic]
                                      ▼
                             operations-service (비동기 처리)
```

- 하위 서비스가 다운돼도 메시지는 브로커에 보관되어 복구 후 처리된다.
- 서비스 간 결합도가 낮아진다 — 발행자는 누가 처리할지 알 필요가 없다.
- 흐름이 여러 서비스에 분산되어 전체 상태를 추적하기 어렵다.

---

### 프로젝트 적용 사례

```
1단계  FeignClient 동기 호출
       └ 통신: 동기 / 트랜잭션 관리: 없음

          ↓ 변화: 트랜잭션 관리 방식 추가 (통신 방식은 그대로)

2단계  Saga 오케스트레이션
       └ 통신: 동기(FeignClient 유지) / 트랜잭션 관리: 보상 트랜잭션

          ↓ 변화: 통신 방식 자체가 바뀜

3단계  코레오그래피 (고도화 방향)
       └ 통신: 비동기(Kafka) / 트랜잭션 관리: 이벤트 기반 보상
```

#### 1단계 — 단순 동기 직접 호출

초기에는 FeignClient로 서비스를 순서대로 호출하는 방식이었다.  
코드는 단순했지만, **중간 단계가 실패했을 때 이전 처리를 되돌릴 방법이 없었다.**

```
order-service → company-service → hub-service → delivery-service → DB 저장
                                       ↑
                               실패 시 이미 처리된 단계 롤백 불가
```

#### 2단계 — Saga 오케스트레이션 (현재 적용)

데이터 불일치 문제를 해결하기 위해 Saga 패턴을 검토했다.  
두 가지 방식을 고려했는데, **코레오그래피**는 Kafka 같은 메시지 브로커 도입이 전제되어 구현 난이도가 높고 기존 FeignClient 코드를 대부분 재작성해야 했다.  
반면 **오케스트레이션**은 기존 동기 호출 구조를 유지하면서 보상 로직만 추가하면 됐기에, 현실적인 판단으로 오케스트레이션을 선택했다.  

order-service가 전체 흐름을 직접 조율하는 **오케스트레이터** 역할을 맡는다.  
각 단계 성공 시 보상 로직을 스택에 등록해두고, 실패 시 역순으로 실행해 이전 상태로 되돌린다.  

```
[주문 생성 Saga]

① 상품 정보 조회   → product-service    ✅  (조회만, 보상 없음)
② 허브 매핑 조회   → company-service    ✅  (조회만, 보상 없음)
③ 재고 예약        → hub-service        ✅  보상 등록: 재고 취소
④ 배송 생성        → delivery-service   ✅  보상 등록: 배송 취소
⑤ 주문 DB 저장     → order-service DB   ✅

③에서 실패 시 → 보상 없음 (스택 비어 있음), 즉시 종료
④에서 실패 시 → ③ 보상(재고 취소) 실행
⑤에서 실패 시 → ④ 보상(배송 취소) → ③ 보상(재고 취소) 순으로 실행
```

```java
// 보상 스택 패턴 (OrderCommandService)
Deque<Runnable> compensations = new ArrayDeque<>();

hubStockPort.reserveStock(order);
compensations.push(() -> hubStockPort.cancelStock(order.getOrderId())); // 보상 등록

try {
    deliveryPort.createDeliveries(order, ...);
    compensations.push(() -> deliveryPort.cancelDeliveries(...));       // 보상 등록
    orderRepository.save(order);
} catch (Exception e) {
    compensations.forEach(Runnable::run); // 역순 보상 실행
    throw e;
}
```

**오케스트레이션의 특징**

- 전체 흐름이 order-service 한 곳에 모여 있어 추적·디버깅이 쉽다.
- 보상 스택이 JVM 메모리에만 존재하므로, 실행 도중 서비스가 재시작되면 보상이 유실되는 한계가 있다.

---

### 3단계 — 고도화 방향 : 이벤트 기반 코레오그래피

오케스트레이션 선택 당시 일단 보류했던 방식이다.  
**각 서비스가 이벤트를 발행하고, 다음 서비스가 그 이벤트를 구독해 스스로 반응**한다.  
중앙 조율자 없이 서비스 간 협력이 이루어진다.

```
[코레오그래피 구조]

order-service    ──▶ [order.created]     ──▶ hub-service (재고 예약)
hub-service      ──▶ [stock.reserved]    ──▶ delivery-service (배송 생성)
delivery-service ──▶ [delivery.created]  ──▶ order-service (주문 확정)

실패 시:
hub-service      ──▶ [stock.failed]      ──▶ order-service (주문 취소)
```

**오케스트레이션 vs 코레오그래피 비교**

|                  | 오케스트레이션 (현재)          | 코레오그래피 (개선 방향)             |
|------------------|-------------------------------|--------------------------------------|
| 흐름 제어        | order-service가 중앙에서 조율 | 각 서비스가 이벤트에 반응            |
| 결합도           | 호출 대상 서비스를 알아야 함  | 이벤트 토픽 이름만 알면 됨           |
| 흐름 추적        | 한 곳에서 파악 가능           | 여러 서비스에 분산 → 추적 어려움     |
| 장애 내구성      | 보상 스택이 메모리에만 존재   | Kafka 메시지가 브로커에 영속 보관됨  |

**이 프로젝트에 적용한다면**

주문 완료 후 Slack 알림·AI 발송 시한 산출처럼 주문 응답을 기다릴 이유가 없는 후속 작업이 적합하다.  
`order.created` 이벤트를 Kafka에 발행하면 operations-service가 구독해 처리하므로,  
order-service는 주문 저장 직후 바로 응답을 반환할 수 있다.  
동시에 현재 오케스트레이션의 메모리 유실 한계도 Kafka의 메시지 영속성으로 해결된다.
