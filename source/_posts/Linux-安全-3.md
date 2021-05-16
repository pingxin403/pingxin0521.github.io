---
title: iptables和Firewalld防火墙
date: 2019-03-25 13:18:59
tags:
 - Linux
 - 安全
categories:
 - Linux
 - 安全 
---

### 前言

所谓防火墙指的是一个由软件和硬件设备组合而成、在内部网和外部网之间、专用网与公共网之间的边界上构造的保护屏障。
防火墙是一种保护计算机网络安全的技术性措施，它通过在网络边界上建立相应的网络通信监控系统来隔离内部和外部网络，以阻挡来自外部的网络入侵。

防火墙有网络防火墙和计算机防火墙的提法。网络防火墙是指在外部网络和内部网络之间设置网络防火墙。这种防火墙又称筛选路由器。网络防火墙检测进入信息的协议、目的地址、端口（网络层）及被传输的信息形式（应用层）等，滤除不符合规定的外来信息。网络防火墙也对用户网络向外部网络发出的信息进行检测。

计算机防火墙是指在外部网络和用户计算机之间设置防火墙。计算机防火墙也可以是用户计算机的一部分。计算机防火墙检测接口规程、传输协议、目的地址及/或被传输的信息结构等，将不符合规定的进入信息剔除。计算机防火墙对用户计算机输出的信息进行检查，并加上相应协议层的标志，用以将信息传送到接收用户计算机（或网络）中去。

<!--more-->

在CentOS 7系统中，firewalld防火墙取代了iptables防火墙。对于接触Linux系统比较早或学习过CentOS 6系统的读者来说，当他们发现曾经掌握的知识在CentOS 7中不再适用，需要全新学习firewalld时，难免会有抵触心理。其实，iptables与firewalld都不是真正的防火墙，它们都只是用来定义防火墙策略的防火墙管理工具而已，或者说，它们只是一种服务。iptables服务会把配置好的防火墙策略交由内核层面的netfilter网络过滤器来处理，而firewalld服务则是把配置好的防火墙策略交由内核层面的nftables包过滤框架来处理。换句话说，当前在Linux系统中其实存在多个防火墙管理工具，旨在方便运维人员管理Linux系统中的防火墙策略，我们只需要配置妥当其中的一个就足够了。虽然这些工具各有优劣，但它们在防火墙策略的配置思路上是保持一致的。大家只要在这多个防火墙管理工具中任选一款并将其学透，就足以满足日常的工作需求了。

### 原理

#### 应用技术

**数据包过滤**
网络上的数据都是以包为单位进行传输的，每一个数据包中都会包含一些特定的信息，如数据的源地址、目标地址、源端口号和目标端口号等。防火墙通过读取数据包中的地址信息来判断这些包是否来自可信任的网络，并与预先设定的访问控制规则进行比较，进而确定是否需对数据包进行处理和操作。数据包过滤可以防止外部不合法用户对内部网络的访问，但由于不能检测数据包的具体内容，所以不能识别具有非法内容的数据包，无法实施对应用层协议的安全处理。 

**网络IP地址转换**
网络IP地址转换是一种将私有IP地址转化为公网IP地址的技术，它被广泛应用于各种类型的网络和互联网中。网络IP地址转换一方面可隐藏内部网络的真实IP地址，使内部网络免受黑客的直接攻击，另一方面由于内部网络使用了私有IP地址，从而有效解决了公网IP 地址不足的问题。

**虚拟专用网络**
虚拟专用网络将分布在不同地域上的局域网或计算机通过加密通信，虚拟出专用的传输通道，从而将它们从逻辑上连成一个整体，不仅省去了建设专用通信线路的费用，还有效地保证了网络通信的安全。

**应用网关**
应用级网关能够检查进出的数据包，通过网关复制传递数据，防止在受信任服务器和客户机与不受信任的主机间直接建立联系。应用级网关能够理解应用层上的协议，能够做复杂一些的访问控制，并做精细的注册和稽核。它针对特别的网络应用服务协议即数据过滤协议，并且能够对数据包分析并形成相关的报告。

