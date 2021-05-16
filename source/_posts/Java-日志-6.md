---
title: 日志框架--JCL
date: 2019-05-12 22:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

Jakarta Commons Logging (JCL)提供的是一个日志(Log)接口(interface)，同时兼顾轻量级和不依赖于具体的日志实现工具。它提供给中间件/日志工具开发者一个简单的日志操作抽象，允许程序开发人员使用不同的具体日志实现工具。

[官网](http://commons.apache.org/proper/commons-logging/)



<!--more-->

JCL这个日志框架跟Log4J，Java Logging API等日志框架不同。JCL采用了设计模式中的“适配器模式”，它对外提供统一的接口，然后在适配类中将对日志的操作委托给具体的日志框架，比如Log4J,Java Logging API等。

#### 简单使用

Maven依赖：

```
<dependency>
	<groupId>commons-logging</groupId>
	<artifactId>commons-logging</artifactId>
	<version>1.2</version>
</dependency>
```

使用：

```
public class commons_loggingDemo {
    Log log= LogFactory.getLog(commons_loggingDemo.class);
    @Test
    public void test() throws IOException {
        log.debug("Debug info.");
        log.info("Info info");
        log.warn("Warn info");
        log.error("Error info");
        log.fatal("Fatal info");
    }
}
```

接下来，在classpath下定义配置文件：commons-logging.properties：

```
#指定日志对象：
org.apache.commons.logging.Log = org.apache.commons.logging.impl.Jdk14Logger
#指定日志工厂：
org.apache.commons.logging.LogFactory = org.apache.commons.logging.impl.LogFactoryImpl
```

在我们的项目中，如果只单纯的依赖了commons-logging，那么默认使用的日志对象就是Jdk14Logger，默认使用的日志工厂就是LogFactoryImpl

#### 接口和类

在JCL中对外有两个统一的接口，分别是Log和LogFactory。

Commons Logging定义了一个自己的`接口 org.apache.commons.logging.Log`，以屏蔽不同日志框架的API差异，这里其实就是用到了`Adapter Pattern（适配器模式）`。

前Commons Logging 1.1.1版本已经实现的适配器实现类有:

1. org.apache.commons.logging.impl.AvalonLogger。
2. org.apache.commons.logging.impl.Jdk13LumberjackLogger:对JDK1.3以及以前版本的日志对象封装
3. org.apache.commons.logging.impl.Jdk14Logger:使用 JDK1.4
4. org.apache.commons.logging.impl.Log4JLogger:使用 Log4J
5. org.apache.commons.logging.impl.LogKitLogger。
6. org.apache.commons.logging.impl.NoOpLog。 common-logging 自带日志实现类。它实现了 Log 接口。 其输出日志的方法中不进行任何操作。
7. org.apache.commons.logging.impl.SimpleLog: common-logging 自带日志实现类。将`日志信息输出至控制台`，建议只是用于开发环境，不要用于生产环境

Commons Loggins 通过一个`抽象工厂类 LogFactory 实现配置文件的加载解析以及实例化日志框架类`。这里面就用到了两个常用的设计模式：`Factory Pattern（工厂模式）、Single Pattern（单例模式）`。

同时，Commons Logging 实现了一个`默认的工厂类 org.apache.commons.logging.impl.LogFactoryImpl`，在未指明工厂实现类的情况下就使用它。

有必要详细说明一下`调用LogFactory.getLog()时发生的事情`。`调用该函数会启动一个发现过程，即找出必需的底层日志记录功能的实现`，具体的发现过程在下面列出：

1. Commons的Logging`首先在CLASSPATH中查找commons-logging.properties文件`。这个属性文件至少定义 org.apache.commons.logging.Log属性，它的值应该是上述任意Log接口实现的完整限定名称。`如果找到 org.apache.commons.logging.Log属相，则使用该属相对应的日志组件。结束发现过程`。
2. 如果上面的步骤失败（文件不存在或属相不存在），`Commons的Logging接着检查系统属性 org.apache.commons.logging.Log`。如果找到org.apache.commons.logging.Log系统属性，则使用该系统属性对应的日志组件。结束发现过程。
3. 如果找不到org.apache.commons.logging.Log系统属性，`Logging接着在CLASSPATH中寻找log4j的类`。如果找到了，Logging就假定应用要使用的是log4j。不过这时log4j本身的属性仍要通过log4j.properties文件正确配置。结束发现过程。
4. 如果`上述查找均不能找到适当的Logging API`，但应用程序正运行在JRE 1.4或更高版本上，则`默认使用JRE 1.4的日志记录功能`。结束发现过程。
5. 最后，如果上述操作都失败（JRE 版本也低于1.4），则应用将`使用内建的SimpleLog`。`SimpleLog把所有日志信息直接输出到System.err`。结束发现过程。

### 实现原理

1. 完整的配置文件内容

```
# 默认值true
org.apache.commons.logging.Log.allowFlawedContext=true
# 默认值true
org.apache.commons.logging.Log.allowFlawedDiscovery=true
# 默认值true
org.apache.commons.logging.Log.allowFlawedHierarchy=true
# 默认值false
use_tccl=false
# 默认值0
priority=0
configId=
# 默认值
org.apache.commons.logging.LogFactory=org.apache.commons.logging.impl.LogFactoryImpl
org.apache.commons.logging.Log=
org.apache.commons.logging.diagnostics.dest=STDERR
```

实际中使用的时候`只需要配置 org.apache.commons.logging.Log=org.apache.commons.logging.impl.Log4JLogger`，或者其他的适配器实现类以及根据项目实际需要实现的自定义适配器类。

2. 使用方法：

```
private Log logger = LogFactory.getLog(Hello.class);  
或 
private Log logger = LogFactory.getLog("Hello");
```

LogFactory.getLog(JulJclTest.class)的源码如下：

```
public static Log getLog(Class clazz) throws LogConfigurationException {
    return getFactory().getInstance(clazz);
}
```

它的执行序列图如下：

![3.png](https://i.loli.net/2019/05/18/5cdfefa0c345624189.png)

1. 首先`从缓存中获取工厂实现类实例`，如果没有就读取配置文件中的`配置项 org.apache.commons.logging.LogFactory 获取工厂实现类并进行实例化`，如果没有配置或者找不到对应的类，就`使用默认的工厂类 org.apache.commons.logging.impl.LogFactoryImpl 进行初始化并放入缓存`。
2. `根据”name“从缓存中获取日志适配器实现类实例，如果没有就新生成一个并放入缓存`。

 **获取LogFactory的过程** 

从下面几种途径来获取LogFactory：

1. 系统属性中获取，即如下形式

```
System.getProperty("org.apache.commons.logging.LogFactory")
```

2. 使用`java的SPI机制`，来搜寻对应的实现，对于java的SPI机制，详细内容可以自行搜索，这里不再说明。搜寻路径如下：

```
META-INF/services/org.apache.commons.logging.LogFactory
```

简单来说就是搜寻哪些jar包中含有搜寻含有上述文件，该文件中指明了对应的LogFactory实现。

3. 从commons-logging的配置文件中寻找commons-logging也是可以拥有自己的配置文件的，名字为`commons-logging.properties，只不过目前大多数情况下，我们都没有去使用它`。如果使用了该配置文件，尝试从配置文件中读取属性`"org.apache.commons.logging.LogFactory"`对应的值。

4. 最后还没找到的话，使用默认的`org.apache.commons.logging.impl.LogFactoryImpl`：LogFactoryImpl是commons-logging提供的默认实现。



根据LogFactory获取Log的过程，这时候就需要`寻找底层是选用哪种类型的日志`，就以commons-logging提供的默认实现为例，来详细看下这个过程：

1. 从commons-logging的配置文件中寻找Log实现类的类名：从`commons-logging.properties`配置文件中寻找属性为`"org.apache.commons.logging.Log"`对应的Log类名。
2. 从系统属性中寻找Log实现类的类名，即如下方式获取：

```
System.getProperty("org.apache.commons.logging.Log");
```

3. 如果上述方式没找到，则从`classesToDiscover属性`中寻找，classesToDiscover属性值如下：

```
private static final String[] classesToDiscover = {
    "org.apache.commons.logging.impl.Log4JLogger",
    "org.apache.commons.logging.impl.Jdk14Logger",
    "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
    "org.apache.commons.logging.impl.SimpleLog"
};
```

它会尝试根据上述类名，依次进行创建，如果能创建成功，则使用该Log，然后返回给用户。

下面针对具体的日志框架，看看commons-logging是如何集成的？

### jcl与jul、log4j1、log4j2、logback的集成原理 

前面介绍了jdk自带的logging、log4j1、log4j2、logback等实际的日志框架。对于开发者而言，每种日志都有不同的写法。如果我们以实际的日志框架来进行编写，代码就限制死了，之后就很难再更换日志系统，很难做到无缝切换。

java web开发就经常提到一项原则：`面向接口编程，而不是面向实现编程`，所以我们应该是按照一套统一的API来进行日志编程，实际的日志框架来实现这套API，这样的话，即使更换日志框架，也可以做到无缝切换。

这就是commons-logging与slf4j的初衷。

下面就来介绍下commons-logging这个门面如何与上述四个实际的日志框架进行集成的呢。介绍之前先说明下日志简称：

- jdk自带的logging -> 简称 jul (java-util-logging)
- apache commons-logging -> 简称 jcl

#### commons-logging与jul集成

对应的maven依赖是：

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```

##### 使用案例

```
private static Log logger=LogFactory.getLog(JulJclTest.class);

public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-jcl trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("commons-logging-jcl debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("commons-logging-jcl info message");
    }
}
```

结果输出如下：

```
四月 27, 2015 11:13:33 下午 com.demo.log4j.JulJclTest main
信息: commons-logging-jcl info message
```

##### 案例分析

案例过程分析，就是看看上述commons-logging的在执行原理的过程中是如何来走的。

1.  获取LogFactory的过程

    我们没有配置系统属性"org.apache.commons.logging.LogFactory”

   我们没有配置commons-logging的commons-logging.properties配置文件

   也没有含有"META-INF/services/org.apache.commons.logging.LogFactory"路径的jar包

   所以commons-logging会使用默认的LogFactoryImpl作为LogFactory。

2. 根据LogFactory获取Log的过程

   我们没有配置commons-logging的commons-logging.properties配置文件

   我们没有配置系统属性"org.apache.commons.logging.Log”，所以就需要`依次根据classesToDiscover中的类名称进行创建`。

   先是创建`org.apache.commons.logging.impl.Log4JLogger`：创建失败，因为该类是依赖org.apache.log4j包中的类的。

   接着创建`org.apache.commons.logging.impl.Jdk14Logger`：创建成功，所以我们返回的就是Jdk14Logger，看下它是如何与jul集成的，`它内部有一个java.util.logging.Logger logger属性`，所以Jdk14Logger的info(“commons-logging-jcl info message”)操作都会转化成由java.util.logging.Logger来实现，上述logger的来历：

```
logger = java.util.logging.Logger.getLogger(name);
```

就是使用jul原生的方式创建的一个java.util.logging.Logger，是如何打印info信息的呢？使用jul原生的方式：

```
logger.log(Level.WARNING,"commons-logging-jcl info message");
```

由于`jul默认的级别是INFO级别`，所以只打出了如下信息：

```
四月 27, 2015 11:41:24 下午 com.demo.log4j.JulJclTest main
信息: commons-logging-jcl info message
```

原生的jdk的logging的日志级别是**FINEST、FINE、INFO、WARNING、SEVERE分别对应我们常见的trace、debug、info、warn、error。**

#### commons-logging与log4j1集成

对应的maven依赖是：

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

##### 使用案例

在类路径下加入log4j的配置文件log4j.properties

```
log4j.rootLogger = trace, console
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss} %m%n
```

使用方式如下：

```
private static Log logger=LogFactory.getLog(Log4jJclTest.class);
public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-log4j trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("commons-logging-log4j debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("commons-logging-log4j info message");
    }
}
```

代码没变，还是使用commons-logging的接口和类来编程，没有log4j的任何影子。这样，commons-logging就与log4j集成了起来，我们可以通过log4j的配置文件来控制日志的显示级别。上述是trace级别(小于debug)，所以trace、debug、info的都会显示出来。

##### 案例分析

案例过程分析，就是看看上述commons-logging的在执行原理的过程中是如何来走的：

1. 获取LogFactory的过程，同上述jcl的过程一样，`使用默认的LogFactoryImpl作为LogFactory`。

2. 根据LogFactory获取Log的过程，同上述jcl的过程一样，最终会`依次根据classesToDiscover中的类名`称进行创建：

（1）先是创建org.apache.commons.logging.impl.Log4JLogger。

（2）创建成功，因为此时含有log4j的jar包，所以返回的是Log4JLogger，我们看下它与commons-logging是如何集成的：它内部有一个org.apache.log4j.Logger  logger属性，这个是log4j的原生Logger。所以Log4JLogger都是委托这个logger来完成的。

org.apache.log4j.Logger logger来历

```
org.apache.log4j.Logger.getLogger(name)
```

使用原生的log4j1的写法来生成，我们知道上述过程会引发log4j1的配置文件的加载，之后就进入log4j1的世界了。

输出日志 测试案例中我们使用commons-logging输出的日志的形式如下（这里的logger是org.apache.commons.logging.impl.Log4JLogger类型）：

```
logger.debug("commons-logging-log4j debug message");
```

其实就会转换成log4j原生的org.apache.log4j.Logger对象（就是上述获取的org.apache.log4j.Logger类型的logger对象）的如下输出：

```
logger.debug("log4j debug message");
```

上述过程最好与log4j1的原生方式对比着看。

#### commons-logging与log4j2集成

对应的maven依赖是：

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<!-- log4j2的API包 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.2</version>
</dependency>
<!-- log4j2的API实现包 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.2</version>
</dependency>
<!-- log4j2与commons-logging的集成包 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-jcl</artifactId>
    <version>2.2</version>
</dependency>
```

