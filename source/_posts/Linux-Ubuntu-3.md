---
title: Ubuntu 软件包管理命令
date: 2019-05-22 11:55:22
tags:
 - Linux
 - Ubuntu
category:
 - Linux
 - Ubuntu
---

RedHat/CentOS类和Debian/Ubuntu类都是Linux发行版，基础命令基本上相同，最大的不同点就是软件包管理器的不同。

<!--more-->

CentOS（Community ENTerprise Operating System）是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成。RedHat Enterprise Linux （RHEL）是企业发行版。它每五年左右更新一次，在系统的稳定性，前瞻性和安全性上有着极大的优势。由于CentOS出自同样的源代码，因此要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。CentOS通常在RedHat的发布后就会很快发行。我们使用CentOS的原因在于RHEL发行版的标准支持服务费用非常高，大约每台服务器800美元左右，对于我们很多拥有数十台甚至上百台服务器的用户来说，这是必须要控制的成本。 

​    Ubuntu是一个以桌面应用为主的Linux操作系统。Ubuntu基于Debian发行版和GNOME桌面环境，与Debian的不同在于它每6个月会发布一个新版本。Ubuntu的目标在于为一般用户提供一个最新的、同时又相当稳定的主要由自由软件构建而成的操作系统。Ubuntu具有庞大的社区力量，用户可以方便地从社区获得帮助。

其中CentOS使用yum/dnf管理器安装`.rpm`软件二进制安装包，Ubuntu使用dpkg/apt安装`.deb`包，当然，随着Linux发行版的多方分天下，导致在Linux上打包成可在多平台运行的二进制基本上不太可能，而且还要面对包之间依赖的问题（基本上代表你是不能安装了，虽然有一部分可以通过aptitude解决），因此有人提出来其他的解决方法，比如：flatpak、snap、appImage、java的jar或者war包等等方案，但是其各有各的缺陷，后面再详细讲解。

#### 安装模式

谈谈Linux世界用户较多的前2大主要分支，

- RedHat Red Hat Enterprise Linux 简称RHEL rpm (RedHat, CentOS, Fedora, Oracle...)
- Debian Ubuntu Server 简称Ubuntu deb (Debian, Ubuntu, Mint, MX Linux...)
- 还有：Arch, Gentoo, SUSE, BSD, Android等... 

前两大分支的包管理有2大阵营，安装文件互不相融。

- 安装文件：*.rpm，RedHat分支，CentOS等，使用yum命令安装...
- 安装文件：*.deb，Debian分支，Ubuntu等，使用apt-get命令安装...

然后2边都推出了新的规则，希望能一统江湖：

- Flatpak 归属 RedHat ；
- Snap 归属 Canonical 。

这两大阵营竞争的同时，

Arch的pacman包管理器，足够多的软件包被越来越多的人接受。源自Arch的Manjaro开箱即用型Linux系统已经成为distrowatch.com排名第一的Linux分支。

下面是新出的3个新出的应用包规则：

- AppImage 是一种很管用的软件磁盘映像。

  优点是：简单方便，下载单独一个文件，双击打开使用即可。删除也方便。
  缺点是：即使你直接从开发者的网站获得软件，仍然不知道应用程序是否已被篡改。
  更新：要重新下载最新的文件。
  https://appimage.org/

- Flatpak 提供隔离的运行时环境，Flatpaks是针对Linux桌面设计的。
  https://flatpak.org/

- Snap  Packages是压缩文件系统。
  Snap软件包是Canonical提出的一个打包概念，针对Linux和物联网而设计。
  https://snapcraft.io/

参考：<http://os.51cto.com/art/201806/575608.htm>

技术区别还没细看代码。技术路线大概差不多：

1. 依赖全部打包，并且通过mount namespace与发行版的文件隔离。
2. 通过其它容器技术控制应用运行时的权限。

1很容易，Firefox/Steam等的跨发行版二进制包一向是这么做的。只是没用mount namespace，在启动脚本里设一下LD_LIBRARY_PATH罢了。2很难，几乎肯定需要应用修改代码。Linux桌面上恶意应用问题又不严重，需求不足。

