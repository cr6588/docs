---
title: "Monitor"
date: 2018-12-07T14:47:40+08:00
draft: true
---

# 使用prometheus与grafana进行监控
## 安装prometheus
参照官网[installation](
https://prometheus.io/docs/prometheus/latest/installation/)进行docker安装.它推荐使用[Data Volume Container](https://docs.docker.com/storage/volumes/#create-and-manage-volumes) 安装。

    #自建volume
    docker volume create prometheus-data
    #查看volume信息
    docker volume inspect prometheus-data
    [
        {
            "CreatedAt": "2018-12-07T14:07:20+08:00",
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "/docker/docker/volumes/prometheus-data/_data",
            "Name": "prometheus-data",
            "Options": {},
            "Scope": "local"
        }
    ]
    #编辑配置.以监控本机与mysql为例。
    cd /docker/docker/volumes/prometheus-data/_data
    vi prometheus.yml

    global:
    scrape_interval:     15s
    evaluation_interval: 15s

    scrape_configs:
    - job_name: linux
        static_configs:
        - targets: ['172.17.0.1:9100']
            labels:
            instance: db1

    - job_name: mysql
        static_configs:
        - targets: ['172.17.0.1:9104']
            labels:
            instance: db1

> 172.17.0.1是docker中宿主机的默认ip.

    #后台启动运行加入-d
    docker run -d -p 9090:9090 -v prometheus-data:/prometheus-data \
       prom/prometheus --config.file=/prometheus-data/prometheus.yml
    #浏览器访问http://ip:9090/targets，看到linux与mysql2个job的state都是down

## 安装Node-exporter
官网有一个[Node-exporter](https://prometheus.io/docs/guides/node-exporter/)的引导可以参考,github见https://github.com/prometheus/node_exporter。
下载在https://github.com/prometheus/node_exporter/releases中。

    tar zxf node_exporter-0.17.0.linux-amd64.tar.gz
    #后台运行
    nohup ./node_exporter  > log.log 2>&1 &
    #防火墙打开9100端口
    firewall-cmd --zone=public --add-port=9100/tcp --permanent  && firewall-cmd --reload
    #查看http://ip:9090/targets其状态是否为up
## 安装mysqld_exporter
参见其[github](https://github.com/prometheus/mysqld_exporter)

    wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.11.0/mysqld_exporter-0.11.0.linux-amd64.tar.gz
    tar zxf mysqld_exporter-0.11.0.linux-amd64
    cat << EOF > .my.cnf
    [client]
    user=xxx
    password=xxx
    EOF
    #后台运行
    nohup ./mysqld_exporter --config.my-cnf=".my.cnf"  > log.log 2>&1 &
    #防火墙打开9104端口
    firewall-cmd --zone=public --add-port=9104/tcp --permanent  && firewall-cmd --reload
    #查看http://ip:9090/targets其状态是否为up
## 安装grafana

    wget https://dl.grafana.com/oss/release/grafana-5.4.0-1.x86_64.rpm 
    yum localinstall grafana-5.4.0-1.x86_64.rpm
    #安装仪表盘在/etc/grafana/grafana.ini的dashboards中加入
    [dashboards.json]
    enabled = true
    path = /var/lib/grafana/dashboards

> 5.0版本已被provisioningp配置取代，在/etc/grafana/provisioning/dashboards中编辑

    # config file version
    apiVersion: 1

    providers:
    - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10 #how often Grafana will scan for changed dashboards
    options:
        path: /var/lib/grafana/dashboards
    #启动服务
    systemctl start grafana-server
    #开放3000端口
    firewall-cmd --zone=public --add-port=3000/tcp --permanent  && firewall-cmd --reload
    #访问ip:3000使用admin/admin登录并修改密码，然后添加datasource,选择prometheus