---
title: Spring boot 自动配置原理和启动原理
date: 2019-06-30 13:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

### 自动配置原理

配置文件到底能写什么?怎么写?自动配置原理;

<!--more-->

可配置的示例:<https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/#data-properties>

SpringBoot启动的时候加载主配置类,开启了自动配置功能 @EnableAutoConfiguration

@EnableAutoConfiguration 作用:

1. 利用AutoConfigurationImportSelector给容器中导入一些组件

2. 可以查看selectImports()方法的内容;

3. List configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置

   ```
   SpringFactoriesLoader.loadFactoryNames()
   扫描所有jar包类路径下META‐INF/spring.factories
   把扫描到的这些文件的内容包装成properties对象
   从properties中获取到EnableAutoConfiguration.class类(类名)对应的值,然后把他们添加在容器META‐INF/spring.factories中
   ```

   将类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中;

   每一个这样的 xxxAutoConfiguration类都是容器中的一个组件,都加入到容器中;用他们来做自动配置;

每一个自动配置类进行自动配置功能;

以HttpEncodingAutoConfiguration(Http编码自动配置)为例解释自动配置原理;

```java
//表示这是一个配置类,以前编写的配置文件一样,也可以给容器中添加组件
@Configuration(proxyBeanMethods = false) 

//启动指定类的ConfigurationProperties功能;将配置文件中对应的值和HttpEncodingProperties绑定起来;并把HttpEncodingProperties加入到ioc容器中
@EnableConfigurationProperties(HttpProperties.class)

//Spring底层@Conditional注解(Spring注解版),根据不同的条件,如果满足指定的条件,整个配置类里面的配置就会生效;判断当前应用是否是web应用,如果是,当前配置类生效
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)

//判断当前项目有没有这个类CharacterEncodingFilter;SpringMVC中进行乱码解决的过滤器;
@ConditionalOnClass(CharacterEncodingFilter.class)

//判断配置文件中是否存在某个配置spring.http.encoding.enabled;如果不存在,判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true,也是默认生效的;
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)

public class HttpEncodingAutoConfiguration {
    //他已经和SpringBoot的配置文件映射了

    	private final HttpProperties.Encoding properties;

    //只有一个有参构造器的情况下,参数的值就会从容器中拿
	public HttpEncodingAutoConfiguration(HttpProperties properties) {
		this.properties = properties.getEncoding();
	}

    //给容器中添加一个组件,这个组件的某些值需要从properties中获取
	@Bean
	@ConditionalOnMissingBean //判断容器没有这个组件
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}

    //给容器中添加本地编码映射bean，默认加载优先级为0，实例化顺序在实现了PriorityOrdered的组件执行完之后
	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}

	private static class LocaleCharsetMappingsCustomizer
			implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {

		private final HttpProperties.Encoding properties;

		LocaleCharsetMappingsCustomizer(HttpProperties.Encoding properties) {
			this.properties = properties;
		}

		@Override
		public void customize(ConfigurableServletWebServerFactory factory) {
			if (this.properties.getMapping() != null) {
				factory.setLocaleCharsetMappings(this.properties.getMapping());
			}
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}

}
```

根据当前不同的条件判断,决定这个配置类是否生效

一但这个配置类生效;这个配置类就会给容器中添加各种组件;这些组件的属性是从对应的properties类中获取的,这些类里面的每一个属性又是和配置文件绑定的;

