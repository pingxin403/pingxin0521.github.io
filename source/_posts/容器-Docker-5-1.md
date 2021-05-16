---
title: Docker 仓库
date: 2019-07-11 10:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

#### Docker Hub

目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)，其中已经包括了数量超过 15,000 的镜像。大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

你可以在 https://cloud.docker.com 免费注册一个 Docker 账号。

可以通过执行 `docker login` 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。

你可以通过 `docker logout` 退出登录。

<!--more-->

**拉取镜像**

你可以通过 `docker search` 命令来查找官方仓库中的镜像，并利用 `docker pull` 命令来将它下载到本地。

例如以 `centos` 为关键词进行搜索：

```bash
$ docker search centos
NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                          The official build of CentOS.                   465       [OK]
tianon/centos                                   CentOS 5 and 6, created using rinse instea...   28
blalor/centos                                   Bare-bones base CentOS 6.5 image                6                    [OK]
saltstack/centos-6-minimal                                                                      6                    [OK]
tutum/centos-6.4                                DEPRECATED. Use tutum/centos:6.4 instead. ...   5                    [OK]
```

可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、收藏数（表示该镜像的受关注程度）、是否官方创建、是否自动创建。

官方的镜像说明是官方项目组创建和维护的，automated 资源允许用户验证镜像的来源和内容。

根据是否是官方提供，可将镜像资源分为两类。

一种是类似 `centos` 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。

还有一种类型，比如 `tianon/centos` 镜像，它是由 Docker 的用户创建并维护的，往往带有用户名称前缀。可以通过前缀 `username/` 来指定使用某个用户提供的镜像，比如 tianon 用户。

另外，在查找的时候通过 `--filter=stars=N` 参数可以指定仅显示收藏数量为 `N` 以上的镜像。

下载官方 `centos` 镜像到本地。

```bash
$ docker pull centos
Pulling repository centos
0b443ba03958: Download complete
539c0211cd76: Download complete
511136ea3c5a: Download complete
7064731afe90: Download complete
```

**推送镜像**

用户也可以在登录后通过 `docker push` 命令来将自己的镜像推送到 Docker Hub。

以下命令中的 `username` 请替换为你的 Docker 账号用户名。

```bash
$ docker tag ubuntu:17.10 username/ubuntu:17.10

$ docker image ls

REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
ubuntu                                                   17.10                  275d79972a86        6 days ago          94.6MB
username/ubuntu                                          17.10                  275d79972a86        6 days ago          94.6MB

$ docker push username/ubuntu:17.10

$ docker search username

NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
username/ubuntu
```

**自动创建**

自动创建（Automated Builds）功能对于需要经常升级镜像内程序来说，十分方便。

有时候，用户创建了镜像，安装了某个软件，如果软件发布新版本则需要手动更新镜像。

