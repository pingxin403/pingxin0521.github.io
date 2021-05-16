---
title: Linux-介绍
date: 2019-03-11 16:55:22
tags:
 - Linux
category:
 - Linux
 - 基础
---

### 前言

现代社会，充满了对电子商品的依赖，而对计算机的依赖为重。随着微机和个人电脑的普及，计算机逐渐开始从实验室走向了学校和公司，并且走向家庭。然而，这些不能脱离操作系统（Operating System）的出现，脱离OS的帮助，计算机入门的门槛还会是那么高，就算电脑的价格再便宜，计算机也不会普及如此迅速。

现如今，个人电脑的操作系统主要有三种：Windows、Mac OS、Linux。 其中Windows的家庭占有率最高，因为其入门简单，价格低，可以在不同电脑上安装运行，但也因为其占用空间大、性能差，饱受诟病；Mac OS是苹果公司所开发的系统，只能在苹果的电脑上安装运行，价格贵，但界面优良，交互性好；Linux操作系统，属于类Unix操作系统，现在几乎所有的公司都在服务器上使用Linux操作系统，其性能高、占用空间小，但入门较难，交互式体验较差，适合单一复杂性任务。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16r3y3lsqj30be0743yz.jpg)

现在所说的Linux操作系统，主要是指使用[Linux内核](https://www.kernel.org/)开发的操作系统，所存在的Linux发行版大概有数百，但是，历史会总是选择留下适合生存的，其中常用的有RedHat、Debian、SUSE、CentOS、Fedora、Ubuntu等（推荐大家搜一下不同）。其中CentOS属于社区开源版，是RedHat的社区版，由红帽公司支持开发，CentOS具有代表性，很多人选择从它开始入门。其他的发行版各有优点，不过入门还是要从典型开始学，之后可以自行学习其他的发行版（[历史](https://futurist.se/gldt/)）。

<!--more-->

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16qozsnyej30m60bhmxi.jpg)

操作系统有两种用户接口：GUI（Graphic User Interface，图形用户接口）和CLI（Command Line Interface，命令行接口）。

