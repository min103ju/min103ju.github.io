---
title: "Day 10: AI 도구 비교 분석"
date: 2026-03-14
tags:
  - 비교분석
  - 회고
  - week2
---

# Day 10: AI 도구 비교 분석

> AI Tools Mastery Curriculum — Week 2, Day 10
> 소요 시간: 30분 | 정리 중심

---

## ① 5개 도구 비교표 완성

### 도구 분류

이번 주에 체험한 5개 도구는 세 가지 카테고리로 나뉜다:

```
터미널 코딩 에이전트:  Claude Code · Gemini CLI · Codex CLI
AI 네이티브 IDE:      Cursor
문서 기반 리서치 도구: NotebookLM
```

같은 카테고리 안에서는 직접 비교가 의미 있지만, 카테고리가 다르면 **대체가 아니라 조합**의 관점으로 봐야 한다.

### 전체 비교표

| 항목 | Claude Code | Gemini CLI | Codex CLI | Cursor | NotebookLM |
|---|---|---|---|---|---|
| **분류** | 터미널 에이전트 | 터미널 에이전트 | 터미널 에이전트 | AI IDE | 리서치 도구 |
| **개발사** | Anthropic | Google | OpenAI | Cursor Inc. | Google |
| **기반 모델** | Claude Sonnet/Opus | Gemini 3 | GPT-5.2~5.4 Codex | 멀티 모델 선택 | Gemini |
| **컨텍스트 윈도우** | 200K 토큰 | 1M 토큰 | 200K 토큰 | 모델별 상이 | 소스당 50만 단어 |
| **오픈소스** | ❌ | ✅ (Apache 2.0) | ✅ (MIT) | ❌ | ❌ |
| **무료 사용** | ❌ (API 키 필요) | ✅ (60회/분) | 한시적 프로모션 | 제한적 Hobby | ✅ (일일 한도) |
| **유료 가격** | Pro $20/월 | AI Pro $19.99/월 | Plus $20/월 | Pro $20/월 | Pro $19.99/월 |

### 터미널 에이전트 심층 비교

| 항목 | Claude Code | Gemini CLI | Codex CLI |
|---|---|---|---|
| **컨텍스트 파일** | CLAUDE.md | GEMINI.md | AGENTS.md |
| **컨텍스트 파일 특징** | 계층적 로딩 | @import 모듈화 지원 | TOML 설정과 분리 |
| **서브 에이전트** | Task() 내장 (최대 10개 병렬) | ❌ | Multi-agent 지원 |
| **Skills** | .claude/skills/ (SKILL.md) | ❌ | $skill-name |
| **Hooks** | settings.json (이벤트 기반) | ❌ | Rules (정책 기반) |
| **샌드박스** | 승인 기반 | 별도 설정 필요 | OS 레벨 (기본 내장) |
| **Plan Mode** | Shift+Tab×2 (시스템 레벨) | ❌ | /plan (프롬프트 레벨) |
| **코드 리뷰** | 수동 요청 | 수동 요청 | /review 내장 |
| **웹 검색** | 서브 에이전트 위임 | Google 검색 내장 | 캐시 기반 (기본) |
| **비대화형 모드** | claude -p | gemini -p | codex exec |
| **컨텍스트 관리** | /clear, /compact | /memory refresh | /compact, /clear |
| **MCP 지원** | ✅ | ✅ (Extensions) | ✅ |
| **IDE 연동** | ❌ (터미널 전용) | VS Code 동반자 | VS Code/Cursor 확장 |
| **JetBrains 지원** | ❌ | ❌ | ❌ (ACP 준비 중) |

### Cursor와 터미널 에이전트 비교

| 항목 | 터미널 에이전트 (Claude Code 등) | Cursor |
|---|---|---|
| **인터페이스** | 터미널 텍스트 | 에디터 GUI + diff 뷰 |
| **코드 변경 확인** | 텍스트 출력으로 확인 | 시각적 diff로 파일별 적용/취소 |
| **코드베이스 탐색** | grep, glob 기반 | 인덱싱 기반 시맨틱 검색 |
| **디버깅** | 텍스트 로그 분석 | Debug 모드 (가설 기반) |
| **자동완성** | ❌ | Tab (실시간, 프로젝트 인식) |
| **병렬 에이전트** | 서브 에이전트 (Claude) | Cloud Agent 최대 8개 |
| **학습 곡선** | 터미널 익숙하면 낮음 | VS Code 익숙하면 낮음 |
| **Spring/Java** | 코드를 읽고 분석 | VS Code 수준 (IntelliJ 대비 약함) |

