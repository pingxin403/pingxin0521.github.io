---
title: Linux-进程和任务管理
date: 2019-03-23 21:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

前一章我们学习了CentOS上的程序管理，那么相对于程序的静态，进程的动态则是更加重要，因为我们的任务都是由进程完成的，为了让计算机更易理解，我们所提交的任务都会被认为是作业，每个作业都会由至少一个进程处理，这样，计算机便完成了我们提出的任务。另外，当一些计划或任务需要实施时，就需要用到任务管理，比如yum 源更新、磁盘检查等等。最后我们会讲到存在于我们内存中虚拟文件系统proc，这个文件系统和我们的内核息息相关。

<!--more-->

### 进程管理

进程是程序在一个数据集合上的一次运行，存在生命周期。

子进程都是由其父进程创建（fork），最初进程称为init进程，为所有进程的父进程。进程具有优先级之分，0-99为实时优先级，100-139为静态优先级，数字越小，优先级越高。用户可以通过nice值来控制进程的优先级，nice的值是-20 到19，对应静态优先级。可以通过`nice`命令查看进程的nice值，通过`renice`命令更改进程的nice值。

linux是抢占式多任务操作系统。进程类型分为前台进程和守护进程。守护进程是在系统引导过程中启动的进程，跟终端无关的进程；前台进程是跟终端相关，通过终端启动的进程。也可把在前台启动的进程送往后台，以守护模式运行。

linux上进程有多个状态：运行态、就绪态、睡眠态、停止态、僵死态，进程分为CPU进程和IO进程。

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1cwdukeefj30hn09kt95.jpg)

进程又分成用户态和内核态，当用户开启一个进程时，该进程处于用户态，当该进程通过调用system call或者接受IO中断或者进程内部错误时，陷入内核态。即当用户态进程遇到中断时，该进程陷入内核态，即可以使用内核管理的硬件资源。

#### CentOS7上的进程管理命令

**pstree**

以树的形状显示进程，安装：`sudo yum install psmisc`

语法：`pstree [option] [PID | USER]`

选项：

```
-a 显示进程启动时的参数
-A 使用ASCII显示
-c 不压缩显示，可以和-g、-p一起使用
-h 当前进程高亮
-H PID 指定进程高亮
-g显示进程组id
-n 根据PID排序
-N type 根据type排序（ipc, mnt, net, pid, user, uts）
-p 显示PID
-s显示选择进程的父进程
PID 从这个PID开始显示，默认为1
User 只显示该用户进程
```

示例：

```shell
[hyp@localhost ~]$ pstree hyp
sshd───bash───pstree
[hyp@localhost ~]$ pstree -cp
systemd(1)─┬─NetworkManager(6686)─┬─{NetworkManager}(6695)
           │                      └─{NetworkManager}(6704)
           ├─VGAuthService(6437)
           ├─agetty(7209)
           ├─auditd(6250)─┬─audispd(6254)─┬─sedispatch(6258)
           │              │               └─{audispd}(6259)
           │              └─{auditd}(6253)
           ├─chronyd(6458)
           ├─crond(7200)
           ├─dbus-daemon(6388)───{dbus-daemon}(6419)
           ├─dnsmasq(7388)───dnsmasq(7389)
           ├─firewalld(8407)───{firewalld}(8638)
           ├─ksmtuned(6530)───sleep(11638)
           ├─libvirtd(7184)─┬─{libvirtd}(7210)
           │                ├─{libvirtd}(7211)
           │                ├─{libvirtd}(7212)
           │                ├─{libvirtd}(7213)
           │                ├─{libvirtd}(7214)
           │                ├─{libvirtd}(7215)
           │                ├─{libvirtd}(7216)
           │                ├─{libvirtd}(7217)
           │                ├─{libvirtd}(7218)
           │                ├─{libvirtd}(7219)
           │                ├─{libvirtd}(7221)
           │                ├─{libvirtd}(7222)
           │                ├─{libvirtd}(7223)
           │                ├─{libvirtd}(7224)
           │                ├─{libvirtd}(7225)
           │                └─{libvirtd}(7238)
           ├─lvmetad(3151)
           ├─master(7500)─┬─pickup(10785)
           │              └─qmgr(7504)
           ├─polkitd(6383)─┬─{polkitd}(6415)
           │               ├─{polkitd}(6426)
           │               ├─{polkitd}(6430)
           │               ├─{polkitd}(6432)
           │               ├─{polkitd}(6451)
           │               └─{polkitd}(6513)
           ├─rpcbind(6396)
           ├─rsyslogd(7176)─┬─{rsyslogd}(7187)
           │                └─{rsyslogd}(7188)
           ├─sshd(7592)───sshd(7596)───bash(7597)───pstree(11640)
           ├─sshd(10449)
           ├─systemd-journal(3131)
           ├─systemd-logind(6385)
           ├─systemd-udevd(3163)
           ├─tuned(7175)─┬─{tuned}(7574)
           │             ├─{tuned}(7575)
           │             ├─{tuned}(7576)
           │             └─{tuned}(7590)
           ├─vmtoolsd(6441)───{vmtoolsd}(6580)
           └─vsftpd(7183)
```

**ps命令**

ps命令用于查看系统中的进程状态，格式为“ps [参数]”。

ps命令的常见参数以及作用如表示。

| 参数 | 作用                               |
| ---- | ---------------------------------- |
| -a   | 显示所有进程（包括其他用户的进程） |
| -u   | 用户以及其他详细信息               |
| -x   | 显示没有控制终端的进程             |
| -e   | 显示所有进程                       |
| -f   | 显示完整格式的进程信息             |
| -F   | 显示完整格式的进程信息             |
| -H   | 以层级结构显示进程的相关信息       |

