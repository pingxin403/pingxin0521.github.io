---
title: SVN入门
date: 2019-03-19 09:18:59
tags:
 - 工具
 - SVN
 - 版本控制
categories:
 - 工具
 - 版本控制
 - SVN
---

### 前言

SVN是Subversion的简称，目前是Apache项目底下的一个开放源代码的版本控制系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS迁移到Subversion。说得简单一点SVN就是用于多个人共同开发同一个项目，共用资源的目的。

<!--more-->

SVN 官网：<https://subversion.apache.org/>

Github SVN 源码：<https://github.com/apache/subversion>

### 简介

Subversion(SVN) 是一个开源的版本控制系統, 也就是说 Subversion 管理着随时间改变的数据。 这些数据放置在一个中央资料档案库(repository) 中。 这个档案库很像一个普通的文件服务器, 不过它会记住每一次文件的变动。 这样你就可以把档案恢复到旧的版本, 或是浏览文件的变动历史。

#### 概念：

- **repository（源代码库）:** 源代码统一存放的地方
- **Checkout（提取）:** 当你手上没有源代码的时候，你需要从repository checkout一份
- **Commit（提交）:** 当你已经修改了代码，你就需要Commit到repository
- **Update (更新):** 当你已经Checkout了一份源代码， Update一下你就可以和Repository上的源代码同步，你手上的代码就会有最新的变更

日常开发过程其实就是这样的（假设你已经Checkout并且已经工作了几天）：Update(获得最新的代码) -->作出自己的修改并调试成功 --> Commit(大家就可以看到你的修改了) 。

如果两个程序员同时修改了同一个文件呢, SVN可以合并这两个程序员的改动，实际上SVN管理源代码是以行为单位的，就是说两个程序员只要不是修改了同一行程序，SVN都会自动合并两种修改。如果是同一行，SVN会提示文件Confict, 冲突，需要手动确认。

#### 主要功能

- （1）目录版本控制

  CVS 只能跟踪单个文件的历史, 不过 Subversion 实作了一个 "虚拟" 的版本控管文件系统, 能够依时间跟踪整个目录的变动。 目录和文件都能进行版本控制。

- （2）真实的版本历史

  自从CVS限制了文件的版本记录，CVS并不支持那些可能发生在文件上，但会影响所在目录内容的操作，如同复制和重命名。除此之外，在CVS里你不能用拥有同样名字但是没有继承老版本历史或者根本没有关系的文件替换一个已经纳入系统的文件。在Subversion中，你可以增加（add）、删除（delete）、复制（copy）和重命名（rename），无论是文件还是目录。所有的新加的文件都从一个新的、干净的版本开始。

- （3）自动提交

  一个提交动作，不是全部更新到了档案库中，就是不完全更新。这允许开发人员以逻辑区间建立并提交变动，以防止当部分提交成功时出现的问题。

- （4）纳入版本控管的元数据

  每一个文件与目录都附有一組属性关键字并和属性值相关联。你可以创建, 并儲存任何你想要的Key/Value对。 属性是随着时间来作版本控管的,就像文件內容一样。

- （5）选择不同的网络层

  Subversion 有抽象的档案库存取概念, 可以让人很容易地实作新的网络机制。 Subversion 可以作为一个扩展模块嵌入到Apache HTTP 服务器中。这个为Subversion提供了非常先进的稳定性和协同工作能力，除此之外还提供了许多重要功能: 举例来说, 有身份认证, 授权, 在线压缩, 以及文件库浏览等等。还有一个轻量级的独立Subversion服务器， 使用的是自定义的通信协议, 可以很容易地通过 ssh 以 tunnel 方式使用。

- （6）一致的数据处理方式

  Subversion 使用二进制差异算法来异表示文件的差异, 它对文字(人类可理解的)与二进制文件(人类无法理解的) 两类的文件都一视同仁。 这两类的文件都同样地以压缩形式储存在档案库中, 而且文件差异是以两个方向在网络上传输的。

