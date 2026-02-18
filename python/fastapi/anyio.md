FastAPI은 내부적으로 Starlette에 강하게 의존하고 있고, Starlette은 AnyIO를 기반으로 비동기를 구현한다. AnyIO란 무엇인가?

AnyIO는 다음과 같은 주요 특정을 가진다.

- 태스크 그룹
- 고수준 네트워킹
- 작업자 스레드
- python 내장 asyncio와 trio 위에서 동작하는 호환성 레이어

Starlette이 정확히 어떤 점 때문에 anyio를 내부적으로 사용하는지 공식적인 문서는 찾지 못했다. 떠다니는 정보를 취합하면 다음과 같은 이유로 확인된다.

- Starlette은 python의 표준 asyncio 뿐만 아니라 Trio 위에서도 동작하는 ASGI 프레임워크를 목표로 했기 때문에 두 백엔드를 지원하는 도구로 anyio를 선택했다.
- python의 표준 asyncio 대비 더 풍부한 기능이 필요했다. (CapacityLimiter, CancelScope 통합, contextvars 전파 등)