---
title: "RocketMQ"
date: 2018-09-03T15:27:54+08:00
categories: ["RocketMQ"]
---

http://rocketmq.apache.org/docs/quick-start/

nohup sh bin/mqnamesrv &

提示JAVA_HOME不存在。但实际已配置，echo $JAVA_HOME之后又正常

nohup sh bin/mqbroker -n localhost:9876 &

提示error='Cannot allocate memory',内存不足，修改bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn256m"

按照官方文档出错太多放弃。（阿里文档一贯风格）...