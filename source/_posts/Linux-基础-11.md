---
title: Linux-磁盘和文件系统管理
date: 2019-03-24 09:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

上一篇文章我们学习了进程和任务管理，进程分为CPU进程和IO进程，其中IO进程是响应IO中断，处理IO任务的。内存是易失性存储，计算机对数据的存储离不开磁盘。对磁盘的管理中，可控的是磁盘的分区和修复。除此以外，我们还需要了解文件系统，文件系统为我们提供了文件存储，linux的思想之一就是一切皆文件，所以文件系统是我们必须要掌握的。今天我们就来学习一下Linux的磁盘管理和文件系统管理。

<!--more-->

### 磁盘管理

磁盘属于IO设备，通过与内存进行数据传输，来为linux内核提供数据和保存数据。只有保存到磁盘上的数据才能算是持久化数据，否则，如果遇到断电或其他故障，内存中的数据就会丢失，还好现在很多电脑内部内置了电池，可以在断电时感知到，并把内存中的数据存储到磁盘中，以此降低损失。

计算机与磁盘的接口有很多种，分别有：

```
IDE(ata)：并口，133MB/s
SCSI：并口，Ultrascsi320, 320MB/S, UltraSCSI640, 640MB/S
SATA：串口，6gbps（常用）
SAS：串口，6gbps
USB：串口，480MB/s （新晋）
```

硬盘也有机械硬盘和固态硬盘，其中机械硬盘属于传统硬盘，速度较慢。固态硬盘用固态电子存储芯片阵列而制成的硬盘，由控制单元和存储单元（FLASH芯片、DRAM芯片）组成，速度较快。

想要对操作系统原理更深一步了解的话，请阅读操作系统有关书籍。

**设备命名**

```
1、IDE硬盘：/dev/hd[a-d] 
2、SCSI/SATA/U盘硬盘：/dev/sd[a-p] 
3、U盘：/dev/sd[a-p]
4、软盘机：/dev/fd[0-1] 
5、打印机：25针: /dev/lp[0-2] & U盘: /dev/usb/lp[0-15] 
6、鼠标： /dev/usb/mouse[0-15] & PS2: /dev/psaux 
7、当前CDROM/DVDROM：/dev/cdrom 
8、当前的鼠标：/dev/mouse 
9、磁带机：IDE: /dev/ht0 
10、SCSI: /dev/st0
```

无论是磁盘还是U盘，在linux上这些设备都是以文件的形式出现。设备分为块设备和字符设备，块设备（磁盘等）随机访问，数据交换单位是“块”；字符设备（键盘等）线性访问，数据交换单位是“字符”。所有的设备文件根据FHS规定，都位于/dev目录下，设备文件关联至设备的驱动程序，是设备的访问入口。我们可以通过`ls -l /dev`发现输出的和平常的输出有所差异,在日期的前面多了一个属性，其实该属性就是设备的设备号。设备号是内核标识设备的一个编号，设备号由主设备号和次设备号组成。主设备号（major），区分设备类型，用于标明设备所需要的驱动程序；次设备号（minor），区分同种类型下的不同的设备，是特定设备的访问入口。

```shell
[hyp@localhost ~]$ ls -l /dev
总用量 0
crw-rw----. 1 root video    10, 175 3月  23 23:16 agpgart
crw-------. 1 root root     10, 235 3月  23 23:16 autofs
drwxr-xr-x. 2 root root         220 3月  23 23:16 block
drwxr-xr-x. 2 root root          80 3月  23 23:16 bsg
crw-------. 1 root root     10, 234 3月  23 23:16 btrfs-control
drwxr-xr-x. 3 root root          60 3月  23 23:16 bus
lrwxrwxrwx. 1 root root           3 3月  23 23:16 cdrom -> sr0
......
```

那么如何找到我们设备所对应的设备文件呢，如何我们就需要知道CentOS对磁盘设备文件的命名，CentOS 6和7统统将硬盘设备文件标识为/dev/sd[a-z]#，#代表磁盘上的分区号。引用设备的方式有设备文件名、卷标和UUID。

**磁盘分区**

磁盘分区方式有两种，一是MBR（Master Boot Record），分为三部分：

```
446bytes：bootloader, 程序，引导启动操作系统的程序；
64bytes：分区表，每16bytes标识一个分区，一共只能有4个分区；
	4主分区
	3主1扩展：
		n逻辑分区
2bytes：MBR区域的有效性标识；55AA为有效；
```

