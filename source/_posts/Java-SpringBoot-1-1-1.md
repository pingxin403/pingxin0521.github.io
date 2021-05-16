---
title: Spring boot Spring MVC （一）
date: 2019-06-30 14:20:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

Web开发是我们平时开发中至关重要的，这里就来介绍一下Spring Boot对Web开发的支持。

<!--more-->

Spring Boot 集成了参数校验、内嵌容器、Restful、JSON 等内容，这些技术在我们进行 Web 开发中必不可少。Spring Boot 对这些技术进行了包装，让我们在 Web 项目开发过程中非常容易地使用这些技术，因此使用方法跟 spring MVC中的方法类似，本文将根据spring boot来讲一些用法，更多框架或者工具用法还请参考其他博文。

代码地址：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-1-Web>

可以说Spring boot是对spring MVC 的解决方案支持，降低了开发 Web 项目的技术难度。

Spring Boot提供了spring-boot-starter-web为Web开发予以支持，spring-boot-starter-web为我们提供了嵌入的Tomcat以及Spring MVC的依赖。

一个好的项目结构会让你开发少一些问题，特别是Spring Boot中启动类要放在root package下面，我的web工程项目结构如下：

![2.jpg](https://i.loli.net/2019/07/12/5d27f6caedb0326959.jpg)

- root package结构：`com.dudu`
- 应用启动类`Application.java`置于root package下，这样使用@ComponentScan注解的时候默认就扫描当前所在类的package
- 实体（Entity）置于`com.dudu.domain`包下
- 逻辑层（Service）置于`com.dudu.service`包下
- controller层（web）置于`com.dudu.controller层`包下
- static可以用来存放静态资源
- templates用来存放默认的模板配置路径

Spring Web MVC框架（通常简称为”Spring MVC”）是一个富”模型，视图，控制器”的web框架。

Spring MVC允许你创建特定的[@Controller](https://github.com/Controller)或[@RestController](https://github.com/RestController) beans来处理传入的HTTP请求。

示例：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {
    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }
    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }
    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }
}
```

#### 静态文件

默认情况下，Spring  Boot从classpath下一个叫/static（/public，/resources或/META-INF/resources）的文件夹或从ServletContext根目录提供静态内容。这使用了Spring   MVC的ResourceHttpRequestHandler，所以你可以通过添加自己的WebMvcConfigurerAdapter并覆写addResourceHandlers方法来改变这个行为（加载静态文件）。

在一个单独的web应用中，容器默认的servlet是开启的，如果Spring决定不处理某些请求，默认的servlet作为一个回退（降级）将从ServletContext根目录加载内容。大多数时候，这不会发生（除非你修改默认的MVC配置），因为Spring总能够通过DispatcherServlet处理请求。

```java
    
//org.springframework.boot.autoconfigure.web.ResourceProperties
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
    //可以设置和静态资源有关的参数,缓存时间等
}

//org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
//自动配置
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    @Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
            //静态资源文件夹映射
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
    //配置欢迎页映射
    		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			return welcomePageHandlerMapping;
		}
    private Optional<Resource> getWelcomePage() {
			String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
			return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
		}

		private Resource getIndexHtml(String location) {
			return this.resourceLoader.getResource(location + "index.html");
		}
 //....   
}
    

```

1. 所有 `/webjars/**` ，都去 classpath:/META-INF/resources/webjars/ 找资源；
   webjars：以jar包的方式引入静态资源

2. `"/**"` 访问当前项目的任何资源，都去（静态资源的文件夹）找映射

   ```
   "classpath:/META‐INF/resources/",
   "classpath:/resources/",
   "classpath:/static/",
   "classpath:/public/"
   "/"：当前项目的根路径
   
   localhost:8080/pingxin === 去静态资源文件夹里面找pingxin
   静态资源默认访问路径是：META-INF/resources > resources > static > public
   
   ```

3. 欢迎页； 静态资源文件夹下的所有index.html页面；被`"/**"`映射；

   ```
   localhost:8080/ 找index页面
   ```

4. 所有的 `**/favicon.ico` 都是在静态资源文件下找

**自定义默认网页图标**

在不修改SpringBoot项目图标的时候，往往默认是一个小树叶图标,在有些时候需要修改图标，不想用它的默认图标，下面是我自己修改的步骤，亲测可行

首先，把SpringBoot项目默认访问图片设置为false
在application.properties中设置关闭Favicon

```
spring.mvc.favicon.enable=false 
```

然后，在resources文件夹下新建static文件夹，把favicon.ico文件拷贝进去

这里有可以把图片转为ico格式网站https://tool.lu/favicon/，有多种不同大小，选择合适的一个就可以了。

到这里，默认访问首页的话图标会变成你设置的图标，但是访问其他网页出现无图的情况，这个时候，在每个页面加入这段代码，最好在**head**标签之间添加

```
<link rel="icon" type="image/x-icon" href="/favicon.ico">
```

一般情况下，页面都是集成的，有index.html等页面组合到一起，找到一个共有的，只要添加一次就可以了，这样在访问每个页面都会看到设置成自己的图标了。

#### WebJars

WebJars 是一个很神奇的东西，可以让大家以 Jar 包的形式来使用前端的各种框架、组件。

WebJars 是将客户端（浏览器）资源（JavaScript，Css等）打成 Jar 包文件，以对资源进行统一依赖管理。WebJars 的 Jar 包部署在 Maven 中央仓库上。

1. 将静态资源版本化，更利于升级和维护。
2. 剥离静态资源，提高编译速度和打包效率。
3. 实现资源共享，有利于统一前端开发。

我们在开发 Java web 项目的时候会使用像 Maven，Gradle 等构建工具以实现对 Jar 包版本依赖管理，以及项目的自动化管理，但是对于 JavaScript，Css 等前端资源包，我们只能采用拷贝到 webapp 下的方式，这样做就无法对这些资源进行依赖管理。那么 WebJars 就提供给我们这些前端资源的 Jar 包形势，我们就可以进行依赖管理。

我们看下`Jquery`的`webjars`，借此来了解下`webjars`包的目录结构。以下是基于`jquery-3.3.1.jar`

```
META-INF
    └─maven
        └─org.webjars.bower
            └─jquery
                └─pom.properties
                └─pom.xml
    └─resources
        └─webjars
            └─jquery
                └─3.3.1
                       └─(静态文件及源码)
