# Environement {#development-environement}

---

* **IDE**
  * VS Code 1.37.1
  * Android Studio 3.5 
* **Runtime**
  * **Java 7** or higher \(Java 8 is recommended\)
  * **Android SDK 22** or higher \(SDK 26 or higher is recommended\)
* **Installation**

  * Repo: JCenter \([https://bintray.com/advwacloud/scada-java-sdk/scada-sdk](https://bintray.com/advwacloud/scada-java-sdk/scada-sdk)\)
  * download by Maven
    ```
    #### Before version 1.0.2 ######
    <dependency>
        <groupId>wisepaas.scada</groupId>
        <artifactId>sdk</artifactId>
        <version>1.0.1</version>
        <type>jar</type>
    </dependency>
    
    #### Start from version 1.0.2 ######
    
    <dependency>
  	  <groupId>wisepaas.datahub</groupId>
  	  <artifactId>edge-sdk</artifactId>
  	  <version>1.0.2</version>
  	  <type>pom</type>
    </dependency>

    ```
  * download by gradle

    ```
    #### Before version 1.0.2 ######
    dependencies {
        implementation 'wisepaas.scada:sdk:1.0.1'
    }
    
    #### Start from version 1.0.2 ######
    dependencies {
        implementation 'wisepaas.datahub:edge-sdk:1.0.2'
    }
    ```

  * Older Android Version \(lower than SDK26\)

    * Need to specify the compile source and target to 1.8
      ```
      compileOptions {
        sourceCompatibility = 1.8
        targetCompatibility = 1.8
      }
      ```



