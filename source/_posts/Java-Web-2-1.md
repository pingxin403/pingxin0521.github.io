---
title: Java servlet (一)
date: 2019-05-15 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

### 什么是Servlet？

处理请求和发送响应的过程是由一种叫做Servlet的程序来完成的，并且Servlet是为了解决实现动态页面而衍生的东西。理解这个的前提是了解一些http协议的东西，并且知道B/S模式(浏览器/服务器)。

<!--more-->

Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：

- 性能明显更好。
- Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
- Servlet 是独立于平台的，因为它们是用 Java 编写的。
- 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
- Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

Servlet架构：

![2.jpg](https://i.loli.net/2019/05/21/5ce370cc1b16354955.jpg)

##### Servlet 任务

Servlet 执行以下主要任务：

- 读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
- 读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等。
- 处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
- 发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
- 发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

##### Servlet 包

Java Servlet 是运行在带有支持 Java Servlet 规范的解释器的 web 服务器上的 Java 类。

Servlet 可以使用 **javax.servlet** 和 **javax.servlet.http** 包创建，它是 Java 企业版的标准组成部分，Java 企业版是支持大型开发项目的 Java 类库的扩展版本。

这些类实现 Java Servlet 和 JSP 规范。在写本教程的时候，二者相应的版本分别是 Java Servlet 2.5 和 JSP 2.1。

Java Servlet 就像任何其他的 Java 类一样已经被创建和编译。在您安装 Servlet 包并把它们添加到您的计算机上的 Classpath 类路径中之后，您就可以通过 JDK 的 Java 编译器或任何其他编译器来编译 Servlet。

#### tomcat和servlet的关系

Tomcat 是Web应用服务器,是一个Servlet/JSP容器. Tomcat 作为Servlet容器,负责处理客户请求,把请求传送给Servlet,并将Servlet的响应传送回给客户.而Servlet是一种运行在支持Java语言的服务器上的组件. Servlet最常见的用途是扩展Java Web服务器功能,提供非常安全的,可移植的,易于使用的CGI替代品.

Java Servlet API 是Servlet容器(tomcat)和servlet之间的接口，它定义了serlvet的各种方法，还定义了Servlet容器传送给Servlet的对象类，其中最重要的就是ServletRequest和ServletResponse。所以说我们在编写servlet时，需要实现Servlet接口，按照其规范进行操作。

1. Tomcat将http请求文本接收并解析，然后封装成HttpServletRequest类型的request对象，所有的HTTP头数据读可以通过request对象调用对应的方法查询到。

2. Tomcat同时会要响应的信息封装为HttpServletResponse类型的response对象，通过设置response属性就可以控制要输出到浏览器的内容，然后将response交给tomcat，tomcat就会将其变成响应文本的格式发送给浏览器

### Servlet生命周期

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

- Servlet 通过调用 **init ()** 方法进行初始化。
- Servlet 调用 **service()** 方法来处理客户端的请求。
- Servlet 通过调用 **destroy()** 方法终止（结束）。
- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

现在让我们详细讨论生命周期的方法。

##### init() 方法

init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。因此，它是用于一次性初始化，就像 Applet 的 init 方法一样。

Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

```
public void init() throws ServletException {
  // 初始化代码...
}
```

##### service() 方法

service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP  请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

下面是该方法的特征：

```
public void service(ServletRequest request, 
                    ServletResponse response) 
      throws ServletException, IOException{
}
```

service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete  等方法。所以，您不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

doGet() 和 doPost() 方法是每次服务请求中最常用的方法。下面是这两种方法的特征。

##### doGet() 方法

GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理。

```
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

##### doPost() 方法

POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理。

```
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

##### destroy() 方法

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：

```
  public void destroy() {
    // 终止化代码...
  }
```

#### 架构图

下图显示了一个典型的 Servlet 生命周期方案。

- 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
- Servlet 容器在调用 service() 方法之前加载 Servlet。
- 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法

![3.jpg](https://i.loli.net/2019/05/21/5ce372767e79297979.jpg)

#### HelloWorld示例

新建maven项目

![](https://i.loli.net/2019/05/21/5ce37f1a006a377056.png)

maven依赖：

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
```

添加插件：

```
<plugin>
<groupId>org.apache.tomcat.maven</groupId>
<artifactId>tomcat7-maven-plugin</artifactId>
<version>2.2</version>
</plugin>
```

ServletTest类：

```
public class ServletTest extends HttpServlet {

    private String message;

    @Override
    public void init() throws ServletException {
        super.init();
        message = "Hello World , Nect To Meet You: " + System.currentTimeMillis();
        System.out.println("servlet初始化……");
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        super.doPost(request,response);
        System.out.println("post");
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.println("<h2>"+message+"</h2>");
        destroy();

    }


    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       doPost(request,response);
    }

    @Override
    public void destroy() {
        System.out.println("servlet销毁！");
        super.destroy();
    }
}
```

web.xml文件：

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Servlet test</display-name>


  <servlet>
    <servlet-name>HelloWorld</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.ServletTest</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>HelloWorld</servlet-name>
    <url-pattern>/HelloWorld</url-pattern>
  </servlet-mapping>

  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>

```

打开终端，运行使用：`mvn tomcat7:run`

### 表单数据

很多情况下，需要传递一些信息，从浏览器到 Web 服务器，最终到后台程序。浏览器使用两种方法可将这些信息传递到 Web 服务器，分别为 GET 方法和 POST 方法。

#### GET 方法

GET 方法向页面请求发送已编码的用户信息。页面和已编码的信息中间用 ? 字符分隔，如下所示：

```
http://www.test.com/hello?key1=value1&key2=value2
```

GET 方法是默认的从浏览器向 Web 服务器传递信息的方法，它会产生一个很长的字符串，出现在浏览器的地址栏中。如果您要向服务器传递的是密码或其他的敏感信息，请不要使用 GET 方法。GET 方法有大小限制：请求字符串中最多只能有 1024 个字符。

这些信息使用 QUERY_STRING  头传递，并可以通过 QUERY_STRING 环境变量访问，Servlet 使用 **doGet()** 方法处理这种类型的请求。

#### POST 方法

另一个向后台程序传递信息的比较可靠的方法是 POST 方法。POST 方法打包信息的方式与 GET 方法基本相同，但是 POST  方法不是把信息作为 URL 中 ?  字符后的文本字符串进行发送，而是把这些信息作为一个单独的消息。消息以标准输出的形式传到后台程序，您可以解析和使用这些标准输出。Servlet  使用 doPost() 方法处理这种类型的请求。

#### 使用 Servlet 读取表单数据

Servlet 处理表单数据，这些数据会根据不同的情况使用不同的方法自动解析：

- **getParameter()：**您可以调用 request.getParameter() 方法来获取表单参数的值。
- **getParameterValues()：**如果参数出现一次以上，则调用该方法，并返回多个值，例如复选框。
- **getParameterNames()：**如果您想要得到当前请求中的所有参数的完整列表，则调用该方法。

#### 使用 URL 的 GET 方法实例

下面是一个简单的 URL，将使用 GET 方法向 HelloForm 程序传递两个值。

```
http://localhost:8080/javaweb/HelloForm?name=%E5%B9%B3%E5%BF%83&url=www.hanyunpeng0521.github.io
```

下面是处理 Web 浏览器输入的 **HelloForm.java** Servlet 程序。我们将使用 **getParameter()**  方法，可以很容易地访问传递的信息：

```
public class HelloForm extends HttpServlet {
    private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public HelloForm() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "使用 GET 方法读取表单数据";
        // 处理中文
        String name =new String(request.getParameter("name").getBytes("ISO8859-1"),"UTF-8");
        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<ul>\n" +
                "  <li><b>站点名</b>："
                + name + "\n" +
                "  <li><b>网址</b>："
                + request.getParameter("url") + "\n" +
                "</ul>\n" +
                "</body></html>");
    }

    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

然后我们在 **web.xml** 文件中添加：

```
  <servlet>
    <servlet-name>HelloForm</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.HelloForm</servlet-class>
  </servlet>
<servlet-mapping>
    <servlet-name>HelloForm</servlet-name>
    <url-pattern>/HelloForm</url-pattern>
  </servlet-mapping>
```

#### 使用表单的 GET 方法实例

下面是一个简单的实例，使用 HTML 表单和提交按钮传递两个值。我们将使用相同的 Servlet HelloForm 来处理输入。

保存这个 HTML 到 hello.html 文件中，位于webapp下

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello From</title>
</head>
<body>
<form action="HelloForm" method="GET">
    网址名：<label>
    <input type="text" name="name">
</label>
    <br/>
    网址：<label>
    <input type="text" name="url"/>
</label>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```

运行，访问如下：

```
http://localhost:8080/javaweb/hello.html
```

输入数据，提交后就会跳转到上面的页面。查看页面URL：

```
http://localhost:8080/javaweb/HelloForm?name=%E5%B9%B3%E5%BF%83&url=www.hanyunpeng0521.github.io
```

#### 使用表单的 POST 方法实例

让我们对上面的 Servlet 做小小的修改，以便它可以处理 GET 和 POST 方法。下面的 **HelloForm.java** Servlet 程序使用 GET 和 POST 方法处理由 Web 浏览器给出的输入。

注意：如果表单提交的数据中有中文数据则需要转码：

```
String name =new String(request.getParameter("name").getBytes("ISO8859-1"),"UTF-8");
```

更改上面得HelloFrom.java，修改doPost方法：

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "使用 POST 方法读取表单数据";
        // 处理中文
        String name =new String(request.getParameter("name").getBytes("ISO8859-1"),"UTF-8");
        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
            "<html>\n" +
            "<head><title>" + title + "</title></head>\n" +
            "<body bgcolor=\"#f0f0f0\">\n" +
            "<h1 align=\"center\">" + title + "</h1>\n" +
            "<ul>\n" +
            "  <li><b>站点名</b>："
            + name + "\n" +
            "  <li><b>网址</b>："
            + request.getParameter("url") + "\n" +
            "</ul>\n" +
            "</body></html>");
    }
