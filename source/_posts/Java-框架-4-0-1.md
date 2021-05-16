---
title: Spring  入坑
date: 2019-05-22 12:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

Spring框架是由于软件开发的复杂性而创建的。Spring使用的是基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅仅限于服务器端的开发。从简单性、可测试性和松耦合性角度而言，绝大部分Java应用都可以从Spring中受益。

- 目的：解决企业应用开发的复杂性
- 功能：使用基本的JavaBean代替EJB，并提供了更多的企业应用功能
- 范围：任何Java应用

**Spring是一个轻量级控制反转(IoC)和面向切面(AOP)的容器框架。**简单来说，Spring是一个分层的JavaSE/EE full-stack(一站式) 轻量级开源框架。

<!--more-->

- 轻量级：与EJB对比，依赖资源少，销毁的资源少。
- 分层： 一站式，每一个层都提供的解决方案

  - web层：struts，spring-MVC

  - service层：spring

  - dao层：hibernate，mybatis ， jdbcTemplate  --> spring-data，spring-tx
  - 测试：spring-test
  - 通用：spring-aop

**优点**

1. 方便解耦，简化开发  （高内聚低耦合）

   Spring就是一个大工厂（容器），可以将所有对象创建和依赖关系维护，交给Spring管理
   Spring工厂是用于生成bean

2. AOP编程的支持

   Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能

3. 声明式事务的支持

   只需要通过配置就可以完成对事务的管理，而无需手动编程

4. 方便程序的测试

   Spring对Junit4支持，可以通过注解方便的测试Spring程序

5. 方便集成各种优秀框架

   Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz等）的直接支持

6. 降低JavaEE API的使用难度

   Spring 对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低

 **spring能帮我们做什么**

1. Spring 能帮我们根据配置文件创建及组装对象之间的依赖关系。
2. Spring 面向切面编程能帮助我们无耦合的实现日志记录，性能统计，安全控制。
3. Spring 能非常简单的帮我们管理数据库事务。
4. Spring 还提供了与第三方数据访问框架（如Hibernate、JPA）无缝集成，而且自己也提供了一套JDBC访问模板来方便数据库访问。
5. Spring 还提供与第三方Web（如Struts1/2、JSF）框架无缝集成，而且自己也提供了一套Spring MVC框架，来方便web层搭建。
6. Spring 能方便的与Java EE（如Java Mail、任务调度）整合，与更多技术整合（比如缓存框架）。

#### spring体系结构

Spring框架至今已集成了20多个模块，这些模块分布在以下模块中：

- 核心容器（Core Container）
- 数据访问/集成（Data Access/Integration）层
- Web层
- AOP（Aspect Oriented Programming）模块
- 植入（Instrumentation）模块
- 消息传输（Messaging）
- 测试（Test）模块

Spring体系结构如下图：

