#  {#development-environement}

# Instructions {#development-environement}

---

## EdgeAgent

### 1. Constructor\(EdgeAgentOptions options\)

New a EdgeAgent object.

```cs
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

EdgeAgent has three event for subscribing.

* Connected: When EdgeAgent is connected to IoTHub.
* Disconnected: When EdgeAgent is disconnected to IoTHub.
* MessageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:
  * WriteValue: Change tag value from cloud.
  * WriteConfig: Change config from cloud.
  * TimeSync: Returns the current time from cloud.
  * ConfigAck: The response of uploading config from edge to cloud.

```cs
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

Connect to IoTHub. When connect success, the connected event will be triggered.

```
edgeAgent.Connect();
```

### 4. Disconnect\(\)

Disonnect to IoTHub. When disconnect success, the disconnected event will be triggered.

```
edgeAgent.Disconnect();
```

### 5. UploadConfig\( ActionType action, EdgeConfig edgeConfig \)

Upload SCADA/Device/Tag Config with Action Type \(Create/Update/Delete\).

```cs
EdgeConfig config = new EdgeConfig();
// set scada condig
// set device config
// set tag config
bool result = _edgeAgent.UploadConfig( ActionType.Create, config ).Result;
```

SCADA Config:

```
config.Scada = new EdgeConfig.ScadaConfig();
```

Device Config:

```cs
EdgeConfig.DeviceConfig device = new EdgeConfig.DeviceConfig()
{
    Id = "Device1",
    Name = "Device1",
    Type = "Smart Device 1",
    Description = "Device 1",
};
```

Analog Tag Config:

```cs
EdgeConfig.AnalogTagConfig analogTag = new EdgeConfig.AnalogTagConfig()
{
    Name = "Volt",
    Description = "Volt",
    ReadOnly = false,
    ArraySize = 0
    SpanHigh = 1000,
    SpanLow = 0,
    EngineerUnit = "V",
    IntegerDisplayFormat = 4,
    FractionDisplayFormat = 2
};
```

Discrete Tag Config:

```cs
EdgeConfig.DiscreteTagConfig discreteTag = new EdgeConfig.DiscreteTagConfig()
{
    Name = "DTag",
    Description = "DTag " + j,
    ReadOnly = false,
    ArraySize = 0
    State0 = "0",
    State1 = "1",
    State2 = "",
    State3 = "",
    State4 = "",
    State5 = "",
    State6 = "",
    State7 = ""
};
```

Text Tag Config:

```cs
EdgeConfig.TextTagConfig textTag = new EdgeConfig.TextTagConfig()
{
    Name = "Text",
    Description = "Text",
    ReadOnly = false,
    ArraySize = 0
};
```

### 6. SendData\( EdgeData data \)

Send tag value to cloud.

```cs
Random random = new Random();
EdgeData data = new EdgeData();
for ( int i = 1; i <= 2; i++ )
{
    for ( int j = 1; j <= 5; j++ )
    {
        EdgeData.Tag aTag = new EdgeData.Tag()
        {
            DeviceId = "Device" + i,
            TagName = "ATag" + j,
            Value = random.NextDouble()
        };
        EdgeData.Tag dTag = new EdgeData.Tag()
        {
            DeviceId = "Device" + i,
            TagName = "DTag" + j,
            Value = j % 2
        };
        EdgeData.Tag tTag = new EdgeData.Tag()
        {
            DeviceId = "Device" + i,
            TagName = "TTag" + j,
            Value = "TEST " + j.ToString()
        };
        data.TagList.Add( aTag );
        data.TagList.Add( dTag );
        data.TagList.Add( tTag );
    }
}
data.Timestamp = DateTime.Now;
bool result = edgeAgent.SendData( data ).Result;
```

An array tag value have to use Dictionary&lt;string, T&gt;, T is defined according to the tag type \(Analog: double, Discrete: int, Text: string\).

```cs
// analog array tag
Dictionary<string, double> dicVal = new Dictionary<string, double>();
dicVal.Add( "0", 0.5 );    // tag index is 0, and it's value is 0.5
dicVal.Add( "1", 1.42 );    // tag index is 1, and it's value is 1.42
dicVal.Add( "2", 2.89 );    // tag index is 2, and it's value is 2.89
EdgeData.Tag aTag = new EdgeData.Tag()
{
    DeviceId = "Device1",
    TagName = "ATag",
    Value = dicVal
};

// discrete array tag
Dictionary<string, int> dicVal = new Dictionary<string, int>();
dicVal.Add( "0", 0 );    // tag index is 0, and it's value is 0
dicVal.Add( "1", 1 );    // tag index is 1, and it's value is 1
dicVal.Add( "2", 2 );    // tag index is 2, and it's value is 2
EdgeData.Tag dTag = new EdgeData.Tag()
{
    DeviceId = "Device1",
    TagName = "DTag",
    Value = dicVal
};

// text array tag
Dictionary<string, string> dicVal = new Dictionary<string, string>();
dicVal.Add( "0", "zero" );    // tag index is 0, and it's value is "zero"
dicVal.Add( "1", "one" );    // tag index is 1, and it's value is "one"
dicVal.Add( "2", "two" );    // tag index is 2, and it's value is "two"
EdgeData.Tag tTag = new EdgeData.Tag()
{
    DeviceId = "Device1",
    TagName = "TTag",
    Value = dicVal
};
```

### 7. SendDeviceStatus\( EdgeDeviceStatus deviceStatus \)

Send Device status to cloud when status changed.

```cs
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

### 8. Property

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | Connection status \(read only\) |



