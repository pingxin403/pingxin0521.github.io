---
title: Apache Tomcat 源码 (零) 
date: 2019-10-31 11:18:59
tags:
 - 服务器
 - Tomcat
categories:
 - 服务器
 - Tomcat
---
Tomcat作为J2EE的开源实现，其代码具有很高的参考价值，我们可以从中汲取很多的知识。作为Java后端程序员，相信有很多人很想了解Tomcat的运行原理。Tomcat的构建是基于Ant和Eclipse的，然而现在很多人都喜欢IDEA+Maven的项目构建方式，本文描述了在Ubuntu 18.04 的环境下，使用IDEA导入Tomcat 9.0.35源码，并运行tomcat源码。

![t12eBt.png](https://s1.ax1x.com/2020/05/31/t12eBt.png)

<!--more-->

**准备条件**

1. Tomcat源码下载地址：<http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.55/src/apache-tomcat-8.5.55-src.tar.gz>
2. IDEA工具：https://www.jetbrains.com/idea/download
3. MAVEN：http://maven.apache.org/download.cgi
4. JDK：自然不用多提了，但是要按照所选源码要求的版本，这里用的是JDK8

安装和下载这些软件包就可以开始搭建调试环境了。

**项目结构**

将下载下来的apache-tomcat-8.5.55-src.tar.gz解压，然后在解压后的apache-tomcat-8.5.55-src目录中新建catalina-home目录和pom.xml文件
apache-tomcat-8.5.55-src目录中的conf和webapps文件夹复制到catalina-home目录中

```bash
$ mkdir catalina-home/{lib,logs,temp,work} -p
$ touch pom.xml
$ mv conf webapps -t catalina-home 
```

pom文件内容如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>Tomcat8.0</artifactId>
    <name>Tomcat8.0</name>
    <version>8.0</version>
 
    <build>
        <finalName>Tomcat8.0</finalName>
        <sourceDirectory>java</sourceDirectory>
        <testSourceDirectory>test</testSourceDirectory>
        <resources>
            <resource>
                <directory>java</directory>
            </resource>
        </resources>
        <testResources>
           <testResource>
             <!--   <directory>test</directory> -->
           </testResource>
        </testResources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
 
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.0</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.5.1</version>
        </dependency>
       
    </dependencies>
</project>
```

实验Idea导入pom

添加Application，Main class设置为org.apache.catalina.startup.Bootstrap

添加VM options 

```
-Dcatalina.home=catalina-home
-Dcatalina.base=catalina-home
-Djava.endorsed.dirs=catalina-home/endorsed
-Djava.io.tmpdir=catalina-home/temp
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=catalina-home/conf/logging.properties
-Dfile.encoding=UTF-8
```

tomcat的源码org.apache.catalina.startup.ContextConfig中的configureStart函数中手动将JSP解析器初始化：

```java
protected synchronized void configureStart() {
        // Called from StandardContext.start()

        if (log.isDebugEnabled()) {
            log.debug(sm.getString("contextConfig.start"));
        }

        if (log.isDebugEnabled()) {
            log.debug(sm.getString("contextConfig.xmlSettings",
                    context.getName(),
                    Boolean.valueOf(context.getXmlValidation()),
                    Boolean.valueOf(context.getXmlNamespaceAware())));
        }

        webConfig();
//添加这个
        context.addServletContainerInitializer(new JasperInitializer(), null);
        
        if (!context.getIgnoreAnnotations()) {
            applicationAnnotationsConfig();
        }
        if (ok) {
            validateSecurityRoles();
        }

        // Configure an authenticator if we need one
        if (ok) {
            authenticatorConfig();
        }

        // Dump the contents of this pipeline if requested
        if (log.isDebugEnabled()) {
            log.debug("Pipeline Configuration:");
            Pipeline pipeline = context.getPipeline();
            Valve valves[] = null;
            if (pipeline != null) {
                valves = pipeline.getValves();
            }
            if (valves != null) {
                for (int i = 0; i < valves.length; i++) {
                    log.debug("  " + valves[i].getClass().getName());
                }
            }
            log.debug("======================");
        }

        // Make our application available if no problems were encountered
        if (ok) {
            context.setConfigured(true);
        } else {
            log.error(sm.getString("contextConfig.unavailable"));
            context.setConfigured(false);
        }

    }
```

运行application，访问<127.0.0.1:8080>



### 架构

#### Http工作原理

HTTP协议是浏览器与服务器之间的数据传送协议。作为应用层协议,HTTP是基于TCP/IP协议来传递数据的(HTML文件、图片、查询结果等),HTTP协议不涉及数据包(Packet)传输,主要规定了客户端和服务器之间的通信格式。

![t1Wxns.png](https://s1.ax1x.com/2020/05/31/t1Wxns.png)

从图上你可以看到,这个过程是:

1. 用户通过浏览器进行了一个操作,比如输入网址并回车,或者是点击链接,接着浏览
   器获取了这个事件。
2. 浏览器向服务端发出TCP连接请求。
3. 服务程序接受浏览器的连接请求,并经过TCP三次握手建立连接。
4. 浏览器将请求数据打包成一个HTTP协议格式的数据包。
5. 浏览器将该数据包推入网络,数据包经过网络传输,最终达到端服务程序。
6. 服务端程序拿到这个数据包后,同样以HTTP协议格式解包,获取到客户端的意图。
7. 得知客户端意图后进行处理,比如提供静态文件或者调用服务端程序获得动态结果。
8. 服务器将响应结果(可能是HTML或者图片等)按照HTTP协议格式打包。
9. 服务器将响应数据包推入网络,数据包经过网络传输最终达到到浏览器。
10. 浏览器拿到数据包后,以HTTP协议的格式解包,然后解析数据,假设这里的数据是
    HTML。
11. 浏览器将HTML文件展示在页面上。

那我们想要探究的Tomcat作为一个HTTP服务器,在这个过程中都做了些什么事情呢?主要是接受连接、解析请求数据、处理请求和发送响应这几个步骤。

#### Http服务器请求处理

浏览器发给服务端的是一个HTTP格式的请求,HTTP服务器收到这个请求后,需要调用服务端程序来处理,所谓的服务端程序就是你写的Java类,一般来说不同的请求需要由不同的Java类来处理。

![t1fZu9.png](https://s1.ax1x.com/2020/05/31/t1fZu9.png)

1. 图1 , 表示HTTP服务器直接调用具体业务类,它们是紧耦合的。
2. 图2,HTTP服务器不直接调用业务类,而是把请求交给容器来处理,容器通过Servlet接口调用业务类。因此Servlet接口和Servlet容器的出现,达到了HTTP服务器与业务类解耦的目的。而Servlet接口和Servlet容器这一整套规范叫作Servlet规范。Tomcat按照Servlet规范的要求实现了Servlet容器,同时它们也具有HTTP服务器的功能。作为Java程序员,如果我们要实现新的业务功能,只需要实现一个Servlet,并把它注册到Tomcat(Servlet容器)中,剩下的事情就由Tomcat帮我们处理了。

#### Servlet容器工作流程

为了解耦,HTTP服务器不直接调用Servlet,而是把请求交给Servlet容器来处理,那Servlet容器又是怎么工作的呢?

当客户请求某个资源时,HTTP服务器会用一个ServletRequest对象把客户的请求信息封装起来,然后调用Servlet容器的service方法,Servlet容器拿到请求后,根据请求的URL和Servlet的映射关系,找到相应的Servlet,如果Servlet还没有被加载,就用反射机制创建这个Servlet,并调用Servlet的init方法来完成初始化,接着调用Servlet的service方法来处理请求,把ServletResponse对象返回给HTTP服务器,HTTP服务器会把响应发送给客户端。

![t1fuAx.png](https://s1.ax1x.com/2020/05/31/t1fuAx.png)

#### Tomcat 整体架构

我们知道如果要设计一个系统,首先是要了解需求,我们已经了解了Tomcat要实现两个核心功能:

1. 处理Socket连接,负责网络字节流与Request和Response对象的转化。
2. 加载和管理Servlet,以及具体处理Request请求。

因此Tomcat设计了两个核心组件连接器(Connector)和容器(Container)来分别做这两件事情。连接器负责对外交流,容器负责内部处理。

![t1fW5V.png](https://s1.ax1x.com/2020/05/31/t1fW5V.png)

#### 连接器 - Coyote

Coyote 是Tomcat的连接器框架的名称 , 是Tomcat服务器提供的供客户端访问的外部接口。客户端通过Coyote与服务器建立连接、发送请求并接受响应 。

Coyote 封装了底层的网络通信(Socket 请求及响应处理),为Catalina 容器提供了统一的接口,使Catalina 容器与具体的请求协议及IO操作方式完全解耦。Coyote 将Socket 输入转换封装为 Request 对象,交由Catalina 容器进行处理,处理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流 。Coyote 作为独立的模块,只负责具体协议和IO的相关操作, 与Servlet 规范实现没有直接关系,因此即便是 Request 和 Response 对象也并未实现Servlet规范对应的接口, 而是在Catalina 中将他们进一步封装为ServletRequest 和 ServletResponse 。

![t1hCqA.png](https://s1.ax1x.com/2020/05/31/t1hCqA.png)

**IO 模型与协议**

在Coyote中 , Tomcat支持的多种I/O模型和应用层协议,具体包含哪些IO模型和应用层协议,请看下:

> Tomcat 支持的IO模型(自8.5/9.0 版本起,Tomcat 移除了 对 BIO 的支持):

- NIO 非阻塞I/O,采用Java NIO类库实现。
- NIO2 异步I/O,采用JDK 7最新的NIO2类库实现。
- APR 采用Apache可移植运行库实现,是C/C++编写的本地库。如果选择该方案,需要单独安装APR库。

Tomcat 支持的应用层协议 :

- HTTP/1.1：这是大部分Web应用采用的访问协议。
- AJP：用于和Web服务器集成(如Apache),以实现对静态资源的优化以及集群部署,当前支持AJP/1.3。
- HTTP/2：HTTP 2.0大幅度的提升了Web性能。下一代HTTP协议 , 自8.5以及9.0版本之后支持。

协议分层 :

![t1hLLj.png](https://s1.ax1x.com/2020/05/31/t1hLLj.png)

在 8.0 之前 , Tomcat 默认采用的I/O方式为 BIO , 之后改为 NIO。 无论 NIO、NIO2还是 APR, 在性能方面均优于以往的BIO。 如果采用APR, 甚至可以达到 Apache HTTP Server 的影响性能。

Tomcat 为了实现支持多种I/O模型和应用层协议,一个容器可能对接多个连接器,就好比一个房间有多个门。但是单独的连接器或者容器都不能对外提供服务,需要把它们组装起来才能工作,组装后这个整体叫作Service组件。这里请你注意,Service本身没有做什么重要的事情,只是在连接器和容器外面多包了一层,把它们组装在一起。Tomcat内可能有多个Service,这样的设计也是出于灵活性的考虑。通过在Tomcat中配置多个Service,可以实现通过不同的端口号来访问同一台机器上部署的不同应用。

#### 连接器组件

![t14Htx.png](https://s1.ax1x.com/2020/05/31/t14Htx.png)

连接器中的各个组件的作用如下:

1. EndPoint

   Coyote 通信端点,即通信监听的接口,是具体Socket接收和发送处理器,是对传输层的抽象,因此EndPoint用来实现TCP/IP协议的。

   Tomcat 并没有EndPoint 接口,而是提供了一个抽象类AbstractEndpoint , 里面定义了两个内部类:Acceptor和SocketProcessor。Acceptor用于监听Socket连接请求。SocketProcessor用于处理接收到的Socket请求,它实现Runnable接口,在Run方法里调用协议处理组件Processor进行处理。为了提高处理能力,SocketProcessor被提交到线程池来执行。

2. Processor

   Coyote 协议处理接口 ,如果说EndPoint是用来实现TCP/IP协议的,那么Processor用来实现HTTP协议,Processor接收来自EndPoint的Socket,读取字节流解析成Tomcat Request和Response对象,并通过Adapter将其提交到容器处理,Processor是对应用层协议的抽象。

3. ProtocolHandler

   Coyote 协议接口, 通过Endpoint 和 Processor , 实现针对具体协议的处理能力。Tomcat 按照协议和I/O 提供了6个实现类 : AjpNioProtocol ,AjpAprProtocol, AjpNio2Protocol , Http11NioProtocol ,Http11Nio2Protocol ,Http11AprProtocol。我们在配置tomcat/conf/server.xml 时 , 至少要指定具体的ProtocolHandler , 当然也可以指定协议名称 , 如 : HTTP/1.1 ,如果安装了APR,那么将使用Http11AprProtocol , 否则使用 Http11NioProtocol 。

4. Adapter

   由于协议不同,客户端发过来的请求信息也不尽相同,Tomcat定义了自己的Request类来“存放”这些请求信息。ProtocolHandler接口负责解析请求并生成Tomcat Request类。但是这个Request对象不是标准的ServletRequest,也就意味着,不能用Tomcat Request作为参数来调用容器。Tomcat设计者的解决方案是引入CoyoteAdapter,这是适配器模式的经典运用,连接器调用CoyoteAdapter的Sevice方法,传入的是Tomcat
   Request对象,CoyoteAdapter负责将Tomcat Request转成ServletRequest,再调用容器的Service方法。

#### 容器 - Catalina

Tomcat是一个由一系列可配置的组件构成的Web容器,而Catalina是Tomcat的servlet容器。

Catalina 是Servlet 容器实现,包含了之前讲到的所有的容器组件,以及后续章节涉及到的安全、会话、集群、管理等Servlet 容器架构的各个方面。它通过松耦合的方式集成Coyote,以完成按照请求协议进行数据读写。同时,它还包括我们的启动入口、Shell程序等。

Tomcat 的模块分层结构图, 如下:

![t1IQZd.png](https://s1.ax1x.com/2020/05/31/t1IQZd.png)

Tomcat 本质上就是一款 Servlet 容器, 因此Catalina 才是 Tomcat 的核心 , 其他模块都是为Catalina 提供支撑的。 比如 : 通过Coyote 模块提供链接通信,Jasper 模块提供JSP引擎,Naming 提供JNDI 服务,Juli 提供日志服务。

Catalina 的主要组件结构如下:

![t1IYz8.png](https://s1.ax1x.com/2020/05/31/t1IYz8.png)

如上图所示, Catalina负责管理Server,而Server表示着整个服务器。Server下面有多个服务Service,每个服务都包含着多个连接器组件Connector(Coyote 实现)和一个容器组件Container。在Tomcat 启动的时候, 会初始化一个Catalina的实例。

Catalina 各个组件的职责:

![t1Ijwd.png](https://s1.ax1x.com/2020/05/31/t1Ijwd.png)

#### Container 结构

Tomcat设计了4种容器,分别是Engine、Host、Context和Wrapper。这4种容器不是平行关系,而是父子关系。, Tomcat通过一种分层的架构,使得Servlet容器具有很好的灵活性。

![t1oi6S.png](https://s1.ax1x.com/2020/05/31/t1oi6S.png)

各个组件的含义 :

![t1oEwj.png](https://s1.ax1x.com/2020/05/31/t1oEwj.png)

我们也可以再通过Tomcat的server.xml配置文件来加深对Tomcat容器的理解。Tomcat采用了组件化的设计,它的构成组件都是可配置的,其中最外层的是Server,其他组件按照一定的格式要求配置在这个顶层容器中。

![t1oKpV.png](https://s1.ax1x.com/2020/05/31/t1oKpV.png)

那么, Tomcat是怎么管理这些容器的呢?你会发现这些容器具有父子关系,形成一个树形结构,你可能马上就想到了设计模式中的组合模式。没错,Tomcat就是用组合模式来管理这些容器的。具体实现方法是,所有容器组件都实现了Container接口,因此组合模式可以使得用户对单容器对象和组合容器对象的使用具有一致性。这里单容器对象指的是最底层的Wrapper,组合容器对象指的是上面的Context、Host或者Engine。

![t1ott1.png](https://s1.ax1x.com/2020/05/31/t1ott1.png)

Container 接口中提供了方法:getParent、SetParent、addChild和removeChild

Container接口扩展了LifeCycle接口,LifeCycle接口用来统一管理各组件的生命周期

