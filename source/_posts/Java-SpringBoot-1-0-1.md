---
title: Spring boot 入坑
date: 2019-06-30 12:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。

首先声明，Spring Boot不是一门新技术，所以不用紧张。从本质上来说，Spring Boot就是Spring,它做了那些没有它你也会去做的Spring Bean配置。它使用 “**习惯优于配置**” （项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须）的理念让你的项目快速运行起来。

用我的话来理解，就是 Spring Boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 Maven 整合了所有的 Jar 包，Spring Boot 整合了所有的框架。

<!--more-->

代码地址：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-0-quickstart>

#### 好处

其实就是简单、快速、方便！平时如果我们需要搭建一个 Spring Web 项目的时候需要怎么做呢？

- 1）配置 web.xml，加载 Spring 和 Spring mvc
- 2）配置数据库连接、配置 Spring 事务
- 3）配置加载配置文件的读取，开启注解
- 4）配置日志文件
- …
- 配置完成之后部署 Tomcat 调试
- …

现在非常流行微服务，如果我这个项目仅仅只是需要发送一个邮件，如果我的项目仅仅是生产一个积分；我都需要这样折腾一遍!

但是如果使用 Spring Boot 呢？

 很简单，我仅仅只需要非常少的几个配置就可以迅速方便的搭建起来一套 Web 项目或者是构建一个微服务！

划重点：简单、快速、方便地搭建项目；对主流开发框架的无配置集成；极大提高了开发、部署效率。

**核心**

Spring将很多魔法带入了Spring应用程序的开发之中，其中最重要的是以下四个核心。

- 自动配置：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置
- 起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一套究竟。

#### 系统要求

- java 8+
- Gradle 4+ or Maven 3.2+
- idea

#### 快速入门

**Maven 构建项目**

1. 访问 http://start.spring.io/

2. 选择构建工具 Maven Project、Java、Spring Boot 版本以及一些工程基本信息

3. 点击 Generate Project 下载项目压缩包

4. 解压后，使用 Idea 导入项目，File -> New -> Model from Existing Source.. -> 选择解压后的文件夹 -> OK，选择 Maven 一路 Next，OK done!

5. 如果使用的是 Eclipse，Import -> Existing Maven Projects -> Next -> 选择解压后的文件夹 -> Finsh，OK done!

**Idea 构建项目**

1. 选择 File -> New —> Project… 弹出新建项目的框

2. 选择 Spring Initializr，Next 也会出现上述类似的配置界面，Idea 帮我们做了集成

3. 填写相关内容后，点击 Next 选择依赖

**项目结构介绍**

Spring Boot 的基础结构共三个文件:

- `src/main/java`  程序开发以及主程序入口
- `src/main/resources` 配置文件
- `src/test/java`  测试程序

另外， Spring Boot 的目录结果如下：

