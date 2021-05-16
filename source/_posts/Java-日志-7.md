---
title: 日志框架--slf4j
date: 2019-05-13 12:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

**slf4j**:Simple Logging Facade for Java，为java提供的简单日志Facade。`Facade：门面，更底层一点说就是接口。`他允许用户以自己的喜好，在工程中通过slf4j接入不同的日志系统。更直观一点，slf4j是个数据线，一端嵌入程序，另一端链接日志系统，从而实现将程序中的信息导入到日志系统并记录。

[官网](https://www.slf4j.org/)

<!--more-->

因此，`slf4j入口就是众多接口的集合，他不负责具体的日志实现，只在编译时负责寻找合适的日志系统进行绑定。`具体有哪些接口，全部都定义在slf4j-api中。查看slf4j-api源码就可以发现，里面除了public final class LoggerFactory类之外，都是接口定义。因此，slf4j-api本质就是一个接口定义。

Slf4j的 OSGI 机制更好兼容支持各种具体日志实现的框架，一图胜千言，官网上的一个图：

![3.png](https://i.loli.net/2019/05/18/5cdff7582340b95529.png)



#### 使用案例

```
private static Logger logger=LoggerFactory.getLogger(Log4jSlf4JTest.class);
public static void main(String[] args){
    if(logger.isDebugEnabled()){
        logger.debug("slf4j-log4j debug message");
    }
    if(logger.isInfoEnabled()){
        logger.debug("slf4j-log4j info message");
    }
    if(logger.isTraceEnabled()){
        logger.debug("slf4j-log4j trace message");
    }
}
```

上述Logger接口、LoggerFactory类都是slf4j自己定义的。

#### 使用原理

LoggerFactory.getLogger(Log4jSlf4JTest.class)的源码如下：

```
public static Logger getLogger(String name) {
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    return iLoggerFactory.getLogger(name);
}
```

上述获取Log的过程大致分成2个阶段：

- 获取ILoggerFactory的过程 (从字面上理解就是生产Logger的工厂)；
- 根据ILoggerFactory获取Logger的过程；

下面来详细说明：

1. 获取ILoggerFactory的过程，又可以分成3个过程：

   从类路径中`寻找org/slf4j/impl/StaticLoggerBinder.class类`

   ```
   ClassLoader.getSystemResources("org/slf4j/impl/StaticLoggerBinder.class")
   ```

   `如果找到多个，则输出 Class path contains multiple SLF4J bindings，表示有多个日志实现与slf4j进行了绑定。`下面看下当出现多个StaticLoggerBinder的时候的输出日志（简化了一些内容）：

   ```
   SLF4J: Class path contains multiple SLF4J bindings.
   SLF4J: Found binding in [slf4j-log4j12-1.7.12.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: Found binding in [logback-classic-1.1.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: Found binding in [slf4j-jdk14-1.7.12.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
   SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
   ```

   `“随机选取"`一个StaticLoggerBinder.class来创建一个单例

   ```
   StaticLoggerBinder.getSingleton();
   ```

    根据上述创建的StaticLoggerBinder单例，`返回一个ILoggerFactory实例`

   ```
   StaticLoggerBinder.getSingleton().getLoggerFactory();
   ```

   所以slf4j与其他实际的日志框架的集成jar包中，`都会含有这样的一个org/slf4j/impl/StaticLoggerBinder.class类文件`，并且提供一个ILoggerFactory的实现。

2. 根据ILoggerFactory获取Logger的过程：这就要看具体的ILoggerFactory类型了，下面的集成来详细说明。

### slf4j与jdk-logging集成

对应的maven依赖为：

```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.12</version>
</dependency>
```

#### 使用案例

```
private static final Logger logger=LoggerFactory.getLogger(JulSlf4jTest.class);
public static void main(String[] args){
    if(logger.isDebugEnabled()){
        logger.debug("jul debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("jul info message");
    }
    if(logger.isWarnEnabled()){
        logger.warn("jul warn message");
    }
}
```

上述的Logger、LoggerFactory都是slf4j自己的API中的内容，没有jdk自带的logging的踪影，然后打出来的日志却是通过jdk自带的logging来输出的，如下：

```
四月 28, 2015 7:33:20 下午 com.demo.log4j.JulSlf4jTest main
信息: jul info message
四月 28, 2015 7:33:20 下午 com.demo.log4j.JulSlf4jTest main
警告: jul warn message
```

下面梳理下整个流程：

1. 获取ILoggerFactory的过程：由于类路径下有org/slf4j/impl/StaticLoggerBinder.class，所以会选择slf4j-jdk14中的StaticLoggerBinder来创建单例对象并返回ILoggerFactory，来看下StaticLoggerBinder中的ILoggerFactory是什么类型：

```
private StaticLoggerBinder() {
    loggerFactory = new org.slf4j.impl.JDK14LoggerFactory();
}
```

所以返回了JDK14LoggerFactory的实例。

2. 根据ILoggerFactory获取Logger的过程，来看下JDK14LoggerFactory是如何`返回一个slf4j定义的Logger接口的实例`的，源码如下：

```
java.util.logging.Logger julLogger = java.util.logging.Logger.getLogger(name);
Logger newInstance = new JDK14LoggerAdapter(julLogger);
```

- 可以看到，就是使用jdk自带的logging的原生方式来先创建一个jdk自己的java.util.logging.Logger实例
- 然后利用JDK14LoggerAdapter将上述的`java.util.logging.Logger包装成slf4j定义的Logger实例`。
- 所以我们使用slf4j来进行编程，最终会委托给jdk自带的java.util.logging.Logger去执行。

#### slf4j与log4j1集成

下图比较清晰的描述了slf4j与log4j之间的关系：

![4.png](https://i.loli.net/2019/05/18/5cdff7583e62754622.png)

当系统采用log4j作为日志框架实现的调用关系：

（1）首先系统包含slf4j-api作为日志接入的接口。

（2）at compile时slf4j-api中public final class LoggerFactor类中 private  final static void bind()  方法会寻找具体的日志实现类绑定，主要通过StaticLoggerBinder.getSingleton();语句调用。

（3）slf4j-log4j12:`链接slf4j-api和log4j中间的适配器。`它实现了slf4j-api中StaticLoggerBinder接口，从而使得在编译时绑定的是slf4j-log4j12的getSingleton()方法。

（4）log4j:这个是具体的日志系统。通过slf4j-log4j12初始化Log4j，达到最终日志的输出。

maven依赖分别为：

```
<!-- slf4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>

<!-- slf4j-log4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.12</version>
</dependency>

<!-- log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

#### 使用案例

第一步：编写log4j.properties配置文件,放到类路径下

```
log4j.rootLogger = debug, console
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss} %m%n
```

第二步：代码中如下使用

```
private static Logger logger=LoggerFactory.getLogger(Log4jSlf4JTest.class);
public static void main(String[] args){
    if(logger.isDebugEnabled()){
        logger.debug("slf4j-log4j debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("slf4j-log4j info message");
    }
    if(logger.isTraceEnabled()){
        logger.trace("slf4j-log4j trace message");
    }
}
```

补充说明：

1. 配置文件同样可以随意放置，如log4j1原生方式加载配置文件的方式

2. 注意两者方式的不同：

```
slf4j:  Logger logger=LoggerFactory.getLogger(Log4jSlf4JTest.class);
log4j:  Logger logger=Logger.getLogger(Log4jTest.class);
```

slf4j的Logger是slf4j定义的接口，而log4j的Logger是类。LoggerFactory是slf4j自己的类。

#### 原理分析

1. 获取对应的ILoggerFactory：从上面的slf4j的原理中我们知道：ILoggerFactory是由StaticLoggerBinder来创建出来的，所以可以简单分成2个过程：

第一个过程：slf4j寻找绑定类StaticLoggerBinder：使用ClassLoader来加载`“org/slf4j/impl/StaticLoggerBinder.class"`这样的类的url，然后就找到了slf4j-log4j12包中的StaticLoggerBinder。

第二个过程：创建出StaticLoggerBinder实例，`并创建出ILoggerFactory`，源码如下：

```
StaticLoggerBinder.getSingleton().getLoggerFactory();
```

以slf4j-log4j12中的StaticLoggerBinder为例，创建出的ILoggerFactory为Log4jLoggerFactory。

2. 根据ILoggerFactory获取Logger的过程，来看下Log4jLoggerFactory是如何返回一个slf4j定义的Logger接口的实例的，源码如下：

```
org.apache.log4j.Logger log4jLogger;
if (name.equalsIgnoreCase(Logger.ROOT_LOGGER_NAME))
    log4jLogger = LogManager.getRootLogger();
else
    log4jLogger = LogManager.getLogger(name);
Logger newInstance = new Log4jLoggerAdapter(log4jLogger);
```

我们可以看到是通过log4j1的原生方式，即使用log4j1的LogManager来获取，`引发log4j1的加载配置文件，然后初始化`，最后返回一个org.apache.log4j.Logger log4jLogger

将上述的org.apache.log4j.Logger `log4jLogger封装成Log4jLoggerAdapter，而Log4jLoggerAdapter是实现了slf4j的接口`，所以我们使用的slf4j的Logger接口实例（这里即Log4jLoggerAdapter）都会委托给内部的org.apache.log4j.Logger实例。

### slf4j与log4j2集成

对应的maven依赖分别是：

```
<!-- slf4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
<!-- log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.2</version>
</dependency>
<!-- log4j-slf4j-impl （用于log4j2与slf4j集成） -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.2</version>
</dependency>
```

#### 使用案例

第一步：编写log4j2的配置文件log4j2.xml，简单如下：

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

第二步：使用方式

```
private static Logger logger=LoggerFactory.getLogger(Log4j2Slf4jTest.class);
public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("slf4j-log4j2 trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("slf4j-log4j2 debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("slf4j-log4j2 info message");
    }
}
```

#### 案例原理分析

来看下具体过程：

1. 获取对应的ILoggerFactory

第一个过程：slf4j寻找绑定类StaticLoggerBinder：使用ClassLoader来加载 `“org/slf4j/impl/StaticLoggerBinder.class"`这样的类的url，然后就找到了log4j-slf4j-impl包中的StaticLoggerBinder。

第二个过程：创建出StaticLoggerBinder实例，并创建出ILoggerFactory：log4j-slf4j-impl包中的StaticLoggerBinder返回的ILoggerFactory是`Log4jLoggerFactory`。

2. 根据ILoggerFactory获取Logger的过程，来看下Log4jLoggerFactory是如何返回一个slf4j定义的Logger接口的实例的，源码如下：

```
@Override
protected Logger newLogger(final String name, final LoggerContext context) {
    final String key = Logger.ROOT_LOGGER_NAME.equals(name) ? LogManager.ROOT_LOGGER_NAME : name;
    return new Log4jLogger(context.getLogger(key), name);
}
@Override
protected LoggerContext getContext() {
    final Class<?> anchor = ReflectionUtil.getCallerClass(FQCN, PACKAGE);
    return anchor == null ? LogManager.getContext() : getContext(ReflectionUtil.getCallerClass(anchor));
}
```

我们可以看到是通过log4j2的原生方式，即使用log4j2的LoggerContext来获取，返回一个org.apache.logging.log4j.core.Logger即log4j2定义的Logger接口实例

将上述的org.apache.logging.log4j.core.Logger封装成Log4jLogger，而Log4jLogger是实现了slf4j的Logger接口的，所以我们使用的slf4j的Logger接口实例（这里即Log4jLogger）都会委托给内部的log4j2定义的Logger实例。

上述获取LoggerContext的过程也是log4j2的原生方式：

```
LogManager.getContext()
```

该操作会去加载log4j2的配置文件，引发log4j2的初始化。

#### slf4j与logback集成

对应的maven依赖为：

```
<!-- slf4j-api -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
<!-- logback -->
<dependency> 
    <groupId>ch.qos.logback</groupId> 
    <artifactId>logback-core</artifactId> 
    <version>1.1.3</version> 
</dependency>
<!-- logback-classic（已含有对slf4j的集成包） --> 
<dependency> 
    <groupId>ch.qos.logback</groupId> 
    <artifactId>logback-classic</artifactId> 
    <version>1.1.3</version> 
</dependency>
```

#### 使用案例

第一步：编写logback的配置文件logback.xml，简单如下：

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

第二步：使用方式

```
private static final Logger logger=LoggerFactory.getLogger(LogbackTest.class);
public static void main(String[] args){
    if(logger.isDebugEnabled()){
        logger.debug("slf4j-logback debug message");
    }
    if(logger.isInfoEnabled()){
        logger.info("slf4j-logback info message");
    }
    if(logger.isTraceEnabled()){
        logger.trace("slf4j-logback trace message");
    }
}
```

#### 原理分析

logback自己定义的`ch.qos.logback.classic.Logger类`就是实现了slf4j定义的Logger接口

1.  获取对应的ILoggerFactory

第一个过程：slf4j寻找绑定类StaticLoggerBinder，使用ClassLoader来加载 `“org/slf4j/impl/StaticLoggerBinder.class"`这样的类的url，然后就找到了logback-classic包中的StaticLoggerBinder。

第二个过程：创建出StaticLoggerBinder实例，并创建出ILoggerFactory，logback-classic包中的StaticLoggerBinder返回的ILoggerFactory是LoggerContext(logback的对象)；创建出单例后，同时会引发logback的初始化，这时候logback就要去寻找一系列的配置文件，尝试加载并解析。

2. 根据ILoggerFactory获取Logger的过程，来看下LoggerContext(logback的对象)是如何返回一个slf4j定义的Logger接口的实例的：

该LoggerContext(logback的对象)返回的`ch.qos.logback.classic.Logger(logback的原生Logger对象)就是slf4j的Logger实现类`。