---
title: "[내일배움캠프 TIL, Day 16] 프로젝트 막바지 : 요구사항 충족 여부 검토 및 배포 자동화"
excerpt: "JPA 더티 체킹을 활용한 캐싱 컬럼 업데이트 과정 정리 및 도커 기반 CI/CD 파이프라인의 원리 이해"

categories:
  - Studylog
tags:
  - [TIL, Spring, Project1, JPA, saveAndFlush, Docker, CI/CD, GitHubActions]

permalink: /studylog/til-day17-project-1/

toc: true
toc_sticky: true

date: 2026-04-28
last_modified_at: 2026-04-28
---

## 1. 오늘 학습 키워드

* **flush() & saveAndFlush()**
  * `flush()`: 영속성 컨텍스트의 변경 내용을 DB에 즉시 동기화
  * `saveAndFlush()`: `save()`와 `flush()`를 한 번에 수행하는 메서드. 엔티티를 영속화하자마자 DB에 쿼리를 날려 즉각적인 데이터 정합성이 필요할 때 사용한다.
* **JPA Persistence Context**: Dirty Checking(더티 체킹)을 통한 자동 변경 감지
* **도커와 GitHub Actions로 배포자동화**
  * Docker & Dockerfile: 서버 환경의 표준화와 레시피화 
  * GitHub Actions: 배포 과정을 대신 수행하는 자동화 로봇

---

## 2. 학습 내용 정리하기

### 💡 가게 평균 평점 캐싱(Caching)과 데이터 동기화

* **캐싱의 필요성**: 목록 조회 시 N+1 문제를 방지하고 조회 성능을 높이기 위해 평점 데이터를 `Store` 테이블에 컬럼으로 관리한다.
* **Dirty Checking 활용**: 엔티티의 상태 변경만으로 Update 쿼리가 생성된다. 불필요한 `save()` 호출을 줄여 코드를 간결하게 유지하고 JPA의 메커니즘을 활용하도록 한다.
* **캡슐화**: `Setter` 대신 의미 있는 이름의 `public` 메서드를 통해 엔티티의 무결성을 보호하한다.

### 🤖 Docker와 CI/CD: 배포 자동화

* **Dockerfile**: 사람이 터미널에 일일이 치던 환경 세팅 명령어(Java 설치, 파일 복사, 실행 등)를 파일로 박제한 것으로, 이 파일만 있으면 어떤 서버에서든 동일한 실행 환경을 보장한다.

  ```code
  # FROM (OS/언어) : 기본 환경 설정. JAR 실행을 위한 이미지
  FROM eclipse-temurin:17-jre
  
  # WORKDIR (디렉토리) : 작업 디렉토리 생성/이동. 도커 내부에서 명령어를 실행할 기본 폴더를 /app으로 지정한다.
  WORKDIR /app
  
  # COPY (파일 복사) : 빌드된 jar 복사
  COPY build/libs/*.jar app.jar
  
  # EXPOSE (포트) : 포트 오픈
  EXPOSE 8080
  
  # ENTRYPOINT (실행) : 자동실행 명령
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```

* **Docker Compose**: 여러 개의 컨테이너(App, DB, Redis 등)를 하나의 파일(docker-compose.yml)로 정의하여 한 번에 띄우고 관리하게 해준다.
  ```code
  # 1. 설정 파일의 문법 버전 (3.8 규격 사용)
  version: "3.8"
  
  # 2. 실행할 컨테이너(서비스)들의 집합
  services:
    # [App]: 우리가 만든 스프링 부트 애플리케이션
    app:
      # 빌드된 이미지가 저장된 창고(ECR)의 주소에서 최신 이미지를 가져옴
      image: ${ECR_REPOSITORY_URL}:latest
      # 도커 내부에서 관리할 컨테이너 이름 설정
      container_name: delivery-app
      # 외부 포트 8080과 컨테이너 내부 포트 8080을 연결 (L:R)
      ports:
        - "8080:8080"
      # DB 접속 정보나 비밀번호 등 민감한 정보를 담은 .env 파일을 읽어옴
      env_file:
        - .env
      # 서버가 예기치 않게 꺼지더라도, 수동으로 끄기 전까지는 항상 자동 재시작
      restart: unless-stopped
  
    # [Nginx]: 외부 사용자가 가장 먼저 만나는 웹 서버 (대문 역할)
    nginx:
      # 공식 Nginx 최신 이미지를 가져옴
      image: nginx:latest
      container_name: nginx-proxy
      # 일반적인 HTTP 접속 포트인 80번을 열어줌
      ports:
        - "80:80"
      # 내 컴퓨터(EC2)의 설정 폴더와 컨테이너 내부의 설정 폴더를 연결 (동기화)
      volumes:
        - ./nginx:/etc/nginx/conf.d
      # 'app' 서비스가 먼저 완벽하게 실행된 뒤에 Nginx를 실행하도록 순서 보장
      depends_on:
        - app
      restart: unless-stopped
  ```
* **GitHub Actions**: 개발자가 코드를 push하면 자동으로 빌드, 테스트, 배포를 수행하는 자동화 엔진.  
  * 로컬 PC나 서버가 아닌, GitHub에서 제공하는 가상 서버(Runner)에서 빌드가 일어나기 때문에 내 컴퓨터 사양이 낮거나 서버(EC2)가 프리티어여도 안정적인 빌드가 가능하다.
  * 스크립트 위치: 반드시 `.github/workflows/` 폴더 내에 `.yml` 파일로 존재해야 GitHub이 인지하고 자동화를 시작한다.

---

## 3. 학습하며 겪었던 문제점 & 에러

