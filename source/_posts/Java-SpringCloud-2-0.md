---
title: Spring Cloud 组件之Ribbon--客户端负载均衡 (三)
date: 2019-10-31 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part2_client-load-balance>

### 客户端负载均衡 - 基于Ribbon

Ribbon是一个基于HTTP和TCP客户端的负载均衡器。Feign中也使用Ribbon，后续会介绍Feign的使用。

Ribbon可以在通过客户端中配置的ribbonServerList服务端列表去轮询访问以达到均衡负载的作用。

<!--more-->

负载均衡在系统架构中是一个非常重要，并且是不得不去实施的内容。因为负载均衡是对系统的高可用、网络压力的缓解和处理能力扩容的重要手段之一。我们通常所说的负载均衡都指的是服务端负载均衡，其中分为硬件负载均衡和软件负载均衡。硬件负载均衡主要通过在服务器节点之间按照专门用于负载均衡的设备，比如F5等；而软件负载均衡则是通过在服务器上安装一些用于负载均衡功能或模块等软件来完成请求分发工作，比如Nginx等。不论采用硬件负载均衡还是软件负载均衡，只要是服务端都能以类似下图的架构方式构建起来：

![1JPmIs.png](https://s2.ax1x.com/2020/02/01/1JPmIs.png)

硬件负载均衡的设备或是软件负载均衡的软件模块都会维护一个下挂可用的服务端清单，通过心跳检测来剔除故障的服务端节点以保证清单中都是可以正常访问的服务端节点。当客户端发送请求到负载均衡设备的时候，该设备按某种算法（比如线性轮询、按权重负载、按流量负载等）从维护的可用服务端清单中取出一台服务端端地址，然后进行转发。

而客户端负载均衡和服务端负载均衡最大的不同点在于上面所提到服务清单所存储的位置。在客户端负载均衡中，所有客户端节点都维护着自己要访问的服务端清单，而这些服务端端清单来自于服务注册中心，比如上一章我们介绍的Eureka服务端。同服务端负载均衡的架构类似，在客户端负载均衡中也需要心跳去维护服务端清单的健康性，默认会创建针对各个服务治理框架的Ribbon自动化整合配置，比如Eureka中的org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration，Consul中的org.springframework.cloud.consul.discovery.RibbonConsulAutoConfiguration。在实际使用的时候，我们可以通过查看这两个类的实现，以找到它们的配置详情来帮助我们更好地使用它。

通过Spring Cloud Ribbon的封装，我们在微服务架构中使用客户端负载均衡调用非常简单，只需要如下两步：

- 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心。

- 服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用。

这样，我们就可以将服务提供者的高可用以及服务消费者的负载均衡调用一起实现了。

#### 示例

当Ribbon与Eureka联合使用时，ribbonServerList会被DiscoveryEnabledNIWSServerList重写，扩展成从Eureka注册中心中获取服务端列表。同时它也会用NIWSDiscoveryPing来取代IPing，它将职责委托给Eureka来确定服务端是否已经启动。

##### 准备工作

- 启动服务注册中心：eureka-server-sn
- 启动服务提供方：user-service
- 修改user-service中的server-port为8001，再启动一个服务提供方：user-service

再次访问：<http://localhost:8761/>

![1uhBND.png](https://s2.ax1x.com/2020/01/27/1uhBND.png)

可以看到COMPUTE-SERVICE服务有两个单元正在运行：

- localhost:user-service:8001 
- localhost:user-service:8000

##### 使用Ribbon实现客户端负载均衡的消费者

构建一个基本Spring Boot项目，并在pom.xml中加入如下内容：

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
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--Ribbon-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>

        <!-- eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
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

        <!-- 热启动，热部署依赖包，为了调试方便，加入此包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
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

在应用主类中，通过`@EnableDiscoveryClient`注解来添加发现服务能力。创建RestTemplate实例，并通过`@LoadBalanced`注解开启均衡负载能力。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MovieServiceSnApplication {

    public static void main(String[] args) {
        SpringApplication.run(MovieServiceSnApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        // Do any additional configuration here
        return builder.build();
    }

}
```

把user-service中的User类copy过来，并删除注解

```java
public class User {

    private Long id;

    private String username;

    private String realname;

    private Integer age;

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

创建MovieController进行服务消费：

```java
@RestController
public class MovieController {

    @Autowired
    private RestTemplate restTemplate;

    //缺点是需要客户端知道对应ip地址,应该使用服务名
    @Value("${userservice.url}")
    private String userServiceUrl;

    @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
    public User findById(@PathVariable Long id) {
        return restTemplate.getForObject(userServiceUrl+"/" + id, User.class);
    }

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/user-instance")
    public List<ServiceInstance> showInfo() {
        return discoveryClient.getInstances("user-service");
    }
}
```

配置文件：

```yml
server:
  port: 8010

eureka:
  client:
    serviceUrl:
      #defaultZone: http://peer1:8761/eureka/,http://peer2:8762/eureka/
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true # 优先注册IP地址而不是hostname
  healthcheck:
    enabled: true # 启用健康检查,注意:需要引用spring boot actuator

spring:
  application:
    name: movie-service

userservice:
  url: http://user-service/
```

启动该应用，并访问两次：<http://127.0.0.1:8010/user/1>

到这里，我们已经通过Ribbon在客户端已经实现了对服务调用的均衡负载。

完整示例可参考:<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part2_client-load-balance>

#### Ribbon源码分析

相信很多熟悉Sprint的读者看到这里一定会产生这样的疑问：RestTemplate不是Spring自己提供的嘛？跟Ribbon的客户端负载均衡又有什么关系呢？在本节中，我们透过现象看本质，探索一下Ribbon是如何通过RestTemplate实现客户端负载均衡的

先，回顾一下之前的消费者示例：我们是如何实现客户端负载均衡的？仔细观察一下之前的实现代码，可以发现载消费者的例子中，可能就@LoadBalanced这个注解是之前没有接触过的，并且从命名上来看也与负载均衡相关。我们不妨以此为线索来看看Spring Cloud Ribbon的源码实现。

从@LoadBalanced注解码的注释中，可以知道该注解用来给RestTemplate标记，以使用负载均衡的客户端（LoadBalancerClient）来配置它。

##### LoadBalancerClient

**org.springframework.cloud.client.loadbalancer.LoadBalancerClient**

```java
/**
 * Represents a client-side load balancer.
 *
 * @author Spencer Gibb
 */
public interface LoadBalancerClient extends ServiceInstanceChooser {

	/**
	 * Executes request using a ServiceInstance from the LoadBalancer for the specified
	 * service.
	 * @param serviceId The service ID to look up the LoadBalancer.
	 * @param request Allows implementations to execute pre and post actions, such as
	 * incrementing metrics.
	 * @param <T> type of the response
	 * @throws IOException in case of IO issues.
	 * @return The result of the LoadBalancerRequest callback on the selected
	 * ServiceInstance.
	 */
	<T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException;

	/**
	 * Executes request using a ServiceInstance from the LoadBalancer for the specified
	 * service.
	 * @param serviceId The service ID to look up the LoadBalancer.
	 * @param serviceInstance The service to execute the request to.
	 * @param request Allows implementations to execute pre and post actions, such as
	 * incrementing metrics.
	 * @param <T> type of the response
	 * @throws IOException in case of IO issues.
	 * @return The result of the LoadBalancerRequest callback on the selected
	 * ServiceInstance.
	 */
	<T> T execute(String serviceId, ServiceInstance serviceInstance,
			LoadBalancerRequest<T> request) throws IOException;

	/**
	 * Creates a proper URI with a real host and port for systems to utilize. Some systems
	 * use a URI with the logical service name as the host, such as
	 * http://myservice/path/to/service. This will replace the service name with the
	 * host:port from the ServiceInstance.
	 * @param instance service instance to reconstruct the URI
	 * @param original A URI with the host as a logical service name.
	 * @return A reconstructed URI.
	 */
	URI reconstructURI(ServiceInstance instance, URI original);

}
```

**英文释义：**

- 为了给一些系统使用，创建一个带有真实host和port的URI。
- 一些系统使用带有原服务名代替host的URI，比如http://myservice/path/to/service。
- 该方法会从服务实例中取出host:port来替换这个服务名。

补充：父接口ServiceInstanceChooser

```java
/**
 * Implemented by classes which use a load balancer to choose a server to send a request
 * to.
 *
 * @author Ryan Baxter
 */
public interface ServiceInstanceChooser {

	/**
	 * Chooses a ServiceInstance from the LoadBalancer for the specified service.
	 * @param serviceId The service ID to look up the LoadBalancer.
	 * @return A ServiceInstance that matches the serviceId.
	 */
	ServiceInstance choose(String serviceId);

}

```

从该接口中，我们可以通过定义的抽象方法来了解到客户端负载均衡器中应具备的几种能力：

▪️**ServiceInstance choose(String serviceId)**：父接口ServiceInstanceChooser的方法，根据传入的服务名serviceId，从负载均衡器中挑选一个对应服务的实例。

▪️ **T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest** **request)**：使用从负载均衡器中挑选出的服务实例来执行请求内容。

▪️**URI reconstructURI(ServiceInstance instance, URI original)**：为系统构建一个合适的host:port形式的URI。在分布式系统中，我们使用逻辑上的服务名称作为host来构建URI（替代服务实例的host:port形式）进行请求，比如http://myservice/path/to/service。在该操作的定义中，前者ServiceInstance对象是带有host和port的具体服务实例，而后者URI对象则是使用逻辑服务名定义为host的URI，而返回的URI内容则是通过ServiceInstance的服务实例详情拼接host:port形式的请求地址。

顺着LoadBalancerClient接口的所属包org.springframework.cloud.client.loadbalancer，我们对其内容进行整理，可以得出如下图的关系：

![1JP4Qf.png](https://s2.ax1x.com/2020/02/01/1JP4Qf.png)

从类的命名上可初步判断LoadBalancerAutoConfiguration为实现客户端负载均衡器的自动化配置类。通过查看源码我们可以验证这一点假设：

**org.springframework.cloud.client.loadbalancer.LoadBalancerAutoConfiguration**

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
@EnableConfigurationProperties(LoadBalancerRetryProperties.class)
public class LoadBalancerAutoConfiguration {

	@LoadBalanced
	@Autowired(required = false)
	private List<RestTemplate> restTemplates = Collections.emptyList();

	@Autowired(required = false)
	private List<LoadBalancerRequestTransformer> transformers = Collections.emptyList();

	@Bean
	public SmartInitializingSingleton loadBalancedRestTemplateInitializerDeprecated(
			final ObjectProvider<List<RestTemplateCustomizer>> restTemplateCustomizers) {
		return () -> restTemplateCustomizers.ifAvailable(customizers -> {
			for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
				for (RestTemplateCustomizer customizer : customizers) {
					customizer.customize(restTemplate);
				}
			}
		});
	}

	@Bean
	@ConditionalOnMissingBean
	public LoadBalancerRequestFactory loadBalancerRequestFactory(
			LoadBalancerClient loadBalancerClient) {
		return new LoadBalancerRequestFactory(loadBalancerClient, this.transformers);
	}

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
	static class LoadBalancerInterceptorConfig {

		@Bean
		public LoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient,
				LoadBalancerRequestFactory requestFactory) {
			return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
		}

		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final LoadBalancerInterceptor loadBalancerInterceptor) {
			return restTemplate -> {
				List<ClientHttpRequestInterceptor> list = new ArrayList<>(
						restTemplate.getInterceptors());
				list.add(loadBalancerInterceptor);
				restTemplate.setInterceptors(list);
			};
		}

	}

	/**
	 * Auto configuration for retry mechanism.
	 */
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryAutoConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public LoadBalancedRetryFactory loadBalancedRetryFactory() {
			return new LoadBalancedRetryFactory() {
			};
		}

	}

	/**
	 * Auto configuration for retry intercepting mechanism.
	 */
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryInterceptorAutoConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public RetryLoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient,
				LoadBalancerRetryProperties properties,
				LoadBalancerRequestFactory requestFactory,
				LoadBalancedRetryFactory loadBalancedRetryFactory) {
			return new RetryLoadBalancerInterceptor(loadBalancerClient, properties,
					requestFactory, loadBalancedRetryFactory);
		}

		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final RetryLoadBalancerInterceptor loadBalancerInterceptor) {
			return restTemplate -> {
				List<ClientHttpRequestInterceptor> list = new ArrayList<>(
						restTemplate.getInterceptors());
				list.add(loadBalancerInterceptor);
				restTemplate.setInterceptors(list);
			};
		}

	}

}
```

从LoadBalancerAutoConfiguration类上的注解可知，Ribbon实现负载均衡自动化配置需要满足下面两个条件：

▪️**@ConditionalOnClass(RestTemplate.class)**：RestTemplate必须存在于当前工程的环境中。

▪️**@ConditionalOnBean(LoadBalancerClient.class)**：在Spring的Bean工程中必须有LoadBalancerClient的实现bean。

该自动配置类，主要做了下面三件事：

- 创建了一个LoadBalancerInterceptor的Bean，用于实现对客户端发起请求时进行拦截，以实现客户端负载均衡。
- 创建了一个RestTemplateCustomizer的Bean，用于给RestTemplate增加LoadbalancerInterceptor
- 维护了一个被@LoadBalanced注解修饰的RestTemplate对象列表，并在这里进行初始化，通过调用RestTemplateCustomizer的实例来给需要客户端负载均衡的RestTemplate增加LoadBalancerInterceptor拦截器。

**org.springframework.cloud.client.loadbalancer.LoadBalancerRetryProperties**

```java
/**
 * Configuration properties for the {@link LoadBalancerClient}.
 *
 * @author Ryan Baxter
 */
