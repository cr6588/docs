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