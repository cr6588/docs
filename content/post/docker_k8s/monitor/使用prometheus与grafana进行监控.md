---
title: "使用prometheus与grafana进行监控"
date: 2018-12-07T14:47:40+08:00
categories: ["Kubernetes"]
---

# 使用prometheus与grafana进行监控
## 安装prometheus
### docker方式
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

> 数据默认是存储在安装目录data文件夹中
> --storage.tsdb.path：这决定了Prometheus写入数据库的位置。默认为data/。
> --storage.tsdb.retention：这决定了何时删除旧数据。默认为15d。

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

    #安装已有的仪表盘在/etc/grafana/grafana.ini的dashboards中加入，可以在安装好后直接去官网下载相关仪表盘
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
    #由于之后的dashboard需要安装饼图插件，所以先安装
    grafana-cli plugins install grafana-piechart-panel
    #点击左侧+号，然后import，填入8919，prometheus一栏选择Prometheus
## 安装[ALERTMANAGER](https://prometheus.io/docs/alerting/overview/)
### 概述
ALERTMANAGER是Prometheus警报管理器与Prometheus。首先Prometheus发送警报规则给ALERTMANAGER,然后ALERTMANAGER管理这些警报，它负责对它们进行重复数据删除，分组和路由，以及正确的接收器集成。最后通过电子邮件，PagerDuty等方式发送通知.
#### [Grouping](https://prometheus.io/docs/alerting/alertmanager/#grouping)
> Grouping categorizes alerts of similar nature into a single notification.
分组将类似性质的警报分类为单个通知.
例如：发生网络分区时，群集中正在运行数十或数百个服务实例。一半的服务实例无法再访问数据库。Prometheus中的警报规则配置为在每个服务实例无法与数据库通信时发送警报。结果，数百个警报被发送到Alertmanager。

作为用户，只能想要获得单个页面，同时仍能够确切地看到哪些服务实例受到影响。因此，可以将Alertmanager配置为按群集和alertname对警报进行分组，以便发送单个紧凑通知。
#### Inhibition
> Inhibition is a concept of suppressing notifications for certain alerts if certain other alerts are already firing.
在某些警报触发时，抑制某些警报
例如：正在触发警报，通知无法访问整个集群。Alertmanager可以配置为在该特定警报触发时将与该集群有关的所有其他警报静音。这可以防止通知数百或数千个与实际问题无关的触发警报。

#### Silences
> Silences are a straightforward way to simply mute alerts for a given time
在给定时间内简单地静音警报的简单方法
#### CONFIGURATION
通过命令行与配置文件配置
### 安装
在[releases](https://github.com/prometheus/alertmanager/releases)中查看版本然后wget下载
本例使用企业微信通知与自定义模版。将alertmanager.yml与alert.tmpl复制到解压后文件下然后运行./alertmanager即可，默认暴露9093端口，通过ip:9093访问


## k8s安装prometheus与kube-state-metrics
参照https://blog.frognew.com/2017/12/using-prometheus-to-monitor-kubernetes.html

复制prometheus文件夹之后kubectl create -f .

之后在grafana中添加该数据源。添加时发现localhost:xx端口不能访问，127.0.0.1:端口能访问
之后一些k8s dashboard中仅能查看部分数据，所以继续安装文中提到的kube-state-metrics。[官方yaml文件](https://github.com/kubernetes/kube-state-metrics/tree/master/kubernetes)
由于需要k8s.gcr.io/addon-resizer的镜像，因此需要科学上网
将其文件保存然后kubectl create -f .
通过dashboard或kubectl查看相关pod是否正常
grafana使用[1. Kubernetes Deployment Statefulset Daemonset metrics](https://grafana.com/dashboards/8588)查看
8588中内存使用统计有问题，直接在prometheus中使用container_memory_working_set_bytes对某个deployment查看发现有三条数据，例如container_memory_working_set_bytes{pod_name=~"^erp-web.*?"}。但提示有一条数据是不含有container_name与image属性的，且不含数据的值大致等于另外2条数据值之和，所以需要排除这条数据。需要加上container_name!=""或者image!=""。
cpu使用率类似。sum (rate (container_cpu_usage_seconds_total{image!="",name=~"^k8s_.*",io_kubernetes_container_name!="POD",pod_name=~"^$Deployment$Statefulset$Daemonset.*$",kubernetes_io_hostname=~"^$Node$"}[1m])) by (pod_name,kubernetes_io_hostname)取过去一分钟以秒为单位消耗cpu时间的平均值。cpu总的使用率对对该数据再进行一次sum即可。