![1.png](https://i.loli.net/2019/07/10/5d25b7d94faf913102.png)

项目结构还是看上去挺清爽的，少了很多配置文件，我们来了解一下默认生成的有什么：

- SpringbootApplication： 一个带有 main() 方法的类，用于启动应用程序
- SpringbootApplicationTests：一个空的 Junit 测试了，它加载了一个使用 Spring Boot 字典配置功能的 Spring 应用程序上下文
- application.properties：一个空的 properties 文件，可以根据需要添加配置属性
- pom.xml： Maven 构建说明文件

采用默认配置可以省去很多配置，当然也可以根据自己的喜欢来进行更改

最后，启动 Application main 方法，至此一个 Java 项目搭建好了！

**基础Maven配置文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
    <!-- spring-boot-starter-parent 是一个特殊的 starter ，
它用来提供相关的 Maven 默认依赖，使用它之后，常用的包依赖就可以省去 version 标签。 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
      <!-- 核心模块，包括自动配置支持、日志和 YAML，
如果引入了 spring-boot-starter-web web 模块可以去掉此配置，
因为 spring-boot-starter-web 自动依赖了 spring-boot-starter。 
-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
  		  <!-- 测试模块，包括 JUnit、Hamcrest、Mockito -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
            <!-- 插件包 -->
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

1. 父级依赖

   有了这个，当前的项目就是Spring Boot项目了，spring-boot-starter-parent是一个特殊的starter,它用来提供相关的Maven默认依赖，使用它之后，常用的包依赖可以省去version标签。关于Spring Boot提供了哪些jar包的依赖，可查看`C:\Users\用户.m2\repository\org\springframework\boot\spring-boot-dependencies\2.1.5.RELEASE\spring-boot-dependencies-2.1.5.RELEASE.pom`或者`~/.m2/repository/org/springframework/boot/spring-boot-dependencies/2.1.5.RELEASE/spring-boot-dependencies-2.1.5.RELEASE.pom`

   如果你不想使用某个依赖默认的版本，您还可以通过覆盖自己的项目中的属性来覆盖各个依赖项，例如，要升级到另一个Spring Data版本系列，您可以将以下内容添加到pom.xml中。

   ```xml
   <properties>
       <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
   </properties>
   ```

   并不是每个人都喜欢继承自spring-boot-starter-parent POM。您可能有您需要使用的自己的公司标准parent，或者您可能更喜欢显式声明所有的Maven配置。
   如果你不想使用spring-boot-starter-parent，您仍然可以通过使用scope = import依赖关系来保持依赖关系管理：

   ```xml
   <dependencyManagement>
        <dependencies>
           <dependency>
               <!-- 他来真正管理Spring Boot应用里面的所有依赖版本
   Spring Boot的版本仲裁中心;
   以后我们导入依赖默认是不需要写版本;(没有在dependencies里面管理的依赖自然需要声明版本号)-->
               <!-- Import dependency management from Spring Boot -->
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-dependencies</artifactId>
               <version>2.2.2.RELEASE</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

   该设置不允许您使用如上所述的属性(properties)覆盖各个依赖项，要实现相同的结果，您需要在spring-boot-dependencies项之前的项目的dependencyManagement中添加一个配置,例如，要升级到另一个Spring Data版本系列，您可以将以下内容添加到pom.xml中。

   ```xml
   <dependencyManagement>
       <dependencies>
           <!-- Override Spring Data release train provided by Spring Boot -->
           <dependency>
               <groupId>org.springframework.data</groupId>
               <artifactId>spring-data-releasetrain</artifactId>
               <version>Fowler-SR2</version>
               <scope>import</scope>
               <type>pom</type>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-dependencies</artifactId>
               <version>1.5.1.RELEASE</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

2. 起步依赖

   Spring Boot提供了很多”开箱即用“的依赖模块，都是以spring-boot-starter-xx作为命名的。举个例子来说明一下这个起步依赖的好处，比如组装台式机和品牌机，自己组装的话需要自己去选择不同的零件，最后还要组装起来，期间有可能会遇到零件不匹配的问题。耗时又消力，而品牌机就好一点，买来就能直接用的，后续想换零件也是可以的。相比较之下，后者带来的效果更好点（这里就不讨论价格问题哈），起步依赖就像这里的品牌机，自动给你封装好了你想要实现的功能的依赖。就比如我们之前要实现web功能，引入了spring-boot-starter-web这个起步依赖。

   Spring Boot通过提供众多起步依赖降低项目依赖的复杂度。起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或者某类功能。

3. Maven插件

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

   上面的配置就是Spring Boot Maven插件，Spring Boot Maven插件提供了许多方便的功能：

   - 把项目打包成一个可执行的超级JAR（uber-JAR）,包括把应用程序的所有依赖打入JAR文件内，并为JAR添加一个描述文件，其中的内容能让你用java -jar来运行应用程序。
   - 搜索public static void main()方法来标记为可运行类。

**应用入口类**

Spring Boot 项目通常有一个名为 \*Application 的入口类，入口类里有一个 main 方法， **这个 main 方法其实就是一个标准的 Javay 应用的入口方法。**

**@SpringBootApplication** 是 Spring Boot 的核心注解，它是一个组合注解，该注解组合了：**@Configuration、@EnableAutoConfiguration、@ComponentScan；** 若不是用 @SpringBootApplication 注解也可以使用这三个注解代替。

@AliasFor:<https://www.jianshu.com/p/abb071165c15>

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;
}
```

- @SpringBootConfiguration:Spring Boot的配置类;标注在某个类上,表示这是一个Spring Boot的配置类;实际上是Configuration注解的别名，

  ```java
  
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Configuration
  public @interface SpringBootConfiguration {
      @AliasFor(
          annotation = Configuration.class
      )
      boolean proxyBeanMethods() default true;
  }
  
  ```

- 其中，**@EnableAutoConfiguration 让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置**，例如，添加了 spring-boot-starter-web 依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，那么 Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  //自动配置包
  @AutoConfigurationPackage
  //Spring的底层注解@Import,给容器中导入一个组件;导入的组件由AutoConfigurationPackages.Registrar.class;
  @Import(AutoConfigurationImportSelector.class)
  //将主配置类(@SpringBootApplication标注的类)的所在包及下面所有子包里面的所有组件扫描到Spring容器;
  public @interface EnableAutoConfiguration {
  
  	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
  
  	Class<?>[] exclude() default {};
  
  	String[] excludeName() default {};
  
  }
  
  
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  //EnableAutoConfigurationImportSelector:导入哪些组件的选择器;
  //将所有需要导入的组件以全类名的方式返回;这些组件就会被添加到容器中;
  //会给容器中导入非常多的自动配置类(xxxAutoConfiguration);就是给容器中导入这个场景需要的所有组件,并配置好这些组件;
  @Import(AutoConfigurationPackages.Registrar.class)
  public @interface AutoConfigurationPackage {
  
  }
  
  //有了自动配置类,免去了我们手动编写配置注入功能组件等的工作;
  SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
  				getBeanClassLoader());
  protected Class<?> getSpringFactoriesLoaderFactoryClass() {
  		return EnableAutoConfiguration.class;
  	}
  	protected ClassLoader getBeanClassLoader() {
  		return this.beanClassLoader;
  	}
  ```

