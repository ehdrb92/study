FastAPI는 내부적으로 HTTP 통신 서버가 어떻게 구현되어 있을까?

FastAPI는 내부적으로 웹 부분은 starlette 데이터 부분은 pydantic을 사용한다.

## starlette

starlette의 소개를 보면 경량 ASGI 프레임워크로 파이썬을 이용해서 비동기 웹 서비스를 개발하는데 이상적이라고 한다.

- 경량에 복잡성이 적은 HTTP 웹 프레임워크
- 웹소켓 지원
- 백그라운드 작업 지원
- 서버 시작과 종료과정에서의 이벤트
- httpx를 이용한 테스트 클라이언트
- CORS, GZip, Static Files, Streaming 응답
- 세션과 쿠키 지원
- 100% 테스트 커버리지
- 100% 타입 어노테이션 소스 코드
- 강력한 의존성이 거의 없음
- asyncio 및 trio 백엔드와 호환
- 강력한 성능

stalette에서는 ASGI 서버로 uvicorn, dephne, hypercorn을 선택해서 사용할 수 있다고 한다.

## Pydantic

Python을 위한 데이터 검증 라이브러리이다. 가장 많이 사용되는 라이브러리이다.

Pydantic 공식 문서에서 소개하는 기능은 다음과 같다.

- 스키마 검증과 직렬화가 파이썬 내장 타입 주석으로 제어된다. 배우기 쉽고, 코드 작성이 적으며, IDE 및 정적 분석 도구와 원활하게 통합된다.
- Pydantic의 핵심 로직은 Rust로 작성되었다. 그래서 Python용 데이터 검증 라이브러리 중 가장 빠르다.
  Pydantic의 핵심 검증 로직은 "pydantic-core"라는 별도의 패키지에 구현되어 있으며, 해당 패키지는 대부분이 Rust로 구현되었다.
- Pydantic 모델은 JSON 구조를 생성할 수 있어 다른 도구와 통합이 용이하다.
- 엄격 모드를 통해 잘못된 데이터를 찾아낼 수 있고, 유연 모드는 데이터를 올바른 유형으로 강제 변환을 시도한다.
- 데이터 클래스와 TypedDict를 포함한 파이썬 표준 파이브러리 유형의 유효성 검사를 지원한다.
- 사용자 정의 검증기 및 직렬화를 통해 데이터 처리 방식을 다양하게 가져갈 수 있다.
- 이미 PyPi에 등록된 8000여개의 패키지가 Pydantic을 사용하고 있다. 큰 생태계를 구축하고 있다.

## FastAPI

FastAPI 공식 문서에서 소개하는 대표적인 기능

- API 문서 자동화 (Swagger UI)
- 다양한 코드 편지기 지원(VScode, Pycharm, ...)
- Pydantic에 의한 데이터 자동 검증
- 보안과 인증기능 통합(HTTP Basic, OAuth2, ...)
- 의존성 주입(Depends)
- Starlette 기능 (웹 서버 기능)
- Pydantic 기능 (데이터베이스 ORM)

FastAPI에서 제공하는 특별한 기능

- Exception Handler를 이용한 전역 예외처리 관리
  - https://fastapi.tiangolo.com/ko/tutorial/handling-errors/
- Middleware를 이용한 요청, 응답 가로채기
  - https://fastapi.tiangolo.com/ko/tutorial/middleware/
- APIRouter를 이용한 컨트롤러 관리
  - https://fastapi.tiangolo.com/ko/tutorial/bigger-applications/
- BackgroudTask를 이용한 가벼운 백그라운드 작업 처리
  - https://fastapi.tiangolo.com/ko/tutorial/background-tasks/
- 정적 파일 서빙
  - https://fastapi.tiangolo.com/ko/tutorial/static-files/
- Lifespan을 이용한 서버 준비
  - https://fastapi.tiangolo.com/ko/advanced/events/