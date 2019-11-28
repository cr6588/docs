---
title: "Job不执行"
date: 2019-10-24T15:58:07+08:00
draft: true
---

console和executor都起来了，但是没有执行对应的job作业，点击立即执行，日志打印msg=job run-at-once triggered, triggeredData:null

检查console的系统配置->ZK集群配置->CONSOLE_ZK_CLUSTER_MAPPING中的zk名称和已有名称是否对应
例如:集群名称是200zk,console默认是default,则此处配置时default:200zk。更新之后有可能不能立即生效，最后重启console生效