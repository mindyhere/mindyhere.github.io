---
title: "[내일배움캠프 TIL, Day 12] JPA Auditing : save() vs saveAndFlush()의 차이"
excerpt: "수정/삭제 로직에서 Auditing 필드가 DTO에 즉시 반영되지 않는 원인 분석과 JPA Flush 타이밍 정리"

categories:
  - Studylog
tags:
  - [TIL, Spring, Project1, DeliveryService, JPA, Auditing, PersistenceContext, DirtyChecking]

permalink: /studylog/til-day12-project-1/

toc: true
toc_sticky: true

date: 2026-04-21
last_modified_at: 2026-04-21
---

## 1. 오늘 학습 키워드
* **JPA Auditing**: `@CreatedDate`, `@LastModifiedDate` 등을 통한 자동 로깅
* **Persistence Context**: 영속성 컨텍스트의 쓰기 지연(Write-behind) 메커니즘
* **Flush Timing**: `PreUpdate` 이벤트가 발생하는 시점과 데이터 동기화
* **save() vs saveAndFlush()**: 즉시 반영 여부에 따른 기술적 차이
* **QueryDSL**

---

## 2. 학습 내용 정리하기

### 🛠️ JPA Auditing과 필드 자동 할당
* **개념**: 엔티티의 생성/수정 시간 및 생성/수정자를 자동으로 관리하는 기능.
* **작동 원리**: `AuditingEntityListener`가 엔티티의 생명주기 이벤트(`PrePersist`, `PreUpdate`)를 감지하여 값을 채워줌.

### 🔍 수정(Update) 시 Audit 필드가 null인 이유
* **상황**: 서비스 로직에서 엔티티 수정 후 바로 DTO로 변환하여 반환할 때 `updatedAt`이 비어있음.
* **원인**:
    1. JPA Auditing의 `PreUpdate`는 실제 DB에 데이터가 반영되는 **Flush** 시점에 실행됨.
    2. `@Transactional` 환경에서 Flush는 보통 트랜잭션이 커밋되는 **메서드 종료 시점**에 발생.
    3. 따라서 메서드 내부에서 DTO를 만드는 시점에는 아직 이벤트가 발생하지 않아 필드 값이 채워지지 않은 상태임.

  * cf. `save()` vs `saveAndFlush()`:

    | 구분           | save()                  | saveAndFlush()         |
    |:-------------|:------------------------|:-----------------------|
    | **반영 시점**    | 트랜잭션 커밋 시 (쓰기 지연)       | 호출 즉시 DB 반영 (Flush 강제) |
    | **Auditing** | 메서드 종료 전에는 필드 미갱신       | 호출 즉시 필드 값 갱신됨         |
    | **성능**       | 일괄 처리(Batching) 가능하여 유리 | 매번 DB 통신이 발생하여 상대적 불리  |

### 🔍 QueryDSL 도입 및 동적 쿼리 최적화

카테고리 검색 시 키워드 유무나 권한(관리자 여부)에 따른 동적 쿼리를 JpaRepository의 메서드 명명 규칙만으로 구현했으나, 
추후 다른 기능 구현 시 복잡한 동적쿼리를 생성할 가능성도 있어서 비교적 간단한 카테고리 search 구현에 도입해 연습해 보았다.

1) 의존성 설정 (build.gradle)  

```code
dependencies {
    // QueryDSL
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
}
``` 
   
2) 사용자 정의 Repository 구조 설계 : JPA와 QueryDSL을 함께 사용하기 위해 Custom 인터페이스 구조를 채택함.
* `CategoryRepository`: `JpaRepository`와 `CategoryRepositoryCustom`을 다중 상속받아 클라이언트에서 단일 창구로 사용.
* `CategoryRepositoryCustom`: QueryDSL을 사용할 메서드 선언.
* `CategoryRepositoryImpl`: `JPAQueryFactory`를 주입받아 실제 동적 쿼리 로직 구현.
    * cf. 관리자 검색 로직: `isAdmin` 플래그에 따라 `deletedAt.isNull()` 조건을 동적으로 조절 -> 일반 사용자는 활성 데이터만 보고 관리자는 삭제된 데이터까지 포함해 관리할 수 있도록 함.



```java
    // CategoryRepositoryImpl implements CategoryRepositoryCustom  
    @Override
    public Page<Category> searchCategories(CategorySearchDTO searchDTO, Pageable pageable) {
        // 데이터 조회 기본쿼리 생성
        JPAQuery<Category> query = queryFactory
            .selectFrom(qCategory)
            .where(
                containsKeyword(searchDTO.getKeyword()),
                isAccessible(searchDTO.getIsAdmin())
            )
            .orderBy(qCategory.createdAt.desc());

        // 페이징처리 여부
        if (pageable.isPaged()) {
            query.offset(pageable.getOffset())
                .limit(pageable.getPageSize());
        }

        List<Category> list = query.fetch();

        // 카운트 -> 페이징
        Long total = queryFactory
            .select(qCategory.count())
            .from(qCategory)
            .where(
                containsKeyword(searchDTO.getKeyword()),
                isAccessible(searchDTO.getIsAdmin())
            )
            .fetchOne();
        
        return new PageImpl<>(list, pageable, total);
    }

    private BooleanExpression containsKeyword(String keyword) {
        return StringUtils.hasText(keyword) ? qCategory.name.contains(keyword) : null;
    }

    private BooleanExpression isAccessible(Boolean isAdmin) {
        // 관리자가 아닌 일반사용자 -> 삭제되지 않은 데이터만 조회
        if (isAdmin == null || !isAdmin) {
            return qCategory.deletedAt.isNull();
        }

        // 관리자 -> 모든 데이터 조회가능(조건 없음)
        return null;
    }
```

---

## 3. 학습하며 겪었던 문제점

### 💡 "꼭 saveAndFlush()를 써야 할까?"
* **상황**: 수정 직후 DTO에 `updatedAt`이 담기지 않아 `saveAndFlush()` 도입을 고민.
* **해결**:  
  비즈니스 요구사항상 응답값에 수정 시각이 반드시 포함되어야 한다면 `saveAndFlush()`를 사용해야한다.  
  하지만 단순히 DB 저장만 잘 되면 되고, 응답에서 `id` 정도만 확인해도 된다면 **변경 감지(Dirty Checking)** 에 맡기고 `save()`나 `flush`를 호출하지 않는 것이 JPA의 설계 의도에 더 부합한다고 볼 수 있다.  
  데이터는 DB에 정상적으로 저장되므로, 이후 조회(`GET`) 시에는 업데이트된 정보를 확인할 수 있다.

---

## 4. Todo
- [x] **Category**: categoryId 로 상세정보 조회 api 구현, api 기능 구현 단위 테스트 -> 정상동작 확인
- [ ] **Area**: Service 구현

---

#내일배움캠프 #단기Java #TIL #SpringBoot #입문프로젝트 #JPA #Auditing #영속성컨텍스트
