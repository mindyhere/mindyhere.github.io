---
title: "[내일배움캠프 TIL, Day 9] 모놀리식 아키텍쳐 프로젝트 설계: API 명세와 소프트 딜리트 전략"
excerpt: "HTTP 메서드 활용, 소프트 딜리트 전략 논의, 그리고 모놀리식 패키지 구조 설계"

categories:
  - Studylog
tags:
  - [TIL, Spring, RestAPI, Monolithic]

permalink: /studylog/til-day9-project-design/

toc: true
toc_sticky: true

date: 2026-04-16
last_modified_at: 2026-04-16
---

## 1. 오늘 학습 키워드
* **RESTful API**: 리소스 중심의 인터페이스 설계 및 HTTP 요청 메서드 활용
* **Soft Delete**: 데이터 물리 삭제 대신 상태값을 통한 논리 삭제 전략
* **ERD & API Specification**: 데이터 구조 및 서비스 인터페이스 정의
* **Monolithic Architecture**: 단일 코드 베이스에서의 효율적인 패키지 구조

---

## 2. 학습 내용 정리하기

### 🌐 HTTP 요청 메서드 정리 및 소프트 딜리트(Soft Delete) 전략

* **GET**: 리소스 조회
* **POST**: 리소스 생성
* **PUT**: 리소스 전체 수정 (덮어쓰기)
* **PATCH**: 리소스 일부 수정 (상태변경)
* **DELETE**: 리소스 삭제

#### 💬 논의사항: 소프트 딜리트에는 어떤 메서드가 적합할까?
데이터를 실제로 삭제하지 않고 `deleted_at` 등의 컬럼을 업데이트하는 '소프트 딜리트'의 경우, 어떤 것으로 통일하는 것이 좋을지 논의하였다.  
우리 팀에서는 상태 변경이라는 의미에 중점을 두고 최종적으로는 PATCH로 통일하는 것으로 결론내렸다.  
다만 튜터님의 해설에 DELETE로 통일되어 있어서 구글링을 해보니 역시 이런 건 케이스 바이 케이스 인 것 같다. 

1. **PATCH**: 리소스의 특정 필드(`is_deleted` 등)만 수정하는 행위에 중점을 둔 케이스. POST나 PUT을 쓰는 경우도 이와 같은 맥락으로 볼 수 있겠다.  
2. **DELETE**: 클라이언트 입장에서는 '삭제'를 요청하는 것이므로, 내부 구현이 물리 삭제인지 논리 삭제인지와 무관하게 DELETE를 사용하여 인터페이스의 직관성을 높일 수 있다.

> **참고할 만한 웹 페이지**
> * [Microsoft REST API 가이드라인 - 삭제 및 수정](https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design)
> * [Baeldung: Soft Delete in Spring JPA](https://www.baeldung.com/spring-jpa-soft-delete)

### 📊 프로젝트 설계 및 진행 프로세스

1. **ERD (Entity Relationship Diagram)**: 서비스의 핵심 엔티티를 도출하고 관계(1:N, N:M)를 정의
2. **API 명세**: Endpoint, Request/Response 형식, 상태 코드를 정의하여 프론트엔드와 소통의 기준을 정의한다.  
   👉🏻 이번 프로젝트에선 백엔드만 구현할 예정인지라 프론트엔드와 소통할 일은 없지만, 실무에서 일할 때도 그렇고 API 명세를 잡고 가는 것이 개발 속도나 상호간에 소통에 큰 도움이 되니 힘들어도 꼭 필요한 작업인 것 같다.  
   이름만 들어본 swagger나 javadoc으로 자동화도 할 수 있다고 하니, 나중에 시간을 내서라도 좀 더 찾아볼 필요가 있는 듯!
3. **기능 명세 & 태스크 분담**

---

### 🏗️ 프로젝트 아키텍처 논의 : 모놀리식 & 계층형 설계

#### 1) 모놀리식 아키텍처 (Monolithic)
* 하나의 애플리케이션에 모든 모든 비즈니스 로직이 통합되어 있는 구조
* 초기 개발 속도를 확보하고, 서비스 간 통신 설정의 복잡도를 줄여 비즈니스 로직 구현에 집중하기에 용이하다. 관리가 단순하다.

#### 2) 계층형 아키텍처 (Layered)
* 애플리케이션을 역할에 따라 수평적인 계층으로 분리하는 방식. 
* **관심사의 분리**: 컨트롤러는 HTTP 요청 처리만, 서비스는 비즈니스 로직만, 리포지토리는 DB 접근만 담당하도록 하여 코드의 독립성을 확보한다.

#### 3) 📂 패키지 구조 (Domain-Driven)  
**도메인(Product) 패키지 내부에서 계층을 나누는 방식**을 선택, 각 도메인 패키지 내부를 **Controller - Service - Repository**의 3계층으로 분리하기로 논의하였다.  
* ⁉️ 아래와 같이 패키지 구조를 수정한 이유
  * 모놀리식의 장점을 살리면서도, 향후 특정 도메인만 MSA로 분리해야 할 때 '응집도'를 높여주는 전략적 선택
  * 예시로 주어진 패키지 구조는 뎁스가 깊고 세분화된 것 같다는 의견으로 모여서 개별 도메인 안에 같은 레벨에 계층별로 패키지를 두기로 단순화

```text
src/main/java/com.example.project
 ├── global (ex. 공통 설정: Security, Config, Error, Utils...)
 └── domain
      ├── product (상품 도메인)
      │    ├── controller
      │    ├── service
      │    ├── repository
      │    ├── entity
      │    └── dto
      └── user (사용자 도메인)
           ├── controller
           ├── service
           ├── entity
           └── dto
```

---

#내일배움캠프 #단기Java #TIL #SpringBoot #프로젝트