```

所以可以看出，静态文件存放规则：`META-INF/resources/webjars/${name}/${version}`。这点官网也有说明的

**引入**

1. [WebJars主官网](http://www.webjars.org/bower) 查找对于的组件，比如 JQuery

   ```xml
   <!-- 加入后,可以在访问时不加版本号 -->
   <!-- https://mvnrepository.com/artifact/org.webjars/webjars-locator -->
   <dependency>
       <groupId>org.webjars</groupId>
       <artifactId>webjars-locator</artifactId>
       <version>0.38</version>
   </dependency>
   
   <!--引入前端 WebJars Jquery-->
   <dependency>
       <groupId>org.webjars</groupId>
       <artifactId>jquery</artifactId>
       <version>3.3.1</version>
   </dependency>
   ```

2. 快速访问：http://localhost:8080/web/webjars/jquery/jquery.min.js

   快速访问：http://localhost:8080/webjars/jquery/3.3.1/jquery.js

3. 页面引入

   ```html
   <script type="text/javacript" src="@{/webjars/jquery/jquery.js}" ></script>
   ```

#### 模板引擎

Spring Boot支持多种模版引擎包括：

- FreeMarker
- Groovy
- Thymeleaf(官方推荐)
- Mustache

JSP技术Spring Boot官方是不推荐的，原因有三：

1. tomcat只支持war的打包方式，不支持可执行的jar。
2. Jetty 嵌套的容器不支持jsp
3. Undertow
4. 创建自定义error.jsp页面不会覆盖错误处理的默认视图，而应该使用自定义错误页面

当你使用上述模板引擎中的任何一个，它们默认的模板配置路径为：`src/main/resources/templates`。当然也可以修改这个路径，具体如何修改，可在后续各模板引擎的配置属性中查询并修改。

#### [Thymeleaf](https://www.thymeleaf.org)

Thymeleaf是一款用于渲染XML/XHTML/HTML5内容的模板引擎。类似JSP，Velocity，FreeMaker等，它也可以轻易的与Spring   MVC等Web框架进行集成作为Web应用的模板引擎。与其它模板引擎相比，Thymeleaf最大的特点是能够直接在浏览器中打开并正确显示模板页面，而不需要启动整个Web应用。它的功能特性如下：

- Spring MVC中@Controller中的方法可以直接返回模板名称，接下来Thymeleaf模板引擎会自动进行渲染
- 模板中的表达式支持Spring表达式语言（Spring EL)
- 表单支持，并兼容Spring MVC的数据绑定与验证机制
- 国际化支持

Spring官方也推荐使用Thymeleaf,所以本篇代码整合就使用Thymeleaf来整合。

```xml
<!--引入thymeleaf模板引擎-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

```

spring-boot-starter-thymeleaf会自动包含spring-boot-starter-web，所以我们就不需要单独引入web依赖了。

**Thymeleaf的使用与语法**

先看看ThymeleafProperties类中的方法

```java
//org.springframework.boot.autoconfigure.thymeleaf.ThymeleafProperties
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    private Charset encoding;
    private boolean cache;
    private Integer templateResolverOrder;
    private String[] viewNames;
    private String[] excludedViewNames;
    private boolean enableSpringElCompiler;
    private boolean enabled;
    private final ThymeleafProperties.Servlet servlet;
    private final ThymeleafProperties.Reactive reactive;

```

只要我们把HTML页面放在classpath:/templates/，Thymeleaf就能自动渲染；

踩的坑：

 由于我Controller中注解给的是`@RestController`，模板引擎不会渲染，只会返回JSON格式，后改为`@Controller`，才能正常渲染。

**编写controller**

```java
@Controller
@RequestMapping("/learn")
public class LearnResourceController {
    @RequestMapping("/")
    public ModelAndView index(){
        List<LearnResouce> learnList =new ArrayList<LearnResouce>();
        LearnResouce bean =new LearnResouce("官方参考文档","Spring Boot Reference Guide","http://docs.spring.io/spring-boot/docs/1.5.1.RELEASE/reference/htmlsingle/#getting-started-first-application");
        learnList.add(bean);
        bean =new LearnResouce("官方SpriongBoot例子","官方SpriongBoot例子","https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples");
        learnList.add(bean);
        bean =new LearnResouce("龙国学院","Spring Boot 教程系列学习","http://www.roncoo.com/article/detail/125488");
        learnList.add(bean);
        bean =new LearnResouce("嘟嘟MD独立博客","Spring Boot干货系列 ","http://tengj.top/");
        learnList.add(bean);
        bean =new LearnResouce("后端编程嘟","Spring Boot教程和视频 ","http://www.toutiao.com/m1559096720023553/");
        learnList.add(bean);
        bean =new LearnResouce("程序猿DD","Spring Boot系列","http://www.roncoo.com/article/detail/125488");
        learnList.add(bean);
        bean =new LearnResouce("纯洁的微笑","Sping Boot系列文章","http://www.ityouknow.com/spring-boot");
        learnList.add(bean);
        bean =new LearnResouce("CSDN——小当博客专栏","Sping Boot学习","http://blog.csdn.net/column/details/spring-boot.html");
        learnList.add(bean);
        bean =new LearnResouce("梁桂钊的博客","Spring Boot 揭秘与实战","http://blog.csdn.net/column/details/spring-boot.html");
        learnList.add(bean);
        bean =new LearnResouce("林祥纤博客系列","从零开始学Spring Boot ","http://412887952-qq-com.iteye.com/category/356333");
        learnList.add(bean);
        ModelAndView modelAndView = new ModelAndView("/index");
        modelAndView.addObject("learnList", learnList);
        return modelAndView;
    }
}
```

**编写html**

引入依赖后就在默认的模板路径`src/main/resources/templates`下编写模板文件即可完成。这里我们新建一个index.html:

```html
<!DOCTYPE html>
<!-- 导入thymeleaf的名称空间，引入后页面就会提示thymeleaf语法 -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>learn Resources</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>

<div style="text-align: center;margin:0 auto;width: 1000px; ">
<table width="100%" border="1" cellspacing="1" cellpadding="0">
    <tr>
        <td>作者</td>
        <td>教程名称</td>
        <td>地址</td>
    </tr>
    <!--/*@thymesVar id="learnList" type=""*/-->
    <tr th:each="learn : ${learnList}">
        <td th:text="${learn.author}"></td>
        <td th:text="${learn.title}"></td>
        <td><a th:href="${learn.url}" target="_blank">点我</a></td>
    </tr>
