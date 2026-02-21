## REST API

REST (Representational State Transfer) 는 2000년 Roy Fielding의 논문에서 소개된 아키텍처 스타일로, 현재 웹 서비스에서 가장 널리 사용되는 통신 방식

### 핵심 개념

REST는 리소스(Resource) 중심으로 설계됩니다. 모든 것을 "자원"으로 보고, URL로 자원을 표현하며 HTTP 메서드로 행위를 나타낸다.

```
GET    /users        → 사용자 목록 조회
POST   /users        → 사용자 생성
GET    /users/1      → 특정 사용자 조회
PUT    /users/1      → 특정 사용자 전체 수정
PATCH  /users/1      → 특정 사용자 일부 수정
DELETE /users/1      → 특정 사용자 삭제
```

### 특징

Stateless (무상태성) 가 핵심 원칙입니다. 서버는 클라이언트의 상태를 저장하지 않으며, 각 요청은 독립적으로 처리됩니다. 덕분에 서버 확장(Scale-out)이 쉽습니다.

그 외에도 Cacheable (HTTP 캐싱 활용 가능), Uniform Interface (일관된 인터페이스), Layered System (중간 계층 구성 가능) 등의 원칙을 따릅니다.

### 장단점

장점으로는 학습 곡선이 낮고, HTTP 표준을 그대로 활용하기 때문에 브라우저, 모바일 등 모든 플랫폼에서 쉽게 사용할 수 있습니다. 또한 생태계가 매우 성숙해 있어 관련 도구나 라이브러리가 풍부합니다.

단점으로는 Over-fetching (필요한 것보다 많은 데이터를 받음)과 Under-fetching (필요한 데이터를 얻기 위해 여러 번 요청해야 함) 문제가 있습니다. 이 문제를 해결하기 위해 나온 것이 바로 다음에 소개할 GraphQL입니다.

### 주요 사용 사례

일반적인 웹/모바일 서비스의 CRUD API, 공개 API (Open API), 마이크로서비스 간 통신 등 거의 모든 범용적인 서비스에서 사용됩니다. GitHub, Twitter, Kakao 등 대부분의 공개 API가 REST 방식을 채택하고 있습니다.