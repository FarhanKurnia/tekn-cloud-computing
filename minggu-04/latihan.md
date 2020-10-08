# OpenStack Deployment on Ubuntu with DevStack
### Step 1: Update Ubuntu system
Login to your Ubuntu system – Can be Desktop or VM in the Cloud and update it.
```bash
farhan@Arctic:~$ sudo apt-get update
farhan@Arctic:~$ sudo apt -y upgrade 
farhan@Arctic:~$ sudo apt -y dist-upgrade
```
Reboot it after an upgrade.
```bash
farhan@Arctic:~$sudo reboot
```
### Step 2: Add Stack User
Devstack should be run as a non-root user with sudo enabled. If you’re running your instance in the cloud, standard logins to cloud images such as “centos” or “ubuntu” or “cloud-user” are usually fine.

For other installations of Ubuntu 18.04, run the commands below to create DevStack deployment user.
```bash
farhan@Arctic:~$ sudo useradd -s /bin/bash -d /opt/stack -m stack
```
Enable sudo privileges for this user without need for a password.
```bash
farhan@Arctic:~$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
stack ALL=(ALL) NOPASSWD: ALL
```
Switch to stack user to test.
```bash
farhan@Arctic:~$ sudo su - stack 
stack@Arctic:~$ sudo su -
root@Arctic:~# 
```
### Step 3: Download DevStack
Clone Destack deployment code from Github.
```bash
root@Arctic:~# su - stack
stack@Arctic:~$ git clone https://git.openstack.org/openstack-dev/devstack
Cloning into 'devstack'...
warning: redirecting to https://opendev.org/openstack/devstack/
remote: Enumerating objects: 46240, done.
remote: Counting objects: 100% (46240/46240), done.
remote: Compressing objects: 100% (21135/21135), done.
remote: Total 46240 (delta 32691), reused 37514 (delta 24400)
Receiving objects: 100% (46240/46240), 9.46 MiB | 871.00 KiB/s, done.
Resolving deltas: 100% (32691/32691), done.
```
Create a local.conf file with 4 passwords and Host IP address.
```bash
stack@Arctic:~$ cd devstack
stack@Arctic:~/devstack$ nano local.conf
```
Add:
```bash
[[local|localrc]]

# Password for KeyStone, Database, RabbitMQ and Service
ADMIN_PASSWORD=StrongAdminSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Host IP - get your Server/VM IP address from ip addr command
HOST_IP=192.168.10.100
```
### Step 4: Start Openstack Deployment on Ubuntu 18.04 with DevStack
Now that you’ve configured the minimum required config to get started with DevStack, start the installation of Openstack.
```bash
stack@Arctic:~/devstack$ ./stack.sh
```

DevStack will install;

[a] Keystone – Identity Service
[b] Glance – Image Service
[c] Nova – Compute Service
[d] Placement – Placement API
[e] Cinder – Block Storage Service
[f] Neutron – Network Service
[i] Horizon – Openstack Dashboard

This will take a 15 – 20 minutes, largely depending on the speed of your internet connection. At the end of the installation process, you should see output like this:



