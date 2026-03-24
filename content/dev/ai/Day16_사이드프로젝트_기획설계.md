---
title: "Day 16: 사이드 프로젝트 기획 & 설계"
date: 2026-03-18
description: "Week 1~3 기술을 총동원하여 북마크 매니저 MCP 서버 사이드 프로젝트를 기획하고 설계한다."
tags:
  - side-project
  - planning
  - week4
---

# Day 16: 사이드 프로젝트 기획 & 설계

> AI Tools Mastery Curriculum — Week 4, Day 16
> 소요 시간: 50분 | 기획 중심


> [!summary] 핵심 배운 점
> - 북마크 매니저 MCP 서버: JSON 기반 CRUD + 5개 도구
> - 체이닝(아이디어→PRD→일정) + NotebookLM(리뷰) 조합
> - Plan Mode로 아키텍처 설계 후 CLAUDE.md 작성


---

## Week 4 목표

Week 1~3에서 배운 기술을 총동원해서 **북마크 매니저 MCP 서버**를 기획→구현→테스트한다.

### Week 4 계획

```
Day 16: 기획 & 설계
  ├─ Claude로 PRD 작성
  ├─ NotebookLM으로 PRD 리뷰 & 개선
  └─ 아키텍처 설계 & CLAUDE.md 작성

Day 17: 코어 기능 구현 — AI 페어 프로그래밍
  ├─ Claude Code로 CRUD 핵심 구현
  ├─ Codex CLI로 테스트 코드 작성
  └─ MCP Inspector로 도구 검증

Day 18: 기능 확장 & MCP 연동
  ├─ 검색/필터 도구 추가
  ├─ Claude Code에 연동
  └─ 멀티 에이전트로 코드 리뷰

Day 19: 테스트 & 최종 정리
  ├─ 에러 처리 강화
  ├─ 문서화 & README
  └─ 최종 회고
```

### 이번 주에 활용할 기술

```
Week 1:
  ├─ CLAUDE.md (Day 1) → 프로젝트 컨텍스트
  ├─ 프롬프트 체이닝 (Day 2) → PRD 작성
  ├─ 서브 에이전트 (Day 3) → 병렬 코드 리뷰
  ├─ Skills (Day 4) → 테스트 자동화
  └─ Hooks (Day 4) → 빌드 후 자동 포맷팅

Week 2:
  ├─ Codex CLI (Day 7) → 테스트 코드 작성
  ├─ NotebookLM (Day 8) → PRD 리뷰 & 개선
  └─ 도구 조합 (Day 10) → 상황별 최적 도구 선택

Week 3:
  ├─ MCP 서버 제작 (Day 11~13) → 새로운 MCP 서버 구축
  └─ 멀티 에이전트 (Day 14) → Fan-out 코드 리뷰
```

---

## ① 아이디어 구체화 & Claude로 PRD 작성

### 프로젝트: 북마크 매니저 MCP 서버

```
핵심 컨셉:
  개발 중 발견한 유용한 URL을 저장/태그/검색하는 MCP 서버.
  Claude Code에서 "React 관련 북마크 찾아줘"라고 하면
  MCP 도구가 검색해서 결과를 반환한다.

특징:
  - DB 없이 JSON 파일 기반 저장
  - CRUD API (생성/조회/수정/삭제)
  - 태그 기반 검색 & 필터링
  - Claude가 자연어로 검색/관리
```

### MCP 도구 명세

| 도구 | 설명 | 입력 | 출력 |
|---|---|---|---|
| `add_bookmark` | 북마크 추가 | url, title, tags[], memo? | 생성된 북마크 |
| `list_bookmarks` | 전체/태그별 목록 조회 | tag?, limit? | 북마크 배열 |
| `search_bookmarks` | 키워드 검색 | query | 매칭된 북마크 배열 |
| `update_bookmark` | 북마크 수정 | id, title?, tags?, memo? | 수정된 북마크 |
| `delete_bookmark` | 북마크 삭제 | id | 삭제 확인 |

### Claude로 PRD 작성 (프롬프트 체이닝)

**Step 1: 아이디어 구체화**

```bash
claude

> "북마크 매니저 MCP 서버를 만들려고 해.
  
  핵심 기능:
  - URL 저장/태그/검색 (JSON 파일 기반, DB 없음)
  - MCP 도구로 Claude Code에서 자연어 관리
  
  다음을 정리해줘:
  - 이 프로젝트가 해결하는 문제
  - 핵심 사용자 시나리오 3가지
  - 구현해야 할 MCP 도구 목록
  - 4일 안에 가능한 MVP 범위"
```

**Step 2: PRD 문서 생성**

```bash
> "위 내용을 기반으로 PRD를 작성해줘.
  
  포함할 항목:
  1. 프로젝트 개요 (한 줄 설명)
  2. 문제 정의
  3. 핵심 기능 (MVP 범위)
  4. MCP 도구 명세 (이름, 설명, 입력/출력)
  5. 데이터 구조 (JSON 스키마)
  6. 일정 (Day 16~19 각각 할 일)
  
  docs/PRD.md로 저장해줘."
```

