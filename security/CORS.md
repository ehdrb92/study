CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)

## 출처란?

출처(origin)는 프로토콜(http/https), 도메인, 포트 번호를 의미한다.

## CORS 탄생 배경

배경은 해커가 악성 스크립트(JavaScript)를 통해 다른 출처의 데이터를 가져오는 코드를 몰래 심어서 사용자가 사용하도록 유도하여 사용자의 데이터를 훔치는 것을 방지하기 위해서였다. 과거에는 웹 서비스를 하나의 서버에서 정적인 웹 페이지를 서빙하거나, 동적인 데이터 처리를 모두 담당하는 경우가 많았다. 이러한 상황에서 해커의 데이터 탈취를 막을 목적으로 웹 브라우저에 SOP(Same-Origin Policy, 동일 출처 정책)를 도입하여 현재 사용 중인 페이지의 출처와 다른 출처에 있는 데이터에 대한 요청을 보내려고 할 경우 이를 완전히 차단하였다.

하지만 현대의 웹 개발은 프론트엔드 서버와 백엔드 서버가 다른 출처에 존재하는 경우가 많다. 그래서 해커의 불순한 요청이 아님에도 출처가 다르다는 문제로 요청이 막히게되었다. 이러한 문제를 해결하기 위해 생긴 것이 CORS이다.

## CORS 동작 원리

CORS는 서버가 응답 헤더를 통해 브라우저에 "해당 출처의 요청은 허용한다"고 알려주는 방식이다.

브라우저는 단순 요청(GET, POST 등, 기본 헤더만 포함)이 아닌 경우, 먼저 OPTION 메서드로 `preflight` 요청을 서버에 보내어 현재 출처가 허용된 출처인지 확인한다. 서버가 올바른 CORS 헤더를 보내고 브라우저가 확인한 뒤 실제 요청을 서버에 보낸다.

대표적인 CORS 헤더는 다음과 같다.

- Access-Control-Allow-Origin: 어떤 출처(origin)를 허용할지 (* 또는 특정 도메인)
- Access-Control-Allow-Methods: 허용할 HTTP 메서드 (GET, POST, PUT 등)
- Access-Control-Allow-Headers: 허용할 요청 헤더 (예: Content-Type, Authorization)

### CORS가 동작하는 좀 더 추가적인 정보

CORS가 사용되는 방식은 두 가지 경우에 따라 나뉠 수 있다. 자바스크립트에서 보내는 요청이 "단순 요청"인지 아닌지이다.

단순 요청은 다음의 조건들에 모두 부합해야한다.

- 메서드: GET, HEAD, POST
- 헤더: CORS-safelisted headers 목록에 존재하는 헤더이고, 요청 헤더 중 커스턴 헤더가 하나라도 포함되면 안된다.
  - Accept
  - Accept-Language
  - Content-Language
  - Content-Type
  - Range
- Content-Type: 만약 포함되어 있다면 다음 세 기지 중 하나여야 한다.
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain
- Content-Type 헤더의 값은 charset 외에 다른 파라미터를 가질 수 없다.
  - Content-Type: text/plain; charset=utf-8 (조건 부합)
  - Content-Type: application/x-www-form-urlencoded; boundary=abc123 (조건 부합하지 않음)
- 자바스크립트 코드에서 추가된 헤더가 존재하는 순간 조건에 부합하지 않는다.

그렇다면 단순 요청일 때 CORS가 어떻게 동작하는가? 참고로 지금부터 설명하는 동작 순서는 당연히 동일 출처 요청이 아니라는 전제이다. 동일 출처이면 다음의 과정이 당연히 필요없다.

1. 브라우저는 요청이 단순 요청임을 확인한 뒤 실제 요청을 바로 서버에 전송한다.
2. 실제 응답을 받는다면, CORS 헤더를 확인한다.
3. 만약 CORS 헤더가 없거나, 현재 출처, 메서드 등이 조건에 부합하지 않는다면 CORS 에러를 발생시키며 응답을 읽지 않는다.
4. 조건에 부합한다면 받은 응답을 읽는다.

이번에는 단순 요청이 아닌 경우이다.

1. 브라우저는 요청이 단순 요청에 부합하지 않는 것을 확인한 뒤 실제 요청 이전에 Preflight 요청을 보낸다.
2. Preflight 응답으로 받은 CORS 헤더를 확인한다.
3. 만약 CORS 헤더가 없거나, 현재 출처, 메서드 등이 조건에 부합하지 않는다면 애초에 실제 요청을 보내지 않는다.
4. 조건에 부합한다면 실제 요청을 보내고 응답을 받아서 읽는다.