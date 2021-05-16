---
title: Spring Cloud  组件之Spring Cloud Consul-- 注册中心 (十一)
date: 2019-11-10 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part9_consul>

文档：<https://cloud.spring.io/spring-cloud-consul/reference/html/>

我们知道 Eureka 2.X 遇到困难停止开发了，但其实对国内的用户影响甚小，一方面国内大都使用的是 Eureka 1.X 系列，另一方面 Spring Cloud 支持很多服务发现的软件，Eureka 只是其中之一，下面是 Spring Cloud 支持的服务发现软件以及特性对比：

<!--more-->

| Feature              | euerka                       | Consul                 | zookeeper             | etcd              |
| :------------------- | :--------------------------- | :--------------------- | :-------------------- | :---------------- |
| 服务健康检查         | 可配支持                     | 服务状态，内存，硬盘等 | (弱)长连接，keepalive | 连接心跳          |
| 多数据中心           | —                            | 支持                   | —                     | —                 |
| kv 存储服务          | —                            | 支持                   | 支持                  | 支持              |
| 一致性               | —                            | raft                   | paxos                 | raft              |
| cap                  | ap                           | ca                     | cp                    | cp                |
| 使用接口(多语言能力) | http（sidecar）              | 支持 http 和 dns       | 客户端                | http/grpc         |
| watch 支持           | 支持 long polling/大部分增量 | 全量/支持long polling  | 支持                  | 支持 long polling |
| 自身监控             | metrics                      | metrics                | —                     | metrics           |
| 安全                 | —                            | acl /https             | acl                   | https 支持（弱）  |
| spring cloud 集成    | 已支持                       | 已支持                 | 已支持                | 已支持            |

