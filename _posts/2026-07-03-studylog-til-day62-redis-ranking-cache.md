---
title: "[내일배움캠프 TIL, Day 62] Redis 캐싱으로 랭킹 성능튜닝 + 측정 기반 의사결정"
excerpt: "설계 단계에서 예측한 병목을 직접 수치로 재확인하고, 랭킹 응답 캐싱으로 개선했다. ZSet 유실 실수가 다음 작업으로 이어진 과정까지"

categories:
  - Studylog
tags:
  - [TIL, Redis, Cache, K6, Grafana]

permalink: /studylog/til-day62-redis-ranking-cache/

toc: true
toc_sticky: true

date: 2026-07-03
last_modified_at: 2026-07-03
---

## 1. 오늘 한 것

어제 Prometheus + Grafana 파이프라인을 구성하고 베이스라인을 잡은 데 이어, 오늘은 예측했던 병목을 직접 수치로 재확인하고 캐싱으로 개선하는 사이클을 완결했다.

* 랭킹 응답 전체 캐싱 구현 → K6 재측정 → Grafana로 전후 비교
* ZSet 잘못 날려서 캐시 미스 폭발 상황 경험 → 다음 작업으로 이어짐
* 일정 파일 기반으로 남은 계획 재조정
* PR 올리기

---

## 2. 설계 단계에서 이미 예측했던 병목

SA 단계에서 매장 상세 캐싱을 의도적으로 보류할 때 이런 판단을 했었다.

> "getStore() 단건은 4쿼리로 고정, 목록 대비 트래픽 낮음. 무효화 포인트도 복잡(기본정보/영업시간/편의시설/이미지 각각). 모니터링 후 실제 병목 확인 시 도입."

랭킹의 병목은 설계·구현 당시에는 몰랐다. MVP 중간점검에서 K6를 처음 돌렸을 때 발견했다.

```
ZSet(store:ranking) → 상위 10개 storeId 조회  ← Redis (빠름)
storeRepository.findActiveStoresByIds(ids)    ← DB 조회 (매 요청마다, 여기가 문제)
```

순위 자체는 ZSet에 캐싱돼 있었으니 빠를 거라고 생각했는데, **매장 상세 N건을 매 요청마다 DB에서 조회**하고 있었다. K6 결과로 나온 p95 **27초**, 
DB 커넥션 10개 내내 점유를 보고 나서야 구조적 문제를 인식했다.

SA 단계에서 매장 상세 캐싱은 "모니터링 후 병목 확인 시 도입"으로 보류해뒀는데, 결국 측정해보니 상세가 아니라 랭킹 응답 전체가 문제였다. 
설계할 때 예측한 부분과 실제 병목이 다른 곳에 있었던 셈이고, 그래서 직접 측정하는 게 중요했다.

---

## 3. 캐싱 구현

해결 방향은 명확했다. 응답 전체를 캐싱해서 ZSet과 DB 조회를 모두 건너뛰는 것.

```
store:ranking:response:{size} → List<StoreRankingResult> JSON (TTL 300s)
```

### 흐름

```
GET /api/v1/stores/ranking
    ↓
store:ranking:response:{size} 캐시 히트?
    ├── YES → 바로 반환 (ZSet/DB 조회 없음)
    └── NO  → ZSet에서 storeId 조회 → DB에서 매장 상세 조회 → 캐싱 후 반환

평점 갱신 이벤트 수신 시 (review.events.v1 Kafka 소비)
    → store:ranking:response:* evictAll() (SCAN 기반, KEYS 블로킹 회피)
```

### 예외 처리 - 캐시는 보조 수단

캐시 조회/저장/무효화 실패가 서비스 중단으로 이어지면 안 된다.

```java
} catch (Exception e) {
    log.warn("랭킹 응답 캐시 조회 실패 - ZSet 조회로 fallback. key={}", cacheKey, e);
    return Optional.empty();
}
```

무효화 실패 시에도 TTL(5분) 만료까지 stale 데이터를 허용하는 트레이드오프를 의도적으로 수용했다. 평점 변경이 최대 5분 늦게 반영될 수 있지만, 무효화 실패로 서비스가 멈추는 것보다 낫다고 판단했다.

---

## 4. 재측정 결과

#### 캐싱 전

<img width="1504" height="499" alt="image" src="https://github.com/user-attachments/assets/6d877d6c-e041-417d-9358-d5793e3a14f7" />

<img width="807" height="206" alt="스크린샷 2026-07-03 오후 3 14 53" src="https://github.com/user-attachments/assets/382ec4c8-013e-45ca-ab3a-00016eb5a085" />

<img width="758" height="564" alt="스크린샷 2026-07-03 오후 3 15 08" src="https://github.com/user-attachments/assets/fad4fda4-acc0-48bf-b60f-066b6a3c8de9" />

