---
title: Linux-程序管理
date: 2019-03-22 21:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

前面我们介绍了文件的相关命令，本章我们就来学一下程序管理。Linux的思想之一就是一切皆文件，而程序包就是由许多个文件组成的，主要有二进制程序、库文件、配置文件、帮助文件，命令则是二进制文件。组织成为一个或有限几个“包”文件。

程序包操作一般包括安装、升级、卸载、查询、校验。

<!--more-->

安装可以说是将所需要的文件拷贝到给定的目录路径下，并且，记录下软件信息和依赖文件。例如，配置文件放在/etc目录下，二进制文件放在PATH环境变量包含的路径下，帮助文件放置位置/usr/share/man下，程序文档放置在/usr/share/doc文件夹下，库文件放置在/usr/lib或/usr/lib64文件夹下，程序数据一般放在/var/lib，其余的请参考[FHS](http://www.pathname.com/fhs/)。

```
#$PATH
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
/home/USERNAME/.local/bin
/home/USERNAME/bin
```

升级则是更改一些文件，在系统上更新记录的版本和依赖文件。

卸载就是删除软件的依赖软件和包括文件以及安装记录。

查询则是通过工具对软件的信息和文档进行查询（一般都是该工具记录的信息）。

校验则是通过对软件包生成散列值来查看软件是否被修改。



一般程序包分为源码包、二进制包。

源码包，顾名思义，是由源代码组成的包，一般是指由C/C++语言编写的文本文件和文档组成，包名命名格式为`name-VERSION.tar.gz`，VERSION由`major.minor.release`组成，这种包获取后需要解压、编译之后，才能进行安装。编译耗时，且容易出错，但适合多个平台，不由包管理器管理。

二进制包是针对某一系统或某一平台的安装包。对于不同平台，也就是不同arch，有这么几种：

```
32位：x86：i386,i486,i586,i686
64位：x86_64:x64,x86_64,amd64
powerpc:ppc
各平台都可用，跟平台无关：noarch
```

对于不同操作系统，由于程序包管理器的不同，包也不同。对于Debian系列，使用的是dpt、dpkg，程序包后缀名为 `.deb`;对于redhat系列（包括CentOS），使用的是rpm（rpm is package manager），程序包后缀是 `.rpm`；等等。

一般rpm包的命名格式为：`name-VERSION-release.arch.rpm`，而由于linux的思想之一是所有的功能都是有一个个小型、单一用途的程序实现。所以程序包开发者一般会做拆分工作，将程序包根据功能，例如：devel, utils, libs等等，进行拆分，以减少耦合性。

但是，当程序调用别的程序、或对别的程序包有所依赖的话，二进制包的安装便复杂了许多。因此，人们创建了可以自动解决依赖关系的前端工具。常用的有：yum（rhel系列系统上rpm包管理器的前端工具）、apt-get (apt-cache)（deb包管理器的前端工具）、zypper（suse的rpm管理器前端工具）、dnf（Fedora 22+系统上rpm包管理器的前端工具）。能在CentOS 7上使用的有yum和dnf。

还有一些命令来发现依赖关系。查看二进制程序所依赖的库文件： `ldd /PATH/TO/BINARY_FILE`管理查看本机装载的库文件：ldconfig ，/sbin/ldconfig -p:显示本机已缓存的所有可用库文件名以及文件路径映射关系，配置文件：/etc/ld.so.conf ,/etc/ld.so.conf.d/*.conf，缓存文件：/etc/ld.so.cache

获取程序包的途径：
(1) 系统发行版的光盘或官方的文件服务器（或镜像站点）：
	http://mirrors.aliyun.com, 
	http://mirrors.sohu.com,
	http://mirrors.163.com 
(2) 项目的官方站点
(3) 第三方组织：
	(a) EPEL
	(b) 搜索引擎
		http://pkgs.org
		http://rpmfind.net 
		http://rpm.pbone.net 
(4) 自动动手，丰衣足食 

建议：检查其合法性
	来源合法性；
	程序包的完整性；

### 二进制包安装

我们首先从二进制包开始学习。二进制报的安装离不开程序包管理器，其功能是将编译好的应用程序的各组成文件打包成一个或几个程序包文件，从而更方便地实现程序包的安装、升级、卸载和查询等管理操作。程序包管理器一般存储以下程序信息：

1、程序包的组成清单（每个程序包都单独实现），有文件清单、安装或卸载时运行的脚本
2、数据库（公共），包括程序包的名称和版本、依赖关系、功能说明、安装生成的各文件的文件路径及校验码信息等等。

#### rpm

CentOS系统上使用rpm命令来管理程序包，rpm的常用功能有安装、升级、卸载、查询和校验、数据库维护等。语法：`rpm  [OPTIONS]  [PACKAGE_FILE]`。

基本命令如下：

```
安装：-i, --install
升级：-U, --update, -F, --freshen
卸载：-e, --erase
查询：-q, --query
校验：-V, --verify
数据库维护：--builddb, --initdb
```

下面来详细说明：

1. **安装**

   语法：`rpm {-i|--install} [install-options] PACKAGE_FILE ...`

   一般安装使用 `rpm  -ivh  PACKAGE_FILE ...`。

   其中-v输出详细信息，-vv输出更详细信息，这两是GENERAL OPTIONS，任何操作都可用。

   ```
   install-options（用在安装时）：
   -h：hash marks输出进度条；每个#表示2%的进度；
   --test：测试安装，检查并报告依赖关系及冲突消息等；
   --nodeps：忽略依赖关系；不建议；
   --replacepkgs：重新安装
   
   注意：rpm可以自带脚本；
   	四类：--noscripts
   		preinstall：安装过程开始之前运行的脚本，%pre ， --nopre
   		postinstall：安装过程完成之后运行的脚本，%post , --nopost
   		preuninstall：卸载过程真正开始执行之前运行的脚本，%preun, --nopreun 
   		postuninstall：卸载过程完成之后运行的脚本，%postun , --nopostun
   		
   --nosignature：不检查包签名信息，不检查来源合法性；
   --nodigest：不检查包完整性信息；
   ```

   示例：

   ```shell
   [hyp@localhost ~]$ sudo rpm -ivh --test /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm
   [sudo] hyp 的密码：
   准备中...                          ################################# [100%]
   [hyp@localhost ~]$ rpm -q zsh
   未安装软件包 zsh
   [hyp@localhost ~]$ sudo rpm -ivh  /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm
   准备中...                          ################################# [100%]
   正在升级/安装...
      1:zsh-5.0.2-31.el7                 ################################# [100%]
   [hyp@localhost ~]$ rpm -q zsh
   zsh-5.0.2-31.el7.x86_64
   [hyp@localhost ~]$ sudo rpm -ivh --replacepkgs /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm
   准备中...                          ################################# [100%]
   正在升级/安装...
      1:zsh-5.0.2-31.el7                 ################################# [100%]
   [hyp@localhost ~]$ rpm -q zsh
   zsh-5.0.2-31.el7.x86_64
   ```

2. 升级

   语法： `pm {-U|--upgrade} [install-options] PACKAGE_FILE ...`

   ​	或 `rpm {-F|--freshen} [install-options] PACKAGE_FILE ...`

   其中-U表示升级或安装，-F表示升级。

   常用语法：`rpm  -Uvh PACKAGE_FILE ...`或 `rpm  -Fvh PACKAGE_FILE ...`。

   其他选项（除了上面给出的install options）：

   ```
   --oldpackage：降级；
   --force：强制升级；
   ```

   > 注意：
   >
   > (1) 不要对内核做升级操作；Linux支持多内核版本并存，因此，直接安装新版本内核；
   >  (2) 如果某原程序包的配置文件安装后曾被修改过，升级时，新版本的程序提供的同一个配置文件不会覆盖原有版本的配置文件，而是把<u>新版本的配置文件重命名</u>(FILENAME.rpmnew)后提供；

   示例：

   ```
   #假设升级Apache httpd
   #查看自己安装的版本
   [hyp@localhost ~]$ rpm -q httpd
   httpd-2.4.6-88.el7.centos.x86_64
   #下载新包：
   
   ```

3. 卸载

   语法：`rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--test] PACKAGE_NAME ...`

   选项：

   ```
   --allmatches：卸载所有匹配指定名称的程序包的各版本；
   --nodeps：忽略依赖关系
   --test：测试卸载，dry run模式
   ```

   示例:

   ```
   
   ```

4. 查询

   语法：`rpm {-q|--query} [select-options] [query-options]`

   [select-options]

   ```
   PACKAGE_NAME：查询指定的程序包是否已经安装，及其版本；
   -a, --all：查询所有已经安装过的包；
   -f  FILE：查询指定的文件由哪个程序包安装生成；
   
   -p, --package PACKAGE_FILE：用于实现对未安装的程序包执行查询操作；
   
   --whatprovides CAPABILITY：查询指定的CAPABILITY由哪个程序包提供；
   --whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖；
   ```

   [query-options]

   ```
   --changelog：查询rpm包的changlog；
   -l, --list：程序安装生成的所有文件列表；
   -i, --info：程序包相关的信息，版本号、大小、所属的包组，等；
   -c, --configfiles：查询指定的程序包提供的配置文件；
   -d, --docfiles：查询指定的程序包提供的文档；
   --provides：列出指定的程序包提供的所有的CAPABILITY；
   -R, --requires：查询指定的程序包的依赖关系；
   --scripts：查看程序包自带的脚本片断；
   ```

   用法示例：

   ```shell
   #-qi  PACKAGE, -qf FILE, -qc PACKAGE, -ql PACKAGE, -qd PACKAGE
   #-qpi  PACKAGE_FILE, -qpl PACKAGE_FILE, -qpc PACKAGE_FILE, ...
   [hyp@localhost ~]$ rpm -qi zsh #查看已安装程序的详细信息
   Name        : zsh
   Version     : 5.0.2
   Release     : 31.el7
   Architecture: x86_64
   Install Date: 2019年03月23日 星期六 10时51分41秒
   Group       : System Environment/Shells
   Size        : 5854390
   License     : MIT
   Signature   : RSA/SHA256, 2018年11月12日 星期一 22时49分55秒, Key ID 24c6a8a7f4a80eb5
   Source RPM  : zsh-5.0.2-31.el7.src.rpm
   Build Date  : 2018年10月31日 星期三 00时48分17秒
   Build Host  : x86-01.bsys.centos.org
   Relocations : (not relocatable)
   Packager    : CentOS BuildSystem <http://bugs.centos.org>
   Vendor      : CentOS
   URL         : http://zsh.sourceforge.net/
   Summary     : Powerful interactive shell
   Description :
   The zsh shell is a command interpreter usable as an interactive login
   shell and as a shell script command processor.  Zsh resembles the ksh
   shell (the Korn shell), but includes many enhancements.  Zsh supports
   command line editing, built-in spelling correction, programmable
   command completion, shell functions (with autoloading), a history
   mechanism, and more.
   [hyp@localhost ~]$ rpm -qf /bin/zsh #查看文件所属包
   zsh-5.0.2-31.el7.x86_64
   [hyp@localhost ~]$ rpm -qc zsh #查看已安装程序所联系的配置文件
   /etc/skel/.zshrc
   /etc/zlogin
   /etc/zlogout
   /etc/zprofile
   /etc/zshenv
   /etc/zshrc
   [hyp@localhost ~]$ rpm -qd zsh #查看已安装程序文档位置
   [hyp@localhost ~]$ rpm -qd zsh #查看已安装程序所有联系文件位置
   [hyp@localhost ~]$ rpm -qpi /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm #查看未安装的包的信息
   #以上查询命令加上p就可以查看未安装程序包的信息
   ```

5. 校验

   语法：`rpm {-V|--verify} [select-options] [verify-options]`

   [verify-options]

   ```
   S file Size differs
   M Mode differs (includes permissions and file type)
   5 digest (formerly MD5 sum) differs
   D Device major/minor number mismatch
   L readLink(2) path mismatch
   U User ownership differs
   G Group ownership differs
   T mTime differs
   P caPabilities differ
   ```

   包来源合法性验正和完整性验正

   ```
   获取并导入信任的包制作者的密钥：
   对于CentOS发行版来说：rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   验正：
   		(1) 安装此组织签名的程序时，会自动执行验正；
   		(2) 手动验正：rpm -K PACKAGE_FILE
   ```

   示例：

   ```shell
   [hyp@localhost ~]$ rpm -K /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm
   /media/CentOS/Packages/zsh-5.0.2-31.el7.x86_64.rpm: rsa sha1 (md5) pgp md5 确定
   ```

6. 数据库

   rpm管理器数据库路径：/var/lib/rpm/，查询操作：通过此处的数据库进行；

   获取帮助：
   			CentOS 6：man rpm
   			CentOS 7：man rpmdb

   	rpm {--initdb|--rebuilddb} [--dbpath DIRECTORY] [--root DIRECTORY]
   	--initdb：初始化数据库，当前无任何数据库可实始化创建一个新的；当前有时不执行任何操作；
   	--rebuilddb：重新构建，通过读取当前系统上所有已经安装过的程序包进行重新创建；

#### yum

存储了众多rpm包，以及包的相关的元数据文件（放置于特定目录下：repodata）；
yum是rpm的前端应用，安装流程：检查安装包--》检查依赖--》下载所有需要包--》依次安装。

可以使用的文件服务器：（ftp）ftp://、（网络）http://、（网络文件系统）nfs://、（本地）<u>file:///</u>

配置文件有两种，一种是`/etc/yum.conf`，为所有仓库提供公共配置；另一种是`/etc/yum.repos.d/*.repo`，为仓库的指向提供配置。

repo文件配置为：

```
[repositoryID] #不要重复 
name=Some name for this repository #命名
baseurl=url://path/to/repository/ #文件服务器位置
enabled={1|0} #是否可用
gpgcheck={1|0} #是否进行gpg校验
gpgkey=URL #如果进行校验，gpg文件的位置
enablegroups={1|0} #是否支持组安装
failovermethod={roundrobin|priority} #默认为：roundrobin，意为随机挑选；
cost=  #默认为1000
```

yum名令的语法：`yum [options] [command] [package ...]`。

常用命令：

```
显示仓库列表：
	repolist [all|enabled|disabled]

显示程序包：
	list
		# yum list [all | glob_exp1] [glob_exp2] [...]
		# yum list {available|installed|updates} [glob_exp1] [...]

安装程序包：
	install package1 [package2] [...]

	reinstall package1 [package2] [...]  (重新安装)

升级程序包：
	update [package1] [package2] [...]

	downgrade package1 [package2] [...] (降级)

检查可用升级：
	check-update

卸载程序包：
	remove | erase package1 [package2] [...]

查看程序包information：
	info [...]

查看指定的特性(可以是某文件)是由哪个程序包所提供：
	provides | whatprovides feature1 [feature2] [...]

清理本地缓存：
	clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]

构建缓存：
	makecache

搜索：
	search string1 [string2] [...]

	以指定的关键字搜索程序包名及summary信息；

查看指定包所依赖的capabilities：
	deplist package1 [package2] [...]

查看yum事务历史：
	history [info|list|packages-list|packages-info|summary|addon-info|redo|undo|rollback|new|sync|stats]

安装及升级本地程序包：
	* localinstall rpmfile1 [rpmfile2] [...]
	   (maintained for legacy reasons only - use install)
	* localupdate rpmfile1 [rpmfile2] [...]
	   (maintained for legacy reasons only - use update)

包组管理的相关命令：
	* groupinstall group1 [group2] [...]
	* groupupdate group1 [group2] [...]
	* grouplist [hidden] [groupwildcard] [...]
	* groupremove group1 [group2] [...]
	* groupinfo group1 [...]
```

选项：

```
--nogpgcheck：禁止进行gpg check；
 -y: 自动回答为“yes”；
 -q：静默模式；
 --disablerepo=repoidglob：临时禁用此处指定的repo；
 --enablerepo=repoidglob：临时启用此处指定的repo；
 --noplugins：禁用所有插件；
```

示例：

```shell
[hyp@localhost ~]$ yum info httpd
已加载插件：fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * c7-media:
6741 packages excluded due to repository priority protections
已安装的软件包
名称    ：httpd
架构    ：x86_64
版本    ：2.4.6
发布    ：88.el7.centos
大小    ：9.4 M
源    ：installed
来自源：c7-media
简介    ： Apache HTTP Server
网址    ：http://httpd.apache.org/
协议    ： ASL 2.0
描述    ： The Apache HTTP Server is a powerful, efficient, and extensible
         : web server.
[hyp@localhost ~]$ yum provides /etc/yum.conf
已加载插件：fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * c7-media:
6741 packages excluded due to repository priority protections
yum-3.4.3-161.el7.centos.noarch : RPM package installer/updater/manager
源    ：c7-media
匹配来源：
文件名    ：/etc/yum.conf



yum-3.4.3-161.el7.centos.noarch : RPM package installer/updater/manager
源    ：@anaconda
匹配来源：
文件名    ：/etc/yum.conf

[hyp@localhost ~]$ yum deplist /media/CentOS/Packages/httpd-2.4.6-88.el7.centos.x86_64.rpm
已加载插件：fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * c7-media:
6741 packages excluded due to repository priority protections
软件包：httpd.x86_64 2.4.6-88.el7.centos
   依赖：/bin/sh
   provider: bash.x86_64 4.2.46-31.el7
   依赖：/etc/mime.types
   provider: mailcap.noarch 2.1.41-2.el7
   依赖：/usr/sbin/groupadd
   provider: shadow-utils.x86_64 2:4.1.5.1-25.el7
   依赖：/usr/sbin/useradd
   provider: shadow-utils.x86_64 2:4.1.5.1-25.el7
   依赖：httpd-tools = 2.4.6-88.el7.centos
   provider: httpd-tools.x86_64 2.4.6-88.el7.centos
   依赖：libapr-1.so.0()(64bit)
   provider: apr.x86_64 1.4.8-3.el7_4.1
   依赖：libaprutil-1.so.0()(64bit)
   provider: apr-util.x86_64 1.5.2-6.el7
   依赖：libc.so.6()(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libc.so.6(GLIBC_2.14)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libc.so.6(GLIBC_2.2.5)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libc.so.6(GLIBC_2.3)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libc.so.6(GLIBC_2.3.4)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libc.so.6(GLIBC_2.4)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libcrypt.so.1()(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libdb-5.3.so()(64bit)
   provider: libdb.x86_64 5.3.21-24.el7
   依赖：libdl.so.2()(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libexpat.so.1()(64bit)
   provider: expat.x86_64 2.1.0-10.el7_3
   依赖：liblua-5.1.so()(64bit)
   provider: lua.x86_64 5.1.4-15.el7
   依赖：libm.so.6()(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libpcre.so.1()(64bit)
   provider: pcre.x86_64 8.32-17.el7
   依赖：libpthread.so.0()(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libpthread.so.0(GLIBC_2.2.5)(64bit)
   provider: glibc.x86_64 2.17-260.el7
   依赖：libselinux.so.1()(64bit)
   provider: libselinux.x86_64 2.5-14.1.el7
   依赖：libsystemd-daemon.so.0()(64bit)
   provider: systemd-libs.x86_64 219-62.el7
   依赖：libsystemd-daemon.so.0(LIBSYSTEMD_DAEMON_31)(64bit)
   provider: systemd-libs.x86_64 219-62.el7
   依赖：libz.so.1()(64bit)
   provider: zlib.x86_64 1.2.7-18.el7
   依赖：rtld(GNU_HASH)
   provider: glibc.x86_64 2.17-260.el7
   依赖：system-logos >= 7.92.1-1
   provider: centos-logos.noarch 70.0.6-3.el7.centos
   依赖：systemd-units
   provider: systemd.x86_64 219-62.el7

```



**使用本地yum源**

1. 挂载光盘到某目录，如/media/cdrom、/media/CentOS等等

   ```shell
   mount -r -t iso9660 /dev/cdrom /media/cdrom
   ```

2. 更改repo配置文件

   ```shell
   [hyp@localhost ~]$ vi /etc/yum.repos.d/CentOS-Media.repo
   [c7-media]
   name=CentOS-$releasever - Media
   baseurl=file:///media/CentOS/
           file:///media/cdrom/
           file:///media/cdrecorder/
   gpgcheck=1
   enabled=1 #更改为1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   
   ```

3. 更改优先级

   ```shell
   [hyp@localhost ~]$ sudo yum install yum-plugin-priorities
   #然后更改repo文件
   [hyp@localhost ~]$ vi /etc/yum.repos.d/CentOS-Media.repo #添加priority=1
   [hyp@localhost ~]$ vi /etc/yum.repos.d/CentOS-Base.repo #添加priority=2，所有repository下
   ```

4. 生成缓存

   ```shell
   [hyp@localhost ~]$ sudo yum  clean all
   [hyp@localhost ~]$ sudo yum  makecache
   ```

5. 测试

   ```shell
   #测试安装httpd
   [hyp@localhost ~]$ sudo yum remove  httpd
   [hyp@localhost ~]$ sudo yum install  httpd #可以发现源变成了c7-media
   ```

#### dnf

DNF(Dandified Yum)是新一代的RPM软件包管理器。
DNF包管理器克服了YUM包管理器的一些瓶颈，提升了包括用户体验，内存占用，依赖分析，运行速度等多方面的内容。
DNF使用RPM,libsolv和hawkey库进行包管理操作，Fedora22已经默认使用DNF。

DNF并未默认安装在RHEL或CentOS7系统中，但可以在使用YUM的同时使用DNF。

1. 安装epel-release依赖：`yum install epel-release` 或者 `yum install epel-release -y`
2. 安装DNF包：`yum install dnf` 或者 `yum install dnf -y`

配置文件位于 `/etc/dnf/dnf.conf`。



Add the following settings in [main] section of /etc/dnf/dnf.conf, and save the file.

```
proxy=http://<ip address>:<port>
proxy_username=<username>
proxy_password=<password>
```

dnf命令和yum命令基本相同。

### 源码包安装

#### 源码包编译安装

源码包安装过程：源代码 --> 预处理 --> 编译(gcc) --> 汇编 --> 链接 --> 执行

源代码组织格式是多文件，文件中的代码之间，很可能存在跨文件依赖关系；

构建工具：

C、C++： make (configure --> Makefile.in --> makefile)
  java: maven

C代码编译安装三步骤：

```
./configure：
	(1) 通过选项传递参数，指定启用特性、安装路径等；执行时会参考用户的指定以及Makefile.in文件生成makefile；
	(2) 检查依赖到的外部环境；
make：
	根据makefile文件，构建应用程序；
make install
```

开发工具：
    	autoconf: 生成configure脚本
    	automake：生成Makefile.in

> 建议：安装前查看INSTALL，README

开源程序源代码的获取：
    		

```
			官方自建站点：
    			apache.org (ASF)
    			mariadb.org
    			...
    		代码托管：
    			SourceForge
    			Github.com
    			code.google.com
```

编译C源代码：
    前提：提供开发工具及开发环境
    开发工具：make, gcc等
    开发环境：开发库，头文件
    glibc：标准库

    通过“包组”提供开发组件
    CentOS 6: "Development Tools", "Server Platform Development"
第一步：configure脚本
	选项：指定安装位置、指定启用的特性

	--help: 获取其支持使用的选项
		选项分类：
			安装路径设定：
				--prefix=/PATH/TO/SOMEWHERE: 指定默认安装位置；默认为/usr/local/
				--sysconfdir=/PATH/TO/SOMEWHERE：配置文件安装位置；
	
			System types:
	
			Optional Features: 可选特性
				--disable-FEATURE
				--enable-FEATURE[=ARG]
	
			Optional Packages: 可选包
				--with-PACKAGE[=ARG]
				--without-PACKAGE

第二步：make

第三步：make install

完成后配置

安装后的配置：
(1) 导出二进制程序目录至PATH环境变量中；
		编辑文件/etc/profile.d/NAME.sh
			export PATH=/PATH/TO/BIN:$PATH

(2) 导出库文件路径
	编辑/etc/ld.so.conf.d/NAME.conf
		添加新的库文件所在目录至此文件中；

​	让系统重新生成缓存：
​		ldconfig [-v]

(3) 导出头文件
	基于链接的方式实现：
		ln -sv 

(4) 导出帮助手册
	编辑/etc/man.config文件
		添加一个MANPATH

**示例**：

```shell
#下载nginx源码：http://120.52.51.15/nginx.org/download/nginx-1.14.2.tar.gz
[hyp@localhost ~]$ wget http://120.52.51.15/nginx.org/download/nginx-1.14.2.tar.gz
[hyp@localhost ~]$ tar zxf nginx-1.14.2.tar.gz
[hyp@localhost ~]$ cd  nginx-1.14.2
[hyp@localhost nginx-1.14.2]$ ./configure  --prefix=/usr/local/nginx
[hyp@localhost nginx-1.14.2]$ sudo make
[hyp@localhost nginx-1.14.2]$ sudo make install
```

#### RPM包打包制作

**制作流程**

1. 安装rpmbuild软件包， `yum -y install rpm-build`。rpm-build通过rpmbuild命令根据本地源码包，通过spec文件中的规则就可以把源码包制作成rpm包。
2. 创建目录，`mkdir -p ~/rpmbuild/{BUILD,SPECS,RPMS, SOURCES,SRPMS}`,建议使用普通用户，在用户家目录中创建
3. 确定好制作的对象，是源码包编译打包还只是一些库文件打包
4. 编写SPEC文件
5. 开始制作

**创建目录**

手动建立工作目录

```
mkdir -p ~/rpmbuild{BUILD,RPMS,S{OURCE,PEC,RPM}S}
```

或者安装rpmdevtools辅助工具包来自动生成rpmbuild目录

```
yum install -y rpmdevtools
rpmdev-setuptree  # 执行此命令来自动生成
```

**目录详解**

| 目录    | 说明                               | macros宏名  |
| ------- | ---------------------------------- | ----------- |
| BUILD   | 编译rpm包的临时目录                | %_builddir  |
| RPMS    | 存放由rpmbuild最终制作好的二进制包 | %_rpmdir    |
| SOURCES | 所有源代码和补丁文件的存放目录     | %_sourcedir |
| SPECS   | 存放SPEC文件的目录(重要)           | %_specdir   |
| SPRMS   | 最终生成的二进制源码包所在目录     | %_srcrpmdir |

------

**注意：** 关于rpmbuild默认工作路径的确定，通常由在/usr/lib/rpm/macros这个文件里的一个叫做%_topdir的宏变量来定义，你也可以使用rpmbuild –showrc | grep _topdir 来进行查看。如果想更改这个目录名，rpm官方并不推荐直接更改这个目录，而是在用户家目录下建立一个名为.rpmmacros的隐藏文件(注意前面的点不能少，这是Linux下隐藏文件的常识)，然后在里面重新定义%_topdir，指向一个新的目录名。这样就可以满足某些“高级”用户的差异化需求了。通常情况下.rpmmacros文件里一般只有一行内容 %_topdir $HOME/newfile

   **准备源码**

下载httpd最新源码，放置SOURCES目录下。

```
[hyp@localhost ~]$ cd ~/rpmbuild/SOURCES/
[hyp@localhost SOURCES]$ sudo wget https://www-eu.apache.org/dist/httpd/httpd-2.4.38.tar.gz

```

**创建spec文件**

spec文件是整个rpm包制作的核心，它的作用如同源码编译程序时的Makefile文件一样。

spec文件包含建立一个rpm包必要的信息，包括哪些文件是包的一部分以及它们安装在哪个目录等等信息。

**注意：spec文件必须由普通用户创建，并且强烈建议使用vi或者vim命令创建。在新建一个spec文件时，系统会默认创建一个spec文件模版。只是该模版是空的，如果没有填写内容的话，是无法保存该文件的。**

下面我们就开始讲解spec文件的相关选项，spec文件内容一般分为如下几个部分：

定义rpm包的信息、定义源码包、定义rpm包的依赖关系、打包前的工作、编译并安装rpm包、安装之后生成的文件、安装前后需要执行的脚本、软件变更日志。

1. rpm包信息

rpm包信息，主要定义用户查询rpm包信息时所显示的内容。它包含rpm包的功能描述、软件版本、版权信息和软件授权类型等等。

详细信息如下：

Name定义该rpm包的名字，必须要填写。

Version定义该rpm包的版本号，建议和源码包的名称保持一致。

Release定义rpm本身的版本号，使用默认值即可。

Summary定义关于该rpm包的一些介绍。

%description定义关于该rpm包的一些描述信息。

Group标识软件包所属类型。

License软件授权类型，比如GPL、Commercial、Shareware。

URL定义软件作者的主页。

rpm包信息中最重要的是NVR，也就是name、version、release。因为最后生成的rpm包的名称就是根据这三项来的。

rpm名称形式，如下：name-version-release.rpm。

2. 定义源码包

Source0用来定义制作rpm包时所需要的源码包。如果制作rpm包时，有多个源码包，那么使用source和数字混合，比如：

source0: tbsys-src.tar.gz

source1: tbnet-src.tar.gz

source2: tair-2.1.0-src.tar.gz

**注意：Source0必须要填写，而且填写的名字必须是和下载源码包名称要一模一样，还要注意只有tar.gz的源码包，才能制作rpm包。**

3. 定义rpm包的依赖关系

rpm包在制作过程中会依赖基本库，而rpm包在安装时有时也需要其他软件包。这些我们都可以通过以下选项进行控制。

BuildRequires定义制作rpm包时，所依赖的基本库。该选项可有可无。

Requires定义安装该rpm包时，所依赖的软件包。该选项可有可无。

> 注意：
>
> 在这里要重点说明一点，Requires定义所依赖的软件包，在进行yum安装时的情况。
>
> 我们在使用yum安装软件A时，yum会在下载完A的rpm包后，对该rpm包进行检查（rpm包中会给出安装该rpm包安装时，所依赖的基础库和软件）。
>
> 如果检查出，A的安装还要依赖软件B，那么此时yum就会自动下载并安装B。B安装完毕后，就会继续安装A。如果是内网yum源的话，我们只需要把B放在内网yum源即可。
>
> 如果检查出，A的安装不需要其他软件的支持，那么yum会自动安装A。

4. 编译并安装rpm包

这一步是非常重要，类似与源码安装的的./configure、make、make install。主要包括%build、%install等选项。如下：

%build定义编译软件包时的操作

%install定义安装软件包，使用默认值即可。

BuildRoot定义安装或编译时使用的虚拟目录，建议使用默认值即可。如下：

%(mktemp-ud%{_tmppath}/%{name}-%{version}-%{release}-XXXXXX)

该参数非常重要，因为在生成rpm包的过程中，执行make install时就会把软件安装到上述的路径中。在打包的时候，同样依赖虚拟目录为根目录进行操作。

5. 安装之后生成的文件

rpm包在进行安装时，会创建相关的目录及文件，我们就可以在此定义。

%files定义rpm包安装时创建的相关目录及文件。

**在该选项中%defattr (-,root,root)一定要注意。它是指定安装文件的属性，分别是(mode,owner,group)，-表示默认值，对文本文件是0644，可执行文件是0755。**

6. 安装前后需要执行的脚本

%prep指定rpm包安装前执行的脚本。在对软件进行打包前，我们还进行其他操作。比如解压tar.gz文件。%prep主要与%setup –q命令配合使用，建议使用默认值即可。

%post指定rpm包安装后执行的脚本。我们在安装完毕rpm包后，执行软件初始化的动作，就可以通过%post来达到目的。比如：apache在安装后，将apachectl拷贝成httpd等操作。默认spec模版文件不存在此选项。

%preun指定rpm包卸载前执行的脚本，该选项主要用于软件升级的时候会执行。默认spec模版文件不存在此选项。

%postun指定rpm包卸载后执行的脚本。默认spec模版文件不存在此选项。

7. 软件变更日志

%changelog主要用于软件的变更日志。该选项可有可无。

如果使用%changelog选项的话，一定要以*开头，以- -结尾。时间格式为，如下：

\* 星期 月 日 年 XXX

\--

示例为：

\* Tue Mar 03 2015 ilanni2.2.27

\--

**spec文件示例**

```shell
[hyp@localhost SPECS]$ cat httpd.spec
Name: httpd
Version: 2.4.38
Release:        1%{?dist}
Summary: compiled from 2.4.38 by pingxin

Group: System Environment/Daemons
License: GPL
URL: https://px.serveblog.net
Source0:httpd-2.4.38.tar.gz


Requires: gcc, gcc-c++, openssl-devel

%description

Apache web server. Compiled from 2.4.38 by pingxin

%prep
%setup -q


%build
./configure --prefix=/usr/local/httpd --enable-so --enable-rewrite --enable-cgi --enable-ssl --enable-charset-lite --enable-suexec --with-suexec-caller=daemon --with-suexec-docroot=/usr/local/httpd/htdocs --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util  --with-pcre=/usr/local/pcre

make %{?_smp_mflags}


%install
rm -rf %{buildroot}
make install DESTDIR=%{buildroot}

%clean

rm -rf %{buildroot}

%files

%defattr(-,root,root,-)

/usr/local/httpd/bin/*

/usr/local/httpd/build/*

/usr/local/httpd/cgi-bin/*

%config /usr/local/httpd/conf/*

/usr/local/httpd/error/*

/usr/local/httpd/htdocs/*

/usr/local/httpd/icons/*

/usr/local/httpd/include/*

/usr/local/httpd/lib/*

%dir /usr/local/httpd/logs

%doc /usr/local/httpd/man/*

%doc /usr/local/httpd/manual/*

/usr/local/httpd/modules/*

%post

cp /usr/local/httpd/bin/apachectl /etc/init.d/httpd

sed -i '1a # chkconfig: 2345 85 15' /etc/init.d/httpd

sed -i '2a # description: apache web server' /etc/init.d/httpd

chkconfig --add httpd

%preun

/etc/init.d/httpd stop

chkconfig --del httpd

%changelog

* 2019年 03月 23日 星期六 pingxin<m13839441583@163.com> 2.4.38

--
```

**生成RPM包**

直接运行：

```
[hyp@localhost ~]$ rpmbuild -bb rpmbuild/SPECS/httpd.spec
```

如果出现configure错误，请参看：https://blog.csdn.net/linghao00/article/details/7926458

如果出现RPM 构建错误，请参看：https://blog.csdn.net/u014007037/article/details/78727526

生成的包位于`~/rpmbuild/RPMS/x86_64/`。

参考：[烂泥：Linux源码包制作RPM包之Apache](https://www.cnblogs.com/ilanni/p/4312581.html)

