---
title: Java JSP (二)
date: 2019-05-17 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

### 四大作用域

我们在定义每一个变量，每一个属性的时候，都会考虑这个变量、属性的作用范围，也就是作用域。我们会根据我们的需求定义最适当作用域内的变量和属性。在写Java，C++等代码的时候，这个作用域问题还比较好理解，无非就是局部变量、全局变量、静态变量等区别。而到了JSP开发中，这个作用域概念就和一些新的名词混在一起，变的模糊，难懂了起来。

<!--more-->

对于JSP中的四大作用域，主要是指以下四个：

- page作用域
- request作用域
- session作用域
- application作用域

这四个作用域的作用范围，由上到下是一个比一个大。下面就对上述的四个作用域分别进行详细的总结。

#### page作用域

page直译就是页面的意思，所以page作用域就比较好理解了——page作用域表示只在当前页面有效。当程序运行跑出了当前的页面，你就无法在其它的页面访问当前页面设置的属性值。

我们都知道，JSP最终会被编译成Servlet文件。在Servlet容器中，每个Servlet都只存在一个实例。但是对于page作用域的属性来说，在当前页面设置的属性只在本次访问该页面有效，当你再次访问该页面时，又会重新初始化页面的属性。例如以下代码：

```null
<%
out.print(pageContext.getAttribute("SiteName")); // 输出null
pageContext.setAttribute("SiteName", "平心");
%>
```

当我在浏览器访问该页面时会输出`null`；当我再重新打开一个该页面时，还会输出`null`，并不会输出”果冻想-一个原创技术文章分享网站”。也就是说，page作用域范围的不会存在线程安全的问题，每一次访问同一个页面，设置的page作用域的属性都是不一样的。

#### request作用域

request表示一次客户端的请求。一次请求的生命周期从客户端发起到服务器接收并响应该请求，或者将该请求`forward`到另一个页面或者Servlet进行处理而结束。在此期间，本次请求的参数，属性都是有效的；一旦客户端刷新浏览器，重新发起请求，则之前的请求参数和属性都将失效。

特别需要注意的是，当我们使用`<jsp:forward .../>`动作将当前请求转向另一个页面或者Servlet的时候，该请求的参数和属性也一并转过去，并不会因为`<jsp:forward .../>`动作而丢失request的参数和属性。

#### session作用域

我一直都在强调session是一个非常重要的概念。当我们向服务器发送第一个请求开始，只要页面不关闭，或者会话未过期（默认30分钟），或者未调用HttpSession的invalidate()方法，接下来的操作都属于同一次会话的范畴。

在JSP中，每当向服务器发送一个请求，服务器响应这个请求的时候，会在客户端的Cookie中写一个session  id值。每次发送请求的时候，会将该session id值一起发送到服务器端，服务器端根据该session  id值来判断每次请求是否属于同一个session的范畴之内。

#### application作用域

application的作用域是最广的，它代表着整个Web应用的全局变量，对每一个页面，每一个Servlet都是有效的。当我们在application中设置属性时，这个属性在任意的一个页面都是可以访问的。

在application作用域中设置的属性如果不手动调用`removeAttribute`函数进行删除的话，那么application中的属性将永远不会删除，如果Web容器发生重启，此时application范围内的所有属性都将丢失。

### JSP自定义标签

自定义标签库是一种非常优秀的表现层组件技术。通过使用自定义标签库，可以在简单的标签中封装复杂的功能。 

那么为什么需要使用自定义的标签呢？主要是基于以下几个原因：

- JSP脚本非常不好阅读，而且这些`<%! %>`、`<% %>`这些东西也非常不好写；
- 当JSP脚本和HTML混在一起时，那更痛苦，当需要表现一些复杂数据时，更是如此；
- 很多公司的美工是直接参与表现层的代码编写的，而相比于HTML来说，JSP代码则更难写。

于此，我们需要一种类似于HTML那种简单写法的语法来完成JSP的工作，所以JSP中就有了自定义标签这个强大的功能。

虽然有了JSP自定义标签，我们可以在表现层写起来更轻松，但是底下我们也要做很多其它的工作，还是应了那句老话：要想人前显贵，就得人后受罪。那么到底如何开发JSP自定义标签呢？大体上分为以下三步：

1. 开发自定义标签处理类
2. 建立一个`*.tld`文件，每个`*.tld`文件对应一个标签库，每个标签库可包含多个标签
3. 在JSP文件中使用自定义标签