</table>
</div>
</body>
</html>
```

> 注：通过xmlns:th=”http://www.thymeleaf.org“ 命令空间，将静态页面转换为动态的视图，需要进行动态处理的元素将使用“th:”前缀。

ok,代码都写好了，让我们看对比下直接打开index.html和启动工程后访问<http://localhost:8080/learn/> 看到的效果，Thymeleaf做到了不破坏HTML自身内容的数据逻辑分离。

更多语法规则参考其他相关博文或者参考：<https://waylau.gitbooks.io/thymeleaf-tutorial/>。

##### Thymeleaf的默认参数配置

在application.properties中可以配置thymeleaf模板解析器属性

```properties
# THYMELEAF (ThymeleafAutoConfiguration)
#开启模板缓存（默认值：true）
spring.thymeleaf.cache=true 
#Check that the template exists before rendering it.
spring.thymeleaf.check-template=true 
#检查模板位置是否正确（默认值:true）
spring.thymeleaf.check-template-location=true
#Content-Type的值（默认值：text/html）
spring.thymeleaf.content-type=text/html
#开启MVC Thymeleaf视图解析（默认值：true）
spring.thymeleaf.enabled=true
#模板编码
spring.thymeleaf.encoding=UTF-8
#要被排除在解析之外的视图名称列表，用逗号分隔
spring.thymeleaf.excluded-view-names=
#要运用于模板之上的模板模式。另见StandardTemplate-ModeHandlers(默认值：HTML5)
spring.thymeleaf.mode=HTML5
#在构建URL时添加到视图名称前的前缀（默认值：classpath:/templates/）
spring.thymeleaf.prefix=classpath:/templates/
#在构建URL时添加到视图名称后的后缀（默认值：.html）
spring.thymeleaf.suffix=.html
#Thymeleaf模板解析器在解析器链中的顺序。默认情况下，它排第一位。顺序从1开始，只有在定义了额外的TemplateResolver Bean时才需要设置这个属性。
spring.thymeleaf.template-resolver-order=
#可解析的视图名称列表，用逗号分隔
spring.thymeleaf.view-names=
```

#### Spring MVC自动配置

Spring Boot为Spring MVC提供适用于多数应用的自动配置功能。在Spring默认基础上，自动配置添加了以下特性：

1. 引入ContentNegotiatingViewResolver和BeanNameViewResolver beans。

   - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向？））

   - ContentNegotiatingViewResolver：组合所有的视图解析器的；

   ==如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来；==

2. 对静态资源的支持，包括对WebJars的支持。

3. 自动注册Converter，GenericConverter，Formatter beans。

   - Converter：转换器； public String hello(User user)：类型转换使用Converter

   - `Formatter` 格式化器； 2017.12.17===Date；

     ```java
     @Bean
     @ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")//在文件中配置日期格式化的规则
     public Formatter<Date> dateFormatter() {
     return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化组件
     }
     //==自己添加的格式化器转换器，我们只需要放在容器中即可==
     ```

4. 对HttpMessageConverters的支持。

   HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json；

   `HttpMessageConverters` 是从容器中确定；获取所有的HttpMessageConverter；

   ==自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）==

5. 自动注册MessageCodeResolver。定义错误代码生成规则

6. 对静态index.html的支持。

7. 对自定义Favicon的支持。

如果想全面控制Spring  MVC，你可以添加自己的@Configuration，并使用@EnableWebMvc对其注解。如果想保留Spring Boot  MVC的特性，并只是添加其他的MVC配置(拦截器，formatters，视图控制器等)，你可以添加自己的WebMvcConfigurerAdapter类型的@Bean（不使用@EnableWebMvc注解）,具体拦截器等配置后续文章会解析。

**扩展**

编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurationSupport {

    @Override
    protected void addViewControllers(ViewControllerRegistry registry) {
//        super.addViewControllers(registry);
        //浏览器发送/cyy，请求来到success页面
        registry.addViewController("/pingxin").setViewName("success");
    }
}
```

原理：

1. WebMvcAutoConfiguration是SpringMVC的自动配置类

2. 在做其他自动配置时会导入；@Import(**EnableWebMvcConfiguration**.class)

   ```java
       @Configuration
       public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
         private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
   
        //从容器中获取所有的WebMvcConfigurer
         @Autowired(required = false)
         public void setConfigurers(List<WebMvcConfigurer> configurers) {
             if (!CollectionUtils.isEmpty(configurers)) {
                 this.configurers.addWebMvcConfigurers(configurers);
                   //一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
                   @Override
                // public void addViewControllers(ViewControllerRegistry registry) {
                 //    for (WebMvcConfigurer delegate : this.delegates) {
                  //       delegate.addViewControllers(registry);
                  //   }
                 }
             }
       }
   ```

3. 容器中所有的WebMvcConfigurer都会一起起作用；

4. 我们的配置类也会被调用；

效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

##### 全面接管SpringMVC

SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置；所有的SpringMVC的自动配置都失效了

我们需要在配置类中添加@EnableWebMvc即可；

```java
//使用WebMvcConfigurer可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /atguigu 请求来到 success
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```

原理：

为什么@EnableWebMvc自动配置就失效了；

1. @EnableWebMvc的核心

   ```
   @Import(DelegatingWebMvcConfiguration.class)
   public @interface EnableWebMvc {
   ```

2. ```
   @Configuration
   public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
   ```

3. ```java
   
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnWebApplication(type = Type.SERVLET)
   @ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
    //容器中没有这个组件的时候，这个自动配置类才生效
   @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
   @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
   @AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
   		ValidationAutoConfiguration.class })
   public class WebMvcAutoConfiguration {
   
   }
   ```

4. @EnableWebMvc将WebMvcConfigurationSupport组件导入进来；

5. 导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能；

##### 如何修改SpringBoot的默认配置

```
1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置

3）、在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置
```

##### 默认访问首页

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
//@EnableWebMvc   不要接管SpringMVC
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
            }
        };
        return adapter;
    }
}

