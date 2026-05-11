---
title: "[내일배움캠프 TIL, Day 25] Redis 기초: 인메모리 저장소와 자료구조"
excerpt: "Redis의 기본 개념과 주요 자료구조, Spring Data Redis와 RedisTemplate을 활용한 캐싱 및 데이터 관리 실습"

categories:
  - Studylog
tags:
  - [TIL, Redis, In-Memory, NoSQL, Database]

permalink: /studylog/til-day25-redis-basic-1/

toc: true
toc_sticky: true

date: 2026-05-11
last_modified_at: 2026-05-11
---

## 1. 오늘 학습 키워드

* **인메모리 저장소(In-Memory Storage)**: 데이터를 메인 메모리(RAM)에 저장하여 고속의 읽기/쓰기를 지원하는 저장소.
* **Redis(Remote Dictionary Server)**: 'Key-Value' 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈 소스 기반의 비관계형 데이터베이스(NoSQL).
  * REmote DIctionary Server를 줄인말. Java의 Map과 같은 방식으로 데이터를 저장하는 데이터베이스
* **Spring Data Redis**: Redis를 추상화하여 편리하게 사용할 수 있게 돕는 라이브러리
  * RedisRepository: 인터페이스 기반의 CRUD 기능 제공 (@RedisHash)
  * RedisTemplate: Redis 명령어에 대한 세밀한 제어가 가능한 저수준 API

---

## 2. 학습 내용 정리하기

### 🚀 인메모리 저장소와 Redis의 활용 사례

#### 인메모리(In-Memory) 저장소

1. 하드디스크(HDD/SSD)가 아닌 RAM에서 데이터를 처리하므로 지연 시간(Latency)이 상대적으로 낮음.
2. 메모리의 특성상 전원이 꺼지면 데이터가 사라지기 때문에 주로 임시 데이터 저장(캐싱, 세션 관리)에 사용된다.
3. RAM은 디스크에 비해 가격이 비싸고 용량이 제한적이기 때문에 전체 데이터를 저장하기보다 자주 조회되는 핵심 데이터 위주로 저장하는 전략이 필요하다.

#### NoSQL의 유형과 특징
Redis는 NoSQL 데이터베이스 중 하나로, 전통적인 RDBMS(관계형 데이터베이스)와 달리 데이터 간의 관계를 정의하지 않고 유연한 설계를 지향한다.

1. **Key-Value Store (Redis)**:
  - 가장 단순하고 빠른 형태. Key 하나에 하나의 Value를 매핑한다.
  - Java의 `Map`이나 Python의 `Dictionary` 구조와 유사하여 직관적이다.
2. **Document Store (MongoDB)**:
  - 데이터를 JSON, BSON, XML 같은 문서 형태로 저장한다.
  - 각 문서마다 구조가 다를 수 있어 복잡한 객체 정보를 저장하고 관리하기에 유리하다.
3. **Column-Family Store (Cassandra)**:
  - 행(Row)마다 다른 컬럼을 가질 수 있으며, 데이터와 타임스탬프를 함께 저장한다.
  - 대규모 데이터 쓰기와 읽기에 최적화되어 있으며 가용성이 높다.

#### Redis의 활용 사례
Redis는 뛰어난 읽기/쓰기 성능을 바탕으로, 일시적이거나 변경이 잦은 데이터를 처리하는 데 쓰인다.
1. 세션 클러스터링 (Session Clustering): 여러 애플리케이션 인스턴스에서 같은 세션 정보를 사용할 수 있도록 한다.
2. 캐싱 (Caching): 자주 사용되는 데이터를 저장해두어, 데이터베이스 조회를 줄이고 전반적인 응답속도를 개선한다.
3. 실시간 리더보드 및 트래킹: 지원하는 다양한 자료구조를 바탕으로 리더보드, 방문수 트래킹, 좌표 기반 검색 등의 기능을 쉽게 구현할 수 있다.
  
### 🛠 Redis 자료구조별 주요 명령어

#### 1) String (문자열)
가장 기본적인 자료구조로, 텍스트뿐만 아니라 숫자(정수) 형태도 저장 가능하며 증감 연산이 지원된다.

