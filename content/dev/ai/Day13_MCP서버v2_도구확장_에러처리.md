---
title: "Day 13: MCP 서버 v2 — 도구 확장 & 에러 처리"
date: 2026-03-17
tags:
  - MCP
  - 에러처리
  - week3
---

# Day 13: MCP 서버 v2 — 도구 확장 & 에러 처리

> AI Tools Mastery Curriculum — Week 3, Day 13
> 소요 시간: 50분 | 구현 중심

---

## ① 도구 추가: 도시 비교 & 옷차림 추천

### 현재 상태 (Day 12)

```
weather-mcp-server v1
├── get_current_weather  → 현재 날씨 조회
└── get_forecast         → 주간 예보 조회
```

### 확장 목표 (Day 13)

```
weather-mcp-server v2
├── get_current_weather  → 현재 날씨 조회 (기존)
├── get_forecast         → 주간 예보 조회 (기존)
├── compare_weather      → 두 도시 날씨 비교 (신규)
└── get_clothing_advice  → 날씨 기반 옷차림 추천 (신규)
```

### Tool 3: 두 도시 날씨 비교

Claude에게 "런던이랑 파리 중 어디가 더 따뜻해?"라고 물으면 `get_current_weather`를 2번 호출할 수도 있지만, **비교 전용 도구**를 제공하면 한 번의 호출로 구조화된 비교 데이터를 줄 수 있다.

```typescript
server.registerTool(
  "compare_weather",
  {
    title: "Compare Weather",
    description: "두 도시의 현재 날씨를 비교합니다. 어디가 더 덥다/춥다/습하다 등을 물어볼 때 사용합니다.",
    inputSchema: {
      city1: z.string().describe("첫 번째 도시명"),
      city2: z.string().describe("두 번째 도시명"),
    },
  },
  async ({ city1, city2 }) => {
    try {
      // 두 도시를 병렬로 조회
      const [geo1, geo2] = await Promise.all([
        geocode(city1),
        geocode(city2),
      ]);
      const [weather1, weather2] = await Promise.all([
        getWeather(geo1.lat, geo1.lon),
        getWeather(geo2.lat, geo2.lon),
      ]);

      const c1 = weather1.current;
      const c2 = weather2.current;

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              city1: {
                name: geo1.name,
                temp: c1.temp,
                feels_like: c1.feels_like,
                humidity: c1.humidity,
                description: c1.weather[0]?.description ?? "정보 없음",
              },
              city2: {
                name: geo2.name,
                temp: c2.temp,
                feels_like: c2.feels_like,
                humidity: c2.humidity,
                description: c2.weather[0]?.description ?? "정보 없음",
              },
              comparison: {
                temp_diff: Math.round((c1.temp - c2.temp) * 10) / 10,
                warmer: c1.temp > c2.temp ? geo1.name : geo2.name,
                more_humid: c1.humidity > c2.humidity ? geo1.name : geo2.name,
              },
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
```

**핵심 패턴: `Promise.all()`로 병렬 호출**

두 도시의 API를 순차로 호출하면 응답 시간이 2배가 된다. `Promise.all()`로 병렬 호출하면 빠른 쪽에 맞춰진다. Java의 `CompletableFuture.allOf()`와 같은 개념이다.

### Tool 4: 날씨 기반 옷차림 추천

이 도구는 **데이터 가공만 하고, 추천 로직은 간단한 규칙 기반**으로 구현한다. 복잡한 판단은 Claude에게 맡기면 된다.

```typescript
server.registerTool(
  "get_clothing_advice",
  {
    title: "Clothing Advice",
    description: "현재 날씨를 기반으로 옷차림 정보를 제공합니다. 오늘 뭐 입을지, 우산이 필요한지 물어볼 때 사용합니다.",
    inputSchema: {
      city: z.string().describe("도시명"),
    },
  },
  async ({ city }) => {
    try {
      const { lat, lon, name } = await geocode(city);
      const weather = await getWeather(lat, lon);
      const current = weather.current;

      // 간단한 규칙 기반 데이터 제공
      const conditions = {
        city: name,
        temp: current.temp,
        feels_like: current.feels_like,
        weather_main: current.weather[0]?.main ?? "Unknown",
        description: current.weather[0]?.description ?? "정보 없음",
        wind_speed: current.wind_speed,
        humidity: current.humidity,
        rain_expected: weather.hourly?.slice(0, 12).some(
          (h: any) => h.weather[0]?.main === "Rain"
        ) ?? false,
        temp_category:
          current.feels_like < 5 ? "매우 추움" :
          current.feels_like < 12 ? "추움" :
          current.feels_like < 20 ? "선선함" :
          current.feels_like < 28 ? "따뜻함" : "더움",
      };

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(conditions, null, 2),
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
```