常用组合选项：aux、ef、eFH

Linux系统中时刻运行着许多进程，如果能够合理地管理它们，则可以优化系统的性能。在Linux系统中，有5种常见的进程状态，分别为运行、中断、不可中断、僵死与停止，其各自含义如下所示。

> **R（运行）：**进程正在运行或在运行队列中等待。
>
> **S（中断）：**进程处于休眠中，当某个条件形成后或者接收到信号时，则脱离该   状态。
>
> **D（不可中断）：**进程不响应系统异步信号，即便用kill命令也不能将其中断。
>
> **Z（僵死）：**进程已经终止，但进程描述符依然存在, 直到父进程调用wait4()系统函数后将进程释放。
>
> **T（停止）：**进程收到停止信号后停止运行。

当执行ps aux命令后通常会看到如表2-7所示的进程状态，表2-7中只是列举了部分输出值，而且正常的输出值中不包括中文注释。

表2-7                                                             进程状态

| USER         | PID      | %CPU             | %MEM       | VSZ                      | RSS                        | TTY      | STAT     | START        | TIME              | COMMAND                  |
| ------------ | -------- | ---------------- | ---------- | ------------------------ | -------------------------- | -------- | -------- | ------------ | ----------------- | ------------------------ |
| 进程的所有者 | 进程ID号 | 运算器占用率     | 内存占用率 | 虚拟内存使用量(单位是KB) | 占用的固定内存量(单位是KB) | 所在终端 | 进程状态 | 被启动的时间 | 实际使用CPU的时间 | 命令名称与参数           |
| root         | 1        | 0.0              | 0.4        | 53684                    | 7628                       | ?        | Ss       | 07:22        | 0:02              | /usr/lib/systemd/systemd |
| root         | 2        | 0.0              | 0.0        | 0                        | 0                          | ?        | S        | 07:22        | 0:00              | [kthreadd]               |
| root         | 3        | 0.0              | 0.0        | 0                        | 0                          | ?        | S        | 07:22        | 0:00              | [ksoftirqd/0]            |
| root         | 5        | 0.0              | 0.0        | 0                        | 0                          | ?        | S<       | 07:22        | 0:00              | [kworker/0:0H]           |
| root         | 7        | 0.0              | 0.0        | 0                        | 0                          | ?        | S        | 07:22        | 0:00              | [migration/0]            |
|              | ………………   | 省略部分输出信息 | ………………     |                          |                            |          |          |              |                   |                          |



> 如前面所提到的，在Linux系统中的命令参数有长短格式之分，长格式和长格式之间不能合并，长格式和短格式之间也不能合并，但短格式和短格式之间是可以合并的，合并后仅保留一个-（减号）即可。另外ps命令可允许参数不加减号（-），因此可直接写成ps aux的样子。

**top命令**

top命令用于动态地监视进程活动与系统负载等信息，其格式为top。

top命令相当强大，能够动态地查看系统运维状态，完全将它看作Linux中的“强化版的Windows任务管理器”。top命令的运行界面下所示。

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1cxn8gxpoj30ma0e6wen.jpg)

top命令执行结果的前5行为系统整体的统计信息，其所代表的含义如下。

> 第1行：系统时间、运行时间、登录终端数、系统负载（三个数值分别为1分钟、5分钟、15分钟内的平均值，数值越小意味着负载越低）。
>
> 第2行：进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。
>
> 第3行：用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。其中数据均为CPU数据并以百分比格式显示，例如“97.1 id”意味着有97.1%的CPU处理器资源处于空闲。
>
> 第4行：物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。
>
> 第5行：虚拟内存总量、虚拟内存使用量、虚拟内存空闲量、已被提前加载的内存量。

运行时命令		

```
排序：
P：以占据CPU百分比排序；
M：以占据内存百分比排序；
T：累积占用CPU时间排序；
首部信息：
uptime信息：l命令
tasks及cpu信息：t命令
内存信息：m命令
退出命令：q
修改刷新时间间隔：s
终止指定的进程：k
```

选项：

```
-d #：指定刷新时间间隔，默认为3秒；
-b：以批次方式显示；
-n #：显示多少批次；
```

**uptime命令**

显示系统时间、运行时长及平均负载；过去1分钟、5分钟和15分钟的平均负载；等待运行的进程队列的长度；

```
[hyp@localhost ~]$ uptime
 19:07:08 up  3:30,  1 user,  load average: 0.00, 0.01, 0.05
```

**pgrep命令**

pgrep 是通过程序的名字来查询进程的工具，一般是用来判断程序是否正在运行。在服务器的配置和管理中，这个工具常被应用，简单明了；另外，还有一个pkill和killall 应用方法差不多，也是直接杀死运行中的程式；如果你想杀掉单个进程，请用kill 来杀掉。

语法：`pgrep [options] <pattern>`

常用选项：

```
-l  列出程序名和进程ID；
-o  进程起始的ID；
-n  进程终止的ID；
-u uid：effective user
-U uid：read user
-t  TERMINAL：与指定的终端相关的进程；
-a：显示完整格式的进程名；
-P pid：显示此进程的子进程；
```

示例：

```shell
[hyp@localhost ~]$ pgrep -l ssh
7592 sshd
7596 sshd
10449 sshd
[hyp@localhost ~]$ pgrep -lo ssh
7592 sshd
```

**pidof命令**

pidof命令用于查询某个指定服务进程的PID值，格式为`pidof [参数] [服务名称]`。

