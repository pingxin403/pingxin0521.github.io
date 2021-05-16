---
title: Docker 三剑客--compose
date: 2019-07-08 12:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

### compose

#### 什么是DockerCompose？

Compose项目是Docker官方的开源项目，负责实现对Docker容器集群的快速编排。它是一个定义和运行多容器的docker应用工具。使用compose，你能通过YMAL文件配置你自己的服务，然后通过一个命令，你能使用配置文件创建和运行所有的服务。

<!--more-->

Compose是一个定位“定义和运行多个Docker容器应用的工具”，其前身是Fig，目前使用的Compose仍然兼容Fig格式的模板文件。

Compose的代码主要使用Python编写，其开源地址为：<https://github.com/docker/compose>。

compose文档:<https://docs.docker.com/compose/compose-file/>

我们知道在Docker中构建自定义的镜像是通过使用Dockerfile模板文件来实现的，从而可以让用户很方便定义一个单独的应用容器。而Compose使用的模板文件就是一个YAML格式文件，它允许用户通过一个docker-compose.yml来定义一组相关联的应用容器为一个项目(project)。

> 注意：Fig时代支持的配置文件名为fig.yml以及fig.yaml；为了兼容遗留的Fig化配置，目前Compose支持的配置文件类型非常丰富，主要有以下几种：fig.yaml、docker-compose.yml、docker-compose.yaml以及用户指定的配置文件路径。（可通过环境变量COMPOSE_FILE或-f参数自定义配置文件）
>

在Compose中有两个重要的概念：

- 服务（service）：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目（project）：由一组关联的应用容器组成的一个完成业务单元，在docker-compose.yml中定义。

以上可以理解为：

服务（service）就是在它下面可以定义应用需要的一些服务，代表配置文件中的每一项服务。每个服务都有自己的名字、使用的镜像、挂载的数据卷、所属的网络、依赖哪些其他服务等等，即以容器为粒度，用户需要Compose所完成的任务。

项目（project）代表用户需要完成的一个项目，即是Compose的一个配置文件可以解析为一个项目，即Compose通过分析指定配置文件，得出配置文件所需完成的所有容器管理与部署操作。

Compose的默认管理对象时项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

#### Docker Compose安装

环境：ubuntu 18.04