@ConfigurationProperties("spring.cloud.loadbalancer.retry")
public class LoadBalancerRetryProperties {

	private boolean enabled = true;

	/**
	 * Returns true if the load balancer should retry failed requests.
	 * @return True if the load balancer should retry failed requests; false otherwise.
	 */
	public boolean isEnabled() {
		return this.enabled;
	}

	/**
	 * Sets whether the load balancer should retry failed requests.
	 * @param enabled Whether the load balancer should retry failed requests.
	 */
	public void setEnabled(boolean enabled) {
		this.enabled = enabled;
	}

}
```

接下来，我们看看LoadBalancerInterceptor拦截器是如何将一个普通的RestTemplate

**org.springframework.cloud.client.loadbalancer.LoadBalancerInterceptor**

```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {

	private LoadBalancerClient loadBalancer;

	private LoadBalancerRequestFactory requestFactory;

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer,
			LoadBalancerRequestFactory requestFactory) {
		this.loadBalancer = loadBalancer;
		this.requestFactory = requestFactory;
	}

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
		// for backwards compatibility
		this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
	}

	@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null,
				"Request URI does not contain a valid hostname: " + originalUri);
		return this.loadBalancer.execute(serviceName,
				this.requestFactory.createRequest(request, body, execution));
	}

}

```

通过源码以及之前的自动化配置类，我们可以看到在拦截器中注入了LoadBalancerInterceptor的实现。当一个被@LoadBalanced注解修饰的RestTemplate对象向外发起HTTP请求时，会被LoadBalancerInterceptor类的intercept函数所拦截。由于我们在使用RestTemplate时候采用了服务名作为host，所以直接从HttpRequest的URI对象中通过getHost()就可以拿到服务名，然后调用execute函数去根据服务名来选择实例并发起实际的请求。

分析到这里，LoadBalancerClient还只是一个抽象的负载均衡器接口，所以我们还需要找到它的具体实现类进一步进行分析，通过查看Ribbon的源码，可以很容易地在org.springframework.cloud.netflix.ribbon包下找到对应的实现类RibbonLoadBalancerClient。

**org.springframework.cloud.netflix.ribbon.RibbonLoadBalancerClient**

```java
public class RibbonLoadBalancerClient implements LoadBalancerClient {

