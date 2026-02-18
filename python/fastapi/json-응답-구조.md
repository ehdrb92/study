FastAPI는 JSON 응답을 반환할 때 기본적으로 JSONResponse라는 데이터 구조를 사용한다. 하지만 이외에도 별도 설치를 통해 외부 라이브러리의 json 데이터 응답 구조를 사용할 수 있다. ORJSONResponse과 UJSONResponse이다. 이 셋을 비교해보자.

## JSONResponse

표준 Python의 json 모듈(json.dumps)을 사용하여 직렬화한다. Starlette에서 제공하는 클래스로, FastAPI가 별도 설치 없이 기본으로 사용한다.

- 별도의 설치 불필요
- 호환성이 가장 좋음
- 성능은 가장 낮음
- jsonable_encoder를 통해 Pydantic 모델, datetime 등을 자동 변환해줌

## ORJSONResponse

Rust로 작성된 orjson 라이브러리를 기반으로 하며, AVX2/SIMD를 활용한 병렬 처리로 빠른 직렬화를 제공한다. 2024년 벤치마크 기준 약 1.1 GB/s의 직렬화 속도를 보인다.

- orjson 별도 설치 필요 — pip install orjson 또는 pip install "fastapi[all]"
- 세 가지 중 가장 빠른 성능
- orjson.dumps가 bytes를 직접 반환하므로 str → bytes 변환 오버헤드 없음
- datetime, UUID, numpy 배열 등을 추가 변환 없이 직접 직렬화 가능

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse

app = FastAPI(default_response_class=ORJSONResponse)

@app.get("/items/")
def read_items():
    return {"item": "foo"}
```

## UJSONResponse

C 기반의 ujson 라이브러리를 사용하며, 2024년 벤치마크 기준 약 600 MB/s의 직렬화 속도를 보인다.

- ujson 별도 설치 필요 — pip install ujson
- JSONResponse보다 빠르나 ORJSONResponse보다는 느림
- ensure_ascii=False로 직렬화하여 non-ASCII 문자를 그대로 출력
- orjson에 비해 옵션 플래그가 제한적
- 요청(request) 파싱은 Starlette이 담당하므로, 응답 클래스 변경이 요청 파싱 성능에는 영향을 주지 않는 점에 주의

```python
from fastapi import FastAPI
from fastapi.responses import UJSONResponse

app = FastAPI(default_response_class=UJSONResponse)

@app.get("/items/")
def read_items():
    return {"item": "foo"}
```

## 결론

호환성이 중요하다면 기본값인 `JSONResponse`를 사용. 성능이 중요하다면 `ORJSONResponse`, `UJSONResponse` 중 하나를 택하면 되는데, 사실 UJSONResponse이 ORJSONResponse 대비 우위에 있는 점이 없으므로 ORJSONResponse을 사용하는 것이 권장된다.