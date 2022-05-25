---
title: "Centos7笔记"
date: 2018-05-25T09:25:07+08:00
categories: ["linux"]
---

https://www.cnblogs.com/moxiaoan/p/5683743.html
CentOS7使用firewalld打开关闭防火墙与端口
1、firewalld的基本使用
启动： systemctl start firewalld
查看状态： systemctl status firewalld 
停止： systemctl disable firewalld
禁用： systemctl stop firewalld
 
2.systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。
启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed

3.配置firewalld-cmd

查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
更新防火墙规则： firewall-cmd --reload
查看区域信息:  firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic


systemctl start firewalld && firewall-cmd --zone=public --add-port=51022/tcp --permanent  && firewall-cmd --reload

那怎么开启一个端口呢
添加
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
重新载入
firewall-cmd --reload
查看
firewall-cmd --zone=public --query-port=80/tcp
删除
firewall-cmd --zone=public --remove-port=80/tcp --permanent
指定ip
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="125.65.112.58" port protocol="tcp" port="8081" accept" && firewall-cmd --reload


firewall-cmd --query-masquerade # 检查是否允许伪装IP
firewall-cmd --permanent --add-masquerade # 允许防火墙伪装IP
firewall-cmd --permanent --remove-masquerade# 禁止防火墙伪装IP
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.0.1:toport=8080

---------------------

---------------------
iptables -t nat -L

#修改hostname
hostnamectl set-hostname name

#### 修改网卡名称
[参考](https://unix.stackexchange.com/questions/205010/centos-7-rename-network-interface-without-rebooting)
ip link set ens33 down
ip link set ens33 name eth0
ip link set eth0 up

ip link set em1 down
ip link set em1 name eth0
ip link set eth0 up
ip link set eth0 up

mv /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-eth0

vi ifcfg-eth0
修改ifcfg-ens33 -> ifcfg-eth0

systemctl restart network

> 断电重启后失效

# 挂载硬盘
> 仅适用于使用 fdisk 命令对一个不大于 2 TB 的数据盘执行分区操作
https://zhuanlan.zhihu.com/p/145773155
````shell
#先查看未指派的分区名称
fdisk -l
#创建硬盘分区
fdisk /dev/xxxxx
交互式操作，首先输入n #回车添加新分区，其他直接回车

#分区完成。输入fdisk -l查看信息
#格式化分区sdb1磁盘的设备名
mkfs.ext4 /dev/sdb1
#建立挂载目录
mkdir /data
#挂载分区
mount /dev/sdb1 /data
#设置开机自动挂载
echo /dev/sdb1 /data ext4 defaults 0 0 >> /etc/fstab
#重启系统,确认是否挂载成功
reboot
````