	private SpringClientFactory clientFactory;

	public RibbonLoadBalancerClient(SpringClientFactory clientFactory) {
		this.clientFactory = clientFactory;
	}

	@Override
	public URI reconstructURI(ServiceInstance instance, URI original) {
		Assert.notNull(instance, "instance can not be null");
		String serviceId = instance.getServiceId();
		RibbonLoadBalancerContext context = this.clientFactory
				.getLoadBalancerContext(serviceId);

		URI uri;
		Server server;
		if (instance instanceof RibbonServer) {
			RibbonServer ribbonServer = (RibbonServer) instance;
			server = ribbonServer.getServer();
			uri = updateToSecureConnectionIfNeeded(original, ribbonServer);
		}
		else {
			server = new Server(instance.getScheme(), instance.getHost(),
					instance.getPort());
			IClientConfig clientConfig = clientFactory.getClientConfig(serviceId);
			ServerIntrospector serverIntrospector = serverIntrospector(serviceId);
			uri = updateToSecureConnectionIfNeeded(original, clientConfig,
					serverIntrospector, server);
		}
		return context.reconstructURIWithServer(server, uri);
	}

	@Override
	public ServiceInstance choose(String serviceId) {
		return choose(serviceId, null);
	}

	/**
	 * New: Select a server using a 'key'.
	 * @param serviceId of the service to choose an instance for
	 * @param hint to specify the service instance
	 * @return the selected {@link ServiceInstance}
	 */
	public ServiceInstance choose(String serviceId, Object hint) {
		Server server = getServer(getLoadBalancer(serviceId), hint);
		if (server == null) {
			return null;
		}
		return new RibbonServer(serviceId, server, isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));
	}

	@Override
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request)
			throws IOException {
		return execute(serviceId, request, null);
	}

	/**
	 * New: Execute a request by selecting server using a 'key'. The hint will have to be
	 * the last parameter to not mess with the `execute(serviceId, ServiceInstance,
	 * request)` method. This somewhat breaks the fluent coding style when using a lambda
	 * to define the LoadBalancerRequest.
	 * @param <T> returned request execution result type
	 * @param serviceId id of the service to execute the request to
	 * @param request to be executed
	 * @param hint used to choose appropriate {@link Server} instance
	 * @return request execution result
	 * @throws IOException executing the request may result in an {@link IOException}
	 */
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint)
			throws IOException {
		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
		Server server = getServer(loadBalancer, hint);
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}
		RibbonServer ribbonServer = new RibbonServer(serviceId, server,
				isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));

		return execute(serviceId, ribbonServer, request);
	}

	@Override
	public <T> T execute(String serviceId, ServiceInstance serviceInstance,
			LoadBalancerRequest<T> request) throws IOException {
		Server server = null;
		if (serviceInstance instanceof RibbonServer) {
			server = ((RibbonServer) serviceInstance).getServer();
		}
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}

		RibbonLoadBalancerContext context = this.clientFactory
				.getLoadBalancerContext(serviceId);
		RibbonStatsRecorder statsRecorder = new RibbonStatsRecorder(context, server);

		try {
			T returnVal = request.apply(serviceInstance);
			statsRecorder.recordStats(returnVal);
			return returnVal;
		}
		// catch IOException and rethrow so RestTemplate behaves correctly
		catch (IOException ex) {
			statsRecorder.recordStats(ex);
			throw ex;
		}
		catch (Exception ex) {
			statsRecorder.recordStats(ex);
			ReflectionUtils.rethrowRuntimeException(ex);
		}
		return null;
	}

	private ServerIntrospector serverIntrospector(String serviceId) {
		ServerIntrospector serverIntrospector = this.clientFactory.getInstance(serviceId,
				ServerIntrospector.class);
		if (serverIntrospector == null) {
			serverIntrospector = new DefaultServerIntrospector();
		}
		return serverIntrospector;
	}

	private boolean isSecure(Server server, String serviceId) {
		IClientConfig config = this.clientFactory.getClientConfig(serviceId);
		ServerIntrospector serverIntrospector = serverIntrospector(serviceId);
		return RibbonUtils.isSecure(config, serverIntrospector, server);
	}

	// Note: This method could be removed?
	protected Server getServer(String serviceId) {
		return getServer(getLoadBalancer(serviceId), null);
	}

	protected Server getServer(ILoadBalancer loadBalancer) {
		return getServer(loadBalancer, null);
	}

	protected Server getServer(ILoadBalancer loadBalancer, Object hint) {
		if (loadBalancer == null) {
			return null;
		}
		// Use 'default' on a null hint, or just pass it on?
		return loadBalancer.chooseServer(hint != null ? hint : "default");
	}

	protected ILoadBalancer getLoadBalancer(String serviceId) {
		return this.clientFactory.getLoadBalancer(serviceId);
	}

	/**
	 * Ribbon-server-specific {@link ServiceInstance} implementation.
	 */
	public static class RibbonServer implements ServiceInstance {

		private final String serviceId;

		private final Server server;

		private final boolean secure;

		private Map<String, String> metadata;

		public RibbonServer(String serviceId, Server server) {
			this(serviceId, server, false, Collections.emptyMap());
		}

		public RibbonServer(String serviceId, Server server, boolean secure,
				Map<String, String> metadata) {
			this.serviceId = serviceId;
			this.server = server;
			this.secure = secure;
			this.metadata = metadata;
		}

		@Override
		public String getInstanceId() {
			return this.server.getId();
		}

		@Override
		public String getServiceId() {
			return this.serviceId;
		}

		@Override
		public String getHost() {
			return this.server.getHost();
		}

		@Override
		public int getPort() {
			return this.server.getPort();
		}

		@Override
		public boolean isSecure() {
			return this.secure;
		}

		@Override
		public URI getUri() {
			return DefaultServiceInstance.getUri(this);
		}

		@Override
		public Map<String, String> getMetadata() {
			return this.metadata;
		}

		public Server getServer() {
			return this.server;
		}

		@Override
		public String getScheme() {
			return this.server.getScheme();
		}

		@Override
		public String toString() {
			final StringBuilder sb = new StringBuilder("RibbonServer{");
			sb.append("serviceId='").append(serviceId).append('\'');
			sb.append(", server=").append(server);
			sb.append(", secure=").append(secure);
			sb.append(", metadata=").append(metadata);
			sb.append('}');
			return sb.toString();
		}

	}

}
```

在execute函数的实现中，第一步做的就是通过getServer根据传入的服务名serviceId去获得具体的服务实例：

通过getServer函数的实现源码，我们可以看到这里获取具体服务实例的时候并没有使用LoadBalancerClient接口中的choose函数，而是使用了ribbon自身的ILoadBalancer接口中定义的chooseServer函数。

可以看到，在该接口中定义了一个客户端负载均衡器需要的一系列抽象动作：

1. addServers:向负载均衡器中维护的实例列表增加服务实例。

2. chooseServer:通过某种策略，从负载均衡器中挑选出一个具体实例已经停止服务，不然负载均衡器在下一次获取服务实例清单前都会认为服务实例均是正常服务。

3. markServerDwon:用来通知和标识负载均衡器中某个实例已经停止服务，不然负载均衡器在下一次获取服务实例清单前都会认为服务实例均是正常服务的。

4. getReachableServers:获取当前正常服务的实例列表。

5. getAllServers:获取所有已知的服务实例列表，包括正常服务和停止服务的实例。


在该接口定义中涉及到的server对象定义的是一个传统的服务端节点，在该类中存储了服务端节点的一些元数据信息，包括：host，port以及一些部署信息等。

![1JiycV.png](https://s2.ax1x.com/2020/02/01/1JiycV.png)

而对于该接口的实现，我们可以整理出如上图所示的结构。我们可以看到BaseloadBalancer类实现了基础的负载均衡，而DynamicServerListLoadBalancer和ZoneAwareLoadBalancer在负载均衡的策略上做了一些功能的扩展。

##### 那么在整合Ribbon的时候Spring Cloud默认采用了那个具体实现呢？

**org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration**

```java
@SuppressWarnings("deprecation")
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties
// Order is important here, last should be the default, first should be optional
// see
// https://github.com/spring-cloud/spring-cloud-netflix/issues/2086#issuecomment-316281653
@Import({ HttpClientConfiguration.class, OkHttpRibbonConfiguration.class,
		RestClientRibbonConfiguration.class, HttpClientRibbonConfiguration.class })
