#  {#development-environement}

# Instructions {#development-environement}

---

## EdgeAgent

### 1. Load Library

Load dynamic library，and includ WISEPaas.h。Must follow the definition as follows when use API:

* EDGE\_AGENT\_OPTION.h: Define construct function structure
* EDGE\_CONFIG.h: Define UploadConfig structure
* EDGE\_DATA.h: Define SendData structure
* EDGE\_DEVICE\_STATUS.h: Define SendDeviceStatus structure

```
/*  load library */

#include "WISEPaaS.h"

void (*SetConnectEvent)();
void (*SetDisconnectEvent)();
void (*SetMessageReceived)();
void (*Constructor)(TOPTION_STRUCT);
void (*Connect)();
void (*Disconnect)();
int (*UploadConfig)(ActionType, TSCADA_CONFIG_STRUCT);
int (*SendData)(TEDGE_DATA_STRUCT);
int (*SendDeviceStatus)(TEDGE_DEVICE_STATUS_STRUCT);

char *error;

void *handle;
handle = dlopen ("./WISEPaaS.so.1.0.0", RTLD_LAZY);

if (!handle) {
    fputs (dlerror(), stderr);
    exit(1);
}

SetConnectEvent = dlsym(handle, "SetConnectEvent");
SetDisconnectEvent = dlsym(handle, "SetDisconnectEvent");
SetMessageReceived = dlsym(handle, "SetMessageReceived");

Constructor = dlsym(handle, "Constructor");
Connect = dlsym(handle, "Connect");
Disconnect = dlsym(handle, "Disconnect");
UploadConfig = dlsym(handle, "UploadConfig");
SendData = dlsym(handle, "SendData");
SendDeviceStatus = dlsym(handle, "SendDeviceStatus");

if ((error = dlerror()) != NULL)  {
    fputs(error, stderr);
    exit(1);
}
```

### 2. Constructor\(TOPTION\_STRUCT option\)

New a EdgeAgent object.

```
TOPTION_STRUCT options;
options.AutoReconnect = true;
options.ReconnectInterval = 1000;
options.ScadaId = "c9851920-ca7f-4cfd-964a-1969aef958f6";
options.Heartbeat = 60;
options.DataRecover = true;
options.ConnectType = DCCS;     // Connection Type (DCCS, MQTT), Default is DCCS
options.Type = Gatway;
options.UseSecure = false;

TMQTT_OPTION_STRUCT mqtt;

switch (options.ConnectType)
{
    case 1: // If ConnectType is DCCS, must fill this options
        options.DCCS.CredentialKey = "9b03e5524c70b6e4c503173c6553abrh"; // Credential Key
        options.DCCS.APIUrl = "https://api-dccs.wise-paas.com/";         // DCCS API Url
        break;

    case 0: // If ConnectType is MQTT, must fill this options
        options.MQTT.HostName = "";
        options.MQTT.Port = 1883;
        options.MQTT.Username = "";
        options.MQTT.Password = "";
        options.MQTT.ProtocolType = TCP;
        break;
}
Constructor(options);
```

### 3. Event

EdgeAgent has three event for subscribing.

* Connected: When EdgeAgent is connected to IoTHub.
* Disconnected: When EdgeAgent is disconnected to IoTHub.
* MessageReceived: When EdgeAgent receives MQTT message from cloud. The message type as follows:
  * WriteValue: Change tag value from cloud.
  * WriteConfig: Change config from cloud.
  * TimeSync: Returns the current time from cloud.
  * ConfigAck: The response of uploading config from edge to cloud.

```
void edgeAgent_Connected(){
    printf("Connect success\n");
    IsConnected = true;
}

void edgeAgent_Disconnected(){
    printf("Disconnected\n");
    IsConnected = false;
}

void edgeAgent_Recieve(char *cmd, char *val){

    if(strcmp(cmd, WirteValueCommand) == 0){
        printf("write value: %s\n", val);
    }
    else if(strcmp(cmd, WriteConfigCommand) == 0){
        printf("write config: %s\n", val);
    }
}

/*  Set Event */
SetConnectEvent(edgeAgent_Connected);
SetDisconnectEvent(edgeAgent_Disconnected);
SetMessageReceived(edgeAgent_Recieve);
```

