---
title: "virtualbox安装centos与docker"
date: 2018-05-24T10:34:19+08:00
categories: ["docker"]
---

通过virtualbox安装CentOS-7-x86_64-Minimal-1804.iso后网络模式选择桥接重启
#禁用防火墙
systemctl stop firewalld
#yum install net-tools
ifconfig
#远程连接后参照 [Centos官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1)

#### docker安装好之后无法启动，journalctl -xe查看有如下提示
````shell
7月 11 15:32:50 cr dockerd[8076]: Error starting daemon: Error initializing network controller: list bridge addresses failed: no available network
````
手动添加
````shell
ip link add name docker0 type bridge 
ip addr add dev docker0 172.17.0.1/16 
````
之后再添加服务
````shell
systemctl start docker && systemctl enable docker
````