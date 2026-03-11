# Spring Boot TODO API - AI 코드 생성 비교 분석

> 동일한 프롬프트(CLAUDE.md)로 Claude, Gemini, Codex 세 AI가 각각 생성한 Spring Boot TODO API 프로젝트를 비교 분석한 문서입니다.

---

## 1. 프로젝트 개요 비교

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

---

## 2. 아키텍처 비교

### 공통점
- Controller → Service → Repository 3계층 구조
- 생성자 주입(Constructor Injection) 사용
- DTO 패턴으로 요청/응답 분리
- Service 계층에 `@Transactional` 적용

### 패키지 구조 비교

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

### 차이점 분석

| 설계 관점 | Claude | Gemini | Codex |
|-----------|--------|--------|-------|
| **DTO 전략** | 단일 TodoRequest (생성/수정 공용) | 단일 TodoRequest (생성/수정 공용) | CreateTodoRequest / UpdateTodoRequest 분리 |
| **예외 처리** | Controller 내 `@ExceptionHandler` | `@RestControllerAdvice` 글로벌 핸들러 | `@ResponseStatus` 커스텀 예외 |
| **Repository** | JpaRepository 상속 | JpaRepository 상속 | 인터페이스 직접 정의 + 구현체 |
| **Entity 설계** | JPA Entity + `@PrePersist`/`@PreUpdate` | JPA Entity + 생성자 초기화 | POJO + `touch()` 메서드 |
| **DTO 타입** | Java Record | Java Record | 일반 Class (getter/setter) |

---

## 3. API 엔드포인트 비교

| 기능 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| 전체 조회 | `GET /api/todos` | `GET /api/todos` | `GET /api/todos` |
| 단건 조회 | `GET /api/todos/{id}` | `GET /api/todos/{id}` | `GET /api/todos/{id}` |
| 생성 | `POST /api/todos` (201) | `POST /api/todos` (201) | `POST /api/todos` (201) |
| 수정 | `PUT /api/todos/{id}` | `PUT /api/todos/{id}` | `PUT /api/todos/{id}` |
| 삭제 | `DELETE /api/todos/{id}` (204) | `DELETE /api/todos/{id}` (204) | `DELETE /api/todos/{id}` (204) |
| 완료 토글 | `PATCH /api/todos/{id}/toggle` | - | - |

> Claude만 완료 토글 전용 엔드포인트를 별도 제공 (6개 엔드포인트 vs 5개)

---

## 4. 데이터 모델 비교

| 필드 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| id | `Long` (IDENTITY) | `Long` (IDENTITY) | `long` (AtomicLong) |
| title | `String` (NOT NULL) | `String` (NOT NULL) | `String` |
| description | `String` (nullable) | - | - |
| completed | `boolean` | `boolean` | `boolean` |
| createdAt | `LocalDateTime` | `LocalDateTime` | `Instant` |
| updatedAt | `LocalDateTime` | - | `Instant` |

### 주요 차이
- **Claude**: description 필드 포함, updatedAt 관리, `@PrePersist`/`@PreUpdate` JPA 콜백 활용
- **Gemini**: 최소한의 필드만 사용, updatedAt 없음
- **Codex**: `Instant`(UTC) 사용으로 타임존 안전, `touch()` 메서드로 수동 타임스탬프 관리, description 없음

---

## 5. 입력 검증(Validation) 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **Validation 의존성** | 없음 | `spring-boot-starter-validation` | `spring-boot-starter-validation` |
| **@Valid 사용** | 미사용 | 사용 | 사용 |
| **@NotBlank** | 미사용 | title에 적용 | title에 적용 |
| **검증 에러 핸들링** | 없음 | GlobalExceptionHandler에서 처리 | Spring 기본 처리 |

> Claude는 validation 의존성 없이 서비스 레벨에서 null 체크만 수행. Gemini와 Codex는 Bean Validation 표준을 따름.

---

