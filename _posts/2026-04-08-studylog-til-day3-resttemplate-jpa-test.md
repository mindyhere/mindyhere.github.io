---
title: "[내일배움캠프 TIL, Day3] RestTemplate 외부 연동부터 JPA 연관관계 심화 및 테스트 코드 입문"
excerpt: "RestTemplate을 이용한 서버 간 통신, JPA 연관관계 주인 및 영속성 전이, JUnit5/Mockito 기반 테스트 전략 정리"

categories:
  - Studylog
tags:
  - [TIL, Spring, JPA, RestTemplate, TestCode]

permalink: /studylog/til-day3-resttemplate-jpa-test/

toc: true
toc_sticky: true

date: 2026-04-08
last_modified_at: 2026-04-08
---

## 1. 오늘 학습 키워드
* **RestTemplate**: 서버 간 API 호출을 위한 동기식 HTTP 클라이언트
* **Entity Relationship**: 1:1, N:1, 1:N, N:M 관계 및 연관관계의 주인(Owner)
* **Persistence Management**: 지연 로딩(Lazy Loading), 영속성 전이(Cascade), 고아 객체 제거
* **Unit & Integration Test**: JUnit5와 Mockito를 활용한 고립된 단위 테스트 및 통합 테스트

---

## 2. 학습 내용 정리하기

### 🌐 RestTemplate: 서버 간의 대화
스프링에서 다른 서버의 API를 호출할 때 사용하는 도구임.
* **GET/POST 요청**: `getForEntity`, `postForEntity`를 통해 JSON과 자바 객체 간의 자동 매핑을 지원함.
* **exchange()**: HTTP Header에 정보를 담아야 할 때 필수적임.
  * **cf. 실무 경험**: 예전 회사에서 벡터 검색 기능을 호출할 때 JWT 토큰을 헤더에 실어 인가(Authorization)를 받았던 과정이 바로 이 방식이었음을 깨달음.
* **UriComponentsBuilder**: 복잡한 URI와 쿼리 파라미터를 동적으로 안전하게 생성함.

### 🏗️ JPA 연관관계와 객체 지향의 차이
DB 테이블은 FK 하나로 양방향 조회가 가능하지만, 객체는 참조 필드가 있는 쪽에서만 상대를 볼 수 있는 '방향성'의 개념이 중요함.
* **외래 키의 주인(Owner)**: 실제 DB의 FK를 관리하는 쪽(주로 N쪽)이며, 주인이 아닌 쪽은 `mappedBy`를 사용하여 읽기 전용임을 명시함.
* **N:M 관계**: JPA가 중간 테이블을 자동으로 생성해주지만, 실무적 유연성을 위해 중간 테이블용 엔티티를 직접 설계하는 방식이 권장됨.

### 🧪 테스트 코드 작성

* **단위 테스트(Unit Test)**: 가장 작은 단위(메서드)를 고립시켜 테스트함. 외부 의존성을 끊기 위해 **Mockito** 라이브러리를 활용함.
  * cf. Mockito: 왜 가짜 객체가 필요한가?  
    서비스 로직만 테스트하고 싶은데, 의존성 주입(DI)으로 얽힌 Repository 때문에 단위 테스트가 불가한 경우, 실제 Bean 주입 없이 서비스 로직만 검증하기 위해, Repository를 **가짜 객체(Mock Object)**로 대체하여 의존성 문제를 해결할 수 있다.  
    `@Mock`으로 가짜 객체를 만들고 `given().willReturn()`으로 동작을 정의(Stubbing)하여 의존성을 분리함.
* **통합 테스트(Integration Test)**: 스프링 컨테이너를 구동하여 전체적인 흐름과 설정을 검증함.

---

## 3. 학습하며 겪었던 문제점 & 에러

### 🔍 개념 재정립 및 심화 학습

#### 1) 지연 로딩(Lazy Loading)과 영속성 전이(Cascade)
* 둘을 영속성 관리의 동일 범주로 생각했으나, 사실은 다른 개념이다.
  * **Lazy Loading**: 성능 최적화를 위해 객체를 실제 사용 시점에 조회하는 것.
  * **Cascade**: 부모의 상태 변화를 자식에게 전파하는 것.
* 두 기능 모두 영속성 컨텍스트를 활용하므로 `@Transactional` 환경이 필수적임을 인지함.

#### 2) 고아 객체 제거(orphanRemoval)의 주의사항
* `@ManyToOne`에서는 이 옵션을 사용할 수 없음.  
자식 입장에서 부모를 삭제할 때 해당 부모를 참조하는 또 다른 자식 객체들이 존재할 수 있기 때문이다. 논리적 정합성을 위해 일대일이나 일대다 관계에서만 사용함.

---

## 4. 내일 학습 할 것은 무엇인지
- [ ] **Spring 심화 코스**
- [ ] **스프링 AOP(Aspect Oriented Programming)**
- [ ] **글로벌 예외 처리**

---

#내일배움캠프 #단기Java #TIL #RestTemplate #JPA심화