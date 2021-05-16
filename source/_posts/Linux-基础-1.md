---
title: Linux-使用虚拟机安装CentOS7
date: 2019-03-12 11:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

环境：Windows 10操作系统

工具：VMware Workstation 15 、CentOS 7 、putty、winscp

### 准备

在Windows 10下安装所需软件、下载所需ISO文件

<!--more-->

**VMware 安装**

在VMware官网[下载](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3mgt4cj310m0m1jse.jpg)

​	下载完成后直接安装就行，如果没有注册码，只会有一个月的试用期，可以从下面链接获取注册码。

注册码链接：https://pan.baidu.com/s/1KkC4n9F3BGod3rmDN1XbZQ        提取码：rhs3 

**CentOS 7下载**

推荐使用国内下载源，官网下载速度太慢了。下面给出国内常用的下载源：

```html
http://mirrors.btte.net/centos/7/isos/x86_64/ 
http://mirrors.cn99.com/centos/7/isos/x86_64/ 
http://mirrors.sohu.com/centos/7/isos/x86_64/ 
http://mirrors.aliyun.com/centos/7/isos/x86_64/ 
http://centos.ustc.edu.cn/centos/7/isos/x86_64/ 
http://mirrors.neusoft.edu.cn/centos/7/isos/x86_64/ 
http://mirror.lzu.edu.cn/centos/7/isos/x86_64/ 
http://mirrors.163.com/centos/7/isos/x86_64/ 
http://ftp.sjtu.edu.cn/centos/7/isos/x86_64/ 
```
​	一般包括以下文件，README.txt是对镜像文件和torrent文件的描述，可以选择下载相应的torrent文件，然后使用加速下载器进行下载，也可以选择iso镜像文件下载，可能会比较大。DVD包括我们经常用到的安装包；Everything包括了发行版发行时的几乎所有安装包；Live是一个测试包，目的在于测试安装环境，LiveGNOME和LiveKDE是两种不同的GUI软件；minimal是迷你包，只包含最小安装（后面会谈到）；NetInstall是网络安装和救援的镜像，使用该镜像会被要求提供安装包链接，而且安装时必须联网。sha256sum.txt是验证文件，可以比对下载的文件是否损坏、被修改。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3fel4pj30gh07t0sn.jpg)

​	如果对系统没有安装时就要求具有什么特别强大的功能，最好是使用最小化安装，DVD和Minimal是常用的包，如果网速较慢，可以选择下载Minimal包，如果想要安装图形界面，选择下载DVD，Everything包太大并不推荐下载使用。

Now，使用VMware创建CentOS虚拟机。

### 创建虚拟机

打开VMware程序，进入主页，选择创建新的虚拟机。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3igwcwj30wc0gbaa2.jpg)

然后选择新建虚拟机向导为自定义，其中典型为简单配置，配置少，适合没有很多要求的用户，自定义包括典型中的所有配置，还增加了硬件设置（虚拟硬件），来选择自定义来学习新建的步骤，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3iclxnj30kp0icmxb.jpg)

选择硬件兼容性，使用最新版，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3im3e9j30oy0j5dft.jpg)

如果使用U盘或光盘安装，请选择程序光盘，如果确定所使用的客户机操作系统，可以直接选择ISO包，此处我们选择稍后安装操作系统，否则会跳过下一步，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3ir5fjj30l00iimx4.jpg)

选择对应的客户机操作系统，此处为Linux 的CentOS 7 64位，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3g6u5fj30k90hq0sn.jpg)

虚拟机命名，需要符合命名规则，尽量使其有意义，这里默认；选择安装位置，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3gow7hj30ii0hqgli.jpg)

设置处理器，默认，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3gfue9j30j70hz745.jpg)

设置内存大小，默认，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3j47lij30i80ghweg.jpg)

设置网络类型，默认下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3j0fvkj30hw0i1mx4.jpg)

设置IO控制器类型，默认，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3gsn2uj30j50ggmx2.jpg)

选择磁盘类型，默认，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3gw275j30hk0h0glh.jpg)

