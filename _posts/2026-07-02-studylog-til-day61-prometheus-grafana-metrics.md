---
title: "[내일배움캠프 TIL, Day 61] Prometheus + Grafana로 Spring Boot 메트릭 수집하기"
excerpt: "Actuator로 메트릭을 노출하고 Prometheus가 수집, Grafana로 시각화하는 파이프라인을 구성해 K6 부하 테스트 결과를 실시간으로 관측한 과정"

categories:
  - Studylog
tags:
  - [TIL, Prometheus, Grafana, K6]

permalink: /studylog/til-day61-prometheus-grafana-metrics/

toc: true
toc_sticky: true

date: 2026-07-02
last_modified_at: 2026-07-02
---

## 1. 오늘 학습 키워드

* Prometheus - Pull 방식 메트릭 수집, scrape_config
* Grafana - Prometheus datasource, Explore, PromQL
* Spring Boot Actuator + Micrometer - 메트릭 노출 엔드포인트
* PromQL - `rate()`, summary vs histogram 차이
* K6 - Docker 기반 부하 테스트, 베이스라인 측정

---

## 2. 개념 정리

### Prometheus가 메트릭을 수집하는 방식 - Pull

대부분의 모니터링 시스템은 애플리케이션이 데이터를 "밀어넣는(Push)" 방식인데, Prometheus는 반대로 주기적으로 "당겨오는(Pull)" 방식을 쓴다.

```
[Push 방식]
애플리케이션 → 모니터링 서버 (앱이 직접 전송)

[Pull 방식 - Prometheus]
Prometheus → 애플리케이션의 /actuator/prometheus 호출 (Prometheus가 주기적으로 요청)
```

Pull 방식의 장점은 **Prometheus가 수집을 제어한다**는 점이다. 앱은 메트릭을 노출만 하면 되고, 언제 얼마나 자주 수집할지는 `scrape_interval`로 중앙에서 관리한다. 
앱이 죽어도 Prometheus는 그냥 "수집 실패"로 기록할 뿐 자기 설정이 바뀌지 않는다.

### Spring Boot가 메트릭을 노출하는 구조

```
[메트릭 노출 스택]
애플리케이션 코드 (비즈니스 로직)
    ↓ 자동 계측
Micrometer (측정 추상화 레이어)
    ↓ Prometheus 포맷으로 변환
Spring Boot Actuator (/actuator/prometheus)
    ↓ Prometheus가 Pull
Prometheus TSDB (시계열 저장소)
    ↓ PromQL 조회
Grafana (시각화)
```

`spring-boot-starter-actuator` + `micrometer-registry-prometheus`를 추가하면 HTTP 요청 수, 응답시간, JVM 메모리 등이 자동으로 계측된다. 
별도 코드 없이 `/actuator/prometheus`에서 수백 개 메트릭이 노출된다.

### docker-compose에서 Prometheus가 로컬 서비스를 바라보는 방법

Prometheus는 도커 컨테이너 안에서 동작하는데, Spring Boot 서비스는 로컬 호스트에서 실행된다. 컨테이너 내부에서 `localhost`는 컨테이너 자신을 가리키므로 로컬 서비스에 접근할 수 없다.

```yaml
# prometheus.yml
static_configs:
  - targets:
      - host.docker.internal:8002  # 컨테이너 → 호스트 접근

# docker-compose.yml
extra_hosts:
  - "host.docker.internal:host-gateway"  # Linux에서 host.docker.internal 활성화
```

`host.docker.internal`은 컨테이너 → 호스트 머신을 가리키는 특수 호스트명이다. Mac에서는 Docker Desktop이 자동으로 설정해주지만, Linux에서는 `extra_hosts`로 직접 매핑해야 한다.

### summary vs histogram - PromQL에서 삽질한 지점

Grafana에서 p95 응답을 보려고 `histogram_quantile(0.95, ...)` 쿼리를 쳤는데 "No data"가 뜨며 한참 헤맸다.

```
[histogram_quantile 사용 조건]
메트릭 타입이 Histogram이어야 함 → _bucket, _sum, _count 세 가지 시리즈 존재

[Spring Boot HTTP 메트릭 실제 타입]
http_server_requests_seconds → Summary 타입
                             → _sum, _count만 있고 _bucket이 없음
                             → histogram_quantile 사용 불가
```

Summary 타입에서 평균 응답시간을 구하는 올바른 PromQL:

```promql
rate(http_server_requests_seconds_sum{uri="/api/v1/stores/ranking"}[5m])
/
rate(http_server_requests_seconds_count{uri="/api/v1/stores/ranking"}[5m])
```

`rate(sum) / rate(count)` = 단위 시간당 누적 응답시간 / 단위 시간당 요청 수 = 평균 응답시간(초)

정확한 p95는 구할 수 없지만 K6가 p95를 직접 측정해주므로, Grafana에서는 "추세와 평균"을 보는 용도로 활용했다.

---

## 3. 이번 프로젝트에 적용한 것들

### ✅ Prometheus + Grafana docker-compose 추가

```yaml
prometheus:
  image: prom/prometheus:v2.53.0
  volumes:
    - ./docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  extra_hosts:
    - "host.docker.internal:host-gateway"

grafana:
  image: grafana/grafana:11.0.0
  volumes:
    - ./docker/grafana/provisioning:/etc/grafana/provisioning
```

Grafana는 `provisioning` 디렉토리를 마운트하면 **컨테이너 시작 시 datasource를 자동 등록**한다. 
매번 UI에서 수동 등록할 필요 없이, `datasources.yml` 파일 하나로 Prometheus/Loki를 코드로 관리할 수 있다.

### ✅ 베이스라인 측정 - K6 + Grafana 연동

K6를 Docker로 실행하면서 Grafana Explore로 동시에 메트릭을 관측했다.

```bash
export TOKEN="eyJ..."
docker run --rm -i grafana/k6 run --env TOKEN="$TOKEN" - < ranking-load-test.js
```

### ✅ 베이스라인 수치 확인

| 엔드포인트 | 조건 | p95 |
|---|---|---|
| 랭킹 (`/api/v1/stores/ranking`) | 캐싱 없음 (상위 10건 매 요청 DB 조회) | **27.08s** |
| 검색 (`/api/v1/stores`) | Redis 캐시 적중 (USER 기준) | **26.67ms** |

랭킹이 27초인 이유는 ZSet으로 순위는 캐싱됐지만, **상위 10건 매장 상세를 매 요청 DB에서 조회**하기 때문이다. 
MVP 구현 테스트에서도 이미 확인한 결과로, 오늘은 Grafana를 활용해 시각화를 해보는 실습 위주로 기준선을 재측정 해보았다. 랭킹 응답 전체를 캐싱해 이 병목을 제거할 예정이다.

---

## 4. 학습하며 겪었던 문제점 & 에러

### 🟠 Prometheus 재시작 시 메트릭 데이터 소실

**문제**: 도커를 껐다 켜니 이전 K6 테스트 중 수집됐던 메트릭이 사라져 Grafana 그래프가 비어 있음

**원인**: `docker-compose.yml`에 Prometheus 데이터 볼륨이 없어 컨테이너 재시작 시 TSDB 초기화

**해결 방향**: 운영이라면 볼륨 추가가 필요하나, 로컬 개발 환경에서는 K6 수치 자체가 근거가 되므로 현재는 허용

---

#내일배움캠프 #단기Java #TIL #Kok #Prometheus #Grafana #K6
