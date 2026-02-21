AWS SAM(Serverless Application Model)은 무엇인가?

이것은 AWS에서 만든 오픈소스 프레임워크로, 서버리스 애플리케이션을 코드로 인프라를 정의하고 배포하기 위한 도구라고 한다. 두 가지 핵심 구성요소로 이루어져 있다. 하나는 AWS SAM 템플릿(template.yaml)이다. CloudFormation의 확장판이며 Lambda 함수, API Gateway, DynamoDB 등의 서버리스 리소스를 간결한 문법으로 정의한다. 다른 하나는 AWS SAM CLI이다. 로컬 테스트, 빌드, 배포 등을 명령어로 수행할 수 있는 개발자 도구이다.

CloudFormation? 이것은 AWS 인프라를 YAML/JSON 파일로 정의하고 배포하는 서비스이다. 모든 AWS 리소스를 지원하지만 서버리스 리소스를 정의하는 방법이 너무 장황하여 사용하기가 어렵다.

SAM은 CloudFormation의 상위 레이어로 서버리스 개발에 특화된 간결한 단축 문법을 제공하고, 배포 시 템플릿을 CloudFormation이 이해할 수 있는 형태로 자동 변환해준다.

## 템플릿 예시

예를 들어, Lambda + API Gateway를 CloudFormation으로 직접 정의하면 수십 줄이 필요하지만, SAM에서는 단 몇 줄로 표현할 수 있다.

```yaml
# SAM template.yaml 예시 (API Gateway + Lambda)
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31  # ← SAM임을 선언하는 핵심 줄!

Resources:
  HelloFunction:
    Type: AWS::Serverless::Function     # ← SAM 전용 타입
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.11
      Events:
        HelloApi:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

이 짧은 코드가 배포 시 내부적으로 Lambda 함수, API Gateway, IAM Role 등 여러 CloudFormation 리소스로 변환된다.

결국 AWS SAM의 핵심적인 기능은 "템플릿 변환"이다. AWS의 서버리스(Lambda, API Gateway, DynamoDB, SQS, SNS, Step Funtions 등...) 서비스를 CloudFormation 정의로 변환해주는 것이다. 참고로 AWS SAM에서 템플릿 변환은 서버리스 서비스에 한에서 가능하고 EC2, RDS와 같은 전통적인 인프라는 SAM 문법으로 추상화시킬 수 없다. 다만 "template.yaml"에 CloudFormation 문법을 그대로 섞어서 함께 사용할 수 있다.

핵심 기능 외에 CloudFormation과 별개로 개발 경험의 전반을 도와주는 도구를 제공한다. AWS에 서버리스 함수 코드 등을 직접 배포하지 않아도, 로컬에서 도커 컨테이너 기반으로 시뮬레이션이 가능하다. 직접 올리고 AWS에서 확인하면 디버깅이 번거로워질 수 있기에 이에 대한 도움을 주는 것이다.

다음부터는 API Gateway + Lambda + SQS + DynamoDB를 사용한다고 가정할 때, template.yaml 작성 예시이다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:        # 모든 함수에 공통 적용할 설정
Description:    # 스택 설명 (선택)
Parameters:     # 외부에서 주입할 변수 (선택)
Resources:      # 실제 리소스 정의 (필수)
Outputs:        # 배포 후 출력할 값 (선택)
```

위는 템플릿의 전체 구조이다. 핵심은 Resources 섹션이고, 나머지는 보조적인 역할을 한다.

```yaml
Transform: AWS::Serverless-2016-10-31
```

해당 라인은 "이 파일은 SAM 템플릿입니다"라고 CloudFormation에 선언하는 것이다. 해당 라인이 없으면 SAM의 단축 문법을 인식하지 못한다.

```yaml
Globals:
  Function:
    Runtime: python3.11
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        STAGE: production
```

Globals는 공통 설정이다. 만약 템플릿을 통해 배포할 함수가 여러 개일 때, 해당 설정이 전체 Lambda 함수에 반영된다.