**Flatpak 和 Snap 都还没有成熟**，之前 Snap 打包 Libreoffice 就有 1GB 大的 bug。从新闻上看 Snap 看起来很火，但是只是 Canonical 单方面在推动，其他发行版的维护者都还没有接收呢，只是 Canonical 的员工自己在其他发行版上发布了个人打包。在安全性上，Flatpak 和 Snap 需要桌面使用 wayland/mir，两者在桌面都还没有流行起来。Flatpak 是 RH 的东西，Snap 是 Canonical 的东西，况且如果采用 Snap，应用商店还是 Canonical 掌控的，其他发行版也许不怎么乐意。 

Flatpak/Snap 也许会作为一个应用安装来源的补充，不过已经足够吸引人了，就像很多人喜欢 Arch 因为它的 AUR 源。

### dpkg

> “dpkg ”是“Debian Packager ”的简写。为 “Debian” 专门开发的套件管理系统，方便软件的安装、更新及移除。所有源自“Debian”的“Linux ”发行版都使用 “dpkg”，例如 “Ubuntu”、“Knoppix ”等。 --《百度百科》

```
Usage: dpkg [<option> ...] <command>
```

主要介绍dpkg常用的一些选项(option):

- -L 列出属于指定软件包的文件，也可以理解为列出指定软件包将所属的文件都安装到什么位置了
- -s 查看指定软件包的详细信息
- -l 列出系统安装以及安装过的软件包。软件包两种状态(rc/ii)，rc表示已经删除，但是配置文件还未清理干净；ii表示软件包正常安装，也就是目前正常安装在系统中
- -P/--purge 删除软件包，并且同时删除配置文件，可以清理`-l`中`rc`状态的软件包
- -S 查找指定文件所属的软件包，类似`rpm -qf file`
- -I 查看指定的未安装的软件包的详细信息
- -c 列出未安装的软件包所包含的文件以及安装后在系统中对应的路径信息
- -i 安装指定的软件包
- -r 卸载安装的软件包
- -B/--auto-deconfigure 安装软件包，即使有损坏也安装

> 有些时候我们通过dpkg安装软件包的时候有些依赖问题，我们可以通过`apt-get install -f`来解决。

#### deb软件包名规则

格式为：Package_Version-Build_Architecture.deb

如：nano_1.3.10-2_i386.deb

```
* 软件包名称(Package Name): nano

* 版本(Version Number):1.3.10

* 修订号(Build Number):2

* 平台(Architecture):i386
```

#### dpkg软件包相关文件

/etc/dpkg/dpkg.cfg  dpkg包管理软件的配置文件【Configuration file with default options】

/var/log/dpkg.log  dpkg包管理软件的日志文件【Default log file (see /etc/dpkg/dpkg.cfg(5) and option --log)】

/var/lib/dpkg/available  存放系统所有安装过的软件包信息【List of available packages.】

/var/lib/dpkg/status   存放系统现在所有安装软件的状态信息

/var/lib/dpkg/info   记安装软件包控制目录的控制信息文件

#### dpkg数据库

dpkg 使用文本文件作为数据库来维护系统中软件，包括文件清单, 依赖关系, 软件状态, 等等详细的内容,通常在 /var/lib/dpkg 目录下。 通常在 status 文件中存储软件状态和控制信息。 在 info/ 目录下备份控制文件， 并在其下的 .list 文件中记录安装文件清单， 其下的 .mdasums 保存文件的 MD5 编码。

例：查询dpkg数据库（显示所有已安装的Deb包）

