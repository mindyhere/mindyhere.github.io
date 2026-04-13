---
title: "[내일배움캠프 TIL, Day5] MSA 설정 관리와 분산 추적 및 비동기 통신"
excerpt: "중앙 집중식 설정 관리(Config Server), 분산 추적(Zipkin), 이벤트 드리븐 아키텍처 핵심 정리"

categories:
  - Studylog
tags:
  - [TIL, MSA, ConfigServer, Zipkin, EDA]

permalink: /studylog/til-day5-msa-advanced/

toc: true
toc_sticky: true

date: 2026-04-10
last_modified_at: 2026-04-10
---

## 1. 오늘 학습 키워드
* **Spring Cloud Config**: 분산 환경에서의 중앙 집중식 설정 관리
* **Security & JWT**: 게이트웨이 기반의 보안 구성 및 무상태(Stateless) 인증
* **Distributed Tracing**: Micrometer와 Zipkin을 활용한 서비스 간 요청 흐름 추적
* **Event-Driven Architecture**: 이벤트를 통한 서비스 간 느슨한 결합(Loose Coupling) 실현

---

## 2. 학습 내용 정리하기

### ⚙️ Spring Cloud Config: 설정의 중앙 집권화
MSA 환경에서 수많은 서비스의 `yml` 설정을 한 곳에서 관리하는 프레임워크
* **환경별 구성**: `application-dev.yml`, `application-prod.yml` 등으로 분리하여 개발/운영 환경에 맞는 설정을 클라이언트에 전달함.
* **중앙 관리**: Git이나 파일 시스템(Native)을 저장소로 사용하여 설정 변경 시 모든 서비스에 일괄 적용 가능함.
* **실시간 반영**: `@RefreshScope`를 사용하면 애플리케이션 재시작 없이도 변경된 설정을 반영할 수 있음.

### 🔐 보안 구성 및 JWT (JSON Web Token)

* **게이트웨이 보안**: 모든 외부 요청을 Gateway에서 인증/인가 처리하고, 내부 서비스는 게이트웨이의 방화벽 안쪽에서 안전하게 통신함.
* **무상태성**: 토큰 자체가 정보를 들고 있으므로 서버가 세션 상태를 저장할 필요가 없어 확장성이 뛰어남.
* **무결성 보장**: 헤더-페이로드-서명 구조 중 '서명'을 통해 토큰의 위조 여부를 검증함.

### 🕵️ 분산 추적 및 로깅 
여러 서비스로 흩어지는 요청을 한눈에 파악하기 위한 가시성 확보 기술

* **주요 개념**:
  - **Trace**: 하나의 요청이 시작부터 끝까지 거치는 전체 경로.
  - **Span**: 전체 경로 중 특정 서비스 내에서의 개별 작업 단위.
    * cf. Micrometer, Zipkin

### ✉️ 이벤트 드리븐 아키텍처 (EDA)
상태 변화를 '이벤트'라는 메시지로 발행하여 비동기적으로 처리하는 설계 스타일
* **핵심 요소**: 이벤트 소스(발행자), 이벤트 버스(중개자/Kafka 등), 이벤트 핸들러(수신자).  
  * 이벤트 생산자 (Producer): "주문이 완료됨"과 같은 상태 변화(이벤트)를 감지하고 메시지를 발행
  * 이벤트 채널 (Message Broker): 발행된 이벤트를 수집하고 필요한 곳에 전달하는 중개자
  * 이벤트 소비자 (Consumer): 채널에 올라온 이벤트를 읽어서 자신의 업무를 수행
* **장점**: 서비스 간 강한 종속성을 제거하여 **느슨한 결합**을 실현함. 호출하는 쪽에서 응답을 기다리지 않아도 되므로 시스템 응답성이 향상됨.

---

## 3. 학습하며 겪었던 문제점 & 에러

#### 1) Config Server 연결 오류 (Connection Refused)
* **상황**: Product Service 기동 시 포트 8888번 연결 오류 발생.  
 👉🏻 Config Server가 먼저 실행되어야 클라이언트가 설정을 받아올 수 있다. (실행 순서 : Config -> Eureka -> Client)

#### 2) 예상과 다른 포트 번호 할당 (랜덤 포트)
* **상황**: 설정 파일에는 19083으로 적었으나 실제로는 랜덤 포트로 실행됨.  
  👉🏻 클라이언트 설정에서 `spring.application.name`이 설정 서버의 파일명과 일치해야 한다. 오타주의!

---


#내일배움캠프 #단기Java #TIL #SpringCloudConfig #분산추적 #이벤트드리븐