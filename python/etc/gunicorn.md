Gunicorn WSGI의 특징들에 대한 정리

## Pre-forking worker model

Pre-forking은 기본적으로 마스터가 각 요청을 처리하는 fork를 생성하는 방식을 의미한다. fork는 각각이 완전히 독립된 공간을 점유하는 프로세스이다. "pre"의 의미는 프로세스가 요청이 들어오기 전에 fork가 된다는 의미이다. 일반적으로 부하가 증가하거나 감소함에 따라 프로세스의 수를 늘리거나 줄일 수 있다. 이게 Gunicorn 서버가 자동으로 늘리거나 줄인다는 의미는 아니다.

pre-forking은 스레드 안전하지 않은 라이브러리 사용 시 유용하다. 또한 요청 내부의 문제가 발생하더라도 해당 요청을 처리하는 프로세스 범위 외부로는 문제가 전파되지 않는다.

## 워커 타입

- Sync Worker
    기본적으로 설정된 작업자 유형으로 한 번에 단일 요청을 처리하는 동기식 작업자 클래스이다. 이 모델은 오류가 발생하더라도 최대 한개의 요청에만 영향을 미치기 때문에 가장 단순하게 이해가 가능하다. 동기식 작업자는 지속적 연결을 지원하지 않는다. 각 연결은 응답이 전송된 후 닫힌다. 수동으로 Keep-Alive 또는 Connection: keep-alive 헤더를 추가하더라도 마찬가지이다.
- Async Worker
    비동기 작업자는 Greenlets(Eventlet 및 Gevent)를 기반으로 한다. Greenlets는 Python을 위한 협업형 다중 스레딩의 구현 방식이다. 일반적으로 응용 프로그램은 이러한 작업자 클래스를 변경 없이 사용할 수 있다. 완전한 Greenlet 지원을 위해서는 애플리케이션에 수정이 필요할 수 있다. 예를 들어 Gevent와 Psycopg를 사용할 경우 psycogreen이 설치되고 설정되어 있는지 확인해야한다.

## Gunicorn의 비동기 워커 vs 비동기 인터페이스 통합

Gunicorn에서 지원하는 비동기 워커 모델을 사용할 경우 웹 프레임워크에 베이스 코드 중 동기적으로 실행되는 코드 중 Gevent 혹은 Eventlet이 지원하는 모든 코드는 비동기로 전환되어 실행된다. 만약 지원되지 않는 코드는 별도로 에러를 발생하지는 않고 동기적으로 실행된다.

비동기 워커 모델을 사용하기 좋은 경우는 다음과 같다.

- 이미 거대한 레거시 코드를 비동기화 시켜야 할 경우
  - import 한 줄로 적용 가능하기 때문

사실 위 한 가지 경우 말고는 비동기 워커 모델의 장점은 없는 것으로 보인다. 그에 비해 단점은 다음과 같다.

- Monkey patching 작업으로 모든 코드를 검사함에 있어 CPU 오버헤드
- 더 많은 메모리 사용량
- Python 레벨에서 컨텍스트 스위칭을 하는 Greenlet은 C 레벨에서 최적화된 Native Async에 비해 무겁고 느리다.
- 문제가 생겼을 때 디버깅 시 스택 트레이스가 복잡하여 어려움이 따른다.

상황에 따라 혼합된 사용을 할 수 있지만 아래와 같은 상황에서 성능 저하가 생길 수 있다.

```python
@app.route('/bad-pattern')
def bad_pattern():
    # ❌ 나쁜 패턴: CPU 작업이 전체를 블로킹
    for i in range(10):
        requests.get('https://api.com/data')  # 비동기 ✓
        heavy_computation()  # 블로킹! ✗
    
    # 다른 모든 요청이 대기하게 됨

@app.route('/good-pattern')
def good_pattern():
    # ✅ 좋은 패턴: I/O 작업 먼저 처리
    # 1. 모든 I/O 작업 시작
    jobs = [gevent.spawn(requests.get, url) for url in urls]
    gevent.joinall(jobs)
    
    # 2. CPU 작업은 나중에
    results = [heavy_computation(job.value) for job in jobs]
    return results
```

생각보다 고려할 점이 많기 때문에 Greenlet도 사용이 그리 간단하다고 할 수는 없는 것으로 보인다.

그에 비해 Gunicorn + Uvicorn과 같이 차라리 Gunicorn은 프로세스의 관리만 수행하고 Uvicorn이 비동기 프로세스를 직접 동작시키도록 하는게 성능적으로나 장기적인 유지보수성으로 보았을 때 장점이 많다.

- Uvicorn은 Native Async를 사용하기 때문에 성능적으로 유리하다.
- 직접 비동기 컨텍스트를 작성해야하지만 개발자 입장에서 유지보수 시 비동기 구역이 눈에 직접 보이는게 더 나은 점이 많다.