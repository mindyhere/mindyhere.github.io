---
title: "[내일배움캠프 TIL, Day 6] MSA 실습 1일차: 게이트웨이 보안 심화, Product 도메인 설계"
excerpt: "Spring Cloud Gateway의 필터 체인 검증, JWT 인증 로직 분리, 엔티티 설계, QueryDSL 설정, 그리고 쿠버네티스(K8s)와 Spring Cloud 비교"

categories:
  - Studylog
tags:
  - [TIL, MSA, Gateway, JWT, JPA, K8s]

permalink: /studylog/til-day6-msa-practice-1/

toc: true
toc_sticky: true

date: 2026-04-13
last_modified_at: 2026-04-13
---

## 1. 오늘 학습 키워드
* **Spring Cloud Gateway**: 전역 필터(Global Filter)를 통한 요청/응답 제어 및 로깅
* **ServerWebExchange**: WebFlux 기반의 비동기 요청/응답 통합 객체 활용
* **JWT Filter Refactoring**: 인증 검증과 데이터 전달(Mutation)의 책임 분리
* **Domain Driven Design (DDD)**: 도메인 모델에 로직을 응집시키는 객체지향 설계
* **Lombok AccessLevel**: 캡슐화를 강화하기 위한 접근 제한자 설정
* **QueryDSL**: 복잡한 조건 검색과 페이징 처리를 위한 동적 쿼리 도구

---

## 2. 학습 내용 정리하기

### 🔐 Gateway & JWT

* **ServerWebExchange의 활용**: `HttpServletRequest`와 달리 비동기 환경에 최적화된 객체로, 요청 정보 조회부터 응답 코드 설정(`setStatusCode`)까지 한 번에 관리함.
* **Filter 역할 분리**:
  - **Pre Filter**: 요청이 들어오는 즉시 URI를 로깅하고 초기 검사를 수행함.
  - **Local JWT Filter**: 토큰의 유효성을 검사하고, `Claims`를 추출함.
  - **Request Mutation**: 인증된 사용자 정보(`user_id`, `role`)를 헤더에 담아 뒤쪽 서비스로 전달하기 위해 `exchange.mutate()`를 사용하여 새로운 복사본을 생성함.
  - **Post Filter**: 서비스로부터 돌아온 응답이 나갈 때 마지막으로 상태 코드를 로깅함.

```java
@Slf4j
@Component
public class LocalJwtAuthenticationFilter implements GlobalFilter {

    @Value("${service.jwt.secret-key}")
    private String secretKey;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // ServerWebExchange: Request에 대한 모든 정보와 Response를 처리할 도구를 하나로 묶은 불변 객체. 비동기방식

        // 1. 요청 경로 확인
        // exchange.getRequest(): 클라이언트가 보낸 URI, 헤더, 쿠키, 파라미터 등을 읽을 수 있다.
        String path = exchange.getRequest().getURI().getPath();
        // 회원가입, 로그인 ->  필터를 적용하지 않음
        if (path.equals("/auth/signIn") || path.equals("/auth/signUp")) {
            return chain.filter(exchange);  // 응답 스트림을 닫고 프로세스 종료
        }

        // 2. 토큰 추출 및 검증
        String token = extractToken(exchange);
        Claims claims = validateToken(token); // 2-1. 검증 및 정보 획득
        if (claims == null) { // 2-2. 검증 실패 처리
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        // 3. 검증 성공 시: 새로운 정보를 담은 exchange로 교체 (Mutate) -> 뒤쪽 서비스(Product, Order 등)로 헤더가 전달됨
        ServerWebExchange mutatedExchange = exchange.mutate()
            .request(r -> r
                          .header("X-User-Id", claims.get("user_id").toString())
                          .header("X-Role", claims.get("role").toString()))
            .build();

        return chain.filter(mutatedExchange); // 4. 내용을 수정한 복사본(mutatedExchange)을 다음으로 전달
    }
    // ... (추출 및 검증 로직 생략)
}
```



```java
@Slf4j
@Component
public class CustomPreFilter implements GlobalFilter, Ordered {
//    GlobalFilter: 게이트웨이로 들어오는 모든 요청에 대해 공통적으로 실행되는 필터임을 선언
//    Ordered: 여러 필터가 있을 때 실행 순서를 결정
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        log.info("##### Pre Filter: Request URI is " + request.getURI());
        // Add any custom logic here
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE; // 가장 높은 우선순위 설정
    }
}

```


