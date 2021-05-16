---
title: Ubuntu 常用软件
date: 2019-04-22 12:55:22
tags:
 - Linux
 - Ubuntu
category:
 - Linux
 - Ubuntu
---
所谓工欲善其事，必先利其器，好的软件或工具，能够在很大程度上提升你的工作效率。对于像我这样把 Linux 直接安装在物理机上当作日常操作系统来使用的人来说，能够找到并成功安装一些 Linux 版的日常软件以及效率软件，是一件多么令人兴奋的事情。这里经过我多年的收集积累，罗列出了一些实用的 Linux 软件，希望能够对大家有所参考价值，另外也作为自己的一份笔记，方便日后重装系统后来查阅安装。

------

> **温馨提示：** 以下安装步骤基于 ubuntu，其他发行版原理类似。对于需要通过 `apt-add-repository` 添加第三方镜像源的安装步骤

#### **命令行软件**

**进程与资源监视器（Htop）**

htop 是 top 的替代品，具有更炫酷更直观的界面效果，能够查看 CPU、内存占用情况以及进程管理等。

```javascript
sudo apt install htop
```

------

**命令帮助工具（tldr）**

tldr 是 man 的替代品，更具有可读性，并且帮助信息简单明了，色彩分明，更为人所接受。

```javascript
sudo apt install tldr
```

------

**Git 命令优化工具（Tig）**

tig 是一款优化 git 命令行的工具，具有更丰富更可观的界面，可在 git 项目目录中使用。

```javascript
sudo apt install tig
```

------

**命令更正提示工具（TheFuck）**

TheFuck 是一款特别棒的命令行工具，可以纠正你输入的错误的命令。如果你忘记命令了，但是记得个大概，你可以大胆输入你的命令，如果输错了，就来句 `fuck`，然后 TheFuck 就会告诉你正确的命令如何使用。

```javascript
#CentOS系统
yum -y update && yum -y install gcc 
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py && yum -y install python-devel
sudo -H pip install thefuck
 
#Ubuntu/Debian系统
sudo apt update
sudo apt install python3-dev python3-pip
sudo pip3 install thefuck


#编辑bashrc配置文件
vim ~/.bashrc
#在文件尾加入一行给thefuck取别名fuck
alias fuck='thefuck'
#使生效
source ~/.bashrc
```

------

**高速下载工具（Axel）**

Axel 是 Linux 下一个不错的 HTTP/FTP 高速下载工具。支持多线程下载、断点续传，且可以从多个地址或者从一个地址的多个连接来下载同一个文件。适合网速不给力时多线程下载提高下载速度。是 wget 命令的替代品。

```javascript
sudo apt install axel
```

------

**终端复用神器（Tmux）**

Tmux 可从一个屏幕上管理多个终端（伪终端）。使用该工具，用户可以连接或断开会话，而保持终端在后台运行。

```javascript
sudo apt install tmux
```

------

**文件查看工具（Bat）**

bat 是一款可用在 Linux 命令行显示文件内容的工具，是 cat 命令的一个替代品，bat 功能要更加强大一些，其在 cat 命令的基础上加入了行号显示、代码高亮和 Git 集成等。

```javascript
 下载地址：https://github.com/sharkdp/bat/releases
```

------

**MySQl** **命令优化工具（MyCli）**

mycli 是 mysql 命令的替代品，该工具可以高亮显示 SQL 命令，并且具有自动补全功能，可以大大提高操作 MySQL 数据库的效率，也让人使用起来更加舒适。

```javascript
sudo apt install mycli
```

**自动补全： bpython**

