# Docker Networking Hands-on Lab

## Section #1 - Networking Basics
### Step 1: The Docker Network Command
Perintah `docker network` adalah perintah utama untuk melakukan konfigurasi dan memanajemen container pada network.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

Pada perintah utama tersebut dapat digunakan sub perintah seperti yang dimunculkan pada output. Seperti yang kita lihat, perintah `docker network` mengizinkan kita untuk membuat network baru, daftar network yang tersedia, menampilkan detail informasi satau atau lebih network dan menghapus network. Perintah tersebut mengizinkan kita untuk connect atau disconnect container dari network.

### Step 2: List networks
Jalankan `docker netwokr ls` untuk melihat container yang tersedia dan host Docker baru-baru ini.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
564ba28a318c   bridge    bridge    local
f3480b56cfeb   host      host      local
f7a088b26b16   none      null      local
```

### Step 3: Inspect a network
Perintah `docker network inspect` digunakan untuk melihat konfigurasi detail dari network yang berisi: nama, ID, Driver, IPAM driver, subnet info, container yang terhubung dan lain-lain.
GUnakan `docker network inspect <network>` untuk melihat detail container dari network yang terpilih, <network> dapat diisi nama network atau ID network yang dipilih.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network inspect bridge
```
```json
[
    {
        "Name": "bridge",
        "Id": "564ba28a318c9661bdf845395407108d325be02c8260ae81436e5d65f7ae19cb",
        "Created": "2020-12-09T11:59:20.956356944Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

### Step 4: List network driver plugins
Perintah `docker info` akan menampilkan banyak informasi tentang instalasi docker.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.0-rc2
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 ......
```
Pada output diatas kita dapat melihat driver network yang dimiliki adalah bridge host ipvlan macvlan null overlay.