所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘;配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "spring.http")//从配置文件中获取指定的值和bean的属性进行绑定
public class HttpProperties {
    /**
	 * Configuration properties for http encoding.
	 */
	public static class Encoding {

```

**精髓**:

1. SpringBoot启动会加载大量的自动配置类
2. 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类;
3. 我们再来看这个自动配置类中到底配置了哪些组件;(只要我们要用的组件有,我们就不需要再来配置了)
4. 给容器中自动配置类添加组件的时候,会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值;
5. xxxxAutoConfigurartion:自动配置类;给容器中添加组件
6. xxxxProperties:封装配置文件中相关属性;

#### @Conditional注解

@Conditional派生注解(Spring注解版原生的@Conditional作用)

作用:必须是@Conditional指定的条件成立,才给容器中添加组件,配置配里面的所有内容才生效;

| 件注解                          | Condition处理类           | 实例                                                         | 解释                                                         |
| ------------------------------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @ConditionalOnBean              | OnBeanCondition           | @ConditionalOnBean(DataSource.class)                         | Spring容器中不存在对应的实例生效                             |
| @ConditionalOnMissingBean       | OnBeanCondition           | @ConditionalOnMissingBean(name = "redisTemplate")            | Spring容器中不存在对应的实例生效                             |
| @ConditionalOnSingleCandidate   | OnBeanCondition           | @ConditionalOnSingleCandidate(FilteringNotifier.class)       | Spring容器中是否存在且只存在一个对应的实例，或者虽然有多个但 是指定首选的Bean生效 |
| @ConditionalOnClass             | OnClassCondition          | @ConditionalOnClass(RedisOperations.class)                   | 类加载器中存在对应的类生效                                   |
| @ConditionalOnMissingClass      | OnClassCondition          | @ConditionalOnMissingClass(RedisOperations.class)            | 类加载器中不存在对应的类生效                                 |
| @ConditionalOnExpression        | OnExpressionCondition     | @ConditionalOnExpression(“’${server.host}’==’localhost’”)    | 判断SpEL 表达式成立生效                                      |
| @ConditionalOnJava              | OnJavaCondition           | @ConditionalOnJava(JavaVersion.EIGHT)                        | 指定Java版本符合要求生效                                     |
| @ConditionalOnProperty          | OnPropertyCondition       | @ConditionalOnProperty(prefix = “spring.aop”, name = “auto”, havingValue = “true”, matchIfMissing = true) | 应用环境中的属性满足条件生效                                 |
| @ConditionalOnResource          | OnResourceCondition       | @ConditionalOnResource(resources=”mybatis.xml”)              | 存在指定的资源文件生效                                       |
| @ConditionalOnWebApplication    | OnWebApplicationCondition |                                                              | 当前应用是Web应用生效                                        |
| @ConditionalOnNotWebApplication | OnWebApplicationCondition |                                                              | 当前应用不是Web应用生效                                      |

​    上面的扩展注解我们可以简单的分为以下几类：

- Bean作为条件：@ConditionalOnBean、@ConditionalOnMissingBean、@ConditionalOnSingleCandidate。
- 类作为条件：@ConditionalOnClass、@ConditionalOnMissingClass。
- SpEL表达式作为条件：@ConditionalOnExpression。
- JAVA版本作为条件: @ConditionalOnJava
- 配置属性作为条件：@ConditionalOnProperty。
- 资源文件作为条件：@ConditionalOnResource。
- 是否Web应用作为判断条件：@ConditionalOnWebApplication、@ConditionalOnNotWebApplication。

自动配置类必须在一定的条件下才能生效;

我们怎么知道哪些自动配置类生效;

我们可以通过启用 `debug=true`属性;来让控制台打印自动配置报告,这样我们就可以很方便的知道哪些自动配置类生效;

##### @Conditional自定义

上面介绍每个扩展注解的时候都特意提到了每个注解对应的Condition实现类。其实我们可以仿照这些Condition实现类来实现我们自己的@Conditional注解。下面我们同个两个简单的实例来看下怎么实现自己的@Conditional扩展注解。

1. 判断是否配置指定属性

   注意，和@ConditionalOnProperty不一样哦，@ConditionalOnProperty是判断是否有属性并且判断值是否等于我们指定的值。我们要实现的注解只判断有没有配置属性，不管属性对应的值。

   扩展注解ConditionalOnPropertyExist。指定我们的Condition实现类OnPropertyExistCondition。并且指定两个参数。一个是参数name用于指定属性。另一个参数exist用于指定是判断存在还是不存在。

   ```java
   @Retention(RetentionPolicy.RUNTIME)
   @Target({ElementType.TYPE, ElementType.METHOD})
   @Documented
   @Conditional(OnPropertyExistCondition.class)
   public @interface ConditionalOnPropertyExist {
   
