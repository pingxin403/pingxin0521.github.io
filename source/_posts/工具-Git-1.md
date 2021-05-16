---
title: Git入门
date: 2019-03-18 09:18:59
tags:
 - 工具
 - Git
 - 版本控制
categories:
 - 工具
 - 版本控制
 - Git
---

### 前言

>工欲善其事必先利其器             --孔子 

如果是对 **版本控制** 有所了解的同学，应该听说过Git。Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件（听说是因为BitKeeper开始收费的原因）。另外一个现在比较常使用的版本控制软件是SVN，SVN是Subversion的简称，目前是Apache项目底下的一个开放源代码的版本控制系统，它的设计目标就是取代CVS。我们会在另外的文章介绍SVN。

<!--more-->

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16rmqctahj307y046wej.jpg)

现如今，几乎所有的开发IDE都支持了Git方式来控制版本，而各大网站也使用Git协议来进行代码托管，如最负盛名的[GitHub](https://github.com)，以及国内的[Coding](https://coding.net)、[码云Gitee](https://gitee.com/)等，都在支持着Git。

### 特性

Git 在保存和处理各种信息的时候，虽然操作起来的命令形式非常相近，但它与其他版本控制系统的做法颇为不同。理解这些差异将有助于你准确地使用 Git 提供的各种工具。

#### 直接记录快照，而非差异比较

Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化，而大多数其他系统则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar 等等）每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容，其他系统工作情况如下：

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16ss80r50j30dw06774b.jpg)

Git不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16ss65vduj30dw066dfx.jpg)

这是 Git 同其他系统的重要区别。它完全颠覆了传统版本控制的套路，并对各个环节的实现方式作了新的设计。Git 更像是个小型的文件系统，但它同时还提供了许多以此为基础的超强工具，而不只是一个简单的 VCS

#### 近乎所有操作都是在本地执行

在 Git 中的绝大多数操作都只需要访问本地文件和资源，不用连网。因为 Git 在本地磁盘上就保存着所有当前项目的历史更新，所以处理起来速度飞快。

举个例子，如果要浏览项目的历史更新摘要，Git 不用跑到外面的服务器上去取数据回来，而直接从本地数据库读取后展示给你看。所以任何时候你都可以马上翻阅，无需等待。如果想要看当前版本的文件和一个月前的版本之间有何差异，Git 会取出一个月前的快照和当前文件作一次差异运算，而不用请求远程服务器来做这件事，或是把老版本的文件拉到本地来作比较。

#### 时刻保持数据完整性

在保存到 Git 之前，所有数据都要进行内容的校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。换句话说，不可能在你修改了文件或目录之后，Git 一无所知。这项特性作为 Git 的设计哲学，建在整体架构的最底层。所以如果文件在传输时变得不完整，或者磁盘损坏导致文件数据缺失，Git 都能立即察觉。

Git 使用 SHA-1 算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串。Git 的工作完全依赖于这类指纹字串，所以你会经常看到这样的哈希值。实际上，所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。

#### 多数操作仅添加数据

常用的 Git 操作大多仅仅是把数据添加到数据库。因为任何一种不可逆的操作，比如删除数据，都会使回退或重现历史版本变得困难重重。在 Git 里，一旦提交快照之后就完全不用担心丢失数据，特别是养成定期推送到其他仓库的习惯的话。这种高可靠性令我们的开发工作安心不少，尽管去做各种试验性的尝试好了，再怎样也不会弄丢数据。

#### 文件的三种状态

~~这是Git中非常重要的一点，必须要记住并理解。~~

对于任何一个文件，在 Git 内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；已暂存表示把已修改的文件放在下次提交时要保存的清单中。

Git 管理项目时，文件流转的三个工作区域：Git 的工作目录，暂存区域，以及本地仓库。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g16ss9i3h2j30dw0cswep.jpg)

每个项目都有一个 Git 目录，它是 Git 用来保存元数据和对象数据库的地方。该目录非常重要，每次克隆镜像仓库的时候，实际拷贝的就是这个目录里面的数据。

从项目中取出某个版本的所有文件和目录，用以开始后续工作的叫做工作目录。这些文件实际上都是从 Git 目录中的压缩对象数据库中提取出来的，接下来就可以在工作目录中对这些文件进行编辑。

所谓的暂存区域只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。

基本的 Git 工作流程如下：

1. 在工作目录中修改某些文件。
2. 对修改后的文件进行快照，然后保存到暂存区域。
3. 提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中。

所以，我们可以从文件所处的位置来判断状态：如果是 Git 目录中保存着的特定版本文件，就属于已提交状态；如果作了修改并已放入暂存区域，就属于已暂存状态；如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

### Git安装

#### 从源码安装

Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。在有 yum 的系统上（比如 Fedora）或者有 apt-get 的系统上（比如 Debian 体系），可以用下面的命令安装：

