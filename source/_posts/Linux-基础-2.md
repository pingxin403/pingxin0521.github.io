---
title: Linux-基础命令
date: 2019-03-12 16:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

Linux上支持的CLI主要有：[bash](http://www.gnu.org/software/bash/)、[zsh](https://ohmyz.sh/)、sh、csh、tcsh、ksh，最普遍的是bash。Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。Shell Script大致都类同，当您学会一种Shell以后，其它的Shell会很快就上手，大多数的时候，一个Shell Script通常可以在很多种Shell上使用。普遍认为，命令行在处理重复和复杂任务时比图形界面更快更方便，这也是很多图形操作系统保留命令行工具的原因。

学习Linux的过程是从熟悉其常用命令开始，然后循序渐进，慢慢掌握其使用、配置和优化，生命本是不断探索的过程，现在让我们来学习吧。

<!--more-->

### 简介

#### 终端

终端设备（terminal）是指经由通信设施向计算机输入程序和数据或接收计算机输出处理结果的*设备*。Linux支持多任务、多用户，以及多个不同种类的终端。

- 物理终端，也被称为控制台（console），是指直接连接在主机上的显示器、键盘鼠标统称。在实际机架式服务器部署中，一般是多台服务器共享一套终端，简称KVM（Keyboard键盘，video显示器，mouse鼠标）。

- 虚拟终端（tty）：附加在物理终端之上，用软件方式虚拟实现，CentOS默认启用6个虚拟终端，可以通过快捷键来切换，切换方式:Ctrl-Alt-F[1--6], 对应的文件是/dev/tty#，图形终端Ctrl+Alt+F7。tty是teletypewriter（电传打字机）的简称。

- 伪终端(pty)：两种应用场景，第一在图形界面下打开的命令行接口，第二基于ssh协议或telnet协议等远程打开的命令行界面，是运维工程师用的最多的一种连接服务器的方式。pts(pseudo-terminal slave)是pty的实现方法。对应的文件是/dev/pts/#。

- 串行终端：串口输出，对应的文件是/dev/ttyS#。

系统正常启动，显示启动过程信息输出到物理终端，当物理终端被系统初始化后，称为虚拟终端（图形界面或 Ctrl+Alt+F[1-6]）打开图形界面模拟一个命令窗口就是伪终端，或者远程登入该系统，该终端也是伪终端

**区别当前系统是那种终端**

使用命令tty，表示当前终端对应的设备文件,(以下#表示数字)

1、结果显示：/dev/pts/#  表示伪终端

2、结果显示：/dev/tty#   表示虚拟终端

3、结果显示：/dev/console 表示物理终端（控制台）

4、结果显示：/dev/ttys#  表示串行终端

```shell
[hyp@localhost ~]$ tty
/dev/pts/0
```

#### 命令格式

```shell
COMMAND [-OPTIONS/--LONG OPTIONS] [ARGUMENTS]
```

**命令**

命令分为两类：由shell程序的自带的命令（内置命令、builtin）；独立的可执行程序文件，文件名即命令名（外部命令）。命令就是一个程序的一次运行，发起命令：请求内核将某个二进制程序运行为一个进程（[程序和进程的区别](https://blog.csdn.net/qq_36812792/article/details/80118923)）。命令本身是一个可执行的程序文件：二进制格式的文件，运行时有可能会调用共享库文件。

程序的组成部分：二进制程序文件、库文件、配置文件、帮助文件。其中二进制、库文件是可执行文件；库文件是不能独立执行，只能被调用时执行。配置文件、帮助文件是可被查看其内容的文件。

多数系统程序文件都存放在：/bin, /sbin, /usr/bin, /usr/sbin，/usr/local/bin, /usr/local/sbin目录下。普通命令：/bin, /usr/bin, /usr/local/bin；管理命令（只有管理员才能运行）：/sbin, /usr/sbin, /usr/local/sbin。

共享库：/lib, /lib64, /usr/lib, /usr/lib64, /usr/local/lib, /usr/local/lib64。32bits的库：/lib, /usr/lib, /usr/local/lib；64bits的库：/lib64, /usr/lib64, /usr/local/lib64。

但是并非所有的命令都有一个在某目录与之对应的可执行程序文件。

shell程序是独特的程序，负责解析用户提供的命令。从环境变量中包含的目录下查找可运行文件，环境变量使用`：`分隔路径，可以通过`echo $PATH`查看环境变量。

```shell
[hyp@localhost ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/hyp/.local/bin:/home/hyp/bin
```

使用`type COMMEND`查看命令分类，内部命令显示 shell相关信息，外部命令则显示$PATH路径或别名。

```shell
[hyp@localhost ~]$ type type
type 是 shell 内嵌
[hyp@localhost ~]$ type ls
ls 是 `ls --color=auto' 的别名
[hyp@localhost ~]$ type touch
touch 是 /usr/bin/touch
```

命令输入方式有两种 ：

1.直接键入命令（可能存在歧义，机器会按照一定的优先级判断命令的执行顺序）

2.敲入路径+命令（这种执行方式命令没有歧义）。单纯键入命令时由于有歧义，所以存在着命令执行的优先级问题

<u>命令（单命令）执行优先级：alias（别名）->shell（内建命令）>hash>PATH</u>

也就是说当一条命令执行时 ：

1.先去判断它是否是别名
2.判段命令是否是内部命令
3.看哈希表是否为空，若不为空，则去hash表中指定的路径查找

 4.若以上三步都不执行，则按照path路径挨个查找,PATH是从左到右查找。

> \command 和 'command' “command”则不使用别名，直接执行第二步。
>
> ; 命令连接符，前一条命令执行结束，再执行下一条命令
>   \ 命令换行符

**选项**

使用OPTIONS指定命令的运行特性，选项有两种表现形式：短选项、长选项。

短选项：`-C`, 例如`-l`, `-d`。有些命令的选项没有短选项，如果同一命令同时使用多个短选项，多数可合并：`-l -d = -ld`。

长选项：--word, 例如--help, --human-readable。长选项不能合并，有些选项可以带参数，此称为选项参数。

**参数**

命令的作用对象；命令对什么生效。不同的命令的参数；有些命令可同时带多个参数，多个之间以空白字符分隔。例如：`ls -ld /var /etc` 

### 获取帮助

linux上命令非常多，各种各种的命令之间用法不一致，不同版本的命令用法也不同，如何查看命令的使用方法就很重要了。

1. 别名。通过`alias`进行查看支持的别名。

   ```shell
   [hyp@localhost ~]$ alias
   alias egrep='egrep --color=auto'
   alias fgrep='fgrep --color=auto'
   alias grep='grep --color=auto'
   alias l.='ls -d .* --color=auto'
   alias ll='ls -l --color=auto'
   alias ls='ls --color=auto'
   alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
   ```

2. 内建命令。使用`help COMMAND`获取帮助。`help`也是内置命令，可以通过`help help`查看帮助，也可以直接使用`help`查看系统所支持的所有bash内建命令。

   ```shell
   [hyp@localhost ~]$ help help
   help: help [-dms] [模式 ...]
       显示内嵌命令的相关信息。
   
       显示内嵌命令的简略信息。如果指定了 PATTERN 模式，
       给出所有匹配 PATTERN 模式的命令的详细帮助，否则打
       印一个帮助主题列表
   
       选项：
         -d        输出每个主题的简短描述
         -m        以伪 man 手册的格式显示使用方法
         -s        为每一个匹配 PATTERN 模式的主题仅显示一个用法
           简介
   
       参数：
         PATTERN   Pattern 模式指定一个帮助主题
   
       退出状态：
       返回成功，除非 PATTERN 模式没有找到或者使用了无效选项。
   ```

3. 外部命令。

   - 命令自带的帮助命令。一般使用`COMMAND --help`查看命令的简短使用帮助，一般命令还支持`-h`查看。

   - `man [CHAPTER] COMMAND`

     使用`/usr/share/man`目录下的帮助文档，文档一般包括这几个内容：NAME(功能性说明)、SYNOPSIS(语法格式)、DESCRIPTION(描述)、OPTIONS(选项)、EXAMPLES(使用示例)、AUTHOR(作者)、BUGS（ 报告程序bug的方式）、SEE ALSO（参考）。命令使用语法：[]是可选内容；<>是必须提供的内容；a|b|c是多选一；...是同类内容可出现多个。

     手册根据功能进行分章节，一般通过`whatis COMMAND`查看所属章节。

     章节分有：

     1：用户命令；
     2：系统调用；
     3：C库调用；
     4：设备文件及特殊文件；
     5：文件格式；（配置文件格式）
     6：游戏使用帮助；
     7：杂项；
     8：管理工具及守护进行；

     **man命令打开手册以后的操作方法**

     ```shell
     翻屏：
     空格键：向文件尾翻一屏；
     b: 向文件首部翻一屏；
     Ctrl+d：向文件尾部翻半屏；
     Ctrl+u：向文件首部翻半屏；
     回车键：向文件尾部翻一行；
     k: 向文件首部翻一行；
     G：跳转至最后一行；
     #G: 跳转至指定行；
     1G：跳转至文件首部；
     
     文本搜索：
     /keyword：从文件首部向文件尾部依次查找；不区分字符大小写；
     ?keyword：从文件尾部向文件首部依次查找；	
     n: 与查找命令方向相同；
     N: 与查找命令方向相反；
     
     退出：
     q: quit
     ```

     选项：

     ```
     -M /PATH/TO/SOMEDIR：到指定目录下查找命令手册并打开之；
     ```

     ​	

   - `info COMMAND`

     获取命令的在线文档,使用查看帮助。

   - 软件自带的帮助文档

     帮助文档位置：`/usr/share/doc/APP-VERSION`。README：程序的相关的信息；INSTALL: 安装帮助；CHANGES：版本迭代时的改动信息；

   - 主流发行版官方文档。例如：http://www.redhat.com/doc

   - 程序官方的文档。

   - 搜索引擎。搜索用法

### 目录操作

Linux文件系统规定：
1、文件名名称严格区分字符大小写；
2、文件可以使用除/以外任意字符；
3、文件名长度不能超过255字符；
4、以.开头的文件为隐藏文件；
.: 当前目录；..: 当前目录的上一级目录；例如：

```shell
/etc/sysconfig/
.: sysconfig
..: /etc
```

默认称当前目录为工作目录（working directory），用户家目录（home）为用户目录，使用`~`表示。

- **pwd**

pwd: printing working directory，显示工作目录。

选项：

-L：--logical，显示当前的路径，有连接文件时，直接显示连接文件的路径，(不加参数时默认此方式)。 

-p：--physical，显示当前的路径，有连接文件时，不使用连接路径，直接显示连接文件所指向的文件。 当包含多层连接文件时，显示连接文件最终指向的文件

```shell
[hyp@localhost bin]$ pwd
/bin
[hyp@localhost bin]$ pwd -L
/bin
[hyp@localhost bin]$ pwd -P
/usr/bin
[hyp@localhost bin]$ ll `pwd` #先执行pwd命令，ll查看文件详细属性
lrwxrwxrwx. 1 root root 7 3月  10 17:30 /bin -> usr/bin
```

- **ls**

ls:list,显示指定目录下的内容。用法：`ls [OPTION]... [FILE]...`。

选项：

-a: 显示所有文件，包括隐藏文件；
-A：显示除.和..之外的所有文件；
-l: --long, 长格式列表，即显示文件的详细属性信息；
	-rw-r--r--. 1 root   root     8957 10月 14 19:34 boot.log
	-：文件类型，-, d, b, c, l, s, p
	rw-r--r--：权限有rwx，读、写、执行。
		rw-：文件属主的权限；
		r--：文件属组的权限；
		r--：其它用户（非属主、属组）的权限；
	1：数字表示文件被硬链接的次数；
	root：文件的属主；
	root：文件的属组；
	8957：数字表示文件的大小，单位是字节；
	10月 14 19:34 ：文件最近一次被修改的时间；
	boot.log：文件名
-h, --human-readable：对文件大小单位换算；换算后结果可能会不是精确值；
-d：查看目录自身而非其内部的文件列表；
-r: reverse, 逆序显示；
-R: recursive，递归显示；

```shell
[hyp@localhost ~]$ ls
1.jpg  tuc.html  桌面
[hyp@localhost ~]$ ls -a
.              .bash_profile  .esd_auth      .mozilla            tuc.html
..             .bashrc        .gitconfig     .npm                .viminfo
1.jpg          .cache         .ICEauthority  .pki                桌面
.bash_history  .config        .lesshst       .speech-dispatcher
.bash_logout   .dbus          .local         .ssh
[hyp@localhost ~]$ ls -A
1.jpg          .bashrc  .esd_auth      .local    .speech-dispatcher  桌面
.bash_history  .cache   .gitconfig     .mozilla  .ssh
.bash_logout   .config  .ICEauthority  .npm      tuc.html
.bash_profile  .dbus    .lesshst       .pki      .viminfo
[hyp@localhost ~]$ ls -l
总用量 108
-rw-r--r--. 1 hyp hyp 94634 3月  13 09:07 1.jpg
-rw-rw-r--. 1 hyp hyp  8850 3月  13 09:01 tuc.html
drwxr-xr-x. 2 hyp hyp    20 3月  11 10:44 桌面
[hyp@localhost ~]$ ls -lh
总用量 108K
-rw-r--r--. 1 hyp hyp  93K 3月  13 09:07 1.jpg
-rw-rw-r--. 1 hyp hyp 8.7K 3月  13 09:01 tuc.html
drwxr-xr-x. 2 hyp hyp   20 3月  11 10:44 桌面
[hyp@localhost ~]$ ls -ld /etc
drwxr-xr-x. 120 root root 8192 3月  17 12:25 /etc
[hyp@localhost ~]$ ls -r
桌面  tuc.html  1.jpg
[hyp@localhost ~]$ ls -R
.:
1.jpg  tuc.html  桌面

./桌面:
Typora
```

常用别名：

```shell
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
```

- **cd**

cd：change directory，切换工作目录。用法：`cd [/PATH/TO/SOMEDIR]`。相关的环境变量：$PWD：当前工作目录；$OLDPWD：上一次的工作目录，使用`-`表示。

cd: 切换回家目录；
	注意：bash中, ~表示家目录；
cd ~：切换回自己的家目录
cd ~USERNAME：切换至指定用户的家目录；
cd -：在上一次所在目录与当前目录之间来回切换；

cd ..:切换到上一级目录。

```shell
[hyp@localhost bin]$ pwd
/bin
[hyp@localhost bin]$ cd
[hyp@localhost ~]$ pwd
/home/hyp
[hyp@localhost ~]$ cd -
/bin
[hyp@localhost bin]$ pwd
/bin
[hyp@localhost bin]$ cd ..
[hyp@localhost /]$ pwd
/
```

- **mkdir**

mkdir : make directory,创建目录。用法：`mkdir [options] [PATH/TO/SOMEPATH]`,

选项：

-m --mode=模式，设定权限<模式> (类似 chmod)，而不是 rwxrwxrwx 减 umask

-p --parents 递归创建目录

-v, --verbose 每次创建新目录都显示信息

```shell
[hyp@localhost ~]$ mkdir test/a1
mkdir: 无法创建目录"test/a1": 没有那个文件或目录
[hyp@localhost ~]$ mkdir -p  test/a1
[hyp@localhost ~]$ ls -ld test/a1
drwxrwxr-x. 2 hyp hyp 6 3月  17 15:04 test/a1
[hyp@localhost ~]$ mkdir  -m 700  test/a2
[hyp@localhost ~]$ ls -ld test/a2
drwx------. 2 hyp hyp 6 3月  17 15:04 test/a2
```

- **rmdir**

rmdir:remove directory,删除空目录。用法：`rmdir [options] [PATH/TO/SOMEPATH]`。

选项：

-p --parents 递归删除目录

```shell
[hyp@localhost ~]$ rmdir test/a2
[hyp@localhost ~]$ ls -l test
总用量 0
drwxrwxr-x. 2 hyp hyp 6 3月  17 15:04 a1
[hyp@localhost ~]$ rmdir -p  test/a1
[hyp@localhost ~]$ ls
1.jpg  tuc.html  桌面
```

- **tree**

tree：列出指定目录下的所有文件，包括子目录里的文件。用法：`tree [options] [PATH/TO/SOMEPATH]`。一般linux系统不会自带tree工具，要手动安装：`yum -y install   tree`。

选项：

(1)tree  -a 显示所有文件和目录（不加-a,则隐藏目录不显示)
(2)tree -d 显示目录名称而非内容
(3)tree -f 在每个文件或目录之前，显示完整的相对路径名称
(4)tree -F 在执行文件，目录，Socket，符号连接，管道名称名称，各自加上"*","/","=","@","|"号。
(5)tree -r 以相反次序排列
(6)tree -t 用文件和目录的更改时间排序
(7)tree -L n 只显示 n 层目录 （n 为数字）
(8)tree -dirsfirst 目录显示在前,文件显示在后
(9)可以加的参数
-A 使用ASNI绘图字符显示树状图而非以ASCII字符组合。
-C 在文件和目录清单加上色彩，便于区分各种类型。
-D 列出文件或目录的更改时间。
-g 列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码。
-i 不以阶梯状列出文件或目录名称。
-I 不显示符合范本样式的文件或目录名称。
-l 如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。
-n 不在文件和目录清单加上色彩。
-N 直接列出文件和目录名称，包括控制字符。
-p 列出权限标示。
-P 只显示符合范本样式的文件或目录名称。
-q 用"?"号取代控制字符，列出文件和目录名称。

-s 列出文件或目录大小。

```shell
[hyp@localhost ~]$ tree
.
├── 1.jpg
├── test1
│   ├── a1
│   ├── a2
│   ├── a3
│   └── b
├── test2
│   ├── a1
│   ├── a2
│   ├── a3
│   └── b
├── test3
│   ├── a1
│   ├── a2
│   ├── a3
│   └── b
├── tuc.html
└── \346\241\214\351\235\242

16 directories, 2 files
```

### 文件查看

- **cat**

cat：concatenate，用于连接文件并打印到标准输出设备上。用法：`cat [OPTION]... [fileName]...`。

选项：

-n：给显示的文本行编号

-E: 显示行结束符$

-v :使用 ^ 和 M- 符号，除了 LFD 和 TAB 之外。

-T :将 TAB 字符显示为 ^I。

-A, --show-all：等价于 -vET。

-e：等价于"-vE"选项；

-t：等价于"-vT"选项；

```shell
[hyp@localhost ~]$ cat -n fstab
     1
     2  #
     3  # /etc/fstab
     4  # Created by anaconda on Sun Mar 10 17:29:58 2019
     5  #
     6  # Accessible filesystems, by reference, are maintained under '/dev/disk'
     7  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
     8  #
     9  /dev/mapper/centos-root /                       xfs     defaults        0 0
    10  UUID=0c9c56e6-2e8d-48ef-be52-40573a73485e /boot                   xfs     defaults        0 0
    11  /dev/mapper/centos-home /home                   xfs     defaults        0 0
    12  /dev/mapper/centos-var  /var                    xfs     defaults        0 0
    13  /dev/mapper/centos-swap swap                    swap    defaults        0 0
```

- **tac**

tac ：逆序输出文本文件。用法：`tac [OPTION]... [fileName]...`。虽然功能相反，但是命令参数却大不相同，个人感觉 tac 的实用价值非常小。

选项：

-b, --before: 在行前添加分隔符。

-r, --regex: 把分隔符当作正则表达式来解析。

-s, --separator=STRING: 使用指定字符串代替新行作为分隔符。

```shell
[hyp@localhost ~]$ tac fstab
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/mapper/centos-var  /var                    xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
UUID=0c9c56e6-2e8d-48ef-be52-40573a73485e /boot                   xfs     defaults        0 0
/dev/mapper/centos-root /                       xfs     defaults        0 0
#
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
# Accessible filesystems, by reference, are maintained under '/dev/disk'
#
# Created by anaconda on Sun Mar 10 17:29:58 2019
# /etc/fstab
#
```

- **more**

more:命令类似 cat ，不过会以一页一页的形式显示，更方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能（与 vi 相似），使用中的说明文件，请按 h 。用法：`more [-dlfpcsu] [-num] [+/pattern] [+linenum] [fileNames..]`。

选项：

-num ：一次显示的行数

+/pattern ：在每个文档显示前搜寻该字串（pattern），然后从该字串之后开始显示

+num ：从第 num 行开始显示

操作命令：

Enter 向下n行，需要定义。默认为1行
Ctrl+F 向下滚动一屏
空格键 向下滚动一屏
Ctrl+B 返回上一屏
= 输出当前行的行号
：f 输出文件名和当前行的行号
V 调用vi编辑器
!命令 调用Shell，并执行命令
q 退出more

/字符串：向下搜索“字符串”的功能

？字符串：向上搜索“字符串”的功能

n：重复前一个搜索（与 / 或 ？ 有关）

N：反向重复前一个搜索（与 / 或 ？ 有关）

`!<cmd> or :!<cmd>`   ：   执行该命令

- **less**

less：与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。语法：`less [参数] 文件...`。

选项：

​	-b <缓冲区大小> 设置缓冲区的大小

　　-e 当文件显示结束后，自动离开

　　-f 强迫打开特殊文件，例如外围设备代号、目录和二进制文件

　　-g 只标志最后搜索的关键词

　　-i 忽略搜索时的大小写

　　-m 显示类似more命令的百分比

　　-N 显示每行的行号

　　-o <文件名> 将less 输出的内容在指定文件中保存起来

　　-Q 不使用警告音

　　-s 显示连续空行为一行

　　-S 行过长时间将超出部分舍弃

　　-x <数字>  将“tab”键显示为规定的数字空格

操作命令：

```shell
	/字符串：向下搜索“字符串”的功能
　　？字符串：向上搜索“字符串”的功能
　　n：重复前一个搜索（与 / 或 ？ 有关）
　　N：反向重复前一个搜索（与 / 或 ？ 有关）
　　b 向后翻一页
　　d 向后翻半页
　　h 显示帮助界面
　　Q 退出less 命令
　　u 向前滚动半页
　　y 向前滚动一行
　　空格键 滚动一行
　　回车键 滚动一页
　　［pagedown］： 向下翻动一页
　　［pageup］： 向上翻动一页
```

显示多个文件：

方式一，传递多个参数给 less，就能浏览多个文件。

`less file1 file2`

方式二，正在浏览一个文件时，使用 :e 打开另一个文件。

```shell
less file1

:e file2
```

当打开多个文件时，使用如下命令在多个文件之间切换

:n - 浏览下一个文件

:p - 浏览前一个文件

- **head**

head：显示文件的开头至标准输出中（默认文件开头的前10行）。用法：`head [OPTION]... FILE...`。

选项：

```shell
-n　　   显示文件的前n行

-c n　　显示文件的前n个字节

-c -n　　显示文件除了最后n个字节的其他内容

-q　　    隐藏文件名（当指定了多个文件时，在内容的前面会以文件名作为开头）

-v　　    显示文件名（默认单个文件不显示，多个文件显示）
```

- **tail**

tail：用于显示文件中末尾的内容（默认显示最后10行内容）。用法：`tail [OPTION]... FILE...`。

选项:

```shell
-f  用于循环读取文件的内容，监视文件的增长

    -F 与-f类似，区别在于当将监视的文件删除重建后-F仍能监视该文件内容-f则不行，-F有重试的功能，会不断重试

    -c N 显示文件末尾N字节的内容

    -n  显示文件末尾n行内容

    -q  显示多文件的末尾内容时，不显示文件名

    -v  显示多文件的末尾内容时，显示文件名（此为tail的默认选项）

    -s N 与-f合用，表示休眠N秒后在读取文件内容（默认为1s）

   --pid=<进程号PID> 与“-f”选项连用，当指定的进程号的进程终止后，自动退出tail命令
```



- **echo**

echo:回显。用法：`echo [SHORT-OPTION]... [STRING]...`。

选项：

```shell
-n: 不进行换行；
-e：让转义符生效；
	\n：换行
	\t：制表符
```

STRING可以使用引号，单引号和双引号均可用；
单引号：强引用，变量引用不执行替换；
`~]# echo '$SHELL'`
双引号：弱引用，变量引用会被替换；
`~]# echo "$SHELL"`

注意：变量引用的正规符号：`${name}`，声明变量：`name=values`

- **cut**

cut：文件内容查看，以某种方式按照文件的行进行分割。用法：`cut [选项]... [文件]...`

参数：

```shell
 -b 按字节选取 忽略多字节字符边界，除非也指定了 -n 标志
 -c    按字符选取
 -d 自定义分隔符，默认为制表符。
 -f 与-d一起使用，指定显示哪个区域。
```

范围控制：

 n:只有第n项
    n-:从第n项一直到行尾
    n-m:从第n项到第m项(包括m)
    -m:从一行的开始到第m项(包括m)
    -:从一行的开始到结束的所有项



```shell
[hyp@localhost ~]$ cut  -d ' ' -f1 fstab

#
#
#
#
#
#
#
/dev/mapper/centos-root
UUID=0c9c56e6-2e8d-48ef-be52-40573a73485e
/dev/mapper/centos-home
/dev/mapper/centos-var
/dev/mapper/centos-swap
```

缺点： 有的时候分隔符很难确定

- **nl**

nl (Number of Lines) 将指定的文件添加行号标注后写到标准输出。如果不指定文件或指定文件为"-" ，程序将从标准输入读取数据。用法：`nl [选项]... [文件]...`

参数：

-b  ：指定行号指定的方式，主要有两种：

-b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；

-b t ：如果有空行，空的那一行不要列出行号(默认值)；

-n  ：列出行号表示的方法，主要有三种：

-n ln ：行号在萤幕的最左方显示；

-n rn ：行号在自己栏位的最右方显示，且不加 0 ；

-n rz ：行号在自己栏位的最右方显示，且加 0 ；

-w  ：行号栏位的占用的位数。

-p 在逻辑定界符处不重新开始计算。

```shell
[hyp@localhost ~]$ nl fstab

     1  #
     2  # /etc/fstab
     3  # Created by anaconda on Sun Mar 10 17:29:58 2019
     4  #
     5  # Accessible filesystems, by reference, are maintained under '/dev/disk'
     6  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
     7  #
     8  /dev/mapper/centos-root /                       xfs     defaults        0 0
     9  UUID=0c9c56e6-2e8d-48ef-be52-40573a73485e /boot                   xfs     defaults        0 0
    10  /dev/mapper/centos-home /home                   xfs     defaults        0 0
    11  /dev/mapper/centos-var  /var                    xfs     defaults        0 0
    12  /dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 文件操作

文件创建总共有种方法：使用编辑命令编辑新文件并保存；使用`touch`命令；使用输出重定向新建文件；复制文件等。

文件具有属性：文件名、存储大小、权限、属组、属主、文件节点、创建时间（ctime）、更改时间（mtime）、访问时间（atime）。

- **touch**

touch：用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。可用于记录时间。

语法：`touch [-acfm][-d<日期时间>][-r<参考文件或目录>] [-t<日期时间>][文件或目录…]`

参数：

-a ：改变档案的读取时间记录。
-m ：改变档案的修改时间记录。
-c： 假如目的档案不存在，不会建立新的档案。与 --no-create 的效果一样。
-f： 不使用，是为了与其他 unix 系统的相容性而保留。
-r： 使用参考档的时间记录，与 --file 的效果一样。
-d： 设定时间与日期，可以使用各种不同的格式。
-t： 设定档案的时间记录，格式与 date 指令相同。
--no-create 不会建立新档案。

```shell
[hyp@localhost ~]$ ll fstab
-rw-r--r--. 1 hyp hyp 617 3月  17 15:10 fstab
[hyp@localhost ~]$ touch fstab
[hyp@localhost ~]$ ll fstab
-rw-r--r--. 1 hyp hyp 617 3月  17 15:27 fstab
[hyp@localhost ~]$ touch test
[hyp@localhost ~]$ ll test
-rw-rw-r--. 1 hyp hyp 0 3月  17 15:28 test
```

- **ln**

ln：link，建立链接文件。

用法：`ln [-s] src  dest`

不加`-s`创建硬链接，加上创建软链接。

区别：

硬链接不能跨文件系统，不支持对目录创建硬链接，由于创建一个对文件inode的引用，所以硬链接的属性和指向的源文件相同。源文件被删除则硬链接链接inode引用数减一，仍然有效。

软链接是新建一个链接文件，该文件保存源文件的绝对路径，源文件被删除则软链接失效，可跨文件系统，可对目录创建链接。

```shell
[hyp@localhost ~]$ ln fstab fstabd
[hyp@localhost ~]$ ln -s fstab fstabs
[hyp@localhost ~]$ ll
总用量 116
-rw-r--r--. 2 hyp hyp   617 3月  17 15:27 fstab
-rw-r--r--. 2 hyp hyp   617 3月  17 15:27 fstabd
lrwxrwxrwx. 1 hyp hyp     5 3月  17 16:01 fstabs -> fstab
hyp@localhost ~]$ ln -s test ts
[hyp@localhost ~]$ ll ts
lrwxrwxrwx. 1 hyp hyp 4 3月  17 16:03 ts -> test
[hyp@localhost ~]$ ls  ts
a1  a24
[hyp@localhost ~]$ ls  test
a1  a24
```

- **cp**

cp:copy ,复制文件（夹）。

语法：

```shell
cp [options] source dest
```

或

```shell
cp [options] source... directory
```

用法：

```shell
cp src /PATH/TO/SOMEPATH #复制文件到目录
cp src1 src2 ...  /PATH/TO/SOMEPATH #复制多个文件到目录
cp -r PATH /PATH/TO/SOMEPATH #复制源文件夹到目标文件夹下
cp src /PATH/TO/dest #使用src覆盖dest文件 
```

选项：

-a：此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容。其作用等于dpR参数组合。

-d：复制时保留链接。这里所说的链接相当于Windows系统中的快捷方式。

-f：覆盖已经存在的目标文件而不给出提示。

-i：与-f选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答"y"时目标文件将被覆盖。

-p：除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。

-r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。

-l：不复制文件，只是生成链接文件。

注：-a通常用来备份。

- **mv**

mv：move ，移动文件（夹）。

语法：`mv [选项] 源文件或目录 目标文件或目录`

用法：

```shell
mv src dest #将源文件名改为目标文件名
mv src /PATH/TO/SOMEPATH #文件移动到其他目录
mv srcpath /PATH/TO/SOMEPATH #目标目录已存在，将源目录移动到目标目录；目标目录不存在则改名
```

参数：

-i: 若指定目录已有同名文件，则先询问是否覆盖旧文件;

-f: 在mv操作要覆盖某已有的目标文件时不给任何指示;

- **rm**

rm:remove,删除文件（夹）。

语法：`rm [选项]... 目录... 删除指定的<文件>(即解除链接)。`

选项：

```
  -d      --directory    删除可能仍有数据的目录 (只限超级用户)

-f      --force          略过不存在的文件，不显示任何信息，强制删除

-i      --interactive 进行任何删除操作前必须先确认

-r/R --recursive    同时删除该目录下的所有目录层

-v      --verbose     详细显示进行的步骤  
```

注意：删除后不能恢复

- **file**

file：文件的类型。用法：`file [文件或目录...]`,多个文件之间使用空格分开，可以使用shell通配符匹配多个文件。

```shell
[hyp@localhost ~]$ ls
1.jpg  tuc.html
[hyp@localhost ~]$ file tuc.html
tuc.html: HTML document, UTF-8 Unicode text, with very long lines
[hyp@localhost ~]$ file 1.jpg
1.jpg: JPEG image data, JFIF standard 1.01, comment: "CREATOR: gd-jpeg v1.0 (using IJG JPEG v62), quality = 95"
```

- **stat**

stat:文件/文件系统的详细信息显示。

用法：`stat 文件名`

![](https://ws1.sinaimg.cn/large/006KyevZgy1g15vuk71umj30t50c9gnu.jpg)

stat显示出inode的内容--inode包含文件的元信息，具体来说有以下内容：

```shell
文件的字节数
文件拥有者的User ID
文件的Group ID
文件的读、写、执行权限
文件的时间戳，共有三个
链接数，即有多少文件名指向这个inode
文件数据block的位置
```

通过ls怎么查询这三个时间？

```shell
  ls -lc filename         列出文件的 ctime
    ls -lu filename         列出文件的 atime
    ls -l filename          列出文件的 mtime --ll默认显示的就是这个时间
```

1. Access Time：简写为atime，表示文件的访问时间。当文件内容被访问时，更新这个时间 
2. Modify Time：简写为mtime，表示文件内容的修改时间，当文件的数据内容被修改时，更新这个时间。 
3. Change Time：简写为ctime，表示文件的状态时间，当文件的状态被修改时，更新这个时间，例如文件的链接数，大小，权限，Blocks数。

ctime指inode上一次改变的时间，mtime指文件内容上一次修改的时间，atime指文件上一次打开的时间。

实例：

```shell
[hyp@localhost ~]$ stat tuc.html
  文件："tuc.html"
  大小：8850            块：24         IO 块：4096   普通文件
设备：fd02h/64770d      Inode：196627      硬链接：1
权限：(0664/-rw-rw-r--)  Uid：( 1000/     hyp)   Gid：( 1000/     hyp)
环境：unconfined_u:object_r:user_home_t:s0
最近访问：2019-03-17 14:32:34.610784851 +0800
最近更改：2019-03-13 09:01:25.726294088 +0800
最近改动：2019-03-13 09:01:25.726294088 +0800
创建时间：-
[hyp@localhost ~]$ ls -l tuc.html #列出mtime
-rw-rw-r--. 1 hyp hyp 8850 3月  13 09:01 tuc.html
[hyp@localhost ~]$ ls -lc tuc.html #列出ctime
-rw-rw-r--. 1 hyp hyp 8850 3月  13 09:01 tuc.html
[hyp@localhost ~]$ ls -lu tuc.html  #列出atime
-rw-rw-r--. 1 hyp hyp 8850 3月  17 14:32 tuc.html
```

### 时间/日期

下节将使用使用`shutdown`命令，里面讲到了定时重启/关机，这节我们就来谈一下有关时间的命令。时间分为系统时间和硬件时间。

- **date**

显示/设置系统时间。

用法：

```shell
date MMDDhhmm[cc]YY.ss #设置时间
date[+FORMAT]
	%Y:四位年份
　　%y：两位年份
　　%m:月
　　%M：分钟
　　%d：日
　　%h：英文简写的月
　　%H：时
　　%S：秒
　　%s：现在距离1970年1月1号0点0分（unix元年）的秒数，timestamp（时间戳）
　　%D:月/日/年
　　%F:年-月-日
　　%T:时：分：秒
```

实例：

```shell
[hyp@localhost ~]$ date
2019年 03月 17日 星期日 17:07:29 CST
[hyp@localhost ~]$ date +%Y-%m-%d
2019-03-17
```

- **time**

使用`time COMMAND`查看命令执行耗费时间，使用`times`查看shell 及其所有子进程的累计用户空间和。

- **clock/hwclock**

两个命令用法一样。

（1）更新机器的硬件时间。命令为：*hwclock --adjust*硬件时钟通常被设置成全球标准时间（UTC），而将时区信息保存在/usr/share/lib/timezone （或者在某些系统中可能是/usr/local/timezone）目录下某个适当的文件中，然后用一个符号链接文件/etc/localtime指向它。

（2）查看硬件时钟。命令为：*hwclock --show*

（3）重置硬件时钟用：hwclock --set --date=mm/dd/yy hh:MM:ss"

- **cal**

显示日历

```shell
用法：
 cal [选项] [[[日] 月] 年]

选项：
 -1, --one        只显示当前月份(默认)
 -3, --three      显示上个月、当月和下个月
 -s, --sunday     周日作为一周第一天
 -m, --monday     周一用为一周第一天
 -j, --julian     输出儒略日
 -y, --year       输出整年
 -V, --version    显示版本信息并退出
 -h, --help       显示此帮助并退出
```

- **时区**

  使用命令`tzselect`，根据提示一步步修改即可，或者直接修改配置文件：

  `cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`，选择对应时区文件对/etc/localtime文件进行覆盖。

- **网络时间同步**

使用ntp进行时间同步，也可以使用ntpdate直接同步（需要提前安装 `sudo yum install ntp`）。

`ntpdate 0.arch.pool.ntp.org`

目前比较新的Linux发行版本都使用了systemd，可以直接使用timedatectl 命令开启ntp同步就可以了

`timedatectl set-ntp yes`

### 关机/重启/注销

讲了那么多，还没讲到关机，同学可能还不知道怎么关机。Linux的关机是指通过软件关闭计算机，在关机之前，需要将缓冲区数据同步到磁盘，将内存中数据持久化，终止进程，然后一步步控制硬件断电。

`shutdown -r/h/c [time]`

常用的关机/重启/注销命令有：

```shell
#关机
shutdown -h now  #立刻关机重启，工作中常用
shutdown -h +1    #1分钟后关机
init 0
halt                      #立即停止系统，需要人工关闭电源
halt -p                  #
poweroff　　　　  #立即停止系统，并且关闭电源
#重启
reboot　　　　　　#工作中常用
shutdown -r now      #工作中常用
shutdown -r +1　　 #一分钟后重启
init 6
#注销
logout
exit　　　　　　#工作中常用
ctrl+d　　　　　#工作中常用
#shutdown命令撤销
shutdown -c
```

注：服务器尽量不要关机。

### 补充命令

- **别名**

使用`alias [别名]`查看别名

使用`alias [别名]='COMMAND'`设置别名

撤销别名：`unalias [别名]`

- **which**

显示命令的绝对路径。

which [options] programname [...]
	--skip-alias：忽略别名

例如：

```shell
[hyp@localhost ~]$ which ls
alias ls='ls --color=auto'
        /usr/bin/ls
[hyp@localhost ~]$ which --skip-alias ls
/usr/bin/ls
[hyp@localhost ~]$ which which
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
        /usr/bin/alias
        /usr/bin/which
```

- **whatis**

显示帮助文档中对命令的描述。

```shell
[hyp@localhost ~]$ whatis whatis
whatis (1)           - display manual page descriptions
[hyp@localhost ~]$ whatis passwd
sslpasswd (1ssl)     - compute password hashes
passwd (1)           - update user's authentication tokens
```

- **whereis**

显示命令的二进制文件、源文件和帮助文件的位置。

whereis [options] name...
 -b         只搜索二进制文件
 -B <目录>  定义二进制文件查找路径
 -m         只搜索 man 手册
 -M <目录>  定义 man 手册查找路径
 -s         只搜索源代码
 -S <目录>  定义源代码查找路径
 -f         终止 <目录> 参数列表
 -u         搜索不常见记录
 -l         输出有效查找路径

```shell
[hyp@localhost ~]$ whereis passwd
passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz
[hyp@localhost ~]$ whereis whereis
whereis: /usr/bin/whereis /usr/share/man/man1/whereis.1.gz
```

- **who**

显示当前已登录的用户信息。用法：who [选项]... [ 文件 | 参数1 参数2 ]

```shell
  -a, --all             等于-b -d --login -p -r -t -T -u 选项的组合
  -b, --boot            上次系统启动时间
  -d, --dead            显示已死的进程
  -H, --heading 输出头部的标题列
  -l，--login           显示系统登录进程
      --lookup          尝试通过 DNS 查验主机名
  -m                    只面对和标准输入有直接交互的主机和用户
  -p, --process 显示由 init 进程衍生的活动进程
  -q, --count           列出所有已登录用户的登录名与用户数量
  -r, --runlevel        显示当前的运行级别
  -s, --short           只显示名称、线路和时间(默认)
  -T, -w, --mesg        用+，- 或 ? 标注用户消息状态
  -u, --users           列出已登录的用户
```



- **w**

显示登录用户以及用户的行为。

- **last**

显示最近的登录信息。它会读取/var/log目录下名称为wtmp的文件，并把该文件记录的登录系统或终端的用户名单全部显示出来。默认显示wtmp的记录，btmp能显示的更详细，可以显示远程登录，例如ssh登录。

选项：

```shell
-num |-n num指定输出记录的条数
-f file 指定记录文件作为查询的log文件
-t YYYYMMDDHHMMSS 显示指定时间之前的登录情况
username 账户名称<``br``>tty 终端机编号
-R 不显示登录系统或终端的主机名称或IP
-a 将登录系统或终端的主机名过IP地址显示在最后一行
-d 将IP地址转成主机名称
-I 显示特定IP登录情况。
-o 读取有linux-libc5应用编写的旧类型wtmp文件
-x 显示系统关闭、用户登录和退出的历史
-F 显示登录的完整时间
-w 在输出中显示完整的用户名或域名
```

第一列：用户名

第二列：终端位置（pts/0伪终端，意味着从SSH或telnet等工具远程连接的用户，图形界面终端归于此类。tty0直接连接到计算机或本地连接的用户。后面的数字代表连接编号）

第三列：登录IP或内核（如果是：0.0或者什么都没有，意味着用户通过本地终端连接。除了重启活动，内核版本会显示在状态中）

第四列：开始时间

第五列：结束时间（still login in尚未退出，down直到正常关机，crash直到强制关机）

第六列：持续时间

```shell
[hyp@localhost ~]$ last -10
hyp      pts/0        192.168.174.1    Sun Mar 17 16:53   still logged in
hyp      pts/0        192.168.174.1    Sun Mar 17 13:55 - 16:52  (02:57)
hyp      pts/0        192.168.174.1    Sun Mar 17 13:52 - 13:53  (00:01)
root     tty1                          Sun Mar 17 10:00 - 10:00  (00:00)
hyp      pts/0        192.168.174.1    Sun Mar 17 09:59 - 13:45  (03:46)
hyp      pts/0        192.168.174.1    Sun Mar 17 09:02 - 09:29  (00:27)
reboot   system boot  3.10.0-957.el7.x Sun Mar 17 09:01 - 17:55  (08:54)
hyp      pts/0        192.168.174.1    Sat Mar 16 17:01 - 18:49  (01:47)
reboot   system boot  3.10.0-957.el7.x Sat Mar 16 16:56 - 20:32  (03:35)
hyp      pts/0        192.168.174.1    Sat Mar 16 16:15 - 16:30  (00:15)

wtmp begins Sun Mar 10 18:06:08 2019
```

- **查看内核版本**

  使用`uname`命令查看

```shell
[hyp@localhost ~]$ uname -a
Linux localhost.localdomain 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

本文篇幅过长，请大家观看下一篇。