```yaml
Parameters:
  Stage:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - prod
```

개발 또는 프로덕션 환경을 분리하는데 활용된다.

```yaml
Resources:

  # ── API Gateway + Lambda (HTTP 요청 처리) ──────────────────
  OrderApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: order-api-handler
      Handler: src/api_handler.lambda_handler
      Runtime: python3.11
      # API Gateway 연결: Events에 정의
      Events:
        CreateOrder: # 해당 부분은 식별을 위한 이름 (같은 함수 내에서 중복 불가)
          Type: Api
          Properties:
            Path: /orders
            Method: post
        GetOrder:
          Type: Api
          Properties:
            Path: /orders/{orderId}
            Method: get
      # 이 함수가 사용할 리소스 접근 권한
      Policies:
        - SQSSendMessagePolicy:          # SAM 정책 템플릿 (단축형)
            QueueName: !GetAtt OrderQueue.QueueName

  # ── SQS Queue ─────────────────────────────────────────────
  OrderQueue:
    Type: AWS::SQS::Queue               # SQS는 CloudFormation 원본 문법 사용
    Properties:
      QueueName: order-queue
      VisibilityTimeout: 60
      # Dead Letter Queue 연결 (처리 실패 메시지 보관)
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt OrderDLQ.Arn
        maxReceiveCount: 3              # 3번 실패하면 DLQ로 이동

  OrderDLQ:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: order-dlq

  # ── Lambda (SQS 메시지 처리) ───────────────────────────────
  OrderProcessorFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: order-processor
      Handler: src/processor.lambda_handler
      Runtime: python3.11
      Events:
        SQSTrigger:
          Type: SQS                     # SQS → Lambda 트리거
          Properties:
            Queue: !GetAtt OrderQueue.Arn
            BatchSize: 10              # 한 번에 처리할 메시지 수
      Policies:
        - DynamoDBCrudPolicy:          # SAM 정책 템플릿
            TableName: !Ref OrderTable

  # ── DynamoDB Table ─────────────────────────────────────────
  OrderTable:
    Type: AWS::DynamoDB::Table         # DynamoDB도 CloudFormation 원본 문법
    Properties:
      TableName: orders
      BillingMode: PAY_PER_REQUEST     # 온디맨드 요금제
      AttributeDefinitions:
        - AttributeName: orderId
          AttributeType: S
      KeySchema:
        - AttributeName: orderId
          KeyType: HASH                # Partition Key
```

Resources에는 각 서비스에 대한 구체적인 정의를 한다. 주문을 받으면 SQS로 보내고, Lambda가 메시지를 가져와서 DynamoDB에 저장하는 구조이다.

```yaml
Outputs:
  ApiEndpoint:
    Description: "API Gateway endpoint URL"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
  OrderTableName:
    Value: !Ref OrderTable
```

Output에는 배포가 끝난 뒤 확인하고 싶은 값에 대한 정의를 한다.

- 리소스 참조 함수:
  - `!Ref`: 리소스의 기본값 참조. !Ref OrderTable → 테이블 이름
  - `!GetAtt`: 리소스의 특정 속성 참조. !GetAtt OrderQueue.Arn → SQS ARN
  - `!Sub`: 문자열 안에 변수 삽입. !Sub "${Stage}-orders"

- SAM 리소스 타입과 CloudFormation 리소스 타입:
  - SAM 단축형: `AWS::Serverless::Function` -> Lambda + IAM Role + 트리거를 한 번에 정의
  - CloudFormation 원본: `AWS::SQS::Queue`, `AWS::DynamoDB::Table` -> SAM에 단축형이 없어서 원본 사용

- SAM 정책 템플릿
  위 예시에서 SQSSendMessagePolicy, DynamoDBCrudPolicy 같은 것들이 SAM Policy Template이다. IAM 정책을 직접 JSON으로 작성하는 대신, AWS가 미리 만들어둔 정책을 이름으로 가져다 쓸 수 있어서 매우 편리하다.

- 관련 문서: https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/serverless-policy-templates.html

