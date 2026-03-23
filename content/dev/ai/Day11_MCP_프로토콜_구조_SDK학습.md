---
title: "Day 11: MCP 프로토콜 구조 & SDK 학습"
date: 2026-03-17
description: "MCP(Model Context Protocol) 스펙을 정독하고 SDK를 학습하여 AI 도구 확장의 기반을 다진다."
tags:
  - mcp
  - sdk
  - week3
---

# Day 11: MCP 프로토콜 구조 & SDK 학습

> AI Tools Mastery Curriculum — Week 3, Day 11
> 소요 시간: 50분 | 학습 + 실습

---

## ① MCP 스펙 문서 정독

### MCP란

MCP(Model Context Protocol)는 **LLM 애플리케이션과 외부 데이터/도구를 연결하는 오픈 프로토콜**이다. 2024년 11월 Anthropic이 공개했고, 현재는 Linux Foundation 산하 Agentic AI Foundation(AAIF)에서 관리한다.

비유하면 **USB-C**와 같다. USB-C 이전에는 기기마다 다른 케이블이 필요했듯이, MCP 이전에는 각 AI 도구와 외부 시스템을 연결할 때마다 커스텀 커넥터를 만들어야 했다. MCP는 이 N×M 문제를 해결한다:

```
MCP 없이:
  Claude ─── 커스텀 코드 ─── GitHub
  Claude ─── 커스텀 코드 ─── Jira
  Cursor ─── 커스텀 코드 ─── GitHub    (또 만들어야 함!)
  Cursor ─── 커스텀 코드 ─── Jira      (또 만들어야 함!)

MCP 있으면:
  Claude ─┐                ┌─── GitHub MCP 서버
  Cursor ─┤── MCP 프로토콜 ──┼─── Jira MCP 서버
  Codex  ─┘                └─── Slack MCP 서버
```

### 아키텍처: Host / Client / Server

MCP는 세 가지 역할로 구성된다:

```
┌─────────────────────────────────────────────┐
│ Host (Claude Desktop, Cursor, IDE 등)        │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Client A │  │ Client B │  │ Client C │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
└───────┼──────────────┼──────────────┼───────┘
        │              │              │
   MCP Protocol   MCP Protocol   MCP Protocol
        │              │              │
   ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐
   │ Server A │  │ Server B │  │ Server C │
   │ (GitHub) │  │ (Jira)   │  │ (Slack)  │
   └──────────┘  └──────────┘  └──────────┘
```

| 역할 | 설명 | 예시 |
|---|---|---|
| **Host** | MCP 클라이언트를 관리하는 LLM 애플리케이션 | Claude Desktop, Cursor, Claude Code |
| **Client** | Host 안에서 Server와 1:1로 연결되는 커넥터 | Host 내부에 자동 생성 |
| **Server** | 외부 도구/데이터를 MCP로 노출하는 서비스 | GitHub MCP, Jira MCP, 내가 만들 서버 |

**우리가 만들 것은 Server다.** Client와 Host는 Claude Code/Cursor 등이 이미 구현해놓았다.

### Server가 제공하는 3가지 기능

| 기능 | 설명 | 비유 |
|---|---|---|
| **Tools** | LLM이 호출할 수 있는 함수 (부작용 가능) | "이 API를 호출해줘" |
| **Resources** | LLM이 읽을 수 있는 데이터 (읽기 전용) | "이 파일을 읽어줘" |
| **Prompts** | 재사용 가능한 프롬프트 템플릿 | "이 형식으로 질문해" |

실무에서 가장 많이 쓰는 건 **Tools**다. Jira 티켓 조회, GitHub PR 생성, DB 쿼리 실행 등이 모두 Tool로 구현된다.

```
Tool 예시:
  - get_jira_ticket(ticket_id) → 티켓 상세 정보 반환
  - create_github_pr(title, body, branch) → PR 생성
  - run_sql_query(query) → 쿼리 결과 반환

Resource 예시:
  - project://docs/architecture.md → 아키텍처 문서
  - config://database/schema → DB 스키마 정보

Prompt 예시:
  - code-review(language) → "이 {language} 코드를 리뷰해줘..."
```

### 통신: JSON-RPC 2.0 over Transport

MCP는 **JSON-RPC 2.0** 메시지 포맷을 사용하고, 두 가지 전송 방식을 지원한다:

| Transport | 용도 | 사용 시나리오 |
|---|---|---|
| **stdio** | 로컬 프로세스 간 통신 | Claude Code에서 로컬 MCP 서버 실행 |
| **Streamable HTTP** | 원격 서버 통신 | 클라우드에 배포된 MCP 서버 |

우리가 만들 서버는 **stdio**부터 시작한다. Claude Code가 MCP 서버를 로컬 프로세스로 실행하고, stdin/stdout으로 JSON-RPC 메시지를 주고받는 방식이다.

### 메시지 흐름 예시

Claude Code에서 "Jira 티켓 CLM-123 정보 알려줘"라고 하면:

```
1. Claude가 Jira MCP 서버의 get_ticket 도구가 적합하다고 판단

2. Client → Server (JSON-RPC Request):
   {
     "jsonrpc": "2.0",
     "id": 1,
     "method": "tools/call",
     "params": {
       "name": "get_ticket",
       "arguments": { "ticket_id": "CLM-123" }
     }
   }

3. Server → Client (JSON-RPC Response):
   {
     "jsonrpc": "2.0",
     "id": 1,
     "result": {
       "content": [
         {
           "type": "text",
           "text": "CLM-123: 배송 지연 알림 시스템 구현\n상태: In Progress\n담당자: Min"
         }
       ]
     }
   }

4. Claude가 응답을 받아서 사용자에게 자연어로 답변
```

### 현재 스펙 버전

2026년 3월 기준, 최신 안정 스펙은 **2025-11-25** 버전이다. 주요 추가 사항:

- **Tasks**: 비동기 장시간 작업 지원 (실험적)
- **OAuth 2.1**: 원격 서버 인증 표준화
- **Elicitation**: 서버가 사용자에게 입력을 요청하는 기능

> **참고 문서**: [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-11-25)

---

## ② TypeScript SDK 설치 & 환경 구성

### SDK 버전 선택

| 버전 | 상태 | 추천 |
|---|---|---|
| v1.x | 안정 (프로덕션 권장) | ✅ 이번 학습에 사용 |
| v2.x | 프리뷰 (2026 Q1 정식 예정) | 새 API 구조, 아직 불안정 |

### 프로젝트 생성

```bash
# 프로젝트 디렉토리 생성
mkdir my-first-mcp-server
cd my-first-mcp-server

# npm 초기화
npm init -y

# SDK + 의존성 설치
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript

# 소스 디렉토리 생성
mkdir src
touch src/index.ts
```

### package.json 설정

```json
{
  "name": "my-first-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "my-mcp-server": "./build/index.js"
  },
  "scripts": {
    "build": "tsc && chmod 755 build/index.js",
    "watch": "tsc --watch"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.0",
    "zod": "^3.25.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "typescript": "^5.8.0"
  }
}
```

**핵심 포인트:**
- `"type": "module"` — ES 모듈 사용 (MCP SDK 필수)
- `zod` — 스키마 검증 라이브러리 (SDK의 필수 의존성)

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### 핵심 import 구조

SDK에서 자주 쓰는 import는 크게 3가지다:

```typescript
// 1. 서버 생성
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";

// 2. stdio 전송 (로컬 실행용)
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

// 3. 스키마 검증
import { z } from "zod";
```

---

## ③ 공식 예제 서버 구동 및 코드 분석

### 최소 MCP 서버 작성

`src/index.ts`:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// 1. 서버 인스턴스 생성
const server = new McpServer({
  name: "my-first-mcp-server",
  version: "1.0.0",
});

// 2. Tool 등록: 두 수를 더하는 계산기
server.registerTool(
  "add",                                      // 도구 이름
  {
    title: "Addition Tool",                    // UI 표시 이름
    description: "두 수를 더합니다",             // Claude가 이걸 보고 도구 사용 판단
    inputSchema: {
      a: z.number().describe("첫 번째 숫자"),
      b: z.number().describe("두 번째 숫자"),
    },
  },
  async ({ a, b }) => ({                       // 핸들러
    content: [
      {
        type: "text",
        text: `${a} + ${b} = ${a + b}`,
      },
    ],
  })
);

// 3. Tool 등록: 현재 시간 반환
server.registerTool(
  "current_time",
  {
    description: "현재 서버 시간을 반환합니다",
  },
  async () => ({
    content: [
      {
        type: "text",
        text: new Date().toISOString(),
      },
    ],
  })
);

// 4. stdio 전송으로 연결
const transport = new StdioServerTransport();
await server.connect(transport);

// STDIO 서버에서는 console.log() 사용 금지!
// stdout이 JSON-RPC 통신에 사용되므로 console.error()만 사용
console.error("MCP Server started");
```

### 코드 구조 분석

```
MCP 서버의 기본 구조:

1. McpServer 인스턴스 생성
   └─ name, version 필수

2. 기능 등록 (하나 이상)
   ├─ server.registerTool()     → LLM이 호출하는 함수
   ├─ server.registerResource() → LLM이 읽는 데이터
   └─ server.registerPrompt()   → 재사용 프롬프트 템플릿

3. Transport 연결
   ├─ StdioServerTransport → 로컬 (Claude Code)
   └─ StreamableHTTPServerTransport → 원격 (HTTP)