- AutoConfigurationImportSelector会加载所有配置的自动配置类

- Spring Boot 还会自动扫描 @SpringBootApplication 所在类的同级包以及下级包里的 Bean** ，所以入口类建议就配置在 grounpID + arctifactID 组合的包名下（这里为 cn.wmyskxz.springboot 包）

- Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值,将这些值作为自动配置类导入到容器中,自动配置类就生效,帮我们进行自动配置工作;以前我们需要自己配置的东西,自动配置类都帮我们;

- J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-<version\>.jar;

SpringBoot默认包扫描机制是：**从启动类所在包开始，扫描当前包及其子包下的所有文件。**

**Spring Boot 的配置文件**

Spring Boot 使用一个全局的配置文件 application.properties 或 application.yml，放置在【src/main/resources】目录或者类路径的 /config 下。

Spring Boot 不仅支持常规的 properties 配置文件，还支持 yaml 语言的配置文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征。

Spring Boot 的全局配置文件的作用是对一些默认配置的配置值进行修改。

yml 需要在 “`:`” 后加一个空格，幸好 IDEA 很好地支持了 yml 文件的格式有良好的代码提示.

我们直接把 .properties 后缀的文件删掉，使用 .yml 文件来进行简单的配置，然后使用 @Value 来获取配置属性。

也可以在配置文件中使用当前配置

**注意：** 我们并没有在 yml 文件中注明属性的类型，而是在使用的时候定义的。

#### 开发一个web服务

没有比较就没有伤害，让我们先看看传统Spring MVC开发一个简单的Hello World Web应用程序，你应该做什么，我能想到一些基本的需求。

- 一个项目结构，其中有一个包含必要依赖的Maven或者Gradle构建文件，最起码要有Spring MVC和Servlet API这些依赖。
- 一个web.xml文件（或者一个WebApplicationInitializer实现），其中声明了Spring的DispatcherServlet。
- 一个启动了Spring MVC的Spring配置
- 一控制器类，以“hello World”相应HTTP请求。
- 一个用于部署应用程序的Web应用服务器，比如Tomcat。

最让人难以接受的是，这份清单里面只有一个东西是和Hello World功能相关的，即控制器，剩下的都是Spring开发的Web应用程序必需的通用模板。

接下来看看Spring Boot如何搞定？
很简单，我仅仅只需要非常少的几个配置就可以迅速方便的搭建起来一套web项目

**引入 web 模块**

1. pom.xml中添加支持web的模块：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
```

2. 编写 Controller 内容

```java
@RestController
public class HelloWorldController {
    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
}
```

`@RestController` 的意思就是 Controller 里面的方法都以 json 格式输出，不用再写什么 jackjson 配置的了！

3. 启动主程序，打开浏览器访问 `http://localhost:8080/quickstart/hello`，就可以看到效果了，有木有很简单！

**单元测试**

打开的`src/test/`下的测试入口，编写简单的 http 请求来测试；使用 mockmvc 进行，利用`MockMvcResultHandlers.print()`打印出执行结果。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloTests {

  
    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
    }

    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello World")));
    }

}

```

**开发环境的调试**

热启动在正常开发项目中已经很常见了吧，虽然平时开发web项目过程中，改动项目启重启总是报错；但springBoot对调试支持很好，修改之后可以实时生效，需要添加以下的配置：

```xml
 <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
