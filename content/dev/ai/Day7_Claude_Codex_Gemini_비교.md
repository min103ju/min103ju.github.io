---
title: "Day 7: OpenAI Codex CLI 체험 & AI 코드 생성 비교"
date: 2026-03-11
description: "OpenAI Codex CLI를 체험하고, Claude Code, Gemini CLI와 함께 Spring Boot TODO API 코드 생성을 비교 분석한다."
tags:
  - codex
  - claude
  - gemini
  - cli
  - comparison
  - week2
---

# Day 7: OpenAI Codex CLI 체험 & AI 코드 생성 비교

> AI Tools Mastery Curriculum — Week 2, Day 7
> 소요 시간: 45분 | 탐색 중심

---

## ① Codex CLI 설치 & 인증

### Codex CLI란

Codex CLI는 OpenAI의 터미널 코딩 에이전트다. Rust로 작성되어 네이티브 바이너리로 동작하며, GPT-5 계열의 Codex 전용 모델을 사용한다.

**핵심 스펙:**
- 모델: GPT-5.2-Codex (기본), GPT-5.3-Codex (최신)
- 인증: ChatGPT 계정 (Plus/Pro/Team) 또는 API 키
- 샌드박스: OS 레벨 격리 (macOS Seatbelt, Linux Landlock)
- MCP 지원: Claude Code, Gemini CLI와 동일한 MCP 서버 연결 가능
- Skills: 재사용 가능한 스킬 시스템 지원
- 오픈소스: MIT 라이선스

**비용:** ChatGPT Plus($20/월) 구독에 포함. 별도 API 키 사용 시 토큰 과금.

### 설치

```bash
# npm으로 설치 (추천)
npm install -g @openai/codex

# 또는 Homebrew (macOS)
brew install --cask codex

# 설치 확인
codex --version
```

### 인증 설정

#### 방법 1: ChatGPT 계정 로그인 (추천)

```bash
codex
# 첫 실행 시 "Sign in with ChatGPT" 선택
# 브라우저가 열리고 OAuth 인증 진행
# 인증 완료 후 자동으로 CLI로 복귀
```

인증 정보는 `~/.codex/auth.json`에 캐시되어 이후 자동 로그인된다.

#### 방법 2: API 키 사용

```bash
# OpenAI 대시보드에서 API 키 발급
export OPENAI_API_KEY="sk-..."

# .zshrc에 추가해두면 영구 적용
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.zshrc

codex
```

API 키 방식은 CI/CD나 headless 환경에서 유용하다.

#### 인증 트러블슈팅

```bash
# 인증 캐시 초기화
codex logout

# 또는 수동으로 삭제
rm -f ~/.codex/auth.json

# 다시 로그인
codex login

# 원격/headless 환경에서 device code 방식 사용
codex login --device-auth
```

### 첫 실행 확인

```bash
cd ~/workspace/your-project
codex
# 전체 화면 TUI(Terminal UI)가 열림
> 이 프로젝트의 구조를 간단히 설명해줘
```

**Git 레포지토리 안에서 실행하는 것을 강력 추천.** Git이 있으면 변경사항을 쉽게 되돌릴 수 있다.

---

## ② Codex CLI 샌드박스 모드 체험

### 샌드박스란

Codex CLI의 가장 큰 차별점은 **OS 레벨 샌드박스**다.
Claude Code와 Gemini CLI는 프롬프트/승인 기반으로 안전성을 확보하지만, Codex는 **운영체제 수준에서 파일/네트워크 접근을 차단**한다.

```
Claude Code:  "이 명령 실행해도 될까요?" → 사용자가 Y/N
Codex CLI:    OS가 물리적으로 쓰기 접근을 차단 (Seatbelt/Landlock)
```

### 세 가지 샌드박스 모드

| 모드 | 파일 접근 | 네트워크 | 용도 |
|---|---|---|---|
| `read-only` | 읽기만 가능 | 차단 | 코드 분석, 리뷰 |
| `workspace-write` (기본) | 프로젝트 폴더 내 쓰기 | 차단 | 일상 개발 |
| `danger-full-access` | 전체 파일시스템 | 허용 | 격리된 환경에서만 |

### 샌드박스 모드 실습

#### 실습 1: read-only 모드 (분석 전용)