选择磁盘，如果存在虚拟磁盘，可以选择现有虚拟磁盘(.vmx),如果想使用U盘进行安装，可以选择使用物理磁盘，这里我们选择新建虚拟磁盘，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3jab4hj30j80gz748.jpg)

接下来设置虚拟磁盘大小，如果可以尽量设置为100GB左右，其它选项默认，选择不立即分配，后续使用中虚拟磁盘文件占用空间会变大，拆分存储，下一步

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3jiv0fj30j30hq749.jpg)

接下来指定虚拟磁盘文件命名，默认，下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3h45h6j30hd0gwt8l.jpg)

最后可以看到所有的配置信息，如果想更改可以选择上一步，或者点击完成后在设置中更改，默认下一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3jr0zkj30if0haaa0.jpg)

然后打开设备设置--》CD/DVD，选择下载完成的ISO映像文件，点选启动时连接。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3jvjytj30o10dgwei.jpg)

此时，我们已经在Windows上新建完成一个虚拟机，但是由于该虚拟机未安装操作系统，不能直接运行，下一节讲怎样安装。

### 安装

​	现在开始在虚拟机上安装CentOS7。打开CentOS 7 64位选项卡，点击上面菜单栏中的开启，打开虚拟机的电源。

#### 安装模式选择（Mode Selection）

​	这三个选项分别是安装、测试镜像和安装、进入救援模式。安装选项会直接进入安装界面；第二选项会先对镜像文件进行检查，之后才进行安装；第三个选项是利用此镜像包对已安装的系统进行恢复，按Enter键以显示其它内容。这里我们选择第二个选项，然后回车。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3fs1fyj30md0dfmwx.jpg)

​	接下来CentOS会进入自检过程，情形如下图所示，等待2-3分钟后会完成这一步。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3fjq68j30l70g4747.jpg)

#### 选择语言（Language Selection）

​	然后会进入到安装引导语言选择过程，此处选择“中文Chinese - 简体中文(中国)”；点击“继续”，进入下一步。

> Tips：在此选择的语言将成为操作系统的默认语言。选择适当的语言还可帮助您在后面的安装中锁定时区。安装程序会尝试根据您在这个页面中的选择定义适当的时区。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3llq6zj30nq0hp750.jpg)

​	进入安装配置界面，主要有本地化（时区、时间、语言和键盘）、软件（安装源、软件选择）、系统（分区配置、网络配置等）。

> 使用鼠标来选择菜单项来配置安装的一部分。当您完成配置部分，或者如果您想以后完成的部分中，单击完成按钮位于屏幕的左上角。
>
> 只有标有警告符号的部分是强制性的。在屏幕底部的注意事项提醒你，这些路段之前必须完成安装就可以开始。其余的部分都是可选的。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3m9kxtj30n30hcwfd.jpg)

#### 网络和主机名（Network & Hostname）

​	第一步应该配置网络和主机名（Network & Hostname），默认虚拟机是和主机进行以太网连接，直接选择打开网卡，默认使用DNCP服务获取IP地址、子网掩码、默认路由、DNS等，同时，在IPv6的配置被设定为自动方式。这种组合适合于大多数安装方案，通常不需要任何修改。也可以选择更改主机名，应注意同一局域网内主机名不应重复，主机名可以是完全限定域名（FQDN）格式hostname.domainname或主机名格式 的短主机名。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3kxw1sj30nd0hnmxo.jpg)

​	如果想要手动配置网络，可点击配置，进入配置IP地址或路由地址等信息，配置过程参考网上博文介绍（一般不推荐在安装过程进行修改），此处不进行修改。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3lbpczj30nh0hn74q.jpg)

#### 日期和时间、时区（Date & Time）

​	日期和时间、时区选择（Date & Time）。选择距离您计算机物理位置最近的城市设置时区。即使您要使用 NTP（网络时间协议）来维护准确系统时钟，也请指定时区。这里您有两种方法选择时区：用鼠标在互动式地图上点击指定城市；您还可以在屏幕上部的列表中选择。

