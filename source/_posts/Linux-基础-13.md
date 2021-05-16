---
title: Linux-网络管理
date: 2019-03-23 14:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

现在说的网络通信其实就是进程之间的通信，同一主机使用IPC机制，不同主机之间使用socket机制。在五层协议中，数据链路层使用MAC地址确定本地局域网内通信，每个网卡（网络适配器）都有一个MAC地址，如果一台电脑既有有线网卡，又有无线网卡，那么就有多个MAC地址；网络层使用IP地址界定通信主机，源和目标，范围在互联网之内；传输层涉及到端口（port），每个进程对应一个端口号，通信范围从进程到进程。

数据链路层使用交换机，只涉及MAC地址，用户一般很少涉及，因为网卡上的MAC地址是固定的；网络层使用路由器，实现跨网络通信，这是接入互联网必不可少的，为了接入，需要设置必要参数：IP/NETMASK、默认路由、DNS服务器；传输层界定进程，在主机范围内进行。

<!--more-->

所以我们所说的网络管理包括网络配置、路由配置、DNS配置等等。

#### 配置方式：

##### 静态指定

**命令：**

```
ifcfg家族：
		ifconfig：配置IP，NETMASK
		route：路由
		netstat：状态及统计数据查看
	iproute2家族：
		ip OBJECT：
			addr：地址和掩码；
			link：接口
			route：路由
		ss：状态及统计数据查看
	CentOS 7：nm(Network Manager)家族
		nmcli：命令行工具
		nmtui：text window 工具	
```

> 注意：
> 	(1) DNS服务器指定	
> 		配置文件：/etc/resolv.conf
> 	(2) 本地主机名配置
> 		命令：hostname
> 		配置文件：/etc/sysconfig/network
> 		CentOS 7：hostnamectl					

**配置文件：**
	RedHat及相关发行版

```
/etc/sysconfig/network-scripts/ifcfg-NETCARD_NAME
```

##### 动态分配

依赖于本地网络中有DHCP服务,DHCP：Dynamic Host Configure Procotol。



### 设备名规则

一个网卡安装到计算机上，linux如何识别呢？

CentOS 7上有一个设备管理器udev，它主要的功能是管理/dev目录底下的设备节点。它同时也是用来接替devfs及热插拔的功能，这意味着它要在添加/删除硬件时处理/dev目录以及所有用户空间的行为，包括加载固件时Linux内核。udev的最新版本依赖于升级后的的uevent接口的最新版本。

其中网卡命名规则文件位于 `/etc/udev/rules.d/70-persistent-net.rules`。

```
 ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{type}=="32", ATTR{address}=="?*00:02:c9:03:00:31:78:f2", NAME="mlx4_ib3"
```

**修改网卡命名示例**

1、查看网卡的驱动并且卸载网卡驱动

```shell
[root@rhel6 ~]# ethtool -i eth0
driver: e1000 #网卡驱动
[root@rhel6 ~]# modprobe -r e1000 #卸载网卡驱动
```

2、修改70-persistent-net.rules文件

3、重新加载网卡驱动并且重启网络服务

```shell
[root@rhel6 ~]# modprobe e1000   #重新加载网卡驱动
[root@rhel6 ~]# /etc/rc.d/init.d/network restart #重启网络服务
```

**传统命名**

以太网（总线型）：ethX, [0,oo)，例如eth0, eth1, ...
PPP网络（点对点）：pppX, [0,...], 例如，ppp0, ppp1, ...

**可预测命名方案（CentOS）**
支持多种不同的命名机制：Firmware, 拓扑结构

(1) 如果Firmware或BIOS为主板上集成的设备提供的索引信息可用，则根据此索引进行命名，如eno1, eno2, ...
(2) 如果Firmware或BIOS为PCI-E扩展槽所提供的索引信息可用，且可预测，则根据此索引进行命名，如ens1, ens2, ...
(3) 如果硬件接口的物理位置信息可用，则根据此信息命名，如enp2s0, ...
(4) 如果用户显式定义，也可根据MAC地址命名，例如enx122161ab2e10, ...
上述均不可用，则仍使用传统方式命名；

