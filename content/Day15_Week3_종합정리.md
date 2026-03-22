---
title: "Day 15: Week 3 종합 정리"
date: 2026-03-18
tags:
  - 회고
  - MCP
  - week3
---

# Day 15: Week 3 종합 정리

> AI Tools Mastery Curriculum — Week 3, Day 15
> 소요 시간: 30분 | 정리 중심

---

## Week 3 요약: 소비자에서 생산자로

Week 2까지는 AI 도구를 **사용하는** 입장이었다. Week 3에서는 MCP 서버를 **직접 만들고**, 멀티 에이전트로 **작업을 설계**하는 단계로 넘어왔다.

```
Week 1: Claude Code 심화 (프롬프트, Skills, Hooks)
Week 2: AI 도구 생태계 탐색 (Gemini, Codex, Cursor, NotebookLM)
Week 3: MCP 서버 제작 & 멀티 에이전트 ← 이번 주
```

---

## Day별 핵심 정리

### Day 11: MCP 프로토콜 구조 & SDK 학습

MCP의 아키텍처와 기본 개념을 이해하고, 최소 서버를 만들었다.

**핵심 개념:**

| 개념 | 설명 |
|---|---|
| MCP | LLM과 외부 도구/데이터를 연결하는 오픈 프로토콜 |
| Host / Client / Server | IDE가 Host, 내부에 Client, 우리가 만드는 건 Server |
| Tools / Resources / Prompts | 함수 호출 / 데이터 읽기 / 프롬프트 템플릿 |
| JSON-RPC 2.0 | MCP의 메시지 포맷 |
| stdio | 로컬 전송 방식 (stdin/stdout으로 통신) |

**기억할 것:**
- 실무에서 90% 이상은 **Tools**로 구현한다
- STDIO 서버에서 `console.log()` 금지 → `console.error()` 사용
- `server.registerTool()`이 현재 권장 API (`server.tool()`은 deprecated)
- MCP Inspector (`npx @modelcontextprotocol/inspector`)로 테스트

### Day 12: MCP 서버 v1 — OpenWeather API 연동

하드코딩부터 시작해서 실제 API를 연동하는 서버를 만들었다.

**구현한 도구:**

| 도구 | 기능 |
|---|---|
| `get_current_weather` | 도시의 현재 날씨 조회 |
| `get_forecast` | 도시의 주간 예보 조회 |

**기억할 것:**
- 구현 순서: **하드코딩 → Inspector 테스트 → 실제 API 교체 → Claude Code 연동**
- API 키는 `process.env`로 관리 (코드에 하드코딩 금지)
- Geocoding: 도시명 → 좌표 변환이 필요 (OpenWeather는 좌표 기반)
- 응답은 `JSON.stringify()`로 구조화된 데이터 반환, 해석은 Claude가 담당
- `claude mcp add -e KEY=value -- node path/to/index.js`로 등록

### Day 13: MCP 서버 v2 — 도구 확장 & 에러 처리

서버를 실무 수준으로 강화했다.

**추가한 도구:**

| 도구 | 기능 |
|---|---|
| `compare_weather` | 두 도시 병렬 비교 (Promise.all) |
| `get_clothing_advice` | 날씨 기반 옷차림 판단 재료 제공 |

**추가한 인프라:**

| 파일 | 역할 |
|---|---|
| `errors.ts` | 커스텀 에러 클래스 (CityNotFoundError, ApiKeyError 등) |
| `logger.ts` | stderr 기반 JSON 로깅 |
| `cache.ts` | 10분 TTL 인메모리 캐시 (API 호출 절감) |

**기억할 것:**
- MCP 서버는 **데이터만 제공**, 해석/추천은 Claude가 한다
- `Promise.all()`로 병렬 API 호출 → 응답 시간 단축
- 에러 유형별 커스텀 클래스 → 사용자 친화적 메시지
- 파일 분리: index.ts(등록) + errors.ts + logger.ts + cache.ts

### Day 14: 멀티 에이전트 오케스트레이션

여러 에이전트가 협력하는 패턴을 학습하고 실습했다.

**3가지 패턴:**

| 패턴 | 구조 | 적합한 상황 |
|---|---|---|
| Fan-out/Fan-in | 병렬 분배 → 종합 | 독립적인 작업 병렬 처리 |
| 파이프라인 | 순차 전달 | 작업 간 의존성 있을 때 |
| 토론/검증 | 병렬 + 비교 | 기술 선택, 아키텍처 결정 |

**기억할 것:**
- 멀티 에이전트의 핵심은 병렬이 아니라 **컨텍스트 분리**
- 서브 에이전트 간 직접 통신 불가 → **파일을 통해** 데이터 전달
- `.claude/agents/`에 커스텀 에이전트 정의 → `@이름`으로 호출
- 모든 작업에 멀티 에이전트를 쓸 필요 없음 → 독립 분리 가능할 때만

---

## Week 3에서 만든 것

```
weather-mcp-server/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts         ← 4개 도구 등록 + 서버 시작
│   ├── errors.ts        ← 커스텀 에러 클래스
│   ├── logger.ts        ← stderr JSON 로깅
│   └── cache.ts         ← 인메모리 캐시
└── build/               ← 빌드 결과

도구 목록:
├── get_current_weather  → 현재 날씨
├── get_forecast         → 주간 예보
├── compare_weather      → 두 도시 비교
└── get_clothing_advice  → 옷차림 판단 재료

커스텀 에이전트:
├── .claude/agents/code-reviewer.md
└── .claude/agents/test-writer.md
```

---

## 핵심 인사이트

### 1. MCP 서버는 "데이터 제공자"다

MCP 서버가 하는 일은 외부 API를 호출해서 데이터를 정리하고 JSON으로 반환하는 것이다. "서울은 따뜻하니 반팔을 입으세요"같은 해석은 Claude가 한다. 이 역할 분리를 이해하면 MCP 서버 설계가 명확해진다.

### 2. 하드코딩 먼저, API 나중

외부 API를 연동할 때 하드코딩 → 테스트 → 실제 API 순서로 진행하면 디버깅 시간이 크게 줄어든다. "MCP 프로토콜 문제인지, API 문제인지" 분리할 수 있기 때문이다.

### 3. 멀티 에이전트는 은탄환이 아니다

컨텍스트 분리와 병렬 처리가 필요할 때 강력하지만, 토큰 소모가 크고 같은 파일 동시 수정은 위험하다. "이 작업을 독립적으로 분리할 수 있는가?"가 적용 판단의 핵심 질문이다.

---

## 다음 주 예고: Week 4

Week 4에서는 **사이드 프로젝트 실전 개발**로 넘어간다. Week 1~3에서 배운 모든 기술을 총동원해서 기획부터 배포까지:

```
Day 16: 사이드 프로젝트 기획 & 설계
Day 17: 코어 기능 구현 — AI 페어 프로그래밍
Day 18: 기능 확장 & MCP 연동
Day 19: 테스트 & 배포
Day 20: 최종 회고 & 종합 정리
```

---

## 참고 리소스

- MCP 공식: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- TypeScript SDK: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- OpenWeather API: [openweathermap.org/api](https://openweathermap.org/api)
- Claude Code 서브 에이전트: [code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)
- Agent Teams (실험적): [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams)