​	网络时间（Network Time Protocol）。如果设备连接到网络时，网络时 间开关将被启用。要设置使用NTP的日期和时间，留在ON位置的网络时间开关，然后点击配置图标，选择使用哪一个NTP服务器，点击完成后，再进来查看是否完成。要手动设置日期和时间，将 开关置于OFF位置。系统时钟应该使用时区选择显示在屏幕的底部正确的日期和时间。如果他们仍然不正确，请手动进行调整。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3mwgt1j30mw0hndh6.jpg)

#### 键盘布局类型（Keyboard Configuration）

​	然后设置键盘布局类型（Keyboard Configuration）。选择您用于安装的正确键盘布局类型“汉语”并将其作为系统默认选择。选择后，点击“完成”。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3kkabtj30mr0he0t5.jpg)

#### 语言支持（Language Support）

​	接着进行语言支持（Language Support）设置。要安装额外的语言环境和语言的方言，从安装摘要屏幕中，选择多个语言的支持。使用鼠标来选择语言，你会为它要安装支持。在左侧面板中，选择您所选择的语言，比如English。然后你可以选择专门针对在右侧面板中您所在地区的语言环境，例如English（United States）。您可以选择多种语言和多个地区。所选择的语言以粗体突出显示在左侧面板。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3kteagj30nq0hhwf1.jpg)

#### 安装源（Installation Source）

​	选择安装源安装系统的一个位置。在这个屏幕上，你可以在本地可用的安装媒体，例如DVD或ISO文件，或者某个网络位置之间进行选择。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3kotcoj30my0h8wf1.jpg)

选择下列选项：

**自动检测安装介质（Auto-detected installation media）**

​	如果您使用完整安装DVD或USB驱动器启动安装时，安装程序会检测到它，并显示在此选项下的基本信息。点击“验证（Verify ）”按钮，以确保媒体适合于安装。这个完整的测试是一样的，如果你选择了这个测试媒体和在启动菜单中安装CentOS所执行的一个，或者如果你使用的“测试媒介及安装CentOS 7”启动选项。

**ISO file（ISO文件）**

​	此选项会出现，如果安装程序检测到一个分区的硬盘驱动器挂载的文件系统。选择此选项，请单击“选择的ISO（Choose an ISO）”按钮，并浏览到安装ISO文件的位置，您的系统上。然后点击“验证”，以确保文件是适合安装。

**On the network（在网络上）**

​	要指定一个网络位置，选择此选项，并从下拉菜单中选择以下选项中进行选择：http://、https://、ftp://、nfs。使用您选择的位置URL的开始，输入到地址框。如果您选择NFS，将会出现你指定的任何NFS挂载选项。如果您的HTTP或HTTP URL指向存储库镜像列表，勾选“该URL指向一个镜像列表”。要配置HTTP或HTTPS源的代理，单击“代理服务器设置”按钮。勾选“启用HTTP代理服务器”，然后输入网址到代理服务器的URL框。如果你的代理服务器需要身份验证，请使用验证，并输入用户名和密码。单击添加。

**额外软件仓库（Additional repositories）**

​	要添加一个存储库中，单击+按钮。要删除存储库中，单击 - 按钮。点击箭头图标恢复到仓库的前面的列表，即以取代那些出席输入的安装源画面时的输入。要激活或停用库，请单击复选框启用列在列表中的每个条目。在窗体的右侧，你可以命名你的额外的资料库，并配置相同的方式在网络上的主存储库。

#### 软件选择（Software Selection）

​	要指定软件包将被安装，选择软件时选择安装摘要屏幕。包组分为基础环境。这些环境是预先定义的一组具有特定用途的软件包；例如，在虚拟化主机环境中包含的一组所需的系统上运行的虚拟机软件程序包。只有一个软件环境可以在安装时选择。

​	对于每一个环境，也有附加组件的形式提供额外的软件包。附加组件列于屏幕的右侧部分，当选择一个新的环境，它们的列表被刷新。**您可以选择多个附加组件的安装环境。选择“带GUI的服务器（Server with GUI）”，然后按“完成”键**。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3mrk5oj30nk0i10ty.jpg)