```

##### 国际化

一般的Spring做国际化的步骤：

1. 编写国际化配置文件；
2. 使用ResourceBundleMessageSource管理国际化资源文件
3. 在页面使用fmt:message取出国际化内容

SpringBoot中做国际化的步骤：

1. 编写国际化配置文件，抽取页面需要显示的国际化消息

   [![lP6qaV.md.png](https://s2.ax1x.com/2019/12/24/lP6qaV.md.png)](https://imgchr.com/i/lP6qaV)

2. SpringBoot自动配置好了管理国际化资源文件的组件；

   ```java
   @ConfigurationProperties(prefix = "spring.messages")
   public class MessageSourceAutoConfiguration {
       
       /**
        * Comma-separated list of basenames (essentially a fully-qualified classpath
        * location), each following the ResourceBundle convention with relaxed support for
        * slash based locations. If it doesn't contain a package qualifier (such as
        * "org.mypackage"), it will be resolved from the classpath root.
        */
       private String basename = "messages";  
       //我们的配置文件可以直接放在类路径下叫messages.properties；
       
       @Bean
       public MessageSource messageSource() {
           ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
           if (StringUtils.hasText(this.basename)) {
               //设置国际化资源文件的基础名（去掉语言国家代码的）
               messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(
                       StringUtils.trimAllWhitespace(this.basename)));
           }
           if (this.encoding != null) {
               messageSource.setDefaultEncoding(this.encoding.name());
           }
           messageSource.setFallbackToSystemLocale(this.fallbackToSystemLocale);
           messageSource.setCacheSeconds(this.cacheSeconds);
           messageSource.setAlwaysUseMessageFormat(this.alwaysUseMessageFormat);
           return messageSource;
       }
   ```

3. 去页面获取国际化的值；
   先设置项目的编码为UTF-8:File->Setting->File Encodings，记得勾选 Transparent xxx；

   ![lPcSM9.png](https://s2.ax1x.com/2019/12/24/lPcSM9.png)

   ```html
   <!DOCTYPE html>
   <html lang="en"  xmlns:th="http://www.thymeleaf.org">
       <head>
           <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
           <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
           <meta name="description" content="">
           <meta name="author" content="">
           <title>Signin Template for Bootstrap</title>
           <!-- Bootstrap core CSS -->
           <link href="asserts/css/bootstrap.min.css" th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" rel="stylesheet">
           <!-- Custom styles for this template -->
           <link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
       </head>
   
       <body class="text-center">
           <form class="form-signin" action="dashboard.html">
               <img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
               <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
               <label class="sr-only" th:text="#{login.username}">Username</label>
               <input type="text" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
               <label class="sr-only" th:text="#{login.password}">Password</label>
               <input type="password" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="">
               <div class="checkbox mb-3">
                   <label>
                   <input type="checkbox" value="remember-me"/> [[#{login.remember}]]
           </label>
               </div>
               <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
               <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
               <a class="btn btn-sm">中文</a>
               <a class="btn btn-sm">English</a>
           </form>
   
       </body>
   
   </html>
   ```

   这样的做的效果：根据浏览器语言设置的信息切换了国际化；

   原理：

   ```java
   //国际化Locale（区域信息对象）；LocaleResolver（获取区域信息对象）；
           @Bean
           @ConditionalOnMissingBean
           @ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
           public LocaleResolver localeResolver() {
               if (this.mvcProperties
                       .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
                   return new FixedLocaleResolver(this.mvcProperties.getLocale());
               }
               AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
               localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
               return localeResolver;
           }
   //默认的就是根据请求头带来的区域信息获取Locale进行国际化
   ```

4. 点击链接切换国际化

   ```java
   /**
    * 可以在链接上携带区域信息
    */
   public class MyLocaleResolver implements LocaleResolver {
       
       @Override
       public Locale resolveLocale(HttpServletRequest request) {
           String l = request.getParameter("l");
           Locale locale = Locale.getDefault();
           if(!StringUtils.isEmpty(l)){
               String[] split = l.split("_");
               locale = new Locale(split[0],split[1]);
           }
           return locale;
       }
   
       @Override
       public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
   
       }
   }
   
    @Bean
       public LocaleResolver localeResolver(){
           return new MyLocaleResolver();
       }
   }
   
   ```

#### JSP

使用SpringBoot官方不推荐的jsp,虽然难度有点大，但是玩起来还是蛮有意思的。

先来看看整体的框架结构，跟前面介绍Thymeleaf的时候差不多，只是多了webapp这个用来存放jsp的目录，静态资源还是放在resources的static下面。

```xml
<!--WEB支持-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--jsp页面使用jstl标签-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>

<!--用于编译jsp-->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```

要支持jsp，需要在application.properties中配置返回文件的路径以及类型

```
spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
```

这里指定了返回文件类型为jsp,路径是在/WEB-INF/jsp/下面。

**控制类**

上面步骤有了，这里就开始写控制类，直接上简单的代码，跟正常的springMVC没啥区别：

```java
@Controller
@RequestMapping("/learn")
public class LearnResourceController {
    @RequestMapping("")
    public ModelAndView index(){
        List<LearnResouce> learnList =new ArrayList<LearnResouce>();
        LearnResouce bean =new LearnResouce("官方参考文档","Spring Boot Reference Guide","http://docs.spring.io/spring-boot/docs/1.5.1.RELEASE/reference/htmlsingle/#getting-started-first-application");
        learnList.add(bean);
        bean =new LearnResouce("官方SpriongBoot例子","官方SpriongBoot例子","https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples");
        learnList.add(bean);
        bean =new LearnResouce("龙国学院","Spring Boot 教程系列学习","http://www.roncoo.com/article/detail/125488");
        learnList.add(bean);
        bean =new LearnResouce("嘟嘟MD独立博客","Spring Boot干货系列 ","http://tengj.top/");
        learnList.add(bean);
        bean =new LearnResouce("后端编程嘟","Spring Boot教程和视频 ","http://www.toutiao.com/m1559096720023553/");
        learnList.add(bean);
        bean =new LearnResouce("程序猿DD","Spring Boot系列","http://www.roncoo.com/article/detail/125488");
        learnList.add(bean);
        bean =new LearnResouce("纯洁的微笑","Sping Boot系列文章","http://www.ityouknow.com/spring-boot");
        learnList.add(bean);
        bean =new LearnResouce("CSDN——小当博客专栏","Sping Boot学习","http://blog.csdn.net/column/details/spring-boot.html");
        learnList.add(bean);
        bean =new LearnResouce("梁桂钊的博客","Spring Boot 揭秘与实战","http://blog.csdn.net/column/details/spring-boot.html");
        learnList.add(bean);
        bean =new LearnResouce("林祥纤博客系列","从零开始学Spring Boot ","http://412887952-qq-com.iteye.com/category/356333");
        learnList.add(bean);
        ModelAndView modelAndView = new ModelAndView("/index");
        modelAndView.addObject("learnList", learnList);
        return modelAndView;
    }
}
```

**jsp页面编写**

```jsp
<body style="background-image: none;">
<div class="body_wrap">
    <div class="container">
        <div class="alert alert-success text-center" role="alert">Sptring Boot学习资源大奉送，爱我就关注嘟嘟公众号：嘟爷java超神学堂</div>
        <table class="table table-striped table-bordered">
            <tr>
                <td>作者</td>
                <td>教程名称</td>
                <td>地址</td>
            </tr>
            <c:forEach var="learn" items="${learnList}">
                <tr class="text-info">
                    <td th:text="${learn.author}">嘟嘟MD</td>
                    <td th:text="${learn.title}">SPringBoot干货系列</td>
                    <td><a href="#" th:href="${learn.url}" class="btn btn-search btn-green" target="_blank"><span>点我</span></a>
                    </td>
                </tr>
            </c:forEach>
        </table>
    </div>
