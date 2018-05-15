---
title: "Web Upgrade"
date: 2018-04-27T10:12:30+08:00
draft: true
---

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

登录拦截器放开/error暂时避免这种错误