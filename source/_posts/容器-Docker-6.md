---
title: Docker 常用示例
date: 2019-10-04 08:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

#### Dockerfile打包SpringBoot的Java Web应用

1. 首先登录[https://start.spring.io/](https://link.jianshu.com/?t=https%3A%2F%2Fstart.spring.io%2F)，选择Maven + WEB来创建一个基于tomcat的web应用。

2. 在创建好的web应用里，仅仅填写controller层来达到简单测试的目的。
   Contoller的代码如下：

<!--more-->

```java
package com.hyp.learn.myapp;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class Controller {
    @GetMapping("/HelloK8s")
    @ResponseBody
    String mongoDBupdate(String name) {
        return "Hello Docker !!!   Hello K8s !!!" ;
    }
}
```

3. 运行mvn compile package来生成一个该web应用的jar包。

**创建Dockerfile文件**

项目根路径下创建Dockerfile

```
touch Dockerfile
```

Dockerfile的内容如下：

```dockerfile
FROM openjdk:8-jdk-alpine
MAINTAINER hyp
LABEL app="myApp" version="0.0.1" by="hyp"
COPY ./target/myapp-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

执行 **docker build -t myapp .** 创建image。
注意：这里的 . 代表当前路径

Build成功以后，可以通过命令**docker images**查看已经创建的image。

运行**docker run --name mywebapp -p 8080:8080 -d myapp**，跑起来！！！

#### 用Dockerfile打造你的自动化构建工具

`自动化构建`是应用发布过程中必不可少的环节， 常用的构建工具有`jenkins` ,`walle` 等。而这些工具在构建应用时通常会有以下问题：

> 1. 需要直接或间接的写一坨用于构建的shell命令等，不易管理、兼容性较差
> 2. 上面一点可能还比较容易解决，但最为致命的是：重度依赖如`jenkins`宿主机或打包机上的软件环境，如`git`, `maven`,`java`等

理想情况是： 不同的应用如java应用、go应用、php应用等等，都可以在某台负责构建的宿主机上并行无干扰的执行构建操作，且构建中依赖的软件环境、构建流程等都可以由开发人员控制。

到目前为止，能很好的完成以上使命的，可能非[docker](https://blog.zhouzhipeng.com/category/docker)莫属了！

在docker的世界里，构建交付的是`镜像`,而能够产生镜像的是`Dockerfile` (手动使用`docker commit` 的另当别论).

在`docker ce 17.05` 之后，出现了一个很重要的特性`Multi-Stage Build` (多阶段构建) , 它将显著提升你的运维生产力!

**在Multi-Stage Build之前**

以下演示以`java` hello world 为例，完整代码在： https://github.com/zhouzhipeng/docker-multi-stage-demo

这是一个标准的maven 项目，仅有个HelloWorld主类。大体构建思路为：

1. 在maven镜像中编译并打包项目
2. 将步骤1中生成的jar拷贝出来
3. 用步骤2得到的jar，在jre镜像中构建并运行jar中的主类

Dockerfile.build 用于编译和打包jar

```

FROM maven:3.5.2-alpine

MAINTAINER hyp

WORKDIR /app

COPY . .

# 编译打包
RUN mvn package -Dmaven.test.skip=true
```

Dockerfile.old 用于运行jar中的主类

```
FROM openjdk:8-jre-alpine

MAINTAINER hyp

WORKDIR /app

COPY docker-multi-stage-demo-1.0-SNAPSHOT.jar .

# 运行main类
CMD java -cp docker-multi-stage-demo-1.0-SNAPSHOT.jar com.zhouzhipeng.HelloWorld

```

注意到，两个dockerfile之间关联的 docker-multi-stage-demo-1.0-SNAPSHOT.jar 文件，需要另外一个build.sh 脚本来串起来.

```bash
#!/usr/bin/env bash


# 1. 先构建出带有产物jar的镜像
docker build -t zhouzhipeng/dockermultistagedemo-build -f Dockerfile.build .

# 2. 临时创建 dockermultistagedemo-build 容器
docker create --name build zhouzhipeng/dockermultistagedemo-build

# 3. 将上面容器中的jar拷贝出来
docker cp build:/app/target/docker-multi-stage-demo-1.0-SNAPSHOT.jar ./

# 4. 构建java执行的镜像
docker build -t zhouzhipeng/dockermultistagedemo -f Dockerfile.old .

# 5. 删除临时jar文件
rm -rf docker-multi-stage-demo-1.0-SNAPSHOT.jar
```

对Dockerfile和shell也了解的朋友相信应该都看得懂，在此不做过多赘述.

**在Multi-Stage Build之后**

看过上一节后，你也许会感觉是不是有点麻烦呢？ 是的，麻烦之处在于不仅要写多个dockerfile，而且还需要一个build.sh 脚本来额外执行。 无疑是增大了构建应用的复杂度！

将上面的Dockerfile.build 和Dockerfile.old 结合起来，稍加修饰，得到如下全新的Dockerfile：

```dockerfile

FROM maven:3.5.2-alpine as builder
MAINTAINER zhouzhipeng <admin@zhouzhipeng.com>
WORKDIR /app
COPY src .
COPY pom.xml .
# 编译打包 (jar包生成路径：/app/target)
RUN mvn package -Dmaven.test.skip=true


FROM openjdk:8-jre-alpine
MAINTAINER zhouzhipeng <admin@zhouzhipeng.com>
WORKDIR /app
COPY --from=builder /app/target/docker-multi-stage-demo-1.0-SNAPSHOT.jar .
# 运行main类
CMD java -cp docker-multi-stage-demo-1.0-SNAPSHOT.jar com.zhouzhipeng.HelloWorld

```

然后，仍然是熟悉的docker build命令

```
docker build -t zhouzhipeng/dockermultistagedemo-new .
```

即可。

细心的你应该不难发现，上面的Dockerfile 中有两处地方不一样，

1. 出现了多个`FROM `语句
2. `COPY` 命令后多了`--from=builder`

这就是今天的主咖 `Multi-Stage Build` , 先来通过一张图来直观感受下什么是所谓的`Multi-Stage Build` (多阶段构建 ）：

![UTOOLS1571733527278.png](https://i.loli.net/2019/10/22/xzPnvmGuq3Uh1eT.png)

通过多阶段构建，既可以保持Dockerfile简洁易读，又可以让最终的产物镜像很“干净”。

**简单理解**

还是以上文中的Dockerfile为例, 如下图所示：

![UTOOLS1571733562481.png](https://i.loli.net/2019/10/22/Lo6NIhRSMkisUGJ.png)

红框中的部分可以看作是一个个独立的“stage” ，可以粗略想象成就是一个独立的Dockerfile内容。

大家知道镜像构建是一层一层叠加的，按照Dockerfile的命令行顺序，由上至下依次执行叠加。 所以，下层的stage才可以引用到上层的stage，为了方便引用到上层的stage，故需要给其取一个名字, 用`as` 操作符。

`FROM` 命令的完整格式如下：

```
FROM <image>[:<tag>] [AS <name>]
```

stage之间交互的是文件，故`COPY` 命令需要扩展，通过`--from=` 来指定需要从上方的哪个"stage" 拷贝文件, 其完整命令格式如下：

```
COPY  --from=<name|index> <src>... <dest>
# 注意--from 是可选的，当上层的stage没有名字时可以按照index(从0开始)的顺序引用，eg. --from=0
```

值得一提的是，默认情况下使用`docker build` 命令构建一个包含多个stage的dockerfile时，最终的产物是最下方的一个stage 所产生的镜像。

当然，如果出于调试原因或其他需求，docker也是支持构建到指定的stage的，使用 `--target builder` 就可以只构建builder镜像。

```
docker build -t zhouzhipeng/builder --target builder .
```

**最后一步**

到目前为止，我们已经有了一个能够一键构建的Dockerfile 文件，接下来就只差让它能够自动构建了！

你可以用你熟悉的`jenkins` 结合github的webhook来实现提交一次代码，就执行一次docker build命令。

当然，我推荐个人体验的话就用官方的docker hub 吧，因为这样你构建的镜像还可以与他人共享。

`Multi-Stage Build` 这一特性非常适合做构建管道流，对于那些依赖环境复杂、流程也复杂的应用来说最合适不过了。

#### Python项目部署打包

创建一个空目录。将目录（`cd`）更改为新目录，创建一个名为的文件`Dockerfile`

```dockerfile
#DockerFile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

这`Dockerfile`是指我们尚未创建的几个文件，即 `app.py`和`requirements.txt`。让我们创建下一个。

再创建两个文件，`requirements.txt`然后`app.py`将它们放在同一个文件夹中`Dockerfile`。这完成了我们的应用程序，您可以看到它非常简单。当上述`Dockerfile`被内置到的图像，`app.py`并且 `requirements.txt`是因为存在`Dockerfile`的`ADD`命令，并从输出`app.py`是通过HTTP得益于访问`EXPOSE` 命令。

```
# requirements.txt
Flask
Redis

#app.py

from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

现在我们看到`pip install -r requirements.txt`为Python安装Flask和Redis库，应用程序打印环境变量`NAME`，以及调用的输出`socket.gethostname()`。最后，因为Redis没有运行（因为我们只安装了Python库，而不是Redis本身），我们应该期望在这里使用它的尝试失败并产生错误消息。

**构建**

```
# ls
app.py  Dockerfile  requirements.txt
```

现在运行build命令。这会创建一个Docker镜像，使用`-t`它来标记，因此它具有友好的名称。

```
# docker build -t friendlyhello .
# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
friendlyhello         latest              bf190f083ac1        17 seconds ago      148MB
```

运行映像

```
# docker run -p 4000:80 friendlyhello
# curl http://127.0.0.1:4000
<h3>Hello World!</h3><b>Hostname:</b> 59e4095e96de<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>% 
```

**编排**

在分布式应用程序中，应用程序的不同部分称为“服务”。

如果想象一个视频共享站点，它可能包括一个用于将应用程序数据存储在数据库中的服务，一个用于在后台进行视频转码的服务。用户上传内容，前端服务等。

服务实际上只是“生产中的容器”。服务只运行一个映像，但它编码了映像的运行方式 - 它应该使用哪些端口，应该运行多少个容器副本，以便服务具有所需的容量，以及等等。

扩展服务会更改运行该软件的容器实例的数量，从而为流程中的服务分配更多计算资源。

使用Docker平台定义，运行和扩展服务非常容易 - 只需编写一个`docker-compose.yml`文件即可。

```dockerfile
# docker-compose.yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: wangshu19930818/friendlyhello:v1
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:

```

在我们`docker stack deploy`首先运行命令之前：

```
docker swarm init
```

> 注意：如果您没有运行，`docker swarm init`则会收到“此节点不是群集管理器”的错误。

```
docker stack deploy -c docker-compose.yml getstartedlab
```

我们的单个服务堆栈在一台主机上运行已部署映像的5个容器实例。

在我们的应用程序中获取一项服务的服务ID：

```
docker ps
```

列出服务任务：

```
# docker service ps getstartedlab_web

ID                  NAME                  IMAGE                              NODE                DESIRED STATE       CURRENT STATE              ERROR               PORTS
ntx0dkfv8doi        getstartedlab_web.1   wangshu19930818/friendlyhello:v1   hyp-HP-Notebook     Running             Preparing 17 seconds ago                       
bx9tlijxzyjf        getstartedlab_web.2   wangshu19930818/friendlyhello:v1   hyp-HP-Notebook     Running             Preparing 17 seconds ago                       
mc4rsvdg2qe9        getstartedlab_web.3   wangshu19930818/friendlyhello:v1   hyp-HP-Notebook     Running             Preparing 17 seconds ago                       
rxlom9y249yb        getstartedlab_web.4   wangshu19930818/friendlyhello:v1   hyp-HP-Notebook     Running             Preparing 17 seconds ago                       
pqjzrxd0v06i        getstartedlab_web.5   wangshu19930818/friendlyhello:v1   hyp-HP-Notebook     Running             Preparing 17 seconds ago  
```

`curl -4 http://localhost:4000`连续多次运行，或者在浏览器中转到该URL并点击刷新几次。

- 将应用程序删除`docker stack rm`：

  ```
  docker stack rm getstartedlab
  ```

- 取下群。

  ```
  docker swarm leave --force
  ```

#### 可视化监控中心搭建

- adviser：负责收集容器的随时间变化的数据
- influxdb：负责存储时序数据
- grafana：负责分析和展示时序数据

1. 部署Influxdb服务

   可以将其视为一个数据库服务，其确实用于存储数据。之所以选用该数据库，原因正如官网所说：

   > Open Source Time Series DB Platform for Metrics & Events (Time Series Data)

   下面我们将该服务部署起来

   ```
   docker run -d -p 8086:8086 \
   -v ~/influxdb:/var/lib/influxdb \
   --name influxdb tutum/influxdb
   ```

   进入influxdb容器内部，并执行influx命令：

   ```
   docker exec -it influxdb influx
   
   CREATE DATABASE "test"
   CREATE USER "root" WITH PASSWORD 'root' WITH ALL PRIVILEGES
   
   exit
   ```

2. 部署cAdvisor服务

   谷歌的cadvisor可以用于收集Docker容器的时序信息，包括容器运行过程中的资源使用情况和性能数据。

   运行cadvisor服务

   ```
   docker run -d \
   -v /:/rootfs -v /var/run:/var/run -v /sys:/sys \
   -v /var/lib/docker:/var/lib/docker \
   --link=influxdb:influxdb --name cadvisor google/cadvisor \
   --storage_driver=influxdb \
   --storage_driver_host=influxdb:8086 \
   --storage_driver_db=test \
   --storage_driver_user=root \
   --storage_driver_password=root
   ```

   特别注意项：

   在运行上述docker时，这里有可能两个其他配置项需要添加（CentOS, RHEL需要）：

   `--privileged=true`

   设置为true之后，容器内的root才拥有真正的root权限，可以看到host上的设备，并且可以执行mount；否者容器内的root只是外部的一个普通用户权限。由于cadvisor需要通过socket访问docker守护进程，在CentOs和RHEL系统中需要这个这个选项。

   `--volume=/cgroup:/cgroup:ro`

   对于CentOS和RHEL系统的某些版本（比如CentOS6），cgroup的层级挂在/cgroup目录，所以运行cadvisor时需要额外添加–volume=/cgroup:/cgroup:ro选项

3. 部署Grafana服务

   grafana则是一款开源的时序数据分析工具，而且界面专业易用，等下等部署好了，大家就能感受到：

   ```
   docker run -d -p 5000:3000 \
   --link=influxdb:influxdb \
   --name grafana grafana/grafana
   ```

**实战**

访问grafana服务

打开localhost:5000来访问grafana的web服务，此时提示你需要登录，注意用户名和密码都是admin

grafana使用参考：<https://www.jianshu.com/p/7e7e0d06709b>

### 参考

1. [用Dockerfile打造你的自动化构建工具](https://segmentfault.com/a/1190000014595993)
2. [docker练习-容器和服务](https://www.cnblogs.com/wwchihiro/p/9293514.html)
3. [详解Docker容器可视化监控中心搭建](https://www.jb51.net/article/138318.htm)
4. [Docker 监控实战](http://blog.oneapm.com/apm-tech/306.html)