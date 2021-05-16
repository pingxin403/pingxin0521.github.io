---
title: Linux-bash特性和文本编辑
date: 2019-03-13 16:55:28
tags: 
 - Linux
 - 脚本
category: 
 - Linux
 - 基础
---

#### 前言

bash的命令语法是Bourne shell命令语法的超集。数量庞大的Bourne shell脚本大多不经修改即可以在bash中执行，只有使用了Bourne的特殊变量或内置命令的脚本才需要修改。 bash的命令语法很多来自Korn shell (ksh) 和 C shell (csh)， 例如命令行编辑，命令历史，目录栈，$RANDOM 和 $PPID 变量，以及POSIX的命令置换语法： $(...)。作为一个交互式的shell，按下TAB键即可自动补全已部分输入的程序名、文件名、变量名等等。

<!--more-->

#### Tab补全

tab补全可以用于文件补全，也可以用于路径补全，还可以补全命令。

如果我们输入的头几个字母是唯一标识，则按一下Tab自动补全，否则要多按一下，出来的是含有我们输入的字母的一些文件或者是路径。

**路径补全**
在给定的起始路径下，以对应路径下的打头字串来逐一匹配起始路径下的每个文件：
tab：
如果能惟一标识，则直接补全；
否则，再一次tab，给出列表；

比如说：

```shell
[root@localhost /]# cd /bo     #按一下tab就可以补全，因为/下只有一个以e开头的目录
[root@localhost /]# cd /boot/    #      这是结果
```


但是如果有多个

```shell
[root@localhost /]# cd /b            #就要按两下tab，出来提示的路径或者文件
bin/ boot/
[root@localhost /]# cd /b
```

**命令补全**
shell程序在接收到用户执行命令的请求，分析完成之后，最左侧的字符串会被当作命令；
命令查找机制：
	查找内部命令；
	根据PATH环境变量中设定的目录，自左而右逐个搜索目录下的文件名；

给定的打头字符串如果能惟一标识某命令程序文件，则直接补全；
不能惟一标识某命令程序文件，再击tab键一次，会给出列表；

```shell
[hyp@localhost ~]$ ll
ll        lldpad    lldptool
[hyp@localhost ~]$ ll
```

直接双击Tab，会匹配所有命令。在`/`双击Tab，显示`/`下的目录。

```shell
[hyp@localhost ~]$ /
bin/   dev/   home/  lib64/ mnt/   proc/  run/   srv/   tmp/   var/
boot/  etc/   lib/   media/ opt/   root/  sbin/  sys/   usr/   web/
```

输入`$`后双击Tab，显示系统所有变量

```shell
$_                          $LOGNAME
$BASH                       $LS_COLORS
$BASH_ALIASES               $MACHTYPE
$BASH_ARGC                  $MAIL
$BASH_ARGV                  $MAILCHECK
$BASH_CMDS                  $OPTERR
$BASH_COMMAND               $OPTIND
$BASH_LINENO                $OSTYPE
$BASHOPTS                   $PATH
$BASHPID                    $PIPESTATUS
$BASH_SOURCE                $PPID
$BASH_SUBSHELL              $PROMPT_COMMAND
$BASH_VERSINFO              $PS1
$BASH_VERSION               $PS2
$colors                     $PS4
$COLUMNS                    $PWD
$COMP_WORDBREAKS            $RANDOM
$DIRSTACK                   $SECONDS
$EUID                       $SELINUX_LEVEL_REQUESTED
$GROUPS                     $SELINUX_ROLE_REQUESTED
$HISTCMD                    $SELINUX_USE_CURRENT_RANGE
$HISTCONTROL                $SHELL
$HISTFILE                   $SHELLOPTS
--More--
```

#### 命令的执行状态

bash通过状态返回值来输出次结果

```shell
成功:0
失败:1-255
命令执行完成后，其状态返回值保存于bash的特殊变量$?
```

执行中中断是130

```shell
[hyp@localhost ~]$ ls  /etc/yum.repos.d/
CentOS-Base.repo      CentOS-Debuginfo.repo  CentOS-Sources.repo
CentOS-Base.repo.bak  CentOS-fasttrack.repo  CentOS-Vault.repo
CentOS-CR.repo        CentOS-Media.repo
[hyp@localhost ~]$ echo $?
0
[hyp@localhost ~]$ ls  /etc/yum.repos.d/^C
[hyp@localhost ~]$ echo $?
130
```

命令正常执行时，有的还回有命令返回值：
根据命令及其功能不同，结果各不相同；

引用命令的执行结果：

```shell
	$(COMMAND)
	或`COMMAND`
```

#### 命令行历史

命令历史一般记录在.bash_history文件中，默认记录1000条。

当前shell中的历史记录会被记录在内存中，只有退出之后才会保存在USERHOME/.bash_history中。

定制history功能可以通过环境变量实现：

```shell
HISTSIZE：shell进程可保留的命令历史的条数；
HISTFILE：持久保存命令历史的文件；默认~/.bash_history
HISTFILESIZE：命令历史文件的大小；
```
history命令：

​	history [-c] [-d 偏移量] [n] 
​	或 history -anrw [文件名] 
​	或 history -ps 参数 [参数...]

​	-c: 清空命令历史；
​	-d offset：删除指定命令历史
​	-r: 从文件读取命令历史至历史列表中；
​	-w：把历史列表中的命令追加至历史文件中；
​	history #：显示最近的#条命令；

调用命令历史列表中的命令：

- !#：再一次执行历史列表中的第#条命令；
- !!：再一次执行上一条命令；
- !STRING：再一次执行命令历史列表中最近一个以STRING开头的命令；
- !number
- !$:前一个命令的参数

注意：命令的重复执行有时候需要依赖于幂等性；

调用上一条命令的最后一个参数：
	快捷键：ESC, .
	字符串：!$

