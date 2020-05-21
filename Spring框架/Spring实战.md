# 第一部分、Spring基础





## 第一章、Spring起步



Spring的发展历程：

最常见的应用形式是基于浏览器的Web应用，后端由关系型数据库作为支撑。尽管这种形式的开发依然有它的价值，Spring也为这种应用提供了良好的支持，但是我们现在感兴趣的还包括如何开发面向云的由微服务组成的应用，这些应用会将数据保存到各种类型的数据库中。另外一个崭新的关注点是反应式编程，它致力于通过非阻塞操作提供更好的扩展性并提升性能。



Spring的核心是一个容器，也被称为应用上下文（Application Context），它们会创建和管理应用组件。这些组件也可以称为bean，会在Spring应用上下文中装配在一起，从而形成一个完整的应用程序。



配置类中：默认情况下，这些bean所对应的bean ID与定义它们的方法名称是相同的



创建第一个项目：

启动类：

```
@SpringBootApplication包含：
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(
        excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
    )
```

> 实际上有用的：
>
> - SpringBootConfiguration：@Configuration注解的特殊形式
> - EnableAutoConfiguration：启动SpringBoot的自动装配
> - ComponentScan：启用组件扫描。扫描@Component、@Service这种注解声明的其他类，并将他们注册到Spring应用上下文中



```
SpringApplication.run(TacoCloudApplication.class, args);
```

创建Spring应用上下文，一个是配置类，一个是启动参数（尽管传递的配置类不一定要和引导类相同，但这是最便利和最经典的做法，尽管也可以在引导类中配置一两个组件，但是还是建议单独创建一个配置类）



测试类：

主要就是@SpringBootTest注解了，提供SpringBoot的测试环境，相当于调用了引导类的SpringApplication.run（）方法

也可以在测试类上面加上@Runwith注解（Junit提供的）以前的测试类头上会带上这个注解@Runwith(SpringRunner.class)，现在好像都没有这个注解了（SpringBoot 2.3.0）

可以跑空的测试类来判断是否能启动应用上下文



编写一个控制类：

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

@Controller注解：并没有做太多事情，仅仅只是让当前类被扫描，装载到容器中

@GetMapping注解：针对"/"发送的HTTP GET请求，那么这个方法将会处理请求

返回值会交给thymeleaf去进行视图解析，加上前缀“/templates/”和后缀".html"。拼接成"/templates/home.html"



编写一个视图页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="https://www.thymeleaf.org/">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
</head>
<body>
    <h1>Welcome to</h1>
    <img th:src="@{/images/Logo.png}">
</body>
</html>
```

要手动引入th标签，要不然报错

唯一需要注意的是`th:src="@{/images/Logo.png}"`，这里的意思是读取"/static/images/Logo.png"这张图片





测试器：对HTML页面的内容断言是很困难的，SpringMVC提供了强大的测试。感觉蛮不好用的，浏览器测试更好





Spring Boot DevTools

作用：

- 代码变更后自动重启（依赖更新后无效，需要重启，配置文件修改了之后有效）
- 面向浏览器的资源（HTML，JS）发生变化时候自动刷新浏览器（需要安装LiveReload插件）
- 自动禁用模板缓存
- 如果使用H2数据库，内置了H2的控制台

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

默认指定打包之后就没用了（只在生产环境有效）

实现了两个类加载器，一个加载Java代码，属性和项目中基本所有的内容，另一个去加载依赖的库，每次发生改变的时候，只需要重启一个类加载器即可，另一个相对稳定的类加载器（加载依赖的那一个）就不会动了。

如果使用了H2数据库，会生成一个URL为http://localhost:8080/h2-console的控制台





## 第二章、开发Web应用



>  主要是MVC层的数据绑定
>
> 章节概括：
>
> - 在浏览器中展现模型数据
> - 处理和校验表单输入
> - 选择视图模板库



REST API：是指控制器直接返回Model对象，不进行页面渲染。



Demo：

实体类

```java
@RequiredArgsConstructor
@Data
public class Ingredient {
    private final Integer id;
    private final String name;
    private final Type type;
    public static enum Type{
        /**
         * 字段注释
         */
        WRAP,PROTEIN,VEGGIES,CHEESE,SAUCE
    }

}
```



Controller层：

```java
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {
    /**
     * 默认就是@GetMapping("")
     */
    @GetMapping
    public String showDesignForm(Model model) {
        List<Ingredient> list = Arrays.asList(
                new Ingredient(1, "Lettuce", Ingredient.Type.PROTEIN),
                new Ingredient(2, "Lettuces", Ingredient.Type.SAUCE),
                new Ingredient(3, "Lettuces", Ingredient.Type.CHEESE),
                new Ingredient(4, "Lettuces", Ingredient.Type.WRAP)
        );
        Ingredient.Type[] types = Ingredient.Type.values();
        for (Ingredient.Type type : types) {
            model.addAttribute(type.toString().toLowerCase(),filterByType(list,type));
        }
        return "design";

    }

    private List<Ingredient> filterByType(List<Ingredient> ingredients, Ingredient.Type type) {
        return ingredients.stream().filter(x->x.getType().equals(type)).collect(Collectors.toList());
    }

}
```

> 作者的编码太厉害了



Thymeleaf这种视图解析器很明显是和特定的Web框架解耦的，那么是如何获取数据的呢？显然无法直接访问到Model对象，是在到达控制器的时候就将数据从Model复制到了request里面，就可以访问到了。



在HTML中使用th：each标签去遍历结果集即可。一般都是`${}`访问request，如果取到的是一个对象，则直接使用对象的toString方法变成字符串展示在页面上。



数据校验

Spring支持Java的Bean校验API（Bean Validation API，JSR-303）

具体支持注解看[常见错误&基础结论](../常见错误&基础结论.md)





如果要写视图控制器，去继承WebMvcConfigurer，重写addViewControllers方法。也就是取代那些需要直接返回view的controller，感觉没啥用。



本章学习了强大的Web框架——Spring MVC

基于注解的，支持校验，支持视图解析







## 第三章、使用数据

使用Spring对JDBC的支持（Java DataBase Connectivity）消除样式代码（什么Class.forName（）这种代码），最后使用JPA（Java Presistence API）重写数据，消除更多冗杂代码



JDBCTemplate仅仅只是省去了大量的模板创建，依然需要自定义SQL，但是可以帮你把返回值按照字段名称自动注入到对象中去，占位符依旧使用？，跟原来的JDBC一模一样。



在插入的时候可以使用Spring提供的封装性较高的SimpleJdbcInsert



Spring Data有很多子项目支持不同的数据库：

- Spring Data JPA：基于关系型数据库进行JPA持久化（直接屏蔽掉了底层关系型数据库的类型，让开发人员不用和数据库打交道了）
- Spring Data MongoDB
- Spring Data Neo4j
- Spring Data Redis...



JPA底层的默认实现就是Hibernate，可以改变（在jpa里面排除hibernate）引入你需要的即可。



创建的Repository中的方法会根据方法名来自动实现（如果条件复杂可以使用@Query注解，然后自定义方法名）



Demo：



application.yml

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
```

