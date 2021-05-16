---
title: Spring boot Spring MVC （二）
date: 2019-06-30 15:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

### 配置嵌入式Servlet容器

SpringBoot2.x支持的嵌入式servlet容器包括tomcat、jetty、undertow、netty。SpringBoot默认使用Tomcat 9作为嵌入式的Servlet容器；

#### 如何定制和修改Servlet容器的相关配置

Spring Boot 2.0.2 版本的嵌入式Servlet容器自动配置是通过WebServerFactoryCustomizer定制器来定制的，而在Spring Boot 1.x 版本中我们是通过EmbeddedServletContainerCustomizer嵌入式的Servlet容器定制器来定制的。

<!--more-->

1. 修改和server有关的配置（ServerProperties、TomcatServletWebServerFactory）

   ```properties
   server.port=8081
   server.servlet.context-path=/crud
   
   server.tomcat.uri-encoding=UTF-8
   
   //通用的Servlet容器设置
   server.xxx
   //Tomcat的设置
   server.tomcat.xxx
   ```

2. 编写一个**WebServerFactoryCustomizer** ：嵌入式的Servlet容器定制器，来修改Servlet容器的配置

   ```java
   @Configuration
   public class MyServerConfigurer {
    	//向IoC容器中添加servlet容器工厂定制器
           //Tomcat的配置TomcatServletWebServerFactory，另外还有Jetty、Undertow、Netty
       //如果自定义,则需要继承AbstractServletWebServerFactory
    	@Bean
   	public WebServerFactoryCustomizer<TomcatServletWebServerFactory> myWebServerFactoryCustomizer() {
   		return new WebServerFactoryCustomizer<TomcatServletWebServerFactory>() {
   			public void customize(TomcatServletWebServerFactory factory) {
   		  	 //设置相关配置
   		   	 factory.setPort(8082);
   			}
   		 };
   	 }
   }
   ```
   
3. 向IoC容器中添加可配置的servlet容器工厂 **ConfigurableServletWebServerFactory**

   ```java
       //向IoC容器中添加servlet容器工厂
       @Bean
       public ConfigurableServletWebServerFactory myConfigurableServletWebServerFactory() {
           
           TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
           factory.setPort(8083);
           return factory;
       }
   ```

其他嵌入式servlet容器配置以此类推。

#### 注册Servlet三大组件【Servlet、Filter、Listener】

SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件，如果要注册三大组件，那怎么办？

**除下面方法，还有Servlet 3的注解方法**

可以用以下方式：

- ServletRegistrationBean

  ```java
  //注册三大组件
  @Bean
  public ServletRegistrationBean myServlet(){
      ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
      return registrationBean;
  }
  ```

- FilterRegistrationBean

  ```java
  @Bean
  public FilterRegistrationBean myFilter(){
      FilterRegistrationBean registrationBean = new FilterRegistrationBean();
      registrationBean.setFilter(new MyFilter());
      registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
      return registrationBean;
  }
  ```

- ServletListenerRegistrationBean

  ```java
  @Bean
  public ServletListenerRegistrationBean myListener(){
      ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
      return registrationBean;
  }
  ```

原理：SpringBoot帮我们自动配置SprinMVC的时候，自动的注册了SpringMVC的前端控制器：DispatcherServlet;

可以参考自动配置类`org.springframework.boot.autoconfigure.web.servletDispatcherServletAutoConfiguration`：

```java

@AutoConfigureOrder(-2147483648)
@Configuration
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({DispatcherServlet.class})
@AutoConfigureAfter({ServletWebServerFactoryAutoConfiguration.class})
public class DispatcherServletAutoConfiguration {
    public static final String DEFAULT_DISPATCHER_SERVLET_BEAN_NAME = "dispatcherServlet";
    public static final String DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME = "dispatcherServletRegistration";
 
//other code...
 
    @Configuration
    @Conditional({DispatcherServletAutoConfiguration.DispatcherServletRegistrationCondition.class})
    @ConditionalOnClass({ServletRegistration.class})
    @EnableConfigurationProperties({WebMvcProperties.class})
    @Import({DispatcherServletAutoConfiguration.DispatcherServletConfiguration.class})
    protected static class DispatcherServletRegistrationConfiguration {
        private final WebMvcProperties webMvcProperties;
        private final MultipartConfigElement multipartConfig;
 
        public DispatcherServletRegistrationConfiguration(WebMvcProperties webMvcProperties, ObjectProvider<MultipartConfigElement> multipartConfigProvider) {
            this.webMvcProperties = webMvcProperties;
            this.multipartConfig = (MultipartConfigElement)multipartConfigProvider.getIfAvailable();
        }
 
        @Bean(
            name = {"dispatcherServletRegistration"}
        )
        @ConditionalOnBean(
            value = {DispatcherServlet.class},
            name = {"dispatcherServlet"}
        )
        public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet) {
            DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet, this.webMvcProperties.getServlet().getPath());
        //默认拦截 / 所有请求，包括静态资源，但是不拦截jsp请求；/*会拦截jsp
        //可以通过修改server.servlet-path来修改默认的拦截请求
            registration.setName("dispatcherServlet");
            registration.setLoadOnStartup(this.webMvcProperties.getServlet().getLoadOnStartup());
            if (this.multipartConfig != null) {
                registration.setMultipartConfig(this.multipartConfig);
            }
 
            return registration;
        }
    }  
//....
}
```

#### SpringBoot能不能支持其他的Servlet容器？

默认支持Tomcat、Jetty和Undertow.

- tomcat：开源，SpringBoot默认，使用比较多