每个进程的进程号码值（PID）是唯一的，因此可以通过PID来区分不同的进程。例如，可以使用如下命令来查询本机上sshd服务程序的PID：

```
[root@linuxprobe ~]# pidof sshd
2156
```

**kill命令**

kill命令用于终止某个指定PID的服务进程，格式为 `kill  [-s signal|-SIGNAL]  pid...`。

```
显示当前系统可用信号：
kill -l [signal]
每个信号的标识方法有三种：
1) 信号的数字标识；
2) 信号的完整名称；
3) 信号的简写名称；
常用信号：
1） SIGHUP：无须关闭进程而让其重读配置文件；
2）SIGINT：终止正在运行的进程，相当于Ctrl+c
9）SIGKILL：杀死运行中的进程；
15）SIGTERM：终止运行中的进程；
18）SIGCONT：
19）SIGSTOP：
```

接下来，我们使用kill命令把上面用pidof命令查询到的PID所代表的进程终止掉，其命令如下所示。这种操作的效果等同于强制停止sshd服务。

```
[root@linuxprobe ~]# kill 2156
```

**killall命令**

killall命令用于终止某个指定名称的服务所对应的全部进程，格式为：“killall [参数] [服务名称]”。

通常来讲，复杂软件的服务程序会有多个进程协同为用户提供服务，如果逐个去结束这些进程会比较麻烦，此时可以使用killall命令来批量结束某个服务程序带有的全部进程。下面以httpd服务程序为例，来结束其全部进程。由于RHEL7系统默认没有安装httpd服务程序，因此大家此时只需看操作过程和输出结果即可，等学习了相关内容之后再来实践。

```
[root@linuxprobe ~]# pidof httpd
13581 13580 13579 13578 13577 13576
[root@linuxprobe ~]# killall httpd
[root@linuxprobe ~]# pidof httpd
[root@linuxprobe ~]# 
```

如果我们在系统终端中执行一个命令后想立即停止它，可以同时按下Ctrl + C组合键（生产环境中比较常用的一个快捷键），这样将立即终止该命令的进程。或者，如果有些命令在执行时不断地在屏幕上输出信息，影响到后续命令的输入，则可以在执行命令时在末尾添加上一个&符号，这样命令将进入系统后台来执行。

**vmstat命令**

报告虚拟内存统计数据。

语法：vmstat  [options]  [delay [count]]

```
procs：
	r：等待运行的进程的个数；CPU上等待运行的任务的队列长度；
	b：处于不可中断睡眠态的进程个数；被阻塞的任务队列的长度；
memory：
	swpd：交换内存使用总量；
	free：空闲的物理内存总量；
	buffer：用于buffer的内存总量；
	cache：用于cache的内存总量；
swap
	si：数据进入swap中的数据速率（kb/s）
	so：数据离开swap的速率(kb/s)
io
	bi：从块设备读入数据到系统的速度（kb/s）
	bo：保存数据至块设备的速率（kb/s）
system
	in：interrupts，中断速率；
	cs：context switch, 上下文 切换的速率；
cpu 
	us： user space
	sy：system
	id：idle
	wa：wait 
	st: stolen
```

选项：
	-s：显示内存统计数据；

示例：

```
[hyp@localhost ~]$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 355780   2776 461372    0    0    30     5   41   75  0  1 99  0  0
```

### 作业控制

作业（用户提交的任务）分为前台作业和后台作业。前台作业(foregroud)可以通过终端启动，且启动后会一直占据终端；后台作业(backgroud)可以通过终端启动，但启动后即转入后台运行（释放终端）。

进程是完成用户提交作业的操作。



(1) 运行中的作业

```
Ctrl+z
```

​	注意：送往后台后，作业会转为停止态；
(2) 尚未启动的作业

```
#COMMAND & 
```

注意：此类作业虽然被送往后台，但其依然与终端相关；如果希望把送往后台的作业剥离与终端的关系：

```
# nohup  COMMAND  &
```

查看所有的作业：

```
# jobs
```

作业控制命令

```
# fg  [[%]JOB_NUM]：把指定的作业调回前台；
# bg  [[%]JOB_NUM]：让送往后台的作业在后台继续运行；
# kill  %JOB_NUM：终止指定的作业，不是PID；
```

### 任务管理

在Linux操作系统中，除了用户即时执行的命令操作以外，还可以配置在指定的时间、指定的日期执行预先计划好的系统管理任务（如定期备份、定期采集监测数据）。CentOS系统中默认已安装了at、cronie软件包，通过atd和crond这两个系统服务实现一次性、周期性计划任务的功能，并分别通过at、crontab命令进行计划任务设置。

未来某一时间点执行的一次性任务使用at或者batch命令。周期性运行某任务使用crontab命令。运行结果会以邮件的方式发生给用户。

> 本地电子邮件服务：
> smtp：simple mail transmission protocol
> pop3：Post Office Procotol
> imap4：Internet Mail Access Procotol
>
> mail命令：
> 	mailx - send and receive Internet mail
>
> ​	MUA：Mail User Agent, 用户收发邮件的工具程序；
> ​	mailx  [-s 'SUBJECT']  username[@hostname]
> ​		邮件正文的生成：
> ​			(1) 交互式输入；. 单独成行可以表示正文结束；Ctrl+d提交亦可；
> ​			(2) 通过输入重定向；
> ​			(3) 通过管道；

**at命令**

一次性计划任务

前提条件：对应的系统服务atd必须已经运行

查看atd服务是否运行：`sudo systemctl status atd`，若没有运行，则使用 `sudo systemctl start atd` 开启服务，开机自运行：`sudo systemctl enable atd`。