开发JSP自定义标签无非就是上述三步，下面就对上述的三步进行详细的讲解与分析。

#### 开发自定义标签类

当我们在表现层使用自定义标签时，你应该拥有最基本的意识，那就是自定义标签最终都得通过Java类来进行处理。是的，自定义标签就是通过Java类处理的，我们通过标签类来封装哪些复杂的功能，对上层提供简单的标签。所以当我们定义我们自己的标签时，工作重点都在编写这个标签类。

自定义标签类都继承一个父类：`javax.servlet.jsp.tagext.SimpleTagSupport`，除了这个条件以外，自定义标签类还要有如下要求：

- 如果标签类包含属性，每个属性都需要有对应的`getter`和`setter`方法
- 需要重写`doTag()`方法，这个方法用于生成页面内容；也就是说，当我们在表现层使用自定义标签时，使用标签的地方将使用`doTag()`输出的内容替代

下面就来写一个最简单的标签类：

```
public class TestTag extends SimpleTagSupport {
    public void doTag() throws JspException, IOException
    {
        getJspContext().getOut().write("baidu--误你终生");
    }
}
```

#### 建立TLD文件

我们写Servlet的时候，会在web.xml中加入定义的Servlet类和URL之间的映射。同理，我们定义了标签类，也需要一个配置文件，将自定义标签和对应的标签类关联起来。在定义自定义标签时，这个配置文件叫做标签库定义文件，是以`tld`后缀结尾的，每个TLD文件对应一个标签库，一个标签库中可包含多个标签。

标签库定义文件的根元素是`taglib`，它可以包含多个tag子元素，每个tag子元素都定义一个标签。我们需要将`tld`文件保存到Web应用的`WEB-INF/`路径，或者`WEB-INF`的任意子路径下。

以下就是我定义的一个简单的`tld`文件。

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">

    <description>Test 1.0</description>
    <tlib-version>1.0</tlib-version>
    <short-name>jtlib</short-name>
    <uri>http://www.pingxin.com/jtlib/</uri>

    <!-- 定义一个标签 -->
    <tag>
        <!-- 定义标签名 -->
        <name>jtTag</name>
        <!-- 定义标签处理类 -->
        <tag-class>com.hyp.learn.javaweb.TestTag</tag-class>
        <!-- 定义标签体内容为空 -->
        <body-content>empty</body-content>
    </tag>
</taglib>
```

标签库定义文件是一个XML文件，该XML文件的根元素是`taglib`元素。`taglib`主要有以下几个子元素：

- `description`，对该tld文件的一些描述性的信息
- `tlib-version`，指定该标签库实现的版本，这是一个作为标识的内部版本号，对程序没有太大的作用
- `short-name`，标签库的默认短名，没有太大的作用
- `uri`，这个属性非常重要，它指定该标签库的URI，相当于指定该标签库的唯一标识。JSP页面中使用标签库时就是根据该URI属性来定位标签库的

接下来，`taglib`元素下可以包含多个tag元素，每个tag元素定义一个标签，tag元素主要有以下子元素：

- `name`，指定标签的名字，JSP页面中就是根据该名称来使用此标签的，所以该子元素非常重要

- `tag-class`，指定标签的处理类，它指定了标签由哪个标签处理类来处理，该子元素极其重要

- ```
  body-content
  ```

  ，指定标签体的内容，该子元素非常重要，它主要可以取以下几个值： 

  - `tagdependent`，指定标签处理类自己负责处理标签体
  - `empty`，该标签只能作为空标签来使用
  - `scriptless`，指定该标签的标签体可以是静态HTML元素、表达式语言，但不能是JSP脚本
  - `dynamic-attributes`，指明该标签是否支持动态属性。只有定义动态属性标签时才需要该子元素

以上就是标签库定义文件的编写规范，以及编写过程中需要注意的内容事项。编写好标签库定义文件以后，将标签库文件放在Web应用的WEB-INF路径，或者任意子路径下，Java Web会自动加载该文件，该文件定义的标签库也将生效。

#### 使用标签库

标签库处理类定义好了，标签库定义文件也写好了，接下来就该小试牛刀了，看看在JSP中到底该怎么使用我们自定义的标签。

在JSP页面中使用标签库需要注意以下两点：

- 标签库URI：确定使用哪个标签库
- 标签名：确定使用哪个标签

具体使用标签库主要分为以下两个步骤：

1. 导入标签库；使用`taglib`编译指令导入标签库，就是将标签库和指定前缀关联起来
2. 使用标签；在JSP页面中使用自定义标签

下面通过一个简单的JSP来说明如何使用自定义标签，JSP页面代码如下：

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://www.pingxin.com/jtlib/" prefix="jt"%>
<html>
<head>
    <title>Page 6</title>
</head>
<body>
<!-- 使用自定义标签  -->
<jt:jtTag />
<jt:jtTag />
</body>
</html>
```

