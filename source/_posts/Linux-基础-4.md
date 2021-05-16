---
title: Linux-权限管理和文本处理
date: 2019-03-19 16:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

linux上使用者都有一个唯一的标识符，称为UID,其中管理员UID为0，在CenOS6上系统用户UID为1到499，在CentOS7上系统用户UID为1到999，普通登录用户UID到65535。linux上用户必须属于一个组（基本组），组也有一个组标识GID，其中管理员组GID为0，在CenOS6上系统用户组GID为1到499，在CentOS7上系统用户组GID为1到999，普通登录用户组GID到65535。

此外，用户也可以属于多个附加组。私有组：组名同用户名，且只包含一个用户；公共组：组内包含了多个用户。组名和GID的解析通过/etc/group文件记录。

<!--more-->

### 认证信息

用户通过给出用户名和密码来登录系统，系统根据/etc/passwd文件来查找用户名对应UID，如果允许该用户登录，则查看密码是否与/etc/shadow里面的密码相同，相同则通过身份认证（Authentication），然后进入授权（Authorization），在用户登录全过程进行审计（Audition），记录其行为。

密码使用策略：

1、使用随机密码；
2、最短长度不要低于8位；
3、应该使用大写字母、小写字母、数字和标点符号四类字符中至少三类；
4、定期更换；

设置密码时会使用单向加密算法进行加密，存储在/etc/shadow，后面登录使用用户输入密码进行单向加密与/etc/shadow存储的密码比较。

/etc/passwd文件存储用户的信息。总共有七个字段，分别是：`name:password:UID:GID:GECOS:directory:shell`。name: 用户名；password：可以是加密的密码，也可是占位符x；UID：用户ID；GID：用户所属的主组的ID号；GECOS：注释信息；directory：用户的家目录；shell：用户的默认shell，登录时默认shell程序；

```
root:x:0:0:root:/root:/bin/bash
```

/etc/shadow文件用来存储用户密码。各字段表示意义：

```
用户名:加密的密码:最近一次修改密码的时间:最短使用期限:最长使用期限:警告期段:过期期限:保留字段
```

/etc/group是组的信息库，里面的字段为：`group_name:password:GID:user_list`。user_list：该组的用户成员；以此组为附加组的用户的用户列表。

用户组也有密码，但不经常用，密码存放在/etc/gshadow。

安装上下文：进程以其发起者的身份运行；进程对文件的访问权限，取决于发起此进程的用户的权限。

系统用户：为了能够让那后台进程或服务类进程以非管理员的身份运行，通常需要为此创建多个普通用户；这类用户从不用登录系统

### 用户命令

需要管理员权限才能使用。创建用户和组的时候，默认使用/etc/login.defs中的默认值。

```shell
#QMAIL_DIR      Maildir
MAIL_DIR        /var/spool/mail #用户默认邮箱位置
#MAIL_FILE      .mail
# Password aging controls:
PASS_MAX_DAYS   99999 #密码最长过期时间，天
PASS_MIN_DAYS   0  #密码最短过期时间，天
PASS_MIN_LEN    5 #密码最小长度
PASS_WARN_AGE   7 #密码过期时间前多少天发出警告
#用户ID
UID_MIN                  1000 #最小自动分配的普通登录用户ID
UID_MAX                 60000 #最大自动分配的普通登录用户ID
# System accounts
SYS_UID_MIN               201 #最小自动分配的系统用户ID
SYS_UID_MAX               999 #最大自动分配的系统用户ID
#用户组ID
GID_MIN                  1000
GID_MAX                 60000
# System accounts
SYS_GID_MIN               201
SYS_GID_MAX               999
#USERDEL_CMD    /usr/sbin/userdel_local #用户删除调用的脚本
#是否创建家目录
CREATE_HOME     yes
#家目录访问权限掩码
UMASK           077
# This enables userdel to remove user groups if no members exist.
USERGROUPS_ENAB yes
# 使用 SHA512加密密码.
ENCRYPT_METHOD SHA512
```

#### 用户组管理

**groupadd**

用户组添加。语法：`groupadd [选项] GROUPNAME`。

选项：

```
-g GID：指定GID；默认是上一个组的GID+1；
-r: 创建系统组；
-p, --password PASSWORD       为新组使用此加密过的密码
 -R, --root CHROOT_DIR         chroot 到的目录，后面再讲
```

**groupmod**

修改用户组属性。语法：`groupmod [选项] GROUPNAME`



```
 -g, --gid GID                 将组 ID 改为 GID
  -n, --new-name NEW_GROUP      改名为 NEW_GROUP
  -o, --non-unique              允许使用重复的 GID
  -p, --password PASSWORD       将密码更改为(加密过的) PASSWORD
  -R, --root CHROOT_DIR         chroot 到的目录，后面再讲
```

**groupdel**

删除用户组。用法：`groupdel [选项] GROUP`。

```
  -R, --root CHROOT_DIR         chroot 到的目录，后面再讲
```

**groupmems**

更改组内成员。用法：`groupmems [选项] [动作]`。

选项：
  -g, --group groupname         更改组 groupname，而不是用户的组(只 root)
  -R, --root CHROOT_DIR         chroot 到的目录，后面再讲

动作：
  -a, --add username            将用户 username 添加到组成员中
  -d, --delete username         从组的成员中删除用户 username
  -p, --purge                   从组中移除所有成员
  -l, --list                    列出组中的所有成员

实例：

```shell
[hyp@promote ~]$ sudo groupadd ping
[hyp@promote ~]$ sudo tail -1 /etc/group
ping:x:1002:
[hyp@promote ~]$ sudo groupmod ping -n newping
[hyp@promote ~]$ sudo tail -1 /etc/group
newping:x:1002:
[hyp@promote ~]$ sudo groupmems -g newping -a root
[hyp@promote ~]$ sudo groupmems -g newping -l
root
[hyp@promote ~]$ sudo groupdel  newping
[hyp@promote ~]$ sudo tail -1 /etc/group  #并不是newping
```

**groups**

查看用户所属组。用法：groups  [用户名]...

实例：

```shell
[hyp@promote ~]$ whoami #查看本地等于用户
hyp
[hyp@promote ~]$ groups hyp #查看此用户所属组
hyp : hyp wheel
```

#### 用户管理

**useradd**

用户添加命令。用法：`useradd [选项] USERNAME`。

