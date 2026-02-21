## AMQP

**AMQP (Advanced Message Queuing Protocol)** 는 2003년 JP Morgan Chase가 금융 거래 시스템의 신뢰성 있는 메시지 전달을 위해 설계한 **오픈 표준 메시지 큐잉 프로토콜**입니다. 말씀하신 RabbitMQ가 AMQP를 구현한 대표적인 메시지 브로커입니다.

### 핵심 개념

MQTT도 Pub/Sub 패턴을 사용하지만, AMQP는 훨씬 정교한 **메시지 라우팅** 메커니즘을 제공합니다. 핵심 구성 요소는 세 가지입니다.

```
[Producer]
    ↓ 메시지 발행
[Exchange]  ←  라우팅 규칙(Binding)에 따라 메시지를 분배
    ↓
[Queue A]  [Queue B]  [Queue C]
    ↓
[Consumer]
```

**Producer**는 메시지를 보내는 측, **Exchange**는 메시지를 받아 적절한 큐로 라우팅하는 우체국 역할, **Queue**는 메시지를 저장하는 버퍼, **Consumer**는 큐에서 메시지를 가져가는 측입니다.

### Exchange의 4가지 라우팅 타입

AMQP의 가장 강력한 특징은 Exchange의 다양한 라우팅 방식입니다.

**Direct Exchange**는 라우팅 키가 정확히 일치하는 큐로 전달합니다.

```
메시지 (routing_key="error") → error 큐로만 전달
메시지 (routing_key="info")  → info 큐로만 전달
```

**Fanout Exchange**는 라우팅 키를 무시하고 연결된 모든 큐에 브로드캐스트합니다.

```
메시지 → Queue A, Queue B, Queue C 모두에 전달
```

**Topic Exchange**는 와일드카드 패턴으로 라우팅합니다.

```
"log.error.payment"  → "log.error.*" 구독 큐에 전달
"log.info.user"      → "log.#" 구독 큐에 전달 (#: 다중 레벨)
```

**Headers Exchange**는 라우팅 키 대신 메시지 헤더 값으로 라우팅합니다.

### Python 예시

```python
import pika

# 연결 설정
connection = pika.BlockingConnection(
    pika.ConnectionParameters("localhost")
)
channel = connection.channel()

# Exchange와 Queue 선언
channel.exchange_declare(exchange="logs", exchange_type="topic")
channel.queue_declare(queue="error_logs", durable=True)  # durable: 서버 재시작 후에도 유지
channel.queue_bind(
    exchange="logs",
    queue="error_logs",
    routing_key="log.error.*"
)

# Producer — 메시지 발행
channel.basic_publish(
    exchange="logs",
    routing_key="log.error.payment",
    body="결제 오류 발생!",
    properties=pika.BasicProperties(
        delivery_mode=2  # 메시지 영속성 보장 (디스크에 저장)
    )
)

# Consumer — 메시지 소비
def callback(ch, method, properties, body):
    print(f"수신: {body.decode()}")
    ch.basic_ack(delivery_tag=method.delivery_tag)  # 처리 완료 확인

channel.basic_consume(queue="error_logs", on_message_callback=callback)
channel.start_consuming()
```

### 메시지 신뢰성 보장 메커니즘

AMQP가 금융권에서 시작된 만큼, 메시지가 절대 유실되지 않도록 하는 장치들이 잘 갖춰져 있습니다.

**Acknowledgement (ACK)** 는 Consumer가 메시지를 정상 처리했음을 브로커에게 확인시켜주는 방식입니다. ACK를 보내기 전에 Consumer가 죽으면 브로커가 자동으로 다른 Consumer에게 재전달합니다.

**Durability**는 Exchange와 Queue를 durable로 선언하고 메시지에 영속성 옵션을 주면, 브로커가 재시작되더라도 메시지가 유실되지 않습니다.

**Dead Letter Queue (DLQ)** 는 처리에 실패하거나 만료된 메시지를 별도 큐로 보내 나중에 분석하거나 재처리할 수 있게 합니다.

### MQTT와의 비교

앞서 다룬 MQTT와 자주 비교되는데, 목적 자체가 다릅니다.

| 항목 | MQTT | AMQP |
|------|------|------|
| 설계 목적 | IoT, 경량 디바이스 | 엔터프라이즈 메시징 |
| 메시지 크기 | 매우 작음 | 상대적으로 큼 |
| 라우팅 | 단순 (Topic) | 정교함 (Exchange 타입) |
| 메시지 보장 | QoS 0/1/2 | ACK + 트랜잭션 |
| 주요 브로커 | Mosquitto, EMQX | RabbitMQ, ActiveMQ |

### 장단점

장점으로는 정교한 메시지 라우팅, 강력한 메시지 유실 방지 메커니즘, 트랜잭션 지원이 있습니다. 브로커가 메시지를 보관하기 때문에 Producer와 Consumer가 동시에 살아있지 않아도 되는 **비동기 디커플링**도 큰 강점입니다.

단점으로는 MQTT에 비해 무겁고 설정이 복잡합니다. 브로커가 중앙에 있어 단일 장애점이 될 수 있고, 클러스터 구성 시 추가적인 인프라 관리가 필요합니다.

### 주요 사용 사례

금융 거래 처리, 주문/결제 시스템처럼 **메시지 유실이 절대 허용되지 않는** 환경에 적합합니다. 또한 마이크로서비스 간 비동기 통신, 대용량 작업 큐(이메일 발송, 이미지 처리 등), 이벤트 드리븐 아키텍처에서 널리 사용됩니다. Pivotal, VMware, 그리고 수많은 금융/이커머스 기업들이 RabbitMQ(AMQP)를 핵심 인프라로 사용합니다.