[Bpython](https://bpython-interpreter.org/) 是对 Python REPL 的一个很好的替代。当我运行 bpython 并开始输入时，建议会立即出现。我没用通过特殊的键盘绑定触发它，甚至没有按下 `Tab` 键。

![UTOOLS1572072843107.png](https://i.loli.net/2019/10/26/WBpUj8tZrvSuzMI.png)

当我出于习惯按下 `Tab` 键时，它会用列表中的第一个建议补全。这是给 CLI 设计带来可发现性性的一个很好的例子。

bpython 的另一个方面是可以展示模块和函数的文档。当我输入一个函数的名字时，它会显示这个函数附带的签名以及文档字符串。这是一个多么令人难以置信的周到设计啊。

**模糊搜索和在线帮助： pgcli**

如果您正在寻找 PostgreSQL 版本的 mycli，请看看 [pgcli](http://pgcli.com/)。 与 mycli 一样，它提供了上下文感知的自动补全。菜单中的项目使用模糊搜索缩小范围。模糊搜索允许用户输入整体字符串中的任意子字符串来尝试找到正确的匹配项。

![UTOOLS1572072885736.png](https://i.loli.net/2019/10/26/qylrVeEKSTOM8Gg.png)

pgcli 和 mycli 在其 CLI 中都实现了这个功能。斜杠命令的文档也作为补全菜单的一部分展示。

#### **图形化软件**

**系统优化工具（Gnome-Twaeks）**

该软件仅适用于 Gnome 桌面环境，可以配置系统字体、主题、图标等，可管理 Gnome 插件、设置开机启动程序，以及进行其他的一些系统优化操作。

```javascript
 # 安装
 sudo apt-get install gnome-tweak-tool
 
 # 运行
 gnome-tweaks
```

**快速启动工具（Albert）**

类似与 Windows 下的 Wox 软件，可通过一个快捷键调出居中于桌面的搜索框，可以秒速搜索电脑上的软件、文件等，也可以配置相关插件获得更强大的功能（例如快速搜索 Google、百度、GitHub等）。

```javascript
 # 安装
 sudo add-apt-repository ppa:noobslab/macbuntu
 sudo apt update
 sudo apt install albert
```

可在 Gnome-Tweaks 中配置为开机启动，在搜索框右上角有个设置图标，可配置快捷键。

**快速启动工具（Synapse）**

这款快速启动工具感觉比 Albert 还要强大一些，搜索文件也是挺快的，并且插件更加丰富。

```javascript
 sudo apt install synapse
```

感觉是不是很炫酷，很值得试一试。

**socks 代理工具（Shadow socks-Qt5）**

```javascript
 下载地址：https://github.com/shadowsocks/shadowsocks-qt5/releases
```

下载后是 `.AppImage`结尾的文件，赋予文件可执行权限，然后直接运行就可以。

如果要添加桌面图标，同样可以参照 `.desktop` 文件书写格式建一个图标文件并存放到 `/usr/share/applications`目录下。 

**文件搜索软件（catfish）**

类似于 Windows 下的 Everything 这个软件，不过亲测搜索速度一般般，跟 Everything 压根没法比，但是它有个强大的功能就是可以搜文件里面的内容，例如你某个文件里写了某行代码，你只记得那行代码但是记不得文件放哪了，你就可以用这个软件来搜索那个文件了。

```javascript
 sudo apt install catfish
```

**垃圾清理工具（BleachBit）**

BleachBit 是一款 Linux 下比较好用的且功能强大的图形化垃圾清理工具，含有很多清理选项，还能够粉碎文件、擦除空闲空间等操作。

```javascript
 sudo apt install bleachbit
```

**Gif 录制软件（peek）**

```javascript
 sudo apt install peek
```

录制的效果还挺好的，文件也不算大。

**截图工具（FlameShot）**

这是一款强大的截图软件，设置比 QQ 截图还强大，中文名叫火焰截图。截取的图像可保存文件或复制到剪切板，也可以对图像进行编辑。

```javascript
 # 安装
 sudo apt install flameshot
 # 配置
 sudo flameshot config
 # 运行
 flameshot gui
```

可在系统设置中添加截图快捷键，在 Gnome-Tweaks 中添加开机启动。

**图像处理软件（GIMP）**

这是一款 Linux 下的 Photoshop 替代品，功能强大到堪比 Photoshop。

```javascript
 # 安装
 sudo add-apt-repository ppa:otto-kesselgulasch/gimp
 sudo apt-get update
 sudo apt-get install gimp gimp-plugin-registry gimp-data-extras
 
 # 运行
 gimp
```

**思维导图软件（XMind）**

XMind 是一款跨平台的思维导图软件，当然也有 Linux 版的。安装 XMind 需要到其官网下载 deb 安装包。

```javascript
 下载地址：https://www.xmind.cn/download/
```

可直接在软件管理器中安装，也可以通过 `dpkg -i xxxx.deb` 命令来安装。 

**知识树笔记工具（CherryTree）**

```javascript
 sudo apt install cherrytree
```

**Markdown 编辑器（Typora）**

Typora 是一款跨平台的，即写即显的 Markdown 编辑器，其不仅使用起来特别方便，而且也有特别多可供选择的主题，还能够轻松导出 HTML 和 PDF。

```javascript
 # 安装
 wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
 sudo add-apt-repository 'deb https://typora.io/linux ./'
 sudo apt-get update
 sudo apt-get install typora
```

> 注意：如果以上方法无法安装，可以到 Typora 官方下载二进制包进行安装。 下载地址：https://www.typora.io/#linux 下载完成后解压，进入到解压目录后可以直接执行命令 `./Typora` 启动。 如果要创建桌面图标，可以参照 `.desktop` 文件的书写格式写，然后放到 `/usr/share/applications` 目录下即可。

**代码编辑器（Sublime Text）**

Sublime Text 就不解释了，几乎每一个程序猿都对它爱不释手，尤其是搞前端开发的。可以直接去其官网下载二进制软件包。

```javascript
 下载地址：http://www.sublimetext.com/3
```

下载完后解压，然后进入解压后的文件夹，可通过命令 `./sublime_text` 启动程序。

如果想要添加桌面图标的话，可直接将解压目录中的 `sublime_text.desktop` 文件复制到 `/usr/share/applications` 目录下。 

**代码编辑器（VS Code）**

Visual Studio Code 也是一款功能强大的代码编辑器，其插件也是相当的丰富，可媲美 Sublime Text。可通过下载 deb 软件包进行安装。

```javascript
 下载地址：https://code.visualstudio.com/
```

**十六进制编辑器（bless）**

可通过十六进制编辑文件。

```javascript
 sudo apt install bless
```

**远程桌面工具（TeamViewer）**

TeamViewer 这款远程桌面软件在业界所享有的美誉我想不用我做太多的介绍了，相比其他远程桌面工具，它不仅画面流畅清晰，而且也很省流量带宽，除了远程桌面，它还有远程会议、桌面分享、远程打印、VPN连接等功能。其下载方式也是去官网下载 deb 软件包进行安装。

```javascript
 下载地址：https://www.teamviewer.cn/cn/download/linux/
```

**远程连接管理工具（remmina）**

该软件可管理 DRP、VNC、SSH 等协议的远程连接。

```javascript
 sudo apt install remmina
```

**FTP 客户端（filezilla）**

类似于 Windows 下的 WinScp 软件，可以直接拖动文件进行本地与服务器之间的文件传输。

```javascript
 sudo apt install filezilla
```

**社区帐号管理工具（Franz）**

一款能够同时登录各种社区平台（网盘、邮箱、推特等）帐号，并接收来自这些帐号的消息等。下载 `.AppImage`程序即可。

```javascript
 下载地址：https://meetfranz.com/#download
```

**社区帐号管理工具（Station）**

类似 Franz，界面更好看，也是能够同时登录多方帐号。

```javascript
 下载地址：https://getstation.com/
```

**虚拟机软件（VirtualBox）**

如果你想要安装多台虚拟计算机作为开发测试使用的环境，那么 VirtualBox 虚拟机软件是个不错的选择，VirtualBox 在 Linux 下其优势尤为突出。同样，你需要在 Oracle VM VirtualBox 官方上下载安装程序。

```javascript
 下载地址：https://www.virtualbox.org/wiki/Linux_Downloads
```

如果上面这个列表中没有使用你的操作系统的，例如 Kali Linux 系统，这时候下载最后一个 `All distributions` 即可。这时候下载到的并不是 deb 软件包，而是以 `.run` 结尾的可执行程序，直接运行该程序即可安装。 

说到 VirtualBox，这里不得不提到它强大的一个功能，即无缝模式。例如当你的虚拟计算机为 Windows 操作系统时，开启无缝模式后，你可以直接在你的 Linux 桌面上看到 Windows 窗口，这看起来好像是两个操作系统完美混合在一起了。开启无缝模式的方式只需要在虚拟机上按下 `右侧Ctrl+L` 快捷键即可。 

**百度网盘**

现在很有资料都是通过百度网盘进行共享的，Linux 下可不能没有百度网盘，同样也是在官网中下载 deb 软件包直接安装即可。

```javascript
 下载地址：https://pan.baidu.com/download
```

**办公软件（WPS）**

Linux 中无法使用微软的 Office 套装，并且 Linux 默认的 LibreOffice 着实不好用。在 Linux 下比较好的 Office 替代品就是金山公司开发的 WPS 套装了，Linux 版 WPS 包含了文字处理（Word）、数据表格（Excel）、演示幻灯片（PowerPoint）和电子书阅读器（PDF Reader）四个软件。同样需要到官网下载 deb 软件包进行安装。

```javascript
 下载地址：https://linux.wps.cn/
```

**网易云音乐**

一款知名的音乐平台软件，官方有开发 Linux 版本，直接去官网下载 deb 软件包即可。

```javascript
 下载地址：https://music.163.com/#/download
```

#### **其他开发软件**

**Quartus Prime**

一款 FPGA 集成开发工具，官方有开发 Linux 版本，直接去官网下载即可。

```javascript
 下载地址：http://fpgasoftware.intel.com/?edition=lite
```

**PhpStorm**

JetBrains 旗下一款优秀的跨平台的 PHP 集成开发工具。

```javascript
 下载地址：https://www.jetbrains.com/phpstorm/
```

**PyCharm**

JetBrains 旗下一款优秀的跨平台的 Python 集成开发工具。

```javascript
 下载地址：https://www.jetbrains.com/pycharm/
```

**WebStorm**

JetBrains 旗下一款优秀的跨平台的 Web 前端集成开发工具。

```javascript
 下载地址：https://www.jetbrains.com/webstorm/
```

**IntelliJIDEA**

JetBrains 旗下一款优秀的跨平台的 Java 集成开发工具。

```javascript
 下载地址：https://www.jetbrains.com/idea/
```

**DataGrip**

JetBrains 旗下一款优秀的跨平台的数据库管理工具，可支持 MySQL、[SQL Server](https://cloud.tencent.com/product/sqlserver?from=10680)、[MongoDB](https://cloud.tencent.com/product/mongodb?from=10680) 等多种数据库。

```javascript
 下载地址：https://www.jetbrains.com/datagrip/
```

**Sublime Merge**

一款功能强大的图形化 Git 仓库管理工具，配合 Sublime 使用效果更佳。

```javascript
 下载地址：https://www.sublimemerge.com/download
```

**SDCC**

SDCC（Small Device C Compiler） 是一个小型设备的 C 语言编译器，该编译器是标准C语言，可以编译 Intel MCS51 架构的微处理器，也可以编译 STM8 等常见 MCU。

```javascript
 下载地址：http://sdcc.sourceforge.net/snap.php
```

**Hex2Bin**

一款命令行烧写工具，可烧写单片机程序（bin 或 hex）到硬件设备上。

```javascript
 下载地址：https://github.com/laborer/stcflash
```

### 附录

#### 网络应用

- [Firefox](https://wiki.deepin.org/index.php?title=Firefox)：火狐浏览器是一个安全高效的浏览器

  <http://www.firefox.org/>

- [Chrome](https://wiki.deepin.org/index.php?title=Chrome)：谷歌浏览器是一个由Google公司开发的网页浏览器

  <https://www.google.com/chrome>

- [Chromium](https://wiki.deepin.org/index.php?title=Chromium)：Chromium是Google的Chrome浏览器驱动引擎，其目的是为了创建一个安全、稳定和快速的浏览器

  <http://www.chromium.org/>

- [傲游云浏览器](https://wiki.deepin.org/index.php?title=傲游云浏览器)：傲游云浏览器是一款基于Chromium开发的云浏览器

  <http://www.maxthon.cn/>

- [Opera](https://wiki.deepin.org/index.php?title=Opera)：Opera是一款网络浏览器

  <http://www.opera.com/>

- [Midori](https://wiki.deepin.org/index.php?title=Midori)：Midori是一个轻量级的网页浏览器

  <http://www.midori-browser.org/>

- [Vivaldi](https://wiki.deepin.org/index.php?title=Vivaldi)：Vivaldi是一款极速浏览器

  <https://vivaldi.com/>

- [Yandex](https://wiki.deepin.org/index.php?title=Yandex)：Yandex是一款免费的浏览器

  <https://www.yandex.com/>

- [TeamViewer](https://wiki.deepin.org/index.php?title=TeamViewer)：TeamViewer是一个用于远程控制、桌面共享和文件传输的简单且快速的解决方案。

  [https://www.teamviewer.com](https://www.teamviewer.com/)

- [Remmina](https://wiki.deepin.org/index.php?title=Remmina)：Remmina是一个远程桌面客户端

  <http://www.remmina.org/>

- [UGet](https://wiki.deepin.org/index.php?title=UGet)：Uget是一个下载管理器

  <http://ugetdm.com/>

- [FileZilla](https://wiki.deepin.org/index.php?title=FileZilla)：FileZilla是一个快速可靠的、跨平台的FTP、FTPS和SFTP客户端

  <http://filezilla-project.org/>

- [Transmission](https://wiki.deepin.org/index.php?title=Transmission)：Transmission是一个BitTorrent客户端软件

  <https://transmissionbt.com/>

- [qBittorrent](https://wiki.deepin.org/index.php?title=QBittorrent)：qBittorrent是一个轻量级BitTorrent客户端

  <https://www.qbittorrent.org/>

- [FlareGet](https://wiki.deepin.org/index.php?title=FlareGet)：FlareGet是一个跨平台的下载管理器和加速器

  <http://flareget.com/>

- [gFTP](https://wiki.deepin.org/index.php?title=GFTP)：gFTP是一个FTP客户端工具

  <https://www.gftp.org/>

- [CrossFTP](https://wiki.deepin.org/index.php?title=CrossFTP)：CrossFTP是一款FTP客戶端工具

  <http://www.crossftp.com/>

- [Xtreme Download Manager](Xtreme Download Manager)：Xtreme Download Manager是一个P2P文件下载软件

  <https://sourceforge.net/projects/xdman/>

- [AnyDesk](https://wiki.deepin.org/index.php?title=AnyDesk)：AnyDesk是一款远程桌面控制应用

  <https://anydesk.com/>

- [Evolution](https://wiki.deepin.org/index.php?title=Evolution)：Evolution是一款电子邮件和日程安排工具

  <http://wiki.gnome.org/Apps/Evolution>

- [雷鸟邮件](https://wiki.deepin.org/index.php?title=雷鸟邮件&action=edit&redlink=1)：雷鸟邮件是一个邮件客户端

  <http://www.mozilla.org/zh-CN/thunderbird/>

- [Nylas N1](Nylas N1)：Nylas N1是一个开源的邮件客户端

  <https://www.nylas.com/>

- [ownCloud](https://wiki.deepin.org/index.php?title=OwnCloud)：ownCloud是一款用来创建私有云服务的工具

  <https://owncloud.org/>

- [Geary](https://wiki.deepin.org/index.php?title=Geary)：Geary是一款桌面电子邮件客户端程序

  <https://wiki.gnome.org/Apps/Geary>
  
- motrix:一款全能的下载工具，支持下载 HTTP、FTP、BT、磁力链、百度网盘等资源

  <https://motrix.app/zh-CN/>



#### 社交沟通

- [BearyChat](https://wiki.deepin.org/index.php?title=BearyChat)：BearyChat是一款为工作场景设计的团队沟通工具

  <https://bearychat.com/>

- [安司密信](https://wiki.deepin.org/index.php?title=安司密信):安司密信是一款社交应用

  <http://www.akey.me/>

- [Telegram](https://wiki.deepin.org/index.php?title=Telegram)：Telegram是一个聊天应用软件

  <https://telegram.org/>

- [Skype](https://wiki.deepin.org/index.php?title=Skype)：Skype是一款即时通讯软件

  <http://skype.gmw.cn/>

- [腾讯通](https://wiki.deepin.org/index.php?title=腾讯通)：腾讯通是一个企业级的实时通信平台

  <http://rtx.tencent.com/>

- [阿里旺旺](https://wiki.deepin.org/index.php?title=阿里旺旺)：阿里旺旺是一款专为淘宝会员量身定做的个人交易沟通软件

  <http://wangwang.taobao.com/>

- [千牛工作台](https://wiki.deepin.org/index.php?title=千牛工作台)：千牛工作台是阿里巴巴官方出品的卖家一站式店铺管理工具

  <https://alimarket.taobao.com/markets/qnww/pc>

- [Pidgin](https://wiki.deepin.org/index.php?title=Pidgin)：Pidgin是一款即时通讯软件

  <http://www.pidgin.im/>

- [Xchat](https://wiki.deepin.org/index.php?title=Xchat)：Xchat是一款跨平台的IRC通讯协议软件

  <http://xchat.org/>

- [HexChat](https://wiki.deepin.org/index.php?title=HexChat)：HexChat是基于XChat的一款聊天工具

  <https://hexchat.github.io/>

- [HipChat](https://wiki.deepin.org/index.php?title=HipChat)：HipChat是一款专为团队内部群聊设计的聊天工具

  <http://www.hipchat.com/>



#### 音乐欣赏

- [深度音乐](https://wiki.deepin.org/index.php?title=深度音乐)：深度音乐是深度科技重新打造的一款专注于本地音乐播放的应用程序

  <https://www.deepin.org/original/deepin-music/>

- [网易云音乐](https://wiki.deepin.org/index.php?title=网易云音乐)：网易云音乐是一款专注于发现与分享的音乐产品

  <http://music.163.com/>

- [Soundnode App](https://wiki.deepin.org/index.php?title=Soundnode_App)：Soundnode App是一款音乐播放器应用

  <http://www.soundnodeapp.com/>

- [Kreogist Mu](https://wiki.deepin.org/index.php?title=Kreogist_Mu)：Kreogist Mu是一个音乐管理中心

  <http://kreogist.github.io/>

- [Clementine](https://wiki.deepin.org/index.php?title=Clementine)：Clementine是一个音乐播放器和媒体库管理器

  <https://www.clementine-player.org/>

- [Spotify](https://wiki.deepin.org/index.php?title=Spotify)：Spotify是一种专有的P2P音乐流媒体服务

  <https://www.spotify.com/>

- [Audacious](https://wiki.deepin.org/index.php?title=Audacious)：Audacious是一款音乐播放器

  <http://audacious-media-player.org/>

- [Rhythmbox](https://wiki.deepin.org/index.php?title=Rhythmbox)：Rhythmbox是一个音乐播放和管理应用

  <https://wiki.gnome.org/Apps/Rhythmbox>

- [Amarok](https://wiki.deepin.org/index.php?title=Amarok)：Amarok是一款音乐播放器

  <https://amarok.kde.org/>

- [Tomahawk](https://wiki.deepin.org/index.php?title=Tomahawk)：Tomahawk是一款网络音乐播放器

  <https://www.tomahawk-player.org/>

- [Musescore](https://wiki.deepin.org/index.php?title=Musescore)：MuseScore是一套作曲写乐谱工具

  <https://musescore.org/>

- [MusE](https://wiki.deepin.org/index.php?title=MusE)：MusE是一个MIDI/音频的音序器

  <http://muse-sequencer.org/>
  
- Mob:喜马拉雅FM客户端：<https://github.com/zenghongtu/Mob>

- ieaseMusic – 可能是目前最好的网易云音乐播放器

  <https://github.com/trazyn/ieaseMusic>

#### 视频播放

- [深度影院](https://wiki.deepin.org/index.php?title=深度影院)：深度影院是深度科技打造的一款专注于本地视频播放的应用程序

  <https://www.deepin.org/original/deepin-movie/>

- [SMplayer](https://wiki.deepin.org/index.php?title=SMplayer)：SMplayer是一款跨平台的视频播放工具

  <http://smplayer.org/>

- [VLC](https://wiki.deepin.org/index.php?title=VLC)：VLC是一款自由、开源的跨平台多媒体播放器及框架

  <http://www.videolan.org/>

- [OpenShot](https://wiki.deepin.org/index.php?title=OpenShot)：OpenShot是一个视频编辑软件

  <http://www.openshot.org/>

- [Baka MPlayer](https://wiki.deepin.org/index.php?title=Baka_MPlayer)：Baka MPlayer是一个多媒体播放器

  <http://bakamplayer.u8sand.net/>

- [Vokoscreen](https://wiki.deepin.org/index.php?title=Vokoscreen)：Vokoscreen是一款有效的强大屏幕录制工具

  <http://www.kohaupt-online.de/hp/>

- [SimpleScreenRecorder](https://wiki.deepin.org/index.php?title=SimpleScreenRecorder)：SimpleScreenRecorder是一款屏幕录制软件

  <http://www.maartenbaert.be/simplescreenrecorder/>

- [Kazam](https://wiki.deepin.org/index.php?title=Kazam)：Kazam是一款简易的桌面屏幕录制工具

  <https://launchpad.net/kazam/>

- [Shotcut](https://wiki.deepin.org/index.php?title=Shotcut)：Shotcut是一款跨平台的视频编辑器软件

  <https://www.shotcut.org/>

- [Open Broadcaster Software](https://wiki.deepin.org/index.php?title=Open_Broadcaster_Software)：Open Broadcaster Software是一款视频录制和直播的应用

  <https://obsproject.com/>

- [Lightworks](https://wiki.deepin.org/index.php?title=Lightworks)：Lightworks是一款非线性视频编辑器

  <https://www.lwks.com/>

- [HandBrake](https://wiki.deepin.org/index.php?title=HandBrake)：HandBrake是一款视频转码工具

  <https://handbrake.fr/>

- [Gaupol](https://wiki.deepin.org/index.php?title=Gaupol)：Gaupol是一个字幕编辑软件

  <http://otsaloma.io/gaupol/>

- [recordMyDesktop](https://wiki.deepin.org/index.php?title=RecordMyDesktop)：recordMyDesktop是一个桌面视频录制软件

  <http://recordmydesktop.sourceforge.net/>

- [ArcTime](https://wiki.deepin.org/index.php?title=ArcTime)：ArcTime是一款可视化字幕编辑器

  <http://www.arctime.org/>

- [Natron](https://wiki.deepin.org/index.php?title=Natron)：Natron是一个跨平台的视频合成软件

  <https://natron.inria.fr/>
  
- LosslessCut – 无损视频剪切工具，适合无人机等大视频文件的初剪

  <https://github.com/mifi/lossless-cut/>



#### 图形图像

- [深度看图](https://wiki.deepin.org/index.php?title=深度看图)：深度看图是深度科技精心打造的一款图片查看和管理应用

  <https://www.deepin.org/original/deepin-image-viewer/>

- [深度截图](https://wiki.deepin.org/index.php?title=深度截图)：深度截图是深度科技开发的深度操作系统下自带的截图工具

  <https://www.deepin.org/original/deepin-screenshot/>

- [亿图图示](https://wiki.deepin.org/index.php?title=亿图图示)：亿图图示是一款综合图形图表制作应用

  <http://www.edrawsoft.cn/>

- [GIMP](https://wiki.deepin.org/index.php?title=GIMP)：GIMP是一款跨平台的图像处理软件

  <https://www.gimp.org/>

- [Krita](https://wiki.deepin.org/index.php?title=Krita)：Krita是一个位图形编辑软件

  <https://krita.org/>

  ```
  sudo add-apt-repository ppa:kritalime/ppa
  sudo apt-get update
  sudo apt-get install krita krita-l10n
  ```

- [Inkscape](https://wiki.deepin.org/index.php?title=Inkscape)：Inkscape是一款矢量绘图软件

  <https://inkscape.org/>

- [LightZone](https://wiki.deepin.org/index.php?title=LightZone)：LightZone是一款数码图象编辑工具

  <http://lightzoneproject.org/>

- [Blender](https://wiki.deepin.org/index.php?title=Blender)：Blender是一款三维动画制作软件

  <https://www.blender.org/>

- [BricsCAD](https://wiki.deepin.org/index.php?title=BricsCAD)：BricsCAD是一个功能强大的CAD平台

  [https://www.bricsys.com](https://www.bricsys.com/)

- [XnConvert](https://wiki.deepin.org/index.php?title=XnConvert)：XnConvert是一款多功能图像批处理工具

  <http://www.xnview.com/en/xnconvert/>

- [XnView MP](https://wiki.deepin.org/index.php?title=XnView_MP)：XnView MP是一款非常棒的图像查看工具

  <http://www.xnview.com/en/xnviewmp/>

- [XnSketch](https://wiki.deepin.org/index.php?title=XnSketch)：XnSketch是一款图片处理应用

  <http://www.xnview.com/en/xnsketch/>

- [Shutter](https://wiki.deepin.org/index.php?title=Shutter)：Shutter是一款广受欢迎的截屏软件

  <http://shutter-project.org/>

- [MyPaint](https://wiki.deepin.org/index.php?title=MyPaint)：MyPaint是一个图像绘画工具

  <http://mypaint.org/>

- [泼辣修图](https://wiki.deepin.org/index.php?title=泼辣修图&action=edit&redlink=1)：泼辣修图（Polarr）是一个智能图片处理工具

  <https://www.polarr.co/>

- [Peek](https://wiki.deepin.org/index.php?title=Peek)：Peek是一款屏幕录制工具

  <https://github.com/phw/peek>

- [Synfig studio](https://wiki.deepin.org/index.php?title=Synfig_studio)：Synfig Studio是一套功能强大的2D矢量动画制作软件

  <http://www.synfig.org/>

- [RawTherapee](https://wiki.deepin.org/index.php?title=RawTherapee)：RawTherapee是一款RAW格式转换软件

  <http://www.rawtherapee.com/>

- [图像查看器](https://wiki.deepin.org/index.php?title=图像查看器&action=edit&redlink=1)：图像查看器（EyeOfGnome）是一款图像查看器软件

  <https://wiki.gnome.org/Apps/EyeOfGnome>

- [HotShots](https://wiki.deepin.org/index.php?title=HotShots)：HotShots是一款易于使用的屏幕捕获工具

  <https://sourceforge.net/projects/hotshots/>

- [Gwenview](https://wiki.deepin.org/index.php?title=Gwenview)：Gwenview是一个图片浏览和管理软件

  <https://www.kde.org/applications/graphics/gwenview/>

- [digiKam](https://wiki.deepin.org/index.php?title=DigiKam)：digiKam是一款跨平台的数字照片管理软件

  <https://www.digikam.org/>

- [Shotwell](https://wiki.deepin.org/index.php?title=Shotwell)：Shotwell是一款轻量级的图片管理软件

  <https://wiki.gnome.org/Apps/Shotwell>

- [yEd Graph Editor](https://wiki.deepin.org/index.php?title=YEd_Graph_Editor)：yEd Graph Editor是一款流程图绘制工具

  <http://www.yworks.com/>



#### 娱乐游戏

- [Steam](https://wiki.deepin.org/index.php?title=Steam)：Steam是一款方便迅速的综合性游戏平台

  <http://store.steampowered.com/>

- [Warsow](https://wiki.deepin.org/index.php?title=Warsow)：Warsow是一款第一人称射击游戏

  <https://www.warsow.net/>

- [Desura](https://wiki.deepin.org/index.php?title=Desura)：Desura是一个正版游戏网站

  <http://www.desura.com/>

- [Snes9x](https://wiki.deepin.org/index.php?title=Snes9x)：Snes9x是一款超级任天堂SFC模拟器

  <http://www.snes9x.com/>

- [Flail Rider](https://wiki.deepin.org/index.php?title=Flail_Rider)：Flail Rider是一款快节奏的桌面游戏

  <http://jushiip.itch.io/flail-rider>

- [MAME](https://wiki.deepin.org/index.php?title=MAME)：MAME是一个模拟器，也是国内玩家最熟悉和最常使用的街机模拟器之一

  <http://mamedev.org/>

- [SuperTux](https://wiki.deepin.org/index.php?title=SuperTux)：SuperTux是一款类似超级马里奥兄弟的游戏

  <http://supertuxproject.org/>

- [AssaultCube](https://wiki.deepin.org/index.php?title=AssaultCube)：AssaultCube是一款第一视角射击游戏

  <http://assault.cubers.net/>

- [PPSSPP](https://wiki.deepin.org/index.php?title=PPSSPP)：PPSSPP是一款非常出色的PSP模拟器

  <http://www.ppsspp.org/>

- [Stunt Rally](https://wiki.deepin.org/index.php?title=Stunt_Rally)：Stunt Rally是一款赛车游戏

  <http://stuntrally.tuxfamily.org/>



#### 办公学习

- [WPS Office](https://wiki.deepin.org/index.php?title=WPS_Office)：WPS Office是由金山软件股份有限公司自主研发的一款办公软件套件

  <http://linux.wps.cn/>

- [LibreOffice](https://wiki.deepin.org/index.php?title=LibreOffice)：LibreOffice是一款功能强大的办公套件

  <https://www.libreoffice.org/>

- [永中Office](https://wiki.deepin.org/index.php?title=永中Office)：永中Office是一款功能强大的办公软件

  <http://www.yozosoft.com/>

- [深度云打印](https://wiki.deepin.org/index.php?title=深度云打印)：深度云打印是由深度科技开发的一种新型打印解决方案

  <https://www.deepin.org/original/deepin-cloud-print/>

- [深度云扫描](https://wiki.deepin.org/index.php?title=深度云扫描)：深度云扫描是由深度科技开发的一种新型扫描技术

  <https://www.deepin.org/original/deepin-cloud-scan/>

- [XMind](https://wiki.deepin.org/index.php?title=XMind)：XMind是一款全球领先的思维导图软件

  <http://www.xmind.net/>

  - 替换脚本: [XMind_amd64.tar.gz](https://pan.baidu.com/s/1pijGyVHFfO1BlDx-ZhtI5g)(提取码: tnkb) -- 包含`XMindCrack.jar`, `XMind.ini`

  - 下载好最新XMind压缩包之后解压, 再解压`XMind_amd64.tar.gz`到`XMind_amd64`目录下.(我的是64bit的系统, 如果是32bit的系统解压到`XMind_i386`目录即可),

  - 在`XMind_amd64`目录下创建run.sh文件,/opt/xmind-8/是安装路径

    ```
    cd /opt/xmind-8/XMind_amd64/
    /opt/xmind-8/XMind_amd64/XMind
    ```

    然后赋予执行权限

    ```
    sudo chmod a+x /opt/xmind-8/XMind_amd64/run.s
    ```

  - 修改host:

    shell下, 输入以修改host:

    ```
    $ sudo vim /etc/hosts
    #在文本末尾添加:
    127.0.0.1 www.xmind.net
    ```

  - 回到解压后的文件夹的根目录, 运行`./setup.sh`. 安装好依赖库之后, 再次进入`XMind_amd64`, 运行`XMind`.

    执行下面，进行验证

    ```
    ./run.sh
    ```

    如果出现问题，可能是权限问题，

    ```
    sudo chown -R $USER /opt/xmind-8/
    ```

    

  - 在XMind主界面依次: `Help -> License`, 复制以下license key即可, 邮箱随便填:

    ```
    XAka34A2rVRYJ4XBIU35UZMUEEF64CMMIYZCK2FZZUQNODEKUHGJLFMSLIQMQUCUBXRENLK6NZL37JXP4PZXQFILMQ2RG5R7G4QNDO3PSOEUBOCDRYSSXZGRARV6MGA33TN2AMUBHEL4FXMWYTTJDEINJXUAV4BAYKBDCZQWVF3LWYXSDCXY546U3NBGOI3ZPAP2SO3CSQFNB7VVIY123456789012345
    ```

    PS: 以上方法是通用的, 即: WIn Mac Linux都可以用这种方式. 替换的`XMind.ini`文件也只是在原始文件上添加了一行`-javaagent:./XMindCrack.jar`以执行Crack文件.
    
  - 桌面快捷方式

    ```
    [Desktop Entry]
    Type=Application
  Name[zh_CN]=XMind
    Exec=/opt/xmind-8/XMind_amd64/run.sh
    X-GNOME-FullName[zh_CN]=XMind
    Comment[zh_CN]=思维导图
    Icon=XMind
    NoDisplay=false
    Path=
    Terminal=false
    X-GNOME-UsesNotifications=false
    Categories=Office;
    ```
    
    

- [为知笔记](https://wiki.deepin.org/index.php?title=为知笔记)：为知笔记是一款云服务笔记软件

  <http://www.wiz.cn/>

- [Notepadqq](https://wiki.deepin.org/index.php?title=Notepadqq)：Notepadqq是一套纯文字编辑器，与Notepad++非常相似。

  <http://notepadqq.altervista.org/>

- [GeoGebra](https://wiki.deepin.org/index.php?title=GeoGebra)：GeoGebra是一个动态数学应用

  <https://www.geogebra.org/>

- [Leanote](https://wiki.deepin.org/index.php?title=Leanote)：Leanote是一款在线的云笔记应用

  <https://leanote.com/>

- [Remarkable](https://wiki.deepin.org/index.php?title=Remarkable)：Remarkable是一款Markdown编辑器

  <https://remarkableapp.github.io/>

- [LyX](https://wiki.deepin.org/index.php?title=LyX)：LyX是一款利用LaTeX来排版的文件编辑软件

  <http://www.lyx.org/>

- [VYM](https://wiki.deepin.org/index.php?title=VYM)：VYM是一款思维导图软件

  <http://www.insilmaril.de/vym/>

- [FocusWriter](https://wiki.deepin.org/index.php?title=FocusWriter)：FocusWriter是一款写作软件

  <http://gottcode.org/focuswriter/>

- [打印机](https://wiki.deepin.org/index.php?title=打印机)：Printers（打印机）是一款用于配置系统打印机服务器的工具

  <http://cyberelk.net/tim/software/system-config-printer/>

- [ChmSee](https://wiki.deepin.org/index.php?title=ChmSee)：ChmSee是一个在Linux下阅读CHM格式帮助文件的软件

  <http://code.google.com/p/chmsee>

- [Gwyddion](https://wiki.deepin.org/index.php?title=Gwyddion)：Gwyddion是一款模块化扫描探针显微镜数据可视化和分析工具

  <http://gwyddion.net/>

- [pyRenamer](https://wiki.deepin.org/index.php?title=PyRenamer)：pyRenamer是一款文件批量改名工具

  <https://launchpad.net/pyrenamer>

- [Scilab](https://wiki.deepin.org/index.php?title=Scilab)：Scilab是一个为工程和科学应用量身定做的数值计算软件

  <http://www.scilab.org/>

- [TeXstudio](https://wiki.deepin.org/index.php?title=TeXstudio)：TeXstudio是一个编写LaTeX文档的集成开发环境

  <http://www.texstudio.org/>

- [Texmaker](https://wiki.deepin.org/index.php?title=Texmaker)：Texmaker是一个LaTeX的编辑环境

  <http://www.xm1math.net/texmaker/>

- [GNU TeXmacs](https://wiki.deepin.org/index.php?title=GNU_TeXmacs)：GNU TeXmacs是一个优秀的科学文档排版软件

  <http://www.texmacs.org/>

- [KWrite](https://wiki.deepin.org/index.php?title=KWrite)：KWrite是一款程序员非常喜欢的文字编辑器

  <https://www.kde.org/applications/utilities/kwrite/>

- [Smallpdf](https://wiki.deepin.org/index.php?title=Smallpdf)：Smallpdf是一款在线PDF处理工具

  <https://smallpdf.com/>

- [Open365](https://wiki.deepin.org/index.php?title=Open365)：Open365是一款Office文档处理软件

  <https://open365.io/>

- [AbiWord](https://wiki.deepin.org/index.php?title=AbiWord)：AbiWord是一款功能完备的高效文字处理软件

  <https://www.abisource.com/>

- [GanttProject](https://wiki.deepin.org/index.php?title=GanttProject)：GanttProject是一款项目计划和管理图绘制应用

  <http://www.ganttproject.biz/>

- [Scan Tailor](https://wiki.deepin.org/index.php?title=Scan_Tailor)：Scan Tailor是一个用于扫描件的后期处理软件

  <http://scantailor.org/>

- [Gnumeric](https://wiki.deepin.org/index.php?title=Gnumeric)：Gnumeric是一款电子表格处理软件

  <http://www.gnumeric.org/>

- [TagSpaces](https://wiki.deepin.org/index.php?title=TagSpaces)：TagSpaces是一款文档管理工具

  <http://tagspaces.org/>

- [QOwnNotes](https://wiki.deepin.org/index.php?title=QOwnNotes)：QOwnNotes是一款支持Markdown和ownCloud同步的文本编辑器

  <http://www.qownnotes.org/>

- [CuteMarkEd](https://wiki.deepin.org/index.php?title=CuteMarkEd)：CuteMarkEd是一款Markdown编辑器

  <http://cloose.github.io/CuteMarkEd/>

- [Moeditor](https://wiki.deepin.org/index.php?title=Moeditor)：Moeditor是一款markdown编辑器

  <https://moeditor.org/>

- [WordMark](https://wiki.deepin.org/index.php?title=WordMark)：WordMark是一款MarkDown编辑器

  <http://wordmarkapp.com/>

- [Haroopad](https://wiki.deepin.org/index.php?title=Haroopad)：Haroopad是一款Linux平台下的markdown编辑器

  [https://pad.haroopress.com](https://pad.haroopress.com/)

- [Cmd Markdown](https://wiki.deepin.org/index.php?title=Cmd_Markdown)：Cmd Markdown是一款MarkDown编辑器

  <https://www.zybuluo.com/cmd/>

- [MarkMyWords](https://wiki.deepin.org/index.php?title=MarkMyWords)：MarkMyWords是一款markdown编辑器

  <https://github.com/voldyman/MarkMyWords>

- [Marp](https://wiki.deepin.org/index.php?title=Marp)：MarkMyWords是一款markdown编辑器

  <https://github.com/yhatt/marp>
  
- stretchly:桌面提醒休息工具<https://github.com/hovancik/stretchly>

- Zettlr – 撰写专业论文的 MarkDown 编辑器<https://www.zettlr.com>

- simplenote-多平台同步笔记:<https://simplenote.com>

- Ao – 第三方开源微软 To-Do 桌面客户端

  <https://github.com/klaussinani/ao>

- GitNote – 基于 Git 的跨平台云笔记工具

  <https://gitnoteapp.com>

#### 翻译阅读

- [福昕阅读器](https://wiki.deepin.org/index.php?title=福昕阅读器)：福昕阅读器（Foxit Reader）是一款PDF文档阅读器

  <https://www.foxitsoftware.cn/>

- [有道词典](https://wiki.deepin.org/index.php?title=有道词典)：有道词典是中国第一个基于Linux下的互联网商业翻译软件

  <http://cidian.youdao.com/>

- [文档查看器](https://wiki.deepin.org/index.php?title=文档查看器)：Evince（文档查看器）是一个支持多种格式的文件浏览器

  <https://wiki.gnome.org/Apps/Evince>

- [Master PDF Editor](https://wiki.deepin.org/index.php?title=Master_PDF_Editor)：Master PDF Editor是一款功能强大的PDF和XPS文档编辑工具

  <http://code-industry.net/masterpdfeditor/>

- [Calibre](https://wiki.deepin.org/index.php?title=Calibre)：Calibre是一个“一站式”的电子书管理软件

  <http://calibre-ebook.com/>

- [GoldenDict](https://wiki.deepin.org/index.php?title=GoldenDict)：GoldenDict是一款词典软件，支持多种词典文件格式

  <http://goldendict.org/>

- [qpdfview](https://wiki.deepin.org/index.php?title=Qpdfview)：qpdfview是一个小巧的PDF阅读器

  <https://launchpad.net/qpdfview>

- [Okular](https://wiki.deepin.org/index.php?title=Okular)：Okular是一个PDF文档阅读软件

  <http://okular.kde.org/>

- [Comix](https://wiki.deepin.org/index.php?title=Comix)：Comix是一款优秀的连环画电子阅读器

  <http://comix.sourceforge.net/>

- [Mendeley](https://wiki.deepin.org/index.php?title=Mendeley)：Mendeley是一款文献管理软件

  <https://www.mendeley.com/>

- [PdfMod](https://wiki.deepin.org/index.php?title=PdfMod)：PdfMod是一个PDF文档编辑应用

  <http://wiki.gnome.org/Apps/PdfMod>

- [PDF-Shuffler](https://wiki.deepin.org/index.php?title=PDF-Shuffler)：PDF-Shuffler是一个PDF合并及分割工具

  <http://pdfshuffler.sourceforge.net/>



#### 编程开发

- [Android Studio](https://wiki.deepin.org/index.php?title=Android_Studio)：Android Studio是一个基于IntelliJ IDEA的Android集成开发环境

  <https://developer.android.com/>

- [Visual Studio Code](https://wiki.deepin.org/index.php?title=Visual_Studio_Code)：Visual Studio Code是一款轻量级代码编辑器

  <https://code.visualstudio.com/>

- [Sublime](https://wiki.deepin.org/index.php?title=Sublime)：Sublime Text是一个代码编辑器

  <http://www.sublimetext.com/>

- [Atom](https://wiki.deepin.org/index.php?title=Atom)：Atom是一个在线文本编辑器

  <http://www.atom.io/>

- [Genymotion](https://wiki.deepin.org/index.php?title=Genymotion)：Genymotion是一款专业的虚拟模拟器

  <https://www.genymotion.com/>

- [GVim](https://wiki.deepin.org/index.php?title=GVim)：Vim是从vi发展而来的一个文本编辑器

  <http://www.vim.org/>

- [UltraEdit](https://wiki.deepin.org/index.php?title=UltraEdit)：UltraEdit是一个文本编辑器

  <http://www.ultraedit.com/>

- [Eclipse IDE](https://wiki.deepin.org/index.php?title=Eclipse_IDE)：Eclipse 是一个开放源代码的、基于Java的可扩展开发平台

  [http://www.eclipse.org](http://www.eclipse.org/)

- [JetBrains IDE](https://wiki.deepin.org/index.php?title=JetBrains_IDE)：JetBrains公司是最好的Java IDE的创造者

  <https://www.jetbrains.com/>

- [Komodo IDE](https://wiki.deepin.org/index.php?title=Komodo_IDE)：Komodo IDE是一款支持多种动态编程语言的集成开发环境

  <http://www.activestate.com/komodo-ide>

- [Navicat](https://wiki.deepin.org/index.php?title=Navicat)：Navicat是一个的数据库管理工具

  <https://www.navicat.com/products/navicat-for-mysql>

- [Qt Creator](https://wiki.deepin.org/index.php?title=Qt_Creator)：Qt Creator是跨平台的轻量级集成开发环境

  [https://www.qt.io](https://www.qt.io/)

- [MySQL Workbench](https://wiki.deepin.org/index.php?title=MySQL_Workbench)：MySQL Workbench是一款专为MySQL设计的ER/数据库建模工具

  <https://www.mysql.com/products/workbench/>

- [SQLiteStudio](https://wiki.deepin.org/index.php?title=SQLiteStudio)：SQLiteStudio是一款可以帮助用户管理sqlite数据库的工具

  <https://sqlitestudio.pl/>

- [DBeaver](https://wiki.deepin.org/index.php?title=DBeaver)：DBeaver是一个通用的数据库管理工具和SQL客户端

  <http://dbeaver.jkiss.org/>

- [Wireshark](https://wiki.deepin.org/index.php?title=Wireshark)：Wireshark是一个网络封包分析软件

  <https://www.wireshark.org/>

- [Code::Blocks](https://wiki.deepin.org/index.php?title=Code::Blocks)：Code::Blocks是一个开源、免费、跨平台的C++ IDE

  <http://www.codeblocks.org/>

- [MyEclipse](https://wiki.deepin.org/index.php?title=MyEclipse)：MyEclipse是一款软件开发工具，是对EclipseIDE的扩展

  <https://www.genuitec.com/products/myeclipse/>

- [SmartGit](https://wiki.deepin.org/index.php?title=SmartGit)：SmartGit是一个Git版本控制系统的图形化客户端程序

  <http://www.syntevo.com/smartgit/>

- [SmartSVN](https://wiki.deepin.org/index.php?title=SmartSVN)：SmartSVN是一个图形化的SVN客户端

  <http://www.smartsvn.com/>

- [SmartCVS](https://wiki.deepin.org/index.php?title=SmartCVS)：SmartCVS是一款用于CVS的版本控制应用

  <http://www.syntevo.com/smartcvs/>

- [SmartSynchronize](https://wiki.deepin.org/index.php?title=SmartSynchronize)：SmartSynchronize是一款检查文件与目录比较的工具

  <http://www.syntevo.com/smartsynchronize/>

- [GNU Emacs](https://wiki.deepin.org/index.php?title=GNU_Emacs)：GNU Emacs是一个可扩展、定制的文本编辑器

  <http://gnuemacs.org/>

- [Deepin Emacs](https://wiki.deepin.org/index.php?title=Deepin_Emacs)：Emacs是一个可自编程和扩展的文本编辑器

  <https://www.deepin.org/original/deepin-emacs/>

- [Spyder](https://wiki.deepin.org/index.php?title=Spyder)：Spyder是一个强大的交互式Python集成开发环境

  <http://pythonhosted.org/spyder/>

- [Brackets](https://wiki.deepin.org/index.php?title=Brackets)：Brackets是一个HTML/CSS/JavaScrip前端WEB集成开发环境

  <http://brackets.io/>

- [CodeLite](https://wiki.deepin.org/index.php?title=CodeLite)：CodeLite是一个C/C++编程语言的跨平台IDE

  <http://codelite.org/>

- [Bluefish](https://wiki.deepin.org/index.php?title=Bluefish)：Bluefish是一款为熟练的Web设计员和程序员而设的编辑器

  <http://bluefish.openoffice.nl/>

- [Eric](https://wiki.deepin.org/index.php?title=Eric)：Eric是一个全功能Python和Ruby的编辑器和IDE

  <http://eric-ide.python-projects.org/>

- [LiteIDE](https://wiki.deepin.org/index.php?title=LiteIDE)：LiteIDE是一款开源、跨平台的轻量级Go语言集成开发环境

  <https://sourceforge.net/projects/liteide/>

- [MonoDevelop](https://wiki.deepin.org/index.php?title=MonoDevelop)：MonoDevelop是个跨平台的集成开发环境

  <http://www.monodevelop.com/>

- [Geany](https://wiki.deepin.org/index.php?title=Geany)：Geany是一个跨平台的开源集成开发环境

  <http://www.geany.org/>

- [NetBeans](https://wiki.deepin.org/index.php?title=NetBeans)：NetBeans是一个集成开发环境

  <https://netbeans.org/>

- [Arduino](https://wiki.deepin.org/index.php?title=Arduino)：Arduino是一个软硬件平台

  <https://www.arduino.cc/>

- [Glade](https://wiki.deepin.org/index.php?title=Glade)：Glade是一个相当不错的图形界面设计工具

  <https://glade.gnome.org/>

- [Gambas](https://wiki.deepin.org/index.php?title=Gambas)：Gambas是一个开发Visual Basic的集成开发环境

  <http://gambas.sourceforge.net/>

- [GitKraken](https://wiki.deepin.org/index.php?title=GitKraken)：GitKraken是一款Git客户端

  <https://www.gitkraken.com/>

- [Anjuta](https://wiki.deepin.org/index.php?title=Anjuta)：Anjuta是一个C、C++的集成开发环境

  <http://anjuta.org/>

- [SciTE](https://wiki.deepin.org/index.php?title=SciTE)：SciTE是一款免费开源的编辑器

  <http://www.scintilla.org/>

- [Yakuake](https://wiki.deepin.org/index.php?title=Yakuake)：Yakuake是一款下拉式终端仿真器

  <https://www.kde.org/applications/system/yakuake/>

- [Kate](https://wiki.deepin.org/index.php?title=Kate)：Kate是一个开源的、先进的文本编辑器

  <http://kate-editor.org/>

- [Lazarus](https://wiki.deepin.org/index.php?title=Lazarus)：Lazarus是一个Pascal集成开发环境

  <http://www.lazarus-ide.org/>

- [Intel XDK](https://wiki.deepin.org/index.php?title=Intel_XDK)：Intel XDK是一款HTML5跨平台集成开发工具

  <https://software.intel.com/en-us/intel-xdk>

- [GkDebconf](https://wiki.deepin.org/index.php?title=GkDebconf)：GkDebconf基本上是一个图形化的前端

  <https://sourceforge.net/projects/gkdebconf/>

- [Okteta](https://wiki.deepin.org/index.php?title=Okteta)：Okteta是一款十六进制编辑器

  <https://www.kde.org/applications/utilities/okteta/>

- [wxMEdit](https://wiki.deepin.org/index.php?title=WxMEdit)：wxMEdit是一个跨平台的文本/十六进制编辑器

  <http://wxmedit.github.io/>

- [Poedit](https://wiki.deepin.org/index.php?title=Poedit)：Poedit是一款.Po文件编辑器

  <http://poedit.net/>

- [D-Feet](https://wiki.deepin.org/index.php?title=D-Feet)：D-Feet是一个易于使用D-bus调试器

  <http://wiki.gnome.org/action/show/Apps/DFeet>

- [PuTTY](https://wiki.deepin.org/index.php?title=PuTTY)：PuTTY是一套远程登陆工具

  [www.putty.org/](https://wiki.deepin.org/index.php?title=Www.putty.org/&action=edit&redlink=1)

- [PyRoom](https://wiki.deepin.org/index.php?title=PyRoom)：PyRoom是一款文本编辑器

  <http://pyroom.org/>

- [Racket](https://wiki.deepin.org/index.php?title=Racket)：Racket是一种计算机程序设计语言

  <http://racket-lang.org/>



#### 系统管理

- [深度文件管理器](https://wiki.deepin.org/index.php?title=深度文件管理器)：深度文件管理器是深度科技开发的一款功能强大、简单易用的文件管理工具

  <https://www.deepin.org/original/dde-file-manager/>

- [深度终端](https://wiki.deepin.org/index.php?title=深度终端)：深度终端是深度科技精心打造的一款终端模拟器

  <https://www.deepin.org/original/deepin-terminal/>

- [深度启动盘制作工具](https://wiki.deepin.org/index.php?title=深度启动盘制作工具)：深度启动盘制作工具是深度科技团队开发的一款系统启动盘制作工具

  <https://www.deepin.org/original/deepin-boot-maker/>

- [归档管理器](https://wiki.deepin.org/index.php?title=归档管理器)：Fileroller（归档管理器）是一个归档管理器

  <http://fileroller.sourceforge.net/>

- [搜狗输入法](https://wiki.deepin.org/index.php?title=搜狗输入法)：搜狗输入法是一款流行的中文输入法

  <http://pinyin.sogou.com/linux/>

- [系统监视器](https://wiki.deepin.org/index.php?title=系统监视器)：系统监视器（gnome-system-monitor）是一个开源的应用程序

  <https://github.com/GNOME/gnome-system-monitor>

- [字体查看器](https://wiki.deepin.org/index.php?title=字体查看器)：字体查看器是一个查看字体的应用程序

  <https://github.com/GNOME/gnome-font-viewer>

- [Deepin OpenSymbol](https://wiki.deepin.org/index.php?title=Deepin_OpenSymbol)：Deepin OpenSymbol是深度科技团队在Wingdings系列字体内容上，重新绘制一系列符号字体

  <https://www.deepin.org/original/deepin-opensymbol/>

- [Nautilus](https://wiki.deepin.org/index.php?title=Nautilus)：Nautilus是一款文件管理器

  <https://wiki.gnome.org/Apps/Nautilus>

- [Boxes](https://wiki.deepin.org/index.php?title=Boxes)：Boxes是一个桌面虚拟化管理工具

  <https://wiki.gnome.org/Apps/Boxes>

- [VirtualBox](https://wiki.deepin.org/index.php?title=VirtualBox)：Oracle VM VirtualBox是一款开源的虚拟机软件

  <https://www.virtualbox.org/>

- [VMware Workstation](https://wiki.deepin.org/index.php?title=VMware_Workstation)：VMware Workstation是一款桌面虚拟计算机软件

  <http://www.vmware.com/>

- [Disks](https://wiki.deepin.org/index.php?title=Disks)：Disks是一款磁盘管理工具

  <https://launchpad.net/gnome-disk-utility>

- [GParted](https://wiki.deepin.org/index.php?title=GParted)：GParted是一款Linux下的功能非常强大的分区工具

  <http://gparted.org/>

- [BleachBit](https://wiki.deepin.org/index.php?title=BleachBit)：BleachBit是一款开源的系统清理工具

  <http://bleachbit.sourceforge.net/>

- [Gnome Terminal](https://wiki.deepin.org/index.php?title=Gnome_Terminal)：Gnome Terminal是一个终端仿真器

  <https://wiki.gnome.org/Apps/Terminal>

- [WinUSB](https://wiki.deepin.org/index.php?title=WinUSB)：WinUSB是一个创建Windows可引导U盘工具

  <https://sourceforge.net/projects/winusb/>

- [UNetbootin](https://wiki.deepin.org/index.php?title=UNetbootin)：UNetbootin是一个跨平台制作启动盘的工具

  <http://unetbootin.github.io/>

- [CPU-G](https://wiki.deepin.org/index.php?title=CPU-G)：CPU-G是一款CPU检测软件

  <https://sourceforge.net/projects/cpug/>

- [Hardinfo](https://wiki.deepin.org/index.php?title=Hardinfo)：HardInfo是一个系统信息查看软件

  <https://github.com/lpereira/hardinfo>

- [Brasero](https://wiki.deepin.org/index.php?title=Brasero)：Brasero是一款CD/DVD刻录软件

  <https://wiki.gnome.org/Apps/Brasero>

- [K3b](https://wiki.deepin.org/index.php?title=K3b)：K3b是一款全功能的CD/DVD烧录软件

  <http://www.k3b.org/>

- [I-Nex](https://wiki.deepin.org/index.php?title=I-Nex)：I-Nex是一款系统信息查询工具

  <https://sourceforge.net/projects/i-nex/>

- [Psensor](https://wiki.deepin.org/index.php?title=Psensor)：PSensor是一款监控硬件温度的工具

  <http://wpitchoune.net/psensor/>

- [ISO Master](https://wiki.deepin.org/index.php?title=ISO_Master)：ISO Master是一个光盘镜像文件编辑器

  <http://littlesvr.ca/isomaster/>

- [Gufw](https://wiki.deepin.org/index.php?title=Gufw)：Gufw是一款防火墙UFW的图形化管理工具

  <http://gufw.org/>

- [Terminator](https://wiki.deepin.org/index.php?title=Terminator)：Terminator是一款跨平台的终端工具

  <https://launchpad.net/terminator/>

- [Cairo-Dock](https://wiki.deepin.org/index.php?title=Cairo-Dock)：Cairo-Dock是一个Dock类软件

  <http://www.glx-dock.org/>

- [Gnome Pie](https://wiki.deepin.org/index.php?title=Gnome_Pie)：Gnome Pie是一款炫酷的程序启动器

  <https://github.com/Simmesimme/Gnome-Pie>

- [Guake](https://wiki.deepin.org/index.php?title=Guake)：Guake是一个下拉式的终端程序

  <http://guake-project.org/>

- [VNC-Client](https://wiki.deepin.org/index.php?title=VNC-Client)：VNC-Client是一款远程连接应用

  <https://launchpad.net/evnc>

- [f.lux](https://wiki.deepin.org/index.php?title=F.lux)：f.lux是一款自动调整屏幕色温亮度的应用

  <https://justgetflux.com/>

- [Redshift](https://wiki.deepin.org/index.php?title=Redshift)：Redshift是一款显示器色温自动调整应用

  <https://github.com/jonls/redshift>

- [Conky Manager](https://wiki.deepin.org/index.php?title=Conky_Manager)：Conky Manager是一款简单的图形化工具

  <https://launchpad.net/conky-manager>

- [Gconf Editor](https://wiki.deepin.org/index.php?title=Gconf_Editor)：Gconf Editor是一个配置编辑软件

  <https://download.gnome.org/sources/gconf-editor/>

- [Tickeys](https://wiki.deepin.org/index.php?title=Tickeys)：Tickeys是一款打字机声效模拟软件

  <https://github.com/BillBillBillBill/Tickeys-linux>

- [Meld](https://wiki.deepin.org/index.php?title=Meld)：Meld是一款可视化的文件及目录对比/合并工具

  <http://meldmerge.org/>

- [Beyond Compare](https://wiki.deepin.org/index.php?title=Beyond_Compare)：Beyond Compare是一款文件/文件夹对比工具

  [http://www.scootersoftware.com](http://www.scootersoftware.com/)

- [Baobab](https://wiki.deepin.org/index.php?title=Baobab)：Baobab是一款分析磁盘使用情况的图形工具

  <http://www.marzocca.net/linux/baobab/>

- [Nero](https://wiki.deepin.org/index.php?title=Nero)：Nero是一款多媒体刻录和编辑应用

  <http://www.nero.com/>

- [Catfish](https://wiki.deepin.org/index.php?title=Catfish)：Catfish是一个文件搜索工具

  <https://launchpad.net/catfish-search>

- [Synapse](https://wiki.deepin.org/index.php?title=Synapse)：Synapse是一款使用Vala语言编写的启动器软件

  <https://launchpad.net/synapse-project>

- [Konsole](https://wiki.deepin.org/index.php?title=Konsole)：Konsole是一个终端模拟器

  <https://konsole.kde.org/>

- [PeaZip](https://wiki.deepin.org/index.php?title=PeaZip)：PeaZip是一款解压缩软件

  <http://www.peazip.org/>

- [B1 Free Archiver](https://wiki.deepin.org/index.php?title=B1_Free_Archiver)：B1 Free Archiver是一款压缩/解压缩应用

  <http://b1.org/>

- [Xarchiver](https://wiki.deepin.org/index.php?title=Xarchiver)：Xarchiver是一款解压/压缩应用

  <http://xarchiver.sourceforge.net/>

- [KShutdown](https://wiki.deepin.org/index.php?title=KShutdown)：KShutDown是一个定时关机工具

  <http://kshutdown.sourceforge.net/>

- [KeePass2](https://wiki.deepin.org/index.php?title=KeePass2)：KeePass2是一个密码管理工具

  <http://keepass.info/>

- [KeePassX](https://wiki.deepin.org/index.php?title=KeePassX)：KeePassX是一个密码管理工具

  <https://www.keepassx.org/>

- [Synergy](https://wiki.deepin.org/index.php?title=Synergy)：Synergy是一款键盘鼠标共享软件

  <http://synergy-project.org/>

- [GtkHash](https://wiki.deepin.org/index.php?title=GtkHash)：GtkHash是一个用来计算消息摘要和checksum的工具

  <http://gtkhash.sourceforge.net/>

- [dupeGuru](https://wiki.deepin.org/index.php?title=DupeGuru)：dupeGuru是一款重复文件清理应用

  <https://www.hardcoded.net/dupeguru/>

- [ANGRYsearch](https://wiki.deepin.org/index.php?title=ANGRYsearch)：ANGRYsearch是一款文件快速搜索工具

  <https://github.com/DoTheEvo/ANGRYsearch>

- [CopyQ](https://wiki.deepin.org/index.php?title=CopyQ)：CopyQ是一款剪贴板软件，适用于大量数据的复制粘贴

  <http://hluk.github.io/CopyQ/>

- [Easystroke](https://wiki.deepin.org/index.php?title=Easystroke)：Easystroke是一个手势识别应用

  <https://github.com/thjaeger/easystroke>

- [CherryTree](https://wiki.deepin.org/index.php?title=CherryTree)：CherryTree是一个支持无限层级分类的笔记软件

  <http://www.giuspen.com/cherrytree/>

- [FF Multi Converter](https://wiki.deepin.org/index.php?title=FF_Multi_Converter)：FF Multi Converter是一个多功能的格式转换工具

  <https://sites.google.com/site/ffmulticonverter/>

- [Simplenote](https://wiki.deepin.org/index.php?title=Simplenote)：Simplenote是一款纯文本笔记记录应用

  <https://github.com/Automattic/simplenote-electron>

- [LuckyBackup](https://wiki.deepin.org/index.php?title=LuckyBackup)：LuckyBackup是一个文件备份和同步工具

  <http://luckybackup.sourceforge.net/>

- [Referencer](https://wiki.deepin.org/index.php?title=Referencer)：Referencer是一款文献管理工具

  <https://launchpad.net/referencer>
  
- starUML:多平台UML工具

  <https://staruml.io>
  
- Etcher – 将「系统镜像文件」快速制作为 USB/SD 启动盘

  <https://www.appinn.com/etcher-burn-better/>
  
  安装参考：<https://github.com/balena-io/etcher>
  
  - Add Etcher debian repository:
  
  ```
  echo "deb https://deb.etcher.io stable etcher" | sudo tee /etc/apt/sources.list.d/balena-etcher.list
  ```
  
  - Trust Bintray.com's GPG key:
  
  ```
  sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 379CE192D401AB61
  ```
  
  - Update and install:
  
  ```
  sudo apt-get update
  sudo apt-get install balena-etcher-electron
  ```
  
  - Uninstall
  
  ```
  sudo apt-get remove balena-etcher-electron
  sudo rm /etc/apt/sources.list.d/balena-etcher.list
  sudo apt-get update
  ```

#### 其他应用

- [Crossover](https://wiki.deepin.org/index.php?title=Crossover)：Crossover是一款可以在Linux中运行Windows软件和游戏的工具

  [www.codeweavers.com](www.codeweavers.com)

- [PlayOnLinux](https://wiki.deepin.org/index.php?title=PlayOnLinux)：PlayOnLinux是一款Wine的前端程序

  <https://www.playonlinux.com/>

- [Wine](https://wiki.deepin.org/index.php?title=Wine)：Wine是一个能够在多种作系统（Linux，macOS 及 BSD 等）上运行 Windows 应用的兼容层

  <https://www.winehq.org/>

- [LibreCAD](https://wiki.deepin.org/index.php?title=LibreCAD)：LibreCAD是一款2D的CAD绘图工具

  <http://librecad.org/>

- [FreeCAD](https://wiki.deepin.org/index.php?title=FreeCAD)：FreeCAD是一款通用的三维CAD建模软件

  <http://www.freecadweb.org/>

- [DraftSight](https://wiki.deepin.org/index.php?title=DraftSight)：DraftSight是一款专业的2D CAD制图软件

  <http://www.3ds.com/products-services/draftsight/>

- [Sweet Home 3D](https://wiki.deepin.org/index.php?title=Sweet_Home_3D)：Sweet Home 3D是一款家装辅助设计软件

  <http://www.sweethome3d.com/>

- [Stellarium](https://wiki.deepin.org/index.php?title=Stellarium)：Stellarium是一款虚拟星象仪的计算机软件

  <http://www.stellarium.org/>

- [QElectroTech](https://wiki.deepin.org/index.php?title=QElectroTech)：QElectroTech是一款电路图绘制软件

  <http://qelectrotech.org/>

- [GNU Octave](https://wiki.deepin.org/index.php?title=GNU_Octave)：GNU Octave是一种高级编程语言

  <https://www.gnu.org/software/octave/>

- [BirdFont](https://wiki.deepin.org/index.php?title=BirdFont)：BirdFont是一个免费的字体编辑器

  <https://birdfont.org/>

- [QCad](https://wiki.deepin.org/index.php?title=QCad)：QCad是一款2D计算机辅助画图软件

  <http://www.qcad.org/>

- [GnuCash](https://wiki.deepin.org/index.php?title=GnuCash)：GnuCash是一款适用于个人或小型企业的财务软件

  <http://www.gnucash.org/>

- [BOINC](https://wiki.deepin.org/index.php?title=BOINC)：BOINC是一款用来贡献计算资源的应用

  <http://boinc.berkeley.edu/>

- [Room Arranger](https://wiki.deepin.org/index.php?title=Room_Arranger)：Room Arranger是一款实时的模拟房屋设计布局的软件

  <http://www.roomarranger.com/>

- [Scribus](https://wiki.deepin.org/index.php?title=Scribus)：Scribus是一款电子杂志制作软件

  <https://www.scribus.net/>

- [KRuler](https://wiki.deepin.org/index.php?title=KRuler)：KRuler是一款制定屏幕分辨率规则和颜色测量的工具

  <https://www.kde.org/applications/graphics/kruler/>

- [Dr.Geo](https://wiki.deepin.org/index.php?title=Dr.Geo)：Dr.Geo是一款交互式的几何形状分布的应用

  <http://www.drgeo.eu/>

- [wxMaxima](https://wiki.deepin.org/index.php?title=WxMaxima)：wxMaxima是一个基于wxWidgets的计算机代数系统应用

  <http://andrejv.github.io/wxmaxima/>
  
- uTools是一个极简、插件化、跨平台的现代桌面软件。通过自由选配丰富的插件，打造你得心应手的工具集合。

  <https://www.u.tools/>

#### 运维工具

- mkcert:证书工具

  <https://github.com/FiloSottile/mkcert/>

- certbot:htps证书工具

  <https://certbot.eff.org/>

- [Angry IP Scanner](https://angryip.org/download/)

  Angry IP Scanner是一个开源网络和IP扫描工具。该软件扫描连接到网络的IP地址，并检查每个设备的状态和可用性。Angry IP Scanner通过为每个扫描的IP地址创建单独的线程，使用多线程方法进行监控;这样可以提高工具IP监控的速度。该工具还支持NetBIOS信息，IP地址范围，Web服务器检测和可自定义的开启工具。

- [Cacti](https://www.cacti.net)

  Cacti是一个基于RRDTool数据记录和图形系统的开源网络监控工具。该工具使用网络轮询和数据收集功能来收集任何规模的网络上的设备信息。这包括为数据收集设计自定义脚本以及支持SNMP轮询的能力。然后，它会以易于理解的图形显示此信息，这些图表可以按照业务最适合的层次结构进行排列。

- [Nagios Core](https://www.nagios.org)

  Nagios Core是一种开源网络监控工具，旨在作为Nagios提供的其他监控和警报软件的基础。它主要是一种性能检查工具，可以在整个基础架构中调度和执行网络性能检查。作为其他Nagios软件使用的性能检查事件处理器，Nagios Core还可以通过Naigos Exchange通过独立的附件扩展其功能。

- [ZABBIX](https://www.zabbix.com)

  Zabbix是一个开源监控工具套件，包括网络监控。Zabbix的网络监控功能包括性能指标分析，例如带宽使用，数据包丢失和CPU/内存利用率。它还可以通过检查处于危急状态的设备来检测网络节点和连接健康问题。当硬件功能下降(网络设备的风扇速度低)或未响应SNMP检查时，Zabbix可以提醒你。

  **[监控三剑客<cacti、nagios、zabbix>](https://blog.51cto.com/13645280/2165369)**

- [Checkmk](https://checkmk.com)

  Checkmk是一个开源的基础架构和应用监控工具，还包括网络监控功能。对于网络监控，Checkmk可以发现和监控交换机和路由器，无线网络和防火墙;该软件支持与多个网络硬件供应商的集成。该解决方案使用基于规则的概念来配置网络和设备监控，允许企业配置整个网络以监控特定指标。

- [Icinga](https://icinga.com)

  Icinga是一个开源网络监控工具，可以测量网络可用性和性能。通过Web界面，企业可以在整个网络基础架构中观察主机和应用。该工具具有原生可扩展性，可以轻松配置为适用于各种设备。还有一些用于特定监控功能的Icinga模块，例如监控VMWare的vSphere云环境和业务流程建模。

- [LibreNMS](https://www.librenms.org)

  LibreNMS是一个开源网络监控系统，它使用多种网络协议来观察网络中的每个设备。LibreNMS API可以检索，管理和绘制其收集的数据，并支持水平扩展，以增强其网络监控功能。该工具具有灵活的警报系统，可根据企业的方法量身定制，以便与你沟通。它们还提供原生iOS和Android应用程序。

- [NetXMS](https://www.netxms.org)

  NetXMS是一种开源基础架构和网络监控和管理解决方案。该工具为IT基础架构的所有层提供灵活的事件处理，报告和可视化图形。对于网络监控，NetXMS提供自动第2层和第3层发现以及完整的SMNPv3支持。该程序还包括主动和被动发现，将扫描探针和信息收集功能结合在一起。

- [ntopng](https://github.com/ntop/ntopng)

  ntopng是一种开源网络流量分析工具，具有网络监控功能。该工具是一种网络流量探测器，可将网络流量分类为不同的标准，包括IP地址和吞吐量。通过表征网络流量，企业可以轻松确定影响网络的不同网络统计信息。虽然ntopng的社区版本是开源的，但也可以使用专业版和企业版。

- [Opmantek NMIS](https://opmantek.com/network-management-system-nmis/)

  Opmantek NMIS是一种开源网络管理解决方案，用于可扩展的网络性能和设备状态监控。NMIS根据业务影响对网络事件进行分类。NMIS包含在Opmantek的NMIS专业软件包中，该软件包还包括用于通过可自定义仪表板和opReports绘制性能图表的opCharts，用于分析性能数据和生成有关此数据的报告。

- [Pandora FMS](https://pandorafms.com)

  Pandora FMS是一种开源监控工具，可帮助企业观察整个IT基础架构。它不仅具有网络监控功能，还具有Windows和Unix服务器以及虚拟接口。对于网络，Pandora FMS包含诸如ICMP轮询，SNMP支持，网络延迟监控和系统过载等功能。还可以在设备上安装代理，以观察设备温度，以及日志文件等因素。

- [Prometheus](https://prometheus.io)

  Prometheus是一个专注于数据收集和分析的开源监控解决方案。它允许用户使用本机工具集设置网络监控功能。该工具能够使用SNMP ping收集设备上的信息，并从设备角度检查网络带宽使用情况以及其他功能。PromQL系统分析数据并允许程序在其监控的系统上生成图形，表格和其他视觉效果。

- [Wireshark](https://www.wireshark.org)

  Wireshark是一种开源网络协议分析仪，具有实时网络数据捕获和分析功能。该工具对多个不同的网络协议执行深度检查，以确定多个级别的网络性能。Wireshark还允许用户捕获数据包并分析它们，即使在网络脱机时也是如此。Wireshark捕获的数据可以以多种通用或共享文件格式存储，允许其他工具帮助解释网络上的数据。

- [mmonit](https://mmonit.com)

  Monit是一个开源监控管理工具（类似supervisor），能够监控linux系统的负载、文件、进程等。当系统负载过高、监控文件被篡改、进程异常退出时，能够发送邮件报警，并能够自动启动或关闭异常进程。Monit内嵌web界面，能够看到当前主机上的监控项状态。

  M/Monit是一个集中式管理多台Monit的可视化工具，也是收费工具，可以免费试用30天。

- [manageengine](https://www.manageengine.cn)

  监控工具公司

- [goaccess](https://www.goaccess.cc)

  **GoAccess** 是一款开源的且具有交互视图界面的**实时** **Web 日志分析工具**，通过你的 **Web 浏览器**或者 *nix 系统下的**终端程序(terminal)**即可访问。

  能为系统管理员提供**快速**且有价值的 HTTP 统计，并以在线可视化服务器的方式呈现。

- [lnmp一键安装](https://lnmp.org)

  LNMP一键安装包是一个用Linux Shell编写的可以为CentOS/RHEL/Fedora/Aliyun/Amazon、Debian/Ubuntu/Raspbian/Deepin/Mint Linux VPS或独立主机安装LNMP(Nginx/MySQL/PHP)、LNMPA(Nginx/MySQL/PHP/Apache)、LAMP(Apache/MySQL/PHP)生产环境的Shell程序。



### 参考

1. <https://cloud.tencent.com/developer/article/1494803>
2. [常用的超赞 Linux 软件大汇总，入行运维必藏！](https://alim0x.gitbooks.io/awesome-linux-software-zh_cn/content/)
3. [适用于 Linux 系统的 10 款最佳媒体服务器软件（上）](https://www.sysgeek.cn/linux-media-server-top10-1/)
4. [适用于 Linux 系统的 10 款最佳媒体服务器软件（下）](https://www.sysgeek.cn/linux-media-server-top10-2/)
5. [如何使用 Slimbook Battery 实现 Ubuntu 高级电源管理](https://www.sysgeek.cn/slimbook-battery-ubuntu/)
6. [八款优秀的Linux天文学软件](http://www.huacolor.com/article/25456.html)

