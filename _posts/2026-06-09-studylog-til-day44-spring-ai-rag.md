---
title: "[내일배움캠프 TIL, Day 44] Spring AI 활용 : ChatClient 계층 설계와 Naive RAG 구현 흐름"
excerpt: "Spring AI를 백엔드 서비스에 붙이는 방법, Naive RAG 두 시퀀스 이해"

categories:
  - Studylog
tags:
  - [TIL, SpringAI, RAG, ChatClient, NaiveRAG]

permalink: /studylog/til-day44-spring-ai-rag/

toc: true
toc_sticky: true

date: 2026-06-09
last_modified_at: 2026-06-09
---

## 1. 오늘 학습 키워드

* **Spring AI 전체 개발 흐름**: Chat → Structured Output → RAG → Tool Calling → Agent/MCP
* **AI Service 계층 분리**: AI 호출은 별도의 AI Service 계층으로 분리
* **ChatClient**: `user().call().content()` 체인으로 LLM을 호출하는 fluent API
* **Structured Output**: AI 응답을 Java DTO로 받는 흐름 (`entity(MyRecord.class)`)
* **RAG**: 모델 재학습 없이 질문 시점에 관련 문서를 검색해 컨텍스트로 주입하는 구조
* **Naive RAG 두 시퀀스**: 문서 적재(Document → Split → Embedding → VectorStore), 질문(Question → Embedding → 유사도 검색 → Context Injection → Answer)
* **QuestionAnswerAdvisor**: VectorStore 검색 결과를 프롬프트에 자동 주입하는 Advisor

---

## 2. 학습 내용 정리하기

### 💡 Spring AI 전체 개발 모델과 계층 설계

Spring AI는 LLM을 호출하는 단순 래퍼가 아니라, Spring 애플리케이션에서 AI 모델과 외부 지식을 연결하기 위한 추상화 계층이다. 구성요소들은 하나의 요청 처리 흐름 안에서 아래 순서로 연결된다.  
각 단계는 이전 단계를 기준선으로 기능을 하나씩 얹는 구조다.
```
Chat → Structured Output → RAG → Tool Calling → Agent/MCP
```


Spring AI를 서비스에 붙일 때는 **AI Service 계층**을 별도로 분리하는 것이 핵심이다.  
AI 호출은 외부 API 호출처럼 보이지만 프롬프트 정책, 응답 형식, 비용, 로그, 문서 검색 품질이 함께 얽혀 있기 때문이다.

```
[Client]
    ↓
[Controller]         HTTP 요청/응답
    ↓
[Application Service] 비즈니스 로직, 서비스 정책 검증
    ↓
[AI Service]         ChatClient + Advisor + Tool + VectorStore 조합
    ↓
[External]           Chat Model API / VectorStore / MCP Server
```

---

### 💡 Structured Output 변환 흐름

자연어 응답은 DB 저장, 로직 분기, Tool Calling 판단 등 후속 처리가 어렵다. Structured Output은 AI 응답을 Java 객체로 직접 받는 방식이다.

```
Prompt + Schema → LLM JSON 응답 → DTO Mapping → Validation → Service 분기
```

`chatClient.prompt().user(message).call().entity(MyRecord.class)` 한 줄로 처리된다.  
프롬프트에 출력 형식 조건(필드명, 허용값 등)을 명시하면 모델이 JSON 형태로 응답하고, Spring AI가 이를 지정한 클래스로 매핑해준다.

---

### 💡 Naive RAG: 두 개의 시퀀스

RAG(Retrieval Augmented Generation)는 모델을 재학습시키는 것이 아니라, 질문 시점에 관련 문서를 검색해 컨텍스트로 주입한 뒤 답변을 생성하는 구조다.  
사내 정책, 최신 공지처럼 모델이 학습하지 않은 정보에 대응할 수 있다.

Naive RAG는 두 시퀀스로 완전히 분리되어 있다.

**① 문서 적재 시퀀스** (1회성, 사전 작업)
```
Document → Split(청크 분할) → Embedding(벡터 변환) → VectorStore 저장
```