**最小安装（Minimal Install）**

​	这个选项只提供运行CentOS 的基本软件包。最小安装为单一目的服务器提供基本需要，并可在这样的安装中最大化性能和安全性。

**基础设施服务器（Infrastructure Server）**

​	这个选项提供在服务器中使用的CentOS 基本安装，不包含桌面。

**文件及打印服务器（File and Print Server）**

​	用于企业的文件、打印及存储服务器。

**基本网页服务器（Basic Web Server）**

​	基本系统平台，加上PHP，Web server，还有MySQL和PostgreSQL数据库的客户端，无桌面。

**虚拟化主机（Virtualization Host）**

​	这个选项提供 KVM 和 Virtual Machine Manager 工具以创建用于虚拟机器的主机。

**带GUI的服务器（Server with GUI）**

​	带有用于操作网络基础设施服务GUI的服务器。

**GNOME桌面（GNOME Desktop）**

​	GNOME是一个非常直观且用户友好的桌面环境。

**KDE Pasma Workspaces（KDE桌面）**

​	一个高度可配置图形用户界面，其中包括面板、桌面、系统图标以及桌面向导和很多功能强大的KDE应用程序。

**开发及生成工作站（Development and Creative Workstation）**

​	这个选项提供在您的CentOS 编译软件所需的工具。

> Tips：如果您使用文本模式安装Linux，您不能进行软件包选择。安装程序只能自动从基本和核心组群中选择软件包。这些软件包足以保证系统在安装完成后可操作，并可安装更新和新的软件包。要更改软件包选择，请在完成安装后，使用 Add/Remove Software 程序根据需要进行修改。

#### 安装位置（Installation Destination）

​	要选择的磁盘和分区的存储空间，您将安装CentOS的，请在安装摘要屏幕安装目标。

> Tips：如果您在文本模式下安装CentOS的，你只能使用本节所述的默认分区方案。您不能添加或删除的分区或文件系统超出了安装程序会自动添加或删除。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3lrx4bj30nl0hmt9g.jpg)

选项如下：

1、特别的及网络磁盘（Specialized & Network Disks）

​	在这个屏幕上，你可以看到可用的本地计算机上的存储设备。您也可以通过单击“添加磁盘”按钮来添加额外的专用或网络设备。

2、自动配置分区（Automatically configure partitioning）

​	自动分区建议，如果你正在做一个干净的安装上以前未使用的存储或不需要让可能存在于存储任何数据。采用这种方式，离开自动配置分区单选按钮的默认选择，让安装程序在你的存储空间，创造必要的分区。

3、我想额外的空间使用（I would like to make additional space available）

​	对于自动分区，你还可以选择“我想使更多的可用空间”复选框来选择如何重新分配其他的文件系统空间。如果您选择了自动分区，但没有足够的存储空间来完成安装，在单击Done（完成），会出现一个对话框。

4、加密我的数据（Encrypt my data）

​	在加密部分中，您可以选择加密我的数据复选框，所有分区进行加密以外的/ boot分区。

5、完成磁盘摘要以及引导程序（Full disk summary and bootloader）

​	在屏幕的底部，是您配置在其上的引导加载程序将被安装在磁盘全盘总结和引导装载程序按钮。

#### 手动分区（Manual Partitioning）

​	新手一般推荐选择自动分区，如果想要选择手动分区，必须对linux分区和文件系统有一定了解。

​	选择手动分区，然后点击完成进入手动分区窗口。手动分区屏幕功能最初在左侧的安装点的单一窗格。该面板是空的，除了有关创建挂载点，也显示现有安装的安装程序检测点。这些装载点被检测到操作系统安装举 办。因此，如果一个分区被几个设备之间共享某些文件系统可能会显示多次。在选择存储设备的总空间和可用空间会显示此窗格下方。

**添加文件系统和分区配置**