</plugins>
</build>
```

该模块在完整的打包环境下运行的时候会被禁用。如果你使用 `java -jar`启动应用或者用一个特定的 classloader 启动，它会认为这是一个“生产环境”。

#### 常见包

**起步依赖**

![](https://i.loli.net/2019/07/12/5d27e00cc76e611929.png)

**其他包**

| Name                                   | Description                                                  | 备注                                                         |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| spring-boot-starter-thymeleaf          | 使MVC Web applications 支持Thymeleaf                         | Thymeleaf是一个JAVA库，一个XML/XHTML/HTML5的可扩展的模板引擎，同类事物：Jsp |
| spring-boot-starter-data-couchbase     | 使用Couchbase 文件存储数据库、Spring Data Couchbase          | Spring Data是一个用于简化数据库访问，并支持云服务的开源框架  |
| spring-boot-starter-artemis            | 为JMS messaging使用Apache Artemis                            | JMS是Java消息服务；HornetQ代码库捐献给 Apache ActiveMQ 社区，它现在成为ActiveMQ旗下的一个子项目，名为 “Artemis” |
| spring-boot-starter-web-services       | 使用Spring Web Services                                      | Spring Web Services是基于Spring框架的Web服务框架，主要侧重于基于文档驱动的Web服务，提供SOAP服务开发，允许通过多种方式创建 Web 服务。 |
| spring-boot-starter-mail               | 使用Java Mail、Spring email发送支持                          | Java Mail、Spring email为邮件发送工具                        |
| spring-boot-starter-data-redis         | 通过Spring Data Redis 、Jedis client使用Redis键值存储数据库  | Jedis 是 Redis 官方首选的 Java 客户端开发包                  |
| spring-boot-starter-web                | 构建Web，包含RESTful风格框架SpringMVC和默认的嵌入式容器Tomcat | RESTful是一种软件架构风格，设计风格而不是标准，只是提供了一组设计原则和约束条件 |
| spring-boot-starter-activemq           | 为JMS使用Apache ActiveMQ                                     | ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线      |
| spring-boot-starter-data-elasticsearch | 使用Elasticsearch、analytics engine、Spring Data Elasticsearch | ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口 |
| spring-boot-starter-integration        | 使用Spring Integration                                       | Spring Integration是Spring框架创建的一个API，面向企业应用集成（EAI） |
| spring-boot-starter-test               | 测试 Spring Boot applications包含JUnit、 Hamcrest、Mockito   | JUnit、 Hamcrest、Mockito为测试框架                          |
| spring-boot-starter-jdbc               | 通过 Tomcat JDBC 连接池使用JDBC                              |                                                              |
| spring-boot-starter-mobile             | 通过Spring Mobile构建Web应用                                 | Spring Mobile 是 Spring MVC 的扩展,用来简化手机上的Web应用开发 |
| spring-boot-starter-validation         | 通过Hibernate Validator使用 Java Bean Validation             | Bean Validation 是一个数据验证的规范；Hibernate Validator是一个数据验证框架 |
| spring-boot-starter-hateoas            | 使用Spring MVC、Spring HATEOAS构建 hypermedia-based RESTful Web 应用 | hypermedia-based似乎是专业术语，博主表示不会翻译；Spring HATEOAS 是一个用于支持实现超文本驱动的 REST Web 服务的开发库 |
| spring-boot-starter-jersey             | 通过 JAX-RS、Jersey构建 RESTful web applications；spring-boot-starter-web的另一替代方案 | JAX-RS是JAVA EE6 引入的一个新技术；Jersey不仅仅是一个JAX-RS的参考实现，Jersey提供自己的API，其API继承自JAX-RS，提供更多的特性和功能以进一步简化RESTful service和客户端的开发 |
| spring-boot-starter-data-neo4j         | 使用Neo4j图形数据库、Spring Data Neo4j                       | Neo4j是一个高性能的,NOSQL图形数据库，它将结构化数据存储在网络上而不是表中 |
| spring-boot-starter-websocket          | 使用Spring WebSocket构建 WebSocket 应用                      | Websocket是一个持久化的协议，相对于HTTP这种非持久的协议来说  |
| spring-boot-starter-aop                | 通过Spring AOP、AspectJ面向切面编程                          | AspectJ是一个面向切面的框架，它扩展了Java语言                |
| spring-boot-starter-amqp               | 使用Spring AMQP、Rabbit MQ                                   | Spring AMQP 是基于 Spring 框架的 AMQP 消息解决方案,提供模板化的发送和接收消息的抽象层,提供基于消息驱动的 POJO；RabbitMQ是一个在AMQP基础上完整的，可复用的企业消息系统 |
| spring-boot-starter-data-cassandra     | 使用Cassandra分布式数据库、Spring Data Cassandra             | Apache Cassandra是一套开源分布式NoSQL数据库系统              |
| spring-boot-starter-social-facebook    | 使用 Spring Social Facebook                                  | Facebook提供用户使用第三方社交网络的账号API，同类事物：QQ第三方登录接口 |
| spring-boot-starter-jta-atomikos       | 为 JTA 使用 Atomikos                                         | JTA，即Java Transaction API，JTA允许应用程序执行分布式事务处理；Atomikos 是一个为Java平台提供增值服务的并且开源类事务管理 |
| spring-boot-starter-security           | 使用 Spring Security                                         | Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架 |
| spring-boot-starter-mustache           | 使MVC Web applications 支持Mustache                          | Mustache是基于JavaScript实现的模版引擎，类似于jQuery Template，但是这个模版更加的轻量级，语法更加的简单易用，很容易上手 |
| spring-boot-starter-data-jpa           | 通过 Hibernate 使用 Spring Data JPA （Spring-data-jpa依赖于Hibernate） | JPA全称Java Persistence API.JPA通过JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中 |
| spring-boot-starter                    | Core starter,包括 自动配置支持、 logging and YAML            | logging是指的Starter的专有框架；YAML是“另一种标记语言”的外语缩写，它参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822 |
| spring-boot-starter-groovy-templates   | 使MVC Web applications 支持Groovy Templates                  | Groovy Templates是模视图模板，同类事物：Jsp                  |
| spring-boot-starter-freemarker         | 使MVC Web applications 支持 FreeMarker                       | FreeMarker是模视图模板，同类事物：Jsp                        |
| spring-boot-starter-batch              | 使用Spring Batch                                             | Spring Batch是一个轻量级的,完全面向Spring的批处理框架,可以应用于企业级大量的数据处理系统 |
| spring-boot-starter-social-linkedin    | 使用Spring Social LinkedIn                                   | LinkedIn提供用户使用第三方社交网络的账号API，同类事物：QQ第三方登录接口 |
| spring-boot-starter-cache              | 使用 Spring caching 支持                                     | Spring caching是Spring的提供的缓存框架                       |
| spring-boot-starter-data-solr          | 通过 Spring Data Solr 使用 Apache Solr                       | Apache Solr 是一个开源的搜索服务器。Solr 使用 Java 语言开发，主要基于 HTTP 和 Apache Lucene 实现 |
| spring-boot-starter-data-mongodb       | 使用 MongoDB 文件存储数据库、Spring Data MongoDB             | Spring Data是一个用于简化数据库访问，并支持云服务的开源框架  |
| spring-boot-starter-jooq               | 使用JOOQ链接SQL数据库；spring-boot-starter-data-jpa、spring-boot-starter-jdbc的另一替代方案 | jOOQ（Java Object Oriented Querying，即面向Java对象查询）是一个高效地合并了复杂SQL、类型安全、源码生成、ActiveRecord、存储过程以及高级数据类型的Java API的类库。 |
| spring-boot-starter-jta-narayana       | Spring Boot Narayana JTA Starter                             | 似乎和jboss.narayana.jta有关                                 |
| spring-boot-starter-cloud-connectors   | 用连接简化的 Spring Cloud 连接器进行云服务就像Cloud Foundry、Heroku那样 | Cloud Foundry是VMware推出的业界第一个开源PaaS云平台；Heroku是一个支持多种编程语言的云平台即服务 |
| spring-boot-starter-jta-bitronix       | 为JTA transactions 使用 Bitronix                             | Bitronix Transaction Manager (BTM) 是一个简单但完整实现了 JTA 1.1 API 的类库，完全支持 XA 事务管理器，提供 JTA API 所需的所有服务，并让代码保持简洁 |
| spring-boot-starter-social-twitter     | 使用 Spring Social Twitter                                   | Twitter提供用户使用第三方社交网络的账号API，同类事物：QQ第三方登录接口 |
| spring-boot-starter-data-rest          | 使用Spring Data REST 以 REST 方式暴露 Spring Data repositories | 博主也不是很明白。原文：exposing Spring Data repositories over REST using Spring Data REST |

**容器依赖**

![](https://i.loli.net/2019/07/12/5d27df332370c46474.png)

#### 总结

当然，秉承对技术的执着，既要认可新技术的改革，也要小心求证，扬长避短。以SpringBoot为例，目前国内项目应用广泛的连接池是阿里的Druid。很遗憾，springboot暂未对该连接池提供支持，要想在springboot工程中使用阿里的连接池，你得使用springboot自定义配置（定义驱动、连接地址、用户名、密码等属性），再生成一个JavaBean，再将配置属性注入到bean中，还要注入启动加载等等系列工作，最后发现代码多了一大堆。再如，如果将SpringBoot工程部署到weblogic容器又得面临一大堆兼容问题，还好SpringBoot官方提供了补丁包。这只是简单举例，要想在大型项目中使用SpringBoot还有更长的路要走，因为大工程牵连体系庞大，需要技术兼容和稳定性也更高。

> 小提示:SpringBoot目前支持的连接池有HikariCP、DBCP 、DBCP2 。

#### 附录A.配置文件

当我们创建一个springboot项目的时候，系统默认会为我们在src/main/java/resources目录下创建一个application.properties。一般我会将application.properties改为application.yml文件进行使用，两种文件格式都支持。

**自定义属性**

在一些情况下，有些参数我们需要它不是一个固定的值，比如密钥、服务端口等。

Spring Boot的属性配置文件中可以通过${random}来产生int值、long值、string字符串或者UUID，来支持属性的随机值。从配置文件中获取符合规则的随机数。随机值会在程序启动时确定。

在application.yml自定义一组属性：

```properties
#自定义属性与加载  cn.hyp.conf.DefinitionConfig.java
cn.hyp.name=PROGRAMMER
cn.hyp.title=Spring Boot
#在application.properties中的各个参数之间也可以直接引用来使用
cn.hyp.desc=${cn.hyp.name}is code refactoring << ${cn.hyp.title} with v2.2.2.RELEASE >>
# 随机字符串，32位MD5字符串
cn.hyp.value=${random.value}
# 随机uuid
#cn.hyp.pazzword=${random.uuid}
# 随机int
cn.hyp.number=${random.int}
# 随机long
cn.hyp.bignumber=${random.long}
# 10以内的随机数
cn.hyp.random1=${random.int(10)}
# 10-20的随机数
cn.hyp.random2=${random.int[10,20]}

