# Day 8: NotebookLM으로 기술 문서 학습

> AI Tools Mastery Curriculum — Week 2, Day 8
> 소요 시간: 40분 | 탐색 중심

---

## ① 기술 문서 업로드

### NotebookLM이란

NotebookLM은 Google의 AI 기반 리서치 도구다. 문서를 업로드하면 그 문서에 대한 **전문가 AI**가 생성되어 질문에 답하고, 요약하고, 심지어 팟캐스트 형태의 오디오로 변환해준다.

코딩 에이전트(Claude Code, Gemini CLI, Codex)와는 완전히 다른 카테고리의 도구다:

```
코딩 에이전트: 코드를 읽고, 수정하고, 실행한다
NotebookLM:   문서를 읽고, 이해하고, 설명한다
```

**핵심 스펙:**
- 무료 (Google 계정만 있으면 사용 가능, 일일 사용량 제한 있음)
- 최대 50개 소스 업로드 가능
- 지원 형식: PDF, Google Docs, Google Slides, 텍스트, 웹 URL, YouTube 링크, 오디오, 이미지
- Audio Overview: 문서를 팟캐스트 스타일 오디오로 변환
- 50개 이상 언어 지원 (오디오 포함)
- Interactive Mode: 오디오 재생 중 질문으로 개입 가능

### 무료 플랜 제한 사항

무료로 충분히 쓸 수 있지만, 일일 한도를 알고 있어야 효율적으로 활용할 수 있다:

| 항목           | 무료 플랜                    | Pro ($19.99/월) |
| ------------ | ------------------------ | -------------- |
| 소스 1개당 글자    | **50만 단어** (약 200페이지 분량) | 동일             |
| 소스 1개당 파일 크기 | **200MB**                | 동일             |
| 노트북당 소스 수    | **50개**                  | 300개           |
| 노트북 수        | **100개**                 | 500개           |
| 일일 채팅 쿼리     | **50회**                  | 500회           |
| 일일 오디오 생성    | **3회**                   | 20회            |

**실용적 팁:**
- 일일 채팅 50회가 실질적 병목이다. 질문을 미리 정리해서 효율적으로 쓰자.
- 오디오 생성 3회/일이므로, 주제를 잘 골라서 한 번에 좋은 결과를 뽑자.
- Spring Boot 공식 문서 PDF는 보통 50만 단어 이내이므로 소스 제한에는 걸리지 않는다.
- YouTube 영상은 길이 제한 없음 (자막이 50만 단어 넘지 않는 한).

### 접속 방법

```
1. https://notebooklm.google.com 접속
2. Google 계정으로 로그인
3. "새 노트북" 클릭
```

### 기술 문서 업로드 실습

백엔드 개발자가 NotebookLM에 업로드하면 유용한 문서 유형:

#### 유형 1: 공식 프레임워크 문서

```
추천 소스:
- Spring Boot 공식 문서 (Reference Documentation)
  → PDF로 다운로드하거나 웹 URL 직접 입력
- Spring Data JPA 레퍼런스
- Spring Security 아키텍처 가이드
```

업로드 방법:
```
1. "소스 추가" 클릭
2. "웹사이트" 선택 → URL 붙여넣기
   예: https://docs.spring.io/spring-boot/reference/
3. 또는 "PDF" 선택 → 다운로드한 문서 업로드
```

#### 유형 2: 기술 블로그/아티클

```
추천 소스:
- Baeldung의 Spring Boot 튜토리얼
- Martin Fowler의 아키텍처 아티클
- InfoQ의 기술 트렌드 리포트
```

#### 유형 3: YouTube 기술 강연

```
추천 소스:
- Spring I/O 컨퍼런스 영상
- InfoQ 기술 발표 영상
- YouTube URL을 직접 소스로 추가하면 자동 트랜스크립트 분석
```

#### 유형 4: 내부 설계 문서 (주의)

