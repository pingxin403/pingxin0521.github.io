---
title: Java servlet 3.0 (三)
date: 2019-05-18 08:19:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

在JavaWeb开发中， 每次编写一个Servlet都需要在**web.xml**文件中进行配置，如下所示：

```xml
<servlet>
    <servlet-name>ActionServlet</servlet-name>
    <servlet-class>com.web.controller.ActionServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>ActionServlet</servlet-name>
    <url-pattern>/servlet/ActionServlet</url-pattern>
</servlet-mapping>
```

<!--more-->

如果servlet过多，容易引起web.xml文件太复杂，servlet 3.0开始支持注解开发

pom.xml依赖，导入servlet包

```xml
<dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
</dependency>
```

在一个web应用中，如果使用注解的类位于WEB-INF/classes目录下，或者如果它们被打成jar包并被放到WEB-INF/lib目录中，使用注解的类将自己处理它们的注解。

web应用部署描述符在web-app元素上有一个新的属性metadata-complete。属性metadata-complete定义了web描述符是否完整，或者在部署的时候jar文件中类文件是否应该被扫描注解和web段。如果属性metadata-complete被设置为true，部署工具必须忽略出现在应用的类文件中和web分段中的任何servlet注解。如果属性metadata-complete没有明确设置或者被设置为false，部署工具必须检查应用中类文件的注解和扫描web片段。

下列是必须被兼容Servlet 3.0的容器支持的注解。

1. @WebServlet

   这个注解用来定义web应用中的一个Servlet注解。这个注解应用在类上，且包含声明的Servlet的元数据。注解上的属性urlPatterns和value必须配置。其它属性可以不用配置，直接使用默认值就好。推荐做法是，当注解上的属性仅仅是url pattern时使用value，当注解上还有其它属性使用时使用urlPattern。不能在一个注解上同时使用value和urlPatterns。如果没有指定Servlet的名字，默认的名字是全路径类名。被注解的servlet必须指明至少一个url Pattern。如果相同的servlet类在部署描述符中声明了两份，但是名字不一样，那么servlet的新实例必须被实例化。如果相同的servlet类通过编程API被添加到ServletContext，那么通过@WebServlet注解声明的值必须被忽略，并且要为有具体名字的servlet必须要创建一个新实例。

   用*@WebServlet*注解的类必须继承*javax.servlet.http.HttpServlet*。

   注解主要参数

   | **属性名**     | **类型**       | **描述**                                                     |
   | -------------- | -------------- | ------------------------------------------------------------ |
   | name           | String         | 指定 Servlet 的 name 属性，等价于 <servlet-name\>。如果没有显式指定，则该 Servlet 的取值即为类的全限定名。 |
   | value          | String[]       | 该属性等价于 urlPatterns 属性。两个属性不能同时使用。        |
   | urlPatterns    | String[]       | 指定一组 Servlet 的 URL 匹配模式。等价于 <url-pattern\> 标签。 |
   | loadOnStartup  | int            | 指定 Servlet 的加载顺序，等价于 <load-on-startup\> 标签。    |
   | initParams     | WebInitParam[] | 指定一组 Servlet 初始化参数，等价于 <init-param\> 标签。     |
   | asyncSupported | boolean        | 声明 Servlet 是否支持异步操作模式，等价于 <async-supported\> 标签。 |
   | description    | String         | 该 Servlet 的描述信息，等价于 <description\> 标签。          |
   | displayName    | String         | 该 Servlet 的显示名，通常配合工具使用，等价于 <display-name\> 标签。 |

   下面是一个简单的示例：

   ```java
   @WebServlet(urlPatterns = {"/simple"}, asyncSupported = true,
   loadOnStartup = -1, name = "SimpleServlet", displayName = "ss",
   initParams = {@WebInitParam(name = "username", value = "tom")}
   )
   public class SimpleServlet extends HttpServlet{ … }
   ```

   如此配置之后，就可以不必在 web.xml 中配置相应的 <servlet\> 和 <servlet-mapping\> 元素了，容器会在部署时根据指定的属性将该类发布为 Servlet。它的等价的 web.xml 配置形式如下：

   ```xml
   
   <servlet>
       <display-name>ss</display-name>
       <servlet-name>SimpleServlet</servlet-name>
       <servlet-class>footmark.servlet.SimpleServlet</servlet-class>
       <load-on-startup>-1</load-on-startup>
       <async-supported>true</async-supported>
       <init-param>
           <param-name>username</param-name>
           <param-value>tom</param-value>
       </init-param>
   </servlet>
   <servlet-mapping>
       <servlet-name>SimpleServlet</servlet-name>
       <url-pattern>/simple</url-pattern>
   </servlet-mapping>
   ```

