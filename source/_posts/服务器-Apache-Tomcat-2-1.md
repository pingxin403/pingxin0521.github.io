---
title: Apache Tomcat 源码分析 (一)
date: 2019-12-03 11:18:59
tags:
 - 服务器
 - Tomcat
categories:
 - 服务器
 - Tomcat
---

Tomcat的前身为Catalina，而Catalina又是一个轻量级的Servlet容器。在美国，catalina是一个很美的小岛。所以Tomcat作者的寓意可能是想把Tomcat设计成一个优雅美丽且轻量级的web服务器。Tomcat从4.x版本开始除了作为支持Servlet的容器外，额外加入了很多的功能，比如：jsp、el、naming等等，所以说Tomcat不仅仅是Catalina。

既然Tomcat首先是一个Servlet容器，我们应该更多的关心Servlet。

<!--more-->

那么，什么是Servlet呢？

在互联网兴起之初，当时的Sun公司（后面被Oracle收购）已然看到了这次机遇，于是设计出了Applet来对Web应用的支持。不过事实却并不是预期那么得好，Sun悲催地发现Applet并没有给业界带来多大的影响。经过反思，Sun就想既然机遇出现了，市场前景也非常不错，总不能白白放弃了呀，怎么办呢？于是又投入精力去搞一套规范出来，这时Servlet诞生了！

**所谓Servlet，其实就是Sun为了让Java能实现动态可交互的网页，从而进入Web编程领域而制定的一套标准！**

一个Servlet主要做下面三件事情：

1. 创建并填充Request对象，包括：URI、参数、method、请求头信息、请求体信息等
2. 创建Response对象
3. 执行业务逻辑，将结果通过Response的输出流输出到客户端

Servlet没有main方法，所以，如果要执行，则需要在一个`容器`里面才能执行，这个容器就是为了支持Servlet的功能而存在，Tomcat其实就是一个Servlet容器的实现。

#### 整体架构图