```

现在，编译部署上述的 Servlet，并使用带有 POST 方法的 hello.html 进行测试，如下所示：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello From</title>
</head>
<body>
<form action="HelloForm" method="POST">
    网址名：<label>
    <input type="text" name="name">
</label>
    <br/>
    网址：<label>
    <input type="text" name="url"/>
</label>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```

运行后，访问以下链接：

```
http://localhost:8080/javaweb/hello.html
```

填入数据后，提交，查看URL：

```
http://localhost:8080/javaweb/HelloForm
```

#### 将复选框数据传递到 Servlet 程序

当需要选择一个以上的选项时，则使用复选框。

下面是一个 HTML 代码实例 checkbox.html，一个带有两个复选框的表单。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>check box</title>
</head>
<body>
<form action="CheckBox" method="POST" target="_blank">
    <label>
        <input type="checkbox" name="runoob" checked="checked" />
    </label> 平心
    <label>
        <input type="checkbox" name="google"  />
    </label> Google
    <label>
        <input type="checkbox" name="taobao" checked="checked" />
    </label> 淘宝
    <input type="submit" value="选择站点" />
</form>
</body>
</html>
```

新建CheckBox.java文件：

```
public class CheckBox extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "读取复选框数据";
        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<ul>\n" +
                "  <li><b>平心标识：</b>: "
                + request.getParameter("runoob") + "\n" +
                "  <li><b>Google 标识：</b>: "
                + request.getParameter("google") + "\n" +
                "  <li><b>淘宝标识：</b>: "
                + request.getParameter("taobao") + "\n" +
                "</ul>\n" +
                "</body></html>");
    }

    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

修改web.xml：

```
<servlet>
    <servlet-name>CheckBox</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.CheckBox</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>CheckBox</servlet-name>
    <url-pattern>/CheckBox</url-pattern>
  </servlet-mapping>
```

运行后，访问以下链接：

```
http://localhost:8080/javaweb/checkbox.html
```

#### 读取所有的表单参数

以下是通用的实例，使用 HttpServletRequest 的 **getParameterNames()** 方法读取所有可用的表单参数。该方法返回一个枚举，其中包含未指定顺序的参数名。

一旦我们有一个枚举，我们可以以标准方式循环枚举，使用 *hasMoreElements()* 方法来确定何时停止，使用 *nextElement()* 方法来获取每个参数的名称。

