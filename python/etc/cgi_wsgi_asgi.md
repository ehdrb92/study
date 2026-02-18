## Web Server

웹 서버란 HTTP 요청에 맞는 웹 페이지(HTML)를 응답해주는 목적으로 존재한다. 웹페이지의 내부 요소가 변하지 않는 정적인(static) 페이지를 응답해주는 방식이다.

`Client <-> WebServer <-> Database`

## Web Application Server

웹 애플리케이션 서버란 웹 서버의 뒤에 존재하며 사용자의 요청에 따라 동적으로 데이터를 생성하여 웹 페이지를 만들어주는 서버를 말한다.

`Client <-> Web Server <-> Web Application Server(Web Server + CGI) <-> Database`

보통 두 서버를 혼합하여 사용하며 웹 서버를 앞단에 두어 정적인 데이터의 응답을 수행하며 복잡한 비즈니스 로직 및 동적인 처리가 필요할 경우 이를 웹 애플리케이션 서버에서 수행한다. 웹 애플리케이션 서버는 정적, 동적인 처리를 모두 할 수 있기 때문에 하나만 두는게 구조상 간편하고 관리가 편해지긴 하겠지만 부하가 커져 성능 저하가 생길 수 있기 때문에 두 서버를 모두 이용하는게 일반적이다.

일반적으로 Django, FastAPI, Spring Boot와 같은 것들이 웹 애플리케이션 서버이다.

## CGI (Common Gateway Interface)

CGI는 웹 서버가 외부 프로그램을 실행하여 동적인 웹 페이지를 생성할 수 있게 하는 인터페이스 표준이라고 한다. 웹 서버와 외부 프로그램 사이의 데이터 전송을 위한 프로토콜이다. 동작 방식은 다음과 같다.

1. 요청 수신: 클라이언트(브라우저)가 CGI 스크립트 URL로 HTTP 요청을 보냄
2. 프로세스 생성: 웹 서버가 새로운 프로세스를 생성하여 CGI 프로그램 실행
3. 환경 변수 설정: 요청 정보(HTTP 헤더, 쿼리 스트링 등)를 환경 변수로 전달
4. 프로그램 실행: CGI 프로그램이 실행되어 동적 콘텐츠 생성
5. 응답 반환: 생성된 결과를 표준 출력(stdout)으로 웹 서버에 전달
6. 클라이언트 전송: 웹 서버가 결과를 HTTP 응답으로 클라이언트에 전송

CGI는 언어 독립적(C, Perl, Python, Shell Script 등)이고 각 요청이 독립적인 프로세스 실행으로 동작한다. 단순하고 안정적이지만 요청마다 프로세스가 존재하기에 메모리 사용량이 많고 성능 오버헤드가 심하다. 현재는 거의 사용되지 않는 방식이다. 대안으로 웹 애플리케이션 서버가 사용된다.

## WSGI(Web Server Gateway Interface)

`Client <-> Web Server <-> WSGI Middleware <-> Web Framework(Django, FastAPI, ...) <-> Database`

웹 서버와 프레임워크 사이에서 요청과 응답을 중계하는 역할을 한다. 웹 서버의 요청은 WSGI에 의해서 Callable Object(Function, Object) 형태로 웹 프레임워크에 전달된다. WSGI Middleware(Server)는 WSGI Interface의 구현체로 gunicorn, uWSGI 등이 존재한다. WSGI는 동기로 동작하기 때문에 동시에 많은 요청을 처리하는데 한계가 존재한다. (Celery를 사용하여 성능 향상 가능)

- 2003년 PEP 333으로 제안, PEP 3333으로 Python 3 지원
- 한 번에 하나의 요청만 처리 (요청-응답 사이클이 완료될 때까지 대기)
- HTTP/1.1만 지원
- 구현이 단순하고 안정적

## ASGI(Asynchronous Server Gateway Interface)

