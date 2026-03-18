# Day 17: 코어 기능 구현 — AI 페어 프로그래밍

> AI Tools Mastery Curriculum — Week 4, Day 17
> 소요 시간: 50분 | 구현 중심

---

## 오늘 할 일

```
1. BookmarkStore 구현 (JSON 파일 CRUD)      — Claude Code
2. MCP 도구 등록 (add, list, delete)         — Claude Code
3. Inspector로 도구 검증                     — MCP Inspector
4. 테스트 코드 작성                          — Codex CLI
```

---

## ① BookmarkStore 구현 — Claude Code

Day 16에서 설계한 BookmarkStore를 Claude Code로 구현한다.

### 프롬프트 체이닝으로 구현

```bash
cd bookmark-mcp-server
claude

> "@.claude/CLAUDE.md를 읽고, @docs/PRD.md를 참고해서
  src/store/bookmarkStore.ts를 구현해줘.
  
  BookmarkStore 클래스:
  - JSON 파일(data/bookmarks.json) 읽기/쓰기
  - getAll, getById, add, update, delete, findByTag, search 메서드
  - ID 생성: bm_${Date.now()}
  - 에러 시 커스텀 에러 throw
  
  src/errors.ts도 같이 만들어줘:
  - BookmarkNotFoundError
  - DuplicateUrlError
  - InvalidUrlError"
```

### 구현 결과 확인 포인트

Claude가 생성한 코드에서 확인할 것:

```
✅ fs.readFile / fs.writeFile 사용 (비동기)
✅ JSON.parse / JSON.stringify 처리
✅ ID 생성 패턴: bm_${Date.now()}
✅ 날짜: new Date().toISOString()
✅ 검색: toLowerCase() + includes()
✅ 중복 URL 체크 (add 시)
✅ 에러 클래스 분리 (errors.ts)
```

### logger.ts 구현

```bash
> "src/logger.ts를 만들어줘.
  STDIO 서버이므로 console.error()로 stderr에 JSON 형식으로 출력.
  Day 13에서 만든 패턴과 동일하게."
```

---

## ② MCP 도구 등록 — Claude Code

### Day 17에서 구현할 도구: add, list, delete (3개)

Day 18에서 search, update를 추가할 예정이므로, 오늘은 핵심 CRUD 3개만 집중한다.

```bash
> "@.claude/CLAUDE.md를 읽고, BookmarkStore를 사용해서
  src/index.ts에 MCP 서버를 구현해줘.
  
  오늘 등록할 도구 3개:
  
  1. add_bookmark
     - 입력: url(필수), title(필수), tags(배열), memo(선택)
     - URL 형식 검증 포함
     - 중복 URL 체크 포함
  
  2. list_bookmarks
     - 입력: tag(선택, 필터링), limit(선택, 기본값 20)
     - tag 지정 시 해당 태그만, 미지정 시 전체
  
  3. delete_bookmark
     - 입력: id(필수)
     - 존재하지 않는 ID면 에러
  
  server.registerTool() 사용.
  각 도구의 description은 Claude가 자동 선택할 수 있도록 적극적으로 작성."
```

### 도구별 핸들러 분리

index.ts가 너무 길어지면 tools/ 폴더로 분리한다:

```bash
> "index.ts가 길어졌으니 도구 핸들러를 분리해줘:
  - src/tools/addBookmark.ts
  - src/tools/listBookmarks.ts
  - src/tools/deleteBookmark.ts
  
  index.ts에서는 import해서 등록만 하도록."
```

---

## ③ Inspector로 도구 검증

### 빌드 & 테스트

```bash
# 빌드
npm run build

# Inspector 실행
npx @modelcontextprotocol/inspector node build/index.js
```

### 테스트 시나리오

Inspector에서 다음 순서로 테스트한다:

```
1. add_bookmark 테스트:
   - url: "https://modelcontextprotocol.io"
   - title: "MCP 공식 사이트"
   - tags: ["mcp", "protocol"]
   - memo: "MCP 학습 참고"
   → 생성된 북마크 확인 (id, createdAt 자동 생성)

2. add_bookmark 중복 테스트:
   - 같은 url로 다시 추가
   → DuplicateUrlError 확인

3. list_bookmarks 테스트:
   - 파라미터 없이 호출 → 전체 목록
   - tag: "mcp" → 필터링 결과

4. delete_bookmark 테스트:
   - id: 위에서 생성된 ID
   → 삭제 확인

5. delete_bookmark 에러 테스트:
   - id: "bm_없는아이디"
   → BookmarkNotFoundError 확인
```

---

## ④ Codex CLI로 테스트 코드 작성

### 왜 Codex CLI인가

Day 7에서 배운 Codex CLI의 강점을 활용한다:
- **OS 레벨 샌드박스**: 테스트 코드가 실수로 파일을 망가뜨리지 않음
- **/review 기능**: 작성된 테스트의 품질을 바로 리뷰
- **read-only 프로필**: BookmarkStore 코드를 분석만 하고 테스트 생성

### Codex CLI로 테스트 작성

```bash
# bookmark-mcp-server 폴더에서 Codex 실행
codex

> "src/store/bookmarkStore.ts의 BookmarkStore 클래스에 대한
  테스트 코드를 작성해줘.
  
  테스트 프레임워크: vitest 또는 jest (프로젝트에 맞는 걸로)
  
  테스트 시나리오:
  1. add: 정상 추가, 중복 URL 에러, URL 형식 에러
  2. getAll: 빈 목록, 여러 개 조회
  3. getById: 존재하는 ID, 존재하지 않는 ID
  4. delete: 정상 삭제, 존재하지 않는 ID 에러
  5. findByTag: 태그 필터링 정상 동작
  6. search: 키워드 검색 (title, url, memo, tags)
  
  테스트용 임시 JSON 파일을 사용해서 실제 데이터에 영향 없게."
```

### 테스트 실행

```bash
# 테스트 의존성 설치 (아직 없다면)
npm install -D vitest

# package.json에 테스트 스크립트 추가
# "scripts": { "test": "vitest run" }

# 테스트 실행
npm test
```

### Codex /review로 테스트 품질 확인

```bash
codex

> /review
# "Review uncommitted changes" 선택
# Codex가 테스트 코드의 커버리지, edge case 누락 등을 리뷰
```

---

## 실습 과제

### 과제 1: BookmarkStore + 에러 클래스 구현 (15분)
1. Claude Code로 bookmarkStore.ts 구현
2. errors.ts, logger.ts 구현
3. 빌드 확인 (npm run build)

### 과제 2: MCP 도구 3개 등록 & 테스트 (20분)
1. add_bookmark, list_bookmarks, delete_bookmark 등록
2. Inspector에서 5가지 시나리오 테스트
3. 에러 케이스 정상 동작 확인

### 과제 3: Codex CLI로 테스트 작성 (15분)
1. Codex CLI에서 테스트 코드 생성
2. npm test로 실행 & 통과 확인
3. /review로 테스트 품질 리뷰

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| 구현 순서 | Store → 도구 등록 → Inspector 테스트 → 단위 테스트 |
| Claude Code | CLAUDE.md + PRD 참조하며 구현 (체이닝) |
| 도구 분리 | tools/ 폴더에 핸들러 파일별 분리 |
| Inspector | 도구별 정상/에러 케이스 수동 테스트 |
| Codex CLI | 테스트 코드 작성 + /review로 품질 확인 |
| 오늘 범위 | CRUD 중 C(add), R(list), D(delete) 3개 |
