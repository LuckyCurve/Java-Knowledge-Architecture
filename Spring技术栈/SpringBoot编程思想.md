# 第一部分、总览Spring Boot





## 第一章、初览Spring Boot



Spring Framework对传统Java EE的颠覆方式为新的概念，即两种技术IoC和DI

其实Spring Framework也是在重复造轮子，放眼JSR规范，IoC的实现方式为JNDI（Java命名和目录接口），而DI则是EJB容器注入

Spring Framework的局限性：由于自身并非容器，基本上不得不随着Java EE容器启动而装载，但SpringBoot的出现改变了这一局面

SpringBoot具备开发（Dev）和运维（Ops）双重特性







## 第二章、理解独立的Spring应用



SpringBoot采用了嵌入式容器，独立于外部容器，对应用生命周期拥有完全自主的控制，从此改变了寄人篱下的境地

> 误区：很多人都以为SpringBoot嵌入式Web容器启动时间少于传统的Servlet容器，实际上没有证据表明，只是一种更快捷的启动方式，提升了开发效率而已



口头上习惯将Spring Boot开发的应用称为SpringBoot应用，实际上是Spring应用，对应着标题。

在原来Web容器外部化的时候，Spring应用上下文是依附于Web容器，Spring应用上下文的操作执行例如初始化操作destroy操作都需要结合回掉函数来达到，但是随着SpringBoot的出现，Web容器从属于Spring应用上下文，此时驱动Spring应用上下文启动的核心组件是SpringBoot核心API SpringApplication，所以是Spring应用，也可以称为SpringBoot应用。



接下来讲述了在非图形化环境下完整开发SpringBoot的HelloWorld例子，非常厉害了

windows集成了tree命令，可以直接展示文件树

图形化界面开发SpringBoot程序



只有在创建可执行JAR的时候才需要使用到Spring-boot-maven-plugin到pom中，如果直接使用maven命令启动则无需此插件

java -jar方式运行的jar文件和mvn spring-boot:run命令基本无异，只是前者是生产环境后者是开发环境



打包后生成两个文件：

- .jar.original：Maven原始打包文件，不包含第三方依赖。
- .jar：加工后的文件，包含第三方依赖，springboot-plugin帮的忙



> JAR文件有时也被称为“Fat JAR”，采用zip压缩格式存储，可以解压zip的也可以解压JAR

其中文件的路径：

![image-20200808112711234](images/image-20200808112711234.png)



使用Java -jar命令运行JAR文件时候，就是标准的JAR执行操作了，会读取MANIFEST.MF文件作为启动引导类。

```
Manifest-Version: 1.0
Created-By: Maven Archiver 3.4.0
Build-Jdk-Spec: 11
Implementation-Title: Community Service
Implementation-Version: 0.0.1-SNAPSHOT
如果是WAR包就是WarLauncher
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.applet.Application
Spring-Boot-Version: 2.2.6.RELEASE
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
```

使用`java -jar`命令启动程序是通过引导类JarLauncher来启动主类Start-Class



手动引入spring-boot-loader项目查看JarLauncher

```java
public class JarLauncher extends ExecutableArchiveLauncher {
    static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";
    static final String BOOT_INF_LIB = "BOOT-INF/lib/";

    public JarLauncher() {
    }

    protected JarLauncher(Archive archive) {
        super(archive);
    }

    protected boolean isNestedArchive(Entry entry) {
        return entry.isDirectory() ? entry.getName().equals("BOOT-INF/classes/") : entry.getName().startsWith("BOOT-INF/lib/");
    }

    public static void main(String[] args) throws Exception {
        (new JarLauncher()).launch(args);
    }
}
```

调用Java -jar命令会执行JarLauncher中的main方法，实际上调用的方法：

```java
protected void launch(String[] args) throws Exception {
    // Spring Boot内部引入了依赖的JAR文件，默认是无法扫描到的，SpringBoot通过注册属性java.protocol.handler.pkgs到classpath中去，即扩展了JAR协议的默认内容
    JarFile.registerUrlProtocolHandler();
    // Create a classloader for the specified archives 创建指定的归档（archives）类加载器
    ClassLoader classLoader = createClassLoader(getClassPathArchives());
    // 实际上继续向下委托给了MainMethodRunner#run方法
    launch(args, getMainClass(), classLoader);
}
```

MainMethodRunner#run：

```java
public void run() throws Exception {
    Class<?> mainClass = Thread.currentThread().getContextClassLoader()
        .loadClass(this.mainClassName);
    Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
    mainMethod.invoke(null, new Object[] { this.args });
}
```

> 这里的mainClassName指的就是MANIFEST.MF中的Start-Class属性，然后通过反射调用main方法



WAR实现方式与此类似



