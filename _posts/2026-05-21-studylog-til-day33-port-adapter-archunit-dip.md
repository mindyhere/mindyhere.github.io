---
title: "[내일배움캠프 TIL, Day 33] 선결제 모델 리팩토링과 Port & Adapter 패턴 적용"
excerpt: "ApplicationEventPublisher로 주문-결제 의존 제거, Port & Adapter 패턴으로 크로스 도메인 참조 정리, ArchUnit DIP 규칙 수정"

categories:
  - Studylog
tags:
  - [TIL, MSA, DDD, ArchUnit, HexagonalArchitecture, DIP, SpringEvent]

permalink: /studylog/til-day33-port-adapter-archunit-dip/

toc: true
toc_sticky: true

date: 2026-05-21
last_modified_at: 2026-05-21
---

## 1. 오늘 학습 키워드

* **ApplicationEventPublisher**: 주문 생성 → 결제 생성 흐름을 이벤트로 위임
* **Port & Adapter (헥사고날 아키텍처)**: 크로스 도메인 참조 정리
* **DIP (의존성 역전 원칙)**: Infrastructure가 Application 인터페이스를 구현하는 방향
* **ArchUnit**: 계층 의존성 규칙과 DIP 가이드 간의 충돌 발견 및 수정

---

## 2. 학습하며 겪었던 문제점 & 에러

### 🤔 선결제 모델 적용 - ApplicationEventPublisher

#### 튜터님께 질문한 내용

B2B 물류 프로젝트에서 결제 흐름에 대해 세 가지 고민이 있었다.

1. **B2B 물류에서 결제 흐름의 적절성**  
   실제 B2B는 월말 정산이 일반적인데, 주문 → 결제 → 배송 흐름이 어색하지 않은지?

2. **PG 연동 없이 2단계 흐름의 의미**  
   실제 PG 없이 UUID로만 처리하면서 `PENDING → COMPLETED` 상태 전이를 유지하는 게 의미 있는지?

3. **결제 생성 API 제거 고려**  
   `POST /orders/{orderId}/payments`를 별도로 두지 않고 주문 생성 시 내부적으로 결제까지 자동 처리하는 게 더 자연스럽지 않을지?

#### 튜터님 피드백

> 1. 소액이나 신규 거래에서는 선결제 모델도 존재하니 어색하지 않다. README에 "선결제 모델을 가정한다"고 한 줄 명시하면 충분하다.
> 2. Mock UUID로만 처리하면서 `PENDING → COMPLETED`를 유지하는 것은 **구조만 흉내내는 함정**이 될 수 있다.
> 3. 제안한 방향이 맞다. 주문 없는 결제 PENDING은 존재 의미가 없으므로, **주문 생성 트랜잭션에 묶는 것이 자연스럽다.** 같은 서비스 내부라면 `ApplicationEventPublisher` 또는 서비스 직접 호출로 충분하고, Kafka 같은 메시지 브로커는 불필요하다.

기존에는 `OrderService`가 `PaymentService`를 직접 주입받아 결제를 생성했으나,   
이 피드백을 바탕으로 `PaymentStatus.PENDING` 제거, 결제 생성/확정 API 제거, `ApplicationEventPublisher` 적용을 진행했다.

**변경 전:**
```java
// OrderService가 PaymentService를 직접 참조 ❌
orderService.createOrder(...);
paymentService.createCompletedPayment(...);
```

**변경 후:**
```
OrderService → OrderCreatedEvent 발행
PaymentEventHandler → 이벤트 수신 → PaymentService.createCompletedPayment()
```

```java
// OrderService
eventPublisher.publishEvent(new OrderCreatedEvent(order.getOrderId(), totalPrice));

// PaymentEventHandler
@EventListener
public void handleOrderCreated(OrderCreatedEvent event) {
    paymentService.createCompletedPayment(event.orderId(), event.totalPrice());
}
```

`@EventListener`는 발행자와 **같은 트랜잭션**에서 실행되므로, 결제 실패 시 주문도 함께 롤백된다.  
`OrderCreatedEvent`는 도메인 사실을 표현하므로 `order/domain/event/`에 위치시켰다.

---

### 🤔 크로스 도메인 참조 문제 - Port & Adapter 패턴 적용

결제 취소 시 `PaymentService`가 주문 취소 가능 여부를 판단해야 하는데, 기존에는 `PaymentService`가 `OrderRepository`를 직접 참조하고 있었다.  
마이크로서비스 내부에 Order와 Payment가 도메인 패키지로 나뉘는데, 과연 이 경계를 넘나드는 게 적절한 방향일지가 계속 고민되어 팀원들에게 조언을 구해 리팩토링을 진행했다.

```
PaymentService (payment/application) → OrderRepository (order/domain) ❌
```

**Port & Adapter 패턴으로 해결:**

```
PaymentService → OrderQueryPort (인터페이스만 앎)
                      ↑ implements
              OrderQueryAdapter (payment/infrastructure)
                      → OrderService (order/application)
```

```java
// payment/application/port/OrderQueryPort.java (Outbound Port)
public interface OrderQueryPort {
    boolean isCancellable(UUID orderId);
}

// payment/infrastructure/OrderQueryAdapter.java
@Component
@RequiredArgsConstructor
public class OrderQueryAdapter implements OrderQueryPort {
    private final OrderService orderService;

    @Override
    public boolean isCancellable(UUID orderId) {
        return orderService.isCancellable(orderId);
    }
}
```