- jetty：小Serlvet容器，适合做长链接，也就是链接上就不断，长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况，很多人开发测试的时候很喜欢用，因为它启动比tomcat要快很多
- undertow：是一个非阻塞的Servlet容器，但是它不支持JSP。

Spring Boot 默认使用Tomcat，一旦引入 spring-boot-starter-web 模块，就默认使用Tomcat容器**。**

**如何替换为其他嵌入式的Servlet容器？**

切换其他Servlet容器，只要修改项目的pom.xml文件即可：

1. 将tomcat依赖移除掉
2. 引入其他Servlet容器依赖

注意：切换了Servlet容器后，只是以server开头的配置不一样外，其他都类似

Tomcat(默认使用)

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器；
</dependency>
```

Jetty

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

Undertow

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

#### 嵌入式Servlet容器自动配置原理

Spring Boot 通过运行主程序类的main()方法来启动Spring Boot 应用，应用启动后调用ServletWebServerApplicationContext这个类中的createWebServer()方法：

```java
private void createWebServer() {
        WebServer webServer = this.webServer;
        ServletContext servletContext = this.getServletContext();
        if (webServer == null && servletContext == null) {
                ServletWebServerFactory factory = this.getWebServerFactory();
                this.webServer = factory.getWebServer(new ServletContextInitializer[]{this.getSelfInitializer()});
        } else if (servletContext != null) {
                try {
                        this.getSelfInitializer().onStartup(servletContext);
                } catch (ServletException var4) {
                        throw new ApplicationContextException("Cannot initialize servlet context", var4);
                }
        }
        this.initPropertySources();
} 
```

该方法最终能够获取到一个与当前应用所导入的Servlet类型相匹配的web服务工厂定制器，例如你的pom文件导入的Servlet依赖为Tomcat

那么最终会生成Tomcat web服务工厂定制器，定制Servlet容器配置，并通过上述代码中的getWebServer()方法创建对应的Servlet容器，并启动容器。

```java
ServletWebServerFactory factory = this.getWebServerFactory();
```

上述代码执行来到EmbeddedWebServerFactoryCustomizerAutoConfiguration（嵌入式web服务工厂定制器自动配置类），根据导入的依赖信息，该配置类会自动创建相应类型的容器工厂定制器（目前Spring Boot 2.x 版本支持tomcat、jetty、undertow、netty），以tomcat为例，这里会创建TomcatWebServerFactoryCustomizer组件：

```java
//导入的Servlet依赖为Tomcat，则创建Tomcat web服务工厂定制器
@ConditionalOnClass({Tomcat.class, UpgradeProtocol.class})
public static class TomcatWebServerFactoryCustomizerConfiguration {
    public TomcatWebServerFactoryCustomizerConfiguration() {
    }

