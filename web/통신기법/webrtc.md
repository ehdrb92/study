## WebRTC

**WebRTC (Web Real-Time Communication)** 는 Google이 2011년 오픈소스로 공개하고 2021년 W3C/IETF 공식 표준이 된 기술로, **브라우저와 모바일 앱 간 P2P 실시간 통신**을 플러그인 없이 가능하게 합니다.

지금까지 소개한 모든 프로토콜은 **클라이언트 ↔ 서버** 구조였지만, WebRTC는 **클라이언트 ↔ 클라이언트** 직접 통신이 핵심입니다.

### 핵심 개념

```
기존 방식:  클라이언트 A → 서버 → 클라이언트 B
WebRTC:     클라이언트 A ←————————→ 클라이언트 B (P2P)
```

서버를 거치지 않기 때문에 **지연이 매우 낮고**, 서버 대역폭 비용도 절감됩니다. 다만 P2P 연결을 맺기 위한 초기 협상 과정이 꽤 복잡합니다.

### 연결 수립 과정

WebRTC 연결은 크게 세 단계를 거칩니다.

**1단계 — 시그널링 (Signaling)**

두 클라이언트가 서로를 찾고 연결 정보를 교환하는 과정입니다. WebRTC 자체는 시그널링 방법을 규정하지 않아서, 개발자가 직접 구현해야 합니다. 보통 WebSocket이나 REST API를 활용합니다.

```
A: "나는 이런 미디어 포맷을 지원해" (SDP Offer)
B: "나도 이런 포맷 지원해, 연결하자" (SDP Answer)
```

**2단계 — ICE (Interactive Connectivity Establishment)**

방화벽, NAT 뒤에 있는 클라이언트끼리 실제로 연결 가능한 경로를 찾는 과정입니다. STUN/TURN 서버가 이 역할을 돕습니다.

```
STUN 서버: "너의 공인 IP/포트가 뭔지 알려줄게" (무료, 경량)
TURN 서버: "P2P 안되면 내가 중계해줄게" (유료, 서버 비용 발생)
```

**3단계 — DTLS/SRTP로 암호화된 P2P 연결 수립**

협상이 끝나면 이후 미디어 데이터는 서버를 거치지 않고 직접 전송됩니다.

### Python 시그널링 서버 예시

```python
# 시그널링 서버 (WebSocket 기반, websockets 라이브러리)
import asyncio
import websockets
import json

rooms = {}  # {room_id: [client1, client2]}

async def signaling(websocket, path):
    room_id = None
    try:
        async for message in websocket:
            data = json.loads(message)
            msg_type = data["type"]

            if msg_type == "join":
                room_id = data["room"]
                rooms.setdefault(room_id, []).append(websocket)
                print(f"{room_id} 방 입장")

            elif msg_type in ("offer", "answer", "ice-candidate"):
                # 같은 방의 상대방에게 시그널링 메시지 전달
                peers = [p for p in rooms.get(room_id, []) if p != websocket]
                for peer in peers:
                    await peer.send(json.dumps(data))

    finally:
        if room_id and websocket in rooms.get(room_id, []):
            rooms[room_id].remove(websocket)

asyncio.run(websockets.serve(signaling, "localhost", 8765))
```

```javascript
// 클라이언트 (브라우저)
const pc = new RTCPeerConnection({
    iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
});

// 카메라/마이크 스트림 획득
const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
stream.getTracks().forEach(track => pc.addTrack(track, stream));

// ICE 후보 발견 시 상대방에게 전송
pc.onicecandidate = (e) => {
    if (e.candidate) ws.send(JSON.stringify({ type: "ice-candidate", candidate: e.candidate }));
};

// Offer 생성 및 전송
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
ws.send(JSON.stringify({ type: "offer", sdp: offer }));
```

### 3가지 핵심 API

WebRTC는 브라우저에서 세 가지 핵심 API를 제공합니다.

**MediaStream API**는 카메라, 마이크, 화면 캡처 등 미디어 입력을 다룹니다. **RTCPeerConnection API**는 P2P 연결 수립과 미디어/데이터 전송을 담당합니다. **RTCDataChannel API**는 미디어 외에도 임의의 데이터를 P2P로 주고받을 수 있게 합니다.

```javascript
// RTCDataChannel — 파일 전송, 게임 데이터 등 임의 데이터 P2P 전송
const dataChannel = pc.createDataChannel("chat");
dataChannel.onmessage = (e) => console.log("수신:", e.data);
dataChannel.send("P2P 메시지!");
```

### 다른 프로토콜과의 비교

| 항목 | WebSocket | WebRTC |
|------|-----------|--------|
| 통신 구조 | 클라이언트 ↔ 서버 | 클라이언트 ↔ 클라이언트 (P2P) |
| 전송 프로토콜 | TCP | UDP (미디어) + TCP (시그널링) |
| 지연(Latency) | 낮음 | **매우 낮음** |
| 미디어 전송 | 별도 처리 필요 | **기본 내장** |
| 구현 복잡도 | 낮음 | **높음** |
| 서버 비용 | 모든 트래픽 통과 | P2P라 미디어 비용 절감 |

### 장단점

장점으로는 브라우저에 기본 내장되어 별도 플러그인이 필요 없고, P2P 특성상 지연이 매우 낮습니다. 미디어 스트리밍에 최적화된 코덱(VP8, VP9, H.264, Opus)을 기본 지원하며, 모든 통신이 암호화(DTLS + SRTP)됩니다.

단점으로는 시그널링, ICE, STUN/TURN 등 초기 연결 과정이 복잡하고, 참여자가 많아지면 P2P 연결 수가 기하급수적으로 늘어나 **SFU/MCU** 같은 미디어 서버가 별도로 필요합니다. 또한 방화벽 환경에 따라 TURN 서버 비용이 발생할 수 있습니다.

```
3명 P2P:  A↔B, B↔C, A↔C  →  연결 3개
10명 P2P: 연결 45개 → 사실상 불가능
→ SFU 서버 도입 필요 (각자 서버 1개씩만 연결)
```

### 주요 사용 사례

화상회의(Google Meet, Zoom 웹버전), 화상통화(WhatsApp Web), 온라인 게임의 실시간 음성채팅, P2P 파일 공유, 라이브 스트리밍 등에 사용됩니다. 특히 **지연이 곧 사용자 경험인** 실시간 미디어 서비스에서 사실상 표준입니다.