```
public class ReadParams extends HttpServlet {
    private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public ReadParams() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        String title = "读取所有的表单数据";
        String docType =
                "<!doctype html public \"-//w3c//dtd html 4.0 " +
                        "transitional//en\">\n";
        out.println(docType +
                "<html>\n" +
                "<head><meta charset=\"utf-8\"><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<table width=\"100%\" border=\"1\" align=\"center\">\n" +
                "<tr bgcolor=\"#949494\">\n" +
                "<th>参数名称</th><th>参数值</th>\n"+
                "</tr>\n");

        Enumeration paramNames = request.getParameterNames();

        while(paramNames.hasMoreElements()) {
            String paramName = (String)paramNames.nextElement();
            out.print("<tr><td>" + paramName + "</td>\n");
            String[] paramValues =
                    request.getParameterValues(paramName);
            // 读取单个值的数据
            if (paramValues.length == 1) {
                String paramValue = paramValues[0];
                if (paramValue.length() == 0)
                    out.println("<td><i>没有值</i></td>");
                else
                    out.println("<td>" + paramValue + "</td>");
            } else {
                // 读取多个值的数据
                out.println("<td><ul>");
                for(int i=0; i < paramValues.length; i++) {
                    out.println("<li>" + paramValues[i]);
                }
                out.println("</ul></td>");
            }
            out.print("</tr>");
        }
        out.println("\n</table>\n</body></html>");
    }

    /**
     * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     */
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        doGet(request, response);
    }
}

```

现在，通过下面的表单尝试上面的 Servlet：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>test Form</title>
</head>
<body>

<form action="ReadParams" method="POST" target="_blank">
    <label>
        <input type="checkbox" name="maths" checked="checked" />
    </label> 数学
    <label>
        <input type="checkbox" name="physics"  />
    </label> 物理
    <label>
        <input type="checkbox" name="chemistry" checked="checked" />
    </label> 化学
    <input type="submit" value="选择学科" />
</form>

</body>
</html>
```

web.xml中添加：

```
  <servlet>
    <servlet-name>ReadParams</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.ReadParams</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>ReadParams</servlet-name>
    <url-pattern>/ReadParams</url-pattern>
  </servlet-mapping>
```

您可以尝试使用上面的 Servlet 来读取其他的表单数据，比如文本框、单选按钮或下拉框等。

### HTTP

#### HTTP报文格式

HTTP报文是HTTP应用程序之间传输的数据块，HTTP报文分为HTTP请求报文和HTTP响应报文，但是无论哪种报文，他的整体格式是类似的，大致都是由起始、首部、主体三部分组成，起始说明报文的动作，首部说明报文的属性，主体则是报文的数据。接下来具体说明。

**HTTP请求报文**

![](https://i.loli.net/2019/06/27/5d14ba61393f087976.png)

请求报文的起始由请求行构成（有些资料称为状态行，名字不一样而已，都是指的一个东西），用来说明该请求想要做什么，由`<Method>、<URL>、<Version>` 三个字段组成，注意每个字段之间都有一个空格。

其中<Method\>字段有不同的值：

- GET   --- 访问服务器的资源

- POST  --- 向服务器发送要修改的数据

- HEAD  --- 获取服务器文档的首部

- PUT   --- 向服务器上传资源

- DELETE--- 删除服务器的资源

 <URL\>字段表示服务器的资源目录定位

<Version\>字段表示使用的http协议版本

首部部分由多个请求头（也叫首部行）构成，那些首部字段名有如下，不全：

```
Accept     指定客户端能够接收的内容格式类型
Accept-Language 指定客户端能够接受的语言类型
Accept-Ecoding  指定客户端能够接受的编码类型
User-Agent      用户代理，向服务器说明自己的操作系统、浏览器等信息
Connection      是否开启持久连接（keepalive）
Host            服务器域名
...
```

主体部分就是报文的具体数据

**HTTP响应报文**

![](https://i.loli.net/2019/06/27/5d14bb0c2c08d43836.png)

响应报文的起始由状态行构成，用来说明服务器做了什么，由`<Version>、<Status-Code>、<Phrase>`三个字段组成，同样的每个字段之间留有空格；

<Status-Code\> 上边已经说明； 

首部由多个响应头(也叫首部行)组成， 首部字段名如下，不全：

```
Server    服务器软件名，Apache/Nginx

Date      服务器发出响应报文的时间

Last-Modified   请求资源的最后的修改时间

