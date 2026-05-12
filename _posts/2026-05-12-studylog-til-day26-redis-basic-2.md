---
title: "[내일배움캠프 TIL, Day 26] Redis 기초: 인메모리 저장소와 자료구조2"
excerpt: "Redis를 활용한 세션 클러스터링, Sorted Set 리더보드 및 캐싱 전략 정리"

categories:
  - Studylog
tags:
  - [TIL, Redis, In-Memory, NoSQL, Database]

permalink: /studylog/til-day26-redis-basic-2/

toc: true
toc_sticky: true

date: 2026-05-12
last_modified_at: 2026-05-12
---

## 1. 오늘 학습 키워드

* **Session Management**: Sticky Session vs Session Clustering
  * Session Clustering: 서버 간 세션 공유를 통한 확장성(Scale-out) 확보.
* **Sorted Set (ZSET)**: 가중치(Score)를 기반으로 정렬된 데이터를 관리하는 Redis 자료구조
* **Caching Strategy**: 서비스 특성에 따른 데이터 조회 및 저장 전략. ache-Aside, Write-Through, Write-Behind
* **Cache Eviction**: 한정된 메모리 자원 관리를 위한 데이터 삭제 정책.

---

## 2. 학습 내용 정리하기

### 1) 세션 관리와 Redis

* **Sticky Session vs Session Clustering**
  * **Sticky Session**: 로드밸런서가 최초 요청 서버로 고정 할당. 구현은 쉽지만 특정 서버 부하 집중 및 장애 시 세션 유실 위험이 큼.
  * **Session Clustering**: 외부 저장소(Redis 등)에 세션을 통합 관리. 서버 확장(Scale Out)에 유리하며 서버 장애 시에도 세션이 유지됨.
* **Spring Session Data Redis**
  * 의존성 추가만으로 자동 구현 가능. 동일 도메인 내 서버 간 쿠키 공유를 통해 자연스러운 세션 연동 지원.
  * 기본적으로 Java 직렬화를 사용하며, 보안 및 호환성을 위해 직렬화 방식(Serializer) 선택 시 주의 필요.

### 2) Redis 자료구조 활용: 리더보드

* **Sorted Set (ZSET)**
  * 점수(Score)를 기준으로 정렬된 상태를 유지하는 자료구조.
  * `Reverse Range` 연산 시 내부적으로 `LinkedHashSet` 형태의 결과를 반환하여 데이터의 순위(Rank) 순서를 엄격히 보장한다.
  * **효율성**: 데이터 추가(ZADD)는 $O(\log N)$, 범위 조회(ZRANGE)는 $O(\log N + M)$으로 매우 빠름.
  * **활용**: 실시간 검색어, 인기 상품 순위, 게임 리더보드 등 '정렬'과 '카운팅'이 빈번한 기능에 최적.
  * SQL과 달리 메모리 기반의 다양한 자료구조(HyperLogLog, Geo 등)를 목적에 맞게 선택 가능.

### 3) 캐싱(Caching) 개념 및 전략
* **캐시**: 데이터베이스(Disk)보다 빠른 메모리(RAM)에 데이터를 임시 저장하여 응답 속도를 높이는 기술
* **주요 용어**
  * **Cache Hit**: 요청한 데이터가 캐시에 존재하여 즉시 반환.
  * **Cache Miss**: 캐시에 데이터가 없어 DB를 조회해야 하는 상황 (지연 발생).
  * **Eviction Policy**: 메모리 부족 시 오래된 데이터 등을 삭제하는 정책.
* **캐싱 전략**
  * **Cache-Aside**
    * 캐시 확인 -> 없으면 DB 조회 후 캐시 저장. (Lazy Loading)
    * 가장 일반적임. 첫 호출은 느리지만 이후는 빠름. DB와 캐시 간 데이터 불일치 가능성 존재.
  * **Write-Through**: 
    * DB와 캐시에 동시에 쓰기. 
    * 캐시는 항상 최신 상태를 유지하지만, 쓰기 성능이 저하되고 사용되지 않는 데이터가 메모리를 점유함.
  * **Write-Behind**: 
    * 캐시에 먼저 쓰고 DB에는 나중에 모아서 반영(Batch). 
    * 쓰기 성능이 극대화되지만, 장애 발생 시 캐시 내 미반영 데이터가 유실될 위험이 있음.

---

#내일배움캠프 #단기Java #TIL #Redis #SessionClustering #CachingStrategy #SortedSet
