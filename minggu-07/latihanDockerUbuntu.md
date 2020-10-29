# Getting Started With Docker on Ubuntu

## Install Docker di Ubuntu

1. Pertama, instal beberapa paket prasyarat yang memungkinkan apt menggunakan paket lewat HTTPS:
```bash
farhan@Arctic:~$ sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```

2. Lalu tambahkan kunci GPG untuk repositori Docker resmi ke sistem Anda:
```bash
farhan@Arctic:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
```

3. Tambahkan repositori Docker ke sumber APT:
```bash
farhan@Arctic:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

4. Selanjutnya, perbarui basis data paket dengan paket Docker dari repo yang baru ditambahkan:
```bash
farhan@Arctic:~$ sudo apt-get update
```

5. Pastikan Anda akan menginstal dari repo Docker alih-alih repo Ubuntu asali, Anda akan melihat keluaran seperti ini, meskipun nomor versi untuk Docker mungkin berbeda:
```bash
farhan@Arctic:~$ apt-cache policy docker-ce
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.13~3-0~ubuntu-focal
  Version table:
     5:19.03.13~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.12~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.11~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.10~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```
Perhatikan bahwa docker-ce belum terinstal, tetapi kandidat untuk instalasi adalah dari repositori Docker untuk Ubuntu 20.04 (focal).

6. Akhirnya, instal Docker:
```bash
farhan@Arctic:~$ sudo apt install docker-ce
```

Docker kini seharusnya sudah terinstal, daemon dimulai, dan prosesnya kini dapat berjalan ketika memulai saat boot. Periksa bahwa ini berjalan:
```bash
farhan@Arctic:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2020-10-30 00:19:06 WIB; 6min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 55344 (dockerd)
      Tasks: 10
     Memory: 37.3M
     CGroup: /system.slice/docker.service
             └─55344 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Okt 30 00:18:57 Arctic dockerd[55344]: time="2020-10-30T00:18:57.598271596+07:00" level=warni>
Okt 30 00:18:57 Arctic dockerd[55344]: time="2020-10-30T00:18:57.598307399+07:00" level=warni>
Okt 30 00:18:57 Arctic dockerd[55344]: time="2020-10-30T00:18:57.598343598+07:00" level=warni>
Okt 30 00:18:57 Arctic dockerd[55344]: time="2020-10-30T00:18:57.599205610+07:00" level=info >
Okt 30 00:18:58 Arctic dockerd[55344]: time="2020-10-30T00:18:58.585039617+07:00" level=info >
Okt 30 00:18:59 Arctic dockerd[55344]: time="2020-10-30T00:18:59.625666570+07:00" level=info >
Okt 30 00:19:06 Arctic dockerd[55344]: time="2020-10-30T00:19:05.706626499+07:00" level=info >
Okt 30 00:19:06 Arctic dockerd[55344]: time="2020-10-30T00:19:05.707313639+07:00" level=info >
Okt 30 00:19:06 Arctic dockerd[55344]: time="2020-10-30T00:19:06.234166717+07:00" level=info >
Okt 30 00:19:06 Arctic systemd[1]: Started Docker Application Container Engine.
lines 1-21/21 (END)
```

 7. untuk menjalankan docker bisa dicoba sebagai berikut.
 ```bash
 farhan@Arctic:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```