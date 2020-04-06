# Instructions {#development-environement}

---

## EdgeAgent {#edgeagent}

### 1. New EdgeAgent\(Options\)

New an edgeAent object.

```js
const edgeSDK = require('wisepaas-datahub-edge-nodejs-sdk');

const options = {
  connectType: edgeSDK.constant.connectType.DCCS,
  DCCS: {
    credentialKey: 'adcekc0f74837e3lqtdlkgfazn4i6qv8',
    APIUrl: 'https://api-dccs-ensaas.sa.wise-paas.com/'
  },
  // MQTT: {
  //   hostName: '127.0.0.1',
  //   port: 1883,
  //   username: 'admin',
  //   password: 'admin',
  //   protocolType: edgeSDK.constant.protocol.TCP
  // },
  useSecure: false,
  autoReconnect: true,
  reconnectInterval: 1000,
  nodeId: '6579dba6-9d19-44a2-b645-c2319c8f1315', // getting from datahub portal
  type: edgeSDK.constant.edgeType.Gateway, // Choice your edge is Gateway or Device, Default is Gateway
  deviceId: 'Device1', // If type is Device, DeviceId must be filled
  heartbeat: 60000, // default is 60 seconds,
  dataRecover: true, // need to recover data or not when disconnected
  ovpnPath: ''// set the path of your .ovpn file, and only for linux os
};
const edgeAgent = new edgeSDK.EdgeAgent(options);
```

Edge SDK porvides paramters for user to set options.  The way to access the parameters is shown below:

```js
const edgeSDK = require('wisepaas-datahub-edge-nodejs-sdk');
console.log(edgeSDK.constant); // edgeSDK.constant includes all parameters of edge sdk.

// All parameters which are included in edgeSDK.constant are shown below
  edgeType: {
    Gateway: 1,
    Device: 2
  },
  connectType: {
    MQTT: 1,
    DCCS: 2
  },
  protocol: {
    TCP: 1,
    WebSocket: 2
  },
  actionType: {
    create: 1,
    update: 2,
    delete: 3
  },
  NODEConfigType: {
    NODE: 1,
    gateway: 2,
    virtualGroup: 3
  },
  TagType: {
    Analog: 1,
    Discrete: 2,
    Text: 3
  },
  status: {
    Offline: 0,
    Online: 1
  },
  MessageType: {
    WriteValue: 0,
    WriteConfig: 1,
    TimeSync: 2,
    ConfigAck: 3
  }
```

### 2. Event

EdgeAgent has three event for subscribing.

* connected: When EdgeAgent is connected to IoTHub.
* disconnected: When EdgeAgent is disconnected to IoTHub.
* messageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:

  * WriteValue: Change tag value from cloud.

  * ConfigAck: The response of uploading config from edge to cloud.

```js
edgeAgent.events.on('connected', () => {
  console.log('Connect success !');
});
edgeAgent.events.on('disconnected', () => {
  console.log('Disconnected... ');
});
edgeAgent.events.on('messageReceived', (msg) => {
  switch (msg.type) {
    case edgeSDK.constant.MessageType.WriteValue:
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
    case edgeSDK.constant.MessageType.ConfigAck:
      console.log('Upload Config Result: ' + msg.message);
      break;
  }
});
```

### 3. connect\(\[callback\]\)

Connect to IoTHub. When connect success, the connected event will be triggered.

connect\(\[callback\]\) supports both promise and callback.

```js
edgeAgent.connect();
```

* Promise example

```js
edgeAgent.connect().then((result) => {
  //if connect successfully, result return true, and vice versa.
  //do something...
},
error => {
  //if connect unsuccessfully, return an error object which contains an error message.
  //do something...
})
```

* Callback example

```js
function costumerCallback(error,result){
//if connect successfully without error, error return null, result return true, and vice versa.
//do something...
}
edgeAgent.connect(costumerCallback);
```

### 4. disconnect\(\[callback\]\)

Disconnect to IoTHub. When disconnect success, the disconnected event will be triggered.

disconnect\(\[callback\]\) supports both promise and callback.

```js
edgeAgent.disconnect();
```

* Promise example

```js
edgeAgent.disconnect().then((result) => {
  //if disconnect successfully, result return true, and vice versa.
  //do something...
},
error => {
  //if disconnect unsuccessfully, return an error object which contains an error message.
  //do something...
})
```

* Callback example

```js
function costumerCallback(error,result){
//if disconnect successfully without error, error return null, result return true, and vice versa.
//do something...
}
edgeAgent.disconnect(costumerCallback);
```

### 5. uploadConfig\(action, edgeConfig, \[callback\]\)

Upload Node/Device/Tag Config with Action Type \(Create/~~Update/Delete~~\).

uploadConfig\(action, edgeConfig\) supports both promise and callback.

* Promise example

```js
const edgeConfig = new edgeSDK.EdgeAgent.EdgeConfig();
// set node condig
// set device config
// set tag config

edgeAgent.uploadConfig(actionType.create, edgeConfig).then(
result => {
    //if upload successfully result return true, and vice versa.
},
error => {
    //if upload config unsuccessfully, return an error object which contains an error message.
});
```

* Callback example

```js
const edgeConfig = new edgeSDK.EdgeAgent.EdgeConfig();
// set node condig
// set device config
// set tag config

function customerCallback(error, result){
//if upload successfully without error, error return null, result return true, and vice versa.
//do something...
}

edgeAgent.uploadConfig(actionType.create, edgeConfig, customerCallback);
```

