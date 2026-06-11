---
title: "[내일배움캠프 TIL, Day 47] MSA SA 문서 설계 — 테이블 & API 명세"
excerpt: "기능명세서 기반으로 API명세 작성하기, ERD 설계"

categories:
  - Studylog
tags:
  - [TIL, final, api, erd]

permalink: /studylog/til-day46-msa-api-erd/

toc: true
toc_sticky: true

date: 2026-06-11
last_modified_at: 2026-06-11
---

## 1. 오늘 학습 키워드

* MSA 환경에서의 알림 테이블 역할 분리 설계
* 폴리모픽 참조 (Polymorphic Reference) — FK 없이 `reference_id` + `reference_type`으로 원본 참조

---

## 2. 학습내용 정리

### 🤔 알림 테이블을 하나로 합쳐야 할까, 나눠야 할까?

처음엔 알림 관련 테이블을 Notification Service 하나에서 관리하는 것인가 했는데, 역할이 다른 두 가지를 구분해야 했다.

- **발송 이력 추적** (PENDING → SENT/FAILED): Kafka 소비 후 Slack 메시지를 실제로 보냈는지 추적하는 감사 로그 성격 → 각 서비스(Waiting, Reservation)에 위치하는 게 맞음
- **사용자 알림함**: 사용자가 앱에서 "내 알림 목록"을 조회하고 읽음 처리하는 UI 데이터 → Notification Service에서 관리

이 둘을 분리하지 않으면 Notification Service가 비대해지고, MSA의 서비스 경계도 흐려진다.
한편 MSA 환경에서 DB 간 FK를 걸 수 없기 때문에, `reference_id` + `reference_type` 폴리모픽 참조로 원본 데이터를 느슨하게 연결할 수 있도록 설계해보았다.


### Kafka로 알림 기능 구현하기 — 간단 적용 예제

### 전체 흐름

알림 기능에서 Kafka를 쓰는 이유는 **서비스 간 직접 호출 없이** 이벤트를 전달하기 위해서다.

```
Waiting Service          Kafka                Notification Service
     │                                                │
     │── waiting.called 이벤트 발행 ──▶ [토픽] ──▶ 이벤트 수신
     │                                                │
     │                                        Slack 알림 발송
     │                                                │
     │                                    p_notifications 저장
```

이번 프로젝트에 적용한다면,
- Waiting Service는 "내가 알림 보내야 해" 대신 "이런 일이 일어났어" 만 발행
- Notification Service는 자신의 속도로 소비, 처리


### 발행 측 (Waiting Service)

```java
// 이벤트 객체
public record WaitingCalledEvent(
    String waitingId,
    String userId,
    String storeId,
    int currentRank
) {}

// Kafka 발행
@Service
@RequiredArgsConstructor
public class WaitingEventPublisher {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    public void publishCalled(Waiting waiting) {
        WaitingCalledEvent event = new WaitingCalledEvent(
            waiting.getId(),
            waiting.getUserId(),
            waiting.getStoreId(),
            waiting.getRank()
        );
        kafkaTemplate.send("waiting.called", waiting.getUserId(), event);
        // key를 userId로 설정 → 같은 유저 이벤트는 항상 같은 파티션으로
    }
}
```


### 수신 측 (Notification Service)

```java
@Component
@RequiredArgsConstructor
public class WaitingEventConsumer {

    private final SlackNotificationSender slackSender;
    private final NotificationRepository notificationRepository;

    @KafkaListener(topics = "waiting.called", groupId = "notification-service")
    public void onWaitingCalled(WaitingCalledEvent event) {
        // 1. Slack 알림 발송
        slackSender.send(event.userId(), "웨이팅 순번이 됐습니다!");

        // 2. 사용자 알림함에 저장 (p_notifications)
        Notification notification = Notification.builder()
            .userId(event.userId())
            .type("WAITING_CALLED")
            .referenceId(event.waitingId())
            .referenceType("WAITING")
            .isRead(false)
            .build();
        notificationRepository.save(notification);
    }
}
```


### 발송 이력 추적 (Waiting Service 내부)

```java
// Waiting Service에서 발송 이력 저장 — p_waiting_notifications
@KafkaListener(topics = "waiting.called", groupId = "waiting-notification-tracker")
public void trackNotification(WaitingCalledEvent event) {
    WaitingNotification record = WaitingNotification.builder()
        .waitingId(event.waitingId())
        .userId(event.userId())
        .notificationType("CALLED")
        .status("PENDING")
        .build();
    waitingNotificationRepository.save(record);

    try {
        // 실제 발송 처리 (또는 별도 서비스에 위임)
        record.markSent();
    } catch (Exception e) {
        record.markFailed(e.getMessage());
    }
    waitingNotificationRepository.save(record);
}
```


### application.yml 핵심 설정

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: notification-service
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
      auto-offset-reset: earliest  # 컨슈머 재시작 시 처음부터 읽기
```


### 핵심 포인트 정리

| 항목 | 내용 |
|---|---|
| 토픽 key | `userId` 설정 → 같은 유저 이벤트 순서 보장 |
| 컨슈머 그룹 | 서비스마다 다른 `groupId` → 각자 독립적으로 소비 |
| `auto-offset-reset: earliest` | 컨슈머 다운 후 재시작 시 누락 이벤트 재처리 |
| 발송 이력 분리 | 감사 목적(`p_waiting_notifications`)과 알림함(`p_notifications`)은 다른 테이블 |

---

## 3. todo

- [ ] SA 문서 피드백 및 아키텍처/인프라 구성
- [ ] 역할 분담

---

#내일배움캠프 #단기Java #TIL
