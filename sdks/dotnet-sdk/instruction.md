#  {#development-environement}

# 使用說明 {#development-environement}

---

## EdgeAgent

### 1. Constructor\(EdgeAgentOptions options\)

初始化 EdgeAgent 實例，並根據傳入參數 EdgeAgentOptions 建立 MQTT 連線客戶端以及 SCADA 相關設定。

```
EdgeAgentOptions options = new EdgeAgentOptions()
{
    ConnectType = ConnectType.DCCS,    // Connection Type (DCCS, MQTT), Default is DCCS
    DCCS = new DCCSOptions()    // If ConnectType is DCCS, must fill this options
    {
        CredentialKey = "1e0e5365c3af88ad3233336c23d43bav",    // Credential Key
        APIUrl = "https://api-dccs.wise-paas.com/"            // DCCS API Url
    },
    MQTT = new MQTTOptions()    // If ConnectType is MQTT, must fill this options
    {
        HostName = "127.0.0.1",
        Port = 1883,
        Username = "admin",
        Password = "admin",
        ProtocolType = Protocol.TCP
    },
    UseSecure = false,
    AutoReconnect = true,
    ReconnectInterval = 1000,
    ScadaId = "5095cf13-f005-4c81-b6c9-68cf038e2b87",    // getting from SCADA portal
    Type = EdgeType.Gateway,    // Choice your edge is Gateway or Device, Default is Gateway
    DeviceId = "SmartDevice1",    // If type is Device, DeviceId must be filled
    Heartbeat = 60000,   // default is 60 seconds,
    DataRecover = true    // need to recover data or not when disconnected
};
EdgeAgent edgeAgent = new EdgeAgent( options );
```

### 2. Event

EdgeAgent 有三種事件供訂閱，分別如下:

* Connected: 當 EdgeAgent 成功連上 Broker 後觸發
* Disconnected: 當 EdgeAgent 連線中斷後觸發
* MessageReceived: 當 EdgeAgent 接收到 MQTT 訊息後觸發，根據 MessageReceivedEventArgs.Type 可以分成以下訊息類型
  * WriteValue: Cloud 端改變 Tag 值同步到 Edge 端
  * WriteConfig: Cloud 端改變 Config 同步到 Edge 端
  * TimeSync: Cloud端回傳目前時間給Edge端，讓Edge端更新OS時間使時間一致
  * ConfigAck: Cloud 端接收 Edge 端 Config 同步的結果回應

```
edgeAgent.Connected += edgeAgent_Connected;
edgeAgent.Disconnected += edgeAgent_Disconnected;
edgeAgent.MessageReceived += edgeAgent_MessageReceived;

private void edgeAgent_Connected( object sender, EdgeAgentConnectedEventArgs e )
{
    // Connected
    Console.WriteLine( "Connect success !" );
}

private void edgeAgent_Disconnected( object sender, DisconnectedEventArgs e )
{
    // Disconnected
    Console.WriteLine( "Disconnected..." );
}

private void edgeAgent_MessageReceived( object sender, MessageReceivedEventArgs e )
{
    switch ( e.Type )
    {
        case MessageType.WriteValue:
            WriteValueCommand wvcMsg = ( WriteValueCommand ) e.Message;
            foreach ( var device in wvcMsg.DeviceList )
            {
                Console.WriteLine( "DeviceId: {0}", device.Id );
                foreach ( var tag in device.TagList )
                {
                    Console.WriteLine( "TagName: {0}, Value: {1}", tag.Name, tag.Value.ToString() );
                }
            }
            break;
        case MessageType.WriteConfig:
            break
        case MessageType.TimeSync:
            TimeSyncCommand tscMsg = ( TimeSyncCommand ) e.Message;
            Console.WriteLine( "UTC Time: {0}", tscMsg.UTCTime.ToString() );
            break;
        case MessageType.ConfigAck:
            ConfigAck cfgAckMsg = ( ConfigAck ) e.Message;
            Console.WriteLine( "Upload Config Result: {0}", cfgAckMsg.Result.ToString() );
            break;
    }
}
```

### 3. Connect\(\)

與 MQTT Broker 連線，連線資訊為建構子的傳入參數 EdgeAgentOptions 取得，連線成功後會觸發 Connected 事件。

```
edgeAgent.Connect();
```

### 4. Disconnect\(\)

與 MQTT Broker 連線，連線資訊為建構子的傳入參數 EdgeAgentOptions 取得，連線成功後會觸發 Disconnected 事件。

```
edgeAgent.Disconnect();
```

### 5. UploadConfig\( ActionType action, EdgeConfig edgeConfig \)

上傳SCADA/Device/Tag Config，並根據ActionType決定是Create/Update/Delete。

```
EdgeConfig config = new EdgeConfig();
// set scada condig
// set device config
// set tag config
bool result = _edgeAgent.UploadConfig( ActionType.Create, config ).Result;
```

SCADA Config設定

```
config.Scada = new EdgeConfig.ScadaConfig()
{
    Id = "5095cf13-f005-4c81-b6c9-68cf038e2b87",
    Name = "TEST_SCADA",
    Description = "For Test"
};
```

Device Config設定

```
EdgeConfig.DeviceConfig device = new EdgeConfig.DeviceConfig()
{
    Id = "Device1",
    Name = "Device1",
    Type = "Smart Device 1",
    Description = "Device 1",
};
```