在Linux上常用的GUI有两种，一种是[GNome](https://www.gnome.org/)，另一种是[KDE](https://kde.org/)，都是基于X Protocol的一种实现，linux上大多GUI都是以[X Windows](https://baike.baidu.com/item/X%20Window/7249336?fromtitle=X%20Windows&fromid=5675632&fr=aladdin)为基础，以C/S模式运行，只要在本地安装对应服务器模块，并且配置合理，就可以通过客户端软件在本地或者远程连接。但是使用GUI进行通信时传输数据量较大，不适宜远程连接时使用。普遍认为，命令行在处理重复和复杂任务时比图形界面更快更方便，这也是很多图形操作系统保留命令行工具的原因。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g15j0w3qa4j30v40dwaa8.jpg)

### Linux哲学思想

Linux哲学思想：

1. 一切皆文件；

2. 小型，单一用途的程序；

3. 连接程序，共同完成复杂功能；

4. 避免令人困惑的用户界面；

5. 配置数据存储在文本中；

解释：
一切皆文件：是 Unix/Linux 的基本哲学之一。不仅普通的文件，目录、字符设备、块设备、 套接字等在 Unix/Linux 中都是以文件被对待；它们虽然类型不同，但是对其提供的却是同一套操作界面。

小型，单一用途的程序：程序和可执行文件不要太复杂，这样才能保证了linux内核的高效运行

连接程序，共同完成复杂功能：复杂的任务可以通过连接多个简单的程序实现复杂的功能。对于复杂的功能linux通过许多简单程序的组合等方式实现，在保证简单功能的高效性的同时，复杂的程序也必然是高效性的

避免令人困惑的用户界面：如windows那样出了问题一般人选择的会是重启，实在是不行的话就是重新安装系统了，因为对于windows那样不是开源的，并且用户界面比较 复杂操作系统出了问题，一般的人是根本没有办法解决的。但是linux就不一样了，第一linux是开源的，无论什么问题都可以通过简洁的命令行实现 排错，修改系统的配置，一切都是简洁明了为基础。

配置数据存储在文本中：linux所有的配置文件都存放在文本配置文件当中，无论什么配置修改都只需修改其配置文件即可，配置文件时文本形式的只需任意一款文本编辑器修改即可而不是类似于windows那样将保存在注册表中，并且windows的注册表需要专门的二进制或十六进制的编辑器才可编辑，修改比较复杂。

### 文件类型

**Linux中文件名、命令严格区分字符大小写。**

普通文件（regular file）：就是一般存取的文件，由ls -al显示出来的属性中，第一个属性为 [-]，例如 [-rwxrwxrwx]。另外，依照文件的内容，又大致可以分为：

1. 纯文本文件（ASCII）：这是Unix系统中最多的一种文件类型，之所以称为纯文本文件，是因为内容可以直接读到的数据，例如数字、字母等等。设 置文件几乎都属于这种文件类型。举例来说，使用命令“cat ~/.bashrc”就可以看到该文件的内容（cat是将文件内容读出来）。

2. 二进制文件（binary）：系统其实仅认识且可以执行二进制文件（binary file）。Linux中的可执行文件（脚本，文本方式的批处理文件不算）就是这种格式的。举例来说，命令cat就是一个二进制文件。

3. 数据格式的文件（data）：有些程序在运行过程中，会读取某些特定格式的文件，那些特定格式的文件可以称为数据文件（data file）。举例来说，Linux在用户登入时，都会将登录数据记录在 /var/log/wtmp文件内，该文件是一个数据文件，它能通过last命令读出来。但使用cat时，会读出乱码。因为它是属于一种特殊格式的文件。

  - 目录文件（directory）：就是目录，第一个属性为 [d]，例如 [drwxrwxrwx]。

  - 符号连接文件（link）：类似Windows下面的快捷方式。第一个属性为 [l]，例如 [lrwxrwxrwx]。

  - 设备与设备文件（device）：与系统外设及存储等相关的一些文件，通常都集中在 /dev目录。通常又分为两种：

  - 块设备文件：就是存储数据以供系统存取的接口设备，简单而言就是硬盘。例如一号硬盘的代码是 /dev/hda1等文件。第一个属性为 [b]。

  - 字符设备文件：即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]。
  ```
  			major number：主设备号，用于标识设备类型，进而确定要加载的驱动程序
  			minor number：次设备号，用于标识同一类型中的不同的设备；
  			8位二进制：0-255
  ```

  - 套接字（sockets）：这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。第一个属性为 [s]，最常在 /var/run目录中看到这种文件类型。

  - 管道（FIFO,pipe）：FIFO也是一种特殊的文件类型，它主要的目的是，解决多个程序同时存取一个文件所造成的错误。FIFO是first-in-first-out（先进先出）的缩写。第一个属性为 [p]。

### 目录结构

