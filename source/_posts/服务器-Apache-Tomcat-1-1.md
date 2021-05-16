---
title: Apache Tomcat入门
date: 2019-04-03 11:18:59
tags:
 - 服务器
 - Tomcat
categories:
 - 服务器
 - Tomcat
---
### 前言

Tomcat服务器是一个免费的开放源代码的Web应用服务器。Tomcat是Apache软件基金会（Apache Software Foundation）的Jakarta项目中的一个核心项目，由Apache、Sun和其他一些公司及个人共同开发而成。由于有了Sun的参与和支持，最新的Servlet 和JSP规范总是能在Tomcat中得到体现，Tomcat 5支持最新的Servlet 2.4和JSP 2.0规范。因为Tomcat技术先进、性能稳定，而且免费，因而深受Java爱好者的喜爱并得到了部分软件开发商的认可，是目前比较流行的Web应用服务器。
<!--more-->

![2.jpg](https://i.loli.net/2019/05/13/5cd937a66330a20998.jpg)

Tomcat 服务器属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应HTML（标准通用标记语言下的一个应用）页面的访问请求。实际上Tomcat是Apache 服务器的扩展，但运行时它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。

诀窍是，当配置正确时，Apache 为HTML页面服务，而Tomcat 实际上运行JSP 页面和Servlet。另外，Tomcat和IIS等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache服务器。目前Tomcat最新版本为9.0。

### tomcat安装

**前提是配置好Java环境：JAVA_HOME**

从[官网](https://tomcat.apache.org/download-90.cgi)下载最新版的tomcat 9 ，选择Core--》tar.gz，下载后解压，重命名为tomcat，并且移动到/opt/apache目录下，设置环境变量。
``` bash
$ sudo vim /etc/profile
#tomcat
export CATALINA_HOME=${APACHE_HOME}/tomcat

$ sudo ln -s /opt/apache/tomcat/bin/catalina.sh /usr/bin/catalina

$ ${CATALINA_HOME}/bin/version.sh
Using CATALINA_BASE:   /opt/apache/tomcat
Using CATALINA_HOME:   /opt/apache/tomcat
Using CATALINA_TMPDIR: /opt/apache/tomcat/temp
Using JRE_HOME:        /opt/java/java1.8
Using CLASSPATH:       /opt/apache/tomcat/bin/bootstrap.jar:/opt/apache/tomcat/bin/tomcat-juli.jar
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
Using CATALINA_BASE:   /opt/apache/tomcat
Using CATALINA_HOME:   /opt/apache/tomcat
Using CATALINA_TMPDIR: /opt/apache/tomcat/temp
Using JRE_HOME:        /opt/java/java1.8
Using CLASSPATH:       /opt/apache/tomcat/bin/bootstrap.jar:/opt/apache/tomcat/bin/tomcat-juli.jar
Tomcat started.
```

#### 配置文件

1. Tomcat的主目录为/usr/local/Tomcat 9/ 其子目录的用处如下：
```
bin/:存放Windows或Linux平台上启动和关闭Tomcat的脚本文件
conf/:存放Tomcat服务器的各种全局配置文件，其中最重要的是server.xml和web.xml
lib/:存放Tomcat运行需要的库文件
logs:存放Tomcat执行时的LOG文件
webapps：Tomcat的默认Web发布目录，可以修改
work：存放jsp编译后产生的class文件
```

2. 各配置文件作用说明
```
catalina.policy：权限控制配置文件,当基于-security选项启动tomcat实例时会读取此配置文件
catalina.properties：Tomcat属性配置文件,设定类加载器路径、安全包列表、调整性能的参数信息
context.xml：上下文配置文件
logging.properties：日志log相关配置文件,定义日志相关的配置信息，例如：日志级别、文件路径
server.xml：主配置文件
Tomcat-users.xml：manager-gui管理用户配置文件
web.xml：Tomcat的servlet、servlet-mapping、filter、MIME等相关配置
```

3. 主配置文件解读
```
server.xml为Tomcat的主要配置文件，可配置Tomcat的启动端口、网站目录、虚拟主机、开启https等重要功能
<Server>                     顶层类元素：一个配置文件中只能有一个<Server>元素，可包含多个Service。
    <Listener/>
    <GlobalNamingResources>
    <Resource/>
    </GlobalNamingResources>
    <Service>                顶层类元素：本身不是容器，可包含一个Engine，多个Connector。
        <Connector/>         连接器类元素：代表通信接口。
        <Engine>          容器类元素：为特定的Service组件处理所有客户请求，可包含多个Host。
            <Host>         容器类元素：为特定的虚拟主机处理所有客户请求，可包含多个Context。
                <Context>   容器类元素：为特定的Web应用处理所有客户请求。 
                </Context>
            </Host>
        </Engine>
     </Service>
</Server>

```

​	![1.png](https://i.loli.net/2019/05/13/5cd93d0c403c169880.png)

4. 属性变量：

```
CATALINA.BASE
CATALINA.HOME
CATALINA.TMPDIR
```
即通过环境变量名获取变量值，但是不能直接使用环境变量名，需要替换`_`为`.`，比如`CATALINA_BASE`，使用就是`CATALINA.BASE`。

5. web.xml

   为部署至此tomcat上的所有实例上所有的web应用程序提供默认部署描述符：通常用于为webapp提供基本的servlet定义和MIME映射表等

   通常有2个存放位置：

   ```
   $CATALINA_BASE/conf/web.xml和每个Web应用程序(/WEB-INFO/web.xml)
   ```

   Tomcat在部署一个应用程序时（包括重启和重新载入），会首先读取$CATALINA_BASE/conf/web.xml，然后再读取WEB-INFO/web.xml

6. webapps体系结构：
    webapp有特定组织格式，是一种层次型目录结构；通常包含servlet代码;jsp页面文件、类文件、部署描述符文件等等，一般会打包成格式文档

   ```
    /：web应用程序的根目录，相对于应用程序而言，即 ROOT目录
    /WEB-INF：此webapp的私有资源目录，通常web.xml和context.xml文件均放置于此处
    /WEB-INF/classes:此webapp自有的类
    /WEB-INF/lib:此webapp自有的能够被打包成jar格式的类
    /META-INF：不同应用程序不同
   ```

7. webapp的归档格式：

   ```
   EJB类的归档的扩展名为.jar
   web应用程序归档扩展名为.war
   资源适配器类的文件扩展名为.rar
   企业级应用程序的扩展名为.ear
   web服务的扩展名为.ear或.war
   ```

8. bin常用执行文件，其中.bat和.sh文件很多都是成对出现的，作用是一样的，一个是Windows的，一个是*inux。

   ```
   startup文件：主要是检查catalina.bat/sh 执行所需环境，并调用catalina.bat 批处理文件。启动tomcat。
   catalina文件：真正启动Tomcat文件，可以在里面设置jvm参数。
    shutdown文件：关闭Tomcat
   version：显示软件信息
   ```

   catalina命令：`catalina.sh ( commands ... )`

   ```
    debug             Start Catalina in a debugger
     debug -security   Debug Catalina with a security manager
     jpda start        Start Catalina under JPDA debugger
     run               Start Catalina in the current window
     run -security     Start in the current window with security manager
     start             Start Catalina in a separate window
     start -security   Start in a separate window with security manager
     stop              Stop Catalina, waiting up to 5 seconds for the process to end
     stop n            Stop Catalina, waiting up to n seconds for the process to end
     stop -force       Stop Catalina, wait up to 5 seconds and then use kill -KILL if still running
     stop n -force     Stop Catalina, wait up to n seconds and then use kill -KILL if still running
     configtest        Run a basic syntax check on server.xml - check exit code for result
     version           What version of tomcat are you running?
   
   ```

#### 主配置文件--server.xml

tomcat的主配置文件server.xml文件的整体结构如下，而且文件结构也符合tomcat的架构层次。

```xml
<Server port="8005" shutdown="SHUTDOWN">
        <Service name="Catalina">
                <Connector port="8080" />
                <Connector  />
                ...
                <Engine name="Catalina" defaultHost="localhost">
                        <Host name="localhost">
                        ...
                        </Host>
                </Engine>
        </Service>
</Server>
```

tomcat是java语言开发的，因此配置文件也相符合于面向对象的思想，例如，文件中的每个元素创建“对象”，并为属性赋值实例化对象。

![ln981J.png](https://s2.ax1x.com/2019/12/28/ln981J.png)

该配置文件中的元素都是大写字母开头。
**Server**
配置文件中的顶层元素，代表一个tomcat实例，并且配置文件中仅能有一个Server。默认使用的实现类是org.apache.catalina.core.StandardServer。

| 属性      | 备注                                          | 属性值   |
| --------- | --------------------------------------------- | -------- |
| classname | 实现server的类名                              |          |
| port      | 接受关闭tomcat请求的端口，仅能绑定至127.0.0.1 | 8005     |
| shutdown  | 定义关闭tomcat的字符串指令                    | SHUTDOWN |

![ln9wtO.png](https://s2.ax1x.com/2019/12/28/ln9wtO.png)

Server内嵌的子元素为 Listener、GlobalNamingResources、Service。

**Listener**
表示一个事件的监听器

| 属性      | 备注               | 属性值    |
| --------- | ------------------ | --------- |
| clsssname | 实现该监听器的类名 |           |
| SSLEngine | 是否使用SSL        | on或者off |

默认配置的 5个Listener 的含义:

```xml
<!‐‐ 用于以日志形式输出服务器 、操作系统、JVM的版本信息 ‐‐>
<Listener className="org.apache.catalina.startup.VersionLoggerListener"/>
<!‐‐ 用于加载(服务器启动) 和 销毁 (服务器停止) APR。 如果找不到APR库, 则会
输出日志, 并不影响Tomcat启动 ‐‐>
<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
<!‐‐ 用于避免JRE内存泄漏问题 ‐‐>
<Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
<!‐‐ 用户加载(服务器启动) 和 销毁(服务器停止) 全局命名服务 ‐‐>
<Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"
/>
<!‐‐ 用于在Context停止时重建Executor 池中的线程, 以避免ThreadLocal 相关的内
存泄漏 ‐‐>
<Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
```

**GlobalNamingResources**
全局JNDI资源，该元素没有任何属性，所有的应用程序都可以引用全局JNDI资源。那什么是JNDI资源呢？JNDI多用于java的数据库连接中，到这里我们首先要想到的是JDBC，在JDBC中，连接数据库需要用户名、用户名密码和数据库名称等参数，将来如果这三个参数中的任何一个修改，则整个程序中相关的地方都需要修改，简直是牵一发而动全身，因此出现JNDI技术，在JNDI中，将连接数据库需要的用户、用户密码和数据库名称等JDBC引用的参数定义为一个整体（即这个整体中包含JDBC要用到类库、数据库驱动程序、用户、用户密码等），然后为这个整体设置一个名称，这样以后连接数据库的时候直接在程序中引用这个整体的名称即可，无论用户、用户密码怎么变换，只要这个整体的名称不变，我们仅要修改这个整体的用户和用户密码，而无需修改整个程序。这里的这个整体就是JNDI数据源（JNDI资源）。

虽然GlobalNamingResources没有定义任何的属性，但是可以在`<GlobalNamingResources>...</GlobalNamingResources>`中嵌套Resource元素。

```xml
<!‐‐ Global JNDI resources
Documentation at /docs/jndi‐resources‐howto.html
‐‐>
<GlobalNamingResources>
<!‐‐ Editable user database that can also be used by
UserDatabaseRealm to authenticate users
‐‐>
<Resource name="UserDatabase" auth="Container"
type="org.apache.catalina.UserDatabase"
description="User database that can be updated and saved"
factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
pathname="conf/tomcat‐users.xml" />
</GlobalNamingResources>
```

**Resource**
定义JNDI数据源

| 属性        | 备注                     | 默认值                       |
| ----------- | ------------------------ | ---------------------------- |
| name        | 定义的JNDI资源的名称     |                              |
| auth        | 指定管理Resource的管理器 | 有Container和Application两种 |
| type        | 使用的类名               |                              |
| description | 描述信息                 | ---                          |

示例：

```xml
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
```

示例表示定义了一个名为UserDataBase的全局JNDI数据源，使用容器管理该Resource，该数据源为加载tomcat-users.xml文件至内存中而定义的用于用户授权的数据库。然后在认证文件tomcat-users.xml中添加如下几行。表示用户名为admin，用户密码为centos的认证用户分别以manager-gui和admin-gui的角色登录manager和host-manager的应用程序。

![ln90hD.png](https://s2.ax1x.com/2019/12/28/ln90hD.png)

**Service**
包含一个或多个Connector和一个Engine的服务组件。属性name表示该Service的名称，默认保持 Catalina即可。`<Service>...</Service>`中可以嵌套Connector元素和Engine元素。

该元素用于创建 Service 实例,默认使用 org.apache.catalina.core.StandardService。默认情况下,Tomcat 仅指定了Service 的名称, 值为 "Catalina"。Service 可以内嵌的元素为 : Listener、Executor、Connector、Engine,其中 : Listener 用于为Service添加生命周期监听器, Executor 用于配置Service 共享线程池,Connector 用于配置Service 包含的链接器, Engine 用于配置Service中链接器对应的Servlet 容器引擎。

```xml
<Service name="Catalina">
...
</Service>
```

一个Server服务器,可以包含多个Service服务。

**Executor**

默认情况下,Service 并未添加共享线程池配置。 如果我们想添加一个线程池, 可以在下添加如下配置:

```xml
<Executor name="tomcatThreadPool"
namePrefix="catalina‐exec‐"
maxThreads="200"
minSpareThreads="100"
maxIdleTime="60000"
maxQueueSize="Integer.MAX_VALUE"
prestartminSpareThreads="false"
threadPriority="5"
className="org.apache.catalina.core.StandardThreadExecutor"/>
```

![t3i0vF.png](https://s1.ax1x.com/2020/05/31/t3i0vF.png)

如果不配置共享线程池,那么 Catalina 各组件在用到线程池时会独立创建。

**Connector**

Connector 用于创建链接器实例。默认情况下,server.xml 配置了两个链接器,一个支持HTTP协议,一个支持AJP协议。因此大多数情况下,我们并不需要新增链接器配置,只是根据需要对已有链接器进行优化。

```xml
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000"
redirectPort="8443" />
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

属性说明:

| 属性              | 备注                        | 属性值                                |
| ----------------- | --------------------------- | ------------------------------------- |
| accepCount        | 等待队列的长度              | 10                                    |
| port              | 连接器监听的端口            |                                       |
| protocol          | 连接器应用的协议类型        | http/https/ajp                        |
| connectionTimeout | 连接的超时时间，单位为毫秒  | 20000                                 |
| address           | 连接器监听的IP地址          | 默认为所有地址，0.0.0.0               |
| maxThreads        | 最大并发连接数              | http连接器默认200，https连接器默认150 |
| enableLookups     | 是否开启DNS反解             | true或false                           |
| clientAuth        | tomcat是否需要认证客户端    | 默认false                             |
| redirectPort      | 重定向至指定端口的Connector | 8443                                  |
| scheme            | 如果为SSL连接器，则为https  | http/https                            |
| secure            | SSL连接器则为true           | true或false                           |
| SSLEnabled        | 是否开启SSL                 | true或false                           |
| sslProtocol       | TLS                         |                                       |

其中ajp协议应用在Apache在反向代理tomcat的场景，Apache和tomcat之间就是使用ajp协议通信，它是一种二进制协议。

**Engine**

代表一个servlet实例，用于处理Connector接受并传递过来的请求。

Engine 作为Servlet 引擎的顶级元素,内部可以嵌入: Cluster、Listener、Realm、Valve和Host。

| 属性        | 备注                                                         | 属性值          |
| ----------- | ------------------------------------------------------------ | --------------- |
| name        | 用于指定Engine 的名称, 默认为Catalina 。该名称会影响一部分Tomcat的存储路径(如临时文件) | Catalina        |
| defaultHost | 默认使用的虚拟主机名称, 当客户端请求指向的主机无效时, 将交由默认的虚拟主机处理, 默认为localhost | 默认为localhost |
| jvmRoute    | 应用于tomcat集群中的session共享，会在一次会话中添加该值，获得session sticky | ---             |

**Host**
Host 元素用于配置一个虚拟主机, 它支持以下嵌入元素:Alias、Cluster、Listener、Valve、Realm、Context。如果在Engine下配置Realm, 那么此配置将在当前Engine下的所有Host中共享。 同样,如果在Host中配置Realm , 则在当前Host下的所有Context中共享。Context中的Realm优先级 > Host 的Realm优先级 > Engine中的Realm优先级。

```xml
<Host name="localhost"
appBase="webapps" unpackWARs="true"
autoDeploy="true">
...
</Host>
```

属性说明:

| 属性       | 备注                                                         | 属性值      |
| ---------- | ------------------------------------------------------------ | ----------- |
| name       | 当前Host通用的网络名称, 必须与DNS服务器上的注册信息一致。 Engine中包含的Host必须存在一个名称与Engine的defaultHost设置一致 |             |
| appBase    | 当前Host的应用基础目录, 当前Host上部署的Web应用均在该目录下(可以是绝对目录,相对路径)。默认为webapps | webapps     |
| unpackWARs | 设置为true, Host在启动时会将appBase目录下war包解压为目录。设置为false, Host将直接从war文件启动。 | true或false |
| autoDeploy | 控制tomcat是否在运行时定期检测并自动部署新增或变更的web应用，默认为true | true或false |

示例：
在Engine中新嵌套一个Host，也就是新增加一个虚拟主机，配置内容如下

```xml
      <Host name="Web1.linux.com" appBase="/data/webapps"
            unpackWARS="true" autoDeploy="true" />
```

创建appBase目录，并且tomcat虚拟主机中的每个应用程序都是单独放在一个目录内，因此创建ROOT目录,

```shell
[root@Web1 ~]# mkdir -p /data/webapps/ROOT
放入jsp测试页面
[root@Web1 ~]# vim /data/webapps/ROOT/index.jsp
<%@ page language="java" import="java.util.*" contentType="text/html; charset=utf-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
    <head>  
        <title>My JSP 'first.jsp' starting page</title>   
  </head>

  <body>
        <!-- 在JSP的页面中以表格的形式打印九九乘法表 -->
     <h1>九九乘法表</h1>
     <%
         for(int i=1;i<=9;i++) {
             for(int j=1;j<=i;j++) {
                 out.println(i+"*"+j+"="+(i*j)+"  ");
        }
        out.println("<br/>");
      }
      %>
    </body>
</html>
```

然后hosts文件中添加Web1.linux.com记录，浏览器中访问。

**Realm**
访问应用程序的时候需要安全认证，即需要输入用户和用户密码，Realm就是指定存储验证信息（用户和用户密码）的数据源。

```
<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
```

表示通过UserDatabaseRealm的方式获取验证信息，并且该数据源为GlobalNamingResources元素定义的全局JNDI资源UserDatebase。Realm元素嵌套的位置代表验证的范围，例如嵌套在Engine中，则Engine中所有的Host虚拟主机都共用这个认证信息。嵌套在Host中，则Host中所有的应用程序都共用这个认证信息。嵌套在Context中，则该应用程序使用这个认证信息。
**Valve**
可以嵌套在Engine、Host、Context元素中的组件，不同的类型可以实现特定的功能，例如AccessLogValve可以实现生成访问日志的功能，RemoteAddrValve可以实现控制远程IP地址的访问。
示例：

```
      <Host name="Web1.linux.com" appBase="/data/webapps"
            unpackWARS="true" autoDeploy="true">
<!--
    directory表示日志存放的目录，prefix和suffix分别表示生成的日志文件名的前缀和后缀，pattern表示记录日志的格式
-->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="Web1_access_log" suffix=".log"
               pattern="%h %l %u %t "%r" %s %b" />
        <Valve className="org.apache.catalina.valves.RemoteAddrValve" 
               deny="192.168.239.1" />
         </Host>
```

继续访问Web1.linux.com，结果没有返回页面内容，并且在logs目录下生成了访问日志。

**Context**
嵌套在Host中的元素，代表某个虚拟主机中的应用程序。

```xml
<Context docBase="myApp" path="/myApp">
....
</Context>
```

常用属性如下

| 属性       | 备注                                                         |
| ---------- | ------------------------------------------------------------ |
| docBase    | Web应用目录或者War包的部署路径。可以是绝对路径,也可以是相对于Host appBase的相对路径 |
| path       | 指定该应用程序映射为服务器根目录的URI路径，即host+path       |
| reloadable | 是否重新加载web应用程序类，默认为false                       |

注：tomcat虚拟主机中的每个应用程序必须单独存在一个目录中，docBase可以使用相对路径和绝对路径，相对路径是相对于Host虚拟主机的appBase目录。

如果path属性值为空字符串，则表示该应用程序为根web应用，即ROOT目录。

示例：

```
<Host name="Web1.linux.com" appBase="/data/webapps"
            unpackWARS="true" autoDeploy="true">
    <Context path="/test" docBase="test" reloadable="true" />
</Host>
```

然后创建web应用程序的jsp代码，URL地址列中输入http://web1.linux.com:8080/test ，查看访问结果。

```shell
[root@Web1 ~]# mkdir -p /data/webapps/test
[root@Web1 test]# vim index.jsp 

<HTML>
    <HEAD>
        <TITLE>JSP test</TITLE>
    </HEAD>
    <BODY>
        <%
        out.println("<h1>Hello World!!</h1>");
        %>
    </BODY>
</HTML>
```

它支持的内嵌元素为:CookieProcessor, Loader, Manager,Realm,Resources,WatchedResource,JarScanner,Valve。

```xml
<Host name="www.tomcat.com" appBase="webapps" unpackWARs="true" autoDeploy="true">
	<Context docBase="D:\servlet_project03" path="/myApp"></Context>
    <Valve className="org.apache.catalina.valves.AccessLogValve"
    directory="logs"
    prefix="localhost_access_log" suffix=".txt"
    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

### Web 应用配置

web.xml 是web应用的描述文件, 它支持的元素及属性来自于Servlet 规范定义 。 在Tomcat 中, Web 应用的描述信息包括 tomcat/conf/web.xml 中默认配置 以及 Web应用 WEB-INF/web.xml 下的定制配置。

1. ServletContext 初始化参数

   我们可以通过 添加ServletContext 初始化参数,它配置了一个键值对,这样我们可以在应用程序中使用 javax.servlet.ServletContext.getInitParameter()方法获取参数。

   ```xml
   <context‐param>
       <param‐name>contextConfigLocation</param‐name>
       <param‐value>classpath:applicationContext‐*.xml</param‐value>
       <description>Spring Config File Location</description>
   </context‐param>
   ```

2. 会话配置
   用于配置Web应用会话,包括 超时时间、Cookie配置以及会话追踪模式。它将覆盖server.xml 和 context.xml 中的配置。

   ```xml
   <session‐config>
       <session‐timeout>30</session‐timeout>
       <cookie‐config>
           <name>JESSIONID</name>
           <domain>www.pingxin.cn</domain>
           <path>/</path>
           <comment>Session Cookie</comment>
           <http‐only>true</http‐only>
           <secure>false</secure>
           <max‐age>3600</max‐age>
       </cookie‐config>
       <tracking‐mode>COOKIE</tracking‐mode>
   </session‐config>
   ```

   配置解析:

   ```
   1 ) session‐timeout : 会话超时时间,单位 分钟
   2) cookie‐config: 用于配置会话追踪Cookie
   name:Cookie的名称
   domain:Cookie的域名
   path:Cookie的路径
   comment:注释
   http‐only:cookie只能通过HTTP方式进行访问,JS无法读取或修改,此项可以增
   加网站访问的安全性。
   secure:此cookie只能通过HTTPS连接传递到服务器,而HTTP 连接则不会传递该
   信息。注意是从浏览器传递到服务器,服务器端的Cookie对象不受此项影响。
   max‐age:以秒为单位表示cookie的生存期,默认为‐1表示是会话Cookie,浏览器
   关闭时就会消失。
   3) tracking‐mode :用于配置会话追踪模式,Servlet3.0版本中支持的追踪模式:
   COOKIE、URL、SSL
   A. COOKIE : 通过HTTP Cookie 追踪会话是最常用的会话追踪机制, 而且
   Servlet规范也要求所有的Servlet规范都需要支持Cookie追踪。
   B. URL : URL重写是最基本的会话追踪机制。当客户端不支持Cookie时,可以采
   用URL重写的方式。当采用URL追踪模式时,请求路径需要包含会话标识信息,Servlet容器
   会根据路径中的会话标识设置请求的会话信息。如:
   http://www.myserver.com/user/index.html;jessionid=1234567890。
   C. SSL : 对于SSL请求, 通过SSL会话标识确定请求会话标识。
   ```

3. Servlet配置
   Servlet 的配置主要是两部分,servlet 和 servlet-mapping :

   ```xml
   <servlet>
       <servlet‐name>myServlet</servlet‐name>
       <servlet‐class>cn.px.web.MyServlet</servlet‐class>
       <init‐param>
           <param‐name>fileName</param‐name>
           <param‐value>init.conf</param‐value>
       </init‐param>
       <load‐on‐startup>1</load‐on‐startup>
       <enabled>true</enabled>
   </servlet>
   <servlet‐mapping>
       <servlet‐name>myServlet</servlet‐name>
       <url‐pattern>*.do</url‐pattern>
       <url‐pattern>/myservet/*</url‐pattern>
   </servlet‐mapping>
   ```

   配置说明:

   > 1 ) servlet‐name : 指定servlet的名称, 该属性在web.xml中唯一。
   > 2) servlet‐class : 用于指定servlet类名
   > 3) init‐param: 用于指定servlet的初始化参数, 在应用中可以通过HttpServlet.getInitParameter 获取。
   > 4) load‐on‐startup: 用于控制在Web应用启动时,Servlet的加载顺序。 值小于0,web应用启动时,不加载该servlet, 第一次访问时加载。
   > 5) enabled: true , false 。 若为false ,表示Servlet不处理任何请求。
   > 6) url‐pattern: 用于指定URL表达式,一个 servlet‐mapping可以同时配置多个 url‐pattern。

   Servlet 中文件上传配置:

   ```xml
   <servlet>
       <servlet‐name>uploadServlet</servlet‐name>
       <servlet‐class>cn.px.web.UploadServlet</servlet‐class>
       <multipart‐config>
           <location>C://path</location>
           <max‐file‐size>10485760</max‐file‐size>
           <max‐request‐size>10485760</max‐request‐size>
           <file‐size‐threshold>0</file‐size‐threshold>
       </multipart‐config>
   </servlet>
   ```

   配置说明：

   ```
   1 ) location:存放生成的文件地址。
   2) max‐file‐size:允许上传的文件最大值。 默认值为‐1, 表示没有限制。
   3) max‐request‐size:针对该 multi/form‐data 请求的最大数量,默认值为‐1, 表示
   无限制。
   4) file‐size‐threshold:当数量量大于该值时, 内容会被写入文件
   ```

4. Listener配置

   Listener用于监听servlet中的事件,例如context、request、session对象的创建、修改、删除,并触发响应事件。Listener是观察者模式的实现,在servlet中主要用于对context、request、session对象的生命周期进行监控。在servlet2.5规范中共定义了8中Listener。在启动时,ServletContextListener 的执行顺序与web.xml 中的配置顺序一致, 停止时执行顺序相反。

   ```xml
   <listener>
   <listener‐
   class>org.springframework.web.context.ContextLoaderListener</listener‐class>
   </listener>
   ```

5. Filter配置

   filter 用于配置web应用过滤器, 用来过滤资源请求及响应。 经常用于认证、日志、加密、数据转换等操作, 配置如下:

   ```xml
   <filter>
       <filter‐name>myFilter</filter‐name>
       <filter‐class>cn.px.web.MyFilter</filter‐class>
       <async‐supported>true</async‐supported>
       <init‐param>
           <param‐name>language</param‐name>
           <param‐value>CN</param‐value>
       </init‐param>
   </filter>
   <filter‐mapping>
       <filter‐name>myFilter</filter‐name>
       <url‐pattern>/*</url‐pattern>
   </filter‐mapping>
   ```

   配置说明:

   > 1 ) filter‐name: 用于指定过滤器名称,在web.xml中,过滤器名称必须唯一。
   > 2) filter‐class : 过滤器的全限定类名, 该类必须实现Filter接口。
   > 3) async‐supported: 该过滤器是否支持异步
   > 4) init‐param :用于配置Filter的初始化参数, 可以配置多个, 可以通过FilterConfig.getInitParameter获取
   > 5) url‐pattern: 指定该过滤器需要拦截的URL。

