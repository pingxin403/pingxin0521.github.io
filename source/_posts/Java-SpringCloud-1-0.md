---
title: Spring Cloud 组件之Eureka--注册中心 (二)
date: 2019-10-30 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part1_service-register-discovery>

注册中心：服务管理，核心是有个服务注册表，心跳机制动态维护。

为什么要用？

微服务应用和机器越来越多，调用方需要知道接口的网络地址，如果靠配置文件的方式去控制网络地址，对于动态新增机器，维护带来很大问题。

主流的注册中心：Zookeeper、Eureka、Consul、ETCD 等。

<!--more-->

![12M536.png](https://s2.ax1x.com/2020/02/07/12M536.png)

服务提供者 Provider：启动的时候向注册中心上报自己的网络信息。

服务消费者 Consumer：启动的时候向注册中心上报自己的网络信息，拉取 Provider 的相关网络信息。

**如何使用Spring Cloud搭建服务注册与发现模块**

这里我们会用到[Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/)，该项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

Eureka 只是其中之一，下面是 Spring Cloud 支持的服务发现软件以及特性对比：

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

### Eureka

所以，我们这里的核心内容就是服务发现模块：Eureka。下面我们动手来做一些尝试。

#### 理论

Eureka专门用于给其他服务注册的称为Eureka Server(服务注册中心)，其余注册到Eureka Server的服务称为Eureka Client。

![1ueqm9.png](https://s2.ax1x.com/2020/01/27/1ueqm9.png)

在Eureka Server一般我们会这样配置：

```text
register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
```

Eureka Client**分为服务提供者和服务消费者**。

- 但很可能，某服务**既是服务提供者又是服务消费者**。

如果在网上看到SpringCloud的**某个服务配置没有"注册"到Eureka-Server也不用过于惊讶**(但是它是可以获取Eureka服务清单的)

- 很可能只是作者把该服务认作为**单纯的服务消费者**，单纯的服务消费者无需对外提供服务，也就无须注册到Eureka中了

```yml
eureka:
  client:
    register-with-eureka: false  # 当前微服务不注册到eureka中(消费端)
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

下面是Eureka的治理机制：

- 服务提供者

- - **服务注册：启动的时候会通过发送REST请求的方式将自己注册到Eureka Server上**，同时带上了自身服务的一些元数据信息。
  - **服务续约：**在注册完服务之后，**服务提供者会维护一个心跳**用来持续告诉Eureka Server: "我还活着 ” 、
  - **服务下线：当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求**给Eureka Server, 告诉服务注册中心：“我要下线了 ”。

- 服务消费者

- - **获取服务：当我们启动服务消费者**的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单
  - **服务调用：服务消费者在获取服务清单后，通过服务名**可以获得具体提供服务的实例名和该实例的元数据信息。在进行服务调用的时候，**优先访问同处一个Zone中的服务提供方**。

- Eureka Server(服务注册中心)：

- - **失效剔除：**默认每隔一段时间（默认为60秒） 将当前清单中超时（默认为90秒）**没有续约的服务剔除出去**。
  - **自我保护：**。EurekaServer 在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%(通常由于网络不稳定导致)。 Eureka Server会将当前的**实例注册信息保护起来**， 让这些实例不会过期，尽可能**保护这些注册信息**。

最后，我们就有了这张图：

![1uejFx.png](https://s2.ax1x.com/2020/01/27/1uejFx.png)

#### 示例

##### 创建“服务注册中心”

创建一个基础的Spring Boot工程，并在`pom.xml`中引入需要的依赖内容：

```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
        <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

通过`@EnableEurekaServer`注解启动一个服务注册中心提供给其他应用进行对话。这一步非常的简单，只需要在一个普通的Spring Boot应用中添加这个注解就能开启此功能，比如下面的例子：

```java

@SpringBootApplication
@EnableEurekaServer
public class EurekaServiceSnApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceSnApplication.class, args);
    }

}
```

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在`application.yml`中问增加如下配置：

```yml
spring:
  application:
    name: eureka-server
server:
  port: 8761

eureka:
  server:
    enableSelfPreservation: false # 本地调试环境下关闭自我保护机制
  instance:
    hostname: localhost
    #prefer-ip-address: true
  client:
    register-with-eureka: false
    fetch-registry: false
  

```

为了与后续要进行注册的服务区分，这里将服务注册中心的端口通过`server.port`属性设置为`8761`。

启动工程后，访问：<http://localhost:8761/>

可以看到下面的页面，其中还没有发现任何服务

![1umKXQ.png](https://s2.ax1x.com/2020/01/27/1umKXQ.png)

##### 创建“服务提供方”

下面我们创建提供服务的客户端，并向服务注册中心注册自己。

假设我们有一个提供计算功能的微服务模块，我们实现一个RESTful API，提供根据用户id返回用户信息的接口

![1uKEkR.png](https://s2.ax1x.com/2020/01/27/1uKEkR.png)

首先，创建一个基本的Spring Boot应用，在`pom.xml`中，加入如下配置：

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>


        <!--eureke client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!-- actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- -->
        <!--&lt;!&ndash; 热启动，热部署依赖包，为了调试方便，加入此包 &ndash;&gt;-->
        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-devtools</artifactId>-->
            <!--<optional>true</optional>-->
        <!--</dependency>-->
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

配置文件

```yml
server:
  port: 8000
#eureka.client.serviceUrl.defaultZone属性对应服务注册中心的配置内容，指定服务注册中心的位置。
eureka:
  client:
    serviceUrl:
      #defaultZone: http://peer1:8761/eureka/,http://peer2:8762/eureka/
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true # 优先注册IP地址而不是hostname
  healthcheck:
    enabled: true # 启用健康检查,注意:需要引用spring boot actuator

#通过spring.application.name属性，我们可以指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。
spring:
  application:
    name: user-service
  jpa:
    generate-ddl: false
    show-sql: true
    hibernate:
      ddl-auto: none
  datasource:
    platform: h2
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
    username: sa
    url: jdbc:h2:mem:test
    driver-class-name: org.h2.Driver
  h2:
    console:
      path: /h2-console
      enabled: true
      settings:
        web-allow-others: true
  logging:
    level:
      root: INFO
      org.hibernate: INFO
      org.hibernate.type.descriptor.sql.BasicBinder: TRACE
      org.hibernate.type.descriptor.sql.BasicExtractor: TRACE

info:
  app:
    name: @project.artifactId@
    encoding: @project.build.sourceEncoding@
    java:
      source: @java.version@
      target: @java.version@
```

数据库文件

```sql
-- schema.sql
drop table if exists user;
create table user
(
  id       bigint generated by default as identity,
  username varchar(40),
  realname varchar(20),
  age      int(3),
  balance  decimal(10, 2),
  primary key (id)
);

-- data.sql
insert into user(id, username, realname, age, balance) values (1, 'account1', '张三', 20, 100.00);
insert into user(id, username, realname, age, balance) values (2, 'account2', '李四', 29, 180.00);
insert into user(id, username, realname, age, balance) values (3, 'account3', '王五', 32, 280.00);
```

User类

```java
@Entity
public class User implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    private String username;

    @Column
    private String realname;

    @Column
    private Integer age;

    @Column
    private BigDecimal balance;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getRealname() {
        return realname;
    }

    public void setRealname(String realname) {
        this.realname = realname;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public void setBalance(BigDecimal balance) {
        this.balance = balance;
    }
}

```

UserRepository接口

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

UserController

```java
@RestController
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public User findById(@PathVariable Long id) {
        Optional<User> userOptional =
                userRepository.findById(id);
        return userOptional.orElse(null);
    }
}

```

最后在主类中通过加上`@EnableDiscoveryClient`注解，该注解能激活Eureka中的`DiscoveryClient`实现，才能实现Controller中对服务信息的输出。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

为了在本机上测试区分服务提供方和服务注册中心，使用`server.port`属性设置不同的端口。

启动该工程后，再次访问：<http://localhost:8761/>

可以看到我们注册的user-service服务

##### 服务消费者

这里设置一个简单的服务消费者，消费者的本质就是API调用者

通过Eureka服务治理框架，我们可以通过服务名来获取具体的服务实例的位置了(IP)。一般在使用SpringCloud的时候**不需要自己手动创建**HttpClient来进行远程调用。

可以使用Spring封装好的**RestTemplate**工具类，使用起来很简单：

```java
// 传统的方式，直接显示写死IP是不好的！
    //private static final String REST_URL_PREFIX = "http://localhost:8001";
	

    /**
     * 使用 使用restTemplate访问restful接口非常的简单粗暴无脑。 (url, requestMap,
     * ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
     */
    @Autowired
    private RestTemplate restTemplate;

	// 服务实例名，配置
    @Value("${userservice.url}")
    private String userServiceUrl;

    @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
    public User findById(@PathVariable Long id) {
        return restTemplate.getForObject(userServiceUrl+"/" + id, User.class);
    }
```

为了实现服务的**高可用**，我们可以将**服务提供者集群**，集群**合理摊分**用户的请求(专业来说就是负载均衡)，可能你会想到nginx。

其实SpringCloud也支持的负载均衡功能，只不过它是**客户端的负载均衡**，这个功能实现就是Ribbon

负载均衡又区分了两种类型：

- 客户端负载均衡(Ribbon)

- - 服务实例的**清单在客户端**，客户端进行负载均衡算法分配。
  - (从上面的知识我们已经知道了：客户端可以从Eureka Server中得到一份服务清单，在发送请求时通过负载均衡算法，**在多个服务器之间选择一个进行访问**)

- 服务端负载均衡(Nginx)

- - 服务实例的**清单在服务端**，服务器进行负载均衡算法分配

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part1_service-register-discovery>

##### 高可用服务注册中心

Eureka Server除了单点运行之外，还可以通过运行多个实例，并进行互相注册的方式来实现高可用的部署，所以我们只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。

已上面的eureka-server为基础，对其改造，构建双节点的服务注册中心。

更改配置文件为

```yml
spring:
  application:
    name: eureka-service-ha
---
spring:
  # profile=peer1
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    # when profile=peer1, hostname=peer1
    hostname: peer1
  client:
    service-url:
      # register self to peer2
      defaultZone: http://peer2:8762/eureka
---
spring:
  # profile=peer2
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    # when profile=peer2, hostname=peer2
    hostname: peer2
  client:
    service-url:
      # register self to peer1
      defaultZone: http://peer1:8761/eureka
```

在`/etc/hosts`文件中添加对peer1和peer2的转换

```
127.0.0.1 peer1
127.0.0.1 peer2
```

通过`spring.profiles.active`属性来分别启动peer1和peer2

```shell
java -jar eureka-server-ha-1.0.0.jar --spring.profiles.active=peer1
java -jar eureka-server-ha-1.0.0.jar --spring.profiles.active=peer2
```

此时访问peer1的注册中心：`http://localhost:8762/`，如下图所示，我们可以看到`registered-replicas`中已经有peer2节点的eureka-server了。同样地，访问peer2的注册中心：`http://localhost:8761/`，能看到`registered-replicas`中已经有peer1节点，并且这些节点在可用分片（available-replicase）之中。我们也可以尝试关闭peer1，刷新`http://localhost:8761/`，可以看到peer1的节点变为了不可用分片（unavailable-replicas）。

**服务注册与发现**

在设置了多节点的服务注册中心之后，我们主需要简单需求服务配置，就能将服务注册到Eureka Server集群中。

修改user-service的配置文件：

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/,http://peer2:8762/eureka/
```

对`eureka.client.serviceUrl.defaultZone`属性做了改动，将注册中心指向了之前我们搭建的peer1与peer2。

虽然上面我们以双节点作为例子，但是实际上因负载等原因，我们往往可能需要在生产环境构建多于两个的Eureka Server节点。那么对于如何配置serviceUrl来让集群中的服务进行同步，需要我们更深入的理解节点间的同步机制来做出决策。

Eureka Server的同步遵循着一个非常简单的原则：只要有一条边将节点连接，就可以进行信息传播与同步。什么意思呢？不妨我们通过下面的实验来看看会发生什么。

- 场景一：假设我们有3个注册中心，我们将peer1、peer2、peer3各自都将serviceUrl指向另外两个节点。换言之，peer1、peer2、peer3是两两互相注册的。启动三个服务注册中心，并将user-service的serviceUrl指向peer1并启动，可以获得如下图所示的集群效果。

  ![1uTMY8.png](https://s2.ax1x.com/2020/01/27/1uTMY8.png)

访问http://localhost:8762/，可以看到3个注册中心组成了集群，compute-service服务通过peer1同步给了与之互相注册的peer2和peer3。

通过上面的实验，我们可以得出下面的结论来指导我们搭建服务注册中心的高可用集群：

- 两两注册的方式可以实现集群中节点完全对等的效果，实现最高可用性集群，任何一台注册中心故障都不会影响服务的注册与发现

##### 用户认证

增加pom依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在application.yml增加一些内容

```yml
security:
	basic:
		enabled: true
    user:
        name: user
        password: password123
```

如果不指定密码，用户默认为user，密码伪随机值，会在启动时打印

然后微服务注册时需要指定用户名和密码：

```
eureka:
	client:
		serviceUrl:
			defaultZone: http://user:password123@localhost:8761/eureka/
```

##### Rest端点

![3qURx0.png](https://s2.ax1x.com/2020/03/06/3qURx0.png)

##### 多网卡环境下的IP选择

1.忽略指定名称的网卡

```
spring:
  cloud:
    inetutils:
      ignored-interfaces: 
        - docker0
        - veth.*
```


2.使用正则表达式，指定使用的网络地址

```
spring:
  cloud:
    inetutils:
      preferred-networks: 
        - 192.168
        - 10.0
```


3.只使用站点本地地址

```
spring:
  cloud:
    inetutils:
      use-only-site-local-interfaces: true
```

4.手动指定IP地址

```
eureka:
  instance:
    prefer-ip-address: true
    ip-address: 127.0.0.1
```

##### 自我保护机制

Eureka自我保护机制，他不会剔除已经挂掉的服务，他会认为这个服务是在尝试重新连接的。

服务端配置文件中我们添加如下配置

```yml
#关闭保护机制，以确保注册中心将不可用的实例正确剔除
eureka.server.enable-self-preservation=false
#（代表是5秒，单位是毫秒,清理失效服务的间隔 ）
eureka.server.eviction-interval-timer-in-ms=5000
```

##### 健康检查

默认情况下，Eureka使用客户端心跳来确定客户端是否已启动。在成功注册后，Eureka始终宣布应用程序处于“UP”状态。

通过启用Eureka运行状况检查可以更改此行为，从而将应用程序状态传播到Eureka。这样，每个其他应用程序都不会向“UP”以外的状态下的应用程序发送流量。

Eureka的健康检查依赖于Springboot-actuator的/health

以下示例显示如何为客户端启用运行状况检查：

```yml
eureka:
  client:
    healthcheck:
      enabled: true
```

#### 原理分析

我们知道Eureka分为两部分，Eureka Server和Eureka Client。Eureka Server充当注册中心的角色，Eureka Client相对于Eureka Server来说是客户端，需要将自身信息注册到注册中心。本文主要介绍的就是在Eureka Client注册到Eureka Server时`RetryableClientQuarantineRefreshPercentage`参数的使用技巧。

##### Eureka Client注册过程分析

Eureka Client注册到Eureka Server时，首先遇到第一个问题就是Eureka Client端要知道Server的地址，这个参数对应的是`eureka.client.service-url.defaultZone`举个例子，在Eureka Client的properties文件中配置如下：

```
eureka.client.service-url.defaultZone=
http://localhost:8761/eureka,http://localhost:8762/eureka,http://localhost:8763/eureka,http://localhost:8764/eureka
```

如上图所示，Eureka Client配置对应的Eureka Server地址分别是8761、8762、8763、8764。这里存在两个问题：

- Eureka Client会将自身信息分别注册到这四个地址吗？
- Eureka Clinent注册机制是怎样的？

源码面前一目了然，带着这两个问题我们通过源码来解答这两个问题。Eureka Client在启动的时候注册源码如下：
`RetryableEurekaHttpClient`中的`execut`方法

```java
    protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
        List<EurekaEndpoint> candidateHosts = null;
        int endpointIdx = 0;

        for(int retry = 0; retry < this.numberOfRetries; ++retry) {
            EurekaHttpClient currentHttpClient = (EurekaHttpClient)this.delegate.get();
            EurekaEndpoint currentEndpoint = null;
            if (currentHttpClient == null) {
                if (candidateHosts == null) {
                    candidateHosts = this.getHostCandidates();
                    if (candidateHosts.isEmpty()) {
                        throw new TransportException("There is no known eureka server; cluster server list is empty");
                    }
                }

                if (endpointIdx >= candidateHosts.size()) {
                    throw new TransportException("Cannot execute request on any known server");
                }

                currentEndpoint = (EurekaEndpoint)candidateHosts.get(endpointIdx++);
                currentHttpClient = this.clientFactory.newClient(currentEndpoint);
            }

            try {
                EurekaHttpResponse<R> response = requestExecutor.execute(currentHttpClient);
                if (this.serverStatusEvaluator.accept(response.getStatusCode(), requestExecutor.getRequestType())) {
                    this.delegate.set(currentHttpClient);
                    if (retry > 0) {
                        logger.info("Request execution succeeded on retry #{}", retry);
                    }

                    return response;
                }

                logger.warn("Request execution failure with status code {}; retrying on another server if available", response.getStatusCode());
            } catch (Exception var8) {
                logger.warn("Request execution failed with message: {}", var8.getMessage());
            }

            this.delegate.compareAndSet(currentHttpClient, (Object)null);
            if (currentEndpoint != null) {
                this.quarantineSet.add(currentEndpoint);
            }
        }

        throw new TransportException("Retry limit reached; giving up on completing the request");
    }
```

按照我的理解，代码精简后内容如下：

```java
int endpointIdx = 0;
//用来保存所有Eureka Server信息(8761、8762、8763、8764)
List<EurekaEndpoint> candidateHosts = null;
//numberOfRetries的值代码写死默认为3次
for (int retry = 0; retry < numberOfRetries; retry++) {
	/**
	 *首次进入循环时，获取全量的Eureka Server信息(8761、8762、8763、8764)
	 */
	if (candidateHosts == null) {
        candidateHosts = getHostCandidates();
    }
	/**
	 *通过endpointIdx自增，依次获取Eureka Server信息，然后发送
	 *注册的Post请求.
	 */
    currentEndpoint = candidateHosts.get(endpointIdx++);
    currentHttpClient = clientFactory.newClient(currentEndpoint);
    try {
       /**
	 	*发送注册的Post请求动作，注意如果成功，则跳出循环，如果失败则
	 	*根据endpointIdx依次获取下一个Eureka Server.
	 	*/
        response = requestExecutor.execute(currentHttpClient);
        return respones;
    } catch (Exception e) {
        //向注册中心(Eureka Server)发起注册的post出现异常时，打印日志...
    }
    //如果此次注册动作失败，将当前的信息保存到quarantineSet中(一个Set集合)
    if (currentEndpoint != null) {
        quarantineSet.add(currentEndpoint);
    }
}
//如果都失败,则以异常形式抛出...
throw new TransportException("Retry limit reached; giving up on completing the request");
```

上面代码中还有一个方法很重要就是`List candidateHosts = getHostCandidates();`接下来看下`getHostCandidates()`方法源码

```java
private List<EurekaEndpoint> getHostCandidates() {
    List<EurekaEndpoint> candidateHosts = clusterResolver.getClusterEndpoints();
    quarantineSet.retainAll(candidateHosts);

    // If enough hosts are bad, we have no choice but start over again
    int threshold = (int) (candidateHosts.size() * transportConfig.getRetryableClientQuarantineRefreshPercentage());
    if (quarantineSet.isEmpty()) {
        // no-op
    } else if (quarantineSet.size() >= threshold) {
        logger.debug("Clearing quarantined list of size {}", quarantineSet.size());
        quarantineSet.clear();
    } else {
        List<EurekaEndpoint> remainingHosts = new ArrayList<>(candidateHosts.size());
        for (EurekaEndpoint endpoint : candidateHosts) {
            if (!quarantineSet.contains(endpoint)) {
                remainingHosts.add(endpoint);
            }
        }
        candidateHosts = remainingHosts;
    }
    return candidateHosts;
}
```

按照我的理解，将代码精简下，只包括关键逻辑，内容如下：

```java
private List<EurekaEndpoint> getHostCandidates() {
    /**
     * 获取所有defaultZone配置的注册中心信息(Eureka Server)，
     * 在本文例子中代表4个(8761、8762、8763、8764)Eureka Server
     */
    List candidateHosts = clusterResolver.getClusterEndpoints();
    /**
     * quarantineSet这个Set集合中保存的是不可用的Eureka Server
     * 此处是拿不可用的Eureka Server与全量的Eureka Server取交集
     */
    quarantineSet.retainAll(candidateHosts);
    /**
     * 根据RetryableClientQuarantineRefreshPercentage参数计算阈值
     * 该阈值后续会和quarantineSet中保存的不可用的Eureka Server个数
     * 作比较，从而判断是否返回全量的Eureka Server还是过滤掉不可用的
     * Eureka Server。
     */
    int threshold = 
       (int) (
        candidateHosts.size()
              *
        transportConfig.getRetryableClientQuarantineRefreshPercentage()
        );
    if (quarantineSet.isEmpty()) {
        /**
         * 首次进入的时候，此时quarantineSet为空，直接返回全量的
         * Eureka Server列表
         */
    } else if (quarantineSet.size() >= threshold) {
        /**
         * 将不可用的Eureka Server与threshold值相比较，如果不可
         * 用的Eureka Server个数大于阈值，则将之间保存的Eureka
         * Server内容直接清空，并返回全量的Eureka Server列表。
         */
        quarantineSet.clear();
    } else {
        /**
         * 通过quarantineSet集合保存不可用的Eureka Server来过滤
         * 全量的EurekaServer，从而获取此次Eureka Client要注册要
         * 注册的Eureka Server实例地址。
         */
        List<EurekaEndpoint> remainingHosts = new ArrayList<>(candidateHosts.size());
        for (EurekaEndpoint endpoint : candidateHosts) {
            if (!quarantineSet.contains(endpoint)) {
                remainingHosts.add(endpoint);
            }
        }
        candidateHosts = remainingHosts;
    }
    return candidateHosts;
}

```

通过源码分析，我们现在初步知道，当Eureka Client向Eureka Server发起注册请求的时候(根据defaultZone寻找Eureka Server列表)，如果有一次请求注册成功，那么后续就不会在向其他Eureka Server发起注册请求。以本文为例，注册中心有四个(8761、8762、8763、8764)。如果8761对应的Eureka Server服务的状态是UP，那么Eureka Client向该注册中心注册成功后，不会再向(8762、8763、8764)对应的Eureka Server发起注册请求(对应程序是在for循环中直接return respones)。

说到这里又引出来另外一个问题，如果8761这个Eureka Server是down掉的呢？

根据源码我们可知Eureka Client首次会向8761这个Server发起注册请求，如果该Server的状态是down，那么它会将该Server保存到quarantineSet这个Set集合中，然后再次访问8762这个Eureka Server，如果8762这个Server的状态依旧是down，它也会把这个Server保存到quarantineSet这个Set集合中，然后继续访问8763这个Server，如果8763这个Server的状态依旧是down，此时除了会将其保存到quarantineSet这个Set集合中之外，还会跳出本次循环。从而结束此次注册过程。

说道这里有人要问接下来会不会向8764这个Server发起注册，答案是否定的，因为循环的次数默认是3次。所以即使8764这个Server的状态是UP，它也不会接收到来自Eureka Client发起的注册信息。

Eureka Client向Eureka Server发起注册信息的过程除了在Eureka Client启动的时候触发，还有另外一种方式，就是后台定时任务。

假设我们上面描述的场景是在Eureka Client启动的时候，因为在启动的时候注册这个过程全部失败了，当后台定时任务执行时，还会进入该注册流程。注意此时quarantineSet的值为3(8761、8762、8763之前注册失败的Eureka Server)。

所以当程序再次进入`getHostCandidates()`方法时，`if (quarantineSet.isEmpty())`这个方法是不满足的，接下来会走`else if (quarantineSet.size() >= threshold)`这个判断，如果这个判断成立，那么会将quarantineSet集合清空，同时返回全量的Eureka Server列表，如果这个判断不成立，会拿quarantineSet集合中保存的内容去过滤Eureka Server的全量列表。以本文为例：

- `quarantineSet`中保存的是(8761、8762、8763)三个Eureka Server
- Eureka Server全量列表的内容是(8761、8762、8763、8764)四个Eureka Server，过滤后返回的结果为8764这个Eureka Server。

在本文的例子中8761、8762、8763这三个Eureka Server的状态是down而8764这个Eureka Server的状态是UP，我们其实是想走到最后的else分支，从而完成过滤操作，并最终得到8764这个Server，遗憾的是它并不会走到这个分支，而是被上面的`else if (quarantineSet.size() >= threshold)`这个分支所拦截，返回的依旧是全量的Eureka Server列表。这样造成的后果就是Eureka Client依旧会依次向(8761、8762、8763)这三个down的Eureka Server发起注册请求。

那么问题的关键在哪里呢？问题的关键就是threshold这个值的由来，因为此时quarantineSet.size()的值为3，而3这个值大于threshold，从而导致，会将quarantineSet集合清空，返回全量的Server列表。
我们知道threshold这个值是根据全量的Eureka Server列表乘以一个可配置的参数计算出来的，在本文的例子当中，我的properties文件中除了defaultZone之外并没有配置这个参数，那么也就是说这个参数是有默认值的，通过源码我们了解到，这个默认值是0.66。具体源码如下：

```java
final class PropertyBasedTransportConfigConstants {
	/**
	 *省略部分源码
	 */
    static class Values {
        static final int SESSION_RECONNECT_INTERVAL = 20*60;
        //默认值为0.66
        static final double QUARANTINE_REFRESH_PERCENTAGE = 0.66;
        static final int DATA_STALENESS_TRHESHOLD = 5*60;
        static final int ASYNC_RESOLVER_REFRESH_INTERVAL = 5*60*1000;
        static final int ASYNC_RESOLVER_WARMUP_TIMEOUT = 5000;
        static final int ASYNC_EXECUTOR_THREADPOOL_SIZE = 5;
    }
}

/**
 *@return the percentage of the full endpoints set above which the   
 *quarantine set is cleared in the range [0, 1.0]
 */
double getRetryableClientQuarantineRefreshPercentage();

```

看到这里就不难理解了，因为这个值是0.66而此时全量的Eureka Server值为4。计算之后的值为2，而由于注册的for循环为3次，所以当第二次发起注册流程的时候quarantineSet的值始终大于threshold。这样就会导致一个问题，就是如果8761、8762、8763一直是down即使8764一直是好的，那么Eureka Client也不会注册成功。而且这个参数值的区间为0到1.

既然通过源码分析我们找到了问题根源，其实对应的我们也找到了解决这个问题的办法，就是对应把这个参数值调大些。
这个值在properties中对应的写法如下：

```properties
eureka.client.transport.retryableClientQuarantineRefreshPercentage = xxx
```

接下来我们修改下properties文件，修改后的内容如下：

```properties
eureka.client.service-url.defaultZone=
http://localhost:8761/eureka,http://localhost:8762/eureka,http://localhost:8763/eureka,http://localhost:8764/eureka
eureka.client.transport.retryableClientQuarantineRefreshPercentage=1
```

接下来按照这个配置再次回顾下上面的流程：

- Eureka Client启动时进行注册(8761、8762、8763的状态是down)，所以此时quarantineSet的值为3.
- 接下来在定时任务中又触发注册事件，此时因为参数的值从0.66调整为1。所以计算出的threshold的值为4。而此时quarantineSet的值为3。所以不会进入到`else if (quarantineSet.size() >= threshold)`分支，而是会进入最后的esle分支。
- 在else分支中会完成过滤功能，最终返回的list中的结果只有一个就是8764这个Eureka Server。
- Eureka Client向8764这个Eureka Server发起注册请求，得到成功相应，并返回。