```
$ dpkg -l
期望状态=未知(u)/安装(i)/删除(r)/清除(p)/保持(h)
| 状态=未安装(n)/已安装(i)/仅存配置(c)/仅解压缩(U)/配置失败(F)/不完全安装(H)/触发器等待(W)/触发器未决(T)
|/ 错误?=(无)/须重装(R) (状态，错误：大写=故障)
||/ 名称           版本         体系结构     描述
+++-==============-============-============-==================================
ii  accountsservic 0.6.45-1ubun amd64        query and manipulate user account 
ii  acl            2.2.52-3buil amd64        Access control list utilities
ii  acpi-support   0.142        amd64        scripts for handling many ACPI eve
ii  acpid          1:2.0.28-1ub amd64        Advanced Configuration and Power I
ii  adduser        3.116ubuntu1 all          add and remove users and groups
ii  adobe-flash-pr 1:20190910.1 amd64        GTK+ control panel for Adobe Flash
ii  adobe-flashplu 1:20190910.1 amd64        Adobe Flash Player plugin
ii  adwaita-icon-t 3.28.0-1ubun all          default icon theme of GNOME (small
ii  adwaita-icon-t 3.28.0-1ubun all          default icon theme of GNOME
ii  adwaita-qt:amd 1.0-2        amd64        Qt 5 port of GNOME’s Adwaita theme
ii  albert         0.16.1       amd64        A sophisticated, plugin-based, sta
ii  alsa-base      1.0.25+dfsg- all          ALSA driver configuration files
ii  alsa-utils     1.1.3-1ubunt amd64        Utilities for configuring and usin
ii  amd64-microcod 3.20180524.1 amd64        Processor microcode firmware for A
...
```

1）第一字符为期望值(Desired=Unknown/Install/Remove/Purge/Hold)，它包括：

> u  Unknown状态未知,这意味着软件包未安装,并且用户也未发出安装请求.
>
> i  Install用户请求安装软件包.
>
> r  Remove用户请求卸载软件包.
>
> p  Purge用户请求清除软件包.
>
> h  Hold用户请求保持软件包版本锁定.

2）第二列,是软件包的当前状态(Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend)

> n  Not软件包未安装.
>
> i  Inst软件包安装并完成配置.
>
> c  Conf-files软件包以前安装过,现在删除了,但是它的配置文件还留在系统中.
>
> u  Unpacked软件包被解包,但还未配置.
>
> f  halF-conf试图配置软件包,但是失败了.
>
> h  Half-inst软件包安装,但是但是没有成功.
>
> w trig-aWait触发器等待
>
> t Trig-pend触发器未决

3）第三列标识错误状态,第一种状态标识没有问题,为空. 其它符号则标识相应问题（Err?=(none)/Reinst-required (Status,Err: uppercase=bad)）

> h  软件包被强制保持,因为有其它软件包依赖需求,无法升级.
>
> r  Reinst-required，软件包被破坏,可能需要重新安装才能正常使用(包括删除).
>
> x  软包件被破坏,并且被强制保持.

案例说明：

ii —— 表示系统正常安装了该软件

pn —— 表示安装了该软件，后来又清除了

un —— 表示从未安装过该软件

iu —— 表示安装了该软件，但是未配置

rc —— 该软件已被删除，但配置文件仍在

#### dpkg子命令

为了方便用户使用，dpkg不仅提供了大量的参数选项, 同时也提供了许多子命令。

比如：

dpkg-deb、dpkg-divert、dpkg-query、dpkg-split、dpkg-statoverride、start-stop-daemon

#### dpkg使用手册

**安装**

）安装相关命令

```
dpkg -i package-name.deb    # --install, 安装软件包，必须是deb包的完整名称。（软件的安装可被拆分为两个对立的过程“解包”和“配置”）

dpkg --unpack package-name.deb  # “解包”：解开软件包到系统目录但不配置,如果和-R一起使用，参数可以是一个目录

dpkg --configure package-name.deb  #“配置”：配置软件包

dpkg -c package-name.deb    #列出 deb 包的内容
```

2）安装相关选项

```
-R, --recursive    Recursively handle all regular files matching pattern *.deb found at specified directories and all of its subdirectories. This can be  used  with -i, -A, --install, --unpack and --avail actions（递归地指向特定目录的所有安装包，可以结合-i, -A, --install, --unpack 与 --avail一起使用）
```

**移除软件包**

```
dpkg -r package-name  # --remove， 移除软件包，但保留其配置文件

dpkg -P package-name  # --purge， 清除软件包的所有文件（removes everything, including conffiles）
```