6. 欢迎页面配置

   welcome-file-list 用于指定web应用的欢迎文件列表。

   ```
   <welcome‐file‐list>
   <welcome‐file>index.html</welcome‐file>
   <welcome‐file>index.htm</welcome‐file>
   <welcome‐file>index.jsp</welcome‐file>
   </welcome‐file‐list>
   ```

   尝试请求的顺序,从上到下。

7. 错误页面配置

   error-page 用于配置Web应用访问异常时定向到的页面,支持HTTP响应码和异常类两种形式。

   ```xml
   <error‐page>
       <error‐code>404</error‐code>
       <location>/404.html</location>
   </error‐page>
   <error‐page>
       <error‐code>500</error‐code>
       <location>/500.html</location>
   </error‐page>
   <error‐page>
       <exception‐type>java.lang.Exception</exception‐type>
       <location>/error.jsp</location>
   </error‐page>
   ```

### 常用配置


#### 修改虚拟主机

修改${CATALINA_BASE}/conf/server.xml

```
  <!--appBase是根目录。localhost是名字，对应conf\contalina\localhost-->
...
<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">  
...
</Host>
...
```

比如设置appBase为/web/www，在/web/www/hello/新建一个index.html，随便写一些文字。

运行tomcat，在浏览器使用`http://localhost:8080/hello/`,就可以看到你在那个文件写的文字。


