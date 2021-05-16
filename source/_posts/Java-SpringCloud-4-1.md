---
title: Spring Cloud  组件之Spring Cloud Gateway-- 微服务网关
date: 2019-11-8 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

作为Netflix Zuul的替代者，Spring Cloud Gateway是一款非常实用的微服务网关，在Spring Cloud微服务架构体系中发挥非常大的作用

<!--more-->

Spring cloud gateway是spring官方基于Spring 5.0、Spring Boot2.0和Project Reactor等技术开发的网关，Spring Cloud Gateway旨在为微服务架构提供简单、有效和统一的API路由管理方式，Spring Cloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Netflix Zuul，其不仅提供统一的路由方式，并且还基于Filer链的方式提供了网关基本的功能，例如：安全、监控/埋点、限流等。

### 示例

#### 使用

这里使用nacos当做注册中心，

**依赖**

```xml
<properties>
            <!-- 统一依赖管理 -->
        <spring.boot.version>2.2.5.RELEASE</spring.boot.version>
        <spring-cloud.version>Hoxton.SR5</spring-cloud.version>
        <spring-cloud-alibaba.version>2.2.1.RELEASE</spring-cloud-alibaba.version>
</properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <artifactId>spring-boot-starter-parent</artifactId>
                <groupId>org.springframework.boot</groupId>
                <version>2.3.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--Spring Cloud 相关依赖-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--Spring Cloud Alibaba 相关依赖-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
</dependencyManagement>
    
<dependencies>
<!-- 配置中心 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <version>3.0.0</version>
    </dependency>
<!-- 注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
</dependencies>
```

解决spring-cloud-starter-gateway与spring-boot-starter-web Jar包冲突:

```xml
<exclusions>
     <exclusion>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
     </exclusion>
 </exclusions>
```

**配置文件**：

```yaml
server:
  port: 8201
spring:
  profiles:
    active: '@profileActive@'
  cloud:
      nacos:
      discovery:
        server-addr: http://127.0.0.1:8848
      config:
        server-addr: http://127.0.0.1:8848
        file-extension: yaml
    gateway:
      discovery:
        locator:
          enabled: true
          #使用小写service-id
          lower-case-service-id: true
      # 路由（routes：路由，它由唯一标识（ID）、目标服务地址（uri）、一组断言（predicates）和一组过滤器组成（filters）。filters 不是必需参数。）
      routes:
      # 路由标识（id：标识，具有唯一性）   简单尝试
        - id: px-mall-auth
        # 目标服务地址（uri：地址，请求转发后的地址）
          uri: lb://px-mall-auth
          # 路由条件（predicates：断言，匹配 HTTP 请求内容）
          predicates:
            - Path=/px-mall-auth/**
          filters:
            - StripPrefix=1
secure:
  ignore:
    urls: #配置白名单路径
      - "/doc.html"
      - "/swagger-resources/**"
      - "/swagger/**"
      - "/**/v2/api-docs"
      - "/**/*.js"
      - "/**/*.css"
      - "/**/*.png"
      - "/**/*.ico"
      - "/webjars/springfox-swagger-ui/**"
      - "/actuator/**"
```



### 核心概念

网关提供API全托管服务，丰富的API管理功能，辅助企业管理大规模的API，以降低管理成本和安全风险，包括协议适配、协议转发、安全策略、防刷、流量、监控日志等贡呢。一般来说网关对外暴露的URL或者接口信息，我们统称为路由信息。如果研发过网关中间件或者使用过Zuul的人，会知道网关的核心是Filter以及Filter Chain（Filter责任链）。Sprig Cloud Gateway也具有路由和Filter的概念。下面介绍一下Spring Cloud Gateway中几个重要的概念。

- 路由。路由是网关最基础的部分，路由信息有一个ID、一个目的URL、一组断言和一组Filter组成。如果断言路由为真，则说明请求的URL和配置匹配

- 断言。Java8中的断言函数。Spring Cloud Gateway中的断言函数输入类型是Spring5.0框架中的ServerWebExchange。Spring Cloud Gateway中的断言函数允许开发者去定义匹配来自于http request中的任何信息，比如请求头和参数等。