**Step 3: 일정 검증**

```bash
> "PRD의 일정을 검토해줘.
  각 Day가 50분 이내에 완료 가능한지 확인하고,
  과한 부분이 있으면 MVP에서 빼줘."
```

### 데이터 구조

```json
// data/bookmarks.json
{
  "bookmarks": [
    {
      "id": "bm_1710403200",
      "url": "https://modelcontextprotocol.io/docs/develop/build-server",
      "title": "MCP 서버 구축 가이드",
      "tags": ["mcp", "typescript", "tutorial"],
      "memo": "Week 3에서 참고한 공식 튜토리얼",
      "createdAt": "2026-03-14T10:00:00.000Z",
      "updatedAt": "2026-03-14T10:00:00.000Z"
    }
  ]
}
```

---

## ② NotebookLM으로 PRD 리뷰 & 개선

### NotebookLM 활용: 기술 리서치가 아닌 "PRD 리뷰어"

기술 리서치는 아이디어에 따라 Claude나 웹 검색으로 충분하다. NotebookLM의 진짜 강점은 **여러 문서를 크로스레퍼런스하며 분석**하는 것이므로, PRD 리뷰 & 개선에 활용한다.

### 워크플로우

```
1. NotebookLM 노트북 생성: "북마크 매니저 PRD 리뷰"

2. 소스 업로드:
   - docs/PRD.md (방금 작성한 PRD)
   - MCP 공식 Best Practices URL
     (https://modelcontextprotocol.io/docs/develop/build-server)
   - 유사 프로젝트 참고 URL 2~3개
     (GitHub에서 bookmark 관련 MCP 서버 검색)

3. PRD 리뷰 질문:
   - "PRD의 도구 명세에 빠진 edge case는 없는지 분석해줘"
   - "MCP Best Practices 기준으로 이 PRD의 도구 설계가 적절한지 평가해줘"
   - "유사 프로젝트 대비 차별화할 수 있는 포인트는?"
   - "JSON 파일 기반 저장의 한계와 대안을 정리해줘"

4. Audio Overview 생성:
   - PRD 리뷰 결과를 오디오로 변환
   - 이동 중에 리뷰 내용 복습하며 개선 아이디어 메모

5. 리뷰 결과 반영:
   - NotebookLM이 지적한 빠진 edge case를 PRD에 추가
   - 도구 설계 개선점 반영
   - docs/prd-review.md로 리뷰 결과 저장
```

### 기술 리서치 대신 PRD 리뷰가 적합한 이유

```
기술 리서치:
  → Claude Code나 웹 검색으로 바로 가능
  → NotebookLM에 넣고 빼는 과정이 오히려 번거로움
  → 아이디어에 맞는 기술은 Claude에게 직접 물어보는 게 빠름

PRD 리뷰:
  → 여러 문서(PRD + Best Practices + 유사 프로젝트)를 
     크로스레퍼런스하며 분석하는 게 NotebookLM의 강점
  → Audio Overview로 리뷰 결과를 이동 중 복습 가능
  → "이 PRD에서 뭘 놓치고 있지?"를 다각도로 검토
```

---

## ③ 아키텍처 설계 & CLAUDE.md 작성

### 아키텍처 설계

Claude Code에서 Plan Mode로 설계한다:

```bash
claude
# Shift+Tab×2로 Plan Mode 진입

> "@docs/PRD.md를 읽고 프로젝트 아키텍처를 설계해줘:

  1. 디렉토리 구조
  2. 각 파일의 역할
  3. 데이터 흐름 (사용자 → Claude → MCP → JSON 파일)
  4. JSON 파일 읽기/쓰기 동시성 처리 방법
  5. 에러 처리 전략
  
  docs/architecture.md로 저장해줘."
```

### 디렉토리 구조

```
bookmark-mcp-server/
├── docs/
│   ├── PRD.md                ← 기획 문서
│   ├── prd-review.md         ← NotebookLM 리뷰 결과
│   └── architecture.md       ← 아키텍처 문서
├── data/
│   └── bookmarks.json        ← 북마크 데이터 (JSON 파일 저장소)
├── src/
│   ├── index.ts              ← McpServer + 도구 등록 + 서버 시작
│   ├── tools/                ← 도구 핸들러 (파일별 분리)
│   │   ├── addBookmark.ts
│   │   ├── listBookmarks.ts
│   │   ├── searchBookmarks.ts
│   │   ├── updateBookmark.ts
│   │   └── deleteBookmark.ts
│   ├── store/                ← JSON 파일 읽기/쓰기
│   │   └── bookmarkStore.ts
│   ├── errors.ts             ← 커스텀 에러
│   └── logger.ts             ← 로깅
├── .claude/
│   └── CLAUDE.md             ← 프로젝트 컨텍스트
├── package.json
└── tsconfig.json
```