       /**
        * 配置文件里面对应的key
        */
       String name() default "";
   
       /**
        * 是否有配置的时候判断通过
        */
       boolean exist() default true;
   
   }
   ```

   OnPropertyExistCondition类就是简单的判断下属性存在与否。

   ```java
   public class OnPropertyExistCondition implements Condition {
       @Override
       public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
           Map<String, Object> annotationAttributes = annotatedTypeMetadata.getAnnotationAttributes(ConditionalOnPropertyExist.class.getName());
           if (annotationAttributes == null) {
               return false;
           }
           String propertyName = (String) annotationAttributes.get("name");
           boolean values = (boolean) annotationAttributes.get("exist");
           String propertyValue = conditionContext.getEnvironment().getProperty(propertyName);
           if(values) {
               return !StringUtils.isEmpty(propertyValue);
           } else {
               return StringUtils.isEmpty(propertyValue);
           }
       }
   }
   ```

2. 判断是否配置指定属性

   我们简单实现这样一个功能，根据指定的系统加载不同的Bean。

   ```java
   @Retention(RetentionPolicy.RUNTIME)
   @Target({ElementType.TYPE, ElementType.METHOD})
   @Documented
   @Conditional(OnSystemCondition.class)
   public @interface ConditionalOnSystem {
   
       /**
        * 指定系统
        */
       SystemType type() default SystemType.WINDOWS;
   
       /**
        * 系统类型
        */
       enum SystemType {
   
           /**
            * windows系统
            */
           WINDOWS,
   
           /**
            * linux系统
            */
           LINUX,
   
           /**
            * mac系统
            */
           MAC
   
       }
   }
   
   public class OnSystemCondition implements Condition {
   
   
       @Override
       public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
           Map<String, Object> annotationAttributes = metadata.getAnnotationAttributes(ConditionalOnSystem.class.getName());
           if (annotationAttributes == null) {
               return false;
           }
           ConditionalOnSystem.SystemType systemType = (ConditionalOnSystem.SystemType) annotationAttributes.get("type");
           switch (systemType) {
               case WINDOWS:
                   return context.getEnvironment().getProperty("os.name").contains("Windows");
               case LINUX:
                   return context.getEnvironment().getProperty("os.name").contains("Linux ");
               case MAC:
                   return context.getEnvironment().getProperty("os.name").contains("Mac ");
           }
           return false;
       }
   }
   ```

### 启动原理

SpringBoot为我们做的自动配置，确实方便快捷，但是对于新手来说，如果不大懂SpringBoot内部启动原理，以后难免会吃亏。所以这次博主就跟你们一起一步步揭开SpringBoot的神秘面纱，让它不在神秘。

代码地址:<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-2-Async>

我们开发任何一个Spring Boot项目，都会用到如下的启动类

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

从上面代码可以看出，Annotation定义（@SpringBootApplication）和类定义（SpringApplication.run）最为耀眼，所以要揭开SpringBoot的神秘面纱，我们要从这两位开始就可以了。

#### SpringBootApplication背后的秘密

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
//...
}
```

虽然定义使用了多个Annotation进行了原信息标注，但实际上重要的只有三个Annotation：

- @Configuration（@SpringBootConfiguration点开查看发现里面还是应用了@Configuration）
- @EnableAutoConfiguration
- @ComponentScan

所以，如果我们使用如下的SpringBoot启动类，整个SpringBoot应用依然可以与之前的启动类功能对等：

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

每次写这3个比较累，所以写一个@SpringBootApplication方便点。接下来分别介绍这3个Annotation。

#### @Configuration