#### 增加虚拟目录

web应用程序指供浏览器访问的程序，通常也简称为web应用。
web应用：例如有a.html 、b.html…..多个web资源，这多个web资源用于对外提供邮件服务，此时应把这多个web资源放在一个目录中，以组成一个web应用（或web应用程序）。
一个web应用由多个静态web资源和动态web资源组成，如
```
html、css、js文件
jsp文件、java程序、支持jar包
配置文件
…
```

组成web应用的这些文件通常我们会使用一个目录组织，这个目录称之为web应用所在目录。

web应用开发好后，若想供外界访问，需要把web应用所在目录交给web服务器管理，这个过程称之为虚似目录的映射。那么在Tomcat服务器中，如何进行虚拟目录的映射呢？总共有如下的几种方式：

**方式一：配置server.xml文件**

修改${CATALINA_BASE}/conf/server.xml
```
Tomcat----conf文件夹----server.xml----Host节点----添加

 <Context path="访问时的路径（虚拟目录）" docBase="在磁盘上的真实路径"  />
```

项目不用必须放在webapps下，可以放在任何地方。修改server.xml需要重启Tomcat服务器

在<Host\></Host\>这对标签加上`  <Context path="/mail" docBase="/web/www/mailweb" />`，即可以访问/web/www/目录下的项目mailweb，这个JavaWeb应用映射到mail这个虚拟目录上，mail这个虚拟目录是由Tomcat服务器管理的，mail是一个硬盘上不存在的目录，是我们自己随便写的一个目录，也就是虚拟的一个目录，所以称之为”虚拟目录”，代码如下：

