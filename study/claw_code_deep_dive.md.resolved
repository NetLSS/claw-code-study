# 🐾 Claw Code 프로젝트 딥다이브

> Claude Code(Anthropic의 AI 코딩 에이전트)의 핵심 구조를 분석하고, Python과 Rust로 클린룸 리라이트하는 오픈소스 프로젝트

---

## 목차

1. [프로젝트 배경](#1-프로젝트-배경)
2. [이 프로젝트는 무엇에 사용되나?](#2-이-프로젝트는-무엇에-사용되나)
3. [프로젝트 구조](#3-프로젝트-구조)
4. [핵심 개념: 에이전트 하네스](#4-핵심-개념-에이전트-하네스)
5. [도구 호출 구조 (read_file 예시)](#5-도구-호출-구조-read_file-예시)
6. [API 통신에서 실제로 오가는 데이터](#6-api-통신에서-실제로-오가는-데이터)
7. [원본과의 기능 비교 현황](#7-원본과의-기능-비교-현황)
8. [실행 방법](#8-실행-방법)
9. [초보 개발자를 위한 학습 포인트](#9-초보-개발자를-위한-학습-포인트)

---

## 1. 프로젝트 배경

2026년 3월 31일, Anthropic의 **Claude Code** 소스코드가 유출되는 사건이 발생했습니다. 이 프로젝트의 창시자(Sigrid Jin / @instructkr)는 유출된 코드를 그대로 보관하는 대신, **원본의 아키텍처(설계 구조)를 분석한 뒤 Python과 Rust로 클린룸 리라이트**를 진행했습니다.

> [!NOTE]
> **클린룸 리라이트(Clean-room Rewrite)** 란?
> 원본 코드를 복사하지 않고, 구조와 동작 방식만 이해한 뒤 완전히 새로 작성하는 방식입니다. 법적·윤리적 문제를 피하기 위해 사용됩니다.

### 주목할 만한 사실

- **역대 최단 시간 5만 스타 달성** — 공개 후 단 2시간 만에 GitHub 50,000 ⭐ 돌파
- **oh-my-codex(OmX)로 제작** — AI 워크플로우 도구를 활용해 AI가 AI 도구를 리라이트하는 메타적인 프로젝트
- **월스트리트저널에서 소개** — 프로젝트 창시자가 Claude Code 파워 유저로 WSJ에 보도됨
- **한국 개발자 커뮤니티(instructkr)** 가 주도하는 프로젝트

---

## 2. 이 프로젝트는 무엇에 사용되나?

**한 마디로: "터미널에서 AI와 대화하면서 코딩 작업을 시키는 도구"** 입니다.

LLM API(예: Anthropic Claude)에 연결하면, 대화 기반으로 **코드 수정, 파일 생성/삭제, 명령어 실행** 등의 작업을 AI가 직접 수행합니다.

### 일반 AI 채팅 vs 에이전트(Claw Code) 비교

| | 웹 채팅 (Claude.ai) | 에이전트 (Claw Code) |
|---|---|---|
| **파일 접근** | ❌ 내 컴퓨터 접근 불가 | ✅ 직접 파일 읽기/쓰기 |
| **명령어 실행** | ❌ 불가 | ✅ 터미널 명령어 실행 |
| **코드 수정** | 코드를 복붙해줌 | ✅ 직접 파일을 수정함 |
| **검색** | ❌ 프로젝트 내 검색 불가 | ✅ grep, glob 등 검색 |
| **연속 작업** | 한 번에 하나씩 | ✅ 여러 도구를 연쇄 호출 |

### 동작 예시

```
사용자: "이 함수에 에러 핸들링 추가해줘"
    ↓
[Claw Code CLI] ← 이 프로젝트가 만드는 것
    ↓
[Anthropic API (Claude)] ← LLM에게 질문 전달
    ↓
AI: "파일을 먼저 읽어볼게요" → 🔧 FileRead 도구 호출
AI: "수정하겠습니다" → 🔧 FileWrite 도구 호출
AI: "테스트 돌려볼게요" → 🔧 Bash 도구 호출
    ↓
💬 "에러 핸들링 추가 완료했습니다!"
```

### 왜 가치가 있나?

1. **학습 목적** — AI 에이전트 시스템이 내부적으로 어떻게 동작하는지 이해
2. **커스터마이징** — 원하는 대로 도구를 추가하거나 동작을 수정 가능
3. **다른 LLM 연결** — Anthropic 외에 다른 LLM API를 연결하는 것도 가능

> Gemini CLI, Cursor, GitHub Copilot Agent 같은 것들도 같은 카테고리의 도구입니다.

---

## 3. 프로젝트 구조

### 최상위 디렉토리

```
claw-code/
├── src/          ← 🐍 Python으로 다시 작성한 코드 (프로토타입)
├── rust/         ← 🦀 Rust로 다시 작성한 코드 (메인 개발 중)
├── tests/        ← ✅ Python 코드 테스트
├── assets/       ← 🖼️ README에 사용되는 이미지들
├── PARITY.md     ← 📊 원본과의 기능 비교 분석 문서
└── README.md     ← 📖 프로젝트 소개
```

### Rust 코드 구조 (메인 개발)

```
rust/crates/
├── api/              ← Anthropic API 통신 (HTTP 요청/응답)
├── commands/         ← /help, /status 같은 슬래시 명령어
├── compat-harness/   ← 호환성 레이어
├── runtime/          ← 핵심 실행 엔진 (대화 관리, 설정, 세션)
├── rusty-claude-cli/ ← CLI 메인 진입점 (사용자가 실행하는 프로그램)
└── tools/            ← AI가 사용하는 도구들 (파일 읽기, 검색 등)
```

### Python 코드 구조 (프로토타입)

| 파일 | 역할 |
|------|------|
| [main.py](file:///Users/sangsulee/claw-code/src/main.py) | CLI 시작점 — 명령어를 받아 실행 |
| [runtime.py](file:///Users/sangsulee/claw-code/src/runtime.py) | 핵심 실행 엔진 |
| [tools.py](file:///Users/sangsulee/claw-code/src/tools.py) | AI가 사용하는 도구 정의 |
| [commands.py](file:///Users/sangsulee/claw-code/src/commands.py) | 슬래시 명령어 정의 |
| [models.py](file:///Users/sangsulee/claw-code/src/models.py) | 데이터 구조 정의 |
| [query_engine.py](file:///Users/sangsulee/claw-code/src/query_engine.py) | 포팅 상태 요약 엔진 |
| [parity_audit.py](file:///Users/sangsulee/claw-code/src/parity_audit.py) | 원본과의 기능 비교 감사 |

| 언어 | 역할 | 장점 |
|------|------|------|
| **Python** (`src/`) | 초기 프로토타입 & 구조 분석 | 빠르게 작성 가능, 읽기 쉬움 |
| **Rust** ([rust/](file:///Users/sangsulee/claw-code/rust/crates/rusty-claude-cli/src/main.rs#3724-3730)) | 최종 프로덕션 버전 (메인) | 빠른 실행 속도, 메모리 안전성 |

---

## 4. 핵심 개념: 에이전트 하네스

AI 에이전트가 실제로 동작하려면 다양한 부품이 필요합니다:

| 부품 | 하는 일 | 비유 |
|------|---------|------|
| **도구(Tools)** | 파일 읽기/쓰기, 검색, 웹 요청 등 | 🔧 AI의 손과 발 |
| **명령어(Commands)** | `/help`, `/status` 같은 사용자 명령 | 🎮 게임 컨트롤러 버튼 |
| **런타임(Runtime)** | AI와 도구를 연결하고 대화를 관리 | 🧠 AI의 뇌 |
| **API 클라이언트** | Anthropic 서버와 통신 | 📡 외부와의 전화선 |
| **세션(Session)** | 대화 기록 저장 | 📝 대화 노트 |
| **훅(Hooks)** | 도구 실행 전/후에 추가 동작 수행 | 🪝 자동화된 감시자 |

이 모든 부품을 하나로 묶어 AI가 실제로 동작할 수 있게 만드는 시스템을 **"하네스(Harness)"** 라고 합니다.

### 전체 동작 흐름

```mermaid
graph LR
    A[👤 사용자 입력] --> B[CLI 진입점]
    B --> C[Runtime 엔진]
    C --> D[Anthropic API]
    D --> E[🤖 AI 응답]
    E --> C
    C --> F{도구 호출 필요?}
    F -->|예| G[Tools 실행]
    G --> C
    F -->|아니오| H[💬 결과 출력]
```

---

## 5. 도구 호출 구조 (read_file 예시)

AI가 [read_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#132-157) 도구를 호출할 때, 코드는 **3개의 레이어**를 거칩니다.

### 1️⃣ 도구 정의 (스펙) — "이런 도구가 있어요"

📄 [rust/crates/tools/src/lib.rs](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs) (79~93행)

```rust
ToolSpec {
    name: "read_file",
    description: "Read a text file from the workspace.",
    input_schema: json!({
        "type": "object",
        "properties": {
            "path": { "type": "string" },
            "offset": { "type": "integer", "minimum": 0 },
            "limit": { "type": "integer", "minimum": 1 }
        },
        "required": ["path"],
    }),
    required_permission: PermissionMode::ReadOnly,
},
```

→ 이 정의가 **Anthropic API에 전달**되어 AI(Claude)가 "나는 [read_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#132-157)이라는 도구를 쓸 수 있구나"라고 인식합니다.

### 2️⃣ 도구 라우팅 (분기) — "AI가 호출하면 어디로 보내?"

📄 [rust/crates/tools/src/lib.rs](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs) (383~408행)

```rust
pub fn execute_tool(name: &str, input: &Value) -> Result<String, String> {
    match name {
        "bash"        => from_value::<BashCommandInput>(input).and_then(run_bash),
        "read_file"   => from_value::<ReadFileInput>(input).and_then(run_read_file),  // ← 여기!
        "write_file"  => from_value::<WriteFileInput>(input).and_then(run_write_file),
        "edit_file"   => from_value::<EditFileInput>(input).and_then(run_edit_file),
        "glob_search" => from_value::<GlobSearchInputValue>(input).and_then(run_glob_search),
        "grep_search" => from_value::<GrepSearchInput>(input).and_then(run_grep_search),
        // ... 나머지 도구들
        _ => Err(format!("unsupported tool: {name}")),
    }
}
```

→ AI 응답에서 `"read_file"` 도구 호출이 오면, [run_read_file](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#419-423) 함수로 분기합니다.

### 3️⃣ 실제 구현 — "진짜 파일을 읽는 코드"

📄 [rust/crates/runtime/src/file_ops.rs](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs) (132~156행)

```rust
pub fn read_file(
    path: &str,
    offset: Option<usize>,
    limit: Option<usize>,
) -> io::Result<ReadFileOutput> {
    let absolute_path = normalize_path(path)?;          // 경로 정리
    let content = fs::read_to_string(&absolute_path)?;  // 실제 파일 읽기!
    let lines: Vec<&str> = content.lines().collect();    // 줄 단위로 분리
    let start_index = offset.unwrap_or(0).min(lines.len());
    let end_index = limit.map_or(lines.len(), |limit| {
        start_index.saturating_add(limit).min(lines.len())
    });
    let selected = lines[start_index..end_index].join("\n"); // 원하는 범위만 추출

    Ok(ReadFileOutput {
        kind: String::from("text"),
        file: TextFilePayload {
            file_path: absolute_path.to_string_lossy().into_owned(),
            content: selected,
            num_lines: end_index.saturating_sub(start_index),
            start_line: start_index.saturating_add(1),
            total_lines: lines.len(),
        },
    })
}
```

→ `fs::read_to_string()`으로 **실제 파일 시스템에서 파일을 읽고**, offset/limit으로 특정 줄 범위만 잘라서 반환합니다.

### 3개 레이어 요약

```
도구 스펙 정의 (ToolSpec)     → API에 전달, AI가 도구 존재를 인식
         ↓
도구 라우팅 (execute_tool)    → 이름으로 분기하여 해당 함수 호출
         ↓
실제 구현 (file_ops.rs)       → fs::read_to_string()으로 파일 읽기
```

이 패턴은 [write_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#158-179), [edit_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#180-219), [bash](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#414-418) 등 **모든 도구에 동일하게 적용**됩니다.

---

## 6. API 통신에서 실제로 오가는 데이터

### 📤 1단계: Claw Code → Anthropic API 요청

사용자가 "main.rs 파일 설명해줘"라고 입력하면, Claw Code가 API에 보내는 요청입니다.

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 16384,
  "system": "You are an AI assistant with access to tools...",
  "messages": [
    {
      "role": "user",
      "content": [{ "type": "text", "text": "main.rs 파일 설명해줘" }]
    }
  ],
  "tools": [
    {
      "name": "read_file",
      "description": "Read a text file from the workspace.",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": { "type": "string" },
          "offset": { "type": "integer", "minimum": 0 },
          "limit": { "type": "integer", "minimum": 1 }
        },
        "required": ["path"]
      }
    }
  ],
  "stream": true
}
```

> [tools](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#2926-2948) 배열에 사용 가능한 도구들의 스펙을 함께 전달합니다. AI는 이걸 보고 "어떤 도구를 쓸 수 있는지" 파악합니다.

### 📥 2단계: Anthropic API → Claw Code 응답 (AI가 도구 호출을 결정)

Claude가 "파일을 먼저 읽어봐야겠다"고 판단하면, 이런 응답을 보냅니다.

```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "main.rs 파일을 먼저 읽어보겠습니다."
    },
    {
      "type": "tool_use",
      "id": "toolu_01A09q90qw90lq917835lq9",
      "name": "read_file",
      "input": {
        "path": "src/main.rs",
        "offset": 0,
        "limit": 200
      }
    }
  ],
  "stop_reason": "tool_use",
  "usage": { "input_tokens": 1200, "output_tokens": 85 }
}
```

> [!IMPORTANT]
> **`stop_reason: "tool_use"`** 가 핵심입니다!
> "나는 아직 답변이 끝나지 않았고, 도구 실행 결과를 기다리고 있어"라는 뜻입니다.

### ⚙️ 3단계: Claw Code가 도구를 실행하고 결과를 얻음

[execute_tool("read_file", ...)](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#383-409) → [run_read_file()](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#419-423) → [read_file()](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#132-157) 순서로 실행되어 결과가 나옵니다:

```json
{
  "type": "text",
  "file": {
    "filePath": "/Users/user/project/src/main.rs",
    "content": "use std::io::{self, Write};\nuse std::path::PathBuf;\n...(파일 내용)...",
    "numLines": 200,
    "startLine": 1,
    "totalLines": 3898
  }
}
```

### 📤 4단계: Claw Code → API에 도구 결과를 전달 (다음 대화 턴)

도구 실행 결과를 **대화 히스토리에 추가**하여 다시 API를 호출합니다:

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 16384,
  "messages": [
    {
      "role": "user",
      "content": [{ "type": "text", "text": "main.rs 파일 설명해줘" }]
    },
    {
      "role": "assistant",
      "content": [
        { "type": "text", "text": "main.rs 파일을 먼저 읽어보겠습니다." },
        {
          "type": "tool_use",
          "id": "toolu_01A09q90qw90lq917835lq9",
          "name": "read_file",
          "input": { "path": "src/main.rs", "offset": 0, "limit": 200 }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": [
            {
              "type": "text",
              "text": "{\"type\":\"text\",\"file\":{\"filePath\":\"...\",\"content\":\"...\",\"numLines\":200,\"startLine\":1,\"totalLines\":3898}}"
            }
          ],
          "is_error": false
        }
      ]
    }
  ]
}
```

> [!NOTE]
> [tool_result](file:///Users/sangsulee/claw-code/rust/crates/api/src/types.rs#42-59)의 `tool_use_id`가 2단계의 [tool_use](file:///Users/sangsulee/claw-code/rust/crates/rusty-claude-cli/src/main.rs#2581-2596)의 [id](file:///Users/sangsulee/claw-code/rust/crates/rusty-claude-cli/src/main.rs#2349-2382)와 **일치**해야 합니다.
> 이를 통해 API가 "이 결과는 아까 그 도구 호출에 대한 응답이구나"라고 연결합니다.

### 📥 5단계: 최종 답변

Claude가 파일 내용을 보고 최종 답변을 생성합니다:

```json
{
  "content": [
    {
      "type": "text",
      "text": "이 main.rs 파일은 Claw Code CLI의 진입점입니다. 주요 기능은..."
    }
  ],
  "stop_reason": "end_turn"
}
```

> `stop_reason: "end_turn"` → 더 이상 도구 호출 없이 **대화가 완료**되었다는 뜻입니다.

### 🔄 전체 흐름 한눈에 보기

```
[사용자] "main.rs 설명해줘"
    ↓
[Claw → API] messages + tools(read_file 스펙)
    ↓
[API → Claw] stop_reason: "tool_use"  ←── 🔑 AI가 도구 호출을 결정!
             content: [{ tool_use: "read_file", input: {path: "src/main.rs"} }]
    ↓
[Claw 내부]  execute_tool("read_file") → 실제 파일 읽기
    ↓
[Claw → API] tool_result: { 파일 내용 JSON }
    ↓
[API → Claw] stop_reason: "end_turn"  ←── 최종 답변 완료
             content: [{ text: "이 파일은..." }]
    ↓
[사용자에게 출력]
```

> [!TIP]
> AI가 여러 도구를 연속으로 호출해야 할 때는 이 루프가 여러 번 반복됩니다.
> 예를 들어 파일을 읽고([read_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#132-157)) → 수정하고([edit_file](file:///Users/sangsulee/claw-code/rust/crates/runtime/src/file_ops.rs#180-219)) → 테스트를 돌리는([bash](file:///Users/sangsulee/claw-code/rust/crates/tools/src/lib.rs#414-418)) 3번의 루프가 돌 수 있습니다.

---

## 7. 원본과의 기능 비교 현황

원본(TypeScript)과 Rust 포트의 기능 비교 현황입니다. 상세 내용은 [PARITY.md](file:///Users/sangsulee/claw-code/PARITY.md)에 정리되어 있습니다.

| 기능 영역 | 상태 | 설명 |
|-----------|------|------|
| 🔧 기본 도구 (파일, 검색, 셸) | ✅ 구현됨 | 핵심 도구는 작동 |
| 📡 API/인증 | ✅ 구현됨 | Anthropic API 통신 가능 |
| 💬 대화 루프 | ✅ 구현됨 | AI와의 대화 가능 |
| 🪝 Hooks | ⚠️ 부분적 | 설정만 파싱, 실행 안 됨 |
| 🔌 Plugins | ❌ 미구현 | 플러그인 시스템 없음 |
| 🎯 Skills | ⚠️ 기초만 | 로컬 파일만 지원 |
| 🖥️ CLI 명령어 | ⚠️ 부분적 | 핵심만 구현, 많은 명령어 누락 |

---

## 8. 실행 방법

### Python 버전

```bash
# 포팅 현황 요약 보기
python3 -m src.main summary

# 현재 워크스페이스 매니페스트 출력
python3 -m src.main manifest

# 테스트 실행
python3 -m unittest discover -s tests -v
```

### Rust 버전

```bash
cd rust/

# 코드 포맷 검사
cargo fmt

# 린트 검사
cargo clippy --workspace --all-targets -- -D warnings

# 테스트 실행
cargo test --workspace
```

---

## 9. 초보 개발자를 위한 학습 포인트

이 프로젝트를 통해 배울 수 있는 것들:

| 학습 주제 | 관련 코드 | 설명 |
|-----------|-----------|------|
| AI 에이전트 아키텍처 | `rust/crates/` 전체 | 도구 호출 루프, API 통신, 세션 관리 |
| Anthropic Tool Use API | `rust/crates/api/src/types.rs` | `tool_use` / `tool_result` 프로토콜 |
| Rust 프로젝트 구조 | `rust/Cargo.toml`, `rust/crates/` | Cargo workspace, crate 분리 패턴 |
| CLI 설계 패턴 | `rust/crates/rusty-claude-cli/` | 인자 파싱, REPL, 슬래시 명령어 |
| 파일 시스템 조작 | `rust/crates/runtime/src/file_ops.rs` | 파일 읽기/쓰기/편집/검색 구현 |
| 클린룸 리라이트 방법론 | 프로젝트 전체 | 다른 언어로 포팅하는 실전 사례 |
| 오픈소스 프로젝트 관리 | `README.md`, `PARITY.md` | 문서화, 테스트, 커뮤니티 운영 |