语法：`at  [OPTION]... TIME`

```
TIME：
	HH:MM [YYYY-mm-dd]
	noon，midnight, teatime
	tomorrow
	now+#
		UNIT：minutes, hours, days, OR weeks
```

at的作业有队列，用单个字母表示，默认都使用a队列；

选项：

```
-l：查看作业队列，相当于atq
-f /PATH/FROM/SOMEFILE：从指定文件中读取作业任务，而不用再交互式输入；
-d：删除指定的作业，相当于atrm；
-c：查看指定作业的具体内容；
-q QUEUE：指明队列；
```

注意：作业执行结果是以邮件发送给提交作业的用户；

**batch命令**

batch会让系统自行选择在系统资源较空闲的时间去执行指定的任务；

**corn**

服务程序cronie：主程序包，提供了crond守护进程及相关辅助工具

确保crond守护进程(daemon)处于运行状态：`systemctl  status  crond.service`

向crond提交作业的方式不同于at，它需要使用专用的配置文件，此文件有固定格式，不建议使用文本编辑器直接编辑此文件；要使用crontab命令；

cron任务分为两类：
系统cron任务：主要用于实现系统自身的维护；手动编辑：/etc/crontab文件
用户cron任务：命令：crontab命令

系统cron的配置格式：/etc/crontab

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

注意：

```
(1) 每一行定义一个周期性任务，共7个字段；
定义周期性时间
user-name : 运行任务的用户身份
command to be executed：任务
(2) 此处的环境变量不同于用户登录后获得的环境，因此，建议命令使用绝对路径，或者自定义PATH环境变量；
(3) 执行结果邮件发送给MAILTO指定的用户
```

用户cron的配置格式：/var/spool/cron/USERNAME

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MO=root
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  *   command to be executed	
```

注意：

```
(1) 每行定义一个cron任务，共6个字段；
	(2) 此处的环境变量不同于用户登录后获得的环境，因此，建议命令使用绝对路径，或者自定义PATH环境变量；
	(3) 邮件发送给当前用户；
```

时间表示法：
(1) 特定值；
	给定时间点有效取值范围内的值；
		注意：day of week和day of month一般不同时使用；
(2) *
	给定时间点上有效取值范围内的所有值；表“每..”
(3) 离散取值：,
	在时间点上使用逗号分隔的多个值； 
		#,#,#
(4) 连续取值：-
	在时间点上使用-连接开头和结束
		#-#
(5) 在指定时间点上，定义步长: 
	/#：#即步长；
	
	注意：
		(1) 指定的时间点不能被步长整除时，其意义将不复存在；
		(2) 最小时间单位为“分钟”，想完成“秒”级任务，得需要额外借助于其它机制；
			定义成每分钟任务：而在利用脚本实现在每分钟之内，循环执行多次；

示例：

```
(1) 3 * * * *：每小时执行一次；每小时的第3分钟；
(2) 3 4 * * 5：每周执行一次；每周5的4点3分；
(3) 5 6 7 * *：每月执行一次；每月的7号的6点5分；
(4) 7 8 9 10 *：每年执行一次；每年的10月9号8点7分；
(5) 9 8 * * 3,7：每周三和周日；
(6) 0 8,20 * * 3,7：
(7) 0 9-18 * * 1-5：
(8) */5 * * * *：每5分钟执行一次某任务；
(9) */7
```

**crontab命令**

语法：`crontab [-u user] [-l | -r | -e] [-i]` 

选项：

​	-e：编辑任务；
​	-l：列出所有任务；
​	-r：移除所有任务；即删除/var/spool/cron/USERNAME文件；
​	-i：在使用-r选项移除所有任务时提示用户确认；
​	-u user：root用户可为指定用户管理cron任务；		

注意：运行结果以邮件通知给当前用户；如果拒绝接收邮件：
(1) COMMAND > /dev/null
(2) COMMAND &> /dev/null

注意：定义COMMAND时，如果命令需要用到%，需要对其转义；但放置于单引号中的%不用转义亦可；

如果期望某时间因故未能按时执行，下次开机后无论是否到了相应时间点都要执行一次，可使用anacron实现；

**anacron命令**

如果我们的Linux主机是24全天全年的处于开机状态，我们只需要atd与crond这两个服务即可，如果我们的服务器并非24小时无间断的启动，那么我们就需要anacron的帮助了。

anacron并不能取代cron去运行某项任务，而是以天为单位或者是在启动后立刻进行anacron的动作，它会去侦测停机期间应该进行但是并没有进行的crontab任务，并将该任务运行一遍后，anacron就会自动停止了。

anacron会以一天、七天、一个月周期去侦测系统中未进行的crontab任务，因此对于某些特殊的使用环境非常有帮助。anacron会去会去分析现在的时间与时间记录档所记载的上次运行anacron的时间，两者比较厚若发现有差异，也就是在某些时刻没有进行crontab，那么此时anacron就会开始执行未运行的crontab了。所以anacron也是听过crontab来运行的，因此anacron运行的时间通常由两个，一个是系统启动期间运行，一个是写入crontab的排程中，这样才能够在特定时间分析系统未进行的crontab工作。我们可以使用ll  /etc/cron*/*ana*的方式来查看anacron的侦测时间。但是我们仔细分析该文件的话，发现它主要是执行anacron命令。

anacron命令的语法如下：

```
-s开始连续的运行各项工作，会一句时间记录当的数据判断是否进行。
-f强制进行，而不去判断时间登录档的时间戳。
-n立即进行未进行的任务，而不延迟等待时间。
-u仅升级时间记录当的时间戳，不进行任何工作。
```

而anacron的配置文件是`/etc/anacrontab`，而它的很多内容则是在`/var/spool/anacron`里面保存。

当anacron下达anacron  -s  cron.daily时，它会有如下的步骤：

```
(1)由/etc/anacrontab分析到cron.daily这项工作名称的天数为一天。
(2)由/var/spool/anacron/cron.daily取出最近一次运行anacron的时间戳。
(3)把取出的时间戳与当前的时间戳相比较，如果差异超过了一天，那么就准备进行命令。
(4)若准备进行命令，根据/etc/anacrontab的配置，将延迟65分钟。
(5)延迟时间后，开始运行后续命令，也就是run-parts  /etc/cron.daily这串命令。
(6)运行完毕后，anacron程序结束。
```



**练习**

```
练习：
1、每12小时备份一次/etc目录至/backups目录中，保存文件 名称格式为“etc-yyyy-mm-dd-hh.tar.xz”
 0 */12 * * * tar -cf /etc /backups/etc-`date +%Y-%m-%d-%H`.tar && xz -z /backups/etc-`date +%Y-%m-%d-%H`.tar
 
