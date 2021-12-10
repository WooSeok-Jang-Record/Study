# Airflow DAG 구축 과정
## DAG란?
- 유향 비순환 그래프 (Directed Acyclic Graph)

## Pipeline 구축 (ESP32 -> MQTT -> Airflow-DAG -> DB & Dash)
- Airflow-DAG 참고 링크 : [https://dodonam.tistory.com/156]</br>

- ESP32-MQTT-AirflowDAG-Plotly-Dash (온/습도 데이터)
```python
import paho.mqtt.client as mqtt
import paho.mqtt.publish as publish
import random
from json import loads

import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Output, Input

from datetime import datetime
import plotly.graph_objs as go
from collections import deque

time = deque(maxlen=20)
time.append(datetime.now())
temperature = deque(maxlen=20)
temperature.append(0.)
humidity = deque(maxlen=20)
humidity.append(0.)

def on_connect(client, userdata, flags, rc):
    print("Subscribing to topic=esp_broker")
    if rc == 0:
        print("Connection established")
    else:
        print("Bad connection returned code=", rc)
    client.subscribe("esp_broker")

def on_message(client, userdata, message):
    msg = loads(message.payload.decode("utf-8"))
    print("message received ", 'Humidity:', msg['Humi'], 'Temperature:', msg['Temp'])
    time.append(datetime.now())
    temperature.append(msg['Temp'])
    humidity.append(msg['Humi'])

def on_log(client, userdata, level, buf):
    print("log: ", buf)    

app = dash.Dash(__name__)
app.config.suppress_callback_exceptions = True
app.layout = html.Div(children=[
    html.H1(children='Temperature and humidity monitoring in real time'),
    html.Div(children='Jinrigwan Room 510, Center of Digital Biomarker Research'),
    html.Div([
        dcc.Graph(id='live-graph-temperature', animate=True),
        dcc.Interval(
            id='graph-update-temperature',
            interval=1*1000
        ),
    ]),
    html.Div([
        dcc.Graph(id='live-graph-humidity', animate=True),
        dcc.Interval(
            id='graph-update-humidity',
            interval=1*1000
        ),
    ]),
])

@app.callback(
    Output('live-graph-temperature', 'figure'),
    Output('live-graph-humidity', 'figure'),
    Input('graph-update-temperature', 'n_intervals'),
    Input('graph-update-humidity', 'n_intervals'),
)
def update_graph_scatter(input_data_temperature, input_data_humidity): #함수명과 인자명은 사용자 임의대로 정할 수 있다. 인자의 순서는 맞추어야 한다.
    data_temperature = go.Scatter(
        x = list(time),
        y = list(temperature),
        name ='Temperature',
        mode = 'lines+markers',
        marker_color = 1,
        line_color="red",
    )

    data_humidity = go.Scatter(
        x = list(time),
        y = list(humidity),
        name = 'Humidity',
        mode = 'lines+markers',
        marker_color = 1,
        line_color="blue",
    )
    xaxis = dict(range=[time[0], time[-1]])
    min_temperature = min(temperature)
    max_temperature = max(temperature)
    margin_temperature = (max_temperature - min_temperature) * 0.1
    min_humidity = min(humidity)
    max_humidity = max(humidity)
    margin_humidity = (max_humidity - min_humidity) * 0.1
    return {
        'data': [data_temperature],
        'layout' : go.Layout(
            xaxis = xaxis,
            yaxis = dict(range=[
                min_temperature - margin_temperature, 
                max_temperature + margin_temperature,
            ]),
            title ='Temperature',
        )
    }, {
        'data': [data_humidity],
        'layout' : go.Layout(
            xaxis = xaxis,
            yaxis = dict(range=[
                min_humidity - margin_humidity, 
                max_humidity + margin_humidity,
            ]),
            title = 'Humidity',
        )
    }

if __name__ == '__main__':
    client = mqtt.Client(f'{random.randint(0, 1000)}')
    client.on_connect = on_connect
    client.on_message = on_message
    client.on_log = on_log
    client.connect('192.168.0.197')
    client.loop_start()

    app.run_server(host='0.0.0.0', debug=True)
    
    
    
