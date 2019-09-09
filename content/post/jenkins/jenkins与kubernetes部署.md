---
title: "Jenkins与kubernetes部署"
date: 2019-07-10T15:23:52+08:00
draft: false
---

# 让jenkins一键更新已有项目在kubernetes中的版本
使用kubernetes部署项目后，会经常进行发版的操作。其过程不仅繁琐且易出错，所以逐步将发版流程逐步整合进jenkins中。在jenkins中会创建2个job,一个包含编译源码、构建推送镜像、产生版本更新与回滚脚本，另一个负责连接到项目环境进行版本更新、状态监测（dubbo与tomcat）与回滚、邮件通知等过程。
## 开始之前

* 已有kubernetes集群，并有相关pod
* 已有docker,git,maven,jdk环境
* 对shell,kubernetes,docker有基本了解

## 安装jenkins in docker

> 若要在容器中操作网卡相关等涉及到系统的操作最好还是直接安装在外部

之前都是在本地直接安装jenkins，这次通过docker镜像进行安装,由于需要在jenkins中使用docker所以将docker挂载进容器中（会产生权限问题,生产环境慎用）.同时创建一个jenkins_home卷挂载到镜像中可通过docker volume inspect jenkins_home查看主机所在目录。最后将maven等其它软件目录页挂载到镜像中

````shell
docker run -p 8080:8080 -p 50000:50000 -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home/:/var/jenkins_home -v /data:/data -v /usr/bin/docker:/usr/bin/docker --device=/dev/net/tun jenkins/jenkins:lts
````
安装中显示一个密码复制它，实际发现安装过程非常缓慢，测网络连接速度正常。



## rpm包安装jenkins

````shell
rpm -ivh jenkins-2.176.1-1.1.noarch.rpm
#修改配置文件，jdk，端口，主目录，启动用户等按需更改
vi /etc/sysconfig/jenkins
systemctl start jenkins
````

