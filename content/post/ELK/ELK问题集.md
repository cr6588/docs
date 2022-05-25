---
title: "ELK问题集"
date: 2018-07-17T11:00:28+08:00
categories: ["ELK"]
---
get-started-elastic-stack

https://www.elastic.co/guide/en/elastic-stack-overview/6.3/get-started-elastic-stack.html#install-elasticsearch

1. centos下start ok, curl失败，status提示which: no java in xxx.

    [root@localhost data]# sudo service elasticsearch start
    Starting elasticsearch (via systemctl):                    [  OK  ]
    [root@localhost data]# curl http://127.0.0.1:9200
    curl: (7) Failed connect to 127.0.0.1:9200; Connection refused
    [root@localhost data]# sudo service elasticsearch status
    ● elasticsearch.service - Elasticsearch
       Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; vendor preset: disabled)
       Active: failed (Result: exit-code) since Tue 2018-07-17 10:37:56 CST; 36s ago
         Docs: http://www.elastic.co
      Process: 20370 ExecStart=/usr/share/elasticsearch/bin/elasticsearch -p ${PID_DIR}/elasticsearch.pid --quiet (code=exited, status=1/FAILURE)
     Main PID: 20370 (code=exited, status=1/FAILURE)
    
    Jul 17 10:37:56 localhost.localdomain systemd[1]: Started Elasticsearch.
    Jul 17 10:37:56 localhost.localdomain systemd[1]: Starting Elasticsearch...
    Jul 17 10:37:56 localhost.localdomain elasticsearch[20370]: which: no java in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin)
    Jul 17 10:37:56 localhost.localdomain systemd[1]: elasticsearch.service: main process exited, code=exited, status=1/FAILURE
    Jul 17 10:37:56 localhost.localdomain systemd[1]: Unit elasticsearch.service entered failed state.
    Jul 17 10:37:56 localhost.localdomain systemd[1]: elasticsearch.service failed.

将/etc/sysconfig/elasticsearch中JAVA_HOME设置成本机的JAVA_HOME

    # Elasticsearch Java path
    JAVA_HOME=/data/jdk1.8.0_161

2. kibana外网不能访问
修改kibana.yml

    server.host: "0.0.0.0"

3. logstash提示could not find java; set JAVA_HOME or ensure java is in PATH

    [root@localhost data]# echo $JAVA_HOME
    /data/jdk1.8.0_161
有java目录,是目录路径有特殊符号改成/data/jdk之后，先卸载再重新安装

    [root@localhost data]# rpm -e logstash-6.3.1.rpm 
    error: package logstash-6.3.1.rpm is not installed
    [root@localhost data]# rpm -i logstash-6.3.1.rpm 
    warning: logstash-6.3.1.rpm: Header V4 RSA/SHA512 Signature, key ID d88e42b4: NOKEY
        package logstash-1:6.3.1-1.noarch is already installed
    [root@localhost data]# rpm -e logstash-1:6.3.1-1.noarch
    warning: /etc/logstash/logstash.yml saved as /etc/logstash/logstash.yml.rpmsave
    [root@localhost data]# rpm -i logstash-6.3.1.rpm 
    warning: logstash-6.3.1.rpm: Header V4 RSA/SHA512 Signature, key ID d88e42b4: NOKEY
    Using provided startup.options file: /etc/logstash/startup.options
    Successfully created system startup script for Logstash

4. Elasticsearch is still initializing the kibana index

    https://stackoverflow.com/questions/31201051/elasticsearch-is-still-initializing-the-kibana-index

    Warning: Removing .kibana index will make you lose all your kibana settings (indexes, graphs, dashboards)
    
    This behavior is sometimes caused by an existing .kibana index. Kindly delete the .kibana index in elasticsearch using following command:
    
    curl -XDELETE http://localhost:9200/.kibana
    After deleting the index, restart Kibana.
    
    If the problem still persists, and you are willing to lose any existing data, you can try deleting all indexes using following command:
    
    curl -XDELETE http://localhost:9200/*
    Followed by restarting Kibana.

    Note: localhost:9200 is the elasticsearch server's host:port, which may be different in your case

