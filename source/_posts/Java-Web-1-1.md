---
title: Java Web入门
date: 2019-05-13 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

#### 什么是Web?

Web在计算机网页开发设计中就是网页的意思。网页是网站中的一个页面，我们平常浏览网站时，看到的都是一个一个的页面，通常它们都是**HTML**格式的。网页可以展示文字、图片、媒体等内容，而这些都是需要通过浏览器来阅读。

<!--more-->

#### Web应用程序的工作原理？

Web应用程序大体上可以分为两种，**静态网站**和**动态网站**。

早期的Web应用主要是静态页面的浏览，即静态网站。这些网站使用**HTML**描写，通常来说随着html代码的生成，页面的内容和显示效果就基本上不会发生变化了——除非你修改页面代码。这些代码放在Web服务器上，用户使用浏览器通过**HTTP协议**请求服务器上的Web页面，服务器上的Web服务器接受到用户的请求处理后，再发送给客户端浏览器，显示给用户。整个过程就像下图：

![1.png](https://i.loli.net/2019/05/14/5cda48b41b1bb27213.png)



而动态网页则不然，页面代码虽然没有变，但是显示的内容却是可以随着时间、环境或者数据库操作的结果而发生改变的。这些网站通常使用**HTML**和**动态脚本语言（入JSP、ASP或者是PHP等）**编写，并将编写后的程序部署到Web服务器上，由Web服务器堆动态脚本代码进行处理，并转化成浏览器可以解析的**HTML**代码，返回给客户端浏览器，显示给用户。

**值得一提的是：**动态网页并非是那些带有动画效果的网页，而是指具有交互性、内容可以自动更新，并且内容会根据访问的时间和访问者而改变的网页。这里所说的交互性是指网页可以根据用户的要求动态改变或响应。
 由此可见，静态网页就像是老式的手机，只能使用系统自带的铃声和功能，而动态网页就像是现代的手机，可以自行添加/删除或者说更改铃声和其他一些设置。

### Web的发展历程

自从1989年由 **Tim Berners-Lee(蒂姆·伯纳斯·李)** 发明了 **World Wide Web** 以来，Web 主要精力了3个阶段，分别是静态文档阶段（指代 Web 1.0）、动态网页阶段（指代 Web 1.5）和 Web 2.0 阶段。

#### 静态文档阶段

处理静态文档阶段的 Web ，主要是用于静态 Web 页面的浏览。用户通过客户端的 Web 浏览器可以访问 Internet 上各个 Web 站点。在每个 Web 站点上，保存着提前编写好的 HTML 格式的 Web 页，以及各 Web 页之间可以实现跳转的超文本链接。通常情况下，这些 Web 页都是通过 HTML 语言编写的。由于受低版本 HTML 语言和旧式浏览器的制约，Web 页面只能包括单纯的文本内容，浏览器页只能显示呆板的文字信息，不过这已经基本满足了建立 Web 站点的初衷，实现了信息资源共享。

随着互联网技术的不断发展以及网上信息呈几何倍数的增长，人们逐渐发现手工编写包含所有信息和内容的页面，对人力和物理都是一种极大的浪费，而且几乎变得难以实现。另外，这样的页面也无法实现各种动态的交互功能。这就促使了 Web 技术进入了发展的第二阶段——动态网页阶段。

#### 动态网页阶段

为了克服静态页面的不足，人们将传统单机环境下的编程技术与 Web 技术相结合，从而形成新的网络编程技术。网络编程技术通过在传统的静态网页中加入各种程序和逻辑控制，从而实现动态和个性化的交流与互动。我们将这种使用网络编程技术创建的页面称为动态页面。动态页面的后缀通常是**.jsp、.php、和.asp**等，而静态页面的后缀通常是**.htm、.html和.shtml**等。

#### Web 2.0 阶段

随着互联网技术的不断发展，又提出了一种新的互联网模式——Web 2.0。这种模式更加以用户为中心，通过网络应用（ Web Applications ）促进网络上人与人间的信息交换和协同合作。

Web 2.0 技术主要包括：博客（ BLOG ）、微博（ Twitter ）、维基百科全书（ Wiki ）、即时信息（ IM ）等。

![2.png](https://i.loli.net/2019/05/14/5cda49007a56b61485.png)

### 网络程序开发的体系结构

随着 Web 2.0 时代的到来，互联网的网络架构已经从传统的 C/S 架构转变为更加方便、快捷的 B/S 架构，B/S 架构大大简化了用户使用网络应用的难度，这种人人都能上网、人人都能使用网络上提供的服务的方法也进一步推动了互联网的繁荣。

理解 C/S 和 B/S 可以通过一些实际的例子。C/S 就像是桌面 QQ 等一些**运行在桌面的程序，**，在**服务端主要就是一个数据库，把所有业务逻辑以及界面的渲染操作交给客户端**去完成。而 B/S 就是我们的浏览器，**把业务逻辑交给服务端完成，客户端仅仅只做界面渲染和数据交换。**

B/S 架构带来了以下两个方面的好处：

- **客户端使用同一的浏览器（ Browser ）。**由于浏览器具有统一性，它不需要特殊的配置和网络连接，有效的屏蔽了不同服务提供商提供给用户使用服务的差异性。另外，最重要的一点，浏览器的交互特性使得用户使用它非常简便，而且用户行为的可继承性非常强，也就是用户只要学会了上网，不管使用的是哪一个应用，一旦学会了，在使用其他互联网服务时同样具有了使用经验，因为它们都是基于同样的浏览器操作界面。
- **服务端（ Server ）基于统一的 HTTP 。**和传统的 C/S 架构使用自定义的应用层协议不同，B/S 价格使用的都是统一的 HTTP。使用同一的 HTTP 也为服务提供商简化了开发模式，使得服务器开发者可以采用相对规范的开发模式，这样可以大大节省开发成本。由于使用统一的 HTTP，所以基于 HTTP 的服务器就有很多，如 **IIS、Tomcat** 等，这些服务器可以直接拿来使用，不需要服务开发者单独来开发。不仅如此，连开发服务的通用框架都不需要单独开发，服务开发者只需要关注提供服务的应用逻辑，其他一切平台和框架都可以直接拿来使用，所以 B/S 架构同样简化了服务器提供者的开发，从而出现了越来越多的互联网服务。

![3.png](https://i.loli.net/2019/05/14/5cda4940be2a883957.png)

#### B/S 网络架构概述

B/S 网络架构从前端到后端都得到了简化，基于统一的应用层协议 HTTP 来交互数据，与大多数传统 C/S 互联网应用程序采用的长连接的交互模式不同，**HTTP 采用无状态的短连接的通信方式，**通常情况下，一次请求就完成了一次数据交互，通常也对应一个业务逻辑，然后这次通信连接就断开了。采用这种方式是为了能够同时服务更多的用户，因为当前互联网应用每天都会处理上亿的用户请求，不可能每个用户访问一次后就一直保持这个连接。

基于 HTTP 本身的特点，目前的 B/S 网络架构大多采用 **CDN** 的架构设计（如上图），既要满足海量用户的访问请求，又要保持用户请求的快速响应，所以现在的网络架构也越来越复杂。

当一个用户在浏览器里输入 *www.taobao.com* 这个 URL 时，将会发生很多操作。首先它会请求 DNS 吧这个域名解析成对应的 IP 地址，然后根据这个 IP 地址在互联网上找到相对应的服务器，向这个服务器发起一个 get 请求，由这个服务器决定返回默认的数据资源给访问的用户。在服务器端实际上还有很复杂的业务逻辑：服务器可能有很多台，到底指定哪一台服务器来处理请求，这需要一个负载均衡设备来平均分配所有用户的请求；还有请求的数据是存储在分布式缓存里还是一个静态文件中，或是在数据库里；当数据返回浏览器时，浏览器解析数据发现还有一些静态资源（ 如 CSS 、JS 或者图片 ）时又会发起另外的 HTTP 请求，而这些请求很可能会在 CDN 上，那么 CDN 服务器又会处理这个用户的请求，大体上一个用户请求会设计这么多的操作。每一个细节都会影响这个请求最终是否会成功。

不管网络架构如何变化，时钟有一些固定不变的原则需要遵守。

-  **互联网上所有资源都要用一个 URL 来表示。**URL 就是同意资源定位符，如果你要发布一个服务或者一个资源到互联网上，让别人能够访问到，那么你首先必须要有一个在世界上独一无二的 URL 。**不要小看这个 URL ，它几乎包含了整个互联网的架构精髓。** 
-  **必须基于 HTTP 与服务端交互。**不管你要访问的事国内的还是国外的数据，是文本数据还是流媒体，都必须按照套路出牌，也就是都得采用统一打招呼的方式，这样人家才会明白你要的是什么。
-  **数据展示必须在浏览器中进行。**当你获取到数据资源后，必须在浏览器上才能恢复它的容貌。

只要满足上面的几点，一个互联网应用基本上就能正确地运行起来了，当然这里面还有很多细节。

### 相关技术

![4.png](https://i.loli.net/2019/05/14/5cda4bc639e7945784.png)

#### 项目结构

maven推荐包结构：

```

├── pom.xml
└── src
    ├── main
    │   ├── java //后端逻辑代码
    │   │   └── mygroup
    │   │       ├── controller
    │   │       │   ├── HomeController.java
    │   │       │   └── PersonController.java
    │   │       ├── dao
    │   │       │   └── PersonDao.java
    │   │       └── model
    │   │           └── Person.java
    │   ├── resources //后端使用到的资源文件
    │   │   ├── db.properties
    │   │   ├── log4j.xml
    │   │   └── META-INF
    │   │       └── persistence.xml
    │   └── webapp //网站前端,所有Web资源（包括HTML，JSP和图形文件等等）
    │       ├── index.html
    │       ├── META-INF 
    │       │   ├── context.xml
    │       │   └── MANIFEST.MF
    │       ├── resources //前端需要使用的资源:图片、css、js
    │       │   └── css
    │       │       └── screen.css
    │       └── WEB-INF//对于Web应用程序，此目录包含支持的Web资源，
    						//包含web.xml文件以classes和lib目录
    │           ├── spring
    │           │   ├── app
    │           │   │   ├── controllers.xml
    │           │   │   └── servlet-context.xml
    │           │   ├── db.xml
    │           │   └── root-context.xml
    │           ├── views
    │           │   ├── edit.jsp
    │           │   ├── home.jsp
    │           │   └── list.jsp
    │           └── web.xml
    └── test //测试
        ├── java
        │   └── mygroup
        │       ├── controller
        │       │   ├── DataInitializer.java
        │       │   ├── HomeControllerTest.java
        │       │   └── PersonControllerTest.java
        │       └── dao
        │           └── PersonDaoTest.java
        └── resources
            ├── db.properties
            ├── log4j.xml
            ├── test-context.xml
            └── test-db.xml
```

### web.xml文件

#### 加载过程

简单说一下，web.xml的加载过程。当我们启动一个WEB项目容器时，容器包括(JBoss,Tomcat等)。首先会去读取web.xml配置文件里的配置，当这一步骤没有出错并且完成之后，项目才能正常的被启动起来。

启动WEB项目的时候，容器首先会去读取web.xml配置文件中的两个节点：`<listener> </listener>`和`<context-param> </context-param>`

紧接着，容器创建一个ServletContext(application),这个web项目的所有部分都将共享这个上下文。容器以`<context-param></context-param>`的name作为键，value作为值，将其转化为键值对，存入ServletContext。　　

容器创建`<listener></listener>`中的类实例，根据配置的class类路径`<listener-class>`来创建监听，在监听中会有初始化方法，启动Web应用时，系统调用Listener的该方法 contextInitialized(ServletContextEvent args)，在这个方法中获得：

```
ServletContext application =ServletContextEvent.getServletContext();
context-param的值= application.getInitParameter("context-param的键");
```

得到这个context-param的值之后，你就可以做一些操作了。

举例：你可能想在项目启动之前就打开数据库，那么这里就可以在<context-param\>中设置数据库的连接方式（驱动、url、user、password），在监听类中初始化数据库的连接。这个监听是自己写的一个类，除了初始化方法，它还有销毁方法，用于关闭应用前释放资源。比如:说数据库连接的关闭，此时，调用contextDestroyed(ServletContextEvent args)，关闭Web应用时，系统调用Listener的该方法。

接着，容器会读取<filter\></filter\>，根据指定的类路径来实例化过滤器。

以上都是在WEB项目还没有完全启动起来的时候就已经完成了的工作。如果系统中有Servlet，则Servlet是在第一次发起请求的时候被实例化的，而且一般不会被容器销毁，它可以服务于多个用户的请求。所以，Servlet的初始化都要比上面提到的那几个要迟。总的来说，web.xml的加载顺序是: <context-param\>-> <listener\> -> <filter\> -> <servlet\>。其中，如果web.xml中出现了相同的元素，则按照在配置文件中出现的先后顺序来加载。

#### XML文档有效性检查

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
```

这段代码指定文件类型定义(DTD)，可以通过它检查XML文档的有效性。下面显示的<!DOCTYPE>元素有几个特性，这些特性告诉我们关于DTD的信息： 

-  web-app定义该文档(部署描述符，不是DTD文件)的根元素 

- PUBLIC意味着DTD文件可以被公开使用 
- " -//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"意味着DTD由Sun Microsystems, Inc.维护。该信息也表示它描述的文档类型是DTD Web Application 2.3，而且DTD是用英文书写的。 
-  URL"http://java.sun.com/dtd/web-app_2_3.dtd"表示D文件的位置。

#### 1.web应用名称设置

```
<display-name>应用名称</display-name>
```

#### 2.web应用描述

```
<description>应用描述</description>
```

#### 3.配置监听器

<listener\>为web应用程序定义监听器，监听器用来监听各种事件，比如：application和session事件，所有的监听器按照相同的方式定义，功能取决去它们各自实现的接口，常用的Web事件接口有如下几个：

1. ServletContextListener：用于监听Web应用的启动和关闭；
2. ServletContextAttributeListener：用于监听ServletContext范围（application）内属性的改变；
3. ServletRequestListener：用于监听用户的请求；
4.  ServletRequestAttributeListener：用于监听ServletRequest范围（request）内属性的改变；
5. HttpSessionListener：用于监听用户session的开始和结束；
6.  HttpSessionAttributeListener：用于监听HttpSession范围（session）内属性的改变。

<listener\>主要用于监听Web应用事件，其中有两个比较重要的WEB应用事件：**应用的启动和停止（**starting
up or shutting down**）**和**Session**的创建和失效（**created or destroyed**）。

应用启动事件发生在应用第一次被Servlet容器装载和启动的时候；停止事件发生在Web应用停止的时候。Session创建事件发生在每次一个新的session创建的时候，类似地Session失效事件发生在每次一个Session失效的时候。为了使用这些Web应用事件做些有用的事情，我们必须创建和使用一些特殊的“监听类”。

它们是实现了以下两个接口中任何一个接口的简单java类：javax.servlet.ServletContextListener或javax.servlet.http.HttpSessionListener，如果想让你的类监听应用的启动和停止事件，你就得实现ServletContextListener接口；想让你的类去监听Session的创建和失效事件，那你就得实现HttpSessionListener接口。

**Listener配置**

配置Listener只要向Web应用注册Listener实现类即可，无序配置参数之类的东西，因为Listener获取的是Web应用ServletContext（application）的配置参数。为Web应用配置Listener的两种方式：

1. 使用@WebListener修饰Listener实现类即可。

2. 在web.xml文档中使用<listener\>进行配置。

```
<!-- 日志监听 -->

<listener>
    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
  </listener>

<!--  配置日志文件地址-->

<context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>/WEB-INF/classes/META-INF/log4j.properties</param-value>
  </context-param>

<context-param>
    <param-name>log4jRefreshInterval</param-name>
    <param-value>120000</param-value>
  </context-param>
```


log4jRefreshInterval 该设置作用为容器会每120秒自动扫描log4j的配置文件，改变日志级别时不需要重启服务器。

```
<!-- 自动装载ApplicationContext的配置信息 -->

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

spring 提供了ServletContentListener的一个实现类ContextLoaderListener监听器，在启动tomcat容器时，该类的作用是自动装载ApplicationContext的配置信息，如果没有设置ContextLoaderListener的初始参数会使用默认参数web-inf路径下的application,xml文件，如果需要读取多个配置文件或修改默认路径，可以利用context-param设置：

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext*.xml</param-value>
</context-param>

<!-- 解决可能出现的内存泄漏问题-->
  <listener>
    <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
  </listener>
```

#### 4.设置应用的环境参数

该参数可以在jsp中通过方法获取，也可以在java代码中获取，放在监听器后面

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext*.xml</param-value>
</context-param>
```

在jsp中获取可以用{initparam.contextConfigLocation}获取对应的value值

在代码中可以通过string param=getservletContent().getParamter('contextConfigLocation');获取

#### 5.设置web应用的过滤器

Filter可认为是Servle的一种“加强版”，主要用于对用户请求request进行预处理，也可以对Response进行后处理，是个典型的处理链。使用Filter的完整流程是：Filter对用户请求进行预处理，接着将请求HttpServletRequest交给Servlet进行处理并生成响应，最后Filter再对服务器响应HttpServletResponse进行后处理。Filter与Servlet具有完全相同的生命周期，且Filter也可以通过<init-param\>来配置初始化参数，获取Filter的初始化参数则使用FilterConfig的getInitParameter()。

换种说法，Servlet里有request和response两个对象，Filter能够在一个request到达Servlet之前预处理request，也可以在离开Servlet时处理response，Filter其实是一个Servlet链。以下是Filter的一些常见应用场合，

1. 认证Filter
2. 日志和审核Filter
3. 图片转换Filter
4. 数据压缩Filter
5. 密码Filter
6. 令牌Filter
7. 触发资源访问事件的Filter
8. XSLT Filter
9. 媒体类型链Filter

Filter可负责拦截多个请求或响应；一个请求或响应也可被多个Filter拦截。创建一个Filter只需两步:

1. 创建Filter处理类
2.  Web.xml文件中配置Filter

Filter必须实现javax.servlet.Filter接口，在该接口中定义了三个方法：

1. void init(FilterConfig config)：用于完成Filter的初始化。FilteConfig用于访问Filter的配置信息。
2. void destroy()：用于Filter销毁前，完成某些资源的回收。
3. void doFilter(ServletRequest request,ServletResponse response,FilterChain chain)：实现过滤功能的核心方法，该方法就是对每个请求及响应增加额外的处理。该方法实现对用户请求request进行预处理，也可以实现对服务器响应response进行后处理---它们的分界线为是否调用了chain.doFilter(request，response)，执行该方法之前，即对用户请求request进行预处理，执行该方法之后，即对服务器响应response进行后处理。

##### Filter配置：

Filter可认为是Servlet的“增强版”，因此Filter配置与Servlet的配置非常相似，需要配置两部分：配置Filter名称和Filter拦截器URL模式。区别在于Servlet通常只配置一个URL，而Filter可以同时配置多个请求的URL。配置Filter有两种方式：

1. 在Filter类中通过Annotation进行配置。

2. 在web.xml文件中通过配置文件进行配置。

 我们使用的是web.xml这种配置方式，下面重点介绍<filter\>内包含的一些元素。
 <filter\>用于指定Web容器中的过滤器，可包含<filter-name\>、<filter-class\>、<init-param\>、<icon\>、<display-name\>、<description\>。

1. <filter-name\>用来定义过滤器的名称，该名称在整个程序中都必须唯一。

2. <filter-class\>元素指定过滤器类的完全限定的名称，即Filter的实现类。

3. <init-param\>为Filter配置参数，与<context-param\>具有相同的元素描述符<param-name\>和<param-value\>。

4. <filter-mapping\>元素用来声明Web应用中的过滤器映射，过滤器被映射到一个servlet或一个URL  模式。这个过滤器的<filter\>和<filter-mapping\>必须具有相同的<filter-name\>，指定该Filter所拦截的URL。过滤是按照部署描述符的<filter-mapping\>出现的顺序执行的。

字符集过滤器，用于处理项目中的乱码问题

```
 <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

encoding 作用为设置解析请求的编码方式为utf-8

forceEncoding=true 的意思是无论客户端请求是否包含编码必须使用过滤器里的编码方式解析请求

filter-mapping 设置的是该过滤器过滤的url范围，即当符合情况的url需要经过该过滤器处理

**缓存控制**

```
<filter>  
    <filter-name>NoCache Filter</filter-name>  
    <filter-class>com.yonyou.mcloud.cas.client.authentication.NoCacheFilter</filter-class>  
</filter>  
  <filter-mapping>  
<filter-name>NoCache Filter</filter-name>  
<!—表示对URL全部过滤-->  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

**登录认证**

```
<!-- 认证过滤器 -->  
  <filter>  
    <filter-name>CAS Authentication Filter</filter-name>  
<filter-class>com.yonyou.mcloud.cas.client.authentication.ExpandAuthenticationFilter</filter-class>  
<init-param>  
      <param-name>casServerLoginUrl</param-name>  
      <param-value>https://dev.yonyou.com:443/sso-server/login</param-value>  
    </init-param>  
    <init-param>  
      <!--这里的server是服务端的IP -->  
      <param-name>serverName</param-name>  
      <param-value>http://10.1.215.40:80</param-value>  
    </init-param>  
  </filter>  
  <filter-mapping>  
     <filter-name>CAS Authentication Filter</filter-name>  
     <url-pattern>/*</url-pattern>  
  </filter-mapping>
```

 登录认证，未登录用户导向CAS Server进行认证。

**单点登出**

```
<filter>  
      <filter-name>CAS Single Sign Out Filter</filter-name>  
      <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>  
</filter>  
<filter-mapping>  
      <filter-name>CAS Single Sign Out Filter</filter-name>  
       <url-pattern>/*</url-pattern>  
</filter-mapping>  
<listener>  
      <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>  
</listener>
```

CAS Server通知CAS Client，删除session，注销登录信息。

**封装request**

```
<filter>  
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
    <filter-class>org.jasig.cas.client.util.HttpServletRequestWrapperFilter</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

 封装request, 支持getUserPrincipal等方法。

 **存放Assertion到ThreadLocal中**

```
<filter>  
    <filter-name>CAS Assertion Thread Local Filter</filter-name>  
    <filter-class>org.jasig.cas.client.util.AssertionThreadLocalFilter</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>CAS Assertion Thread Local Filter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

**禁用浏览器缓存**

```
<filter>  
    <filter-name>NoCache Filter</filter-name>  
    <filter-class>com.yonyou.mcloud.cas.client.authentication.NoCacheFilter</filter-class>  
 </filter>  
 <filter-mapping>  
    <filter-name>NoCache Filter</filter-name>  
    <url-pattern>/*</url-pattern>  
 </filter-mapping>
```

**CAS Client向CAS Server进行ticket验证**

```
<!-- 验证ST/PT过滤器 -->  
<filter>  
   <filter-name>CAS Validation Filter</filter-name>  
    <filter-class>org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>  
   <init-param>  
      <param-name>casServerUrlPrefix</param-name>  
      <param-value>https://dev.yonyou.com:443/sso-server</param-value>  
   </init-param>  
   <init-param>  
      <param-name>serverName</param-name>  
      <param-value>http://10.1.215.40:80</param-value>  
   </init-param>  
   <init-param>  
      <param-name>proxyCallbackUrl</param-name>  
      <param-value>https://dev.yonyou.com:443/business/proxyCallback</param-value>  
   </init-param>  
   <init-param>  
      <param-name>proxyReceptorUrl</param-name>  
      <param-value>/proxyCallback</param-value>  
   </init-param>  
   <init-param>  
      <param-name>proxyGrantingTicketStorageClass</param-name>  
<param-value>com.yonyou.mcloud.cas.client.proxy.MemcachedBackedProxyGrantingTicketStorageImpl</param-value>  
   </init-param>  
   <!-- 解决中文问题 -->  
   <init-param>  
      <param-name>encoding</param-name>  
      <param-value>UTF-8</param-value>  
   </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>CAS Validation Filter</filter-name>  
    <url-pattern>/proxyCallback</url-pattern>  
</filter-mapping>  
<filter-mapping>  
    <filter-name>CAS Validation Filter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

**请求转换为标准的http方法**

浏览器form表单只支持get和post请求，而delete，put等method并不支持，spring3.0添加了一个过滤器，可以将这些请求转换为标准的http方法，似的支持get，post，put，delete，请求，该过滤器为HiddenHttpMethodFilter。HiddenHttpMethodFilter必须放在DispatcherServlet之前。

```
<filter>
    <filter-name>HttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>HttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

#### 6.servlet配置

Servlet通常称为服务器端小程序，是运行在服务器端的程序，用于处理及响应客户的请求。Servlet是个特殊的java类，继承于HttpServlet。客户端通常只有GET和POST两种请求方式，Servlet为了响应则两种请求，必须重写doGet()和doPost()方法。大部分时候，Servlet对于所有的请求响应都是完全一样的，此时只需要重写service()方法即可响应客户端的所有请求。

另外，HttpServlet有两个方法：

1. init(ServletConfig config)：创建Servlet实例时，调用该方法的初始化Servlet资源。
2. destroy()：销毁Servlet实例时，自动调用该方法的回收资源。

通常无需重写init()和destroy()两个方法，除非需要在初始化Servlet时，完成某些资源初始化的方法，才考虑重写init()方法，如果重写了init()方法，应在重写该方法的第一行调用super.init(config)，该方法将调用HttpServlet的init()方法。如果需要在销毁Servlet之前，先完成某些资源的回收，比如关闭数据库连接，才需要重写destory方法()。

Servlet的生命周期：
创建Servlet实例有两个时机：

1. 客户端第一次请求某个Servlet时，系统创建该Servlet的实例，大部分Servlet都是这种Servlet。
2. Web应用启动时立即创建Servlet实例，即load-on-start Servlet。

每个Servlet的运行都遵循如下生命周期：

1. 创建Servlet实例。
2. Web容器调用Servlet的init()方法，对Servlet进行初始化。
3.  Servlet初始化后，将一直存在于容器中，用于响应客户端请求，如果客户端发送GET请求，容器调用Servlet的doGet()方法处理并响应请求；如果客户端发送POST请求，容器调用Servlet的doPost()方法处理并响应请求。或者统一使用service()方法处理来响应用户请求。
4. Web容器决定销毁Servlet时，先调用Servlet的destory()方法，通常在关闭Web应用时销毁Servlet实例。

##### Servlet配置

为了让Servlet能响应用户请求，还必须将Servlet配置在web应用中，配置Servlet需要修改web.xml文件。从Servlet3.0开始，配置Servlet有两种方式：

1. 在Servlet类中使用@WebServlet Annotation进行配置。
2. 在web.xml文件中进行配置。

我们用web.xml文件来配置Servlet，需要配置<servlet\>和<servlet-mapping\>。<servlet\>用来声明一个Servlet。<icon\>、<display-name\>和<description\>元素的用法和<filter\>的用法相同。<init-param\>元素与<context-param\>元素具有相同的元素描述符，可以使用<init-param\>子元素将初始化参数名和参数值传递给Servlet，访问Servlet配置参数通过ServletConfig对象来完成，ServletConfig提供如下方法：
java.lang.String.getInitParameter(java.lang.String name)：用于获取初始化参数
ServletConfig获取配置参数的方法和ServletContext获取配置参数的方法完全一样，只是ServletConfig是取得当前Servlet的配置参数，而ServletContext是获取整个Web应用的配置参数。

1. <description\>、<display-name\>和<icon\>

   - <description\>：为Servlet指定一个文本描述。

   - <display-name\>：为Servlet提供一个简短的名字被某些工具显示。

   - <icon\>：为Servlet指定一个图标，在图形管理工具中表示该Servlet。

2. <servlet-name\>、<servlet-class\>和<jsp-file\>元素

   <servlet\>必须含有<servlet-name\>和<servlet-class\>，或者<servlet-name\>和<jsp-file\>。 描述如下：

   - <servlet-name\>用来定义servlet的名称，该名称在整个应用中必须是惟一的。

   - <servlet-class\>用来指定servlet的完全限定的名称。

   - <jsp-file\>用来指定应用中JSP文件的完整路径。这个完整路径必须由/开始。

3.  <load-on-startup\>
   如果load-on-startup元素存在，而且也指定了jsp-file元素，则JSP文件会被重新编译成Servlet，同时产生的Servlet也被载入内存。<load-on-startup\>的内容可以为空，或者是一个整数。这个值表示由Web容器载入内存的顺序。
   举个例子：如果有两个Servlet元素都含有<load-on-startup\>子元素，则<load-on-startup\>子元素值较小的Servlet将先被加载。如果<load-on-startup\>子元素值为空或负值，则由Web容器决定什么时候加载Servlet。如果两个Servlet的<load-on-startup\>子元素值相同，则由Web容器决定先加载哪一个Servlet。`<load-on-startup>1</load-on-startup>`表示启动容器时，初始化Servlet。

4. <servlet-mapping\>
   <servlet-mapping\>含有<servlet-name\>和<url-pattern\>

   - <servlet-name\>：Servlet的名字，唯一性和一致性，与<servlet\>元素中声明的名字一致。
   - <url-pattern\>：指定相对于Servlet的URL的路径。该路径相对于web应用程序上下文的根路径。<servlet-mapping\>将URL模式映射到某个Servlet，即该Servlet处理的URL。

5. 加载Servlet的过程 
   容器的Context对象对请求路径(URL)做出处理，去掉请求URL的上下文路径后，按路径映射规则和Servlet映射路径（<url- pattern\>）做匹配，如果匹配成功，则调用这个Servlet处理请求。 

```
<servlet>
    <servlet-name>test</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>test</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

**配置缺省Servlet**
如果某个Servlet的映射路径仅仅为一个正斜杠（/），那么这个Servlet就成为当前Web应用程序的缺省Servlet。
凡是在web.xml文件中找不到匹配的<servlet-mapping\>元素的URL，它们的访问请求都将交给缺省Servlet处理，也就是说，缺省Servlet用于处理所有其他Servlet都不处理的访问请求。

**classpath和classpath\*区别：**同名资源存在时，classpath只从第一个符合条件的classpath中加载资源，而classpath*会从所有的classpath中加载符合条件的资源。classpath*，需要遍历所有的classpath，效率肯定比不上classpath，因此在项目设计的初期就尽量规划好资源文件所在的路径，避免使用classpath*来加载。

#### 7.进行session失效时间设定，单位为分钟

```
<session-config>
    <session-timeout>30</session-timeout>
 </session-config>
```

#### 8.设置项目欢迎页

<welcome-file-list\>包含一个子元素<welcome-file\>，<welcome-file\>用来指定首页文件名称。<welcome-file-list\>元素可以包含一个或多个<welcome-file\>子元素。如果在第一个<welcome-file\>元素中没有找到指定的文件，Web容器就会尝试显示第二个，以此类推。

```
<!-- 配置在web.xml中的一个欢迎页，用于当用户在url中输入工程名称或者输入web容器url（如http://localhost:8080/或http://localhost:8080/test/）时直接跳转的页面-->
<welcome-file-list>
<welcome-file>login.jsp</welcome-file>
</welcome-file-list>
```

#### 9.配置错误页

当出现404错误时跳转错误页面

```
<error-page>
    <error-code>404</error-code>
    <location>/error.jsp</location>
  </error-page>
```

#### 10.jsp配置

```
<jsp-config>
    <jsp-property-group>
      <url-pattern>*.jsp</url-pattern>
      <trim-directive-whitespaces>true</trim-directive-whitespaces>
    </jsp-property-group>
  </jsp-config>
```

<jsp-property-group\>元素主要子元素

<description\>：设定的说明；

<display-name\>：设定名称；

<url-pattern\>：设定值所影响的范围，如：/CH2 或 /*.jsp；

<el-ignored\>：若为true，表示不支持EL 语法；

<scripting-invalid\>：若为true，表示不支持<% scripting %>语法；

<page-encoding\>：设定JSP 网页的编码；

<include-prelude\>：设置JSP 网页的抬头，扩展名为.jspf；

<include-coda\>：设置JSP 网页的结尾，扩展名为.jspf。

trim-directive-whitespaces=true，作用是删除页面中的多余的空格

#### 11.预防xss攻击

```
<context-param>
    <param-name>defaultHtmlEscape</param-name>
    <param-value>true</param-value>
  </context-param>
```

#### 12.验证码web配置

```
 <servlet>
    <servlet-name>Kaptcha</servlet-name>
    <servlet-class>com.linkage.eduselfapp.core.kaptcha.KaptchaServlet</servlet-class>
    <init-param>
      <param-name>kaptcha.border</param-name>
      <param-value>no</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.font.color</param-name>
      <param-value>black</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.char.length</param-name>
      <param-value>5</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.font.size</param-name>
      <param-value>25</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.image.width</param-name>
      <param-value>90</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.image.height</param-name>
      <param-value>45</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.char.space</param-name>
      <param-value>2</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.char.string</param-name>
      <param-value>23456789</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.textproducer.font.names</param-name>
      <param-value>??,Arial,Courier</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.obscurificator.impl</param-name>
      <param-value>com.google.code.kaptcha.impl.ShadowGimpy</param-value>
    </init-param>
    <init-param>
      <param-name>kaptcha.noise.impl</param-name>
      <param-value>com.google.code.kaptcha.impl.NoNoise</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>Kaptcha</servlet-name>
    <url-pattern>/kaptcha.jpg</url-pattern>
  </servlet-mapping>
```

#### 13.设置访问权限

```
<security-constraint>
    <web-resource-collection>
      <web-resource-name>web</web-resource-name>
      <url-pattern>/*</url-pattern>
      <http-method>PUT</http-method>
      <http-method>DELETE</http-method>
      <http-method>HEAD</http-method>
      <http-method>OPTIONS</http-method>
      <http-method>TRACE</http-method>
    </web-resource-collection>
    <auth-constraint/>
  </security-constraint>
  <login-config>
    <auth-method>BASIC</auth-method>
  </login-config>
```

<security-constraint\> 的子元素 <http-method\> 是可选的，如果没有 <http-method\> 元素，这表示将禁止所有 HTTP 方法访问相应的资源。 
子元素 <auth-constraint\> 需要和 <login-config\> 相配合使用，但可以被单独使用。如果没有 <auth-constraint\> 子元素，这表明任何身份的用户都可以访问相应的资源。也就是说，如果 <security-constraint\> 中没有 <auth-constraint\> 子元素的话，配置实际上是不起中用的。如果加入了 <auth-constraint\> 子元素，但是其内容为空，这表示所有身份的用户都被禁止访问相应的资源。

security-constriaint元素的用途是用来指示服务器使用何种验证方法了.此元素在web.xml中应该出现在login-config 的紧前面

login-config 为设置http规范，base64

### war包

**JAR包 ：**

JAR 文件格式以流行的 ZIP 文件格式为基础。

与 ZIP 文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。

JAR 文件与 ZIP 文件唯一的区别就是在 JAR 文件的内容中，包含了一个 META-INF/MANIFEST.MF 文件，这个文件是在生成 JAR 文件的时候自动创建的。

作用：

作为工具包和类库；这个是最基本的作用，在大型项目中，一般会依赖N多JAR包。作为应用工程和扩展的构建单元；开发大型应用的时候，一般会将应用分成几个单元，每个单元用jar包封装，并相互依赖。作为组件、applet  或者插件程序的部署单位；用于打包与组件相关联的辅助资源。

典型的jar包内部结构如下：

```
tools.jar 
  |  resource.xml                    // 资源配置文件 
  |  other.xml 
  | 
  |— META-INF           
               MANIFEST.MF          // jar包的描述文件 
  |— com                            // 类的包目录 
         |—test  
                   util.class       // java类文件
```

**WAR包 :**


WAR(Web Archive file)网络应用程序文件，是与平台无关的文件格式，它允许将许多文件组合成一个压缩文件。war专用在web方面 。JAVA WEB工程，都是打成WAR包进行发布。

典型的war包内部结构如下：

```
webapp.war 
  |    index.jsp 
  | 
  |— images 
  |— META-INF 
  |— WEB-INF 
          |   web.xml                   // WAR包的描述文件 
          | 
          |— classes 
          |          action.class       // java类文件，编译后的字节码
          | 
          |— lib 
                    other.jar           // 依赖的jar包 
                    share.jar
```

**EAR包：**

JAR（Java Archive，Java 归档文件）是与平台无关的文件格式，它允许将许多文件组合成一个压缩文件。

为 J2EE 应用程序创建的 JAR 文件是 EAR 文件（企业 JAR 文件）。

针对企业级项目，实际上EAR包中包含WAR包和几个企业级项目的配置文件而已，一般服务器选择WebSphere等，都会使用EAR包。

典型的ear包内部结构如下

```
app.ear 
   |   ejb.jar                // ejb-jar包 
   |   other.jar              // 普通的jar包 
   |   webapp.war         　　// war包 
   | 
   |—META-INF 
          application.xml 　 // EAR描述文件
```

WEB标准包是war方式，J2EE标准包使用的是ear方式，区别就在与你必须在支持j2ee的环境下才能使用ear方式，比如在tomcat中就不能使用ear方式，但是在weblogic中两种都可以，ear方式所包含的范围比war方式广很多，就好比一个大圆里面的小圆，是包含与被包含关系。

### 技术名称及官网

Spring Framework Spring容器 <http://projects.spring.io/spring-framework/>

Spring MVC框架 <http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc>

Apache Shiro 安全框架 <http://shiro.apache.org/>

Spring session 分布式Session管理 <http://projects.spring.io/spring-session/>

MyBatis ORM框架 <http://www.mybatis.org/mybatis-3/zh/index.html>

MyBatis Generator 代码生成 <http://www.mybatis.org/generator/index.html>

PageHelper MyBatis物理分页插件 <http://git.oschina.net/free/Mybatis_PageHelper>

Druid 数据库连接池 <https://github.com/alibaba/druid>

FluentValidator 校验框架 <https://github.com/neoremind/fluent-validator>

Thymeleaf 模板引擎 <http://www.thymeleaf.org/>

Velocity 模板引擎 <http://velocity.apache.org/>

ZooKeeper 分布式协调服务 <http://zookeeper.apache.org/>

Dubbo 分布式服务框架 <http://dubbo.io/>

TBSchedule & elastic-job 分布式调度框架 <https://github.com/dangdangdotcom/elastic-job>

Redis 分布式缓存数据库 <https://redis.io/>

Solr & Elasticsearch 分布式全文搜索引擎 <http://lucene.apache.org/solr/>

Java初高级一起学习分享，喜欢可以我的学习群64弍46衣3凌9，或加资料群69似64陆0吧3    <https://www.elastic.co/>

Quartz 作业调度框架 <http://www.quartz-scheduler.org/>

Ehcache 进程内缓存框架 <http://www.ehcache.org/>

ActiveMQ 消息队列 <http://activemq.apache.org/>

JStorm 实时流式计算框架 <http://jstorm.io/>

FastDFS 分布式文件系统 <https://github.com/happyfish100/fastdfs>

Log4J 日志组件 <http://logging.apache.org/log4j/1.2/>

Swagger2 接口测试框架 <http://swagger.io/>

sequence 分布式高效ID生产 <http://git.oschina.net/yu120/sequence>

AliOSS & Qiniu & QcloudCOS 云存储  <https://www.aliyun.com/product/oss/> <http://www.qiniu.com/https://www.qcloud.com/product/cos>

Protobuf & json 数据序列化 <https://github.com/google/protobuf>

Jenkins 持续集成工具 <https://jenkins.io/index.html>

Maven 项目构建管理  <http://maven.apache.org/>

### 参考

1. [Java Web——Web概述](https://www.jianshu.com/p/9719bdaa38a1)
2. [Java中的JAR/EAR/WAR包的文件夹结构说明](https://hck.iteye.com/blog/1751951)