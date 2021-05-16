---
title: Spring MVC 学习(二)
date: 2019-07-10 17:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

### 数值传递

#### 返回值类型

SpringMVC 通过注解实现的 Controller 常用的返回值类型主要有三种：返回字符串，返回 ModelAndView，返回 void。

<!--more-->

`springmvc.xml` 配置文件的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 开启spring的组件扫描，自动加载bean -->
    <context:component-scan base-package="com.lyu.qjl.interview.controller" />

    <!-- 可以用mvc的注解驱动 --> 
    <mvc:annotation-driven />

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 配置视图名的默认前缀 -->
        <property name="prefix" value="/"></property>
        <!-- 配置视图名的默认后缀 -->
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

**返回字符串**

应用场景1：直接返回视图名称，例如跳转到某个功能主页。

下面的 Controller 是用来处理用户页面的请求，gotoUserEdit 方法是用来处理跳转到用户编辑页面的请求。

```java
import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import com.lyu.qjl.interview.entity.User;
import com.lyu.qjl.interview.vo.UserVO;

/**
 * 类名称：处理用户请求的handler
 */
@Controller
public class UserController{

    /**
     * 跳转到编辑用户页面
     * @return
     */
    @RequestMapping("/gotoUserEdit")
    public String gotoUserEdit() {
        return "userEdit";
    }
}
```

这个 gotoUserEdit 方法匹配 /gotoUserEdit 请求。返回值为一个 userEdit 字符串，表示的是一个视图名，但是由于我们在 springmvc.xml 中已经配置了视图名的前缀和后缀，所以前端控制器实际上获得的视图名为 p`refix配置的值 + returnStr（方法返回值） + suffix配置`的值，即 /userEdit.jsp，所以页面最终会跳转到网站根目录下的 userEdit.jsp。

**Question**：我想访问某个页面我直接去输入这个页面的地址 /userEdit.jsp 不就可以了吗，为什么还要先去请求 SpringMVC，再由它返回？

**Answer**：对于 WebContent 这一层目录下面的页面，用户如果知道某个 jsp 页面可以直接访问。但是，通常情况下为了保证页面的安全，我们一般的做法是在 WebContent 这一层目录下只留一个引导页面（index.jsp）作为跳转，把网站相关的页面放入 WEB-INF 文件夹下保护起来。因为放在 WEB-INF 文件夹下的页面没有办法通过地址栏直接访问，只能通过后台的跳转来间接的访问，所以就需要请求 SpingMVC 来返回相应的页面。

注：login.jsp, main.jsp, userEdit.jsp, userList.jsp, userMain.jsp 都是放在 WEB-INF 这层目录下的，只有 index.jsp 是放在 WebContext 这一层下的。

直接访问 WEB-INF 文件夹下的 login.jsp 页面的结果是404

这样我们就没有办法直接从地址栏去访问 WEB-INF 下的 login.jsp 页面了，只能通过 SpringMVC 来进行页面的跳转了。

这里还需要注意一点：由于页面已经放到了 WEB-INF 目录下，所以 springmvc.xml 配置文件中的视图解析器中的前缀也要对应的有所改动：

```
        <property name="prefix" value="/WEB-INF/"></property>
```

**扩展**：

大家注意到那个放在 WebContent 那一层目录下的 index.jsp 页面了吗？它在实际应用中有什么作用呢？

它一般配置为项目的欢迎页面，即 web.xml 中的 welcome-file。当我们在地址栏输入 localhost:8080/interview 的时候，首先就会访问该页面，然后由该页面向后台发请求，通过后台的 Controller 跳转到被 WEB-INF 文件夹保护起来的登录页面 login.jsp，最后，展现在我们面前的就是一个登录页面。 