```bash
codex -s read-only
> EnrollmentService의 구조를 분석하고 개선점을 알려줘
```

이 모드에서는 Codex가 파일을 읽을 수는 있지만 **어떤 파일도 수정할 수 없다.**
Claude Code의 Plan Mode와 비슷한 용도지만, 시스템 레벨에서 강제된다는 차이가 있다.

#### 실습 2: workspace-write 모드 (기본)

```bash
codex  # 기본값이 workspace-write
> TodoController에 할일 완료 처리 API를 추가해줘
```

프로젝트 폴더 안의 파일만 수정할 수 있고, 바깥 파일시스템과 네트워크는 차단된다.
`npm install` 같은 네트워크가 필요한 명령은 실패한다.

#### 실습 3: 네트워크 허용 (필요 시)

```bash
# workspace-write + 네트워크 허용
codex -c 'sandbox_workspace_write.network_access=true' "의존성을 업데이트해줘"
```

### 승인 정책 (Approval Policy)

샌드박스와 별개로, Codex의 행동에 대한 **승인 정책**도 설정할 수 있다:

| 정책                | 동작            | 용도      |
| ----------------- | ------------- | ------- |
| `on-request` (기본) | 중요 작업 전 승인 요청 | 일상 개발   |
| `never`           | 승인 없이 자동 실행   | 자동화, CI |
| `untrusted`       | 모든 작업에 승인 요청  | 민감한 코드  |

```bash
# 기본: 중요 작업만 승인
codex

# 완전 자동화 (승인 스킵 + workspace-write 샌드박스)
codex --full-auto "테스트를 실행하고 실패하는 것을 수정해줘"

# 세션 중에 모드 변경
> /approvals auto
> /approvals readonly
```

### config.toml로 기본값 설정

```toml
# ~/.codex/config.toml

model = "gpt-5.2-codex"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

# 네트워크가 필요한 프로젝트용 프로필
[profiles.with-network]
sandbox_mode = "workspace-write"
approval_policy = "on-request"
[profiles.with-network.sandbox_workspace_write]
network_access = true

# 분석 전용 프로필
[profiles.review]
sandbox_mode = "read-only"
approval_policy = "on-request"
```

프로필 사용:
```bash
codex -p review "이 PR의 변경사항을 리뷰해줘"
codex -p with-network "npm install 후 테스트 실행해줘"
```

---

## ③ 코드 생성/리팩토링 결과 비교 분석

### 3도구 비교 실험

이제 Claude Code, Gemini CLI, Codex CLI 세 도구를 동일 태스크로 비교한다.

#### 태스크 1: 코드 분석

```
# 세 도구 모두 동일 프롬프트
> EnrollmentService 클래스를 분석해줘.
  의존 관계, 데이터 흐름, 잠재적 문제점을 정리해줘.
```

#### 태스크 2: 리팩토링 제안

```
> EnrollmentService에서 @Transactional 안에 외부 API 호출이 있는 부분을
  찾아서 리팩토링 계획을 세워줘. 코드는 작성하지 마.
```

#### 태스크 3: 코드 생성

```
> CourseService에 강좌 검색 API를 추가해줘.
  키워드, 카테고리, 가격대로 필터링할 수 있어야 해.
  Specification 패턴을 사용해줘.
```

### 비교 기록 매트릭스

```markdown
## 3도구 비교 결과

### 태스크 1: 코드 분석
| 항목 | Claude Code | Gemini CLI | Codex CLI |
|---|---|---|---|
| 탐색 파일 수 | __ 개 | __ 개 | __ 개 |
| 분석 깊이 (1-5) | __ | __ | __ |
| 문제점 발견 수 | __ 개 | __ 개 | __ 개 |
| 응답 속도 | __ 초 | __ 초 | __ 초 |
| 컨벤션 반영 | Yes/No | Yes/No | Yes/No |

### 태스크 2: 리팩토링 제안
| 항목 | Claude Code | Gemini CLI | Codex CLI |
|---|---|---|---|
| 문제 식별 정확도 (1-5) | __ | __ | __ |
| 계획 합리성 (1-5) | __ | __ | __ |
| 리스크 분석 포함 | Yes/No | Yes/No | Yes/No |

### 태스크 3: 코드 생성
| 항목 | Claude Code | Gemini CLI | Codex CLI |
|---|---|---|---|
| 코드 품질 (1-5) | __ | __ | __ |
| 컨벤션 준수 (1-5) | __ | __ | __ |
| 바로 실행 가능 | Yes/No | Yes/No | Yes/No |
| 수동 수정 필요 수 | __ 개 | __ 개 | __ 개 |
```