主分区和扩展分区的标识：1-4，逻辑分区：5+。其中逻辑分区大小不能超过扩展分区大小。扩展分区不可用与构造文件系统，只有扩展分区中的逻辑分区才能够构造文件系统。

GPT是对MBR的改进。

**fdisk命令**

分区管理命令，最多管理15个分区

1、查看磁盘的分区信息：
`fdisk -l [-u] [device...]`：列出指定磁盘设备上的分区情况；

我们通过 `fdisk -l DEVICE`查看分区时，可以看到后面System显示出分区的类别。可以在进入磁盘管理界面时查看支持的分区类型。

2、管理分区
`fdisk  device`

fdisk提供了一个交互式接口来管理分区，它有许多子命令，分别用于不同的管理功能；所有的操作均在内存中完成，没有直接同步到磁盘；直到使用w命令保存至磁盘上；

常用命令：

```
	n：创建新分区
	d：删除已有分区
	t：修改分区类型
	l：查看所有已经ID
	w：保存并退出
	q：不保存并退出
	m：查看帮助信息
	p：显示现有分区信息
```



> 注意：在已经分区并且已经挂载其中某个分区的磁盘设备上创建的新分区，内核可能在创建完成后无法直接识别；

查看：cat  /proc/partitions
通知内核强制重读磁盘分区表：
	CentOS 5：partprobe [device]
	CentOS 6,7：partx, kpartx
		partx -a [device]
		kpartx -af [device]

​		partprobe [device]		
其他的分区创建工具：parted, sfdisk；

示例：

```shell
[hyp@localhost ~]$ sudo fdisk -l /dev/sda

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0001a3f1

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1            2048        6143        2048   83  Linux
/dev/sda2   *        6144      415743      204800   83  Linux
/dev/sda3          415744    59152383    29368320   8e  Linux LVM
[hyp@localhost ~]$ sudo fdisk  /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：n
Partition type:
   p   primary (3 primary, 0 extended, 1 free)
   e   extended
Select (default e):
Using default response e
已选择分区 4
起始 扇区 (59152384-83886079，默认为 59152384)：
将使用默认值 59152384
Last 扇区, +扇区 or +size{K,M,G} (59152384-83886079，默认为 83886079)：+2G
分区 4 已设置为 Extended 类型，大小设为 2 GiB

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。
[hyp@localhost ~]$ sudo partprobe /dev/sda
[hyp@localhost ~]$ sudo fdisk -l /dev/sda  #多出一个扩展分区

磁盘 /dev/sda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0001a3f1

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1            2048        6143        2048   83  Linux
/dev/sda2   *        6144      415743      204800   83  Linux
/dev/sda3          415744    59152383    29368320   8e  Linux LVM
/dev/sda4        59152384    63346687     2097152    5  Extended

```

**df命令**

df查看当前磁盘使用情况。

用法：df [选项]... [文件]...

选项 ： -h 使用恰当的单位显示容量。

示例:

```shell
[hyp@localhost ~]$ df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   18G  2.3G   16G   13% /
devtmpfs                 475M     0  475M    0% /dev
tmpfs                    487M     0  487M    0% /dev/shm
tmpfs                    487M  7.7M  479M    2% /run
tmpfs                    487M     0  487M    0% /sys/fs/cgroup
/dev/sda2                197M  118M   80M   60% /boot
/dev/mapper/centos-var   5.0G  384M  4.7G    8% /var
/dev/mapper/centos-home  3.0G  316M  2.7G   11% /home
tmpfs                     98M     0   98M    0% /run/user/1000
[hyp@localhost ~]$ df -l
文件系统                   1K-块    已用     可用 已用% 挂载点
/dev/mapper/centos-root 18864128 2408776 16455352   13% /
devtmpfs                  485968       0   485968    0% /dev
tmpfs                     497948       0   497948    0% /dev/shm
tmpfs                     497948    7844   490104    2% /run
tmpfs                     497948       0   497948    0% /sys/fs/cgroup
/dev/sda2                 201380  120292    81088   60% /boot
/dev/mapper/centos-var   5232640  392176  4840464    8% /var
/dev/mapper/centos-home  3135488  323368  2812120   11% /home
tmpfs                      99592       0    99592    0% /run/user/1000
```

**du命令**

查看文件在磁盘上占用的空间大小，

用法：du [选项] [文件]

选项：

-h 输出文件系统分区使用的情况，例如：10KB，10MB，10GB等

  -s 显示文件或整个目录的大小，默认单位是KB