public class RibbonClientConfiguration {

	/**
	 * Ribbon client default connect timeout.
	 */
	public static final int DEFAULT_CONNECT_TIMEOUT = 1000;

	/**
	 * Ribbon client default read timeout.
	 */
	public static final int DEFAULT_READ_TIMEOUT = 1000;

	/**
	 * Ribbon client default Gzip Payload flag.
	 */
	public static final boolean DEFAULT_GZIP_PAYLOAD = true;

	@RibbonClientName
	private String name = "client";

	// TODO: maybe re-instate autowired load balancers: identified by name they could be
	// associated with ribbon clients

	@Autowired
	private PropertiesFactory propertiesFactory;

	@Bean
	@ConditionalOnMissingBean
	public IClientConfig ribbonClientConfig() {
		DefaultClientConfigImpl config = new DefaultClientConfigImpl();
		config.loadProperties(this.name);
		config.set(CommonClientConfigKey.ConnectTimeout, DEFAULT_CONNECT_TIMEOUT);
		config.set(CommonClientConfigKey.ReadTimeout, DEFAULT_READ_TIMEOUT);
		config.set(CommonClientConfigKey.GZipPayload, DEFAULT_GZIP_PAYLOAD);
		return config;
	}

	@Bean
	@ConditionalOnMissingBean
	public IRule ribbonRule(IClientConfig config) {
		if (this.propertiesFactory.isSet(IRule.class, name)) {
			return this.propertiesFactory.get(IRule.class, config, name);
		}
		ZoneAvoidanceRule rule = new ZoneAvoidanceRule();
		rule.initWithNiwsConfig(config);
		return rule;
	}

	@Bean
	@ConditionalOnMissingBean
	public IPing ribbonPing(IClientConfig config) {
		if (this.propertiesFactory.isSet(IPing.class, name)) {
			return this.propertiesFactory.get(IPing.class, config, name);
		}
		return new DummyPing();
	}

	@Bean
	@ConditionalOnMissingBean
	@SuppressWarnings("unchecked")
	public ServerList<Server> ribbonServerList(IClientConfig config) {
		if (this.propertiesFactory.isSet(ServerList.class, name)) {
			return this.propertiesFactory.get(ServerList.class, config, name);
		}
		ConfigurationBasedServerList serverList = new ConfigurationBasedServerList();
		serverList.initWithNiwsConfig(config);
		return serverList;
	}

	@Bean
	@ConditionalOnMissingBean
	public ServerListUpdater ribbonServerListUpdater(IClientConfig config) {
		return new PollingServerListUpdater(config);
	}

	@Bean
	@ConditionalOnMissingBean
	public ILoadBalancer ribbonLoadBalancer(IClientConfig config,
			ServerList<Server> serverList, ServerListFilter<Server> serverListFilter,
			IRule rule, IPing ping, ServerListUpdater serverListUpdater) {
		if (this.propertiesFactory.isSet(ILoadBalancer.class, name)) {
			return this.propertiesFactory.get(ILoadBalancer.class, config, name);
		}
		return new ZoneAwareLoadBalancer<>(config, rule, ping, serverList,
				serverListFilter, serverListUpdater);
	}

	@Bean
	@ConditionalOnMissingBean
	@SuppressWarnings("unchecked")
	public ServerListFilter<Server> ribbonServerListFilter(IClientConfig config) {
		if (this.propertiesFactory.isSet(ServerListFilter.class, name)) {
			return this.propertiesFactory.get(ServerListFilter.class, config, name);
		}
		ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
		filter.initWithNiwsConfig(config);
		return filter;
	}

	@Bean
	@ConditionalOnMissingBean
	public RibbonLoadBalancerContext ribbonLoadBalancerContext(ILoadBalancer loadBalancer,
			IClientConfig config, RetryHandler retryHandler) {
		return new RibbonLoadBalancerContext(loadBalancer, config, retryHandler);
	}

	@Bean
	@ConditionalOnMissingBean
	public RetryHandler retryHandler(IClientConfig config) {
		return new DefaultLoadBalancerRetryHandler(config);
	}

	@Bean
	@ConditionalOnMissingBean
	public ServerIntrospector serverIntrospector() {
		return new DefaultServerIntrospector();
	}

	@PostConstruct
	public void preprocess() {
		setRibbonProperty(name, DeploymentContextBasedVipAddresses.key(), name);
	}

	static class OverrideRestClient extends RestClient {

		private IClientConfig config;

		private ServerIntrospector serverIntrospector;

		protected OverrideRestClient(IClientConfig config,
				ServerIntrospector serverIntrospector) {
			super();
			this.config = config;
			this.serverIntrospector = serverIntrospector;
			initWithNiwsConfig(this.config);
		}

		@Override
		public URI reconstructURIWithServer(Server server, URI original) {
			URI uri = updateToSecureConnectionIfNeeded(original, this.config,
					this.serverIntrospector, server);
			return super.reconstructURIWithServer(server, uri);
		}

		@Override
		protected Client apacheHttpClientSpecificInitialization() {
			ApacheHttpClient4 apache = (ApacheHttpClient4) super.apacheHttpClientSpecificInitialization();
			apache.getClientHandler().getHttpClient().getParams().setParameter(
					ClientPNames.COOKIE_POLICY, CookiePolicy.IGNORE_COOKIES);
			return apache;
		}

	}

}
```

从RibbonClientConfiguration配置类，可以知道在整合时默认采用ZoneAvoidanceRule来实现负载均衡器。

下面，我们再回到RibbonLoadBalancer的execute函数逻辑，在通过ZoneAwareLoaderBalancer的chooseServer函数获取了负载均衡策略分配到的服务实例对象server之后，将其内容包装成RibbonServer对象（该对象除了存储了服务实例的信息之外，还增加了服务名serviceId、是否需要使用HTTPS等其他信息），然后使用该对象再回调LoadBalancerInterceptor请求拦截器中LoadBalancerRequest的apply(final ServiceInstance	 instance)函数，向一个实际的具体服务实例发起请求，从而实现一开始以服务名为host的URI请求，到实际的host:post形式的具体地址的转换。

```java
public interface LoadBalancerRequest<T> {

	T apply(ServiceInstance instance) throws Exception;

}
```

 `apply(ServiceInstance	 instance)`函数中传入的ServiceIntance接口是对服务实例的抽象定义。在该接口中暴露服务治理系统中每个服务实例需要提供的一些基本信息，比如：serviceId、host、port等，具体定义如下：

**org.springframework.cloud.client.ServiceInstance**

```java
/**
 * Represents an instance of a service in a discovery system.
 *
 * @author Spencer Gibb
 * @author Tim Ysewyn
 */
public interface ServiceInstance {

	/**
	 * @return The unique instance ID as registered.
	 */
	default String getInstanceId() {
		return null;
	}

	/**
	 * @return The service ID as registered.
	 */
	String getServiceId();

	/**
	 * @return The hostname of the registered service instance.
	 */
	String getHost();

	/**
	 * @return The port of the registered service instance.
	 */
	int getPort();

	/**
	 * @return Whether the port of the registered service instance uses HTTPS.
	 */
	boolean isSecure();

	/**
	 * @return The service URI address.
	 */
	URI getUri();

	/**
	 * @return The key / value pair metadata associated with the service instance.
	 */
	Map<String, String> getMetadata();