​	添加一个文件系统是一个两步骤的过程。首先创建一个特定的分区方案的挂载点。挂载点会出现在左窗格中。接下来，你可以用在右窗格中，您可以在其中选择一个名称，设备类型，文件系统类型，标签的选项进行自定义，以及是否加密或重新格式化相应分区。

​	对于手动创建的每个新的挂载点，可以设置从位于左侧窗格中的下拉菜单中的分区方案。可用的选项有标准分区，BTRFS，LVM和LVM精简配置。需要注意的是/ boot分区将始终位于一个标准分区，无论在此菜单中选择的值。

​	另外，创建使用“+”按钮在面板的底部各个挂载点。添加一个新的安装点对话框，然后打开。无论是选择从安装点下拉菜单中预设的路径或键入您自己的 - 例如，选择/根分区或者/ boot启动分区。然后输入分区的大小，使用普通大小的单位，如MB，GB或TB，到需要的容量文本字段 - 例如，键入2GB创建一个2G的分区。如果将该字段留空，或者如果你指定的大小大于可用空间，所有剩余的可用空间将被使用。进入这些细节之后，点击“添加挂载点”按钮来创建分区。

​	要自定义分区或卷时，在左侧窗格中选择挂载点及以下的自定义功能，然后出现在右边：

- 名称（Name）：分配一个名称，一个LVM或于Btrfs量。需要注意的是在创建时标准分区自动命名和他们的名字不能被编辑，如/ home被分配的名称SDA1。

- 挂载点（Mount point）：输入分区的挂载点。例如，如果分区是根分区，输入/;进入/启动的/ boot分区，依此类推。对于一个交换分区，挂载点不应设置 - 设置文件系统类型，以交换就足够了。


- 标签（Label）：分配一个标签的分区。标签是用来让你很容易地识别和解决单个分区。


- 所需的容量（Desired capacity）：输入的分区的所需尺寸。您可以使用普通大小的单位，如千字节，兆字节，GB或TB。兆是默认选项，如果你不指定任何单位。


- 设备类型（Device type）：标准分区，BTRFS，LVM或LVM精简配置之间进行选择。如果被选择用于分隔两个或多个磁盘，RAID也将是可用的。检查相邻的加密框，分区加密。系统将提示您以后设置密码。



下面是设备类型简短描述，以及它们是如何被使用：

- 标准分区：标准分区可以包含文件系统或交换空间，也能提供一个容器，用于软件RAID和LVM物理卷。


- 逻辑卷（LVM）：创建一个LVM分区自动生成一个LVM逻辑卷。 LVM可以在使用物理磁盘时，提高性能。


- LVM精简配置：使用自动精简配置，你可以管理的自由空间，被称为精简池，它可以根据需要由应用程序时，可以分配给设备任意数量的存储池。所需的存储空间具有成本效益的分配时，薄池可以动态地扩展。


- BTRFS：Btrfs是一个具有几个设备相同的特征的文件系统。它能够处理和管理多个文件，大文件和大体积比的ext2，ext3和ext4文件系统。


- 软件RAID：创建两个或两个以上的软件RAID分区允许你创建一个RAID设备。一个RAID分区被分配给每个磁盘的系统上。


- 文件系统（File system ）：在下拉菜单中，选择该分区中的相应的文件系统类型。检查相邻的格式化对话框格式化现有的分区，或将其选中，以保留您的数据。


下面是文件系统简短描述，以及它们是如何被使用：

- XFS：XFS是一个支持的文件系统多达16艾字节（约16万TB）一个高度可扩展，高性能的文件系统中，文件多达8个艾字节（约800万太字节），和目录结构包含数千万条目。 XFS支持元数据日志，这有利于更快的崩溃恢复。 XFS文件系统也可以进行碎片整理和调整，同时安装并激活。这个文件系统是默认选择，并强烈推荐。一个XFS分区支持的最大大小为500 TB。


- EXT4：ext4文件系统是基于ext3文件系统，并采用了多项改进。这些措施包括对更大文件系统和更大的文件，磁盘空间，对子目录的目录中的数量没有限制，更快的文件系统检查速度更快，更有效地分配支持，更强大的日志记录。


