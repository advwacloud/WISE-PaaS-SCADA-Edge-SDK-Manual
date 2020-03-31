#  {#development-environement}

# Instructions {#development-environement}

---

## EdgeAgent

### 1. Load Library

Load shared library and include the header file named DatahubEdge.h
This header file contains the other statements as below:

* EDGE\_AGENT\_OPTION.h:  Define construct function structure
* EDGE\_CONFIG.h: 		  Define UploadConfig structure
* EDGE\_DATA.h: 		  Define SendData structure>
* EDGE\_DEVICE\_STATUS.h: Define SendDeviceStatus structure

```C
/*  load shared object library (DatahubEdge.so.1.0.2) */
#include "DatahubEdge.h"

char *error;
void *handle;
handle = dlopen ("./DatahubEdge.so.1.0.2", RTLD_LAZY);
if (!handle) {
    fputs (dlerror(), stderr);
    exit(1);
}
```

Load all of the EdgeAgent function declarations

```C
void (*SetConnectEvent)();
void (*SetDisconnectEvent)();
void (*SetMessageReceived)();
void (*Constructor)(TOPTION_STRUCT);
void (*Connect)();
void (*Disconnect)();
int  (*UploadConfig)(ActionType, TSCADA_CONFIG_STRUCT);
int  (*SendData)(TEDGE_DATA_STRUCT);
int  (*SendDeviceStatus)(TEDGE_DEVICE_STATUS_STRUCT);

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

### 2. Constructor\(TOPTION\_STRUCT options\)

Construct EdgeAgent object by using a symbolic structure which refer to TOPTION_STRUCT 

```C
/*  Set Construct */
TOPTION_STRUCT options;
```
This header shows how the members of this object act.

inc/EDGE_AGENT_OPTION.h
```C
typedef struct OPTION_STRUCT {
	bool AutoReconnect;
	int ReconnectInterval;
	char * NodeId;
	char * DeviceId;
	EdgeType Type;
	int Heartbeat;
	bool DataRecover;
	ConnectType ConnectType;
	bool UseSecure;
	char * OvpnPath;
	TMQTT_OPTION_STRUCT MQTT;
	TDCCS_OPTION_STRUCT DCCS;
} TOPTION_STRUCT, * PTOPTION_STRUCT;
```

Declare Connection Variable
```C
options.AutoReconnect = true;
options.ReconnectInterval = 1000;
options.NodeId = "YOUR_NODE_ID"; 	// getting from SCADA portal
options.Heartbeat = 60; 			// default is 60 seconds
options.DataRecover = true;			// need to recover data or not when disconnected
options.ConnectType = MQTT; 		// set your connect type: DCCS or MQTT
options.Type = Gatway;				// Choice your edge is Gateway or Device, Default is Gateway
options.UseSecure = false;
options.OvpnPath = "Your_OVPN_FILE_PATH//YOUR_OVPN_FILE_NAME";
	
TMQTT_OPTION_STRUCT mqtt;

