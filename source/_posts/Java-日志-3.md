---
title: 日志框架--Log4j2
date: 2019-05-11 12:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

### [Log4j2](http://logging.apache.org/log4j/2.x/)

log4j2与log4j1发生了很大的变化，不兼容。log4j1仅仅作为一个实际的日志框架，slf4j、commons-logging作为门面，统一各种日志框架的混乱格局，现在log4j2也想跳出来充当门面了，也想统一大家了。

<!--more-->

#### Log4j2的特性及改进

- API分离：`Log4j2将API与实现分离开来`。开发人员现在可以很清楚地知道能够使用哪些没有兼容问题的类和方法，同时又允许通过自己实现来增强功能。

- 改进的性能：`Log4j2的性能在某些关键领域比Log4j 1.x更快`，而且大多数情况下与Logback相当。

- 多个API支持：Log4j2提供最棒的性能的同时，还`支持SLF4J和公共日志记录API`。

- 自动配置加载：像Logback一样，一旦配置发生改变，`Log4j2可以自动载入这些更改后的配置信息`，又与Logback不同，配置发生改变时不会丢失任何日志事件。

- 高级过滤功能：与Logback类似，`Log4j2可以支持基于上下文数据、标记，正则表达式以及日志事件中的其他组件的过滤`。Log4j2能够专门指定适用于所有的事件，无论这些事件在传入Loggers之前还是正在传给appenders。另外，过滤器还可以与Loggers关联起来。与Logback不同的是，Filter公共类可以用于任何情况。

- 插件架构：`所有可以配置的组件都以Log4j插件的形式来定义`。同样地，不需要修改任何Log4j代码就可以创建新的Appender、Layout、Pattern Convert 等等。Log4j自动识别预定义的插件，如果在配置中引用到这些插件，Log4j就自动载入使用。

- 属性支持：`属性可以在配置文件中引用，也可以直接替代或传入潜在的组件，属性在这些组件中能够动态解析`。属性可以是配置文件，系统属性，环境变量，线程上下文映射以及事件中的数据中定义的值。用户可以通过增加自己的Lookup插件来定制自己的属性。

- **更为先进的API（Modern API）**在这之前，程序员们以如下方式进行日志记录：

  ```
   if(logger.isDebugEnabled()) {
          logger.debug("Hi, " + u.getA() + “ “ + u.getB());
      }
  
  ```

  许多人都会抱怨上述代码的可读性太差了。`如果有人忘记写if语句，程序输出中会多出很多不必要的字符串`。现在，Java虚拟机（JVM）也许对字符串的打印和输出进行了很多优化，但是难道我们仅仅依靠JVM优化来解决上述问题？log4j 2.0开发团队鉴于以上考虑对API进行了完善。现在你可以这样写代码：

  ```
  logger.debug("Hi, {} {}", u.getA(), u.getB());
  ```

  和其它一些流行的日志框架一样， **新的API也支持变量参数的占位符功能**。

- **log4j 2.0还支持其它一些很棒的功能，像Markers和flow tracing**：

```
    private Logger logger = LogManager.getLogger(MyApp.class.getName());
    private static final Marker QUERY_MARKER = MarkerManager.getMarker("SQL");
    ...
    public String doQuery(String table) {
        logger.entry(param);
        logger.debug(QUERY_MARKER, "SELECT * FROM {}", table);
        return logger.exit();
    }
```

`Markers可以帮助你很快地找到具体的日志项（Log Entries）`。而在某个方法的开头和结尾调用Flow Traces中的一些方法，你可以在日志文件中看到很多新的跟踪层次的日志项，也就是说，`你的程序工作流（Program Flow）被记录下来了`。下面是Flow Traces的一些例子：

```
19:08:07.056 TRACE com.test.TestService 19 retrieveMessage - entry
19:08:07.060 TRACE com.test.TestService 46 getKey - entry
```

- **插件式的架构**：`log4j 2.0支持插件式的架构`。你可以根据需要自行扩展log4j 2.0，这非常简单。首先，你要为你的扩展建立好命名空间，然后告诉log4j 2.0在哪能够找到它。

