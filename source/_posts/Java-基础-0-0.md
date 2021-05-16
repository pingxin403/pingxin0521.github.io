---
title: Java 简介与环境配置
date: 2019-04-02 23:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 前言

*Why?*

第一、简单易学；第二、岗位就业容易；第三、Java在IT行业属于大哥地位。如果达到高级程序员水平后，很多人在这个阶段选择了不同的方向，就业方向很多。

*What?*

Java是一门面向对象编程语言，不仅吸收了C++语言的各种优点，还摒弃了C++里难以理解的多继承、指针等概念，因此Java语言具有功能强大和简单易用两个特征。
Java语言作为静态面向对象编程语言的代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程。

Java具有简单性、面向对象、分布式、健壮性、安全性、平台独立与可移植性、多线程、动态性等特点 。

<!--more-->
1. Java语言是简单的：
Java语言的语法与C语言和C++语言很接近，使得大多数程序员很容易学习和使用。另一方面，Java丢弃了C++中很少使用的、很难理解的、令人迷惑的那些特性，如操作符重载、多继承、自动的强制类型转换。特别地，Java语言不使用指针，而是引用。并提供了自动的废料收集，使得程序员不必为内存管理而担忧。

2. Java语言是面向对象的：
Java语言提供类、接口和继承等面向对象的特性，为了简单起见，只支持类之间的单继承，但支持接口之间的多继承，并支持类与接口之间的实现机制（关键字为implements）。Java语言全面支持动态绑定，而C++语言只对虚函数使用动态绑定。总之，Java语言是一个纯的面向对象程序设计语言。
3. Java语言是分布式的：
Java语言支持Internet应用的开发，在基本的Java应用编程接口中有一个网络应用编程接口（java net），它提供了用于网络应用编程的类库，包括URL、URLConnection、Socket、ServerSocket等。Java的RMI（远程方法激活）机制也是开发分布式应用的重要手段。

4. Java语言是健壮的：
Java的强类型机制、异常处理、垃圾的自动收集等是Java程序健壮性的重要保证。对指针的丢弃是Java的明智选择。Java的安全检查机制使得Java更具健壮性。

5. Java语言是安全的：
Java通常被用在网络环境中，为此，Java提供了一个安全机制以防恶意代码的攻击。除了Java语言具有的许多安全特性以外，Java对通过网络下载的类具有一个安全防范机制（类ClassLoader），如分配不同的名字空间以防替代本地的同名类、字节代码检查，并提供安全管理机制（类SecurityManager）让Java应用设置安全哨兵。

6. Java语言是体系结构中立的：
Java程序（后缀为java的文件）在Java平台上被编译为体系结构中立的字节码格式（后缀为class的文件），然后可以在实现这个Java平台的任何系统中运行。这种途径适合于异构的网络环境和软件的分发。

7. Java语言是可移植的：
这种可移植性来源于体系结构中立性，另外，Java还严格规定了各个基本数据类型的长度。Java系统本身也具有很强的可移植性，Java编译器是用Java实现的，Java的运行环境是用ANSI C实现的。

8. Java语言是解释型的：
如前所述，Java程序在Java平台上被编译为字节码格式，然后可以在实现这个Java平台的任何系统中运行。在运行时，Java平台中的Java解释器对这些字节码进行解释执行，执行过程中需要的类在联接阶段被载入到运行环境中。