#### 功能

防火墙对流经它的网络通信进行扫描，这样能够过滤掉一些攻击，以免其在目标计算机上被执行。防火墙还可以关闭不使用的端口。而且它还能禁止特定端口的流出通信，封锁特洛伊木马。最后，它可以禁止来自特殊站点的访问，从而防止来自不明入侵者的所有通信。

**网络安全的屏障**
一个防火墙（作为阻塞点、控制点）能极大地提高一个内部网络的安全性，并通过过滤不安全的服务而降低风险。由于只有经过精心选择的应用协议才能通过防火墙，所以网络环境变得更安全。如防火墙可以禁止诸如众所周知的不安全的NFS协议进出受保护网络，这样外部的攻击者就不可能利用这些脆弱的协议来攻击内部网络。防火墙同时可以保护网络免受基于路由的攻击，如IP选项中的源路由攻击和ICMP重定向中的重定向路径。防火墙应该可以拒绝所有以上类型攻击的报文并通知防火墙管理员。

**强化网络安全策略**
通过以防火墙为中心的安全方案配置，能将所有安全软件（如口令、加密、身份认证、审计等）配置在防火墙上。与将网络安全问题分散到各个主机上相比，防火墙的集中安全管理更经济。例如在网络访问时，一次一密口令系统和其它的身份认证系统完全可以不必分散在各个主机上，而集中在防火墙一身上。

**监控审计**
如果所有的访问都经过防火墙，那么，防火墙就能记录下这些访问并作出日志记录，同时也能提供网络使用情况的统计数据。当发生可疑动作时，防火墙能进行适当的报警，并提供网络是否受到监测和攻击的详细信息。另外，收集一个网络的使用和误用情况也是非常重要的。首先的理由是可以清楚防火墙是否能够抵挡攻击者的探测和攻击，并且清楚防火墙的控制是否充足。而网络使用统计对网络需求分析和威胁分析等而言也是非常重要的。

**防止内部信息的外泄**
通过利用防火墙对内部网络的划分，可实现内部网重点网段的隔离，从而限制了局部重点或敏感网络安全问题对全局网络造成的影响。再者，隐私是内部网络非常关心的问题，一个内部网络中不引人注意的细节可能包含了有关安全的线索而引起外部攻击者的兴趣，甚至因此而暴漏了内部网络的某些安全漏洞。使用防火墙就可以隐蔽那些透漏内部细节如Finger，DNS等服务。Finger显示了主机的所有用户的注册名、真名，最后登录时间和使用shell类型等。但是Finger显示的信息非常容易被攻击者所获悉。攻击者可以知道一个系统使用的频繁程度，这个系统是否有用户正在连线上网，这个系统是否在被攻击时引起注意等等。防火墙可以同样阻塞有关内部网络中的DNS信息，这样一台主机的域名和IP地址就不会被外界所了解。除了安全作用，防火墙还支持具有Internet服务性的企业内部网络技术体系VPN（虚拟专用网）。

**日志记录与事件通知**
进出网络的数据都必须经过防火墙，防火墙通过日志对其进行记录，能提供网络使用的详细统计信息。当发生可疑事件时，防火墙更能根据机制进行报警和通知，提供网络是否受到威胁的信息。



### Iptables

在早期的Linux系统中，默认使用的是iptables防火墙管理服务来配置防火墙。尽管新型的firewalld防火墙管理服务已经被投入使用多年，但是大量的企业在生产环境中依然出于各种原因而继续使用iptables。

#### 策略与规则链

防火墙会从上至下的顺序来读取配置的策略规则，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。一般而言，防火墙策略规则的设置有两种：一种是“通”（即放行），一种是“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

iptables服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

