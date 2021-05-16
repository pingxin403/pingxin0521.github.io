---
title: Docker 特性
date: 2019-07-06 11:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

#### Docker 端口映射

```
# Find IP address of container with ID <container_id> 通过容器 id 获取 ip 
$ docker inspect <container_id> | grep IPAddress | cut -d ’"’ -f 4
```

<!--more-->

无论如何，这些 ip 是基于本地系统的并且容器的端口非本地主机是访问不到的。此外，除了端口只能本地访问外，对于容器的另外一个问题是这些 ip 在容器每次启动的时候都会改变。

Docker 解决了容器的这两个问题，并且给容器内部服务的访问提供了一个简单而可靠的方法。Docker 通过端口绑定主机系统的接口，允许非本地客户端访问容器内部运行的服务。为了简便的使得容器间通信，Docker 提供了这种连接机制。

**自动映射端口**

-P使用时需要指定--expose选项，指定需要对外提供服务的端口

```
$ sudo docker run -t -P --expose 22 --name server  ubuntu:14.04
```

使用docker run -P自动绑定所有对外提供服务的容器端口，映射的端口将会从没有使用的端口池中 (49000..49900) 自动选择，你可以通过docker ps、docker inspect <container_id>或者docker port <container_id> <port\>确定具体的绑定信息。

**绑定端口到指定接口**

基本语法

```
$ sudo docker run -p [([<host_interface>:[host_port]])|(<host_port>):]<container_port>[/udp] <image> <cmd>
```

默认不指定绑定 ip 则监听所有网络接口。

绑定 TCP 端口

```bash
# 将容器的TCP端口8080绑定到主机127.0.0.1上的TCP端口80。
$ sudo docker run -p 127.0.0.1:80:8080 <image> <cmd> 
# 将容器的TCP端口8080绑定到主机127.0.0.1上动态分配的TCP端口。
$ sudo docker run -p 127.0.0.1::8080 <image> <cmd> 
# 将容器的TCP端口8080绑定到主机所有可用接口上的TCP端口80。
$ sudo docker run -p 80:8080 <image> <cmd> 
# 将容器的TCP端口8080绑定到所有可用接口上动态分配的TCP端口
$ sudo docker run -p 8080 <image> <cmd>

#使用 docker port 来查看当前映射的端口配置，也可以查看到绑定的地址
$ docker port nostalgic_morse 5000
127.0.0.1:49155.
```

注意：

- 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）
- `-p` 标记可以多次使用来绑定多个端口

例如

```bash
$ docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
```

**绑定 UDP 端口**

```bash
# Bind UDP port 5353 of the container to UDP port 53 on 127.0.0.1 of the host machine. 
$ sudo docker run -p 127.0.0.1:53:5353/udp <image> <cmd>
```

#### Docker 网络配置

