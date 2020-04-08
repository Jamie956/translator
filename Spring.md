# Spring Framework



翻译文档：Spring Framework 5.2.5 -> Core

链接：https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#spring-core



## 1.IoC 容器

### 1.1.介绍Spring IoC容器和Beans

IoC（Inversion of Control，控制反转），IoC也叫DI（Dependency Injection，依赖注入）。调用工厂方法，仅传入参数，构造实例对象。在创建bean时，容器就注入这些依赖。通过类构造或者一些机制比如Service Locator pattern，使bean的控制实例化或依赖位置发生反转。



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



#### 1.4.2.依赖与配置

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```



**转型**

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
              	<!-- value是double，向上转型，转成float -->
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```



**Null和空字符串处理**

空字符串：

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

相当于
```java
exampleBean.setEmail("");
```



Null值：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

相当于

```java
exampleBean.setEmail(null);
```



#### 1.4.3.使用 `denpends-on`

bean属性`depends-on`可以关联多个依赖关系的bean，先去初始化他们

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```



#### 1.4.4.懒加载beans

`ApplicationContext`预初始化式eagerly饿汉式，创建和配置全部单例beans，如果你不喜欢，可以使用懒加载，IoC在你第一次请求bean的时候再去创建实例

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
```



```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```



#### 1.4.5.自动装配



#### 1.4.6.方法注入

假设有单例bean A，非单例bean B，A用到B，A只会初始化一次，如果B更改了，那么A里的B就不是最新，怎么保证A里的B是新的？



Lookup Method Injection：容器重写方法，返回lookup结果，lookup一般是prototype bean，cglib动态生成二进制subclass重写方法

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>

```



### 1.5.bean域

同样一个菜（instance），做多少碟，都是按一个菜谱（bean definition）



- 定义好的bean创建的object，可以塞了各个依赖和配置
- 可以控制这个object的域



bean域：

- singletion：默认的域，对于每个IoC容器，单个bean定义，只创建单个实例
- prototype：单个bean定义，可以用来创建多个实例
- request：单个bean定义，每个HTTP请求，都会按照定义创建一个实例
- session：单个bean定义，对应HTTP Session的生命周期，产生一个Session就创建一个实例
- application：to lifecycle of a ServletContext
- websocket：to lifecycle of a WebSocket



#### 1.5.1.单例域

singleton scope

singleton只有一个共享实例，根据id从容器获取bean

对于singleton scope的bean，容器只会根据bean的定义创建一个实例，实例储存在cache里

![](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/singleton.png)



#### 1.5.2.原型域

prototype scope

每次获取的bean，都是一个新创建的bean

![](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/prototype.png)

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```



#### 1.5.4.web相关的域



Request, Session, Application, and WebSocket Scopes



```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

bean userPreferences 属于session scope，每产生一个新的session都会有一个新的实例，而userManager默认属于singleton scope，就出现这样的问题，userManager只会初始化一遍，所以他的属性userManager.userPreferences是不会变的，即使userPreferences发生了改变，所以userPreferences在新建一个实例的时候，得告诉userManager，方法是userManager实例调用方法时使用代理，注入userPreferences，获取最新版本的userPreferences。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```



使用JDK代理：

```xml
<aop:scoped-proxy proxy-target-class="false"/>
```



#### 1.5.5.自定义域

custom scopes



### 1.6.自定义bean

#### 1.6.1.生命周期回调

**初始化回调**：`org.springframework.beans.factory.InitializingBean`接口 在容器装配好全部属性之后，执行初始化工作。



`InitializingBean`接口声明了这个方法：

```java
void afterPropertiesSet() throws Exception;
```



- 不推荐使用``InitializingBean`接口，会造成代码与spring的耦合

  ```xml
  <bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
  ```

  ```java
  public class AnotherExampleBean implements InitializingBean {
      @Override
      public void afterPropertiesSet() {
          // do some initialization work
      }
  }
  ```

- 可以使用`init-method`或者注解` @PostConstruct`

  ```xml
  <bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
  ```

  ```java
  public class ExampleBean {
      public void init() {
          // do some initialization work
      }
  }
  ```

  

**销毁回调**：`org.springframework.beans.factory.DisposableBean`实现该能力，当包含be an的容器销毁时，获取一个回调



`DisposableBean`声明了这个方法：

```java
void destroy() throws Exception;
```



- 不推荐直接使用`DisposableBean`接口

  ```xml
  <bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
  
  ```

  ```java
  public class AnotherExampleBean implements DisposableBean {
      @Override
      public void destroy() {
          // do some destruction work (like releasing pooled connections)
      }
  }
  ```

  

- 推荐使用` @PreDestroy`注解或者声明一个方法`destroy-method`

  ```xml
  <bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
  ```

  ```java
  public class ExampleBean {
      public void cleanup() {
          // do some destruction work (like releasing pooled connections)
      }
  }
  ```

  

**默认初始化方法和销毁方法**

统一使用`init()`做初始化方法，`destroy()`做销毁方法，就不用像上面那样每个bean配一个方法

```java
public class DefaultBlogService implements BlogService {
    private BlogDao blogDao;
    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }
    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```xml
<beans default-init-method="init">
    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>
</beans>
```



