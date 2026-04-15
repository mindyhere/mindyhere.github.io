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
last_modified_at: 2026-04-15
---

> 🌱 강의를 통해 배운 이론 종합 정리하고 내 프로젝트에 적용해보자!  
> 실무에서 담당했던 과제를 간단한 토이프로젝트로 Springboot와 JPA 환경에 적용해며 겪은 삽질기록 추가

---

## 1. Spring MVC와 객체지향 아키텍처의 재정립

⚒️ 핵심 개념: 3-Layer Architecture & JPA 영속성
* DispatcherServlet & MVC: 모든 요청의 입구 역할을 하는 프론트 컨트롤러 패턴을 복습 👉🏻 계층 간의 명확한 역할 분담(SRP)
* JPA 영속성 컨텍스트: 1차 캐시와 Dirty Checking(변경 감지)을 통한 성능 최적화, **Managed(영속) vs Detached(준영속)** 상태 변화에 따른 사이드 이펙트를 방지하는 법

## 2. 외부 연동과 코드의 품질 (Test & AOP)

💻 핵심 개념: 단위 테스트와 부가 기능의 모듈화
* JUnit5 & Mockito: Repository 등 외부 의존성을 Mock 객체로 대체하여 서비스 로직만 고립시켜 검증하는 **단위 테스트** 전략
* RestTemplate: 서버 간 API 호출 시 Header에 JWT 토큰을 실어 인가(Authorization)를 처리하는 기법 복습
* Spring AOP & Global Exception: 공통 로직(로깅, 권한 체크)을 Aspect로 분리하고, @RestControllerAdvice로 예외 처리를 규격화 👉🏻 핵심 로직의 가독성 향상

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