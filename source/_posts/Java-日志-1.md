---
title: 日志框架--JUL
date: 2019-05-12 23:18:59
tags:
 - Java
 - 日志
categories:
 - Java
 - 日志
---

### [JUL](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)

JDK Logging的使用很简单，如下代码所示，先使用Logger类的静态方法getLogger就可以获取到一个logger，然后在任何地方都可以通过获取到的logger进行日志输入。比如类似logger.info("Main running.")的调用。

<!--more-->

```
import java.util.logging.Level;
import java.util.logging.Logger;
 
public class LoggerTest {
      private static Logger logger = Logger.getLogger("com.bes.logging");
      public static void main(String argv[]) {
               // Log a FINEtracing message
               logger.info("Main running.");
               logger.fine("doingstuff");
               try {
                         Thread.currentThread().sleep(1000);// do some work
               } catch(Exception ex) {
                         logger.log(Level.WARNING,"trouble sneezing", ex);
               }
               logger.fine("done");
      }
}
```

其中的Logger是：java.util.logging.Logger 

不做任何代码修改和JDK配置修改的话，运行上面的例子，你会发现，控制台只会出现【Main running.】这一句日志。如下问题应该呈现在你的大脑里…

1，【Main running.】以外的日志为什么没有输出？怎么让它们也能够出现？

2，日志中出现的时间、类名、方法名等是从哪里输出的？

3，为什么日志就会出现在控制台？

4，大型的系统可能有很多子模块(可简单理解为有很多包名)，如何对这些子模块进行单独的日志级别控制？

5，扩充：apache那个流行的log4j项目和JDK的logging有联系吗，怎么实现自己的LoggerManager？

带着这些问题，可能你更有兴趣了解一下JDK的logging机制，本章为你分析这个简单模块的机制。

#### 组件

**logger**

对于logger，需要知道其下几个方面

1. 代码需要输入日志的地方都会用到Logger，这几乎是一个JDK logging模块的代言人，我们常常用Logger.getLogger("com.aaa.bbb");获得一个logger，然后使用logger做日志的输出。

2. logger其实只是一个逻辑管理单元，其多数操作都只是作为一个中继者传递别的<角色>，比如说：Logger.getLogger(“xxx”)的调用将会依赖于LogManager类，使用logger输入日志信息的时候会调用logger中的所有handler进行日志的输入。

3. logger是有层次关系的，我们可一般性的理解为包名之间的父子继承关系。每个logger通常以java包名为其名称。子logger通常会从父logger继承logger级别、handler、ResourceBundle名(与国际化信息有关)等。

4. 整个JVM会存在一个名称为空的root logger，所有匿名的logger都会把root logger作为其父

 **LogManager**

整个JVM内部所有logger的管理，logger的生成、获取等操作都依赖于它，也包括配置文件的读取。LogManager中会有一个Hashtable【private Hashtable<String,WeakReference<Logger\>> loggers】用于存储目前所有的logger，如果需要获取logger的时候，Hashtable已经有存在logger的话就直接返回Hashtable中的，如果hashtable中没有logger，则新建一个同时放入Hashtable进行保存。 

**Handler**

用来控制日志输出的，比如JDK自带的ConsoleHanlder把输出流重定向到System.err输出，每次调用Logger的方法进行输出时都会调用Handler的publish方法，每个logger有多个handler。我们可以利用handler来把日志输入到不同的地方(比如文件系统或者是远程Socket连接)。目前有

- FileHandler：输出到文件，默认Level为INFO，Formatter为XMLFormatter；；
- ConsoleHandler：输出到控制台，默认Level为INFO，流为System.err，Formatter为SimpleFormatter；
- SocketHandler：输出到socket；
- MemoryHandler：输出到内存；
- 可自定义日志级别，继承java.util.logging.Formatter类即可。

**Formatter**

日志在真正输出前需要进行一定的格式话：比如是否输出时间？时间格式？是否输入线程名？是否使用国际化信息等都依赖于Formatter。 目前有SimpleFormatter和XMLFormatter两种格式，分别对应简单格式和xml格式。可自定输出格式，继承抽象类java.util.logging.Formatter即可。

**Log Level**

