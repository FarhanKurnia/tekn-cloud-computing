# Installasi Apache OFBiz

### Install Packet yang dibutuhkan
Install paket java
```bash
farhan@Arctic:~$ sudo apt-get install apt-transport-https ca-certificates wget dirmngr gnupg software-properties-common unzip -y
farhan@Arctic:~$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8AC3B29174885C03
farhan@Arctic:~$ sudo add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
farhan@Arctic:~$ sudo apt-get install adoptopenjdk-8-hotspot -y
```

Setelah itu cek apakah sudah terinstall dengan baik.
```bash
farhan@Arctic:~$ java -version
openjdk version "1.8.0_265"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_265-b01)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.265-b01, mixed mode)
```

### Installasi OFBiz
Download OFBiz, kemudian extract file yang telah didownload.
```bash
farhan@Arctic:~$ wget https://archive.apache.org/dist/ofbiz/apache-ofbiz-16.11.05.zip
farhan@Arctic:~$ cd apache-ofbiz-16.11.05
farhan@Arctic:~/apache-ofbiz-16.11.05$ sudo ./gradlew cleanAll loadDefault
```

Setelah instalasi selesai:
```bash

```