2、每周2、4、7备份/var/log/secure文件至/logs目录中，文件名格式为“secure-yyyymmdd”；
0 3 * * 2,4,7 cp /var/log/secure /logs/secure-`date +%Y%m%d` 

3、每两小时取出当前系统/proc/meminfo文件中以S或M开头的行信息追加至/tmp/meminfo.txt文件中；
0 */2 * * * grep "^[SM]" /proc/meminfo >> /tmp/meminfo.txt
```



### proc虚拟文件系统

proc文件系统位于/proc/文件夹下，存在于内存中，存储的是内核中状态信息。有以下两种：

内核参数：可设置其值从而调整内核运行特性的参数，位于/proc/sys/；
状态变量：其用于输出内核中统计信息或状态信息，仅用于查看；

使用ls命令查看，可以发现一些以数字命名的文件夹，这些文件夹对应PID为文件夹名的进程，可以从中获取进程信息。

Linux在系统运行时修改内核参数(/proc/sys与/etc/sysctl.conf)，而不需要重新引导系统，这个功能是通过/proc虚拟文件系统实现的。

在/proc/sys目录下存放着大多数的内核参数，并且设计成可以在系统运行的同时进行更改, 可以通过更改/proc/sys中内核参数对应的文件达到修改内核参数的目的(修改过后，保存配置文件就马上自动生效)，不过重新启动机器后之前修改的参数值会失效，所以只能是一种临时参数变更方案。(适合调试内核参数优化值的时候使用，如果设置值有问题，重启服务器还原原来的设置参数值了。简单方便。)

但是如果调试内核参数优化值结束后，需要永久保存参数值，就要通过修改/etc/sysctl.conf内的内核参数来永久保存更改。但只是修改sysctl文件内的参数值，确认保存修改文件后，设定的参数值并不会马上生效，如果想使参数值修改马上生效，并且不重启服务器，可以执行下面的命令：

```
#sysctl –p
```

下面介绍一下/proc/sys下内核文件与配置文件sysctl.conf中变量的对应关系：

由于可以修改的内核参数都在/proc/sys目录下，所以sysctl.conf的变量名省略了目录的前面部分（/proc/sys）。

即将/proc/sys中的文件转换成sysctl中的变量依据下面两个简单的规则：

1．去掉前面部分/proc/sys

2．将文件名中的斜杠变为点

这两条规则可以将/proc/sys中的任一文件名转换成sysctl中的变量名。

例如：

```
/proc/sys/net/ipv4/ip_forward ＝》 net.ipv4.ip_forward
/proc/sys/kernel/hostname ＝》 kernel.hostname
```

可以使用下面命令查询所有可修改的变量名

```
# sysctl –a
```

[linux 内核参数优化](https://www.cnblogs.com/weifeng1463/p/6825532.html)

**free命令**

free命令可以查看内存使用情况

用法：free [options]

选项： -h 人性化显示

​		-l 详细显示

示例：

```shell
[hyp@localhost ~]$ free
              total        used        free      shared  buff/cache   available
Mem:         995896      171200      518008        7844      306688      618540
Swap:       2097148           0     2097148
[hyp@localhost ~]$ free -l
              total        used        free      shared  buff/cache   available
