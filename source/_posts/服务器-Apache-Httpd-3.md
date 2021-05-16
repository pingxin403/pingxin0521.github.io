---
title: Apache Httpd配置tomcat集群
date: 2020-06-01 11:18:59
tags:
 - 服务器
 - Httpd
 - Tomcat
categories:
 - 服务器
 - Apache Httpd
---

#### 负载均衡

所谓负载均衡(loadbalance)所指的是,在服务器端短时间内获得大量的请求,单一服务器无法在一个较短的时间内响应这些请求,此时服务器需要一个机制,请求按照多个服务器不同的负载能力,把这些请求合理的分配。

<!--more-->

集群(cluster)的作用则是在多个服务器之间共享用户信息,资源等。

关于集群的优点不多说了,直接开始主题,怎样配置一个tomcat的集群.

apache2.4以上的版本对ajp的链接器而且性能比原来的jk有的大的提升所以推荐使用ajp。先下载安装apache2.4(目前最新版),httpd.conf增加如下配置,去掉注释就行了

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

如果没有这些模块,可以使用前面的动态加载 SO 的方法进行重新加载。

#### 配置Apache

然后在httpd.conf最后增加下面增加如下的配置

```
ProxyRequests Off
ProxyPass / balancer://tomcatcluster/ lbmethod=byrequests
stickysession=JSESSIONID nofailover=Off timeout=5 maxattempts=3
ProxyPassReverse / balancer://tomcatcluster/
<Proxy balancer://tomcatcluster>
BalancerMember ajp://localhost:8209 route=tomcat1
BalancerMember ajp://localhost:8409 route=tomcat2
</Proxy>
```

注意我这里的localhost:8209,localhost:8409 是两个我本地两个tomcat的运行端口tomcat1,tomcat2是我本地tomcat的jvmRoute的配置下面我会说明,在修改配置的时候tomcat有4个端口需要修改分别是主server运行端口,http服务运行端口,ajp运行端口,集群接受session运行端口

下载安装各个tomcat,我是本机运行了多个tomcat也可以分多个机器每个运行一到多个tomcat.

如果你用虚拟host的时候也可做如下配置：

```
vi /data/httpd/apache/conf/httpd.conf
# 把"#include conf/extra/httpd-vhosts.conf" 前面的"#"去掉然后编辑
vi /usr/local/apache/conf/extra/httpd-vhosts.conf
<VirtualHost 192.168.1.8:8111>
ServerAdmin zhdf@msn.com
ServerName localhost:8111
ServerAlias www.rymall.cn
ErrorLog logs/rymall.cn-error_log
CustomLog logs/rymall.cn-access_log common
<Proxy balancer://tomcatcluster>
BalancerMember ajp://127.0.0.1:8209 route=tomcat1
BalancerMember ajp://127.0.0.1:8409 route=tomcat2
</Proxy>
<Location />
ProxyPass balancer://tomcatcluster/ lbmethod=byrequests
stickysession=JSESSIONID nofailover=Off timeout=5 maxattempts=3
ProxyPassReverse balancer://tomcatcluster/
</Location>
</VirtualHost>
```

#### 配置tomcat

**修改端口**

- 需要修改<Server port="8005" shutdown="SHUTDOWN"\>端口,如果一样则不能停止tomcat。

- tomcat的默认WEB服务端口是8080,默认的模式是单独服务,我的二个tomcat的WEB服务端口修改为8888/9999

  修改位置为tomcat的安装目录下的conf/server.xml

  修改前的配置为

  ```
  <Connector port="8080" maxHttpHeaderSize="8192"
  maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
  enableLookups="false" redirectPort="8443" acceptCount="100"
  connectionTimeout="20000" disableUploadTimeout="true" />
  ```

  修改后的配置为

  ```
  <Connector port="8888" maxHttpHeaderSize="8192"
  maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
  enableLookups="false" redirectPort="8443" acceptCount="100"
  connectionTimeout="20000" disableUploadTimeout="true" />
  ```

  依次修改每个tomcat的监听端口(8888/9999)

- 修改ajp的端口

  ```
  <Connector port="8209" enableLookups="false" redirectPort="8443"protocol="AJP/1.3" />
  ```


  另一个修改为8429。

**设置ajp**

分别修改tomcat的配置文件conf/server.xml,修改内容如下:

修改前

```
<!-- You should set jvmRoute to support load-balancing via AJP ie :
<Engine name="Standalone" defaultHost="localhost" jvmRoute="jvm1">
-->
<!-- Define the top level container in our container hierarchy -->
<Engine name="Catalina" defaultHost="localhost">
修改后
<!-- You should set jvmRoute to support load-balancing via AJP ie :-->
<Engine name="Standalone" defaultHost="localhost" jvmRoute="tomcat1">
区别-----------------注释去掉
<!-- Define the top level container in our container hierarchy
<Engine name="Catalina" defaultHost="localhost">
-->
区别------------------加上注释
```

将其中的jvmRoute="jvm1"分别修改为jvmRoute="tomcat1"和jvmRoute="tomcat2"

**设置集群**

在tomcat的配置文件/conf/server.xml中去掉Cluster节点的注释(这个Cluster是用来做session同步的保证多个tomcat能达到session同步),这里注意一下,如果是一台机器运行多个tomcat请更改下

```
<Receiver
className="org.apache.catalina.cluster.tcp.ReplicationListener"
tcpListenAddress="auto"
tcpListenPort="4004"
tcpSelectorTimeout="100"
tcpThreadCount="6"/>
```

的tcpListenPort端口保证tomcat接受数据运行在不同的端口.

下面的节点请不要修改:

```
<Membership
className="org.apache.catalina.cluster.mcast.McastService"
mcastAddr="228.0.0.4"
mcastPort="45564"
mcastFrequency="500"
mcastDropTime="3000"/>
```

如果你需要配置不同的集群,比如tomcat1,tomcat2运行为一个集群,tomcat3,tomcat4运行在第二个集群请修改mcastAddr和mcastPort只要这两个参数一样会被认为在同个集群下,session会被同步.

**设置session同步**

对于要进行负载和集群的的tomcat目录下的webapps中的应用中的WEB-INF中的web.xml文件要添加如下一句配置:
例如:Tomcat\webapps\handup\WEB-INF\web.xml

```
<distributable/>
```

配置前

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" version="2.4">
<display-name>TomcatDemo</display-name>
</web-app>
```

配置后

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/j2eehttp://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" version="2.4">
<display-name>TomcatDemo</display-name>
<distributable/>
</web-app>
```

为了在TOMCAT容器中SESSION复制可用,必须完成以下步骤:

- 为了集群能够工作,你必须使用你系统上的多点传送可使用
- 为了有些使用 SESSION 复制,所有 TOMCAT 例程必须同样配置。这意味着WEB 应用程序必须统一的部署在集群中的每台服务器上。这些配置同样简化了集群管理,维护和发现维修故障的任务。
- 在 server.xml 未注释的 Cluster 和 Valve (ReplicationValve) 元素。
  起用 server.xm中的 CLUSTER 元素意味着所有 WEB CONTEXT 的 SESSION 管理器将会改变。因此 当运行一个集群的时候,你应该确保只有一个需要被集成 WEB 应用程序并且移除其他的。
- 如果服务器例程运行在同样的机器上,应该确保每个例程的 tcpListenPort 属性是一致的。
- 确保 web.xml 有 distributable 属性
- 所有的 SESSION 属性必须实现 java.io.Serializable 接口

