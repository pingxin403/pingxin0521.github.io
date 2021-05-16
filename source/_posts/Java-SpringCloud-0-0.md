---
title: Spring Cloud 入门 (一)
date: 2019-05-30 12:18:59
tags:
 - Java
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

[Spring Cloud](https://spring.io/projects/spring-cloud)为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线）。分布式系统的协调导致了样板模式, 使用Spring Cloud开发人员可以快速地支持实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好，包括开发人员自己的笔记本电脑，裸机数据中心，以及Cloud Foundry等托管平台。

版本：Hoxton.RELEASE

<!--more-->

Spring Cloud是一个基于Spring Boot实现的云原生应用开发工具，它为基于JVM的云原生应用开发中涉及的配置管理、服务发现、熔断器、智能路由、微代理、控制总线、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

Spring Cloud具有如下特点：

- 约定大于配置
- 适用于各种环境
- 隐藏了组件的复杂性，并提供声明式、无XML式的配置方式
- 开箱即用，快速启动
- 组件丰富，功能齐全
- ......

Spring Cloud作为第二代微服务的代表性框架，已经在国内众多大中小型的公司有实际应用案例。许多公司的业务线全部拥抱Spring Cloud，部分公司选择部分拥抱Spring Cloud。例如，拍拍贷资深架构师杨波老师就根据自己的实际经验以及对Spring Cloud的深入调研，并结合国内一线互联网大厂的开源项目应用实践结果，认为Spring Cloud技术栈中的有些组件离生产级开发尚有一定距离，最后提出了一个可供中小团队参考的微服务架构技术栈，又被称为“中国特色的微服务架构技术栈1.0”：

![1ukl0P.png](https://s2.ax1x.com/2020/01/27/1ukl0P.png)

#### 特性

Spring Cloud专注于提供良好的开箱即用经验的典型用例和可扩展性机制覆盖。

- 分布式/版本化配置
- 服务注册和发现
- 路由
- service - to - service调用
- 负载均衡
- 断路器
- 分布式消息传递

#### 云原生应用程序

[云原生](https://pivotal.io/platform-as-a-service/migrating-to-cloud-native-application-architectures-ebook)是一种应用开发风格，鼓励在持续交付和价值驱动开发领域轻松采用最佳实践。相关的学科是建立[12-factor Apps](http://12factor.net/)，其中开发实践与交付和运营目标相一致，例如通过使用声明式编程和管理和监控。Spring Cloud以多种具体方式促进这些开发风格，起点是一组功能，分布式系统中的所有组件都需要或需要时轻松访问。

许多这些功能都由[Spring Boot](http://projects.spring.io/spring-boot)覆盖，我们在Spring Cloud中建立。更多的由Spring Cloud提供为两个库：Spring Cloud Context和Spring Cloud Commons。Spring Cloud上下文为Spring Cloud应用程序（引导上下文，加密，刷新范围和环境端点）的`ApplicationContext`提供实用程序和特殊服务。Spring Cloud Commons是一组在不同的Spring Cloud实现中使用的抽象和常用类（例如Spring Cloud Netflix vs. Spring Cloud Consul）。

如果由于“非法密钥大小”而导致异常，并且您正在使用Sun的JDK，则需要安装Java加密扩展（JCE）无限强度管理策略文件。

有关详细信息，请参阅以下链接：

- [Java 6 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)
- [Java 7 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)
- [Java 8 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

将文件解压缩到`JDK / jre / lib / security`文件夹（无论您使用的是哪个版本的`JRE / JDK x64 / x86`）。

#### springcloud的版本说明

springcloud项目是由多个独立项目集合而成的，每个项目都是独立的，各自进行自己的迭代和版本发布。所以springcloud不方便使用版本号来管理，而是使用版本名。以避免和子项目版本号的冲突。

版本名的来源是伦敦的地铁站名，以字母排序。比如最早的Release版本为Angel，第二个Release版本为Brixton。。。

当一个版本的update积累的比较多或者解决了一个严重bug时，会发布一个ServiceRelease版本，简称SR，后面带的数字就是该大版本下的第一次发布。

![1uF7yn.png](https://s2.ax1x.com/2020/01/27/1uF7yn.png)

#### 5大常用组件

SpringCloud的组件相当繁杂，拥有诸多子项目。重点关注Netflix

![1uFBJe.png](https://s2.ax1x.com/2020/01/27/1uFBJe.png)

- Spring Cloud Netflix：核心组件，可以对多个Netflix OSS开源套件进行整合，包括以下几个组件：
  - Eureka：服务治理组件，包含服务注册与发现
  - Hystrix：容错管理组件，实现了熔断器
  - Ribbon：客户端负载均衡的服务调用组件
  - Feign：基于Ribbon和Hystrix的声明式服务调用组件
  - Zuul：网关组件，提供智能路由、访问过滤等功能
  - Archaius：外部化配置组件
- Spring Cloud Config：配置管理工具，实现应用配置的外部化存储，支持客户端配置信息刷新、加密/解密配置内容等。
- Spring Cloud Bus：事件、消息总线，用于传播集群中的状态变化或事件，以及触发后续的处理
- Spring Cloud Security：基于spring security的安全工具包，为我们的应用程序添加安全控制
- Spring Cloud Consul : 封装了Consul操作，Consul是一个服务发现与配置工具（与Eureka作用类似），与Docker容器可以无缝集成
- ......

**下面只简单介绍下经常用的5个**

- 服务发现——Netflix Eureka
- 客服端负载均衡——Netflix Ribbon
- 断路器——Netflix Hystrix
- 服务网关——Netflix Zuul
- 分布式配置——Spring Cloud Config

##### Eureka

![1uFxW4.png](https://s2.ax1x.com/2020/01/27/1uFxW4.png)

作用：实现服务治理（服务注册与发现）

简介：Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。

由两个组件组成：Eureka服务端和Eureka客户端。

- Eureka服务端用作服务注册中心。支持集群部署。

- Eureka客户端是一个java客户端，用来处理服务注册与发现。

在应用启动时，Eureka客户端向服务端注册自己的服务信息，同时将服务端的服务信息缓存到本地。客户端会和服务端周期性的进行心跳交互，以更新服务租约和服务信息。

##### Ribbon

![1ukpl9.png](https://s2.ax1x.com/2020/01/27/1ukpl9.png)

作用：Ribbon，主要提供客户侧的软件负载均衡算法。

简介：Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。

注意看上图，关键点就是将外界的rest调用，根据负载均衡策略转换为微服务调用。Ribbon有比较多的负载均衡策略，以后专门讲解。

##### Hystrix

![1ukkTK.png](https://s2.ax1x.com/2020/01/27/1ukkTK.png)

作用：断路器，保护系统，控制故障范围。

简介：为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

##### Zuul

![1ukZfe.png](https://s2.ax1x.com/2020/01/27/1ukZfe.png)

作用：api网关，路由，负载均衡等多种作用

简介：类似nginx，反向代理的功能，不过netflix自己增加了一些配合其他组件的特性。

在微服务架构中，后端服务往往不直接开放给调用端，而是通过一个API网关根据请求的url，路由到相应的服务。当添加API网关后，在第三方调用端和服务提供方之间就创建了一面墙，这面墙直接与调用方通信进行权限控制，后将请求均衡分发给后台服务端。

##### Config

![1uknld.png](https://s2.ax1x.com/2020/01/27/1uknld.png)

作用：配置管理

简介：SpringCloud Config提供服务器端和客户端。服务器存储后端的默认实现使用git，因此它轻松支持标签版本的配置环境，以及可以访问用于管理内容的各种工具。

这个还是静态的，得配合Spring Cloud Bus实现动态的配置更新。

#### Spring Cloud第一代

Spring Cloud自从推出之后，给大家的感觉就是Spring Cloud做`它最擅长的事，也就是高度抽象和封装，强强联手整合最优东西为我所用`，比如Netflix开源的Eureka，Hystrix，Ribbon等。而且提供`多种技术选型，态度中立而选最优`。8天前也就是2018年11月19号左右，Netflix的开源项目Hystrix宣布状态，不再开发新功能，处于维护状态。引发朋友圈的一些思考。

> 虽然Eureka，Hystrix等不再继续开发或维护，但是目前来说不影响使用，不管怎么说感谢开源，向Netflix公司的开源致敬。

随着Spring Cloud生态圈的发展与成长，Spring Cloud陆续推出了自己的一些组件，挑选主要组件说明如下表所示:

| 组件                               | 来源                   | 说明                                                         |
| :--------------------------------- | :--------------------- | :----------------------------------------------------------- |
| Spring-cloud-openfeign             | 基于Feign的升级        | 服务之间调用的必备组件                                       |
| spring-cloud-zuul                  | 来源于Netflix Zuul     | 目前还在继续维护，但是已经有自己的Spring Cloud Gateway,不久将来逐渐淘汰 |
| spring-cloud-eureka                | 集成于Netflix Eureka   | 目前还在跟随Spring Cloud版本升级维护，最终也会被替代         |
| spring-cloud-config                | 自研                   | 功能不足，国内使用其它配置中心替代，比如携程的Apollo         |
| 全链路监控(sleuth+zikpin或pinpont) | sleuth自研，其它第三方 | 国内目前使用最多的是skywaling等上生产                        |
| spring-cloud-ribbon                | 来源于Netflix集成      | ribbon目前还在跟随Spring Cloud版本维护中，目前孵化未来替代品spring-cloud-lb |
| Spring-cloud-hystrix               | 来源于Netflix集成      | 目前还在跟随Spring Cloud版本维护中目前已经孵化spring-cloud-r4j |

#### Spring Cloud 第二代

Spring Cloud第一代和第二代的组件组合汇总，如下表所示。

|                  | Spring Cloud第一代          | Spring Cloud第二代                           |
| :--------------- | :-------------------------- | :------------------------------------------- |
| 网关             | Spring Cloud Zuul           | Spring Cloud Gateway                         |
| 注册中心         | eureka(不再更新)，Consul,ZK | 阿里Nacos，拍拍贷radar等可选                 |
| 配置中心         | spring cloud config         | 阿里Nacos，携程Apollo，随行付Config Keeper   |
| 客户端软负载均衡 | Ribbon                      | spring-cloud-loadbalancer                    |
| 熔断器           | Hystrix                     | spring-cloud-r4j(Resilience4J)，阿里Sentinel |

由于Zuul性能一般，zuul 2.x(一直跳票，虽最终开源）但是Spring Cloud官方已经推出Spring Cloud gateway,Spring Cloud中国社区很久之前已经证实，Spring Cloud将不会集成zuul 2.x，也就是说在不就未来Zuul将从Spring Cloud生态圈中退出。

ribbon由于不支持webFlux的负载均衡，Spring Cloud官方很早就在孵化器项目中孵化spring-cloud-loadbalancer，目前已经将代码合并到spring-cloud-common中，预计在Spring Cloud G版可以使用，预计2018年12月底realese。

至于Hystrix，Netflix在2018年11月19号左右，Netflix的开源项目Hystrix宣布状态，不再开发新功能，处于维护状态，其实在之前Spring Cloud官方就在孵化spring-cloud-r4j.





### 参考

1. [Spring Cloud 微服务架构学习笔记与示例](https://www.cnblogs.com/edisonchou/p/java_spring_cloud_foundation_sample_list.html)
2. [Spring cloud应该怎么入门？](https://www.zhihu.com/question/283286745/answer/763040709)