**查询**

```
dpkg -l package-name-pattern  # --list, 查看系统中软件包名符合pattern模式的软件包

dpkg -L package-name  # --listfiles, 查看package-name对应的软件包安装的文件及目录

dpkg -p package-name  # --print-avail, 显示包的具体信息

dpkg -s package-name  # --status, 查看package-name（已安装）对应的软件包信息

dpkg -S filename-search-pattern  # --search, 从已经安装的软件包中查找包含filename的软件包名称
```

（Tip：也可使用子命令dpkg-query来进行查询操作）

例1：列出系统上安装的与dpkg相关的软件包

```
dpkg -l \*dpkg*
```

例2：查看dpkg软件包安装到系统中的文件

```
dpkg -L dpkg
```

更多dpkg的使用方法可在命令行里使用*man dpkg*来查阅 或直接使用*dpkg --help*。

### apt

虽然我们在使用**dpkg**时，已经解决掉了 软件安装过程中的大量问题，但是当依赖关系不满足时，仍然需要手动解决，而**apt**这个工具解决了这样的问题，linux distribution 先将软件放置到对应的服务器中，然后分析软件的依赖关系，并且记录下来，然后当客户端有安装软件需求时，通过清单列表与本地的dpkg以存在的软件数据相比较，就能从网络端获取所有需要的具有依赖属性的软件了。

Ubuntu采用集中式的软件仓库机制，将各式各样的软件包分门别类地存放在软件仓库中，进行有效地组织和管理。然后，将软件仓库置于许许多多的镜像服务器中，并保持基本一致。这样，所有的Ubuntu用户随时都能获得最新版本的安装软件包。因此，对于用户，这些镜像服务器就是他们的软件源（Reposity）

然而，由于每位用户所处的网络环境不同，不可能随意地访问各镜像站点。为了能够有选择地访问，在Ubuntu系统中，使用软件源配置文件/etc/apt/sources.list列出最合适访问的镜像站点地址。

**例1**：apt-get的更新过程

> 1）执行apt-get update
>
> 2）程序分析/etc/apt/sources.list
>
> 3）自动连网寻找list中对应的Packages/Sources/Release列表文件，如果有更新则下载之，存入/var/lib/apt/lists/目录
>
> 4）然后 apt-get install 相应的包 ，下载并安装。

即使这样，软件源配置文件只是告知Ubuntu系统可以访问的镜像站点地址，但那些镜像站点具体都拥有什么软件资源并不清楚。若每安装一个软件包，就在服务器上寻找一遍，效率是很低的。因而，就有必要为这些软件资源列个清单（建立索引文件），以便本地主机查询。

用户可以使用“apt-get update”命令刷新软件源，建立更新软件包列表。在Ubuntu Linux中，“apt-get update”命令会扫描每一个软件源服务器，并为该服务器所具有软件包资源建立索引文件，存放在本地的/var/lib/apt/lists/目录中。 使用apt-get执行安装、更新操作时，都将依据这些索引文件，向软件源服务器申请资源。因此，在计算机设备空闲时，经常使用“apt-get update”命令刷新软件源，是一个好的习惯。

**例2**：apt-get install原理图