### 도구별 특성 비교 정리

비교 실험이 끝나면, 각 도구의 **고유한 강점**을 정리한다:

| 특성 | Claude Code | Gemini CLI | Codex CLI |
|---|---|---|---|
| **컨텍스트 크기** | 200K 토큰 | 1M 토큰 | 200K 토큰 |
| **샌드박스** | 승인 기반 | 없음 (별도 설정) | OS 레벨 (기본 내장) |
| **서브 에이전트** | Task() 내장 | 없음 | Multi-agent 지원 |
| **컨텍스트 파일** | CLAUDE.md | GEMINI.md (@import) | AGENTS.md |
| **Skills** | .claude/skills/ | 없음 | $skill-name |
| **Hooks** | settings.json | 없음 | 없음 (rules로 대체) |
| **코드 리뷰** | 수동 요청 | 수동 요청 | /review 내장 |
| **웹 검색** | 서브 에이전트 위임 | Google 검색 내장 | 캐시 기반 (기본) |
| **비대화형** | claude -p | gemini -p | codex exec |
| **무료 사용** | API 키 필요 (유료) | 60회/분 무료 | ChatGPT Plus 필요 |
| **클라우드 연동** | 없음 | 없음 | Codex Web (비동기) |

### 상황별 도구 추천 (초안)

비교 실험 후 자신만의 추천을 정리한다:

```markdown
## 내 상황별 추천 도구

### 코드 분석 & 리뷰
추천: ________
이유:

### 리팩토링
추천: ________
이유:

### 새 코드 생성
추천: ________
이유:

### 테스트 작성
추천: ________
이유:

### 대규모 코드베이스 탐색
추천: ________
이유:

### CI/CD 자동화
추천: ________
이유:
```

---

## ④ Spring Boot TODO API - AI 코드 생성 비교 분석

> 동일한 프롬프트(CLAUDE.md)로 Claude, Gemini, Codex 세 AI가 각각 생성한 Spring Boot TODO API 프로젝트를 비교 분석한 문서입니다.

### 프로젝트 개요 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **Spring Boot** | 4.0.0 | 4.0.0 | 3.5.0 |
| **Java** | 25 | 25 | 25 |
| **빌드 스크립트** | build.gradle (Groovy) | build.gradle (Groovy) | build.gradle.kts (Kotlin DSL) |
| **서버 포트** | 8080 | 8082 | 8081 |
| **데이터 저장소** | H2 파일 DB | H2 인메모리 DB | ConcurrentHashMap (인메모리) |
| **ORM** | Spring Data JPA | Spring Data JPA | 없음 (직접 구현) |
| **소스 파일 수** | 8개 | 9개 | 11개 |
| **테스트 수** | 7개 | 2개 | 5개 |

### 아키텍처 비교

#### 공통점
- Controller → Service → Repository 3계층 구조
- 생성자 주입(Constructor Injection) 사용
- DTO 패턴으로 요청/응답 분리
- Service 계층에 `@Transactional` 적용

#### 패키지 구조 비교

```
Claude                          Gemini                          Codex
─────                          ──────                          ─────
com.example.todo               com.example.todo                com.example.todo
├── controller/                ├── controller/                 ├── web/
│   └── TodoController         │   └── TodoController          │   ├── TodoController
├── service/                   ├── service/                    │   └── dto/
│   └── TodoService            │   └── TodoService             │       ├── CreateTodoRequest
├── repository/                ├── repository/                 │       ├── UpdateTodoRequest
│   └── TodoRepository         │   └── TodoRepository          │       └── TodoResponse
├── entity/                    ├── entity/                     ├── service/
│   └── Todo                   │   └── Todo                    │   ├── TodoService
├── dto/                       ├── dto/                        │   └── TodoNotFoundException
│   ├── TodoRequest            │   ├── TodoRequest             ├── repository/
│   └── TodoResponse           │   └── TodoResponse            │   ├── TodoRepository (인터페이스)
└── TodoApplication            ├── exception/                  │   └── InMemoryTodoRepository
                               │   └── GlobalExceptionHandler  ├── domain/
                               └── TodoApplication             │   └── Todo
                                                               └── TodoApplication
```