命名格式的组成

```
en：ethernet
wl：wlan
ww：wwan
```

名称类型

```
o<index>：集成设备的设备索引号；
s<slot>：扩展槽的索引号；
x<MAC>：基于MAC地址的命名；
p<bus>s<slot>：基于总线及槽的拓扑结构进行命名；
```

### 网络配置相关文件

网络配置参考文件：/usr/share/doc/initscripts-<版本号>/sysconfig.txt

网卡的配置在：/etc/sysconfig/network-scripts/下，配置文件：ifcfg-网卡名

配置文件示例：

```
[root@rhel6 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
BOOTPROTO=static|dhcp|none
IPADDR=192.168.0.6
NETMASK=255.255.255.0
#PREFIX=24 #子网掩码
GATEWAY=192.168.0.1
DNS1=114.114.114.114
DNS2=8.8.8.8
DNS3=1.1.1.1
TYPE=Ethernet
ONBOOT=yes
HWADDR=00:0C:29:DB:C9:5C
#MACADDR=00:0C:29:DB:C9:5A		#修改MAC地址
UUID=38d329c5-b1bb-491b-a669-47422cfda764
NM_CONTROLLED=no
```

**网络配置文件常用配置参数详解：**

- DEVICE：此配置文件应用到的设备
- HWADDR：对应的设备的MAC地址
- BOOTPROTO：激活此设备时使用的地址配置协议，常用的dhcp, static, none, bootp；使用dhcp或bootp时，会使用DHCP客户机在本机运行。
- NM_CONTROLLED：NM是NetworkManager的简写，此网卡是否接受NM控制；建议为“no”(NetworkManager:图形界面的网络配置工具，不支持桥接，强烈建议关闭)
- ONBOOT：在系统引导时是否激活此设备
- TYPE：接口类型，常见有的Ethernet, Bridge
- UUID：设备的惟一标识
- IPADDR：指明IP地址
- NETMASK：子网掩码
- GATEWAY: 默认网关
- DNS1：第一个DNS服务器指向
- DNS2：第二个DNS服务器指向
- USERCTL：普通用户是否可控制此设备
- PEERDNS：如果BOOTPROTO的值为“dhcp”，是否允许dhcp server分配的dns服务器指向信息直接覆盖至/etc/resolv.conf文件中

**其他相关配置文件**

路由配置文：/etc/sysconfig/network-scripts/route-interface

- NETWOEK/NETMASK via GATEWAY

DNS配置文件：/etc/resolv.conf

- nameserver DNS_IP

本地网络解析配置文件：/etc/hosts

- IP　　hostname alias

主机名配置文件：

- centos6.x：/etc/sysconfig/network
- centos7.x：/etc/hostname

### ifcfg命令家族

使用`yum install net-tools`安装该命令族。

#### ifconfig 

配置IP，NETMASK；网络接口及地址查看和管理

语法：

- -a：查看启用和被禁用的网卡信息
- interface {up|down}：启用或禁用网卡
- interface IP/NETMASK：临时设置IP
- interface [-]promisc：设置网卡的工作在混杂模式
- -s interface：查看指定网卡的流量信息

示例：

```shell
# 显示所有接口:
ifconfig -a
# 显示指定网卡信息： 
ifconfig    ens33                
# 启动关闭指定网卡:
ifconfig ens33 up/down    
# 为网卡配置和删除IPv6地址:   
ifconfig ens33  add/del   33ffe:3240:800:1005::2/64         
# 用ifconfig修改MAC地址： 
ifconfig ens33 hw ether 00:AA:BB:CC:dd:EE        
# 为指定网卡配置IP地址：
ifconfig ens33 192.168.2.10 netmask 255.255.255.0 broadcast 192.168.2.255
# 设置最大传输单元：
ifconfig eth0 mtu 1500    #设置能通过的最大数据包大小为 1500 bytes
```

#### route 

