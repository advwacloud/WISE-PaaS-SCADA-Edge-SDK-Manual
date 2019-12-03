# Environement {#development-environement}

---

* C version
  * Standard C99
* GNU C Compiler

  * GNU/Linux, MinGW\(Windows\), Unix
  * calling /usr/bin/gcc or /usr/bin/c99

* Required Package
  * libmosquitto-dev >= 1.5.7  [https://packages.debian.org/sid/libmosquitto-dev](https://packages.debian.org/sid/libmosquitto-dev)

* Referance Package
  * libcurl >= 7.55.0 [https://curl.haxx.se/docs/releases.html](https://curl.haxx.se/docs/releases.html)
  * libsqlite3 >= 3.18 [https://cppget.org/libsqlite3](https://cppget.org/libsqlite3)
  * cJSON > 1.7.5 [https://github.com/DaveGamble/cJSON](https://github.com/DaveGamble/cJSON)

#### Build dynamic libraries \(WISEPaaS.so files\).

* Open a terminal/console/command prompt, change to the directory where you cloned Processing, and type:
  ```
  make build
  ```

  This points to a script which invokes gcc after having added the -std=c99 flag

* build sample
  ```
  gcc test.c -ldl -g -o sample -std=c99 \(or -std=iso9899:1999\)
  ```