-a或-all  显示目录中个别文件的大小。   

-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   

--max-depth 显示目录深度

示例：

```shell
[hyp@localhost ~]$ du --max-depth=1 /usr/local/
76      /usr/local/bin
4       /usr/local/etc
0       /usr/local/games
0       /usr/local/include
0       /usr/local/lib
0       /usr/local/lib64
0       /usr/local/libexec
0       /usr/local/sbin
4       /usr/local/share
0       /usr/local/src
165320  /usr/local/Typora
165404  /usr/local/
[hyp@localhost ~]$ du --max-depth=1 -h /usr/local/
76K     /usr/local/bin
4.0K    /usr/local/etc
0       /usr/local/games
0       /usr/local/include
0       /usr/local/lib
0       /usr/local/lib64
0       /usr/local/libexec
0       /usr/local/sbin
4.0K    /usr/local/share
0       /usr/local/src
162M    /usr/local/Typora
162M    /usr/local/

```



### 文件系统管理

那么我们只是创建一个分区就可以用了吗？当然不是，我们需要在分区之上构建一个文件系统。磁盘格式化：低级格式化（分区之前进行，划分磁道）、高级格式化（分区之后对分区进行，创建文件系统）。linux使用VFS（Virtual File System）机制，可以使linux对不同的文件系统操作相同。

查看Linux支持的文件系统：`ls -l /lib/modules/$(uname -r)/kernel/fs`

查看Linux支持的文件系统(已载入到内存中)：`cat /proc/filesystems`

文件系统分类：

```
Linux的文件系统: ext2(无日志功能), ext3, ext4, xfs, reiserfs, btrfs
光盘：iso9660
网络文件系统：nfs, cifs
集群文件系统：gfs2, ocfs2
内核级分布式文件系统：ceph
windows的文件系统：vfat, ntfs
伪文件系统：proc, sysfs, tmpfs, hugepagefs
Unix的文件系统：UFS， FFS， JFS
交换文件系统：swap
用户空间的分布式文件系统：mogilefs, moosefs, glusterfs
```

常用文件系统工具：

```
创建文件系统的工具
	mkfs
不同类型的文件系统mkfs.ext2, mkfs.ext3, mkfs.ext4, mkfs.xfs, mkfs.vfat, ...
	mke2fs ext系列文件系统专用
	mkswap 创建交换分区
检测及修复文件系统的工具
	fsck
		fsck.ext2, fsck.ext3, ...
	e2fsck
查看其属性的工具（ext文件系统）
	dumpe2fs
	tune2fs
	blkid 块设备UUID查询
	e2label ext文件系统label查看修改
调整文件系统特性：
	tune2fs 修改ext文件系统属性
```

**blkid**命令：

```
	blkid device
	blkid  -L LABEL：根据LABEL定位设备
	blkid  -U  UUID：根据UUID定位设备
```



#### ext系列文件系统

全称Linux extended file system, extfs，即Linux扩展文件系统，Ext2就代表第二代文件扩展系统，Ext3/Ext4以此类推，它们都是Ext2的升级版，只不过为了快速恢复文件系统，减少一致性检查的时间，增加了日志功能，所以Ext2被称为**索引式文件系统**，而Ext3/Ext4被称为**日志式文件系统**。

创建命令：

```
mkfs.ext2, mkfs.ext3, mkfs.ext4
mkfs -t ext2 = mkfs.ext2
```

ext系列文件系统专用管理工具：mke2fs
	语法：`mke2fs [OPTIONS]  device`

选项：

			-t {ext2|ext3|ext4}：指明要创建的文件系统类型
				mkfs.ext4 = mkfs -t ext4 = mke2fs -t ext4
			-b {1024|2048|4096}：指明文件系统的块大小；
			-L LABEL：指明卷标；
			-j：创建有日志功能的文件系统ext3；
				mke2fs -j = mke2fs -t ext3 = mkfs -t ext3 = mkfs.ext3
			-i #：bytes-per-inode，指明inode与字节的比率；即每多少字节创建一个Indode; 
			-N #：直接指明要给此文件系统创建的inode的数量；
			-m #：指定预留的空间，百分比；
			-O [^]FEATURE：以指定的特性创建目标文件系统； 

`e2label`命令：ext文件系统卷标的查看与设定

```
查看：e2label device
设定：e2label device LABEL
```

`tune2fs`命令：查看或修改ext系列文件系统的某些属性 
注意：块大小创建后不可修改；

语法：`tune2fs [OPTIONS] device`
	-l：查看超级块的内容；