参考[官方](https://github.com/docker/compose/releases)

下载docker-compose 二进制文件

```
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-uname-s–uname -m > /usr/local/bin/docker-compose && chmod +x/usr/local/bin/docker-compose
```

如果使用前面文章的进行安装docker-ce，可以直接安装。

```
sudo apt install docker-compose
```

因为Compose是Python编写的，我们可以将其当做一个Python应用从pip源中安装。

当在本地环境安装，则执行：

```
sudo pip install docker-compose
```

#### 常用命令

在我们使用Compose前，可以通过执行`docker-compose --help|-h`来查看Compose基本命令用法。

我们也可以通过执行`docker-compose [COMMAND] --help`或者`docker-compose --help [COMMAND]`来查看某个具体的使用格式。

从上面的提示中我们可以知道Compose命令的基本的使用格式为：

```
docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
```

命令选项如下：

- -f，–file FILE指定使用的Compose模板文件，默认为docker-compose.yml，可以多次指定。
- -p，–project-name NAME指定项目名称，默认将使用所在目录名称作为项目名。
- -x-network-driver 使用Docker的可拔插网络后端特性（需要Docker 1.9 及以后版本）
- -x-network-driver DRIVER指定网络后端的驱动，默认为bridge（需要Docker 1.9 及以后版本）
- -verbose输出更多调试信息

- -v，–version打印版本并退出

Docker Compose常用命令列表如下：

![](https://i.loli.net/2019/08/01/5d4278fcf050867373.png)

1. build

   格式为：

   ```
   docker-compose build [options] [--build-arg key=val...] [SERVICE...]
   ```

   构建（重新构建）项目中的服务容器。

   选项包括：

   - –compress 通过gzip压缩构建上下环境
   - –force-rm 删除构建过程中的临时容器
   - –no-cache 构建镜像过程中不使用缓存
   - –pull 始终尝试通过拉取操作来获取更新版本的镜像
   - -m, --memory MEM 为构建的容器设置内存大小

   - –build-arg key=val 为服务设置build-time变量

2. help：获得一个命令的帮助。

3. kill

   格式为：

   ```
   docker-compose kill [options] [SERVICE...]
   ```


   通过发送SIGKILL信号来强制停止服务容器。
   支持通过-s参数来指定发送的信号，例如通过如下指令发送SIGINT信号：

   ```
   docker-compose kill -s SIGINT
   ```

4. config
   格式为：

   ```
   docker-compose config [options]
   ```


   验证并查看compose文件配置。

   选项包括：

   - –resolve-image-digests 将镜像标签标记为摘要
   - -q, --quiet 只验证配置，不输出。 当配置正确时，不输出任何内容，当文件配置错误，输出错误信息
   - –services 打印服务名，一行一个

   - –volumes 打印数据卷名，一行一个

5. create
   格式为：

   ```
   docker-compose create [options] [SERVICE...]
   ```


   为服务创建容器.只是单纯的create，还需要使用start启动compose。

   选项包括：

   - –force-recreate 重新创建容器，即使它的配置和镜像没有改变，不兼容–no-recreate参数
   - –no-recreate 如果容器已经存在，不需要重新创建. 不兼容–force-recreate参数
   - –no-build 不创建镜像，即使缺失

   - –build 创建容器前，生成镜像

6. down
   格式为：

   ```
   docker-compose down [options]
   ```


   停止和删除容器、网络、卷、镜像，这些内容是通过docker-compose up命令创建的. 默认值删除 容器 网络，可以通过指定 rmi 、volumes参数删除镜像和卷。

   选项包括：

   - –rmi type 删除镜像，类型必须是: ‘all’: 删除compose文件中定义的所以镜像；‘local’: 删除镜像名为空的镜像
   - -v, --volumes 删除已经在compose文件中定义的和匿名的附在容器上的数据卷

   - –remove-orphans 删除服务中没有在compose中定义的容器

7. exec
   格式为：

   ```
   docker exec [options] SERVICE COMMAND [ARGS...]
   ```

   与docker exec命令功能相同，可以通过service name登陆到容器中。

   与docker exec命令功能相同，可以通过service name登陆到容器中。

   选项包括：

   - -d 分离模式，后台运行命令.
   - –privileged 获取特权.
   - –user USER 指定运行的用户.
   - -T 禁用分配TTY. By default docker-compose exec分配 a TTY.

   - –index=index 当一个服务拥有多个容器时，可通过该参数登陆到该服务下的任何服务，例如：docker-compose exec --index=1 web /bin/bash ，web服务中包含多个容器

8. logs
   格式为：

   ```
   docker-compose logs [options] [SERVICE...]
   ```

   查看服务容器的输出。默认情况下，docker-compose将对不同的服务输出使用不同的颜色来区分。可以通过–no-color来关闭颜色。

9. pause
   格式为：

   ```
   docker-compose pause [SERVICE...]
   ```


   暂停一个服务容器。

10. port
    格式为：

    ```
    docker-compose port [options] SERVICE PRIVATE_PORT
    ```

    显示某个容器端口所映射的公共端口。

    选项包括：

    - –protocol=proto 指定端口协议，TCP（默认值）或者UDP
    - –index=index 如果同意服务存在多个容器，指定命令对象容器的序号（默认为1）

11. ps
    格式为：

    ```
    docker-compose ps [options] [SERVICE...]
    ```
    
    列出项目中目前的所有容器。
    
    选项包括：
    
    - -q 只打印容器的ID信息
    
12. pull

    格式为：

    ```
    docker-compose pull [options] [SERVICE...]
    ```

    拉取服务依赖的镜像。

    选项包括：

    - –ignore-pull-failures 忽略拉取镜像过程中的错误
    - –parallel 多个镜像同时拉取
    - –quiet 拉取镜像过程中不打印进度信息

13. push
    格式为：

    ```
    docker-compose push [options] [SERVICE...]
    ```

    推送服务依的镜像。

    选项包括：

    - –ignore-push-failures 忽略推送镜像过程中的错误

14. restart
    格式为：

    ```
    docker-compose restart [options] [SERVICE...]
    ```

    重启项目中的服务。

    选项包括：

    - -t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为10秒）

15. rm
    格式为：

    ```
    docker-compose rm [options] [SERVICE...]
    删除所有（停止状态的）服务容器。
    ```

    删除所有（停止状态的）服务容器。

    选项包括：

    - –f, --force 强制直接删除，包括非停止状态的容器
    - -v 删除容器所挂载的数据卷

16. run
    格式为：

    ```
    docker-compose run [options] [-v VOLUME...] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
    ```

    在指定服务上执行一个命令。

    例如：

    ```
    docker-compose run ubuntu ping www.anumbrella.net
    ```
    
    将会执行一个ubuntu容器，并执行`ping www.anumbrella.net`命令。
    
    默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。该命令类似于启动容器后运行指定的命令，相关卷、链接等都会按照配置自动创建。有两个不同点：
    
    给定命令将会覆盖原有的自动运行命令
    不会自动创建端口，以避免冲突
    如果不希望自动启动关联的容器，可以使用–no-deps选项，例如：
    
    ```
    docker-compose run --no-deps web
    ```
    
    将不会启动web容器关联的其他容器。
    
    将不会启动web容器关联的其他容器。
    
    选项包括：
    
    - -d 在后台运行服务容器
    
    - –name NAME 为容器指定一个名字
    
    - –entrypoint CMD 覆盖默认的容器启动指令
    
    - -e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量
    
    - -u, --user="" 指定运行容器的用户名或者uid
    
    - –no-deps 不自动启动管理的服务容器
    
    - –rm 运行命令后自动删除容器，d模式下将忽略
    
    - -p, --publish=[] 映射容器端口到本地主机
    
    - –service-ports 配置服务端口并映射到本地主机
    
    - -v, --volume=[] 绑定一个数据卷，默认为空
    
    - -T 不分配伪tty，意味着依赖tty的指令将无法运行
    
    - -w, --workdir="" 为容器指定默认工作目录
    
17. scale
    格式为：

    ```
    docker-compose scale [options] [SERVICE=NUM...]
    ```

    设置指定服务运行的容器个数。
    通过service=num的参数来设置数量。例如：

    ```
    docker-compose scale web=3 db=2
    ```

    将启动3个容器运行web服务，2个容器运行db服务。一般情况下，当指定书目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

    选项包括：

    - -t, --timeout TIMEOUT 停止容器时候的超时（默认为10秒）

18. start
    格式为：

    ```
    docker-compose start [SERVICE...]
    ```

    启动已经存在的服务容器。

19. stop
    格式为：

    ```
    docker-compose stop [options] [SERVICE...]
    ```

    停止已经处于运行状态的容器，但不删除它。

    选项包括：

    - -t, --timeout TIMEOUT 停止容器时候的超时（默认为10秒）

20. top
    格式为：

    ```
    docker-compose stop [options] [SERVICE...]
    ```


    显示各个容器运行的进程情况。

21. unpause
    格式为：

    ```
    docker-compose unpause [SERVICE...]
    ```
    
    恢复处于暂停状态中的服务。
    
22. up
    格式为：

    ```
    docker-compose up [options] [--scale SERVICE=NUM...] [SERVICE...]
    ```

    up命令十分强大，它尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一些列操作。链接的服务都将会被自动启动，除非已经处于运行状态。

    多数情况下我们可以直接通过该命令来启动一个项目。

    选项包括：

    - d 在后台运行服务容器

    - –no-color 不使用颜色来区分不同的服务的控制输出
    - –no-deps 不启动服务所链接的容器
    - –force-recreate 强制重新创建容器，不能与–no-recreate同时使用
    - –no-recreate 如果容器已经存在，则不重新创建，不能与–force-recreate同时使用
    - –no-build 不自动构建缺失的服务镜像
    - –build 在启动容器前构建服务镜像
    - –abort-on-container-exit 停止所有容器，如果任何一个容器被停止，不能与-d同时使用
    - -t, --timeout TIMEOUT 停止容器时候的超时（默认为10秒）
    - –remove-orphans 删除服务中没有在compose文件中定义的容器
    - –scale SERVICE=NUM 设置服务运行容器的个数，将覆盖在compose中通过scale指定的参数

Compose命令集：https://docs.docker.com/compose/reference/

#### Compose模板文件

模板文件是使用 `Compose` 的核心，涉及到的指令关键字也比较多。但大家不用担心，这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

```yaml
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

注意每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中再次设置。

Docker Compose的模板文件主要分为3个区域，为：

**services**
服务，在它下面可以定义应用需要的一些服务，每个服务都有自己的名字、使用的镜像、挂载的数据卷、所属的网络、依赖哪些其他服务等等。

**volumes**
数据卷，在它下面可以定义的数据卷（名字等等），然后挂载到不同的服务下去使用。

**networks**
应用的网络，在它下面可以定义应用的名字、使用的网络类型等等。

> 注意：每个服务都必须通过image指令指定镜像或build指令（需要Dockerfile）等来自动构建生成镜像。如果使用build指令，在Dockefile中设置的选项（例如：CMD、EXPOSE、VOLUME、ENV等）将会自动被获取，无需在docker-compose.yml中再次设置。
>

Docker Compose常用模板文件主要命令：

![](https://i.loli.net/2019/08/01/5d4280ae52b4c49353.png)

**一些主要指令的用法**

1. `build`

   指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

   ```yaml
   version: '3'
   services:
   
     webapp:
       build: ./dir
   ```

   你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。

   使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

   使用 `arg` 指令指定构建镜像时的变量。

   ```yaml
   version: '3'
   services:
   
     webapp:
       build:
         context: ./dir
         dockerfile: Dockerfile-alternate
         args:
           buildno: 1
   ```

   使用 `cache_from` 指定构建镜像的缓存

   ```yaml
   build:
     context: .
     cache_from:
       - alpine:latest
       - corp/web_app:3.14
   ```

2. `cap_add, cap_drop`

   指定容器的内核能力（capacity）分配。

   例如，让容器拥有所有能力可以指定为：

   ```yaml
   cap_add:
     - ALL
   ```

   去掉 NET_ADMIN 能力可以指定为：

   ```yaml
   cap_drop:
     - NET_ADMIN
   ```

3. `command`

   覆盖容器启动后默认执行的命令。

   ```yaml
   command: echo "hello world"
   ```

4. `configs`

   仅用于 `Swarm mode`。

5. `cgroup_parent`

   指定父 `cgroup` 组，意味着将继承该组的资源限制。

   例如，创建了一个 cgroup 组名称为 `cgroups_1`。

   ```yaml
   cgroup_parent: cgroups_1
   ```

6. `container_name`

   指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。

   ```yaml
   container_name: docker-web-container
   ```

   > 注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

7. `deploy`

   仅用于 `Swarm mode`

8. `devices`

   指定设备映射关系。

   ```yaml
   devices:
     - "/dev/ttyUSB1:/dev/ttyUSB0"
   ```

9. `depends_on`

   解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

   ```yaml
   version: '3'
   
   services:
     web:
       build: .
       depends_on:
         - db
         - redis
   
     redis:
       image: redis
   
     db:
       image: postgres
   ```

   > 注意：`web` 服务不会等待 `redis` `db` 「完全启动」之后才启动。

10. `dns`

    自定义 `DNS` 服务器。可以是一个值，也可以是一个列表。

    ```yaml
    dns: 8.8.8.8
    
    dns:
      - 8.8.8.8
      - 114.114.114.114
    ```

11. `dns_search`

    配置 `DNS` 搜索域。可以是一个值，也可以是一个列表。

    ```yaml
    dns_search: example.com
    
    dns_search:
      - domain1.example.com
      - domain2.example.com
    ```

12. `tmpfs`

    挂载一个 tmpfs 文件系统到容器。

    ```yaml
    tmpfs: /run
    tmpfs:
      - /run
      - /tmp
    ```

13. `env_file`

    从文件中获取环境变量，可以为单独的文件路径或列表。

    如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

    如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

    ```bash
    env_file: .env
    
    env_file:
      - ./common.env
      - ./apps/web.env
      - /opt/secrets.env
    ```

    环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

    ```bash
    # common.env: Set development environment
    PROG_ENV=development
    ```

14. `environment`

    设置环境变量。你可以使用数组或字典两种格式。

    只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

    ```yaml
    environment:
      RACK_ENV: development
      SESSION_SECRET:
    
    environment:
      - RACK_ENV=development
      - SESSION_SECRET
    ```

    如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](http://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括

    ```bash
    y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
    ```

15. `expose`

    暴露端口，但不映射到宿主机，只被连接的服务访问。

    仅可以指定内部端口为参数

    ```yaml
    expose:
     - "3000"
     - "8000"
    ```

16. `external_links`

    > 注意：不建议使用该指令。

    链接到 `docker-compose.yml` 外部的容器，甚至并非 `Compose` 管理的外部容器。

    ```yaml
    external_links:
     - redis_1
     - project_db_1:mysql
     - project_db_1:postgresql
    ```

17. `extra_hosts`

    类似 Docker 中的 `--add-host` 参数，指定额外的 host 名称映射信息。

    ```yaml
    extra_hosts:
     - "googledns:8.8.8.8"
     - "dockerhub:52.1.157.61"
    ```

    会在启动后的服务容器中 `/etc/hosts` 文件中添加如下两条条目。

    ```bash
    8.8.8.8 googledns
    52.1.157.61 dockerhub
    ```

18. `healthcheck`

    通过命令检查容器是否健康运行。

    ```yaml
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 1m30s
      timeout: 10s
      retries: 3
    ```

19. `image`

    指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

    ```yaml
    image: ubuntu
    image: orchardup/postgresql
    image: a4bc65fd
    ```

20. `labels`

    为容器添加 Docker 元数据（metadata）信息。例如可以为容器添加辅助说明信息。

    ```yaml
    labels:
      com.startupteam.description: "webapp for a startup team"
      com.startupteam.department: "devops department"
      com.startupteam.release: "rc3 for v1.0"
    ```

21. `links`

    注意：不推荐使用该指令。

22. `logging`

    配置日志选项。

    ```yaml
    logging:
      driver: syslog
      options:
        syslog-address: "tcp://192.168.0.42:123"
    ```

    目前支持三种日志驱动类型。

    ```yaml
    driver: "json-file"
    driver: "syslog"
    driver: "none"
    ```

    `options` 配置日志驱动的相关参数。

    ```yaml
    options:
      max-size: "200k"
      max-file: "10"
    ```

23. `network_mode`

    设置网络模式。使用和 `docker run` 的 `--network` 参数一样的值。

    ```yaml
    network_mode: "bridge"
    network_mode: "host"
    network_mode: "none"
    network_mode: "service:[service name]"
    network_mode: "container:[container name/id]"
    ```

24. `networks`

    配置容器连接的网络。

    ```yaml
    version: "3"
    services:
    
      some-service:
        networks:
         - some-network
         - other-network
    
    networks:
      some-network:
      other-network:
    ```

25. `pid`

    跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 ID 来相互访问和操作。

    ```yaml
    pid: "host"
    ```

26. `ports`

    暴露端口信息。

    使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

    ```yaml
    ports:
     - "3000"
     - "8000:8000"
     - "49100:22"
     - "127.0.0.1:8001:8001"
    ```

    *注意：当使用 HOST:CONTAINER 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 YAML 会自动解析 xx:yy 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。*

27. `secrets`

    存储敏感数据，例如 `mysql` 服务密码。

    ```yaml
    version: "3.1"
    services:
    
    mysql:
      image: mysql
      environment:
        MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
      secrets:
        - db_root_password
        - my_other_secret
    
    secrets:
      my_secret:
        file: ./my_secret.txt
      my_other_secret:
        external: true
    ```

28. `security_opt`

    指定容器模板标签（label）机制的默认属性（用户、角色、类型、级别等）。例如配置标签的用户名和角色名。

    ```yaml
    security_opt:
        - label:user:USER
        - label:role:ROLE
    ```

29. `stop_signal`

    设置另一个信号来停止容器。在默认情况下使用的是 SIGTERM 停止容器。

    ```yaml
    stop_signal: SIGUSR1
    ```

30. `sysctls`

    配置容器内核参数。

    ```yaml
    sysctls:
      net.core.somaxconn: 1024
      net.ipv4.tcp_syncookies: 0
    
    sysctls:
      - net.core.somaxconn=1024
      - net.ipv4.tcp_syncookies=0
    ```

31. `ulimits`

    指定容器的 ulimits 限制值。

    例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。

    ```yaml
      ulimits:
        nproc: 65535
        nofile:
          soft: 20000
          hard: 40000
    ```

32. `volumes`

    数据卷所挂载路径设置。可以设置宿主机路径 （`HOST:CONTAINER`） 或加上访问模式 （`HOST:CONTAINER:ro`）。

    该指令中路径支持相对路径。

    ```yaml
    volumes:
     - /var/lib/mysql
     - cache/:/tmp/cache
     - ~/configs:/etc/configs/:ro
    ```

**其它指令**

此外，还有包括 `domainname, entrypoint, hostname, ipc, mac_address, privileged, read_only, shm_size, restart, stdin_open, tty, user, working_dir`等指令，基本跟 `docker run` 中对应参数的功能一致。

指定服务容器启动后执行的入口文件。

```yaml
entrypoint: /code/entrypoint.sh
```

指定容器中运行应用的用户名。

```yaml
user: nginx
```

指定容器中工作目录。

```yaml
working_dir: /code
```

指定容器中搜索域名、主机名、mac 地址等。

```yaml
domainname: your_website.com
hostname: test
mac_address: 08-00-27-00-0C-0A
```

允许容器中运行一些特权命令。

```yaml
privileged: true
```

指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 `always` 或者 `unless-stopped`。

```yaml
restart: always
```

以只读模式挂载容器的 root 文件系统，意味着不能对容器内容进行修改。

```yaml
read_only: true
```

打开标准输入，可以接受外部输入。

```yaml
stdin_open: true
```

模拟一个伪终端。

```yaml
tty: true
```

**读取变量**

Compose 模板文件支持动态读取主机的系统环境变量和当前目录下的 `.env` 文件中的变量。

例如，下面的 Compose 文件将从运行它的环境中读取变量 `${MONGO_VERSION}` 的值，并写入执行的指令中。

```yaml
version: "3"
services:

db:
  image: "mongo:${MONGO_VERSION}"
```

如果执行 `MONGO_VERSION=3.2 docker-compose up` 则会启动一个 `mongo:3.2` 镜像的容器；如果执行 `MONGO_VERSION=2.8 docker-compose up` 则会启动一个 `mongo:2.8` 镜像的容器。

若当前目录存在 `.env` 文件，执行 `docker-compose` 命令时将从该文件中读取变量。

在当前目录新建 `.env` 文件并写入以下内容。

```bash
# 支持 # 号注释
MONGO_VERSION=3.6
```

执行 `docker-compose up` 则会启动一个 `mongo:3.6` 镜像的容器。

**更新容器**

当服务的配置发生更改时，可使用 docker-compose up 命令更新配置

此时，Compose 会删除旧容器并创建新容器，新容器会以不同的 IP 地址加入网络，名称保持不变，任何指向旧容起的连接都会被关闭，重新找到新容器并连接上去
[compose-file](https://docs.docker.com/compose/compose-file/)。

#### Docker Compose示例

**示例一**

首先新建一个文件夹compose，然后创建文件docker-compose.yml。

```
$ mkdir compose && cd compose
$ touch docker-compose.yml
```

编辑docker-compose.yml如下：

```yml
version: '2'
services:
  web1:
    image: nginx
    ports: 
      - "8080:80"
    container_name: "web1"
  web2:
    image: nginx
    ports: 
      - "8081:80"
    container_name: "web2"
#volumes:

#networks:

```

在这里我们声明了服务，并在下面定义了两个容器web1，web2，同时指定镜像为nginx，并指定了name或端口，而网络和数据卷并没有进行声明。

接着我们使用`docker-compose up` 启动该服务。

```bash
$ docker-compose up      
Creating network "compose_default" with the default driver
Creating web1 ... 
Creating web2 ... 
Creating web2
Creating web1 ... done
Attaching to web2, web1

```

我们可以查看到命令中，首先创建了一个默认的compose_default的网络，然后创建容器，并「Attaching to …」，将网络应用到服务上。

我们可以查看一下具体的网络，使用docker network ls 如下：

```
$docker network ls
NETWORK ID          NAME                  DRIVER              SCOPE
e0fe902ae5b0        bridge                bridge              local
d0246306a058        compose_default       bridge              local
26d695438cc0        composetest_default   bridge              local
e0730e46e361        docker_gwbridge       bridge              local
653b092f26ef        host                  host                local
c1129fb04ab6        none                  null                local
```

然后我们访问：<http://127.0.0.1:8080>或<http://127.0.0.1:8081>，可以发现终端中输出具体的访问日志。

这里我们启动的服务并不是以守护进程启动了，所以关闭客户端就关闭了服务。如果要一守护进程启动，加上-d参数。如下：

```
docker-compose up -d
```

接下来我们可以使用一些前文介绍的命令，`docker-compose ps`查看服务

```
$ docker-compose ps   
Name         Command          State          Ports        
----------------------------------------------------------
web1   nginx -g daemon off;   Up      0.0.0.0:8080->80/tcp
web2   nginx -g daemon off;   Up      0.0.0.0:8081->80/tcp

```

`docker-compose stop [name]` 停止服务，`docker-compose start [name]`启动服务

`docker-compose rm [name]`删除服务，需要停止服务，否则使用-f参数，与`docker rm`命令类似

> 注意：这个docker-compose rm不会删除应用的网络和数据卷。查看一下网络，可以看到应用创建的网络

如果要删除数据卷、网络、应用则使用`docker-compose down`命令。

`docker-compose logs -f [name]`查看具体服务的日志。

通过`docker-compose exec [name] shell`可以进入容器内部，例如，进入web1容器内部，使用`docker-compose exec web1 /bin/bash`命令。

现在我们更改docker-compose文件，更改为如下配置：

```
version: '2'
services:
  web1:
    image: nginx
    ports: 
      - "8080:80"
    container_name: "web1"
    networks:
      - dev
  web2:
    image: nginx
    ports: 
      - "8081:80"
    container_name: "web2"
    networks:
      - dev
      - pro
  web3:
    image: nginx
    ports: 
      - "8082:80"
    container_name: "web3"
    networks:
      - pro

networks:
  dev:
    driver: bridge
  pro:
    driver: bridge

#volumes:

```

我们添加了容器web3，并指定了网络。使用`docker-compose down`关闭先前的服务，重新启动服务。

我们指定了dev、pro网络，同时给web1指定了dev网络，web2指定了dev、pro网络，web3指定了pro网络，因此web1和web2可以互相访问到，web2和web3可以互相访问到，而web1与web3就无法访问。这就可以通过配置来实现容器直接的互通和隔离。

现在我们再次更改docker-compose.yml模板文件，如下：

```
version: '2'
services:
  web1:
    image: nginx
    ports: 
      - "8080:80"
    container_name: "web1"
    networks:
      - dev
    volumes:
      - ntfs:/data
  web2:
    image: nginx
    ports: 
      - "8081:80"
    container_name: "web2"
    networks:
      - dev
      - pro
    volumes:
      - ./:/usr/share/nginx/html
  web3:
    image: nginx
    ports: 
      - "8082:80"
    container_name: "web3"
    networks:
      - pro
    volumes:
      - ./:/usr/share/nginx/html

networks:
  dev:
    driver: bridge
  pro:
    driver: bridge

volumes:
  ntfs:
    driver: local
```

我们添加了volumes声明，在web1中我们挂载数据在本地，而web2、web3我们则挂载compose当前目录与nginx的/usr/share/nginx/html相通。同样使用docker-compose down关闭先前的服务，并重新启动服务。

然后我们在当前compose目录新建index.html文件，这里的index.html随便编写一些内容，

然后我们可以进入web2、web3中，在容器的/usr/share/nginx/html中我们可以查看到与我们compose目录一致。

```
$ docker-compose exec web2 /bin/bash
root@9aa823919742:/# ls /usr/share/nginx/html/
docker-compose.yml  index.html
root@9aa823919742:/# cat /usr/share/nginx/html/index.html 
hello World
root@9aa823919742:/# exit
exit
$ docker-compose exec web3 /bin/bash 
root@bf6eea37239b:/# ls  /usr/share/nginx/html/
docker-compose.yml  index.html
root@bf6eea37239b:/# cat /usr/share/nginx/html/index.html
hello World
root@bf6eea37239b:/# exit
exit
```

然后我们访问http://localhost:8081和http://localhost:8082，得到的结果与我们预期的也一致。

通过Compose，我们快速创建一套基于Docker容器的服务栈，然后使用docker-compose脚本来启动，停止和重启应用，非常适合组合使用多个容器进行开发的场景。

**示例二**

最常见的项目是 web 网站，该项目应该包含 web 应用和缓存。

下面我们用 `Python` 来建立一个能够记录页面访问次数的 web 网站。

1. 创建测试项目文件夹

   ```
   $ mkdir composetest
   $ cd composetest
   ```

2. 编辑app.py并保存（描述：简单的一个httpserver，主要是为了类似tomcat的一个sevlet，当访问一次，redis节点就增加一个，就可以看到相应的输出）

   ```python
   from flask import Flask
   from redis import Redis
   
   app = Flask(__name__)
   redis = Redis(host='redis', port=6379)
   
   @app.route('/')
   def hello():
       count = redis.incr('hits')
       return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)
   
   if __name__ == "__main__":
       app.run(host="0.0.0.0", debug=True)
   ```

3. 编写 `Dockerfile` 文件，内容为

   ```
   FROM python:3.7
   ADD . /code
   WORKDIR /code
RUN pip install redis flask
   CMD ["python", "app.py"]
   ```

4. 定义服务

   编写 `docker-compose.yml` 文件，这个是 Compose 使用的主模板文件。

   Compose文件定义了2个服务，web和redis。

   Web服务：

   1 从当前目录下的dockerfile创建

   2 容器的5000端口与宿主机5000端口绑定

   3 将项目目录与容器内的/code目录绑定

   4 web服务与redis服务建立连接

   ```dockerfile
   version: '3'
   services:
     web:
       build: .
       ports:
        - "5000:5000"
     redis:
       image: "redis:latest"
   ```

5. 通过compose运行app服务

   ```
   $ docker-compose up
   ```
   
   此时访问本地 `5000` 端口，每次刷新页面，计数就会加 1。

备注：

```bash
$ docker-compose up –d #（后台启动）
$ docker-compose stop #（停止运行）
```

**示例三**

我们现在将使用 `Docker Compose` 配置并运行一个 `Django/PostgreSQL` 应用。

在一切工作开始前，需要先编辑好三个必要的文件。

第一步，因为应用将要运行在一个满足所有环境依赖的 Docker 容器里面，那么我们可以通过编辑 `Dockerfile` 文件来指定 Docker 容器要安装内容。内容如下：

```docker
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

以上内容指定应用将使用安装了 Python 以及必要依赖包的镜像。

第二步，在 `requirements.txt` 文件里面写明需要安装的具体依赖包名。

```bash
Django>=1.8,<2.0
psycopg2
```

第三步，`docker-compose.yml` 文件将把所有的东西关联起来。它描述了应用的构成（一个 web 服务和一个数据库）、使用的 Docker 镜像、镜像之间的连接、挂载到容器的卷，以及服务开放的端口。

```yaml
version: "3"
services:

  db:
    image: postgres

  web:
    build: .
    command: python3 manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    links:
      - db
```

现在我们就可以使用 `docker-compose run` 命令启动一个 `Django` 应用了。

```bash
$ docker-compose run web django-admin.py startproject django_example .
```

Compose 会先使用 `Dockerfile` 为 web 服务创建一个镜像，接着使用这个镜像在容器里运行 `django-admin.py startproject composeexample` 指令。

这将在当前目录生成一个 `Django` 应用。

```bash
$ ls
Dockerfile       docker-compose.yml          django_example       manage.py       requirements.txt
```

如果你的系统是 Linux,记得更改文件权限。

```bash
sudo chown -R $USER:$USER .
```

首先，我们要为应用设置好数据库的连接信息。用以下内容替换 `django_example/settings.py` 文件中 `DATABASES = ...` 定义的节点内容。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

这些信息是在 [postgres](https://store.docker.com/images/postgres/) 镜像固定设置好的。然后，运行 `docker-compose up` ：

```bash
$ docker-compose up

django_db_1 is up-to-date
Creating django_web_1 ...
Creating django_web_1 ... done
Attaching to django_db_1, django_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".

web_1  | Performing system checks...
web_1  |
web_1  | System check identified no issues (0 silenced).
web_1  |
web_1  | November 23, 2017 - 06:21:19
web_1  | Django version 1.11.7, using settings 'django_example.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
```

这个 `Django` 应用已经开始在你的 Docker 守护进程里监听着 `8000` 端口了。打开 `127.0.0.1:8000` 即可看到 `Django` 欢迎页面。

你还可以在 Docker 上运行其它的管理命令，例如对于同步数据库结构这种事，在运行完 `docker-compose up` 后，在另外一个终端进入文件夹运行以下命令即可：

```bash
$ docker-compose run web python manage.py syncdb
```

**示例四**

我们现在将使用 `Compose` 配置并运行一个 `Rails/PostgreSQL` 应用。

在一切工作开始前，需要先设置好三个必要的文件。

首先，因为应用将要运行在一个满足所有环境依赖的 Docker 容器里面，那么我们可以通过编辑 `Dockerfile` 文件来指定 Docker 容器要安装内容。内容如下：

```docker
FROM ruby
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
RUN bundle install
ADD . /myapp
```

以上内容指定应用将使用安装了 Ruby、Bundler 以及其依赖件的镜像。

 下一步，我们需要一个引导加载 Rails 的文件 `Gemfile` 。 等一会儿它还会被 `rails new` 命令覆盖重写。

```bash
source 'https://gems.ruby-china.com'
gem 'rails', '4.0.2'
```

最后，`docker-compose.yml` 文件才是最神奇的地方。 `docker-compose.yml` 文件将把所有的东西关联起来。它描述了应用的构成（一个 web 服务和一个数据库）、每个镜像的来源（数据库运行在使用预定义的 PostgreSQL 镜像，web 应用侧将从本地目录创建）、镜像之间的连接，以及服务开放的端口。

```yaml
version: "3"
services:

  db:
    image: postgres
    ports:
      - "5432"

  web:
    build: .
    command: bundle exec rackup -p 3000
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    links:
      - db
```

所有文件就绪后，我们就可以通过使用 `docker-compose run` 命令生成应用的骨架了。

```bash
$ docker-compose run web rails new . --force --database=postgresql --skip-bundle
```

`Compose` 会先使用 `Dockerfile` 为 web 服务创建一个镜像，接着使用这个镜像在容器里运行 `rails new` 和它之后的命令。一旦这个命令运行完后，应该就可以看一个崭新的应用已经生成了。

```bash
$ ls
Dockerfile   app          docker-compose.yml      tmp
Gemfile      bin          lib          vendor
Gemfile.lock condocker-compose       log
README.rdoc  condocker-compose.ru    public
Rakefile     db           test
```

在新的 `Gemfile` 文件去掉加载 `therubyracer` 的行的注释，这样我们便可以使用 Javascript 运行环境：

```bash
source 'https://gems.ruby-china.com'

gem 'therubyracer', platforms: :ruby
```

现在我们已经有一个新的 `Gemfile` 文件，需要再重新创建镜像。（这个会步骤会改变 Dockerfile 文件本身，所以需要重建一次）。

```bash
$ docker-compose build
```

应用现在就可以启动了，但配置还未完成。Rails 默认读取的数据库目标是 `localhost`，我们需要手动指定容器的 `db` 。同样的，还需要把用户名修改成和 postgres 镜像预定的一致。 打开最新生成的 `database.yml` 文件。用以下内容替换：

```bash
development: &default
  adapter: postgresql
  encoding: unicode
  database: postgres
  pool: 5
  username: postgres
  password:
  host: db

test:
  <<: *default
  database: myapp_test
```

现在就可以启动应用了。

```bash
$ docker-compose up
```

如果一切正常，你应该可以看到 PostgreSQL 的输出，几秒后可以看到这样的重复信息：

```bash
myapp_web_1 | [2014-01-17 17:16:29] INFO  WEBrick 1.3.1
myapp_web_1 | [2014-01-17 17:16:29] INFO  ruby 2.0.0 (2013-11-22) [x86_64-linux-gnu]
myapp_web_1 | [2014-01-17 17:16:29] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
```

最后， 我们需要做的是创建数据库，打开另一个终端，运行：

```bash
$ docker-compose run web rake db:create
```

这个 web 应用已经开始在你的 docker 守护进程里面监听着 3000 端口了。

**示例五**

`Compose` 可以很便捷的让 `Wordpress` 运行在一个独立的环境中。

假设新建一个名为 `wordpress` 的文件夹，然后进入这个文件夹。

1. 创建 docker-compose.yml 文件

   [`docker-compose.yml`](https://github.com/yeasy/docker_practice/blob/master/compose/demo/wordpress/docker-compose.yml) 文件将开启一个 `wordpress` 服务和一个独立的 `MySQL` 实例：

   ```yaml
   version: "3"
   services:
   
      db:
        image: mysql:5.7
        volumes:
          - db_data:/var/lib/mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: somewordpress
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wordpress
          MYSQL_PASSWORD: wordpress
   
      wordpress:
        depends_on:
          - db
        image: wordpress:latest
        ports:
          - "8000:80"
        restart: always
        environment:
          WORDPRESS_DB_HOST: db:3306
          WORDPRESS_DB_USER: wordpress
          WORDPRESS_DB_PASSWORD: wordpress
   volumes:
     db_data:
   ```

2. 构建并运行项目

   运行 `docker-compose up -d` Compose 就会拉取镜像再创建我们所需要的镜像，然后启动 `wordpress` 和数据库容器。 接着浏览器访问 `127.0.0.1:8000` 端口就能看到 `WordPress` 安装界面了。

#### docker内通过127.0.0.1访问宿主机报错:Connection refused

已通过docker启动mongodb,监听端口为27017. 直接启动应用(不通过docker)可以正常访问到mongodb,但是通过docker访问却不行,访问的url为: mongodb://127.0.0.1:27017或[mongodb://localhost:27017](https://links.jianshu.com/go?to=mongodb%3A%2F%2Flocalhost%3A27017)

```
2019-04-18 06:05:52.694 [cluster-ClusterId{value='5cb813be26261e30fc3db87f', description='null'}-127.0.0.1:27017] DEBUG org.mongodb.driver.cluster - Updating cluster description to  {type=UNKNOWN, servers=[{address=127.0.0.1:27017, type=UNKNOWN, state=CONNECTING, exception={com.mongodb.MongoSocketOpenException: Exception opening socket}, caused by {java.net.ConnectException: Connection refused}}]
```

docker是一个虚拟环境,127.0.0.1和localhost指的是虚拟环境内部,而不是外部宿主机,所以无法这样访问.

**解决方案**

1. 对于mac和windows,可以使用host.docker.internal替换127.0.0.1,如 [mongodb://host.docker.internal:27017](https://links.jianshu.com/go?to=mongodb%3A%2F%2Fhost.docker.internal%3A27017)
2. 对于Linux可以采用如下方案(后续应该也可以用上面的方案,但是当前docker还没有修改[此问题](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fdocker%2Flibnetwork%2Fpull%2F2348)):

- 创建一个桥接网络
  下面的localNet是网络名字,可自行修改;关于192.168.0.0这个子网,也可以自行定义.
  默认按照下面的命令,执行后将可以通过192.168.0.1访问宿主机.
  `docker network create -d bridge --subnet 192.168.0.0/24 --gateway 192.168.0.1 localNet`
- 使用192.168.0.1替换127.0.0.1,如mongodb://192.168.0.1:27017



### 参考

1.[Docker三剑客——Compose项目](https://www.cntofu.com/book/139/compose/README.md)