- `SET {key} {value}` / `GET {key}`: 데이터 설정 및 조회
- `INCR`, `DECR`: 정수 값 1씩 증가/감소
- `MSET`, `MGET`: 여러 키-값을 한 번에 처리

#### 2) List (연결 리스트)
데이터를 순서대로 저장하며, 스택(Stack)이나 큐(Queue)를 구현하기에 최적화되어 있다.
- `LPUSH`, `RPUSH`: 왼쪽/오른쪽 끝에 삽입
- `LPOP`, `RPOP`: 왼쪽/오른쪽 끝에서 추출
- `LRANGE {key} {start} {end}`: 특정 범위의 원소 확인
- **활용 사례**: 워커 큐(Worker Queue), SNS 타임라인

#### 3) Set (집합)
중복을 허용하지 않는 고유한 값들의 모임. 집합 연산(교집합, 합집합 등)이 강력하다.
- `SADD`, `SREM`: 원소 추가 및 제거
- `SISMEMBER`: 특정 원소 존재 여부 확인 (T/F)
- `SINTER`, `SUNION`: 교집합 및 합집합 조회
- **활용 사례**: 중복 없는 방문자 수 계산, 사용자 관심사 매칭

#### 4) Hash (해시)
하나의 키 내부에 여러 개의 필드-값 쌍을 저장하는 구조. 객체를 표현하기에 적합하다.
- `HSET {key} {field} {value}`: 필드 단위 데이터 설정
- `HGETALL {key}`: 해당 키의 모든 필드와 값 조회
- **활용 사례**: 사용자 세션 정보 저장, 장바구니 구현

#### 5) Sorted Set (정렬된 집합)
값과 함께 '점수(Score)'를 저장하며, 점수를 기준으로 오름차순 자동 정렬된다.
- `ZADD {key} {score} {member}`: 점수와 함께 값 추가
- `ZRANK`, `ZREVRANK`: 특정 값의 순위 확인 (오름차순/내림차순)
- `ZRANGE`, `ZREVRANGE`: 범위별 순위 데이터 조회
- **활용 사례**: 실시간 리더보드(순위표), 처리량 제한(Rate Limiter)

### 🕹️ SpringBoot App-Redis 실습

#### 1) Spring Data Redis 접근 방식
Spring Boot에서 Redis를 사용하는 방법은 크게 두 가지로 나뉜다.

* `RedisRepository`
  * 특징: JPA와 유사한 방식으로 인터페이스를 정의하여 사용.
  * 어노테이션: @RedisHash("item")를 클래스에 선언하여 Redis의 Hash 자료구조를 활용.
  * ID 관리: @Id 필드를 String으로 선언하면 UUID가 자동으로 배정되어 유연한 키 관리가 가능함.

* `RedisTemplate`
  * 특징: 특정 자료형(String, Set, List 등)에 특화된 Operations 객체를 통해 명령을 수행.
  * 주요 메서드: opsForValue() (문자열), opsForSet() (집합), opsForHash() 등.
  * 유연성: 객체를 JSON 형태로 저장하거나, 특정 키에 대해 만료 시간(expire)을 직접 설정하는 등 세밀한 조작이 필요할 때 유리함.


#### 2) 직렬화 전략 (Serialization)

Redis는 기본적으로 바이트 배열을 저장하므로, 자바 객체를 어떻게 변환할지가 중요하다.

* `StringRedisTemplate`: 키와 값 모두를 일반 문자열(String)로 다룰 때 사용.
* `Custom RedisTemplate`: `Jackson2JsonRedisSerializer` 등을 활용하여 객체를 JSON 형태로 자동 변환해 저장하도록 설정 가능.
  * 설정 예시: `template.setValueSerializer(RedisSerializer.json())`;


#### 3)  Redis의 주요 기능 활용

* 데이터 만료 (TTL): `redisTemplate.expire(key, 10, TimeUnit.SECONDS)`와 같이 데이터가 자동으로 삭제될 시간을 지정하여 캐시 메모리를 효율적으로 관리함.
* 집합 자료구조 (Set Operations): 중복을 허용하지 않는 데이터(예: 취미 리스트)를 관리할 때 SetOperations를 활용하여 서버 측에서 효율적인 연산 수행.

---

#내일배움캠프 #단기Java #TIL #Redis #NoSQL #SpringDataRedis #캐싱전략