显示并设置Linux内核中的网络路由表

语法：

- -n：以数字方式显示，不解析，提高效率
- add {-net | -host} NETWORK/NETMASK gw GATEWAY dev DEVICE 添加路由
- {add | del} default gw GATEWAY 添加或删除默认路由
- del {-net | -host} NETWORK/NETMASK gw GATEWAY 删除路由

示例：

```shell
# 显示当前路由：
route -n
# 添加网关：
route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0 
# 屏蔽一条路由：
route add -net 224.0.0.0 netmask 240.0.0.0 reject 
# 删除路由记录：
route del -net 224.0.0.0 netmask 240.0.0.0
# 添加设置默认网关：
route add default gw 192.168.120.240
# 删除默认网关：
route del default gw 192.168.120.240
```



#### netstat

Linux中网络系统的状态信息

语法：

- -n：以数字方式显示，不解析，提高效率
- -r：查看路由表
- -t：TCP相关
- -u：UDP相关
- -w：裸套接字
- -l：查看处于监听状态的端口
- -a：查看所有状态的端口
- -e：显示更详细的信息
- -p：查看相关的进程PID
- -i：显示网卡流量
- -I  <interface>：查看指定网卡的流量信息 == ifconfig -s interface

常用组合：-tan,  -uan,  -tnl,  -unl,  -tunlp



注意：通过配置文件/etc/sysconfig/network-scripts/ifcfg-IFACE来识别接口并完成配置；

#### 配置主机名

**hostname**：
查看：hostname
配置：hostname  HOSTNAME
当前系统有效，重启后无效；

**hostnamectl**

主机名称配置工具

```
status                 Show current hostname settings
set-hostname NAME      Set system hostname
set-icon-name NAME     Set icon name for host
set-chassis NAME       Set chassis type for host
set-deployment NAME    Set deployment environment for host
set-location NAME      Set location for host
```

配置文件：/etc/sysconfig/network

```
HOSTNAME=<HOSTNAME>
```

注意：此方法的设置不会立即生效； 但以后会一直有效；

#### 配置DNS服务器

配置文件：/etc/resolv.conf

```
nameserver   DNS_SERVER_IP
```

如何测试(host/nslookup/dig)

```
# dig  -t  A  FQDN
FQDN --> IP

# dig  -x  IP
IP --> FQDN
```



### iproute2命令家族

#### ip

显示或修改路由、设备、协议或通道

常用用法：

- link

  - set interface {up|down}：启用或禁用网卡
  - show interface：显示指定网卡信息

