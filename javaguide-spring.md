This article is mainly to pass some questions to deepen everyone's understanding of Spring, so there is not much code involved! This article has been organized for a long time, and I have not paid attention to many of the following problems myself in the process of using Spring. I also temporarily checked many materials and books to make up. There are also many articles on Spring FAQ/interview questions on the Internet. I feel that most of them are copying each other, and many of the questions are not very good, and some answers have problems. So, I spent a week of spare time sorting it out, hoping to help everyone.

## 1. What is the Spring framework?

Spring is a lightweight development framework designed to improve the development efficiency of developers and the maintainability of the system. Spring official website: <https://spring.io/>.

We generally say that the Spring framework refers to the Spring Framework, which is a collection of many modules. Using these modules can help us develop easily. These modules are: core container, data access/integration, Web, AOP (Aspect Oriented Programming), tools, messaging and testing modules. For example, the Core component in the Core Container is the core of all Spring components, the Beans component and the Context component are the basis for implementing IOC and dependency injection, and the AOP component is used to implement aspect-oriented programming.

The 6 features of Spring listed on Spring's official website:

- **Core technologies**: Dependency Injection (DI), AOP, events, resources, i18n, validation, data binding, type conversion, SpEL.
- **Testing**: Simulation object, TestContext framework, Spring MVC test, WebTestClient.
- **Data access**: Transaction, DAO support, JDBC, ORM, marshalling XML.
- **Web support**: Spring MVC and Spring WebFlux web framework.
- **Integration**: Remote processing, JMS, JCA, JMX, email, tasks, scheduling, caching.
- **Language**: Kotlin, Groovy, dynamic languages.

## 2. List some important Spring modules?

The figure below corresponds to the Spring 4.x version. At present, the Portlet component of the Web module in the latest 5.x version has been abandoned, and the WebFlux component for asynchronous response processing has been added.

![Spring main modules](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Spring主要模块.png)

- **Spring Core：** Basically, it can be said that all other functions of Spring need to rely on this class library. Mainly provide IoC dependency injection function.
- **Spring  Aspects**: This module provides support for integration with AspectJ.
- **Spring AOP**: Provides an aspect-oriented programming implementation.
- **Spring JDBC**: Java database connection.
- **Spring JMS**: Java messaging service.
- **Spring ORM**: Used to support ORM tools such as Hibernate.
- **Spring Web**: Provide support for creating web applications.
- **Spring Test**: Provides support for JUnit and TestNG testing.

## 3. @RestController vs @Controller

**`Controller` return to a page**

Using `@Controller` alone without adding `@ResponseBody` is generally used when you want to return a view. This situation is a more traditional Spring MVC application, which corresponds to the situation where the front and back ends are not separated.

![SpringMVC traditional workflow](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringMVC传统工作流程.png)

**`@RestController` return data in JSON or XML format**

But `@RestController` only returns objects, and the object data is directly written into the HTTP response (Response) in the form of JSON or XML. This situation is a RESTful Web service, which is also the most commonly used situation in daily development (front-end and back-end separation).

![SpringMVC+RestController](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringMVCRestController.png)

**`@Controller +@ResponseBody` return data in JSON or XML format**

If you need to develop RESTful Web services before Spring 4, you need to use `@Controller` combined with the annotation of `@ResponseBody`, that is to say `@Controller` +`@ResponseBody` = `@RestController` (added after Spring 4 Notes).

> The function of the `@ResponseBody` annotation is to convert the object returned by the `Controller` method into a specified format through an appropriate converter, and then write it into the body of the HTTP response (Response) object, usually used to return JSON or XML data , there are many cases of returning JSON data.

![Spring3.MVC RESTful web service workflow](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/Spring3.xMVCRESTfulWeb服务工作流程.png)

Reference:

- https://dzone.com/articles/spring-framework-restcontroller-vs-controller （Image Source）
- https://javarevisited.blogspot.com/2017/08/difference-between-restcontroller-and-controller-annotations-spring-mvc-rest.html?m=1

## 4. Spring IOC & AOP

### 4.1 Talk about your understanding of Spring IoC and AOP