**② 질문 시퀀스** (매 요청마다)
```
사용자 질문 → Question Embedding → VectorStore 유사도 검색
           → 관련 문서 반환 → 프롬프트 컨텍스트 주입 → 답변 생성
```

Spring AI에서 각 단계를 담당하는 구성요소는 아래와 같다.

| 구성요소 | 역할 |
|---|---|
| DocumentReader | 문서 읽기 |
| TokenTextSplitter | 청크 분할 |
| EmbeddingModel | 벡터 변환 |
| VectorStore | 저장 / 검색 |
| QuestionAnswerAdvisor | 컨텍스트 주입 |
| ChatClient | 모델 호출 |

**QuestionAnswerAdvisor**는 ② 질문 시퀀스 전체를 자동으로 처리해주는 구성요소다.  
`ChatClient`에 Advisor로 붙이기만 하면, 사용자 질문 임베딩 → VectorStore 검색 → 프롬프트 주입이 ChatClient 호출 중간에 자동으로 끼어든다.

```
chatClient.prompt().user(question).call()
          ↑
          QuestionAnswerAdvisor가 중간에 개입:
          질문 임베딩 → VectorStore 검색 → 컨텍스트 주입 후 모델 호출
```

---

### 💡 Naive RAG의 한계와 Advanced RAG

Naive RAG는 구조가 단순한 만큼 한계도 명확하다.

* **질문 품질에 의존**: 사용자가 모호하거나 짧게 질문하면 유사도 검색 결과도 부정확해진다
* **고정된 청크 검색**: 유사도 기반 top-k 검색만 하기 때문에, 관련 문서가 있어도 검색에서 누락될 수 있다
* **빈 컨텍스트 문제**: 관련 문서가 없을 때 모델이 일반 지식으로 답변해 hallucination이 발생할 수 있다
* **후처리 없음**: 검색된 문서를 그대로 컨텍스트로 넣기 때문에 노이즈나 중복이 섞일 수 있다

Spring AI 공식 문서(1.1.6)에서는 이런 한계를 극복하기 위한 `RetrievalAugmentationAdvisor`를 제공한다. `QuestionAnswerAdvisor`가 Naive RAG용 단순 Advisor라면, `RetrievalAugmentationAdvisor`는 RAG 파이프라인을 모듈 단위로 조립할 수 있는 Advanced RAG용 구조다.

```
[Pre-Retrieval]  QueryTransformer (질문 재작성/압축/번역), MultiQueryExpander
      ↓
[Retrieval]      VectorStoreDocumentRetriever (유사도 검색)
      ↓
[Post-Retrieval] DocumentPostProcessor (re-ranking, 중복 제거)
      ↓
[Generation]     ContextualQueryAugmenter (컨텍스트 주입)
```

Naive RAG → Advanced RAG로 넘어가는 핵심 포인트는 **질문 전처리(Query Transformation)** 단계다. 사용자 질문을 그대로 검색에 쓰는 게 아니라, LLM이 먼저 질문을 더 검색하기 좋은 형태로 재작성한 뒤 VectorStore를 조회한다.


---

## 참고문헌

* [Spring AI Reference - Retrieval Augmented Generation](https://docs.spring.io/spring-ai/reference/api/retrieval-augmented-generation.html)
* [Spring AI Reference - Advisors API](https://docs.spring.io/spring-ai/reference/api/advisors.html)
* [Spring AI Reference - Chat Client API](https://docs.spring.io/spring-ai/reference/api/chatclient.html)
* [Spring AI Reference - Structured Output](https://docs.spring.io/spring-ai/reference/api/structured-output-converter.html)
* [내일배움캠프 Spring AI 특강 교안 - [260609] Spring AI 활용: RAG 구현](https://www.notion.so/teamsparta/260609-Spring-AI-RAG-3792dc3ef51480859d1ee2c9c6fd404c)

---

#내일배움캠프 #단기Java #TIL #SpringAI #RAG #ChatClient #StructuredOutput #NaiveRAG
