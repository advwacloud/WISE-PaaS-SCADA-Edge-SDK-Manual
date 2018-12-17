# 使用說明 {#development-environement}

---

## agent

### 1. NewAgent\( options agent.EdgeAgentOptions \)

New a EdgeAgent object.

```
options := agent.NewEdgeAgentOptions()  // set default value
options.ReconnectInterval = 1, // MQTT reconnect interval seconds, default is 1
options.ScadaID = "5095cf13-f005-4c81-b6c9-68cf038e2b87"  // Getting from SCADA portal
options.DeviceID = "deviceId  // If type is Device, DeviceId must be filled
options.Type = agent.EdgeType["Gateway"]  // Choice your edge is Gateway or Device, Default is Gateway
options.HeartBeatInterval = 60  // Default is 60 seconds
options.DataRecover = true  // Need to recover data or not when disconnected, default is true
options.ConnectType =  agent.ConnectType["DCCS"]  // Connection type (DCCS, MQTT), default is DCCS
options.UseSecure = false
options.MQTT = agent.MQTTOptions {  // If connectType is MQTT, must fill this options
  HostName:     "127.0.0.1",
  Port:         1883,
  UserName:     "admin",
  Password:     "admin",
  ProtocalType: Protocol["TCP"],  // MQTT protocal (TCP, Websocket), default is TCP
}
options.DCCS = agent.DCCSOptions {  // // If connectType is DCCS, must fill this options
  URL: "https://api-dccs.wise-paas.com/", // DCCS API Url
  Key: "1e0e5365c3af88ad3233336c23d43bav",  // Creadential key
}

edgeAgent := agent.NewAgent( options )
```

### 2. Event

EdgeAgent 有三種事件供訂閱，分別如下:

* OnConnect: 當 EdgeAgent 成功連上 Broker 後觸發
* OnDisconnect: 當 EdgeAgent 連線中斷後觸發
* OnMessageReceive: 當 EdgeAgent 接收到 MQTT 訊息後觸發，根據 agent.MessageType 可以分成以下訊息類型
  * WriteValue: Cloud 端改變 Tag 值同步到 Edge 端
  * TimeSync: Cloud端回傳目前時間給Edge端，讓Edge端更新OS時間使時間一致
  * ConfigAck: Cloud 端接收 Edge 端 Config 同步的結果回應

```go
edgeAgent.SetOnConnectHandler(onConnectHandler)
edgeAgent.SetOnDisconnectHandler(onDisconnectHandler)
edgeAgent.SetOnMessageReceiveHandler(onMessageReceiveHandler)

func onConnectHandler (a agent.Agent) {
  fmt.Println("Connect success")
}
func onDisconnectHandler (a agent.Agent) {
  fmt.Println("Connect success")
}
func onMessageReceiveHandler (args agent.MessageReceivedEventArgs) {
  msgType := args.Type
  msg := args.Message

  switch msgType {
    case MessageType["WriteValue"]:  // message format: WriteDataMessage
      for _, device := range message.DeviceList {
        fmt.Println("DeviceId: ", device.ID)
        for _, tag := range device.TagList {
          fmt.Println("TagName: ", tag.Name, ", Value: ", tag.Value)
        }
      }
    case MessageType["ConfigAck"]:  // message format: ConfigAckMessage
      fmt.Println(message.Result)
    case MessageType["TimeSync"]: //message format: TimeSyncMessage
      fmt.Println(message.UTCTime)
  }
}
```

### 3. Connect\(\)

與 MQTT Broker 連線，連線資訊為建構子的傳入參數 EdgeAgentOptions 取得，連線成功後會觸發 OnConnect 事件。

```go
edgeAgent.Connect();
```

### 4. Disconnect\(\)

與 MQTT Broker 連線，連線資訊為建構子的傳入參數 EdgeAgentOptions 取得，連線成功後會觸發 OnDisconnect 事件。

```go
edgeAgent.Disconnect();
```

### 5. UploadConfig\( action agent.Action, config agent.EdgeConfig \)

上傳SCADA/Device/Tag Config，並根據ActionType決定是Create/Update/Delete。

```go
config = agent.EdgeConfig()

result = edgeAgent.UploadConfig(agent.Action["Create"], config)
```

SCADA config setting

```go
scadaConfig = agent.NewScadaConfig("scadaName")
scadaConfig.SetDescription("For Test")
scadaConfig.SetPrimaryIP("127.0.0.1")
// scadaConfig.SetBackupIP()
scadaConfig.SetPrimaryPort(80)
// scadaConfig.SetBackupPort()
scadaConfig.SetType(agent.EdgeType["Gateway"])# EdgeType (Gatewat, Device)


// add scada config
config.Scada = scadaConfig
```

Device config setting

```go
deviceConfig = agent.NewDeviceConfig("deviceId")
deviceConfig.SetName("DeviceName")
deviceConfig.SetComPortNumber(0)
deviceConfig.SetType("Device Type")
deviceConfig.SetDescription("Description")
deviceConfig.SetIP("127.0.0.1")
deviceConfig.SetPort(0)

// add devices
scadaConfig.DeviceList = append(scadaConfig.DeviceList, deviceConfig)
```