#### 차이점 분석

| 설계 관점 | Claude | Gemini | Codex |
|-----------|--------|--------|-------|
| **DTO 전략** | 단일 TodoRequest (생성/수정 공용) | 단일 TodoRequest (생성/수정 공용) | CreateTodoRequest / UpdateTodoRequest 분리 |
| **예외 처리** | Controller 내 `@ExceptionHandler` | `@RestControllerAdvice` 글로벌 핸들러 | `@ResponseStatus` 커스텀 예외 |
| **Repository** | JpaRepository 상속 | JpaRepository 상속 | 인터페이스 직접 정의 + 구현체 |
| **Entity 설계** | JPA Entity + `@PrePersist`/`@PreUpdate` | JPA Entity + 생성자 초기화 | POJO + `touch()` 메서드 |
| **DTO 타입** | Java Record | Java Record | 일반 Class (getter/setter) |

### API 엔드포인트 비교

| 기능 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| 전체 조회 | `GET /api/todos` | `GET /api/todos` | `GET /api/todos` |
| 단건 조회 | `GET /api/todos/{id}` | `GET /api/todos/{id}` | `GET /api/todos/{id}` |
| 생성 | `POST /api/todos` (201) | `POST /api/todos` (201) | `POST /api/todos` (201) |
| 수정 | `PUT /api/todos/{id}` | `PUT /api/todos/{id}` | `PUT /api/todos/{id}` |
| 삭제 | `DELETE /api/todos/{id}` (204) | `DELETE /api/todos/{id}` (204) | `DELETE /api/todos/{id}` (204) |
| 완료 토글 | `PATCH /api/todos/{id}/toggle` | - | - |

> Claude만 완료 토글 전용 엔드포인트를 별도 제공 (6개 엔드포인트 vs 5개)

### 데이터 모델 비교

| 필드 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| id | `Long` (IDENTITY) | `Long` (IDENTITY) | `long` (AtomicLong) |
| title | `String` (NOT NULL) | `String` (NOT NULL) | `String` |
| description | `String` (nullable) | - | - |
| completed | `boolean` | `boolean` | `boolean` |
| createdAt | `LocalDateTime` | `LocalDateTime` | `Instant` |
| updatedAt | `LocalDateTime` | - | `Instant` |

#### 주요 차이
- **Claude**: description 필드 포함, updatedAt 관리, `@PrePersist`/`@PreUpdate` JPA 콜백 활용
- **Gemini**: 최소한의 필드만 사용, updatedAt 없음
- **Codex**: `Instant`(UTC) 사용으로 타임존 안전, `touch()` 메서드로 수동 타임스탬프 관리, description 없음

### 입력 검증(Validation) 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **Validation 의존성** | 없음 | `spring-boot-starter-validation` | `spring-boot-starter-validation` |
| **@Valid 사용** | 미사용 | 사용 | 사용 |
| **@NotBlank** | 미사용 | title에 적용 | title에 적용 |
| **검증 에러 핸들링** | 없음 | GlobalExceptionHandler에서 처리 | Spring 기본 처리 |

> Claude는 validation 의존성 없이 서비스 레벨에서 null 체크만 수행. Gemini와 Codex는 Bean Validation 표준을 따름.

### 예외 처리 전략 비교

#### Claude
```java
// Controller 내부 @ExceptionHandler
@ExceptionHandler(NoSuchElementException.class)
public ResponseEntity<String> handleNotFound(NoSuchElementException e) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getMessage());
}
```
- NoSuchElementException을 Controller에서 직접 처리
- 단순 문자열 응답

#### Gemini
```java
// 별도 GlobalExceptionHandler 클래스
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(IllegalArgumentException.class)     // 404
    @ExceptionHandler(MethodArgumentNotValidException.class) // 400
    @ExceptionHandler(Exception.class)                     // 500
}
```
- 전역 예외 핸들러로 중앙화
- 400, 404, 500 모두 처리
- JSON 형태의 에러 응답

