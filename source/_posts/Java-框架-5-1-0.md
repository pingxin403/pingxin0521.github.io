---
title: Spring MVC 学习（一）
date: 2019-07-9 16:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

Spring MVC 是一个模型 - 视图 - 控制器（MVC）的Web框架建立在中央前端控制器servlet（DispatcherServlet），它负责发送每个请求到合适的处理程序，使用视图来最终返回响应结果的概念。Spring MVC 是 Spring 产品组合的一部分，它享有 Spring IoC容器紧密结合Spring松耦合等特点，因此它有Spring的所有优点。

<!--more-->

JavaEE体系结构包括四层，从上到下分别是应用层、Web层、业务层、持久层。Struts和SpringMVC是Web层的框架，Spring是业务层的框架，Hibernate和MyBatis是持久层的框架。

#### 为什么要使用SpringMVC？

很多应用程序的问题在于处理业务数据的对象和显示业务数据的视图之间存在紧密耦合，通常，更新业务对象的命令都是从视图本身发起的，使视图对任何业务对象更改都有高度敏感性。而且，当多个视图依赖于同一个业务对象时是没有灵活性的。

SpringMVC是一种基于Java，实现了Web MVC设计模式，请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将Web层进行职责解耦。基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发，SpringMVC也是要简化我们日常Web开发。

#### MVC设计模式

MVC设计模式的任务是将包含业务数据的模块与显示模块的视图解耦。这是怎样发生的？在模型和视图之间引入重定向层可以解决问题。此重定向层是控制器，控制器将接收请求，执行更新模型的操作，然后通知视图关于模型更改的消息。

传统MVC

