
#### 1.minikube

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

#### 3.kubeadm安装（1.12.2）

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

    yum install -y kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet
    # Run kubeadm config images pull prior to kubeadm init to verify connectivity to gcr.io registries.验证是否能拉取相关镜像，若不能则找到相关镜像库镜像拉取
    
    kubeadm config images pull
    #从私有库拉取相关镜像
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
    
    #master节点执行，network使用flannel前置条件
    kubeadm init --pod-network-cidr=10.244.0.0/16
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

    #使用flannel
    sysctl net.bridge.bridge-nf-call-iptables=1
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
    #查看flannel状态
    kubectl get pods --all-namespaces
    #从节点向指定ip开放相关端口
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="10250" accept"
    firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="www.xxxx.cn" port protocol="tcp" port="30000-32767" accept"
    #从节点加入主节点使用刚刚的kubeadm join
    kubeadm join xxx:6443 --token gd7nxxxxxx --discovery-token-ca-cert-hash sha256:xxxx
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
    vi dashboard-adminuser.yaml
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

    kubectl apply -f dashboard-adminuser.yaml
    #复制token
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    #找出admin-user的token，登录即可
    
    #查看日志
    kubectl describe pod kubernetes-dashboard-77fd78f978-4vzk2无法查看描述必须加上--namespace=kube-system
    kubectl describe pod kubernetes-dashboard-77fd78f978-4vzk2 --namespace=kube-system

    ...failed to set bridge addr: "cni0" already has an IP address different from 10.244.1.1/24

    kubectl get pods --all-namespaces
    kubectl get service kubernetes-dashboard -n kube-system



#### 参考文献

1. https://kubernetes.io/docs/setup/independent/install-kubeadm/
2. https://www.cnblogs.com/crysmile/p/9648406.html
3. http://blog.51cto.com/11887934/2050590
