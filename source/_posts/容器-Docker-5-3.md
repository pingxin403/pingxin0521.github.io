---
title: Docker 监控
date: 2019-08-11 18:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

随着Docker被大规模的部署应用，一台Docker主机有可能运行了成百上千个容器，那如何通过可视化的方式了解Docker主机以及其上的容器资源状态以及健康就变得越来越重要。当然很多厂商都推出了自己的监控解决方案，例如Docker那企业需要考量哪些标准去评估这些监控工具的优劣呢？

<!--more-->

1. 是否易于部署
2. 信息呈现的详细度
3. 整个部署过程中日志的聚集程度
4. 数据报警能力
5. 是否可以监控非Docker的资源
6. 投入的成本大小

接下来我们会讨论下docker常用的监控工具，比如docker监控命令、sysdig、cAdvisor、Weave Scope、Prometheus、Cloud Insight等等。

#### docker监控命令

Docker本身已经自带了一些监控命令，包括docker system（docker主机） 、docker ps/top/stats（docker容器）

**docker system df**

```shell
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              8                   1                   1.432GB             1.091GB (76%)
Containers          1                   1                   432B                0B (0%)
Local Volumes       4                   1                   777.8MB             593MB (76%)
Build Cache         0                   0                   0B                  0B
$ docker system df -v
Images space usage:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE                SHARED SIZE         UNIQUE SIZE         CONTAINERS
eg_apt_cacher_ng    latest              1d7ba8e0c34c        58 minutes ago      98.17MB             64.19MB             33.98MB             0
grafana/grafana     latest              05d1bcf30d16        8 days ago          207.4MB             5.553MB             201.9MB             0
ubuntu              latest              775349758637        2 weeks ago         64.19MB             64.19MB             0B                  0
alpine              latest              965ea09ff2eb        3 weeks ago         5.553MB             5.553MB             0B                  0
mysql               5.7                 cd3ed0dfff7e        4 weeks ago         436.6MB             115.1MB             321.5MB             0
mysql               latest              c8ee894bd2bd        4 weeks ago         456.3MB             115.1MB             341.1MB             1
google/cadvisor     v0.27.3             64aa5b3694b2        23 months ago       59.1MB              0B                  59.1MB              0
tutum/influxdb      latest              c061e5808198        3 years ago         289.7MB             0B                  289.7MB             0

Containers space usage:

CONTAINER ID        IMAGE               COMMAND                  LOCAL VOLUMES       SIZE                CREATED             STATUS              NAMES
7295cb280f25        mysql               "docker-entrypoint.s…"   1                   432B                28 minutes ago      Up 14 minutes       mysql

Local Volumes space usage:

VOLUME NAME                                                        LINKS               SIZE
dcb416db0b95f9fffee5fb0f1717a4e46ebb17d9304da80953e8dbf928d59981   0                   592.9MB
fba90c8cc525cbcc556225ee997eacb4fb857ade664a0680b0609bcb569d1309   0                   148.4kB
5d42b1d37682a53b2405bf92980c1955e6a199cf8871de160a249603eebc2e80   0                   0B
mysql_data                                                         1                   184.7MB

Build cache usage: 0B

CACHE ID            CACHE TYPE          SIZE                CREATED             LAST USED           USAGE               SHARED

```

统计docker主机上Images、Containers、Local Volumes、Build Cache等的space usage

-v参数可以获得更急详细的信息

**docker info**（docker system info）

显示docker客户端模式和服务端主机操作系统、内核版本以及其上docker版本、容器数量、镜像数量等等的统计信息

```shell
$ docker system info
Client:
 Debug Mode: false

Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 11
 Server Version: 19.03.4
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-862.14.4.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 1.796GiB
 Name: iZbp125p95hhdb1kcx81ogZ
 ID: LP2A:PULR:SZMU:IDER:LUUN:JMCH:RD7S:GBSV:7QUS:E5YJ:465Y:4LXM
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

```

**docker ps**（docker container ps等价于docker container ls）

列出docker host上目前运行的所有容器，-a参数，将列出所有的容器

**docker top <CONTAINER\>** （docker container top）

查询运行的容器内的进程

```shell
$ docker top mysql
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
polkitd             5645                5615                0                   14:40               pts/0               00:00:07            mysqld

```

**docker stats [CONTAINER]**（docker container stats）

动态实时显示docker host上各个运行容器所占资源利用率统计，包括CPU、Memory、Network I/O、Block I/O、PID

```
$ docker stats 
```

#### [Sysdig](https://sysdig.com)

Sysdig = system（系统）+dig（挖掘）。Sysdig 是一个开源系统发掘工具，用于系统级别的勘察和排障，可以把它看作一系列Linux系统工具的组合，主要包括：

