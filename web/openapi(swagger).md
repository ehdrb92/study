Open API(OpenAPI Specification, OAS)란 RESTful API의 구조와 기능을 표준화된 방식으로 기술(describe)하기 위한 명세(Specification)이다.

"특정 API는 어떤 주소(endpoint)로 접근하고, 어떤 데이터를 보내야 하며, 어떤 응답이 돌아오는가"를 사람과 컴퓨터 모두가 읽을 수 있는 형식(YAML 또는 JSON)으로 정리한 설계 문거이다. 이전에는 Swagger하는 이름으로 불렸고, 현재는 Linux Foundation 산하의 OpenAPI Initiative에서 관리한다.

## Open API는 왜 필요할까?

API를 만들면, 이걸 사용하는 다른 개발자가 이해하거나 먼 미래에 자신이 다시봐도 이해할 수 있어야한다.

- 어떤 URL로 요청을 보내야 하나?
- GET인가, POST인가?
- 어떤 파라미터가 필요하고, 타입은?
- 응답 데이터의 구조는?
- 인증은 어떻게?

이걸 말로 설명하거나, 따로 Word 문서로 작성하면 관리가 어렵고 최신 상태를 유지하기 힘들다. OpenAPI를 쓰면 이 모든 내용을 하나의 표준 파일로 관리할 수 있다.

## OpenAPI 파일의 기본 구조

```
# 1. 기본 영역 (버전, 서버 정보, API 설명)
openapi: 3.0.3
info:
  title: My Simple API
  version: 1.0.0
servers:
  - url: https://api.example.com/v1

# 2. tags 영역 (API 그룹 분류)
tags:
  - name: users
    description: 사용자 관련 API

# 3. paths 영역 (엔드포인트 정의)
paths:
  /users/{id}:
    get:
      tags:
        - users
      summary: 사용자 조회
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

# 4. components 영역 (재사용 가능한 스키마 정의)
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

## OpenAPI의 주요 장점

OpenAPI 명세가 있으면 문서화, 테스트 케이스, 클라이언트 라이브러리를 도구로 자동 생성할 수 있다.

1. 자동 문서화 — Swagger UI 같은 도구를 이용해 OpenAPI 파일 하나로 인터랙티브한 API 문서를 자동 생성할 수 있다.
2. 코드 자동 생성 — OpenAPI Generator를 통해 서버/클라이언트 코드를 여러 언어로 자동 생성할 수 있다.
3. 테스트 자동화 — API 명세를 기반으로 요청/응답 유효성 검사를 자동화할 수 있다.
4. 언어 독립적 — 특정 개발 언어에 구애받지 않고 사용할 수 있다.
