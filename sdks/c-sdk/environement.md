# Environement {#development-environement}

---

* C version
  * Standard C99
* GNU C Compiler

  * GNU/Linux, MinGW\(Windows\), Unix
  * calling /usr/bin/gcc or /usr/bin/c99

* Reference Package

  * libmosquitto-dev
  * [https://packages.debian.org/sid/libmosquitto-dev](https://packages.debian.org/sid/libmosquitto-dev)

#### Build dynamic libraries \(WISEPaaS.so files\).

* via command line
  ```
  make build
  ```

  This points to a script which invokes gcc after having added the -std=c99 flag

build: WISEPaaS.so.1.0.0  
    gcc test.c -ldl -g -o sample -std=c99 \(or -std=iso9899:1999\)