Linux的发行版众多，如何保证每个版本的文件目录统一呢？为此社区组织发行了[Filesystem Hierarchy Standard](http://www.pathname.com/fhs/)（FHS），所有发行版必须遵循这个标准设置目录结构，所以说，人们可以减少学习多个发行版的时间。网站上有多种格式的文档，同学们可以选择适合的下载，以备后面使用查看。

Linux目录采用倒置树形结构，路径分隔符为`/`。

相对路径是指本目录为参考，`.`代表本目录，`..`代表上一级目录，绝对路径是指从根目录开始一直到目标文件的路径。

下面是对目录的简要说明：

| /      | 根目录                                                       |
| ------ | ------------------------------------------------------------ |
| /bin   | 所有用户可用的基本命令程序文件                               |
| /boot  | 放置内核及LILO、GRUB等导引程序(bootloader)的文件，用于启动。 |
| /dev   | 硬盘，分区，键盘，鼠标，USB，tty等所有的设备文件都放在这个目录。 |
| /etc   | 系统的所有配置文件都存放在此目录中。                         |
| /home  | 普通的家目录的集中位置；一般每个普通用户的家目录默认为此目录下与用户名同名的子目录，/home/USERNAME |
| /lib   | 为系统启动或根文件系统上的应用程序(/bin, /sbin等)提供共享库，以及为内核提供内核模块,libc.so.：动态链接的C库；ld：运行时链接器/加载器；modules：用于存储内核模块的目录。 |
| /media | 挂接CD-ROM等设备的目录                                       |
| /mnt   | 移动设备文件系统的临时挂点                                   |
| /opt   | 存放后来追加的用户应用程序                                   |
| /root  | 管理员之家                                                   |
| /sbin  | 存放系统管理所需要的命令                                     |
| /tmp   | 临时文件目录，重新启动时被清除                               |
| /usr   | 存放只能读的命令和其他文件。/usr/X11R6　X Window系统/usr/bin　用户和管理员的标准命令/usr/include　c/c++等各种开发语言环境的标准include文件/usr/lib　应用程序及程序包的连接库/usr/local/　系统管理员安装的应用程序目录/usr/local/share　系统管理员安装的共享文件/usr/sbin　用户和管理员的标准命令/usr/share　存放使用手册等共享文件的目录/usr/share/dict　存放词表的目录（选项）/usr/share/man　系统使用手册/usr/share/misc　一般数据/usr/share/sgml　SGML数据（选项）/usr/share/xml　XML数据（选项） |
| /var   | 存放应用程序数据和日志记录的目录，例如，Apache Web服务器的文档一般就放在/var/www/html下。/var/cache　应用程序缓存目录/var/account　处理账号日志（选项）/var/crash　系统错误信息（选项）/var/games　游戏数据/var/lib　　各种状态数据/var/lock　文件锁定纪录/var/log　日志记录/var/mail　电子邮件/var/opt　/opt目录的变量数据/var/run　进程的标示数据/var/spool　存放电子邮件，打印任务等的队列目录。/var/spool/rwho　/var/tmp　临时文件目录/var/yp　NIS等黄页数据（选项） |
| /proc  | 基于内存的虚拟文件系统，用于为内核及进程存储其相关信息；它们多为内核参数，例如net.ipv4.ip_forward, 虚拟为net/ipv4/ip_forward, 存储于/proc/sys/, 因此其完整路径为/proc/sys/net/ipv4/ip_forward |
| /sys   | sysfs虚拟文件系统提供了一种比proc更为理想的访问内核数据的途径；其主要作用在于为管理Linux设备提供一种统一模型的的接口 |

可以用一张思维导图来表示：

![](https://ws1.sinaimg.cn/large/006KyevZgy1g15j0wnokvj30yg12cacx.jpg)



### 常用命令

![](https://ws1.sinaimg.cn/large/006KyevZgy1g15j0xj2wlj30y315k78j.jpg)

下一章讲述CentOS 7虚拟机的安装的基础配置。

### Docker 部署linux学习环境

docker安装以及使用请参考docker博文，怕麻烦的请跳过，

#### Busybox

BusyBox 是一个集成了一百多个最常用 Linux 命令和工具（如 cat、echo、grep、mount、telnet 等）的精简工具箱，它只需要几 MB 的大小，很方便进行各种快速验证，被誉为“Linux 系统的瑞士军刀”。

BusyBox 可运行于多款 POSIX 环境的操作系统中，如 Linux（包括 Android）、Hurd、FreeBSD 等。

**获取官方镜像**

在 Docker Hub 中搜索 busybox 相关的镜像。

```bash
$ docker search busybox
NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
busybox                         Busybox base image.                             755       [OK]
progrium/busybox                                                                63                   [OK]
radial/busyboxplus              Full-chain, Internet enabled, busybox made...   11                   [OK]
odise/busybox-python                                                            3                    [OK]
multiarch/busybox               multiarch ports of ubuntu-debootstrap           2                    [OK]
azukiapp/busybox                This image is meant to be used as the base...   2                    [OK]
...
```

读者可以看到最受欢迎的镜像同时带有 OFFICIAL 标记，说明它是官方镜像。用户使用 docker pull 指令下载镜像 `busybox:latest`：

```bash
$ docker pull busybox:latest
busybox:latest: The image you are pulling has been verified
e433a6c5b276: Pull complete
e72ac664f4f0: Pull complete
511136ea3c5a: Pull complete
df7546f9f060: Pull complete
Status: Downloaded newer image for busybox:latest
```

下载后，可以看到 busybox 镜像只有2.433 MB：

```bash
$ docker image ls
REPOSITORY                   TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox                   latest              e72ac664f4f0        6 weeks ago         2.433 MB
```

**运行 busybox**

启动一个 busybox 容器，并在容器中执行 grep 命令。

```bash
$ docker run -it busybox
/ # grep
BusyBox v1.22.1 (2014-05-22 23:22:11 UTC) multi-call binary.

Usage: grep [-HhnlLoqvsriwFE] [-m N] [-A/B/C N] PATTERN/-e PATTERN.../-f FILE [FILE]...

Search for PATTERN in FILEs (or stdin)

        -H      Add 'filename:' prefix
        -h      Do not add 'filename:' prefix
        -n      Add 'line_no:' prefix
        -l      Show only names of files that match
        -L      Show only names of files that don't match
        -c      Show only count of matching lines
        -o      Show only the matching part of line
        -q      Quiet. Return 0 if PATTERN is found, 1 otherwise
        -v      Select non-matching lines
        -s      Suppress open and read errors
        -r      Recurse
        -i      Ignore case
        -w      Match whole words only
        -x      Match whole lines only
        -F      PATTERN is a literal (not regexp)
        -E      PATTERN is an extended regexp
        -m N    Match up to N times per file
        -A N    Print N lines of trailing context
        -B N    Print N lines of leading context
        -C N    Same as '-A N -B N'
        -e PTRN Pattern to match
        -f FILE Read pattern from file
```

查看容器内的挂载信息。

```bash
/ # mount
rootfs on / type rootfs (rw)
none on / type aufs (rw,relatime,si=b455817946f8505c)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,mode=755)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
/dev/disk/by-uuid/b1f2dba7-d91b-4165-a377-bf1a8bed3f61 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/disk/by-uuid/b1f2dba7-d91b-4165-a377-bf1a8bed3f61 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/disk/by-uuid/b1f2dba7-d91b-4165-a377-bf1a8bed3f61 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro,data=ordered)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
proc on /proc/sys type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sysrq-trigger type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/irq type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/bus type proc (ro,nosuid,nodev,noexec,relatime)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,mode=755)
```

busybox 镜像虽然小巧，但包括了大量常见的 Linux 命令，读者可以用它快速熟悉 Linux 命令。

**相关资源**

- `Busybox` 官网：https://busybox.net/
- `Busybox` 官方仓库：https://git.busybox.net/busybox/
- `Busybox` 官方镜像：https://hub.docker.com/_/busybox/
- `Busybox` 官方仓库：https://github.com/docker-library/busybox

#### Alpine

`Alpine` 操作系统是一个面向安全的轻型 `Linux` 发行版。它不同于通常 `Linux` 发行版，`Alpine` 采用了 `musl libc` 和 `busybox` 以减小系统的体积和运行时资源消耗，但功能上比 `busybox` 又完善的多，因此得到开源社区越来越多的青睐。在保持瘦身的同时，`Alpine` 还提供了自己的包管理工具 `apk`，可以通过 `https://pkgs.alpinelinux.org/packages` 网站上查询包信息，也可以直接通过 `apk` 命令直接查询和安装各种软件。

`Alpine` 由非商业组织维护的，支持广泛场景的 `Linux`发行版，它特别为资深/重度`Linux`用户而优化，关注安全，性能和资源效能。`Alpine` 镜像可以适用于更多常用场景，并且是一个优秀的可以适用于生产的基础系统/环境。

`Alpine` Docker 镜像也继承了 Alpine Linux 发行版的这些优势。相比于其他 `Docker`镜像，它的容量非常小，仅仅只有 5 MB 左右（对比 Ubuntu 系列镜像接近 200 MB），且拥有非常友好的包管理机制。官方镜像来自 `docker-alpine` 项目。

目前 Docker 官方已开始推荐使用 `Alpine` 替代之前的 `Ubuntu` 做为基础镜像环境。这样会带来多个好处。包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。

下表是官方镜像的大小比较：

```bash
REPOSITORY          TAG           IMAGE ID          VIRTUAL SIZE
alpine              latest        4e38e38c8ce0      4.799 MB
debian              latest        4d6ce913b130      84.98 MB
ubuntu              latest        b39b81afc8ca      188.3 MB
centos              latest        8efe422e6104      210 MB
```

**获取并使用官方镜像**

由于镜像很小，下载时间往往很短，读者可以直接使用 `docker run` 指令直接运行一个 `Alpine` 容器，并指定运行的 Linux 指令，例如：

```bash
$ docker run alpine echo '123'
123
```

**迁移至 Alpine 基础镜像**

目前，大部分 Docker 官方镜像都已经支持 Alpine 作为基础镜像，可以很容易进行迁移。

例如：

- ubuntu/debian -> alpine
- python:2.7 -> python:2.7-alpine
- ruby:2.3 -> ruby:2.3-alpine

另外，如果使用 `Alpine` 镜像替换 `Ubuntu` 基础镜像，安装软件包时需要用 apk 包管理器替换 apt 工具，如

```bash
$ apk add --no-cache <package>
```

`Alpine` 中软件安装包的名字可能会与其他发行版有所不同，可以在 `https://pkgs.alpinelinux.org/packages` 网站搜索并确定安装包名称。如果需要的安装包不在主索引内，但是在测试或社区索引中。那么可以按照以下方法使用这些安装包。

```bash
$ echo "http://dl-4.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
$ apk --update add --no-cache <package>
```

**相关资源**

- `Alpine` 官网：http://alpinelinux.org/
- `Alpine` 官方仓库：https://github.com/alpinelinux
- `Alpine` 官方镜像：https://hub.docker.com/_/alpine/
- `Alpine` 官方镜像仓库：https://github.com/gliderlabs/docker-alpine

#### Debian/Ubuntu

Debian 和 Ubuntu 都是目前较为流行的 Debian 系的服务器操作系统，十分适合研发场景。Docker Hub 上提供了官方镜像，国内各大容器云服务也基本都提供了相应的支持。

```
$ docker run -it debian bash
$ docker run -ti ubuntu:18.04 /bin/bash
```

- `Debian` 官网：https://www.debian.org/
- `Neuro Debian` 官网：http://neuro.debian.net/
- `Debian` 官方仓库：https://github.com/Debian
- `Debian` 官方镜像：https://hub.docker.com/_/debian/
- `Debian` 官方镜像仓库：https://github.com/tianon/docker-brew-debian/
- `Ubuntu` 官网：http://www.ubuntu.org.cn/global
- `Ubuntu` 官方仓库：https://github.com/ubuntu
- `Ubuntu` 官方镜像：https://hub.docker.com/_/ubuntu/
- `Ubuntu` 官方镜像仓库：https://github.com/tianon/docker-brew-ubuntu-core

#### CentOS/Fedora

```
$ docker run -it centos bash
$ docker run -it fedora bash
```

- `Fedora` 官网：https://getfedora.org/
- `Fedora` 官方仓库：https://github.com/fedora-infra
- `Fedora` 官方镜像：https://hub.docker.com/_/fedora/
- `Fedora` 官方镜像仓库：https://github.com/fedora-cloud/docker-brew-fedora
- `CentOS` 官网：https://getfedora.org/
- `CentOS` 官方仓库：https://github.com/CentOS
- `CentOS` 官方镜像：https://hub.docker.com/_/centos/
- `CentOS` 官方镜像仓库：https://github.com/CentOS/CentOS-Dockerfiles