```（xml）
<Host name="localhost"  appBase="webapps"
      unpackWARs="true" autoDeploy="true">

  <!-- SingleSignOn valve, share authentication between web applications
       Documentation at: /docs/config/valve.html -->
  <!--
  <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
  -->

  <!-- Access log processes all example.
       Documentation at: /docs/config/valve.html
       Note: The pattern used is equivalent to using pattern="common" -->
  <Context path="/mail" docBase="/web/www/mailweb" />

</Host>
```

其中，Context表示上下文，代表的就是一个JavaWeb应用，Context元素有两个属性：

path：用来配置虚拟目录，必须以”/”开头。
docBase：配置此虚拟目录对应着硬盘上的Web应用所在目录。

现在可以通过`http://localhost:8080/mail`访问web应用/web/www/mailweb。


**方式二（不用修改server.xml文件，推荐使用此方法）**

每修改一次server.xml，就得重启Tomcat服务器，为了解决这个问题：

在 conf----contalina----localhost----新建xml文件----将 <Context docBase="在磁盘上的真实路径"  /\>放在新建的xml文件中，server.xml就不用写这句话了。

注意：此时不用写path了，xml文件的名字就是path（文件名 会自动映射为 虚拟目录 ），访问时，在后边加xml文件的名字即可访问。

