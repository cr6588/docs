# eclipse项目server无法启动，没有spring-web-XXXX.jar包

eclipse项目server无法启动，没有spring-web-XXXX.jar包，运行时报错：

Error configuring application listener of class org.springframework.web.context.ContextLoaderLis

可能的原因是项目根目录中的.classpath
eclipse项目server无法启动，没有spring-web-XXXX.jar包
文件中maven没有指定lib路径：
org.eclipse.jst.component.dependency = /WEB-INF/lib 
加上就好了。
也可项目右键properties->web deployment assembly->add->java build path entries->双击Maven dependencies