##### 使用案例

编写log4j2的配置文件log4j2.xml，简单如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="debug">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

使用案例如下：

```
private static Log logger=LogFactory.getLog(Log4j2JclTest.class);
public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-log4j trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("commons-logging-log4j debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("commons-logging-log4j info message");
    }
}
```

仍然是使用commons-logging的Log接口和LogFactory来进行编写，看不到log4j2的影子。但是这时候含有上述几个jar包，log4j2就与commons-logging集成了起来。

##### 案例分析

案例过程分析，就是看看上述commons-logging的在执行原理的过程中是如何来走的：

1. 先来看下上述 log4j-jcl（log4j2与commons-logging的集成包）的来历： 我们知道，commons-logging原始的jar包中使用了`默认的LogFactoryImpl作为LogFactory`，该默认的LogFactoryImpl中的`classesToDiscover（到上面查看它的内容）并没有log4j2对应的Log实现类`。所以我们就`不能使用这个原始包中默认的LogFactoryImpl了`，需要重新指定一个，并且需要给出一个apache的Log实现（该Log实现是用于log4j2的），所以就产生了log4j-jcl这个jar包

   里面的LogFactoryImpl就是要准备替换commons-logging中默认的LogFactoryImpl（其中META-INF/services/下的那个文件起到重要的替换作用，下面详细说）

   这里面的Log4jLog便是针对log4j2的，而commons-logging中的原始的Log4JLogger则是针对log4j1的。它们都是commons-logging的Log接口的实现。

