---
title: Spring boot 安全框架
date: 2019-07-03 08:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

Apache Shiro 是一个功能强大、灵活的，开源的安全框架。它可以干净利落地处理身份验证、授权、企业会话管理和加密。

<!--more-->

代码位置：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-19-shiro>

Apache Shiro 的首要目标是易于使用和理解。安全通常很复杂，甚至让人感到很痛苦，但是 Shiro 却不是这样子的。一个好的安全框架应该屏蔽复杂性，向外暴露简单、直观的 API，来简化开发人员实现应用程序安全所花费的时间和精力。

Shiro 能做什么呢？

- 验证用户身份
- 用户访问权限控制，比如：1、判断用户是否分配了一定的安全角色。2、判断用户是否被授予完成某个操作的权限
- 在非 Web 或 EJB 容器的环境下可以任意使用 Session API
- 可以响应认证、访问控制，或者 Session 生命周期中发生的事件
- 可将一个或以上用户安全数据源数据组合成一个复合的用户 “view”(视图)
- 支持单点登录(SSO)功能
- 支持提供“Remember Me”服务，获取用户关联信息而无需登录
  …

等等——都集成到一个有凝聚力的易于使用的 API。

Shiro 致力在所有应用环境下实现上述功能，小到命令行应用程序，大到企业应用中，而且不需要借助第三方框架、容器、应用服务器等。当然 Shiro 的目的是尽量的融入到这样的应用环境中去，但也可以在它们之外的任何环境下开箱即用。

#### Apache Shiro Features 特性

Apache Shiro 是一个全面的、蕴含丰富功能的安全框架。下图为描述 Shiro 功能的框架图：

