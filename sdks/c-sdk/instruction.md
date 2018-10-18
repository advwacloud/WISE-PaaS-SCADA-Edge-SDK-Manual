#  {#development-environement}

# 使用說明 {#development-environement}

---

## EdgeAgent

### 1. Constructor\(TOPTION_STRUCT option\)

引用 EDGE_AGENT_OPTION.h 內所定義的結構來初始化 EdgeAgent，根據傳入參數 option 建立 MQTT 連線客戶端以及 SCADA 相關設定。

```
TOPTION_STRUCT options;
options.AutoReconnect = true;
options.ReconnectInterval = 1000;
options.ScadaId = "c9851920-ca7f-4cfd-964a-1969aef958f6";
options.Heartbeat = 60;
options.DataRecover = true;
options.ConnectType = DCCS; 	// Connection Type (DCCS, MQTT), Default is DCCS
options.Type = Gatway;
options.UseSecure = false;

TMQTT_OPTION_STRUCT mqtt;

switch (options.ConnectType)
{
	case 1: // If ConnectType is DCCS, must fill this options
		options.DCCS.CredentialKey = "9b03e5524c70b6e4c503173c6553abrh"; // Credential Key
		options.DCCS.APIUrl = "https://api-dccs.wise-paas.com/";		 // DCCS API Url
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

### 3. Connect\(\)

與 MQTT Broker 連線，連線資訊可經由定義 TOPTION_STRUCT 結構後的 options 取得，連線成功後會觸發 Connected 事件。

```
Connect();
```

### 4. Disconnect\(\)

與 MQTT Broker 連線，離線資訊可經由定義 TOPTION_STRUCT 結構後的 options 取得，離線成功後會觸發 Disconnected 事件。

```
Disconnect();
```

### 5. UploadConfig\( ActionType action, TSCADA_CONFIG_STRUCT edgeConfig \)

上傳SCADA/Device/Tag Config，並根據ActionType決定是Create/Update/Delete。

```
TSCADA_CONFIG_STRUCT config;
ActionType action = Create; // Create, Update od Delete
// set scada condig
// set device config
// set tag config
bool result = Constructor(options);;
```

SCADA Config設定

```
config.Id = Scada ID"; 
config.Description = "description";
config.Name = "For Test";
config.PrimaryPort = 1883;
config.BackupPort = 1883;
config.Type = 1;
```

Device Config設定

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

Analog Tag Config設定

```
PTANALOG_TAG_CONFIG analogTag = malloc(sizeof(struct ANALOG_TAG_CONFIG));

analogTag.Name = "TestName";    
analogTag.Description = "description";          
analogTag.ReadOnly = false;
analogTag.ArraySize = 0;
analogTag.AlarmStatus = false;
analogTag.SpanHigh = 1000;
analogTag.SpanLow = 0;
analogTag.EngineerUnit = "enuit";
analogTag.IntegerDisplayFormat = 4;
analogTag.FractionDisplayFormat = 2;
analogTag.HHPriority = 1;
analogTag.HHAlarmLimit = 1;
analogTag.HighPriority = 1;
analogTag.HighAlarmLimit = 1;
analogTag.LowPriority = 1;
analogTag.LowAlarmLimit = 1;
analogTag.LLPriority = 1;
analogTag.LLAlarmLimit = 1;
analogTag.NeedLog = true;
```

Discrete Tag Config設定

```
PTDISCRETE_TAG_CONFIG discreteTag = malloc(sizeof(struct DISCRETE_TAG_CONFIG));

discreteTag.NAme = "TestName"
discreteTag.Description = "description";
discreteTag.ReadOnly = false;
discreteTag.ArraySize = 0;
discreteTag.AlarmStatus = false;
discreteTag.State0 = "0";
discreteTag.State1 = "1";
discreteTag.State2 = "";
discreteTag.State3 = "";
discreteTag.State4 = "";
discreteTag.State5 = "";
discreteTag.State6 = "";
discreteTag.State7 = "";
discreteTag.State0AlarmPriority = 0;
discreteTag.State1AlarmPriority = 0;
discreteTag.State2AlarmPriority = 0;
discreteTag.State3AlarmPriority = 0;
discreteTag.State4AlarmPriority = 0;
discreteTag.State5AlarmPriority = 0;
discreteTag.State6AlarmPriority = 0;
discreteTag.State7AlarmPriority = 0;
```

Text Tag Config設定

```
PTTEXT_TAG_CONFIG textTag = malloc(sizeof(struct TEXT_TAG_CONFIG));

textTag.Name = "TestName";
textTag.Description = "description";
textTag.ReadOnly = false;
textTag.ArraySize = 0;
textTag.AlarmStatus = false;
```

### 6. SendData\( EdgeData data \)

上傳設備的Tag Value。

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

### 7. SendDeviceStatus\( TEDGE_DEVICE_STATUS_STRUCT deviceStatus \)

上傳Device Status \(狀態有改變再送即可\)。

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

### 8. 屬性

| Property Name | Data Type | Description |
| :--- | :--- | :--- |
| IsConnected | boolean | 判斷連線狀態 \(read only\) |



