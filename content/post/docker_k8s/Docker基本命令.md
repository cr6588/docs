---
title: "Docker基本命令"
date: 2018-05-24T10:34:19+08:00
categories: ["docker"]
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
#### docker registry:2私服搭建（已弃用）
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

docker-compose安装用yum provides docker-compose搜索。若有结果用yum install安装，没有结果见https://docs.docker.com/compose/install/

docker-compose up -d

私服镜像图形化查看工具
https://github.com/Joxit/docker-registry-ui

    #进入私服容器修改配置文件允许跨域
    docker exec -it 91a1a0d5eae8 /bin/sh
    vi /etc/docker/registry/config.yml
    Access-Control-Allow-Origin: ['http://192.168.199.206:8001']
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Allow-Headers: ['Authorization']
    Access-Control-Max-Age: [1728000]
    Access-Control-Allow-Credentials: [true]
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
    

    #URL中https是由于私服加入了证书
    docker run -d -p 8001:80 -e URL=https://192.168.199.206:5000 -e DELETE_IMAGES=true joxit/docker-registry-ui:static
由于docker-registry-ui对删除功能支持不完善，弃用
#### docker nexus3私服搭建
根据[nexus3 dockerhub](https://hub.docker.com/r/sonatype/nexus3/)说明

    #创建nexus-data目录持久化存储
    mkdir /docker/nexus-data && chown -R 200 /docker/nexus-data
    #由于docker端口会无视防火墙，直接使用主机的网络端口，不进行映射。默认会使用8081端口
    docker run -d --net=host --name nexus -v /docker/nexus-data:/nexus-data sonatype/nexus3
访问ip:8081进行登录，默认用户名密码为admin/admin123，（新版本已经更改/nexus-data/admin.password，登录时会有提示）
进入设置页面创建一个docker(hosted)类型的Repositories，且其http端口设置为8082，其余默认设置即可
由于docker login是采用https所以需要将其访问改成https，这里有2种方式，1种是采用nginx进行代理，另外一种是将证书与nexus3关联。因为有很多软件逐渐都变成需要https,而nginx这种方式跟软件无关，所以这里采用第一种方式

    #创建一个自定义证书
    mkdir -p certs
    openssl req \
      -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
      -x509 -days 9999 -out certs/domain.crt
[安装nginx](https://nginx.org/en/linux_packages.html)

    #编辑配置文件,将5000端口代理到8082端口
    vi /etc/nginx/nginx.conf
    #http末尾增加
    http {
        #....

        client_max_body_size 2g;
        upstream backend.example.com {
            server 你的ip:5000;
        }
        server {
            listen      5000 ssl;
            server_name 你的ip;
    
            #证书路径
            ssl_certificate        /docker/certs/domain.crt;
            ssl_certificate_key    /docker/certs/domain.key;
            #ssl_client_certificate /etc/ssl/certs/ca.crt;
            ssl_session_cache shared:SSL:1m;
            ssl_session_timeout 5m;
            ssl_ciphers HIGH:!aNULL:!MD5;
            ssl_prefer_server_ciphers on;
    
            location / {
                proxy_pass http://你的ip:8082;
            #...
        }
    }
    #启动nginx
    systemctl start nginx
    #将ip:5000加入到daemon.json
    vi /etc/docker/daemon.json 
    {
      "insecure-registries" : ["ip:5000"]
    }
    #重启docker
    systemctl restart docker
    #重启nexus3容器
    docker start nexus
    #稍等一会之后登陆
    docker login ip:5000
    #输入admin/admin123测试
    #接着上传一个镜像到nexus3
    docker pull nginx
    docker tag nginx ip:5000/nginx
    docker push ip:5000/nginx
    #在ui上查看是否有该镜像

##### 删除镜像

 之前ui删除都是软删除，并未在磁盘中把空间占用释放
 参照官方说明[How to delete docker images from Nexus Repository Manager](https://support.sonatype.com/hc/en-us/articles/360009696054-How-to-delete-docker-images-from-Nexus-Repository-Manager)，先在Ui上删除刚刚创建的ip:5000/nginx镜像，接着查看磁盘空间
    
    du -sh /docker/nexus-data

之后创建一个Docker - Delete unused manifests and images类型的task将各个孤立未被其它镜像关联的层进行软删除，之后创建一个'Admin - Compact blob store'类型的task以真正释放磁盘空间。创建好了之后对2个task依次执行，再查看磁盘空间可以看到空间已经减少。

自动清理
在cleanup policies中创建一个策略，选择了docker,以及根据上传超过多少天，上次下载多少天来删除。可以预览是否有符合条件的结果。创建之后还需要去Repositories中相应的库中把Cleanup Policy指定为刚才创建的策略，然后系统会有一个Admin - Cleanup repositories using their associated policies的task来执行这个策略。手动执行下在执行前2个task,若刚才预览有结果，磁盘空间会减少。


##### 查看日志
journalctl -u docker.service
##### 本机容器图形化查看工具portainer
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
> (COMMAND_FAILED: '/usr/sbin/iptables -w2 -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.2 --dport 9000 -j ACCEPT' failed: iptables: No chain/target/match by that name.
> 重启防火墙

##### 显示docker ps完整信息
docker ps --no-trunc

##### 显示docker磁盘使用docker volume相关
````shell
[root@cr ~]# docker volume -h
Flag shorthand -h has been deprecated, please use --help

Usage:	docker volume COMMAND

Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes

Run 'docker volume COMMAND --help' for more information on a command.
[root@cr ~]# docker volume ls
DRIVER              VOLUME NAME
local               jenkins_home
[root@cr ~]# docker volume inspect jenkins_home
[
    {
        "CreatedAt": "2019-07-11T17:20:33+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/jenkins_home/_data",
        "Name": "jenkins_home",
        "Options": null,
        "Scope": "local"
    }
]
````
#### 以指定用户进入容器执行shell
docker exec --user root -it c0d5cf8d7b0a bash
#### 启动docker自动启动容器
docker run --restart=always
如果已经启动了则可以使用如下命令：
docker update --restart=always <CONTAINER ID>

##### 说明
[^1]: 登录阿里云->容器镜像服务->镜像加速器