> 在进行路由选择前处理数据包（PREROUTING）；
>
> 处理流入的数据包（INPUT）；
>
> 处理流出的数据包（OUTPUT）；
>
> 处理转发的数据包（FORWARD）；
>
> 在进行路由选择后处理数据包（POSTROUTING）。

一般来说，从内网向外网发送的流量一般都是可控且良性的，因此我们使用最多的就是INPUT规则链，该规则链可以增大黑客人员从外网入侵内网的难度。

但是，仅有策略规则还不能保证社区的安全，保安还应该知道采用什么样的动作来处理这些匹配的流量，比如“允许”、“拒绝”、“登记”、“不理它”。这些动作对应到iptables服务的术语中分别是ACCEPT（允许流量通过）、REJECT（拒绝流量通过）、LOG（记录日志信息）、DROP（拒绝流量通过）。“允许流量通过”和“记录日志信息”都比较好理解，这里需要着重讲解的是REJECT和DROP的不同点。就DROP来说，它是直接将流量丢弃而且不响应；REJECT则会在拒绝流量后再回复一条“您的信息已经收到，但是被扔掉了”信息，从而让流量发送方清晰地看到数据被拒绝的响应信息。

当把Linux系统中的防火墙策略设置为REJECT拒绝动作后，流量发送方会看到端口不可达的响应。

而把Linux系统中的防火墙策略修改成DROP拒绝动作后，流量发送方会看到响应超时的提醒。但是流量发送方无法判断流量是被拒绝，还是接收方主机当前不在线。



#### 基本命令

iptables是一款基于命令行的防火墙策略管理工具，具有大量参数，学习难度较大。好在对于日常的防火墙策略配置来讲，大家无需深入了解诸如“四表五链”的理论概念，只需要掌握常用的参数并做到灵活搭配即可，这就足以应对日常工作了。

iptables命令可以根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配，一旦匹配成功，iptables就会根据策略规则所预设的动作来处理这些流量。另外，再次提醒一下，防火墙策略规则的匹配顺序是从上至下的，因此要把较为严格、优先级较高的策略规则放到前面，以免发生错误。

语法：`Iptables [-ttable] COMMAND chain CRETIRIA -j ACTION `

**相关参数说明：** 　　 

```
　-t table：3个filter nat mangle 
　　：定义如何对规则进行管理 　　 
　　chain：指定你接下来的规则到底是在哪个链上操作的，当定义策略的时候，是可以省略的； 
　　CRETIRIA:指定匹配标准； 　 
　　-j ACTION:指定如何进行处理；
```

**常用命令**

- iptables -A 将一个规则添加到链末尾
- iptables -D 将指定的链中删除规则
- iptables -F 将指定的链中删除所有规则
- iptables -I 将在指定链的指定编号位置插入一个规则
- iptables -L 列出指定链中所有规则
- iptables -t nat -L 列出所有NAT链中所有规则
- iptables -N 建立用户定义链
- iptables -X 删除用户定义链
- iptables -P 修改链的默认设置，如将iptables -P INPUT DROP (将INPUT链设置为DROP)

常见参数：

- --dport 指定目标TCP/IP端口 如 –dport 80
- --sport 指定源TCP/IP端口 如 –sport 80
- -p tcp 指定协议为tcp
- -p icmp 指定协议为ICMP
- -p udp 指定协议为UDP
- -j DROP 拒绝
- -j ACCEPT 允许
- -j REJECT 拒绝并向发出消息的计算机发一个消息
- -j LOG 在/var/log/messages中登记分组匹配的记录
- -m mac –mac 绑定MAC地址
- -m limit –limit 1/s 1/m 设置时间策列
- -m multiport      多端口匹配,和dport或sport结合，使用 `，`区分端口号，可用于匹配非连续或连续端口；最多指定15个端口
- -s 10.10.0.0或10.10.0.0/16 指定源地址或地址段
- -d 10.10.0.0或10.10.0.0/16 指定目标地址或地址段
- -s ! 10.10.0.0 指定源地址以外的