**打包WAR文件是一种兼容措施，既能被WarLauncher启动，又能兼容Servlet容器环境，换言之，WarLauncher和JarLauncher并无本质差别，所以建议SpringBoot应用使用非传统方式部署时，尽可能地使用JAR归档方式







## 第三章、理解固化的Maven依赖



SpringBoot为我们提供的固化依赖如：spring-boot-starter-parent和spring-boot-dependencies都提供了如版本信息管理等的固化依赖，如果项目有自己的parent就需要使用到后者了

这里有介绍后者的使用：https://docs.spring.io/spring-boot/docs/2.3.2.RELEASE/maven-plugin/reference/html/#using-import



spring-boot-dependencies是spring-boot-starter-parent的parent，其中包含着大量的dependencyManagement指定的版本号信息，甚至还包含maven plugin的管理信息。



就dependency两者好像并没有实质性的差别，但是在plugin上，spring-boot-starter-parent好像对plugin进行了定制，尽量还是使用parent吧







## 第四章、理解嵌入式Web容器



SpringBoot应用直接嵌入Tomcat、Jetty 和Undertow作为核心特性，在2.0版本还嵌入了Netty来做异步请求处理。



Java对HTTP请求的处理仅有两种选择：Servlet和其他。

前者几乎垄断了Java Web的开发，Tomcat和Jetty作为Servlet的经典实现，后来的Undertow作为JBoss社区推出的新一代Servlet3.1+规范的容器（其实就是Servlet3.1规范推出的时候的实现产品），成为嵌入式Servlet容器的新选择，嵌入式容器不是Spring自家推出的功能，而是Tomcat和Jetty在很久以前就已经支持嵌入式容器了。

![image-20200809154456222](images/image-20200809154456222.png)

至于Reactive Web容器，默认实现为Netty Web Server，与业内其他基于Netty实现的Web Server类似，如Eclipse vert.x。在Spring Boot Webflux中也是使用前者作为默认嵌入式容器实现，也可以通过spring-boot-starter-reactive-netty引入，也支持嵌入式容器。

热门的Web容器均支持嵌入式容器，并非Spring独创



2.2.6的web-starter的Web容器默认实现是9.0.33的Tomcat



接下来是对Tomcat、Jetty和Undertow作为嵌入式Servlet Web容器的讨论



Tomcat：

点进官网选择最老的版本7，可以看到

![image-20200809155335453](images/image-20200809155335453.png)



即使是7版本也对嵌入式容器有支持

实际上Tomcat自己发行了Maven插件，可以不用部署直接打包成可运行的JAR或WAR文件，估计SpringBoot有偷哦。

但不是将Tomcat作为嵌入式容器打包进项目当中，依然是以Tomcat容器为主体，这点和SPringBoot有所不同

不过Tomcat官方只是将该插件维护到了Tomcat7





Jetty：

切换到Jetty相当简单，在Web-starter exclusion掉tomcat-starter，再引入Jetty-starter即可

undertow也是类似





嵌入式Reactive Web容器



嵌入式Reactive Web容器作为SpringBoot2.0的新特性，处于被动激活的状态，只有检测到starter-webflux之后才会激活，并且当web和webflux同时存在的情况下后者会被忽略





可以使用SpringBoot2.0引入的一种ApplicationContext实现——WebServerApplicationContext来完成获取当前WebServer类型：

```java
@Autowired
WebServerApplicationContext context;

@Override
public void run(ApplicationArguments args) throws Exception {
    System.out.println("=======================================");
    System.out.println(context.getWebServer().getClass().getName());
    System.out.println("=======================================");
}
```

上面是通过让主类来实现ApplicationRunner接口来实现的，等效于下面的代码：

```java
@Bean
public ApplicationRunner runner(WebServerApplicationContext context) {
    return args -> {
        System.out.println("=======================================");
        System.out.println(context.getWebServer().getClass().getName());
        System.out.println("=======================================");
    }
}
```

注册进IoC中的ApplicationRunner会在应用启动后回掉 





另一种查看WebServer的方法：WebServerInitializedEvent（弥补非Web应用类型的场景，感觉 没咋读懂，既然引入了WebServer就必然是Web应用类型了呀，在非web情况下注入WebServerApplicationContext会失败）

```java
@EventListener(WebServerInitializedEvent.class)
public void run(WebServerInitializedEvent event){
    System.out.println(event.getWebServer().getClass().getName());
}
```



如今的Tomcat、jetty、Undertow都支持Reactive，可以直接在webflux的starter外直接加入对应的Servlet容器依赖，而不用再去排除掉默认的实现了，估计是SpringBoot2.0推出的webflux的starter支持



一旦引入了这些依赖到classpath路径，我们就可以利用SpringBoot的自动装配特性来完成后续的配置工作。