2. 获取获取LogFactory的过程 这个过程就和jul、log4j1的集成过程不太一样了。`通过java的SPI机制`，找到了org.apache.commons.logging.LogFactory对应的实现，即在log4j-jcl包中找到的，其中`META-INF/services/org.apache.commons.logging.LogFactory`中的内容是：

```
org.apache.logging.log4j.jcl.LogFactoryImpl
```

​	即指明了使用log4j-jcl中的LogFactoryImpl作为LogFactory。

3. 根据LogFactory获取Log的过程就来看下log4j-jcl中的LogFactoryImpl是怎么实现的：

   ```
   public class LogFactoryImpl extends LogFactory {
   
       private final LoggerAdapter<Log> adapter = new LogAdapter();
       //略
   }
   ```

   这个`LoggerAdapter是lo4j2中的一个适配器接口类`，根据log4j2生产的原生的org.apache.logging.log4j.Logger实例，将它包装成你指定的泛型类。这里使用的LoggerAdapter实现是LogAdapter，它的内容如下：

   ```
   public class LogAdapter extends AbstractLoggerAdapter<Log> {
       @Override
       protected Log newLogger(final String name, final LoggerContext context) {
           return new Log4jLog(context.getLogger(name));
       }
       @Override
       protected LoggerContext getContext() {
           return getContext(ReflectionUtil.getCallerClass(LogFactory.class));
       }
   }
   ```

   我们可以看到，它其实就是将原生的log4j2的Logger封装成Log4jLog。这里就可以看明白了，下面来详细的走下流程，看看是什么时候来初始化log4j2的：

   - 首先获取log4j2中的重要配置对象`LoggerContext`，LogAdapter的实现如上面的源码（使用父类的getContext方法），父类方法的内容如下：

   ```
   LogManager.getContext(cl, false);
   ```

   我们可以看到这其实就是使用log4j2的LogManager进行初始化的，`至此就进入log4j2的初始化的世界了`。

   -  log4j2的LoggerContext初始化完成后，该生产一个log4j2原生的Logger对象，使用log4j2原生的方式：

   ```
   context.getLogger(name)
   ```

   - 将上述方式产生的Log4j原生的Logger实例进行包装，包装成Log4jLog

   ```
   new Log4jLog(context.getLogger(name));
   ```

   至此，我们通过Log4jLog实例打印的日志都是委托给了它内部包含的log4j2的原生Logger对象了。上述过程最好与log4j2的原生方式对比着看

