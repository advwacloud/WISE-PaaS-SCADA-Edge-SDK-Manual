#  {#development-environement}

# Instructions {#development-environement}

---

## EdgeAgent

### 1. Constructor\(EdgeAgentOptions options\)

New a EdgeAgent object.

```
EdgeAgentOptions options = new EdgeAgentOptions();

options.ConnectType = ConnectType.DCCS; // Connection Type (DCCS, MQTT), Default is DCCS

options.DCCS = new DCCSOptions("1e0e5365c3af88ad3233336c23d43bav", "https://api-dccs.wise-paas.com/");
options.MQTT = new MQTTOptions("127.0.0.1", 1883, "admin", "pwd", Protocol.TCP);

options.UseSecure = false;
options.AutoReconnect = true;

options.ScadaId = "5095cf13-f005-4c81-b6c9-68cf038e2b87"; // getting from SCADA portal
options.Type = EdgeType.Gateway; // Choice your edge is Gateway or Device, Default is Gateway
options.DeviceId = "SmartDevice1"; // If type is Device, DeviceId must be filled
options.Heartbeat = 60000; // default is 60 seconds,
options.DataRecover = true; // need to recover data or not when disconnected

options.AndroidPackageName = getPackageName(); // If OS is Android, you need to pass this parameter to use dataRecover

EdgeAgent edgeAgent  = new EdgeAgent(options, agentListener);
```

### 2. Event

EdgeAgent has three event for subscribing.

* Connected: When EdgeAgent is connected to IoTHub.
* Disconnected: When EdgeAgent is disconnected to IoTHub.
* MessageReceived: When EdgeAgent received MQTT message from cloud. The message type as follows:
  * WriteValue: Change tag value from cloud.
  * TimeSync: Returns the current time from cloud.
  * ConfigAck: The response of uploading config from edge to cloud.

```
EdgeAgentListener agentListener = new EdgeAgentListener() {
    @Override
    public void Connected(EdgeAgent agent, EdgeAgentConnectedEventArgs args) {
        System.out.println("Connected");
    }

    @Override
    public void Disconnected(EdgeAgent agent, DisconnectedEventArgs args) {
        System.out.println("Disconnected");
    }

    @Override
    public void MessageReceived(EdgeAgent agent, MessageReceivedEventArgs e) {
        System.out.println("MessageReceived");
        switch (e.Type) {
            case Const.MessageType.WriteValue:
                WriteValueCommand wvcMsg = (WriteValueCommand) e.Message;
                for (WriteValueCommand.Device device: wvcMsg.DeviceList) {
                    System.out.println("DeviceId: " + device.Id);
                    for (WriteValueCommand.Tag tag: device.TagList) {
                        System.out.printf("TagName: %s, Value: %s\n", tag.Name, tag.Value.toString());
                    }
                }
                break;
            case Const.MessageType.TimeSync:
                TimeSyncCommand tscMsg = (TimeSyncCommand) e.Message;
                System.out.println("UTC Time: " + tscMsg.UTCTime.toString());
                break;
            case Const.MessageType.ConfigAck:
                ConfigAck cfgAckMsg = (ConfigAck) e.Message;
                String result = cfgAckMsg.Result.toString();
                break;
        }
    }
};

EdgeAgent edgeAgent  = new EdgeAgent(options, agentListener);
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

```
EdgeConfig config = new EdgeConfig();
// set scada condig
// set device config
// set tag config
Boolean result = agent.UploadConfig(Const.ActionType.Create, config);
```

SCADA Config:

```
config.Scada = new EdgeConfig.ScadaConfig();
config.Scada.Name = "TEST_SCADA";
config.Scada.Description = "For Test";
```

Device Config:

```
EdgeConfig.DeviceConfig device = new EdgeConfig.DeviceConfig();

device.Id = "Device1";
device.Name = "Device1";
device.Type = "Smart Device 1";
device.Description = "Device1";
```

Analog Tag Config:

```
EdgeConfig.AnalogTagConfig analogTag = new EdgeConfig.AnalogTagConfig();

analogTag.Name = "Volt";
analogTag.Description = "Volt";
analogTag.ReadOnly = false;
analogTag.ArraySize = 0;
analogTag.SpanHigh = 1000.0;
analogTag.SpanLow = 0.0;
analogTag.EngineerUnit = "V";
analogTag.IntegerDisplayFormat = 4;
analogTag.FractionDisplayFormat = 2;
```

Discrete Tag Config:

```
EdgeConfig.DiscreteTagConfig discreteTag = new EdgeConfig.DiscreteTagConfig();