Node Config:

```js
const nodeConfig = new edgeSDK.EdgeAgent.NodeConfig();

// this is required property below
nodeConfig.name = 'Test Node'; 
nodeConfig .description = 'Test Node';

edgeConfig.node= nodeConfig;
```

If you do not need the optional properties, you can skip to set the properties.

Device Config:

```js
const deviceConfig = new edgeSDK.EdgeAgent.DeviceConfig();

// these are required properties below
deviceConfig.id = 'Device1'; 
deviceConfig.name = 'Device 1'; 
deviceConfig.type = 'Smart Device'; 

// these are optional properties below
deviceConfig.description = 'Device 1'; 
deviceConfig.retentionPolicyName = ''; //retention policy name
```

If you do not need the optional properties, you can skip to set the properties.

Analog Tag Config:

```js
const analogTagConfig = new edgeSDK.EdgeAgent.AnalogTagConfig();
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

```js
const discreteTagConfig = new edgeSDK.EdgeAgent.DiscreteTagConfig();
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

```js
const textTagConfig = new edgeSDK.EdgeAgent.TextTagConfig();
let tTagList = [];

textTagConfig.name = 'TTag1';
textTagConfig.description = 'TTag1';
textTagConfig.readyOnly = false;
textTagConfig.arraySize = 0;

tTagList.push(textTagConfig)
```

Finally, add tag list to device config and add device to node config:

```js
deviceConfig.analogTagList = anaTagList; // add analog tag list to device.analogTagList
deviceConfig.discreteTagList = disTagList; // add discrete tag list to device.discreteTagList
deviceConfig.textTagList = tTagList; // add text tag list to device.textTagList 

edgeConfig.node.deviceList.push(deviceConfig) // add the device config to node.deviceList
```

### 6. sendData\(data, \[callback\]\)

Send tag value to cloud.

sendData\(data, \[callback\]\) supports both promise and callback.

* Promise example

```js
const data = new edgeSDK.EdgeAgent.EdgeData();

  for (let i = 1; i <= 2; i++) {
    for (let j = 1; j <= 5; j++) {
      const ATag = new edgeSDK.EdgeAgent.Tag();
      ATag.deviceId = 'Device' + i;
      ATag.tagName = 'ATag' + j;
      ATag.value = Math.floor(Math.random() * 100) + 1;

      const DTag = new edgeSDK.EdgeAgent.Tag();
      DTag.deviceId = 'Device' + i;
      DTag.tagName = 'DTag' + j;
      DTag.value = j % 2;

      const TTag = new edgeSDK.EdgeAgent.Tag();
      TTag.deviceId = 'Device' + i;
      TTag.tagName = 'TTag' + j;
      TTag.value = 'TEST' + j.toString();

      data.tagList.push(ATag);
      data.tagList.push(DTag);
      data.tagList.push(TTag);
    }
  }

edgeAgent.sendData(data).then(
result => {
  //if send data successful, result return true, and vice versa.
}, 
error => {
 //if send data unsuccessfully, return an error object which contains an error message.
});
```

* Callback example

```js
const data = new edgeSDK.EdgeAgent.EdgeData();

  for (let i = 1; i <= 2; i++) {
    for (let j = 1; j <= 5; j++) {
      const ATag = new edgeSDK.EdgeAgent.Tag();
      ATag.deviceId = 'Device' + i;
      ATag.tagName = 'ATag' + j;
      ATag.value = Math.floor(Math.random() * 100) + 1;

      const DTag = new edgeSDK.EdgeAgent.Tag();
      DTag.deviceId = 'Device' + i;
      DTag.tagName = 'DTag' + j;
      DTag.value = j % 2;

      const TTag = new edgeSDK.EdgeAgent.Tag();
      TTag.deviceId = 'Device' + i;
      TTag.tagName = 'TTag' + j;
      TTag.value = 'TEST' + j.toString();

      data.tagList.push(ATag);
      data.tagList.push(DTag);
      data.tagList.push(TTag);
    }
  }
function customerCallback(error, result){
//if send data successful without error, error return null, result return true, and vice versa.
//do something...
}

edgeAgent.sendData(data, customerCallback);
```

### 7. sendDeviceStatus\(devieStatus, \[callback\]\)

Send Device status to cloud when status changed.

sendDeviceStatus\(devieStatus, \[callback\]\) supports both promise and callback.

* Promise example

```js
const devieStatus = new edgeSDK.EdgeAgent.EdgeDeviceStatus();

  for (let i = 1; i <= 2; i++) {
    const device = new edgeSDK.EdgeAgent.DeviceStatus();
    device.id = 'Device' + i; 
    device.status = 1;// offline = 0, online = 1
    devieStatus.deviceList.push(device);
  }

edgeAgent.sendDeviceStatus(devieStatus).then(
result => {
  //if send device status successfully, result return true, and vice versa.
},
error => {
  //if send device status unsuccessfully, return an error object which contains an error message.
}
);
```

* Callback example

```js
const devieStatus = new edgeSDK.EdgeAgent.EdgeDeviceStatus();

  for (let i = 1; i <= 2; i++) {
    const device = new edgeSDK.EdgeAgent.DeviceStatus();
    device.id = 'Device' + i;
    device.status = 1; // offline = 0, online = 1
    devieStatus.deviceList.push(device);
  }

function customerCallback(error, result){
  //if send device status successfully, error return null, and result return true, and vice versa.
}
edgeAgent.sendDeviceStatus(devieStatus, customerCallback);
```



