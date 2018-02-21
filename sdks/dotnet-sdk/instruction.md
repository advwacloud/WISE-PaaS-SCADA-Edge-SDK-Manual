# 使用說明 {#development-environement}

---

## EdgeAgent

### 1. 建構子

##### 初始化 EdgeAgent 實例，並根據傳入參數 EdgeAgentOptions 建立 MQTT 連線客戶端以及 SCADA 相關設定。

```
EdgeAgentOptions options = new EdgeAgentOptions()
{
    HostName = "127.0.0.1",
    Port = 1883,
    Username = "admin",
    Password = "admin",
    ProtocolType = Protocol.TCP,
    UseSecure = false,
    AutoReconnect = true,
    ReconnectInterval = 1000,
    ScadaId = "5095cf13-f005-4c81-b6c9-68cf038e2b87",    // getting from SCADA portal
    Heartbeat = 60000   // default is 60 seconds
};
EdgeAgent edgeAgent = new EdgeAgent( options );
```

### 2. 事件

##### EdgeAgent 有三種事件供訂閱, 分別如下:

* Connected: 當 EdgeAgent 成功連上 Broker 後觸發
* Disconnected: 當 EdgeAgent 連線中斷後觸發
* MessageReceived: 當 EdgeAgent 接收到 MQTT 訊息後觸發，根據 MessageReceivedEventArgs.Type 可以分成以下訊息類型:

  * DataOn
    : 資料開始上傳
  * DataOff: 資料停止上傳
  * WriteValue: Cloud 端改變 Tag 值同步到 Edge 端
  * WriteConfig  
    : Cloud 端改變 Config 同步到 Edge 端

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
            WriteValueCommandMessage wvCmdMsg = (WriteValueCommandMessage)e.Message;
            foreach ( var item in wvCmdMsg.D.Val )
            {
                Console.Write( "Tag: {0}, ", item.Key );
                Console.WriteLine( "Value: {0}", item.Value );
            }
            break;
        case MessageType.WriteConfig:
            break;
        case MessageType.DataOn:
            DataOnCommandMessage dataOnMsg = (DataOnCommandMessage)e.Message;
            // ready to send tag value
            EdgeData data = new EdgeData();
            bool result = edgeAgent.SendData( data ).Result;
            break;
        case MessageType.DataOff:
            DataOffCommandMessage dataOffMsg = (DataOffCommandMessage)e.Message;
            timer1.Enabled = false;
            break;
        case MessageType.ConfigAck:
            ConfigAckMessage cfgAckMsg = (ConfigAckMessage)e.Message;
            Console.WriteLine( "Upload Config Result: {0}", cfgAckMsg.D.Cfg.ToString() );
            break;
    }
}
```