	/**
	 * @return The scheme of the service instance.
	 */
	default String getScheme() {
		return null;
	}

}
```

上面提到的具体包装Server服务实例的RibbonServer对象就是ServiceInstance接口的实现，可以看到它除了包含了server对象之外，还存储了服务名、是否使用https标识以及一个map类型的元数据集合。

那么apply(final ServiceInstance instance)函数，在接收到了具体ServiceInstance实例后，是如何通过LoadBalancerClient接口中的reconstructURI操作来组织具体请求地址的呢？

##### apply(ServiceInstance instance)的两个实现

**org.springframework.cloud.client.loadbalancer.AsyncLoadBalancerInterceptor**

```java
public class AsyncLoadBalancerInterceptor implements AsyncClientHttpRequestInterceptor {

	private LoadBalancerClient loadBalancer;

	public AsyncLoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
		this.loadBalancer = loadBalancer;
	}

	@Override
	public ListenableFuture<ClientHttpResponse> intercept(final HttpRequest request,
			final byte[] body, final AsyncClientHttpRequestExecution execution)
			throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		return this.loadBalancer.execute(serviceName,
				new LoadBalancerRequest<ListenableFuture<ClientHttpResponse>>() {
					@Override
					public ListenableFuture<ClientHttpResponse> apply(
							final ServiceInstance instance) throws Exception {
						HttpRequest serviceRequest = new ServiceRequestWrapper(request,
								instance, AsyncLoadBalancerInterceptor.this.loadBalancer);
						return execution.executeAsync(serviceRequest, body);
					}

				});
	}

}
```

**org.springframework.cloud.client.loadbalancer.LoadBalancerRequestFactory**

```java
public class LoadBalancerRequestFactory {

	private LoadBalancerClient loadBalancer;

	private List<LoadBalancerRequestTransformer> transformers;

	public LoadBalancerRequestFactory(LoadBalancerClient loadBalancer,
			List<LoadBalancerRequestTransformer> transformers) {
		this.loadBalancer = loadBalancer;
		this.transformers = transformers;
	}

	public LoadBalancerRequestFactory(LoadBalancerClient loadBalancer) {
		this.loadBalancer = loadBalancer;
	}

	public LoadBalancerRequest<ClientHttpResponse> createRequest(
			final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) {
		return instance -> {
			HttpRequest serviceRequest = new ServiceRequestWrapper(request, instance,
					this.loadBalancer);
			if (this.transformers != null) {
				for (LoadBalancerRequestTransformer transformer : this.transformers) {
					serviceRequest = transformer.transformRequest(serviceRequest,
							instance);
				}
			}
			return execution.execute(serviceRequest, body);
		};
	}

}
```

从apply的实现中，我们可以看到它具体执行的时候，还创建了ServiceRequestWrapper对象，该对象继承了HttpRequestWrapper并重写了getURI函数，重写后的getURI会通过调用LoadBalancerClient接口的reconstructURI函数来重新构建一个URI进行访问

**org.springframework.cloud.client.loadbalancer.ServiceRequestWrapper**

```java
public class ServiceRequestWrapper extends HttpRequestWrapper {

	private final ServiceInstance instance;

	private final LoadBalancerClient loadBalancer;

	public ServiceRequestWrapper(HttpRequest request, ServiceInstance instance,
			LoadBalancerClient loadBalancer) {
		super(request);
		this.instance = instance;
		this.loadBalancer = loadBalancer;
	}

	@Override
	public URI getURI() {
		URI uri = this.loadBalancer.reconstructURI(this.instance, getRequest().getURI());
		return uri;
	}

}
```

在LoadBalancerInterceptor拦截器中，InterceptingRequestExecution的实例具体执行exection.execute(serviceRequest,body)时，会调用InterceptingClientHttpRequest的内部类InterceptingRequestExecution类的execute函数，具体实现如下：

org.springframework.http.client.InterceptingClientHttpRequest的内部类

```java
class InterceptingClientHttpRequest extends AbstractBufferingClientHttpRequest {
    private final ClientHttpRequestFactory requestFactory;
    private final List<ClientHttpRequestInterceptor> interceptors;
    private HttpMethod method;
    private URI uri;

    protected InterceptingClientHttpRequest(ClientHttpRequestFactory requestFactory, List<ClientHttpRequestInterceptor> interceptors, URI uri, HttpMethod method) {
        this.requestFactory = requestFactory;
        this.interceptors = interceptors;
        this.method = method;
        this.uri = uri;
    }

    public HttpMethod getMethod() {
        return this.method;
    }

    public String getMethodValue() {
        return this.method.name();
    }

    public URI getURI() {
        return this.uri;
    }

    protected final ClientHttpResponse executeInternal(HttpHeaders headers, byte[] bufferedOutput) throws IOException {
        InterceptingClientHttpRequest.InterceptingRequestExecution requestExecution = new InterceptingClientHttpRequest.InterceptingRequestExecution();
        return requestExecution.execute(this, bufferedOutput);
    }

    private class InterceptingRequestExecution implements ClientHttpRequestExecution {
        private final Iterator<ClientHttpRequestInterceptor> iterator;

        public InterceptingRequestExecution() {
            this.iterator = InterceptingClientHttpRequest.this.interceptors.iterator();
        }

        public ClientHttpResponse execute(HttpRequest request, byte[] body) throws IOException {
            if (this.iterator.hasNext()) {
                ClientHttpRequestInterceptor nextInterceptor = (ClientHttpRequestInterceptor)this.iterator.next();
                return nextInterceptor.intercept(request, body, this);
            } else {
                HttpMethod method = request.getMethod();
                Assert.state(method != null, "No standard HTTP method");
                ClientHttpRequest delegate = InterceptingClientHttpRequest.this.requestFactory.createRequest(request.getURI(), method);
                request.getHeaders().forEach((key, value) -> {
                    delegate.getHeaders().addAll(key, value);
                });
                if (body.length > 0) {
                    if (delegate instanceof StreamingHttpOutputMessage) {
                        StreamingHttpOutputMessage streamingOutputMessage = (StreamingHttpOutputMessage)delegate;
                        streamingOutputMessage.setBody((outputStream) -> {
                            StreamUtils.copy(body, outputStream);
                        });
                    } else {
                        StreamUtils.copy(body, delegate.getBody());
                    }
                }

                return delegate.execute();
            }
        }
    }
}
```

可以看到在创建请求的时候requestFactory.createRequest(request.getURI,request.getMethod)，这里的会调用之前介绍的ServiceRequestWrapper对象中重写的getURI函数。

```java
public class ServiceRequestWrapper extends HttpRequestWrapper {

	private final ServiceInstance instance;

	private final LoadBalancerClient loadBalancer;

	public ServiceRequestWrapper(HttpRequest request, ServiceInstance instance,
			LoadBalancerClient loadBalancer) {
		super(request);
		this.instance = instance;
		this.loadBalancer = loadBalancer;
	}

	@Override
	public URI getURI() {
		URI uri = this.loadBalancer.reconstructURI(this.instance, getRequest().getURI());
		return uri;
	}

}
```

此时，它就会使用RibbonLoadBalancerClient中实现的reconstructURI来组织具体请求的服务实例地址。

从reconstructURI函数中，我们可以看到，它通过ServiceInstance实例对象的servicId，从SpringClientFactory类的clientFactory对象中获取对应serviceId的负载均衡器的上下文RibbonLoadBalancerContext对象。然后根据ServiceInstance中的信息来构建具体服务实例信息的server对象，并使用RibbonLoadBalancerContext对象的reconstructURIWithServer函数来构建服务实例等URI。

为了帮助理解，简单介绍一下上面提到的SpringClientFactory和RibbonLoadBalancerContext:

- SpringClientFactory类是一个用来创建客户端负载均衡器的工厂类，该工厂类会为每一个不同名的Ribbon客户端生成不同的Spring上下文。

- RibbonLoadBalancerContext类是LoadBalancerContext的子类，该类用于存储一些被负载均衡使用的上下文内容和API操作(reconstructURIWithServer就是其中之一)

**com.netflix.loadbalancer.LoadBalancerContext**

```java
public class LoadBalancerContext implements IClientConfigAware {
    private static final Logger logger = LoggerFactory.getLogger(LoadBalancerContext.class);

