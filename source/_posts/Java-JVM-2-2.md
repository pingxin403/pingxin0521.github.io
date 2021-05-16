---
title: jvm 监控工具
date: 2019-11-01 19:18:59
tags:
 - Java
categories:
 - Java
 - JVM
---

在实际工作中，在进行jvm调优或者分析内存泄露、溢出等问题时，熟练掌握JVM常用的监控工具能够帮助更快地定位问题所在，目前记录一下使用过的常用的jvm监控工具以及其使用、和对应分析过程。

<!--more-->

### 常用命令

#### 查看进程线程的方法

1. windows
   - 任务管理器可以查看进程和线程数,也可以用来杀死进程
   - tasklist 查看进程
   - taskkill 杀死进程

2. linux
   - `ps -fe` 查看所有进程
   - `ps -fT -p <PID>` 查看某个进程(PID)的所有线程
   - `kill` 杀死进程
   - `top` 按大写 H 切换是否显示线程
   - `top -H -p <PID>` 查看某个进程(PID)的所有线程

#### 查看jvm使用的垃圾回收器

在开始了jvm调优或者内存问题分析时，我们首先要了解的就是JVM使用垃圾回收器是什么，才能结合监控工具，做出正确的结果分析。那么如何查看JVM运行时使用的垃圾回收器是什么？

如果在程序的jvm启动参数中没有指定使用什么垃圾回收器，我们可以直接在命令行执行以下命令

```
java -XX:+PrintCommandLineFlags -version
```

得到以下输出，此命令默认打印出jvm运行使用的参数以及jvm版本信息，默认情况下不指定垃圾回收器类型的话，从虚拟机参数是没办法知道垃圾回收器类型的。

```
-XX:InitialHeapSize=393148480 -XX:MaxHeapSize=6290375680 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC 
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (build 1.8.0_252-8u252-b09-1~18.04-b09)
OpenJDK 64-Bit Server VM (build 25.252-b09, mixed mode)
```

### Java的命令行工具

**jps（JVM Process Status Tools）**

jps是参照Unix系统的取名规则命名的，而他的功能和ps的功能类似，可以列举正在运行的饿虚拟机进程并显示虚拟机执行的主类以及这些进程的唯一ID（LVMID，对应本机来说和PID相同），他的用法如下：

```
jps [option] [hostid]
```

其中hostid默认为本机，而option选项包含以下选项

| Option | Function                                 |
| ------ | ---------------------------------------- |
| -q     | 只输出LVMID                              |
| -m     | 输出JVM启动时传给主类的方法              |
| -l     | 输出主类的全名，如果是Jar则输出jar的路径 |
| -v     | 输出JVM的启动参数                        |

**jstat（JVM Statistics Monitoring Tools）**

jstat主要用于监控虚拟机的各种运行状态信息，如类的装载、内存、垃圾回收、JIT编译器等，在没有GUI的服务器上，这款工具是首选的一款监控工具。其用法如下：

```
jstat [option vmid [interval [s|ms] [vount] ] ]
```

参数interval和count分别表示查询间隔和查询次数，如每1毫秒查询一次进程20445的垃圾回收情况，监控20次，命令如下所示：

```
jstat –gc 20445 1 20
```

