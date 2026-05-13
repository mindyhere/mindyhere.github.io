---

title: "[내일배움캠프 TIL, Day 27] 대규모 시스템 아키텍처: Kafka & Redis 기반 MSA 설계"
excerpt: "Kafka를 이용한 비동기 메시징 인프라 구축과 Redis의 다양한 활용 전략(캐싱, 세션, 리더보드), 그리고 MSA의 핵심인 Saga 패턴 복습"

categories:
  - Studylog
tags:
  - [TIL, Redis, Kafka, MSA, EDA, EventDriven]

permalink: /studylog/til-day27-kafka-redis-msa-remind/

toc: true
toc_sticky: true

date: 2026-05-13
last_modified_at: 2026-05-13

---

## 1. 키워드 요약
* **Message Broker**: Apache Kafka (Zookeeper, Broker, Topic, Partition)
* **Messaging Pattern**: Pub/Sub, Producer/Consumer, Consumer Group
* **In-Memory Data Store**: Redis (Session Clustering, Caching, Sorted Set)
* **MSA Transaction**: Distributed Transaction, Saga Pattern (Orchestration & Choreography)

---

## 2. 학습 내용 정리하기

### 🚀 Apache Kafka를 활용한 비동기 메시징 시스템

대규모 시스템에서 서비스 간 결합도를 낮추고 데이터 스트림을 안정적으로 처리하기 위해 Kafka 인프라를 실습했다.

#### 1) 인프라 및 환경 구성 (Docker)

* **Zookeeper & Broker**: 클러스터 상태 관리 및 실제 데이터 저장을 담당하는 핵심 컴포넌트를 Docker Compose로 통합 관리
* **Kafka UI**: GUI를 통해 토픽 상태, 파티션 부하, 컨슈머 그룹의 오프셋을 실시간 모니터링

#### 2) 주요 메커니즘

* **순서 보장 (Ordering)**: 메시지 발행 시 특정 `Key`를 지정하여 동일 파티션으로 할당함으로써 **파티션 내에서의 메시지 처리 순서를 보장(Partition-level Ordering).**
* **부하 분산 (Scalability)**: `Consumer Group` 내 여러 컨슈머가 파티션을 나누어 병렬 처리하며, 특정 컨슈머 장애 시 리밸런싱을 통해 중단 없는 데이터 소비 가능.
* **비동기 처리**: `KafkaTemplate`과 `@KafkaListener`를 활용해 요청-응답의 블로킹을 제거, 시스템 전반의 응답성 향상.

### 🚀 Redis와 MSA의 시너지 전략

Redis는 메모리 기반의 속도를 바탕으로 MSA의 분산 환경에서 발생하는 데이터 Latency를 줄이는 등 다양한 문제 해결을 위한 핵심 컴포넌트로 활용할 수 있다.

#### 1) 주요 활용 사례
1. **세션 클러스터링 (Session Clustering)**:
   - 서버가 Scale-out 되어도 사용자의 인증 상태를 유지하기 위해 Redis를 공유 세션 저장소로 활용.
2. **캐싱 전략 (Caching Strategy)**:
   - 자주 조회되는 데이터나 연산 비용이 높은 데이터를 메모리에 상주시켜 메인 DB(RDBMS)의 I/O 부하를 경감하며, **적절한 TTL(Time-To-Live) 설정을 통해 데이터 정합성을 유지.**
3. **실시간 리더보드 (Leaderboard)**:
   - `Sorted Set` 자료구조를 활용해 대량의 데이터를 DB 연산 없이 실시간 랭킹 순으로 정렬/조회.
4. **처리율 제한 (Rate Limiter)**:
   - 특정 시간당 요청 횟수를 관리하여 시스템 과부하 방지 및 안정성 확보.

### 🚀 MSA의 분산 트랜잭션 관리: Saga 패턴

마이크로서비스 환경에서는 각 서비스가 독립된 DB를 가지므로 전통적인 ACID 트랜잭션 유지가 어렵다. 이를 위해 **'최종적 일관성(Eventual Consistency)'**을 보장하는 Saga 패턴을 활용할 수 있다.

#### 1) Saga 패턴의 두 가지 방식

| 구분     | Choreography (코레오그래피)      | Orchestration (오케스트레이션)        |
|:-------|:---------------------------|:-------------------------------|
| **특징** | 중앙 제어자 없이 서비스 간 이벤트 교환     | 중앙 매니저(Orchestrator)가 워크플로우 제어 |
| **장점** | 구조가 단순하며 서비스 간 결합도가 매우 낮음  | 복잡한 비즈니스 로직의 흐름 파악 및 제어 용이     |
| **단점** | 비즈니스 흐름 파악 및 디버깅이 어려울 수 있음 | 매니저 서비스에 대한 의존성 및 복잡도 증가       |

#### 2) 보상 트랜잭션 (Compensating Transaction)

- 작업 도중 실패 발생 시, 이전에 완료된 단계들을 논리적으로 취소(되돌리기)하는 이벤트를 발생시켜 데이터의 일관성을 맞추는 기법.
- **설계 고려사항**: 분산 트랜잭션 특성상 **격리성(Isolation) 수준이 낮아** 중간 데이터 노출에 대한 설계적 고려가 필요함.

---
#내일배움캠프 #단기Java #TIL #Redis #Kafka #MSA #SagaPattern #Architecture #EDA #EventDriven