不必说，这是做容易理解的一个，也是logging为什么能帮助我们适应从开发调试到部署上线等不同阶段对日志输出粒度的不同需求。JDK Log级别从高到低为OFF(2^31-1)—>SEVERE(1000)—>WARNING(900)—>INFO(800)—>CONFIG(700)—>FINE(500)—>FINER(400)—>FINEST(300)—>ALL(-2^31)，每个级别分别对应一个数字，输出日志时级别的比较就依赖于数字大小的比较。但是需要注意的是：不仅是logger具有级别，handler也是有级别，也就是说如果某个logger级别是FINE，客户希望输入FINE级别的日志，如果此时logger对应的handler级别为INFO，那么FINE级别日志仍然是不能输出的。

示例：

```
public class JavaLogStudyMain {
    public static void main(String[] args) throws Exception {
        //简单格式的Formatter
        SimpleFormatter sf = new SimpleFormatter();
        //xml格式的formatter
        XMLFormatter xf = new XMLFormatter();
 
        //输出到文件的hander,指定日志输出级别为ALL
        FileHandler fh = new FileHandler("jul_study.log");
        fh.setLevel(Level.ALL);
        fh.setFormatter(sf);
//        fh.setFormatter(xf);
 
        //输出到控制台的handler,指定日志输出级别为INFO
        ConsoleHandler ch = new ConsoleHandler();
        ch.setLevel(Level.INFO);
 
        //获取logger实例，相同名的只能生成一个实例
        Logger logger = Logger.getLogger("javaLog");
        logger.addHandler(fh);   //添加输出handler
        logger.addHandler(ch);   //添加输出handler
        logger.setLevel(Level.ALL);  //指定logger日志级别
 
        //日志输出简写形式，有不同的级别，可带参数，其它类似
        logger.log(Level.INFO, "this is a info, {0}", "p1");
        logger.log(Level.WARNING, "this is a warn, {0} {1}", new Object[]{"p1", "p2"});
        logger.log(Level.SEVERE, "this is a severe");
        logger.log(Level.FINE, "this is a fine");
        logger.log(Level.FINEST, "this is a finest");
        logger.log(Level.CONFIG, "this is a config");
        logger.log(Level.INFO, "this is a info", new Exception("my excep")); //带异常输出
 
        //日志输出简写形式，有不同的级别
        logger.severe("severe log");
        logger.warning("warning log");
        logger.info("info log");
        logger.config("config log");
        logger.fine("fine log");
        logger.finer("finer log");
        logger.finest("finest log");
    }
}
```

 **总结**：

1. LogManager与logger是1对多关系，整个JVM运行时只有一个LogManager，且所有的logger均在LogManager中

2. logger与handler是多对多关系，logger在进行日志输出的时候会调用所有的hanlder进行日志的处理

3. handler与formatter是一对一关系，一个handler有一个formatter进行日志的格式化处理

很明显：logger与level是一对一关系，hanlder与level也是一对一关系 

#### 配置

JDK默认的logging配置文件为：$JAVA_HOME/jre/lib/logging.properties，可以使用系统属性java.util.logging.config.file指定相应的配置文件对默认的配置文件进行覆盖，配置文件中通常包含以下几部分定义：

1.  handlers：用逗号分隔每个Handler，这些handler将会被加到root logger中。也就是说即使我们不给其他logger配置handler属性，在输出日志的时候logger会一直找到root logger，从而找到handler进行日志的输入。

2.  .level是root logger的日志级别

3. <handler\>.xxx是配置具体某个handler的属性，比如java.util.logging.ConsoleHandler.formatter便是为ConsoleHandler配置相应的日志Formatter.

4. logger的配置，所有以[.level]结尾的属性皆被认为是对某个logger的级别的定义，如`com.bes.server.level=FINE`是给名为`[com.bes.server]`的logger定义级别为FINE。顺便说下，前边提到过logger的继承关系，如果还有`com.bes.server.webcontainer`这个logger，且在配置文件中没有定义该logger的任何属性，那么其将会从[com.bes.server]这个logger进行属性继承。除了级别之外，还可以为logger定义handler和useParentHandlers(默认是为true)属性，如`com.bes.server.handler=com.bes.test.ServerFileHandler`(需要是一个extends java.util.logging.Handler的类)，`com.bes.server.useParentHandlers=false`(意味着com.bes.server这个logger进行日志输出时，日志仅仅被处理一次，用自己的handler输出，不会传递到父logger的handler)。以下是JDK配置文件示例

   ```
   ############################################################
   # Handler specific properties.
   # Describes specific configuration info for Handlers.
   ############################################################
   
   # default file output is in user's home directory.
   java.util.logging.FileHandler.pattern = %h/java%u.log
   java.util.logging.FileHandler.limit = 50000
   java.util.logging.FileHandler.count = 1
   java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter
   
   # Limit the message that are printed on the console to INFO and above.
   java.util.logging.ConsoleHandler.level = INFO
   java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter
   
   # Example to customize the SimpleFormatter output format 
   # to print one-line log message like this:
   #     <level>: <log message> [<date/time>]
   #
   # java.util.logging.SimpleFormatter.format=%4$s: %5$s [%1$tc]%n
   
   ############################################################
   # Facility specific properties.
   # Provides extra control for each logger.
   ############################################################
   
   # For example, set the com.xyz.foo logger to only log SEVERE
   # messages:
   com.xyz.foo.level = SEVERE
   ```

