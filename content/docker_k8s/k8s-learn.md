
#### 1.minikube(主机非虚拟安装，否则直接跳过)

##### 1.1 [安装kubernetctl](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md#client-binaries)
##### 1.3 [安装virtualbox](https://www.cnblogs.com/wangshuyi/p/6927113.html)
##### 1.2 [安装minikube](https://github.com/kubernetes/minikube/releases)

    [root@cr6588 data]# minikube start
    Starting local Kubernetes v1.10.0 cluster...
    Starting VM...
    E1113 10:59:55.111232    2586 start.go:168] Error starting host: Error creating host: Error executing step: Running precreate checks.
    : We support Virtualbox starting with version 5. Your VirtualBox install is "WARNING: The vboxdrv kernel module is not loaded. Either there is no module\n         available for the current kernel (3.10.0-862.el7.x86_64) or it failed to\n         load. Please recompile the kernel module and install it by\n\n           sudo /sbin/vboxconfig\n\n         You will not be able to start VMs until this problem is fixed.\n5.2.22r126460". Please upgrade at https://www.virtualbox.org.

    /sbin/vboxconfig
    yum install gcc  kernel-devel
    cd /usr/src/kernels/
    #mv 3.10.0-862.14.4.el7.x86_64 3.10.0-862.el7.x86_64

    #vboxdrv.sh: Building VirtualBox kernel modules.
    #grep: 不匹配的 ) 或 \)

    #yum remove kernel-devel
    #virtualbox需要本机kernel-devel对应的版本
    yum install "kernel-devel-uname-r == $(uname -r)"

    #没有开启虚拟化
    [root@cr6588 kernels]# minikube start
    Starting local Kubernetes v1.10.0 cluster...
    Starting VM...
    E1113 14:15:48.202789    3254 start.go:168] Error starting host: Error creating host: Error executing step: Running precreate checks.
    : This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory.
#### 2.docker安装（18.06.1.ce-3.el7）
    # 安装依赖包
    yum install -y yum-utils device-mapper-persistent-data lvm2

    # 添加Docker软件包源
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    #关闭测试版本list（只显示稳定版）
    sudo yum-config-manager --enable docker-ce-edge
    sudo yum-config-manager --enable docker-ce-test

    # 更新yum包索引
    yum makecache fast

    #NO.1 直接安装Docker CE （will always install the highest  possible version，可能不符合你的需求）
    yum install docker-ce

    #NO.2 指定版本安装
    yum list docker-ce --showduplicates|sort -r  
    yum install docker-ce-18.06.1.ce-3.el7
    #启动并加入服务
    systemctl start docker && systemctl enable docker
    #设置私有registry
    vi /etc/docker/daemon.json
    {
        "registry-mirrors": ["https://xxx.com"],
        "insecure-registries" : ["ip:5000", "xxxxxx.xx.cn:5000"]
    }
    #重启生效
    systemctl restart docker

#### 3.kubeadm安装（1.12.2）

    #修改主机名称
    hostnamectl set-hostname name
    #关闭swap
    swapoff -a
    vi /etc/fstab
    注释swap

    systemctl stop firewalld
    setenforce 0
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    加载ipvs相关模块以及安装依赖关系
    yum install ipset ipvsadm conntrack-tools.x86_64 -y

    modprobe ip_vs_rr
    modprobe ip_vs_wrr
    modprobe ip_vs_sh
    modprobe ip_vs
