---
layout: post
title: Spring IoC 和 AOP
description: Spring IoC 和 AOP 的概念
summary: Spring IoC 和 AOP 的概念。
tags: [spring]
---

## IoC
IoC（Inverse of Control：控制反转）是一种设计思想，就是将**原本在程序中手动创建对象的控制权，交由Spring框架来管理**。 IoC 在其他语言中也有应用，并非 Spirng 特有。 IoC 容器是 Spring 用来实现 IoC 的载体，IoC 容器实际上就是个Map（key，value），Map 中存放的是各种对象。

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

## Spring IoC 的初始化过程

![Spring_IoC_的初始化过程](../../../../assets/images/Spring_IoC_的初始化过程.jpg)

[Spring IoC 初始化源码分析](https://doocs.github.io/source-code-hunter/#/?id=ioc-%e5%ae%b9%e5%99%a8)

## IoC 和 DI的区别
IoC 控制反转，指将对象的创建权，反转到Spring容器，DI 依赖注入，指Spring创建对象的过程中，将对象依赖属性通过配置进行注入。

## BeanFactory 接口和 ApplicationContext 接口的区别 
1. ApplicationContext 接口继承BeanFactory接口，Spring核心工厂是BeanFactory，BeanFactory采取延迟加载，第一次 getBean() 时才会初始化 Bean，ApplicationContext 是会在加载配置文件时初始化 Bean；
2. ApplicationContext 是对 BeanFactory 扩展，它可以进行国际化处理、事件传递和 bean 自动装配以及各种不同应用层的 Context 实现。

开发中基本都在使用ApplicationContext, web项目使用WebApplicationContext ，很少用到BeanFactory。
```java
BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
IHelloService helloService = (IHelloService) beanFactory.getBean("helloService");
helloService.sayHello();
```

## AOP
AOP（Aspect-Oriented Programming：面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

![Spring_AOP](../../../../assets/images/Spring_AOP.png)

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用Cglib，这时候 Spring AOP 会使用 Cglib 生成一个被代理对象的子类来作为代理，如下图所示：

![Spring_AOP_jdk_proxy和cglib_proxy](../../../../assets/images/Spring_AOP_jdk_proxy和cglib_proxy.jpg)

当然也可以使用 AspectJ，Spring AOP 已经集成了AspectJ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP。

## Spring AOP 和 AspectJ AOP 的区别
Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。 Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ  相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

## AOP 术语解释
- 横切关注点：从每个方法中抽取出来的同一类非核心业务。
- 切面 (Aspect)：封装横切关注点信息的类，每个关注点体现为一个通知方法。
- 通知 (Advice)：切面必须要完成的各个具体工作。
- 目标 (Target)：被通知的对象。
- 代理 (Proxy)：向目标对象应用通知之后创建的代理对象。
-  连接点 (Joinpoint)：横切关注点在程序代码中的具体体现，对应程序执行的某个特定位置。例如：类某个方法调用前、调用后、方法捕获到异常后等。
    在应用程序中可以使用横纵两个坐标来定位一个具体的连接点：

![](../../../../assets/images/SpringAOP横纵坐标定位连接点.jpg)

- 切入点 (pointcut)：定位连接点的方式。每个类的方法中都包含多个连接点，所以连接点是类中客观存在的事物。如果把连接点看作数据库中的记录，那么切入点就是查询条件--AOP 可以通过切入点定位到特定的连接点。切点通过 `org.springframework.aop.Pointcut` 接口进行描述，它使用类和方法作为连接点的查询条件。
- 织入 (Weaving)：把切面（aspect）连接到其它的应用程序类型或者对象上，并创建一个被通知（advised）的对象。 这些可以在编译时（例如使用 AspectJ 编译器），类加载时和运行时完成。 Spring和其他纯 Java AOP 框架一样，在运行时完成织入。

![](../../../../assets/images/AOP术语.png)

## 通知的类型
- 前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。
-  返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
- 抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。
后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
- 环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。
 
环绕通知是最常用的一种通知类型。大部分基于拦截的 AOP 框架，例如 Nanning 和 JBoss4，都只提供环绕通知。

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

## JDK 代理、CGLIB 和 AspectJ 的区别
