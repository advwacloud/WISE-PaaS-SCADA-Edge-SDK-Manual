# Release Note {#development-environement}

---
* \[1.0.2\] - 2020-03-25
 * Add
    * 實作config cache機制
    * 利用config cache實現FractionDisplayFormat預處理機制
    * 紀錄上次的config, 如果config相同就不再傳送
  * Fix
    - 修正斷線後data recover每次只會緩存一個封包的問題
  * Update
    - 將nscada改名成node (or datahub), topic name除外 
    - remove deprecation properties: node's ID, name, and description
    - support "Delsert" action of UploadConfig
    - remove deprecation properties about alarm
    - add RetentionPolicyName property of DeviceConfig
    - dataRecoverTimer改成只在connected才啟動, disconnected時停止
* \[1.0.1\] - 2019-10-17
  * Update
    * Downgrade to Java7, rewrite the syntax and usage of Java8, the purpose is to support down to Android sdk 22
* \[1.0.0\] - 2019-10-07
  * Added
    * Complete the data recover and mqtt secure connection

* \[1.0.0-beta\] - 2019-09-27
  * Added
    * The main functions are completed, the data recover and mqtt secure connection are not supported yet.



