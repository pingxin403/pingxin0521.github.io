---
title: nginx 安装
date: 2019-08-20 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

### 前言

没有听过Nginx？那么一定听过它的"同行"Apache吧！Nginx同Apache一样都是一种WEB服务器。基于REST架构风格，以统一资源描述符(Uniform Resources Identifier)URI或者统一资源定位符(Uniform Resources Locator)URL作为沟通依据，通过HTTP协议提供各种网络服务。

<!--more-->

然而，这些服务器在设计之初受到当时环境的局限，例如当时的用户规模，网络带宽，产品特点等局限并且各自的定位和发展都不尽相同。这也使得各个WEB服务器有着各自鲜明的特点。

Apache的发展时期很长，而且是毫无争议的世界第一大服务器。它有着很多优点：稳定、开源、跨平台等等。它出现的时间太长了，它兴起的年代，互联网产业远远比不上现在。所以它被设计为一个重量级的。它不支持高并发的服务器。在Apache上运行数以万计的并发访问，会导致服务器消耗大量内存。操作系统对其进行进程或线程间的切换也消耗了大量的CPU资源，导致HTTP请求的平均响应速度降低。

这些都决定了Apache不可能成为高性能WEB服务器，轻量级高并发服务器Nginx就应运而生了。

俄罗斯的工程师Igor Sysoev，他在为Rambler Media工作期间，使用C语言开发了Nginx。Nginx作为WEB服务器一直为Rambler Media提供出色而又稳定的服务。

然后呢，Igor Sysoev将Nginx代码开源，并且赋予自由软件许可证。

由于：

- Nginx使用基于事件驱动架构，使得其可以支持数以百万级别的TCP连接
- 高度的模块化和自由软件许可证使得第三方模块层出不穷（这是个开源的时代啊~）
- Nginx是一个跨平台服务器，可以运行在Linux，Windows，FreeBSD，Solaris，AIX，Mac OS等操作系统上
- 这些优秀的设计带来的是极大的稳定性

所以，Nginx火了！

**Nginx相关地址**

源码：https://trac.nginx.org/nginx/browser

官网：http://www.nginx.org/

中文文档：<http://tengine.taobao.org/nginx_docs/cn/docs/>

### 功能

Nginx是一款自由的、开源的、高性能的HTTP服务器和反向代理服务器；同时也是一个IMAP、POP3、SMTP代理服务器；Nginx可以作为一个HTTP服务器进行网站的发布处理，另外Nginx可以作为反向代理进行负载均衡的实现。

