---
title: "Desc"
date: 2018-09-18T10:20:11+08:00
draft: true
---

### Docker
#### Docker是什么？
是一个应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上。它会涉及到以下部分
* Docker Client客户端
* Docker Daemon守护进程
* Docker Image镜像
* DockerContainer容器

##### Docker Client客户端
即从官网下载的docker软件客户端
##### Docker Daemon守护进程
下载的docker软件中含有服务端，启动服务端之后成为Docker Daemon守护进程。通过docker相关命令（即客户端）访问这个守护进程可以进行一系列操作。默认是不能远程访问的。
##### Docker Image镜像
一个特殊的文件系统，提供应用运行时所需的程序、库、资源、配置等文件，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 镜像不包含任何动态数据，其内容在构建之后也不会被改变。镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

> 那么对于我们目前的应用应该需要哪些镜像？

##### Docker Container容器
镜像的实例化。在镜像创建好了之后，就会创建容器来使用镜像。就像是类和实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器的实质是进程，但与直接在宿主执行的进程不同，它是通过基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等操作系统层面的虚拟化技术进行隔离的。但又与传统虚拟机vmware或virtualbox不同，后者需要虚拟出一套硬件并运行一个完整的操作系统，且虚拟机中的程序运行内核依赖虚拟机中的操作系统，而前者使用的是宿主机的内核。

> 同样对于我们目前的应用应该会有哪些容器？

#### 作用

* 环境一致性：由于将应用的所有依赖都已打包，所以杜绝了“我的机器行，你的机器运行不了的情况”
* 批量部署与迁移更加方便：将镜像制作好后，不必再去像以前一样处理应用所需的依赖，节省运维开发时间。
* 维护和扩展更轻松：由于镜像的分层可以很好的切换历史应用的版本，同时也可以利用基础镜像进行扩展

#### 缺点
对操作系统有要求，入门以及使用都有一定门槛, 尤其是镜像的制作，容器的管理与数据的持久化等。在宿主机需要运行守护进程，容器运行都要基于其守护进程，相比直接运行程序多了一层。

##### 与未用docker对比
未用docker时，按部就班的安装相关应用依赖环境，多台时每次要安装一次环境。改变环境后每台都需要详细配置
用了docker后，每台需要安装docker环境，然后着重在一台维护应用需要的环境镜像，其它机器在更新了镜像之后拉取镜像即可，方便维护和部署多台环境

### kubernetes
#### kubernetes是什么？
 > Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications
 > Kubernetes是一个用于容器化应用程序的自动化部署，扩展和管理的开源系统
 
谷歌开源的容器集群管理系统，是 Google 多年大规模容器管理技术 Borg 的开源版本。
主要元件有：
* Master – 大总管，可做为主节点
* Node – 主要工作的节点，上面运行了许多容器。可想作一台虚拟机。K8S可操控高达1,000个nodes以上
* masters和nodes组成集群(Clusters)
更具体来看
![x](https://blog.gcp.expert/material/2017/02/k8s_arch-1024x437.png)

现在着重K8S最重要的三个部分，即
* Pod
* Service
* Deployments (Replication Controller)

##### Pod
容器是位于pod内部，一个pod包含一个以上的容器，这造成K8S与一般容器不同的操作概念。在Docker里，Docker container是最小单位，但在K8S可想作pod为最小单位。从以下pod的特性来看，就可以了解为什么它是K8S裡面三巨头之一了。

1. Pod 拥有不确定的生命週期，这意味著您不晓得任一pod是否会永久保留
2. Pod 内有一个让所有container共用的Volume，这会与Docker不同
3. Pod 採取shared IP，内部所有的容器皆使用同一个Pod IP，这也与Docker不同
4. Pod 内的众多容器都会和Pod同生共死，就像桃园三结义一样！
 > 对于我们的应用，可以怎样规划Pod?
##### Service
K8S的 Service 有它的独特方法，我们看看它的特性
1. 每个Service包含著一个以上的pod
2. 每个Service有个独立且固定的IP地址 – Cluster IP
3. 客户端访问Service时，会经由上述提过的proxy来达到负载平衡、与各pod连结的结果
4. 利用标籤选择器(Label Selector)，聪明地选择那些已贴上标籤的pod
##### Deployments
旧版的K8S使用了副本控制器(Replication Controller)的名词，在新版已经改成 Deployments萝。Deployments顾名思义掌控了部署Kubernetes服务的一切。它主要掌管了Replica Set的个数，而Replica Set的组成就是一个以上的Pod。

1. Deployments 的设定档，可以指定replica，并保证在该replica的数量运作
2. Deployments 会检查pod的状态
3. Deployments 下可执行滚动更新或者回滚

#### 作用
##### 自动包装
根据资源需求和其他约束自动放置容器，同时不会牺牲可用性，混合关键和最大努力的工作负载，以提高资源利用率并节省更多资源。

##### 自我修复
重新启动失败的容器，在节点不可用时，替换和重新编排节点上的容器，终止不对用户定义的健康检查做出响应的容器，并且不会在客户端准备投放之前将其通告给客户端。

##### 横向缩放
使用简单的命令或 UI，或者根据 CPU 的使用情况自动调整应用程序副本数。

##### 服务发现和负载均衡
不需要修改您的应用程序来使用不熟悉的服务发现机制，Kubernetes 为容器提供了自己的 IP 地址和一组容器的单个 DNS 名称，并可以在它们之间进行负载均衡。

##### 自动部署和回滚
Kubernetes 逐渐部署对应用程序或其配置的更改，同时监视应用程序运行状况，以确保它不会同时终止所有实例。 如果出现问题，Kubernetes会为您恢复更改，利用日益增长的部署解决方案的生态系统。

##### 密钥和配置管理
部署和更新密钥和应用程序配置，不会重新编译您的镜像，不会在堆栈配置中暴露密钥(secrets)。

##### 存储编排
自动安装您所选择的存储系统，无论是本地存储，如公有云提供商 GCP 或 AWS, 还是网络存储系统 NFS, iSCSI, Gluster, Ceph, Cinder, 或 Flocker。

##### 批处理
除了服务之外，Kubernetes还可以管理您的批处理和 CI 工作负载，如果需要，替换出现故障的容器。

#### 参考文献

1. https://baike.baidu.com/item/Docker
2. https://my.oschina.net/choerodon/blog/1936197
3. https://juejin.im/post/5b260ec26fb9a00e8e4b031a
4. https://blog.gcp.expert/kubernetes-gke-introduction/
5. https://kubernetes.io/cn/