```
-u, --uid UID：指定UID；
-g, --gid GROUP：指定基本组ID，此组得事先存在；
-G, --groups GROUP1[,GROUP2,...[,GROUPN]]]：指明用户所属的附加组，多个组之间用逗号分隔；
-c, --comment COMMENT：指明注释信息；
-d, --home HOME_DIR：以指定的路径为用户的家目录；通过复制/etc/skel此目录并重命名实现；指定的家目录路径如果事先存在，则不会为用户复制环境配置文件；
-s, --shell SHELL：指定用户的默认shell，可用的所有shell列表存储在/etc/shells文件中；
-r, --system：创建系统用户；
-m, --create-home     创建用户的主目录
-M, --no-create-home          不创建用户的主目录
-D, --defaults                显示或更改默认的 useradd 配置
-e, --expiredate EXPIRE_DATE  新账户的过期日期
-f, --inactive INACTIVE       新账户的密码不活动期
-k, --skel SKEL_DIR   使用此目录作为SHELL目录
-b, --base-dir BASE_DIR       新账户的主目录的基目录
useradd -D：显示创建用户的默认配置；
useradd -D 选项: 修改默认选项的值；
修改的结果保存于/etc/default/useradd文件中；
```

**userdel**

删除用户。用法：`userdel [选项] USERNAME`。

选项：

```
-r：删除用户时一并删除其家目录
```

**usermod**

修改用户信息。用法：`usermod [选项] USERNAME`。

选项：

```
-u, --uid UID：修改用户的ID为此处指定的新UID；
-g, --gid GROUP：修改用户所属的基本组；
-G, --groups GROUP1[,GROUP2,...[,GROUPN]]]：修改用户所属的附加组；原来的附加组会被覆盖；
-a, --append：与-G一同使用，用于为用户追加新的附加组；
-c, --comment COMMENT：修改注释信息；
-d, --home HOME_DIR：修改用户的家目录；用户原有的文件不会被转移至新位置；
-m, --move-home：只能与-d选项一同使用，用于将原来的家目录移动为新的家目录；
-l, --login NEW_LOGIN：修改用户名；
-s, --shell SHELL：修改用户的默认shell；
-L, --lock：锁定用户密码；即在用户原来的密码字符串之前添加一个"!"；
-U, --unlock：解锁用户的密码；
```

**id**

显示用户的真实ID。语法：`id [OPTION]... [USER]`。

组后面第一个组为默认有效组，其他为附加组。

选项：

```
-u: 仅显示有效的UID；
-g: 仅显示用户的基本组ID; 
-G：仅显示用户所属的所有组的ID；
-n: 显示名字而非ID；
```

实例：

```shell
[hyp@localhost ~]$ id root
uid=0(root) gid=0(root) 组=0(root)
[hyp@localhost ~]$ id hyp
uid=1000(hyp) gid=1000(hyp) 组=1000(hyp),10(wheel)
```

**su**

switch user：切换登录用户。

登录式切换：会通过读取目标用户的配置文件来重新初始化
	su - USERNAME
	su -l USERNAME
非登录式切换：不会读取目标用户的配置文件进行初始化
	su USERNAME

注意：管理员可无密码切换至其它任何用户；

-c 'COMMAND'：仅以指定用户的身份运行此处指定的命令。



**练习题**：

练习1：创建用户gentoo，UID为4001，基本组为gentoo，附加组为distro(GID为5000)和peguin(GID为5001)；

```shell
[hyp@promote ~]$ sudo groupadd -g 5000 distro
[hyp@promote ~]$ sudo groupadd -g 5001 peguin
[hyp@promote ~]$ tail -2 /etc/group
distro:x:5000:
peguin:x:5001:
[hyp@promote ~]$ sudo useradd -u 4001 gentoo  -G 5000,5001
[hyp@promote ~]$ tail -3 /etc/group
distro:x:5000:gentoo
peguin:x:5001:gentoo
gentoo:x:4001:
[hyp@promote ~]$ sudo tail -1 /etc/passwd
gentoo:x:4001:4001::/home/gentoo:/bin/bash
```

练习2：创建用户fedora，其注释信息为"Fedora Core"，默认shell为/bin/tcsh；

```shell
[hyp@promote ~]$ sudo useradd fedora -c 'Fedora Core' -s /bin/tcsh
[hyp@promote ~]$ tail -1 /etc/passwd
fedora:x:4002:4002:Fedora Core:/home/fedora:/bin/tcsh
```

练习3：修改gentoo用户的家目录为/var/tmp/gentoo；要求其原有文件仍能被用户访问；

```shell
[hyp@promote ~]$ sudo usermod gentoo -m -d /var/tmp/gentoo/
```

练习4：为gentoo新增附加组netadmin；

```shell
[hyp@promote ~]$ sudo groupadd netadmin -g 4003
[hyp@promote ~]$ sudo usermod gentoo  -aG 4003
[hyp@promote ~]$ tail -4 /etc/group
peguin:x:5001:gentoo
gentoo:x:4001:
fedora:x:4002:
netadmin:x:4003:gentoo
```

练习5：删除所创建的用户组和用户

```shell
[hyp@promote ~]$ sudo groupdel 'distro'
[hyp@promote ~]$ sudo groupdel 'peguin'
[hyp@promote ~]$ sudo groupdel 'netadmin'
#由于用户组fedora和gentoo是用户的主组
[hyp@promote ~]$ sudo groupdel 'gentoo'
groupdel：不能移除用户“gentoo”的主组
[hyp@promote ~]$ sudo groupdel 'fedora'
groupdel：不能移除用户“fedora”的主组
#需要先删除用户，加上-r会删除用户家目录和邮件目录。此时也会删除对应的主组
[hyp@promote ~]$ sudo userdel -r fedora
[hyp@promote ~]$ sudo userdel -r gentoo
```

#### 命令补充

**passwd**

用户密码修改。语法：`passwd [选项] USERNAME`

(1) passwd：修改用户自己的密码；
(2) passwd USERNAME：修改指定用户的密码，但仅root有此权限；

```
-l, -u：锁定和解锁用户；
-d：清除用户密码串；
-e DATE: 过期期限，日期；
-i DAYS：非活动期限；
-n DAYS：密码的最短使用期限；
-x DAYS：密码的最长使用期限；
-w DAYS：警告期限；
--stdin：
	echo "PASSWORD" | passwd --stdin USERNAME
```

**chage**

更改用户密码过期信息。chage [OPTION]  USERNAME

选项：

```
  -d, --lastday 最近日期        将最近一次密码设置时间设为“最近日期”
  -E, --expiredate 过期日期     将帐户过期时间设为“过期日期”
  -I, --inactive INACITVE       过期 INACTIVE 天数后，设定密码为失效状态
  -l, --list                    显示帐户年龄信息
  -m, --mindays 最小天数        将两次改变密码之间相距的最小天数设为“最小天数”
  -M, --maxdays 最大天数        将两次改变密码之间相距的最大天数设为“最大天数”
  -W, --warndays 警告天数       将过期警告天数设为“警告天数”
```

