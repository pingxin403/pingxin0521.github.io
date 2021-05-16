---
title: Ubuntu 优化
date: 2019-04-22 11:55:22
tags:
 - Linux
 - Ubuntu
category:
 - Linux
 - Ubuntu
---
### 前言

由于博主的电脑老化，运行Windows时风扇高速运转，电脑时不时卡顿，所以为了更好的体验和使用，我开始寻找是否有一些可以对用户友好，并且占用更少资源的操作系统，然后发现其实Ubuntu就是一个很好的操作系统，并且具有很高水平的可定制桌面，有QQ、百度网盘的解决方案，现在转入Ubuntu可以说风险更小一些，当然Ubuntu有它的不足，很多在Windows上常用的软件在Ubuntu上很少有适合的，所以有所取舍。

> 推荐安装Ubuntu 18.04 TLS，因为是长期支持版，更适合在个人电脑上安装，新版本在虚拟机安装玩玩就行了。

<!--more-->

### 优化


Ubuntu的启动速度非常快，按了开机键之后很快就进入桌面。但我们仍然可以充分利用内存，通过多种方法让开机速度更快。某些方法真的可以提速，对于旧电脑的效果尤其明显。

选用轻量级的桌面环境，关闭不必要的应用程序，可以让一台旧电脑如释重负。如果强行在旧电脑上运行Ubuntu的Gnome桌面环境，系统的速度会拖得很慢很慢。

#### 预加载

预载是一个后台服务，可监控系统上使用的程序。它能找出程序使用的库（libraries）和二进制文件（binaries），预先加载到内存中，使程序的启动速度更快。例如，你可能经常在开机后打开Firefox浏览器和LibreOffice，那么设置了预载之后，系统在启动时会自动把这两个程序的文件加载到内存中。你再登陆系统打开这两个程序时，会发现它们比以前启动得更快。

大多数Ubuntu系统在默认情况下都没有启用预载，只有少数版本安装了这项服务。运行以下命令可以安装预载服务：
```
sudo apt-get install preload
```
这样就装好了！预载服务会在后台运行，不会打扰你的工作。你可以在 /etc/preload.conf 文件中修改预载的设置，但一般情况下使用默认设置就可以了。

#### 设置随机启动程序
你可以让某些程序在系统开机时随机启动。某些程序安装好之后也会默认随机启动——例如Dropbox。如果随机启动的程序很多，或者你的系统比较慢，那么你的系统就要花更多时间来启动。你可以在随机启动程序对话框（Startup Applications dialog）中禁止某些程序的随机启动。

很多默认启动的系统服务没有显示在列表当中。运行下面的命令，可以对这些服务进行设置：
```
sudo sed -i 's/NoDisplay=true/NoDisplay=false/g' /etc/xdg/autostart/*.desktop
```
这个命令修改了随机启动服务的文件属性，把参数“NoDisplay”的值由“true”改为“false”，让这项服务显示在随机启动的列表当中。运行了命令之后，重新打开随机启动程序对话框，你就能看到之前被隐藏的系统服务了。

除非你这些系统服务是干什么的，否则不要更改默认的启动设置。例如，如果你电脑没有蓝牙设备，那么可以禁止蓝牙管理器的随机启动；如果你使用Ubuntu One服务，就不要禁止它随机启动。

你只需要去掉程序前面的打钩，就可以禁止该程序随机启动了。不要点击Remove，那会从系统上删除该程序的。如果你想让程序恢复随机启动，在前面打钩就可以了

#### 使用轻量级的桌面环境
如果你的电脑配置比较旧，运行Ubuntu的Gnome桌面比较吃力，可以选一款轻量级的桌面环境。LXDE、XFCE都是很好的选择，如果你需要的是最简洁的桌面，可以用Xmonad。这些桌面环境都能保证最基本的桌面功能。


#### 选用轻量级的应用程序

轻量级的应用程序和轻量级的桌面环境搭配起来，能进一步提升旧电脑的系统性能。例如，你可以用Abiword代替LibreOffice，Abiword虽然功能少些，但速度更快。

如果你使用Mozilla的Thunderbird或GNOME的Evolution收发邮件，可以试试Sylpheed，它是一个轻量级的邮件管理器，带有图形界面。大多数软件都能找到轻量级的替代选择，在Google搜索一下就能找到。你甚至可以关闭所有图形界面，用终端完成所有操作——你会找到很多基于终端界面运行的软件。

#### 缩短启动菜单的延时
如果你电脑装了多个系统，Ubuntu的GRUB启动菜单会预留10秒的延时让你选择一个系统。如果你没有选择，10秒后会自动进入默认的系统。如果你通常都是进入默认系统，可以把延时缩短，节省开机时间。

运行下面的命令，在文本编辑器中打开 `/etc/default/grub` 文件，可以修改启动延时：
```
sudo vim /etc/default/grub
```
把GRUB_TIMEOUT的值改为小于10的整数。可以设为最小值1，以后如果你开机时需要选择启动菜单，可以按上下方向键或Esc键。