`Client <-> Web Server <-> WSGI Middleware <-> ASGI Server <-> Web Framework(Django, FastAPI, ...) <-> Database`

WSGI와 유사한 구조를 가지지만 기본적으로 모든 요청을 비동기로 처리한다. ASGI는 WSGI와 호환이 되기 때문에 혼합하여 사용하는 경우가 많이 존재한다.

- 2016년 등장, Django Channels 프로젝트에서 시작
- 비동기 I/O 지원 (async/await 문법 활용)
- HTTP/2, WebSocket, Server-Sent Events 등 지원
- 동시에 여러 요청 처리 가능
- WSGI 애플리케이션도 실행 가능 (하위 호환성)

## WSGI vs ASGI 뭘 사용해야 할까?

사실 일반적으로 ASGI를 선택하는 것이 합리적이다. 대부분의 웹 서버는 데이터베이스 통신, 외부 API 연동 등이 많아지기 때문에 CPU의 유휴 시간이 늘어나기 때문에 비동기 방식이 성능상 이점을 많이 가져갈 수 있다. 그리고 WSGI에서는 지원하지 않는 HTTP/2, WebSocket 등의 현대적인 기술을 ASGI는 지원한다.

그럼에도 WSGI를 선택한다면 어떤 이유가 있는가?

첫 번째는 단순함이 우선인 상황이다. 비동기 방식의 경우 코드가 실행되는 중 입출력 작업이 수행될 경우 일시중지되고 다른 코루틴으로 실행권이 넘어간다. 해당 상황에서 리소스의 안전한 획득과 헤제를 위해 컨텍스트 매니저 내부에서 수행되는 것이 대부분이다. 그렇기 때문에 코드 작성 자체에 복잡함과 어려움이 따를 수 있다. 그리고 외부 라이브러리를 사용할 시 비동기 컨텍스트를 지원하지 않으면 비동기 함수 내부에서 동작하더라도 의미가 없어지게 된다. 두 번째는 CPU 집약적인 작업이 대부분인 경우이다. 이때도 사실 비동기 방식 자체가 통하지 않는다. 세 번째로 검증된 안정성이 중요할 때이다. 금융, 의료 등 비즈니스 로직의 정확성이 매우 중요한 시스템이거나 다운타임이 큰 손실로 이어지는 서비스의 경우 비동기가 올바른 선택이 아닐 수 있다.

여기서 검증된 안정성 부분의 경우 비동기 자체가 다운타임의 손실을 일으킨다는 것은 아니다. WSGI의 경우 20년 이상의 검증된 패턴을 사용하지만 ASGI는 상대적으로 새로운 기술이다. 그렇기 때문에 발견되지 않은 엣지 케이스의 존재와 비동기 특유의 복잡한 버그 패턴, 디버깅과 프로파일링의 어려움 등이 존재할 수 밖에 없다. 그렇기 때문에 상황에 따라 WSGI 사용이 유리한 경우도 있다.


## Gunicorn vs Uvicorn 워커 프로세스 관리 차이점

- Gunicorn
  - Pre-fork 워커 모델: 마스터 프로세스가 워커들을 미리 fork하여 효율적인 리소스 공유
  - 자동 워커 재시작: 메모리 누수 방지를 위한 max-requests 옵션
  - 유연한 워커 타입: sync, async, tornado, eventlet 등 다양한 워커 클래스 지원
  - 상세한 워커 모니터링: 워커 상태 추적, 타임아웃 관리, 좀비 프로세스 처리
  - Graceful reload: 무중단 재시작 지원
  - 동적 워커 조정: 부하에 따른 워커 수 조정
- Uvicorn
  - 기본적인 멀티프로세싱만 지원
  - 워커 재시작이나 모니터링 기능이 제한적
  - 프로세스 간 통신이나 상태 공유 메커니즘이 단순함
  - 에러 복구나 fault tolerance 기능이 상대적으로 부족

