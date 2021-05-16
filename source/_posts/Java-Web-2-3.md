---
title: Java servlet 源码分析(四)
date: 2019-05-30 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

Servlet是运行在Web服务器上的小程序，通过http协议和客户端进行交互。

 这里的客户端一般为浏览器，发送http请求（request）给服务器（如Tomcat）。服务器接收到请求后选择相应的Servlet进行处理，并给出响应（response）。

<!--more-->

![UTOOLS1576646693200.png](https://upload.cc/i1/2019/12/18/myVvU8.png)

从这里可以看出Servlet并不是独立运行的程序，而是以服务器为宿主，由服务器进行调度的。通常我们把能够运行Servlet的服务器称作Servlet容器，如Tomcat。

这里Tomcat为什么能够根据客户端的请求去选择相应的Servlet去执行的呢？答案是：Servlet规范。

因为Servlet和Servlet容器都是遵照Servlet规范去开发的。简单点说：我们要写一个Servlet，就需要直接或间接实现javax.servlet.Servlet。并且在web.xml中进行相应的配置。Tomcat在接收到客户端的请求时，会根据web.xml里面的配置去加载、初始化对应的Servlet实例。这个就是规范，就是双方约定好的。

#### 样例分析

在进一步解释Servlet原理、分析源码之前，我们先介绍下如何在JavaWeb中使用Servlet。方法很简单：

1. 编写自己的Servlet类，这里可以使用开发工具（STS、Myeclipse等）根据向导快速的生成一个Servlet类。
2. 在web.xml中配置servlet。

这里的知识很简单，所以不做过多赘述。直接上代码。（这里需要注意的是，servlet3.0之后提供了注解WebServlet的方式配置servlet，只是配置的形式不同而已，没有本质区别。所以下文还是为web.xml为例）

```java
//TestServlet.java
public class TestServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public TestServlet() {
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().append("Served at: ").append(request.getContextPath());
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}

```

web.xml

```xml
<servlet>
    <servlet-name>TestServlet</servlet-name>
    <servlet-class>com.nantang.servlet.TestServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>TestServlet</servlet-name>
    <url-pattern>/test</url-pattern>
</servlet-mapping>
```

启动Tomcat，浏览器访问/test。将会访问TestServlet。返回客户端请求的上下文路径。

![UTOOLS1576647570073.png](https://i.loli.net/2019/12/18/LbxoTGJBUM398zg.png)

这里需要扩展的有几点：

1. 如果一个servlet需要映射多个url-pattern，那么就在<servlet-mapping\></servlet-mapping\>标签下写多个<url-pattern\></url-pattern\>，如：

   ```xml
   <servlet-mapping>
       <servlet-name>TestServlet</servlet-name>
       <url-pattern>/test1</url-pattern>
       <url-pattern>/test2</url-pattern>
   </servlet-mapping>
   ```

2. 对于不同的servlet，不允许出现相同的url-pattern。

3. 如果不同的servlet，它们的url-patter存在包含关系，那么容器会调用更具象的servlet去处理客户端请求。比如有两个servlet，servlet1的url-pattern是"/\*"，servlet2的url-pattern是"/test"。那么这个时候如果客户端调用的url是http://localhost:8080/demo/test，容器会使用servlet2去处理客户端的请求。虽然说"/\*"和"/test"都匹配客户请求的url，但是容器会选择更贴切的。这里不会出现多个servlet处理同一个请求的现象。

#### 源码分析

上面说过，我们自己编写的Servlet类都必须直接或间接实现javax.servlet.Servlet。可是上面的例子TestServlet继承的是HttpServlet，那是因为HttpServlet间接的实现了javax.servlet.Servlet。下面是HttpServlet的继承层级（类图中的方法并没有一一列举，因为下面会逐一解释）：

![UTOOLS1576647687241.png](https://i.loli.net/2019/12/18/na8c5ADshMgoeZl.png)

下面我们由上往下层层分析：

##### ServletContext

一个web应用对应一个ServletContext实例，这个实例是应用部署启动后，servlet容器为应用创建的，ServletContext的作用范围是整个应用。ServletContext实例包含了所有servlet共享的资源信息。通过提供一组方法给servlet使用，用来和servlet容器通讯，比如获取文件的MIME类型、分发请求、记录日志等。

这里需要注意一点，如果你的应用是分布式部署的，那么每台服务器实例上部署的应用实例都各自拥有一个ServletContext实例。

![UTOOLS1576647800014.png](https://i.loli.net/2019/12/18/eUK32yZpRE4Hwj8.png)



下面我们先逐个分析ServletContext中servlet3.0之前的规范定义的方法（其中有三个方法是servlet3.0规范定义的，放在一起讲是出于方便的考虑，讲到的时候会特别说明）。

```java
public interface ServletContext {
    public String getContextPath();

    public ServletContext getContext(String uripath);

    public int getMajorVersion();
    public int getMinorVersion();
    public int getEffectiveMajorVersion();
    public int getEffectiveMinorVersion();
    
    public String getServerInfo();
    public String getServletContextName();

    public String getMimeType(String file);
    
    public void log(String msg);
    public void log(String message, Throwable throwable);

    public Set<String> getResourcePaths(String path);
 
    public URL getResource(String path) throws MalformedURLException;
    public InputStream getResourceAsStream(String path);
    
    public RequestDispatcher getRequestDispatcher(String path);
    public RequestDispatcher getNamedDispatcher(String name);

    public String getRealPath(String path);

    public String getInitParameter(String name);
    public Enumeration<String> getInitParameterNames();
    public boolean setInitParameter(String name, String value);

    public Object getAttribute(String name);
    public Enumeration<String> getAttributeNames();
    public void setAttribute(String name, Object object);
    public void removeAttribute(String name);

    // 省略servlet3.0及3.1规范定义的方法
}
```

1. getContextPath

   方法返回web应用的上下文路径。就是我们部署的应用的根目录名称。拿Tomcat举例，我们在webapps部署了应用demo。那么方法返 回"/demo"。如果是部署在ROOT下，那么方法返回空字符串""。这里的路径可以再server.xml里面修改，比如我们的demo应用路径修改 为"/test"：

   ```bash
   <Context docBase="demo" path="/test" reloadable="true" source="org.eclipse.jst.j2ee.server:demo"/>
   ```

   那么方法将会返回"/test"。

2. getContext

   方法入参为uriPath（String），是一个资源定位符的路径。返回一个ServletContext实例。我们说过一个web应用对应一个ServletContext实例，那么这个方法根据资源的路径返回其servlet上下文。

   比如说我们当前应用是demo，这个时候我们要访问servlet容器中的另外一个应用test中的资源index.jsp，假使资源的路径为/test/index.jsp。那么我们就可用通过调用getContext("/test/index.jsp")去获取test应用的上下文。如果在servlet容器中找不到该资源或者该资源限制了外部的访问，那么方法返回null。（这个方法一般配合RequestDispatcher使用，实现请求转发）。

3.  getMajorVersion、getMinorVersion、getEffectiveMajorVersion、getEffectiveMinorVersion

   getMajorVersion和getMinorVersion分别返回当前servlet容器支持的Servlet规范最高版本和最低版本。
    getEffectiveMajorVersion和getEffectiveMinorVersion分别返回当前应用基于的Servlet规范最高版本和最低版本，是servlet3.0规范增加的新特性。

   所以一般情况下：

   ```
   getMajorVersion>=getEffectiveMajorVersion>getEffectiveMinorVersion>=getMinorVersion
   ```

4. getServerInfo、getServletContextName

   getServerInfo返回servlet容器的名称和版本，格式为servername/versionnumber。比如我在Tomcat下测试，输出的信息是：Apache Tomcat/9.0.0.M10。当然容器也可以多返回些额外的信息，这个就看各个servlet容器的实现了。

   getServletContextName返回应用的名称，这里的名称是web.xml里面配置的display-name，如果没配置则返回null。

   ```xml
   <display-name>Archetype Created Web Application</display-name>
   ```

5. getMimeType

   方法返回文件的MIME类型，MIME类型是容器配置的。可用通过web.xml进行配置，比如：

   ```xml
   <mime-mapping>  
       <extension>doc</extension>  
       <mime-type>application/vnd.ms-word</mime-type>  
   </mime-mapping>
   ```

   那么我们用浏览器打开文件的时候发现如果是doc文件，则会调用相应的word程序去打开。

6. log

   两个重载的log方法都是记录日志到servlet日志文件，这个对于有编程经验的来说没什么好解释的。需要注意的是servlet日志文件的路径由具体的 servlet容器自己去决定。如果你是在MyEclipse、STS这类的IDE中跑应用的话，那么日志信息将在控制台（Console）输出。如果是 发布到Tomcat下的话，日志文件是Tomcat目录下的/logs/localhost.yyyy-MM-dd.log。

7. getResourcePaths

   根据传入的路径，列出该路径下的所有资源路径。返回的路径是相对于web应用的上下文根或者相对于/WEB-INF/lib目录下的各个JAR包里面的/META-INF/resources目录。
   比如我们的web应用下有这些资源：`/welcome.html，/catalog/index.html，/catalog/products.html， /catalog/offers/books.html，/catalog/offers/music.html，/customer /login.jsp，/WEB-INF/web.xml，/WEB-INF/classes /com.acme.OrderServlet.class，/WEB-INF/lib/catalog.jar!/META-INF/resources/catalog/moreOffers/books.html`。
   如果调用方法getResourcePaths("/")，那么返回的是`{"/welcome.html", "/catalog/", "/customer/", "/WEB-INF/"}`。
   如果调用方法getResourcePaths("/catalog/")，那么返回的是`{"/catalog/index.html", "/catalog/products.html", "/catalog/offers/"}`。

   这里需要注意的是：

   - 路径一定要以"/"开头，结尾的"/"可要可不要。
   - 路径要从应用根目录下开始，如果调用方法getResourcePaths("/offers/")，此时返回的是null。

8. getResource和getResourceAsStream

   getResource将指定路径的资源封装成URL实例并返回，getResourceAsStream获取指定路径资源的输入流InputStream并返回。关于URL和InputStream的解释和使用，不在本篇博文的关注点。

   这里和上面一样，资源的路径以"/"开头，这个路径是相对于应用上下文根目录或者相对于/WEB-INF/lib目录下的各个JAR包里面的/META-INF/resources目录。在搜索资源的时候，servlet容器先是从应用上下文根目录下搜索再从JAR文件搜索，对于JAR文件的搜索顺序（哪个JAR先，哪个JAR后），servlet规范没有规定。

9.  getRequestDispatcher和getNamedDispatcher

   将指定的资源包装成RequestDispatcher实例并返回。区别是前者根据资源路径（该路径相对于当前应用上下文根），后者根据资源的名称（通过服务器控制台或者web.xml里面配置的，比如web.xml里面配置servlet的名称）。

   RequestDispatcher这个接口，看名字就知道主要用来进行分发的。所有的资源都可以包装成RequestDispatcher实例（主要是用于包装servlet），然后调用它的方法进行转发和包含。这里比较简单，不过过多赘述，直接上源码。（这里只列关键的两个方法，简单介绍下forward。如有兴趣深入，可自行去研究RequestDispatcher相关知识）

   ```java
   public interface RequestDispatcher {
       public void forward(ServletRequest request, ServletResponse response) throws ServletException, IOException;     
       public void include(ServletRequest request, ServletResponse response) throws ServletException, IOException;
   }
   ```

   还拿我们上面的demo应用例子来说：客户端访问http://xxx.xxx.xxx.xxx:xxxx/demo/test，这个时候访问我们的 TestServlet。这个时候如果要把请求转发到另外一个servlet，假使这个servlet的资源路径是/demo/test2。那么我们可以 调用getRequestDispatcher("/demo/test2")把资源包装成RequestDispatcher实例，再调用forward的方法。就实现了请求的转发。

   请求的转发使用起来很简单，因为servlet容器提供了ServletContext实例，我们在应用中只需要调用它的API就行。这个时候容器为我们做了很多事情，容器会根据资源的路径去获取ServletContext实例，正如上面的getContext方法。这里不一定就是当前 ServletContext，可以使其他应用的。如果找到了ServletContext，容器再将资源包装成RequestDispatcher实例进行转发。

10. getRealPath

    根据资源虚拟路径，返回实际路径。

     比如说应用中有个JSP页面index.jsp，调用getRealPath("index.jsp")，则返回index.jsp文件在文件系统中的绝对路径。在windows下或许是这样：D:\xxx\xxx\index.jsp，在linux下或许是这样：/root/xxx/index.jsp。

     这里可能存在应用中有多个index.jsp，它们在不同的路径下。这时候servlet容器是先从应用根目录下向下查找，再从/WEB-INF/lib目录下的各个JAR包里面的/META-INF/resources目录查找（前提是servlet容器解压了这些JAR），把找到的第一个资源绝对路径返回。

11. getInitParameter、getInitParameterNames、setInitParameter

    getInitParameter和getInitParameterNames是用来获取应用的初始化参数相关数据的，参数的作用域是整个应用。这个参数是在web.xml里面配置的（如下所示）或者使用setInitParameter方法设置。getInitParameter是根据参数名获取参数值，getInitParameterNames获取参数名集合。对于setInitParameter需要注意的是，如果设置的参数名已经存在会设置失败，这是servlet3.0规范增加的新特性。

    这里需要注意的是在web.xml配置多个初始化参数时，应该写多个<context-param\></context-param\>对，而不是在一个<context-param\></context-param\>对里面写多个<param-name\></param-name\>和<param-value\></param-value\>对。

    ```xml
    <context-param>
        <param-name>param1</param-name>
        <param-value>1</param-value>
    </context-param>    
    <context-param>
        <param-name>param2</param-name>
        <param-value>2</param-value>
    </context-param>
    ```

12. getAttribute、getAttributeNames、setAttribute、removeAttribute

    应用的属性相关操作，建议属性名遵循java包名的风格。这里需要讲的是setAttribute，当设置的属性名已经存在，则会替换掉就的属性值。如果设置属性值为null，那么效果和removeAttribute是一样的。

    这里需要注意的是，这些属性都是应用级的，在一个地方设置的属性可以被应用中其他地方使用。

下面我们逐个分析ServletContext中servlet3.0及3.1的规范中定义的方法。

```java
public interface ServletContext {
    // 省略servlet3.0之前的规范定义的方法
  
    public static final String TEMPDIR = "javax.servlet.context.tempdir";
    public static final String ORDERED_LIBS = "javax.servlet.context.orderedLibs";

    public ServletRegistration.Dynamic addServlet(String servletName, String className);
    public ServletRegistration.Dynamic addServlet(String servletName, Servlet servlet);
    public ServletRegistration.Dynamic addServlet(String servletName, Class <? extends Servlet> servletClass);
    public <T extends Servlet> T createServlet(Class<T> clazz) throws ServletException;
    public ServletRegistration getServletRegistration(String servletName);
    public Map<String, ? extends ServletRegistration> getServletRegistrations();

    public FilterRegistration.Dynamic addFilter(String filterName, String className);
    public FilterRegistration.Dynamic addFilter(String filterName, Filter filter);
    public FilterRegistration.Dynamic addFilter(String filterName, Class <? extends Filter> filterClass);
    public <T extends Filter> T createFilter(Class<T> clazz) throws ServletException;
    public FilterRegistration getFilterRegistration(String filterName);
    public Map<String, ? extends FilterRegistration> getFilterRegistrations();
    
    public void addListener(String className);
    public <T extends EventListener> void addListener(T t);
    public void addListener(Class <? extends EventListener> listenerClass);
    public <T extends EventListener> T createListener(Class<T> clazz) throws ServletException; 

    public SessionCookieConfig getSessionCookieConfig();
    public void setSessionTrackingModes(Set<SessionTrackingMode> sessionTrackingModes);
    public Set<SessionTrackingMode> getDefaultSessionTrackingModes();
    public Set<SessionTrackingMode> getEffectiveSessionTrackingModes();

    public JspConfigDescriptor getJspConfigDescriptor();

    public ClassLoader getClassLoader();

    public void declareRoles(String... roleNames);

    public String getVirtualServerName();
}
```

1. TEMPDIR和ORDERED_LIBS

   这是两个ServletContext的属性名，Servlet规范建议属性名遵循java包名的风格。

   TEMPDIR 对应的属性值是个java.io.File对象，表示该ServletContext对应的临时目录。拿tomcat来说,比如有个应用叫demo，那么 demo对应的ServletContext临时目录就是`{Tomcat目录}\work\Catalina\localhost\demo`。

   ORDERED_LIBS 对应的属性值是个java.util.List<java.lang.String>对象，每个元素表示WEB-INF/lib目录下的 JAR包名称。这些名称的排序是根据它们的web-fragment名称排序的，如果<absolute-ordering\>里面没有配 置<others\>的话，可能会存在冲突。如果没有在web.xml里面配置<absolute-ordering\>或在web-fragment.xml里面配置<ordering\>，那么属性值为null。这里涉及到servlet3.0增加的新特性————web模块化，感兴趣的读者自行去了解，这里不做引申。

2. addServlet、createServlet、getServletRegistration、getServletRegistrations

   三个重载方法addServlet提供了编程式的向servlet容器中注入servlet的方式，其达到的效果和在web.xml中配置或者使用WebServlet注解配置servlet是一样的。只不过这种方式更加灵活、动态。addServlet方法返回的是ServletRegistration实例（ervletRegistration.Dynamic集成ServletRegistration），使用这个实例可以进一步配置servlet的注册信息，比如配置url-pattern。

   createServlet的所用是实例化一个servlet，得到一个servlet实例。这个传入的表示servlet类的Class实例必须要有个无参构造函数，因为这个方法的底层实现就是利用发射机制调用默认的无参构造函数进行实例化。这个方法返回的servlet实例没有多大的使用意义，可能还是需要调用相应的addServlet进行注册。

   getServletRegistration根据servlet名称查找其注册信息，即ServletRegistration实例。

   getServletRegistrations是查询当前servlet上下文中所有的servlet的注册信息。

3. addFilter、createFilter、getFilterRegistration、getFilterRegistrations

   这里的几个方法和上面的servlet的几个方法很类似，也是提供编程的方式实现filter。不作赘述。

4. addListener、createListener

   这里的几个方法和上面的servlet的几个方法很类似，也是提供编程的方式实现listener。不作赘述。注意观察可以发现，这里没有类似getXxxRegistration的方法，这是因为listener不需要像url-pattern的这类注册信息。

5. getSessionCookieConfig、setSessionTrackingModes、getDefaultSessionTrackingModes、getEffectiveSessionTrackingModes

   getSessionCookieConfig方法返回SessionCookieConfig实例，这个实例可以用于获取和设置会话跟踪的cookie的属性。多次调用getSessionCookieConfig方法返回的SessionCookieConfig实例是同一个，说明SessionCookieConfig是单例的。

   setSessionTrackingModes用于设置会话的跟踪模式，getDefaultSessionTrackingModes用于获取默认的会话跟踪模式，getEffectiveSessionTrackingModes用于获取有效的会话跟踪模式。默认情况下，getDefaultSessionTrackingModes返回的会话跟踪模式就是有效的。

6.  getJspConfigDescriptor

   获取web.xml和web-fragment.xml中配置的<jsp-config\>数据

7. getClassLoader

   获取当前servlet上文的类加载器

8. declareRoles

   该方法用于申明安全角色

9.  getVirtualServerName

   方法返回servlet上下文（即应用）部署的逻辑主机名，比如在本机跑Tomcat，那么这个方法返回"Catalina/localhost"。

**总结**

至此，ServletContext的相关属性和方法大致讲解完了。ServletContext是个很重要的东西，在每次的servlet规范更新中，这个接口都有较大的变化。因为ServletContext是容器和应用沟通的桥梁，从一定程度上讲ServletContext就是servlet规范的体现。

##### ServletConfig

ServletConfig实例是由servlet容器构造的，当需要初始化servlet的时候，容器根据web.xml中的配置以及运行时环境构造出ServletConfig实例，并通过回调servlet的init方法传递给servlet（这个方法后面会讲到）。所以一个servlet实例对应一个ServletConfig实例。

![UTOOLS1576648520976.png](https://i.loli.net/2019/12/18/QuUSf9R7vjKiBcl.png)

```java
public interface ServletConfig {
//getServletName方法返回servlet实例的名称，这个就是我们在web.xml中<servlet-name>标签中配置的名字，当然也可以在服务器控制台去配置。
//如果这两个地方都没有配置servlet名称，那么将会返回servlet的类名。

    public String getServletName();
//getServletContext方法返回ServletContext实例，也就是我们上面说的应用上下文。
    public ServletContext getServletContext();

//这两个方法是用来获取servlet的初始化参数的，这个参数是在web.xml里面配置的（如下所示）。getInitParameter是根据参数名获取参数值，getInitParameterNames获取参数名集合。

//这里需要注意的是当需要配置多个初始化参数时，应该写多个<init-param></init-param>对，而不是在一个<init-param></init-param>对里面写多个<param-name></param-name>和<param-value></param-value>对。
  
    public String getInitParameter(String name);

    public Enumeration getInitParameterNames();
}
```

#####  Servlet

最原始最简单的JaveWeb模型，就是一个servlet容器上运行着若干个servlet用来处理客户端的请求。所以说servlet是JavaWeb最核心的东西，我们的业务逻辑基本上都是通过servlet实现的（虽然现在有各种框架，不用去直接编写servlet，但本质上还是在使用servlet）。

```java
public interface Servlet {
    
    public void init(ServletConfig config) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```

所有的servlet都是javax.servlet.Servlet的子类，就像Java里面所有的类都是Object的子类一样。Servlet类规定了每个servlet应该实现的方法，这个是遵循Servlet规范的。但是自定义的servlet一般不用直接实现Servlet，而是继承javax.servlet.GenericServlet或者javax.servlet.http.HttpServlet就行了。

我们上面的TestServlet就是继承HttpServlet，这是因为HttpServlet间接实现了Servlet，提供了通用的功能。所以我们在自定义的TestServlet里面只需要专注实现业务逻辑就行了。

Servlet里面有三个比较重要的方法：init、service、destroy。它们被称作是servlet生命周期的方法，它们都是由servlet容器调用。另外两个方法用于获取servlet相关信息的，需要根据业务逻辑进行实现和调用。

![UTOOLS1576648670605.png](https://i.loli.net/2019/12/18/hAuTfPj6FJiSwMO.png)

1. init

   init方法是servlet的初始化方法，当客户端第一次请求servlet的时候，JVM对servlet类进行加载和实例化。（如果需要容器启动时就初始化servlet，可以在web.xml配置<load-on-startup\>1</load-on-startup\>）

   这里需要注意的是，servlet会先执行默认的构造函数，然后回调servlet实例的init方法，传入ServletConfig参数。这个参数上面说过，是servlet容器根据web.xml中的配置和运行时环境构造的实例。通过init方法注入到servlet。init方法在servlet的生命周期中只会被调用一次，在客户端的后续请求中将不会再调用。

2. service

   service方法是处理业务逻辑的核心方法。当servlet容器接收到客户端的请求后，会根据web.xml中配置的<url-pattern\>找到相应的servlet，回调service方法处理客户端的请求并给出响应。

3. destroy

   JDK文档解释这个方法说：这个方法会在所有的线程的service()方法执行完成或者超时后执行。这里只是说明了，当servlet容器要去调用destroy方式的时候，需要等待一会，等待所有线程都执行完或者达到超时的限制。

   这里并没有说清楚什么情况下servlet容器会触发这个动作。How Tomcat Works一书中对这个做了解释：当servlet容器关闭或需要更多内存的时候，会销毁servlet。这个方法就使得servlet容器拥有回收资源的能力。

   同样地，destroy方法在servlet的生命周期中只会被调用一次。

4. getServletConfig

   这个方法返回ServletConfig实例，这个对象即为servlet容器回调init方法的时候传入的实例。所以自定义的Servlet一般的实现方式为：在init方法里面把传入的ServletConfig存储到servlet的属性字段。在getServletConfig的实现里返回该实例。这个在后续解释javax.servlet.GenericServlet的源码时，能够看到。

5. getServletInfo

   返回关于servlet的信息，这个由自定义的servlet自行实现，不过一般建议返回servlet的作者、版本号、版权等信息

##### GenericServlet

GenericServlet从名字就能看的出来是servlet的一般实现，实现了servlet具有的通用功能，所以我们自定义的servlet一般不需要直接实现Servlet接口，只需要集成GenericServlet。GenericServlet实现了Servlet和ServletConfig接口。

**GenericServlet对Servlet接口的实现**

```java
private transient ServletConfig config;

public void init(ServletConfig config) throws ServletException {
    this.config = config;
    this.init();
}

public void init() throws ServletException {}

public abstract void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;

public void destroy() {}

public ServletConfig getServletConfig() {
    return config;
}

public String getServletInfo() {
    return "";
}
```

可以说，GenericServlet对Servlet方法的实现逻辑非常简单。就是把一些必要的逻辑写了下。

1. init方法就是把容器传入的ServletConfig实力存储在类的私有属性conifg里面，然后调用一个init无参的空方法。这么做的意义在于，我们如果想在自定义的servlet类里面在初始化的时候添加些业务逻辑，只需要重写无参的init方法就好了，我们不需要关注ServletConfig实例的存储细节了。

2. service和destroy方法并未实现具体逻辑。

3. getServletConfig就是返回init方法里面存储的config。getServletInfo就是返回空字符串，如果有业务需要，可以在子类里面重写。

**GenericServlet对于ServletConfig接口的实现**

```java
public String getServletName() {
    ServletConfig sc = getServletConfig();
    if (sc == null) {
        throw new IllegalStateException(
            lStrings.getString("err.servlet_config_not_initialized"));
    }
    return sc.getServletName();
}

public ServletContext getServletContext() {
    ServletConfig sc = getServletConfig();
    if (sc == null) {
        throw new IllegalStateException(
            lStrings.getString("err.servlet_config_not_initialized"));
    }
    return sc.getServletContext();
}

public String getInitParameter(String name) {
    ServletConfig sc = getServletConfig();
    if (sc == null) {
        throw new IllegalStateException(
            lStrings.getString("err.servlet_config_not_initialized"));
    }
    return sc.getInitParameter(name);
}

public Enumeration getInitParameterNames() {
    ServletConfig sc = getServletConfig();
    if (sc == null) {
        throw new IllegalStateException(
            lStrings.getString("err.servlet_config_not_initialized"));
    }
    return sc.getInitParameterNames();
}
```

这四个方法的实现就跟一个模子刻出来的一样，都是取得ServletConfig实例，然后调用相应的方法。其实GenericServlet完全没有必要实现ServletConfig，这么做仅仅是为了方便。当我们集成GenericServlet写自己的servlet的时候，如果需要获取servlet的配置信息如初始化参数，就不需要写形如：“`ServletConfig sc = getServletConfig();if (sc == null) ...;return sc.getInitParameterNames();`”这些冗余代码了。除此之外，没有别的意义。

##### HttpServlet

HttpServlet是一个针对HTTP协议的通用实现，它实现了HTTP协议中的基本方法get、post等，通过重写service方法实现方法的分派。

![UTOOLS1576649138089.png](https://i.loli.net/2019/12/18/h8fcX5FRK9sbDeI.png)



```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException{
    HttpServletRequest    request;
    HttpServletResponse    response;
    try {
        request = (HttpServletRequest) req;
        response = (HttpServletResponse) res;
    } catch (ClassCastException e) {
        throw new ServletException("non-HTTP request or response");
    }
    service(request, response);
}
```

重写的service方法将参数转换成HttpServletRequest和HttpServletResponse，并调用自己的另一个重载service方法。

```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
    String method = req.getMethod();
    if (method.equals(METHOD_GET)) {
        long lastModified = getLastModified(req);
        if (lastModified == -1) {
            doGet(req, resp);
        } else {
            long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
            if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                maybeSetLastModified(resp, lastModified);
                doGet(req, resp);
            } else {
                resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
            }
        }
    } else if (method.equals(METHOD_HEAD)) {
        long lastModified = getLastModified(req);
        maybeSetLastModified(resp, lastModified);
        doHead(req, resp);
    } else if (method.equals(METHOD_POST)) {
        doPost(req, resp);        
    } else if (method.equals(METHOD_PUT)) {
        doPut(req, resp);            
    } else if (method.equals(METHOD_DELETE)) {
        doDelete(req, resp);        
    } else if (method.equals(METHOD_OPTIONS)) {
        doOptions(req,resp);        
    } else if (method.equals(METHOD_TRACE)) {
        doTrace(req,resp);        
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[1];
        errArgs[0] = method;
        errMsg = MessageFormat.format(errMsg, errArgs);        
        resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
    }
}
```

这个方法的的逻辑也很简单，就是解析出客户端的request是哪种中方法，如果是get方法则调用doGet，如果是post则调用doPost等等。这样我们在继承HttpServlet的时候就无需重写service方法，我们可以根据自己的业务重写相应的方法。一般情况下我们的应用基本就是get和post调用。那么我们只需要重写doGet和doPost就行了。

这里需要注意的是3-15行代码，这里对资源（比如页面）的修改时间进行验证，判断客户端是否是第一次请求该资源，或者该资源是否被修改过。如果这两个条件有一个被满足那么就调用doGet方法。否则返回状态304（HttpServletResponse.SC_NOT_MODIFIED），这个状态就是告诉客户端（浏览器），可以只用自己上一次对该资源的缓存。

不过HttpServlet对于判断资源修改时间的逻辑非常简单粗暴：

```java
protected long getLastModified(HttpServletRequest req) {
    return -1;
}
```

方法始终返回-1，这样就会导致每次都会调用doGet方法从服务器取资源而不会使用浏览器的本地缓存。所以如果我们自己的servlet要使用浏览器的缓存，降低服务器的压力，就需要重写getLastModified方法。

最后我们来看一下HttpServlet对http一些方法的实现，在所有的方法中，HttpServlet已经对doOptions和doTrace方法实现了通用的逻辑，所以我们一般不用重写这两个方法，感兴趣的可以自己去看下源码。

这里我们列举下最常用的两个方法doGet和doPost：

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)throws ServletException, IOException{
    String protocol = req.getProtocol();
    String msg = lStrings.getString("http.method_get_not_supported");
    if (protocol.endsWith("1.1")) {
        resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
    } else {
        resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
    }
}
    
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
    String protocol = req.getProtocol();
    String msg = lStrings.getString("http.method_post_not_supported");
    if (protocol.endsWith("1.1")) {
        resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
    } else {
        resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
    }
}
```

其实这两个实现里面也没做什么有用的逻辑，所以一般情况下都要重写这两个方法，就像我们最初的TestServlet那样。doHead、doPut、doDelete也是这样的代码模板，所以如果有业务需要的话，我们都要重写对应的方法。

#### 总结

讲了这么多，可以这么说：Servlet是JavaWeb里面最核心的组件。只有对它完全融会贯通，才能去进一步去理解上层框架Struts、Spring等。

另外需要明确的是：一个Web应用对应一个ServletContext，一个Servlet对应一个ServletConfig。每个Servlet都是单例的，所以需要自己处理好并发的场景。



### 参考

1. [JavaWeb--ServletContext](https://www.jianshu.com/p/31d27181d542)
2. [JavaWeb——Servlet（全网最详细教程包括Servlet源码分析）](https://blog.csdn.net/qq_19782019/article/details/80292110)