**설계 포인트:** 옷차림 "추천 문장"을 MCP 서버가 만들지 않는다. `temp_category`, `rain_expected` 같은 **판단 재료만 제공**하고, "패딩을 입으세요" 같은 자연어 답변은 Claude가 만든다. Day 11에서 배운 원칙 그대로다: MCP는 데이터, 해석은 LLM.

---

## ② 에러 핸들링 & 로깅 구현

### 현재 문제점

Day 12의 에러 처리는 최소한이다. 실무 수준으로 강화하자:

```
현재: catch (error) → "오류: {message}" 반환 (끝)
목표: 에러 유형별 처리 + 로깅 + 재시도
```

### 에러 유형별 처리

`src/errors.ts` 파일을 새로 만든다:

```typescript
// src/errors.ts

export class WeatherApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public endpoint: string
  ) {
    super(message);
    this.name = "WeatherApiError";
  }
}

export class CityNotFoundError extends Error {
  constructor(public city: string) {
    super(`도시를 찾을 수 없습니다: ${city}`);
    this.name = "CityNotFoundError";
  }
}

export class ApiKeyError extends Error {
  constructor() {
    super("API 키가 유효하지 않습니다. OPENWEATHER_API_KEY를 확인해주세요.");
    this.name = "ApiKeyError";
  }
}
```

### getWeather 함수 개선

```typescript
async function getWeather(lat: number, lon: number): Promise<any> {
  const url = `${BASE_URL}/data/3.0/onecall?lat=${lat}&lon=${lon}&units=metric&lang=kr&exclude=minutely&appid=${API_KEY}`;

  const res = await fetch(url);

  if (res.status === 401) {
    throw new ApiKeyError();
  }

  if (!res.ok) {
    throw new WeatherApiError(
      `API 요청 실패: ${res.statusText}`,
      res.status,
      "onecall"
    );
  }

  return res.json();
}
```

### geocode 함수 개선

```typescript
async function geocode(city: string): Promise<{ lat: number; lon: number; name: string }> {
  const url = `${BASE_URL}/geo/1.0/direct?q=${encodeURIComponent(city)}&limit=1&appid=${API_KEY}`;

  const res = await fetch(url);

  if (res.status === 401) {
    throw new ApiKeyError();
  }

  if (!res.ok) {
    throw new WeatherApiError(
      `Geocoding 요청 실패: ${res.statusText}`,
      res.status,
      "geocoding"
    );
  }

  const data = await res.json();

  if (!Array.isArray(data) || data.length === 0) {
    throw new CityNotFoundError(city);
  }

  return { lat: data[0].lat, lon: data[0].lon, name: data[0].name };
}
```

### Tool 핸들러에서 에러 유형별 메시지

```typescript
async ({ city }) => {
  try {
    // ... 기존 로직
  } catch (error: any) {
    let message: string;

    if (error instanceof CityNotFoundError) {
      message = `"${error.city}"라는 도시를 찾을 수 없습니다. 영문 도시명으로 다시 시도해주세요.`;
    } else if (error instanceof ApiKeyError) {
      message = "API 인증 오류입니다. 서버 관리자에게 문의해주세요.";
    } else if (error instanceof WeatherApiError) {
      message = `날씨 서비스 오류 (${error.statusCode}): 잠시 후 다시 시도해주세요.`;
    } else {
      message = `예상치 못한 오류: ${error.message}`;
    }

    console.error(`[ERROR] ${error.name}: ${error.message}`);

    return {
      content: [{ type: "text", text: message }],
      isError: true,
    };
  }
}
```

### 로깅 유틸리티

`src/logger.ts`:

```typescript
// src/logger.ts

type LogLevel = "INFO" | "WARN" | "ERROR";

export function log(level: LogLevel, message: string, data?: Record<string, any>) {
  const timestamp = new Date().toISOString();
  const entry = { timestamp, level, message, ...data };

  // STDIO 서버이므로 반드시 stderr로 출력
  console.error(JSON.stringify(entry));
}
```

사용 예시:

```typescript
import { log } from "./logger.js";

// Tool 호출 시작
log("INFO", "get_current_weather 호출", { city });

// API 호출
log("INFO", "OpenWeather API 호출", { endpoint: "onecall", lat, lon });

// 에러 발생
log("ERROR", "API 호출 실패", { statusCode: 401, endpoint: "onecall" });
```

---

## ③ 응답 캐싱 & 최종 통합

### 간단한 인메모리 캐시

날씨 데이터는 10분 단위로 갱신되므로, 같은 좌표의 반복 호출을 캐싱하면 API 호출 횟수를 줄일 수 있다 (무료 한도 1,000/일):

`src/cache.ts`:

```typescript
// src/cache.ts

interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

export class SimpleCache<T> {
  private store = new Map<string, CacheEntry<T>>();

  constructor(private ttlMs: number = 10 * 60 * 1000) {} // 기본 10분

  get(key: string): T | null {
    const entry = this.store.get(key);
    if (!entry) return null;

    if (Date.now() - entry.timestamp > this.ttlMs) {
      this.store.delete(key);
      return null;
    }

    return entry.data;
  }

  set(key: string, data: T): void {
    this.store.set(key, { data, timestamp: Date.now() });
  }
}
```

getWeather에 캐시 적용:

```typescript
import { SimpleCache } from "./cache.js";

const weatherCache = new SimpleCache<any>(10 * 60 * 1000); // 10분 TTL

	async function getWeatherCached(lat: number, lon: number): Promise<any> {
	  const key = `${lat.toFixed(2)},${lon.toFixed(2)}`;
	  const cached = weatherCache.get(key);
	
	  if (cached) {
	    log("INFO", "캐시 히트", { key });
	    return cached;
	  }
	
	  log("INFO", "캐시 미스 → API 호출", { key });
	  const data = await getWeather(lat, lon);
	  weatherCache.set(key, data);
	  return data;
	}
```

그런 다음 모든 Tool 핸들러에서 `getWeather()` 대신 `getWeatherCached()`를 사용한다.

### 최종 파일 구조

```
weather-mcp-server/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts        ← McpServer + Tool 등록 + 서버 시작
│   ├── errors.ts       ← 커스텀 에러 클래스
│   ├── logger.ts       ← 로깅 유틸리티
│   └── cache.ts        ← 인메모리 캐시
└── build/              ← 빌드 결과 (tsc가 생성)
```

Day 12에서는 index.ts 하나에 모든 코드를 넣었지만, Day 13에서 **파일을 분리**했다. 이것은 실무에서 MCP 서버를 유지보수할 때 중요한 패턴이다:

```
index.ts   → 서버 설정 + Tool 등록 (what)
errors.ts  → 에러 정의 (how to fail)
logger.ts  → 로깅 (observability)
cache.ts   → 캐싱 (performance)
```

### import 연결

`src/index.ts` 상단:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { WeatherApiError, CityNotFoundError, ApiKeyError } from "./errors.js";
import { log } from "./logger.js";
import { SimpleCache } from "./cache.js";
```

> **참고:** TypeScript에서 로컬 파일을 import할 때 `.js` 확장자를 붙여야 한다.
> 소스는 `.ts`이지만, 컴파일 후 Node.js가 실행하는 건 `.js`이므로 import 경로도 `.js`로 쓴다.

---

## 실습 과제

### 과제 1: 도구 추가 (20분)
1. `compare_weather` 도구 추가
2. `get_clothing_advice` 도구 추가
3. Inspector에서 두 도구 테스트
4. Claude Code에서 "서울이랑 도쿄 비교해줘" 테스트

### 과제 2: 에러 처리 강화 (15분)
1. `errors.ts`, `logger.ts` 파일 생성
2. geocode, getWeather 함수에 에러 처리 적용
3. 존재하지 않는 도시명으로 테스트
4. stderr에 로그가 정상 출력되는지 확인

### 과제 3: 캐시 적용 (15분)
1. `cache.ts` 파일 생성
2. `getWeatherCached()` 함수 구현
3. 같은 도시를 2번 연속 조회 → 두 번째에 "캐시 히트" 로그 확인
4. API 호출 횟수가 줄어드는지 확인

---

## 오늘의 핵심 정리

| 포인트 | 설명 |
|---|---|
| 도구 추가 | 비교, 옷차림 등 **데이터 조합/가공** 도구는 유용하다 |
| Promise.all() | 병렬 API 호출로 응답 시간 단축 |
| 데이터만 제공 | 옷차림 "추천 문장"이 아니라 판단 재료를 제공, 해석은 Claude |
| 커스텀 에러 | CityNotFoundError 등 유형별로 분리하면 사용자 메시지 품질↑ |
| 로깅 | STDIO 서버에서는 반드시 console.error() (stderr) |
| 캐싱 | 10분 TTL 인메모리 캐시로 API 호출 횟수 절감 |
| 파일 분리 | index.ts(등록) + errors.ts + logger.ts + cache.ts |

---

## 참고 리소스

- Day 12 코드: weather-mcp-server v1 (이전 Day 가이드 참조)
- Promise.all() 문서: [MDN Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
- MCP 에러 처리 패턴: [modelcontextprotocol.io/docs/develop/build-server](https://modelcontextprotocol.io/docs/develop/build-server)
- OpenWeather API 무료 한도: 1,000 calls/day (One Call 3.0)
