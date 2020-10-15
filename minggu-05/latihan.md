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
2020-10-15 19:57:55,364 |main                 |EntityDataLoadContainer       |I| =-=-=-=-=-=-= Finished the data load with 17175 rows changed.
2020-10-15 19:57:55,364 |main                 |ContainerLoader               |I| Started container dataload-container
2020-10-15 19:57:55,366 |main                 |ContainerLoader               |I| Shutting down containers
2020-10-15 19:57:55,541 |main                 |ContainerLoader               |I| Stopping container dataload-container
2020-10-15 19:57:55,541 |main                 |ContainerLoader               |I| Stopped container dataload-container
2020-10-15 19:57:55,542 |main                 |ContainerLoader               |I| Stopping container service-container
2020-10-15 19:57:55,628 |main                 |JobPoller                     |I| Shutting down JobPoller.
2020-10-15 19:57:55,629 |OFBiz-JobPoller      |JobPoller                     |I| JobPoller thread started.
2020-10-15 19:57:55,629 |main                 |JobPoller                     |I| JobPoller shutdown completed.
2020-10-15 19:57:55,630 |OFBiz-JobPoller      |JobPoller                     |I| JobPoller thread stopped.
2020-10-15 19:57:55,631 |main                 |ServiceContainer              |I| Removing from cache dispatcher: entity-default
2020-10-15 19:57:55,632 |main                 |ServiceDispatcher             |I| De-Registering dispatcher: entity-default
2020-10-15 19:57:55,632 |main                 |ServiceDispatcher             |I| Shutting down the service engine...
2020-10-15 19:57:55,632 |main                 |ContainerLoader               |I| Stopped container service-container
2020-10-15 19:57:55,632 |main                 |ContainerLoader               |I| Stopping container component-container
2020-10-15 19:57:55,632 |main                 |ContainerLoader               |I| Stopped container component-container
:loadDefault

BUILD SUCCESSFUL

Total time: 46 mins 27.991 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.13/userguide/gradle_daemon.html
```

Setelah installasi OFBiz selesai, jalankan service OFBiz.
```bash
farhan@Arctic:~/apache-ofbiz-16.11.05$ sudo ./gradlew "ofbiz --load-data readers=seed"
farhan@Arctic:~/apache-ofbiz-16.11.05$ sudo ./gradlew "ofbiz --load-data readers=seed,seed-initial,ext"
farhan@Arctic:~/apache-ofbiz-16.11.05$ sudo ./gradlew ofbiz
```

Membuka OFBiz di Browser. Untuk user/password : admin/ofbiz
(gambar/LoginOfbiz.png)
(gambar/dashboard.png)

Default dashboard: https://SERVER_IP:8443/ordermgr/control/main. 
e-Commerce: https://SERVER_IP:8443/ecommerce
WebTools: https://SERVER_IP:8443/webtools
Catalog Manager: https://SERVER_IP:8443/catalog





