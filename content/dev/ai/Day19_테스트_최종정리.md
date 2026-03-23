---
title: "Day 19: 테스트 & 최종 정리"
date: 2026-03-19
description: "북마크 매니저의 에러 처리를 강화하고, 19일간의 AI 도구 마스터리 커리큘럼을 최종 정리한다."
tags:
  - testing
  - final-review
  - week4
---

# Day 19: 테스트 & 최종 정리

> AI Tools Mastery Curriculum — Week 4, Day 19
> 소요 시간: 50분 | 마무리 중심

---

## 오늘 할 일

```
1. 에러 처리 강화 & edge case 보완      — Claude Code
2. README & 문서화                     — Claude Code
3. 4주 커리큘럼 최종 회고               — 정리
```

---

## ① 에러 처리 강화 & edge case 보완

### Codex CLI로 edge case 탐색

Day 7에서 배운 Codex CLI의 read-only 모드로 코드를 분석한다:

```bash
codex -s read-only

> "bookmark-mcp-server의 모든 도구 핸들러를 분석하고,
  처리되지 않은 edge case를 목록으로 알려줘.
  
  확인할 항목:
  - URL 형식이 잘못된 경우 (http/https 없는 URL 등)
  - tags가 빈 배열인 경우
  - title이 빈 문자열인 경우
  - 매우 긴 문자열 입력
  - JSON 파일이 손상된 경우
  - JSON 파일이 없는 경우 (최초 실행)"
```

### Claude Code로 보완

Codex가 찾아낸 edge case를 Claude Code로 수정한다:

```bash
claude

> "다음 edge case를 처리해줘:

  1. JSON 파일이 없는 경우:
     - 최초 실행 시 data/bookmarks.json이 없으면 자동 생성
     - 빈 배열로 초기화

  2. URL 검증 강화:
     - http:// 또는 https://로 시작하는지 확인
     - 빈 문자열 거부

  3. 입력값 정리:
     - title 앞뒤 공백 제거 (trim)
     - tags 중복 제거
     - 빈 태그 문자열 필터링

  4. JSON 파일 손상 대비:
     - JSON.parse 실패 시 빈 배열로 초기화하고 경고 로그"
```

### 빌드 & 최종 테스트

```bash
npm run build

# Inspector로 edge case 테스트
npx @modelcontextprotocol/inspector node build/index.js
```

```
테스트 체크리스트:
  □ 빈 title로 add → 에러 메시지 확인
  □ http 없는 URL로 add → InvalidUrlError 확인
  □ 같은 URL 중복 add → DuplicateUrlError 확인
  □ 없는 ID로 update → BookmarkNotFoundError 확인
  □ 없는 ID로 delete → BookmarkNotFoundError 확인
  □ 빈 검색어로 search → 전체 반환 또는 에러
  □ 특수문자 포함 검색 → 정상 동작
  □ data/bookmarks.json 삭제 후 실행 → 자동 생성
```

---

## ② README & 문서화

### Claude Code로 README 작성

```bash
claude

> "이 프로젝트의 README.md를 작성해줘.
  GitHub에 올릴 수 있는 수준으로:

  1. 프로젝트 소개 (한 줄 설명 + 핵심 기능)
  2. 설치 방법 (npm install, 빌드)
  3. 사용 방법
     - Claude Code에 연결하는 법
     - 사용 가능한 도구 5개와 예시
  4. 개발 (빌드, 테스트, Inspector)
  5. 프로젝트 구조 (디렉토리 트리)
  6. 라이선스"
```

### 최종 파일 구조 확인

```bash
> "프로젝트 전체 파일 목록을 보여주고,
  각 파일의 역할을 한 줄로 설명해줘."
```

완성된 프로젝트 구조:

```
bookmark-mcp-server/
├── README.md               ← 프로젝트 소개 & 사용 방법
├── package.json
├── tsconfig.json
├── docs/
│   ├── PRD.md              ← 기획 문서
│   ├── prd-review.md       ← NotebookLM 리뷰 결과
│   └── architecture.md     ← 아키텍처 문서
├── data/
│   └── bookmarks.json      ← 북마크 데이터
├── src/
│   ├── index.ts            ← MCP 서버 진입점 + 도구 등록
│   ├── tools/
│   │   ├── addBookmark.ts      ← 북마크 추가
│   │   ├── listBookmarks.ts    ← 목록 조회
│   │   ├── searchBookmarks.ts  ← 키워드 검색
│   │   ├── updateBookmark.ts   ← 북마크 수정
│   │   └── deleteBookmark.ts   ← 북마크 삭제
│   ├── store/
│   │   └── bookmarkStore.ts    ← JSON 파일 CRUD
│   ├── errors.ts           ← 커스텀 에러 클래스
│   └── logger.ts           ← 로깅 유틸리티
├── tests/
│   └── bookmarkStore.test.ts   ← 단위 테스트
├── build/                  ← 컴파일 결과
└── .claude/
    └── CLAUDE.md           ← 프로젝트 컨텍스트
```

---

## ③ 4주 커리큘럼 최종 회고

### 4주간 배운 것 한눈에