...
```

主体部分是响应报文的具体数据。

小tips：关于更多请求头和响应头（即首部字段名）的说明请参考<http://tools.jb51.net/table/http_header>

#### Servlet 客户端 HTTP 请求

当浏览器请求网页时，它会向 Web 服务器发送特定信息，这些信息不能被直接读取，因为这些信息是作为 HTTP 请求的头的一部分进行传输的。您可以查看 [HTTP 协议](https://www.runoob.com/http/http-tutorial.html) 了解更多相关信息。

以下是来自于浏览器端的重要头信息，您可以在 Web 编程中频繁使用

| 头信息              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| Accept              | 这个头信息指定浏览器或其他客户端可以处理的 MIME 类型。值 **image/png** 或 **image/jpeg** 是最常见的两种可能值。 |
| Accept-Charset      | 这个头信息指定浏览器可以用来显示信息的字符集。例如 ISO-8859-1。 |
| Accept-Encoding     | 这个头信息指定浏览器知道如何处理的编码类型。值 **gzip** 或 **compress** 是最常见的两种可能值。 |
| Accept-Language     | 这个头信息指定客户端的首选语言，在这种情况下，Servlet 会产生多种语言的结果。例如，en、en-us、ru 等。 |
| Authorization       | 这个头信息用于客户端在访问受密码保护的网页时识别自己的身份。 |
| Connection          | 这个头信息指示客户端是否可以处理持久 HTTP 连接。持久连接允许客户端或其他浏览器通过单个请求来检索多个文件。值 **Keep-Alive** 意味着使用了持续连接。 |
| Content-Length      | 这个头信息只适用于 POST 请求，并给出 POST 数据的大小（以字节为单位）。 |
| Cookie              | 这个头信息把之前发送到浏览器的 cookies 返回到服务器。        |
| Host                | 这个头信息指定原始的 URL 中的主机和端口。                    |
| If-Modified-Since   | 这个头信息表示只有当页面在指定的日期后已更改时，客户端想要的页面。如果没有新的结果可以使用，服务器会发送一个 304 代码，表示 **Not Modified** 头信息。 |
| If-Unmodified-Since | 这个头信息是 If-Modified-Since 的对立面，它指定只有当文档早于指定日期时，操作才会成功。 |
| Referer             | 这个头信息指示所指向的 Web 页的 URL。例如，如果您在网页 1，点击一个链接到网页 2，当浏览器请求网页 2 时，网页 1 的 URL 就会包含在 Referer 头信息中。 |
| User-Agent          | 这个头信息识别发出请求的浏览器或其他客户端，并可以向不同类型的浏览器返回不同的内容。 |

##### 读取 HTTP 头的方法

下面的方法可用在 Servlet 程序中读取 HTTP 头。这些方法通过 *HttpServletRequest* 对象可用。

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **Cookie[] getCookies()** 返回一个数组，包含客户端发送该请求的所有的 Cookie 对象。 |
| 2    | **Enumeration getAttributeNames()** 返回一个枚举，包含提供给该请求可用的属性名称。 |
| 3    | **Enumeration getHeaderNames()** 返回一个枚举，包含在该请求中包含的所有的头名。 |
| 4    | **Enumeration getParameterNames()** 返回一个 String 对象的枚举，包含在该请求中包含的参数的名称。 |
| 5    | **HttpSession getSession()** 返回与该请求关联的当前 session 会话，或者如果请求没有 session 会话，则创建一个。 |
| 6    | **HttpSession getSession(boolean create)** 返回与该请求关联的当前 HttpSession，或者如果没有当前会话，且创建是真的，则返回一个新的 session 会话。 |
| 7    | **Locale getLocale()** 基于 Accept-Language 头，返回客户端接受内容的首选的区域设置。 |
| 8    | **Object getAttribute(String name)** 以对象形式返回已命名属性的值，如果没有给定名称的属性存在，则返回 null。 |
| 9    | **ServletInputStream getInputStream()** 使用 ServletInputStream，以二进制数据形式检索请求的主体。 |
| 10   | **String getAuthType()** 返回用于保护 Servlet 的身份验证方案的名称，例如，"BASIC" 或 "SSL"，如果JSP没有受到保护则返回 null。 |
| 11   | **String getCharacterEncoding()** 返回请求主体中使用的字符编码的名称。 |
| 12   | **String getContentType()** 返回请求主体的 MIME 类型，如果不知道类型则返回 null。 |
| 13   | **String getContextPath()** 返回指示请求上下文的请求 URI 部分。 |
| 14   | **String getHeader(String name)** 以字符串形式返回指定的请求头的值。 |
| 15   | **String getMethod()** 返回请求的 HTTP 方法的名称，例如，GET、POST 或 PUT。 |
| 16   | **String getParameter(String name)** 以字符串形式返回请求参数的值，或者如果参数不存在则返回 null。 |
| 17   | **String getPathInfo()** 当请求发出时，返回与客户端发送的 URL 相关的任何额外的路径信息。 |
| 18   | **String getProtocol()** 返回请求协议的名称和版本。          |
| 19   | **String getQueryString()** 返回包含在路径后的请求 URL 中的查询字符串。 |
| 20   | **String getRemoteAddr()** 返回发送请求的客户端的互联网协议（IP）地址。 |
| 21   | **String getRemoteHost()** 返回发送请求的客户端的完全限定名称。 |
| 22   | **String getRemoteUser()** 如果用户已通过身份验证，则返回发出请求的登录用户，或者如果用户未通过身份验证，则返回 null。 |
| 23   | **String getRequestURI()** 从协议名称直到 HTTP 请求的第一行的查询字符串中，返回该请求的 URL 的一部分。 |
| 24   | **String getRequestedSessionId()** 返回由客户端指定的 session 会话 ID。 |
| 25   | **String getServletPath()** 返回调用 JSP 的请求的 URL 的一部分。 |
| 26   | **String[] getParameterValues(String name)** 返回一个字符串对象的数组，包含所有给定的请求参数的值，如果参数不存在则返回 null。 |
| 27   | **boolean isSecure()** 返回一个布尔值，指示请求是否使用安全通道，如 HTTPS。 |
| 28   | **int getContentLength()** 以字节为单位返回请求主体的长度，并提供输入流，或者如果长度未知则返回 -1。 |
| 29   | **int getIntHeader(String name)** 返回指定的请求头的值为一个 int 值。 |
| 30   | **int getServerPort()** 返回接收到这个请求的端口号。         |
| 31   | **int getParameterMap()** 将参数封装成 Map 类型。            |

##### HTTP Header 请求实例

下面的实例使用 HttpServletRequest 的 **getHeaderNames()** 方法读取 HTTP 头信息。该方法返回一个枚举，包含与当前的 HTTP 请求相关的头信息。

一旦我们有一个枚举，我们可以以标准方式循环枚举，使用 *hasMoreElements()* 方法来确定何时停止，使用 *nextElement()* 方法来获取每个参数的名称。

```
public class DisplayHeader extends HttpServlet {
    // 处理 GET 方法请求的方法
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException
    {
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "HTTP Header 请求实例";
        String docType =
                "<!DOCTYPE html> \n";
        out.println(docType +
                "<html>\n" +
                "<head><meta charset=\"utf-8\"><title>" + title + "</title></head>\n"+
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<table width=\"100%\" border=\"1\" align=\"center\">\n" +
                "<tr bgcolor=\"#949494\">\n" +
                "<th>Header 名称</th><th>Header 值</th>\n"+
                "</tr>\n");

        Enumeration headerNames = request.getHeaderNames();

        while(headerNames.hasMoreElements()) {
            String paramName = (String)headerNames.nextElement();
            out.print("<tr><td>" + paramName + "</td>\n");
            String paramValue = request.getHeader(paramName);
            out.println("<td> " + paramValue + "</td></tr>\n");
        }
        out.println("</table>\n</body></html>");
    }
    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}

```

以上测试实例是位于 javaweb 项目下，对应的 web.xml 配置添加：

```
  <servlet>
    <servlet-name>DisplayHeader</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.DisplayHeader</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>DisplayHeader</servlet-name>
    <url-pattern>/DisplayHeader</url-pattern>
```

访问URL：

```
http://localhost:8080/javaweb/DisplayHeader
```

#### Servlet 服务器 HTTP 响应

正如前面的章节中讨论的那样，当一个 Web 服务器响应一个 HTTP 请求时，响应通常包括一个状态行、一些响应报头、一个空行和文档。一个典型的响应如下所示：

```
HTTP/1.1 200 OK
Content-Type: text/html
Header2: ...
...
HeaderN: ...
  (Blank Line)
