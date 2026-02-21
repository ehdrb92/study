## GraphQL

GraphQL은 Facebook(현 Meta)이 2012년 내부적으로 개발하고 2015년 오픈소스로 공개한 쿼리 언어이자 런타임입니다. REST API의 Over-fetching / Under-fetching 문제를 해결하기 위해 탄생했습니다.

### 핵심 개념

REST가 "URL + HTTP 메서드"로 요청을 정의하는 반면, GraphQL은 클라이언트가 원하는 데이터의 구조를 직접 명시합니다. 엔드포인트는 보통 `/graphql` 하나만 존재합니다.

```graphql
# 클라이언트가 원하는 필드만 골라서 요청
query {
  user(id: "1") {
    name
    email
    posts {
      title
      createdAt
    }
  }
}
```

### 세 가지 작업 타입

GraphQL에는 세 가지 작업 타입이 있습니다.

Query는 데이터 조회, Mutation은 데이터 변경(생성/수정/삭제), Subscription은 실시간 데이터 구독입니다.

```graphql
# Mutation — 데이터 생성
mutation {
  createUser(name: "홍길동", email: "hong@example.com") {
    id
    name
  }
}

# Subscription — 실시간 구독
subscription {
  messageAdded(roomId: "42") {
    content
    sender
  }
}
```

### 스키마 기반 설계

GraphQL은 스키마(Schema) 를 통해 API의 타입과 구조를 명확하게 정의합니다. 이 스키마가 클라이언트와 서버 간의 "계약" 역할을 합니다.

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post]
}

type Post {
  id: ID!
  title: String!
  createdAt: String!
}

type Query {
  user(id: ID!): User
  users: [User]
}
```

스키마만 보면 어떤 데이터를 요청할 수 있는지 바로 알 수 있어서, 자체 문서화(Self-documenting) 효과도 있습니다.

### 장단점

장점으로는 필요한 데이터만 정확히 받아올 수 있어 네트워크 효율이 높고, 단일 엔드포인트로 다양한 요청을 처리할 수 있습니다. 또한 강타입 스키마 덕분에 프론트엔드-백엔드 간 협업이 수월하고, GraphiQL 같은 도구로 API 탐색도 편리합니다.

단점으로는 REST보다 초기 설정과 학습 비용이 높고, 캐싱이 복잡합니다 (HTTP 캐싱을 그대로 쓰기 어렵고 별도 전략이 필요). 또한 잘못 설계하면 서버에서 N+1 문제가 발생할 수 있어 DataLoader 같은 도구로 별도 최적화가 필요합니다.

### 주요 사용 사례

여러 클라이언트(웹, 모바일, 태블릿)가 각자 다른 형태의 데이터를 필요로 할 때 특히 강력합니다. Meta, GitHub, Shopify, Twitter 등이 GraphQL을 사용하고 있으며, 특히 BFF(Backend For Frontend) 패턴과 잘 어울립니다.
