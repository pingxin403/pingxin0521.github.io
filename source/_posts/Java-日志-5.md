---
title: 日志框架--logback
date: 2019-05-11 22:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

### logback

Logback是由log4j创始人设计的一个开源日志组件。LogBack被分为3个组件，logback-core, logback-classic 和 logback-access。

[官网](https://logback.qos.ch/)

[中文文档](http://www.logback.cn/)

<!--more-->

1. logback-core:提供了LogBack的核心功能，是另外两个组件的基础。
2. logback-classic:实现了Slf4j的API，所以当想配合Slf4j使用时，需要引入logback-classic。
3. logback-access:为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。

实际上，这两个日志框架都出自同一个开发者之手，但Logback 相对于 Log4J 有更多的优点

- 同样的代码路径，Logback 执行更快
- 更充分的测试
- 原生实现了 SLF4J API（Log4J 还需要有一个中间转换层）
- 内容更丰富的文档
- 支持 XML 或者 Groovy 方式配置
- 配置文件自动热加载
- 从 IO 错误中优雅恢复
- 自动删除日志归档
- 自动压缩日志成为归档文件
- 支持 Prudent 模式，使多个 JVM 进程能记录同一个日志文件
- 支持配置文件中加入条件判断来适应不同的环境
- 更强大的过滤器
- 支持 SiftingAppender（可筛选 Appender）
- 异常栈信息带有包信息

具体参考[这里](https://my.oschina.net/xianggao/blog/522467)。

logback 提供的配置方式有以下几种：

- 编程式配置
- xml 格式
- groovy 格式

Maven依赖

```
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
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.12</version>
</dependency>
```

使用方式

```
private static final Logger logger=LoggerFactory.getLogger(LogbackTest.class);

public static void main(String[] args){
    if(logger.isDebugEnabled()){
        logger.debug("slf4j-logback debug message");
    }
    if(logger.isInfoEnabled()){
        logger.debug("slf4j-logback info message");
    }
    if(logger.isTraceEnabled()){
        logger.debug("slf4j-logback trace message");
    }

    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);
}
```



补充：

- 官方使用方式，其实就和slf4j集成了起来：上述的`Logger、LoggerFactory都是slf4j自己的接口与类`。

- `没有配置文件的情况下，使用的是默认配置`。搜寻配置文件的过程如下：

  > Logback尝试在类路径中查找名为logback.groovy的文件。
  >
  > 如果未找到此类文件，则logback会尝试在类路径中查找名为logback-test.xml的文件。
  >
  > 如果找不到这样的文件，它会检查类路径中的文件logback.xml。
  >
  > 如果未找到此类文件，并且正在执行的JVM具有ServiceLoader（JDK  6及更高版本），则将使用ServiceLoader来解析com.qos.logback.classic.spi.Configurator的实现。  将使用找到的第一个实现。 有关更多详细信息，请参阅ServiceLoader文档。
  >
  >  如果以上都不成功，则logback将使用BasicConfigurator自动配置自身，这将导致日志记录输出定向到控制台。
  >
  >  第四步也是最后一步是为了在没有配置文件的情况下提供默认（但非常基本）的日志记录功能。



也可以在类路径下加上一个类似如下的logback.xml的配置文件，如下：

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

#### 使用过程简单分析

1. slf4j与底层的日志系统进行绑定，在jar包中寻找`org/slf4j/impl/StaticLoggerBinder.class`这个类，如果找到`多个StaticLoggerBinder`，则表明目前底层有多个实际的日志框架，`slf4j会随机选择一个`。
2. 使用上述找到的StaticLoggerBinder创建一个实例，并返回一个ILoggerFactory实例：

```
return StaticLoggerBinder.getSingleton().getLoggerFactory()；
```

以logback-classic中的StaticLoggerBinder为例，在StaticLoggerBinder.getSingleton()过程中：会去加载解析配置文件 源码如下：

```
public URL findURLOfDefaultConfigurationFile(boolean updateStatus) {
    ClassLoader myClassLoader = Loader.getClassLoaderOfObject(this);
    //寻找logback.configurationFile的系统属性
    URL url = findConfigFileURLFromSystemProperties(myClassLoader, updateStatus);
    if (url != null) {
      return url;
    }
    //寻找logback.groovy
    url = getResource(GROOVY_AUTOCONFIG_FILE, myClassLoader, updateStatus);
    if (url != null) {
      return url;
    }
    //寻找logback-test.xml
    url = getResource(TEST_AUTOCONFIG_FILE, myClassLoader, updateStatus);
    if (url != null) {
      return url;
    }
    //寻找logback.xml
    return getResource(AUTOCONFIG_FILE, myClassLoader, updateStatus);
}
```

目前路径都是定死的，只有`logback.configurationFile`的系统属性是可以更改的，所以如果我们想更改配置文件的位置（不想放在类路径下），则需要设置这个系统属性：

```
System.setProperty("logback.configurationFile", "/path/to/config.xml");
```

解析完配置文件后，返回的`ILoggerFactory实例的类型是LoggerContext`（它包含了配置信息）。

- 根据返回的ILoggerFactory实例，来获取Logger，就是根据上述的`LoggerContext来创建一个Logger`，每个logger与LoggerContext建立了关系，并放到LoggerContext的缓存中，就是LoggerContext的如下属性：

```
private Map<String, Logger> loggerCache;
```

其实上述过程就是slf4j与其他日志系统的绑定过程。`不同的日志系统与slf4j集成，都会有一个StaticLoggerBinder类，并会拥有一个ILoggerFactory的实现。`

###  深入分析

本文章用到的组件如下：请自行到Maven仓库下载！

```
logback-access-1.0.0.jar
logback-classic-1.0.0.jar
logback-core-1.0.0.jar
slf4j-api-1.6.0.jar
```

maven配置： 这样依赖包全部自动下载了！

```
    <dependency>  
        <groupId>ch.qos.logback</groupId>  
        <artifactId>logback-classic</artifactId>  
        <version>1.0.11</version>  
    </dependency>
```

#### Logback配置

Logback建立于三个主要类之上：`Logger、Appender 和 Layout`。`Logger类是logback-classic模块的一部分`，`而Appender和Layout接口来自logback-core`。作为一个多用途模块，logback-core不包含任何logger。

Logger作为`日志的记录器，把它关联到应用的对应的context上后`，主要用于存放日志对象，也可以定义日志类型、级别。

Appender主要`用于指定日志输出的目的地`，目的地可以是控制台、文件、远程套接字服务器、 MySQL、 PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。

Layout负责把事件转换成字符串，`格式化的日志信息的输出`。

各个logger 都被关联到一个 LoggerContext，`LoggerContext负责制造logger，也负责以树结构排列各 logger`。如果  logger的名称带上一个点号后是另外一个 logger的名称的前缀，那么，前者就被称为后者的祖先。如果logger与其后代  logger之间没有其他祖先，那么，前者就被称为子logger 之父。比如，名为"com.foo""的 logger  是名为"com.foo.Bar"之父。root logger 位于 logger 等级的最顶端，root logger  可以通过其名称取得，如下所示：

```
Logger rootLogger =LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);
```

其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger 名称为参数。`用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用`。

Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于 ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将`从有被分配级别的最近的祖先那里继承级别`。`root logger 默认级别是 DEBUG`。

##### 打印方法与基本的选择规则

打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO 的记录语句。`记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用`。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。

**该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR**。

##### Logback默认配置

如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback `默认地会调用BasicConfigurator ，创建一个最小化配置`。最小化配置由一个`关联到根 logger 的ConsoleAppender 组成`。`输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化`。`root logger 默认级别是 DEBUG`。

#### Logback的配置文件

Logback 配置文件的语法非常灵活。`正因为灵活，所以无法用 DTD 或 XML schema 进行定义`。尽管如此，可以这样描述配置文件的基本结构：以<configuration\>开头，后面有零个或多个<appender\>元素，有零个或多个<logger\>元素，有最多一个<root\>元素。

Logback默认配置的步骤

(1). 尝试在 classpath 下查找文件 `logback-test.xml`；

(2). 如果文件不存在，则查找文件 `logback.xml`；

(3). 如果两个文件都不存在，logback 用 `BasicConfigurator 自动对自己进行配置`，这会导致记录输出到控制台。

**mysql数据配置**

```mysql
BEGIN;
DROP TABLE IF EXISTS logging_event_property;
DROP TABLE IF EXISTS logging_event_exception;
DROP TABLE IF EXISTS logging_event;
COMMIT;

BEGIN;
CREATE TABLE logging_event 
  (
    timestmp         BIGINT NOT NULL,
    formatted_message  TEXT NOT NULL,
    logger_name       VARCHAR(254) NOT NULL,
    level_string      VARCHAR(254) NOT NULL,
    thread_name       VARCHAR(254),
    reference_flag    SMALLINT,
    arg0              VARCHAR(254),
    arg1              VARCHAR(254),
    arg2              VARCHAR(254),
    arg3              VARCHAR(254),
    caller_filename   VARCHAR(254) NOT NULL,
    caller_class      VARCHAR(254) NOT NULL,
    caller_method     VARCHAR(254) NOT NULL,
    caller_line       CHAR(4) NOT NULL,
    event_id          BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY
  );
COMMIT;


BEGIN;
CREATE TABLE logging_event_property
  (
    event_id       BIGINT NOT NULL,
    mapped_key        VARCHAR(254) NOT NULL,
    mapped_value      TEXT,
    PRIMARY KEY(event_id, mapped_key),
    FOREIGN KEY (event_id) REFERENCES logging_event(event_id)
  );
COMMIT;


BEGIN;
CREATE TABLE logging_event_exception
  (
    event_id         BIGINT NOT NULL,
    i                SMALLINT NOT NULL,
    trace_line       VARCHAR(254) NOT NULL,
    PRIMARY KEY(event_id, i),
    FOREIGN KEY (event_id) REFERENCES logging_event(event_id)
  );
COMMIT;
```

Logback.xml 文件

```
</configuration>
<?xml version="1.0" encoding="UTF-8"?>
<!--
    scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
    scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
    debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <property name="APP_NAME" value="base-mvc-web" />
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="/home/taomk" />
    <!--
        每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。
     -->
    <contextName>${APP_Name}</contextName>
    <!--
        key:标识此<timestamp> 的名字；datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。
    -->
    <timestamp key="BOOT_SECOND" datePattern="yyyyMMdd'T'HHmmss"/>
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!--
            encoder：对日志进行格式化，未配置class属性时，默认配置为PatternLayoutEncoder
        -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!-- 只输出level级别的日志 -->
        <filter class = "ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <!--
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
            -->
        </filter>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/${APP_NAME}.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <!--
            encoder：对日志进行格式化，未配置class属性时，默认配置为PatternLayoutEncoder
        -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
        <!-- 只输出level级别以上的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
    </appender>
    <!--日志异步到数据库 -->  
    <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        <!--日志异步到数据库 --> 
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           <!--连接池 --> 
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
              <user>root</user>
              <password>root</password>
            </dataSource>
        </connectionSource>
  </appender>
    <!--
        name：用来指定受此logger约束的某一个包或者具体的某一个类。
        level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前logger将会继承上级的级别。
        additivity：是否向上级logger传递打印信息。默认是true。
        <logger>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger。
    -->
    <!-- show parameters for hibernate sql 专为 Hibernate 定制 --> 
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder"  level="TRACE" />  
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor"  level="DEBUG" />  
    <logger name="org.hibernate.SQL" level="DEBUG" />  
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />  
    
    <!--myibatis log configure--> 
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
    
   
    <!--
        <root>：也是<logger>元素，但是它是根logger。只有一个level属性，应为已经被命名为"root".
    -->
    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
         <appender-ref ref="DB"/>
    </root>
</configuration>
```

LogbackDemo.java

```
    package logback;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    public class LogbackDemo {
        private static Logger log = LoggerFactory.getLogger(LogbackDemo.class);
        public static void main(String[] args) {
            log.trace("======trace");
            log.debug("======debug");
            log.info("======info");
            log.warn("======warn");
            log.error("======error");	 
            String name = "Aub";
            String message = "3Q";
            String[] fruits = { "apple", "banana" };	
            // logback提供的可以使用变量的打印方式，结果为"Hello,Aub!"
            log.info("Hello,{}!", name);	
            // 可以有多个参数,结果为“Hello,Aub! 3Q!”
            log.info("Hello,{}!   {}!", name, message);    		
            // 可以传入一个数组，结果为"Fruit:  apple,banana"
            log.info("Fruit:  {},{}", fruits); 
        }
    }
```

#### Spring中LogbackConfigListener使用

 在Web.xml中，添加如下配置：

```
    <!-- logback 日志配置 start -->
    <context-param>
        <param-name>logbackConfigLocation</param-name>
        <param-value>classpath:logback.xml</param-value>
    </context-param>

    <listener>
        <listener-class>ch.qos.logback.ext.spring.web.LogbackConfigListener</listener-class>
    </listener>
    <!-- logback 日志配置 end -->
```

