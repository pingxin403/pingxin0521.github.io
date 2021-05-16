---
title: 日志框架--Log4j
date: 2019-05-11 13:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

### [log4j](http://logging.apache.org/log4j/1.2/)

Apache Log4j是当前在J2EE和J2SE开发中用得最多的日志框架（几乎所有项目都用它），因为它具有出色的性能、灵活的配置以及丰富的功能，并且在业务有特殊的要求时，可以使用自定义组件来代替框架中已有的组件来满足要求。

[补充教程](https://www.yiibai.com/log4j)

<!--more-->

基本上所有的大型应用，包括我们常用的框架，比如hibernate；spring；struts等，在其内部都做了一定数量的日志信息。为什么要做这些日志信息，在系统中硬编码日志记录信息是调试系统，观察系统运行状态的一种方式。可能大部分程序员都还记得自己最开始写代码的时候，写一个方法总是出错，就喜欢使用System.out.println(“1111111”)之类的代码来查看程序的运行是否按照自己想要的方式在运行，其实这些sysout就是日志记录信息，但使用System.out.println或者System.err.println在做日志的时候有其非常大的局限性：

`第一，所有的日志信息输出都是平行的。`比如我不能单独的控制只输出某一个或者某几个模块里面的日志调试信息；或者我不能控制只输出某一些日志。

`第二，日志信息的输出不能统一控制。`比如我不能控制什么时候打印信息，什么时候不打印信息，如果我不想打印任何信息，我只能选择到所有的java文件中去注释这些日志信息。

`第三，日志信息的输出方式是固定的。`比如，在编码期，就只能输出到eclipse的控制台上，在tomcat环境中就只能输出到tomcat的日志文件中，我不能选择比如输出日志到数据库或者输出日志到Email中等等复杂的控制。

所以，综上所述，在编码阶段，往往需要一种统一的硬编码日志方式，来记录程序的运行状态，并且能够提供简单的方式，统一控制日志的输出粒度和输出方式。Log4J就是在这种情况下诞生的，而现在Log4J已经被广泛的采用，成为了最流行的日志框架之一。

Log4J提供了非常简单的使用方式，如果不需要深入去发现Log4J，或者需要自己去扩展Log4J，那么使用起来是非常方便的。而Log4J也非常关注性能的问题，因为在应用中大量的采用日志的方式，会带来一些反面的问题：

1. 会使一段代码的篇幅增大，增加阅读的难度；
2. 增加代码量，会在一定程度上降低系统运行性能；

而Log4J在性能上不断的优化，使普通的日志输出的性能甚至做到比System.out.println更快。最后，Log4J提供了非常方便的扩展方式，能让我们非常简单的自定义自己的日志输出级别，日志输出方式等。

log4j maven依赖如下：

```
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

示例：

```
public class Log4jTest {
    private static Logger logger = Logger.getLogger(Log4jTest.class);

    public static void main(String[] args) {
        // 从字面意思上看非常简单，我们使用了一个基础配置器，并调用其configure()方法，即配置方法完成了配置。
        BasicConfigurator.configure();
        logger.debug("my first log4j info");
    }
}
```

就这么简单，通过这个例子，其实我们已经演示完了Log4j的使用方式，要把Log4J加入你的应用，只需要这么几步：

1. 导入Log4J的包（或者在maven中配置）；
2. 完成Log4J的配置。在上面的应用中，我们仅仅是使用了`最简单也最缺乏灵活性的BasicConfigurator来完成配置`，后面我们会逐步看到更灵活的配置方式，但是配置总是需要的一个步骤。
3. 对于每一个需要日志输出的类来说，`通过Logger.getLogger方法`得到一个专属于这个类的日志记录器，
4. 使用日志记录器完成日志信息的输出；在上面的例子中，我们仅仅看到了logger的debug方法，后面会看到更多的日志记录方式。

Log4j支持两种配置文件格式：

- XML格式
- properties（key=value）文件（常用）

#### 获取Logger的原理

三种情况来说明：

**第一种情况**：没有指定配置文件路径 

1. 第一步： `引发LogManager的类初始化`：Logger.getLogger(Log4jTest.class)的源码如下：

```
    static public Logger getLogger(Class clazz) {
        return LogManager.getLogger(clazz.getName());
    }
```

2. 第二步：`初始化一个logger仓库Hierarchy`，Hierarchy的源码如下：

```
    public class Hierarchy implements LoggerRepository, RendererSupport, ThrowableRendererSupport {
        private LoggerFactory defaultFactory； // 创建Logger的工厂
        Hashtable ht; // 用来存放上述工厂创建的Logger
        Logger root; 作为根Logger
        //其他略
    }
```

LogManager在类初始化的时候如下方式来实例化Hierarchy：

```
    static {
        // new RootLogger作为root logger，默认是debug级别
        Hierarchy h = new Hierarchy(new RootLogger((Level) Level.DEBUG));
        // 略
    }
```

最后把Hierarchy绑定到LogManager上，可以在任何地方来获取这个logger仓库Hierarchy

3. 第三步：在LogManager的类初始化的过程中`默认寻找类路径下的配置文件`，通过org.apache.log4j.helpers.Loader类来加载类路径下的配置文件：

```
Loader.getResource("log4j.xml");
Loader.getResource("log4j.properties");
```

`优先选择xml配置文件`。

4. 第四步：`解析上述配置文件`

- 如果是xml文件则org.apache.log4j.xml.DOMConfigurator类来解析
- 如果是properties文件，则使用org.apache.log4j.PropertyConfigurator来解析

不再详细说明解析过程，看下解析后的结果：

- 设置RootLogger的级别
- 对RootLogger添加一系列我们配置的appender（我们通过logger来输出日志，通过logger中的appender指明了日志的输出目的地）

5. 第五步：当一切都准备妥当后，就该获取Logger了 使用logger仓库Hierarchy中内置的LoggerFactory工厂来创建Logger了，并缓存起来，同时将logger仓库Hierarchy设置进新创建的Logger中。

**第二种情况**，手动来加载不在类路径下的配置文件 PropertyConfigurator.configure 执行时会去进行上述的配置文件解析，源码如下：

```
public static void configure(java.net.URL configURL) {
     new PropertyConfigurator().doConfigure(configURL,
                        LogManager.getLoggerRepository());
}
```

- 仍然先会引发LogManager的类加载，创建出logger仓库Hierarchy，同时尝试加载类路径下的配置文件，此时没有则不进行解析，**此时logger仓库Hierarchy中的RootLogger默认采用debug级别**，没有appender而已。
- 然后解析配置文件，对上述logger仓库Hierarchy的RootLogger进行级别的设置，添加appender。
- 此时再去调用Logger.getLogger，不会导致LogManager的类初始化（因为已经加载过了）。

**第三种情况**，配置文件在类路径下，而我们又手动使用PropertyConfigurator去加载

也就会造成2次加载解析配置文件，仅仅会造成覆盖而已（对于RootLogger进行从新设置级别，删除原有的appender，重新加载新的appender），所以`多次加载解析配置文件以最后一次为准`。

#### 主要对象总结

- LogManager： 它的类加载会创建logger仓库Hierarchy，并尝试寻找类路径下的配置文件，如果有则解析
- Hierarchy ： 包含三个重要属性：
- LoggerFactory logger的创建工厂
- Hashtable 用于存放上述工厂创建的logger
- Logger root logger,用于承载解析文件的结果，设置级别，同时存放appender
- PropertyConfigurator: 用于解析log4j.properties文件
- Logger : 我们用来输出日志的对象

### Log4j体系结构

当我们在描述为系统做日志这个动作的时候，实际上描述了3个点；类似于小学语文学语法一样。做日志，其实就是在规定，在什么地方用日志记录器以什么样的格式做日志。`把三个最重要的点抽取出来，即什么地方，日志记录器，什么格式`。在Log4J中，就使用了三个最重要的组件来描述这三个要素，即：

> Logger：日志记录器
>
> Appender：什么地方
>
> Layout：什么格式

#### Logger的结构

Logger就是最重要的，我们直接用来完成日志的工具。使用日志框架一个非常重要的目的就是让程序员能够方便的控制特定的日志的输出。`要能够控制特定日志的输出，就需要创建一种特殊的结构来划分我们的日志记录器。而对于JAVA程序员来说，按照包的层次关系划分Logger是最容易想到，也最贴近我们的代码结构的`。

之前我们在创建Logger的时候，都是使用Logger.getLogger(Class)方法来得到一个类绑定的日志记录器的。`实际上，当我们说把一个日志记录器绑定在一个类上，这种说法是不准确的，正确的说，我们仅仅是使用给定的类的全限定名为Logger取了一个名字`。这里请大家注意一下，Logger都是有名字的，假如我们除开rootLogger不谈，我们可以把Logger想象成一张表里面的数据，Logger对应的名字就是其主键，当两个Logger的名字相同，这两个Logger就是同一个Logger实例。我们可以简单的通过一个实例来验证：

```
@Test
public void testLoggerName(){
    BasicConfigurator.configure();
    Logger log1=Logger.getLogger("a");
    Logger log2=Logger.getLogger("a");
    log1.info(log1==log2);
}
```

控制台打印：

```
0 [main] INFO a  - true；
```

说明`Logger自身维护着每一个名字的Logger实例的引用，保证相同名字的Logger在不同地方获取到的实例是一致的`，这样就允许我们在统一的代码中配置不同Logger的特性。

另外，`Logger的层次结构，也是靠Logger的名字来区分的`，比如：名称为java的Logger就是java.util的父Logger；java.util是java.util.Collection的父Logger；`Logger的体系结构和package的结构划分类似，使用.来区分`；所以我们前面才说，使用类的全限定名是最简单，也是最符合logger的体系结构的命名方式。当然，你也可能使用任何的能想到的方式去处理Logger的命名；这也是可以的。

看到这里，可能有的人会提出疑问，那既然Logger的层次结构是`按照Logger的名字来创建的，那在创建Logger的时候，是否必须按照其结构来顺序创建Logger？`比如：

```
Logger log1=Logger.getLogger(“cd”);
Logger log2=Logger.getLogger(“cd.itcast”);
Logger log3=Logger.getLogger(“cd.itcast.log”);
```

是否必须要按照这个顺序创建呢？不需要。在做前面的示例的时候，我们说到，大家可以认为当我们通过Logger.getLogger(“cd.itcast.log”)的时候，Log4J其实为我们创建了cd；cd.itcast和cd.itcast.log这三个Logger；`其实不然，如果是这样的话，Log4J可能会产生非常多不必要的Logger`。所以，真正的方式应该是，`当我通过Logger.getLogger(“cd.itcast.log”)的时候，Log4J仅仅为我们创建了cd.itcast.log这个Logger`；而当我们`再次使用Logget.getLogger(“cd.itcast”)的时候，Log4J才会为我们创建cd.itcast这个Logger，并且Log4J会自动的去寻找这个Logger的上下级关系，并自动的把这个新创建的Logger添加到已有的Logger结构体系中`。

上面说到，除了rootLogger之外，其他的Logger都有名字，那么rootLogger呢?rootLogger有下面的规范：

> rootLogger是没有名字的；
>
> rootLogger是自动存在的；
>
> rootLogger必须设置Level等级；
>
> rootLogger没有名字，所以无法像其他的类一样，可以通过Logger.getLogger(String)得到，他只能通过Logger.getRootLogger方法得到；而我们前面使用的Logger.getLogger(Class)方法，其实相当于Logger.getLogger(Class.getName())而已；所以真正赋予Logger的是一个名字，而不是类型；

`在Logger中，非常重要的一个组件，就是Logger的日志级别，即Level`。关于Level的划分，使用，我们在前面的例子中已经大概了解到了，下面来看看完整的Level的定义和其在Logger体系中的继承方式，这是一个很简单的概念。

首先，在Log4J中，为我们默认的定义了7种不同的Level级别，即`all<debug<info<warn<error<fatal<off`；而这7中Level级别又刚好对应着Level类中的7个默认实例：`Level.ALL;Level.DEBUG;Level.INFO;Level.WARN;Level.ERROR;Level.FATAL和Level.OFF`。我们在之前也只看到了其中的debug到fatal这5种；这五种日志级别正好对应着Logger中的五个方法，即：

```
public class Logger {
    // 输出日志方法:
    public void debug(Object message);
    public void info(Object message);
    public void warn(Object message);
    public void error(Object message);
    public void fatal(Object message);
    // 输出带有错误的日志方法：
    public void debug(Object message, Throwable t);
    public void info(Object message, Throwable t);
    public void warn(Object message, Throwable t);
    public void error(Object message, Throwable t);
    public void fatal(Object message, Throwable t);
    // 更通用的输出日志方法:
    public void log(Level p, Object message);
}
```

为什么这里列出了11个方法，其实大家仔细看一下就知道了，最上面5个方法和中间的5个方法其实是对应的，只是中间的5个方法都带有对应的错误信息。而更下面的log方法，是允许我们直接使用Level类型来输出日志，这给了我们直接使用自定义的Level级别提供了输出日志的方式。换句话说，debug(Object  message)其实就是log(Level.DEBUG,message)的简写而已。

要能输出日志，按照常规来说，每一个Logger实例都应该设置其对应的Level；但是如果这样做，会让我们控制Logger的Level非常麻烦，而`Log4J借助Logger的层级关系，使用继承的方式来最大限度的减少Level的设置`。这个规律我们在之前的代码中已经讲过。

`一种不太常用的方式是使用Threshold方式来限制日志输出`。Threshold我们可以理解为门槛，它和Level的概念完全一致，只是使用方式有一些区别，先来看代码：

```
@Test
public void testLogLevel2() {
    BasicConfigurator.configure();

    Logger logger = Logger.getLogger("cd.itcast");
    logger.getLoggerRepository().setThreshold(Level.WARN);
    Logger barLogger = Logger.getLogger("cd.itcast.log");

    logger.warn("logger warn");
    logger.debug("logger debug");
    barLogger.info("bar logger info");
    barLogger.debug("bar logger debug");
}
```

我们先创建了一个名字为cd.itcast的Logger，这次我们并没有设置其Level，而是`使用logger.getLoggerRepository()方法得到了一个LoggerRepository对象`。这个LoggerRepository可以理解为`Logger栈，简单说，就是从当前Logger开始其下的所有的子Logger`。然后，我们设置了这个栈的日志门槛，Threshold就是门槛的意思，`换句话说就是最低日志输出级别为WARN`，那么其下的所有的Logger的最低日志输出级别就变为了WARN。所以，这段代码执行的结果是：

```
0 [main] WARN cd.itcast  - logger warn
```

Threshold优先级>Level优先级，这里需要注意一点，`Threshold优先级大于Level优先级，不等于Level就没有了意义`。假如Threshold设置的级别为DEBUG，而Level设置的等级为INFO，那么最终，logger.debug还是不会输出日志。

Threshold的方式和Level一样，`如果子Logger设置了自己的Threshold，则会使用自己的Threshold，如果没有设置，则继续向上查询，直到找到一个父类的Threshold或者rootLogger的level`。只要父Logger的Threshold设置好了，子Logger的Level也失效了。

上面简单的介绍了Logger和Level的意义和使用方式，其实，在真正使用Log4J的时候，我们一般都不需要使用代码的方式去配置Level或者Threshold，这些配置更多的时候是使用配置文件来完成的，这个后面会详细介绍。但是，不管使用什么样的配置文件，最终也会解释成这样的配置代码，所以，理解了这些代码，再去使用配置文件，会更加清楚到底配置文件在干什么，同样，为我们自己去扩展Log4J的配置，想要自己去实现自定义的配置文件格式，提供了可能。

#### Appender的结构

`使用Logger的日志记录方法，仅仅是发出了日志记录的事件，具体日志要记录到什么地方，需要Appender的支持`。在Log4J中，Appender定义了日志输出的目的地。在上面所有的示例当中，我们日志输出的目的地都是控制台，在Log4j中，还有非常多的Appender可供选择，可以将日志输出到文件，网络，数据库等等，这个后面再介绍。

说到这里，可能有人就已经会思考，既然Logger对象的info()等方法仅仅是发出了日志记录的事件，还需要指定输出目的地；那么我们之前的示例代码也并没有为任何一个Logger设置Appender啊？其实这很好理解，我们回顾一下之前的Level，按道理，也应该为每一个Logger指定对应的日志输出级别，但是我们也并没有这样做，`正是因为Logger本身存在一个完整的体系结构，而Level能够在这个结构中自下而上的继承。同理，Appender也具有这种继承的特性`。下面给出一段代码，先看看怎么使用代码的方式指定Appender：

```
@Test
public void testLogAppender1(){
    Logger log1=Logger.getLogger("cd");
    log1.setLevel(Level.DEBUG);
    log1.addAppender(new ConsoleAppender(new SimpleLayout()));
    Logger log2=Logger.getLogger("cd.itcast.log");
    log2.info("log2 info");
    log2.debug("log2 debug");
}
```

第一句代码设置了日志级别为DEBUG；`第二条代码，调用了Logger的addAppender 方法添加了一个ConsoleAppender；从类的名字上看就知道这是一个把日志输出到控制台上的Appender，在创建ConsoleAppender的时候，又传入了一个SimpleLayout的实例`；关于Layout下面再介绍，现在只需要关注Appender的继承特性。接下来，又创建了一个cd.itcast.log的子Logger；并且使用这个Logger输出了两条日志信息。运行测试，输出：

```
INFO - log2 info
DEBUG - log2 debug
```

请注意这个输出，很明显已经和之前的输出信息的格式完全不一样了，这里的输出格式就是由我们在cd这个Logger上面设置的ConsoleAppender+SimpleLayout所规定的。从这个例子中，我们可以看到，我们改变了cd这个Logger的Appender；`他下面的子Logger自然就继承了这个Appender，输出了另外一种格式的信息`。从这段代码中，我们能看出Logger的继承性，假如我们把代码修改为以下这样：

```
@Test
public void testLogAppender1(){
    BasicConfigurator.configure();
    Logger log1=Logger.getLogger("cd");
    log1.setLevel(Level.DEBUG);
    log1.addAppender(new ConsoleAppender(new SimpleLayout()));
    Logger log2=Logger.getLogger("cd.itcast.log");
    log2.info("log2 info");
    log2.debug("log2 debug");
}

```

在这段代码中，我们仅仅只是添加了BasicConfigurator来完成一个基本的配置。我们先来分析下这段代码。首先我们完成了基本的配置，从前面的测试代码中，我们可以知道，这条代码为rootLogger设置了一个DEBUG的Level；`另外，现在我们知道了，这条代码肯定还为rootLogger添加了一个ConsoleAppender`。然后我们创建了名字为cd的Logger，并且另外添加了一个Appender；接着又创建了名字为cd.itcast.log的Logger，最后使用这个Logger输出了两条日志信息。根据前面的Level的表现，我们猜想，当使用cd.itcast.log这个Logger做日志的时候，`因为这个Logger本身是没有添加任何Appender，所以他会向上查询任何一个添加了Appender的父Logger，即找到了cd这个Logger，最后使用cd这个Logger完成日志`，那么我们预计的结果是这段代码和上一段代码输出相同。

我们来运行一下这段代码，输出：

```
INFO - log2 info
0 [main] INFO cd.itcast.log  - log2 info
DEBUG - log2 debug
0 [main] DEBUG cd.itcast.log  - log2 debug
```

和我们预测的结果不一样。log2 info和log2  debug分别被输出了两次。我们观察结果，两条日志的输出都是先有一条ConsoleAppender+SimpleLayout的方式输出的然后紧跟一条使用BasicConfigurator的输出方式。那我们就能大胆的猜测了，`Logger上的Appender不光能继承其父Logger上的Appender，更重要的是，他不光只继承一个，而是只要是其父Logger，其上指定的Appender都会追加到这个子Logger之上`。所以，这个例子中，`cd.itcast.log这个Logger不光继承了cd这个Logger上的Appender，还得到了rootLogger上的Appender`；所以输出了这样的结果。在Log4J中，这个特性叫做Appender的追加性。默认情况下，所有的Logger都自动具有追加性，通过一个表来说明：

![3.png](https://i.loli.net/2019/05/18/5cdfd99dc3a6d67692.png)

但是，在某些情况下，这样做反而会引起日志输出的混乱。有些时候，我们并不希望Logger具有追加性。比如在上面这张表中，我们想让cd.itcast.log只需要继承A2和自己的A3Appender，而不想使用root上面的A1  Appender，又该怎么做呢？

其实很简单，`在Logger上，都有一个setAdditivity方法，如果设置setAdditivity为false，则该logger的子类停止追加该logger之上的Appender；如果设置为true，则具有追加性`。修改一下上表：

![4.png](https://i.loli.net/2019/05/18/5cdfd9d51454b52186.png)

再来一段代码看看是否如此：

```
@Test
public void testLogAppender2() throws Exception{
    BasicConfigurator.configure();
    Logger log1=Logger.getLogger("cd");
    log1.setAdditivity(false);
    log1.addAppender(new ConsoleAppender(new SimpleLayout()));

    Logger log2=Logger.getLogger("cd.itcast");
    log2.addAppender(new FileAppender(new SimpleLayout(),"a0.log"));

    Logger log3=Logger.getLogger("cd.itcast.log");
    log3.info("log2 info");
}
```

先来分析这段代码，在这段代码中有一些新的知识，简单理解即可。首先，我们`使用BasicConfigurator.configure()方法配置了rootLogger`；接着定义了名称为cd的Logger；并为其添加了一个ConsoleAppender，但是这里，我们这里`设置了additivity为false，即cd和cd之后的logger都不会再添加rootLogger的Appender了`。接下来，我们创建了cd.itcast这个Logger，并且为这个Logger指定了一个FileAppender。FileAppender很简单，除了同样要指定一个Layout，这个在后面介绍，第二个参数还需要指定输出日志的名称；最后，我们创建了cd.itcast.log，并使用这个Logger输出日志。按照上面的表所展示的规律，因为cd.itcast.log没有指定任何的Appender，所以向上查询。找到cd.itcast，cd.itcast.log得到其上的FileAppender，因为cd.itcast没有设置additivity，默认为true，继续向上查找，cd.itcast.log会得到cd的ConsoleAppender；但是因为cd设置了additivity为false，所以不再向上查询，最后，cd.itcast.log会向FileAppender和ConsoleAppender输出日志。

运行测试，结果：

```
INFO - log2 info
```

并且在应用下增加一个a0.log文件，内容为INFO - log2 info。符合我们的预期。

**在Log4J中，一个Logger可以添加多个Appender，不管是通过继承的方式还是通过调用Logger.addAppender方法添加。只要添加到了某一个Logger之上，在这个Logger之上的任何一个可以被输出的日志都会分别输出到所有的Appender之上。**

#### Layout的结构

 Logger规定了输出什么日志，Appender规定了日志输出到哪里，当然，我们还会奢望，以什么样的方式输出日志。这就涉及到之前我们在观察Appender的时候创建ConsoleAppender和FileAppender都需要传入的Layout。`在Log4J中，Layout对象提供了以什么样的方式格式化日志。这个对象是绑定在Appender之上的，一般在Appender创建的时候指定`。

下面就简单看一个`最常使用的Layout：PatternLayout`。`PatternLayout允许使用标准的输出格式来指定格式化日志消息的样式`。举个简单的例子，可能之前大家看到的使用BasicConfigurator配置的rootLogger输出的日志样式和我们使用ConsoleAppender(new SimpleLayout)创建的输出样式完全不一样。那大抵类似：

```
0 [main] INFO cd.itcast.core.LogicProcessor  - process some logic
```

那这样的日志输出样式是什么样子的呢？一个代码：

```
@Test
public void testPatternLayout(){
    Logger log=Logger.getLogger("cd");
    String pattern="%r [%t] %-5p %c - %m%n";
    log.addAppender(new ConsoleAppender(new PatternLayout(pattern)));
    Logger log2=Logger.getLogger("cd.itcast.log");
    log2.info("log2 info");
}
```

运行测试输出：

```
0 [main] INFO  cd.itcast.log - log2 info
```

符合我们的预期。注意粗体字。`首先定义了一个格式化日志的模式，在这个模式中，有很多以%开头的参数，每一个特定的参数代表着一种日志的内容`。比如%r代表从Log4j启动到运行这条日志的时间差，单位为毫秒；第二个参数%t代表运行该日志的线程名称；第三个参数%-5p，首先-5代表这个字符总占用5个位置，p代表日志输出级别；%c代表输出日志的Logger的名字；%m代表输出的日志内容；%n代表分行。可以看到，其实PatternLayout的使用是非常简单的，只需要了解所有内置的参数和其表示的含义，再按照想要的方式输出即可。

#### 三个组件的使用

前面简单的了解了Log4J中最重要的3个组件，下面我们来看看`Log4j是怎么使用这3个组件完成当我们调用logger.debug()方法能在控制台上打印出日志信息的`。

1. `第一步，继承数体系上的门槛检查`：首先当调用info()方法后，Log4J会立刻使用该Logger所在的体系结构中设置的门槛去检查当前日志的级别。如果级别不够，立刻作废当前日志请求。
2. `第二步，Level级别检查`：使用当前Logger上设置的或者继承的Level级别来检查当前的日志级别。如果当前日志级别不够，立刻作废当前日志请求。
3. `第三步，创建LoggingEvent对象`：当日志审核通过，Log4J就会创建一个LoggingEvent对象（即日志事件对象）。在该对象中，会保存和本次日志相关的所有参数信息，包括日志内容，日志时间等。
4. `第四步，执行Appender`：当创建完成LoggingEvent对象时候，会该对象交给当前logger上起作用的所有的Appender对象，并调用这些对象的doAppend方法来处理日志消息。
5. `第五步，格式化日志消息`：接下来，会使用每一个Appender绑定的Layout对象（如果有）来格式化日志消息。Layout对象会把LoggingEvent格式化成最终准备输出的String。
6. `第六步，输出日志消息`：当得到最终要输出的String对象之后，appender会把字符输出到最终的目标上，比如控制台或者文件。

三个主要组件的组成结构：

![5.png](https://i.loli.net/2019/05/18/5cdfda9c1b55738853.png)

日志执行流程图：

![6.png](https://i.loli.net/2019/05/18/5cdfdabc7100225173.png)

日志类结构图：

![7.png](https://i.loli.net/2019/05/18/5cdfdae087c5776575.png)





### 配置文件

通过前面的示例，我们已经对Log4j的使用方式有了一定的了解。我们再返回去看我们第二个稍微复杂的例子。`单看Configure和LogicProcessor两个类，从代码的角度来看，在这两个类中硬编码日志，是没有问题的，也是没法优化的，比如在代码中添加log.info(string)，这种代码是必要的；在这种情况下，我应用中所有的类必要引入的也就只是一个Logger类，引入这个复杂性也是我们能够控制的`。但是最重要的注意力，我们来思考，在真正一个应用当中，我们的测试类又该怎么表现呢？通过最开始的示例代码，我们已经知道，`要正常的运行Log4J的日志功能，必须要至少对rootLogger进行相关的配`，之前随着我们的代码的复杂度越来越高，我们发现，要能够控制我们的日志的输出级别，各个模块的日志输出控制，我们必须要在测试代码中加入大量的Log4J的代码；比如设定Level，Threshold，Appender，Layout，Pattern等等代码，换句话说，`如果这些代码都必须硬编码到程序中，我们必须要在我们的应用启动的时候就执行这些代码`，比如在WEB环境中，就要使用`ServletContextListener`等来初始化Log4J环境，或者在我们的桌面应用的启动过程中，使用这些代码来控制Log4J的初始化。这样做的后果是，虽然我们也能达到统一控制日志输出的目的，但是我们仍然需要使用大量的代码来控制这些内容；也没法达到一种灵活统一配置的方式，因为我们知道，`在大部分的情况下，使用配置文件的方式肯定优于代码的配置方式`。

Log4J的配置文件(Configuration File)就是`用来设置记录器的级别、存放器和布局的，它可接key=value格式的设置或xml格式的设置信息`。通过配置，可以创建出Log4J的运行环境。

#### Properties配置文件

Log4J配置文件的基本格式如下：

```
#配置根Logger
log4j.rootLogger  =   [ level ]   ,  appenderName1 ,  appenderName2 ,  …

#配置日志信息输出目的地Appender
log4j.appender.appenderName  =  fully.qualified.name.of.appender.class 
log4j.appender.appenderName.option1  =  value1 
… 
log4j.appender.appenderName.optionN  =  valueN 

#配置日志信息的格式（布局）
log4j.appender.appenderName.layout  =  fully.qualified.name.of.layout.class 
log4j.appender.appenderName.layout.option1  =  value1 
… 
log4j.appender.appenderName.layout.optionN  =  valueN 
```

其中 [level] 是日志输出级别，共有5级：

```
FATAL       0  记录影响系统正常运行，可能导致系统崩溃的事件
ERROR       3  记录影响业务流程正常进行，导致业务流程提前终止的事件
WARN        4  记录未预料到，可能导致业务流程无法进行的事件
INFO        6  记录系统启动/停止，模块加载/卸载之类事件
DEBUG       7  记录业务详细流程，用户鉴权/业务流程选择/数据存取事件
TRACE          记录系统进出消息，码流信息
TRACE <debug<info<warn<error<fatal
```

Appender 为日志输出目的地，Log4j提供的appender有以下几种：

```
org.apache.log4j.ConsoleAppender（控制台），
org.apache.log4j.FileAppender（文件），
org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），
org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件），
org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
```

Layout：日志输出格式，Log4j提供的layout有以下几种：

```
org.apache.log4j.HTMLLayout（以HTML表格形式布局），
org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）
```

打印参数: Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，如下：

```
%m   输出代码中指定的消息
%p   输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL 
%r   输出自应用启动到输出该log信息耗费的毫秒数 
%c   输出所属的类目，通常就是所在类的全名 
%t   输出产生该日志事件的线程名 
%n   输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n” 
%d   输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss , SSS}，输出类似：2002年10月18日  22 ： 10 ： 28 ， 921  
%l   输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java: 10 ) 
```

日志记录规则：

1. `必须是含义明确的完整语句`：

   推荐格式1：主语 + 谓语 log.info(“The system is in primary state”);

   推荐格式2：动名词 + 宾语 log.debug(“Saving the user information into the database”);

2. 推荐记录业务流程消息

   在业务流程开始和业务流程结束时打印接收和发送出的消息内容，严禁在内部函数内多次打印消息内容；

3. 推荐记录函数关键参数，关键数据结构

4. 推荐记录导致业务错误的异常栈空间

5. 不推荐记录函数出入口

6. 不推荐记录行号

#### 在代码中初始化Logger

1. `在程序中调用BasicConfigurator.configure()方法`：给根记录器增加一个ConsoleAppender，输出格式通过PatternLayout设为"%-4r [%t] %-5p %c %x - %m%n"，还有根记录器的默认级别是Level.DEBUG.
2. 配置放在文件里，`通过命令行参数传递文件名字`，通过PropertyConfigurator.configure(args[x])解析并配置；
3. 配置放在文件里，`通过环境变量传递文件名等信息`，利用log4j默认的初始化过程解析并配置；
4. 配置放在文件里，`通过应用服务器配置传递文件名等信息`，利用一个特殊的servlet来完成配置。

#### 为不同的Appender设置日志输出级别

当调试系统时，我们往往注意的只是异常级别的日志输出，但是通常所有级别的输出都是放在一个文件里的，如果日志输出的级别是DEBUG！？那就慢慢去找吧。

这时我们也许会想要是能把异常信息单独输出到一个文件里该多好啊。当然可以，Log4j已经提供了这样的功能，我们`只需要在配置中修改Appender的Threshold 就能实现`,比如下面的例子：

```
### set log levels ###
log4j.rootLogger = debug ,  stdout ,  D ,  E ,jdbc