>     重启会失效，参考[Linux启动自动加载模块](https://www.jianshu.com/p/69e0430a7d20)在/etc/sysconfig/modules/建立ipvs.modules并写入
>     #!/bin/bash
>     modprobe ip_vs_rr
>     modprobe ip_vs_wrr
>     modprobe ip_vs_sh
>     modprobe ip_vs
>     给ipvs.modules加入执行权限
>     chmod +x ipvs.modules

    查看是否成功
    lsmod| grep ip_vs
    2、开启内核转发，并使之生效，末尾EOF前不要有空格
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF

    sysctl -p /etc/sysctl.d/k8s.conf

    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

    yum install kubelet-1.12.2-0 kubeadm-1.12.2-0 kubectl-1.12.2-0
    #yum install kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet
    # Run kubeadm config images pull prior to kubeadm init to verify connectivity to gcr.io registries.验证是否能拉取相关镜像，若不能则找到相关镜像库镜像拉取
    
    kubeadm config images pull
    #从私有库拉取相关镜像
    docker login www.xxxx.cn:5000
    #输入相用户密码
    docker pull www.xxxx.cn:5000/k8s.gcr.io/kube-controller-manager:v1.12.2
    docker pull www.xxxx.cn:5000/k8s.gcr.io/kube-apiserver:v1.12.2
    docker pull www.xxxx.cn:5000/k8s.gcr.io/kube-scheduler:v1.12.2
    docker pull www.xxxx.cn:5000/k8s.gcr.io/etcd:3.2.24
    docker pull www.xxxx.cn:5000/k8s.gcr.io/coredns:1.2.2
    docker pull www.xxxx.cn:5000/k8s.gcr.io/pause:3.1
    docker pull www.xxxx.cn:5000/k8s.gcr.io/kube-proxy:v1.12.2

    docker tag www.xxxx.cn:5000/k8s.gcr.io/kube-controller-manager:v1.12.2 k8s.gcr.io/kube-controller-manager:v1.12.2
    docker tag www.xxxx.cn:5000/k8s.gcr.io/kube-apiserver:v1.12.2 k8s.gcr.io/kube-apiserver:v1.12.2 
    docker tag www.xxxx.cn:5000/k8s.gcr.io/kube-scheduler:v1.12.2 k8s.gcr.io/kube-scheduler:v1.12.2
    docker tag www.xxxx.cn:5000/k8s.gcr.io/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24
    docker tag www.xxxx.cn:5000/k8s.gcr.io/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2
    docker tag www.xxxx.cn:5000/k8s.gcr.io/pause:3.1 k8s.gcr.io/pause:3.1
    docker tag www.xxxx.cn:5000/k8s.gcr.io/kube-proxy:v1.12.2 k8s.gcr.io/kube-proxy:v1.12.2
    #确认相关镜像在主从上都存在，否则有可能后面虽然加入节点成功，但一直是notready状态
    #master节点执行，network使用flannel前置条件.若需要制定版本则加入--kubernetes-version=v1.12.2
    kubeadm init --pod-network-cidr=10.244.0.0/16
    #master节点执行，network使用calico前置条件.若需要制定版本则加入
    kubeadm init --pod-network-cidr=192.168.0.0/16
    #master节点执行，network使用weave net前置条件
    kubeadm init
    #出错时还原
    kubeadm reset
    #记录下从节点加入命令
    kubeadm join xxx:6443 --token gd7nxxxxxx --discovery-token-ca-cert-hash sha256:xxxx
    #To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    # Alternatively, if you are the root user, you can run:
    export KUBECONFIG=/etc/kubernetes/admin.conf

    #使用flannel（不支持NetworkPolicy,部署的应用会无视firewalld完全暴露在公网中）
    sysctl net.bridge.bridge-nf-call-iptables=1
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
    #使用calico
    kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
    #使用weave net
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    #查看状态
    kubectl get pods --all-namespaces

>     ...failed to set bridge addr: "cni0" already has an IP address different from 10.244.1.1/24
>     删除cni0网卡（ip link delete cni0）可以解决，但为了预防以后莫名错误，完全重置一次，参见https://github.com/kubernetes/kubernetes/issues/39557
>     kubeadm reset
>     systemctl stop kubelet
>     systemctl stop docker
>     rm -rf /var/lib/cni/
>     rm -rf /var/lib/kubelet/*
>     rm -rf /etc/cni/
>     ifconfig cni0 down
>     ifconfig flannel.1 down
>     ifconfig docker0 down
>     ip link delete cni0
>     ip link delete flannel.1
>     ifconfig docker0 up
>     systemctl start docker
>     systemctl start kubelet

>     ...Readiness probe failed: calico/node is not ready: BIRD is not ready: BGP not established with 10.192.0.1
>     calico健康检查出错，calico的ip自动检测给了一个错误IP10.192.0.1，在将https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml下载下来，在calico-node的env加入IP_AUTODETECTION_METHOD,其eth.*视其节点间通信的网卡决定

````
...
            - name: IP
              value: "autodetect"
            - name: IP_AUTODETECTION_METHOD
              value: "interface=eth.*"
...
````


    #主节点向指定ip开放相关端口
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="6443" accept"
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="2379-2380" accept"
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="10250-10252" accept"
    #从节点向指定ip开放相关端口
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="10250" accept"
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="30000-32767" accept"
    #默认主节点不会编排容器，若希望启用则运行
    kubectl taint nodes --all node-role.kubernetes.io/master-
    #从节点加入主节点
    #修改节点名称
    hostnamectl --static set-hostname [主机名]
    #在node的hosts文件加入解析
    127.0.0.1 localhost [主机名]
    #使用刚刚的kubeadm join
    kubeadm join xxx:6443 --token gd7nxxxxxx --discovery-token-ca-cert-hash sha256:xxxx
    #token有效期24小时，过期之后用创建token，查找hash值即可
    kubeadm token create
    kubeadm token list
    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
    #主节点查看节点信息
    kubectl get nodes


> init之后如果关机重启kubelet正常的话，root用户若使用kubelctl提示无法连接则
export KUBECONFIG=/etc/kubernetes/admin.conf

#### 4.dashboard安装
    #由于执行过一次最简安装后无法访问本地，查看kubectl get pods --all-namespaces
    kube-system   kube-scheduler-localhost.localdomain            1/1     Running             0          91m
    kube-system   kubernetes-dashboard-77fd78f978-4vzk2           0/1     ContainerCreating   0          13m
    一直处于ContainerCreating中，于是删除,又通过自检证书方式重建
    kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    #创建自已的证书
    openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout certs/dashboard.key \
    -x509 -days 365 -out certs/dashboard.crt

    kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    #查看服务，发现服务运行但无法访问
    kubectl get service kubernetes-dashboard -n kube-system
    #删除
    kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    #下载文件
    curl -O https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    #在kubernetes-dashboard.yaml中添加service的type为NodePort
    ...
      namespace: kube-system
    spec:
        type: NodePort
        ports:
            - port: 443
    ...
    #重新生成密钥并部署
    kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system
    kubectl apply -f kubernetes-dashboard.yaml
    #查看服务映射的本地端口
    kubectl get service kubernetes-dashboard -n kube-system
    NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
    kubernetes-dashboard   NodePort   10.109.126.132   <none>        443:31474/TCP   20m
    #访问子节点https://ip:31474, 需要登录。若没有登录界面，直接进入，访问https://192.168.199.206:31474/#!/login
    ![x](/images/k8s_login.png)
    #创建用户
    cat <<EOF > dashboard-adminuser.yaml

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system
    EOF

    kubectl apply -f dashboard-adminuser.yaml
    #查找，复制admin-user的token，登录即可
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    
    #查看日志
    kubectl describe pod kubernetes-dashboard-77fd78f978-4vzk2无法查看描述必须加上--namespace=kube-system
    kubectl describe pod kubernetes-dashboard-77fd78f978-4vzk2 --namespace=kube-system
    kubectl logs kubernetes-dashboard-77fd78f978-4vzk2 -n kube-system

    ...failed to set bridge addr: "cni0" already has an IP address different from 10.244.1.1/24

    kubectl get pods --all-namespaces
    kubectl get service kubernetes-dashboard -n kube-system

> 使用flannel启用防火墙后dashboard可能无法安装，需要开启
    firewall-cmd --permanent --add-masquerade # 允许防火墙伪装
并开启相关端口，参照[install-kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)开启
> 使用calico必须启用防火墙，需要开启
    firewall-cmd --permanent --add-masquerade # 允许防火墙伪装
并开启相关端口，参照[System requirements](https://docs.projectcalico.org/v3.3/getting-started/kubernetes/requirements)开启
5473需要开启否则应用内部无法通过服务名:端口访问
> 使用weave-net必须启用防火墙，需要开启
    firewall-cmd --permanent --add-masquerade # 允许防火墙伪装
并开启相关端口，参照[FAQ](https://www.weave.works/docs/net/latest/faq/)开启
TCP 6783-6784 and UDP 6783-6784需要开启否则应用内部通过服务名:端口访问会时快时慢

#### redis安装

    参见docker hub中[redis](https://hub.docker.com/_/redis/)官方镜像文档说明，直接在dashboard上进行图形化安装

> 安装后查看服务的yaml文档中的端口发现
        "protocol": "TCP",
        "port": 6388,
        "targetPort": 6379,
        "nodePort": 31783
其中port是集群内部的端口，31783是节点以及外部可以访问的端口，6388是pod的端口，也就是若外网链接访问时首先经过31783进入然后经过6388最后指向6379。使用kubectl expose deployment/redis --type="NodePort" --port 6379创建service时，port与targetPort都会是6379

图形化安装之后删除,创建redis持久化安装


    #创建pv与pvc
    vi redis-pv.yaml
    
````
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
````

    kubectl create -f redis-pv.yaml
    #会在从节点生成/data/redis-data目录

    #创建deployment
	{
	  "kind": "Deployment",
	  "apiVersion": "extensions/v1beta1",
	  "metadata": {
		"name": "redis-persistent",
		"namespace": "default",
		"labels": {
		  "k8s-app": "redis-persistent"
		}
	  },
	  "spec": {
		"replicas": 1,
		"selector": {
		  "matchLabels": {
			"k8s-app": "redis-persistent"
		  }
		},
		"template": {
		  "metadata": {
			"name": "redis-persistent",
			"labels": {
			  "k8s-app": "redis-persistent"
			}
		  },
		  "spec": {
			"containers": [
			  {
				"name": "redis-persistent",
				"image": "redis:5",
				"command": [
				  "redis-server"
				],
				"args": [
				  "--appendonly yes"
				],
				"volumeMounts": [
				  {
					"name": "redis-persistent-storage",
					"mountPath": "/data"
				  }
				]
			  }
			],
			"volumes": [
			  {
				"name": "redis-persistent-storage",
				"persistentVolumeClaim": {
				  "claimName": "redis-pv-claim"
				}
			  }
			]
		  }
		}
	  }
	}
    #当要使用自己的redis.conf时将自己的conf文件复制到从节点/data/redis-data下，args改为/data/xxx.conf即可

> 挂载磁盘时注意路径，不要将磁盘挂载到镜像本身就有的路径，例如/data中本身含有应用相关文件，然后又挂载到/data时，会覆盖镜像已有的data

#### 部署自己的应用

在创建好自己的镜像并上传上去后，通过dashboard安装时发现镜像始终无法拉取。但是通过docker直接拉取私有镜像时可以的。搜寻一番之后发现是k8s拉取私有镜像时需要docker相关账号信息。
##### 配置secret拉取私有仓库镜像
  kubectl create secret docker-registry registrysecret --docker-server=ip或域名:端口号  --docker-username=admin --docker-password=xxxx
有2种方式加载secret
###### 1.将密钥加载到yaml文件

	apiVersion: v1
	kind: ReplicationController
	metadata:
	  name: webapp
	spec:
	  replicas: 2
	  template:
		metadata:
		  name: webapp
		  labels:
			app: webapp
		spec:
		  containers:
		  - name: webapp
			imagePullPolicy: Always
			image: e5:8889/tomcat:latest
			ports:
			  - containerPort: 80
		  imagePullSecrets:
		  - name: registrysecret
###### 2.将secret直接加载到默认帐号中

    kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "registrysecret"}]}'
    #查看帐号配置
    kubectl get serviceaccounts default -o yaml
    #在dashboard更改下版本或者删除重建部署即可

##### 启动应用后pod中容器不能通过域名访问外网

  #进入容器查看
  kubectl get pods
  kubectl kubectl exec -it pod名称 -- /bin/bash
  #退出
  crtl+p && crtl+q

> 如果Pod具有多个Container，请使用--container或-c在kubectl exec命令中指定Container 。例如，假设您有一个名为my-pod的Pod，而Pod有两个名为main-app和helper-app的容器。以下命令将打开主应用程序Container的shell。

    #交互模式进入容器
    kubectl exec -it my-pod --container main-app -- /bin/bash

网上搜索一番发现在https://github.com/kubernetes/kubernetes/issues/57096#issuecomment-351029125中看到是防火墙未开启udp端口的问题，但具体是哪一个端口不能确定，经过同事之前更改dns服务器之后就能在容器中ping通，应该是dns问题，在容器中更改/etc/resolv.conf之后发现可以ping通，但不能每次容器都要去更改吧，也不应该这样做，所以又寻找其它方法，又搜索了很久发现很多人都遇到过这个问题，一部分是通过sysctl net.bridge.bridge-nf-call-iptables=1可以恢复，但测试之后未能成功。在搜索过程看到一些处理这个问题的过程是弄清ping之后请求的走向，通过查看防火墙日志确定在哪一过程被拦截，但这方面积累比较薄弱，且dns的容器无法进入，于是又继续查找。之后把防火墙关了重启服务器发现可以ping通,kubectl exec my-pod ping www.baidu.com,认为是防火墙端口的问题，而dns是coredns是通过安装flanel带来的，最后去flanel的[Troubleshooting](https://github.com/coreos/flannel/blob/master/Documentation/troubleshooting.md#firewalls)的中firewalls指出

    When using udp backend, flannel uses UDP port 8285 for sending encapsulated packets.
    When using vxlan backend, kernel uses UDP port 8472 for sending encapsulated packets.
但不能确定是哪一个就一个个测试，最终发现在节点启用8472/udp后，在master执行

    systemctl daemon-reload
    systemctl restart kubelet
master再kubectl exec my-pod ping www.baidu.com就能ping通，但速度较慢，在master也启用8472/udp后ping的速度明显加快

> 回顾整个过程可以发现在最初是找对方向的，但之后又跑偏了，说明对k8s的网络管理方面理解的很薄弱，然后对centos7如何监控请求走向还是不会，待提高的地方很多。

##### 容器需要使用host
在只有docker时可以通过docker run时加入--add-host="localhost example.com":127.0.0.1，而在k8s时可以在pod的[spec.hostAliases](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)实现。eg:

    ...
        spec:
      hostAliases:
      - ip: "10.101.4.13"
        hostnames:
        - "xxx.xx.xx"
      - ip: "xxx.xx.xx.xxx"
        hostnames:
        - "user.xx.xxx.cn"
      - ip: "118.xxx.xx.xxx"
        hostnames:
        - "picture.xxx.cn"
        - "static.xxx.cn"
      containers:
      - name: ...
      ...
##### statefulset应用
有状态应用于headless service搭配，它的主要特点是ClusterIP为None,以一个zookeeper应用为例
````
kind: PersistentVolume
apiVersion: v1
metadata:
  name: zookeeper-pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/docker/pv/zookeeper"
---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper
  labels:
    app: zookeeper
spec:
  ports:
  - port: 2181
    name: zookeeper
  clusterIP: None
  selector:
    app: zookeeper
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zookeeper
spec:
  serviceName: "zookeeper"
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      containers:
      - name: zookeeper
        image: zookeeper
        ports:
        - containerPort: 2181
          name: zookeeper
        volumeMounts:
        - name: zk
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: zk
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
````
由于未配置动态存储，先设置一个hostPath类型的pv，然后设置headless service,最后设置StatefulSet，其中volumeClaimTemplates中描述了pvc，会查询符合要求的pv后自动创建pvc,在StatefulSet中pod与pvc会绑定，当伸缩数量为2时，需要新建pv满足


#### 参考文献

1. https://kubernetes.io/docs/setup/independent/install-kubeadm/
2. https://www.cnblogs.com/crysmile/p/9648406.html
3. http://blog.51cto.com/11887934/2050590
4. https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/
