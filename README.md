# linux-server

## Introduction
In this project, I set up a linux server on an AWS Lightsail Ubuntu machine. The server is running Apache, and the web application is based on Flask and PostgreSQL.

## Configurations
Several configurations needed to be made to this server before it was functional. These are described below.

### Acquire Lightsail Machine
Open an AWS account [here](aws.amazon.com), and create an OS only machine with Ubuntu.


### Install updates to your system
Run these commands and then reboot. Repeat this process until you login to your machine and it says that you have 0 updates remaining.

```
sudo apt update
sudo apt full-upgrade
```

### Create the `grader` user and give it sudo privileges
Connect to your Lightsail machine using the built-in browser SSH window and run the following commands.

```
sudo adduser grader
sudo usermod -aG sudo grader
```

### Configure and enable UFW
Allow only three specific ports, enable the `ufw`, and finally restart the ssh service.

```
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw allow 2200/tcp
sudo ufw enable
sudo service ssh restart
```

### Configure the time zone
Run `sudo dpkg-reconfigure tzdata`

### Install and configure Apache



## Bibliography
1. [https://wiki.apache.org/httpd/ClientDeniedByServerConfiguration]
2. [https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9]
3. [https://stackoverflow.com/questions/38298652/permissionerror-errno-13-permission-denied-flask-run]
4. [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04]
5. [https://github.com/hicham-alaoui/ha-linux-server-config]
6. 