discreteTag.Name = "DTag";
discreteTag.Description = "DTag";
discreteTag.ReadOnly = false;
discreteTag.ArraySize = 0;
discreteTag.State0 = "0";
discreteTag.State1 = "1";
discreteTag.State2 = "";
discreteTag.State3 = "";
discreteTag.State4 = "";
discreteTag.State5 = "";
discreteTag.State6 = "";
discreteTag.State7 = "";
```

Text Tag Config:

```
EdgeConfig.TextTagConfig textTag = new EdgeConfig.TextTagConfig();

textTag.Name = "TTag";
textTag.Description = "TTag ";
textTag.ReadOnly = false;
textTag.ArraySize = 0;
```

### 6. SendData\( EdgeData data \)

Send tag value to cloud.

```
Random random = new Random();
EdgeData data = new EdgeData();
for (int i = 1; i <= 2; i++) {
    for (int j = 1; j <= 5; j++) {
        EdgeData.Tag aTag = new EdgeData.Tag(); {
            aTag.DeviceId = "Device" + i;
            aTag.TagName = "ATag" + j;
            aTag.Value = random.nextInt(100);
        }
        EdgeData.Tag dTag = new EdgeData.Tag(); {
            dTag.DeviceId = "Device" + i;
            dTag.TagName = "DTag" + j;
            dTag.Value = j % 2;
        }

        EdgeData.Tag tTag = new EdgeData.Tag(); {
            tTag.DeviceId = "Device" + i;
            tTag.TagName = "TTag" + j;
            tTag.Value = "TEST " + j;
        }

        data.TagList.add(aTag);
        data.TagList.add(dTag);
        data.TagList.add(tTag);
    }
}
data.Timestamp = new Date();
Boolean result = agent.SendData(data);
```

An array tag value have to use Dictionary&lt;string, T&gt;, T is defined according to the tag type \(Analog: double, Discrete: int, Text: string\).

```
// analog array tag
HashMap<String, Double> mapVal = new HashMap<String, Double>();
mapVal.put("0", 0.5); // tag index is 0, and it's value is 0.5
mapVal.put("1", 1.42); // tag index is 1, and it's value is 1.42
mapVal.put("2", 2.89); // tag index is 2, and it's value is 2.89
EdgeData.Tag aTag = new EdgeData.Tag();

aTag.DeviceId = "Device1";
aTag.TagName = "ATag1";
aTag.Value = mapVal;

// discrete array tag
HashMap<String, int> mapVal = new HashMap<String, int>();
mapVal.put("0", 0); // tag index is 0, and it's value is 0
mapVal.put("1", 1); // tag index is 1, and it's value is 1
mapVal.put("2", 2); // tag index is 2, and it's value is 2
EdgeData.Tag dTag = new EdgeData.Tag();

dTag.DeviceId = "Device1";
dTag.TagName = "DTag";
dTag.Value = mapVal;

// text array tag
HashMap<String, String> mapVal = new HashMap<String, String>();
mapVal.put("0", "zero"); // tag index is 0, and it's value is "zero"
mapVal.put("1", "one");  // tag index is 1, and it's value is "one"
mapVal.put("2", "two");  // tag index is 2, and it's value is "two"
EdgeData.Tag tTag = new EdgeData.Tag();

tTag .DeviceId = "Device1";
tTag .TagName = "TTag";
tTag .Value = mapVal;
```

### 7. SendDeviceStatus\( EdgeDeviceStatus deviceStatus \)

Send Device status to cloud when status changed.

```
EdgeDeviceStatus deviceStatus = new EdgeDeviceStatus();

for (int i = 1; i <= deviceCount; i++) {
    EdgeDeviceStatus.Device device = new EdgeDeviceStatus.Device();
    device.Id = "Device" + i;
    device.Status = Const.Status.Online;

    deviceStatus.DeviceList.add(device);
}

deviceStatus.Timestamp = new Date();
Boolean result = agent.SendDeviceStatus(deviceStatus);
```

### 8. IsConnected\(\)

Connection status

```
 Boolean status = IsConnected();
```

### 9. Data Recover on Android

* You must specify `AndroidPackageName`  of constructor to use dataRevocer on andorid.   
  Because Scada SDK is a JavaSE project, there is no way to use `getPackageName()`.
* **If it is in a non-android environment, you can use the datarecover function without any other settings.**

```
EdgeAgentOptions options = new EdgeAgentOptions();
// ...
options.AndroidPackageName = getPackageName();
EdgeAgent edgeAgent  = new EdgeAgent(options, agentListener);
```