mail.xml文件中的内容如下：
```
<Context docBase="/web/www/mailweb" />
```

此时tomcat服务器会自动检测到添加的这个xml文件，并部署上相应的web应用，即部署上mailweb这个web应用。在浏览器中输入`http://localhost:8080/mailweb/hello.html`，则tomcat会自动找到/web/www/下的名为mailweb的web应用。


*附：context元素的常用属性*

属性说明：

- crossContext  如果想在应用内调用ServletContext.getContext()来返回在该虚拟主机上运行的其他web application的request dispatcher,设为true。在安全性很重要的环境中，设为false，使得getContext()总是返回null。缺省值为false。
- docBase         该web应用的文档基准目录（Document Base，也称为Context Root），或者是WAR文件的路径。可以使用绝对路径，也可以使用相对于context所属的Host的appBase路径。
- override         如果想利用该Context元素中的设置覆盖DefaultContext中相应的设置，设为true。缺省情况下为false，使用DefaultContext中的设置。
- path              web应用的context路径。catalina将每个URL的起始和context path进行比较，选择合适的web应用处理该请求。特定Host下的context path必须是惟一的。如果context path为空字符串（""），这个context是所属Host的缺省web应用
- reloadable      如果希望Catalina监视/WEB-INF/classes/和/WEB-INF/lib下面的类是否发生变化，在发生变化的时候自动重载web application，设为true。这个特征在开发阶段很有用，但也大大增加了服务器的开销。因此，在发布以后，不推荐使用。
- debug           是否为调试模式，如果在开发时显示调试信息，设为true。