```shell
$ yum install curl-devel expat-devel gettext-devel \
  openssl-devel zlib-devel perl-ExtUtils-MakeMaker

#ubuntu
$ apt install -y  build-essential
$ apt install  -y  libexpat1-dev gettext libz-dev libssl-dev libcurl4-openssl-dev
```

然后从官网上下载最新的源代码包：[https://git-scm.com/download](https://git-scm.com/download)

然后编译并安装：

```shell
$ tar -zxf git-2.9.5.tar.gz
$ cd git-2.9.5
$ make prefix=/usr/local all
$ sudo make prefix=/usr/local install
```

现在已经可以用 `git` 命令了，用 `git` 把 Git 项目仓库克隆到本地，以便日后随时更新：

```shell
$ git clone git://git.kernel.org/pub/scm/git/git.git
```

#### Linux安装

可以参考[官网文档](https://www.git-scm.com/download/linux)。

#### 在Mac上安装

图形化的Git安装工具：[http://sourceforge.net/projects/git-osx-installer/](http://sourceforge.net/projects/git-osx-installer/)。

#### 在Windows上安装

直接安装即可：[https://gitforwindows.org/](https://gitforwindows.org/)。

给 Windows 用户的敬告：你应该在 该软件提供的 Unix 风格的 shell 来运行 Git。在 Unix 风格的 shell 中，可以使用本书中提及的复杂多行的命令。对于那些需要在 Windows 命令行中使用 Git 的用户，必须注意：在参数中间有空格的时候，必须使用双引号将参数括起来（在 Linux 中是单引号）；另外，如果扬抑符（^）作为参数的结尾，并且作为这一行的最后一个字符，则这个参数也需要用双引号括起来。因为扬抑符在 Windows 命令行中表示续行（译注：即下一行为这一行命令的继续）。

### 初次运行配置

一般在新的系统上，我们都需要先配置下自己的 Git 工作环境。配置工作只需一次，以后升级时还会沿用现在的配置。当然，如果需要，你随时可以用相同的命令修改已有的配置。

Git 提供了一个叫做 `git config` 的工具（译注：实际是 `git-config` 命令，只不过可以通过 `git` 加一个名字来呼叫此命令。），专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- `/etc/gitconfig` 文件：系统中对所有用户都普遍适用的配置。若使用 `git config` 时用 `--system` 选项，读写的就是这个文件。
- `~/.gitconfig` 文件：用户目录下的配置文件只适用于该用户。若使用 `git config` 时用 `--global` 选项，读写的就是这个文件。
- 当前项目的 Git 目录中的配置文件（也就是工作目录中的 `.git/config` 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 `/etc/gitconfig` 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 `.gitconfig` 文件。主目录即 `$HOME` 变量指定的目录，一般都是 `C:\Documents and Settings\$USER`。此外，Git 还会尝试找寻 `/etc/gitconfig` 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

#### 用户信息

第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录：

```shell
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

如果用了 `--global` 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 `--global` 选项重新配置即可，新的设定保存在当前项目的 `.git/config` 文件里。

#### 文本编辑器

接下来要设置的是默认使用的文本编辑器。Git 需要你输入一些额外消息的时候，会自动调用一个外部文本编辑器给你用。默认会使用操作系统指定的默认编辑器，一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：

```shell
$ git config --global core.editor emacs
```

#### 差异分析工具

还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：

```shell
$ git config --global merge.tool vimdiff
```

Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。

#### 查看配置信息

要检查已有的配置信息，可以使用 `git config --list` 命令：

```shell
$ git config --list
user.name=Scott Chacon
user.email=schacon@gmail.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```

有时候会看到重复的变量名，那就说明它们来自不同的配置文件（比如 `/etc/gitconfig` 和 `~/.gitconfig`），不过最终 Git 实际采用的是最后一个。

也可以直接查阅某个环境变量的设定，只要把特定的名字跟在后面即可，像这样：

```shell
$ git config user.name
Scott Chacon
```

#### 获取帮助

想了解 Git 的各式工具该怎么用，可以阅读它们的使用帮助，方法有三：

```shell
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

比如，要学习 config 命令可以怎么用，运行：

```shell
$ git help config
```

### github提速

我们可以通过修改hosts文件来提速,获取github的IP地址

1.手动访问：https://www.ipaddress.com/ 网址，然后依次获取以下三个网址的IP

```
github.com
github.global.ssl.fastly.net
codeload.github.com
```

修改系统本地的hosts文件，添加内容

```
$ sudo vim /etc/hosts
140.82.113.4 github.com
199.232.5.194 github.global.ssl.fastly.net
140.82.114.9 codeload.github.com
```

2.重启网络服务

```
$ sudo /etc/init.d/networking restart
```

参考：[《Pro Git》](http://iissnan.com/progit/)