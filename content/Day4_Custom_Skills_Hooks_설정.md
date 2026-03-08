# Day 4: Custom Skills & Hooks 설정

> AI Tools Mastery Curriculum — Week 1, Day 4
> 소요 시간: 50분 | 실습 중심

---

## ① .claude/skills/ 폴더 구조 & SKILL.md 작성

### Skills란

Skills는 Claude Code에게 **필요할 때만 로드되는 지식 패키지**다.
CLAUDE.md가 "항상 읽는 프로젝트 설명서"라면, Skills는 "필요할 때 꺼내보는 전문 매뉴얼"이다.

핵심 설계 원칙은 **Progressive Disclosure(점진적 공개)** 다:

```
1단계: 메타데이터만 스캔 (~100 토큰)
  → Claude가 이 스킬이 지금 필요한지 판단

2단계: SKILL.md 본문 로드 (<5,000 토큰)
  → 필요하다고 판단되면 지시사항 로드

3단계: 보조 파일 로드 (필요 시)
  → scripts/, references/, assets/ 의 파일을 필요할 때만 읽음
```

이 구조 덕분에 스킬을 10개, 20개 설치해놓아도 실제로 사용하는 스킬만 컨텍스트를 소모한다.

### 폴더 구조

```
.claude/skills/
├── api-test-generator/
│   ├── SKILL.md              ← 필수: 메타데이터 + 지시사항
│   ├── scripts/              ← 선택: 실행 가능한 스크립트
│   │   └── generate_test.py
│   ├── references/           ← 선택: 참조 문서 (필요시 로드)
│   │   └── test-patterns.md
│   └── assets/               ← 선택: 템플릿, 아이콘 등
│       └── test-template.java
│
└── code-reviewer/
    ├── SKILL.md
    └── references/
        └── review-checklist.md
```

**저장 위치에 따른 범위:**

| 위치 | 범위 | 용도 |
|---|---|---|
| `~/.claude/skills/` | 개인 전역 (모든 프로젝트) | 개인 워크플로우 |
| `.claude/skills/` (프로젝트 루트) | 프로젝트 전체 | 팀 공유 |

### SKILL.md 작성법

SKILL.md는 두 부분으로 구성된다:

```markdown
---
# ① YAML Frontmatter (메타데이터)
name: api-test-generator
description: >
  Spring Boot REST API의 단위 테스트를 자동 생성한다.
  API 테스트, 컨트롤러 테스트, MockMvc 테스트를 요청할 때 사용한다.
  테스트 코드 작성을 요청받으면 항상 이 스킬을 확인할 것.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# ② Markdown 본문 (지시사항)

# API Test Generator

Spring Boot REST API에 대한 단위 테스트를 생성한다.

## 테스트 작성 규칙

1. MockMvc를 사용한 컨트롤러 테스트 작성
2. @WebMvcTest 어노테이션 사용 (전체 컨텍스트 로딩 금지)
3. 서비스 의존성은 @MockBean으로 모킹
4. 정상 케이스, 유효성 검증 실패, 인증 실패 3가지 시나리오 필수

## 테스트 구조

ALWAYS use this exact structure:

```java
@WebMvcTest(SomeController.class)
class SomeControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean SomeService someService;

    @Test @DisplayName("정상 요청 시 200 반환")
    void success_case() { ... }

    @Test @DisplayName("잘못된 입력 시 400 반환")
    void validation_fail() { ... }

    @Test @DisplayName("인증 없이 요청 시 401 반환")
    void unauthorized() { ... }
}
```

## 참고

상세 테스트 패턴은 `references/test-patterns.md`를 참고한다.
```

### Frontmatter 필드 상세

| 필드 | 필수 | 설명 |
|---|---|---|
| `name` | ✅ | 스킬 이름, `/slash-command`로도 사용됨 |
| `description` | ✅ | Claude가 이 스킬을 언제 사용할지 판단하는 기준 |
| `allowed-tools` | 선택 | 이 스킬 활성화 시 사용 가능한 도구 제한 |
| `disable-model-invocation` | 선택 | `true`로 설정하면 자동 호출 방지 (수동만 가능) |
| `user-invocable` | 선택 | `false`로 설정하면 /메뉴에서 숨김 (백그라운드 지식) |
| `argument-hint` | 선택 | 자동완성 힌트 (예: `[class-name]`) |

### description 작성 팁

