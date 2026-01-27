# MQTT
MQTT(Message Queue Telemetry Transport)는 앤디 스텐포드-클락(Andy Stanford-Clark)과 알렌 니퍼(Arlen Nipper)가 인공위성을 통해 오일 파이프라인 원격 검침 시스템을 연결하기 위해 1999년부터 설계한 것으로 2010년, 로열티 없는 상태로 출시되어 2014년 표준이 되었습니다. 이후 MQTT V3.1 및 V3.1.1을 중심으로 IoT(Internet of Things) 장치 사이 저전력·저대역폭 환경에 최적화된 경량 메시징 프로토콜 중 하나로 자리매김하고 있으며 2018년 1월 최신 버전인 MQTT V5가 승인된 상태입니다.

MQTT 시스템은 중계 서버 역할을 하는 브로커(broker)와 브로커에 토픽을 기반으로 메시지를 보내고 받는 클라이언트로 구성되는데, 클라이언트는 브로커로부터 토픽 메시지를 구독하는 구독자(subscriber)와 토픽에 메시지를 발행하는 발행자(publisher)로 나누어집니다.

## MQTT 개요 
MQTT는 TCP/IP를 기반으로 한 메시지 버스 시스템으로 클라이언트(발행자, 구독자)와 서버(브로커), 세션, 구독, 토픽으로 구성되며, 게시/구독 모델을 사용하여 메시징을 수행하는 간단한 방법을 제공합니다. 따라서 저전력 센서나 전화기, 내장형 컴퓨터 또는 마이크로 컨트롤러와 같은 모바일 장치를 사용하는 사물 인터넷 메시징에 적합합니다.

![MQTT Architecture](res/MQTT%20Architecture.png)

### MQTT 브로커 
브로커는 TCP 포트 1883을 사용하는 일종의 서버 프로그램으로 발행자와 구독자 사이 중계자 역할만 하므로 서버 대신 브로커란 용어를 사용합니다. 브로커가 발행자로부터 메시지를 수신하면 토픽을 기반으로 이를 게시한 후 토픽에 가입한 모든 구독자에게 배포합니다. 기본적으로 게시된 메시지 중 가입자가 없는 메시지나 배포된 메시지는 제거되지만 보존 속성을 가진 메시지나 영구 세션에서 발행한 메시지는 최소한 다음 배포까지 유지됩니다.

브로커는 IBM MQ와 같은 상용 제품부터 EMQ, mosquitto와 같은 오픈소스에 이르기까지 다양한 제품들이 있는데, 이 중 이클립스 재단에서 배포하는 mosquitto는 MQTT V5.0 프로토콜까지 지원하며, 가볍고 빨라 라즈베리파이와 같은 저전력 단일 보드 컴퓨터부터 서버에 이르기까지 모든 장치에 사용하기 적합합니다.

다음은 운영체제 별로 mosquitto 브로커를 설치하는 방법입니다.

- Linux
```sh 
sudo apt install mosquitto  
```
- Windows 
```sh 
winget install EclipseFoundation.Mosquitto
```
- macOS
```sh
brew install mosquitto 
```

설치가 완료되면 mosquitto는 백그라운드 서비스로 동작하는데, 윈도우에서는 사용자가 작업 관리자의 서비스 탭 또는 서비스 관리자를 실행한 후 mosquitto 항목을 찾아 컨텍스트 메뉴(마우스 우클릭)에서 시작을 선택해야 합니다.
또한 설정 파일인 mosquitto.conf를 편집해 외부 접속 허용, 익명 허용, 보안 로그인과 같은 각종 설정을 변경할 수 있습니다.

- 리눅스: /etc/mosquitto/conf.d/
- 윈도우: mosquitto 설치 폴더

공개용 브로커 간단한 테스트가 목적이라면 브로커를 자신의 컴퓨터에 설치하지 않고 인터넷 클라우드를 통해 무료로 사용할 수 있는 공개용 브로커를 사용해도 됩니다. 특히 공개용 브로커는 인터넷을 통해 언제 어디서든 접속이 가능하므로 사설 IP 환경에서 특히 유용한 선택입니다.

| Name | Broker Address | TCP Port | TLS Port | WebSocket Port| Message Retention |
|:---|:---|:---|:---|:---|:---|
| Eclipse | mqtt.eclipse.org | 1883	| N/A | 80, 443 | YES |
| Mosquitto | test.mosquitto.org | 1883	| 8883, 8884 | 80 | YES |
| HiveMQ | broker.hivemq.com | 1883	| N/A | 8000 | YES |
| Flespi | mqtt.flespi.io | 1883 | 8883 | 80, 443 | YES |
| Dioty	| mqtt.dioty.co | 1883 | 8883 |	8080, 8880 | YES |
| Fluux	| mqtt.fluux.io | 1883 | 8883 | N/A | N/A |
| EMQX | broker.emqx.io | 1883 | 8883| 8083 | YES | 

### MQTT 클라이언트 
해당 장치가 발행자면 토픽에 대한 메시지를 발행해 브로커에 게시할 수 있고 구독자는 특정 토픽을 등록하여 메시지를 수신할 수 있습니다. 하나의 클라이언트는 구독자 또는 발행자이거나 모두일 수 있습니다. 보안 기능을 활성화하면 브로커에 연결할 때 사용자 ID와 비밀번호 또는 인증서를 이용해 인증합니다.

클라이언트는 일반적으로 공개된 MQTT 클라이언트 라이브러리를 이용해 구현하지만 이클립스 재단에서 mosquitto와 함께 제공하는 mosquitto-clients 나 EMQX의 MQTTX 처럼 범용으로 구현된 프로그램들도 있습니다. 특히 MQTTX는 운영체제 별로 GUI 및 CLI을 지원하며, 웹 환경까지 지원해 상황에 맞게 선택할 수 있습니다.

