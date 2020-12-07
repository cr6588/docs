---
title: "k8s问题集"
date: 2020-08-14T13:48:25+08:00
draft: false
---

# k8s证书过期

关于证书的官方说明文档https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#automatic-certificate-renewal
其中说明了通过kubeadm安装生成的客户证书只有1年有效期。
通过kubeadm alpha certs check-expiration检查证书是否到期

可以看到etcd的三个证书已经过期，使用kubeadm alpha certs renew all将所有证书续约后，仍无法连接。之后将/etc/kubernetes/pki目录下所有证书备份,/etc/kubernetes下conf文件备份，重新生成所有证书，以及conf文件
kubeadm init phase certs all
kubeadm init phase kubeconfig all
重启相关进程，可以连接到相关容器，但始终提示相关认证失败。在刚才的文档也有说明在1.17之前的版本有bug(之前是1.15)详见https://github.com/kubernetes/kubeadm/issues/1753。其中也提出升级时会自动续约证书，但间隔时间要小于1年。尝试按照https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
将版本升级到1.16，但dns的一个检查始终无法通过，最后卸载kubeadm，安装 1.18版本

# k8s 删除Terminating状态的namespace

在kubectl create -f dashboard.yaml时，由于里面有namespace,且包含所有dashboard内容，在删除时等待了很久发现还在等待。
kubectl get ns之后发现kubernetes-dashboard的状态是Terminating，然后重新尝试删除以及加上--force仍然没有删除。
搜索之后发现在https://github.com/kubernetes/kubernetes/issues/60807 中提到将finalizers中的kubernetes移除掉置为空数组
尝试kubectl edit namespaces/kubernetes-dashboard 并修改保存后，再次kubectl get ns之后发现没有变化，猜测是kubectl edit之后需要执行
一条命令使其生效。后来按照https://medium.com/@craignewtondev/how-to-fix-kubernetes-namespace-deleting-stuck-in-terminating-state-5ed75792647e 的方式解决

    kubectl get namespace kubernetes-dashboard -o json > logging.json
    ···
    "uid": "e9516a8b-764f-11e9-9621-0a9c41ba9af6"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    ...
    ##去除kubernetes
    "spec": {
        "finalizers": [
        ]
    },
    ...
    kubectl replace --raw "/api/v1/namespaces/kubernetes-dashboard/finalize" -f ./logging.json

再次kubectl get ns已被删除
