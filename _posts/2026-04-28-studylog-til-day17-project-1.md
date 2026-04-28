---
title: "[내일배움캠프 TIL, Day 16] 도메인 간 협업과 데이터 정합성 : 가게 평점 캐싱"
excerpt: "리뷰-가게 도메인 간의 JPA 더티 체킹을 활용한 캐싱 컬럼 업데이트 과정 정리"

categories:
  - Studylog
tags:
  - [TIL, Spring, Project1, JPA, saveAndFlush]

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

---

## 2. 학습 내용 정리하기

### 💡 가게 평균 평점 캐싱(Caching)과 데이터 동기화
* **캐싱의 필요성**: 목록 조회 시 N+1 문제를 방지하고 조회 성능을 높이기 위해 평점 데이터를 `Store` 테이블에 컬럼으로 관리한다.
* **Dirty Checking 활용**: 엔티티의 상태 변경만으로 Update 쿼리가 생성된다. 불필요한 `save()` 호출을 줄여 코드를 간결하게 유지하고 JPA의 메커니즘을 활용하도록 한다.
* **캡슐화**: `Setter` 대신 의미 있는 이름의 `public` 메서드를 통해 엔티티의 무결성을 보호하한다.

---

## 3. 학습하며 겪었던 문제점 & 에러

### "누가 어디를 구현해야할까?" ~~이래서 PM이 필요합니다.~~
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

---

## 4. Todo

- [ ] 프로젝트 마감 제출 전 최종 점검
- [ ] Dockerfile 작성 및 CI/CD 배포 스크립트 확인

---

#내일배움캠프 #단기Java #TIL #SpringBoot #입문프로젝트
