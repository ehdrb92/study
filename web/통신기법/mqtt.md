## MQTT

**MQTT (Message Queuing Telemetry Transport)** 는 1999년 IBM의 Andy Stanford-Clark과 Arlen Nipper가 개발한 **경량 메시징 프로토콜**입니다. 원래는 송유관 모니터링처럼 네트워크가 불안정하고 대역폭이 좁은 환경을 위해 설계되었으며, 현재는 **IoT(Internet of Things)** 의 표준 프로토콜로 자리잡았습니다.

### 핵심 개념 — Pub/Sub 패턴

앞서 소개한 REST, GraphQL, gRPC는 모두 **요청-응답(Request-Response)** 방식이었습니다. MQTT는 전혀 다른 **발행-구독(Publish-Subscribe)** 패턴을 사용합니다.

```
[Publisher]  →  "topic: 온도센서/거실"  →  [Broker]  →  [Subscriber A]
                                                      →  [Subscriber B]
```

클라이언트는 서로를 직접 알 필요가 없고, **브로커(Broker)** 가 중간에서 메시지를 중계합니다. 대표적인 브로커로는 Mosquitto, EMQX, HiveMQ 등이 있습니다.

### Topic 구조

MQTT는 **토픽(Topic)** 이라는 계층적 문자열로 메시지를 분류합니다. 슬래시(`/`)로 계층을 구분하며, 와일드카드도 지원합니다.

```
home/livingroom/temperature   # 거실 온도
home/bedroom/humidity         # 침실 습도
home/+/temperature            # 모든 방의 온도 (+: 단일 레벨 와일드카드)
home/#                        # home 하위 모든 메시지 (#: 다중 레벨 와일드카드)
```

### Python 예시

```python
import paho.mqtt.client as mqtt

BROKER = "broker.hivemq.com"
TOPIC = "home/livingroom/temperature"

# Subscriber
def on_message(client, userdata, message):
    print(f"수신: {message.topic} = {message.payload.decode()}")

sub_client = mqtt.Client()
sub_client.on_message = on_message
sub_client.connect(BROKER, 1883)
sub_client.subscribe(TOPIC)
sub_client.loop_start()

# Publisher
pub_client = mqtt.Client()
pub_client.connect(BROKER, 1883)
pub_client.publish(TOPIC, payload="23.5")  # 온도 23.5도 발행
```

### QoS (서비스 품질 등급)

MQTT의 중요한 특징 중 하나가 **QoS(Quality of Service)** 레벨입니다. 네트워크가 불안정한 환경을 고려해 메시지 전달 보장 수준을 선택할 수 있습니다.

| QoS | 의미 | 특징 |
|-----|------|------|
| 0 | 최대 1회 전달 | 전달 보장 없음, 가장 빠름 |
| 1 | 최소 1회 전달 | 중복 수신 가능 |
| 2 | 정확히 1회 전달 | 가장 안전, 가장 느림 |

### 경량성

MQTT의 고정 헤더는 **단 2바이트**입니다. HTTP 헤더가 수백 바이트에 달하는 것과 비교하면 극단적으로 가볍습니다. 덕분에 메모리와 배터리가 제한된 센서, 임베디드 장치에서도 문제없이 동작합니다.

### 장단점

장점으로는 매우 낮은 대역폭 사용, 불안정한 네트워크에서도 안정적인 동작, Pub/Sub 특성상 하나의 메시지를 다수에게 동시 전달이 가능하다는 점이 있습니다.

단점으로는 브로커가 반드시 필요해 단일 장애점(SPOF)이 될 수 있고, 요청-응답 패턴이 기본이 아니라서 일반적인 API 서버로 사용하기엔 적합하지 않습니다. 또한 기본적으로 메시지 보안이 없어 TLS 설정을 별도로 해야 합니다.

### 주요 사용 사례

스마트홈(온도, 조명 제어), 공장 자동화, 차량 텔레매트릭스, 의료 기기 모니터링 등 **IoT 환경** 전반에 걸쳐 사용됩니다. AWS IoT Core, Azure IoT Hub, Google Cloud IoT 등 주요 클라우드 플랫폼도 모두 MQTT를 지원합니다.
