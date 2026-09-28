# Supermemory 전수조사 분석 정리 (한국어)

> 대상 저장소: **https://github.com/supermemoryai/supermemory**
> 포크 저장소: **https://github.com/bmshin94/supermemory**
> 작성일: 2026-09-28

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [쉬운 설명 (비유 버전)](#2-쉬운-설명-비유-버전)
3. [폴더 구조 전수조사](#3-폴더-구조-전수조사)
4. [설치 및 사용법](#4-설치-및-사용법)
5. [플러그인 vs 스킬 vs MCP](#5-플러그인-vs-스킬-vs-mcp)
6. [API 토큰 사용 여부](#6-api-토큰-사용-여부)
7. [GitHub에서 유명한 이유](#7-github에서-유명한-이유)
8. [로컬 에이전트 구축에 도움이 되는가](#8-로컬-에이전트-구축에-도움이-되는가)
9. [React / PHP 로 만들 수 있는가](#9-react--php-로-만들-수-있는가)
10. [수익화 아이디어](#10-수익화-아이디어)
11. [주의사항 및 체크리스트](#11-주의사항-및-체크리스트)
12. [참고 링크](#12-참고-링크)

---

## 1. 프로젝트 개요

**Supermemory = AI를 위한 기억(Memory) · 컨텍스트 엔진**

AI가 대화가 끝나면 모든 것을 잊어버리는 문제를 해결한다. 대화에서 사실(fact)을 자동 추출하고,
사용자 프로필을 만들고, 정보가 바뀌면 자동으로 갱신하며, 유효기간이 지난 정보는 스스로 잊는다.

### 핵심 기능 5가지

| 기능 | 설명 |
|---|---|
| Memory | 대화에서 사실만 추출. 시간 변화 / 모순 / 자동 망각 처리 |
| User Profiles | 자동 유지되는 사용자 컨텍스트. `static`(안정적 사실) + `dynamic`(최근 활동). 1회 호출 약 50ms |
| Hybrid Search | RAG + Memory 를 하나의 쿼리로 동시 검색 |
| Connectors | Google Drive · Gmail · Notion · OneDrive · GitHub 실시간 웹훅 동기화 |
| Multi-modal Extractors | PDF, 이미지(OCR), 영상(전사), 코드(AST 기반 청킹) |

### 저장소 실측 지표 (GitHub API 조회 결과)

| 항목 | 값 |
|---|---|
| Stars | **30,967** |
| Forks | **2,714** |
| Open Issues | 125 |
| 생성일 | 2024-02-27 |
| 주 언어 | TypeScript |
| 라이선스 | **MIT** |
| 저장소 크기 | 약 212 MB |

### 벤치마크

| 벤치마크 | 제작 | 결과 |
|---|---|---|
| LongMemEval | 학계 | #1 |
| LoCoMo | Snap Research | #1 |
| ConvoMem | Salesforce | #1 |

- LongMemEval 기준 **95% Recall@15**, 컨텍스트 **99.4% 감소** (약 720 토큰만 추가)
- 자체 개발 SMFS(Supermemory Filesystem): Claude 기준 토큰 3.0배 절감, Codex 기준 1.75배 절감

---

## 2. 쉬운 설명 (비유 버전)

### 단골 식당 비유

- **기억 없는 AI** = 매일 바뀌는 알바생. 매번 "저 고수 못 먹어요"를 다시 말해야 함.
- **Supermemory** = 사장님 노트. 한 번 말하면 적어두고, 다음에 알아서 반영.

### 이 노트가 똑똑한 이유 3가지

1. **중요한 것만 적음** — "오늘 날씨 좋네요"는 안 적고, "고수 못 먹어요"는 적음
2. **바뀌면 덮어씀** — "서울 거주" → "부산으로 이사" 하면 자동 갱신 (RAG는 둘 다 반환해서 혼란)
3. **유통기한 관리** — "내일 시험 있음"은 시험 후 자동 삭제 (automatic forgetting)

### 컨테이너 태그 = 노트 여러 권

```js
containerTag: "user_민수"     // 이 문자열만 바꾸면 기억 저장소가 분리됨
```

`user_민수`, `user_영희`, `project_회사`, `project_개인` … 서로 절대 섞이지 않는다.
필드 하나로 멀티테넌시가 완성된다.

### RAG vs Memory

| | RAG (일반 검색) | Memory (Supermemory) |
|---|---|---|
| 비유 | 도서관 | 사장님 노트 |
| 저장 대상 | 문서 조각 | 사람에 대한 사실 |
| 결과 | 누가 물어도 동일 | 사람마다 다름 |
| 모순 처리 | 서울/부산 둘 다 반환 | 부산으로 자동 정리 |
| 시간 개념 | 없음 | 있음 (최신 우선) |

Supermemory는 `searchMode: "hybrid"` 로 이 둘을 동시에 굴린다.

### 가장 중요한 오해 주의

> 이 저장소는 **"Supermemory를 쓰는 도구들의 모음"**이지,
> **"Supermemory 엔진 그 자체"가 아니다.**

- 있는 것: SDK, MCP 서버, 문서, 시각화 컴포넌트, 플레이그라운드
- 없는 것: **실제 메모리 엔진(백엔드 API 서버)** → `api.supermemory.ai` 비공개
- 로컬 바이너리에는 엔진이 들어있지만 **컴파일된 바이너리**라 소스는 비공개

비유: 맥도날드가 "포장지 디자인 + 주문 앱 코드"는 공개했지만 "소스 레시피"는 공개하지 않은 것.

---

## 3. 폴더 구조 전수조사

Turborepo 기반 모노레포. 패키지 매니저는 **bun** (>= 1.2.17).

```
supermemory/
├── apps/
│   ├── mcp/                       ★ 핵심. MCP 서버 (Cloudflare Workers)
│   ├── docs/                        Mintlify 문서 사이트 (mdx 100개+)
│   ├── web/                       ! 사실상 리다이렉트 페이지 (파일 6개)
│   ├── memory-graph-playground/     메모리 그래프 데모
│   ├── sdk-playground/              SDK 테스트 (TS + Python)
│   └── raycast-extension/           macOS Raycast 확장
│
├── packages/
│   ├── tools/                     ★ @supermemory/tools v2.3.0 (핵심 SDK 래퍼)
│   ├── ai-sdk/                      Vercel AI SDK 연동
│   ├── memory-graph/                @supermemory/memory-graph v0.2.3 (Canvas 시각화)
│   ├── lib/                         공용 유틸 (auth, api, queries, similarity …)
│   ├── ui/ hooks/ validation/       공용 React 컴포넌트 · 훅 · Zod 스키마
│   ├── docs-test/                   문서 예제 검증
│   ├── agent-framework-python/      Microsoft Agent Framework 연동
│   ├── openai-sdk-python/           OpenAI SDK 연동
│   ├── cartesia-sdk-python/         음성 AI 연동
│   └── pipecat-sdk-python/          실시간 음성 대화 연동
│
├── .github/workflows/               CI 2개 + 패키지 자동 배포 등 총 11개
├── CLAUDE.md / CONTRIBUTING.md / LICENSE(MIT)
├── README.md / README.zh-CN.md      (영어 + 중국어)
└── package.json / turbo.json / biome.json
```

### 조사 중 확인한 주요 사실

#### (1) `apps/web/` 은 리다이렉트 페이지

`apps/web/app/page.tsx:8` — "We moved." 문구 표시 후 5초 뒤 `console.supermemory.ai` 로 이동.
TS/TSX 파일이 총 6개뿐이며, 실제 대시보드는 비공개 저장소에 있다.

#### (2) 백엔드 API 서버가 저장소에 없음

`apps/` 하위에 `api/` 디렉터리가 존재하지 않는다.
`packages/tools/src/shared/memory-client.ts:47` 확인 결과:

```ts
const response = await fetch(`${baseUrl}/v4/profile`, {
  headers: { Authorization: `Bearer ${apiKey}` },
})
```

전부 원격 API를 호출하는 **클라이언트 코드**다.

#### (3) `apps/mcp/` 가 이 저장소의 알짜

`apps/mcp/src/server/tools/index.ts:19` 의 `registerAllTools()` 가 등록하는 도구 16개:

| 분류 | 도구 |
|---|---|
| 모델이 보는 도구 (8) | `search_memory`, `get_profile`, `list_documents`, `get_document`, `list_memories`, `list_spaces`, `who_am_i`, `add_memory` |
| MCP App 런처 (4) | `select-space`, `memory-graph`, `guided-save`, `upload-file` |
| App 전용 숨김 도구 (4) | `set-active-tag`, `save-memory`, `prepare-file-upload`, `fetch-graph-data` |

런타임 설계 (`apps/mcp/README.md:6-13`):

- MCP SDK v2, **요청마다 새 `McpServer` 인스턴스 생성** (stateless)
- MCP `2026-07-28` 프로토콜 + 2025 클라이언트 호환
- **매 요청 OAuth 토큰 검증**
- 활성 Space 는 Cloudflare **Durable Object** 에 저장
- Space 상태 키 = `organizationId + userId`

Space 해석 우선순위:

1. 명시적 `containerTag` 인자
2. 계정의 저장된 활성 space
3. 기본값 `sm_project_default`

보안 설계 (`apps/mcp/README.md:141`):
> `SpaceState` 는 활성 space 의 container tag 만 저장한다.
> **베어러 토큰 · MCP 클라이언트 신원 · 프로토콜 메시지는 절대 저장하지 않는다.**

#### (4) Python 패키지가 4개

`agent-framework-python`(Microsoft), `openai-sdk-python`, `cartesia-sdk-python`, `pipecat-sdk-python`.
cartesia / pipecat 은 둘 다 **음성 AI** 계열 → 회사가 음성 에이전트 메모리 시장을 겨냥하고 있다는 신호.

#### (5) 로컬 vs Enterprise 기능 차이

`apps/docs/self-hosting/local-vs-enterprise.mdx:14` 기준:

| | Supermemory local | Enterprise |
|---|---|---|
| 메모리 엔진 | 임베디드 그래프 엔진 | 매니지드 그래프 엔진 |
| 모델 | BYOK (완전 오프라인 가능) | **자체 튜닝 모델** |
| 인증 | 자동 생성 API 키 1개 | 조직 단위 인증 · 접근 제어 |
| 팀 접근 | 1머신 1조직 | 멀티 멤버 · 역할 · 스코프 키 |
| 관측성 | **서버 로그뿐** | 대시보드 (사용량 · 수집 모니터링 · 요청 로그) |
| 커넥터 | **없음** | Drive / Notion / Gmail / OneDrive |
| 확장성 | 1머신 1프로세스 | 글로벌 분산 · 탄력적 확장 |

---

## 4. 설치 및 사용법

용도에 따라 4가지 경로가 있다.

### 경로 1 — 내 AI 도구에 기억 붙이기 (가장 쉬움, 약 3분)

Claude Code:

```bash
claude mcp add --transport http supermemory https://mcp.supermemory.ai/mcp
```

Cursor / Claude Desktop / Windsurf / VS Code — 설정 JSON:

```json
{
  "mcpServers": {
    "supermemory": {
      "url": "https://mcp.supermemory.ai/mcp"
    }
  }
}
```

설정 파일 위치:

| 클라이언트 | 경로 |
|---|---|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `~/.cursor/mcp.json` |
| VS Code | `.vscode/mcp.json` |

첫 사용 시 브라우저가 열리며 **OAuth 로그인**. API 키 직접 입력 불필요.
OAuth 서버는 `/.well-known/oauth-protected-resource/mcp` 로 자동 디스커버리된다.

사용은 자동이다. Cursor / Claude Code 에서는 `/context` 로 프로필을 즉시 주입할 수 있다.

전용 플러그인 (MCP보다 통합이 깊음, 모두 별도 저장소):

- Claude Code: https://github.com/supermemoryai/claude-supermemory
- Muse Code: https://github.com/supermemoryai/muse-supermemory
- Cursor: https://github.com/supermemoryai/cursor-supermemory
- Codex: https://github.com/supermemoryai/codex-supermemory
- OpenClaw: https://github.com/supermemoryai/openclaw-supermemory
- OpenCode: https://github.com/supermemoryai/opencode-supermemory
- Hermes agent (Nous Research): https://github.com/NousResearch/hermes-agent

### 경로 2 — 내 앱/서비스에 기억 붙이기 (개발자용)

```bash
npm install supermemory    # 또는
pip install supermemory
```

```typescript
import Supermemory from "supermemory";

const client = new Supermemory({ apiKey: process.env.SUPERMEMORY_API_KEY });

// 저장
await client.add({
  content: "유저가 TypeScript와 함수형 패턴을 선호함",
  containerTag: "user_123",
});

// 프로필 + 검색 한 번에 (약 50ms)
const { profile, searchResults } = await client.profile({
  containerTag: "user_123",
  q: "이 유저는 어떤 코딩 스타일을 좋아해?",
});
// profile.static  → ["TypeScript 선호", "함수형 패턴 선호"]
// profile.dynamic → ["API 연동 작업 중"]
```

프레임워크 래퍼:

```typescript
// Vercel AI SDK
import { withSupermemory } from "@supermemory/tools/ai-sdk";
const model = withSupermemory(openai("gpt-4o"), {
  containerTag: "user_123",
  customId: "conv-1",
});

// Mastra
import { withSupermemory } from "@supermemory/tools/mastra";
const agent = new Agent(withSupermemory(config, "user-123", { mode: "full" }));
```

지원 프레임워크: Vercel AI SDK · LangChain · LangGraph · OpenAI Agents SDK · Mastra ·
Agno · Voltagent · Claude Memory Tool · n8n · Microsoft Agent Framework · Pipecat · Cartesia

주요 API:

| 메서드 | 용도 |
|---|---|
| `client.add()` | 콘텐츠 저장 (텍스트, 대화, URL, HTML) |
| `client.profile()` | 프로필 + 선택적 검색을 한 번에 |
| `client.search()` | 메모리/문서 하이브리드 검색 (`searchMode`) |
| `client.search.documents()` | 메타데이터 필터 문서 검색 (legacy v3) |
| `client.documents.uploadFile()` | PDF, 이미지, 영상, 코드 업로드 |
| `client.documents.list()` | 문서 목록 및 필터 |
| `client.settings.update()` | 메모리 추출 · 청킹 설정 |

### 경로 3 — 내 컴퓨터에서 직접 실행 (로컬 셀프호스팅)

```bash
curl -fsSL https://supermemory.ai/install | bash
# 또는
npx supermemory local

supermemory-server        # → http://localhost:6767
```

첫 부팅 시 대화형 마법사가 모델 설정을 안내하고 API 키(`sm_...`)를 출력한다.

```typescript
const client = new Supermemory({
  apiKey: "sm_...",
  baseURL: "http://localhost:6767",   // 이 한 줄만 변경
});
```

완전 오프라인 구성:

- LLM: **Ollama** (`gpt-oss:20b` 권장)
- 임베딩: `Xenova/bge-base-en-v1.5` (기본값, API 키 불필요)
- 데이터: `./.supermemory` 디렉터리 하나에 전부

### 경로 4 — 이 저장소 자체를 개발 (기여자용)

```bash
git clone https://github.com/supermemoryai/supermemory.git
cd supermemory
bun install                   # npm 아님. bun >= 1.2.17
cp .env.example .env.local
bun run dev:local             # web:3000, mcp:8788, docs:3003, graph:3004
```

주의사항 (`CONTRIBUTING.md:45-47`):

- `bun run dev` (`:local` 없이)는 **내부 팀 전용** — portless 로 `*.dev.supermemory.ai` HTTPS 제공,
  `bun run setup:dev` 로 443 포트 바인딩 + 로컬 CA 신뢰가 필요하다. OSS 기여자는 불필요.
- 로컬 개발 시 **Google/GitHub OAuth 로그인은 localhost 로 되돌아오지 않는다.**
  매직링크 / 이메일 OTP 로 로그인할 것.

품질 검사 명령:

```bash
bun run format-lint     # Biome
bun run check-types     # TypeScript
bun run build           # Turbo
```

### 에이전트용 추가 도구

```bash
npx supermemory setup            # 프로젝트 감지 후 통합 플로우
npx supermemory add "..." --tag user_123
npx supermemory search "..." --tag user_123
npx supermemory profile --tag user_123
```

---

## 5. 플러그인 vs 스킬 vs MCP

**정답: 전부 다다.** 본질은 **API 서비스**이고, 나머지는 그것을 감싼 표면(surface)이다.

```
                  Supermemory 엔진 (비공개, api.supermemory.ai)
                              ↑
        ┌──────────┬──────────┼──────────┬──────────┐
      SDK         MCP      플러그인     스킬        CLI
   (npm/pip)    (서버)     (6개)     (skills)    (npx)
```

| 형태 | 정체 | 위치 | 용도 |
|---|---|---|---|
| SDK | npm / pip 라이브러리 | `packages/tools/` | 내 앱에 코드로 통합 |
| MCP | Cloudflare Workers 서버 | `apps/mcp/` | AI 도구에 기억 부착 |
| 플러그인 | 각 AI 툴 전용 패키지 | 별도 저장소 6개 | 더 깊은 통합 |
| 스킬 | 에이전트 교육용 문서 | `supermemoryai/skills` | API 환각 방지 |
| CLI | `npx supermemory` | 비공개 | 터미널 셋업 / 스모크 테스트 |

### 반드시 구분해야 할 것: MCP가 2종류다

`apps/docs/agents-and-mcp.mdx:10` 에 명시되어 있다.

| MCP 종류 | 주소 | 역할 |
|---|---|---|
| **Memory MCP** | `https://mcp.supermemory.ai/mcp` | AI가 **나를 기억**하게 함 (일반 사용자용) |
| **Docs MCP** | `https://supermemory.ai/docs/mcp` | AI가 **Supermemory 문서를 검색** (개발자용) |

### 스킬 설치

```bash
npx skills add https://github.com/supermemoryai/skills --skill supermemory
```

스킬은 기억 기능이 아니라 "이 API는 이렇게 쓰는 것"을 에이전트에게 가르치는 문서 묶음이다.
공식 권장 조합: **스킬 + Docs MCP + `npx supermemory setup`**

---

## 6. API 토큰 사용 여부

| 사용 방식 | 토큰 필요? | 방법 |
|---|---|---|
| MCP (`mcp.supermemory.ai`) | **아니오** | OAuth 브라우저 로그인 |
| 플러그인 | 대부분 아니오 | 설치 시 OAuth |
| SDK (코드 통합) | **예** | `Authorization: Bearer sm_...` |
| 로컬 서버 | 자체 발급 | 첫 부팅 시 자동 생성, 외부 전송 없음 |

### API 키 발급

1. https://console.supermemory.ai 접속
2. 로그인 → API Keys
3. 키 생성 → `sm_...` 복사
4. `.env` 에 `SUPERMEMORY_API_KEY=sm_...`

```bash
curl https://api.supermemory.ai/v3/search \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  -d '{"q": "hello"}'
```

### Scoped Key (멀티테넌시 서비스 구축 시 필수)

`apps/docs/authentication.mdx:59` 기준. 마스터 키를 클라이언트에 노출하지 않기 위한 제한 키다.

```bash
curl https://api.supermemory.ai/v3/auth/scoped-key \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_MASTER_KEY' \
  -d '{
    "containerTag": "my-project",
    "name": "my-key-name",
    "expiresInDays": 30
  }'
```

- 허용 엔드포인트: `/v3/documents`, `/v3/memories`, `/v4/memories`, `/v3/search`, `/v4/search`, `/v4/profile`
- **차단**: 결제 조회, 조직 설정 관리, 추가 키 발급

### 커넥터 브랜딩

사용자가 외부 서비스를 연결할 때 기본적으로 "Log in to **Supermemory**" 가 뜬다.
자체 OAuth 자격증명을 넣으면 내 앱 이름으로 바꿀 수 있다 (Google Drive, Notion, OneDrive).

```typescript
await client.settings.update({
  googleDriveCustomKeyEnabled: true,
  googleDriveClientId: "your-client-id.apps.googleusercontent.com",
  googleDriveClientSecret: "your-client-secret",
});
```

---

## 7. GitHub에서 유명한 이유

2년 만에 30,967 스타 = 하루 평균 약 42개.

1. **타이밍** — 2024~2026 AI 판의 최대 미해결 문제가 "장기 기억"
2. **남의 벤치마크 3개 전부 1등** — LongMemEval(학계), LoCoMo(Snap), ConvoMem(Salesforce)
3. **자극적인 숫자** — 컨텍스트 99.4% 감소 = API 비용 약 1/166. 돈 이야기라 파급력이 큼
4. **낮은 진입장벽** — 설치 한 줄, 설정 JSON 3줄, 3분이면 동작
5. **MCP 붐 선점** — "MCP + 메모리" 대표 주자 포지션 확보
6. **광범위한 통합** — Claude Code, Cursor, Codex, OpenCode, Windsurf, VS Code, Raycast, n8n, LangChain, Mastra … 어디서 검색해도 노출
7. **유명 AI 랩 채택** — Nous Research 의 Hermes agent 가 메모리 프로바이더로 사용
8. **영리한 마케팅** — `git.new/memory` 단축 URL, MemoryBench 오픈소스 공개(경쟁사 Mem0/Zep 과 직접 비교 유도), 중국어 README 로 중화권 공략

### 냉정한 평가

- 스타 30,967 ≠ 실제 프로덕션 사용자 30,967명. 상당수는 북마크용
- 포크 2,714개 대부분은 읽기용 포크
- **엔진이 비공개**라 "오픈소스 3만 스타" 타이틀에는 일부 거품이 있다
- 종합: **제품력 70% + 마케팅 30%** 정도가 정확한 평가

---

## 8. 로컬 에이전트 구축에 도움이 되는가

**결론: 매우 도움이 된다. 단, 조건부다.**

### 도움이 되는 점

**(1) 완전 오프라인 스택 구성 가능**

```
LLM    : Ollama (gpt-oss:20b)      → 로컬
임베딩  : Xenova/bge-base-en-v1.5   → 로컬 (API 키 불필요)
메모리  : supermemory-server :6767   → 로컬
데이터  : ./.supermemory/            → 로컬
```

사내망, 의료/법률 데이터, 프라이버시 민감 프로젝트에 적합.

**(2) 가장 어려운 부분을 건너뛴다**

| 직접 구현 | Supermemory 사용 |
|---|---|
| 벡터DB 선택 · 튜닝 | 불필요 |
| 임베딩 모델 선택 · 배치 처리 | 불필요 |
| 청킹 전략 (크기 / 오버랩) | 불필요 |
| 중복 제거 | 불필요 |
| **시간순 모순 해결** (가장 어려움) | 불필요 |
| 만료 정보 삭제 | 불필요 |
| 예상 2~3개월 | 예상 1시간 |

**(3) 프로토타입 → 프로덕션 전환이 매끄럽다** — `baseURL` 한 줄만 변경

**(4) MCP 서버 구현 레퍼런스로 최고** — `apps/mcp/` 에서 배울 것:

- MCP SDK v2 stateless 패턴 (요청마다 새 `McpServer`)
- OAuth 리소스 서버 (`/.well-known/oauth-protected-resource/mcp`)
- **모델이 보는 툴 vs 앱 전용 숨김 툴** 분리 패턴
- Durable Object 상태 관리 (토큰은 저장하지 않음)
- 3단계 폴백: 명시적 인자 → 저장된 활성 space → 기본값

**(5) 컨텍스트 절감 효과** — 로컬 LLM 은 컨텍스트가 길어지면 속도가 급락하므로 체감 효과가 더 크다

### 주의할 점

| 항목 | 내용 |
|---|---|
| 로컬 = 1머신 1프로세스 | 커넥터 없음, 팀 권한 없음, 대시보드 없음 |
| 엔진이 블랙박스 | 바이너리라 내부 수정 · 디버깅 불가 |
| 추출 품질이 모델 의존 | 문서에 "Enterprise는 자체 튜닝 모델" 명시 → 로컬은 품질이 낮을 수 있음 |
| 벤치마크 1등은 클라우드 기준 | Ollama 조합의 성능은 보장되지 않음 |
| 벤더 락인 | `containerTag` 구조에 깊게 종속되면 이전 비용이 큼 |

### 권장 전략

```
1단계: 로컬 서버로 프로토타입 (무료, 빠름)
2단계: 메모리 접근을 자체 인터페이스로 한 겹 감싸기
       interface MemoryProvider { save(); recall(); }
       → Mem0 / Zep / 자체 구현으로 교체 가능하게
3단계: 품질 부족 시 클라우드 API 로 전환 (baseURL 한 줄)
```

---

## 9. React / PHP 로 만들 수 있는가

| 질문 | 답 |
|---|---|
| React 로 Supermemory 를 쓸 수 있나 | **가능** (보안 주의사항 있음) |
| PHP 로 Supermemory 를 쓸 수 있나 | **가능** (공식 SDK 없지만 REST API) |
| React/PHP 로 Supermemory 같은 걸 만들 수 있나 | 부분적으로 가능 (아래 참고) |

### React 에서 사용하기

**절대 하면 안 되는 것:**

```tsx
"use client";
const client = new Supermemory({ apiKey: "sm_xxxxx" });  // 브라우저에 키 노출
```

**올바른 방법 — 서버 경유 (BFF):**

```ts
// app/api/memory/route.ts  (Next.js 서버 사이드)
import Supermemory from "supermemory";

const client = new Supermemory({
  apiKey: process.env.SUPERMEMORY_API_KEY,      // 서버에만 존재
});

export async function POST(req: Request) {
  const session = await getSession(req);
  if (!session) return new Response("Unauthorized", { status: 401 });

  const { query } = await req.json();

  const { profile, searchResults } = await client.profile({
    containerTag: `user_${session.userId}`,     // 서버가 태그 결정
    q: query,
  });

  return Response.json({ profile, searchResults });
}
```

```tsx
// components/MemoryPanel.tsx  (클라이언트)
"use client";
import { useQuery } from "@tanstack/react-query";

export function MemoryPanel({ query }: { query: string }) {
  const { data, isLoading } = useQuery({
    queryKey: ["memory", query],
    queryFn: () =>
      fetch("/api/memory", {
        method: "POST",
        body: JSON.stringify({ query }),
      }).then((r) => r.json()),
  });

  if (isLoading) return <Spinner />;
  return (
    <ul>
      {data.profile.static.map((f: string) => (
        <li key={f}>{f}</li>
      ))}
    </ul>
  );
}
```

**핵심 원칙 2가지**

1. API 키는 반드시 서버에만 둔다
2. `containerTag` 는 **서버가 세션에서 결정**한다.
   클라이언트가 보낸 값을 그대로 쓰면 다른 사용자의 기억을 조회할 수 있다

보너스: `npm i @supermemory/memory-graph` 로 그래프 시각화 컴포넌트를 그대로 사용 가능.
Canvas 기반이며 물리 시뮬레이션 · 히트테스트 · 뷰포트 · 공간 인덱스가 구현되어 있고 테스트도 12개 있다.

### PHP 에서 사용하기

공식 PHP SDK 는 없지만 REST API 이므로 간단히 래핑할 수 있다.

```php
<?php
class SupermemoryClient {
    private $apiKey;
    private $baseUrl;

    public function __construct(string $apiKey, string $baseUrl = 'https://api.supermemory.ai') {
        $this->apiKey  = $apiKey;
        $this->baseUrl = $baseUrl;
    }

    private function request(string $path, array $body): array {
        $ch = curl_init($this->baseUrl . $path);
        curl_setopt_array($ch, [
            CURLOPT_POST           => true,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER     => [
                'Content-Type: application/json',
                'Authorization: Bearer ' . $this->apiKey,
            ],
            CURLOPT_POSTFIELDS     => json_encode($body),
            CURLOPT_TIMEOUT        => 15,
        ]);
        $res  = curl_exec($ch);
        $code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($code >= 400) {
            throw new RuntimeException("Supermemory error $code: $res");
        }
        return json_decode($res, true);
    }

    public function add(string $content, string $tag): array {
        return $this->request('/v3/documents', [
            'content'      => $content,
            'containerTag' => $tag,
        ]);
    }

    public function profile(string $tag, ?string $q = null): array {
        $body = ['containerTag' => $tag, 'include' => ['static', 'dynamic']];
        if ($q) { $body['q'] = $q; }
        return $this->request('/v4/profile', $body);
    }

    public function search(string $q, string $tag): array {
        return $this->request('/v3/search', ['q' => $q, 'containerTag' => $tag]);
    }
}
```

WordPress 플러그인, Laravel 패키지 제작이 충분히 가능하다.

### "직접 Supermemory 같은 것을 만들기"의 현실성

| 계층 | React/PHP 로 가능? | 현실성 |
|---|---|---|
| UI / 대시보드 | 가능 | 문제 없음 |
| API 서버 | 가능 (Laravel / Next API) | 문제 없음 |
| 저장소 | PostgreSQL + **pgvector** | 충분함 |
| 임베딩 | 외부 API 호출 | OpenAI / Cohere 로 해결 |
| 사실 추출 | LLM 프롬프트 | 가능하나 품질 튜닝이 어려움 |
| 모순 해결 · 자동 망각 | 매우 어려움 | 연구 난이도 |
| 그래프 엔진 | PHP 로는 성능 한계 | Rust / Go 영역 |

**결론**

- 개인 프로젝트 / 소규모 SaaS 수준: `PostgreSQL + pgvector + 임베딩 API + LLM 추출 프롬프트` 로 80% 커버 가능
- "Supermemory 를 성능으로 이기겠다": 비권장. 연구랩 규모로 2년 이상 투입된 결과물이다

**권장 방향: 감싸서 팔기**

```
Supermemory (엔진)
    ↓ 우리가 감쌈
React / PHP 로 만든 "특정 시장 전용 제품"
    ↓
그 시장에 판매
```

---

## 10. 수익화 아이디어

**대전제: MIT 라이선스 = 상업적 이용, 수정, 재배포, 판매 모두 자유** (저작권 고지 필요)

### 전략 A — 감싸서 팔기 (Wrapper)

Supermemory 는 범용이라 "누구나 쓸 수 있지만 아무에게도 딱 맞지는 않다".
특정 직군에 맞게 감싸면 그 직군은 비용을 지불한다.

| # | 아이디어 | 설명 | 가격 예시 | 난이도 |
|---|---|---|---|---|
| 1 | AI 상담 기록 비서 | 상담사가 내담자별 히스토리 관리. 상담 전 자동 브리핑 | 월 ₩49,000 / 상담사 | ★★ |
| 2 | 학원 · 과외 AI 조교 | 학생별 취약점 기억, 학부모 리포트 자동 생성 | 월 ₩30,000 / 강사 | ★★ |
| 3 | WordPress 기억형 챗봇 플러그인 | 방문자 문의 기억, 재방문 시 맥락 유지 (PHP) | Lite 무료 + Pro 연 $79 | ★★ |
| 4 | 개인 세컨드 브레인 SaaS | Gmail/Notion/Drive 연결 후 자연어 질의, 아침 브리핑 | 월 $12 | ★★★ |
| 5 | 영업 CRM 메모리 애드온 | Salesforce/HubSpot 옆의 기억 레이어, 미팅 전 브리핑 카드 | 월 $30~50 / 시트 | ★★★ |
| 6 | **한국어 특화 메모리 서비스** | 벤치마크가 전부 영어 기준. 한국어 존댓말/줄임말/신조어 튜닝 | API 재판매 · 온프레미스 구축 | ★★★ |

보충 설명:

- **#2 학원 조교** — 한국 사교육 시장 규모가 크고, 학부모 리포트 자동화가 킬러 기능
- **#3 WordPress 플러그인** — WordPress 가 전 세계 웹의 약 43%. PHP 로 바로 구현 가능.
  CodeCanyon 판매 또는 자체 구독
- **#4 세컨드 브레인** — Mem, Rewind 등 경쟁이 치열하므로 니치(예: 개발자 전용)로 좁혀야 함
- **#6 한국어 특화** — 현재 이 영역을 진지하게 하는 곳이 사실상 없음.
  로컬 서버를 쓰면 원가가 거의 0

### 전략 B — 곡괭이 팔기 (Picks & Shovels)

| # | 아이디어 | 설명 | 가격 예시 | 난이도 |
|---|---|---|---|---|
| 7 | **Supermemory 로컬 관리 대시보드** | 로컬 버전은 로그뿐. React 로 메모리 목록/검색/편집/사용량/그래프 제공 | 일회성 $79 or 월 $9 | ★★ |
| 8 | 메모리 마이그레이션 도구 | Mem0/Zep/ChatGPT 내보내기 → Supermemory 일괄 이관 GUI | 건당 $29 / 기업 컨설팅 $2,000+ | ★★ |
| 9 | 브라우저 확장 · 모바일 앱 | 드래그 → 우클릭 "기억하기", iOS 공유시트 저장 | 무료 + Pro $4.99/월 | ★★ |
| 10 | 팀용 공유 메모리 레이어 | 로컬(팀기능 없음)과 Enterprise(고가) 사이의 중간 제품 | 팀당 월 $99 (10인) | ★★★★ |

보충 설명:

- **#7 대시보드** — `local-vs-enterprise.mdx:20` 에서 로컬은 "서버 로그"뿐이라고 명시.
  명확한 빈칸이고, `@supermemory/memory-graph` 를 무료로 활용 가능. 엔진을 건드리지 않아도 됨
- **#8 마이그레이션** — 저장소에 `apps/docs/migration/mem0-migration-script.py` 스크립트 1개만 존재.
  GUI 도구로 만들면 상품성이 있음
- **#9 확장/앱** — `CONTRIBUTING.md` 에 browser-extension 언급이 있으나 실제 폴더는 없음.
  Raycast 확장만 존재하고 Alfred / Chrome / iOS 는 없음
- **#10 팀 레이어** — Enterprise 와 정면 충돌하므로 니치(예: 개발팀 전용)로 포지셔닝 필요

### 전략 C — 지식 팔기

| # | 아이디어 | 설명 | 가격 예시 | 난이도 |
|---|---|---|---|---|
| 11 | **AI 메모리 구축 강의** | "3만 스타 오픈소스로 나만의 AI 비서 만들기". 한국어 자료가 거의 없음 | ₩99,000 (인프런 기준) | ★ |
| 12 | 기업 도입 컨설팅 | 사내 문서 온프레미스 구축 + 커넥터 연동 + 교육 | 프로젝트당 ₩500만~3,000만 | ★★★ |

### 종합 비교

| # | 아이디어 | 난이도 | 수익 규모 | 실현 속도 | 추천도 |
|---|---|---|---|---|---|
| 11 | 강의 | ★ | 중 | 매우 빠름 | ★★★★★ |
| 3 | WP 플러그인 | ★★ | 중 | 매우 빠름 | ★★★★★ |
| 7 | 로컬 대시보드 | ★★ | 중 | 매우 빠름 | ★★★★★ |
| 2 | 학원 조교 | ★★ | 상 | 빠름 | ★★★★ |
| 6 | 한국어 특화 | ★★★ | 상 | 느림 | ★★★★ |
| 12 | 컨설팅 | ★★★ | 최상 | 느림 | ★★★★ |
| 8 | 마이그레이션 | ★★ | 하 | 매우 빠름 | ★★★ |
| 5 | CRM 애드온 | ★★★ | 최상 | 느림 | ★★★ |
| 1 | 상담 비서 | ★★ | 상 | 빠름 | ★★★ |
| 9 | 확장 / 앱 | ★★ | 하 | 빠름 | ★★★ |
| 4 | 세컨드 브레인 | ★★★ | 중 | 느림 | ★★ |
| 10 | 팀 레이어 | ★★★★ | 상 | 느림 | ★★ |

### 권장 3개월 플랜

```
1개월차 — 빠른 검증
  #7 로컬 대시보드 (React) 또는 #3 WordPress 플러그인 (PHP)
  엔진을 건드리지 않아도 되고, 기존 기술 스택과 맞음
  Product Hunt / GitHub 공개 후 반응 측정

2개월차 — 현금 흐름
  #11 강의 제작. 1개월차 결과물을 교재로 재활용
  한국어 자료 공백을 선점

3개월차 — 본게임
  1~2개월차에서 확인한 실제 수요를 바탕으로
  #2(학원) / #6(한국어 특화) / #12(컨설팅) 중 선택
```

---

## 11. 주의사항 및 체크리스트

### 저장소 이해 관련

| 항목 | 내용 |
|---|---|
| 엔진 비공개 | 이 저장소는 클라이언트 · 플러그인 · 문서 모음. 메모리 엔진 본체는 비공개 |
| 호스티드 API 유료 | 무료 티어 초과 시 과금. 데이터가 외부 서버에 저장됨 |
| 벤더 락인 | `containerTag` 구조에 깊게 종속되면 이전 비용이 큼 |
| 로컬 제약 | 커넥터 없음, 팀 기능 없음, 대시보드 없음, 1머신 1프로세스 |
| 벤치마크 조건 | 1등 기록은 클라우드 + 자체 튜닝 모델 기준. 로컬 성능은 별개 |

### 수익화 전 체크리스트

| 항목 | 내용 |
|---|---|
| 라이선스 | MIT 저작권 고지 포함. 재배포 시 LICENSE 파일 동봉 |
| 상표권 | "Supermemory" 는 브랜드. "Supermemory 기반 ○○" 는 가능, 제품명 직접 사용은 불가 |
| 원가 | 호스티드 API 사용 시 마진 계산 필수. 로컬 서버는 원가가 거의 0 |
| 개인정보 | 상담 · 의료 · 교육 분야는 개인정보보호법 확인. 로컬/온프레미스 강력 권장 |
| 락인 대비 | `MemoryProvider` 인터페이스로 한 겹 추상화 |
| 정책 리스크 | 본사의 무료 티어 · 가격 정책 변경 가능성에 대한 Plan B |

### 보안 체크리스트 (구현 시)

| 항목 | 내용 |
|---|---|
| API 키 | 절대 클라이언트 번들에 포함 금지. 서버 환경변수로만 |
| containerTag | 클라이언트 입력값을 그대로 신뢰 금지. 서버 세션에서 결정 |
| Scoped Key | 외부에 키를 노출해야 한다면 컨테이너 제한 + 만료일 설정 키 사용 |
| 인증 | BFF 엔드포인트에 반드시 세션 검증 추가 |

---

## 12. 참고 링크

### 저장소

| 이름 | 주소 |
|---|---|
| **본 저장소 (원본)** | https://github.com/supermemoryai/supermemory |
| **포크 저장소** | https://github.com/bmshin94/supermemory |
| 조직 | https://github.com/supermemoryai |
| 단축 URL | https://git.new/memory |

### 공식 사이트

| 이름 | 주소 |
|---|---|
| 문서 | https://supermemory.ai/docs |
| 퀵스타트 | https://supermemory.ai/docs/quickstart |
| 셀프호스팅 | https://supermemory.ai/docs/self-hosting/overview |
| 대시보드 | https://console.supermemory.ai |
| 리서치 | https://supermemory.ai/research |
| MemoryBench | https://supermemory.ai/docs/memorybench/overview |
| Discord | https://supermemory.link/discord |
| X (Twitter) | https://twitter.com/supermemory |

### 패키지

| 이름 | 주소 |
|---|---|
| npm | https://www.npmjs.com/package/supermemory |
| PyPI | https://pypi.org/project/supermemory/ |

### 엔드포인트

| 이름 | 주소 |
|---|---|
| Memory MCP | `https://mcp.supermemory.ai/mcp` |
| Docs MCP | `https://supermemory.ai/docs/mcp` |
| API | `https://api.supermemory.ai` |
| 로컬 서버 | `http://localhost:6767` |

### 공식 플러그인 (별도 저장소)

| 대상 | 주소 |
|---|---|
| Claude Code | https://github.com/supermemoryai/claude-supermemory |
| Muse Code | https://github.com/supermemoryai/muse-supermemory |
| Cursor | https://github.com/supermemoryai/cursor-supermemory |
| Codex | https://github.com/supermemoryai/codex-supermemory |
| OpenClaw | https://github.com/supermemoryai/openclaw-supermemory |
| OpenCode | https://github.com/supermemoryai/opencode-supermemory |
| Skills | https://github.com/supermemoryai/skills |
| Hermes agent (Nous Research) | https://github.com/NousResearch/hermes-agent |

### 벤치마크

| 이름 | 주소 |
|---|---|
| LongMemEval | https://github.com/xiaowu0162/LongMemEval |
| LoCoMo | https://github.com/snap-research/locomo |
| ConvoMem | https://github.com/Salesforce/ConvoMem |

---

*이 문서는 저장소 전수조사 및 GitHub API 실측 데이터를 기반으로 작성되었습니다.*