<!doctype ...>
<html>
<head>...</head>
<body>
...
</body>
</html>
```

状态行包括 HTTP 版本（在本例中为 HTTP/1.1）、一个状态码（在本例中为 200）和一个对应于状态码的短消息（在本例中为 OK）。

下表总结了从 Web 服务器端返回到浏览器的最有用的 HTTP 1.1 响应报头，您会在 Web 编程中频繁地使用它们

| 头信息              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| Allow               | 这个头信息指定服务器支持的请求方法（GET、POST 等）。         |
| Cache-Control       | 这个头信息指定响应文档在何种情况下可以安全地缓存。可能的值有：**public、private** 或 **no-cache** 等。Public 意味着文档是可缓存，Private 意味着文档是单个用户私用文档，且只能存储在私有（非共享）缓存中，no-cache 意味着文档不应被缓存。 |
| Connection          | 这个头信息指示浏览器是否使用持久 HTTP 连接。值 **close** 指示浏览器不使用持久 HTTP 连接，值 **keep-alive** 意味着使用持久连接。 |
| Content-Disposition | 这个头信息可以让您请求浏览器要求用户以给定名称的文件把响应保存到磁盘。 |
| Content-Encoding    | 在传输过程中，这个头信息指定页面的编码方式。                 |
| Content-Language    | 这个头信息表示文档编写所使用的语言。例如，en、en-us、ru 等。 |
| Content-Length      | 这个头信息指示响应中的字节数。只有当浏览器使用持久（keep-alive）HTTP 连接时才需要这些信息。 |
| Content-Type        | 这个头信息提供了响应文档的 MIME（Multipurpose Internet Mail Extension）类型。 |
| Expires             | 这个头信息指定内容过期的时间，在这之后内容不再被缓存。       |
| Last-Modified       | 这个头信息指示文档的最后修改时间。然后，客户端可以缓存文件，并在以后的请求中通过 **If-Modified-Since** 请求头信息提供一个日期。 |
| Location            | 这个头信息应被包含在所有的带有状态码的响应中。在 300s 内，这会通知浏览器文档的地址。浏览器会自动重新连接到这个位置，并获取新的文档。 |
| Refresh             | 这个头信息指定浏览器应该如何尽快请求更新的页面。您可以指定页面刷新的秒数。 |
| Retry-After         | 这个头信息可以与 503（Service Unavailable 服务不可用）响应配合使用，这会告诉客户端多久就可以重复它的请求。 |
| Set-Cookie          | 这个头信息指定一个与页面关联的 cookie。                      |

##### 设置 HTTP 响应报头的方法

下面的方法可用于在 Servlet 程序中设置 HTTP 响应报头。这些方法通过 *HttpServletResponse* 对象可用。

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **String encodeRedirectURL(String url)** 为 sendRedirect 方法中使用的指定的 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。 |
| 2    | **String encodeURL(String url)** 对包含 session 会话 ID 的指定 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。 |
| 3    | **boolean containsHeader(String name)** 返回一个布尔值，指示是否已经设置已命名的响应报头。 |
| 4    | **boolean isCommitted()** 返回一个布尔值，指示响应是否已经提交。 |
| 5    | **void addCookie(Cookie cookie)** 把指定的 cookie 添加到响应。 |
| 6    | **void addDateHeader(String name, long date)** 添加一个带有给定的名称和日期值的响应报头。 |
| 7    | **void addHeader(String name, String value)** 添加一个带有给定的名称和值的响应报头。 |
| 8    | **void addIntHeader(String name, int value)** 添加一个带有给定的名称和整数值的响应报头。 |
| 9    | **void flushBuffer()** 强制任何在缓冲区中的内容被写入到客户端。 |
| 10   | **void reset()** 清除缓冲区中存在的任何数据，包括状态码和头。 |
| 11   | **void resetBuffer()** 清除响应中基础缓冲区的内容，不清除状态码和头。 |
| 12   | **void sendError(int sc)** 使用指定的状态码发送错误响应到客户端，并清除缓冲区。 |
| 13   | **void sendError(int sc, String msg)** 使用指定的状态发送错误响应到客户端。 |
| 14   | **void sendRedirect(String location)** 使用指定的重定向位置 URL 发送临时重定向响应到客户端。 |
| 15   | **void setBufferSize(int size)** 为响应主体设置首选的缓冲区大小。 |
| 16   | **void setCharacterEncoding(String charset)** 设置被发送到客户端的响应的字符编码（MIME 字符集）例如，UTF-8。 |
| 17   | **void setContentLength(int len)** 设置在 HTTP Servlet 响应中的内容主体的长度，该方法设置 HTTP Content-Length 头。 |
| 18   | **void setContentType(String type)** 如果响应还未被提交，设置被发送到客户端的响应的内容类型。 |
| 19   | **void setDateHeader(String name, long date)** 设置一个带有给定的名称和日期值的响应报头。 |
| 20   | **void setHeader(String name, String value)** 设置一个带有给定的名称和值的响应报头。 |
| 21   | **void setIntHeader(String name, int value)** 设置一个带有给定的名称和整数值的响应报头。 |
| 22   | **void setLocale(Locale loc)** 如果响应还未被提交，设置响应的区域。 |
| 23   | **void setStatus(int sc)** 为该响应设置状态码。              |

##### HTTP Header 响应实例

您已经在前面的实例中看到 setContentType() 方法，下面的实例也使用了同样的方法，此外，我们会用 **setIntHeader()** 方法来设置 **Refresh** 头。

```
public class Refresh extends HttpServlet {
    // 处理 GET 方法请求的方法
    public void doGet(HttpServletRequest request,
                      HttpServletResponse response)
            throws ServletException, IOException
    {
        // 设置刷新自动加载时间为 3 秒
        response.setIntHeader("Refresh", 3);
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        //使用默认时区和语言环境获得一个日历  
        Calendar cale = Calendar.getInstance();
        //将Calendar类型转换成Date类型  
        Date tasktime=cale.getTime();
        //设置日期输出的格式  
        SimpleDateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //格式化输出  
        String nowTime = df.format(tasktime);
        PrintWriter out = response.getWriter();
        String title = "自动刷新 Header 设置";
        String docType =
                "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n"+
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<p>当前时间是：" + nowTime + "</p>\n");
    }
    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request,
                       HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```

以上测试实例是位于 javaweb 项目下，对应的 web.xml 配置添加：

```
  <servlet>
    <servlet-name>Refresh</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.Refresh</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>Refresh</servlet-name>
    <url-pattern>/Refresh</url-pattern>
  </servlet-mapping>
```

访问URL：

```
http://localhost:8080/javaweb/Refresh
```

#### Servlet HTTP 状态码

HTTP 请求和 HTTP 响应消息的格式是类似的，结构如下：

- 初始状态行 + 回车换行符（回车+换行）
- 零个或多个标题行+回车换行符
- 一个空白行，即回车换行符
- 一个可选的消息主体，比如文件、查询数据或查询输出

例如，服务器的响应头如下所示：

```
HTTP/1.1 200 OK
Content-Type: text/html
Header2: ...
...
HeaderN: ...
  (Blank Line)
<!doctype ...>
<html>
<head>...</head>
<body>
...
</body>
</html>
```

状态行包括 HTTP 版本（在本例中为 HTTP/1.1）、一个状态码（在本例中为 200）和一个对应于状态码的短消息（在本例中为 OK）。

以下是可能从 Web 服务器返回的 HTTP 状态码和相关的信息列表：

| 代码 | 消息                          | 描述                                                         |
| ---- | ----------------------------- | ------------------------------------------------------------ |
| 100  | Continue                      | 只有请求的一部分已经被服务器接收，但只要它没有被拒绝，客户端应继续该请求。 |
| 101  | Switching Protocols           | 服务器切换协议。                                             |
| 200  | OK                            | 请求成功。                                                   |
| 201  | Created                       | 该请求是完整的，并创建一个新的资源。                         |
| 202  | Accepted                      | 该请求被接受处理，但是该处理是不完整的。                     |
| 203  | Non-authoritative Information |                                                              |
| 204  | No Content                    |                                                              |
| 205  | Reset Content                 |                                                              |
| 206  | Partial Content               |                                                              |
| 300  | Multiple Choices              | 链接列表。用户可以选择一个链接，进入到该位置。最多五个地址。 |
| 301  | Moved Permanently             | 所请求的页面已经转移到一个新的 URL。                         |
| 302  | Found                         | 所请求的页面已经临时转移到一个新的 URL。                     |
| 303  | See Other                     | 所请求的页面可以在另一个不同的 URL 下被找到。                |
| 304  | Not Modified                  |                                                              |
| 305  | Use Proxy                     |                                                              |
| 306  | *Unused*                      | 在以前的版本中使用该代码。现在已不再使用它，但代码仍被保留。 |
| 307  | Temporary Redirect            | 所请求的页面已经临时转移到一个新的 URL。                     |
| 400  | Bad Request                   | 服务器不理解请求。                                           |
| 401  | Unauthorized                  | 所请求的页面需要用户名和密码。                               |
| 402  | Payment Required              | *您还不能使用该代码。*                                       |
| 403  | Forbidden                     | 禁止访问所请求的页面。                                       |
| 404  | Not Found                     | 服务器无法找到所请求的页面。.                                |
| 405  | Method Not Allowed            | 在请求中指定的方法是不允许的。                               |
| 406  | Not Acceptable                | 服务器只生成一个不被客户端接受的响应。                       |
| 407  | Proxy Authentication Required | 在请求送达之前，您必须使用代理服务器的验证。                 |
| 408  | Request Timeout               | 请求需要的时间比服务器能够等待的时间长，超时。               |
| 409  | Conflict                      | 请求因为冲突无法完成。                                       |
| 410  | Gone                          | 所请求的页面不再可用。                                       |
| 411  | Length Required               | "Content-Length" 未定义。服务器无法处理客户端发送的不带 Content-Length 的请求信息。 |
| 412  | Precondition Failed           | 请求中给出的先决条件被服务器评估为 false。                   |
| 413  | Request Entity Too Large      | 服务器不接受该请求，因为请求实体过大。                       |
| 414  | Request-url Too Long          | 服务器不接受该请求，因为 URL 太长。当您转换一个 "post" 请求为一个带有长的查询信息的 "get" 请求时发生。 |
| 415  | Unsupported Media Type        | 服务器不接受该请求，因为媒体类型不被支持。                   |
| 417  | Expectation Failed            |                                                              |
| 500  | Internal Server Error         | 未完成的请求。服务器遇到了一个意外的情况。                   |
| 501  | Not Implemented               | 未完成的请求。服务器不支持所需的功能。                       |
| 502  | Bad Gateway                   | 未完成的请求。服务器从上游服务器收到无效响应。               |
| 503  | Service Unavailable           | 未完成的请求。服务器暂时超载或死机。                         |
| 504  | Gateway Timeout               | 网关超时。                                                   |
| 505  | HTTP Version Not Supported    | 服务器不支持"HTTP协议"版本。                                 |

##### 设置 HTTP 状态代码的方法

下面的方法可用于在 Servlet 程序中设置 HTTP 状态码。这些方法通过 *HttpServletResponse* 对象可用。

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **public void setStatus ( int statusCode )** 该方法设置一个任意的状态码。setStatus 方法接受一个 int（状态码）作为参数。如果您的响应包含了一个特殊的状态码和文档，请确保在使用 *PrintWriter* 实际返回任何内容之前调用 setStatus。 |
| 2    | **public void sendRedirect(String url)** 该方法生成一个 302 响应，连同一个带有新文档 URL 的 *Location* 头。 |
| 3    | **public void sendError(int code, String message)** 该方法发送一个状态码（通常为 404），连同一个在 HTML 文档内部自动格式化并发送到客户端的短消息。 |

##### HTTP 状态码实例

下面的例子把 407 错误代码发送到客户端浏览器，浏览器会显示 "Need authentication!!!" 消息。

```
public class ShowError extends HttpServlet {
    // 处理 GET 方法请求的方法
    public void doGet(HttpServletRequest request,
                      HttpServletResponse response)
            throws ServletException, IOException
    {
        // 设置错误代码和原因
        response.sendError(407, "Need authentication!!!" );
    }
    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request,
                       HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```

web.xml添加：

```
 <servlet>
    <servlet-name>ShowError</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.ShowError</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>ShowError</servlet-name>
    <url-pattern>/ShowError</url-pattern>
  </servlet-mapping>
```

现在，调用上面的 Servlet 将显示407.

### Servlet 编写过滤器

Servlet 过滤器可以动态地拦截请求和响应，以变换或使用包含在请求或响应中的信息。

可以将一个或多个 Servlet 过滤器附加到一个 Servlet 或一组 Servlet。Servlet 过滤器也可以附加到  JavaServer Pages (JSP) 文件和 HTML 页面。调用 Servlet 前调用所有附加的 Servlet 过滤器。

Servlet 过滤器是可用于 Servlet 编程的 Java 类，可以实现以下目的：

- 在客户端的请求访问后端资源之前，拦截这些请求。
- 在服务器的响应发送回客户端之前，处理这些响应。

根据规范建议的各种类型的过滤器：

- 身份验证过滤器（Authentication Filters）。
- 数据压缩过滤器（Data compression Filters）。
- 加密过滤器（Encryption Filters）。
- 触发资源访问事件过滤器。
- 图像转换过滤器（Image Conversion Filters）。
- 日志记录和审核过滤器（Logging and Auditing Filters）。
- MIME-TYPE 链过滤器（MIME-TYPE Chain Filters）。
- 标记化过滤器（Tokenizing Filters）。
- XSL/T 过滤器（XSL/T Filters），转换 XML 内容。

过滤器通过 Web 部署描述符（web.xml）中的 XML 标签来声明，然后映射到您的应用程序的部署描述符中的 Servlet 名称或 URL 模式。

当 Web 容器启动 Web 应用程序时，它会为您在部署描述符中声明的每一个过滤器创建一个实例。

Filter的执行顺序与在web.xml配置文件中的配置顺序一致，一般把Filter配置在所有的Servlet之前。

#### Servlet 过滤器方法

过滤器是一个实现了 javax.servlet.Filter 接口的 Java 类。javax.servlet.Filter 接口定义了三个方法：

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **public void doFilter (ServletRequest, ServletResponse, FilterChain)** 该方法完成实际的过滤操作，当客户端请求方法与过滤器设置匹配的URL时，Servlet容器将先调用过滤器的doFilter方法。FilterChain用户访问后续过滤器。 |
| 2    | **public void init(FilterConfig filterConfig)** web  应用程序启动时，web 服务器将创建Filter  的实例对象，并调用其init方法，读取web.xml配置，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作（filter对象只会创建一次，init方法也只会执行一次）。开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。 |
| 3    | **public void destroy()** Servlet容器在销毁过滤器实例前调用该方法，在该方法中释放Servlet过滤器占用的资源。 |

#### FilterConfig 使用

Filter 的 init 方法中提供了一个 FilterConfig 对象。

如 web.xml 文件配置如下：

```
<filter>
    <filter-name>LogFilter</filter-name>
    <filter-class>com.runoob.test.LogFilter</filter-class>
    <init-param>
        <param-name>Site</param-name>
        <param-value>pingxin</param-value>
    </init-param>
    </filter>
```

在 init 方法使用 FilterConfig 对象获取参数：

```
public void  init(FilterConfig config) throws ServletException {
    // 获取初始化参数
    String site = config.getInitParameter("Site"); 
    // 输出初始化参数
    System.out.println("网站名称: " + site); 
}
```

**FilterConfig接口**

用户在配置filter时，可以使用为filter配置一些初始化参数，当web容器实例化Filter对象，调用其init方法时，会把封装了filter初始化参数的filterConfig对象传递进来。因此开发人员在编写filter时，通过filterConfig对象的方法，就可获得：

- 　　String getFilterName()：得到filter的名称。
- 　　String getInitParameter(String name)： 返回在部署描述中指定名称的初始化参数的值。如果不存在返回null.
- 　　Enumeration getInitParameterNames()：返回过滤器的所有初始化参数的名字的枚举集合。
- 　　public ServletContext getServletContext()：返回Servlet上下文对象的引用。
  范例：利用FilterConfig得到filter配置信息

#### Servlet 过滤器实例

以下是 Servlet 过滤器的实例，将输出网站名称和地址。本实例让您对 Servlet 过滤器有基本的了解，您可以使用相同的概念编写更复杂的过滤器应用程序：

```
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 获取初始化参数
        String site = filterConfig.getInitParameter("Site");

