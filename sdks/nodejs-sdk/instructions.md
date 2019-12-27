# Instructions {#development-environement}

---

## EdgeAgent {#edgeagent}

### 1. New EdgeAgent\(Options\)

```
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
```