```
사내 문서를 업로드할 때는 보안 정책을 반드시 확인한다.
NotebookLM은 Google 서버에서 처리되므로,
기밀 문서는 업로드하지 않는 것이 원칙이다.

대안: 공개된 기술 문서, 오픈소스 프로젝트 문서만 업로드
```

### 효과적인 노트북 구성 전략

노트북 하나에 관련 문서를 묶어서 구성한다:

```
📓 노트북 1: "Spring Boot 심화"
   ├─ Spring Boot Reference (PDF)
   ├─ Spring Data JPA Guide (URL)
   └─ Baeldung: Transaction Management (URL)

📓 노트북 2: "시스템 설계 패턴"
   ├─ Martin Fowler: Patterns of Enterprise Architecture (PDF)
   ├─ 마이크로서비스 패턴 아티클 3개 (URL)
   └─ InfoQ: Event-Driven Architecture 발표 (YouTube)

📓 노트북 3: "Kafka & 이벤트 아키텍처"
   ├─ Apache Kafka 공식 문서 (PDF)
   ├─ Confluent Blog 아티클 2개 (URL)
   └─ Spring Kafka 레퍼런스 (URL)
```

이렇게 주제별로 노트북을 나누면 AI가 해당 주제에 대해 더 정확한 답변을 한다.

---

## ② 오디오 요약 생성 & 통근시 청취

### Audio Overview란

NotebookLM의 가장 인상적인 기능이다. 업로드한 문서를 **두 명의 AI 호스트가 대화하는 팟캐스트 형태**로 변환해준다. 단순한 TTS(텍스트 음성 변환)가 아니라, "음...", "아하!" 같은 자연스러운 추임새까지 포함된 실제 팟캐스트 수준의 오디오가 생성된다.

### 오디오 생성 방법

```
1. 노트북에 소스를 1개 이상 업로드
2. 우측 "Notebook guide" 패널 확인
3. "Audio Overview" 섹션에서 "Generate" 클릭
4. 생성까지 수 분 소요 (소스 양에 따라 다름)
5. 재생 또는 다운로드
```

### 오디오 커스터마이징

생성 전에 다음을 조정할 수 있다:

| 옵션 | 설정값 | 용도 |
|---|---|---|
| 길이 | Brief(2-3분) / Standard(5-6분) / Extended(8-10분) | 통근 시간에 맞춰 조절 |
| 전문성 수준 | Novice / Intermediate / Expert | 배경지식 수준에 맞춤 |
| 집중 영역 | 특정 토픽 지정 가능 | "Transaction Management에 집중해줘" |
| 톤 | Casual / Analytical / Educational | 선호 스타일 |
| 출력 언어 | 50개+ 언어 | 한국어 선택 가능 |

### 커스터마이징 예시

```
"Spring Boot의 @Transactional 동작 원리에 집중해줘.
 Expert 수준으로, 전파 속성(Propagation)과 
 격리 수준(Isolation Level)을 깊이 다뤄줘."
```

이렇게 하면 Transaction 관련 내용만 집중적으로 다루는 오디오가 생성된다.

### Interactive Mode (대화형 모드)

오디오 재생 중에 **질문으로 개입**할 수 있다:

```
1. 오디오 재생 중 "Join" 버튼 클릭
2. 텍스트 또는 음성으로 질문 입력
3. AI 호스트가 질문에 답한 후 다시 본론으로 복귀
```

예시:
```
[AI 호스트가 @Transactional의 REQUIRED 전파에 대해 설명 중]

당신: "REQUIRES_NEW와 NESTED의 차이점을 좀 더 설명해줘"

[AI 호스트가 해당 질문에 답한 후 이어서 진행]
```

수동적 청취를 **능동적 학습**으로 바꿔주는 기능이다.

### 통근 활용 워크플로우

```
전날 저녁:
  1. 다음날 학습할 주제의 문서를 NotebookLM에 업로드
  2. Audio Overview 생성 (Extended, Expert)
  3. 다운로드

다음날 통근:
  1. 이어폰으로 오디오 청취
  2. 궁금한 부분 메모 (음성 메모 or 노트 앱)

출근 후:
  1. NotebookLM에서 궁금했던 부분 Q&A
  2. 또는 Claude Code에서 해당 내용 실습
```

