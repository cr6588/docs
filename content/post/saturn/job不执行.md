---
title: "saturn"
date: 2019-10-24T15:58:07+08:00
draft: true
---

#### job不执行
console和executor都起来了，但是没有执行对应的job作业，点击立即执行，日志打印msg=job run-at-once triggered, triggeredData:null

检查console的系统配置->ZK集群配置->CONSOLE_ZK_CLUSTER_MAPPING中的zk名称和已有名称是否对应
例如:集群名称是200zk,console默认是default,则此处配置时default:200zk。更新之后有可能不能立即生效，最后重启console生效


#### 添加域名后console退出

日志提示无法从eth0与bond0获取ip信息，saturn只能从这2个网卡获取ip，将网卡更改成eth0或bond0
[参考](https://unix.stackexchange.com/questions/205010/centos-7-rename-network-interface-without-rebooting)

````
    ip link set ens33 down
    ip link set ens33 name eth0
    ip link set eth0 up

    mv /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-eth0
    vi ifcfg-eth0
    修改ifcfg-ens33 -> ifcfg-eth0
    systemctl restart network
````