#### 캐싱 후

<img width="1505" height="498" alt="image" src="https://github.com/user-attachments/assets/b676cd3a-1593-4544-90e6-fbc6e6bc1e40" />

<img width="850" height="549" alt="image" src="https://github.com/user-attachments/assets/2ea35a18-e57e-4200-ae52-6c5527a9b044" />

#### 개선 폭

| **지표** | **Before** | **After** | **개선** |
| --- | --- | --- | --- |
| p95 | 25.14s | 9.5ms | 약 **2,600배** |
| RPS | 5.88/s | 1,076/s | 약 **183배** |
| DB 커넥션 | 10개 지속 점유 | 스파이크 후 0 | 캐시 히트 구간 DB 미사용 |
| 처리 요청 수(2분) | 762건 | 129,269건 | 약 170배 |

캐싱 후 p95 그래프를 보면 중간에 잠깐 튀는 구간이 있다. 이건 TTL 만료 직후 첫 요청이 캐시 미스로 DB를 찌르는 정상 패턴이다. 문제가 아니라 캐싱이 제대로 동작한다는 증거다.

---

## 5. ZSet 날린 실수 - 대시보드 & 로그 모니터링

재측정 준비 중에 Redis를 정리하다가 `flushDB`로 `store:ranking` ZSet까지 함께 날려버렸다.

서비스 로그에서 바로 잡혔다. `[캐시 미스]` 로그가 요청마다 계속 찍히는 걸 보고 이상하다는 걸 알아챘다. 
캐시 히트가 한 번도 안 되고 있는 거였다. ZSet도 비어있으니 `getTopRanking()`이 빈 리스트를 반환했고, 응답 캐시가 영구히 채워지지 않는 상태였다. 
결과적으로 랭킹 응답은 전부 빈 배열(`rankings: []`)로 반환됐다.

Grafana 그래프에서도 DB 커넥션이 잠깐 튀는 패턴이 보이긴 했지만, 직접적인 단서는 로그였다.

복구는 DB의 `average_rating`이 원본으로 남아있어서 재시딩 스크립트로 해결했지만, 이 과정에서 구조적인 문제를 하나 발견했다.

**ZSet이 없을 때 DB로 직접 조회하는 fallback이 없다.**

현재 코드에 TODO로 남겨뒀다.

```java
// TODO: Redis 전체 장애 시 ZSet도 비어 빈 리스트 반환됨 — DB 직접 조회 fallback 미구현 (보류)
List<UUID> rankedIds = storeRankingRepository.getTopRanking(safeSize);
```

원래 Spring Batch로 주기적 재동기화를 고도화에서 구현할 계획이었는데, 남은 일정을 고려하면 정기 자동화보다 ZSet 유실 시 즉시 DB 기반으로 재구성하는 lazy rebuild fallback을 먼저 구현하는 게 현실적이라고 판단했다.

---

## 6. 남은 일정 재조정

오늘 일정 파일을 다시 열어서 남은 작업을 점검했다.  
설계 단계에서 예정 외 작업(StoreHours 정비)도 있었고, 오늘 ZSet 유실 건으로 lazy rebuild 작업이 하나 더 붙었다. 

```
✅ 완료: 메트릭 계측 + 랭킹 캐싱 + 재측정
🔴 ZSet lazy rebuild fallback
🟠 매장 상세 캐싱 데이터 판정 / Consumer 모니터링
🟡 [보류] Spring Batch 정기 자동화 - "위험 인지 + lazy rebuild로 즉시 대응 + 자동화는 의도적 보류"
```

대표 작업 하나를 제대로 완결하는 게 낫다는 튜터님의 피드백을 가이드 삼아 재조정 해보았다.

---

## 7. 오늘의 회고

캐싱 구현 자체는 복잡하지 않았다. 설계할 때 이미 어떤 구조가 병목이 될지 적어뒀고, K6로 재측정하니 예측이 그대로 나왔다. 그 수치를 보고 캐싱하면 되는 거였다.  
ZSet을 날린 건 실수지만, 결과적으로 로그 없이는 놓쳤을 구조적 문제를 발견했다. 서비스 로그에 캐시 미스가 계속 찍히는 걸 보고 이상하다는 걸 알아챘다. 로그가 없었으면 "랭킹이 왜 빈 배열이지?" 정도로 끝났을 것이다.  
설계 단계의 판단을 수치로 검증하고, 모니터링으로 예상 밖 문제를 잡고, 다음 작업으로 연결하는 흐름이 오늘 하루에 다 들어있었다. 그게 오늘 가장 의미 있었던 부분이다.

---

#내일배움캠프 #단기Java #TIL #Kok #Redis #Cache #K6 #Grafana