#### log管理

日志配置位于${CATALINA_BASE}/conf/logging.properties 。
以下文件都是在tomcat启动时自动生成的日志文件，按照日期自动备份
```
localhost.log　　程序异常没有被捕获的时候抛出的地方【常用】
catalina.log　　程序的输出、tomcat的日志输出【常用】
localhost_access_log.txt　　tomcat访问日志记录【常用】
manager.log　　webapps/manager项目生成的日志文件
host-manager.log　　webapps/host-manager项目生成的日志文件
```

**tomcat日志分为五类：**
```
catalina 、 localhost 、 manager 、 admin 、 host-manager
```
 tomcat 其实还有记录访问日志（localhost_access_log.txt），配置位于${CATALINA_BASE}/conf/server.xml的Host节点中的Value内。

**tomcat日志还分有级别：**

```
SEVERE (highest value) >WARNING >INFO >CONFIG >FINE >FINER >FINEST (lowest value)
```

**日志级别的修改：**

 修改${CATALINA_BASE}/conf/logging.properties文件
示例：
1. 设置 catalina 日志的级别为： FINE
1catalina.org.apache.juli.FileHandler.level = FINE
2. 禁用 catalina 日志的输出：
1catalina.org.apache.juli.FileHandler.level = OFF
3. 输出 catalina 所有的日志消息：
1catalina.org.apache.juli.FileHandler.level = ALL