- EXT3：ext3文件系统是基于ext2文件系统上，它有一个主要优点。使用文件系统减少花费的时间恢复崩溃后的文件系统，因为没有必要通过每次碰撞发生时运行fsck实用程序来检查元数据的一致性的文件系统。


- EXT2：ext2文件系统支持标准Unix文件类型，包括普通文件，目录或符号链接。它还提供了分派长文件名，最多255个字符的能力。


- VFAT：VFAT文件系统是Linux文件系统与FAT文件系统上的Microsoft Windows长文件名兼容。


- swap：交换分区被用于支持虚拟内存。换句话说，数据被写入到交换分区的时候没有足够的内存来存储您的系统正在处理的数据。


- BIOS boot：需要有一个GUID分区表（GPT）在BIOS中的系统引导设备一个非常小的分区。


- EFI系统分区：需要有一个GUID分区表（GPT）在UEFI系统引导装置一个小分区。

​	单击“更新按钮”保存更改并选择其他分区进行定制。请注意，更改将不会应用，直到你真正开始从安装摘要页面安装。单击“全部重置”按钮来放弃所有修改的所有分区并重新开始。

**创建LVM逻辑卷（Create LVM Logical Volume）**

逻辑卷管理（LVM）介绍了底层的物理存储空间，如硬盘驱动器或LUN一个简单的逻辑视图。在物理存储分区表示为物理卷，可以组合成卷组。每个卷组可以分为多个逻辑卷，其中每一个是类似于标准的磁盘分区。因此，LVM逻辑卷作为可以跨越多个物理磁盘上的分区。

Tips：需要注意的是LVM配置只有在图形安装程序中可用。 在文本模式安装，LVM配置不可用。如果你需要从头开始创建一个LVM配置，按Ctrl + Alt+ F2来使用不同的虚拟控制台，并运行LVM命令。要返回到文本模式安装，请按CTRL + ALT + F1。

CentOS的安装至少需要一个分区，但CentOS建议至少有四个：/boot、/、/home、swap。您还可以创建你需要额外的分区。

1. 创建/boot


​	分区挂载到/boot包含了操作系统的内核，它允许你的系统CentOS，以及在系统引导过程中使用的文件。由于大多数固件的限制，创建一个较小的分区来保存这些建议。在大多数情况下，一个500 MB的启动分区是足够的。

​	首先创建一个特定的分区方案的挂载点。挂载点会出现在左窗格中的下拉菜单中选择“LVM”。

​	使用“+”按钮在面板的底部各个挂载点。添加一个新的安装点对话框，然后打开。无论是选择从安装点下拉菜单中预设的路径或输入/boot。然后输入分区的大小，使用普通大小的单位500M。点击“添加挂载点”按钮来创建分区。

2. 创建/

​	这是“/”，或者根目录下，是位于根目录是目录结构的顶层。默认情况下，所有的文件将被写入该分区，除非在不同的分区上安装路径写入（例如，/boot或 /home）。虽然有5GB的根分区，可以安装一个最小的安装，建议至少分配10GB，这样就可以完全安装，选择所有软件包组。

> Tips：不要混淆/目录下的/root目录。 /root目录是root用户的主目录。 /root目录有时被称为斜线根从根目录区分开来。
>

​	使用“+”按钮在面板的底部各个挂载点。添加一个新的安装点对话框，然后打开。无论是选择从安装点下拉菜单中预设的路径或输入/。然后输入分区的大小，使用普通大小的单位8GB。点击“添加挂载点”按钮来创建分区。

3. 创建/home

​	从系统数据分开存储的用户数据，创建卷组的/home目录内的专用分区。这个分区应根据将要存储在本地，用户数量等的数据量的大小。这将使你升级或重新安装CentOS，但不删除用户的数据文件。

​	使用“+”按钮在面板的底部各个挂载点。添加一个新的安装点对话框，然后打开。无论是选择从安装点下拉菜单中预设的路径或输入/home。然后输入分区的大小，使用普通大小的单位10GB。点击“添加挂载点”按钮来创建分区。