- （7）有效的分支(branch)与标签(tag)

  在分支与标签上的消耗并不必一定要与项目大小成正比。 Subversion 建立分支与标签的方法, 就只是复制该项目, 使用的方法就类似于硬连接（hard-link）。 所以这些操作只会花费很小, 而且是固定的时间。

- （8）Hackability

  Subversion没有任何的历史包袱; 它主要是一群共用的 C 程序库, 具有定义完善的API。这使得 Subversion 便于维护, 并且可被其它应用程序与程序语言使用。

#### 优于CVS之处

1、原子提交。一次提交不管是单个还是多个文件，都是作为一个整体提交的。在这当中发生的意外例如传输中断，不会引起数据库的不完整和数据损坏。

2、重命名、复制、删除文件等动作都保存在版本历史记录当中。

3、对于二进制文件，使用了节省空间的保存方法。（简单的理解，就是只保存和上一版本不同之处）

4、目录也有版本历史。整个目录树可以被移动或者复制，操作很简单，而且能够保留全部版本记录。

5、分支的开销非常小。

6、优化过的数据库访问，使得一些操作不必访问数据库就可以做到。这样减少了很多不必要的和数据库主机之间的网络流量。



SVN适合对访问控制、权限分配和代码安全性等要求比较高的项目。

### 安装

不建议源码安装，依赖过多(apr--->apr-util--->scons--->openssl--->serf--->svn)。

ubuntu 18.10 上进行源码编译。

```(xml)
cd ~
sudo mkdir /soft
sudo chown erge /soft
sudo aptitude install gcc g++ \
libxml2 libxml2-dev \
libssl-dev curl libcurl4-openssl-dev \
libpcre3 libpcre3-dev \
libgd-dev nghttp2 zlib1g-dev libutf8proc-dev
wget http://mirrors.tuna.tsinghua.edu.cn/apache/subversion/subversion-1.11.1.tar.gz
tar zxvf subversion-*.tar.gz
cd subversion-*/
./get-deps.sh
#如果没有安装apr和apr-util
cd apr
./configure --prefix=/usr/local/apr
make -j 4 && make install
cd ../apr-util
./configure --prefix=/usr/local/apr-util
make -j 4 && make install
cd ..
./configure --prefix=/soft/svn --with-apr=/soft/apr --with-apr-util=/soft/apr-util --with-lz4=internal
make
make install
```

之后配置环境变量，可以通过`svnversion`查看。


#### CentOS上安装

```shell
$ yum install subversion
$ yum install mod_dav_svn
```

