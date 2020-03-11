# Spring

## 本次复习中要关注的点

- 什么是Spring，有什么用
- IoC
- AoP
- 数据访问
- 事务管理
- Web

## 什么是Spring、为什么要使用Spring

>Spring是一个轻量级的Java开发框架，这个框架有许多成员，比如Spring Security，Spring Batch等彼此独立  
模块，提供着实用的功能。  
在整个技术架构中，处于最底层的是`SpringCore`，它提供了依赖注入管理和最基本的基础工具，接着是`AOP`模块，它是其他模块赖以生存的基础，比如事务管理。还有其他诸如`Web`的模块，也提供着重要的功能。  
使用了Spring，可以统一开发的模式，减少重复造轮子，方便快捷地集成其他框架，极大地提高了开发的速率和规范。

## IoC

>Inversion of Control，就是控制翻转的意思。原来使用一个对象，必须使用者本身去创建，再使用。现在不需要自己去创建，而是有一个IoC容器来为你提供这个对象。  
IoC可以帮助我们**解耦**各业务对象间依赖关系的对象绑定方式。  
控制反转是目的，依赖注入是手段  
举个例子，你去洗浴城，你想要一个小姐，你不需要自己去找，你可以让客服中心给你找过来，客服中心就是IOC容器，持有资源

IoC Service Provider的职责

- 业务对象的构建管理
- 业务对象之间的依赖绑定

Spring的IoC容器，不止是拥有IoC容器的基本功能，还有其他诸如对象生命周期管理、线程管理的功能，Spring有两种容器类型：

- BeanFactory：有基本的功能，更轻量级，初始化速度更快；
  - XmlBeanFactory
- ApplicationContext：持有BeanFactory，除了拥有BeanFactory的功能外，还有统一资源加载策略、事件发布，国际化信息支持等功能，要求更多的系统资源，初始化时间更长。
  - FileSystemXmlApplicationContext
  - ClassPathXmlApplicationContext
  - XmlWebApplicationContext

Bean的Scope：用来管理Bean的数量和存活时间：

- singleton：容器内只有一个，随容器生，随容器灭
- prototype：每次需要时都是新建的，随后容器不管理其生命周期
- request：每个web请求都生成一个，请求结束则结束生命，是一种特殊的prototype
- session：相对于request，存活时间更长
- global session：在基于portlet的Web应用程序中才有意义
- 用户自定义的scope

BeanFactoryPostProcessor：允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。
BeanFactoryPostProcessor存在于容器启动阶段，而BeanPostProcessor存在于对象实例化阶段，不可混淆。

- PropertyPlaceholderConfigurer
- PropertyOverrideConfigurer
- CustomEditorConfigurer

容器实现功能过程：

- 容器启动阶段
  - 加载配置、分析配置信息
  - 装备到BeanDefinition
  - 其他后处理，比如加入BeanFactoryPostProcessor
- Bean实例化阶段
  - 设置对象属性
  - 检查Aware相关接口并设置相关依赖
  - BeanPostProcessor前置处理
  - 如果是InitializingBean，则调用afterPropertiesSet方法
  - 执行init-method（如果有）
  - BeanPostProcessor后置处理
  - 注册必要的Destruction相关回调接口
  - 是否实现DisposableBean接口
  - 执行destroy方法（如果有）

一级缓存：单例缓存池 一个同步MAP

IDEA 的断点是可以加条件的

beanDefination
    是否懒加载
    是否依赖于什么
    作用域是怎么样的

## AOP

>什么是AOP：传统的开发范式，比如OOP，是一个主业务逻辑从上至下进行编码。系统由多个主逻辑业务组成。但是在多个逻辑业务中，有一些点是共同的，比如安全权限检查，日志记录。它们本身属于系统性需求，却散落在各个业务逻辑需求之中。我们可以关注这些横切点，把它们抽离出来，以模块形式开发并于原先的业务模块结合。这种方式就是面向切面编程：AOP;

### 两个AOP时代

- 静态AOP时代：早期的AspectJ，通过特定的编译器，将横切点编译并织入到系统的静态类中。性能好，但灵活性不够。
- 动态AOP时代：织入过程在系统运行之后才开始，更加灵活，但是会影响性能。

### Java平台上的AOP实现

- 动态代理：JDK1.3之后支持的，只要实现特定的接口，系统就能为目标类创建代理类，并将横切逻辑织入；
- 动态字节码增强：使用Java工具库，比如ASM或者CGLIB，动态构建字节码的class文件，相对于动态代理，不需要实现特定的接口。**无法对final修饰的类或者方法起作用**,Spring优先使用动态代理，次之使用动态字节码增强技术；
- Java代码生成：比如EJB容器，可以根据描述符，动态地生成Java代码，再部署到容器中
- 自定义类加载器：在原先的类进入加载器之前，改动class文件，加入横切逻辑，再交给虚拟机运行。一些服务器会限制整个类加载体系。
- AOL扩展：现在的AspectJ或者Spring AOP，用面向切面语言来描述切面编程概念，更具表达性，性能也很好。

### 组成AOP编程的几个概念

- Joinpoint：织入的位置
- Pointcut：Joinpoint的表述方式，或者说，Pointcut用Joinpoint来说明条件，Pointcut之间可以做逻辑运算
- Advice：具体的织入逻辑
- Aspect：一个切面的模块封装，包含了Pointcut和相关的Advice建议

### 织入器