总结到这里，对于如何定义一个简单的标签，以及如何使用这个简单的自定义标签，大家心里应该都有数了，接下来看看自定义标签的一些高级用法，而这些高级用法才是开发中真正经常碰到的。

#### 带属性的标签

很多时候，单纯的一个标签根本无法满足我们的需要。我们需要在标签中添加一些属性，通过设置的这些属性值，我们可以进行更丰富的操作，比如执行属性值中指定的SQL语句，从而返回并显示查询结果。下面就来说说如何在自定义标签中添加属性的功能。

首先，带属性的标签必须要为每个属性提供对应的setter和getter方法；例如，我们的自定义标签将会有三个属性，分别是：name、price和store。那么我们对应的标签处理类会变成下面这个样子：

```
public class TestTag extends SimpleTagSupport {
    // 标签定义的属性
    private String name;
    private String price;
    private String store;

    // 需要为每个属性定义setter和getter方法
    public void setName(String name)
    {
        this.name = name;
    }

    public String getName()
    {
        return this.name;
    }

    public void setPrice(String price)
    {
        this.price = price;
    }

    public String getPrice()
    {
        return this.price;
    }

    public void setStore(String store)
    {
        this.store = store;
    }

    public String getStore()
    {
        return this.store;
    }

    public void doTag() throws JspException, IOException
    {
        JspWriter out = getJspContext().getOut();
        out.write("书名：" + name);
        out.write("价格：" + price);
        out.write("书店：" + store);
    }
}
```

每个属性都必须要有对应的setter和getter方法，这是不能少的。标签对应的处理类进行了对应的变更；同理，对于标签库定义文件也要进行对应的修改，需要为`<tag... />`元素增加`<attribute... />`子元素，每个attribute子元素定义一个标签属性。在定义attribute元素时，需要为其指定以下几个子元素：

- `name`，设置属性名；在使用自定义标签时，就是通过该名指定属性名
- `required`，设置该属性是否为必须属性，取值为true或false
- `fragment`，设置该属性是否支持JSP脚本、表达式等动态内容，取值为true或false

为此，我们的标签库定义文件就要修改为如下这样：

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">

    <description>Test 1.0</description>
    <tlib-version>1.0</tlib-version>
    <short-name>jtlib</short-name>
    <uri>http://www.pingxin.com/jtlib/</uri>

    <!-- 定义一个标签 -->
    <tag>
        <!-- 定义标签名 -->
        <name>jtTag</name>
        <!-- 定义标签处理类 -->
        <tag-class>com.hyp.learn.javaweb.TestTag</tag-class>
        <!-- 定义标签体内容为空 -->
        <body-content>empty</body-content>

        <!-- 配置name属性 -->
        <attribute>
            <name>name</name>
            <required>true</required>
            <fragment>true</fragment>
        </attribute>

        <!-- 配置price属性 -->
        <attribute>
            <name>price</name>
            <required>true</required>
            <fragment>true</fragment>
        </attribute>

        <!-- 配置store属性 -->
        <attribute>
            <name>store</name>
            <required>true</required>
            <fragment>true</fragment>
        </attribute>
        
    </tag>
</taglib>
```

接下来，我们就可以在JSP中使用这个带有属性定义的自定义标签了，具体代码如下：

```
<jt:jtTag name="Java编程思想" price="75.58" store="三味书屋" /> <br/>
<jt:jtTag name="看见" price="25.10" store="书海书屋" /> <br/>
<jt:jtTag name="狼图腾" price="23.00" store="国家图书馆" /> <br/>
```

这就是带有属性的自定义标签。

#### 带标签体的标签

可以看到，上面自定义的标签都是没有任何标签体的，都是一个简单的标签。那么如果我们希望标签中带有标签体，这又如何完成呢？

对于带标签体的自定义标签，我们可以在标签内嵌入静态的HTML内容和动态的JSP内容，通常用于循环输出一些内容。下面就通过一个简单的例子进行说明。

标签类的处理代码如下：

```
public class TestTag extends SimpleTagSupport {
    // 标签定义的属性
    private String collection;
    private String item;