---

## ③ Q&A 기반 학습 + 실무 문제 해결 시나리오

### Q&A 기반 학습

NotebookLM의 진짜 강점은 **업로드한 문서에 근거한 Q&A**다.
ChatGPT나 Claude가 일반 지식으로 답하는 것과 달리, NotebookLM은 **업로드한 소스에서만 답변**을 생성한다. 출처가 명확하다.

### 학습 시나리오 1: Spring Boot 트랜잭션 심화

```
📓 노트북: "Spring Transaction"
소스: Spring Framework Reference - Transaction Management (PDF/URL)

Q&A 예시:

Q: "@Transactional의 propagation 속성별로 동작이 어떻게 다른지 
    표로 정리해줘"
→ 문서 기반 정확한 표 생성

Q: "REQUIRED와 REQUIRES_NEW를 쓸 때 각각의 롤백 범위를 
    구체적으로 설명해줘"
→ 문서에서 관련 부분을 인용하며 설명

Q: "외부 API 호출이 @Transactional 안에 있을 때 발생할 수 있는 
    문제를 이 문서 기준으로 설명해줘"
→ 문서 내용과 연결해서 문제점 분석
```

### 학습 시나리오 2: JPA N+1 문제 해결

```
📓 노트북: "JPA Performance"
소스: 
  - Spring Data JPA Reference (URL)
  - Baeldung: JPA N+1 Problem (URL)
  - Vlad Mihalcea: High-Performance Java Persistence (PDF 일부)

Q&A 예시:

Q: "N+1 문제가 발생하는 모든 케이스를 정리해줘"
→ 소스에서 OneToMany, ManyToOne, EntityGraph 관련 내용 종합

Q: "fetch join과 @EntityGraph의 차이점을 이 문서들에서 찾아서 비교해줘"
→ 여러 소스를 크로스레퍼런스해서 비교

Q: "@BatchSize vs fetch join, 어떤 상황에서 뭘 써야 할지 
    이 문서들의 권장사항을 정리해줘"
→ 각 소스의 권장사항을 종합해서 가이드 생성
```

### 학습 시나리오 3: 신기술 빠르게 파악하기

```
📓 노트북: "Virtual Threads"
소스:
  - JEP 444: Virtual Threads (URL)
  - Spring Blog: Virtual Threads in Spring Boot 3.2 (URL)
  - InfoQ: Virtual Threads Deep Dive 발표 (YouTube)

Q&A 예시:

Q: "Virtual Thread를 Spring Boot에서 활성화하는 방법과 
    주의사항을 정리해줘"
→ 공식 문서 + 블로그 + 발표 내용을 종합

Q: "기존 Thread Pool 방식과 Virtual Thread의 성능 차이를 
    이 소스들에서 언급한 벤치마크를 기반으로 정리해줘"
→ 여러 소스의 벤치마크 데이터 종합

Q: "Virtual Thread 사용 시 synchronized 블록의 pinning 문제를 
    설명해줘"
→ 소스에서 관련 경고/주의사항 추출
```

### 실무 문제 해결에 활용

NotebookLM은 단순 학습뿐 아니라 **실무 문제를 해결할 때 레퍼런스 조사 도구**로도 쓸 수 있다:

```
상황: WebClient에서 간헐적 ConnectionTimeout 발생

1. 관련 문서 업로드
   - Spring WebClient Reference
   - Reactor Netty Connection Pool 문서
   - "WebClient Connection Pool Tuning" 관련 블로그 3개

2. 문제 상황 기반 Q&A
   Q: "WebClient의 기본 커넥션 풀 설정은 어떻게 되고,
       저트래픽 서비스에서 커넥션이 만료되는 시나리오를 설명해줘"
   
   Q: "이 문서들에서 권장하는 저트래픽 서비스의 
       WebClient 커넥션 풀 최적 설정은?"
   
   Q: "maxIdleTime과 evictInBackground 설정의 관계를 설명해줘"

3. 답변을 바탕으로 실제 코드 수정 (Claude Code에서)
```