#### Codex
```java
// 커스텀 예외 + @ResponseStatus
@ResponseStatus(HttpStatus.NOT_FOUND)
public class TodoNotFoundException extends RuntimeException { }
```
- 가장 간결한 접근
- Spring의 `@ResponseStatus` 활용

### 트랜잭션 관리 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| 클래스 레벨 | `@Transactional(readOnly = true)` | `@Transactional(readOnly = true)` | `@Transactional` |
| 읽기 메서드 | 클래스 레벨 상속 | 클래스 레벨 상속 | `@Transactional(readOnly = true)` 개별 적용 |
| 쓰기 메서드 | `@Transactional` 개별 오버라이드 | `@Transactional` 개별 오버라이드 | 클래스 레벨 상속 |

> Claude와 Gemini는 "읽기 기본, 쓰기 오버라이드" 패턴. Codex는 "쓰기 기본, 읽기 오버라이드" 패턴. 실제로 Codex는 JPA를 사용하지 않아 트랜잭션의 실질적 효과는 제한적.

### 테스트 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **테스트 수** | 7개 | 2개 | 5개 |
| **테스트 방식** | Mockito Mock | Mockito Mock | 실제 InMemoryRepository 사용 |
| **프레임워크** | JUnit 5 + Mockito + AssertJ | JUnit 5 + Mockito + AssertJ | JUnit 5 + AssertJ |
| **한국어 테스트명** | `@DisplayName` 사용 | `@DisplayName` 사용 | 미사용 (영어) |
| **예외 테스트** | getTodoById_notFound | 없음 | getMissingTodoThrows |

#### 테스트 커버리지 상세

| 테스트 케이스 | Claude | Gemini | Codex |
|---------------|--------|--------|-------|
| 생성 | O | O | O |
| 전체 조회 | O | - | O |
| 단건 조회 | O | - | O (생성과 결합) |
| 수정 | O | O | O |
| 삭제 | O | - | O |
| 토글 | O | - | - |
| 조회 실패(404) | O | - | O |

> Claude가 가장 포괄적(7개), Codex가 실용적(5개), Gemini는 최소한(2개)

#### 테스트 설계 철학 차이
- **Claude/Gemini**: Mock 기반 단위 테스트 — Repository를 Mockito로 모킹하여 Service 로직만 격리 검증
- **Codex**: 통합 단위 테스트 — 실제 InMemoryRepository를 주입하여 Service+Repository를 함께 검증. Mock 불필요한 깔끔한 설계