### 输出到控制台 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern =  %d{ABSOLUTE} %5p %c{ 1 }:%L - %m%n

### 输出到日志文件 ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = logs/log.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG ## 输出DEBUG级别以上的日志
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 保存异常信息到单独文件 ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File = logs/error.log ## 异常日志文件名
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR ## 只输出ERROR级别以上的日志!!!
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

#数据库
log4j.appender.jdbc=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.jdbc.driver=com.mysql.cj.jdbc.Driver
log4j.appender.jdbc.URL=jdbc:mysql://localhost:3306/learn?useUnicode=true&characterEncoding=UTF-8
log4j.appender.jdbc.user=root
log4j.appender.jdbc.password=970603
log4j.appender.jdbc.sql=INSERT INTO log4j_messages (message, class,priority, log_date) values ('%m', '%c', '%p','%d{yyyy-MM-dd hh:mm:ss}')
log4j.appender.jdbc.layout=org.apache.log4j.PatternLayout


```

Mysql代码：

```
create table log4j_messages
    (
    log_id BIGINT UNSIGNED not null PRIMARY key AUTO_INCREMENT,
    message varchar(2000),
    classtype varchar(255),
    priority varchar(64),
    log_date datetime
    );
```

[代码中使用]：

```
public class TestLog4j {
    public static void main(String[] args) {
        PropertyConfigurator.configure("D:/Code/conf/log4j.properties");
        Logger logger = Logger.getLogger(TestLog4j.class);
        logger.debug("debug");
        logger.error("error");
    }
}
```

log4j中log.isDebugEnabled(), log.isInfoEnabled()和log.isTraceEnabled()作用，项目在应用log4j打印Debug,Info和Trace级别的log时需要加上对应的三个方法进行过滤，代码如下：

```
if (log.isDebugEnabled()) {
    log.debug(" From: " + req.getFrom().toString() + " To: " + req.getTo().toString() + " CallId: " +  req.getCallId() + " msg:" + msg);
}
```

其作用是因为Debug,Info和Trace一般会打印比较详细的信息，而且打印的次数较多，`如果我们不加log.isDebugEnabled()等进行预先判断`，当系统loglevel设置高于Debug或Info或Trace时，虽然系统不会答应出这些级别的日志，但是每次还是会拼接参数字符串，影响系统的性能。

#### 配置文件示例

```
log4j.rootLogger=DEBUG,A1,R
#log4j.rootLogger=INFO,A1,R