### BookmarkStore 설계

DB 대신 JSON 파일로 CRUD를 구현하는 핵심 모듈:

```typescript
// src/store/bookmarkStore.ts 설계 (Day 17에서 구현)

export interface Bookmark {
  id: string;
  url: string;
  title: string;
  tags: string[];
  memo?: string;
  createdAt: string;
  updatedAt: string;
}

export class BookmarkStore {
  // JSON 파일에서 전체 북마크 로드
  async getAll(): Promise<Bookmark[]>

  // ID로 단일 조회
  async getById(id: string): Promise<Bookmark | null>

  // 새 북마크 추가 → JSON 파일에 저장
  async add(data: Omit<Bookmark, 'id' | 'createdAt' | 'updatedAt'>): Promise<Bookmark>

  // 북마크 수정 → JSON 파일 갱신
  async update(id: string, data: Partial<Bookmark>): Promise<Bookmark>

  // 북마크 삭제 → JSON 파일에서 제거
  async delete(id: string): Promise<boolean>

  // 태그로 필터링
  async findByTag(tag: string): Promise<Bookmark[]>

  // 키워드 검색 (title, url, memo, tags 대상)
  async search(query: string): Promise<Bookmark[]>
}
```

### CLAUDE.md

```markdown
# CLAUDE.md

## WHAT: 프로젝트 개요
북마크 매니저 MCP 서버 — URL을 저장/태그/검색하는 MCP 서버.
Claude Code에서 자연어로 북마크를 관리할 수 있다.
DB 없이 JSON 파일 기반 저장.

## WHY: 기술 스택 & 의사결정
- TypeScript + MCP SDK (v1.x)
- JSON 파일 기반 저장 (data/bookmarks.json)
- stdio 전송 (로컬 실행, Claude Code 연동)
- DB를 쓰지 않는 이유: MVP 범위, 설정 최소화

## HOW: 코딩 컨벤션

### 파일 구조
- tools/ 에 도구 핸들러 분리 (파일당 1개 도구)
- store/ 에 BookmarkStore 클래스 (JSON 파일 읽기/쓰기)
- index.ts는 등록과 서버 시작만 담당

### 도구 구현 규칙
- server.registerTool() 사용 (server.tool() deprecated)
- inputSchema는 zod로 정의
- 응답은 JSON.stringify(data, null, 2)
- 에러 시 isError: true 반환

### JSON 파일 저장 규칙
- 읽기: fs.readFile → JSON.parse
- 쓰기: JSON.stringify → fs.writeFile
- ID 생성: `bm_${Date.now()}`
- 날짜: ISO 8601 형식 (new Date().toISOString())

### 검색 규칙
- 대소문자 무시 (toLowerCase())
- title, url, memo, tags 모두 검색 대상
- 부분 일치 (includes)

### 에러 처리
- BookmarkNotFoundError: ID로 조회 실패
- DuplicateUrlError: 같은 URL 중복 등록 시
- InvalidUrlError: URL 형식 검증 실패

### 로깅
- console.log() 금지 (STDIO 서버)
- console.error() 또는 log() 유틸리티 사용

### 빌드 & 테스트
- npm run build (tsc 컴파일)
- npx @modelcontextprotocol/inspector로 도구 테스트
- claude mcp add로 Claude Code 연동 테스트
```

---

## 실습 과제

### 과제 1: PRD 작성 (20분)
1. Claude에게 체이닝으로 PRD 작성
2. 도구 명세 5개 + 데이터 구조 확인
3. 일정 검증 (Day 17~19 각각 50분 가능한지)
4. docs/PRD.md 저장

### 과제 2: NotebookLM PRD 리뷰 (15분)
1. NotebookLM에 PRD + MCP 공식 문서 업로드
2. edge case, 도구 설계 적절성 질문
3. 리뷰 결과를 PRD에 반영
4. docs/prd-review.md 저장

### 과제 3: 아키텍처 & CLAUDE.md (15분)
1. Plan Mode로 아키텍처 설계
2. docs/architecture.md 저장
3. CLAUDE.md 작성 (WHAT/WHY/HOW)
4. 디렉토리 구조 생성

```bash
mkdir -p bookmark-mcp-server/{docs,data,src/{tools,store},.claude}
cd bookmark-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript
echo '{"bookmarks":[]}' > data/bookmarks.json
```

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| 프로젝트 | 북마크 매니저 MCP 서버 (JSON 파일 기반 CRUD) |
| MCP 도구 | add, list, search, update, delete 5개 |
| NotebookLM | 기술 리서치가 아닌 PRD 리뷰 & 크로스레퍼런스 분석 |
| 아키텍처 | tools/ 분리 + store/ (BookmarkStore) + errors.ts |
| CLAUDE.md | WHAT/WHY/HOW, JSON 파일 저장 규칙 포함 |
| 데이터 | data/bookmarks.json, ID는 `bm_${Date.now()}` |
