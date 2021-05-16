---
title: Java JSP (一)
date: 2019-05-16 18:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

### 基础知识

#### 什么是JSP？

JSP就是Java Server Pages，就是通过在标准的HTML页面中嵌入Java代码。整个页面由两部分组成：

1. 静态部分：标准的HTML标签、静态的页面内容，这些内容与静态HTML页面相同
2. 动态部分：受Java程序控制的内容，这些内容由Java程序来动态生成

<!--more-->

在Servlet中嵌入大量的静态文本格式，导致Servlet的开发效率低，代码难维护，而JSP就是为了解决这个问题而出现的；

JSP的本质还是Servlet，每个JSP页面就是一个Servlet实例。JSP页面由系统编译成Servlet，Servlet再负责响应用户请求。也就是说，JSP是Servlet的另外一种形式，使用JSP时，其实还是使用Servlet，因为Web应用中的每个JSP页面都会由Servlet容器生成对应的Servlet，比如Tomcat，最终都是由对应的Servlet来处理用户的请求。

#### JSP注释

```
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title>Insert title here</title>
    </head>
    <body>
        <%-- 这里是JSP注释 --%>
        <%
            /**
             *这个是多行注释
             */
            for (int i = 0; i < 3; ++i)
            {
        out.println("Hello World"+"<br/>"); // 这个是Java单行注释
            }
        %>

        <!-- 这个是HTML注释 -->
    </body>
</html>
```

#### JSP声明

JSP声明用于声明变量和方法。在JSP中，我们似乎不会看到的类的直接定义，方法貌似可以独立存在，这好像有点脱离了Java的轨迹。实际上，JSP声明将会被转换成对应的Servlet的成员变量或成员方法，所以JSP声明依然是遵循着Java语法的。

JSP的声明语法格式如下：

```
<%! 声明部分 %>
```

例如：

```
<%!
    int i = 0;
    private String getDefaultWord()
    {
        return "Helo World";
    }
%>
```

#### 输出JSP表达式

JSP提供了一种输出表达式值的简单方法，可以输出各种类型数据，具体包括int、double、boolean、String、Object等，具体格式如下：

```null
<%= 表达式%>
```

例如：

```null
<%!
    private int i = 0;
    private String defaultWord = "Hello World";
    private boolean isFinished = true;
    private double money = 23.45f;

    public String toString()
    {
        return "输出JSP表达式测试类";
    }
%>

<%-- 输出int类型值 --%>
<p><%= i%></p>

<%-- 输出String类型值 --%>
<p><%= defaultWord %></p>

<%-- 输出boolean类型值 --%>
<p><%= isFinished %></p>

<%-- 输出double类型值 --%>
<p><%= money %></p>

<%-- 输出Object类型值 --%>
<p><%= this %></p>

```

当输出Object类型的值时，会调用对应的toString方法。

#### JSP脚本

在JSP页面中可以包含任何可以执行的Java代码，所有可执行Java代码都可以通过如下方式嵌入JSP页面中，

```null
<% Java代码 %>
```

例如：

```null
<%
for (int i = 0; i < 3; ++i)
{
    out.println("Hello World");
%>
<br/>
<%} %>

<%
out.println("重要的事情说三遍");
%>
```

### 指令

JSP指令用来声明JSP页面的一些属性，例如编码格式、文档类型等。这些指令用来告知JSP引擎如何处理该JSP页面。经常使用的编译指令有以下三个：

- page指令：该指令是针对当前页面属性的指令；
- include指令：用于指定包含另一个页面；
- taglib指令：用于定义和访问自定义标签。

使用编译指令的语法格式如下：

```null
<%@ 编译指令名 属性名=“属性值”...%>
```

现在就分别对以上三个编译指令进行总结。

#### page指令

`page`指令是最常用的指令，用来声明JSP页面的属性等。比如：

```
<%@ page language="java" contentType="text/html"; charset="utf-8" %>
```

需要注意的是，任何`page`允许的属性都只能出现一次，否则会出现编译错误；`import`属性除外，`import`可以多次出现。

我们可以在`page`指令中设置以下的属性：

