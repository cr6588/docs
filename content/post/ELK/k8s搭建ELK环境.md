---
title: "k8s搭建ELK环境"
date: 2018-12-17T09:20:51+08:00
categories: ["ELK"]
---

# elasticsearch部署（v6.5.3）

elasticsearch.yml
````
#集群名，且不配置节点名称，避免多个k8s多个pod时，配置文件不同
 cluster.name: es
#host设置成0.0.0.0时让任何ip都能访问，但其属于k8s内部网络，所以对k8s做好安全配置即可
 network.host: 0.0.0.0
 discovery.zen.ping.unicast.hosts: ["es-zen:9300"]
 discovery.zen.minimum_master_nodes: 1
 action.destructive_requires_name: true
````
esDockerfile
由于elasticsearch不能以root用户启动，所以需要创建用户以及授权
````
FROM jdk8-centos7
COPY elasticsearch-6.5.3 /elasticsearch-6.5.3
RUN groupadd es;\
	useradd es -g es -p elasticsearch;\
	chown -R es /elasticsearch-6.5.3;\
	chgrp -R es /elasticsearch-6.5.3
EXPOSE 9200 9300
USER es
CMD ["./elasticsearch-6.5.3/bin/elasticsearch"]
````
es-deployment.yml
本想采用configmap配置，然后当作volume挂载，但es6.5.3中配置文件增多了（user,role相关），只挂载elasticsearch.yml时会错，且只是当一个日志查看，所以使用hostPath方式将配置文件放入其指定的目录中，这样可以修改配置，也不太繁琐，注意文件权限
````
#pv不区分namespace
kind: PersistentVolume
apiVersion: v1
metadata:
  name: es-data
  labels:
    type: local
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
#按照实际情况更改相关存储，创建之后有可能没有相关权限，需要对文件夹增加权限
    path: "/docker/es/data"
---
#pvc区分namespace
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: es-data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: es-config
  labels:
    type: local
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
#按照实际情况更改相关存储，创建之后有可能没有相关权限，需要对文件夹增加权限
    path: "/docker/es/config"
---
#pvc区分namespace
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: es-config-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    name: es-deployment
  name: es
spec:
  replicas: 1
  selector:
    matchLabels:
      app: es
  template:
    metadata:
      labels:
        app: es
    spec:
      containers:
      - image: xxxxip:5000/es:6.5.3
        name: es
#        command:
#        - "./elasticsearch-6.5.3/bin/elasticsearch"
#        args:
#        - "--config.file=/etc/prometheus/prometheus.yml"
#        - "--storage.tsdb.path=/prometheus"
#        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9200
          protocol: TCP
        - containerPort: 9300
          protocol: TCP
        volumeMounts:
        - mountPath: "/elasticsearch-6.5.3/data"
          name: data
        - mountPath: "/elasticsearch-6.5.3/config"
          name: config-volume
#        resources:
#          requests:
#            cpu: 100m
#            memory: 100Mi
#          limits:
#            cpu: 500m
#            memory: 2500Mi
#      serviceAccountName: prometheus
#      imagePullSecrets: 
#        - name: regsecret
      volumes:
      - name: data
#按照实际情况更改相关存储
        persistentVolumeClaim: 
          claimName: es-data-claim
      - name: config-volume
        persistentVolumeClaim: 
          claimName: es-config-claim
#        configMap:
#          name: es-config
````
service.yml
````
#filebeat http数据端口,用节点ip加之后生成的随机端口访问
kind: Service
apiVersion: v1
metadata:
  labels:
    app: es
  name: es
spec:
  type: NodePort
  ports:
  - port: 9200
    targetPort: 9200
  selector:
    app: es
---
#zen集群发现端口
kind: Service
apiVersion: v1
metadata:
  labels:
    app: es
  name: es-zen
spec:
  ports:
  - port: 9300
    targetPort: 9300
  selector:
    app: es
````
# kibana部署（v6.5.3）

配置文件kibana.yml
````
server.host: "0.0.0.0"
#跟之前es的services名称相对应
elasticsearch.url: "http://es:9200"
````
kibanaDockerfile
````
FROM jdk8-centos7
COPY kibana-6.5.3-linux-x86_64 /kibana-6.5.3-linux-x86_64
EXPOSE 5601
CMD ["./kibana-6.5.3-linux-x86_64/bin/kibana"]
````
kibana-deployment.yml
````
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: kibana
  namespace: default
  labels:
    k8s-app: kibana
spec:
  replicas: 1
  selector:
    matchLabels: 
      k8s-app: kibana
  template:
    metadata:
      name: kibana
      labels:
        k8s-app: kibana
    spec:
      containers:
      - name: kibana
###自己私有镜像库的ip
        image: xxxx/kibana:v6.5.3
---
# ------------------- kibana Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kibana
  name: kibana
  namespace: default
spec:
  type: NodePort
  ports:
    - port: 5601
      targetPort: 5601
  selector:
    k8s-app: kibana
````
# filebeat部署（v6.5.3）
配置文件filebeat.yaml
更改相关ip,并搜集/docker/erp-pv/erp-logs/下所有文件中，以时间开头的日志，并将其后到下一个时间开头之间的日志合并为一行进行搜集
````
###################### Filebeat Configuration Example #########################

# This file is an example configuration file highlighting only the most common
# options. The filebeat.reference.yml file from the same directory contains all the
# supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/filebeat/index.html

# For more available modules and options, please see the filebeat.reference.yml sample
# configuration file.

#=========================== Filebeat inputs =============================

