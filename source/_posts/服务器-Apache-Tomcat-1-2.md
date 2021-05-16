---
title: Apache Tomcat 优化
date: 2020-05-09 11:18:59
tags:
 - 服务器
 - Tomcat
categories:
 - 服务器
 - Tomcat
---
tomcat服务器在JavaEE项目中使用率非常高，所以在生产环境对tomcat的优化也变得非常重要了

<!--more-->

### 性能测试

对于系统性能,用户最直观的感受就是系统的加载和操作时间,即用户执行某项操作的耗时。从更为专业的角度上讲,性能测试可以从以下两个指标量化。

1. 响应时间:如上所述,为执行某个操作的耗时。大多数情况下,我们需要针对同一个操作测试多次,以获取操作的平均响应时间。
2. 吞吐量:即在给定的时间内,系统支持的事务数量,计算单位为 TPS。

通常情况下,我们需要借助于一些自动化工具来进行性能测试,因为手动模拟大量用户的并发访问几乎是不可行的,而且现在市面上也有很多的性能测试工具可以使用,如:ApacheBench、ApacheJMeter、WCAT、WebPolygraph、LoadRunner。

#### ApacheBench

ApacheBench (ab)是一款ApacheServer基准的测试工具,用户测试Apache Server的服务能力(每秒处理请求数),它不仅可以用户Apache的测试,还可以用于测试Tomcat、Nginx、lighthttp、IIS等服务器。

1. 安装

   ```bash
   yum install httpd‐tools
   #或
   sudo apt install apache2-utils
   
   $ ab -V                                       
   This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
   Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
   Licensed to The Apache Software Foundation, http://www.apache.org/
   
   ```