**网络访问log**

在开发/测试环境，日志是非常重要的。而公司对于测试环境进行了控制，只有配置人员能连接访问，而开发人员是无法获取该服务器的信息的。在出现错误时，没有异常日志，开发是很难重现问题的。因此需要对中间件 tomcat 进行配置，将日志放到某个目录下，开发人员可以通过浏览器就能查看日志。

目的： 能通过浏览器检查tomcat日志

这样利用上面介绍的方法设置虚拟目录，映射到${CATALINA_BASE}/logs下。

使用方式二，在 conf----contalina----localhost----新建xml文件----将 <Context docBase="在磁盘上的真实路径"  />放在新建的xml文件。

log.xml文件内容如下：
```
<Context docBase="${CATALINA_BASE}/logs" />
```

这样可以打开`http://127.0.0.1/logs/catalina.out`。


#### 设置管理员

Tomcat Manager是Tomcat自带的、用于对Tomcat自身以及部署在Tomcat上的应用进行管理的web应用。Tomcat是Java领域使用最广泛的服务器之一，因此Tomcat Manager也成为了使用非常普遍的功能应用。

在默认情况下，Tomcat Manager是处于禁用状态的。准确地说，Tomcat Manager需要以用户角色进行登录并授权才能使用相应的功能，不过Tomcat并没有配置任何默认的用户，因此需要我们进行相应的用户配置之后才能使用Tomcat Manager。

要想配置管理员用户名和密码，需要修改tomcat安装文件下的conf中的tomcat-user.xml文件。
在配置文件<tomcat-users\>节点下添加如下xml

```（xml）
<role rolename="manager"/>　  
<role rolename="manager-gui"/>　  
<role rolename="admin"/>　  
<role rolename="admin-gui"/>　  
<user username="admin" password="admin" roles="admin-gui,admin,manager-gui,manager"/>　  
```

如上所示，我们只需要在tomcat-users节点中配置相应的role(角色/权限)和user(用户)即可。一个user节点表示单个用户，属性username和password分别表示登录的用户名和密码，属性roles表示该用户所具备的权限。

