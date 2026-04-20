---
title: "[내일배움캠프 TIL, Day 11] 입문 프로젝트 작업 1일차: JPA 영속성 메커니즘과 도메인 연관관계 설계"
excerpt: "배달 주문 플랫폼 백엔드 시스템 구축 시작 : Store, Category, Area"

categories:
  - Studylog
tags:
  - [TIL, Spring, Project1, DeliveryService]

permalink: /studylog/til-day11-project-1/

toc: true
toc_sticky: true

date: 2026-04-20
last_modified_at: 2026-04-20
---

## 1. 오늘 학습 키워드
* **JPA 연관관계 설정**과 **영속성 컨텍스트**의 이해 및 적용

---

## 2. 학습 내용 정리하기

### 🌐 도메인 연관관계 설계 (N:1 단방향 매핑)  

음식점(Store)을 중심으로 Category와 Area가 연결되는 구조.

* **다대일(@ManyToOne) 단방향 매핑**  
  * 하나의 카테고리/지역에 여러 음식점이 속하므로 Store 측에서 @ManyToOne 관계를 맺음. 
* **지연 로딩(FetchType.LAZY) 전략**  
  * 연관된 엔티티를 실제 사용하는 시점에 쿼리가 실행되도록 설정하여 성능 최적화 및 N+1 문제 예방
* **외래 키 매핑 (@JoinColumn)**  
  * `@JoinColumn(name = "category_id")` 등을 통해 DB 테이블 상의 물리적인 외래 키 명칭을 명시적으로 지정

### 🔍 JPA 영속성 메커니즘: 변경 감지(Dirty Checking)

* **영속 상태(Managed)**
    * `@Transactional` 범위 내에서 조회된 엔티티는 영속성 컨텍스트가 관리하는 상태가 됨.
* **Dirty Checking**
    * 트랜잭션이 끝나는 시점에 엔티티의 초기 상태(스냅샷)와 현재 상태를 비교하여, 변경된 부분이 있다면 자동으로 `UPDATE` 쿼리를 실행함.
 
> **프로젝트 적용? Category Service(CRUD)**  
> updateCategory: 별도의 `save()` 없이 필드 수정만으로 DB 동기화할 수 있다.  
> deleteCategory: `deletedAt` 필드 수정만으로 소프트 딜리트(Soft Delete) 구현.  
> findActiveCategory: Service 클래스 내에 private 메서드를 정의해 "ID 조회 + 삭제 여부 확인 + 예외 처리" 로직을 한 곳에서 관리하도록 구현

---

## 3. Todo
- [ ] **Category**: categoryId 로 상세정보 조회 api 구현, api 기능 구현 단위 테스트
- [ ] **Area**: Service 구현

---

#내일배움캠프 #단기Java #TIL #SpringBoot #입문프로젝트