Analog Tag config setting

```go
analogConfig = agent.NewAnaglogTagConfig("AnalogTag")
analogConfig.SetDescription("AnalogTag")
analogConfig.SetReadOnly(True)
analogConfig.SetArraySize(0)
analogConfig.SetSpanHigh(10)
analogConfig.SetSpanLow(0)
analogConfig.SetEngineerUnit("cm")
analogConfig.SetIntegerDisplayFormat(2)
analogConfig.SetFractionDisplayFormat(4)

// add tags
deviceConfig.AnalogTagList = append(deviceConfig.AnalogTagList, analogConfig)
```

Discrete Tag config setting

```go
discreteConfig = agent.NewDiscreteTagConfig("DiscreteTag")
discreteConfig.SetDescription("DiscreteTag")
discreteConfig.SetReadOnly(false)
discreteConfig.SetArraySize(2)
discreteConfig.SetState0("1")
discreteConfig.SetState1("0")
// discreteConfig.SetState2()
// discreteConfig.SetState3()
// discreteConfig.SetState4()
// discreteConfig.SetState5()
// discreteConfig.SetState6()
// discreteConfig.SetState7()
)

// add tags
deviceConfig.DiscreteTagList = append(deviceConfig.DiscreteTagList, discreteConfig)
```

Text Tag config setting

```go
textConfig = agent.NewTextTagConfig("TextTag")
textConfig.SetDescription("TextTag")
textConfig.SetReadOnly(true)
textConfig.SetArraySize(0)

// add tags
deviceConfig.TextTagList = append(deviceConfig.TextTagList, textConfig)
```

### 6. SendData\( data agent.EdgeData \)

上傳設備的Tag Value。

```go
edgeData := agent.EdgeData{
  TimeStamp: time.Now(),
}

// deviceNum: 3
// tagNum: 5
for i := 1; i < 4; i++ {
  for j := 1; j < 6; j++ {
    deviceId := fmt.Sprintf("%s%d", "Device", i)
    tagName := fmt.Sprintf("%s%d", "ATag" + j)
    value := random.uniform(0, 100)
    tag := agent.EdgeTag{
      DeviceID: deviceId,
      TagName: tagName,
      Value: value,
    }
    edgeData.TagList = append(edgeData.TagList, tag)
  }
  for j := 1; j < 6; j++ {
    deviceId := fmt.Sprintf("%s%d", "Device", i)
    tagName := fmt.Sprintf("%s%d", "DTag" + j)
    value := random.randint(0,99)
    value = value % 2
    tag := agent.EdgeTag{
      DeviceID: deviceId,
      TagName: tagName,
      Value: value,
    }
    edgeData.TagList = append(edgeData.TagList, tag)
  }
  for j := 1; j < 6; j++ {
    deviceId := fmt.Sprintf("%s%d", "Device", i)
    tagName := fmt.Sprintf("%s%d", "TTag" + j)
    value = random.uniform(0, 100)
    value = "TEST " + str(value)
    tag = EdgeTag(deviceId, tagName, value)
    tag := agent.EdgeTag{
      DeviceID: deviceId,
      TagName: tagName,
      Value: value,
    }
    edgeData.TagList = append(edgeData.TagList, tag)
  }
}

result = edgeAgent.SendData(edgeData)
```

若是測點是屬於Array tag，則測點的Value參數必須使用`Map[string]interface{}`, `interface{}`根據測點類型定義 \(Analog: double, Discrete: int, Text: string\)

```go
# analog array tag
dicVal := map[string]int {
  "0": 0.5,
  "1": 1.42,
}
tag := agent.EdgeTag{
  DeviceID: "deviceId", 
  TagName: "ATag",
  Value: dicVal
}

# discrete array tag
dicVal = map[string]int {
  "0": 0,
  "1": 1,
}
tag = agent.EdgeTag{
  DeviceID: "deviceId",
  TagName: "DTag",
  Value: dicVal.
}

# text array tag
dicVal = map[string]int {
  "0": "zero",
  "1": "one"
}
tag = agent.EdgeTag{
  DeviceID: "deviceId",
  TagName: "TTag",
  Value: dicVal
}
```

### 7. SendDeviceStatus\( status agent.EdgeDeviceStatus \)

上傳Device Status \(狀態有改變再送即可\)。

```go
status := agent.EdgeDeviceStatus{
  TimeStamp: time.Now(),
}

// deviceNum: 3
for i := 1; i < 4; i++ {
  s := {
    ID: fmt.Sprintf("%s%d", "Device", i),
    Status: agent.Status["Online"], // Online, Offline
  }
  status.DeviceList = append(status.DeviceList, s)
}

result = edgeAgent.SendDeviceStatus( status )
```

### 8. 屬性

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | 判斷連線狀態 \(read only\) |