其他操作系统上安装请参考[官网](http://subversion.apache.org/packages.html)。

### SVN命令

**1、**将文件checkout到本地目录

```shell
svn checkout path（path是服务器上的目录）
例如：svn checkout svn://192.168.1.1/pro/domain
简写：svn co
```

**2、**往版本库中添加新的文件

```shell
svn add file
例如：svn add test.php(添加test.php)
svn add *.php(添加当前目录下所有的php文件)
```

**3、**将改动的文件提交到版本库

```shell
 svn commit -m "LogMessage" [-N] [--no-unlock] PATH![img](http://hiphotos.baidu.com/li9chuan/pic/item/b74a713dbbc099a53c6d9730.jpg)(如果选择了保持锁，就使用--no-unlock开关)
例如：svn commit -m "add test file for my test" test.php
简写：svn ci
```

**4、**加锁/解锁

```shell
svn lock -m "LockMessage" [--force] PATH![img](http://hiphotos.baidu.com/li9chuan/pic/item/992779eea23a95b0b3fb9530.jpg)
例如：svn lock -m "lock test file" test.php
svn unlock PATH
```

**5、**更新到某个版本

```shell
 svn update -r m path
 例如：
 svn update如果后面没有目录，默认将当前目录以及子目录下的所有文件都更新到最新版本。
 svn update -r 200 test.php(将版本库中的文件test.php还原到版本200)
 svn update test.php(更新，于版本库同步。如果在提交的时候提示过期的话，是因为冲突，
 需要先update，修改文件，然后清除svn resolved，最后再提交commit)
 简写：svn up
```

**6、**查看文件或者目录状态

```
1）svn status path![img](http://hiphotos.baidu.com/li9chuan/pic/item/82dd75fb13ac835d6c22eb30.jpg)（目录下的文件和子目录的状态，正常状态不显示）
 【?：不在svn的控制中；M：内容被修改；C：发生冲突；A：预定加入到版本库；K：被锁定】
2）svn status -v path![img](http://hiphotos.baidu.com/li9chuan/pic/item/0d4b6316ce758f0121a4e930.jpg)(显示文件和子目录状态)
第一列保持相同，第二列显示工作版本号，第三和第四列显示最后一次修改的版本号和修改人。
注：svn status、svn diff和 svn revert这三条命令在没有网络的情况下也可以执行的，原因是svn在本地的.svn中保留了本地版本的原始拷贝。
简写：svn st

输出的前七栏各占一个字符宽度:
    第一栏: 表示一个项目是增加、删除，还是修改
      “ ” 无修改
      “A” 增加
      “C” 冲突
      “D” 删除
      “I” 忽略
      “M” 改变
      “R” 替换
      “X” 未纳入版本控制的目录，被外部引用的目录所创建
      “?” 未纳入版本控制
      “!” 该项目已遗失(被非 svn 命令删除)或不完整
      “~” 版本控制下的项目与其它类型的项目重名
    第二栏: 显示目录或文件的属性状态
      “ ” 无修改
      “C” 冲突
      “M” 改变
    第三栏: 工作副本目录是否被锁定
      “ ” 未锁定
      “L” 锁定
    第四栏: 已调度的提交是否包含副本历史
      “ ” 没有历史
      “+” 包含历史
    第五栏: 该条目相对其父目录是否已切换，或者是外部引用的文件
      “ ” 正常
      “S” 已切换
      “X” 被外部引用创建的文件
    第六栏: 版本库锁定标记
      (没有 -u)
      “ ” 没有锁定标记
      “K” 存在锁定标记
      (使用 -u)
      “ ” 没有在版本库中锁定，没有锁定标记
      “K” 在版本库中被锁定，存在锁定标记
      “O” 在版本库中被锁定，锁定标记在一些其他工作副本中
      “T” 在版本库中被锁定，存在锁定标记但已被窃取
      “B” 没有在版本库中被锁定，存在锁定标记但已被破坏
    第七栏: 项目冲突标记
      “ ” 正常
      “C” 树冲突
    如果项目包含于树冲突之中，在项目状态行后会附加行，说明冲突的种类。

  是否过期的信息出现的位置是第九栏(与 -u 并用时):
      “*” 服务器上有更新版本
      “ ” 工作副本是最新版的

  剩余的栏位皆为变动宽度，并以空白隔开:
    工作版本号(使用 -u 或 -v 时；被复制时显示“-”)
    最后提交的版本与最后提交的作者(使用 -v 时)
    工作副本路径总是最后一栏，所以它可以包含空白字符。
```

**7、**删除文件

```
svn delete path -m "delete test fle"
例如：svn delete svn://192.168.1.1/pro/domain/test.php -m "delete test file"
或者直接svn delete test.php 然后再svn ci -m 'delete test file‘，推荐使用这种
简写：svn (del, remove, rm)
```

**8、**查看日志

```
svn log path
例如：svn log test.php 显示这个文件的所有修改记录，及其版本号的变化
```

**9、**查看文件详细信息

```
 svn info path
   例如：svn info test.php
```

**10、**比较差异

```
svn diff path(将修改的文件与基础版本比较)
例如：svn diff test.php
svn diff -r m:n path(对版本m和版本n比较差异)
 例如：svn diff -r 200:201 test.php
 简写：svn di
```

**11、**将两个版本之间的差异合并到当前文件

```shell
svn merge -r m:n path
例如：svn merge -r 200:205 test.php（将版本200与205之间的差异合并到当前文件，但是一般都会产生冲突，需要处理一下）
```

**12、**SVN 帮助

```shell
svn help
svn help ci
```

\------------------------------------------------------------------------------

以上是常用命令，下面写几个不经常用的

\------------------------------------------------------------------------------

**13、**版本库下的文件和目录列表

```shell
svn list path #显示path目录下的所有属于版本库的文件和目录
简写：svn ls
```

**14、**创建纳入版本控制下的新目录

svn mkdir: 创建纳入版本控制下的新目录。
用法: 1、mkdir PATH...
         2、mkdir URL...
创建版本控制的目录。
1、每一个以工作副本 PATH 指定的目录，都会创建在本地端，并且加入新增
     调度，以待下一次的提交。
2、每个以URL指定的目录，都会透过立即提交于仓库中创建。
在这两个情况下，所有的中间目录都必须事先存在。



**15、**恢复本地修改

**svn revert**: 恢复原始未改变的工作副本文件 (恢复大部份的本地修改)。revert:
用法: revert PATH...
注意: 本子命令不会存取网络，并且会解除冲突的状况。但是它不会恢复
        被删除的目录



**16、**代码库URL变更

**svn switch (sw):** 更新工作副本至不同的URL。
用法: 1、switch URL [PATH]
        2、switch --relocate FROM TO [PATH...]

1、更新你的工作副本，映射到一个新的URL，其行为跟“svn update”很像，也会将
     服务器上文件与本地文件合并。这是将工作副本对应到同一仓库中某个分支或者标记的
     方法。
2、改写工作副本的URL元数据，以反映单纯的URL上的改变。当仓库的根URL变动
    (比如方案名或是主机名称变动)，但是工作副本仍旧对映到同一仓库的同一目录时使用
    这个命令更新工作副本与仓库的对应关系。



**17、**解决冲突

**svn resolved:** 移除工作副本的目录或文件的“冲突”状态。
用法: resolved PATH...
注意: 本子命令不会依语法来解决冲突或是移除冲突标记；它只是移除冲突的
        相关文件，然后让 PATH 可以再次提交。



**18、**输出指定文件或URL的内容。

**svn** **cat** 目标[@版本]...如果指定了版本，将从指定的版本开始查找。

```
svn cat -r PREV filename > filename
```

(PREV 是上一版本,也可以写具体版本号,这样输出结果是可以提交的)



参考：[Linux SVN 命令详解](https://www.cnblogs.com/feifei-cyj/p/7904947.html)



### 生命周期

#### 创建版本库

版本库相当于一个集中的空间，用于存放开发者所有的工作成果。版本库不仅能存放文件，还包括了每次修改的历史，即每个文件的变动历史。

Create 操作是用来创建一个新的版本库。大多数情况下这个操作只会执行一次。当你创建一个新的版本库的时候，你的版本控制系统会让你提供一些信息来标识版本库，例如创建的位置和版本库的名字。

#### 检出

Checkout 操作是用来从版本库创建一个工作副本。工作副本是开发者私人的工作空间，可以进行内容的修改，然后提交到版本库中。

#### 更新

顾名思义，update 操作是用来更新版本库的。这个操作将工作副本与版本库进行同步。由于版本库是由整个团队共用的，当其他人提交了他们的改动之后，你的工作副本就会过期。

让我们假设 Tom 和 Jerry 是一个项目的两个开发者。他们同时从版本库中检出了最新的版本并开始工作。此时，工作副本是与版本库完全同步的。然后，Jerry 很高效的完成了他的工作并提交了更改到版本库中。

此时 Tom 的工作副本就过期了。更新操作将会从版本库中拉取 Jerry 的最新改动并将 Tom 的工作副本进行更新。

#### 执行变更

当检出之后，你就可以做很多操作来执行变更。编辑是最常用的操作。你可以编辑已存在的文件来，例如进行文件的添加/删除操作。

你可以添加文件/目录。但是这些添加的文件目录不会立刻成为版本库的一部分，而是被添加进待变更列表中，直到执行了 commit 操作后才会成为版本库的一部分。

同样地你可以删除文件/目录。删除操作立刻将文件从工作副本中删除掉，但该文件的实际删除只是被添加到了待变更列表中，直到执行了 commit 操作后才会真正删除。

Rename 操作可以更改文件/目录的名字。"移动"操作用来将文件/目录从一处移动到版本库中的另一处。

#### 复查变化

当你检出工作副本或者更新工作副本后，你的工作副本就跟版本库完全同步了。但是当你对工作副本进行一些修改之后，你的工作副本会比版本库要新。在 commit 操作之前复查下你的修改是一个很好的习惯。

Status 操作列出了工作副本中所进行的变动。正如我们之前提到的，你对工作副本的任何改动都会成为待变更列表的一部分。Status 操作就是用来查看这个待变更列表。

Status 操作只是提供了一个变动列表，但并不提供变动的详细信息。你可以用 diff 操作来查看这些变动的详细信息。

#### 修复错误

我们来假设你对工作副本做了许多修改，但是现在你不想要这些修改了，这时候 revert 操作将会帮助你。

Revert 操作重置了对工作副本的修改。它可以重置一个或多个文件/目录。当然它也可以重置整个工作副本。在这种情况下，revert 操作将会销毁待变更列表并将工作副本恢复到原始状态。

#### 解决冲突

合并的时候可能会发生冲突。Merge 操作会自动处理可以安全合并的东西。其它的会被当做冲突。例如，"hello.c" 文件在一个分支上被修改，在另一个分支上被删除了。这种情况就需要人为处理。Resolve 操作就是用来帮助用户找出冲突并告诉版本库如何处理这些冲突。

#### 提交更改

Commit 操作是用来将更改从工作副本到版本库。这个操作会修改版本库的内容，其它开发者可以通过更新他们的工作副本来查看这些修改。

在提交之前，你必须将文件/目录添加到待变更列表中。列表中记录了将会被提交的改动。当提交的时候，我们通常会提供一个注释来说明为什么会进行这些改动。这个注释也会成为版本库历史记录的一部分。Commit 是一个原子操作，也就是说要么完全提交成功，要么失败回滚。用户不会看到成功提交一半的情况。



### SVN启动模式

首先,在服务端进行SVN版本库的相关配置

手动新建版本库目录

```shell
[hyp@localhost ~]$ sudo mkdir /opt/svn
[hyp@localhost Auth]$ sudo chmod -R o+rw /opt/svn/
```

利用svn命令创建版本库

```shell
[hyp@localhost ~]$ sudo svnadmin create /opt/svn/Auth
```

使用命令svnserve启动服务

```shell
[hyp@localhost ~]$ sudo svnserve -d -r 目录 --listen-port 端口号
```

- **-r:** 配置方式决定了版本库访问方式。
- **--listen-port:** 指定SVN监听端口，不加此参数，SVN默认监听3690
- 由于-r 配置方式的不一样，SVN启动就可以有两种不同的访问方式

方式一：-r直接指定到版本库(称之为单库svnserve方式）

- ```
  [hyp@localhost ~]$ sudo svnserve -d -r /opt/svn/Auth
  ```

- 在这种情况下，一个svnserve只能为一个版本库工作。

- authz配置文件中对版本库权限的配置应这样写：

- ```
  [groups]
  admin=user1
  dev=user2
  [/]
  @admin=rw
  user2=r
  ```

- 使用类似这样的URL：svn://192.168.0.1/　即可访问runoob版本库

- 方式二：指定到版本库的上级目录(称之为多库svnserve方式)

- ```
  [hyp@localhost ~]$ sudo svnserve -d -r /opt/svn
  ```

- 这种情况，一个svnserve可以为多个版本库工作

- authz配置文件中对版本库权限的配置应这样写：

- ```
  [groups]
  admin=user1
  dev=user2
  [runoob:/]
  @admin=rw
  user2=r

  [runoob01:/]
  @admin=rw
  user2=r
  ```

- 如果此时你还用[/]，则表示所有库的根目录，同理，[/src]表示所有库的根目录下的src目录。

- 使用类似这样的URL：svn://192.168.0.1/Auth，即可访问Auth版本库。

- **查看SVN进程**

- ```shell
  # ps -ef|grep svn|grep -v grep
  root 12538 1 0 14:40 ? 00:00:00 svnserve -d -r /opt/svn/repositories
  ```

-  **检测SVN端口**

- ```shell
  # netstat -ln |grep 3690
  tcp 0 0 0.0.0.0:3690 0.0.0.0:* LISTEN
  ```

- **停止重启SVN：**

- ```shell
   killall svnserve #停止
   svnserve -d -r /opt/svn # 启动
  ```

### 创建版本库

使用svn命令创建资源库

```shell
[hyp@localhost ~]$ sudo svnadmin create /opt/svn/Auth
```

进入/opt/svn/Auth/conf目录 修改默认配置文件配置，包括svnserve.conf、passwd、authz 配置相关用户和权限。

1、svn服务配置文件svnserve.conf

svn服务配置文件为版本库目录中的文件conf/svnserve.conf。该文件仅由一个[general]配置段组成。

```
[general]
anon-access = none
auth-access = write
password-db = /home/svn/passwd
authz-db = /home/svn/authz
realm = tiku
```

- **anon-access:** 控制非鉴权用户访问版本库的权限，取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 缺省值：read
- **auth-access:** 控制鉴权用户访问版本库的权限。取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 缺省值：write
- **authz-db:** 指定权限配置文件名，通过该文件可以实现以路径为基础的访问控制。 除非指定绝对路径，否则文件位置为相对conf目录的相对路径。 缺省值：authz
- **password-db** 使用哪个文件作为账号文件，缺省值：passwd
- **realm:** 指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的 认证域相同，建议使用相同的用户名口令数据文件。 缺省值：一个UUID(Universal Unique IDentifier，全局唯一标示)。

2、用户名口令文件passwd

用户名口令文件由svnserve.conf的配置项password-db指定，缺省为conf目录中的passwd。该文件仅由一个[users]配置段组成。

[users]配置段的配置行格式如下：

```shell
#<用户名> = <口令>
[hyp@localhost ~]$ cat /opt/svn/passwd
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
# harry = harryssecret
# sally = sallyssecret
admin=admin
ping=123456

```

3、权限配置文件

权限配置文件由svnserve.conf的配置项authz-db指定，缺省为conf目录中的authz。该配置文件由一个[groups]配置段和若干个版本库路径权限段组成。

[groups]配置段中配置行格式如下：

```
<用户组> = <用户列表>
```

版本库路径权限段的段名格式如下：

```shell
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
g_admin = admin,ping

[/]
admin=rw  #对所有的仓库的主目录/下文件都可以读写
[Auth:/]
ping=rw #ping对Auth仓库的主目录下/下文件都可读写
*=r #所有用户都可对Auth仓库的主目录下/下文件都可读

```

本例是使用`svnserve -d -r /opt/svn` 以多库svnserve方式启动SVN，所以URL：`svn://192.168.0.1/Auth`

### 检出

上一章，我们创建了仓库Auth，多库方式运行SVN服务器：`svnserve -d -r /opt/svn`。

```shell
[hyp@localhost ~]$ svn checkout  svn://127.0.0.1/Auth --username=ping #输入密码
[hyp@localhost ~]$ ls -ld Auth/
drwxrwxr-x. 3 hyp hyp 18 3月  22 16:58 Auth/
```

你想查看更多关于版本库的信息，执行 info 命令。

### 提交

上一章，我们checkout仓库Auth，位于/home/hyp/Auth/下。

我们增加一个README文件。

```shell
[hyp@localhost Auth]$ cat README
Hello SVN
```

查看工作副本的状态：

```shell
[hyp@localhost Auth]$ svn status
?       README
```

此时 readme的状态为？，说明它还未加到版本控制中。

将文件readme加到版本控制，等待提交到版本库。

```shell
[hyp@localhost Auth]$ svn add README
A         README
```

查看工作副本中的状态：

```shell
[hyp@localhost Auth]$ svn status
A       README
```

此时 readme的状态为A,它意味着这个文件已经被成功地添加到了版本控制中。

为了把 readme 存储到版本库中，使用 commit -m 加上注释信息来提交。

如果你忽略了 -m 选项， SVN会打开一个可以输入多行的文本编辑器来让你输入提交信息。

```shell
[hyp@localhost Auth]$ svn commit -m "hahah"
正在增加       README
传输文件数据.
提交后的版本为 1。
```

### 版本回退

当我们想放弃对文件的修改，可以使用 **SVN revert** 命令。

svn revert 操作将撤销任何文件或目录里的局部更改。

我们对文件 readme 进行修改,查看文件状态。

```shell
[hyp@localhost Auth]$ svn status
M       README
```

这时我们发现修改错误，要撤销修改，通过 svn revert 文件 readme 回归到未修改状态。

```shell
[hyp@localhost Auth]$ svn revert README
已恢复“README”
[hyp@localhost Auth]$ svn status
[hyp@localhost Auth]$
```

进行 revert 操作之后，readme 文件恢复了原始的状态。 revert 操作不单单可以使单个文件恢复原状， 而且可以使整个目录恢复原状。恢复目录用 -R 命令，如下。

```shell
svn revert -R trunk
```

但是，假如我们想恢复一个已经提交的版本怎么办。

为了消除一个旧版本，我们必须撤销旧版本里的所有更改然后提交一个新版本。这种操作叫做 reverse merge。

首先，找到仓库的当前版本，现在是版本 22，我们要撤销回之前的版本，比如版本 21。

```shell
svn merge -r 22:21 readme
```

### 查看历史信息

通过svn命令可以根据时间或修订号去除过去的版本，或者某一版本所做的具体的修改。以下四个命令可以用来查看svn 的历史：

- **svn log:** 用来展示svn 的版本作者、日期、路径等等。
- **svn diff:** 用来显示特定修改的行级详细信息。
- **svn cat:** 取得在特定版本的某文件显示在当前屏幕。
- **svn list:** 显示一个目录或某一版本存在的文件。

### 分支

Branch 选项会给开发者创建出另外一条线路。当有人希望开发进程分开成两条不同的线路时，这个选项会非常有用。

比如项目 demo 下有两个小组，svn 下有一个 trunk 版。

由于客户需求突然变化，导致项目需要做较大改动，此时项目组决定由小组 1 继续完成原来正进行到一半的工作（某个模块），小组 2 进行新需求的开发。

那么此时，我们就可以为小组2建立一个分支，分支其实就是 trunk 版（主干线）的一个copy版，不过分支也是具有版本控制功能的，而且是和主干线相互独立的，当然，到最后我们可以通过（合并）功能，将分支合并到 trunk 上来，从而最后合并为一个项目。

我们在本地副本中创建一个 **my_branch** 分支。

```shell
[hyp@localhost Auth]$ ls
branches  README  tags  trunk
[hyp@localhost Auth]$ svn cp trunk/ branches/branche_1
A         branches/branche_1
```

查看状态：

```shell
[hyp@localhost Auth]$ svn status
A       branches
A       branches/branche_1
A       branches/branche_1/crontab
A       branches/branche_1/file1
A       branches/branche_1/file2
A       branches/branche_1/fstab
A       branches/branche_1/inittab
A       branches/branche_1/mtab
A       branches/branche_1/rwtab
A       branches/branche_1/statetab
A       tags
A       trunk
A       trunk/crontab
A       trunk/file1
A       trunk/file2
A       trunk/fstab
A       trunk/inittab
A       trunk/mtab
A       trunk/rwtab
A       trunk/statetab
```

提交新增的分支到版本库。

```shell
[hyp@localhost Auth]$ svn commit -m "add branch_1"
正在增加       branches
正在增加       branches/branche_1
正在增加       branches/branche_1/crontab
正在增加       branches/branche_1/file1
正在增加       branches/branche_1/file2
正在增加       branches/branche_1/fstab
正在增加       branches/branche_1/inittab
正在增加       branches/branche_1/mtab
正在增加       branches/branche_1/rwtab
正在增加       branches/branche_1/statetab
正在增加       tags
正在增加       trunk
正在增加       trunk/crontab
正在增加       trunk/file1
正在增加       trunk/file2
正在增加       trunk/fstab
正在增加       trunk/inittab
正在增加       trunk/mtab
正在增加       trunk/rwtab
正在增加       trunk/statetab
传输文件数据................
提交后的版本为 2。
```

接着我们就到 my_branch 分支进行开发，切换到分支路径并创建 index.html 文件。

```shell
[hyp@localhost Auth]$ cd branches/branche_1/
[hyp@localhost branche_1]$ ls
crontab  file1  file2  fstab  inittab  mtab  rwtab  statetab
```

将 index.html 加入版本控制，并提交到版本库中。

```shell
[hyp@localhost branche_1]$ echo '<!--remember me?-->' > index.html
[hyp@localhost branche_1]$ svn status
?       index.html
[hyp@localhost branche_1]$ svn add index.html
A         index.html
[hyp@localhost branche_1]$ svn commit -m "add index.html"
正在增加       index.html
传输文件数据.
提交后的版本为 3。
[hyp@localhost branche_1]$ svn update
正在升级 '.':
版本 3。
```

切换到 trunk，执行 svn update，然后将 my_branch 分支合并到 trunk 中。

```
[hyp@localhost trunk]$ svn update
正在升级 '.':
版本 3。
[hyp@localhost trunk]$ svn merge ../branches/branche_1/
--- 正在合并 r3 到 “.”:
A    index.html
--- 记录合并 r3 到“.”的信息:
 U   .
```

此时查看目录，可以看到 trunk 中已经多了 my_branch 分支创建的 index.html 文件。

```
root@runoob:~/svn/runoob01/trunk# ll
total 16
drwxr-xr-x 2 root root 4096 Nov  7 03:52 ./
drwxr-xr-x 6 root root 4096 Jul 21 19:19 ../
-rw-r--r-- 1 root root   36 Nov  7 02:23 HelloWorld.html
-rw-r--r-- 1 root root    0 Nov  7 03:52 index.html
-rw-r--r-- 1 root root   22 Nov  7 03:06 readme
```

将合并好的 trunk 提交到版本库中。

```shell
[hyp@localhost trunk]$  svn commit -m "add index.html"
正在发送       .
正在增加       index.html

提交后的版本为 4。
```

参考： [SVN 分支](http://www.runoob.com/svn/svn-branch.html)

### 标签

版本管理系统支持 tag 选项，通过使用 tag 的概念，我们可以给某一个具体版本的代码一个更加有意义的名字。

Tags 即标签主要用于项目开发中的里程碑，比如开发到一定阶段可以单独一个版本作为发布等，它往往代表一个可以固定的完整的版本，这跟 VSS 中的 Tag 大致相同。

我们在本地工作副本创建一个 tag。

```shell
[hyp@localhost Auth]$ svn copy trunk/ tags/v1.0
A         tags/v1.0
```

上面的代码成功完成，新的目录将会被创建在 tags 目录下。

```shell
[hyp@localhost Auth]$ ls tags/
v1.0
[hyp@localhost Auth]$ ls tags/v1.0/
crontab  file1  file2  fstab  index.html  inittab  mtab  rwtab  statetab
```

查看状态。

```shell
[hyp@localhost Auth]$ svn status
A  +    tags/v1.0
A  +    tags/v1.0/crontab
A  +    tags/v1.0/file1
A  +    tags/v1.0/file2
A  +    tags/v1.0/fstab
A  +    tags/v1.0/inittab
A  +    tags/v1.0/mtab
A  +    tags/v1.0/rwtab
A  +    tags/v1.0/statetab
```

提交tag内容。

```shell
[hyp@localhost Auth]$ svn commit -m "tags v1.0"
正在增加       tags/v1.0
正在增加       tags/v1.0/crontab
正在增加       tags/v1.0/file1
正在增加       tags/v1.0/file2
正在增加       tags/v1.0/fstab
正在增加       tags/v1.0/inittab
正在增加       tags/v1.0/mtab
正在增加       tags/v1.0/rwtab
正在增加       tags/v1.0/statetab

提交后的版本为 5。
```

### 常见问题

1、"svn: E155010: 提交失败"问题解决

输入：`svn status`

响应：

```
!       branches/tag0.1
```

显示这样就是虽然使用rm命令删除文件，但是并没有在控制系统中删除，所以，可以使用以下用法来改进。

```shell
[hyp@localhost Auth]$ svn delete --force branches/tag0.1
D         branches/tag0.1
```

2、
