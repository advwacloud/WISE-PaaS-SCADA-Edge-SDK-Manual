# Sample Code {#development-environement}

---

* GitHub

  * [WISEPaaS.SCADA.Python.SDK.Sample](https://github.com/advwacloud/WISEPaaS.SCADA.Golang.SDK.Sample)

* 使用方式:
  1. 進入各功能資料夾(例如: connect 功能)
  ``` go
  $ cd connect
  ```
  2. 修改 mqtt connection options
  ``` go
  options := SDK.NewEdgeAgentOptions()
	options.MQTT = &SDK.MQTTOptions{
		HostName: "127.0.0.1",
    Port: 1883,
    UserName: "admin",
    Password: "admin",
    ProtocalType: SDK.Protocol["TCP"],
	}
	options.ConnectType = SDK.ConnectType["MQTT"]
	options.ScadaID = "aca6be88-921c-4cfa-a0a7-e2e59e83e424"
  ```
  3. 執行 `main.go`
  ``` go
  go run main.go
  ```