```java
@Slf4j
@Component
public class CustomPostFilter implements GlobalFilter, Ordered {

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {
    return chain.filter(exchange).then(Mono.fromRunnable(() -> {
      ServerHttpResponse response = exchange.getResponse();
      log.info("##### Post Filter: Response status code is " + response.getStatusCode());
      // Add any custom logic here
    }));
  }

  @Override
  public int getOrder() {
    return Ordered.LOWEST_PRECEDENCE; // 가장 낮은 우선순위 설정
  }
}
```




### 🏗️ 엔티티 설계 : Product Entity 예제
데이터만 담는 바구니가 아니라, 스스로의 상태를 관리하는 **풍성한 도메인 모델**을 지향함.
* **캡슐화 강화**: `@AllArgsConstructor(access = AccessLevel.PROTECTED)`와 빌더의 `PRIVATE` 설정을 통해 무분별한 객체 생성을 막고 정적 팩토리 메서드(`createProduct`)로 생성을 일원화함.
* **비즈니스 메서드**: Setter 대신 명확한 의미를 가진 `updateProduct`, `deleteProduct` 메서드를 내부에 두어 데이터 일관성을 유지함.
* **JPA 생명주기 콜백**: `@PrePersist`, `@PreUpdate`를 사용하여 생성/수정 시간을 시스템이 자동으로 관리하도록 자동화함.



```java
@Getter
@NoArgsConstructor
@AllArgsConstructor(access = AccessLevel.PROTECTED) // 외부에서 모든 필드 생성자 호출 차단
@Builder(access = AccessLevel.PRIVATE)             // 외부에서 빌더 직접 사용 차단.정적 팩토리 메서드 유도
@Table(name = "products")
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private Integer price;
    // ... 필드 생략

    @PrePersist // JPA 생명주기 콜백: Insert 쿼리 실행 직전 자동실행
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    public static Product createProduct(ProductRequestDto requestDto, String userId) {
        return Product.builder()
            .name(requestDto.getName())
            .price(requestDto.getPrice())
            .createdBy(userId)
            .build();
    }
}
```




### 🔍 QueryDSL 설정
복잡한 검색 조건과 페이징 처리를 위해 도입함.
* **JPAQueryFactory**: `EntityManager`를 주입받아 빈으로 등록하며, 런타임이 아닌 컴파일 시점에 쿼리 오류를 잡아낼 수 있는 안정성을 확보함.

### ☸️ 인프라의 확장 (Kubernetes)

* **개념**: 컨테이너 배포, 확장, 운영을 자동화하는 오케스트레이션 도구.
* **Spring Cloud와의 차이**:
  - **Spring Cloud**: 서비스 간 통신(Discovery, Config) 등 애플리케이션 내부 로직 중심.
  - **Kubernetes**: 컨테이너 이미지 기반의 배포 관리, 자가 치유(Self-healing) 등 인프라 운영 중심.
* **고민점**: 도입 시 확장성과 자동화 측면에서 유리하나, 운영 비용이 발생하고 분산 환경에서의 디버깅이 어려워질 수 있음.

---

## 3. 학습하며 겪었던 문제점 & 에러

#### 1) 필터 로그 순서 및 검증 문제
* **상황**: `/auth/signIn` 요청 시 JWT 필터 로그가 찍히지 않음.
* **해결**: 설계상 로그인 경로는 `permitAll`처럼 필터를 스킵하도록 구현했기 때문임. `Pre Filter` -> `Post Filter` 순으로 200 OK 로그가 찍힌다면 정문 보안 시스템이 의도대로 작동 중인 것임.

#### 2) Netty MacOS DNS 라이브러리 경고
* **상황**: 실행 시 `Unable to load netty-resolver-dns-native-macos` 에러 발생.
* **해결**: 맥 OS 전용 라이브러리 부재로 인한 경고이며, 시스템 기본값(fallback)으로 자동 전환되므로 로컬 실습에는 지장이 없음을 확인함.

---

#내일배움캠프 #단기Java #TIL #SpringCloudGateway #JWT인증 #DDD #K8s