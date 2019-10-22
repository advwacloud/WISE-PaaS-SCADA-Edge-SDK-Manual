# Instructions {#development-environement}

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

EdgeAgent has three event for subscribing.

* Connected: When EdgeAgent is connected to IoTHub.
* Disconnected: When EdgeAgent is disconnected to IoTHub.
* MessageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:
  * WriteValue: Change tag value from cloud.
  * WriteConfig: Change config from cloud.
  * TimeSync: Returns the current time from cloud.
  * ConfigAck: The response of uploading config from edge to cloud.

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

Connect to IoTHub. When connect success, the connected event will be triggered.

```go
edgeAgent.Connect();
```

### 4. Disconnect\(\)

Disconnect to IoTHub. When disconnect success, the disconnected event will be triggered.

```go
edgeAgent.Disconnect();
```

### 5. UploadConfig\( action agent.Action, config agent.EdgeConfig \)

Upload SCADA/Device/Tag Config with Action Type \(Create/Update/Delete\).

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

Send tag value to cloud.

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

An array tag value have to use Dictionary&lt;string, T&gt;, T is defined according to the tag type \(Analog: double, Discrete: int, Text: string\).

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

Send Device status to cloud when status changed.

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

### 8. Property

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | Connection Status \(read only\) |



