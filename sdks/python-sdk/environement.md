# Environement {#development-environement}

---

* Runtime
  * Python: 3.6
  * Pip: 1.8
* Python Package:
  * WISE\_PaaS\_SCADA\_Python\_SDK

#### Installation

* via command line

  ```
  pip install WISE_PaaS_SCADA_Python_SDK
  ```

* upgrade package version

  ```
  pip install WISE_PaaS_SCADA_Python_SDK --upgrade
  ```

* check installed version

```
pip show WISE_PaaS_SCADA_Python_SDK

/*
output>>
Name: WISE-PaaS-SCADA-Python-SDK
Version: 1.0.10
Summary: The WISEPaaS_SCADA_Python_SDK package allows developers to write Python applications which access 
the WISE-PaaS Platform via MQTT or MQTT over the Secure WebSocket Protocol.
*/
```

#### FAQ

1. If you get some error messages about SSL certificate as follows:

```
[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:720
```

* Windows
  * Install [Win32/Win64 OpenSSL](https://slproweb.com/products/Win32OpenSSL.html).
* MacOS
  * You have two options:

  Run an install command shipped with Python 3.6
  ```
  cd /Applications/Python\ 3.6/
  ./Install\ Certificates.command
  ```

  Or install the [certifi package](https://pypi.org/project/certifi/) with

  ```
  pip install certifi
  ```



