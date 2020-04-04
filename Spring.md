# Spring Framework



翻译文档：Spring Framework 5.2.5 -> Core

链接：https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#spring-core



## 1.IoC 容器

### 1.1.介绍Spring IoC容器和Beans

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



### 1.2.容器概述

`org.springframework.context.ApplicationContext`接口实现IoC，负责实例化、配置和装配bean。通过读取配置元数据，容器获取object实例化、配置和装配的指令。配置的元数据可以是XML、Java注解、Java Code。

`ApplicationContext`接口的实现：在单应用中，通常创建一个`ClassPathXmlApplicationContext` 或者 `FileSystemXmlApplicationContet`实例



应用classes与配置元数据组合，`ApplicationContext`创建并且初始化后，配置好并且应用已经是可执行

![The Spring IoC container](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/container-magic.png)

#### 1.2.1.配置元数据

- XML-based configuration

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="..." class="...">  
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <bean id="..." class="...">
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <!-- more bean definitions go here -->
  
  </beans>
  ```

  - id属性是一个字符串，识别区分bean
  - class属性定义bean的类型和类名

- Annotation-based configuration

- Java-based configuration



#### 1.2.2.实例化一个容器

字符串path路径提供给`ApplicationContext`构造器，加载配置元数据，比如加载本地文件、Java CLASSPATH



`ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");`



services.xml

```xml
<beans >
    <bean id="petStore" class="">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
    </bean>
</beans>
```



daos.xml

```xml
<beans>
    <bean id="accountDao" class=""></bean>
    <bean id="itemDao" class=""></bean>
</beans>
```



services.xml: bean.petStore.accountDao.ref -> daos.xml: bean.accountDao



**导入多个XML配置**

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

不推荐使用的路径`classpath:../services.xml`

绝对路径`file:C:/config/services.xml`



#### 1.2.3. 使用容器

`ApplicationContext`接口增强工厂方法的能力，维护各个bean的注册列表和依赖，使用`getBean`可以获取实例bean



访问bean的例子：

```java
// 创建和配置beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// 获取根据配置实例化的bean
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// 调用实例方法
List<String> userList = service.getUsernameList();
```



### 1.3.Bean 概述

IoC容器管理着多个beans，这些beans是根据用户提供的元数据配置创建的

定义bean的属性：

- Class
- Name
- Scope
- Contructor arguments
- Properties
- Autowiring mode
- Lazy initialization mode
- Initialization method
- Description method



#### 1.3.1.Beans的命名空间

在容器内，每一个bean都有唯一识别

基于XML配置时，id、name区分bean，id是唯一



别名

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```



1.4.

...

1.16.



## 2.资源

## 3.校验、数据绑定、类型转换

## 4.Spring表达式(SpEL)

## 5.切面编程

## 6.AOP 接口

## 7.Null安全性

## 8.数据缓冲和代码

## 9.附录