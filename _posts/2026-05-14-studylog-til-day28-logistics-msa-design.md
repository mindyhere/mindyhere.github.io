---
title: "[내일배움캠프 TIL, Day 28] 물류 플랫폼 MSA 설계: SA 문서화 및 도메인 고도화"
excerpt: "B2B 물류 시스템의 핵심 도메인 설계, Seller Order 계층 도입을 통한 정산 구조 개선, 그리고 SKU 단위의 정교한 재고 관리 전략 수립"

categories:
   - Studylog
tags:
   - [TIL, MSA, Architecture, Logistics, DDD, B2B]

permalink: /studylog/til-day28-logistics-msa-design/

toc: true
toc_sticky: true

date: 2026-05-14
last_modified_at: 2026-05-14
---

## 1. 키워드 요약
* **MSA Architecture**: 6대 독립 서비스 분리 (Auth/User, Company, Hub, Order, Delivery, Operations)
* **Hierarchical Order**: `Orders` → `Store_Orders` → `Order_Items` 3단계 계층 구조
* **Inventory Unit**: SKU(Stock Keeping Unit) 기반 옵션 조합 재고 관리
* **Logistics Model**: Hub-and-Spoke (중앙 허브 경유 모델)

---

## 2. 학습 내용 정리하기

### 🚀 B2B 물류 플랫폼을 위한 도메인 모델링 고도화

단순 커머스를 넘어 전국 단위 물류 네트워크를 관리하기 위한 MSA 기반의 플랫폼 설계(SA)를 진행하며 데이터 정합성과 운영 효율성을 동시에 고려했다. 

#### 1) 판매자별 관리 계층(`p_company_orders`) 도입

기존의 2단 구조(Order-Items)에서는 여러 판매자의 상품을 묶음 결제했을 때 부분 취소나 정산 처리가 어렵다는 피드백을 바탕으로, 이를 해결하기 위해 중간 계층을 삽입했다.
* **정산 및 취소 단위 확보**: 업체별 주문(`p_company_orders`) 단위로 금액을 합산하여 판매자 정산 로직을 명확히 함.
* **부분 취소 지원**: 특정 업체 상품만 전체 취소하거나 상태를 변경할 수 있는 유연한 구조 확보. 

### 2) SKU 단위의 정교한 재고 관리 (`p_warehouse_inventory`)

"검정/256GB"와 같은 옵션 조합 자체이 실제 물류 현장에서 관리되는 최소 단위(SKU)임 구두로 논의 했었는데, 이를 설계 문서에 누락한 것을 발견하였다.
문서에도 구체화해 추후 SA 문서를 보고서 작업할 수 있도록 보완했다. 

* **옵션 ID 참조**: 재고 테이블이 상품 ID가 아닌 옵션 조합 ID(`product_option_id`)를 직접 참조하도록 변경. 
* **동시성 제어**: 대량 발주 시 발생하는 데이터 충돌을 방지하기 위해 `version` 컬럼을 활용한 낙관적 락(Optimistic Lock) 적용. 

### 🚀 MSA 기반 서비스 분리 전략 (6대 마이크로서비스)

비즈니스 책임 범위와 확장성을 고려하여 시스템을 6개의 서비스로 분리하고, 각 서비스가 독립된 데이터베이스 스키마를 가지도록 설계했다.  
~~아직.. 보완할 부분이 남았는지 모르겠다. 부디 피드백 통화하게해주세요🙏🏻😭~~

| 서비스명 | 핵심 역할                                     | 주요 메커니즘                                        |
|:---|:------------------------------------------|:-----------------------------------------------|
| **User** | 회원가입 승인 및 역할별 권한 관리       | 승인 기반(PENDING/APPROVED) 프로세스   |
| **Hub** | 17개 허브 및 SKU 단위 재고 관리     | Redis 캐싱을 통한 허브 정보 조회 최적화      |
| **Company** | 입점 업체 및 상품 옵션 관리          | 업체 소속 허브 검증 및 마스터 데이터 관리       |
| **Order** | 결제 및 다단계 주문 계층 관리         | 임시 주문함(`p_order_drafts`) 기능 구현 |
| **Delivery** | 배송 경로 생성 및 매니저 배정         | Hub-and-Spoke 기반 최적 배송 시퀀스 생성  |
| **Operations** | 클레임, 알림(Slack), AI 분석 지원  | Gemini API 연동을 통한 최종 발송 시한 도출  |

### 🤷🏻‍♀️ TODO : 물류 도메인의 특화 로직 설계?! 

#### 1) 배송 경로 및 AI 최적화
- [ ] **P2P + Hub to Hub Relay**:
  - P2P (Direct): 인접 허브 간에는 중간 경유지 없이 직접 연결하여 최단 거리 배송 수행.
  - Hub-to-Hub Relay: 배송 거리가 일정 수준 이상인 경우, 중간 허브를 릴레이 방식으로 거쳐 최종 목적지까지 전달.
- [ ] **AI 발송 시한 예측**: Gemini API를 통해 배송 경로와 납품 기한을 분석하여 최적의 발송 시한을 도출하고 슬랙으로 알림. 

---
#내일배움캠프 #단기Java #TIL #MSA #물류시스템 #SA설계 #DomainModeling #B2B #SpringCloud #GeminiAPI #Redis #Architecture