Claude는 현재 스킬을 **과소 호출(undertrigger)** 하는 경향이 있다.
따라서 description은 약간 "적극적으로" 쓰는 것이 좋다:

```yaml
# ❌ 소극적 (잘 트리거 안 됨)
description: API 테스트 생성 도구

# ✅ 적극적 (잘 트리거 됨)
description: >
  Spring Boot REST API의 단위 테스트를 자동 생성한다.
  API 테스트, 컨트롤러 테스트, MockMvc 테스트를 요청할 때 사용한다.
  테스트 코드 작성, 테스트 커버리지 개선, 테스트 리팩토링을 
  언급하면 항상 이 스킬을 확인할 것.
```

### SKILL.md 본문 작성 원칙

**1,500~2,000단어를 목표**로 한다. 이보다 길어지면 references/에 분리한다.

작성 시 핵심은 **"다른 Claude 인스턴스에게 알려주는 것"** 이라는 관점이다.
Claude가 이미 알고 있는 일반적인 내용은 빼고, 프로젝트에 특화된 비자명한 정보를 담는다:

```
❌ "MockMvc는 Spring의 테스트 도구입니다" (Claude가 이미 앎)
✅ "우리 프로젝트에서는 @SpringBootTest 대신 @WebMvcTest를 쓴다" (프로젝트 고유)
```

### 스킬 호출 방법

```bash
# 방법 1: 슬래시 커맨드로 직접 호출
> /api-test-generator CourseController

# 방법 2: 자연어로 요청 (Claude가 description 기반으로 자동 판단)
> CourseController에 대한 단위 테스트를 작성해줘

# 방법 3: 명시적 지정
> api-test-generator 스킬을 사용해서 테스트를 만들어줘
```

---

## ② 반복 작업용 커스텀 스킬 1개 제작

### 실전 제작: "Gradle 모듈 스캐폴딩" 스킬

Spring Boot 멀티 모듈 프로젝트에서 새 모듈을 추가할 때마다 반복하는 작업을 스킬로 만든다.

#### Step 1: 폴더 생성

```bash
mkdir -p .claude/skills/module-scaffold/references
```

#### Step 2: SKILL.md 작성

```markdown
---
name: module-scaffold
description: >
  Spring Boot 멀티 모듈 프로젝트에 새 Gradle 모듈을 추가한다.
  새 모듈 생성, 모듈 추가, 서브 프로젝트 생성을 요청할 때 사용한다.
  모듈 구조나 프로젝트 세팅과 관련된 작업에도 이 스킬을 확인할 것.
argument-hint: [module-name]
allowed-tools: Read, Write, Bash, Glob
---

# Module Scaffold

Spring Boot 멀티 모듈 프로젝트에 새 모듈을 생성한다.

## 생성 절차

1. 모듈 디렉토리 생성
2. build.gradle 작성
3. 패키지 구조 생성
4. settings.gradle에 include 추가
5. 루트 build.gradle에 의존성 추가 (필요시)

## 디렉토리 구조

새 모듈은 반드시 다음 구조를 따른다:

```
{module-name}/
├── build.gradle
└── src/
    ├── main/
    │   ├── java/com/example/{module}/
    │   │   ├── controller/
    │   │   ├── service/
    │   │   ├── repository/
    │   │   ├── domain/
    │   │   └── dto/
    │   └── resources/
    │       └── application.yml
    └── test/
        └── java/com/example/{module}/
```

## build.gradle 템플릿

```groovy
plugins {
    id 'java-library'
    id 'org.springframework.boot' apply false
    id 'io.spring.dependency-management'
}

