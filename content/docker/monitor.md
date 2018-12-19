---
title: "Monitor"
date: 2018-12-07T14:47:40+08:00
draft: true
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
### CONFIGURATION
通过命令行与配置文件配置


## k8s安装
参照https://blog.frognew.com/2017/12/using-prometheus-to-monitor-kubernetes.html
由于使用节点存储增加了pv与pvc
prometheus.rbac.yml
````
#定义了Prometheus容器访问k8s apiserver所需的ServiceAccount和ClusterRole及ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system
````
prometheus.config.yml
````
#configmap中的prometheus的配置文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-system
data:
  prometheus.yml: |
    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    scrape_configs:
    
    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
    
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

    - job_name: 'kubernetes-cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      metrics_path: /probe
      params:
        module: [http_2xx]
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-ingresses'
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
````
prometheus.deploy.yml
````
#pv不区分namespace
kind: PersistentVolume
apiVersion: v1
metadata:
  name: erp-monitor
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
#按照实际情况更改相关存储，创建之后有可能没有相关权限，需要对文件夹增加权限
    path: "/docker/erp-pv/erp-monitor"
---
#pvc区分namespace
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: erp-monitor-claim
  namespace: kube-system
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    name: prometheus-deployment
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - image: prom/prometheus
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/prometheus"
          name: data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      serviceAccountName: prometheus
      imagePullSecrets: 
        - name: regsecret
      volumes:
      - name: data
#按照实际情况更改相关存储
        persistentVolumeClaim: 
          claimName: erp-monitor-claim
      - name: config-volume
        configMap:
          name: prometheus-config
````
prometheus.svc.yml中没有定义nodePort，让k8s自行分配，然后通过kubectl get svc -n kube-system查看节点暴露的端口号
````
#Prometheus的Servic，需要将Prometheus以NodePort, LoadBalancer或使用Ingress暴露到集群外部，这样外部的Prometheus才能访问它
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
  selector:
    app: prometheus
````
将四个文件保存在同一个文件下，之后kubectl create -f .
之后在grafana中添加该数据源。添加时发现localhost:xx端口不能访问，127.0.0.1:端口能访问
之后一些k8s dashboard中仅能查看部分数据，所以继续安装文中提到的kube-state-metrics。[官方yaml文件](https://github.com/kubernetes/kube-state-metrics/tree/master/kubernetes)
由于需要k8s.gcr.io/addon-resizer的镜像，因此需要科学上网
将其文件保存然后kubectl create -f .
通过dashboard或kubectl查看相关pod是否正常
grafana使用[1. Kubernetes Deployment Statefulset Daemonset metrics](https://grafana.com/dashboards/8588)查看