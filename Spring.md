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



#### 1.3.2.beans的实例化

容器构造bean实例

对于静态内部类，`class=com.example.SomeThing$OtherThing`



**静态工厂实例化bean**

```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```



```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```



**使用已经实例化bean的工厂方法来实例化**

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```



```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```



### 1.4.依赖

bean/object之间的依赖



#### 1.4.1.依赖注入

根据object定义的依赖，通过构造参数、参数给工厂方法，或者工厂方法构造返回之后给实例设置属性

注入的两种方式：

- Constructor-based Dependency Injection

  - 构造参数固定

    ```java
    package x.y;
    
    //POJO（Plain Ordinary Java Object）
    public class ThingOne {
        public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
            // ...
        }
    }
    ```

    ```xml
    <beans>
        <bean id="beanOne" class="x.y.ThingOne">
            <constructor-arg ref="beanTwo"/>
            <constructor-arg ref="beanThree"/>
        </bean>
        <bean id="beanTwo" class="x.y.ThingTwo"/>
        <bean id="beanThree" class="x.y.ThingThree"/>
    </beans>
    ```

  - 构造参数类型匹配/索引

    ```java
    package examples;
    
    public class ExampleBean {
        // Number of years to calculate the Ultimate Answer
        private int years;
        // The Answer to Life, the Universe, and Everything
        private String ultimateAnswer;
        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
    
    ```

    ```xml
    <!-- 根据参数类型 -->
    <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg type="int" value="7500000"/>
        <constructor-arg type="java.lang.String" value="42"/>
    </bean>
    
    <!-- 根据参数索引 -->
    <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg index="0" value="7500000"/>
        <constructor-arg index="1" value="42"/>
    </bean>
    ```

  - 构造参数名

    ```xml
    <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg name="years" value="7500000"/>
        <constructor-arg name="ultimateAnswer" value="42"/>
    </bean>
    ```

    ```java
    package examples;
    
    public class ExampleBean {
        // Fields omitted
        @ConstructorProperties({"years", "ultimateAnswer"})
        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
    ```

    

- Setter-based Dependency Injection：工厂方法调取无参构造函数或无参静态构造函数实例化后，调取bean的 setter进行注入

  ```java
  public class ExampleBean {
      private AnotherBean beanOne;
      private YetAnotherBean beanTwo;
      private int i;
    
      public void setBeanOne(AnotherBean beanOne) {
          this.beanOne = beanOne;
      }
      public void setBeanTwo(YetAnotherBean beanTwo) {
          this.beanTwo = beanTwo;
      }
      public void setIntegerProperty(int i) {
          this.i = i;
      }
  }
  ```

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <!-- setter injection using the nested ref element -->
      <property name="beanOne">
          <ref bean="anotherExampleBean"/>
      </property>
  
      <!-- setter injection using the neater ref attribute -->
      <property name="beanTwo" ref="yetAnotherBean"/>
      <property name="integerProperty" value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  ```



用Constructor-based 还是 Setter-based 注入？

- Constructor-based：确保值不为空，直接返回构造好初始好的实例，
- Setter-based：大量的构造参数，有bad code的味道，有更多的责任。要检查空值。好处就是可以在Setter里重新配置或重注入



#### 1.4.2.依赖于配置





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