4. server.connect(transport) 호출
```

> **참고:** 이전 API인 `server.tool()`, `server.resource()`, `server.prompt()`는
> deprecated 되었다. 새 코드에서는 반드시 `server.registerTool()` 등 register* API를 사용할 것.

### server.registerTool() 파라미터 상세

```typescript
server.registerTool(
  "tool_name",             // (1) 도구 이름 — LLM이 호출할 때 사용
  {                        // (2) 설정 객체
    title: "표시 이름",     //   - UI에 보여줄 이름 (선택)
    description: "도구 설명", //  - LLM이 이 도구를 쓸지 판단하는 기준
    inputSchema: {          //   - zod로 정의한 파라미터
      param1: z.string(),
      param2: z.number().optional(),
    },
  },
  async ({ param1, param2 }) => ({  // (3) 핸들러 — 실제 로직
    content: [
      { type: "text", text: "결과" }
    ],
  })
);
```

**server.tool()과 달리 설정이 하나의 객체로 합쳐졌다.**
description, inputSchema, title을 한 객체 안에 넣는 구조다.

**Skills의 description처럼, 여기서도 description이 매우 중요하다.** Claude가 이 설명을 보고 "이 도구를 쓸지 말지" 판단한다.

### 빌드 & 테스트

```bash
# 빌드
npm run build

# MCP Inspector로 테스트 (브라우저 기반 디버깅 도구)
npx @modelcontextprotocol/inspector node build/index.js
```

MCP Inspector가 `http://127.0.0.1:6274`에서 열린다:
1. Transport type: `stdio` 선택
2. Command: `node` 입력
3. Arguments: `build/index.js` 입력
4. Connect 클릭
5. Tools 탭에서 **List Tools** 클릭 → `add`와 `current_time` 확인
6. `add` 선택 → a: 10, b: 20 입력 → Run tool
7. 결과: "10 + 20 = 30" 확인

### Claude Code에 연결하기

```bash
# Claude Code에 MCP 서버 등록
claude mcp add my-first-mcp-server node /절대경로/my-first-mcp-server/build/index.js
```

등록 후 Claude Code에서 확인:

```bash
claude

> 10과 20을 더해줘
# Claude가 my-first-mcp-server의 add 도구를 호출
# 결과: "10 + 20 = 30"

> 지금 몇 시야?
# Claude가 current_time 도구를 호출
# 결과: "2026-03-14T15:30:00.000Z"
```

### STDIO 서버의 중요 주의사항

```
⚠️ STDIO 서버에서 console.log() 사용 금지!

stdout은 JSON-RPC 메시지 통신에 사용된다.
console.log()가 stdout에 출력하면 JSON-RPC 메시지가 깨진다.

❌ console.log("디버그 메시지")     → stdout → 통신 방해
✅ console.error("디버그 메시지")   → stderr → 안전
```

---

## 실습 과제

### 과제 1: MCP 아키텍처 이해 (10분)
1. [modelcontextprotocol.io](https://modelcontextprotocol.io) 방문
2. Architecture → Core Architecture 섹션 읽기
3. Host/Client/Server 관계를 자신의 말로 정리
4. Tool/Resource/Prompt 차이를 정리

### 과제 2: SDK 설치 & 서버 작성 (25분)
1. 위 가이드대로 프로젝트 생성
2. 최소 MCP 서버 코드 작성
3. `npm run build`로 빌드 확인
4. MCP Inspector로 도구 동작 확인

### 과제 3: Claude Code 연결 (15분)
1. `claude mcp add`로 서버 등록
2. Claude Code에서 도구 호출 테스트
3. 도구가 자동으로 선택되는지 확인
4. `claude mcp list`로 등록된 서버 목록 확인

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| MCP 역할 | LLM과 외부 도구/데이터를 연결하는 오픈 프로토콜 |
| 아키텍처 | Host(IDE) → Client(커넥터) → Server(우리가 만들 것) |
| 3가지 기능 | Tools(함수 호출), Resources(데이터 읽기), Prompts(템플릿) |
| 통신 | JSON-RPC 2.0 / stdio(로컬) 또는 Streamable HTTP(원격) |
| SDK | `@modelcontextprotocol/sdk` + `zod` (필수 의존성) |
| server.tool() | 이름, 설명, inputSchema(zod), 핸들러 4개 파라미터 |
| 테스트 | MCP Inspector (`npx @modelcontextprotocol/inspector`) |
| 연결 | `claude mcp add` 로 Claude Code에 등록 |
| 주의 | STDIO 서버에서 console.log() 금지 (console.error() 사용) |

---

## 참고 리소스

- MCP 공식 사이트: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- 스펙 문서 (2025-11-25): [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-11-25)
- TypeScript SDK: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- SDK npm: [@modelcontextprotocol/sdk](https://www.npmjs.com/package/@modelcontextprotocol/sdk)
- SDK 문서: [ts.sdk.modelcontextprotocol.io](https://ts.sdk.modelcontextprotocol.io/)
- 공식 튜토리얼: [Build an MCP Server](https://modelcontextprotocol.io/docs/develop/build-server)
- MCP Inspector: `npx @modelcontextprotocol/inspector`
- 2026 로드맵: [blog.modelcontextprotocol.io](http://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/)
