---
title: "With Docker Quick Start"
date: 2018-04-11T16:57:25+08:00
---

### 准备环境
参考[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)

docker安装：https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1

系统：centos7

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

    git --version    ## 查看自带的版本git version 1.8.3.1
    yum remove git   ## 移除原来的版本

官方git:https://github.com/git/git/releases

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

### 开始构建
按照[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)导入依赖git clone并添加相关代码后执行构建

    ./mvnw install dockerfile:build
运行

    docker run -p 8080:8080 -t springio/gs-spring-boot-docker
错误信息

    [root@localhost initial]# docker run -p 8080:8080 -t springio/gs-spring-boot-docker
    docker: Error response from daemon: driver failed programming external connectivity on endpoint zen_heyrovsky (2aa8004e111f72975c941012fe7f58082b443115592a60d454b4b17a3370c06b):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8080 -j DNAT --to-destination 172.17.0.2:8080 ! -i docker0: iptables: No chain/target/match by that name.
     (exit status 1)).
根据提示应该是iptables防火墙问题，CentOS7默认最小安装没有iptables且开始使用firewalld

    yum install firewalld
在CentOS7中service被systemctl替代，启动

    systemctl start firewalld
重启docker

    systemctl restart docker
重新运行

    docker run -p 8080:8080 -t springio/gs-spring-boot-docker
按照http://www.cnblogs.com/flasheryu/p/5919657.html 解决，其中说了一句

> 在启动firewalld之后，iptables被激活，此时没有docker chain，重启docker后被加入到iptable里面。

ok,终于完成quick-start

#### 总结
第一次使用会遇到很多坑，官方文档说15分钟，我用了3天左右......很多环境配置都是第一次，尤其是centos7与6有很大不同，好多还是不理解。打算在接下对项目的逐步改造中去理解。对了，生产环境一定要配置好firewalld!!!
