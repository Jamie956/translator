# Spring Framework



翻译文档：Spring Framework 5.2.5 -> Core

链接：https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#spring-core



## 1.IoC 容器

### 1.1介绍Spring IoC容器和Beans

这一章讲的是Sprig Framework如何实现IoC（Inversion of Control，控制反转），IoC也叫DI（Dependency Injection，依赖注入）。调用工厂方法，仅传入参数，构造实例对象。在创建bean时，容器就注入这些以来（参数？）。通过类构造或者一些机制比如Service Locator pattern，使bean的控制实例化或依赖位置发生反转。



跟IoC有关的包：`org.springframework.beans`、`org.springframework.context`

`BeanFactory`接口提供一个配置机制，可以管理任何类型的object

`ApplicationContext`	是`BeanFactory`的子接口，它的作用：

- 更好与AOP特性整合
- 信息资源处理，比如国际化
- 事件发布
- 应用层声明上下文，比如`WebApplicationContext`



简单来说，`BeanFactory`提供的是一个配置框架和基本功能，`ApplicationContext`则是增加了一些企业级定制功能。



在Spring里，是由object形成骨架，而这个object被Spring IoC容器管理，这个object就叫做bean。bean是一个object，由Spring IoC 容器负责他的实例化、装配和其他一些管理。



1.2

1.3

1.4

...

1.16



## 2.资源

## 3.校验、数据绑定、类型转换

## 4.Spring表达式(SpEL)

## 5.切面编程

## 6.AOP 接口

## 7.Null安全性

## 8.数据缓冲和代码

## 9.附录