修改指定文件系统的属性：

```
-j：ext2 --> ext3；
-L LABEL：修改卷标；
-m #：调整预留空间百分比；
-O [^]FEATHER：开启或关闭某种特性；

-o [^]mount_options：开启或关闭某种默认挂载选项
acl
^acl
```

`dumpe2fs`命令：显示ext系列文件系统的属性信息

```
dumpe2fs  [-h] device
```

用于实现文件系统检测的工具

因进程意外中止或系统崩溃等 原因导致定稿操作非正常终止时，可能会造成文件损坏；此时，应该检测并修复文件系统； 建议，离线进行； 

ext系列文件系统的专用工具：
	`e2fsck` : check a Linux ext2/ext3/ext4 file system
		`e2fsck [OPTIONS]  device`
选项：

```
-y：对所有问题自动回答为yes; 
-f：即使文件系统处于clean状态，也要强制进行检测；
```

文件系统修复工具（不限于ext）			
**fsck**：check and repair a Linux file system

```
-t fstype：指明文件系统类型；
fsck -t ext4 = fsck.ext4
-a：无须交互而自动修复所有错误；
-r：交互式修复；
```

示例：

```shell
#从刚刚创建的扩展分区上创建一个逻辑分区sda5，同步后创建文件系统
[hyp@localhost ~]$ sudo mke2fs -t ext4 -b 1024 -L ping /dev/sda5
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=ping
OS type: Linux
块大小=1024 (log=0)
分块大小=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
25688 inodes, 102400 blocks
5120 blocks (5.00%) reserved for the super user
第一个数据块=1
Maximum filesystem blocks=33685504
13 block groups
8192 blocks per group, 8192 fragments per group
1976 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: 完成
正在写入inode表: 完成
Creating journal (4096 blocks): 完成
Writing superblocks and filesystem accounting information: 完成
[hyp@localhost ~]$ sudo e2label /dev/sda5 #卷积标识为ping
ping
[hyp@localhost ~]$ sudo e2label /dev/sda5 pingxin
[hyp@localhost ~]$ sudo e2label /dev/sda5
pingxin
[hyp@localhost ~]$ sudo tune2fs -l /dev/sda5 #显示文件系统超级块信息
[hyp@localhost ~]$ sudo dumpe2fs /dev/sda5 #显示文件系统属性信息
[hyp@localhost ~]$ sudo e2fsck /dev/sda5 #检查是否出错
e2fsck 1.42.9 (28-Dec-2013)
pingxin: clean, 11/25688 files, 8896/102400 blocks
[hyp@localhost ~]$ sudo fsck -t ext4 /dev/sda5 #调用e2fsck
fsck，来自 util-linux 2.23.2
e2fsck 1.42.9 (28-Dec-2013)
pingxin: clean, 11/25688 files, 8896/102400 blocks

```

#### XFS文件系统

CentOS7上默认使用的文件系统是XFS，XFS一种高性能的日志文件系统，最早于1993年，由Silicon Graphics为他们的IRIX操作系统而开发，是IRIX 5.3版的默认文件系统。2000年5月，Silicon Graphics以GNU通用公共许可证发布这套系统的源代码，之后被移植到Linux 内核上。XFS 特别擅长处理大文件，同时提供平滑的数据传输。

使用的程序包为`xfsprogs`，CentOS6及之前版本的可以安装后进行使用。

创建和检查的步骤和命令与上面ext文件系统相似，使用创建命令 `mkfs.xfs`，检测命令 `xfs_repair`，还有一些以xfs开头的命令。

示例：

```shell
#新建逻辑分区sda6，同步后构建xfs文件系统
[hyp@localhost ~]$ sudo mkfs.xfs   -b size=2048   -L ping /dev/sda6
meta-data=/dev/sda6              isize=512    agcount=4, agsize=12800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=2048   blocks=51200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=2048   blocks=1445, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[hyp@localhost ~]$ sudo blkid /dev/sda6
/dev/sda6: LABEL="ping" UUID="ff544866-aa96-46c5-ac4b-cdbd254fd6df" TYPE="xfs"
[hyp@localhost ~]$ sudo xfs_repair /dev/sda6
Phase 1 - find and verify superblock...
Phase 2 - using internal log
        - zero log...
        - scan filesystem freespace and inode maps...
        - found root inode chunk
Phase 3 - for each AG...
        - scan and clear agi unlinked lists...
        - process known inodes and perform inode discovery...
        - agno = 0
        - agno = 1
        - agno = 2
        - agno = 3
        - process newly discovered inodes...
Phase 4 - check for duplicate blocks...
        - setting up duplicate extent list...
        - check for inodes claiming duplicate blocks...
        - agno = 0
        - agno = 1
        - agno = 2
        - agno = 3
Phase 5 - rebuild AG headers and trees...
        - reset superblock...
Phase 6 - check inode connectivity...
        - resetting contents of realtime bitmap and summary inodes
        - traversing filesystem ...
        - traversal finished ...
        - moving disconnected inodes to lost+found ...
Phase 7 - verify and correct link counts...
done

```

