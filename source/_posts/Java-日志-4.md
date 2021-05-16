---
title: 日志框架--总结
date: 2019-05-13 22:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

> 学习的过程，离不开归根溯源的过程，所以记住她的历史和根源。

log4j可以当之无愧地说是Java日志框架的元老，1999年发布首个版本，2012年发布最后一个版本，2015年正式宣布终止，至今还有无数的系统在使用log4j，甚至很多新系统的日志框架选型仍在选择log4j。

然而老的不等于好的，在IT技术层面更是如此。尽管log4j有着出色的历史战绩，但早已不是Java日志框架的最优选择。

<!--more-->

在log4j被Apache Foundation收入门下之后，由于理念不合，log4j的作者Ceki离开并开发了slf4j和logback。

slf4j因其优秀的性能和理念很快受到了广泛欢迎，2016年的统计显示，github上的热门Java项目中，slf4j是使用率第二名的类库（第一名是junit）。

logback则吸取了log4j的经验，实现了很多强大的新功能，再加上它和slf4j能够无缝集成，也受到了欢迎。

在这期间，Apache Logging则一直在关门憋大招，log4j2在beta版鼓捣了几年，终于在2014年发布了GA版，不仅吸收了logback的先进功能，更通过优秀的锁机制、LMAX Disruptor、"无垃圾"机制等先进特性，在性能上全面超越了log4j和logback。

#### slf4j

slf4j是一个“日志门面”（Logging Facade），而不是一个完整的日志框架。它提供了一套记录日志的api，但不提供输出日志的功能，而是通过对接如log4j、java.util.logging等日志框架，来实现日志的输出。

所以说slf4j的对标选手实际上是Jakarta Commons Logging（简称JCL），二者的最大区别在于与日志服务的绑定机制

JCL采用的动态绑定机制：

1. 在进程启动时尝试获取名为"org.apache.commons.logging.Log"的配置属性（可与在commons-logging.properties文件中配置，或使用Java代码进行配置），按配置选取对应的日志输出服务
2. 如果没有获取到对应配置属性，会尝试在系统参数中寻找名为"org.apache.commons.logging.Log"的参数项
3. 如果1,2均没有获取到，会在classpath下寻找log4j的相关class，如果找到，则使用log4j作为日志输出服务
4. 如果没有找到log4j，则尝试使用java.util.logging包作为日志输出服务
5. 如果上述都失败，则使用SimpleLog作为日志输出服务，即将所有日志输出至控制台标准输出System.err

JCL的动态绑定机制基于ClassLoader实现，缺点一是效率较低，二是容易引发混乱，在一个复杂甚至混乱的依赖环境下，确定当前正在生效的日志服务是很费力的，特别是在程序开发和设计人员并不理解JCL的机制时，三是最致命的问题：在使用了自定义ClassLoader的程序中，使用JCL会引发各类问题，例如内存泄露、与OSGI冲突等。

而slf4j则简单得多，采用静态绑定机制：

- slf4j为各类日志输出服务提供了适配库，如slf4j-log4j12，slf4j-simple，slf4j-jdk14等。一个Java工程下只能引入一个slf4j适配库
- slf4j会加载org.slf4j.impl.StaticLoggerBinder作为输出日志的实现类。这个类在每个适配库中都存在，所以slf4j不需要像JCL一样主动去寻找日志输出实现，自然而然地就能与具体的日志输出实现绑定起来
- 当需要更换日志输出服务时（比如从logback切换回log4j），只需要替换掉适配库即可

所以slf4j不仅对比JCL有性能上的优势，使用slf4j的程序员也不需要去翻找配置文件或追踪启动过程就能够清除明白地了解当前使用的是什么日志输出服务

slf4j的优势还不止此：

1. 强制输出String，避免不规范代码

   传统的日志api都接收Object类型的参数，在程序员不遵守规范时容易引发一些错误，比如：

   ```
   SomeObject obj;
   //...
   logger.info(obj); //如果SomeObject并未覆盖toString()方法，这里就只记下来了hashcode
   ```

   又如：

   ```
   try {
     //...
   } catch(Exception e) {
     logger.error(e); //未记录异常stacktrace
   }
   ```

   slf4j的api强制要求传入String类型的参数，能够在一定程度上避免此类不规范的代码出现。

2. 日志模板功能

   在使用传统的日志api时，可能会有这样的代码：

   ```
   logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
   ```

   这引发了两个问题：

   1. 需要编写拼接字符串的代码，使开发效率降低
   2. 即使不需要输出这条日志（比如当前日志级别是ERROR时），也会执行拼接字符串的操作，消耗额外性能，占用额外内存

   而slf4j使用日志模板功能解决了这两个问题：

   ```
   logger.debug("Entry number: {} is {}", i, String.valueOf(entry[i]));
   ```

   不仅开发变得简单了，而且slf4j只会在此条日志确实需要输出时才会去拼装字符串。
    并且在输出异常信息时也可以使用模板，不会妨碍stacktrace的输出：

   ```
   String s = "Hello world";
   try {
     Integer i = Integer.valueOf(s);
   } catch (NumberFormatException e) {
     logger.error("Failed to format {}", s, e);
   }
   ```