2. @WebFilter

   这个注解用来定义web应用中的Filter。这个注解应用在类上，并且包含有关filter的元数据。如果没有指明Filter的名字，那么默认的名字将是全路径类名。注解的属性URLPattern，servltName，或者value必须被配置。所有其它属性可选，并且有默认值可用。推荐的做法是，当注解上仅有url pattern属性的时候使用value，当还有其它属性被使用时使用urlPatterns。不能再同一个注解上同时使用value和urlPatterns。

   用@WebFilter注解的类必须继承javax.servlet.Filter。

   ```java
   import javax.servlet.*;
   import javax.servlet.annotation.WebFilter;
   import javax.servlet.annotation.WebInitParam;
   import javax.servlet.http.HttpServletRequest;
   import java.io.IOException;
   
   @WebFilter(filterName = "filter1", urlPatterns="/*",
           dispatcherTypes = {DispatcherType.REQUEST, DispatcherType.FORWARD},
           initParams={@WebInitParam(name="account",value="1234"),@WebInitParam(name="hotusm",value="1234")}
           )
   
   public class MyFilter implements Filter {
       @Override
       public void init(final FilterConfig filterConfig) throws ServletException {
           
           String account = filterConfig.getInitParameter("account");
           String hotusm = filterConfig.getInitParameter("hotusm");
           
           System.out.println("account:"+account+" hotusm:"+hotusm);
       }
   
       @Override
       public void doFilter(final ServletRequest request, final ServletResponse response, final FilterChain chain) throws IOException, ServletException {     
           chain.doFilter(request, response);
       }
   
       @Override
       public void destroy() {
           
       }
   }
   ```

   注解的主要参数及其含义

   | **属性名**      | **类型**       | **描述**                                                     |
   | --------------- | -------------- | ------------------------------------------------------------ |
   | filterName      | String         | 指定过滤器的 name 属性，等价于 <filter-name\>                |
   | value           | String[]       | 该属性等价于 urlPatterns 属性。但是两者不应该同时使用。      |
   | urlPatterns     | String[]       | 指定一组过滤器的 URL 匹配模式。等价于 <url-pattern\> 标签。  |
   | servletNames    | String[]       | 指定过滤器将应用于哪些 Servlet。取值是 @WebServlet 中的 name 属性的取值，或者是 web.xml 中 <servlet-name\> 的取值。 |
   | dispatcherTypes | DispatcherType | 指定过滤器的转发模式。具体取值包括： ASYNC、ERROR、FORWARD、INCLUDE、REQUEST。 |
   | initParams      | WebInitParam[] | 指定一组过滤器初始化参数，等价于 <init-param\> 标签。        |
   | asyncSupported  | boolean        | 声明过滤器是否支持异步操作模式，等价于 <async-supported\> 标签。 |
   | description     | String         | 该过滤器的描述信息，等价于 <description\> 标签。             |
   | displayName     | String         | 该过滤器的显示名，通常配合工具使用，等价于 <display-name\> 标签。 |

   下面是一个简单的示例：

   ```
   
   @WebFilter(servletNames = {"SimpleServlet"},filterName="SimpleFilter")
   public class LessThanSixFilter implements Filter{...}
   ```

   如此配置之后，就可以不必在 web.xml 中配置相应的 <filter\> 和 <filter-mapping\> 元素了，容器会在部署时根据指定的属性将该类发布为过滤器。它等价的 web.xml 中的配置形式为：

   ```xml
   <filter>
       <filter-name>SimpleFilter</filter-name>
       <filter-class>xxx</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>SimpleFilter</filter-name>
       <servlet-name>SimpleServlet</servlet-name>
   </filter-mapping>
   ```

