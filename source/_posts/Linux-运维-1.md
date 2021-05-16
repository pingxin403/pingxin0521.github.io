---
title: Linux-文件服务器
date: 2019-06-24 21:55:22
tags: 
 - Linux
 - 运维
category: 
 - Linux
 - 运维
---

### FTP

一般来讲，人们将计算机联网的首要目的就是获取资料，而文件传输是一种非常重要的获取资料的方式。今天的互联网是由几千万台个人计算机、工作站、服务器、小型机、大型机、巨型机等具有不同型号、不同架构的物理设备共同组成的，而且即便是个人计算机，也可能会装有Windows、Linux、UNIX、Mac等不同的操作系统。为了能够在如此复杂多样的设备之间解决问题解决文件传输问题，文件传输协议（FTP）应运而生。

FTP是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用20、21号端口，其中端口20（数据端口）用于进行数据传输，端口21（命令端口）用于接受客户端发出的相关FTP命令与参数。FTP服务器普遍部署于内网中，具有容易搭建、方便管理的特点。而且有些FTP客户端工具还可以支持文件的多点下载以及断点续传技术，因此FTP服务得到了广大用户的青睐。FTP协议的传输拓扑如图

<!--more-->

![image.png](https://i.loli.net/2019/09/16/QNpRbg4EhoXUDq3.png)

FTP服务器是按照FTP协议在互联网上提供文件存储和访问服务的主机，FTP客户端则是向服务器发送连接请求，以建立数据传输链路的主机。FTP协议有下面两种工作模式。

> **主动模式**：FTP服务器主动向客户端发起连接请求。
>
> **被动模式**：FTP服务器等待客户端发起连接请求（FTP的默认工作模式）。

vsftpd（very secure ftp daemon，非常安全的FTP守护进程）是一款运行在Linux操作系统上的FTP服务程序，不仅完全开源而且免费，此外，还具有很高的安全性、传输速度，以及支持虚拟用户验证等其他FTP服务程序不具备的特点。

**安装**

1. centos

   ```
   yum install vsftpd
   ```

2. ubuntu

   ```
   sudo apt install vsftpd
   ```

iptables防火墙管理工具默认禁止了FTP传输协议的端口号，因此在正式配置vsftpd服务程序之前，为了避免这些默认的防火墙策略“捣乱”，还需要清空iptables防火墙的默认策略，并把当前已经被清理的防火墙策略状态保存下来：

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

vsftpd服务程序的主配置文件（/etc/vsftpd/vsftpd.conf或/etc/vsftpd.conf）内容总长度达到123行，但其中大多数参数在开头都添加了井号（#），从而成为注释信息，大家没有必要在注释信息上花费太多的时间。我们可以在grep命令后面添加-v参数，过滤并反选出没有包含井号（#）的参数行（即过滤掉所有的注释信息），然后将过滤后的参数行通过输出重定向符写回原始的主配置文件中：

```
[root@linuxprobe ~]# mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_bak
[root@linuxprobe ~]# grep -v "#" /etc/vsftpd/vsftpd.conf_bak > /etc/vsftpd/vsftpd.conf
[root@linuxprobe ~]# cat /etc/vsftpd/vsftpd.conf
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

vsftpd服务程序常用的参数以及作用

| 参数                                              | 作用                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| listen=[YES\|NO]                                  | 是否以独立运行的方式监听服务                                 |
| listen_address=IP地址                             | 设置要监听的IP地址                                           |
| listen_port=21                                    | 设置FTP服务的监听端口                                        |
| download_enable＝[YES\|NO]                        | 是否允许下载文件                                             |
| userlist_enable=[YES\|NO] userlist_deny=[YES\|NO] | 设置用户列表为“允许”还是“禁止”操作                           |
| max_clients=0                                     | 最大客户端连接数，0为不限制                                  |
| max_per_ip=0                                      | 同一IP地址的最大连接数，0为不限制                            |
| anonymous_enable=[YES\|NO]                        | 是否允许匿名用户访问                                         |
| anon_upload_enable=[YES\|NO]                      | 是否允许匿名用户上传文件                                     |
| anon_umask=022                                    | 匿名用户上传文件的umask值                                    |
| anon_root=/var/ftp                                | 匿名用户的FTP根目录                                          |
| anon_mkdir_write_enable=[YES\|NO]                 | 是否允许匿名用户创建目录                                     |
| anon_other_write_enable=[YES\|NO]                 | 是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限） |
| anon_max_rate=0                                   | 匿名用户的最大传输速率（字节/秒），0为不限制                 |
| local_enable=[YES\|NO]                            | 是否允许本地用户登录FTP                                      |
| local_umask=022                                   | 本地用户上传文件的umask值                                    |
| local_root=/var/ftp                               | 本地用户的FTP根目录                                          |
| chroot_local_user=[YES\|NO]                       | 是否将用户权限禁锢在FTP目录，以确保安全                      |
| local_max_rate=0                                  | 本地用户最大传输速率（字节/秒），0为不限制                   |

#### Vsftpd服务程序

vsftpd作为更加安全的文件传输的服务程序，允许用户以三种认证模式登录到FTP服务器上。

**匿名开放模式**：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。

**本地用户模式**：是通过[Linux系统](https://www.linuxprobe.com/)本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

**虚拟用户模式**：是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

ftp是Linux系统中以命令行界面的方式来管理FTP传输服务的客户端工具。我们首先手动安装这个ftp客户端工具，以便在后续实验中查看结果。

```
[root@linuxprobe ~]# yum install ftp
#或
[root@linuxprobe ~]# apt install ftp
```

**匿名访问模式**

在vsftpd服务程序中，匿名开放模式是最不安全的一种认证模式。任何人都可以无需密码验证而直接登录到FTP服务器。这种模式一般用来访问不重要的公开文件（在生产环境中尽量不要存放重要文件）。当然，如果采用第8章中介绍的防火墙管理工具（如Tcp_wrappers服务程序）将vsftpd服务程序允许访问的主机范围设置为企业内网，也可以提供基本的安全性。

vsftpd服务程序默认开启了匿名开放模式，我们需要做的就是开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。需要注意的是，针对匿名用户放开这些权限会带来潜在危险，我们只是为了在Linux系统中练习配置vsftpd服务程序而放开了这些权限，不建议在生产环境中如此行事。

可以向匿名用户开放的权限参数以及作用

| 参数                        | 作用                               |
| --------------------------- | ---------------------------------- |
| anonymous_enable=YES        | 允许匿名访问模式                   |
| anon_umask=022              | 匿名用户上传文件的umask值          |
| anon_upload_enable=YES      | 允许匿名用户上传文件               |
| anon_mkdir_write_enable=YES | 允许匿名用户创建目录               |
| anon_other_write_enable=YES | 允许匿名用户修改目录名称或删除目录 |

在vsftpd服务程序的主配置文件中正确填写参数，然后保存并退出。还需要重启vsftpd服务程序，让新的配置参数生效。在此需要提醒各位读者，在生产环境中或者在RHCSA、[RHCE](https://www.linuxprobe.com/)、[RHCA](https://www.linuxprobe.com/)认证考试中一定要把配置过的服务程序加入到开机启动项中，以保证服务器在重启后依然能够正常提供传输服务：

```
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
 ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service
```

现在就可以在客户端执行ftp命令连接到远程的FTP服务器了。在vsftpd服务程序的匿名开放认证模式下，其账户统一为anonymous，密码为空。而且在连接到FTP服务器后，默认访问的是/var/ftp目录。我们可以切换到该目录下的pub目录中，然后尝试创建一个新的目录文件，以检验是否拥有写入权限

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Permission denied.
```

系统显示拒绝创建目录！我们明明在前面清空了iptables防火墙策略，而且也在vsftpd服务程序的主配置文件中添加了允许匿名用户创建目录和写入文件的权限啊。建议大家先不要着急往下看，而是自己思考一下这个问题的解决办法，以锻炼您的Linux系统排错能力。

前文提到，在vsftpd服务程序的匿名开放认证模式下，默认访问的是/var/ftp目录。查看该目录的权限得知，只有root管理员才有写入权限。怪不得系统会拒绝操作呢！下面将目录的所有者身份改成系统账户ftp即可（该账户在系统中已经存在），这样应该可以了吧：

```
[root@linuxprobe ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 3 root root 16 Jul 13 14:38 /var/ftp/pub
[root@linuxprobe ~]# chown -Rf ftp /var/ftp/pub
[root@linuxprobe ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 3 ftp root 16 Jul 13 14:38 /var/ftp/pub
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Create directory operation failed.
```

系统再次报错！尽管我们在使用ftp命令登入FTP服务器后，再创建目录时系统依然提示操作失败，但是报错信息却发生了变化。在没有写入权限时，系统提示“权限拒绝”（Permission denied）所以怀疑是权限的问题。但现在系统提示“创建目录的操作失败”（Create directory operation failed），想必各位读者也应该意识到是SELinux服务在“捣乱”了吧。

下面使用getsebool命令查看与FTP相关的SELinux域策略都有哪些：

```
[root@linuxprobe ~]# getsebool -a | grep ftp
ftp_home_dir --> off
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
sftpd_anon_write --> off
sftpd_enable_homedirs --> off
sftpd_full_access --> off
sftpd_write_ssh_home --> off
tftp_anon_write --> off
tftp_home_dir --> off
```

我们可以根据经验（需要长期培养，别无它法）和策略的名称判断出是ftpd_full_access--> off策略规则导致了操作失败。接下来修改该策略规则，并且在设置时使用-P参数让修改过的策略永久生效，确保在服务器重启后依然能够顺利写入文件。

```
[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

> 再次提醒各位读者，在进行下一次实验之前，一定记得将虚拟机还原到最初始的状态，以免多个实验相互产生冲突。

现在便可以顺利执行文件创建、修改及删除等操作了。

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
257 "/pub/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
ftp> exit
221 Goodbye.
```

**本地用户模式**

相较于匿名开放模式，本地用户模式要更安全，而且配置起来也很简单。如果大家之前用的是匿名开放模式，现在就可以将它关了，然后开启本地用户模式。

| 参数                | 作用                                              |
| ------------------- | ------------------------------------------------- |
| anonymous_enable=NO | 禁止匿名访问模式                                  |
| local_enable=YES    | 允许本地用户模式                                  |
| write_enable=YES    | 设置可写权限                                      |
| local_umask=022     | 本地用户模式创建文件的umask值                     |
| userlist_deny=YES   | 启用“禁止用户名单”，名单文件为ftpusers和user_list |
| userlist_enable=YES | 开启用户作用名单文件功能                          |

在vsftpd服务程序的主配置文件中正确填写参数，然后保存并退出。还需要重启vsftpd服务程序，让新的配置参数生效。在执行完上一个实验后还原了虚拟机的读者，还需要将配置好的服务添加到开机启动项中，以便在系统重启自后依然可以正常使用vsftpd服务。

```
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
 ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service
```

按理来讲，现在已经完全可以本地用户的身份登录FTP服务器了。但是在使用root管理员登录后，系统提示如下的错误信息：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): root
530 Permission denied.
Login failed.
ftp>
```

可见，在我们输入root管理员的密码之前，就已经被系统拒绝访问了。这是因为vsftpd服务程序所在的目录中默认存放着两个名为“用户名单”的文件（ftpusers和user_list）。

vsftpd服务程序为了保证服务器的安全性而默认禁止了root管理员和大多数系统用户的登录行为，这样可以有效地避免黑客通过FTP服务对root管理员密码进行暴力破解。如果您确认在生产环境中使用root管理员不会对系统安全产生影响，只需按照上面的提示删除掉root用户名即可。我们也可以选择ftpusers和user_list文件中没有的一个普通用户尝试登录FTP服务器

在采用本地用户模式登录FTP服务器后，默认访问的是该用户的家目录

而且该目录的默认所有者、所属组都是该用户自己，因此不存在写入权限不足的情况。

在配置妥当后再使用本地用户尝试登录下FTP服务器，分别执行文件的创建、重命名及删除等命令。操作均成功！

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.2)
Name (192.168.10.10:root): linuxprobe
331 Please specify the password.
Password:此处输入该用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
257 "/home/linuxprobe/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
ftp> exit
221 Goodbye.
```

