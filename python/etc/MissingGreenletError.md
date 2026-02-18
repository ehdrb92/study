https://medium.com/@vickypalaniappan12/sqlalchemy-missinggreenleterror-656825b3ce13

MissingGreenletError는 SQLAlchemy를 asyncio나 다른 비동기 프레임워크와 함께 사용할 때 자주 발생하는 문제이다. SQLAlchemy는 기본적으로 동기 방식이기 때문에 발생하며, 이는 데이터베이스 작업이 결과 대기 중 이벤트 루프를 차단하기 때문이다.

오류의 근본 원인을 알기 위해서는 SQLAlchemy와 asyncio가 어떻게 함께 작동하는지 이해해야 한다.

async 함수 내에서 동기식 SQLAlchemy 함수나 메서드를 호출하면, 이벤트 루프는 데이터베이스 작업이 완료될 때까지 다른 작업으로 전환할 수 없다. 이는 이벤트 루프가 차단되어 애플리케이션이 응답하지 않거나 성능 문제가 발생할 수 있음을 의미한다.

관계가 맺어진 두 클래스가 존재하고 참조를 통해 다른 테이블 데이터에 접근할 때 다음 에러가 발생한다.

`sqlalchemy.exc.MissingGreenlet: greenlet_spawn has not been called; can't call await_only() here. Was IO attempted in an unexpected place? (Background on this error at: https://sqlalche.me/e/20/xd2s)`

SQLAlchemy는 참조 객체에 접근할 때 동기식으로 지연 로딩(lazy loading)한다. SQLAlchemy는 동기식 작업을 수행할 때 greenlet을 사용하는데, 비동기 컨텍스트에서는 greenlet이 존재하지 않는다.

MissingGreenletError를 피하기 위해서는 두 가지 방식을 사용할 수 있다.

1. 즉시 로딩(eager loading)을 통해 초기 쿼리 할때 관계 객체 데이터를 함께 가져온다.
2. AsyncAttrs를 사용한다.

selectin, joinload 등 즉시 로딩 방식을 사용하면 쿼리할 때 이미 관계 객체 데이터를 함께 가져오기 때문에 동기식 참조를 하지 않아 에러가 발생하지 않는다.

AsyncAttrs는 SQLAlchemy에서 제공하는 내장 믹스인으로, awaitable_attrs를 도입하여 awaitable로 처리할 수 있도록 한다. 이를 통해 참조 객체 접근 시 비동기 로딩 및 접근을 가능하게 해준다.

AsyncAttrs는 비동기식 처리를 가능하게 하지만 N+1 문제는 가지고 간다. 그렇기 때문에 지연 로딩이 이점을 가지는 상황이 아니라면 즉시 로딩 방식을 사용하는 것이 좋다.