**配置文件**

配置文件位置： /etc/sysconfig/iptables

#### 常用用法

#####  配置Filter表防火墙

1. 查看iptables的配置信息

   ```
   # iptables -L -n
   ```

2. 清除原有防火墙规则

   - 清除预设表filter中的所有规则链的规则

   ```
   # iptables -F
   ```

   - 清除预设表filter中使用者自定链中的规则

   ```
   # iptables -X
   ```

   - 保存防火墙设置

   ```
   # /etc/init.d/iptables save
   或
   # service iptables save
   ```

3. 设定预设规则

   ```
   -- 请求接入包丢弃
   [root@home ~]# iptables -P INPUT DROP
   -- 接受响应数据包
   [root@home ~]# iptables -P OUTPUT ACCEPT
   -- 转发数据包丢弃 
   [root@home ~]# iptables -P FORWARD DROP
   ```

4. 添加防火墙规则

   首先添加INPUT链,INPUT链的默认规则是DROP,所以我们就写需要ACCETP(通过)的链。

   - **开启SSH服务端口**

   ```
   [root@tp ~]# iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   [root@tp ~]# iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
   ```

   > 注:如果在预设设置把OUTPUT设置成DROP策略的话，就需要设置OUTPUT规则，否则无法进行SSH连接。

   - **开启Web服务端口**

   ```
   [root@tp ~]# iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
   [root@tp ~]# iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   ```

   - **开启邮件服务的25、110端口**

   ```
   [root@tp ~]# iptables -A INPUT -p tcp --dport 110 -j ACCEPT
   [root@tp ~]# iptables -A INPUT -p tcp --dport 25 -j ACCEPT
   ```

   - **开启FTP服务的21端口**

   ```
   [root@tp ~]# iptables -A INPUT -p tcp --dport 21 -j ACCEPT
   [root@tp ~]# iptables -A INPUT -p tcp --dport 20 -j ACCEPT
   ```

   - **开启DNS服务的53端口**

   ```
   [root@tp ~]# iptables -A INPUT -p tcp --dport 53 -j ACCEPT
   ```

   - **设置icmp服务**

   ```
   [root@tp ~]# iptables -A OUTPUT -p icmp -j ACCEPT (OUTPUT设置成DROP的话)
   [root@tp ~]# iptables -A INPUT -p icmp -j ACCEPT    (INPUT设置成DROP的话)
   ```

   - **允许loopback**

   > 不然会导致DNS无法正常关闭等问题

   ```
   [root@tp ~]# IPTABLES -A INPUT -i lo -p all -j ACCEPT 
   (如果是INPUT DROP)
   [root@tp ~]# IPTABLES -A OUTPUT -o lo -p all -j ACCEPT
   (如果是OUTPUT DROP)
   ```

   - **减少不安全的端口连接**

   ```
   [root@tp ~]# iptables -A OUTPUT -p tcp --sport 31337 -j DROP
   [root@tp ~]# iptables -A OUTPUT -p tcp --dport 31337 -j DROP
   ```

   > 说明：有些特洛伊木马会扫描端口31337到31340(即黑客语言中的 elite 端口)上的服务。既然合法服务都不使用这些非标准端口来通信,阻塞这些端口能够有效地减少你的网络上可能被感染的机器和它们的远程主服务器进行独立通信的机会。此外，其他端口也一样,像:31335、27444、27665、20034 NetBus、9704、137-139（smb）,2049(NFS)端口也应被禁止。

   - **只允许某台主机或某个网段进行SSH连接**

   只允许192.168.0.3的机器进行SSH连接

   ```
   [root@tp ~]# iptables -A INPUT -s 192.168.0.3 -p tcp --dport 22 -j ACCEPT
   ```

   如果允许或限制一段IP地址可用`192.168.0.0/24`表示192.168.0.1-255端的所有IP, 24表示子网掩码数。

   ```
   [root@tp ~]# iptables -A INPUT -s 192.168.0.0/24 -p tcp --dport 22 -j ACCEPT
   ```

   > 注意：指定某个主机或者某个网段进行SSH连接，需要在iptables配置文件中的`-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT`
   > 删除，因为它表示所有地址都可以登陆.

   如果只允许除了192.168.0.3的主机外都能进行SSH连接

   ```
   [root@tp ~]# iptables -A INPUT -s ! 192.168.0.3 -p tcp --dport 22 -j ACCEPT
   ```

   - **开启转发功能**

   在做NAT网络配置时,FORWARD默认规则是DROP时,必须开启数据包转发功能

   ```
   [root@tp ~]# iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
   [root@tp ~]# iptables -A FORWARD -i eth1 -o eh0 -j ACCEPT
   ```

   - **丢弃坏的TCP包**

   ```
   [root@tp ~]#iptables -A FORWARD -p TCP ! --syn -m state --state NEW -j DROP
   ```

   - **处理IP碎片数量，防止DDOS攻击，允许每秒100个**

   ```
   [root@tp ~]#iptables -A FORWARD -f -m limit --limit 100/s --limit-burst 100 -j ACCEPT
   ```

   - **设置ICMP包过滤, 允许每秒1个包, 限制触发条件是10个包**

   ```
   [root@tp ~]#iptables -A FORWARD -p icmp -m limit --limit 1/s --limit-burst 10 -j ACCEPT
   ```

   - **DROP非法连接**

   ```
   [root@tp ~]# iptables -A INPUT   -m state --state INVALID -j DROP
   [root@tp ~]# iptables -A OUTPUT  -m state --state INVALID -j DROP
   [root@tp ~]# iptables -A FORWARD -m state --state INVALID -j DROP
   ```

   - **允许所有已经建立的和相关的连接**

   ```
   [root@tp ~]# iptables-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   [root@tp ~]# iptables-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   [root@tp ~]# /etc/rc.d/init.d/iptables save
   ```

