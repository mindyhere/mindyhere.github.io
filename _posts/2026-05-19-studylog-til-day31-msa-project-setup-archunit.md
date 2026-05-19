---
title: "[내일배움캠프 TIL, Day 31] 2차 팀프로젝트 셋업 : 도메인 모델링과 아키텍처 규칙 적용"
excerpt: "ArchUnit 규칙 정비, 엔티티 ERD 검토, record DTO, AuditorAware, Config Server 에러 원인 분석"

categories:
  - Studylog
tags:
  - [TIL, MSA, Spring-Cloud, ArchUnit, DDD, JPA]

permalink: /studylog/til-day31-msa-project-setup-archunit/

toc: true
toc_sticky: true

date: 2026-05-19
last_modified_at: 2026-05-19
---

## 1. 오늘 학습 키워드

* **ArchUnit**: `allowEmptyShould`, `domain/core/` 패키지 규칙, 계층별 패키지 경로 일치

---

## 2. 학습하며 겪었던 문제점 & 에러

### 🤔 ArchUnit 규칙 정비

`allowEmptyShould(false)`로 올려둔 PR을 머지하면서, 구현 초기 단계에서는 모든 패키지가 채워져 있지 않아 테스트가 계속 실패하는 문제가 있었다.

**`allowEmptyShould` 동작 방식**

| 설정 | 동작 |
|---|---|
| `true` | 조건에 매칭되는 클래스가 없으면 테스트 통과 (초기 구현 단계에 적합) |
| `false` | 매칭 클래스가 없어도 무조건 실패 → 누락 강제 확인 (최종 제출 전에 적합) |

구현 초기 단계이므로 전체를 `true`로 설정해두고, 각자 마이크로서비스에 구현이 어느 정도 진행되었다면 `false`를 적용해 규칙을 확인하도록 하는 방법이 좋을 것 같다.  

**최종 패키지 구조**

```
order/
├── presentation/
│   └── controller/         ← *Controller.java
├── application/
│   └── service/            ← *Service.java
├── domain/
│   ├── core/               ← @Entity, enum, 순수 자바 클래스
│   └── repository/         ← *Repository.java (순수 인터페이스. 필요 시 선택사항)
└── infrastructure/
    └── repository/         ← *JpaRepository.java, RepositoryImpl.java 와 같은 구현체
```

**`domain/core/` 규칙 추가**

팀원들과 논의 해본 결과, 엄격한 DDD를 적용하기에는 다소 무리가 있다고 판단했다.  
JpaRepository 인터페이스는 `infrastructure/repository` 하위에 곧바로 두고, JPA 엔티티, enum, 순수 자바 클래스를 `domain/core/`에 모두 두기로 최종 합의했다.
규칙에는 다음 두 가지를 추가 작성했다.

```java
/**
 * [Domain 계층 규칙]
 * 1. JPA 엔티티(@Entity)는 반드시 ..domain.core 아래에 위치해야 함.
 * 2. ..domain 패키지 내 enum 클래스는 반드시 ..domain.core 아래에 위치해야 함.
 * 3. 순수 자바 Repository 인터페이스는 'Repository'로 끝나야 하며 ..domain.repository 아래에 위치해야 함.
 *    (JpaRepository를 상속하는 인터페이스는 JpaRepository로 끝나며 infrastructure에 위치)
 */
@ArchTest
static final ArchRule domain_layer_entity_location_rule =
    classes().that().areAnnotatedWith(Entity.class)
        .should().resideInAPackage("..domain.core..")
        .allowEmptyShould(true)
        .as("JPA @Entity 클래스는 반드시 ..domain.core 패키지 아래에 위치해야 합니다.");

@ArchTest
static final ArchRule domain_layer_enum_location_rule =
    classes().that().resideInAPackage("..domain..")
        .and().areEnums()
        .should().resideInAPackage("..domain.core..")
        .allowEmptyShould(true)
        .as("Domain 계층의 enum 클래스는 반드시 ..domain.core 패키지 아래에 위치해야 합니다.");
```


### 🤔 MSA에서 다른 서비스 엔티티 참조 원칙

```java
// ❌ MSA에서 불가능 - 다른 서비스의 DB에 JPA 관계 불가
@ManyToOne
private Delivery delivery;

// ✅ 올바른 방식 - ID만 보관, 데이터 필요 시 FeignClient 호출
@Column(name = "delivery_id")
private UUID deliveryId;
```

**기준**: 해당 엔티티의 테이블이 내 서비스 DB에 있으면 `@ManyToOne`, 다른 서비스 DB에 있으면 `UUID`.  
`order-service`에서 JPA 관계를 쓸 수 있는 건 `Order ↔ CompanyOrder ↔ OrderItem` 세 엔티티 간에만이다.

### 🤔 B2B 주문 구조 이해

설계문서나 ERD를 볼 때마다 계속 헷갈리는 부분이 주문 구조 부분인 것 같다. 바로 직전에 했던 프로젝트와 겹쳐서 그런걸까...  
어쨌거나! 이번 프로젝트는 B2B 물류 플랫폼을 만드는 것이고, 하나의 주문이 여러 업체의 상품을 묶어 발주하는 구조를 가정하였기 대문에, 최종적으로는 다음과 같은 구조가 된다.

```
Order (전체 주문 - 주문자 입장)
├── CompanyOrder (A업체 주문건) ← A업체가 자기 주문만 확인
│   ├── OrderItem (상품1)
│   └── OrderItem (상품2)
└── CompanyOrder (B업체 주문건) ← B업체가 자기 주문만 확인
    └── OrderItem (상품3)
```

`CompanyOrder`는 수령/공급 업체 입장에서 자기 몫만 조회하기 위한 분리 단위이고, `Order`는 주문자가 한 번에 여러 업체에 발주하는 전체 주문 묶음이다.

---

## 3. todo

- [ ] `OrderService` CRUD 로직 구현 (주문 생성, 조회, 취소, 출고 처리)
- [ ] `OrderRepository` 연결 및 기본 동작 확인
- [ ] 동시성 제어 포인트 파악 (재고 예약, 주문 상태 변경)
- [ ] `PaymentService` CRUD 로직 구현

언제다하지...🫠🫥

---

#내일배움캠프 #단기Java #TIL #MSA #ArchUnit #DDD #JPA