switch (options.ConnectType)
{
    case 1: // If ConnectType is DCCS, must fill this options
        options.DCCS.CredentialKey = "9b03e5524c70b6e4c503173c6553abrh"; // Credential Key
        options.DCCS.APIUrl = "https://api-dccs.wise-paas.com/";         // DCCS API Url
        break;

    case 0: // If ConnectType is MQTT, must fill this options
        options.MQTT.HostName = "127.0.0.1";
        options.MQTT.Port = 1883;
        options.MQTT.Username = "admin";
        options.MQTT.Password = "admin";
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

```C
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

### 4. Using OpenVPN Client to connect \(\)

To connect a VPN connection, start OpenVPN Connect, select an imported .ovpn file
You can use the property called "options.OvpnPath" to determine whether the OpenVPN Connect or not.

```
TOPTION_STRUCT options; // your edge object
options.OvpnPath = "Your_OVPN_FILE_PATH//YOUR_OVPN_FILE.ovpn"; // use openvpn connect
options.OvpnPath = ""; // do not use openvpn connect
```


### 5. Connect\(\)

Connect to IoTHub. When connect success, the connected event will be triggered.

```
Connect();
```

### 6. Disconnect\(\)

Disonnect to IoTHub. When disconnect success, the disconnected event will be triggered.

```
Disconnect();
```

### 7. UploadConfig\( ActionType action, TNODE\_CONFIG\_STRUCT edgeConfig \)

Upload Node/Device/Tag Config with Action Type \(Create/Update/Delete\).



```C
TNODE_CONFIG_STRUCT config;
ActionType action = Create; // Create, Update or Delete

/* Set Edge Config here */
// ---- Ref 7.1.Set node condig 
// ---- Ref 7.2.Set device config 
//      |---- Ref 7.3.Set tag config:
//          |--Ref 7.3.1 Set analog tag config
//          |--Ref 7.3.2 Set discrete tag config 
//          |--Ref 7.3.3 Set text tag config 

bool result = UploadConfig(action, config);
```

##### 7.1. Set NODE Config:

```C
TNODE_CONFIG_STRUCT config;
ActionType action = Create;

config.Id = options.NodeId; 
config.Description = "YOUR_NODE_DESCRIPTION";
config.Name = "YOUR_NODE_NAME";
config.Type = 1;
```

##### 7.2. Set Device Config:

```C
PTDEVICE_CONFIG_STRUCT device = malloc(sizeof(struct DEVICE_CONFIG_STRUCT));

device.Name = "YOUR_DEVICE_NAME";
device.Type = "YOUR_DEVICE_TYPE";
device.Description = "YOUR_DEVICE_DESCRIPTION";
config.DeviceNumber = device_num;
config.DeviceList = device;
```

##### 7.3.1 Set Analog Tag Config:

```C
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

##### 7.3.2 Set Discrete Tag Config:

```C
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

##### 7.3.3 Set Text Tag Config:

```C
PTTEXT_TAG_CONFIG textTag = malloc(sizeof(struct TEXT_TAG_CONFIG));

textTag.Name = "TestName";
textTag.Description = "description";
textTag.ReadOnly = false;
textTag.ArraySize = 0;
```

### 8. SendData\( TEDGE_DATA_STRUCT data \)

Send tag value to cloud.
Attributes：According to Tag type, properties can be used as follows:

| EdgeData Structure | Tag Type Structure |Property | Data Type | Description |
| :--- | :--- | :--- | :--- | :--- |
| TEDGE_DATA_STRUCT | AnalogTagList | Value | double | analog tag value |
| TEDGE_DATA_STRUCT | DiscreteTagList | Value | unsigned integer | discrete tag value |
| TEDGE_DATA_STRUCT | TextTagList | Value | pointer to character | text tag value |

Use TEDGE_DATA_STRUCT structure to conscruct your data object
```C
TEDGE_DATA_STRUCT data;

PTEDGE_DEVICE_STRUCT data_device = malloc(device_num * sizeof(struct EDGE_DEVICE_STRUCT));

int analog_tag_num = 1;
int discrete_tag_num = 1;
int text_tag_num = 1;

PTEDGE_ANALOG_TAG_STRUCT analog_data_tag = malloc(analog_tag_num * sizeof(struct EDGE_ANALOG_TAG_STRUCT));
PTEDGE_DISCRETE_TAG_STRUCT discrete_data_tag = malloc(discrete_tag_num * sizeof(struct EDGE_DISCRETE_TAG_STRUCT));
PTEDGE_TEXT_TAG_STRUCT text_data_tag = malloc(text_tag_num * sizeof(struct EDGE_TEXT_TAG_STRUCT));

analog_data_tag.Name = "AnalogTagName";
analog_data_tag.Value = YOUR_TAG_VALUE; // type: double

discrete_data_tag.Name = "DiscreteTagName";
discrete_data_tag.Value = YOUR_TAG_VALUE; // type: unsigned integer

text_data_tag.Name = "TextTagName";
text_data_tag.Value = YOUR_TAG_VALUE; // type: pointer to character

data_device.AnalogTagNumber = analog_tag_num;
data_device.AnalogTagList = analog_data_tag;

data_device.DiscreteTagNumber = discrete_tag_num;
data_device.DiscreteTagList = discrete_data_tag;

data_device.TextTagNumber = text_tag_num;
data_device.TextTagList = text_data_tag;

data_device.Id = "DeviceID";

data.DeviceNumber = 1;
data.DeviceList = data_device;

bool result = SendData(data);
```
Similar to allocate tag value in normal tags, an array tag value have to create a new array data instance (PTEDGE_ARRAY_TAG_STRUCT) and reference it in another tag instance (PTEDGE_TAG_STRUCT)

```c

/* analog array sample */
PTEDGE_ANALOG_TAG_STRUCT analog_data_tag = malloc(analog_tag_num * sizeof(struct EDGE_ANALOG_TAG_STRUCT));
PTEDG_ANALOG_ARRAY_TAG_STRUCT analog_data_array_tag = malloc(3 * sizeof(struct EDGE_ANALOG_ARRAY_TAG_STRUCT));

for ( int tag_num = 0; tag_num < analog_tag_num; tag_num++ ){

	/* construct array tag data */
	for(int idx = 0; idx< array_size; idx++){
		analog_data_array_tag[idx].Index = idx;
		analog_data_array_tag[idx].Value = value;
	}

	analog_data_tag[tag_num].ArraySize = array_size;
	analog_data_tag[tag_num].ArrayList = analog_data_array_tag;

}

data_device[YOUR_DEVICE_IDX].AnalogTagNumber = analog_tag_num;
data_device[YOUR_DEVICE_IDX].AnalogTagList = analog_data_tag;
data_device.Id = "DeviceID";

data.DeviceNumber = 1;
data.DeviceList = data_device;

bool result = SendData(data); 
```

### 9. SendDeviceStatus\( TEDGE\_DEVICE\_STATUS\_STRUCT deviceStatus \)

Send Device status to cloud when status changed.

```c
TEDGE_DEVICE_STATUS_STRUCT status;
status.DeviceNumber = device_num;

PTDEVICE_LIST_STRUCT dev_list = malloc(sizeof(struct DEVICE_LIST_STRUCT));
asprintf(&simDevId, "%s", "DeviceID");
dev_list.Id = simDevId;
dev_list.Status = 1;

status.DeviceList = dev_list;

bool result = SendDeviceStatus(status);
```



