---
title: 使用IDE开发工具远程调试Java代码
date: 2020-06-07 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

服务端程序运行在一台远程服务器上，我们可以在本地服务端的代码（前提是本地的代码必须和远程服务器运行的代码一致）中设置断点，每当有请求到远程服务器时时能够在本地知道远程服务端的此时的内部状态

<!--more-->

1. 服务端使用特定JVM参数运行代码

   要让远程服务器运行的代码支持远程调试，则启动的时候必须加上特定的JVM参数，这些参数是：

   ```
   1 -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
   ```

   其中5005是调试端口

2. 本地连接远程服务器debug端口

   ![USTBcD.png](https://s1.ax1x.com/2020/07/05/USTBcD.png)

   ![USTyBd.png](https://s1.ax1x.com/2020/07/05/USTyBd.png)

   配置完成，连接调试。

   在Servlet中，增加断点即可调试。

   