Mem:         995896      171156      518032        7844      306708      618584
Low:         995896      477864      518032
High:             0           0           0
Swap:       2097148           0     2097148
```

下面是对内存查看free命令输出内容的解释：

- total:总计物理内存的大小。
- used:已使用多大。
- free:可用有多少。
- Shared:多个进程共享的内存总额。
- Buffers/cached:磁盘缓存的大小。

第三行(-/+ buffers/cached):

- used:已使用多大。
- free:可用有多少。

### 远程控制服务

#### 配置sshd服务

SSH（Secure [Shell](https://www.linuxcool.com/)）是一种能够以安全的方式提供远程登录的协议，也是目前远程管理Linux系统的首选方式。在此之前，一般使用FTP或Telnet来进行远程登录。但是因为它们以明文的形式在网络中传输账户密码和数据信息，因此很不安全，很容易受到黑客发起的中间人攻击，这轻则篡改传输的数据信息，重则直接抓取服务器的账户密码。

想要使用SSH协议来远程管理Linux系统，则需要部署配置sshd服务程序。sshd是基于SSH协议开发的一款远程管理服务程序，不仅使用起来方便快捷，而且能够提供两种安全验证的方法：

> 基于口令的验证—用账户和密码来验证登录；
>
> 基于密钥的验证—需要在本地生成密钥对，然后把密钥对中的公钥上传至服务器，并与服务器中的公钥进行比较；该方式相较来说更安全。

前文曾多次强调“Linux系统中的一切都是文件”，因此在Linux系统中修改服务程序的运行参数，实际上就是在修改程序配置文件的过程。sshd服务的配置信息保存在/etc/ssh/sshd_config文件中。运维人员一般会把保存着最主要配置信息的文件称为主配置文件，而配置文件中有许多以井号开头的注释行，要想让这些配置参数生效，需要在修改参数后再去掉前面的井号。sshd服务配置文件中包含的重要参数如表9-1所示。



| 参数                              | 作用                                |
| --------------------------------- | ----------------------------------- |
| Port 22                           | 默认的sshd服务端口                  |
| ListenAddress 0.0.0.0             | 设定sshd服务器监听的IP地址          |
| Protocol 2                        | SSH协议的版本号                     |
| HostKey /tc/ssh/ssh_host_key      | SSH协议版本为1时，DES私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_rsa_key | SSH协议版本为2时，RSA私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_dsa_key | SSH协议版本为2时，DSA私钥存放的位置 |
| PermitRootLogin yes               | 设定是否允许root管理员直接登录      |
| StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接  |
| MaxAuthTries 6                    | 最大密码尝试次数                    |
| MaxSessions 10                    | 最大终端数                          |
| PasswordAuthentication yes        | 是否允许密码验证                    |
| PermitEmptyPasswords no           | 是否允许空密码登录（很不安全）      |



在RHEL 7系统中，已经默认安装并启用了sshd服务程序。接下来使用ssh命令进行远程连接，其格式为“ssh [参数] 主机IP地址”。要退出登录则执行exit命令。

```
[root@linuxprobe ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 4f:a7:91:9e:8d:6f:b9:48:02:32:61:95:48:ed:1e:3f.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password:此处输入远程主机root管理员的密码
Last login: Wed Apr 15 15:54:21 2017 from 192.168.10.10
[root@linuxprobe ~]# 
[root@linuxprobe ~]# exit
logout
Connection to 192.168.10.10 closed.
```

如果禁止以root管理员的身份远程登录到服务器，则可以大大降低被黑客暴力破解密码的几率。下面进行相应配置。首先使用Vim文本编辑器打开sshd服务的主配置文件，然后把第48行#PermitRootLogin yes参数前的井号（#）去掉，并把参数值yes改成no，这样就不再允许root管理员远程登录了。记得最后保存文件并退出。

```
[root@linuxprobe ~]# vim /etc/ssh/sshd_config 
 ………………省略部分输出信息………………
 46 
 47 #LoginGraceTime 2m
 48 PermitRootLogin no
 49 #StrictModes yes
 50 #MaxAuthTries 6
 51 #MaxSessions 10
 52
 ………………省略部分输出信息………………
```

再次提醒的是，一般的服务程序并不会在配置文件修改之后立即获得最新的参数。如果想让新配置文件生效，则需要手动重启相应的服务程序。最好也将这个服务程序加入到开机启动项中，这样系统在下一次启动时，该服务程序便会自动运行，继续为用户提供服务。

```
[root@linuxprobe ~]# systemctl restart sshd
[root@linuxprobe ~]# systemctl enable sshd
```

这样一来，当root管理员再来尝试访问sshd服务程序时，系统会提示不可访问的错误信息。虽然sshd服务程序的参数相对比较简单，但这就是在Linux系统中配置服务程序的正确方法。大家要做的是举一反三、活学活用，这样即便以后遇到了陌生的服务，也一样可以搞定了。

```
[root@linuxprobe ~]# ssh 192.168.10.10
root@192.168.10.10's password:此处输入远程主机root用户的密码
Permission denied, please try again.
```

#### 安全密钥验证

加密是对信息进行编码和解码的技术，它通过一定的算法（密钥）将原本可以直接阅读的明文信息转换成密文形式。密钥即是密文的钥匙，有私钥和公钥之分。在传输数据时，如果担心被他人监听或截获，就可以在传输前先使用公钥对数据加密处理，然后再行传送。这样，只有掌握私钥的用户才能解密这段数据，除此之外的其他人即便截获了数据，一般也很难将其破译为明文信息。

一言以蔽之，在生产环境中使用密码进行口令验证终归存在着被暴力破解或嗅探截获的风险。如果正确配置了密钥验证方式，那么sshd服务程序将更加安全。我们下面进行具体的配置，其步骤如下。

**第1步**：在客户端主机中生成“密钥对”。

```
[root@linuxprobe ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):按回车键或设置密钥的存储路径
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):直接按回车键或设置密钥的密码
Enter same passphrase again:再次按回车键或设置密钥的密码
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
40:32:48:18:e4:ac:c0:c3:c1:ba:7c:6c:3a:a8:b5:22 root@linuxprobe.com
The key's randomart image is:
+--[ RSA 2048]----+
|+*..o .          |
|*.o  +           |
|o*    .          |
|+ .    .         |
|o..     S        |
|.. +             |
|. =              |
|E+ .             |
|+.o              |
+-----------------+
```

**第2步**：把客户端主机中生成的公钥文件传送至远程主机：

```
[root@linuxprobe ~]# ssh-copy-id 192.168.10.10
The authenticity of host '192.168.10.20 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 4f:a7:91:9e:8d:6f:b9:48:02:32:61:95:48:ed:1e:3f.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.10.10's password:此处输入远程服务器密码
Number of key(s) added: 1
Now try logging into the machine, with: "ssh '192.168.10.10'"
and check to make sure that only the key(s) you wanted were added.
```

**第3步**：对服务器进行设置，使其只允许密钥验证，拒绝传统的口令验证方式。记得在修改配置文件后保存并重启sshd服务程序。

```
[root@linuxprobe ~]# vim /etc/ssh/sshd_config 
 ………………省略部分输出信息………………
 74 
 75 # To disable tunneled clear text passwords, change to no here!
 76 #PasswordAuthentication yes
 77 #PermitEmptyPasswords no
 78 PasswordAuthentication no
 79 
 ………………省略部分输出信息………………
