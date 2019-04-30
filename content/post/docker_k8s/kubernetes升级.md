---
title: "Kubernetes升级"
date: 2019-04-04T14:06:56+08:00
draft: false
---

#### Kubernetes v1.12 to v1.13
参照官方[Upgrading kubeadm clusters from v1.12 to v1.13](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-13/)说明升级

##### master节点
yum list --showduplicates kubeadm --disableexcludes=kubernetes
yum install -y kubeadm-1.13.x-0 --disableexcludes=kubernetes
kubeadm upgrade plan

由于升级需要k8s.gcr.io镜像，但无法连接外网，所以使用微软的镜像站替代，详见[GCR Proxy Cache 帮助](http://mirror.azure.cn/help/gcr-proxy-cache.html)。
这里有2中方式可以替代
1.利用kubeadm config images list得到需要的镜像，然后像docker pull gcr.azk8s.cn/google_containers/kube-apiserver:v1.13.0这样挨个把镜像下载下来
最后docker tag gcr.azk8s.cn/google_containers/kube-apiserver:v1.13.0 k8s.gcr.io/kube-apiserver:v1.13.0更改名称即可
2.更改镜像的默认依赖库
通过kubeadm config view可以看到
···
imageRepository: k8s.gcr.io
···
所以只需要把imageRepository改成gcr.azk8s.cn/google_containers即可

通过kubeadm config -h的帮助最后发现通过kubeadm config upload from-file --config kubeadm.yaml可以实现
首先创建已有的kubeadm.yaml文件，将kubeadm config view所有信息复制到kubeadm.yaml中然后将
imageRepository: k8s.gcr.io ->imageRepository: gcr.azk8s.cn/google_containers
最后kubeadm config upload from-file --config kubeadm.yaml即可

kubeadm upgrade apply v1.13.0
yum install -y kubelet-1.13.x-0 --disableexcludes=kubernetes
yum install -y kubectl-1.13.x-0 --disableexcludes=kubernetes
kubectl drain 节点名称 --ignore-daemonsets

##### node节点
yum install -y kubectl-1.13.x-0 --disableexcludes=kubernetes
kubeadm upgrade node config --kubelet-version v1.13.x
yum install -y kubelet-1.13.x-0 kubeadm-1.13.x-0 --disableexcludes=kubernetes

##### 重启所有节点
systemctl restart kubelet
kubectl uncordon $NODE
kubectl get nodes


#### Kubernetes v1.13 to v1.14 升级问题
参照官方[Upgrading kubeadm clusters from v1.13 to v1.14](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-14/)说明升级。
需要注意的是在升级yum install -y kubeadm-1.14.x-0 --disableexcludes=kubernetes时，其依赖的kubernetes-cni会升级，而kubernetes-cni又依赖kubelet所以，在这一步kubelet就会跟着升级了。然后kubeadm upgrade plan相关提示会跟官方不一样，不过继续照着做就行，暂未发现有何影响。
使用weavenet的一个节点正常升级，但使用flannel的节点无法升级到1.14.0提示192.168.0.206:2379无法访问，经查发现在ectd配置中只允许127.0.0.1访问
![x](/images/ectd1.png)
，而另一台已经升级好的weavenet节点发现是
![x](/images/ectd2.png)
但是更改ip时发现会报错，暂时未升级