控制命令历史记录的方式:
 	环境变量 HISTCONTROL
	 ignoredups:忽略重复的命令
 	ignorespace:忽略以空白字符开头的命令
 	ignoreboth：以上两者同时生效

```shell
[root@localhost ~]# HISTCONTROL=ignoredups
[root@localhost ~]# echo $HISTCONTROL
 ignoredups
```

#### 命令行展开

~：自动展开为用户的家目录，或指定的用户的家目录
~USERNAME:  给定用户的家目录，比如cd ~，cd ~ nick 
{}：可承载一个以逗号分隔的路径列表，并能够将其展开为多个路径
`/tmp/{x/{a,b},y,z}  = /tmp/x/a/,/tmp/x,b  ,  /tmp/y/,/tmp/z/`

实例：

```shell
创建如下目录结构：
			/tmp/mysysroot/
				|-- bin
				|-- etc
				|   `-- sysconfig
				|       `-- network-scripts
				|-- sbin
				|-- usr
				|   |-- bin
				|   |-- lib
				|   |-- lib64
				|   |-- local
				|   |   |-- bin
				|   |   |-- etc
				|   |   |-- lib
				|   |   `-- sbin
				|   `-- sbin
				`-- var
				    |-- cache
				    |-- log
				    `-- run

~]# mkdir -pv /tmp/mysysroot/{bin,sbin,etc/sysconfig/network-scripts,usr/{bin,sbin,local/{bin,sbin,etc,lib},lib,lib64},var/{cache,log,run}}
```



#### 快捷键

**查找命令**
history 或 h   显示命令历史列表
Ctrl + r          逆向搜索历史命令    -> 多次按ctrl+r可往前查找类似命令；Ubuntu系统可再 /etc/inputrc 末尾添加"\C-f":forward-search-history 设置正向搜索Ctrl + f ，当Ctrl+r搜索过头，便可按Ctrl+f正向搜索
Ctrl + g         从逆向搜索模式退出
Ctrl + p         历史中的上一条命令
Ctrl + n         历史中的下一条命令

**Bang(!)命令**
!num            执行命令历史列表的第num条命令
!!                  执行上一条命令
!str               执行以stri开头的命令
!str:p            打印输出以str开头的命令
!?str?            执行含有stri字符串的最新命令

**控制命令**
Ctrl + 1       清屏
Ctrl + s       阻止屏幕输出 
Ctrl + q      允许屏幕输出
Ctrl + c       终止命令
Ctrl + z       挂起命令

**编辑命令**
Ctrl + a      移到命令行首
Ctrl + e      移到命令行尾
Ctrl + u      从光标处删除至命令行首
Ctrl + k      从光标处删除至命令行尾
Ctrl + w     从光标处删除至字首
Alt  + d     从光标处删除至字尾

#### 引用

```shell
强引用：''
弱引用：""
命令引用：``
```

#### 别名

使用`alias [别名]`查看别名

使用`alias [别名]='COMMAND'`设置别名

撤销别名：`unalias [别名]`

#### 命令hash

缓存此前命令的查找结果：key-value
			key：搜索键
			value：值

使用`hash`命令：

hash：列出
hash -d COMMAND：删除
hash -r：清空

#### IO重定向

Shell提供了数种语法，可以修改默认的IO的来源端和目的端，就是标准输入和输出的地方。

可用于输入的设备：键盘设备、文件系统上的常规文件、网卡等；
可用于输出的设备：显示器、文件系统上的常规文件、网卡等；

程序的数据流有三种：
	输入的数据流；<-- 标准输入(stdin)，键盘；
	输出的数据流：--> 标准输出(stdout)，显示器；
	错误输出流：  --> 错误输出(stderr)，显示器；

**文件描述符（file descriptor）**

| 文件                   | 文件描述符                              |
| ---------------------- | --------------------------------------- |
| 输入文件——标准输入     | 0（默认为终端（网上有说默认为键盘的）） |
| 输出文件——标准输出     | 1（默认为终端）                         |
| 错误输出文件——标准错误 | 2（默认为终端）                         |

**IO重定向**

| <    | 修改标准输入   | sort < ucid.txt       | 默认下，标准输入为终端，此时可以更改为你想要的地方           |
| ---- | -------------- | --------------------- | ------------------------------------------------------------ |
| <<   |                | Command << delimiter  | 从标准输入中读入，直到遇到delimiter分割符                    |
| >    | 修改标准输出   | ls -l > listinfo.txt  | 默认下，标准输出为终端，此时可以修改默认输出的地方。譬如可以将标准输出的内容写在文件中。 如果文件已存在，会被覆盖掉。 |
| >>   | 输出附件到文件 | ls -l >> listinfo.txt | 与[>]不一样的是，[>]会清空原来的内容，而[>>]只是将标准输出追加到文件结尾处。 |
| \|   | 建立管道       | program1 \| program2  | 1. program1的标准输出为program2的标准输入； 2. 管道的执行效率比使用临时文件的程序起码高一个数量级； |

```shell
		# set -C
			禁止覆盖输出重定向至已存在的文件；
			此时可使用强制覆盖输出：>|
		# set +C
			关闭上述特性
```
错误输出流重定向：2>, 2>>

**特殊文件：/dev/null 与/dev/tty**

/dev/null  传送到此文件的数据都会被系统丢掉,，就是输出到一个空设备的意思。

/dev/tty   程序打开此文件时，Linux会自动将它重定向到一个终端。

**绑定重定向**

| Commond >&m  | 标准输出重定向到文件描述符m中 |
| ------------ | ----------------------------- |
| Command <&-  | 关闭标准输入                  |
| Command 0>&- | 关闭标准输出                  |

此时我们再去理解[2>&1]，就容易多了。[2]是标准错误的文件描述符，而[>&1]的意思重定向到标准输出。那么定时任务的解释就是，将[clear_logs.sh]执行的标准输出和标准错误重定向到[/dev/null]（就是丢掉输出的内容）。

我是这样理解（不一定正确）上面的定时任务的（分2部分）：

