일반적으로 FastAPI의 ASGI 프레임워크 도구로 uvicorn을 사용한다. uvicorn은 어떻게 HTTP 바이너리를 파싱하고, FastAPI의 라우팅 함수에게 전달할까?

전체 흐름을 요약하면 다음과 같다.

```
네트워크 소켓 (raw bytes)
    ↓
asyncio transport → Protocol.data_received()  ← 파싱 시작점
    ↓
httptools 또는 h11 라이브러리가 HTTP 파싱
    ↓
ASGI scope/receive/send 형식으로 변환
    ↓
FastAPI (Starlette) → 라우팅, 의존성 주입 등
```

```python
# uvicorn\protocols\http\httptools_impl.py

def data_received(self, data: bytes) -> None:
    self._unset_keepalive_if_required()

    try:
        self.parser.feed_data(data)
    except httptools.HttpParserError:
        msg = "Invalid HTTP request received."
        self.logger.warning(msg)
        self.send_400_response(msg)
        return
    except httptools.HttpParserUpgrade:
        if self._should_upgrade():
            self.handle_websocket_upgrade()
        else:
            self._unsupported_upgrade_warning()

# uvicorn\protocols\http\h11_impl.py

def data_received(self, data: bytes) -> None:
    self._unset_keepalive_if_required()

    self.conn.receive_data(data)
    self.handle_events()
```

위는 httptools와 h11에서 HTTP 바이너리를 받고 파싱하는 `data_received()` 메서드 부분만 가져온 것이다. 상세한 내부 파싱 과정까지는 여기서 다루지는 않겠다.

uvicorn은 기본적으로 httptools가 설치되어있다면, httptools를 우선적으로 사용한다. httptools는 nodejs의 http 파서(llhttp)를 python 바인딩한 라이브러리로, C 기반이기 때문에 순수 python으로 개발된 h11 대비 고성능으로 알려져있다.

그렇다면 무조건 httptools를 사용하는 것이 이득이 아닌가? 하지만 부득이하게 h11을 사용해야할 상황도 있다.

- httptools는 C 확장(Cython)이 필요하기 때문에, 컴파일 도구가 없는 환경에서는 설치가 제한될 수 있다. 이에 비해 h11은 순수 python이기 때문에 좀 더 자유롭다.
- httptools는 C 수준에서 동작한다는 구조때문에 파싱 중의 내부 상태를 들여다보기 힘들다. 반면 h11은 conn.states, conn.next_event() 등을 직접 찍어보면서 HTTP 파싱 흐름을 추적하기 훨씬 수월하다.
- httptools와 h11은 완전히 동일한 HTTP 요청 처리 구조를 가지지않는다. 그래서 httptools가 호환성 문제가 있을 경우 h11로 교체를 하면 문제가 해결되는 경우가 있다.

정리하면 일반적인 경우나 성능이 중요한 경우라면 httptools를 사용하고, 호환성 제한이 있거나 디버깅 등 특수한 목적을 가지고 있을 경우 h11을 사용할 수 있다.