</div>
</body>
```

启动类不变还是最简单的

**内嵌Tomcat容器运行项目**

基本配置好了就可以启动项目，通过<http://localhost:8080/learn> 访问

##### 内嵌Tomcat属性配置

关于Tomcat的偶有属性都在org.springframework.boot.autoconfigure.web.ServerProperties配置类中做了定义，我们只需在application.properties配置属性做配置即可。通用的Servlet容器配置都已”server”左右前缀，而Tomcat特有配置都以”server.tomcat”作为前缀。下面举一些常用的例子。

**配置Servlet容器**：

```properties
#配置程序端口，默认为8080
server.port= 8080
#用户绘画session过期时间，以秒为单位
server.session.timeout=
# 配置默认访问路径，默认为/
server.servlet.context-path=
```

**配置Tomcat：**

```properties
# 配置Tomcat编码,默认为UTF-8
server.tomcat.uri-encoding=UTF-8
# 配置最大线程数
server.tomcat.max-threads=1000
```

#### 错误处理机制

默认效果：对浏览器返回一个错误页面，如果是其他客户端，默认响应一个json数据

原理：可以参照`ErrorMvcAutoConfiguration` : 错误处理的自动配置类；

给容器中添加了以下组件：

1. DefaultErrorAttributes：帮我们在页面共享信息;

   ```java
   //ErrorMvcAutoConfiguration.java
       @Bean
       @ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
       public DefaultErrorAttributes errorAttributes() {
           return new DefaultErrorAttributes();
       }
   
   // DefaultErrorAttributes.java
       @Override
       public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
               boolean includeStackTrace) {
           Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
           errorAttributes.put("timestamp", new Date());
           addStatus(errorAttributes, requestAttributes);
           addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
           addPath(errorAttributes, requestAttributes);
           return errorAttributes;
       }
   ```

2. BasicErrorController: 处理默认的 /error请求

   ```java
   //ErrorMvcAutoConfiguration.java
       @Bean
       @ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
       public BasicErrorController basicErrorController(ErrorAttributes errorAttributes) {
           return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
                   this.errorViewResolvers);
       }
   
   //BasicErrorController.java 
   @Controller
   @RequestMapping("${server.error.path:${error.path:/error}}")
   public class BasicErrorController extends AbstractErrorController {
     
       //产生html类型的数据；浏览器发送的请求来到这个方法处理
       @RequestMapping(produces = "text/html")
       public ModelAndView errorHtml(HttpServletRequest request,
               HttpServletResponse response) {
           HttpStatus status = getStatus(request);
           Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
                   request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
           response.setStatus(status.value());
           //去哪个页面作为错误页面；包含页面地址和页面内容
           ModelAndView modelAndView = resolveErrorView(request, response, status, model);
           return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
       }
   
       ////产生json数据，其他客户端来到这个方法处理；
       @RequestMapping
       @ResponseBody
       public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
           Map<String, Object> body = getErrorAttributes(request,
                   isIncludeStackTrace(request, MediaType.ALL));
           HttpStatus status = getStatus(request);
           return new ResponseEntity<Map<String, Object>>(body, status);
       }
   ```

3. ErrorPageCustomizer

   ```java
   @Value("${error.path:/error}")
       private String path = "/error";  //系统出现错误以后来到error请求进行处理；（web.xml注册的错误页面规则）
   ```

4. DefaultErrorViewResolver

   ```java
   //ErrorMvcAutoConfiguration.java        
           @Bean
           @ConditionalOnBean(DispatcherServlet.class)
           @ConditionalOnMissingBean
           public DefaultErrorViewResolver conventionErrorViewResolver() {
               return new DefaultErrorViewResolver(this.applicationContext,
                       this.resourceProperties);
           }
   
   
   //DefaultErrorViewResolver.java
       @Override
       public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
               Map<String, Object> model) {
           ModelAndView modelAndView = resolve(String.valueOf(status), model);
           if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
               modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
           }
           return modelAndView;
       }
   
       private ModelAndView resolve(String viewName, Map<String, Object> model) {
           String errorViewName = "error/" + viewName;
           TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
                   .getProvider(errorViewName, this.applicationContext);
           if (provider != null) {
               return new ModelAndView(errorViewName, model);
           }
           return resolveResource(errorViewName, model);
       }   
   ```

步骤：

一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被**BasicErrorController**处理； 响应页面；去哪个页面是由**DefaultErrorViewResolver**解析得到的；

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    //所有的ErrorViewResolver得到ModelAndView
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

##### 定制错误的页面

1. **有模板引擎的情况下；error/状态码;** 【将错误页面命名为  错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到  对应的页面；

   我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）；

   页面能获取的信息:

   ```
   timestamp：时间戳
   status：状态码
   error：错误提示
   exception：异常对象
   message：异常消息
   errors：JSR303数据校验的错误都在这里
   ```

2. 没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找

3. 以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

##### 定制错误的json数据

1. 自定义异常处理&返回定制json数据；

   ```java
   //MyExceptionHandler.java
   @ControllerAdvice
   public class MyExceptionHandler {
   
       @ResponseBody
       @ExceptionHandler(UserNotExistException.class)
       public Map<String,Object> handleException(Exception e){
           Map<String,Object> map = new HashMap<>();
           map.put("code","user.notexist");
           map.put("message",e.getMessage());
           return map;
       }
   }
   ```

2. 转发到/error进行自适应响应效果处理

   ```java
       @ExceptionHandler(UserNotExistException.class)
       public String handleException(Exception e, HttpServletRequest request){
            //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
           /**
            * Integer statusCode = (Integer) request
            .getAttribute("javax.servlet.error.status_code");
            */
           request.setAttribute("javax.servlet.error.status_code",500);
           Map<String,Object> map = new HashMap<>();
           map.put("code","user.notexist");
           map.put("message",e.getMessage());
           //转发到/error
           return "forward:/error";
       }
   ```

3. 将我们的定制数据携带出去；

   出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）；

   - 完全来编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，放在容器中；
   - 页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到；容器中DefaultErrorAttributes.getErrorAttributes()；默认进行数据处理的；

   自定义ErrorAttributes

   ```java
   @Component
   public class MyErrorAttributes extends DefaultErrorAttributes {
       @Override
       public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
           Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
           map.put("company","pccw");
   
           //我们的异常处理器携带的数据
   //        Map<String, Object> ext = (Map<String, Object>) requestAttributes.getAttribute("ext",0);
   //        map.put("ext",ext);
           return map;
       }
   }
   ```

   最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容

#### 统一返回值

1. RequestBodyAdvice 是对请求响应的json串进行处理，一般使用环境是处理解密的json串。

   需要在controller层中调用方法上加入@RequestMapping和@RequestBody;后者若没有，则RequestBodyAdvice实现方法不执行。当然可以添加`@ControllerAdvice(annotations = RestController.class)`，选则作用的注解。

2. ResponseBodyAdvice是对请求响应后对结果的处理，一般使用环境是处理加密的json串。

   需要在controller层中调用方法上加入@RequstMapping和@ResponseBody;后者若没有，则ResponseBodyAdvice实现方法不执行。当然可以添加`@ControllerAdvice(annotations = RestController.class)`，选则作用的注解。

这里使用ResponseBodyAdvice统一返回值

在我们提供后端 API 给前端时，我们需要告前端，这个 API 调用结果是否成功：

- 如果**成功**，成功的数据是什么。后续，前端会取数据渲染到页面上。
- 如果**失败**，失败的原因是什么。一般，前端会将原因弹出提示给用户。

这样，我们就需要有统一的返回结果，而不能是每个接口自己定义自己的风格。一般来说，统一的全局返回信息如下：

- 成功时，返回**成功的状态码** + **数据**。
- 失败时，返回**失败的状态码** + **错误提示**。

在标准的 RESTful API 的定义，是推荐使用 [HTTP 响应状态码](https://zh.wikipedia.org/wiki/HTTP状态码) 返回状态码。一般来说，我们实践很少这么去做，主要有如下原因：

- 业务返回的错误状态码很多，HTTP 响应状态码无法很好的映射。例如说，活动还未开始、订单已取消等等。
- 国内开发者对 HTTP 响应状态码不是很了解，可能只知道 200、403、404、500 几种常见的。这样，反倒增加学习成本。

所以，实际项目在实践时，我们会将状态码放在 Response Body **响应内容**中返回。

在全局统一返回里，我们至少需要定义三个字段：

- `code`：状态码。无论是否成功，必须返回。
  - 成功时，状态码为 0 。
- `data`：数据。成功时，返回该字段。
- `message`：错误提示。失败时，返回该字段。

返回结果封装类RestResult：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult {

    public static Integer CODE_SUCCESS = CommonCode.SUCCESS.getCode();

    /**
     * 错误码
     */
    private Integer code;
    /**
     * 错误提示
     */
    private String message;
    /**
     * 返回数据
     */
    private Object data;

    public static CommonResult of(Object data) {
        return new CommonResult(CommonCode.SUCCESS.getCode(), CommonCode.SUCCESS.getMessage(), data);
    }

    public static CommonResult of(Exception e) {

        return new CommonResult(CommonCode.SYS_ERROR.getCode(), e.getMessage(), Collections.emptyMap());
    }

    public static CommonResult of(Exception e, Integer code) {

        return new CommonResult(code, e.getMessage(), Collections.emptyMap());
    }

    public static CommonResult error(Integer code, String message) {
        Assert.isTrue(!CODE_SUCCESS.equals(code), "code 必须是错误的！");
        CommonResult result = new CommonResult();
        result.code = code;
        result.message = message;
        return result;
    }

    public static CommonResult success(Object data) {
        return CommonResult.of(data);
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isSuccess() { // 方便判断是否成功
        return CODE_SUCCESS.equals(code);
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isError() { // 方便判断是否失败
        return !isSuccess();
    }
}
```

