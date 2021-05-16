---
title: Docker 三剑客--Swarm
date: 2019-07-08 19:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

### 三剑客

**Docker Machine**

Docker Machine是一个简化安装Docker环境的工具，主要作用是创建和管理docker主机。一般公司云服务器毕竟少，个人觉得Docker Machine在实际使用中用处不大。直接在云服务器上安装即可。

**docker-compose**

这个和下面要介绍的docker-swarm用处较大，compose是服务编排，即事先安排好服务的启动，依赖，创建服务的数量等等。主要作用在于服务的安排和部署上。dcoker-compose技术，就是通过一个.yml配置文件，将所有的容器的部署方法、文件映射、容器连接等等一系列的配置写在一个配置文件里。便利了复杂的多服务的部署。

**docker-swarm**

服务的集群。创建docker服务集群。通过暴露的简单的几条api命令实现集群的创建和使用。与docker-compose配合使用。docker-swarm关注的更多的是服务的查询，实际使用中不需要我们手动通过docker-swarm命令创建服务，创建服务交给docker-compose。

文档：<https://docs.docker.com/reference/>

<!--more-->

### Swarm

在Docker的官方文档当中，我们可以看到在Docker 1.12及更高版本中，Swarm模式与Docker Engine集成。那么Dokcer Swarm到底是个什么？

Docker Swarm是Docker官方的三剑客项目之一，提供Docker容器集群服务，是Docker官方对容器云生态镜像支持的核心方案。它是Docker公司推出的官方容器集群平台，基于Go语言实现，代码在https://github.com/docker/swarm。

Docker Swarm是原生支持Docker集群管理的工具。它可以把多个Docker主机组成的系统转换为单一的虚拟Docker主机，使得容器可以组成跨主机的子网网络。

在很多台机器上部署Docker，组成一个Docker集群，并把整个集群的资源抽象成资源池，使用者部署Docker应用的时候，只需要将应用交给Swarm，Swarm会根据整个集群资源的使用情况来分配资源给部署的Docker应用，可以将这个集群的资源利用率达到最大。

使用Docker CLI创建群集，将应用程序服务部署到群集，并管理群体行为。其主要的目的就是更好的帮助用户管理多个Docker Engine，方便用户使用，像使用Docker Engine一样使用容器集群服务。

**集群**

集群是一组协同工作的服务实体（可理解为服务器），用以提供比单一服务实体更具扩展性与可用性的服务平台。在客户端看来，一个集群就像是一个服务实体，但事实上集群由一组服务实体组成。与单一服务实体相比较，集群提供了以下两个关键特性：

- 可扩展性——集群的性能不限于单一的服务实体，新的服务实体可以动态地加入到集群，从而增强集群的性能。

- 高可用性——集群通过服务实体冗余使客户端免于轻易遇到out of service的警告。在集群中，同样的服务可以由多个服务实体提供。如果一个服务实体失败了，另一个服务实体会接管失败的服务实体。集群提供的从一个出错的服务实体恢复到另一个服务实体的功能增强了应用的可用性。

##### Docker集群服务概念

