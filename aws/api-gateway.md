API Gateway는 REST 및 WebSocket API를 생성, 배포, 유지, 모니터링하기 위한 서비스이다.

일반적으로는 다양한 AWS 클라우드 서비스를 스테이지별로 묶어서 배포하기 위한 목적으로 사용


## 기능

- 상태 저장(WebSocket) 및 상태 비저장(HTTP 및 REST) API에 대한 지원.
- 강력하고 유연한 인증 메커니즘(예: AWS Identity and Access Management 정책, Lambda 권한 부여자 함수, Amazon Cognito 사용자 풀 등).
- 변경 사항을 안전하게 롤아웃하기 위한 Canary 릴리스 배포.
- API 사용 및 API 변경에 대한 CloudTrail 로깅 및 모니터링.
- 경보 설정 기능을 포함한 CloudWatch 액세스 로깅 및 실행 로깅. 자세한 내용은 Amazon CloudWatch 지표를 사용한 REST API 실행 모니터링 및 CloudWatch 지표로 WebSocket API 실행 모니터링 단원을 참조하십시오.
- CloudFormation 템플릿을 사용하여 API 생성을 활성화할 수 있는 기능. 자세한 내용은 Amazon API Gateway 리소스 유형 참조 및 Amazon API Gateway V2 리소스 유형 참조를 참조하세요.
- 사용자 지정 도메인 이름 지원.
- 일반적인 웹 익스플로잇으로부터 API를 보호하기 위해 AWS WAF와 통합.
- 성능 지연 시간 파악 및 학습을 위해 AWS X-Ray와 통합.