![1neLbn.png](https://s2.ax1x.com/2020/01/26/1neLbn.png)

Authentication（认证）, Authorization（授权）, Session Management（会话管理）, Cryptography（加密）被 Shiro 框架的开发团队称之为应用安全的四大基石。那么就让我们来看看它们吧：

- **Authentication（认证）：**用户身份识别，通常被称为用户“登录”
- **Authorization（授权）：**访问控制。比如某个用户是否具有某个操作的使用权限。
- **Session Management（会话管理）：**特定于用户的会话管理,甚至在非web 或 EJB 应用程序。
- **Cryptography（加密）：**在对数据源使用加密算法加密的同时，保证易于使用。

还有其他的功能来支持和加强这些不同应用环境下安全领域的关注点。特别是对以下的功能支持：

- Web支持：Shiro 提供的 Web 支持 api ，可以很轻松的保护 Web 应用程序的安全。
- 缓存：缓存是 Apache Shiro 保证安全操作快速、高效的重要手段。
- 并发：Apache Shiro 支持多线程应用程序的并发特性。
- 测试：支持单元测试和集成测试，确保代码和预想的一样安全。
- “Run As”：这个功能允许用户假设另一个用户的身份(在许可的前提下)。
- “Remember Me”：跨 session 记录用户的身份，只有在强制需要时才需要登录。

> 注意： Shiro 不会去维护用户、维护权限，这些需要我们自己去设计/提供，然后通过相应的接口注入给 Shiro

#### High-Level Overview 高级概述

在概念层，Shiro 架构包含三个主要的理念：Subject，SecurityManager和 Realm。下面的图展示了这些组件如何相互作用，我们将在下面依次对其进行描述。

![1nezCT.png](https://s2.ax1x.com/2020/01/26/1nezCT.png)

- Subject：当前用户，Subject 可以是一个人，但也可以是第三方服务、守护进程帐户、时钟守护任务或者其它–当前和软件交互的任何事件。
- SecurityManager：管理所有Subject，SecurityManager 是 Shiro 架构的核心，配合内部安全组件共同组成安全伞。
- Realms：用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。

我们需要实现Realms的Authentication 和 Authorization。其中 Authentication 是用来验证用户身份，Authorization 是授权访问控制，用于对用户进行的操作授权，证明该用户是否允许进行当前操作，如访问某个链接，某个资源文件等。

### 实践

#### 认证流程

1. 对未认证的用户请求进行拦截，跳转到认证页面。
2. 用户通过用户名+密码及其他凭证进行身份认证，认证成功跳转成功页面，认证失败提示相关失败信息。

创建`SecurityManager`安全管理器 > 主体`Subject`提交认证信息 > `SecurityManager`安全管理器认证 >  `SecurityManager`调用`Authenticator`认证器认证 >`Realm`验证

#### 授权流程

Shiro的授权流程跟认证流程类似：

1. 创建`SecurityManager`安全管理器
2. `Subject`主体带授权信息执行授权，请求到`SecurityManager`
3. `SecurityManager`安全管理器调用`Authorizer`授权
4. `Authorizer`结合主体一步步传过来的授权信息与`Realm`中的数据比对，授权

主体进行授权请求有两种方式，一种是编程式，一种是注解式。

1. 编程式：通过`Subject`的`hasRole()`进行角色的校检，通过`isPermitted()`进行权限的校检

2. 注解式：
   首先需要开启Aop注解,在ShiroConfig类中新增如下方法：

   ```java
    /**
        * 开启aop注解支持
        * 即在controller中使用 @RequiresPermissions("user/userList")
        */
       @Bean
       public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager){
           AuthorizationAttributeSourceAdvisor attributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
           //设置安全管理器
           attributeSourceAdvisor.setSecurityManager(securityManager);
           return attributeSourceAdvisor;
       }
   
       @Bean
       @ConditionalOnMissingBean
       public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
           DefaultAdvisorAutoProxyCreator defaultAAP = new DefaultAdvisorAutoProxyCreator();
           defaultAAP.setProxyTargetClass(true);
           return defaultAAP;
       }
   ```

   对于未授权的用户需要进行友好的提示，一般返回特定的403页面，在ShiroConfig类中新增如下方法实现：

   ```java
    /**
        * 处理未授权的异常，返回自定义的错误页面（403）
        * @return
        */
       @Bean
       public SimpleMappingExceptionResolver simpleMappingExceptionResolver() {
           SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
           Properties properties = new Properties();
           /*未授权处理页*/
           properties.setProperty("UnauthorizedException", "403.html");
           resolver.setExceptionMappings(properties);
           return resolver;
       }
   ```

   解释下上面的代码`properties.setProperty("UnauthorizedException","403.html");`用来对抓取到`UnauthorizedException`未授权异常进行处理，重定向到`403.html`页面。我们需要创建403页面`403.html`

   ```xml
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>未授权</title>
   </head>
   <body>
   403,您没有访问权限
   </body>
   </html>
   ```

   然后在Controller使用注解`@RequiresRoles("xxx")`和`@RequiresPermissions("xxx")`进行角色和权限的校检

总结：一般对未授权用户需要统一返回指定403页面的，使用注解更加方便；需要做业务逻辑（比如对未授权的请求记录进行预警等），使用编程式更加方便。

#### 前端的shiro标签

通常前端页面展示需要与用户的权限对等，即只给用户看到他们权限内的内容。`（比如用户"张小黑的猫"，对应角色“cat”,对应权限“sing”和“rap”，那么该用户登录后只展示Cat、sing和rap的按钮）`
 通常解决方式有两种：

 其一：登录后通过读取数据库中角色和权限，获取需要展示的菜单内容，动态的在前端渲染；
 其二：所有内容都在前端写好，通过前端的shiro标签控制对应权限内容部分的渲染。

 `（前端的shiro标签有Jsp标签、Freemarker标签、Thymeleaf标签等，演示用的为thymeleaf标签）`

> Shiro标签说明：
> guest标签：`<shiro:guest></shiro:guest>`，用户没有身份验证时显示相应信息，即游客访问信息。
> user标签：`<shiro:user></shiro:user>`，用户已经身份验证/记住我登录后显示相应的信息。
> authenticated标签:`<shiro:authenticated></shiro:authenticated>`，用户已经身份验证通过，即Subject.login登录成功，不是记住我登录的。
> notAuthenticated标签:`<shiro:notAuthenticated></shiro:notAuthenticated>`，用户已经身份验证通过，即没有调用Subject.login进行登录，包括记住我自动登录的也属于未进行身份验证。
> principal标签:`<shiro: principal/><shiro:principal property="username"/>`，相当`((User)Subject.getPrincipals()).getUsername()`。
> lacksPermission标签：`<shiro:lacksPermission name="org:create"></shiro:lacksPermission>`，如果当前Subject没有权限将显示body体内容。
> hasRole标签：`<shiro:hasRole name="admin"></shiro:hasRole>`，如果当前Subject有角色将显示body体内容。
> hasAnyRoles标签：`<shiro:hasAnyRoles name="admin,user"></shiro:hasAnyRoles>`，如果当前Subject有任意一个角色（或的关系）将显示body体内容。
> lacksRole标签：`<shiro:lacksRole name="abc"></shiro:lacksRole>`，如果当前Subject没有角色将显示body体内容。
> hasPermission标签：`<shiro:hasPermission name="user:create"></shiro:hasPermission>`，如果当前Subject有权限将显示body体内容

1. pom.xml引入相应的依赖

   ```xml
    <!-- 兼容于thymeleaf的shiro -->
           <dependency>
               <groupId>com.github.theborakompanioni</groupId>
               <artifactId>thymeleaf-extras-shiro</artifactId>
               <version>2.0.0</version>
           </dependency>
   ```

2. ShiroConfig中加入配置

   ```java
    /**
        * 用于thymeleaf模板使用shiro标签
        * @return
        */
       @Bean
       public ShiroDialect shiroDialect() {
           return new ShiroDialect();
       }
   ```

注意：使用前现在html标签内引入shiro标签，即`<html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">`

示例：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>测试Thymeleaf的使用</title>
</head>
<body>
<h3 th:text="${name}"></h3>
<hr/>
<div shiro:hasPermission="user:add">
进入用户添加功能： <a href="add">用户添加</a><br/>
</div>
<div shiro:hasPermission="user:update">
进入用户更新功能： <a href="update">用户更新</a><br/>
</div>
<a href="toLogin">登录</a>
</body>
</html>

```

#### 动态权限配置（读取yml文件方式）

在使用时对权限的配置都是写死的，如果后期项目需要维护，更改权限配置，就无可避免的要进行代码的修改。
 比如：现在需要将`cat`角色改为`rabbit`,那么编程式授权需要将`subject.hasRole("cat")`改为`subject.hasRole("rabbit")`，注解式授权需要将`@RequiresRoles("cat")`改为`@RequiresRoles("rabbit")`，使用起来很笨重。

这时候，动态配置权限便显得很有必要了，我们通过读取数据库或者权限的配置文件将权限注入，如果需要修改，我们只需要修改数据库或者修改相关的配置文件即可，无需重新部署项目。

1. 定义关于权限的配置文件

   ```yml
   ##动态权限配置文件
   #List<Map<String, String>>
   permission-config:
     perms:
       - url: /cat
         permission: roles[cat]
       - url: /dog
         permission: roles[dog]
       - url: /sing
         permission: perms[sing]
       - url: /jump
         permission: perms[jump]
       - url: /rap
         permission: perms[rap]
       - url: /basketball
         permission: perms[basketball]
   ```

2. 将配置文件信息的内容转化为`List>`，注入到`ShiroConfig`中。
   新建`PermsMap`类

   ```java
   @Component
   @ConfigurationProperties(prefix = "permission-config")
   public class PermsMap {
   
       private List<Map<String,String>> perms;
   
       public List<Map<String, String>> getPerms() {
           return perms;
       }
   
       public void setPerms(List<Map<String, String>> perms) {
           this.perms = perms;
       }
   }
   ```

   注意：prefix="permission-config"和perms要与.yml文件中的属性对应
3. 修改`ShiroConfig.java`
   先使用`@Autowired`注入`PermsMap`，切记不能使用`new PermsMap ()`

   ```css
      @Autowired
       PermsMap permsMap;
   ```

   更改过滤链

   ```dart
    /**
        * 配置Shiro的Web过滤器，拦截浏览器请求并交给SecurityManager处理
        * @return
        */
       @Bean
       public ShiroFilterFactoryBean webFilter(SecurityManager securityManager){
   
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           //设置securityManager
           shiroFilterFactoryBean.setSecurityManager(securityManager);
   
           //配置拦截链 使用LinkedHashMap,因为LinkedHashMap是有序的，shiro会根据添加的顺序进行拦截
           // Map<K,V> K指的是拦截的url V值的是该url是否拦截
           Map<String,String> filterChainMap = new LinkedHashMap<String,String>(16);
           //配置退出过滤器logout，由shiro实现
           filterChainMap.put("/logout","logout");
           //authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问,先配置anon再配置authc。
           filterChainMap.put("/login","anon");
           // 未授权界面;
           shiroFilterFactoryBean.setUnauthorizedUrl("/403");
   
   //        //权限注入
   //        filterChainMap.put("/cat","roles[cat]");
           
           //动态权限注入
           List<Map<String,String>> perms = permsMap.getPerms();
           perms.forEach(perm->filterChainMap.put(perm.get("url"),perm.get("permission")));
   
           filterChainMap.put("/**", "authc");
   
           //设置默认登录的URL.
           shiroFilterFactoryBean.setLoginUrl("/login");
           System.out.println(filterChainMap);
           shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainMap);
           return shiroFilterFactoryBean;
       }
   ```

   `动态注入要满足格式filterChainMap.put("/cat","roles[cat]");`
    这里面又额外配置了未授权界面`shiroFilterFactoryBean.setUnauthorizedUrl("/403");`对未通过权限验证的界面进行统一跳转403页面的操作,所以需要配置对应的页面跳转。

   ```kotlin
    @GetMapping("/403")
       public String page_403(){
        return "403";
       }
   ```

   更改`AuthorizationController`

   ```kotlin
   @RestController
   public class AuthorizationController {
   
       @GetMapping("/cat")
       public String cat(){
           return "cat";
       }
       @GetMapping("/dog")
       public String dog(){
           return "dog";
       }
       @GetMapping("/sing")
       public String sing(){
           return "sing";
       }
       @GetMapping("/jump")
       public String jump(){
           return "jump";
       }
       @GetMapping("/rap")
       public String rap(){
           return "rap";
       }
       @GetMapping("/basketball")
       public String basketball(){
           return "basketball";
       }
   }
   ```

#### setUnauthorizedUrl("/403")不起作用

SpringBoot中集成Shiro的时候， 配置setUnauthorizedUrl("/403")了，但是不起作用，只会在控制台打印`UnauthorizedException`异常信息

原因：
 Shiro源码中是这样做的：

```tsx
    private void applyUnauthorizedUrlIfNecessary(Filter filter) {
        String unauthorizedUrl = this.getUnauthorizedUrl();
        if(StringUtils.hasText(unauthorizedUrl) && filter instanceof AuthorizationFilter) {
            AuthorizationFilter authzFilter = (AuthorizationFilter)filter;
            String existingUnauthorizedUrl = authzFilter.getUnauthorizedUrl();
            if(existingUnauthorizedUrl == null) {
                authzFilter.setUnauthorizedUrl(unauthorizedUrl);
            }
        }

    }
```

只有perms，roles，ssl，rest，port才是属于AuthorizationFilter，而anon，authcBasic，authc，user是AuthenticationFilter，所以unauthorizedUrl设置后不起作用，只会在控制台打印异常信息。

1. 第一种方式

   ```java
   @Configuration
   public class ExceptionConf {
   
       @Bean
       public SimpleMappingExceptionResolver resolver() {
           SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
           Properties properties = new Properties();
           properties.setProperty("org.apache.shiro.authz.UnauthorizedException", "/403");
           resolver.setExceptionMappings(properties);
           return resolver;
       }
   }
   ```

2. 用spring mvc的统一异常处理类HandlerExceptionResolver

   定义一个类继承HandlerExceptionResolver，然后判断UnauthorizedException异常即可。

   ```java
   public class MyExceptionResolver implements HandlerExceptionResolver {
   
       @Override
       public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
           if (e instanceof UnauthorizedException) {
               ModelAndView mv = new ModelAndView("/403");
               return mv;
           }
           return null;
       }
   }
   ```

   然后注册该bean

   ```java
     // 注册统一异常处理bean
       @Bean
       public MyExceptionResolver myExceptionResolver() {
           return new MyExceptionResolver();
       }
   ```