这里的@Configuration对我们来说不陌生，它就是JavaConfig形式的Spring Ioc容器的配置类使用的那个@Configuration，SpringBoot社区推荐使用基于JavaConfig的配置形式，所以，这里的启动类标注了@Configuration之后，本身其实也是一个IoC容器的配置类。

- 表达形式层面：任何一个标注了@Configuration的Java类定义都是一个JavaConfig配置类。

- 注册bean定义层面：任何一个标注了@Bean的方法，其返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id。
- 表达依赖注入关系层面：如果一个bean的定义依赖其他bean,则直接调用对应的JavaConfig类中依赖bean的创建方法就可以了。

#### @ComponentScan

@ComponentScan这个注解在Spring中很重要，它对应XML配置中的元素，@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。

我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

> 注：所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages。

#### @EnableAutoConfiguration

个人感觉@EnableAutoConfiguration这个Annotation最为重要，所以放在最后来解读，大家是否还记得Spring框架提供的各种名字为@Enable开头的Annotation定义？比如@EnableScheduling、@EnableCaching、@EnableMBeanExport等，@EnableAutoConfiguration的理念和做事方式其实一脉相承，简单概括一下就是，**借助@Import的支持，收集和注册特定场景相关的bean定义**。

- @EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。
- @EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器。

而@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器，仅此而已！

@EnableAutoConfiguration作为一个复合Annotation,其自身定义关键信息如下：

```java
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    //...
}
```

其中，最关键的要属@Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。就像一只“八爪鱼”一样

借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以大功告成！

