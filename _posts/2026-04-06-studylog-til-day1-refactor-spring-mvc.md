---
title: "[내일배움캠프 TIL, Day1] 기본기 재정립: 빌드 도구와 Spring MVC 동작원리"
excerpt: "Gradle과 HTTP 통신 규약, Spring MVC의 핵심인 DispatcherServlet의 동작 흐름을 재정립해보자"

categories:
  - Studylog
tags:
  - [GitHub, Git]

permalink: /studylog/til-day1-refactor-spring-mvc/

toc: true
toc_sticky: true

date: 2026-04-06
last_modified_at: 2026-04-06
---

## 1. 오늘 학습 키워드
* **Gradle**: 빌드 자동화 도구 및 라이브러리 관리
* **Spring vs Spring Boot**: 프레임워크의 개념과 자동 설정의 차이
* **Web Server vs WAS**: 정적/동적 컨텐츠 처리 서버의 차이
* **HTTP 프로토콜**: 웹 통신의 기본 규약 및 메시지 구조
* **Spring MVC & DispatcherServlet**: 스프링의 웹 처리 패턴과 핵심 서블릿 동작 원리

---

## 2. 학습 내용 정리하기

### 🏗️ 빌드 도구와 서버 기초
* **Gradle**: 프로젝트에 필요한 외부 라이브러리를 자동으로 가져오고, 소스 코드를 실행 가능한 파일로 빌드해주는 도구
* **Spring vs Spring Boot**: Spring이 개발자가 일일이 설정해야 하는 조립식 키트라면, Spring Boot는 복잡한 설정을 미리 완료해두어 개발을 빠르게 시작할 수 있도록 도와준다.
* **Web Server vs WAS**:
   * **Web Server**: HTML, CSS, 이미지 같은 **정적 리소스**를 단순히 전달하는 역할 (예: Nginx, Apache)
   * **WAS (Web Application Server)**: DB 연동이나 비즈니스 로직을 수행하여 상황에 맞는 **동적 리소스**를 생성하고 전달 (예: Tomcat)


### 🎨 Spring MVC 패턴과 DispatcherServlet
MVC 패턴은 역할을 분담하여 유지보수를 쉽게 만듭니다.
* **Model**: 데이터와 비즈니스 로직
* **View**: 사용자에게 보여지는 화면
* **Controller**: 요청을 받아 모델과 뷰를 잇는 역할

**DispatcherServlet 동작 흐름:**

1. 모든 요청을 **DispatcherServlet**이 가장 먼저 받는다. (프론트 컨트롤러).
2. **Handler Mapping**을 통해 요청을 처리할 컨트롤러를 찾는다.
3. **Controller**가 비즈니스 로직을 수행하고 결과(Model)와 뷰 이름을 반환한다.
4. **View Resolver**가 실제 뷰 파일을 찾아 화면을 렌더링하여 사용자에게 응답한다.

### 🌐 HTTP 메시지 구조 및 상태 코드
**메시지 구조:**
* **Header (메타 데이터)**: 브라우저 정보, 데이터 형식 등 '데이터에 대한 설명'
* **Payload (실제 데이터)**: 서버와 주고받는 실제 '내용물'
* **참고**: GET 메서드는 바디(Payload)가 없는 것이 약속이며, URL 뒤의 쿼리 스트링을 사용

**HTTP 상태 코드:**
* **1xx (Informational)**: 요청을 받았으며 처리가 계속 진행 중
* **2xx (Success)**: 요청 성공
* **3xx (Redirection)**: 요청 완료를 위해 추가 동작(페이지 이동 등)이 필요
* **4xx (Client Error)**: 클라이언트가 요청을 잘못 보냄 (예: 404 Not Found)
* **5xx (Server Error)**: 서버 쪽에서 문제가 발생

---

## 3. 학습하며 겪었던 문제점 & 에러

### 🔍 문제 & 에러 정의
- **상황**: 기존 회사 프로젝트를 위해 `~/.zprofile`에 **Java 8(Zulu 8)** 경로가 고정(`export JAVA_HOME`)되어 있어, 부트캠프 프로젝트 빌드 시 버전 충돌 발생.
- **원인**: 터미널 실행 시마다 Java 8이 기본값으로 잡혀 있어, 상위 버전이 필요한 스프링 부트 프로젝트가 정상적으로 동작하지 않음.

### 🧪 내가 한 시도
- `echo $JAVA_HOME` 명령어로 현재 설정된 경로 확인.
- `nano ~/.zprofile`을 통해 기존 설정 파일의 내용을 점검하고, 고정된 경로가 문제임을 파악.

### ✅ 해결 방법
- **환경변수의 동적 관리**: 고정된 `export` 방식을 지우고, 필요할 때마다 버전을 바꿀 수 있도록 `alias`를 설정함.
- **설정 내용**:

  ```zsh
  # Java 버전 전환을 위한 alias 설정
  alias setJava8='export JAVA_HOME=$(/usr/libexec/java_home -v 1.8); java -version'
  alias setJava17='export JAVA_HOME=$(/usr/libexec/java_home -v 17); java -version'

  # 부트캠프를 위해 기본값은 17로 설정
  export JAVA_HOME=$(/usr/libexec/java_home -v 17)
  ```
  
---

## 4. 내일 학습 할 것은 무엇인지
- [ ] **JPA 영속성 관리 및 환경 설정**: 영속성 컨텍스트의 원리, 엔티티 생명주기, 프로젝트 내 JPA 의존성 및 기본 설정 실습
- [ ] **객체-DB 매핑 및 SQL 복습**

---

#내일배움캠프 #단기Java #TIL