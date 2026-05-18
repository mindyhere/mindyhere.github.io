---
title: "[내일배움캠프 TIL, Day 30] 2차 팀 프로젝트 셋업"
excerpt: "레이어드 아키텍처 패키지 설계, 멀티모듈 구조에서 공통 모듈 의존성 해결"

categories:
  - Studylog
tags:
  - [TIL, MSA, Spring-Cloud, Layered-Architecture, Multi-Module, Gradle]

permalink: /studylog/til-day30-msa-project-setup/

toc: true
toc_sticky: true

date: 2026-05-18
last_modified_at: 2026-05-18
---

## 1. 오늘 학습 키워드

* **MSA 프로젝트 구조**: 모노레포(Multi-Module) 전략, 서비스 시작 순서
* **Layered Architecture**: 표현 → 응용 → 도메인 → 인프라 계층 구조, 도메인별 패키지 구성
* **공통 모듈 활용**

---

## 2. 학습 내용 정리하기

### 📌 MSA 프로젝트, 공통 프로젝트 구조 합의

사전에 껍데기 프로젝트 생성한 것과 SA 기획 문서 작성 한 내용을 토대로 프로젝트 기본 세팅을 마쳤다.

**모노레포(Multi-Module) 구조를 선택한 이유**
- FeignClient로 서비스 간 호출이 많아, 인터페이스 변경 시 한 레포에서 함께 확인 가능
- `settings.gradle`에 `include`만 추가하면 새 서비스 추가가 간단
- `common` 모듈로 공통 코드(BaseEntity 등) 공유가 쉬움

```
sparta-logistics/          ← 루트 (1개의 Git 레포)
├── build.gradle
├── settings.gradle
├── docker-compose.yml     ← 1차 구현 완료 즈음에 별도 작성 예정
├── eureka-server/
├── api-gateway/
├── user-service/
├── hub-service/
├── order-service/
├── delivery-service/
├── company-service/
└── operations-service/
```

> 🤔 염려되는 지점?  
> 일단, **Hub Service가 가장 많은 서비스로부터 FeignClient 호출을 받기 때문에**, Hub 담당자가 재고/경로 관련 API를 먼저 완성해줘야 병목이 생기지 않을 것 같다.   
> 더불어 MSA 프로젝트가 처음인 상황에서 각자의 구현 작업 속도가 제각각일텐데, conflict상황 등 문제 발생에 어떻게 빠르게 대처할 수 있을지가 우려된다.

---

#### (1) Layered Architecture 패키지 구조

과제 요구사항은 **표현 → 응용 → 도메인 → 인프라스트럭처** 계층 구조다. 도메인이 여러 개일 때 아래 두 가지 방식을 고려할 수 있다.

**방법 A: 레이어 우선** (서비스가 단순할 때)
```
user-service/
├── presentation/
├── application/
├── domain/
└── infrastructure/
```

**방법 B: 도메인 우선** (기능이 많고 팀원이 도메인별로 나뉠 때)
```
order-service/
├── order/
│   ├── presentation/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
└── payment/
    ├── presentation/
    ├── application/
    ├── domain/
    └── infrastructure/
```

개인적으로는 MSA도 익숙하지 않은 상태로 과제프로젝트 수준에서 depth가 지나치게 깊어질 것 같아 방법A로 가는 것이 어떨지 의견을 냈었다.  
하지만 order-service처럼 개별 microservice 안에서 주문/결제로 코어 도메인이 나뉘는 경우 **방법 B**가 더 응집도가 높고 관리하기 편할 것이고, 확장성을 고려하면 B안이 낫겠다는 것으로 결론 내려졌다.  

---

### (2) Repository는 어디에?

레이어드 아키텍처에서 Repository를 `domain/`에 인터페이스로 두고 `infrastructure/`에 구현체를 두는 **DIP(의존성 역전)** 방식이 있다.  
그러나 과제 요구사항에서는 "현실적으로는 외부 연동 등 이유로 하위 계층에 직접 의존하는 등 유연하게 적용되기도 합니다."라는 단서를 달아두고 있다.  
DIP를 엄격하게 지키지 않아도 된다면, `infrastructure/`에 `JpaRepository`를 바로 `extends`하고 `application`에서 직접 주입해서 사용하면 어떨까?  
이 부분은 내일 또 팀원들에게 의견을 물어보고 진행해야겠다.

```java
// infrastructure/repository/OrderRepository.java
public interface OrderRepository extends JpaRepository<Order, UUID> {
    Optional<Order> findByOrderIdAndDeletedAtIsNull(UUID orderId);
}

// application/OrderService.java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository; // 직접 주입, 문제없음
}
```

---

## 3. 학습하며 겪었던 문제점 & 에러

### 🤔 문제 상황: Common 모듈 의존성 설정하기
 - `common/entity/BaseEntity.java` 클래스를 `order-service`의 엔티티에서 상속받으려 했으나 **"Cannot resolve symbol 'BaseEntity'"** 에러 발생.
 - `common`은 단순한 폴더일 뿐, Gradle이 인식하는 '모듈'이 아니었기 때문에 발생한 문제

### 🔍 해결 과정
#### Step 1: `settings.gradle`에 모듈 등록. root 프로젝트의 `settings.gradle` 파일에 `common`을 하위 모듈로 포함시킨다. 

```code
rootProject.name = 'sparta-logistics'
include 'common' // 추가
include 'order-service'
// ...기타 서비스들
```

#### Step 2: `common/build.gradle` 생성 및 설정

`common` 모듈 자체도 빌드 설정이 필요하다. 이 때, 다른 서비스에서 라이브러리처럼 참조하므로 실행 가능한 JAR가 아닌 일반 JAR로 빌드되도록 설정한다.

```code
dependencies {
    // ...필요한 의존성 추가
}

bootJar { enabled = false } // 실행 파일 생성 X
jar { enabled = true }      // 라이브러리 JAR 생성 O

```

#### Step 3: 각 서비스 모듈에서 의존성 추가

각 microservice(ex. `order-service`)의 `build.gradle`에서 `common` 프로젝트를 참조하도록 선언한다.

```code
dependencies {
    implementation project(':common') // 공통 모듈 의존성 추가
    // ...기타 의존성
}
```

위 의존성 설정 과정 후 Gradle을 리로드하자 `import com.sparta.common.entity.BaseEntity;`가 정상적으로 작동하는 것을 확인할 수 있었다.

---

#내일배움캠프 #단기Java #TIL #MSA #팀프로젝트 #레이어드아키텍처 #FeignClient #Docker #패키지구조