3. @WebInitParam

   这个注解用来指明任何必须传递给Servlet或者Filter的参数。它是WebServlet和WebFilter注解的一个属性。

   相当于<init-param\>参数以及含义

   | **属性名**  | **类型** | **是否可选** | **描述**                                |
   | ----------- | -------- | ------------ | --------------------------------------- |
   | name        | String   | 否           | 指定参数的名字，等价于 <param-name\>。  |
   | value       | String   | 否           | 指定参数的值，等价于 <param-value\>。   |
   | description | String   | 是           | 关于参数的描述，等价于 <description\>。 |

4. @WebListener

   替代

   ```
   <listener>
   <listener-class>ListenerClass</listener-class>
   </listener>
   ```

   注解WebListener用来注解一个监听者，以便获取web应用上下文中各种操作的事件。用@WebListener注解的类必须实现下列接口的一个：

   - javax.servlet.ServletContextListener

   - javax.servlet.ServletContextAttributeListener

   - javax.servlet.ServletRequestListener

   - javax.servlet.ServletRequestAttributeListener

   - javax.servlet.http.HttpSessionListener

   - javax.servlet.http.HttpSessionAttributeListener

   ```java
   import javax.servlet.annotation.WebListener;
   import javax.servlet.http.HttpSessionEvent;
   import javax.servlet.http.HttpSessionListener;
   
   @WebListener("This is only a demo listener")
   public class MyHttpSessionListener implements HttpSessionListener{
   
       @Override
       public void sessionCreated(HttpSessionEvent se) {
           System.out.println("创建session ");
       }
   
       @Override
       public void sessionDestroyed(HttpSessionEvent se) {
           System.out.println("销毁session ");
       }
   
   }
   ```

5. @MultipartConfig

   当应用在一个Servlet上，这个注解表明它期望的请求是mime/multipart类型。与servlet对应的HttpServletRequest对象必须要能够通过方法getParts和getPart来遍历不同的mime附件。javax.serlvet.annotation.MultipartConfig的location属性和<multipart-config\>的<location\>元素会被解析为绝对路径，并且默认值是javax.servlet.context.tempdir。如果指定了一个相对地址，它会相对于tempdir这个位置。绝对路径和相对路径的测试必须通过java.io.File.isAbsolute 来完成

   | **属性名**        | **类型** | **是否可选** | **描述**                                                     |
   | ----------------- | -------- | ------------ | ------------------------------------------------------------ |
   | fileSizeThreshold | int      | 是           | 当数据量大于该值时，内容将被写入文件。                       |
   | location          | String   | 是           | 存放生成的文件地址。                                         |
   | maxFileSize       | long     | 是           | 允许上传的文件最大值。默认值为 -1，表示没有限制。            |
   | maxRequestSize    | long     | 是           | 针对该 multipart/form-data 请求的最大数量，默认值为 -1，表示没有限制。 |

   例子:

   ```java
   <form action="upload" enctype="multipart/form-data" method="POST">
       <input name="file" type="file" />
       <input type="submit" value="Upload" />
   </form>
   
   import java.io.File;
   import java.io.IOException;
   import java.io.PrintWriter;
   
   import javax.servlet.ServletException;
   import javax.servlet.annotation.MultipartConfig;
   import javax.servlet.annotation.WebServlet;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import javax.servlet.http.Part;
   
   @MultipartConfig(location="/data")
   @WebServlet(name="upload",urlPatterns={"/upload"})
   public class UploadServlet extends HttpServlet{
       
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           req.setCharacterEncoding("utf-8");  
           resp.setContentType("text/html; charset=UTF-8");  
           PrintWriter out = resp.getWriter();  
           Part part = req.getPart("file");  
           String fileName = part.getHeader("content-disposition");
           System.out.println(fileName);
           System.out.println(fileName.substring(fileName.lastIndexOf(File.separator)+1, fileName.length()-1));
           part.write( fileName.substring(fileName.lastIndexOf(File.separator)+1, fileName.length()-1));
           out.write("{'type':'1','msg':'success'}");
       }
   }
   ```

