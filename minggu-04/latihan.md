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


