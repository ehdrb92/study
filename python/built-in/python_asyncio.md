eventloop는 asyncio의 핵심 컴포넌트이고, coroutine은 asyncio에서 기본 실행 단위라고 할 수 있다.

## 요구사항

- 10초마다 Job이 생성된다.
- 각 Job은 5 ~ 15초 랜덤한 시간을 소모한다.
- Job들은 동시성으로 실행된다.

## 구현

```python
import asyncio
from datetime import datetime
from random import randint

async def run_job() -> None:
    delay = randint(5, 15)
    print(f'{datetime.now()} sleep for {delay} seconds')
    await asyncio.sleep(delay)  # 5~15초 동안 잠자기
    print(f'{datetime.now()} finished ({delay} sec)')

async def main() -> None:
    while True:
        asyncio.create_task(run_job())
        await asyncio.sleep(10)

asyncio.run(main())

# 2022-03-27 00:55:59.585462 sleep for 13 seconds
# 2022-03-27 00:56:09.590352 sleep for 14 seconds
# 2022-03-27 00:56:12.588084 finished (13 sec)
# 2022-03-27 00:56:19.598264 sleep for 13 seconds
# 2022-03-27 00:56:23.593074 finished (14 sec)
# 2022-03-27 00:56:29.605253 sleep for 6 seconds
# 2022-03-27 00:56:32.602354 finished (13 sec)
# 2022-03-27 00:56:35.608464 finished (6 sec)
```

출력 예시를 보면 이전 Job이 끝나기 전에 다음 작업이 시작되는 것을 볼 수 있다. 이러한 작업을 수행할 수 있는것은 asyncio.create_task()를 이용해서 비동기로 작업을 수행시켰기 때문이다.

### await run_job()을 사용하면 안되나?

main 함수에서 await로 run_job()을 수행시키면 Job이 끝날때까지 main 함수를 실행시키는 메인 스레드가 이벤트 루프에 실행권을 넘기지 않고 대기하게된다. 결국 Job이 10초를 초과해서 작업을 수행하면 그만큼 기다려야하기 때문에 10초에 한번씩 작업 실행을 못하게된다.

await는 내부적으로 두 가지 동작을한다. 첫 번째로 await 뒤의 코루틴(Awaitable 객체)을 이벤트 루프에 실행해달라고 등록한다. 두 번째로 등록한 코루틴 작업이 끝날 때까지 대기하며 실행권을 이벤트 루프에 넘긴다. 이벤트 루프는 코루틴 작업을 수행한 뒤에 실행권을 스레드에 넘긴다.

이벤트 루프는 여러 코루틴 사이에서 실행권을 주고 받으며 협력 멀티태스킹을 달성한다.

위 상황에서는 작업의 결과값이 필요없다. 이때 사용할 수 있는게 `asyncio.create_task()`이다. 해당 함수는 실행시키는 코루틴을 이벤트 루프에 등록하고 코루틴이 종료되었을 때 결과를 Future 객체로 반환한다. 반환되는 Future 객체 또한 Awaitable 객체이기 때문에 await를 붙여 코루틴 종료까지 기다릴 수 있다. 하지만 위 상황에서는 결과값이 필요없기 때문에 await를 붙이지 않은것이다.

### run_job()을 await 없이 사용하면 안되나?

await가 대기하는 명령을 내리기 때문에 `asyncio.create_task()` 대신 `run_job()`을 그대로 사용하면 안되나라고 생각할 수 있다. 하지만 실제로 그렇게 코드를 작성 후 실행하면 에러가 발생한다.

`RuntimeWarning: coroutine 'run_job' was never awaited`

이유는 run_job은 일반 함수가 아니라는 것이다. async로 작성된 함수는 코루틴 객체를 생성하여 반환할 뿐 실제로 내부 동작을 수행하지는 않는다. 코루틴 객체는 추후에 실행가능한 객체이다. 그리고 코루틴 객체는 이벤트 루프에 등록되지 않으면 동작하지 않는다. await는 코루틴 작업이 끝날때까지 대기하라는 작업도 수행하지만 async 함수를 이벤트 루프에 등록하는 작업도 수행한다.

             ┌──Coroutine
             │
Awaitable◄───┤
             │
             └──Future◄────────Task

## 동시 실행

이벤트 루프는 어떻게 동시 실행을 구현하는가? asyncio에는 공리가 존재한다. "스레드당 실행 중인 이벤트 루프는 하나"라는 제약조건이다. 즉, 아무리 많은 코루틴을 하나의 이벤트 루프에서 동시 실행 시켜도 싱글 스레드에서 동작한다. Cpython 기준으로 이벤트 루프 내부의 큐에 코루틴을 등록하고 한번에 하나씩 번갈아 실행한다.

              일반 함수                                   비동기 함수

     Caller            Function                 Caller            Coroutine
┌─────────────┐    ┌─────────────┐         ┌─────────────┐     ┌─────────────┐
│             │    │             │         │             │     │             │
│      │      │    │             │         │      │      │     │             │
│      │      │    │             │         │      │      │     │             │
│      ▼      │    │             │         │      ▼      │     │             │
│      │      │    │             │         │      │      │     │             │
│ call └──────┼────┼───────┐     │         │ call └──────┼─────┼──────┐      │
│             │    │       │     │         │             │     │      │      │
│             │    │       │     │         │             │     │      ▼      │
│             │    │       │     │         │             │     │      │      │
│             │    │       │     │         │      ┌──────┼─────┼──────┘      │
│             │    │       │     │         │      │      │     │      suspend│
│             │    │       │     │         │      ▼      │     │             │
│             │    │       │     │         │      │      │     │             │
│             │    │       │     │         │      └──────┼─────┼──────┐      │
│             │    │       │     │         │ resume      │     │      │      │
│             │    │       │     │         │             │     │      ▼      │
│             │    │       │     │         │             │     │      │      │
│             │    │       │     │         │      ┌──────┼─────┼──────┘      │
│             │    │       │     │         │      │      │     │      suspend│
│             │    │       │     │         │      ▼      │     │             │
│             │    │       │     │         │      │      │     │             │
│             │    │       │     │         │      └──────┼─────┼──────┐      │
│             │    │       │     │         │ resume      │     │      │      │
│             │    │       │     │         │             │     │      ▼      │
│             │    │       │     │         │             │     │      │      │
│      ┌──────┼────┼───────┘     │         │      ┌──────┼─────┼──────┘      │
│      │      │    │      return │         │      │      │     │      return │
│      │      │    │             │         │      │      │     │             │
│      ▼      │    │             │         │      ▼      │     │             │
│             │    │             │         │             │     │             │
└─────────────┘    └─────────────┘         └─────────────┘     └─────────────┘

일반 함수는 호출될 때 스택에 올라간 후 실행권을 가지고 코드를 동작시킨다. 수행이 끝나면 실행권을 호출한 곳으로 반납하고 스택에서 사라진다. 코루틴은 언제든 실행권 반납이 가능하다.

## 또 다른 동시 실행 방법

`gather` 혹은 `wait`와 같은 함수들을 사용할 수 있다. 하지만 해당 함수들은 기다릴 코루틴(혹은 Future)을 매개변수에 넣어줘야 하기 때문에 개수를 사전에 알고 있어야 한다.

## 이벤트 루프와 멀티 스레드

사실 항상 모든 코루틴이 하나의 이벤트 루프에서 실행되지 않는다. 스레드를 둘 이상으로 만들어서 작업을 시키면 스레드마다 이벤트 루프가 있을 수 있다. `run_coroutine_threadsafe`를 사용하면 코루틴을 어떤 스레드에서 실행시킬지 선택할 수 있고, `run_in_executor`를 사용하면 이벤트 루프를 하나 더 만들지 않더라도 다른 스레드에서 함수를 실행할 수 있다. 다른 스레드가 개입되고, 공유 자원을 스레드들이 사용하면 접근 제한을 고민해야 한다. asyncio 모듈에서 제공하는 Synchronization primitive 들은 thread-safe 하지 않기 때문이다.

## Future란?

Future는 Python에 국한된 개념이 아닌 비동기 프로그래밍에서 널리 사용되는 개념이다. 주로 비동기 연산의 결과를 저장하는 객체를 나타낸다. 비동기 함수의 경우 호출하는 곳에서 의도적으로 연산 종료를 기다리지 않으면 반환 값을 바로 받아볼 수 없다. 그래서 Future는 비동기 함수가 호출한 곳에 나중에 반환 값을 채워줄 목적으로 Future를 만들어 주는 것이다.

Python에는 서로 다른 두 Future 객체가 존재한다. 하나는 `concurrent.futures.Future`, 다른 하나는 `asyncio.Future`이다. 여기서 설명하는 Future는 후자이며 둘은 서로 다른 목적을 위해 만들어진 것이기에 관련이 없으며 호환도 되지 않는다.

## Future의 기능

- Awaitable이기 때문에 await 키워드로 결과를 대기할 수 있다.
- done()으로 연산이 끝났는지 확인 가능하다.
- set_result()로 연산의 결과를 지정 가능하다.
- cancel()로 연산을 취소 가능하다.
- add_done_callback()으로 콜백함수 등록이 가능하다.

## 참고

https://tech.buzzvil.com/blog/asyncio-no-1-coroutine-and-eventloop/
https://medium.com/delivus/understanding-pythons-asyncio-a-deep-dive-into-the-event-loop-89a6c5acbc84