而自动创建允许用户通过 Docker Hub 指定跟踪一个目标网站（目前支持 [GitHub](https://github.com/) 或 [BitBucket](https://bitbucket.org/)）上的项目，一旦项目发生新的提交或者创建新的标签（tag），Docker Hub 会自动构建镜像并推送到 Docker Hub 中。

要配置自动创建，包括如下的步骤：

- 创建并登录 Docker Hub，以及目标网站；
- 在目标网站中连接帐户到 Docker Hub；
- 在 Docker Hub 中 [配置一个自动创建](https://registry.hub.docker.com/builds/add/)；
- 选取一个目标网站中的项目（需要含 `Dockerfile`）和分支；
- 指定 `Dockerfile` 的位置，并提交创建。

之后，可以在 Docker Hub 的 [自动创建页面](https://registry.hub.docker.com/builds/) 中跟踪每次创建的状态。

### 私有仓库

#### 安装软件配置

Docker 官方提供了 docker registry 的构建方法  [docker-registry](https://github.com/docker/docker-registry)

##### 快速构建

快速构建 docker registry 通过以下两步:

- 安装 docker
- 运行 registry:docker run -p 5000:5000 registry

这种方法通过 Docker hub 使用官方镜像  [official image from the Docker hub](https://registry.hub.docker.com/_/registry/)

##### 不使用容器构建 registry

安装必要的软件

```
$ sudo apt-get install build-essential python-dev libevent-dev python-pip liblzma-dev
```

配置 docker-registry

```
sudo pip install docker-registry
```

或者 使用 github clone 手动安装

```
$ git clone https://github.com/dotcloud/docker-registry.git
$ cd docker-registry/
$ cp config/config_sample.yml config/config.yml
$ mkdir /data/registry -p
$ pip install .
```

运行

```
docker-registry
```

**高级启动方式 [不推荐]**

使用gunicorn控制:

```
gunicorn -c contrib/gunicorn_config.py docker_registry.wsgi:application
```

或者对外监听开放

```
gunicorn --access-logfile - --error-logfile - -k gevent -b 0.0.0.0:5000 -w 4 --max-requests 100 docker_registry.wsgi:application
```

##### 提交指定容器到私有库

```
$ docker tag ubuntu:19.04 私有库IP:5000/ubuntu:19.04
$ docker push 私有库IP:5000/ubuntu
```

更多的配置选项推荐阅读官方文档:

- [Docker-Registry README](https://github.com/docker/docker-registry/blob/master/README.md)
- [Docker-Registry advanced use](https://github.com/docker/docker-registry/blob/master/ADVANCED.md)

#### 从docker构建

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。

[`docker-registry`](https://docs.docker.com/registry/) 是官方提供的工具，可以用于构建私有的镜像仓库。

**安装运行 docker-registry**

你可以通过获取官方 `registry` 镜像来运行。

```bash
$ docker run -d -p 5000:5000 --restart=always --name registry registry
```

这将使用官方的 `registry` 镜像来启动私有仓库。默认情况下，仓库会被创建在容器的 `/var/lib/registry` 目录下。你可以通过 `-v` 参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到本地的 `/opt/data/registry` 目录。

```bash
$ docker run -d \
    -p 5000:5000 \
    -v /opt/data/registry:/var/lib/registry \
    registry
```

**在私有仓库上传、搜索、下载镜像**

创建好私有仓库之后，就可以使用 `docker tag` 来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 `127.0.0.1:5000`。

先在本机查看已有的镜像。

```bash
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

使用 `docker tag` 将 `ubuntu:latest` 这个镜像标记为 `127.0.0.1:5000/ubuntu:latest`。

格式为 `docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]`。

```bash
$ docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
127.0.0.1:5000/ubuntu:latest      latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

使用 `docker push` 上传标记的镜像。

```bash
$ docker push 127.0.0.1:5000/ubuntu:latest
The push refers to repository [127.0.0.1:5000/ubuntu]
373a30c24545: Pushed
a9148f5200b0: Pushed
cdd3de0940ab: Pushed
fc56279bbb33: Pushed
b38367233d37: Pushed
2aebd096e0e2: Pushed
latest: digest: sha256:fe4277621f10b5026266932ddf760f5a756d2facd505a94d2da12f4f52f71f5a size: 1568
```

用 `curl` 查看仓库中的镜像。

```bash
$ curl 127.0.0.1:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

这里可以看到 `{"repositories":["ubuntu"]}`，表明镜像已经被成功上传了。

先删除已有镜像，再尝试从私有仓库中下载这个镜像。

```bash
$ docker image rm 127.0.0.1:5000/ubuntu:latest

$ docker pull 127.0.0.1:5000/ubuntu:latest
Pulling repository 127.0.0.1:5000/ubuntu:latest
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete

$ docker image ls
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
127.0.0.1:5000/ubuntu:latest       latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

**注意事项**

如果你不想使用 `127.0.0.1:5000` 作为仓库地址，比如想让本网段的其他主机也能把镜像推送到私有仓库。你就得把例如 `192.168.199.100:5000` 这样的内网地址作为私有仓库地址，这时你会发现无法成功推送镜像。

这是因为 Docker 默认不允许非 `HTTPS` 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制，或者查看下一节配置能够通过 `HTTPS` 访问的私有仓库。

对于使用 `systemd` 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirror": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "192.168.199.100:5000"
  ]
}
```

> 注意：该文件必须符合 `json` 规范，否则 Docker 将不能启动。

#### 私有仓库高级配置

新建一个文件夹，以下步骤均在该文件夹中进行。

**准备站点证书**

如果你拥有一个域名，国内各大云服务商均提供免费的站点证书。你也可以使用 `openssl` 自行签发证书。

这里假设我们将要搭建的私有仓库地址为 `docker.domain.com`，下面我们介绍使用 `openssl` 自行签发 `docker.domain.com` 的站点 SSL 证书。

第一步创建 `CA` 私钥。

```bash
$ openssl genrsa -out "root-ca.key" 4096
```

第二步利用私钥创建 `CA` 根证书请求文件。

```bash
$ openssl req \
          -new -key "root-ca.key" \
          -out "root-ca.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=Your Company Name Docker Registry CA'
```

> 以上命令中 `-subj` 参数里的 `/C` 表示国家，如 `CN`；`/ST` 表示省；`/L` 表示城市或者地区；`/O` 表示组织名；`/CN` 通用名称。

第三步配置 `CA` 根证书，新建 `root-ca.cnf`。

```bash
[root_ca]
basicConstraints = critical,CA:TRUE,pathlen:1
keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
subjectKeyIdentifier=hash
```

第四步签发根证书。

```bash
$ openssl x509 -req  -days 3650  -in "root-ca.csr" \
               -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
               -extfile "root-ca.cnf" -extensions \
               root_ca
```

第五步生成站点 `SSL` 私钥。

```bash
$ openssl genrsa -out "docker.domain.com.key" 4096
```

第六步使用私钥生成证书请求文件。

```bash
$ openssl req -new -key "docker.domain.com.key" -out "site.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=docker.domain.com'
```

第七步配置证书，新建 `site.cnf` 文件。

```bash
[server]
authorityKeyIdentifier=keyid,issuer
basicConstraints = critical,CA:FALSE
extendedKeyUsage=serverAuth
keyUsage = critical, digitalSignature, keyEncipherment
subjectAltName = DNS:docker.domain.com, IP:127.0.0.1
subjectKeyIdentifier=hash
```

第八步签署站点 `SSL` 证书。

```bash
$ openssl x509 -req -days 750 -in "site.csr" -sha256 \
    -CA "root-ca.crt" -CAkey "root-ca.key"  -CAcreateserial \
    -out "docker.domain.com.crt" -extfile "site.cnf" -extensions server
```

这样已经拥有了 `docker.domain.com` 的网站 SSL 私钥 `docker.domain.com.key` 和 SSL 证书 `docker.domain.com.crt` 及 CA 根证书 `root-ca.crt`。

新建 `ssl` 文件夹并将 `docker.domain.com.key` `docker.domain.com.crt` `root-ca.crt`这三个文件移入，删除其他文件。

**配置私有仓库**

私有仓库默认的配置文件位于 `/etc/docker/registry/config.yml`，我们先在本地编辑 `config.yml`，之后挂载到容器中。

```yaml
version: 0.1
log:
  accesslog:
    disabled: true
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/auth/nginx.htpasswd
http:
  addr: :443
  host: https://docker.domain.com
  headers:
    X-Content-Type-Options: [nosniff]
  http2:
    disabled: false
  tls:
    certificate: /etc/docker/registry/ssl/docker.domain.com.crt
    key: /etc/docker/registry/ssl/docker.domain.com.key
health:
  storagedriver:
    enabled: true
    interval: 10s
threshold: 3
```

**生成 http 认证文件**

```bash
$ mkdir auth

$ docker run --rm \
    --entrypoint htpasswd \
    registry \
    -Bbn username password > auth/nginx.htpasswd
```

> 将上面的 `username` `password` 替换为你自己的用户名和密码。

**编辑 docker-compose.yml**

```yaml
version: '3'

services:
  registry:
    image: registry
    ports:
      - "443:443"
    volumes:
      - ./:/etc/docker/registry
      - registry-data:/var/lib/registry

volumes:
  registry-data:
```

**修改 hosts**

编辑 `/etc/hosts`

```bash
127.0.0.1 docker.domain.com
```

**启动**

```bash
$ docker-compose up -d
```

这样我们就搭建好了一个具有权限认证、TLS 的私有仓库，接下来我们测试其功能是否正常。

**测试私有仓库功能**

由于自行签发的 CA 根证书不被系统信任，所以我们需要将 CA 根证书 `ssl/root-ca.crt` 移入 `/etc/docker/certs.d/docker.domain.com` 文件夹中。

```bash
$ sudo mkdir -p /etc/docker/certs.d/docker.domain.com

$ sudo cp ssl/root-ca.crt /etc/docker/certs.d/docker.domain.com/ca.crt
```

登录到私有仓库。

```bash
$ docker login docker.domain.com
```

尝试推送、拉取镜像。

```bash
$ docker pull ubuntu:18.04

$ docker tag ubuntu:18.04 docker.domain.com/username/ubuntu:18.04

$ docker push docker.domain.com/username/ubuntu:18.04

$ docker image rm docker.domain.com/username/ubuntu:18.04

$ docker pull docker.domain.com/username/ubuntu:18.04
```

如果我们退出登录，尝试推送镜像。

```bash
$ docker logout docker.domain.com

$ docker push docker.domain.com/username/ubuntu:18.04

no basic auth credentials
```

发现会提示没有登录，不能将镜像推送到私有仓库中。

**注意事项**

如果你本机占用了 `443` 端口，你可以配置 [Nginx 代理](https://docs.docker.com/registry/recipes/nginx/)，这里不再赘述。

#### Nexus3.x 的私有仓库

使用 Docker 官方的 Registry 创建的仓库面临一些维护问题。比如某些镜像删除以后空间默认是不会回收的，需要一些命令去回收空间然后重启 Registry 程序。在企业中把内部的一些工具包放入 Nexus 中是比较常见的做法，最新版本 `Nexus3.x` 全面支持 Docker 的私有镜像。所以使用 [`Nexus3.x`](https://www.sonatype.com/download-oss-sonatype/) 一个软件来管理 `Docker` , `Maven` , `Yum` , `PyPI` 等是一个明智的选择。

查找镜像

```
docker search nexus
```

选取使用次数较多是镜像 拉取镜像

```
docker pull sonatype/nexus3
```

查看拉取的镜像

```
docker images
```

启动容器

```
$ docker volume create nexus_data
$ docker run -d --name nexus3 \
 --restart=always \
-p 8081:8081 \
-p 8082:8082  \
-p 8083:8083  \
-p 8084:8084  \
-v nexus_data:/nexus-data \
sonatype/nexus3
```

映射端口对应的用途：

- 8081：nexus3网页端
- 8082：docker(hosted)私有仓库，可以pull和push
- 8083：docker(proxy)代理远程仓库，只能pull
- 8084：docker(group)私有仓库和代理的组，只能pull

查看容器日志

```
docker logs -f nexus3
```

确保正常启动后 使用浏览器访问http://服务器ip:8081

点击右上角登录 账号密码：admin/admin123

**创建仓库**

创建一个私有仓库的方法： `Repository->Repositories` 点击右边菜单 `Create repository` 选择 `docker (hosted)`

- Name: 仓库的名称
- HTTP: 仓库单独的访问端口
- Enable Docker V1 API: 如果需要同时支持 V1 版本请勾选此项（不建议勾选）。
- Hosted -> Deployment pollcy: 请选择 Allow redeploy 否则无法上传 Docker 镜像。

其它的仓库创建方法请各位自己摸索，还可以创建一个 docker (proxy) 类型的仓库链接到 DockerHub 上。再创建一个 docker (group) 类型的仓库把刚才的 hosted 与 proxy 添加在一起。主机在访问的时候默认下载私有仓库中的镜像，如果没有将链接到 DockerHub 中下载并缓存到 Nexus 中。

**添加访问权限**

菜单 `Security->Realms` 把 Docker Bearer Token Realm 移到右边的框中保存。

添加用户规则：菜单 `Security->Roles`->`Create role` 在 `Privlleges` 选项搜索 docker 把相应的规则移动到右边的框中然后保存。

添加用户：菜单 `Security->Users`->`Create local user` 在 `Roles` 选项中选中刚才创建的规则移动到右边的窗口保存。

**NGINX 加密代理**

证书的生成请参见上面证书生成

NGINX 示例配置如下

```nginx
upstream register
{
    server "YourHostName OR IP":5001; #端口为上面添加的私有镜像仓库是设置的 HTTP 选项的端口号
    check interval=3000 rise=2 fall=10 timeout=1000 type=http;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_4xx;
}

server {
    server_name YourDomainName;#如果没有 DNS 服务器做解析，请删除此选项使用本机 IP 地址访问
    listen       443 ssl;
    
    ssl_certificate key/example.crt;
    ssl_certificate_key key/example.key;
    
    ssl_session_timeout  5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;
    large_client_header_buffers 4 32k;
    client_max_body_size 300m;
    client_body_buffer_size 512k;
    proxy_connect_timeout 600;
    proxy_read_timeout   600;
    proxy_send_timeout   600;
    proxy_buffer_size    128k;
    proxy_buffers       4 64k;
    proxy_busy_buffers_size 128k;
    proxy_temp_file_write_size 512k;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://register;
        proxy_read_timeout 900s;

    }
    error_page   500 502 503 504  /50x.html;
}
```

**连接仓库**

> 其他机器需要连接仓库才能进行push、pull等操作

如果不启用 SSL 加密可以通过前面的方法添加信任地址到 Docker 的配置文件中然后重启 Docker

使用 SSL 加密以后程序需要访问就不能采用修改配置的访问了。具体方法如下：

```bash
$ openssl s_client -showcerts -connect YourDomainName OR HostIP:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >ca.crt
$ cat ca.crt | sudo tee -a /etc/ssl/certs/ca-certificates.crt
$ systemctl restart docker
```

使用 `docker login YourDomainName OR HostIP` 进行测试，用户名密码填写上面 Nexus 中生成的。



连接仓库前需要进行配置 vim /etc/docker/daemon.json

```
    {
    "insecure-registries": ["172.16.77.71:8082" ]
    }
    
    systemctl daemon-reload
    systemctl restart docker
```

登录仓库

```
docker login -u admin -p admin123 172.16.77.71:8082  #注意这里的端口是配置仓库时选择的端口号
```

上传镜像

```
docker tag nginx:latest 172.16.77.71:8082/nginx:0.1
docker push 172.16.77.71:8082/nginx:0.1
```

拉取镜像

```
docker pull 172.16.77.71:8082/nginx:0.1
```

搜索镜像

```
[root@k8s-77-40 torch]# docker search 172.16.77.71:8082/nginx
NAME                          DESCRIPTION         STARS               OFFICIAL              AUTOMATED
172.16.77.71:8082/nginx:0.1                       0
```

**总结**

到此，使用nexus搭建的docker私有仓库配置完毕。公司常用的镜像可以存放在私有仓库里 毕竟官方的[dockerhub](https://hub.docker.com/)太慢

#### harbor搭建

[harbor git 地址](https://github.com/goharbor/harbor)

优点：

1. 本身自代 docker 私有仓库
2. 支持基于角色的权限管理
3. 支持 LDAP

harbor支持k8s的helm安装和本地安装，我这次先择的安装方式是本地安装。

1. 前置条件

   docker、docker-compose

2. 直接选择编译好的包

   有两个包`Harbor offline installer` 和 `Harbor online installer`，两者的区别的是 `Harbor offline installer` 里就包含的 Harbor 需要使用的镜像文件。

   下载成功，并解压

   ```shell
   tar -zxvf harbor-offline-installer-v1.7.1.tgz
   ```

   进入解压的目录，并 `ls`

3. 编辑配置文件

   `harbor.cfg` 是这个项目的配置文件

   将 hostname 改成你本机的网址或IP

   ```
   hostname = A.B.C.D  # 写你自己的网址或IP，公网访问要写公网IP
   ```

   支持Http 访问

   ```
   customize_crt = false
   ```

4. 运行

   修改完配置文件后，运行 `./prepare`，它会哪所配置文件修改一文件

   运行 `./install.sh`

   运行成功，`docker ps` 查看，可以看到服务已经起来了。

5. 常用管理命令

   - 停止服务： `docker-compose stop`
   - 开始服务： `docker-compose start`

完成了私有库的搭建后，可以再安装一个k8s集群后台管理系统（wayne系统介绍）。

#### wayne

在此首先要感谢360公司和这个项目开发者的奉献，并为这个项目点个星星。

> Wayne 是一个通用的、基于 Web 的 Kubernetes 多集群管理平台。通过可视化 Kubernetes 对象模板编辑的方式，降低业务接入成本， 拥有完整的权限管理系统，适应多租户场景，是一款适合企业级集群使用的发布平台。

[wayne项目git地址](https://github.com/Qihoo360/wayne/blob/master/README-CN.md)

搭建的过程这里就不再说了，git 里说的已经很明白了。

**界面**

系统分成两部分：

- 前台部分：负责项目的部署和发布
- 后台部分：负责k8s集群的配置、项目的创建等

通过右上角的选项可以进行切换

### 问题

#### [解决：Docker 启动的容器内部时间比服务器时间晚 8 小时，容器内部时间与宿主机时间不一致](https://www.centos.bz/2019/07/解决：docker-启动的容器内部时间比服务器时间晚-8-小时/)

1.[docker](https://www.centos.bz/tag/docker/) 方式启动容器 nexus3 ，运行正常，但查日志时发现容器时间比宿主机时间晚8小时，内外时间不一致。

2.解决方法：启动容器时加入时间挂载，使用宿主机时间：

```
 -v /etc/localtime:/etc/localtime:ro
```

如： 启动 nexus3 的完整命令为：

```
docker run -tid -p 8081:8081 --privileged=true --name nexus3 -v $PWD/nexus-data:/var/nexus-data  -v /etc/localtime:/etc/localtime:ro --restart=always docker.io/sonatype/nexus3
```

可以用 [zdump](http://askubuntu.com/questions/31842/how-to-read-time-zone-information) 命令来读取 /etc/localtime 文件的内容：

```
zdump -v /etc/localtime
```

[/etc/timezone 的内容是时区](https://wiki.debian.org/TimeZoneChanges)，比如 Asia/ShangHai。

#### 降低Docker镜像大小

1. [7 步精简 Docker 镜像几百MB](https://www.centos.bz/2019/07/7-步精简-docker-镜像几百mb/)
2. [Docker 的 Image 太大，怎么变小？](https://www.centos.bz/2019/07/docker-的-image-太大，怎么变小？/)
3. https://www.jianshu.com/p/77af52a75ad8