## CLI를 통한 배포 기본

sam cli를 기반으로 배포하는 전체 로직을 정리하면 다음과 같다.

1. Lambda 함수 코드 + 템플릿(template.yaml) 준비
2. `sam validate` 템플릿 문법 검사
3. `sam build` 빌드 (의존성 설치 + 패키징)
4. `sam local` 로컬 테스트 (선택)
5. `sam deploy` AWS에 배포

```yaml
sam validate --lint
```

배포 전에 template.yaml에 문법적인 오류가 없는지 확인한다. `--lint` 옵션을 붙이면 더 꼼꼼하게 검사한다.

```yaml
sam build
```

해당 명령어는 세부적으로 두 가지 일을 한다.

- 의존성 설치: python 코드를 배포한다면 `requirements.txt`를 읽어서 패키지를 설치하고, 코드와 함께 압축한다.
- `.aws-sam/` 경로 생성: 빌드 결과물이 저장되는 경로를 생성한다. `sam local` 혹은 `sam deploy`는 원본 코드가 아니라 해당 경로를 기준으로 동작한다. 변환된 템플릿과 관련 코드들이 저장된다.

```yaml
sam local invoke OrderApiFunction --event events/test_event.json
```

해당 명령어는 Lambda 함수를 직접 실행한다.

```json
// events/test_event.json
{
  "httpMethod": "POST",
  "path": "/orders",
  "body": "{\"item\": \"coffee\", \"quantity\": 2}"
}
```

json에 정의해둔 테스트용 입력 값이 로컬에서 lambda를 실행 시 전달된다.

```bash
sam local start-api
# → http://localhost:3000 으로 API 호출 가능
```

```bash
curl -X POST http://localhost:3000/orders \
  -H "Content-Type: application/json" \
  -d '{"item": "coffee"}'
```

API Gateway를 실행하고 curl 혹은 postman과 같은 도구로 테스트가 가능하다.

```bash
sam deploy --guided


# 처음 실행하면 아래처럼 대화형으로 설정을 물어본다.
Stack Name [sam-app]: my-order-app
AWS Region [us-east-1]: ap-northeast-2
Confirm changes before deploy [y/N]: y
Allow SAM CLI IAM role creation [Y/n]: Y
Save arguments to configuration file [Y/n]: Y   # ← samconfig.toml에 저장
```

만약 설정을 저장하면 `samconfig.toml`파일이 생성되고, 이후부터는 별도 설정없이 자동 배포된다.

### 배포 과정 내부 동작

1. sam deploy 실행
2. 빌드 결과물을 S3 버킷에 업로드 -> SAM이 자동으로 버킷 생성
3. template.yaml을 CloudFormation 형식으로 변환
4. CloudFormation 스택 생성/업데이트
5. Outputs 출력 (API Endpoint URL 등)

```toml
version = 0.1
[default.deploy.parameters]
stack_name = "my-order-app"
region = "ap-northeast-2"
confirm_changeset = true
capabilities = "CAPABILITY_IAM"
s3_prefix = "my-order-app"
```

`samconfig.toml`파일의 예시이다. 해당 파일을 직접 수정해서 설정을 바꿀 수 있다. 해당 파일을 공유하면 동일한 배포 설정을 팀원과 공유하게된다.

```bash
sam delete
```

해당 명령어는 CloudFormation 스택 전체를 삭제해서 생성된 모든 리소스를 정리한다. 테스트 후 AWS 요금이 발생하지 않도록 꼭 정리해두는 습관이 필요하다.

## 로컬 시뮬레이션 관련

AWS API Gateway와 Lambda는 완전한 로컬 시뮬레이션을 제공하지만, SQS 혹은 DynamoDB와 같은 서비스는 SAM이 시뮬레이션을 직접 제공하지 않는다. 대신 DynamoDB는 AWS에서 공식 로컬 에뮬레이터를 도커 이미지로 제공하고, SQS는 `LocalStack`이라는 서드파티 도구를 제공한다. LocalStack은 SQS뿐 아니라 S3, SNS 등 여러 AWS 서비스를 로컬에서 에뮬레이션해주는 도구이다.