### 프론트엔드(index.html) 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **API 연동** | REST API 호출 | REST API 호출 | localStorage (API 미연동) |
| **언어** | 한국어 UI | 한국어 UI | 영어 UI |
| **XSS 방지** | `escapeHtml()` 함수 구현 | 없음 | 없음 |
| **중복 제출 방지** | `submitting` 플래그 | 없음 | 없음 |
| **삭제 확인** | 없음 | confirm() 대화상자 | 없음 |
| **완료 통계** | 완료 카운터 표시 | 없음 | 없음 |
| **빈 상태 메시지** | O | 없음 | O |
| **Enter 키 지원** | O | O | O (form submit) |
| **디자인 색상** | 파란색 (#4a90d9) | 파란색 (#4A90E2) | 파란색 (#2563eb) |

> Claude의 프론트엔드가 가장 완성도 높음 (XSS 방지, 중복 제출 방지, 완료 통계). Codex는 REST API를 호출하지 않고 localStorage만 사용하여 백엔드와 분리되어 있음.

### 데이터베이스 설정 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **저장소 타입** | H2 파일 DB (`file:./data/tododb`) | H2 인메모리 (`mem:tododb`) | ConcurrentHashMap |
| **데이터 영속성** | 재시작 후 유지 | 재시작 시 소실 | 재시작 시 소실 |
| **H2 콘솔** | 활성 (`/h2-console`) | 활성 | 없음 |
| **SQL 로깅** | 활성 (포맷팅) | 활성 (포맷팅) | 없음 |
| **DDL 전략** | `ddl-auto: update` | `ddl-auto: update` | 해당 없음 |
| **로깅 레벨** | 기본값 | DEBUG (com.example.todo) | 기본값 |
| **스레드 안전** | DB 트랜잭션 | DB 트랜잭션 | ConcurrentHashMap + AtomicLong |

> Claude만 파일 기반 H2를 사용하여 데이터가 재시작 후에도 유지됨. Codex는 DB 자체를 사용하지 않고 자체 인메모리 저장소를 구현.

### 코드 품질 종합 평가

#### 평가 기준별 점수 (5점 만점)

| 평가 기준 | Claude | Gemini | Codex |
|-----------|:------:|:------:|:-----:|
| 아키텍처 설계 | ★★★★☆ | ★★★★☆ | ★★★★★ |
| 코드 가독성 | ★★★★★ | ★★★★☆ | ★★★★☆ |
| 입력 검증 | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| 예외 처리 | ★★★★☆ | ★★★★★ | ★★★☆☆ |
| 테스트 충실도 | ★★★★★ | ★★☆☆☆ | ★★★★☆ |
| 프론트엔드 완성도 | ★★★★★ | ★★★☆☆ | ★★☆☆☆ |
| 데이터 영속성 | ★★★★★ | ★★★☆☆ | ★★☆☆☆ |
| 확장성/유연성 | ★★★★☆ | ★★★☆☆ | ★★★★★ |
| 스레드 안전성 | ★★★☆☆ (DB 의존) | ★★★☆☆ (DB 의존) | ★★★★★ |
| 현대적 Java 활용 | ★★★★★ | ★★★★★ | ★★★☆☆ |

### 각 프로젝트의 강점과 약점

#### Claude — "가장 완성도 높은 풀스택 구현"
**강점:**
- 가장 완성도 높은 프론트엔드 (XSS 방지, 중복 제출 방지, 완료 통계)
- 풍부한 테스트 커버리지 (7개 테스트)
- 파일 기반 H2 DB로 실제 데이터 영속성 보장
- Java Record를 활용한 깔끔한 DTO
- 완료 토글 전용 PATCH 엔드포인트 제공
- description 필드로 더 풍부한 데이터 모델

**약점:**
- Bean Validation 미적용 (의존성 자체가 없음)
- 예외 처리가 Controller 로컬에 한정

#### Gemini — "예외 처리에 강한 백엔드 중심 구현"
**강점:**
- `@RestControllerAdvice` 전역 예외 핸들러 (400/404/500 모두 처리)
- Bean Validation 적용 (`@NotBlank`, `@Valid`)
- DEBUG 레벨 로깅 설정
- 삭제 시 confirm() 대화상자

**약점:**
- 테스트가 2개로 가장 적음
- updatedAt 필드 없음 (수정 시점 추적 불가)
- 인메모리 H2로 재시작 시 데이터 소실
- 프론트엔드에 XSS 방지 없음

#### Codex — "설계 원칙에 충실한 아키텍트형 구현"
**강점:**
- Repository 인터페이스 직접 정의 → DB 교체 용이한 설계
- ConcurrentHashMap + AtomicLong으로 스레드 안전성 직접 구현
- Create/Update Request DTO 분리로 명확한 API 계약
- `Instant` 사용으로 타임존 안전한 시간 관리
- `touch()` 패턴으로 깔끔한 타임스탬프 관리
- Spring Boot 3.5.0 (안정 버전) 선택

**약점:**
- 프론트엔드가 REST API를 호출하지 않음 (localStorage만 사용)
- DTO에 Java Record 미사용 (일반 클래스)
- JPA/DB 없이 인메모리만 사용 (프롬프트의 H2 요구사항 미충족)
- `@Transactional`이 실질적 효과 없음 (DB 미사용)

### 프롬프트 준수도 비교

CLAUDE.md에 명시된 요구사항 충족 여부:

| 요구사항 | Claude | Gemini | Codex |
|----------|:------:|:------:|:-----:|
| Controller→Service→Repository 3계층 | ✅ | ✅ | ✅ |
| Service에 @Transactional 적용 | ✅ | ✅ | ✅ |
| 가독성·확장성 우선 코드 | ✅ | ✅ | ✅ |
| H2 등 임베디드 DB 사용 | ✅ | ✅ | ❌ (인메모리 Map) |
| 단위 테스트 (Service 중심) | ✅ | ✅ (최소) | ✅ |
| REST API 코드 | ✅ | ✅ | ✅ |
| index.html 샘플 화면 | ✅ (API 연동) | ✅ (API 연동) | ⚠️ (localStorage만) |

### 종합 결론

**Claude** — 프론트엔드부터 데이터 영속성까지 가장 완성도 높은 결과물. 테스트도 가장 풍부하고, 프롬프트 요구사항을 가장 충실히 이행. 다만 Bean Validation 미적용은 아쉬운 점.

**Gemini** — 전역 예외 핸들러와 입력 검증이 가장 체계적. 프로덕션 수준의 에러 핸들링 패턴을 보여줌. 하지만 테스트 부족과 updatedAt 미관리는 약점.

**Codex** — Repository 인터페이스 추상화, DTO 분리, 스레드 안전성 등 소프트웨어 설계 원칙에 가장 충실. 그러나 프롬프트의 H2 DB 요구사항을 미충족하고, 프론트엔드가 백엔드와 연동되지 않는 점이 가장 큰 한계.

---

## Codex CLI 유용한 명령어 치트시트

```bash
# 세션 내 슬래시 명령어
/status           # 현재 모드, 모델, 샌드박스 상태 확인
/model            # 모델 변경
/approvals auto   # 승인 모드 변경
/review           # 내장 코드 리뷰 실행
/skills           # 사용 가능한 스킬 목록
/stats            # 세션 통계
/compact          # 컨텍스트 압축
/clear            # 컨텍스트 초기화

# 파일 참조
> @src/main/java/com/example/TodoService.java 이 파일 분석해줘

# CLI 실행 옵션
codex                                    # 기본 (workspace-write + on-request)
codex -s read-only                       # 읽기 전용 모드
codex --full-auto "태스크"                # 완전 자동화
codex -p review "리뷰해줘"               # 프로필 지정
codex exec "태스크"                       # 비대화형 (스크립팅용)
codex exec --full-auto "테스트 실행"      # CI용 완전 자동화

# config.toml 위치
~/.codex/config.toml                     # 글로벌 설정

# AGENTS.md (CLAUDE.md에 해당)
# 프로젝트 루트에 AGENTS.md를 만들면 Codex가 자동으로 읽음
```

---

## 실습 과제

### 과제 1: 설치 및 인증 (10분)
1. `npm install -g @openai/codex`
2. `codex` 실행 → ChatGPT 계정으로 로그인
3. `/status`로 모델, 샌드박스 모드 확인
4. 간단한 프롬프트로 동작 확인

### 과제 2: 샌드박스 모드 체험 (10분)
1. `codex -s read-only`로 코드 분석 요청 → 파일 수정 불가 확인
2. 기본 모드에서 파일 수정 요청 → 프로젝트 폴더 내 수정 확인
3. config.toml에 프로필 2개 설정 (review, with-network)

### 과제 3: 3도구 비교 실험 (25분)
1. 동일 프로젝트에서 3가지 태스크를 Claude/Gemini/Codex로 실행
2. 비교 매트릭스 작성
3. 상황별 도구 추천 초안 작성

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| Codex CLI | OpenAI의 Rust 기반 터미널 에이전트, ChatGPT Plus 포함 |
| 인증 | ChatGPT OAuth(추천) 또는 API 키 |
| 샌드박스 핵심 | OS 레벨 격리가 기본 — read-only / workspace-write / danger-full-access |
| Claude와 차이 | Claude=승인 기반 안전, Codex=OS 레벨 격리 |
| --full-auto | 승인 스킵 + workspace-write, CI/자동화에 적합 |
| config.toml | 프로필로 상황별 설정 전환 (review, with-network 등) |
| AGENTS.md | CLAUDE.md/GEMINI.md에 해당하는 프로젝트 컨텍스트 파일 |

---

## 참고 리소스

- Codex CLI 공식: [developers.openai.com/codex/cli](https://developers.openai.com/codex/cli/)
- Quickstart: [developers.openai.com/codex/quickstart](https://developers.openai.com/codex/quickstart/)
- GitHub: [github.com/openai/codex](https://github.com/openai/codex)
- Changelog: [developers.openai.com/codex/changelog](https://developers.openai.com/codex/changelog/)
- 인증 가이드: [developers.openai.com/codex/auth](https://developers.openai.com/codex/auth/)
- 샌드박스 심화: [How Codex CLI Flags Actually Work](https://www.vincentschmalbach.com/how-codex-cli-flags-actually-work-full-auto-sandbox-and-bypass/)

*분석일: 2026-03-11*
