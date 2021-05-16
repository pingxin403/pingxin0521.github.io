---
title: Spring Cloud Alibaba 组件之Nacos--注册中心、配置中心（九）
date: 2019-11-4 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part8_alibaba>

#### 什么是Spring Cloud Alibaba

1.  阿里巴巴结合自身微服务实践,开源的微服务全家桶
2. 在Spring Cloud项目中孵化,很可能成为Spring Cloud第二代的标准实现
3.  在业界广泛使用，已有很多成功案例

<!--more-->

#### 应用场景

1. 大型复杂的系统,例如大型电商系统
2. 高并发系统,例如大型门户网站,商品秒杀系统
3. 需求不明确,且变更很快的系统,例如创业公司业务系统

#### Spring Cloud Alibaba和Spring Cloud 的区别和联系

SpringCloud Alibaba是SpringCloud的子项目，SpringCloud Alibaba符合SpringCloud标准
比较SpringCloud第一代与SpringCloud Alibaba的优势，如下如：

![1Yy4SS.png](https://s2.ax1x.com/2020/02/02/1Yy4SS.png)

![1Yy5Qg.png](https://s2.ax1x.com/2020/02/02/1Yy5Qg.png)

#### 主要功能

- **服务限流降级**：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道

#### 组件

- **[Sentinel](https://github.com/alibaba/Sentinel)**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **[Nacos](https://github.com/alibaba/Nacos)**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **[RocketMQ](https://rocketmq.apache.org/)**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **[Dubbo](https://github.com/apache/dubbo)**：Apache Dubbo是一款高性能 Java RPC 框架。
- **[Seata](https://github.com/seata/seata)**：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- **[Alibaba Cloud ACM](https://www.aliyun.com/product/acm)**：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **[Alibaba Cloud OSS](https://www.aliyun.com/product/oss)**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **[Alibaba Cloud SchedulerX](https://help.aliyun.com/document_detail/43136.html)**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- **[Alibaba Cloud SMS](https://www.aliyun.com/product/sms)**: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

### Nacos

[Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html) 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

 Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

服务（Service）是 Nacos 世界的一等公民。Nacos 支持几乎所有主流类型的“服务”的发现、配置和管理：

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)

[Spring Cloud RESTful Service](https://spring.io/understanding/REST)

Nacos 的关键特性包括:

-  服务发现和服务健康监测

  Nacos 支持基于 DNS 和基于 RPC 的服务发现。服务提供者使用 [原生SDK](https://nacos.io/zh-cn/docs/sdk.html)、[OpenAPI](https://nacos.io/zh-cn/docs/open-API.html)、或一个[独立的Agent TODO](https://nacos.io/zh-cn/docs/other-language.html)注册 Service 后，服务消费者可以使用[DNS TODO](https://nacos.io/zh-cn/docs/xx) 或[HTTP&API](https://nacos.io/zh-cn/docs/open-API.html)查找和发现服务。

  Nacos 提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求。Nacos 支持传输层 (PING 或 TCP)和应用层 (如 HTTP、MySQL、用户自定义）的健康检查。 对于复杂的云环境和网络拓扑环境中（如 VPC、边缘网络等）服务的健康检查，Nacos 提供了 agent 上报模式和服务端主动检测2种健康检查模式。Nacos 还提供了统一的健康检查仪表盘，帮助您根据健康状态管理服务的可用性及流量。

-  动态配置服务

  动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。

  动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。

  配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。

  Nacos 提供了一个简洁易用的UI ([控制台样例 Demo](http://console.nacos.io/nacos/index.html)) 帮助您管理所有的服务和应用的配置。Nacos 还提供包括配置版本跟踪、金丝雀发布、一键回滚配置以及客户端配置更新状态跟踪在内的一系列开箱即用的配置管理特性，帮助您更安全地在生产环境中管理配置变更和降低配置变更带来的风险。

- 动态 DNS 服务

  动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务。动态DNS服务还能让您更容易地实现以 DNS 协议为基础的服务发现，以帮助您消除耦合到厂商私有服务发现 API 上的风险。

  Nacos 提供了一些简单的 [DNS APIs TODO](https://nacos.io/zh-cn/docs/xx) 帮助您管理服务的关联域名和可用的 IP:PORT 列表.

-   服务及其元数据管理

  Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的 SLA 以及最首要的 metrics 统计数据。

#### Nacos 全景图

![1JZZ6I.png](https://s2.ax1x.com/2020/02/01/1JZZ6I.png)

阿里巴巴中间件团队出品的Nacos来作为新一代的服务管理中间件。

#### Nacos Server部署

##### 安装nacos server

下载地址：https://github.com/alibaba/nacos/releases

下载完成之后，解压。根据不同平台，执行不同命令，启动单机版Nacos服务：

- Linux/Unix/Mac：`sh startup.sh -m standalone`
- Windows：`cmd startup.cmd -m standalone`

`startup.sh`脚本位于Nacos解压后的bin目录下

启动之后 nacos默认端口为8848, 补充一点，nacos不仅可以做注册中心，还可以做分布式配置，相对SpringCloud 在git做配置较好点

##### docker 运行nacos server

- Clone 项目

  ```powershell
  git clone https://github.com/nacos-group/nacos-docker.git
  cd nacos-docker
  ```

- 单机模式 Derby

  ```powershell
  docker-compose -f example/standalone-derby.yaml up
  ```

- 单机模式 Mysql

  ```powershell
  docker-compose -f example/standalone-mysql.yaml up
  ```

- 集群模式

  ```powershell
  docker-compose -f example/cluster-hostname.yaml up 
  ```

- 关闭服务

  ```
  docker-compose -f example/standalone-derby.yaml stop
  ```

- 服务注册

  ```powershell
  curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
  ```

- 服务发现

  ```powershell
  curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instances?serviceName=nacos.naming.serviceName'
  ```

- 发布配置

  ```powershell
  curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=helloWorld"
  ```

- 获取配置

  ```powershell
    curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
  ```

- Nacos 控制台

  link：http://127.0.0.1:8848/nacos/

**通用属性**

| name                          | description                     | option                                 |
| ----------------------------- | ------------------------------- | -------------------------------------- |
| MODE                          | cluster模式/standalone模式      | cluster/standalone default **cluster** |
| NACOS_SERVERS                 | nacos cluster地址               | eg. ip1,ip2,ip3                        |
| PREFER_HOST_MODE              | 是否支持hostname                | hostname/ip default **ip**             |
| NACOS_SERVER_PORT             | nacos服务器端口                 | default **8848**                       |
| NACOS_SERVER_IP               | 多网卡下的自定义nacos服务器IP   |                                        |
| SPRING_DATASOURCE_PLATFORM    | standalone 支持 mysql           | mysql / empty default empty            |
| MYSQL_MASTER_SERVICE_HOST     | mysql 主节点host                |                                        |
| MYSQL_MASTER_SERVICE_PORT     | mysql 主节点端口                | default : **3306**                     |
| MYSQL_MASTER_SERVICE_DB_NAME  | mysql 主节点数据库              |                                        |
| MYSQL_MASTER_SERVICE_USER     | 数据库用户名                    |                                        |
| MYSQL_MASTER_SERVICE_PASSWORD | 数据库密码                      |                                        |
| MYSQL_SLAVE_SERVICE_HOST      | mysql从节点host                 |                                        |
| MYSQL_SLAVE_SERVICE_PORT      | mysql从节点端口                 | default :3306                          |
| MYSQL_DATABASE_NUM            | 数据库数量                      | default :2                             |
| JVM_XMS                       | -Xms                            | default :2g                            |
| JVM_XMX                       | -Xmx                            | default :2g                            |
| JVM_XMN                       | -Xmn                            | default :1g                            |
| JVM_MS                        | -XX:MetaspaceSize               | default :128m                          |
| JVM_MMS                       | -XX:MaxMetaspaceSize            | default :320m                          |
| NACOS_DEBUG                   | 开启远程调试                    | y/n default :n                         |
| TOMCAT_ACCESSLOG_ENABLED      | server.tomcat.accesslog.enabled | default :false                         |

##### Kubernetes Nacos

参考：https://nacos.io/zh-cn/docs/use-nacos-with-kubernetes.html

#### Nacos Spring Cloud 快速开始

- 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-config 实现配置的动态变更。
- 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现。

如果需要使用 Spring Cloud Greenwich 版本，请在 dependencyManagement 中添加如下内容

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

##### 启动配置管理

1. 添加依赖：

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

2. 在 `bootstrap.yml` 中配置 Nacos server 的地址和应用名

   ```yml
   spring:
     application:
       name: nacos-config-service
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
           file-extension: properties
   ```

   说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

   在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

   ```plain
   ${prefix}-${spring.profile.active}.${file-extension}
   ```

   - `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
   - `spring.profile.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profile.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
   - `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

3. 通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

   ```java
   @RestController
   @RequestMapping("/config")
   @RefreshScope
   public class ConfigController {
   
       @Value("${useLocalCache:false}")
       private boolean useLocalCache;
   
       @RequestMapping("/get")
       public boolean get() {
           return useLocalCache;
       }
   }
   ```

4. 首先通过调用 [Nacos Open API](https://nacos.io/zh-cn/docs/open-API.html) 向 Nacos Server 发布配置：dataId 为`nacos-config-service.properties`，内容为`useLocalCache=true`

   ```shell
   curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos-config-service.properties&group=DEFAULT_GROUP&content=useLocalCache=true"
   ```

5. 运行 `NacosConfigApplication`，调用 `curl http://localhost:8900/config/get`，返回内容是 `true`。

6. 再次调用 [Nacos Open API](https://nacos.io/zh-cn/docs/open-API.html) 向 Nacos server 发布配置：dataId 为`example.properties`，内容为`useLocalCache=false`

   ```shell
   curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos-config-service.properties&group=DEFAULT_GROUP&content=useLocalCache=false"
   ```

7. 再次访问 `http://localhost:8090/config/get`，此时返回内容为`false`，说明程序中的`useLocalCache`值已经被动态更新了。

Nacos 并不是通过推的方式将服务端最新的配置信息发送给客户端的，而是客户端维护了一个长轮询的任务，定时去拉取发生变更的配置信息，然后将最新的数据推送给 Listener 的持有者。

客户端是通过一个定时任务来检查自己监听的配置项的数据的，一旦服务端的数据发生变化时，客户端将会获取到最新的数据，并将最新的数据保存在一个 CacheData 对象中，然后会重新计算 CacheData 的 md5 属性的值，此时就会对该 CacheData 所绑定的 Listener 触发 receiveConfigInfo 回调。

考虑到服务端故障的问题，客户端将最新数据获取后会保存在本地的 snapshot 文件中，以后会优先从文件中获取配置信息的值。

##### 启动服务发现

加入依赖

```xml

        <!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-discovery -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
```

更改配置

```yml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

启动类上增加注解

```
@EnableDiscoveryClient
```

其他操作跟使用eureka做注册中心相同

#### 知识点

##### Nacos更多配置信息

```properties
spring.cloud.nacos.discovery.server-addr  #Nacos Server 启动监听的ip地址和端口
spring.cloud.nacos.discovery.service  #给当前的服务命名
spring.cloud.nacos.discovery.weight  #取值范围 1 到 100，数值越大，权重越大
spring.cloud.nacos.discovery.network-interface #当IP未配置时，注册的IP为此网卡所对应的IP地址，如果此项也未配置，则默认取第一块网卡的地址
spring.cloud.nacos.discovery.ip  #优先级最高
spring.cloud.nacos.discovery.port  #默认情况下不用配置，会自动探测
spring.cloud.nacos.discovery.namespace #常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

spring.cloud.nacos.discovery.access-key  #当要上阿里云时，阿里云上面的一个云账号名
spring.cloud.nacos.discovery.secret-key #当要上阿里云时，阿里云上面的一个云账号密码
spring.cloud.nacos.discovery.metadata    #使用Map格式配置，用户可以根据自己的需要自定义一些和服务相关的元数据信息
spring.cloud.nacos.discovery.log-name   日志文件名
spring.cloud.nacos.discovery.enpoint   #地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址
ribbon.nacos.enabled  #是否集成Ribbon 一般都设置成true即可
```

##### 多环境管理

在Nacos中，本身有多个不同管理级别的概念，包括：`Data ID`、`Group`、`Namespace`。只要利用好这些层级概念的关系，就可以根据自己的需要来实现多环境的管理。

1. 使用Data ID与profiles实现

   `Data ID`在Nacos中，我们可以理解为就是一个Spring Cloud应用的配置文件名。

   我们知道默认情况下`Data ID`的名称格式是这样的：`${spring.application.name}.properties`，即：以Spring Cloud应用命名的properties文件。

   实际上，`Data ID`的规则中，还包含了环境逻辑，这一点与Spring Cloud Config的设计类似。我们在应用启动时，可以通过`spring.profiles.active`来指定具体的环境名称，此时客户端就会把要获取配置的`Data ID`组织为：`${spring.application.name}-${spring.profiles.active}.properties`。

   > 实际上，更原始且最通用的匹配规则，是这样的：`${spring.cloud.nacos.config.prefix}`-`${spring.profile.active}`.`${spring.cloud.nacos.config.file-extension}`。而上面的结果是因为`${spring.cloud.nacos.config.prefix}`和`${spring.cloud.nacos.config.file-extension}`都使用了默认值。

2. 使用Group实现

   `Group`在Nacos中是用来对`Data ID`做集合管理的重要概念。所以，如果我们把一个环境的配置视为一个集合，那么也就可以实现不同环境的配置管理。对于`Group`的用法并没有固定的规定，所以我们在实际使用的时候，需要根据我们的具体需求，可以是架构运维上对多环境的管理，也可以是业务上对不同模块的参数管理。为了避免冲突，我们需要在架构设计之初，做好一定的规划。

3. 使用Namespace实现，建议使用

   `Namespace`在本系列教程中，应该还是第一次出现。先来看看官方的概念说明：用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的`Group`或`Data ID`的配置。`Namespace`的常用场景之一是不同环境的配置的区分隔离，例如：开发测试环境和生产环境的资源（如配置、服务）隔离等。

> 注意：不论用哪一种方式实现。对于指定环境的配置（`spring.profiles.active=DEV`、`spring.cloud.nacos.config.group=DEV_GROUP`、`spring.cloud.nacos.config.namespace=83eed625-d166-4619-b923-93df2088883a`），都不要配置在应用的`bootstrap.properties`中。而是在发布脚本的启动命令中，用`-Dspring.profiles.active=DEV`的方式来动态指定，会更加灵活！。

##### 加载多个配置

通过之前的学习，我们已经知道Spring应用对Nacos中配置内容的对应关系是通过下面三个参数控制的：

- spring.cloud.nacos.config.prefix
- spring.cloud.nacos.config.file-extension
- spring.cloud.nacos.config.group

默认情况下，会加载`Data ID=${spring.application.name}.properties`，`Group=DEFAULT_GROUP`的配置。

假设现在有这样的一个需求：我们想要对所有应用的Actuator模块以及日志输出做统一的配置管理。所以，我们希望可以将Actuator模块的配置放在独立的配置文件`actuator.properties`文件中，而对于日志输出的配置放在独立的配置文件`log.properties`文件中。通过拆分这两类配置内容，希望可以做到配置的共享加载与统一管理。

这时候，我们只需要做以下两步，就可以实现这个需求：

**第一步**：在Nacos中创建`Data ID=actuator.properties`，`Group=DEFAULT_GROUP`和`Data ID=log.properties`，`Group=DEFAULT_GROUP`的配置内容。

**第二步**：在Spring Cloud应用中通过使用`spring.cloud.nacos.config.ext-config`参数来配置要加载的这两个配置内容，比如：

```properties
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true
spring.cloud.nacos.config.ext-config[1].data-id=log.properties
spring.cloud.nacos.config.ext-config[1].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[1].refresh=true
```

可以看到，`spring.cloud.nacos.config.ext-config`配置是一个数组List类型。每个配置中包含三个参数：`data-id`、`group`，`refresh`；前两个不做赘述，与Nacos中创建的配置相互对应，`refresh`参数控制这个配置文件中的内容时候支持自动刷新，默认情况下，只有默认加载的配置才会自动刷新，对于这些扩展的配置加载内容需要配置该设置时候才会实现自动刷新。

##### 共享配置

通过上面加载多个配置的实现，实际上我们已经可以实现不同应用共享配置了。但是Nacos中还提供了另外一个便捷的配置方式，比如下面的设置与上面使用的配置内容是等价的：

```
spring.cloud.nacos.config.shared-dataids=actuator.properties,log.properties
spring.cloud.nacos.config.refreshable-dataids=actuator.properties,log.properties
```

- `spring.cloud.nacos.config.shared-dataids`参数用来配置多个共享配置的`Data Id`，多个的时候用用逗号分隔
- `spring.cloud.nacos.config.refreshable-dataids`参数用来定义哪些共享配置的`Data Id`在配置变化时，应用中可以动态刷新，多个`Data Id`之间用逗号隔开。如果没有明确配置，默认情况下所有共享配置都不支持动态刷新

##### 配置加载的优先级

当我们加载多个配置的时候，如果存在相同的key时，我们需要深入了解配置加载的优先级关系。

在使用Nacos配置的时候，主要有以下三类配置：

- A: 通过`spring.cloud.nacos.config.shared-dataids`定义的共享配置
- B: 通过`spring.cloud.nacos.config.ext-config[n]`定义的加载配置
- C: 通过内部规则（`spring.cloud.nacos.config.prefix`、`spring.cloud.nacos.config.file-extension`、`spring.cloud.nacos.config.group`这几个参数）拼接出来的配置

要弄清楚这几个配置加载的顺序，我们从日志中也可以很清晰的看到，我们可以做一个简单的实验：

```
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.shared-dataids=log.properties
spring.cloud.nacos.config.refreshable-dataids=log.properties
```

根据上面的配置，应用分别会去加载三类不同的配置文件

后面加载的配置会覆盖之前加载的配置，所以优先级关系是：`A < B < C`

##### 数据持久化

在之前的教程中，我们对于Nacos服务端自身并没有做过什么特殊的配置，一切均以默认的单机模式运行，完成了上述所有功能的学习。但是，Nacos的单机运行模式仅适用于学习与测试环境，对于有高可用要求的生产环境显然是不合适的。那么，我们是否可以直接启动多个单机模式的Nacos，然后客户端指定多个Nacos节点就可以实现高可用吗？答案是否定的。

在搭建Nacos集群之前，我们需要先修改Nacos的数据持久化配置为MySQL存储。默认情况下，Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只要支持MySQL的存储。

配置Nacos的MySQL存储只需要下面三步：

1. 安装数据库，版本要求：5.6.5+

2. 初始化MySQL数据库，数据库初始化文件：`nacos-mysql.sql`，该文件可以在Nacos程序包下的`conf`目录下获得。

3. 修改`conf/application.properties`文件，增加支持MySQL数据源配置，添加（目前只支持mysql）数据源的url、用户名和密码。配置样例如下：

   ```
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=
   ```

   到这里，Nacos数据存储到MySQL的配置就完成了，可以尝试继续用单机模式启动Nacos。然后再根据之前学习的Nacos配置中心的用法来做一些操作，配合MySQL工具就可以看到数据已经写入到数据库中了。

#### 自动配置

##### 分布式配置中心

1. META-INF/spring.factories

   ```properties
   
   org.springframework.cloud.bootstrap.BootstrapConfiguration=\
   com.alibaba.cloud.nacos.NacosConfigBootstrapConfiguration
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.alibaba.cloud.nacos.NacosConfigAutoConfiguration,\
   com.alibaba.cloud.nacos.endpoint.NacosConfigEndpointAutoConfiguration
   org.springframework.boot.diagnostics.FailureAnalyzer=\
   com.alibaba.cloud.nacos.diagnostics.analyzer.NacosConnectionFailureAnalyzer
   ```

2. com.alibaba.cloud.nacos.NacosConfigBootstrapConfiguration配置客户端启动时加载的bean

   ```java
   @Configuration
   @ConditionalOnProperty(
       name = {"spring.cloud.nacos.config.enabled"},
       matchIfMissing = true
   )
   public class NacosConfigBootstrapConfiguration {
       public NacosConfigBootstrapConfiguration() {
       }
   
       @Bean
       @ConditionalOnMissingBean
       public NacosConfigProperties nacosConfigProperties() {
           return new NacosConfigProperties();
       }
   
       @Bean
       public NacosPropertySourceLocator nacosPropertySourceLocator(NacosConfigProperties nacosConfigProperties) {
           return new NacosPropertySourceLocator(nacosConfigProperties);
       }
   }
   ```

3. NacosConfigAutoConfiguration

   ```java
   @Configuration
   @ConditionalOnProperty(name = "spring.cloud.nacos.config.enabled", matchIfMissing = true)
   public class NacosConfigAutoConfiguration {
   
   	@Bean
   	public NacosConfigProperties nacosConfigProperties(ApplicationContext context) {
   		if (context.getParent() != null
   				&& BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
   						context.getParent(), NacosConfigProperties.class).length > 0) {
   			return BeanFactoryUtils.beanOfTypeIncludingAncestors(context.getParent(),
   					NacosConfigProperties.class);
   		}
   		return new NacosConfigProperties();
   	}
   	
   	@Bean
   	public NacosRefreshProperties nacosRefreshProperties() {
   		return new NacosRefreshProperties();
   	}
   
   	@Bean
   	public NacosRefreshHistory nacosRefreshHistory() {
   		return new NacosRefreshHistory();
   	}
   
   	@Bean
   	public NacosContextRefresher nacosContextRefresher(
   			NacosConfigProperties configProperties,
   			NacosRefreshProperties nacosRefreshProperties,
   			NacosRefreshHistory refreshHistory) {
   		return new NacosContextRefresher(nacosRefreshProperties, refreshHistory,
   				configProperties.configServiceInstance());
   	}
   }
   ```

4. NacosConfigProperties常见的配置属性

   ```java
   @ConfigurationProperties(NacosConfigProperties.PREFIX)
   public class NacosConfigProperties {
   
   	/**
   	 * Prefix of {@link NacosConfigProperties}.
   	 */
   	public static final String PREFIX = "spring.cloud.nacos.config";
   
   	private static final Logger log = LoggerFactory
   			.getLogger(NacosConfigProperties.class);
   
   	@Autowired
   	private Environment environment;
   
   	@PostConstruct
   	public void init() {
   		this.overrideFromEnv();
   	}
   
   	private void overrideFromEnv() {
   		if (StringUtils.isEmpty(this.getServerAddr())) {
   			String serverAddr = environment
   					.resolvePlaceholders("${spring.cloud.nacos.config.server-addr:}");
   			if (StringUtils.isEmpty(serverAddr)) {
   				serverAddr = environment
   						.resolvePlaceholders("${spring.cloud.nacos.server-addr:}");
   			}
   			this.setServerAddr(serverAddr);
   		}
   	}
   
   	/**
   	 * nacos config server address.
   	 */
   	private String serverAddr;
   
   	/**
   	 * encode for nacos config content.
   	 */
   	private String encode;
   
   	/**
   	 * nacos config group, group is config data meta info.
   	 */
   	private String group = "DEFAULT_GROUP";
   
   	/**
   	 * nacos config dataId prefix.
   	 */
   	private String prefix;
   
   	/**
   	 * the suffix of nacos config dataId, also the file extension of config content.
   	 */
   	private String fileExtension = "properties";
   
   	/**
   	 * timeout for get config from nacos.
   	 */
   	private int timeout = 3000;
   
   	/**
   	 * nacos maximum number of tolerable server reconnection errors.
   	 */
   	private String maxRetry;
   
   	/**
   	 * nacos get config long poll timeout.
   	 */
   	private String configLongPollTimeout;
   
   	/**
   	 * nacos get config failure retry time.
   	 */
   	private String configRetryTime;
   
   	/**
   	 * If you want to pull it yourself when the program starts to get the configuration
   	 * for the first time, and the registered Listener is used for future configuration
   	 * updates, you can keep the original code unchanged, just add the system parameter:
   	 * enableRemoteSyncConfig = "true" ( But there is network overhead); therefore we
   	 * recommend that you use {@link ConfigService#getConfigAndSignListener} directly.
   	 */
   	private boolean enableRemoteSyncConfig = false;
   
   	/**
   	 * endpoint for Nacos, the domain name of a service, through which the server address
   	 * can be dynamically obtained.
   	 */
   	private String endpoint;
   
   	/**
   	 * namespace, separation configuration of different environments.
   	 */
   	private String namespace;
   
   	/**
   	 * access key for namespace.
   	 */
   	private String accessKey;
   
   	/**
   	 * secret key for namespace.
   	 */
   	private String secretKey;
   
   	/**
   	 * context path for nacos config server.
   	 */
   	private String contextPath;
   
   	/**
   	 * nacos config cluster name.
   	 */
   	private String clusterName;
   
   	/**
   	 * nacos config dataId name.
   	 */
   	private String name;
   
   	/**
   	 * the dataids for configurable multiple shared configurations , multiple separated by
   	 * commas .
   	 */
   	private String sharedDataids;
   
   	/**
   	 * refreshable dataids , multiple separated by commas .
   	 */
   	private String refreshableDataids;
   
   	/**
   	 * a set of extended configurations .
   	 */
   	private List<Config> extConfig;
   
   	private static ConfigService configService;
   	}
   ```

5. com.alibaba.cloud.nacos.endpoint.NacosConfigEndpointAutoConfiguration对应API端口的自动配置

   ```java
   @ConditionalOnWebApplication
   @ConditionalOnClass(value = Endpoint.class)
   @ConditionalOnProperty(name = "spring.cloud.nacos.config.enabled", matchIfMissing = true)
   public class NacosConfigEndpointAutoConfiguration {
   
   	@Autowired
   	private NacosConfigProperties nacosConfigProperties;
   
   	@Autowired
   	private NacosRefreshHistory nacosRefreshHistory;
   
   	@ConditionalOnMissingBean
   	@ConditionalOnEnabledEndpoint
   	@Bean
   	public NacosConfigEndpoint nacosConfigEndpoint() {
   		return new NacosConfigEndpoint(nacosConfigProperties, nacosRefreshHistory);
   	}
   
   	@Bean
   	public NacosConfigHealthIndicator nacosConfigHealthIndicator() {
   		return new NacosConfigHealthIndicator(
   				nacosConfigProperties.configServiceInstance());
   	}
   }
   
   ```

6. com.alibaba.cloud.nacos.diagnostics.analyzer.NacosConnectionFailureAnalyzer异常分析

   ```java
   public class NacosConnectionFailureAnalyzer
   		extends AbstractFailureAnalyzer<NacosConnectionFailureException> {
   
   	@Override
   	protected FailureAnalysis analyze(Throwable rootFailure,
   			NacosConnectionFailureException cause) {
   		return new FailureAnalysis(
   				"Application failed to connect to Nacos server: \""
   						+ cause.getServerAddr() + "\"",
   				"Please check your Nacos server config", cause);
   	}
   
   }
   ```

##### 注册中心客户端

1. /META-INF/spring.factories:可以看到

   ```properties
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     com.alibaba.cloud.nacos.NacosDiscoveryAutoConfiguration,\
     com.alibaba.cloud.nacos.ribbon.RibbonNacosAutoConfiguration,\
     com.alibaba.cloud.nacos.endpoint.NacosDiscoveryEndpointAutoConfiguration,\
     com.alibaba.cloud.nacos.discovery.NacosDiscoveryClientAutoConfiguration,\
     com.alibaba.cloud.nacos.discovery.configclient.NacosConfigServerAutoConfiguration
   org.springframework.cloud.bootstrap.BootstrapConfiguration=\
     com.alibaba.cloud.nacos.discovery.configclient.NacosDiscoveryClientConfigServiceBootstrapConfiguration
   
   ```

2. com.alibaba.cloud.nacos.NacosDiscoveryAutoConfiguration

   ```java
   @Configuration
   @EnableConfigurationProperties
   @ConditionalOnNacosDiscoveryEnabled
   @ConditionalOnProperty(
       value = {"spring.cloud.service-registry.auto-registration.enabled"},
       matchIfMissing = true
   )
   @AutoConfigureAfter({AutoServiceRegistrationConfiguration.class, AutoServiceRegistrationAutoConfiguration.class})
   public class NacosDiscoveryAutoConfiguration {
       public NacosDiscoveryAutoConfiguration() {
       }
   
       @Bean
       public NacosServiceRegistry nacosServiceRegistry(NacosDiscoveryProperties nacosDiscoveryProperties) {
           return new NacosServiceRegistry(nacosDiscoveryProperties);
       }
   
       @Bean
       @ConditionalOnBean({AutoServiceRegistrationProperties.class})
       public NacosRegistration nacosRegistration(NacosDiscoveryProperties nacosDiscoveryProperties, ApplicationContext context) {
           return new NacosRegistration(nacosDiscoveryProperties, context);
       }
   
       @Bean
       @ConditionalOnBean({AutoServiceRegistrationProperties.class})
       public NacosAutoServiceRegistration nacosAutoServiceRegistration(NacosServiceRegistry registry, AutoServiceRegistrationProperties autoServiceRegistrationProperties, NacosRegistration registration) {
           return new NacosAutoServiceRegistration(registry, autoServiceRegistrationProperties, registration);
       }
   }
   
   ```

   

