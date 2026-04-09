---
title: "[내일배움캠프 TIL, Day4] Spring 심화 완강 & MSA 아키텍처 입문"
excerpt: "Controller 테스트, AOP 모듈화, 전역 예외 처리 및 MSA 핵심 컴포넌트(Eureka, Gateway) 정리"

categories:
  - Studylog
tags:
  - [TIL, Spring, AOP, ExceptionHandling, MSA, SpringCloud]

permalink: /studylog/til-day4-spring-advanced-msa/

toc: true
toc_sticky: true

date: 2026-04-09
last_modified_at: 2026-04-09
---

## 1. 오늘 학습 키워드
* **Controller Test**: `@WebMvcTest`와 Mock 객체를 활용한 슬라이스 테스트
* **Spring AOP**: 공통 관심사(Aspect) 분리를 통한 부가기능 모듈화
* **Global Exception Handling**: `@RestControllerAdvice`를 이용한 예외 처리 중앙화
* **MSA (Microservices Architecture)**: 모놀리틱 아키텍처와의 차이 및 특징
* **Spring Cloud**: Eureka(Service Discovery), Gateway, FeignClient(Load Balancing), Resilience4j(Circuit Breaker)

---

## 2. 학습 내용 정리하기

### 🧪 Controller 테스트와 Mocking
* **@WebMvcTest**: 모든 빈을 띄우지 않고 컨트롤러 계층만 테스트함.
* **@MockBean**: 컨텍스트에 가짜 빈을 주입하여 실제 서비스 로직과의 의존성을 끊음.
* **Security Test**: Spring Security가 적용된 경우 가짜 필터(`MockSpringSecurityFilter`)나 가짜 인증 객체를 주입하여 인가 과정을 시뮬레이션함.

### 🏗️ Spring AOP: 핵심 기능과 부가 기능의 분리

* **개념**: 여러 메서드에 공통적으로 들어가는 로직(로그, 시간 측정, 권한 체크 등)을 한 곳에서 관리함.
* **Advice 종류**:
  - `@Around`: 가장 강력함. 전/후 처리 모두 가능.
  - `@Before`, `@AfterReturning`, `@AfterThrowing` 등 시점별 세밀한 제어 가능.
* **Pointcut**: 어디에 이 기능을 적용할지 `execution` 언어로 정의함.

  ```
  // 1. 가장 기본적인 형태: 특정 패키지 내 모든 메서드에 적용
  // 모든 리턴 타입 / 해당 패키지 및 하위 패키지의 모든 클래스 / 모든 메서드(모든 파라미터)
  @Around("execution(* com.sparta.myselectshop.controller..*(..))")
  public Object executeAllInPackage(ProceedingJoinPoint joinPoint) throws Throwable { ... }
  
  // 2. 특정 클래스의 모든 메서드에 적용
  // 모든 리턴 타입 / ProductService 클래스의 모든 메서드 / 파라미터 0개 이상
  @Before("execution(* com.sparta.myselectshop.service.ProductService.*(..))")
  public void logServiceAccess() { ... }
  
  // 3. 특정 이름으로 시작하는 메서드에만 적용
  // 모든 리턴 타입 / 이름이 'get'으로 시작하는 모든 메서드 / 파라미터 0개 이상
  @AfterReturning("execution(* get*(..))")
  public void afterGetMethods() { ... }
  
  // 4. 파라미터 타입에 따른 매핑
  // 모든 리턴 타입 / 모든 메서드 / 첫 번째 파라미터가 Long 타입인 경우만
  @Before("execution(* *(Long, ..))")
  public void beforeWithLongParam() { ... }
  ```


### 🛡️ API 예외 처리 및 에러 메시지 관리
* **@RestControllerAdvice**: 전역적으로 발생하는 예외를 한 곳에서 잡아 응답을 규격화함. 코드 중복이 획기적으로 줄어들고 유지보수가 용이해짐.
* **MessageSource**: 에러 메시지를 `messages.properties`에 관리하여 코드와 텍스트를 분리함.

### 🌐 MSA(마이크로서비스 아키텍처) 입문

* **Monolithic**: 모든 기능이 하나의 프로젝트에 있음. 프로젝트 초기에 빠른 개발과 배포가 가능하지만, 규모가 커지면 코드 베이스가 비대해져 작은 수정에도 전체를 재배포해야 하는 관리의 한계가 있다.
* **MSA**: 기능을 독립적인 서비스로 분리. 각 서비스가 유연하게 확장 가능하지만, 서비스 간 네트워크 통신 비용과 분산 환경에서의 데이터 일관성 관리 등 복잡성이 증가한다.
* **핵심 컴포넌트**:
  - **Eureka**: 서비스 인스턴스의 주소(IP, Port)를 등록하고 관리하는 중앙 레지스트리. 서비스 간의 위치를 찾는 'Service Discovery' 역할을 수행하며 각 서비스의 헬스 체크를 담당한다.
  - **API Gateway**: 클라이언트 요청의 단일 진입점. 적절한 서비스로의 라우팅, 인증/인가, 로깅 등을 중앙에서 일괄 처리한다.
  - **FeignClient**: 선언적 인터페이스를 통해 복잡한 HTTP 호출 코드를 대체하는 웹 클라이언트. 내부적으로 로드밸런싱을 수행하여 요청을 분산한다.
  - **Circuit Breaker**: 장애 전파 방지를 위한 안전장치. 호출 실패율이 임계치를 넘으면 회로를 **Open 상태**로 전환하여 즉시 **Fallback** 응답을 반환함으로써 시스템 붕괴를 막는다.

---

## 3. 학습하며 겪었던 문제점 & 에러

### 🔍 개념 재정립 및 심화 학습

#### 1) Resilience4j: 서킷 브레이커의 필요성
* MSA 환경에서 A 서비스가 B 서비스를 호출할 때, B 서비스가 장애라면 A 서비스까지 응답 대기 상태에 빠지는 '장애 전파' 위험이 있다.
  * `Fallback` 메서드를 제공하여 장애 발생 시 사용자에게 에러 대신 기본값이나 안내 메시지를 즉시 응답하도록 설계한다.

#### 2) API Gateway 필터의 동작 순서
* 프리 필터(Pre Filter)는 타겟 서비스로 요청이 넘어가기 전에 인증 등을 수행하고, 포스트 필터(Post Filter)는 서비스로부터 온 응답을 클라이언트에게 돌려주기 전에 마지막 처리를 수행한다.

---

## 4. 내일 학습 할 것은 무엇인지
- [ ] **MSA 보안 구성**: Gateway와 연동한 통합 인증/인가 체계 및 보안 프로토콜 학습
- [ ] **Spring Cloud Config**: 여러 마이크로서비스의 설정 파일(yml)을 중앙에서 관리하고 실시간 반영하는 법 익히기
- [ ] **분산 추적(Distributed Tracing)**: Zipkin과 Sleuth 등을 활용해 여러 서비스로 흩어지는 요청의 흐름을 한눈에 파악하기
- [ ] **이벤트 드리븐 아키텍처(EDA)**: 메시지 브로커(Kafka, RabbitMQ 등)를 활용하여 서비스 간 결합도를 낮추는 비동기 통신 방식 이해하기

---

#내일배움캠프 #단기Java #TIL #AOP #예외처리 #MSA #SpringCloud