- AspectJ使用的是acj
- SpringAOP使用的是proxyFactory类

### 应用场景

- 异常处理
- 安全检查
- 日志
- 缓存

### 有关公开当前调用的代理对象的探讨

- 问题：Spring AOP为目标对象创建了代理对象，并对method1和method2都有对应的织入逻辑，但是当目标对象的method1中有对method2的调用逻辑的时候。调用method1，method1的织入逻辑会执行，但是method2的织入逻辑却不会执行
- 原因：因为代理对象在走织入方法之后，又回到目标对象去走业务方法，就是method1，当时也在目标对象上走完method2，而不是在代理对象上走method2，所以method2的织入逻辑没有被执行。
- 解决方法：用AopContext.currentProxy来调出代理对象，然后用代理对象执行method2，或者用其他方法，只要能够让代理对象走method2就好。
- 其他：AspectJ是将代码织入到目标对象中，所以不会出现这种问题。

## 数据访问

### 统一的数据访问异常层次体系

>背景：不同的数据库操作，会抛出不同的异常。不同的数据库厂商也有不同的异常的类型。所以这留给的客户端比较多要做的额外工作。

Spring将各个数据库厂商，多种不同的异常分类转译为其他异常，大部分是`unchecked exception`。这样做向客户端屏蔽了不同异常的差异性，更能集中处理异常。  
Spring创建了一个异常层次体系，根接口是`DataAccessException`,有各种具体的异常实现了这个接口：

- CleanupFailureDataAccessException
- DataAccessResourceFailureException
- ConcurrencyFailureException
- ......

### JDBC API的最佳实践

>背景：JDBC虽然是标准的API，但是使用过程还是挺繁琐的，比如创建连接，获取statement对象，处理异常，关闭连接等，每一个操作都需要这些过程，所以造成大量的重复工作，也容易疏漏细节。

Spring基于`模板方法模式`，创建JdbcTemplate，其有两个主要的职责：

- 封装所有基于JDBC的数据访问代码，以统一的格式和规范来使用JDBC API；
- 对SQLException所提供的异常信息进行转译，纳入自身的异常层次体系，简化了客户端对异常的处理；

具体实现：将连接管理、statement管理，异常处理等相同的代码封装起来，其他的业务逻辑，由实现了`StatementCallback`接口的实现类来执行

### 对各种ORM的集成

- 统一的资源管理方式：类似于使用JDBC的难题，以统一的方式对连接等资源进行管理；
- 特定于ORM的数据访问异常到Spring统一异常体系的转译；
- 统一的数据访问事务管理及控制方式：让统一的事务管理抽象层去统一管理；

## 事务管理

### 有关事务的楔子

#### 事务的特性

- 原子性
- 一致性
- 隔离性
  - read uncommitted
  - read committed
  - repeatable read
  - serializable
- 持久性

#### 事务家族的成员

- resource manager：简称RM，负责存储并管理系统数据资源的状态，比如数据库服务器，JMS消息服务器；
- transaction processing monitor：简称TPM或者TP Monitor。职责是在分布式事务场景中协调包含多个RM的事务处理；
- transaction manager：简称为TM，TMP中的核心模块。

全局事务：涉及多个RM，采用两阶段提交协议；  
局部事务：只有一个RM；

### Spring事务框架

>基本原则：让事务管理的关注点与数据访问关注点相分离。即无需关心需要什么事务资源，也无需关心这些资源是否参与事务、如何参与事务；

#### 三个主要接口

- PlatformTransactionManager
  - 界定事务边界；
  - 参照`TransactionDefinition`的属性来开启相关的事务；
- TransactionDefinition
  - 隔离级别：对于数据库的隔离级别
  - 传播行为
    - PROPAGATION_REQUIRED：如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。
    - PROPAGATION_SUPPORTS：如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行。
    - PROPAGATION_MANDATORY：如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。
    - PROPAGATION_REQUIRES_NEW：开启一个新的事务。如果一个事务已经存在，则先将这个存在的事务挂起。
    - PROPAGATION_NOT_SUPPORTED：总是非事务地执行，并挂起任何存在的事务。
    - PROPAGATION_NEVER：总是非事务地执行，如果存在一个活动事务，则抛出异常。
    - PROPAGATION_NESTED：如果一个活动的事务存在，则运行在一个嵌套的事务中。 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行。
  - 事务的超时时间
  - 是否为只读事务
- TransactionStatus
  - 查询事务状态
  - 对事务进行有限的控制，比如在当前事务中创建内部嵌套事务、通过setRollbackOnly()方法使事务回滚

## Web MVC框架

>Spring MVC对请求处理期间涉及的各种关注点进行了合理而完全的分离，并且明确设置了相应的角色用于建模并处理整个生命周期中的各个关注点。

- DispatcherServlet：拦截所有的请求
- HandlerMapping：返回HandlerExecutionChain
- HandlerExecutionChain：包含了所有的Handler和HandlerIntercepter
- HandlerAdapter：从`HandlerExecutionChain`中识别出应该使用什么handler
- handler：具体处理者，controller就是一种handler
- ModelAndView：由handler返回，持有数据和视图信息
- ViewResolver：决定哪一个视图来进行渲染
- View：视图，执行具体的渲染逻辑，生成最后的输出结果

Spring MVC中个角色交互图
![MVC中个角色交互图](https://img-blog.csdn.net/20181022214913888?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MjYyOTY4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
  