#### 简单过程分析

##### Logger的获取

1. 首先是调用Logger的如下方法获得一个logger

```
    public static synchronized Logger getLogger(String name) {
           LogManager manager =LogManager.getLogManager();
        returnmanager.demandLogger(name);
    }
```

2. 上面的调用会触发java.util.logging.LoggerManager的类初始化工作，LoggerManager有一个静态化初始化块(这是会先于LoggerManager的构造函数调用的~_~)：

```
static {
   AccessController.doPrivileged(newPrivilegedAction<Object>() {
       public Object run() {
           String cname =null;
           try {
               cname =System.getProperty("java.util.logging.manager");
               if (cname !=null) {
                  try {
                       Class clz =ClassLoader.getSystemClassLoader().loadClass(cname);
                       manager= (LogManager) clz.newInstance();
                   } catch(ClassNotFoundException ex) {
               Class clz =Thread.currentThread().getContextClassLoader().loadClass(cname);
                      manager= (LogManager) clz.newInstance();
                   }
               }
           } catch (Exceptionex) {
              System.err.println("Could not load Logmanager \"" + cname+ "\"");
              ex.printStackTrace();
           }
           if (manager ==null) {
               manager = newLogManager();
           }
      
           manager.rootLogger= manager.new RootLogger();
          manager.addLogger(manager.rootLogger);
 
           Logger.global.setLogManager(manager);
          manager.addLogger(Logger.global);
 
           return null;
       }
   });
}
```

从静态初始化块中可以看出LoggerManager是可以使用系统属性java.util.logging.manager指定一个继承自java.util.logging.LoggerManager的类进行替换的，比如Tomcat启动脚本中就使用该机制以使用自己的LoggerManager。

不管是JDK默认的java.util.logging.LoggerManager还是自定义的LoggerManager，初始化工作中均会给LoggerManager添加两个logger，一个是名称为””的root logger，且logger级别设置为默认的INFO；另一个是名称为global的全局logger，级别仍然为INFO。

LogManager类初始化完成之后就会读取配置文件(默认为$JAVA_HOME/jre/lib/logging.properties)，把配置文件的属性名<->属性值这样的键值对保存在内存中，方便之后初始化logger的时候使用。

3. A步骤中Logger类发起的getLogger操作将会调用java.util.logging.LoggerManager的如下方法：

```
     Logger demandLogger(String name) {
       Logger result =getLogger(name);
       if (result == null) {
           result = newLogger(name, null);
           addLogger(result);
           result =getLogger(name);
       }
       return result;
     }
```

可以看出，LoggerManager首先从现有的logger列表中查找，如果找不到的话，会新建一个looger并加入到列表中。当然很重要的是新建looger之后需要对logger进行初始化，这个初始化详见`java.util.logging.LoggerManager#addLogger()`方法中，改方法会根据配置文件设置logger的级别以及给logger添加handler等操作。

到此为止logger已经获取到了，你同时也需要知道此时你的logger中已经有级别、handler等重要信息，下面将分析输出日志时的逻辑。 

##### 日志的输出

首先我们通常会调用Logger类下面的方法，传入日志级别以及日志内容。

```
    public void log(Levellevel, String msg) {
          if (level.intValue() < levelValue ||levelValue == offValue) {
              return;
          }
          LogRecord lr = new LogRecord(level, msg);
          doLog(lr);
    }
```

该方法可以看出，Logger类首先是进行级别的校验，如果级别校验通过，则会新建一个LogRecord对象，LogRecord中除了日志级别，日志内容之外还会包含调用线程信息，日志时刻等；之后调用doLog(LogRecord lr)方法

