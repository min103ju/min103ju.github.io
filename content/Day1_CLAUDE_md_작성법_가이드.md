# Day 1: CLAUDE.md 작성법 & 프로젝트 컨텍스트 설계

> AI Tools Mastery Curriculum — Week 1, Day 1
> 소요 시간: 40분 | 실습 중심

---

## ① CLAUDE.md 구조 학습 (WHAT/WHY/HOW 프레임워크)

CLAUDE.md는 Claude Code가 세션 시작 시 자동으로 읽는 파일이다.
매번 새 세션마다 Claude는 프로젝트에 대해 아무것도 모르는 상태에서 시작하기 때문에, 이 파일이 유일한 "영구 기억"이 된다.

### 핵심 원칙 3가지

**첫째, 150줄 이내로 유지한다.**
Claude Code의 시스템 프롬프트 자체가 약 50개의 지시를 포함하고 있고, frontier 모델이 안정적으로 따를 수 있는 지시는 150~200개 수준이다.
CLAUDE.md에 지시를 과도하게 넣으면 전체 지시 따르기 품질이 균일하게 하락한다.
즉, 아래쪽 지시만 무시하는 게 아니라 **모든 지시를 균일하게 덜 따르게** 된다.

**둘째, "Claude가 자주 틀리는 것"을 적는다.**
모든 명령어, 모든 컨벤션을 나열하려 하지 말고, Claude가 실제로 실수하는 부분만 기록하는 것이 훨씬 효과적이다.
매뉴얼이 아니라 **"실수 교정 노트"** 에 가깝다.

**셋째, 구체적인 대안을 함께 제시한다.**
"~하지 마라"만 쓰면 Claude가 막히게 된다. 반드시 대안을 함께 써야 한다.

```
❌ --foo-bar 플래그를 사용하지 마라
✅ --foo-bar 대신 --baz를 사용해라
```

### WHAT / WHY / HOW 프레임워크

```markdown
# 프로젝트명

## WHAT (이 프로젝트는 무엇인가)
- 기술 스택, 프로젝트 구조, 모노레포 맵
- 핵심 디렉토리와 역할
- 의존성 관계

## WHY (왜 이렇게 되어 있는가)
- 각 모듈의 목적과 비즈니스 맥락
- 아키텍처 결정 배경 (ADR)
- 도메인 용어 정의

## HOW (어떻게 작업하는가)
- 빌드/테스트/린트 명령어
- 코딩 컨벤션 (Claude가 틀리는 것 위주)
- 검증 방법 (변경 후 확인 절차)
```

각 섹션의 **분량 배분**이 중요하다.
많은 사람들이 WHAT에 너무 많은 비중을 두는데, 실제로 가장 임팩트가 큰 건 HOW 섹션이다.
Claude는 코드를 읽으면서 WHAT은 어느 정도 파악할 수 있지만, 팀 고유의 작업 방식은 알려주지 않으면 알 수 없기 때문이다.

> **권장 비율: WHAT 30% / WHY 20% / HOW 50%**

---

## ② 실무 프로젝트에 CLAUDE.md 작성

### 작성 예시 (글로벌 배송/클레임 시스템 기준)

```markdown
# Musinsa Global Shipping & Claims System

## WHAT
Spring Boot 기반 글로벌 배송/클레임 처리 시스템.
- /global-shipping: MFS 글로벌 배송 정책 API, 로컬 배포 처리
- /claims: 클레임 접수/처리 시스템 (PHP 레거시 + Spring Boot 신규)
- /integration: 외부 연동 (T-Mall, Zozotown, LX Pantos, Flexport)

Jira 프로젝트: GPRD (Global Product Delivery), CLM (Claims)
Atlassian workspace: musinsa-oneteam

## WHY
- 트랜잭션 경계와 외부 API 호출을 분리하는 구조
  → 외부 API 실패가 DB 트랜잭션을 롤백시키지 않도록
- LogisticsBusinessType이 API별로 다르게 동작함 주의
- PHP 레거시 클레임 시스템은 점진적으로 Spring Boot로 이관 중

## HOW

### 빌드 & 테스트
- ./gradlew build
- ./gradlew test (단위테스트만)
- ./gradlew integrationTest (통합테스트, 로컬 DB 필요)

### 코딩 컨벤션
- DateTime: LocalDateTime 대신 OffsetDateTime 사용할 것
  → 타임존 직렬화 이슈 방지 (Asia/Seoul UTC+9)
- WebClient 커넥션풀: 저트래픽 서비스는 maxConnections=10으로 제한
- JSON 컬럼 쿼리: MySQL JSON_EXTRACT 사용 시 인덱스 미적용 주의

### 변경 검증
1. 단위 테스트 통과 확인
2. API 변경 시 Swagger 문서 업데이트
3. 외부 API 연동 변경 시 시퀀스 다이어그램 업데이트

### 하지 말 것
- @Transactional 안에서 외부 API 호출하지 말 것
  → 별도 서비스 메서드로 분리 후 호출
- Caffeine 캐시 TTL을 5분 이상 설정하지 말 것
  → feature flag 시스템과 충돌 가능
```

