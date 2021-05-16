---
title: SpringCloud组件之Spring Cloud Config （七）
date: 2019-11-2 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part6_confog>

### 分布式配置中心

#### 为什么要统一管理微服务配置

随着业务的发展、微服务架构的升级，服务的数量、程序的配置日益增多（各种微服务、各种服务器地址、各种参数），传统的配置文件方式和数据库的方式已无法满足开发人员对配置管理的要求，即

- 安全性：配置跟随源代码保存在代码库中，容易造成配置泄漏；
- 时效性：修改配置，需要重启服务才能生效；
- 局限性：无法支持动态调整：例如日志开关、功能开关；

<!--more-->

**其实说白了，就是当业务需求有变更时，可以通过修改配置文件或者参数的形式，能够自动更新配置。减少不必要的重启服务的操作。正常情况下，一般的业务系统都有个参数配置表的，里面记录着不同业务参数，以此来应对不同的需求场景，也就是需求口中常说的：要能灵活配置。**

而在微服务中，由于每个微服务都是独立的数据库，传统的配置无法满足了。所以才出现了分布式配置中心服务，专门来解决此类问题的。

在微服务架构中，微服务的统一配置管理一般有以下需求：

- 集中管理配置：一个使用微服务架构的应用系统可能会包括成千上万个微服务，因此集中管理配置是非常有必要的。
- 不同环境不同配置：数据源配置在不同的环境(开发、测试、预发布、生产等)中是不同的。
- 运行期间可动态调整：可根据各个微服务的负载情况，动态调整数据源连接池大小或熔断阈值，并且在调整配置时不重启微服务。
- 配置修改后自动更新：如配置内容发生变化，微服务能够自动更新配置。

综上所述，对于微服务架构而言，一个通用的配置管理机制必不可少，常见做法是使用配置服务器管理配置。目前市面上开源的配置中心有很多，如

- **Apollo（阿波罗）**：携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
- **Qconf**：一个分布式配置管理工具，360出品。
- **Disconf**：专注于各种「分布式系统配置管理」的「通用组件」和「通用平台」, 提供统一的「配置管理服务」。