**虚拟用户模式**

我们最后讲解的虚拟用户模式是这三种模式中最安全的一种认证模式，当然，因为安全性较之于前面两种模式有了提升，所以配置流程也会稍微复杂一些。

**第1步**：创建用于进行FTP认证的用户数据库文件，**其中奇数行为账户名，偶数行为密码**。例如，我们分别创建出zhangsan和lisi两个用户，密码均为redhat：

```
[root@linuxprobe ~]# cd /etc/vsftpd/
[root@linuxprobe vsftpd]# vim vuser.list
zhangsan
redhat
lisi
redhat
```

但是，明文信息既不安全，也不符合让vsftpd服务程序直接加载的格式，因此需要使用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。

```
[root@linuxprobe vsftpd]#  apt install db-util
[root@linuxprobe vsftpd]# db_load -T -t hash -f vuser.list vuser.db
[root@linuxprobe vsftpd]# file vuser.db
vuser.db: Berkeley DB (Hash, version 9, native byte-order)
[root@linuxprobe vsftpd]# chmod 600 vuser.db
[root@linuxprobe vsftpd]# rm -f vuser.list
```

**第2步**：创建vsftpd服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户。FTP服务用于存储文件的根目录指的是，当虚拟用户登录后所访问的默认位置。

由于Linux系统中的每一个文件都有所有者、所属组属性，例如使用虚拟账户“张三”新建了一个文件，但是系统中找不到账户“张三”，就会导致这个文件的权限出现错误。为此，需要再创建一个可以映射到虚拟用户的系统本地用户。简单来说，就是让虚拟用户默认登录到与之有映射关系的这个系统本地用户的家目录中，虚拟用户创建的文件的属性也都归属于这个系统本地用户，从而避免Linux系统无法处理虚拟用户所创建文件的属性权限。