### NotebookLM의 고유 영역

NotebookLM은 코딩 도구가 아니므로 직접 비교 대상이 아니다. 다른 도구와 **조합**해서 쓸 때 가치가 있다:

| 항목 | NotebookLM | 코딩 에이전트/IDE |
|---|---|---|
| **입력** | PDF, URL, YouTube, 오디오 | 코드베이스 |
| **출력** | 요약, Q&A, 오디오 팟캐스트 | 코드, 커밋, 테스트 |
| **강점** | 여러 소스 크로스레퍼런스 | 실제 코드 수정/실행 |
| **오디오 생성** | ✅ (팟캐스트 스타일) | ❌ |
| **소스 기반 답변** | ✅ (출처 명확) | 일반 지식 기반 |
| **무료** | ✅ (일일 채팅 50회, 오디오 3회) | 도구별 상이 |

---

## ② 상황별 최적 도구 추천 매트릭스

### 작업 유형별 추천

| 작업 | 1순위 추천 | 2순위 추천 | 이유 |
|---|---|---|---|
| **코드 분석 & 리뷰** | Claude Code | Codex CLI (/review) | 서브 에이전트로 컨텍스트 격리 분석, Codex는 내장 리뷰 기능 |
| **멀티 파일 리팩토링** | Cursor (Agent) | Claude Code | Cursor는 diff 뷰가 직관적, Claude는 서브 에이전트로 대규모 분석 |
| **새 코드 생성** | Cursor (Agent) | Claude Code | Cursor는 프로젝트 인덱싱으로 기존 패턴 반영, Agent 자율 실행 |
| **테스트 작성** | Claude Code | Cursor (Agent) | Skills로 테스트 패턴 표준화 가능, Cursor도 멀티 파일 생성 가능 |
| **대규모 코드베이스 탐색** | Claude Code | Gemini CLI | 서브 에이전트 병렬 탐색이 핵심, Gemini는 1M 컨텍스트 |
| **빠른 프로토타이핑** | Cursor (Agent) | Codex CLI (--full-auto) | Cursor는 시각적 확인, Codex는 완전 자동화 |
| **CI/CD 자동화** | Codex CLI (exec) | Gemini CLI (-p) | 비대화형 모드 + 샌드박스, Gemini는 무료 |
| **기술 문서 학습** | NotebookLM | - | 유일한 문서→오디오 변환 도구, 소스 기반 Q&A |
| **디버깅** | IntelliJ (디버거) | Cursor (Debug) | 실제 디버거는 대체 불가, Cursor Debug는 보조 |
| **컨벤션 강제** | Claude Code (Hooks) | Cursor (.cursor/rules/) | Hooks는 시스템 레벨 보장, Rules는 프롬프트 레벨 |

### 비용 최적화 관점

| 예산 | 추천 조합 | 월 비용 |
|---|---|---|
| **무료** | Gemini CLI + NotebookLM | $0 |
| **최소 비용** | Claude Code (Pro) + NotebookLM | $20 |
| **균형** | Claude Code (Pro) + Cursor (Pro) + NotebookLM | $40 |
| **올인** | Claude Code + Cursor + Codex + NotebookLM | $60+ |

### 개발 단계별 추천 워크플로우

