# quartz-customizer: Quartz 커스터마이징 에이전트

Quartz 블로그의 컴포넌트, 플러그인, 테마, 레이아웃을 수정합니다.

## 입력
- 수정 요청 사항: $ARGUMENTS

## 작업 전 필수 확인

### 1. 현재 설정 파악
- `quartz.config.ts` 읽기 (사이트 설정, 플러그인 목록)
- `quartz.layout.ts` 읽기 (레이아웃 구성)
- `quartz/styles/custom.scss` 읽기 (커스텀 스타일)

### 2. 수정 가능 영역
| 영역 | 파일 위치 | 설명 |
|------|----------|------|
| 사이트 설정 | `quartz.config.ts` | 제목, 테마, 폰트, 색상, 플러그인 |
| 레이아웃 | `quartz.layout.ts` | 사이드바, 헤더, 푸터 구성 |
| 커스텀 스타일 | `quartz/styles/custom.scss` | CSS 오버라이드 |
| 컴포넌트 | `quartz/components/*.tsx` | Preact 컴포넌트 |
| 인라인 스크립트 | `quartz/components/scripts/*.inline.ts` | 클라이언트 JS |
| 플러그인 | `quartz/plugins/transformers/` | 마크다운 변환 로직 |

### 3. 수정 절차
1. 변경 전 관련 파일 전체 읽기
2. 변경 사항 명확히 설명 후 수정
3. TypeScript 타입 호환성 확인
4. `npm run check` 실행하여 타입/포맷 검증
5. `npx quartz build` 실행하여 빌드 성공 확인

### 4. 주의사항
- Preact 컴포넌트는 React가 아님 (`preact/hooks` import)
- SCSS 변수는 `quartz/styles/_variables.scss` 참조
- 인라인 스크립트는 빌드 시 번들링됨 (DOM API 직접 사용)
- `quartz.config.ts` 수정 시 반드시 빌드 테스트
- upstream Quartz 업데이트와 충돌 가능성 고려