- strace：追踪某个进程产生和接收的系统调用。
- tcpdump：分析网络数据，监控原始网络通信。
- lsof： list opened files, 列出打开的文件。
- top：监控系统性能工具。
- htop ：交互式的进程浏览器，可以用来替换 top 命令。
- iftop ：主要用来显示本机网络流量情况及各相互通信的流量集合。
- lua：一个小巧的脚本语言。该语言的设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

Sysdig 的特性之一在于它不仅能分析Linux 系统的“现场”状态，也能将该状态保存为转储文件以供离线分析检查。你也可以自定义 Sysdig 的行为，通过内建的名为凿子（chisel）的小脚本增强其功能。所以 Sysdig 经常被翻译为系统之锹。通过 Sysdig 工具，用户能够很方便地查看到主机上所有应用程序的cpu、文件 i/o、网络访问状况，这个工具最初的产生就是为了取代传统服务器上的一系列系统检测工具如strace、tcpdump、htop、iftop、lsof 等。它的logo被设计为一个铲子的轮廓，这就寓意着Sysdig对系统信息的强大挖掘能力。

Sysdig不仅可以快速进行系统数据的收集和分析，而且还专门提供了容器级别的信息采集命令，支持查看指定容器之间的网络流量、查看特定容器的 CPU使用情况等。下图是 Sysdig 监控 Docker 容器的示意图。