# ConsoleAppender 输出
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%c]-[%p] %m%n

# File 输出 一天一个文件,输出路径可以定制,一般在根路径下
log4j.appender.R=org.apache.log4j.DailyRollingFileAppender
log4j.appender.R.File=blog_log.txt
log4j.appender.R.MaxFileSize=500KB
log4j.appender.R.MaxBackupIndex=10
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c] [%p] - %m%n

# 文件配置Sample2
# 下面给出的Log4J配置文件实现了输出到控制台，文件，回滚文件，发送日志邮件，输出到数据库日志表，自定义标签等全套功能。
log4j.rootLogger=DEBUG,CONSOLE,A1,im 
#DEBUG,CONSOLE,FILE,ROLLING_FILE,MAIL,DATABASE
log4j.addivity.org.apache=true

################### 
# Console Appender 
################### 
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender 
log4j.appender.Threshold=DEBUG 
log4j.appender.CONSOLE.Target=System.out 
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout 
log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n 
#log4j.appender.CONSOLE.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[THREAD] n%c[CATEGORY]%n%m[MESSAGE]%n%n

##################### 
# File Appender 
##################### 
log4j.appender.FILE=org.apache.log4j.FileAppender 
log4j.appender.FILE.File=file.log 
log4j.appender.FILE.Append=false 
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout 
log4j.appender.FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n 
# Use this layout for LogFactor 5 analysis