这里需要设置ddl，默认不会创建表的，会报错



实体类：

```java
@Entity
@Data
public class TechStack {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String techName;
}
```



repo：

```java
public interface TechStackRepo extends CrudRepository<TechStack,Integer> {

    List<TechStack> findByTechNameLike(String name);
}
```



这里的方法都会根据方法名自动实现





## 第四章、保护Spring



启动Spring Security来保护Spring

引入这个配置之后，会自动进行安全的基本配置

访问网站需要输入用户名和密码：用户名默认为user，密码为控制台输出的一个字符串，也是通过浏览器的JSESSIONID来匹配到服务器中的Session域的，如果清除了Cookie就需要重新登录。

默认安全信息：

- 所有HTTP请求都需要认证
- 不需要特定的角色和权限
- 没有登录页面（默认页面是HTTP basic认证的框）
- 默认只有一个用户user



增加用户：

1.用户数据量不大，修改次数少。直接存内存：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //需要加密方式
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("buzz").password(new BCryptPasswordEncoder().encode("222")).roles("USER");
    }

}
```

要指定密码加密方式



具体的加密方式：

•BCryptPasswordEncoder：使用bcrypt强哈希加密。

•NoOpPasswordEncoder：不进行任何转码。

•Pbkdf2PasswordEncoder：使用PBKDF2加密。

•SCryptPasswordEncoder：使用scrypt哈希加密。

•StandardPasswordEncoder：使用SHA-256哈希加密。



2.存储到数据库中，是使用JDBC来存储的，可以百度下



都是指定好的，用起来极其不方便





自定义用户认证：

依靠Spring Data JPA

对比了下Mybatis+Shiro，突然知道为什么大多数国内的厂商都倾向于Shiro了，Spring Security提供了太多的功能了（然而这些功能大多数情况下都用不上）。







## 第五章、使用配置属性





细粒度的自动配置

配置类中的@Bean方法，将返回值注入，很容易测试：

```java
@Bean
public String hello(){
    String a = "hello world";
    System.out.println(a);
    return a;
}
```

在配置类中加入这一行即可，会明显的看到hello world的输出

![image-20200521103711284](images/image-20200521103711284.png)

Spring完成的自动属性注入

例如Server.port=8080可以在任何一个阶段设置（JVM参数，命令行参数，操作系统环境变量（命名格式需要改变，Spring会自动适配））



接下来讲述了一些常规的可配置项，

数据库配置：例如spring.datasource

嵌入式服务器配置：例如：Server.port=0（随便选择一个可用端口）在不关心应用在哪个端口启动的情况非常重要，如成为一个服务注册到注册中心去

指定配置文件（在主配置文件里面指定）:

```yaml
spring:
  profiles:
    active: dev
```

配置文件命名：application-{profiles}.yaml

这是设置的最糟糕的一种方式，会直接使用指定的配置文件（无论是在开发环境还是在生产环境都是一样）。无法享受到生产环境和开发环境不同所带来的便利

可以在IDE中手动指定：

![image-20200521152955643](images/image-20200521152955643.png)

或者使用命令行的方式：

![image-20200521153010111](images/image-20200521153010111.png)





日志的级别设置：

```log
logging:
  level:
    root: debug
  file:
#    好像路径没用，都直接输出到项目的根目录下面了
    path: D://
    name: Test.log
```

必须使用到root上面去，要不然报错



配置类的写法：

```java
@Configuration
@ConfigurationProperties("greeting")
@Setter
public class HelloConfig {
    private String hello;

    @Bean
    public String bean() {
        System.out.println(hello);
        return hello;
    }
}

```

一定要setter注解，要不然属性注入不进去



只要是被扫描到的，无论是使用Copeonent系列的还是Configuration，里面被标注了bean方法都会被加载，返回值进入到IOC容器中。因为COnfiguration也就相当于是Component的变种（标注在配置类上面）





可以根据当前profiles的不同来选择配置（如果指定的是dev，但dev配置没有这个属性，就会再回去读取默认的配置）和指定是否创建bean

```java
@Configuration
@ConfigurationProperties("greeting")
@Data
@Profile("!dev")
public class HelloConfig {
    private String hello;
    
    @Bean
    @Profile("!dev")
    public String hello() {
        System.out.println(hello);
        return hello;
    }
}
```

如果是以下配置`@Profile({"!dev","!dev2"})`，是表示只要满足其中一个条件即可