#####  配置NAT表防火墙

1. 查看本机关于NAT的设置情况

   ```
   [root@tp rc.d]# iptables -t nat -L
   ```

2. 清除NAT规则

   ```
   [root@tp ~]# iptables -F -t nat
   [root@tp ~]# iptables -X -t nat
   [root@tp ~]# iptables -Z -t nat
   ```

3. 添加规则

   添加基本的NAT地址转换，添加规则时，我们只添加DROP链，因为默认链全是ACCEPT。

   - **防止外网用内网IP欺骗**

   ```
   [root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 10.0.0.0/8 -j DROP
   [root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 172.16.0.0/12 -j DROP
   [root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 192.168.0.0/16 -j DROP
   ```

   - **禁止与211.101.46.253的所有连接**

   ```
   [root@tp ~]# iptables -t nat -A PREROUTING -d 211.101.46.253 -j DROP
   ```

   - **禁用FTP(21)端口**

   ```
   [root@tp ~]# iptables -t nat -A PREROUTING -p tcp --dport 21 -j DROP
   ```

   只禁用211.101.46.253地址的FTP连接,其他连接可以进行。

   ```shell
   [root@tp ~]# iptables -t nat -A PREROUTING -p tcp --dport 21 -d 211.101.46.253 -j DROP
   ```

   站在服务器的角度：当内网服务器要访问外网时，需要做源地址转换；

   ```shell
   # 打开转发功能
   sysctl -w net.ipv4.ip_forward=1
   
   # 把 192.168.1.0 网段流出的数据的 source ip address 修改成为 10.0.0.11：
   iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 10.0.0.11
   
   　　当外网主机要访问内网服务器服务时，需要做目标地址转换；
   
   # 打开转发功能
   sysctl -w net.ipv4.ip_forward=1
   
   # 把访问 10.0.0.11:2222 的访问转发到 192.168.1.11:22 上：：
   iptables -t nat -A PREROUTING -d 10.0.0.11 -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.11:22
   
   　　单纯的将访问本机80端口的请求转发到8080上
   
   iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
   ```

   

