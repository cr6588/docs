---
title: "Project Upgrade"
date: 2018-04-13T13:51:23+08:00
draft: true
---

项目迁移至spring-boot

### 添加spring-boot


    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.0.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <version>2.0.1.RELEASE</version>
    </dependency>

编译时报错

    [proguard] java.io.IOException: Can't read [D:\maven_repo\org\apache\logging\log4j\log4j-api\2.10.0\log4j-api-2.10.0.jar] (Can't process class [META-INF/versions/9/org/apache/logging/log4j/util/ProcessIdUtil.class] (Unsupported class version number [53.0] (maximum 52.0, Java 1.8)))
     [proguard]     at proguard.InputReader.readInput(InputReader.java:188)
     [proguard]     at proguard.InputReader.readInput(InputReader.java:158)
     [proguard]     at proguard.InputReader.readInput(InputReader.java:136)
     [proguard]     at proguard.InputReader.execute(InputReader.java:88)
     [proguard]     at proguard.ProGuard.readInput(ProGuard.java:218)
     [proguard]     at proguard.ProGuard.execute(ProGuard.java:82)
     [proguard]     at proguard.ProGuard.main(ProGuard.java:538)
     [proguard] Caused by: java.io.IOException: Can't process class [META-INF/versions/9/org/apache/logging/log4j/util/ProcessIdUtil.class] (Unsupported class version number [53.0] (maximum 52.0, Java 1.8))
     [proguard]     at proguard.io.ClassReader.read(ClassReader.java:112)
     [proguard]     at proguard.io.FilteredDataEntryReader.read(FilteredDataEntryReader.java:87)
     [proguard]     at proguard.io.FilteredDataEntryReader.read(FilteredDataEntryReader.java:87)
     [proguard]     at proguard.io.JarReader.read(JarReader.java:65)
     [proguard]     at proguard.io.DirectoryPump.readFiles(DirectoryPump.java:65)
     [proguard]     at proguard.io.DirectoryPump.pumpDataEntries(DirectoryPump.java:53)
     [proguard]     at proguard.InputReader.readInput(InputReader.java:184)
     [proguard]     ... 6 more
     ...

降低spring-boot版本至1.5.10.RELEASE编译成功

前往https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#boot-documentation 官方文档参考

发现[在没有Parent POM的情况下使用Spring Boot](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#using-boot-maven-without-a-parent)

由于项目原先使用log4j,而spring-boot默认是logback所以需要排除log4j
排除冲突JAR包 https://blog.csdn.net/wangb_java/article/details/60330000
直接在pom.xml搜索冲突的jar包，右键选择Exculde

### 加入dubbo
仍然使用传统xml文件配置

出现xxService is not visible from class loader...
详见https://github.com/dangdangdotcom/dubbox/issues/218
由于spring-boot-devtools导致
若出现xxxServiceImpl cannot be cast to xxxServiceImpl spring-boot-devtools原因仍然类似

当把provider中的bean与service移入到另外一个子模块时，mvn clean install后出现

    org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.cr6588.dao.UserDao.getUserList
        at org.apache.ibatis.binding.MapperMethod$SqlCommand.<init>(MapperMethod.java:225)
        at org.apache.ibatis.binding.MapperMethod.<init>(MapperMethod.java:48)

去除spring-boot-devtools又显示正常，不得不暂时移除spring-boot-devtools。后期需要对spring-boot-devtools做出调查