```
<configuration … packages="de.grobmeier.examples.log4j2.plugins">
```

根据上述配置，`log4j 2将会在de.grobmeier.examples.log4j2.plugins包中找寻你的扩展插件`。如果你建立了多个命名空间，没关系，用逗号分隔就可以了。

下面是一个简单的扩展插件：

```
    @Plugin(name = "Sandbox", type = "Core", elementType = "appender")
    public class SandboxAppender extends AppenderBase {

        private SandboxAppender(String name, Filter filter) {
            super(name, filter, null);
        }
 
        public void append(LogEvent event) {
            System.out.println(event.getMessage().getFormattedMessage());
        }
 
        @PluginFactory
        public static SandboxAppender createAppender(
             @PluginAttr("name") String name,
             @PluginElement("filters") Filter filter) {
            return new SandboxAppender(name, filter);
        }
    }
```

上面标有`@PluginFactory注解的方法是一个工厂`，它的两个参数直接从配置文件读取。我用@PluginAttr和@PluginElement进行了实现。

剩下的就非常简单了。**由于我写的是一个Appender，因此得继承AppenderBase这个类。该类必须实现append()方法，从而进行实际的逻辑处理。除了Appender，你甚至可以实现自己的Logger和Filter**。

- **强大的配置功能（Powerful Configuration）** log4j 2的配置变得非常简单。如果你习惯了之前的配置方式，也不用担心，你只要花很少的时间就可以从之前的方式转换到新的方式。请看下面的配置：

```
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration status="OFF">
        <appenders>
            <Console name="Console" target="SYSTEM_OUT">
                <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
            </Console>
        </appenders>
        <loggers>
            <logger name="com.foo.Bar" level="trace" additivity="false">
                <appender-ref ref="Console"/>
            </logger>
            <root level="error">
                <appender-ref ref="Console"/>
            </root>
        </loggers>
    </configuration>
```

上面说的只是一部分改进，**你还可以自动重新加载配置文件**：

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration monitorInterval="30">
    ...
</configuration>
```

监控的时间间隔单位为秒，最小值是5。这意味着，log4j 2在配置改变的情况下可以重新配置日志记录行为。如果值设置为0或负数，log4j 2不会对配置变更进行监测。`最为称道的一点是：不像其它日志框架， log4j 2.0在重新配置的时候不会丢失之前的日志记录`。

还有一个非常不错的改进，那就是：**同XML相比，如果你更加喜欢JSON，你可以自由地进行基于JSON的配置了**：

```
{
    "configuration": {
        "appenders": {
            "Console": {
                "name": "STDOUT",
                "PatternLayout": {
                    "pattern": "%m%n"
                }
            }
        },
        "loggers": {
            "logger": {
                "name": "EventLogger",
                "level": "info",
                "additivity": "false",
                "appender-ref": {
                    "ref": "Routing"
                }
            },
            "root": {
                "level": "error",
                "appender-ref": {
                    "ref": "STDOUT"
                }
            }
        }
    }
}
```

- **Java 5 并发性（Concurrency）**

有一段文档是这样描述的：`“log4j 2利用Java 5中的并发特性支持，尽可能地执行最低层次的加锁...”`。Apache log4j 2.0解决了许多在log4j 1.x中仍然存留的死锁问题。如果你的程序仍然饱受内存泄漏的折磨，请毫不犹豫地试一下log4j 2.0。



#### 简单使用

log4j2分成2个部分：

- log4j-api： 作为日志接口层，用于统一底层日志系统
- log4j-core : 作为上述日志接口的实现，是一个实际的日志框架

需要使用的Maven依赖：

```
    <dependencies>
      <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.11.2</version>
      </dependency>
      <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.11.2</version>
      </dependency>
    </dependencies>
```

使用方式：

- 第一步：编写log4j2.xml配置文件（目前log4j2只支持xml json yuml，不再支持properties文件）

```xml
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

- 第二步： 使用方式

```
private static final Logger logger=LogManager.getLogger(Log4j2Test.class);
public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.debug("log4j trace message");
    }
    if(logger.isDebugEnabled()){
        logger.debug("log4j debug message");
    }
    if(logger.isInfoEnabled()){
        logger.debug("log4j info message");
    }
}
```

