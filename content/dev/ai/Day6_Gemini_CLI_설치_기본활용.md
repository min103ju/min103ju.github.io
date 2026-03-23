---
title: "Day 6: Gemini CLI 설치 & 기본 활용"
date: 2026-03-10
tags:
  - gemini
  - cli
  - week2
---

# Day 6: Gemini CLI 설치 & 기본 활용

> AI Tools Mastery Curriculum — Week 2, Day 6
> 소요 시간: 45분 | 탐색 중심

---

## ① Gemini CLI 설치

### Gemini CLI란

Gemini CLI는 Google의 오픈소스 터미널 AI 에이전트다. Claude Code와 같은 포지션이지만, Google의 Gemini 모델을 사용한다.

**핵심 스펙:**
- 무료 티어: 60회/분, 1,000회/일 (Google 계정으로 로그인)
- 모델: Gemini 2.5 Pro (1M 토큰 컨텍스트 윈도우)
- 내장 도구: Google 검색, 파일 조작, 셸 명령, 웹 페치
- MCP 지원: Claude Code와 동일한 MCP 서버 연결 가능
- 오픈소스: Apache 2.0 라이선스

### 설치 방법

#### 방법 1: npx (설치 없이 즉시 실행)

```bash
npx @google/gemini-cli
```

설치 없이 바로 실행할 수 있어서 체험용으로 좋다.

#### 방법 2: 글로벌 설치

```bash
npm install -g @google/gemini-cli
```

설치 후 `gemini` 명령어로 실행한다.

#### 사전 요구사항

- Node.js 20 이상
- npm 또는 yarn

```bash
# Node.js 버전 확인
node --version  # v20 이상이어야 함

# 글로벌 설치
npm install -g @google/gemini-cli

# 설치 확인
gemini --version
```

### 인증 설정

#### 방법 1: Google 계정 로그인 (무료, 추천)

```bash
gemini
# 첫 실행 시 테마 선택 → "Login with Google" 선택 → 브라우저에서 로그인
```

무료 티어(60회/분, 1,000회/일)로 충분히 체험할 수 있다.

#### 방법 2: API 키 사용

```bash
# Google AI Studio에서 API 키 발급: https://aistudio.google.com/apikey
export GEMINI_API_KEY="YOUR_API_KEY"

# .zshrc 또는 .bashrc에 추가해두면 영구 적용
echo 'export GEMINI_API_KEY="YOUR_API_KEY"' >> ~/.zshrc
```

### 첫 실행 확인

```bash
gemini
# 테마 선택 후 프롬프트가 나타나면 성공
> 안녕, 간단히 자기소개 해줘
```

하단 상태 바에 모델명(Gemini 2.5 Pro 등)과 컨텍스트 사용량이 표시된다.

### 모델 확인 및 변경

v0.29.0부터 Gemini 3가 기본 모델이 되었다. Preview Features 설정은 더 이상 필요 없다.

```bash
gemini
# 하단 상태 바에서 현재 모델 확인 (Auto (Gemini 3) 등)

# 모델을 변경하고 싶을 때
> /model
# 목록에서 선택 (Gemini 3, Gemini 2.5 Pro 등)
```

---

## ② GEMINI.md 컨텍스트 파일 설정

### GEMINI.md란

GEMINI.md는 Claude Code의 CLAUDE.md에 해당하는 파일이다.
세션 시작 시 자동으로 로드되어 모든 프롬프트에 함께 전달된다.

**CLAUDE.md와의 핵심 차이점:**

| 항목 | CLAUDE.md | GEMINI.md |
|---|---|---|
| 전역 위치 | `~/.claude/CLAUDE.md` | `~/.gemini/GEMINI.md` |
| 프로젝트 위치 | 프로젝트 루트 | 프로젝트 루트 (+ 하위 디렉토리) |
| 하위 디렉토리 | 지원 | 지원 (하위 스캔이 더 적극적) |
| 모듈화 | 별도 파일 참조 불가 | `@file.md` 임포트 지원 |
| 확인 명령어 | 없음 (자동 로드) | `/memory show` |
| 동적 추가 | 없음 | `/memory add "텍스트"` |
| 리로드 | 세션 재시작 | `/memory refresh` |

### 계층 구조

GEMINI.md도 CLAUDE.md처럼 계층적으로 로드된다:

```
~/.gemini/GEMINI.md              ← 전역 (모든 프로젝트)
~/workspace/lms-backend/GEMINI.md    ← 프로젝트 루트
~/workspace/lms-backend/payment/GEMINI.md  ← 하위 모듈별
```

로딩 순서: 전역 → 프로젝트 루트 → 하위 디렉토리 (상위→하위 순서로 결합)

### GEMINI.md 작성

Week 1에서 만든 CLAUDE.md를 기반으로 GEMINI.md를 만든다.
내용은 동일하되, 파일명만 바꾸면 된다:

```markdown
# Online Learning Platform (LMS)

## Project Overview
Spring Boot 기반 온라인 학습 플랫폼 백엔드.
- /course-api: 강좌 생성/조회/수강신청 REST API
- /payment: 수강료 결제 (PG 연동, 환불 처리)
- /streaming: 영상 스트리밍 연동 (Vimeo OTT, AWS MediaConvert)
- /notification: 수강 알림 (이메일, 푸시 알림)

멀티 모듈 Gradle 프로젝트, Java 17, Spring Boot 3.2

## Coding Conventions
- DateTime: LocalDateTime 대신 OffsetDateTime 사용할 것
- WebClient 커넥션풀: 저트래픽 서비스는 maxConnections=10으로 제한
- @Transactional 안에서 외부 API 호출하지 말 것

## Build & Test
- ./gradlew build
- ./gradlew test
- ./gradlew integrationTest
```

### @import로 모듈화

GEMINI.md만의 고유 기능으로, 대규모 컨텍스트를 파일별로 분리할 수 있다:

```markdown
# GEMINI.md (프로젝트 루트)

# LMS Backend

## 공통 규칙
@./docs/coding-conventions.md
@./docs/api-guidelines.md

## 모듈별 컨텍스트
@./payment/PAYMENT_CONTEXT.md
@./streaming/STREAMING_CONTEXT.md
```

이 기능은 CLAUDE.md에는 없는 GEMINI.md만의 장점이다. 컨텍스트가 큰 프로젝트에서 관리가 편하다.

### 설정 확인 및 동적 관리

```bash
gemini

# 현재 로드된 컨텍스트 전체 보기
> /memory show

# 세션 중에 컨텍스트 추가
> /memory add "환불 API는 반드시 idempotency key를 포함해야 한다"

# GEMINI.md 수정 후 리로드
> /memory refresh
```

---

## ③ 동일 코드 분석 태스크 Claude vs Gemini 비교

### 비교 실험 설계

공정한 비교를 위해 **동일 조건**을 맞춘다:

```
동일한 프로젝트 디렉토리에서 실행
동일한 프롬프트 사용
동일한 컨텍스트 파일 (CLAUDE.md ↔ GEMINI.md 내용 동일)
```

### 비교 태스크 3가지

#### 태스크 1: 코드 분석

```
# Claude Code
cd ~/workspace/lms-backend
claude
> EnrollmentService 클래스를 분석해줘.
  의존 관계, 데이터 흐름, 잠재적 문제점을 정리해줘.

# Gemini CLI
cd ~/workspace/lms-backend
gemini
> EnrollmentService 클래스를 분석해줘.
  의존 관계, 데이터 흐름, 잠재적 문제점을 정리해줘.
```

**비교 포인트:**
- [ ] 코드베이스 탐색 범위 (얼마나 많은 파일을 읽었나)
- [ ] 분석의 깊이 (표면적 vs 구조적)
- [ ] 잠재적 문제점 발견 수
- [ ] 응답 속도

#### 태스크 2: 리팩토링 제안

```
> EnrollmentService에서 @Transactional 안에 외부 API 호출이 있는 부분을
  찾아서 리팩토링 계획을 세워줘. 코드는 작성하지 마.
```

**비교 포인트:**
- [ ] 문제 코드를 정확히 식별했는가
- [ ] 리팩토링 방향이 합리적인가
- [ ] 기존 코드 패턴을 존중했는가
- [ ] 리스크 분석을 포함했는가

#### 태스크 3: 테스트 생성

```
> CourseService의 createCourse() 메서드에 대한 단위 테스트를 작성해줘.
  정상 케이스, 유효성 실패, 권한 없음 3가지 시나리오를 포함해.
```

**비교 포인트:**
- [ ] 테스트 구조와 네이밍
- [ ] Mock 처리 방식
- [ ] 엣지 케이스 커버리지
- [ ] 컨벤션 준수 여부 (GEMINI.md/CLAUDE.md 반영)

### 비교 기록 템플릿

```markdown
## Claude Code vs Gemini CLI 비교 결과

### 태스크 1: 코드 분석
| 항목 | Claude Code | Gemini CLI |
|---|---|---|
| 탐색한 파일 수 | __ 개 | __ 개 |
| 분석 깊이 (1-5) | __ | __ |
| 발견한 문제점 수 | __ 개 | __ 개 |
| 응답 속도 | __ 초 | __ 초 |
| 특이사항 | | |

### 태스크 2: 리팩토링 제안
| 항목 | Claude Code | Gemini CLI |
|---|---|---|
| 문제 식별 정확도 (1-5) | __ | __ |
| 계획의 합리성 (1-5) | __ | __ |
| 리스크 분석 포함 | Yes / No | Yes / No |
| 특이사항 | | |

### 태스크 3: 테스트 생성
| 항목 | Claude Code | Gemini CLI |
|---|---|---|
| 코드 품질 (1-5) | __ | __ |
| 컨벤션 준수 (1-5) | __ | __ |
| 엣지 케이스 커버 | __ 개 | __ 개 |
| 바로 실행 가능 여부 | Yes / No | Yes / No |
| 특이사항 | | |

### 종합 소감
- Claude Code의 강점:
- Gemini CLI의 강점:
- 상황별 추천:
```