index.jsp 中的内容如下：

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<%
// 转发一个 "/login" 请求给后台的 Controller，由 Controller 返回 WEB-INF 目录下的 login.jsp 页面
request.getRequestDispatcher("/login").forward(request, response);
%>
```

应用场景2：在登录成功的时候重定向到主页面，登录失败的时候转发回登录页面

String 类型的返回值除了返回一个可以被视图解析器解析的视图名以外，还可以返回 含有 redirect 或 forward 标签的字符串，如下：

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * 类名称：用于处理登录请求的处理器
 */
@Controller
public class LoginController{

    @RequestMapping("/loginUser")
    public String login(String username, String password, Model model) throws Exception {
        if (username != null && password != null) {
            if ("admin".equals(username) && "123".equals(password)) {
                // 登录成功，重定向到主页面
                return "redirect:/main";
            } else {
                // 向 request 域中设置错误提示信息
                model.addAttribute("loginFlag", "用户名或密码错误");
                // 登录失败，转发到登录页面
                return "forward:/login";
            }
        } else {
            // 只是把模型放到request域里面，并没有真正的安排好
            model.addAttribute("loginFlag", "用户名或密码错误");
            // 登录失败，转发到登录页面
            return "forward:/login";
        }
    }

    @RequestMapping("/login")
    // 对外部提供接口的时候需要指定某些参数
    public String login() {
        return "login";
    }

    @RequestMapping("/main")
    public String main() {
        return "main";
    }
}
```

redirect:/main 表示这是一个重定向到 /main 的请求， forward:/login 表示这是一个转发到 /login 的请求。

注意：如果在返回的字符串前加上了 redirect 或者 forward 标签，就不会走视图解析器，而是直接转发或重定向到指定的路径，所以对于重定向标签后面就不能加 WEB-INF ，因为重定向是客户端重新发送的请求，WEB-INF 目录下的页面仍然访问不到，但是对于转发标签的话后面可以跟 WEB-INF。

以上就是 Controller 的返回值为 String 时的另一种常见用途。

**Question**：为什么登录成功需要重定向到主页面？

**Answer**：因为转发客户端只发送了一次请求，如果登录成功转发到主页面的话，浏览器的地址栏中的 url 保留的是提交表单的请求，此时如果按 F5 刷新，就会造成二次提交表单，增大服务端的压力不说，还影响用户的体验，所以登录成功时要重定向到主页面，这样浏览器的地址栏保留的就是当前主页面的 url，无论怎么刷新都没事。 

**Question**：为什么登录失败需要转发回登录页面？

**Answer**：用户登录失败，我们需要给用户提供相应的提示信息，如：“用户名密码不匹配”。如果重定向到登录页面，用户就没有办法获取到服务端设置在 request 域里面的错误提示信息，因为重定向的本质是客户端发送了两次请求，在第二次请求里面获取不到第一次请求中的数据，所以没有办法给用户相应的提示信息。而转发只发送了一次请求，转发回登录页面的时候可以获取到请求中的数据，所以登录失败时需要转发会登录页面。 

**返回ModelAndView**

应用场景：其实这种返回值的应用场景比较随意，既可以用来做页面的跳转，也可以在跳转到页面的时候携带一些数据，例如：查询用户列表。

```java
import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import com.lyu.qjl.interview.entity.User;
import com.lyu.qjl.interview.vo.UserVO;

/**
 * 类名称：处理用户请求的handler
 */
@Controller
public class UserController{

    @RequestMapping("/getUserList")
    public ModelAndView getUserList() {
        System.out.println("进入getUseList");
        ModelAndView modelAndView = new ModelAndView();

        List<User> userList = new ArrayList<User>();
        User user1 = new User("arry", 18, "男");
        user1.setUserId(1L);
        User user2 = new User("cc", 28, "女");
        user2.setUserId(2L);
        User user3 = new User("dd", 38, "男");
        user3.setUserId(3L);
        userList.add(user1);
        userList.add(user2);
        userList.add(user3);

        // 将数据放入modelAndView对象
        modelAndView.addObject("userList", userList);

        // 将返回的逻辑视图名称放入modelAndView对象
        modelAndView.setViewName("userList");

        return modelAndView;
    }
}
```

如果想在跳转到某个页面的时候携带一些数据，调用 ModelAndView 对象的 addObject 方法设置一些属性即可，不想带数据也可以不设置，但是必须得设置视图名 setViewName ，要不然它哪知道返回到哪个页面去呢？

**返回void**

如果是返回 void 会，就必须要在方法的参数中添加 HttpServletRequest 和 HttpServletResponse 来进行页面的跳转，代码如下：

```java
public void test(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 转发到指定页面
    request.getRequestDispatcher("xxx").forward(request, response);
    // 或者重定向到指定页面
    response.sendRedirect("/xxx");
}
```

其实，从本质上来说前面两种方法（String，ModelAndView）就是间接调用的 request 和 response 这个两个对象来进行页面的跳转，以及数据的存储。

