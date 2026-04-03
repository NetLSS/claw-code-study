# 🌊 SSE 스트리밍 완전 정복

> "API 호출을 여러 번 하는 건가요?" — 아닙니다!
> **한 번만 요청**하고, **응답이 실시간으로 조금씩** 도착하는 것입니다.

---

## 목차

1. [먼저 알아야 할 것: 일반 HTTP 통신](#1-먼저-알아야-할-것-일반-http-통신)
2. [SSE란 무엇인가?](#2-sse란-무엇인가)
3. [왜 SSE가 필요한가? — ChatGPT/Claude가 글자를 하나씩 출력하는 이유](#3-왜-sse가-필요한가--chatgptclaude가-글자를-하나씩-출력하는-이유)
4. [SSE의 실제 데이터 형태](#4-sse의-실제-데이터-형태)
5. [Claw Code에서 SSE를 처리하는 방법](#5-claw-code에서-sse를-처리하는-방법)
6. [전체 흐름 정리](#6-전체-흐름-정리)

---

## 1. 먼저 알아야 할 것: 일반 HTTP 통신

보통 우리가 API를 호출하면 이렇게 동작합니다:

```
📱 클라이언트                              🖥️ 서버
    |                                       |
    |──── "안녕하세요" 요청 ────────────────→|
    |                                       |  (서버가 전체 답변을 다 만듦)
    |                                       |  (5초 ~ 30초 대기...)
    |←── "안녕하세요! 저는 Claude입니다.     |
    |     무엇을 도와드릴까요?" 응답 ────────|
    |                                       |
```

**특징:**
- 요청 1번 → 응답 1번
- 서버가 답변을 **전부 완성한 다음에** 한꺼번에 보냄
- 사용자는 답변이 올 때까지 **빈 화면을 보며 기다림** 😴

> [!IMPORTANT]
> **이것이 "일반 HTTP"입니다.**
> 맛집에서 음식을 주문하고 **전부 다 나올 때까지 기다리는 것**과 같습니다.
> 전채 → 메인 → 디저트가 모두 준비될 때까지 빈 테이블 앞에서 대기합니다.

---

## 2. SSE란 무엇인가?

**SSE (Server-Sent Events)** 는 서버가 응답을 **한 번에 다 보내지 않고, 조금씩 실시간으로 보내는** 방식입니다.

```
📱 클라이언트                              🖥️ 서버
    |                                       |
    |──── "안녕하세요" 요청 1번! ──────────→|
    |                                       |
    |←── "안녕" ─────────────────────────── |  ← 0.1초 후
    |←── "하세요!" ──────────────────────── |  ← 0.2초 후
    |←── " 저는" ────────────────────────── |  ← 0.3초 후
    |←── " Claude" ──────────────────────── |  ← 0.4초 후
    |←── "입니다." ──────────────────────── |  ← 0.5초 후
    |←── " 무엇을" ──────────────────────── |  ← 0.6초 후
    |←── " 도와드릴까요?" ───────────────── |  ← 0.7초 후
    |←── [완료 신호] ────────────────────── |  ← 0.8초 후
    |                                       |
```

**핵심 포인트:**
- ✅ 요청은 **딱 1번**입니다! (여러 번 요청하는 게 아닙니다!)
- ✅ 응답이 **여러 조각으로 나뉘어** 실시간으로 도착합니다
- ✅ HTTP 연결이 **열린 상태로 유지**되면서 서버가 데이터를 계속 밀어넣습니다

> [!TIP]
> 뷔페가 아니라 **회전 초밥**을 떠올리면 됩니다. 🍣
> 주문(요청)은 한 번, 하지만 초밥이 **컨베이어 벨트를 타고 하나씩** 도착합니다.
> 첫 번째 초밥이 도착하면 **바로 먹을 수 있어요!** (전부 다 올 때까지 기다릴 필요 없음)

---

## 3. 왜 SSE가 필요한가? — ChatGPT/Claude가 글자를 하나씩 출력하는 이유

ChatGPT나 Claude 웹 채팅을 사용해보셨다면, 답변이 **한 글자씩 타자를 치듯** 나타나는 것을 보셨을 겁니다.

### SSE를 안 쓰면 (일반 HTTP)

```
사용자: "Rust의 장점 10가지를 알려줘"
        ↓
     [빈 화면... 15초 대기... ⏳]
        ↓
     💥 갑자기 긴 글이 한꺼번에 나타남!
```

**문제점:**
- 😤 사용자는 15초 동안 **아무것도 안 보임** → "고장났나?" 불안함
- 🤯 답변이 길수록 대기 시간이 더 김 (30초? 1분?)
- ❌ 중간에 "그건 됐고 다른 거 물어볼래" 할 수 없음

### SSE를 쓰면 (스트리밍)

```
사용자: "Rust의 장점 10가지를 알려줘"
        ↓ (0.3초 후)
     "1. 메모리 안전성 — "
        ↓ (0.5초 후)
     "1. 메모리 안전성 — Rust는 소유권 시스템을 통해..."
        ↓ (0.8초 후)
     "2. 제로 코스트 추상화 — "
        ↓ (계속 이어짐...)
```

**장점:**
- 😊 0.3초 만에 **첫 글자가 보임** → 즉각적인 반응감
- 📖 응답을 **읽으면서 동시에** 나머지 답변이 생성됨
- ⏱️ 체감 대기 시간이 **극적으로 줄어듦**

> [!NOTE]
> **LLM은 토큰을 하나씩 순서대로 생성**합니다.
> 답변 전체를 미리 만들어두는 게 아니라, "다음에 올 단어"를 계속 예측하는 방식입니다.
> SSE는 이 특성과 완벽히 맞습니다 — 생성되는 즉시 보내면 되니까요!

---

## 4. SSE의 실제 데이터 형태

SSE 데이터는 특별한 텍스트 형식을 따릅니다. JSON도 아니고, XML도 아닌 **SSE만의 간단한 규약**이 있습니다.

### 기본 규칙

```
event: 이벤트_이름
data: 실제_데이터_내용

```

- `event:` 줄 → 이 데이터가 뭔지 알려주는 **이름표**
- `data:` 줄 → 실제 전달하려는 **데이터** (보통 JSON)
- **빈 줄** → 하나의 메시지 끝! (다음 메시지와의 **경계선**)

### Claude API에서 실제로 오는 SSE 데이터

사용자가 "안녕"이라고 보냈을 때, Claude API가 보내는 SSE 스트림의 **실제 모습**입니다:

```
event: message_start                          ──┐
data: {"type":"message_start","message":{...}}  │ 📦 1번 메시지: "대화 시작!"
                                                │
                                              ──┘ (빈 줄 = 메시지 끝)
event: content_block_start                    ──┐
data: {"type":"content_block_start",            │ 📦 2번 메시지: "텍스트 블록 시작"
       "index":0,                               │
       "content_block":{"type":"text","text":""}}│
                                              ──┘
event: content_block_delta                    ──┐
data: {"type":"content_block_delta",            │ 📦 3번 메시지: 텍스트 조각 "안녕"
       "index":0,                               │
       "delta":{"type":"text_delta",            │
                "text":"안녕"}}                  │
                                              ──┘
event: content_block_delta                    ──┐
data: {"type":"content_block_delta",            │ 📦 4번 메시지: 텍스트 조각 "하세요!"
       "index":0,                               │
       "delta":{"type":"text_delta",            │
                "text":"하세요!"}}               │
                                              ──┘
event: content_block_delta                    ──┐
data: {"type":"content_block_delta",            │ 📦 5번 메시지: 텍스트 조각 " 무엇을..."
       "index":0,                               │
       "delta":{"type":"text_delta",            │
                "text":" 무엇을 도와드릴까요?"}}  │
                                              ──┘
event: content_block_stop                     ──┐
data: {"type":"content_block_stop","index":0}   │ 📦 6번 메시지: "텍스트 블록 끝"
                                              ──┘
event: message_delta                          ──┐
data: {"type":"message_delta",                  │ 📦 7번 메시지: 종료 사유 + 토큰 사용량
       "delta":{"stop_reason":"end_turn"},       │
       "usage":{"input_tokens":12,              │
                "output_tokens":8}}              │
                                              ──┘
event: message_stop                           ──┐
data: {"type":"message_stop"}                   │ 📦 8번 메시지: "끝!"
                                              ──┘
```

### 각 이벤트의 역할 정리

```
📨 message_start        → "지금부터 AI 응답이 시작됩니다"
📝 content_block_start  → "텍스트(또는 도구호출) 블록이 시작됩니다"
✏️ content_block_delta  → "여기 텍스트 조각이요!" (이게 여러 번 반복!)
📝 content_block_stop   → "이 블록 끝났습니다"
📊 message_delta        → "종료 사유와 토큰 사용량입니다"
🏁 message_stop         → "전체 응답 완료!"
```

> [!IMPORTANT]
> **핵심은 `content_block_delta`입니다!**
> 이 이벤트가 **여러 번 반복**되면서 텍스트가 조금씩 도착합니다.
> "안녕" → "하세요!" → " 무엇을 도와드릴까요?" 이렇게요.
> 이것이 바로 **타자를 치듯** 글자가 하나씩 나타나는 원리입니다.

### 도구 호출이 포함된 경우

AI가 "파일을 읽어볼게요"라고 하면서 동시에 read_file 도구를 호출할 때:

```
[content_block_delta] text: "파일을 먼저"
[content_block_delta] text: " 읽어보겠습니다."
[content_block_stop]  ← 텍스트 블록 끝

[content_block_start] type: "tool_use", name: "read_file"  ← 도구 호출 블록 시작!
[content_block_delta] partial_json: '{"path":'
[content_block_delta] partial_json: '"src/main.rs"'
[content_block_delta] partial_json: '}'
[content_block_stop]  ← 도구 호출 블록 끝

[message_delta] stop_reason: "tool_use"  ← ⚠️ "end_turn"이 아니라 "tool_use"!
[message_stop]
```

> `stop_reason`이 `"tool_use"` 라는 것은:
> "나는 아직 답변이 안 끝났어요! 도구 실행 결과를 기다리고 있어요!" 라는 뜻입니다.

---

## 5. Claw Code에서 SSE를 처리하는 방법

### 문제: 네트워크 데이터는 "깔끔하게" 오지 않는다

SSE 명세상 각 메시지는 빈 줄(`\n\n`)로 구분됩니다.
하지만 실제 네트워크에서는 **TCP 패킷 크기에 따라 임의로 잘려서** 옵니다:

```
이상적인 상황 (실제로는 거의 없음):
──────────────
[패킷 1] "event: content_block_delta\ndata: {\"text\":\"Hello\"}\n\n"
[패킷 2] "event: content_block_delta\ndata: {\"text\":\" World\"}\n\n"

실제 상황 (자주 발생):
──────────────
[패킷 1] "event: content_block_delta\ndata: {\"text\":\"Hel"
[패킷 2] "lo\"}\n\nevent: content_block_delta\ndata: {\"text\":\" Wor"
[패킷 3] "ld\"}\n\n"
```

보이시나요? 패킷 1은 메시지가 **중간에 잘렸고**, 패킷 2에는 **이전 메시지의 끝 + 새 메시지의 시작**이 섞여 있습니다.

### 해결: SseParser — 버퍼에 모아서 경계를 찾는다

📄 [rust/crates/api/src/sse.rs](file:///Users/sangsulee/claw-code/rust/crates/api/src/sse.rs)

```rust
pub struct SseParser {
    buffer: Vec<u8>,   // 🪣 "아직 처리 안 된 데이터" 보관소
}
```

동작 원리를 **비유로** 설명하면:

```
SseParser는 "문장 완성 게임"을 하는 사람과 같습니다.

1. 누군가 단어 카드를 무작위 크기로 잘라서 던져줍니다
2. 나는 카드를 받을 때마다 책상 위에 이어붙입니다 (buffer에 추가)
3. 온점(.)을 발견하면 → "문장 하나 완성!" → 완성된 문장을 처리
4. 온점이 아직 없으면 → 계속 카드를 받으며 기다림
```

여기서 "온점"에 해당하는 것이 **빈 줄 `\n\n`** 입니다.

### 코드로 보기

```rust
impl SseParser {
    pub fn push(&mut self, chunk: &[u8]) -> Result<Vec<StreamEvent>> {
        // 1. 새로 온 데이터를 버퍼 끝에 붙인다
        self.buffer.extend_from_slice(chunk);

        let mut events = Vec::new();

        // 2. 버퍼에서 완전한 프레임(\n\n으로 끝나는)을 찾아 처리
        while let Some(frame) = self.next_frame() {
            if let Some(event) = parse_frame(&frame)? {
                events.push(event);
            }
        }

        // 3. 완성된 이벤트들을 반환 (0개일 수도 있음!)
        Ok(events)
    }

    fn next_frame(&mut self) -> Option<String> {
        // 버퍼에서 "\n\n" 위치를 찾는다
        let position = self.buffer.windows(2)
            .position(|window| window == b"\n\n")?;

        // 찾았으면: 그 위치까지 잘라내서 반환
        let frame = self.buffer.drain(..position + 2).collect();
        Some(String::from_utf8_lossy(&frame).into_owned())
    }
}
```

### 실제 동작 시뮬레이션

```
시간순서대로 네트워크 패킷이 도착합니다:

─── 패킷 1 도착 ───
받은 데이터: "event: content_block_delta\ndata: {\"text\":\"Hel"
버퍼 상태:   "event: content_block_delta\ndata: {\"text\":\"Hel"
\n\n 있나?    ❌ → 아직 프레임 완성 안 됨
반환:        [] (빈 배열)

─── 패킷 2 도착 ───
받은 데이터: "lo\"}\n\n"
버퍼 상태:   "event: content_block_delta\ndata: {\"text\":\"Hello\"}\n\n"
\n\n 있나?    ✅ → 프레임 완성!
파싱 결과:   ContentBlockDelta { text: "Hello" }
반환:        [ContentBlockDelta("Hello")]
             (버퍼는 비워짐)

─── 패킷 3 도착 ───
받은 데이터: "event: content_block_delta\ndata: {\"text\":\" World\"}\n\nevent: message_stop\ndata: {\"type\":\"message_stop\"}\n\n"
버퍼 상태:   (위 전체)
\n\n 있나?    ✅ → 프레임 2개 완성!
반환:        [ContentBlockDelta(" World"), MessageStop]
```

> [!TIP]
> **한 패킷에서 이벤트가 0개, 1개, 또는 여러 개 나올 수 있습니다!**
> - 데이터가 중간에 잘렸으면 → 0개 (다음 패킷을 기다림)
> - 딱 하나의 프레임이 완성됐으면 → 1개
> - 여러 프레임이 한꺼번에 왔으면 → 여러 개

---

## 6. 전체 흐름 정리

### 한 눈에 보는 SSE 스트리밍 전체 과정

```
                                ┌─────────────────────────────┐
                                │     Anthropic Claude 서버     │
                                │                              │
 ① HTTP POST 요청 (1번만!)      │  ② AI가 토큰을 하나씩 생성    │
 {"messages":[...],             │  "안녕" → "하세요" → ...      │
  "stream": true}  ──────────→  │                              │
                                │  ③ 생성되는 즉시 SSE로 전송    │
                                │  event: content_block_delta   │
                                │  data: {"text":"안녕"}         │
                                │                              │
                                │  event: content_block_delta   │
                                │  data: {"text":"하세요"}       │
                                │         ...                   │
                                └──────────┬───────────────────┘
                                           │
                          ④ 네트워크 전송 (TCP 패킷으로 분할)
                                           │
                                ┌──────────▼───────────────────┐
                                │     Claw Code (클라이언트)     │
                                │                              │
                                │  ⑤ SseParser                 │
                                │  패킷 → 버퍼 → \n\n 찾기      │
                                │  → StreamEvent 변환           │
                                │                              │
                                │  ⑥ 화면 출력                  │
                                │  "안녕" (표시)                 │
                                │  "안녕하세요" (이어서 표시)      │
                                │  "안녕하세요 무엇을..." (계속)  │
                                └──────────────────────────────┘
```

### 일반 HTTP vs SSE 비교 최종 정리

| | 일반 HTTP | SSE 스트리밍 |
|---|---|---|
| **요청 횟수** | 1번 | 1번 (동일!) |
| **응답 방식** | 전체 완성 후 한 번에 | 조금씩 실시간으로 |
| **HTTP 연결** | 요청-응답 후 닫힘 | 응답 완료까지 열려있음 |
| **첫 데이터까지 시간** | 길다 (전체 생성 시간) | 짧다 (첫 토큰 생성 시간) |
| **사용자 경험** | 빈 화면 → 갑자기 전체 표시 | 타자 치듯 한 글자씩 표시 |
| **API 요청의 `stream` 파라미터** | `false` (또는 생략) | `true` |
| **비유** | 🍽️ 코스 요리 전부 다 나올 때까지 대기 | 🍣 회전 초밥 하나씩 도착 |

### 핵심 한 줄 요약

> **SSE 스트리밍은 "여러 번 요청"하는 것이 아니라,
> "한 번 요청"하고 "응답이 여러 조각으로 나뉘어 실시간으로 도착"하는 것입니다.**

---

## 💡 보너스: 직접 확인해보기

터미널에서 curl로 SSE 스트리밍을 직접 볼 수 있습니다:

```bash
curl -N https://api.anthropic.com/v1/messages \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 100,
    "stream": true,
    "messages": [{"role": "user", "content": "Hi"}]
  }'
```

`-N` 플래그는 curl의 버퍼링을 끄는 옵션입니다.
실행하면 `event:` 와 `data:` 줄이 **실시간으로** 터미널에 출력되는 것을 볼 수 있습니다!

> `"stream": true` → SSE 스트리밍 활성화
> `"stream": false` (또는 생략) → 일반 HTTP (전체 응답을 한 번에)
