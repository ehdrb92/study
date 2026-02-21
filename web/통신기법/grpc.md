## gRPC

gRPC는 Google이 개발하고 2016년 오픈소스로 공개한 고성능 RPC(Remote Procedure Call) 프레임워크입니다. "다른 서버의 함수를 마치 로컬 함수처럼 호출한다"는 개념을 현대적으로 구현한 것입니다.

### 핵심 개념

REST나 GraphQL이 "리소스" 또는 "데이터"를 중심으로 생각한다면, gRPC는 "함수 호출" 을 중심으로 생각합니다.

```
# REST 방식의 사고
POST /users/1/activate

# gRPC 방식의 사고
userService.ActivateUser(userId: 1)
```

gRPC는 두 가지 핵심 기술 위에서 동작합니다.

Protocol Buffers (Protobuf) 를 데이터 직렬화 포맷으로 사용하고, HTTP/2 를 전송 프로토콜로 사용합니다.

### Protocol Buffers

JSON 대신 Protobuf라는 바이너리 포맷을 사용합니다. .proto 파일로 데이터 구조와 서비스를 정의하면, 각 언어에 맞는 코드를 자동으로 생성해줍니다.

```protobuf
// user.proto
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

message GetUserRequest {
  int32 id = 1;
}

service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (Empty) returns (stream User);  // 서버 스트리밍
}
```

이 .proto 파일 하나로 Python, Go, Java, Node.js 등 다양한 언어의 클라이언트/서버 코드를 자동 생성할 수 있습니다.

```python
# 자동 생성된 코드를 활용한 Python 서버 예시
import grpc
from concurrent import futures
import user_pb2
import user_pb2_grpc

class UserServicer(user_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        # request.id로 유저 조회
        return user_pb2.User(id=request.id, name="홍길동", email="hong@example.com")

server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
user_pb2_grpc.add_UserServiceServicer_to_server(UserServicer(), server)
server.add_insecure_port("[::]:50051")
server.start()
```

### HTTP/2 기반의 4가지 통신 방식

gRPC는 HTTP/2 덕분에 REST에는 없는 다양한 통신 패턴을 지원합니다.

Unary RPC는 일반적인 요청-응답 방식이고, Server Streaming은 서버가 스트림으로 여러 응답을 보내는 방식입니다. Client Streaming은 클라이언트가 스트림으로 여러 요청을 보내고 하나의 응답을 받는 방식이며, Bidirectional Streaming은 양방향으로 동시에 스트림을 주고받는 방식입니다.

```protobuf
service ChatService {
  // Unary
  rpc GetMessage (MessageRequest) returns (Message);

  // Server Streaming
  rpc ListMessages (Empty) returns (stream Message);

  // Client Streaming
  rpc SendLogs (stream LogEntry) returns (Summary);

  // Bidirectional Streaming
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}
```

### REST/GraphQL과 성능 비교

| 항목 | REST (JSON) | gRPC (Protobuf) |
|------|------------|-----------------|
| 직렬화 | 텍스트 기반 | 바이너리 |
| 페이로드 크기 | 크다 | **~5~10배 작다** |
| 처리 속도 | 보통 | **~5~7배 빠르다** |
| 전송 프로토콜 | HTTP/1.1 | **HTTP/2** |
| 멀티플렉싱 | ❌ | ✅ |

### 장단점

장점으로는 뛰어난 성능과 낮은 레이턴시, 강타입 인터페이스로 인한 안정성, 다양한 언어 지원, 스트리밍 지원이 있습니다.

단점으로는 브라우저에서 직접 사용하기 어렵고 (gRPC-Web이라는 별도 레이어 필요), Protobuf가 바이너리라 사람이 직접 읽기 어려워 디버깅이 불편합니다. 또한 REST에 비해 초기 설정 비용이 높습니다.

### 주요 사용 사례

마이크로서비스 간 내부 통신에 가장 적합합니다. 서비스 간 대량의 데이터를 빠르게 주고받아야 하거나, 실시간 스트리밍이 필요한 경우에 특히 강점을 보입니다. Netflix, Uber, Dropbox 등이 내부 서비스 통신에 gRPC를 활용하고 있습니다.

### REST도 HTTP/2를 사용할 수 있다. 그럼에도 실질적인 성능 차이는 존재

REST는 **아키텍처 스타일**이고, HTTP/2는 **전송 프로토콜**이기 때문에 둘은 독립적입니다. REST API도 HTTP/2 위에서 충분히 동작할 수 있습니다.

**REST + HTTP/2를 쓰면** 멀티플렉싱, 헤더 압축, 서버 푸시 등 HTTP/2의 이점을 그대로 누릴 수 있습니다. 실제로 많은 현대 웹 서버(nginx, Apache 등)가 HTTP/2를 지원하고 있고, REST API에도 적용 가능합니다.

그럼에도 **gRPC가 성능 면에서 유리한 실질적인 이유**는 HTTP 버전보다는 **Protobuf vs JSON** 의 차이에 있습니다. JSON은 텍스트 기반이라 파싱 비용이 크고 페이로드도 크지만, Protobuf는 바이너리 직렬화라 크기도 작고 파싱도 훨씬 빠릅니다. 즉, 같은 HTTP/2를 쓰더라도 gRPC가 빠른 건 Protobuf 덕분인 경우가 큽니다.

또한 gRPC는 HTTP/2의 스트리밍 기능을 **프레임워크 수준에서 적극적으로 활용**하도록 설계되어 있는 반면, REST는 개발자가 직접 신경 써야 하는 부분이 많습니다.

요약하자면, 앞서 표의 "REST = HTTP/1.1" 비교는 일반적인 관행을 기준으로 한 것이었고, 정확한 차이는 **JSON vs Protobuf** 직렬화 포맷에 있다고 보는 게 맞습니다.