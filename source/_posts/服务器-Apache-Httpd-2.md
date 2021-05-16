---
title: Apache Httpd 优化
date: 2019-05-31 11:18:59
tags:
 - 服务器
 - Apache
 - Httpd
categories:
 - 服务器
 - Apache Httpd
---

Apache即现阶段比较流行的Web服务，是一个多模块化的Web服务，使用简单，速度快，稳定性好，可以做负载均衡及代理服务器来使用。

Apache有三种工作模式分别是   ：Prefork  MPM、Worker  MPM、Event     MPM

<!--more-->

1. prefork 多进程模式
   一个主进程，负责生成多个子进程，也称工作进程，进程之间独立，每个进程之间只能有一个线程，优点是稳定，缺点是内存占用大，每个进程响应一个用户请求。

2. worker 多线程模式
   一个主进程生多个子进程，每个子进程生成多个线程，默认25个，每个线程响应一个用户请求，优点：线程之间内存共享，内存利用率高，缺点：安全性稳定性较差，一个进程崩掉，整个进程内的线程都一起挂。
3. event 事件驱动模式
   一个进程处理多个请求，前面两种模型在处理高并发请求时，很快会耗光服务器的可用进程，event 把进程进行分工，采用专用的进程来监听请求保持连接，因为保持连接只需要极少的资源。

可以通过`apachectl -V`查看当前Apache的工作模式。

```bash
$ apachectl -V
Server version: Apache/2.4.29 (Ubuntu)
Server built:   2020-03-13T12:26:16
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.6.3, APR-UTIL 1.6.1
Compiled using: APR 1.6.3, APR-UTIL 1.6.1
Architecture:   64-bit
Server MPM:     prefork
  threaded:     no
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_SYSVSEM_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
 -D HTTPD_ROOT="/etc/apache2"
 -D SUEXEC_BIN="/usr/lib/apache2/suexec"
 -D DEFAULT_PIDLOG="/var/run/apache2.pid"
 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
 -D DEFAULT_ERRORLOG="logs/error_log"
 -D AP_TYPES_CONFIG_FILE="mime.types"
 -D SERVER_CONFIG_FILE="apache2.conf"

```

