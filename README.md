# linux-server

## Introduction
In this project, I set up a linux server on an AWS Lightsail Ubuntu machine. The server is running Apache, and the web application is based on Flask and PostgreSQL.

## Configurations
Several configurations needed to be made to this server before it was functional. These are described below.

### Acquire Lightsail Machine
Open an AWS account [here](aws.amazon.com), and create an OS only machine with Ubuntu.

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
