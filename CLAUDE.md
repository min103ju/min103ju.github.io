# CLAUDE.md - Quartz Blog (min103ju.github.io)

---

## WHAT - 프로젝트 개요

Quartz 4.5.2 기반 정적 블로그(Digital Garden). AI 도구 마스터리 커리큘럼(Day1~19) 콘텐츠를 Obsidian으로 작성하고 Quartz로 빌드하여 GitHub Pages에 배포한다.

### 기술 스택
- **SSG**: Quartz 4.5.2 (TypeScript, Preact, SCSS)
- **런타임**: Node.js 22+, npm 10.9.2+
- **콘텐츠**: Markdown (Obsidian Flavored) + YAML frontmatter
- **배포**: GitHub Actions → GitHub Pages (브랜치: `v4`)
- **주요 라이브러리**: Remark/Rehype, Shiki, KaTeX, Flexsearch, D3.js

### 핵심 디렉토리
```
content/           # 블로그 콘텐츠 (Markdown 파일)
quartz/
  components/      # Preact UI 컴포넌트 (34개)
  plugins/         # Transformer(14) / Filter(1) / Emitter(14)
  styles/          # SCSS 스타일시트
  i18n/            # 다국어 로케일 (33개)
quartz.config.ts   # 사이트 설정 (제목, 테마, 플러그인)
quartz.layout.ts   # 페이지 레이아웃 구성
```

---

## WHY - 아키텍처 결정 배경

### Obsidian + Quartz 조합을 선택한 이유
- Obsidian의 위키링크(`[[]]`), 콜아웃, 그래프뷰를 그대로 웹에서 지원
- 로컬 Obsidian vault → Git push → 자동 빌드/배포의 심리스 워크플로우
- `ObsidianFlavoredMarkdown` 플러그인이 Obsidian 문법을 HTML로 변환

### 브랜치 전략
- `v4` 브랜치가 메인이자 배포 브랜치 (Quartz 4.x 컨벤션)
- 커밋 메시지: `Quartz sync: {날짜}` 형식으로 Obsidian 동기화 기록

### 콘텐츠 구조 설계
- `Day{N}_{주제}.md` 네이밍으로 학습 커리큘럼 순서 표현
- frontmatter에 title, date, tags 포함
- `private/`, `templates/`, `.obsidian/` 폴더는 빌드에서 제외

---

## HOW - 작업 가이드

### 빌드 & 개발 명령어
```bash
npx quartz build              # 정적 사이트 빌드 → public/
npx quartz build --serve      # 로컬 개발 서버 (핫 리로드)
npm run check                 # TypeScript 타입 체크 + Prettier 포맷 검증
npm run format                # Prettier 자동 포맷팅
npm test                      # 테스트 실행
```

### 콘텐츠 작성 규칙
- 파일 위치: `content/` 디렉토리
- frontmatter 필수 포함 (title, date)
- draft 상태 글은 `RemoveDrafts` 필터로 빌드에서 제외
- Obsidian 문법 사용 가능: 위키링크, 콜아웃, 체크박스, Mermaid

### 코드 스타일
- Prettier: 100자 줄폭, 2칸 들여쓰기, 세미콜론 없음, trailing comma
- TypeScript strict 모드
- JSX: Preact (`react-jsx` with `preact` import source)

### 배포 프로세스
1. `v4` 브랜치에 push
2. `.github/workflows/deploy.yml` 자동 실행
3. Node 22 환경에서 `npx quartz build` 실행
4. `public/` 디렉토리를 GitHub Pages에 업로드

### 커스터마이징 시 주의사항
- 컴포넌트 수정: `quartz/components/` (Preact + inline scripts)
- 플러그인 수정: `quartz/plugins/` (transformers, filters, emitters)
- 스타일 수정: `quartz/styles/custom.scss`
- 레이아웃 변경: `quartz.layout.ts` (사이드바, 헤더, 푸터 구성)
- 사이트 설정: `quartz.config.ts` (제목, 테마 색상, 폰트, 분석도구)

### 콘텐츠 네이밍 & 구조 컨벤션
- 폴더: `content/{카테고리}/{서브카테고리}/` (dev/ai/, life/parenting/ 등)
- 파일명: 한글 허용, 공백 금지, 언더스코어 구분
- 태그: 소문자 영문 + 하이픈 (예: claude-code, api-integration)
- frontmatter 필수: title, date, tags
- frontmatter 권장: description (SEO 메타 디스크립션용)

### 자주 하는 실수 방지
- `quartz.config.ts` 수정 후 빌드 테스트 없이 push하지 말 것
- content 파일명에 공백 포함 시 URL 인코딩 문제 주의
- Obsidian 플러그인 설정(`.obsidian/`)은 Git에 포함되지만 빌드와 무관
- `CustomOgImages` 플러그인은 빌드 속도를 위해 현재 비활성 상태
