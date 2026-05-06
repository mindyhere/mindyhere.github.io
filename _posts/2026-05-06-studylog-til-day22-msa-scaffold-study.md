---
title: "[내일배움캠프 TIL, Day 22] mini MSA : 우왕좌왕 껍데기 프로젝트1"
excerpt: "MSA 기본 아키텍처(aka 껍데기) 설계"

categories:
  - Studylog
tags:
  - [TIL, mini-project, MSA, skeleton-project]

permalink: /studylog/til-day22-msa-scaffold-study/

toc: true
toc_sticky: true

date: 2026-05-06
last_modified_at: 2026-05-06
---

## 1. 오늘 학습 키워드

* **MSA Infrastructure**: Eureka (Service Discovery), API Gateway, Config Server
* **Repository Strategy**: Multi-Repo vs Mono-Repo (Multi-Module)
* **통신 및 상태 관리**: Feign Client, Spring Boot Actuator, Health-check

---

## 2. 학습 내용 정리하기

### (1) 핵심 인프라 서비스

* Service Discovery (Eureka)
  * Eureka Server: 마이크로서비스들의 위치(IP, Port)를 동적으로 관리하는 '중앙 주소록' (@EnableEurekaServer)
  * Eureka Client: 서비스 기동 시 정보 등록(Register) 및 주기적 상태 보고(Heartbeat) 수행
  * 동작 원리: 서비스 이름 기반 통신으로 하드코딩된 주소 의존성 제거

* API Gateway
  * 단일 진입점: 외부 클라이언트의 모든 요청을 받아 적절한 서비스로 배달하는 단일 창구
  * 주요 기능: 유레카 연동 동적 라우팅, 인증/인가, 부하 분산, 로그 기록 등 공통 로직 처리

* Config Server (Spring Cloud Config)
  * 설정 중앙화: 각 서비스의 설정을 개별 서버가 아닌 한 곳에서 통합 관리 (@EnableConfigServer)
  * 동작 흐름: 서비스 구동 시 최우선적으로 설정 정보를 주입받아 환경별(dev, prod) 분리 용이

### (2) 리포지토리 구성 전략: Multi-Repo vs Multi-Module

* **Multi-Repo (멀티 리포지토리)**
  * **구조**: 서비스 하나당 하나의 독립된 Git 리포지토리를 운영.
  * **장점**: 서비스 간 독립성이 완벽히 보장되어 개별 배포 싸이클 운영이 자유롭고, 리포지토리별 접근 권한 관리가 용이함.
  * **단점**: 서비스가 늘어날수록 프로젝트 관리 포인트가 급격히 증가하며, 공통 코드(DTO, 유틸리티 등) 공유를 위해 별도의 라이브러리 배포 과정이 필요함.

* **Multi-Module / Mono-Repo (멀티 모듈 / 모노레포)**
  * **구조**: 단일 Git 리포지토리 내에서 Gradle/Maven의 멀티 모듈 기능을 활용해 여러 서비스를 관리.
  * **장점**: `common` 모듈을 통해 코드 재사용이 매우 간편하며, 단일 커밋으로 여러 서비스의 변경 사항을 원자적으로 관리 가능. 전체적인 시스템 구조 파악 및 IDE 통합 개발 환경 구축에 유리함.
  * **단점**: 리포지토리 규모가 커지면 빌드 및 테스트 시간이 증가할 수 있으며, 특정 서비스의 변경이 전체 CI 파이프라인에 부하를 줄 수 있음 (Selective Build 등의 최적화 필요).

<details>
<summary>🎨 Multi-Repo vs. Multi-Module (클릭하여 보기)</summary>

<img src="/assets/images/posts_img/msa_repo_strategy_comparison_v2.svg" width="600">

</details>

### (3) 서비스 간 통신 및 운영

* Feign Client (선언적 HTTP 클라이언트)
  * 인터페이스 정의만으로 타 서비스 API 호출 가능 (@EnableFeignClients)
  * 유레카 서버와 통합되어 서비스 이름 기반의 로드밸런싱 통신 수행

* Actuator (모니터링 및 상태 관리)
  * 애플리케이션 내부 상태를 HTTP 엔드포인트로 노출하는 라이브러리
  * 핵심 엔드포인트: /health(정상 작동 여부), /refresh(설정 변경 실시간 반영), /beans(등록된 빈 확인)
  * 필수 설정: exposure.include: "*" 설정을 통한 접근 권한 개방 필요

---

## 3. 학습하며 겪었던 문제점 & 에러

### '멀티모듈'과 '모노레포'의 관리 계층 차이를 명확히 구분하는 과정에서 어려움 발생

* **문제 인식**: 단순히 여러 프로젝트가 섞여 있는 상태를 넘어, 빌드 도구(Gradle)의 설정 계층과 버전 관리(Git)의 저장소 계층 간의 관계를 혼동함.  
👉🏻 '멀티모듈'은 빌드 시스템 상의 논리적 구분(Gradle)이며, '모노레포'는 소스 코드 관리(Git) 관점의 구분이다.  
👉🏻 MSA 구조 탐색을 주요 목적으로 하는 프로젝트이므로 멀티모듈+모노레포 전략을 채택하는 것이 적합하다고 최종 판단하였다
---

#내일배움캠프 #단기Java #TIL #MSA #미니프로젝트 #멀티모듈전략
