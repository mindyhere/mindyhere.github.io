---
title: "[내일배움캠프 TIL, Day 32] 2차 팀프로젝트: 4계층 DDD 구조 개선"
excerpt: "4계층 DDD Architecture 가이드 적용기 + ArchUnit .or() 버그"

categories:
  - Studylog
tags:
  - [TIL, MSA, DDD, ArchUnit]

permalink: /studylog/til-day2-msa-project-4layer-ddd/

toc: true
toc_sticky: true

date: 2026-05-20
last_modified_at: 2026-05-20
---

## 1. 오늘 학습 키워드

* **DDD 4계층 의존성**: `toCommand()` 패턴으로 Application → Presentation 역방향 의존 제거
* **Repository 분리**: 순수 자바 인터페이스(domain) vs JpaRepository 구현체(infrastructure)
* **ArchUnit**: `.or()` 연산자의 패키지 필터 누락 버그

---

## 2. 학습하며 겪었던 문제점 & 에러

### 🤔 계층 DDD 구조 개선: Request DTO 의존성 방향 수정 + Repository 분리

초기 구현에서 Application 계층의 Service가 Presentation 계층의 Request DTO를 직접 받아 사용하고 있었다.

```java
// ❌ 기존: Application → Presentation 역방향 의존
public OrderResult createOrder(OrderCreateRequest request) { ... }

// ✅ 개선: toCommand()로 변환 책임을 Presentation에 위임
// Controller
orderService.createOrder(request.toCommand(), requesterId);

// Request DTO (Presentation)
public CreateOrderCommand toCommand() { ... }

// Service (Application)
public OrderResult createOrder(CreateOrderCommand command) { ... }
```

의존성 방향이 항상 **Presentation → Application** 단방향이어야 한다는 원칙을 코드로 강제했다.

Repository도 동일한 맥락에서 분리했다. JPA 문법에 종속된 메서드명을 도메인 계층에 노출하지 않기 위해 순수 자바 인터페이스(domain)와 JpaRepository 구현체(infrastructure)를 분리했다.

```java
// domain/repository — 순수 자바 인터페이스, JPA 의존성 없음
public interface OrderRepository {
    Optional<Order> findOrderById(UUID orderId);  // 의미 중심 메서드명
}

// infrastructure/repository — Domain Repository 구현체
// OrderJpaRepository를 주입받아 위임, JPA 세부 사항(soft delete 필터 등)을 여기서 캡슐화
@Repository
@RequiredArgsConstructor
public class OrderRepositoryImpl implements OrderRepository {

    private final OrderJpaRepository orderJpaRepository; // JPA 인터페이스 주입

    @Override
    public Optional<Order> findOrderById(UUID orderId) {
        // JPA 메서드명은 infrastructure 안에서만 노출
         return orderJpaRepository.findByOrderIdAndDeletedAtIsNull(orderId);
    }
}
```

- cf. 가이드 문서 참고

```
//요청 흐름
HTTP Request Body
    ↓ (역직렬화)
OrderCreateRequest          ← Presentation 소유
    ↓ (Controller에서 변환)
CreateOrderCommand          ← Application 소유
    ↓ (Service에서 도메인 객체 생성)
Order (Domain Entity)       ← Domain 소유 (= JPA Entity)
    ↓ (JPA 직접 저장 — 변환 없음!)
Database

// 응답 흐름
Database
    ↓ (JPA 조회 — 변환 없이 직접 도메인 객체 반환!)
Order (Domain Entity)       ← Domain 소유 (= JPA Entity)
    ↓ (Service에서 변환)
OrderResult                 ← Application 소유
    ↓ (Controller에서 변환)
OrderResponse               ← Presentation 소유
    ↓ (직렬화)
HTTP Response Body
```