统一异常封装类：

```java
/**
 * @description: 对于需要给用户展示的异常，统一抛出此异常
 */
public class BusinessException extends RuntimeException implements Serializable {

    /**
     * 错误码
     */
    private CommonCode code;

    private Map<String, ? extends Object> errors;

    public BusinessException(CommonCode code, Map<String, Object> errors) {
        super(code.getMessage());
        this.code = code;
        this.errors = errors;
    }

    public BusinessException(String message, CommonCode code, Map<String, ? extends Object> errors) {
        super(message);
        this.code = code;
        this.errors = errors;
    }

    public BusinessException(String message, Throwable cause, CommonCode code, Map<String, ? extends Object> errors) {
        super(message, cause);
        this.code = code;
        this.errors = errors;
    }

    public BusinessException(Throwable cause, CommonCode code, Map<String, ? extends Object> errors) {
        super(cause.getMessage(), cause);
        this.code = code;
        this.errors = errors;
    }

    public BusinessException(CommonCode code) {
        super(code.getMessage());
        this.code = code;
        this.errors = new HashMap<>();
    }

    public BusinessException(String message, CommonCode code) {
        super(message);
        this.code = code;
        this.errors = new HashMap<>();
    }

    public BusinessException(String message, Throwable cause, CommonCode code) {
        super(message, cause);
        this.code = code;
    }

    public BusinessException(Throwable cause, CommonCode code) {
        super(cause.getMessage(), cause);
        this.code = code;
    }

    public CommonCode getCode() {
        return this.code;
    }

    @Override
    public String toString() {
        Map<String, Object> map = new HashMap<>();
        map.put("code", this.getCode().getCode());
        map.put("message", this.getMessage());
        return JSONObject.toJSONString(map, SerializerFeature.WriteMapNullValue);
    }

    public Map<String, ? extends Object> getErrors() {
        return this.errors;
    }

    public Integer getCodeId() {
        return this.code.getCode();
    }
}

```

异常枚举：