| 属性名称     | 取值范围                 | 描述                                                         |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| language     | java                     | 指定解释该JSP文件时采用的语言，默认为java                    |
| extends      | 任何类的全名（包含包名） | 指定JSP页面编译所产生的Java类所继承的父类，或所实现的接口    |
| import       | 任何类名、包名           | 引入该JSP中用到的类、包等。JSP会默认导入四个包：`java.lang.*`、`javax.servlet.*`、`javax.servlet.jsp.*`、`javax.servlet.http` |
| session      | true、false              | 指明该JSP页面是否内置Session对象。如果为true，则内置Session对象；否则不内置Session对象。默认为true |
| buffer       | none、数字+kb            | 指定缓存大小。当autoFlush为true时有效                        |
| autoFlush    | true、false              | 是否运行缓存。如果为true，则使用`out.println()`等方法输出的字符串并不是立刻到达客户端的，而是暂存在缓存中；当缓存满、程序执行完毕或者执行`out.flush()`操作时才到客户端。默认为true |
| isThreadSafe | true、false              | 用来设置JSP页面是否可以多线程访问。当设置为true时，JSP页面能同时响应多个客户的请求；当设置为false时，JSP页面同一时刻只能响应一个客户的请求，其他客户需要排队等待。默认值为true |
| info         | 任意字符串               | 指明JSP的信息。该信息可以通过`Servlet.getServletInfo()`方法获得 |
| errorPage    | 某个JSP页面的相对路径    | 指明一个错误显示页面。如果该JSP程序抛出了一个未捕获的异常，则转到errorPage指定的页面。errorPage指定的页面通常isErrorPage属性为true，且内置的exception对象为未捕获的异常 |
| contentType  | 合法的文档类型           | 客户端根据该属性判断文档类型，具体的请参见[这里](https://www.iana.org/assignments/media-types/media-types.xhtml) |
| pageEncoding | 指定生成网页的编码字符集 | JSP文件本身的编码，将JSP翻译成Java源码时，就是根据pageEncoding的编码格式读取的 |
| isErrorPage  | true、false              | 设置JSP页面是否为错误处理页面；如果该页面本身已是错误处理页面，则通常无须指定errorPage属性 |

#### include指令

使用`include`指令，可以将一个外部文件嵌入到当前的JSP文件中。编译时，当前的JSP文件完全包含了被包含页面的代码。举个例子说明：

页面一（page1.jsp）主要代码：

```
<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@ page pageEncoding="UTF-8"%>

<body>
    <%
    out.println("这是页面一");
    %>

    <%-- 这里包含页面二 --%>
    <%@ include file="page2.jsp" %>
</body>
```

页面二（page2.jsp）主要代码：

```
<%@ page pageEncoding="UTF-8"%>
<%
out.println("这是页面二");
%>
```

运行该程序，在生成class和java目录下，我们都无法找到page2_jsp.class和page2_jsp.java。这说明页面二还未经过编译就已经添加到页面一中了，这就好比直接将页面二的代码写到页面一中，请记住这一点，这将和后面总结到的include动作是截然相反的原理。

#### taglib指令

JSP支持标签技术，使用标签功能可以实现视图代码重用，很少量的代码就能实现很复杂的显示效果。由于`taglib指令`是一项非常重要的技术，我们可以自定义我们自己的标签库，后续总结自定义标签库中我再结合自定义的标签库一起总结`taglib`指令，这里就不废话了。

### 动作

指令是告知Servlet引擎如何编译当前JSP页面，而动作指令只是运行时的动作。JSP动作是对常用的JSP功能的抽象与封装，它只是JSP脚本的标准化写法。指令影响编译，动作影响运行。

JSP动作主要有以下七个：

- jsp:forward：执行页面转向，将请求的转发到下一个页面
- jsp:param：用于传递参数，必须与其它支持参数的标签一起使用
- jsp:include：用于动态引入一个JSP页面，注意与`include指令`进行区别
- jsp:plugin：用于下载Applet到客户端执行，现在已经基本放弃这种东西了
- jsp:useBean：创建一个JavaBean的实例
- jsp:setProperty：设置JavaBean实例的属性值
- jsp:getProperty：获取JavaBean实例的属性值

下面就对上述的七种JSP动作分别进行总结。

##### `jsp:forward`动作

`jsp:forward`动作用于将页面响应转发到另外的页面。可以转发到静态的HTML页面，也可以转发到动态的JSP页面，还可以转发到Servlet。

`jsp:forward`动作的语法如下：

```
<jsp:forward page="relativeURL">
    <jsp:param name="" value="" />
</jsp:forward>
```

`<jsp:param>`用来增加请求参数，我们可以在目的页面使用`HttpServletRequest`类的`getParameter()`方法获取。下面通过一个例子来说明，如何从页面一转到页面二。

页面一（page1.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面一");
    %>

    <jsp:forward page="page2.jsp">
        <jsp:param name="site" value="http://www.baidu.com" />
    </jsp:forward>
</body>
```

页面二（page2.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面二");
    %>
    <br />
    <%= request.getParameter("site") %>
</body>
```

可以看到并没有输出：这是页面一。这是由于从表面看，`<jsp:forward>`动作是将用户请求“转发”到了另一个新页面；而实际上，`<jsp:forward>`并没有重新向新页面发送请求，它只是完全采用了新页面代替原有页面来对用户生成响应————请求依然是一次请求，所以之前的请求参数、请求属性都不会丢失，而是都“转到”了新的页面。但是，浏览器的URL并不会因为`<jsp:forward>`而发生改变。

##### jsp:param动作

`jsp:param`动作用于设置参数值。由于单独的`jsp:param`动作没有实际意义，所以这个动作本身不能单独使用，而是需要结合以下三个动作一起使用：

- jsp:include
- jsp:forward
- jsp:plugin

当与`jsp:include`动作指令结合使用时，`jsp:param`动作用于将参数值传入被导入的页面；当与`jsp:forward`动作结合使用时，`jsp:param`动作用于将参数传入被转向的新页面；当与`jsp:plugin`结合使用时，则用于将参数传入页面的JavaBean实例或Applet实例。

##### jsp:include动作

`jsp:include`动作用于包含某个页面，但是它不会导入被`jsp:include`页面的编译指令，仅仅将被导入页面的body部分的内容插入当前页面。

`jsp:include`动作的语法格式如下：

```
<jsp:include page="relativeURL" flush="true">
    <jsp:param name="" value="" />
</jsp:include>
```

`flush`属性用于指定输出缓存是否转移到被导入的文件中；如果指定为true，则包含在被导入文件中；如果为false，则包含在原文件中。通过一个简单的例子来说明问题：

页面一（page1.jsp）主要代码：

```
<%
    out.println("这是页面一");
%>
<br/>
<jsp:include page="page2.jsp">
    <jsp:param name="site" value="http://www.baidu.com" />
</jsp:include>
```

页面二（page2.jsp）主要代码

```
<%
    out.println("这是页面二");
%>
 <br />
<%= request.getParameter("site") %>
```

在page1中是通过调用`include`方法来包含page2的输出的。

##### jsp:plugin动作

这种已经真的退出历史舞台的东西，直接pass掉了。如果你还感兴趣，直接去百度吧。

##### jsp:useBean动作

首先需要明白Java Bean就是普通的Java类，也叫做POJO(Plain Ordinary Java Object，普通Java对象)，不要被这些名词所迷惑了。

其次，以下三个动作都是与Java Bean相关联的：

- jsp:useBean动作
- jsp:setProperty动作
- jsp:getProperty动作

`<jsp:useBean>`动作用于在JSP中定义一个Java Bean对象和初始化这个Java实例，具体的语法格式为：

```
<jsp:useBean id="beanId" class="className" scope="Value" />
```

| 属性名 | 取值范围                                        | 描述                                                         |
| ------ | ----------------------------------------------- | ------------------------------------------------------------ |
| id     | 合法的Java变量名称                              | 指明Java Bean对象的名称；在JSP中可以使用该名称引用该Java Bean对象 |
| class  | Java Bean类的全名                               | Java Bean类的全名，包含包名                                  |
| scope  | 可以取值为：page、request、session和application | page：该JavaBean实例仅在该页面有效；request：该JavaBean实例在本次请求有效；session：该JavaBean实例在本次会话内有效；application：该JavaBean实例在本应用内一直有效 |

有的时候，我们经常被scope这样的作用于搞晕了，在后面的文章中，将会再具体的总结`page`、`request`、`session`和`application`这四个作用域。

下面通过一个简单的实例来演示`jsp:useBean`动作：定义一个简单的Java类：

定义一个简单的Java类：

```
public class PersonInfo
{
    private String name;
    private int age;
    private String sex;

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }

    public String getSex()
    {
        return sex;
    }

    public void setSex(String sex)
    {
        this.sex = sex;
    }
}
```

页面page1.jsp主要代码：

```
<body>
    <form action="page2.jsp" method="get">
        <fieldset>
            <legend align="center">请填写个人信息</legend>
            <table align="center" width="400">
                <tr>
                    <td>姓名</td>
                    <!-- 这里的name值需要和Java类中的属性值进行对应 -->
                    <td><input type="text" name="name" value="" /></td>
                </tr>

                <tr>
                    <td>年龄</td>
                    <td><input type="text" name="age" value="" /></td>
                </tr>

                <tr>
                    <td>性别</td>
                    <td>
                        <input type="radio" name="sex" value="Male" />Male
                        <input type="radio" name="sex" value="Female" />Female
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td><input type="submit" name="submit" value="提交" class="button" /></td>
                </tr>
            </table>
        </fieldset>
    </form>
</body>
```

页面page2.jsp主要代码：

```
<body>
    <%-- 实例化PersonInfo类 --%>
    <jsp:useBean id="personInfo" class="PersonInfo" scope="page" />

    <%-- 设置所有属性，也可以设置单个属性 --%>
    <jsp:setProperty name="personInfo" property="*" />

    <form>
        <fieldset>
            <legend align="center">您填写的个人信息</legend>
            <table align="center" width="400">
                <tr>
                    <td>姓名</td>

                    <%-- 取属性值 --%>
                    <td><jsp:getProperty name="personInfo" property="name" /></td>
                </tr>

                <tr>
                    <td>年龄</td>
                    <td><jsp:getProperty name="personInfo" property="age" /></td>
                </tr>

                <tr>
                    <td>性别</td>
                    <td><jsp:getProperty name="personInfo" property="sex" /></td>
                </tr>

                <tr>
                    <td></td>
                    <td><input type="button" name="back" value="返回" class="button" onclick="history.go(-1);" /></td>
                </tr>
            </table>
        </fieldset>
    </form>
</body>
```

##### jsp:setProperty动作

`jsp:setProperty`动作的语法格式为：

```
<jsp:setProperty name="Bean Name" property="propertyName" value="value" />
```

当使用通配符”*”的时候，就需要页面中的属性name和Java类中的属性名称相匹配。具体请参见上述的代码例子。

##### jsp:getProperty动作

`jsp:getProperty动作`动作的语法格式为：

```
<jsp:getProperty name="Bean Name" property="propertyName" />
```

### jsp内置对象

在示例代码中经常会使用`out.println`这样的写法，当时很好奇，这个`out`是个什么对象？看着也不像个类名啊，怎么不用实例化就可以直接用了？后来才知道`out`是JSP中的一个内置对象。

在JSP脚本中包含九个内置对象，这九个内置对象都是Servlet API接口的实例，只是在你访问页面的时候，由对应Servlet的_jspService方法来创建了这些实例。说白了，在你的JSP脚本中，已经有了以下九个可以直接使用的实例对象：

- `application`对象：`javax.servlet.ServletContext`的实例，该实例代表JSP所属的Web应用本身；
- `config`对象：`javax.servlet.ServletConfig`的实例，该实例代表该JSP的配置信息；
- `exception`对象：`java.lang.Throwable`的实例，该实例代表其它页面中的异常和错误；
- `out`对象：`javax.servlet.jsp.JspWriter`的实例，该实例代表JSP页面的输出流；
- `page`对象：代表当前页面本身，也就是Servlet中的this；
- `pageContext`对象：`javax.servlet.jsp.PageContext`的实例，该对象代表该JSP页面上下文；
- `request`对象：`javax.servlet.http.HttpServletRequest`的实例，该对象封装了一次请求，客户端的请求参数都封装在该对象里；
- `response`对象：`javax.servlet.http.HttpServletResponse`的实例，代表服务器对客户端的响应；
- `session`对象：`javax.servlet.http.HttpSession`的实例，该对象代表一次会话。

#### application对象

`application`对象一般有如下两个作用：

- 在整个Web应用的多个JSP、Servlet之间共享数据；
- 访问Web应用的全局配置参数。

由于`application`对象代表Web应用本身，具有全局操作性，可以用于Web应用全局范围内的数据传递和共享；同时，`application`可以从web.xml中读取全局配置；总之，`application`可以作为一个Web应用全局的数据设置与访问点。

**全局共享数据**

`application`对象通过以下两个函数来设置属性和取属性值：

- setAttribute(String attrName, Object value); 设置全局属性值
- getAttribute(String attrName); 取全局属性值

下面通过一个简单的例子进行说明，页面一（page1.jsp）和页面二（page2.jsp）是同一个Web应用下的两个没有任何关系的页面；在页面一中通过`application`对象设置全局属性，在页面二中通过`application`对象获取页面一中设置的全局属性值。

页面一（page1.jsp）主要代码：

```
<%
application.setAttribute("defaultWebsite", "http://www.baidu.com");
application.setAttribute("websiteInfo", "百度--最大的中文搜索引擎");
%>
```

页面二（page2.jsp）主要代码：

```
<body>
    <%= application.getAttribute("defaultWebsite") %>
    <%= application.getAttribute("websiteInfo") %>
</body>
```

如果不访问页面一，而直接访问页面二，此时只会输出null；只有访问完页面一，设置了全局的属性defaultWebsite和websiteInfo两个属性以后，再访问页面二就可以获得在页面一种设置的全局属性值了。

`application`不仅可以用于两个JSP页面之间共享数据，还可以用于Servlet和JSP之间共享数据。我们可以把`application`当做一个容器，任何JSP、Servlet都可以把某个变量放入`application`中保存，并为之指定一个属性名；而该Web应用里的其它JSP、Servlet就可以根据该属性名来得到这个变量。

下面通过一个Servlet来获取`application`对象中的属性值：

```
PrintWriter out = response.getWriter();
ServletContext sc = getServletConfig().getServletContext();
out.println(sc.getAttribute("defaultWebsite"));
out.println(sc.getAttribute("websiteInfo"));
```

在Servlet中并没有`application`对象，但是每个Web应用只有一个ServletContext实例，在Servlet中则必须通过代码获取`application`对象。

**访问全局配置**

通过在web.xml文件中，我们可以配置全局的配置参数。我们可以通过`application`对象获取Web应用的配置参数。现在通过一个简单的例子来说明：

在web.xml中配置全局参数：

```
<context-param>
    <param-name>defaultWebsite</param-name>
    <param-value>http://www.baidu.com</param-value>
</context-param>

<context-param>
    <param-name>websiteInfo</param-name>
    <param-value>百度--最大的中文搜索引擎</param-value>
</context-param>
```

在JSP中可以通过如下代码获取全局配置参数：

```
<%= application.getInitParameter("defaultWebsite") %>
<%= application.getInitParameter("websiteInfo") %>
```

在整个Web应用中，在JSP中都可以使用`application`对象调用`getInitParameter`方法获取全局配置参数；在Servlet同样可以通过如下代码来获得全局配置参数：

```
// 获取上下文
ServletContext servletContext = getServletConfig().getServletContext();

// 获得参数
String defaultWebsite = servletContext.getInitParameter("defaultWebsite");
String websiteInfo = servletContext.getInitParameter("websiteInfo");
```

总归于此，在`application`对象的主要用法大抵上就以上两种，涉及`application`对象的主要方法有以下几个：

- setAttribute
- getAttribute
- getInitParameter

#### config对象

每次写完一个JSP页面文件，直接就运行了，也没有说有什么JSP配置文件啊，倒是在写servlet的时候，需要在web.xml中进行配置。今天在学习JSP内置config对象的时候，发现原来JSP也可以进行配置，然后通过JSP内置的config对象可以获取配置的初始化参数值。

JSP内置对象config是javax.servlet.ServletConfig类的实例。ServletConfig封装了在web.xml中配置的初始化参数；在JSP中可以通过config来获取这些参数值。下面就先总结如何在web.xml中配置JSP。

基本上每一个Java Web应用都有一个web.xml配置文件，下面就在web.xml中配置JSP页面。

```
<web-app ...>
    <!-- 根据JSP页面，定义一个servlet -->
    <servlet>
        <servlet-name>ConfigDemo</servlet-name>
        <jsp-file>/page1.jsp</jsp-file>

        <init-param>
            <param-name>defaultWebsite</param-name>
            <param-value>http://www.baidu.com</param-value>
        </init-param>

        <init-param>
            <param-name>websiteInfo</param-name>
            <param-value>百度--最坑的中文广告引擎</param-value>
        </init-param>
    </servlet>

    <!-- JSP与URL的映射，可以在浏览器地址栏输入url-pattern配置的值访问页面 -->
    <servlet-mapping>
        <servlet-name>ConfigDemo</servlet-name>
        <url-pattern>/config</url-pattern>
    </servlet-mapping>
</web-app>
```

配置JSP页面与servlet的最大区别就在于`<jsp-file>`和`<servlet-class>`这两个标签。JSP中通过`<jsp-file>`标签指定JSP页面的位置和名字；servlet则通过`<servlet-class>`标签来确定servlet类的包名和类名。

上面在web.xml中配置了JSP页面，并配置了两个初始化参数，现在我们就可以通过JSP内置的config参数来获取初始化参数值。

```
<body>
    <%= config.getInitParameter("defaultWebsite") %>
    <br />
    <%= config.getInitParameter("websiteInfo") %>
</body>
```

访问url：http://localhost:8080/javaweb/config

**servlet中使用config**

其实我们在开发的时候，几乎不会在web.xml中配置JSP页面；这主要由JSP的功能作用决定的，JSP主要用来做表现层，几乎不会涉及到配置或后台数据的处理；更多的时候，我们都是在servlet中来读取配置数据，完成业务逻辑，下面就通过几行代码来说说如何在servlet中使用config。

```
PrintWriter out = response.getWriter();
ServletConfig conf = getServletConfig();
out.println(conf.getInitParameter("defaultWebsite"));
out.println(conf.getInitParameter("websiteInfo"));
```

在servlet没有内置的config对象，需要调用`getServletConfig`函数获得config对象，之后就可以调用`getInitParameter`函数获得初始化参数值了。

由于config主要用来读取web.xml中配置的页面初始化参数，而我们几乎不会在web.xml中对JSP页面进行配置，所以在JSP页面中，我们很少使用config对象，但是了解一下内置的config对象，让我们对JSP会多了一种认知，多一种理解，为以后设计我们自己的系统提供多一种视野。

#### exception对象

每个程序员写的代码不可能是完美无缺的，而程序也会偶尔抽风；那么如果程序抽风了，出现了异常了该怎么办？出现了异常，我们就需要处理异常；但是在JSP脚本中无须处理异常，JSP脚本包含的所有可能出现的异常都可以交给专门处理错误的页面进行处理。

打开JSP编译生成的对应Servlet源码文件，在_jspService方法中，我们可以看到这样的代码结构：

```null
try {
    // ......
} catch (java.lang.Throwable t) {
    // ......

    if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
} finally {
    // ......
}
```

你会发现，_jspService方法被大大的一个`try...catch...finally`异常捕获结构包围住。这也就表明，在JSP中抛出的异常，基本上都逃不出这个`try...catch...finally`的魔掌。再看`catch`中这行代码：

```null
if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
```

一旦捕获到了异常，并且`_jspx_page_context`不为null，就由`_jspx_page_context`调用`handlePageException`来处理异常。

`_jspx_page_context`首先判断抛出异常的当前JSP页面page指令是否指定了errorPage属性，如果指定了，则将请求forward到errorPage属性指定的页面；否则就使用系统页面来输出异常信息。

这就是JSP页面中出现异常的处理方式，但是说了这么多，这和JSP内置的`exception`对象有什么关系呢？接着看。

**exception对象**

JSP页面中有一个内置的`exception`对象，这个`exception`对象是Throwable的实例。当在JSP中发生错误或异常时，就会将捕获的`java.lang.Throwable t`赋值给内置对象`exception`异常对象。由此可见，`exception`对象仅在异常处理页面中才有效，通过前面的异常处理流程，我们可以看到这一点。下面通过一个简单的例子来说明如何使用内置`exception`对象。

页面一（page1.jsp）主要代码：

```null
<%@ page errorPage="/page2.jsp" %>
<%
int a = 10;
int b = 0;
int c = a / b; // 抛出异常
%>
```

页面二（page2.jsp）主要代码：

```null
<!-- 输出异常信息 -->
<%=exception.toString() %>
```

在页面一中，指定page编译指令erroePage属性值为page2.jsp，当除以0时，会抛出异常，页面继而forward到页面二进行异常处理。

在以后开发中，我们可以定义一个统一的异常处理页面，所有的JSP页面中将errorPage属性指定为统一的异常处理页面，这样方便页面风格的统一，也方便页面调试。

内置`exception`对象没有多少东西可讲的，但是作为九个内置对象的一份子，还是好好总结一下吧，说不定哪天会用到呢。

#### out对象

内置的`out`对象是`javax.servlet.jsp.JspWriter`类的实例。我们可以使用`out`对象向客户端输出字符类内容。

由于在使用内置`out`对象的地方都可以使用更为简洁的输出表达式`<%= ... %>`来代替，所以在实际的JSP页面中，我们很少使用内置的`out`对象。但是输出表达式`<%= ... %>`的本质就是`out.print(...)`。

由于内置`out`对象的内容实在太少了，没有多少东西可以总结的，下面就将常用的API总结一下，要不这篇文章就真的啥内容都没有了。

| 名称                  | 说明                                    |
| --------------------- | --------------------------------------- |
| void println()        | 向客户端打印字符串                      |
| void clear()          | 清除缓冲区，在flush之后调用会抛出异常   |
| void clearBuffer()    | 清除缓冲区，在flush之后调用不会抛出异常 |
| void flush()          | 将缓冲区内容输出到客户端                |
| int getBufferSize()   | 返回缓存大小，单位KB                    |
| int getRemaining()    | 返回缓存剩余大小，单位KB                |
| boolean isAutoFlush() | 返回缓冲区满时，是自动flush还是抛出异常 |
| void close()          | 关闭输出流                              |

#### page对象

内置`page`对象作为`javax.servlet.jsp.HttpJspPage`类的实例，代表当前的JSP页面，也表示当前JSP编译后的Servlet类的对象，就好比普通Java类中的this。

#### pageContext对象

在学习编程的时候，我们经常会碰到context这个词，什么process context，thread  context等等。现在的技术书籍中都把这个词叫做“上下文”，有的时候，“上下文”这个概念还是蛮难理解的，说白了“上下文”就是一个环境，这个环境中可能会包含很多的信息，例如变量、数据等信息。

在JSP的内置对象中，也有一个`pageContext`对象，这个对象代表着页面上下文，也就是当前页面所在的环境。`pageContext`对象是`javax.servlet.jsp.PageContext`类实例，我们可以通过这个对象来访问page、request、session和application作用域下的变量。

`pageContext`对象通过以下四个API来设置和获取页面上下文中的属性值（具体来说只有两个）：

| 名称                                                    | 描述                     |
| ------------------------------------------------------- | ------------------------ |
| Object getAttribute(String name)                        | 取得page范围内的name属性 |
| Object getAttribute(String name, int scope)             | 取得指定范围内的name属性 |
| void setAttribute(String name, Object value)            | 设置page范围内的name属性 |
| void setAttribute(String name, Object value, int scope) | 设置指定范围内的name属性 |

对于上述API中的参数scope可以取以下值：

```
public static final int PAGE_SCOPE          = 1; // 对应于page范围
public static final int REQUEST_SCOPE       = 2; // 对应于request范围
public static final int SESSION_SCOPE       = 3; // 对应于session范围
public static final int APPLICATION_SCOPE   = 4; // 对应于application范围
```

#### request对象

内置`request`对象是`javax.servlet.ServletRequest`类的实例，代表客户端发起的请求；这是一个非常重要的对象。在B/S架构中，客户端一般都是浏览器，用户通过浏览器向服务器发送请求，从而达到访问服务器的目的。而浏览器具体怎么访问服务器，得到自己需要的数据呢？这个过程复杂的去了，大体上分为以下三步：

1. 用户通过浏览器发送request；
2. 服务器处理request；
3. 将response传给客户端，由浏览器渲染显示。

这三大步，每一步分开总结都能成一篇文章；而这里只是对JSP中的内置`request`对象进行扫盲性总结。用户通过浏览器发送请求，浏览器会将请求信息都包装在一个请求中，发送给服务器端；服务器得到这个请求以后，会从这个请求从得到所有的请求信息，将这些请求信息封装成一个request对象，也就是JSP中内置的这个`request`对象。也就是说，这个`request`对象是对客户请求数据的封装，那么这个`request`对象到底封装了哪些数据呢？

对于一个请求中的数据主要分为两部分：

- 请求头，这部分数据通常由浏览器自动添加；
- 请求参数，这部分由用户自己在页面上添加；

下面通过一个最简单的页面来输出所有的请求头数据。一个最简单的HTML页面，给服务器发送一个不带任何请求参数的请求，到达服务器以后，服务器将所有的请求头数据返回客户端输出。

ndex.html主要代码：

```null
<body>
    <form action="page1.jsp">
        <input type="submit" value="获取请求头参数" />
    </form>
</body>
```

page1.jsp主要代码：

```null
<body>
    <%
    // 获得所有的请求头
    Enumeration<String> headerNames = request.getHeaderNames();
    while (headerNames.hasMoreElements())
    {
        String headerName = headerNames.nextElement();

        // 获得请求头对应的参数
        out.println("[" + headerName + "] => [" + request.getHeader(headerName) + "]<br />");
    }
    %>
</body>    
```

客户端向服务器发送的每一次请求都会把上述输出的请求头带上。而光有这些请求头数据，对于服务器来说用处不大，而我们需要收集用户的真正需求。

对于百度首页的那个搜索框来说，那只是一个简单的表单，用来收集用户的搜索信息，一旦用户提交了请求，表单里输入的搜索信息就会提交给百度的后台处理程序。此时你输入的搜索信息就是请求参数，对于服务器来说，这个请求参数才是我们需要的，下面就通过一个简单的程序来获取用户提交的请求参数。

index.html主要代码：

```null
<body>
    <form action="page1.jsp" method="get">
        <input type="text" name="search" />
        <input type="submit" value="百度一下" />
    </form>
</body>
```

page1.jsp主要代码：

```null
<%
// 获得输入的搜索字符串
out.println("你搜索的内容为：");
out.println(new String(request.getParameter("search").getBytes("ISO8859-1"),"UTF-8"));
%>
```

我们可以通过调用内置`request`对象的`getParameter`函数获取用户输入的信息，从而再完成其它一些功能。关于表单form中的method，有`GET`和`POST`两种

对于内置的`request`对象，也提供了以下两个函数：

- setAttribute(String attrName, Object value); 设置本次请求关联的属性值
- getAttribute(String attrName); 取本次请求关联属性值

#### response对象

JSP内置的`response`对象是`javax.servlet.http.HttpServletResponse`的实例，代表服务器对客户端的响应。在大多数情况下，我们向服务器发起一个请求，服务器都会响应客户端的请求，并将响应内容发送回客户端。但是在实际开发中，当我们需要向客户端传回什么内容的时候，要么使用内置的`out`对象，要么就使用输出表达式；而这个内置的`response`对象却非常少用。的确是这样的，内置的`out`对象和输出表达式基本上能够满足我们的需要，但是为什么还要有这个内置的`response`对象呢？

内置的`out`是JspWriter的实例（输出表达式本质上也是out），而JspWriter又是Writer的子类，由于Writer是字符流，所以无法输出非字符内容。如果我们需要在JSP页面中动态的生成一张图片，此时使用`out`对象则无法完成；又或我们需要将客户端的请求重定向到别的地方，内置的`out`也是无法完成的；又或向客户端增加Cookie，内置的`out`也是无法完成的。这都是内置`out`对象的限制，基于此，我们还是有必要来看看内置`response`对象的具体应用。

大家在一些网站进行登陆的时候，或者对文章发表评论时，都要求输入动态生成的验证码，而这个动态码是由服务器生成的一张动态码图片，然后发送到客户端的。由于这个动态码图片不是字符流，所以只能通过`response`对象来响应这类请求。

```
<%@ page language="java" contentType="image/jpeg"%>
<%@ page pageEncoding="UTF-8" %>
<%@ page import="java.awt.image.*, java.io.*, java.awt.*, java.awt.Color, java.util.Random" %>
<%@ page import="com.sun.image.codec.jpeg.JPEGCodec, com.sun.image.codec.jpeg.JPEGImageEncoder" %>


<!DOCTYPE>
<html>
<head>
<title>页面一</title>
</head>
<body>
    <%!
    // 随机字典，去掉了O,0,I,1这些难分辨的字符
    public static final char[] CHARS = {'2', '3', '4', '5', '6', '7', '8', '9', 
        'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'V', 'U', 'W', 'X', 'Y', 'Z'};

    public static Random random = new Random();

    // 获取随机6位随机数
    public static String getRandomString()
    {
        StringBuilder buffer = new StringBuilder();
        for (int i = 0; i < 6; ++i)
        {
            buffer.append(CHARS[random.nextInt(CHARS.length)]);
        }

        return buffer.toString();
    }

    // 获得随机的颜色
    public static Color getRandomColor()
    {
        return new Color(random.nextInt(255), random.nextInt(255), random.nextInt(255));
    }

    // 获得某个颜色的反颜色
    public static Color getReverseColor(Color c)
    {
        return new Color(255 - c.getRed(), 255 - c.getGreen(), 255 - c.getBlue());
    }
    %>

    <%
    String randomString = getRandomString();
    request.getSession(true).setAttribute("randomstring", randomString); // 将随机字符串放到Session中存起来

    int width = 100;
    int height = 30;

    Color bgColor = getRandomColor(); // 背景颜色
    Color frontColor = getReverseColor(bgColor);

    BufferedImage bi = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
    Graphics2D g = bi.createGraphics(); // 获取绘图对象
    g.setFont(new Font(Font.SANS_SERIF, Font.BOLD, 16)); // 设置字体
    g.setColor(bgColor);
    g.fillRect(0, 0, width, height);
    g.setColor(frontColor);
    g.drawString(randomString, 18, 20);

    // 绘制噪声点
    for (int i = 0, n = random.nextInt(100); i < n; ++i)
    {
        g.drawRect(random.nextInt(width), random.nextInt(height), 1, 1);
    }

    // 
    ServletOutputStream identifyPic = response.getOutputStream();

    ImageIO.write(bim, "JPEG", response.getOutputStream());
     
    identifyPic.flush();
    %>
</body>
</html>
```

当使用`response`对象向客户端输出二进制数据时，需要调用`getOutputStream`输出二进制数据。

**response实现重定向**

关于重定向，首先你需要明白重定向的原理。重定向是利用服务器返回的状态码来实现的，客户端向服务器发送请求的时候，服务器通过response的setStatus方法设置一个状态码（如果不设置，系统会默认设置），然后服务器会向客户端返回这个状态码。如果服务器返回了301或者302，则浏览器会到新的网址重新请求。关于返回码301和302的具体说明如下：

| 状态码 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| 301    | 永久重定向；请求的网页已永久移动到新的位置，当服务器返回此响应(作为一个GET或HEAD请求的响应)，它会自动转发请求到新的位置 |
| 302    | 临时重定向；指示资源临时在另一个位置，该位置通过Location指定 |

下面通过一段简单的代码来说明如何使用response对象来实现重定向。

```null
<%
response.setStatus(301); // 设置返回码
response.setHeader("Location", "http://www.jellythink.com");
%>
```

实现重定向就是两步：

1. 设置301或者302返回码；
2. 设置新的Location；

为了写出更简单的代码，将上述的两步封装成了一个函数：

```null
<%
// 返回码是302
response.sendRedirect("http://www.baidu.com");
%>
```

其实当使用重定向这种方法时，跳转是在客户端实现的。也就是客户端实际发起了两次请求，第一次请求取得了服务器返回的重定向码和重定向的地址；第二次访问真实地址。

**使用response增加Cookie**

我们在进行网站开发的时候，经常会使用Cookie来记录某些信息。一旦用户下次再访问该网站的时候，网站就可以自动获取之前记录的Cookie信息。比如你登陆新浪微博的时候，会提供是否记住账号的功能，当登陆成功以后，会将你的账号信息以Cookie的形式记录下来，当你打开浏览器再次访问新浪微博的时候，就可以免去输入账号密码，直接登陆新浪微博。

但是使用内置的`out`对象是无法完成向客户端增加Cookie的功能，只有`response`对象才能胜任这项工作。但是增加Cookie的过程比较繁琐的，主要分为以下几步：

1. 使用Cookie(String name, String value)构造函数，创建Cookie实例；
2. 设置Cookie的生命期限，即该Cookie可以“存活”多长时间；
3. 调用`response`对象的`addCookie(Cookie cookie)`向客户端写Cookie。

下面通过几个例子来说明如何增加Cookie。

一个最简单的登陆界面：

```null
<body>
    <form action="page1.jsp" method="post">
                    用户名:<input type="text" name="username" /><br />
                    密    码:<input type="password" name="password" /><br />
      <input type="submit" value="登陆" />
      <input type="reset" value="重置" />  
    </form>
</body>
```

action对应page1.jsp的主要代码：

```null
<%
    String userName = request.getParameter("username");
    String password = request.getParameter("password");

    // 创建Cookie对象
    Cookie cUserName = new Cookie("USERNAME", userName);
    Cookie cPassword = new Cookie("PASSWORD", password);

    // 先不设置Cookie的有效时间，直接向客户端设置Cookie
    response.addCookie(cUserName);
    response.addCookie(cPassword);
%>
```

运行程序以后，我们可以通过浏览器自带的工具查看添加的Cookie值；当我们关闭浏览器，再次访问的时候，发现之前记录的Cookie值不在了。这是为什么呢？

由于我们没有设置Cookie的有效时间，Cookie是记录在浏览器内存中的，随着一次会话的结束（关闭浏览器），对应的Cookie值也就都消失了。接下来，我们对Cookie添加有效时间。

```null
<%
    String userName = request.getParameter("username");
    String password = request.getParameter("password");

    // 创建Cookie对象
    Cookie cUserName = new Cookie("USERNAME", userName);
    Cookie cPassword = new Cookie("PASSWORD", password);

    // 设置有效时间3600秒
    cUserName.setMaxAge(3600);
    cPassword.setMaxAge(3600);

    // 向客户端设置Cookie
    response.addCookie(cUserName);
    response.addCookie(cPassword);
%>
```

这样的话，Cookie信息就记录到本地硬盘上，并不会随着浏览器的关闭而消失。同时，每次向服务器发送请求的时候，这些Cookie信息也会一起发送到服务器端。

#### session对象

内置的`session`对象是`javax.servlet.http.HttpSession`类的实例，主要用于在服务器端保存用户信息，它代表着一次用户与服务器之间的会话。这里涉及到一个会话的概念。

>    从客户端浏览器连接服务器开始，到客户端浏览器与服务器断开为止，这个过程统称为一次会话。 

其实更笼统的理解就是，你打开客户端浏览器，向服务器发送请求进行交互，到你关闭客户端浏览器，这就是一次会话。有的时候，服务器也会主动断开会话，比如服务器无法提供服务等等情况。

我们都知道HTTP是无状态的，也就是说你本次发送的请求，和你下次发送的请求，服务器无法知道这两个请求是来自同一个用户请求，还是分别来自两个不同用户的请求；而有的时候，我们却非常需要每个请求的状态信息，需要区分这些请求是否来自同一个用户，这就有了session这个概念。

session通常用于跟踪用户的状态，会话等一些比较敏感的数据信息，比如判断用户当前是否为登陆状态等。虽然session是一个非常重要的概念和对象，但是关于它的内容却没有多少，很多的时候，我们都是围绕着session这个概念，开发一些比较复杂的功能，比如单点登录、自动认证、登录等功能。

当你打开浏览器，向服务器发送请求，你发送多次不同的请求，服务器给你响应的是不同的页面，但这都属于一个session的范畴，这就意味着session范围内的属性在这些不同的页面之间是可以共享的；但是你需要记住，一旦你关闭浏览器，即当前session结束，对应该session范围内的属性将全部丢失。

对于`session`有以下两个常用方法：

- setAttribute(String attrName, Object attrValue)
- getAttribute(String attrName)

关于这两个方法的用法和`request`、`application`和`pageContext`对象设置属性、取属性的方式是一致的。下面通过一个小小的例子进行说明。

页面一（page1.jsp）主要代码：

```null
<%
session.setAttribute("defaultWebsite", "http://www.百度.com");
session.setAttribute("webInfo", "百度--中国最大中文忽悠网站");
%>
```

页面二（page2.jsp）主要代码：

```null
<%= session.getAttribute("defaultWebsite") %><br/>
<%= session.getAttribute("webInfo") %>
```

由于页面一和页面二之间没有任何关系，当我们通过客户端浏览器访问页面一以后，再去访问页面二，此时这两个页面属于同一个session的范畴，理所当然的页面二就可以获取页面一中session设置的属性。

