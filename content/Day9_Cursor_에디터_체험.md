# Day 9: Cursor / Windsurf 에디터 체험

> AI Tools Mastery Curriculum — Week 2, Day 9
> 소요 시간: 50분 | 탐색 중심

---

## ① Cursor 설치 & 실무 프로젝트 열기

### Cursor란

Cursor는 VS Code를 포크해서 만든 **AI 네이티브 IDE**다. 플러그인으로 AI를 붙인 게 아니라, 에디터 자체가 AI를 중심으로 설계되었다. 코드베이스 전체를 인덱싱해서 AI가 프로젝트를 "이해"한 상태에서 동작한다.

**Claude Code, Gemini CLI, Codex CLI와의 핵심 차이:**

```
터미널 에이전트 (Claude Code 등):  터미널에서 텍스트 기반 대화
AI IDE (Cursor):                   에디터 안에서 코드를 보면서 AI 활용
```

터미널 에이전트는 코드를 "텍스트로 읽고 텍스트로 수정"한다.
Cursor는 에디터에서 **diff를 시각적으로 보여주고, 파일 트리를 탐색하면서** AI를 활용한다.

### 가격 정책

| 플랜 | 월 비용 | 핵심 기능 |
|---|---|---|
| Hobby (무료) | $0 | 제한된 Agent 요청 + 제한된 Tab 완성 |
| Pro | $20/월 | 무제한 Tab + $20 크레딧 풀 + Cloud Agent |
| Pro+ | $60/월 | Pro의 3배 사용량 |
| Ultra | $200/월 | Pro의 20배 사용량 |

무료 Hobby 플랜으로도 체험은 가능하지만, 일상 개발용으로는 부족하다.
7일 무료 Pro 트라이얼이 있으니 이번 체험에 활용하자.

### 설치

```bash
# macOS
brew install --cask cursor

# 또는 cursor.com에서 직접 다운로드
# Windows, macOS, Linux 모두 지원
```

### VS Code 설정 마이그레이션

Cursor는 VS Code 포크이므로, 기존 VS Code 설정을 그대로 가져올 수 있다:

```
1. Cursor 첫 실행 시 "Import VS Code Settings" 선택
2. 확장, 테마, 키바인딩 자동 마이그레이션
3. 기존 VS Code 확장 대부분 호환
```

### 프로젝트 열기 & 인덱싱

```
1. File → Open Folder → 프로젝트 폴더 선택
2. Cursor가 자동으로 코드베이스 인덱싱 시작
3. 인덱싱 완료까지 수 분 소요 (프로젝트 크기에 따라)
4. 완료 후 AI가 프로젝트 전체를 "이해"한 상태
```

인덱싱이 완료되면 "이 프로젝트의 인증 로직이 어디에 있어?"같은 질문에 정확하게 파일을 찾아준다.

### .cursorrules 설정 (CLAUDE.md에 해당)

Cursor에도 프로젝트별 AI 지시 파일이 있다:

```markdown
<!-- .cursor/rules/project.md -->
# Project Rules

## 코딩 컨벤션
- DateTime: LocalDateTime 대신 OffsetDateTime 사용
- @Transactional 안에서 외부 API 호출 금지
- WebClient maxConnections=10

## 테스트 규칙
- @WebMvcTest 사용 (@SpringBootTest 금지)
- 정상/유효성실패/인증실패 3가지 시나리오 필수

## 빌드
- ./gradlew build
- ./gradlew test
```

CLAUDE.md와 내용은 동일하되, 위치와 형식이 다르다.

---

## ② Composer 모드 & Agent 모드 활용

### Cursor의 3가지 AI 모드

| 모드 | 단축키 | 용도 | 자율성 |
|---|---|---|---|
| **Tab** | 자동 | 코드 자동완성 (한 줄~수 줄) | 최소 |
| **Inline Edit** | `Cmd+K` | 선택 영역 수정 | 중간 |
| **Composer/Agent** | `Cmd+I` | 멀티 파일 편집, 자율 실행 | 최대 |

### Tab — AI 자동완성

코딩 중 자동으로 다음 코드를 예측한다. GitHub Copilot과 유사하지만, Cursor는 **프로젝트 전체를 인덱싱**해서 예측하므로 정확도가 높다.

```java
// Tab이 다음 줄을 예측
public CourseResponse getCourse(Long courseId) {
    // Tab: return courseRepository.findById(courseId)
    //          .map(CourseResponse::from)
    //          .orElseThrow(() -> new CourseNotFoundException(courseId));
}
```

Tab은 코딩 흐름을 끊지 않고 속도를 높이는 데 최적화되어 있다.

### Inline Edit (Cmd+K) — 선택 영역 수정

코드를 선택하고 `Cmd+K`를 누르면 해당 부분만 AI로 수정한다:

```
1. 수정할 코드 블록 선택
2. Cmd+K
3. "이 메서드를 비동기로 전환해줘" 입력
4. 변경 사항 diff로 확인 후 적용/취소
```

작은 범위의 정밀한 수정에 적합하다. 한 메서드, 한 클래스 수준.

