---
title: Docker 入门
date: 2019-07-05 10:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

### 安装

[教程](https://www.cntofu.com/book/139/index.html)

安装需要的包

<!--more-->

```
sudo apt install apt-transport-https ca-certificates software-properties-common curl
```

添加 GPG 密钥，并添加 Docker-ce 软件源，这里还是以中国科技大学的 Docker-ce 源为例

```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
$(lsb_release -cs) stable"
```

添加成功后更新软件包缓存

```
sudo apt update
```

安装 Docker-ce

```
sudo apt install docker-ce
```

设置开机自启动并启动 Docker-ce（安装成功后默认已设置并启动，可忽略）

```
sudo systemctl enable docker
sudo systemctl start docker
```

测试运行

```
sudo docker run hello-world
```

**或者使用官方脚本自动安装**

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

#### 镜像源修改

docker默认的源为国外官方源，下载速度较慢，可改为国内，加速

方案一

修改或新增 /etc/docker/daemon.json

```bash
# vi /etc/docker/daemon.json

{
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}

systemctl restart docker.service
```

方案二

修改或新增 /etc/sysconfig/docker，在OPTIONS变量后追加参数  `--registry-mirror=https://pee6w651.mirror.aliyuncs.com`

```bash
# vi /etc/sysconfig/docker

OPTIONS='--selinux-enabled --log-driver=journald --registry-mirror=https://pee6w651.mirror.aliyuncs.com'
```

Docker国内源说明：

- Docker 官方中国区：<https://registry.docker-cn.com>
- 网易：<http://hub-mirror.c.163.com>
- 中国科技大学:<https://docker.mirrors.ustc.edu.cn>
- 阿里云:<https://pee6w651.mirror.aliyuncs.com>

**解决docker每次都需要输入sudo的权限问题**

```bash
$ sudo groupadd docker
$ sudo gpasswd -a ${USER} docker
$ newgrp docker
$ sudo service docker restart
#测试添加用户组（可选）
$ docker run hello-world
```

#### 常用命令

关键字：

- 镜像 images
- 镜像名 image_name
- 镜像id image_id
- 容器 container
- 容器名 con_name
- 容器id con_id



![UTOOLS1572002191472.png](https://i.loli.net/2019/10/25/9zhuRZdMsUEeoQl.png)

从公网拉取一个镜像

```
docker pull images_name
```

查看已有的docker镜像

```
docker images
```

查看帮助

```
docker command --help
```

查看镜像列表

```
docker search nginx
```

启动一个容器

```
#基于hello-world镜像启动一个容器，如果本地没有镜像会从公网拉取过来，这次做为测试用
docker run hello-world
```

导出镜像

```
docker save -o image_name.tar image_name
docker load -i  image_name.tar
```

删除镜像

```
docker rmi image_name -f
```

启动一个容器

```
docker run --name=con_name images
--name  #设置容器名
```

基于创建好的容器自定义docker镜像

```
docker commit -m "con_name" con_id image_name
```

创建一个容器的同时进入这个容器

```
docker run -it --name=con_name images
-it     #在启动之后进入这个容器
```

创建一个容器，放入后台运行，把物理机80端口映射到容器的80端口

```
docker run -d -p 81:80 image_name
#-p 参数说明
-p hostPort:containerPort
-p ip:hostPort:containerPort
-p ip::containerPort
-p hostPort:containerPort:udp
```

看容器的端口映射情况

```
docker port con_id
```

查看正在运行的容器

```
docker ps 
```

查看所有的容器

```
docker ps -a
```

动态查看容器日志

```
docker logs -f con_name
```

进入容器

```
docker attach con_name
```

退出容器

```
# 方法一
exit
# 方法二
ctrl+p && ctrl+q (一起按，注意顺序，退出后容器依然保持启动状态)
```

删除容器

```
docker rm  con_name
#强制删除需要加-f，不加-f不能删除正在运行中的容器，非常危险，最好不用
```

查看docker网络

```
[root@docker ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3f91f2097286        bridge              bridge              local
d7675dbd247c        docker_gwbridge     bridge              local
5b36c7e947fd        host                host                local
ims6qkpikafu        ingress             overlay             swarm
85ba10e7ef79        none                null                local
```

创建一个docker网络my-docker

```
docker network create -d bridge \
--subnet=192.168.0.0/24 \
--gateway=192.168.0.100 \
--ip-range=192.168.0.0/24 \
my-docker
```

利用刚才创建的网络启动一个容器

```
#docker run --network=my-docker --ip=192.168.0.5 -itd --name=con_name -h lb01 image_name
--network   #指定容器网络
--ip        #设定容器ip地址
-h          #给容器设置主机名
```

查看容器pid

```
#方法一：
docker top con_name

#方法二：
docker inspect --format "{{.State.Pid}}" con_name
```

运行dockerfile并给dockerfile创建的镜像建立名字

```
docker build -t mysql:3.6.34 `pwd`
```

mariadb容器启动前需先设置密码方法

```
docker run -d -P -e MYSQL_ROOT_PASSWORD=password  img_id
```

docker修改镜像名

```
docker tag imageid name:tag
```

进入docker容器脚本

```
[root@docker ~]# cat nsenter.sh 
PID=`docker inspect --format "{{.State.Pid}}" $1`
nsenter -t $PID -u --mount -i -n -p
```

创建一个网络

```
docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_net2
```

将容器添加到my_net2网络 connect

```
docker network connect my_net2 oldboy1
```

docker日志模块

使用filebeat收集日志

```
{
  "registry-mirrors": ["https://56px195b.mirror.aliyuncs.com"],
  "cluster-store":"consul://192.168.56.13:8500",
  "cluster-advertise": "192.168.56.11:2375",
  "log-driver": "fluentd",
  "log-opts": {
        "fluentd-address":"192.168.56.13:24224",
        "tag":"linux-node1.example.com"
        }
}
```

##### Docker 拷贝文件

1. 从容器里面拷文件到宿主机？

   答：在宿主机里面执行以下命令

   ```
   docker cp 容器名：要拷贝的文件在容器里面的路径       要拷贝到宿主机的相应路径
   ```

   示例： 假设容器名为testtomcat,要从容器里面拷贝的文件路为：`/usr/local/tomcat/webapps/test/js/test.js`,现在要将test.js从容器里面拷到宿主机的/opt路径下面，那么命令应该怎么写呢？

   答案：在宿主机上面执行命令

   ```
   docker cp testtomcat：/usr/local/tomcat/webapps/test/js/test.js /opt
   ```

2. 从宿主机拷文件到容器里面

   答：在宿主机里面执行如下命令

   ```
   docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径
   ```

   示例：假设容器名为testtomcat,现在要将宿主机/opt/test.js文件拷贝到容器里面                                                               的`/usr/local/tomcat/webapps/test/js`路径下面，那么命令该怎么写呢？

   答案：在宿主机上面执行如下命令

   ```
   docker cp /opt/test.js testtomcat：/usr/local/tomcat/webapps/test/js
   ```

##### Docker清理空间

用了一段时间Docker后，会发现它占用了不少硬盘空间。还好Docker 引入了解决方法，它提供了简单的命令System来查看/清理Docker使用的磁盘空间。

Docker 的内置 CLI 指令docker system df，可用于查询镜像（Images）、容器（Containers）和本地卷（Local Volumes）等空间使用大户的空间占用情况。

```
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              15                  0                   7.402GB             7.402GB (100%)
Containers          3                   0                   0B                  0B
Local Volumes       2                   0                   81.38MB             81.38MB (100%)
Build Cache         0                   0                   0B                  0B
```

可以进一步通过-v参数查看空间占用细节

**空间清理**

| 不同状态 | 已使用镜像（used image）                   | 未引用镜像（unreferenced image） | 悬空镜像（dangling image）              |
| :------- | :----------------------------------------- | :------------------------------- | :-------------------------------------- |
| 镜像含义 | 指所有已被容器（包括已停止的）关联的镜像。 | 没有被分配或使用在容器中的镜像   | 未配置任何 Tag （也就无法被引用）的镜像 |

Docker内置自动清理：

通过 Docker 内置的 CLI 指令docker system prune来进行自动空间清理。

```
$ docker system prune --help

Usage: docker system prune [OPTIONS]

Remove unused data

Options:
 -a, --all       Remove all unused images not just dangling ones
   --filter filter  Provide filter values (e.g. 'label=<key>=<value>')
 -f, --force      Do not prompt for confirmation
   --volumes     Prune volumes
```

docker system prune 自动清理说明：

该指令默认会清除所有如下资源：

1. 已停止的容器（container）
2. 未被任何容器所使用的卷（volume）
3. 未被任何容器所关联的网络（network）
4. 所有悬空镜像（image）。

该指令默认只会清除悬空镜像，未被使用的镜像不会被删除。添加-a 或 --all参数后，可以一并清除所有未使用的镜像和悬空镜像。

可以添加-f 或 --force参数用以忽略相关告警确认信息。

```
$ docker system prune -a
```

#### 示例

##### 运行kalilinux

**首先是pull镜像**

```bash
$ docker pull kalilinux/kali-linux-docker
```

**运行容器**

```
$ docker run --privileged=true -p 3316:22  -p 20022:22 -p 20080:80 -p 20006:3306 -p 20079:6379 -p 20717:27017 -tid --name kali kalilinux/kali-linux-docker 
```

参数: –name 指定生成的容器的名称 
-i: 以交互模式运行容器，保证容器中STDIN是开启的。通常与 -t 同时使用； 
-t: 为容器重新分配一个伪tty终端，通常与 -i 同时使用； 
-d: 后台运行容器，并返回容器ID； 
-p:可以指定要映射的IP和端口，但是在一个指定端口上只可以绑定一个容器。支持的格式有 hostPort:containerPort、ip:hostPort:containerPort、 ip::containerPort。 

kalilinux/kali-linux-docker 则是镜像名称，镜像ID也可以的。

**查看运行情况**

```
$ docker ps
CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS              PORTS                  NAMES
3088cc48de3c        kalilinux/kali-linux-docker   "bash"              10 seconds ago      Up 8 seconds        0.0.0.0:3316->22/tcp   kali

```

**进入容器终端安装ssh服务**

```
$  docker exec -t -i kali /bin/bash
```

换源：

```
$ apt install vim
$ vim /etc/apt/sources.list

#阿里云
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb http://mirrors.aliyun.com/kali-security kali-rolling/updates main contrib non-free

#中科大 
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib  
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib 


#清华大学
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free

#浙大
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free

#东软大学
deb http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib
deb-src http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib

#官方源
deb http://http.kali.org/kali kali-rolling main non-free contrib
deb-src http://http.kali.org/kali kali-rolling main non-free contrib
```

安装ssh服务（默认为root权限用户）：

- 先执行更新

  ```
   apt-get update
   apt install -y  vim git python python3 net-tools
  ```

- 安装ssh-client命令

  ```
   apt-get install openssh-client
  ```

- 安装ssh-server命令

  ```
   apt-get install openssh-server
  ```

- 安装完成以后，先启动服务

  安装完成以后，先启动服务

  ```
   /etc/init.d/ssh start
  ```

  启动后，可以通过“ps -e|grep ssh”查看是否正确启动。 

- 最后编辑sshd_config文件

  ```
  vim  /etc/ssh/sshd_config
  ```

  将PermitRootLogin xxxx-password 改为 PermitRootLogin yes。 

- 重启ssh服务

  ```
  service ssh restart
  ```

- 设置ssh密码

  设置ssh密码

  ```
  passwd root
  ```

- 查看容器ip:

  查看容器ip:

  ```
  apt-get install net-tools
  ```

- 然后输入ip -4 addr，查看容器ip。

- 安装kali工具集

  ```
  apt install -y kali-linux-all
  ```

**ssh链接容器中的系统**

退出容器终端exit。 

在宿主机中，docker ps -a 命令中可以查看到容器的端口。在宿主机中，可以使用localhost进行登录，也可以使用刚才ifconfig查看的ip进行登录

```
$ ssh root@localhost -p 3316
```

如果要从外部进行登录，就要使用宿主机的ip，端口不变。这样我们的任务就完成了。接下来，就可以自由玩耍服务器了。 x.x.x.x就是宿主机的ip地址。

```
ssh root@x.x.x.x -p 3316
```

**提交更改的镜像**

刚刚我们对容器进行了安装，可以如果删掉话。每次都要重新执行上面的操作，所以可以对镜像进行提交保存。

```
docker commit  [改变了的容器的ID]  [REPOSITORY:TAG] 
```

如

```
$ docker commit 3088cc48de3c pingxin/kaliubuntu
$ docker images pingxin/kaliubuntu
```

然后以后使用新镜像运行就可以了。

最后打包了一个kalilinux到阿里云的hub管理平台上。

**ssh登录centos**

上述步骤太过繁琐，可以通过Dockerfile来创建自己的images，然后在运行

```dockerfile
FROM centos:latest
MAINTAINER "pingxin" "m13839441583@163.com"

RUN yum install -y openssh-server

#修改root用户密码
#用以下命令修改密码时，密码中最好不要包含特殊字符，如"!"，否则可能会失败；
RUN /bin/echo "123456" | passwd --stdin root

#生成密钥
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key \
    && ssh-keygen -t rsa -f /etc/ssh/ssh_host_ecdsa_key \
    && ssh-keygen -t rsa -f /etc/ssh/ssh_host_ed25519_key

#修改配置信息
RUN /bin/sed -i 's/.*session.*required.*pam_loginuid.so.*/session optional pam_loginuid.so/g' /etc/pam.d/sshd \
    && /bin/sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config \
    && /bin/sed -i "s/#UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config


EXPOSE 22

CMD ["/usr/sbin/sshd","-D"]
```

然后运行

```shell
$ docker build -t pingxin/centos-ssh:laset  .
$ docker run --name ssh -d  pingxin/centos-ssh:latest
```

#### docker daemon远程连接设置

Docker为C/S架构，服务端为docker daemon，客户端为docker.service.支持本地unix socket域套接字通信与远程socket通信。默认为本地unix socket通信，要支持远程客户端访问需要做如下设置（仅用于测试，生产环境开启会极大增加不安全性：由于开了监听端口，任何人可以通过远程连接到docker daemon服务器进行操作）。

docker和服务端守护进程通信有两种方式一个就是docker自己的client(docker cli)还有一个就是用户可以自己写程序，调用docker的remote-api来和docker的守护进程通信。还有要知道的是docker客户端和服务端是使用socket连接方式的，有三种socket连接方式，一种是直接连接本地的socket文件`unix:///var/run/docker/sock`，第二种是`tcp://host:prot`，第三种是`fd://socketfd`，默认docker都是直接连接本地的socket文件，而且还支持`fd://socketfd`，如果我们要支持远程连接就必须要加上tcp连接方式

docker建议在`/etc/docker/daemon.json`文件中修改docker启动参数

默认docker不创建这个文件，所以我们要创建并且添加上我们要的配置，不要的可以不加，比如我的就是

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com","http://hub-mirror.c.163.com"],
  "labels": ["name=docker-server"],
  "hosts": [
        "tcp://0.0.0.0:2376",
        "unix:///var/run/docker.sock"
    ]
}
```

第一行是仓库地址，第二行是给docker-daemon做一个标签，第三行hosts就是连接方式，现在docker同时支持两种连接方式了

然后，通过 `dockerd` 启动守护进程

接着我们直接在客户端centos机器上连接服务端的机器，输入下面命令 `docker -H tcp://192.168.0.83:2376 info` 

-H后面就是指定连接的服务端地址 info表示查看服务端daemon的信息

如果你不想每次都输入-H参数，那么你可以在客户端机器加上下面的环境变量
`export DOCKER_HOST="tcp://192.168.0.83:2376"`

如果你想连接本地那么改回环境变量即可

```
export DOCKER_HOST=""
```

如果你还想查看更多关于连接的问题你可以查看下面这篇博客
[远程连接docker daemon，Docker Remote API](https://deepzz.com/post/dockerd-and-docker-remote-api.html)

上面的前提是开放接口或者关闭防火墙

### 图形化管理工具

#### Portainer

[Portainer](https://www.portainer.io)是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。

**优点**
  （1）支持容器管理、镜像管理(导入、导出)。

  （2）轻量级，消耗资源少。

  （3）基于docker api，安全性高，可指定docker api端口，支持TLS证书认证。

  （4）支持权限分配。

  （5）支持集群。

  （6）github上目前持续维护更新。

##### portainer安装

采用docker方式安装。下载汉化包：链接: https://pan.baidu.com/s/1jlD4M5g9IkqCPtfc27wS3Q 提取码: aa8m 

汉化包解压后我放在`/opt/xauto/portainer/public`,装载到容器的`/public`目录。

```shell
$ docker pull portainer/portainer
$ docker volume create portainer_data
$ docker run -d --name portainer --restart=always -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock  -v portainer_data:/data portainer/portainer
```

至此，安装完毕，访问http://127.0.0.1:9000/ 体验吧。

#### Kitematic

Kitematic是一个 Docker GUI 工具，它可以更快速、更简单的运行Docker，现在已经支持 Ubuntu、Mac 和 Windows。Kitematic 目前在 [Github](https://github.com/docker/kitematic) 上开源，而它也早在 2015 年就已经被 Docker 收购。Kitematic 完全自动化了 Docker 安装和设置过程，并提供了一个直观的图形用户接口（GUI）来运行 Docker。通过 GUI 你可以非常容易的创建、运行和管理你的容器，不需要使用命令行或者是在 Docker CLI 和 GUI之间来回切换；同时也可以方便的修改环境变量、查看日志以及配置数据卷等。

### 相关命令

检查docker.service运行后，使用docker -h 查看命令。

Usage:	docker [OPTIONS] COMMAND

命令有管理命令和命令，管理命令是对不同模块的管理，命令则是常用命令。

可以通过docker COMMAND --help查看用法和选项。

#### 管理命令

- builder     Manage builds
- config      Manage Docker configs
- container   Manage containers
- context     Manage contexts
- engine      Manage the docker engine
- image       Manage images
- network     Manage networks
- node        Manage Swarm nodes
- plugin      Manage plugins
- secret      Manage Docker secrets
- service     Manage services
- stack       Manage Docker stacks
- swarm       Manage Swarm
- system      Manage Docker
- trust       Manage trust on Docker images
- volume      Manage volumes

#### 常用命令

**docker command**

```bash
$ docker --help  # docker 命令帮助
Commands:
    attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
    build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
    commit    Create a new image from a container's changes # 提交当前容器为新的镜像
    cp        Copy files/folders from the containers filesystem to the host path
              # 从容器中拷贝指定文件或者目录到宿主机中
    create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
    diff      Inspect changes on a container's filesystem   # 查看 docker 容器变化
    events    Get real time events from the server          # 从 docker 服务获取容器实时事件
    exec      Run a command in an existing container        # 在已存在的容器上运行命令
    export    Stream the contents of a container as a tar archive   
              # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
    history   Show the history of an image                  # 展示一个镜像形成历史
    images    List images                                   # 列出系统当前镜像
    import    Create a new filesystem image from the contents of a tarball  
              # 从tar包中的内容创建一个新的文件系统映像[对应 export]
    info      Display system-wide information               # 显示系统相关信息
    inspect   Return low-level information on a container   # 查看容器详细信息
    kill      Kill a running container                      # kill 指定 docker 容器
    load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
    login     Register or Login to the docker registry server   
              # 注册或者登陆一个 docker 源服务器
    logout    Log out from a Docker registry server         # 从当前 Docker registry 退出
    logs      Fetch the logs of a container                 # 输出当前容器日志信息
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT
              # 查看映射端口对应的容器内部源端口
    pause     Pause all processes within a container        # 暂停容器
    ps        List containers                               # 列出容器列表
    pull      Pull an image or a repository from the docker registry server
              # 从docker镜像源服务器拉取指定镜像或者库镜像
    push      Push an image or a repository to the docker registry server
              # 推送指定镜像或者库镜像至docker源服务器
    restart   Restart a running container                   # 重启运行的容器
    rm        Remove one or more containers                 # 移除一个或者多个容器
    rmi       Remove one or more images                 
              # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
    run       Run a command in a new container
              # 创建一个新的容器并运行一个命令
    save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
    search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
    start     Start a stopped containers                    # 启动容器
    stop      Stop a running containers                     # 停止容器
    tag       Tag an image into a repository                # 给源中镜像打标签
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    unpause   Unpause a paused container                    # 取消暂停容器
    version   Show the docker version information           # 查看 docker 版本号
    wait      Block until a container stops, then print its exit code   
              # 截取容器停止时的退出状态值
Run 'docker COMMAND --help' for more information on a command.
```

 **docker option**

```bash
Usage of docker:
  --api-enable-cors=false                Enable CORS headers in the remote API                      # 远程 API 中开启 CORS 头
  -b, --bridge=""                        Attach containers to a pre-existing network bridge         # 桥接网络
                                           use 'none' to disable container networking
  --bip=""                               Use this CIDR notation address for the network bridge's IP, not compatible with -b
                                         # 和 -b 选项不兼容，具体没有测试过
  -d, --daemon=false                     Enable daemon mode                                         # daemon 模式
  -D, --debug=false                      Enable debug mode                                          # debug 模式
  --dns=[]                               Force docker to use specific DNS servers                   # 强制 docker 使用指定 dns 服务器
  --dns-search=[]                        Force Docker to use specific DNS search domains            # 强制 docker 使用指定 dns 搜索域
  -e, --exec-driver="native"             Force the docker runtime to use a specific exec driver     # 强制 docker 运行时使用指定执行驱动器
  --fixed-cidr=""                        IPv4 subnet for fixed IPs (ex: 10.20.0.0/16)
                                           this subnet must be nested in the bridge subnet (which is defined by -b or --bip)
  -G, --group="docker"                   Group to assign the unix socket specified by -H when running in daemon mode
                                           use '' (the empty string) to disable setting of a group
  -g, --graph="/var/lib/docker"          Path to use as the root of the docker runtime              # 容器运行的根目录路径
  -H, --host=[]                          The socket(s) to bind to in daemon mode                    # daemon 模式下 docker 指定绑定方式[tcp or 本地 socket]
                                           specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.
  --icc=true                             Enable inter-container communication                       # 跨容器通信
  --insecure-registry=[]                 Enable insecure communication with specified registries (no certificate verification for HTTPS and enable HTTP fallback) (e.g., localhost:5000 or 10.20.0.0/16)
  --ip="0.0.0.0"                         Default IP address to use when binding container ports     # 指定监听地址，默认所有 ip
  --ip-forward=true                      Enable net.ipv4.ip_forward                                 # 开启转发
  --ip-masq=true                         Enable IP masquerading for bridge's IP range
  --iptables=true                        Enable Docker's addition of iptables rules                 # 添加对应 iptables 规则
  --mtu=0                                Set the containers network MTU                             # 设置网络 mtu
                                           if no value is provided: default to the default route MTU or 1500 if no default route is available
  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file                            # 指定 pid 文件位置
  --registry-mirror=[]                   Specify a preferred Docker registry mirror                  
  -s, --storage-driver=""                Force the docker runtime to use a specific storage driver  # 强制 docker 运行时使用指定存储驱动
  --selinux-enabled=false                Enable selinux support                                     # 开启 selinux 支持
  --storage-opt=[]                       Set storage driver options                                 # 设置存储驱动选项
  --tls=false                            Use TLS; implied by tls-verify flags                       # 开启 tls
  --tlscacert="/root/.docker/ca.pem"     Trust only remotes providing a certificate signed by the CA given here
  --tlscert="/root/.docker/cert.pem"     Path to TLS certificate file                               # tls 证书文件位置
  --tlskey="/root/.docker/key.pem"       Path to TLS key file                                       # tls key 文件位置
  --tlsverify=false                      Use TLS and verify the remote (daemon: verify client, client: verify daemon) # 使用 tls 并确认远程控制主机
  -v, --version=false                    Print version information and quit                         # 输出 docker 版本信息
```



#### 容器生命周期管理

- run：创建一个新的容器并运行一个命令

  ```
  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  ```

  OPTIONS说明：

  - **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
  - **-d:** 后台运行容器，并返回容器ID；
  - **-i:** 以交互模式运行容器，通常与 -t 同时使用；
  - **-P:** 随机端口映射，容器内部端口**随机**映射到主机的高端口
  - **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
  - **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
  - **--name="nginx-lb":** 为容器指定一个名称；
  - **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
  - **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
  - **-h "mars":** 指定容器的hostname；
  - **-e username="ritchie":** 设置环境变量；
  - **--env-file=[]:** 从指定文件读入环境变量；
  - **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
  - **-m :**设置容器使用内存最大值；
  - **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
  - **--link=[]:** 添加链接到另一个容器；
  - **--expose=[]:** 开放一个端口或一组端口；
  - **--volume , -v:**	绑定一个卷

  示例：

  ```bash
  #使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。
  $ docker run --name mynginx -d nginx:latest
  #使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。
  $ docker run -P -d nginx:latest
  #使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 8081 端口,主机的目录 /data 映射到容器的 /data。
  $ docker run -p 8081:80 -v /data:/data -d nginx:latest
  #绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。
  $ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
  #使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
  $ docker run -it nginx:latest /bin/bash
  ```

  

- start/stop/restart:启动/停止/重启一个或多个已经被停止的容器,参数是容器名

- kill:杀掉一个运行中的容器

  选项：

  -  -s :向容器发送一个信号，默认是KILL

  ```
  $ docker kill -s KILL mynginx
  ```

- rm：删除一个或多少容器

  选项：

  - **-f :**通过SIGKILL信号强制删除一个运行中的容器
  - **-l :**移除容器间的网络连接，而非容器本身
  - **-v :**-v 删除与容器关联的卷

  ```bash
  # 强制删除容器db01、db02
  $ docker rm -f db01 db02
  # 移除容器nginx01对容器db01的连接，连接名db
  $ docker rm -l db 
  # 删除容器nginx01,并删除容器挂载的数据卷
  $ docker rm -v nginx01
  ```

- pause/unpause:暂停/恢复容器中所有的进程。

- create:创建一个新的容器但不启动它,用法同 docker run

- exec:在运行的容器中执行命令

  OPTIONS说明：

  - **-d :**分离模式: 在后台运行
  - **-i :**即使没有附加也保持STDIN 打开
  - **-t :**分配一个伪终端

  ```bash
  # 在容器 mynginx 中开启一个交互模式的终端:
  $ docker exec -i -t  mynginx /bin/bash
  # 也可以通过 docker ps -a 命令查看已经在运行的容器，然后使用容器 ID 进入容器。
  #查看已经在运行的容器 ID：
  $ docker ps -a 
  #通过 exec 命令对指定的容器执行 bash:
  $ docker exec -it <容器id> /bin/bash
  
  ```


#### 容器操作

- ps：列出正在运行容器信息

  ```
  docker ps [OPTIONS]
  ```

  OPTIONS说明：

  - **-a :**显示所有的容器，包括未运行的。
  - **-f :**根据条件过滤显示的内容。
  - **--format :**指定返回值的模板文件。
  - **-l :**显示最近创建的容器。
  - **-n :**列出最近创建的n个容器。
  - **--no-trunc :**不截断输出。
  - **-q :**静默模式，只显示容器编号。
  - **-s :**显示总的文件大小。

  **根据条件过滤显示的内容**

  根据标签过滤

  ```
  $ docker run -d --name=test-nginx --label color=blue nginx
  $ docker ps --filter "label=color"
  $ docker ps --filter "label=color=blue"
  ```

  根据名称过滤

  ```
  $ docker ps --filter"name=test-nginx"
  ```

  根据状态过滤

  ```
  $ docker ps -a --filter 'exited=0'
  $ docker ps --filter status=running
  $ docker ps --filter status=paused
  ```

  根据镜像过滤

  ```
  #镜像名称
  $ docker ps --filter ancestor=nginx
  
  #镜像ID
  $ docker ps --filter ancestor=d0e008c6cf02
  ```

  根据启动顺序过滤

  ```
  $ docker ps -f before=9c3527ed70ce
  $ docker ps -f since=6e63f6ff38b0
  ```

- inspect：获取容器/镜像的元数据

  ```
  docker inspect [OPTIONS] NAME|ID [NAME|ID...]
  ```

  OPTIONS说明：

  - **-f :**指定返回值的模板文件。
  - **-s :**显示总的文件大小。
  - **--type :**为指定类型返回JSON。

  获取正在运行的容器mymysql的 IP。

  ```
  $ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mymysql
  ```

- top：查看容器中运行的进程信息，支持 ps 命令参数。

  ```
  docker top [OPTIONS] CONTAINER [ps OPTIONS]
  ```

  容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

  查看所有运行容器的进程信息。

  ```
  for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
  ```

- attach ：连接到正在运行中的容器。

  ```
  docker attach [OPTIONS] CONTAINER
  ```

  要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似）。

  官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

  容器mynginx将访问日志指到标准输出，连接到容器查看访问信息。

  ```
  $ docker attach --sig-proxy=false mynginx
  192.168.239.1 - - [10/Jul/2016:16:54:26 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
  ```

- events：从服务器获取实时事件

  ```
  docker events [OPTIONS]
  ```

  OPTIONS说明：

  - **-f ：**根据条件过滤事件；
  - **--since ：**从指定的时间戳后显示所有事件;
  - **--until ：**流水时间显示到指定的时间为止；

  显示docker 2016年7月1日后的所有事件。

  ```
  $ docker events  --since="1467302400"
  ```

- logs:获取容器的日志

  ```
  docker logs [OPTIONS] CONTAINER
  ```

  OPTIONS说明：

  - **-f :** 跟踪日志输出
  - **--since :**显示某个开始时间的所有日志
  - **-t :** 显示时间戳
  - **--tail :**仅列出最新N条容器日志

  跟踪查看容器mynginx的日志输出。

  ```
  $ docker logs -f mynginx
  ```

  查看容器mynginx从2016年7月1日后的最新10条日志。

  ```
  $ docker logs --since="2016-07-01" --tail=10 mynginx
  ```

- wait:阻塞运行直到容器停止，然后打印出它的退出代码。

- export:将文件系统作为一个tar归档文件导出到STDOUT。

- port:列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。

#### 容器rootfs命令

- commit:从容器创建一个新的镜像。
- cp:用于容器与主机之间的数据拷贝
- diff**:** 检查容器里文件结构的更改。

#### 镜像仓库

- login/logout:登陆/登出到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
- pull:从镜像仓库中拉取或者更新指定镜像
- push:将本地的镜像上传到镜像仓库,要先登陆到镜像仓库
- search: 从Docker Hub查找镜像

#### 本地镜像管理

- images:列出本地镜像。

  - **-a :**列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
  - **--digests :**显示镜像的摘要信息；
  - **-f :**显示满足条件的镜像；
  - **--format :**指定返回值的模板文件；
  - **--no-trunc :**显示完整的镜像信息；
  - **-q :**只显示镜像ID。

- rmi: 删除本地一个或多少镜像。

  - **-f :**强制删除；
  - **--no-prune :**不移除该镜像的过程镜像，默认移除；

- tag:标记本地镜像，将其归入某一仓库。

  ```
  docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
  ```

- build:命令用于使用 Dockerfile 创建镜像。

- history:查看指定镜像的创建历史。

- save:将指定镜像保存成 tar 归档文件。

- load:导入使用 docker save 命令导出的镜像。

- import: 从归档文件中创建镜像。

  docker 容器导入导出有两种方法：

  一种是使用 save 和 load 命令

  使用例子如下：

  ```
  docker save ubuntu:load>/root/ubuntu.tar
  docker load<ubuntu.tar
  ```

  一种是使用 export 和 import 命令

  使用例子如下：

  ```
  docker export 98ca36> ubuntu.tar
  cat ubuntu.tar | sudo docker import - ubuntu:import
  ```

- info显示 Docker 系统信息，包括镜像和容器数。

#### docker image 管理命令

Usage: docker image *COMMAND*

COMMANDS：

| 指令    | 描述                                                 |
| ------- | ---------------------------------------------------- |
| ls      | 列出本机镜像                                         |
| build   | 构建镜像来自Dockerfile                               |
| history | 查看镜像历史                                         |
| inspect | 显示一个或多个镜像详细信息                           |
| pull    | 从镜像仓库拉取镜像文件                               |
| push    | 推送本地镜像到仓库                                   |
| rm      | 移除一个或多个本地镜像文件                           |
| prune   | 移除未使用的镜像，没有被标记或未被任何容器应用的镜像 |
| tag     | 创建一个引用源镜像标记目标镜像                       |
| export  | 导出*容器*文件系统到tar归档文件                      |
| import  | 导入*容器*文件系统到tar归档文件创建镜像              |
| save    | 保存一个或多个镜像文件到一个tar归档文件              |
| load    | 加载镜像文件来自tar归档或标准输入                    |

#### docker container 管理命令

**Usage: docker container COMMAND**
COMMANDS：

| 指令    | 描述                                         |
| ------- | -------------------------------------------- |
| attach  | 附加本地标准输入、输出和错误到一个运行的容器 |
| commit  | 创建一个新景象来自一个容器                   |
| cp      | 拷贝文件/文件夹到一个容器                    |
| create  |                                              |
| diff    |                                              |
| exec    | 在运行容器中执行命令                         |
| export  |                                              |
| inspect | 显示一个或多个容器的详细信息                 |
| kill    |                                              |
| logs    | 获取一个容器日志                             |
| ls      | 列出容器                                     |
| pause   |                                              |
| port    | 列出或指定容器端口映射                       |
| prune   |                                              |
| rename  |                                              |
| restart |                                              |
| rm      | 删除一个或多个容器                           |
| run     |                                              |
| start   | 启动容器                                     |
| stats   | 显示容器资源使用统计                         |
| stop    | 停止容器                                     |
| top     | 显示一个容器运行的进程                       |
| unpause |                                              |
| update  | 更新一个或多个容器配置                       |
| wait    |                                              |

**docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]**

COMMANDS：

| 指令                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| -i，--interactive           | 交互式                                                       |
| -t，--tty                   | 分配一个伪终端                                               |
| -d，--detach                | 运行容器到后台                                               |
| -a,--attach list            | 附加到运行的容器                                             |
| --dns list                  | 设置DNS服务器                                                |
| -e，--env list              | 设置环境变量                                                 |
| --env-file list             | 从文件中读取环境变量                                         |
| -p，--publish list          | 发布指定的容器和宿主机之间端口映射关系                       |
| -P，--publish-all           | 发布容器所有EXPOSE的端口到宿主机随机端口                     |
| -h，--hostname-all          | 设置容器主机名                                               |
| --ip string                 | 指定容器IP，只能用于自定义网络                               |
| --link list                 | 添加连接到另一个容器                                         |
| --network                   | 连接容器到一个网络                                           |
| --mount mount               | 挂载宿主机分区到容器                                         |
| -v，--volume list           | 挂载宿主机目录到容器                                         |
| --restart string            | 容器退出时重启策略，默认no [always、on-failure]              |
| --add-host list             | 添加其他主机到容器中/etc/hosts                               |
| -m，--memory                | 容器可以使用的最大内存                                       |
| --memory-swap               | 允许交换到磁盘的内存量                                       |
| --memory-swappiness=<0-100> | 容器使用SWAP分区交换的百分比（0-100，默认为-1）              |
| --memory-reservation        | 内存软限制，Docker检测主机容器争用或内存不足时所激活的软限制，使用此选项，值必须设置低于--memory，以使其优先 |
| --oom-kill-disable          | 当宿主机内存不足时，内核会杀死容器中的进程。建议设置了-memory选项再禁用0M，如果没有设置，主机可能会耗尽内存 |
| --cpus                      | 限制容器可以使用多少可用的cpu资源                            |
| --cpuset-cpus               | 限制容器可以使用特定的cpu                                    |
| cpu-shares                  | 此值设置为大于或小于默认1024值，以增加或减小容器的权重，并使其可以访问主机cpu周期的更大或更小比例 |

更多命令[参考](https://docs.docker.com/engine/reference/commandline/docker/)

### 配置文件

#### daemon.js

docker安装后默认没有daemon.json这个配置文件，需要进行手动创建。配置文件的默认路径：/etc/docker/daemon.json

一般情况，配置文件 daemon.json中配置的项目参数，在启动参数中同样适用，有些可能不一样（具体可以查看官方文档），但需要注意的一点，配置文件中如果已经有某个配置项，则无法在启动参数中增加，会出现冲突的错误。

如果在daemon.json文件中进行配置，需要docker版本高于1.12.6(在这个版本上不生效，1.13.1以上是生效的)

参数 
daemon.json文件可配置的参数表，我们在配置的过程中，只需要设置我们需要的参数即可，不必全部写出来。详细参考官网。

官方的配置地址：https://docs.docker.com/engine/reference/commandline/dockerd/#/configuration-reloading。

官方的配置地址：https://docs.docker.com/engine/reference/commandline/dockerd/#options

官方的配置地址：https://docs.docker.com/engine/reference/commandline/dockerd/#/linux-configuration-file

```js
{
"api-cors-header":"",
"authorization-plugins":[],
"bip": "",
"bridge":"",
"cgroup-parent":"",
"cluster-store":"",
"cluster-store-opts":{},
"cluster-advertise":"",
"debug": true, #启用debug的模式，启用后，可以看到很多的启动信息。默认false
"default-gateway":"",
"default-gateway-v6":"",
"default-runtime":"runc",
"default-ulimits":{},
"disable-legacy-registry":false,
"dns": ["192.168.1.1"], # 设定容器DNS的地址，在容器的 /etc/resolv.conf文件中可查看。
"dns-opts": [], # 容器 /etc/resolv.conf 文件，其他设置
"dns-search": [], # 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的 主机时，DNS不仅搜索host，还会搜索host.example.com 。
#注意：如果不设置， Docker 会默认用主机上的 /etc/resolv.conf 来配置容器。
 
"exec-opts": [],
"exec-root":"",
"fixed-cidr":"",
"fixed-cidr-v6":"",
"graph":"/var/lib/docker", ＃已废弃，使用data-root代替,这个主要看docker的版本
"data-root":"/var/lib/docker", ＃Docker运行时使用的根路径,根路径下的内容稍后介绍，默认/var/lib/docker
"group": "", #Unix套接字的属组,仅指/var/run/docker.sock
"hosts": [], #设置容器hosts
"icc": false,
"insecure-registries": [], #配置docker的私库地址
"ip":"0.0.0.0",
"iptables": false,
"ipv6": false,
"ip-forward": false, #默认true, 启用 net.ipv4.ip_forward ,进入容器后使用 sysctl -a | grepnet.ipv4.ip_forward 查看
 
"ip-masq":false,
"labels":["nodeName=node-121"], # docker主机的标签，很实用的功能,例如定义：–label nodeName=host-121
 
"live-restore": true,
"log-driver":"",
"log-level":"",
"log-opts": {},
"max-concurrent-downloads":3,
"max-concurrent-uploads":5,
"mtu": 0,
"oom-score-adjust":-500,
"pidfile": "", #Docker守护进程的PID文件
"raw-logs": false,
"registry-mirrors":["xxxx"], #镜像加速的地址，增加后在 docker info中可查看。
"runtimes": {
"runc": {
"path": "runc"
},
"custom": {
"path":"/usr/local/bin/my-runc-replacement",
"runtimeArgs": [
"--debug"
]
}
},
"selinux-enabled": false, #默认 false，启用selinux支持
 
"storage-driver":"",
"storage-opts": [],
"swarm-default-advertise-addr":"",
"tls": true, #默认 false, 启动TLS认证开关
"tlscacert": "", #默认 ~/.docker/ca.pem，通过CA认证过的的certificate文件路径
"tlscert": "", #默认 ~/.docker/cert.pem ，TLS的certificate文件路径
"tlskey": "", #默认~/.docker/key.pem，TLS的key文件路径
"tlsverify": true, #默认false，使用TLS并做后台进程与客户端通讯的验证
"userland-proxy":false,
"userns-remap":""
}
```

修改完成后reload配置文件

```bash
sudo systemctl daemon-reload
```

重启docker服务

```
sudo systemctl restart docker.service
```

查看服务

```
sudo docker info
```

#### docker在centos7中运行systemctl命令

怎么样才能在docker中运行systemctl命令呢，经过多次踩坑终于找到办法。

首先，systemctl是需要docker容器运行时，拥有系统真正的root权限。即在docker run命令式要加上 `--privileged=true`

网上说，大约在0.6版，privileged被引入docker。使用该参数，container内的root拥有真正的root权限。否则，container内的root只是外部的一个普通用户权限。privileged启动的容器，可以看到很多host上的设备，并且可以执行mount。甚至允许你在docker容器中启动docker容器。systemctl就需要如此的权限，不然在容器中运行systemctl命令时，会报无权限的错误：

```
Failed to get D-Bus connection: Operation not permitted；
```


其次，如果想在容器启动时，启动systemctl，那么有两种方法。

1.在dockerfile中加入：

```
CMD ["/usr/sbin/init"]
```

此处命令的意思是，在容器启动时，运行/usr/sbin/init目录下的脚本，主要是启动dbus-daemon。

2. 在启动容器的时候，运行/usr/sbin/init，即在docker run 命令最后，加上/usr/sbin/init。例：

```
docker run --privileged=true  -p 20022:22 -p 20080:80 -p 20006:3306 -p 20079:6379 -p 42717:27017 -tid --name centos centos:latest /usr/sbin/init
```

 

### 参考

1. [Docker最全教程——从理论到实战（二）](https://www.cnblogs.com/codelove/p/10036608.html)