    public void setCollection(String collection)
    {
        this.collection = collection;
    }

    public String getCollection()
    {
        return this.collection;
    }

    public void setItem(String item)
    {
        this.item = item;
    }

    public String getItem()
    {
        return this.item;
    }


    public void doTag() throws JspException, IOException
    {
        JspContext context = getJspContext();

        // 根据属性值，获得设置的对象
        Collection itemList = (Collection)context.getAttribute(collection);
        for (Object s : itemList)
        {
            // 设置属性值，供标签体使用
            context.setAttribute(item, s);

            // 输出标签体，这个是重点
            getJspBody().invoke(null);
        }
    }
}

```

标签库定义文件：

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">

    <description>Test 1.0</description>
    <tlib-version>1.0</tlib-version>
    <short-name>jtlib</short-name>
    <uri>http://www.pingxin.com/jtlib/</uri>

    <!-- 定义一个标签 -->
    <tag>
        <!-- 定义标签名 -->
        <name>jtTag</name>
        <!-- 定义标签处理类 -->
        <tag-class>com.hyp.learn.javaweb.TestTag</tag-class>
        <!-- 定义标签体内容为空 -->
        <body-content>scriptless</body-content>

        <!-- 配置属性 -->
        <attribute>
            <name>collection</name>
            <required>true</required>
            <fragment>true</fragment>
        </attribute>

        <attribute>
            <name>item</name>
            <required>true</required>
            <fragment>true</fragment>
        </attribute>

    </tag>
</taglib>
```

对于标签库定义文件来说，主要是`<body-content>`字段的值需要设置为`scriptless`。最重要的是JSP页面里如何使用带有标签体的自定义标签，具体代码如下：

```
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %><%--
  Created by IntelliJ IDEA.
  User: hyp
  Date: 2019/5/21
  Time: 下午5:19
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://www.pingxin.com/jtlib/" prefix="jt"%>
<html>
<head>
    <title>Page 6</title>
</head>
<body>
<%
    // 创建一个List对象
    List<String> obj = new ArrayList<String>();
    obj.add("博客园");
    obj.add("开源中国");
    obj.add("百度");
    pageContext.setAttribute("collection", obj);
%>

<jt:jtTag collection="collection" item="item">
    ${pageScope.item} <br />
</jt:jtTag>

</body>
</html>

```

由于JSP主要作为表现层，上面只是通过一个简单的例子来说明如何表现数据；很多时候，我们后台返回一个数据集合，我们就可以通过这种方式来表现数据。

### Filter

Filter是什么？在工作的过程中，我们经常听到Filter这个词，那这货到底是个什么东西呢？这篇文章将对Java Web开发中遇到的Filter进行详细的分析与总结。

Filter，译为过滤器；既然是过滤器，那么主要过滤什么呢？Filter主要对客户端的请求和服务器的响应进行过滤。说直白点就是下面这两个意思：

- 客户端的请求到达服务器，服务器真正开始处理这个请求之前，要经过Filter的过滤
- 服务器真正的处理完这个请求，生成响应之后，要经过Filter的过滤，才能将响应发送给客户端

也就是说，我们可以通过Filter技术，对web服务器管理的所有web资源，例如JSP、Servlet、静态图片文件或静态 html文件等进行拦截，从而实现一些特殊的功能。例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。

Filter在请求到达服务器之后和服务器生成响应到客户端之前都会发挥它的作用，我通过下图来看看Filter的工作角色：