# 取引用变量的值，若不存在时，取默认值
#cn.hyp.map.key1=${user.username:默认值1}
# 变量不存在时，取默认值
#cn.hyp.map.key2=${abcd:默认值2}
```

如果需要读取配置文件的值只需要加@Value(“${属性名}”)，案例：

```java
/**
 * @author hyp
 * 自定义属性与加载:只需要使用@Component注解将类注册到容器内就可以方便使用
 */
@Component
public class DefinitionConfig implements Serializable{

	private static final long serialVersionUID = 7063410079055294317L;

	@Value("${cn.hyp.name}")
	private String name;

	@Value("${cn.hyp.title}")
	private String title;

	@Value("${cn.hyp.desc}")
	private String desc;

	@Value("${cn.hyp.value}")
	private String value;

	@Value("${cn.hyp.number}")
	private Integer number;

	@Value("${cn.hyp.bignumber}")
	private Long bignumber;

	@Value("${cn.hyp.random1}")
	private Integer random1;

	@Value("${cn.hyp.random2}")
	private Integer random2;

	@Override
	public String toString() {
		return "DefinitionConfig [name=" + name + ", title=" + title + ", desc=" + desc + ", value=" + value + ", number=" + number + ", bignumber=" + bignumber + ", random1=" + random1 + ", random2=" + random2 + "]";
	}
	//get set

}


