---
title: SpringCloud组件之Zuul--API网关（六）
date: 2019-11-2 18:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part5_zuul>

### API网关 - 基于Zuul

我们使用Spring Cloud Netflix中的Eureka实现了服务注册中心以及服务注册与发现；而服务间通过Ribbon或Feign实现服务的消费以及均衡负载；通过Spring Cloud Config实现了应用多环境的外部化配置以及版本管理。为了使得服务集群更为健壮，使用Hystrix的融断机制来避免在微服务架构中个别服务出现异常时引起的故障蔓延。

<!--more-->

在该架构中，我们的服务集群包含：内部服务Service A和Service B，他们都会注册与订阅服务至Eureka Server，而Open Service是一个对外的服务，通过均衡负载公开至服务调用方。本文我们把焦点聚集在对外服务这块，这样的实现是否合理，或者是否有更好的实现方式呢？

先来说说这样架构需要做的一些事儿以及存在的不足：

- 首先，破坏了服务无状态特点。为了保证对外服务的安全性，我们需要实现对服务访问的权限控制，而开放服务的权限控制机制将会贯穿并污染整个开放服务的业务逻辑，这会带来的最直接问题是，破坏了服务集群中REST API无状态的特点。从具体开发和测试的角度来说，在工作中除了要考虑实际的业务逻辑之外，还需要额外可续对接口访问的控制处理。
- 其次，无法直接复用既有接口。当我们需要对一个即有的集群内访问接口，实现外部服务访问时，我们不得不通过在原有接口上增加校验逻辑，或增加一个代理调用来实现权限控制，无法直接复用原有的接口。

面对类似上面的问题，我们要如何解决呢？下面进入本文的正题：服务网关！

为了解决上面这些问题，我们需要将权限控制这样的东西从我们的服务单元中抽离出去，而最适合这些逻辑的地方就是处于对外访问最前端的地方，我们需要一个更强大一些的均衡负载器，它就是本文将来介绍的：服务网关。

服务网关是微服务架构中一个不可或缺的部分。通过服务网关统一向外系统提供REST API的过程中，除了具备服务路由、均衡负载功能之外，它还具备了权限控制等功能。Spring Cloud Netflix中的Zuul就担任了这样的一个角色，为微服务架构提供了前门保护的作用，同时将权限控制这些较重的非业务逻辑内容迁移到服务路由层面，使得服务集群主体能够具备更高的可复用性和可测试性。

#### 工作原理

zuul的核心是一系列的filters, 其作用类比Servlet框架的Filter，或者AOP。zuul把请求路由到用户处理逻辑的过程中，这些filter参与一些过滤处理，比如Authentication，Load Shedding等