    @Bean
    public TomcatWebServerFactoryCustomizertomcatWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
        return new TomcatWebServerFactoryCustomizer(environment,serverProperties);
    }
}
```

需要注意的是，该自动配置类有一个@EnableConfigurationProperties注解：

```java
@EnableConfigurationProperties({ServerProperties.class})
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {
```

该注解用于启动指定类ServerProperties（Servlet容器相关的配置类）中的ConfigurationProperties功能，将配置文件中对应的属性值与配置类中的属性值进行映射，并将该配置类添加到IOC容器中：

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties {
```

至此，TomcatWebServerFactoryCustomizer组件创建完成，对应的服务配置类也已添加到IOC容器。

下面的处理涉及到一个非常重要的类-->WebServerFactoryCustomizerBeanPostProcessor（web服务工厂定制器组件的后置处理器），该类负责在bean组件初始化之前执行初始化工作。

该类先从IOC容器中获取所有类型为WebServerFactoryCustomizer（web服务工厂定制器）的组件：

```java
private Collection<WebServerFactoryCustomizer<?>> getWebServerFactoryCustomizerBeans() {
    return this.beanFactory.getBeansOfType(WebServerFactoryCustomizer.class, false, false).values();
}
```

通过源码的调试，从IOC容器中获取到5个定制器，其中包含有前面创建的TomcatWebServerFactoryCustomizer定制器：

![lFyu6J.png](https://s2.ax1x.com/2019/12/25/lFyu6J.png)

获取到所有的定制器后，后置处理器调用定制器的customize()方法来给嵌入式的Servlet容器进行配置（默认或者自定义的配置）：

```java
private void postProcessBeforeInitialization(WebServerFactory webServerFactory) {
    ((Callbacks)LambdaSafe.callbacks(WebServerFactoryCustomizer.class, this.getCustomizers(), webServerFactory, new Object[0]).withLogger(WebServerFactoryCustomizerBeanPostProcessor.class)).invoke((customizer) -> {
        customizer.customize(webServerFactory);
    });
}
```

TomcatWebServerFactoryCustomizer定制器调用customize()定制方法，获取到Servlet容器相关配置类ServerProperties，进行自动配置：

```java
	@Override
	public void customize(ConfigurableTomcatWebServerFactory factory) {
		ServerProperties properties = this.serverProperties;
		ServerProperties.Tomcat tomcatProperties = properties.getTomcat();
		PropertyMapper propertyMapper = PropertyMapper.get();
		propertyMapper.from(tomcatProperties::getBasedir).whenNonNull().to(factory::setBaseDirectory);
		propertyMapper.from(tomcatProperties::getBackgroundProcessorDelay).whenNonNull().as(Duration::getSeconds)
				.as(Long::intValue).to(factory::setBackgroundProcessorDelay);
		customizeRemoteIpValve(factory);
		propertyMapper.from(tomcatProperties::getMaxThreads).when(this::isPositive)
				.to((maxThreads) -> customizeMaxThreads(factory, tomcatProperties.getMaxThreads()));
		propertyMapper.from(tomcatProperties::getMinSpareThreads).when(this::isPositive)
				.to((minSpareThreads) -> customizeMinThreads(factory, minSpareThreads));
		propertyMapper.from(this.serverProperties.getMaxHttpHeaderSize()).whenNonNull().asInt(DataSize::toBytes)
				.when(this::isPositive)
				.to((maxHttpHeaderSize) -> customizeMaxHttpHeaderSize(factory, maxHttpHeaderSize));
		propertyMapper.from(tomcatProperties::getMaxSwallowSize).whenNonNull().asInt(DataSize::toBytes)
				.to((maxSwallowSize) -> customizeMaxSwallowSize(factory, maxSwallowSize));
		propertyMapper.from(tomcatProperties::getMaxHttpFormPostSize).asInt(DataSize::toBytes)
				.when((maxHttpFormPostSize) -> maxHttpFormPostSize != 0)
				.to((maxHttpFormPostSize) -> customizeMaxHttpFormPostSize(factory, maxHttpFormPostSize));
		propertyMapper.from(tomcatProperties::getAccesslog).when(ServerProperties.Tomcat.Accesslog::isEnabled)
				.to((enabled) -> customizeAccessLog(factory));
		propertyMapper.from(tomcatProperties::getUriEncoding).whenNonNull().to(factory::setUriEncoding);
		propertyMapper.from(properties::getConnectionTimeout).whenNonNull()
				.to((connectionTimeout) -> customizeConnectionTimeout(factory, connectionTimeout));
		propertyMapper.from(tomcatProperties::getConnectionTimeout).whenNonNull()
				.to((connectionTimeout) -> customizeConnectionTimeout(factory, connectionTimeout));
		propertyMapper.from(tomcatProperties::getMaxConnections).when(this::isPositive)
				.to((maxConnections) -> customizeMaxConnections(factory, maxConnections));
		propertyMapper.from(tomcatProperties::getAcceptCount).when(this::isPositive)
				.to((acceptCount) -> customizeAcceptCount(factory, acceptCount));
		propertyMapper.from(tomcatProperties::getProcessorCache)
				.to((processorCache) -> customizeProcessorCache(factory, processorCache));
		propertyMapper.from(tomcatProperties::getRelaxedPathChars).as(this::joinCharacters).whenHasText()
				.to((relaxedChars) -> customizeRelaxedPathChars(factory, relaxedChars));
		propertyMapper.from(tomcatProperties::getRelaxedQueryChars).as(this::joinCharacters).whenHasText()
				.to((relaxedChars) -> customizeRelaxedQueryChars(factory, relaxedChars));
		customizeStaticResources(factory);
		customizeErrorReportValve(properties.getError(), factory);
	}
```

至此，嵌入式Servlet容器的自动配置完成。

#### 嵌入式Servlet容器启动原理

什么时候创建嵌入式的Servlet容器工厂？什么时候获取嵌入式的Servlet容器并启动Tomcat?

1. SpringBoot应用启动运行run方法

2. refreshContext(context);SpringBoot刷新IoC容器【创建IoC容器对象，并初始化容器，创建容器中的每一个组件】；如果是web应用创建**AnnotationConfigEmbeddedWebApplicationContext**，否则：**AnnotationConfigApplicationContext**

3. refresh(context);  **刷新刚才创建好的IoC容器；**

   ```java
      public void refresh() throws BeansException, IllegalStateException {
         synchronized (this.startupShutdownMonitor) {
            // Prepare this context for refreshing.
            prepareRefresh();
   
            // Tell the subclass to refresh the internal bean factory.
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
            // Prepare the bean factory for use in this context.
            prepareBeanFactory(beanFactory);
   
            try {
               // Allows post-processing of the bean factory in context subclasses.
               postProcessBeanFactory(beanFactory);
   
               // Invoke factory processors registered as beans in the context.
               invokeBeanFactoryPostProcessors(beanFactory);
   
               // Register bean processors that intercept bean creation.
               registerBeanPostProcessors(beanFactory);
   
               // Initialize message source for this context.
               initMessageSource();
   
               // Initialize event multicaster for this context.
               initApplicationEventMulticaster();
   
               // Initialize other special beans in specific context subclasses.
               onRefresh();
   
               // Check for listener beans and register them.
               registerListeners();
   
               // Instantiate all remaining (non-lazy-init) singletons.
               finishBeanFactoryInitialization(beanFactory);
   
               // Last step: publish corresponding event.
               finishRefresh();
            }
   
            catch (BeansException ex) {
               if (logger.isWarnEnabled()) {
                  logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
               }
   
               // Destroy already created singletons to avoid dangling resources.
               destroyBeans();
   
               // Reset 'active' flag.
               cancelRefresh(ex);
   
               // Propagate exception to caller.
               throw ex;
            }
   
            finally {
               // Reset common introspection caches in Spring's core, since we
               // might not ever need metadata for singleton beans anymore...
               resetCommonCaches();
            }
         }
      }
   ```

4. onRefresh(); web的IoC容器ServletWebServerApplicationContext重写了onRefresh方法

   ```java
   protected void onRefresh() {
           super.onRefresh();
   
           try {
               this.createWebServer();
           } catch (Throwable var2) {
               throw new ApplicationContextException("Unable to start web server", var2);
           }
       }
   ```

5. web IoC容器会创建嵌入式的Servlet容器；**ServletWebServerApplicationContext.createWebServer**();

   ```java
       private void createWebServer() {
           WebServer webServer = this.webServer;
           ServletContext servletContext = this.getServletContext();
           //未创建
           if (webServer == null && servletContext == null) {
               //创建Web服务工厂对象
               ServletWebServerFactory factory = this.getWebServerFactory();
               this.webServer = factory.getWebServer(new ServletContextInitializer[]{this.getSelfInitializer()});
           } else if (servletContext != null) {
               try {
                   this.getSelfInitializer().onStartup(servletContext);
               } catch (ServletException var4) {
                   throw new ApplicationContextException("Cannot initialize servlet context", var4);
               }
           }
   
           this.initPropertySources();
       }
   ```

6. 应用启动后，根据导入的依赖信息，创建了相应的Servlet容器工厂，以tomcat为例，创建嵌入式的Tomcat容器工厂TomcatServletWebServerFactory，调用getWebServer()方法创建Tomcat容器

   ```java
       public WebServer getWebServer(ServletContextInitializer... initializers) {
           if (this.disableMBeanRegistry) {
               Registry.disableRegistry();
           }
   
           Tomcat tomcat = new Tomcat();
           File baseDir = this.baseDirectory != null ? this.baseDirectory : this.createTempDir("tomcat");
           tomcat.setBaseDir(baseDir.getAbsolutePath());
           Connector connector = new Connector(this.protocol);
           connector.setThrowOnFailure(true);
           tomcat.getService().addConnector(connector);
           this.customizeConnector(connector);
           tomcat.setConnector(connector);
           tomcat.getHost().setAutoDeploy(false);
           this.configureEngine(tomcat.getEngine());
           Iterator var5 = this.additionalTomcatConnectors.iterator();
   
           while(var5.hasNext()) {
               Connector additionalConnector = (Connector)var5.next();
               tomcat.getService().addConnector(additionalConnector);
           }
   
           this.prepareContext(tomcat.getHost(), initializers);
           return this.getTomcatWebServer(tomcat);
       }
   //...
       protected TomcatWebServer getTomcatWebServer(Tomcat tomcat) {
           //调用TomcatWebServer类的有参构造器
       //如果配置的端口号>=0，创建Tomcat容器
           return new TomcatWebServer(tomcat, this.getPort() >= 0);
       }
   //...
   ```

7. 分析TomcatWebServer类的有参构造器：

   ```java
   public TomcatWebServer(Tomcat tomcat, boolean autoStart) {
       this.monitor = new Object();
       this.serviceConnectors = new HashMap();
       Assert.notNull(tomcat, "Tomcat Server must not be null");
       this.tomcat = tomcat;
       this.autoStart = autoStart;
       this.initialize();
       }
   ```

8. 执行initialize()，调用start()方法完成了tomcat容器的启动：

   ```java
   private void initialize() throws WebServerException {
       logger.info("Tomcat initialized with port(s): " + this.getPortsDescription(false));
       Object var1 = this.monitor;
       synchronized(this.monitor) {
           try {
               this.addInstanceIdToEngineName();
               Context context = this.findContext();
               context.addLifecycleListener((event) -> {
                   if (context.equals(event.getSource()) && "start".equals(event.getType())) {
                       this.removeServiceConnectors();
                   }
               });
               //启动tomcat容器
               this.tomcat.start();
               this.rethrowDeferredStartupExceptions();
   ```

**总结**

1. Spring Boot 根据导入的依赖信息，自动创建对应的web服务工厂定制器；
2. web服务工厂定制器组件的后置处理器获取所有类型为web服务工厂定制器的组件（包含实现WebServerFactoryCustomizer接口，自定义的定制器组件），依次调用customize()定制接口，定制Servlet容器配置；
3. 嵌入式的Servlet容器工厂创建tomcat容器，初始化并启动容器。

#### 使用外置的Servlet容器

嵌入式Servlet容器：应用打成可执行的jar

- 优点：简单、便携；
- 缺点：默认不支持JSP、优化定制比较复杂（使用定制器【ServerProperties、自定义WebServerFactoryCustomizer】，自己编写嵌入式Servlet容器的创建工厂【ConfigurableServletWebServerFactory】）；

外置的Servlet容器：外面安装Tomcat---应用war包的方式打包；

**步骤**

1. 必须创建一个war项目；

2. 将嵌入式的Tomcat指定为provided；

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
   </dependency>
   ```

3. 必须编写一个SpringBootServletInitializer的子类，并调用configure方法

   ```java
   public class ServletInitializer extends SpringBootServletInitializer {
   
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
          //传入SpringBoot应用的主程序
         return application.sources(SpringBootWebJspApplication.class);
      }
   
   }
   ```

4. 启动服务器就可以使用

**原理**

jar包：执行SpringBoot主类的main方法，启动IoC容器，创建嵌入式的Servlet容器；

war包：启动服务器，**服务器启动SpringBoot应用**【SpringBootServletInitializer】，启动IoC容器；

servlet3.0规则：

1. 服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例
2. ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名
3. 还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；

流程：

1. 启动Tomcat

2. Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer**

3. SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set<Class<?\>\>；SpringServletContainerInitializer实现ServletContainerInitializer接口的onStartup方法，为这些WebApplicationInitializer类型的类创建实例；

   ```java
   @HandlesTypes(WebApplicationInitializer.class)
   public class SpringServletContainerInitializer implements ServletContainerInitializer {
   @Override
   	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
   			throws ServletException {
   
   		List<WebApplicationInitializer> initializers = new LinkedList<>();
   
   		if (webAppInitializerClasses != null) {
   			for (Class<?> waiClass : webAppInitializerClasses) {
   				// Be defensive: Some servlet containers provide us with invalid classes,
   				// no matter what @HandlesTypes says...
   				if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
   						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
   					try {
   						initializers.add((WebApplicationInitializer)
   								ReflectionUtils.accessibleConstructor(waiClass).newInstance());
   					}
   					catch (Throwable ex) {
   						throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
   					}
   				}
   			}
   		}
   
   		if (initializers.isEmpty()) {
   			servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
   			return;
   		}
   
   		servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
   		AnnotationAwareOrderComparator.sort(initializers);
   		for (WebApplicationInitializer initializer : initializers) {
   			initializer.onStartup(servletContext);
   		}
   	}
   //..
   }
   ```

4. 每一个WebApplicationInitializer都调用自己的onStartup；

5. 相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

6. SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

   ```java
   protected WebApplicationContext createRootApplicationContext(
         ServletContext servletContext) {
       //1、创建SpringApplicationBuilder
      SpringApplicationBuilder builder = createSpringApplicationBuilder();
      StandardServletEnvironment environment = new StandardServletEnvironment();
      environment.initPropertySources(servletContext, null);
      builder.environment(environment);
      builder.main(getClass());
      ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
      if (parent != null) {
         this.logger.info("Root context already created (using as parent).");
         servletContext.setAttribute(
               WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
         builder.initializers(new ParentContextApplicationContextInitializer(parent));
      }
      builder.initializers(
            new ServletContextApplicationContextInitializer(servletContext));
      builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
       
       //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
      builder = configure(builder);
       
       //使用builder创建一个Spring应用
      SpringApplication application = builder.build();
      if (application.getSources().isEmpty() && AnnotationUtils
            .findAnnotation(getClass(), Configuration.class) != null) {
         application.getSources().add(getClass());
      }
      Assert.state(!application.getSources().isEmpty(),
            "No SpringApplication sources have been defined. Either override the "
                  + "configure method or add an @Configuration annotation");
      // Ensure error pages are registered
      if (this.registerErrorPageFilter) {
         application.getSources().add(ErrorPageFilterConfiguration.class);
      }
       //启动Spring应用
      return run(application);
   }
   ```

7. Spring的应用就启动并且创建IoC容器

   ```java
   public ConfigurableApplicationContext run(String... args) {
      StopWatch stopWatch = new StopWatch();
      stopWatch.start();
      ConfigurableApplicationContext context = null;
      FailureAnalyzers analyzers = null;
      configureHeadlessProperty();
      SpringApplicationRunListeners listeners = getRunListeners(args);
      listeners.starting();
      try {
         ApplicationArguments applicationArguments = new DefaultApplicationArguments(
               args);
         ConfigurableEnvironment environment = prepareEnvironment(listeners,
               applicationArguments);
         Banner printedBanner = printBanner(environment);
         context = createApplicationContext();
         analyzers = new FailureAnalyzers(context);
         prepareContext(context, environment, listeners, applicationArguments,
               printedBanner);
          
          //刷新IOC容器
         refreshContext(context);
         afterRefresh(context, applicationArguments);
         listeners.finished(context, null);
         stopWatch.stop();
         if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                  .logStarted(getApplicationLog(), stopWatch);
         }
         return context;
      }
      catch (Throwable ex) {
         handleRunFailure(context, listeners, analyzers, ex);
         throw new IllegalStateException(ex);
      }
   }
   ```

**==启动Servlet容器，再启动SpringBoot应用==**

### Cors 跨域

解决跨域的方式有很多，例如说，在 Nginx 上配置处理跨域请求的参数。又例如说，项目中有网关服务，统一配置处理。当然，本文既然是 Spring Boot SpringMVC 入门，那么必然只使用 SpringMVC 来解决跨域。目前一共有三种方案：

- 方式一，使用 `@CrossCors` 注解，配置每个 API 接口。
- 方式二，使用 `CorsRegistry.java` 注册表，配置每个 API 接口。
- 方案三，使用 `CorsFilter.java` **过滤器**，处理跨域请求。

**@CrossCors**

`@CrossCors` 注解，添加在类或方法上，标记该类/方法对应接口的 Cors 信息。

`@CrossCors` 注解的**常用属性**，如下：

- `origins` 属性，设置允许的请求来源。`[]` 数组，可以填写多个请求来源。默认值为 `*` 。
- `value` 属性，和 `origins` 属性相同，是它的别名。
- `allowCredentials` 属性，是否允许客户端请求发送 Cookie 。默认为 `false` ，不允许请求发送 Cookie 。
- `maxAge` 属性，本次预检请求的有效期，单位为秒。默认值为 1800 秒。

`@CrossCors` 注解的**不常用属性**，如下：

- `methods` 属性，设置允许的请求方法。`[]` 数组，可以填写多个请求方法。默认值为 `GET` + `POST` 。
- `allowedHeaders` 属性，允许的请求头 Header 。`[]` 数组，可以填写多个请求来源。默认值为 `*` 。
- `exposedHeaders` 属性，允许的响应头 Header 。`[]` 数组，可以填写多个请求来源。默认值为 `*` 。

一般情况下，我们在**每个** Controller 上，添加 `@CrossCors` 注解即可。当然，如果某个 API 接口希望做自定义的配置，可以在 Method 方法上添加。示例如下：

```java
// TestController.java