    protected String clientName = "default";          

    protected String vipAddresses;

    protected int maxAutoRetriesNextServer = DefaultClientConfigImpl.DEFAULT_MAX_AUTO_RETRIES_NEXT_SERVER;
    protected int maxAutoRetries = DefaultClientConfigImpl.DEFAULT_MAX_AUTO_RETRIES;

    protected RetryHandler defaultRetryHandler = new DefaultLoadBalancerRetryHandler();


    protected boolean okToRetryOnAllOperations = DefaultClientConfigImpl.DEFAULT_OK_TO_RETRY_ON_ALL_OPERATIONS.booleanValue();

    private ILoadBalancer lb;

    private volatile Timer tracer;

    public LoadBalancerContext(ILoadBalancer lb) {
        this.lb = lb;
    }

    /**
     * Delegate to {@link #initWithNiwsConfig(IClientConfig)}
     * @param clientConfig
     */
    public LoadBalancerContext(ILoadBalancer lb, IClientConfig clientConfig) {
        this.lb = lb;
        initWithNiwsConfig(clientConfig);        
    }

    public LoadBalancerContext(ILoadBalancer lb, IClientConfig clientConfig, RetryHandler handler) {
        this(lb, clientConfig);
        this.defaultRetryHandler = handler;
    }

    /**
     * Set necessary parameters from client configuration and register with Servo monitors.
     */
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        if (clientConfig == null) {
            return;    
        }
        clientName = clientConfig.getClientName();
        if (clientName == null) {
            clientName = "default";
        }
        vipAddresses = clientConfig.resolveDeploymentContextbasedVipAddresses();
        maxAutoRetries = clientConfig.getPropertyAsInteger(CommonClientConfigKey.MaxAutoRetries, DefaultClientConfigImpl.DEFAULT_MAX_AUTO_RETRIES);
        maxAutoRetriesNextServer = clientConfig.getPropertyAsInteger(CommonClientConfigKey.MaxAutoRetriesNextServer,maxAutoRetriesNextServer);

        okToRetryOnAllOperations = clientConfig.getPropertyAsBoolean(CommonClientConfigKey.OkToRetryOnAllOperations, okToRetryOnAllOperations);
        defaultRetryHandler = new DefaultLoadBalancerRetryHandler(clientConfig);
        
        tracer = getExecuteTracer();

        Monitors.registerObject("Client_" + clientName, this);
    }

    public Timer getExecuteTracer() {
        if (tracer == null) {
            synchronized(this) {
                if (tracer == null) {
                    tracer = Monitors.newTimer(clientName + "_LoadBalancerExecutionTimer", TimeUnit.MILLISECONDS);                    
                }
            }
        } 
        return tracer;        
    }

    public String getClientName() {
        return clientName;
    }

    public ILoadBalancer getLoadBalancer() {
        return lb;    
    }

    public void setLoadBalancer(ILoadBalancer lb) {
        this.lb = lb;
    }
//...    
}
```

从LoadBalancerContext的reconstructURIWithServer函数中我们可以看到，它同reconstructURI的定义类似。只是reconstructURI的第一个保存具体服务实例的参数使用了Spring Cloud定义的ServiceInstance，而reconstructURIWithServer中使用了Netflix中定义的server，所以在RibbonLoadBalancerClient实现reconstructURI时候，做了一次转换，使用ServiceInstance的host和port信息来构建了一个Server对象来给reconstructURIWithServer使用。从reconstructURIWithServer的实现逻辑中，我们可以看到，它从Server对象中获取host和port信息，然后根据以服务名为host的URI对象original中获取其他请求信息 ，将两者内容进行拼接整合，形成最终要访问的服务实例等具体地址。

另外，从RibbonLoadBalancerClient的execute的函数逻辑中，我们还能看到在回调拦截器中，执行具体的请求后，ribbon还通过RibbonStatsRecorder对象对服务的请求还进行了跟踪记录，有兴趣的读者可以继续研究。

分析到这里，我们已经可以大致理清Spring Cloud Ribbon中实现客户端负载均衡的基本脉络，了解了它是如何通过LoadBalancerInterceptor拦截器对RequestTemplate的请求进行拦截，并利用Spring Cloud的负载均衡器LoadBalancerClient将以服务名为host的URI转换成具体的服务实例地址的过程。同时通过分析LoadBalancerClient的Ribbon实现RibbonLoadBalancerClient，可以知道在使用Ribbon实现负载均衡器的时候，实际使用的还是Ribbon中定义ILoadBalancer接口的实现，自动化配置会采用ZoneAwareLoadBalancer的实例来实现客户端负载均衡。

##### 负载均衡器

通过之前的分析，我们已经对Spring Cloud如何使用有了基本的了解。虽然Spring Cloud中定义了LoadBalancerClient为负载均衡器等接口，并且针对Ribbon实现了RibbonLoadBalancerClient，但是它在具体实现客户端负载均衡时，则是通过Ribbon的ILoadBalancer接口实现。下面我们根据ILoadBalancer接口的实现类逐个看看它都是如何实现客户端负载均衡的。

![1Jk8L8.png](https://s2.ax1x.com/2020/02/01/1Jk8L8.png)

**AbstractLoadBalancer**

com.netflix.loadbalancer.AbstractLoadBalancer

```java
public abstract class AbstractLoadBalancer implements ILoadBalancer {
    
    public enum ServerGroup{
        ALL,
        STATUS_UP,
        STATUS_NOT_UP        
    }
        
    /**
     * delegate to {@link #chooseServer(Object)} with parameter null.
     */
    public Server chooseServer() {
    	return chooseServer(null);
    }

    
    /**
     * List of servers that this Loadbalancer knows about
     * 
     * @param serverGroup Servers grouped by status, e.g., {@link ServerGroup#STATUS_UP}
     */
    public abstract List<Server> getServerList(ServerGroup serverGroup);
    
    /**
     * Obtain LoadBalancer related Statistics
     */
    public abstract LoadBalancerStats getLoadBalancerStats();    
}
```

AbstractLoadBalancer是ILoadBalancer接口的抽象实现。在该抽象类中定义了一个关于测试服务实例的分组枚举类ServerGroup，它包含了三种不同类型：

- ALL：所有服务实例

- STATUS_UP：正常服务的实例

- STATUS_NOT_UP：停止服务的实例


另外，还实现了一个chooseServer()函数，该函数通过调用接口中的chooseServer(Object key)实现，其中参数key为null，表示在选择具体服务实例时忽略key的条件判断。

最后还定义了两个抽象函数：

- getServerList(ServerGroup serverGroup)：定义了根据分组类型来获取不同的服务实例列表

- getLoadBalancerStats()：定义了获取LoadBalancerStats对象的方法，loadBalancerStats对象被用来存储负载均衡器中各个服务实例当前的属性和统计信息，这些信息非常有用，我们可以利用这些信息来观察负载均衡的运行

**BaseLoadBalancer**

com.netflix.loadbalancer.BaseLoadBalancer

```java
public class BaseLoadBalancer extends AbstractLoadBalancer implements
        PrimeConnections.PrimeConnectionListener, IClientConfigAware {

    private static Logger logger = LoggerFactory
            .getLogger(BaseLoadBalancer.class);
    private final static IRule DEFAULT_RULE = new RoundRobinRule();
    private final static SerialPingStrategy DEFAULT_PING_STRATEGY = new SerialPingStrategy();
    private static final String DEFAULT_NAME = "default";
    private static final String PREFIX = "LoadBalancer_";

    protected IRule rule = DEFAULT_RULE;

    protected IPingStrategy pingStrategy = DEFAULT_PING_STRATEGY;

    protected IPing ping = null;

    @Monitor(name = PREFIX + "AllServerList", type = DataSourceType.INFORMATIONAL)
    protected volatile List<Server> allServerList = Collections
            .synchronizedList(new ArrayList<Server>());
    @Monitor(name = PREFIX + "UpServerList", type = DataSourceType.INFORMATIONAL)
    protected volatile List<Server> upServerList = Collections
            .synchronizedList(new ArrayList<Server>());

    protected ReadWriteLock allServerLock = new ReentrantReadWriteLock();
    protected ReadWriteLock upServerLock = new ReentrantReadWriteLock();

    protected String name = DEFAULT_NAME;

    protected Timer lbTimer = null;
    protected int pingIntervalSeconds = 10;
    protected int maxTotalPingTimeSeconds = 5;
    protected Comparator<Server> serverComparator = new ServerComparator();

    protected AtomicBoolean pingInProgress = new AtomicBoolean(false);

    protected LoadBalancerStats lbStats;

    private volatile Counter counter = Monitors.newCounter("LoadBalancer_ChooseServer");

    private PrimeConnections primeConnections;

    private volatile boolean enablePrimingConnections = false;
    
    private IClientConfig config;
    
    private List<ServerListChangeListener> changeListeners = new CopyOnWriteArrayList<ServerListChangeListener>();

    private List<ServerStatusChangeListener> serverStatusListeners = new CopyOnWriteArrayList<ServerStatusChangeListener>();
    //...
    }