MQTTX 설치 프로그램은 다음 링크를 통해 다운로드 받아 쉽게 설치할 수 있습니다. 
- [MQTTX](https://mqttx.app/downloads)

### 토픽
토픽은 메시지에 대한 발행/구독 패턴의 기준으로 클라이언트 사이 미리 정의한 의미대로 정보를 교환할 수 있게 합니다. 대소 문자를 구분하는 계층 구조의 UTF-8 문자열로 파일 시스템의 경로와 같이 슬래시(/)로 구분되며 발행자와 브로커, 브로커와 구독자 사이 토픽에 따른 메시지 흐름을 구분합니다.

#### 토픽 구조 
$SYS 토픽을 제외하고 기본 또는 표준화된 토픽 구조는 없습니다. $SYS 토픽은 브로커에 대한 정보를 공개하기 위해 대부분의 브로커에서 예약되어 있습니다.

![](res/Topic.png)

앞의 그림의 토픽은 3개의 레벨로 구성되어 있습니다. 첫번째는 Sensor, 두번째는 imu, 세번째는 value 입니다. 

#### 토픽 생성 및 제거 
발행자는 메시지를 발행할 때마다 토픽을 포함하므로 브로커는 토픽이 없으면 만든 후 게시합니다. 또한 구독자도 토픽에 가입할 때 토픽이 없으면 브로커는 만든 후 가입시킵니다. 토픽은 세션이 제거될 때 제거됩니다.

#### 토픽 필터 
발행자가 토픽에 메시지를 게시할 때는 항상 개별적인 발행만 허용하므로 두 가지 토픽에 메시지를 게시하려면 두 번 발행해야 합니다. 하지만 구독자는 여러 토픽에 가입하는 대신 토픽 필터를 적용해 한 번만 가입해도 됩니다. 토픽 필터는 단일 레벨 필터일 '+'와 다중 레벨 필터인 '#' 중 하나를 이용해 레벨 또는 이후 모든 레벨을 와일드카드(wildcard)로 설정합니다.

- +: 단일 레벨 와일드카드로 레벨의 모든 문자 치환
- #: 다중 레벨 와일드카드로 현재 레벨과 이후 모든 레벨의 문자 치환
    - $SYS/#: 브로커의 모든 토픽에 가입

아래와 같은 토픽이 있다고 가정하겠습니다. 

- Sensor/imu/value
- Sensor/ultrasonic/distance
- Sensor/cds/value

이 때 Sensor 하위에서 value로 끝나는 모든 토픽을 구독한다면 다음과 같이 필터를 적용할 수 있습니다. 

- Sensor/+/value

Sensor 하위 모든 토픽을 구독하는 경우에는 다음과 같이 적용합니다. 

- Sensor/# 

### 세션 
세션은 연결된 클라이언트와 브로커 사이 상호작용을 관리하기 위한 것으로 클라이언트와 브로커는 세션 정보 중 서비스 품질을 나타내는 QoS를 통해 "적어도 한 번" 및 "정확히 한 번" 발행과 "정확히 한 번" 구독을 보장합니다. 또한 구독자가 작성한 구독도 세션 정보에 포함됩니다.

다음은 세션에 저장되는 내용입니다.

- 구독이 없는 세션의 존재 여부
- 모든 구독
- 클라이언트가 확인하지 않은 QoS = 1 또는 QoS = 2 메시지
- 클라이언트가 끊어진 동안 놓친 모든 새로운 QoS = 1 또는 QoS = 2 메시지
- 클라이언트에서 아직 확인되지 않은 모든 수신된 QoS = 2 메시지

클라이언트를 연결할 때 이전 세션 정보를 유지하거나 유지하지 않도록 선택할 수 있는데, 영구 세션(CleanSession = False)은 세션 상태를 유지하고 클린 세션(CleanSession = True)은 유지하지 않습니다.

클라이언트가 클라이언트 ID와 브로커 주소, 클린 세션 여부, 사용자 ID, 비밀번호 또는 인증서를 이용해 브로커에 연결을 요청하면 브로커는 세션 정보가 이전 연결에서 저장되어 있는지 확인합니다. 이전 세션이 존재할 때 클린 세션이면 클라언트와 브로커의 이전 세션 상태는 지워지고 영구 세션이면 이전 세션 상태를 다시 사용합니다. 이전 세션이 없으면 새 세션이 시작됩니다.

#### 발행 
클린 세션으로 클라이언트를 연결하면 보류 중인 모든 발행의 송신이 제거됩니다. 또한 클린 세션에서 QoS = 0으로 발행된 메시지는 영향을 주지 않지만 QoS = 1 및 QoS = 2로 발행한 메시지는 게시되지 않고 잃어버릴 수 있습니다.

#### 구독
세션 상태에서 구독은 논리적으로 클라이언트의 토픽을 연결하며 토픽에 가입한 클라이언트는 게시된 메시지를 수신할 수 있습니다. 클린 세션으로 클라이언트를 연결하면 브로커는 클라이언트에 대한 이전 구독을 모두 제거합니다. 또한 연결이 끊어지면 세션 중에 클라이언트가 만든 모든 새 구독도 함께 제거됩니다. 반면에 영구 세션에서는 클라이언트가 만드는 구독이 연결되기 전에 클라이언트에 있었던 모든 구독에 추가됩니다. 클라이언트가 연결을 끊어도 모든 구독은 활성 상태를 유지합니다.

클린 세션은 기본 세션 모드로 세션 범위 내에서만 게시된 메시지를 받을 수 있으므로 구독은 일시적입니다. 따라서 세션이 닫히면 세션 상태도 지워지므로 이후 게시된 메시지는 잃어버린다. 하지만 영구 세션은 구독이 영구적이므로 클라이언트의 연결을 끊었다가 다시 연결하면 QoS = 1 또는 QoS = 2로 게시된 메시지를 받게 됩니다.

![session](res/session.png)

어떤 세션이든 한 번 만들어지면 전체 세션 동안 지속되므로 모드를 바꾸려면 클라이언트의 연결을 끊었다가 다시 연결하는 방법밖에 없습니다. 영구 세션을 끊고 클린 세션으로 다시 연결하면 클라이언트에 대한 모든 이전 구독 및 수신되지 않은 모든 발행이 삭제됩니다.

#### 클라이언트 ID  
클라이언트 ID는 브로커가 세션 안에서 각각의 클라이언트를 구분하는 유일한 식별자로 숫자와 영문자 대소문자 및 '_' 문자를 조합해 만듭니다. CleanSession = True일 때는 브로커가 동적으로 만들지만 CleanSession = False에서는 클라이언트가 반드시 유효한 문자열로 클라이언트 ID를 부여해야 하는데, 물리적으로 다른 장치에서 동일한 클라이언트 ID를 사용하면 보류 중인 게시 및 활성 구독이 자동으로 새 장치로 전송되므로 장애가 발생한 장치의 이전이 쉽습니다.

브로커가 동적으로 만드는 클라이언트 ID는 고유성이 보장되지만 클라이언트에서 명시적으로 만들 때는 브로커에 이미 등록된 클라이언트 ID인지 알 수 없으므로 128bit 고유 문자열을 생성하는 uuid 툴이나 네트워크 인터페이스의 MAC 주소 등을 사용하는 것과 같이 스스로 특별한 규칙을 적용할 필요가 있습니다.

### MQTT 클라이언트 라이브러리 
대부분의 브로커는 MQTT 클라이언트 라이브러리를 함께 제공하므로 MQTT 클라이언트 라이브러리의 종류는 브로커만큼 다양합니다. 그중 IBM으로부터 시작된 Paho API는 Java부터 C, 파이썬에 이르기까지 다양한 언어를 지원하는 오픈소스 MQTT 클라이언트 라이브러리로, 동기/블로킹 및 비동기 API를 지원합니다. 동기/블로킹 기능을 사용하면 간결하게 구현할 수 있으며, 비동기 기능은 고성능 MQTT 클라이언트를 작성할 수 있도록 높은 처리량을 제공합니다.

Paho API는 콜백 기반이며 사용자 정의 비즈니스 로직을 다른 이벤트(예: 메시지 수신시 또는 브로커 연결이 끊어진 경우)에 연결합니다. 또한 TLS 기반 보안 통신을 비롯해 최신 MQTT V5.0와 V3.1.1, v3.1을 모두 지원합니다.
파이썬용 Paho API 설치는 다음과 같습니다.

```sh 
pip install paho.mqtt<2.0.0
```
[주의] 2024년 2월에 중요한 업데이트가 포함된 V2.0.0이 출시되었으며, 이 문서는 1.X 버전을 기준으로 설명합니다. 따라서 paho-mqtt를 설치할 때는 버전을 2.0.0 미만을 설치해 주세요.
만약 V2.0.0 이상을 설치했다면 링크를 참조하세요.
- [Migrations](https://eclipse.dev/paho/files/paho.mqtt.python/html/migrations.html)

# MQTT 응용 
MQTT 응용 구현에 사용하는 MQTT API는 내부 소켓 인터페이스로 사용자를 대신해 브로커에 메시지를 발행하고 구독에 가입한 후 메시지가 배포될 때마다 이를 사용자에게 전달하므로 일반적인 소켓 프로그램보다 쉽습니다.

## 단방향 클라이언트 
발행자와 구독자는 통신 방향에 따라 개별적으로 구현하거나 하나로 구현할 수 있습니다. 개별 클라이언트 구현은 단방향으로 메시지가 흐르므로 발행자가 메시지를 브로커에 게시하면 브로커는 이를 구독자에게 배포합니다.

브로커는 원하는 브로커에 연결하여 활용하면 됩니다. 무료, 상용 모두 관계없습니다. 여기서는 XConvey의 HMI에서 동작하는 mosquitto 브로커를 활용하겠습니다. 무료로 활용가능하며 익명 연결을 허용하는 브로커입니다. 브로커 주소는 XConvey의 HMI IP 주소 입니다. 이 때, XConvey의 HMI와 같은 네트워크에 연결되어 있어야 합니다.

### simple_pub.py 
아주 단순한 토픽 발행자입니다. 0부터 254까지 숫자를 1초마다 발행합니다. 발행하는 토픽은 iot/topic/test 입니다. 

```python
import paho.mqtt.client as mqtt
import time

client = mqtt.Client()

def main():
    client.connect("<HMI IP Address>")
    client.loop_start()
    number = 0
    while True:
        number = number + 1
        if number > 255:
            number = 0
        client.publish("iot/topic/test", number)
        time.sleep(1)

if __name__ == "__main__":
    main()
```

### simple_sub.py 
토픽을 구독하여 출력하는 구독자입니다. 토픽이 발행되면 수신한 토픽의 명칭과 데이터를 출력합니다. 

```python
import paho.mqtt.client as mqtt

client = mqtt.Client()

def on_message(client, userdata, message):
    print(message.topic, int(message.payload))

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        client.subscribe("iot/topic/test")

def main():
    client.on_connect = on_connect
    client.on_message = on_message

    client.connect("<HMI IP Address>")
    client.loop_forever()

if __name__ == "__main__":
    main()
```

브로커에 접속을 시도하여 "iot/topic/test" 토픽을 구독합니다. 발행된 토픽이 있다면 토픽의 명칭과 payload를 화면에 출력합니다. 토픽이 발행될때 마다 on_message() 콜백이 호출되어 관련 정보가 출력됩니다. 

## 콜백 이벤트 
네트워크 루프는 사용자를 대신해 네트워크 상태를 감시하다가 이벤트가 발행하면 이를 콜백으로 응용프로그램에 알립니다. 따라서 호출되는 콜백은 이벤트의 종류에 따라 다릅니다. 콜백 이벤트는 다음과 같이 정리할 수 있습니다. 

- on_connect(client, userdata, flags, rc) 
    - 브로커에 연결되면 호출, rc 가 0 이면 정상 연결 
- on_disconnect(client, userdata, rc)
    - 브로커와 연결이 끊어지면 호출, rc 가 0 이면 정상 종료 
- on_publish(client, userdata, mid)
    - 메시지를 발행한 후 브로커로부터 확인을 받았을 때 호출
        - QoS 1 과 2에서 사용
- on message(client, userdata, message)
    - 구독한 토픽의 메시지를 수신했을 때 호출 
- on_subscribe(client, userdata, mid, granted_qos)
    - subscribe() 호출 후 구독이 성공했을 때 호출 
- on_unsubscribe(client, userdata, mid)
    - unsubscribe() 호출 후 구독 해제가 완료되었을 때 호출 
- on_log(client, userdata, level, buf)
    - 디버깅용 로그 메시지를 수신했을 때 호출 

### 연결 및 해제 콜백 
on_connect() 콜백은 브로커로부터 CONNACK 패킷을 수신하면 호출됩니다. CONNACK는 connect*()로 송신한 CONNECT 패킷에 대한 응답으로, 일정 시간 동안 CONNACK를 수신하지 못하면 OSError 예외가 발생합니다.

on_disconnect() 콜백은 disconnect()로 브로커에 DISCONNECT 패킷을 송신한 후 진행 중인 작업이 있다면 이를 중단하고 호출됩니다. disconnect()를 호출한 클라이언트는 reconnect()만 유효하며 reconnect_delay_set()으로 기본값이 1초인 연결 지연 시간을 조정할 수 있습니다.

#### cb_client_connect_a.py
아래 코드는 아주 단순한 토픽 구독자를 구현한 내용입니다. 처음 실행할 때 브로커에 접속을 시도하고 브로커 접속이 정상적으로 수행되면 "iot/sensor/value" 토픽을 구독하고 발행된 토픽이 있다면 화면에 토픽의 정보를 출력합니다. 

```python
import paho.mqtt.client as mqtt
import signal

client = mqtt.Client()

def signal_handler(signal, frame):
    client.disconnect()

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("connected...")
        client.subscribe("iot/sensor/value")
    else:
        print("connection fail:", rc)
        sys.exit(0)

def on_disconnect(client, userdata, rc):
    print("reconnected...")
    client.reconnect()

def on_message(client, userdata, message):
    print(message.payload.decode())

def main():
    signal.signal(signal.SIGINT, signal_handler)
    client.on_connect = on_connect
    client.on_disconnect = on_disconnect
    client.on_message = on_message
    client.reconnect_delay_set(5)

    try:
        client.connect_async("<HMI IP Address>")
        client.loop_forever()
    except OSError as e:
        print(e)

if __name__ == "__main__":
    main()
```

여기서 singnal() 등록은 프로그램에 외부 인터럽트 (Ctrl+c 와 같은 키입력)을 감지하기 위함입니다. Ctrl+c 키 입력이 감지되면 접속되어 있는 브로커 연결을 해제합니다. 여기서 on_disconnect() 가 호출되며 reconnect() 를 통해 브로커에 접속을 다시 시도합니다. 이때 reconnect_delay_set(5) 설정에 따라 5초 후 재접속을 시도합니다. 이 프로그램은 Ctrl+c 입력을 한번이 아니라 여러번 연달아 입력해야 프로그램이 종료됩니다. 

#### cb_client_connect_b.py
다음의 코드는 "iot/sensor/value" 토픽을 발행하는 발행자입니다. 브로커에 접속되면 on_connect() 가 호출되어 메시지를 출력하고 10개의 토픽을 발행하고 브로커와 연결을 해제합니다. 

```python
import paho.mqtt.client as mqtt
import sys

client = mqtt.Client()

def publish_value(client, i):
    client.publish("iot/sensor/value", i)

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("connected...")
        for i in range(10):
            publish_value(client, i)

        print("completed...")
        client.disconnect()
    else:
        print("connection fail:", rc)
        sys.exit(0)

def on_disconnect(client, userdata, rc):
    print("disconnected...")

def main():
    client.on_connect = on_connect
    client.on_disconnect = on_disconnect

    try:
        client.connect_async("<HMI IP Address>")
        client.loop_forever()
    except OSError as e:
        print(e)

if __name__ == "__main__":
    main()
```

### 발행 및 수신 콜백 
토픽 발행 및 구독 관련 콜백은 on_message, on_publish 등을 활용할 수 있습니다. 

on_publish()는 publish()로 PUBLISH 패킷을 송신하거나(QoS = 0), PUBACK 응답 패킷을 수신하거나(QoS = 1), PUBCOMP 2차 응답 패킷을 수신하면(QoS = 2) 호출되므로 publish()로 발행한 메시지는 가급적 on_publish() 콜백으로 브로커에 송신 또는 게시되었는지 확인하는 것이 좋습니다.

#### cb_client_pub.py

```python
import paho.mqtt.client as mqtt
import time 
from xconvey import Sorting

client = mqtt.Client()
sorting = Sorting() 
index = 0
qos = 0

def publish_sorting_photo(client):
    global sorting, qos 
    client.publish("iot/sorting/photo/value", int(sorting.photo), qos)
    time.sleep(1)

def on_publish(client, userdata, mid):
    global index, qos
    if index > 3:
        index = 0
        qos += 1
    else:
        index += 1

    if qos > 2:
        client.disconnect()
    else:
        publish_sorting_photo(client)

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        publish_sorting_photo(client)

def main():
    client.on_connect = on_connect
    client.on_publish = on_publish
    try:
        client.connect_async("<HMI IP Address>")
        client.loop_forever()
            
    except OSError as e:
        print(e)

if __name__ == "__main__":
    main()
```
#### cb_client_sub.py

```python
import paho.mqtt.client as mqtt
import sys

client = mqtt.Client()

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        client.subscribe("iot/sorting/photo/value")
    else:
        print("connection fail:", rc)
        sys.exit(0)

def on_message(client, userdata, message):
    print(int(message.payload))

def main():
    client.on_connect = on_connect
    client.on_message = on_message
    try:
        client.connect_async("<HMI IP Address>")
        client.loop_forever()
    except OSError as e:
        print(e)

if __name__ == "__main__":
    main()
```