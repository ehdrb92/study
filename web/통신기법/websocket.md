## WebSocket

**WebSocket**은 2011년 RFC 6455로 표준화된 프로토콜로, **클라이언트와 서버 간 양방향 실시간 통신**을 가능하게 합니다. HTTP의 한계를 극복하기 위해 등장했습니다.

### 등장 배경

HTTP는 기본적으로 클라이언트가 요청해야만 서버가 응답할 수 있습니다. 실시간 기능(채팅, 알림 등)을 구현하려면 아래처럼 궁여지책을 써야 했습니다.

```
Polling:          클라이언트가 주기적으로 계속 요청 ("새 메시지 있어?")
Long Polling:     서버가 응답을 일부러 늦게 줌 (비효율적)
```

WebSocket은 이 문제를 근본적으로 해결합니다.

### 핵심 개념 — 지속적인 연결

WebSocket은 HTTP로 **핸드셰이크(Handshake)** 를 한 번 수행한 후, 연결을 HTTP에서 WebSocket으로 **업그레이드**합니다. 이후 연결이 끊기기 전까지 양쪽이 자유롭게 메시지를 주고받을 수 있습니다.

```
1. 클라이언트 → 서버: HTTP 업그레이드 요청
   GET /chat HTTP/1.1
   Upgrade: websocket
   Connection: Upgrade

2. 서버 → 클라이언트: 101 Switching Protocols

3. 이후: 양방향 자유 통신 (HTTP 오버헤드 없음)
```

### Python 예시

```python
# 서버 (websockets 라이브러리)
import asyncio
import websockets

connected_clients = set()

async def handler(websocket):
    connected_clients.add(websocket)
    try:
        async for message in websocket:
            print(f"수신: {message}")
            # 연결된 모든 클라이언트에게 브로드캐스트
            for client in connected_clients:
                await client.send(f"[broadcast] {message}")
    finally:
        connected_clients.remove(websocket)

async def main():
    async with websockets.serve(handler, "localhost", 8765):
        await asyncio.Future()  # 서버 계속 실행

asyncio.run(main())
```

```python
# 클라이언트
import asyncio
import websockets

async def client():
    uri = "ws://localhost:8765"
    async with websockets.connect(uri) as websocket:
        await websocket.send("안녕하세요!")
        response = await websocket.recv()
        print(f"응답: {response}")

asyncio.run(client())
```

### REST/HTTP와의 비교

| 항목 | HTTP (REST) | WebSocket |
|------|------------|-----------|
| 연결 방식 | 요청마다 연결/해제 | 한 번 연결 후 유지 |
| 통신 방향 | 단방향 (클라이언트 → 서버) | **양방향** |
| 헤더 오버헤드 | 매 요청마다 발생 | 최초 핸드셰이크만 |
| 실시간성 | 낮음 | **높음** |
| 서버 부하 | Polling 시 높음 | 상대적으로 낮음 |

### 장단점

장점으로는 진정한 양방향 실시간 통신이 가능하고, 연결을 유지하기 때문에 매 요청마다 헤더를 붙일 필요가 없어 오버헤드가 적습니다. 또한 브라우저에서 기본 API로 지원하기 때문에 별도 라이브러리 없이도 사용할 수 있습니다.

단점으로는 연결을 계속 유지해야 하므로 서버 리소스를 지속적으로 점유합니다. 또한 HTTP와 달리 상태를 유지하기 때문에 수평 확장(Scale-out) 시 **Sticky Session** 이나 **Redis Pub/Sub** 같은 별도 전략이 필요합니다. 그리고 REST처럼 표준화된 규격이 없어 메시지 포맷을 직접 설계해야 합니다.

### 주요 사용 사례

실시간성이 핵심인 서비스에 적합합니다. 채팅 애플리케이션, 온라인 게임, 주식/코인 실시간 시세, 협업 도구(Google Docs처럼 동시 편집), 스포츠 실시간 스코어 등에서 활발히 사용됩니다. Slack, Discord, Figma 등이 WebSocket을 핵심 통신 수단으로 사용합니다.