稍微的总结一下，讲了 SpringMVC 的 Controller 的三种返回值类型：String，ModelAndView，void。在讲 String 的时候扩展了一些 redirect 和 forward 标签，提了一下在实际应用中转发和重定向的用处，以及实际项目中需要把页面放入 WEB-INF 目录里保护起来。ModelAndView 可以携带数据也可以不携带数据，以及前两种方法本质上都是在操作 request 和 response 对象。

#### 数据传递

从url到Controller、从view到Controller、Controller到view以及Controller之间的数据传递。

##### 从url传递

1. @RequestParam的使用

   常见的url中会是?name=XXX&pwd=XXX的这种，如果想获取name,pwd，可以使用.@RequestParam。假如参数是可选参数，可以设置required=false,默认是true,value也要与url的对应。

   ```java
   @RequestMapping(value="/index.do")
       public ModelAndView login(@RequestParam("name")String name,@RequestParam(value="pwd",required=false) String pwd){
           ModelAndView modelAndView = new ModelAndView("Index");
           System.out.println("name:"+name+" pwd:"+pwd);
           return modelAndView;
           }
   ```

2. @PathVariable的使用

   有的url的格式是url/param1/param2这种，这种获取值可以使用.@PathVariable。

   ```java
   @RequestMapping(value="/login/{name}/{pwd}",method=RequestMethod.GET)
       public ModelAndView login1(@PathVariable("name") String name,@PathVariable("pwd") String pwd){
           ModelAndView modelAndView = new ModelAndView("Index"); 
           System.out.println("name:"+name+" pwd:"+pwd);
           return modelAndView;
           }
   ```

##### 从view传递

这里主要用form表单演示。

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Insert title here</title>
</head>
<body>

<form id="form1" name="myform" method="post" action="../../login.do" >
      用户:<input type="text" name="name"/>
      密码:<input type="password" name="pwd"/>
      <input type="submit"/>
 </form>