#### IoC

IoC (Inverse of Control: Inversion of Control) is a **design idea**，It is **to transfer the control of objects created manually in the program to the Spring framework to manage.** IoC is also used in other languages and is not unique to Spring. **The IoC container is the carrier used by Spring to implement IoC. The IoC container is actually a Map (key, value), and various objects are stored in the Map.**

The interdependency between objects is handed over to the IoC container to manage, and the IoC container completes the injection of the objects. This simplifies application development to a large extent and liberates applications from complex dependencies.  **The IoC container is like a factory. When we need to create an object, we only need to configure the configuration file/annotation, without considering how the object is created.** In actual projects, a Service class may have hundreds or even thousands of classes as its bottom layer. If we need to instantiate this Service, you may have to figure out the constructors of all the bottom classes of this Service every time. Drive people crazy. If you use IoC, you only need to configure it, and then reference it where necessary, which greatly increases the maintainability of the project and reduces the difficulty of development.

In the Spring era, we generally used XML files to configure Beans. Later, developers felt that XML files were not good for configuration, so SpringBoot annotation configuration slowly became popular.

Recommended reading: https://www.zhihu.com/question/23277575/answer/169698662

**The initialization process of Spring IoC：** 

![The initialization process of Spring IoC](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringIOC初始化过程.png)

IoC source code reading

- https://javadoop.com/post/spring-ioc

#### AOP

AOP (Aspect-Oriented Programming: Aspect-Oriented Programming) can encapsulate logic or responsibilities (such as transaction processing, log management, authority control, etc.) that are not related to the business but are commonly called by business modules, so as to facilitate **Reduce system code duplication**, **reduce the coupling degree between modules**, and **facilitate future scalability and maintainability**.

**Spring AOP is based on dynamic proxy**, if the object to be proxy implements a certain interface, then Spring AOP will use **JDK Proxy** to create proxy objects, and for objects that do not implement the interface, JDK Proxy cannot be used for proxying. At this time, Spring AOP will use **Cglib** to generate a subclass of the proxy object as a proxy, as shown in the following figure:

![SpringAOPProcess](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/SpringAOPProcess.jpg)

Of course you can also use AspectJ, Spring AOP has integrated AspectJ, AspectJ should be regarded as the most complete AOP framework in the Java ecosystem.

After using AOP, we can abstract some common functions and use them directly where they are needed, which greatly simplifies the amount of code. It is also convenient when we need to add new functions, which also improves system scalability. AOP is used in scenarios such as log function and transaction management.

### 4.2 What is the difference between Spring AOP and AspectJ AOP？

**Spring AOP is a runtime enhancement, while AspectJ is a compile-time enhancement.** Spring AOP is based on proxy (proxying), while AspectJ is based on bytecode operations (bytecode Manipulation)

Spring AOP has integrated AspectJ, and AspectJ should be regarded as the most complete AOP framework in the Java ecosystem. AspectJ is more powerful than Spring AOP, but Spring AOP is relatively simpler.

If we have fewer aspects, there is little difference in performance between the two. However, when there are too many aspects, it is best to choose AspectJ, which is much faster than Spring AOP.

## 5. Spring bean

### 5.1 What are the scopes of beans in Spring?

- singleton : The only bean instance, the bean in Spring is singleton by default.
- prototype : Each request will create a new bean instance.
- request : Each HTTP request will generate a new bean, which is only valid in the current HTTP request.
- session : Each HTTP request will generate a new bean, which is only valid in the current HTTP session.
- global-session： The global session scope is only meaningful in portlet-based web applications, which is no longer available in Spring 5. Portlet is a small Java Web plug-in that can generate fragments of semantic code (for example: HTML). They are based on portlet containers and can handle HTTP requests like servlets. However, unlike servlets, each portlet has a different session

### 5.2 Do you understand the thread safety of singleton beans in Spring?

There is indeed a security problem. Because, when multiple threads operate the same object, there will be thread safety issues in the write operation of the member variables of this object.

However, in general, the commonly used beans such as `Controller`, `Service`, and `Dao` are stateless. Stateless beans cannot save data and are therefore thread-safe.

