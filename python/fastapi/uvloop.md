uvicorn은 기본적으로 uvloop와 httptools를 활용하여 매우 빠른 비동기 서버를 구현하는 것을 목표로 삼았다. httptools는 따로 다루었는데, uvloop도 동일하게 nodejs의 libuv(C 기반 비동기 입출력 라이브러리)으로 구현되어 있어서 일반적으로 순수 python으로 구현된 asyncio 대비 2~4배 빠른 속도를 가지고 있다고 알려져 있다.

```python
# uvicorn\loops\auto.py

def auto_loop_factory(use_subprocess: bool = False) -> Callable[[], asyncio.AbstractEventLoop]:
    try:
        import uvloop  # noqa
    except ImportError:  # pragma: no cover
        from uvicorn.loops.asyncio import asyncio_loop_factory as loop_factory

        return loop_factory(use_subprocess=use_subprocess)
    else:  # pragma: no cover
        from uvicorn.loops.uvloop import uvloop_loop_factory

        return uvloop_loop_factory(use_subprocess=use_subprocess)
```

위는 uvicorn이 실행될 때 어떤 도구를 사용할 지 코드로 정의된 부분이다. 기본적으로 uvloop가 설치되어있다면 uvloop를 사용하고, 아니라면 asyncio를 사용한다.

또한 다음과 같이 명령어로 루프의 종류를 선택할 수 있다.

`uvicorn main:app --loop uvloop`

`--loop` 옵션은 auto, asyncio, uvloop 중 선택할 수 있다. 이 외에도 공식문서에 따르면 다른 루프 선택지를 사용할 수 있다. "https://uvicorn.dev/concepts/event-loop/"