![1.png](https://i.loli.net/2019/07/30/5d4066b501ba868582.png)

Dokcer 通过使用 Linux 桥接提供容器之间的通信，docker0 桥接接口的目的就是方便 Docker 管理。当 Docker daemon 启动时需要做以下操作：

- creates the docker0 bridge if not present
  - \# 如果 docker0 不存在则创建
- searches for an IP address range which doesn’t overlap with an existing route
  - \# 搜索一个与当前路由不冲突的 ip 段
- picks an IP in the selected range
  - \# 在确定的范围中选择 ip
- assigns this IP to the docker0 bridge
  - \# 绑定 ip 到 docker0

##### Docker 四种网络模式

四种网络模式摘自  [Docker 网络详解及 pipework 源码解读与实践](http://www.infoq.com/cn/articles/docker-network-and-pipework-open-source-explanation-practice)

docker run 创建 Docker 容器时，可以用 --net 选项指定容器的网络模式，Docker 有以下 4 种网络模式：

- host 模式，使用 --net=host 指定。
- container 模式，使用 --net=container:NAMEorID 指定。
- none 模式，使用 --net=none 指定。
- bridge 模式，使用 --net=bridge 指定，默认设置。

**host 模式**

如果启动容器的时候使用 host 模式，那么这个容器将不会获得一个独立的 Network Namespace，而是和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。

例如，我们在 10.10.101.105/24 的机器上用 host 模式启动一个含有 web 应用的 Docker 容器，监听 tcp 80 端口。当我们在容器中执行任何类似 ifconfig 命令查看网络环境时，看到的都是宿主机上的信息。而外界访问容器中的应用，则直接使用 10.10.101.105:80 即可，不用任何 NAT 转换，就如直接跑在宿主机中一样。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

**container 模式**

这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

**none模式**

这个模式和前两个不同。在这种模式下，Docker 容器拥有自己的 Network Namespace，但是，并不为 Docker容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。需要我们自己为 Docker 容器添加网卡、配置 IP 等。

**bridge模式**

![2.png](https://i.loli.net/2019/07/30/5d4067059c79261063.png)

bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置 IP 等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上。当 Docker server 启动时，会在主机上创建一个名为 docker0 的虚拟网桥，此主机上启动的 Docker 容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。接下来就要为容器分配 IP 了，Docker 会从 RFC1918 所定义的私有 IP 网段中，选择一个和宿主机不同的IP地址和子网分配给 docker0，连接到 docker0 的容器就从这个子网中选择一个未占用的 IP 使用。如一般 Docker 会使用 172.17.0.0/16 这个网段，并将 172.17.42.1/16 分配给 docker0 网桥（在主机上使用 ifconfig 命令是可以看到 docker0 的，可以认为它是网桥的管理接口，在宿主机上作为一块虚拟网卡使用）

更多：<https://www.cnblogs.com/zuxing/articles/8780661.html>和<https://www.cnblogs.com/yy-cxd/p/6553624.html>

##### 列出当前主机网桥

安装：

Centos系统

```
$ yum install bridge-utils
```

Ubuntu系统   

```
$ apt-get  install bridge-utils
```

使用：

```
$ sudo brctl show # brctl 工具依赖 bridge-utils 软件包 bridge name bridge id STP enabled interfaces
docker0		8000.024229f93edd	no		
```

##### 查看当前 docker0 ip

```
$ sudo ip address show docker0

4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:29:f9:3e:dd brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

```

在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到 docker0。每个容器都会配置同 docker0 ip 相同网段的专用 ip 地址，docker0 的 IP 地址被用于所有容器的默认网关。

##### 运行一个容器

```
$ sudo docker run -t -i -d ubuntu /bin/bash
52f811c5d3d69edddefc75aff5a4525fc8ba8bcfa1818132f9dc7d4f7c7e78b4 
$ sudo brctl show
bridge name bridge id STP enabled interfaces
docker0 8000.fef213db5a66 no vethQCDY1N
```

以上, docker0 扮演着 52f811c5d3d6 container 这个容器的虚拟接口 vethQCDY1N interface 桥接的角色。

**使用特定范围的 IP**

Docker 会尝试寻找没有被主机使用的 ip 段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对 ip 进一步规划。Docker 允许你管理 docker0 桥接或者通过-b选项自定义桥接网卡，需要安装bridge-utils软件包。

基本步骤如下：

- ensure Docker is stopped
  - \# 确保 docker 的进程是停止的
- create your own bridge (bridge0 for example)
  - \# 创建自定义网桥
- assign a specific IP to this bridge
  - \# 给网桥分配特定的 ip
- start Docker with the -b=bridge0 parameter
  - \# 以 -b 的方式指定网桥

```
# Stopping Docker and removing docker0 
$ sudo service docker stop 
$ sudo ip link set dev docker0 down 
$ sudo brctl delbr docker0 # Create our own bridge 
$ sudo brctl addbr bridge0 
$ sudo ip addr add 192.168.5.1/24 dev bridge0 
$ sudo ip link set dev bridge0 up # Confirming that our bridge is up and running 
$ ip addr show bridge0
4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global bridge0
       valid_lft forever preferred_lft forever # Tell Docker about it and restart (on Ubuntu) $ echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker $ sudo service docker start
```

参考文档:  [Network Configuration](https://docs.docker.com/articles/networking/)

##### 不同主机间容器通信

不同容器之间的通信可以借助于 pipework 这个工具：

```
$ git clone https://github.com/jpetazzo/pipework.git
$ sudo cp -rp pipework/pipework /usr/local/bin/
```

**安装相应依赖软件**

```
$ sudo apt-get install iputils-arping bridge-utils -y
```

**桥接网络**

桥接网络可以参考  [日常问题处理 Tips](https://github.com/opskumu/Day/blob/master/tips/tips.md) 关于桥接的配置说明，这里不再赘述。

```
# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.000c291412cd       no              eth0
docker0         8000.56847afe9799       no              vetheb48029
```

可以删除 docker0，直接把 docker 的桥接指定为 br0。也可以保留使用默认的配置，这样单主机容器之间的通信可以通过 docker0，而跨主机不同容器之间通过 pipework 新建 docker 容器的网卡桥接到 br0，这样跨主机容器之间就可以通信了。

- ubuntu

```
$ sudo service docker stop
$ sudo ip link set dev docker0 down
$ sudo brctl delbr docker0
$ echo 'DOCKER_OPTS="-b=br0"' >> /etc/default/docker
$ sudo service docker start
```

- CentOS 7/RHEL 7

```
$ sudo systemctl stop docker
$ sudo ip link set dev docker0 down
$ sudo brctl delbr docker0
$ cat /etc/sysconfig/docker | grep 'OPTIONS='
OPTIONS=--selinux-enabled -b=br0 -H fd://
$ sudo systemctl start docker
```

**pipework**

![3.png](https://i.loli.net/2019/07/31/5d4069f73357735528.png)

不同容器之间的通信可以借助于 pipework 这个工具给 docker 容器新建虚拟网卡并绑定 IP 桥接到 br0

```
$ git clone https://github.com/jpetazzo/pipework.git
$ sudo cp -rp pipework/pipework /usr/local/bin/
$ pipework 
Syntax:
pipework <hostinterface> [-i containerinterface] <guest> <ipaddr>/<subnet>[@default_gateway] [macaddr][@vlan]
pipework <hostinterface> [-i containerinterface] <guest> dhcp [macaddr][@vlan]
pipework --wait [-i containerinterface]
```

如果删除了默认的 docker0 桥接，把 docker 默认桥接指定到了 br0，则最好在创建容器的时候加上--net=none，防止自动分配的 IP 在局域网中有冲突。

```
$ sudo docker run --rm -ti --net=none ubuntu:19.04 /bin/bash
root@a46657528059:/#
$                  # Ctrl-P + Ctrl-Q 回到宿主机 shell，容器 detach 状态
$ sudo docker  ps
CONTAINER ID    IMAGE          COMMAND       CREATED         STATUS          PORTS      NAMES
a46657528059    ubuntu:19.04   "/bin/bash"   4 minutes ago   Up 4 minutes               hungry_lalande
$ sudo pipework br0 -i eth0 a46657528059 192.168.115.10/24@192.168.115.2 
# 默认不指定网卡设备名，则默认添加为 eth1
# 另外 pipework 不能添加静态路由，如果有需求则可以在 run 的时候加上 --privileged=true 权限在容器中手动添加，
# 但这种安全性有缺陷，可以通过 ip netns 操作
$ sudo docker attach a46657528059
root@a46657528059:/# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 86:b6:6b:e8:2e:4d  
          inet addr:192.168.115.10  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::84b6:6bff:fee8:2e4d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:648 (648.0 B)  TX bytes:690 (690.0 B)
root@a46657528059:/# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.115.2   0.0.0.0         UG    0      0        0 eth0
192.168.115.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

使用ip netns添加静态路由，避免创建容器使用--privileged=true选项造成一些不必要的安全问题：

```
$ docker inspect --format="{{ .State.Pid }}" a46657528059 # 获取指定容器 pid
6350
$ sudo ln -s /proc/6350/ns/net /var/run/netns/6350
$ sudo ip netns exec 6350 ip route add 192.168.0.0/16 dev eth0 via 192.168.115.2
$ sudo ip netns exec 6350 ip route    # 添加成功
192.168.0.0/16 via 192.168.115.2 dev eth0 
... ...
```

在其它宿主机进行相应的配置，新建容器并使用 pipework 添加虚拟网卡桥接到 br0，测试通信情况即可。

另外，pipework 可以创建容器的 vlan 网络，这里不作过多的介绍了，官方文档已经写的很清楚了，可以查看以下两篇文章：

- [Pipework 官方文档](https://github.com/jpetazzo/pipework)
- [Docker 网络详解及 pipework 源码解读与实践](http://www.infoq.com/cn/articles/docker-network-and-pipework-open-source-explanation-practice)

#### Dockerfile

虽然我们可以通过docker commit命令来手动创建镜像，但是通过Dockerfile文件，可以帮助我们自动创建镜像，并且能够自定义创建过程。本质上，Dockerfile就是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。它简化了从头到尾的构建流程并极大的简化了部署工作。使用dockerfile构建镜像有以下好处：

- 像编程一样构建镜像，支持分层构建以及缓存；
- 可以快速而精确地重新创建镜像以便于维护和升级；
- 便于持续集成；
- 可以在任何地方快速构建镜像

##### Dockerfile指令

Dockerfile 是一个包含创建镜像所有命令的文本文件，通过docker build命令可以根据 Dockerfile 的内容构建镜像，在介绍如何构建之前先介绍下 Dockerfile 的基本语法结构。

文档:<https://docs.docker.com/engine/reference/builder/>

Dockerfile 有以下指令选项:

- FROM
- MAINTAINER(已弃用)
- RUN
- CMD
- EXPOSE
- ENV
- ADD
- COPY
- ENTRYPOINT
- VOLUME
- USER
- WORKDIR
- ONBUILD

**1.FROM**

FROM 指令用于设置在新映像创建过程期间将使用的容器映像。

格式：

```
FROM <image>
```

示例：

```
FROM nginx
FROM microsoft/dotnet:2.1-aspnetcore-runtime
```

- FROM指定构建镜像的基础源镜像，如果本地没有指定的镜像，则会自动从 Docker 的公共库 pull 镜像下来。
- FROM必须是 Dockerfile 中**非注释行的第一个指令**，即一个 Dockerfile 从FROM语句开始。
- FROM可以在一个 Dockerfile 中出现多次，如果有需求在一个 Dockerfile 中创建多个镜像。
- 如果FROM语句没有指定镜像标签，则默认使用latest标签。

指定创建镜像的用户

```
MAINTAINER <name>
```

**2.RUN**

RUN 指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。

exec 方式会被解析为一个 JSON 数组，所以必须使用双引号而不是单引号。exec 方式不会调用一个命令 shell，所以也就不会继承相应的变量

格式：

```
RUN ["", "", ""]
RUN
```

示例：

```
RUN apt-get update
RUN mkdir -p /usr/src/redis
RUN apt-get update && apt-get install -y libgdiplus
RUN ["apt-get","install","-y","nginx"]
```

注意：每一个指令都会创建一层，并构成新的镜像。当运行多个指令时，会产生一些非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。因此，在很多情况下，我们可以合并指令并运行，例如：`RUN apt-get update && apt-get install -y libgdiplus`。在命令过多时，一定要注意格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯

RUN产生的缓存在下一次构建的时候是不会失效的，会被重用，可以使用--no-cache选项，即docker build --no-cache，如此便不会缓存。

**3.COPY**

COPY 指令将文件和目录复制到容器的文件系统。文件和目录需位于相对于 Dockerfile 的路径中。

COPY复制新文件或者目录从 并且添加到容器指定路径中 。用法同ADD，唯一的不同是不能指定远程文件 URLS。

格式：

```
COPY <src>... <dest>
```

如果源或目标包含空格，请将路径括在方括号和双引号中。

```
COPY ["", ""]
```

示例：

```
COPY . .
COPY nginx.conf /etc/nginx/nginx.conf
COPY . /usr/share/nginx/html
COPY hom* /mydir/
```

**4.ADD**

ADD 指令与 COPY 指令非常类似，但它包含更多功能。除了将文件从主机复制到容器映像，ADD 指令还可以使用 URL 规范从远程位置复制文件。

格式：

```
ADD<source> <destination>
```

ADD复制本地主机文件、目录或者远程文件 URLS 从 并且添加到容器指定路径中 。

支持通过 [Go](http://lib.csdn.net/base/go) 的正则模糊匹配，具体规则可参见  [Go filepath.Match](http://golang.org/pkg/path/filepath/#Match)

```
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character
```

- **destination路径必须是绝对路径**，如果不存在，会自动创建对应目录
- source路径必须是 Dockerfile 所在路径的相对路径
- 如果是一个目录，只会复制目录下的内容，而目录本身则不会被复制

示例：

```
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

此示例会将 Python for Windows下载到容器映像的 c:\temp 目录。 

**5.WORKDIR**

WORKDIR 指令用于为其他 Dockerfile 指令（如 RUN、CMD）设置一个工作目录，并且还设置用于运行容器映像实例的工作目录。

格式：

```
WORKDIR /path/to/workdir
```

为后续的RUN、CMD、ENTRYPOINT指令配置工作目录。可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

最终路径是/a/b/c。

WORKDIR指令可以在ENV设置变量之后调用环境变量:

```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
```

最终路径则为 /path/$DIRNAME。

**6.CMD**

CMD指令用于设置部署容器映像的实例时要运行的默认命令。例如，如果该容器将承载 NGINX Web 服务器，则 CMD 可能包括用于启动Web服务器的指令，如 nginx.exe。 如果 Dockerfile 中指定了多个CMD 指令，只会计算最后一个指令。

CMD的目的是为了在启动容器时提供一个默认的命令执行选项。如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的命令。

> CMD会在启动容器的时候执行，build 时不执行，而RUN只是在构建镜像的时候执行，后续镜像构建完成之后，启动容器就与RUN无关了，这个初学者容易弄混这个概念，这里简单注解一下。

格式：

```
CMD  "executable","param1","param2"
CMD  "param1","param2"
CMD command param1 param2 (shell form)
```

示例：

```
CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]
CMD c:\\Apache24\\bin\\httpd.exe -w
```

**7.ENTRYPOINT**

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖，而CMD是可以被覆盖的。如果需要覆盖，则可以使用docker run --entrypoint选项。

每个 Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个生效。

格式：

```
ENTRYPOINT ["", ""]
ENTRYPOINT  "executable", "param1", "param2"
ENTRYPOINT command param1 param2 (shell form)
```

示例：

```
ENTRYPOINT ["dotnet", "Magicodes.Admin.Web.Host.dll"] 
```

Exec form ENTRYPOINT 例子：

通过ENTRYPOINT使用 exec form 方式设置稳定的默认命令和选项，而使用CMD添加默认之外经常被改动的选项。

```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

通过 Dockerfile 使用ENTRYPOINT展示前台运行 Apache 服务

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Shell form ENTRYPOINT 例子：

这种方式会在/bin/sh -c中执行，会忽略任何CMD或者docker run命令行选项，为了确保docker stop能够停止长时间运行ENTRYPOINT的容器，确保执行的时候使用exec选项。

```
FROM ubuntu
ENTRYPOINT exec top -b
```

如果在ENTRYPOINT忘记使用exec选项，则可以使用CMD补上:

```
FROM ubuntu
ENTRYPOINT top -b
CMD --ignored-param1 # --ignored-param2 ... --ignored-param3 ... 依此类推
```

**8.ENV**

ENV命令用于设置环境变量。这些变量以”key=value”的形式存在，并可以在容器内被脚本或者程序调用。这个机制给在容器中运行应用带来了极大的便利。

格式：

```
ENV <key> <value>       # 只能设置一个变量
ENV <key>=<value> ...   # 允许一次设置多个变量
```

指定一个环节变量，会被后续RUN指令使用，并在容器运行时保留。

示例：

```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
#等于
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

**9.EXPOSE**

告诉 Docker 服务端容器对外映射的本地端口，需要在 docker run 的时候使用-p或者-P选项生效。

格式：

```
EXPOSE <port> [<port>...]
```

示例：

```
EXPOSE 80
```

**10.VOLUME**

```
VOLUME ["/data"]
```

创建一个可以从本地主机或其他容器挂载的挂载点

**11.ONBUILD**

```
ONBUILD [INSTRUCTION]
```

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile 使用如下的内容创建了镜像 image-A：

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

如果基于 image-A 创建新的镜像时，新的 Dockerfile 中使用 FROM image-A 指定基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

```
# Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
```

使用ONBUILD指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild。

**12.LABEL**

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

该`LABEL`指令将元数据添加到image。A `LABEL`是键值对。要在`LABEL`值中包含空格，请像在命令行分析中一样使用引号和反斜杠。一些用法示例：

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

image可以有多个标签。您可以在一行上指定多个标签。在Docker 1.10之前的版本中，这减小了最终映像的大小，但是情况不再如此。您仍然可以选择通过以下两种方式之一在一条指令中指定多个标签：

```none
LABEL multi.label1="value1" multi.label2="value2" other="value3"
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

基本或父image（该`FROM`行中的image）中包含的标签由您的图像继承。如果标签已经存在但具有不同的值，则最近应用的值将覆盖任何先前设置的值。

要查看image的标签，请使用`docker inspect`命令。

```
"Labels": {
    "com.example.vendor": "ACME Incorporated"
    "com.example.label-with-value": "foo",
    "version": "1.0",
    "description": "This text illustrates that label-values can span multiple lines.",
    "multi.label1": "value1",
    "multi.label2": "value2",
    "other": "value3"
},
```

说了这么多，我们可以用下图来一言以蔽之：

![1.png](https://i.loli.net/2019/07/30/5d4061af7261012038.png)

**Dockerfile Examples**

```dockerfile
# Nginx
#
# VERSION               0.0.1
FROM      ubuntu
MAINTAINER pingxin <m13839441583@163.com>
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server


# Firefox over VNC
#
# VERSION               0.3
FROM ubuntu
# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir ~/.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'
EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]


# Multiple images example
#
# VERSION               0.1
FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f
FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4
# You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```

**docker build**

```bash
$ docker build --help
Usage: docker build [OPTIONS] PATH | URL | -
Build a new image from the source code at PATH
  --force-rm=false     Always remove intermediate containers, even after unsuccessful builds # 移除过渡容器，即使构建失败
  --no-cache=false     Do not use cache when building the image                              # 不实用 cache        
  -q, --quiet=false    Suppress the verbose output generated by the containers               
  --rm=true            Remove intermediate containers after a successful build               # 构建成功后移除过渡层容器
  -t, --tag=""         Repository name (and optionally a tag) to be applied to the resulting image in case of success
```

##### 转义字符

在许多情况下，Dockerfile 指令需要跨多个行；这可通过转义字符完成。 默认 Dockerfile 转义字符是反斜杠 \。 由于反斜杠在 Windows 中也是一个文件路径分隔符，这可能导致出现问题。

以下示例显示使用默认转义字符跨多个行的单个 RUN 指令。

```
FROM microsoft/windowsservercore
RUN powershell.exe -Command \
$ErrorActionPreference = 'Stop'; \
wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
Remove-Item c:\python-3.5.1.exe -Force 
```

要修改转义字符，必须在 Dockerfile 最开始的行上放置一个转义分析程序指令。 如以下示例所示：

```
# escape=`
FROM microsoft/windowsservercore
RUN powershell.exe -Command `
$ErrorActionPreference = 'Stop'; `
wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
Remove-Item c:\python-3.5.1.exe -Force
```

注意，只有两个值可用作转义字符：\ 和 ` 。

##### 优化

- 使用.dockerignore文件

  为了在docker build过程中更快上传和更加高效，应该使用一个.dockerignore文件用来排除构建镜像时不需要的文件或目录。例如,除非. [Git](http://lib.csdn.net/base/git) 在构建过程中需要用到，否则你应该将它添加到.dockerignore文件中，这样可以节省很多时间。

- 避免安装不必要的软件包

  为了降低复杂性、依赖性、文件大小以及构建时间，应该避免安装额外的或不必要的包。例如，不需要在一个[数据库](http://lib.csdn.net/base/mysql) 镜像中安装一个文本编辑器。

- 每个容器都跑一个进程

  在大多数情况下，一个容器应该只单独跑一个程序。解耦应用到多个容器使其更容易横向扩展和重用。如果一个服务依赖另外一个服务，可以参考  [Linking Containers Together](https://docs.docker.com/userguide/dockerlinks/) 。

- 最小化层

  我们知道每执行一个指令，都会有一次镜像的提交，镜像是分层的结构，对于Dockerfile，应该找到可读性和最小化层之间的平衡。

- 多行参数排序

  如果可能，通过字母顺序来排序，这样可以避免安装包的重复并且更容易更新列表，另外可读性也会更强，添加一个空行使用\换行:

  ```
  RUN apt-get update && apt-get install -y \
    bzr \
    cvs \
    git \
    mercurial \
    subversion
  ```

- 创建缓存

  镜像构建过程中会按照Dockerfile的顺序依次执行，每执行一次指令 Docker 会寻找是否有存在的镜像缓存可复用，如果没有则创建新的镜像。如果不想使用缓存，则可以在docker build时添加--no-cache=true选项。

  从基础镜像开始就已经在缓存中了，下一个指令会对比所有的子镜像寻找是否执行相同的指令，如果没有则缓存失效。在大多数情况下只对比Dockerfile指令和子镜像就足够了。ADD和COPY指令除外，执行ADD和COPY时存放到镜像的文件也是需要检查的，完成一个文件的校验之后再利用这个校验在缓存中查找，如果检测的文件改变则缓存失效。RUN apt-get -y update命令只检查命令是否匹配，如果匹配就不会再执行更新了。

> 为了有效地利用缓存，你需要保持你的 Dockerfile 一致，并且尽量在末尾修改。

#### 容器数据管理

 docker管理数据的方式有两种：

- 数据卷
- 数据卷容器

##### 数据卷

- `数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

  - `数据卷` 可以在容器之间共享和重用
  - 对 `数据卷` 的修改会立马生效
  - 对 `数据卷` 的更新，不会影响镜像
  - `数据卷` 默认会一直存在，即使容器被删除

  > 注意：`数据卷` 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的 `数据卷`。

**添加一个数据卷**

你可以使用-v选项添加一个数据卷，或者可以使用多次-v选项为一个 docker 容器运行挂载多个数据卷。

```
$ sudo docker run --name data -v /data -t -i ubuntu:19.04 /bin/bash # 创建数据卷绑定到到新建容器，新建容器中会创建 /data 数据卷 bash-4.1# ls -ld /data/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /data/
bash-4.1# df -Th
Filesystem    Type    Size  Used Avail Use% Mounted on
... ...
              ext4     91G  4.6G   82G   6% /data
```

创建的数据卷可以通过docker inspect获取宿主机对应路径

```
$ sudo docker inspect data
... ... "Volumes": { "/data": "/var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9" }, # 可以看到创建的数据卷宿主机路径 ... ...
```

或者直接指定获取

```
$ sudo docker inspect --format="{{ .Volumes }}" data
map[/data: /var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9]
```

**挂载宿主机目录为一个数据卷**

-v选项除了可以创建卷，也可以挂载当前主机的一个目录到容器中。

```
$ sudo docker run --name web -v /source/:/web -t -i ubuntu:18.04 /bin/bash
bash-4.1# ls -ld /web/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /web/
bash-4.1# df -Th
... ...
              ext4     91G  4.6G   82G   6% /web
bash-4.1# exit 
```

默认挂载卷是可读写的，可以在挂载时指定只读

```
$ sudo docker run --rm --name test -v /source/:/test:ro -t -i ubuntu:18.04 /bin/bash
```



##### 数据卷容器

如果你有一些持久性的数据并且想在容器间共享，或者想用在非持久性的容器上，最好的方法是创建一个数据卷容器，然后从此容器上挂载数据。

创建数据卷容器

```
$ sudo docker run -t -i -d -v /test --name test ubuntu:18.04 echo hello
```

使用--volumes-from选项在另一个容器中挂载 /test 卷。不管 test 容器是否运行，其它容器都可以挂载该容器数据卷，当然如果只是单独的数据卷是没必要运行容器的。

```
$ sudo docker run -t -i -d --volumes-from test --name test1 ubuntu:18.04 /bin/bash
```

添加另一个容器

```
$ sudo docker run -t -i -d --volumes-from test --name test2 ubuntu:18.04 /bin/bash
```

也可以继承其它挂载有 /test 卷的容器

```
$ sudo docker run -t -i -d --volumes-from test1 --name test3 ubuntu:18.04 /bin/bash
```

![3.png](https://i.loli.net/2019/07/31/5d40f6f62e16190175.png)

##### 备份、恢复或迁移数据卷

**备份**

```
$ sudo docker run --rm --volumes-from test -v $(pwd):/backup ubuntu:18.04 tar cvf /backup/test.tar /test
tar: Removing leading `/' from member names
/test/
/test/b
/test/d
/test/c
/test/a
```

启动一个新的容器并且从test容器中挂载卷，然后挂载当前目录到容器中为 backup，并备份 test 卷中所有的数据为 test.tar，执行完成之后删除容器--rm，此时备份就在当前的目录下，名为test.tar。

```
$ ls # 宿主机当前目录下产生了 test 卷的备份文件 test.tar test.tar
```

**恢复**

你可以恢复给同一个容器或者另外的容器，新建容器并解压备份文件到新的容器数据卷

```
$ sudo docker run -t -i -d -v /test --name test4 ubuntu:18.04  /bin/bash 
$ sudo docker run --rm --volumes-from test4 -v $(pwd):/backup ubuntu:19.04 tar xvf /backup/test.tar -C / 
# 恢复之前的文件到新建卷中，执行完后自动删除容器 test/ test/b test/d test/c test/a
```

##### 删除 Volumes

Volume 只有在下列情况下才能被删除：

- docker rm -v删除容器时添加了-v选项
- docker run --rm运行容器时添加了--rm选项

否则，会在/var/lib/docker/vfs/dir目录中遗留很多不明目录。

参考文档：

- [Managing Data in Containers](http://docs.docker.com/userguide/dockervolumes/#data-volumes)
- [深入理解Docker Volume（一）](http://dockerone.com/article/128)
- [深入理解Docker Volume（二）](http://dockerone.com/article/129)

#### 链接容器

docker 允许把多个容器连接在一起，相互交互信息。docker 链接会创建一种容器父子级别的关系，其中父容器可以看到其子容器提供的信息。

##### 容器命名

在创建容器时，如果不指定容器的名字，则默认会自动创建一个名字，这里推荐给容器命名：

- 给容器命名方便记忆，如命名运行 web 应用的容器为 web
- 为 docker 容器提供一个参考，允许方便其他容器调用，如把容器 web 链接到容器 db

可以通过--name选项给容器自定义命名：

```
$ sudo docker run -d -t -i --name test ubuntu:19.04 bash              
$ sudo docker  inspect --format="{{ .Nmae }}" test
/test
```

> 注：容器名称必须唯一，即你只能命名一个叫test的容器。如果你想复用容器名，则必须在创建新的容器前通过docker rm删除旧的容器或者创建容器时添加--rm选项。

##### 链接容器

链接允许容器间安全通信，使用--link选项创建链接。

```
$ sudo docker run -d --name db training/postgres
```

基于 training/postgres 镜像创建一个名为 db 的容器，然后下面创建一个叫做 web 的容器，并且将它与 db 相互连接在一起

```
$ sudo docker run -d -P --name web --link db:db training/webapp python app.py
```

--link <name or id\>:alias选项指定链接到的容器。

查看 web 容器的链接关系:

```
$ sudo docker inspect -f "{{ .HostConfig.Links }}" web
[/db:/web/db]
```

可以看到 web 容器被链接到 db 容器为/web/db，这允许 web 容器访问 db 容器的信息。

容器之间的链接实际做了什么？一个链接允许一个源容器提供信息访问给一个接收容器。在本例中，web 容器作为一个接收者，允许访问源容器 db 的相关服务信息。Docker 创建了一个安全隧道而不需要对外公开任何端口给外部容器，因此不需要在创建容器的时候添加-p或-P指定对外公开的端口，这也是链接容器的最大好处，本例为 PostgreSQL 数据库。

Docker 主要通过以下两个方式提供连接信息给接收容器：

- 环境变量
- 更新/etc/hosts文件

**环境变量**

当两个容器链接，Docker 会在目标容器上设置一些环境变量，以获取源容器的相关信息。

首先，Docker 会在每个通过--link选项指定别名的目标容器上设置一个<alias\>_NAME环境变量。如果一个名为 web 的容器通过--link db:webdb被链接到一个名为 db 的数据库容器，那么 web 容器上会设置一个环境变量为WEBDB_NAME=/web/webdb.

以之前的为例，Docker 还会设置端口变量:

```
$ sudo docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432           
DB_PORT_5432_TCP=tcp://172.17.0.5:5432  # <name>_PORT_<port>_<protocol> 协议可以是 TCP 或 UDP
DB_PORT_5432_TCP_PROTO=tcp
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_ADDR=172.17.0.5
. . .
```

> 注：这些环境变量只设置给容器中的第一个进程，类似一些守护进程 (如 sshd ) 当他们派生 shells 时会清除这些变量

**更新/etc/hosts文件**

除了环境变量，Docker 会在目标容器上添加相关主机条目到/etc/hosts中，上例中就是 web 容器。

```
$ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  db
```

> /etc/host文件在源容器被重启之后会自动更新 IP 地址，而环境变量中的 IP 地址则不会自动更新的。

#### 容器互联

随着 Docker 网络的完善，强烈建议大家将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 `--link` 参数。

下面先创建一个新的 Docker 网络。

```bash
$ docker network create -d bridge my-net
```

`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于 Swarm mode

**连接容器**

运行一个容器并连接到新建的 `my-net` 网络

```bash
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```

打开新的终端，再运行一个容器并加入到 `my-net` 网络

```bash
$ docker run -it --rm --name busybox2 --network my-net busybox sh
```

再打开一个新的终端查看容器信息

```bash
$ docker container ls

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b47060aca56b        busybox             "sh"                11 minutes ago      Up 11 minutes                           busybox2
8720575823ec        busybox             "sh"                16 minutes ago      Up 16 minutes                           busybox1
```

下面通过 `ping` 来证明 `busybox1` 容器和 `busybox2` 容器建立了互联关系。

在 `busybox1` 容器输入以下命令

```bash
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms
```

用 ping 来测试连接 `busybox2` 容器，它会解析成 `172.19.0.3`。

同理在 `busybox2` 容器执行 `ping busybox1`，也会成功连接到。

```bash
/ # ping busybox1
PING busybox1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.064 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.143 ms
```

这样，`busybox1` 容器和 `busybox2` 容器建立了互联关系。

如果你有多个容器之间需要互相连接，推荐使用 Docker Compose。

后面会讲到。

#### 配置 DNS

如何自定义配置容器的主机名和 DNS 呢？秘诀就是 Docker 利用虚拟文件来挂载容器的 3 个相关配置文件。

在容器中使用 `mount` 命令可以看到挂载信息：

```bash
$ mount
/dev/disk/by-uuid/1fec...ebdf on /etc/hostname type ext4 ...
/dev/disk/by-uuid/1fec...ebdf on /etc/hosts type ext4 ...
tmpfs on /etc/resolv.conf type tmpfs ...
```

这种机制可以让宿主主机 DNS 信息发生更新后，所有 Docker 容器的 DNS 配置通过 `/etc/resolv.conf` 文件立刻得到更新。

配置全部容器的 DNS ，也可以在 `/etc/docker/daemon.json` 文件中增加以下内容来设置。

```json
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```

这样每次启动的容器 DNS 自动配置为 `114.114.114.114` 和 `8.8.8.8`。使用以下命令来证明其已经生效。

```bash
$ docker run -it --rm ubuntu:18.04  cat etc/resolv.conf

nameserver 114.114.114.114
nameserver 8.8.8.8
```

如果用户想要手动指定容器的配置，可以在使用 `docker run` 命令启动容器时加入如下参数：

`-h HOSTNAME` 或者 `--hostname=HOSTNAME` 设定容器的主机名，它会被写到容器内的 `/etc/hostname` 和 `/etc/hosts`。但它在容器外部看不到，既不会在 `docker container ls` 中显示，也不会在其他的容器的 `/etc/hosts` 看到。

`--dns=IP_ADDRESS` 添加 DNS 服务器到容器的 `/etc/resolv.conf` 中，让容器用这个服务器来解析所有不在 `/etc/hosts` 中的主机名。

`--dns-search=DOMAIN` 设定容器的搜索域，当设定搜索域为 `.example.com` 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 `host.example.com`。

> 注意：如果在容器启动时没有指定最后两个参数，Docker 会默认用主机上的 `/etc/resolv.conf` 来配置容器。

#### 高级网络配置

当 Docker 启动时，会自动在主机上创建一个 `docker0` 虚拟网桥，实际上是 Linux 的一个 bridge，可以理解为一个软件交换机。它会在挂载到它的网口之间进行转发。

同时，Docker 随机分配一个本地未占用的私有网段（在 [RFC1918](http://tools.ietf.org/html/rfc1918) 中定义）中的一个地址给 `docker0` 接口。比如典型的 `172.17.42.1`，掩码为 `255.255.0.0`。此后启动的容器内的网口也会自动分配一个同一网段（`172.17.0.0/16`）的地址。

当创建一个 Docker 容器的时候，同时会创建了一对 `veth pair` 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 `eth0`；另一端在本地并被挂载到 `docker0` 网桥，名称以 `veth` 开头（例如 `vethAQI2QT`）。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所有容器之间一个虚拟共享网络。

![2.png](https://i.loli.net/2019/08/01/5d4305e94722a26586.png)

接下来的部分将介绍在一些场景中，Docker 所有的网络定制配置。以及通过 Linux 命令来调整、补充、甚至替换 Docker 默认的网络配置。

##### 快速配置指南

下面是一个跟 Docker 网络相关的命令列表。

其中有些命令选项只有在 Docker 服务启动的时候才能配置，而且不能马上生效。

- `-b BRIDGE` 或 `--bridge=BRIDGE` 指定容器挂载的网桥
- `--bip=CIDR` 定制 docker0 的掩码
- `-H SOCKET...` 或 `--host=SOCKET...` Docker 服务端接收命令的通道
- `--icc=true|false` 是否支持容器之间进行通信
- `--ip-forward=true|false` 请看下文容器之间的通信
- `--iptables=true|false` 是否允许 Docker 添加 iptables 规则
- `--mtu=BYTES` 容器网络中的 MTU

下面2个命令选项既可以在启动服务时指定，也可以在启动容器时指定。在 Docker 服务启动的时候指定则会成为默认值，后面执行 `docker run` 时可以覆盖设置的默认值。

- `--dns=IP_ADDRESS...` 使用指定的DNS服务器
- `--dns-search=DOMAIN...` 指定DNS搜索域

最后这些选项只有在 `docker run` 执行时使用，因为它是针对容器的特性内容。

- `-h HOSTNAME` 或 `--hostname=HOSTNAME` 配置容器主机名
- `--link=CONTAINER_NAME:ALIAS` 添加到另一个容器的连接
- `--net=bridge|none|container:NAME_or_ID|host` 配置容器的桥接模式
- `-p SPEC` 或 `--publish=SPEC` 映射容器端口到宿主主机
- `-P or --publish-all=true|false` 映射容器所有端口到宿主主机

##### 容器访问控制

容器的访问控制，主要通过 Linux 上的 `iptables` 防火墙来进行管理和实现。`iptables` 是 Linux 上默认的防火墙软件，在大部分发行版中都自带。

1. 容器访问外部网络

   容器要想访问外部网络，需要本地系统的转发支持。在Linux 系统中，检查转发是否打开。

   ```bash
   $sysctl net.ipv4.ip_forward
   net.ipv4.ip_forward = 1
   ```

   如果为 0，说明没有开启转发，则需要手动打开。

   ```bash
   $sysctl -w net.ipv4.ip_forward=1
   ```

   如果在启动 Docker 服务的时候设定 `--ip-forward=true`, Docker 就会自动设定系统的 `ip_forward` 参数为 1。

2. 容器之间访问

   容器之间相互访问，需要两方面的支持。

   - 容器的网络拓扑是否已经互联。默认情况下，所有容器都会被连接到 `docker0` 网桥上。
   - 本地系统的防火墙软件 -- `iptables` 是否允许通过。

3. 访问所有端口

   当启动 Docker 服务时候，默认会添加一条转发策略到 iptables 的 FORWARD 链上。策略为通过（`ACCEPT`）还是禁止（`DROP`）取决于配置`--icc=true`（缺省值）还是 `--icc=false`。当然，如果手动指定 `--iptables=false` 则不会添加 `iptables` 规则。

   可见，默认情况下，不同容器之间是允许网络互通的。如果为了安全考虑，可以在 `/etc/default/docker` 文件中配置 `DOCKER_OPTS=--icc=false` 来禁止它。

4. 访问指定端口

   在通过 `-icc=false` 关闭网络访问后，还可以通过 `--link=CONTAINER_NAME:ALIAS` 选项来访问容器的开放端口。

   例如，在启动 Docker 服务时，可以同时使用 `icc=false --iptables=true` 参数来关闭允许相互的网络访问，并让 Docker 可以修改系统中的 `iptables` 规则。

   此时，系统中的 `iptables` 规则可能是类似

   ```bash
   $ sudo iptables -nL
   ...
   Chain FORWARD (policy ACCEPT)
   target     prot opt source               destination
   DROP       all  --  0.0.0.0/0            0.0.0.0/0
   ...
   ```

   之后，启动容器（`docker run`）时使用 `--link=CONTAINER_NAME:ALIAS` 选项。Docker 会在 `iptable` 中为 两个容器分别添加一条 `ACCEPT` 规则，允许相互访问开放的端口（取决于 `Dockerfile` 中的 `EXPOSE` 指令）。

   当添加了 `--link=CONTAINER_NAME:ALIAS` 选项后，添加了 `iptables` 规则。

   ```bash
   $ sudo iptables -nL
   ...
   Chain FORWARD (policy ACCEPT)
   target     prot opt source               destination
   ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
   ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
   DROP       all  --  0.0.0.0/0            0.0.0.0/0
   ```

   注意：`--link=CONTAINER_NAME:ALIAS` 中的 `CONTAINER_NAME` 目前必须是 Docker 分配的名字，或使用 `--name` 参数指定的名字。主机名则不会被识别。

##### 映射容器端口到宿主主机的实现

默认情况下，容器可以主动访问到外部网络的连接，但是外部网络无法访问到容器。

1. 容器访问外部实现

   容器所有到外部网络的连接，源地址都会被 NAT 成本地系统的 IP 地址。这是使用 `iptables` 的源地址伪装操作实现的。

   查看主机的 NAT 规则。

   ```bash
   $ sudo iptables -t nat -nL
   ...
   Chain POSTROUTING (policy ACCEPT)
   target     prot opt source               destination
   MASQUERADE  all  --  172.17.0.0/16       !172.17.0.0/16
   ...
   ```

   其中，上述规则将所有源地址在 `172.17.0.0/16` 网段，目标地址为其他网段（外部网络）的流量动态伪装为从系统网卡发出。MASQUERADE 跟传统 SNAT 的好处是它能动态从网卡获取地址。

2. 外部访问容器实现

   容器允许外部访问，可以在 `docker run` 时候通过 `-p` 或 `-P` 参数来启用。

   不管用那种办法，其实也是在本地的 `iptable` 的 nat 表中添加相应的规则。

   使用 `-P` 时：

   ```bash
   $ iptables -t nat -nL
   ...
   Chain DOCKER (2 references)
   target     prot opt source               destination
   DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:49153 to:172.17.0.2:80
   ```

   使用 `-p 80:80` 时：

   ```bash
   $ iptables -t nat -nL
   Chain DOCKER (2 references)
   target     prot opt source               destination
   DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
   ```

   注意：

   - 这里的规则映射了 `0.0.0.0`，意味着将接受主机来自所有接口的流量。用户可以通过 `-p IP:host_port:container_port` 或 `-p IP::port` 来指定允许访问容器的主机上的 IP、接口等，以制定更严格的规则。
   - 如果希望永久绑定到某个固定的 IP 地址，可以在 Docker 配置文件 `/etc/docker/daemon.json` 中添加如下内容。

   ```json
   {
     "ip": "0.0.0.0"
   }
   ```

##### 配置 docker0 网桥

Docker 服务默认会创建一个 `docker0` 网桥（其上有一个 `docker0` 内部接口），它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。

Docker 默认指定了 `docker0` 接口 的 IP 地址和子网掩码，让主机和容器之间可以通过网桥相互通信，它还给出了 MTU（接口允许接收的最大传输单元），通常是 1500 Bytes，或宿主主机网络路由上支持的默认值。这些值都可以在服务启动的时候进行配置。

- `--bip=CIDR` IP 地址加掩码格式，例如 192.168.1.5/24
- `--mtu=BYTES` 覆盖默认的 Docker mtu 配置

也可以在配置文件中配置 DOCKER_OPTS，然后重启服务。

由于目前 Docker 网桥是 Linux 网桥，用户可以使用 `brctl show` 来查看网桥和端口连接信息。

```bash
$ sudo brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.3a1d7362b4ee       no              veth65f9
                                             vethdda6
```

*注：`brctl` 命令在 Debian、Ubuntu 中可以使用 `sudo apt-get install bridge-utils` 来安装。

每次创建一个新容器的时候，Docker 从可用的地址段中选择一个空闲的 IP 地址分配给容器的 eth0 端口。使用本地主机上 `docker0` 接口的 IP 作为所有容器的默认网关。

```bash
$ sudo docker run -i -t --rm base /bin/bash
$ ip addr show eth0
24: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 32:6f:e0:35:57:91 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::306f:e0ff:fe35:5791/64 scope link
       valid_lft forever preferred_lft forever
$ ip route
default via 172.17.42.1 dev eth0
172.17.0.0/16 dev eth0  proto kernel  scope link  src 172.17.0.3
```

##### 自定义网桥

除了默认的 `docker0` 网桥，用户也可以指定网桥来连接各个容器。

在启动 Docker 服务的时候，使用 `-b BRIDGE`或`--bridge=BRIDGE` 来指定使用的网桥。

如果服务已经运行，那需要先停止服务，并删除旧的网桥。

```bash
$ sudo systemctl stop docker
$ sudo ip link set dev docker0 down
$ sudo brctl delbr docker0
```

然后创建一个网桥 `bridge0`。

```bash
$ sudo brctl addbr bridge0
$ sudo ip addr add 192.168.5.1/24 dev bridge0
$ sudo ip link set dev bridge0 up
```

查看确认网桥创建并启动。

```bash
$ ip addr show bridge0
4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global bridge0
       valid_lft forever preferred_lft forever
```

在 Docker 配置文件 `/etc/docker/daemon.json` 中添加如下内容，即可将 Docker 默认桥接到创建的网桥上。

```json
{
  "bridge": "bridge0",
}
```

启动 Docker 服务。

新建一个容器，可以看到它已经桥接到了 `bridge0` 上。

可以继续用 `brctl show` 命令查看桥接的信息。另外，在容器中可以使用 `ip addr` 和 `ip route` 命令来查看 IP 地址配置和路由信息。

##### 工具和示例

在介绍自定义网络拓扑之前，你可能会对一些外部工具和例子感兴趣：

- pipework

  Jérôme Petazzoni 编写了一个叫 [pipework](https://github.com/jpetazzo/pipework) 的 shell 脚本，可以帮助用户在比较复杂的场景中完成容器的连接。

- playground

  Brandon Rhodes 创建了一个提供完整的 Docker 容器网络拓扑管理的 [Python库](https://github.com/brandon-rhodes/fopnp/tree/m/playground)，包括路由、NAT 防火墙；以及一些提供 HTTP, SMTP, POP, IMAP, Telnet, SSH, FTP 的服务器。

##### 编辑网络配置文件

Docker 1.2.0 开始支持在运行中的容器里编辑 `/etc/hosts`, `/etc/hostname` 和 `/etc/resolve.conf` 文件。

但是这些修改是临时的，只在运行的容器中保留，容器终止或重启后并不会被保存下来。也不会被 `docker commit` 提交。

##### 示例：创建一个点到点连接

默认情况下，Docker 会将所有容器连接到由 `docker0` 提供的虚拟子网中。

用户有时候需要两个容器之间可以直连通信，而不用通过主机网桥进行桥接。

解决办法很简单：创建一对 `peer` 接口，分别放到两个容器中，配置成点到点链路类型即可。

首先启动 2 个容器：

```bash
$ docker run -i -t --rm --net=none base /bin/bash
root@1f1f4c1f931a:/#
$ docker run -i -t --rm --net=none base /bin/bash
root@12e343489d2f:/#
```

找到进程号，然后创建网络命名空间的跟踪文件。

```bash
$ docker inspect -f '{{.State.Pid}}' 1f1f4c1f931a
2989
$ docker inspect -f '{{.State.Pid}}' 12e343489d2f
3004
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/2989/ns/net /var/run/netns/2989
$ sudo ln -s /proc/3004/ns/net /var/run/netns/3004
```

创建一对 `peer` 接口，然后配置路由

```bash
$ sudo ip link add A type veth peer name B

$ sudo ip link set A netns 2989
$ sudo ip netns exec 2989 ip addr add 10.1.1.1/32 dev A
$ sudo ip netns exec 2989 ip link set A up
$ sudo ip netns exec 2989 ip route add 10.1.1.2/32 dev A

$ sudo ip link set B netns 3004
$ sudo ip netns exec 3004 ip addr add 10.1.1.2/32 dev B
$ sudo ip netns exec 3004 ip link set B up
$ sudo ip netns exec 3004 ip route add 10.1.1.1/32 dev B
```

现在这 2 个容器就可以相互 ping 通，并成功建立连接。点到点链路不需要子网和子网掩码。

此外，也可以不指定 `--net=none` 来创建点到点链路。这样容器还可以通过原先的网络来通信。

利用类似的办法，可以创建一个只跟主机通信的容器。但是一般情况下，更推荐使用 `--icc=false` 来关闭容器之间的通信。

#### 实践

1. https://idig8.com/2018/07/27/docker-chuji-04/