</body>
</html>
```

1. 直接将请求参数名作为Controller中方法的形参

   这里在Controller中设置参数是页面form表单控件的name属性。

   ```java
   @RequestMapping(value="/login.do",method=RequestMethod.POST)
       public ModelAndView login2(HttpServletRequest request,HttpServletResponse response,String name,String pwd){
           ModelAndView modelAndView = new ModelAndView("Index"); 
           System.out.println("name:"+name+" pwd:"+pwd);
           return modelAndView;
           }
   ```

2. 使用Pojo对象（就是封装的类，类中封装的字段作为参数）绑定请求参数值，原理是利用Set的页面反射机制找到User对象中的属性

   这个使用pojo有点类似之前学的,我们可以定义一个User类,然后Controller的参数类型是User.form表单的action这里要注意下要设置成action="../../login3.do".

   ```java
   public class User {
       private String name;
       private String pwd;
       public String getName() {
           return name;
       }
       public void setName(String name) {
           this.name = name;
       }
       public String getPwd() {
           return pwd;
       }
       public void setPwd(String pwd) {
           this.pwd = pwd;
       }
   }
   
   @RequestMapping(value="/login3.do")
       public ModelAndView login3(User user){
           ModelAndView modelAndView = new ModelAndView("Index"); 
           System.out.println("name:"+user.getName()+" pwd:"+user.getPwd());
           return modelAndView;
           }
   ```

3. 使用原生的Servlet API 作为Controller 方法的参数

   不仅是通过view传到Controller，url传参数到Controller也一样.既然有HttpServletRequest，我们可以通过request对象获取相关参数。

   ```java
           String username=request.getParameter("name");
           System.out.println("username:"+username);
   ```

4. 传递数组

   ```html
   <form id="form1" name="myform" method="post" action="../../login.do" >
      <input type="checkbox" name="hobby" value="1" />跑步
       <input type="checkbox" name="hobby" value="2" />游泳
       <input type="checkbox" name="hobby" value="3" />麻将
       <input type="checkbox" name="hobby" value="4" />吃
         <input type="submit"/>
    </form>
   </body>
   </html>
   ```

5. 时间传递

   只需要在Controller中增加InitBinder然后页面传的时间格式需要与dateFormat 的一致。

   ```java
       @InitBinder
       protected void init(HttpServletRequest request, ServletRequestDataBinder binder) {
           SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-mm-dd");
           dateFormat.setLenient(false);
           binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
       }
   ```

   ```html
   <form id="form1" name="myform" method="post" action="../../login.do" >
         用户:<input type="text" name="name"/>
         密码:<input type="password" name="pwd"/>
         出生日期:<input type="date" name="birthday"/>
         <input type="submit"/>
    </form>
   ```

   这里设置出生日期的type=date，可以在页面选择时间。然后在Controller中获取

   ```java
   @RequestMapping(value="/login.do",method=RequestMethod.POST)
       public ModelAndView login2(String name, String pwd,Date birthday){
           ModelAndView modelAndView = new ModelAndView("Index"); 
           System.out.println("name:"+name+" pwd:"+pwd+" birthday:"+birthday);
           return modelAndView;
           }
   ```

#### 数据绑定

前后端数据的传递过程中肯定会涉及到数据的绑定，因为一个个参数单独接收很费事且代码效率会大大降低，下面就对各种类型的数据进行绑定。

**重点**：表单大量数据（多对象）的传递绑定

1. 基本类型数据绑定

   在controller中写一个int参数绑定的方法，

   ```
   @RequestMapping("/int")
   @ResponseBody
   public  String getInt(int id){
       return ""+id;
   }
   ```

   发送请求：<http://localhost:8080/int?3>

   响应：此时会报500错误，接收不到名为id的参数

   发送请求：<http://localhost:8080/int?id=3>

   响应：3

   结论：用基本类型进行参数绑定时，就必须传入key值，且value值必须是声明的基本类型，否则将报错

2. 包装类型参数绑定（推荐使用）

   ```java
   @RequestMapping("/integer")
   @ResponseBody
   public  String getInt(Integer id){
       return ""+id;
   }
   ```

   请求：

   ```
   http://localhost:8080/integer?3  
   http://localhost:8080/integer  
   http://localhost:8080/integer？id=
   ```

   上面的三个请求响应都是：null

   请求：<http://localhost:8080/integer?id=3>

   响应：3

   结论：包装类型绑定参数时，key的值可以不传，数据也可以为空，但是要想绑定成功绑定，传的key值要和里面绑定的参数名一致

   如果想要必须传入某个参数进行绑定，可以用@RequestParam，用了这个只有所需绑定的参数必须传入，否则报错

3. 数组元素绑定

   ```java
   @RequestMapping("/array")
   @ResponseBody
   public String getUser(String[] name){
       return  Arrays.toString(name);
   }
   ```

   请求：<http://localhost:8080/array?name=xiaoshu&name=xiaozhang>
   响应：

   ```
   ["xiaoshu","xiaozhang"]
   ```

4. 多层级对象的绑定

   ```java
   @RequestMapping("/user")
   @ResponseBody
   public User getUser(User user){
       return user;
   }
   ```

   请求：<http://localhost:8080/user?name=xiaoshu&id=1&address=hangzhou&admin.name=xiaoli>

   响应:

   ```
   {"id":1,"name":"xiaoshu","address":"hangzhou","admin":{"name":"xiaoli","address":null}}
   ```

   结论：将参数绑定到对象内层对象的属性中，如这里例子中的加上admin.name

5. 同属性对象参数绑定

   这里有两个对象user和admin，它们有两个相同的属性name和address

   ```java
   @RequestMapping("/list")
   @ResponseBody
   public String list(User user, Admin admin) {
   return user.toString()+""+admin.toString();
   }
   ```

   发送请求：<http://localhost:8080/list?name=xiaoshu&address=home>
   得到：

   ```
   User{id=null, name='xiaoshu', address='home'}Admin{name='xiaoshu', address='home'}
   ```

   发现属性被同时绑定到了两个对象上

   我们可以通过在controller中配置一个@InitBinder进行分开绑定

   ```java
   @InitBinder("user")
   public void initUser(WebDataBinder binder){
       binder.setFieldDefaultPrefix("user.");
   }
   @InitBinder("admin")
   public void  initAdmin(WebDataBinder binder){
       binder.setFieldMarkerPrefix("admin.");
   
   }
   ```

   此时发送请求：<http://localhost:8080/list?user.name=xiaoshu&user.address=home>
   得到：

   ```
   User{id=null, name='xiaoshu', address='home'}Admin{name='null', address='null'}
   ```

   可以看到带user.前缀的参数只绑定到了user对象中

   那如果试一下没有配置@InitBinder时用user.和admin.作为前缀进行请求发送
   请求：<http://localhost:8080/list?user.name=xiaoshu&user.address=home&admin.name=xiaozhang&admin.address=hangzhou>
   响应：

   ```
   User{id=null, name='null', address='null'}Admin{name='null', address='null'}
   ```

   发现都是为空，参数未能成功绑定

   结论：进行同属性参数绑定时，要区分绑定时需在属性前加上对象前缀，并在controller中配置@InitBinder来明确哪个前缀的属性绑定到哪个对象中

   没有配置@InitBinder加前缀不能成功绑定

6. List绑定

   ```java
   @RequestMapping("/list")
   @ResponseBody
   public String getUser(AdminList admin) {
           return admin.toString();
       }
   ```

   请求：<http://localhost:8080/list?admins[0].name=xiaoshu&admins[1].name=xiaozhang>
   响应：

   ```
   {"admins":[{"name":"xiaoshu","address":null},{"name":"xiaozhang","address":null}]}
   ```

   结论：这里不能直接用List去绑定，而是要创建一个类类中创建一个List去绑定，AdminList中创建了一个ArrayList。

7. Map绑定

   ```java
   @RequestMapping("/map")
   @ResponseBody
   public AdminMap getMap(AdminMap adminMap){
       return  adminMap;
   }
   ```

   AdminMap中维护了一个Hashmap

   请求：<http://localhost:8080/map?admins['X'].name=xiaoshu&admins['Y'].name=xiaozhang>
   响应：

   ```
   {"admins":{"Y":{"name":"xiaozhang","address":null},"X":{"name":"xiaoshu","address":null}}}
   ```

   结论：map存放可以保证key的唯一性，过滤冲重复数据

8. 重点介绍表单多对象传递的绑定(特别是多对象的属性名相同的情况)

    要在一张表单中提交多个对象，并且还要在后台Controller 中精准的绑定接收。可是，这些对象中的参数名可能相同，后台接收入参时无法像struts那样jsp表单中使用Object.Param形式对表单进行精准绑定入参，我们都知道struts2默认就是这种方案，这是因为struts2采用了OGNL，并通过栈（根对象）进行操作的，而且栈中默认有action实例，所以很自然的没有这种问题。另一方面说，Struts用这种方式绑定入参，却牺牲了性能。虽然springmvc不能像struts那样方便的很直接的入参绑定，当然，spring多对象绑定入参也提供了方法。

   今天就以前台表单提交两个对象做实例。为了扩大影响，我让这两个对象的属性相同。

    User.Java 和Addr.java

   ```java
   public class User implements Serializable{  
   String id;  
   String name;  
   //get..set....  
   }  
   public class Addr implements Serializable{  
     
   String id;  
     
   String name;//set..get...  
   }  
   ```

   前台JSP

   ```jsp
   <form action="/test/test" method="post">  
      <input type="text" name="user.id" value="huo_user_id">  
      <input type="text" name="user.name" value="huo_user_name">  
      <input type="text" name="addr.id" value="huo_addr_id">  
      <input type="text" name="addr.name" value="huo_addr_name">  
      <input type="submit" value="提交">  
   </form>  
   ```

   若字段过多或者一个属性字段供几个对象使用，则需要前端组装处理：

   ```javascript
    $("#submitForm").attr("action", basePath + "xtapply/submitApplyASD?" + new Date().getTime())
   .append($("<input>").attr("name", "creditProductId").val("${product.creditProductId}").hide())
   .append($("<input>").attr("name", "productName").val("${product.productName}").hide())
   .append($("<input>").attr("name", "bankName").val("${product.bankName}").hide())
   .append($("<input>").attr("name", "xtFddbr.identyNoCounty").val($("#identyArea option:selected").text()).hide())
   .append($("<input>").attr("name", "xtQyxx.companyBusinessProvince").val($("#qyProvince option:selected").text()).hide())
   .append($("<input>").attr("name", "xtQyxx.companyBusinessCity").val($("#qyCity option:selected").text()).hide())
   .append($("<input>").attr("name", "xtQyxx.companyBusinessCounty").val($("#qyArea option:selected").text()).hide());
   $("#submitForm").submit();
   
   ```

   看到这种情况，很容易想到struts进行绑定非常方便，可是，谁让我们要使用SpringMVC呢。。。这种情况springMVC直接进行入参，是不能接收到参数的。

   **解决思路**：

   使用 @InitBinder 注解进行绑定参数。前台表单中name属性仍然使用Object.Param形式传入。（另一种解决思路：扩展spring的HandlerMethodArgumentResolver以支持自定义的数据绑定方式。）

   ```java
        @InitBinder("user")  
           public void initBinderUser(WebDataBinder binder) {  
               binder.setFieldDefaultPrefix("user.");  
           }  
   ```

   此处使用@InitBinder() 中间的value，用于指定命令/表单属性或请求参数的名字，符合该名字的将使用此处的DataBinder，如我们的@ModelAttribute("user1") User user1 将使用@InitBinder("user1")指定的DataBinder绑定；如果不指定value值，那么所有的都将使用。

   DataBinder.setFieldDefaultPrefix 意思是设置参数的前缀，如我们的是"user1."，此处不能少了"."，

   这种方式的缺点：

   1. 不支持Path variable的绑定，如/test1/{user1.id}这种情况的绑定；

   2. 不支持如集合/数组的绑定；

   **后台接收 完整代码**：

   ```java
   @Controller  
   @RequestMapping("/test")  
   public class TestController {  
   // 绑定变量名字和属性，参数封装进类  
       @InitBinder("user")  
       public void initBinderUser(WebDataBinder binder) {  
           binder.setFieldDefaultPrefix("user.");  
       }  
       // 绑定变量名字和属性，参数封装进类  
       @InitBinder("addr")  
       public void initBinderAddr(WebDataBinder binder) {  
           binder.setFieldDefaultPrefix("addr.");  
       }  
         
         
       @RequestMapping("/test")  
       @ResponseBody  
       public Map<String,Object> test(HttpServletRequest request,@ModelAttribute("user") User user,@ModelAttribute("addr") Addr addr){  
           Map<String,Object> map=new HashMap<String,Object>();  
           map.put("user", user);  
           map.put("addr", addr);  
           return map;  
       }  
   ```

### 原理

#### HandlerMapping

处理器映射器将会把请求映射为 HandlerExecutionChain 对象（包含一个 Handler 处理器对象、多个 HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略。

处理器映射器有三种，三种可以共存，相互不影响，分别是BeanNameUrlHandlerMapping、SimpleUrlHandlerMapping和ControllerClassNameHandlerMapping；

**BeanNameUrlHandlerMapping**

默认映射器，即使不配置，默认就使用这个来映射请求。

```xml
<!-- 配置映射处理器：根据bean(自定义Controller)的name属性的url去寻找handler；springmvc默认的映射处理器是
        BeanNameUrlHandlerMapping
         -->
        <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>


        <!-- 配置处理器适配器来执行Controlelr ,springmvc默认的是
        SimpleControllerHandlerAdapter
        -->
        <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>

        <!-- 配置自定义Controller -->
        <bean id="myController" name="/hello.do" class="com.hyp.learn.controller.HelloController"></bean>

```

**SimpleUrlHandlerMapping**

该处理器映射器可以配置多个映射对应一个处理器.

```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings">
		<props>
			<prop key="/ss.do">testController</prop>
			<prop key="/abc.do">testController</prop>
		</props>
	</property>
</bean>
//上面的这个映射配置表示多个*.do文件可以访问同一个Controller。	
<bean id="testController" name="/hello.do" class="com.hyp.learn.controller.HelloController"></bean>

```

**ControllerClassNameHandlerMapping**

该处理器映射器可以不用手动配置映射, 通过[[类名.do](http://xn--eqrt45g.do)]来访问对应的处理器.

```xml
//这个Mapping一配置, 我们就可以使用Controller的 [类名.do]来访问这个Controller.
<bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"></bean>

```

#### HandlerAdapter

处理器适配器有两种，可以共存，分别是SimpleControllerHandlerAdapter和HttpRequestHandlerAdapter。

**SimpleControllerHandlerAdapter**

SimpleControllerHandlerAdapter是默认的适配器，所有实现了org.springframework.web.servlet.mvc.Controller 接口的处理器都是通过此适配器适配, 执行的。

**HttpRequestHandlerAdapter**

该适配器将http请求封装成HttpServletResquest 和HttpServletResponse对象. 所有实现了 org.springframework.web.HttpRequestHandler 接口的处理器都是通过此适配器适配, 执行的. 实例如下所示:

1. 配置HttpRequestHandlerAdapter适配器

   ```xml
   <!-- 配置HttpRequestHandlerAdapter适配器 -->
   <bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"></bean>
   ```

2. 编写处理器

   ```java
   public class HttpController implements HttpRequestHandler{
   
   	public void handleRequest(HttpServletRequest request, HttpServletResponse response)
   			throws ServletException, IOException {
   		//给Request设置值，在页面进行回显
   		request.setAttribute("hello", "这是HttpRequestHandler！");
   		//跳转页面
   		request.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(request, response);
   	}
   }
   
   ```

3. index页面

   ```html
   <html>
   <body>
   <h1>${hello}</h1>
   </body>
   </html>
   
   ```

**Adapter源码分析**

模拟场景：前端控制器（DispatcherServlet）接收到Handler对象后，传递给对应的处理器适配器（HandlerAdapter），处理器适配器调用相应的Handler方法。

1. 模拟处理器

   ```java
   //以下是Controller接口和它的是三种实现 
   public interface Controller {
   }
   
   public class SimpleController implements Controller{
   	public void doSimpleHandler() {
   		System.out.println("Simple...");
   	}
   }
   
   public class HttpController implements Controller{
   	public void doHttpHandler() {
   		System.out.println("Http...");
   	}
   }
   
   public class AnnotationController implements Controller{
   	public void doAnnotationHandler() {
   		System.out.println("Annotation..");
   	}
   } 
   
   ```

2. 模拟处理器适配器

   ```java
   //以下是HandlerAdapter接口和它的三种实现
   public interface HandlerAdapter {
   	public boolean supports(Object handler);
   	public void handle(Object handler);
   }
   
   public class SimpleHandlerAdapter implements HandlerAdapter{
   	public boolean supports(Object handler) {
   		return (handler instanceof SimpleController);
   	}
   
   	public void handle(Object handler) {
   		((SimpleController)handler).doSimpleHandler();
   	}
   }
   
   public class HttpHandlerAdapter implements HandlerAdapter{
   	public boolean supports(Object handler) {
   		return (handler instanceof HttpController);
   	}
   
   	public void handle(Object handler) {
   		((HttpController)handler).doHttpHandler();
   	}
   }
   
   public class AnnotationHandlerAdapter implements HandlerAdapter{
   	public boolean supports(Object handler) {
   		return (handler instanceof AnnotationController);
   	}
   
   	public void handle(Object handler) {
   		((AnnotationController)handler).doAnnotationHandler();
   	}
   }
   
   ```

3. 模拟DispatcherServlet

   ```java
   public class Dispatcher {
   	public static List<HandlerAdapter> handlerAdapter = new ArrayList<HandlerAdapter>();
   	
   	public Dispatcher(){
   		handlerAdapter.add(new SimpleHandlerAdapter());
   		handlerAdapter.add(new HttpHandlerAdapter());
   		handlerAdapter.add(new AnnotationHandlerAdapter());
   	}
   	
   	//核心功能
   	public void doDispatch() {
   		//前端控制器（DispatcherServlet）接收到Handler对象后
   		//SimpleController handler = new SimpleController();
   		//HttpController handler = new HttpController();
   		AnnotationController handler = new AnnotationController();
   		
   		//传递给对应的处理器适配器（HandlerAdapter）
   		HandlerAdapter handlerAdapter = getHandlerAdapter(handler);
   		
   		//处理器适配器调用相应的Handler方法
   		handlerAdapter.handle(handler);
   	}
   	
   	//通过Handler找到对应的处理器适配器（HandlerAdapter）
   	public HandlerAdapter getHandlerAdapter(Controller handler) {
   		for(HandlerAdapter adapter : handlerAdapter){
   			if(adapter.supports(handler)){
   				return adapter;
   			}
   		}
   		return null;
   	}
   }
   
   ```

4. 测试

   ```java
   public class Test {
   	public static void main(String[] args) {
   		Dispatcher dispather = new Dispatcher();
   		dispather.doDispatch();
   	}
   }
   
   ```

#### Handler

这里只介绍上文提到的两种处理器, 除此之外还有很多适用于各种应用场景的处理器, 尤其是Controller接口还有很多实现类, 大家可以自行去了解.

**Controller**

org.springframework.web.servlet.mvc.Controller, 该处理器对应的适配器是 SimpleControllerHandlerAdapter.

```java
public interface Controller {

	/**
	 * Process the request and return a ModelAndView object which the DispatcherServlet
	 * will render. A {@code null} return value is not an error: it indicates that
	 * this object completed request processing itself and that there is therefore no
	 * ModelAndView to render.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @return a ModelAndView to render, or {@code null} if handled directly
	 * @throws Exception in case of errors
	 */
	ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;

}

```

该处理器方法用于处理用户提交的请求, 通过调用service层代码, 实现对用户请求的计算响应, 并最终将计算所得数据及要响应的页面封装为一个ModelAndView 对象, 返回给前端控制器(DispatcherServlet).

Controller接口的实现类:

![1.png](https://i.loli.net/2019/07/14/5d2ac5b8a4dc637622.png)

**HttpRequestHandler**

org.springframework.web.HttpRequestHandler, 该处理器对应的适配器是 HttpRequestHandlerAdapter.

```java
public interface HttpRequestHandler {
    void handleRequest(HttpServletRequest var1, HttpServletResponse var2) throws ServletException, IOException;
}

```

该处理器方法没有返回值, 不能像 ModelAndView 一样, 将数据及目标视图封装为一个对象, 但可以将数据放入Request, Session等域属性中, 并由Request 或 Response完成目标页面的跳转.

#### ViewResolver

视图解析器负责将处理结果生成View视图. 这里介绍两种常用的视图解析器:

**InternalResourceViewResolver**

该视图解析器用于完成对当前Web应用内部资源的封装和跳转. 而对于内部资源的查找规则是, 将ModelAndView中指定的视图名称与视图解析器配置的前缀与后缀想结合, 拼接成一个Web应用内部资源路径. 内部资源路径 = 前缀 + 视图名称 + 后缀.

InternalResourceViewResolver解析器会把处理器方法返回的模型属性都存放到对应的request中, 然后将请求转发到目标URL.

1. 处理器

   ```java
   public class MyController implements Controller {
       @Override
       public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
           ModelAndView mv = new ModelAndView();
           mv.addObject("hello", "hello world!");
           mv.setViewName("index");
           return mv;
       }
   }
   
   ```

2. 视图解析器配置

   ```xml
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/jsp/"></property>
           <property name="suffix" value=".jsp"></property>
      </bean>
   
   ```

   当然, 若不指定前缀与后缀, 直接将内部资源路径写到 setViewName()中也可以, 相当于前缀与后缀均为空串.

   ```java
   mv.setViewName("/WEB-INF/jsp/index.jsp");
   ```

**BeanNameViewResolver**

InternalResourceViewResolver视图解析器存在一个问题, 就是只可以完成将内部资源封装后的跳转, 无法跳转向外部资源, 如外部网页.

BeanNameViewResolver 视图解析器将资源(内部资源和外部资源)封装为bean实例, 然后在 ModelAndView 中通过设置bean实例的id值来指定资源. 在配置文件中可以同时配置多个资源bean.

1. 处理器

   ```java
   public class MyController implements Controller {
       @Override
       public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
   
           return new ModelAndView("myInternalView");
   //        return new ModelAndView("baidu");
       }
   }
   
   ```

2. 视图解析器配置

   ```xml
   <!--视图解析器-->
   <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
   <!--内部资源view-->
   <bean id="myInternalView" class="org.springframework.web.servlet.view.JstlView">
       <property name="url" value="/jsp/show.jsp"/>
   </bean>
   <!--外部资源view-->
   <bean id="baidu" class="org.springframework.web.servlet.view.RedirectView">
       <property name="url" value="https://www.baidu.com/"/>
   </bean>
   
   ```

   了解完这两种视图解析器后是不是有种熟悉感, 是的, 就是请求转发和重定向.

#### 中文乱码解决

**Get请求乱码**

Tomcat8已经解决了Get请求乱码, 如果是Tomcat8以下的版本, 可以使用以下两种方法:

- 更改Tomcat的配置文件server.xml

  ![2.png](https://i.loli.net/2019/07/14/5d2ac668b839339294.png)

- 对参数进行重新编码

  ```java
  String userName =new
   String(request.getParamter(“userName”).getBytes(“ISO-8859-1”),“UTF-8”); //ISO-8859-1是Tomcat8以下版本的默认编码
  ```

**Post请求乱码**

在web.xml中加入：

```xml
<filter>
    <filter-name>characterEncoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```



