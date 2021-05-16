---
title: Spring Cloud Alibaba 组件之Sentinel--分布式系统的流量防卫兵 (十)
date: 2019-11-5 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part8_alibaba>

#### Sentinel是什么

<!--more-->

[Sentinel](https://github.com/alibaba/Sentinel/wiki)的官方标题是：分布式系统的流量防卫兵。从名字上来看，很容易就能猜到它是用来作服务稳定性保障的。对于服务稳定性保障组件，如果熟悉Spring Cloud的用户，第一反应应该就是Hystrix。但是比较可惜的是Netflix已经宣布对Hystrix停止更新。那么，在未来我们还有什么更好的选择呢？除了Spring Cloud官方推荐的resilience4j之外，目前Spring Cloud Alibaba下整合的Sentinel也是用户可以重点考察和选型的目标。

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

#### 使用Sentinel实现接口限流

限流组件Sentinel

- Sentinel是把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- 默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- 自带控台动态修改限流策略。但是每次服务重启后就丢失了。所以它也支持ReadableDataSource 目前支持file, nacos, zk, apollo 这4种类型

Sentinel的使用分为两部分：

- sentinel-dashboard：与hystrix-dashboard类似，但是它更为强大一些。除了与hystrix-dashboard一样提供实时监控之外，还提供了流控规则、熔断规则的在线维护等功能。
- 客户端整合：每个微服务客户端都需要整合sentinel的客户端封装与配置，才能将监控信息上报给dashboard展示以及实时的更改限流或熔断规则等。

下面我们就分两部分来看看，如何使用Sentienl来实现接口限流。

**部署Sentinel控制台**

1. 安装：[Sentinel/releases](https://github.com/alibaba/Sentinel/releases)

2. 启动控制台

   执行 Java 命令 `java -jar sentinel-dashboard.jar` 默认的监听端口为 `8080`

   ```
   -Dserver.port=8888：指定端口
   -Dsentinel.dashboard.auth.username=sentinel: 用于指定控制台的登录用户名为 sentinel；
   -Dsentinel.dashboard.auth.password=123456: 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel
   -Dserver.servlet.session.timeout=7200: 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟；
   ```

3. 访问

   打开[http://localhost:8080](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080) 即可看到控制台界面

**接入Sentinel**

创建项目cloud-sentinel

1. 引入 Sentinel starter

   ```xml
           <!--Sentinel starter-->
           <!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-sentinel -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
               <version>2.1.1.RELEASE</version>
           </dependency>
   
   ```

2. 配置文件如下

   ```yml
   server:
     port: 8910
   
   
   spring:
     application:
       name: cloud-sentinel
   
     cloud:
       sentinel:
         #Sentinel 控制台地址
         transport:
           dashboard: localhost:8080
           #取消Sentinel控制台懒加载
         eager: true
   ```

3. 接入限流埋点

   Sentinel 默认为所有的 HTTP 服务提供了限流埋点。引入依赖后自动完成所有埋点。只需要在控制配置限流规则即可

   如果需要对某个特定的方法进行限流或降级，可以通过 @SentinelResource 注解来完成限流的埋点

   ```java
   @SentinelResource("resource")
   @RequestMapping("/sentinel/hello")
   public Map<String,Object> hello(){
           Map<String,Object> map=new HashMap<>(2);
           map.put("appName",appName);
           map.put("method","hello");
           return map;
   }
   ```

如果控制台没有找到自己的应用，可以先调用一下进行了 Sentinel 埋点的 URL 或方法或着禁用Sentinel 的赖加载`spring.cloud.sentinel.eager=true`

#### Sentinel的基本配置

```properties
spring.cloud.sentinel.enabled              #Sentinel自动化配置是否生效
spring.cloud.sentinel.eager               #取消Sentinel控制台懒加载
spring.cloud.sentinel.transport.dashboard   #Sentinel 控制台地址
spring.cloud.sentinel.transport.heartbeatIntervalMs        #应用与Sentinel控制台的心跳间隔时间
spring.cloud.sentinel.log.dir            #Sentinel 日志文件所在的目录
```

#### 配置限流规则

**配置 URL 限流规则**

控制器随便添加一个普通的http方法

```java
  /**
     * 通过控制台配置URL 限流
     * @return
     */
    @RequestMapping("/sentinel/test")
    public Map<String,Object> test(){
        Map<String,Object> map=new HashMap<>(2);
        map.put("appName",appName);
        map.put("method","test");
        return map;
    }
```

点击新增流控规则。为了方便测试阀值设为 1

![1YvtvF.png](https://s2.ax1x.com/2020/02/02/1YvtvF.png)

浏览器重复请求 http://localhost:8910/sentinel/test 如果超过阀值就会出现如下界面

![1YxEZ9.png](https://s2.ax1x.com/2020/02/02/1YxEZ9.png)

整个URL限流就完成了。但是返回的提示不够友好。

**配置自定义限流规则(@SentinelResource埋点)**

自定义限流规则就不是添加方法的访问路径。 配置的是@SentinelResource注解中value的值。比如`@SentinelResource("resource")`就是配置路径为resource

![1YxQMD.png](https://s2.ax1x.com/2020/02/02/1YxQMD.png)

访问：http://localhost:8910/sentinel/hello

通过`@SentinelResource`注解埋点配置的限流规则如果没有自定义处理限流逻辑，当请求到达限流的阀值时就返回404页面

**自定义限流处理逻辑**

@SentinelResource 注解包含以下属性：

- value: 资源名称，必需项（不能为空）
- entryType: 入口类型，可选项（默认为 EntryType.OUT）
- blockHandler:blockHandlerClass中对应的异常处理方法名。参数类型和返回值必须和原方法一致
- blockHandlerClass：自定义限流逻辑处理类



```java
 //通过注解限流并自定义限流逻辑
 @SentinelResource(value = "resource2", blockHandler = "handleException", blockHandlerClass = {ExceptionUtil.class})
 @RequestMapping("/sentinel/test2")
    public Map<String,Object> test2() {
        Map<String,Object> map=new HashMap<>();
        map.put("method","test2");
        map.put("msg","自定义限流逻辑处理");
        return  map;
    }

public class ExceptionUtil {

    public static Map<String,Object> handleException(BlockException ex) {
        Map<String,Object> map=new HashMap<>();
        System.out.println("Oops: " + ex.getClass().getCanonicalName());
        map.put("Oops",ex.getClass().getCanonicalName());
        map.put("msg","通过@SentinelResource注解配置限流埋点并自定义处理限流后的逻辑");
        return  map;
    }
}
```

控制台新增resource2的限流规则并设置阀值为1。访问http://localhost:8910/sentinel/test2 请求到达阀值时机会返回自定义的异常消息

![1tSxIA.png](https://s2.ax1x.com/2020/02/02/1tSxIA.png)

基本的限流处理就完成了。 但是每次服务重启后 之前配置的限流规则就会被清空因为是内存态的规则对象.所以下面就要用到Sentinel一个特性ReadableDataSource 获取文件、数据库或者配置中心是限流规则

**读取文件的实现限流规则**

一条限流规则主要由下面几个因素组成：

- resource：资源名，即限流规则的作用对象
- count: 限流阈值
- grade: 限流阈值类型（QPS 或并发线程数）
- limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
- strategy: 调用关系限流策略
- controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）
   SpringCloud alibaba集成Sentinel后只需要在配置文件中进行相关配置，即可在 Spring 容器中自动注册 DataSource，这点很方便。配置文件添加如下配置

```yml
spring:
  cloud:
    sentinel:
      #通过文件读取限流规则
      datasource:
        ds1:
          file:
            file: classpath:flowrule.json
            data-type: json
            rule-type: flow
```

在resources新建一个文件 比如flowrule.json 添加限流规则

```json
[
  {
    "resource": "resource",
    "controlBehavior": 0,
    "count": 1,
    "grade": 1,
    "limitApp": "default",
    "strategy": 0
  },
  {
    "resource": "resource3",
    "controlBehavior": 0,
    "count": 1,
    "grade": 1,
    "limitApp": "default",
    "strategy": 0
  }
]

```

重新启动项目。出现如下日志说明文件读取成功

```
 [Sentinel Starter] DataSource ds1-sentinel-file-datasource start to loadConfig
 [Sentinel Starter] DataSource ds1-sentinel-file-datasource load 2 FlowRule
```

刷新Sentinel 控制台 限流规则就会自动添加进去

**使用Nacos存储限流规则**

下面我们将同时使用到`Nacos`和`Sentinel Dashboard`，所以可以先把`Nacos`和`Sentinel Dashboard`启动起来。

默认配置下启动后，它们的访问地址（后续会用到）为：

- Nacos：http://localhost:8848/
- Sentinel Dashboard：http://localhost:8080/

应用配置：

1. 在Spring Cloud应用的`pom.xml`中引入Spring Cloud Alibaba的Sentinel模块和Nacos存储扩展：

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
   
           <!--Sentinel starter-->
           <!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-sentinel -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
               <version>2.1.1.RELEASE</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/com.alibaba.csp/sentinel-datasource-nacos -->
           <dependency>
               <groupId>com.alibaba.csp</groupId>
               <artifactId>sentinel-datasource-nacos</artifactId>
               <version>1.7.1</version>
           </dependency>
   ```

2. 在Spring Cloud应用中添加配置信息：

   ```yml
   spring:
     application:
       name: cloud-sentinel
   
     #Sentinel 控制台地址
     cloud:
       sentinel:
         transport:
           dashboard: localhost:8080
           #取消Sentinel控制台懒加载
         eager: true
         #通过文件读取限流规则
         datasource:
           ds2:
             nacos:
               server-addr: localhost:8848
               dataId: ${spring.application.name}-sentinel
               groupId: DEFAULT_GROUP
               rule-type: flow
   ```

   - `spring.cloud.sentinel.datasource.ds.nacos.server-addr`：nacos的访问地址，，根据上面准备工作中启动的实例配置
   - `spring.cloud.sentinel.datasource.ds.nacos.groupId`：nacos中存储规则的groupId
   - `spring.cloud.sentinel.datasource.ds.nacos.dataId`：nacos中存储规则的dataId
   - `spring.cloud.sentinel.datasource.ds.nacos.rule-type`：该参数是spring cloud alibaba升级到0.2.2之后增加的配置，用来定义存储的规则类型。所有的规则类型可查看枚举类：`org.springframework.cloud.alibaba.sentinel.datasource.RuleType`，每种规则的定义格式可以通过各枚举值中定义的规则对象来查看，比如限流规则可查看：`com.alibaba.csp.sentinel.slots.block.flow.FlowRule`

   这里对于dataId使用了`${spring.application.name}`变量，这样可以根据应用名来区分不同的规则配置。

3. 创建应用主类，并提供一个rest接口，比如：

   ```java
   @SpringBootApplication
   public class SentinelApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SentinelApplication.class, args);
       }
   
   }
   @Slf4j
   @RestController
   public class TestController {
       @GetMapping("/hello")
       public String hello() {
           return "hanyunpeng0521.github.io";
       }
   }
   ```

4. Nacos中创建限流规则的配置，比如：

   ![1tE0Vf.png](https://s2.ax1x.com/2020/02/02/1tE0Vf.png)

   其中：`Data ID`、`Group`就是上面**第二步**中配置的内容。配置格式选择JSON，并在配置内容中填入下面的内容：

   ```json
   [
       {
           "resource": "/hello",
           "limitApp": "default",
           "grade": 1,
           "count": 5,
           "strategy": 0,
           "controlBehavior": 0,
           "clusterMode": false
       }
   ]
   ```

   可以看到上面配置规则是一个数组类型，数组中的每个对象是针对每一个保护资源的配置对象，每个对象中的属性解释如下：

   - resource：资源名，即限流规则的作用对象
   - limitApp：流控针对的调用来源，若为 default 则不区分调用来源
   - grade：限流阈值类型（QPS 或并发线程数）；`0`代表根据并发数量来限流，`1`代表根据QPS来进行流量控制
   - count：限流阈值
   - strategy：调用关系限流策略
   - controlBehavior：流量控制效果（直接拒绝、Warm Up、匀速排队）
   - clusterMode：是否为集群模式

在完成了上面的整合之后，对于接口流控规则的修改就存在两个地方了：Sentinel控制台、Nacos控制台。

这个时候，需要注意当前版本的Sentinel控制台不具备同步修改Nacos配置的能力，而Nacos由于可以通过在客户端中使用Listener来实现自动更新。所以，在整合了Nacos做规则存储之后，需要知道在下面两个地方修改存在不同的效果：

- Sentinel控制台中修改规则：仅存在于服务的内存中，不会修改Nacos中的配置值，重启后恢复原来的值。
- Nacos控制台中修改规则：服务的内存中规则会更新，Nacos中持久化规则也会更新，重启后依然保持。

#### 自动配置

Sentinel如何自动配置到Spring boot中来的？

1. com.alibaba.cloud.sentinel.SentinelWebAutoConfiguration针对普通Web APP

   ```java
   @Configuration
   //尽可在Servlet的WebAPP中，不能在REACTIVE的Webapp
   @ConditionalOnWebApplication(
       type = Type.SERVLET
   )
   //默认注册的过滤器CommonFilter
   @ConditionalOnClass({CommonFilter.class})
   //属性必须为true，默认为true
   @ConditionalOnProperty(
       name = {"spring.cloud.sentinel.enabled"},
       matchIfMissing = true
   )
   //属性配置类
   @EnableConfigurationProperties({SentinelProperties.class})
   public class SentinelWebAutoConfiguration {
       private static final Logger log = LoggerFactory.getLogger(SentinelWebAutoConfiguration.class);
       @Autowired
       private SentinelProperties properties;
       @Autowired
       private Optional<UrlCleaner> urlCleanerOptional;
       @Autowired
       private Optional<UrlBlockHandler> urlBlockHandlerOptional;
       @Autowired
       private Optional<RequestOriginParser> requestOriginParserOptional;
   
       public SentinelWebAutoConfiguration() {
       }
   
       @PostConstruct
       public void init() {
           this.urlBlockHandlerOptional.ifPresent(WebCallbackManager::setUrlBlockHandler);
           this.urlCleanerOptional.ifPresent(WebCallbackManager::setUrlCleaner);
           this.requestOriginParserOptional.ifPresent(WebCallbackManager::setRequestOriginParser);
       }
   //注册过滤器
       @Bean
       @ConditionalOnProperty(
           name = {"spring.cloud.sentinel.filter.enabled"},
           matchIfMissing = true
       )
       public FilterRegistrationBean sentinelFilter() {
           FilterRegistrationBean<Filter> registration = new FilterRegistrationBean();
           com.alibaba.cloud.sentinel.SentinelProperties.Filter filterConfig = this.properties.getFilter();
           //默认过滤所有路径
           if (filterConfig.getUrlPatterns() == null || filterConfig.getUrlPatterns().isEmpty()) {
               List<String> defaultPatterns = new ArrayList();
               defaultPatterns.add("/*");
               filterConfig.setUrlPatterns(defaultPatterns);
           }
   
           registration.addUrlPatterns((String[])filterConfig.getUrlPatterns().toArray(new String[0]));
           Filter filter = new CommonFilter();
           registration.setFilter(filter);
           registration.setOrder(filterConfig.getOrder());
           registration.addInitParameter("HTTP_METHOD_SPECIFY", String.valueOf(this.properties.getHttpMethodSpecify()));
           log.info("[Sentinel Starter] register Sentinel CommonFilter with urlPatterns: {}.", filterConfig.getUrlPatterns());
           return registration;
       }
   }
   ```

2. org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication

   ```java
   @Target({ ElementType.TYPE, ElementType.METHOD })
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Conditional(OnWebApplicationCondition.class)
   public @interface ConditionalOnWebApplication {
   
   	/**
   	 * The required type of the web application.
   	 * @return the required web application type
   	 */
   	Type type() default Type.ANY;
   
   	/**
   	 * Available application types.
   	 */
   	enum Type {
   
   		/**
   		 * Any web application will match.
   		 */
   		ANY,
   
   		/**
   		 * Only servlet-based web application will match.
   		 */
   		SERVLET,
   
   		/**
   		 * Only reactive-based web application will match.
   		 */
   		REACTIVE
   
   	}
   
   }
   
   ```

3. SentinelProperties更多地使用内部类

   ```java
   @ConfigurationProperties(prefix = SentinelConstants.PROPERTY_PREFIX)
   @Validated
   public class SentinelProperties {
   
   	/**
   	 * Earlier initialize heart-beat when the spring container starts when the transport
   	 * dependency is on classpath, the configuration is effective.
   	 */
   	private boolean eager = false;
   
   	/**
   	 * Enable sentinel auto configure, the default value is true.
   	 */
   	private boolean enabled = true;
   
   	/**
   	数据源配置
   	 * Configurations about datasource, like 'nacos', 'apollo', 'file', 'zookeeper'.
   	 */
   	private Map<String, DataSourcePropertiesConfiguration> datasource = new TreeMap<>(
   			String.CASE_INSENSITIVE_ORDER);
   
   	/**
   	 * Transport configuration about dashboard and client.
   	 */
   	private Transport transport = new Transport();
   
   	/**
   	 * Metric configuration about resource.
   	 */
   	private Metric metric = new Metric();
   
   	/**
   	 * Web servlet configuration when the application is web, the configuration is
   	 * effective.
   	 */
   	private Servlet servlet = new Servlet();
   
   	/**
   	 * Sentinel filter when the application is web, the configuration is effective.
   	 */
   	private Filter filter = new Filter();
   
   	/**
   	 * Sentinel Flow configuration.
   	 */
   	private Flow flow = new Flow();
   
   	/**
   	日志配置
   	 * Sentinel log configuration {@link LogBase}.
   	 */
   	private Log log = new Log();
   
   	/**
   	 * Add HTTP method prefix for Sentinel Resource.
   	 */
   	private Boolean httpMethodSpecify = false;
   	
   	}
   ```

4. com.alibaba.cloud.sentinel.SentinelWebFluxAutoConfiguration针对REACTIVE的Web APP

   ```java
   @Configuration
   @ConditionalOnWebApplication(type = Type.REACTIVE)
   @ConditionalOnClass(SentinelReactorTransformer.class)
   @ConditionalOnProperty(name = "spring.cloud.sentinel.enabled", matchIfMissing = true)
   @EnableConfigurationProperties(SentinelProperties.class)
   public class SentinelWebFluxAutoConfiguration {
   
   	private static final Logger log = LoggerFactory
   			.getLogger(SentinelWebFluxAutoConfiguration.class);
   
   	private final List<ViewResolver> viewResolvers;
   	private final ServerCodecConfigurer serverCodecConfigurer;
   
   	@Autowired
   	private Optional<BlockRequestHandler> blockRequestHandler;
   
   	public SentinelWebFluxAutoConfiguration(
   			ObjectProvider<List<ViewResolver>> viewResolvers,
   			ServerCodecConfigurer serverCodecConfigurer) {
   		this.viewResolvers = viewResolvers.getIfAvailable(Collections::emptyList);
   		this.serverCodecConfigurer = serverCodecConfigurer;
   	}
   
   	@PostConstruct
   	public void init() {
   		blockRequestHandler.ifPresent(WebFluxCallbackManager::setBlockHandler);
   	}
   
   	@Bean
   	@Order(-2)
   	@ConditionalOnProperty(name = "spring.cloud.sentinel.filter.enabled", matchIfMissing = true)
   	public SentinelBlockExceptionHandler sentinelBlockExceptionHandler() {
   		return new SentinelBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
   	}
   
   	@Bean
   	@Order(-1)
   	@ConditionalOnProperty(name = "spring.cloud.sentinel.filter.enabled", matchIfMissing = true)
   	public SentinelWebFluxFilter sentinelWebFluxFilter() {
   		log.info("[Sentinel Starter] register Sentinel SentinelWebFluxFilter");
   		return new SentinelWebFluxFilter();
   	}
   
   }
   ```

5. com.alibaba.cloud.sentinel.datasource

   数据源配置：

   ```properties
   nacos = com.alibaba.csp.sentinel.datasource.nacos.NacosDataSource
   file =com.alibaba.csp.sentinel.datasource.FileRefreshableDataSource
   apollo = com.alibaba.csp.sentinel.datasource.apollo.ApolloDataSource
   zk = com.alibaba.csp.sentinel.datasource.zookeeper.ZookeeperDataSource
   redis = com.alibaba.csp.sentinel.datasource.redis.RedisDataSource
   ```

   对应的配置类位于com.alibaba.cloud.sentinel.datasource.config下