实例：

```shell
[hyp@localhost ~]$ chage -l hyp
最近一次密码修改时间                                    ：从不
密码过期时间                                    ：从不
密码失效时间                                    ：从不
帐户过期时间                                            ：从不
两次改变密码之间相距的最小天数          ：0
两次改变密码之间相距的最大天数          ：99999
在密码过期之前警告的天数        ：7
[hyp@localhost ~]$ sudo chage -W 30 hyp
[sudo] hyp 的密码：
[hyp@localhost ~]$ chage -l hyp
最近一次密码修改时间                                    ：从不
密码过期时间                                    ：从不
密码失效时间                                    ：从不
帐户过期时间                                            ：从不
两次改变密码之间相距的最小天数          ：0
两次改变密码之间相距的最大天数          ：99999
在密码过期之前警告的天数        ：30
```

**pwck**

检查密码文件的完整性。用法：`pwck [选项] [passwd [ shadow ]]`。

pwck 命令检查用户及其认证信息的完整性。它检查 /etc/passwd 和/etc/shadow格式正确、数据有效。将会提示用户删除格式不正确或者有其它错误的项。



**gpasswd**

为组提供管理员，文件/etc/gshadow。用法：`gpasswd [OPTION] GROUPNAME`。

选项：

```
 -a, --add USER                向组 GROUP 中添加用户 USER
  -d, --delete USER             从组 GROUP 中添加或删除用户
  -Q, --root CHROOT_DIR         要 chroot 进的目录
  -r, --delete-password         remove the GROUP's password
  -R, --restrict                向其成员限制访问组 GROUP
  -M, --members USER,...        设置组 GROUP 的成员列表
  -A, --administrators ADMIN,...        设置组的管理员列表
```

除非使用 -A 或 -M 选项，不能结合使用这些选项。

```shell
[hyp@localhost ~]$ sudo useradd -m ping
[hyp@localhost ~]$ id ping
uid=1002(ping) gid=1002(ping) 组=1002(ping)
[hyp@localhost ~]$ sudo gpasswd -a ping hyp
正在将用户“ping”加入到“hyp”组中
[hyp@localhost ~]$ id ping
uid=1002(ping) gid=1002(ping) 组=1002(ping),1000(hyp)
[hyp@localhost ~]$ sudo gpasswd -d ping hyp
正在将用户“ping”从“hyp”组中删除
[hyp@localhost ~]$ id ping
uid=1002(ping) gid=1002(ping) 组=1002(ping)
[hyp@localhost ~]$ sudo userdel -r ping
```

**grpck**

检查组文件的完整性（etc/group以及/etc/gshadow文件）。用法：`grpck [选项] [group [ shadow ]]`。

**newgrp**

临时切换指定的组为基本组。用法：`newgrp [-] [group]`

```shell
[hyp@localhost ~]$ id
uid=1000(hyp) gid=1000(hyp) 组=1000(hyp),10(wheel) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[hyp@localhost ~]$  newgrp wheel
[hyp@localhost ~]$ id
uid=1000(hyp) gid=10(wheel) 组=10(wheel),1000(hyp) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

如果一个用户同时隶属于两个或两个以上分组，需要切换到其它用户组来执行一些操作，就用到了newgrp命令切换当前登陆所在组。

**chsh**

更改用户默认Shell脚本。用法：`chsh [选项] [用户名]`

选项：

```
 -s, --shell <shell>  指定登录 shell
 -l, --list-shells    打印 shell 列表并退出
```

示例：

```shell
[hyp@localhost ~]$ chsh -l
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

**finger**

管理用户个人办公信息，需要安装 `sudo yum install finger`。

```shell
[hyp@localhost ~]$ finger -s  hyp
Login     Name       Tty      Idle  Login Time   Office     Office Phone   Host
hyp       hyp        pts/0          Mar 20 12:59                           (192.168.174.1)
[hyp@localhost ~]$ finger hyp
Login: hyp                              Name: hyp
Directory: /home/hyp                    Shell: /bin/bash
On since 三 3月 20 12:59 (CST) on pts/0 from 192.168.174.1
   5 seconds idle
No mail.
No Plan.
```

**chfn**

改变`finger`命令显示的信息。语法：`chfn [选项] [用户名]`。

选项：

```
 -f, --full-name <全名>       真实姓名
 -o, --office <办公>          办公号码
 -p, --office-phone <电话>   办公电话
 -h, --home-phone <电话>     住宅电话
```

示例：

```shell
[hyp@localhost ~]$ finger hyp
Login: hyp                              Name: hyp
Directory: /home/hyp                    Shell: /bin/bash
On since 三 3月 20 12:59 (CST) on pts/0 from 192.168.174.1
   2 seconds idle
No mail.
No Plan.
[hyp@localhost ~]$ chfn -f 韩云朋 hyp
Changing finger information for hyp.
密码：
Finger information changed.
[hyp@localhost ~]$ finger hyp
Login: hyp                              Name: 韩云朋
Directory: /home/hyp                    Shell: /bin/bash
On since 三 3月 20 12:59 (CST) on pts/0 from 192.168.174.1
   6 seconds idle
No mail.
No Plan.
```

#### 理解chroot

chroot，即 change root directory (更改 root 目录)。在 linux 系统中，系统默认的目录结构都是以 `/`，即是以根 (root) 开始的。而在使用 chroot 之后，系统的目录结构将以指定的位置作为 `/` 位置。

用法：`chroot [选项] 新根 [命令 [参数]...]`
　或：`chroot 选项`

在经过 chroot 之后，系统读取到的目录和文件将不在是旧系统根下的而是新根下(即被指定的新的位置)的目录结构和文件，因此它带来的好处大致有以下3个：

1. 增加了系统的安全性，限制了用户的权力；

   在经过 chroot 之后，在新根下将访问不到旧系统的根目录结构和文件，这样就增强了系统的安全性。这个一般是在登录 (login) 前使用 chroot，以此达到用户不能访问一些特定的文件。

2. 建立一个与原系统隔离的系统目录结构，方便用户的开发；

   使用 chroot 后，系统读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构。在这个新的环境中，可以用来测试软件的静态编译以及一些与系统不相关的独立开发。

3. 切换系统的根目录位置，引导 Linux 系统启动以及急救系统等。

   chroot 的作用就是切换系统的根位置，而这个作用最为明显的是在系统初始引导磁盘的处理过程中使用，从初始 RAM 磁盘 (initrd) 切换系统的根位置并执行真正的 init。另外，当系统出现一些问题时，我们也可以使用 chroot 来切换到一个临时的系统。