일반적으로 배포 환경별로 다음과 같은 선택 가이드가 존재한다.

- 단일 서버 컴퓨터에서 배포할 경우: Gunicorn + Uvicorn 조합 사용 (Gunicorn이 프로세스 관리, Uvicorn을 비동기 워커 클래스로 사용)
- 다중의 서버 컴퓨터에서 배포할 경우: Kubernetes + Docker의 컨테이너마다 단일 Uvicorn 사용
- 개발 환경: 단일 Uvicorn 사용

## WSGI 비교

- Gunicorn
  - 설정이 간단하기 사용하기 쉽다.
  - Pre-fork worker 모델로 안정성이 높다.
  - 다양한 worker 타입을 지원한다. (sync, async, gevent, eventlet)
  - Python으로 작성되어 디버깅이 쉽다.
  - 윈도우를 지원하지 않는다.
  - 정적 파일을 서빙하지 못해 앞 단에 Nginx와 같은 웹 서버 필요
  - 동기 worker 사용 시 동시 연결 수 제한
- uWSGI
  - 매우 다양한 기능과 설정 옵션
  - 정적 파일 서빙, 캐싱, 로드 밸런싱 등 내장
  - 다양한 프로토콜 지원 (HTTP, uwsgi, FastCGI)
  - 뛰어난 성능과 메모리 효율성
  - 설정이 복잡하고 학습 곡선이 가파름
- Waitress
  - Pure Python으로 윈도우 포함 모든 플랫폼 지원
  - 안정성과 가벼움
  - 기능이 제한적
  - Gunicorn 혹은 uWSGI에 비해 낮은 성능
  - worker 프로세스 모델 미지원

## WSGI 무엇을 선택할까?

일단 Waitress는 윈도우 배포가 아닌 상황에서는 사용할 일이 없다. Gunicorn, uWSGI에 비해서 기능이 너무 제한적이다. Gunicorn vs uWSGI 뭘 선택할까? 여기서 Gunicorn을 선택하는 것이 일반적으로 좋다. Gunicorn은 python으로 작성되었다. 그렇기에 Python을 이용해서 웹 서비스를 개발하는 상황에서 Gunicorn에서 발생한 문제에 대한 디버깅이 상대적으로 쉬울 수 있다. 또한 uWSGI에서 지원하지만 Gunicorn에서 지원하지 않는 다양한 기능은 Gunicorn + Nginx 조합을 사용하여 커버할 수 있다. Nginx가 경량 웹 서버로서 uWSGI에서 제공하는 기능들을 제공하며 Gunicorn + Nginx로 나뉘면 책임 분리가 되기 때문에 유지 보수에 장점 또한 생긴다.

uWSGI가 Gunicorn에 비해 성능이 좋은 것은 사실이다. uWSGI가 성능이 더 좋은 대표적인 이유는 다음과 같다.

- C언어로 구현되어 시스템 콜을 직접 호출, 낮은 수준의 최적화가 있다. 그에 비해 Gunicorn의 Python 구현체이기에 GIL 영향, 인터프리터 오버헤드 영향이 있다.
- 프로토콜 처리 시 Gunicorn은 Python 레벨에서 HTTP를 파싱 처리한다. uWSGI는 uwsgi 프로토콜을 이용해 C 레벨에서 바이너리 프로토콜 파싱을 한다.
- uWSGI는 메모리 풀링(C 코드), 미리 할당된 버퍼 재사용을 수행한다. Gunicorn은 Python의 가비지 컬렉터에 의존하여 메모리 할당/헤제의 오버헤드가 크다.