@RestController
public class ConfController {
	
	@Resource
	DefinitionConfig conf;
	
	/**
	 * 测试自定义配置属性加载
	 * @return
	 */
	@RequestMapping("/conf")
	public String getConfig() {
		System.out.println(conf.toString());
		return conf.toString();
	}
}
```

启动工程，访问：localhost:8080/quickstart/conf,返回结果.

**将配置文件的属性赋给实体类**

有时候我们会把配置文件中的属性赋值给一个类,上述更改在DefinitionConfig上添加即可，这样就不用每个属性上都添加Value注解,另外@Component可加可不加。原理很简单，利用反射技术

```
@ConfigurationProperties(prefix = "cn.hyp")
```

启动工程，访问：localhost:8080/quickstart/conf,返回结果.

```xml
<!‐‐导入配置文件处理器,配置文件进行绑定就会有提示‐‐>
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring‐boot‐configuration‐processor</artifactId>
<optional>true</optional>
</dependency>
```

比较：

![lCwqGn.png](https://s2.ax1x.com/2019/12/24/lCwqGn.png)

如果说,我们只是在某个业务逻辑中需要获取一下配置文件中的某项值,使用@Value;

如果说,我们专门编写了一个javaBean来和配置文件进行映射,我们就直接使用@ConfigurationProperties;

**自定义配置文件映射值到实体类**

自定义dubbo.properties文件

```properties
##自定义属性与加载  
dubbo.resAddress=zookeeper://10.10.10.30:2181?backup=10.10.10.40:2181,10.10.10.45:2181
dubbo.appName=dsc-core
dubbo.resUsername=
dubbo.resPassowrd=
dubbo.protocol=dubbo
dubbo.port=57777
dubbo.monAddress=
dubbo.accepts=700
dubbo.connections=2
```

自定义实体类，注意实体类需要以下三个注解：

```java
@Configuration
@PropertySource("classpath:dubbo.properties")
//或者不使用@Value
//@ConfigurationProperties(prefix = "dubbo")
public class DubboConfig {

	@Value("${dubbo.resAddress}")
	private String resAddress;

	@Value("${dubbo.appName}")
	private String appName;

	@Value("${dubbo.resUsername}")
	private String resUsername;