######################## 
# Rolling File 
######################## 
log4j.appender.ROLLING_FILE=org.apache.log4j.RollingFileAppender 
log4j.appender.ROLLING_FILE.Threshold=ERROR 
log4j.appender.ROLLING_FILE.File=rolling.log 
log4j.appender.ROLLING_FILE.Append=true 
log4j.appender.ROLLING_FILE.MaxFileSize=10KB 
log4j.appender.ROLLING_FILE.MaxBackupIndex=1 
log4j.appender.ROLLING_FILE.layout=org.apache.log4j.PatternLayout 
log4j.appender.ROLLING_FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n

#################### 
# Socket Appender 
#################### 
log4j.appender.SOCKET=org.apache.log4j.RollingFileAppender 
log4j.appender.SOCKET.RemoteHost=localhost 
log4j.appender.SOCKET.Port=5001 
log4j.appender.SOCKET.LocationInfo=true 
# Set up for Log Facter 5 
log4j.appender.SOCKET.layout=org.apache.log4j.PatternLayout 
log4j.appender.SOCET.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[THREAD]%n%c[CATEGORY]%n%m[MESSAGE]%n%n

######################## 
# Log Factor 5 Appender 
######################## 
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender 
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000

######################## 
# SMTP Appender 
####################### 
log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender 
log4j.appender.MAIL.Threshold=FATAL 
log4j.appender.MAIL.BufferSize=10 
log4j.appender.MAIL.From=chenyl@yeqiangwei.com
log4j.appender.MAIL.SMTPHost=mail.hollycrm.com 
log4j.appender.MAIL.Subject=Log4J Message 
log4j.appender.MAIL.To=chenyl@yeqiangwei.com
log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout 
log4j.appender.MAIL.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n

