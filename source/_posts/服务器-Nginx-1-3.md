---
title: nginx 架构
date: 2019-08-28 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

### nginx请求处理流程

![UTOOLS1572180522734.png](https://i.loli.net/2019/10/27/SiaO1wKGvVBj6sP.png)

<!--more-->

大致有三种流量，进入nginx中，nginx中有三种大的状态机，处理tcp/udp的传输层状态机，处理应用层的http状态机，处理邮件的mail状态机(叫状态机是因为nginx中核心的绿色的框是用非阻塞的事件驱动处理引擎，一旦使用这种异步处理引擎后，通常要用状态机来把请求正确的识别和处理)，基于这样的事件状态处理机，我们在解析出请求需要访问静态资源时，可以看到走左下方的箭头去找静态资源，做反向代理时，对反向代理的内容也可以做磁盘缓存，但是当内存不足以缓存静态资源信息时，会退化成阻塞的线程调用，所以要有线程池来进行处理，对于每一个处理完成的请求，我们会记录access日志和error日志。更多的时候，nginx作为负载均衡和反向代理来使用，这时我们可以把请求通过协议传输到后面的服务器，也可以通过应用层的一些协议代理到响应的应用服务器

### Nginx进程模型

![UTOOLS1572180872702.png](https://i.loli.net/2019/10/27/hcoSFsuZiDkqy5a.png)

- 多进程而非多线程，保证健壮性
- Master进程为父进程，Master 进程管理worker进程，**父子进程之间通过信号进行通信**
- cache 需要在多个worker进程间共享，还要被cache loader和cache manager 使用
- cache loader 缓存的载入，cache manager缓存的管理。**worker进程间通信使用共享内存**
- worker进程数量应和cpu内核数相同，每个worker进程绑定一个cpu核，保证在运行过程中worker进程使用cpu核上的缓存

#### nginx进程管理：信号

![UTOOLS1572182414469.png](https://i.loli.net/2019/10/27/hwWpol4CHGs7QfM.png)

- 当worker进程死亡时会向master进程发送CHLD信号，然后master进程依据此信号创建新的worker进程

- master进程通过接受信号来管理worker进程：

  可以通过命令控制：TERM/INT（立刻停止进程，nginx -s stop）、QUIT（优雅的停止，nginx -s quit）、HUP（重载配置文件，nginx -s reload）、USER1（重新打开日志文件，日志文件切割，nginx -s reopen）

  只能通过kill命令发送信号：USER2、WINCH(关闭旧进程，用于nginx升级时配置使用) 

- 推荐由master进程来管理worker进程的状态

- 调用命令行跟直接向master进程发送信号的效果是一样的

![lE1tRx.png](https://s2.ax1x.com/2019/12/26/lE1tRx.png)

1. 在创建master进程时，先建立需要监听的socket（listenfd），然后从master进程中fork()出多个worker进程，如此一来每个worker进程都可以监听用户请求的socket。一般来说，当一个连接进来后，所有Worker都会收到通知，但是只有一个进程可以接受这个连接请求，其它的都失败，这是所谓的惊群现象。nginx提供了一个**accept_mutex（互斥锁）**，有了这把锁之后，同一时刻，就只会有一个进程在accpet连接，这样就不会有惊群问题了。

2. 先打开accept_mutex选项，只有获得了accept_mutex的进程才会去添加accept事件。nginx使用一个叫ngx_accept_disabled的变量来控制是否去竞争accept_mutex锁。**ngx_accept_disabled** = nginx单进程的所有连接总数 / 8 -空闲连接数量，当ngx_accept_disabled大于0时，不会去尝试获取accept_mutex锁，ngx_accept_disable越大，让出的机会就越多，这样其它进程获取锁的机会也就越大。不去accept，每个worker进程的连接数就控制下来了，其它进程的连接池就会得到利用，这样，nginx就控制了多进程间连接的平衡。

3. **每个worker进程都有一个独立的连接池**，连接池的大小是worker_connections。这里的连接池里面保存的其实不是真实的连接，它只是一个worker_connections大小的一个ngx_connection_t结构的数组。并且，nginx会通过一个链表free_connections来保存所有的空闲ngx_connection_t，每次获取一个连接时，就从空闲连接链表中获取一个，用完后，再放回空闲连接链表里面。一个nginx能建立的最大连接数，应该是`worker_connections * worker_processes`。当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是`worker_connections * worker_processes`，而如果是HTTP作为反向代理来说，最大并发数量应该是`worker_connections * worker_processes/2`。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

   相关的配置：

   ```nginx
   worker_processes  1; // 工作进程数，建议设成CPU总核心数。
   events { // 多路复用IO模型机制，epoll . select  ....根据操作系统不同来选择。linux 默认epoll
       use epoll; //io 模型
       worker_connections  1024; // 每个woker进程的最大连接数，数值越大，并发量允许越大
   }
   http{
   　　sendfile  on;//开启零拷贝
   }
   ```

#### 连接池

为了提高Nginx的访问速度，Nginx使用了连接池。连接池是一个数组，里面预先分配了很多个(根据配置文件的配置)ngx_connection_s结构。

当有客户端请求连接时，就从该数组中找到一个没有使用的ngx_connection_s，用来连接用户。

当用户close时，Nginx并没有释放掉这些数组，而是标记为可连接，然后等待下个客户端的连接。

![UTOOLS1572187983377.png](https://i.loli.net/2019/10/27/H8G7bhQc1MnuJPx.png)

配置：worker_connections

#### nginx进程间通信方式

![UTOOLS1572188853235.png](https://i.loli.net/2019/10/27/7ued6cslV4RYUyZ.png)

锁为自旋锁

使用红黑树、单链表

![UTOOLS1572189058091.png](https://i.loli.net/2019/10/27/NxJhcduCDnovrPS.png)

示例：

![UTOOLS1572239267109.png](https://i.loli.net/2019/10/28/w8rLzTkxH2hbcRg.png)

#### Slab内存管理

把一整块内存划分给红黑树使用

![UTOOLS1572238947228.png](https://i.loli.net/2019/10/28/hmM6kH8zc4TNYdX.png)

slab使用统计

![UTOOLS1572239524990.png](https://i.loli.net/2019/10/28/kTUZCB3QYlOrsoq.png)

**OpenResty开启ngx_slab_stst**

在openresty编译时使用tengine中的ngx_slab_stst模块

```shell
# 下载 tengine
$ wget http://tengine.taobao.org/download/tengine-2.3.0.tar.gz
# 解压 tengine
$ tar -xf tengine-2.3.0.tar.gz 
$ ls tengine-2.3.0/modules/ngx_slab_stat/

#进入 openresty 源码目录添加新模块
$ wget https://openresty.org/download/openresty-1.15.8.2.tar.gz
$ tar zxf openresty-1.15.8.2.tar.gz
$ cd openresty-1.15.8.2
$ ./configure --prefix=/opt/openresty --add-module=../tengine-2.3.0/modules/ngx_slab_stat/
#执行编译
$ make

# 备份原来的 nginx 可执行二进制文件
$ mv /opt/openresty/nginx/sbin/nginx /opt/openresty/nginx/sbin/nginx.old
#将编译好的 nginx 可执行二进制文件复制到原始 nginx 的 sbin 目录
$ cp -r build/nginx-1.15.8/objs/nginx /opt/openresty/nginx/sbin/
#验证是否成功安装 ngx_slab_stat
$ /opt/openresty/nginx/sbin/nginx -V
```

修改配置文件

```nginx
    ...
    lua_shared_dict dogs 10m;
    server {
        ...
        
	location /slab_stat {
		slab_stat;
	}
	location /set {
		content_by_lua_block {
			local dogs = ngx.shared.dogs
			dogs:set("Jim",8)
			ngx.say("STORED")
		}
	}
	location /get {
		content_by_lua_block {
                        local dogs = ngx.shared.dogs
                        ngx.say(dogs:get("Jim"))
                }

	}
	...
	}

```

重载配置文件后：

```shell
$ curl 127.0.0.1:80/set
STORED
$ curl 127.0.0.1:80/get
8
$ curl 127.0.0.1:80/slab_stat
* shared memory: dogs
total:       10240(KB) free:       10168(KB) size:           4(KB)
pages:       10168(KB) start:00007FAF19934000 end:00007FAF1A324000
slot:           8(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:          16(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:          32(Bytes) total:         127 used:           1 reqs:           1 fails:           0
slot:          64(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:         128(Bytes) total:          32 used:           2 reqs:           2 fails:           0
slot:         256(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:         512(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:        1024(Bytes) total:           0 used:           0 reqs:           0 fails:           0
slot:        2048(Bytes) total:           0 used:           0 reqs:           0 fails:           0

```

#### nginx 的reload处理流程

![UTOOLS1572183132944.png](https://i.loli.net/2019/10/27/G4IdOqClUjmPyhb.png)

1. 改变了nginx配置之后，HUP signal的信号需要发送给主进程。
2. 主进程首先会检测新配置的语法有效性。
3. 尝试应用新的配置
   - 打开日志文件，并且新分配一个socket来监听。
   - 如果失败，则回滚改变，还是会使用原有的配置。
   - 如果成功，则使用新的配置，新建一个线程。新建成功后发送一个关闭消息给旧的进程。要求旧线程优雅的关闭。

4. 旧的线程 受到信号后会继续服务，当所有请求的客户端被服务后，旧线程关闭。

![UTOOLS1572183712606.png](https://i.loli.net/2019/10/27/CQJEwXsu4zLnad1.png)

不停机载入新配置：

![UTOOLS1572183282101.png](https://i.loli.net/2019/10/27/MnmRvl2Suw51QzA.png)

老配置的worker进程在新进程建立完成后不再监听接口，只处理未完成的连接，在配置文件中可以添加worker_shutdown_timeout来限制老worker进程的存活时间。

#### 热升级

nginx 的热部署和其并发模型有着密不可分的关系。说白了，就是因为 master 进程的关系。当通知 ngnix 重读配置文件的时候，master 进程会进行语法错误的判断。如果存在语法错误的话，返回错误，不进行装载；如果配置文件没有语法错误，那么 ngnix 也不会将新的配置调整到所有 worker 中。而是，先不改变已经建立连接的 worker，等待 worker 将所有请求结束之后，将原先在旧的配置下启动的 worker 杀死，然后使用新的配置创建新的 worker。

**流程**：

![UTOOLS1572183937393.png](https://i.loli.net/2019/10/27/JNnLF1iGVc7EP54.png)

不停机更新nginx二进制文件

![UTOOLS1572184132160.png](https://i.loli.net/2019/10/27/sc64SEubKeXhDnA.png)

**实践**

假设有两个不同版本的nginx，一个正在运行，另一个则要替换正在运行的nginx。

```shell
#安装依赖项
$ yum install -y pcre-devel zlib-devel gcc wget vim  make
 
#低版本1.14.2
$ cd 
$ wget http://nginx.org/download/nginx-1.14.2.tar.gz
$ tar zxf nginx-1.14.2.tar.gz
$ cd nginx-1.14.2
$ ./configure --prefix=/opt/nginx --with-http_stub_status_module
$ make && make install
#然后运行该nginx
$ /opt/nginx/sbin/nginx -V
nginx version: nginx/1.14.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
configure arguments: --prefix=/opt/nginx --with-http_stub_status_module

$ /opt/nginx/sbin/nginx
$ ps -ef | grep nginx

#高版本1.16.1
$ cd 
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
$ tar zxf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
$ ./configure --prefix=/opt/nginx --with-http_stub_status_module
$ make 
#备份旧版本的nginx的执行程序
$ mv /opt/nginx/sbin/nginx /opt/nginx/sbin/nginx.old
#替换旧的Nginx的执行程序
$ cp objs/nginx /opt/nginx/sbin/nginx
$ /opt/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
configure arguments: --prefix=/opt/nginx --with-http_stub_status_module
#命令替换完毕

#平滑升级,更换worker进程
$ ps -ef | grep nginx
root      3897     1  0 07:38 ?        00:00:00 nginx: master process /opt/nginx/sbin/nginx
nobody    3898  3897  0 07:38 ?        00:00:00 nginx: worker process
root      6434    14  0 07:41 pts/1    00:00:00 grep --color=auto nginx
$ kill -USR2 `cat /opt/nginx/logs/nginx.pid`
#发送USR2信号给旧版本主进程号,使nginx的旧版本停止接收请求，用nginx新版本接替，且老进程处理完所有请求，关闭所有连接后，停止
$ ps -ef | grep nginx
root      3897     1  0 07:38 ?        00:00:00 nginx: master process /opt/nginx/sbin/nginx
nobody    3898  3897  0 07:38 ?        00:00:00 nginx: worker process
root      6440  3897  0 07:42 ?        00:00:00 nginx: master process /opt/nginx/sbin/nginx
nobody    6441  6440  0 07:42 ?        00:00:00 nginx: worker process
root      6443    14  0 07:42 pts/1    00:00:00 grep --color=auto nginx

#会发现目前系统中有2个nginx在运行，此时需要告诉 进程的nginx 优雅的关闭。
$ kill -WINCH `cat /opt/nginx/logs/nginx.pid.oldbin`
#这样nginx热部署就成功了
$ ps -ef | grep nginx
root      3897     1  0 07:38 ?        00:00:00 nginx: master process /opt/nginx/sbin/nginx
root      6440  3897  0 07:42 ?        00:00:00 nginx: master process /opt/nginx/sbin/nginx
nobody    6441  6440  0 07:42 ?        00:00:00 nginx: worker process
root      6459    14  0 08:00 pts/1    00:00:00 grep --color=auto nginx
```

保留旧的 master 进程是为了在新的版本存在问题时，可以快速回退到原版本。**如果发现问题要紧急回滚呢？**

```shell
$ mv /opt/nginx/sbin/nginx.old /opt/nginx/sbin/nginx -y
# 拉起旧版本的worker进程（-HUP 相当于 -s reload）
$ kill -HUP `cat /opt/nginx/logs/nginx.pid.oldbin`
# 让新版本的 worker 不再接受请求
$ kill -USR2 `cat /opt/nginx/logs/nginx.pid`
# 关闭新版本的 woker 进程
$ kill -WINCH `cat /opt/nginx/logs/nginx.pid`
```

如果确认无误要退出老版本的 nginx，可以执行命令：

```shell
$ kill -QUIT `cat /opt/nginx/logs/nginx.pid`
#或者退出新版本
$ kill -QUIT `cat /opt/nginx/logs/nginx.pid.oldbin`
```

#### 优雅的关闭worker进程

关闭nginx两种方式 `nginx -s stop` 立即停止nginx进程 `nginx -s quit` 优雅地关闭worker进程

![UTOOLS1572184221579.png](https://i.loli.net/2019/10/27/GHCMZNRw39dsyDQ.png)

- 主要针对http请求

所谓的优雅的关闭，是针对 worker 进程而言的，因为只有 worker 进程 才会处理请求。如果我们在处理一个连接的时候，不管连接此时对于请求是怎样一个作用，直接去关闭链接会导致用户收到错误，所以优雅地关闭就是指 Nginx 的 worker 进程 可以识别出当前连接没有正在处理请求，这个时候再把连接进行关闭。

对于某些请求 Nginx 无法做到优雅地关闭 worker 进程，比如当 Nginx 代理 websocket 协议的时候，在 websocket 后面进行通讯的 frame 桢里面，Nginx 是不解析他的桢的；Nginx 做 TCP 层或者 UDP 层反向代理的时候，也没有办法识别一个请求需要经历多少报文才算是结束；但是对于 HTTP 请求，Nginx 可以做到，所以优雅地关闭主要针对的是 HTTP 请求。

接下来我们去看一下优雅地关闭 worker 进程都有哪些流程。

开始优雅的关闭worker进程后

1. 设置定时器 worker_shutdown_timeout 设置多少秒后关闭连接
2. 关闭监听句柄，不在接收新的连接
3. 关闭空闲连接，nginx为了保证连接的快速可靠，会保持一些空闲连接。
4. 在循环中等待全部连接关闭， 两种情况，一种循环的等待nginx连接关闭了，另一种超过了worker_shutdown_timeout进程时间，连接会立即关闭。
5. 退出进程

### nginx事件

事件驱动模型是Nginx服务器保障完整功能和具有良好性能的重要机制之一。

事件驱动模型一般是由`事件收集器`，`事件发送器`，`事件处理器`三部分基本单元组成。

- 其中， `事件收集器`专门负责收集所有的事件，包括来自用户的（如鼠标单击事件、键盘输入事件等）、来自硬件的（如时钟事件等）和来自软件的（如操作系统、应用程序本身等）。
- `事件发送器`负责将收集器收集到的事件分发到目标对象中。目标对象就是事件处理器所处的位置。
- 事件处理器主要负责具体事件的响应工作，它往往要到实现阶段才完全确定。

在程序设计过程中，对事件驱动机制的实现方式有多种，这里介绍batch programming，即批次程序设计。批次的程序设计是一种比较初级的程序设计方式。使用批次程序设计的软件，其流程是由程序设计师在实际编码过程中决定的，也就是说，在程序运行的过程中，事件的发生、事件的发送和事件的处理都是预先设计好的。由此可见，事件驱动程序设计更多的关注了事件产生的随机性，使得应用程序能够具备相当的柔性，可以应付种种来自用户、硬件和系统的离散随机事件，这在很大程度上增强了用户和软件的交互性和用户操作的灵活性。

事件驱动程序可以由任何编程语言来实现，只是难易程度有别。如果一个系统是以事件驱动程序模型作为编程基础的，那么，它的架构基本上是这样的：预先设计一个事件循环所形成的程序，这个事件循环程序构成了“事件收集器”，它不断地检查目前要处理的事件信息，然后使用“事件发送器”传递给“事件处理器”。“事件处理器”一般运用虚函数机制来实现。

nginx连接对应两个事件：读事件和写事件

**网络事件**

![UTOOLS1572184576768.png](https://i.loli.net/2019/10/27/AOTGYRo1MkfPKFn.png)

![UTOOLS1572184965379.png](https://i.loli.net/2019/10/27/fPEsCXJRSVmvtK1.png)

tcp协议与非阻塞接口

![UTOOLS1572185015999.png](https://i.loli.net/2019/10/27/FqQwTC3r7X6MYbv.png)

**nginx事件循环**

![UTOOLS1572185390671.png](https://i.loli.net/2019/10/27/rUf9qgvoOtVpYPX.png)

**请求切换**

![UTOOLS1572186333057.png](https://i.loli.net/2019/10/27/kYAxXdWZVqp7nmO.png)

#### 事件驱动模型

Nginx服务器响应和处理Web请求的过程，就是基于事件驱动模型的，它也包含事件收集器、事件发送器和事件处理器等三部分基本单元。Nginx的“事件收集器”和“事件发送器”的实现没有太大的特点，重点介绍一下它的“事件处理器”。
通常，我们在编写服务器处理模型的程序时，基于事件驱动模型，“目标对象”中的“事件处理器”可以有以下几种实现办法：

- “事件发送器”每传递过来一个请求，“目标对象”就创建一个新的进程，调用“事件处理器”来处理该请求。
- “事件发送器”每传递过来一个请求，“目标对象”就创建一个新的线程，调用“事件处理器”来处理该请求。
- “事件发送器”每传递过来一个请求，“目标对象”就将其放入一个待处理事件的列表，使用非阻塞I/O方式调用“事件处理器”来处理该请求。

以上的三种处理方式，各有特点，第一种方式，由于创建新的进程的开销比较大，会导致服务器性能比较差，但其实现相对来说比较简单。

第二种方式，由于要涉及到线程的同步，故可能会面临死锁、同步等一系列问题，编码比较复杂。

第三种方式，在编写程序代码时，逻辑比前面两种都复杂。大多数网络服务器采用了第三种方式，逐渐形成了所谓的“事件驱动处理库”。

事件驱动处理库又被称为多路IO复用方法，最常见的包括以下三种：select模型，poll模型和epoll模型。Nginx服务器还支持rtsig模型、kqueue模型、dev/poll模型和eventport模型等。通过Nginx配置可以使得Nginx服务器支持这几种事件驱动处理模型。这里详细介绍以下它们。

1. select库

   select库，是各个版本的Linux和Windows平台都支持的基本事件驱动模型库，并且在接口的定义上也基本相同，只是部分参数的含义略有差异。使用select库的步骤一般是：

   首先，创建所关注事件的描述符集合。对于一个描述符，可以关注其上面的（Read)事件、写（Write)事件以及异常发送（Exception）事件，所以要创建三类事件描述符集合，分别用来收集读事件的描述符、写事件的描述符和异常事件的描述符。

   其次，调用底层提供的select()函数，等待事件发生。这里需要注意的一点是，select的阻塞与是否设置非阻塞I/O是没有关系的。

   然后，轮询所有事件描述符集合中的每一个事件描述符，检查是否有相应的事件发生，如果有，就进行处理。
   Nginx服务器在编译过程中如果没有为其指定其他高性能事件驱动模型库，它将自动编译该库。我们可以使用--with-select_module和--without-select_module两个参数强制Nginx是否编译该库。

2. poll库

   poll库，作为Linux平台上的基本事件驱动模型，实在Linux2.1.23中引入的。Windows平台不支持poll库。

   poll与select的基本工作方式是相同的，都是现创建一个关注事件的描述符集合，再去等待这些事件发生，然后在轮询描述符集合，检查有没有事件发生，如果有，就进行处理。

   poll库与select库的主要区别在于，select库需要为读事件、写事件和异常事件分别创建一个描述符集合，因此在最后轮询的时候，需要分别轮询这三个集合。而poll库只需要创建一个集合，在每个描述符对应的结构上分别设置读事件、写事件或者异常事件，最后轮询的时候，可以同时检查这三种事件是否发生。可以说，poll库是select库的优化实现。

   Nginx服务器在编译过程中如果没有为其制定其他高性能事件驱动模型库，它将自动编译该库。我们可以使用--with-poll_module和--without-poll_module两个参数强制Nginx是否编译该库。

3. **epoll**库

   epoll库是Nginx服务器支持的高性能事件驱动库之一，它是公认的非常优秀的事件驱动模型，和poll库及select库有很大的不同。epoll属于poll库的一个变种，是在Linux 2.5.44中引入的，在Linux 2.6以上的版本都可以使用它。poll库和select库在实际工作中，最大的区别在于效率。

   ![UTOOLS1572186025944.png](https://i.loli.net/2019/10/27/eF5XjKRVMxYZfDE.png)

   从前面的介绍我们知道，它们的处理方式都是创建一个待处理事件列表，然后把这个列表发给内核，返回的时候，再去轮询检查这个列表，以判断事件是否发生。这样在描述符比较多的应用中，效率就显得比较低下了。一种比较好的做法是，把描述符列表的管理交给内核负责，一旦有某种事件发生，内核把发生事件的描述符列表通知给进程，这样就避免了轮询整个描述符列表。epoll库就是这样一种模型。

   首先，epoll库通过相关调用通知内核创建一个由N个描述符的事件列表。然后，给这些描述符设置所关注的事件，并把它添加到内核的事件列表中去，在具体的编码过程中也可以通过相关调用对事件列表中的描述符进行修改和删除。

   完成设置之后，epoll库就开始等待内核通知事件发生了。某一事件发生后，内核将发生事件的描述符列表上报给epoll库。得到事件列表的epoll库，就可以进行事件处理了。

   epoll库在Linux平台上是最高效的。它支持一个进程打开大数目的事件描述符，上限是系统可以打开文件的最大数目。同时，epoll库的IO效率不随描述符数目增加而线性下降，因为它只会对内核上报的“活跃”的描述符进行操作。

4. rtsig模型

   rtsig是Real-Time Signal的缩写，是实时信号的意思。从严格意义上说，rtsig模型并不是常用的事件驱动模型，但Nginx服务器使用了使用实时信号对事件进行响应的支持，官方文档中将rtsig模型与其他的事件驱动模型并列。

   使用rtsig模型时，工作进程会通过系统内核建立一个rtsig队列用于存放标记事件发生（在Nginx服务器应用中特指客户端请求发生）的信号。每个事件发生时，系统内核就会产生一个信号存放到rtsig队列中等待工作进程的处理。

   需要指出的是，rtsig队列有长度限制，超过该长度后就会发生溢出。默认情况下，Linux系统事件信号队列的最大长度设置为1024，也就是同时最多可以存放1024个发生事件的信号。在Linux 2.6.6-mm2之前的版本中，系统各个进程的事件信号队列是由内核统一管理的，用户可以通过修改内核参数/proc/sys/kernel/rtsig-max/来自定义该长度设置。在Linux 2.6.6-mm2之后的版本中，该内核参数被取消，系统各个进程分别拥有各自的事件信号队列，这个队列的大小由Linux系统的RLIMT_SIGPENGIND参数定义，在执行setrlimit()系统调用时确定该大小。Nginx提供了worker_rlimit_sigpending参数用于调节这种情况下的事件信号队列长度。

   当rtsig队列发生溢出时，Nginx将暂时停止使用rtsig模型，而调用poll库处理未处理的事件，直到rgsit信号队列全部清空，然后再次启动rtsig模型，以防止新的溢出发生。

   Nginx在配置文件中提供了相关参数对rtsig模型的使用配置。编译Nginx服务器时，使用--with-rtsig_module配置选项来启用rtsig模型的编译。

5. 其他事件驱动模型

   除了以上四种主要的事件驱动模型，Nginx服务器针对特定的Linux平台提供了响应的事件驱动模型支持。目前实现的主要有kqueue模型、/dev/poll模型和eventport模型等。

   - kqueue模型，是用于支持BSD系列平台的高效事件驱动模型，主要用在FreeBSD 4.1及以上版本、OpenBSD 2.9及以上版本、NetBSD 2.0及以上版本以及Mac OS X平台上。该模型也是poll库的一个变种，其和epoll库的处理方式没有本质上的区别，都是通过避免轮询操作提供效率。该模型同时支持条件触发（level-triggered,也叫水平触发，只要满足条件就触发一个事件）和边缘触发（edge-triggered，每当状态变化时，触发一个事件）。如果大家在这些平台下使用Nginx服务器，建议选在该模型用于请求处理，以提高Nginx服务器的处理性能。
   - /dev/poll模型，适用于支持Unix衍生平台的高效事件驱动模型，其主要在Solaris711/99及以上版本、HP/UX 11.22及以上版本、IRIX 6.5.15及以上版本和Tru64 UNIX 5.1A及以上版本的平台中使用。该模型是Sun公司在开发Solaris系列平台时提出的用于完成事件驱动机制的方案，它使用了虚拟的/dev/poll设备，开发人员可以将要监视的文件描述符加入这个设备，然后通过ioctl()调用来获取事件通知。在以上提到的平台中，建议使用该模型处理请求。

   - eventport模型，适用于支持Solaris 10及以上版本平台的高效事件驱动模型。该模型也是Sun公司在开发Solaris系列平台时提出的用于完成事件驱动机制的方案，它可以有效防止内核崩溃情况的发生。Nginx服务器为此提供了支持。

   以上就是Nginx服务器支持的事件驱动库。可以看到，Nginx服务器针对不同的Linux或Unix衍生平台提供了多种事件驱动模型的处理，尽量发挥系统平台本身的优势，最大程度地提高处理客户端请求事件的能力。在实际工作中，我们需要根据具体情况和应用情景选择使用不同的事件驱动模型，以保证Nginx服务器的高效运行。

Nginx 的连接处理机制在不同的操作系统会采用不同的 I/O 模型，要根据不同的系统选择不同的事件处理模型，可供选择的事件处理模型有：kqueue 、rtsig 、epoll 、/dev/poll 、select 、poll ，其中 select 和 epoll 都是标准的工作模型，kqueue 和 epoll 是高效的工作模型，不同的是 epoll 用在 Linux 平台上，而 kqueue 用在 BSD 系统中。

1. 在 Linux 下，Nginx 使用 epoll 的 I/O 多路复用模型

2. 在 Freebsd 下，Nginx 使用 kqueue 的 I/O 多路复用模型

3. 在 Solaris 下，Nginx 使用 /dev/poll 方式的 I/O 多路复用模型

4. 在 Windows 下，Nginx 使用 icop 的 I/O 多路复用模型

```nginx
......
events {
use epoll;
}
......
```

#### I/O模型

参考：<https://pingxin0521.coding.me/2019/12/07/周记-3-0/>

##### nginx是异步非阻塞

1. 异步

   异步通常是指调用之后，直接返回，如果有结果后通过消息通知，或者调用注册的回调函数进行处理，其核心在于有结果后通过其他执行流程进行通知和处理，不影响现有的执行流程执行。

   而nginx通过超时的epoll方式进行监听连接和或者监听接收多个socket的数据，这里虽然存在超时等待，当还是说它是异步，实际上更确切说是epoll内部的异步机制，即epoll是对不同的socket通过注册感兴趣的事件，内部不是轮询等待这些socket，而是通过在事件发生后收到通知方式对事件进行处理，这里进行超时等待实际上是为了留有时间让事件发生，回收通知。详细过程可参考[【Linux学习】epoll详解](https://blog.csdn.net/xiajun07061225/article/details/9250579)

   总结：**nginx的异步是指epoll的异步机制，即内部探测socket事件发生是通过消息通知方式**

2. 非阻塞

   非阻塞通常是指调用指定调用后，若不满足当前需求直接返回，若满足需求则处理，处理之后返回，因此这种方式实际是需要轮询访问直至成功。

   nginx是采用一个进程单线程方式进行多个socket数据处理和建立连接，因此在每个socket进行接收数据或者建立链接的socket时必须采用非阻塞的模式，这样才不会导致一个socket阻塞后，导致其他的socket饿死的情况。

   总结：**nginx的非阻塞是指nginx对于建立链接的socket和每个建立的连接是采用非阻塞的模式进行数据处理的**

### nginx模块

nginx的强大在于它丰富的模块

![UTOOLS1572187393033.png](https://i.loli.net/2019/10/27/CvH79tGzcTKUYyu.png)

核心模块：core module

HTTP modules（http模块）：

- Standard HTTP modules标准的http模块
- Optional HTTP modules可选的http模块
- Mail modules邮件模块
- Stream modules：流模块，传输层代理，4层tcp和udp代理，传输层代理
- 3rd party modules第三方模块

**模块分类**

![UTOOLS1572187601410.png](https://i.loli.net/2019/10/27/NYvPfplLUZ3bo2y.png)

#### Nginx容器

![UTOOLS1572241638781.png](https://i.loli.net/2019/10/28/nkAw9QoXOfNF5Y8.png)

- 数组：ngx_arr_t多块连续内存块

- 链表：ngx_list_t

- 队列：ngx_queue_t

- 哈希表：静态不变的内容，

  ![UTOOLS1572241826040.png](https://i.loli.net/2019/10/28/863h4RCdye9Nfir.png)

  所有使用hash表的模块

  ![image-20191028135133388](/home/hyp/.config/Typora/typora-user-images/image-20191028135133388.png)

- 红黑树：ngx_rbtree_t，二叉查找树，

  ![UTOOLS1572242094163.png](https://i.loli.net/2019/10/28/vzJU2dDFa4MEtoX.png)

  使用到rbtree的模块

  ![UTOOLS1572242250811.png](https://i.loli.net/2019/10/28/y41ViBnOYuvpWK5.png)

- 基数树

#### nginx动态模块

NGINX 1.9.11开始增加加载动态模块支持，从此不再需要替换nginx文件即可增加第三方扩展。目前官方只有几个模块支持动态加载，第三方模块需要升级支持才可编译成模块。

```
# ./configure --help | grep dynamic
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-stream=dynamic              enable dynamic TCP proxy module
  --add-dynamic-module=PATH          enable dynamic external module
```

如上可看出官方支持5个动态模块编译，需要增加第三方模块，使用参数--add-dynamic-module=即可。

![UTOOLS1572242314131.png](https://i.loli.net/2019/10/28/B4REaXLhmsb3lq7.png)

```shell
$ wget https://openresty.org/download/openresty-1.15.8.2.tar.gz
$ tar zxf openresty-1.15.8.2.tar.gz
$ cd openresty-1.15.8.2
$  ./configure --help | grep dynamic
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --add-dynamic-module=PATH          enable dynamic external module

```

这里使用`--with-http_image_filter_module`进行测试

```
$ ./configure --prefix=/root/nginx --with-http_image_filter_module=dynamic
$ make && make install
$ cd /root/nginx
$ nginx/sbin/nginx -V
nginx version: openresty/1.15.8.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/root/nginx/nginx --with-cc-opt=-O2 --add-module=../ngx_devel_kit-0.3.1rc1 --add-module=../echo-nginx-module-0.61 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.31 --add-module=../ngx_lua-0.10.15 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../rds-json-nginx-module-0.15 --add-module=../rds-csv-nginx-module-0.09 --add-module=../ngx_stream_lua-0.0.7 --with-ld-opt=-Wl,-rpath,/root/nginx/luajit/lib --with-http_image_filter_module=dynamic --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_ssl_module

$ ls nginx/modules/
```

**NGINX动态模块语法**

```
load_module 
```

Default: —

配置段: main

说明：版本必须>=1.9.11

实例：`load_module modules/ngx_mail_module.so;`

**使用**

在`nginx\html`下增加图片

```
$ vim nginx\conf\nginx.conf

load_module modules/ngx_http_image_filter_module.so;

...
location / {
            root   html;
            index  index.html index.htm;
            image_filter resize 15 10;
            image_filter_buffer 100M;
        }
...
```

### Nginx 的高可用方案

Nginx 作为反向代理服务器，所有的流量都会经过 Nginx，所以 Nginx 本身的可靠性是我们首先要考虑的问题

**keepalived**

Keepalived 是 Linux 下一个轻量级别的高可用解决方案，Keepalived 软件起初是专为 LVS 负载均衡软件设计的，

用来管理并监控 LVS 集群系统中各个服务节点的状态，后来又加入了可以实现高可用的 VRRP 功能。因此，Keepalived 除了能够管理 LVS 软件外，

还可以作为其他服务（例如：Nginx、Haproxy、MySQL 等）的高可用解决方案软件Keepalived 软件主要是通过 VRRP 协议实现高可用功能的。

VRRP 是 VirtualRouter RedundancyProtocol(虚拟路由器冗余协议）的缩写，VRRP 出现的目的就是为了解决静态路由单点故障问题的，

它能够保证当个别节点宕机时，整个网络可以不间断地运行;(简单来说，vrrp 就是把两台或多态路由器设备虚拟成一个设备，实现主备高可用)

所以，Keepalived 一方面具有配置管理 LVS 的功能，同时还具有对 LVS 下面节点进行健康检查的功能，另一方面也可实现系统网络服务的高可用功能

LVS 是 Linux Virtual Server 的缩写，也就是 Linux 虚拟服务器，在 linux2.4 内核以后，已经完全内置了 LVS 的各个功能模块。

它是工作在四层的负载均衡，类似于 Haproxy, 主要用于实现对服务器集群的负载均衡。

关于四层负载，我们知道 osi 网络层次模型的 7 层模模型（应用层、表示层、会话层、传输层、网络层、数据链路层、物理层）；

四层负载就是基于传输层，也就是ip+端口的负载；而七层负载就是需要基于 URL 等应用层的信息来做负载，同时还有二层负载（基于 MAC）、三层负载（IP）；

常见的四层负载有：LVS、F5； 七层负载有:Nginx、HAproxy; 在软件层面，Nginx/LVS/HAProxy 是使用得比较广泛的三种负载均衡软件

对于中小型的 Web 应用，可以使用 Nginx、大型网站或者重要的服务并且服务比较多的时候，可以考虑使用 LVS 

**轻量级的高可用解决方案**

LVS 四层负载均衡软件（Linux virtual server）监控 lvs 集群系统中的各个服务节点的状态

VRRP 协议（虚拟路由冗余协议）linux2.4 以后，是内置在 linux 内核中的

lvs(四层) -> HAproxy 七层

lvs(四层) -> Nginx(七层)

![lE1vmF.png](https://s2.ax1x.com/2019/12/26/lE1vmF.png)





### 参考

1. [Nginx（三）--Nginx 的高可用](https://www.cnblogs.com/flgb/p/10909295.html)