---
title: "[내일배움캠프 TIL, Day 15] 서비스 구조 시각화 및 도메인 서비스의 캡슐화"
excerpt: "가시성 확보를 위한 아키텍처 다이어그램을 작성해보고 서비스 책임 캡슐화에 대해 고민해보는 시간"

categories:
  - Studylog
tags:
  - [TIL, Spring, Project1, DeliveryService, InfraArchitecture, REST_API, SpringSecurityConfig]

permalink: /studylog/til-day15-project-1/

toc: true
toc_sticky: true

date: 2026-04-24
last_modified_at: 2026-04-24
---

## 1. 오늘 학습 키워드

* **제한된 자원을 고려한 인프라 설계** : AWS 프리티어의 낮은 RAM에서의 서버 배치 고민?!
* **도메인 서비스 캡슐화** : 서비스 간 직접적인 리포지토리 참조 지양 및 책임 분리
* **메서드 보안** : `@EnableMethodSecurity`와 `@PreAuthorize`의 작동 원리

---

## 2. 학습 내용 정리하기

### 🏗️ 인프라 설계 그려보기

* **고민** : AWS 프리티어(t3.micro)는 RAM이 1GB로 매우 제한적인데, DB가 이미 EC2에 설치되어 있는 상황이다.  
Spring Boot와 PostgreSQL을 하나의 인스턴스에 둘 것인가(Local Native), 아니면 DB를 분리하는게 나을까?
  * Native(Self-Managed): EC2 내부에 직접 설치 시 네트워크 지연은 최소화되나 RAM 점유율 싸움이 발생함.
  * RDS(Managed Service): 인스턴스 RAM은 아낄 수 있지만, 튜터님 의견으로는 경우에 따라 DNS 비용이 발생할 수 있다고 한다.  
  또 잠깐 찾아보니 프리티어 RDS 자체의 성능 한계와 네트워크 Latency가 발생할 수도 있다고... 👉🏻 EC2를 두개 놓는 것은 어떨까🤔? 

<details>
<summary>🎨 직접 그린 인프라 초기 설계도 (클릭하여 보기)</summary>

<img src="/assets/images/posts_img/infra-sketch1.jpg" width="600" alt="[초기 설계도-RDS]">
<br>
<img src="/assets/images/posts_img/infra-sketch2.jpg" width="600" alt="[초기 설계도-Native]">

</details>

* **결정 및 시각화** : 
  1. 현재는 학습단계이고 각자의 경험치가 너무 다르니 RDS에 대해 더 찾아보고 DNS로 인한 비용을 피할 수 없다면 EC2를 2개로 구축해서 각각 DB와 애플리케이션 서버를 두도록 시도해보리고 했다.
  2. 그마저도 실패한다면 단일 EC2 내부에 **Docker(Spring Boot)** 와 **Native(PostgreSQL-이미 설치된 상태이니까!)** 를 공존시키는 아키텍처를 설계했다.

### 🛠️ 도메인 서비스 간 협력과 캡슐화
* **문제점** : `StoreService`가 `CategoryRepository`와 `AreaRepository`를 직접 참조하여 삭제 여부나 활성 상태를 일일이 확인하고 있었음.   
-> 해당 도메인에서 처음에 `private` 으로 메서드를 만들다보니 로직이 중복되고 있었다는 걸 나중에 깨달았다. 때문에 도메인 간 결합도를 높이고 로직 중복을 야기하는 문제가 발생했다.
* **개선** : 
    1. 각 도메인 서비스(`CategoryService`, `AreaService`)에 유효한 엔티티를 조회하고 예외를 던지는 `public` 메서드(`findCategoryById`, `findActiveAreaById`)를 정의.
    2. `StoreService`는 리포지토리 대신 **서비스를 주입받아 호출**하도록 한다.

