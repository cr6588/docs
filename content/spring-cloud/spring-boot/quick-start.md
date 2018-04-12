---
title: "Quick Start"
date: 2018-04-11T16:57:25+08:00
draft: true
---

参考[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)准备环境
docker安装：https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1
使用centos7
jdk配置

    vi /etc/profile

    export JAVA_HOME=/usr/java/jdk
    export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
    export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

    echo $JAVA_HOME

    . /etc/profile

    echo $JAVA_HOME
maven配置
改下载仓库

    <localRepository>/data/mvn_resp</localRepository>
改成阿里源

    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>

git下载

    [root@Git ~]# git --version    ## 查看自带的版本git version 1.8.3.1
    [root@Git ~]# yum remove git   ## 移除原来的版本

https://github.com/git/git/releases

    wget https://github.com/git/git/archive/v2.17.0.tar.gz
    tar -zxf v2.17.0.tar.gz
    cd git-2.17.0
    make configure
    ./configure --prefix=/data/git ##配置目录
    make profix=/data/git
    make install
缺少xxx,yum install xxx
加入环境变量

    vi /etc/profile
    export PATH=$PATH:/data/git/bin
保存后生效

    . /etc/profile
    
    