위와 같은 성능 차이가 있음에도 Gunicorn이 더 인기있는 이유는 무엇인가? 그것은 성능 차이가 있다고 한들 굉장히 미세한 단위에서 일어나기 때문이다. 일반적인 웹 서비스에서는 데이터베이스 쿼리, 외부 API 호출, 비즈니스 로직 등 내부적으로 많은 동작이 함께 수반된다. 웹 서버 자체의 성능 차이는 다른 요인들에 비하면 굉장히 낮은 수준이기 때문에 실무의 생산성에 비교했을 때 웹 서버 교체로 얻는 최적화 효과에 비해 투입되는 자원이 너무 많기 때문에 사실 해당 요인까지 신경쓰지 않는다. 대체로 다음과 같은 최적화를 우선순위로 한다.

1. 데이터베이스 쿼리 최적화 (가장 큰 영향)
2. 캐싱 전략
3. 비동기 처리
4. 웹 서버 선택

실제 영향도:
- DB 최적화: 10x ~ 100x 개선 가능
- 캐싱: 10x ~ 50x 개선 가능
- 비동기 처리: 2x ~ 10x 개선 가능
- 웹 서버 변경: 1.1x ~ 1.3x 개선

위와 같이 병목이 생긴다면 웹 서버 문제는 아닐 가능성이 매우 크다는 것이다. 마이크로 초 단위의 아주 미세한 성능 최적화가 필요하거나 단일 서버에서 최대 성능을 뽑아야 하는경우가 아니라면 차라리 다른 요인에 대한 최적화 방법을 고민하고 수평 확장으로 서버를 늘리는게 비용적으로 더 낮게 투입될 수 있다는 것이다.

## ASGI 비교

- Uvicorn
  - 비동기 프로그래밍에 대한 지원이 좋다. FastAPI에서 공식적으로 권장하는 ASGI 서버
  - WebSocket 지원
  - uvloop와 httptools가 설치되어있다면, 기본적으로 사용하여 고성능 네트워크 제공
  - 사용하기 쉽다.
  - 개발 단계에서 간단하게 사용하기 좋다.
  - 소규모 프로젝트 배포에 효과적이다.
- Hypercorn
  - HTTP/2, HTTP/3 지원
  - WebSocket 지원
  - 다양한 이벤트 루프 (asyncio, uvloop, trio)
  - Uvicorn보다 느린 성능
- Daphne
  - Django Channels 프로젝트에서 개발 (Django와 뛰어난 호환)
  - WebSocket 또는 real-time 통신(long-lived connection)에 효과적
  - 채팅 앱, 알림 시스템 그 외 영구적인 연결이 필요한 곳이 효과적

## Uvicorn vs Hypercorn vs Daphne 성능 차이 왜 발생할까?

https://medium.com/@onegreyonewhite/2024-comparing-asgi-servers-uvicorn-hypercorn-and-daphne-addb2fd70c57

위 문서에서는 uvicorn, hypercorn, daphne의 성능을 비교하는 테스트를 진행했다. 결과적으로 uvicorn이 압도적으로 좋은 성능을 가지고 있으며, hypercorn과 daphne이 유사한 성능을 보였다. 해당 테스트는 "Hello, World!"를 응답하는 간단한 API에 대한 테스트 였으며, 내부적으로 데이터 가져오기 작업이 없으며 단순한 응답의 비동기 처리만 수행했기 때문에 상황에 따라 다른 결과가 나올 수 있음을 주의하였다.

사실 uvicorn과 hypercorn은 동일하게 HTTP 파싱에 h11을 사용하며 uvloop를 내장하고 있다. 내부적으로 거의 동일할 것인데 왜 성능 차이가 극명하게 드러날까? 가장 유력한 원인으로 hypercorn은 다양한 고급 기능을 제공한다. 이러한 무거운 내장 기능들이 성능 오버헤드에 영향을 끼쳤을 가능성이 있다. 그리고 uvicorn의 공식 문서에서는 uvicorn이 지원 가능한 경우에 httptools를 이용해 HTTP 파싱을 수행한다고 되어있다. httptools는 C로 작성된 도구로 python으로 작성된 h11에 비해 좋은 성능을 가진다.