```java
    // 예시
    // ---- CategoryService  
    //delete
    @Transactional
    public CategoryResponseDTO deleteCategory(UUID categoryId, String username) {
        Category category = findCategoryById(categoryId);
        category.delete(username);
    
        return CategoryResponseDTO.from(category);
    }
    
    // ---- AreaService    
    // 삭제되지 않은 데이터 조회
    @Transactional(readOnly = true)
    public Area findAreaById(UUID areaId) {
        return areaRepository.findByIdAndDeletedAtIsNull(areaId)
            .orElseThrow(() -> new CustomException(ErrorCode.AREA_NOT_FOUND));
    }
    
    // 유효한(삭제되지 않고 활성화된) 운영 지역 조회
    @Transactional(readOnly = true)
    public Area findActiveAreaById(UUID areaId) {
        return areaRepository.findByIdAndIsActiveTrueAndDeletedAtIsNull(areaId)
            .orElseThrow(() -> new CustomException(ErrorCode.AREA_NOT_FOUND));
    }

    // ---- StoreService    
    @Transactional
    public StoreResponseDTO createStore(StoreRequestDTO requestDTO, String username) {
        // Category 삭제 여부 확인
        Category category = categoryService.findCategoryById(requestDTO.getCategoryId());
        // Area 활성화여부 확인
        Area area = areaService.findActiveAreaById(requestDTO.getAreaId());

        User owner = userRepository.findById(username)
                .orElseThrow(() -> new CustomException(ErrorCode.UNAUTHORIZED_ACCESS));
        
        // 생략...
    }
```

### API 설계: RequestMapping과 경로 캡슐화

* **고민** : Menu 도메인은 API 설계상 URL 경로가 `/api/v1/menus/*` 와 `/api/v1/stores/*`를 쓰고 있어서 클래스 상단에 `@RequestMapping("/api/v1")` 으로 된 것을 보고, 내가 담당하는 StoreController에 도메인 경로를 명시하면 매핑이 꼬이지 않을까 갑자기 헷갈리시 시작했다.

* **학습 내용** :
  * 구체성 우선 원칙: 스프링은 가장 구체적으로 매핑된 경로(Most Specific Match)를 우선함.
  * 경로 분리: `/api/v1/stores`로 클래스 레벨 매핑을 가져가도 팀원의 `/api/v1/stores/{id}/menus`와는 문자열 단위로 비교해서 다른 경로로 인식하기 때문에 절대 충돌하지 않는다.  
    👉🏻 즉, 클래스 상단에 도메인을 명시함으로써 메서드 내부의 중복 경로를 제거하고 가독성을 높일 수 있다.(도메인 캡슐화)

### EnableWebSecurity와 EnableMethodSecurity 의 차이? 메서드 보안?!

* `@EnableWebSecurity`
  * 역할: Spring Security의 웹 보안 인프라(필터 체인)를 활성화한다.
  * 기능: HTTP 요청 가로채기, CSRF 방어, 세션 관리, 로그인/로그아웃 등 웹 레이어 전체에 대한 보안 설정
  * 메서드 보완: URL 패턴 기반 보안(http.authorizeHttpRequests)은 "어떤 주소로 들어올 수 있는가"를 결정하고, @PreAuthorize는 "그 주소 안의 어떤 기능을 실행할 수 있는가"를 더 세밀하게 제어하며 상호 보완합니다.

* `@EnableMethodSecurity`
  * 역할: 애플리케이션에서 @PreAuthorize, @PostAuthorize 같은 메서드 보안 기능(AOP)을 활성화하는 설정
  * 필요성: 이 어노테이션이 설정 클래스(SecurityConfig)에 있어야만 메서드에 붙인 @PreAuthorize가 실제로 동작한다.  (Spring Security는 성능 최적화를 위해 이 기능을 기본적으로 켜두지 않는다. )

* `@PreAuthorize`
  * 역할: 개별 메서드(컨트롤러, 서비스 등)가 실행되기 직전에 권한을 검사하는 실행 조건을 정의함.
  * 특징: SpEL(Spring Expression Language)을 사용하여 hasRole(), hasAnyAuthority() 등 복잡한 권한 로직을 문자열로 선언할 수 있다.
  * 동작: 조건이 맞지 않으면 AccessDeniedException을 발생시켜 메서드 실행 자체를 차단한다.

---

## 3. Todo

- [ ] 각 도메인(Area, Category, Store) 에 해당하는 시퀀스 다이어그램 그리기
- [ ] 연관관계에 있는 필드 삭제 메서드 리팩토링

---

#내일배움캠프 #단기Java #TIL #SpringBoot #입문프로젝트 #InfraArchitecture #CloudComputing #SpringSecurity