### 4. Connect\(\)

Connect to IoTHub. When connect success, the connected event will be triggered.

```
Connect();
```

### 5. Disconnect\(\)

Disonnect to IoTHub. When disconnect success, the disconnected event will be triggered.

```
Disconnect();
```

### 6. UploadConfig\( ActionType action, TSCADA\_CONFIG\_STRUCT edgeConfig \)

Upload SCADA/Device/Tag Config with Action Type \(Create/Update/Delete\).

```
TSCADA_CONFIG_STRUCT config;
ActionType action = Create; // Create, Update od Delete
// set scada condig
// set device config
// set tag config
bool result = UploadConfig(action, config);
```

SCADA Config:

```
config.Id = Scada ID"; 
config.Description = "description";
config.Name = "For Test";
config.PrimaryPort = 1883;
config.BackupPort = 1883;
config.Type = 1;
```

Device Config:

```
PTDEVICE_CONFIG_STRUCT device = malloc(sizeof(struct DEVICE_CONFIG_STRUCT));

device.Name = simDevName;
device.Type = "DType";
device.Description = "DDESC";
device.IP = "127.0.0.1";
device.Port = 1;

config.DeviceNumber = device_num;
config.DeviceList = device;
```

Analog Tag Config:

```
PTANALOG_TAG_CONFIG analogTag = malloc(sizeof(struct ANALOG_TAG_CONFIG));

analogTag.Name = "TestName";    
analogTag.Description = "description";          
analogTag.ReadOnly = false;
analogTag.ArraySize = 0;
analogTag.SpanHigh = 1000;
analogTag.SpanLow = 0;
analogTag.EngineerUnit = "enuit";
analogTag.IntegerDisplayFormat = 4;
analogTag.FractionDisplayFormat = 2;
```

Discrete Tag Config:

```
PTDISCRETE_TAG_CONFIG discreteTag = malloc(sizeof(struct DISCRETE_TAG_CONFIG));

discreteTag.NAme = "TestName"
discreteTag.Description = "description";
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
PTTEXT_TAG_CONFIG textTag = malloc(sizeof(struct TEXT_TAG_CONFIG));

textTag.Name = "TestName";
textTag.Description = "description";
textTag.ReadOnly = false;
textTag.ArraySize = 0;
```

### 7. SendData\( EdgeData data \)

Send tag value to cloud.

```
TEDGE_DATA_STRUCT data;

PTEDGE_DEVICE_STRUCT data_device = malloc(sizeof(struct EDGE_DEVICE_STRUCT));
PTEDGE_TAG_STRUCT data_tag = malloc(sizeof(struct EDGE_TAG_STRUCT));

int value = rand()%1000;
char *simValue = NULL;

data_tag.Name = "tagName";
asprintf(&simValue, "%d", value);
data_tag.Value = simValue;

data_device.TagNumber = tag_num;
data_device.TagList = data_tag;
data_device.Id = "DeviceID";

data.DeviceNumber = 1;
data.DeviceList = data_device;

bool result = SendData(data);
```

### 8. SendDeviceStatus\( TEDGE\_DEVICE\_STATUS\_STRUCT deviceStatus \)

Send Device status to cloud when status changed.

```
TEDGE_DEVICE_STATUS_STRUCT status;
status.DeviceNumber = device_num;

PTDEVICE_LIST_STRUCT dev_list = malloc(sizeof(struct DEVICE_LIST_STRUCT));
asprintf(&simDevId, "%s", "DeviceID");
dev_list.Id = simDevId;
dev_list.Status = 1;

status.DeviceList = dev_list;

bool result = SendDeviceStatus(status);
```

### 9. Property

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | Connection status \(read only\) |