## Section #2 - Bridge Networking
### Step 1: The Basics
Setiap instalasi docker akan dibua network dengan nama bridge.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
564ba28a318c   bridge    bridge    local
f3480b56cfeb   host      host      local
f7a088b26b16   none      null      local
```
Output diatas menampilkan bridge network dengan cakupan lokal. Berarti network tersebut hanya tersedia di docker host. Brodge driver menyediakan single-host network. 
Semua network dibuat dengan bridge driver berdasarkan linux bridge (Virtual Switch).
Install perintah `brtcl` dan gunakan untuk melihat daftar bridge linux pada docker host kita. Jalankan `sudo apt-get install bridge-utils`.
```bash
f7a088b26b16   none      null      local
[node1] (local) root@192.168.0.28 ~
$ apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
v3.12.1-79-g28248c561e [http://dl-cdn.alpinelinux.org/alpine/v3.12/main]
v3.12.1-78-g8ca0a357ed [http://dl-cdn.alpinelinux.org/alpine/v3.12/community]
OK: 12745 distinct packages available
[node1] (local) root@192.168.0.28 ~
$ apk add bridge
(1/1) Installing bridge (1.5-r4)
OK: 404 MiB in 151 packages
```

Lalu jalankan `brctl` show untuk melihat daftar bridge pada docker list.
```bash
[node1] (local) root@192.168.0.28 ~
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.024235989020       no
```

pada output tersebut menampilkan single linux bridge dengan nama docker0. Bridge ini otomatis dibuat untuk bridge network. Lalu dapat kita lihat tidak ada interfaces yang connect di dalamnya.
Gunakan perintah `ip a` untuk melihat detail dari bridge docker0.
```bash
[node1] (local) root@192.168.0.28 ~`
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueuestate DOWN
    link/ether 02:42:35:98:90:20 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
7929: eth0@if7930: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 52:3d:e4:72:0f:b8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.28/23 scope global eth0
       valid_lft forever preferred_lft forever
7935: eth1@if7936: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:12:00:3b brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.59/16 scope global eth1
       valid_lft forever preferred_lft forever
```

### Step 2: Connect a container
Network bridge adalah default network untuk containr baru. Kecuali dispesifikasikan network yang berbeda, semua container baru akan terhubung dengan network bridge.
Buatlah container baru dengan perintah `run -dt ubuntu sleep infinity`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker run -dt ubuntu sleep infinity
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
da7391352a9b: Pull complete
14428a6d4bcd: Pull complete
2c2d948710f2: Pull complete
Digest: sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c
Status: Downloaded newer image for ubuntu:latest
eae5ff5af07b713bca6e6d831ca2aeb8a54ba74cb9ac4ecef6c0fcf9122e411f
```

Perintah ini akan membuat container baru berdasarkan ubuntu:latest image dan akan menjalankan perintah sleep untuk tetap menjalankan container di background. 
Gunakan untuk perintah `docker ps` untuk melihat container yang berjalan.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker ps
CONTAINER ID   IMAGE     COMMAND            CREATED         STATUS    PORTS     NAMES
eae5ff5af07b   ubuntu    "sleep infinity"   3 minutes ago   Up 3 minutes             recursing_bardeen
```

Kita tidak memberikan perintah network pada saat membuat container tersebut, maka otomatiss akan ditambahkan pada network bridge.
Jalankan perintah `brctl show` untuk melihat.
```bash
[node1] (local) root@192.168.0.28 ~
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.024235989020       no              veth0e98228
```

Perhatikan bagaimana docker0 brdge sekarang, saat ini di bagan interface sudah connect.
Coba kita inspect lagi dengan perintah `docker network inspect bridge`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network inspect bridge
```
```json
[
    {
        "Containers": {
            "eae5ff5af07b713bca6e6d831ca2aeb8a54ba74cb9ac4ecef6c0fcf9122e411f": {
                "Name": "recursing_bardeen",
                "EndpointID": "f66d3016809a9048ef0acb5187ce6d127e4a4d3b143e523f4a83564adb0a7d4f",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        }
    }
]
```

### Step 3: Test network connectivity
Pada output sebelumnya terlihat IP Address dari container tersebut. Ping IP address dari container dengan perintah `ping -c5 <IPv4 Address>`.
```bash
[node1] (local) root@192.168.0.28 ~
$ ping -c5 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.275 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.082 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.130 ms
64 bytes from 172.17.0.2: seq=3 ttl=64 time=0.127 ms
64 bytes from 172.17.0.2: seq=4 ttl=64 time=0.082 ms

--- 172.17.0.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.082/0.139/0.275 ms
```

Terlihat balasan diatas menampilkan bahwa docker host dapat ping ke container dengaan bridge network.
Tapi kita juga bisa membuat container daat terhubung dengan dunia luar juga, install ping dan ping www.github.com.
Pertama, kita perlu mendapatkan ID container di langkah sebelumnya. Jalankan docker PS seperti seberlumnya.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker ps
CONTAINER ID   IMAGE     COMMAND            CREATED         STATUS   PORTS     NAMES
16afd35c30d7   ubuntu    "sleep infinity"   3 seconds ago   Up 1 second             busy_gates
```

Selanjutnya, jalankan shell di dalam container ubuntu dengan perintah `docker exec -it <CONTAINER ID> /bin/bash`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker exec -it 16afd35c30d7 /bin/bash
root@16afd35c30d7:/#
```

Selanjutnya install ping dengan menjalankan perintah `apt-get update && apt-get install -y iputils-ping`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker exec -it 16afd35c30d7 /bin/bash
root@16afd35c30d7:/# apt-get update && apt-get install -y iputils-ping
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [103 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [1167 B]
Get:5 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [643 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [464 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:10 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [855 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [875 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [136 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [30.4 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [4248 B]
Fetched 16.5 MB in 2s (7095 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libcap2 libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2 libcap2-bin libpam-cap
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 90.5 kB of archives.
After this operation, 333 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2 amd64 1:2.32-1 [15.9 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2-bin amd64 1:2.32-1 [26.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 iputils-ping amd64 3:20190709-3 [40.1 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/main amd64 libpam-cap amd64 1:2.32-1 [8352 B]
Fetched 90.5 kB in 0s (207 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libcap2:amd64.
(Reading database ... 4121 files and directories currently installed.)
Preparing to unpack .../libcap2_1%3a2.32-1_amd64.deb ...
Unpacking libcap2:amd64 (1:2.32-1) ...
Selecting previously unselected package libcap2-bin.
Preparing to unpack .../libcap2-bin_1%3a2.32-1_amd64.deb ...
Unpacking libcap2-bin (1:2.32-1) ...
Selecting previously unselected package iputils-ping.
Preparing to unpack .../iputils-ping_3%3a20190709-3_amd64.deb ...
Unpacking iputils-ping (3:20190709-3) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../libpam-cap_1%3a2.32-1_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.32-1) ...
Setting up libcap2:amd64 (1:2.32-1) ...
Setting up libcap2-bin (1:2.32-1) ...
Setting up libpam-cap:amd64 (1:2.32-1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Cannot locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up iputils-ping (3:20190709-3) ...
Processing triggers for libc-bin (2.31-0ubuntu9.1) ...
root@16afd35c30d7:/#
```

Lalu ping github dengan perintah `ping -c5 www.github.com`
```bash
root@16afd35c30d7:/#   ping -c5 www.github.com
PING github.com (140.82.113.4) 56(84) bytes of data.
64 bytes from lb-140-82-113-4-iad.github.com (140.82.113.4): icmp_seq=1 ttl=51 time=0.948 ms
64 bytes from lb-140-82-113-4-iad.github.com (140.82.113.4): icmp_seq=2 ttl=51 time=0.927 ms
64 bytes from lb-140-82-113-4-iad.github.com (140.82.113.4): icmp_seq=3 ttl=51 time=2.30 ms
64 bytes from lb-140-82-113-4-iad.github.com (140.82.113.4): icmp_seq=4 ttl=51 time=1.05 ms
64 bytes from lb-140-82-113-4-iad.github.com (140.82.113.4): icmp_seq=5 ttl=51 time=0.943 ms

--- github.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 0.927/1.233/2.301/0.535 ms
```

Untuk keluar dari container gunakan perintah `exit`.
```bash
root@16afd35c30d7:/# exit
exit
[node1] (local) root@192.168.0.28 ~
$
```

Setelah keluar kita perlu menghentikan container dengan perintah `stop <docker_id>`
```bash
[node1] (local) root@192.168.0.28 ~
$ docker stop 16afd35c30d7
16afd35c30d7
[node1] (local) root@192.168.0.28 ~
```

### Step 4: Configure NAT for external connectivity
Pada tahap ini kita akan memulai conatiner baru dengan NGINX dan map port 8080 ke port 80 di dalam container. Berarti traffik yang melakukan hit ke docker host dengan port 8080 akan dilewatkan ke port 80 di dalam container.
Jalankan container baru dari official NGINX image dengan perintah `docker run --name web1 -d -p 8080:80 nginx`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker run --name web1 -d -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
852e50cd189d: Pull complete
571d7e852307: Pull complete
addb10abd9cb: Pull complete
d20aa7ccdb77: Pull complete
8b03f1e11359: Pull complete
Digest: sha256:6b1daa9462046581ac15be20277a7c75476283f969cb3a61c8725ec38d3b01c3
Status: Downloaded newer image for nginx:latest
9a6a86b3a812276aeeec0d33995cdda8c989f08aac9e3b6f75cdf5ae1d1c4651
```

Cek pada docker bahwa container sudah ada dengan perintah `dockerps`
```bash
[node1] (local) root@192.168.0.28 ~
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                 NAMES
9a6a86b3a812   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   web1
```

Untuk melengkapi langkah sebelumnya kita membutuhkan IP Address dari docker host kita.
```bash
[node1] (local) root@192.168.0.28 ~
$ curl 127.0.0.1:8080
```
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Section #3 - Overlay Networking
### Step 1: The Basics
Pada tahap ini kita akan menginisialisasi swar baru, menggabungkan single worker node dan memastikan operasi berjalan.
Jalankan  `docker swarm init --advertise-addr $(hostname -i)`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker swarm init --advertise-addr $(hostname -i)
Swarm initialized: current node (2egs9qhdzcllq9hwek0slfrf9) is now a manager.
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-6atm0o174z9xcrjlbssn4qw07e95mjgbvq5se24op9citeioqk-bkitqp7cilu6ugkzjczckrcz6 192.168.0.28:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Lalu join dengan node diatas.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker swarm join \
> --token SWMTKN-1-6atm0o174z9xcrjlbssn4qw07e95mjgbvq5se24op9citeioqk-bkitqp7cilu6ugkzjczckrcz6 \
> 192.168.0.28:2377
Error response from daemon: This node is already part of a swarm. Use "docker swarm leave" to leave this swarm and join another one.
```

Lalu cek node dengan perintah `docker node ls`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker node lsID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
2egs9qhdzcllq9hwek0slfrf9 *   node1      Ready     Active         Leader           20.10.0-rc2
[node1] (local) root@192.168.0.28 ~
```

### Step 2: Create an overlay network
Sekarang waktunya untuk membuat overlay network.
Buat overlay baru dengan nama “overnet” dengan perintah docker network create -d overlay overnet.
```bash
$ docker network create -d overlay overnet
kiws0g4orgt52lvf5zw13d1uz
```

Gunakan perintah `docker network ls` untuk melihat apakah pembuatan network overlay kita sudah berhasil.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
01badbfdc47d   bridge            bridge    local
816a03be209a   docker_gwbridge   bridge    local
da79f3bb879e   host              host      local
03vyyjqmamjp   ingress           overlay   swarm
c63745369d39   none              null      local
kiws0g4orgt5   overnet           overlay   swarm
```

Jalankan perintah yang sama pada terminal kedua.
```bash
[node2] (local) root@192.168.0.27 ~
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a864793bf251   bridge    bridge    local
9c033cfada78   host      host      local
f6c82da29095   none      null      local
```

Perhatikan bahwa `overnet` network tidak muncul dalam list. Ini karena docker hanya meng-extend overlay network ke host ketika dibutuhkan saja.
Ini biasanya ketika host menjalankan task dari service yang telah dibuat oleh network.
Gunakan `docker inspect <network>` untuk melihat lebih detail tentang overnet network, jalankan pada terminal pertama.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network inspect overnet
```
```json
[
    {
        "Name": "overnet",
        "Id": "kiws0g4orgt52lvf5zw13d1uz",
        "Created": "2020-12-09T14:29:22.961131078Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.1.0/24",
                    "Gateway": "10.0.1.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": null
    }
]
```

### Step 3: Create a service
Sekarang kita perlu swarm yang telah diinisialisasi dan sebuah overlay network, sekarang saatnya membuat service yang digunakan network.
Eksekusi perintah dengan terminal pertama untuk membuat service baru dengan nama myservice pada overnet network degan 2 task/replika.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker service create --name myservice \
> --network overnet \
> --replicas 2 \
> ubuntu sleep infinity
d5uz736khvjmozoz06o35i2qu
overall progress: 2 out of 2 tasks
1/2: running
2/2: running
verify: Service converged
```

Untuk memastikan service yang dibuat dan kedua replika berjalan gunakan perintah `docker service ls`.
```bash
$ docker service ls
ID             NAME        MODE         REPLICAS   IMAGE           PORTS
d5uz736khvjm   myservice   replicated   2/2        ubuntu:latest
```
2/2 dalam kolom REPLICAS menunjukan kedua task dalam service up dan berjalan.


Pastikan singl task (replica) is running diantara 2 node dalam swarm deengan perintah `docker service ps myservice`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker service ps myservice
ID             NAME          IMAGE           NODE      DESIRED STATE   CURRENT STATE           ERROR     PORTS
renjyv0jiwp7   myservice.1   ubuntu:latest   node1     Running         Running 2 minutes ago
c2sl5f5xvuze   myservice.2   ubuntu:latest   node1     Running         Running 2 minutes ago
```
Pastikan task/replika berjalan di 2 nodes yang berbeda.

Sekarang kedua node berjalan di task overnet network dan bisa dilihat. Jalankan `docker network ls` dari terminal kedua untuk memastikan.
```bash
[node2] (local) root@192.168.0.27 ~
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a864793bf251   bridge    bridge    local
9c033cfada78   host      host      local
f6c82da29095   none      null      local
```

### Step 4: Test the network
Untuk melengkapi tahapan kita akan membutuhkan IP address dari service task yang berjalan di node2.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network inspect overnet
```
```json
[
    {
        "Name": "overnet",
        "Id": "kiws0g4orgt52lvf5zw13d1uz",
        "Created": "2020-12-09T14:48:24.5223318Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.1.0/24",
                    "Gateway": "10.0.1.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4e40550d0cc2d69334c8cd20383e4a37e3a57c1bdba926cb624622a770f1e810": {
                "Name": "myservice.1.renjyv0jiwp74vmgfi17c080d",
                "EndpointID": "273f2fb9ae3754de914ca418ce49466144fe6092204310b1c361994a2078ed34",
                "MacAddress": "02:42:0a:00:01:03",
                "IPv4Address": "10.0.1.3/24",
                "IPv6Address": ""
            },
            "8b046fcee206c596761a5dc7d0802070b568ee8b825599a913a7cef9a05fab3d": {
                "Name": "myservice.2.c2sl5f5xvuze409ofvy8sqo5e",
                "EndpointID": "3e73ec82c70996a95be73c5c3a2e42373f0992bc58d838777e3aaf045eee26d1",
                "MacAddress": "02:42:0a:00:01:04",
                "IPv4Address": "10.0.1.4/24",
                "IPv6Address": ""
            },
            "lb-overnet": {
                "Name": "overnet-endpoint",
                "EndpointID": "7327fc94ed2938898b462a114f94ac80ec37cb397835021ae3ccee44738bcc35",
                "MacAddress": "02:42:0a:00:01:05",
                "IPv4Address": "10.0.1.5/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "828716f7518d",
                "IP": "192.168.0.28"
            }
        ]
    }
]
```

Jalankan `docker ps` untuk melihat container ID pada service node2
```bash
[node1] (local) root@192.168.0.28 ~
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS               NAMES
8b046fcee206   ubuntu:latest   "sleep infinity"         18 minutes ago   Up 18 minutes               myservice.2.c2sl5f5xvuze409ofvy8sqo5e
4e40550d0cc2   ubuntu:latest   "sleep infinity"         18 minutes ago   Up 18 minutes               myservice.1.renjyv0jiwp74vmgfi17c080d
9a6a86b3a812   nginx           "/docker-entrypoint.…"   59 minutes ago   Up 59 minutes   0.0.0.0:8080->80/tcp   web1
```

Masuk ke dalam container dari service node2
```bash
[node1] (local) root@192.168.0.28 ~
$ docker exec -it 8b046fcee206 /bin/bash
```

Lalu install ping.
```bash
root@8b046fcee206:/# apt-get update && apt-get install -y iputils-ping
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [644 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:8 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [465 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [1167 B]
Get:10 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [103 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [876 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [856 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [136 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [30.4 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [4248 B]
Fetched 16.5 MB in 3s (6532 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libcap2 libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2 libcap2-bin libpam-cap
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 90.5 kB of archives.
After this operation, 333 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2 amd64 1:2.32-1 [15.9 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2-bin amd64 1:2.32-1 [26.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 iputils-ping amd64 3:20190709-3 [40.1 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/main amd64 libpam-cap amd64 1:2.32-1 [8352 B]
Fetched 90.5 kB in 0s (196 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libcap2:amd64.
(Reading database ... 4121 files and directories currently installed.)
Preparing to unpack .../libcap2_1%3a2.32-1_amd64.deb ...
Unpacking libcap2:amd64 (1:2.32-1) ...
Selecting previously unselected package libcap2-bin.
Preparing to unpack .../libcap2-bin_1%3a2.32-1_amd64.deb ...
Unpacking libcap2-bin (1:2.32-1) ...
Selecting previously unselected package iputils-ping.
Preparing to unpack .../iputils-ping_3%3a20190709-3_amd64.deb ...
Unpacking iputils-ping (3:20190709-3) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../libpam-cap_1%3a2.32-1_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.32-1) ...
Setting up libcap2:amd64 (1:2.32-1) ...
Setting up libcap2-bin (1:2.32-1) ...
Setting up libpam-cap:amd64 (1:2.32-1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Cannot locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up iputils-ping (3:20190709-3) ...
Processing triggers for libc-bin (2.31-0ubuntu9.1) ...
```

Lalu coba ping IP pada service node 1
```bash
root@8b046fcee206:/# ping -c5 10.0.1.3
PING 10.0.1.3 (10.0.1.3) 56(84) bytes of data.
64 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=0.194 ms
64 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=0.054 ms
64 bytes from 10.0.1.3: icmp_seq=3 ttl=64 time=0.051 ms
64 bytes from 10.0.1.3: icmp_seq=4 ttl=64 time=0.061 ms
64 bytes from 10.0.1.3: icmp_seq=5 ttl=64 time=0.063 ms

--- 10.0.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3997ms
rtt min/avg/max/mdev = 0.051/0.084/0.194/0.054 ms
```

Untuk cek IP node tersebut dapat berapa bisa menggunakan perintah `docker network inspect overnet`.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker network inspect overnet
```
```json
[
    {
        "Name": "overnet",
        "Id": "kiws0g4orgt52lvf5zw13d1uz",
        "Created": "2020-12-09T14:48:24.5223318Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.1.0/24",
                    "Gateway": "10.0.1.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4e40550d0cc2d69334c8cd20383e4a37e3a57c1bdba926cb624622a770f1e810": {
                "Name": "myservice.1.renjyv0jiwp74vmgfi17c080d",
                "EndpointID": "273f2fb9ae3754de914ca418ce49466144fe6092204310b1c361994a2078ed34",
                "MacAddress": "02:42:0a:00:01:03",
                "IPv4Address": "10.0.1.3/24",
                "IPv6Address": ""
            },
            "8b046fcee206c596761a5dc7d0802070b568ee8b825599a913a7cef9a05fab3d": {
                "Name": "myservice.2.c2sl5f5xvuze409ofvy8sqo5e",
                "EndpointID": "3e73ec82c70996a95be73c5c3a2e42373f0992bc58d838777e3aaf045eee26d1",
                "MacAddress": "02:42:0a:00:01:04",
                "IPv4Address": "10.0.1.4/24",
                "IPv6Address": ""
            },
            "lb-overnet": {
                "Name": "overnet-endpoint",
                "EndpointID": "7327fc94ed2938898b462a114f94ac80ec37cb397835021ae3ccee44738bcc35",
                "MacAddress": "02:42:0a:00:01:05",
                "IPv4Address": "10.0.1.5/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "828716f7518d",
                "IP": "192.168.0.28"
            }
        ]
    }
]
```

Masuk kembali ke dalam container dan jalankan `cat /etc/resolv.conf` untuk melihat name server.
```bash
root@8b046fcee206:/# cat /etc/resolv.conf
search 51ur3jppi0eupdptvsj42kdvgc.bx.internal.cloudapp.net
nameserver 127.0.0.11
options ndots:0
```

Lalu ping `myservice`.
```bash
root@8b046fcee206:/# ping -c5 myservice
PING myservice (10.0.1.2) 56(84) bytes of data.
64 bytes from 10.0.1.2 (10.0.1.2): icmp_seq=1 ttl=64 time=0.170 ms
64 bytes from 10.0.1.2 (10.0.1.2): icmp_seq=2 ttl=64 time=0.084 ms
64 bytes from 10.0.1.2 (10.0.1.2): icmp_seq=3 ttl=64 time=0.052 ms
64 bytes from 10.0.1.2 (10.0.1.2): icmp_seq=4 ttl=64 time=0.073 ms
64 bytes from 10.0.1.2 (10.0.1.2): icmp_seq=5 ttl=64 time=0.057 ms

--- myservice ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3998ms
rtt min/avg/max/mdev = 0.052/0.087/0.170/0.042 ms
```

Pada ping tersebut yang membalas adalah 10.0.1.2 yaitu virtual IP nya myservice.
```bash
[node1] (local) root@192.168.0.28 ~
$ docker service inspect myservice
```
```json
[
    {
        "ID": "d5uz736khvjmozoz06o35i2qu",
        "Version": {
            "Index": 15
        },
        "CreatedAt": "2020-12-09T14:48:24.359447795Z",
        "UpdatedAt": "2020-12-09T14:48:24.3611596Z",
        "Spec": {
            "Name": "myservice",
            "Labels": {},
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "ubuntu:latest@sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c",
                    "Args": [
                        "sleep",
                        "infinity"
                    ],
                    "Init": false,
                    "StopGracePeriod": 10000000000,
                    "DNSConfig": {},
                    "Isolation": "default"
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "Delay": 5000000000,
                    "MaxAttempts": 0
                },
                "Placement": {
                    "Platforms": [
                        {
                            "Architecture": "amd64",
                            "OS": "linux"
                        },
                        {
                            "OS": "linux"
                        },
                        {
                            "Architecture": "arm64",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "ppc64le",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "s390x",
                            "OS": "linux"
                        }
                    ]
                },
                "Networks": [
                    {
                        "Target": "kiws0g4orgt52lvf5zw13d1uz"
                    }
                ],
                "ForceUpdate": 0,
                "Runtime": "container"
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 2
                }
            },
            "UpdateConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "RollbackConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {
                "Mode": "vip"
            },
            "VirtualIPs": [
                {
                    "NetworkID": "kiws0g4orgt52lvf5zw13d1uz",
                    "Addr": "10.0.1.2/24"
                }
            ]
        }
    }
]
```

## Cleaning up
Untuk menghapus service docker dapat menggunakan perintah berikut.
`docker service rm myservice`
Lalu cek dengan `docker ps`
Lalu untuk menghapus docker gunakan `docker kill <yourcontainerid1> <yourcontainerid2>`
Lalu setelah itu hapus node 1 dan node 2 menggunakan docker swarm leave --force command to do that.
`docker swarm leave --force`