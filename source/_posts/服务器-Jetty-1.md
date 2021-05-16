---
title: Jetty 轻量级的servlet服务器
date: 2020-04-03 11:18:59
tags:
 - 服务器
 - Jetty
categories:
 - 服务器
 - Jetty
---

Jetty 是一个开源的servlet容器，它为基于Java的web容器，例如JSP和servlet提供运行环境。Jetty是使用Java语言编写的，它的API以一组JAR包的形式发布。开发人员可以将Jetty容器实例化成一个对象，可以迅速为一些独立运行（stand-alone）的Java应用提供网络和web连接。

由于其轻量、灵活的特性，Jetty也被应用于一些知名产品中，例如ActiveMQ、Maven、Spark、GoogleAppEngine、Eclipse、Hadoop等。

Jetty较于Tomcat属于轻量级，而且在处理高并发细粒度请求的场景下显得更快速高效。所以使其也拥有众多使用场景，合适的选择应该为：云平台本身的门户网站放在Tomcat内，而云台托管的Java Web应该是部署在Jetty内的。

<!--more-->

官网:<https://www.eclipse.org/jetty/>

**为什么使用Jetty？**

1. 异步的 Servlet，支持更高的并发量  
2. 模块化的设计，更灵活，更容易定制，也意味着更高的资源利用率
3. 在面对大量长连接的业务场景下，Jetty 默认采用的 NIO 模型是更好的选择
4. 将jetty嵌入到应用中，使一个普通应用可以快速支持 http 服务

**Jetty的基本架构**

![GADiaF.png](https://s1.ax1x.com/2020/03/28/GADiaF.png)



#### jetty配置https的方式

Jetty 需要使用的Key文件为keystore，而各大服务商申请的Key文件一般为pem等文件

**首先生成数字证书，生成证书到 D:\localhost.keystore**
使用 JDK 的 keytool 命令，生成证书（包含证书 / 公钥 / 私钥）到 `D:\localhost.keystore`：

```
keytool -genkey -keystore "D:\localhost.keystore" -alias localhost -keyalg RSA
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  localhost
您的组织单位名称是什么?
  [Unknown]:  sishuok.com
您的组织名称是什么?
  [Unknown]:  sishuok.com
您所在的城市或区域名称是什么?
  [Unknown]:  beijing
您所在的省/市/自治区名称是什么?
  [Unknown]:  beijing
该单位的双字母国家/地区代码是什么?
  [Unknown]:  cn
CN=localhost, OU=sishuok.com, O=sishuok.com, L=beijing, ST=beijing, C=cn是否正确
?
  [否]:  y

输入 <localhost> 的密钥口令
        (如果和密钥库口令相同, 按回车):
再次输入新口令:
```