```
    private void doLog(LogRecord lr) {
          lr.setLoggerName(name);
          String ebname =getEffectiveResourceBundleName();
          if (ebname != null) {
              lr.setResourceBundleName(ebname);
              lr.setResourceBundle(findResourceBundle(ebname));
          }
          log(lr);
    }
```

doLog(LogRecord lr)方法中设置了ResourceBundle信息(这个与国际化有关)之后便直接调用log(LogRecord record) 方法 

```
   public void log(LogRecord record) {
          if (record.getLevel().intValue() <levelValue || levelValue == offValue) {
              return;
          }
          synchronized (this) {
              if (filter != null &&!filter.isLoggable(record)) {
                  return;
              }
          }
          Logger logger = this;
          while (logger != null) {
              Handler targets[] = logger.getHandlers();
 
              if(targets != null) {
                  for (int i = 0; i < targets.length; i++){
                           targets[i].publish(record);
                       }
              }
 
              if(!logger.getUseParentHandlers()) {
                       break;
              }
 
              logger= logger.getParent();
          }
    }
```

很清晰，while循环是重中之重，首先从logger中获取handler，然后分别调用handler的publish(LogRecordrecord)方法。while循环证明了前面提到的会一直把日志委托给父logger处理的说法，当然也证明了可以使用logger的useParentHandlers属性控制日志不进行往上层logger传递的说法。到此为止logger对日志的控制差不多算是完成，接下来的工作就是看handler的了，这里我们以java.util.logging.ConsoleHandler为例说明日志的输出。

```
public class ConsoleHandler extends StreamHandler {
    public ConsoleHandler() {
          sealed = false;
          configure();
          setOutputStream(System.err);
          sealed = true;
    }
```

ConsoleHandler构造函数中除了需要调用自身的configure()方法进行级别、filter、formatter等的设置之外，最重要的我们最关心的是setOutputStream(System.err)这一句，把系统错误流作为其输出。而ConsoleHandler的publish(LogRecordrecord)是继承自java.util.logging.StreamHandler的，如下所示：   

```
 public synchronized void publish(LogRecord record) {
       if(!isLoggable(record)) {
           return;
       }
       String msg;
       try {
           msg =getFormatter().format(record);
       } catch (Exception ex){
           // We don't want tothrow an exception here, but we
           // report theexception to any registered ErrorManager.
           reportError(null,ex, ErrorManager.FORMAT_FAILURE);
           return;
       }
       
       try {
           if (!doneHeader) {
              writer.write(getFormatter().getHead(this));
               doneHeader =true;
           }
           writer.write(msg);
       } catch (Exception ex){
           // We don't want tothrow an exception here, but we
           // report theexception to any registered ErrorManager.
           reportError(null,ex, ErrorManager.WRITE_FAILURE);
       }
    }
```

方法逻辑也很清晰，首先是调用Formatter对消息进行格式化，说明一下：格式化其实是进行国际化处理的重要契机。然后直接把消息输出到对应的输出流中。需要注意的是handler也会用自己的level和LogRecord中的level进行比较，看是否真正输出日志。

#### 自定义logger

这里通过继承Level、Handler、Formatter自定义Logger，日志存在文件中，且每个不同的loggerName存不同日志文件中。

