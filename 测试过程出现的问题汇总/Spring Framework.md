# 问题汇总



## 1、IoC Bean之间的继承和注入关系



如果存在以下情况，

```java
Class B extends A;
```

此时IoC中存在A，B两个实例，此时`@Autowired`B就是注入的B实例，`@Autowired`A就是注入的A实例，不会报错。但是如果是如下情况：

```java
Class B extends A;
Class C extends B;
```

此时IoC容器中存在B，C两个实例，此时`@Autowired`C就会报错。