为了方便管理FTP服务器上的数据，可以把这个系统本地用户的家目录设置为/var目录（该目录用来存放经常发生改变的数据）。并且为了安全起见，我们将这个系统本地用户设置为不允许登录FTP服务器，这不会影响虚拟用户登录，而且还可以避免黑客通过这个系统本地用户进行登录。

```
[root@linuxprobe ~]# useradd -d /var/ftproot -s /sbin/nologin virtual
[root@linuxprobe ~]# ls -ld /var/ftproot/
drwx------. 3 virtual virtual 74 Jul 14 17:50 /var/ftproot/
[root@linuxprobe ~]# chmod -Rf 755 /var/ftproot/
```

**第3步**：建立用于支持虚拟用户的PAM文件。

PAM（可插拔认证模块）是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式。要想把PAM功能和作用完全讲透，至少要一个章节的篇幅才可以（对该主题感兴趣的读者敬请关注本书的进阶篇，里面会详细讲解PAM）。

通俗来讲，PAM是一组安全机制的模块，系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行任何修改。PAM采取了分层设计（应用程序层、应用接口层、鉴别模块层）的思想

![image.png](https://i.loli.net/2019/09/16/4ha5sQB9xXMTdgm.png)

新建一个用于虚拟用户认证的PAM文件vsftpd.vu，其中PAM文件内的“db=”参数为使用db_load命令生成的账户密码数据库文件的路径，但不用写数据库文件的后缀：

```
[root@linuxprobe ~]# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```

**第4步**：在vsftpd服务程序的主配置文件中通过pam_service_name参数将PAM认证文件的名称修改为vsftpd.vu，PAM作为应用程序层与鉴别模块层的连接纽带，可以让应用程序根据需求灵活地在自身插入所需的鉴别功能模块。当应用程序需要PAM认证时，则需要在应用程序中定义负责认证的PAM配置文件，实现所需的认证功能。

例如，在vsftpd服务程序的主配置文件中默认就带有参数pam_service_name=vsftpd，表示登录FTP服务器时是根据/etc/pam.d/vsftpd文件进行安全认证的。现在我们要做的就是把vsftpd主配置文件中原有的PAM认证文件vsftpd修改为新建的vsftpd.vu文件即可。

| 参数                       | 作用                                                        |
| -------------------------- | ----------------------------------------------------------- |
| anonymous_enable=NO        | 禁止匿名开放模式                                            |
| local_enable=YES           | 允许本地用户模式                                            |
| guest_enable=YES           | 开启虚拟用户模式                                            |
| guest_username=virtual     | 指定虚拟用户账户                                            |
| pam_service_name=vsftpd.vu | 指定PAM文件                                                 |
| allow_writeable_chroot=YES | 允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求 |

**第5步**：为虚拟用户设置不同的权限。虽然账户zhangsan和lisi都是用于vsftpd服务程序认证的虚拟账户，但是我们依然想对这两人进行区别对待。比如，允许张三上传、创建、修改、查看、删除文件，只允许李四查看文件。这可以通过vsftpd服务程序来实现。只需新建一个目录，在里面分别创建两个以zhangsan和lisi命名的文件，其中在名为zhangsan的文件中写入允许的相关权限（使用匿名用户的参数）：

```
[root@linuxprobe ~]# mkdir /etc/vsftpd/vusers_dir/
[root@linuxprobe ~]# cd /etc/vsftpd/vusers_dir/
[root@linuxprobe vusers_dir]# touch lisi
[root@linuxprobe vusers_dir]# vim zhangsan
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

然后再次修改vsftpd主配置文件，通过添加user_config_dir参数来定义这两个虚拟用户不同权限的配置文件所存放的路径。为了让修改后的参数立即生效，需要重启vsftpd服务程序并将该服务添加到开机启动项中：

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
guest_enable=YES
guest_username=virtual
allow_writeable_chroot=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd.vu
userlist_enable=YES
tcp_wrappers=YES
user_config_dir=/etc/vsftpd/vusers_dir
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
 ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service
```

**第6步**：设置SELinux域允许策略，然后使用虚拟用户模式登录FTP服务器。相信大家可以猜到，SELinux会继续来捣乱。所以，先按照前面实验中的步骤开启SELinux域的允许策略，以免再次出现操作失败的情况

此时，不但可以使用虚拟用户模式成功登录到FTP服务器，还可以分别使用账户zhangsan和lisi来检验他们的权限。当然，读者在生产环境中一定要根据真实需求来灵活配置参数，不要照搬这里的实验操作。

##### Docker配置FTP

```
$ docker pull fauria/vsftpd 

$ docker run -d -p 21:21 -p 20:20 -p 21100-21110:21100-21110 -v /home/ftp:/home/vsftpd -e FTP_USER=test -e FTP_PASS=123456 -e PASV_ADDRESS=192.168.2.142 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21110 --name vsftpd --restart=always fauria/vsftpd
 
 
#-p进行端口绑定映射
# -v进行文件目录的映射 FTP_UESR 和FTP_PASS如果设定了会在container里面的 
#/etc/vsftpd/virtual_users.txt
 
#PASV_MIN_PORT和PASV_MAX_PORT映射的是被动模式下端口使用范围
 
#PASV_ADDRESS指的的宿主机地址
```

**参数说明：**

- **/home/ftp:/home/vsftpd**：映射 **docker** 容器 **ftp** 文件根目录（冒号前面是宿主机的目录）
- **-p**：映射 **docker** 端口（冒号前面是宿主机的端口）
- **-e FTP_USER=test -e FTP_PASS=test** ：设置默认的用户名密码（都为 **test**）
- **PASV_ADDRESS**：宿主机 **ip**，当需要使用被动模式时必须设置。
- **PASV_MIN_PORT~ PASV_MAX_PORT**：给客服端提供下载服务随机端口号范围，默认 **21100-21110**，与前面的 **docker** 端口映射设置成一样。

如果 CentOS 服务器有防火墙，为了让客户端能够访问 ftp 服务。我们可以关闭防火墙，或者执行如下命令配置 firewall 防火墙策略：

```
firewall-cmd --permanent --add-port=20/tcp
firewall-cmd --permanent --add-port=21/tcp
firewall-cmd --permanent --add-port=21100/tcp
firewall-cmd --permanent --add-port=21101/tcp
firewall-cmd --permanent --add-port=21102/tcp
firewall-cmd --permanent --add-port=21103/tcp
firewall-cmd --permanent --add-port=21104/tcp
firewall-cmd --permanent --add-port=21105/tcp
firewall-cmd --permanent --add-port=21106/tcp
firewall-cmd --permanent --add-port=21107/tcp
firewall-cmd --permanent --add-port=21108/tcp
firewall-cmd --permanent --add-port=21109/tcp
firewall-cmd --permanent --add-port=21110/tcp
firewall-cmd --reload
```

**新建用户文件夹**

```
#首先执行如下命令进入到容器里面：
docker exec -i -t vsftpd bash
#由于前面我们启动的时候设置用户名为 test，已经自动创建对应的用户文件夹（所以下面这个文件夹无需我们再次手动创建）
mkdir /home/vsftpd/test
#为方便演示，在 test 用户文件夹下新建一个 1.txt 文件
echo `uname -a` >> /home/vsftpd/test/1.txt
```

我们可以直接使用浏览器进行访问，地址如下

[ftp://test:123456@192.168.2.142:21](ftp://test:123456@192.168.2.142:21)

**附：增加一个新用户**
前面我们在启动服务的时候就创建了个默认用户 test。如果需要新增一个新用户，假设用户名：hangge，密码：123456，具体操作如下。 

```shell
#首先执行如下命令进入到容器里面
docker exec -i -t vsftpd bash
# 创建新用户的文件夹：
mkdir /home/vsftpd/hangge
# 编辑用户配置文件：
vim /etc/vsftpd/virtual_users.txt
#在文件中添加新用户的用户名和密码,其中奇数行为账户名，偶数行为密码
#保存退出后执行如下命令，把登录的验证信息写入数据库
/usr/bin/db_load -T -t hash -f /etc/vsftpd/virtual_users.txt /etc/vsftpd/virtual_users.db
#最后退出容器，并重启容器可以使用新用户连接 FTP 服务了。
exit
docker restart vsftpd
```

#### SFTP

SFTP 是Secure File Transfer Protocol的缩写，安全文件传送协议。可以为传输文件提供一种安全的加密方法。
SFTP 与 FTP有着几乎一样的语法和功能。

SFTP 为 SSH的一部分，是一种传输档案至 Blogger 伺服器的安全方式。其实在SSH软件包中，已经包含了一个叫作SFTP(Secure File Transfer Protocol的安全文件传输子系统，SFTP本身没

有单独的守护进程，它必须使用SSHD守护进程（端口号默认是22）来完成相应的连接操作，所以从某种意义上来说，SFTP并不像一个服务器程序，而更像是一个客户端程序。

SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。

但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多，如果您对网络安全性要求更高时，可以使用SFTP代替FTP。

Ubuntu系统默认是没有SSH服务的，故要检查SSH服务是否已安装。

```
$ ps -e | grep ssh
 2997 ?        00:00:00 ssh-agent
15230 ?        00:00:00 sshd
```

安装

```
sudo yum install openssh-client openssh-server
#或
sudo apt-get install openssh-client openssh-server
```

**开启SFTP**

开启命令：

```
sudo systemctl start ssh
sudo systemctl enable ssh
```

**ssh配置文件**

sshd服务配置文件默认位置/etc/ssh/sshd_config。

1. 常见的SSH服务器监听的选项如下：

```
 Port 22               //监听的端口号为22
 Protocol 2            //使用SSH V2协议
 ListenAdderss 0.0.0.0   //监听的地址为所有的地址
 UserDNS no         //禁止DNS反向解析
```

2. 常见用户登录控制选项如下：

```
 PermitRootLogin no              // 禁止root用户登录
 PermitEmptyPasswords no    // 禁止空密码用户登录
 LoginGraceTime 2m             // 登录验证时间为2分钟
 MaxAuthTries 6                   //  最大重试次数6次
 AllowUsers steven               // 只允许steven用户登录
 DenyUsers steven               //  不允许登录用户 steven
```

3. 常见登录验证方式如下：

```
 PasswordAuthentication  yes       //启用密码验证
 PubkeyAuthentication    yes         //启用密匙验证
 AuthorsizedKeysFile .ssh/authorized_keys  //指定公钥数据库文件
```

SSH服务启动后，即可远程登陆，登陆命令格式为：ssh 帐号@IP地址，例如：

```
$ ssh user@10.234.5.81
```

其中帐号指的是Ubuntu的登录帐号

**数据传输**

完成SSH服务配置之后即可实现基于SSH的数据传输，最常用方便的指令便是scp，以下是常用scp指令：

```
$ scp -r usr@43.224.34.73:/home/lk   /root  //将远程IP地址为43.224.34.73的usr用户下路径为 /home/lk 的所有文件拷贝到本地 /root 文件夹中

$ scp usr@43.224.34.73:/home/lk/test.jar   /root  //将远程IP地址为43.224.34.73的usr用户下路径为 /home/lk 的test.jar文件拷贝到本地 /root 文件夹中

$ scp -r /root  usr@43.224.34.73:/home/lk    //将本地 /root 中的所有文件拷贝到远程IP地址为43.224.34.73的usr用户下路径为 /home/lk 的文件夹中

$ scp /root/test.jar   usr@43.224.34.73:/home/lk   //将本地 /root 中的test.jar文件拷贝到远程IP地址为43.224.34.73的usr用户下路径为 /home/lk 的文件夹中
```

scp的通用指令格式为：scp [参数] [原路径] [目标路径]

其中-r参数意为：递归复制整个目录

##### 使用Docker快速搭建sftp服务

参考：https://hub.docker.com/r/atmoz/sftp

```
docker pull atmoz/sftp
```

如果你想给sftp配置多个用户可以有两个方式，1、在容器中创建用户并指派权限，2、在宿主机上编写用户文件然后挂载到容器中

```
docker run --name sftp -v /etc/sftp/users.conf:/etc/sftp/users.conf:ro -v /home/sftp:/home  --privileged=true -p 3333:22  --restart=always -d atmoz/sftp
```

- -v /etc/sftp/users.conf:/etc/sftp/users.conf:ro　　将本地的/host/users.conf映射到容器的/etc/sftp/users.conf，并且在容器内为只读
- -v /home/sftp:/home　　将本地/home/sftp目录映射到容器/home下存放上传的文件

　　创建本地 /etc/sftp/users.conf文件

```
foo:123:1001:100
bar:abc:1002:100
baz:xyz:1003:100
```

其中 user:pass:uid:gid　　用户名:密码:用户id:组id

这里创建的用户目录默认组和用户都是root没有权限，需要手动修改一下。

### Samba

在windows网络环境中，主机之间进行文件和打印机共享是通过微软公司自己的SMB/CIFS网络协议实现的。SMB（Server Message Block，服务消息块）和CIFS（Common Internet File System，通用互联网文件系统）协议是微软的私有协议，在Samba项目出现之前，并不能直接与Linux/UNIX系统进行通信。

Samba是著名的开源软件项目之一，它在Linux/UNIX系统中实现了微软的SMB/CIFS网络协议，从而使得跨平台的文件共享变得更加容易。在部署Windows、Linux/UNIX混合平台的企业环境时，选用Samba可以很好地解决不同系统之间的文件互访问题。

```
yum install samba
```

 Samba服务程序中的参数以及作用

| [global]   |                                                              | #全局参数。                                         |
| ---------- | ------------------------------------------------------------ | --------------------------------------------------- |
|            | workgroup = MYGROUP                                          | #工作组名称                                         |
|            | server string = Samba Server Version %v                      | #服务器介绍信息，参数%v为显示SMB版本号              |
|            | log file = /var/log/samba/log.%m                             | #定义日志文件的存放位置与名称，参数%m为来访的主机名 |
|            | max log size = 50                                            | #定义日志文件的最大容量为50KB                       |
|            | security = user                                              | #安全验证的方式，总共有4种                          |
|            | #share：来访主机无需验证口令；比较方便，但安全性很差         |                                                     |
|            | #user：需验证来访主机提供的口令后才可以访问；提升了安全性    |                                                     |
|            | #server：使用独立的远程主机验证来访主机提供的口令（集中管理账户） |                                                     |
|            | #domain：使用域控制器进行身份验证                            |                                                     |
|            | passdb backend = tdbsam                                      | #定义用户后台的类型，共有3种                        |
|            | #smbpasswd：使用smbpasswd命令为系统用户设置Samba服务程序的密码 |                                                     |
|            | #tdbsam：创建数据库文件并使用pdbedit命令建立Samba服务程序的用户 |                                                     |
|            | #ldapsam：基于LDAP服务进行账户验证                           |                                                     |
|            | load printers = yes                                          | #设置在Samba服务启动时是否共享打印机设备            |
|            | cups options = raw                                           | #打印机的选项                                       |
| [homes]    |                                                              | #共享参数                                           |
|            | comment = Home Directories                                   | #描述信息                                           |
|            | browseable = no                                              | #指定共享信息是否在“网上邻居”中可见                 |
|            | writable = yes                                               | #定义是否可以执行写入操作，与“read only”相反        |
| [printers] |                                                              | #打印机共享参数                                     |
|            | comment = All Printers                                       |                                                     |
|            | path = /var/spool/samba                                      | #共享文件的实际路径(重要)。                         |
|            | browseable = no                                              |                                                     |
|            | guest ok = no                                                | #是否所有人可见，等同于"public"参数。               |
|            | writable = no                                                |                                                     |
|            | printable = yes                                              |                                                     |

#### 配置共享资源

Samba服务程序的主配置文件与前面学习过的Apache服务很相似，包括全局配置参数和区域配置参数。全局配置参数用于设置整体的资源共享环境，对里面的每一个独立的共享资源都有效。区域配置参数则用于设置单独的共享资源，且仅对该资源有效。创建共享资源的方法很简单

| 参数                                                  | 作用                       |
| ----------------------------------------------------- | -------------------------- |
| [database]                                            | 共享名称为database         |
| comment = Do not arbitrarily modify the database file | 警告用户不要随意修改数据库 |
| path = /home/database                                 | 共享目录为/home/database   |
| public = no                                           | 关闭“所有人可见”           |
| writable = yes                                        | 允许写入操作               |

**第1步**：创建用于访问共享资源的账户信息。在RHEL 7系统中，Samba服务程序默认使用的是用户口令认证模式（user）。这种认证模式可以确保仅让有密码且受信任的用户访问共享资源，而且验证过程也十分简单。不过，只有建立账户信息数据库之后，才能使用用户口令认证模式。另外，Samba服务程序的数据库要求账户必须在当前系统中已经存在，否则日后创建文件时将导致文件的权限属性混乱不堪，由此引发错误。

pdbedit命令用于管理SMB服务程序的账户信息数据库，格式为“pdbedit [选项] 账户”。在第一次把账户信息写入到数据库时需要使用-a参数，以后在执行修改密码、删除账户等操作时就不再需要该参数了。pdbedit命令中使用的参数以及作用如表

| 参数      | 作用                   |
| --------- | ---------------------- |
| -a 用户名 | 建立Samba用户          |
| -x 用户名 | 删除Samba用户          |
| -L        | 列出用户列表           |
| -Lv       | 列出用户详细信息的列表 |

**第2步**：创建用于共享资源的文件目录。在创建时，不仅要考虑到文件读写权限的问题，而且由于/home目录是系统中普通用户的家目录，因此还需要考虑应用于该目录的SELinux安全上下文所带来的限制。在前面对Samba服务程序配置文件中的注释信息进行过滤时，这些过滤的信息中就有关于SELinux安全上下文策略的说明，我们只需按照过滤信息中有关SELinux安全上下文策略中的说明中给的值进行修改即可。修改完毕后执行restorecon命令，让应用于目录的新SELinux安全上下文立即生效。

**第3步**：设置SELinux服务与策略，使其允许通过Samba服务程序访问普通用户家目录。执行getsebool命令，筛选出所有与Samba服务程序相关的SELinux域策略，根据策略的名称（和经验）选择出正确的策略条目进行开启即可：

**第4步**：在Samba服务程序的主配置文件中，根据表所提到的格式写入共享信息。在原始的配置文件中，[homes]参数为来访用户的家目录共享信息，[printers]参数为共享的打印机设备。

```
[root@linuxprobe ~]# vim /etc/samba/smb.conf 
[global]
 workgroup = MYGROUP
 server string = Samba Server Version %v
 log file = /var/log/samba/log.%m
 max log size = 50
 security = user
 passdb backend = tdbsam
 load printers = yes
 cups options = raw
[database]
 comment = Do not arbitrarily modify the database file
 path = /home/database
 public = no
 writable = yes
```

**第5步**：Samba服务程序的配置工作基本完毕。接下来重启smb服务（Samba服务程序在Linux系统中的名字为smb）并清空iptables防火墙，然后就可以检验配置效果了。

```
[root@linuxprobe ~]# systemctl restart smb
[root@linuxprobe ~]# systemctl enable smb
 ln -s '/usr/lib/systemd/system/smb.service' '/etc/systemd/system/multi-user.target.wants/smb.service'
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

#### Docker部署Samba

samba 镜像地址：https://hub.docker.com/r/dperson/samba

```
docker pull dperson/samba
```

如果开启了 Iptables，则要分别开放 TCP 139、445 和 UDP 137、138 端口：

```
sudo iptables -I INPUT -p tcp --dport 139 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 445 -j ACCEPT

sudo iptables -I INPUT -p udp --dport 137 -j ACCEPT
sudo iptables -I INPUT -p udp --dport 138 -j ACCEPT
```

启动容器

```
docker run -it -p 139:139 -p 445:445 --name samba -d --rm  \
 -v /home/samba/txl/tp5/web:/mount \
 dperson/samba \
 -u "samba;123456" \
 -s "samba;/mount/;yes;no;yes;all;all;all" \
 -w "WORKGROUP" \
 -g "force user= samba" \
 -g "guest account= samba"
```

 -v  代表需要映射的目录， -u  代表目录的owner， -g  代表修改 smb.conf 配置文件的 global 配置。

用户名samba，密码123456

### NFS网络文件系统

如果大家觉得Samba服务程序的配置太麻烦，而且恰巧需要共享文件的主机都是Linux系统，刘遄老师非常推荐大家在客户端部署NFS服务来共享文件。NFS（网络文件系统）服务可以将远程Linux系统上的文件共享资源挂载到本地主机的目录上，从而使得本地主机（Linux客户端）基于TCP/IP协议，像使用本地主机上的资源那样读写远程Linux系统上的共享文件。

NFS服务器可以让PC将网络中的NFS服务器共享的目录挂载到本地端的文件系统中，而在本地端的系统中来看，那个远程主机的目录就好像是自己的一个磁盘分区一样，在使用上相当便利；

NFS一般用来存储共享视频，图片等静态数据。

![image.png](https://i.loli.net/2019/09/16/jHG37Nc1mnDSdWF.png)

当我们在NFS服务器设置好一个共享目录/home/public后，其他的有权访问NFS服务器的NFS客户端就可以将这个目录挂载到自己文件系统的某个挂载点，这个挂载点可以自己定义，如上图客户端A与客户端B挂载的目录就不相同。并且挂载好后我们在本地能够看到服务端/home/public的所有数据。如果服务器端配置的客户端只读，那么客户端就只能够只读。如果配置读写，客户端就能够进行读写。挂载后，NFS客户端查看磁盘信息命令：#df –h。

既然NFS是通过网络来进行服务器端和客户端之间的数据传输，那么两者之间要传输数据就要有想对应的网络端口，NFS服务器到底使用哪个端口来进行数据传输呢？基本上NFS这个服务器的端口开在2049,但由于文件系统非常复杂。因此NFS还有其他的程序去启动额外的端口，这些额外的用来传输数据的端口是随机选择的，是小于1024的端口；既然是随机的那么客户端又是如何知道NFS服务器端到底使用的是哪个端口呢？这时就需要通过远程过程调用（Remote Procedure Call,RPC）协议来实现了！

**RPC与NFS通讯原理：**

因为NFS支持的功能相当多，而不同的功能都会使用不同的程序来启动，每启动一个功能就会启用一些端口来传输数据，因此NFS的功能对应的端口并不固定，客户端要知道NFS服务器端的相关端口才能建立连接进行数据传输，而RPC就是用来统一管理NFS端口的服务，并且统一对外的端口是111，RPC会记录NFS端口的信息，如此我们就能够通过RPC实现服务端和客户端沟通端口信息。PRC最主要的功能就是指定每个NFS功能所对应的port number,并且通知客户端，记客户端可以连接到正常端口上去。

那么RPC又是如何知道每个NFS功能的端口呢？

首先当NFS启动后，就会随机的使用一些端口，然后NFS就会向RPC去注册这些端口，RPC就会记录下这些端口，并且RPC会开启111端口，等待客户端RPC的请求，如果客户端有请求，那么服务器端的RPC就会将之前记录的NFS端口信息告知客户端。如此客户端就会获取NFS服务器端的端口信息，就会以实际端口进行数据的传输了。

注意：在启动NFS SERVER之前，首先要启动RPC服务（即portmap服务，下同）否则NFS SERVER就无法向RPC服务区注册，另外，如果RPC服务重新启动，原来已经注册好的NFS端口数据就会全部丢失。因此此时RPC服务管理的NFS程序也要重新启动以重新向RPC注册。特别注意：一般修改NFS配置文档后，是不需要重启NFS的，直接在命令执行systemctl reload nfs或exportfs –rv即可使修改的/etc/exports生效

**NFS客户端和NFS服务器通讯过程：**

![image.png](https://i.loli.net/2019/09/16/E8YZRiKcvdNDQ24.png)



1. 首先服务器端启动RPC服务，并开启111端口

2. 服务器端启动NFS服务，并向RPC注册端口信息

3. 客户端启动RPC（portmap服务），向服务端的RPC(portmap)服务请求服务端的NFS端口

4. 服务端的RPC(portmap)服务反馈NFS端口信息给客户端。

5. 客户端通过获取的NFS端口来建立和服务端的NFS连接并进行数据的传输。

#### 部署

安装NFS服务，需要安装两个软件，分别是：

- RPC主程序：rpcbind

  NFS 其实可以被视为一个 RPC 服务，因为启动任何一个 RPC 服务之前，我们都需要做好 port 的对应 (mapping) 的工作才行，这个工作其实就是『 rpcbind 』这个服务所负责的！也就是说， 在启动任何一个 RPC 服务之前，我们都需要启动 rpcbind 才行！ (在 CentOS 5.x 以前这个软件称为 portmap，在 CentOS 6.x 之后才称为 rpcbind 的！)。

- NFS主程序：nfs-utils

  就是提供 rpc.nfsd 及 rpc.mountd 这两个 NFS daemons 与其他相关 documents 与说明文件、执行文件等的软件！这个就是 NFS 服务所需要的主要软件。


**NFS的相关文件：**

- 主要配置文件：/etc/exports
  这是 NFS 的主要配置文件了。该文件是空白的，有的系统可能不存在这个文件，主要手动建立。NFS的配置一般只在这个文件中配置即可。
- NFS 文件系统维护指令：/usr/sbin/exportfs
  这个是维护 NFS 分享资源的指令，可以利用这个指令重新分享 /etc/exports 变更的目录资源、将 NFS Server 分享的目录卸除或重新分享。
- 分享资源的登录档：/var/lib/nfs/*tab
  在 NFS 服务器的登录文件都放置到 /var/lib/nfs/ 目录里面，在该目录下有两个比较重要的登录档， 一个是 etab ，主要记录了 NFS 所分享出来的目录的完整权限设定值；另一个 xtab 则记录曾经链接到此 NFS 服务器的相关客户端数据。
- 客户端查询服务器分享资源的指令：/usr/sbin/showmount
  这是另一个重要的 NFS 指令。exportfs 是用在 NFS Server 端，而 showmount 则主要用在 Client 端。showmount 可以用来察看 NFS 分享出来的目录资源。


```
[root@localhost ~]# yum install -y rpc-bind nfs-utils   
#安装nfs服务
[root@localhost ~]# yum install -y rpcbind
#安装rpc服务
```

**启动服务和设置开启启动**

```
[root@localhost ~]# systemctl start rpcbind    #先启动rpc服务
[root@localhost ~]# systemctl enable rpcbind   #设置开机启动
[root@localhost ~]# systemctl start nfs-server nfs-secure-server      
#启动nfs服务和nfs安全传输服务
[root@localhost ~]# systemctl enable nfs-server nfs-secure-server
[root@localhost /]# firewall-cmd --permanent --add-service=nfs
success   #配置防火墙放行nfs服务
[root@localhost /]# firewall-cmd  --reload 
success
```

**第三步：配置共享文件目录，编辑配置文件：**

首先创建共享目录，然后在/etc/exports配置文件中编辑配置即可。

```
[root@localhost /]# mkdir /public
#创建public共享目录
[root@localhost /]# vi /etc/exports
	/public 192.168.245.0/24(ro)
	/protected 192.168.245.0/24（rw）
[root@localhost /]# systemctl reload nfs 
#重新加载NFS服务，使配置文件生效
```

配置文件说明：

格式： 共享目录的路径 允许访问的NFS客户端（共享权限参数）

如上，共享目录为/public , 允许访问的客户端为192.168.245.0/24网络用户，权限为只读。

请注意，NFS客户端地址与权限之间没有空格。

NFS输出保护需要用到kerberos加密（none，sys，krb5，krb5i，krb5p），格式sec=XXX

none：以匿名身份访问，如果要允许写操作，要映射到nfsnobody用户，同时布尔值开关要打开，setsebool nfsd_anon_write 1

sys：文件的访问是基于标准的文件访问，如果没有指定，默认就是sys， 信任任何发送过来用户名

krb5：客户端必须提供标识，客户端的表示也必须是krb5，基于域环境的认证

krb5i：在krb5的基础上做了加密的操作，对用户的密码做了加密，但是传输的数据没有加密

krb5p：所有的数据都加密

**用于配置NFS服务程序配置文件的参数：**

![image.png](https://i.loli.net/2019/09/16/pjC4OXgSFfuU5qd.png)



#### Docker部署

在使用docker云平台（这个可以包含公有的云平台，也可以是自己内部搭建的云平台）过程中，同一个应用系统一般都涉及多个镜像，比如一般都会涉及到Web逻辑的镜像、Nginx等代理镜像及数据库镜像，而且它们都会使用到存储，将内部的可变数据持久化到存储上，这样保证容器重启或删除后，数据依然存在。
而如果同一个系统启动的镜像太多了，如果每个镜像申请一个存储目录，那么需要申请的存储目录就可能会很多，管理起来可能会不太方便。
本文针对如下情况，描述尝试的方案：

- 多个镜像均需挂载的目录，进行存储数据；
- 想只申请一份挂载的存储空间，想建立其子目录以适应不同的镜像挂载；
- 在k8s云平台中，又限制只能直接挂载PVC存储，不能对PVC下的子目录进行挂载
- 当前无可用的公共NAS或NFS的挂载点

因此，需要启动一个docker容器，用来运行NFS服务器，将同一个PVC的存储分别映射出不同的子目录供不同镜像挂载，以实现：对于镜像来说，可直接挂载自己的目录，不会对其它目录造成影响，其容器内部目录结构也不需要做任何改变。

##### 单挂载点

使用的镜像为itsthenetwork/nfs-server-alpine:latest，可直接pull到本地，启动容器方法：

```
$ docker pull  itsthenetwork/nfs-server-alpine:latest
$ docker run -d --name nfs_server --privileged -p 2050:2049 \
-v /本地需共享的主机目录:/nfsroot \
-h nfsserver \
-e SHARED_DIRECTORY=/nfsroot \
itsthenetwork/nfs-server-alpine:latest

docker run -d --name nfs_server --privileged -p 2050:2049 \
-v /home/nfs:/nfsroot \
-h nfsserver \
-e SHARED_DIRECTORY=/nfsroot \
itsthenetwork/nfs-server-alpine:latest
```

其中-p的端口2050为容器映射出来主机的端口；
-h表示为本容器设定一个主机名，这样在其他容器中可使用主机名的方式访问（有可能容器在重启后其IP会变化）；
-e为设置本容器的环境变更，表示本容器对外提供的根为【/】的挂载点。

**客户端容器连接**

假设已经存在另外一些客户端的镜像，本文以alpine linux为例：

```
$ docker pull alpine:latest 
$ docker run -it --rm   --name nfs_client --privileged  \
--link nfs_server:nfsserver \
alpine:latest /bin/sh
```

上述命令只是启动临时的容器，作为验证使用，若需长期启动容器并挂载，需要将-it换成-d，更多具体的容器操作，可查阅相关资料。

其中--link表示本容器会访问到容器名为nfs_server且其主机名为nfsserver的容器，添加此参数后会在本容器的/etc/hosts文件中添加一行

```css
172.xxx.xxx.xx      nfsserver nfsserver nfs_server
```

还需注意，此容器需添加--privileged参数表示容器的用户有SYS_ADMIN权限，此时才可顺利的执行mount命令。

然后就是在容器中进行mount了：

```shell
mount -t nfs4 nfsserver:/ /media
```

将nfs的根目录挂载到本地的/home/vol01目录中。

注意：
此时只能挂载mfsserver:/这个根，不能再往后写其子目录，即使子目录已经建立好，因为当前服务器中/etc/exports中只指明了这个根是对外提供挂载的，而没有指明其子目录。
/etc/exports文件声明对外挂载的目录，文件内容示例如下：

```shell
/nfsroot *(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
```

若单节点的搭建还存在问题，可再参考[官方github说明](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fsjiveson%2Fnfs-server-alpine)

**小结**

上述安装过程展现了各目录之间的前后关系（假设S表示服务容器，C表示客户容器）：

- S中启动时已经挂载PVC存储到【/nfsroot】目录
- S启动时会使用/etc/exports文件，表示本地目录是【/nfsroot】，对外挂载时是默认的【/】
- C端mount将S的【/】映射到本地【/home/vol01】
- 最终实现【存储目录】->【/nfsroot】->【/】->【/media】的转换

#### 多挂载点

参考文章[NFS (Network File System) 服务器共享多个目录](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Fx_i_y_u_e%2Farticle%2Fdetails%2F44924651)，结合[github在线提问](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fsjiveson%2Fnfs-server-alpine%2Fissues%2F13)，即我们可有2种方法来实现多目录共享：

- 重新编译镜像方式：
  下载nfs-server-alpine的github源码，按在线提问方法修改confd/tmpl/exports.tmpl文件，注意其中的fsid=0参数只能有1个，表示对外提供的根服务目录，然后重新编译镜像并启动容器。
- 直接修改容器内容：
  可在启动nfs-server-alpine容器后，进入容器中手动修改/etc/exports文件，然后再将容器反向保存为镜像，这样以后启动新容器时就还是能提供多目录的挂载点了。示例：

```
/nfsroot *(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
/nfsroot/vol1 *(rw,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
/nfsroot/vol2 *(rw,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
/nfsroot/vol3 *(rw,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
```

但无论使用哪种方案，最终目标都是生成合适的/etc/exports文件。

**客户端的挂载**

客户端挂载时，可使用如下方式：

```shell
mount -t nfs4 nasserver:/vol1 /home/vol_1/
```

即目录关系为：
【存储】->【/nfsroot】->此目录下新手工建了vol1到vol3子目录->【exports文件中/nfsroot/vol1到vol3】->【客户端/vol1到vol3】->【客户端/home/vol_1/或其他本地目录】

**总结**

至此，所有的工作就已经完成了。
经验就是只要弄明白单文件挂载，多文件挂载也就很轻松了。

### docker 搭建 OpenLDAP

镜像：https://github.com/osixia/docker-openldap

**什么是 LDAP？**

LDAP是轻量目录访问协议，英文全称是Lightweight Directory Access Protocol，一般都简称为LDAP。LDAP的核心规范可参见RFC定义：[LDAPman RFC page](http://www.ldapman.org/ldap_rfcs.html)

LDAP目录以树状的层次结构来存储数据。如果你对自顶向下的DNS树或UNIX文件的目录树比较熟悉，也就很容易掌握LDAP目录树这个概念了。就象DNS的主机名那样，LDAP目录记录的标识名（Distinguished Name，简称DN）是用来读取单个记录，以及回溯到树的顶部。

**为什么用 OpenLDAP？**

用LDAP的原因，主要可以总结为三点：轻，快，好。

轻，表现在搭建简单，开放的Internet标准，支持跨平台的Internet协议。与市场上和开源领域的大多数产品兼容。配置简单，重复开发和对接的成本低。

快，读的速度快。目录是一个为查询、浏览和搜索而优化的数据库，它成树状结构组织数据，类似文件目录一样。目录数据库和关系数据库不同，它有优异的读性能（但写性能差，并且没有事务处理、回滚等复杂功能，不适于存储修改频繁的数据。）

好，动态，灵活，易扩展。

那么为什么用OpenLDAP呢？当然是因为免费而方便咯。

**怎么用 OpenLDAP？**

1. 使用yum install等等传统搭建方式

2. 使用DOCKER。吹爆docker，虽然LDAP已经很轻了，但是在windows系统上，选择docker仍然不失为非常便利的方式

**搭建 OpenLDAP**

```
$ docker pull osixia/openldap 
```

##### 创建几个目录

/mnt/sdb1/ldap/data/
/mnt/sdb1/ldap/conf/slapd.d
/mnt/sdb1/ldap/conffile
data目录保存数据
slapd.d目录保存配置
conffile目录用来与容器交换文件

**运行容器**

```
$ docker run --restart=always \
-v /mnt/sdb1/ldap/data/:/var/lib/ldap \
-v /mnt/sdb1/ldap/conf/slapd.d:/etc/ldap/slapd.d \
-v /mnt/sdb1/ldap/conffile/:/home/ldap/conffile \
-p 389:389 -p 636:636 \
--name openldap -d osixia/openldap --loglevel debug
```

**生成密码**

```
$ docker exec -it openldap slappasswd
New password: 
Re-enter new password: 
{SSHA}ZbbjXA1zx3Mng6UL/1FCBusX49bRT6vJ
```

输入两次密码会生成加密后的密码
记下最后一行生成的加密串

**修改管理员密码**

在/mnt/sdb1/ldap/conffile下创建一个文件chrootpw.ldif

```undefined
dn: olcDatabase={0}config,cn=config
changetype: modify
replace:olcRootPW
olcRootPW: {SSHA}ZbbjXA1zx3Mng6UL/1FCBusX49bRT6vJ
```

最后一行为刚才生成的加密串
执行编辑好的 chrootpw.ldif 文件

```bash
docker exec -it openldap ldapadd -Y EXTERNAL -H ldapi:/// -f /home/ldap/conffile/chrootpw.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```

最后一行表示修改成功

**添加基础的 Schema**

例如添加corba

```cpp
docker exec -it openldap  ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/corba.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=corba,cn=schema,cn=config"
```

基础的Schema在容器目录/etc/ldap/schema/下，
根据需要添加，重复添加可能会报错

**在 LDAP 数据库中设置根域和数据库超级管理员**

在/mnt/sdb1/ldap/conffile下创建一个文件domain-dbadmin.ldif

```csharp
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=***,dc=net

dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=***,dc=net

dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}ZbbjXA1zx3Mng6UL/1FCBusX49bRT6vJ

dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcAccess
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcAccess
olcAccess: to *
            by self read
            by users none
            by dn.base="cn=Manager,dc=shsbnu,dc=net" write
            by anonymous auth
```

olcRootDN是管理员
olcRootPW是管理员密码，也需要添入加密串，可以相同或重新生成
olcRootDN 和 olcSuffix 需要修改成相应的域地址
olcAccess 格式保持一致，最好不要使用tab
执行该 ldif 文件

```bash
docker exec -it openldap ldapmodify -Y EXTERNAL -H ldapi:/// -f /home/ldap/conffile/domain-dbadmin.ldif
```

**数据恢复**

原数据导出

```jsx
slapcat > /opt/ldap/ldapdbak.ldif
```

将导出后的数据复制到容器所在服务器的/mnt/sdb1/ldap/conffile目录下
数据导入

```bash
docker exec -it openldap slapadd -l /home/ldap/conffile/ldapdbak.ldif
```





### 参考

1. [详解Ubuntu下ssh服务的安装与登陆（ssh远程登陆）](https://www.jb51.net/article/97589.htm)
2. [scp命令详解](https://www.cnblogs.com/likui360/p/6011769.html)
3. [使用Vsftpd服务传输文件](https://www.linuxprobe.com/chapter-11.html)
4. [OpenLDAP 在 CentOS 7 上的极速搭建教程](https://www.jianshu.com/p/dc7112873e68)
5. [osixia](https://github.com/osixia) / [docker ](https://github.com/osixia)[-openldap](https://github.com/osixia/docker-openldap)
6. [ladp配置](https://www.jianshu.com/p/aefb071e4b45)