### 1.7.bean定义的继承





一个bean的定义配置信息包括构造参数、属性、容器定制信息（初始化方法、静态工厂方法名）等等。子bean可以继承父的定义，而且可以重写

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```



### 1.8.容器扩展



#### 1.8.1.自定义beans

使用`BeanPostProcessor`接口定义回调方法，可以操作容器实例逻辑



#### 1.8.2.自定义配置元数据

使用`BeanFactoryPostProcessor`，能够在容器实例化bean之前修改配置元数据



#### 1.8.3.自定义实例逻辑

`org.springframework.beans.factory.FactoryBean`接口 可以操作IoC容器实例逻辑

`FactoryBean`接口方法：

- `Object getObject()`获取工厂创建的实例
- `boolean isSingleton()`



### 1.9.基于注解的容器配置



#### 1.9.1.@Required



用在Setter，表示在配置的时候必须要有值

```java
public class SimpleMovieLister {
    private MovieFinder movieFinder;
    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    // ...
}
```



#### 1.9.2.@Autowired

- 用在构造器

  ```java
  public class MovieRecommender {
      private final CustomerPreferenceDao customerPreferenceDao;
      @Autowired
      public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
          this.customerPreferenceDao = customerPreferenceDao;
      }
  }
  ```

- 用在Setter

  ```java
  public class SimpleMovieLister {
      private MovieFinder movieFinder;
      @Autowired
      public void setMovieFinder(MovieFinder movieFinder) {
          this.movieFinder = movieFinder;
      }
  }
  ```



- 随便一个方法

  ```java
  public class MovieRecommender {
      private MovieCatalog movieCatalog;
      private CustomerPreferenceDao customerPreferenceDao;
      @Autowired
      public void prepare(MovieCatalog movieCatalog,
              CustomerPreferenceDao customerPreferenceDao) {
          this.movieCatalog = movieCatalog;
          this.customerPreferenceDao = customerPreferenceDao;
      }
  }
  ```

- 同时用在变量和构造函数

  ```java
  public class MovieRecommender {
      private final CustomerPreferenceDao customerPreferenceDao;
      @Autowired
      private MovieCatalog movieCatalog;
      @Autowired
      public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
          this.customerPreferenceDao = customerPreferenceDao;
      }
  }
  ```

  

#### 1.9.3@Primary



#### 1.9.4.@Qualifier

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}

```



#### 1.9.6.Using `CustomAutowireConfigurer`



#### 1.9.7.@Resource



#### 1.9.8.@Value

用在参数

```java
@Component
public class MovieRecommender {
    private final String catalog;
    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```



读取配置

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```



配置

```java
catalog.name=MovieCatalog
```



#### 1.9.9.@PostContruct & @PreDestroy

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```



### 1.10.扫描和组件管理



#### 1.10.1.@Component

#### 1.10.3. 自动检测class

具备检测能力

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```



标有注解`@Service`/`@Repository`才能被检测出来



#### 1.10.4.使用filter自定义扫描

类的默认注解有 @Component, @Repository, @Service, @Controller, @Configuration



可以自定义filter来实现@Component这些注解同样功能



filter：

- includeFilters
- excludeFilters



注解实现

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```



xml实现

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```



#### 1.10.5.在组件定义bean元数据

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```



#### 1.10.6.命名自动检测组件

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```



#### 1.10.7.给组件提供scrope

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```



### 1.12.基于java容器配置

#### 1.12.1.@Bean and @Configuration

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

相当于

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```



`@Bean`一般出现在`@Configuration`类下，确保full mode



#### 1.12.2.实例化容器

`AnnotationConfigApplicationContext`

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```



register写法

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```



**组件扫描**

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```



```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```



#### 1.12.3.使用@Bean注解









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