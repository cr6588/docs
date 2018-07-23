---
title: "Error"
date: 2018-05-17T16:46:22+08:00
draft: true
---
1.编译时突然死机，重启后编译报错

    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-install-plugin:2.4:install (default-install) on project erp: Failed to install metadata com.sjdf:erp/maven-metadata.xml: Could not parse metadata D:\maven_repo\com\sjdf\erp\maven-metadata-local.xml: only whitespace content allowed before start tag and not \u0 (position: START_DOCUMENT seen \u0... @1:1) -> [Help 1]

删除D:\maven_repo\com\sjdf\erp\maven-metadata-local.xml文件

2.一直打印zookeeper心跳检测日志

    org.apache.zookeeper.ClientCnxn - Got ping response for sessionid

https://www.jianshu.com/p/fcb6eb0a86d6 <br>
项目jar包存放的lib目录，同时存在logback与log4j,但使用时是log4j去除log4j相关jar包即可.说明在对项目使用mvn clean时并没有删除以前的jar包
