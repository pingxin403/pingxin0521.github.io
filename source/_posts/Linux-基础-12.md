---
title: Linux-服务管理
date: 2019-03-23 11:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

服务其实是一款软件，这个软件可以提供服务，例如，远程安全登录ssh使用的服务就是sshd所提供的。有一些服务是相互依赖的，依赖其他服务。比如sftp服务依赖于ssh服务。

说白了，服务就是一个进程，有一个固定且已知的端口，然后当该进程开启时，通过其他主机进程或本机其他进程通信，提供服务。

而服务管理，可以算上的任务管理和进程管理的结合，只是针对一定的软件对象。

<!--more-->

CentOS 7系统有默认的服务管理器，只能管理使用二进制包安装（使用rpm或yum）的服务，对于源码包安装的服务，默认不能管理。

CentOS 7使用新的机制管理服务，管理工具改为 `systemd`。

Systemd：系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程

#### unit类型  

systemd核心概念unit（单元）类型：unit表示不同类型的systemd对象，通过配置文件进行标识和配置；
		文件中主要包含了系统服务、监听socket、保存的系统快照以及其它与init相关的信息

```
service ：文件扩展名为.service, 用于定义系统服务

target ：文件扩展名为.target，用于模拟实现运行级别

device  ：用于定义内核识别的设备 

mount：定义文件系统挂载点   

socket：用于标识进程间通信用的socket文件，也可在系统启动时，延迟启动服务，实现按需启动 

snapshot ：管理系统快照  

swap：用于标识swap设备   

automount ：文件系统的自动挂载点  

path：用于定义文件系统中的一个文件或目录使用,常用于当文件系统变化时，延迟激活服务
```

那么如何查看这些类型呢?其实很简单只需执行 systemctl -t service-type

示例：

```shell
[hyp@localhost ~]$ systemctl -t target
UNIT                  LOAD   ACTIVE SUB    DESCRIPTION
basic.target          loaded active active Basic System
cryptsetup.target     loaded active active Local Encrypted Volumes
getty.target          loaded active active Login Prompts
local-fs-pre.target   loaded active active Local File Systems (Pre)
local-fs.target       loaded active active Local File Systems
multi-user.target     loaded active active Multi-User System
network-online.target loaded active active Network is Online
network-pre.target    loaded active active Network (Pre)
network.target        loaded active active Network
paths.target          loaded active active Paths
remote-fs-pre.target  loaded active active Remote File Systems (Pre)
remote-fs.target      loaded active active Remote File Systems
rpcbind.target        loaded active active RPC Port Mapper
slices.target         loaded active active Slices
sockets.target        loaded active active Sockets
sound.target          loaded active active Sound Card
swap.target           loaded active active Swap
sysinit.target        loaded active active System Initialization
timers.target         loaded active active Timers
....
```



#### 常用命令：

```
Systemctl start <单元>立即启动单元
Systemctl stop <单元>立即关闭单元
Systemctl restart <单元>立即重启单元
Systemctl reload <单元>重读单元配置 （类似刷新）
Systemctl status <单元> 输出单元运行状态
Systemctl is-enable <单元> 查看单元是否自启动
Systemctl enable <单元> 设置开机自启
Systemctl disable <单元> 取消开机自启
Systemctl is-active <单元>查看单元是否正在运行
systemctl mask .service #禁用指定服务
systemctl unmask .service #激活指用服务 
```



#### 分析系统状态：

```
显示 系统状态:

$ systemctl status
输出激活的单元：

$ systemctl
下面命令等效：

$ systemctl list-units
输出执行失败的单元：

$ systemctl --failed
全部可用的单元文件存放在 /usr/lib/systemd/system/ 和 /etc/systemd/system/ 文件夹（后者优先级更高）。

查看全部已安装服务：

$ systemctl list-unit-files
```

#### 电源管理：