和log4j1是不同的。此时Logger是log4j-api中定义的接口，而log4j1中的Logger则是类。

#### 使用过程简单分析

1. 获取底层使用的`LoggerContextFactory`：同样LogManager的类加载会去寻找log4j-api定义的LoggerContextFactory接口的底层实现，获取方式有三种：

- 第一种： `尝试从jar中寻找log4j2.component.properties文件`，如果配置了log4j2.loggerContextFactory则使用该LoggerContextFactory

- 第二种：如果没找到，尝试从jar包中寻找`META-INF/log4j-provider.properties`文件，如log4j-core-2.2中就有该文件

  如果找到多个，`取优先级最高的`（该文件中指定了LoggerContextFactory，同时指定了优先级FactoryPriority），如log4j-core-2.2中log4j-provider.properties的文件内容如下：

 ```
  LoggerContextFactory = org.apache.logging.log4j.core.impl.Log4jContextFactory
  Log4jAPIVersion = 2.1.0
  FactoryPriority= 10
 ```

- 第三种情况：上述方式还没找到，就使用`默认的SimpleLoggerContextFactory`

2. 使用LoggerContextFactory获取`LoggerContext`

3. 根据LoggerContext获取`Logger`，以log4j-core为例：会首先判断LoggerContext是否被初始化过了，没有则进行初始化

4. 获取ConfigurationFactory,从配置中获取和插件中获取（log4j-core核心包中有三个YamlConfigurationFactory、 JsonConfigurationFactory、XmlConfigurationFactory）

以上文的案例中，会使用XmlConfigurationFactory来加载log4j2.xml配置文件，`LoggerContext初始化后，就可以获取或者创建Logger了`

#### 主要对象总结

- LogManager： 它的类加载会去寻找LoggerContextFactory接口的底层实现，会从jar包中的配置文件中寻找，如上面所述
- LoggerContextFactory ： 用于创建LoggerContext，不同的日志实现系统会有不同的实现，如log4j-core中的实现为Log4jContextFactory
- PropertyConfigurator: 用于解析log4j.properties文件
- LoggerContext : 它包含了配置信息，并能创建log4j-api定义的Logger接口实例，并缓存这些实例
- ConfigurationFactory：上述LoggerContext解析配置文件，需要用到ConfigurationFactory，目前有三个YamlConfigurationFactory、JsonConfigurationFactory、XmlConfigurationFactory，分别解析yuml  json xml形式的配置文件

### Log4j2日志级别

在log4j2中，一共有五种log level，分别为TRACE, DEBUG,INFO, WARN, ERROR 以及FATAL。详细描述如下：

- FATAL：用在极端的情形中，即`必须马上获得注意的情况`。这个程度的错误通常需要触发运维工程师的寻呼机。
- ERROR：显示一个错误，或一个通用的错误情况，但还`不至于会将系统挂起`。这种`程度的错误一般会触发邮件的发送`，将消息发送到alert list中，运维人员可以在文档中记录这个bug并提交。
- WARN：不一定是一个bug，但是有人可能会想要知道这一情况。`如果有人在读log文件，他们通常会希望读到系统出现的任何警告`。
- INFO：用于`基本的、高层次`的诊断信息。在长时间运行的代码段开始运行及结束运行时应该产生消息，以便知道现在系统在干什么。但是这样的信息不宜太过频繁。
- DEBUG：用于`协助低层次的调试`。
- TRACE：用于`展现程序执行的轨迹`。

### 概念介绍

#### Logger层次关系

相比于纯粹的System.out.println方式，使用logging API的最首要以及最重要的优势是可以`在禁用一些log语句块的同时允许其他的语句块的输出`。这一能力建立在一种假设之上，即所有在应用中可能出现的logging语句可以按照开发者定义的标准分成不同的类型。

在Log4j 1.x版本时，**Logger的层次是靠Logger类之间的关系来维护的**。但在Log4j2中， **Logger的层次则是靠LoggerConfig对象之间的关系来维护的**。