#### swap文件系统

Linux上的交换分区必须使用独立的文件系统；且文件系统的System ID必须为82，即设置分区类型为82。

创建swap设备：mkswap命令
		`mkswap [OPTIONS]  device`
			-L LABEL：指明卷标
			-f：强制
启用：swapon
	`swapon  [OPTION]  [DEVICE]`
		-a：定义在`/etc/fstab`文件中的所有swap设备；	
禁用：swapoff
	`swapoff DEVICE`

####  文件系统挂载

首先对于一个设备文件，它下面有至少一个分区文件，而分区文件只有在构造完文件系统后才能够被使用，文件系统只有在不挂载的时候才能进行修复工作。根文件系统这外的其它文件系统要想能够被访问，都必须通过“关联”至根文件系统上的某个目录来实现，此关联操作即为“挂载”；此目录即为“挂载点”。

挂载点：mount_point，用于作为另一个文件系统的访问入口

```
(1) 事先存在；
(2) 应该使用未被或不会被其它进程使用到的目录；
(3) 挂载点下原有的文件将会被隐藏；
```

一般来说，文件系统的挂载命令是`mount`，卸载命令是 `umont`。

**munot**

语法：`mount  [-nrw]  [-t vfstype]  [-o options]  device  dir`

命令选项：

```
-r：readonly，只读挂载； 
-w：read and write, 读写挂载； 
-n：默认情况下，设备挂载或卸载的操作会同步更新至/etc/mtab文件中；-n用于禁止此特性；
	

-t vfstype：指明要挂载的设备上的文件系统的类型；多数情况下可省略，此时mount会通过blkid来判断要挂载的设备的文件系统类型；

-L LABEL：挂载时以卷标的方式指明设备；
	mount -L LABEL dir
	
-U UUID：挂载时以UUID的方式指明设备；
	mount -U UUID dir

-o options：挂载选项
	sync/async：同步/异步操作；
	atime/noatime：文件或目录在被访问时是否更新其访问时间戳；
	diratime/nodiratime：目录在被访问时是否更新其访问时间戳；
	remount：重新挂载； 
	acl：支持使用facl功能；

# mount -o acl  device dir 

# tune2fs  -o  acl  device 

ro：只读 
rw：读写 
dev/nodev：此设备上是否允许创建设备文件；
exec/noexec：是否允许运行此设备上的程序文件；
auto/noauto：
user/nouser：是否允许普通用户挂载此文件系统；
suid/nosuid：是否允许程序文件上的suid和sgid特殊权限生效；					

defaults：Use default options: rw, suid, dev, exec, auto, nouser, async, and relatime.
```

一个使用技巧：
可以实现将目录绑定至另一个目录上，作为其临时访问入口；

```
	mount --bind  源目录  目标目录
```

​	
查看当前系统所有已挂载的设备：

```
mount 
cat  /etc/mtab
cat  /proc/mounts
```

挂载光盘：

```
mount  -r  /dev/cdrom  mount_point
```

光盘设备文件：/dev/cdrom, /dev/dvd

挂载U盘：
事先识别U盘的设备文件；

挂载本地的回环设备：https://blog.csdn.net/baimafujinji/article/details/78810042

```
mount  -o  loop  /PATH/TO/SOME_LOOP_FILE   MOUNT_POINT 
```

**umount**

语法：`umount  device|dir`

注意：正在被进程访问到的挂载点无法被卸载；
查看被哪个或哪些进程所战用：

```
# lsof  MOUNT_POINT
# fuser -v  MOUNT_POINT
```

终止所有正在访问某挂载点的进程：

```
# fuser  -km  MOUNT_POINT
```

**/etc/fstab文件**

设定除根文件系统以外的其它文件系统能够开机时自动挂载：/etc/fstab文件

每行定义一个要挂载的文件系统及相关属性：