- 过滤器。一个标准的Spring webFilter。Spring cloud gateway中的filter分为两种类型的Filter，分别是Gateway Filter和Global Filter。过滤器Filter将会对请求和响应进行修改处理

![12KImQ.png](https://s2.ax1x.com/2020/02/07/12KImQ.png)

如上图所示，Spring cloudGateway发出请求。然后再由Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway web handler。Handler再通过指定的过滤器链将请求发送到我们实际的服务执行业务逻辑，然后返回。

1. 请求发送到网关，DispatcherHandler是Http请求的中央分发器，将请求匹配到相应的HandlerMapping;
2. 请求和处理器之间有一个映射关系，网关将会对请求进行路由，handler会匹配RoutePredicateHandlerMapping，以匹配到对应的Route；
3. 经过RoutePredicateHandlerMapping处理后，请求会发送到Web处理器，该WebHandler代理了一系列网关过滤器和全局过滤，此时会对请求或者响应头进行处理；
4. 最后转发到具体的代理服务；

> DispatcherHandler--->RoutePredicateHandlerMapping
>  RoutePredicateHandlerMapping---->FilteringWebHandler
>  FilteringWebHandler----->DefaultGatewayFilterChain

#### Route路由

什么是路由？一个路由就代表一个真实的下游请求路径，路由在gateway里面有两个重要的辅助概念-路由定义和定位器；

- 路由定义RouteDefinition:路由定义是Gateway Properties中的属性，可以定义获取路由的方式；
- 定位器Locator：定位器分为路由定位器RouteLocatorRouteDefinitionLocator，
   RouteLocator用来获取routes,而RouteDefinitionLocator则用来获RouteDefinitions；

#### RouteDefinition路由定义

路由定义有五个基本属性：

- id: 路由id,默认为uuid;
- predicates: 路由断言定义列表；
- fliters:该路由需要经过的过滤器；
- URI:该路由请求的路径；
- order:优先级 ；

### 定位器

#### RouteLocator

用来获取路由的定位器，其源码如下:

```csharp
public interface RouteLocator {

    Flux<Route> getRoutes();
}
```

其内置了3个路由定位器实现类：

- CachingRouteLocator:具备缓存功能的路由定位器，其内具有监听RefreshRoutesEvent事件，并能及时刷新缓存；

```css
    @EventListener(RefreshRoutesEvent.class)
    /* for testing */ void handleRefresh() {
        refresh();
    }
```

- CompositeRouteLocator:组合路由定位器，其内有代理了一系列的定位器，来实现路由查询；
- RouteDefinitionRouteLocator:基于路由定义的路由定位器，根据路由定义定位器获取路由定义并将其转换成对应的路由；代码如下

```java
public Flux<Route> getRoutes() {
        return this.routeDefinitionLocator.getRouteDefinitions()
                .map(this::convertToRoute)
                //TODO: error handling
                .map(route -> {
                    if (logger.isDebugEnabled()) {
                        logger.debug("RouteDefinition matched: " + route.getId());
                    }
                    return route;
                });
    }
```

也可以自定义一个路由定位器，如下代码:

```rust
@SpringBootApplication
public class DemogatewayApplication {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("path_route", r -> r.path("/get")
                .uri("http://httpbin.org"))//将所有/get开头的请求路径转发到httpbin.org
            .route("host_route", r -> r.host("*.myhost.org")
                .uri("http://httpbin.org"))//将域名以myhost.org结尾的请求转发
            .route("rewrite_route", r -> r.host("*.rewrite.org")
                .filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/${segment}"))
                .uri("http://httpbin.org"))//将域名以myhost.org结尾的请求转发并进行路径重写，保留/foo/后面路径；
            .route("hystrix_route", r -> r.host("*.hystrix.org")
                .filters(f -> f.hystrix(c -> c.setName("slowcmd")))
                .uri("http://httpbin.org"))
            .route("hystrix_fallback_route", r -> r.host("*.hystrixfallback.org")
                .filters(f -> f.hystrix(c -> c.setName("slowcmd").setFallbackUri("forward:/hystrixfallback")))
                .uri("http://httpbin.org"))
            .route("limit_route", r -> r
                .host("*.limited.org").and().path("/anything/**")
                .filters(f -> f.requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
                .uri("http://httpbin.org"))
            .build();
    }
}
```

#### 路由定义定位器

路由定义定位器用来获取路由定义的，路由定义可以在配置文件中进行设置,gateway内置了三个路由定义定位器：

- CompositeRouteDefinitionLocator：其内代理了一系列RouteDefinitionLocators,通过其来获取路由定义集合;
- RouteDefinitionRepository & InMemoryRouteDefinitionRepository:该定位器实现了路由定义的增删改查等操作，gateway内置端点接口会用到这些操作，同时也可以通过该路由定义定位器可以实现路由仓库，来动态更新路由定义信息；
- CachingRouteDefinitionLocator:带有缓存功能路由定义定位器；
- DiscoveryClientRouteDefinitionLocator：服务发现路由定义定位器，通过服务发现机制以及Spel表达式来动态更新路由定义信息；获取路由源码如下:

```dart
@Override
    public Flux<RouteDefinition> getRouteDefinitions() {
                 //获取Spel表达式
        SpelExpressionParser parser = new SpelExpressionParser();
        Expression includeExpr = parser.parseExpression(properties.getIncludeExpression());
        Expression urlExpr = parser.parseExpression(properties.getUrlExpression());

        Predicate<ServiceInstance> includePredicate;
        if (properties.getIncludeExpression() == null || "true".equalsIgnoreCase(properties.getIncludeExpression())) {
            includePredicate = instance -> true;
        } else {
            includePredicate = instance -> {
                Boolean include = includeExpr.getValue(evalCtxt, instance, Boolean.class);
                if (include == null) {
                    return false;
                }
                return include;
            };
        }
              //获取service clients以及其instances，通过serviceId替换的方式获取路由定义
        return Flux.fromIterable(discoveryClient.getServices())
                .map(discoveryClient::getInstances)
                .filter(instances -> !instances.isEmpty())
                .map(instances -> instances.get(0))
                .filter(includePredicate)
                .map(instance -> {
                    String serviceId = instance.getServiceId();

                    RouteDefinition routeDefinition = new RouteDefinition();
                    routeDefinition.setId(this.routeIdPrefix + serviceId);
                    String uri = urlExpr.getValue(evalCtxt, instance, String.class);
                    routeDefinition.setUri(URI.create(uri));

                    final ServiceInstance instanceForEval = new DelegatingServiceInstance(instance, properties);

                    for (PredicateDefinition original : this.properties.getPredicates()) {
                        PredicateDefinition predicate = new PredicateDefinition();
                        predicate.setName(original.getName());
                        for (Map.Entry<String, String> entry : original.getArgs().entrySet()) {
                            String value = getValueFromExpr(evalCtxt, parser, instanceForEval, entry);
                            predicate.addArg(entry.getKey(), value);
                        }
                        routeDefinition.getPredicates().add(predicate);
                    }

                    for (FilterDefinition original : this.properties.getFilters()) {
                        FilterDefinition filter = new FilterDefinition();
                        filter.setName(original.getName());
                        for (Map.Entry<String, String> entry : original.getArgs().entrySet()) {
                            String value = getValueFromExpr(evalCtxt, parser, instanceForEval, entry);
                            filter.addArg(entry.getKey(), value);
                        }
                        routeDefinition.getFilters().add(filter);
                    }

                    return routeDefinition;
                });
    }
```

- PropertiesRouteDefinitionLocator： 通过配置文件来获取路由定义集合，其源码如下：

```java
@Override
    public Flux<RouteDefinition> getRouteDefinitions() {
        return Flux.fromIterable(this.properties.getRoutes());
    }
```

### Predicate断言

Predicate通过Predicate函数式接口来判断当前请求是否满足选择条件，Predicate分为以下条件划分为不同的路由断言工厂类

#### Datetime类型

根据日期判断是否满足路由选择条件，其有三个工厂类:

- AfteRoutePredicateFactory: 接收一个日期参数，判断请求日期是否晚于指定日志;
- BeforeRoutePredicateFactory:接收一个日期参数，判断请求日期是否早于指定日期；
- BetweenRoutePredicateFactory:接收两个日期参数，判断请求日期是否位于之间;
   其配置如下：

```yaml
spring:
    cloud: 
        gateway:
            routes: 
            - id: after_route_id
            uri: http://www.baidu.com
            predicates: 
            - After= 2018-12-30T23:59:59.789+08:00[Asia/Shanghai]
```

#### Cookie类型

根据请求的Cookie正则匹配是否通过选择条件;
 唯一工厂类：CookieRoutePredicateFactory;
 其配置实例如下:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

#### Header头断言类型

请求头属性正则匹配是否通过选择，唯一工厂类：HeaderRoutePredicateFactory,其配置如下:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

#### Host域名断言类型

Host域名正则匹配，是否通过选择，唯一工厂类：HostRoutePredicateFactory,
 其配置如下:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

#### Method断言类型

基于方法类型来进行相应匹配选择，唯一工厂类MethodRoutePredicateFactory;
其配置如下：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET
```

#### Path断言类型

Path断言类型会通过接受的两个参数来判断是否匹配选择，唯一工厂类：PathRoutePredicateFactory,其配置如下:

```tsx
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Path=/foo/{segment},/bar/{segment}
```

#### Query查询类型

Query类型通过接受两个参数：一个要求的参数和一个可选的regexp表达式，来匹配是否选择，其工厂类为QueryRoutePredicateFactory,配置示例如下:

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=baz，ba.
```

路由必须满足有一个baz的查询参数，且该参数值必须满足ba.表达式，方可通过请求；

#### RemoteAddr类型

该类型通过设置一系列的IPv4或者IPv6地址以及子网掩码形式的字符串，来进行匹配，其唯一工厂类为：RemoteAddrRoutePredicateFactory;
 其配置如下：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

路由请求必须满足192.168.1.1/24指定的ip域内方可通过选择;

#### Weight类型

Weight类型通过设置权重，来进行路由选择，其配置如下：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

上例说明路由百分之80将发向[https://weighthigh.org](https://links.jianshu.com/go?to=https%3A%2F%2Fweighthigh.org),百分20发向[https://weightlow.org](https://links.jianshu.com/go?to=https%3A%2F%2Fweightlow.org)

### Filter过滤器

#### 配置文件

fliter需要在配置文件bootstrap.yml中进行相应设置，或者手动定义一个实现了Filter；
 配置文件设置filter属性如下：

```bash
    gateway:
        discovery:
           locator:
              enabled: true
              predicates:
                  -  Path='/api/**/'+serviceId+'/**'                 
              filters:
              - name: RewritePath
                args: 
                     regexp: "'/api/.*/' + serviceId + '/(?<remaining>.*)'"                 
                     replacement: "'/${remaining}'"
```

通过设置name便可调用相应的过滤器工厂类创建一个过滤器，上例中，前缀RewritePath便会通过RewritePathGatewayFilterFactory创建一个GatewayFilter对象，来进行相应过滤操作；

#### GlobalFilter

gateway网关Filter分为以下Global和Define，其中Gobal有以下几种(按ordered从小到大):

- AdaptCachedBodyGlobalFilter(Integer.Min_VALU):通过适配缓存requestbody参数来实现request的重新构建；
- NettyWriteResponseFilter(-1):将当前请求的响应进行输出到服务调用方；
- ForwardPathFilter(0)：解析路径并将路径转发
- GatewayMetricsFilter(0)：网关性能测量器，可以用来收集网关请求参数，方便创建一个Grafana Dashbord；
- RouteToRequestUrlFilter(10000)：转换路由中的URI
- LoadBalancerClientFilter(10100)：通过负载均衡客户端根据路由的URL解析转换成真实的请求URL
- WebsocketRoutingFilter(Integer.MAX_VALUE-1)：负责处理Websocket类型请求响应信息；
- ForwardRoutingFilter(Integer.MAX_VALUE)：其根据 forward:// 前缀( Scheme )过滤处理，将请求转发到当前网关实例本地接口。
- NettyRoutingFilter(Integer.MAX_VALUE)):通过HttpClient客户端转发真实的URL请求到服务提供方，并将响应写入到当前的请求响应中；