user节点的roles属性值与role节点的rolename属性值相对应，表示当前用户具备该role节点所表示的角色权限。当然，一个用户可以具备多种权限，因此属性roles的值可以是多个rolename，多个rolename之间以英文逗号隔开即可。

稍加思考，我们就应该猜测到，rolename的属性值并不是随意的内容，否则Tomcat怎么能够知道我们随便定义的rolename表示什么样的权限呢。实际上，Tomcat已经为我们定义了4种不同的角色——也就是4个rolename，我们只需要使用Tomcat为我们定义的这几种角色就足够满足我们的工作需要了。

以下是Tomcat Manager 4种角色的大致介绍(下面URL中的*为通配符)：

```
manager-gui
允许访问html接口(即URL路径为/manager/html/*)
manager-script
允许访问纯文本接口(即URL路径为/manager/text/*)
manager-jmx
允许访问JMX代理接口(即URL路径为/manager/jmxproxy/*)
manager-status
允许访问Tomcat只读状态页面(即URL路径为/manager/status/*)
```
从Tomcat Manager内部配置文件中可以得知，manager-gui、manager-script、manager-jmx均具备manager-status的权限，也就是说，manager-gui、manager-script、manager-jmx三种角色权限无需再额外添加manager-status权限，即可直接访问路径`/manager/status/*`.

#### APR模式

Tomcat 有三种(bio,nio.apr) 运行模式，首先来简单介绍下

**bio** 
bio(blocking  I/O)，顾名思义，即阻塞式I/O操作，表示Tomcat使用的是传统的Java  I/O操作(即java.io包及其子包)。Tomcat在默认情况下，就是以bio模式运行的。遗憾的是，就一般而言，bio模式是三种运行模式中性能最低的一种。我们可以通过Tomcat  Manager来查看服务器的当前状态。

**nio** 
是[Java SE](http://lib.csdn.net/base/javase) 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java  nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking  I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。

想运行在该模式下，直接修改server.xml里的Connector节点,修改protocol为 

```
<Connector port="80" protocol="org.apache.coyote.http11.Http11NioProtocol" 
	connectionTimeout="20000" 
	URIEncoding="UTF-8" 
	useBodyEncodingForURI="true" 
	enableLookups="false" 
	redirectPort="8443" /> 
```

**apr** 
(Apache  Portable Runtime/Apache可移植运行库)，是Apache  HTTP服务器的支持库。你可以简单地理解为，Tomcat将以JNI的形式调用Apache  HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。 Tomcat  apr也是在Tomcat上运行高并发应用的首选模式。

要tomcat支持apr，必须要安装apr和native，这样tomcat可以利用apache的apr接口，使用操作系统的部分本地操作，从而提升性能。

Tomcat的运行模式有3种.修改他们的运行模式.3种模式的运行是否成功,可以看他的启动控制台,或者启动日志.或者登录他们的默认页面http://localhost:8080/查看其中的服务器状态。 

#### Docker 安装 Tomcat

前提是配置好docker环境，具体参考docker文章。

```
$ docker pull tomcat 
$ docker images  tomcat              
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              latest              238e6d7313e3        12 days ago         506MB

```

运行容器

```
$ docker run --name tomcat -p 8080:8080  -v ~/docker/tomcat/webapps:/usr/local/tomcat/webapps -v ~/docker/tomcat/conf:/usr/local/tomcat/conf -v ~/docker/tomcat/logs:/usr/local/tomcat/logs -d tomcat
```

命令说明：

**-p 8080:8080：**将容器的8080端口映射到主机的8080端口

**-v ~/docker/tomcat/test:/usr/local/tomcat/webapps/test：**将主机中当前目录下的test挂载到容器的/test

查看容器启动情况

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                    NAMES
382ed6d38bf9        tomcat              "catalina.sh run"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   tomcat

```

浏览器访问http://127.0.0.1:8080

**将war 包上传部署到tomcat容器**

```bash
$ docker cp postgres-BUILD-SNAPSHOT.war 382ed6d38bf9:/usr/local/tomcat/webapps
// 重启容器
$ docker restart 382ed6d38bf9
```

访问服务

```
# 访问格式：浏览器访问宿主机IP：8080/war包名（即项目名）/接口名
http://localhost:8080/postgres-BUILD-SNAPSHOT/home
```

更复杂的情况请参看其他docker文章。

#### Tomcat支持HTTPS

1. 生成秘钥库文件。

   ```
   keytool ‐genkey ‐alias tomcat ‐keyalg RSA ‐keystore tomcatkey.keystore
   ```

   输入对应的密钥库密码, 秘钥密码等信息之后,会在当前文件夹中出现一个秘钥库文件:tomcatkey.keystore

2. 将秘钥库文件 tomcatkey.keystore 复制到tomcat/conf 目录下。

3. 配置tomcat/conf/server.xml

   ```xml
   <Connector port="8443"protocol="org.apache.coyote.http11.Http11NioProtocol"
   maxThreads="150" schema="https" secure="true" SSLEnabled="true">
       <SSLHostConfig certificateVerification="false">
           <Certificate
           certificateKeystoreFile="D:/DevelopProgramFile/apache‐tomcat‐8.5.42‐
           windows‐x64/apache‐tomcat‐8.5.42/conf/tomcatkey.keystore"
           certificateKeystorePassword="pingxin"
           type="RSA" />
       </SSLHostConfig>
   </Connector>
   ```

4. 访问Tomcat ,使用https协议。