There are two common solutions:

2. Define a `ThreadLocal` member variable in the class, and save the required variable member variables in `ThreadLocal` (a recommended way).
2. Change the scope of Bean to "prototype": each request will create a new bean instance, naturally there will be no thread safety issues.


### 5.3 What is the difference between @Component and @Bean?

1. The target is different: `@Component` annotation acts on the class, while `@Bean` annotation acts on the method.
2. `@Component` is usually automatically detected and automatically assembled into the Spring container through classpath scanning (we can use the `@ComponentScan` annotation to define the path to be scanned and find out the beans that identify the classes that need to be assembled and automatically assembled to Spring Container). The `@Bean` annotation is usually defined by us in the method marked with the annotation to generate the bean. `@Bean` tells Spring that this is an example of a certain class, and it will be returned to me when I need it.
3. The `@Bean` annotation is more customizable than the `Component` annotation, and in many places we can only register beans through the `@Bean` annotation. For example, when we reference classes in third-party libraries that need to be assembled into the `Spring` container, it can only be achieved through `@Bean`.

Example of using `@Bean` annotation:

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

The above code is equivalent to the following xml configuration

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

The following example is not possible with `@Component`.

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### 5.4 What are the annotations for declaring a class as a Spring bean?

We generally use the `@Autowired` annotation to autowire beans. To identify a class as a class that can be used for the `@Autowired` annotation autowired bean, use the following annotations to achieve:

- `@Component` : General annotation, you can mark any class as a `Spring` component. If a Bean does not know which layer it belongs to, it can be annotated with `@Component`.
- `@Repository` : The corresponding persistence layer is the Dao layer, which is mainly used for database-related operations.
- `@Service` : Corresponding to the service layer, it mainly involves some complex logic, which requires the Dao layer.
- `@Controller` : Corresponding to the Spring MVC control layer, it is mainly used to accept user requests and call the Service layer to return data to the front-end page.

### 5.5 Bean life cycle in Spring?

There are many articles on this part of the Internet, the following content is organized from: <https://yemengying.com/2016/07/14/spring-bean-life-cycle/>，In addition to this article, I recommend another very good article: <https://www.cnblogs.com/zrtqsk/p/3735273.html>.

- The Bean container finds the definition of Spring Bean in the configuration file.
- The Bean container uses the Java Reflection API to create an instance of the Bean.
- If some attribute values are involved, use the `set()` method to set some attribute values.
- If the Bean implements the `BeanNameAware` interface, call the `setBeanName()` method and pass in the Bean's name.
- If the Bean implements the `BeanClassLoaderAware` interface, call the `setBeanClassLoader()` method and pass in the instance of the `ClassLoader` object.
- Similar to the above, if other `*.Aware` interfaces are implemented, the corresponding method is called.
- If there is a `BeanPostProcessor` object related to the Spring container that loaded this Bean, execute the `postProcessBeforeInitialization()` method
- If the Bean implements the `InitializingBean` interface, execute the `afterPropertiesSet()` method.
- If the definition of the Bean in the configuration file contains the init-method attribute, execute the specified method.
- If there is a `BeanPostProcessor` object related to the Spring container that loaded this Bean, execute the `postProcessAfterInitialization()` method
- When the bean is to be destroyed, if the bean implements the `DisposableBean` interface, execute the `destroy()` method.
- When the bean is to be destroyed, if the definition of the bean in the configuration file contains the destroy-method attribute, execute the specified method.

Icon:

![Spring Bean life cycle](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/48376272.jpg)

A similar Chinese version:

![Spring Bean life cycle](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)

## 6. Spring MVC

### 6.1 说说自己对于 Spring MVC 了解?

谈到这个问题，我们不得不提提之前 Model1 和 Model2 这两个没有 Spring MVC 的时代。