```

一个负载均衡器的基础实现，该负载均衡器的作用是维护一张可以将服务设置到服务池的任意表。

一个ping能觉得一个服务的活性。本质上，这个类保存一个“all”服务列表和一个“up”服务列表，还有支持消费者使用它们。

BaseLoadBalancer类是Ribbon负载均衡器的基础实现类，在该类中定义很多关于均衡负载相关的基础内容：

定义并维护了两个存储服务实例server对象的列表。一个用于存储所有服务实例的清单，一个用于存储正常服务的实例清单。

定义了之前我们提到的用来存储负载均衡器各服务实例属性和统计信息的LoadBalancerStats对象。

定义了检查服务实例是否正常服务的ping对象，在BaseLoadBalancer中默认为null，需要在构造时注入它的具体实现。

定义了检查服务实例操作的执行策略对象IPingStrategy，在BaseLoadBalancer中默认使用了该类中定义的静态内部类SerialPingStrategy实现。根据源码，我们可以看到该策略采用线性遍历ping服务实例等方式实现检查。该策略在当IPing当实现速度不理想，或是Server列表过大时，可能会影响系统性能，这时候需要通过实现IPingStrategy接口并重写pingServers(IPing ping, Server[] servers)函数去扩展ping的执行策略。

```java
   /**
     * Default implementation for <c>IPingStrategy</c>, performs ping
     * serially, which may not be desirable, if your <c>IPing</c>
     * implementation is slow, or you have large number of servers.
     */
    private static class SerialPingStrategy implements IPingStrategy {

        @Override
        public boolean[] pingServers(IPing ping, Server[] servers) {
            int numCandidates = servers.length;
            boolean[] results = new boolean[numCandidates];

            logger.debug("LoadBalancer:  PingTask executing [{}] servers configured", numCandidates);

            for (int i = 0; i < numCandidates; i++) {
                results[i] = false; /* Default answer is DEAD. */
                try {
                    // NOTE: IFF we were doing a real ping
                    // assuming we had a large set of servers (say 15)
                    // the logic below will run them serially
                    // hence taking 15 times the amount of time it takes
                    // to ping each server
                    // A better method would be to put this in an executor
                    // pool
                    // But, at the time of this writing, we dont REALLY
                    // use a Real Ping (its mostly in memory eureka call)
                    // hence we can afford to simplify this design and run
                    // this
                    // serially
                    if (ping != null) {
                        results[i] = ping.isAlive(servers[i]);
                    }
                } catch (Exception e) {
                    logger.error("Exception while pinging Server: '{}'", servers[i], e);
                }
            }
            return results;
        }
    }
```

定义了负载均衡的处理规则IRule对象，从BaseLoadBalancer中chooseServer(Object key)的实现源码，我们可以知道负载均衡器实际进行服务实例选择任务是委托给了IRule实现中的choose函数中实现而在这里，默认初始化了RoundRobinRule为IRule的实现对象。RoundRobinRule实现了最基本且常用的线性负载均衡规则。

```java
/**
 * The most well known and basic load balancing strategy, i.e. Round Robin Rule.
 *
 * @author stonse
 * @author Nikos Michalakis <nikos@netflix.com>
 *
 */
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

 **启动ping任务**：在BaseLoadBalancer的默认构造函数中，会直接启动一个用于定时检查server是否健康的任务。该任务默认的执行间隔为：10秒。

实现了ILoadBalancer接口定义的负载均衡器应具备的一系列基本操作：

**addServer(List newServer)**：向负载均衡器中增加新的服务实例列表，该实现将原本已经维护着的所有服务实例清单allServerList和新传入的服务实例清单newServers都加入到newList进行处理，在BaseLoadBalancer中实现的时候会使用新的列表覆盖旧的列表。而之后介绍的几个扩展实现类对于服务实例清单更新的优化都是通过对setServersList函数的重写来实现的。

```java
    /**
     * Add a server to the 'allServer' list; does not verify uniqueness, so you
     * could give a server a greater share by adding it more than once.
     */
    public void addServer(Server newServer) {
        if (newServer != null) {
            try {
                ArrayList<Server> newList = new ArrayList<Server>();

                newList.addAll(allServerList);
                newList.add(newServer);
                setServersList(newList);
            } catch (Exception e) {
                logger.error("LoadBalancer [{}]: Error adding newServer {}", name, newServer.getHost(), e);
            }
        }
    }
```

**chooseServer(Object key)**：挑选一个具体的服务实例，在上面介绍IRule的时候，已经做了说明。

**markServerDown(Server server)**：标记某个服务实例暂停服务。

```java
    public void markServerDown(Server server) {
        if (server == null || !server.isAlive()) {
            return;
        }

        logger.error("LoadBalancer [{}]:  markServerDown called on [{}]", name, server.getId());
        server.setAlive(false);
        // forceQuickPing();

        notifyServerStatusChangeListener(singleton(server));
    }

    public void markServerDown(String id) {
        boolean triggered = false;

        id = Server.normalizeId(id);

        if (id == null) {
            return;
        }

        Lock writeLock = upServerLock.writeLock();
    	writeLock.lock();
        try {
            final List<Server> changedServers = new ArrayList<Server>();

            for (Server svr : upServerList) {
                if (svr.isAlive() && (svr.getId().equals(id))) {
                    triggered = true;
                    svr.setAlive(false);
                    changedServers.add(svr);
                }
            }

            if (triggered) {
                logger.error("LoadBalancer [{}]:  markServerDown called for server [{}]", name, id);
                notifyServerStatusChangeListener(changedServers);
            }

        } finally {
            writeLock.unlock();
        }
    }
```

 **getReachableServers()：**获取可用的（可达的）实例列表。由于BaseLoadBalancer中单独维护了一个正常服务的实例清单，所以直接返回即可。

**getAllServer()**:获取所有的服务实例列表。由于BaseLoadBalancer中单独维护了一个所有服务的实例清单，所以也直接返回它即可。

```java
    /**
     * return the server
     * 
     * @param index
     * @param availableOnly
     */
    public Server getServerByIndex(int index, boolean availableOnly) {
        try {
            return (availableOnly ? upServerList.get(index) : allServerList
                    .get(index));
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public List<Server> getServerList(boolean availableOnly) {
        return (availableOnly ? getReachableServers() : getAllServers());
    }

    @Override
    public List<Server> getReachableServers() {
        return Collections.unmodifiableList(upServerList);
    }

    @Override
    public List<Server> getAllServers() {
        return Collections.unmodifiableList(allServerList);
    }

    @Override
    public List<Server> getServerList(ServerGroup serverGroup) {
        switch (serverGroup) {
        case ALL:
            return allServerList;
        case STATUS_UP:
            return upServerList;
        case STATUS_NOT_UP:
            ArrayList<Server> notAvailableServers = new ArrayList<Server>(
                    allServerList);
            ArrayList<Server> upServers = new ArrayList<Server>(upServerList);
            notAvailableServers.removeAll(upServers);
            return notAvailableServers;
        }
        return new ArrayList<Server>();
    }

```