### Composer (Cmd+I) — 멀티 파일 편집

Cursor의 핵심 기능. 자연어로 지시하면 **여러 파일을 동시에 편집**한다:

```
Cmd+I 누르고:

> "수강신청 API에 정원 체크 로직을 추가해줘.
  CourseService에 정원 확인 메서드를 추가하고,
  EnrollmentService에서 수강신청 전에 호출하고,
  정원 초과 시 CourseFullException을 던져줘."
```

Composer가 하는 일:
1. CourseService.java에 `checkCapacity()` 메서드 추가
2. EnrollmentService.java에 호출 코드 추가
3. CourseFullException.java 새 파일 생성
4. 각 파일의 변경사항을 **diff로 보여줌**
5. 파일별로 적용/취소 선택 가능

**Composer의 핵심 장점: 변경사항을 시각적 diff로 확인할 수 있다.**
터미널 에이전트에서는 코드가 텍스트로 출력되지만, Cursor에서는 에디터의 diff 뷰로 확인한다.

### Agent 모드 — 자율 실행

Composer의 확장판. AI가 **자율적으로 계획→실행→확인→수정**을 반복한다:

```
Cmd+I → Agent 모드 선택:

> "프로젝트에 Spring Security 기본 설정을 추가해줘.
  JWT 인증, SecurityConfig, 필터 체인까지."
```

Agent가 하는 일:
1. 필요한 의존성 확인 (build.gradle)
2. SecurityConfig.java 생성
3. JwtFilter.java 생성
4. 터미널에서 빌드 실행
5. 에러 발생 시 자동 수정
6. 테스트 실행 후 통과 확인

**Composer vs Agent 차이:**

```
Composer: 사용자 → Cursor → 사용자 → Cursor → 완료
          (매 파일 변경마다 승인)

Agent:    사용자 → Cursor [계획→실행→관찰→수정] → 완료
          (자율적으로 루프)
```

간단한 작업(파일 3개 이하)은 Composer, 복잡한 작업(빌드/테스트 포함)은 Agent가 적합하다.

### @컨텍스트 참조

Composer/Agent에서 특정 파일이나 폴더를 참조할 수 있다:

```
> @src/main/java/com/example/service/ 이 폴더의 서비스들을 
  분석하고 공통 패턴을 정리해줘

> @CourseService.java 이 파일을 리팩토링해줘

> @https://docs.spring.io/spring-boot/reference/web.html 
  이 문서를 참고해서 구현해줘
```

### .cursor/rules/ 로 AI 행동 제어

프로젝트별 AI 규칙을 설정할 수 있다:

```markdown
<!-- .cursor/rules/java-conventions.md -->
# Java Conventions

## 필수 규칙
- OffsetDateTime 사용 (LocalDateTime 금지)
- @Transactional 내 외부 API 호출 금지
- 테스트에 @WebMvcTest 사용

## 응답 형식
- 한국어로 설명
- 변경 이유를 항상 주석으로 포함
```

---

## ③ IntelliJ 워크플로우와 비교 분석

### 백엔드 개발자가 비교해야 할 포인트

IntelliJ는 백엔드 개발자의 메인 IDE다. Cursor를 대체할 것인지, 병행할 것인지 판단하려면 실무 워크플로우 기준으로 비교해야 한다.

### 기능별 비교

| 기능 | IntelliJ IDEA | Cursor |
|---|---|---|
| **Java/Spring 지원** | 최고 수준 (네이티브) | VS Code 수준 (확장 필요) |
| **리팩토링** | 정적 분석 기반 (안전) | AI 기반 (유연하지만 검증 필요) |
| **디버깅** | 완전한 디버거 | VS Code 수준 디버거 |
| **AI 자동완성** | AI Assistant 플러그인 | 네이티브 (Tab, 프로젝트 인덱싱) |
| **멀티 파일 편집** | 수동 | Composer로 자동 |
| **AI Agent** | 없음 (Codex ACP로 일부 가능) | 네이티브 Agent 모드 |
| **Gradle 통합** | 네이티브 | 터미널 기반 |
| **DB 도구** | Database 탭 내장 | 없음 (외부 도구 필요) |
| **Git 통합** | 강력한 내장 도구 | VS Code 수준 |
| **가격** | Ultimate $249/년 | Pro $20/월 ($240/년) |

### 실무 시나리오별 비교

#### 시나리오 1: 새 API 엔드포인트 추가

```
IntelliJ:
  1. Controller 클래스에서 메서드 생성 (타이핑)
  2. Service 인터페이스 작성 → Alt+Enter로 구현체 생성
  3. Repository 메서드 추가
  4. DTO 작성
  → 정적 타입 검사가 실시간으로 동작, 안전

Cursor:
  1. Cmd+I → "수강 검색 API를 추가해줘"
  2. Composer가 Controller + Service + Repository + DTO 동시 생성
  3. diff로 확인 후 적용
  → 빠르지만 생성된 코드 검토 필요
```

#### 시나리오 2: 복잡한 리팩토링

