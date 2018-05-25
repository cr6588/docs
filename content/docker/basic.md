---
title: "Basic"
date: 2018-05-24T10:34:19+08:00
draft: true
---

    #docker帮助
    docker --help
    #docker镜像列表
    docker images
    #docker运行容器列表
    docker ps
    #停止容器
    docker stop <docker ps中的container id>
    #创建加载磁盘的容器
    docker run -it -v /data/dockerbuild:/mnt apline /bin/sh
    #构建容器，最后加.表示当前目录
    docker build -t jdk8/alpine:v0.1 .
# 关于Dockerfile的ADD命令 https://blog.csdn.net/kiloveyousmile/article/details/80211351
#添加文件
例如：

ADD my.cnf /etc/mysql
1
ADD my.cnf /etc/mysql/
1
以上两条命令均可以将my.cnf文件添加到/etc/mysql文件夹下面。

添加文件夹

Dockerfile添加文件夹，则必须镜像中存在和当前文件夹同名的文件夹才行。例如，我希望将当前目录下的views文件夹添加到docker镜像中的app文件夹下。也许你会采用这样的方式：

ADD views /app
1
这样其实并不能实现，应该通过下面的方式：

ADD views /app/views
1
也就是说：镜像中存在和当前需要拷贝或添加的文件夹同名的文件夹时，才能够拷贝或添加成功。
#bash安装
https://www.oschina.net/translate/alpine-linux-install-bash-using-apk-command


#Cenos基础镜像
使用https://cr.console.aliyun.com/?spm=5176.2020520130.aliyun_topbar.7.PSGJpE#/accelerator 阿里云镜像加速
docker hub直接搜索官方centos镜像
启动并进入容器
docker start -a <container_Id>
crtl+d直接退出容器停止运行
crtl+p之后crtl+q退出容器不会停止运行
#redis后台启动
redis.conf中
daemonize yes
#(iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 32777 -j DNAT --to-destination 172.17.0.2:8082 ! -i docker0: iptables: No chain/target/match by that name.
再重启firewalld后映射端口可能报此错，重启docker即可