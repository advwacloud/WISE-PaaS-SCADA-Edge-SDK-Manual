# 使用說明 {#development-environement}

---

## EdgeAgent
### 1. Constructor\(EdgeAgentOptions options\)
初始化 EdgeAgent 實例，並根據傳入參數 EdgeAgentOptions 建立 MQTT 連線客戶端以及 SCADA 相關設定。

```
options = EdgeAgentOptions(
  reconnectInterval = 1, # MQTT reconnect interval seconds
  scadaId = '5095cf13-f005-4c81-b6c9-68cf038e2b87', # Getting from SCADA portal
  deviceId = 'deviceId', # If type is Device, DeviceId must be filled
  type = constant.EdgeType['Gateway'], # Choice your edge is Gateway or Device, Default is Gateway
  heartbeat = 60, # Default is 60 seconds
  dataRecover = True, # Need to recover data or not when disconnected
  connectType = constant.ConnectType['DCCS'], # Connection type (DCCS, MQTT), default is DCCS
  MQTT = MQTTOptions( # If connectType is MQTT, must fill this options
    hostName = "127.0.0.1",
    port = 1883,
    userName = "admin",
    password = "admin",
    protocalType = constant.Protocol['TCP'] # MQTT protocal (TCP, Websocket), default is TCP
  ),
  DCCS = DCCSOptions(
    apiUrl = "https://api-dccs.wise-paas.com/", # DCCS API Url
    credentialKey = "1e0e5365c3af88ad3233336c23d43bav" # Creadential key
  )
)

edgeAgent = EdgeAgent( options );
```

### 2. Event
EdgeAgent 有三種事件供訂閱，分別如下:
* on_connected: 當 EdgeAgent 成功連上 Broker 後觸發
* on_disconnected: 當 EdgeAgent 連線中斷後觸發
* on_message: 當 EdgeAgent 接收到 MQTT 訊息後觸發，根據 Constants.MessageType 可以分成以下訊息類型
  * WriteValue: Cloud 端改變 Tag 值同步到 Edge 端
  * WriteConfig: Cloud 端改變 Config 同步到 Edge 端
  * TimeSync: Cloud端回傳目前時間給Edge端，讓Edge端更新OS時間使時間一致
  * ConfigAck: Cloud 端接收 Edge 端 Config 同步的結果回應

```
edgeAgent.on_connected = edgeAgent_on_connected
edgeAgent.on_disconnected = edgeAgent_on_disconnected
edgeAgent.on_message = edgeAgent_on_message

def edgeAgent_on_connected(agent, isConnected):
  print("Connect success")

def edgeAgent_on_disconnected(agent, isDisconnected):
  print("Disconnected")

def edgeAgent_on_message(agent, messageReceivedEventArgs):
  # messageReceivedEventArgs format: Model.Event.MessageReceivedEventArgs
  type = messageReceivedEventArgs.type
  message = messageReceivedEventArgs.message
  if type == constants.MessageType['WriteValue']:
    # message format: Model.Edge.WriteValueCommand
    for device in message.deviceList:
      print("deviceId: {0}".format(device.id))
      for tag in device.tagList:
        print("tagName: {0}, Value: {1}".format(tag.name, str(tag.value)))
  elif type == constants.MessageType['WriteConfig']:
    print('WriteConfig')
  elif type == constants.MessageType['TimeSync']:
    # message format: Model.Edge.TimeSyncCommand
    print(str(message.UTCTime))
  elif type == constants.MessageType['ConfigAck']:
    # message format: Model.Edge.ConfigAck
    print("Upload Config Result: {0}}.format(str(message.result)))
```