`Swarm` 是使用 [`SwarmKit`](https://github.com/docker/swarmkit/) 构建的 Docker 引擎内置（原生）的集群管理和编排工具。

使用 `Swarm` 集群之前需要了解以下几个概念。

**节点**

运行 Docker 的主机可以主动初始化一个 `Swarm` 集群或者加入一个已存在的 `Swarm` 集群，这样这个运行 Docker 的主机就成为一个 `Swarm` 集群的节点 (`node`) 。

节点分为管理 (`manager`) 节点和工作 (`worker`) 节点。

管理节点用于 `Swarm` 集群的管理，`docker swarm` 命令基本只能在管理节点执行（节点退出集群命令 `docker swarm leave` 可以在工作节点执行）。一个 `Swarm` 集群可以有多个管理节点，但只有一个管理节点可以成为 `leader`，`leader` 通过 `raft` 协议实现。

工作节点是任务执行节点，管理节点将服务 (`service`) 下发至工作节点执行。管理节点默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。

来自 Docker 官网的这张图片形象的展示了集群中管理节点与工作节点的关系。

![1.png](https://i.loli.net/2019/08/02/5d4313f2b142766482.png)

##### 服务和任务

任务 （`Task`）是 `Swarm` 中的最小的调度单位，目前来说就是一个单一的容器。

服务 （`Services`） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：

- `replicated services` 按照一定规则在各个工作节点上运行指定个数的任务。
- `global services` 每个工作节点上运行一个任务

两种模式通过 `docker service create` 的 `--mode` 参数指定。

来自 Docker 官网的这张图片形象的展示了容器、任务、服务的关系。

![2.png](https://i.loli.net/2019/08/02/5d4313f2bdca374934.png)

总结一下： 

swarm以节点（node）的方式组织集群（cluster）；同时每个节点上面可以部署一个或者多个服务（service），每个服务又可以包括一个或者多个容器（container）。

Swarm的架构图如下： 

![1.jpeg](https://i.loli.net/2019/08/01/5d4241806892114684.jpeg)

#### 基本命令

docker swarm init 创建Swarm，同时让swarm-manager 成为 manager node。 执行该命令后，显示添加worker node和manager node 需要执行的命令。 –advertise-addr 指定与其他node通信的地址。

```
docker swarm init --advertise-addr [ip]
```

查看swarm worker的连接令牌

```
docker swarm join-token worker
```

查看swarm manager的连接令牌

```
docker swarm join-token manager
```

加入docker swarm集群，这里的token和ip更换为具体需求下的即可;如果忘记可以用docker swarm join-token worker查看。

```
docker swarm join --token SWMTKN-1-1next4gw4m7yx3j6q1f4z4ooimvp3pc1ahsnnwifu5w1h6b247-f10wphfpnyyunzphr1qp4kew1 192.168.1.245:2377
```

查看集群中的节点

```
docker node ls
```

将节点升级为manager

```
docker node promote work-node1
```

将节点降级为worker

```
docker node demote work-node1
```

只能删除down状态的节点，如何要删除active状态的，需要加上–force

```
docker node rm --force worker-manager
```

离开swarm集群

```
docker swarm leave --force
```

查看服务列表

```
docker service ls
```

查看具体服务信息，这里是web服务

```
docker service ps web
```

创建一个名称为web，副本为3，开放端口为80的nginx服务

```
docker service create --name web --replicas 3 -p 80:80 nginx
```

删除一个服务

```
docker service rm web
```

查看集群中节点信息

```
docker node inspect work-node1 --pretty
```

查看服务的详细信息

```
docker service inspect --pretty <SERVICE-ID>/<SERVICE-NAME>
```

调度程序不会将新任务分配给节点。 
调度程序关闭任何现有任务并在可用节点上安排它们。

```
docker node update --availability drain work-node1
```

调度程序可以将任务分配给节点

```
docker node update --availability active work-node1
```

调度程序不向节点分配新任务，但是现有任务仍然保持运行

```
docker node update --availability pause work-node1
```

为指定的服务删除一个开放端口

```
docker service update --publish-rm 8080:80 web
```

为指定的服务添加一个开放端口

```
docker service update --publish-add 8080:80 web
```

回滚至之前版本

```
docker service update --rollback mysql
```

扩展或缩放服务中容器的数量

```
docker service scale web=5
```

更多命令[参考](https://docs.docker.com/engine/reference/commandline/docker/)

#### Swarm 集群

**初始化集群**

 `Docker Machine` 可以在数秒内创建一个虚拟的 Docker 主机，下面我们使用它来创建三个 Docker 主机，并加入到集群中。

我们首先创建一个 Docker 主机作为管理节点。

```bash
$ docker-machine create -d virtualbox manager
```

我们使用 `docker swarm init` 在管理节点初始化一个 `Swarm` 集群。

```bash
$ docker-machine ssh manager

docker@manager:~$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 `--advertise-addr` 指定 IP。

> 执行 `docker swarm init` 命令的节点自动成为管理节点。

**增加工作节点**

我们初始化了一个 `Swarm` 集群，拥有了一个管理节点，下面我们继续创建两个 Docker 主机作为工作节点，并加入到集群中。

```bash
$ docker-machine create -d virtualbox worker1

$ docker-machine ssh worker1

docker@worker1:~$ docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.    
$ docker-machine create -d virtualbox worker2

$ docker-machine ssh worker2

docker@worker1:~$ docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.    
```

> 注意：一些细心的读者可能通过 `docker-machine create --help` 查看到 `--swarm*` 等一系列参数。该参数是用于旧的 `Docker Swarm`,与本章所讲的 `Swarm mode` 没有关系。

**查看集群**

经过上边的两步，我们已经拥有了一个最小的 `Swarm` 集群，包含一个管理节点和两个工作节点。

在管理节点使用 `docker node ls` 查看集群。

```bash
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
dxn1zf6l61qsb1josjja83ngz *  manager   Ready   Active        Leader
```

#### 部署服务

我们使用 `docker service` 命令来管理 `Swarm` 集群中的服务，该命令只能在管理节点运行。

**新建服务**

现在我们在上一节创建的 `Swarm` 集群中运行一个名为 `nginx` 服务。

```bash
$ docker service create --replicas 3 -p 80:80 --name nginx nginx:1.13.7-alpine
```

现在我们使用浏览器，输入任意节点 IP ,即可看到 nginx 默认页面。

**查看服务**

使用 `docker service ls` 来查看当前 `Swarm` 集群运行的服务。

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
kc57xffvhul5        nginx               replicated          3/3                 nginx:1.13.7-alpine   *:80->80/tcp
```

使用 `docker service ps` 来查看某个服务的详情。

```bash
$ docker service ps nginx
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
pjfzd39buzlt        nginx.1             nginx:1.13.7-alpine   swarm2              Running             Running about a minute ago
hy9eeivdxlaa        nginx.2             nginx:1.13.7-alpine   swarm1              Running             Running about a minute ago
36wmpiv7gmfo        nginx.3             nginx:1.13.7-alpine   swarm3              Running             Running about a minute ago
```

使用 `docker service logs` 来查看某个服务的日志。

```bash
$ docker service logs nginx
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 2017/11/25 02:10:30 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.4, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.102"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:26 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:27 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 2017/11/25 02:10:27 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.2, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.101"
```

**服务伸缩**

我们可以使用 `docker service scale` 对一个服务运行的容器数量进行伸缩。

当业务处于高峰期时，我们需要扩展服务运行的容器数量。

```bash
$ docker service scale nginx=5
```

当业务平稳时，我们需要减少服务运行的容器数量。

```bash
$ docker service scale nginx=2
```

**删除服务**

使用 `docker service rm` 来从 `Swarm` 集群移除某个服务。

```bash
$ docker service rm nginx
```

#### 在 Swarm 集群中使用 compose 文件

正如之前使用 `docker-compose.yml` 来一次配置、启动多个容器，在 `Swarm` 集群中也可以使用 `compose` 文件 （`docker-compose.yml`） 来配置、启动多个服务。

上一节中，我们使用 `docker service create` 一次只能部署一个服务，使用 `docker-compose.yml` 我们可以一次启动多个关联的服务。

我们以在 `Swarm` 集群中部署 `WordPress` 为例进行说明。

```yaml
version: "3"

services:
  wordpress:
    image: wordpress
    ports:
      - 80:80
    networks:
      - overlay
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    deploy:
      mode: replicated
      replicas: 3

  db:
    image: mysql
    networks:
       - overlay
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    deploy:
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

volumes:
  db-data:
networks:
  overlay:
```

在 `Swarm` 集群管理节点新建该文件，其中的 `visualizer` 服务提供一个可视化页面，我们可以从浏览器中很直观的查看集群中各个服务的运行节点。

在 `Swarm` 集群中使用 `docker-compose.yml` 我们用 `docker stack` 命令，下面我们对该命令进行详细讲解。

**部署服务**

部署服务使用 `docker stack deploy`，其中 `-c` 参数指定 compose 文件名。

```bash
$ docker stack deploy -c docker-compose.yml wordpress
```

现在我们打开浏览器输入 `任一节点IP:8080` 即可看到各节点运行状态。

在浏览器新的标签页输入 `任一节点IP` 即可看到 `WordPress` 安装界面，安装完成之后，输入 `任一节点IP` 即可看到 `WordPress` 页面。

**查看服务**

```bash
$ docker stack ls
NAME                SERVICES
wordpress           3
```

**移除服务**

要移除服务，使用 `docker stack down`

```bash
$ docker stack down wordpress
Removing service wordpress_db
Removing service wordpress_visualizer
Removing service wordpress_wordpress
Removing network wordpress_overlay
Removing network wordpress_default
```

该命令不会移除服务所使用的 `数据卷`，如果你想移除数据卷请使用 `docker volume rm`

#### 在 Swarm 集群中管理敏感数据

在动态的、大规模的分布式集群上，管理和分发 `密码`、`证书` 等敏感信息是极其重要的工作。传统的密钥分发方式（如密钥放入镜像中，设置环境变量，volume 动态挂载等）都存在着潜在的巨大的安全风险。

Docker 目前已经提供了 `secrets` 管理功能，用户可以在 Swarm 集群中安全地管理密码、密钥证书等敏感数据，并允许在多个 Docker 容器实例之间共享访问指定的敏感数据。

> 注意： `secret` 也可以在 `Docker Compose` 中使用。

我们可以用 `docker secret` 命令来管理敏感信息。接下来我们在上面章节中创建好的 Swarm 集群中介绍该命令的使用。

这里我们以在 Swarm 集群中部署 `mysql` 和 `wordpress` 服务为例。

**创建 secret**

我们使用 `docker secret create` 命令以管道符的形式创建 `secret`

```bash
$ openssl rand -base64 20 | docker secret create mysql_password -

$ openssl rand -base64 20 | docker secret create mysql_root_password -
```

**查看 secret**

使用 `docker secret ls` 命令来查看 `secret`

```bash
$ docker secret ls

ID                          NAME                  CREATED             UPDATED
l1vinzevzhj4goakjap5ya409   mysql_password        41 seconds ago      41 seconds ago
yvsczlx9votfw3l0nz5rlidig   mysql_root_password   12 seconds ago      12 seconds ago
```

**创建 MySQL 服务**

创建服务相关命令已经在前边章节进行了介绍，这里直接列出命令。

```bash
$ docker network create -d overlay mysql_private

$ docker service create \
     --name mysql \
     --replicas 1 \
     --network mysql_private \
     --mount type=volume,source=mydata,destination=/var/lib/mysql \
     --secret source=mysql_root_password,target=mysql_root_password \
     --secret source=mysql_password,target=mysql_password \
     -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
     -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
     -e MYSQL_USER="wordpress" \
     -e MYSQL_DATABASE="wordpress" \
     mysql:latest
```

如果你没有在 `target` 中显式的指定路径时，`secret` 默认通过 `tmpfs` 文件系统挂载到容器的 `/run/secrets` 目录中。

```bash
$ docker service create \
     --name wordpress \
     --replicas 1 \
     --network mysql_private \
     --publish target=30000,port=80 \
     --mount type=volume,source=wpdata,destination=/var/www/html \
     --secret source=mysql_password,target=wp_db_password,mode=0400 \
     -e WORDPRESS_DB_USER="wordpress" \
     -e WORDPRESS_DB_PASSWORD_FILE="/run/secrets/wp_db_password" \
     -e WORDPRESS_DB_HOST="mysql:3306" \
     -e WORDPRESS_DB_NAME="wordpress" \
     wordpress:latest
```

查看服务

```bash
$ docker service ls

ID            NAME   MODE        REPLICAS  IMAGE
wvnh0siktqr3  mysql      replicated  1/1       mysql:latest
nzt5xzae4n62  wordpress  replicated  1/1       wordpress:latest
```

现在浏览器访问 `IP:30000`，即可开始 `WordPress` 的安装与使用。

通过以上方法，我们没有像以前通过设置环境变量来设置 MySQL 密码， 而是采用 `docker secret` 来设置密码，防范了密码泄露的风险。

#### 在 Swarm 集群中管理配置数据

在动态的、大规模的分布式集群上，管理和分发配置文件也是很重要的工作。传统的配置文件分发方式（如配置文件放入镜像中，设置环境变量，volume 动态挂载等）都降低了镜像的通用性。

在 Docker 17.06 以上版本中，Docker 新增了 `docker config` 子命令来管理集群中的配置信息，以后你无需将配置文件放入镜像或挂载到容器中就可实现对服务的配置。

> 注意：`config` 仅能在 Swarm 集群中使用。

这里我们以在 Swarm 集群中部署 `redis` 服务为例。

**创建 config**

新建 `redis.conf` 文件

```bash
port 6380
```

此项配置 Redis 监听 `6380` 端口

我们使用 `docker config create` 命令创建 `config`

```bash
$ docker config create redis.conf redis.conf
```

**查看 config**

使用 `docker config ls` 命令来查看 `config`

```bash
$ docker config ls

ID                          NAME                CREATED             UPDATED
yod8fx8iiqtoo84jgwadp86yk   redis.conf          4 seconds ago       4 seconds ago
```

**创建 redis 服务**

```bash
$ docker service create \
     --name redis \
     # --config source=redis.conf,target=/etc/redis.conf \
     --config redis.conf \
     -p 6379:6380 \
     redis:latest \
     redis-server /redis.conf
```

如果你没有在 `target` 中显式的指定路径时，默认的 `redis.conf` 以 `tmpfs` 文件系统挂载到容器的 `/config.conf`。

经过测试，redis 可以正常使用。

以前我们通过监听主机目录来配置 Redis，就需要在集群的每个节点放置该文件，如果采用 `docker config` 来管理服务的配置信息，我们只需在集群中的管理节点创建 `config`，当部署服务时，集群会自动的将配置文件分发到运行服务的各个节点中，大大降低了配置信息的管理和分发难度。

#### SWarm mode 与滚动升级

在 部署服务 一节中我们使用 `nginx:1.13.7-alpine` 镜像部署了一个名为 `nginx` 的服务。

现在我们想要将 `NGINX` 版本升级到 `1.13.12`，那么在 Swarm mode 中如何升级服务呢？

你可能会想到，先停止原来的服务，再使用新镜像部署一个服务，不就完成服务的 “升级” 了吗。

这样做的弊端很明显，如果新部署的服务出现问题，原来的服务删除之后，很难恢复，那么在 Swarm mode 中到底该如何对服务进行滚动升级呢？

答案就是使用 `docker service update` 命令。

```bash
$ docker service update \
    --image nginx:1.13.12-alpine \
    nginx
```

以上命令使用 `--image` 选项更新了服务的镜像。当然我们也可以使用 `docker service update` 更新任意的配置。

`--secret-add` 选项可以增加一个密钥

`--secret-rm` 选项可以删除一个密钥

更多选项可以通过 `docker service update -h` 命令查看。

**服务回退**

现在假设我们发现 `nginx` 服务的镜像升级到 `nginx:1.13.12-alpine` 出现了一些问题，我们可以使用命令一键回退。

```bash
$ docker service rollback nginx
```

现在使用 `docker service ps` 命令查看 `nginx` 服务详情。

```bash
$ docker service ps nginx

ID                  NAME                IMAGE                  NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
rt677gop9d4x        nginx.1             nginx:1.13.7-alpine   VM-20-83-debian     Running             Running about a minute ago
d9pw13v59d00         \_ nginx.1         nginx:1.13.12-alpine  VM-20-83-debian     Shutdown            Shutdown 2 minutes ago
i7ynkbg6ybq5         \_ nginx.1         nginx:1.13.7-alpine   VM-20-83-debian     Shutdown            Shutdown 2 minutes ago
```

结果的输出详细记录了服务的部署、滚动升级、回退的过程。

#### Swarm实战

以下使用 `docker service ps <服务名>`或 `docker service ls`查看服务

1. 创建基本的集群环境

   接下来会创建三个节点的Swarm集群。swarm-manager 是 manager node，swarm-worker1 和 swarm-worker2 是 worker node，可以认为是worker-node1、worker-node2节点。 不能在同一主机上运行多个swarm node，因此，需要使用到虚拟机。
   swarm-manager、swarm-worker1和swarm-worker2均为Linux系统服务器，系统为Ubuntu 18.04。 

   先在swarm-manager使用docker swarm init命令

   ```
   docker swarm init --advertise-addr 192.168.2.65
   ```

   这里的ip为具体本机服务ip;初始化为swarm-manager节点，使用docker node ls 我们可以查看到创建的节点。

   然后我们在swarm-worker1、和swarm-worker2服务器下面分别输入刚才初始化成功后的提示消息，比如在swarm-worker2节点下输入下面命令，token由manager初始化时给出

   ```
   docker swarm join --token SWMTKN-1-6dhytzhsxmfw1pfxttw7g0gcy7xddkhz12cp29nsnsxj9b32g2-185ya5tj5xl3gwur5gj3kgio1 192.168.2.65:2377
   ```

   This node joined a swarm as a worker. 提示我们添加节点成功。

   当我们去swarm-manager节点进行查看时可以发现，出现了三个节点，其中的两个就是刚才我们添加的。

   当然我们也可以删除某个节点，使用docker node rm , 这个只能删除down状态的节点，如何要删除active状态的。需要加上–force。

   ```
   docker node rm --force sprkw7t8irlb4l3dtvwszpdjj
   ```

   至此，三节点的swarm集群就已经搭建好了，操作还是相当简单的。

2. 服务创建及伸缩

   在上面我们搭建好了集群环境。现在我们在manager节点上部署一个基本的nginx服务，服务数量为（副本数）2个，公开指定端口是8080映射容器80，使用nginx镜像。

   ```
   docker service create --replicas 2 --name nginx --publish 8080:80  nginx
   ```

   通过docker service ls可以查看当前swarm中的服务。 可以看到服务正在启动过程当中。

   稍等一会我们再去查看服务，并使用docker service ps nginx查看具体的nginx服务。可以发现，nginx服务已经启动起来了，并且在不同的节点里面运行起来了。这里主要在swarm-manager和swarm-worker1里面分别启动了一个容器。我们知道平时通常会运行多个实例。这样可以负载均衡，同时也能实现服务器高可用。比如现在我们想把nginx服务数目提升到5个，提高可用性，就可以使用如下命令：

   ```
   docker service scale nginx=5
   ```

   我们可以查看到启动了5个nginx服务，并且分布在不同的节点，从中可以看到swarm-manager节点里启动了1个，swarm-worker1和swarm-worker2分别启动了2个节点。

   我们可以去相应节点服务器，通过docker ps -a 查看，比如：swarm-manager节点，只有一个容器运行；swarm-worker1节点，有两个容器运行；swarm-worker2节点，有两个容器运行

   我们知道默认配置下manager node 也是 worker node，所以 swarm-manager 上也运行了副本。如果不希望在 manager 上运行 service，可以进行排空，就是将该节点上的容器排空。命令如下：

   ```
   docker node update --availability drain [NODE-ID]
   ```

   执行过后，我们再查看具体服务情况,我们可以看到在swarm-manager上的容器nginx状态是shutdown了，然后在swarm-worker2上启动了一个nginx服务。查看节点状态，docker node ls可以发现manager上的状态为drain。

   当然如果我们要减少服务数目，直接把scale的数据设置比当前启动的服务少即可。比如减少服务数到3个，如下命令：

   ```
   docker service scale nginx=3
   ```

   shutdown为先前排除manager节点里的容器，可以看到只有三个运行的服务。 

3. 服务之间的通信

   比如我们在创建多个服务之间要通信，某个服务不要求与外部通信（比如数据库等）。这里就用到了服务发现。

   我们就可以使用建立overlay网络，部署 service 到 overlay网络上去。

   比如新建dev网络环境：

   ```
   docker network create --driver overlay dev
   ```

   然后我们新建服务，就可以让服务部署在该网络上。比如下面我们新建nginx版本1.12.2的服务，副本为2，开放80端口，部署到我们新建的dev网络上。

   ```
   docker service create --replicas 2 --name nginx-dev --publish 9090:80  --network dev nginx:1.12.2
   ```

   我们可以看到服务正在启动，我们可以在不同的节点下看到dev网络有该容器，而相同网络下的服务就可以进行通信，ip直接是可以访问到的。

4. 服务镜像升级和回滚

   在前面我们部署了多个副本的服务，如果我们要升级某个服务该怎么办，如下，我们对刚刚dev网络环境下的nginx:1.12.2进行升级，升级到1.13.12版本。使用如下命令：

   ```
   docker service update --image nginx:1.13.12 nginx-dev
   ```

   可以看到服务状态，新版本运行起来了，旧版本被停止了。 

   Swarm 将按照如下步骤执行滚动更新：

   - 停止第一个副本。
   - 调度任务，选择 worker node。
   - 在 worker 上用新的镜像启动副本。
   - 如果副本（容器）运行成功，继续更新下一个副本；如果失败，暂停整个更新过程。

   默认配置下，Swarm 一次只更新一个副本，并且两个副本之间没有等待时间。我们可以通过 –update-parallelism 设置并行更新的副本数目，通过 –update-delay 指定滚动更新的间隔时间。

   比如执行如下命令：

   ```
   docker service update --replicas 6 --update-parallelism 2 --update-delay 1m30s nginx-dev
   ```

   service 增加到六个副本，每次更新两个副本，间隔时间一分半钟。

   Swarm 还有个回滚功能，如果更新后效果不理想，可以通过 –rollback 快速恢复到更新之前的状态

   ```
   docker service update --rollback nginx-dev
   ```

   可以看到服务已经回滚到先前的版本。到此docker中swarm的简单使用就到此结束，后面再会介绍Docker三剑客的其他内容。

### 参考

1.[Docker三剑客——Swarm](https://blog.csdn.net/Anumbrella/article/details/80369913)