安装 [polkit](https://wiki.archlinux.org/index.php/Polkit) 后才能够让一般用户身份使用电源管理。

```
重新启动：

$ systemctl reboot
退出系统并停止电源：

$ systemctl poweroff
待机：

$ systemctl suspend
休眠：

$ systemctl hibernate
混合休眠模式（同一时候休眠到硬盘并待机）：

$ systemctl hybrid-sleep
```

#### 单元文件：

Unit文件存放目录（从下到上优先级增加）

- /etc/systemd/system 系统或用户提供的配置文件（特权用户存放）


- /run/systemd/system 软件运行时生成的配置文件（非特权用户存放）


- /use/lib/systemd/system 系统或第三方软件配置文件（系统更新将被覆盖）

##### 支持文件类型

```
automount 自动挂载文件系统

.device 主要用于定义设备之间的依赖，对应/dev目录下的设备

.mount 替代/etc/fstab文件

.path 用于监控指定目录的变化，并触发其他unit运行

.scope Systemd管理，描述系统服务的分组信息

.service 守护进程的相关操作

.slice 描述cgroup的信息

.snapshot Systemd unit运行状态的快照

.swap 定义虚拟内存的交换分区

.target 对unit进行逻辑分组，引导其他unit运行，替代SYSV运行级别。

.timer 由Systemd中时间触发的动作，替代crontab

```

##### Service文件字段说明

[ Unit ]

1、Description

描述文字

2、Documentation

文档，可以是一个或多个文档的URL路径

3、Requires

依赖列表，在当前服务启动时同时启动，如何有失败，则当前服务将被终止。

4、Wants

依赖列表，不考虑是否启动成功。

5、After

依赖列表，列表中所有模块启动完成，才会启动当前服务。

6、Before

启动当前服务后，才启动列表中的模块。

7、BindsTo

强关联依赖列表，在运行过程中，如果列表服务意外结束或重启，当前服务也会跟着终止或重启。

8、PartOf

BindsTo的子集，只有当PartOf列出的模块失败或重启时，当前服务才终止或重启。

9、OnFailure

当前服务启动失败时，则启动列表中模块。

10、Conflicts

定义冲突模块，如果列表中模块已经在运行，则不启动当前服务。

[ Install ]

说明：此段中的配置需要通过systemctl enable命令来激活，通过systemctl disable命令禁用。

1、WantedBy

与Wants作用相似，只是列出的不是服务所依赖的模块，而是依赖当前服务的模块。

2、RequiredBy

依赖当前服务的模块。

3、Also

当前服务被enable/disable时，将自动enable/disable列表中的模块

[ Service ]

服务生命周期控制

1、Type

服务的类型，simple（默认）、forking。如果服务程序启动后会通过fork系统调用创建子进程，然后关闭程序本身进程，则应该将Type设置为forking，否则Systemd将不会跟踪子进程的行为，而认为服务已经退出。

2、RemainAfterExit

true/false 默认为false。当值为true时，Systemd只负责启动服务进程，之后即便退出仍会认为服务在运行。主要提供给一些非常驻内存，而是启动后立即退出，然后等待消息按需启动的特殊类型。

3、ExecStart

指定服务启动的主要命令，仅一个。

4、ExecStartPre

指定在启动ExecStart命令前的准备工作，可以有多个。

5、ExecStartPost

指定在启动ExecStart命令后的收尾工作，可以有多个。

6、TimeoutStartSec

启动服务的等待秒数，超时则Systemd认为服务启动失败，设置为0关闭超时检测。

7、ExecStop

停止服务所需要执行的主要命令。

8、ExecPost

指定在ExecStop命令执行后的收尾工作，可以有多个。

9、TimeoutStopSec

停止服务的等待时间，超时则认为没有成功停止，Systemd会使用SIGKILL信号强行杀死服务进程。

10、Restart

指定在什么情况下需要重启服务进程。常用值：no（默认）、no-success、on-failure、on-abnormal、on-abort、always。

| **服务退出原因** | **no** | **always** | **on-failure** | **on-abnormal** | **on-abort** | **no-success** |
| ---------------- | ------ | ---------- | -------------- | --------------- | ------------ | -------------- |
| 正常退出         |        | √          |                |                 |              | √              |
| 异常退出         |        | √          | √              |                 |              |                |
| 启动/停止超时    |        | √          | √              | √               |              |                |
| 被异常KILL       |        | √          | √              | √               | √            |                |

11、RestartSec

如果服务需要被重启，该值为服务被重启前的等待秒数。

12、ExecReload

重新加载服务所需要执行的主要命令。

13、Environment

为服务添加环境变量。

14、Nice

服务的进程优先级，值越小优先级越高，默认为0，-20~19（不建议低于-5（内核中断优先级））。

15、WorkingDirectory

指定服务的工作目录。

16、RootDirectory

指定服务进程的根目录，如果配置了此参数，服务将无法访问指定目录以外的任何文件。

17、User

指定运行服务的用户。

18、Group

指定运行服务的用户组。

19、LimitCPU/LimitSTACK/LimitNOFILE/LimitNPROC

限制服务可用的系统资源。

20、PIDFile

指定PID文件目录。

修改完成记得使用：

sudo systemctl daemon-reload 重载Unit文件

#### 目标（target）

启动级别（runlevel）是一个旧的概念。

如今，systemd 引入了一个和启动级别功能相似又不同的概念——目标（target）。不像数字表示的启动级别，每一个目标都有名字和独特的功能，而且能同一时候启用多个。一些目标继承其它目标的服务，并启动新服务。

systemd 提供了一些模仿 sysvinit 启动级别的目标。仍能够使用旧的 telinit 启动级别 命令切换。

##### 获取当前目标

不要使用 runlevel 命令了：

```
$ systemctl list-units --type=target
```

##### 创建新目标

在 Fedora 中，启动级别 0、1、3、5、6 都被赋予特定用途。而且都对应一个 systemd 的目标。然而，没有什么非常好的移植用户定义的启动级别（2、4）的方法。要实现相似功能，能够以原有的启动级别为基础。创建一个新的目标 /etc/systemd/system/<新目标>（能够參考 /usr/lib/systemd/system/graphical.target），创建 /etc/systemd/system/<新目标>.wants 文件夹，向当中加入额外服务的链接（指向 /usr/lib/systemd/system/ 中的单元文件）。

##### 目标表

| SysV 启动级别 | Systemd 目标                                          | 凝视                                                    |
| ------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| 0             | runlevel0.target, poweroff.target                     | 中断系统（halt）                                        |
| 1, s, single  | runlevel1.target, rescue.target                       | 单用户模式                                              |
| 2, 4          | runlevel2.target, runlevel4.target, multi-user.target | 用户自己定义启动级别。通常识别为级别3。                 |
| 3             | runlevel3.target, multi-user.target                   | 多用户，无图形界面。用户能够通过终端或网络登录。        |
| 5             | runlevel5.target, graphical.target                    | 多用户。图形界面。继承级别3的服务。并启动图形界面服务。 |
| 6             | runlevel6.target, reboot.target                       | 重新启动                                                |
| emergency     | emergency.target                                      | 急救模式（Emergency shell）                             |

##### 切换启动级别/目标

systemd 中。启动级别通过“目标单元”訪问。通过例如以下命令切换：

```
# systemctl isolate graphical.target
```

该命令对下次启动无影响。

等价于telinit 3 或 telinit 5。

##### 改动默认启动级别/目标

开机启动进的目标是 default.target。默认链接到 graphical.target （大致相当于原来的启动级别5）。

能够通过内核參数更改默认启动级别：

- systemd.unit=multi-user.target （大致相当于级别3）
- systemd.unit=rescue.target （大致相当于级别1）

还有一个方法是改动 default.target。能够通过 systemctl 改动它：

```
# systemctl set-default multi-user.target
```

要覆盖已经设置的default.target。请使用 force:

```
# systemctl set-default -f multi-user.target
```

能够在 systemctl 的输出中看到命令执行的效果：链接 /etc/systemd/system/default.target 被创建。指向新的默认启动级别。



参考：[Systemd 中的Unit 文件](https://blog.csdn.net/fanper/article/details/80758086)