@RestController
@RequestMapping("/test")
@CrossOrigin(origins = "*", allowCredentials = "true") // 允许所有来源，允许发送 Cookie
public class TestController {

    /**
     * 获得指定用户编号的用户
     *
     * @return 用户
     */
    @GetMapping("/get")
    @CrossOrigin(allowCredentials = "false") // 允许所有来源，不允许发送 Cookie
    public UserVO get() {
        return new UserVO().setId(1).setUsername(UUID.randomUUID().toString());
    }

}
```

在绝大数场合下，肯定是在 Controller 上，添加 `@CrossOrigin(allowCredentials = "true")` 即可。

在前端使用符合 CORS 规范的网络库时，例如说 Vue 常用的网络库 [axios](https://github.com/axios/axios) ，在发起非简单请求时，会自动先先发起 `OPTIONS` “预检”请求，要求服务器确认是否能够这样请求。这样，这个请求就会被 SpringMVC 的拦截器所处理。

此时，如果我们的拦截器认为 `handler` **一定**是 HandlerMethod 类型时，就会导致报错。例如说，艿艿在 UserSecurityInterceptor 拦截器中，会认为 `handler` 是 HandlerMethod 类型，然后通过它获得 `@RequiresLogin` 注解信息，判断是否需要登陆。然后，实际上，此时 `handler` 是 PreFlightHandler 类型，则会导致抛出异常。如下图所示：

![QzFXLR.png](https://s2.ax1x.com/2019/12/22/QzFXLR.png)

此时，有两种解决方案：

- 1）检查每个拦截器的实现，是不是依赖于 `handler` 是 HandlerMethod 的逻辑，进行修复。
- 2）不使用该方案，而是采用 CorsFilter 过滤器，避免 `OPTIONS` 预检查走到拦截器里。

显然，`1）` 的成本略微有点高，所以一般情况下，推荐 `2）` 。

**CorsRegistry**

显然，在每个 Controller 上配置 `@CrossOrigin` 注解，是挺麻烦一事。所以更多的情况下，我们会选择配置 CorsRegistry 注册表。

修改 SpringMVCConfiguration 配置类，增加 CorsRegistry 相关的配置。代码如下：

```java
// SpringMVCConfiguration.java