![image.png](https://i.loli.net/2019/10/14/7xEAFOfmRSpKG58.png)

**关于代理**

说到代理，首先我们要明确一个概念，所谓代理就是一个代表、一个渠道；

此时就涉及到两个角色，一个是被代理角色，一个是目标角色，被代理角色通过这个代理访问目标角色完成一些任务的过程称为代理操作过程；如同生活中的专卖店,客人到adidas专卖店买了一双鞋，这个专卖店就是代理，被代理角色就是adidas厂家，目标角色就是用户。

#### 正向代理

说反向代理之前，我们先看看正向代理，正向代理也是大家最常接触的到的代理模式，我们会从两个方面来说关于正向代理的处理模式，分别从软件方面和生活方面来解释一下什么叫正向代理。

在如今的网络环境下，我们如果由于技术需要要去访问国外的某些网站，此时你会发现位于国外的某网站我们通过浏览器是没有办法访问的，此时大家可能都会用一个操作FQ进行访问，FQ的方式主要是找到一个可以访问国外网站的代理服务器，我们将请求发送给代理服务器，代理服务器去访问国外的网站，然后将访问到的数据传递给我们！

上述这样的代理模式称为正向代理，正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息。

![uL0RMR.png](https://s2.ax1x.com/2019/10/12/uL0RMR.png)

客户端必须设置正向代理服务器，当然前提是要知道正向代理服务器的IP地址，还有代理程序的端口。

总结来说：**正向代理，"它代理的是客户端，代客户端发出请求"，是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。**

正向代理的用途：

（1）访问原来无法访问的资源，如Google
（2） 可以做缓存，加速访问资源
（3）对客户端访问授权，上网进行认证
（4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

#### 反向代理

明白了什么是正向代理，我们继续看关于反向代理的处理方式，举例如我大天朝的某宝网站，每天同时连接到网站的访问人数已经爆表，单个服务器远远不能满足人民日益增长的购买欲望了，此时就出现了一个大家耳熟能详的名词：分布式部署；也就是通过部署多台服务器来解决访问人数限制的问题；某宝网站中大部分功能也是直接使用Nginx进行反向代理实现的，并且通过封装Nginx和其他的组件之后起了个高大上的名字：Tengine，有兴趣的童鞋可以访问Tengine的官网查看具体的信息：http://tengine.taobao.org/。那么反向代理具体是通过什么样的方式实现的分布式的集群操作呢，我们先看一个示意图（我把服务器和反向代理框在一块，同属于一个环境，后面我有介绍）：

![uLBRfg.png](https://s2.ax1x.com/2019/10/12/uLBRfg.png)

通过上述的图解大家就可以看清楚了，多个客户端给服务器发送的请求，Nginx服务器接收到之后，按照一定的规则分发给了后端的业务处理服务器进行处理了。此时~请求的来源也就是客户端是明确的，但是请求具体由哪台服务器处理的并不明确了，Nginx扮演的就是一个反向代理角色。

客户端是无感知代理的存在的，反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。因为客户端不需要任何配置就可以访问。

反向代理，"它代理的是服务端，代服务端接收请求"，主要用于服务器集群分布式部署的情况下，反向代理隐藏了服务器的信息。

反向代理的作用：
（1）保证内网的安全，通常将反向代理作为公网访问地址，Web服务器是内网
（2）负载均衡，通过反向代理服务器来优化网站的负载

**项目场景**

通常情况下，我们在实际项目操作时，正向代理和反向代理很有可能会存在在一个应用场景中，正向代理代理客户端的请求去访问目标服务器，目标服务器是一个反向单利服务器，反向代理了多台真实的业务处理服务器。具体的拓扑图如下：


![uLB2tS.png](https://s2.ax1x.com/2019/10/12/uLB2tS.png)

#### 二者区别

截了一张图来说明正向代理和反向代理二者之间的区别，如图。
![uL0gz9.png](https://s2.ax1x.com/2019/10/12/uL0gz9.png)

图解：

在正向代理中，Proxy和Client同属于一个LAN（图中方框内），隐藏了客户端信息；

在反向代理中，Proxy和Server同属于一个LAN（图中方框内），隐藏了服务端信息；

实际上，Proxy在两种代理中做的事情都是替服务器代为收发请求和响应，不过从结构上看正好左右互换了一下，所以把后出现的那种代理方式称为反向代理了。

#### 负载均衡

我们已经明确了所谓代理服务器的概念，那么接下来，Nginx扮演了反向代理服务器的角色，它是以依据什么样的规则进行请求分发的呢？不用的项目应用场景，分发的规则是否可以控制呢？

这里提到的客户端发送的、Nginx反向代理服务器接收到的请求数量，就是我们说的负载量。

请求数量按照一定的规则进行分发到不同的服务器处理的规则，就是一种均衡规则。

所以~将服务器接收到的请求按照规则分发的过程，称为负载均衡。

负载均衡在实际项目操作过程中，有硬件负载均衡和软件负载均衡两种，硬件负载均衡也称为硬负载，如F5负载均衡，相对造价昂贵成本较高，但是数据的稳定性安全性等等有非常好的保障，如中国移动中国联通这样的公司才会选择硬负载进行操作；更多的公司考虑到成本原因，会选择使用软件负载均衡，软件负载均衡是利用现有的技术结合主机硬件实现的一种消息队列分发机制。

![uLDuHP.png](https://s2.ax1x.com/2019/10/12/uLDuHP.png)

Nginx支持的负载均衡调度算法方式如下：

1. weight轮询(默认，常用)：接收到的请求按照权重分配到不同的后端服务器，即使在使用过程中，某一台后端服务器宕机，Nginx会自动将该服务器剔除出队列，请求受理情况不会受到任何影响。 这种方式下，可以给不同的后端服务器设置一个权重值(weight)，用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的。
2. ip_hash（常用）：每个请求按照发起客户端的ip的hash结果进行匹配，这样的算法下一个固定ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题。
3. fair：智能调整调度算法，动态的根据后端服务器的请求处理到响应的时间进行均衡分配，响应时间短处理效率高的服务器分配到请求的概率高，响应时间长处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是Nginx默认不支持fair算法，如果要使用这种调度算法，请安装upstream_fair模块。
4. url_hash：按照访问的url的hash结果分配请求，每个请求的url会指向后端固定的某个服务器，可以在Nginx作为静态服务器的情况下提高缓存效率。同样要注意Nginx默认不支持这种调度算法，要使用的话需要安装Nginx的hash软件包。

#### 几种常用web服务器对比

| **对比项\服务器** | **Apache** | **Nginx** | **Lighttpd** |
| ----------------- | ---------- | --------- | ------------ |
| Proxy代理         | 非常好     | 非常好    | 一般         |
| Rewriter          | 好         | 非常好    | 一般         |
| Fcgi              | 不好       | 好        | 非常好       |
| 热部署            | 不支持     | 支持      | 不支持       |
| 系统压力          | 很大       | 很小      | 比较小       |
| 稳定性            | 好         | 非常好    | 不好         |
| 安全性            | 好         | 一般      | 一般         |
| 静态文件处理      | 一般       | 非常好    | 好           |
| 反向代理          | 一般       | 非常好    | 一般         |

### 安装

#### 安装包安装

系统平台：CentOS Linux 7 (Core) x86_64。

基础环境准备:

```bash
#确认系统⽹络
[root@Nginx	~]#	ping	baidu.com
#关闭firewalld
[root@Nginx	~]#	systemctl	stop firewalld
[root@Nginx	~]#	systemctl	disable firewalld
#临时关闭selinux
[root@Nginx	~]#	setenforce	0
#初始化基本⽬录
[root@Nginx	~]#	yum install -y gcc gcc-c++ autoconf \
pcre pcre-devel make automake wget httpd-tools vim tree
#配置Nginx官⽅Yum源
[root@Nginx	~]#	vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx_repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
#安装Nginx
[root@Nginx	~]#	yum install nginx -y
#查看Nginx当前版本
[root@Nginx	~]#	nginx -v
nginx	version:	nginx/1.12.2

```

**Nginx安装目录**

为了让大家更清晰的了解 Nginx 软件的全貌，有必要介绍下 Nginx 安装后整体的目录结构及文件功能

```
[root@Nginx	~]#	rpm -ql nginx
```

如下表格对 Nginx 安装目录做详细概述

![image.png](https://i.loli.net/2019/10/12/tPLiIXvKjyYkZUH.png)

#### 编译安装

系统平台：Ubuntu 18.04。

**安装编译工具及库文件**

```
useradd -s /sbin/nologin -M www
mkdir /opt/server
apt-get install build-essential libtool gcc automake autoconf make php php-cgi

```

**安装依赖插件**

1. 首先要安装 PCRE

   PCRE 作用是让 Nginx 支持 Rewrite 功能。

   ```bash
   cd /usr/local/src/
   wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
   tar zxvf pcre-8.35.tar.gz
   cd pcre-8.35
   ./configure  --enable-utf8 --enable-unicode-properties
   # --enable-utf8 --enable-unicode-properties 支持中文正则
   make && sudo make install
   pcre-config --version
   ```

   

2. 安装zlib, 支持gzip压缩

   ```
   wget http://zlib.net/zlib-1.2.11.tar.gz
   tar -zxvf zlib-1.2.11.tar.gz
   cd zlib-1.2.11
   ./configure
   make && sudo make install
   ```

3. 安装ssl

   ```
   wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
   tar -zxvf openssl-1.1.1g.tar.gz
   cd openssl-1.1.1g
   ./config
   make && sudo make install
   ```

**安装 Nginx**

1下载 Nginx，下载地址：http://nginx.org/download/

```bash
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/opt/server/nginx --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module
make &&  make install
```

编译选项

![image.png](https://i.loli.net/2019/10/12/fp9lYHSuiDbmANa.png)



5、查看nginx版本

```
sudo ln -s /opt/server/nginx/sbin/nginx  /usr/sbin/nginx
nginx -v
```

到此，nginx安装完成。

6、增加vim支持

```
mkdir ~/.vim
cp -r contrib/vim/* ~/.vim
```

**Nginx 配置**

创建 Nginx 运行使用的用户 www：

```bash
sudo groupadd www 
sudo useradd -g www www
mkdir /www/html -p
echo "hello world" >> /www/html/index.html
echo "<?php echo phpinfo();?>" >> /www/html/index.php
```

配置nginx.conf ，将/opt/server/nginx/conf/nginx.conf替换为以下内容

```nginx
$ cat /opt/server/nginx/conf/nginx.conf

#user root;
worker_processes  auto; #设置值和CPU核心数一致
error_log logs/nginx_error.log crit; #日志位置和日志级别
pid nginx.pid;
#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;
events
{
  use epoll;
  worker_connections 65535;
}
http
{
  include mime.types;
  default_type application/octet-stream;
  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';
  
#charset gb2312;
     
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;
     
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 60;
  tcp_nodelay on;
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  gzip on; 
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;
 
  #limit_zone crawler $binary_remote_addr 10m;
 #下面是server虚拟主机的配置
 server
  {
    listen 8080;#监听端口
    server_name localhost;#域名
    index index.html index.htm index.php;
    root /www/html;#站点目录
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
    {
      expires 30d;
  # access_log off;
    }
	# PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI协议默认配置.
        # Fastcgi服务器和程序(PHP,Python)沟通的协议.
        location ~ \.php$ {
            # 设置监听端口
            fastcgi_pass   127.0.0.1:9000;
            # 设置nginx的默认首页文件(上面已经设置过了，可以删除)
            fastcgi_index  index.php;
            # 设置脚本文件请求的路径
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            # 引入fastcgi的配置文件
            include        fastcgi_params;
        }
    location ~ .*\.(js|css)?$
    {
      expires 15d;
   # access_log off;
    }
    access_log off;
  }
	include conf/extra/*.conf;
}
```

检查配置文件nginx.conf的正确性命令：

```bash
nginx -t -c /opt/server/nginx/conf/nginx.conf
#加载配置文件
nginx -s reload -c /opt/server/nginx/conf/nginx.conf
```

**启动 Nginx**

Nginx 启动命令如下：

```
nginx  -c /opt/server/nginx/conf/nginx.conf
```

![ttLRDs.png](https://s1.ax1x.com/2020/06/02/ttLRDs.png)

查看php页面：

```
php-cgi -b 127.0.0.1:9000
```

![ttxr6O.png](https://s1.ax1x.com/2020/06/02/ttxr6O.png)

**Nginx 其他命令**

以下包含了 Nginx 常用的几个命令：

```
nginx -s reload            # 重新载入配置文件
nginx -s reopen            # 重启 Nginx
nginx -s stop              # 停止 Nginx
kill -HUP `cat /opt/nginx/nginx.pid` #平滑重启
```

**Nginx常用模块**
![image.png](https://i.loli.net/2019/10/12/W6wzyZmcKtLhAik.png)

#### Docker 安装 Nginx

```
$ docker pull nginx
```

等待下载完成后，我们就可以在本地镜像列表里查到 REPOSITORY 为 nginx 的镜像。

```
$ docker images nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              e445ab08b2be        6 days ago          126MB

```

以下命令使用 NGINX 默认的配置来启动一个 Nginx 容器实例：

```
$ docker run --name nginx-test -p 8081:80 -d nginx
```

- `runoob-nginx-test` 容器名称。
- the `-d`设置容器在在后台一直运行。
- the `-p` 端口进行映射，将本地 8081 端口映射到容器内部的 80 端口。

执行以上命令会生成一串字符串，类似 **6dd4380ba70820bd2acc55ed2b326dd8c0ac7c93f68f0067daecad82aef5f938**，这个表示容器的 ID，一般可作为日志的文件名。

我们可以使用 docker ps 命令查看容器是否有在运行：

```
$ docker ps
CONTAINER ID        IMAGE        ...               PORTS                  NAMES
6dd4380ba708        nginx        ...      0.0.0.0:8081->80/tcp   nginx-test
```

PORTS 部分表示端口映射，本地的 8081 端口映射到容器内部的 80 端口。

在浏览器中打开 **http://127.0.0.1:8081/**

### 参考

1. [Nginx 一个高性能的HTTP和反向代理服务器](https://www.cnblogs.com/wcwnina/p/9946747.html)
2. [Nginx 安装与部署配置以及Nginx和uWSGI开机自启](https://www.cnblogs.com/wcwnina/p/8728430.html)
3. [基于Linux和Nginx的Django部署](https://www.cnblogs.com/wcwnina/p/9906081.html)
4. [Nginx 相关介绍(Nginx是什么?能干嘛?)](https://www.cnblogs.com/wcwnina/p/8728391.html)
5. [Centos 7安装Nginx+PHP+MariaDB环境搭建WordPress博客](https://www.centos.bz/2018/11/centos-7安装nginxphpmariadb环境搭建wordpress博客/)

### 附录

nginx衍生版本：[tengine](https://github.com/alibaba/tengine)和[openresty](https://github.com/agentzh/ngx_openresty)

#### OpenResty

OpenResty是一个基于 [Nginx](http://openresty.org/cn/nginx.html) 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

OpenResty通过汇聚各种设计精良的 [Nginx](http://openresty.org/cn/nginx.html) 模块（主要由 OpenResty 团队自主开发），从而将 [Nginx](http://openresty.org/cn/nginx.html) 有效地变成一个强大的通用 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 [Nginx](http://openresty.org/cn/nginx.html) 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。

OpenResty的目标是让你的Web服务直接跑在 [Nginx](http://openresty.org/cn/nginx.html) 服务内部，充分利用 [Nginx](http://openresty.org/cn/nginx.html) 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。

参考 [组件](http://openresty.org/cn/components.html) 可以知道 OpenResty中包含了多少软件。

参考 [上路](http://openresty.org/cn/getting-started.html) 学习如何从最简单的 hello world 开始使用 OpenResty® 开发 HTTP 业务，或前往 [下载](http://openresty.org/cn/download.html) 直接获取 OpenResty的源代码包开始体验。

**二进制安装**

参考：<http://openresty.org/cn/linux-packages.html>

**编译安装**

依赖：

```
yum install pcre-devel openssl-devel gcc curl -y
```

编译安装

```

# 下载编译
 wget https://openresty.org/download/openresty-1.15.8.2.tar.gz
 tar zxf openresty-1.15.8.2.tar.gz
 cd openresty-1.15.8.2
# 编译
./configure \
--prefix=/opt/openresty \
--with-pcre-jit \
--without-http_redis2_module \
--with-http_iconv_module \
--with-stream \
--with-luajit \
--with-http_stub_status_module \
--with-http_gzip_static_module
make & make install
```

#### tengine

Tengine是由淘宝网发起的Web服务器项目。它在[Nginx](http://nginx.org/)的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如[淘宝网](http://www.taobao.com/)，[天猫商城](http://www.tmall.com/)等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。

从2011年12月开始，Tengine成为一个开源项目，Tengine团队在积极地开发和维护着它。Tengine团队的核心成员来自于[淘宝](http://www.taobao.com/)、[搜狗](http://www.sogou.com/)等互联网企业。

[下载](http://tengine.taobao.org/download/tengine-2.3.2.tar.gz)

**特性**

- 继承Nginx-1.17.3的所有特性，兼容Nginx的配置；
- 支持HTTP的[CONNECT](http://tengine.taobao.org/document_cn/proxy_connect_cn.html)方法，可用于正向代理场景；
- [支持异步OpenSSL](http://tengine.taobao.org/document_cn/ngx_http_ssl_asynchronous_mode_cn.html)，可使用硬件如:[QAT](http://tengine.taobao.org/document_cn/tengine_qat_ssl_cn.html)进行HTTPS的加速与卸载；
- 增强相关运维、监控能力,比如[异步打印日志及回滚](http://tengine.taobao.org/document_cn/ngx_log_pipe_cn.html),[本地DNS缓存](http://tengine.taobao.org/document_cn/core_cn.html),[内存监控](http://tengine.taobao.org/document_cn/ngx_debug_pool_cn.html)等；
- Stream模块支持[server_name](http://tengine.taobao.org/document_cn/stream_sni_cn.html)指令；
- 更加强大的负载均衡能力，包括[一致性hash模块](http://tengine.taobao.org/document_cn/http_upstream_consistent_hash_cn.html)、[会话保持模块](http://tengine.taobao.org/document_cn/http_upstream_session_sticky_cn.html)，[还可以对后端的服务器进行主动健康检查](http://tengine.taobao.org/document_cn/http_upstream_check_cn.html)，根据服务器状态自动上线下线，以及[动态解析upstream中出现的域名](http://tengine.taobao.org/document_cn/http_upstream_dynamic_cn.html)；
- [输入过滤器机制](http://blog.zhuzhaoyuan.com/2012/01/a-mechanism-to-help-write-web-application-firewalls-for-nginx/)支持。通过使用这种机制Web应用防火墙的编写更为方便；
- 支持设置proxy、memcached、fastcgi、scgi、uwsgi[在后端失败时的重试次数](http://tengine.taobao.org/document_cn/ngx_limit_upstream_tries_cn.html)
- [动态脚本语言Lua](https://github.com/alibaba/tengine/blob/master/modules/ngx_http_lua_module/README.markdown)支持。扩展功能非常高效简单；
- 支持按指定关键字(域名，url等)[收集Tengine运行状态](http://tengine.taobao.org/document_cn/http_reqstat_cn.html)；
- [组合多个CSS、JavaScript文件的访问请求变成一个请求](http://tengine.taobao.org/document_cn/http_concat_cn.html)；
- [自动去除空白字符和注释](http://tengine.taobao.org/document_cn/http_trim_filter_cn.html)从而减小页面的体积
- 自动根据CPU数目设置进程个数和绑定CPU亲缘性；
- [监控系统的负载和资源占用从而对系统进行保护](http://tengine.taobao.org/document_cn/http_sysguard_cn.html)；
- [显示对运维人员更友好的出错信息，便于定位出错机器；](http://tengine.taobao.org/document_cn/http_footer_filter_cn.html)
- [更强大的防攻击（访问速度限制）模块](http://tengine.taobao.org/document_cn/http_limit_req_cn.html)；
- [更方便的命令行参数，如列出编译的模块列表、支持的指令等](http://tengine.taobao.org/document_cn/commandline_cn.html)；
- 可以根据访问文件类型设置过期时间；
- ……

**安装**

安装依赖：

```
yum install gcc-devel openssl-devel zlib-devel pcre-devel
```

预编译：

```
$ wget http://tengine.taobao.org/download/tengine-2.3.0.tar.gz
$ tar -xf tengine-2.3.0.tar.gz 
$ cd tengine-2.3.0
$ ./configure --prefix=/opt/tengine --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre 
$ make && make install
```