- addr

  - add IP/NETMASK [label interface:#] [scope {global | link | host}] [broadcast IP] dev interface：添加配置临时地址

    - label：指定别名

    - scope：作用域

    - - global：作用域为全局
      - link：仅和此网卡相连的网络生效
      - host：仅主机可用

    - broadcast：设定广播地址

  - del dev interface [label interface:#]：删除IP

  - flush dev interface [label interface:#]：清空IP

- route
  - add IP/NETMASK via GATEWAY dev interface：添加路由
  - add default via GATEWAY dev interface：添加默认路由
  - del IP/NETMASK via GATEWAY dev interface：删除路由
  - flush：清空路由表
  - show：查看路由表

常用命令：

```
ip help                                                 查看ip命令使用帮助
ip link                                                   查看数据链路层信息
ip link set eth1 up|down                          设置eth1网卡启用|禁用
ip address|a                                       查看网卡信息
ip route|r                                            查看路由信息
ip route add|del IP/24 via gateway                添加路由
ip address add 2.2.2.2/24 dev eth0                添加IP地址
ip address add 2.2.2.2/24 dev eth0 label eth0：2添加别名网卡IP地址
ip address flush dev eth0                         清空eth0网卡上所有ip地址
```

#### ss

显示网络连接信息

选项：

- -n：以数字方式显示，不解析，提高效率
- -t：TCP相关
- -u：UDP相关
- -w：裸套接字
- -x：显示unix sock相关信息
- -l：查看处于监听状态的端口
- -a：查看所有状态的端口
- -e：显示更详细的信息
- -p：查看相关的进程PID
- -m：内存用量
- -o：计时器信息
- -s：显示当前socket详细信息
- state TCP_STATE '( dport = :ssh or sport = :ssh )'
  - established
  - listen
  - fin_wait_1
  - fin_wait_2
  - syn_sent
  - syn_recv

常见用法：

```
ss -l                # 显示本地打开的所有端口
ss -pl               # 显示每个进程具体打开的socket
ss -t -a             # 显示所有tcp socket
ss -u -a             # 显示所有的UDP Socekt
ss -o state established ‘( dport = :ssh or sport = :ssh )’   #显示所有已建立的ssh连接
ss -o state established ‘( dport = :http or sport = :http )’ #显示所有已建立的HTTP连接
ss -s 列出当前socket详细信息
```

### nm家族

#### nmcli

命令行工具，地址配置工具（CentOS7.x）；

子命令补全功能：`yum install bash-completion` ，依赖epel源。

语法： `nmcli [OPTIONS] OBJECT { COMMAND | help }`

可以使用nmcli OBJECT --help 查看帮助

对象：

```
  g[eneral]       网络管理器总的状态和选项
  n[etworking]    整体网络控制
  r[adio]         网络管理器无线交换机
  c[onnection]    网络管理器的连接
  d[evice]        被网络管理器管理的设备
  a[gent]         网络管理器安全代理
  m[onitor]       监视网络管理器变化
```

选项：

```
 -o[verview]                 概述模式 (隐藏默认值)
  -t[erse]                   简洁的输出
  -p[retty]                  漂亮的输出
  -m[ode] tabular|multiline          输出模式
  -c[olors] auto|yes|no         是否在输出中使用颜色
  -f[ields] <field1,field2,...>|all|common    指定输出字段
  -g[et-values] <field1,field2,...>|all|common  快捷方式- m表格- t - f
  -e[scape] yes|no     逃避列分隔符的值
  -a[sk]               找失踪的参数
  -s[how-secrets]      允许显示密码
  -w[ait] <seconds>   设置超时等待完成操作
```

COMMAND

	device - show and manage network interfaces
	COMMAND := { status | show | connect | disconnect | delete | wifi | wimax }
	
	connection - start, stop, and manage network connections
	COMMAND := { show | up | down | add | edit | modify | delete | reload | load }
				
	modify [ id | uuid | path ] <ID> [+|-]<setting>.<property> <value>
				
	如何修改IP地址等属性：
	# nmcli  conn  modify  IFACE  [+|-]setting.property  value
	ipv4.address
	ipv4.gateway
	ipv4.dns1
	ipv4.method
	manual
常见用法：

```
 nmcli connection show           # 查看当前连接状态
 nmcli connection reload         # 重启服务
 nmcli connection show -active   # 显示活动的连接
 nmcli connection show "lan eth0" # 显示指定一个网络连接配置
 nmcli device status             # 显示设备状态
 nmcli device show eno16777736   # 显示指定接口属性
 nmcli device show               # 显示全部接口属性
 nmcli con up static             # 启用static连接配置
 nmcli con up default            # 启用default连接配置 
 nmcli con add help              # 查看帮助
```

#### nmtui

可视化网络配置工具

### 网络客户端命令

#### ifup/ifdown

控制网卡接口是否开启。

用法：ifup <设备名>

​	ifdown <设备名>

#### ping

安装`yum install initscripts`。

一般用于检测网络通与不通。ping发送一个ICMP回声请求消息给目的地并报告是否收到所希望的ICMP回声应答。它是用来检查网络是否通畅或者网络连接速度的命令。

网络上的机器都有唯一确定的IP地址，我们给目标IP地址发送一个数据包，对方就要返回一个同样大小的数据包，根据返回的数据包我们可以确定目标主机的存在，可以初步判断目标主机的操作系统等。

语法：`ping [OPTIONS] IP/HOSTNAME`

选项：

- -c count：ping的次数
- -W timeout：超时时间，配合-c使用
- -I ipaddress：指定用自己主机的IP去ping对方主机
- -s size：每次ping发出的数据包大小，最大值65507
- -f：竭尽自己主机的能力发出数据包



使用Ping检查连通性：		

```
•1. 使用 ifconfig观察本地网络设置是否正确;
•2. Ping 127.0.0.1，127.0.0.1回送地址Ping回送地址是为了检查本地的TCP/IP协议有没有设置好;
•3. Ping本机IP地址，这样是为了检查本机的IP地址是否设置有误;
•4. Ping本网网关或本网IP地址，这样的是为了检查硬件设备是否有问题，也可以检查本机与本地网络连接是否正常;(在非局域网中这一步骤可以忽略)
•5.Ping本地DNS地址，这样做是为了检查DNS是否能够将IP正确解析。
•6.Ping远程IP地址，这主要是检查本网或本机与外部的连接是否正常。
```

#### tcpdump

抓包工具

- -n：禁止解析IP
- -i interface：指定网卡接口
- tcp|udp|icmp|arp：指定包协议

#### mtr

网络诊断工具

#### traceroute

跟踪路由

语法：`traceroute [OPTION] URL`

#### tracepath

跟踪路由

语法：`tracepath [OPTION] URL`

#### wget

网络下载工具

语法：`links [OPTIONS] URL`

选项：

- -q：静默模式
- -c：断点续传
- -P /path/DIRNAME：下载的文件保存到指定文件夹
- -O /path/FILENAME：下载的文件保存到指定位置，并重命名
- --limit-rate=# K|M：限速至# K|M

#### links

字符界面web浏览器，安装：`yum install links`

语法：`links [OPTIONS] URL`

- -source：查看网页源代码
- -dump：只显示文字

#### 登录用户通信

**write**
给登录用户发送信息

	write user [tty] 
	EOF结束

**wall**

	给所有在线用户发送信息
	wall 信息

**mail**
mail — 发送和接收邮件

用法：`mail -eiIUdEFntBDNHRVv~ -T FILE -u USER -h hops -r address -s SUBJECT -a FILE -q FILE -f FILE -A ACCOUNT -b USERS -c USERS -S OPTION users`

#### 网桥管理工具brctl 

**安装**

Centos系统

```
$ yum install bridge-utils
```

Ubuntu系统   

```
$ apt-get  install bridge-utils
```

**使用**

1.添加网桥(br0)

```
$ brctl addbr br0
```

注：设置br0可用

```
$ sudo ifconfig br0 192.168.100.1 netmask  255.255.255.0
```

2.查看网桥

  1）显示所有的网桥信息

       $ sudo brctl show

 2）显示某个网桥(br0)的信息

      $ sudo brctl show br0

3.删除网桥(br0) 

```
$  sudo brctl delbr br0 
```

4. 将eth0端口加入网桥br0

    ```
    $ brctl addif br0 eth0
    ```

5. 从网桥br0中删除eth0端口

    ```
    $ brctl delif br0 eth0
    ```

其他：

```
Usage: brctl [commands]
commands:
	addbr     	<bridge>		add bridge
	delbr     	<bridge>		delete bridge
	addif     	<bridge> <device>	add interface to bridge
	delif     	<bridge> <device>	delete interface from bridge
	hairpin   	<bridge> <port> {on|off}	turn hairpin on/off
	setageing 	<bridge> <time>		set ageing time
	setbridgeprio	<bridge> <prio>		set bridge priority
	setfd     	<bridge> <time>		set bridge forward delay
	sethello  	<bridge> <time>		set hello time
	setmaxage 	<bridge> <time>		set max message age
	setpathcost	<bridge> <port> <cost>	set path cost
	setportprio	<bridge> <port> <prio>	set port priority
	show      	[ <bridge> ]		show a list of bridges
	showmacs  	<bridge>		show a list of mac addrs
	showstp   	<bridge>		show bridge stp info
	stp       	<bridge> {on|off}	turn stp on/off
```

