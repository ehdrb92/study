## SSE (Server-Sent Events)

**SSE**는 HTML5 표준의 일부로, **서버에서 클라이언트로 단방향 실시간 스트리밍**을 가능하게 하는 기술입니다. WebSocket보다 단순하고 HTTP 위에서 그대로 동작한다는 것이 핵심입니다.

### 핵심 개념

WebSocket이 양방향 통신을 위해 프로토콜 자체를 업그레이드하는 반면, SSE는 **HTTP 연결을 그대로 유지하면서 서버가 데이터를 계속 흘려보내는** 방식입니다. 클라이언트는 한 번 연결하면 서버가 보내는 이벤트를 계속 수신합니다.

```
클라이언트 → 서버: GET /stream (한 번만)
서버 → 클라이언트: data: 이벤트1 (계속 흘려보냄)
서버 → 클라이언트: data: 이벤트2
서버 → 클라이언트: data: 이벤트3
                    ...
```

### 메시지 포맷

SSE의 메시지 포맷은 매우 단순합니다. 텍스트 기반이며 `data:`, `event:`, `id:`, `retry:` 필드를 사용합니다.

```
data: 안녕하세요\n\n                     # 기본 메시지

event: temperature\n                     # 이벤트 타입 지정
data: {"value": 23.5}\n\n

id: 42\n                                 # 메시지 ID (재연결 시 활용)
data: 중요한 메시지\n\n

retry: 3000\n                            # 재연결 대기 시간 (ms)
data: 재연결 설정\n\n
```

### Python 예시

```python
# 서버 (FastAPI)
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

async def event_generator():
    count = 0
    while True:
        count += 1
        # SSE 포맷에 맞게 전송
        yield f"id: {count}\n"
        yield f"event: update\n"
        yield f"data: 현재 카운트 = {count}\n\n"
        await asyncio.sleep(1)

@app.get("/stream")
async def stream():
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream",  # SSE의 핵심 Content-Type
        headers={
            "Cache-Control": "no-cache",
            "X-Accel-Buffering": "no"    # nginx 버퍼링 방지
        }
    )
```

```javascript
// 클라이언트 (브라우저 JavaScript)
const evtSource = new EventSource("/stream");

// 기본 메시지 수신
evtSource.onmessage = (e) => {
    console.log("수신:", e.data);
};

// 특정 이벤트 타입 수신
evtSource.addEventListener("update", (e) => {
    console.log("업데이트:", e.data);
});

// 에러 처리
evtSource.onerror = (e) => {
    console.error("연결 오류");
};
```

### SSE의 자동 재연결

SSE의 편리한 기능 중 하나는 **브라우저가 자동으로 재연결**을 시도한다는 점입니다. 연결이 끊기면 브라우저가 알아서 서버에 재연결하며, `id` 필드를 활용하면 마지막으로 받은 메시지 이후부터 이어받을 수도 있습니다.

```
# 재연결 시 클라이언트가 자동으로 보내는 헤더
Last-Event-ID: 42   ← 마지막으로 받은 메시지 ID
```

### WebSocket과의 비교

| 항목 | SSE | WebSocket |
|------|-----|-----------|
| 통신 방향 | 서버 → 클라이언트 (단방향) | 양방향 |
| 프로토콜 | HTTP 그대로 | HTTP → WS 업그레이드 |
| 구현 복잡도 | 낮음 | 높음 |
| 자동 재연결 | ✅ 브라우저 기본 지원 | ❌ 직접 구현 필요 |
| 바이너리 전송 | ❌ 텍스트만 | ✅ |
| HTTP/2 호환 | ✅ 자연스럽게 활용 | 별도 고려 필요 |

### 장단점

장점으로는 HTTP를 그대로 사용하기 때문에 기존 인프라(로드밸런서, 프록시 등)와 호환성이 좋고, 구현이 단순합니다. 자동 재연결과 이벤트 ID 기반 복구도 기본 제공됩니다.

단점으로는 서버 → 클라이언트 단방향이라 클라이언트가 서버로 데이터를 보내려면 별도의 REST API 호출이 필요합니다. 또한 브라우저마다 동시 연결 수 제한이 있어 (HTTP/1.1 기준 도메인당 6개) 많은 탭을 열면 문제가 생길 수 있습니다. 다만 HTTP/2 환경에서는 이 제한이 사실상 해소됩니다.

### 주요 사용 사례

클라이언트가 데이터를 보낼 필요 없이 **서버의 변경사항을 실시간으로 받아야 하는 경우**에 최적입니다. 대표적으로 **ChatGPT처럼 AI 응답을 토큰 단위로 스트리밍**하는 경우, 실시간 알림, 대시보드 실시간 업데이트, 로그 스트리밍 등에 활용됩니다.
