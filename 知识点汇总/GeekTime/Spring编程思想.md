# Spring编程思想





## Spring编程模型



1、面向对象编程，包含的特性有：

契约接口：Aware，BeanPostProcessor（Bean初始化前后回调函数）

设计模式：观察者模式、组合模式、模板模式



Aware接口对应回调模式，Spring3.1引入，如子接口ApplicationContextAware：

```java
public interface ApplicationContextAware extends Aware {

	void setApplicationContext(ApplicationContext applicationContext) throws BeansException;

}
```

就可以完成对ApplicationContext的回调获取



监视器模式，具体实现就是Spring的事件发布

我们使用的时候只需要使用Publisher来多拨事件，通过ApplicationListener来捕获事件即可

其实Publisher内部调用了ApplicationEventMulticaster的multicastEvent方法，内部是通过ApplicationEventMulticaster这个多播器来实现的。



组合模式，例子为CompositeCacheManager

```java
public class CompositeCacheManager implements CacheManager, InitializingBean {

	private final List<CacheManager> cacheManagers = new ArrayList<>();
}
```

该类实现了CacheManager同时内部也持有CacheManager的列表完成缓存合并



模板模式，JdbcTemplate





2、面向切面编程

动态代理：AopProxy

字节码提升：ASM（类似于汇编语言层面的提升）、CGLIB（动态字节码生成技术）、AspectJ



动态代理AopProxy有几个实现类：

JdkDynamicAopProxy——>CglibAopProxy

越来越底层





3、面向元编程

注解：模式注解（@Component、@Controller、@Service、@Repository）

配置：Environment底层抽象、PropertySources、BeanDefinition

泛型：ResolvableType泛型处理



因为Java注解是不能实现派生的，所以需要在注解上面进行重复标注

如在@Controller和@Service上面就需要标注上@Component注解来实现元注解标注



如Environment就可以获取到Profile，实现在不同Profile下的不同配置信息





4、面向函数

主要提供的@FunctionalInterface接口和WebFlux环境





5、面向模块

主要是@Enable*注解





## Spring IoC基础



大量使用到了Java Bean，在1.1的时候引入，类似于反射，但是比Java的反射好用的太多

使用示例（输出当前Bean所有的方法（构造器方法除外，即使是显式构造器也不会出现在输出当中））：

```java
public static void main(String[] args) throws IntrospectionException {
    BeanInfo beanInfo = Introspector.getBeanInfo(Person.class, Object.class);

  Arrays.stream(beanInfo.getMethodDescriptors()).forEach(System.out::println);
}
```

输出：

```java
java.beans.MethodDescriptor[name=getName; method=public java.lang.String cn.luckycurve.beans.Person.getName()]
java.beans.MethodDescriptor[name=setAge; method=public void cn.luckycurve.beans.Person.setAge(java.lang.Integer)]
java.beans.MethodDescriptor[name=setName; method=public void cn.luckycurve.beans.Person.setName(java.lang.String)]
java.beans.MethodDescriptor[name=getAge; method=public java.lang.Integer cn.luckycurve.beans.Person.getAge()]
```



核心方法是：`public static BeanInfo getBeanInfo(Class<?> beanClass, Class<?> stopClass) throws IntrospectionException`

如果不使用stopClass则会直接输出所有的类，使用stopClass则会屏蔽掉stopClass中的方法，只显示当前层级的方法。



通过BeanInfo还可以获取到PropertyDescriptor，Spring通过PropertyEditor来将Text转换成为相应的数据类型：

```java
public static void main(String[] args) throws IntrospectionException {
    BeanInfo beanInfo = Introspector.getBeanInfo(Person.class, Object.class);

    Arrays.stream(beanInfo.getPropertyDescriptors()).forEach(propertyDescriptor -> {
        String propertyDescriptorName = propertyDescriptor.getName();
        if (Objects.equals("age", propertyDescriptorName)) {
            propertyDescriptor.setPropertyEditorClass(StringToIntegerPropertyEditor.class);
        }
    });
}

static class StringToIntegerPropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        Integer value = Integer.valueOf(text);
        setValue(value);
    }
}
```

