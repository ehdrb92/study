API를 개발하고 통합 테스트 혹은 부하 테스트를 수행하려면 상황에 맞는 더미 데이터가 필요하다. 더미 데이터를 생성하는 좋은 방법은 뭐가 있을까?

## 더미 데이터 생성 라이브러리

직접 스크립트를 작성하는 대신 전용 라이브러리를 활용하면 훨씬 빠르게 다양한 데이터를 만들 수 있다.

Faker + factory_boy (Python 생태계 표준에 가까운 조합)

Faker는 이름, 주소, 이메일, 날짜 등 현실적인 가짜 데이터를 생성하고, factory_boy는 모델 단위로 팩토리 클래스를 정의해 객체를 일괄 생성하는 구조를 제공한다.

```python
import factory
from faker import Faker

fake = Faker("ko_KR")  # 한국어 로케일 지원

class UserFactory(factory.Factory):
    class Meta:
        model = dict  # 실제 모델 클래스로 교체 가능

    name = factory.Faker("name", locale="ko_KR")
    email = factory.LazyAttribute(lambda o: f"{fake.user_name()}@example.com")
    age = factory.Faker("random_int", min=18, max=65)
    phone = factory.Faker("phone_number", locale="ko_KR")

# 한 번에 100개 생성
users = UserFactory.create_batch(100)
```

완전히 무작위한 데이터를 사용하면 테스트 재현이 어려워지는 문제가 있는데, factory_boy는 reseed_random()으로 시드를 고정해 동일한 결과를 재현할 수 있게 해준다.

관계(연관) 모델도 SubFactory로 자동 생성할 수 있어서, 예를 들어 Order를 만들 때 연결된 User를 자동으로 함께 생성하는 식으로 쓸 수 있다.

Mimesis — 대용량이 필요할 때 Faker보다 훨씬 빠른 성능을 내는 대안 라이브러리이다. 대규모 데이터셋 생성이나 성능이 중요한 경우 mimesis가 더 적합하며, factory_boy와의 통합도 지원한다.

## 부하 테스트 도구 내 데이터 생성

부하 테스트는 데이터 생성과 요청 전송을 같이 처리해야 하는데, 많이 쓰이는 조합은 다음과 같다.

Locust (Python)

Python으로 테스트 시나리오를 직접 작성할 수 있어 데이터 생성 로직을 함께 넣기 쉽고, 웹 UI로 실시간 모니터링도 가능하다.

```python
from locust import HttpUser, task, between
from faker import Faker

fake = Faker("ko_KR")

class APIUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def create_order(self):
        payload = {
            "customer_name": fake.name(),
            "product_id": fake.random_int(min=1, max=100),
            "quantity": fake.random_int(min=1, max=10),
        }
        self.client.post("/api/orders", json=payload)
```

k6 (JavaScript) — CI/CD 파이프라인과 통합하기 좋고, CSV 파일로 동적 데이터셋을 불러와 요청에 활용할 수 있다.

Artillery — YAML로 시나리오를 작성하고 JavaScript 훅으로 동적 데이터를 삽입하는 방식으로, 설정이 비교적 단순하다.

## OpenAPI 스펙 기반 자동 생성

API 스펙(OpenAPI/Swagger)이 있다면 이를 활용해 데이터를 자동으로 생성하는 방식도 있다.

```python
# schemathesis: OpenAPI 스펙에서 자동으로 테스트 케이스 생성
import schemathesis

schema = schemathesis.from_uri("http://localhost:8000/openapi.json")

@schema.parametrize()
def test_api(case):
    case.call_and_validate()
```

`schemathesis`나 `hypothesis`를 이용하면 스펙에 정의된 타입/제약조건에 맞는 다양한 엣지 케이스 데이터를 자동 생성해준다. 직접 케이스를 떠올리기 힘든 경계값이나 예외 상황을 발견하는 데 특히 유용하다.

## 요약

- 통합 테스트용 시드 데이터: Faker + factory_boy
- 대용량 데이터 빠르게 생성: mimesis
- 부하 테스트 (Python 친화): Locust + Faker
- CI/CD 연동 부하 테스트: k6
- 스펙 기반 자동 테스트: schemathesis