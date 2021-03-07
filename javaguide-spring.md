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

![Spring Bean life cycle](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/48376272.jpg)

A similar Chinese version:

![Spring Bean life cycle](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)

## 6. Spring MVC

### 6.1 Tell me about your understanding of Spring MVC?

When it comes to this issue, we have to mention the previous Model1 and Model2 eras without Spring MVC.

- **Model1 era** : Many friends who learn Java back-end late may not have been exposed to JavaWeb application development in Model1 mode. In Model1 mode, the entire Web application is almost entirely composed of JSP pages, and only a few JavaBeans are used to handle database connection and access operations. In this mode, JSP is both the control layer and the presentation layer. Obviously, there are many problems with this model. For example, ① The control logic and performance logic are mixed together, resulting in extremely low code reuse rate; ② The front-end and back-end depend on each other, it is difficult to test and the development efficiency is extremely low;
- **Model2 era** ：Friends who have studied Servlet and have done related Demos should understand the development model of "Java Bean (Model) + JSP (View,) + Servlet (Controller)", which is the early JavaWeb MVC development model. Model: The data involved in the system, namely dao and bean. View: Display the data in the model, just for display. Controller: Process user requests and send them to, return data to JSP and display it to users.

There are still many problems in Model2 mode. The degree of abstraction and encapsulation of Model2 is far from enough. When using Model2 for development, wheels will inevitably be reinvented, which greatly reduces the maintainability and reusability of the program. So many JavaWeb development-related MVC frameworks emerged such as Struts2, but Struts2 is relatively cumbersome. With the popularity of the Spring lightweight development framework, the Spring MVC framework appeared in the Spring ecosystem. Spring MVC is currently the best MVC framework. Compared with Struts2, Spring MVC is simpler and more convenient to use, with higher development efficiency, and Spring MVC runs faster.

MVC is a design pattern, and Spring MVC is an excellent MVC framework. Spring MVC can help us develop a more concise Web layer, and it is naturally integrated with the Spring framework. Under Spring MVC, we generally divide back-end projects into Service layer (processing business), Dao layer (database operations), Entity layer (entity class), Controller layer (control layer, returning data to the foreground page).

**The simple schematic diagram of Spring MVC is as follows:**

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)

### 6.2 Do you understand the working principle of SpringMVC?

