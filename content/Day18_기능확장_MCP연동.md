# Day 18: 기능 확장 & MCP 연동

> AI Tools Mastery Curriculum — Week 4, Day 18
> 소요 시간: 50분 | 구현 + 연동

---

## 오늘 할 일

```
1. 나머지 도구 구현 (search, update)          — Claude Code
2. Claude Code에 MCP 서버 연동               — claude mcp add
3. 자연어로 북마크 관리 테스트                 — Claude Code
4. 멀티 에이전트로 코드 리뷰                   — 서브 에이전트
```

---

## ① 나머지 도구 구현 — search, update

### search_bookmarks 구현

```bash
claude

> "@.claude/CLAUDE.md를 읽고 search_bookmarks 도구를 구현해줘.
  
  - 키워드로 title, url, memo, tags를 검색
  - 대소문자 무시
  - 부분 일치 (includes)
  - 결과가 없으면 빈 배열 반환 (에러 아님)
  - src/tools/searchBookmarks.ts에 핸들러 작성
  - src/index.ts에 등록"
```

### update_bookmark 구현

```bash
> "update_bookmark 도구를 구현해줘.
  
  - 입력: id(필수), title?(선택), tags?(선택), memo?(선택)
  - url은 수정 불가 (의도적 제한)
  - 존재하지 않는 ID면 BookmarkNotFoundError
  - updatedAt 자동 갱신
  - src/tools/updateBookmark.ts에 핸들러 작성
  - src/index.ts에 등록"
```

### 빌드 & Inspector 테스트

```bash
npm run build
npx @modelcontextprotocol/inspector node build/index.js
```

Inspector에서 추가 테스트:

```
1. 북마크 3개 추가:
   - MCP 공식 사이트 (tags: mcp, protocol)
   - TypeScript 핸드북 (tags: typescript, tutorial)
   - Spring Boot 가이드 (tags: spring, java)

2. search_bookmarks 테스트:
   - query: "typescript" → TypeScript 핸드북 매칭
   - query: "tutorial" → TypeScript 핸드북 매칭 (태그 검색)
   - query: "없는키워드" → 빈 배열

3. update_bookmark 테스트:
   - 첫 번째 북마크의 tags에 "favorite" 추가
   - updatedAt이 갱신되었는지 확인

4. 전체 플로우:
   - add → list → search → update → list → delete
```

---

## ② Claude Code에 MCP 서버 연동

### 서버 등록

```bash
# bookmark-mcp-server를 Claude Code에 등록
claude mcp add bookmark-mcp-server \
  -- node /Users/mini/Documents/curriculum/bookmark/build/index.js
```

### 등록 확인

```bash
claude mcp list
# bookmark-mcp-server가 목록에 있는지 확인
```

---

## ③ 자연어로 북마크 관리 테스트

Claude Code에서 자연어로 북마크를 관리해본다. **Claude가 도구를 자동으로 선택하는지**가 핵심이다.

### 테스트 시나리오

```bash
claude

# 시나리오 1: 북마크 추가
> "이 URL을 북마크에 저장해줘: https://zod.dev
  제목은 'Zod 공식 문서'이고 태그는 typescript, validation으로."
# → Claude가 add_bookmark 도구 자동 호출

# 시나리오 2: 목록 조회
> "typescript 관련 북마크 보여줘"
# → Claude가 list_bookmarks(tag: "typescript") 호출

# 시나리오 3: 키워드 검색
> "validation 관련해서 저장해둔 거 있어?"
# → Claude가 search_bookmarks(query: "validation") 호출

# 시나리오 4: 수정
> "방금 추가한 Zod 북마크에 메모를 추가해줘: 'MCP 서버 입력 검증에 사용'"
# → Claude가 update_bookmark 호출

# 시나리오 5: 복합 작업
> "지금 저장된 북마크 중에서 tutorial 태그가 붙은 것들을 정리해줘.
  각각 제목, URL, 메모를 표로 만들어줘."
# → Claude가 list_bookmarks 또는 search_bookmarks 호출 후 표 작성

# 시나리오 6: 삭제
> "Zod 문서 북마크를 삭제해줘"
# → Claude가 search로 ID 찾고 → delete_bookmark 호출
```

### description이 잘 작동하는지 확인

Claude가 도구를 자동으로 선택하지 못하면 description을 개선해야 한다:

```
❌ 도구를 못 찾는 경우:
  → description이 너무 간결하거나 키워드가 부족

✅ 개선 방법 (Day 4에서 배운 적극적 description):
  "북마크를 추가합니다"
  → "새 URL을 북마크에 저장합니다. 북마크 추가, URL 저장,
     링크 저장, 페이지 저장을 요청할 때 사용합니다."
```

---

## ④ 멀티 에이전트로 코드 리뷰

Day 14에서 배운 Fan-out/Fan-in 패턴으로 프로젝트 전체를 리뷰한다.

### 병렬 코드 리뷰

```bash
claude

> "bookmark-mcp-server 프로젝트를 3개 관점에서 병렬 리뷰해줘.
  각각 서브 에이전트로 수행하고 결과를 종합해줘:

  1. 데이터 무결성: BookmarkStore의 JSON 파일 읽기/쓰기에서
     데이터 손실 가능성, 동시성 이슈, edge case 분석

  2. 도구 설계: 5개 도구의 description 품질, inputSchema 적절성,
     에러 메시지 사용자 친화성 분석

  3. 코드 품질: TypeScript 타입 안전성, 에러 처리 일관성,
     중복 코드 존재 여부 분석

  종합 결과를 심각도별(High/Medium/Low)로 정리해줘."
```

### 리뷰 결과 반영

서브 에이전트 리뷰에서 나온 개선사항을 바로 수정한다:

```bash
> "위 코드 리뷰 결과에서 High 심각도 이슈를 먼저 수정해줘."
```

---

## 실습 과제

### 과제 1: search + update 구현 (15분)
1. search_bookmarks, update_bookmark 도구 구현
2. Inspector에서 테스트
3. 전체 CRUD 플로우 확인

### 과제 2: Claude Code 연동 테스트 (20분)
1. `claude mcp add`로 서버 등록
2. 6가지 자연어 시나리오 테스트
3. 도구 자동 선택이 안 되는 경우 description 개선
4. 복합 작업 (검색 → 표 작성) 테스트

### 과제 3: 멀티 에이전트 코드 리뷰 (15분)
1. Fan-out/Fan-in으로 3관점 병렬 리뷰
2. High 심각도 이슈 수정
3. 수정 후 빌드 확인

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| 도구 완성 | 5개 도구 전부 구현 (add/list/search/update/delete) |
| Claude 연동 | `claude mcp add`로 등록 → 자연어로 관리 |
| 자동 선택 | description 품질이 도구 자동 선택의 핵심 |
| 복합 작업 | Claude가 여러 도구를 조합해서 작업 가능 |
| 코드 리뷰 | Fan-out/Fan-in으로 3관점 병렬 리뷰 |
