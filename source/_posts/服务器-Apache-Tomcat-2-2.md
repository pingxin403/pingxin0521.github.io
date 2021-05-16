---
title: Apache Tomcat 源码分析 (二)
date: 2019-12-04 11:18:59
tags:
 - 服务器
 - Tomcat
categories:
 - 服务器
 - Tomcat
---

### Tomcat的启动流程

<!--more-->

基于Tomcat 8.5.55

tomcat的启动的起点是Server.start()方法，在这里它会依次启动`Container`和 `Connector`相关组件，最后到达EndPoint（Tomcat启动的Socket管理者），完成整个启动过程。如下图是个简易过程：

![luvoGQ.png](https://s2.ax1x.com/2019/12/29/luvoGQ.png)

#### Bootstrap

`initClassLoaders()` 创建以下类加载器，一般catalinaLoader 和sharedLoader 都默认和 commonLoader 为同一个

1. - `commonLoader` 加载 catalina 相关顶层公用的类，默认扫描 `CATALINA_HOME`, `CATALINA_BASE` 下的class
   - `catalinaLoader` 加载tomcat 应用相关的类
   - `sharedLoader` 加载 Web 应用的类

`catalinaLoader` 被设置为 ContextClassLoader

创建 `org.apache.catalina.startup.Catalina`。将 `Catalina.parentClassLoader` 设置为 `sharedLoader`。依次调用`Catalina.load()` 和 `Catalina.start()` 方法。

#### 初始化 Initialisation

##### Catalina.load()

1. `Catalina.load()` 读取 conf/server.xml 中的配置作为 `Digester` 的输入。根据配置生成相应的 `Server`, `Listener`, `Service`, `Engine`, `Host`, `Context` 等及其内部相关组件(如`Valve`, `Listener`)。

   `Engine`, `Host`, `Context` 的配置读取逻辑被单独放在了`EngineRuleSet`, `HostRuleSet`, `ContextRuleSet` 中。**特别需要关注伴随`Host`, `Context` 生成的`HostConfig`, `ContextConfig`。**（`EngineConfig` 没有实际内容略去）这两个 Config class 会监听 Container 的生命时间，作出许多关键配置。

2. `getServer().init()` 初始化 Server 组件

##### StandardServer.init()

`StandardServer.initInternal()` 遍历所有其 `services` 调用 `init()` 方法

##### StandardService.init()

`StandardService.initInternal()` 依次init 其管辖组件

```
Engine.init()
Executor.init()
MapperListener.init()
Connector.init()
```

LifecycleBase 的 `start` 方法中，会检查若 state == NEW 则先 `init()`，Host, Context, Wrapper 均由此方法init

##### Context.init()

ContextConfig 会监听 `AFTER_INIT_EVENT`, 并触发 `ContextConfig.init()`

创建 `Digester` 依照 `ContextRuleSet` 读取配置文件并设置相应属性。

#### 启动 Start

##### Catalina.start()

`Catalina.start()` 调用 `Server.start()`。`StandardServer` 遍历 `services` 调用 `Service.start()`

##### Service.start()

`Service.start()` 依次 start 其管辖组件

```
Engine.start()Executor.start()MapperListener.start()Connector.start()
```

##### MapperListener

1. 循环向engine 及其child containers 注册自己为 listener, 以监听containers 的变动。
2. 循环遍历 engine 及其child containers, 注册进 `Mapper` 里。

##### StandardThreadExecutor

backed by a `ThreadPoolExecutor`, with default minSpareThreads = 25, maxThreads = 200. maxQueueSize = Integer.MAX_VALUE

生成对应的ThreadPoolExecutor.

##### Connector

Tomcat的Connector是Coyote connector的一种实现，这是tomcat的官方解释：The Coyote HTTP/1.1 Connector element represents a Connector component that supports the HTTP/1.1 protocol. It enables Catalina to function as a stand-alone web server, in addition to its ability to execute servlets and JSP pages.
 Tomcat8之后默认使用nio作为接受请求策略，默认在Service启动的时候进行初始化，当然也可以单独启动，在默认的构造函数中会初始化ProtocolHandler

![luxpRJ.png](https://s2.ax1x.com/2019/12/29/luxpRJ.png)

tomcat中支持两种协议的连接器：HTTP/1.1与AJP/1.3

HTTP/1.1协议负责建立HTTP连接，web应用通过浏览器访问tomcat服务器用的就是这个连接器，默认监听的是8080端口；现在高版本也支持http2

AJP/1.3协议负责和其他HTTP服务器建立连接，监听的是8009端口，比如tomcat和apache或者iis集成时需要用到这个连接器。

![luxEdK.png](https://s2.ax1x.com/2019/12/29/luxEdK.png)

**`Connector`的启动其实就是`ProtocolHandler`的启动**，如下图：





#### Containers

`ContainerBase.startInternal()` 依次执行

1. 向`startStopExecutor` 提交 `start()` 所有子container的任务并等待完成。（`startStopExecutor` 就是一个普通的线程池）。
2. start pipeline
3. 在deamon线程上start `ContainerBackgroundProcessor` ，会定期遍历调用自己和子类的 `backgroundProcess()` 方法。

#### HostConfig

监听了Host 生命周期事件，实现了Context 的动态部署。

- 监听 Host 的 `START_EVENT` 事件
  若设置了 deployOnStartup （默认为true），则会执行 `deployApps()`

- 监听 Host `PERIODIC_EVENT` 事件
  如果设置了 `autoDeploy` （默认为true)，则会检查文件更新并执行 `deployApps()`

- 1. 检查加载的Application 是否有更新（通过文件的lastModified 来检查）
  2. 检查是否存在旧版本的Application 可以 undeploy，是则undeploy
  3. 调用 `deployApps()` 尝试搜索及deploy 新的 application。

##### deployApps()

依次

- 执行`deployDescriptors()` 搜索 `$CATALINA_BASE/{engine_name}/{host_name}` 下的 xml 文件作为context config 进行部署
- 执行`deployWARs()` 搜索部署 `appbase` 下的所有 war
- 执行`deployDirectories()` 搜索部署 `appbase` 下的所有目录

`deployWARs()`和`deployDirectories()`会以目录下的`META-INF/context.xml` 作为context的配置，`deployDescriptors()`以 `$CATALINA_BASE/{engine_name}/{host_name}/*.xml` 作为配置，生成对应的 `StandardContext`, 并调用 `addChild()` 方法，添加到当前host。
值得一提的是`addChild()`的过程中会 start 子容器，而start 方法中会检查容器状态，适时 `init()` 容器。如此完成Context 的动态加载。

#### Context.start()

简要列举`Context.start()` 的步骤

1. start WebResourceRoot
2. `new WebappLoader(getParentClassLoader())` 生成 WebappLoader。
3. start WebappLoader, 创建webapp 级别的classloader `ParallelWebappClassLoader`
4. 设置 `ParallelWebappClassLoader` 为当前 ContextClassLoader
5. 发出 `CONFIGURE_START_EVENT` 事件。
6. start child container 和 pipeline
7. 调用`initializers`里所有 `ServletContainerInitializer` 的 `onStartup` 方法
8. `listenerStart()`
9. `filterStart()`
10. `loadOnStartup(findChildren())` 实例化并init 所有 load on startup 的servlet
11. 将 ContextClassLoader 设置回去

#### ContextConfig.configureStart()

ContextConfig 监听 `CONFIGURE_START_EVENT`，调用`configureStart()`以对Context 进行配置。`configureStart()` 调用 `webConfig()` 完成配置

1. 解析global, host level 的web-fragments
2. `processServletContainerInitializers()` 搜索所有的 `ServletContainerInitializers`。这里使用了 `ClassLoader.getResources()` 来搜索所有的 `META-INF/services/javax.servlet.ServletContainerInitializer` 资源
3. 把web-fragments merge 进web.xml 配置里
4. `configureContext()` 利用merge 完的配置再次配置 Context。
5. 将web-fragment 相关的 META-INF/resource 添加进当前 Context
6. 把之前搜索到的`ServletContainerInitializer` 添加到 Context.

### Http请求处理过程

终于进行到`Connector`的分析阶段了，这也是Tomcat里面最复杂的一块功能了。`Connector`中文名为`连接器`，既然是连接器，它肯定会连接某些东西，连接些什么呢？

> `Connector`用于接受请求并将请求封装成Request和Response，然后交给`Container`进行处理，`Container`处理完之后再交给`Connector`返回给客户端。

要理解`Connector`，我们需要问自己4个问题。

1. Connector如何接受请求的？
2. 如何将请求封装成Request和Response的？
3. 封装完之后的Request和Response如何交给Container进行处理的？
4. Container处理完之后如何交给Connector并返回给客户端的？

先来一张`Connector`的整体结构图

![lnt5Pf.png](https://s2.ax1x.com/2019/12/29/lnt5Pf.png)

【注意】：不同的协议、不同的通信方式，`ProtocolHandler`会有不同的实现。在Tomcat 9.0.29中，`ProtocolHandler`的类继承层级如下图所示。

![lKC156.png](https://s2.ax1x.com/2019/12/29/lKC156.png)

针对上述的类继承层级图，我们做如下说明：

1. ajp和http11是两种不同的协议
2. nio、nio2和apr是不同的通信方式
3. 协议和通信方式可以相互组合。

`ProtocolHandler`包含三个部件：`Endpoint`、`Processor`、`Adapter`。

1. `Endpoint`用来处理底层Socket的网络连接，`Processor`用于将`Endpoint`接收到的Socket封装成Request，`Adapter`用于将Request交给Container进行具体的处理。
2. `Endpoint`由于是处理底层的Socket网络连接，因此`Endpoint`是用来实现`TCP/IP协议`的，而`Processor`用来实现`HTTP协议`的，`Adapter`将请求适配到Servlet容器进行具体的处理。
3. `Endpoint`的抽象实现类AbstractEndpoint里面定义了`Acceptor`和`AsyncTimeout`两个内部类和一个`Handler接口`。`Acceptor`用于监听请求，`AsyncTimeout`用于检查异步Request的超时，`Handler`用于处理接收到的Socket，在内部调用`Processor`进行处理。

至此，我们已经明白了问题（1）、（2）和（3）。至于（4），当我们了解了Container自然就明白了

#### Connector源码分析入口

 我们在`Service`标准实现`StandardService`的源码中发现，其`init()`、`start()`、`stop()`和`destroy()`方法分别会对Connectors的同名方法进行调用。而一个`Service`对应着多个`Connector`。

```java
//Service.init()
@Override
protected void initInternal() throws LifecycleException {
    super.initInternal();

    if (engine != null) {
        engine.init();
    }

    // Initialize any Executors
    for (Executor executor : findExecutors()) {
        if (executor instanceof JmxEnabled) {
            ((JmxEnabled) executor).setDomain(getDomain());
        }
        executor.init();
    }

    // Initialize mapper listener
    mapperListener.init();

    // Initialize our defined Connectors
    synchronized (connectorsLock) {
        for (Connector connector : connectors) {
            try {
                connector.init();
            } catch (Exception e) {
                String message = sm.getString(
                        "standardService.connector.initFailed", connector);
                log.error(message, e);

                if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"))
                    throw new LifecycleException(message);
            }
        }
    }
}

//Service.start()
@Override
protected void startInternal() throws LifecycleException {
    if(log.isInfoEnabled())
        log.info(sm.getString("standardService.start.name", this.name));
    setState(LifecycleState.STARTING);

    // Start our defined Container first
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }

    mapperListener.start();

    // Start our defined Connectors second
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            try {
                // If it has already failed, don't try and start it
                if (connector.getState() != LifecycleState.FAILED) {
                    connector.start();
                }
            } catch (Exception e) {
                log.error(sm.getString(
                        "standardService.connector.startFailed",
                        connector), e);
            }
        }
    }
}

```

我们知道`Connector`实现了`Lifecycle`接口，所以它是一个`生命周期组件`。所以`Connector`的启动逻辑入口在于`init()`和`start()`。

##### Connector构造方法

在分析之前，我们看看`server.xml`，该文件已经体现出了tomcat中各个组件的大体结构。

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```

在这个文件中，我们看到一个`Connector`有几个关键属性，`port`和`protocol`是其中的两个。`server.xml`默认支持两种协议：`HTTP/1.1`和`AJP/1.3`。其中`HTTP/1.1`用于支持http1.1协议，而`AJP/1.3`用于支持对apache服务器的通信。

接下来我们看看构造方法。

```java
public Connector() {
    this(null); // 1. 无参构造方法，传入参数为空协议，会默认使用`HTTP/1.1`
}

public Connector(String protocol) {
    setProtocol(protocol);
    // Instantiate protocol handler
    // 5. 使用protocolHandler的类名构造ProtocolHandler的实例
    ProtocolHandler p = null;
    try {
        Class<?> clazz = Class.forName(protocolHandlerClassName);
        p = (ProtocolHandler) clazz.getConstructor().newInstance();
    } catch (Exception e) {
        log.error(sm.getString(
                "coyoteConnector.protocolHandlerInstantiationFailed"), e);
    } finally {
        this.protocolHandler = p;
    }

    if (Globals.STRICT_SERVLET_COMPLIANCE) {
        uriCharset = StandardCharsets.ISO_8859_1;
    } else {
        uriCharset = StandardCharsets.UTF_8;
    }
}

@Deprecated
public void setProtocol(String protocol) {
    boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
            AprLifecycleListener.getUseAprConnector();

    // 2. `HTTP/1.1`或`null`，protocolHandler使用`org.apache.coyote.http11.Http11NioProtocol`，不考虑apr
    if ("HTTP/1.1".equals(protocol) || protocol == null) {
        if (aprConnector) {
            setProtocolHandlerClassName("org.apache.coyote.http11.Http11AprProtocol");
        } else {
            setProtocolHandlerClassName("org.apache.coyote.http11.Http11NioProtocol");
        }
    }
    // 3. `AJP/1.3`，protocolHandler使用`org.apache.coyote.ajp.AjpNioProtocol`，不考虑apr
    else if ("AJP/1.3".equals(protocol)) {
        if (aprConnector) {
            setProtocolHandlerClassName("org.apache.coyote.ajp.AjpAprProtocol");
        } else {
            setProtocolHandlerClassName("org.apache.coyote.ajp.AjpNioProtocol");
        }
    }
    // 4. 其他情况，使用传入的protocol作为protocolHandler的类名
    else {
        setProtocolHandlerClassName(protocol);
    }
}
```

从上面的代码我们看到构造方法主要做了下面几件事情：

1. 无参构造方法，传入参数为空协议，会默认使用`HTTP/1.1`
2. `HTTP/1.1`或`null`，protocolHandler使用`org.apache.coyote.http11.Http11NioProtocol`，不考虑apr
3. `AJP/1.3`，protocolHandler使用`org.apache.coyote.ajp.AjpNioProtocol`，不考虑apr
4. 其他情况，使用传入的protocol作为protocolHandler的类名
5. 使用protocolHandler的类名构造ProtocolHandler的实例

##### Connector.init()

```java
@Override
protected void initInternal() throws LifecycleException {
    super.initInternal();

    // Initialize adapter
    // 1. 初始化adapter
    adapter = new CoyoteAdapter(this);
    protocolHandler.setAdapter(adapter);

    // Make sure parseBodyMethodsSet has a default
    // 2. 设置接受body的method列表，默认为POST
    if (null == parseBodyMethodsSet) {
        setParseBodyMethods(getParseBodyMethods());
    }

    if (protocolHandler.isAprRequired() && !AprLifecycleListener.isAprAvailable()) {
        throw new LifecycleException(sm.getString("coyoteConnector.protocolHandlerNoApr",
                getProtocolHandlerClassName()));
    }
    if (AprLifecycleListener.isAprAvailable() && AprLifecycleListener.getUseOpenSSL() &&
            protocolHandler instanceof AbstractHttp11JsseProtocol) {
        AbstractHttp11JsseProtocol<?> jsseProtocolHandler =
                (AbstractHttp11JsseProtocol<?>) protocolHandler;
        if (jsseProtocolHandler.isSSLEnabled() &&
                jsseProtocolHandler.getSslImplementationName() == null) {
            // OpenSSL is compatible with the JSSE configuration, so use it if APR is available
            jsseProtocolHandler.setSslImplementationName(OpenSSLImplementation.class.getName());
        }
    }

    // 3. 初始化protocolHandler
    try {
        protocolHandler.init();
    } catch (Exception e) {
        throw new LifecycleException(
                sm.getString("coyoteConnector.protocolHandlerInitializationFailed"), e);
    }
}
```

`init()`方法做了3件事情

1. 初始化adapter
2. 设置接受body的method列表，默认为POST
3. 初始化protocolHandler

从`ProtocolHandler类继承层级`我们知道`ProtocolHandler`的子类都必须实现`AbstractProtocol`抽象类，而`protocolHandler.init();`的逻辑代码正是在这个抽象类里面。我们来分析一下。

```java
@Override
public void init() throws Exception {
    if (getLog().isInfoEnabled()) {
        getLog().info(sm.getString("abstractProtocolHandler.init", getName()));
    }

    if (oname == null) {
        // Component not pre-registered so register it
        oname = createObjectName();
        if (oname != null) {
            Registry.getRegistry(null, null).registerComponent(this, oname, null);
        }
    }

    if (this.domain != null) {
        rgOname = new ObjectName(domain + ":type=GlobalRequestProcessor,name=" + getName());
        Registry.getRegistry(null, null).registerComponent(
                getHandler().getGlobal(), rgOname, null);
    }

    // 1. 设置endpoint的名字，默认为：http-nio-{port}
    String endpointName = getName();
    endpoint.setName(endpointName.substring(1, endpointName.length()-1));
    endpoint.setDomain(domain);
    
    // 2. 初始化endpoint
    endpoint.init();
}
```

我们接着分析一下`Endpoint.init()`里面又做了什么。该方法位于`AbstactEndpoint`抽象类，该类是基于模板方法模式实现的，主要调用了子类的`bind()`方法。

```java
public abstract void bind() throws Exception;
public abstract void unbind() throws Exception;
public abstract void startInternal() throws Exception;
public abstract void stopInternal() throws Exception;

public void init() throws Exception {
    // 执行bind()方法
    if (bindOnInit) {
        bind();
        bindState = BindState.BOUND_ON_INIT;
    }
    if (this.domain != null) {
        // Register endpoint (as ThreadPool - historical name)
        oname = new ObjectName(domain + ":type=ThreadPool,name=\"" + getName() + "\"");
        Registry.getRegistry(null, null).registerComponent(this, oname, null);

        ObjectName socketPropertiesOname = new ObjectName(domain +
                ":type=ThreadPool,name=\"" + getName() + "\",subType=SocketProperties");
        socketProperties.setObjectName(socketPropertiesOname);
        Registry.getRegistry(null, null).registerComponent(socketProperties, socketPropertiesOname, null);

        for (SSLHostConfig sslHostConfig : findSslHostConfigs()) {
            registerJmx(sslHostConfig);
        }
    }
}
```

继续分析`bind()`方法，我们终于看到了我们想要看的东西了。关键的代码在于`serverSock.socket().bind(addr,getAcceptCount());`，用于绑定`ServerSocket`到指定的IP和端口。

```java
@Override
public void bind() throws Exception {

    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = (getAddress()!=null?new InetSocketAddress(getAddress(),getPort()):new InetSocketAddress(getPort()));
        //绑定ServerSocket到指定的IP和端口
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }

    serverSock.configureBlocking(true); //mimic APR behavior

    // Initialize thread count defaults for acceptor, poller
    if (acceptorThreadCount == 0) {
        // FIXME: Doesn't seem to work that well with multiple accept threads
        acceptorThreadCount = 1;
    }
    if (pollerThreadCount <= 0) {
        //minimum one poller thread
        pollerThreadCount = 1;
    }
    setStopLatch(new CountDownLatch(pollerThreadCount));

    // Initialize SSL if needed
    initialiseSsl();

    selectorPool.open();
}
```

好了，我们已经分析完了`init()`方法，接下来我们分析`start()`方法。关键代码就一行，调用`ProtocolHandler.start()`方法。

##### Connector.start()

```java
@Override
protected void startInternal() throws LifecycleException {

    // Validate settings before starting
    if (getPort() < 0) {
        throw new LifecycleException(sm.getString(
                "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
    }

    setState(LifecycleState.STARTING);

    try {
        protocolHandler.start();
    } catch (Exception e) {
        throw new LifecycleException(
                sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
    }
}
```

我们深入`ProtocolHandler.start()`方法。

1. 调用`Endpoint.start()`方法
2. 开启异步超时线程，线程执行单元为`Asynctimeout`

```java
@Override
public void start() throws Exception {
    if (getLog().isInfoEnabled()) {
        getLog().info(sm.getString("abstractProtocolHandler.start", getName()));
    }

    // 1. 调用`Endpoint.start()`方法
    endpoint.start();

    // Start async timeout thread
    // 2. 开启异步超时线程，线程执行单元为`Asynctimeout`
    asyncTimeout = new AsyncTimeout();
    Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
    int priority = endpoint.getThreadPriority();
    if (priority < Thread.MIN_PRIORITY || priority > Thread.MAX_PRIORITY) {
        priority = Thread.NORM_PRIORITY;
    }
    timeoutThread.setPriority(priority);
    timeoutThread.setDaemon(true);
    timeoutThread.start();
}
```

这儿我们重点关注`Endpoint.start()`方法

```java
public final void start() throws Exception {
    // 1. `bind()`已经在`init()`中分析过了
    if (bindState == BindState.UNBOUND) {
        bind();
        bindState = BindState.BOUND_ON_START;
    }
    startInternal();
}

@Override
public void startInternal() throws Exception {
    if (!running) {
        running = true;
        paused = false;

        processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                socketProperties.getProcessorCache());
        eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                        socketProperties.getEventCache());
        nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                socketProperties.getBufferPool());

        // Create worker collection
        // 2. 创建工作者线程池
        if ( getExecutor() == null ) {
            createExecutor();
        }
        
        // 3. 初始化连接latch，用于限制请求的并发量
        initializeConnectionLatch();

        // Start poller threads
        // 4. 开启poller线程。poller用于对接受者线程生产的消息（或事件）进行处理，poller最终调用的是Handler的代码
        pollers = new Poller[getPollerThreadCount()];
        for (int i=0; i<pollers.length; i++) {
            pollers[i] = new Poller();
            Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
            pollerThread.setPriority(threadPriority);
            pollerThread.setDaemon(true);
            pollerThread.start();
        }
        // 5. 开启acceptor线程
        startAcceptorThreads();
    }
}

protected final void startAcceptorThreads() {
    int count = getAcceptorThreadCount();
    acceptors = new Acceptor[count];

    for (int i = 0; i < count; i++) {
        acceptors[i] = createAcceptor();
        String threadName = getName() + "-Acceptor-" + i;
        acceptors[i].setThreadName(threadName);
        Thread t = new Thread(acceptors[i], threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();
    }
}
```

1. `bind()`已经在`init()`中分析过了
2. 创建工作者线程池
3. 初始化连接latch，用于限制请求的并发量
4. 创建轮询Poller线程。poller用于对接受者线程生产的消息（或事件）进行处理，poller最终调用的是Handler的代码
5. 创建Acceptor线程

#### Connector请求逻辑

分析完了`Connector`的启动逻辑之后，我们就需要进一步分析一下http的请求逻辑，当请求从客户端发起之后，需要经过哪些操作才能真正地得到执行？

#### Acceptor

Acceptor线程主要用于监听套接字，将已连接套接字转给Poller线程。Acceptor线程数由AbstracEndPoint的acceptorThreadCount成员变量控制，默认值为1

AbstractEndpoint.Acceptor是AbstractEndpoint类的静态抽象类，实现了Runnable接口，部分代码如下：

```java
public abstract static class Acceptor implements Runnable {
    public enum AcceptorState {
        NEW, RUNNING, PAUSED, ENDED
    }

    protected volatile AcceptorState state = AcceptorState.NEW;
    public final AcceptorState getState() {
        return state;
    }

    private String threadName;
    protected final void setThreadName(final String threadName) {
        this.threadName = threadName;
    }
    protected final String getThreadName() {
        return threadName;
    }
}
```

NioEndpoint的Acceptor成员内部类继承了AbstractEndpoint.Acceptor：

```java
protected class Acceptor extends AbstractEndpoint.Acceptor {
    @Override
    public void run() {
        int errorDelay = 0;

        // Loop until we receive a shutdown command
        while (running) {

            // Loop if endpoint is paused
            // 1. 运行过程中，如果`Endpoint`暂停了，则`Acceptor`进行自旋（间隔50毫秒） `       
            while (paused && running) {
                state = AcceptorState.PAUSED;
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
            // 2. 如果`Endpoint`终止运行了，则`Acceptor`也会终止
            if (!running) {
                break;
            }
            state = AcceptorState.RUNNING;

            try {
                //if we have reached max connections, wait
                // 3. 如果请求达到了最大连接数，则wait直到连接数降下来
                countUpOrAwaitConnection();

                SocketChannel socket = null;
                try {
                    // Accept the next incoming connection from the server
                    // socket
                    // 4. 接受下一次连接的socket
                    socket = serverSock.accept();
                } catch (IOException ioe) {
                    // We didn't get a socket
                    countDownConnection();
                    if (running) {
                        // Introduce delay if necessary
                        errorDelay = handleExceptionWithDelay(errorDelay);
                        // re-throw
                        throw ioe;
                    } else {
                        break;
                    }
                }
                // Successful accept, reset the error delay
                errorDelay = 0;

                // Configure the socket
                if (running && !paused) {
                    // setSocketOptions() will hand the socket off to
                    // an appropriate processor if successful
                    // 5. `setSocketOptions()`这儿是关键，会将socket以事件的方式传递给poller
                    if (!setSocketOptions(socket)) {
                        closeSocket(socket);
                    }
                } else {
                    closeSocket(socket);
                }
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error(sm.getString("endpoint.accept.fail"), t);
            }
        }
        state = AcceptorState.ENDED;
    }
}
```

从以上代码可以看到：

- countUpOrAwaitConnection函数检查当前最大连接数，若未达到maxConnections则加一，否则等待；
- socket = serverSock.accept()这一行中的serverSock正是NioEndpoint的bind函数中打开的ServerSocketChannel。为了引用这个变量，NioEndpoint的Acceptor类是成员而不再是静态类；
- setSocketOptions函数调用上的注释表明该函数将已连接套接字交给Poller线程处理。

setSocketOptions方法接着处理已连接套接字：

```java
protected boolean setSocketOptions(SocketChannel socket) {
    // Process the connection
    try {
        //disable blocking, APR style, we are gonna be polling it
        socket.configureBlocking(false);
        Socket sock = socket.socket();
        socketProperties.setProperties(sock);

        NioChannel channel = nioChannels.pop();
        if (channel == null) {
            SocketBufferHandler bufhandler = new SocketBufferHandler(
                    socketProperties.getAppReadBufSize(),
                    socketProperties.getAppWriteBufSize(),
                    socketProperties.getDirectBuffer());
            if (isSSLEnabled()) {
                channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
            } else {
                channel = new NioChannel(socket, bufhandler);
            }
        } else {
            channel.setIOChannel(socket);
            channel.reset();
        }
        // 将channel注册到poller，注意关键的两个方法，`getPoller0()`和`Poller.register()`
        getPoller0().register(channel);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        try {
            log.error("",t);
        } catch (Throwable tt) {
            ExceptionUtils.handleThrowable(tt);
        }
        // Tell to close the socket
        return false;
    }
    return true;
}
```

- 从NioChannel栈中出栈一个，若能重用（即不为null）则重用对象，否则新建一个NioChannel对象；
- getPoller0方法利用轮转法选择一个Poller线程，利用Poller类的register方法将上述NioChannel对象注册到该Poller线程上；
- 若成功转给Poller线程该函数返回true，否则返回false。返回false后，Acceptor类的closeSocket函数会关闭通道和底层Socket连接并将当前最大连接数减一。

##### Poller

Poller线程主要用于以较少的资源轮询已连接套接字以保持连接，当数据可用时转给工作线程。

Poller线程数由NioEndPoint的pollerThreadCount成员变量控制，默认值为2与可用处理器数二者之间的较小值。
Poller实现了Runnable接口，可以看到构造函数为每个Poller打开了一个新的Selector。

```java
public class Poller implements Runnable {
    private Selector selector;
    private final SynchronizedQueue<PollerEvent> events =
            new SynchronizedQueue<>();
    // 省略一些代码
    public Poller() throws IOException {
        this.selector = Selector.open();
    }

    public Selector getSelector() { return selector;}
    // 省略一些代码
}
```

将channel注册到poller，注意关键的两个方法，`getPoller0()`和`Poller.register()`。先来分析一下`getPoller0()`，该方法比较关键的一个地方就是`以取模的方式`对poller数量进行轮询获取。

```java

/**
 * The socket poller.
 */
private Poller[] pollers = null;
private AtomicInteger pollerRotater = new AtomicInteger(0);
/**
 * Return an available poller in true round robin fashion.
 *
 * @return The next poller in sequence
 */
public Poller getPoller0() {
    int idx = Math.abs(pollerRotater.incrementAndGet()) % pollers.length;
    return pollers[idx];
}
```

接下来我们分析一下`Poller.register()`方法。因为`Poller`维持了一个`events同步队列`，所以`Acceptor`接受到的channel会放在这个队列里面，放置的代码为`events.offer(event);`

```java
public class Poller implements Runnable {

    private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

    /**
     * Registers a newly created socket with the poller.
     *
     * @param socket    The newly created socket
     */
    public void register(final NioChannel socket) {
        socket.setPoller(this);
        NioSocketWrapper ka = new NioSocketWrapper(socket, NioEndpoint.this);
        socket.setSocketWrapper(ka);
        ka.setPoller(this);
        ka.setReadTimeout(getSocketProperties().getSoTimeout());
        ka.setWriteTimeout(getSocketProperties().getSoTimeout());
        ka.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());
        ka.setSecure(isSSLEnabled());
        ka.setReadTimeout(getConnectionTimeout());
        ka.setWriteTimeout(getConnectionTimeout());
        PollerEvent r = eventCache.pop();
        ka.interestOps(SelectionKey.OP_READ);//this is what OP_REGISTER turns into.
        if ( r==null) r = new PollerEvent(socket,ka,OP_REGISTER);
        else r.reset(socket,ka,OP_REGISTER);
        addEvent(r);
    }

    private void addEvent(PollerEvent event) {
        events.offer(event);
        if ( wakeupCounter.incrementAndGet() == 0 ) selector.wakeup();
    }
}
```

##### PollerEvent

接下来看一下PollerEvent，PollerEvent实现了Runnable接口，用来表示一个轮询事件，代码如下：

```java
public static class PollerEvent implements Runnable {
    private NioChannel socket;
    private int interestOps;
    private NioSocketWrapper socketWrapper;

    public PollerEvent(NioChannel ch, NioSocketWrapper w, int intOps) {
        reset(ch, w, intOps);
    }

    public void reset(NioChannel ch, NioSocketWrapper w, int intOps) {
        socket = ch;
        interestOps = intOps;
        socketWrapper = w;
    }

    public void reset() {
        reset(null, null, 0);
    }

    @Override
    public void run() {
        if (interestOps == OP_REGISTER) {
            try {
                socket.getIOChannel().register(
                        socket.getPoller().getSelector(), SelectionKey.OP_READ, socketWrapper);
            } catch (Exception x) {
                log.error(sm.getString("endpoint.nio.registerFail"), x);
            }
        } else {
            final SelectionKey key = socket.getIOChannel().keyFor(socket.getPoller().getSelector());
            try {
                if (key == null) {
                    socket.socketWrapper.getEndpoint().countDownConnection();
                    ((NioSocketWrapper) socket.socketWrapper).closed = true;
                } else {
                    final NioSocketWrapper socketWrapper = (NioSocketWrapper) key.attachment();
                    if (socketWrapper != null) {
                        //we are registering the key to start with, reset the fairness counter.
                        int ops = key.interestOps() | interestOps;
                        socketWrapper.interestOps(ops);
                        key.interestOps(ops);
                    } else {
                        socket.getPoller().cancelledKey(key);
                    }
                }
            } catch (CancelledKeyException ckx) {
                try {
                    socket.getPoller().cancelledKey(key);
                } catch (Exception ignore) {}
            }
        }
    }

}
```

在run函数中：

- 若感兴趣集是自定义的OP_REGISTER，则说明该事件表示的已连接套接字通道尚未被轮询线程处理过，那么将该通道注册到Poller线程的Selector上，感兴趣集是OP_READ，通道注册的附件是一个NioSocketWrapper对象。从Poller的register方法添加事件即是这样的过程；
- 否则获得已连接套接字通道注册到Poller线程的Selector上的SelectionKey，为key添加新的感兴趣集。

##### 重访Poller

上文提到Poller类实现了Runnable接口，其重写的run方法如下所示。

```java
public boolean events() {
    boolean result = false;
    PollerEvent pe = null;
    for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++ ) {
        result = true;
        try {
            //直接调用run方法
            pe.run();
            pe.reset();
            if (running && !paused) {
                eventCache.push(pe);
            }
        } catch ( Throwable x ) {
            log.error("",x);
        }
    }
    return result;
}

@Override
public void run() {
    // Loop until destroy() is called
    while (true) {
        boolean hasEvents = false;

        try {
            if (!close) {
                /执行PollerEvent的run方法
                hasEvents = events();
                if (wakeupCounter.getAndSet(-1) > 0) {
                    //if we are here, means we have other stuff to do
                    //do a non blocking select
                    keyCount = selector.selectNow();
                } else {
                    keyCount = selector.select(selectorTimeout);
                }
                wakeupCounter.set(0);
            }
            if (close) {
                events();
                timeout(0, false);
                try {
                    selector.close();
                } catch (IOException ioe) {
                    log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                }
                break;
            }
        } catch (Throwable x) {
            ExceptionUtils.handleThrowable(x);
            log.error("",x);
            continue;
        }
        //either we timed out or we woke up, process events first
        if ( keyCount == 0 ) hasEvents = (hasEvents | events());

        // 获取当前选择器中所有注册的“选择键(已就绪的监听事件)”
        Iterator<SelectionKey> iterator =
            keyCount > 0 ? selector.selectedKeys().iterator() : null;
        // Walk through the collection of ready keys and dispatch
        // any active event.
        // 对已经准备好的key进行处理
        while (iterator != null && iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            NioSocketWrapper attachment = (NioSocketWrapper)sk.attachment();
            // Attachment may be null if another thread has called
            // cancelledKey()
            if (attachment == null) {
                iterator.remove();
            } else {
                iterator.remove();
                // 真正处理key的地方
                processKey(sk, attachment);
            }
        }//while

        //process timeouts
        timeout(keyCount,hasEvents);
    }//while

    getStopLatch().countDown();
}
```

- 若队列里有元素则会先把队列里的事件均执行一遍，PollerEvent的run方法会将通道注册到Poller的Selector上；
- 对select返回的SelectionKey进行处理，由于在PollerEvent中注册通道时带上了NioSocketWrapper附件，因此这里可以用SelectionKey的attachment方法得到，接着调用processKey去处理已连接套接字通道。

我们接着分析`processKey()`，该方法又会根据key的类型，来分别处理读和写。

1. 处理读事件，比如生成Request对象
2. 处理写事件，比如将生成的Response对象通过socket写回客户端

```java
protected void processKey(SelectionKey sk, NioSocketWrapper attachment) {
    try {
        if ( close ) {
            cancelledKey(sk);
        } else if ( sk.isValid() && attachment != null ) {
            if (sk.isReadable() || sk.isWritable() ) {
                if ( attachment.getSendfileData() != null ) {
                    processSendfile(sk,attachment, false);
                } else {
                    unreg(sk, attachment, sk.readyOps());
                    boolean closeSocket = false;
                    // 1. 处理读事件，比如生成Request对象
                    // Read goes before write
                    if (sk.isReadable()) {
                        if (!processSocket(attachment, SocketEvent.OPEN_READ, true)) {
                            closeSocket = true;
                        }
                    }
                    // 2. 处理写事件，比如将生成的Response对象通过socket写回客户端
                    if (!closeSocket && sk.isWritable()) {
                        if (!processSocket(attachment, SocketEvent.OPEN_WRITE, true)) {
                            closeSocket = true;
                        }
                    }
                    if (closeSocket) {
                        cancelledKey(sk);
                    }
                }
            }
        } else {
            //invalid key
            cancelledKey(sk);
        }
    } catch ( CancelledKeyException ckx ) {
        cancelledKey(sk);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        log.error("",t);
    }
}
```

我们继续来分析方法`processSocket()`。

1. 从`processorCache`里面拿一个`Processor`来处理socket，`Processor`的实现为`SocketProcessor`
2. 将`Processor`放到工作线程池中执行

```java
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
        SocketEvent event, boolean dispatch) {
    try {
        if (socketWrapper == null) {
            return false;
        }
        // 1. 从`processorCache`里面拿一个`Processor`来处理socket，`Processor`的实现为`SocketProcessor`
        SocketProcessorBase<S> sc = processorCache.pop();
        if (sc == null) {
            sc = createSocketProcessor(socketWrapper, event);
        } else {
            sc.reset(socketWrapper, event);
        }
        // 2. 将`Processor`放到工作线程池中执行
        Executor executor = getExecutor();
        if (dispatch && executor != null) {
            executor.execute(sc);
        } else {
            sc.run();
        }
    } catch (RejectedExecutionException ree) {
        getLog().warn(sm.getString("endpoint.executor.fail", socketWrapper) , ree);
        return false;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        // This means we got an OOM or similar creating a thread, or that
        // the pool and its queue are full
        getLog().error(sm.getString("endpoint.process.fail"), t);
        return false;
    }
    return true;
}
```

dispatch参数表示是否要在另外的线程中处理，上文processKey各处传递的参数都是true。

- dispatch为true且工作线程池存在时会执行executor.execute(sc)，之后是由工作线程池处理已连接套接字；
- 否则继续由Poller线程自己处理已连接套接字。

AbstractEndPoint类的createSocketProcessor是抽象方法，NioEndPoint类实现了它：

```java
@Override
protected SocketProcessorBase<NioChannel> createSocketProcessor(
        SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
    return new SocketProcessor(socketWrapper, event);
}
```

接着我们分析`SocketProcessor.doRun()`方法（`SocketProcessor.run()`方法最终调用此方法）。该方法将处理逻辑交给`Handler`处理，当event为null时，则表明是一个`OPEN_READ`事件。

该类的注释说明SocketProcessor与Worker的作用等价。

```java
/**
 * This class is the equivalent of the Worker, but will simply use in an
 * external Executor thread pool.
 */
protected class SocketProcessor extends SocketProcessorBase<NioChannel> {

    public SocketProcessor(SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
        super(socketWrapper, event);
    }

    @Override
    protected void doRun() {
        NioChannel socket = socketWrapper.getSocket();
        SelectionKey key = socket.getIOChannel().keyFor(socket.getPoller().getSelector());

        try {
            int handshake = -1;

            try {
                if (key != null) {
                    if (socket.isHandshakeComplete()) {
                        // No TLS handshaking required. Let the handler
                        // process this socket / event combination.
                        handshake = 0;
                    } else if (event == SocketEvent.STOP || event == SocketEvent.DISCONNECT ||
                            event == SocketEvent.ERROR) {
                        // Unable to complete the TLS handshake. Treat it as
                        // if the handshake failed.
                        handshake = -1;
                    } else {
                        handshake = socket.handshake(key.isReadable(), key.isWritable());
                        // The handshake process reads/writes from/to the
                        // socket. status may therefore be OPEN_WRITE once
                        // the handshake completes. However, the handshake
                        // happens when the socket is opened so the status
                        // must always be OPEN_READ after it completes. It
                        // is OK to always set this as it is only used if
                        // the handshake completes.
                        event = SocketEvent.OPEN_READ;
                    }
                }
            } catch (IOException x) {
                handshake = -1;
                if (log.isDebugEnabled()) log.debug("Error during SSL handshake",x);
            } catch (CancelledKeyException ckx) {
                handshake = -1;
            }
            if (handshake == 0) {
                SocketState state = SocketState.OPEN;
                // Process the request from this socket
                // 将处理逻辑交给`Handler`处理，当event为null时，则表明是一个`OPEN_READ`事件
                if (event == null) {
                    state = getHandler().process(socketWrapper, SocketEvent.OPEN_READ);
                } else {
                    state = getHandler().process(socketWrapper, event);
                }
                if (state == SocketState.CLOSED) {
                    close(socket, key);
                }
            } else if (handshake == -1 ) {
                close(socket, key);
            } else if (handshake == SelectionKey.OP_READ){
                socketWrapper.registerReadInterest();
            } else if (handshake == SelectionKey.OP_WRITE){
                socketWrapper.registerWriteInterest();
            }
        } catch (CancelledKeyException cx) {
            socket.getPoller().cancelledKey(key);
        } catch (VirtualMachineError vme) {
            ExceptionUtils.handleThrowable(vme);
        } catch (Throwable t) {
            log.error("", t);
            socket.getPoller().cancelledKey(key);
        } finally {
            socketWrapper = null;
            event = null;
            //return to cache
            if (running && !paused) {
                processorCache.push(this);
            }
        }
    }
}
```

Handler的关键方法是process(),虽然这个方法有很多条件分支，但是逻辑却非常清楚，主要是调用Processor.process()方法。

```java
@Override
public SocketState process(SocketWrapperBase<S> wrapper, SocketEvent status) {
    try {
     
        if (processor == null) {
            processor = getProtocol().createProcessor();
            register(processor);
        }

        processor.setSslSupport(
                wrapper.getSslSupport(getProtocol().getClientCertProvider()));

        // Associate the processor with the connection
        connections.put(socket, processor);

        SocketState state = SocketState.CLOSED;
        do {
            // 关键的代码，终于找到你了
            state = processor.process(wrapper, status);

        } while ( state == SocketState.UPGRADING);
        return state;
    } 
    catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        // any other exception or error is odd. Here we log it
        // with "ERROR" level, so it will show up even on
        // less-than-verbose logs.
        getLog().error(sm.getString("abstractConnectionHandler.error"), e);
    } finally {
        ContainerThreadMarker.clear();
    }

    // Make sure socket/processor is removed from the list of current
    // connections
    connections.remove(socket);
    release(processor);
    return SocketState.CLOSED;
}
```

##### Processor

```java
//createProcessor 
protected Http11Processor createProcessor() {                          
    // 构建 Http11Processor
    Http11Processor processor = new Http11Processor(
            proto.getMaxHttpHeaderSize(), (JIoEndpoint)proto.endpoint, // 1. http header 的最大尺寸
            proto.getMaxTrailerSize(),proto.getMaxExtensionSize());
    processor.setAdapter(proto.getAdapter());
    // 2. 默认的 KeepAlive 情况下, 每个 Socket 处理的最多的 请求次数
    processor.setMaxKeepAliveRequests(proto.getMaxKeepAliveRequests());
    // 3. 开启 KeepAlive 的 Timeout
    processor.setKeepAliveTimeout(proto.getKeepAliveTimeout());      
    // 4. http 当遇到文件上传时的 默认超时时间 (300 * 1000)    
    processor.setConnectionUploadTimeout(
            proto.getConnectionUploadTimeout());                      
    processor.setDisableUploadTimeout(proto.getDisableUploadTimeout());
    // 5. 当 http 请求的 body size超过这个值时, 通过 gzip 进行压缩
    processor.setCompressionMinSize(proto.getCompressionMinSize());  
    // 6. http 请求是否开启 compression 处理    
    processor.setCompression(proto.getCompression());                  
    processor.setNoCompressionUserAgents(proto.getNoCompressionUserAgents());
    // 7. http body里面的内容是 "text/html,text/xml,text/plain" 才会进行 压缩处理
    processor.setCompressableMimeTypes(proto.getCompressableMimeTypes());
    processor.setRestrictedUserAgents(proto.getRestrictedUserAgents());
    // 8. socket 的 buffer, 默认 9000
    processor.setSocketBuffer(proto.getSocketBuffer());       
    // 9. 最大的 Post 处理尺寸的大小 4 * 1000    
    processor.setMaxSavePostSize(proto.getMaxSavePostSize());          
    processor.setServer(proto.getServer());
    processor.setDisableKeepAlivePercentage(
            proto.getDisableKeepAlivePercentage());                    
    register(processor);                                               
    return processor;
}
```

这儿我们主要关注的是`Processor`对于读的操作，也只有一行代码。调用`service()`方法。

```java
public abstract class AbstractProcessorLight implements Processor {

    @Override
    public SocketState process(SocketWrapperBase<?> socketWrapper, SocketEvent status)
            throws IOException {

        SocketState state = SocketState.CLOSED;
        Iterator<DispatchType> dispatches = null;
        do {
            if (dispatches != null) {
                DispatchType nextDispatch = dispatches.next();
                state = dispatch(nextDispatch.getSocketStatus());
            } else if (status == SocketEvent.DISCONNECT) {
                // Do nothing here, just wait for it to get recycled
            } else if (isAsync() || isUpgrade() || state == SocketState.ASYNC_END) {
                state = dispatch(status);
                if (state == SocketState.OPEN) {
                    // There may be pipe-lined data to read. If the data isn't
                    // processed now, execution will exit this loop and call
                    // release() which will recycle the processor (and input
                    // buffer) deleting any pipe-lined data. To avoid this,
                    // process it now.
                    state = service(socketWrapper);
                }
            } else if (status == SocketEvent.OPEN_WRITE) {
                // Extra write event likely after async, ignore
                state = SocketState.LONG;
            } else if (status == SocketEvent.OPEN_READ){
                // 调用`service()`方法
                state = service(socketWrapper);
            } else {
                // Default to closing the socket if the SocketEvent passed in
                // is not consistent with the current state of the Processor
                state = SocketState.CLOSED;
            }

            if (getLog().isDebugEnabled()) {
                getLog().debug("Socket: [" + socketWrapper +
                        "], Status in: [" + status +
                        "], State out: [" + state + "]");
            }

            if (state != SocketState.CLOSED && isAsync()) {
                state = asyncPostProcess();
                if (getLog().isDebugEnabled()) {
                    getLog().debug("Socket: [" + socketWrapper +
                            "], State after async post processing: [" + state + "]");
                }
            }

            if (dispatches == null || !dispatches.hasNext()) {
                // Only returns non-null iterator if there are
                // dispatches to process.
                dispatches = getIteratorAndClearDispatches();
            }
        } while (state == SocketState.ASYNC_END ||
                dispatches != null && state != SocketState.CLOSED);

        return state;
    }
}
```

`Processor.service()`方法比较重要的地方就两点。该方法非常得长，也超过了200行，在此我们不再拷贝此方法的代码。

1. 生成Request和Response对象
2. 调用`Adapter.service()`方法，将生成的Request和Response对象传进去

##### Adapter

`Adapter`用于连接`Connector`和`Container`，起到承上启下的作用。`Processor`会调用`Adapter.service()`方法。我们来分析一下，主要做了下面几件事情：

1. 根据coyote框架的request和response对象，生成connector的request和response对象（是HttpServletRequest和HttpServletResponse的封装）
2. 补充header
3. 解析请求，该方法会出现代理服务器、设置必要的header等操作
4. 真正进入容器的地方，调用Engine容器下pipeline的阀门
5. 通过request.finishRequest 与 response.finishResponse(刷OutputBuffer中的数据到浏览器) 来完成整个请求

```java
@Override
public void service(org.apache.coyote.Request req, org.apache.coyote.Response res)
        throws Exception {

    // 1. 根据coyote框架的request和response对象，生成connector的request和response对象（是HttpServletRequest和HttpServletResponse的封装）
    Request request = (Request) req.getNote(ADAPTER_NOTES);
    Response response = (Response) res.getNote(ADAPTER_NOTES);

    if (request == null) {
        // Create objects
        request = connector.createRequest();
        request.setCoyoteRequest(req);
        response = connector.createResponse();
        response.setCoyoteResponse(res);

        // Link objects
        request.setResponse(response);
        response.setRequest(request);

        // Set as notes
        req.setNote(ADAPTER_NOTES, request);
        res.setNote(ADAPTER_NOTES, response);

        // Set query string encoding
        req.getParameters().setQueryStringCharset(connector.getURICharset());
    }

    // 2. 补充header
    if (connector.getXpoweredBy()) {
        response.addHeader("X-Powered-By", POWERED_BY);
    }

    boolean async = false;
    boolean postParseSuccess = false;

    req.getRequestProcessor().setWorkerThreadName(THREAD_NAME.get());

    try {
        // Parse and set Catalina and configuration specific
        // request parameters
        // 3. 解析请求，该方法会出现代理服务器、设置必要的header等操作
        // 用来处理请求映射 (获取 host, context, wrapper, URI 后面的参数的解析, sessionId )
        postParseSuccess = postParseRequest(req, request, res, response);
        if (postParseSuccess) {
            //check valves if we support async
            request.setAsyncSupported(
                    connector.getService().getContainer().getPipeline().isAsyncSupported());
            // Calling the container
            // 4. 真正进入容器的地方，调用Engine容器下pipeline的阀门
            connector.getService().getContainer().getPipeline().getFirst().invoke(
                    request, response);
        }
        if (request.isAsync()) {
            async = true;
            ReadListener readListener = req.getReadListener();
            if (readListener != null && request.isFinished()) {
                // Possible the all data may have been read during service()
                // method so this needs to be checked here
                ClassLoader oldCL = null;
                try {
                    oldCL = request.getContext().bind(false, null);
                    if (req.sendAllDataReadEvent()) {
                        req.getReadListener().onAllDataRead();
                    }
                } finally {
                    request.getContext().unbind(false, oldCL);
                }
            }

            Throwable throwable =
                    (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);

            // If an async request was started, is not going to end once
            // this container thread finishes and an error occurred, trigger
            // the async error process
            if (!request.isAsyncCompleting() && throwable != null) {
                request.getAsyncContextInternal().setErrorState(throwable, true);
            }
        } else {
            //5. 通过request.finishRequest 与 response.finishResponse(刷OutputBuffer中的数据到浏览器) 来完成整个请求
            request.finishRequest();
            //将 org.apache.catalina.connector.Response对应的 OutputBuffer 中的数据 刷到 org.apache.coyote.Response 对应的 InternalOutputBuffer 中, 并且最终调用 socket对应的 outputStream 将数据刷出去( 这里会组装 Http Response 中的 header 与 body 里面的数据, 并且刷到远端 )
            response.finishResponse();
        }

    } catch (IOException e) {
        // Ignore
    } finally {
        AtomicBoolean error = new AtomicBoolean(false);
        res.action(ActionCode.IS_ERROR, error);

        if (request.isAsyncCompleting() && error.get()) {
            // Connection will be forcibly closed which will prevent
            // completion happening at the usual point. Need to trigger
            // call to onComplete() here.
            res.action(ActionCode.ASYNC_POST_PROCESS,  null);
            async = false;
        }

        // Access log
        if (!async && postParseSuccess) {
            // Log only if processing was invoked.
            // If postParseRequest() failed, it has already logged it.
            Context context = request.getContext();
            // If the context is null, it is likely that the endpoint was
            // shutdown, this connection closed and the request recycled in
            // a different thread. That thread will have updated the access
            // log so it is OK not to update the access log here in that
            // case.
            if (context != null) {
                context.logAccess(request, response,
                        System.currentTimeMillis() - req.getStartTime(), false);
            }
        }

        req.getRequestProcessor().setWorkerThreadName(null);

        // Recycle the wrapper request and response
        if (!async) {
            request.recycle();
            response.recycle();
        }
    }
}
```

##### 请求预处理

postParseRequest方法对请求做预处理，如对路径去除分号表示的路径参数、进行URI解码、规格化（点号和两点号）

```java
protected boolean postParseRequest(org.apache.coyote.Request req, Request request,
        org.apache.coyote.Response res, Response response) throws IOException, ServletException {
    // 省略部分代码
    MessageBytes decodedURI = req.decodedURI();

    if (undecodedURI.getType() == MessageBytes.T_BYTES) {
        // Copy the raw URI to the decodedURI
        decodedURI.duplicate(undecodedURI);

        // Parse the path parameters. This will:
        //   - strip out the path parameters
        //   - convert the decodedURI to bytes
        parsePathParameters(req, request);

        // URI decoding
        // %xx decoding of the URL
        try {
            req.getURLDecoder().convert(decodedURI, false);
        } catch (IOException ioe) {
            res.setStatus(400);
            res.setMessage("Invalid URI: " + ioe.getMessage());
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
        // Normalization
        if (!normalize(req.decodedURI())) {
            res.setStatus(400);
            res.setMessage("Invalid URI");
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
        // Character decoding
        convertURI(decodedURI, request);
        // Check that the URI is still normalized
        if (!checkNormalize(req.decodedURI())) {
            res.setStatus(400);
            res.setMessage("Invalid URI character encoding");
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
    } else {
        /* The URI is chars or String, and has been sent using an in-memory
            * protocol handler. The following assumptions are made:
            * - req.requestURI() has been set to the 'original' non-decoded,
            *   non-normalized URI
            * - req.decodedURI() has been set to the decoded, normalized form
            *   of req.requestURI()
            */
        decodedURI.toChars();
        // Remove all path parameters; any needed path parameter should be set
        // using the request object rather than passing it in the URL
        CharChunk uriCC = decodedURI.getCharChunk();
        int semicolon = uriCC.indexOf(';');
        if (semicolon > 0) {
            decodedURI.setChars
                (uriCC.getBuffer(), uriCC.getStart(), semicolon);
        }
    }

    // Request mapping.
    MessageBytes serverName;
    if (connector.getUseIPVHosts()) {
        serverName = req.localName();
        if (serverName.isNull()) {
            // well, they did ask for it
            res.action(ActionCode.REQ_LOCAL_NAME_ATTRIBUTE, null);
        }
    } else {
        serverName = req.serverName();
    }

    // Version for the second mapping loop and
    // Context that we expect to get for that version
    String version = null;
    Context versionContext = null;
    boolean mapRequired = true;

    while (mapRequired) {
        // This will map the the latest version by default
        connector.getService().getMapper().map(serverName, decodedURI,
                version, request.getMappingData());
        // 省略部分代码
    }
    // 省略部分代码
}
```

以MessageBytes的类型是T_BYTES为例：

- parsePathParameters方法去除URI中分号表示的路径参数；
- req.getURLDecoder()得到一个UDecoder实例，它的convert方法对URI解码，这里的解码只是移除百分号，计算百分号后两位的十六进制数字值以替代原来的三位百分号编码；
- normalize方法规格化URI，解释路径中的“.”和“..”；
- convertURI方法利用Connector的uriEncoding属性将URI的字节转换为字符表示；
- 注意connector.getService().getMapper().map(serverName, decodedURI, version, request.getMappingData()) 这行，之前Service启动时MapperListener注册了该Service内的各Host和Context。根据URI选择Context时，Mapper的map方法采用的是convertURI方法解码后的URI与每个Context的路径去比较

##### 容器处理

如果请求可以被传给容器的Pipeline即当postParseRequest方法返回true时，则由容器继续处理，在service方法中有connector.getService().getContainer().getPipeline().getFirst().invoke(request, response)这一行：

- Connector调用getService返回StandardService；
- StandardService调用getContainer返回StandardEngine；
- StandardEngine调用getPipeline返回与其关联的StandardPipeline；

#### Engine处理请求

我们在前面的文章中讲过**`StandardEngine`**的构造函数为自己的Pipeline添加了基本阀**`StandardEngineValve`**，代码如下：

```java
public StandardEngine() {
    super();
    pipeline.setBasic(new StandardEngineValve());
    try {
        setJvmRoute(System.getProperty("jvmRoute"));
    } catch(Exception ex) {
        log.warn(sm.getString("standardEngine.jvmRouteFail"));
    }
}
```

接下来我们看看`StandardEngineValve`的`invoke()`方法。该方法主要是选择合适的Host，然后调用Host中pipeline的第一个`Valve的invoke()`方法。

```java
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Select the Host to be used for this Request
    Host host = request.getHost();
    if (host == null) {
        response.sendError
            (HttpServletResponse.SC_BAD_REQUEST,
             sm.getString("standardEngine.noHost",
                          request.getServerName()));
        return;
    }
    if (request.isAsyncSupported()) {
        request.setAsyncSupported(host.getPipeline().isAsyncSupported());
    }

    // Ask this Host to process this request
    host.getPipeline().getFirst().invoke(request, response);
}
```

该方法很简单，校验该Engline 容器是否含有Host容器，如果不存在，返回400错误，否则继续执行 `host.getPipeline().getFirst().invoke(request, response)`，可以看到 Host 容器先获取自己的管道，再获取第一个阀门，我们再看看该阀门的 invoke 方法。

#### Host处理请求

分析Host的时候，我们从Host的构造函数入手，该方法主要是设置基础阀门。

```
public StandardHost() {
    super();
    pipeline.setBasic(new StandardHostValve());
}
```

StandardPipeline调用getFirst得到第一个阀去处理请求，由于基本阀是最后一个，所以最后会由基本阀去处理请求。

StandardHost的Pipeline里面一定有 ErrorReportValve 与 StandardHostValve两个Valve，ErrorReportValve主要是检测 Http 请求过程中是否出现过什么异常, 有异常的话, 直接拼装 html 页面, 输出到客户端。

我们看看ErrorReportValve的invoke方法：

```java
public void invoke(Request request, Response response)
    throws IOException, ServletException {
    // Perform the request
    // 1. 先将 请求转发给下一个 Valve
    getNext().invoke(request, response);  
    // 2. 这里的 isCommitted 表明, 请求是正常处理结束    
    if (response.isCommitted()) {               
        return;
    }
    // 3. 判断请求过程中是否有异常发生
    Throwable throwable = (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
    if (request.isAsyncStarted() && ((response.getStatus() < 400 &&
            throwable == null) || request.isAsyncDispatching())) {
        return;
    }
    if (throwable != null) {
        // The response is an error
        response.setError();
        // Reset the response (if possible)
        try {
            // 4. 重置 response 里面的数据(此时 Response 里面可能有些数据)
            response.reset();                  
        } catch (IllegalStateException e) {
            // Ignore
        }
         // 5. 这就是我们常看到的 500 错误码
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
    }
    response.setSuspended(false);
    try {
        // 6. 这里就是将 异常的堆栈信息组合成 html 页面, 输出到前台        
        report(request, response, throwable);                                   
    } catch (Throwable tt) {
        ExceptionUtils.handleThrowable(tt);
    }
    if (request.isAsyncStarted()) {          
        // 7. 若是异步请求的话, 设置对应的 complete (对应的是 异步 Servlet)                   
        request.getAsyncContext().complete();
    }
}
```

该方法首先执行了下个阀门的 invoke 方法。然后根据返回的Request 属性设置一些错误信息。那么下个阀门是谁呢？其实就是基础阀门了：StandardHostValve，该阀门的 invoke 的方法是如何实现的呢？

```java
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Select the Context to be used for this Request
    Context context = request.getContext();
    if (context == null) {
        response.sendError
            (HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
             sm.getString("standardHost.noContext"));
        return;
    }

    // Bind the context CL to the current thread
    if( context.getLoader() != null ) {
        // Not started - it should check for availability first
        // This should eventually move to Engine, it's generic.
        if (Globals.IS_SECURITY_ENABLED) {
            PrivilegedAction<Void> pa = new PrivilegedSetTccl(
                    context.getLoader().getClassLoader());
            AccessController.doPrivileged(pa);                
        } else {
            Thread.currentThread().setContextClassLoader
                    (context.getLoader().getClassLoader());
        }
    }
    if (request.isAsyncSupported()) {
        request.setAsyncSupported(context.getPipeline().isAsyncSupported());
    }

    // Don't fire listeners during async processing
    // If a request init listener throws an exception, the request is
    // aborted
    boolean asyncAtStart = request.isAsync(); 
    // An async error page may dispatch to another resource. This flag helps
    // ensure an infinite error handling loop is not entered
    boolean errorAtStart = response.isError();
    if (asyncAtStart || context.fireRequestInitEvent(request)) {

        // Ask this Context to process this request
        try {
            context.getPipeline().getFirst().invoke(request, response);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            if (errorAtStart) {
                container.getLogger().error("Exception Processing " +
                        request.getRequestURI(), t);
            } else {
                request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, t);
                throwable(request, response, t);
            }
        }

        // If the request was async at the start and an error occurred then
        // the async error handling will kick-in and that will fire the
        // request destroyed event *after* the error handling has taken
        // place
        if (!(request.isAsync() || (asyncAtStart &&
                request.getAttribute(
                        RequestDispatcher.ERROR_EXCEPTION) != null))) {
            // Protect against NPEs if context was destroyed during a
            // long running request.
            if (context.getState().isAvailable()) {
                if (!errorAtStart) {
                    // Error page processing
                    response.setSuspended(false);

                    Throwable t = (Throwable) request.getAttribute(
                            RequestDispatcher.ERROR_EXCEPTION);

                    if (t != null) {
                        throwable(request, response, t);
                    } else {
                        status(request, response);
                    }
                }

                context.fireRequestDestroyEvent(request);
            }
        }
    }

    // Access a session (if present) to update last accessed time, based on a
    // strict interpretation of the specification
    if (ACCESS_SESSION) {
        request.getSession(false);
    }

    // Restore the context classloader
    if (Globals.IS_SECURITY_ENABLED) {
        PrivilegedAction<Void> pa = new PrivilegedSetTccl(
                StandardHostValve.class.getClassLoader());
        AccessController.doPrivileged(pa);                
    } else {
        Thread.currentThread().setContextClassLoader
                (StandardHostValve.class.getClassLoader());
    }
}
```

首先校验了Request 是否存在 Context，其实在执行 CoyoteAdapter.postParseRequest 方法的时候就设置了，如果Context 不存在，就返回500，接着还是老套路：context.getPipeline().getFirst().invoke，该管道获取的是基础阀门：StandardContextValve，我们还是关注他的 invoke 方法。

#### Context处理请求

接着Context会去处理请求，同理，StandardContextValve的invoke方法会被调用：

```java
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {
    // Disallow any direct access to resources under WEB-INF or META-INF
    MessageBytes requestPathMB = request.getRequestPathMB();
    if ((requestPathMB.startsWithIgnoreCase("/META-INF/", 0))
            || (requestPathMB.equalsIgnoreCase("/META-INF"))
            || (requestPathMB.startsWithIgnoreCase("/WEB-INF/", 0))
            || (requestPathMB.equalsIgnoreCase("/WEB-INF"))) {
        response.sendError(HttpServletResponse.SC_NOT_FOUND);
        return;
    }

    // Select the Wrapper to be used for this Request
    Wrapper wrapper = request.getWrapper();
    if (wrapper == null || wrapper.isUnavailable()) {
        response.sendError(HttpServletResponse.SC_NOT_FOUND);
        return;
    }

    // Acknowledge the request
    try {
        response.sendAcknowledgement();
    } catch (IOException ioe) {
        container.getLogger().error(sm.getString(
                "standardContextValve.acknowledgeException"), ioe);
        request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, ioe);
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        return;
    }

    if (request.isAsyncSupported()) {
        request.setAsyncSupported(wrapper.getPipeline().isAsyncSupported());
    }
    wrapper.getPipeline().getFirst().invoke(request, response);
}
```

#### Wrapper处理请求

Wrapper是一个Servlet的包装，我们先来看看构造方法。主要作用就是设置基础阀门`StandardWrapperValve`。

```java
public StandardWrapper() {
    super();
    swValve=new StandardWrapperValve();
    pipeline.setBasic(swValve);
    broadcaster = new NotificationBroadcasterSupport();
}
```

接下来我们看看`StandardWrapperValve`的`invoke()`方法。

```java
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Initialize local variables we may need
    boolean unavailable = false;
    Throwable throwable = null;
    // This should be a Request attribute...
    long t1=System.currentTimeMillis();
    requestCount.incrementAndGet();
    StandardWrapper wrapper = (StandardWrapper) getContainer();
    Servlet servlet = null;
    Context context = (Context) wrapper.getParent();

    // Check for the application being marked unavailable
    if (!context.getState().isAvailable()) {
        response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardContext.isUnavailable"));
        unavailable = true;
    }

    // Check for the servlet being marked unavailable
    if (!unavailable && wrapper.isUnavailable()) {
        container.getLogger().info(sm.getString("standardWrapper.isUnavailable",
                wrapper.getName()));
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                    sm.getString("standardWrapper.isUnavailable",
                            wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                    sm.getString("standardWrapper.notFound",
                            wrapper.getName()));
        }
        unavailable = true;
    }

    // Allocate a servlet instance to process this request
    try {
        // 关键点1：这儿调用Wrapper的allocate()方法分配一个Servlet实例
        if (!unavailable) {
            servlet = wrapper.allocate();
        }
    } catch (UnavailableException e) {
        container.getLogger().error(
                sm.getString("standardWrapper.allocateException",
                        wrapper.getName()), e);
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardWrapper.isUnavailable",
                                    wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                       sm.getString("standardWrapper.notFound",
                                    wrapper.getName()));
        }
    } catch (ServletException e) {
        container.getLogger().error(sm.getString("standardWrapper.allocateException",
                         wrapper.getName()), StandardWrapper.getRootCause(e));
        throwable = e;
        exception(request, response, e);
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.allocateException",
                         wrapper.getName()), e);
        throwable = e;
        exception(request, response, e);
        servlet = null;
    }

    MessageBytes requestPathMB = request.getRequestPathMB();
    DispatcherType dispatcherType = DispatcherType.REQUEST;
    if (request.getDispatcherType()==DispatcherType.ASYNC) dispatcherType = DispatcherType.ASYNC;
    request.setAttribute(Globals.DISPATCHER_TYPE_ATTR,dispatcherType);
    request.setAttribute(Globals.DISPATCHER_REQUEST_PATH_ATTR,
            requestPathMB);
    // Create the filter chain for this request
    // 关键点2，创建过滤器链，类似于Pipeline的功能
    ApplicationFilterChain filterChain =
            ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);

    // Call the filter chain for this request
    // NOTE: This also calls the servlet's service() method
    try {
        if ((servlet != null) && (filterChain != null)) {
            // Swallow output if needed
            if (context.getSwallowOutput()) {
                try {
                    SystemLogHandler.startCapture();
                    if (request.isAsyncDispatching()) {
                        request.getAsyncContextInternal().doInternalDispatch();
                    } else {
                        // 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
                        filterChain.doFilter(request.getRequest(),
                                response.getResponse());
                    }
                } finally {
                    String log = SystemLogHandler.stopCapture();
                    if (log != null && log.length() > 0) {
                        context.getLogger().info(log);
                    }
                }
            } else {
                if (request.isAsyncDispatching()) {
                    request.getAsyncContextInternal().doInternalDispatch();
                } else {
                    // 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
                    filterChain.doFilter
                        (request.getRequest(), response.getResponse());
                }
            }

        }
    } catch (ClientAbortException e) {
        throwable = e;
        exception(request, response, e);
    } catch (IOException e) {
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        throwable = e;
        exception(request, response, e);
    } catch (UnavailableException e) {
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        //            throwable = e;
        //            exception(request, response, e);
        wrapper.unavailable(e);
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardWrapper.isUnavailable",
                                    wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                        sm.getString("standardWrapper.notFound",
                                    wrapper.getName()));
        }
        // Do not save exception in 'throwable', because we
        // do not want to do exception(request, response, e) processing
    } catch (ServletException e) {
        Throwable rootCause = StandardWrapper.getRootCause(e);
        if (!(rootCause instanceof ClientAbortException)) {
            container.getLogger().error(sm.getString(
                    "standardWrapper.serviceExceptionRoot",
                    wrapper.getName(), context.getName(), e.getMessage()),
                    rootCause);
        }
        throwable = e;
        exception(request, response, e);
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        throwable = e;
        exception(request, response, e);
    }

    // Release the filter chain (if any) for this request
    // 关键点4，释放掉过滤器链及其相关资源
    if (filterChain != null) {
        filterChain.release();
    }

    // 关键点5，释放掉Servlet及相关资源
    // Deallocate the allocated servlet instance
    try {
        if (servlet != null) {
            wrapper.deallocate(servlet);
        }
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.deallocateException",
                         wrapper.getName()), e);
        if (throwable == null) {
            throwable = e;
            exception(request, response, e);
        }
    }

    // If this servlet has been marked permanently unavailable,
    // unload it and release this instance
    // 关键点6，如果servlet被标记为永远不可达，则需要卸载掉它，并释放这个servlet实例
    try {
        if ((servlet != null) &&
            (wrapper.getAvailable() == Long.MAX_VALUE)) {
            wrapper.unload();
        }
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.unloadException",
                         wrapper.getName()), e);
        if (throwable == null) {
            throwable = e;
            exception(request, response, e);
        }
    }
    long t2=System.currentTimeMillis();

    long time=t2-t1;
    processingTime += time;
    if( time > maxTime) maxTime=time;
    if( time < minTime) minTime=time;
}
```

通过阅读源码，我们发现了几个关键点。现罗列如下，后面我们会逐一分析这些关键点相关的源码。

1. 关键点1：这儿调用Wrapper的allocate()方法分配一个Servlet实例
2. 关键点2，创建过滤器链，类似于Pipeline的功能
3. 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
4. 关键点4，释放掉过滤器链及其相关资源
5. 关键点5，释放掉Servlet及相关资源
6. 关键点6，如果servlet被标记为永远不可达，则需要卸载掉它，并释放这个servlet实例

**关键点1 - Wrapper分配Servlet实例**

我们来分析一下Wrapper.allocate()方法

```java
@Override
public Servlet allocate() throws ServletException {

    // If we are currently unloading this servlet, throw an exception
    // 卸载过程中，不能分配Servlet
    if (unloading) {
        throw new ServletException(sm.getString("standardWrapper.unloading", getName()));
    }

    boolean newInstance = false;

    // If not SingleThreadedModel, return the same instance every time
    // 如果Wrapper没有实现SingleThreadedModel，则每次都会返回同一个Servlet
    if (!singleThreadModel) {
        // Load and initialize our instance if necessary
        // 实例为null或者实例还未初始化，使用synchronized来保证并发时的原子性
        if (instance == null || !instanceInitialized) {
            synchronized (this) {
                if (instance == null) {
                    try {
                        if (log.isDebugEnabled()) {
                            log.debug("Allocating non-STM instance");
                        }

                        // Note: We don't know if the Servlet implements
                        // SingleThreadModel until we have loaded it.
                        // 加载Servlet
                        instance = loadServlet();
                        newInstance = true;
                        if (!singleThreadModel) {
                            // For non-STM, increment here to prevent a race
                            // condition with unload. Bug 43683, test case
                            // #3
                            countAllocated.incrementAndGet();
                        }
                    } catch (ServletException e) {
                        throw e;
                    } catch (Throwable e) {
                        ExceptionUtils.handleThrowable(e);
                        throw new ServletException(sm.getString("standardWrapper.allocate"), e);
                    }
                }
                // 初始化Servlet
                if (!instanceInitialized) {
                    initServlet(instance);
                }
            }
        }

        if (singleThreadModel) {
            if (newInstance) {
                // Have to do this outside of the sync above to prevent a
                // possible deadlock
                synchronized (instancePool) {
                    instancePool.push(instance);
                    nInstances++;
                }
            }
        }
        // 非单线程模型，直接返回已经创建的Servlet，也就是说，这种情况下只会创建一个Servlet
        else {
            if (log.isTraceEnabled()) {
                log.trace("  Returning non-STM instance");
            }
            // For new instances, count will have been incremented at the
            // time of creation
            if (!newInstance) {
                countAllocated.incrementAndGet();
            }
            return instance;
        }
    }

    // 如果是单线程模式，则使用servlet对象池技术来加载多个Servlet
    synchronized (instancePool) {
        while (countAllocated.get() >= nInstances) {
            // Allocate a new instance if possible, or else wait
            if (nInstances < maxInstances) {
                try {
                    instancePool.push(loadServlet());
                    nInstances++;
                } catch (ServletException e) {
                    throw e;
                } catch (Throwable e) {
                    ExceptionUtils.handleThrowable(e);
                    throw new ServletException(sm.getString("standardWrapper.allocate"), e);
                }
            } else {
                try {
                    instancePool.wait();
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
        }
        if (log.isTraceEnabled()) {
            log.trace("  Returning allocated STM instance");
        }
        countAllocated.incrementAndGet();
        return instancePool.pop();
    }
}
```

总结下来，注意以下几点即可：

1. 卸载过程中，不能分配Servlet
2. 如果不是单线程模式，则每次都会返回同一个Servlet（默认Servlet实现方式）
3. `Servlet`实例为`null`或者`Servlet`实例还`未初始化`，使用synchronized来保证并发时的原子性
4. 如果是单线程模式，则使用servlet对象池技术来加载多个Servlet

接下来我们看看`loadServlet()`方法

```java
public synchronized Servlet loadServlet() throws ServletException {

    // Nothing to do if we already have an instance or an instance pool
    if (!singleThreadModel && (instance != null))
        return instance;

    PrintStream out = System.out;
    if (swallowOutput) {
        SystemLogHandler.startCapture();
    }

    Servlet servlet;
    try {
        long t1=System.currentTimeMillis();
        // Complain if no servlet class has been specified
        if (servletClass == null) {
            unavailable(null);
            throw new ServletException
                (sm.getString("standardWrapper.notClass", getName()));
        }

        // 关键的地方，就是通过实例管理器，创建Servlet实例，而实例管理器是通过特殊的类加载器来加载给定的类
        InstanceManager instanceManager = ((StandardContext)getParent()).getInstanceManager();
        try {
            servlet = (Servlet) instanceManager.newInstance(servletClass);
        } catch (ClassCastException e) {
            unavailable(null);
            // Restore the context ClassLoader
            throw new ServletException
                (sm.getString("standardWrapper.notServlet", servletClass), e);
        } catch (Throwable e) {
            e = ExceptionUtils.unwrapInvocationTargetException(e);
            ExceptionUtils.handleThrowable(e);
            unavailable(null);

            // Added extra log statement for Bugzilla 36630:
            // https://bz.apache.org/bugzilla/show_bug.cgi?id=36630
            if(log.isDebugEnabled()) {
                log.debug(sm.getString("standardWrapper.instantiate", servletClass), e);
            }

            // Restore the context ClassLoader
            throw new ServletException
                (sm.getString("standardWrapper.instantiate", servletClass), e);
        }

        if (multipartConfigElement == null) {
            MultipartConfig annotation =
                    servlet.getClass().getAnnotation(MultipartConfig.class);
            if (annotation != null) {
                multipartConfigElement =
                        new MultipartConfigElement(annotation);
            }
        }

        // Special handling for ContainerServlet instances
        // Note: The InstanceManager checks if the application is permitted
        //       to load ContainerServlets
        if (servlet instanceof ContainerServlet) {
            ((ContainerServlet) servlet).setWrapper(this);
        }

        classLoadTime=(int) (System.currentTimeMillis() -t1);

        if (servlet instanceof SingleThreadModel) {
            if (instancePool == null) {
                instancePool = new Stack<>();
            }
            singleThreadModel = true;
        }

        // 调用Servlet的init方法
        initServlet(servlet);

        fireContainerEvent("load", this);

        loadTime=System.currentTimeMillis() -t1;
    } finally {
        if (swallowOutput) {
            String log = SystemLogHandler.stopCapture();
            if (log != null && log.length() > 0) {
                if (getServletContext() != null) {
                    getServletContext().log(log);
                } else {
                    out.println(log);
                }
            }
        }
    }
    return servlet;
}
```

关键的地方有两个：

1. 通过实例管理器，创建Servlet实例，而实例管理器是通过特殊的类加载器来加载给定的类
2. 调用Servlet的init方法

**关键点2 - 创建过滤器链**

创建过滤器链是调用的`org.apache.catalina.core.ApplicationFilterFactory`的`createFilterChain()`方法。我们来分析一下这个方法。该方法需要注意的地方已经在代码的comments里面说明了。

```java
public static ApplicationFilterChain createFilterChain(ServletRequest request,
        Wrapper wrapper, Servlet servlet) {

    // If there is no servlet to execute, return null
    if (servlet == null)
        return null;

    // Create and initialize a filter chain object
    // 1. 如果加密打开了，则可能会多次调用这个方法
    // 2. 为了避免重复生成filterChain对象，所以会将filterChain对象放在Request里面进行缓存
    ApplicationFilterChain filterChain = null;
    if (request instanceof Request) {
        Request req = (Request) request;
        if (Globals.IS_SECURITY_ENABLED) {
            // Security: Do not recycle
            filterChain = new ApplicationFilterChain();
        } else {
            filterChain = (ApplicationFilterChain) req.getFilterChain();
            if (filterChain == null) {
                filterChain = new ApplicationFilterChain();
                req.setFilterChain(filterChain);
            }
        }
    } else {
        // Request dispatcher in use
        filterChain = new ApplicationFilterChain();
    }

    filterChain.setServlet(servlet);
    filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());

    // Acquire the filter mappings for this Context
    StandardContext context = (StandardContext) wrapper.getParent();
    // 从这儿看出过滤器链对象里面的元素是根据Context里面的filterMaps来生成的
    FilterMap filterMaps[] = context.findFilterMaps();

    // If there are no filter mappings, we are done
    if ((filterMaps == null) || (filterMaps.length == 0))
        return (filterChain);

    // Acquire the information we will need to match filter mappings
    DispatcherType dispatcher =
            (DispatcherType) request.getAttribute(Globals.DISPATCHER_TYPE_ATTR);

    String requestPath = null;
    Object attribute = request.getAttribute(Globals.DISPATCHER_REQUEST_PATH_ATTR);
    if (attribute != null){
        requestPath = attribute.toString();
    }

    String servletName = wrapper.getName();

    // Add the relevant path-mapped filters to this filter chain
    // 类型和路径都匹配的情况下，将context.filterConfig放到过滤器链里面
    for (int i = 0; i < filterMaps.length; i++) {
        if (!matchDispatcher(filterMaps[i] ,dispatcher)) {
            continue;
        }
        if (!matchFiltersURL(filterMaps[i], requestPath))
            continue;
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
            context.findFilterConfig(filterMaps[i].getFilterName());
        if (filterConfig == null) {
            // FIXME - log configuration problem
            continue;
        }
        filterChain.addFilter(filterConfig);
    }

    // Add filters that match on servlet name second
    // 类型和servlet名称都匹配的情况下，将context.filterConfig放到过滤器链里面
    for (int i = 0; i < filterMaps.length; i++) {
        if (!matchDispatcher(filterMaps[i] ,dispatcher)) {
            continue;
        }
        if (!matchFiltersServlet(filterMaps[i], servletName))
            continue;
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
            context.findFilterConfig(filterMaps[i].getFilterName());
        if (filterConfig == null) {
            // FIXME - log configuration problem
            continue;
        }
        filterChain.addFilter(filterConfig);
    }

    // Return the completed filter chain
    return filterChain;
}
```

**关键点3 - 调用过滤器链的doFilter**

ApplicationFilterChain类的doFilter函数代码如下,它会将处理委托给internalDoFilter函数。

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response)
    throws IOException, ServletException {

    if( Globals.IS_SECURITY_ENABLED ) {
        final ServletRequest req = request;
        final ServletResponse res = response;
        try {
            java.security.AccessController.doPrivileged(
                new java.security.PrivilegedExceptionAction<Void>() {
                    @Override
                    public Void run()
                        throws ServletException, IOException {
                        internalDoFilter(req,res);
                        return null;
                    }
                }
            );
        } catch( PrivilegedActionException pe) {
            Exception e = pe.getException();
            if (e instanceof ServletException)
                throw (ServletException) e;
            else if (e instanceof IOException)
                throw (IOException) e;
            else if (e instanceof RuntimeException)
                throw (RuntimeException) e;
            else
                throw new ServletException(e.getMessage(), e);
        }
    } else {
        internalDoFilter(request,response);
    }
}
```

ApplicationFilterChain类的internalDoFilter函数代码如下：

```java
// 1. `internalDoFilter`方法通过pos和n来调用过滤器链里面的每个过滤器。pos表示当前的过滤器下标，n表示总的过滤器数量
// 2. `internalDoFilter`方法最终会调用servlet.service()方法
private void internalDoFilter(ServletRequest request,
                              ServletResponse response)
    throws IOException, ServletException {

    // Call the next filter if there is one
    // 1. 当pos小于n时, 则执行Filter
    if (pos < n) {
        // 2. 得到 过滤器 Filter，执行一次post++
        ApplicationFilterConfig filterConfig = filters[pos++];
        try {
            Filter filter = filterConfig.getFilter();

            if (request.isAsyncSupported() && "false".equalsIgnoreCase(
                    filterConfig.getFilterDef().getAsyncSupported())) {
                request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR, Boolean.FALSE);
            }
            if( Globals.IS_SECURITY_ENABLED ) {
                final ServletRequest req = request;
                final ServletResponse res = response;
                Principal principal =
                    ((HttpServletRequest) req).getUserPrincipal();

                Object[] args = new Object[]{req, res, this};
                SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
            } else {
                // 4. 这里的 filter 的执行 有点递归的感觉, 通过 pos 来控制从 filterChain 里面拿出那个 filter 来进行操作
                // 这里把this（filterChain）传到自定义filter里面，我们自定义的filter，会重写doFilter，在这里会被调用，doFilter里面会执行业务逻辑，如果执行业务逻辑成功，则会调用 filterChain.doFilter(servletRequest, servletResponse); ，filterChain就是这里传过去的this；如果业务逻辑执行失败，则return，filterChain终止，后面的servlet.service(request, response)也不会执行了
                // 所以在 Filter 里面所调用 return, 则会终止 Filter 的调用, 而下面的 Servlet.service 更本就没有调用到
                filter.doFilter(request, response, this);
            }
        } catch (IOException | ServletException | RuntimeException e) {
            throw e;
        } catch (Throwable e) {
            e = ExceptionUtils.unwrapInvocationTargetException(e);
            ExceptionUtils.handleThrowable(e);
            throw new ServletException(sm.getString("filterChain.filter"), e);
        }
        return;
    }

    // We fell off the end of the chain -- call the servlet instance
    try {
        if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
            lastServicedRequest.set(request);
            lastServicedResponse.set(response);
        }

        if (request.isAsyncSupported() && !servletSupportsAsync) {
            request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR,
                    Boolean.FALSE);
        }
        // Use potentially wrapped request from this point
        if ((request instanceof HttpServletRequest) &&
                (response instanceof HttpServletResponse) &&
                Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            Principal principal =
                ((HttpServletRequest) req).getUserPrincipal();
            Object[] args = new Object[]{req, res};
            SecurityUtil.doAsPrivilege("service",
                                       servlet,
                                       classTypeUsedInService,
                                       args,
                                       principal);
        } else {
            //当pos等于n时，过滤器都执行完毕，终于执行了熟悉的servlet.service(request, response)方法。
            servlet.service(request, response);
        }
    } catch (IOException | ServletException | RuntimeException e) {
        throw e;
    } catch (Throwable e) {
        e = ExceptionUtils.unwrapInvocationTargetException(e);
        ExceptionUtils.handleThrowable(e);
        throw new ServletException(sm.getString("filterChain.servlet"), e);
    } finally {
        if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
            lastServicedRequest.set(null);
            lastServicedResponse.set(null);
        }
    }
}
```

自定义Filter

```java
@WebFilter(urlPatterns = "/*", filterName = "myfilter")
public class FileterController implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Filter初始化中");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        System.out.println("登录逻辑");
        if("登录失败"){
            response.getWriter().write("登录失败");
            //后面的拦截器和servlet都不会执行了
            return;
        }
        //登录成功，执行下一个过滤器
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("Filter销毁中");
    }
}
```

- **pos和n是ApplicationFilterChain的成员变量，分别表示过滤器链的当前位置和过滤器总数，所以当pos小于n时，会不断执行ApplicationFilterChain的doFilter方法；**
- **当pos等于n时，过滤器都执行完毕，终于执行了熟悉的servlet.service(request, response)方法。**

### 参考

1. [Tomcat源码分析（下载、启动）](https://www.cnblogs.com/Soy-technology/p/11223604.html)