```
public class SelfJavaLogMain {
    //生成单例日志输出handler实例
    private static SelfHandle sf = new SelfHandle();
 
    /**
     * 根据class获取logger
     * @param clazz
     * @return
     */
    public static Logger getLogger(Class clazz){
        return getLogger(clazz.getSimpleName());
    }
 
    /**
     * 根据loggerName获取logger
     * @param loggerName
     * @return
     */
    public static Logger getLogger(String loggerName){
        Logger logger = Logger.getLogger(loggerName);
        logger.addHandler(sf);
        return logger;
    }
 
    /**
     * 自定义日志格式formatter
     */
    public static class SelfFormater extends Formatter {
 
        public static SelfFormater selfFormater = new SelfFormater();
 
        @Override
        public String format(LogRecord record) {
            return String.format("[%s] %s %s.%s: %s\r\n", DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"),
                    record.getLevel().getName(), record.getSourceClassName(), record.getSourceMethodName(), record.getMessage());
        }
 
        public static SelfFormater getInstance() {
            return selfFormater;
        }
    }
 
    /**
     * 自定义日志level
     */
    public static class SelfLevel extends Level {
 
        public static SelfLevel ERROR = new SelfLevel("ERROR", 910);
 
        /**
         * @param name  自定义级别名称
         * @param value 自定义级别的权重值
         */
        protected SelfLevel(String name, int value) {
            super(name, value);
        }
    }
 
    /**
     * 自定义日志输出handler
     */
    public static class SelfHandle extends Handler {
 
        /**
         * 日志写入的位置
         *
         * @param record
         */
        @Override
        public void publish(LogRecord record) {
            try {
                //每一个不同的loggerName分别记在不同的日志文件中
                File file = new File(String.format("%s_%s.log",record.getLoggerName(), DateFormatUtils.format(new Date(),"yyyy-MM-dd")));
                if(!file.exists()){
                    file.createNewFile();
                }
                FileWriter fw = new FileWriter(file,true);
                PrintWriter pw = new PrintWriter(fw);
                String log = String.format("[%s] %s %s.%s: %s\r\n", DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"),
                        record.getLevel().getName(), record.getSourceClassName(), record.getSourceMethodName(), record.getMessage());
                pw.write(log);
                pw.flush();
            } catch (IOException e) {
                e.printStackTrace();
            }
 
        }
 
        /**
         * 刷新缓存区，保存数据
         */
        @Override
        public void flush() {
        }
 
        /**
         * 关闭
         *
         * @throws SecurityException
         */
        @Override
        public void close() throws SecurityException {
        }
    }
 
    public static void main(String[] args) throws Exception {
        //获取自定义的logger
        Logger logger = getLogger(SelfJavaLogMain.class);
        logger.info("info log");
        logger.severe("severe log");
        //使用自定义的logger level
        logger.log(SelfLevel.ERROR, "error log");
    }
}
```

#### 总结

至此，整个日志输出过程已经分析完成。细心的读者应该可以解答如下四个问题了。

1. 【Main running.】以外的日志为什么没有输出？怎么让它们也能够出现？

   这就是JDK默认的logging.properties文件中配置的handler级别和跟级别均为info导致的，如果希望看到FINE级别日志，需要修改logging.properties文件,同时进行如下两个修改

```
java.util.logging.ConsoleHandler.level= FINE//修改
com.bes.logging.level=FINE//添加
```

2. 日志中出现的时间、类名、方法名等是从哪里输出的？

   请参照`[java.util.logging.ConsoleHandler.formatter= java.util.logging.SimpleFormatter]`配置中指定的java.util.logging.SimpleFormatter类，其`publicsynchronized String format(LogRecord record)` 方法说明了一切。

```
public synchronized String format(LogRecord record) {
    StringBuffer sb = new StringBuffer();
    // Minimize memory allocations here.
    dat.setTime(record.getMillis());
    args[0] = dat;
    StringBuffer text = new StringBuffer();
    if (formatter == null) {
        formatter = new MessageFormat(format);
    }
    formatter.format(args, text, null);
    sb.append(text);
    sb.append(" ");
    if (record.getSourceClassName() != null) {     
        sb.append(record.getSourceClassName());
    } else {
        sb.append(record.getLoggerName());
    }
    if (record.getSourceMethodName() != null) {
        sb.append(" ");
       sb.append(record.getSourceMethodName());
    }
    sb.append(lineSeparator);
    String message = formatMessage(record);
   sb.append(record.getLevel().getLocalizedName());
    sb.append(": ");
    sb.append(message);
    sb.append(lineSeparator);
    if (record.getThrown() != null) {
        try {
            StringWriter sw = newStringWriter();
            PrintWriter pw = newPrintWriter(sw);
           record.getThrown().printStackTrace(pw);
            pw.close();
             sb.append(sw.toString());
        } catch (Exception ex) {
        }
    }
    return sb.toString();
}
```

3. 为什么日志就会出现在控制台？

   看到java.util.logging.ConsoleHandler 类构造方法中的[setOutputStream(System.err)]语句，相信你已经明白。

4. 大型的系统可能有很多子模块(可简单理解为有很多包名)，如何对这些子模块进行单独的日志级别控制？

   在logging.properties文件中分别对各个logger的级别进行定义，且最好使用java.util.logging.config.file属性指定自己的配置文件。

5. 扩充：apache那个流行的log4j项目和JDK的logging有联系吗，怎么实现自己的LoggerManager？

   没联系，两个都是日志框架具体实现，两者的LoggerManager实现逻辑大体一致，只是具体细节不同。



### 参考

1. [JDK Logging深入分析](https://blog.csdn.net/qingkangxu/article/details/7514770)