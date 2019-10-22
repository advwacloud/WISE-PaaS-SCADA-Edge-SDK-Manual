# Instructions {#development-environement}

---

## EdgeAgent

### 1. Constructor\(EdgeAgentOptions options\)

New a EdgeAgent object.

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

edgeAgent = EdgeAgent( options = options );
```

### 2. Event

EdgeAgent has three event for subscribing.

* Connected: When EdgeAgent is connected to IoTHub.
* Disconnected: When EdgeAgent is disconnected to IoTHub.
* MessageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:
  * WriteValue: Change tag value from cloud.
  * WriteConfig: Change config from cloud.
  * TimeSync: Returns the current time from cloud.
  * ConfigAck: The response of uploading config from edge to cloud.

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

### 3. Connect\(\)

Connect to IoTHub. When connect success, the connected event will be triggered.

```
edgeAgent.connect();
```

### 4. Disconnect\(\)

Disconnect to IoTHub. When disconnect success, the disonnected event will be triggered.

```
edgeAgent.disconnect();
```

### 5. UploadConfig\( ActionType action, EdgeConfig edgeConfig \)

Upload SCADA/Device/Tag Config with Action Type \(Create/Update/Delete\).

```
config = EdgeConfig()
# set scada config
# set device config
# set tag config

result = edgeAgent.uploadConfig(constants.ActionType['Create'], edgeConfig = config)
```

SCADA config setting

```
scadaConfig = ScadaConfig(
  name = 'scadaName',
  description = 'For Test',
  primaryIP = None,
  backupIP = None,
  primaryPort = None,
  backupPort = None,
  scadaType = constant.EdgeType['Gateway'] # EdgeType (Gatewat, Device)
)
config.scada = scadaConfig
```

Device config setting

```
deviceConfig = DeviceConfig(
  id = 'DeviceId',
  name = 'DeviceName',
  comPortNumber = None,
  deviceType = 'Device Type',
  description = 'Description',
  ip = None,
  port = None
)
config.scada.deviceList.append(deviceConfig)
```

Analog Tag config setting

```
analogTag = AnalogTagConfig(
  name = 'AnalogTag',
  description = 'AnalogTag',
  readOnly = True,
  arraySize = 0,
  spanHigh = 10,
  spanLow = 0,
  engineerUnit = 'cm',
  integerDisplayFormat = 2,
  fractionDisplayFormat = 4
)
config.scada.deviceList[0].analogTagList.append(analogTag)
```

Discrete Tag config setting

```
discreteTag = DiscreteTagConfig(
  name = 'DiscreteTag',
  description = 'DiscreteTag',
  readOnly = False,
  arraySize = 2,
  state0 = '1',
  state1 = '0',
  state2 = None,
  state3 = None,
  state4 = None,
  state5 = None,
  state6 = None,
  state7 = None
)
config.scada.deviceList[0].discreteTagList.append(discreteTag)
```

Text Tag config setting

```
textTag = TextTagConfig(
  name = 'TextTag',
  description = 'TextTag',
  readOnly = True,
  arraySize = 0
)
config.scada.deviceList[0].textTagList.append(textTag)
```

### 6. SendData\( EdgeData data \)

Send tag value to cloud.

```
edgeData = EdgeData()
for i in range(1, 3):
  for j in range(1, 5):
    deviceId = 'Device' + str(i)
    tagName = 'ATag' + str(j)
    value = random.uniform(0, 100)
    tag = EdgeTag(deviceId, tagName, value)
    edgeData.tagList.append(tag)
  for j in range(1, 5):
    deviceId = 'Device' + str(i)
    tagName = 'DTag' + str(j)
    value = random.randint(0,99)
    value = value % 2
    tag = EdgeTag(deviceId, tagName, value)
    edgeData.tagList.append(tag)
  for j in range(1, 5):
    deviceId = 'Device' + str(i)
    tagName = 'TTag' + str(j)
    value = random.uniform(0, 100)
    value = 'TEST ' + str(value)
    tag = EdgeTag(deviceId, tagName, value)
    edgeData.tagList.append(tag)

result = edgeAgent.sendData(data = edgeData)
```

An array tag value have to use Dictionary&lt;string, T&gt;, T is defined according to the tag type \(Analog: double, Discrete: int, Text: string\).

```
# analog array tag
dicVal = {
  "0": 0.5,
  "1": 1.42
};
tag = EdgeTag(deviceId = 'deviceId', tagName = 'ATag', value = dicVal)

# discrete array tag
dicVal = {
  "0": 0,
  "1": 1
};
tag = EdgeTag(deviceId = 'deviceId', tagName = 'DTag', value = dicVal)

# text array tag
dicVal = {
  "0": 'zero',
  "1": 'one'
};
tag = EdgeTag(deviceId = 'deviceId', tagName = 'TTag', value = dicVal)
```

### 7. SendDeviceStatus\( EdgeDeviceStatus deviceStatus \)

Send Device status to cloud when status changed.

```
deviceStatus = EdgeDeviceStatus()
for i in range(1, 3):
  id = 'deviceId' + str(i)
  status = EdgeStatus(id = id, status = constant.Status['Online'])

  deviceStatus.deviceList.append( status )

result = edgeAgent.sendDeviceStatus( deviceStatus )
```

### 8. Property

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| isConnected | boolean | Connection Status \(read only\) |