##### 常用显示模块

 开放ssh服务端口

```
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --sport 22 -j ACCEPT
```

修改默认规则为DROP

```
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

1. multiport: 多端口匹配

    可用于匹配非连续或连续端口；最多指定15个端口；
    实例

```
iptables -A INPUT -p tcp -m multiport --dport 22,80 -j ACCEPT
iptables -A OUTPUT -p tcp -m multiport --sport 22,80 -j ACCEPT
```

2. iprange: 匹配指定范围内的地址

    匹配一段连续的地址而非整个网络时有用
    实例：

```
iptables -A INPUT -p tcp -m iprange --src-range 192.168.118.0-192.168.118.60 --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp -m iprange --dst-range 192.168.118.0-192.168.118.60 --sport 22 -j ACCEPT
```

3. string: 字符串匹配，能够检测报文应用层中的字符串

    字符匹配检查高效算法：kmp, bm  
    能够屏蔽非法字符
    实例：

```
#注意该条规则需要添加到OUTPUT链，当服务端返回数据报文检查到有关键字"sex"时，则丢弃该报文，可用于web敏感词过滤
iptables -A OUTPUT -p tcp --dport 80 -m string --algo kmp --string "sex" -j DROP
```

4. connlimit: 连接数限制，对每IP所能够发起并发连接数做限制；

    实例：

```
#默认INPUT 为 DROP. 每个ip对ssh服务的访问最大为3个并发连接，超过则丢弃

iptables -A INPUT -p tcp  --dport 22 -m connlimit ! --connlimit-above 3 -j ACCEPT
```

5. limit: 速率限制
    limit-burst: 设置默认阀值

```
#默认放行10个，当到达limit-burst阀值后，平均6秒放行1个

iptables -A INPUT -p icmp -m limit --limit 10/minute --limit-burst 10 -j ACCEPT
```

6. state: 状态检查

    连接追踪中的状态：
        NEW: 新建立一个会话
        ESTABLISHED：已建立的连接
        RELATED: 有关联关系的连接
        INVALID: 无法识别的连接

```
#放行ssh的首次连接状态
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT  
```

##### 针对特定的服务定制相关规则

1. 对ssh进行管控，1小时内最多发起5个连接，防止黑客暴力破解ssh

```shell
# 清空默认规则
iptables -F
iptables -X

# 添加已建立的连接规则
iptables -A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

# 设置默认规则
iptables -P INPUT DROP
iptables -P OUTPUT DROP

# 设置iptables记录匹配ssh规则
iptables -I INPUT  2 -p tcp --syn -m state --state NEW -j LOG --log-level 5 --log-prefix "[SSH Login]:"

# 使用recent显示模块限定每小时最多匹配到5次，超过则丢弃。
iptables -A INPUT -p tcp --syn -m state --state NEW -m recent --name OPENSSH --update --seconds 3600 --hitcount 5 -j DROP
iptables -A INPUT -p tcp --syn -m state --state NEW -m recent --name OPENSSH --set -j ACCEPT
```



2. 对web服务进行并发管控,防止Ddos

```shell
# 清空默认规则
iptables -F
iptables -X

# 添加已建立的连接规则
iptables -A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

# 设置默认规则
iptables -P INPUT DROP
iptables -P OUTPUT DROP

# 每个ip的并发连接请求最大50，超过则丢弃，建议调大值，容易误伤nat上网用户
iptables -A INPUT -p tcp --syn --dport 80 -m state --state NEW -m connlimit ! --connlimit-above 50 -j DROP
```



3. 对icmp进行流控，防止icmp攻击

```shell
# 清空默认规则
iptables -F
iptables -X