[root@linuxprobe ~]# systemctl restart sshd
```

**第4步**：在客户端尝试登录到服务器，此时无须输入密码也可成功登录。

```
[root@linuxprobe ~]# ssh 192.168.10.10
Last login: Mon Apr 13 19:34:13 2017
```

#### 远程传输命令

scp（secure copy）是一个基于SSH协议在网络之间进行安全传输的命令，其格式为“scp [参数] 本地文件 远程帐户@远程IP地址:远程目录”。

与第2章讲解的cp命令不同，cp命令只能在本地硬盘中进行文件复制，而scp不仅能够通过网络传送数据，而且所有的数据都将进行加密处理。例如，如果想把一些文件通过网络从一台主机传递到其他主机，这两台主机又恰巧是Linux系统，这时使用scp命令就可以轻松完成文件的传递了。scp命令中可用的参数以及作用如表9-2所示。

| 参数 | 作用                     |
| ---- | ------------------------ |
| -v   | 显示详细的连接进度       |
| -P   | 指定远程主机的sshd端口号 |
| -r   | 用于传送文件夹           |
| -6   | 使用IPv6协议             |



在使用scp命令把文件从本地复制到远程主机时，首先需要以绝对路径的形式写清本地文件的存放位置。如果要传送整个文件夹内的所有数据，还需要额外添加参数-r进行递归操作。然后写上要传送到的远程主机的IP地址，远程服务器便会要求进行身份验证了。当前用户名称为root，而密码则为远程服务器的密码。如果想使用指定用户的身份进行验证，可使用用户名@主机地址的参数格式。最后需要在远程主机的IP地址后面添加冒号，并在后面写上要传送到远程主机的哪个文件夹中。只要参数正确并且成功验证了用户身份，即可开始传送工作。由于scp命令是基于SSH协议进行文件传送的，而9.2.2小节又设置好了密钥验证，因此当前在传输文件时，并不需要账户和密码。

```
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# scp /root/readme.txt 192.168.10.20:/home
root@192.168.10.20's password:此处输入远程服务器中root管理员的密码
readme.txt 100% 26 0.0KB/s 00:00
```

此外，还可以使用scp命令把远程主机上的文件下载到本地主机，其命令格式为“scp [参数] 远程用户@远程IP地址:远程文件 本地目录”。例如，可以把远程主机的系统版本信息文件下载过来，这样就无须先登录远程主机，再进行文件传送了，也就省去了很多周折。

```
[root@linuxprobe ~]# scp 192.168.10.20:/etc/redhat-release /root
root@192.168.10.20's password:此处输入远程服务器中root管理员的密码
redhat-release 100% 52 0.1KB/s 00:00 
[root@linuxprobe ~]# cat redhat-release 
Red Hat Enterprise Linux Server release 7.0 (Maipo)
```

#### 不间断会话服务

大家在学习sshd服务时，不知有没有注意到这样一个事情：当与远程主机的会话被关闭时，在远程主机上运行的命令也随之被中断。

如果我们正在使用命令来打包文件，或者正在使用脚本安装某个服务程序，中途是绝对不能关闭在本地打开的终端窗口或断开网络链接的，甚至是网速的波动都有可能导致任务中断，此时只能重新进行远程链接并重新开始任务。还有些时候，我们正在执行文件打包操作，同时又想用脚本来安装某个服务程序，这时会因为打包操作的输出信息占满用户的屏幕界面，而只能再打开一个执行远程会话的终端窗口，时间久了，难免会忘记这些打开的终端窗口是做什么用的了。

screen是一款能够实现多窗口远程控制的开源服务程序，简单来说就是为了解决网络异常中断或为了同时控制多个远程终端窗口而设计的程序。用户还可以使用screen服务程序同时在多个远程会话中自由切换，能够做到实现如下功能。

> **会话恢复**：即便网络中断，也可让会话随时恢复，确保用户不会失去对远程会话的控制。
>
> **多窗口**：每个会话都是独立运行的，拥有各自独立的输入输出终端窗口，终端窗口内显示过的信息也将被分开隔离保存，以便下次使用时依然能看到之前的操作记录。
>
> **会话共享**：当多个用户同时登录到远程服务器时，便可以使用会话共享功能让用户之间的输入输出信息共享。

##### 管理远程会话

screen命令能做的事情非常多：可以用-S参数创建会话窗口；用-d参数将指定会话进行离线处理；用-r参数恢复指定会话；用-x参数一次性恢复所有的会话；用-ls参数显示当前已有的会话；以及用-wipe参数把目前无法使用的会话删除，等等。

下面创建一个名称为backup的会话窗口。请各位读者留心观察，当在命令行中敲下这条命令的一瞬间，屏幕会快速闪动一下，这时就已经进入screen服务会话中了，在里面运行的任何操作都会被后台记录下来。

```
[root@linuxprobe ~]# screen -S backup
[root@linuxprobe ~]# 
```

执行命令后会立即返回一个提示符。虽然看起来与刚才没有不同，但实际上可以查看到当前的会话正在工作中。

```
[root@linuxprobe ~]# screen -ls
There is a screen on:
32230.backup (Attached)
1 Socket in /var/run/screen/S-root.
```

要想退出一个会话也十分简单，只需在命令行中执行exit命令即可。

```
[root@linuxprobe ~]# exit
[screen is terminating]
```

在日常的生产环境中，其实并不是必须先创建会话，然后再开始工作。可以直接使用screen命令执行要运行的命令，这样在命令中的一切操作也都会被记录下来，当命令执行结束后screen会话也会自动结束。

```
[root@linuxprobe ~]# screen vim memo.txt
welcome to linuxprobe.com
```

为了演示screen不间断会话服务的强大之处，我们先来创建一个名为linux的会话，然后强行把窗口关闭掉（这与进行远程连接时突然断网具有相同的效果）：

```
[root@linuxprobe ~]# screen -S linux
[root@linuxprobe ~]# 
[root@linuxprobe ~]# tail -f /var/log/messages 
Feb 20 11:20:01 localhost systemd: Starting Session 2 of user root.
Feb 20 11:20:01 localhost systemd: Started Session 2 of user root.
Feb 20 11:21:19 localhost dbus-daemon: dbus[1124]: [system] Activating service name='com.redhat.SubscriptionManager' (using servicehelper)
Feb 20 11:21:19 localhost dbus[1124]: [system] Activating service name='com.redhat.SubscriptionManager' (using servicehelper)
Feb 20 11:21:19 localhost dbus-daemon: dbus[1124]: [system] Successfully activated service 'com.redhat.SubscriptionManager'
Feb 20 11:21:19 localhost dbus[1124]: [system] Successfully activated service 'com.redhat.SubscriptionManager'
Feb 20 11:30:01 localhost systemd: Starting Session 3 of user root.
Feb 20 11:30:01 localhost systemd: Started Session 3 of user root.
Feb 20 11:30:43 localhost systemd: Starting Cleanup of Temporary Directories...
Feb 20 11:30:43 localhost systemd: Started Cleanup of Temporary Directories.
```

由于刚才关闭了会话窗口，这样的操作在传统的远程控制中一定会导致正在运行的命令也突然终止，但在screen不间断会话服务中则不会这样。我们只需查看一下刚刚离线的会话名称，然后尝试恢复回来就可以继续工作了：

```
[root@linuxprobe ~]# screen -ls
There is a screen on:
 13469.linux (Detached)