3. slf4j的桥接功能

   虽然slf4j如此优秀，但一些类库因为历史原因仍然在使用JCL作为日志api（如Spring等），为此slf4j还推出了jcl-over-slf4j桥接库，能够把使用JCL的API输出的日志桥接到slf4j上，方便那些想要使用slf4j作为日志门面但同时又要使用Spring等需要依赖JCL的类库的系统。

   对于自动依赖JCL的类库，如要桥接至slf4j的话，除了引入jcl-over-slf4j适配库之外，还需要把JCL库从classpath中移除。可以在maven配置中将JCL库标记为provided：

   ```
   <dependency>
   <groupId>commons-logging</groupId>
   <artifactId>commons-logging</artifactId>
   <version>1.1.1</version>
   <scope>provided</scope>
   </dependency>
   
   ```

   同时slf4j还提供了log4j-over-slf4j等桥接库，能够在不改动代码的前提下把使用各种日志框架输出的日志都桥接到slf4j，如下图：

   ![3.png](https://i.loli.net/2019/05/18/5ce0058ebdff873999.png)

   

#### logback与log4j2

logback和log4j2都宣称自己是log4j的后代，一个是出于同一个作者，另一个则是在名字上根正苗红。

撇开血统不谈，比较一下log4j2和logback：

- log4j2比logback更新：log4j2的GA版在2014年底才推出，比logback晚了好几年，这期间log4j2确实吸收了slf4j和logback的一些优点（比如日志模板），同时应用了不少的新技术
- 由于采用了更先进的锁机制和LMAX Disruptor库，log4j2的性能优于logback，特别是在多线程环境下和使用异步日志的环境下
- 二者都支持Filter（应该说是log4j2借鉴了logback的Filter），能够实现灵活的日志记录规则（例如仅对一部分用户记录debug级别的日志）
- 二者都支持对配置文件的动态更新
- 二者都能够适配slf4j，logback与slf4j的适配应该会更好一些，毕竟省掉了一层适配库
- logback能够自动压缩/删除旧日志
- logback提供了对日志的HTTP访问功能
- log4j2实现了“无垃圾”和“低垃圾”模式。简单地说，log4j2在记录日志时，能够重用对象（如String等），尽可能避免实例化新的临时对象，减少因日志记录产生的垃圾对象，减少垃圾回收带来的性能下降

log4j2和logback各有长处，总体来说，如果对性能要求比较高的话，log4j2相对还是较优的选择。

### 选型

#### Commons Logging与Slf4j实现机制对比

- Commons logging实现机制

  Commons logging是通过动态查找机制，在程序运行时，使用自己的ClassLoader寻找和载入本地具体的实现。详细策略可以查看commons-logging-*.jar包中的org.apache.commons.logging.impl.LogFactoryImpl.java文件。由于OSGi不同的插件使用独立的ClassLoader，OSGI的这种机制保证了插件互相独立, 其机制限制了commons logging在OSGi中的正常使用。

- Slf4j实现机制

  Slf4j在编译期间，静态绑定本地的LOG库，因此可以在OSGi中正常使用。它是通过查找类路径下org.slf4j.impl.StaticLoggerBinder，然后绑定工作都在这类里面进。

#### 选择日志框架

1. slf4j已经成为了Java日志组件的明星选手，可以完美替代JCL，使用JCL桥接库也能完美兼容一切使用JCL作为日志门面的类库，现在的新系统已经没有不使用slf4j作为日志API的理由了

2. 日志记录服务方面，log4j在功能上输于logback和log4j2，在性能方面log4j2则全面超越log4j和logback。所以新系统应该在logback和log4j2中做出选择，对于性能有很高要求的系统，应优先考虑log4j2

如果是在一个新的项目中建议使用Slf4j与Logback组合，这样有如下的几个优点：

- Slf4j实现机制决定Slf4j限制较少，使用范围更广。由于Slf4j在编译期间，静态绑定本地的LOG库使得通用性要比Commons logging要好。

- Logback拥有更好的性能。Logback声称：某些关键操作，比如判定是否记录一条日志语句的操作，其性能得到了显著的提高。这个操作在Logback中需要3纳秒，而在Log4J中则需要30纳秒。LogBack创建记录器（logger）的速度也更快：13毫秒，而在Log4J中需要23毫秒。更重要的是，它获取已存在的记录器只需94纳秒，而Log4J需要2234纳秒，时间减少到了1/23。跟JUL相比的性能提高也是显著的。

- Commons Logging开销更高 在使Commons Logging时为了减少构建日志信息的开销，通常的做法是：

  ```
  if(log.isDebugEnabled()){
  log.debug("User name： " +
  user.getName() + " buy goods id ：" + good.getId());
  }
  ```

  Slf4j阵营，你只需这么做：

  ```
  log.debug(“User name：{} ,buy goods id ：{}”, user.getName(),good.getId());
  ```

  也就是说，slf4j把构建日志的开销放在了它确认需要显示这条日志之后，减少内存和cup的开销，使用占位符号，代码也更为简洁；

- Logback文档免费。Logback的所有文档是全面免费提供的，不象Log4J那样只提供部分免费文档而需要用户去购买付费文档

  

### 总结

java里常见的日志库有java.util.logging(JDKlog)、Apache log4j、log4j2、logback、slf4j等等。 这么多的日志框架里如何选择。 

首先需要梳理的是日志接口，以及日志接口对应的实现。然后再考虑如何选择使用哪个日志框架。 



![6.png](https://i.loli.net/2019/05/18/5ce0016fe3be871241.png)

目前的日志框架有jdk自带的：logging，log4j1、log4j2、logback

目前用于实现日志统一的框架（日志门面\接口）：apache的commons-logging、slf4j

为了理清它们的关系，与繁杂的各种集成jar包，如下：

- log4j、log4j-api、log4j-core
- log4j-1.2-api、log4j-jcl、log4j-slf4j-impl、log4j-jul
- logback-core、logback-classic、logback-access
- commons-logging
- slf4j-api、slf4j-log4j12、slf4j-simple、jcl-over-slf4j、slf4j-jdk14、log4j-over-slf4j、slf4j-jcl

#### 各种jar包总结

1. log4j1:
   
- log4j：log4j1的全部内容
  
2. log4j2:

   - log4j-api:log4j2定义的API

   - log4j-core:log4j2上述API的实现

3. logback:

   - logback-core:logback的核心包

   - logback-classic：logback实现了slf4j的API

4. commons-logging:

   - commons-logging:commons-logging的原生全部内容

   - log4j-jcl:commons-logging到log4j2的桥梁

   - jcl-over-slf4j：commons-logging到slf4j的桥梁

5. slf4j转向某个实际的日志框架：

   场景介绍：如 使用slf4j的API进行编程，底层想使用log4j1来进行实际的日志输出，这就是slf4j-log4j12干的事。

   - slf4j-log4j12：使用log4j-1.2作为日志输出服务

   - slf4j-jdk14：使用java.util.logging作为日志输出服务

   - slf4j-jcl：使用JCL作为日志输出服务

   - slf4j-simple：日志输出至System.err

   - slf4j-nop：不输出日志

   - log4j-slf4j-impl：使用log4j2作为日志输出服务

   logback天然与slf4j适配，不需要额外引入适配库（毕竟是一个作者写的）

6. 某个实际的日志框架转向slf4j：

   场景介绍：如 使用log4j1的API进行编程，但是想最终通过logback来进行输出，所以就需要先将log4j1的日志输出转交给slf4j来输出，slf4j再交给logback来输出。将log4j1的输出转给slf4j，这就是log4j-over-slf4j做的事。这一部分主要用来进行实际的日志框架之间的切换（下文会详细讲解）。

   - log4j-over-slf4j：将使用log4j api输出的日志桥接至slf4j

   - jcl-over-slf4j：将使用JCL api输出的日志桥接至slf4j

   - jul-to-slf4j：将使用java.util.logging输出的日志桥接至slf4j
   
   - log4j-to-slf4j：将使用log4j2输出的日志桥接至slf4j
   
    slf4j唯独没有提供log4j2的适配库和桥接库，log4j-slf4j-impl和log4j-to-slf4j都是Apache Logging自己开发的，看样子Ceki和Apache Logging的梁子真的很深啊……倒是Apache没有端架子，可能也是因为slf4j太火了吧

#### 集成总结

##### commons-logging与其他日志框架集成

1. commons-logging与jdk-logging集成，需要的jar包：
   - commons-logging

2. commons-logging与log4j1集成，需要的jar包：

   - commons-logging

   - log4j

3. commons-logging与log4j2集成，需要的jar包：

   - commons-logging

   - log4j-api

   - log4j-core

   - log4j-jcl(集成包)

4. commons-logging与logback集成，需要的jar包：

   - logback-core

   - logback-classic

   - slf4j-api、jcl-over-slf4j(2个集成包,可以不再需要commons-logging)

5. commons-logging与slf4j集成，需要的jar包：

   - jcl-over-slf4j(集成包，不再需要commons-logging)

   - slf4j-api

##### slf4j与其他日志框架集成

![5.png](https://i.loli.net/2019/05/18/5cdffd9ac807966995.png)

jar包说明如下：

![](https://i.loli.net/2019/05/18/5ce003124904574742.png)

与其他框架集成：

1. slf4j与jdk-logging集成，需要的jar包：
   - slf4j-api

   - slf4j-jdk14(集成包)
2. slf4j与log4j1集成，需要的jar包：

   - slf4j-api

   - log4j

   - slf4j-log4j12(集成包)
3. slf4j与log4j2集成，需要的jar包：

   - slf4j-api

   - log4j-api

   - log4j-core

   - log4j-slf4j-impl(集成包)
4. slf4j与logback集成，需要的jar包：

   - slf4j-api

   - logback-core

   - logback-classic(集成包)
5. slf4j与commons-logging集成，需要的jar包：

   - slf4j-api

   - commons-logging

   - slf4j-jcl(集成包)



#### 日志系统之间的切换

![3.png](https://i.loli.net/2019/05/18/5ce0058ebdff873999.png)

##### log4j无缝切换到logback

我们已经在代码中使用了log4j1的API来进行日志的输出，现在想不更改已有代码的前提下，使之通过logback来进行实际的日志输出。已使用的jar包：

- log4j

使用案例：

```
private static final Logger logger=Logger.getLogger(Log4jTest.class);
public static void main(String[] args){
    if(logger.isInfoEnabled()){
        logger.info("log4j info message");
    }
}
```

上述的Logger是log4j1自己的org.apache.log4j.Logger，在上述代码中，我们在使用log4j1的API进行编程。现在如何能让上述的日志输出通过logback来进行输出呢？只需要更换一下jar包就可以：

第一步：去掉log4j jar包,Maven中使用exclude

第二步：加入以下jar包

- `log4j-over-slf4j（实现log4j1切换到slf4j）`
- slf4j-api
- logback-core
- `logback-classic`

第三步：在类路径下加入logback的配置文件

##### jdk-logging无缝切换到logback

```
private static final Logger logger=Logger.getLogger(JulSlf4jLog4jTest.class.getName());

public static void main(String[] args){
    logger.log(Level.INFO,"jul info a msg");
    logger.log(Level.WARNING,"jul waring a msg");
}
```

可以看到上述是使用jdk-logging自带的API来进行编程的，现在我们想这些日志交给logback来输出，解决办法如下：

第一步：加入以下jar包：

- `jul-to-slf4j （实现jdk-logging切换到slf4j）`
- slf4j-api
- logback-core
- `logback-classic`

第二步：在类路径下加入logback的配置文件

第三步：在代码中加入如下代码：

```
static {
    SLF4JBridgeHandler.install();
}
```

##### commons-logging切换到logback

使用的jar包

- commons-logging

案例如下：

```
private static Log logger=LogFactory.getLog(JulJclTest.class);

public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-jcl trace message");
    }
}   
```

可以看到我们使用commons-logging的API来进行日志的编程操作，现在想切换成logback来进行日志的输出（这其实就是commons-logging与logback的集成），解决办法如下：

第一步：去掉commons-logging jar包（其实去不去都无所谓）

第二步：加入以下jar包：

- `jcl-over-slf4j（实现commons-logging切换到slf4j）`
- slf4j-api
- logback-core
- logback-classic

第三步：在类路径下加入logback的配置文件

#### 冲突说明

##### jcl-over-slf4j 与 slf4j-jcl 冲突

- jcl-over-slf4j： commons-logging切换到slf4j
- slf4j-jcl : slf4j切换到commons-logging

如果这两者共存的话，必然造成相互委托，造成内存溢出。

##### log4j-over-slf4j 与 slf4j-log4j12 冲突

- log4j-over-slf4j ： log4j1切换到slf4j
- slf4j-log4j12 : slf4j切换到log4j1

如果这两者共存的话，必然造成相互委托，造成内存溢出。但是`log4j-over-slf4内部做了一个判断，可以防止造成内存溢出`：即判断slf4j-log4j12 jar包中的org.slf4j.impl.Log4jLoggerFactory是否存在，`如果存在则表示冲突了，抛出异常提示用户要去掉对应的jar包`

##### jul-to-slf4j 与 slf4j-jdk14 冲突

- jul-to-slf4j ： jdk-logging切换到slf4j
- slf4j-jdk14 : slf4j切换到jdk-logging

如果这两者共存的话，必然造成相互委托，造成内存溢出。



### 参考：

[该让log4j退休了 - 论Java日志组件的选择](https://www.jianshu.com/p/85d141365d39)