```shell
clear_logs.sh > /dev/null     #将clear_logs.sh执行的标准输出输出到/dev/null
clear_logs.sh 2> /dev/null    #将clear_logs.sh执行的标准错误输出到/dev/null，只是clear_logs.sh不是执行了2次，只是1次。这里的&1代表的就是/dev/null
```

**管道**

管道：连接程序，实现将前一个命令的输出直接定向后一个程序当作输入数据流
				COMMAND1 | COMMAND2 | COMMAND3 | ...

tee命令：COMMAND | tee /PATH/TO/SOMEFILE

**tr命令**
tr [OPTION]... SET1 [SET2]
把输入的数据当中的字符，凡是在SET1定义范围内出现的，通通对位转换为SET2出现的字符

​			用法1：
​				tr SET1 SET2 < /PATH/FROM/SOMEFILE
​			用法2：
​				tr -d SET1 < /PATH/FROM/SOMEFILE

注意：不修改原文件

```shell
练习1：把/etc/passwd文件的前6行的信息转换为大写字符后输出；
				head -n 6 /etc/passwd | tr 'a-z' 'A-Z'
```

参考：[[Linux\]基本I/O重定向](https://www.cnblogs.com/rond/p/3530843.html)

#### 多命令执行

使用分号`；`隔开不同命令，命令依次执行。

如：`COMMAND1 ；COMMAND2 ； COMMAND3；...`

通过逻辑运算符：与（&&）、或(||)、非(!)

```shell
			与：
				1 && 1 = 1
				1 && 0 = 0
				0 && 1 = 0
				0 && 0 = 0
			或：
				1 || 1 = 1
				1 || 0 = 1
				0 || 1 = 1
				0 || 0 = 0
			非：
				! 1 = 0
				! 0 = 1
```

短路法则：

~]# COMMAND1 && COMMAND2
	COMMAND1为“假”，则COMMAND2不会再执行；
	否则，COMMAND1为“真”，则COMMAND2必须执行；

~]# COMMAND1 || COMMAND2
	COMMAND1为“真”，则COMMAND2不会再执行；
	否则，COMMAND1为“假”，则COMMAND2必须执行；

#### 变量

程序是由指令和数据组成，指令由程序文件提供，数据则由IO设备、文件、管道、变量等提供。

变量是由变量名和指向的内存空间组成，赋值name=values。

变量类型规定了存储格式、表示数据范围、参与的运算等等。

bash使用弱类型变量，把所有变量统统视作字符型，bash中的变量无需事先声明；相当于，把声明和赋值过程同时实现；声明：类型，变量名。

变量替换：把变量名出现的位置替换为其所指向的内存空间中数据；
变量引用：${var_name}, $var_name
变量名：变量名只能包含数字、字母和下划线，而且不能以数字开头，不能够使用程序的保留字，例如if, else, then, while等等；

**bash变量类型：**

本地变量：作用域仅为当前shell进程；
环境变量：作用域为当前shell进程及其子进程；
局部变量：作用域仅为某代码片断(函数上下文)；

位置参数变量：当执行脚本的shell进程传递的参数；
特殊变量：shell内置的有特殊功用的变量；例如：$?：0：成功；1-255：失败

本地变量：
	变量赋值：name=value
	变量引用：${name}, $name
		""：变量名会替换为其值；
		''：变量名不会替换为其值；
	命令引用：${COMMAND},name=`COMMAND`
	查看变量：set
	撤销变量：unset name
		注意：此处非变量引用；

环境变量：
	变量赋值：
		(1) export name=value
		(2) name=value
			export name
		(3) declare -x name=value
		(4) name=value
			declare -x name
	变量引用：${name}, $name

> 注意：bash内嵌了许多环境变量(通常为全大写字符)，用于定义bash的工作环境
> 	PATH, HISTFILE, HISTSIZE, HISTFILESIZE, HISTCONTROL, SHELL, HOME, UID, PWD, OLDPWD

查看环境变量：export, declare -x, printenv, env
撤销环境变量：unset name

只读变量：
	(1) declare -r name
	(2) readonly name

> 只读变量无法重新赋值，并且不支持撤销；存活时间为当前shell进程的生命周期，随shell进程终止而终止；

在 Linux 系统登录时主要生效的环境变量配置文件有以下 5 个：

