---
title: Java Web相关问题
date: 2019-05-17 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

思考： 目标资源是给谁使用的。
给服务器使用的： / 表示在当前web应用的根目录（webRoot下）
给浏览器使用的： / 表示在webapps的根目录下

<!--more-->

```html
1.转发
    request.getRequestDispatcher("/target.html").forward(request, response);
    
2.请求重定向
    response.sendRedirect("/day11/target.html");
    
3.html页面的超连接href
    response.getWriter().write("<html><body><a href='/day11/target.html'>超链接</a></body></html>");
    
4.html页面中的form提交地址
    response.getWriter().write("<html><body><form action='/day11/target.html'><input type='submit'/></form></body></html>");
```

`.`代表java命令运行目录

```
 在web项目中， . 代表在tomcat/bin目录下开始，所以不能使用这种相对路径。
 使用web应用下加载资源文件的方法
```

ServletContext.getRealPath("路径")读取,返回资源文件的绝对路径

```dart
String path = this.getServletContext().getRealPath("/WEB-INF/classes/db.properties")
```

ServletContext.getResourceAsStream("路径")

```kotlin
InputStream in = this.getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties
```

#### web项目目录分析，文件存放位置

```
1）如果JSP，JS文件放在WEB-INF目录下**【服务器外部】**是根本无法访问的。

2）放在WEB-INF目录下的JSP文件，可以通过服务器内部转向进行访问（主要是为了页面的安全）

3）因为JS是通过客户端向服务器请求的（即需要能够从服务器外部访问），所以图片以及一些JS,CSS只能放在WEB-INF外面 

```

web应用的目录结构：

![lQsXHP.png](https://s2.ax1x.com/2019/12/30/lQsXHP.png)

等效于下图

![lQsxN8.png](https://s2.ax1x.com/2019/12/30/lQsxN8.png)

src： 存放java源码，编译后的文件都会放在WEB-INF下的classes文件夹下
WebContent：项目创建完后，只有这个目录有用，因为web项目需要的所有文件都在这里。

web.xml:

```xml
<servlet-mapping> 
<servlet-name>handleservlet</servlet-name> 
<url-pattern>/handleservlet</url-pattern>此映射是相对于当前web应用的 
</servlet-mapping> 

```

所有相对路径都是由”/”开头的

比如：/image/a.gif，/user/main.jsp，

**html相对路径**

有个html文件：a.html，其中有

```
<link href="one.css" rel="stylesheet" type="text/css">，
```

其中href属性表示引用的css文件的路径。

`one.css`：表示one.css和a.hmtl处于同一个目录
`user/one.css`：表示one.css处于a.html所在目录的子目录user中，即user是a.html在同一个目录 。
`../one.css`：表示one.css位于a.hmtl上一级目录下，
`../../one.css`：表示one.css位于a.hmtl上一级目录的上一级目录下，
`./`：表示和a.hmtl同一目录
我们称上述相对路径为

#### 服务器端的地址

服务器端的相对地址指的是相对于你的web应用的地址，这个地址是在服务器端解析的 （不同于html和JavaScript中的相对地址，他们是 由客户端浏览器解析的 ）。

也就是说这时候在jsp和servlet中的相对地址应该是相对于你的web应用，即相对于<http://192.168.0.1/test/>的。

其用到的地方有：

1. forward：servlet中的request.getRequestDispatcher(address);这个address是在服务器端解析的。
   所以你要forward到user/a.jsp应该这么写：

   ```
   request.getRequestDispatcher("/user/a.jsp ")
   ```

   这个/相对于当前的web应用test，其绝对地址就是：<http://192.168.0.1/test/user/a.jsp>。

2. redirect：在jsp中

   ```
   <%response.sendRedirect("/rtccp/user/a.jsp");%> 
   ```

#### 客户端的地址

所有的html中的相对地址都是相对于<http://192.168.0.1/>的，而不是<http://192.168.0.1/test/>的 。

html中的form表单的action属性的地址应该是相对于<http://192.168.0.1/>的。所以，如果提交到user/a.jsp为：action＝"/test/user/a.jsp" ；提交到servlet为action＝"/test/handleservlet"

Javascript也是在客户端解析的，所以其相对路径和form表单一样。

#### 站点根目录和css路径问题 (jsp是服务器端程序，地址是变化的，引用时一般用站点根目录的相对路径)

我们称类似这样的相对路径`/test/`. 为相对于站点根目录 的相对路径 。

当在jsp中引入css时，如果其相对路径相对于当前jsp文件的，而在一个和这个jsp的路径不一样的servlet中forward这个jsp时，就会发现这个css样式根本没有起作用。这是因为在servlet中转发时css的路径就是相对于这个servlet的相对路径而非jsp的路径了。所以这时候不能在jsp中用这样的路径：

```
<link href="one.css" rel="stylesheet" type="text/css">
或者
<link href="../../one.css" rel="stylesheet" type="text/css">
```

类似href=”one.css”和../../one.css的html相对路径是相对于引用这个css的文件(a.jsp)的相对路径 。而在servlet中转发时就是相对于这个servlet的相对路径了，因为jsp路径和servlet路径是不一样的 ，所以这样的引用肯定是出错的。

所以这个时候，要用站点根目录，就是相对于<http://192.168.0.1/>的目录，以“/”开头。

因此上述错误应更正为href=”/test/one.css” 类似的站点根目录的相对目录。这样在servlet转发后和jsp中都是相对于站点根目录的相对路径 ，就能正确使用所定义的css样式了。

#### 使用说明

1. sendRedirect

   ```
   servlet和jsp里面一样
   
   response.sendRedirect(); 
   ```

2. include

   这种也是上面提到的forward形式，request的值会保存

   ```
   1)  servlet里面
   
       request.getRequestDispatcher("jsp2.jsp" ).include(request, response);  
   
   2)  jsp里面
   
       <jsp:include page= "include.jsp" />
   ```

   **说明**
   页面会同时包含页面1和页面2的内容，地址栏不变。
   使用request.setAttribute的内容，可以正常使用

3. forword

   ```
   1)  servlet里面
   
       request.getRequestDispatcher("jsp2.jsp" ).forward(request,response);  
   
   2)  jsp里面
   
   <jsp:forward page= "include.jsp" />
   
   ```

   说明
   页面会是页面2的内容，地址栏不变
   使用request.setAttribute的内容，可以正常使用

#### JSP中用相对路径引用JS,CSS文件的三种情况

1. 一个tomcat上都跑多个工程, 用工程名来区分

   因为我的的URL是 :<http://localhost/工程名/home/index.jsp>,多了一个工程名,所以要加 <%=request.getContextPath() %>,如:

   ```
   <script src="<%=request.getContextPath() %> /home/test.js"></script>
   ```

   写<%=request.getContextPath() %>太麻烦，可以在每一个jsp文件顶部加入以下内容后，

   ```jsp
   <%   
   String path = request.getContextPath();   
   String basePath = request.getScheme()+"://" +request.getServerName()+":" +request.getServerPort()+path+"/" ;   
   %>   
   <base href="<%=basePath%>" > 
   ```

   就可直接使用

   ```
   <script src="/home/test.js"></script>
   ```

   

### 参考

1. [HTML、JSP、Servlet中的相对路径和绝对路径 页面跳转问题](https://blog.csdn.net/wym19830218/article/details/5503533/)

   