`Logger和LoggerConfig均是有名称的实体`。Logger的命名是大小写敏感的，并且服从如下的分层命名规则。（与java包的层级关系类似）。例如：com.foo是com.foo.Bar的父级；java是java.util的父级，是java.util.vector的祖先。

`root LoggerConfig位于LoggerConfig层级关系的最顶层`。它将永远存在与任何LoggerConfig层次中。任何一个希望与root LoggerConfig相关联的Logger可以通过如下方式获得：

```
Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);
```

其他的Logger实例可以调用LogManager.getLogger 静态方法并传入想要得到的Logger的名称来获得。

#### LoggerContext

LoggerContext在Logging System中扮演了锚点的角色。根据情况的不同，一个应用可能同时存在于多个有效的LoggerContext中。在同一LoggerContext下，log system是互通的。如：Standalone Application、Web Applications、Java EE Applications、"Shared" Web Applications 和REST Service Containers，就是不同广度范围的log上下文环境。

####  Configuration

 每一个LoggerContext都有一个有效的Configuration。Configuration包含了所有的Appenders、上下文范围内的过滤器、LoggerConfigs以及StrSubstitutor.的引用。在重配置期间，新与旧的Configuration将同时存在。当所有的Logger对象都被重定向到新的Configuration对象后，旧的Configuration对象将被停用和丢弃。

#### Logger

 如前面所述， Loggers 是通过调用LogManager.getLogger方法获得的。`Logger对象本身并不实行任何实际的动作。它只是拥有一个name 以及与一个LoggerConfig相关联`。它继承了AbstractLogger类并实现了所需的方法。`当Configuration改变时，Logger将会与另外的LoggerConfig相关联，从而改变这个Logger的行为`。

获得Logger,使用相同的名称参数来调用getLogger方法将获得来自同一个Logger的引用。如：

```
Logger x = Logger.getLogger("wombat");

Logger y = Logger.getLogger("wombat"); // x和y指向的是同一个Logger对象。
```

**log4j环境的配置是在应用的启动阶段完成的。优先进行的方式是通过读取配置文件来完成**。

`log4j使采用类名（包括完整路径）来定义Logger 名变得很容易`。这是一个很有用且很直接的Logger命名方式。使用这种方式命名可以很容易的定位这个log message产生的类的位置。当然，log4j也支持任意string的命名方式以满足开发者的需要。不过，`使用类名来定义Logger名仍然是最为推崇的一种Logger命名方式`。

#### LoggerConfig

`当Logger在configuration中被描述时，LoggerConfig对象将被创建`。`LoggerConfig包含了一组过滤器`。LogEvent在被传往Appender之前将先经过这些过滤器。`过滤器中包含了一组Appender的引用`。`Appender则是用来处理这些LogEvent的`。

`每一个LoggerConfig会被指定一个Log级别`。可用的Log级别包括TRACE, DEBUG,INFO, WARN, ERROR 以及FATAL。需要注意的是，在log4j2中，Log的级别是一个Enum型变量，是不能继承或者修改的。`如果希望获得更多的分割粒度，可用考虑使用Markers来替代`。

在Log4j 1.x 和Logback 中都有“层次继承”这么个概念。但是`在log4j2中，由于Logger和LoggerConfig是两种不同的对象，因此“层次继承”的概念实现起来跟Log4j 1.x 和Logback不同`。具体情况下面的五个例子：

