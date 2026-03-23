---
title: "Day 12: MCP 서버 v1 — OpenWeather API 연동"
date: 2026-03-17
tags:
  - MCP
  - OpenWeather
  - API연동
  - week3
---

# Day 12: MCP 서버 v1 — OpenWeather API 연동

> AI Tools Mastery Curriculum — Week 3, Day 12
> 소요 시간: 50분 | 구현 중심

---

## ① OpenWeather API 분석

### API 키 발급

1. [openweathermap.org](https://openweathermap.org) 회원가입
2. 로그인 → My API Keys 페이지에서 API 키 확인
3. **One Call API 3.0** 구독 활성화 (무료 티어: 1,000 calls/day)

> **주의:** API 키 발급 후 활성화까지 최대 2시간 걸릴 수 있다.
> One Call 3.0 구독 시 신용카드 등록이 필요하지만, 무료 한도 내에서는 과금되지 않는다.
> Billing plans에서 "Calls per day" 제한을 1,000으로 설정해두면 안전하다.

### 사용할 엔드포인트

| 엔드포인트 | 용도 | 무료 |
|---|---|---|
| One Call 3.0 | 현재 날씨 + 48시간 예보 + 8일 예보 + 알림 | ✅ 1,000/일 |
| Geocoding | 도시명 → 위도/경도 변환 | ✅ |

### API 호출 예시

```bash
# 1. 도시명 → 좌표 변환 (Geocoding)
curl "http://api.openweathermap.org/geo/1.0/direct?q=Seoul&limit=1&appid=YOUR_API_KEY"
# → [{ "lat": 37.5665, "lon": 126.978, "name": "Seoul", ... }]

# 2. 좌표 → 날씨 조회 (One Call 3.0)
curl "https://api.openweathermap.org/data/3.0/onecall?lat=37.5665&lon=126.978&units=metric&lang=kr&exclude=minutely&appid=YOUR_API_KEY"
```

### 응답 구조 (핵심 필드)

```json
{
  "current": {
    "temp": 18.5,
    "feels_like": 17.2,
    "humidity": 65,
    "wind_speed": 3.6,
    "weather": [
      { "main": "Clouds", "description": "구름 조금" }
    ]
  },
  "daily": [
    {
      "dt": 1710403200,
      "temp": { "min": 8.2, "max": 19.5 },
      "weather": [{ "main": "Clear", "description": "맑음" }]
    }
  ],
  "alerts": [
    { "event": "Heavy Rain Warning", "description": "..." }
  ]
}
```

---

## ② MCP 서버 구현 (하드코딩 → 실제 API)

### Step 1: 프로젝트 생성

Day 11에서 만든 프로젝트를 확장하거나, 새로 만든다:

```bash
mkdir weather-mcp-server
cd weather-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript
mkdir src
```

package.json과 tsconfig.json은 Day 11과 동일하게 설정한다.

### Step 2: 하드코딩 버전 먼저 만들기

API 연동 전에 **가짜 데이터로 먼저 동작을 확인**한다. 실무에서도 외부 API 연동 시 이 패턴을 추천한다:

`src/index.ts` (하드코딩 버전):

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "weather-mcp-server",
  version: "1.0.0",
});

// Tool 1: 현재 날씨 조회 (하드코딩)
server.registerTool(
  "get_current_weather",
  {
    title: "Current Weather",
    description: "지정한 도시의 현재 날씨를 조회합니다. 날씨, 기온, 습도 등을 알고 싶을 때 사용합니다.",
    inputSchema: {
      city: z.string().describe("도시명 (예: Seoul, Tokyo, London)"),
    },
  },
  async ({ city }) => ({
    content: [
      {
        type: "text",
        text: JSON.stringify({
          city,
          temp: 18.5,
          feels_like: 17.2,
          humidity: 65,
          description: "구름 조금",
          wind_speed: 3.6,
        }, null, 2),
      },
    ],
  })
);