**Shared libraries（共享库） / runtimes pluggability（运行时插件能力）**

1. Servlet容器启动会扫描，当前应用里面每一个jar包的ServletContainerInitializer的实现
2. 提供ServletContainerInitializer的实现类，并且必须绑定在`/src/META-INF/services/javax.servlet.ServletContainerInitializer`或者`/src/webapp/META-INF/javax.servlet.ServletContainerInitializer`文件的内容就是ServletContainerInitializer实现类的全类名；

```java
//HelloService及其子类
public interface HelloService {
}

public abstract class AbstractHelloService implements HelloService {
}

public interface HelloServiceExt extends HelloService {
}

public class HelloServiceImpl implements HelloService {
}

//UserServlet
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class UserServlet extends HttpServlet {
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		// TODO Auto-generated method stub
		resp.getWriter().write("tomcat...");
	}

}

//UserFilter
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public class UserFilter implements Filter {

	@Override
	public void destroy() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {
		// 过滤请求
		System.out.println("UserFilter...doFilter...");
		//放行
		arg2.doFilter(arg0, arg1);
		
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		// TODO Auto-generated method stub
		
	}

}

//UserListener
import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

/**
 * 监听项目的启动和停止
 *
 */
public class UserListener implements ServletContextListener {

	
	//监听ServletContext销毁
	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		// TODO Auto-generated method stub
		System.out.println("UserListener...contextDestroyed...");
	}

	//监听ServletContext启动初始化
	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		// TODO Auto-generated method stub
		ServletContext servletContext = arg0.getServletContext();
		System.out.println("UserListener...contextInitialized...");
	}

}

import java.util.EnumSet;
import java.util.Set;

import javax.servlet.DispatcherType;
import javax.servlet.FilterRegistration;
import javax.servlet.ServletContainerInitializer;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration;
import javax.servlet.annotation.HandlesTypes;

import com.hyp.learn.service.HelloService;

//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类，子接口等）传递过来；
//传入感兴趣的类型；
@HandlesTypes(value={HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {

	/**
	 * 应用启动的时候，会运行onStartup方法；
	 * 
	 * Set<Class<?>> arg0：感兴趣的类型的所有子类型；
	 * ServletContext arg1:代表当前Web应用的ServletContext；一个Web应用一个ServletContext；
	 * 
	 * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
	 * 2）、使用编码的方式，在项目启动的时候给ServletContext里面添加组件；
	 * 		必须在项目启动的时候来添加；
	 * 		1）、ServletContainerInitializer得到的ServletContext；
	 * 		2）、ServletContextListener得到的ServletContext；
	 */
	@Override
	public void onStartup(Set<Class<?>> arg0, ServletContext sc) throws ServletException {
		// TODO Auto-generated method stub
		System.out.println("感兴趣的类型：");
		for (Class<?> claz : arg0) {
			System.out.println(claz);
		}
		
       //第三方库的注册
        
		//注册组件  ServletRegistration  
		ServletRegistration.Dynamic servlet = sc.addServlet("userServlet", new UserServlet());
		//配置servlet的映射信息
		servlet.addMapping("/user");
		
		
		//注册Listener
		sc.addListener(UserListener.class);
		
		//注册Filter  FilterRegistration
		FilterRegistration.Dynamic filter = sc.addFilter("userFilter", UserFilter.class);
		//配置Filter的映射信息
		filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
		
	}

}
```

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面`META-INF/services/javax.servlet.ServletContainerInitializer`指定的实现类，启动并运行这个实现类的方法；传入感兴趣的类型；

#### 异步处理