1. 可用看到，应用中的LoggerConfig只有root这一种。因此，对于所有的Logger而言，都只能与该LoggerConfig相关联而没有别的选择。

   ![1.jpg](https://i.loli.net/2019/05/18/5cdfe9c4e262833285.jpg)

   

2. 在例子二中可以看到，有5种不同的LoggerConfig存在于应用中，而每一个Logger都被与最匹配的LoggerConfig相关联着，并且拥有不同的Log Level。

   ![2.jpg](https://i.loli.net/2019/05/18/5cdfe9ca61e8073192.jpg)

3. 可以看到Logger root、X、X.Y.Z都找到了与各种名称相同的LoggerConfig。而LoggerX.Y没有与其名称相完全相同的LoggerConfig。怎么办呢？它最后选择了X作为它的LoggerConfig，`因为X LoggerConfig拥有与其最长的匹配度`。

   ![3.jpg](https://i.loli.net/2019/05/18/5cdfe9d36808456372.jpg)

4. 可以看到，现在应用中有两个配置好的LoggerConfig：root和X。而Logger有四个：root、X、X.Y、X.Y.Z。其中，root和X都能找到完全匹配的LoggerConfig，而X.Y和X.Y.Z则没有完全匹配的LoggerConfig，那么它们将选择哪个LoggerConfig作为自己的LoggerConfig呢？由图上可知，它们都选择了X而不是root作为自己的LoggerConfig，`因为在名称上，X拥有最长的匹配度`。

   ![4.jpg](https://i.loli.net/2019/05/18/5cdfe9d388c2d57319.jpg)

5. 可以看到，现在应用中有三个配置好的LoggerConfig，分别为：root、X、X.Y。同时，有四个Logger，分别为：root、X、X.Y以及X.YZ。其中，名字能完全匹配的是root、X、X.Y。那么剩下的X.YZ应该匹配X还是匹配X.Y呢？答案是X。`因为匹配是按照标记点（即“.”）来进行的，只有两个标记点之间的字串完全匹配才算，否则将取上一段完全匹配的字串的长度作为最终匹配长度`。

   ![5.jpg](https://i.loli.net/2019/05/18/5cdfe9d38b61b95592.jpg)

#### Filter

与防火墙过滤的规则相似，`log4j2的过滤器也将返回三类状态：Accept（接受）, Deny（拒绝） 或Neutral（中立）`。其中，Accept意味着不用再调用其他过滤器了，这个LogEvent将被执行；Deny意味着马上忽略这个event，并将此event的控制权交还给过滤器的调用者；Neutral则意味着这个event应该传递给别的过滤器，如果再没有别的过滤器可以传递了，那么就由现在这个过滤器来处理。

#### Appender

由logger的不同来决定一个logging request是被禁用还是启用只是log4j2的情景之一。log4j2还允许将logging   request中log信息打印到不同的目的地中。在log4j2的世界里，不同的输出位置被称为Appender。目前，Appender可以是console、文件、远程socket服务器、Apache  Flume、JMS以及远程 UNIX 系统日志守护进程。`一个Logger可以绑定多个不同的Appender`。

`可以调用当前Configuration的addLoggerAppender函数来为一个Logger增加`。如果不存在一个与Logger名称相对应的LoggerConfig，那么相应的LoggerConfig将被创建，并且新增加的Appender将被添加到此新建的LoggerConfig中。尔后，所有的Loggers将会被通知更新自己的LoggerConfig引用（PS：`一个Logger的LoggerConfig引用是根据名称的匹配长度来决定的，当新的LoggerConfig被创建后，会引发一轮配对洗牌`）。

在某一个Logger中被启用的logging request将被转发到该Logger相关联的的所有Appenders上，`并且还会被转发到LoggerConfig的父级的Appenders上`。

这样会产生一连串的遗传效应。例如，对LoggerConfig  B来说，它的父级为A，A的父级为root。如果在root中定义了一个Appender为console，那么所有启用了的logging  request都会在console中打印出来。另外，如果LoggerConfig  A定义了一个文件作为Appender，那么使用LoggerConfig A和LoggerConfig B的logger 的logging  request都会在该文件中打印，并且同时在console中打印。

如果想避免这种遗传效应的话，可以在configuration文件中做如下设置：

```
additivity="false" // 默认为true
```

这样，就可以关闭Appender的遗传效应了。

#### Layout

通常，用户不止希望能定义log输出的位置，还希望可以定义输出的格式。这就可以`通过将Appender与一个layout相关联来实现`。Log4j中定义了一种类似C语言printf函数的打印格式，如"%r [%t] %-5p %c - %m%n" 格式在真实环境下会打印类似如下的信息：

```
176 [main] INFO  org.foo.Bar - Located nearest gas station.
```

其中，各个字段的含义分别是：

```
%r 指的是程序运行至输出这句话所经过的时间（以毫秒为单位）；
%t 指的是发起这一log request的线程；
%c 指的是log的level；
%m 指的是log request语句携带的message；
%n 为换行符；
```

### Log4j2的LogEvent分析

#### 如何产生LogEvent

调用Logger对象的info、error、trace等函数时，就会产生LogEvent。LogEvent跟LoggerConfig一样，也是由Level的。LogEvent的Level主要是用在Event传递时，判断在哪里停下。

```
    import org.apache.logging.log4j.LogManager;
    import org.apache.logging.log4j.Logger;
    public class Test {
        private static Logger logger = LogManager.getLogger("HelloWorld");
        public static void main(String[] args){
            Test.logger.info("hello,world");
            Test.logger.error("There is a error here");
        }

    }
```

如代码中所示，这样就产生了两个LogEvent。

#### LogEvent的传递是怎样的

我们在IDE中运行一下这个程序，看看会有什么输出。发现，只有ERROR的语句输出了，那么INFO的语句呢？

不着急，先来看看工程目录结构,可以看到，工程中没有写入任何的配置文件。所以，application应该是使用了默认的LoggerConfig Level。那么默认的Level是多少呢？`默认的输出地是console，默认的级别是ERROR级别`。

那么，为什么默认ERROR级别会导致INFO级别的信息被拦截呢？看如下表格：

![7.jpg](https://i.loli.net/2019/05/18/5cdfeaafbe1ba11998.jpg)

左边竖栏是Event的Level，右边横栏是LoggerConfig的Level。Yes的意思就是这个event可以通过filter，no的意思就是不能通过filter。

可以看到，INFO级别的Event是无法被ERROR级别的LoggerConfig的filter接受的。所以，INFO信息不会被输出。

### Log4j2的配置文件重定位

如果想要改变默认的配置，那么就需要configuration file。Log4j的配置是写在log4j.properties文件里面，但是`Log4j2就只能写在XML和JSON文件里了`。

**（1）放在classpath（src）下，以log4j2.xml命名**：使用Log4j2的一般都约定俗成的写一个log4j2.xml放在src目录下使用。这一点没有争议。

**（2）将配置文件放到别处**：在系统工程里面，将log4j2的配置文件放到src目录底下很不方便。如果能把工程中用到的所有配置文件都放在一个文件夹里面，当然就更整齐更好管理了。但是想要实现这一点，前提就是Log4j2的配置文件能重新定位到别处去，而不是放在classpath底下。

如果没有设置"log4j.configurationFile" system property的话，`application将在classpath中按照如下查找顺序来找配置文件`：

```
   log4j2-test.json 或log4j2-test.jsn文件

　　log4j2-test.xml文件

　　log4j2.json 或log4j2.jsn文件

　　log4j2.xml文件
```

这就是为什么在src目录底下放log4j2.xml文件可以被识别的原因了。`如果想将配置文件重命名并放到别处，就需要设置系统属性log4j.configurationFile`。设置的方式是在VM arguments中写入该属性的key和value：

```
-Dlog4j.configurationFile="D:\learning\blog\20130115\config\LogConfig.xml"
```

测试的java程序如上文，在此不再重复。

### 默认配置

本来以为Log4J2应该有一个默认的配置文件的，不过好像没有找到（通过DefaultConfiguration，初始化一个最小化配置），下面这个配置文件等同于缺省配置：

```
    <?xml version="1.0" encoding="UTF-8"?>  
    <configuration status="OFF">  
        <appenders>  
            <Console name="Console" target="SYSTEM_OUT">  
                <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>  
            </Console>  
        </appenders>  
        <loggers>  
            <root level="error">  
                <appender-ref ref="Console"/>  
            </root>  
        </loggers>  
    </configuration>
```

### 第一个配置例子

配置Log4j 2可以有四种方法(其中任何一种都可以):

1. 通过一个格式为`XML或JSON`的配置文件。
2. 以编程方式,通过创建一个`ConfigurationFactory工厂和Configuration实现`。
3. 以编程方式,通过调用`api暴露`在配置界面添加组件的默认配置。
4. 以编程方式,通过调用`Logger内部类`上的方法。

注意,与Log4j 1.x不一样的地方,公开的Log4j 2 API没有提供任何添加、修改或删除 appender和过滤器或者操作配置文件的方法。

`Log4j能够自动配置本身在初始化期间`。当Log4j启动它将定位所有的ConfigurationFactory插件和安排然后在加权从最高到最低。`Log4j包含两个ConfigurationFactory实现,一个用于JSON和XML`。加载配置文件流程如下：

1. Log4j将检查“Log4j的配置文件“系统属性,如果设置,将`尝试加载配置使用 ConfigurationFactory 匹配的文件扩展`。
2. 如果`没有系统属性设置JSON ConfigurationFactory log4j2-test将寻找`. json或 log4j2-test.json在类路径中。
3. 如果`没有这样的文件发现XML ConfigurationFactory log4j2-test将寻找`. xml在 类路径。
4. 如果`一个测试文件无法找到JSON ConfigurationFactory log4j2将寻找`. log4j2.json或 在类路径中。
5. 如果`一个JSON文件无法找到XML ConfigurationFactory将试图定位 log4j2`。 xml在类路径中。
6. 如果`没有配置文件可以找到了 DefaultConfiguration 将被使用`。 这将导致日志输出到控制台。

```
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration status="OFF">
        <appenders>
            <Console name="Console" target="SYSTEM_OUT">
                <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
            </Console>
        </appenders>
        <loggers>
            <!--我们只让这个logger输出trace信息，其他的都是error级别-->
            <!--
            additivity开启的话，由于这个logger也是满足root的，所以会被打印两遍。
            不过root logger 的level是error，为什么Bar 里面的trace信息也被打印两遍呢
            -->
            <logger name="cn.lsw.base.log4j2.Hello" level="trace" additivity="false">
                <appender-ref ref="Console"/>
            </logger>
            <root level="error">
                <appender-ref ref="Console"/>
            </root>
        </loggers>
    </configuration>
```

我们这里看到了配置文件里面是name很重要，没错，这个name可不能随便起（其实可以随便起）。这个机制意思很简单。就是类似于java  package一样，比如我们的一个包：cn.lsw.base.log4j2。而且，可以发现我们前面生成Logger对象的时候，命名都是通过  Hello.class.getName(); 这样的方法，为什么要这样呢？

 很简单，因为有所谓的Logger 继承的问题。比如  如果你给cn.lsw.base定义了一个logger，那么他也适用于cn.lsw.base.lgo4j2这个logger。`名称的继承是通过点（.）分隔的`。然后你可以猜测上面loggers里面有一个子节点不是logger而是root，而且这个root没有name属性。这个root相当于根节点。你所有的logger都适用与这个logger，所以，即使你在很多类里面通过类名.class.getName()   得到很多的logger，而且没有在配置文件的loggers下面做配置，他们也都能够输出，`因为他们都继承了root的log配置`。

我们上面的这个配置文件里面还定义了一个logger，他的名称是 cn.lsw.base.log4j2.Hello  ，这个名称其实就是通过前面的Hello.class.getName();  得到的，我们为了给他单独做配置，这里就生成对于这个类的logger，上面的配置基本的意思是只有cn.lsw.base.log4j2.Hello  这个logger输出trace信息，也就是他的日志级别是trace，其他的logger则继承root的日志配置，日志级别是error，只能打印出ERROR及以上级别的日志。

如果这里logger  的name属性改成cn.lsw.base，则这个包下面的所有logger都会继承这个log配置（这里的包是log4j的logger  name的“包”的含义，不是java的包，你非要给Hello生成一个名称为“myhello”的logger，他也就没法继承cn.lsw.base这个配置了。

那有人就要问了，他不是也应该继承了root的配置了么，那么会不会输出两遍呢？我们在配置文件中给了解释，`如果你设置了additivity="false"，就不会输出两遍`。

### 复杂一点的配置

```
   <?xml version="1.0" encoding="UTF-8"?>
    <!--
        Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出。 
    -->
    <!--
        monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数。
    -->
    <configuration status="error" monitorInterval=”30″>
        <!--先定义所有的appender-->
        <appenders>
            <!--这个输出控制台的配置-->
            <Console name="Console" target="SYSTEM_OUT">
                <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
                <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
                <!--这个都知道是输出日志的格式-->
                <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
            </Console>
            <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用-->
            <File name="log" fileName="log/test.log" append="false">
                <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
            </File> 
            <!-- 这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
            <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
                <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
                <SizeBasedTriggeringPolicy size="50MB"/>
                <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
                <DefaultRolloverStrategy max="20"/>
            </RollingFile>
        </appenders>
        <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
        <loggers>
            <!--建立一个默认的root的logger-->
            <root level="trace">
                <appender-ref ref="RollingFile"/>
                <appender-ref ref="Console"/>
            </root> 
        </loggers>
    </configuration> 

```

**扩展组件**

1. ConsoleAppender：输出结果到System.out或是System.err。
2. FileAppender：输出结果到指定文件，同时可以指定输出数据的格式。append=“false”指定不追加到文件末尾
3. RollingFileAppender：自动追加日志信息到文件中，直至文件达到预定的大小，然后自动重新生成另外一个文件来记录之后的日志。

**过滤标签**

1. ThresholdFilter：用来过滤指定优先级的事件。
2. TimeFilter：设置start和end，来指定接收日志信息的时间区间。

#### Appender之Syslog配置

log4j2中对syslog的简单配置，这里就不重复展示log4j2.xml了：

```
<Syslog name="SYSLOG" host="localhost" port="514" protocol="UDP" facility="LOCAL3"/>
```

host是指你将要把日志写到的目标机器，可以是ip（本地ip或远程ip，远程ip在实际项目中很常见，有专门的日志服务器来存储日志），也可以使用主机名，如果是本地，还可以使用localhost或127.0.0.1。

Port指定端口，默认514，参见/etc/rsyslog.conf（以Fedora系统为例，下同）。protocol指定传输协议，这里是UDP，`facility是可选项`，后面可以看到用法。

**Syslog及Syslog-ng相关配置（Fedora）**

运行程序之前，需要修改：`/etc/rsyslog.conf`。

把这两行前的#去掉，即取消注释：

```
#$ModLoad imudp
#$UDPServerRun 514
```

这里启用udp监听，514是默认监听端口，重启syslog：

```
service syslog restart
```

大部分日志会默认写到/var/log/messages中，如果不想写到这个文件里，可以按下面修改，这样local3的日志就会写到app.log中。这里的local3即 log4j2.xml中facility的配置。

```
*.info;mail.none;authpriv.none;cron.none;local3.none                /var/log/messages
```

新增一行：

```
local3.*                                                            /var/log/app.log
```

除了使用自带的syslog，我们`也可以使用syslog的替代品，比如syslog-ng`，这对于log4j2.xml配置没有影响。 安装：

```
yum install syslog-ng
```

启动：

```
service syslog-ng start
```

其配置文件为：

```
/etc/syslog-ng/syslog-ng.conf
```

启动前把source一节中这一行取消注释即可：

```
#udp(ip(0.0.0.0) port(514));
```

这个端口会和syslog冲突，可以使用别的端口比如50014，同时修改log4j2.xml中的port属性。另外提一下，使用非默认端口，要求log4j版本在1.2.15或以上。

syslog-ng本身也可以设置把日志送到远程机器上，在源机器上的syslog-ng.conf中添加：

```
destination d_remote1 {udp(153.65.171.73 port(514));};
```

这表示源机器上的syslog-ng会把接收到的日志送到远程主机153.65.171.73的514端口上，只要保证153.65.171.73上的syslog-ng正常运行并监听对应的端口即可。

### Log4j2与Spring集成

web.xml配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/root-context.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- log4j2-begin -->
    <listener>
        <listener-class>org.apache.logging.log4j.web.Log4jServletContextListener</listener-class>
    </listener>
    <filter>
        <filter-name>log4jServletFilter</filter-name>
        <filter-class>org.apache.logging.log4j.web.Log4jServletFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>log4jServletFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>FORWARD</dispatcher>
        <dispatcher>INCLUDE</dispatcher>
        <dispatcher>ERROR</dispatcher>
    </filter-mapping>
    <!-- log4j2-end -->
    
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```