![UTOOLS1576384007416.png](https://i.loli.net/2019/12/15/Q3bgv6oEkCyfmHB.png)

**核心容器**

核心容器由**spring-core，spring-beans，spring-context，spring-context-support和spring-expression**（SpEL，Spring表达式语言，Spring Expression Language）等模块组成，它们的细节如下： 

- **spring-core**模块提供了框架的基本组成部分，包括 IoC 和依赖注入功能。
- **spring-beans** 模块提供 BeanFactory，工厂模式的微妙实现，它移除了编码式单例的需要，并且可以把配置和依赖从实际编码逻辑中解耦。
- **context**模块建立在由**core**和 **beans** 模块的基础上建立起来的，它以一种类似于JNDI注册的方式访问对象。Context模块继承自Bean模块，并且添加了国际化（比如，使用资源束）、事件传播、资源加载和透明地创建上下文（比如，通过Servelet容器）等功能。Context模块也支持Java  EE的功能，比如EJB、JMX和远程调用等。**ApplicationContext**接口是Context模块的焦点。**spring-context-support**提供了对第三方库集成到Spring上下文的支持，比如缓存（EhCache,  Guava, JCache）、邮件（JavaMail）、调度（CommonJ, Quartz）、模板引擎（FreeMarker,  JasperReports, Velocity）等。
- **spring-expression**模块提供了强大的表达式语言，用于在运行时查询和操作对象图。它是JSP2.1规范中定义的统一表达式语言的扩展，支持set和get属性值、属性赋值、方法调用、访问数组集合及索引的内容、逻辑算术运算、命名变量、通过名字从Spring  IoC容器检索对象，还支持列表的投影、选择以及聚合等。

它们的完整依赖关系如下图所示：

![2.png](https://i.loli.net/2019/06/08/5cfbc96f23e6012584.png)

**数据访问/集成**

数据访问/集成层包括 JDBC，ORM，OXM，JMS 和事务处理模块，它们的细节如下： 

（注：JDBC=Java Data Base Connectivity，ORM=Object Relational Mapping，OXM=Object XML Mapping，JMS=Java Message Service）

- **JDBC** Spring-jdbc模块提供了JDBC抽象层，它消除了冗长的JDBC编码和对数据库供应商特定错误代码的解析。
- **ORM** Spring-orm模块提供了对流行的对象关系映射API的集成，包括JPA、JDO和Hibernate等。通过此模块可以让这些ORM框架和spring的其它功能整合，比如前面提及的事务管理。
- **OXM** Spring-oxm模块提供了对OXM实现的支持，比如JAXB、Castor、XML Beans、JiBX、XStream等。
- **JMS** Spring-jms模块包含生产（produce）和消费（consume）消息的功能。从Spring 4.1开始，集成了spring-messaging模块。
- **事务** Spring-tx模块为实现特殊接口类及所有的 POJO 支持编程式和声明式事务管理。（注：编程式事务需要自己写beginTransaction()、commit()、rollback()等事务管理方法，声明式事务是通过注解或配置由spring自动处理，编程式事务粒度更细）

**Web**

Web 层由 Web，Web-MVC，Web-Socket 和 Web-Portlet 组成，它们的细节如下：

- **Web** 模块提供面向web的基本功能和面向web的应用上下文，比如多部分（multipart）文件上传功能、使用Servlet监听器初始化IoC容器等。它还包括HTTP客户端以及Spring远程调用中与web相关的部分。。
- **Web-MVC** 模块为web应用提供了模型视图控制（MVC）和REST Web服务的实现。Spring的MVC框架可以使领域模型代码和web表单完全地分离，且可以与Spring框架的其它所有功能进行集成。
- **Web-Socket** 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。
- **Web-Portlet** 模块提供了用于Portlet环境的MVC实现，并反映了spring-webmvc模块的功能。

**其他**

还有其他一些重要的模块，像 AOP，Aspects，Instrumentation，Web 和测试模块，它们的细节如下：

- **AOP** 模块提供了面向方面的编程实现，允许你定义方法拦截器和切入点对代码进行干净地解耦，从而使实现功能的代码彻底的解耦出来。使用源码级的元数据，可以用类似于.Net属性的方式合并行为信息到代码中。
- **Aspects** 模块提供了与 **AspectJ** 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架。
- **Instrumentation** 模块在一定的应用服务器中提供了类 instrumentation 的支持和类加载器的实现。
- **Messaging** Spring4.0以后新增了消息（Spring-messaging）模块，该模块提供了对消息传递体系结构和协议的支持。
- **测试** Spring-test模块支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试

#### 常用术语：

- **框架：**是能**完成一定功能**的**半成品**。
   框架能够帮助我们完成的是：**项目的整体框架、一些基础功能、规定了类和对象如何创建，如何协作等**，当我们开发一个项目时，框架帮助我们完成了一部分功能，我们自己再完成一部分，那这个项目就完成了。
   
- **非侵入式设计：**
   从框架的角度可以理解为：**无需继承框架提供的任何类**
   这样我们在更换框架时，之前写过的代码几乎可以继续使用。
   
- **轻量级和重量级：**
   轻量级是相对于重量级而言的，**轻量级一般就是非入侵性的、所依赖的东西非常少、资源占用非常少、部署简单等等**，其实就是**比较容易使用**，而**重量级正好相反**。
   
- **JavaBean：**
   即**符合 JavaBean 规范**的 Java 类
   
- **POJO：**即 **Plain Old Java Objects，简单老式 Java 对象**
   它可以包含业务逻辑或持久化逻辑，但**不担当任何特殊角色**且**不继承或不实现任何其它Java框架的类或接口。**
   
   > 注意：bean 的各种名称——虽然 Spring 用 bean 或者 JavaBean 来表示应用组件，但并不意味着 Spring 组件必须遵循 JavaBean 规范，一个 Spring 组件可以是任意形式的 POJO。
   
- **容器：**
   在日常生活中容器就是一种盛放东西的器具，从程序设计角度看就是**装对象的的对象**，因为存在**放入、拿出等**操作，所以容器还要**管理对象的生命周期**。

#### 核心模块

![1.jpg](https://i.loli.net/2019/06/08/5cfb3c0600ad290396.jpg)

如果作为一个整体，这些模块为你提供了开发企业应用所需的一切。但你不必将应用完全基于Spring框架。你可以自由地挑选适合你的应用的模块而忽略其余的模块。

就像你所看到的，所有的Spring模块都是在核心容器之上构建的。容器定义了Bean是如何创建、配置和管理的——更多的Spring细节。当你配置你的应用时，你会潜在地使用这些类。但是作为一名开发者，你最可能对影响容器所提供的服务的其它模块感兴趣。这些模块将会为你提供用于构建应用服务的框架，例如AOP和持久性。

Spring框架由7个定义良好的模块(组件)组成，各个模块可以独立存在，也可以联合使用。

1. 核心容器

   这是Spring框架最基础的部分，它提供了依赖注入（DependencyInjection）特征来实现容器对Bean的管理。这里最基本的概念是BeanFactory，它是任何Spring应用的核心。BeanFactory是工厂模式的一个实现，它使用IoC将应用配置和依赖说明从实际的应用代码中分离出来。

2. 应用上下文（Context）模块

   核心模块的BeanFactory使Spring成为一个容器，而上下文模块使它成为一个框架。这个模块扩展了BeanFactory的概念，增加了对国际化（I18N）消息、事件传播以及验证的支持。

   另外，这个模块提供了许多企业服务，例如电子邮件、JNDI访问、EJB集成、远程以及时序调度（scheduling）服务。也包括了对模版框架例如Velocity和FreeMarker集成的支持。

3. Spring的AOP模块

   Spring在它的AOP模块中提供了对面向切面编程的丰富支持。这个模块是在Spring应用中实现切面编程的基础。为了确保Spring与其它AOP框架的互用性，Spring的AOP支持基于AOP联盟定义的API。AOP联盟是一个开源项目，它的目标是通过定义一组共同的接口和组件来促进AOP的使用以及不同的AOP实现之间的互用性。通过访问他们的站点，你可以找到关于AOP联盟的更多内容。

   Spring的AOP模块也将元数据编程引入了Spring。使用Spring的元数据支持，你可以为你的源代码增加注释，指示Spring在何处以及如何应用切面函数。

4. JDBC抽象和DAO模块

   使用JDBC经常导致大量的重复代码，取得连接、创建语句、处理结果集，然后关闭连接。Spring的JDBC和DAO模块抽取了这些重复代码，因此你可以保持你的数据库访问代码干净简洁，并且可以防止因关闭数据库资源失败而引起的问题。

   这个模块还在几种数据库服务器给出的错误消息之上建立了一个有意义的异常层。使你不用再试图破译神秘的私有的SQL错误消息！

   另外，这个模块还使用了Spring的AOP模块为Spring应用中的对象提供了事务管理服务。

4. 对象/关系映射集成模块

   对那些更喜欢使用对象/关系映射工具而不是直接使用JDBC的人，Spring提供了ORM模块。Spring并不试图实现它自己的ORM解决方案，而是为几种流行的ORM框架提供了集成方案，包括Hibernate、JDO和iBATIS  SQL映射。Spring的事务管理支持这些ORM框架中的每一个也包括JDBC。

5. Spring的Web模块

   Web上下文模块建立于应用上下文模块之上，提供了一个适合于Web应用的上下文。另外，这个模块还提供了一些面向服务支持。例如：实现文件上传的multipart请求，它也提供了Spring和其它Web框架的集成，比如Struts、WebWork。

6. Spring的MVC框架

   Spring为构建Web应用提供了一个功能全面的MVC框架。虽然Spring可以很容易地与其它MVC框架集成，例如Struts，但Spring的MVC框架使用IoC对控制逻辑和业务对象提供了完全的分离。

   它也允许你声明性地将请求参数绑定到你的业务对象中，此外，Spring的MVC框架还可以利用Spring的任何其它服务，例如国际化信息与验证。

**jar包功能**

1. **spring.jar** 是包含有完整发布模块的单个jar 包。但是不包括mock.jar, aspects.jar, spring-portlet.jar, and spring-hibernate2.jar。
2. **spring-src.zip**就是所有的源代码压缩包。
3. 除了spring.jar 文件，Spring 还包括有其它21 个独立的jar 包，各自包含着对应的Spring组件，用户可以根据自己的需要来选择组合自己的jar 包，而不必引入整个spring.jar 的所有类文件。
4. **spring-core.jar**
   这个jar 文件包含Spring 框架基本的核心工具类。Spring 其它组件要都要使用到这个包里的类，是其它组件的基本核心，当然你也可以在自己的应用系统中使用这些工具类。
   外部依赖Commons Logging， (Log4J)。
5. **spring-beans.jar**
   这个jar  文件是所有应用都要用到的，它包含访问配置文件、创建和管理bean 以及进行Inversion of Control / Dependency  Injection（IoC/DI）操作相关的所有类。如果应用只需基本的IoC/DI 支持，引入spring-core.jar  及spring-beans.jar 文件就可以了。
   外部依赖spring-core，(CGLIB)。
6. **spring-aop.jar**
   这个jar 文件包含在应用中使用Spring 的AOP 特性时所需的类和源码级元数据支持。使用基于AOP 的Spring特性，如声明型事务管理（Declarative Transaction Management），也要在应用里包含这个jar包。
   外部依赖spring-core， (spring-beans，AOP Alliance， CGLIB，Commons Attributes)。
7. **spring-context.jar**
   这个jar 文件为Spring 核心提供了大量扩展。可以找到使用Spring ApplicationContext特性时所需的全部类，JDNI 所需的全部类，instrumentation组件以及校验Validation 方面的相关类。
   外部依赖spring-beans, (spring-aop)。
8. **spring-dao.jar**
   这个jar 文件包含Spring DAO、Spring Transaction 进行数据访问的所有类。为了使用声明型事务支持，还需在自己的应用里包含spring-aop.jar。
   外部依赖spring-core，(spring-aop， spring-context， JTA API)。
9. **spring-jdbc.jar**
   这个jar 文件包含对Spring 对JDBC 数据访问进行封装的所有类。
   外部依赖spring-beans，spring-dao。
10. **spring-support.jar**
    这个jar 文件包含支持UI模版（Velocity，FreeMarker，JasperReports），邮件服务，脚本服务(JRuby)，缓存Cache（EHCache），任务计划Scheduling（uartz）方面的类。
    外部依赖spring-context, (spring-jdbc, Velocity, FreeMarker, JasperReports, BSH, Groovy, JRuby, Quartz, EHCache)
11. **spring-web.jar**
    这个jar 文件包含Web 应用开发时，用到Spring 框架时所需的核心类，包括自动载入Web Application Context 特性的类、Struts 与JSF 集成类、文件上传的支持类、Filter 类和大量工具辅助类。
    外部依赖spring-context, Servlet API, (JSP API, JSTL, Commons FileUpload, COS)。
12. **spring-webmvc.jar**
    这个jar 文件包含Spring MVC 框架相关的所有类。包括框架的Servlets，Web MVC框架，控制器和视图支持。当然，如果你的应用使用了独立的MVC 框架，则无需这个JAR 文件里的任何类。
    外部依赖spring-web, (spring-support，Tiles，iText，POI)。
13. **spring-portlet.jar**
    spring自己实现的一个类似Spring MVC的框架。包括一个MVC框架和控制器。
    外部依赖spring-web， Portlet API，(spring-webmvc)。
14. **spring-struts.jar**
    Struts框架支持，可以更方便更容易的集成Struts框架。
    外部依赖spring-web，Struts。
15. **spring-remoting.jar**
    这个jar 文件包含支持EJB、远程调用Remoting（RMI、Hessian、Burlap、Http Invoker、JAX-RPC）方面的类。
    外部依赖spring-aop， (spring-context，spring-web，Hessian，Burlap，JAX-RPC，EJB API)。
16. **spring-jmx.jar**
    这个jar包提供了对JMX 1.0/1.2的支持类。
    外部依赖spring-beans，spring-aop， JMX API。
17. **spring-jms.jar**
    这个jar包提供了对JMS 1.0.2/1.1的支持类。
    外部依赖spring-beans，spring-dao，JMS API。
18. **spring-jca.jar**
    对JCA 1.0的支持。
    外部依赖spring-beans，spring-dao， JCA API。
19. **spring-jdo.jar**
    对JDO 1.0/2.0的支持。
    外部依赖spring-jdbc， JDO API， (spring-web)。
20. **spring-jpa.jar**
    对JPA 1.0的支持。
    外部依赖spring-jdbc， JPA API， (spring-web)。
21. **spring-hibernate.jar**
    对Hibernate 2.1的支持，已经不建议使用。
    外部依赖spring-jdbc，Hibernate2，(spring-web)。
22. **spring-toplink.jar**
    对TopLink框架的支持。
    外部依赖spring-jdbc，TopLink。
23. **spring-ibatis.jar**
    对iBATIS SQL Maps的支持。
    外部依赖spring-jdbc，iBATIS SQL Maps。
24. **spring-mock.jar**
    这个jar  文件包含Spring 一整套mock 类来辅助应用的测试。Spring 测试套件使用了其中大量mock  类，这样测试就更加简单。模拟HttpServletRequest 和HttpServletResponse 类在Web  应用单元测试是很方便的。并且提供了对JUnit的支持。
    外部依赖spring-core。
25. **spring-aspects.jar**
    提供对AspectJ的支持，以便可以方便的将面向方面的功能集成进IDE中，比如Eclipse AJDT。
    外部依赖。
26. **spring-agent.jar**
    Spring的InstrumentationSavingAgent (为InstrumentationLoadTimeWeaver)，一个设备代理包，可以参考JDK1.5的Instrumentation功能获得更多信息。
    外部依赖none (for use at JVM startup: "-javaagent:spring-agent.jar")。
27. **spring-tomcat-weaver.jar**
    扩展Tomcat的ClassLoader，使其可以使用instrumentation（设备）类。
    外部依赖none (for deployment into Tomcat's "server/lib" directory)。

可以根据jar包上述名称使用maven依赖或gradle依赖，具体如下

```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <spring.version>5.2.2.RELEASE</spring.version>
    </properties>   
<dependencyManagement>
        <dependencies>

            <!-- Spring -->
            <!-- 1)包含Spring 框架基本的核心工具类。Spring 其它组件要都要使用到这个包里的类，是其它组件的基本核心 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 2)这个jar 文件是所有应用都要用到的，它包含访问配置文件、创建和管理bean 以及进行Inversion of Control
                / Dependency Injection（IoC/DI）操作相关的所有类。如果应用只需基本的IoC/DI 支持，引入spring-core.jar
                及spring-beans.jar 文件就可以了。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 3)这个jar 文件为Spring 核心提供了大量扩展。可以找到使用Spring ApplicationContext特性时所需的全部类，JDNI
                所需的全部类，instrumentation组件以及校验Validation 方面的相关类。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 4) 这个jar 文件包含对Spring 对JDBC 数据访问进行封装的所有类。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 5) 为JDBC、Hibernate、JDO、JPA等提供的一致的声明式和编程式事务管理。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 6)Spring web 包含Web应用开发时，用到Spring框架时所需的核心类，包括自动载入WebApplicationContext特性的类、Struts与JSF集成类、文件上传的支持类、Filter类和大量工具辅助类。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 7)包含SpringMVC框架相关的所有类。 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- 8)Spring test 对JUNIT等测试框架的简单封装 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
                <scope>test</scope>
            </dependency>
            <!-- 9)Spring test 对AOP的简单封装 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <!-- 10)spring-context-support 提供了对第三方库集成到Spring上下文的支持 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${spring.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

#### IOC

Inversion of Control，是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

IoC可以认为是一种全新的设计模式，但是理论和时间成熟相对较晚，并没有包含在GoF中。

IOC 容器具有依赖注入功能的容器，它可以创建对象，IOC 容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。通常new一个实例，控制权由程序员控制，而"控制反转"是指new实例工作不由程序员来做而是交给Spring容器来做。在Spring中BeanFactory是IOC容器的实际代表者。

![1.jpg](https://i.loli.net/2019/06/08/5cfbca28b99c624934.jpg)

**Interface Driven Design接口驱动**，接口驱动有很多好处，可以提供不同灵活的子类实现，增加代码稳定和健壮性等等，但是接口一定是需要实现的，也就是如下语句迟早要执行：`AInterface a = new AInterfaceImp();` 这样一来，耦合关系就产生了，如：

```java
classA
{
    AInterface a;
 
    A(){}
     
    AMethod()//一个方法
    {
        a = new AInterfaceImp();
    }
}
```

Class A与AInterfaceImp就是依赖关系，如果想使用AInterface的另外一个实现就需要更改代码了。当然我们可以建立一个Factory来根据条件生成想要的AInterface的具体实现，即：

```java
InterfaceImplFactory
{
   AInterface create(Object condition)
   {
      if(condition == condA)
      {
          return new AInterfaceImpA();
      }
      else if(condition == condB)
      {
          return new AInterfaceImpB();
      }
      else
      {
          return new AInterfaceImp();
      }
    }
}
```

表面上是在一定程度上缓解了以上问题，但实质上这种代码耦合并没有改变。通过IoC模式可以彻底解决这种耦合，它把耦合从代码中移出去，放到统一的XML 文件中，通过一个容器在需要的时候把这个依赖关系形成，即把需要的接口实现注入到需要它的类中，这可能就是“依赖注入”说法的来源了。

IoC模式，系统中通过引入实现了IoC模式的IoC容器，即可由IoC容器来管理对象的生命周期、依赖关系等，从而使得应用程序的配置和依赖性规范与实际的应用程序代码分离。其中一个特点就是通过文本的配置文件进行应用程序组件间相互关系的配置，而不用重新修改并编译具体的代码。

当前比较知名的IoC容器有：Pico Container、Avalon 、Spring、JBoss、HiveMind、EJB等。

在上面的几个IoC容器中，轻量级的有Pico Container、Avalon、Spring、HiveMind等，超重量级的有EJB，而半轻半重的有容器有JBoss，Jdon等。

可以把IoC模式看作工厂模式的升华，把IoC容器看作是一个大工厂，只不过这个大工厂里要生成的对象都是在XML文件中给出定义的。利用Java 的“反射”编程，根据XML中给出的类定义生成相应的对象。从实现来看，以前在工厂模式里写死了的对象，IoC模式改为配置XML文件，这就把工厂和要生成的对象两者隔离，极大提高了灵活性和可维护性。

IoC中最基本的Java技术就是“反射”编程。通俗的说，反射就是根据给出的类名（字符串）来生成对象。这种编程方式可以让应用在运行时才动态决定生成哪一种对象。反射的应用是很广泛的，像Hibernate、Spring中都是用“反射”做为最基本的技术手段。

在过去，反射编程方式相对于正常的对象生成方式要慢10几倍，这也许也是当时为什么反射技术没有普遍应用开来的原因。但经SUN改良优化后，反射方式生成对象和通常对象生成方式，速度已经相差不大了（但依然有一倍以上的差距）。

**类型**

- 类型1 (基于接口): 可服务的对象需要实现一个专门的接口，该接口提供了一个对象，可以重用这个对象查找依赖(其它服务)。
- 类型2 (基于setter): 通过JavaBean的属性(setter方法)为可服务对象指定服务。
- 类型3 (基于构造函数): 通过构造函数的参数为可服务对象指定服务。

IoC是一个很大的概念,可以用不同的方式实现。其主要形式有两种：

- 依赖查找：容器提供回调接口和上下文条件给组件。EJB和Apache Avalon 都使用这种方式。这样一来，组件就必须使用容器提供的API来查找资源和协作对象，仅有的控制反转只体现在那些回调方法上（也就是上面所说的 类型1）：容器将调用这些回调方法，从而让应用代码获得相关资源。
- 依赖注入：组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。容器全权负责的组件的装配，它会把符合依赖关系的对象通过JavaBean属性或者构造函数传递给需要的对象。通过JavaBean属性注射依赖关系的做法称为设值方法注入(Setter Injection)；将依赖关系作为构造函数参数传入的做法称为构造器注入（Constructor Injection）

**实现方式**

1. 实现数据访问层

   数据访问层有两个目标。第一是将数据库引擎从应用中抽象出来，这样就可以随时改变数据库—比方说，从微软SQL变成Oracle。不过在实践上很少会这么做，也没有足够的理由和能力去通过使用实现数据访问层而对现有的应用进行重构。

   第二个目标是将数据模型从数据库实现中抽象出来。这使得数据库或代码开源根据需要改变，同时只会影响主应用的一小部分——数据访问层。这一目标是值得的，为了在现有系统中实现它进行必要的重构。

2. 模块与接口重构

   依赖注入背后的一个核心思想是单一功能原则（single  responsibility  principle）。该原则指出，每一个对象应该有一个特定的目的，而应用需要利用这一目的的不同部分应当使用合适的对象。这意味着这些对象在系统的任何地方都可以重用。但在现有系统里面很多时候都不是这样的。

3. 随时增加单元测试

   把功能封装到整个对象里面会导致自动测试困难或者不可能。将模块和接口与特定对象隔离，以这种方式重构可以执行更先进的单元测试。按照后面再增加测试的想法继续重构模块是诱惑力的，但这是错误的。

4. 使用服务定位器而不是构造注入

   实现控制反转不止一种方法。最常见的办法是使用构造注入，这需要在对象首次被创建时提供所有的软件依赖。然而，构造注入要假设整个系统都使用这一模式，这意味着整个系统必须同时进行重构。这很困难、有风险，且耗时。

#### AOP

> 在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方
> 式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个
> 热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑
> 的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高
> 了开发的效率。

先说AOP在Java里是利用反射机制实现（你也可以认为是动态代理，不过动态代理也是反射机制实现的，所以还是先不要管动态代理，我们这里化繁为简，不让它干扰咱们对AOP的理解），如何使用AOP呢，很简单滴，等下介绍。

要理解切面编程，就需要先理解什么是切面。用刀把一个西瓜分成两瓣，切开的切口就是切面；炒菜，锅与炉子共同来完成炒菜，锅与炉子就是切面。web层级设计中，web层->网关层->服务层->数据层，每一层之间也是一个切面。编程中，**对象与对象之间，方法与方法之间，模块与模块之间都是一个个切面。**

下面先说AOP是什么样的思想，我们一步一步慢慢来，先看一下传统程序的流程，比如银行系统会有一个取款流程

![1.png](https://i.loli.net/2019/06/08/5cfb35e1ce8ee60149.png)

我们可以把方框里的流程合为一个，另外系统还会有一个查询余额流程，我们先把这两个流程放到一起：

![2.png](https://i.loli.net/2019/06/08/5cfb35e1ce8f221146.png)

有没有发现，这个两者有一个相同的验证流程，我们先把它们圈起来再说下一步：

![3.png](https://i.loli.net/2019/06/08/5cfb35e21a02558996.png)

有没有想过可以把这个验证用户的代码是提取出来，不放到主流程里去呢，这就是AOP的作用了

有了AOP，你写代码时不要把这个验证用户步骤写进去，即完全不考虑验证用户，你写完之后，在另我一个地方，写好验证用户的代码，然后告诉Spring你要把这段代码加到哪几个地方，Spring就会帮你加过去，而不要你自己Copy过去，这里还是两个地方，如果你有多个控制流呢，这个写代码的方法可以大大减少你的时间

不过AOP的目的不是这样，这只是一个“副作用”，真正目的是，**你写代码的时候，事先只需考虑主流程，而不用考虑那些不重要的流程**，懂C的都知道，良好的风格要求在函数起始处验证参数，如果在C上可以用AOP，就可以先不管校验参数的问题，事后使用AOP就可以隔山打牛的给所有函数一次性加入校验代码，而你只需要写一次校验代码。不知道C的没关系，举一个通用的例子，经常在debug的时候要打log吧，你也可以写好主要代码之后，把打log的代码写到另一个单独的地方，然后命令AOP把你的代码加过去，注意**AOP不会把代码加到源文件里，但是它会正确的影响最终的机器代码**。

现在大概明白了AOP了吗，我们来理一下头绪，上面那个方框像不像个平面，你可以把它当块板子，这块板子插入一些控制流程，这块板子就可以当成是AOP中的一个切面。所以AOP的本质是在一系列纵向的控制流程中，把那些相同的子流程提取成一个横向的面，这句话应该好理解吧，我们把纵向流程画成一条直线，然把相同的部分以绿色突出，如下图左，而AOP相当于把相同的地方连一条横线，如下图右，这个图没画好，大家明白意思就行。

![4.png](https://i.loli.net/2019/06/08/5cfb35e21a0b668736.png)

这个验证用户这个子流程就成了一个条线，也可以理解成一个切面，aspect的意思我认为是方面，你用什么实物去类比，只要你能理解都可以。这里的切面只插了两三个流程，如果其它流程也需要这个子流程，也可以插到其它地方去。

##### 相关概念

看过了上面的例子，我想大家脑中对AOP已经有了一个大致的雏形，但是又对上面提到的切面之类的术语有一些模糊的地方，接下来就来讲解一下AOP中的相关概念，了解了AOP中的概念，才能真正的掌握AOP的精髓。

这里还是先给出一个比较专业的概念定义：

- Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。
- Joint point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。
- Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。
- Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。
- Target（目标对象）：织入 Advice 的目标对象.。
- Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程

然后举一个容易理解的例子：

看完了上面的理论部分知识, 我相信还是会有不少朋友感觉到 AOP 的概念还是很模糊, 对 AOP 中的各种概念理解的还不是很透彻. 其实这很正常, 因为 AOP 中的概念是在是太多了, 我当时也是花了老大劲才梳理清楚的.
下面我以一个简单的例子来比喻一下 AOP 中 Aspect, Joint point, Pointcut 与 Advice之间的关系.

让我们来假设一下, 从前有一个叫爪哇的小县城, 在一个月黑风高的晚上, 这个县城中发生了命案. 作案的凶手十分狡猾, 现场没有留下什么有价值的线索. 不过万幸的是, 刚从隔壁回来的老王恰好在这时候无意中发现了凶手行凶的过程, 但是由于天色已晚, 加上凶手蒙着面, 老王并没有看清凶手的面目, 只知道凶手是个男性, 身高约七尺五寸. 爪哇县的县令根据老王的描述, 对守门的士兵下命令说: 凡是发现有身高七尺五寸的男性, 都要抓过来审问. 士兵当然不敢违背县令的命令, 只好把进出城的所有符合条件的人都抓了起来.

来让我们看一下上面的一个小故事和 AOP 到底有什么对应关系.

首先我们知道, 在 Spring AOP 中 Joint point 指代的是所有方法的执行点, 而 point cut 是一个描述信息, 它修饰的是 Joint point, 通过 point cut, 我们就可以确定哪些 Joint point 可以被织入 Advice. 对应到我们在上面举的例子, 我们可以做一个简单的类比, Joint point 就相当于 爪哇的小县城里的百姓,pointcut 就相当于 老王所做的指控, 即凶手是个男性, 身高约七尺五寸, 而 Advice 则是施加在符合老王所描述的嫌疑人的动作: 抓过来审问.
为什么可以这样类比呢?

- Joint point ： 爪哇的小县城里的百姓: 因为根据定义, Joint point 是所有可能被织入 Advice 的候选的点, 在 Spring AOP中, 则可以认为所有方法执行点都是 Joint point. 而在我们上面的例子中, 命案发生在小县城中, 按理说在此县城中的所有人都有可能是嫌疑人.

- Pointcut ：男性, 身高约七尺五寸: 我们知道, 所有的方法(joint point) 都可以织入 Advice, 但是我们并不希望在所有方法上都织入 Advice, 而 Pointcut 的作用就是提供一组规则来匹配joinpoint, 给满足规则的 joinpoint 添加 Advice. 同理, 对于县令来说, 他再昏庸, 也知道不能把县城中的所有百姓都抓起来审问, 而是根据凶手是个男性, 身高约七尺五寸, 把符合条件的人抓起来. 在这里 凶手是个男性, 身高约七尺五寸 就是一个修饰谓语, 它限定了凶手的范围, 满足此修饰规则的百姓都是嫌疑人, 都需要抓起来审问.

- Advice ：抓过来审问, Advice 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 Joint point 上的. 同理, 对比到我们的例子中, 抓过来审问 这个动作就是对作用于那些满足 男性, 身高约七尺五寸 的爪哇的小县城里的百姓.

- Aspect：Aspect 是 point cut 与 Advice 的组合, 因此在这里我们就可以类比: “根据老王的线索, 凡是发现有身高七尺五寸的男性, 都要抓过来审问” 这一整个动作可以被认为是一个 Aspect.

最后是一个描述这些概念之间关系的图： 

![5.png](https://i.loli.net/2019/06/08/5cfb371156ce932322.png)

##### 其他的一些内容

AOP中的Joinpoint可以有多种类型：构造方法调用，字段的设置和获取，方法的调用，方法的执行，异常的处理执行，类的初始化。

也就是说在AOP的概念中我们可以在上面的这些Joinpoint上织入我们自定义的Advice，但是在Spring中却没有实现上面所有的joinpoint，确切的说，Spring只支持方法执行类型的Joinpoint。

**Advice 的类型**

- before advice, 在 join point 前被执行的 advice. 虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在 before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)

- after return advice, 在一个 join point 正常返回后执行的 advice
- after throwing advice, 当一个 join point 抛出异常后执行的 advice
- after(final) advice, 无论一个 join point 是正常退出还是发生了异常, 都会被执行的 advice.
- around advice, 在 join point 前和 joint point 退出后都执行的 advice. 这个是最常用的 advice.
- introduction，introduction可以为原有的对象增加新的属性和方法。

在Spring中，通过动态代理和动态字节码技术实现了AOP，这些内容，我们将在以后进行讲解。

#### 入门示例

1. 控制反转（IoC）

   ioc(inverse of controll ) 控制反转: 所谓控制反转就是把创建对象(bean),和维护对象(bean)的关系的权利从程序中转移到spring的容器(applicationContext.xml),而程序本身不再维护。之前开发中，直接new一个对象即可。学习spring之后需要实例对象时，从spring工厂（容器）中获得。

   maven依赖：

   ```xml
   
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-beans</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>${spring.version}</version>
           </dependency>
   ```

   接口和实现类：

   ```java
   public interface UserService {
   	public void addUser();
   }
   public class UserServiceImpl implements UserService {
   	@Override
   	public void addUser() {
   		System.out.println("a_ico add user");
   	}
   }
   ```

   配置文件：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans 
          					   http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="userService" class="com.hyp.learn.spring.service.impl.UserServiceImpl"/>
   </beans>
   ```

   说明：

   - 位置：任意，开发中一般在classpath下（src）
   - 名称：任意，开发中常用applicationContext.xml
   - 内容：添加schema约束，约束文件位置：spring-framework-3.2.0.RELEASE\docs\spring-framework-reference\html\ xsd-config.html

   测试：

   ```java
   @Test
   public void demo02(){
   	//从spring容器获得
   	//1 获得容器
   	String xmlPath = "applicationContext01.xml";
   	ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
   	//2获得内容 --不需要自己new，都是从spring容器获得
   	UserService userService = (UserService) applicationContext.getBean("userService");
   	userService.addUser();
   }
   ```

2. 依赖注入（DI）

   依赖：一个对象需要使用另一个对象

   注入：通过setter方法进行另一个对象实例设置。

   maven依赖：

   ```xml
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-beans</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>${spring.version}</version>
           </dependency>
   ```

   例如：

   ```java
   class BookServiceImpl{
       //之前开发：接口 = 实现类  （service和dao耦合）
   	//private BookDao bookDao = new BookDaoImpl();
   	//spring之后 （解耦：service实现类使用dao接口，不知道具体的实现类）
   	private BookDao bookDao;
   	//setter方法
   }
   ```

   说明：模拟spring执行过程

   - 创建service实例：BookService bookService = new BookServiceImpl() -->IoC  <bean\>
   - 创建dao实例：BookDao bookDao = new BookDaoImple() -->IoC
   - 将dao设置给service：bookService.setBookDao(bookDao); -->DI   <property\>

   案例：创建BookService接口和实现类，创建BookDao接口和实现类，将dao和service配置 xml文件

   其中dao为：

   ```java
   public interface BookDao {
   	public void addBook();
   }
   public class BookDaoImpl implements BookDao {
   	@Override
   	public void addBook() {
   		System.out.println("di  add book");
   	}
   }
   ```

   service为：

   ```java
   public interface BookService {
   	public abstract void addBook();
   }
   public class BookServiceImpl implements BookService {
   	// 方式1：之前，接口=实现类
   //	private BookDao bookDao = new BookDaoImpl();
   	// 方式2：接口 + setter
   	private BookDao bookDao;
   	public void setBookDao(BookDao bookDao) {
   		this.bookDao = bookDao;
   	}
   	@Override
   	public void addBook(){
   		this.bookDao.addBook();
   	}
   }
   ```

    配置文件：

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans 
          					   http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!-- 创建service -->
       <bean id="bookService" class="com.hyp.learn.spring.demo1.BookServiceImpl">
           <property name="bookDao" ref="bookDao"/>
       </bean>
   
       <!-- 创建dao实例 -->
       <bean id="bookDao" class="com.hyp.learn.spring.demo1.BookDaoImpl"/>
   </beans>
   ```

   测试：

   ```java
       @Test
       public void demo02(){
           //从spring容器获得
           String xmlPath = "applicationContext01.xml";
           ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
           BookService bookService = (BookService) applicationContext.getBean("bookService");
   
           bookService.addBook();//di  add book
       }
   
   ```
   
   

### 参考：

1. [Spring 全家桶8大知识点详解（内含Spring相关知识图谱）](https://blog.csdn.net/Alex_lagou/article/details/88642117)