9. Java是高性能的：
与那些解释型的高级脚本语言相比，Java的确是高性能的。事实上，Java的运行速度随着JIT(Just-In-Time）编译器技术的发展越来越接近于C++。

10. Java语言是多线程的：
在Java语言中，线程是一种特殊的对象，它必须由Thread类或其子（孙）类来创建。通常有两种方法来创建线程：其一，使用型构为Thread(Runnable)的构造子类将一个实现了Runnable接口的对象包装成一个线程，其二，从Thread类派生出子类并重写run方法，使用该子类创建的对象即为线程。值得注意的是Thread类已经实现了Runnable接口，因此，任何一个线程均有它的run方法，而run方法中包含了线程所要运行的代码。线程的活动由一组方法来控制。Java语言支持多个线程的同时执行，并提供多线程之间的同步机制（关键字为synchronized）。

11. Java语言是动态的：
Java语言的设计目标之一是适应于动态变化的环境。Java程序需要的类能够动态地被载入到运行环境，也可以通过网络来载入所需要的类。这也有利于软件的升级。另外，Java中的类有一个运行时刻的表示，能进行运行时刻的类型检查。



*Where？*

其运行在Java虚拟机（JVM）之上。JDK（Java Development Kit）称为Java开发包或Java开发工具，是一个编写Java的Applet小程序和应用程序的程序开发环境。JDK是整个Java的核心，包括了Java运行环境（Java Runtime Envirnment），一些Java工具和Java的核心类库（Java API）。不论什么Java应用服务器实质都是内置了某个版本的JDK。主流的JDK是Sun公司发布的JDK，除了Sun之外，还有很多公司和组织都开发了自己的JDK，例如，IBM公司开发的JDK，BEA公司的Jrocket，还有GNU组织开发的JDK。

![1.jpeg](https://i.loli.net/2019/06/08/5cfb226330d8a39265.jpeg)

另外，可以把Java API类库中的Java SE API子集和Java虚拟机这两部分统称为JRE（JAVA Runtime Environment），JRE是支持Java程序运行的标准环境。

JRE是个运行环境，JDK是个开发环境。因此写Java程序的时候需要JDK，而运行Java程序的时候就需要JRE。而JDK里面已经包含了JRE，因此只要安装了JDK，就可以编辑Java程序，也可以正常运行Java程序。但由于JDK包含了许多与运行无关的内容，占用的空间较大，因此 *运行普通的Java程序无须安装JDK，而只需要安装JRE即可* 。

*When?*

Java可以编写桌面应用程序、Web应用程序、分布式系统和嵌入式系统应用程序等。

1. Android应用
许多的 Android应用都是Java程序员开发者开发。虽然 Android运用了不同的JVM以及不同的封装方式，但是代码还是用Java语言所编写。相当一部分的手机中都支持JAVA游戏，这就使很多非编程人员都认识了JAVA。

2. 在金融业应用的服务器程序
Java在金融服务业的应用非常广泛，很多第三方交易系统、银行、金融机构都选择用Java开发，因为相对而言，Java较安全   。大型跨国投资银行用Java来编写前台和后台的电子交易系统，结算和确认系统，数据处理项目以及其他项目。大多数情况下，Java被用在服务器端开发，但多数没有任何前端，它们通常是从一个服务器（上一级）接收数据，处理后发向另一个处理系统（下一级处理）。

3. 网站
Java 在电子商务领域以及网站开发领域占据了一定的席位。开发人员可以运用许多不同的框架来创建web项目，SpringMVC，Struts2.0以及frameworks。即使是简单的 servlet，jsp和以struts为基础的网站在政府项目中也经常被用到。例如医疗救护、保险、教育、国防以及其他的不同部门网站都是以Java为基础来开发的。

4. 嵌入式领域
Java在嵌入式领域发展空间很大。在这个平台上，只需130KB就能够使用Java技术（在智能卡或者传感器上）。

5. 大数据技术
Hadoop以及其他大数据处理技术很多都是用Java，例如Apache的基于Java的HBase和Accumulo以及 ElasticSearchas。

6. 高频交易的空间
Java平台提高了这个平台的特性和即使编译，他同时也能够像 C++ 一样传递数据。正是由于这个原因，Java成为的程序员编写交易平台的语言，因为虽然性能不比C++，但开发人员可以避开安全性，可移植性和可维护性等问题。

7. 科学应用
Java在科学应用中是很好选择，包括自然语言处理。最主要的原因是因为Java比C++或者其他语言相对其安全性、便携性、可维护性以及其他高级语言的并发性更好。

*Who?*

很多人对java很嫌弃，说它又臭又长，性能比不上C、C++。但是Java常年居在排名第一的编程语言也是有他的原因：新手学习很快，编程有规范，维护性很好。

什么人能够学习Java呢？任何人都可以学习，它不像C、C++那样复杂，任何错误都是可以追踪的。

什么人使用java呢？网络应用程序开发人员、Android开发人员、嵌入式开发人员等等。

*How?*

由四方面组成：
（1）Java编程语言
（2）Java类文件格式
（3）Java虚拟机
（4）Java应用程序接口

当编辑并运行一个Java程序时，需要同时涉及到这四种方面。使用文字编辑软件（例如记事本、写字板、UltraEdit等）或集成开发环境（Eclipse、MyEclipse、IDEA等）在Java源文件中定义不同的类   ，通过调用类（这些类实现了Java API）中的方法来访问资源系统，把源文件编译生成一种二进制中间码，存储在class文件中，然后再通过运行与操作系统平台环境相对应的Java虚拟机来运行class文件，执行编译产生的字节码，调用class文件中实现的方法来满足程序的Java API调用。

### 配置Java开发环境

Java语言尽量保证系统内存在1G以上，其他工具如下所示：
```
Linux 系统、Mac OS 系统、Windows 95/98/2000/XP，WIN 7/8系统。
Java JDK 7、8……
Notepad 编辑器、vim或者其他编辑器。
IDE：Eclipse、IDEA
```

>作者的环境是Ubuntu 18.04，安装Java 8、Java 11，配置常用环境是Java 8， 编程IDE是IntelliJ IDEA  18.3 Ultimate Edition

删除Ubuntu自带的openjdk

```
sudo apt-get remove openjdk*
```

PS：若不想删除openJDK，想要二者并存，只需要设置默认的jdk即可

```
sudo update-alternatives --install /usr/bin/java java /opt/java/java-11/bin/java 300 
sudo update-alternatives --install /usr/bin/javac javac /opt/java/java-11/bin/javac 300
```

#### Java环境

JVM ：英文名称（Java Virtual Machine），就是我们耳熟能详的 Java 虚拟机。它只认识 xxx.class 这种类型的文件，它能够将 class 文件中的字节码指令进行识别并调用操作系统向上的 API 完成动作。所以说，jvm 是 Java 能够跨平台的核心，具体的下文会详细说明。

JRE ：英文名称（Java Runtime Environment），我们叫它：Java 运行时环境。它主要包含两个部分，jvm 的标准实现和 Java 的一些基本类库。它相对于 jvm 来说，多出来的是一部分的 Java 类库。

JDK ：英文名称（Java Development Kit），Java 开发工具包。jdk 是整个 Java 开发的核心，它集成了 jre 和一些好用的小工具。例如：javac，java，jar等。

我们配置java环境需要安装JDK，从[官网](https://www.oracle.com/technetwork/java/javase/overview/index.html)下载对应版本。

![](https://i.loli.net/2019/04/08/5caaca86967e8.png)

我们下载java 11 和java 8 。

![](https://i.loli.net/2019/04/08/5caaca86a817a.png)

选择java 11 安装包。

![](https://i.loli.net/2019/04/08/5caaca86a3daf.png)

选择java 8 安装包。

![](https://i.loli.net/2019/04/08/5caaca86b3d70.png)

把安装包下载后，解压并重命名，移动到`/opt/java/`下，如下：

``` （shell）
$ tree -L 2  /opt/java/
/opt/java/
├── java-11
│   ├── bin
│   ├── conf
│   ├── include
│   ├── jmods
│   ├── legal
│   ├── lib
│   ├── README.html
│   └── release
└── java-8
     ├── bin
     ├── COPYRIGHT
     ├── include
     ├── javafx-src.zip
     ├── jre
     ├── lib
     ├── LICENSE
     ├── man
     ├── README.html
     ├── release
     ├── src.zip
     ├── THIRDPARTYLICENSEREADME-JAVAFX.txt
     └── THIRDPARTYLICENSEREADME.txt

```

我们为java 11配置环境变量：

``` （shell）
$ sudo vim /etc/profile
#在文件最后面添加以下，保存并退出

#java
export JAVA_HOME=/opt/java/java-11
export JER_HOME=${JAVA_HOME}/jre
export PATH=${PATH}:${JAVA_HOME}/bin

$ source /etc/profile
#验证是否设置成功
$ java -version
java version "11.0.3" 2019-04-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.3+12-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.3+12-LTS, mixed mode)
```

**解决sudo情况下PATH无效**
```（shell）
$ sudo java --version
#会出现没有找到java命令
$ vim ~/.bashrc
#找到alias声明位置，在后面添加如下，保存并退出

alias sudo="sudo env PATH=$PATH"

$ source  ~/.bashrc

$sudo java --version
hyp@hyp-HP-Notebook:~$ sudo java -version
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```

#### IDE配置

从[官网](https://www.jetbrains.com/idea/download/previous.html)下载IDE，记得选择ultimate的2018.3版本。

下载完成后，解压后重命名为idea-IU，移动到/opt/下,如下：

```（shell）
$ tree /opt/idea-IU/ -L 1
/opt/idea-IU/
├── bin
├── build.txt
├── help
├── Install-Linux-tar.txt
├── lib
├── license
├── plugins
└── redist

6 directories, 2 files
```


*绿化方法：*

链接:https://pan.baidu.com/s/15ZRaCZdQU_wzobLd5zjirg 提取码:5ky3 

下载后修改/opt/idea-IU/bin/idea.vmoptions 和/opt/idea-/bin/idea64.vmoptions，在末尾添加`-javaagent:/PATH/TO/JetbrainsldesCrack-4.2.jar`。

下载设置文件，以减少配置时间：链接: https://pan.baidu.com/s/1AEccqjKkqQQYV73Kye6AiA 提取码: wbqx

在命令行打开/opt/idea-IU/bin/idea.sh，选择导入设置，导入上面下载的jar文件。

激活时选择Activation Code，输入以下内容：

```
ThisCrackLicenseId-{
"licenseId":"ThisCrackLicenseId",
"licenseeName":"hyp",
"assigneeName":"",
"assigneeEmail":"hyp@163.com",
"licenseRestriction":"lalala",
"checkConcurrentUse":false,
"products":[
{"code":"II","paidUpTo":"2099-12-31"},
{"code":"DM","paidUpTo":"2099-12-31"},
{"code":"AC","paidUpTo":"2099-12-31"},
{"code":"RS0","paidUpTo":"2099-12-31"},
{"code":"WS","paidUpTo":"2099-12-31"},
{"code":"DPN","paidUpTo":"2099-12-31"},
{"code":"RC","paidUpTo":"2099-12-31"},
{"code":"PS","paidUpTo":"2099-12-31"},
{"code":"DC","paidUpTo":"2099-12-31"},
{"code":"RM","paidUpTo":"2099-12-31"},
{"code":"CL","paidUpTo":"2099-12-31"},
{"code":"PC","paidUpTo":"2099-12-31"}
],
"hash":"2911276/0",
"gracePeriodDays":7,
"autoProlongated":false}
```

至此IDE设置完成。

后面可以选择一些常用的插件：

- 阿里代码规约检测：Alibaba Java Coding Guidelines
- 快捷键提示工具：Key promoter X
- 代码注解插件： Lombok
- 代码生成工具：CodeMaker
- 单元测试测试生成工具：JUnitGenerator V2.0
- Mybatis 工具：Free Mybatis plugin
- JSON转领域对象工具：GsonFormat4DataBinding
- 字符串工具：String Manipulation
- 生成对象set方法：GenerateAllSetter
- Redis可视化：Iedis
- K8s工具：Kubernetes
- 中英文翻译工具：Translation
- CodeGlance：代码编辑区缩略图插件
- Background Image Plus+：当把背景设置成你自己心仪的的图片
- Coding counter：统计代码行数
- Maven辅助神器：Maven Helper
- RESTful 服务开发辅助工具集: RestfulToolkit

可以从这里直接下载：链接：https://pan.baidu.com/s/11jwGITC7yq4dXaOFE892zw 密码：47cj

如果只是配置Java一般环境，上面就可以了，下面是一些非新手食用的教程。

#### Maven配置

Maven是Apache基金会发布的项目管理工具软件，我们需要在项目过程使用到它。

从[官网](http://maven.apache.org/download.cgi)下载最新版，选择Binary tar.gz archive。下载完成后解压，重命名为maven，放置到`/opt/xauto/`目录下，配置环境变量到/etc/profile。
```shell
$ sudo vim /etc/profile

#在末尾处添加以下配置

#apache
export APACHE_HOME=/opt/xauto
#maven
export M2_HOME=${APACHE_HOME}/maven
export PATH=${PATH}:${M2_HOME}/bin

$ source /etc/profile

$ mvn -v
Apache Maven 3.6.2 (40f52333136460af0dc0d7232c0dc0bcf0d9e117; 2019-08-27T23:06:16+08:00)
Maven home: /opt/xauto/maven
Java version: 1.8.0_221, vendor: Oracle Corporation, runtime: /opt/java/java-8/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "4.15.0-64-generic", arch: "amd64", family: "unix"

```

#### tomcat配置

Tomcat服务器是一个免费的开放源代码的Web应用服务器。Tomcat是Apache软件基金会（Apache Software Foundation）的Jakarta项目中的一个核心项目，由Apache、Sun和其他一些公司及个人共同开发而成。由于有了Sun的参与和支持，最新的Servlet 和JSP规范总是能在Tomcat中得到体现，Tomcat 5支持最新的Servlet 2.4和JSP 2.0规范。因为Tomcat技术先进、性能稳定，而且免费，因而深受Java爱好者的喜爱并得到了部分软件开发商的认可，是目前比较流行的Web应用服务器。

从[官网](https://tomcat.apache.org/download-90.cgi)下载最新版的tomcat 9 ，选择Core--》tar.gz，下载后解压，重命名为tomcat，并且移动到/opt/apache目录下，设置环境变量。
``` （shell）
$ sudo vim /etc/profile
#tomcat
export CATALINA_HOME=/opt/server/tomcat/

$ sudo ln -s /opt/server/tomcat/bin/catalina.sh /usr/bin/catalina

$ ${CATALINA_HOME}/bin/version.sh
Using CATALINA_BASE:   /opt/apache/tomcat
Using CATALINA_HOME:   /opt/apache/tomcat
Using CATALINA_TMPDIR: /opt/apache/tomcat/temp
Using JRE_HOME:        /opt/java/java1.8
Using CLASSPATH:       /opt/server/tomcat/bin/bootstrap.jar:/opt/apache/tomcat/bin/tomcat-juli.jar
Server version: Apache Tomcat/9.0.17
Server built:   Mar 13 2019 15:55:27 UTC
Server number:  9.0.17.0
OS Name:        Linux
OS Version:     4.18.0-17-generic
Architecture:   amd64
JVM Version:    1.8.0_201-b09
JVM Vendor:     Oracle Corporation

#打开tomcat，默认是http://localhost:8080/，可以在浏览器查看。
$ sudo catalina start  
Using CATALINA_BASE:   /opt/server/tomcat/
Using CATALINA_HOME:   /opt/server/tomcat/
Using CATALINA_TMPDIR: /opt/server/tomcat/temp
Using JRE_HOME:        /opt/java/java1.8
Using CLASSPATH:       /opt/server/tomcat/bin/bootstrap.jar:/opt/server/tomcat/bin/tomcat-juli.jar
Tomcat started.
```


### 第一个Java程序

编写一个Hello.java文件，里面内容如下：
```（java）
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```
>注：String args[] 与 String[] args 都可以执行，但推荐使用 String[] args，这样可以避免歧义和误读。

运行以上实例，输出结果如下：
```(shell)
$ javac HelloWorld.java
$ java HelloWorld
Hello World
```
**Java程序执行流程：**

![ZCuIFs.png](https://s2.ax1x.com/2019/06/23/ZCuIFs.png)

如上图所示，首先Java源代码文件(.java后缀)会被Java编译器编译为字节码文件(.class后缀)，然后由JVM中的类加载器加载各个类的字节码文件，加载完毕之后，交由JVM执行引擎执行。在整个程序执行过程中，JVM会用一段空间来存储程序执行期间需要用到的数据和相关信息，这段空间一般被称作为Runtime Data Area（运行时数据区），也就是我们常说的JVM内存。因此，在Java中我们常常说到的内存管理就是针对这段空间进行管理（如何分配和回收内存空间）。

**Java 源程序与编译型运行区别：**

![Java运行.png](https://i.loli.net/2019/04/08/5cab17ef81dbb.png)

以上我们使用了两个命令 javac 和 java。

javac 后面跟着的是java文件的文件名，例如 HelloWorld.java。 该命令用于将 java 源文件编译为 class 字节码文件，如： javac HelloWorld.java。

运行javac命令后，如果成功编译没有错误的话，会出现一个 HelloWorld.class 的文件。

java 后面跟着的是java文件中的类名,例如 HelloWorld 就是类名，如: java HelloWorld。

>注意：java命令后面不要加.class。

#### Jshell使用

从java9开始，java开始引入了类似于python的交互式 REPL（Read-Eval-Print Loop，读取-求值-输出 循环）工具。使用 JShell，你可以输入代码片段并马上看到运行结果，然后就可以根据需要作出调整。

使用JShell，您可以一次输入一个程序元素，立即查看结果，并根据需要进行调整。
Java程序开发通常涉及以下过程：

- 写一个完整的程序。
- 编译它并修复任何错误。
- 运行程序。
- 弄清楚它有什么问题。
- 编辑它。
- 重复这个过程。

JShell可帮助您在开发程序时尝试代码并轻松探索选项。您可以测试单个语句，尝试不同的方法变体，并在JShell会话中试验不熟悉的API。JShell不替换IDE。在开发程序时，将代码粘贴到JShell中进行试用，然后将JShell中的工作代码粘贴到程序编辑器或IDE中。

首先我们要安装jdk，并且版本要>=9。我这里安装的是jdk11，并且配置好环境变量。

```shell
#启动
$ jshell
#要以详细模式启动JShell，请使用以下-v选项
$ jshell -v
#退出：
/exit

```

Jshell提供给我们的所有命令：

```
jshell> /help
|  键入 Java 语言表达式, 语句或声明。
|  或者键入以下命令之一:
|  /list [<名称或 id>|-all|-start]
|       列出您键入的源
|  /edit <名称或 id>
|       编辑源条目
|  /drop <名称或 id>
|       删除源条目
|  /save [-all|-history|-start] <文件>
|       将片段源保存到文件
|  /open <file>
|       打开文件作为源输入
|  /vars [<名称或 id>|-all|-start]
|       列出已声明变量及其值
|  /methods [<名称或 id>|-all|-start]
|       列出已声明方法及其签名
|  /types [<名称或 id>|-all|-start]
|       列出类型声明
|  /imports
|       查看已导入的包
|  /exit [<integer-expression-snippet>]
|       退出 jshell 工具
|  /env [-class-path <路径>] [-module-path <路径>] [-add-modules <模块>] ...
|       查看或更改评估上下文
|  /reset [-class-path <路径>] [-module-path <路径>] [-add-modules <模块>]...
|       重置 jshell 工具
|  /reload [-restore] [-quiet] [-class-path <路径>] [-module-path <路径>]...
|       重置和重放相关历史记录 -- 当前历史记录或上一个历史记录 (-restore)
|  /history
|       您键入的内容的历史记录
|  /help [<command>|<subject>]
|       获取有关使用 jshell 工具的信息
|  /set editor|start|feedback|mode|prompt|truncation|format ...
|       设置配置信息
|  /? [<command>|<subject>]
|       获取有关使用 jshell 工具的信息
|  /!
|       重新运行上一个片段 -- 请参阅 /help rerun
|  /<id>
|       按 ID 或 ID 范围重新运行片段 -- 参见 /help rerun
|  /-<n>
|       重新运行以前的第 n 个片段 -- 请参阅 /help rerun
|
|  有关详细信息, 请键入 '/help', 后跟
|  命令或主题的名称。
|  例如 '/help /list' 或 '/help intro'。主题:
|
|  intro
|       jshell 工具的简介
|  id
|       片段 ID 以及如何使用它们的说明
|  shortcuts
|       片段和命令输入提示, 信息访问以及
|       自动代码生成的按键说明
|  context
|       /env /reload 和 /reset 的评估上下文选项的说明
|  rerun
|       重新评估以前输入片段的方法的说明
```

**运行代码片段**

使用详细选项启动JShell以获得最大可用反馈量：

```
jshell -v
|  欢迎使用 JShell -- 版本 11.0.2
|  要大致了解该版本, 请键入: /help intro
```

在提示符处输入以下示例语句，并查看显示的输出：

```
jshell> int x = 45
x ==> 45
|  已创建 变量 x : int
```

首先，显示结果。将其读作：变量x的值为45.因为您处于详细模式，所以还会显示所发生情况的描述。

注意：如果未输入分号，则会自动将终止分号添加到完整代码段的末尾。

当输入的表达式没有命名变量时，会创建一个临时变量，以便稍后可以引用该值。以下示例显示表达式和方法结果的临时值。该示例还显示了…> 在代码段需要多行输入完成时使用的continuation prompt（）：

```
jshell> String twice(String s) {
   ...>   return s + s;
   ...> }
|  已创建 方法 twice(String)

jshell> twice("Oecan")
$4 ==> "OecanOecan"
|  已创建暂存变量 $4 : String

```

**改变定义**

在试验代码时，您可能会发现变量，方法或类的定义没有按照您希望的方式执行。通过输入新的定义可以轻松更改定义，该定义将覆盖先前的定义。

要更改变量，方法或类的定义，只需输入新定义即可。例如，twice在定义该方法尝试片段得到在下面的示例中的新定义：

```shell
jshell> String twice(String s) {
   ...>   return "Twice: " + s;
   ...> }
|  已修改 方法 twice(String)
|    更新已覆盖 方法 twice(String)

jshell> twice("thing")
$6 ==> "Twice: thing"
|  已创建暂存变量 $6 : String
```

还可以改变变量的类型定义。以下示例显示x从String更改int为：

```
jshell> int x = 45
x ==> 45
|  已创建 变量 x : int

jshell> String x
x ==> null
|  已替换 变量 x : String
|    更新已覆盖 变量 x : int

```

 **查看默认导入和使用自动补全功能**

默认情况下，JShell提供了一些常用包的导入，我们可以使用 **import** 语句导入必要的包或是从指定的路径的包，来运行我们的代码片段。我们可以输入以下命令列出所有导入的包：

```
jshell> /imports 
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*
```

**自动补全的功能**