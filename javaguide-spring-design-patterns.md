点击关注[公众号](#公众号)及时获取笔主最新更新文章，并可免费领取本文档配套的《Java面试突击》以及Java工程师必备学习资源。

<!-- TOC -->

- [Inversion of Control (IoC) and Dependency Injection (DI)](#inversion-of-control-(ioc)-and-dependency-injection-(di))
- [Factory design pattern](#factory-design-pattern)
- [Singleton design pattern](#singleton-design-pattern)
- [Agency design pattern](#agency-design-pattern)
    - [Application of Proxy Mode in AOP](#application-of-proxy-mode-in-aop)
    - [What is the difference between Spring AOP and AspectJ AOP?](#what-is-the-difference-between-spring-aop-and-aspectj-aop)
- [Template method design pattern](#template-method-design-pattern)
- [Observer design pattern](#observer-design-pattern)
    - [Three roles in the Spring event-driven model](#three-roles-in-the-spring-event-driven-model)
        - [Event role](#event-role)
        - [Event listener role](#event-listener-role)
        - [Event publisher role](#event-publisher-role)
    - [Summary of Spring's event flow](#summary-of-spring's-event-flow)
- [Adapter design pattern](#adapter-design-pattern)
    - [Adapter design pattern in spring AOP](#adapter-design-pattern-in-spring-aop)
    - [Adapter design pattern in spring MVC](#adapter-design-pattern-in-spring-mvc)
- [Decorator design pattern](#decorator-design-pattern)
- [Summary](#Summary)
- [Reference](#Reference)

<!-- /TOC -->

Which design patterns are used in JDK? Which design patterns are used in Spring? These two questions are more common in interviews. I searched the Internet about the design patterns in Spring. The explanations are almost the same, and most of them are old. So, it took a few days to summarize it myself. As my personal ability is limited, you can point out any mistakes in the article. In addition, the length of the article is limited. I just gave a brief introduction to the interpretation of design patterns and some source code. The main purpose of this article is to review the design patterns in Spring.

Design Patterns represent the best computer programming practices in object-oriented software development. Different types of design patterns are widely used in the Spring framework. Let's take a look at what design patterns are there?

## Inversion of Control (IoC) and Dependency Injection (DI)

**IoC (Inversion of Control)** It is a very, very important concept in Spring. It is not a technology, but a decoupling design idea. Its main purpose is to use the "third party" (IOC container in Spring) to achieve decoupling between objects that have dependencies (IOC container manages objects, you can just use them), thereby reducing the degree of coupling between codes. **IOC is a principle, not a pattern. The following patterns (but not limited to) implement the IoC principle.**

![ioc-patterns](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ioc-patterns.png)

**The Spring IOC container is like a factory. When we need to create an object, we only need to configure the configuration file/annotation, without considering how the object is created.** The IOC container is responsible for creating objects, connecting objects together, configuring these objects, and processing the entire life cycle of these objects from creation until they are completely destroyed.

In an actual project, if a Service class has hundreds or even thousands of classes as its bottom layer, we need to instantiate the Service. You may have to figure out the constructors of all the bottom-level classes of this Service every time, which may change People are driving crazy. If you use IOC, you only need to configure it, and then reference it where needed, which greatly increases the maintainability of the project and reduces the difficulty of development. Regarding the understanding of Spring IOC, I recommend reading this answer from Zhihu: <https://www.zhihu.com/question/23277575/answer/169698662>, which is very good.

**How to understand inversion of control?** For example: "Object A depends on object B. When object A needs to use object B, you must create it yourself. But when the system introduces the IOC container, object A and object B have lost direct contact. At this time when the object A needs to use the object B, we can specify the IOC container to create an object B to inject into the object A". The process in which object A obtains dependent object B changes from active behavior to passive behavior, and the right to control is reversed. This is the origin of the name of control reversal.

**DI (Dependecy Inject) is a design pattern for achieve inversion of control. Dependency injection is to pass instance variables into an object.**

## Factory design pattern

Spring uses the factory pattern to create bean objects through `BeanFactory` or `ApplicationContext`.

**Comparison of the two:**

-  `BeanFactory`: Delayed injection (injects only when a bean is used). Compared with `ApplicationContext`, it takes up less memory and the program starts faster.
- `ApplicationContext`: When the container starts, whether you use it or not, create all beans at once. `BeanFactory` only provides the most basic dependency injection support, `ApplicationContext` extends `BeanFactory`, in addition to the function of `BeanFactory`, there are more additional functions, so general developers will use `ApplicationContext` more.

Three implementation classes of ApplicationContext:

1. `ClassPathXmlApplication`: Treat the context file as a class path resource.
2. `FileSystemXmlApplication`: Load the context definition information from the XML file in the file system.
3. `XmlWebApplicationContext`: Load context definition information from XML files in the Web system.

Example:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
 
public class App {
	public static void main(String[] args) {
		ApplicationContext context = new FileSystemXmlApplicationContext(
				"C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");
 
		HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
		obj.getMsg();
	}
}
```

## Singleton design pattern

In our system, there are some objects that we actually only need one, such as: thread pool, cache, dialog box, registry, log object, and objects that act as device drivers such as printers and graphics cards. In fact, this type of object can only have one instance. If multiple instances are created, some problems may occur, such as: abnormal program behavior, excessive resource usage, or inconsistent results.

**The benefits of using the singleton pattern:**

- For frequently used objects, the time spent creating objects can be omitted, which is a very considerable system overhead for those heavyweight objects;
- As the number of new operations is reduced, the frequency of using system memory will also be reduced, which will reduce GC pressure and shorten GC pause time.

**The default scope of beans in Spring is singleton (singleton).** In addition to singleton scope, beans in Spring also have the following scopes:

- prototype: Each request will create a new bean instance.
- request: Each HTTP request will generate a new bean, which is only valid in the current HTTP request.
- session: Each HTTP request will generate a new bean, which is only valid in the current HTTP session.
- global-session: Global session scope, only meaningful in portlet-based web applications, Spring 5 is no longer there. Portlet is a small Java Web plug-in that can generate fragments of semantic code (for example: HTML). They are based on portlet containers and can handle HTTP requests like servlets. However, unlike servlets, each portlet has a different session.

design pattern**The way Spring implements a singleton:**

- xml: `<bean id="userService" class="top.snailclimb.UserService" scope="singleton"/>`
- annotation: `@Scope(value = "singleton")`

**Spring implements the singleton mode through the special way of `ConcurrentHashMap` to implement the singleton registry. The core code for Spring to implement a singleton is as follows**

```java
// Realize singleton registry through ConcurrentHashMap (thread safety)
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // Check if there is an instance in the cache  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...A lot of code omitted
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...A lot of code omitted
                // If the instance object does not exist, we register it in the singleton registry.
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //Add objects to the singleton registry
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

## Agency design pattern

### Application of Proxy Mode in AOP

AOP (Aspect-Oriented Programming) be able to take those that have nothing to do with the business，**but it is encapsulated by the logic or responsibilities (such as transaction processing, log management, permission control, etc.) called by the business modules**，it is convenient to **reduce the repetitive code** of the system, **reduce the degree of coupling between modules**, and **is conducive to future scalability and maintainability**.

**Spring AOP is based on dynamic proxy**，If the object to be proxied implements a certain interface, then Spring AOP will use **JDK Proxy** to create proxy objects, and for objects that do not implement the interface, JDK Proxy cannot be used for proxying. At this time, Spring AOP will use **Cglib** to generate a subclass of the proxy object as a proxy, as shown in the following figure:

![SpringAOPProcess](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/SpringAOPProcess.jpg)

Of course you can also use AspectJ, Spring AOP has integrated AspectJ, AspectJ should be regarded as the most complete AOP framework in the Java ecosystem.

After using AOP, we can abstract some common functions and use them directly where they are needed, which greatly simplifies the amount of code. It is also convenient when we need to add new functions, which also improves system scalability. AOP is used in scenarios such as log function, transaction management, and so on.

### What is the difference between Spring AOP and AspectJ AOP?

**Spring AOP is a runtime enhancement, while AspectJ is a compile-time enhancement.** Spring AOP is based on Proxying, while AspectJ is based on Bytecode Manipulation.

Spring AOP has integrated AspectJ, and AspectJ should be regarded as the most complete AOP framework in the Java ecosystem. AspectJ is more powerful than Spring AOP, but Spring AOP is relatively simpler.

If we have fewer aspects, there is little difference in performance between the two. However, when there are too many aspects, it is best to choose AspectJ, which is much faster than Spring AOP.

## Template method design pattern

The template method pattern is a behavioral design pattern that defines the skeleton of an algorithm in operation, while delaying some steps to subclasses. The template method allows subclasses to redefine the implementation of certain specific steps of an algorithm without changing the structure of an algorithm.

```java
public abstract class Template {
    //This is our template method
    public final void TemplateMethod(){
        PrimitiveOperation1();  
        PrimitiveOperation2();
        PrimitiveOperation3();
    }

    protected void  PrimitiveOperation1(){
        //Current class implementation
    }
    
    //Methods implemented by the quilt class
    protected abstract void PrimitiveOperation2();
    protected abstract void PrimitiveOperation3();

}
public class TemplateImpl extends Template {

    @Override
    public void PrimitiveOperation2() {
        //Current class implementation
    }
    
    @Override
    public void PrimitiveOperation3() {
        //Current class implementation
    }
}

```

Spring's `jdbcTemplate`, `hibernateTemplate` and other classes that end with Template for database operations, they use the template mode. Under normal circumstances, we all use inheritance to implement the template mode, but Spring does not use this method, but uses the Callback mode in conjunction with the template method mode, which not only achieves the effect of code reuse, but also increases flexibility.

## Observer design pattern

The observer pattern is an object behavior pattern. It represents a dependency between an object and an object. When an object changes, the objects that the object depends on will also react. The Spring event-driven model is a classic application of the observer pattern. The Spring event-driven model is very useful and can decouple our code in many scenarios. For example, we need to update the product index every time we add a product. At this time, we can use the observer mode to solve this problem.

### Three roles in the Spring event-driven model

#### Event role

 `ApplicationEvent` (under the `org.springframework.context` package) acts as an event. This is an abstract class that inherits `java.util.EventObject` and implements the `java.io.Serializable` interface.

The following events exist by default in Spring, and they are all implementations of `ApplicationContextEvent` (inherited from `ApplicationContextEvent`):

- `ContextStartedEvent`: Event triggered after `ApplicationContext` is started;
- `ContextStoppedEvent`: Event triggered after `ApplicationContext` is stopped;
- `ContextRefreshedEvent`: Event triggered after `ApplicationContext` is initialized or refreshed;
- `ContextClosedEvent`: Event triggered after `ApplicationContext` is closed.

![ApplicationEvent-Subclass](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ApplicationEvent-Subclass.png)

#### Event listener role

`ApplicationListener` plays the role of event listener. It is an interface in which only one `onApplicationEvent()` method is defined to handle `ApplicationEvent`. The source code of the `ApplicationListener` interface class is as follows. It can be seen from the interface definition that the events in the interface only need to implement `ApplicationEvent`. Therefore, in Spring, we only need to implement the `onApplicationEvent()` method of the `ApplicationListener` interface to complete the monitoring event.

```java
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

#### Event publisher role

`ApplicationEventPublisher` acts as the publisher of the event, and it is also an interface.

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}

```

The `publishEvent()` method of the `ApplicationEventPublisher` interface is implemented in the `AbstractApplicationContext` class. Reading the implementation of this method, you will find that the event is actually broadcasted through the `ApplicationEventMulticaster`. The specific content is too much, so I will not analyze it here, and I may write an article to mention it later.

### Summary of Spring's event flow

1. Define an event: implement an inherited from `ApplicationEvent`, and write the corresponding constructor;
2. Define an event listener: implement the `ApplicationListener` interface and override the `onApplicationEvent()` method;
3. Use the event publisher to publish messages: You can publish messages through the `publishEvent()` method of `ApplicationEventPublisher`.

Example:

```java
// Define an event, inherit from ApplicationEvent and write the corresponding constructor
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }

    
// Define an event listener, implement the ApplicationListener interface, and override the onApplicationEvent() method;
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    //Use onApplicationEvent to receive messages
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("The information received is: " + msg);
    }

}
// To publish events, you can publish messages through the publishEvent() method of ApplicationEventPublisher.
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        //Post an event
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}

```

When the `publish()` method of `DemoPublisher` is called, such as `demoPublisher.publish("Hello")`, the console will print out: `The information received is: Hello`.

## Adapter design pattern

The Adapter Pattern transforms an interface into another interface that the customer wants. The Adapter Pattern enables classes that are incompatible with the interface to work together, and its alias is Wrapper.

### Adapter design pattern in spring AOP

We know that the implementation of Spring AOP is based on the proxy mode, but the enhancement or advice of Spring AOP uses the adapter mode, and the related interface is `AdvisorAdapter`. The commonly used types of Advice are: `BeforeAdvice` (before the target method is called, pre-notification), `AfterAdvice` (after the target method is called, post-notification), `AfterReturningAdvice` (after the target method is executed, before return) and so on. Each type of Advice (notification) has a corresponding interceptor: `MethodBeforeAdviceInterceptor`, `AfterReturningAdviceAdapter`, and `AfterReturningAdviceInterceptor`. Spring's predefined notifications must be adapted to objects of the `MethodInterceptor` interface (method interceptor) type through the corresponding adapter (for example, `MethodBeforeAdviceInterceptor` is responsible for adapting `MethodBeforeAdvice`).

### Adapter design pattern in spring MVC

In Spring MVC, `DispatcherServlet` calls `HandlerMapping` according to the request information, and resolves the `Handler` corresponding to the request. After parsing to the corresponding `Handler` (that is, the `Controller` controller we usually call), it starts to be processed by the `HandlerAdapter` adapter. `HandlerAdapter` is the expected interface, the specific adapter implementation class is used to adapt to the target class, and `Controller` is the class that needs to be adapted.

**Why use the adapter pattern in Spring MVC?** There are many types of `Controller` in Spring MVC, and different types of `Controller` process requests in different ways. If you don't use the adapter mode, `DispatcherServlet` directly obtains the corresponding type of `Controller`, you need to judge by yourself, like the following code:

```java
if(mappedHandler.getHandler() instanceof MultiActionController){  
   ((MultiActionController)mappedHandler.getHandler()).xxx  
}else if(mappedHandler.getHandler() instanceof XXX){  
    ...  
}else if(...){  
   ...  
}  
```

If we add another `Controller` type, we need to add another line of judgment statement to the above code. This form makes the program difficult to maintain and violates the open and closed principle of design pattern-open to extension, closed to modification.

## Decorator design pattern

The decorator mode can dynamically add some additional properties or behaviors to the object. Compared to using inheritance, the decorator model is more flexible. To put it simply, when we need to modify the original function, but we do not want to directly modify the original code, design a Decorator to cover the original code. In fact, the decorator pattern is used in many places in the JDK, such as the `InputStream` family. There are `FileInputStream` (reading files) and `BufferedInputStream` under the `InputStream` class (increasing the cache, greatly improving the speed of reading files) Other subclasses have extended its functions without modifying the `InputStream` code.

![Schematic diagram of decorator mode](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Decorator.jpg)

When configuring DataSource in Spring, DataSource may be a different database and data source. Can we dynamically switch between different data sources based on customer needs without modifying the original code? At this time, we need to use the decorator mode (I don't understand the specific principles too much at this point). The wrapper pattern used in Spring contains `Wrapper` or `Decorator` in the class name. These classes basically dynamically add some additional responsibilities to an object

## Summary

What design patterns are used in the Spring framework?

- **Factory design pattern**: Spring uses the factory pattern to create bean objects through `BeanFactory` and `ApplicationContext`.
- **Agency design pattern**: Implementation of Spring AOP function.
- **Singleton design pattern**: Beans in Spring are singletons by default.
- **Template method pattern**: Spring's `jdbcTemplate`, `hibernateTemplate` and other classes that end with Template for database operations, they use the template mode.
- **Decorator design pattern**: Our project needs to connect to multiple databases, and different customers will visit different databases according to their needs during each visit. This model allows us to dynamically switch between different data sources according to customer needs.
- **Observer pattern**: The Spring event-driven model is a classic application of the observer pattern.
- **Adapter pattern**: The enhancement or advice of Spring AOP uses the adapter pattern, and spring MVC also uses the adapter pattern adaptation `Controller`.
- ......

## Reference

- 《Spring technology insider》
- <https://blog.eduonix.com/java-programming-2/learn-design-patterns-used-spring-framework/>
- <http://blog.yeamin.top/2018/03/27/单例模式-Spring单例实现原理分析/>
- <https://www.tutorialsteacher.com/ioc/inversion-of-control> 
- <https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html>
- <https://juejin.im/post/5a8eb261f265da4e9e307230>
- <https://juejin.im/post/5ba28986f265da0abc2b6084>

## No public

如果大家想要实时关注我更新的文章以及分享的干货的话，可以关注我的公众号。

**《Java面试突击》:** 由本文档衍生的专为面试而生的《Java面试突击》V2.0 PDF 版本[公众号](#公众号)后台回复 **"Java面试突击"** 即可免费领取！

**Java工程师必备学习资源:** 一些Java工程师常用学习资源公众号后台回复关键字 **“1”** 即可免费无套路获取。 

![我的公众号](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/167598cd2e17b8ec.png)