```
1. 기술 조사/학습 단계
   NotebookLM: 공식 문서 업로드 → Q&A → 오디오로 통근 학습

2. 설계/분석 단계  
   Claude Code (Plan Mode): 코드베이스 분석, 서브 에이전트 병렬 탐색
   또는 Cursor (Plan 모드): 시각적으로 코드 탐색

3. 구현 단계
   Cursor (Agent): 멀티 파일 편집, diff로 시각적 확인
   Claude Code: 터미널에서 빠른 구현, Skills로 반복 작업 표준화

4. 테스트 단계
   Claude Code: Skills로 테스트 패턴 자동화
   Codex CLI: --full-auto로 테스트 실행 + 자동 수정

5. 리뷰 단계
   Codex CLI: /review 내장 코드 리뷰
   Claude Code: 커스텀 에이전트(@code-reviewer)로 리뷰

6. 배포/CI 단계
   Codex CLI: exec 모드로 CI 파이프라인 통합
   Gemini CLI: -p 모드로 스크립트 자동화 (무료)
```

### 나만의 도구 조합 결정 가이드

아래 질문에 답하면 자신에게 맞는 조합이 나온다:

```
Q1. 주 개발 IDE는?
  → IntelliJ: Claude Code(터미널 보조) + NotebookLM(학습)
  → VS Code: Cursor(메인) + Claude Code(복잡한 작업)

Q2. 예산은?
  → 무료만: Gemini CLI + NotebookLM
  → $20/월: Claude Code Pro + NotebookLM
  → $40/월: Claude Code + Cursor + NotebookLM

Q3. 주요 작업은?
  → 코드 분석/리팩토링 위주: Claude Code (서브 에이전트 + Skills)
  → 새 기능 구현 위주: Cursor (Agent + diff 뷰)
  → CI/자동화 위주: Codex CLI (exec + --full-auto)
  → 학습/문서 소화 위주: NotebookLM (오디오 + Q&A)
```

---

## 실습 과제

### 과제 1: 비교표 완성 (15분)
1. 이번 주 체험 결과를 바탕으로 위 비교표의 빈칸 채우기
2. 자신의 체험 기반으로 별점(1-5) 또는 한줄평 추가
3. "이건 비교표에 없지만 중요하다" 싶은 항목 추가

### 과제 2: 나만의 도구 조합 결정 (15분)
1. 결정 가이드 3가지 질문에 답하기
2. "내 메인 도구"와 "보조 도구" 확정
3. 개발 단계별 워크플로우 자신의 실무에 맞게 커스터마이징

---

## Week 2 전체 회고

### 이번 주 배운 것

```
Day 6:  Gemini CLI — 무료 + 1M 컨텍스트 + @import
Day 7:  Codex CLI — OS 레벨 샌드박스 + /review + --full-auto
Day 8:  NotebookLM — 문서→오디오 + 소스 기반 Q&A
Day 9:  Cursor — AI IDE + Agent/Plan/Debug/Ask 4모드
Day 10: 종합 비교 — 대체가 아니라 조합
```

### 핵심 인사이트

**"최고의 도구"는 없다. "최적의 조합"이 있다.**

각 도구는 고유한 강점이 있고, 그 강점이 빛나는 상황이 다르다:
- 터미널에서 깊이 파고들 때: Claude Code
- 넓은 컨텍스트로 빠르게 훑을 때: Gemini CLI
- 안전하게 자동화할 때: Codex CLI
- 시각적으로 확인하며 편집할 때: Cursor
- 기술 문서를 소화할 때: NotebookLM

### 다음 주 예고

Week 3에서는 **소비자에서 생산자로** 전환한다.
MCP 프로토콜을 이해하고, 나만의 MCP 서버를 직접 만들고, 멀티 에이전트 오케스트레이션까지.

---

## 참고 리소스

### 각 도구 공식 사이트
- Claude Code: [code.claude.com](https://code.claude.com)
- Gemini CLI: [github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
- Codex CLI: [developers.openai.com/codex/cli](https://developers.openai.com/codex/cli/)
- Cursor: [cursor.com](https://cursor.com)
- NotebookLM: [notebooklm.google.com](https://notebooklm.google.com)

### 비교 참고
- [LLM API Pricing March 2026 (TLDL)](https://www.tldl.io/resources/llm-api-pricing-2026) — 모델별 가격 비교
- [Cursor in JetBrains (ACP)](https://cursor.com/changelog) — 2026.3.4 공지
- [Codex Pricing](https://developers.openai.com/codex/pricing/) — 플랜별 상세