![lntyxe.png](https://s2.ax1x.com/2019/12/29/lntyxe.png)

从上图我们看出，最核心的两个组件--连接器（Connector）和容器（Container）起到`心脏`的作用，他们至关重要！他们的作用如下：

1. Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化;
2. Container用于封装和管理Servlet，以及具体处理Request请求；

一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但是可以有多个Connectors，这是因为一个服务可以有多个连接，如同时提供Http和Https链接，也可以提供向相同协议不同端口的连接,示意图如下（Engine、Host、Context下边会说到）：

![lntwUx.png](https://s2.ax1x.com/2019/12/29/lntwUx.png)

多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了！所以整个 Tomcat 的生命周期由 Server 控制。

 另外，上述的包含关系或者说是父子关系，都可以在tomcat的conf目录下的`server.xml`配置文件中看出

![lntdV1.png](https://s2.ax1x.com/2019/12/29/lntdV1.png)

上边的配置文件，还可以通过下边的一张结构图更清楚的理解：

![lntr8O.png](https://s2.ax1x.com/2019/12/29/lntr8O.png)

下面我们逐一来分析各个组件的功能：

1. `Server`表示服务器，提供了一种优雅的方式来启动和停止整个系统，不必单独启停连接器和容器
2. `Service`表示服务，`Server`可以运行多个服务。比如一个Tomcat里面可运行订单服务、支付服务、用户服务等等
3. 每个`Service`可包含`多个Connector`和`一个Container`。因为每个服务允许同时支持多种协议，但是每种协议最终执行的Servlet却是相同的
4. `Connector`表示连接器，比如一个服务可以同时支持AJP协议、Http协议和Https协议，每种协议可使用一种连接器来支持
5. Container表示容器，可以看做Servlet容器
   - `Engine` -- 引擎
   - `Host` -- 主机
   - `Context` -- 上下文
   - `Wrapper` -- 包装器
6. Service服务之下还有各种支撑组件，下面简单罗列一下这些组件
   - `Manager` -- 管理器，用于管理会话Session
   - `Logger` -- 日志器，用于管理日志
   - `Loader` -- 加载器，和类加载有关，只会开放给Context所使用
   - `Pipeline` -- 管道组件，配合Valve实现过滤器功能
   - `Valve` -- 阀门组件，配合Pipeline实现过滤器功能
   - `Realm` -- 认证授权组件

除了连接器和容器，管道组件和阀门组件也很关键，我们通过一张图来看看这两个组件

![lnthIP.png](https://s2.ax1x.com/2019/12/29/lnthIP.png)

#### Connector和Container的微妙关系

由上述内容我们大致可以知道一个请求发送到Tomcat之后，首先经过Service然后会交给我们的Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端，这样整个请求的就处理完了！

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议！

#### Connector架构分析

Connector用于接受请求并将请求封装成Request和Response，然后交给Container进行处理，Container处理完之后在交给Connector返回给客户端。

因此，我们可以把Connector分为四个方面进行理解：

1. Connector如何接受请求的？
2. 如何将请求封装成Request和Response的？
3. 封装完之后的Request和Response如何交给Container进行处理的？

首先看一下Connector的结构图，如下所示：

![lnt5Pf.png](https://s2.ax1x.com/2019/12/29/lnt5Pf.png)

Connector就是使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的，Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

1. Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。
2. Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。
3. Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。

#### Container如何处理请求的

Container处理请求是使用Pipeline-Valve管道来处理的！（Valve是阀门之意）

Pipeline-Valve是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。

但是！Pipeline-Valve使用的责任链模式和普通的责任链模式有些不同！区别主要有以下两点：

1. 每个Pipeline都有特定的Valve，而且是在管道的最后一个执行，这个Valve叫做BaseValve，BaseValve是不可删除的；

2. 在上层容器的管道的BaseValve中会调用下层容器的管道。

我们知道Container包含四个子容器，而这四个子容器对应的BaseValve分别在：StandardEngineValve、StandardHostValve、StandardContextValve、StandardWrapperValve。

Pipeline的处理流程图如下：

![lntoRS.png](https://s2.ax1x.com/2019/12/29/lntoRS.png)

1. Connector在接收到请求后会首先调用最顶层容器的Pipeline来处理，这里的最顶层容器的Pipeline就是EnginePipeline（Engine的管道）；
2. 在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve
3. 当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！
4. 当所有的Pipeline-Valve都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给Connector了，Connector在通过Socket的方式将结果返回给客户端。

### 生命周期

Lifecycle，其实就是一个状态机，对组件的由生到死状态的管理。我们来看看其总的状态转换图，如下图所示：

![lK9Jns.png](https://s2.ax1x.com/2019/12/29/lK9Jns.png)

- 当组件在`STARTING_PREP`、`STARTING`或`STARTED`时，调用`start()`方法没有任何效果
- 当组件在`NEW`状态时，调用`start()`方法会导致`init()`方法被立刻执行，随后`start()`方法被执行
- 当组件在`STOPPING_PREP`、`STOPPING`或`STOPPED`时，调用`stop()`方法没有任何效果
- 当一个组件在`NEW`状态时，调用`stop()`方法会将组件状态变更为`STOPPED`，比较典型的场景就是组件启动失败，其子组件还没有启动。当一个组件停止的时候，它将尝试停止它下面的所有子组件，即使子组件还没有启动。

#### Lifecycle方法

我们看看Lifecycle有哪些方法，如下所示：

```java
public interface Lifecycle {
    // 添加监听器
    public void addLifecycleListener(LifecycleListener listener);
    // 获取所监有听器
    public LifecycleListener[] findLifecycleListeners();
    // 移除某个监听器
    public void removeLifecycleListener(LifecycleListener listener);
    // 初始化方法
    public void init() throws LifecycleException;
    // 启动方法
    public void start() throws LifecycleException;
    // 停止方法，和start对应
    public void stop() throws LifecycleException;
    // 销毁方法，和init对应
    public void destroy() throws LifecycleException;
    // 获取生命周期状态
    public LifecycleState getState();
    // 获取字符串类型的生命周期状态
    public String getStateName();
}
```

#### LifecycleBase

`LifecycleBase`是`Lifecycle`的基本实现。我们逐一来看Lifecycle的各个方法。

##### 增加、删除和获取监听器

```java
private final List<LifecycleListener> lifecycleListeners = new CopyOnWriteArrayList<>();

@Override
public void addLifecycleListener(LifecycleListener listener) {
    lifecycleListeners.add(listener);
}
@Override
public LifecycleListener[] findLifecycleListeners() {
    return lifecycleListeners.toArray(new LifecycleListener[0]);
}
@Override
public void removeLifecycleListener(LifecycleListener listener) {
    lifecycleListeners.remove(listener);
}
```

1. 生命周期监听器保存在一个线程安全的List中，`CopyOnWriteArrayList`。所以add和remove都是直接调用此List的相应方法。
2. findLifecycleListeners返回的是一个数组，为了线程安全，所以这儿会生成一个新数组。

##### init()

```java
@Override
public final synchronized void init() throws LifecycleException {
    // 非NEW状态，不允许调用init()方法
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }

    try {
        // 初始化逻辑之前，先将状态变更为`INITIALIZING`
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        // 初始化，该方法为一个abstract方法，需要组件自行实现
        initInternal();
        // 初始化完成之后，状态变更为`INITIALIZED`
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
        // 初始化的过程中，可能会有异常抛出，这时需要捕获异常，并将状态变更为`FAILED`
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.initFail",toString()), t);
    }
}
```

`setStateInternal`方法用于维护状态，同时在状态转换成功之后触发事件。为了状态的可见性，所以state声明为volatile类型的。

```java
private volatile LifecycleState state = LifecycleState.NEW;

private synchronized void setStateInternal(LifecycleState state,
        Object data, boolean check) throws LifecycleException {
    if (log.isDebugEnabled()) {
        log.debug(sm.getString("lifecycleBase.setState", this, state));
    }

    // 是否校验状态
    if (check) {
        // Must have been triggered by one of the abstract methods (assume
        // code in this class is correct)
        // null is never a valid state
        // state不允许为null
        if (state == null) {
            invalidTransition("null");
            // Unreachable code - here to stop eclipse complaining about
            // a possible NPE further down the method
            return;
        }

        // Any method can transition to failed
        // startInternal() permits STARTING_PREP to STARTING
        // stopInternal() permits STOPPING_PREP to STOPPING and FAILED to
        // STOPPING
        if (!(state == LifecycleState.FAILED ||
                (this.state == LifecycleState.STARTING_PREP &&
                        state == LifecycleState.STARTING) ||
                (this.state == LifecycleState.STOPPING_PREP &&
                        state == LifecycleState.STOPPING) ||
                (this.state == LifecycleState.FAILED &&
                        state == LifecycleState.STOPPING))) {
            // No other transition permitted
            invalidTransition(state.name());
        }
    }

    // 设置状态
    this.state = state;
    // 触发事件
    String lifecycleEvent = state.getLifecycleEvent();
    if (lifecycleEvent != null) {
        fireLifecycleEvent(lifecycleEvent, data);
    }
}
```

我们看看`fireLifecycleEvent`方法，

```java
public void fireLifecycleEvent(String type, Object data) {
    // 事件监听,观察者模式的另一种方式
    LifecycleEvent event = new LifecycleEvent(lifecycle, type, data);
    LifecycleListener interested[] = listeners;// 监听器数组 关注 事件(启动或者关闭事件)
    // 循环通知所有生命周期时间侦听器
    for (int i = 0; i < interested.length; i++)
        // 每个监听器都有自己的逻辑
        interested[i].lifecycleEvent(event);
}
```

 首先, 创建一个事件对象, 然通知所有的监听器发生了该事件.并做响应.

##### start()

```java
@Override
public final synchronized void start() throws LifecycleException {
    // `STARTING_PREP`、`STARTING`和`STARTED时，将忽略start()逻辑
    if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
            LifecycleState.STARTED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
        }

        return;
    }

    // `NEW`状态时，执行init()方法
    if (state.equals(LifecycleState.NEW)) {
        init();
    }

    // `FAILED`状态时，执行stop()方法
    else if (state.equals(LifecycleState.FAILED)) {
        stop();
    }

    // 不是`INITIALIZED`和`STOPPED`时，则说明是非法的操作
    else if (!state.equals(LifecycleState.INITIALIZED) &&
            !state.equals(LifecycleState.STOPPED)) {
        invalidTransition(Lifecycle.BEFORE_START_EVENT);
    }

    try {
        // start前的状态设置
        setStateInternal(LifecycleState.STARTING_PREP, null, false);
        // start逻辑，抽象方法，由组件自行实现
        startInternal();
        // start过程中，可能因为某些原因失败，这时需要stop操作
        if (state.equals(LifecycleState.FAILED)) {
            // This is a 'controlled' failure. The component put itself into the
            // FAILED state so call stop() to complete the clean-up.
            stop();
        } else if (!state.equals(LifecycleState.STARTING)) {
            // Shouldn't be necessary but acts as a check that sub-classes are
            // doing what they are supposed to.
            invalidTransition(Lifecycle.AFTER_START_EVENT);
        } else {
            // 设置状态为STARTED
            setStateInternal(LifecycleState.STARTED, null, false);
        }
    } catch (Throwable t) {
        // This is an 'uncontrolled' failure so put the component into the
        // FAILED state and throw an exception.
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(sm.getString("lifecycleBase.startFail", toString()), t);
    }
}
```

##### stop()

```java
@Override
public final synchronized void stop() throws LifecycleException {
    // `STOPPING_PREP`、`STOPPING`和STOPPED时，将忽略stop()的执行
    if (LifecycleState.STOPPING_PREP.equals(state) || LifecycleState.STOPPING.equals(state) ||
            LifecycleState.STOPPED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStopped", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStopped", toString()));
        }

        return;
    }

    // `NEW`状态时，直接将状态变更为`STOPPED`
    if (state.equals(LifecycleState.NEW)) {
        state = LifecycleState.STOPPED;
        return;
    }

    // stop()的执行，必须要是`STARTED`和`FAILED`
    if (!state.equals(LifecycleState.STARTED) && !state.equals(LifecycleState.FAILED)) {
        invalidTransition(Lifecycle.BEFORE_STOP_EVENT);
    }

    try {
        // `FAILED`时，直接触发BEFORE_STOP_EVENT事件
        if (state.equals(LifecycleState.FAILED)) {
            // Don't transition to STOPPING_PREP as that would briefly mark the
            // component as available but do ensure the BEFORE_STOP_EVENT is
            // fired
            fireLifecycleEvent(BEFORE_STOP_EVENT, null);
        } else {
            // 设置状态为STOPPING_PREP
            setStateInternal(LifecycleState.STOPPING_PREP, null, false);
        }

        // stop逻辑，抽象方法，组件自行实现
        stopInternal();

        // Shouldn't be necessary but acts as a check that sub-classes are
        // doing what they are supposed to.
        if (!state.equals(LifecycleState.STOPPING) && !state.equals(LifecycleState.FAILED)) {
            invalidTransition(Lifecycle.AFTER_STOP_EVENT);
        }
        // 设置状态为STOPPED
        setStateInternal(LifecycleState.STOPPED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(sm.getString("lifecycleBase.stopFail",toString()), t);
    } finally {
        if (this instanceof Lifecycle.SingleUse) {
            // Complete stop process first
            setStateInternal(LifecycleState.STOPPED, null, false);
            destroy();
        }
    }
}
```

##### destroy()

```java
@Override
public final synchronized void destroy() throws LifecycleException {
    // `FAILED`状态时，直接触发stop()逻辑
    if (LifecycleState.FAILED.equals(state)) {
        try {
            // Triggers clean-up
            stop();
        } catch (LifecycleException e) {
            // Just log. Still want to destroy.
            log.warn(sm.getString(
                    "lifecycleBase.destroyStopFail", toString()), e);
        }
    }

    // `DESTROYING`和`DESTROYED`时，忽略destroy的执行
    if (LifecycleState.DESTROYING.equals(state) ||
            LifecycleState.DESTROYED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyDestroyed", toString()), e);
        } else if (log.isInfoEnabled() && !(this instanceof Lifecycle.SingleUse)) {
            // Rather than have every component that might need to call
            // destroy() check for SingleUse, don't log an info message if
            // multiple calls are made to destroy()
            log.info(sm.getString("lifecycleBase.alreadyDestroyed", toString()));
        }

        return;
    }

    // 非法状态判断
    if (!state.equals(LifecycleState.STOPPED) &&
            !state.equals(LifecycleState.FAILED) &&
            !state.equals(LifecycleState.NEW) &&
            !state.equals(LifecycleState.INITIALIZED)) {
        invalidTransition(Lifecycle.BEFORE_DESTROY_EVENT);
    }

    try {
        // destroy前状态设置
        setStateInternal(LifecycleState.DESTROYING, null, false);
       // 抽象方法，组件自行实现
        destroyInternal();
        // destroy后状态设置
        setStateInternal(LifecycleState.DESTROYED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.destroyFail",toString()), t);
    }
}
```

##### 模板方法

从上述源码看得出来，`LifecycleBase`是使用了状态机+模板模式来实现的。模板方法有下面这几个：

```java
// 初始化方法
protected abstract void initInternal() throws LifecycleException;
// 启动方法
protected abstract void startInternal() throws LifecycleException;
// 停止方法
protected abstract void stopInternal() throws LifecycleException;
// 销毁方法
protected abstract void destroyInternal() throws LifecycleException;
```

#### 总结

Lifecycle其实非常简单，代码也不复杂，但是剖析其实现对于我们理解组件的生命周期有很大的帮助，也有助于我们对设计模式的回顾。

### Valve

`Valve`作为一个个基础的阀门，扮演着业务实际执行者的角色。我们看看`Valve`这个接口有哪些方法。

```java
public interface Valve {
    // 获取下一个阀门
    public Valve getNext();
    // 设置下一个阀门
    public void setNext(Valve valve);
    // 后台执行逻辑，主要在类加载上下文中使用到
    public void backgroundProcess();
    // 执行业务逻辑
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
    // 是否异步执行
    public boolean isAsyncSupported();
}
```

#### Contained

`ValveBase`、`Pipeline`及其他相关组件都实现了`Contained`接口，我们看看这个接口有哪些方法。很简单，就是get/set容器操作。

```java
public interface Contained {

    /**
     * Get the {@link Container} with which this instance is associated.
     *
     * @return The Container with which this instance is associated or
     *         <code>null</code> if not associated with a Container
     */
    Container getContainer();


    /**
     * Set the <code>Container</code> with which this instance is associated.
     *
     * @param container The Container instance with which this instance is to
     *  be associated, or <code>null</code> to disassociate this instance
     *  from any Container
     */
    void setContainer(Container container);
}
```

#### ValveBase

从Valve的类层次结构，我们发现几乎所有Valve都继承了`ValveBase`这个抽象类，所以这儿我们需要分析一下它。

![lntHMQ.png](https://s2.ax1x.com/2019/12/29/lntHMQ.png)

```java
public abstract class ValveBase extends LifecycleMBeanBase implements Contained, Valve {
    // 国际化管理器，可以支持多国语言
    protected static final StringManager sm = StringManager.getManager(ValveBase.class);

    //------------------------------------------------------ Instance Variables

    // 无参构造方法，默认不支持异步
    public ValveBase() {
        this(false);
    }
    // 有参构造方法，可传入异步支持标记
    public ValveBase(boolean asyncSupported) {
        this.asyncSupported = asyncSupported;
    }


    //------------------------------------------------------ Instance Variables

    // 异步标记
    protected boolean asyncSupported;
    // 所属容器
    protected Container container = null;
    // 容器日志组件对象
    protected Log containerLog = null;
    // 下一个阀门
    protected Valve next = null;


    //-------------------------------------------------------------- Properties

    // 获取所属容器
    @Override
    public Container getContainer() {
        return container;
    }
    // 设置所属容器
    @Override
    public void setContainer(Container container) {
        this.container = container;
    }
    // 是否异步执行
    @Override
    public boolean isAsyncSupported() {
        return asyncSupported;
    }
    // 设置是否异步执行
    public void setAsyncSupported(boolean asyncSupported) {
        this.asyncSupported = asyncSupported;
    }
    // 获取下一个待执行的阀门
    @Override
    public Valve getNext() {
        return next;
    }
    // 设置下一个待执行的阀门
    @Override
    public void setNext(Valve valve) {
        this.next = valve;
    }


    //---------------------------------------------------------- Public Methods

    // 后台执行，子类实现
    @Override
    public void backgroundProcess() {
        // NOOP by default
    }
    // 初始化逻辑
    @Override
    protected void initInternal() throws LifecycleException {
        super.initInternal();
        // 设置容器日志组件对象到当前阀门的containerLog属性
        containerLog = getContainer().getLogger();
    }
    // 启动逻辑
    @Override
    protected synchronized void startInternal() throws LifecycleException {
        setState(LifecycleState.STARTING);
    }
    // 停止逻辑
    @Override
    protected synchronized void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);
    }
    // 重写toString，格式为[${containerName}]
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder(this.getClass().getName());
        sb.append('[');
        if (container == null) {
            sb.append("Container is null");
        } else {
            sb.append(container.getName());
        }
        sb.append(']');
        return sb.toString();
    }


    // -------------------- JMX and Registration  --------------------

    // 设置获取MBean对象的keyProperties，格式如：a=b,c=d,e=f...
    @Override
    public String getObjectNameKeyProperties() {
        StringBuilder name = new StringBuilder("type=Valve");

        Container container = getContainer();

        name.append(container.getMBeanKeyProperties());

        int seq = 0;

        // Pipeline may not be present in unit testing
        Pipeline p = container.getPipeline();
        if (p != null) {
            for (Valve valve : p.getValves()) {
                // Skip null valves
                if (valve == null) {
                    continue;
                }
                // Only compare valves in pipeline until we find this valve
                if (valve == this) {
                    break;
                }
                if (valve.getClass() == this.getClass()) {
                    // Duplicate valve earlier in pipeline
                    // increment sequence number
                    seq ++;
                }
            }
        }

        if (seq > 0) {
            name.append(",seq=");
            name.append(seq);
        }

        String className = this.getClass().getName();
        int period = className.lastIndexOf('.');
        if (period >= 0) {
            className = className.substring(period + 1);
        }
        name.append(",name=");
        name.append(className);

        return name.toString();
    }
    // 获取所属域，从container获取
    @Override
    public String getDomainInternal() {
        Container c = getContainer();
        if (c == null) {
            return null;
        } else {
            return c.getDomain();
        }
    }
}
```

### Pipeline

`Pipeline`作为一个管道，我们可以简单认为是一个Valve的集合，内部会对这个集合进行遍历，调用每个元素的业务逻辑方法`invoke()`。

是不是这样呢？我们还是分析一下源码，先看看接口定义。

```java
public interface Pipeline {
    // ------------------------------------------------------------- Properties