- /etc/profile。
- /etc/profile.d/*.sh。
- ~/.bash_profile。
- -/.bashrc。
- /etc/bashrc。

读取顺序：

![](https://ws1.sinaimg.cn/large/006KyevZgy1g19iis2b4xj30go06ddgi.jpg)



**source**

source 命令会强制执行脚本中的全部命令，而忽略脚本文件的权限。该命令主要用于让重新配置的环境变量配置文件强制生效。

source 命令格式如下：

```shell
[root@localhost ~]# source 配置文件
或
[root@localhost ~]#.配置文件
```

举个例子：

```shell
[root@localhost ~]# source -/.bashrc
或
[raot@localhost ~]#. ~/.bashrc
```

"."就是 source 命令，使用哪种方法都是可以的。原来修改了环境变量配置文件，如果要想让其生效，则必须注销或重启系统。现在只要使用 source 命令就可以省略注销或重启的过程，更加方便。

#### 通配

globbing：文件名通配(整体文件名匹配，而非部分)

```shell
*：匹配任意长度的任意字符
	pa*, *pa*, *pa, *p*a*
		pa, paa, passwd
?：匹配任意单个字符
	pa?, ??pa, p?a, p?a?
		pa, paa, passwd
[]：匹配指定范围内的任意单个字符
	有几种特殊格式：
		[a-z], [A-Z], [0-9], [a-z0-9]
		[[:upper:]]：所有大写字母
		[[:lower:]]：所有小写字母
		[[:alpha:]]：所有字母
		[[:digit:]]：所有数字
		[[:alnum:]]：所有的字母和数字
		[[:space:]]：所有空白字符
		[[:punct:]]：所有标点符号

		pa[0-9][0-9], 2[0-9][0-9]
[^]：匹配指定范围外的任意单个字符
	[^[:upper:]]
	[^0-9]
	[^[:alnum:]]
```

练习题：

```shell
练习1：显示/var目录下所有以l开头，以一个小写字母结尾，且中间出现一位任意字符的文件或目录；
	ls -d /var/l?[[:lower:]]
	ls -d /var/l?[a-z]
练习2：显示/etc目录下，以任意一位数字开头，且以非数字结尾的文件或目录；
	ls -d /etc/[0-9]*[^0-9]
	ls -d /etc/[[:digit:]]*[^[:digit:]]
练习3：显示/etc目录下，以非字母开头，后面跟一个字母及其它任意长度任意字符的文件或目录；
	ls -d /etc/[^a-z][a-z]*
	ls  [^[:alpha:]][[:alpha:]]*
练习4：复制/etc目录下，所有以m开头，以非数字结尾的文件或目录至/tmp/magedu.com目录；
	cp -r /etc/m*[^0-9] /tmp/magedu.com/
	
练习5：复制/usr/share/man目录下，所有以man开头，后跟一个数字结尾的文件或目录至/tmp/man/目录下；
	cp -r /usr/share/man/man[0-9] /tmp/man/

练习6：复制/etc目录下，所有以.conf结尾，且以m,n,r,p开头的文件或目录至/tmp/conf.d/目录下；
	cp -r /etc/[mnrp]*.conf /tmp/conf.d/
```



#### 补充

**登录信息**

我们在登录 tty1~tty6 这 6 个本地终端时，会有几行的欢迎界面。这些欢迎信息是保存在哪里的？可以修改吗？当然可以修改，这些欢迎信息保存在 /etc/issue 文件中，我们查看一下这个文件：

```shell
[hyp@localhost ~]$ cat /etc/issue
\S
Kernel \r on an \m
```

系统在每次登录时，会依赖这个文件的配置显示欢迎界面。在 /etc/issue 文件中允许使用转义符调用相应信息，其支持的转义符可以通过 man agetty 命令查询。

| 转义符 | 作 用                                |
| ------ | ------------------------------------ |
| \d     | 显示当前系统日期                     |
| \s     | 显示操作系统名称                     |
| \1     | 显示登录的终端号，这个转义符比较常用 |
| \m     | 显示硬件体系结构，如i386、i686等     |
| \n     | 显示主机名                           |
| \o     | 显示域名                             |
| \r     | 显示内核版本                         |
| \t     | 显示当前系统时间                     |
| \u     | 显示当前登录用户的序列号             |

配置 /etc/issue 文件会在本地终端登录时显示欢迎信息，如果远程登录（如 ssh 远程登录，或 Telnet 远程登录）需要显示欢迎信息，则需要配置 /etc/issue.net 文件。使用这个文件时有两点需要注意：

- 在 /etc/issue 文件中支持的转义符在 /etc/issue.net 文件中不能使用。
- ssh 远程登录是否显示 /etc/issue.net 文件中的欢迎信息，是由 ssh 的配置文件决定的。

如果我们需要 ssh 远程登录可以査看 /etc/issue.net 文件中的欢迎信息，那么首先需要修改 ssh 的配置文件 /etc/ssh/sshd_config，加入如下内容：

```shell
[root@localhost ~]# cat /etc/ssh/sshd_config ...省略部分输出...
\# no default banner path
\#Banner none
Banner /etc/issue.net
…省略部分输出…
```

这样，在 ssh 远程登录时，也可以显示欢迎信息，只是不能再识别"\d"和"\l"等信息了。

/etc/motd 文件中也是有欢迎信息的，这个文件和 /etc/issue 及 /etc/issue.net 文件的区别是：/etc/issue 及 /etc/issue.net 文件是在用户登录之前显示欢迎信息的；而 /etc/motd 文件是在用户输入用户名和密码，正确登录之后显示欢迎信息的。/etc/motd 文件中的欢迎信息，不论是本地登录，还是远程登录，都可以显示。

#### echo 显示内容颜色

echo 显示内容颜色，需要使用 -e 参数

-e :打开反斜杠转义 (默认不打开) ,可以转义 “\n, \t” 等

-n:在最后不自动换行

```
str="pingxin"
echo -e "\033[字背景颜色;文字颜色m ${str} \033[0m"
```

注：文字颜色后面有个m 

```
#字体颜色：30m-37m 黑、红、绿、黄、蓝、紫、青、白
str="pingxin"
echo -e "\033[30m ${str}\033[0m"      ## 黑色字体
echo -e "\033[31m ${str}\033[0m"      ## 红色
echo -e "\033[32m ${str}\033[0m"      ## 绿色
echo -e "\033[33m ${str}\033[0m"      ## 黄色
echo -e "\033[34m ${str}\033[0m"      ## 蓝色
echo -e "\033[35m ${str}\033[0m"      ## 紫色
echo -e "\033[36m ${str}\033[0m"      ## 青色
echo -e "\033[37m ${str}\033[0m"      ## 白色

#背景颜色：40-47 黑、红、绿、黄、蓝、紫、青、白
str="kimbo zhang"
echo -e "\033[41;37m ${str} \033[0m"     ## 红色背景色，白色字体
echo -e "\033[41;33m ${str} \033[0m"     ## 红底黄字
echo -e "\033[1;41;33m ${str} \033[0m"   ## 红底黄字 高亮加粗显示
echo -e "\033[5;41;33m ${str} \033[0m"   ## 红底黄字 字体闪烁显示
echo -e "\033[47;30m ${str} \033[0m"     ## 白底黑字
echo -e "\033[40;37m ${str} \033[0m"     ## 黑底白字
#其他参数说明
#　\033[1;m 设置高亮加粗
#　　\033[4;m 下划线
#　　\033[5;m 闪烁
```

示例：定义函数，用于日志写入等

```
#!/bin/bash
## 写日志
## 参数1：字符串
## 参数2：颜色 (红色：失败报错，绿色：成功，黄色：警告)

function func_write_log()
{
    var_str=$1
    var_color=$2
    var_curr_timestamp=`date "+%Y-%m-%d %H:%M:%S"`

    ## 判断参数1 是否是空字符串
    if [ "x${var_str}" == "x" ];then
        var_str=""
    else
        var_str="${var_curr_timestamp} ${var_str}"
    fi

    ## 判断颜色
    if [ "${var_color}" == "green" ];then
        var_str="\n\033[32m${var_str}\033[0m"
    elif [ "${var_color}" == "yellow" ];then
        var_str="\033[33m${var_str}\033[0m"
    elif [ "${var_color}" == "red" ];then
        var_str="\033[1;41;33m${var_str}\033[0m"
    else
        var_str="\033[37m${var_str}\033[0m"
    fi

    ## 打印输出
    echo -e "${var_str}"
    #echo -e "${var_str}" >> ${var_path}/test_${var_curr_timestamp}.log 2>&1  #写入日志文件
}

## 函数调用
func_write_log "pingxin" "red"
```

#### printf格式化输出

```
printf [format] [文本1] [文本2] ..
```

**常用格式替换符**

| %s    | 字符串                                |
| ----- | ------------------------------------- |
| %f    | 浮点格式                              |
| %c    | ASCII字符，即显示对应参数的第一个字符 |
| %d,%i | 十进制整数                            |
| %o    | 八进制值                              |
| %u    | 不带正负号的十进制值                  |
| %x    | 十六进制值（a-f）                     |
| %X    | 十六进制值（A-F）                     |
| %%    | 表示%本身                             |

**常用转义字符**

| \a   | 警告字符，通常为ASCII的BEL字符 |
| ---- | ------------------------------ |
| \b   | 后退                           |
| \f   | 换页                           |
| \n   | 换行                           |
| \r   | 回车                           |
| \t   | 水平制表符                     |
| \v   | 垂直制表符                     |
| \\   | 表示\本身                      |

**使用案例**

```
[root@C ~]# printf "%s\n" 1 2 3 4
1
2
3
4
 
[root@C ~]# printf "%f\n" 1 2 3 4
1.000000
2.000000
3.000000
4.000000
 
[root@C ~]# printf "%.2f\n" 1 2 3 4
1.00
2.00
3.00
4.00
 
[root@C ~]# printf " (%s) " 1 2 3 4 ; echo ""
 (1)  (2)  (3)  (4)
 
[root@C ~]# printf "%s %s\n" 1 2 3 4
1 2
3 4
 
[root@C ~]# printf "%s %s %s\n" 1 2 3 4
1 2 3
4
 
#“-” 表示左对齐，“10 10 4” 表示占的字符位数，不够空格补全
[root@C ~]# printf "%-10s %-10s %-4s \n" 姓名 性别 年龄 皮特 男 18 南瓜 男 18  
姓名     性别     年龄
皮特     男        18  
南瓜     男        18  
```

### 文本编辑器

文本编辑器是对文本进行创建修改的程序，常用的编辑器有：vi、sed、nano、awk等。其中grep、sed和awk被称作文本三剑客，Grep（grep,egrep,fgrep): 文本过滤工具，  Sed： 文本编辑工具， Awk: 文本报告生成器。

vi现在是用vim来代替，如果安装了vim，可以看到/bin/vi其实是/bin/vim的软链接；

sed是一种流编编器，它是文本处理中非常中的工具，能够完美的配合正则表达式便用，功能能不同凡响。

nano是一个字符终端的文本编辑器，有点像DOS下的editor程序。它比vi/vim要简单得多，比较适合Linux初学者使用，某些Linux发行版的默认编辑器就是nano；

awk是一种编程语言，用于对文本和数据进行处理， 具有强大的**文本格式化**能力。

#### nano

nano命令可以打开指定文件进行编辑，默认情况下它会自动断行，即在一行中输入过长的内容时自动拆分成几行，但用这种方式来处理某些文件可能会带来问题，比如Linux系统的配置文件，自动断行就会使本来只能写在一行上的内容折断成多行了，有可能造成系统不灵了。因此，如果你想避免这种情况出现，就加上-w选项吧。

安装

```shell
#CentOS体系
yum -y install nano
#Debian/Ubuntu体系
apt-get install -y nano
```

语法：

```
nano [option] [[+行,列]/PATH/TO/FILE]...
```

命令：

```
Alt+6  #复制一整行
Ctrl+K  #剪贴一整行
Ctrl+k #删除一整行
Ctrl+U  #粘贴
Ctrl+Y  #上一页
Ctrl+V  #下一页
Ctrl+W，然后输入你要搜索的关键字，回车确定。这将会定位到第一个匹配的文本，接着可以用
Alt+W，来定位到下一个匹配的文本。
Ctrl+O  #保留
Ctrl+X  #退出，如其你修正了文献，会要你输入保留文献名，径直确认便可。
注意:在nano帮助文档里，Ctrl-键被表示为一个脱字符（^），因此Ctrl+W被写成了^W，等等。Alt-键被表示为一个M（从"Meta"而来），因此Alt+W被写成了M-W。
```

#### Vim

vim相对nano来说，比较复杂，功能也比较强大，是vi编辑器的加强。代码补完、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。简单的来说， vi 是老式的字处理器，不过功能已经很齐全了，但是还是有可以进步的地方。 vim 则可以说是程序开发者的一项很好用的工具。

vim总共有三种模式：命令模式（Command mode），输入模式（Insert mode）和底线命令模式（Last line mode）。

##### 命令模式：

用户刚刚启动 vi/vim，便进入了命令模式。

此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。比如我们此时按下i，并不会输入一个字符，i被当作了一个命令。

以下是常用的几个命令：

- **i** 切换到输入模式，以输入字符。
- **x** 删除当前光标所在处的字符。
- **:** 切换到底线命令模式，以在最底一行输入命令。

若想要编辑文本：启动Vim，进入了命令模式，按下i，切换到输入模式。

命令模式只有一些最基本的命令，因此仍要依靠底线命令模式输入更多命令。

##### 输入模式

在命令模式下按下i就进入了输入模式。

在输入模式中，可以使用以下按键：

- **字符按键以及Shift组合**，输入字符
- **ENTER**，回车键，换行
- **BACK SPACE**，退格键，删除光标前一个字符
- **DEL**，删除键，删除光标后一个字符
- **方向键**，在文本中移动光标
- **HOME**/**END**，移动光标到行首/行尾
- **Page Up**/**Page Down**，上/下翻页
- **Insert**，切换光标为输入/替换模式，光标将变成竖线/下划线
- **ESC**，退出输入模式，切换到命令模式

##### 底线命令模式

在命令模式下按下:（英文冒号）就进入了底线命令模式。

底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。

在底线命令模式中，基本的命令有（已经省略了冒号）：

- q 退出程序
- w 保存文件

按ESC键可随时退出底线命令模式。

简单的说，我们可以将这三个模式想成底下的图标来表示：

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1arwew04gj30lv0er3yt.jpg)

直接输入 **vi 文件名** 就能够进入 vi 的一般模式了。请注意，记得 vi 后面一定要加文件名，不管该文件存在与否！

##### 按键功能

vim的键盘图如下：

![](https://ws3.sinaimg.cn/large/005BYqpggy1g1aql2k6hxj30sc0j9tat.jpg)



第一部份：一般模式可用的光标移动、复制粘贴、搜索替换等

| 移动光标的方法                                               |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| h 或 向左箭头键(←)                                           | 光标向左移动一个字符                                         |
| j 或 向下箭头键(↓)                                           | 光标向下移动一个字符                                         |
| k 或 向上箭头键(↑)                                           | 光标向上移动一个字符                                         |
| l 或 向右箭头键(→)                                           | 光标向右移动一个字符                                         |
| 如果你将右手放在键盘上的话，你会发现 hjkl 是排列在一起的，因此可以使用这四个按钮来移动光标。 如果想要进行多次移动的话，例如向下移动 30 行，可以使用 "30j" 或 "30↓" 的组合按键， 亦即加上想要进行的次数(数字)后，按下动作即可！ |                                                              |
| [Ctrl] + [f]                                                 | 屏幕『向下』移动一页，相当于 [Page Down]按键 (常用)          |
| [Ctrl] + [b]                                                 | 屏幕『向上』移动一页，相当于 [Page Up] 按键 (常用)           |
| [Ctrl] + [d]                                                 | 屏幕『向下』移动半页                                         |
| [Ctrl] + [u]                                                 | 屏幕『向上』移动半页                                         |
| +                                                            | 光标移动到非空格符的下一行                                   |
| -                                                            | 光标移动到非空格符的上一行                                   |
| n<space\>                                                    | 那个 n 表示『数字』，例如 20 。按下数字后再按空格键，光标会向右移动这一行的 n 个字符。例如 20<space> 则光标会向后面移动 20 个字符距离。 |
| 0 或功能键[Home]                                             | 这是数字『 0 』：移动到这一行的最前面字符处 (常用)           |
| $ 或功能键[End]                                              | 移动到这一行的最后面字符处(常用)                             |
| H                                                            | 光标移动到这个屏幕的最上方那一行的第一个字符                 |
| M                                                            | 光标移动到这个屏幕的中央那一行的第一个字符                   |
| L                                                            | 光标移动到这个屏幕的最下方那一行的第一个字符                 |
| G                                                            | 移动到这个档案的最后一行(常用)                               |
| nG                                                           | n 为数字。移动到这个档案的第 n 行。例如 20G 则会移动到这个档案的第 20 行(可配合 :set nu) |
| gg                                                           | 移动到这个档案的第一行，相当于 1G 啊！ (常用)                |
| n<Enter\>                                                    | n 为数字。光标向下移动 n 行(常用)                            |
| 搜索替换                                                     |                                                              |
| /word                                                        | 向光标之下寻找一个名称为 word 的字符串。例如要在档案内搜寻 vbird 这个字符串，就输入 /vbird 即可！ (常用) |
| ?word                                                        | 向光标之上寻找一个字符串名称为 word 的字符串。               |
| n                                                            | 这个 n 是英文按键。代表重复前一个搜寻的动作。举例来说， 如果刚刚我们执行 /vbird 去向下搜寻 vbird 这个字符串，则按下 n 后，会向下继续搜寻下一个名称为 vbird 的字符串。如果是执行 ?vbird 的话，那么按下 n 则会向上继续搜寻名称为 vbird 的字符串！ |
| N                                                            | 这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。 例如 /vbird 后，按下 N 则表示『向上』搜寻 vbird 。 |
| 使用 /word 配合 n 及 N 是非常有帮助的！可以让你重复的找到一些你搜寻的关键词！ |                                                              |
| :n1,n2s/word1/word2/g                                        | n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代为 word2 ！举例来说，在 100 到 200 行之间搜寻 vbird 并取代为 VBIRD 则： 『:100,200s/vbird/VBIRD/g』。(常用) |
| :1,$s/word1/word2/g或 :%s/word1/word2/g                      | 从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 ！(常用) |
| :1,$s/word1/word2/gc或 :%s/word1/word2/gc                    | 从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 ！且在取代前显示提示字符给用户确认 (confirm) 是否需要取代！(常用) |
| 删除、复制与贴上                                             |                                                              |
| x, X                                                         | 在一行字当中，x 为向后删除一个字符 (相当于 [del] 按键)， X 为向前删除一个字符(相当于 [backspace] 亦即是退格键) (常用) |
| nx                                                           | n 为数字，连续向后删除 n 个字符。举例来说，我要连续删除 10 个字符， 『10x』。 |
| dd                                                           | 删除游标所在的那一整行(常用)                                 |
| ndd                                                          | n 为数字。删除光标所在的向下 n 行，例如 20dd 则是删除 20 行 (常用) |
| d1G                                                          | 删除光标所在到第一行的所有数据                               |
| dG                                                           | 删除光标所在到最后一行的所有数据                             |
| d$                                                           | 删除游标所在处，到该行的最后一个字符                         |
| d0                                                           | 那个是数字的 0 ，删除游标所在处，到该行的最前面一个字符      |
| yy                                                           | 复制游标所在的那一行(常用)                                   |
| nyy                                                          | n 为数字。复制光标所在的向下 n 行，例如 20yy 则是复制 20 行(常用) |
| y1G                                                          | 复制游标所在行到第一行的所有数据                             |
| yG                                                           | 复制游标所在行到最后一行的所有数据                           |
| y0                                                           | 复制光标所在的那个字符到该行行首的所有数据                   |
| y$                                                           | 复制光标所在的那个字符到该行行尾的所有数据                   |
| p, P                                                         | p 为将已复制的数据在光标下一行贴上，P 则为贴在游标上一行！ 举例来说，我目前光标在第 20 行，且已经复制了 10 行数据。则按下 p 后， 那 10 行数据会贴在原本的 20 行之后，亦即由 21 行开始贴。但如果是按下 P 呢？ 那么原本的第 20 行会被推到变成 30 行。 (常用) |
| J                                                            | 将光标所在行与下一行的数据结合成同一行                       |
| c                                                            | 重复删除多个数据，例如向下删除 10 行，[ 10cj ]               |
| u                                                            | 复原前一个动作。(常用)                                       |
| [Ctrl]+r                                                     | 重做上一个动作。(常用)                                       |
| 这个 u 与 [Ctrl]+r 是很常用的指令！一个是复原，另一个则是重做一次～ 利用这两个功能按键，你的编辑，嘿嘿！很快乐的啦！ |                                                              |
| .                                                            | 不要怀疑！这就是小数点！意思是重复前一个动作的意思。 如果你想要重复删除、重复贴上等等动作，按下小数点『.』就好了！ (常用) |

##### 第二部份：一般模式切换到编辑模式的可用的按钮说明

| 进入输入或取代的编辑模式                                     |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| i, I                                                         | 进入输入模式(Insert mode)： i 为『从目前光标所在处输入』， I 为『在目前所在行的第一个非空格符处开始输入』。 (常用) |
| a, A                                                         | 进入输入模式(Insert mode)： a 为『从目前光标所在的下一个字符处开始输入』， A 为『从光标所在行的最后一个字符处开始输入』。(常用) |
| o, O                                                         | 进入输入模式(Insert mode)： 这是英文字母 o 的大小写。o 为『在目前光标所在的下一行处输入新的一行』； O 为在目前光标所在处的上一行输入新的一行！(常用) |
| r, R                                                         | 进入取代模式(Replace mode)： r 只会取代光标所在的那一个字符一次；R会一直取代光标所在的文字，直到按下 ESC 为止；(常用) |
| 上面这些按键中，在 vi 画面的左下角处会出现『--INSERT--』或『--REPLACE--』的字样。 由名称就知道该动作了吧！！特别注意的是，我们上面也提过了，你想要在档案里面输入字符时， 一定要在左下角处看到 INSERT 或 REPLACE 才能输入喔！ |                                                              |
| [Esc]                                                        | 退出编辑模式，回到一般模式中(常用)                           |

##### 第三部份：一般模式切换到指令行模式的可用的按钮说明

| 指令行的储存、离开等指令                                     |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| :w                                                           | 将编辑的数据写入硬盘档案中(常用)                             |
| :w!                                                          | 若文件属性为『只读』时，强制写入该档案。不过，到底能不能写入， 还是跟你对该档案的档案权限有关啊！ |
| :q                                                           | 离开 vi (常用)                                               |
| :q!                                                          | 若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。    |
| 注意一下啊，那个惊叹号 (!) 在 vi 当中，常常具有『强制』的意思～ |                                                              |
| :wq                                                          | 储存后离开，若为 :wq! 则为强制储存后离开 (常用)              |
| ZZ                                                           | 这是大写的 Z 喔！若档案没有更动，则不储存离开，若档案已经被更动过，则储存后离开！ |
| :w [filename]                                                | 将编辑的数据储存成另一个档案（类似另存新档）                 |
| :r [filename]                                                | 在编辑的数据中，读入另一个档案的数据。亦即将 『filename』 这个档案内容加到游标所在行后面 |
| :n1,n2 w [filename]                                          | 将 n1 到 n2 的内容储存成 filename 这个档案。                 |
| :! command                                                   | 暂时离开 vi 到指令行模式下执行 command 的显示结果！例如 『:! ls /home』即可在 vi 当中察看 /home 底下以 ls 输出的档案信息！ |
| vim 环境的变更                                               |                                                              |
| :set nu                                                      | 显示行号，设定之后，会在每一行的前缀显示该行的行号           |
| :set nonu                                                    | 与 set nu 相反，为取消行号！                                 |

特别注意，在 vi/vim 中，数字是很有意义的！数字通常代表重复做几次的意思！ 也有可能是代表去到第几个什么什么的意思。

举例来说，要删除 50 行，则是用 『50dd』 对吧！ 数字加在动作之前，如我要向下移动 20 行呢？那就是『20j』或者是『20↓』即可。

##### 配置

在目录/etc、下面，有个名为vimrc的文件，这就是系统中公共的vim配置文件，对所有用户都开放。而在每个用户的主目录下，都可以自己建立私有的配置文件，命名为：”.vimrc”。

可以根据自己需求选择配置文件位置，每个配置一行。

```
设置语法高亮：syntax on/off
  "显示行号
    set nu
    set number
  "更改缩进大小
    set tabstop=2
  "颜色主题
    color koehler
  "显示标尺
    set ruler
  "去空行  
    nnoremap <F2> :g/^\s*$/d<CR> 
  "激活鼠标
    set mouse=a
  "新建标签  
    map <M-F2> :tabnew<CR>  
  "列出当前目录文件  
    map <F3> :tabnew .<CR>  
  "打开树状文件目录  
    map <C-F3> \be  
  "C，C++ 按F5编译运行（需要安装gcc、gcc-c++）
    map <F5> :call CompileRunGcc()<CR>
    func! CompileRunGcc()
        exec "w"
        if &filetype == 'c'
            exec "!g++ % -o %<"
            exec "! ./%<"
        elseif &filetype == 'cpp'
            exec "!g++ % -o %<"
            exec "! ./%<"
        elseif &filetype == 'java' 
            exec "!javac %" 
            exec "!java %<"
        elseif &filetype == 'sh'
            :!./%
        endif
    endfunc
  "C,C++的调试
    map <F8> :call Rungdb()<CR>
    func! Rungdb()
        exec "w"
        exec "!g++ % -g -o %<"
        exec "!gdb ./%<"
    endfunc
  "自动缩进
    set autoindent
    set cindent
  "统一缩进为4
    set softtabstop=2
    set shiftwidth=2
```

如果自己不想一个一个配置可以使用一键配置，直接在命令行执行下面命令，可以配置一个开发环境： 

```shell
#!/bin/bash 
git clone --recursive https://github.com/Leptune/vim-for-coding.git ~/vim-for-coding
cd ~ 
mv .vim .vimbak &> /dev/null 
mv .vimrc .vimrcbak &> /dev/null 
mv vim-for-coding .vim 
ln -s .vim/vimrc .vimrc
```

#### Linux命令发送Http GET/POST请求

**curl命令模拟Get请求：**

1、使用curl命令：

```shell
`curl ``"http://www.baidu.com"` `如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地``curl -i ``"http://www.baidu.com"` `显示全部信息``curl -I ``"http://www.baidu.com"` `只显示头部信息``curl -v ``"http://www.baidu.com"`  `显示get请求全过程解析`
```

2、使用wget命令：

```
`wget ``"http://www.baidu.com"`
```

**curl命令模拟Get请求携带参数：**

```
`curl -v http:``//127.0.0.1:80/xcloud/test?version=1&client_version=1.1.0&seq=1001&host=aaa.com`
```

上述命令在linux系统，get请求携带的参数只到version=1，”&”符号在linux系统中为后台运行的操作符，此处需要使用反斜杠”\”转义，即：

```
`curl -v http:``//127.0.0.1:80/xcloud/test?version=1\&client_version=1.1.0\&seq=1001\&host=aaa.com`
```

或者

```
`curl -v ``"http://127.0.0.1:80/xcloud/test?version=1&client_version=1.1.0&seq=1001&host=aaa.com"`
```

**Post请求**

1、使用curl命令，通过-d参数，把访问参数放在里面，如果没有参数，则不需要-d，

```
`curl -d ``"username=user1&password=123"` `"www.test.com/login"`
```

2、使用wget命令

```
`wget –post-data ``'username=user1&password=123'` `http:``//www.baidu.com`
```

3、发送格式化json请求

```
`curl -i -k -H ``"Content-type: application/json"` `-X POST -d ``'{"version":"6.6.0", "from":"mu", "product_version":"1.1.1.0"}'` `https:``//10.10.10.10:80/test`
```

**curl和wget区别**

curl模拟的访问请求一般直接在控制台显示，而wget则把结果保存为一个文件。如果结果内容比较少，需要直接看到结果可以考虑使用curl进行模拟请求，如果返回结果比较多，则可考虑wget进行模拟请求。

#### 给文件追加多行内容

**追加到文件结尾**

```shell
#1.
$ cat >> lb.txt<<EOF
> hellow
> world
>EOF

#2. 
$ echo "hellow
world" >> lb.txt

#3. 
$ echo -e "hellow\nworld" >> lb.txt
#"-e"表示激活转义字符，"\n"表示换行，"\t"表示Tab键

#4. 
$ cat >> lb.txt
hellow
world
#使用Ctrl+c或Ctrl+d结束输入

```

**指定的行前/后插入指定内容**

```shell
# 原文件内容
$ cat lb.txt
hellow
world
sina
baidu

#在"world"行的下面插入一行内容
$ sed '/world/a\taobao' lb.txt
hellow
world
taobao
sina
baidu

#在"world"行的下面插入多行内容

$ sed '/world/a\taobao\njingdong\naliyun' lb.txt
hellow
world
taobao
jingdong
aliyun
sina
baidu
#注释："\n"表示换行


#在"world"行的上面插入一行内容

$ sed '/world/i\taobao' lb.txt
hellow
taobao
world
sina
baidu
#注释：把参数"a"换成参数"i"


#在"world"行的下面插入多行内容

$ sed '/world/i\taobao\njingdong\naliyun' lb.txt
hellow
taobao
jingdong
aliyun
world
sina
baidu
```

**如果文件中有多行匹配，结果会在匹配的行都加上内容**

```shell
#原文件内容
$ cat lb.txt
hellow
world
    worldd
sina
baidu

$ sed '/world/a\taobao' lb.txt
hellow
world
taobao
        worldd
taobao
sina
baidu

$ sed '/^world/a\taobao' lb.txt
hellow
world
taobao
        worldd
sina
baidu
#注释：使用正则表达式匹配，匹配以world开头的行

$ sed '/\bworld\b/a\taobao' lb.txt
hellow
world
taobao
        worldd
sina
baidu
#注释：使用正则表达式匹配，匹配单词边界


$sed '2a\taobao' lb.txt
hellow
world
taobao
        worldd
sina
baidu
#注释：根据文件内容行号

```

#### 参考

1. [curl使用指南](https://www.jianshu.com/p/fc0eb6c60816)

