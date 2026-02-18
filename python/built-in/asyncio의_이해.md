참고 자료: https://medium.com/delivus/understanding-pythons-asyncio-a-deep-dive-into-the-event-loop-89a6c5acbc84

## Event Loop

기본적으로 파이썬에 asyncio는 이벤트 루프를 중심으로 구축된다. 이벤트 루프는 다양한 작업을 조율하는 관리자의 역할이다.

이벤트 루프는 몇 개의 핵심 데이터 구조를 운영한다.

1. 동작을 기다리는 작업(task)들이 존재하는 대기 큐
2. 스케쥴된 콜백의 작업 함수들이 존재하는 콜백 큐
3. 입출력 이벤트를 감시하는 파일 디스크립터들의 배열
4. 시간 기반 이벤트들의 배열


## Coroutine

코루틴은 파이썬에서 비동기 프로그래밍의 기본 단위이다. 비동기 함수를 정의하면 파이썬은 코루틴 객체를 생성한다.

```python
async def fetch_data():
    await asyncio.sleep(1)
    return "data"
```

위와 같은 코드가 있다면 다음과 같이 동작한다.

1. 파이썬은 제너레이터를 기반으로하는 코루틴 객체를 생성한다.
2. 코루틴은 비동기 함수 내부에 정의된 await 포인트마다 중지된다.
3. 중지되는 순간 통제권을 이벤트 루프에 반환한다.
4. 이벤트 루프는 반환된 통제권을 다른 작업에 넘기고 언제 재개할지 관리한다.


## Task

작업은 asyncio가 코루틴을 관리하는 방식이다. 작업은 코루틴을 래핑하고, 추가 기능을 제공한다.

```python
class Task:
    def __init__(self, coro, loop):
        self.coro = coro        # The wrapped coroutine
        self._loop = loop       # Reference to the event loop
        self._state = 'PENDING' # Current state
        self._result = None     # Store the result
        self._exception = None  # Store any exception
```

작업 상태와 전환은 다음과 같다.

- PENDING: 초기 상태
- RUNNING: 현재 동작 중
- DONE: 실행 완료
- CANCELLED: 완료되기 전 취소


## Selector

asyncio는 selectors(Linux의 epoll이나 BSD의 kqueue와 유사)를 사용하여 입출력 작업을 효율적으로 관리한다. 작동 방식은 다음과 같다.

1. 입출력을 수행할 때 asyncio는 파일 디스크립터를 selector에 등록한다.
2. selector는 여러 파일 디스크립터를 동시에 감시한다.
3. 입출력 준비가 되면 selector가 이벤트 루프에 알린다.
4. 이벤트 루프는 준비된 코루틴을 재개한다.


## 흔하게 발생하는 비동기 관련 문제점과 모범 사례

1. 이벤트 루프 블록킹
    가장 큰 실수는 코루틴 내부에서 CPU 부하가 높은 작업을 처리하는 것이다.

    ```python
    async def bad_practice():
    # This blocks the event loop!**
    result = [i ** 2 for i in range(10000000**)]
    return result

    async def good_practice():
        # Run in a thread pool instead
        result = await loop.run_in_executor(None, compute_squares)
        return result
    ```

2. 너무 많은 작업을 실행
    코루틴 내부 작업이 가벼운 작업이라도, 너무 많은 작업을 동시에 실행시키면 성능에 영향을 줄 수 있다.

    ```python
    # Bad: Creating tasks in an unbounded loop
    async def spam_tasks():
        while True:
            asyncio.create_task(some_coroutine())

    # Good: Using a semaphore to limit concurrent tasks
    sem = asyncio.Semaphore(100)
    async def controlled_tasks():
        async with sem:
            await some_coroutine()
    ```


## 성능 고려 사항

- 작업 생성: 작업이 가벼워도 자원을 소모하지 않는 것은 아니다. 상황을 보고 신중하게 생성할 수 있도록 하자.
- 이벤트 루프 오버헤드: await 키워드는 컨텍스트 스위칭을 유발한다. 가능하다면 소규모로 작업을 그룹화하여 나누자.
- 메모리 사용량: 코루틴은 자체 스택 프레임을 유지한다. 장시간 실행되는 코루틴을 유의해야한다.


## 코루틴이 제너레이터 기반이라는 것은 무슨 의미일까?

먼저 제너레이터에 대해 간단히 보면, 일반 함수처럼 def로 정의되지만 내부에 yield 키워드를 포함한다. 제너레이터 함수가 실행되면 즉시 실행되는 것이 아니라 제너레이터 객체를 반환한다. 제너레이터 객체의 next() 혹은 send() 등의 메서드를 호출할 때마다 yield 위치까지 실행하고 값을 내놓은 뒤 멈추기를 반복한다.

- 제너레이터 함수는 반복을 위해 사용될 수 있다.
- yield를 통해 값을 내놓고 정지할 수 있다.
- 내부 상태를 보존한 채 다음 호출 시 이어서 실행된다.

그렇다면 코루틴은?

- 제너레이터와 유사하게 yield 또는 await 등을 통해 실행을 중간에 일시정지할 수 있다.
- 내부 상태를 보존하며 다시 실행될 수 있다.
- 값을 외부에 내보내거나 내부로 받을 수 있다.

위 두 가지 특징에 따라 코루틴은 제너레이터 기반이라는 기술적 맥락을 가질 수 있다.

- 파이썬 2.5 버전에서는 제너레이터의 기능이 강화되어 yield가 표현식으로 바뀌고, send, throw, close 메소드 등이 객체에 추가되었다. 해당 기능들이 제너레이터가 입력값을 받고 중단/재개 가능한 형태로 바뀌게 하였고, 코루틴으로 확장할 수 있는 기틀을 만들었다.
- "제너레이터가 코루틴의 일종"이기도 했고, "제너레이터가 코루틴으로 발전했다"고 볼 수도 있다.
- 파이썬 3.5부터 async def/await 구문이 도입되면서, 제너레이터 기반 방식보다는 네이티브 코루틴 방식이 주류가 되었다.


## Coroutine, Task, Future 클래스의 관계

우선 Future 객체이다. 문서에 따르면, asyncio.Future 객체는 비동기 연산의 최종 결과를 나타내는 특별한 awaitable 객체라고 한다. 즉 어떤 작업이 아직 완료되지 않은 상태일 때(예: 네트워크 응답, I/O 작업), 나중에 완료되면 결과가 설정되거나 예외가 설정되거나 취소될 수 있는 컨테이너(container) 역할을 하는 객체이다.

Task는 위에서도 한번 살펴봤는데, 코루틴 객체를 래핑하여 이벤트 루프에 스케쥴링할 수 있도록 만드는 객체이다. 또한 Future 클래스를 상속한 클래스이기도 하다. Future를 상속하도록 설계된 이유는 Task의 역할이 "코루틴을 실행하고 결과(혹은 예외)를 내놓을 것" 이기 때문에 Future의 추후 나올 결과의 역할을 하기 위해서이다. 요약하면 Task는 코루틴을 래핑하여 실행시키는 Future라고 볼 수 있다.

구조적 예시는 다음과 같다.

```
Awaitable (인터페이스 / 개념)
 ├── Coroutine (코루틴 객체)        ← async def 함수 호출 시 얻는 것
 ├── Future                        ← 비동기 결과를 나타내는 객체
 │      └── Task                   ← Future를 상속하고, 코루틴을 감싸 실행하는 객체
```