### 계층적 활용

CLAUDE.md는 여러 위치에 둘 수 있다:

```
~/.claude/CLAUDE.md              ← 개인 전역 (모든 프로젝트 공통)
~/workspace/project/CLAUDE.md    ← 프로젝트 루트
~/workspace/project/claims/CLAUDE.md  ← 하위 모듈별
```

전역 파일에는 개인 선호(한국어 응답, IntelliJ 사용 등)를 넣고,
프로젝트 파일에는 기술적 내용을 넣는 구조가 좋다.

### 전역 CLAUDE.md 예시

```markdown
# 개인 설정
- 응답은 한국어로
- IDE: IntelliJ IDEA
- 터미널: macOS zsh
- Git: 커밋 메시지는 Conventional Commits (feat/fix/refactor)
- 테스트 실행 후 결과를 항상 보여줄 것
```

---

## ③ 적용 전/후 응답 품질 비교

이 단계가 가장 중요하다. 실제로 차이를 체감해야 CLAUDE.md를 유지하는 동기가 생긴다.

### 비교 실험 방법

**Step 1: CLAUDE.md 없이 동일 프롬프트 실행**

```bash
# CLAUDE.md가 없는 상태에서 시작
cd ~/Documents/workspace/your-project
claude
> 배송 상태 변경 API에 Zozotown 주문 상태 동기화 기능을 추가해줘
```

이때 Claude의 응답을 관찰한다. 아마 이런 점들이 보일 것이다:

- `LocalDateTime`을 사용할 가능성이 높음
- `@Transactional` 안에서 외부 API를 호출할 수 있음
- 커넥션풀 설정이 기본값(무제한)일 수 있음
- 기존 코드 패턴과 다른 스타일로 작성

**Step 2: CLAUDE.md 적용 후 동일 프롬프트 실행**

```bash
# CLAUDE.md를 프로젝트 루트에 배치한 후
claude
> /clear
> 배송 상태 변경 API에 Zozotown 주문 상태 동기화 기능을 추가해줘
```

**Step 3: 비교 포인트 체크**

| 비교 항목 | CLAUDE.md 없이 | CLAUDE.md 적용 후 |
|---|---|---|
| DateTime 타입 | LocalDateTime 사용 | OffsetDateTime 사용 |
| 트랜잭션 구조 | 외부 API 호출이 @Transactional 내부 | 트랜잭션과 외부 호출 분리 |
| WebClient 설정 | 기본값 또는 임의 설정 | maxConnections=10 명시 |
| 에러 처리 | 일반적인 try-catch | 프로젝트 패턴에 맞는 처리 |
| 코드 스타일 | Claude 기본 스타일 | 프로젝트 컨벤션 반영 |

### 스크린샷 캡처 & 기록 템플릿

```markdown
## CLAUDE.md 적용 전/후 비교 기록

### 테스트 프롬프트
"배송 상태 변경 API에 Zozotown 주문 상태 동기화 기능을 추가해줘"

### Before (CLAUDE.md 없음)
- [스크린샷]
- 문제점: LocalDateTime 사용, 트랜잭션 내 외부 API 호출

### After (CLAUDE.md 적용)
- [스크린샷]
- 개선점: OffsetDateTime 사용, 서비스 분리, 컨벤션 준수

### 체감 효과
- 수정이 필요한 부분: X개 → Y개로 감소
- "이건 고쳐야겠다" 싶은 포인트 비율: 약 N% 감소
```

### 추가 비교 프롬프트 추천

하나의 프롬프트로만 비교하면 우연일 수 있으니, 2~3개 더 시도해본다:

```
"클레임 환불 처리 로직에서 관세 계산을 리팩토링해줘"
"MFS 글로벌 배송 정책 API에 대한 단위 테스트를 작성해줘"
"T-Mall 해외 배송 라벨 업데이트 폴링 로직의 에러 처리를 개선해줘"
```

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| CLAUDE.md의 본질 | 매뉴얼이 아니라 "Claude가 실수하는 것을 바로잡는 노트" |
| 분량 | 150줄 이내, HOW 섹션에 비중 50% |
| 유지 방법 | Claude가 틀리는 새로운 패턴을 발견할 때마다 추가 |
| 계층 활용 | 전역(개인 선호) + 프로젝트(기술) + 모듈(상세) |
| 검증 | 동일 프롬프트로 적용 전/후 비교, 스크린샷 기록 |

---

## 참고 리소스

- Anthropic 공식: [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- HumanLayer 블로그: [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- GitHub 레퍼런스: [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- Anthropic Docs: [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
