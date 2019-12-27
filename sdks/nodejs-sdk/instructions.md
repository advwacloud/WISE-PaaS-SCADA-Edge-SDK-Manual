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