### 1. "누가 어디를 구현해야할까?" ~~이래서 PM이 필요한가보다~~
리뷰/가게 도메인 담당 두 사람 모두 까먹고 있던 요구사항을 또 발견했다. 왜 이런건 꼭 한 밤 중에 발견하는 걸까🤣
* **요구사항**: 리뷰가 생성/수정/삭제(CUD)될 때마다 해당 가게의 평균 평점을 재계산하여 Store 테이블의 캐싱 컬럼(average_rating)을 업데이트해야 함.  
👉🏻 가게 목록 조회 시 평균 평점 노출 (N+1 문제 방지) + 데이터 정합성 유지

* **R&R 분리**: 처음엔 내가 Store 담당이니 계산부터 업데이트까지 다 해야 하나?- 하는 생각부터 들었지만, 도메인 간 협업이 필요한 부분이라고 생각을 고쳐먹고 코멘트를 달았다.
  * 평균평점 **계산** 은 `ReviewRepository`에서 `AVG` 쿼리를 실행해 결과값을 산출하여 전달한다.
  * `Store`는 전달받은 값을 엔티티에 반영하는 **헬퍼 메서드** 를 제공하여 자신의 필드를 업데이트 한다.
  `setter`를 열어 단순히 값을 주입하는 게 아니라, 엔티티 내부에 방어 로직(null 체크, 반올림 등)을 포함한 메서드를 public으로 열어두어 객체 스스로 자신의 무결성을 지키게 한다.
    
    ```java
    // Store (Entity)
    public void updateAverageRating(Double newAvg) {
        // 리뷰가 0개 되어 null이 올 경우를 방어
        double value = (newAvg == null) ? 0.0 : newAvg;
        // 소수점 첫째 자리까지 반올림 처리
        this.averageRating = Math.round(value * 10) / 10.0;
    }
    
    // ReviewService
    private void updateStoreAverageRating(Store store) {
        Double newAverageRating = reviewRepository.findAverageRatingByStoreId(store.getId());
        store.updateAverageRating(newAverageRating); // <- 내가 제공하는 헬퍼메서드는 여기서 호출된다.
    }
    ```

* **팀원의 아이디어와 최적화**: 코멘트로 팀원에게 노티할 때만 해도 Order를 거쳐 Store를 찾아야 한다고 생각했으나, 후에 수정한 소스코드를 보니 리뷰 생성 시점에 Store를 직접 참조하도록 한 것을 확인했다. 
  1. 참조 간소화: `Review` entity를 다시 확인해보니 `Store`를 직접 참조하고 있어 연산 과정울 단순화 할 수 있는 구조였다. (`review.getStore()` 로 즉시 접근 가능) 
  2. 동기화 & 정합성 보장: 리뷰 삭제 직후 `flush()`를 호출하여, 방금 지운 리뷰가 평균 계산에 포함되지 않도록 정합성 확보한 것을 확인할 수 있었다. 
  3. 더티 체킹: 별도의 store.save() 없이 @Transactional 내에서 엔티티 메서드 호출만으로 데이터베이스에 반영한다.

### 2. "기억을 헤집으며 배포 CI/CD 다시 공부하기"

2년 전 취준 시절에 훈련과정 다 끝나고 혼자서 구글링에 의존해 파이널 팀프로젝트를 AWS EC2에 수동 배포해 본 경험이 있다.
클라우드 컴퓨팅이고 뭐고 AWS EC2 인스턴스 생성, Docker 컨테이너/이미지 등등 용어도 낯설고 개념도 모르면서 localhost말고 웹에 띄워보고 싶다는 욕심 하나로 시작했다가 
당초 목표했던 배포 자동화까지는 못 가고 수동배포에서 그쳤다.  
그 기록이라도 남겨놨으면 좋았을 것을... 그때도 했던 개고생을 다시 마주할 줄이야 😱

이번에는 다행히 팀원들과 함께 하는 중이어서 나는 정말 그림만 그렸다. 머릿속으로 상상만 해 본 수준이다.  
그럼에도 불구하고 흐릿한 기억 속의 경험과 팀원들의 작업물(Dockerfile, docker-compose, 배포 워크플로우 스크립트)을 바탕으로 속성으로 공부하면서, 미미하게나마 배포 과정에 대한 이해를 높일 수 있었다. 

* **과거의 나**: 기억은 잘 안나는데, 내가 필요한/작업한 이미지를 도커 허브에 올리고 내리고 했던 것 같다. ~~분명 도커를 쓰긴 썼는데.. docker run 어쩌고 해서 실행도 했던 것 같은데..~~  
  확실한 것 하나는 터미널로 코드 쳐가며 완전 수동 배포했다는 것.(프리티어의 메모리 부족과 싸우며...!) 
* **현재의 팀 프로젝트**: 모든 과정을 `Dockerfile`에 정의하고, 스크립트로 정의한 후 `GitHub Actions`라는 로봇에게 맡긴다.
  1. 빌드 자동화: GitHub Actions가 코드를 감지해 자동으로 빌드하고 도커 이미지를 굽는다. 
  2. 이미지 저장: 생성된 이미지는 ECR(이미지 창고)에 보관되어 버전 관리가 된다.
  3. 배포 자동화: 서버에 직접 접속할 필요 없이, 스크립트가 EC2에 접속해 `docker-compose pull & up` 명령을 내린다.

CI/CD 자동화로 편리함 뿐만아니라, 개발-테스트-운영 환경의 일관성을 유지하고 휴먼 에러를 방지하는 **"신뢰의 시스템"** 을 만드는 과정임을 배웠다.

---

## 4. Todo

- [ ] 프로젝트 마감 제출 전 최종 점검
- [ ] Dockerfile 작성 및 CI/CD 배포 스크립트 공부
- [ ] 발표자료에 필요한 내용 보완

---

#내일배움캠프 #단기Java #TIL #SpringBoot #입문프로젝트