`PaymentService`는 `OrderRepository`, `Order`, `CompanyOrder`가 뭔지 전혀 모른다.  
나중에 order/payment 서비스가 분리되면 `OrderQueryAdapter`만 FeignClient로 교체하면 된다.

**Port 위치 고민:**  
처음엔 `payment/domain/service/`에 뒀지만, `PaymentService(application 계층)`가 사용하는 Outbound Port이므로 `payment/application/port/`가 더 정확하다고 판단내렸다.

**헥사고날 적용 수준 및 Adapter 구현 방식 고민:**

`OrderQueryAdapter`가 무엇을 참조해야 하는지, 어디까지 헥사고날을 적용해야 하는지 여러 방면으로 검토했다.

| 방식 | 내용 | 검토결과 |
|---|---|---|
| `OrderRepository` 직접 참조 | Infrastructure → Domain (ArchUnit 허용) | ❌ order 도메인 내부를 직접 노출 |
| `OrderQueryService` 별도 생성 | order 전용 조회 서비스 분리 | ❌ 메서드 하나를 위한 클래스 → 오버엔지니어링<br>(cf. usecase가 더 늘어날 때 고려) |
| `OrderQueryUseCase` 인터페이스 (Inbound Port) | order가 공개 API를 인터페이스로 노출 | ❌ 풀 헥사고날 → 현재 규모에서 오버엔지니어링<br>(cf. usecase가 더 늘어날 때 고려) |
| **`OrderService` 직접 참조** | order가 허용한 공개 서비스 진입점 사용 | **✅ 채택** |

`OrderRepository`는 order 도메인 내부 구현이지만, `OrderService`는 order가 외부에 공개한 진입점이다.  
`isCancellable` 로직도 비즈니스 규칙이므로 Repository가 아닌 Service 계층에 두는 것이 책임 분리에 맞다고 생각했다.  
완전한 헥사고날(UseCase 인터페이스 분리)은 오버엔지니어링으로 판단하고, **`OrderService` 직접 참조로 타협점을 정했다.**

> 나중에 **클레임/환불** 등으로 order 조회 수요가 여러 곳에서 생기거나, 조회 메서드가 3~4개 이상 쌓이면 그때 `OrderQueryService`로 분리해야할지도...🥲

---

### 🤔 ArchUnit 규칙과 DIP 가이드 충돌

`OrderQueryAdapter(Infrastructure)`가 `OrderService(Application)`를 참조하자 CI가 터졌다.

```
[ERROR] layered_architecture_rule FAILED
Application 계층은 Presentation만 접근 가능한데 Infrastructure가 접근함
```

기존 규칙:
```java
.whereLayer("Application").mayOnlyBeAccessedByLayers("Presentation")
.whereLayer("Infrastructure").mayOnlyBeAccessedByLayers("Application", "Domain")
```

프로젝트 DIP 가이드를 다시 살펴보니:
> "Infrastructure → Application 의 인터페이스 구현 (DIP!)"  
> "Infrastructure는 Domain과 Application 두 계층 모두를 바라봅니다"

몇 번을 다시 읽고 내린 결론은 **규칙 자체가 가이드와 불일치하고 있었다**는 점! 여담이지만 4계층 DDD 가이드는 읽을 때마다 새롭다.🫠

**수정된 규칙:**
```java
// Application: Infrastructure도 접근 가능 (DIP 인터페이스 구현 허용)
.whereLayer("Application").mayOnlyBeAccessedByLayers("Presentation", "Infrastructure")

// Infrastructure: 아무도 직접 import하지 않음 (Spring DI가 런타임에 주입)
.whereLayer("Infrastructure").mayNotBeAccessedByAnyLayer()
```

**핵심 이해:**
- `Infrastructure → Application`: Infrastructure가 Application 인터페이스를 `implements` (DIP) ✅
- `Application ↛ Infrastructure`: Application은 인터페이스만 정의하고 Infrastructure를 모름 ✅
- Spring DI가 런타임에 구현체를 주입 → ArchUnit(컴파일타임 정적 분석)에는 안 잡힘

---

## 3. 오늘 구현한 것들

- [x] 선결제 모델: 주문 생성 시 결제 자동 생성 (ApplicationEventPublisher)
- [x] 결제 생성/확정 API 제거, PaymentStatus PENDING 제거
- [x] 출고 준비 확인 / 출고 완료 API 구현 (ORDERED → PREPARING → SHIPPED)
- [x] 결제 취소 조건 추가 (Order.PENDING + CompanyOrder SHIPPED/DELIVERED 없을 때)
- [x] Port & Adapter 패턴으로 payment → order 도메인 간 경계 정리
- [x] ArchUnit 규칙을 DIP 가이드에 맞게 수정

## 4. todo

- [ ] Hub Service FeignClient 연동 (재고 예약/차감/복원)
- [ ] Delivery Service FeignClient 연동 (배송 생성)
- [ ] 권한별 필터링 (주문/결제 목록 조회)

---

#내일배움캠프 #단기Java #TIL #MSA #DDD #ArchUnit #HexagonalArchitecture #DIP #SpringEvent