![2.png](https://i.loli.net/2019/07/13/5d297457e537592246.png)

改进后

![3.png](https://i.loli.net/2019/07/13/5d29745803b1c30263.png)

#### SpringMVC架构

SpringMVC是Spring的一部分。

SpringMVC架构图所下所示

![](https://i.loli.net/2019/07/13/5d297342f3a5892824.png)

**SpringMVC的核心架构：**

![4.png](https://i.loli.net/2019/07/13/5d2974ba5f04119190.png)

具体流程：

1. 首先浏览器发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

2. DispatcherServlet——>HandlerMapping，处理器映射器将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器对象、多个HandlerInterceptor拦截器）对象；

3. DispatcherServlet——>HandlerAdapter，处理器适配器将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；

4. HandlerAdapter——>调用处理器相应功能处理方法，并返回一个ModelAndView对象（包含模型数据、逻辑视图名）；

5. ModelAndView对象（Model部分是业务对象返回的模型数据，View部分为逻辑视图名）——> ViewResolver， 视图解析器将把逻辑视图名解析为具体的View；

6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构；

7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。

#### 入门程序

maven依赖

```xml
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-web</artifactId>
<version>${spring.version}</version>
</dependency>

<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-webmvc</artifactId>
<version>${spring.version}</version>
</dependency>

<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-test</artifactId>
</dependency>
 <dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <scope>provided</scope>
</dependency>
```

web.xml

```xml
<web-app>
  <servlet>
  <!-- 加载前端控制器 -->
  <servlet-name>springmvc</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 
  	   加载配置文件
  	   默认加载规范：
  	   * 文件命名：servlet-name-servlet.xml====springmvc-servlet.xml
  	   * 路径规范：必须在WEB-INF目录下面
  	   修改加载路径：
   -->
   <init-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>classpath:springmvc.xml</param-value>   
   </init-param>
  </servlet>
  
  <servlet-mapping>
  <servlet-name>springmvc</servlet-name>
  <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>

```

springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <beans>
         <context:component-scan base-package="com.hyp.learn"/>
        <!-- 配置映射处理器：根据bean(自定义Controller)的name属性的url去寻找handler；springmvc默认的映射处理器是
        BeanNameUrlHandlerMapping
         -->
        <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>


        <!-- 配置处理器适配器来执行Controlelr ,springmvc默认的是
        SimpleControllerHandlerAdapter
        -->
        <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>

<!-- 配置自定义Controller -->
	<bean id="myController" name="/hello.do" class="org.controller.MyController"></bean>
	

        <!-- 配置sprigmvc视图解析器：解析逻辑试图；
            后台返回逻辑试图：index
            视图解析器解析出真正物理视图：前缀+逻辑试图+后缀====/WEB-INF/jsps/index.jsp
        -->
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/jsps/"></property>
            <property name="suffix" value=".jsp"></property>
        </bean>
    </beans>

</beans>
```

HelloController

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest,
                                      HttpServletResponse httpServletResponse)
            throws Exception {
        ModelAndView mav = new ModelAndView();
        mav.addObject("message", "Hello Spring MVC");
		//返回物理视图
		//mv.setViewName("/WEB-INF/jsps/index.jsp");
		
		//返回逻辑视图
		mv.setViewName("index");        return mav;
    }
}
```

index.jsp，必须添加`isELIgnored="false"`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" isELIgnored="false"%>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
<h2>${message}</h2>
</body>
</html>

```

测试地址：<http://localhost:8080/springmvc/hello>

#### 常用注释

这里不介绍Spring中的注解方法。

**组件型注解**

1. 在SpringMVC 中，控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在SpringMVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。此外Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到。

   @Controller 用于标记在一个类上，使用它标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。单单使用@Controller 标记在一个类上还不能真正意义上的说它就是SpringMVC 的一个控制器类，因为这个时候Spring 还不认识它。那么要如何做Spring 才能认识它呢？这个时候就需要我们把这个控制器类交给Spring 来管理。有两种方式：

   - 在SpringMVC 的配置文件中定义MyController 的bean 对象。

   - 在SpringMVC 的配置文件中告诉Spring 该到哪里去找标记为@Controller 的Controller 控制器。

   ```xml
   <!--方式一-->
   <bean class="com.host.app.web.controller.MyController"/>
   <!--方式二-->
   < context:component-scan base-package = "com.host.app.web" />//路径写到controller的上一层(扫描包详解见下面浅析)
   ```

2. @Component 在类定义之前添加@Component注解，他会被spring容器识别，并转为bean。

3. @Repository 对Dao实现类进行注解 (特殊的@Component)

4. @Service 用于对业务逻辑层进行注解， (特殊的@Component)

以上四种注解都是注解在类上的，被注解的类将被spring初始话为一个bean，然后统一管理。

**请求和参数型注解**

1. @RequestMapping：用于处理请求地址映射，可以作用于类和方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

   - value：定义request请求的映射地址

   - method：定义地request址请求的方式，包括【GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE.】默认接受get请求，如果请求方式和定义的方式不一样则请求无法成功。

   - params：指定request中必须包含某些参数值是，才让该方法处理。

   - headers：定义request请求中必须包含某些指定的请求头，如：RequestMapping(value =  "/something", headers = "content-type=text/*")说明请求中必须要包含"text/html",  "text/plain"这中类型的Content-type头，才是一个匹配的请求。

   - consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

   - produces：指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

   ```java
   @RequestMapping(value="/requestTest.do",params = {"name=sdf"},headers = {"Accept-Encoding=gzip, deflate, br"},method = RequestMethod.GET)
        public String getIndex(){
            System.out.println("请求成功");
            return "index";
        }
   ```

   上面代码表示请求的方式为GET请求，请求参数必须包含name=sdf这一参数，然后请求头中必须有 Accept-Encoding=gzip, deflate, br这个类型头，否则不能访问到该方法。

   这样通过注解就能对一个请求进行约束了。

2. @RequestParam：用于获取传入参数的值

   - value：参数的名称
   - required：定义该传入参数是否必须，默认为true，（和@RequestMapping的params属性有点类似）

   ```java
   @RequestMapping("/requestParams1.do")
       public String requestParams1(@RequestParam(required = false) String name){
           System.out.println("name = "+name);
           return "index";
       }
       @RequestMapping("/requestParams2.do")
       public String requestParams2(@RequestParam(value = "name",required = false) String names){
           System.out.println("name = "+names);
           return "index";
       }
   ```

   两种请入参方式是一样的，显示声明value的名称时，入参参数名和value一样，没有显示声明的话，像第一种方式声明的，入参参数名和函数参数变量名一样。

3. @PathViriable：用于定义路径参数值

   - value：参数的名称
   - required：定义传入参数是否为必须值

   ```java
   @RequestMapping("/{myname}/pathVariable2.do")
       public String pathVariable2(@PathVariable(value = "myname") String name){
           System.out.println("myname = "+name);
           return "index";
       }
   ```

   这个路径声明了{myname}作为路径参数，那么这一段路径将为任意值，@PathVariable将可以根据value获取路径的值。

4. @ResponseBody：作用于方法上，可以将整个返回结果以某种格式返回，如json或xml格式。

   ```java
   @RequestMapping("/{myname}/pathVariable2.do")
   @ResponseBody
   public String pathVariable2(@PathVariable(value = "myname") String name){
       System.out.println("myname = "+name);
       return "index";
   }
   ```

   它返回的不是一个页面，而是把字符串“index”直接在页面打印出来了，这其实和如下代码时类似的。

   ```java
   PrintWriter out = resp.getWriter();
   out.print("index");
   out.flush();
   ```

5. @CookieValue：用于获取请求的Cookie值

   ```java
   @RequestMapping("/requestParams.do")
   public String requestParams(@CookieValue("JSESSIONID") String cookie){
       return "index";
   }
   ```

6. @ModelAttribute：用于把参数保存到model中，可以注解方法或参数，注解在方法上的时候，该方法将在处理器方法执行之前执行，然后把返回的对象存放在 session（前提时要有@SessionAttributes注解）
   或模型属性中，@ModelAttribute(“attributeName”) 在标记方法的时候指定，若未指定，则使用返回类型的类名称（首字母小写）作为属性名称。　

   ```java
   @ModelAttribute("user")
   public UserEntity getUser(){
       UserEntity userEntityr = new UserEntity();
       userEntityr.setUsername("asdf");
       return userEntityr;
   }
   
   @RequestMapping("/modelTest.do")
   public String getUsers(@ModelAttribute("user") UserEntity user){
       System.out.println(user.getUsername());
       return "/index";
   }
   ```

   如上代码中，使用了@ModelAttribute("user")注解，在执行控制器前执行，然后将生成一个名称为user的model数据，在控制器中我们通过注解在参数上的@ModelAttribute获取参数，然后将model应用到控制器中，在jsp页面中我们同样可以使用它，

   ```jsp
   <body>
       ${user.username}
   </body>
   ```

7. @SessionAttributes

   默认情况下Spring MVC将模型中的数据存储到request域中。当一个请求结束后，数据就失效了。如果要跨页面使用。那么需要使用到session。而@SessionAttributes注解就可以使得模型中的数据存储一份到session域中。配合@ModelAttribute("user")使用的时候,会将对应的名称的model值存到session中

   ```java
   @Controller
   @RequestMapping("/test")
   @SessionAttributes(value = {"user","test1"})
   public class LoginController{
       @ModelAttribute("user")
       public UserEntity getUser(){
           UserEntity userEntityr = new UserEntity();
           userEntityr.setUsername("asdf");
           return userEntityr;
       }
   
       @RequestMapping("/modelTest.do")
       public String getUsers(@ModelAttribute("user") UserEntity user ,HttpSession session){
           System.out.println(user.getUsername());
           System.out.println(session.getAttribute("user"));
           return "/index";
       }
   }
   ```

   结合上一个例子的代码，加了@SessionAttributes注解，然后请求了两次，第一次session中不存在属性名为user的值，第二次请求的时候发现session中又有了，这是因为，这是因为第一次请求时，model数据还未保存到session中请求结束返回的时候才保存，在第二次请求的时候已经可以获取上一次的model了

注意：@ModelAttribute("user") UserEntity user获取注解内容的时候，会先查询session中是否有对应的属性值，没有才去查询Model。

#### RequestMapping

**使用 @RequestMapping 来映射 Request 请求与处理器**

方式一、通过常见的类路径和方法路径结合访问controller方法

方式二、使用uri模板

```java
@Controller
@RequestMapping ( "/test/{variable1}" )
public class MyController {

    @RequestMapping ( "/showView/{variable2}" )
    public ModelAndView showView( @PathVariable String variable1, @PathVariable ( "variable2" ) int variable2) {
       ModelAndView modelAndView = new ModelAndView();
       modelAndView.setViewName( "viewName" );
       modelAndView.addObject( " 需要放到 model 中的属性名称 " , " 对应的属性值，它是一个对象 " );
       return modelAndView;
    }
}
```

URI 模板就是在URI 中给定一个变量，然后在映射的时候动态的给该变量赋值。如URI 模板<http://localhost/app/{variable1}/index.html>，这个模板里面包含一个变量variable1 ，那么当我们请求<http://localhost/app/hello/index.html> 的时候，该URL 就跟模板相匹配，只是把模板中的variable1 用hello 来取代。这个变量在SpringMVC 中是使用@PathVariable 来标记的。在SpringMVC 中，我们可以使用@PathVariable 来标记一个Controller 的处理方法参数，表示该参数的值将使用URI 模板中对应的变量的值来赋值。

代码中我们定义了两个URI 变量，一个是控制器类上的variable1 ，一个是showView 方法上的variable2 ，然后在showView 方法的参数里面使用**@PathVariable** 标记使用了这两个变量。所以当我们使用/test/hello/showView/2.do 来请求的时候就可以访问到MyController 的showView 方法，这个时候variable1 就被赋予值hello ，variable2 就被赋予值2 ，然后我们在showView 方法参数里面标注了参数variable1 和variable2 是来自访问路径的path 变量，这样方法参数variable1 和variable2 就被分别赋予hello 和2 。方法参数variable1 是定义为String 类型，variable2 是定义为int 类型，像这种简单类型在进行赋值的时候Spring 是会帮我们自动转换的。

在上面的代码中我们可以看到在标记variable1 为path 变量的时候我们使用的是@PathVariable ，而在标记variable2 的时候使用的是@PathVariable(“variable2”) 。这两者有什么区别呢？第一种情况就默认去URI 模板中找跟参数名相同的变量，但是这种情况只有在使用debug 模式进行编译的时候才可以，而第二种情况是明确规定使用的就是URI 模板中的variable2 变量。当不是使用debug 模式进行编译，或者是所需要使用的变量名跟参数名不相同的时候，就要使用第二种方式明确指出使用的是URI 模板中的哪个变量。

 除了在请求路径中使用URI 模板，定义变量之外，**@RequestMapping 中还支持通配符`“* ”`**。如下面的代码就可以使用/myTest/whatever/wildcard.do 访问到Controller 的testWildcard 方法。如：

```java
@Controller
@RequestMapping ( "/myTest" )
public class MyController {
    @RequestMapping ( "*/wildcard" )
    public String testWildcard() {
       System.out.println( "wildcard------------" );
       return "wildcard" ;
    }  
}
```

当@RequestParam中没有指定参数名称时，Spring 在代码是debug 编译的情况下会默认取更方法参数同名的参数，如果不是debug 编译的就会报错。

**使用 @RequestMapping 的一些高级用法**

1. params属性

   ```java
   @RequestMapping (value= "testParams" , params={ "param1=value1" , "param2" , "!param3" })
       public String testParams() {
          System. out .println( "test Params..........." );
          return "testParams" ;
       }
   ```

   用@RequestMapping 的params 属性指定了三个参数，这些参数都是针对请求参数而言的，它们分别表示参数param1 的值必须等于value1 ，参数param2 必须存在，值无所谓，参数param3 必须不存在，只有当请求/testParams.do 并且满足指定的三个参数条件的时候才能访问到该方法。所以当请求/testParams.do?param1=value1&param2=value2 的时候能够正确访问到该testParams 方法，当请求/testParams.do?param1=value1&param2=value2&param3=value3 的时候就不能够正常的访问到该方法，因为在@RequestMapping 的params 参数里面指定了参数param3 是不能存在的。

2. method属性

   ```java
   @RequestMapping (value= "testMethod" , method={RequestMethod. GET , RequestMethod. DELETE })
       public String testMethod() {
          return "method" ;
       }
   ```

   在上面的代码中就使用method 参数限制了以GET 或DELETE 方法请求/testMethod 的时候才能访问到该Controller 的testMethod 方法。

3. headers属性

   ```java
   @RequestMapping (value= "testHeaders" , headers={ "host=localhost" , "Accept" })
       public String testHeaders() {
          return "headers" ;
       }
   ```

   headers 属性的用法和功能与params 属性相似。在上面的代码中当请求/testHeaders.do 的时候只有当请求头包含Accept 信息，且请求的host 为localhost 的时候才能正确的访问到testHeaders 方法。

**@RequestMapping 标记的处理器方法支持的方法参数和返回类型**

1. 支持的方法参数类型

   - HttpServlet 对象，主要包括HttpServletRequest 、HttpServletResponse 和HttpSession 对象。 这些参数Spring 在调用处理器方法的时候会自动给它们赋值，所以当在处理器方法中需要使用到这些对象的时候，可以直接在方法上给定一个方法参数的申明，然后在方法体里面直接用就可以了。但是有一点需要注意的是在使用HttpSession 对象的时候，如果此时HttpSession 对象还没有建立起来的话就会有问题。

   - Spring 自己的WebRequest 对象。 使用该对象可以访问到存放在HttpServletRequest 和HttpSession 中的属性值。

   - InputStream 、OutputStream 、Reader 和Writer 。 InputStream 和Reader 是针对HttpServletRequest 而言的，可以从里面取数据；OutputStream 和Writer 是针对HttpServletResponse 而言的，可以往里面写数据。

   - 使用@PathVariable 、@RequestParam 、@CookieValue 和@RequestHeader 标记的参数。

   - 使用@ModelAttribute 标记的参数。

   - java.util.Map 、Spring 封装的Model 和ModelMap 。 这些都可以用来封装模型数据，用来给视图做展示。

   - 实体类。 可以用来接收上传的参数。

   - Spring 封装的MultipartFile 。 用来接收上传文件的。

   - Spring 封装的Errors 和BindingResult 对象。 这两个对象参数必须紧接在需要验证的实体对象参数之后，它里面包含了实体对象的验证结果。

2. 支持的返回类型

   - 一个包含模型和视图的ModelAndView 对象。

   - 一个模型对象，这主要包括Spring 封装好的Model 和ModelMap ，以及java.util.Map ，当没有视图返回的时候视图名称将由RequestToViewNameTranslator 来决定。

   - 一个View 对象。这个时候如果在渲染视图的过程中模型的话就可以给处理器方法定义一个模型参数，然后在方法体里面往模型中添加值。

   - 一个String 字符串。这往往代表的是一个视图名称。这个时候如果需要在渲染视图的过程中需要模型的话就可以给处理器方法一个模型参数，然后在方法体里面往模型中添加值就可以了。

   - 返回值是void 。这种情况一般是我们直接把返回结果写到HttpServletResponse 中了，如果没有写的话，那么Spring 将会利用RequestToViewNameTranslator 来返回一个对应的视图名称。如果视图中需要模型的话，处理方法与返回字符串的情况相同。

   - 如果处理器方法被注解@ResponseBody 标记的话，那么处理器方法的任何返回类型都会通过HttpMessageConverters 转换之后写到HttpServletResponse 中，而不会像上面的那些情况一样当做视图或者模型来处理。

   - 除以上几种情况之外的其他任何返回类型都会被当做模型中的一个属性来处理，而返回的视图还是由RequestToViewNameTranslator 来决定，添加到模型中的属性名称可以在该方法上用@ModelAttribute(“attributeName”) 来定义，否则将使用返回类型的类名称的首字母小写形式来表示。使用@ModelAttribute 标记的方法会在@RequestMapping 标记的方法执行之前执行。

**使用 @ModelAttribute 和 @SessionAttributes 传递和保存数据**

SpringMVC 支持使用 @**ModelAttribute** 和 @**SessionAttributes** 在不同的模型（model）和控制器之间共享数据。 **@ModelAttribute** 主要有两种使用方式，一种是标注在方法上，一种是标注在 Controller 方法参数上。

当 @**ModelAttribute** 标记在方法上的时候，该方法将在处理器方法执行之前执行，然后把返回的对象存放在 session 或模型属性中，属性名称可以使用 @**ModelAttribute**(“attributeName”) 在标记方法的时候指定，若未指定，则使用返回类型的类名称（首字母小写）作为属性名称。关于 @ModelAttribute 标记在方法上时对应的属性是存放在 session 中还是存放在模型中，我们来做一个实验，看下面一段代码。

```java
@Controller
@RequestMapping ( "/myTest" )
public class MyController {

    @ModelAttribute ( "hello" )
    public String getModel() {
       System. out .println( "-------------Hello---------" );
       return "world" ;
    }

    @ModelAttribute ( "intValue" )
    public int getInteger() {
       System. out .println( "-------------intValue---------------" );
       return 10;
    }

    @RequestMapping ( "sayHello" )
    public void sayHello( @ModelAttribute ( "hello" ) String hello, @ModelAttribute ( "intValue" ) int num, @ModelAttribute ( "user2" ) User user, Writer writer, HttpSession session) throws IOException {
       writer.write( "Hello " + hello + " , Hello " + user.getUsername() + num);
       writer.write( "\r" );
       Enumeration enume = session.getAttributeNames();
       while (enume.hasMoreElements())
           writer.write(enume.nextElement() + "\r" );
    }

    @ModelAttribute ( "user2" )
    public User getUser(){
       System. out .println( "---------getUser-------------" );
       return new User(3, "user2" );
    }
}
```

当我们请求 /myTest/sayHello.do 的时候使用 @ModelAttribute 标记的方法会先执行，然后把它们返回的对象存放到模型中。最终访问到 sayHello 方法的时候，使用 @ModelAttribute 标记的方法参数都能被正确的注入值。执行结果如下所示：

```
 Hello world,Hello user210
```

由执行结果我们可以看出来，此时 session 中没有包含任何属性，也就是说上面的那些对象都是存放在模型属性中，而不是存放在 session 属性中。那要如何才能存放在 session 属性中呢？这个时候我们先引入一个新的概念 @SessionAttributes ，它的用法会在讲完 @ModelAttribute 之后介绍，这里我们就先拿来用一下。我们在 MyController 类上加上 @SessionAttributes 属性标记哪些是需要存放到 session 中的。看下面的代码：

```java
@Controller
@RequestMapping ( "/myTest" )
@SessionAttributes (value={ "intValue" , "stringValue" }, types={User. class })
public class MyController {

    @ModelAttribute ( "hello" )
    public String getModel() {
       System. out .println( "-------------Hello---------" );
       return "world" ;
    }

    @ModelAttribute ( "intValue" )
    public int getInteger() {
       System. out .println( "-------------intValue---------------" );
       return 10;
    }
   
    @RequestMapping ( "sayHello" )
    public void sayHello(Map<String, Object> map, @ModelAttribute ( "hello" ) String hello, @ModelAttribute ( "intValue" ) int num, @ModelAttribute ( "user2" ) User user, Writer writer, HttpServletRequest request) throws IOException {
       map.put( "stringValue" , "String" );
       writer.write( "Hello " + hello + " , Hello " + user.getUsername() + num);
       writer.write( "\r" );
       HttpSession session = request.getSession();
       Enumeration enume = session.getAttributeNames();
       while (enume.hasMoreElements())
           writer.write(enume.nextElement() + "\r" );
       System. out .println(session);
    }

    @ModelAttribute ( "user2" )
    public User getUser() {
       System. out .println( "---------getUser-------------" );
       return new User(3, "user2" );
    }
}
```

在上面代码中我们指定了属性为 intValue 或 stringValue 或者类型为 User 的都会放到 Session中，利用上面的代码当我们访问 /myTest/sayHello.do 的时候，结果如下：

```
 Hello world,Hello user210
```

仍然没有打印出任何 session 属性，这是怎么回事呢？怎么定义了把模型中属性名为 intValue 的对象和类型为 User 的对象存到 session 中，而实际上没有加进去呢？难道我们错啦？我们当然没有错，只是在第一次访问 /myTest/sayHello.do 的时候 @SessionAttributes 定义了需要存放到 session 中的属性，而且这个模型中也有对应的属性，但是这个时候还没有加到 session 中，所以 session 中不会有任何属性，等处理器方法执行完成后 Spring 才会把模型中对应的属性添加到 session 中。所以当请求第二次的时候就会出现如下结果：

```
 Hello world,Hello user210
user2
intValue
stringValue
```

当 @ModelAttribute 标记在处理器方法参数上的时候，表示该参数的值将从模型或者 Session 中取对应名称的属性值，该名称可以通过 @ModelAttribute(“attributeName”) 来指定，若未指定，则使用参数类型的类名称（首字母小写）作为属性名称。

**@PathVariable和@RequestParam的区别** 

请求路径上有个id的变量值，可以通过@PathVariable来获取  @RequestMapping(value = "/page/{id}", method = RequestMethod.GET)  
@RequestParam用来获得静态的URL请求入参     spring注解时action里用到。

简介：

handler method 参数绑定常用的注解,我们根据他们处理的Request的不同内容部分分为四类：（主要讲解常用类型）

A、处理**requet uri** 部分（这里指uri template中variable，不含queryString部分）的注解：   @PathVariable;

B、处理**request header**部分的注解：   @RequestHeader, @CookieValue;

C、处理**request body**部分的注解：@RequestParam,  @RequestBody;

D、处理**attribute**类型是注解： @SessionAttributes, @ModelAttribute;

- @PathVariable

  当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。

  ```java
  @Controller  
  @RequestMapping("/owners/{ownerId}")  
  public class RelativePathUriTemplateController {  
    
    @RequestMapping("/pets/{petId}")  
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
      // implementation omitted   
    }  
  }
  ```

  上面代码把URI template 中变量 ownerId的值和petId的值，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称。

-  @RequestHeader、@CookieValue

  @RequestHeader 注解，可以把Request请求header部分的值绑定到方法的参数上。

  这是一个Request 的header部分：

  ```java
  //Host                    localhost:8080  
  //Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
  //Accept-Language         fr,en-gb;q=0.7,en;q=0.3  
  //Accept-Encoding         gzip,deflate  
  //Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7  
  //Keep-Alive              300  
  @RequestMapping("/displayHeaderInfo.do")  
  public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,  
  @RequestHeader("Keep-Alive") long keepAlive)  {  
  }  
  
  ```

  上面的代码，把request header部分的 Accept-Encoding的值，绑定到参数encoding上了， Keep-Alive header的值绑定到参数keepAlive上。

  @CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。

  例如有如下Cookie值：

  ```java
  //　　JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
  
  
  @RequestMapping("/displayHeaderInfo.do")  
  public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {  
  } 
  ```

  即把JSESSIONID的值绑定到参数cookie上。

- @RequestParam, @RequestBody

  @RequestParam 

  A） 常用来处理简单类型的绑定，**通过Request.getParameter() 获取的String可直接转换为简单类型的情况**（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理**get 方式中queryString的值**，也可以处理**post方式中 body data的值**；

  B）用来处理Content-Type: 为 `application/x-www-form-urlencoded`编码的内容，提交方式GET、POST；

  C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；

  ```java
  @Controller  
  @RequestMapping("/pets")  
  @SessionAttributes("pet")  
  public class EditPetForm {  
      @RequestMapping(method = RequestMethod.GET)  
   public String setupForm(@RequestParam("petId") int petId, ModelMap model) {  
         Pet pet = this.clinic.loadPet(petId);  
     model.addAttribute("pet", pet);  
     return "petForm";  
     }
  }
  ```

  @RequestBody

  该注解常用来处理Content-Type: 不是`application/x-www-form-urlencoded`编码的内容，例如application/json, application/xml等；

  它是通过使用HandlerAdapter 配置的`HttpMessageConverters`来解析post data body，然后绑定到相应的bean上的。

  因为配置有FormHttpMessageConverter，所以也可以用来处理 `application/x-www-form-urlencoded`的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

  ```java
  @RequestMapping(value = "/something", method = RequestMethod.PUT)  
  public void handle(@RequestBody String body, Writer writer) throws IOException {  
    writer.write(body);  
  } 
  ```

- @SessionAttributes, @ModelAttribute

  @SessionAttributes:

  该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。

  该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象；

  示例代码：

  ```java
  @Controller  
  @RequestMapping("/editPet.do")  
  @SessionAttributes("pet")  
  public class EditPetForm {  
      // ...   
  } 
  ```

  @ModelAttribute

  该注解有两个用法，一个是用于方法上，一个是用于参数上；

  用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；

  用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：

  A） @SessionAttributes 启用的attribute 对象上；

  B） @ModelAttribute 用于方法上时指定的model对象；

  C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。

  用到方法上@ModelAttribute的示例代码：

  ```java
  @ModelAttribute  
  public Account addAccount(@RequestParam String number) {  
      return accountManager.findAccount(number);  
  } 
  ```

  这种方式实际的效果就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）。

  用在参数上的@ModelAttribute示例代码：

  ```java
  @RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)  
  public String processSubmit(@ModelAttribute Pet pet) {     
  } 
  ```

  首先查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上。

- < context:component-scan base-package = "" />浅析

  component-scan 默认扫描的注解类型是 @Component，不过，在 @Component 语义基础上细化后的 @Repository, @Service 和 @Controller 也同样可以获得 component-scan 的青睐

  有了`<context:component-scan>`，另一个`<context:annotation-config/>`标签根本可以移除掉，因为已经被包含进去了

  另外`<context:annotation-config/>`还提供了两个子标签

  ```
  <context:include-filter> //指定扫描的路径
   <context:exclude-filter> //排除扫描的路径
  ```

  `<context:component-scan>`有一个use-default-filters属性，属性默认为true,表示会扫描指定包下的全部的标有@Component的类，并注册成bean.也就是@Component的子注解@Service,@Reposity等。

  这种扫描的粒度有点太大，如果你只想扫描指定包下面的Controller或其他内容则设置use-default-filters属性为false，表示不再按照scan指定的包扫描，而是按照`<context:include-filter>`指定的包扫描，示例：

  ```xml
  <context:component-scan base-package="com.tan" use-default-filters="false">
          <context:include-filter type="regex" expression="com.tan.*"/>//注意后面要写.*
  </context:component-scan>
  当没有设置use-default-filters属性或者属性为true时，表示基于base-packge包下指定扫描的具体路径
  <context:component-scan base-package="com.tan" >
          <context:include-filter type="regex" expression=".controller.*"/>
          <context:include-filter type="regex" expression=".service.*"/>
          <context:include-filter type="regex" expression=".dao.*"/>
  </context:component-scan>
  ```

   效果相当于：

  ```xml
  <context:component-scan base-package="com.tan" >
          <context:exclude-filter type="regex" expression=".model.*"/>
  </context:component-scan>
  ```

   注意：本人尝试时无论哪种情况`<context:include-filter>`和`<context:exclude-filter>`都不能同时存在

