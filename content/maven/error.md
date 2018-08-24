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

3.启动web时一直提示DataSource没有配置，但是不需要DataSouce且已经排除，打开debug发现是引入了DruidDataSource，但是在pom.xml的Dependency Hierarchy视图搜索druid未找到，通过命令行mvn dependency:tree发现com.cr6588:spring-beans始终依赖druid

    [INFO] +- com.cr6588:spring-beans:jar:0.0.1:compile
    ...
    [INFO] |  +- com.alibaba:druid-spring-boot-starter:jar:1.1.0:compile
    [INFO] |  |  \- com.alibaba:druid:jar:1.1.0:compile
    [INFO] |  |     +- com.alibaba:jconsole:jar:1.8.0:system
    [INFO] |  |     \- com.alibaba:tools:jar:1.8.0:system
但是对com.cr6588:spring-beans mvn dependency:tree发现其没有对druid的依赖，然后反复对这2各项目进行clean，install以及将druid-spring-boot-starter从对顶层父级项目完全移到另外一个提供者项目，然后对web项目mvn dependency:tree发现仍然有druid，十分不解。最后直接对最顶层父级项目mvn clean一下终于解决。