在以上服务发现的软件中，Euerka 和 Consul 使用最为广泛。如果大家对注册中心的概念和 Euerka 不太了解的话， 可以参考我前期的文章：[Spring Cloud 组件之Eureka--注册中心 (二)](https://hanyunpeng0521.github.io/Java-SpringCloud-1-0/) ，本篇文章主要给大家介绍 Spring Cloud Consul 的使用。

#### Consul 介绍

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较 为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

**Consul 的优势：**

- 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft。
- 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。
- 支持健康检查。 etcd 不提供此功能。
- 支持 http 和 dns 协议接口。 zookeeper 的集成较为复杂, etcd 只支持 http 协议。
- 官方提供 web 管理界面, etcd 无此功能。
- 综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究。

**特性：**

- 服务发现
- 健康检查
- Key/Value 存储
- 多数据中心

**Consul 角色**

- client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群。
- server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其它数据中心通讯。 每个数据中心的 server 数量推荐为 3 个或是 5 个。

Consul 客户端、服务端还支持夸中心的使用，更加提高了它的高可用性。

![121x81.png](https://s2.ax1x.com/2020/02/07/121x81.png)

Consul 工作原理：

![121zgx.png](https://s2.ax1x.com/2020/02/07/121zgx.png)

- 1、当 Producer 启动的时候，会向 Consul 发送一个 post 请求，告诉 Consul 自己的 IP 和 Port
- 2、Consul 接收到 Producer 的注册后，每隔10s（默认）会向 Producer 发送一个健康检查的请求，检验Producer是否健康
- 3、当 Consumer 发送 GET 方式请求 /api/address 到 Producer 时，会先从 Consul 中拿到一个存储服务 IP 和 Port 的临时表，从表中拿到 Producer 的 IP 和 Port 后再发送 GET 方式请求 /api/address
- 4、该临时表每隔10s会更新，只包含有通过了健康检查的 Producer

Spring Cloud Consul 项目是针对 Consul 的服务治理实现。Consul 是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。

#### Consul VS Eureka

Eureka 是一个服务发现工具。该体系结构主要是客户端/服务器，每个数据中心有一组 Eureka 服务器，通常每个可用区域一个。通常 Eureka 的客户使用嵌入式 SDK 来注册和发现服务。对于非本地集成的客户，官方提供的 Eureka 一些 REST 操作 API，其它语言可以使用这些 API 来实现对 Eureka Server 的操作从而实现一个非 jvm 语言的 Eureka Client。

Eureka 提供了一个弱一致的服务视图，尽可能的提供服务可用性。当客户端向服务器注册时，该服务器将尝试复制到其它服务器，但不提供保证复制完成。服务注册的生存时间（TTL）较短，要求客户端对服务器心跳检测。不健康的服务或节点停止心跳，导致它们超时并从注册表中删除。服务发现可以路由到注册的任何服务，由于心跳检测机制有时间间隔，可能会导致部分服务不可用。这个简化的模型允许简单的群集管理和高可扩展性。

Consul 提供了一些列特性，包括更丰富的健康检查，键值对存储以及多数据中心。Consul 需要每个数据中心都有一套服务，以及每个客户端的 agent，类似于使用像 Ribbon 这样的服务。Consul agent 允许大多数应用程序成为 Consul 不知情者，通过配置文件执行服务注册并通过 DNS 或负载平衡器 sidecars 发现。

Consul 提供强大的一致性保证，因为服务器使用 Raft 协议复制状态 。Consul 支持丰富的健康检查，包括 TCP，HTTP，Nagios / Sensu 兼容脚本或基于 Eureka 的 TTL。客户端节点参与基于 Gossip 协议的健康检查，该检查分发健康检查工作，而不像集中式心跳检测那样成为可扩展性挑战。发现请求被路由到选举出来的 leader，这使他们默认情况下强一致性。允许客户端过时读取取使任何服务器处理他们的请求，从而实现像 Eureka 这样的线性可伸缩性。

Consul 强烈的一致性意味着它可以作为领导选举和集群协调的锁定服务。Eureka 不提供类似的保证，并且通常需要为需要执行协调或具有更强一致性需求的服务运行 ZooKeeper。

Consul 提供了支持面向服务的体系结构所需的一系列功能。这包括服务发现，还包括丰富的运行状况检查，锁定，密钥/值，多数据中心联合，事件系统和 ACL。Consul 和 consul-template 和 envconsul 等工具生态系统都试图尽量减少集成所需的应用程序更改，以避免需要通过 SDK 进行本地集成。Eureka 是一个更大的 Netflix OSS 套件的一部分，该套件预计应用程序相对均匀且紧密集成。因此 Eureka 只解决了一小部分问题，可以和 ZooKeeper 等其它工具可以一起使用。

Consul 强一致性(C)带来的是：

服务注册相比 Eureka 会稍慢一些。因为 Consul 的 raft 协议要求必须过半数的节点都写入成功才认为注册成功 Leader 挂掉时，重新选举期间整个 Consul 不可用。保证了强一致性但牺牲了可用性。

Eureka 保证高可用(A)和最终一致性：

服务注册相对要快，因为不需要等注册信息 replicate 到其它节点，也不保证注册信息是否 replicate 成功 当数据出现不一致时，虽然 A, B 上的注册信息不完全相同，但每个 Eureka 节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求 A 查不到，但请求 B 就能查到。如此保证了可用性但牺牲了一致性。

其它方面，eureka 就是个 servlet 程序，跑在 servlet 容器中; Consul 则是 go 编写而成。

#### Consul 安装

Consul 不同于 Eureka 需要单独安装，访问[Consul 官网](https://www.consul.io/downloads.html)下载 Consul 的最新版本

根据不同的系统类型选择不同的安装包，从下图也可以看出 Consul 支持所有主流系统。

![123Z8I.png](https://s2.ax1x.com/2020/02/07/123Z8I.png)

可以选择自己的系统进行安装。

我这里选择使用docker安装集群

```shell
docker pull consul
#启动第一个consul服务：consul1
docker run --name consul1 -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600 consul agent -server -bootstrap-expect 2 -ui -bind=0.0.0.0 -client=0.0.0.0
#获取 consul server1 的 ip 地址
JOIN_IP=`docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul1`
#启动第二个consul服务：consul2， 并加入consul1（使用join命令）
docker run --name consul2 -d -p 8501:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join $JOIN_IP
#启动第三个consul服务：consul3，并加入consul1
docker run --name consul3 -d -p 8502:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join $JOIN_IP
docker run --name consul4 -d -p 8503:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join $JOIN_IP
docker run --name consul5 -d -p 8504:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join $JOIN_IP


#使用完
docker rm -f consul1 consul2 consul3 consul4 consul5
```

8500 http 端口，用于 http 接口和 web ui
8300 server rpc 端口，同一数据中心 consul server 之间通过该端口通信
8301 serf lan 端口，同一数据中心 consul client 通过该端口通信
8302 serf wan 端口，不同数据中心 consul server 通过该端口通信
8600 dns 端口，用于服务发现
-bbostrap-expect 2: 集群至少两台服务器，才能选举集群leader
-ui：运行 web 控制台
-bind： 监听网口，0.0.0.0 表示所有网口，如果不指定默认未127.0.0.1，则无法和容器通信
-client ： 限制某些网口可以访问

宿主机浏览器访问：http://localhost:8500 或者 http://localhost:8501 或者 http://localhost:8502

任意stop掉其中一个consul，只要剩余consul数目大于等于两个，宿主机就能正常访问对应的链接；

创建test.json文件，以脚本形式注册服务到consul：

test.json文件内容如下：

```json
{
    "ID": "test-service1",
    "Name": "test-service1",
    "Tags": [
        "test",
        "v1"
    ],
    "Address": "127.0.0.1",
    "Port": 8000,
    "Meta": {
        "X-TAG": "testtag"
    },
    "EnableTagOverride": false,
    "Check": {
        "DeregisterCriticalServiceAfter": "90m",
        "HTTP": "http://zhihu.com",
        "Interval": "10s"
    }
}
```

通过 http 接口注册服务（端口可以是8500. 8501， 8502等能够正常访问consul的就行）：

```
curl -X PUT --data @test.json http://localhost:8500/v1/agent/service/register
```

宿主机浏览器访问以下链接可以看到所有通过健康检查的可用test-server1服务列表

（任意正常启动consul的端口皆可）：

```
http://localhost:8501/v1/health/service/test-server1?passing
```

其它应用程序可以通过这种方式轮询获取服务列表，这就是微服务能够动态知道其依赖微服务可用列表的原理。

解绑定：

```
curl -i -X PUT http://127.0.0.1:8501/v1/agent/service/deregister/test-server1
```

集群方式需要至少启动两个consul server，本机调试web应用时，为了方便可以用 -dev 参数方式仅启动一个consul server

```shell
docker run --name consul0 -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600 consul agent -dev -bind=0.0.0.0 -client=0.0.0.0
```

#### 快速开始

##### Consul服务发现

服务发现是基于微服务体系结构的关键原则之一。 尝试手动配置每个客户端或某种形式的约定可能非常困难，并且可能非常脆弱。 Consul通过HTTP API和DNS提供服务发现服务。 Spring Cloud Consul利用HTTP API进行服务注册和发现。 这并不妨碍非Spring Cloud应用程序利用DNS接口。 

1. 依赖

   ```xml
   <dependency>
           <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-consul-discovery</artifactId>
   </dependency>
   ```

2. 注册consul

   当客户向Consul注册时，它会提供有关自身的元数据，例如主机和端口，ID，名称和标签。 默认情况下会创建HTTP检查，Consul每10秒命中一次/ health端点。 如果运行状况检查失败，则将服务实例标记为严重。

   ```java
   @SpringBootApplication
   @RestController
   public class Application {
    
       @RequestMapping("/")
       public String home() {
           return "Hello world";
       }
    
       public static void main(String[] args) {
           new SpringApplicationBuilder(Application.class).web(true).run(args);
       }
    
   }
   ```

   （以上是一个普通的Spring Boot应用程序）。 如果Consul客户端位于localhost:8500以外的其他位置，则需要配置以查找客户端。如：

   ```yml
   spring:
     cloud:
       consul:
         host: localhost
         port: 8500
   ```

   注意：如果您使用Spring Cloud Consul Config，则需要将上述值放在bootstrap.yml而不是application.yml中。

   从Environment获取的默认服务名称，实例ID和端口分别是`${spring.application.name}`，Spring Context ID和`${server.port}`。

   要禁用Consul Discovery Client，可以将spring.cloud.consul.discovery.enabled设置为false【客户端可以设置注册到 Consul 中，也可以不注册到 Consul 注册中心中，根据我们的业务来选择，只需要在使用服务时通过 Consul 对外提供的接口获取服务信息即可】。

   要禁用服务注册，可以将spring.cloud.consul.discovery.register设置为false。

3. HTTP健康检测

   Consul实例的运行状况检查默认为“/ health”，这是Spring Boot Actuator应用程序中有用端点的默认位置。 如果使用非默认上下文路径或servlet路径（例如server.servletPath = /foo）或管理端点路径（例如management.context-path = /admin），则需要更改这些，即使对于Actuator应用程序也是如此。 还可以配置Consul用于检查健康端点的时间间隔。 “10s”和“1m”分别代表10秒和1分钟。

   综上健康检测是需要依赖Spring Boot Actuator。

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   ```

   application.yml

   ```yml
   spring:
     cloud:
       consul:
         discovery:
           healthCheckPath: ${management.context-path}/health
           healthCheckInterval: 15s
   ```

   - 元数据和Consul标签

     Consul尚不支持服务元数据。 Spring Cloud的ServiceInstance有一个Map <String，String>元数据字段。 Spring Consul使用Consul标签来近似元数据，直到Consul正式支持元数据。 形式为key = value的标签将被拆分并分别用作Map键和值。 没有等号=的标签将用作键和值。

     application.yml.

     ```yml
     spring:
       cloud:
         consul:
           discovery:
             tags: foo=bar, baz
     ```

     上面的配置将生成一个带有foo→bar和baz→baz的映射。

   - 使Consul实例ID唯一

     默认情况下，consul实例注册的ID等于其Spring Application Context ID。 默认情况下，Spring Application Context ID为$ {spring.application.name}：逗号，分隔，配置文件：${server.port}。 对于大多数情况，这将允许一台服务的多个实例在一台计算机上运行。 如果需要进一步的唯一性，使用Spring Cloud可以通过在spring.cloud.consul.discovery.instanceId中提供唯一标识符来覆盖它。 例如：

     application.yml. 

     ```yml
     spring:
       cloud:
         consul:
           discovery:
             instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
     ```

##### 查找服务

1. 用Ribbon

   Spring Cloud支持Feign（一个REST客户端构建器）和Spring RestTemplate，用于使用逻辑服务名称/ ID而不是物理URL查找服务。 Feign和发现感知RestTemplate都使用Ribbon进行客户端负载平衡。

   如果您想使用RestTemplate访问服务STORES，只需在springboot主类中声明如下：

   ```java
   @LoadBalanced
   @Bean
   public RestTemplate loadbalancedRestTemplate() {
        new RestTemplate();
   }
   ```

   并像如下使用它（注意我们使用Consul的STORES服务名称而不完全限定域名）：

   ```java
   @Autowired
   RestTemplate restTemplate;
   public String getFirstProduct() {
      return this.restTemplate.getForObject("https://STORES/products/1", String.class);
   }
   ```

2. 用DiscoveryClient

   您还可以使用org.springframework.cloud.client.discovery.DiscoveryClient，它为不特定于Netflix的发现客户端提供简单的API，例如：

   ```java
   @Autowired
   private DiscoveryClient discoveryClient;
    
   public String serviceUrl() {
       List<ServiceInstance> list = discoveryClient.getInstances("STORES");
       if (list != null && list.size() > 0 ) {
           return list.get(0).getUri();
       }
       return null;
   }
   ```

#### Consul失败重试

如果您希望应用程序启动时偶尔可能无法使用领事代理，您可以要求它在失败后继续尝试。 您需要在类路径中添加spring-retry和spring-boot-starter-aop。 默认行为是重试6次，初始退避间隔为1000毫秒，指数乘数为1.1，以便后续退避。 您可以使用spring.cloud.consul.retry.*配置属性配置这些属性（和其他属性）。 这适用于Spring Cloud Consul Config和Discovery注册。

```xml
<dependency>
   <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
   </dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

#### 整合Hystrix的断路器

应用程序可以使用Spring Cloud Netflix项目提供的Hystrix断路器，将此启动器包含在项目pom.xml中：spring-cloud-starter-hystrix。 Hystrix不依赖于Netflix Discovery Client。 @EnableHystrix注释应放在配置类（通常是springbooyt主类）上。 然后可以使用@HystrixCommand对方法进行注释，以便通过断路器进行保护。 

#### Hystrix检测整合Turbine和Consul

Turbine（由Spring Cloud Netflix项目提供）聚合多个Hystrix指标流实例，因此仪表板可以显示聚合视图。 Turbine使用DiscoveryClient接口查找相关实例。 要将Turbine与Spring Cloud Consul一起使用，请以类似于以下示例的方式配置Turbine应用程序：

pom.xml. 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-turbine</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

请注意，Turbine依赖性不是startter。 turbine包括对Netflix Eureka的支持。

application.yml. 

```yml
spring.application.name: turbine
applications: consulhystrixclient1,consulhystrixclient2
turbine:
  aggregator:
    clusterConfig: ${applications}
  appConfig: ${applications}
```

clusterConfig和appConfig部分必须匹配，因此将逗号分隔的服务ID列表放入单独的配置属性中非常有用。

```java
@EnableTurbine
@SpringBootApplication
public class Turbine {
    public static void main(String[] args) {
        SpringApplication.run(DemoturbinecommonsApplication.class, args);
    }
}
```

#### 自动配置原理

**org.springframework.cloud.spring-cloud-consul-core**

consul的核心配置

1. org.springframework.cloud.consul.ConsulAutoConfiguration

   ```java
   @Configuration(
       proxyBeanMethods = false
   )
   @EnableConfigurationProperties
   @ConditionalOnConsulEnabled
   public class ConsulAutoConfiguration {
       public ConsulAutoConfiguration() {
       }
   
       @Bean
       @ConditionalOnMissingBean
       public ConsulProperties consulProperties() {
           return new ConsulProperties();
       }
   
       @Bean
       @ConditionalOnMissingBean
       public ConsulClient consulClient(ConsulProperties consulProperties) {
           int agentPort = consulProperties.getPort();
           String agentHost = !StringUtils.isEmpty(consulProperties.getScheme()) ? consulProperties.getScheme() + "://" + consulProperties.getHost() : consulProperties.getHost();
           if (consulProperties.getTls() != null) {
               TLSConfig tls = consulProperties.getTls();
               com.ecwid.consul.transport.TLSConfig tlsConfig = new com.ecwid.consul.transport.TLSConfig(tls.getKeyStoreInstanceType(), tls.getCertificatePath(), tls.getCertificatePassword(), tls.getKeyStorePath(), tls.getKeyStorePassword());
               return new ConsulClient(agentHost, agentPort, tlsConfig);
           } else {
               return new ConsulClient(agentHost, agentPort);
           }
       }
   
       @ConditionalOnClass({Retryable.class, Aspect.class, AopAutoConfiguration.class})
       @Configuration(
           proxyBeanMethods = false
       )
       @EnableRetry(
           proxyTargetClass = true
       )
       @Import({AopAutoConfiguration.class})
       @EnableConfigurationProperties({RetryProperties.class})
       protected static class RetryConfiguration {
           protected RetryConfiguration() {
           }
   
           @Bean(
               name = {"consulRetryInterceptor"}
           )
           @ConditionalOnMissingBean(
               name = {"consulRetryInterceptor"}
           )
           public RetryOperationsInterceptor consulRetryInterceptor(RetryProperties properties) {
               return (RetryOperationsInterceptor)RetryInterceptorBuilder.stateless().backOffOptions(properties.getInitialInterval(), properties.getMultiplier(), properties.getMaxInterval()).maxAttempts(properties.getMaxAttempts()).build();
           }
       }
   
       @Configuration(
           proxyBeanMethods = false
       )
       @ConditionalOnClass({Endpoint.class})
       protected static class ConsulHealthConfig {
           protected ConsulHealthConfig() {
           }
   
           @Bean
           @ConditionalOnMissingBean
           @ConditionalOnEnabledEndpoint
           public ConsulEndpoint consulEndpoint(ConsulClient consulClient) {
               return new ConsulEndpoint(consulClient);
           }
   
           @Bean
           @ConditionalOnMissingBean
           @ConditionalOnEnabledHealthIndicator("consul")
           public ConsulHealthIndicator consulHealthIndicator(ConsulClient consulClient) {
               return new ConsulHealthIndicator(consulClient);
           }
       }
   }
   
   ```

2. ConsulProperties:通用配置类

   ```java
   @ConfigurationProperties("spring.cloud.consul")
   @Validated
   public class ConsulProperties {
       @NotNull
       private String host = "localhost";
       private String scheme;
       @NotNull
       private int port = 8500;
       private boolean enabled = true;
       private ConsulProperties.TLSConfig tls;
   }
   ```

3. RetryProperties：重试配置类

   ```java
   @ConfigurationProperties("spring.cloud.consul.retry")
   public class RetryProperties {
       private long initialInterval = 1000L;
       private double multiplier = 1.1D;
       private long maxInterval = 2000L;
       private int maxAttempts = 6;
       }
   ```

**org.springframework.cloud.spring-cloud-consul-discovery**

客户端注册配置包

1. META-INF/spring.factories:编写可能需要注册到sb里面的Bean类

   ```properties
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.springframework.cloud.consul.discovery.RibbonConsulAutoConfiguration,\
   org.springframework.cloud.consul.discovery.configclient.ConsulConfigServerAutoConfiguration,\
   org.springframework.cloud.consul.serviceregistry.ConsulAutoServiceRegistrationAutoConfiguration,\
   org.springframework.cloud.consul.serviceregistry.ConsulServiceRegistryAutoConfiguration,\
   org.springframework.cloud.consul.discovery.ConsulDiscoveryClientConfiguration,\
   org.springframework.cloud.consul.discovery.reactive.ConsulReactiveDiscoveryClientConfiguration,\
   org.springframework.cloud.consul.discovery.ConsulCatalogWatchAutoConfiguration, \
   org.springframework.cloud.consul.support.ConsulHeartbeatAutoConfiguration
   org.springframework.cloud.bootstrap.BootstrapConfiguration=\
   org.springframework.cloud.consul.discovery.configclient.ConsulDiscoveryClientConfigServiceBootstrapConfiguration
   
   ```

   每个自动配置都涉及到consul的架构和设计原理

### 参考

1. [docker上搭建consul集群全流程](https://www.cnblogs.com/lonelyxmas/p/10880717.html)