	@Value("${dubbo.resPassowrd}")
	private String resPassowrd;

	@Value("${dubbo.protocol}")
	private String protocol;

	@Value("${dubbo.port}")
	private int port;

	@Value("${dubbo.accepts}")
	private int accepts;

	@Value("${dubbo.connections}")
	private int connections;

	@Override
	public String toString() {
		return "DubboConfig [resAddress=" + resAddress + ", appName=" + appName + ", resUsername=" + resUsername + ", resPassowrd=" + resPassowrd + ", protocol=" + protocol + ", port=" + port + ", accepts=" + accepts + ", connections=" + connections + "]";
	}
}

@RestController
public class ConfController {
	
	@Resource
	DubboConfig dubbo;

	
	/**
	 * 测试自定义的额外文件的配置信息
	 * @return
	 */
	@RequestMapping("/dubbo")
	public String dubboConfig() {
		System.out.println(dubbo.toString());
		return dubbo.toString();
	}
}
```

**多个环境配置文件**

在实际的开发和使用的时候，我们需要不同的配置环境；格式为application-{profile}.properties，其中{profile}对应你的环境标识，比如：

```
application-dev.properties：开发环境

application-test.properties：测试环境

application-prod.properties：生产环境
```

那么怎么使用？我们只需在application.yml中加：

```
#指定环境为dev
spring.profiles.active=dev
```

或者在application.yml中使用

```
spring.profiles.active=@profiles.active@
```

当执行`mvn clean package -P test `命令时， @profiles.active@ 会替换成 test

**@PropertySource&@ImportResource&@Bean**

1. @PropertySource:加载指定的配置文件;

   ```java
   @PropertySource(value = {"classpath:person.properties"})
   ```

2. @ImportResource:导入Spring的配置文件,让配置文件里面的内容生效;Spring Boot里面没有Spring的配置文件,我们自己编写的配置文件,也不能自动识别;想让Spring的配置文件生效,加载进来;@ImportResource标注在一个配置类上

   ```java
   @ImportResource(locations = {"classpath:beans.xml"})
   //导入Spring的配置文件让其生效
   ```

   SpringBoot推荐给容器中添加组件的方式;推荐使用全注解的方式

   1. 配置类@Configuration------>Spring配置文件
   2. 使用@Bean给容器中添加组件

**加载顺序**

1. springboot配置文件的加载位置

   springboot启动会扫描一下位置的application.properties或者application.yml作为默认的配置文件

   - 当前项目目录下的一个/config子目录
   - 当前项目目录
   - 项目的resources即一个classpath下的/config包
   - 项目的resources即classpath根路径（root）

   ![Qv10Ve.png](https://s2.ax1x.com/2019/12/21/Qv10Ve.png)

   加载的优先级顺序是从上向下加载，并且所有的文件都会被加载，高优先级的内容会覆盖底优先级的内容，形成互补配置

   也可以通过指定配置**spring.config.location**来改变默认配置，一般在项目已经打包后，我们可以通过指令`java -jar xxxx.jar --spring.config.location=D:/kawa/application.yml`来加载外部的配置

   在**工程根路径下或者根路径的config下面的配置文件，在工程打包时候不会被打包进去**

2. springboot外部配置的加载顺序

   springboot外部配置加载顺序如下，优先级从高到底，并且高优先级的配置覆盖底优先级的配置形成互补配置

   1. 命令行参数
      => 比如：java -jar xxxx.jar --server.port=8087 --server.context-path=/show 多个配置中间用空格分开，由jar包外向jar包内进行加载，比如和工程平级目录下面的配置文件优先级高于jar包内部的配置文件

   2. 来自java:comp/env的JNDI属性

   3. Java系统属性(System.getProperties())

   4. 操作系统环境变量

   5. RandomValuePropertySource配置的random.*属性值

      **由jar包外向jar包内进行寻找;优先加载带profile**

   6. jar包外部的application-{profile}.propertie或application.yml(带spring.profile)配置文件           

   7. jar包内部的application-{profile}.propertie或application.yml(带spring.profile)配置文件

      **再来加载不带profile**

   8. jar包外部的application.propertie或application.yml(不带spring.profile)配置文件

   9. jar包内部的application.propertie或application.yml(不带spring.profile)配置文件

   10. @Configuration注解类上的@PropertySource

   11. 通过SpringApplication.setDefaultProperties指定的默认属性

   所有支持的配置加载来源：[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)

3. 读取顺序

   如果在不同的目录中存在多个配置文件，它的读取顺序是：

   - config/application.properties（项目根目录中config目录下）
   - config/application.yml
   - application.properties（项目根目录下）
   - application.yml
   - resources/config/application.properties（项目resources目录中config目录下
   - resources/config/application.yml
   - resources/application.properties（项目的resources目录下）
   - resources/application.yml

   如果同一个配置属性，在多个配置文件都配置了，默认使用第1个读取到的，后面读取的不覆盖前面读取到的。

   创建SpringBoot项目时，一般的配置文件放置在项目的resources目录下，因为配置文件的修改，通过热部署不用重新启动项目，而热部署的作用范围是classpath下

4. 配置文件的生效顺序，会对值进行覆盖

   - @TestPropertySource 注解
   - 命令行参数
   - Java系统属性（System.getProperties()）
   - 操作系统环境变量
   - 只有在`random.*`里包含的属性会产生一个RandomValuePropertySource
   - 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
   - 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
   - 在@Configuration类上的@PropertySource注解
   - 默认属性（使用SpringApplication.setDefaultProperties指定）

5. 其他配置

   - 端口配置
      `server.port=8090`
- 时间格式化
      `spring.jackson.date-format=yyyy-MM-dd HH:mm:ss`
   - 时区设置
   `spring.jackson.time-zone=Asia/Chongqing`

**多模块中的共用配置文件**

公共模块comm有些参数是配置文件里配置的，其他的应用依赖comm包，这样一来每个应用都需要配置一个与comm相同的参数才行

![](https://s3.ax1x.com/2020/12/27/rIltXR.png)

application.yml

```yaml
spring:
 profiles:
  include: commdev,dev