2. 测试性能

   ![t3Vl0e.png](https://s1.ax1x.com/2020/05/31/t3Vl0e.png)

   结果说明

   ![t3ZmNj.png](https://s1.ax1x.com/2020/05/31/t3ZmNj.png)

   重要指标

   ![t3Z0v6.png](https://s1.ax1x.com/2020/05/31/t3Z0v6.png)

   

   

## 优化

对于tomcat的优化，主要是从2个方面入手，一是，tomcat自身的配置，另一个是tomcat所运行的jvm虚拟机的调优。

### tomcat自身优化

tomcat 9.0.34

开启status页面管理

```shell
#修改配置文件，配置tomcat的管理用户
vim conf/tomcat-users.xml
#写入如下内容：
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="admin"/>
<role rolename="admin-gui"/>
<user username="tomcat" password="tomcat" roles="admin-gui,admin,manager-gui,manager"/>
#保存退出

#如果是tomcat7，配置了tomcat用户就可以登录系统了，但是tomcat8中不行，还需要修改另一个配置文件，否则访问不了，提示403
vim webapps/manager/META-INF/context.xml

#将<Valve的内容注释掉
<Context antiResourceLocking="false" privileged="true" >
 <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
#保存退出即可


```

查看服务器状态：<http://xxx/manager/status>

![Y1Zz8J.png](https://s1.ax1x.com/2020/05/09/Y1Zz8J.png)

#### 禁用AJP连接

在服务状态页面中可以看到，默认状态下会启用AJP服务，并且占用8009端口。

![Y1ezo8.png](https://s1.ax1x.com/2020/05/09/Y1ezo8.png)

什么是AJP呢？

AJP（Apache JServer Protocol）
 AJPv13协议是面向包的。WEB服务器和Servlet容器通过TCP连接来交互；为了节省SOCKET创建的昂贵代价，WEB服务器会尝试维护一个永久TCP连接到servlet容器，并且在多个请求和响应周期过程会重用连接。

![Y1m9Jg.png](https://s1.ax1x.com/2020/05/09/Y1m9Jg.png)

我们一般是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用。

修改conf下的server.xml文件，将AJP服务禁用掉即可。

```
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

重启tomcat，查看效果。可以看到AJP服务以及不存在了。

#### 3种运行模式

tomcat的运行模式有3种：

1. bio
    默认的模式,性能非常低下,没有经过任何优化处理和支持.
2. nio
    nio(new I/O)，是Java SE 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。
3. apr
    安装起来最困难,但是从操作系统级别来解决异步的IO问题,大幅度的提高性能.

推荐使用nio，不过，在tomcat8中有最新的nio2，速度更快，建议使用nio2.

设置nio2：

```xml
<Connector executor="tomcatThreadPool"  port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
               connectionTimeout="20000"
               redirectPort="8443" />
```

##### apr

1. 安装软件

   ```shell
   apt-get install libapr1-dev libssl-dev gcc make libexpat1-dev libtool
   cd /usr/local/src
   wget  https://mirrors.cnnic.cn/apache/apr/apr-1.6.3.tar.gz
   tar xf apr-1.6.3.tar.gz
   cd apr-1.6.3/
   ./configure --prefix=/usr/local/apr
   make && make install
       
   cd /usr/local/src
   wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
   ar -xzxf openssl-1.1.0h.tar.gz
   cd openssl-1.1.0h/
   ./config --prefix=/usr/local/openssl -fPIC
   // 注意这里需要加入 -fPIC参数，否则后面在安装tomcat native 组件会出错
   // 注意：不要按照提示去运行 make depend
   make && make install
   #openssl安装完成继续更新系统环境
   #修改历史的OpenSSL文件设置备份
   mv /usr/bin/openssl /usr/bin/openssl.old
   mv /usr/include/openssl /usr/include/openssl.old
   #设置软连接使其使用新的OpenSSL版本 刚刚安装的OpenSSL默认安装在/usr/local/ssl
   ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
   ln -s /usr/local/openssl/include/openssl /usr/include/openssl 
   #更新动态链接库数据
   echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
   ldconfig -v
   我们再来看看 OpenSSL 版本信息 openssl version,如为1.0.2则成功升级openssl
    
   cd /usr/local/src
   wget https://mirrors.cnnic.cn/apache/apr/apr-iconv-1.2.2.tar.gz
   tar xf apr-iconv-1.2.2.tar.gz
   cd apr-iconv-1.2.2/
   ./configure   --with-apr=/usr/local/apr  --prefix=/usr/local/apr-iconv
   make && make install
    
   cd /usr/local/src
   wget  https://mirrors.cnnic.cn/apache/apr/apr-util-1.6.1.tar.gz
   tar xf apr-util-1.6.1.tar.gz  
   cd apr-util-1.6.1/
   ./configure --prefix=/usr/local/apr-util  --with-apr=/usr/local/apr   --with-apr-iconv=/usr/local/apr-iconv/bin/apriconv
   make && make install
    
   cd /usr/local/tomcat/bin/
   tar xf tomcat-native.tar.gz
   cd  /usr/local/tomcat/bin/tomcat-native-1.2.16-src/native
   ./configure --with-apr=/usr/local/apr  --with-java-home=/usr/local/jdk8.0     
   make && make install
    
   echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib
   export LD_RUN_PATH=$LD_RUN_PATH:/usr/local/apr/lib' >> /etc/profile
    
   source /etc/profile
   ```

2. 修改配置

   ```xml
   vim conf/server.xml
   
   <Connector      port="80"        protocol="org.apache.coyote.http11.Http11AprProtocol"
                   maxThreads="1000"
                   minSpareThreads="100"
                   acceptCount="900"
                   disableUploadTimeout="true"
                   connectionTimeout="20000"
                   URIEncoding="UTF-8"
                   enableLookups="false"
                   redirectPort="8443"
                   compression="on"
                   compressionMinSize="1024"
                  />
   ```

   

#### 线程优化

具体的线程优化可以参照tomcat文档，目录在 `\webapps\docs\config\http.html`  这里只介绍几个重要的参数。由于是tomcat8+ ，默认线程连接使用的是NIO或NIO2。

|       参数        |                             解释                             |
| :---------------: | :----------------------------------------------------------: |
| `maxConnections`  | 服务器所能接受最大的请求和处理的连接数，NIO和NIO2默认为10000，APR默认是8192. |
|   `maxThreads`    |               同一时间点上处理线程的最大数量。               |
|   `acceptCount`   | 当所有可能的请求处理线程都在使用时，传入连接请求的最大队列长度。 队列已满时收到的任何请求都将被拒绝。 默认值为100。 |
| `minSpareThreads` |             最小空闲线程数，默认为10，不易过小。             |

主要的线程优化就这些，还有其他不重要的可以自己看文档。

我们在`\conf\server.xml`中可以看到 把参数添加到如下的配置中即可。

```xml
<!--将注释打开-->
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="500" minSpareThreads="50" prestartminSpareThreads="true" maxQueueSize="100"/>
<!--
参数说明：
maxThreads：最大并发数，默认设置 200，一般建议在 500 ~ 1000，根据硬件设施和业务来判断
minSpareThreads：Tomcat 初始化时创建的线程数，默认设置 25
prestartminSpareThreads： 在 Tomcat 初始化的时候就初始化 minSpareThreads 的参数值，如果不等于 true，minSpareThreads 的值就没啥效果了
maxQueueSize，最大的等待队列数，超过则拒绝请求
-->

<!--在Connector中设置executor属性指向上面的执行器-->
<Connector executor="tomcatThreadPool"  port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

```

保存退出，重启tomcat，查看效果。

#### 配置优化

有时候只有线程优化也是不够的。我们要了解tomcat的其他配置才可以做到完美。

**参数配置**

|     参数     |                             解释                             |          对应文档位置          |
| :----------: | :----------------------------------------------------------: | :----------------------------: |
| `autoDeploy` | 当设置为true时，tomcat会在启动时，定时的去检查是否有新的项目更新。默认为true。因此我们需要将这个参数设置为false。 | /webapps/docs/config/host.html |

我们同样需要在`\conf\server.xml`中配置。

![Y1ewq0.png](https://s1.ax1x.com/2020/05/09/Y1ewq0.png)

### JVM优化

JVM内存参数:

![t3ZRPA.png](https://s1.ax1x.com/2020/05/31/t3ZRPA.png)

#### 设置并行垃圾回收器

```
#年轻代、老年代均使用并行收集器，初始堆内存64M，最大堆内存512M
JAVA_OPTS="-XX:+UseParallelGC -XX:+UseParallelOldGC -Xms64m -Xmx512m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"
```

#### 查看gc日志文件

将gc.log文件上传到gceasy.io查看gc中是否存在问题。

如果gc次数过多，尝试扩充年轻代大小

#### 调整年轻代大小

```
JAVA_OPTS="-XX:+UseParallelGC -XX:+UseParallelOldGC -Xms128m -Xmx1024m -XX:NewSize=64m -XX:MaxNewSize=256m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"
```

将初始堆大小设置为128m，最大为1024m

初始年轻代大小64m，年轻代最大256m

#### 设置G1垃圾回收器

```
#设置了最大停顿时间100毫秒，初始堆内存128m，最大堆内存1024m
JAVA_OPTS="-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -Xms128m -Xmx1024m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:../logs/gc.log"
```

#### 系统参数优化

优化网卡驱动可以有效提升性能，这个对于集群环境工作的时候尤为重要。由于我们采用了linux服务器，所以优化内核参数也是一个非常重要的工作。给一个参考的优化参数：

1. 修改/etc/sysctl.cnf文件，在最后追加如下内容：

   ```
   net.core.netdev_max_backlog = 32768
    net.core.somaxconn = 32768
    net.core.wmem_default = 8388608
    net.core.rmem_default = 8388608
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.ipv4.ip_local_port_range = 1024 65000 net.ipv4.route.gc_timeout = 100
    net.ipv4.tcp_fin_timeout = 30
    net.ipv4.tcp_keepalive_time = 1200
    net.ipv4.tcp_timestamps = 0
    net.ipv4.tcp_synack_retries = 2
    net.ipv4.tcp_syn_retries = 2
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_mem = 94500000 915000000 927000000 net.ipv4.tcp_max_orphans = 3276800
   net.ipv4.tcp_max_syn_backlog = 65536
   ```

2.  保存退出，执行sysctl -p生效

### 使用 Undertow 来替代Tomcat

> Undertow 是红帽公司开发的一款**基于 NIO 的高性能 Web 嵌入式服务器**

Spring Boot项目中的引入方式 :

```
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
<!--替换内置默认容器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
            <version>1.5.10.RELEASE</version>
        </dependency>
```

配置：

```properties
# Undertow 日志存放目录
server.undertow.accesslog.dir=
# 是否启动日志
server.undertow.accesslog.enabled=false 
# 日志格式
server.undertow.accesslog.pattern=common
# 日志文件名前缀
server.undertow.accesslog.prefix=access_log
# 日志文件名后缀
server.undertow.accesslog.suffix=log
# HTTP POST请求最大的大小
server.undertow.max-http-post-size=0 
# 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接, 默认设置每个CPU核心一个线程
server.undertow.io-threads=4
# 阻塞任务线程池, 当执行类似servlet请求阻塞操作, undertow会从这个线程池中取得线程,它的值设置取决于系统的负载
server.undertow.worker-threads=20
# 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理
# 每块buffer的空间大小,越小的空间被利用越充分
server.undertow.buffer-size=1024
# 每个区分配的buffer数量 , 所以pool的大小是buffer-size * buffers-per-region
server.undertow.buffers-per-region=1024
# 是否分配的直接内存
server.undertow.direct-buffers=true
```



### 参考

1. [Tomcat优化（内存，并发，缓存，安全，网络，系统等）](https://cloud.tencent.com/developer/article/1444703)
2. [Tomcat优化性能调优及代码优化建议](https://www.jianshu.com/p/ca6ed42ec561)
3. <https://www.jianshu.com/p/cf605092817d>
4. [ubuntu中的tomcat使用apr模式](https://blog.csdn.net/wokuailewozihao/article/details/81478239)
5. [系统优化怎么做-Tomcat优化](https://segmentfault.com/a/1190000015918707)