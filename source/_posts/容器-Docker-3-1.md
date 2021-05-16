---
title: Docker 三剑客--Machine
date: 2019-07-08 16:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

### Machine

我们知道在多个集群服务环境下，安装管理Docker的容器，要使用的是Docker Swarm，而使用Docker Swarm的情况是在多个集群的服务器已经搭建好Docker环境的情况下进行。如果在多台服务环境下，并没有安装好Docker环境，想要快速搭建一套Docker主机集群，这里就要使用到Docker Machine。

<!--more-->

通过上面的基本简介，我们知道了Docker Machine是用来干啥的。即Docker Machine是负责使用Docker的第一步，让我们可以在多种平台上快速安装Docker环境，同时也支持多种平台，让用户可以在短时间内搭建一套Docker主机集群。

Docker Machine是Docker官方的开源项目，负责实现对Docker主机本身进行管理，由Go语言编写，开源地址为https://github.com/docker/machine。

通过Machine用户可以在本地任意指定被Machine管理的Docker主机，并对其进行操作。Docker Machine 的定位主要是“在本地或者云环境中创建Docker”。其基本功能为：

- 在指定的节点上安装Docker引擎，配置其为Docker主机
- 集中管理所有Docker主机

总之Docker Machine 是一个工具，它允许你在虚拟宿主机上安装 Docker Engine ，并使用 docker-machine 命令管理这些宿主机。你可以使用 Machine 在你本地的 Mac 或 Windows box、公司网络、数据中心、或像 AWS 或 Digital Ocean 这样的云提供商上创建 Docker 宿主机。

使用 docker-machine 命令，你可以启动、审查、停止和重新启动托管的宿主机、升级 Docker 客户端和守护程序、并配置 Docker 客户端与你的宿主机通信。

在上面的内容中我们提到了Docker Engine（引擎），这里既然提及到了，那么我们就来介绍一下Docker Engine。

**Docker Engine**

当人们说“Docker”时，这里通常指的就是Docker Engine。Docker Engine其实就是一个Client-Server架构的应用。主要如下图所示： 

