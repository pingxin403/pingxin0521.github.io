---
title: Docker 基础概念
date: 2019-07-05 08:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

#### Docker内部构建

要理解Docker内部构建，需要理解以下三种部件：

- Docker镜像（Image）

- Docker容器（Container）

- Docker仓库（repository）

基本上理解了这三个概念，就理解了Docker的整个生命周期。

<!--more-->

**1）Docker镜像（Image）**

Docker镜像就是一个只读的模板。比如，一个镜像可以包含一个完整的ubuntu操作系统环境，里面仅安装了apache或用户需要的其他应用程序。镜像可以用来创建Docker容器。另外Docker提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人哪里下载一个已经做好的镜像来直接使用。

**2）Docker容器（Container）**

Docker利用容器来运行应用。容器是从镜像创建的运行实例，它可以被启动、开始、停止、 删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看做是一个简易版的Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

**3）Docker仓库（repository）**

![UTOOLS1571991329534.png](https://i.loli.net/2019/10/25/8NugPBQwLJXxUmT.png)

仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈， 并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是Docker Hub， 存放了数量庞大的镜像供用户下载。国内的公开仓库包括Docker Pool等，可以提供大陆用户更稳定快速的访问。当然，用户也可以在本地网络内创建一个私有仓库。当用户创建了自己的镜像之后就可以使用push命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上pull下来就可以了。

注：Docker仓库的概念跟Git类似，注册服务器可以理解为GitHub这样的托管服务。

#### Docker引擎

docker引擎是一个c/s结构的应用，主要组件见下图：

![UTOOLS1571991004283.png](https://i.loli.net/2019/10/25/aLl2hSp63KtmJqx.png)

- Server是一个常驻进程

- REST API 实现了client和server间的交互协议

- CLI 实现容器和镜像的管理，为用户提供统一的操作界面

![UTOOLS1571991054638.png](https://i.loli.net/2019/10/25/oFyEdVCUpsKvmnI.png)

#### Docker构架

Docker使用C/S架构，Client 通过接口与Server进程通信实现容器的构建，运行和发布。client和server可以运行在同一台集群，也可以通过跨主机实现远程通信。

![UTOOLS1571991094075.png](https://i.loli.net/2019/10/25/RpJE9ArUzfTWb5i.png)

描述： C/S架构，docker是三部分组成：client,docker_host(server),registry(仓库)，无论是C，还是S端，由docker一个程序来提供，有很多子程序，其中有一个子程序叫daemon,表示运行为守护进程，所以运行为daemon就是把服务器变成为守护进程服务器来使用，可以监听在某个套接字之上，为了安全保是提供本机unix.socket的套接字，如mysql是由tcp/ip加端口，和socket文件来实现通信，docker也是C/S架构，所以服务器端是监听在某个套接字上，但是docker支持三种类型的套接字：ipv4地址加端口，ipv6地址加端口，unix sock file实际是监听在本地的一个文件上，这样另外两个没有监听，导致一个结果是只允许客户端是本地的

在docker主机上有容器，镜像，是docker主机上非常重要的组成部分，镜像来自于docker registry(docker镜像仓库)，默认是到docker hub上下载镜像，镜像是分层构建的，很多基层镜像是可以共享给很多上层镜像使用的，启动容器时就是基于容器来启动，在镜像的基础上为容器创建一个专用的可写层，所以镜像也要在docker主机上存储，但是在主机上运行那些容器是无论评估的，所以有
一个专门的仓库来存储镜像，docker hub可能有几十万个镜像，下载时是默认使用https来操作，client与docker host之间也是使用http/https协议，docker的api也是一个标准的restful的api.

docker运行过程中，开始下载时可能比较慢，要从国外下载镜像，取决于互联网连接速度，在大陆有一台docker的镜像服务器CA，但是加速的效果不是很好，可以使用阿里云使用的加速(快，每个人都有自己私有账号，要在阿里云的开发者平台上注册一个账号)

随着Docker的不断流行与发展，docker公司（或称为组织）也开启了商业化之路，Docker 从 17.03版本之后分为 CE（Community Edition） 和 EE（Enterprise Edition）。

参看：<https://www.cnblogs.com/webenh/p/11254387.html>

#### docke核心技术

核心技术架构

![UTOOLS1571991533238.png](https://i.loli.net/2019/10/25/TQ8Sg5YhD4XFuvy.png)

**名字空间（Namespaces）**

名字空间是 Linux 内核一个强大的特性。每个容器都有自己单独的名字空间，运行在其中的应用都像是在独立的操作系统中运行一样。名字空间保证了容器之间彼此互不影响。

1. pid 名字空间
   不同用户的进程就是通过 pid 名字空间隔离开的，且不同名字空间中可以有相同 pid。所有的 LXC 进程在Docker 中的父进程为Docker进程，每个 LXC 进程具有不同的名字空间。同时由于允许嵌套，因此可以很方便的实现嵌套的 Docker 容器。

2. net 名字空间
   有了 pid 名字空间, 每个名字空间中的 pid 能够相互隔离，但是网络端口还是共享 host 的端口。网络隔离是通过 net 名字空间实现的， 每个 net 名字空间有独立的 网络设备, IP 地址, 路由表, /proc/net 目录。这样每个容器的网络就能隔离开来。Docker 默认采用 veth 的方式，将容器中的虚拟网卡同 host 上的一 个Docker网桥 docker0 连接在一起。

3. ipc 名字空间
   容器中进程交互还是采用了 Linux 常见的进程间交互方法(interprocess communication – IPC), 包括信号量、消息队列和共享内存等。然而同 VM 不同的是，容器的进程间交互实际上还是 host 上具有相同 pid 名字空间中的进程间交互，因此需要在 IPC 资源申请时加入名字空间信息，每个 IPC 资源有一个唯一的 32位 id。

4. mnt 名字空间
   类似 chroot，将一个进程放到一个特定的目录执行。mnt 名字空间允许不同名字空间的进程看到的文件结构不同，这样每个名字空间 中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个名字空间中的容器在 /proc/mounts 的信息只包含所在名字空间的 mount point。

5. uts 名字空间
   UTS(“UNIX Time-sharing System”) 名字空间允许每个容器拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非 主机上的一个进程。

6. user 名字空间
每个容器可以有不同的用户和组 id, 也就是说可以在容器内用容器内部的用户执行程序而非主机上的用户。

**控制组（cgroups）**

控制组（cgroups）是 Linux 内核的一个特性，主要用来对共享资源进行隔离、限制、审计等。只有能控制分配到容器的资源，才能避免当多个容器同时运行时的对系统资源的竞争。
控制组技术最早是由 Google 的程序员 2006 年起提出，Linux 内核自 2.6.24 开始支持。

控制组可以提供对容器的内存、CPU、磁盘 IO 等资源的限制和审计管理。

**联合文件系统（UnionFS）**

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into asingle virtual filesystem)。
联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 中使用的 AUFS（AnotherUnionFS）就是一种联合文件系统。 AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。

Docker 目前支持的联合文件系统种类包括 AUFS, btrfs, vfs 和 DeviceMapper。

**容器格式**

最初，Docker 采用了 LXC 中的容器格式。自 1.20 版本开始，Docker 也开始支持新的 libcontainer 格式，并作为默认选项。

**工具**

- docker：docker 的命令行工具，是给用户和 docker daemon 建立通信的客户端。

- dockerd：dockerd 是 docker 架构中一个常驻在后台的系统进程，称为 docker daemon，dockerd 实际调用的还是 containerd 的 api 接口（rpc 方式实现）,docker daemon 的作用主要有以下两方面：接收并处理 docker client 发送的请求；管理所有的 docker 容器。有了 containerd 之后，dockerd 可以独立升级，以此避免之前 dockerd 升级会导致所有容器不可用的问题。

- containerd:containerd 是 dockerd 和 runc 之间的一个中间交流组件，docker 对容器的管理和操作基本都是通过 containerd 完成的。containerd 的主要功能有：

  - 容器生命周期管理
  - 日志管理
  - 镜像管理
  - 存储管理
  - 容器网络接口及网络管理

- containerd-shim

  containerd-shim 是一个真实运行容器的载体，每启动一个容器都会起一个新的containerd-shim的一个进程， 它直接通过指定的三个参数：容器id，boundle目录（containerd 对应某个容器生成的目录，一般位于：/var/run/docker/libcontainerd/containerID，其中包括了容器配置和标准输入、标准输出、标准错误三个管道文件），运行时二进制（默认为runC）来调用 runc 的 api 创建一个容器，上面的 docker 进程图中可以直观的显示。其主要作用是：

  - 它允许容器运行时(即 runC)在启动容器之后退出，简单说就是不必为每个容器一直运行一个容器运行时(runC)
  - 即使在 containerd 和 dockerd 都挂掉的情况下，容器的标准 IO 和其它的文件描述符也都是可用的
  - 向 containerd 报告容器的退出状态

  有了它就可以在不中断容器运行的情况下升级或重启 dockerd，对于生产环境来说意义重大。

- runC

  runC 是 Docker 公司按照 OCI 标准规范编写的一个操作容器的命令行工具，其前身是 libcontainer 项目演化而来，runC 实际上就是 libcontainer 配上了一个轻型的客户端，是一个命令行工具端，根据 OCI（开放容器组织）的标准来创建和运行容器，实现了容器启停、资源隔离等功能。

  一个例子，使用 runC 运行 busybox 容器:

  ```
  # mkdir /container
  # cd /container/
  # mkdir rootfs
  
  准备容器镜像的文件系统,从 busybox 镜像中提取
  # docker export $(docker create busybox) | tar -C rootfs -xvf -    
  # ls rootfs/
  bin  dev  etc  home  proc  root  sys  tmp  usr  var
  
  有了rootfs之后，我们还要按照 OCI 标准有一个配置文件 config.json 说明如何运行容器，
  包括要运行的命令、权限、环境变量等等内容，runc 提供了一个命令可以自动帮我们生成
  # docker-runc spec
  # ls
  config.json  rootfs
  # docker-runc run simplebusybox    #启动容器
  / # ls
  bin   dev   etc   home  proc  root  sys   tmp   usr   var
  / # hostname
  runc
  ```

#### docker event state

![UTOOLS1571991436004.png](https://i.loli.net/2019/10/25/fTpbRWod2mICk49.png)

**nginx镜像**

描述: 一般是基于centos,ubuntu安装的，可能比较大，还可以给其他小的发行版本安装，alpine是一个专门用于构建非常小的镜像的微型发行版本，可以给程序运行提供基础的环境，但是体积很小，测试时可以使用，但是缺少调试使用的工具，生产环境尽量自行做镜像，而且带调试工具的.

**Alpine知识**

Alpine 操作系统是一个面向安全的轻型 Linux 发行版。它不同于通常 Linux 发行版，Alpine 采用了 musl libc 和 busybox 以减小系统的体积和运行时资源消耗，但功能上比 busybox 又完善的多，因此得到开源社区越来越多的青睐。在保持瘦身的同时，Alpine 还提供了自己的包管理工具 apk，可以通过 https://pkgs.alpinelinux.org/packages 网站上查询包信息，也可以直接通过 apk 命令直接查询和安装各种软件。

Alpine 由非商业组织维护的，支持广泛场景的 Linux发行版，它特别为资深/重度Linux用户而优化，关注安全，性能和资源效能。Alpine 镜像可以适用于更多常用场景，并且是一个优秀的可以适用于生产的基础系统/环境。
Alpine Docker 镜像也继承了 Alpine Linux 发行版的这些优势。相比于其他 Docker 镜像，它的容量非常小，仅仅只有 5 MB 左右（对比 Ubuntu 系列镜像接近 200 MB），且拥有非常友好的包管理机制。官方镜像来自 docker-alpine 项目。

目前 Docker 官方已开始推荐使用 Alpine 替代之前的 Ubuntu 做为基础镜像环境。这样会带来多个好处。包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。

#### 参考

1. 《CentsOS下安装Docker》：https://www.cnblogs.com/wq3435/p/6479768.html
2. 《Docker 命令大全》：http://www.runoob.com/docker/docker-command-manual.html
3. 《深入分析Docker镜像原理》：https://blog.csdn.net/xuguokun1986/article/details/79295947
4. 《创建自己的Docker基础镜像》：http://www.cnblogs.com/cocowool/p/make_your_own_base_docker_image.html
5. 《Linux文件系统之aufs》：https://segmentfault.com/a/1190000008489207
6. 《Container内不需要OS，为何需要OS的基础镜像？》：http://dockone.io/question/6
7. 浅谈linux中的根文件系统（rootfs的原理和介绍）》：https://blog.csdn.net/LEON1741/article/details/78159754
8. 《Docker容器原理与实现》：https://wenku.baidu.com/view/9f5ab08df424ccbff121dd36a32d7375a417c63f.html
9. 《Docker 核心技术与实现原理》：https://draveness.me/docker

10. 《DOCKER基础技术》：https://coolshell.cn/articles/17010.html

11. 《Docker底层架构核心技术》：http://www.dockerinfo.net/698.html

12. 《Docker的概念及剖析原理和特点》：http://blog.51cto.com/kangshuo/1930487
13. [Use of containerd-shim in docker-architecture](https://groups.google.com/forum/#!topic/docker-dev/zaZFlvIx1_k)
14.  [从 docker 到 runC](https://www.cnblogs.com/sparkdev/p/9129334.html)
15.  [OCI 和 runc：容器标准化和 docker](http://cizixs.com/2017/11/05/oci-and-runc/)
16.  [Open Container Initiative](https://github.com/opencontainers)