    // 获取基本阀门
    public Valve getBasic();
    // 设置基本阀门
    public void setBasic(Valve valve);

    // --------------------------------------------------------- Public Methods

    // 添加阀门
    public void addValve(Valve valve);
    // 获取阀门数组
    public Valve[] getValves();
    // 删除阀门
    public void removeValve(Valve valve);
    // 获取首个阀门
    public Valve getFirst();
    // 管道内所有阀门是否异步执行
    public boolean isAsyncSupported();
    // 获取管道所属的容器
    public Container getContainer();
    // 设置管道所属的容器
    public void setContainer(Container container);
    // 查找非异步执行的所有阀门，并放置到result参数中，所以result不允许为null
    public void findNonAsyncValves(Set<String> result);
}
```

#### StandardPipeline

接着我们分析一下`Pipeline`唯一的实现`StandardPipeline`。代码很长，但是都很简单。

```java
public class StandardPipeline extends LifecycleBase
        implements Pipeline, Contained {

    private static final Log log = LogFactory.getLog(StandardPipeline.class);

    // ----------------------------------------------------------- Constructors

    // 构造一个没有所属容器的管道
    public StandardPipeline() {
        this(null);
    }

    // 构造一个有所属容器的管道
    public StandardPipeline(Container container) {
        super();
        setContainer(container);
    }

    // ----------------------------------------------------- Instance Variables

    /**
     * 基本阀门，最后执行的阀门
     */
    protected Valve basic = null;

    /**
     * 管道所属的容器
     */
    protected Container container = null;

    /**
     * 管道里面的首个执行的阀门
     */
    protected Valve first = null;

    // --------------------------------------------------------- Public Methods

    // 是否异步执行，如果一个阀门都没有，或者所有阀门都是异步执行的，才返回true
    @Override
    public boolean isAsyncSupported() {
        Valve valve = (first!=null)?first:basic;
        boolean supported = true;
        while (supported && valve!=null) {
            supported = supported & valve.isAsyncSupported();
            valve = valve.getNext();
        }
        return supported;
    }

    // 查找所有未异步执行的阀门
    @Override
    public void findNonAsyncValves(Set<String> result) {
        Valve valve = (first!=null) ? first : basic;
        while (valve != null) {
            if (!valve.isAsyncSupported()) {
                result.add(valve.getClass().getName());
            }
            valve = valve.getNext();
        }
    }

    // ------------------------------------------------------ Contained Methods

    // 获取所属容器
    @Override
    public Container getContainer() {
        return (this.container);
    }

    // 设置所属容器
    @Override
    public void setContainer(Container container) {
        this.container = container;
    }

    // 初始化逻辑，默认没有任何逻辑
    @Override
    protected void initInternal() {
        // NOOP
    }

    // 开始逻辑，调用所有阀门的start方法
    @Override
    protected synchronized void startInternal() throws LifecycleException {
        // Start the Valves in our pipeline (including the basic), if any
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).start();
            current = current.getNext();
        }

        setState(LifecycleState.STARTING);
    }

    // 停止逻辑，调用所有阀门的stop方法
    @Override
    protected synchronized void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);

        // Stop the Valves in our pipeline (including the basic), if any
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).stop();
            current = current.getNext();
        }
    }

    // 销毁逻辑，移掉所有阀门，调用removeValve方法
    @Override
    protected void destroyInternal() {
        Valve[] valves = getValves();
        for (Valve valve : valves) {
            removeValve(valve);
        }
    }

    /**
     * 重新toString方法
     */
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("Pipeline[");
        sb.append(container);
        sb.append(']');
        return sb.toString();
    }

    // ------------------------------------------------------- Pipeline Methods

    // 获取基础阀门
    @Override
    public Valve getBasic() {
        return (this.basic);
    }

    // 设置基础阀门
    @Override
    public void setBasic(Valve valve) {
        // Change components if necessary
        Valve oldBasic = this.basic;
        if (oldBasic == valve)
            return;

        // Stop the old component if necessary
        // 老的基础阀门会被调用stop方法且所属容器置为null
        if (oldBasic != null) {
            if (getState().isAvailable() && (oldBasic instanceof Lifecycle)) {
                try {
                    ((Lifecycle) oldBasic).stop();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.setBasic: stop", e);
                }
            }
            if (oldBasic instanceof Contained) {
                try {
                    ((Contained) oldBasic).setContainer(null);
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                }
            }
        }

        // Start the new component if necessary
        // 新的阀门会设置所属容器，并调用start方法
        if (valve == null)
            return;
        if (valve instanceof Contained) {
            ((Contained) valve).setContainer(this.container);
        }
        if (getState().isAvailable() && valve instanceof Lifecycle) {
            try {
                ((Lifecycle) valve).start();
            } catch (LifecycleException e) {
                log.error("StandardPipeline.setBasic: start", e);
                return;
            }
        }

        // Update the pipeline
        // 替换pipeline中的基础阀门，就是讲基础阀门的前一个阀门的next指向当前阀门
        Valve current = first;
        while (current != null) {
            if (current.getNext() == oldBasic) {
                current.setNext(valve);
                break;
            }
            current = current.getNext();
        }

        this.basic = valve;
    }

    // 添加阀门
    @Override
    public void addValve(Valve valve) {
        // Validate that we can add this Valve
        // 设置所属容器
        if (valve instanceof Contained)
            ((Contained) valve).setContainer(this.container);

        // Start the new component if necessary
        // 调用阀门的start方法
        if (getState().isAvailable()) {
            if (valve instanceof Lifecycle) {
                try {
                    ((Lifecycle) valve).start();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.addValve: start: ", e);
                }
            }
        }

        // Add this Valve to the set associated with this Pipeline
        // 设置阀门，将阀门添加到基础阀门的前一个
        if (first == null) {
            first = valve;
            valve.setNext(basic);
        } else {
            Valve current = first;
            while (current != null) {
                if (current.getNext() == basic) {
                    current.setNext(valve);
                    valve.setNext(basic);
                    break;
                }
                current = current.getNext();
            }
        }

        container.fireContainerEvent(Container.ADD_VALVE_EVENT, valve);
    }

    // 获取阀门数组
    @Override
    public Valve[] getValves() {
        ArrayList<Valve> valveList = new ArrayList<>();
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            valveList.add(current);
            current = current.getNext();
        }

        return valveList.toArray(new Valve[0]);
    }

    // JMX方法，在此忽略
    public ObjectName[] getValveObjectNames() {
        ArrayList<ObjectName> valveList = new ArrayList<>();
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof JmxEnabled) {
                valveList.add(((JmxEnabled) current).getObjectName());
            }
            current = current.getNext();
        }

        return valveList.toArray(new ObjectName[0]);
    }

    // 移除阀门
    @Override
    public void removeValve(Valve valve) {
        Valve current;
        if(first == valve) {
            // 如果待移出的阀门是首个阀门，则首个阀门的下一个阀门变成首个阀门
            first = first.getNext();
            current = null;
        } else {
            current = first;
        }
        // 遍历阀门集合，并进行移除
        while (current != null) {
            if (current.getNext() == valve) {
                current.setNext(valve.getNext());
                break;
            }
            current = current.getNext();
        }

        if (first == basic) first = null;

        // 设置阀门所属容器为null
        if (valve instanceof Contained)
            ((Contained) valve).setContainer(null);

        // 调用待移除阀门的stop方法和destroy方法，并触发移除阀门事件
        if (valve instanceof Lifecycle) {
            // Stop this valve if necessary
            if (getState().isAvailable()) {
                try {
                    ((Lifecycle) valve).stop();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.removeValve: stop: ", e);
                }
            }
            try {
                ((Lifecycle) valve).destroy();
            } catch (LifecycleException e) {
                log.error("StandardPipeline.removeValve: destroy: ", e);
            }
        }

        container.fireContainerEvent(Container.REMOVE_VALVE_EVENT, valve);
    }

    // 获取首个阀门，如果阀门列表为null，返回基础阀门
    @Override
    public Valve getFirst() {
        if (first != null) {
            return first;
        }
        return basic;
    }
}
```

#### 总结

通过上面的代码分析，我们发现了几个关键的设计模式：

1. 模板方法模式，父类定义框架，子类实现
2. 责任链模式，就是这儿的管道/阀门的实现方式，每个阀门维护一个next属性指向下一个阀门

### Tomcat 类加载

java类加载参考：[https://pingxin0521.coding.me/2019/06/09/Java-JVM-4-3/#%E7%B1%BB%E5%8A%A0%E8%BD%BD](https://pingxin0521.coding.me/2019/06/09/Java-JVM-4-3/#类加载)

#### Tomcat 的类加载器是怎么设计的？

首先，我们来问个问题：

**Tomcat 如果使用默认的类加载机制行不行？**

我们思考一下：Tomcat是个web容器， 那么它要解决什么问题：

1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖**同一个第三方类库的不同版本**，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。
2. 部署在同一个web容器中相同的类库**相同的版本可以共享**。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的。
3. **web容器也有自己依赖的类库**，不能于应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。
4. web容器要支持jsp的修改，我们知道，jsp 文件最终也是要编译成class文件才能在虚拟机中运行，但程序运行后修改jsp已经是司空见惯的事情，否则要你何用？ 所以，web容器需要支持 jsp 修改后不用重启。

再看看我们的问题：Tomcat 如果使用默认的类加载机制行不行？

答案是不行的。为什么？我们看，第一个问题，如果使用默认的类加载器机制，那么是无法加载两个相同类库的不同版本的，默认的类加器是不管你是什么版本的，只在乎你的全限定类名，并且只有一份。第二个问题，默认的类加载器是能够实现的，因为他的职责就是保证唯一性。第三个问题和第一个问题一样。我们再看第四个问题，我们想我们要怎么实现jsp文件的热修改（楼主起的名字），jsp 文件其实也就是class文件，那么如果修改了，但类名还是一样，类加载器会直接取方法区中已经存在的，修改后的jsp是不会重新加载的。那么怎么办呢？我们可以直接卸载掉这jsp文件的类加载器，所以你应该想到了，每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件。

**Tomcat 如何实现自己独特的类加载机制？**

我们看看他们的设计图：

![lntXaq.png](https://s2.ax1x.com/2019/12/29/lntXaq.png)

我们在这张图中看到很多类加载器，除了Jdk自带的类加载器，我们尤其关心Tomcat自身持有的类加载器。仔细一点我们很容易发现：Catalina类加载器和Shared类加载器，他们并不是父子关系，而是兄弟关系。为啥这样设计，我们得分析一下每个类加载器的用途，才能知晓。

1. Common类加载器，负责加载Tomcat和Web应用都复用的类
2. Catalina类加载器，负责加载Tomcat专用的类，而这些被加载的类在Web应用中将不可见
3. Shared类加载器，负责加载Tomcat下所有的Web应用程序都复用的类，而这些被加载的类在Tomcat中将不可见
4. WebApp类加载器，负责加载具体的某个Web应用程序所使用到的类，而这些被加载的类在Tomcat和其他的Web应用程序都将不可见
5. Jsp类加载器，每个jsp页面一个类加载器，不同的jsp页面有不同的类加载器，方便实现jsp页面的热插拔

#### 源码阅读

Tomcat启动的入口在`Bootstrap的main()方法`。`main()`方法执行前，必然先执行其`static{}`块。所以我们首先分析`static{}`块，然后分析`main()`方法

```java
//Bootstrap.static{}
static {
    // 获取用户目录
    // Will always be non-null
    String userDir = System.getProperty("user.dir");

    // 第一步，从环境变量中获取catalina.home，在没有获取到的时候将执行后面的获取操作
    // Home first
    String home = System.getProperty(Globals.CATALINA_HOME_PROP);
    File homeFile = null;

    if (home != null) {
        File f = new File(home);
        try {
            homeFile = f.getCanonicalFile();
        } catch (IOException ioe) {
            homeFile = f.getAbsoluteFile();
        }
    }

    // 第二步，在第一步没获取的时候，从bootstrap.jar所在目录的上一级目录获取
    if (homeFile == null) {
        // First fall-back. See if current directory is a bin directory
        // in a normal Tomcat install
        File bootstrapJar = new File(userDir, "bootstrap.jar");

        if (bootstrapJar.exists()) {
            File f = new File(userDir, "..");
            try {
                homeFile = f.getCanonicalFile();
            } catch (IOException ioe) {
                homeFile = f.getAbsoluteFile();
            }
        }
    }

    // 第三步，第二步中的bootstrap.jar可能不存在，这时我们直接把user.dir作为我们的home目录
    if (homeFile == null) {
        // Second fall-back. Use current directory
        File f = new File(userDir);
        try {
            homeFile = f.getCanonicalFile();
        } catch (IOException ioe) {
            homeFile = f.getAbsoluteFile();
        }
    }

    // 重新设置catalinaHome属性
    catalinaHomeFile = homeFile;
    System.setProperty(
            Globals.CATALINA_HOME_PROP, catalinaHomeFile.getPath());

    // 接下来获取CATALINA_BASE（从系统变量中获取），若不存在，则将CATALINA_BASE保持和CATALINA_HOME相同
    // Then base
    String base = System.getProperty(Globals.CATALINA_BASE_PROP);
    if (base == null) {
        catalinaBaseFile = catalinaHomeFile;
    } else {
        File baseFile = new File(base);
        try {
            baseFile = baseFile.getCanonicalFile();
        } catch (IOException ioe) {
            baseFile = baseFile.getAbsoluteFile();
        }
        catalinaBaseFile = baseFile;
    }
   // 重新设置catalinaBase属性
    System.setProperty(
            Globals.CATALINA_BASE_PROP, catalinaBaseFile.getPath());
}
```

我们把代码中的注释搬下来总结一下：

1. 获取用户目录
2. 第一步，从环境变量中获取catalina.home，在没有获取到的时候将执行后面的获取操作
3. 第二步，在第一步没获取的时候，从bootstrap.jar所在目录的上一级目录获取
4. 第三步，第二步中的bootstrap.jar可能不存在，这时我们直接把user.dir作为我们的home目录
5. 重新设置catalinaHome属性
6. 接下来获取CATALINA_BASE（从系统变量中获取），若不存在，则将CATALINA_BASE保持和CATALINA_HOME相同
7. 重新设置catalinaBase属性

简单总结一下，就是加载并设置catalinaHome和catalinaBase相关的信息，以备后续使用。

##### main()

main方法大体分成两块，一块为**init**，另一块为**load+start**。

```java
public static void main(String args[]) {
    // 第一块，main方法第一次执行的时候，daemon肯定为null，所以直接new了一个Bootstrap对象，然后执行其init()方法
    if (daemon == null) {
        // Don't set daemon until init() has completed
        Bootstrap bootstrap = new Bootstrap();
        try {
            bootstrap.init();
        } catch (Throwable t) {
            handleThrowable(t);
            t.printStackTrace();
            return;
        }
        // daemon守护对象设置为bootstrap
        daemon = bootstrap;
    } else {
        // When running as a service the call to stop will be on a new
        // thread so make sure the correct class loader is used to prevent
        // a range of class not found exceptions.
        Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
    }

    // 第二块，执行守护对象的load方法和start方法
    try {
        String command = "start";
        if (args.length > 0) {
            command = args[args.length - 1];
        }

        if (command.equals("startd")) {
            args[args.length - 1] = "start";
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stopd")) {
            args[args.length - 1] = "stop";
            daemon.stop();
        } else if (command.equals("start")) {
            daemon.setAwait(true);
            daemon.load(args);
            daemon.start();
            if (null == daemon.getServer()) {
                System.exit(1);
            }
        } else if (command.equals("stop")) {
            daemon.stopServer(args);
        } else if (command.equals("configtest")) {
            daemon.load(args);
            if (null == daemon.getServer()) {
                System.exit(1);
            }
            System.exit(0);
        } else {
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");
        }
    } catch (Throwable t) {
        // Unwrap the Exception for clearer error reporting
        if (t instanceof InvocationTargetException &&
                t.getCause() != null) {
            t = t.getCause();
        }
        handleThrowable(t);
        t.printStackTrace();
        System.exit(1);
    }
}
```

我们点到`init()`里面去看看~

```java
public void init() throws Exception {
    // 非常关键的地方，初始化类加载器s，后面我们会详细具体地分析这个方法
    initClassLoaders();

    // 设置上下文类加载器为catalinaLoader，这个类加载器负责加载Tomcat专用的类
    Thread.currentThread().setContextClassLoader(catalinaLoader);
    // 暂时略过，后面会讲
    SecurityClassLoad.securityClassLoad(catalinaLoader);

    // 使用catalinaLoader加载我们的Catalina类
    // Load our startup class and call its process() method
    if (log.isDebugEnabled())
        log.debug("Loading startup class");
    Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
    Object startupInstance = startupClass.getConstructor().newInstance();

    // 设置Catalina类的parentClassLoader属性为sharedLoader
    // Set the shared extensions class loader
    if (log.isDebugEnabled())
        log.debug("Setting startup class properties");
    String methodName = "setParentClassLoader";
    Class<?> paramTypes[] = new Class[1];
    paramTypes[0] = Class.forName("java.lang.ClassLoader");
    Object paramValues[] = new Object[1];
    paramValues[0] = sharedLoader;
    Method method =
        startupInstance.getClass().getMethod(methodName, paramTypes);
    method.invoke(startupInstance, paramValues);

    // catalina守护对象为刚才使用catalinaLoader加载类、并初始化出来的Catalina对象
    catalinaDaemon = startupInstance;
}
```

关键的方法`initClassLoaders`，这个方法负责初始化Tomcat的类加载器。通过这个方法，我们很容易验证我们上一小节提到的Tomcat类加载图。

```java
private void initClassLoaders() {
    try {
        // 创建commonLoader，如果未创建成果的话，则使用应用程序类加载器作为commonLoader
        commonLoader = createClassLoader("common", null);
        if( commonLoader == null ) {
            // no config file, default to this loader - we might be in a 'single' env.
            commonLoader=this.getClass().getClassLoader();
        }
        // 创建catalinaLoader，父类加载器为commonLoader
        catalinaLoader = createClassLoader("server", commonLoader);
        // 创建sharedLoader，父类加载器为commonLoader
        sharedLoader = createClassLoader("shared", commonLoader);
    } catch (Throwable t) {
        // 如果创建的过程中出现异常了，日志记录完成之后直接系统退出
        handleThrowable(t);
        log.error("Class loader creation threw exception", t);
        System.exit(1);
    }
}
```

所有的类加载器的创建都使用到了方法createClassLoader，所以，我们进一步分析一下这个方法。createClassLoader用到了CatalinaProperties.getProperty("xxx")方法，这个方法用于从conf/catalina.properties文件获取属性值。

```java
private ClassLoader createClassLoader(String name, ClassLoader parent)
    throws Exception {
    // 获取类加载器待加载的位置，如果为空，则不需要加载特定的位置，使用父类加载返回回去。
    String value = CatalinaProperties.getProperty(name + ".loader");
    if ((value == null) || (value.equals("")))
        return parent;
    // 替换属性变量，比如：${catalina.base}、${catalina.home}
    value = replace(value);

    List<Repository> repositories = new ArrayList<>();

   // 解析属性路径变量为仓库路径数组
    String[] repositoryPaths = getPaths(value);

    // 对每个仓库路径进行repositories设置。我们可以把repositories看成一个个待加载的位置对象，可以是一个classes目录，一个jar文件目录等等
    for (String repository : repositoryPaths) {
        // Check for a JAR URL repository
        try {
            @SuppressWarnings("unused")
            URL url = new URL(repository);
            repositories.add(
                    new Repository(repository, RepositoryType.URL));
            continue;
        } catch (MalformedURLException e) {
            // Ignore
        }

        // Local repository
        if (repository.endsWith("*.jar")) {
            repository = repository.substring
                (0, repository.length() - "*.jar".length());
            repositories.add(
                    new Repository(repository, RepositoryType.GLOB));
        } else if (repository.endsWith(".jar")) {
            repositories.add(
                    new Repository(repository, RepositoryType.JAR));
        } else {
            repositories.add(
                    new Repository(repository, RepositoryType.DIR));
        }
    }
    // 使用类加载器工厂创建一个类加载器
    return ClassLoaderFactory.createClassLoader(repositories, parent);
}
```

我们来分析一下`ClassLoaderFactory.createClassLoader`--类加载器工厂创建类加载器。

```java
public static ClassLoader createClassLoader(List<Repository> repositories,
                                            final ClassLoader parent)
    throws Exception {

    if (log.isDebugEnabled())
        log.debug("Creating new class loader");

    // Construct the "class path" for this class loader
    Set<URL> set = new LinkedHashSet<>();
    // 遍历repositories，对每个repository进行类型判断，并生成URL，每个URL我们都要校验其有效性，有效的URL我们会放到URL集合中
    if (repositories != null) {
        for (Repository repository : repositories)  {
            if (repository.getType() == RepositoryType.URL) {
                URL url = buildClassLoaderUrl(repository.getLocation());
                if (log.isDebugEnabled())
                    log.debug("  Including URL " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.DIR) {
                File directory = new File(repository.getLocation());
                directory = directory.getCanonicalFile();
                if (!validateFile(directory, RepositoryType.DIR)) {
                    continue;
                }
                URL url = buildClassLoaderUrl(directory);
                if (log.isDebugEnabled())
                    log.debug("  Including directory " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.JAR) {
                File file=new File(repository.getLocation());
                file = file.getCanonicalFile();
                if (!validateFile(file, RepositoryType.JAR)) {
                    continue;
                }
                URL url = buildClassLoaderUrl(file);
                if (log.isDebugEnabled())
                    log.debug("  Including jar file " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.GLOB) {
                File directory=new File(repository.getLocation());
                directory = directory.getCanonicalFile();
                if (!validateFile(directory, RepositoryType.GLOB)) {
                    continue;
                }
                if (log.isDebugEnabled())
                    log.debug("  Including directory glob "
                        + directory.getAbsolutePath());
                String filenames[] = directory.list();
                if (filenames == null) {
                    continue;
                }
                for (int j = 0; j < filenames.length; j++) {
                    String filename = filenames[j].toLowerCase(Locale.ENGLISH);
                    if (!filename.endsWith(".jar"))
                        continue;
                    File file = new File(directory, filenames[j]);
                    file = file.getCanonicalFile();
                    if (!validateFile(file, RepositoryType.JAR)) {
                        continue;
                    }
                    if (log.isDebugEnabled())
                        log.debug("    Including glob jar file "
                            + file.getAbsolutePath());
                    URL url = buildClassLoaderUrl(file);
                    set.add(url);
                }
            }
        }
    }

    // Construct the class loader itself
    final URL[] array = set.toArray(new URL[set.size()]);
    if (log.isDebugEnabled())
        for (int i = 0; i < array.length; i++) {
            log.debug("  location " + i + " is " + array[i]);
        }

    // 从这儿看，最终所有的类加载器都是URLClassLoader的对象~~
    return AccessController.doPrivileged(
            new PrivilegedAction<URLClassLoader>() {
                @Override
                public URLClassLoader run() {
                    if (parent == null)
                        return new URLClassLoader(array);
                    else
                        return new URLClassLoader(array, parent);
                }
            });
}
```

我们已经对initClassLoaders分析完了，接下来分析SecurityClassLoad.securityClassLoad，我们看看里面做了什么事情

```java
public static void securityClassLoad(ClassLoader loader) throws Exception {
    securityClassLoad(loader, true);
}

static void securityClassLoad(ClassLoader loader, boolean requireSecurityManager) throws Exception {

    if (requireSecurityManager && System.getSecurityManager() == null) {
        return;
    }

    loadCorePackage(loader);
    loadCoyotePackage(loader);
    loadLoaderPackage(loader);
    loadRealmPackage(loader);
    loadServletsPackage(loader);
    loadSessionPackage(loader);
    loadUtilPackage(loader);
    loadValvesPackage(loader);
    loadJavaxPackage(loader);
    loadConnectorPackage(loader);
    loadTomcatPackage(loader);
}

 private static final void loadCorePackage(ClassLoader loader) throws Exception {
    final String basePackage = "org.apache.catalina.core.";
    loader.loadClass(basePackage + "AccessLogAdapter");
    loader.loadClass(basePackage + "ApplicationContextFacade$PrivilegedExecuteMethod");
    loader.loadClass(basePackage + "ApplicationDispatcher$PrivilegedForward");
    loader.loadClass(basePackage + "ApplicationDispatcher$PrivilegedInclude");
    loader.loadClass(basePackage + "ApplicationPushBuilder");
    loader.loadClass(basePackage + "AsyncContextImpl");
    loader.loadClass(basePackage + "AsyncContextImpl$AsyncRunnable");
    loader.loadClass(basePackage + "AsyncContextImpl$DebugException");
    loader.loadClass(basePackage + "AsyncListenerWrapper");
    loader.loadClass(basePackage + "ContainerBase$PrivilegedAddChild");
    loadAnonymousInnerClasses(loader, basePackage + "DefaultInstanceManager");
    loader.loadClass(basePackage + "DefaultInstanceManager$AnnotationCacheEntry");
    loader.loadClass(basePackage + "DefaultInstanceManager$AnnotationCacheEntryType");
    loader.loadClass(basePackage + "ApplicationHttpRequest$AttributeNamesEnumerator");
}
```

这儿其实就是使用catalinaLoader加载tomcat源代码里面的各个专用类。我们大致罗列一下待加载的类所在的package：

1. org.apache.catalina.core.*
2. org.apache.coyote.*
3. org.apache.catalina.loader.*
4. org.apache.catalina.realm.*
5. org.apache.catalina.servlets.*
6. org.apache.catalina.session.*
7. org.apache.catalina.util.*
8. org.apache.catalina.valves.*
9. javax.servlet.http.Cookie
10. org.apache.catalina.connector.*
11. org.apache.tomcat.*

好了，至此我们已经分析完了init里面涉及到的几个关键方法

##### WebApp类加载器

到这儿，我们隐隐感觉到少分析了点什么！没错，就是WebApp类加载器。整个启动过程分析下来，我们仍然没有看到这个类加载器。它又是在哪儿出现的呢？

我们知道WebApp类加载器是Web应用私有的，而每个Web应用其实算是一个Context，那么我们通过Context的实现类应该可以发现。在Tomcat中，Context的默认实现为`StandardContext`，我们看看这个类的`startInternal()方法`，在这儿我们发现了我们感兴趣的WebApp类加载器。

```java
protected synchronized void startInternal() throws LifecycleException {
    if (getLoader() == null) {
        WebappLoader webappLoader = new WebappLoader(getParentClassLoader());
        webappLoader.setDelegate(getDelegate());
        setLoader(webappLoader);
    }
}
```

入口代码非常简单，就是webappLoader不存在的时候创建一个，并调用`setLoader`方法。

