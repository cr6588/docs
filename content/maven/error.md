---
title: "Error"
date: 2018-05-17T16:46:22+08:00
draft: true
---
##### 1. 编译时突然死机，重启后编译报错

    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-install-plugin:2.4:install (default-install) on project erp: Failed to install metadata com.sjdf:erp/maven-metadata.xml: Could not parse metadata D:\maven_repo\com\sjdf\erp\maven-metadata-local.xml: only whitespace content allowed before start tag and not \u0 (position: START_DOCUMENT seen \u0... @1:1) -> [Help 1]

删除D:\maven_repo\com\sjdf\erp\maven-metadata-local.xml文件

##### 2. 一直打印zookeeper心跳检测日志

    org.apache.zookeeper.ClientCnxn - Got ping response for sessionid

https://www.jianshu.com/p/fcb6eb0a86d6 <br>
项目jar包存放的lib目录，同时存在logback与log4j,但使用时是log4j去除log4j相关jar包即可.说明在对项目使用mvn clean时并没有删除以前的jar包

##### 3. 未依赖druid时打出的包含有druid

启动web时一直提示DataSource没有配置，但是不需要DataSouce且已经排除，打开debug发现是引入了DruidDataSource，但是在pom.xml的Dependency Hierarchy视图搜索druid未找到，通过命令行mvn dependency:tree发现com.cr6588:spring-beans始终依赖druid

    [INFO] +- com.cr6588:spring-beans:jar:0.0.1:compile
    ...
    [INFO] |  +- com.alibaba:druid-spring-boot-starter:jar:1.1.0:compile
    [INFO] |  |  \- com.alibaba:druid:jar:1.1.0:compile
    [INFO] |  |     +- com.alibaba:jconsole:jar:1.8.0:system
    [INFO] |  |     \- com.alibaba:tools:jar:1.8.0:system
但是对com.cr6588:spring-beans mvn dependency:tree发现其没有对druid的依赖，然后反复对这2各项目进行clean，install以及将druid-spring-boot-starter从对顶层父级项目完全移到另外一个提供者项目，然后对web项目mvn dependency:tree发现仍然有druid，十分不解。最后直接对最顶层父级项目mvn clean一下终于解决。

##### 4. spring boot mybatis提示invalid bound statement

这次错误其实非常简单，但过程很曲折。在做[spring-cloud-learn](https://github.com/cr6588/spring-cloud-learn)，启动完项目后登录时后台提示

    org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.cr6588.dao.UserDao.getUserList ....

记得当时刚好在前2天碰到是通过在provider里加一个

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
    </dependency>

然后测试getUserList就通过了，但再次碰到加上provider本身不应该使用tomcat,所以很疑惑为什么加个tomcat能成功。但再mvn clean install之后发现没用了。然后就又在网上搜索了一下这种情况，很多都是说加@MapperScan注解，不符合当前情况，之后又切换到spring-boot-1.5.10.RELEASE分支，然后测试其provider，能够通过测试。将其逻辑代码覆盖master分支后，仍然不能测试通过。之后覆盖pom文件，当spring-beans中注释掉hibernate-validator时，突然测试通过了，由于之前mvn clean install的问题，反复mvn clean install发现也可以，由于覆盖的工作是在另一个新开的备份eclipse中进行，所以需要将其移入主eclipse。然后在主eclipse中注释掉hibernate-validator后mvn clean install发现还是测试不同，之后又返回测试发现又不可以。对maven打包实在有点无语。之后又继续覆盖provider的pom文件，在确保dependencies完全一样时，测试还不能通过，(好绝望的感觉)。抱着试一试的态度将build也覆盖了，然后mvn clean install,突然测试通过了。之后又反复的mvn clean install发现测试可以通过，这时又搜索了下spring-boot-maven-plugin 打包 invalid bound statement，大部分都提示没有xml文件解决办法，而恰恰第一种方案就是加resources配置。此时终于明白根源是打包时未将mapper的xml文件打包至项目target/classes下。但感觉非常疑惑的是为何当初加入spring-boot-starter-tomcat能够测试通过，注释hibernate-validator时也能测试通过？退回之前的版本在此加入spring-boot-starter-tomcat时发现也不能测试通过了，很不理解当初是测试通过的。很怀疑当初某一次的mvn install时将mapper的xml文件一直保留，之后mvn clean 把其删了之后一直就不能测试通过？这个过程从前一天中午维持到第二天上午。再次体现对maven的一些使用还是很有问题