```
IntelliJ:
  1. 클래스 선택 → Refactor → Extract Interface
  2. 정적 분석으로 모든 참조 자동 업데이트
  3. 컴파일 에러 0개 보장
  → 안전하지만 복잡한 구조 변경은 수동

Cursor:
  1. Cmd+I → "이 서비스에서 외부 API 호출을 분리해줘"
  2. Agent가 새 클래스 생성, 기존 코드 수정, 테스트 실행
  3. 빌드 실패 시 자동 수정
  → 유연하지만 AI가 놓치는 참조가 있을 수 있음
```

#### 시나리오 3: 디버깅

```
IntelliJ:
  1. 브레이크포인트 설정
  2. Debug 모드로 실행
  3. 변수 값 실시간 확인, 스텝 오버/인투
  → 압도적 우위. Cursor로는 대체 불가.

Cursor:
  1. Debug Mode에서 AI가 에러 원인 추적
  2. 가설 기반 디버깅 제안
  → 보조 도구로는 유용하지만, 실제 디버거 대체는 불가
```

### 실무적 결론: 대체가 아니라 병행

대부분의 백엔드 개발자에게 현실적인 조합은:

```
메인 IDE:    IntelliJ IDEA (디버깅, 리팩토링, Spring 지원)
보조 도구 1: Claude Code (터미널 기반 AI, 분석/생성)
보조 도구 2: Cursor (멀티 파일 편집, 시각적 diff 확인)
```

Cursor가 IntelliJ를 완전히 대체하기 어려운 이유:
- Java/Spring 생태계 지원이 IntelliJ에 비해 약함
- 정적 분석 기반 리팩토링의 안전성을 AI가 보장할 수 없음
- 디버거 품질 차이가 큼

Cursor가 IntelliJ보다 나은 영역:
- 멀티 파일 AI 편집 (Composer)
- 코드베이스 전체를 대상으로 한 자연어 질문
- 빠른 프로토타이핑 (Agent 모드)

### 최신 소식: Cursor in JetBrains (ACP)

2026년 3월부터 **Cursor가 JetBrains IDE 안에서 사용 가능**해졌다. Agent Client Protocol(ACP)을 통해 IntelliJ 안에서 Cursor의 AI Agent를 사용할 수 있다.

```
IntelliJ → Cursor ACP 플러그인 설치 → Cursor 계정 로그인
→ IntelliJ의 강력한 Java 지원 + Cursor의 AI Agent를 동시에 사용
```

이 조합이 성숙하면 "IntelliJ vs Cursor"가 아니라 "IntelliJ + Cursor"가 될 수 있다.

---

## 실습 과제

### 과제 1: Cursor 설치 & 설정 (10분)
1. cursor.com에서 다운로드 → 설치
2. VS Code 설정 마이그레이션
3. 프로젝트 열기 → 인덱싱 완료 대기
4. .cursor/rules/ 에 프로젝트 규칙 작성

### 과제 2: Composer & Agent 체험 (25분)
1. Tab 자동완성으로 코드 작성 체험
2. Cmd+K로 Inline Edit 체험 (메서드 하나 수정)
3. Cmd+I Composer로 멀티 파일 편집 체험
4. Agent 모드로 기능 하나 자율 구현 체험
5. 각 모드의 적합한 상황 메모

### 과제 3: IntelliJ 비교 분석 (15분)
1. 동일 태스크를 IntelliJ(수동)과 Cursor(AI)로 수행
2. 속도, 정확도, 안전성 비교 기록
3. "어떤 상황에서 어떤 도구가 더 나은가" 결론 정리
4. Cursor in JetBrains (ACP) 설치 체험 (가능하면)

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| Cursor 본질 | VS Code 포크 기반 AI 네이티브 IDE |
| Tab | 프로젝트 인덱싱 기반 자동완성, 코딩 흐름 유지 |
| Inline Edit (Cmd+K) | 선택 영역 정밀 수정 |
| Composer (Cmd+I) | 멀티 파일 동시 편집, diff로 확인 |
| Agent | Composer + 자율 실행 루프 (계획→실행→관찰→수정) |
| vs IntelliJ | Java/Spring/디버깅은 IntelliJ, AI 편집은 Cursor |
| 실무 결론 | 대체가 아닌 병행. IntelliJ(메인) + Claude Code/Cursor(보조) |
| ACP | Cursor가 JetBrains 안에서 동작 가능 (2026.3~) |
| 가격 | 무료(제한적), Pro $20/월, 7일 무료 트라이얼 |

---

## 참고 리소스

- Cursor 공식: [cursor.com](https://cursor.com)
- 가격: [cursor.com/pricing](https://cursor.com/pricing)
- Changelog: [cursor.com/changelog](https://cursor.com/changelog)
- Agent 가이드: [cursor.com/product](https://cursor.com/product)
- Cursor in JetBrains: [ACP 공지](https://cursor.com/changelog) (2026.3.4)
- 커뮤니티 가이드: [cursor-guide (GitHub)](https://github.com/slava-kudzinau/cursor-guide)