5. tar.gz安装Elasticsearch5.0时，root用户运行出错
因为安全问题elasticsearch 不让用root用户直接运行，所以要创建新用户
建议创建一个单独的用户用来运行ElasticSearch
创建es用户组及es用户

    groupadd es
    #创建用户名为es密码为elasticsearch的用户，并添加到es组
    useradd es -g es -p elasticsearch
    #创建ELK目录
    mkdir ELK
    chown es:es ELK
    chown es:es elasticsearch-5.5.0.tar.gz
    su es
    tar -xzf elasticsearch-5.5.0.tar.gz
    vi config/elasticsearch.yml

     path.data: /data/ELK/elasticsearch-5.5.0/data
    #
    # Path to log files:
    #
     path.logs: /data/ELK/elasticsearch-5.5.0/logs

[es@localhost elasticsearch-5.5.0]$ mkdir data
[es@localhost elasticsearch-5.5.0]$ mkdir logs
[es@localhost elasticsearch-5.5.0]$ ./bin/elasticsearch

6. 在kibana中输入filebeat*无法创建index
有可能是filebeat已经将index存进es中且自身所在的data中也已记录相关，但index被删除导致再次创建时因为需要搜集的文件日志已经被记录所以无法创建，进而说明DELETE index时并没有删除filebeat中的相关记录。
7. es单机集群配置

https://www.elastic.co/guide/en/elasticsearch/reference/5.5/important-settings.html#bootstrap.memory_lock

     cluster.name: esCluster
     node.name: node2
     path.data: /data/ELK/esnode2/data
     path.logs: /data/ELK/esnode2/logs
     network.host: 127.0.0.1
     http.port: 9201
     discovery.zen.ping.unicast.hosts: ["127.0.0.1", "127.0.0.1:9201"]
     discovery.zen.minimum_master_nodes: 2
     action.destructive_requires_name: true
8. ~~es报错（已过时）~~

    [1]: max number of threads [1024] for user [es] is too low, increase to at least [2048]
    [2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    [3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk

[1]vi /etc/security/limits.conf
添加es - nproc 2048 #es为运行ELK相关的用户
另外一个65536使用es - nofile 65536
[2] 

    https://github.com/docker-library/elasticsearch/issues/111
    set max_map_count value (Linux)
    sudo sysctl -w vm.max_map_count=262144
[3] es配置文件加入

    bootstrap.memory_lock: false 


9. 非内网集群时
    https://www.jianshu.com/p/149a8da90bbc
由于transport.tcp.port默认是9300，之间是以此端口通信而非9200所以当防火墙只开放了9200时虽然curl http://xxx:9200能正确显示，但是集群会一直提示节点

10. ps -ef|grep kibana, kill 之后无法杀死

    bin/kibana是脚本，启动的是另一个程序。用ps -ef|grep node找到进程杀掉，kibana找不到

11. es运行久了之后内存溢出

这个问题一般隔了一段时间都会出现一次，在网上找了半天，都出现了这个问题都是堆内存溢出，但都没什么好的解决方式。
目前是解决方式是

    1.将es占用端口例如30200改成其它端口启动，防止生产环境还在提交写入请求
    2.将kibana的es端口改成1中的端口重启
    3.在kibana中删除部分indexs然后将端口改回重启

12. 启动提示could not find java in bundled JDK at /root/data/elasticsearch-7.16.2/jdk/bin/java
启动提示could not find javaxxxxxx，然后去/root/data/elasticsearch-7.16.2/jdk/bin/java -version时，又提示
error while loading shared libraries: libjli.so。将elasticsearch-7.16.2目录从root目录移动至es用户目录，再进行启动

13.es报错
````
bootstrap check failure [1] of [3]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
bootstrap check failure [2] of [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
bootstrap check failure [3] of [3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
````
[1]vi /etc/security/limits.conf
添加es - nproc 2048 #es为运行ELK相关的用户
另外一个65536使用es - nofile 65536
[2]
vi /etc/sysctl.conf
末尾增加：
vm.max_map_count=262144
之后执行：
sysctl -p
[3]
增加3个中的一个配置