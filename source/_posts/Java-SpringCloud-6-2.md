---
title: SpringCloud组件之Spring Cloud Sleuth (八)
date: 2019-11-3 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part7_sleuth>

### 微服务跟踪

微服务架构是一个分布式架构，它按业务划分服务单元，一个分布式系统往往有很多个服务单元。由于服务单元数量众多，业务的复杂性，如果出现了错误和异常，很难去定位。主要体现在，一个请求可能需要调用很多个服务，而内部服务的调用复杂性，决定了问题难以定位。所以微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题，很快定位。

<!--more-->

举个例子，在微服务系统中，一个来自用户的请求，请求先达到前端A（如前端界面），然后通过远程调用，达到系统的中间件B、C（如负载均衡、网关等），最后达到后端服务D、E，后端经过一系列的业务逻辑计算最后将数据返回给用户。对于这样一个请求，经历了这么多个服务，怎么样将它的请求过程的数据记录下来呢？这就需要用到服务链路追踪。

Google开源的 Dapper链路追踪组件，并在2010年发表了论文《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》，这篇文章是业内实现链路追踪的标杆和理论基础，具有非常大的参考价值。
目前，链路追踪组件有Google的Dapper，Twitter 的Zipkin，以及阿里的Eagleeye （鹰眼）等，它们都是非常优秀的链路追踪开源组件。

#### Dapper基本术语

Spring Cloud Sleuth采用的是Google的开源项目Dapper的专业术语。

- Span：基本工作单元，发送一个远程调度任务 就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID。
- Trace：一系列Span组成的一个树状结构。请求一个微服务系统的API接口，这个API接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Trace。
- Annotation：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束 。这些注解包括以下：
  - cs - Client Sent -客户端发送一个请求，这个注解描述了这个Span的开始
  - sr - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络传输的时间。
  - ss - Server Sent （服务端发送响应）–该注解表明请求处理的完成(当请求返回客户端)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。
  - cr - Client Received （客户端接收响应）-此时Span的结束，如果cr的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。

![1G7Qvq.png](https://s2.ax1x.com/2020/02/01/1G7Qvq.png)

#### Zipkin

[Zipkin](https://zipkin.io/) 是一个开放源代码分布式的跟踪系统，每个服务向zipkin报告计时数据，zipkin会根据调用关系通过Zipkin UI生成依赖关系图。

Zipkin是Twitter开源的分布式跟踪系统，基于Dapper论文设计而来，主要功能是收集系统的时序数据，从而追踪微服务架构的系统延时问题，此外还提供了一个非常友好的界面来帮助追踪分析数据。

Zipkin提供了可插拔数据存储方式：In-Memory、MySql、Cassandra以及Elasticsearch。为了方便在开发环境我直接采用了In-Memory方式进行存储，生产数据量大的情况则推荐使用Elasticsearch。

**部署Zipkin服务**

利用Docker来部署Zipkin服务再简单不过了：

```shell
docker run -d -p 9411:9411 \
--name zipkin \
openzipkin/zipkin
```

完成之后浏览器打开：`localhost:9411`可以看到Zipkin的可视化界面：

![1GxQxg.png](https://s2.ax1x.com/2020/02/01/1GxQxg.png)

**JAR包方式**

```shell
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```

**依赖**

```xml
<!-- zipkin服务端，如果使用代码方式运行 -->
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <version>2.12.9</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.12.9</version>
        </dependency>  
<!-- zipkin客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

**参数配置**

```yml
spring:
  zipkin:
    base-url: http://127.0.0.1:9411
  sleuth:
    sampler:
      percentage: 1.0
```

这里的`base-url`是zipkin服务端的地址，`percentage`是采样比例，设置为1.0时代表全部强求都需要采样。Sleuth默认采样算法的实现是Reservoir sampling，具体的实现类是`PercentageBasedSampler`，默认的采样比例为: 0.1(即10%)。

#### Spring Cloud Sleuth与Zipkin的配合使用

源码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part7_sleuth>

下图是一个接入Zipkin之后的服务调用简易流程图：

![1G4ZsU.png](https://s2.ax1x.com/2020/02/01/1G4ZsU.png)

运行顺序：首先运行eureka-service-sn（、zipkin-service-server，如果使用源码方式，这里使用docker方式运行），其次运行user-service与movie-service，然后运行zuul-service-7，访问http://localhost:5000/user/1得到数据结果，最后访问zipkin server首页<http://localhost:9411>，填入起始时间、结束时间等筛选条件后，点击Find a trace按钮，可以看到trace列表

### 参考

1. [微服务链路追踪原理](https://www.cnblogs.com/enochzzg/p/10987438.html)

2. [使用Spring Cloud Sleuth在应用中进行日志跟踪](https://blog.csdn.net/peterwanghao/article/details/79967634)

   