4. 创建swap

​	交换分区支持虚拟内存;数据被写入到交换分区的时候没有足够的内存来存储您的系统正在处理的数据。交换容量是系统存储器的工作量，而不是整个系统内存的功能，因此是不等于系统的总的内存大小。因此，分析系统将运行哪些应用程序和负载这些应用将服务，以确定该系统存储器工作量是非常重要的。

​	使用“+”按钮在面板的底部各个挂载点。添加一个新的安装点对话框，然后打开。无论是选择从安装点下拉菜单中预设的路径或输入swap。然后输入分区的大小，剩余容量。点击“添加挂载点”按钮来创建分区。

​	当已创建和定制的所有文件系统和挂载点，点击“完成”按钮。如果您选择任何加密文件系统，你将被提示创建密码。然后，会出现一个对话框，显示所有与存储相关 的操作，安装程序将采取的摘要。这包括创建，调整大小或删除分区和文件系统。您可以查看所有的更改，然后单击“取消并返回到自定义分区”回去。要确认摘 要，单击“接受更改 ”，返回到安装摘要页面。分区的任何其它设备，在安装目标选择它们，回到手动分区屏幕，并按照本节所述相同的过程，一般配置如下即可。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3lxr1cj30nf0hkmxu.jpg)

#### 开始安装（Begin Installation）

​	剩下的可以保持默认配置，此时开始安装按钮变得可用，点击按钮进行安装。

​	一旦你点击在安装摘要屏幕开始安装，会出现进度画面。屏幕上的安装进度，代表了选择的软件包安装到你的系统。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3mly0tj30mt0gzwfj.jpg)

**ROOT密码（Root Password）**

​	设置根帐号和密码是在安装过程中的重要一步。 root帐号（也称为超级用户）来安装软件包，升级RPM包，并执行大部分的系统维护。 root帐户可以完全控制您的系统。出于这个原因，根帐户最好只用来执行系统维护或管理。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3keaflj30n60h1glw.jpg)

**创建用户（User Account）**

​	在安装过程中创建一个普通的（非root）用户帐户，请单击进度屏幕上的用户设置。创建用户屏幕，让您设置普通用户帐号，并配置其参数。虽然建议在安装过程中办，这一步是可选的，后安装完成后可以进行。

​	在普通环境下，一般需要设置普通用户，且不允许root用户远程登录服务器。此处我们创建一个新用户hyp,配置密码，可以配置为管理员，也可以开机后登录root用户，然后更改为管理员。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3k9d49j30ni0hhjrr.jpg)

**安装完成（Installation Complete）**

​	点击重启按钮，重新启动系统并开始使用CentOS。记得要取出安装介质，如果它会自动重新启动时不弹出。

### 基本配置

#### 远程登录

远程登录有多种方式，其中最流行的CLI远程登录协议是SSH协议，让我们认识一下SSH协议：

*SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH最初是UNIX系统上的一个程序，后来又迅速扩展到其他操作平台。SSH在正确使用时可弥补网络中的漏洞。SSH客户端适用于多种平台。几乎所有UNIX平台—包括HP-UX、Linux、AIX、Solaris、Digital UNIX、Irix，以及其他平台，都可运行SSH。*

​	CentOS7 安装完成后重启，使用root登录。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3hg8k1j30j807ngld.jpg)

​	检查网络是否连通，使用命令`ping -c 4 baidu.com`，一般情况下是正常可连的，如果出错，请参考网上博文。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3hp89uj30id04bmwx.jpg)

​	查看是否开启SSH协议：

```shell
[hyp@localhost ~]$ ps -aux | grep ssh
root       7127  0.0  0.4 112760  4324 ?        Ss   19:21   0:00 /usr/sbin/sshd -D
```

​	使用 `ip -4 addr`查看本地IP地址，可以看到本地IP地址为192.168.174.133。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3i01xej30nk03h3ya.jpg)