![image.png](https://i.loli.net/2019/09/27/sf9zbWBx6wci7La.png)

**apt相关文件**

var/lib/dpkg/available    文件的内容是软件包的描述信息, 该软件包括当前系统所使用的Debian 安装源中的所有软件包,其中包括当前系统中已安装的和未安装的软件包.

/etc/apt/sources.list  记录软件源的地址（当你执行 sudo apt-get install xxx 时，Ubuntu 就去这些站点下载软件包到本地并执行安装）

/var/cache/apt/archives  已经下载到的软件包都放在这里（用 apt-get install 安装软件时，软件包的临时存放路径）

/var/lib/apt/lists    使用apt-get update命令会从/etc/apt/sources.list中下载软件列表，并保存到该目录

**源文件**

apt的源文件由配置文件/etc/apt/sources.list指定，该文件配置:

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

最后执行如下命令更新源

```
sudo apt-get update
sudo apt-get upgrade
```

#### apt使用手册

apt-get 是一个下载安装软件包的简单命令行接口。最常用的命令是update(更新)和install(安装)。

常用选项：

> -h 本帮助文件。
>
> -q 输出到日志 - 无进展指示
>
> -qq 不输出信息，错误除外
>
> -d 仅下载 - 不安装或解压归档文件
>
> -s 不实际安装。模拟执行命令
>
> -y 假定对所有的询问选是，不提示
>
> -f 尝试修正系统依赖损坏处
>
> -m 如果归档无法定位，尝试继续
>
> -u 同时显示更新软件包的列表
>
> -b 获取源码包后编译
>
> -V 显示详细的版本号
>
> -c=? 阅读此配置文件
>
> -o=? 设置自定的配置选项，如 -o dir::cache=/tmp

常用命令：

> 1）apt-get update  更新源
>
> ​     【aptitude update】
>
> 2）apt-get dist-upgrade  升级系统到相应的发行版(根据 source.list 的配置)
>
> ​     【aptitude dist-upgrade】
>
> 3）apt-get upgrade  更新所有已经安装的软件包
>
> ​     【aptitude upgrade】
>
> 4）apt-get install package_name  安装软件包(加上 --reinstall重新安装)
>
> ​    apt-get install package_name=version    安装指定版本的软件包
>
> ​    【aptitude install package_name】
>
> 5）apt-get remove  package_name    卸载一个已安装的软件包（保留配置文件）
>
> ​    【aptitude remove package_name】
>
> 6）apt-get purge package_name  移除软件包（删除配置信息）
>
> ​     或apt-get  --purge remove packagename
>
> ​    【aptitude purge package_name】
>
> 7）apt-get check  检查是否有损坏的依赖
>
> 8）apt-get autoclean  删除你已经删掉的软件（定期运行这个命令来清除那些已经卸载的软件包的.deb文件。通过这种方式，您可以释放大量的磁盘空间。如果您的需求十分迫切，可以使用apt-get clean以释放更多空间。这个命令会将已安装软件包裹的.deb文件一并删除。大多数情况下您不会再用到这些.debs文件，因此如果您为磁盘空间不足 而感到焦头烂额，这个办法也许值得一试）
>
> ​    【aptitude autoclean】
>
> 9）apt-get clean    把安装的软件的备份也删除，不过这样不会影响软件的使用
>
> ​    【aptitude clean】

**apt-cache**

apt-cache - query the APT cach.

apt-cache performs a variety of operations on APT's package cache.  apt-cache does not manipulate the state of the system but does provide operations to search and generate interesting output from the package metadata.

> 1）apt-cache depends packagename  了解使用依赖
>
> 2）apt-cache rdepends packagename  是查看该包被哪些包依赖
>
> 3）apt-cache search  packagename  搜索包
>
> ​    【aptitude search packagename】
>
> 4）apt-cache show packagename  获取包的相关信息，如说明、大小、版本等
>
> ​    【aptitude show packagename】
>
> 5）apt-cache  showpkg packagename    显示软件包的大致信息

（注：中括号【】内的aptitude也是类似于apt-*的一个包管理上层工具）

### 参考：

1. 阿里云开源镜像站：[http://mirrors.aliyun.com/](https://link.jianshu.com/?t=http://mirrors.aliyun.com/)
2. 网易开源镜像站：[http://mirrors.163.com/](https://link.jianshu.com/?t=http://mirrors.163.com/)
3. Ubuntu官方网站：[https://www.ubuntu.com](https://link.jianshu.com/?t=https://www.ubuntu.com)
4. Debian官方网站：[https://www.debian.org/doc/user-manuals](https://link.jianshu.com/?t=https://www.debian.org/doc/user-manuals)
5. [https://www.debian.org/doc/manuals/debian-handbook/apt.zh-cn.html](https://link.jianshu.com/?t=https://www.debian.org/doc/manuals/debian-handbook/apt.zh-cn.html)