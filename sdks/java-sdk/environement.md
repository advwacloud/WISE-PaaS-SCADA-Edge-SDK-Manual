# Environement {#development-environement}

---

* IDE
  * VS Code 1.37.1
  * Android Studio 3.5 
* Runtime
  * Java SE 8
  * Android SDK 26 (API level 26) 以上
* Maven Repo
  * 目前還沒將SDK上傳至遠端Repo, 可以先在本地端使用mavenLocal()來測試
    * 將SDK專案maven install至local maven repo, 接著在android
    * 接著在android build.gradle裡相依套件加入sdk
    ```
    dependencies {
        implementation 'wisepaas.scada:sdk:1.0.0'
    }
    ```

