# [MQTT] mosquitto - python 테스트
## 참고
mqtt-python tutorial [http://www.steves-internet-guide.com/mqtt-python-beginners-course/]</br>
[https://nagy.tistory.com/6]
[https://souk0721.github.io/studynote/2018/07/11/mqtt-python-test-01.html]
MQTT 가장 간략한 예시 : [https://jusths.tistory.com/24]
### 설치
- Ubuntu 서버에 MQTT broker(mosquitto)가 서비스 되어 있다는 전제하여 Python으로 테스트를 진행함.
- python MQTT 라이브러리 설치

```python 
pip install paho-mqtt
```
</br>

- pub (발행) 코드
 ```python
 import paho.mqtt.publish as publish

msgs = \
[
    {
        'topic':"/seoul/yuokok",
        'payload':"multiple 1"
    },

    (
        "/seoul/yuokok",
        "multiple 2", 0, False
    )
]
publish.multiple(msgs, hostname="test.mosquitto.org")
#Topic /seoul/yuokok 에 문자값 multiple 1, multiple 2를 발행한다.
```

- Sub (구독) 코드
```python
import paho.mqtt.client as mqtt

# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, flags, rc):
    print("Connected with result code "+str(rc))

    # Subscribing in on_connect() means that if we lose the connection and
    # reconnect then subscriptions will be renewed.
    client.subscribe("/seoul/yuokok") # Topic /seoul/yuokok을 구독한다.

# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    print(msg.topic+" "+str(msg.payload))

client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

client.connect("test.mosquitto.org") # - 서버 IP '테스트를 위해 test.mosquitto.org'로 지정

# Blocking call that processes network traffic, dispatches callbacks and
# handles reconnecting.
# Other loop*() functions are available that give a threaded interface and a
# manual interface.
client.loop_forever()
```
</br>

sub.py 를 만들어 위의 코드를 삽입하고 python sub.py 를 실행하고 pub.py 를 만들어 python pub.py 를 실행하면, sub.py 를 실행한 커맨드 창에 아애와 같이 나온다.

```python
/seoul/yuokok multiple 2
/seoul/yuokok multiple 1
```