```java
@Getter
public enum CommonCode {


    // ========== 系统级别 ==========
    SUCCESS(0, "成功"),
    SYS_ERROR(2001001000, "服务端发生异常"),
    MISSING_REQUEST_PARAM_ERROR(2001001001, "参数缺失"),

    // ========== 用户模块 ==========
    USER_NOT_FOUND(1001002000, "用户不存在"),

    // ========== 订单模块 ==========

    // ========== 商品模块 ==========
    ;

    /**
     * 错误码
     */
    private int code;
    /**
     * 错误提示
     */
    private String message;


    CommonCode(int code, String message) {
        this.code = code;
        this.message = message;
    }
}
```

统一返回值处理：

```java
@ControllerAdvice(annotations = RestController.class)
public class GlobalResponseBodyHandler implements ResponseBodyAdvice<Object> {

    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType mediaType, Class converterType, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        if (null == body) {
            return CommonResult.of(Collections.emptyMap());
        } else if (!(body instanceof CommonResult)) {
            CommonResult responseResult = CommonResult.of(body);
            //因为handler处理类的返回类型是String，为了保证一致性，这里需要将ResponseResult转回去
            if (body instanceof String) {
                return JSON.toJSONString(responseResult);
            }
            return responseResult;
        }


        return body;
    }

}
```

统一异常处理：

```java
@ControllerAdvice(annotations = RestController.class)
public class GlobalExceptionHandler {
    private static Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 处理 ServiceException 异常
     */
    @ResponseBody
    @ExceptionHandler(value = ServiceException.class)
    public CommonResult serviceExceptionHandler(HttpServletRequest req, ServiceException ex) {
        logger.debug("[serviceExceptionHandler]", ex);
        // 包装 CommonResult 结果
        return CommonResult.error(ex.getCodeId(), ex.getMessage());
    }

    /**
     * 处理 MissingServletRequestParameterException 异常
     * <p>
     * SpringMVC 参数不正确
     */
    @ResponseBody
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    public CommonResult missingServletRequestParameterExceptionHandler(HttpServletRequest req, MissingServletRequestParameterException ex) {
        logger.debug("[missingServletRequestParameterExceptionHandler]", ex);
        // 包装 CommonResult 结果
        return CommonResult.error(CommonCode.MISSING_REQUEST_PARAM_ERROR.getCode(),
                CommonCode.MISSING_REQUEST_PARAM_ERROR.getMessage());
    }

    /**
     * 处理其它 Exception 异常
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public CommonResult exceptionHandler(HttpServletRequest req, Exception e) {
        // 记录异常日志
        logger.error("[exceptionHandler]", e);
        // 返回 ERROR CommonResult
        return CommonResult.error(CommonCode.SYS_ERROR.getCode(),
                CommonCode.SYS_ERROR.getMessage());
    }

}
```

#### 加解密请求

Springboot中RequestBodyAdvice与ResponseBodyAdvice文档地址： [阅读文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/RequestBodyAdviceAdapter.html)

对json串进行RSA加密解密的方式，自定义一注解，进行判断是否需要对json参数进行加密解密的处理，即是否执行。代码如下：

```java
 import org.springframework.web.bind.annotation.Mapping;
 import java.lang.annotation.*;
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Mapping
@Documented
/*@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)*/
public @interface SecurityParameter {
 
    /**
     * 入参是否解密，默认解密
     */
    boolean inDecode() default true;
 
    /**
     * 出参是否加密，默认加密
     */
    boolean outEncode() default true;
}


@Slf4j
@ControllerAdvice(annotations = RestController.class)
public class DecodeRequestBodyAdvice implements RequestBodyAdvice {
 
    /**
     * 加载配置文件信息
     */
    @Autowired
    private ReadProBean readProBean;
 
    @Override
    public boolean supports(MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        //这里设置成false 它就不会再走这个类了
        return methodParameter.getMethod().isAnnotationPresent(SecurityParameter.class);
    }
 
    @Override
    public HttpInputMessage beforeBodyRead(HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        try {
            log.info("开始对接受值进行解密操作");
            // 定义是否解密
            boolean encode = false;
            //获取注解配置的包含和去除字段
            SecurityParameter serializedField = methodParameter.getMethodAnnotation(SecurityParameter.class);
            //入参是否需要解密
            encode = serializedField.inDecode();
            log.info("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行解密!");
            return new MyHttpInputMessage(httpInputMessage, encode);
        } catch (IOException e) {
            e.printStackTrace();
            log.error("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行解密出现异常：" + e.getMessage());
            throw new RuntimeException("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行解密出现异常：" + e.getMessage());
        }
    }
 
    @Override
    public Object afterBodyRead(Object o, HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        return o;
    }
 
    @Override
    public Object handleEmptyBody(Object o, HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        return o;
    }
 
 
    //这里实现了HttpInputMessage 封装一个自己的HttpInputMessage
    class MyHttpInputMessage implements HttpInputMessage {
        HttpHeaders headers;
        InputStream body;
 
        public MyHttpInputMessage(HttpInputMessage httpInputMessage, boolean encode) throws IOException {
            this.headers = httpInputMessage.getHeaders();
            // 解密操作
            this.body = decryptBody(httpInputMessage.getBody(), encode);
        }
 
        @Override
        public InputStream getBody() {
            return body;
        }
 
        @Override
        public HttpHeaders getHeaders() {
            return headers;
        }
    }
 
    /**
     * 对流进行解密操作
     *
     * @param in
     * @return
     */
    public InputStream decryptBody(InputStream in, Boolean encode) throws IOException {
        try {
            // 获取 json 字符串
            String bodyStr = IOUtils.toString(in, "UTF-8");
            if (StringUtils.isEmpty(bodyStr)) {
                throw new RuntimeException("无任何参数异常!");
            }
            // 获取 inputData 加密串
            String inputData = JSONObject.fromObject(bodyStr).get(readProBean.getRequestInput()).toString();
 
            // 验证是否为空
            if (StringUtils.isEmpty(inputData)) {
                throw new RuntimeException("参数【" + readProBean.getRequestInput() + "】缺失");
            } else {
 
                // 是否解密
                if (!encode) {
                    log.info("没有解密标识不进行解密!");
                    return IOUtils.toInputStream(inputData, "UTF-8");
                }
 
                // 开始解密
                log.info("对加密串开始解密操作!");
                String decryptBody = null;
                try {
                    decryptBody = RSAUtils.decryptDataOnJava(inputData, readProBean.getPrivateKey());
                    log.info("操作结束!");
                    return IOUtils.toInputStream(decryptBody, "UTF-8");
                } catch (Exception e) {
                    e.printStackTrace();
                    throw new RuntimeException("RSA解密异常：" + e.getMessage());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("获取参数【" + readProBean.getRequestInput() + "】异常：" + e.getMessage());
        }
    }
}



@Slf4j
@ControllerAdvice(annotations = RestController.class)
public class EncodeResponseBodyAdvice implements ResponseBodyAdvice {
 
    /**
     * 加载配置文件信息
     */
    @Autowired
    private ReadProBean readProBean;
 
    @Override
    public boolean supports(MethodParameter methodParameter, Class aClass) {
        //这里设置成false 它就不会再走这个类了
        return methodParameter.getMethod().isAnnotationPresent(SecurityParameter.class);
    }
 
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter methodParameter, MediaType mediaType, Class aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        try {
            log.info("开始对返回值进行加密操作!");
            // 定义是否解密
            boolean encode = false;
            //获取注解配置的包含和去除字段
            SecurityParameter serializedField = methodParameter.getMethodAnnotation(SecurityParameter.class);
            //入参是否需要解密
            encode = serializedField.outEncode();
            log.info("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行加密!");
            return encryptedBody(body, encode);
        } catch (IOException e) {
            e.printStackTrace();
            log.error("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行加密出现异常：" + e.getMessage());
            throw new RuntimeException("对方法method :【" + methodParameter.getMethod().getName() + "】数据进行加密出现异常：" + e.getMessage());
        }
    }
 
    public Object encryptedBody(Object body, Boolean encode) throws IOException {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            String result = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(body);
            if (!encode) {
                log.info("没有加密表示不进行加密!");
                return body;
            }
            log.info("对字符串开始加密!");
            try {
                String outputData = RSAUtils.encryptedDataOnJava(result, readProBean.getPublicKey());
                JSONObject json = new JSONObject();
                json.put(readProBean.getResponseOutput(), outputData);
                log.info("操作结束!");
                return json;
            } catch (Exception e) {
                e.printStackTrace();
                throw new RuntimeException("对返回数据加密出现异常：" + e.getMessage());
            }
 
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new RuntimeException("获取返回值出现异常:" + e.getMessage());
        }
    }
}




```