# 添加已建立的连接规则
iptables -A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

# 设置默认规则
iptables -P INPUT DROP
iptables -P OUTPUT DROP

# 利用limit模块限制icmp速率，阀值为10个，当到达10个后，限速每秒钟1个
iptables -A INPUT -p icmp --icmp-type 0 -m limit --limit 1/s --limit-burst 10 -j ACCEPT
```

##### iptables 独立日志配置

当在iptables中启用日志记录时，会被当做系统日志记录到/var/log/message里面，如果想要独立日志配置如下：

```shell
] # grep -r 'iptables' /etc/rsyslog.conf  #在该文件添加下面三行
kern.*     /var/log/iptables.log

] # systemctl restart rsyslog

#在iptables添加日志选项 
] #iptables -A INPUT  -j LOG --log-prefix "iptables"
#这样就可以记录所有的记录了，只要通过了防火墙都会记录到日志里
] #iptables -A INPUT  -p tcp -j LOG --log-prefix "iptables icmp warn"
#这样就只记录tcp日志
#然后保存并重启防火墙配置
] # iptables-save
] # iptables-restart

```

### Filewalld

CentOS 7系统中集成了多款防火墙管理工具，其中firewalld（Dynamic Firewall Manager of Linux systems，Linux系统的动态防火墙管理器）服务是默认的防火墙配置管理工具，它拥有基于CLI（命令行界面）和基于GUI（图形用户界面）的两种管理方式。安装它，只需`yum install firewalld`；如果需要图形界面的话，则再安装`yum install firewall-config`。

![firewall.png](https://i.loli.net/2019/03/25/5c98cc2398e31.png)

相较于传统的防火墙管理配置工具，firewalld支持动态更新技术并加入了区域（zone）的概念。简单来说，区域就是firewalld预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。例如，我们有一台笔记本电脑，每天都要在办公室、咖啡厅和家里使用。按常理来讲，这三者的安全性按照由高到低的顺序来排列，应该是家庭、公司办公室、咖啡厅。当前，我们希望为这台笔记本电脑指定如下防火墙策略规则：在家中允许访问所有服务；在办公室内仅允许访问文件共享服务；在咖啡厅仅允许上网浏览。在以往，我们需要频繁地手动设置防火墙策略规则，而现在只需要预设好区域集合，然后只需轻点鼠标就可以自动切换了，从而极大地提升了防火墙策略的应用效率。firewalld中常见的区域名称（默认为public）以及相应的策略规则如表。

| 区域     | 默认规则策略                                                 |
| -------- | ------------------------------------------------------------ |
| trusted  | 允许所有的数据包                                             |
| home     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量 |
| internal | 等同于home区域                                               |
| work     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量 |
| public   | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量 |
| external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| dmz      | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| block    | 拒绝流入的流量，除非与流出的流量相关                         |
| drop     | 拒绝流入的流量，除非与流出的流量相关                         |

过滤规则

- source: 根据源地址过滤
- interface: 根据网卡过滤
- service: 根据服务名过滤
- port: 根据端口过滤
- icmp-block: icmp 报文过滤，按照 icmp 类型配置
- masquerade: ip 地址伪装
- forward-port: 端口转发
- rule: 自定义规则

其中，过滤规则的优先级遵循如下顺序

1.source
2.interface
3.firewalld.conf

#### 配置文件

**系统配置目录**

​    /usr/lib/firewalld/services，目录中存放定义好的网络服务和端口参数，系统参数，不能修改。

**用户配置目录**

​        /etc/firewalld/

#### 命令使用

具体的规则管理，可以使用firewall-cmd ，具体的使用方法可以

```shell
$ firewall-cmd --help
--zone=NAME                        # 指定 zone
--permanent                  # 永久修改，--reload 后生效
--timeout=seconds  # 持续效果，到期后自动移除，用于调试
					#不能与 --permanent 同时使用
```

**查看规则**

```
查看运行状态