![18tDUA.png](https://s2.ax1x.com/2020/02/01/18tDUA.png)

### Spring Cloud Config

[Spring Cloud Config](https://spring.io/projects/spring-cloud-config)为分布式系统外部化配置提供了服务端和客户端的支持，包括Config Server和Config Client两部分，其架构图如下图所示：

![1QuYbF.png](https://s2.ax1x.com/2020/01/29/1QuYbF.png)

各个微服务在启动时会请求Config Server以获取所需要的配置属性，然后缓存这些属性以提高性能

目前支持`git`、`svn`、`vault`、`jdbc`和`本地`几种存储方式。最常用的存储方式就是`git`了。简单来说，各客户端程序通过访问服务端获取相应的配置信息。接下来我们看看下面这张图

![18lZLR.png](https://s2.ax1x.com/2020/01/31/18lZLR.png)

- 远程Git仓库：存储配置文件。
- ConfigServer：分布式配置管理中心，会于维护自己的git仓库信息。
- 本地Git仓库：在ConfigServer中，每次客户端请求获取配置信息时，都会从git仓库获取最新的配置到本地，然后本地读取并返回，远程无法获取时，使用本地仓库信息。

从上图可以看出，`Config Server`巧妙地通过`git clone`将配置信息存于本地，起到了缓存的作用，即使当`Git`服务端无法访问的时候，依然可以取`Config Server`中的缓存内容进行使用。

#### 示例

文章源码：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part6_config>

需要用到一些已放到git的配置文件，这里我已将其放到了github方便大家可以直接拿来测试用，仓库地址为：<https://github.com/hanyunpeng0521/spring-cloud-learn.config>,**default和dev进行加密，后续会进行设置**

```
端点与配置文件的映射规则如下：

/{application}/{profile}[/{label}]

/{application}-{profile}.yml

/{label}/{application}-{profile}.yml

/{application}-{profile}.properties

/{label}/{application}-{profile}.properties

其中，application: 表示微服务的虚拟主机名，即配置的spring.application.name

profile: 表示当前的环境，dev, test or production?

label: 表示git仓库分支，master or relase or others repository name? 默认是master
```

注册中心的配置参考前面的文章。

##### Server端

创建工程：`confg-server`

1. 引入pom依赖。

   ```xml
           <!-- eureka -->
           <!-- 客户端依赖 -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
   
   
           <!-- spring cloud config -->
   
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-server</artifactId>
           </dependency>
   
           <!-- actuator -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   
           <!-- 热启动，热部署依赖包，为了调试方便，加入此包 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <optional>true</optional>
           </dependency>
   ```

2. 启动类加入@EnableConfigServer注解，声明是`ConfigServer`。

   ```java
   @SpringBootApplication
   @EnableConfigServer
   @EnableDiscoveryClient
   public class ConfigServiceApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ConfigServiceApplication.class, args);
       }
   
   }
   ```

3. 配置文件

   ```yml
   server:
     port: 8080
   management:
     security:
       enabled: false
   
   spring:
     application:
       name: sampleservice-config-server
     cloud:
       config:
         server:
           git:
             # 配置Git仓库地址
             uri: https://github.com/hanyunpeng0521/spring-cloud-learn.config
             default-label: master
             refresh-rate: 10
             # 搜索路径，即配置文件的目录，可配置多个，逗号分隔。默认为根目录
             searchPaths: 
       #              # Git仓库账号（如果需要认证）
   #          username: 
   #              # Git仓库密码（如果需要认证）
   #          password: 
   #注册到注册中心
   eureka:
     client:
       serviceUrl:
         defaultZone: http://127.0.0.1:8761/eureka/
   
   ```

4. 启动应用，访问<http://127.0.0.1:8080/sampleservice-foo-test.properties>回了配置文件的信息，说明已经读取到远程仓库信息了。

   根据映射关系访问<http://127.0.0.1:8080/sampleservice-foo/test/master>,返回的信息:

   ```json
   {
   	"name": "sampleservice-foo",
   	"profiles": ["test"],
   	"label": "master",
   	"version": "f8689feff69ed1c64953656886841eb2dfb88f73",
   	"state": null,
   	"propertySources": [{
   		"name": "https://github.com/hanyunpeng0521/spring-cloud-learn.config/sampleservice-foo-test.properties",
   		"source": {
   			"profile": "test-1.0"
   		}
   	}, {
   		"name": "https://github.com/hanyunpeng0521/spring-cloud-learn.config/sampleservice-foo.properties",
   		"source": {
   			"profile": "{crypt}fb1b5fb438c2ad2f0262805c89faf0de54cc3e4feb4c9668d0c75445ac808588"
   		}
   	}]
   }
   ```

   此时，查看控制台，可以获悉本地也保存着一份配置信息。

   ```
   2020-01-31 23:04:14.443  INFO 9359 --- [nio-8080-exec-8] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/tmp/config-repo-6716417217928496351/sampleservice-foo-test.properties
   2020-01-31 23:04:14.443  INFO 9359 --- [nio-8080-exec-8] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/tmp/config-repo-6716417217928496351/sampleservice-foo.properties
   ```

   此时，修改远程的配置文件，再次访问可以看见返回的参数是最新修改后的参数值了，大家可以自行试试。

##### Client端

创建一个客户端:`confg-client`。当然也可以改造原来的应用了，只需加入相应pom文件和配置文件即可。

1. 加入pom依赖

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
   
           <!-- spring cloud bus -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-bus-amqp</artifactId>
           </dependency>
   
           <!-- eureka -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
   
           <!-- spring cloud config -->
   
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-client</artifactId>
           </dependency>
   
           <!-- actuator -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   
           <!-- 热启动，热部署依赖包，为了调试方便，加入此包 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <optional>true</optional>
           </dependency>
   ```

2. 创建启动类，就是一个正常的web应用。

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class ConfigClientApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ConfigClientApplication.class, args);
       }
   
   }
   ```

3. 配置文件添加：`bootstrap.properties`和常规的`application.properties`。

   ```yml
   #bootstrap.yml
   spring:
     application:
       name: sampleservice-foo
     cloud:
       config:
         # config server的地址，如果不使用注册中心，需要配置该项
         #uri: http://127.0.0.1:8080/
         # profile对应config server所获取的配置文件中的{profile}
         profile: prod
         # 指定Git仓库的分支，对应config server所获取的配置文件的{label}
         label: master
         discovery:
           enabled: true    # 表示使用服务发现组件中的Config Server，而不自己指定Config Server的uri，默认false
           service-id: sampleservice-config-server  # 指定Config Server在服务发现中的serviceId，默认是configserver
   eureka:
     client:
       serviceUrl:
         defaultZone: http://127.0.0.1:8761/eureka/
   
   #application.yml
   server:
     port: 8081
   
   #开启refresh接口
   management:
     endpoint:
       shutdown:
         enabled: false
     endpoints:
       web:
         exposure:
           include: "*"
   ```

   **这里需要注意：**

   > `spring-cloud-config`相关的属性**必须配置在`bootstrap.properties`中**，config部分内容才能被正确加载。因为config的相关配置会先于`application.properties`，而`bootstrap.properties`的加载也是先于`application.properties`。

4. 编写一个控制层，利用`@Value`进行参数测试

   ```java
   @RestController
   @RefreshScope // @RefreshScope注解不能少，否则即使调用/refresh，配置也不会刷新
   public class ConfigClientController {
       @Value("${profile}")
       private String profile;
   
       @GetMapping("/profile")
       public String hello() {
           return this.profile;
       }
   }
   ```

5. 启动应用，访问：<http://127.0.0.1:8081>,可以看见配置信息已经被正确返回了

自此，一个简单的配置中心示例就结束了。 **目前为止，我们还没有手动去修改远程的配置文件参数值，可以试试，在修改后，客户端去返回相应的参数值，会发现还是旧的，并没有进行更新操作。因为配置文件是是在应用启动的时候进行加载的，而且远程仓库修改了配置文件，客户端并不知道已经修改了，不会发起请求的。**

#### 高可用

将配置中心服务化，本身是为了实现高可用。而实现高可用的手段是很多的，最常用的就是`负载均衡`。客户端不直连服务端，而是访问`负载均衡`服务，由`负载均衡`来动态选择需要访问的服务端。只是`Spring Cloud Config`天然的就能进行服务化配置，所以，实际中可以根据实际的业务需求来进行合理化抉择的。

![188ZLj.png](https://s2.ax1x.com/2020/01/31/188ZLj.png)

其次，对于使用了`git`或者`svn`作为存储方式时，本身配置仓库的高可用也是一个需要考虑的事项。本身如`github`或者`码云`这些第三方`git`仓库而言，已经实现了高可用了。但一般上部署的微服务都是内网服务，所以一般上是使用如`gitlab`开源的`git`仓库管理系统进行自建，此时就需要考虑本身仓库的高可用了。

我们上面的示例使用的是github存储配置，github本身就是高可用的，因此需要考虑的是config-server的高可用，上面我们已经使用了eureka注册中心，也已经配置完成，我们可以试着运行多个config-server实例

#### refresh实现刷新

在默认情况下，客户端是不会自动感知配置的变化的。此时，我们可以使用`/refresh`端点来进行配置更新。 现在，我们改造下**客户端**。

1. 加入端点依赖

   ```yml
           <!-- actuator -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   ```

2. 修改配置，这里需要注意,`2.0`之后，默认只开启了端点`info`、`health`。其他的需要通过`management.endpoints.web.exposure.include`进行额外配置。

   ```yml
   
   #开启refresh接口
   management:
     endpoint:
       shutdown:
         enabled: false
     endpoints:
       web:
         exposure:
           include: refresh
   ```

3. 修改controller类

   ```java
   @RestController
   @RefreshScope // @RefreshScope注解不能少，否则即使调用/refresh，配置也不会刷新
   public class ConfigClientController {
       @Value("${profile}")
       private String profile;
   
       @GetMapping("/profile")
       public String hello() {
           return this.profile;
       }
   }
   ```

4. 启动应用，修改远程仓库内容，使用`Postman`使用`POST`访问：`http://127.0.0.1:8081/actuator/refresh`。

   返回值即为有变动的参数值。再次访问可以获取最新值

#### 配置的批量刷新

我们改造服务端和客户端

![1QuL5j.png](https://s2.ax1x.com/2020/01/29/1QuL5j.png)

1. 给配置中心模块添加依赖

   ```xml
    <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-bus-amqp</artifactId>
           </dependency>
           <!--服务客户端 -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
   ```

2. 修改配置中心配置文件，主要是连接rabbitmq，以及注册到服务中心中，还有一个开启所有端点，主要是bus-refresh端点

   ```yml
   server:
     port: 8080
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   
   spring:
     application:
       name: sampleservice-config-server
     cloud:
       config:
         server:
           git:
             # 配置Git仓库地址
             uri: https://github.com/hanyunpeng0521/spring-cloud-learn.config
             default-label: master
             refresh-rate: 10
       #              # Git仓库账号（如果需要认证）
   #          username: m13839441583@163.com
   #              # Git仓库密码（如果需要认证）
   #          password: hyp212655
       bus:
         trace:
           enabled: true # 开启cloud bus跟踪
     rabbitmq:
       host: 127.0.0.1
       port: 5672
       username: guest
       password: guest
       virtual-host: my_vhost
   eureka:
     client:
       serviceUrl:
         defaultZone: http://127.0.0.1:8761/eureka/
   ```

3. 给客户端微服务都加入以下依赖

   ```xml
   
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-bus-amqp</artifactId>
           </dependency>
   ```

4. 客户端微服务都修改bootstrap.yml文件，用于连接rabbitmq，连接配置中心

   ```yml
   spring:
     application:
       name: sampleservice-foo
     cloud:
       config:
         # config server的地址
         uri: http://127.0.0.1:8080/
         # profile对应config server所获取的配置文件中的{profile}
         profile: prod
         # 指定Git仓库的分支，对应config server所获取的配置文件的{label}
         label: master
         discovery:
           enabled: true                            # 表示使用服务发现组件中的Config Server，而不自己指定Config Server的uri，默认false
           service-id: sampleservice-config-server  # 指定Config Server在服务发现中的serviceId，默认是configserver
   
     rabbitmq:
       host: 127.0.0.1
       port: 5672
       username: guest
       password: guest
       virtual-host: my_vhost
   eureka:
     client:
       serviceUrl:
         defaultZone: http://127.0.0.1:8761/eureka/
   ```

**存在的问题：如果调用127.0.0.1:8080/bus/refresh，此地址来刷新服务，则127.0.0.1:8080，节点也不会做通知，不会更新服务的配置信息**

到此，实现了配置的半自动更新；

##### 配置的全自动更新

要做到配置信息的自动更新，只差一步，那就是修改配置文件后，自动调用刷新方法,前提是有公网ip

这只需要在github、gitlab或码云上做配置即可：

以码云为例：

![18JMxU.png](https://s2.ax1x.com/2020/01/31/18JMxU.png)

到此就可以实现，配置的自动刷新

#### 配置文件加密

配置项中通常会包括一些敏感的信息，比如密码等，config server支持对这些敏感信息进行对称加密保存，在运行时解密。

##### config server开启加密功能

config server端开启加密功能需要以下几步

1. 下载并安装JCEPolicy
2. 配置对称加密的密钥
3. 验证加密功能

**下载并安装JCEPolicy**

使用下面的脚本下载jce_policy-8.zip文件，解压后，复制UnlimitedJCEPolicyJDK8下的两个jar文件，放到`${JAVA_HOME}/jre/lib/security`目录下

```shell
#   下载文件
curl -q -L -C - -b "oraclelicense=accept-securebackup-cookie" -o /tmp/jce_policy-8.zip -O http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip

#   解压。解压后会得到一个UnlimitedJCEPolicyJDK8文件夹
unzip /tmp/jce_policy-8.zip

#   拷贝文件到指定的文职
cp UnlimitedJCEPolicyJDK8/*.jar ${JAVA_HOME}/jre/lib/security

```

**配置对称加密的密钥**

这一步主要是配置一个复杂的密钥。可以直接在config server的`bootstrap.yml`配置文件中加入一行配置

```yml
encrypt:
  key: \@PINGXIN123456789123456789pingxin
```

如果你不想把密钥写到代码中，还可以在运行config server的服务器上设置一个环境变量

```
export ENCRYPT_KEY=@PINGXIN123456789123456789pingxin
```

以上两种方式都可以。

另外，为了不让加密前的明文通过http接口直接展示，还要在`bootstrap.properties`

```yml
spring:
  cloud:
    config:
      server:
        encrypt:
          enabled: false
```

**验证加密功能**

config server在启动时如果检测到了对应的配置项或环境变量，会自动启用加密功能，并新增两个POST接口 `/encrypt`和 `/decrypt`。这两个接口通过后台配置的密钥来进行加密和解密

##### 对配置项进行加密存储

在git仓库中加入包含加密内容的配置项，如**sampleservice-foo-dev.properties**：

```properties
profile={cipher}85afc1c2f72fa830b4878310ece3dce67a52028e83167c169049d18cf3b01168
```

`{cipher}`是一个前缀，如果一个配置项的值是以这个前缀开头的，则表示这是一个加密的内容，客户端在获取到以后，会自动对其解密。

修改config-client客户端微服务配置文件bootstrap.yml

```
      profile: dev
```

##### 使用非对称加密

1. 使用命令行工具，如config--server项目中使用bash，在其中输入：

   ```shell
   keytool -genkeypair -alias mytestkey -keyalg RSA \
     -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
     -keypass pingxin -keystore server.jks -storepass hyp0521
   ```

   此时会在config-server项目中根目录产生一个server.jks文件，保存的是非对称加密的密钥，打开后是乱码。

2. 将server.jks文件放入config-server项目的classpath下，在config-server项目中新建 bootstrap.yml，注意，必须是这个文件名。在其中配置如下，其中的值与生成时的命令行对应：

   ```yml
   #非对称加密
   encrypt:
    keyStore:
    location: classpath:/server.jks
       password: hyp0521
       alias: mytestkey
       secret: pingxin
   ```

3. 此时，启动config-server工程，依旧可以使用/encrypt和/decrypt接口进行属性的加密和解密，不过和对称加密的相比，密文长度更长，也更安全。同时，之前对称加密的密文，无法解密

### Applo

[Apollo](https://github.com/ctripcorp/apollo)（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。
Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

Apollo整体架构原理：

![18NpP1.png](https://s2.ax1x.com/2020/02/01/18NpP1.png)

#### 特性

- 统一管理不同环境、不同集群的配置
  - Apollo提供了一个统一界面集中式管理不同环境（environment）、不同集群（cluster）、不同命名空间（namespace）的配置。
  - 同一份代码部署在不同的集群，可以有不同的配置，比如zookeeper的地址等
  - 通过命名空间（namespace）可以很方便地支持多个不同应用共享同一份配置，同时还允许应用对共享的配置进行覆盖
- 配置修改实时生效（热发布）
  - 用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序
- 版本发布管理
  - 所有的配置发布都有版本概念，从而可以方便地支持配置的回滚
- 灰度发布
  - 支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例
- 权限管理、发布审核、操作审计
  - 应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节，从而减少人为的错误。
  - 所有的操作都有审计日志，可以方便地追踪问题
- 客户端配置信息监控
  - 可以在界面上方便地看到配置在被哪些实例使用
- 提供Java和.Net原生客户端
  - 提供了Java和.Net的原生客户端，方便应用集成
  - 支持Spring Placeholder, Annotation和Spring Boot的ConfigurationProperties，方便应用使用（需要Spring 3.1.1+）
  - 同时提供了Http接口，非Java和.Net应用也可以方便地使用
- 提供开放平台API
  - Apollo自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。不过Apollo出于通用性考虑，不会对配置的修改做过多限制，只要符合基本的格式就能保存，不会针对不同的配置值进行针对性的校验，如数据库用户名、密码，Redis服务地址等
  - 对于这类应用配置，Apollo支持应用方通过开放平台API在Apollo进行配置的修改和发布，并且具备完善的授权和权限控制
- 部署简单
  - 配置中心作为基础服务，可用性要求非常高，这就要求Apollo对外部依赖尽可能地少
  - 目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来
  - Apollo还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数

由于apollo相比spring cloud config等主流配置中心多了可视化管理配置和灰度发布等高级特性，所以打算放弃spring cloud config 改用apollo。

#### 部署apollo

Apollo项目中内置了Eureka注册中心，如果我们想使用自己外部独立的注册中心，就要把Apollo项目中的Eureka剥离。

1. 到github apollo的release页面下载需要的三个zip包。
   https://github.com/ctripcorp/apollo/releases

   ![1GY12V.png](https://s2.ax1x.com/2020/02/01/1GY12V.png)

2. 安装mysql 5.7或者以后版本，然后运行如下两个apollo初始化脚本，初始化两个基本库。

   https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloportaldb.sql

   https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloconfigdb.sql

3. 将三个zip包上传到服务器并解压，每个zip包解压出来都有一个application-github.properties文件，修改其中的mysql ip地址、端口、用户名、密码信息，指向本地的mysql数据库，然后注意将application-github.properties文件放在jar包同级目录，如下是我的目录结构和application-github.properties文件内容。

   ```shell
   [root@localhost apollo]# ls
   admin-service  config-service  protal
   [root@localhost admin-service]# ls
   apollo-adminservice-1.4.0.jar  application-github.properties  app.properties  shutdown.sh  startup.sh
   [root@localhost protal]# ls
   apollo-env.properties  apollo-portal-1.4.0.jar  application-github.properties  app.properties  shutdown.sh  startup.sh
   [root@localhost config-service]# ls
   apollo-configservice-1.4.0.jar  application-github.properties  app.properties  shutdown.sh  startup.sh
   
   # cat application-github.properties
   spring.datasource.url = jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8
   spring.datasource.username = root
   spring.datasource.password = 123456
   ```

4. 修改portal目录下的apollo-env.properties文件，填上对应的各个环境配置中心地址，我是单机版所以都填了本地：

   ```shell
   [root@localhost protal]# cat apollo-env.properties 
   local.meta=http://localhost:8080
   dev.meta=http://localhost:8080
   fat.meta=http://localhost:8080
   uat.meta=http://localhost:8080
   lpt.meta=${lpt_meta}
   pro.meta=http://localhost:8080
   ```

5. 依次运行三个jar包：

   ```
   java -jar apollo-configservice-1.4.0.jar
   java -jar apollo-adminservice-1.4.0.jar
   java -jar apollo-portal-1.4.0.jar
   ```

6. 浏览器访问8070端口，看到如下界面说明启动成功，初始用户名密码为：apollo/admin

   ![1GUF81.png](https://s2.ax1x.com/2020/02/01/1GUF81.png)

   



### 参考

1. [分布式配置中心选型](https://www.cnblogs.com/xiaodf/p/11214775.html)
2. [微服务之SpringCloud架构第六篇（上）——配置中心（Apollo）](https://blog.csdn.net/pilihaotian/article/details/82956798)
3. [Spring Cloud配置文件加密](https://blog.csdn.net/u014792352/article/details/73163714)