servlet线程不再需要一直阻塞，直到业务处理完毕才再输出响应，最后才结束该servlet线程，在接收到请求后，servlet可以将耗时的操作委派给另一个线程去完成，自己在不生成响应的情况下返回至容器。

servlet3之前，一个普通servlet的主要工作流程大致如下：首先，servlet接受到请求以后，可能需要对请求携带的数据进行一些预处理，接着，调用业务接口的某些方法，以完成业务处理。最后，根据处理的结果提交响应，servlet线程结束。其中业务处理通常是非常耗时的，主要体现在数据库操作以及跨网络调用等，在此过程中servlet线程一直处于**阻塞状态**，直到业务方法执行完毕。对于并发较大的应用，这有可能造成性能瓶颈

servlet3针对这个问题提供了异步处理支持，Servlet请求将请求转交给另一个线程去执行业务逻辑，线程本身返回至容器，此时servlet还没有生成响应数据。异步线程处理完业务以后，可以直接生成响应结果或者转交给其它servlet处理。如此一来，servlet不再一直处于阻塞状态以等待业务逻辑的处理，而是启动异步线程后可以立即返回。

接收到request请求之后，由tomcat工作线程从HttpServletRequest中获得一个异步上下文AsyncContext对象，然后由tomcat工作线程把AsyncContext对象传递给业务处理线程，同时tomcat工作线程归还到工作线程池，这一步就是异步开始。在业务处理线程中完成业务逻辑的处理，生成response返回给客户端。在Servlet3.0中虽然处理请求可以实现异步，但是InputStream和OutputStream的IO操作还是阻塞的，当数据量大的request body 或者 response body的时候，就会导致不必要的等待。从**Servlet3.1以后增加了非阻塞IO，需要tomcat8.x支持**。

**使用步骤**

我们使用的大致步骤如下：

1. 声明Servlet，增加asyncSupported属性，开启异步支持。@WebServlet(urlPatterns = "/AsyncLongRunningServlet", asyncSupported = true)
2. 通过request获取异步上下文AsyncContext。AsyncContext asyncCtx = request.startAsync();
3. 开启业务逻辑处理线程，并将AsyncContext 传递给业务线程。executor.execute(new AsyncRequestProcessor(asyncCtx, secs));
4. 在异步业务逻辑处理线程中，通过asyncContext获取request和response，处理对应的业务。
5. 业务逻辑处理线程处理完成逻辑之后，调用AsyncContext 的complete方法。asyncContext.complete();从而结束该次异步线程处理。

**示例**

异步处理特性可以应用于 Servlet 和过滤器两种组件，由于异步处理的工作模式和普通工作模式在实现上有着本质的区别，因此默认情况下，Servlet 和过滤器并没有开启异步处理特性，如果希望使用该特性，则必须按照如下的方式启用

```java
对于使用传统的部署描述文件 (web.xml) 配置 Servlet 和过滤器的情况，
Servlet 3.0 为 <servlet> 和 <filter> 标签增加了 <async-supported> 子标签，
该标签的默认取值为 false，要启用异步处理支持，则将其设为 true 即可
<servlet>
    <servlet-name>DemoServlet</servlet-name>
    <servlet-class>footmark.servlet.DemoServlet</servlet-class>
    <async-supported>true</async-supported>
</servlet>   

对于使用 Servlet 3.0 提供的 @WebServlet 和 @WebFilter 进行 Servlet 或过滤器配置的情况，
这两个注解都提供了 asyncSupported 属性，默认该属性的取值为 false，
要启用异步处理支持，只需将该属性设置为 true 即可。以 @WebFilter 为例，
其配置方式如下所示：

@WebFilter(urlPatterns = "/demo",asyncSupported = true)
public class DemoFilter implements Filter{...}

```

一个简单的模拟异步处理的servlet:

```java
import javax.servlet.AsyncContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.concurrent.ThreadPoolExecutor;

@WebServlet(urlPatterns = "/AsyncLongRunningServlet", asyncSupported = true)
public class AsyncLongRunningServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response) throws ServletException, IOException {
        long startTime = System.currentTimeMillis();
        System.out.println("AsyncLongRunningServlet Start::Name="
                + Thread.currentThread().getName() + "::ID="
                + Thread.currentThread().getId());

        request.setAttribute("org.apache.catalina.ASYNC_SUPPORTED", true);

        String time = request.getParameter("time");
        int secs = Integer.valueOf(time);
        // max 10 seconds
        if (secs > 10000)
            secs = 10000;

        AsyncContext asyncCtx = request.startAsync();
        asyncCtx.addListener(new AppAsyncListener());
        asyncCtx.setTimeout(9000);//异步servlet的超时时间,异步Servlet有对应的超时时间，如果在指定的时间内没有执行完操作，response依然会走原来Servlet的结束逻辑，后续的异步操作执行完再写回的时候，可能会遇到异常。

        ThreadPoolExecutor executor = (ThreadPoolExecutor) request
                .getServletContext().getAttribute("executor");

        executor.execute(new AsyncRequestProcessor(asyncCtx, secs));
        long endTime = System.currentTimeMillis();
        System.out.println("AsyncLongRunningServlet End::Name="
                + Thread.currentThread().getName() + "::ID="
                + Thread.currentThread().getId() + "::Time Taken="
                + (endTime - startTime) + " ms.");
    }
}
```

除此之外，Servlet 3.0 还为异步处理提供了一个监听器，使用 AsyncListener 接口表示。它可以监控如下四种事件：

1. 异步线程开始时，调用 AsyncListener 的 onStartAsync(AsyncEvent event) 方法；
2. 异步线程出错时，调用 AsyncListener 的 onError(AsyncEvent event) 方法；
3. 异步线程执行超时，则调用 AsyncListener 的 onTimeout(AsyncEvent event) 方法；
4. 异步执行完毕时，调用 AsyncListener 的 onComplete(AsyncEvent event) 方法；

```java
import javax.servlet.AsyncEvent;
import javax.servlet.AsyncListener;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebListener;
import java.io.IOException;
import java.io.PrintWriter;
@WebListener
public class AppAsyncListener implements AsyncListener {
    @Override
    public void onComplete(AsyncEvent asyncEvent) throws IOException {
        System.out.println("AppAsyncListener onComplete");
        // we can do resource cleanup activity here
    }

    @Override
    public void onError(AsyncEvent asyncEvent) throws IOException {
        System.out.println("AppAsyncListener onError");
        //we can return error response to client
    }

    @Override
    public void onStartAsync(AsyncEvent asyncEvent) throws IOException {
        System.out.println("AppAsyncListener onStartAsync");
        //we can log the event here
    }

    @Override
    public void onTimeout(AsyncEvent asyncEvent) throws IOException {
        System.out.println("AppAsyncListener onTimeout");
        //we can send appropriate response to client
        ServletResponse response = asyncEvent.getAsyncContext().getResponse();
        PrintWriter out = response.getWriter();
        out.write("TimeOut Error in Processing");
    }
}

// Servlet上下文监听器，可以在里面初始化业务线程池
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

@WebListener
public class AppContextListener implements ServletContextListener {
    public void contextInitialized(ServletContextEvent servletContextEvent) {

        // create the thread pool
        ThreadPoolExecutor executor = new ThreadPoolExecutor(100, 200, 50000L,
                TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(100));
        servletContextEvent.getServletContext().setAttribute("executor",
                executor);

    }

    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        ThreadPoolExecutor executor = (ThreadPoolExecutor) servletContextEvent
                .getServletContext().getAttribute("executor");
        executor.shutdown();
    }
}

//业务工作线程
import javax.servlet.AsyncContext;
import java.io.IOException;
import java.io.PrintWriter;

public class AsyncRequestProcessor implements Runnable {
    private AsyncContext asyncContext;
    private int secs;

    public AsyncRequestProcessor() {
    }

    public AsyncRequestProcessor(AsyncContext asyncCtx, int secs) {
        this.asyncContext = asyncCtx;
        this.secs = secs;
    }

    @Override
    public void run() {
        System.out.println("Async Supported? "
                + asyncContext.getRequest().isAsyncSupported());
        longProcessing(secs);
        try {
            PrintWriter out = asyncContext.getResponse().getWriter();
            out.write("Processing done for " + secs + " milliseconds!!");
        } catch (IOException e) {
            e.printStackTrace();
        }
        //complete the processing
        asyncContext.complete();
    }

    private void longProcessing(int secs) {
        // wait for given time before finishing
        try {
            Thread.sleep(secs);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**Servlet3非阻塞IO**

Servlet3.1以后增加了非阻塞IO实现，需要Tomcat8.x以上支持。根据Servlet3.1规范中的描述”非阻塞 IO 仅对在 Servlet 中的异步处理请求有效，否则，当调用 ServletInputStream.setReadListener 或ServletOutputStream.setWriteListener 方法时将抛出IllegalStateException“。可以说Servlet3的非阻塞IO是对Servlet3异步的增强。Servlet3的非阻塞是利用java.util.EventListener的事件驱动机制来实现的。

```java
//AsyncLongRunningServlet.java 接收请求，获取读取请求监听器ReadListener

