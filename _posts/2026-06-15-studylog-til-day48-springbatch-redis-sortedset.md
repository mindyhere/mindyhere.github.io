---
title: "[내일배움캠프 TIL, Day 48] Spring Batch & Redis 개념 정리"
excerpt: "Spring Batch 핵심 개념과 Redis Sorted Set 랭킹 적용 포인트 정리"

categories:
  - Studylog
tags:
  - [TIL, SpringBatch, Redis, SortedSet]

permalink: /studylog/til-daㅛ48-springbatch-redis-sortedset/

toc: true
toc_sticky: true

date: 2026-06-15
last_modified_at: 2026-06-15
---

## 1. 오늘 학습 키워드

* Spring Batch — Job/Step 구조, Tasklet vs Chunk
* Redis Sorted Set — 랭킹 조회 전략 및 설계 적용

---

## 2. Spring Batch 개념 정리

### 왜 단순 반복문 대신 Spring Batch를 쓰는가?

| 항목 | 단순 반복문 | Spring Batch |
|---|---|---|
| 실패 추적 | 어느 지점에서 실패했는지 알기 어려움 | Job/Step 단위로 실패 지점 명확히 추적 |
| 실행 이력 | 직접 로깅 필요 | JobRepository가 자동 관리 |
| 장애 영향 범위 | 전체 롤백 또는 무결성 보장 어려움 | Chunk 단위 트랜잭션으로 범위 제한 |

### Tasklet vs Chunk

**Tasklet** — 단순 1회성 작업에 사용
- `execute()` 메서드 하나로 구성
- `RepeatStatus.FINISHED`를 반환하면 Step 종료
- 예: 테이블 초기화, Redis 키 삭제, 단순 루프

**Chunk** — 대량 데이터를 일정 단위로 나눠 처리
- Reader → Processor → Writer 흐름
- Chunk 단위로 트랜잭션 커밋 → 실패 시 해당 Chunk만 롤백
- 예: DB 페이징 읽기 → 가공 → Redis/DB 쓰기

### Step 분리 기준

처리 방식이 다르거나 실패 복구 기준이 다를 때 Step을 분리한다.

```
rankingRebuildJob
├── Step 1: rankingInitStep  (Tasklet)  — DEL store:ranking  ← 1회 실행
└── Step 2: rankingLoadStep  (Chunk)    — DB 페이징 → ZADD   ← N건씩 반복
```

> Step 1이 실패하면 Step 2는 시작하지 않는다. DEL과 ZADD를 같은 Writer에 넣으면 Chunk마다 DEL이 실행되어 이전 Chunk 결과가 지워지는 버그가 발생한다.

---

## 3. Redis 적용 포인트

### 개념 — Redis Sorted Set

`ZADD key score member` 형태로 score 기준 자동 정렬되는 자료구조.

- `ZADD store:ranking 4.8 storeId:1` — 삽입/업데이트 (Upsert)
- `ZREVRANGE store:ranking 0 19` — score 내림차순으로 상위 20개 조회

### 설계 적용 포인트

**전체 랭킹 조회** → Redis Sorted Set 사용

```
GET /stores?sort=rating
→ ZREVRANGE store:ranking 0 19  (Redis, 단순 읽기)
```

**필터 + 평점순 정렬** → DB 사용

```
GET /stores?category=KOREAN&sido=서울&sort=rating
→ SELECT ... WHERE category = ? AND sido = ? ORDER BY average_rating DESC
```

Sorted Set은 전체 storeId를 score 기준으로 정렬한 구조이므로, 특정 조건으로 필터링된 부분집합에는 적용할 수 없다. 조건 조합이 들어오면 DB `ORDER BY`로 처리한다.

**Reconciliation (보정)** — Kafka Consumer 장애 또는 메시지 유실로 `average_rating`이 실제 리뷰 평균과 틀어졌을 때, 배치가 `p_reviews`에서 `AVG(rating)`을 재계산해 덮어쓰고 Sorted Set을 재구성한다.

```
배치:  SELECT store_id, AVG(rating) FROM p_reviews GROUP BY store_id
       → p_stores.average_rating 업데이트
       → ZADD store:ranking 재구성
```


---

#내일배움캠프 #단기Java #TIL