![3.png](https://i.loli.net/2019/08/01/5d424bcbdabd295515.png)

从上图中我们可以看出Docker Engine主要包括以下几个部分：

- Docker Daemon — docker的守护进程，属于C/S中的server
- Docker REST API — docker daemon向外暴露的REST接口
- Docker CLI — docker向外暴露的命令行接口（Command Line API）

docker engine接受docker从CLI命令，例如 docker run 运行一个容器，docker ps可以列出运行容器，docker image ls 列出镜像，等等。

**Docker Engine与Docker Machine**

Docker Machine是一个供应和管理Docker化主机的工具（Docker Engine上的主机）。

![4.png](https://i.loli.net/2019/08/01/5d424d87df76e66136.png)

通常，在本地系统上安装Docker Machine，Docker Machine就会拥有自己的命令行客户端 docker-machine和Docker Engine客户端docker。

你可以使用Machine在一个或多个虚拟系统上安装Docker Engine。

这些虚拟系统可以是本地的（例如当您使用Machine在Mac或Windows上的VirtualBox中安装和运行Docker引擎）或远程（如当您使用Machine在云提供者上配置Docker化主机时）。这些通过Machine被Docker化的主机本身，有时也称为被管理的宿主机(“machines”)。

#### 安装

Docker Machine可以在多种操作系统平台上安装，包括Linux、Mac os已经Windows。

在使用Machine前，首先要先安装好Docker，如果没有安装，需要提前进行安装。

从命令行安装：

On OS X

```
$ curl -L https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine
```

On Linux

```
$ curl -L https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
    chmod +x /tmp/docker-machine &&
    sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

On Windows with git bash

```
$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"
```

也可以选择下载[发行版](https://github.com/docker/machine/releases)

#### 命令

Docker Machine中每个命令都有一系列的参数，可以通过下面的命令来查看具体用法：

```
docker-machine <COMMAND> -h
```

Docker Machine命令列表如下：

![](https://i.loli.net/2019/08/01/5d425d9e097f016499.png)

下面具体介绍一下部分命令的用法：

1. active

   格式： `docker-machine active [OPTIONS] [arg…]`

   查看当前激活状态的Docker主机。激活状态意味着当前的DOCKER_HOST环境变量指向该主机。 

2. config

   格式： `docker-machine config [OPTIONS] [arg…]`

   查看到激活Docker主机的连接信息。 

3. create

   格式： `docker-machine create [OPTIONS] [arg…]`

   创建一个Docker主机。 
   选项包括：

   –dirve，-d “none”指定驱动类型；
   –engine-install-url “`https://get.docker.com`” 配置Docker主机时候的安装URL;
   –engine-opt option 以键值对格式指定所创建Docker引擎的参数；
   –engine-insecure-registry option 以键值对格式指定所创建Docker引擎允许访问的不支持认证的注册仓库服务；
   –engine-registry-mirror option 指定使用注册仓库镜像；
   –engine-label option 为所创建的Docker引擎添加标签；
   –engine-storage-driver 存储后端驱动类型；
   –engine-env option 指定环境变量；
   –swarm 指定使用Swarm；
   –swarm-image “swarm:latest”使用Swarm时候采用的镜像；
   –swarm-master 配置集群作为Swarm集群的master节点；
   –swarm-discovery Swarm 集群的服务发现机制参数；
   –swarm-strategy “spread” Swarm默认调度策略；
   –swarm-opt option 任意传递给Swarm的参数；
   –swarm-host “tcp://0.0.0.0:3376” 指定地址将监听Swarm master节点请求；
   –swarm-addr 从指定地址发送广播加入Swarm集群服务。

   比如创建一个命名为think的主机，ip地址为47.98.40.244，指定代理镜像地址和驱动类型，如下：

   ```
   $ docker-machine create --driver generic --generic-ip-address=47.98.40.244 --engine-registry-mirror=https://registry.docker-cn.com think
   ```

4. env

   格式： docker-machine env [OPTIONS] [arg…]

   显示连接到某个主机需要的环境变量。

   比如查看think主机的环境变量，`docker-machine env think`。 

5. inspect

   格式： `docker-machine inspect [OPTIONS] [arg…]`

   以json格式输出指定Docker主机的详细信息。

   使用如下命令查看think主机的详细信息。

   ```
   docker-machine insepct think
   ```

6. ip

   获取指定Docker主机地址。查看think主机的ip地址。

   ```
   docker-machine ip think
   ```

7. kill

   直接杀死指定的Docker主机。 指定Docker主机会强行停止。

8. ls

   格式： `docker-machine ls [OPTIONS] [arg…]`

   列出所有管理的主机。

   可以通过–fileter来输入某些Docker主机，支持过滤器包括名称正则表达式、驱动类型、Swarm管理节点名称、状态等。

   还支持–quiet，-q减少无关输出信息。

**驱动**

通过 `-d` 选项可以选择支持的驱动类型。

- amazonec2
- azure
- digitalocean
- exoscale
- generic
- google
- hyperv
- none
- openstack
- rackspace
- softlayer
- virtualbox
- vmwarevcloudair
- vmwarefusion
- vmwarevsphere

第三方驱动请到 [第三方驱动列表](https://github.com/docker/docker.github.io/blob/master/machine/AVAILABLE_DRIVER_PLUGINS.md) 查看

#### 实战

接下来我们就进入Docker Machine的实战部分，我们知道Docker Machine通过多种后端驱动来管理不同的资源，包括虚拟机、本地主机和云平台等，通过–driver可以选择支持的驱动类型。在这里我们主要分两种情况来实际应用。

1. 本地主机

   这种情况驱动适合主机操作系统和SSH服务都已经安装好，需要部署Docker环境的要求。 
   在实验环境中，我们有三个运行Ubuntu系统的主机，其中有一个安装好了Docker环境，同时安装了Docker Machine,

   我们知道对于 Docker Machine 来说，术语 Machine 就是运行 docker daemon 的主机。“创建 Machine” 指的就是在主机上安装和部署 docker。

   首先我们使用docker-machine ls查看一下当前的machine，可以看到这里并没有machine。接下来我们创建第一个 `machine：think - 47.98.40.244`。

   创建 machine 要求能够无密码登录远程主机，所以需要先通过如下命令将 ssh key 拷贝到 39.108.119.87：

   ```
   ssh-copy-id 47.98.40.244
   ```

   如果在执行过程中报错没有.pub的文件的ID，则可能是没有公共的id_rsa.pub文件。执行如下命令，然后一路回车。

   ```
   ssh-keygen
   ```

   然后就会在~/.ssh目录下就会出现id_rsa.pub文件，接着重新执行上面的命令，输入服务器登录密码，就可以完成key添加的配置。

   在上面的准备工作完成后，就可以创建我们的machine，创建一个命名为think的machine。执行命令如下：

   ```
   docker-machine create --driver generic --generic-ip-address=47.98.40.244 --engine-registry-mirror=https://registry.docker-cn.com think
   ```

   这里是在Linux环境中部署，直接使用Generic驱动，如果在其他环境中部署docker环境，可以使用其他驱动，详情文档参考https://docs.docker.com/machine/drivers/。

   –driver generic 为指定驱动类型，–generic-ip-address 为指定目标系统的IP，–engine-registry-mirror 为指定镜像源地址（这里更改为国内镜像源，不然后面安装容器可能无法pull镜像下来），并命名为 think。

   Docker Machine执行操作的步骤：

   1. 通过SSH登录到远程主机上。
   2. 负责在远处主机上执行安装docker步骤。
   3. 拷贝证书。
   4. 配置 docker daemon。
   5. 启动 docker 服务。

   再次执行 `docker-machine ls` ，可以查看到具体的相关服务

   我们可以发现创建的machine。

   同时登陆到39.108.x.x服务器，我们可以查看到docker的运行环境以及安装完毕。 

   在/etc/systemd/system/docker.service.d目录下我们可以查看到具体的daemon配置文件，编辑配置文件

   在配置信息中：

   - -H tcp://0.0.0.0:2376 是使 docker daemon 接受远程连接的地址。
   - –tls* 对远程连接启用安全认证和加密的配置。

   同时我们也可以查看到hostname也已经设置为think。 

   使用同样的方法，我们给120.79.x.x服务器进行安装machine。执行如下命令：

   ```
   docker-machine create --driver generic --generic-ip-address=120.79.x.x  --engine-registry-mirror=https://registry.docker-cn.com dev
   ```

   在使用时，需要将ip地址更换为自己实践的服务主机地址即可。

   创建成功后使用`docker-machine ls` 可以查看已经部署好的环境，可以看到think、dev主机已经准备就绪了。

   目前为止两台服务器都已经部署好了Docker的环境。

   在上面我们通过machine部署好了Docker环境，接下来我们就使用docker-machine的命令对集群的docker环境进行管理。

   我们知道docker连接指定目标主机服务器，可以通过-H或者指定DOCKER_HOST来实现。比如连接39.108.119.87服务器，可以通过指`DOCKER_HOST=tcp://39.108.119.87:2376`，也可以`-H tcp://39.108.119.87:2376` 来实现访问。

   在machine中就比较方便了，使用docker-machine env think，就可以查看到think服务的配置环境。 

   然后我们要切换到think服务器进行操作，根据上面的提示，执行`eval $(docker-machine env think)`。 

   我们再次执行`docker-machine ls`，可以发现think主机active被激活了，同时使用`docker ps -a`可以看到think主机上暂时并没有容器的。

   然后我们所有的操作就可以像在think主机上进行操作是一样的，比如我们要运行一个nginx容器。

   ```
   docker run --name think-nginx -d -p 8080:80 nginx
   ```

   可以看到我们在think主机上运行了一个nginx容器。接着同样我们可以切换主机环境进入dev主机环境操作。 

   可以看到dev主机上并没有运行什么容器。

   除此之外Docker Machine还有很多其他方便管理集群使用的命令，这里大致介绍一下。

   查看某个docker daemon的配置信息：

   ```
   docker-machine config think
   ```

   还可以通过ssh命令进行连接，进入具体的主机中。比如进入think主机服务器当中，就可以使用`docker-machine ssh think`命令来连接。

   可以查看到我们在think主机上运行的nginx容器。

   使用docker-machine scp 我们还可以在不同 machine 之间拷贝文件，比如：

   ```
   docker-machine scp [machine1:][path] [machine2:[path]
   ```

   比如我们将think主机下tmp目录下的a.txt文件拷贝到dev主机下的tmp目录下。

   比如我们将think主机下tmp目录下的a.txt文件拷贝到dev主机下的tmp目录下。

   ```
   docker-machine scp think:/tmp/a.txt dev:/tmp
   ```

   可以看到我们执行了scp命令，再使用ssh连接dev主机，到tmp目录下可以发现a.txt 文件。

   docker-machine还有stop/start/restart 命令，通过stop/start/restart 我们可以控制docker主机的运行状态。

   Machine还支持更新操作，通过`docker-machine upgrade` 更新 machine 的 docker 到最新版本，同时支持批量执行。

   ```
   docker-machine upgrade think dev
   ```

   通过上面的一些列操作，在多主机环境下 Docker Machine 可以极大地提升我们的工作效率。接下来我们在讲解一下虚拟化平台使用Docker Machine。

2. 虚拟化平台

   可以通过virtualbox驱动支持本地（需要已安装virtualbox）启动一个虚拟机并配置为Docker主机：

   ```
   docker-machine create --driver virtualbox --engine-registry-mirror=https://registry.docker-cn.com vbox
   ```

   如果没有安装virtualbox会报错的，先安装virtualbox:[参考wiki](https://www.virtualbox.org/wiki/Linux_Downloads)或者[教程](https://ywnz.com/linuxyffq/3304.html)。

   接着我们再重新执行创建Machine命令。不出意外，将启动一个全新的虚拟机，并安装Docker引擎。

   由于服务器是阿里云ESC不支持虚拟机里虚拟机，所以作者这一步没有执行验证了。

   除了virtualbox驱动外，docker也还支持其他的虚拟化平台。

   后面的管理操作就与前面本地主机一致，这里就不再复述。

#### 在阿里云使用

参考：<https://blog.csdn.net/bwlab/article/details/79515045>

#### 参考

1. [Docker 三剑客之 Machine 项目](https://www.cntofu.com/book/139/machine/README.md)
2. [『中级篇』在linux/mac下通过Docker-Machine在阿里云上的使用（11）](https://idig8.com/2018/07/29/docker-zhongji-11/)