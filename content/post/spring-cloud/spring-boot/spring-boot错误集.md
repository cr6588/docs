---
title: "spring-boot错误集"
date: 2018-04-13T13:51:23+08:00
categories: ["spring boot"]
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
参考https://www.cnblogs.com/EasonJim/p/7056700.html，加入, Integer.class解决,但是由于插件获取分页对象时与以前参数名不同，所以后续仍要重新更改

转成spring-boot后断点发现分页插件没有被使用，原来是在yml文件写的plugins.参考官方文档http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html 中Configuration处，发现根本没有plugins配置，后来将plugins配置写入config-location指向的xml文件中，注意plugins顺序，写错了会提示出错并说明了plugins顺序。最终application.yml文件中mybatis如

    mybatis:
        config-location: classpath:sqlMapConfig.xml
        mapper-locations: classpath*:com/sjdf/erp/user/dao/mapper/*.xml

### 关于yml文件
官方推荐使用yml文件，若使用application.yml且配置了spring.profiles在junit中一定要指定spring.profiles.active。类似

    @SpringBootTest(properties = "spring.profiles.active=dev") 
否则测试时会报依赖错误，类似

    Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.cr6588.dao.UserDao' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
        at org.springframework.beans.factory.support.DefaultListableBeanFactory.raiseNoMatchingBeanFound(DefaultListableBeanFactory.java:1493) ~[spring-beans-4.3.14.RELEASE.jar:4.3.14.RELEASE]
        at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1104) ~[spring-beans-4.3.14.RELEASE.jar:4.3.14.RELEASE]
但是若使用application.properties文件则不需要指定。多种环境配置时最好指定。命令行使用--spring.profiles.active=prod会覆盖main方法中的代码。类似

    public static void main(String[] args) throws Exception {
        SpringApplication app = new SpringApplication(App.class);
        Map<String, Object> defaultProperties = new HashMap<String, Object>();
        defaultProperties.put("spring.profiles.active", "dev");
        app.setDefaultProperties(defaultProperties);
        app.run(args);
    }

    java -jar spring-boot-mybatis-xml-beanconfig-yml-0.0.1.jar --spring.profiles.active=prod


1.5.10.RELEASE
不要排除

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>log4j-over-slf4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

### 升级到2.0.1报错
将原项目spring-boot升级到2.0.1之后，许多相关版本也更新至最新版

druid-spring-boot-starter升至1.1.9,
报错
	Caused by: java.lang.ClassNotFoundException: com.alibaba.druid.filter.logging.Log4j2Filter
将druid升级至1.1.9
报错
java.lang.NoSuchMethodError: com.alibaba.druid.sql.ast.expr.SQLAggregateExpr.getOption()Lcom/alibaba/druid/sql/ast/expr/SQLAggregateExpr$Option;
	at com.dangdang.ddframe.rdb.sharding.parser.visitor.basic.mysql.MySQLSelectVisitor.visit(MySQLSelectVisitor.java:110)
	at com.alibaba.druid.sql.visitor.SQLASTOutputVisitor.printExpr(SQLASTOutputVisitor.java:886)
由于Sharding-JDBC1.4.1依赖的druid被替换造成不兼容，升级Sharding-JDBC版本至2.0.1时发现和项目已使用的分库分表代码不兼容的地方较多，遂升级成1.5.4.1。发现该版本不支持or条件，然后项目中大量使用了or故放弃。
因为Sharding-JDBC1.4.1依赖的druid版本为1.0.9，而druid-spring-boot-starter最低依赖版本为1.1.0故舍弃使用改为自定义dataSource配置.后来又使用druid-spring-boot-starter的最低版本暂无问题。

启动的时候发布广播事件时一直提示log4j找不到，但项目已经改为使用logback了，感觉很奇怪

web项目含有jsp时，添加tomcat-embed-jasper
然后参照https://blog.csdn.net/zhoucheng05_13/article/details/77915294 更改
之后启动时出现

    at org.springframework.boot.autoconfigure.jdbc.DataSourceProperties.determineDriverClassName(DataSourceProperties.java:247)
        at org.springframework.boot.autoconfigure.jdbc.DataSourceProperties.initializeDataSourceBuilder(DataSourceProperties.java:184)
        at org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.createDataSource(DataSourceConfiguration.java:42)
        at org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration$Tomcat.dataSource(DataSourceConfiguration.java:56)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
参照https://blog.csdn.net/hengyunabc/article/details/78762097一文追踪原因,在查找DataSourceConfiguration$Tomcat引用时使用ctrl+alt+h一直未找到。后来右键该类选择References->workspace或直接ctrl+shift+g即可.最终排除掉

再次启动出现

    Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!
    at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:296)
    at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:107)
    at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:5118)
    at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5634)
    at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:145)
    at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1571)
    at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1561)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
在maven-war-plugin中增加<failOnMissingWebXml>false</failOnMissingWebXml>

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.4</version>
        <configuration>
            <archive>
                <addMavenDescriptor>false</addMavenDescriptor>
            </archive>
            <includeEmptyDirectories>false</includeEmptyDirectories>
            <webappDirectory>${basedir}/src/main/webapp</webappDirectory>
            <warSourceDirectory>${basedir}/src/main/webapp</warSourceDirectory>
            <!--如果想在没有web.xml文件的情况下构建WAR，请设置为false。-->      
            <failOnMissingWebXml>false</failOnMissingWebXml>
            <!-- <webXml>${basedir}/src/main/webapp/WEB-INF/web.xml</webXml> -->
        </configuration>
    </plugin>

再次启动出现

    java.lang.IllegalStateException: No WebApplicationContext found: no ContextLoaderListener or DispatcherServlet registered?
    at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:252)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:240)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:207)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:212)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:94)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:504)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:141)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
    at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:620)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:88)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:502)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1132)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:684)
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1533)
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1489)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)
这个问题困扰了很久，直接通过main方法启动没有问题，但在外部tomcat中启动就一直显示错误，期间在一个demo项目中搭建了一个web项目，无法重现。在默认配置中spring-boot会初始化默认的WebApplicationContext，且在错误日志中也出现了

    2018-05-02 09:51:10 - INFO - Initializing Spring embedded WebApplicationContext
    2018-05-02 09:51:10 - INFO - Root WebApplicationContext: initialization completed in 3599 ms
    2018-05-02 09:51:11 - DEBUG - Added existing Servlet initializer bean 'dispatcherServletRegistration'; order=2147483647, resource=class path resource [org/springframework/boot/autoconfigure/web/DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration.class]

说明有WebApplicationContext，但为什么会提示No WebApplicationContext found: no ContextLoaderListener or DispatcherServlet registered?。最后在项目中只注入一个testController，取消其它额外的注入发现在外部tomcat中没问题。最后逐步增加终于发现spring-session配置类的问题

添加400相关错误页面，在之前的web项目中，错误页面结构如下
![x](/images/400.png)
在网上许多教程说定义EmbeddedServletContainerCustomizer类似

    @Bean
    public EmbeddedServletContainerCustomizer containerCustomizer() {
        return container -> {
                ErrorPage errorPage400 = new ErrorPage(HttpStatus.BAD_REQUEST,"/400.jsp");
                ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND,"/404.jsp");
                ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR,"/500.jsp");
                container.addErrorPages(errorPage404, errorPage500, errorPage400);
            };
    }
试了下是有用但仅限于用内置容器，当部署在外部tomcat中就无用，后来从其命名前缀Embedded（嵌入式）也可体现出来，亦或是方法没用对？
在官方文档
https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-error-handling 推荐继承BasicErrorController，继承之后如下

    @Controller
    public class CustomErrorController extends BasicErrorController {
    
        public CustomErrorController(ServerProperties serverProperties) {
            super(new DefaultErrorAttributes(), serverProperties.getError());
        }
    
        @Override
        public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
            HttpStatus status = getStatus(request);
            String errorViewName = "500";
            if (status != null) {
                Integer statusCode = Integer.valueOf(status.toString());
                if (statusCode == HttpStatus.NOT_FOUND.value()) {
                    errorViewName = "404";
                }
            }
            return new ModelAndView("error/" + errorViewName);
        }
    
    }
错误页面url默认为/error,view的路径=spring.mvc.view.prefix配置的路径+error/400(500) + spring.mvc.view.suffix。
在外部tomcat中能够正确导向到错误页面，但是在使用嵌入式的容器运行时发现被登录拦截器拦截了。而登录拦截器在外部tomcat中没有登录是会导向到登录页面的，有点不理解
拦截器部分代码如下

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String url = request.getRequestURI();
        if (noLoginRequired(url)) {
            return true;
        }
        HttpSession session = request.getSession();
        Object userVo = session.getAttribute(ConstLogin.SESSION_USER);
        if (userVo != null && userVo instanceof SessionUserVo) {
            return true;
        }
        ....

调试发现在内置tomcat容器中访问/xxxx时userVo能从session中获取到，但在之后报错导向到/error时userVo为空，而在外部tomcat中2次都能获取到userVo。查看session发现在内置tomcat中session类似（/xxxx与/error一组），2次session引用不一致

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@2ddb8765
    org.apache.catalina.session.StandardSessionFacade@557797ad
    
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@1e203ca8
    org.apache.catalina.session.StandardSessionFacade@557797ad
    
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@94f25ba
    org.apache.catalina.session.StandardSessionFacade@557797ad
外置tomcat中，2次session引用一致

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@769a7a87
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@769a7a87

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper@6ede7418
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper@6ede7418

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@7c8e1ce2
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@7c8e1ce2

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@4cfdb71f
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@4cfdb71f

    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@6fbdb46d
    org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper@6fbdb46d

登录拦截器放开/error暂时避免这种错误.

将项目部署至eclipse的servers中的tomcat时，一直提示缺少web.xml，但项目并不需要web.xml.在.settings/org.eclipse.wst.common.component文件（对应项目右键->Properties->Deployment Assembly）中看到一行web.xml配置

    <wb-resource deploy-path="/WEB-INF/web.xml" source-path="/src/main/webapp/WEB-INF/web.xml"/>
将其注释后不再出现该提示。但取消注释后也不再出现该提示，另外一个web项目也有这行但从未出现这个提示。尚不清楚原因

javax/el/ELManager出错
    Caused by: java.lang.NoClassDefFoundError: javax/el/ELManager

加入
        <dependency>
            <groupId>javax.el</groupId>
            <artifactId>javax.el-api</artifactId>
            <version>3.0.0</version>
        </dependency>
tomcat8不需要

tomcat version:7.0.47出现
严重: Unable to process Jar entry [module-info.class] from Jar [jar:file:/D:/maven_repo/org/apache/logging/log4j/log4j-api/2.10.0/log4j-api-2.10.0.jar!/] for annotations
org.apache.tomcat.util.bcel.classfile.ClassFormatException: Invalid byte tag in constant pool: 19
..
升级tomcat ，tomcat 8.0.45未出现