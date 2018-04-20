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

出现
    
    Factory method 'sqlSessionFactory' threw exception; nested exception is java.lang.NoSuchMethodError: org.mybatis.spring.SqlSessionFactoryBean.setVfs(Ljava/lang/Class;)V
或者

    Caused by: org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.binding.BindingException: Parameter 'id' not found. Available parameters are [arg1, arg0, param1, param2]
    at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:77)
    at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:446)
    at com.sun.proxy.$Proxy70.selectList(Unknown Source)
    at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:230)
    at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:137)
查看mybatis版本问题，找了好久才意识到引入的依赖包中使用了以前的版本，排除调以前的版本后在demo项目中顺利启动。后来想想仔细查看错误信息都NoSuchMethodError这么明显了应该想到的是版本问题的，还是太虎了。。。

在实际项目中先升级mybatis版本后，引入分页插件以后发现提示

    Caused by: org.apache.ibatis.plugin.PluginException: Could not find method on interface org.apache.ibatis.executor.statement.StatementHandler named prepare. Cause: java.lang.NoSuchMethodException: org.apache.ibatis.executor.statement.StatementHandler.prepare(java.sql.Connection)
        at org.apache.ibatis.plugin.Plugin.getSignatureMap(Plugin.java:87)
        at org.apache.ibatis.plugin.Plugin.wrap(Plugin.java:44)
参考https://www.cnblogs.com/EasonJim/p/7056700.html，加入, Integer.class解决

转成spring-boot后断点发现分页插件没有被使用，原来是在yml文件写的plugins.参考官方文档http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html 中Configuration处，发现根本没有plugins配置，后来将plugins配置写入config-location指向的xml文件中，注意plugins顺序，写错了会提示出错并说明了plugins顺序。最终application.yml文件中mybatis如

    mybatis:
        config-location: classpath:sqlMapConfig.xml
        mapper-locations: classpath*:com/sjdf/erp/user/dao/mapper/*.xml