1 Socket in /var/run/screen/S-root.
[root@linuxprobe ~]# screen -r linux
[root@linuxprobe ~]#
[root@linuxprobe ~]# tail -f /var/log/messages
Feb 20 11:20:01 localhost systemd: Starting Session 2 of user root.
Feb 20 11:20:01 localhost systemd: Started Session 2 of user root.
Feb 20 11:21:19 localhost dbus-daemon: dbus[1124]: [system] Activating service name='com.redhat.SubscriptionManager' (using servicehelper)
Feb 20 11:21:19 localhost dbus[1124]: [system] Activating service name='com.redhat.SubscriptionManager' (using servicehelper)
Feb 20 11:21:19 localhost dbus-daemon: dbus[1124]: [system] Successfully activated service 'com.redhat.SubscriptionManager'
Feb 20 11:21:19 localhost dbus[1124]: [system] Successfully activated service 'com.redhat.SubscriptionManager'
Feb 20 11:30:01 localhost systemd: Starting Session 3 of user root.
Feb 20 11:30:01 localhost systemd: Started Session 3 of user root.
Feb 20 11:30:43 localhost systemd: Starting Cleanup of Temporary Directories...
Feb 20 11:30:43 localhost systemd: Started Cleanup of Temporary Directories.
Feb 20 11:40:01 localhost systemd: Starting Session 4 of user root.
Feb 20 11:40:01 localhost systemd: Started Session 4 of user root.
```

如果我们突然又想到了还有其他事情需要处理，也可以多创建几个会话窗口放在一起使用。如果这段时间内不再使用某个会话窗口，可以把它设置为临时断开（detach）模式，随后在需要时再重新连接（attach）回来即可。这段时间内，在会话窗口内运行的程序会继续执行。

##### 会话共享功能

screen命令不仅可以确保用户在极端情况下也不丢失对系统的远程控制，保证了生产环境中远程工作的不间断性，而且它还具有会话共享、分屏切割、会话锁定等实用的功能。其中，会话共享功能是一件很酷的事情，当多个用户同时控制主机的时候，它可以把屏幕内容共享出来，也就是说每个用户都可以看到相同的内容。

![image.png](https://i.loli.net/2019/09/15/xTXjasZymKbBg6c.png)

要实现会话共享功能，首先使用ssh服务程序将终端A远程连接到服务器，并创建一个会话窗口。

```
[root@client A ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 70:3b:5d:37:96:7b:2e:a5:28:0d:7e:dc:47:6a:fe:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password:此处输入root管理员密码
Last login: Wed May 4 07:56:29 2017
[root@client A ~]# screen -S linuxprobe
[root@client A ~]#
```

然后，使用ssh服务程序将终端B远程连接到服务器，并执行获取远程会话的命令。接下来，两台主机就能看到相同的内容了。

```
[root@client B ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 70:3b:5d:37:96:7b:2e:a5:28:0d:7e:dc:47:6a:fe:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password:此处输入root管理员密码
Last login: Wed Feb 22 04:55:38 2017 from 192.168.10.10
[root@client B ~]# screen -x 
[root@client B ~]
```