```
Week 1: Claude Code 심화
  Day 1:  CLAUDE.md (WHAT/WHY/HOW)
  Day 2:  프롬프트 체이닝 & Plan Mode
  Day 3:  서브 에이전트 & /clear, /compact
  Day 4:  Skills & Hooks
  Day 5:  Week 1 회고

Week 2: AI 도구 생태계
  Day 6:  Gemini CLI (1M 컨텍스트, 무료)
  Day 7:  Codex CLI (샌드박스, /review)
  Day 8:  NotebookLM (문서→오디오, Q&A)
  Day 9:  Cursor (Agent/Plan/Debug/Ask 4모드)
  Day 10: 5개 도구 비교 분석

Week 3: MCP 서버 제작
  Day 11: MCP 프로토콜 구조 & SDK
  Day 12: MCP 서버 v1 (OpenWeather 연동)
  Day 13: MCP 서버 v2 (도구 확장, 에러 처리, 캐싱)
  Day 14: 멀티 에이전트 오케스트레이션
  Day 15: Week 3 종합 정리

Week 4: 사이드 프로젝트
  Day 16: 기획 & 설계 (PRD, 아키텍처, CLAUDE.md)
  Day 17: 코어 구현 (BookmarkStore + MCP 도구 3개 + Codex 테스트)
  Day 18: 기능 확장 & 연동 (search/update + Claude Code 연동 + 코드 리뷰)
  Day 19: 마무리 (edge case + README + 회고)
```

### 4주 전과 후 비교

```
Before (Day 0):
  - Claude Code 단일 프롬프트만 사용
  - AI 도구 = "질문하면 답해주는 것"
  - MCP가 뭔지 모름

After (Day 19):
  - CLAUDE.md + Skills + Hooks로 Claude Code 커스터마이징
  - 5개 AI 도구의 강점을 파악하고 상황별로 선택
  - MCP 서버를 직접 만들어서 Claude Code에 연동
  - 멀티 에이전트로 병렬 작업 설계
  - AI를 활용한 기획→구현→테스트→문서화 전체 사이클 경험
```

### 핵심 인사이트 5가지

**1. CLAUDE.md가 모든 것의 시작이다**
AI 도구의 품질은 컨텍스트에 달려있다. CLAUDE.md 하나로 응답 품질이 극적으로 달라진다.

**2. 도구마다 강점이 다르다**
Claude Code(심층 분석), Gemini CLI(넓은 컨텍스트), Codex CLI(안전한 자동화), Cursor(시각적 편집), NotebookLM(문서 소화). 최고의 도구는 없고 최적의 조합이 있다.

**3. MCP는 데이터를 제공하고 해석은 LLM이 한다**
MCP 서버가 "서울은 따뜻하니 반팔 입으세요"라고 할 필요 없다. 기온 데이터만 주면 Claude가 알아서 답변한다.

**4. 멀티 에이전트의 핵심은 컨텍스트 분리다**
병렬 처리보다 중요한 건 각 에이전트가 깨끗한 컨텍스트에서 작업하는 것이다.

**5. 하드코딩 먼저, 연동 나중**
외부 API든 MCP든, 가짜 데이터로 구조를 검증한 뒤에 실제 연동하면 디버깅 시간이 크게 줄어든다.

### 다음 단계 추천

4주 커리큘럼을 마친 후 도전할 수 있는 것들:

```
단기 (1~2주):
  - 북마크 매니저에 Streamable HTTP 전송 추가 (원격 서버화)
  - 다른 서비스 연동 MCP 서버 제작 (GitHub, Notion 등)
  - 팀 내 "코드 허심탄회"에서 AI 도구 활용 발표

중기 (1~2개월):
  - Agent Teams(실험적) 활용한 멀티 에이전트 워크플로우
  - MCP 서버를 npm 패키지로 배포
  - 실무 프로젝트에 CLAUDE.md + Skills + Hooks 적용

장기:
  - 사내 MCP 서버 생태계 구축 (Jira, Confluence, 사내 API)
  - AI 도구 활용 가이드 사내 공유
  - 오픈소스 MCP 서버 커뮤니티 기여
```

---

## 실습 과제

### 과제 1: edge case 보완 (15분)
1. Codex CLI read-only로 edge case 분석
2. Claude Code로 보완 구현
3. Inspector로 edge case 테스트 체크리스트 통과

### 과제 2: README & 문서화 (15분)
1. Claude Code로 README.md 작성
2. 최종 파일 구조 확인
3. 프로젝트 전체 빌드 & 테스트 통과

### 과제 3: 회고 작성 (20분)
1. 4주간 가장 임팩트 있었던 학습 3가지 정리
2. "Before → After" 비교 작성
3. 다음 단계 계획 수립
4. 블로그 또는 사내 공유용 회고문 초안 작성

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| edge case | Codex(분석) → Claude Code(수정) 조합 |
| 문서화 | README는 GitHub 공개 수준으로 |
| 최종 산출물 | MCP 서버 + 5개 도구 + 테스트 + 문서 |
| 4주 핵심 | CLAUDE.md → 도구 조합 → MCP 제작 → 실전 적용 |
| 다음 단계 | Streamable HTTP, npm 배포, 실무 적용 |