이렇게 **NotebookLM으로 조사 → Claude Code로 구현**하는 워크플로우가 된다.

### NotebookLM의 부가 기능

오디오와 Q&A 외에도 유용한 기능들:

| 기능 | 설명 | 활용 |
|---|---|---|
| **Study Guide** | 소스 기반 학습 가이드 자동 생성 | 새 기술 학습 시 |
| **Flashcards** | 핵심 개념 플래시카드 생성 | 반복 학습 |
| **Briefing Doc** | 소스를 요약한 브리핑 문서 생성 | 팀 공유, 회의 준비 |
| **Timeline** | 소스 내 이벤트를 시간순 정리 | 기술 히스토리 파악 |
| **FAQ** | 소스 기반 FAQ 자동 생성 | 온보딩 문서화 |

---

## NotebookLM vs 코딩 에이전트 비교

| 항목 | NotebookLM | Claude Code / Gemini / Codex |
|---|---|---|
| 목적 | 문서 이해 & 학습 | 코드 작성 & 실행 |
| 입력 | PDF, URL, YouTube 등 | 코드베이스 |
| 출력 | 요약, Q&A, 오디오, 가이드 | 코드, 커밋, 테스트 |
| 강점 | 여러 소스의 크로스레퍼런스 | 실제 코드 수정/실행 |
| 오디오 | AI 팟캐스트 생성 | 없음 |
| 비용 | 완전 무료 | 유료 (Gemini CLI 제외) |
| 조합 활용 | 조사/학습 단계 | 구현/실행 단계 |

**최적의 조합: NotebookLM(조사/학습) → Claude Code(구현)**

---

## 실습 과제

### 과제 1: 노트북 구성 & 문서 업로드 (10분)
1. https://notebooklm.google.com 접속
2. "Spring Boot 심화" 노트북 생성
3. 소스 3개 이상 업로드 (공식 문서 URL + 블로그 + YouTube)
4. 자동 생성된 요약 확인

### 과제 2: Audio Overview 생성 (10분)
1. "Audio Overview" 생성 클릭
2. 커스터마이징: Expert 수준, 특정 주제 집중
3. 생성된 오디오 재생 & 품질 평가
4. 다운로드해서 통근용 저장
5. (가능하면) Interactive Mode로 질문 개입 체험

### 과제 3: Q&A 기반 학습 (20분)
1. 학습 시나리오 1~3 중 하나를 선택
2. 최소 5개 질문으로 심층 Q&A 수행
3. 답변의 정확성을 공식 문서와 대조 확인
4. 실무 문제 해결 시나리오 1건 수행
5. "NotebookLM으로 조사 → Claude Code로 구현" 워크플로우 체험

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| NotebookLM 본질 | 문서 기반 전문가 AI (코딩 에이전트와 다른 카테고리) |
| Audio Overview | 문서를 팟캐스트 스타일 오디오로 변환, 통근 활용에 최적 |
| Interactive Mode | 오디오 재생 중 질문으로 개입 가능 (수동→능동 학습) |
| Q&A 강점 | 업로드한 소스에서만 답변, 출처가 명확 |
| 조합 워크플로우 | NotebookLM(조사/학습) → Claude Code(구현) |
| 노트북 전략 | 주제별로 분리하면 답변 품질이 올라감 |
| 비용 | 무료 (일일 채팅 50회, 오디오 3회 제한. Pro $19.99/월로 확장 가능) |

---

## 참고 리소스

- NotebookLM: [notebooklm.google.com](https://notebooklm.google.com)
- Google 공식 블로그: [Audio Overviews 소개](https://blog.google/technology/ai/notebooklm-audio-overviews/)
- 튜토리얼: [NotebookLM Guide (DataCamp)](https://www.datacamp.com/tutorial/notebooklm)
- 활용 가이드: [NotebookLM Tutorial (YUV.AI)](https://yuv.ai/learn/notebooklm)