@Override
public void addCorsMappings(CorsRegistry registry) {
    // 添加全局的 CORS 配置
    registry.addMapping("/**") // 匹配所有 URL ，相当于全局配置
            .allowedOrigins("*") // 允许所有请求来源
            .allowCredentials(true) // 允许发送 Cookie
            .allowedMethods("*") // 允许所有请求 Method
            .allowedHeaders("*") // 允许所有请求 Header
//                .exposedHeaders("*") // 允许所有响应 Header
            .maxAge(1800L); // 有效期 1800 秒，2 小时
}
```

- 这里配置匹配的路径是 `/**` ，从而实现全局 CORS 配置。
- 如果想要配置单个路径的 CORS 配置，可以通过 `CorsRegistry#addMapping(String pathPattern)` 方法，继续往其中添加 CORS 配置。
- 如果想要更安全，可以 `originns` 属性，只填写允许的前端域名地址。

这种方式，一样会存在@CrossCors提供到的坑坑坑坑坑，因为这两者的实现方式是一致的。所以，继续看CorsFilter方案。

**CorsFilter**

在 Spring Web 中，内置提供 CorsFilter 过滤器，实现对 CORS 的处理。**推荐**

配置方式很简单，既然是 Filter 过滤器，就可以采用通过 Bean 的方式，进行配置。所以修改 SpringMVCConfiguration 配置类，增加 CorsFilter 相关的配置。代码如下：

```java
// SpringMVCConfiguration.java

@Bean
public FilterRegistrationBean<CorsFilter> corsFilter() {
    // 创建 UrlBasedCorsConfigurationSource 配置源，类似 CorsRegistry 注册表
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    // 创建 CorsConfiguration 配置，相当于 CorsRegistration 注册信息
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(Collections.singletonList("*")); // 允许所有请求来源
    config.setAllowCredentials(true); // 允许发送 Cookie
    config.addAllowedMethod("*"); // 允许所有请求 Method
    config.setAllowedHeaders(Collections.singletonList("*")); // 允许所有请求 Header
    // config.setExposedHeaders(Collections.singletonList("*")); // 允许所有响应 Header
    config.setMaxAge(1800L); // 有效期 1800 秒，2 小时
    source.registerCorsConfiguration("/**", config);
    // 创建 FilterRegistrationBean 对象
    FilterRegistrationBean<CorsFilter> bean = new FilterRegistrationBean<>(
            new CorsFilter(source)); // 创建 CorsFilter 过滤器
    bean.setOrder(0); // 设置 order 排序。这个顺序很重要哦，为避免麻烦请设置在最前
    return bean;
}
```

最后在推荐一篇文章，感觉写的不错 [《Spring 里那么多种 CORS 的配置方式，到底有什么区》](https://segmentfault.com/a/1190000019485883) 。

### HttpMessageConverter

代码地址：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-1-Web>

在 Spring MVC 中，可以使用 `@RequestBody` 和 `@ResponseBody` 两个注解，分别完成**请求报（内容）到对象**和**对象到响应报文（内容）**的转换，底层这种灵活的消息转换机制，就是 Spring 3.x 中新引入的 [HttpMessageConverter](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/http/converter/HttpMessageConverter.java) ，即消息转换器机制。

示例的UserController中，我们明明返回的是 UserVO 对象，最后输出给前端时，变成了 JSON 字符串，这就是使用了 [MappingJackson2HttpMessageConverter](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.java) 消息转换器，将 UserVO 对象转换成 JSON 字符串，写回给前端。

在一些业务场景下，前端提交给后端 API 参数，比较复杂，那么可能我们希望能够使用 JSON 的格式，提交给后端 API 接口。此时，我们又可以使用 MappingJackson2HttpMessageConverter 消息转换器，将 JSON 字符串，转换成对应的对象

让我们再来一起看看 HttpMessageConverter 消息转换器接口，代码如下：

```java
public interface HttpMessageConverter<T> {
// 是否能够读取指定的 mediaType 内容类型，转换成对应的 clazz 对象
	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
// 是否能够将 clazz 对象，序列化成 mediaType 内容类型
	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
// 获得 HttpMessageConverter 能够支持的内容类型。
	List<MediaType> getSupportedMediaTypes();
// 读取请求内容，转换成 clazz 对象
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;
// 将 clazz 对象，序列化成 contentType 内容类型，写入到响应。
	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}

```

- 在**请求**时，我们在请求头 `Content-Type` 上，表示请求内容（Request Body）的内容类型。这样，SpringMVC 会从自己的 HttpMessageConverter **数组**中，通过 `#canRead(clazz, mediaType)` 方法，判断是否够读取指定的 `mediaType` 内容类型，转换成对应的 `clazz` 对象。如果可以，则调用 `#read(Class clazz, HttpInputMessage inputMessage)` 方法，读取请求内容，转换成 `clazz` 对象。
- 在**响应**时，我们在请求头 `Accept` 上，表示请求内容（Response Body）的内容类型。这样，SpringMVC 会从自己的 HttpMessageConverter **数组**中，通过 `#canWrite(clazz, mediaType)` 方法，判断是否能够将 `clazz` 对象，序列化成 `mediaType` 内容类型。如果可以，则调用 `#write(contentType, outpu tMessage)` 方法， 将 `clazz` 对象，序列化成 `contentType` 内容类型，写入到响应。

**来支持xml请求格式**

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-23/lab-springmvc-23-02/pom.xml) 文件中，**额外**引入 [`jackson-dataformat-xml`](https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml) 依赖。如下：

```xml
<!-- 引入 jackson 对 xml 的转换器，实现对 XML 的序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

修改 SpringMVCConfiguration 配置类，增加 MappingJackson2XmlHttpMessageConverter 相关的配置，用于 XML 格式的 HttpMessageConverter 消息转换器。代码如下：

```java
    // SpringMVCConfiguration.java

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 增加 XML 消息转换器
        Jackson2ObjectMapperBuilder xmlBuilder = Jackson2ObjectMapperBuilder.xml();
        xmlBuilder.indentOutput(true);
        converters.add(new MappingJackson2XmlHttpMessageConverter(xmlBuilder.build()));
    }

```

在 UserController 类中，我们添加一个 API 接口，新增用户，方便我们 XML/JSON 格式的请求和响应。代码如下：

```java
// UserController.java

@PostMapping(value = "/add",
        // ↓ 增加 "application/xml"、"application/json" ，针对 Content-Type 请求头
        consumes = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE},
        // ↓ 增加 "application/xml"、"application/json" ，针对 Accept 请求头
        produces = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE}
)
public UserVO add(@RequestBody UserVO user) {
    return user;
}
```

- 请求
  - 在接口参数上，我们添加了 `@RequestBody` 注解。
  - 添加 `consumes` 属性，增加 `"application/xml"`、`"application/json"` ，针对 `Content-Type` 请求头。
- 响应
  - 因为 UserController 已经添加了 `@RestController` 注解，我们无需重复给改 API 接口，添加 `@ReponseBody` 注解。
  - 添加 `produces` 属性，增加 `"application/xml"`、`"application/json"` ，针对 `Accept` 请求头。

### json 接口开发

在以前使用 Spring 开发项目，需要提供 json 接口时需要做哪些配置呢

> 1. 添加 jackjson 等相关 jar 包
> 2. 配置 Spring Controller 扫描
> 3. 对接的方法添加 @ResponseBody

就这样我们会经常由于配置错误，导致406错误等等，Spring Boot 如何做呢，只需要类添加 `@RestController` 即可，默认类中的方法都会以 json 的格式返回

```java
public class User {
    private String name;
    private int age;
    private String pass;
    //setter、getter省略
}

@RestController
public class WebController {

    @RequestMapping(name="/getUser", method= RequestMethod.POST)
    public User getUser() {
        User user=new User();
        user.setName("小明");
        user.setAge(12);
        user.setPass("123456");
        return user;
    }
}
```

- @RestController 注解相当于 @ResponseBody ＋ @Controller 合在一起的作用，如果 Web 层的类上使用了 @RestController 注解，就代表这个类中所有的方法都会以 JSON 的形式返回结果，也相当于 JSON 的一种快捷使用方式；
- @RequestMapping(name="/getUser", method= RequestMethod.POST)，以 /getUser 的方式去请求，method= RequestMethod.POST 是指只可以使用 Post 的方式去请求，如果使用 Get 的方式去请求的话，则会报 405 不允许访问的错误。

在 test 包下新建 WebControllerTest 测试类，对 getUser() 方法进行测试。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class WebControllerTest {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new WebController()).build();
    }

    @Test
    public void getUser() throws Exception {

        String responseString = mvc.perform(MockMvcRequestBuilders.post("/getUser"))
                .andReturn().getResponse().getContentAsString();
        System.out.println("result : " + responseString);
    }
}
```

这里的测试代码和上面的稍有区别，“.andReturn().getResponse().getContentAsString(); ”的意思是获取请求的返回信息，并将返回信息转换为字符串，最后将请求的响应结果打印出来。

执行完 Test，返回结果如下：

```
result : {"name":"小明","age":12,"pass":"123456"}
```

说明 Spring Boot 自动将 User 对象转成了 JSON 进行返回。那么如果返回的是一个 list 呢，在 WebController 添加方法 getUsers()，代码如下：

```java
@RequestMapping("/getUsers")
public List<User> getUsers() {
    List<User> users=new ArrayList<User>();
    User user1=new User();
    user1.setName("neo");
    user1.setAge(30);
    user1.setPass("neo123");
    users.add(user1);
    User user2=new User();
    user2.setName("小明");
    user2.setAge(12);
    user2.setPass("123456");
    users.add(user2);
   return users;
}
```

添加测试方法：

```java
@Test
public void getUsers() throws Exception {
    String responseString = mockMvc.perform(MockMvcRequestBuilders.get("/getUsers"))
            .andReturn().getResponse().getContentAsString();
    System.out.println("result : "+responseString);
}
```

执行测试方法，返回内容如下：

```
result : [{"name":"neo","age":30,"pass":"neo123"},{"name":"小明","age":12,"pass":"123456"}]
```

说明不管是对象还是集合或者对象嵌套，Spring Boot 均可以将其转换为 JSON 字符串，这种方式特别适合系统提供接口时使用。

#### jackjson驼峰转下划线

有如下几种方法

1. 通过ObjectMapper设置

   ```java
   mapper.setPropertyNamingStrategy(com.fasterxml.jackson.databind.PropertyNamingStrategy.SNAKE_CASE);
   mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
   ```

2. 通过在application.properties增加如下配置

   ```properties
   spring.jackson.property-naming-strategy=CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES
   ```

   注意事项，当开启@EnableSwagger2注解时候，会报jackjson异常，查看是swagger使用的api比较旧，不支持

3. 采用在实体增加注解实现

   ```java
   //实现驼峰转下划线
   @JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
   public class BaseResultVO {
   }
   ```

   实体继承该类即可。如此不用每个类的字段都注明@jsonproperty注解

#### 整合 Fastjson

在 pom.xml 文件中，额外引入 fastjson 依赖。如下：

```xml
<!-- 引入 Fastjson ，实现对 JSON 的序列化 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
```

修改 SpringMVCConfiguration 配置类，增加 FastJsonHttpMessageConverter 相关的配置。代码如下：

```java
// SpringMVCConfiguration.java

@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    // 创建 FastJsonHttpMessageConverter 对象
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
    // 自定义 FastJson 配置
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    fastJsonConfig.setCharset(Charset.defaultCharset()); // 设置字符集
    fastJsonConfig.setSerializerFeatures(SerializerFeature.DisableCircularReferenceDetect); // 剔除循环引用
    fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
    // 设置支持的 MediaType
    fastJsonHttpMessageConverter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON,
            MediaType.APPLICATION_JSON_UTF8));
    // 添加到 converters 中
    converters.add(0, fastJsonHttpMessageConverter); // 注意，添加到最开头，放在 MappingJackson2XmlHttpMessageConverter 前面
}
```

至此，SpringMVC 整合 Fastjson 已经完成，还是蛮方便的。

### HandlerInterceptor 拦截器

在使用 SpringMVC 的时候，我们可以使用 [HandlerInterceptor](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/HandlerInterceptor.java) ，拦截 SpringMVC 处理请求的过程，自定义前置和处理的逻辑。例如说：

- 日志拦截器，记录请求与响应。这样，我们可以知道每一次请求的参数，响应的结果，执行的时长等等信息。
- 认证拦截器，我们可以解析前端传入的用户标识，例如说 `access_token` 访问令牌，获得当前用户的信息，记录到 ThreadLocal 中。这样，后续的逻辑，只需要通过 ThreadLocal 就可以获取到用户信息。
- 授权拦截器，我们可以通过每个 API 接口需要的授权信息，进行判断，当前请求是否允许访问。例如说，用户是否登陆，是否有该 API 操作的权限等等。
- 限流拦截器，我们可以通过每个 API 接口的限流配置，进行判断，当前请求是否超过允许的请求频率，避免恶意的请求，打爆整个系统。

HandlerInterceptor 接口，定义了三个拦截点。代码如下：

```java
// HandlerInterceptor.java

public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```

首先，我们要普及一个概念。我们的每一个 API 请求，会对应到一个 `handler` 处理器。

然后，我们先来看一段伪代码，看看这三个拦截点和 `handler` 的执行过程，是怎么结合的。代码如下：

```java
// 伪代码
Exception ex = null;
try {
    // 前置处理
    if (!preHandle(request, response, handler)) {
        return;
    }

    // 执行处理器，即执行 API 的逻辑
    handler.execute();

    // 后置处理
    postHandle(request, response, handler);
} catch(Exception exception) {
    // 如果发生了异常，记录到 ex 中
    ex = exception;
} finally {
    afterCompletion(request, response, handler);
}
```

多个 HandlerInterceptor 们，可以组成一个 Chain **拦截器链**。那么，整个执行的过程，就变成：

- 首先，按照 HandlerInterceptor 链的**正序**，执行 `#preHandle(...)` 方法。
- 然后，执行 `handler` 的逻辑处理。
- 之后，按照 HandlerInterceptor 链的**倒序**，执行 `#postHandle(...)` 方法。
- 最后，按照 HandlerInterceptor 链的**倒序**，执行 `#afterCompletion(...)` 方法。

这里，我们先只说了**正常**执行的情况。那么**异常**执行的情况呢？例如说：

- 某个 HandlerInterceptor 执行 `#preHandle(...)` 方法，返回 `false` 的情况。
- `handler` 的逻辑处理，抛出 Exception 异常的情况。
- 某个 HandlerInterceptor 执行 `#afterCompletion(...)` 方法，抛出 Exception 异常的情况。

更多操作可以参考Spring MVC的文章