// Tool 2: 주간 예보 조회 (하드코딩)
server.registerTool(
  "get_forecast",
  {
    title: "Weather Forecast",
    description: "지정한 도시의 주간 날씨 예보를 조회합니다. 내일 날씨, 이번 주 날씨를 알고 싶을 때 사용합니다.",
    inputSchema: {
      city: z.string().describe("도시명 (예: Seoul, Tokyo, London)"),
      days: z.number().min(1).max(8).default(3).describe("예보 일수 (1-8, 기본값 3)"),
    },
  },
  async ({ city, days }) => ({
    content: [
      {
        type: "text",
        text: JSON.stringify({
          city,
          forecast: Array.from({ length: days }, (_, i) => ({
            date: new Date(Date.now() + (i + 1) * 86400000).toISOString().split("T")[0],
            temp_min: 8 + i,
            temp_max: 18 + i,
            description: "맑음",
          })),
        }, null, 2),
      },
    ],
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
console.error("Weather MCP Server started (hardcoded mode)");
```

빌드 & 테스트:

```bash
npm run build
npx @modelcontextprotocol/inspector node build/index.js
```

Inspector에서 `get_current_weather`와 `get_forecast`가 정상 동작하는지 확인한다.

### Step 3: 실제 API 연동

하드코딩이 잘 동작하면, 실제 OpenWeather API로 교체한다:

`src/index.ts` (실제 API 버전):

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "weather-mcp-server",
  version: "1.0.0",
});

// 환경변수에서 API 키 읽기
const API_KEY = process.env.OPENWEATHER_API_KEY;
if (!API_KEY) {
  console.error("ERROR: OPENWEATHER_API_KEY 환경변수가 설정되지 않았습니다.");
  process.exit(1);
}

const BASE_URL = "https://api.openweathermap.org";

// 도시명 → 좌표 변환
async function geocode(city: string): Promise<{ lat: number; lon: number; name: string }> {
  const url = `${BASE_URL}/geo/1.0/direct?q=${encodeURIComponent(city)}&limit=1&appid=${API_KEY}`;
  const res = await fetch(url);
  const data = await res.json();

  if (!Array.isArray(data) || data.length === 0) {
    throw new Error(`도시를 찾을 수 없습니다: ${city}`);
  }

  return { lat: data[0].lat, lon: data[0].lon, name: data[0].name };
}

// One Call API 3.0 호출
async function getWeather(lat: number, lon: number): Promise<any> {
  const url = `${BASE_URL}/data/3.0/onecall?lat=${lat}&lon=${lon}&units=metric&lang=kr&exclude=minutely&appid=${API_KEY}`;
  const res = await fetch(url);

  if (!res.ok) {
    throw new Error(`OpenWeather API 오류: ${res.status} ${res.statusText}`);
  }

  return res.json();
}

// Tool 1: 현재 날씨 조회
server.registerTool(
  "get_current_weather",
  {
    title: "Current Weather",
    description: "지정한 도시의 현재 날씨를 조회합니다. 날씨, 기온, 습도 등을 알고 싶을 때 사용합니다.",
    inputSchema: {
      city: z.string().describe("도시명 (예: Seoul, Tokyo, London)"),
    },
  },
  async ({ city }) => {
    try {
      const { lat, lon, name } = await geocode(city);
      const weather = await getWeather(lat, lon);
      const current = weather.current;

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              city: name,
              temp: current.temp,
              feels_like: current.feels_like,
              humidity: current.humidity,
              description: current.weather[0]?.description ?? "정보 없음",
              wind_speed: current.wind_speed,
              alerts: weather.alerts?.map((a: any) => a.event) ?? [],
            }, null, 2),
          },
        ],
      };
    } catch (error: any) {
      return {
        content: [{ type: "text", text: `오류: ${error.message}` }],
        isError: true,
      };
    }
  }
);

// Tool 2: 주간 예보 조회
server.registerTool(
  "get_forecast",
  {
    title: "Weather Forecast",
    description: "지정한 도시의 주간 날씨 예보를 조회합니다. 내일 날씨, 이번 주 날씨를 알고 싶을 때 사용합니다.",
    inputSchema: {
      city: z.string().describe("도시명 (예: Seoul, Tokyo, London)"),
      days: z.number().min(1).max(8).default(3).describe("예보 일수 (1-8, 기본값 3)"),
    },
  },
  async ({ city, days }) => {
    try {
      const { lat, lon, name } = await geocode(city);
      const weather = await getWeather(lat, lon);
      const daily = weather.daily?.slice(0, days) ?? [];

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              city: name,
              forecast: daily.map((d: any) => ({
                date: new Date(d.dt * 1000).toISOString().split("T")[0],
                temp_min: d.temp.min,
                temp_max: d.temp.max,
                description: d.weather[0]?.description ?? "정보 없음",
                humidity: d.humidity,
              })),
            }, null, 2),
          },
        ],
      };
    } catch (error: any) {
      return {
        content: [{ type: "text", text: `오류: ${error.message}` }],
        isError: true,
      };
    }
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
console.error("Weather MCP Server started");
```

### 핵심 구현 패턴 정리

```
1. 환경변수로 API 키 관리
   → process.env.OPENWEATHER_API_KEY
   → 코드에 키를 하드코딩하지 않는다

2. 도우미 함수 분리
   → geocode(): 도시명 → 좌표
   → getWeather(): 좌표 → 날씨 데이터
   → Tool 핸들러는 이 함수들을 조합만 한다

3. 에러 처리
   → try/catch로 감싸고
   → isError: true 반환하면 Claude가 에러를 인식

4. 응답은 항상 JSON.stringify()
   → Claude가 구조화된 데이터를 잘 해석한다
```

---

## ③ Claude Code에서 연동 테스트

### 빌드

```bash
npm run build
```

### MCP Inspector로 먼저 테스트

```bash
# 환경변수와 함께 Inspector 실행
OPENWEATHER_API_KEY=your_key_here npx @modelcontextprotocol/inspector node build/index.js
```

Inspector에서 확인:
1. `get_current_weather` → city: "Seoul" → Run tool → 실제 날씨 데이터 확인
2. `get_forecast` → city: "Seoul", days: 3 → Run tool → 3일 예보 확인

### Claude Code에 등록

```bash
# 환경변수와 함께 MCP 서버 등록
claude mcp add weather-mcp-server \
  -e OPENWEATHER_API_KEY=your_key_here \
  -- node /절대경로/weather-mcp-server/build/index.js
```

### Claude Code에서 테스트

```bash
claude

> 서울 날씨 어때?
# Claude가 get_current_weather 도구를 호출
# "서울의 현재 기온은 18.5°C, 구름 조금, 습도 65%입니다."

> 도쿄 이번 주 날씨 예보 알려줘
# Claude가 get_forecast 도구를 호출
# 7일간의 날씨 예보를 자연어로 정리해줌

> 런던이랑 파리 중에 이번 주 어디가 더 따뜻해?
# Claude가 get_forecast를 2번 호출 (런던 + 파리)
# 비교 분석 결과를 정리해줌
```

### 등록 확인 & 관리

```bash
# 등록된 MCP 서버 목록
claude mcp list

# 서버 제거
claude mcp remove weather-mcp-server
```

---

## 실습 과제

### 과제 1: 하드코딩 버전 구현 (15분)
1. 프로젝트 생성 & Step 2 코드 작성
2. 빌드 & Inspector로 두 가지 도구 테스트
3. 정상 동작 확인

### 과제 2: 실제 API 연동 (20분)
1. OpenWeather API 키 발급
2. Step 3 코드로 교체
3. Inspector에서 실제 날씨 데이터 확인
4. 에러 케이스 테스트 (존재하지 않는 도시명 등)

### 과제 3: Claude Code 연동 (15분)
1. `claude mcp add`로 서버 등록
2. 자연어로 날씨 질문 → 도구 자동 호출 확인
3. 두 도시 비교 질문으로 다중 호출 확인

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| 구현 순서 | 하드코딩 먼저 → Inspector 테스트 → 실제 API 교체 |
| API 키 관리 | process.env로 환경변수 사용 (코드에 하드코딩 금지) |
| Geocoding | 도시명 → 좌표 변환이 필요 (OpenWeather는 좌표 기반) |
| 에러 처리 | try/catch + isError: true 반환 |
| 응답 형식 | JSON.stringify()로 구조화된 데이터 반환 |
| 등록 | `claude mcp add -e KEY=value -- node path/to/index.js` |
| 테스트 | Inspector 먼저, Claude Code 나중 |

---

## 참고 리소스

- OpenWeather API 키 발급: [openweathermap.org/appid](https://openweathermap.org/appid)
- One Call API 3.0 문서: [openweathermap.org/api/one-call-3](https://openweathermap.org/api/one-call-3)
- Geocoding API 문서: [openweathermap.org/api/geocoding-api](https://openweathermap.org/api/geocoding-api)
- MCP Inspector: `npx @modelcontextprotocol/inspector`
- MCP 공식 서버 구축 튜토리얼: [modelcontextprotocol.io/docs/develop/build-server](https://modelcontextprotocol.io/docs/develop/build-server)