import javax.servlet.AsyncContext;
import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(urlPatterns = "/AsyncLongRunningServlet2", asyncSupported = true)
public class AsyncLongRunningServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        AsyncContext actx = request.startAsync();//通过request获得AsyncContent对象

        actx.setTimeout(30*3000);//设置异步调用超时时长

        ServletInputStream in = request.getInputStream();
        //异步读取（实现了非阻塞式读取）
        in.setReadListener(new MyReadListener(in,actx));
        //直接输出到页面的内容(不等异步完成就直接给页面)
        PrintWriter out = response.getWriter();
        out.println("<h1>直接返回页面，不等异步处理结果了</h1>");
        out.flush();
    }

}

//MyReadListener.java 异步处理

import javax.servlet.AsyncContext;
import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import java.io.IOException;
import java.io.PrintWriter;

public class MyReadListener implements ReadListener {
    private ServletInputStream inputStream;
    private AsyncContext asyncContext;
    public MyReadListener(ServletInputStream input,AsyncContext context){
        this.inputStream = input;
        this.asyncContext = context;
    }
    //数据可用时触发执行
    @Override
    public void onDataAvailable() throws IOException {
        System.out.println("数据可用时触发执行");
    }

    //数据读完时触发调用
    @Override
    public void onAllDataRead() throws IOException {
        try {
            Thread.sleep(3000);//暂停5秒，模拟耗时处理数据
            PrintWriter out = asyncContext.getResponse().getWriter();
            out.write("数据读完了");
            out.flush();
            System.out.println("数据读完了");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    //数据出错触发调用
    @Override
    public void onError(Throwable t){
        System.out.println("数据 出错");
        t.printStackTrace();
    }
}
```

Servlet3.1的非阻塞IO从下面图中可以看出是面对InputStream 和 OutPutStream流的，这里的非阻塞IO跟我们常说的JDK NIO不是一个概念，Servlet3.1的非阻塞是同jdk的事件驱动机制来实现。

```java
public interface ReadListener extends java.util.EventListener
```

**总结**

通讯模型中的NIO可以利用很少的线程处理大量的连接，提高了机器的吞吐量。Servlet的异步处理机制使得我们可以将请求异步到独立的业务线程去执行，使得我们能够将请求线程和业务线程分离。通讯模型的NIO跟Servlet3的异步没有直接关系。但是我们将两种技术同时使用就更增加了以tomcat为容器的系统的处理能力。自从Servlet3.1以后增加了非阻塞的IO，这里的非阻塞IO是面向inputstream和outputstream流，通过jdk的事件驱动模型来实现，更一步增强了Servlet异步的高性能，可以认为是一种增强版的异步机制。







### 参考

1. [Servlet3使用总结](https://blog.csdn.net/Nerver_77/article/details/80368087)
2. [servlet3异步原理与实践](https://www.jianshu.com/p/c23ca9d26f64)