保存修改好的文件，运行下面的命令更新启动菜单，才能完成设置：
```
sudo update-grub2
```
你还可以使用软件[Grub-Customizer](https://www.howtogeek.com/howto/43471/how-to-configure-the-linux-grub2-boot-menu-the-easy-way/)，能够修改启动菜单的详细设置。

```
sudo add-apt-repository ppa:danielrichter2007/grub-customizer
sudo apt-get update
sudo apt-get install grub-customizer
```
打开该应用
```
sudo grub-customizer
```
#### 创建一个交换分区

如果您的计算机上的RAM较少，则应在初始Ubuntu安装期间创建一个交换分区。通常根据你的实际内存，交换分区是双倍的。如果您有2 GB RAM，则交换分区将为4 GB。

这个分区将使用您的硬盘作为内存来加速应用程序启动和后台系统进程。如果您的RAM大于4 GB，则放弃制作此交换分区。

如果您在Ubuntu安装过程中忘记了使用Swap分区，参考[如何在安装Ubuntu之后进行Swap分区](http://www.linuxidc.com/Linux/2017-07/145673.htm) 。

#### 调整交换分区的参数值（swappiness）
这个方法是有争议的。应该把swappiness设为多少才最合适，Linux内核的开发者对此存在不同的看法。

swappiness影响着Linux内核的运行速度——也就是说，swappiness的值越大，从内存转移到硬盘交换分区的数据就越多，但系统性能会相对降低。Swappiness的值可以从0到100。

0表示系统内核最大限度地使用物理内存运行程序，尽量不使用交换分区。

100表示系统内核最大限度地利用交换分区运行程序，尽量减轻内存的负担。

Ubuntu系统把swappiness参数默认设为60。如果你发现Ubuntu系统过多地使用交换分区，降低了系统性能，你可以调低swappiness的数值，比如降到10。

下面的命令可以临时把swappiness的值改为10：
```
sudo sysctl vm.swappiness=10
```
但是下次重启系统后，swappiness又会恢复为默认值。如果你不想恢复默认，可以修改/etc/sysctl.conf文件：
```
sudo vim /etc/sysctl.conf
```
打开文件，找到vm.swappiness，修改它的数值。如果找不到，可以在文件末尾添加一行命令，格式如下：
```
vm.swappiness=10
```
最后保存修改即可。

#### “降低”Compiz效果
要显着加速Ubuntu系统，您必须尽量减少Compiz效应的使用。 有很多默认加载的Compiz效果，这使得Ubuntu系统变慢。 因此，尽量禁用一些眼睛引人注意的Compiz效果，以加快你在Ubuntu上的统一桌面。
```
sudo apt-get install compizconfig-settings-manager
```

#### 减少过热
笔记本电脑上过热问题非常普遍。这使得笔记本电脑运行缓慢并且性能不佳。在Ubuntu存储库中有一个非常有效的工具，它可以帮助冷却你的系统，这将使Ubuntu系统平稳和快速。安装TLP后，不需要执行任何配置，只需运行命令即可。
```
sudo add-apt-repository ppa:linrunner/tlp
sudo apt-get update
sudo apt-get install tlp tlp-rdw
sudo tlp start
```
检查TLP是否正常运行
```
sudo tlp-stat
```
如果想要关闭TLP, `vi /etc/default/tlp`将其中的　＂TLP_ENABLE=1＂　改为　＂TLP_ENABLE=0＂即可．如果想要定制更多的东西，也可以去配置这个文件．

卸载:
要删除tlp，请打开终端并运行命令：
```
sudo apt-get remove --autoremove tlp
```
要删除PPA，请启动Software＆Updates并导航到“其他软件”选项卡。

您还可以安装笔记本电脑模式工具，通过减慢硬盘速度和内核控制来帮助降低功耗。要安装它，运行以下命令...
```
sudo add-apt-repository ppa:ubuntuhandbook1/apps
sudo apt-get update
sudo apt-get install laptop-mode-tools
```
安装后获取GUI进一步定制。运行命令...
```
sudo lmt-config-gui
```

使用如下命令来安装 CPUFREQ 指示器：
```
sudo apt-get install indicator-cpufreq
```
重启你的电脑并使用 Powersave 模式


#### 调整 LibreOffice 来使它更快

如果你是频繁使用 office 产品的用户，那么你会想要稍微调整默认的 LibreOffice 使它更快。这里你将调整内存选项。打开 Open LibreOffice，进入 “Tools->Options”。在那里，从左边的侧栏选择“Memory”并启用 “Systray Quickstarter” 以及增加内存分配。

如果你的计算机有很大内存空间，例如 4G 以上，可以尝试启用「系统任务栏快速启动」选项。启动该选项后，LibreOffice 的一部分将会驻留于内存当中，以快速打开文档以及进行更快速度的响应。

1. 打开 LibreOffice Writer。
2. 打开「工具」—「选项」—「内存」选项卡，或使用 Alt + F12 快捷键。
3. 勾选「启用系统任务栏快速启动」之后点击「确定」按钮即可。

一旦启用该选项，在打开文档时，将可以在系统托盘中看到 LibreOffice 图标。

另一种加快 LibreOffice 速度和响应时间的方法便是禁用 Java 运行时环境。

1. 打开 LibreOffice Writer。
2. 打开「工具」—「选项」—「高级」选项卡，或使用 Alt + F12 快捷键。
3. 取消勾选「使用 Java 运行时环境」选项。

如果你常用 LibreOffice Writer 和 LibreOffice Calc 的话，禁用 Java 并不会影响到日常功能的使用。但如果你要使用一些基于 LibreOffice 的特殊功能，可能需要重新启用 Java 才能完成任务。不过在要需要 Java 支持时，LibreOffice 会自动弹出消息提示以询问用户是否打开。

默认情况下，LibreOffice 对文档的修改记录和撤消步骤数为 100，其实我们大多数的日常文档编辑都不太会撤销 100 步的编辑，所以将撤消步骤数减少至 20 可以有效减少内存使用以提升 LibreOffice 性能。

1. 打开 LibreOffice Writer。
2. 打开「工具」—「选项」—「内存」选项卡，或使用 Alt + F12 快捷键。
3. 将「插入对象缓冲区」中的「对象的数目」调整到 20 并点击「确定」即可。


#### 移除Apt-Get的翻译包

如果你在`sudo apt-get update`之后仔细观注过终端输出，定然会在其中发现一些与语言翻译有关的行。如果您在服务器上 **只使用英文** ，就无需翻译包数据库了。
```
sudo gedit /etc/apt/apt.conf.d/00aptitude
```
将这行代码附加到文件末尾：
```
Acquire::Languages "none";
```
#### Stacer系统优化工具

Stacer 建立于采用 Electron 架构的开放网络技术，所有 Electron 附带的 baggage 也是存在的。对于一款系统优化程序来说，采用像 Electron 这样”资源匮乏“的基础来构建似乎有悖常理了点，但是考虑到不需要在后台运行，这也算不上问题，至少我们可以随用随关。

从[官网下载](https://github.com/oguzhaninan/Stacer/releases)对应安装包,然后安装
```
sudo dpkg --install Stacer*.deb
```
卸载：运行`sudo dpkg -r Stacer`

或者：

```
sudo add-apt-repository ppa:oguzhaninan/stacer -y
sudo apt-get update
sudo apt-get install stacer -y
```



### 清理

#### 清理下载的软件包

不过与你想象的可能有很大的不同，Ubuntu系统在运行时是不会产生无用垃圾的。这一点与Windows系统有很大的不同。但是我们在升级系统时，软件管理器下载的软件包，系统则不会自动删除，其实这样做也是考虑到你可能会再次安装从而加快再次安装的速度考虑。当然了，我们普通用户，一旦下载安装完毕，其安装包也就没有存在的必要了，当然如果你是要安装更新并管理一大堆电脑的系统管理员就另当别论咯。更何况，我们再次安装时，只要你选择了一个合适的软件源，那下载速度一样是飞快的。因此，我们隔一段时间就可清理一下apt-get等软件管理器下载下来的安装包咯。

我们先看一下，这些安装包占了多大空间吧。按快捷键ctrl+alt+t打开终端，输入命令
```
du  –h  /var/cache/apt/archives
```
回车之后，我们就可以看到安装包所占用的空间咯。

那我们就来删除这些软件包吧。若你生性小心谨慎，那就只删除那些你已经将其卸载掉的软件的软件。删除你已经卸载掉的软件包的命令为
```
sudo apt-get autoclean
```
若你想清理出更多的空间，而且网速又比较快的话，那你大可以把电脑上存储的安装包全部卸载咯，命令为
```
sudo apt-get clean
```
还有一类软件包，我们每个人都应该删除，那就是你已经卸载了，但是一些只有它依赖而别的软件包都不需要的软件包还留在你的系统里。说简单点就是，类似于你在windows系统中卸载软件时残留在系统里的垃圾咯。卸载这些孤立包的命令为
```
sudo apt-get autoremove
```
不过apt-get autoremove只会删除经apt-get自动安装的依赖包，而你自己手动安装的依赖包则不会被删除，这时我们可以用deborphan来彻底删除．
```
sudo apt-get install deborphan
```
列出孤儿软件包
```
sudo deborphan
```
将它们删除（谨慎）
```
deborphan | xargs sudo apt-get purge -y
```
找出系统上哪些软件包留下了残余的配置文件

```
dpkg --list | grep "^rc"
```

**rc**表示软件包已经删除（**R**emove），但配置文件（**C**onfig-file）还在. 现在提取这些软件包的名称．

```
dpkg --list | grep "^rc" | cut -d " " -f 3
```

删除这些软件包

```
dpkg --list | grep "^rc" | cut -d " " -f 3 | xargs sudo dpkg --purge
```

如果你只想删除某个软件包的配置文件，那么可以使用下面的命令

```
sudo dpkg --purge <package-name>
```

#### 清理日志文件

日志文件会变得越来越大，我们可以用ncdu工具来查看大日志文件．
```
sudo apt-get install ncdu
sudo ncdu /var/log
```
我们可以用下面的命令来清空日志文件的内容．
```
sudo dd if=/dev/null of=/var/log/<日志文件名>.log
```

#### 删除不用的老旧内核

若你的系统更新过好多次，如Ubuntu，在系统升级的过程中，其所使用Linux内核也可能更新。因此，升级多次后，你的boot文件夹就会变得比较大，其原因就是因为虽然系统更新升级了新内核，但是老内核依然留在了你的系统中。也许你会说系统太笨了，不知道升级了新的就该把老的删除吗？实际上，不删除掉老的内核也是一种安全测试。虽然说，系统升级包在释放出之前已经进行了广泛的测试，但依然可能有意外存在，所以才不删除掉老的内核，以便于使用新升级的内核无法启动时，你能马上使用老内核进行启动，不至于导致你无法进入系统的悲剧。不过在你升级完毕，重启后能进入系统后，说明新内核已经很好的兼容了你的电脑，那么你就可以放心大胆的删除掉老内核咯，也好腾出更多空间让你使用哦。

不过老内核时一定要小心，那就是——千万不要删错咯。所以删除之前要先看一看你现在正在使用的内核是哪一个。方法是在终端中输入命令
```
uname --r
```
然后看其显示的内核版本是多少。看准了自己使用的内核后，你就可以放心大胆的删除那些不用的老内核。

打开终端，敲入命令
```bash
dpkg --get-selections | grep linux
```
然后将不用的内核文件image、头文件headers删除掉就可以咯。在终端中输入命令
```bash
sudo apt-get purge  内核文件名  头文件名
#如
sudo apt-get purge  *-4.18.0-22-*
```
删除内核后，就可以省下很多空间

之后在使用dpkg --get-selections|grep linux命令查看一下是不是已经删除了呢。

#### baobab硬盘空间用量分析工具
baobab是一个图形界面工具，可以帮助我们查找系统中哪个目录或文件占据了大量空间．在终端里运行下面的命令
```
sudo baobab
```
其实我们也可以用上面所提到了ncdu工具来查看大容量目录和文件．比如查看`/home/<username>/`
```
sudo ncdu /home/<username>
```
不过用ncdu的话，每查看一个目录就要输入一次命令，建议在服务器上用ncdu，在桌面版本用图形化的baobab工具．

#### 删除大容量软件包
首先安装debian-goodies
```
sudo apt-get install debian-goodies
```
然后输入下面的命令
```
dpigs -H
```
接下来你就可以删除你不用的软件包了．上面的命令默认只会显示前10个结果，你可指定结果的个数，比如20个
```
dpigs -H --lines=20
```
#### 清除已卸载软件的残留配置文件
在我们使用系统的过程中，有时候需要把不用的软件给卸载掉。若你无需再次安装该软件，可以把软件的配置文件也清理掉，此时在卸载软件的时候，尽可能使用sudo apt-get purge xxxxx（xxxx为要卸载的软件名），这样可以将软件以及它的配置文件均卸载干净。

不过由于这样或那样的原因，系统中有时候会残留下某些已卸载软件的配置文件。如果想清除掉这些残留的配置文件，我们可以使用一款常见的软件来完成——新力得软件包管理器（synaptic）来清除已卸载软件的残留配置文件

这个方法适用于多种Ubuntu以及Debain系的Linux系统，如Mint以及深度Linux系统

#### 命令行下rm的垃圾箱

在/bin下创建saferm.sh文件，写入下面的内容

```bash
#!/bin/bash
##
## saferm.sh
## Safely remove files, moving them to GNOME/KDE trash instead of deleting.
## Made by Eemil Lagerspetz
## Login   <vermind@drache>
## 
## Started on  Mon Aug 11 22:00:58 2008 Eemil Lagerspetz
## Last update Sat Aug 16 23:49:18 2008 Eemil Lagerspetz
##

version="1.16";

## flags (change these to change default behaviour)
recursive="" # do not recurse into directories by default
verbose="true" # set verbose by default for inexperienced users.
force="" #disallow deleting special files by default
unsafe="" # do not behave like regular rm by default

## possible flags (recursive, verbose, force, unsafe)
# don't touch this unless you want to create/destroy flags
flaglist="r v f u q"

# Colours
blue='\e[1;34m'
red='\e[1;31m'
norm='\e[0m'

## trashbin definitions
# this is the same for newer KDE and GNOME:
trash_desktops="$HOME/.local/share/Trash/files"
# if neither is running:
trash_fallback="$HOME/Trash"

# use .local/share/Trash?
use_desktop=$( ps -U $USER | grep -E "gnome-settings|gnome-session|startkde|mate-session|mate-settings|mate-panel|gnome-shell|lxsession|unity" )

# mounted filesystems, for avoiding cross-device move on safe delete
filesystems=$( mount | awk '{print $3; }' )

if [ -n "$use_desktop" ]; then
    trash="${trash_desktops}"
    infodir="${trash}/../info";
    for k in "${trash}" "${infodir}"; do
        if [ ! -d "${k}" ]; then mkdir -p "${k}"; fi
    done
else
    trash="${trash_fallback}"
fi

usagemessage() {
	echo -e "This is ${blue}saferm.sh$norm $version. LXDE and Gnome3 detection.
    Will ask to unsafe-delete instead of cross-fs move. Allows unsafe (regular rm) delete (ignores trashinfo).
    Creates trash and trashinfo directories if they do not exist. Handles symbolic link deletion.
    Does not complain about different user any more.\n";
	echo -e "Usage: ${blue}/path/to/saferm.sh$norm [${blue}OPTIONS$norm] [$blue--$norm] ${blue}files and dirs to safely remove$norm"
	echo -e "${blue}OPTIONS$norm:"
	echo -e "$blue-r$norm      allows recursively removing directories."
	echo -e "$blue-f$norm      Allow deleting special files (devices, ...)."
  echo -e "$blue-u$norm      Unsafe mode, bypass trash and delete files permanently."
	echo -e "$blue-v$norm      Verbose, prints more messages. Default in this version."
  echo -e "$blue-q$norm      Quiet mode. Opposite of verbose."
	echo "";
}

detect() {
    if [ ! -e "$1" ]; then fs=""; return; fi
    path=$(readlink -f "$1")
    for det in $filesystems; do
        match=$( echo "$path" | grep -oE "^$det" )
        if [ -n "$match" ]; then
            if [ ${#det} -gt ${#fs} ]; then
                fs="$det"
            fi
        fi
    done
}


trashinfo() {
#gnome: generate trashinfo:
	bname=$( basename -- "$1" )
    fname="${trash}/../info/${bname}.trashinfo"
    cat <<EOF > "${fname}"
[Trash Info]
Path=$PWD/${1}
DeletionDate=$( date +%Y-%m-%dT%H:%M:%S )
EOF
}

setflags() {
    for k in $flaglist; do
	reduced=$( echo "$1" | sed "s/$k//" )
	if [ "$reduced" != "$1" ]; then
	    flags_set="$flags_set $k"
	fi
    done
  for k in $flags_set; do
	if [ "$k" == "v" ]; then
	    verbose="true"
	elif [ "$k" == "r" ]; then 
	    recursive="true"
	elif [ "$k" == "f" ]; then 
	    force="true"
	elif [ "$k" == "u" ]; then 
	    unsafe="true"
	elif [ "$k" == "q" ]; then 
    unset verbose
	fi
  done
}

performdelete() {
			# "delete" = move to trash
			if [ -n "$unsafe" ]
			then
			  if [ -n "$verbose" ];then echo -e "Deleting $red$1$norm"; fi
		    #UNSAFE: permanently remove files.
		    rm -rf -- "$1"
			else
			  if [ -n "$verbose" ];then echo -e "Moving $blue$k$norm to $red${trash}$norm"; fi
		    mv -b -- "$1" "${trash}" # moves and backs up old files
			fi
}

askfs() {
  detect "$1"
  if [ "${fs}" != "${tfs}" ]; then
    unset answer;
    until [ "$answer" == "y" -o "$answer" == "n" ]; do
      echo -e "$blue$1$norm is on $blue${fs}$norm. Unsafe delete (y/n)?"
      read -n 1 answer;
    done
    if [ "$answer" == "y" ]; then
      unsafe="yes"
    fi
  fi
}

complain() {
  msg=""
  if [ ! -e "$1" -a ! -L "$1" ]; then # does not exist
    msg="File does not exist:"
	elif [ ! -w "$1" -a ! -L "$1" ]; then # not writable
    msg="File is not writable:"
	elif [ ! -f "$1" -a ! -d "$1" -a -z "$force" ]; then # Special or sth else.
    	msg="Is not a regular file or directory (and -f not specified):"
	elif [ -f "$1" ]; then # is a file
    act="true" # operate on files by default
	elif [ -d "$1" -a -n "$recursive" ]; then # is a directory and recursive is enabled
    act="true"
	elif [ -d "$1" -a -z "${recursive}" ]; then
		msg="Is a directory (and -r not specified):"
	else
		# not file or dir. This branch should not be reached.
		msg="No such file or directory:"
	fi
}

asknobackup() {
  unset answer
	until [ "$answer" == "y" -o "$answer" == "n" ]; do
	  echo -e "$blue$k$norm could not be moved to trash. Unsafe delete (y/n)?"
	  read -n 1 answer
	done
	if [ "$answer" == "y" ]
	then
	  unsafe="yes"
	  performdelete "${k}"
	  ret=$?
		# Reset temporary unsafe flag
	  unset unsafe
	  unset answer
	else
	  unset answer
	fi
}

deletefiles() {
  for k in "$@"; do
	  fdesc="$blue$k$norm";
	  complain "${k}"
	  if [ -n "$msg" ]
	  then
		  echo -e "$msg $fdesc."
    else
    	#actual action:
    	if [ -z "$unsafe" ]; then
    	  askfs "${k}"
    	fi
		  performdelete "${k}"
		  ret=$?
		  # Reset temporary unsafe flag
		  if [ "$answer" == "y" ]; then unset unsafe; unset answer; fi
      #echo "MV exit status: $ret"
      if [ ! "$ret" -eq 0 ]
      then 
        asknobackup "${k}"
      fi
      if [ -n "$use_desktop" ]; then
          # generate trashinfo for desktop environments
        trashinfo "${k}"
      fi
    fi
	done
}

# Make trash if it doesn't exist
if [ ! -d "${trash}" ]; then
    mkdir "${trash}";
fi

# find out which flags were given
afteropts=""; # boolean for end-of-options reached
for k in "$@"; do
	# if starts with dash and before end of options marker (--)
	if [ "${k:0:1}" == "-" -a -z "$afteropts" ]; then
		if [ "${k:1:2}" == "-" ]; then # if end of options marker
			afteropts="true"
		else # option(s)
    		    setflags "$k" # set flags
    	        fi
	else # not starting with dash, or after end-of-opts
		files[++i]="$k"
	fi
done

if [ -z "${files[1]}" ]; then # no parameters?
	usagemessage # tell them how to use this
	exit 0;
fi

# Which fs is trash on?
detect "${trash}"
tfs="$fs"

# do the work
deletefiles "${files[@]}"
```

赋予saferm.sh执行权限：

```bash
$ sudo chmod a+x /bin/saferm.sh
```

然后编辑用户目录下的.bashrc文件

```bash
$ sudo vim ~/.bashrc 

#增加
alias rm="saferm.sh"
```

然后就可以在命令行中使用rm命令，对于文件夹使用`rm -r`,对于无界面的Linux，垃圾箱位于`~/Trash`;对于有界面的linux，垃圾箱位于`~/.local/share/Trash/`。如果有的界面未识别，可以在

```shell
# use .local/share/Trash?
use_desktop=$( ps -U $USER | grep -E "gnome-settings|gnome-session|startkde|mate-session|mate-settings|mate-panel|gnome-shell|lxsession|unity" )
```

添加您的桌面类型。

### 问题

#### 网络

**关于有线连接无法连接问题**

这点是ubuntu一直存在的bug，经常发生插上网线，桌面不显示以太网已连接，有的是显示有线存在，但是点击连接，连接不上

这个bug ubuntu18.04_gnome尤其严重。主要是没有主动分配到ip

解决办法:`sudo dhclient enp1s0`,通过dhcp动态分配ip，尽管桌面上无显示，但是ifconfig查看已分配ip，这里的enp1s0是我的有线网卡名称

**关于dsl网络链接**

18.04网络管理没有dsl链接方式，可以通过pppoe方法实现，但仅仅启动了ipv4,可以通过ipv4上网

关于ppp0只有ipv4,开启ipv6不能正常访问网络问题

在/etc/ppp/options最后添加如下内容，并重启网络
```
+ipv6 ipv6cp-use-ipaddr
```
详见，[ubuntu18.04(gnome3) dsl(pppoe)连接网络](https://blog.csdn.net/scylhy/article/details/80113536)

**ipv4下使用ipv6**

安装miredo,不稳定

**关于线缆已拔出**

修改`/etc/NetworkManager/NetworkManager.conf` 中
```
[ifupdown]
mananged=true
```
重启网络管理服务
```
sudo service network-manager restart
```
这时，会显示有线链接

**使用timeshift回转后开机卡在Loding initial ramdisk**

原因：timeshift保存镜像时保存了以前的内核文件或者配置，而现在的ubuntu已经更新到新的内核版本，在恢复镜像之后，会导致内核和配置不统一，bios认为磁盘上有新内核，但实际上没有，以致于一直卡在开机界面。

处理方法：在开机bios选项--》选择ubuntu高级选项--》选择第二个不带recovery的内核--》正常开机。

运行命令：

```bash
sudo apt upgrade -y
sudo update-grub2
sudo reboot
```

ok！！！正常开机。

为了确保不再出错，删除旧的内核后再更新内核，删除方法参考上面。

最后，使用TimeShift保存恢复后的系统，删除以前的镜像。

并且始终铭记这种错误，不要慌。

#### 使用

**普通用户可以执行，sudo提示command not found**

出于安全方面的考虑，使用sudo执行命令将在一个最小化的环境中执行，环境变量都重置成默认状态。所以PATH这个变量又不包括附加的执行目录了.

可以使用printenv这个命令，检查当前的PATH变量.
```
pritnenv PATH
sudo printenv PATH
```
为什么要这么设计，因为$PATH变量是一个很危险的环境变量，例如下面这篇文章就讲了黑客如何利用$PATH变量获得root权限

[利用 PATH 环境变量进行 Linux 提权](http://www.52bug.cn/hkjs/5034.html)

《UNIX&LINUX大学教程》这本书里也提到，如果$PATH变量中包括了一些谁都有权限读写的目录，黑客可以通过撰写名为cd、ls之类的脚本，这样原来的系统命令就被替换为了黑客命令

常用方法：

1. 修改/etc/sudoers文件的secure_path，或者使用visudo命令，就可以打开/etc/sudoers文件进行修改，在secure_path后面加入pip的执行路径，我这里是/usr/local/bin，保存即可生效。

2. 把路径写完整（推荐）

3. env命令，在用户目录下的.bashrc中添加以下别名。
```
alias sudo="sudo env PATH=$PATH"
```

#### 解决ubuntu无法调整和保存屏幕亮度的问题

ubuntu无法调整屏幕亮度，对笔记本来说很耗电，同时也很刺眼，因为它是默认以最大亮度来工作的。

所谓的调整，方法为下面的其中一种：

1. Fn＋左右的快捷键，亮度没有变化
2. 在亮度与锁屏中拉动进度条亮度没有变化

```
fn调节的是/sys/class/backlight/acpi_video0/brightness文件

而I卡的文件是/sys/class/backlight/intel_backlight/brightness。
```

##### 什么是i卡？

三种主要品牌显卡： Nvidia ， AMD/ATI 和 Intel

1. Nvidia

   提供最基本的仅支持 2D 的开源驱动(只提供闭源驱动)。但闭源驱动的性能非常好，与 Windows 上的性能几乎差不多。而且 Nvidia 的驱动更新很频繁，而且他们还会使用 VDPAU 加速 API 来提供快速视频加速，这个加速 API 功能仅被当前最新的 Adobe Flash beta 支持。所以，如果你经常观看全屏高清视频的话，一块 Nvidia 显卡加上他们的驱动应该是最佳方案了。但是 Nvidia 至今还不支持 Xrandr 协议，Xrandr 协议可以允许 X 来调整显示分辨率，或者扩展/克隆到外部显示器。

2. AMD/ATI

   在 AMD 收购 ATI 之前，可以说在 Linux 上基本没有像样的 ATI 驱动。不过自从被 AMD 收购后，情况就变得大为不同。ATI 的闭源 Linux 驱动有了跨越式的发展，而且还支持 Xrandr 协议，这样你就可以完全使用 Ubuntu 内置分辨率调整工具了。而且在性能方面也非常好，也可以与 Wine 一起很好的工作。AMD 在 Linux 驱动方面确实贡献卓越。当然有一点与 Nividia 驱动相似的，那就是也不支持 KMS 。闭源的 AMD 驱动使用与 Nvidia 不同的视频 API ，而是唤作的 VA-API，不幸的是 Adobe 目前至今还没有支持它，所以基于 Flash 的高清视频受到一定的影响。另外与 Nvidia 相比欠缺的一点是，AMD 驱动需要花费更多的时间来支持新版内核及新的 X Server 版本，但对于 Ubuntu 用户来说并不是问题，因为它会默认搭载在 Ubuntu 发行版中

3. Intel

   可以说， Intel 是开源 Linux 图形卡驱动方面的王者，他们只发布 Linux 平台上的开源驱动，这也意味着你能体验到像 KMS 及 Xrandar 支持这样的所有功能。但 Intel 也并不完美，如果你拥有一块基于 GMA500 的卡的话，它基本上无法工作于 Ubuntu 上，因为这是英特尔购买了其他公司的芯片组后并更名了它，而且他们也不能为其开发开源驱动，虽然目前英特尔还在解决此问题。Intel 的另外一个最大缺点是他们的硬件性能远远不如 AMD 和 Nvidia ，并且对于游戏支持也不够好。


如果对于你来说有开源驱动是非常重要的事，那么你可以用 Intel 或 AMD 的卡；如果你更关注性能，那么你可以用 AMD 或 Nvidia 的卡。

总的来说， AMD/ATI 是更加前沿，更加值得推荐，因为他们在提供稳定开源驱动的同时，还提供了可靠快速的闭源驱动，堪称两全其美。

##### 解决办法

回来原来的问题

一种比较将就的方法就是刚开机的时候就按Fn＋左右键，这样就可以改变亮度了。一旦进去之后就不可以改变了。

比较完美的方法如下：

1. 配置grub使得fn可以调节亮度

   首先，修改grub的用户配置文件, /etc/default/grub

   注意该配置文件是grub2才有的, 首先你必须明确你的grub版本是多少, 较低的版本没有这个设置文件
   更过关于配置文件的信息,请参见

   [修改系统启动项 grub2配置的方法 ubuntu](http://www.cnblogs.com/saptechnique/archive/2012/04/05/2433643.html)

   [grub2的/etc/default/grub文件详解](http://blog.chinaunix.net/uid-26495963-id-3058498.html)

   ```
   sudo vim /etc/default/grub 
   ```


   将`GRUB_CMDLINE_LINUX=""`改为`GRUB_CMDLINE_LINUX="acpi_backlight=vendor"`
   或者使用下面更复杂的配置

   ```
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" 
   GRUB_CMDLINE_LINUX=""
   ```


   修改为

   ```
   acpi_osi=Linux acpi_backlight=vendor
   GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash acpi_osi=Linux”
   GRUB_CMDLINE_LINUX="acpi_backlight=vendor“
   ```


   更新grub.cfg

   ```
   sudo update-grub
   ```


    查看grub.cfg，可以发现每个启动项都加入了`”acpi_backlight=vendor`”

   ```
   grub.cfg位于/boot/grub/grub.cfg
   ```

2. 其实用screen就可以完美解决！

   要使用screen，请先执行

   ```
   sudo apt-get install screen
   ```

更多参考：https://blog.csdn.net/weixin_43599336/article/details/85981442

Intel的电脑支持：https://github.com/intel/compute-runtime

### 跑分

UnixBench是一款开源的测试 unix 系统基本性能的工具,是比较通用的测试VPS性能的工具。

UnixBench会测试系统各个方面一系列的性能,然后将每个测试结果和一个基准值进行比较,得到一个索引值,所有测试项目的索引值结合在一起形成一个测试分数值.

UnixBench也支持多CPU系统的测试，默认的行为是测试两次，第一次是一个进程的测试，第二次是N份测试，N等于CPU个数。

基本测试项如下:

    简单的2D和3D图形测试
    测试系统的单任务性能
    测试系统的多任务性能
    测试系统并行处理的能力
    CPU,内存,或者磁盘

UnixBench一个基于系统的基准测试工具，不单纯是CPU 内存 或者磁盘测试工具。测试结果不仅仅取决于硬件，也取决于系统、开发库、甚至是编译器。

UnixBench测试执行完大约需要10-30分钟.

UnixBench 参考 Linux性能测试UnixBench一键脚本：

```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/unixbench.sh

chmod +x unixbench.sh

sudo ./unixbench.sh
```

我的跑分如下：

```bash
========================================================================
   BYTE UNIX Benchmarks (Version 5.1.3)

   System: hyp-HP-Notebook: GNU/Linux
   OS: GNU/Linux -- 4.18.0-17-generic -- #18~18.04.1-Ubuntu SMP Fri Mar 15 15:27:12 UTC 2019
   Machine: x86_64 (x86_64)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   CPU 0: Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz (4800.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET, Intel virtualization
   CPU 1: Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz (4800.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET, Intel virtualization
   CPU 2: Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz (4800.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET, Intel virtualization
   CPU 3: Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz (4800.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET, Intel virtualization
   10:10:44 up 6 min,  1 user,  load average: 0.74, 0.67, 0.34; runlevel 5

------------------------------------------------------------------------
Benchmark Run: 五 5月 10 2019 10:10:44 - 10:38:50
4 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables       28416931.3 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     4525.4 MWIPS (9.6 s, 7 samples)
Execl Throughput                               4132.6 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        611296.1 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          164625.2 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       1701905.9 KBps  (30.0 s, 2 samples)
Pipe Throughput                              941048.4 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 174782.3 lps   (10.0 s, 7 samples)
Process Creation                              10212.0 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   8821.3 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                   2200.5 lpm   (60.0 s, 2 samples)
System Call Overhead                         694356.1 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   28416931.3   2435.0
Double-Precision Whetstone                       55.0       4525.4    822.8
Execl Throughput                                 43.0       4132.6    961.1
File Copy 1024 bufsize 2000 maxblocks          3960.0     611296.1   1543.7
File Copy 256 bufsize 500 maxblocks            1655.0     164625.2    994.7
File Copy 4096 bufsize 8000 maxblocks          5800.0    1701905.9   2934.3
Pipe Throughput                               12440.0     941048.4    756.5
Pipe-based Context Switching                   4000.0     174782.3    437.0
Process Creation                                126.0      10212.0    810.5
Shell Scripts (1 concurrent)                     42.4       8821.3   2080.5
Shell Scripts (8 concurrent)                      6.0       2200.5   3667.5
System Call Overhead                          15000.0     694356.1    462.9
                                                                   ========
System Benchmarks Index Score                                        1191.8

------------------------------------------------------------------------
Benchmark Run: 五 5月 10 2019 10:38:50 - 11:07:15
4 CPUs in system; running 4 parallel copies of tests

Dhrystone 2 using register variables       70170307.2 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                    14926.5 MWIPS (9.6 s, 7 samples)
Execl Throughput                               9576.2 lps   (29.8 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks       1044346.7 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          270007.6 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       2763055.8 KBps  (30.0 s, 2 samples)
Pipe Throughput                             2683502.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 564190.4 lps   (10.0 s, 7 samples)
Process Creation                              27142.4 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                  15357.6 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                   1592.7 lpm   (60.1 s, 2 samples)
System Call Overhead                        2271195.1 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   70170307.2   6012.9
Double-Precision Whetstone                       55.0      14926.5   2713.9
Execl Throughput                                 43.0       9576.2   2227.0
File Copy 1024 bufsize 2000 maxblocks          3960.0    1044346.7   2637.2
File Copy 256 bufsize 500 maxblocks            1655.0     270007.6   1631.5
File Copy 4096 bufsize 8000 maxblocks          5800.0    2763055.8   4763.9
Pipe Throughput                               12440.0    2683502.1   2157.2
Pipe-based Context Switching                   4000.0     564190.4   1410.5
Process Creation                                126.0      27142.4   2154.2
Shell Scripts (1 concurrent)                     42.4      15357.6   3622.1
Shell Scripts (8 concurrent)                      6.0       1592.7   2654.6
System Call Overhead                          15000.0    2271195.1   1514.1
                                                                   ========
System Benchmarks Index Score                                        2536.9

======= Script description and score comparison completed! ======= 

```

### 自定义手势

转Ubuntu系统后以后终于领会到两个系统间触控板控制的差距，在Windows中可以使用二指、三指和四指完成页面滚动、页面放大缩小、程序间的切换等一些基本操作，但是在Ubuntu中只能通过两指完成页面滚动功能。

在Ubuntu系统中长按Win(super)键可以查看一些Ubuntu的快捷键，在系统的**setting - keyboard**选项下面可以查看和设置这些快捷键，在命令行终端的**edit -preference**选项下面也可以设置命令行终端的一些快捷键。

[fusuma](https://github.com/iberianpig/fusuma)是一个实现多指控制器的包，能够识别手指在触控板上的扫动动作(swipe)和捏动作(pinch)。

1. 给当前用户访问触控板的权限

```
$ sudo gpasswd -a $USER input
```

也可以用命令

```
$ sudo usermod -a -G input $USER
```

在执行这一步后需要重新登录账户。

2. 安装fusuma包和相关依赖包

   ```
   $ sudo apt-get install libinput-tools 
   $ sudo gem install fusuma
   $ sudo apt-get install xdotool
   ```

3. 确保触控板的info传输到GNOME桌面环境

   ```
   $ gsettings set org.gnome.desktop.peripherals.touchpad send-events enabled
   ```

4. 配置自定义手势

   这里在.config下面新建fusuma文件夹并在其中添加YML配置文件。

   ```
   $ mkdir -p ~/.config/fusuma
   $ gedit ~/.config/fusuma/config.yml
   ```

Github上面提供了centos的自定义手势，但是不能在Ubuntu上面使用，这里我根据自己的需求设置了Ubuntu的自定义手势。

二指：上下左右：页面滑动  拉近拉远：放大缩小

三指：上：显示所有运行的窗口 下：显示桌面 左：返回到前一个界面 右：前进到下一个界面(浏览器中适用)

四指：上：显示搜索窗口 下：暂且设置为返回桌面 左右：选择文件夹中的文件

需要说明的是，这里的threshold表示键盘检测的敏感度，interval表示程序检测连续两次键盘手势的时间间隔，default值均为1，但是我想要更好的检测效果，将两个参数都设置成比较小的值。

```
swipe:
  3: 
    left: 
      command: 'xdotool key alt+Left'
    right: 
      command: 'xdotool key alt+Right'
    up:
      command: 'xdotool key super+W'
    down: 
      command: 'xdotool key ctrl+super+d'
  4:
    left: 
      command: 'xdotool key super+Left'
    right: 
      command: 'xdotool key super+Right'
    up: 
      command: 'xdotool key super+a'
    down: 
      command: 'xdotool key ctrl+super+d'
pinch:
  2:
    in:
      command: 'xdotool key ctrl+plus'
      threshold: 0.1
    out:
      command: 'xdotool key ctrl+minus'
      threshold: 0.1
 
threshold:
  swipe: 0.1
  pinch: 0.1
 
interval:
  swipe: 0,1
  pinch: 0.1
```