## 构建基础镜像
构建一个使用centos7与java的基础镜像。且修改默认时区,nmap-ncat用于端口检测
Dockerfile如下
````dockerfile
FROM centos:7
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
    systemd-tmpfiles-setup.service ] || rm -f $i; done); \
    rm -f /lib/systemd/system/multi-user.target.wants/*;\
    rm -f /etc/systemd/system/*.wants/*;\
    rm -f /lib/systemd/system/local-fs.target.wants/*; \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
    rm -f /lib/systemd/system/basic.target.wants/*;\
    rm -f /lib/systemd/system/anaconda.target.wants/*;\
    localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8 ;\
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime;\
    yum install -y nmap-ncat

ADD jdk1.8.0_192 /data/jdk1.8.0_192
#配置java环境变量
ENV JAVA_HOME /data/jdk1.8.0_192
ENV PATH $PATH:$JAVA_HOME/bin:$JAVA_HOME/lib
ENV LANG=zh_CN.UTF-8
ENV LANGUAGE=zh_CN:zh
````
构建该镜像
````
docker build -f jdk8-centos-dockerfile -t jdk8-centos7 .
````
## 登录镜像中心
由于需要将之后构建的镜像推送到其它镜像库中，所以我们需要登录其中。可以选择使用nexus搭建或使用官方的docker hub,也可直接使用各个云厂商的容器镜像服务，此示例以华为云的SWR为例，参见官方[获取长期有效docker login指令](https://support.huaweicloud.com/usermanual-swr/swr_01_1000.html)进行登录

![](/images/jenkins/11.png)

登录后会在隐藏目录/root/.docker的config.json文件加入auths信息.

在启动jenkins后进入容器，再执行登录即可
````shell
docker exec -it  c0d5cf8d7b0a  bash
docker login -u cn-north-1@xxx -p xxx swr.cn-north-1.myhuaweicloud.com
````

> 不使用docker安装jenkins时,将.docker目录复制到jenkins所在目录，默认是在/var/lib/jenkins，然后将jenkins加入docker组usermod -G docker jenkins

## 创建源码构建job
该job包含编译源码、构建与推送镜像、产生版本更新与回滚脚本

创建一个自由风格的软件项目

![](/images/jenkins/1.png)

只保留最近3次构建

![](/images/jenkins/2.png)

填写git源码仓库

![](/images/jenkins/3.png)

将当前任务进行保存。由于项目编译需要maven,jdk所以将maven,jdk添加进jenkins中。进入系统管理->全局工具配置

![](/images/jenkins/4.png)

填入jdk路径
![](/images/jenkins/12.png)

填入docker路径
![](/images/jenkins/13.png)

填入maven路径

![](/images/jenkins/5.png)

![](/images/jenkins/6.png)

注意修改maven下载包保存路径，权限以及镜像配置

![](/images/jenkins/7.png)

![](/images/jenkins/8.png)

回到之前的任务并加入maven操作

![](/images/jenkins/9.png)

加入maven之后编译打包已经可以执行了，那么接下来就是需要将jar包或者war包构建成镜像。因此这里开始加入自己的脚本

> 若修改了全局工具配置的maven配置，需要删除已有maven操作

![](/images/jenkins/10.png)

````shell
#为了方便，将mvn clean install之后的jar包等文件复制当前的一个erp目录下,同时为了下次不使用这些文件先将其删除
rm -rf erp
mkdir erp
\cp -rf ../../erp-cache/target/lib erp
\cp -rf ../../erp-cache/target/erp-cache-pg.jar erp
#使用华为云的镜像仓库
export imageUrl=swr.cn-north-1.myhuaweicloud.com
export erp_version=$(date +"%Y-%m-%d_%H-%M-%S")
#构建镜像以之前的jdk8-centos7镜像为基础镜像,将cr6588组织改为自己的组织
docker build -t $imageUrl/cr6588/erp-cache:$erp_version .
#推送镜像
docker push $imageUrl/cr6588/erp-cache:$erp_version
#将版本更新与回滚命令写入文件中（先清空，在进行不换行追加，分成2个文件是实际中会构建多个模块）
cat /dev/null > updateImages
cat /dev/null > rollback
echo -e "updateImages=\c" > updateImages
echo -e "rollback=\c" > rollback
echo -e "kubectl set image deployments/erp-cache erp-cache=$imageUrl/erp/erp-cache:$erp_version && \c" >> updateImages
echo -e "kubectl rollout undo deployments/erp-cache && \c" >> rollback
#最后追加kubectl get pods与一个空串换行
echo -e "kubectl get pods \c" >> updateImages
echo -e "kubectl get pods \c" >> rollback
echo "" >> updateImages
echo "" >> rollback
cat updateImages
cat rollback
````
dockerfile

````dockerfile
FROM jdk8-centos7
ADD erp/lib /data/erp/lib
COPY erp/erp-cache-pg.jar /data/erp/
#容器运行时暴露的端口  
EXPOSE 20001
CMD ["java", "-jar", "/data/erp/erp-cache-pg.jar"]
````

执行构建时提示无法拉取代码发现需要修改hosts文件,直接进入容器执行命令是jenkins用户，此时需要以root用户进入容器中修改host文件

````shell
docker exec --user root -it c0d5cf8d7b0a bash
echo "192.168.1.252 git.xxxx.net" >> /etc/hosts
echo "192.168.1.252 nexus.xxxx.net" >> /etc/hosts
crtl+p crlt+q退出
````

将jenkins加入启动docker时启动
````shell
docker update --restart=always c0d5cf8d7b0a
````

在容器中执行docker命令提示缺少libltdl.so.7，进行安装

````shell
apt-get update && apt-get install -y libltdl7
````
再次构建，会提示缺少权限，这里在宿主机修改/var/run/docker.sock权限
````shell
chmod 666 /var/run/docker.sock
````
或者以root用户进入容器修改，更改所属用户，或组或权限都可，但都会有一定安全隐患

````shell
docker exec --user root -it c0d5cf8d7b0a bash
#更改所属用户
chown jenkins  /var/run/docker.sock
#更改所属组
chgrp jenkins  /var/run/docker.sock
#权限
chmod 666 /var/run/docker.sock
````

在使用jenkins 2.176.1时对shell解析与2.150.1不同，下图左边为2.150.1,右边为2.176.1
![](/images/jenkins/14.png)
![](/images/jenkins/15.png)

可看出右边执行echo -e "updateImages=\c" > updateImages时将-e也输入到updateImages中

将所有脚本写入test.sh中，加入权限chmod +x test.sh再次构建，shell改为./test.sh
![](/images/jenkins/16.png)

> 后来在本机直接安装jenkins2.176.1发现无此问题

此时已经获得更新脚本与回滚脚本，所以接下来就要将这个命令传递到下一个job中。这里使用Parameterized Trigger插件
![](/images/jenkins/17.png)

这个插件用于参数化触发其它job构建，所以先创建一个版本更新的job用于填入这个插件中。
如下图，填入job名称，传递参数使用properties类型，填入updateImages与rollback文件所在路径，当前这个job就已完成

![](/images/jenkins/18.png)
![](/images/jenkins/19.png)

## 创建版本更新job

此job主要是接受上一job的更新越回滚脚本，连接到k8s服务器执行版本更新，检测状态，若不正常就执行回滚，最后发送构建结果邮件

勾选参数化构建过程，写入上一job的脚本中的名称updateImages与rollback
![](/images/jenkins/20.png)

在之后的过程就可使用${updateImages}与${rollback}引用

Build添加执行shell将${updateImages}与${rollback}写入tempUpdateImages.sh与tempRollback.sh文件暂存

````
#updateImages参数是多条命令,作为参数直接传到prod.sh中执行会报错。因此写入新脚本，在prod中调用这个脚本
cat /dev/null > tempUpdateImages.sh
echo '#!/bin/bash' > tempUpdateImages.sh
echo 'set -e -u' >> tempUpdateImages.sh
echo "${updateImages}" >> tempUpdateImages.sh

cat /dev/null > tempRollback.sh
echo '#!/bin/bash' > tempRollback.sh
echo 'set -e -u' >> tempRollback.sh
echo "${rollback}" >> tempRollback.sh
````
![](/images/jenkins/21.png)

在当前job目录（默认是/var/jenkins_home/workspace/版本更新）写入脚本prod.sh与checkStatus.sh

prod.sh
````shell
#!/bin/bash
#一旦出错与未传参立即退出
set -e -u
export KUBECONFIG=/etc/kubernetes/admin.conf
./tempUpdateImages.sh
sleep 10
#检测版本
./checkStatus.sh erp-cache- 20001
kubectl get pods
````

checkStatus.sh
````shell
#!/bin/bash
set -e -u
erpModule=$1
erpPort=$2
#检测版本
#$1不会引用参数
podName=$(kubectl get pods|grep $erpModule |awk '{print $1}')
#重试秒数
waitMillion=10
for((i = 0; i < 3; i++))
do
    #进入容器执行nc 127.0.0.1 端口号，睡眠0.1秒后输出状态，再在外部输入第一行并将结果删除\r
    dubboStatus=$(kubectl exec $podName -- bash -c "(echo status;sleep 0.1)| nc 127.0.0.1 $erpPort"|awk 'NR==1'|tr -d '\r' )
    echo $dubboStatus
    if [[ "$dubboStatus" == "OK" ]]; then
        echo "$podName中dubbo状态正常"
        break
    else
        if (( i < 2 )); then
            echo "$podName中dubbo状态异常，$waitMillion秒后重新获取状态"
            sleep $waitMillion
        else
            echo "$podName中3次dubbo状态检测异常,进行回滚"
            ./tempRollback.sh
            exit 1
        fi
    fi
done
````

增加Publish Over SSH插件用于连接到服务器并执行脚本

![](/images/jenkins/22.png)

在系统管理->系统设置->Publish over SSH中增加服务器配置

![](/images/jenkins/23.png)

返回版本更新中增加构建步骤

![](/images/jenkins/24.png)

设置超时与出错即失败

![](/images/jenkins/25.png)
![](/images/jenkins/26.png)

在连接到140后就会将当前目录下以.sh结尾的文件上传到远程的root目录，并执行prod.sh脚本，在prod.sh中执行tempUpdateImages.sh版本更新，接着调用checkStatus.sh检测状态。在checkStatus.sh中是在容器中利用nc ip 端口号判定dubbo状态是否是OK来决定应用状态是否正常，正常退出，不正常则回滚并错误退出

> Q:为什么不把${updateImages}直接传给prod.sh，然后在prod.sh直接使用$1?

> A:试过，无法执行

最后就是发送邮件通知
在系统管理->系统设置->Jenkins Location中填入管理员邮箱

![](/images/jenkins/27.png)

填入密码相关认证,实际测试发现发件邮箱必须管理员邮箱地址才行，原因未知?

![](/images/jenkins/28.png)

返回job中，增加邮件通知

![](/images/jenkins/29.png)

自此整个过程构建完成，在生产中需要注意权限，切勿直接在生产环境中这样使用