- **Model1 时代** : 很多学 Java 后端比较晚的朋友可能并没有接触过  Model1 模式下的 JavaWeb 应用开发。在 Model1 模式下，整个 Web 应用几乎全部用 JSP 页面组成，只用少量的 JavaBean 来处理数据库连接、访问等操作。这个模式下 JSP 既是控制层又是表现层。显而易见，这种模式存在很多问题。比如①将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低；②前端和后端相互依赖，难以进行测试并且开发效率极低；
- **Model2 时代** ：学过 Servlet 并做过相关 Demo 的朋友应该了解“Java Bean(Model)+ JSP（View,）+Servlet（Controller）  ”这种开发模式,这就是早期的 JavaWeb MVC 开发模式。Model:系统涉及的数据，也就是 dao 和 bean。View：展示模型中的数据，只是用来展示。Controller：处理用户请求都发送给 ，返回数据给 JSP 并展示给用户。

Model2 模式下还存在很多问题，Model2的抽象和封装程度还远远不够，使用Model2进行开发时不可避免地会重复造轮子，这就大大降低了程序的可维护性和复用性。于是很多JavaWeb开发相关的 MVC 框架应运而生比如Struts2，但是 Struts2 比较笨重。随着 Spring 轻量级开发框架的流行，Spring 生态圈出现了 Spring MVC 框架， Spring MVC 是当前最优秀的 MVC 框架。相比于 Struts2 ， Spring MVC 使用更加简单和方便，开发效率更高，并且 Spring MVC 运行速度更快。

MVC 是一种设计模式,Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的Web层的开发，并且它天生与 Spring 框架集成。Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。

**Spring MVC 的简单原理图如下：**

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)

### 6.2 SpringMVC 工作原理了解吗?

**原理如下图所示：**
![SpringMVC运行原理](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

上图的一个笔误的小问题：Spring MVC 的入口函数也就是前端控制器 `DispatcherServlet` 的作用是接收请求，响应结果。

**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

## 7. Spring 框架中用到了哪些设计模式？

关于下面一些设计模式的详细介绍，可以看笔主前段时间的原创文章[《面试官:“谈谈Spring中都用到了那些设计模式?”。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd) 。

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......

## 8. Spring 事务

### 8.1 Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

### 8.2 Spring 事务中的隔离级别有哪几种?

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:**  使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:**   允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:**  对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:**   最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 8.3 Spring 事务中哪几种事务传播行为?

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 8.4 @Transactional(rollbackFor = Exception.class)注解了解吗？

我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚。

关于 `@Transactional ` 注解推荐阅读的文章：

- [透彻的掌握 Spring 中@transactional 的使用](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)

## 9. JPA

### 9.1 如何使用JPA在数据库中非持久化一个字段？

假如我们有有下面一个类：

```java
Entity(name="USER")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private Long id;
    
    @Column(name="USER_NAME")
    private String userName;
    
    @Column(name="PASSWORD")
    private String password;
  
    private String secrect;
  
}
```

如果我们想让`secrect` 这个字段不被持久化，也就是不被数据库存储怎么办？我们可以采用下面几种方法：

```java
static String transient1; // not persistent because of static
final String transient2 = “Satish”; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

一般使用后面两种方式比较多，我个人使用注解的方式比较多。


## 参考

- 《Spring 技术内幕》
- <http://www.cnblogs.com/wmyskxz/p/8820371.html>
- <https://www.journaldev.com/2696/spring-interview-questions-and-answers>
- <https://www.edureka.co/blog/interview-questions/spring-interview-questions/>
- https://www.cnblogs.com/clwydjgs/p/9317849.html
- <https://howtodoinjava.com/interview-questions/top-spring-interview-questions-with-answers/>
- <http://www.tomaszezula.com/2014/02/09/spring-series-part-5-component-vs-bean/>
- <https://stackoverflow.com/questions/34172888/difference-between-bean-and-autowired>

## 公众号

如果大家想要实时关注我更新的文章以及分享的干货的话，可以关注我的公众号。

**《Java面试突击》:** 由本文档衍生的专为面试而生的《Java面试突击》V2.0 PDF 版本[公众号](#公众号)后台回复 **"Java面试突击"** 即可免费领取！

**Java工程师必备学习资源:** 一些Java工程师常用学习资源公众号后台回复关键字 **“1”** 即可免费无套路获取。 

![公众号](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/javaguide1.jpg)
