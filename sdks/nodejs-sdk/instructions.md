# Instructions {#development-environement}

---

## EdgeAgent {#edgeagent}

### 1. New EdgeAgent\(Options\)

New  an edgeAent object.

```
const EdgeSDK= require('');
const options = {
  connectType: 1, // MQTT=0 DCCS=1
  DCCS: {
    credentialKey: '1e0e5365c3af88ad3233336c23d43bav',
    APIUrl: 'https://api-dccs.wise-paas.com/'
  },
  // MQTT: {
  //   hostName: '127.0.0.1',
  //   port: 1883,
  //   username: 'admin',
  //   password: 'admin',
  //   protocolType: 0
  // },
  useSecure: false,
  autoReconnect: true,
  reconnectInterval: 1000,
  scadaId: '5095cf13-f005-4c81-b6c9-68cf038e2b87', // getting from SCADA portal
  type: 0, // Choice your edge is Gateway or Device, Default is Gateway
  deviceId: 'Device1', // If type is Device, DeviceId must be filled
  heartbeat: 60000, // default is 60 seconds,
  dataRecover: true // need to recover data or not when disconnected
};
const edgeAgent = new EdgeSDK.EdgeAgent(options);
```

### 2. Event

EdgeAgent has three event for subscribing.

* connected: When EdgeAgent is connected to IoTHub.
* disconnected: When EdgeAgent is disconnected to IoTHub.
* messageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:

  * WriteValue: Change tag value from cloud.

  * ConfigAck: The response of uploading config from edge to cloud.

```
edgeAgent.connected = edgeAgentConnected;
edgeAgent.disconnected = edgeAgentDisconnected;
edgeAgent.messageReceived = edgeAgentMessageReceived;

function edgeAgentConnected () {
    console.log('Connect success !');
}

function edgeAgentDisconnected () {
  console.log('Disconnected... ');
}

function edgeAgentMessageReceived (msg) {
  switch (msg.type) {
    case MessageType.WriteValue:
      for (const device of msg.message.deviceList) {
        console.log('DeviceId: ' + device.id);
        for (const tag of device.tagList) {
          if (typeof tag.value === 'object') {
            for (const aryTag in tag.value) {
              console.log('TagName: ' + tag.name + ', Index: ' + aryTag + ', Value: ' + tag.value[aryTag]);
            }
          } else {
            console.log('TagName: ' + tag.name + ', Value: ' + tag.value);
          }
        }
      }
      break;
    case MessageType.ConfigAck:
      console.log('Upload Config Result: ' + msg.message);
      break;
  }
}
```

### 3. Connect\(\)

Connect to IoTHub. When connect success, the connected event will be triggered.

```
edgeAgent.connect();
```

### 4. Disconnect\(\)

Disconnect to IoTHub. When disconnect success, the disconnected event will be triggered.

```
edgeAgent.disconnect();
```

### 5. UploadConfig\(action, edgeConfig\)

Upload SCADA/Device/Tag Config with Action Type \(Create/Update/Delete\).

```
const edgeConfig = new EdgeSDK.EdgeAgent.EdgeConfig();
// set scada condig
// set device config
// set tag config

edgeAgent.uploadConfig(actionType.create, edgeConfig).then(
res => {
    //if upload successful res return true
},
error => {
    //if upload config occur any exception return error
    console.log(error);//show the error message of the exception
});
```

SCADA Config:

```
edgeConfig.scada =  new EdgeSDK.EdgeAgent.ScadaConfig()
```

Device Config:

```
const deviceConfig = new EdgeSDK.EdgeAgent.DeviceConfig();

deviceConfig.id = 'Device1';
deviceConfig.name = 'Device 1';
deviceConfig.type = 'Smart Device';
deviceConfig.description = 'Device 1';
```

Analog Tag Config:

```
const analogTagConfig = new EdgeSDK.EdgeAgent.AnalogTagConfig();
let anaTagList = [];

analogTagConfig.name = 'ATag1';
analogTagConfig.description = ' ATag1';
analogTagConfig.readOnly = false;
analogTagConfig.arraySize = 0;
analogTagConfig.spanHigh = 1000;
analogTagConfig.spanLow = 0;
analogTagConfig.engineerUnit = '';
analogTagConfig.integerDisplayFormat = 4;
analogTagConfig.fractionDisplayFormat = 2;

anaTagList.push(analogTagConfig);
```

Discrete Tag Config:

```
const discreteTagConfig = new EdgeSDK.EdgeAgent.DiscreteTagConfig();
let disTagList = []; 

discreteTagConfig.name = 'DTag1';
discreteTagConfig.description  = 'DTag1';
discreteTagConfig.arraySize = 0;
discreteTagConfig.state0 = '0';
discreteTagConfig.state1 = '1';
discreteTagConfig.state2 = 'NotUsed';
discreteTagConfig.state3 = 'NotUsed';
discreteTagConfig.state4 = 'NotUsed';
discreteTagConfig.state5 = 'NotUsed';
discreteTagConfig.state6 = 'NotUsed';
discreteTagConfig.state7 = 'NotUsed';

disTagList.push(discreteTagConfig);
```

Text Tag Config:

```
const textTagConfig = new EdgeSDK.EdgeAgent.TextTagConfig();
let tTagList = [];

textTagConfig.name = 'TTag1';
textTagConfig.description = 'TTag1';
textTagConfig.readyOnly = false;
textTagConfig.arraySize = 0;

tTagList.push(textTagConfig)
```

Finally, add tag list to device config, and add deive to scada config:

```
deviceConfig.analogTagList = anaTagList; // add analog tag list to device.analogTagList
deviceConfig.discreteTagList = disTagList; // add discrete tag list to device.discreteTagList
deviceConfig.textTagList = tTagList; // add text tag list to device.textTagList 

edgeConfig.scada.deviceList.push(deviceConfig) // add the device config to scada.deviceList
```