Analog Tag Config設定

```
EdgeConfig.AnalogTagConfig analogTag = new EdgeConfig.AnalogTagConfig()
{
    Name = "Volt",
    Description = "Volt",
    ReadOnly = false,
    ArraySize = 0,
    AlarmStatus = false,
    SpanHigh = 1000,
    SpanLow = 0,
    EngineerUnit = "V",
    IntegerDisplayFormat = 4,
    FractionDisplayFormat = 2,
    HHPriority = 0,
    HHAlarmLimit = 0,
    HighPriority = 0,
    HighAlarmLimit = 0,
    LowPriority = 0,
    LowAlarmLimit = 0,
    LLPriority = 0,
    LLAlarmLimit = 0
};
```

Discrete Tag Config設定

```
EdgeConfig.DiscreteTagConfig discreteTag = new EdgeConfig.DiscreteTagConfig()
{
    Name = "DTag",
    Description = "DTag " + j,
    ReadOnly = false,
    ArraySize = 0,
    AlarmStatus = false,
    State0 = "0",
    State1 = "1",
    State2 = "",
    State3 = "",
    State4 = "",
    State5 = "",
    State6 = "",
    State7 = "",
    State0AlarmPriority = 0,
    State1AlarmPriority = 0,
    State2AlarmPriority = 0,
    State3AlarmPriority = 0,
    State4AlarmPriority = 0,
    State5AlarmPriority = 0,
    State6AlarmPriority = 0,
    State7AlarmPriority = 0
};
```

Text Tag Config設定

```
EdgeConfig.TextTagConfig textTag = new EdgeConfig.TextTagConfig()
{
    Name = "Text",
    Description = "Text",
    ReadOnly = false,
    ArraySize = 0,
    AlarmStatus = false
};
```

### 6. SendData\( EdgeData data \)

上傳設備的Tag Value。

```
Random random = new Random();
EdgeData data = new EdgeData();
for ( int i = 1; i <= 2; i++ )
{
    EdgeData.Device device = new EdgeData.Device()
    {
        Id = "Device" + i,
        TagList = new List<EdgeData.Tag>()
    };

    for ( int j = 1; j <= 5; j++ )
    {
        EdgeData.Tag aTag = new EdgeData.Tag()
        {
            Name = "ATag" + j,
            Value = random.NextDouble()
        };
        EdgeData.Tag dTag = new EdgeData.Tag()
        {
            Name = "DTag" + j,
            Value = j % 2
        };
        EdgeData.Tag tTag = new EdgeData.Tag()
        {
            Name = "TTag" + j,
            Value = "TEST " + j.ToString()
        };
        device.TagList.Add( aTag );
        device.TagList.Add( dTag );
        device.TagList.Add( tTag );
    }
    data.DeviceList.Add( device );
}
data.Timestamp = DateTime.Now;
bool result = edgeAgent.SendData( data ).Result;
```

若是測點是屬於Array tag，則測點的Value參數必須使用Dictionary&lt;string, T&gt;，T根據測點類型定義 \(Analog: double, Discrete: int, Text: string\)。

```
// analog array tag
Dictionary<string, double> dicVal = new Dictionary<string, double>();
dicVal.Add( "0", 0.5 );    // tag index is 0, and it's value is 0.5
dicVal.Add( "1", 1.42 );    // tag index is 1, and it's value is 1.42
dicVal.Add( "2", 2.89 );    // tag index is 2, and it's value is 2.89
EdgeData.Tag aTag = new EdgeData.Tag()
{
    Name = "ATag",
    Value = dicVal
};

// discrete array tag
Dictionary<string, int> dicVal = new Dictionary<string, int>();
dicVal.Add( "0", 0 );    // tag index is 0, and it's value is 0
dicVal.Add( "1", 1 );    // tag index is 1, and it's value is 1
dicVal.Add( "2", 2 );    // tag index is 2, and it's value is 2
EdgeData.Tag dTag = new EdgeData.Tag()
{
    Name = "DTag",
    Value = dicVal
};

// text array tag
Dictionary<string, string> dicVal = new Dictionary<string, string>();
dicVal.Add( "0", "zero" );    // tag index is 0, and it's value is "zero"
dicVal.Add( "1", "one" );    // tag index is 1, and it's value is "one"
dicVal.Add( "2", "two" );    // tag index is 2, and it's value is "two"
EdgeData.Tag tTag = new EdgeData.Tag()
{
    Name = "TTag",
    Value = dicVal
};
```

### 7. SendDeviceStatus\( EdgeDeviceStatus deviceStatus \)

上傳Device Status \(狀態有改變再送即可\)。

```
EdgeDeviceStatus deviceStatus = new EdgeDeviceStatus();
for ( int i = 1; i <= 2; i++ )
{
    EdgeDeviceStatus.Device device = new EdgeDeviceStatus.Device()
    {
        Id = "Device" + i,
        Status = Status.Online
    };
    deviceStatus.DeviceList.Add( device );
}
deviceStatus.Timestamp = DateTime.Now;
bool result = edgeAgent.SendDeviceStatus( deviceStatus ).Result;
```

### 8. 屬性

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | 判斷連線狀態 \(read only\) |