**The principle is shown in the figure below:**
![SpringMVC operating principle](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

A small problem with a clerical error in the above figure: The entry function of Spring MVC, which is the front controller `DispatcherServlet`, is to receive requests and respond to results.

**Process description (important):**

1. The client (browser) sends a request directly to the DispatcherServlet.
2. `DispatcherServlet` calls `HandlerMapping` according to the request information, and analyzes the `Handler` corresponding to the request.
3. After parsing to the corresponding `Handler` (that is, the `Controller` controller we usually call), it starts to be processed by the `HandlerAdapter` adapter.
4. `HandlerAdapter` will call the real handler according to `Handler` to process the request and process the corresponding business logic.
5. After the processor finishes processing the business, it will return a `ModelAndView` object, `Model` is the returned data object, and `View` is a logical `View`.
6. `ViewResolver` will find the actual `View` according to the logic `View`.
7. `DispaterServlet` passes the returned `Model` to `View` (view rendering).
8. Return `View` to requester (browser)

## 7. What design patterns are used in the Spring framework?

For a detailed introduction to some of the following design patterns, you can read the original article of the author some time ago[《Interviewer: "Talk about those design patterns used in Spring?".》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd) 。

- **Factory design pattern**: Spring uses the factory pattern to create bean objects through `BeanFactory` and `ApplicationContext`.
- **Agency design pattern**: Implementation of Spring AOP function.
- **Singleton design pattern**: Beans in Spring are singletons by default.
- **Template method pattern**: Spring's `jdbcTemplate`, `hibernateTemplate` and other classes that end with Template for database operations, they use the template mode.
- **Wrapper design pattern**: Our project needs to connect to multiple databases, and different customers will visit different databases according to their needs during each visit. This model allows us to dynamically switch between different data sources according to customer needs.
- **Observer mode**: The Spring event-driven model is a classic application of the observer pattern.
- **Adapter mode**: The enhancement or advice of Spring AOP uses the adapter pattern, and spring MVC also uses the adapter pattern adaptation `Controller`.
- ......

## 8. Spring transaction

### 8.1 How many ways does Spring manage transactions?

1. Programmatic transactions, hard-coded in the code. (Not recommended)
2. Declarative transaction, configured in the configuration file (recommended)

**Declarative transactions are divided into two types:**

1. XML-based declarative transaction
2. Annotation-based declarative transaction

### 8.2 What are the isolation levels in Spring transactions?

**Five constants representing isolation levels are defined in the TransactionDefinition interface:**

- **TransactionDefinition.ISOLATION_DEFAULT:**  Use the default isolation level of the back-end database, the default REPEATABLE_READ isolation level used by Mysql, the default READ_COMMITTED isolation level used by Oracle.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** The lowest isolation level, allowing to read data changes that have not yet been committed, **may cause dirty reads, phantom reads or non-repeatable reads**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:**   Allows to read data that has been committed by concurrent transactions, **can prevent dirty reads, but phantom reads or non-repeatable reads may still occur**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:**  The results of multiple reads of the same field are consistent, unless the data is modified by the transaction itself, ** can prevent dirty reads and non-repeatable reads, but phantom reads may still occur. **
- **TransactionDefinition.ISOLATION_SERIALIZABLE:**   The highest isolation level, fully obeys the ACID isolation level. All transactions are executed one by one, so that there is no interference between transactions, that is to say, **this level can prevent dirty reads, non-repeatable reads, and phantom reads**. But this will seriously affect the performance of the program. Normally, this level is not used either.

### 8.3 What kinds of transaction propagation behaviors in Spring transactions?

**Support current affairs:**

- **TransactionDefinition.PROPAGATION_REQUIRED：** If there is a current transaction, then join the transaction; if there is no current transaction, then create a new transaction.
- **TransactionDefinition.PROPAGATION_SUPPORTS：** If there is currently a transaction, then join the transaction; if there is no current transaction, continue to run in a non-transactional manner.
- **TransactionDefinition.PROPAGATION_MANDATORY：** If there is currently a transaction, then join the transaction; if there is no current transaction, an exception is thrown. (Mandatory: mandatory)

**The situation where the current transaction is not supported:**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** Create a new transaction, if there is a transaction currently, then suspend the current transaction.
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** Run in non-transactional mode. If there is a transaction currently, the current transaction is suspended.
- **TransactionDefinition.PROPAGATION_NEVER：** Run in non-transactional mode. If there is a transaction currently, an exception is thrown.

**Other situations:**

- **TransactionDefinition.PROPAGATION_NESTED：** If there is a transaction currently, a transaction is created to run as a nested transaction of the current transaction; if there is no transaction currently, the value is equivalent to TransactionDefinition.PROPAGATION_REQUIRED.

### 8.4 @Transactional(rollbackFor = Exception.class) annotation understand?

We know: Exception is divided into runtime exception RuntimeException and non-runtime exception. Transaction management is very important for enterprise applications. Even if abnormal conditions occur, it can also ensure data consistency.

When the `@Transactional` annotation is applied to a class, all public methods of the class will have the transaction attribute of this type. At the same time, we can also use this annotation at the method level to override the class-level definition. If this annotation is added to the class or method, then the method in this class throws an exception, it will be rolled back, and the data in the database will also be rolled back.

If the `rollbackFor` property is not configured in the `@Transactional` annotation, then the transaction will only be rolled back when it encounters `RuntimeException`, plus `rollbackFor=Exception.class`, you can make the transaction encounter non-running It also rolls back when it is abnormal.

Recommended reading articles about `@Transactional` annotation:

- [Thoroughly grasp the use of @transactional in Spring](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)

## 9. JPA

### 9.1 How to use JPA to non-persistent a field in the database?

If we have the following class:

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

What if we want the `secrect` field not to be persisted, that is, not to be stored in the database? We can use the following methods:

```java
static String transient1; // not persistent because of static
final String transient2 = “Satish”; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

Generally, the latter two methods are used more often, and I personally use annotations more.


## Reference

- 《Spring technology insider》
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