相关的输出参数介绍可参照官方的[说明](http://docs.oracle.com/javase/1.5.0/docs/tooldocs/share/jstat.html)

选项option代表用户需要查询的虚拟机的信息，主要分为3类：类装载、垃圾回收和运行期的编译情况，具体如下表所示：

| Option            | Function                                                     |
| ----------------- | ------------------------------------------------------------ |
| -class            | 监视类的装载、卸载数量以及类的装载总空间和耗费时间等         |
| -gc               | 监视Java堆，包含eden、2个survivor区、old区和永久带区域的容量、已用空间、GC时间合计等信息 |
| -gccapcity        | 监视内容与-gc相同，但输出主要关注Java区域用到的最大和最小空间 |
| -gcutil           | 监视内容与-gc相同，但输出主要关注已使用空间占总空间的百分比  |
| -gccause          | 与-gcutil输出信息相同，额外输出导致上次GC产生的原因          |
| -gcnew            | 监控新生代的GC情况                                           |
| -gcnewcapacity    | 与-gcnew监控信息相同，输出主要关注使用到的最大和最小空间     |
| -gcold            | 监控老生代的GC情况                                           |
| -gcoldcapacity    | 与-gcold监控信息相同，输出主要关注使用到的最大和最小空间     |
| -gcpermcapacity   | 输出永久带用到的最大和最小空间                               |
| -compiler         | 输出JIT编译器编译过的方法、耗时信息                          |
| -printcompilation | 输出已经被JIT编译的方法                                      |

比如jstat -gcutil pid查看对应java进程gc情况

```
jstat -gcutil 3109
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
 32.71   0.00  22.70  67.27  94.43  90.80     82    1.584    12    0.521    2.104

s0: 新生代survivor space0简称 就是准备复制的那块 单位为%
s1:指新生代s1已使用百分比，为0的话说明没有存活对象到这边
e:新生代eden(伊甸园)区域(%)
o:老年代(%)
ygc:新生代  次数
ygct:minor gc耗时
fgct:full gc耗时(秒)
GCT: ygct+fgct 耗时
```

**jinfo（JVM configuration Info for Java）**

Jinfo的作用是实时查看虚拟机的各项参数信息jps –v可以查看虚拟机在启动时被显式指定的参数信息，但是如果你想知道默认的一些参数信息呢？除了去查询对应的资料以外，jinfo就显得很重要了。jinfo的用法如下：

```
Jinfo [option] pid

如 jinfo –sysprops {pid}
```

**jmap（JVM Memory Map for Java）**

jmap用于生成堆快照（heapdump）。当然我们有很多方法可以取到对应的dump信息，如我们通过JVM启动时加入启动参数 –XX:HeapDumpOnOutOfMemoryError参数，可以让JVM在出现内存溢出错误的时候自动生成dump文件，亦可以通过-XX:HeapDumpOnCtrlBreak参数，在运行时使用ctrl+break按键生成dump文件，当然我们也可以使用kill -3 pid的方式去恐吓JVM生成dump文件。jmap的作用不仅仅是为了获取dump文件，还可以用于查询finalize执行队列、Java堆和永久带的详细信息，如空间使用率、垃圾回收器等。其运行格式如下：

jmap [option] vmip

Option的信息如下表所示

| Option         | Function                                                     |
| -------------- | ------------------------------------------------------------ |
| -dump          | 生成对应的dump信息，用法为-dump:[live,]format=b,file={fileName} |
| -finalizerinfo | 显示在F-Queue中等待的Finalizer方法的对象（只在linux下生效）  |
| -heap          | 显示堆的详细信息、垃圾回收器信息、参数配置、分代详情等       |
| -histo         | 显示堆栈中的对象的统计信息，包含类、实例数量和合计容量       |
| -permstat      | 以ClassLoder为统计口径显示永久带的内存状态                   |
| -F             | 当虚拟机对-dump无响应时可使用这个选项强制生成dump快照        |

示例：`jmap -dump:format=b,file=heap.dump 20445`



**jhat（JVM Heap Analysis Tool）**

jhat是用来分析dump文件的一个微型的HTTP/HTML服务器，它能将生成的dump文件生成在线的HTML文件，让我们可以通过浏览器进行查阅，然而实际中我们很少使用这个工具，因为一般服务器上设置的堆、栈内存都比较大，生成的dump也比较大，直接用jhat容易造成内存溢出，而是我们大部分会将对应的文件拷贝下来，通过其他可视化的工具进行分析。启用法如下：

```
jhat {dump_file}
```

执行命令后，我们看到系统开始读取这段dump信息，当系统提示Server is ready的时候，用户可以通过在浏览器键入http://ip:7000进行查询。

**jstack（JVM Stack Trace for java）**

jstack用于JVM当前时刻的线程快照，又称threaddump文件，它是JVM当前每一条线程正在执行的堆栈信息的集合。生成线程快照的主要目的是为了定位线程出现长时间停顿的原因，如线程死锁、死循环、请求外部时长过长导致线程停顿的原因。通过jstack我们就可以知道哪些进程在后台做些什么？在等待什么资源等！其运行格式如下：

```
jstack [option] vmid
```

相关的option和function如下表所示

| Option | Function                                 |
| ------ | ---------------------------------------- |
| -F     | 当正常输出的请求不响应时强制输出线程堆栈 |
| -l     | 除堆栈信息外，显示关于锁的附加信息       |
| -m     | 显示native方法的堆栈信息                 |

示例：`jstack -l 20445`

### Java图形化工具

可以根据内存使用情况间接了解内存使用和gc情况

1. JConsole工具。

   ```
   概述选项：监控JVM和一些监控变量的信息。 
   内存选项：内存使用信息 
   线程选项：线程使用信息 
   类选项：类调用信息 
   VM摘要：JVM的信息 
   MBean选项：所有MBean 的信息MBean 展示了所有以一般形式注册到JVM 上的MBean 。MBean 允许你获取所有的平台信息，包括那些不能从其他标签页获取到的信息。注意，其他标签页上的一些信息也在MBean 这里显示。另外，你可以使用 MBean 标签管理你自己的应用的MBean
   ```

   连接远程Java程序：

   - 需要以如下方式运行你的 java 类

     ```
     java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 -Dcom.sun.management.jmxremote.authenticate=是否认证 java类
     ```

   - 修改 /etc/hosts 文件将 127.0.0.1 映射至主机名

   如果要认证访问,还需要做如下步骤

   - 复制 jmxremote.password 文件
   - 修改 jmxremote.password 和 jmxremote.access 文件的权限为 600 即文件所有者可读写
   - 连接时填入 controlRole(用户名),R&D(密码)

2. Java VisualVM 

   JDK1.6 中Java 引入了一个新的可视化的JVM 监控工具：Java VisualVM。

   VisualVM 官方网站：http://visualvm.java.net/

   ```
   jvisualvm
   ```

使用参考：

<https://blog.csdn.net/wuzhiwei549/article/details/80593698>

<https://www.jianshu.com/p/b5093103435a>

#### MAT

mat,全程内存分析工具，专门分析内存溢出时，转存的快照，从而定位问题。

想要分析内存快照，需要配置内存dump，在jvm启动参数中加入以下参数,配置好dump存储路径方便查找。

```
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/usr/local/dump/error.hprof
```

必要的[mat工具下载地址](https://www.eclipse.org/mat/)

#### 离线工具查看

1. [gchisto](https://github.com/jewes/gchisto)

2. [gcviewer](https://github.com/chewiebug/GCViewer)

3. [jvmstat](https://www.oracle.com/technetwork/java/jvmstat-142257.html)

   目录结构比较清晰，很容易就能分辨出各目录的功能及作用：

   ```
   bat：windows启动程序
   bin：linux启动程序
   docs：相关文档
   etc：linux相关依赖库
   jars：相关jar包 
   ```

   使用jvmstat 之前需要配置相应环境变量，环境变量配置如下：

   ```
   JVMSTAT_HOME：jvmstat安装目录
   JVMSTAT_JAVA_HOME：JDK所在目录，与JAVA_HOME值相同
   ```

    配置好两个环境变量之后就可以运行jvmstat 了，运行命令为

   ```
   visualgc pid
   #windows 系统进入bat 目录后运行该命令
   #linux 系统进入bin 目录后运行该命令
   ```

   jvmstat 作为一款基本的JVM 图形化监控工具，优点就是简单易用，我们可以非常直观的观察堆内存的使用情况，当然仅仅为堆内存，所以jvmstat 具有一定的局限性。

4. YourKit Java Profiler

   YourKit 是一个用于分析Java 与.NET 应用程序的智能工具，YourKit Java Profiler 已经被IT 专业人士与分析师公认为最好的分析工具。通过YourKit 技术解决方案可以以非常高的的专业水平分析出CPU 与内存使用情况

   YourKit Java Profiler 还获得了Java Developer's Journal(Java 开发者杂志)的编辑选择奖，其功能的强大可见一斑。

   YourKit 网站官方：[http://www.yourkit.com](http://www.yourkit.com/)

### 新工具

#### [Arthas](https://arthas.gitee.io/) 

`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱。

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？
7. 怎么快速定位应用的热点，生成火焰图？

`Arthas`支持JDK 6+，支持Linux/Mac/Winodws，采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断。

**安装（选一种即可）**：

1. 使用`arthas-boot`

   ```bash
   curl -O https://arthas.gitee.io/arthas-boot.jar
   java -jar arthas-boot.jar
   ```

2. 使用as.sh

   ```bash
   curl -L https://alibaba.github.io/arthas/install.sh | sh
   ```

3. 通过Cloud Toolkit插件使用Arthas一键诊断远程服务器，Idea进入 Plugins 选项，搜索“Alibaba Cloud Toolkit”，并安装即可。

`arthas-boot`是`Arthas`的启动程序，它启动后，会列出所有的Java进程，用户可以选择需要诊断的目标进程。第一次会下载依赖文件

```bash
java -jar arthas-boot.jar --repo-mirror aliyun --use-http
```

**命令：**

1. 基础命令

   - help——查看命令帮助信息
   - [cat](https://alibaba.github.io/arthas/cat.html)——打印文件内容，和linux里的cat命令类似
   - [echo](https://alibaba.github.io/arthas/echo.html)–打印参数，和linux里的echo命令类似
   - [grep](https://alibaba.github.io/arthas/grep.html)——匹配查找，和linux里的grep命令类似
   - [tee](https://alibaba.github.io/arthas/tee.html)——复制标准输入到标准输出和指定的文件，和linux里的tee命令类似
   - [pwd](https://alibaba.github.io/arthas/pwd.html)——返回当前的工作目录，和linux命令类似
   - cls——清空当前屏幕区域
   - session——查看当前会话的信息
   - [reset](https://alibaba.github.io/arthas/reset.html)——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
   - version——输出当前目标 Java 进程所加载的 Arthas 版本号
   - history——打印命令历史
   - quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
   - stop——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
   - [keymap](https://alibaba.github.io/arthas/keymap.html)——Arthas快捷键列表及自定义快捷键

2. jvm相关

   - [dashboard](https://alibaba.github.io/arthas/dashboard.html)——当前系统的实时数据面板
   - [thread](https://alibaba.github.io/arthas/thread.html)——查看当前 JVM 的线程堆栈信息
   - [jvm](https://alibaba.github.io/arthas/jvm.html)——查看当前 JVM 的信息
   - [sysprop](https://alibaba.github.io/arthas/sysprop.html)——查看和修改JVM的系统属性
   - [sysenv](https://alibaba.github.io/arthas/sysenv.html)——查看JVM的环境变量
   - [vmoption](https://alibaba.github.io/arthas/vmoption.html)——查看和修改JVM里诊断相关的option
   - [perfcounter](https://alibaba.github.io/arthas/perfcounter.html)——查看当前 JVM 的Perf Counter信息
   - [logger](https://alibaba.github.io/arthas/logger.html)——查看和修改logger
   - [getstatic](https://alibaba.github.io/arthas/getstatic.html)——查看类的静态属性
   - [ognl](https://alibaba.github.io/arthas/ognl.html)——执行ognl表达式
   - [mbean](https://alibaba.github.io/arthas/mbean.html)——查看 Mbean 的信息
   - [heapdump](https://alibaba.github.io/arthas/heapdump.html)——dump java heap, 类似jmap命令的heap dump功能

3. class/classloader相关

   - [sc](https://alibaba.github.io/arthas/sc.html)——查看JVM已加载的类信息
   - [sm](https://alibaba.github.io/arthas/sm.html)——查看已加载类的方法信息
   - [jad](https://alibaba.github.io/arthas/jad.html)——反编译指定已加载类的源码
   - [mc](https://alibaba.github.io/arthas/mc.html)——内存编译器，内存编译`.java`文件为`.class`文件
   - [redefine](https://alibaba.github.io/arthas/redefine.html)——加载外部的`.class`文件，redefine到JVM里
   - [dump](https://alibaba.github.io/arthas/dump.html)——dump 已加载类的 byte code 到特定目录
   - [classloader](https://alibaba.github.io/arthas/classloader.html)——查看classloader的继承树，urls，类加载信息，使用classloader去getResource

4. monitor/watch/trace相关

   > 请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `stop` 或将增强过的类执行 `reset` 命令。

   - [monitor](https://alibaba.github.io/arthas/monitor.html)——方法执行监控
   - [watch](https://alibaba.github.io/arthas/watch.html)——方法执行数据观测
   - [trace](https://alibaba.github.io/arthas/trace.html)——方法内部调用路径，并输出方法路径上的每个节点上耗时
   - [stack](https://alibaba.github.io/arthas/stack.html)——输出当前方法被调用的调用路径
   - [tt](https://alibaba.github.io/arthas/tt.html)——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

5. profiler/火焰图

   - [profiler](https://alibaba.github.io/arthas/profiler.html)–使用[async-profiler](https://github.com/jvm-profiling-tools/async-profiler)对应用采样，生成火焰图

6. options

   - [options](https://alibaba.github.io/arthas/options.html)——查看或设置Arthas全局开关

7. 管道

   Arthas支持使用管道对上述命令的结果进行进一步的处理，如`sm java.lang.String * | grep 'index'`

   - grep——搜索满足条件的结果
   - plaintext——将命令的结果去除ANSI颜色
   - wc——按行统计输出结果

8. 后台异步任务

   当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了，详情请参考[这里](https://alibaba.github.io/arthas/async.html)

   - 使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行（生命周期默认为1天）
   - jobs——列出所有job
   - kill——强制终止任务
   - fg——将暂停的任务拉到前台执行
   - bg——将暂停的任务放到后台执行

9. [Web Console](https://alibaba.github.io/arthas/web-console.html)

   通过websocket连接Arthas。用户在attach成功之后，可以直接访问：http://127.0.0.1:3658/。默认情况下，arthas只listen 127.0.0.1，所以如果想从远程连接，则可以使用 `--target-ip`参数指定listen的IP

10. 以java agent方式启动

    ```
    java -javaagent:/tmp/test/arthas-agent.jar -jar arthas-demo.jar
    ```

    默认的配置项在解压目录里的`arthas.properties`文件里。参考： https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html

11. Arthas Spring Boot Starter

    配置maven依赖：

    ```xml
            <dependency>
                <groupId>com.taobao.arthas</groupId>
                <artifactId>arthas-spring-boot-starter</artifactId>
                <version>${arthas.version}</version>
            </dependency>
    ```

    应用启动后，spring会启动arthas，并且attach自身进程。

    