![Ma2Anf.png](https://s2.ax1x.com/2019/11/15/Ma2Anf.png)

下面先来看下Sysdig的安装：

Sysdig官方提供三种安装方式（以Ubuntu系统为例）：

```
$ curl -s https://s3.amazonaws.com/download.draios.com/stable/install-sysdig | sudo bash

#或

$ curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | sudo apt-key add -
$ sudo curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list 
$ sudo apt-get update
$ sudo apt-get -y install linux-headers-$(uname -r)
$ sudo apt-get- y install sysdig

#或
$ docker run -i -t \
--name sysdig \
--privileged \
-v /var/run/docker.sock:/host/var/run/docker.sock \
-v /dev:/host/dev \
-v /proc:/host/proc:ro \
-v /boot:/host/boot:ro \
-v /lib/modules:/host/lib/modules:ro \
-v /usr:/host/usr:ro \
sysdig/sysdig


```

Sysdig安装后，我们就可以体验Sysdig的强大功能了。

首先，Sysdig提供了强大的交互式工具——csysdig（这个top命令类似）

![MaR8xI.png](https://s2.ax1x.com/2019/11/15/MaR8xI.png)

终端输入csysdig，默认进入系统Processes监控页面,最上面显示目前监控的是整个系统Processes,中间动态实时显示系统Processes的变化

中间左面，列出了sysdig监控的项目，如Processes、Directories、Containers、K8s Controllers等等，右面展示监控的详细信息说明

按上下方向键选择Containers,按Enter进入Containers监控页面

当然sysdig也提供了非交互式的命令——sysdig

比如查询容器weavescope 的cpu变化：

```
sysdig -pc -c topprocs_cpu container.name=weavescope
```

下面是sysdig --help给出的examples，sysdig的命令还是相对比较全面的。有兴趣的读者可以体验下。

```
Capture all the events from the live system and print them to screen

   $sysdig

Capture all the events from the live system and save them to disk

   $sysdig -w dumpfile.scap

Read events from a file and print them to screen

   $sysdig -r dumpfile.scap

Print all the open system calls invoked by cat

   $sysdig proc.name=cat and evt.type=open

Print the name of the files opened by cat

   $sysdig -p"%evt.arg.name" proc.name=cat and evt.type=open
```

#### cAdvisor

cAdvisor是Google用来监测单节点的资源信息的监控工具。虽然Docker提供了一些CLI（dockerps/top/stats等）的命令行的功能，但cAdvisor图形化的提供了一目了然的单节点多容器的资源监控功能。而且cAdvisor是免费的，cAdvisor作为一个很不错的工具，已经引起越来越多人的关注。

```shell
$ docker run -d -p 8080:8080 \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:rw \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--name=cadvisor \
google/cadvisor:latest

```

**登录cAdvisor的UI界面：**

在浏览器通过yourIp:8080就可以登录cAdvisor的UI界面。

cAdvisor实现了两个层次的监控：

1. 主机，包括主机的Processes、CPU、Memory、Network、Filesystem

2. 容器，包括容器的Processes、CPU、Memory、Network、Filesystem

点击Docker Containers进入Docker Containers页面,该页面列出了主机上运行的containers（docker ps）、docker信息（docker info）、pull到本地的images（docker images）

cAdvisor非常的简单，而且很好的从主机和容器两个层面实现了资源的监控，但也存在如下的局限性：

1. 只能监控单主机，如果想监控多主机及其上的容器，需要在每台主机上部署cAdvisor。即便是这样Google的Kubernetes中也缺省地将其作为单节点的资源监控工具，各个节点缺省会被安装上cAdvisor。

2. 监控的资源是实时的，并不能反映一段时间内的变化趋势。这是因为cAdvisor并不储存数据，如果配合第三方工具，例如InfluxDB、Grafana就可以达到理想的效果。

#### weavescope

本节介绍Weave Scope，WeaveScope是一款开源项目，项目地址：[https://github.com/weaveworks/scope](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fweaveworks%2Fscope)。Weave Scope会自动生成容器之间的关系图，方便理解容器之间的关系，也方便监控容器化和微服务化的应用。

Weave Scope的安装步骤：（前提已安装Docker）

下载scope的二进制安装文件：

```
curl -L https://github.com/weaveworks/scope/releases/download/latest_release/scope -o /usr/local/bin/scope
```

（本质上就是一个shell脚本，感兴趣的读者可以下载后查看）

赋予可执行权限：

```
chmod a+x /usr/local/bin/scope
```

启动scope：              

```
scope launch
```

该scope脚本将从DockerHub上下载Scope镜像并启动容器。

scope的其它命令可以通过scope help查询。

根据scope launch最后的提示，浏览器输入[http://192.168.1.108:4040/](https://link.jianshu.com?t=http%3A%2F%2F192.168.1.108%3A4040%2F)，进入weavescope页面。可以对PROCESS、CONTAINERS、HOSTS分别以图形和图表的形式列出。

![MaHl0s.png](https://s2.ax1x.com/2019/11/15/MaHl0s.png)

其中：

- PROCESS可以按照NAME显示；
- CONTAINERS可以按照DNS NAME和IMAGE显示；
- HOSTS可以按照WEAVENET网络显示。
- 左上角区域：提供搜索功能
- 左下角区域：对显示的对象按照不同的条件进行过滤显示。比如CONTAINERS可以选择系统容器还是应用容器，运行的容器还是停止的容器等等。
- 右上角区域：live和pause，分别表示监控的是实时的资源，还是几秒钟之前的，两者之间可以切换。
- 右下角区域：+/-可以对中间区域的对象进行放大和缩小；在下面提供页面重载、页面加深、HELP等功能。

对于图表界面：

![MaHIAI.png](https://s2.ax1x.com/2019/11/15/MaHIAI.png)

下面分别对PROCESS、CONTAINERS和HOSTS分别说明：

1. PROCESS

   点击scope-probe后会显示该进程的详细信息，包括：

   - STATUS（CPU、MEMORY、OPENFILES）
   - INFO（PID、COMMAND、PARENTPID、THREADS）
   - INBOUND
   - OUTBOUND

2. CONTAINERS

   点击后显示cadvisor容器的详细信息，包括：

   - STATUS（CPU、MEMORY）
   - INFO（IMAGE、COMMAND、STATE、NETWORKS、UPTIME、RESTART、IPS、PORTS、CREATED、ID）
   - INBOUND
   - OUTBOUND
   - PROCESS
   - ENVIRONMENTVARIABLES（PATH）
   - DOCKER LABELS（MAINTAINER、WORKS WEAVE ROLE）
   - IMAGE（ID、NAME、SIZE、VIRTUAL SIZE）

   另外，还包括对容器的控制，从左至右依次为attch、exec shell、restart、pause、stop，如果执行了pause还会有unpause，执行了stop还会有start、remove

3. HOSTS

   主机的详细信息，包括：

   - STATUS（CPU、MEMORY、LOAD）
   - INFO（KERNEL VERSION、UPTIME、HOSTNAME、OS、LOCALNETWORKS、SCOPE VERSION）
   - INBOUND
   - OUTBOUND
   - CONTAINERS（CPU、MEMORY）
   - PROCESSES（PID、CPU、MEMORY）
   - CONTAINER IMAGES

**多主机管理**

其实weavescope可以做到对多主机进行监控。

```
scope launch host1-ip host2-ip ... hostn-ip
```

以两台主机为例：

Ubuntu-001：192.168.1.108

Ubuntu-002：192.168.1.109

在两台Docker主机分别执行`scope launch 192.168.1.108 192.168.1.109`

然后分别以http://192.168.1.108:4040/和http://192.168.1.109:4040/均可以。

为什么scope可以监控多主机呢？这是因为scope容器启动使用的是host网络