![11k7yF.png](https://s2.ax1x.com/2020/01/30/11k7yF.png)

Zuul使用一系列不同类型的过滤器，使我们能够快速灵活地将功能应用于我们的边缘服务。这些过滤器可帮助我们执行以下功能

- 身份验证和安全性 - 确定每个资源的身份验证要求并拒绝不满足这些要求的请求
- 洞察和监控 - 在边缘跟踪有意义的数据和统计数据，以便为我们提供准确的生产视图
- 动态路由 - 根据需要动态地将请求路由到不同的后端群集
- 压力测试 - 逐渐增加群集的流量以衡量性能。
- Load Shedding - 为每种类型的请求分配容量并删除超过限制的请求
- 静态响应处理 - 直接在边缘构建一些响应，而不是将它们转发到内部集群

**过滤器的生命周期**

![11kOoR.png](https://s2.ax1x.com/2020/01/30/11kOoR.png)

##### spring-cloud-zuul是如何映射路由的？

zuul的路由映射是使用springMVC功能，我们知道springMVC有两大核心组件：

- HandlerMapping：映射器

- HandlerAdapter：适配器

  具体的springMVC原理这里不做讲解，我们来看下zuul是如何自定义HandlerMapping来注册路由映射的？下图是springMVC的类继承关系

  ![13U4PI.png](https://s2.ax1x.com/2020/01/31/13U4PI.png)

  很清晰看到Zuul提供的ZuulHandlerMapping是AbstractUrlHandlerMapping的子类，这个类是根据url来查找处理器，核心处理方法在lookupHandler里面：

  ```java
  @Override
  protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
      if (this.errorController != null && urlPath.equals(this.errorController.getErrorPath())) {
          return null;
      }
      //过滤忽略的路由规则
      String[] ignored = this.routeLocator.getIgnoredPaths().toArray(new String[0]);
      if (PatternMatchUtils.simpleMatch(ignored, urlPath)) {
          return null;
      }
      RequestContext ctx = RequestContext.getCurrentContext();
      if (ctx.containsKey("forward.to")) {
          return null;
      }
      if (this.dirty) {
          synchronized (this) {
              if (this.dirty) {//如果没有加载过路由或者路由有刷新，则加载路由
                  registerHandlers();
                  this.dirty = false;
              }
          }
      }
      //根据url调用父类获取处理器
      return super.lookupHandler(urlPath, request);
  }
  
  private void registerHandlers() {
      //使用路由定位器获取路由规则
      Collection<Route> routes = this.routeLocator.getRoutes();
      if (routes.isEmpty()) {
          this.logger.warn("No routes found from RouteLocator");
      }
      else {
          for (Route route : routes) {
              //调用父类，注册处理器
              registerHandler(route.getFullPath(), this.zuul);
          }
      }
  }
  ```

梳理一下，以上方法的核心几步：

- 判断urlPath是否被忽略，如果忽略则返回null
- 判断路由规则有没有加载过或者更新过，没有加载或者有更新则重新加载
- 注册处理器的时候，使用的是`ZuulController`，是`Controller`的子类，对应的适配器是`SimpleControllerHandlerAdapter`，也就说每一个路由规则公共处理器都是`ZuulController`，这个处理器最终会调用`ZuulServlet`经过zuul定义的和自定义的拦截器，这个zuul的核心，后面我们作详细讲解。
- 根据url找到处理器，返回

##### 路由定位器

在上面我们注册了路由规则，而路由规则是由路由定位器获取，那么zuul给我们提供哪些路由定位器，类图如下：

![13aCsU.png](https://s2.ax1x.com/2020/01/31/13aCsU.png)

- SimpleRouteLocator：主要加载配置文件的路由规则
- DiscoveryClientRouteLocator：服务发现的路由定位器，去注册中心如Eureka，consul等拿到服务名称，以这样的方式`/服务名称/**`映射成路由规则
- CompositeRouteLocator：复合路由定位器，主要集成所有的路由定位器（如配置文件路由定位器，服务发现定位器，自定义路由定位器等）来路由定位。
- RefreshableRouteLocator：路由刷新，只有实现了此接口的路由定位器才能被刷新

**扩展**：

1. 这里我们可以实现自己的路由定位器，扩展自己想要的功能，如从数据库加载路由规则

2. 利用服务发现的路由定位器去加载理由规则的时候，我们只是简单的是把serviceId映射成路由规则，有的时间我们还是想在serviceId和路由之间提供约定 ，于是我们可以使用PatternServiceRouteMapper来实现

   ```java
   @Bean
   public PatternServiceRouteMapper serviceRouteMapper() {
       return new PatternServiceRouteMapper(
           "(?<name>^.+)-(?<version>v.+$)",
           "${version}/${name}");
   }
   ```

   这样serviceId：`myusers-v1`将被映射到路由`/ v1 / myusers / **`，这里任何正则表达式都可以接受，根据自己需要自己设定。

##### 过滤器

前面提到，所以路由请求都会被控制器`ZuulController`拦截到，最终交由`ZuulServlet`来处理，核心处理代码如下：

```java
@Override
public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse) throws ServletException, IOException {
    try {
        init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);
        // Marks this request as having passed through the "Zuul engine", as opposed to servlets
        // explicitly bound in web.xml, for which requests will not have the same data attached
        RequestContext context = RequestContext.getCurrentContext();
        context.setZuulEngineRan();
        try {
            preRoute();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            route();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            postRoute();
        } catch (ZuulException e) {
            error(e);
            return;
        }
    } catch (Throwable e) {
        error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
    } finally {
        RequestContext.getCurrentContext().unset();
    }
}
```

这段代码体现了zuul过滤器的生命周期，官方提供了一张图很形象的展示：

![11kOoR.png](https://s2.ax1x.com/2020/01/30/11kOoR.png)

uul把过滤器分为四个阶段，分别是

- pre：主要是在请求路由之前调用，很多验证可以在这里做
- route：在路由请求时候被调用，主要用来转发请求
- post：主要用来处理响应请求
- error：当错误发生时，会经由这个类型的过滤器处理

zuul为我们提供了各个阶段的过滤器一共10个

![13alee.png](https://s2.ax1x.com/2020/01/31/13alee.png)

这里我们来着重看下路由阶段的两个过滤器

- SimpleHostRoutingFilter：主要提供当路由设置url方式时，由这个路由器来转发请求，使用的是apache的`CloseableHttpClient`来发送http请求
- RibbonRoutingFilter：当路由设置serviceId时，由此过滤器来转发请求，这里集成了ribbon，Hystrix，实现负载均衡，熔断的功能；默认情况下也是使用apache的`HttpClient`来转发请求

#### zuul组件

- zuul-core--zuul核心库，包含编译和执行过滤器的核心功能。
- zuul-simple-webapp--zuul Web应用程序示例，展示了如何使用zuul-core构建应用程序。
- zuul-netflix--lib包，将其他NetflixOSS组件添加到Zuul中，例如使用功能区进去路由请求处理。
- zuul-netflix-webapp--webapp，它将zuul-core和zuul-netflix封装成一个简易的webapp工程包。

#### 开始使用Zuul

引入依赖spring-cloud-starter-zuul、spring-cloud-starter-eureka，如果不是通过指定serviceId的方式，eureka依赖不需要，但是为了对服务集群细节的透明性，还是用serviceId来避免直接引用url的方式吧。

```xml
        <!--eureke client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>


        <!--zuul-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
```

应用主类使用`@EnableZuulProxy`注解开启Zuul

```java
@EnableZuulProxy
@SpringCloudApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

}
```

*这里用了`@SpringCloudApplication`注解，之前没有提过，通过源码我们看到，它整合了`@SpringBootApplication`、`@EnableDiscoveryClient`、`@EnableCircuitBreaker`，主要目的还是简化配置。这几个注解的具体作用这里就不做详细介绍了，之前的文章已经都介绍过。*

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public @interface SpringCloudApplication {
}

@EnableCircuitBreaker
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import({ZuulProxyMarkerConfiguration.class})
public @interface EnableZuulProxy {
}
```

`application.yml`中配置Zuul应用的基础信息，如：应用名、服务端口等

```yml
server:
  port: 5000

spring:
  application:
    name: gateway-zuul
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true # 优先注册IP地址而不是hostname
  healthcheck:
    enabled: true # 启用健康检查,注意:需要引用spring boot actuator
```

#### Zuul配置

完成上面的工作后，Zuul已经可以运行了，但是如何让它为我们的微服务集群服务，还需要我们另行配置，下面详细的介绍一些常用配置内容。

#### 服务路由

通过服务路由的功能，我们在对外提供服务的时候，只需要通过暴露Zuul中配置的调用地址就可以让调用方统一的来访问我们的服务，而不需要了解具体提供服务的主机信息了。

在Zuul中提供了下面映射方式：

- 通过url直接映射，我们可以如下配置

  ```yml
  # 构建路由地址
  zuul:
    routes:
      # 这里可以自定义
      demo2:
        # 匹配的路由规则
        path: /demo/**
        # 路由的目标地址
        url: http://localhost:8090/
  ```

- 通过服务名

  ```yml
  # 配置Eureka地址
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka/
  # 构建路由地址
  zuul:
    routes:
      # 这里可以自定义
      demo2:
        # 匹配的路由规则
        path: /demo/**
        # 路由的目标服务名
        serviceId: demo
  # 关闭使用eureka负载路由
  #ribbon:
  #  eureka:
  #    enabled: false
  # 如果不使用eureka的话，需要自己定义路由的那个服务的其他负载服务
  #demo:
  #  ribbon:
      # 这里写你要路由的demo服务的所有负载服务请求地址，本项目只启动一个，因此只写一个
  #    listOfServers: http://localhost:8090/
  ```

  所有`/demo/`开头的请求都交给`demo`处理
  
- zuul通过与eureka的整合，实现了对服务实例的自动化维护，所以使用服务路由配置的时候，不需要向传统路由配置方式那样为serviceId指定具体服务实例地址，只需要通过`zuul.routes.<route>.path`与`zuul.routes.<route>.serviceId`参数对的方式进行配置即可

  ```yml
  zuul:
    routes:
      # 这里可以自定义
      demo2:
        # 匹配的路由规则
        path: /demo/**
        # 路由的目标服务名
        serviceId: demo
  ```

  除了path和serviceId键值对的配置方式之外，还有一种简单的配置:`zuul.routes.<serviceId>=<path>`，其中<serviceId\>用来指定路由的具体服务名，<path\>用来配置匹配的请求表达式

  ```yml
  zuul:
    routes:
      demo: /demo/**
  ```

*推荐使用serviceId的映射方式，除了对Zuul维护上更加友好之外，serviceId映射方式还支持了断路器，对于服务故障的情况下，可以有效的防止故障蔓延到服务网关上而影响整个系统的对外服务*

##### 路径匹配

在zuul中，路由匹配的路径表达式采用ant风格定义

| 通配符 | 说明                             |
| ------ | -------------------------------- |
| ？     | 匹配任意单个字符                 |
| *      | 匹配任意数量的字符               |
| **     | 匹配任意数量的字符，支持多级目录 |

##### 忽略表达式

通过path参数定义的ant表达式已经能够完成api网关上的路由规则配置功能，但是为了更细粒度和更为灵活地配置理由规则，zuul还提供了一个忽略表达式参数`zuul.ignored-patterns`。该参数可以用来设置不希望被api网关进行路由的url表达式

```yml
zuul:
  routes:
    demo:
      path: /demo/**
      serviceId: demo
  # 不路由demo2开头的任意请求
  ignored-patterns: /demo2/**
```

##### 路由前缀

为了方便地为路由规则增加前缀信息，zuul提供了`zuul.prefix`参数来进行设置。比如，希望为网关上的路由规则增加/api前缀，那么我们可以在配置文件中增加配置:`zuul.prefix=/api`。另外，对于代理前缀会默认从路径中移除，我们可以通过设置`zuul.strip-prefix=false`(默认为true，默认为true时前缀生效，比如`http://localhost:8080/api/demo/test`）来关闭该移除代理前缀的动作

##### 本地跳转

在zuul实现的api网关路由功能中，还支持forward形式的服务端跳转配置。实现方式非常简单，只需要通过使用path与url的配置方式就能完成，通过url中使用forward来指定需要跳转的服务器资源路径

1. 在zuul服务中添加一个接口

   ```java
   /**
    * @author Gjing
    **/
   @RestController
   public class HelloController {
   
       @GetMapping("/test/hello")
       public String test() {
           return "hello zuul";
       }
   }
   ```

2. 配置文件

   ```java
   zuul:
     routes:
       zuul-service:
         path: /api/**
         serviceId: forward:/test/
   ```

   启动后访问`http://localhost:8080/api/hello`即可

##### cookie与头信息

默认情况下，spring cloud zuul在请求路由时，会过滤掉http请求头信息中一些敏感信息，防止它们被传递到下游的外部服务器。默认的敏感头信息通过zuul.sensitiveHeaders参数定义，默认包括cookie,set-Cookie,authorization三个属性。所以，我们在开发web项目时常用的cookie在spring cloud zuul网关中默认时不传递的，这就会引发一个常见的问题，如果我们要将使用了spring security，shiro等安全框架构建的web应用通过spring cloud zuul构建的网关来进行路由时，由于cookie信息无法传递，我们的web应用将无法实现登录和鉴权。为了解决这个问题，以下介绍两种配置方式

- 通过设置全局参数为空来覆盖默认值

```yaml
zuul:
  routes:
    demo:
      path: /demo/**
      serviceId: demo
  # 允许敏感头，设置为空就行了
  sensitive-headers:
```

- 通过指定路由的参数来设置

```yaml
zuul:
  routes:
    demo:
      path: /demo/**
      serviceId: demo
      # 将指定路由的敏感头设置为空
      sensitiveHeaders:
```

#### 使用Zuul过滤器

为了让api网关组件可以被更方便的使用，它在http请求生命周期的各个阶段默认实现了一批核心过滤器，它们会在api网关服务启动的时候被自动加载和启动。我们可以在源码中查看和了解它们，它们定义与spring-cloud-netflix-core模块的org.springframework.cloud.netflix.zuul.filters包下。在默认启动的过滤器中包含三种不同生命周期的过滤器，这些过滤器都非常重要，可以帮组我们理解zuul对外部请求处理的过程，以及帮助我们在此基础上扩展过滤器去完成自身系统需要的功能

对于Zuul的请求声明周期来说，一共有4种标准过滤器类型：

1. PRE：在请求被路由之前调用，可利用这种过滤器实现身份验证、记录调试信息等操作；

   - **ServletDetectionFilter**

   > ServletDetectionFilter：它的执行顺序为-3，是最先被执行的过滤器。该过滤器总是会被执行，主要用来检测当前请求是通过Spring的DispatcherServlet处理运行的，还是通过ZuulServlet来处理运行的。它的检测结果会以布尔类型保存在当前请求上下文的isDispatcherServletRequest参数中，这样后续的过滤器中，我们就可以通过RequestUtils.isDispatcherServletRequest()和RequestUtils.isZuulServletRequest()方法来判断请求处理的源头，以实现后续不同的处理机制。一般情况下，发送到api网关的外部请求都会被Spring的DispatcherServlet处理，除了通过/zuul/*路径访问的请求会绕过DispatcherServlet（比如之前我们说的大文件上传），被ZuulServlet处理，主要用来应对大文件上传的情况。另外，对于ZuulServlet的访问路径/zuul/*，我们可以通过zuul.servletPath参数进行修改。

   - **Servlet30WrapperFilter**

   > 它的执行顺序为-2，是第二个执行的过滤器，目前的实现会对所有请求生效，主要为了将原始的HttpServletRequest包装成Servlet30RequestWrapper对象。

   - **FormBodyWrapperFilter**

   > 它的执行顺序为-1，是第三个执行的过滤器。该过滤器仅对两类请求生效，第一类是Context-Type为application/x-www-form-urlencoded的请求，第二类是Context-Type为multipart/form-data并且是由String的DispatcherServlet处理的请求（用到了ServletDetectionFilter的处理结果）。而该过滤器的主要目的是将符合要求的请求体包装成FormBodyRequestWrapper对象

   - **DebugFilter**

   > 它的执行顺序为1，是第四个执行的过滤器，该过滤器会根据配置参数zuul.debug.request和请求中的debug参数来决定是否执行过滤器中的操作。而它的具体操作内容是将当前请求上下文中的debugRouting和debugRequest参数设置为true。由于在同一个请求的不同生命周期都可以访问到这二个值，所以我们在后续的各个过滤器中可以利用这二个值来定义一些debug信息，这样当线上环境出现问题的时候，可以通过参数的方式来激活这些debug信息以帮助分析问题，另外，对于请求参数中的debug参数，我们可以通过zuul.debug.parameter来进行自定义

   - **PreDecorationFilter**

   > 执行顺序是5，是pre阶段最后被执行的过滤器，该过滤器会判断当前请求上下文中是否存在forward.do和serviceId参数，如果都不存在，那么它就会执行具体过滤器的操作（如果有一个存在的话，说明当前请求已经被处理过了，因为这二个信息就是根据当前请求的路由信息加载进来的）。而当它的具体操作内容就是为当前请求做一些预处理，比如说，进行路由规则的匹配，在请求上下文中设置该请求的基本信息以及将路由匹配结果等一些设置信息等，这些信息将是后续过滤器进行处理的重要依据，我们可以通过RequestContext.getCurrentContext()来访问这些信息。另外，我们还可以在该实现中找到对HTTP头请求进行处理的逻辑，其中包含了一些耳熟能详的头域，比如X-Forwarded-Host,X-Forwarded-Port。另外，对于这些头域是通过zuul.addProxyHeaders参数进行控制的，而这个参数默认值是true，所以zuul在请求跳转时默认会为请求增加X-Forwarded-*头域，包括X-Forwarded-Host,X-Forwarded-Port，X-Forwarded-For，X-Forwarded-Prefix,X-Forwarded-Proto。也可以通过设置zuul.addProxyHeaders=false关闭对这些头域的添加动作

2. ROUTING：将请求路由到微服务，可利用这种过滤器用于构建发送给微服务的请求；

   - **RibbonRoutingFilter**

   > 它的执行顺序为10，是route阶段的第一个执行的过滤器。该过滤器只对请求上下文中存在serviceId参数的请求进行处理，即只对通过serviceId配置路由规则的请求生效。而该过滤器的执行逻辑就是面向服务路由的核心，它通过使用ribbon和hystrix来向服务实例发起请求，并将服务实例的请求结果返回

   - **SimpleHostRoutingFilter**

   > 它的执行顺序为100，是route阶段的第二个执行的过滤器。该过滤器只对请求上下文存在routeHost参数的请求进行处理，即只对通过url配置路由规则的请求生效。而该过滤器的执行逻辑就是直接向routeHost参数的物理地址发起请求，从源码中我们可以知道该请求是直接通过httpclient包实现的，而没有使用Hystrix命令进行包装，所以这类请求并没有线程隔离和断路器的保护

   - **SendForwardFilter**

   > 它的执行顺序是500，是route阶段第三个执行的过滤器。该过滤器只对请求上下文中存在的forward.do参数进行处理请求，即用来处理路由规则中的forward本地跳转装配

3. POST：在路由到微服务以后执行，可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等；

   - **SendErrorFilter**

   > 它的执行顺序是0，是post阶段的第一个执行的过滤器。该过滤器仅在请求上下文中包含error.status_code参数（由之前执行的过滤器设置的错误编码）并且还没有被该过滤器处理过的时候执行。而该过滤器的具体逻辑就是利用上下文中的错误信息来组成一个forward到api网关/error错误端点的请求来产生错误响应

   - **SendResponseFilter**

   > 它的执行顺序为1000，是post阶段最后执行的过滤器，该过滤器会检查请求上下文中是否包含请求响应相关的头信息，响应数据流或是响应体，只有在包含它们其中一个的时候执行处理逻辑。而该过滤器的处理逻辑就是利用上下文的响应信息来组织需要发送回客户端的响应内容

4. ERROR：在其他阶段发生错误时执行该过滤器；　　

![11EN4I.png](https://s2.ax1x.com/2020/01/30/11EN4I.png)

```java
public class PreRequestLogFilter extends ZuulFilter {
    private static final Logger LOGGER = LoggerFactory.getLogger(PreRequestLogFilter.class);

    // 返回过滤器的类型
    // 可选项：pre, route, post, error
    @Override
    public String filterType() {
        return "pre";
    }

    // 返回一个int值来指定过滤器的执行顺序
    // 不同的过滤器允许返回相同的数字
    @Override
    public int filterOrder() {
        return 1;
    }

    // 返回一个boolean值来判断该过滤器是否要执行
    @Override
    public boolean shouldFilter() {
        return true;
    }

    // 过滤器具体逻辑
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        PreRequestLogFilter.LOGGER.info(String.format("send %s request to %s",
                request.getMethod(),
                request.getRequestURL().toString()));

        return null;
    }
}
```

部分场景下，想要禁用部分过滤器，只需要在配置文件中设置即可，例如这里禁用PreRequestLogFilter过滤器：

```yml
zuul:
  # 禁用指定过滤器设置
  PreRequestLogFilter:
    pre:
      disable: true # 禁用我们创建的PreRequestLogFilter过滤器
```

#### Zuul的容错与回退

Zuul自身就带有Hystrix，但是**它监控的粒度是微服务级别，而不是某个API**，当某个API不可用时，会统一抛500错误码的异常页。我们可以为Zuul添加回退，以实现更友好的返回信息。实现也很简单，只需要实现FallbackProvider接口即可。这里要注意的是，对于Edgware之前的版本（即Dalston及更低版本）需要实现的是ZuulFallbackProvider接口，而Edgware及之后的版本要实现的是FallbackProvider接口。因为FallbackProvider是ZuulFallbackProvider的子接口，而它的好处就是多了一个接口可以获取可能造成回退的原因

```java
@Component
public class UserFallbackProvider implements FallbackProvider {

    @Override
    public String getRoute() {
        // 表明是为哪个微服务提供回退
        return "user-service";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                // fallback时的状态码
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                // 数字类型的状态码
                return getStatusCode().value();
            }

            @Override
            public String getStatusText() throws IOException {
                // 状态文本信息
                return getStatusCode().getReasonPhrase();
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                // 响应体
                return new ByteArrayInputStream("User Service is not avaiable, please try again later."
                        .getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                // headers设定
                HttpHeaders headers = new HttpHeaders();
                MediaType mt = new MediaType("application", "json",
                        Charset.forName("UTF-8"));
                headers.setContentType(mt);

                return headers;
            }
        };
    }
}

@Component
public class GlobalFallback implements FallbackProvider {
    /**
     * 这里配置是为哪个服务提供回退，*号代表所有服务
     */
    @Override
    public String getRoute() {
        return "*";
    }

    /**
     * 回退返回
     */
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            @Override
            @NonNull
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.BAD_REQUEST;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.BAD_REQUEST.value();
            }

            @Override
            @NonNull
            public String getStatusText() throws IOException {
                return HttpStatus.BAD_REQUEST.getReasonPhrase();
            }

            @Override
            public void close() {

            }

            @Override
            @NonNull
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("服务器异常请稍后再试".getBytes(StandardCharsets.UTF_8));
            }

            @Override
            @NonNull
            public HttpHeaders getHeaders() {
                HttpHeaders httpHeaders = new HttpHeaders();
                httpHeaders.setContentType(MediaType.APPLICATION_JSON);
                return httpHeaders;
            }
        };
    }
}
```

**相关方法介绍**

| 方法名           | 说明                                |
| ---------------- | ----------------------------------- |
| getRoute         | 为哪个服务提供回退，*号代表所有服务 |
| fallbackResponse | 回退响应                            |
| getStatusCode    | 回退时的状态码                      |
| getRawStatusCode | 数字类型状态码                      |
| getStatusText    | 状态文本                            |
| close            | 这个不用管                          |
| getBody          | 响应体                              |
| getHeaders       | 返回的响应头                        |

**配置超时时间**

```yml
# 负载均衡超时时间设置
ribbon:
  ReadTimeout: 读超时时间（单位毫秒）
  socketTimeOut: 连接超时时间（单位毫秒）
```

注意！！！：如果zuul配置了熔断fallback的话，熔断超时也要配置，不然如果你配置的ribbon超时时间大于熔断的超时（Hystirx超时默认1秒），那么会先走熔断，相当于你配的ribbon超时就不生效了，ribbon和hystrix是同时生效的，哪个值小哪个生效，另一个就看不到效果了。

```yml
hystrix:
  command:
   # default为默认所有，可以配置指定服务名
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 超时时间（单位毫秒）
```

#### Zuul限流

微服务开发中有时需要对API做限流保护，防止网络攻击，比如做一个短信验证码API，限制客户端的请求速率能在一定程度上抵御短信轰炸攻击，降低损失。微服务网关是每个请求的必经入口，非常适合做一些API限流、认证之类的操作

**限流策略**

| 限流粒度/类型                    | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| Authenticated User               | 使用经过身份验证的用户名或“匿名”                             |
| Request Origin                   | 使用用户原始请求                                             |
| URL                              | 使用下游服务的请求路径                                       |
| ROLE                             | 使用经过身份验证的用户角色                                   |
| Request method                   | 使用HTTP请求方法                                             |
| Global configuration per service | 这个不验证请求Origin，Authenticated User或URI，要使用这个，请不要设置type |

**可用的实现**

| 存储类型 | 说明                                     |
| -------- | ---------------------------------------- |
| consul   | 基于consul                               |
| redis    | 基于redis，使用时必须引入redis相关依赖   |
| JPA      | 基于SpringDataJPA，需要用到数据库        |
| MEMORY   | 基于本地内存，默认                       |
| BUKET4J  | 使用一个Java编写的基于令牌桶算法的限流库 |

Bucket4j实现需要相关的bean @Qualifier("RateLimit"):

- JCache - javax.cache.Cache
- Hazelcast - com.hazelcast.core.IMap
- Ignite - org.apache.ignite.IgniteCache
- Infinispan - org.infinispan.functional.ReadWriteMap

##### 常见的配置属性

| 属性名              | 值                                                           | 默认值                                            |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| enabled             | true/false                                                   | false                                             |
| behind-proxy        | true/false                                                   | false                                             |
| add-response-header | true/false                                                   | false                                             |
| key-prefix          | string                                                       | ${spring.application.name:rate-limit-application} |
| repository          | CONSUL, REDIS, JPA, BUCKET4J_JCACHE, BUCKET4J_HAZELCAST, BUCKET4J_INFINISPAN, BUCKET4J_IGNITE | -                                                 |
| default-policy-list | list-of-policy                                               | -                                                 |
| policy-list         | Map of Lists of Policy                                       | -                                                 |
| postFilterOrder     | int                                                          | FilterConstants.SEND_RESPONSE_FILTER_ORDER - 10   |
| preFilterOrder      | int                                                          | FilterConstants.FORM_BODY_WRAPPER_FILTER_ORDER    |

**policy的相关属性**

| 属性名           | 值                        | 默认值 |
| ---------------- | ------------------------- | ------ |
| limit            | number of calls           | -      |
| quota            | time of calls             | -      |
| refresh-interval | seconds                   | 60     |
| type             | [ORIGIN, USER, URL, ROLE] | []     |

**发生错误如何处理**

```java
  @Bean
  public RateLimiterErrorHandler rateLimitErrorHandler() {
    return new DefaultRateLimiterErrorHandler() {
        @Override
        public void handleSaveError(String key, Exception e) {
            // custom code
        }

        @Override
        public void handleFetchError(String key, Exception e) {
            // custom code
        }

        @Override
        public void handleError(String msg, Exception e) {
            // custom code
        }
    }
  }
```

##### 搭建Zuul结合Ratelimit服务

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
   </dependency>
   <dependency>
       <groupId>com.marcosbarbero.cloud</groupId>
       <artifactId>spring-cloud-zuul-ratelimit</artifactId>
       <version>2.2.3.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. 启动类标注解

   ```java
   @SpringBootApplication
   @EnableEurekaClient
   @EnableZuulProxy
   public class ZuulRatelimitApplication {
       public static void main(String[] args) {
           SpringApplication.run(ZuulRatelimitApplication.class, args);
       }
   }
   ```

3. 配置文件

   ```yml
   server:
     port: 8080
   spring:
     application:
       name: zuul-ratelimit
     redis:
       host: localhost
       password: 
   zuul:
     # 配置路由
     routes:
       demo:
         path: /demo/**
         serviceId: demo
     # 配置限流
     ratelimit:
       enabled: true
       # 对应存储类型(用来统计存储统计信息)
       repository: redis
       # 配置路由的策略
       policy-list:
         demo:
           # 每秒允许多少个请求
           - limit: 2
             # 刷新时间(单位秒)
             refresh-interval: 1
             # 根据什么统计
             type:
               - url
   ```

4. 启动后进行访问

   由于我们配置的是一秒只允许两个请求，当我们超过时，会抛出过多请求异常

   ![13NbNR.png](https://s2.ax1x.com/2020/01/31/13NbNR.png)

   

代码位置：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part5_zuul>

可以测试验证的内容：

- 路由规则：依次启动eureka，user-service，movie-service，zuul-service，然后通过zuul访问user-service接口
- 负载均衡：依次启动eureka，多个user-service，zuul-service，然后多次访问user-service，最后查看日志信息（两个user-service都会打印hibernate日志信息），验证是否达到负载均衡的效果。**PS**：Zuul内置了Ribbon来实现负载均衡。　　
- 路由端点：依次启动eureka，user-service，movie-service，zuul-service，然后浏览器访问zuul-service（http://localhost:5000/routes）可以得到路由端点信息

- 路由配置：示例主要演示了路由前缀、全局敏感设置以及路由规则设置
- 大文件上传设置：针对超大文件上传（比如500M），需要在Zuul中提升超时设置

#### 结合Swagger

网关层swagger配置

1. 加入Swagger的依赖：

   ```xml
   <!-- Swagger -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger-ui</artifactId>
       <version>2.9.2</version>
   </dependency>
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger2</artifactId>
       <version>2.9.2</version>
   </dependency>
   ```

2. swagger配置类

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig {
   
       //是否开启swagger，正式环境一般是需要关闭的，可根据springboot的多环境配置进行设置
       @Value(value = "${swagger.enabled}")
       Boolean swaggerEnabled;
   
       @Bean
       public Docket createRestApi() {
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   // 是否开启
                   .enable(swaggerEnabled).
                           select()
                   // 扫描的路径包
                   .apis(RequestHandlerSelectors.basePackage("com.hyp.learn.zuul"))
                   // 指定路径处理PathSelectors.any()代表所有的路径
                   .paths(PathSelectors.any()).build().pathMapping("/");
       }
   
       private ApiInfo apiInfo() {
           return new ApiInfoBuilder()
                   .title("XXXXX系统")
                   .description("XXXX系统接口文档说明")
                   .termsOfServiceUrl("http://localhost:5000")
                   .version("1.0")
                   .build();
       }
   
   }
   ```

3. swagger文档资源配置类`DocumentationConfig`

   ```java
   @Component
   @Primary
   public class DocumentationConfig implements SwaggerResourcesProvider {
   
       private final RouteLocator routeLocator;
   
       public DocumentationConfig(RouteLocator routeLocator) {
           this.routeLocator = routeLocator;
       }
   
       @Override
       public List<SwaggerResource> get() {
           //利用routeLocator动态引入微服务
           List<SwaggerResource> resources = new ArrayList<>();
           resources.add(swaggerResource("zuul-gateway", "/v2/api-docs", "1.0"));
           //循环 使用Lambda表达式简化代码
           routeLocator.getRoutes().forEach(route -> {
               //动态获取
               resources.add(swaggerResource(route.getId(), route.getFullPath().replace("**", "v2/api-docs"), "1.0"));
           });
           //也可以直接 继承 Consumer接口
   //		routeLocator.getRoutes().forEach(new Consumer<Route>() {
   //
   //			@Override
   //			public void accept(Route t) {
   //				// TODO Auto-generated method stub
   //
   //			}
   //		});
           return resources;
       }
   
       private SwaggerResource swaggerResource(String name, String location, String version) {
           SwaggerResource swaggerResource = new SwaggerResource();
           swaggerResource.setName(name);
           swaggerResource.setLocation(location);
           swaggerResource.setSwaggerVersion(version);
           return swaggerResource;
       }
   }
   ```

4. 服务的swagger配置

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig{
   
       @Bean
       public Docket createRestApi() {
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
                   .apis(RequestHandlerSelectors.basePackage("com.hyp.learn.ms"))
                   .paths(PathSelectors.any())
                   .build();
       }
   
       private ApiInfo apiInfo() {
           return new ApiInfoBuilder()
                   .title("基本信息API")
                   .description("基本信息接口文档说明")
                   .version("1.0")
                   .build();
       }
   }
   ```
   
5. 访问 （网关地址/swagger-ui.html） 本例为 http://localhost:5000/swagger-ui.html。这样就可以在下拉列表中随时切换，查看分布式系统的接口文档了。



### 参考

1. https://gitee.com/didispace/swagger-butler
2. [白话SpringCloud | 第十一章：路由网关(Zuul)：利用swagger2聚合API文档](https://my.oschina.net/xiedeshou/blog/2249851)