        // 输出初始化参数
        System.out.println("网站名称: " + site);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        // 输出站点名称
        System.out.println("站点网址：http://hanyunpeng0521.github.io");

        // 把请求传回过滤链
        chain.doFilter(request,response);
    }

    @Override
    public void destroy() {
        /* 在 Filter 实例被 Web 容器从服务移除之前调用 */
    }
}
```

这边使用前文提到的 DisplayHeader.java 为例子

##### Web.xml 中的 Servlet 过滤器映射（Servlet Filter Mapping）

定义过滤器，然后映射到一个 URL 或 Servlet，这与定义 Servlet，然后映射到一个 URL 模式方式大致相同。在部署描述符文件 **web.xml** 中为 filter 标签创建下面的条目：

```
<filter>
    <filter-name>LogFilter</filter-name>
    <filter-class>com.hyp.learn.javaweb.LogFilter</filter-class>
    <init-param>
      <param-name>Site</param-name>
      <param-value>pingxin</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>LogFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <servlet>
    <servlet-name>DisplayHeader</servlet-name>
    <servlet-class>com.hyp.learn.javaweb.DisplayHeader</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>DisplayHeader</servlet-name>
    <url-pattern>/DisplayHeader</url-pattern>
  </servlet-mapping>

```

上述过滤器适用于所有的 Servlet，因为我们在配置中指定 **/\*** 。如果您只想在少数的 Servlet 上应用过滤器，您可以指定一个特定的 Servlet 路径。

现在试着以常用的方式调用任何 Servlet，您将会看到在 Web 服务器中生成的日志。您也可以使用 Log4J 记录器来把上面的日志记录到一个单独的文件中。

接下来我们访问这个实例地址 **http://localhost:8080/javaweb/DisplayHeader**, 然后在控制台看下输出内容.

#### 使用多个过滤器

Web 应用程序可以根据特定的目的定义若干个不同的过滤器。假设您定义了两个过滤器 *AuthenFilter* 和 *LogFilter*。您需要创建一个如下所述的不同的映射，其余的处理与上述所讲解的大致相同：

```
<filter>
   <filter-name>LogFilter</filter-name>
   <filter-class>com.hyp.learn.javaweb.LogFilter</filter-class>
   <init-param>
      <param-name>test-param</param-name>
      <param-value>Initialization Paramter</param-value>
   </init-param>