filebeat.inputs:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /docker/erp-pv/erp-logs/*
    - /docker/erp-pv/erp-logs/*/*
    #- /docker/dockerbuild/apache-tomcat-8.0.52/logs/logistics/*.log
    #- c:\programdata\elasticsearch\logs\*
  #exclude_files: ["^erp"]
  #0或1个[(2018-07-19或者11-Jul-2018)
  multiline.pattern: '^\[?([0-9]{4}-[0-9]{2}-[0-9]{2}|[0-9]{2}-([a-z]|[A-Z]){3}-[0-9]{4})'
  multiline.negate: true
  multiline.match: after
    #- c:\programdata\elasticsearch\logs\*

  # Exclude lines. A list of regular expressions to match. It drops the lines that are
  # matching any regular expression from the list.
  #exclude_lines: ['^DBG']

  # Include lines. A list of regular expressions to match. It exports the lines that are
  # matching any regular expression from the list.
  #include_lines: ['^ERR', '^WARN']

  # Exclude files. A list of regular expressions to match. Filebeat drops the files that
  # are matching any regular expression from the list. By default, no files are dropped.
  #exclude_files: ['.gz$']

  # Optional additional fields. These fields can be freely picked
  # to add additional information to the crawled log files for filtering
  #fields:
  #  level: debug
  #  review: 1

  ### Multiline options

  # Multiline can be used for log messages spanning multiple lines. This is common
  # for Java Stack Traces or C-Line Continuation

  # The regexp Pattern that has to be matched. The example pattern matches all lines starting with [
  #multiline.pattern: ^\[

  # Defines if the pattern set under pattern should be negated or not. Default is false.
  #multiline.negate: false

  # Match can be set to "after" or "before". It is used to define if lines should be append to a pattern
  # that was (not) matched before or after or as long as a pattern is not matched based on negate.
  # Note: After is the equivalent to previous and before is the equivalent to to next in Logstash
  #multiline.match: after


#============================= Filebeat modules ===============================

filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml

  # Set to true to enable config reloading
  reload.enabled: false

  # Period on which files under path should be checked for changes
  #reload.period: 10s

#==================== Elasticsearch template setting ==========================

setup.template.settings:
  index.number_of_shards: 3
  #index.codec: best_compression
  #_source.enabled: false

#================================ General =====================================

# The name of the shipper that publishes the network data. It can be used to group
# all the transactions sent by a single shipper in the web interface.
#name:

# The tags of the shipper are included in their own field with each
# transaction published.
#tags: ["service-X", "web-tier"]

# Optional fields that you can specify to add additional information to the
# output.
#fields:
#  env: staging


#============================== Dashboards =====================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards is disabled by default and can be enabled either by setting the
# options here, or by using the `-setup` CLI flag or the `setup` command.
#setup.dashboards.enabled: false

# The URL from where to download the dashboards archive. By default this URL
# has a value which is computed based on the Beat name and version. For released
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

#============================== Kibana =====================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "http://xxxxip:30137"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

#============================= Elastic Cloud ==================================

# These settings simplify using filebeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:

# The cloud.auth setting overwrites the `output.elasticsearch.username` and
# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
#cloud.auth:

#================================ Outputs =====================================

# Configure what output to use when sending the data collected by the beat.

#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["xxxxip:30200"]

  # Optional protocol and basic auth credentials.
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"

#----------------------------- Logstash output --------------------------------
#output.logstash:
  # The Logstash hosts
  #hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

#================================ Procesors =====================================

# Configure processors to enhance or manipulate events generated by the beat.

processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~

#================================ Logging =====================================

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
#logging.level: debug

# At debug level, you can selectively enable logging only for some components.
# To enable all selectors use ["*"]. Examples of other selectors are "beat",
# "publish", "service".
#logging.selectors: ["*"]

#============================== Xpack Monitoring ===============================
# filebeat can export internal metrics to a central Elasticsearch monitoring
# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
# reporting is disabled by default.

# Set to true to enable the monitoring reporter.
#xpack.monitoring.enabled: false

# Uncomment to send the metrics to Elasticsearch. Most settings from the
# Elasticsearch output are accepted here as well. Any setting that is not set is
# automatically inherited from the Elasticsearch output configuration, so if you
# have the Elasticsearch output configured, you can simply uncomment the
# following line.
#xpack.monitoring.elasticsearch:
logging.level: info
logging.to_files: true
logging.to_syslog: false
logging.files:
  path: logs
  name: filebeat.log
  keepfiles: 7
````
设置filebeat启动脚本。在访问kibana后按照kibana的提示进行设置
start.sh
````
#!/bin/bash
cd /filebeat
./filebeat modules enable elasticsearch
./filebeat setup
./filebeat -e
````
filebeatDockerfile
````
FROM jdk8-centos7
COPY filebeat-6.5.3-linux-x86_64 /filebeat
CMD ["./filebeat/start.sh"]
````
filebeat-deployment.yml
````
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: filebeat
  namespace: default
  labels:
    k8s-app: filebeat
spec:
  replicas: 1
  selector:
    matchLabels: 
      k8s-app: filebeat
  template:
    metadata:
      name: filebeat
      labels:
        k8s-app: filebeat
    spec:
      containers:
      - name: filebeat
        image: filebeat:6.5.3
        volumeMounts:
        - name: filebeat-storage
          mountPath: /docker/erp-pv/
      volumes:
      - name: filebeat-storage
#替换自己的存储
        persistentVolumeClaim: 
          claimName: erp-pv-claim
````