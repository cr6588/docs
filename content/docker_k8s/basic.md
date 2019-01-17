---
title: "Basic"
date: 2018-05-24T10:34:19+08:00
draft: true
---

#### Docker基本命令

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
    docker run -p 127.0.0.1:6379:6379 -it -v /data/dockerbuild:/mnt --name erp-env2 erp-env:v0.1 /bin/bash
    docker run -p 20001:20001 -p 21001:21001 -p 22001:22001 -p 23001:23001 -p 24001:24001 -p 25001:25001 -p 26001:26001 -p 27001:27001 -it -v /data/dockerbuild:/mnt --name erp-base sjdf/erp-base:v0.1 --add-host=static.scdf.cn:118.123.12.120 --add-host=zookeeper.jtongi.cn:172.17.0.1 --add-host=user.mysql.jtongi.cn:172.17.0.1 --add-host=picture.scdf.cn:118.123.12.120 --add-host=es.jtongi.cn:118.123.12.107 /bin/bash 

    docker run -p 20001:20001 -p 21001:21001 -p 22001:22001 -p 23001:23001 -p 24001:24001 -p 25001:25001 -p 26001:26001 -p 27001:27001 -it -v /data/dockerbuild:/mnt --add-host=static.scdf.cn:118.123.12.120 --add-host=zookeeper.jtongi.cn:172.17.0.1 --add-host=user.mysql.jtongi.cn:172.17.0.1 --add-host=picture.scdf.cn:118.123.12.120 --add-host=es.jtongi.cn:118.123.12.107 --name jdk8-centos7 cr6588/jdk8-centos7 /bin/bash 
    
    docker run -p 127.0.0.1:8080:8080 -it -v /data/dockerbuild:/mnt --add-host=static.scdf.cn:118.123.12.120 --add-host=zookeeper.jtongi.cn:172.17.0.1 --add-host=user.mysql.jtongi.cn:172.17.0.1 --add-host=picture.scdf.cn:118.123.12.120 --add-host=es.jtongi.cn:118.123.12.107 --name 8080 8080 /bin/bash
    #docker使用的网络实际上和宿主机一样
    docker run -it -v /data/dockerbuild:/mnt --net=host --name nethost cr6588/jdk8-centos7 /bin/bash
    # 添加单个hosts
    docker run -it nginx --add-host=localhost:127.0.0.1
    # 添加多个hosts
    docker run -it nginx --add-host=localhost:127.0.0.1 --add-host=example.com:127.0.0.1 
    # 一个ip对应多个hosts
    docker run -it nginx --add-host="localhost example.com":127.0.0.1

    docker run -p 20001:20001  -it -v /data/dockerbuild:/mnt --add-host=zookeeper.jtongi.cn:172.17.0.1 --name erp-cache erp-cache:latest
    
    #构建容器，最后加.表示当前目录
    docker build -f redis-zookeeper -t redis-zookeeper-houtai .
    docker build -t jdk8/alpine:v0.1 .
    #重启docker时自动启动容器在run时加入--restart=always
    

#### [关于Dockerfile的ADD命令](https://blog.csdn.net/kiloveyousmile/article/details/80211351)
##### 添加文件
例如：

    ADD my.cnf /etc/mysql
    ADD my.cnf /etc/mysql/

以上两条命令均可以将my.cnf文件添加到/etc/mysql文件夹下面。

添加文件夹

Dockerfile添加文件夹，则必须镜像中存在和当前文件夹同名的文件夹才行。例如，我希望将当前目录下的views文件夹添加到docker镜像中的app文件夹下。也许你会采用这样的方式：

    ADD views /app

这样其实并不能实现，应该通过下面的方式：

    ADD views /app/views

也就是说：镜像中存在和当前需要拷贝或添加的文件夹同名的文件夹时，才能够拷贝或添加成功。

#### [bash安装](https://www.oschina.net/translate/alpine-linux-install-bash-using-apk-command)
#### Cenos基础镜像

1. 使用[阿里云镜像加速](https://cr.console.aliyun.com/?spm=5176.2020520130.aliyun_topbar.7.PSGJpE#/accelerator)[^1]

2. docker hub直接搜索官方centos镜像

3. 启动并进入容器

    docker start -a <container_Id>

4. 退出容器

* crtl+d直接退出容器停止运行
* crtl+p之后crtl+q退出容器不会停止运行

#### redis后台启动

redis.conf中

    daemonize yes

#### iptables failed报错
在重启firewalld后映射端口可能报

    (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 32777 -j DNAT --to-destination 172.17.0.2:8082 ! -i docker0: iptables: No chain/target/match by that name.
    
重启docker即可

#### 映射多个指定端口使用多个-p，自动分配映射端口使用-P

    docker run -d -p 80:80 -p 22:22
    docker run -p 6379:6379 -p 2181:2181 -it -v /data/dockerbuild:/mnt --name erp-env2 erp-env:v0.1

#### 磁盘占满
docker默认存放目录在/var/lib/docker，磁盘满了之后将其移动到新目录并加入软连接

    mv /var/lib/docker /docker
    ln -s /docker/docker /var/lib/docker
#### docker私服搭建
##### 生成个人证书
mkdir -p certs
openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt
##### 创建用户
 mkdir auth
 docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd

> 密码以2个!!结尾会出错

vi docker-compose.yml



registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /path/data:/var/lib/registry
    - /path/certs:/certs
    - /path/auth:/auth

/path/data:宿主机挂载的磁盘

docker-compose up -d

##### 查看日志
journalctl -u docker.service
##### 容器图形化查看工具portainer
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
> (COMMAND_FAILED: '/usr/sbin/iptables -w2 -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.2 --dport 9000 -j ACCEPT' failed: iptables: No chain/target/match by that name.
> 重启防火墙

##### 说明
[^1]: 登录阿里云->容器镜像服务->镜像加速器