参考：[chroot 命令实例讲解](https://yq.aliyun.com/articles/82756)

### 权限管理

使用`ls -l`可以查看文件（夹）的访问权限，左三位：定义user(owner)的权限；中三位：定义group的权限；右三位：定义other的权限。r为可读，数字4表示；w为可写，数字2表示；x为可执行，数字1表示。

```
练习：rw-rw-r--, rwxrwxr-x, rwxr-x---, rw------, rwxr-xr-x
	664,775,750,600,755
```

进程安全上下文，进程对文件的访问权限应用模型：进程的属主与文件的属主是否相同；如果相同，则应用属主权限；否则，则检查进程的属主是否属于文件的属组；如果是，则应用属组权限；否则，就只能应用other的权限。

`rwx`对文件来说，r表示可以获取文件数据，w表示可以修改文件的数据，x表示可将此文件运行。

`rwx`对文件夹来说，r表示可以获取其下的文件列表，x表示看完已修改此目录下的文件列表，即创建或删除文件，x表示可以进入该目录，使用该目录作为工作目录，必须以rw为前提。

#### 权限命令

**chmod**

更改文件（夹）权限。

语法：

chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL-MODE FILE...
chmod [OPTION]... --reference=RFILE FILE...

授权对象：u属主，g属组，o其他，a所有

授权表示法：+[rwx]增加权限，-[rwx]去除权限，=[rwx]设置权限

或者直接使用数字表示：`chmod 744 filename`

选项：

-R, --recursive：递归修改

注意：用户仅能修改属主为自己的那些文件的权限

示例：

```shell
[hyp@localhost ~]$ ls -l test
-rw-rw-r--. 1 hyp hyp 8872 3月  19 22:03 test
[hyp@localhost ~]$ chmod g-w test
[hyp@localhost ~]$ ls -l test
-rw-r--r--. 1 hyp hyp 8872 3月  19 22:03 test
[hyp@localhost ~]$ chmod 664 test
[hyp@localhost ~]$ ls -l test
-rw-rw-r--. 1 hyp hyp 8872 3月  19 22:03 test
```

**练习题：**

用户对目录有写权限，但对目录下的文件没有写权限时，能否修改此文件内容？能否删除此文件？

```shell
[hyp@localhost ~]$ mkdir ping
[hyp@localhost ~]$ ll -d ping/
drwxrwxr-x. 2 hyp hyp 6 3月  20 16:26 ping/
[hyp@localhost ~]$ echo 'cat /etc/fstab' > ping/fstab
[hyp@localhost ~]$ ll ping/fstab
-rw-rw-r--. 1 hyp hyp 15 3月  20 16:27 ping/fstab
[hyp@localhost ~]$ chmod 440 ping/fstab
[hyp@localhost ~]$ echo "nihao" >> ping/fstab #不能修改
bash: ping/fstab: 权限不够
[hyp@localhost ~]$ rm ping/fstab
rm：是否删除有写保护的普通文件 "ping/fstab"？y
[hyp@localhost ~]$ ll ping/fstab #可以删除
ls: 无法访问ping/fstab: 没有那个文件或目录
```

**umask**

umask是文件权限反向掩码。新建文件比新建文件夹确少可执行权限。所以新建文件夹权限为 `777-umask`，新建文件权限为 `666-umask`。使用 `umask [mode]`更改掩码值（此类设定仅对当前shell进程有效），也可以在创建文件夹时，设置权限（不是rwx-umask）。

```shell
[hyp@localhost ~]$ umask #查看umask值，后三位为属主、属组、其他
0002
[hyp@localhost ~]$ mkdir newdir
[hyp@localhost ~]$ ls -ld newdir
drwxrwxr-x. 2 hyp hyp 6 3月  20 16:31 newdir
[hyp@localhost ~]$ echo "new file" > newfile
[hyp@localhost ~]$ ls -l newfile
-rw-rw-r--. 1 hyp hyp 9 3月  20 16:32 newfile
[hyp@localhost ~]$ mkdir -m 700 newdir2
[hyp@localhost ~]$ ls -ld newdir2
drwx------. 2 hyp hyp 6 3月  20 16:38 newdir2
```

#### 丛属关系管理命令

**chown**

更改文件（夹）的属主，语法：`chown [OPTION]... [OWNER][:[GROUP]] FILE...`。

使用选项-R递归修改。

**chgrp**

更改文件（夹）的属组，语法： `chgrp [OPTION]... GROUP FILE...`。

使用选项-R递归修改。

仅管理员可修改文件的属主和属组

**练习：**

1、新建系统组mariadb, 新建系统用户mariadb, 属于mariadb组，要求其没有家目录，且shell为/sbin/nologin；尝试root切换至用户，查看其命令提示符；

```shell
[hyp@localhost ~]$ sudo useradd  mariadb -s /sbin/nologin -M
[hyp@localhost ~]$ tail -1 /etc/passwd
mariadb:x:1002:1002::/home/mariadb:/sbin/nologin
[hyp@localhost ~]$ ls /home/
hyp
[hyp@localhost ~]$ su root
密码：
[root@localhost hyp]# su mariadb
This account is currently not available.
[root@localhost hyp]# exit
exit
[hyp@localhost ~]$ sudo userdel -r mariadb
userdel：未找到 mariadb 的主目录“/home/mariadb”
```

2、新建GID为5000的组mageedu，新建用户gentoo，要求其家目录为/users/gentoo，密码同用户名；

```shell
[hyp@localhost ~]$ sudo groupadd -g 5000 mageedu
[hyp@localhost ~]$ sudo useradd -d /users/gentoo gentoo
useradd：无法创建目录 /users/gentoo
[hyp@localhost ~]$ tail -1
[hyp@localhost ~]$ tail -1 /etc/passwd
gentoo:x:1002:1002::/users/gentoo:/bin/bash
[hyp@localhost ~]$ sudo passwd gentoo
更改用户 gentoo 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[hyp@localhost ~]$ sudo tail -1 /etc/shadow
gentoo:$6$9rccDYAn$L0A3spOJnaUWjH76doWPmF3J8oLgFPeCWxBl6F3JBT5rm4wIH0sd5rqtdvHkpqEt4i08iqwHxFVCuAczxEASq0:17975:0:99999:7:::
```

3、新建用户fedora，其家目录为/users/fedora，密码同用户名；

```shell
[hyp@localhost ~]$ sudo useradd fedora -d /users/fedora
useradd：无法创建目录 /users/fedora
[hyp@localhost ~]$ sudo passwd fedora
更改用户 fedora 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[hyp@localhost ~]$ tail -1 /etc/passwd
fedora:x:1003:1003::/users/fedora:/bin/bash
[hyp@localhost ~]$ sudo tail -1 /etc/shadow
fedora:$6$seONS9db$6K.fe5EtbCgPlzT78Ix3m6ltDcya.s2/djd.R5xOjyl3jIC5y1LTpLFBG4TfNdFKlDj.7KRmBfU5BS5DqDo4k/:17975:0:99999:7:::
```

4、新建用户www, 其家目录为/users/www；删除www用户，但保留其家目录；

```shell
[hyp@localhost ~]$ sudo useradd www -d /users/www
[hyp@localhost ~]$ tail -1 /etc/passwd
www:x:1004:1004::/users/www:/bin/bash
[hyp@localhost ~]$ sudo userdel www
[hyp@localhost ~]$ ll -d /users/www
drwx------. 3 1004 1004 78 3月  20 17:00 /users/www
```

5、为用户gentoo和fedora新增附加组mageedu; 

```shell
[hyp@localhost ~]$ sudo groupmems -g mageedu -a fedora
[hyp@localhost ~]$ sudo groupmems -g mageedu -a gentoo
[hyp@localhost ~]$ sudo groupmems  -g mageedu -l
fedora  gentoo
```

6、复制目录/var/log至/tmp/目录，修改/tmp/log及其内部的所有文件的属组为mageedu，并让属组对目录本身拥有写权限；

```shell
[hyp@localhost ~]$ sudo mv  /var/log /tmp/
[hyp@localhost ~]$ ls -ld  /tmp/log/
drwxr-xr-x. 16 root root 4096 3月  20 12:59 /tmp/log/
[hyp@localhost ~]$ sudo chgrp -R mageedu  /tmp/log/  #更改组
[hyp@localhost ~]$ ls -ld  /tmp/log/
drwxr-xr-x. 16 root mageedu 4096 3月  20 12:59 /tmp/log/
[hyp@localhost ~]$ sudo chmod g+w /tmp/log/ #更改组权限
[hyp@localhost ~]$ ls -ld  /tmp/log/
drwxrwxr-x. 16 root mageedu 4096 3月  20 12:59 /tmp/log/
```

### 特殊权限

#### SUID、SGID和SBIT

进程是发起此进程用户的代理，因此以此用户的身份和权限完成所有操作；

(1) 判断进程的属主，是否为被访问的文件属主；如果是，则应用属主的权限；否则进入第2步；
(2) 判断进程的属主，是否属于被访问的文件属组；如果是，则应用属组的权限；否则进入第3步；
(3) 应用other的权限；

**SUID**

当s这个标志出现在文件所有者的x权限上时，如/usr/bin/passwd这个文件的权限状态：“-rwsr-xr-x.”，此时就被称为Set UID，简称为SUID。那么这个特殊权限的特殊性的作用是什么呢？
1、SUID权限仅对二进制程序(binary program)有效；
2、执行者对于该程序需要具有x的可执行权限；
3、本权限仅在执行该程序的过程中有效(run-time)；
4、执行者将具有该程序拥有者(owner)的权限。
SUID的目的就是：让本来没有相应权限的用户运行这个程序时，可以访问他没有权限访问的资源。用户运行某程序时，如果此程序拥有SUID权限，那么程序运行为进程时，进程的属主不是发起者，而程序文件自己的属主；

passwd就是一个很鲜明的例子，下面我们就来了解一下这相passwd执行的过程。

我们知道，系统中的用户密码是保存在/etc/shadow中的，而这个文件的权限是----------. （这个权限和以前版本的RHEL也有差别，以前的是-r--------）。其实有没有r权限不重要，因为我们的root用户是拥有最高的权限，什么都能 干了。关键是要把密码写入到/etc/shadow中。我们知道，除了root用户能修改密码外，用户自己同样也能修改密码，为什么没有写入权限，还能修 改密码，就是因为这个SUID功能。

下面就是passwd这个命令的执行过程
1、因为/usr/bin/passwd的权限对任何的用户都是可以执行的，所以系统中每个用户都可以执行此命令。
2、而/usr/bin/passwd这个文件的权限是属于root的。
3、当某个用户执行/usr/bin/passwd命令的时候，就拥有了root的权限了。
4、于是某个用户就可以借助root用户的权力，来修改了/etc/shadow文件了。
5、最后，把密码修改成功。

> 注：这个SUID只能运行在二进制的程序上（系统中的一些命令），不能用在脚本上（script），因为脚本还是把很多的程序集合到一起来执行，而不是脚本自身在执行。同样，这个SUID也不能放到目录上，放上也是无效的。

**SGID**

我们前面讲过，当s这个标志出现在文件所有者的x权限上时，则就被称为Set UID。那么把这个s放到文件的所属用户组x位置上的话，就是SGID。如开头的/usr/bin/wall命令。
那么SGID的功能是什么呢？和SUID一样，只是SGID是获得该程序所属用户组的权限。
这相SGID有几点需要我们注意：
1、SGID对二进制程序有用；
2、程序执行者对于该程序来说，需具备x的权限；
3、SGID主要用在目录上；
理解了SUID，我想SGID也很容易理解。如果用户在此目录下具有w权限的话，若使用者在此目录下建立新文件，则新文件的群组与此目录的群组相同。当目录属组有写权限，且有SGID权限时，那么所有属于此目录的属组，且以属组身份在此目录中新建文件或目录时，新文件的属组不是用户的基本组，而是此目录的属组。

**Sticky Bit**

这个就是针对others来设置的了，和上面两个一样，只是功能不同而已。
SBIT（Sticky Bit）目前只针对目录有效，对于目录的作用是：当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除。对于属组或全局可写的目录，组内的所有用户或系统上的所有用户对在此目录中都能创建新文件或删除所有的已有文件；如果为此类目录设置Sticky权限，则每个用户能创建新文件，且只能删除自己的文件；

最具有代表的就是/tmp目录，任何人都可以在/tmp内增加、修改文件（因为权限全是rwx），但仅有该文件/目录建立者与 root能够删除自己的目录或文件。系统上的/tmp和/var/tmp目录默认均有sticky权限。

> 注：这个SBIT对文件不起作用。

**设置方法**

和我们前面说的rwx差不多，也有两种方式，一种是以字符，一种是以数字。
4 为 SUID ＝ u+s
2 为 SGID ＝ g+s
1 为 SBIT ＝ o+t

**为什么会有大写的S和T**

因为他们这个位置没有了x权限，如果没有了x权限，根据我们上面讲的内容，其实，这个特 殊的权限就相当于一个空的权限，没有意义。也就是说，如果你看到特殊权限位置上变成了大写的了，那么，就说明，这里有问题，需要排除。

参考：[Linux之特殊权限（SUID/SGID/SBIT）](https://www.cnblogs.com/dyh004/p/6378456.html)

#### 文件属性控制

在Linux中，有一些系统文件，对系统的运行有着至关重要的作用，如/etc/fstab等，一般不允许修改，这个时候，我们可以赋予文件/目录`r--------`的权限；然而，还有一个更为简单有效的命令`chattr`可以实现该功能！

特殊权限的要求：

> - 所支持的文件系统包括：`ext2`、`ext3`、`ext4`和`xfs`
>
> - 一般要求内核版本不低于2.2(查看版本的命令如下):
>
>   > uname -a
>   > lsb_release -a
>
> - 不能保护 `/` `/tmp` `/dev` `/var`目录
>
> - chattr 只能由root用户使用

**chattr**



```shell
chattr [ -RVf ] [ -v version ] [ mode ] files...
```

最关键的是在[mode]部分，[mode]部分是由`+-=`和`[ASacDdIijsTtu]`这些字符组合的，这部分是用来控制文件的属性。

> - **+** : 在原有参数设定基础上，追加参数
> - **-** ：在原有参数设定基础上，移除参数
> - **=** ：更新为指定参数设定
> - A ：文件或目录的 atime (access time)不可被修改(modified),可以有效预防例如手提电脑磁盘I/O错误的发生
> - **a** ：即append，设定该参数后，只能向文件中添加数据，而不能删除；
> - b：不更新文件或目录的最后存取时间。
> - c ：即compresse，设定文件是否经压缩后再存储。读取时需要经过自动解压操作。
> - d：将文件或目录排除在倾倒操作之外。
> - **i** ：设定文件不能被删除、改名、设定链接关系，同时不能写入或新增内容；对目录
> - s ：保密性地删除文件或目录，即硬盘空间被全部收回
> - S：即时更新文件或目录。
> - u ：与s相反，当设定为u时，数据内容其实还存在磁盘中，可以用于undeletion.

选项：

```
　-R 递归处理，将指定目录下的所有文件及子目录一并处理。
　　-v<版本编号> 设置文件或目录版本。
　　-V 显示指令执行过程。
　　+<属性> 开启文件或目录的该项属性。
　　-<属性> 关闭文件或目录的该项属性。
　　=<属性> 指定文件或目录的该项属性。
```

**lsattr**

语法：

```shell
lsattr [ -RVadv ] [ files...  ]
```

参数如下：

> - a : 列出目录下的所有文件，包括隐藏文件
> - d : 查看本目录自身的权限
> - -R ：连同子目录的数据一并列出来！

参考：[Linux文件权限与属性详解 之 chattr & lsattr](https://www.cnblogs.com/Jimmy1988/p/7265816.html)

### ACL权限

**什么是ACL**

ACL(Access Control List)，访问控制列表。让不属于任何一个组的用户，只是以单用户的身份被赋予特定权限。ACL可以针对单一用户、单一文件或目录来进行r、w、x的权限设置，对于需要特殊权限的使用状况非常有帮助。

CentOS7上文件系统为**XFS**，**XFS**默认支持ACL。而ext文件系统需要使用下面命令查看：

```
df -Th #找到根目录/的挂载点
dumpe2fs -h <挂载点>  #ext文件系统
xfs_info <挂载点> #xfs文件系统
```

系统默认的是不会安装ACL权限的，因此需要我们自己动手：

- **RPM 包**：
  前提：能够获取到系统安装包
  命令：`rpm -ivh libacl-x.x.xx-x.x acl-x.x.xx-x.x.rpm`
- **yum**:
  前提：主机已经联网，且yum可用
  命令：`yum -y insatll libacl acl`

**ACL文件设置**

- **getfacl**: 获取文件或目录的ACL设置信息
  命令： getfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...

  user:USERNAME:MODE
  group:GROUPNAME:MODE

  参数：

  > -a , --access：显示文件或目录的访问控制列表
  > -d , --default：显示文件或目录的默认（缺省）的访问控制列表
  > -c , --omit-header：不显示默认的访问控制列表
  > -R , --recursive：操作递归到子目录

- **setfacl**: 设置文件或目录的ACL设置信息
  命令：setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...

  ```
  赋权给用户：
  setfacl  -m  u:USERNAME:MODE  FILE...
  赋权级组：
  setfacl  -m  g:GROUPNAME:MODE FILE...
  
  撤销赋权：
  setfacl  -x u:USERNAME  FILE...
  setfacl  -x  g:GROUPNAME  FILE...
  ```

  参数：

  > -m, --modify=acl：修改文件或目录的扩展ACL设置信息
  > -x, --remove=acl：从文件或目录删除一个扩展的ACL设置信息
  > -b, --remove-all：删除所有的扩展的ACL设置信息
  > -k, --remove-default：删除缺省的acl设置信息
  > -n, --no-mask：不要重新计算有效权限。setfacl默认会重新计算ACL mask，除非mask被明确的制定
  > -d, --default：设置默认的ACL设置信息（只对目录有效）
  > -R, --recursive：操作递归到所有子目录和 文件

  

示例：

```shell
[hyp@localhost home]$ sudo setfacl -m u:hyp:rwx git
[hyp@localhost home]$ sudo setfacl -m g:wheel:r-x git
[hyp@localhost home]$ getfacl git
# file: git
# owner: git
# group: git
user::rwx
user:hyp:rwx
group::---
group:wheel:r-x
mask::rwx
other::---
[hyp@localhost home]$ sudo setfacl -x g:wheel git
[hyp@localhost home]$ sudo setfacl -x u:hyp git
[hyp@localhost home]$ getfacl git
# file: git
# owner: git
# group: git
user::rwx
group::---
mask::---
other::---
```

### 管理员权限

一般情况下，很少需要登录root管理员用户，因为其权限过大，为了避免做出不可挽回的操作，应该尽量不使用root用户。但是，普通用户不具有执行管理员命令的权限，所以linux提供了`sudo`命令，这个命令可以使具有管理员权限的普通用户执行管理员命令。

#### sudo

（1) 特征：

- 对用户的执行命令权限进行限制
- 提供了日志记录，可详细记录每个用户具体的操作(http://blog.csdn.net/xyz846/article/details/26406955)
- 临时性的时间戳(一般为5min)，在此期间使用sudo命令，不需要再输入密码
- 配置文件为`/etc/sudoers`,可以使`root`对用户集中管理

（2) 工作流程：

- 当用户执行sudo时，系统寻找/etc/sudoers文件，判断该用户是否具备执行sudo的权限
- 确认用户权限后，让用户输入自身的密码
- 若密码合法，则开始执行sudo后续的命令
- root执行sudo时不需要输入密码
- 自身切换自身也不需要输入密码



sudo [optional]  [argument]:
选项:

> -b：在后台执行指令； -h：显示帮助
> -H：将HOME环境变量设为新身份的HOME环境变量
> -k：结束密码的有效期限，也就是下次再执行sudo时便需要输入密码
> -l：列出目前用户可执行与无法执行的指令
> -p：改变询问密码的提示符号
> -s：执行指定的shell
> -u<用户>：以指定的用户作为新的身份。若不加上此参数，则预设以root作为新的身份；
> -v：延长密码有效期限5分钟；

**Argument**:

> 需要运行的指令

#### 配置

那么如何让用户拥有管理员权限呢？

对sudo权限的配置其实就是修改 /etc/sudoers 文件，有两种方式实现：

> - vim /etc/sudoers
> - **visudo** ： 推荐使用

  方法一： 修改 /etc/sudoers 文件，找到下面一行，把前面的注释（#）去掉

```
## Allows people in group wheel to run all commands
%wheel    ALL=(ALL)    ALL
```

然后修改用户，使其属于root组（wheel），命令如下：

```
#usermod -g wheel tommy
```

修改完毕，现在可以用tommy帐号登录，然后用命令 sudo，即可获得root权限进行操作。

方法二： 修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行，如下所示：

```
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
tommy   ALL=(ALL)     ALL
```

修改完毕，现在可以用tommy帐号登录，然后用命令 sudo ，即可获得root权限进行操作。

方法三： 修改 /etc/passwd 文件，找到如下行，把用户ID修改为 0 ，如下所示：

```
tommy:x:500:500:tommy:/home/tommy:/bin/bash
```

修改后如下

```
tommy:x:0:500:tommy:/home/tommy:/bin/bash
```

保存，用tommy账户登录后，直接获取的就是root帐号的权限。

友情提醒：虽然方法三看上去简单方便，但一般不推荐使用，推荐使用方法二。  

示例：

```shell
[hyp@localhost ~]$ sudo useradd tommy -M
[hyp@localhost ~]$ sudo passwd tommy
更改用户 tommy 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[hyp@localhost ~]$ sudo usermod -g wheel tommy
[hyp@localhost ~]$ su - tommy
密码：
上一次登录：五 3月 22 14:03:09 CST 2019pts/1 上
su: 警告：无法更改到 /home/tommy 目录: 没有那个文件或目录
-bash-4.2$ sudo ls /

我们信任您已经从系统管理员那里了解了日常注意事项。
总结起来无外乎这三点：

    #1) 尊重别人的隐私。
    #2) 输入前要先考虑(后果和风险)。
    #3) 权力越大，责任越大。

[sudo] tommy 的密码：
bin   dev  home  lib64  mnt  proc  run   srv  tmp    usr  web
boot  etc  lib   media  opt  root  sbin  sys  users  var

```

### 文本处理

**wc**

word count，为统计指定文件中的字节数、字数、行数，并将统计结果显示输出。

语法：`wc [选项] 文件...`

选项：

```
-c 统计字节数。
-l 统计行数。
-m 统计字符数。这个标志不能与 -c 标志一起使用。
-w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串。
-L 打印最长行的长度。
```

**cut**

是一个选取命令，以行为单位，用指定分隔符将行切分为若干字段，选取所需要的字段。

语法：`cut [option] files`

选项：

```
-d：用来定义分隔符，默认为tab键，一般与-f配合使用（如果分隔符是空格，必须是两个单引号之间确实有一个空格，是一个哦，不是支持多个）
-f：需要选取的字段，根据-d切分的字段集选取，下标从1开始
-s：表示不包括那些不含分隔符的行，用于去掉注释或者标题一类的信息
-c：以字符为单位进行分割，可以选取指定字符
-b：以字节为单位进行分割，可以选取指定字节，这些字节位置将忽略多字节字符边界（比如：汉字），除非同时指定了-n参数
-n：取消分割多字节字符，只能和-b参数配合使用，即如果字符的最后一个字节落在由-b参数列表指定的范围之内，则该字符将被选出，否则，该字符将被排除。
不难看出上面参数中，-f、-c、-b都是用来表示提取指定范围数据的，这个范围的表示方法如下：
    N：只取第N项
    N-：从第N项一直到行尾
    N-M：从第N项到第M项（包括M项）
    -M：从第一项到第M项（包括M项）
    -：从第一项开始到结束的所有项
```

示例：

```shell
#输出PATH变量第五个路径和后面的路径
[hyp@localhost ~]$ echo $PATH | cut -d: -f5-
/home/hyp/.local/bin:/home/hyp/bin
```

**sort**

将文本文件内容加以排序,sort可针对文本文件的内容，以行为单位来排序，默认是按照ASCII字符排序。

语法： `sort [option] [file]....`

选项：

```
-n：基于数值大小而非字符进行排序；
-t CHAR：指定分隔符；
-k #：用于排序比较的字段；
-r：逆序排序；
-f：忽略字符大小写
-u：重复的行只保留一份；
	复复行：连续且相同；
  +<起始栏位>-<结束栏位>   以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。
```

**uniq**

报告或移除重复的行，不附加任何选项时去除临近的相同项。

语法：`uniq [option] [file|stdin]`

```
  -c, --count           在每行前加上表示相应行目出现次数的前缀编号
  -d, --repeated        只输出重复的行
  -D, --all-repeated[=delimit-method    显示所有重复的行
                        delimit-method={none(default),prepend,separate}
                        以空行为界限
  -f, --skip-fields=N   比较时跳过前N 列
  -i, --ignore-case     在比较的时候不区分大小写
  -s, --skip-chars=N    比较时跳过前N 个字符
  -u, --unique          只显示唯一的行
  -z, --zero-terminated 使用'\0'作为行结束符，而不是新换行
  -w, --check-chars=N   对每行第N 个字符以后的内容不作对照
```

**diff**

用于比较文件的内容，特别是比较两个版本不同的文件以找到改动的地方。diff在命令行中打印每一个行的改动。最新版 本的diff还支持二进制文件。diff程序的输出被称为补丁 (patch)，因为Linux系统中还有一个patch程序，可以根据diff的输出将 a.c的文件内容更新为b.c。diff是svn、cvs、git等版本控制工具不可或缺的一部分。

diff 命令能比较单个文件或者目录内容。如果指定比较的是文件，则只有当输入为文本文件时才有效。以逐行的方式，比较文本文件的异同处。如果指定比较的是目录的 的时候，diff 命令会比较两个目录下名字相同的文本文件。列出不同的二进制文件、公共子目录和只在一个目录出现的文件。

语法：`diff[选项][文件1或目录1][文件2或目录2]`

- FILE1 FILE2 ：源是一个文件，目标也是文件。这两个文件必须是文本文件。以逐行的方式，比较文本文件的异同处。 
  e.g. diff 1.txt 2.txt
- DIR1 DIR2 ：源是一个目录，目标是目录。diff 命令会比较两个目录下名字相同的文本文件，依照字母次序排序，列出不同的二进制文件，列出公共子目录，列出只在一个目录出现的文件。 
  e.g. diff dir1 dir2
- FILE DIR ：源是一个文件，目标是目录。diff命令把源文件与目标目录下的同名文件比较。 
  e.g. diff 1.txt dir2
- DIR FILE ：源是一个目录，目标是文件（不是目录）。源目录下所有文件中与目标文件同名的文件，将用来与目标文件比较。 
  e.g. diff dir1 2.txt

选项：

```
- 　指定要显示多少行的文本。此参数必须与-c或-u参数一并使用。
-b ——  忽略一行中的空字符的区别（例如“Hello World!!” 与 “Hello        World!!”认为是一样的）
-B —— 忽略空白行
-c 　显示全部内文，并标出不同之处。
-i —— 忽略大小写的不同
-a逐行比较文本文件
-u 显示有差异行的前后几行(上下文), 默认是前后各3行, 这样, patch中带有更多的信息.
-p显示代码所在的c函数的信息
-N 选项确保补丁文件将正确地处理已经创建或删除文件的情况。
-r —— 如果diff后面接的目录时，会递归比较子目录中的文件不同
```

diff输出内容有三种格式：

（1）正常格式（normal diff）
（2）上下文格式（context diff）
（3）合并格式（unified diff）

示例：

```shell
[hyp@localhost ~]$ ls
file1  file2
[hyp@localhost ~]$ diff file1 file2 #正常格式
4d3
< 4
[hyp@localhost ~]$ diff -c file1 file2  #上下文格式
*** file1       2019-03-21 20:22:58.144610337 +0800
--- file2       2019-03-21 20:23:21.149312085 +0800
***************
*** 1,7 ****
  1
  2
  3
- 4
  55
  6
  755
--- 1,6 ----
#第一部分的两行，显示两个文件的基本情况：文件名和时间信息。
#"***"表示变动前的文件，"---"表示变动后的文件。
#第二部分是15个星号，将文件的基本情况与变动内容分割开。
#第三部分显示变动前的文件
#文件内容的每一行最前面，还有一个标记位。如果为空，表示该行无变化；
#如果是感叹号（!），表示该行有改动；如果是减号（-），表示该行被删除；
#如果是加号（+），表示该行为新增。
#第四部分显示变动后的文件
#除了变动行（第4行）以外，也是上下文各显示三行，总共显示7行。
[hyp@localhost ~]$ diff -u file1 file2 #合并格式
--- file1       2019-03-21 20:22:58.144610337 +0800
+++ file2       2019-03-21 20:23:21.149312085 +0800
@@ -1,7 +1,6 @@
 1
 2
 3
-4
 55
 6
 755
#第一部分的两行，显示两个文件的基本情况：文件名和时间信息。
#"---"表示变动前的文件，"+++"表示变动后的文件。
#第二部分变动的位置用两个@作为起首和结束前面的"-1,7"分成三个部分：
#减号表示第一个文件（即f1），"1"表示第1行，"7"表示连续7行。
#合在一起，就表示下面是第一个文件从第1行开始的连续7行。
#同样的，"+1,7"表示变动后，成为第二个文件从第1行开始的连续7行。
#第三部分显示变动前的文件
#文件内容的每一行最前面，还有一个标记位。如果为空，表示该行无变化；
#如果是感叹号（!），表示该行有改动；如果是减号（-），表示该行被删除；
#如果是加号（+），表示该行为新增。
#第四部分显示变动后的文件
#除了有变动的那些行以外，也是上下文各显示3行。它将两个文件的上下文，合并显示在一起，所以叫做"合并格式"。
#每一行最前面的标志位，空表示无变动，减号表示第一个文件删除的行，加号表示第二个文件新增的行。

```

**cmp**

diff是以行为单位进行文件的比较，cmp是以字节为单位进行文件的比较。

语法：`cmp [option] filename_1 filename_2 `

选项：

-b —— 将所有的不同的字节的地方都显示出来，若没有-s，则cmp会默认输出第一个发现的的不同点。

备注：使用cmp工具可以比较二进制文件的不同

示例:

```shell
[hyp@localhost ~]$ cat file1
1
2
3
4
55
6
755
8
9
10

[hyp@localhost ~]$ cat file2
1
2
3
55
6
755
8
9
10

[hyp@localhost ~]$ cmp -b file1 file2
file1 file2 不同：第 4 行，第 7 字节为  64 4  65 5
```

**patch**

向文件打补丁。

在项目中，有些模块是开源的，没有源码或者不能改动源码，想要修复、优化里面的Bug，这时就需要用到patch了。制作补丁有两种法法，diff和quilt。

语法：

`patch [OPTIONS] -i /PATH/TO/PATCH_FILE /PATH/TO/OLDFILE`

`patch /PATH/TO/OLDFILE < /PATH/TO/PATCH_FILE`

（1）diff生成补丁
`diff -u file1.c file2.c > file.patch`
(2)打patch
`patch file.c < file.patch`

示例：

```shell
[hyp@localhost ~]$ cat file1
1
2
3
4
5
6
7
8
9
10
[hyp@localhost ~]$ cat file2
1
2
3
4
87
5
6
7
8
9
10
[hyp@localhost ~]$ diff -u file1 file2 > file.patch
[hyp@localhost ~]$ cat file.patch
--- file1       2019-03-21 20:15:00.615187329 +0800
+++ file2       2019-03-21 20:15:28.306738257 +0800
@@ -2,6 +2,7 @@
 2
 3
 4
+87
 5
 6
 7
[hyp@localhost ~]$ patch file.c < file.patch
patching file file.c
Hunk #1 FAILED at 2.
patch: **** Can't reopen file file.c : No such file or directory
[hyp@localhost ~]$ patch file1 < file.patch
patching file file1
[hyp@localhost ~]$ diff file1 file2
```

练习：取出`ip addr show ens33`命令结果中的ip地址；

```shell
[hyp@localhost ~]$ ip addr show ens33 | cut -d ' ' -f6 | head -3 | tail -1 |cut -d'/' -f1
```