![1.png](https://i.loli.net/2019/07/12/5d27f54a0052420821.png)

**自动配置幕后英雄：SpringFactoriesLoader详解**

SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。

```java
public abstract class SpringFactoriesLoader {
    //...
    public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        ...
    }


    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        ....
    }
}
```

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类

@EnableAutoConfiguration自动配置的魔法骑士就变成了：**从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。**

#### 深入探索SpringApplication执行流程

SpringApplication的run方法的实现是我们本次旅程的主要线路，该方法的主要流程大体可以归纳如下：

1. 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：

   - 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
   - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
   - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
   - 推断并设置main方法的定义类。

   ```java
   	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
   		this.resourceLoader = resourceLoader;
   		Assert.notNull(primarySources, "PrimarySources must not be null");
           //保存主配置类
   		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
   		this.webApplicationType = WebApplicationType.deduceFromClasspath();
           //从类路径下找到META‐INF/spring.factories配置的所有ApplicationContextInitializer;然后保存起来
   		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
           //从类路径下找到ETA‐INF/spring.factories配置的所有ApplicationListener
   		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
           //从多个配置类中找到有main方法的主配置类
   		this.mainApplicationClass = deduceMainApplicationClass();
   	}
   ```

2. SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。

   ```java
   public ConfigurableApplicationContext run(String... args) {
   		StopWatch stopWatch = new StopWatch();
   		stopWatch.start();
   		ConfigurableApplicationContext context = null;
   		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
   		configureHeadlessProperty();
       //获取SpringApplicationRunListeners;从类路径下META‐INF/spring.factories
   		SpringApplicationRunListeners listeners = getRunListeners(args);
       //回调所有的获取SpringApplicationRunListener.starting()方法
   		listeners.starting();
   		try {
               //封装命令行参数
   			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
   			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
               //创建环境完成后回调SpringApplicationRunListener.environmentPrepared();表示环境准备完成
   			configureIgnoreBeanInfo(environment);
               
   			Banner printedBanner = printBanner(environment);
               //创建ApplicationContext;决定创建web的ioc还是普通的ioc
   			context = createApplicationContext();
   			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
   					new Class[] { ConfigurableApplicationContext.class }, context);
               //准备上下文环境;将environment保存到ioc中;而且applyInitializers();
   //applyInitializers():回调之前保存的所有的ApplicationContextInitializer的initialize方法
   //回调所有的SpringApplicationRunListener的contextPrepared();
   			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
     //prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded();
               
               //刷新容器;ioc容器初始化(如果是web应用还会创建嵌入式的Tomcat);Spring注解版
               //扫描,创建,加载所有组件的地方;(配置类,组件,自动配置)
   			refreshContext(context);
   
   			afterRefresh(context, applicationArguments);
   			stopWatch.stop();
   			if (this.logStartupInfo) {
   				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
   			}
               //回调所有的SpringApplicationRunListener的started();
   
   			listeners.started(context);
          //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
               //ApplicationRunner先回调,CommandLineRunner再回调
   			callRunners(context, applicationArguments);
   		}
   		catch (Throwable ex) {
   			handleRunFailure(context, ex, exceptionReporters, listeners);
   			throw new IllegalStateException(ex);
   		}
   
   		try {
   			listeners.running(context);
   		}
   		catch (Throwable ex) {
   			handleRunFailure(context, ex, exceptionReporters, null);
   			throw new IllegalStateException(ex);
   		}
   		return context;
   	}
   ```

3. 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。

4. 遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。

5. 如果SpringApplication的showBanner属性被设置为true，则打印banner。

6. 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

7. ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

8. 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。

9. 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

10. 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

11. 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

12. 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

13. 正常情况下，遍历执行SpringApplicationRunListener的finished()方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished()方法，只不过这种情况下会将异常信息一并传入处理）

去除事件通知点后，整个流程如下：

![1.jpg](https://i.loli.net/2019/07/12/5d27f5c5e3be925647.jpg)

### 事件监听机制

配置在META-INF/spring.factories

1. ApplicationContextInitializer

   ```java
   public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
       @Override
       public void initialize(ConfigurableApplicationContext applicationContext) {
           System.out.println("ApplicationContextInitializer...initialize..." + applicationContext);
       }
   }
   ```

2. SpringApplicationRunListener

   ```java
   public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {
       public HelloSpringApplicationRunListener(SpringApplication application, String[] args) {
   
       }
   
       @Override
       public void starting() {
           System.out.println("SpringApplicationRunListener...starting...");
       }
   
       @Override
       public void environmentPrepared(ConfigurableEnvironment environment) {
           Object o = environment.getSystemProperties().get("os.name");
           System.out.println("SpringApplicationRunListener...environmentPrepared.." + o);
       }
   
       @Override
       public void contextPrepared(ConfigurableApplicationContext context) {
           System.out.println("SpringApplicationRunListener...contextPrepared...");
       }
   
       @Override
       public void contextLoaded(ConfigurableApplicationContext context) {
           System.out.println("SpringApplicationRunListener...contextLoaded...");
       }
   
       @Override
       public void started(ConfigurableApplicationContext context) {
           System.out.println("SpringApplicationRunListener...started..." + context);
       }
   
       @Override
       public void running(ConfigurableApplicationContext context) {
           System.out.println("SpringApplicationRunListener...running..." + context);
       }
   }
   ```

3. 配置(META-INF/spring.factories)

   ```
   org.springframework.context.ApplicationContextInitializer=com.hyp.learn.jpa.listener.HelloApplicationContextInitializer
   org.springframework.boot.SpringApplicationRunListener=\
     com.hyp.learn.jpa.listener.HelloSpringApplicationRunListener
   ```

只需要放在ioc容器中

1. ApplicationRunner

   ```java
   @Component
   public class HelloApplicationRunner implements ApplicationRunner {
       @Override
       public void run(ApplicationArguments args) throws Exception {
           System.out.println("ApplicationRunner...run....");
       }
   }
   ```

2. CommandLineRunner

   ```java
   @Component
   public class HelloCommandLineRunner implements CommandLineRunner {
       @Override
       public void run(String... args) throws Exception {
           System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
       }
   }
   
   ```

### 自定义Starters

starter:

1. 这个场景需要使用到的依赖是什么?
2. 如何编写自动配置

pom依赖

```xml
<!--指定生成属性配置提示 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

相关配置类

```java
@Configuration//指定这个类是一个配置类
@ConditionalOnXXX //在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter //指定自动配置类的顺序,需要在META‐INF/spring.factories中声明的配置，才能指定加载顺序
@Bean //给容器中添加组件
@ConfigurationPropertie //结合相关xxxProperties类来绑定相关的配置
@EnableConfigurationProperties //让xxxProperties生效加入到容器中
```

配置文件：

```properties
#自动配置类要能加载
#将需要启动就加载的自动配置类,配置在META‐INF/spring.factories

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
```

**模式**

- 启动器只用来做依赖导入;
- 专门来写一个自动配置模块;
- 启动器依赖自动配置;别人只需要引入启动器(starter)
- mybatis-spring-boot-starter;自定义启动器名-spring-boot-starter

![lYy0ts.png](https://s2.ax1x.com/2020/01/02/lYy0ts.png)

源码查看：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-10-starter>

### 解决项目启动时初始化资源

在我们实际工作中，总会遇到这样需求，在项目启动的时候需要做一些初始化的操作，比如初始化线程池，提前加载好加密证书等。今天就给大家介绍一个 Spring Boot 神器，专门帮助大家解决项目启动初始化资源操作。

这个神器就是 `CommandLineRunner`，`CommandLineRunner` 接口的 `Component` 会在所有 `Spring Beans `都初始化之后，`SpringApplication.run() `之前执行，非常适合在应用程序启动之初进行一些数据初始化的工作。

接下来我们就运用案例测试它如何使用，在测试之前在启动类加两行打印提示，方便我们识别 `CommandLineRunner` 的执行时机。

```
@SpringBootApplication
public class CommandLineRunnerApplication {
	public static void main(String[] args) {
		System.out.println("The service to start.");
		SpringApplication.run(CommandLineRunnerApplication.class, args);
		System.out.println("The service has started.");
	}
}
```

接下来我们直接创建一个类继承 `CommandLineRunner` ，并实现它的 `run()` 方法。

```
@Component
public class Runner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The Runner start to initialize ...");
    }
}
```

我们在 `run()` 方法中打印了一些参数来看出它的执行时机。完成之后启动项目进行测试：

```
...
The service to start.

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.0.RELEASE)
...
2018-04-21 22:21:34.706  INFO 27016 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-21 22:21:34.710  INFO 27016 --- [           main] com.neo.CommandLineRunnerApplication     : Started CommandLineRunnerApplication in 3.796 seconds (JVM running for 5.128)
The Runner start to initialize ...
The service has started.
```

根据控制台的打印信息我们可以看出 `CommandLineRunner` 中的方法会在 Spring Boot 容器加载之后执行，执行完成后项目启动完成。

如果我们在启动容器的时候需要初始化很多资源，并且初始化资源相互之间有序，那如何保证不同的 `CommandLineRunner` 的执行顺序呢？Spring Boot 也给出了解决方案。那就是使用 `@Order` 注解。

我们创建两个 `CommandLineRunner` 的实现类来进行测试：

第一个实现类：

```
@Component
@Order(1)
public class OrderRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner1 start to initialize ...");
    }
}
```

第二个实现类：

```
@Component
@Order(2)
public class OrderRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner2 start to initialize ...");
    }
}
```

添加完成之后重新启动，观察执行顺序：

```
...
The service to start.

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.0.RELEASE)
...
2018-04-21 22:21:34.706  INFO 27016 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-21 22:21:34.710  INFO 27016 --- [           main] com.neo.CommandLineRunnerApplication     : Started CommandLineRunnerApplication in 3.796 seconds (JVM running for 5.128)
The OrderRunner1 start to initialize ...
The OrderRunner2 start to initialize ...
The Runner start to initialize ...
The service has started.
```

通过控制台的输出我们发现，添加 `@Order` 注解的实现类最先执行，并且`@Order()`里面的值越小启动越早。

在实践中，使用`ApplicationRunner`也可以达到相同的目的，两着差别不大。看来使用 Spring Boot 解决初始化资源的问题非常简单。