```

经测试，dev也就是说上层的配置文件要放在后面，他会覆盖前面的相同参数，如果后面的配置文件里没有配置，就采用commdev公共模块的共用参数

#### docker打包

在pom.xml中添加 Docker 镜像名称

```
<properties>
	<docker.image.prefix>springboot</docker.image.prefix>
</properties>
```

plugins 中添加 Docker 构建插件：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
		<!-- Docker maven plugin -->
		<plugin>
			<groupId>com.spotify</groupId>
			<artifactId>docker-maven-plugin</artifactId>
			<version>1.0.0</version>
			<configuration>
				<imageName>${docker.image.prefix}/${project.artifactId}</imageName>
				<dockerDirectory>src/main/docker</dockerDirectory>
				<resources>
					<resource>
						<targetPath>/</targetPath>
						<directory>${project.build.directory}</directory>
						<include>${project.build.finalName}.jar</include>
					</resource>
				</resources>
			</configuration>
		</plugin>
		<!-- Docker maven plugin -->
	</plugins>
</build>
```

在目录`src/main/docker`下创建 Dockerfile 文件，Dockerfile 文件用来说明如何来构建镜像。

```dockerfile
FROM openjdk:8-jdk-alpine
MAINTAINER hyp
LABEL app="myApp" version="0.0.1" by="hyp"
COPY spring-boot-0-quickstart.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

将项目 `spring-boot-docker` 拷贝服务器中，进入项目路径下进行打包测试。

```
#打包
mvn package
#启动
java -jar target/spring-boot-0-quickstart.jar
```

看到 Spring Boot 的启动日志后表明环境配置没有问题，接下来我们使用 DockerFile 构建镜像。

```
mvn package docker:build
```

第一次构建可能有点慢，当看到以下内容的时候表明构建成功：

```
Step 1/5 : FROM openjdk:8-jdk-alpine

 ---> a3562aa0b991
Step 2/5 : MAINTAINER hyp

 ---> Using cache
 ---> 941ddaa2100f
Step 3/5 : LABEL app="myApp" version="0.0.1" by="hyp"

 ---> Using cache
 ---> fa97f86f9b72
Step 4/5 : COPY spring-boot-0-quickstart.jar app.jar

 ---> f0c40b811c0a
Step 5/5 : ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

 ---> Running in 3e92d80df02b
Removing intermediate container 3e92d80df02b
 ---> 4e64300d6202
ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}
Successfully built 4e64300d6202
Successfully tagged springboot/spring-boot-0-quickstart:latest
[INFO] Built springboot/spring-boot-0-quickstart
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11.182 s
[INFO] Finished at: 2020-01-26T21:11:30+08:00
[INFO] ------------------------------------------------------------------------

```

使用`docker images`命令查看构建好的镜像,`springboot/spring-boot-0-quickstart` 就是我们构建好的镜像，下一步就是运行该镜像

```shell
docker run -p 8080:8080 -t springboot/spring-boot-0-quickstart 
```

#### 参考：

1. [嘟嘟独立博客](http://tengj.top/categories/Spring-Boot%E5%B9%B2%E8%B4%A7%E7%B3%BB%E5%88%97/)
2. [SpringBoot配置文件——加载顺序](https://www.jianshu.com/p/3c615bd42799)