## 6. 예외 처리 전략 비교

### Claude
```java
// Controller 내부 @ExceptionHandler
@ExceptionHandler(NoSuchElementException.class)
public ResponseEntity<String> handleNotFound(NoSuchElementException e) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getMessage());
}
```
- NoSuchElementException을 Controller에서 직접 처리
- 단순 문자열 응답

### Gemini
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

### Codex
```java
// 커스텀 예외 + @ResponseStatus
@ResponseStatus(HttpStatus.NOT_FOUND)
public class TodoNotFoundException extends RuntimeException { }
```
- 가장 간결한 접근
- Spring의 `@ResponseStatus` 활용

---

## 7. 트랜잭션 관리 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| 클래스 레벨 | `@Transactional(readOnly = true)` | `@Transactional(readOnly = true)` | `@Transactional` |
| 읽기 메서드 | 클래스 레벨 상속 | 클래스 레벨 상속 | `@Transactional(readOnly = true)` 개별 적용 |
| 쓰기 메서드 | `@Transactional` 개별 오버라이드 | `@Transactional` 개별 오버라이드 | 클래스 레벨 상속 |

> Claude와 Gemini는 "읽기 기본, 쓰기 오버라이드" 패턴. Codex는 "쓰기 기본, 읽기 오버라이드" 패턴. 실제로 Codex는 JPA를 사용하지 않아 트랜잭션의 실질적 효과는 제한적.

---

## 8. 테스트 비교

| 항목 | Claude | Gemini | Codex |
|------|--------|--------|-------|
| **테스트 수** | 7개 | 2개 | 5개 |
| **테스트 방식** | Mockito Mock | Mockito Mock | 실제 InMemoryRepository 사용 |
| **프레임워크** | JUnit 5 + Mockito + AssertJ | JUnit 5 + Mockito + AssertJ | JUnit 5 + AssertJ |
| **한국어 테스트명** | `@DisplayName` 사용 | `@DisplayName` 사용 | 미사용 (영어) |
| **예외 테스트** | getTodoById_notFound | 없음 | getMissingTodoThrows |

### 테스트 커버리지 상세

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

### 테스트 설계 철학 차이
- **Claude/Gemini**: Mock 기반 단위 테스트 — Repository를 Mockito로 모킹하여 Service 로직만 격리 검증
- **Codex**: 통합 단위 테스트 — 실제 InMemoryRepository를 주입하여 Service+Repository를 함께 검증. Mock 불필요한 깔끔한 설계

---

## 9. 프론트엔드(index.html) 비교

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

---

## 10. 데이터베이스 설정 비교

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

---

## 11. 코드 품질 종합 평가

### 평가 기준별 점수 (5점 만점)

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

---

## 12. 각 프로젝트의 강점과 약점

### Claude
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

### Gemini
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

### Codex
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

---

## 13. 프롬프트 준수도 비교

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

---

## 14. 종합 결론

### Claude — "가장 완성도 높은 풀스택 구현"
프론트엔드부터 데이터 영속성까지 가장 완성도 높은 결과물. 테스트도 가장 풍부하고, 프롬프트 요구사항을 가장 충실히 이행. 다만 Bean Validation 미적용은 아쉬운 점.

### Gemini — "예외 처리에 강한 백엔드 중심 구현"
전역 예외 핸들러와 입력 검증이 가장 체계적. 프로덕션 수준의 에러 핸들링 패턴을 보여줌. 하지만 테스트 부족과 updatedAt 미관리는 약점.

### Codex — "설계 원칙에 충실한 아키텍트형 구현"
Repository 인터페이스 추상화, DTO 분리, 스레드 안전성 등 소프트웨어 설계 원칙에 가장 충실. 그러나 프롬프트의 H2 DB 요구사항을 미충족하고, 프론트엔드가 백엔드와 연동되지 않는 점이 가장 큰 한계.

---

*분석일: 2026-03-11*