$ firewall-cmd --state

查看已被激活的 Zone 信息

$ firewall-cmd --get-active-zones
public
  interfaces: eth0 eth1

查看指定接口的 Zone 信息

$ firewall-cmd --get-zone-of-interface=eth0
public

查看指定级别的接口

$ firewall-cmd --zone=public --list-interfaces
eth0

查看指定级别的所有信息，譬如 public

$ firewall-cmd --zone=public --list-all
public (default, active)
  interfaces: eth0
  sources:
  services: dhcpv6-client http ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

查看所有级别被允许的信息

$ firewall-cmd --get-service

查看重启后所有 Zones 级别中被允许的服务，即永久放行的服务

$ firewall-cmd --get-service --permanent
```

##### 管理规则

```bash
# firewall-cmd --panic-on          # 丢弃

# firewall-cmd --panic-off          # 取消丢弃

# firewall-cmd --query-panic        # 查看丢弃状态

# firewall-cmd --reload            # 更新规则，不重启服务

# firewall-cmd --complete-reload    # 更新规则，重启服务

添加某接口至某信任等级，譬如添加 eth0 至 public，永久修改

# firewall-cmd --zone=public --add-interface=eth0 --permanent

设置 public 为默认的信任级别

# firewall-cmd --set-default-zone=public

a. 管理端口

列出 dmz 级别的被允许的进入端口

# firewall-cmd --zone=dmz --list-ports

允许 tcp 端口 8080 至 dmz 级别

# firewall-cmd --zone=dmz --add-port=8080/tcp

允许某范围的 udp 端口至 public 级别，并永久生效

# firewall-cmd --zone=public --add-port=5060-5059/udp --permanent

b. 网卡接口

列出 public zone 所有网卡

# firewall-cmd --zone=public --list-interfaces

将 eth0 添加至 public zone，永久

# firewall-cmd --zone=public --permanent --add-interface=eth0

eth0 存在与 public zone，将该网卡添加至 work zone，并将之从 public zone 中删除

# firewall-cmd --zone=work --permanent --change-interface=eth0

删除 public zone 中的 eth0，永久

# firewall-cmd --zone=public --permanent --remove-interface=eth0

c. 管理服务

添加 smtp 服务至 work zone

# firewall-cmd --zone=work --add-service=smtp

移除 work zone 中的 smtp 服务

# firewall-cmd --zone=work --remove-service=smtp

d. 配置 external zone 中的 ip 地址伪装

查看

# firewall-cmd --zone=external --query-masquerade

打开伪装

# firewall-cmd --zone=external --add-masquerade

关闭伪装

# firewall-cmd --zone=external --remove-masquerade

e. 配置 public zone 的端口转发

要打开端口转发，则需要先

# firewall-cmd --zone=public --add-masquerade

然后转发 tcp 22 端口至 3753

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=3753

转发 22 端口数据至另一个 ip 的相同端口上

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toaddr=192.168.1.100

转发 22 端口数据至另一 ip 的 2055 端口上

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=2055:toaddr=192.168.1.100

f. 配置 public zone 的 icmp

查看所有支持的 icmp 类型

# firewall-cmd --get-icmptypes

destination-unreachable echo-reply echo-request parameter-problem redirect router-advertisement router-solicitation source-quench time-exceeded

列出

# firewall-cmd --zone=public --list-icmp-blocks

添加 echo-request 屏蔽

# firewall-cmd --zone=public --add-icmp-block=echo-request [--timeout=seconds]

移除 echo-reply 屏蔽

# firewall-cmd --zone=public --remove-icmp-block=echo-reply

g. IP 封禁

# firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='222.222.222.222' reject"
```



### 参考：

1. [ Iptables与Firewalld防火墙](https://www.linuxprobe.com/chapter-08.html)

2. [iptables 用法及常用模块总结](https://www.linuxidc.com/Linux/2017-10/147347.htm)