![1.png](https://i.loli.net/2019/05/21/5ce3d287cf15494400.png)

Filter就好比在一条处理链上，依次排开，Request和Response依次经过Filter的处理。这里就使用了《责任链模式》。通过上图，我们可以很清楚的知道Filter的作用位置和角色。明白了这儿以后，接下来看看如何创建Filter，以及如何使用Filter。

**创建Filter的步骤**

创建Filter和创建Servlet类似，主要分为以下两步：

- 创建Filter处理类
- 在web.xml文件中配置Filter

下面就从上面的两步进行详细的总结，并通过实际的例子进行分析。

**1.  创建Filter处理类**

创建Filter类必须要实现`javax.servlet.Filter`接口，在这个接口中主要定义了以下三个方法：

- `void init(FilterConfig config)`；用于完成Filter的初始化
- `void destroy()`；用于Filter销毁前，完成某些资源的回收
- `void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`；实现过滤功能，对每个请求及响应进行拦截和处理

下面就实现一个Filter类，对每个到达服务器的请求、以及服务器的响应进行日志记录。具体的Filter类实现如下：

```
public class LogFilter implements Filter
{
    private FilterConfig config;

    public void init(FilterConfig config)
    {
        this.config = config;
    }

    public void destroy()
    {
        this.config = null;
    }

    // 这个方法是Filter的核心方法
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    throws IOException, ServletException
    {
        // 对用户的请求进行处理
        ServletContext context = this.config.getServletContext();
        long begin = System.currentTimeMillis();

        // 输出过滤信息
        System.out.println("开始过滤...");
        HttpServletRequest hRequest = (HttpServletRequest)request;
        System.out.println("Filter已经截获到用户请求的地址:" + hRequest.getServletPath());

        // 处理完以后，将请求交给下一个Filter或者Servlet处理
        chain.doFilter(request, response);

        // 对服务器的响应进行处理
        long end = System.currentTimeMillis();
        System.out.println("过滤结束");
        System.out.println("请求被定为到:" + hRequest.getRequestURI() + "; 所花费的时间为:" + (end - begin));
    }
}
```

在上面的代码中，`doFilter`方法是最重要的，实现该方法就可实现对用户请求进行预处理，也可实现对服务器响应进行后处理——它们的分界线为是否调用了`chain.doFilter()`，执行该方法之前，即对用户请求进行预处理；执行该方法之后，即对服务器响应进行后处理。

**2. 配置Filter**

同开发Servlet一样，写完了类，接下来就是配置了，我们需要在web.xml文件中配置Filter。具体的配置和Servlet配置如出一辙。

```
<filter>
    <filter-name>log</filter-name>
    <filter-class>com.jellythink.practise.LogFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>log</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
  </filter-mapping>
```

上面配置中比较重要的就是`url-pattern`和`dispatcher`了。

- 通过配置`url-pattern`，我们就可以控制对哪些URL进行过滤，哪些URL不进行过滤

- 通过配置

  ```
  dispatcher
  ```

  ，可以指定以哪种方式访问资源可以被Filter拦截；对于

  ```
  dispatcher
  ```

  可以配置以下值： 

  - REQUEST，默认为REQUEST；当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过`RequestDispatcher`的`include()`或`forward()`方法访问时，那么该过滤器就不会被调用
  - INCLUDE；如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用；例如：`<jsp:include file="/.."/>`
  - FORWARD；如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用；例如：`<jsp:forward page="" />`或通过page指令的errorPage转发页面`<%@ page errorPage="error.jsp" %>`
  - ERROR；如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用

在Web开发中，Filter是一个非常重要的概念；在后续的学习中，我们会发现很多优秀的框架都会用到Filter这个特性，从而实现对客户端请求的拦截。

### Listener

Listener就是监听器，监听着某个事件的发生。当监听的事件发生时，则要通知这个监听器去“干”一些事情。这篇文章就要对Java Web开发中的这个Listener说道说道。

我们都知道，Web应用在Web容器中运行，Web应用内部会不断的产生各种事件，例如Web应用被启动、Web应用被停止、用户Session开始、用户Session结束等；一般情况来说，我们并不在意这些事件的发生，但是有的时候，实现某些需求却要在这些事件上做文章。那么如何做文章呢？

当这些事件发生时，它需要去通知那些关注这个事件的“人”，这都是基于《观察者模式》实现了该功能。正好Servlet API正好提供了大量监听器来“关注”Web应用的内部事件，从而允许当Web内部事件发生时回调事件监听器内的方法。

在Servlet API中目前提供的Web事件监听器接口有如下几个：

| 接口名称                          | 作用                                                  |
| --------------------------------- | ----------------------------------------------------- |
| `ServletContextListener`          | 用于监听Web应用的启动和关闭                           |
| `ServletContextAttributeListener` | 用于监听ServletContext范围（application）内属性的改变 |
| `ServletRequestListener`          | 用于监听用户请求                                      |
| `ServletRequestAttributeListener` | 用于监听ServletRequest范围（request）内属性的改变     |
| `HttpSessionListener`             | 用于监听用户Session的开始和结束                       |
| `HttpSessionAttributeListener`    | 用于监听HttpSession范围（session）内属性的改变        |

下面就通过实际的代码来总结如何实现监听器接口，以及如何配置它。

#### 实现Listener类

使用监听器的功能只需要两步：

- 定义实现相关Listener接口的类
- 在web.xml文件中配置Listener

就这些。下面通过实现`ServletContextListener`接口来仔细看看如何定义一个Listener类、以及如何配置这个Listener类。

```
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class ServletContextListenerImpl implements ServletContextListener
{
    // 应用启动时，该方法会被调用
    public void contextInitialized(ServletContextEvent sce)
    {
        System.out.println("应用启动......");
    }

    // 应用关闭时，该方法会被调用
    public void contextDestroyed(ServletContextEvent sce)
    {
        System.out.println("应用关闭......");
    }
}
```

上面这个类实现了`ServletContextListener`接口；当应用启动、关闭时，都会回调对应的方法。

**配置Listener**

同Servlet、Filter一样，完成了对应的类以后，都需要在Web.xml中配置这个类名，从而让Web容器能够找到对应的类。对于Listener的配置，相对于Servlet和Filter来说更简单一些。具体配置如下：

```
<listener>
    <listener-class>com.jellythink.practise.ServletContextListenerImpl</listener-class>
</listener>
```

**ServletRequestListener代码示例**

上面对`ServletContextListener`接口进行了简单的说明，下面通过一段代码来演示如何使用`ServletRequestListener`接口。

```

import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.http.HttpServletRequest;

public class ServletRequestListenerImpl implements ServletRequestListener
{
      // 当用户请求结束、被销毁时触发该方法
      public void requestDestroyed(ServletRequestEvent sre)
      {
          HttpServletRequest req = (HttpServletRequest)sre.getServletRequest();
          System.out.println("++++发向" + req.getRequestURI() + "请求被初始化++++");
      }

      // 当用户请求到达、初始化时触发该方法
      public void requestInitialized(ServletRequestEvent sre)
      {
          HttpServletRequest req = (HttpServletRequest)sre.getServletRequest();
          System.out.println("++++发向" + req.getRequestURI() + "请求被销毁++++");
      }
}
```

当实现了`ServletRequestListener`接口以后，每当有请求到达服务器，并正确初始化`HttpServletRequest`对象以后，或者用户结束了请求以后，都会回调对应的监听器方法。

**ServletRequestAttributeListener代码示例**

如果实现了`ServletRequestAttributeListener`监听器接口，当在request作用域下的属性发生了变化时，就会回调对应的方法，下面就通过一段代码来实现`ServletRequestAttributeListener`监听器。

```
public class ServletRequestAttributeListenerImpl implements ServletRequestAttributeListener
{
      // 当向request作用域添加属性时，触发该方法
      public void attributeAdded(ServletRequestAttributeEvent srqe)
      {
          ServletRequest req = srqe.getServletRequest();
          String attrName = srqe.getName();
          Object attrValue = srqe.getValue();
          System.out.println(req + "范围内添加了名为:" + attrName + ", 值为:" + attrValue + "的属性");
      }

      // 当从request作用域删除属性时，触发该方法
      public void attributeRemoved(ServletRequestAttributeEvent srqe)
      {
          ServletRequest req = srqe.getServletRequest();
          String attrName = srqe.getName();
          Object attrValue = srqe.getValue();
          System.out.println(req + "范围内删除名为:" + attrName + ", 值为:" + attrValue + "的属性");
      }

      // 当修改request作用域属性值时，触发该方法
      public void attributeReplaced(ServletRequestAttributeEvent srqe)
      {
          ServletRequest req = srqe.getServletRequest();
          String attrName = srqe.getName();
          Object attrValue = srqe.getValue();
          System.out.println(req + "范围内名为:" + attrName + ", 值为:" + attrValue + "的属性被修改了");
      }
}
```

这就是关于属性相关的监听器的用法，就这些。

Listener是非常重要的东西，可能我在这里说你也不会体会到。有朝一日，你要实现某些需求时，当你想到Listener时，你会发现这个东西太好用了。

### Servlet实现JSON

#### 解决HttpServletResponse输出的中文乱码问题

主要有这三行代码就可以了。

```
request.setCharacterEncoding("utf-8");
response.setHeader("Content-type", "text/html;charset=UTF-8");
response.setCharacterEncoding("utf-8");
```

使用库：

```
	<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.0.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.57</version>
    </dependency>
```

JSON类：

```
public class ResultCode {
    @JSONField(name = "code", ordinal = 0)
    private int code;
    @JSONField(name = "err", ordinal = 1)
    private int error;
    @JSONField(name = "msg", ordinal = 2)
    private String msg = "";
    @JSONField(name = "data", ordinal = 3)
    private Object data = new ArrayList<>();

    public ResultCode() {
    }

    public ResultCode(int code, int error, String msg, Object data) {
        this.code = code;
        this.error = error;
        if (null != msg) {
            this.msg = msg;
        }
        if (null != data) {
            this.data = data;
        }
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public int getError() {
        return error;
    }

    public void setError(int error) {
        this.error = error;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }


    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "ResultCode{" +
                "code=" + code +
                ", error=" + error +
                ", msg='" + msg + '\'' +
                ", data=" + data +
                '}';
    }
}
```

MakeCode接口类：

```
public interface MakeCode {
    public ResultCode code(HttpServletRequest request, HttpServletResponse response);
}
```

Servlet基类：

```
public abstract  class ISIServlet extends HttpServlet implements MakeCode{
    protected ResultCode code=null;

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        response.setHeader("Content-type", "text/html;charset=UTF-8");
        response.setCharacterEncoding("utf-8");
        code(request,response);
        PrintWriter out = response.getWriter();
        out.println(JSON.ToJSON(code));
        out.flush();
        out.close();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request,response);
    }
}
```

获取JSON数据Servlet:

```
public class MyServlet extends ISIServlet {
    
    @Override
    public ResultCode code(HttpServletRequest request, HttpServletResponse response) {
        List weather=null;
        try {
        	//获取数据
            weather= ServicesUtils.getDao().getUseWeather();
            if (null!=weather) {
                code = ServicesUtils.code(1, 0, "", weather);
            }else {
                code= ServicesUtils.error("系统错误");
            }
        } catch (ISIException e) {
            code=ServicesUtils.error(e.getMessage());
        }
        return code;
    }
}
```

### 面试题

1. 请谈谈，转发和重定向 之间的区别？

   **转发过程**：客户端首先发送一个请求到服务器，服务器匹配Servlet，并指定执行。当这个Servlet执行完后，它要调用getRequestDispacther()方法，把请求转发给指定的Servlet_list.jsp，整个流程都是在服务端完成的，而且是在同一个请求里面完成的，因此Servlet和jsp共享同一个request，在Servlet里面放的所有东西，在student_list.jsp中都能取出来。因此，student_list.jsp能把结果getAttribute()出来，getAttribute()出来后执行完把结果返回给客户端，整个过程是一个请求，一个响应。

   **重定向过程：**客户端发送一个请求到服务器端，服务器匹配Servlet，这都和请求转发一样。Servlet处理完之后调用了sendRedirect()这个方法，这个方法是response方法。所以，当这个Servlet处理完后，看到response.sendRedirect()方法，立即向客户端返回个响应，响应行告诉客户端你必须再重新发送一个请求，去访问student_list.jsp，紧接着客户端收到这个请求后，立刻发出一个新的请求，去请求student_list.jsp,在这两个请求互不干扰、相互独立，在前面request里面setAttribute()的任何东西，在后面的request里面都获得不了。因此，在sendRedirect()里面是两个请求，两个响应。

    Forward是在服务器端的跳转，就是客户端一个请求给服务器，服务器直接将请求相关参数的信息原封不动的传递到该服务器的其他jsp或Servlet去处理。而sendRedirect()是客户端的跳转，服务器会返回客户端一个响应报头和新的URL地址，原来的参数信息如果服务器没有特殊处理就不存在了，浏览器会访问新的URL所指向的Servlet或jsp，这可能不是原来服务器上的webService了。

   **总结：**

   ​    1、转发是在服务器端完成的，重定向是在客户端发生的；

   ​    2、转发的速度快，重定向速度慢；

   ​    3、转发是同一次请求，重定向是两次请求；

   ​    4、转发地址栏没有变化，重定向地址栏有变化；

   ​    5、转发必须是在同一台服务器下完成，重定向可以在不同的服务器下完成。