![](https://i.loli.net/2019/06/03/5cf5333c252f161100.png)

在编译安装httpd时可以使用--with-mpm=mpm进行指定，httpd2.4版本以后使用--enable-mpms-shared=all全部安装，在配置文件中修改配置即可。

**Prefork MPM**

![1.jpg](https://i.loli.net/2019/06/03/5cf5311a9841a14375.jpg)

默认的工作模式是Prefork MPM，这种模式采用的是预派生子进程方式，用单独的子进程来处理请求，子进程间互相独立，互不影响，大大的提高了稳定性，但每个进程都会占用内存，所以消耗系统资源过高；

Prefork MPM 工作原理：控制进程Master首先会生成“StartServers”个进程，“StartServers”可以在Apache主配置文件里配置，然后为了满足“MinSpareServers”设置的最小空闲进程个数，会建立一个空闲进程，等待一秒钟，继续创建两个空闲进程，再等待一秒钟，继续创建四个空闲进程，以此类推，会不断的递归增长创建进程，最大同时创建32个空闲进程，直到满足“MinSpareServers”设置的空闲进程个数为止。Apache的预派生模式不必在请求到来的时候创建进程，这样会减小系统开销以增加性能，不过Prefork MPM是基于多进程的模式工作的，每个进程都会占用内存，这样资源消耗也较高。

**Worker MPM**

![2.jpg](https://i.loli.net/2019/06/03/5cf5311aaeec959271.jpg)

Worker MPM是Apche 2.0版本中全新的支持多进程多线程混合模型的MPM，由于使用线程来处理HTTP请求，所以效率非常高，而对系统的开销也相对较低，Worker MPM也是基于多进程的，但是每个进程会生成多个线程，由线程来处理请求，这样可以保证多线程可以获得进程的稳定性；

Worker MPM工作原理： 控制进程Master在最初会建立“StartServers”个进程，然后每个进程会创建“ThreadPerChild”个线程，多线程共享该进程内的资源，同时每个线程独立的处理HTTP请求，为了不在请求到来的时候创建线程，Worker MPM也可以设置最大最小空闲线程，Worker MPM模式下同时处理的请求=ThreadPerChild*进程数，也就是MaxClients，如果服务负载较高，当前进程数不满足需求，Master控制进程会fork新的进程，最大进程数不能超过ServerLimit数，如果需要，可以调整这些对应的参数，比如，如果要调整StartServers的数量，则也要调整 ServerLimit的值

**Event MPM**

![3.jpg](https://i.loli.net/2019/06/03/5cf5311b11d9931984.jpg)

这个是 Apache中最新的模式，在现在版本里的已经是稳定可用的模式。它和 worker模式很像，最大的区别在于，它解决了 keep-alive 场景下 ，长期被占用的线程的资源浪费问题（某些线程因为被keep-alive，挂在那里等待，中间几乎没有请求过来，一直等到超时）。
event MPM中，会有一个专门的线程来管理这些 keep-alive 类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放。这样，一个线程就能处理几个请求了，实现了异步非阻塞。
event MPM在遇到某些不兼容的模块时，会失效，将会回退到worker模式，一个工作线程处理一个请求。官方自带的模块，全部是支持event MPM的。

**总结：**

- Prefork MPM： 使用多个进程，每个进程只有一个线程，每个进程再某个确定的时间只能维持一个连接，有点是稳定，缺点是内存消耗过高。

- Worker MPM： 使用多个进程，每个进程有多个线程，每个线程在某个确定的时间只能维持一个连接，内存占用比较小，是个大并发、高流量的场景，缺点是一个线程崩溃，整个进程就会连同其任何线程一起挂掉。

- Event  MPM:   使用多进程多线程+epoll的模式，

注意一点，event MPM需要Linux系统（Linux 2.6+）对Epoll的支持，才能启用。
还有，需要补充的是HTTPS的连接（SSL），它的运行模式仍然是类似worker的方式，线程会被一直占用，知道连接关闭。部分比较老的资料里，说event MPM不支持SSL，那个说法是几年前的说法，现在已经支持了。

#### 修改MPM模块配置

1. 启用MPM模块配置文件

   在Apace安装目录/conf/extra目录中有一个名为httpd-mpm.conf的配置文件。该文件主要用于进行MPM模块的相关配置。不过，在默认情况下，Apache的MPM模块配置文件并没有启用。因此，我们需要在httpd.conf文件中启用该配置文件，如下所示：

   ```
   # Server-pool management (MPM specific)
   # Include conf/extra/httpd-mpm.conf (去掉该行前面的注释符号"#")
   ```

2. 修改MPM模块配置文件中的相关配置

   在启动MPM模块配置文件后，我们就可以使用文本编辑器打开该配置文件，我们可以看到，在该配置文件中有许多`<IfModule>`配置节点，如下图所示： 

   ![1.png](https://i.loli.net/2019/06/03/5cf534f62e10496102.png)

   

#### prefork的工作原理及配置

如果不用“--with-mpm”显式指定某种MPM,prefork就是Unix平台上缺省的MPM。它所采用的预派生子进程方式也是Apache 1.3中采用的模式。prefork本身并没有使用到线程,2.0版使用它是为了与1.3版保持兼容性;另一方面,prefork用单独的子进程来处理不同的请求,进程之间是彼此独立的,这也使其成为最稳定的MPM之一。

若使用prefork,在make编译和make install安装后,使用“httpd -l”来确定当前使用的MPM,应该会看到prefork.c(如果看到worker.c说明使用的是worker MPM,依此类推)。

再查看缺省生成的httpd.conf配置文件,里面包含如下配置段:

```
<IfModule prefork.c>
StartServers 5
MinSpareServers 5
MaxSpareServers 10
MaxClients 150
MaxRequestsPerChild 0
</IfModule>
# StartServers:　　默认初始化预派生的进程数
# MinSpareServers:　　最小空闲子进程数
# MaxSpareServers:　　最大空闲子进程数
# ServerLimit: 最大派生的进程数(MaxRequestWorkers的上限值，最大不能超过这个值可以等于)
# MaxRequestWorkers:　　最大程请求数（最大并发数）
# MaxConnectionsPerChild:每个进程可处理的请求数，达到最大后该进程将被杀死，0代表没有限制
```

prefork的工作原理是,控制进程在最初建立“StartServers”个子进程后,为了满足MinSpareServers设置的需要创建一个进程,等待一秒钟,继续创建两个,再等待一秒钟,继续创建四个......如此按指数级增加创建的进程数,最多达到每秒32个,直到满足MinSpareServers设置的值为止。这就是预派生(prefork)的由来。这种模式可以不必在请求到来时再产生新的进程,从而减小了系统开销以增加性能。

MaxSpareServers设置了最大的空闲进程数,如果空闲进程数大于这个值,Apache会自动kill掉一些多余进程。这个值不要设得过大,但如果设的值比MinSpareServers小,Apache会自动把其调整为MinSpareServers+ 1。如果站点负载较大,可考虑同时加大MinSpareServers和MaxSpareServers。MaxRequestsPerChild设置的是每个子进程可处理的请求数。每个子进程在处理了

“MaxRequestsPerChild”个请求后将自动销毁。0意味着无限,即子进程永不销毁。虽然缺省设为0可以使每个子进程处理更多的请求,但如果设成非零值也有两点重要的好处:

- 可防止意外的内存泄漏;
-  在服务器负载下降的时侯会自动减少子进程数。

因此,可根据服务器的负载来调整这个值。我认为10000左右比较合适。

MaxClients是这些指令中最为重要的一个,设定的是Apache可以同时处理的请求,是对Apache性能影响最大的参数。其缺省值150是远远不够的,如果请求总数已达到这个值(可通过ps -ef|grep http|wc -l来确认),那么后面的请求就要排队,直到某个已处理请求完毕。

这就是系统资源还剩下很多而HTTP访问却很慢的主要原因。系统管理员可以根据硬件配置和负载情况来动态调整这个值。虽然理论上这个值越大,可以处理的请求就越多,但Apache默认的限制不能大于256。如果把这个值设为大于256,那么 Apache将无法起动。事实上,256对于负载稍重的站点也是不够的。在Apache 1.3中,这是个硬限制。

如果要加大这个值,必须在“configure”前手工修改的源代码树下的src/include/httpd.h中查找 256,就会发现“#define HARD_SERVER_LIMIT 256”这行。把256改为要增大的值(如4000),然后重新编译Apache即可。在Apache 2.0中新加入了ServerLimit指令,使得无须重编译Apache就可以加大MaxClients。下面是我的prefork配置段:

```
<IfModule prefork.c>
StartServers 10
MinSpareServers 10
MaxSpareServers 15
ServerLimit 2000
MaxClients 1000
MaxRequestsPerChild 10000
</IfModule>
```

上述配置中,ServerLimit的最大值是20000,对于大多数站点已经足够。如果一定要再加大这个数值,对位于源代码树下server/mpm/prefork/prefork.c中以下两行做相应修改即可:

```
#define DEFAULT_SERVER_LIMIT 256
#define MAX_SERVER_LIMIT 20000
```

#### worker的工作原理及配置

相对于prefork,worker是2.0 版中全新的支持多线程和多进程混合模型的MPM。由于使用线程来处理,所以可以处理相对海量的请求,而系统资源的开销要小于基于进程的服务器。但是, worker也使用了多进程,每个进程又生成多个线程,以获得基于进程服务器的稳定性。这种MPM的工作方式将是Apache 2.0的发展趋势。

在configure -with-mpm=worker后,进行make编译、make install安装。在缺省生成的httpd.conf中有以下配置段:

```
<IfModule worker.c>
StartServers 2
MaxClients 150
MinSpareThreads 25
MaxSpareThreads 75
ThreadsPerChild 25
MaxRequestsPerChild 0
</IfModule>
# StartServers:　　默认初始化开始的子进程数
# MinSpareThreads:　　最小空闲线程数
# MaxSpareThreads:　　最大空闲线程数
# ThreadsPerChild:　　每个子进程产生的线程数
# MaxRequestWorkers:　　最大请求子进程数
# MaxConnectionsPerChild:　　每个子进程处理多少个请求后被杀死
```

worker的工作原理是,由主控制进程生成“StartServers”个子进程,每个子进程中包含固定的ThreadsPerChild线程数,各个线程独立地处理请求。同样,为了不在请求到来时再生成线程,MinSpareThreads和MaxSpareThreads设置了最少和最多的空闲线程数;而MaxClients设置了所有子进程中的线程总数。如果现有子进程中的线程总数不能满足负载,控制进程将派生新的子进程。

MinSpareThreads和MaxSpareThreads的最大缺省值分别是75和250。这两个参数对Apache的性能影响并不大,可以按照实际情况相应调节。

ThreadsPerChild是worker MPM中与性能相关最密切的指令。ThreadsPerChild的最大缺省值是64,如果负载较大,64也是不够的。这时要显式使用 ThreadLimit指令,它的最大缺省值是20000。上述两个值位于源码树server/mpm/worker/worker.c中的以下两行:

```
#define DEFAULT_THREAD_LIMIT 64
#define MAX_THREAD_LIMIT 20000
```

这两行对应着ThreadsPerChild和ThreadLimit的限制数。最好在configure之前就把64改成所希望的值。注意,不要把这两个值设得太高,超过系统的处理能力,从而因Apache不起动使系统很不稳定。

Worker模式下所能同时处理的请求总数是由子进程总数乘以ThreadsPerChild值决定的,应该大于等于MaxClients。如果负载很大,现有的子进程数不能满足时,控制进程会派生新的子进程。默认最大的子进程总数是16,加大时也需要显式声明ServerLimit(最大值是 20000)。这两个值位于源码树server/mpm/worker/worker.c中的以下两行:

```
#define DEFAULT_SERVER_LIMIT 16
#define MAX_SERVER_LIMIT 20000
```

需要注意的是,如果显式声明了ServerLimit,那么它乘以ThreadsPerChild的值必须大于等于MaxClients,而且 MaxClients必须是ThreadsPerChild的整数倍,否则Apache将会自动调节到一个相应值(可能是个非期望值)。下面是我的 worker配置段:

```
<IfModule worker.c>
StartServers 3
MaxClients 2000
ServerLimit 25
MinSpareThreads 50
MaxSpareThreads 200
ThreadLimit 200ThreadsPerChild 100
MaxRequestsPerChild 0
</IfModule>
```

通过上面的叙述,可以了解到Apache 2.0中prefork和worker这两个重要MPM的工作原理,并可根据实际情况来配置Apache相关的核心参数,以获得最大的性能和稳定性

#### event MPM模块：

 event模式是非常新的。事实上，它只在Apache2.4版本中被作为稳定版发布。event模式和Worker模式工作原理相同，它也是使用进程和线程。它们最大的区别在与event模式会为每个请求创建一个线程，而不是为一个http连接创建一个线程。
有一种情况那就是当你喜欢使用线程但是有一个应用程序，这个应用程序使用了较长的keepalive超时时间时这种模式很适用。在Worker MPM中，线程是和连接绑定的，并且无论http请求是否被处理都保持被占用状态。
在event MPM中，如果处理连接的线程只是用来处理当前请求并且会在请求处理完成后立即释放，不管被父进程处理的http连接的情况。同时，当线程在请求被处理完成立即释放后可以被用来处理其他请求。这意味着需要更少的线程！
event就是worker模式的变种，他解决的keep-alive长连接的时候占用线程资源被浪费的问题，在event模式中会有一些专门的线程用来管理这些keep-alive类型的线程，当有真实请求过来的时候，将请求传递给服务器的线程，执行完毕后，又允许它释放。这增强了在高并发场景下的请求处理。

缺点：event模式不能很好的支持https的访问（HTTP认证相关的问题）。

![4.png](https://i.loli.net/2019/06/03/5cf533d125d6788213.png)

```
<IfModule mpm_event_module>
StartServers 3
MinSpareThreads 75
MaxSpareThreads 250
ThreadsPerChild 25
MaxRequestWorkers 400
MaxConnectionsPerChild 0
</IfModule> 
# StartServers:　　默认初始化开始的子进程数
# MinSpareThreads:　　最小空闲线程数
# MaxSpareThreads:　　最大空闲线程数
# ThreadsPerChild:　　每个子进程产生的线程数
# MaxRequestWorkers:　　最大请求子进程数
# MaxConnectionsPerChild:　　每个子进程处理多少个请求后被杀死
```

#### 限制Apache并发连接数

我们知道当网站以http方式提供软件下载时,若是每个用户都开启多个线程并没有带宽的限制,将很快达到http的最大连接数或者造成网络阻塞,使得网站的许多正常服务都无法运行.下面我们添加mod_limitipconn模块,来控制http的并发连接数.

```
wget http://dominia.org/djao/limit/mod_limitipconn-0.24.tar.bz2
tar zxvf mod_limitipconn-0.24.tar.bz2
cd mod_limitipconn-0.24
sudo /opt/servers/httpd/bin/apxs -c -i -a mod_limitipconn.c
# 编译好后会自动把mod_rewrite.so拷贝到/usr/local/apache/modules下,并修改你
的httpd.conf文件.
sudo vim /opt/servers/httpd/conf/httpd.conf
# 在最后一行加入
ExtendedStatus On
<IfModule limitipconn_module>
MaxConnPerIP 2
</IfModule>
#所限制的目录所在,此处表示主机的根目录MaxConnPerIP 2
#所限制的每个IP并发连接数为2个
# 保存退出.
/usr/local/apache/bin/apachectl start
```

#### 防止文件被盗链

我们刚才已经限制了IP并发数,但如果对方把链接盗链到别的页面,我们刚才做的就毫无意义了,因为他完全可以通过迅雷或快车进行下载.所以就这种情况,我们要引用mod_rewrite.so模块.这样,当他盗链了文件,通过mod_rewrite.so模块把页面引到了一个事先我们制定好的错误页面里,这样就防止了盗链

```
/opt/servers/httpd/bin/apxs -c -i -a /opt/servers/httpd/modules/mappers/mod_rewrite.la
# 编译好后会自动把mod_rewrite.so拷贝到/opt/servers/httpd/modules下,并修改你
的httpd.conf文件.
# vi /usr/local/apache/conf/httpd.conf
RewriteEngine onRewriteCond %{HTTP_REFERER} !
^http://www.squall.cn/.*$ [NC]RewriteCond %{HTTP_REFERER} !
^http://www.squall.cn$ [NC]RewriteCond %{HTTP_REFERER} !
^http://squall.cn/.*$ [NC]RewriteCond %{HTTP_REFERER} !
^http://squall.cn$ [NC]RewriteRule .*\\.
(jpg gif png
bmp tar gz rar zip
http://www.squall.cn/error.htm [R,NC]
exe)
```

#### 模块配置：

```
	(1)启用压缩
        LoadModule deflate_module modules/mod_deflate.so
    (2)启用重写
        LoadModule rewrite_module modules/mod_rewrite.so  
    (3)启用默认扩展，支持在这里进行修改httpd主要配置
        Include conf/extra/httpd-default.conf  
    (4)提供文件描述符缓存支持，从而提高Apache性能
        LoadModule file_cache_module modules/mod_file_cache.so
    (5)启用基于URI键的内容动态缓冲(内存或磁盘)  
        LoadModule cache_module modules/mod_cache.so  
    (6)启用基于磁盘的缓冲管理器
        LoadModule cache_disk_module modules/mod_cache_disk.so  
    (7)基于内存的缓冲管理器
        LoadModule socache_memcache_module modules/mod_socache_memcache.so 
    (8)屏蔽所有不必要的模块
        #LoadModule authn_file_module modules/mod_authn_file.so  
        #LoadModule authn_dbm_module modules/mod_authn_dbm.so  
        #LoadModule authn_anon_module modules/mod_authn_anon.so  
        #LoadModule authn_dbd_module modules/mod_authn_dbd.so  
        #LoadModule authn_socache_module modules/mod_authn_socache.so  
        #LoadModule authn_core_module modules/mod_authn_core.so  
        #LoadModule authz_host_module modules/mod_authz_host.so  
        #LoadModule authz_groupfile_module modules/mod_authz_groupfile.so  
        #LoadModule authz_user_module modules/mod_authz_user.so  
        #LoadModule authz_dbm_module modules/mod_authz_dbm.so  
        #LoadModule authz_owner_module modules/mod_authz_owner.so  
        #LoadModule authz_dbd_module modules/mod_authz_dbd.so  
        LoadModule authz_core_module modules/mod_authz_core.so  
        LoadModule access_compat_module modules/mod_access_compat.so  
        #LoadModule auth_basic_module modules/mod_auth_basic.so  
        #LoadModule auth_form_module modules/mod_auth_form.so  
        #LoadModule auth_digest_module modules/mod_auth_digest.so 
    （9）已经过时屏蔽
        #LoadModule autoindex_module modules/mod_autoindex.so
    （10）用于定义缺省文档index.php、index.jsp等
        LoadModule dir_module modules/mod_dir.so
    （11）用于定义记录文件格式
        LoadModule log_config_module modules/mod_log_config.so
    （12）定义文件类型的关联
        LoadModule mime_module modules/mod_mime.so
    （13）减少10%左右的重复请求
        LoadModule expires_module modules/mod_expires.so  
    （14）允许apache修改或清除传递到cgi或ssi页面的环境变量
        LoadModule env_module modules/mod_env.so
    （15）根据客户端请求头字段设置环境变量，如果不需要则屏蔽掉
        #LoadModule setenvif_module modules/mod_setenvif.so
    （16）生成描述服务器状态的页面
        #LoadModule status_module modules/mod_status.so
    （17）别名
        LoadModule alias_module modules/mod_alias.so
    （18）url地址重写模块
        LoadModule rewrite_module modules/mod_rewrite.so
    （19）jk_mod 负载均衡调度模块
        LoadModule    jk_module modules/mod_jk.so 
    （20）过滤模块，使用缓存必须启用过滤模块
        LoadModule filter_module modules/mod_filter.so 
    （21）关闭服务器版本信息
        LoadModule version_module modules/mod_version.so
    （22）自动修正用户输入的url错误 
        LoadModule speling_module modules/mod_speling.so
```

#### 添加监听

在/etc/apache2/conf-enabled/monitor.conf添加如下内容

```xml
<location /server-status> //server-status 这个名字可以任意的取
　　SetHandler server-status
　　Order Deny,Allow
　　Deny from nothing //禁止的访问地址,nothing 表示没有禁止访问的地址
　　Allow from all //表示允许的地址访问；all 表示所有的地址都可以访问
</location> 
ExtendedStatus On //表示的是待会访问的时候能看到详细的请求信息
<Location /server-info> SetHandler server-info
　　Order allow,deny
　　Deny from nothing
　　Allow from all 
</Location>
```

重启apache访问
 http://IP地址：端口/server-status
 http://IP地址：端口/server-info
 http://IP地址：端口/server-status ?refresh=N
 N将表示访问状态页面可以每N秒自动刷新一次

#### 压缩配置

apache通过`mod_deflate`模块实现页面压缩，要想进行页面压缩必须启用以下两个模块

```
LoadModule deflate_module modules/mod_deflate.so  
LoadModule filter_module modules/mod_filter.so
```

页面压缩模块配置

```
<ifmodule mod_deflate.c>  
#设定压缩率，压缩率1 -9, 6是建议值，不能太高，消耗过多的内存，影响服务器性能  
DeflateCompressionLevel 6  

AddOutputFilterByType DEFLATE text/plain  
AddOutputFilterByType DEFLATE text/html  
AddOutputFilterByType DEFLATE text/php  
AddOutputFilterByType DEFLATE text/xml  
AddOutputFilterByType DEFLATE text/css  
AddOutputFilterByType DEFLATE text/javascript  
AddOutputFilterByType DEFLATE application/xhtml+xml  
AddOutputFilterByType DEFLATE application/xml  
AddOutputFilterByType DEFLATE application/rss+xml  
AddOutputFilterByType DEFLATE application/atom_xml  
AddOutputFilterByType DEFLATE application/x-javascript  
AddOutputFilterByType DEFLATE application/x-httpd-php  
AddOutputFilterByType DEFLATE application/x-httpd-fastphp  
AddOutputFilterByType DEFLATE application/x-httpd-eruby  
AddOutputFilterByType DEFLATE image/svg+xml  
AddOutputFilterByType DEFLATE application/javascript  

#插入过滤器  
SetOutputFilter DEFLATE  

#排除不需要压缩的文件  
SetEnvIfNoCase Request_URI \.(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.(?:gif|jpe?g|png)$ no-gzip don’t-vary  
SetEnvIfNoCase Request_URI \.pdf$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.avi$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.mov$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.mp3$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.mp4$ no-gzip dont-vary  
SetEnvIfNoCase Request_URI \.rm$ no-gzip dont-vary  
</ifmodule>
```

#### keepAlive

在HTTP 1.0中和Apache服务器的一次连接只能发出一次HTTP请求，而KeepAlive参数支持HTTP  1.1版本的一次连接，多次传输功能，这样就可以在一次连接中发出多个HTTP请求。从而避免对于同一个客户端需要打开不同的连接。很多请求通过同一个  TCP连接来发送，可以节约网络和系统资源。

```
（1）keepAlive启用场景
    如果有较多的js,css,图片访问，则需要开启长链接
    如果内存较少，大量的动态页面请求，文件访问，则关闭长链接，节省内存，提高apache访问的稳定性
    如果内存充足，cpu较好，服务器性能优越，则是否开启长链接对访问性能都不会产生影响
（2）keepAlive配置
    在Apache的配置文件httpd.conf中，设置：
    1、Timeout  60 默认为60s修改为30s
    2、KeepAlive on  设置为on状态
    4、KeepAliveTimeout 默认为5s,如果值设置过高，由于每个进程都要保持一定时间对应该用户，而无法应付其他用户请求访问，从而导致服务器性能下降。
    5、MaxKeepAliveRequests 100  如果设置为0表示无限制，建议最好设置一个值  
    把MaxKeepAliveRequests设置的尽量大，可以在一次连接中进行更多的HTTP请求。但在我们的测试中还发现，把 MaxKeepAliveRequests设置成1000，则评测的客户端容易出现“Send requesttimed out”的错误，所以具体数值还要根据自己的情形来设置。
```

#### 其他参数设置

- HostnameLookups 和其他DNS考虑

在Apache1.3以前的版本中,HostnameLookups默认被设为 On 。它会带来延迟,因为对每一个请求都需要作一次DNS查询。现在的版本默认都是off。

- FollowSymLinks 和 SymLinksIfOwnerMatch

如果网站空间中没有使用 Options FollowSymLinks ,或使用了 `Options SymLinksIfOwnerMatch` ,Apache就必须执行额外的系统调用以验证符号连接。文件名的每一个组成部分都需要一个额外的调用。例如,如果设置了:

```
DocumentRoot /www/htdocs
<Directory />
Options SymLinksIfOwnerMatch
</Directory>
```

在请求"/index.html"时,Apache将对"/www"、"/www/htdocs"、"/www/htdocs/index.html"执行lstat()调用。而且lstat()的执行结果不被缓存,因此对每一个请求都要执行一次。如果确实需要验证符号连接的安全性,则可以这样:

```
DocumentRoot /www/htdocs
<Directory />
Options FollowSymLinks
</Directory>
<Directory /www/htdocs>
Options -FollowSymLinks +SymLinksIfOwnerMatch
</Directory>
```

这样,至少可以避免对DocumentRoot路径的多余的验证。注意,如果Alias或RewriteRule中含有DocumentRoot以外的路径,那么同样需要增加这样的段。为了得到最佳性能,应当放弃对符号连接的保护,在所有地方都设置FollowSymLinks ,并放弃使用SymLinksIfOwnerMatch 。

- AllowOverride

如果网站空间允许覆盖(通常是用.htaccess文件),则Apache会试图对文件名
的每一个组成部分都打开.htaccess ,例如:

```
DocumentRoot /www/htdocs
<Directory />
AllowOverride all
</Directory>
```

如果请求"/index.html",则Apache会试图打开"/.htaccess"、"/www/.htaccess"、"/www/htdocs/.htaccess"。其解决方法和前面所述的Options FollowSymLinks 类似。为了得到最佳性能,应当对文件系统中所有的地方都使用AllowOverride None 。

- DirectoryIndex index 而使用完整的列表,如:

  ```
  DirectoryIndex index.cgi index.pl index.shtml index.html 
  ```

  其中最常用的应该放在前面。

到此,我们就对Apache做了一次全面优化,性能比原来明显地有了很大的提高.这次实施过程到此也就圆满的结束了.相信大家通过读完我的这篇文章后,对Apache优化也有了一些心得,相信你在工作中也会处理好突发事件

### 用ab对性能进行测试

用ab对Apache负载均衡集群的性能测试:

Apache服务器自带有一个叫AB(ApacheBench)的工具,在bin目录下使用这个工具可以对服务器进行负载测试。

用法:`ab -n 全部请求数 -c 并发数 测试url`

例如:

```
$ /opt/servers/httpd/bin/ab -n 1000 -c 100 http://127.0.0.1:8088/
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        Apache/2.4.39
Server Hostname:        127.0.0.1
Server Port:            8088

Document Path:          /
Document Length:        45 bytes

Concurrency Level:      100
Time taken for tests:   0.190 seconds
Complete requests:      1000
Failed requests:        420
   (Connect: 0, Receive: 0, Length: 420, Exceptions: 0)
Non-2xx responses:      420
Total transferred:      372160 bytes
HTML transferred:       151680 bytes
Requests per second:    5250.58 [#/sec] (mean)
Time per request:       19.046 [ms] (mean)
Time per request:       0.190 [ms] (mean, across all concurrent requests)
Transfer rate:          1908.26 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    4   1.6      4       7
Processing:     4   14   3.1     14      28
Waiting:        1   13   3.3     13      26
Total:          8   18   2.9     18      34

Percentage of the requests served within a certain time (ms)
  50%     18
  66%     18
  75%     19
  80%     19
  90%     20
  95%     23
  98%     28
  99%     29
 100%     34 (longest request)
```



### 问题集锦

1. 加载

```
LoadModule authz_core_module modules/mod_authz_core.so  
Invalid command 'Require', perhaps misspelled or defined by a module not included in the server configuration`
```

2. 配置信息后面不能跟随注释，注释必须另起一行

```
CacheDefaultExpire takes one argument, The default time in seconds to cache a document
```

3. 关键字错误 AddOutputFileByType 应该是

```
AddOutputFitlerByType  
Invalid command 'AddOutputFileByType', perhaps misspelled or defined by a module not included in the server configuration
```

4. 启用

```
LoadModule setenvif_module modules/mod_setenvif.so  
Invalid command 'SetEnvIfNoCase', perhaps misspelled or defined by a module not included in the server configuration
```

5. ifModule注释不能跟在配置参数后面，否则会导致配置解析失败

```
AH00526: Syntax error on line 558 of /usr/local/cp-httpd-2.4.18/conf/httpd.conf:
CacheDefaultExpire takes one argument, The default time in seconds to cache a document
```



### 参考：

1. [Apache的各种优化以及安全配置详解](https://blog.csdn.net/kangshuo2471781030/article/details/79174066)