RSAUtils工具类：<https://github.com/hanyunpeng0521/utils/blob/master/src/main/java/com/hyp/learn/base/util/RSAUtils.java>

测试Controller

```java

@Slf4j
@RestController
public class LoginController {
 
    /**
     * @Author: Mr.Yan
     * @create: 2019/3/27
     * @MethodName: login
     * @Param: [response, request]
     * @Return: net.sf.json.JSONObject
     * @description: 自动对参数加密解密
     */
    @SecurityParameter(inDecode = false,outEncode = false)
    @RequestMapping("/login")
    public ResultModel login( HttpServletResponse response, HttpServletRequest request,
                              @RequestBody(required = false)  Map<String,Object> map){
        log.info("进入login后台");
        System.out.println("输出account:————————————————" + map.get("account"));
        System.out.println("输出password:————————————————" + map.get("password"));
 
        Map<String,Object> resultMap =  new HashMap<>();
        resultMap.put("userid","200");
        return ResultTools.result(0,"",resultMap);
    }
}
```

读取配置文件Bean

```java
@Data
@Component
@ConfigurationProperties(prefix = "rsa")
public class ReadProBean {
 
    /**
     * RSA 加密定义公钥
     */
    private String publicKey;
 
    /**
     * RSA 加密定义私钥
     */
    private String privateKey;
 
    /**
     * 前台发送的加密的json串命名
     */
    private String requestInput;
 
    /**
     * 后台发送的加密的json串命名
     */
    private String responseOutput;
 
  
 
}
```

application.properties文件中配置

```properties
 
# rsa 加密公钥
rsa.publicKey = MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCowwu8ftM+/PPlG5sNFjFtiNG4zsMCRTmfQuMXK6B6YV9Fs1k9a6z6sUVOsqvC2f3QecaOt2obNs8UdPw/EleOcYk/8Z7asVVJcARcYHiQ8NkBUmPyUQKAFiKjVvhtHpCfWZDfurHEQI/rE2mGCB9nMNJcvaSMjK/U0uyEMweqawIDAQAB
# rsa 加密私钥
rsa.privateKey = MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAKjDC7x+0z788+Ubmw0WMW2I0bjOwwJFOZ9C4xcroHphX0WzWT1rrPqxRU6yq8LZ/dB5xo63ahs2zxR0/D8SV45xiT/xntqxVUlwBFxgeJDw2QFSY/JRAoAWIqNW+G0ekJ9ZkN+6scRAj+sTaYYIH2cw0ly9pIyMr9TS7IQzB6prAgMBAAECgYEAmGZi/9sMC5LE8b4HPD8xbbgjpB/bzP4UtjTh/LeiGUI7licLTMMjF9TkQNhq8fCIHC8MVy9dO6w4P0IR1SdMNtfx674b3H6WjSpCXT9B4GVqzv6xiAh1r0wDoE7mJtLiAI5EkPxZm+IEim6D89mVn8/aUjkqkQ7ZLwAu2uvOHKkCQQDSm2rJP3o31pCpEGN2vJMg3Cy2R5woGUejmnnVM6ot8+up0L4z5Q9AmRaEXt1TGyateSt44/yGNEIsRiPYy90VAkEAzSLAwzU68awJmMwDznK44QMdIDGkIweLNJcRD5SO/ne4giQmYnDQTfB4Q48QIiYKp0RwJJRc7PTcxbVF7vJJfwJAWRo16KT5gUw+8bgkTKTlnl5ocEoFsBVZ8Ma3StNL6ZssFjFhdzUu6caa9y/ndXSkPXppQQE74k+Tu4WFPwCpLQJBALfFFXkLe8W7QFGxGwvcvIFfv7zym7+B55RybSdPCBcxe4qjBfwUYpggAC1Nwb9F4y9b4Tbz7pec+RbpQUBBr9MCQEGl3q9ZX/Qh7YFt9bVVHew/WAzJLapCdcrV3mgQQQFHaTvdEGtwYh4t/VwGVFNRHqCN5yS4LlD9YS5sGOf1U3s=
# requestBody json 参数
rsa.requestInput = inputData
# requestBody json 参数
rsa.responseOutput = outputData
```



### 参考

1. <https://www.jianshu.com/p/a23b26b23353>
2. [Springboot中RequestBodyAdvice与ResponseBodyAdvice的正确使用](https://blog.csdn.net/yanmh007/article/details/88871705)