######################## 
# JDBC Appender 
####################### 
log4j.appender.DATABASE=org.apache.log4j.jdbc.JDBCAppender 
log4j.appender.DATABASE.URL=jdbc:mysql://localhost:3306/test 
log4j.appender.DATABASE.driver=com.mysql.jdbc.Driver 
log4j.appender.DATABASE.user=root 
log4j.appender.DATABASE.password= 
log4j.appender.DATABASE.sql=INSERT INTO LOG4J (Message) VALUES ('[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n') 
log4j.appender.DATABASE.layout=org.apache.log4j.PatternLayout 
log4j.appender.DATABASE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
log4j.appender.A1=org.apache.log4j.DailyRollingFileAppender 
log4j.appender.A1.File=SampleMessages.log4j 
log4j.appender.A1.DatePattern=yyyyMMdd-HH'.log4j' 
log4j.appender.A1.layout=org.apache.log4j.xml.XMLLayout

################### 
#自定义Appender 
################### 
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net 
log4j.appender.im.username = username 
log4j.appender.im.password = password 
log4j.appender.im.recipient = corlin@yeqiangwei.com
log4j.appender.im.layout=org.apache.log4j.PatternLayout 
log4j.appender.im.layout.ConversionPattern =[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
```

#### XML配置文件示例

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

    <!--
        ConsoleAppender : 输出至控制台
        选项：
            Threshold=WARN:指定日志消息的输出最低层次。
            ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。
            Target=System.err：默认情况下是：System.out,指定输出控制台。
    -->
    <appender name="consoleAppender" class="org.apache.log4j.ConsoleAppender">
        <!-- PatternLayout : 控制日志输出格式 -->
        <!--
            Log4j提供的layout有以下4种：
                1. org.apache.log4j.HTMLLayout(以HTML表格形式布局)
                选项：
                    LocationInfo=true:默认值是false,输出java文件名称和行号。
                    Title=my app file: 默认值是 Log4J Log Messages。
                2. org.apache.log4j.SimpleLayout(包含日志信息的级别和信息字符串)
                3. org.apache.log4j.TTCCLayout(包含日志产生的时间、线程、类别等等信息)
                4. org.apache.log4j.XMLLayout(以XML形式布局)
                选项：
                    LocationInfo=true:默认值是false,输出java文件和行号。
                5. org.apache.log4j.PatternLayout(可以灵活地指定布局模式)
                如果使用PatternLayout布局就要指定的打印信息的具体格式ConversionPattern,打印参数如下：
                    %p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL,
                    %d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
                    %r: 输出自应用启动到输出该log信息耗费的毫秒数
                    %c: 输出日志信息所属的类目，通常就是所在类的全名
                    %t: 输出产生该日志事件的线程名
                    %l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)
                    %x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。
                    %%: 输出一个"%"字符
                    %F: 输出日志消息产生时所在的文件名称
                    %L: 输出代码中的行号
                    %m: 输出代码中指定的消息,产生的日志具体信息
                    %n: 输出一个回车换行符，Windows平台为"/r/n"，Unix平台为"/n"输出日志信息换行
                    可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如：
                    1)%20c：指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，默认的情况下右对齐。
                    2)%-20c:指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，"-"号指定左对齐。
                    3)%.30c:指定输出category的名称，最大的宽度是30，如果category的名称大于30的话，就会将左边多出的字符截掉，但小于30的话也不会有空格。
                    4)%20.30c:如果category的名称小于20就补空格，并且右对齐，如果其名称长于30字符，就从左边交远销出的字符截掉。
        -->
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p:%m %n==> %l%n%n"/>
        </layout>
    </appender>

    <!--
        FileAppender : 输出至指定文件
        选项：
            Threshold=WARN:指定日志消息的输出最低层次。
            ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。
            File=mylog.txt:指定消息输出到mylog.txt文件。
            Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。
    -->
    <appender name="fileAppender" class="org.apache.log4j.FileAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p:%m %n==> %l%n%n"/>
        </layout>

        <!-- 增加日志Filter过滤范围 -->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="LevelMin" value="DEBUG" />
            <param name="LevelMax" value="DEBUG" />
        </filter>
    </appender>

    <!--
        DailyRollingFileAppender : 每天产生一个日志文件
        选项：
            Threshold=WARN:指定日志消息的输出最低层次。
            ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。
            File=mylog.txt:指定消息输出到mylog.txt文件。
            Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。
            DatePattern='.'yyyy-ww:每周滚动一次文件，即每周产生一个新的文件。当然也可以指定按月、周、天、时和分。即对应的格式如下：
                1)'.'yyyy-MM: 每月
                2)'.'yyyy-ww: 每周
                3)'.'yyyy-MM-dd: 每天
                4)'.'yyyy-MM-dd-a: 每天两次
                5)'.'yyyy-MM-dd-HH: 每小时
                6)'.'yyyy-MM-dd-HH-mm: 每分钟
    -->
    <appender name="file" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="/home/taomk/base-rose-web.log"/>
        <param name="DatePattern" value="'_'yyyy-MM-dd'.log'"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p:%m %n==> %l%n%n"/>
        </layout>
    </appender>

    <!--
        RollingFileAppender : 文件到达指定大小时产生一个新文件
        选项：
            Threshold=WARN:指定日志消息的输出最低层次。
            ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。
            File=mylog.txt:指定消息输出到mylog.txt文件。
            Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。
            MaxFileSize=100KB:后缀可以是KB, MB 或者是 GB. 在日志文件到达该大小时，将会自动滚动，即将原来的内容移到mylog.log.1文件。
            MaxBackupIndex=2:指定可以产生的滚动文件的最大数。
    -->
    <appender name="file" class="org.apache.log4j.RollingFileAppender">
        <param name="File" value="/home/taomk/base-rose-web.log"/>
        <param name="DatePattern" value="'_'yyyy-MM-dd'.log'"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p:%m %n==> %l%n%n"/>
        </layout>
    </appender>

    <!-- WriterAppender : 将日志信息以流格式发送到任何地方 -->
    <appender name="file" class="org.apache.log4j.WriterAppender">
        <param name="File" value="/home/taomk/base-rose-web.log"/>
        <param name="DatePattern" value="'_'yyyy-MM-dd'.log'"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p:%m %n==> %l%n%n"/>
        </layout>
    </appender>

    <logger name="org.apache.ibatis">
        <!-- level优先级分别为FATAL、ERROR、WARN、INFO、DEBUG 5个级别(由高到低) -->
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <logger name="org.mybatis.spring">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <logger name="java.sql">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <logger name="java.sql.Connection">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <logger name="java.sql.Statement">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <logger name="java.sql.PreparedStatement">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <!--
        additivity：它是 子Logger 是否继承 父Logger 的 输出源（appender） 的标志位。
        具体说，默认情况下子Logger会继承父Logger的appender，也就是说子Logger会在父Logger的appender里输出。
        若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出。
    -->
    <logger name="org.springframework" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="file"/>
    </logger>

    <root>
        <level value="INFO"/>
        <!-- 优先级 -->
        <priority value="debug"/>
        <appender-ref ref="file"/>
        <appender-ref ref="stdout"/>
    </root>
</log4j:configuration>
```

#### 高级使用

**发送email通知管理员**：

1.首先下载JavaMail和JAF,

```
http://java.sun.com/j2ee/ja/javamail/index.html
http://java.sun.com/beans/glasgow/jaf.html
```

在项目中引用mail.jar和activation.jar。

2.写配置文件

```
# 将日志发送到email
log4j.logger.MailLog=WARN,A5
#  APPENDER A5
log4j.appender.A5=org.apache.log4j.net.SMTPAppender
log4j.appender.A5.BufferSize=5
log4j.appender.A5.To=chunjie@yeqiangwei.com
log4j.appender.A5.From=error@yeqiangwei.com
log4j.appender.A5.Subject=ErrorLog
log4j.appender.A5.SMTPHost=smtp.263.net
log4j.appender.A5.layout=org.apache.log4j.PatternLayout
log4j.appender.A5.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n
```

3.调用代码：

```
//把日志发送到mail
Logger logger3 = Logger.getLogger("MailLog");
logger3.warn("warn!!!");
logger3.error("error!!!");
logger3.fatal("fatal!!!");
```

**在后台输出所有类别的错误**：

1.写配置文件

```
# 在后台输出
log4j.logger.console=DEBUG, A1
# APPENDER A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n
```

2.调用代码

```
Logger logger1 = Logger.getLogger("console");
logger1.debug("debug!!!");
logger1.info("info!!!");
logger1.warn("warn!!!");
logger1.error("error!!!");
logger1.fatal("fatal!!!");
```

#### 日志文件路径分析

**方法一：解决的办法自然是用相对路径代替绝对路径，其实log4j的FileAppender本身就有这样的机制，如**：

```
log4j.appender.logfile.File=${WORKDIR}/logs/app.log
```

`其中“${WORKDIR}/”是个变量，会被System Property中的“WORKDIR”的值代替`。这样，我们就可以在log4j加载配置文件之前，先用System.setProperty ("WORKDIR", WORKDIR);设置好根路径，此操作可通过一初始的servlet进行。

**方法二：可以使用服务器环境变量，log4j的配置文件支持服务器的vm的环境变量，格式类似${catalina.home}**

```
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=${catalina.home}/logs/logs_tomcat.log
log4j.appender.R.MaxFileSize=10KB
```

其中的${catalina.home}并非windows系统的环境变量，这个环境变量就不需要在Windows系统的环境变量中设置。之所以这样，`你可以看看tomcat/bin/catalina.bat(startup,shutdown都是调用这个)里面自带有-Dcatalina.home= "%CATALINA_HOME%"` 。继承这个思想，所以你也可以自己设定一个参数-Dmylog.home="D:/abc/log"到对应的服务器java启动的vm参数中。

**方法三：通过servlet初始化init()方法中加载file属性实现相对路径**

具体实现:做一个servlet,在系统加载的时候,就`把properties的文件读到一个properties文件中.那个file的属性值(我使用的是相对目录)改掉(前面加上系统的根目录),让后把这个properties对象设置到propertyConfig中去`,这样就初始化了log的设置.在后面的使用中就用不着再配置了。

一般在我们开发项目过程中,`log4j日志输出路径固定到某个文件夹`,这样如果我换一个环境,日志路径又需要重新修改,比较不方便,目前我采用了动态改变日志路径方法来实现相对路径保存日志文件。

```
public class Log4jInit extends HttpServlet {

	private static Logger logger = Logger.getLogger(Log4jInit.class);

	public Log4jInit() {
	}

	public void init(ServletConfig config) throws ServletException {
		String prefix = config.getServletContext().getRealPath("/");
		String file = config.getInitParameter("log4j");
		String filePath = prefix + file;
		Properties props = new Properties();
		try {
			FileInputStream istream = new FileInputStream(filePath);
			props.load(istream);
			istream.close();
			// toPrint(props.getProperty("log4j.appender.file.File"));
			String logFile = prefix
					+ props.getProperty("log4j.appender.file.File");// 设置路径
			props.setProperty("log4j.appender.file.File", logFile);
			PropertyConfigurator.configure(props);// 装入log4j配置信息
		} catch (IOException e) {
			toPrint("Could not read configuration file [" + filePath + "].");
			toPrint("Ignoring configuration file [" + filePath + "].");
			return;
		}
	}

	public static void toPrint(String content) {
		System.out.println(content);
	}
}
```

Web.xml配置：

```
<servlet>
    <servlet-name>log4j-init</servlet-name>
    <servlet-class>Log4jInit</servlet-class>
    <init-param>
        <param-name>log4j</param-name>
	<param-value>WEB-INF/classes/log4j.properties</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

```

注意：`上面的load-on-startup设为0，以便在Web容器启动时即装入该Servlet`。log4j.properties文件放在根的properties子目录中，也可以把它放在其它目录中。应该把.properties文件集中存放，这样方便管理。

### Log4j性能优化

1. log4j已成为大型系统必不可少的一部分，log4j可以很方便的帮助我们在程序的任何位置输出所要打印的信息，便于我们对系统在调试阶段和正式运行阶段对问题分析和定位。由于日志级别的不同，对系统的性能影响也是有很大的差距，`日志级别越高，性能越高`。
2. log4j对系统性能的影响程度主要体现在以下几方面：
3. 日志输出的目的地，`输出到控制台的速度比输出到文件系统的速度要慢`。
4. 日志`输出格式不一样对性能也会有影响`，如简单输出布局(SimpleLayout)比格式化输出布局(PatternLayout)输出速度要快。可以根据需要尽量采用简单输出布局格式输出日志信息。
5. 日志`级别越低输出的日志内容就越多，对系统系能影响很大`。
6. 日志输出方式的不同，对系统系能也是有一定影响的，`采用异步输出方式比同步输出方式性能要高`。
7. 每次接收到日志输出事件就`打印一条日志内容比当日志内容达到一定大小时打印系能要低`。
8. 针对以上几点对系能的影响中的第4,5点，对日志配置文件做如下配置：
9. 设置日志缓存，以及缓存大小

```
log4j.appender.A3.BufferedIO=true
#Buffer单位为字节，默认是8K，IO BLOCK大小默认也是8K
log4j.appender.A3.BufferSize=8192
```

以上配置说明，当日志内容达到8k时，才会将日志输出到日志输出目的地。

10. 设置日志输出为异步方式

```
    <appender name="DRFOUT" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="logs/brws.log" />
        <param name="Append" value="true" />
        <param name="DatePattern" value="yyyy_MM_dd'.'" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d [%t] %-5p %l %x - %m%n" />
        </layout>
    </appender>
    <appender name="ASYNCOUT" class="org.apache.log4j.AsyncAppender">
        <param name="BufferSize" value="512" />
        <appender-ref ref="DRFOUT" />
    </appender>
```

- **同步情况**：各线程`直接获得输出流进行输出(线程间不需要同步)`。
- **异步情况**：

1. 各线程将`日志写到缓存，继续执行下面的任务`(这里是异步的)。
2. 日志线程发现需要`记日志时独占缓存(与此同时各线程等待，此时各线程是被阻塞住的)`，从缓存中取出日志信息，获得输出流进行输出，将缓存解锁(各线程收到提醒，可以接着写日志了)。

众所周知，`磁盘IO操作、网络IO操作、JDBC操作等都是非常耗时的，日志输出的主要性能瓶颈也就是在写文件、写网络、写JDBC的时候`。日志是肯定要记的，而要采用异步方式记，也就`只有将这些耗时操作从主线程当中分离出去才真正的实现性能提升，也只有在线程间同步开销小于耗时操作时使用异步方式才真正有效`！

现在我们接着分别来看看这几种记录日志的方式：

1. 将`日志记录到本地文件` 同样都是写本地文件Log4j本身有一个buffer处理入库，采用异步方式并不一定能提高性能(主要是如何配置好缓存大小)；而`线程间的同步开销则是非常大的`！因此在使用本地文件记录日志时不建议使用异步方式。
2. 将`日志记录到JMS JMS本身是支持异步消息的`，如果不考虑JMS消息创建的开销，也不建议使用异步方式。
3. 将`日子记录到SOCKET 将日志通过Socket发送`，纯网络IO操作不需要反馈，因此也不会耗时。
4. 将`日志记录到数据库` 众所周知JDBC是几种方式中最耗时的：网络、磁盘、数据库事务，`都使JDBC操作异常的耗时`，在这里采用异步方式入库倒是一个不错的选择。
5. 将`日志记录到SMTP 同JDBC`。

异步输出日志工作原理：AsyncAppender采用的是`生产者消费者的模型`进行异步地`将Logging Event送到对应的Appender中`。

1. 生产者：外部应用了`Log4j的系统的实时线程`，实时将`Logging Event传送进AsyncAppender里`。
2. 中转：`Buffer和DiscardSummary`。
3. 消费者：`Dispatcher线程和appenders`。

工作原理：

1） Logging Event进入AsyncAppender，AsyncAppender会调用append方法，`在append方法中会去把logging Event填入Buffer中`，当消费能力不如生产能力时，AsyncAppender会把超出Buffer容量的Logging Event放到DiscardSummary中，作为消费速度一旦跟不上生成速度，中转buffer的溢出处理的一种方案。

2） `AsyncAppender有个线程类Dispatcher`，它是一个简单的线程类，实现了Runnable接口。它是AsyncAppender的后台线程。

Dispatcher所要做的工作是：

1. `锁定Buffer`，让其他要对Buffer进行操作的线程阻塞。

2.  `看Buffer的容量是否满了`，如果满了就将Buffer中的Logging Event全部取出，并清空Buffer和DiscardSummary；如果没满则等待Buffer填满Logging Event，然后notify Disaptcher线程。

3.  将取出的`所有Logging Event交给对应appender`进行后面的日志信息推送。

以上是AsyncAppender类的两个关键点：**append方法和Dispatcher类，通过这两个关键点实现了异步推送日志信息的功能**，这样如果大量的Logging Event进入AsyncAppender，就可以游刃有余地处理这些日志信息了。

#### Spring的Log4jConfigListener使用

`使用spring中的Log4jConfigListener可以随时调整打印日志的级别而不用重启服务`，在web.xml中添加如下内容：

```
<!--如果不定义webAppRootKey参数，那么webAppRootKey就是缺省的"webapp.root"。但最好设置，以免项目之间的名称冲突。
    定义以后，在Web Container启动时将把ROOT的绝对路径写到系统变量里。
    然后log4j的配置文件里就可以用${webName.root}来表示Web目录的绝对路径，把log文件存放于webapp中。
    如果该变量不存在，会存储在项目所在根盘符中，默认为webapp.root。
    此参数用于后面的“Log4jConfigListener”-->
<context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>webName.root</param-value>
</context-param>

<!--由Sprng载入的Log4j配置文件位置-->
<context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>/WEB-INF/log4j.properties</param-value>
</context-param>

<!--Spring默认刷新Log4j配置文件的间隔,单位为millisecond-->
<context-param>
    <param-name>log4jRefreshInterval</param-name>
    <param-value>60000</param-value>
</context-param>

<!-- Web 项目 Spring 加载 Log4j 的监听 -->
<listener>
    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
</listener>
```

### 参考

[Log4j架构分析与实战](https://my.oschina.net/xianggao/blog/518059)