**DynamicServerListLoadBalancer**

com.netflix.loadbalancer.DynamicServerListLoadBalancer

```java
/**
 * A LoadBalancer that has the capabilities to obtain the candidate list of
 * servers using a dynamic source. i.e. The list of servers can potentially be
 * changed at Runtime. It also contains facilities wherein the list of servers
 * can be passed through a Filter criteria to filter out servers that do not
 * meet the desired criteria.
 * 
 * @author stonse
 * 
 */
public class DynamicServerListLoadBalancer<T extends Server> extends BaseLoadBalancer {
    private static final Logger LOGGER = LoggerFactory.getLogger(DynamicServerListLoadBalancer.class);

    boolean isSecure = false;
    boolean useTunnel = false;

    // to keep track of modification of server lists
    protected AtomicBoolean serverListUpdateInProgress = new AtomicBoolean(false);

    volatile ServerList<T> serverListImpl;

    volatile ServerListFilter<T> filter;

    protected final ServerListUpdater.UpdateAction updateAction = new ServerListUpdater.UpdateAction() {
        @Override
        public void doUpdate() {
            updateListOfServers();
        }
    };

    protected volatile ServerListUpdater serverListUpdater;

    public DynamicServerListLoadBalancer() {
        super();
    }
    //..
}
```

DynamicServerListLoadBalancer类继承于BaseLoadBalancer类，它是对基础负载均衡器的扩展。在该负载均衡器中，实现了服务实例清单的在运行期的动态更新能力；同时，它还具备了对服务实例清单的过滤功能，也就是说我们可以通过过滤器来选择性的获取一批服务实例清单。

**ServerList**

从DynamicServerListLoadBalancer的成员定义中，我们马上可以发现新增了一个关于服务列表的操作对象：ServerList serverListImpl。其中范型T从类名中对于T的限定DynamicServerListLoadBalancer可以获知它是一个server的子类，即代表了一个具体的服务实例的扩展类。而Server接口定义如下所示：

```java
/**
 * Interface that defines the methods sed to obtain the List of Servers
 * @author stonse
 *
 * @param <T>
 */
public interface ServerList<T extends Server> {

    public List<T> getInitialListOfServers();
    
    /**
     * Return updated list of servers. This is called say every 30 secs
     * (configurable) by the Loadbalancer's Ping cycle
     * 
     */
    public List<T> getUpdatedListOfServers();   

}
```

它定义了两个抽象方法：getInitialListOfServers用于获取初始化的服务实例清单，而getUpdatedListOfServers用于获取更新的服务实例清单。那么该接口的实现有哪些呢？通过搜索源码，我们可以整出如下图的结构：

![1JABhd.png](https://s2.ax1x.com/2020/02/01/1JABhd.png)

从图中我们可以看到有很多个ServerList的实现类，那么在DynamicServerListLoadBalancer中的ServerList默认配置到底使用了哪个具体实现呢？既然在该负载均衡器中需要实现服务实例的动态更新，那么势必需要ribbon具备访问eureka来获取服务实例的能力，所以我们从Spring Cloud整合ribbon与eureka的EurekaRibbonClientConfiguration，在该类中可以找到下面创建ServerList实例的内容。

```java
public class DomainExtractingServerList implements ServerList<DiscoveryEnabledServer> {
    private ServerList<DiscoveryEnabledServer> list;
    private final RibbonProperties ribbon;
    private boolean approximateZoneFromHostname;

    public DomainExtractingServerList(ServerList<DiscoveryEnabledServer> list, IClientConfig clientConfig, boolean approximateZoneFromHostname) {
        this.list = list;
        this.ribbon = RibbonProperties.from(clientConfig);
        this.approximateZoneFromHostname = approximateZoneFromHostname;
    }

    public List<DiscoveryEnabledServer> getInitialListOfServers() {
        List<DiscoveryEnabledServer> servers = this.setZones(this.list.getInitialListOfServers());
        return servers;
    }

    public List<DiscoveryEnabledServer> getUpdatedListOfServers() {
        List<DiscoveryEnabledServer> servers = this.setZones(this.list.getUpdatedListOfServers());
        return servers;
    }
    //..
    }
```

那么DiscoveryEnabledNIWSServerList是如何实现这两个服务实例的获取的呢？从源码中可以看到这两个方法都是通过该类中的私有函数obtainServersViaDiscovery来通过服务发现机制来实现服务实例的获取。

```java
    public List<DiscoveryEnabledServer> getInitialListOfServers() {
        return this.obtainServersViaDiscovery();
    }

    public List<DiscoveryEnabledServer> getUpdatedListOfServers() {
        return this.obtainServersViaDiscovery();
    }

    private List<DiscoveryEnabledServer> obtainServersViaDiscovery() {
        List<DiscoveryEnabledServer> serverList = new ArrayList();
        if (this.eurekaClientProvider != null && this.eurekaClientProvider.get() != null) {
            EurekaClient eurekaClient = (EurekaClient)this.eurekaClientProvider.get();
            if (this.vipAddresses != null) {
                String[] var3 = this.vipAddresses.split(",");
                int var4 = var3.length;

                for(int var5 = 0; var5 < var4; ++var5) {
                    String vipAddress = var3[var5];
                    List<InstanceInfo> listOfInstanceInfo = eurekaClient.getInstancesByVipAddress(vipAddress, this.isSecure, this.targetRegion);
                    Iterator var8 = listOfInstanceInfo.iterator();

                    while(var8.hasNext()) {
                        InstanceInfo ii = (InstanceInfo)var8.next();
                        if (ii.getStatus().equals(InstanceStatus.UP)) {
                            if (this.shouldUseOverridePort) {
                                if (logger.isDebugEnabled()) {
                                    logger.debug("Overriding port on client name: " + this.clientName + " to " + this.overridePort);
                                }

                                InstanceInfo copy = new InstanceInfo(ii);
                                if (this.isSecure) {
                                    ii = (new Builder(copy)).setSecurePort(this.overridePort).build();
                                } else {
                                    ii = (new Builder(copy)).setPort(this.overridePort).build();
                                }
                            }

                            DiscoveryEnabledServer des = this.createServer(ii, this.isSecure, this.shouldUseIpAddr);
                            serverList.add(des);
                        }
                    }

                    if (serverList.size() > 0 && this.prioritizeVipAddressBasedServers) {
                        break;
                    }
                }
            }

            return serverList;
        } else {
            logger.warn("EurekaClient has not been initialized yet, returning an empty list");
            return new ArrayList();
        }
    }
```

而obtainServersViaDiscovery的实现逻辑如下，主要依靠EurekaClient从服务注册中心中获取到具体的服务实例InstanceInfo列表(EurekaClient的具体实现，这里传入的vipAddress可以理解为逻辑上的服务名，比如“USER-SERVICE”)。接着，对这些服务实例进行遍历，将状态为“UP”（正常服务）的实例转换成DiscoveryEnabledServer对象，最后将这些实例组织成列表返回。

在DiscoveryEnabledNIWSServerList中通过EurekaClient从服务注册中心获取到最新的服务实例清单后，返回的List到了DomainExtractingServerList类中，将继续通过setZones函数进行处理。而这里的处理具体内容如下所示，主要完成将DiscoveryEnabledNIWSServerList返回的List列表中的元素，转换成内部定义的DiscoveryEnabledServer的子类对象DomainExtractingServer，在该对象的构造函数中将为服务实例对象设置一些必要的属性，比如id、zone、isAliveFlag、readyToServe等信息。



### 参考

1. [Ribbon详解](https://www.jianshu.com/p/1bd66db5dc46)