#### commons-logging与logback集成

对应的maven依赖是：

```
<!-- 替代了commons-logging，下面详细说明 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.12</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
<dependency> 
    <groupId>ch.qos.logback</groupId> 
    <artifactId>logback-core</artifactId> 
    <version>1.1.3</version> 
</dependency> 
<dependency> 
    <groupId>ch.qos.logback</groupId> 
    <artifactId>logback-classic</artifactId> 
    <version>1.1.3</version> 
</dependency>
```

##### 使用案例

- 首先在类路径下编写logback的配置文件logback.xml，简单如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>
  <root level="DEBUG">          
    <appender-ref ref="STDOUT" />
  </root>  
</configuration>
```

- 使用方式：

```
private static Log logger=LogFactory.getLog(LogbackTest.class);
public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-jcl trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("commons-logging-jcl debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("commons-logging-jcl info message");
    }
}
```

完全是用commons-logging的API来完成日志编写。

##### 案例分析

`logback本身的使用其实就和slf4j绑定了起来`，现在要想指定commons-logging的底层log实现是logback，则需要2步走:

- 第一步： 先将commons-logging底层的log实现转向slf4j(`jcl-over-slf4j干的事`)
- 第二步： 再根据slf4j的选择底层日志原理，我们使之选择上logback

这样就可以完成commons-logging与logback的集成。即写着commons-logging的API，底层却是logback来进行输出，然后来具体分析下整个过程的源码实现：

- 先看下jcl-over-slf4j都有哪些内容(它可以替代了commons-logging)

  ![4.png](https://i.loli.net/2019/05/18/5cdff5ffcb0ac44575.png)

- commons-logging中的Log接口和LogFactory类等，这是我们使用commons-logging编写需要的接口和类。

- 去掉了commons-logging原生包中的一些Log实现和默认的LogFactoryImpl，`只有SLF4JLog实现和SLF4JLogFactory`。

这就是jcl-over-slf4j的大致内容。这里可以与commons-logging原生包中的内容进行下对比

![5.png](https://i.loli.net/2019/05/18/5cdff5ffef62171354.png)

获取获取LogFactory的过程
jcl-over-slf4j包中的LogFactory和commons-logging中原生的LogFactory不一样，`jcl-over-slf4j中的LogFactory直接限制死，是SLF4JLogFactory`，源码如下：

```
public abstract class LogFactory {
    static LogFactory logFactory = new SLF4JLogFactory();
    //略
}

```

- 根据LogFactory获取Log的过程，这就需要看下jcl-over-slf4j包中的SLF4JLogFactory的源码内容：

```
Log newInstance;
Logger slf4jLogger = LoggerFactory.getLogger(name);
if (slf4jLogger instanceof LocationAwareLogger) {
    newInstance = new SLF4JLocationAwareLog((LocationAwareLogger) slf4jLogger);
} else {
    newInstance = new SLF4JLog(slf4jLogger);
}
```

可以看到其实是用slf4j的LoggerFactory先创建一个slf4j的Logger实例(这其实就是单独使用logback的使用方式）。

然后再将这个`Logger实例封装成common-logging定义的Log接口实现`，即SLF4JLog或者SLF4JLocationAwareLog实例。

所以我们使用的commons-logging的Log接口实例都是委托给slf4j创建的Logger实例（slf4j的这个实例又是选择logbakc后产生的，即slf4j产生的Logger实例最终还是委托给logback中的Logger的）。