dependencies {
    implementation project(':core')
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

## 체크리스트

생성 후 반드시 확인:
- [ ] settings.gradle에 include 추가됨
- [ ] ./gradlew :새모듈:build 성공
- [ ] core 모듈 의존성 정상
- [ ] 패키지 네이밍 컨벤션 준수 (com.example.{module})
```

#### Step 3: 테스트

```bash
claude
> /module-scaffold notification
# 또는
> 새 모듈 "notification"을 추가해줘
```

Claude가 스킬의 지시대로 디렉토리 구조, build.gradle, settings.gradle 수정까지 일관되게 수행하는지 확인한다.

### 더 만들어볼 만한 스킬 아이디어

| 스킬 이름                  | 용도                                                 | 반복 빈도   |
| ---------------------- | -------------------------------------------------- | ------- |
| `api-docs-generator`   | API 엔드포인트에서 Swagger/OpenAPI 문서 자동 생성               | 매주      |
| `entity-crud`          | JPA 엔티티 → Repository/Service/Controller CRUD 일괄 생성 | 새 엔티티마다 |
| `migration-script`     | Flyway/Liquibase DB 마이그레이션 스크립트 생성                 | 스키마 변경시 |
| `pr-description`       | 변경사항 분석 → PR 설명 자동 작성                              | 매 PR    |
| `architecture-diagram` | Mermaid로 아키텍처 다이어그램 생성                             | 구조 변경시  |

---

## ③ Hooks(pre/post commit) 설정 및 테스트

### Hooks란

Hooks는 Claude Code의 **특정 시점에 자동으로 실행되는 셸 커맨드**다.
Skills가 "지식"이라면, Hooks는 "자동화된 행동 규칙"이다.

핵심 차이:
- **Skills**: Claude의 응답 품질을 높이는 **지식** (프롬프트 확장)
- **Hooks**: Claude의 행동에 **자동으로 반응**하는 코드 (이벤트 트리거)

프롬프트로 "항상 포맷팅해줘"라고 하면 Claude가 가끔 까먹는다.
Hooks로 설정하면 **매번 100% 실행된다.** 프롬프트는 제안이고, Hooks는 보장이다.

### Hook 이벤트 종류

| 이벤트 | 발생 시점 | 대표 용도 |
|---|---|---|
| `PreToolUse` | 도구 실행 **직전** | 위험한 명령 차단, 파일 보호 |
| `PostToolUse` | 도구 실행 **직후** | 자동 포맷팅, 린트, 자동 커밋 |
| `Stop` | Claude 응답 완료 시 | 알림 전송, 로그 기록 |
| `SessionStart` | 세션 시작 시 | 환경 설정, 컨텍스트 주입 |
| `Notification` | Claude가 입력을 기다릴 때 | 데스크탑 알림 |

### Hook 설정 파일 위치

```
~/.claude/settings.json          ← 개인 전역 (모든 프로젝트)
.claude/settings.json            ← 프로젝트 공유 (Git 커밋됨)
.claude/settings.local.json      ← 프로젝트 로컬 (Git 무시됨)
```

### 실전 Hook 1: 파일 수정 후 자동 포맷팅

Claude가 파일을 수정할 때마다 자동으로 Prettier/포맷터를 실행한다.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**동작 원리:**
1. Claude가 Edit 또는 Write 도구를 사용 (파일 수정)
2. PostToolUse 이벤트 발생
3. matcher가 "Edit|Write"에 매칭
4. jq로 수정된 파일 경로를 추출
5. Prettier로 해당 파일 포맷팅

### 실전 Hook 2: 위험한 명령 차단

Claude가 `rm -rf`, `git reset --hard` 같은 위험한 명령을 실행하려 할 때 차단한다.

먼저 스크립트를 만든다:

```bash
#!/usr/bin/env bash
# .claude/hooks/pre-bash-firewall.sh

set -euo pipefail

COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command')

# 위험한 명령어 패턴 체크
if echo "$COMMAND" | grep -qE '(rm -rf|git reset --hard|git push -f|DROP TABLE|DROP DATABASE)'; then
  echo "BLOCKED: 위험한 명령어가 감지되었습니다: $COMMAND" >&2
  exit 2  # exit 2 = 차단 (Claude에게 이유를 알려줌)
fi

exit 0  # exit 0 = 허용
```

```bash
chmod +x .claude/hooks/pre-bash-firewall.sh
```

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/pre-bash-firewall.sh"
          }
        ]
      }
    ]
  }
}
```

**Exit 코드 규칙:**
- `0` = 허용 (통과)
- `2` = 차단 (stderr 메시지가 Claude에게 전달됨)
- 그 외 = 비차단 에러 (경고만 표시)

### 실전 Hook 3: git commit 전 린트/테스트 실행

Claude가 `git commit`을 하려고 할 때, 먼저 린트와 테스트를 실행한다.

```bash
#!/usr/bin/env bash
# .claude/hooks/pre-commit-check.sh

set -euo pipefail

COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command')

# git commit 명령인지 확인
if echo "$COMMAND" | grep -q '^git commit'; then
  echo "Pre-commit 검사를 실행합니다..."
  
  # Checkstyle/Lint 실행
  if ! ./gradlew checkstyleMain 2>/dev/null; then
    echo "BLOCKED: Checkstyle 검사 실패. 코드 스타일을 수정해주세요." >&2
    exit 2
  fi
  
  # 단위 테스트 실행
  if ! ./gradlew test 2>/dev/null; then
    echo "BLOCKED: 단위 테스트 실패. 테스트를 수정해주세요." >&2
    exit 2
  fi
  
  echo "모든 검사 통과!"
fi

exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/pre-commit-check.sh",
            "timeout": 180
          }
        ]
      }
    ]
  }
}
```

`timeout: 180`은 테스트 실행이 오래 걸릴 수 있으므로 3분 제한을 설정한 것이다.

### 실전 Hook 4: Claude 작업 완료 시 데스크탑 알림

Claude가 응답을 마치면 데스크탑 알림을 보낸다. 긴 작업을 시켜놓고 다른 일을 할 때 유용하다.

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.message' | xargs -I {} osascript -e 'display notification \"{}\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

(macOS 기준. Linux는 `notify-send`로 대체)

### Hook 설정 및 테스트 방법

```bash
# 방법 1: /hooks 명령어로 인터랙티브 설정
claude
> /hooks

# 방법 2: settings.json 직접 편집
vi .claude/settings.json

# 설정 확인
> /hooks  ← 현재 등록된 훅 목록 확인

# 디버깅
claude --debug  ← 훅 실행 상세 로그 확인
Ctrl+O          ← verbose 모드 토글 (훅 출력 표시)
```

### Hook 디버깅 팁

훅이 동작하지 않을 때 체크할 것:

```
1. matcher가 도구 이름과 정확히 일치하는가? (대소문자 구분)
   - 올바름: "Edit|Write"
   - 틀림: "edit|write"

2. 스크립트에 실행 권한이 있는가?
   - chmod +x .claude/hooks/your-script.sh

3. 셸 프로파일의 echo가 JSON 파싱을 방해하지 않는가?
   - ~/.zshrc에 무조건적 echo가 있으면 훅 출력에 섞임
   - 해결: echo를 if [[ $- == *i* ]]; then ... fi 로 감싸기

4. 올바른 이벤트 타입인가?
   - 차단은 PreToolUse에서만 가능 (exit 2)
   - PostToolUse에서는 이미 실행된 후라 차단 불가
```

---

## 실습 과제

### 과제 1: 스킬 제작 (20분)
1. 자신의 프로젝트에서 자주 반복하는 작업 하나를 골라 스킬로 제작
2. `.claude/skills/{name}/SKILL.md` 생성
3. `/skill-name`으로 호출 테스트
4. 자연어 요청으로 자동 트리거되는지 확인

### 과제 2: Hook 3개 설정 (20분)
1. PostToolUse: 파일 수정 후 자동 포맷팅
2. PreToolUse: 위험 명령어 차단
3. Notification: 작업 완료 알림
4. 각 훅이 동작하는지 테스트

### 과제 3: GitHub에 공유 (10분)
1. `.claude/` 폴더를 Git에 커밋
2. `settings.local.json`은 `.gitignore`에 추가
3. 팀이 clone 후 바로 사용할 수 있는 상태인지 확인

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| Skills 본질 | 필요할 때만 로드되는 지식 패키지 (Progressive Disclosure) |
| SKILL.md 구조 | YAML Frontmatter(메타데이터) + Markdown(지시사항) |
| description 팁 | 적극적으로 작성 ("~할 때 항상 이 스킬을 확인할 것") |
| 본문 분량 | 1,500~2,000단어, 길면 references/로 분리 |
| Hooks 본질 | 이벤트 기반 자동 실행 셸 커맨드 (프롬프트는 제안, 훅은 보장) |
| Exit 코드 | 0=허용, 2=차단(stderr→Claude), 그 외=경고 |
| 핵심 패턴 | PreToolUse=방어, PostToolUse=정리, Notification=알림 |

---

## 참고 리소스

- Claude Code 공식: [Extend Claude with skills](https://code.claude.com/docs/en/skills)
- Claude Code 공식: [Automate workflows with hooks](https://code.claude.com/docs/en/hooks-guide)
- Anthropic: [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Anthropic GitHub: [Skills 레포지토리](https://github.com/anthropics/skills) — 공식 스킬 예시 모음
- 커뮤니티: [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
- 실전 가이드: [Demystifying Claude Code Hooks](https://www.brethorsting.com/blog/2025/08/demystifying-claude-code-hooks/)