​	在Windows上安装Putty软件，官网下载地址如下：请下载[最新版](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)。然后安装，安装完成后选择打开PUTTY.exe文件。然后输入虚拟机的IP地址

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3lfth1j30wm0ggaam.jpg)

输入root，回车后输入密码，密码输入时不会显示。可以通过`ip -4 addr`查看ip。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3k45k5j30me09jjre.jpg)



#### 数据传输

​	虽然可以通过远程登录来登录CentOS 7，对CentOS进行操作，但是不能进行CentOS和Windows之间文件的传输。

首先介绍一下FTP：

*FTP就是文件传输协议。用于互联网双向传输，控制文件下载空间在服务器复制文件从本地计算机或本地上传文件复制到服务器上的空间。FTP协议包括两个组成部分，其一为FTP服务器，其二为FTP客户端。其中FTP服务器用来存储文件，用户可以使用FTP客户端通过FTP协议访问位于FTP服务器上的资源。在开发网站的时候，通常利用FTP协议把网页或程序传到Web服务器上。此外，由于FTP传输效率非常高，在网络上传输大的文件时，一般也采用该协议。*

*默认情况下FTP协议使用TCP端口中的 20和21这两个端口，其中20用于传输数据，21用于传输控制信息。但是，是否使用20作为传输数据的端口与FTP使用的传输模式有关，如果采用主动模式，那么数据传输端口就是20；如果采用被动模式，则具体最终使用哪个端口要服务器端和客户端协商决定。*

​	设置CentOS的FTP服务，通过安装`vsftpd`来开启ftp。

```shell
	yum install vsftpd
```

​	安装完成后，开启FTP服务。

```shell
systemctl enable vsftpd #开机自启动
systemctl start vsftpd #开启服务
systemctl status vsftpd #查看运行状态
```

​	通过PUTTY安装文件夹下PSFTP.exe连接CentOS 7，打开软件后连接，使用open连接ip，然后输入用户名、密码 。

```shell
psftp: no hostname specified; use "open host.name" to connect
psftp> open 192.168.174.133
login as: root
root@192.168.174.133's password:
Remote working directory is /root
```

使用`dir`或`pwd`测试。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13p3i8yldj30la07fq2v.jpg)

但是PSFTP.exe编码是ftp的规定文件名编码为`iso-8859-1`，这样文件名为中文的文件在传输过程中，文件名会变成乱码，而且在Windows上使用命令行的话，会有些不太友好，这样说来这就是一个不适用的解决方案。

#### 数据传输改进

新的解决方案是使用SFTP协议进行传输，使用软件WinSCP进行可视化，以达到拖拽上传下载的功能。

我们再来介绍一下SFTP协议：

*sftp是Secure File Transfer Protocol的缩写，安全文件传送协议。可以为传输文件提供一种安全的网络的加密方法。sftp 与 ftp 有着几乎一样的语法和功能。SFTP 为 SSH的其中一部分，是一种传输档案至 Blogger 伺服器的安全方式。其实在SSH软件包中，已经包含了一个叫作SFTP(Secure File Transfer Protocol)的安全文件信息传输子系统，SFTP本身没有单独的守护进程，它必须使用sshd守护进程（端口号默认是22）来完成相应的连接和答复操作，所以从某种意义上来说，SFTP并不像一个服务器程序，而更像是一个客户端程序。SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多。*

如果你的CentOS开启这SSH协议的话，就能够使用SFTP协议来传输数据。

查看是否运行SFTP也就是查看是否支持SSH协议，支持则可以运行。

在Windows上安装[WinSCP](https://winscp.net/eng/download.php)，安装完成后进行连接配置，使用SFTP协议，输入主机名和用户名，在高级配置中设置支持UTF-8编码。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13q79h7f8j30m60gtq36.jpg)



然后选择保存，下次可直接点击进行连接。保存完成后点击登录，然后输入密码，确认无误后，会在右边的窗口，以Windows的样式显示用户家目录下的所有文件，这样传输文件可以直接拖拽。



OK，由于篇幅过长，本次教程到此结束。其他内容敬请期待。



