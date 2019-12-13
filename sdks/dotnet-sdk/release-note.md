# Release Note {#development-environement}

---

* \[1.0.8\] - 2019-12-xx

  * Added

    * UploadConfig增加"Delsert" action, 作用是讓雲端強制直接套用本次上傳的Config

    * DeviceConfig增加RetentionPolicyName欄位, 透過該欄位可自動與雲端設定的Retention Policy綁定

  * Updated

    * 移除EdgeConfig.ScadaConfig中的ID, Name和Description屬性

* \[1.0.7\] - 2018-10-02

  * Updated

    * EdgeData Property
    * 每個Data Message若包含超過100 Tag, 則會被切割成多個Message傳送
    * 重新定義Message Received中各種Message類型物件, 讓使用上更直覺

* \[1.0.6\] - 2018-08-01

  * Added

    * 增加IsConnected屬性, 可以主動判斷現在連線狀態

* \[1.0.5\] - 2018-08-01

  * Added

    * 斷線後自動重新獲取新的credential重連機制

  * Fixed

    * 調整Connect\(\)和Disconnect\(\)為非同步

* \[1.0.4\] - 2018-05-15

  * Fixed

    * 解決資料封包無法自訂時間問題

* \[1.0.3\] - 2018-05-14

  * Added

    * 支援DCCS

* \[1.0.2\] - 2018-03-22

  * Added

    * 支援斷點續傳

* \[1.0.1\] - 2018-02-22

* \[1.0.0\] - 2017-02-20