다만 현실적으로 로컬 환경을 모두 맞춰서 구성하는 것은 매우 번거롭다. 그래서 대부분의 팀들은 API Gateway와 Lambda 외에 서비스는 개발용 AWS 계정 혹은 스택을 별도로 띄워서 테스트하는 방식을 선택하기도 한다.

## 배포 아티팩트(AWS S3에 생성된 zip 파일 등)의 관리

배포 아티팩트 버킷은 `sam delete` 명령으로 자동 제거되지 않는다. AWS 정책 상 비어있지 않은 S3 버킷은 자동 삭제되지 않기 때문이다. 그리고 SAM이 생성한 S3 버킷은 CloudForamtion 스택 외부 리소스이다. SAM이 내부적으로 aws-sam-cli-managed-default라는 별도 스택을 만들어서 이 버킷을 관리하는데, sam delete는 애플리케이션 스택만 삭제하고 이 관리용 스택은 건드리지 않는다.

완전히 정리할 목적이라면 다음 순서로 직접 제거해야 한다.

```bash
# 1. 애플리케이션 스택 삭제
sam delete

# 2. S3 버킷 내용물 비우기 (AWS 콘솔에서 하거나 CLI로)
aws s3 rm s3://aws-sam-cli-managed-default-samclisourcebucket-xxxx --recursive

# 3. SAM 관리용 스택 삭제 (버킷도 함께 삭제됨)
aws cloudformation delete-stack --stack-name aws-sam-cli-managed-default
```

## SAM 배포 환경 분리

SAM에서 환경을 분리하는 방식은 크게 두 가지이다.

- Parameters + samconfig.toml — 하나의 템플릿, 환경별 값만 다르게 주입
- 별도 samconfig 파일 — 환경별 설정 파일을 분리

첫 번째 "Parameters로 환경 변수 정의"이다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Parameters:
  Stage:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev

Globals:
  Function:
    Runtime: python3.11
    Timeout: 30
    Environment:
      Variables:
        STAGE: !Ref Stage                          # Lambda 코드에서 os.environ["STAGE"]로 접근

Resources:
  OrderApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub "order-api-${Stage}"      # dev/prod별 함수 이름 분리
      Handler: src/handler.lambda_handler

  OrderTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub "orders-${Stage}"            # dev/prod별 테이블 이름 분리
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: orderId
          AttributeType: S
      KeySchema:
        - AttributeName: orderId
          KeyType: HASH
```

template.yaml 파일이다. `!Sub "order-api-${Stage}"` 처럼 리소스 이름에 Stage를 붙이면, dev/prod가 같은 AWS 계정 안에서도 충돌 없이 공존할 수 있다.

두 번째, "samconfig.toml — 환경별 배포 설정 분리" 이다.

```toml
# samconfig.toml

version = 0.1

# ── dev 환경 ────────────────────────────────
[dev.deploy.parameters]
stack_name = "my-order-app-dev"
region = "ap-northeast-2"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Stage=dev"

# ── prod 환경 ───────────────────────────────
[prod.deploy.parameters]
stack_name = "my-order-app-prod"
region = "ap-northeast-2"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Stage=prod"
confirm_changeset = true              # prod는 배포 전 변경사항 확인
```

`--guided` 없이 환경별로 배포할 수 있도록 설정을 저장해두는 파일이다.

```bash
sam deploy --config-env dev    # dev 배포
sam deploy --config-env prod   # prod 배포
```

---

환경별 설정값이 많아지면 — `toml` 파일 분리

환경별 차이가 많아지면 파일 자체를 분리하는 게 더 깔끔해요.
```
your-project/
├── template.yaml
├── samconfig.dev.toml
└── samconfig.prod.toml
```

배포 시 --config-env 옵션으로 환경을 지정합니다.

```bash
sam deploy --config-file samconfig.dev.toml
sam deploy --config-file samconfig.prod.toml
```