</filter>

<filter>
   <filter-name>AuthenFilter</filter-name>
   <filter-class>com.hyp.learn.javaweb.AuthenFilter</filter-class>
   <init-param>
      <param-name>test-param</param-name>
      <param-value>Initialization Paramter</param-value>
   </init-param>
</filter>

<filter-mapping>
   <filter-name>LogFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>

<filter-mapping>
   <filter-name>AuthenFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### 过滤器的应用顺序

web.xml 中的 filter-mapping 元素的顺序决定了 Web 容器应用过滤器到 Servlet 的顺序。若要反转过滤器的顺序，您只需要在 web.xml 文件中反转 filter-mapping 元素即可。

例如，上面的实例将先应用 LogFilter，然后再应用 AuthenFilter，但是下面的实例将颠倒这个顺序：

```
<filter-mapping>
   <filter-name>AuthenFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>

<filter-mapping>
   <filter-name>LogFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

过滤器中我们可以根据 doFilte() 方法中的 request 对象获取表单参数信息，例如我们可以获取到请求的用户名和密码进行逻辑处理，也可以通过 response 对用户做出回应。比如如果验证用户名不正确，禁止用户访问 web 资源，并且向浏览器输出提示，告诉用户用户名或者密码不正确等等；

```
public void doFilter(ServletRequest req, ServletResponse resp,
FilterChain chain) throws IOException, ServletException {

    //获取请求信息(测试时可以通过get方式在URL中添加name)
    //http://localhost:8080/servlet_demo/helloword?name=123
    String name = req.getParameter("name");

    // 过滤器核心代码逻辑
    System.out.println("过滤器获取请求参数:"+name);
    System.out.println("第二个过滤器执行--网站名称：www.runoob.com");

    if("123".equals(name)){
        // 把请求传回过滤链
        chain.doFilter(req, resp);
    }else{
        //设置返回内容类型
        resp.setContentType("text/html;charset=GBK");

        //在页面输出响应信息
        PrintWriter out = resp.getWriter();
        out.print("<b>name不正确，请求被拦截，不能访问web资源</b>");
        System.out.println("name不正确，请求被拦截，不能访问web资源");
    }
}
```

#### web.xml配置各节点说明

- `<filter>`

  指定一个过滤器。  

  - `<filter-name>`用于为过滤器指定一个名字，该元素的内容不能为空。
  - `<filter-class>`元素用于指定过滤器的完整的限定类名。
  - `<init-param>`元素用于为过滤器指定初始化参数，它的子元素`<param-name>`指定参数的名字，`<param-value>`指定参数的值。
  - 在过滤器中，可以使用`FilterConfig`接口对象来访问初始化参数。

- `<filter-mapping>`

  元素用于设置一个 Filter 所负责拦截的资源。一个Filter拦截的资源可通过两种方式来指定：Servlet 名称和资源访问的请求路径  

  - `<filter-name>`子元素用于设置filter的注册名称。该值必须是在`<filter>`元素中声明过的过滤器的名字
  - `<url-pattern>`设置 filter 所拦截的请求路径(过滤器关联的URL样式)

- `<servlet-name>`指定过滤器所拦截的Servlet名称。

- `<dispatcher>`指定过滤器所拦截的资源被 Servlet 容器调用的方式，可以是`REQUEST`,`INCLUDE`,`FORWARD`和`ERROR`之一，默认`REQUEST`。用户可以设置多个`<dispatcher>`子元素用来指定 Filter 对资源的多种调用方式进行拦截。

- `<dispatcher>`

  子元素可以设置的值及其意义  

  - `REQUEST`：当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或forward()方法访问时，那么该过滤器就不会被调用。
  - `INCLUDE`：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用。
  - `FORWARD`：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用。
  - `ERROR`：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。

### 面试题

1. Http与Https的区别：

   - HTTP 的URL 以http:// 开头，而HTTPS 的URL 以https:// 开头

   - HTTP 是不安全的，而 HTTPS 是安全的

   - HTTP 标准端口是80 ，而 HTTPS 的标准端口是443

   - 在OSI 网络模型中，HTTP工作于应用层，而HTTPS 的安全传输机制工作在传输层

   - HTTP 无法加密，而HTTPS 对传输的数据进行加密

   - HTTP无需证书，而HTTPS 需要CA机构wosign的颁发的SSL证书

2. 什么是Http协议无状态协议?怎么解决Http协议无状态协议?

   无状态协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息

    也就是说，当客户端一次HTTP请求完成以后，客户端再发送一次HTTP请求，HTTP并不知道当前客户端是一个”老用户“。

   可以使用Cookie来解决无状态的问题，Cookie就相当于一个通行证，第一次访问的时候给客户端发送一个Cookie，当客户端再次来的时候，拿着Cookie(通行证)，那么服务器就知道这个是”老用户“。

3. URI和URL的区别

   **URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。**
   
   - Web上可用的每种资源如HTML文档、图像、视频片段、程序等都是一个来URI来定位的
   
   - URI一般由三部组成：
   
     - 访问资源的命名机制
   
     - 存放资源的主机名
   
     - 资源自身的名称，由路径表示，着重强调于资源。
   
   **URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。**
   
   - URL是Internet上用来描述信息资源的字符串，主要用在各种WWW客户程序和服务器程序上，特别是著名的Mosaic。
   
   - 采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等。URL一般由三部组成：
   
     - 协议(或称为服务方式)
   
     - 存有该资源的主机IP地址(有时也包括端口号)
   
     - 主机资源的具体地址。如目录和文件名等
   
   **URN，uniform resource name，统一资源命名，是通过名字来标识资源，比如mailto:java-net@java.sun.com。**
   
   - URI是以一种抽象的，高层次概念定义统一资源标识，而URL和URN则是具体的资源标识的方式。URL和URN都是一种URI。笼统地说，每个  URL 都是 URI，但不一定每个 URI 都是 URL。这是因为 URI 还包括一个子类，即统一资源名称  (URN)，它命名资源但不指定如何定位资源。上面的 mailto、news 和 isbn URI 都是 URN 的示例。
   
   在Java的URI中，**一个URI实例可以代表绝对的，也可以是相对的，只要它符合URI的语法规则。而URL类则不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的。**
   
   **在Java类库中，URI类不包含任何访问资源的方法，它唯一的作用就是解析。**
   
   **相反的是，URL类可以打开一个到达资源的流。**
   
4. 常用的HTTP方法有哪些？

   - GET： 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器
   - POST：用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。
   - PUT： 传输文件，报文主体中包含文件内容，保存到对应URI位置。
   - HEAD： 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
   - DELETE：删除文件，与PUT方法相反，删除对应URI位置的文件。
   - OPTIONS：查询相应URI支持的HTTP方法。

5. HTTPS工作原理

   - 一、首先HTTP请求服务端生成证书，客户端对证书的有效期、合法性、域名是否与请求的域名一致、证书的公钥（RSA加密）等进行校验；
   - 二、客户端如果校验通过后，就根据证书的公钥的有效， 生成随机数，随机数使用公钥进行加密（RSA加密）；
   - 三、消息体产生的后，对它的摘要进行MD5（或者SHA1）算法加密，此时就得到了RSA签名；
   - 四、发送给服务端，此时只有服务端（RSA私钥）能解密。
   - 五、解密得到的随机数，再用AES加密，作为密钥（此时的密钥只有客户端和服务端知道）。

   ![1.png](https://i.loli.net/2019/06/27/5d14bda97ed8137950.png)

   