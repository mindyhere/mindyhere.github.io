---
title: "[내일배움캠프 TIL, Day 8] MSA 강의 요약: Spring 심화부터 마이크로서비스 실무 적용까지"
excerpt: "Spring MVC의 동작 원리부터 MSA 핵심 컴포넌트 이해"

categories:
  - Studylog
tags:
  - [TIL, Spring, JPA, MSA]

permalink: /studylog/til-day8-msa-lecture-summary/

toc: true
toc_sticky: true

date: 2026-04-15
last_modified_at: 2026-04-16
---

> 🌱 강의를 통해 배운 이론 종합 정리하고 내 프로젝트에 적용해보자!  
> 실무에서 담당했던 과제를 간단한 토이프로젝트로 Springboot와 JPA 환경에 적용해며 겪은 삽질기록 추가

---

## 1. Spring MVC와 객체지향 아키텍처의 재정립

⚒️ 핵심 개념: 3-Layer Architecture & JPA 영속성
* HTTP 메서드 : GET(조회) ,POST(생성), PUT(전체 수정), PATCH(부분 수정), DELETE(삭제)
* DispatcherServlet & MVC: 모든 요청의 입구 역할을 하는 프론트 컨트롤러 패턴을 복습 👉🏻 계층 간의 명확한 역할 분담(SRP)
  * (클라이언트에서 api호출) -> (서버)DispatcherServlet에서 요청을 받아서 요청을 처리할 Controller에게 연결 -> (요청처리) -> 응답 리턴 -> HTTP 응답
  * Controller(클라이언트와 비즈니스 로직을 연결하는 역할. 요청과 응답) <-> Service(비즈니스 로직 처리) <-> Repository(데이터에 접근하는 계청)  
  관심사를 분리해 수정과 코드 재사용 등 유지보수를 용이하게 한다.
* JPA 영속성 컨텍스트: 1차 캐시와 Dirty Checking(변경 감지)을 통한 성능 최적화, **Managed(영속) vs Detached(준영속)** 상태 변화에 따른 사이드 이펙트를 방지하는 법
  * JPA : 자바 ORM 기술 표준 명세, 애플리케이션과 JDBC 사이에서 DB 작업을 자동화해준다.
    * 영속성 컨텍스트: Entity 객체를 효율적으로 관리하는 공간.  
    **쓰기 지연 저장소**: 트랜잭션 commit 시점에 SQL을 모아서 한 번에 실행한다.  
    **변경 감지(Dirty Checking**): 영속 상태의 Entity 수정 시, 최초 상태와 비교하여 변경 사항을 자동으로 SQL에 반영한다. 
    
## 2. JPA 연관 관계 심화 및 Spring에서 제공하는 편리한 부가기능들

💻 핵심 개념: 인증과 인가, 단위 테스트, 부가 기능의 모듈화
* 인증과 인가 : **인증**은 사용자가 실제 유저인지 확인하는 절차 & **인가**는 특정 리소스에 대한 접근 허가 여부, 권한을 확인하는 것
  * **쿠키-세션** : 서버가 세션 ID를 통해 클라이언트 상태를 유지하는 방식
  * **JWT(JSON Web Token)** : 토큰 자체에 정보를 담아 암호화하며, 서버가 상태를 저장하지 않는(Stateless) 방식 👉🏻 동시 접속자가 많을 때 서버 부하를 줄일 수 있다.
* RestTemplate: 서버 간 API 호출 시 Header에 JWT 토큰을 실어 인가(Authorization)를 처리하는 기법 👉🏻 `exchange`를 사용하여 헤더 정보를 포함한 복합적인 요청이 가능하다.
* JPA 연관 관계 심화 : DB 테이블에는 방향 개념이 없으나, JPA Entity는 단방향과 양방향 관계가 존재하고, Entity 간에 1:1, N:1, 1:N, N:M 관계를 매핑하여 다룰 수 있다.  
  * **지연 로딩(Lazy Loading)** : 연관된 Entity 정보를 실제 필요한 시점에 가져오는 방식. 트랜잭션 내에서 영속성 컨텍스트가 유지되어야 한다.
  * **영속성 전이(CASCADE)** : 특정 Entity의 작업이 연관된 Entity까지 전파되는 기능. PERSIST, REMOVE 옵션 등이 있다.
  * **고아 객체(orphanRemoval)** : 연관 관계가 끊어진 자식 Entity를 자동 삭제하는 옵션
* **JUnit5와 Mockito**를 활용한 단위테스트
  * Repository 등 외부 의존성을 Mock 객체(가짜 객체)로 대체하여 서비스 로직만 고립시켜 검증하는 **단위 테스트** 전략
  * cf. 통합 테스트(Integration Test) : 여러 모듈을 연결하여 상호 작용을 검증 👉🏻 `@SpringBootTest`를 사용하여 실제 스프링 환경을 구동한다.
* Spring AOP & Global Exception: 공통 로직(로깅, 권한 체크)을 Aspect로 분리하고, `@RestControllerAdvice`로 예외 처리를 규격화 👉🏻 핵심 로직의 가독성 향상

## 3. MSA 생태계와 분산 환경의 이해

🌐 핵심 개념: Spring Cloud Ecosystem
* **Service Discovery (Eureka)**: 서비스 인스턴스의 위치를 동적으로 관리하고 헬스 체크를 수행하는 중앙 레지스트리
* **API Gateway & Security**: 클라이언트 요청의 단일 진입점. ServerWebExchange를 활용한 비동기 필터링과 JWT 기반의 통합 인증/인가 체계를 구축할 수 있다.
* **Distributed Tracing & Zipkin**: Zipkin과 Micrometer를 활용해 여러 서비스로 흩어지는 요청의 Trace와 Span을 추적하여 시스템 가시성을 확보한다.
* **Event-Driven Architecture (EDA)**: 서비스 간의 강한 결합을 끊고 메시지 브로커를 통해 비동기적으로 소통하는 구조

---

### ⁉️ 실습하며 겪은 문제 & 해결

* backoffice 개인프로젝트에서 계층 구조 설계  
  * Program -> GridMeta -> GridGroup -> ColumnMeta로 이어지는 도메인 계층 구조를 JPA 연관관계 설계
* 풍성한 도메인 모델: `@PrePersist`, `@PreUpdate`를 활용하여 비즈니스 로직을 엔티티 내부에 응집시켜 엔티티 책임 강화
* Gateway Security: 실습을 통해 JWT 필터에서 exchange.mutate()를 사용하여 인증된 사용자 정보를 하위 서비스로 안전하게 전달하는 기법을 적용

  > Gateway 필터 검증?
  > * **상황**: 로그인/회원가입 요청 시 JWT 필터 로그가 남지 않아 필터 동작 여부 혼동.
  > * 설계상 `permitAll` 경로에 대해 필터를 스킵하도록 구현했음을 로직 분석을 통해 확인.  
  > `Pre Filter`와 `Post Filter`의 로그 순서를 통해 정문 보안 시스템이 의도대로 작동함을 확인할 수 있었다.

---

#내일배움캠프 #단기Java #TIL #SpringCloud #MSA #JPA #리팩토링