```
6个字段：
(1) 要挂载的设备：
		设备文件；
		LABEL
		UUID
		伪文件系统：如sysfs, proc, tmpfs等
	(2) 挂载点 
		swap类型的设备的挂载点为swap；
	(3) 文件系统类型；
	(4) 挂载选项
		defaults：使用默认挂载选项；
		如果要同时指明多个挂载选项，彼此间以事情分隔；
			defaults,acl,noatime,noexec
	(5) 转储频率
		0：从不备份；
		1：每天备份；
		2：每隔一天备份；
	(6) 自检次序
		0：不自检；
		1：首先自检，通常只能是根文件系统可用1；
		2：次级自检
		...
```

设置完成后使用该命令查看设置是否成功：

`mount  -a`：可自动挂载定义在此文件中的所支持自动挂载的设备；

### LVM管理

 LVM是 Logical Volume Manager(逻辑卷管理)的简写，可以理解为我们在不停机的状态下对各个分区大小进行动态调整，且调整过程中不会影响我们的数据。

​    LVM： Logical Volume Manager，Version：2，也是我们使用时的LVM2工具。

参考：[LVM基础](https://my.oschina.net/comics/blog/2239930)

### RAID

磁盘阵列（Redundant Arrays of Independent Drives，RAID），有“独立磁盘构成的具有冗余能力的阵列”之意。

磁盘阵列是由很多块独立的磁盘，组合成一个容量巨大的磁盘组，利用个别磁盘提供数据所产生加成效果提升整个磁盘系统效能。利用这项技术，将数据切割成许多区段，分别存放在各个硬盘上。

磁盘阵列还能利用同位检查（Parity Check）的观念，在数组中任意一个硬盘故障时，仍可读出数据，在数据重构时，将数据经计算后重新置入新硬盘中。

参看：[磁盘阵列](https://baike.baidu.com/item/%E7%A3%81%E7%9B%98%E9%98%B5%E5%88%97/1149823?fromtitle=RAID&fromid=33858&fr=aladdin#2)

​	[RAID分类](https://blog.51cto.com/13706064/2138912)

实现方法：硬件实现方式、软件实现方式。CentOS 6上的软件RAID的实现：结合内核中的md(multi devices)。

**mdadm**命令：

语法：`mdadm [mode] <raiddevice> [options] <component-devices>` 

支持的RAID级别：LINEAR, RAID0, RAID1, RAID4, RAID5, RAID6, RAID10;

模式：
	创建：-C
	装配: -A
	监控: -F
	管理：-f, -r, -a

<raiddevice>: /dev/md#
<component-devices>: 任意块设备
添加到/etc/fstab文件时使用UUID，名称会在开机时改变

-C: 创建模式
	-n #: 使用#个块设备来创建此RAID；
	-l #：指明要创建的RAID的级别；
	-a {yes|no}：自动创建目标RAID设备的设备文件；
	-c CHUNK_SIZE: 指明块大小；
	-x #: 指明空闲盘的个数；

	例如：创建一个10G可用空间的RAID5；给出n+x个
	sudo mdadm -C /dev/md0 -a yes -n 3 -l 5 -x 1  /dev/sda{5,6,7,8}
-D：显示raid的详细信息；
	mdadm -D /dev/md#

管理模式：

```
   -f: 标记指定磁盘为损坏；mdadm -f /dev/md# /dev/sda#
	-a: 添加磁盘		mdadm -a /dev/md# /dev/sda#
	-r: 移除磁盘		mdadm -r /dev/md# /dev/sda#
```

观察md的状态：

```
	cat /proc/mdstat
```

停止md设备：

```
	mdadm -S /dev/md#
```

参考：[使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/chapter-07.html)

### 存储快照

在 Linux 操作系统上，有多种实现存储快照的方案，如使用 LVM、ZFS 存储池、Btrfs 文件系统等。

Btrfs 文件系统具有透明压缩、软 RAID、快照等诸多实用功能，而且配置和管理起来比其他文件系统都要简单不少。

所以，Btrfs 目前是我心目中最完美的仓库盘专用文件系统！

> **注意**
>
> Btrfs 的 I/O 性能相比其他文件系统还是要逊色不少的。如果磁盘需要大量且频繁的 I/O 操作，建议选择其他文件系统。

snapper 是一款快照管理实用工具，支持多种文件系统，当然也包括 Btrfs。

相比 Btrfs 自带的快照管理工具，snapper 可以更方便、直观地对快照进行管理、比较，而且还有定时创建快照的功能。