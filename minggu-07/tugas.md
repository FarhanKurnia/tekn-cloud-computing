## Image Docker

1. Disini saya mencari image ubuntu pada Docker melalui terminal di Ubuntu.

```bash
farhan@Arctic:~$ docker search ubuntu
NAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   11456               [OK]                
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   470                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   249                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   228                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   110                 [OK]                
neurodebian                                               NeuroDebian provides neuroscience research s…   72                  [OK]                
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5      ubuntu-16-nginx-php-phpmyadmin-mysql-5          50                                      [OK]
ubuntu-debootstrap                                        debootstrap --variant=minbase --components=m…   44                  [OK]                
nuagebec/ubuntu                                           Simple always updated Ubuntu docker images w…   24                                      [OK]
i386/ubuntu                                               Ubuntu is a Debian-based Linux operating sys…   24                                      
1and1internet/ubuntu-16-apache-php-5.6                    ubuntu-16-apache-php-5.6                        14                                      [OK]
1and1internet/ubuntu-16-apache-php-7.0                    ubuntu-16-apache-php-7.0                        13                                      [OK]
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mariadb-10   ubuntu-16-nginx-php-phpmyadmin-mariadb-10       11                                      [OK]
1and1internet/ubuntu-16-nginx-php-5.6-wordpress-4         ubuntu-16-nginx-php-5.6-wordpress-4             7                                       [OK]
1and1internet/ubuntu-16-apache-php-7.1                    ubuntu-16-apache-php-7.1                        6                                       [OK]
darksheer/ubuntu                                          Base Ubuntu Image -- Updated hourly             5                                       [OK]
pivotaldata/ubuntu                                        A quick freshening-up of the base Ubuntu doc…   4                                       
1and1internet/ubuntu-16-nginx-php-7.0                     ubuntu-16-nginx-php-7.0                         4                                       [OK]
1and1internet/ubuntu-16-nginx-php-7.1-wordpress-4         ubuntu-16-nginx-php-7.1-wordpress-4             3                                       [OK]
pivotaldata/ubuntu16.04-build                             Ubuntu 16.04 image for GPDB compilation         2                                       
1and1internet/ubuntu-16-php-7.1                           ubuntu-16-php-7.1                               1                                       [OK]
1and1internet/ubuntu-16-sshd                              ubuntu-16-sshd                                  1                                       [OK]
smartentry/ubuntu                                         ubuntu with smartentry                          1                                       [OK]
pivotaldata/ubuntu-gpdb-dev                               Ubuntu images for GPDB development              1                                       
pivotaldata/ubuntu16.04-test                              Ubuntu 16.04 image for GPDB testing             0     
```
Pada kolom OFFICIAL, OK menandakan image tersebut dibuat dan didukung oleh perusahaan yang ada di balik proyek ini.

2. Jalankan perintah berikut ini untuk mengunduh image ubuntu resmi:
```bash
farhan@Arctic:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
6a5697faee43: Downloading [=====>                                             ]  2.939MB/28.56MB
ba13d3bc422b: Download complete 
a254829d9e55: Download complete 
```

3. Tunggu sampai pull complate
```bash
farhan@Arctic:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
6a5697faee43: Pull complete 
ba13d3bc422b: Pull complete 
a254829d9e55: Pull complete 
Digest: sha256:fff16eea1a8ae92867721d90c59a75652ea66d29c05294e6e2f898704bdb8cf1
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

4. Lalu setelah itu jalankan container
```bash
farhan@Arctic:~$ docker run -it ubuntu
```

5. Disini saya sudah masuk container ubuntu:
```bash
root@c1876a8467b1:/# cat /etc/issue
Ubuntu 20.04.1 LTS \n \l
```

6. Untuk melihat Untuk melihat semua kontainer — aktif dan tidak aktif, jalankan docker ps -a:
```bash
farhan@Arctic:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
c1876a8467b1        ubuntu              "/bin/bash"         3 minutes ago       Exited (0) 23 seconds ago                       strange_khorana
7ef13f9584ce        hello-world         "/hello"            30 minutes ago      Exited (0) 30 minutes ago     
```

---
Referensi:
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-id)