👉🏻 [팀스파르타: 4계층 DDD Architecture 가이드](https://substantial-visage-888.notion.site/4-DDD-Architecture-DIP-327861c68fb1815ea7d6dafbdb4ebfb0)

### 🤔 ArchUnit `.or()` 연산자의 패키지 필터 누락

`domain_prefix_naming_rule`의 Repository 검증 규칙에서 `.or()`를 사용했더니, 앞에 걸어둔 패키지 필터(`.resideInAPackage()`)가 무시되는 버그가 발생했다.

```java
// ❌ 문제: .or() 이후 패키지 필터 소멸 → 전체 클래스 대상으로 매칭됨
classes().that().resideInAPackage("..order.infrastructure.repository..")
        .and().haveSimpleNameEndingWith("RepositoryImpl")
        .or().haveSimpleNameEndingWith("JpaRepository")  // ← 전체 스캔
        .should().haveSimpleNameContaining("Order")
```

결과적으로 `order_prefix_rule`이 `PaymentJpaRepository`를 잡고,  
`payment_prefix_rule`이 `OrderJpaRepository`를 잡는 크로스 도메인 검증 오류가 발생했다.

**원인**: ArchUnit DSL에서 `.or()`는 `that()` 절의 패키지 조건을 리셋하고 전체 클래스를 대상으로 OR 조건을 적용한다.

```java
// ✅ 해결: 패키지 필터를 각각 독립적으로 적용되도록 규칙 분리
classes().that().resideInAPackage("..order.infrastructure.repository..")
        .and().haveSimpleNameEndingWith("RepositoryImpl")
        .should().haveSimpleNameContaining("Order"),

classes().that().resideInAPackage("..order.infrastructure.repository..")
        .and().haveSimpleNameEndingWith("JpaRepository")
        .should().haveSimpleNameContaining("Order")
```

ArchUnit DSL에서 `.or()`는 직관과 다르게 동작한다. 앞의 `.that()` 패키지 조건이 리셋되어 전체 클래스를 대상으로 OR 조건이 적용된다.  
복합 조건이 필요하면 CompositeArchRule 안에서 규칙을 분리하는 것이 안전하다.

### 💡 주문 상품(CompanyOrder/OrderItem status) 출고는 API 명세에 누락된거 같은데...?

내가 구현해야하는 부분 중에 Hub, Delivery 서비스 쪽과 연계해야할 부분이 좀 있는 것 같다.
API명세를 멸심히 작성했다고 생각했는데, 누락된 부분이 있는 것 같기도 하고, 혹은 용어/개념이 달라서 파악이 덜 된 걸지도 🥲

일단 우리가 작성한 SA문서에 서비스 흐름을 아래와 같이 잡아 놨으니, 출고 흐름을 구현하기 전에 Hub, Delivery 담당 팀원과 아래 내용을 먼저 맞춰야 할 것 같다. 

> ### 주문 및 출고 프로세스
> 
> **서비스 흐름:** Order Service → Hub Service → Delivery Service → Operations Service
> 
> ```
> 1. 임시 주문    Order Service
>                사용자가 p_order_drafts에 품목 담기
>       ↓
> 2. 주문 생성    Order Service
>                p_orders 생성 및 업체별 p_company_orders 분할 생성
>       ↓
> 3. 재고 예약    Hub Service (FeignClient 호출)
>                p_warehouse_inventory에서 상품 옵션(SKU)별
>                낙관적 락(version)을 활용한 재고 예약
>                → 재고 부족 시 주문 실패, 전체 롤백
>       ↓
> 4. 출고 준비    Hub Service
>                허브 관리자가 출고 준비 확인
>                p_company_orders.status → PREPARING 
>       ↓
> 5. 배송 생성    Delivery Service (FeignClient 호출)
>                배송 및 전체 경로(p_delivery_routes) 일괄 생성
>                p_company_orders.status → SHIPPED 
>       ↓
> 6. AI 분석     Operations Service (FeignClient 호출)
>                Gemini API로 발송 시한(final_deadline_at) 계산
>       ↓
> 7. 슬랙 알림   Operations Service
>                허브 담당자에게 최종 발송 시한 포함 메시지 발송
> ```

**Hub 담당자와 협의할 것**

- 출고 준비 확인 시 Hub Service가 Order Service의 어떤 엔드포인트를 호출하는지  
  (Hub → Order 방향 FeignClient 구조이므로 Order Service에서 PATCH 엔드포인트를 열어줘야 할까?)
- 재고 예약 실패 시 Order Service 롤백 흐름을 어떻게 처리할지...? 😱

**Delivery 담당자와 협의할 것**

- 배송 생성 요청 시 필요한 파라미터 스펙 (companyOrderId? orderId?)
- 배송 생성 응답에 `deliveryId` 포함 여부  
  (Order Service가 `OrderItem.delivery_id`를 저장하려면 응답값으로 받아야 한다..!)

협의가 선행되지 않으면 FeignClient 인터페이스를 잘못 작성하게 되므로, 구현보다 협의를 먼저 진행할 예정이다.

---

## 3. todo

- [ ] 출고 준비 / 출고 완료 API 구현 (`CompanyOrderStatus`: PREPARING, SHIPPED 추가)
- [ ] Delivery Service FeignClient 연동 (팀원과 API 스펙 협의 선행)
- [ ] Hub Service FeignClient 호출 방향 확인 (Hub → Order 역방향 호출 구조)
- [ ] Draft 기본 CRUD 구현

할 게 산더미다...🫠
역시 결제 PG 연동은 스킵해야겠다. 물류 서비스에 집중해보자!💪🏻

---

#내일배움캠프 #단기Java #TIL #MSA #DDD #ArchUnit