### 비교 시 관찰할 차별화 포인트

단순히 "어디가 더 좋다"보다, **각 도구의 특성**을 파악하는 게 목적이다:

**Gemini CLI의 차별점:**
- 1M 토큰 컨텍스트 (Claude Code 대비 큰 윈도우)
- Google 검색 내장 (웹 기반 리서치에 강점)
- `/memory add`로 세션 중 동적 컨텍스트 추가
- `@file.md` 임포트로 컨텍스트 모듈화
- 비대화형 모드: `gemini -p "프롬프트"`로 스크립트 자동화 가능
- 무료 티어가 넉넉 (60회/분)

**Claude Code의 차별점:**
- 서브 에이전트 (Task)로 병렬 처리
- Skills 시스템으로 재사용 가능한 지식 패키지
- Hooks로 이벤트 기반 자동화
- Plan Mode로 시스템 레벨 읽기 전용

### Gemini CLI 유용한 명령어 치트시트

비교 실험하면서 활용할 수 있는 Gemini CLI 고유 명령어:

```bash
# 슬래시 명령어
/memory show        # 현재 로드된 컨텍스트 확인
/memory refresh     # GEMINI.md 리로드
/memory add "텍스트" # 전역 컨텍스트에 추가
/model              # 모델 변경
/settings           # 설정 변경
/restore            # 파일 변경 롤백 (체크포인팅)
/chat save          # 대화 저장
/chat share         # 대화 공유용 마크다운 생성
/stats              # 세션 통계

# 비대화형 모드 (스크립팅)
gemini -p "이 프로젝트의 구조를 분석해줘"
gemini -p "프롬프트" --output-format json    # JSON 출력

# 파일/이미지 참조
> @src/main/java/com/example/CourseService.java 이 파일을 분석해줘
> @screenshot.png 이 에러 화면을 분석해줘

# 셸 명령 실행
> !ls -la src/                  # 단일 셸 명령
> !                             # 셸 모드 진입 (다시 !로 나가기)
```

---

## 실습 과제

### 과제 1: Gemini CLI 설치 및 설정 (10분)
1. `npm install -g @google/gemini-cli`
2. `gemini` 실행 → Google 로그인
3. `/model`로 현재 모델 확인 (Gemini 3가 기본)
4. 간단한 프롬프트로 동작 확인

### 과제 2: GEMINI.md 설정 (10분)
1. CLAUDE.md 내용을 기반으로 GEMINI.md 작성
2. 프로젝트 루트에 배치
3. `/memory show`로 로드 확인
4. `/memory add`로 동적 추가 테스트

### 과제 3: 비교 실험 (25분)
1. 3가지 태스크를 Claude Code → Gemini CLI 순서로 실행
2. 각 태스크별 비교 포인트 기록
3. 비교 매트릭스 초안 작성
4. "어떤 상황에서 어떤 도구가 더 나은가" 결론 정리

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| Gemini CLI | Google의 오픈소스 터미널 AI 에이전트, 무료 60회/분 |
| GEMINI.md | CLAUDE.md의 Gemini 버전, `@import`와 `/memory` 명령이 추가 기능 |
| 비교의 목적 | "어디가 더 좋다"가 아니라 "각 도구의 특성"을 파악하는 것 |
| Gemini 강점 | 1M 컨텍스트, Google 검색 내장, @import 모듈화, 넉넉한 무료 티어 |
| Claude 강점 | 서브 에이전트, Skills, Hooks, Plan Mode |
| 비대화형 모드 | `gemini -p "프롬프트"`로 스크립트/CI 자동화 가능 |

---

## 참고 리소스

- Gemini CLI 공식: [GitHub](https://github.com/google-gemini/gemini-cli)
- Gemini CLI 문서: [geminicli.com](https://geminicli.com)
- GEMINI.md 가이드: [Provide context with GEMINI.md](https://geminicli.com/docs/cli/gemini-md/)
- 튜토리얼 시리즈: [Gemini CLI Tutorial (Medium)](https://medium.com/google-cloud/gemini-cli-tutorial-series-77da7d494718)
- 치트시트: [Gemini CLI Cheatsheet](https://www.philschmid.de/gemini-cli-cheatsheet)
- Google Codelabs: [Gemini CLI Hands-on](https://codelabs.developers.google.com/gemini-cli-hands-on)
