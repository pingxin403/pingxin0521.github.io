---
title: Apache Httpd入门
date: 2019-05-31 11:18:59
tags:
 - 服务器
 - Apache
 - Httpd
categories:
 - 服务器
 - Apache Httpd
---

http协议是C/S架构：我们可以把浏览器（如：IE,Firefox,Safari,Chrome,Opera）看做客户端，当然我们也可以用命令行（elinks,curl）当做一个客户端。而服务器端就特别多，比如擅长处理静态页面的服务器httpd，lighttpd，nginx，gws等等，还有一些应用程序服务器IIS,Tomcat，jetty，resin等等。客户端和服务器之间的交互都是使用http协议的request（请求报文）和response（响应报文来实现的）。其实http协议非常简单，就是由着两种格式的报文组成，即客户端请求报文（request）和服务端响应报文（response）。

httpd俗称Apache。其实早期它是美国的一个官方组织所研发的一款web服务器。早期也就叫做httpd，后来httpd这个项目完成了，功能基本上也就完善了，大家知道一个项目完成之后就会解散项目成员，后来这帮开发httpd的程序员就散入了各个大公司，但是他们都很喜欢这个httpd，也不愿意看见它没落下去，于是他们就通过互联网自发组织起来来维护这个web服务器，一旦发现它有漏斗就打补丁，如果需要新功能就开发新特性，这就是早起的社区模型。随着时间的推移，这些牛逼的开发者对httpd进行各种打补丁，使得其功能越来越强大。后来这些开发人员就说这个是一个打满补丁的服务器（a pachey server).

<!--more-->

httpd是一个高度模块化组成的。即它的功能都是模块化的。

**DSO（Dynamic Shared Object）**

也就意味着你在安装httpd的时候，可以自定义选择你想要安装用到它的功能，它有些功能不不需要的可以选择不安装

**MPM（Multipath Processing Module)**

多道处理模块，非一个模块，而是对一种特性的称谓。

- prefork（多进程模型）:一个进程响应一个请求,需要用到select(),其最大可以设置的空闲线程上线为1024，它提前创建出来这些空闲的线程就是为了响应客户请求，因此等到客户来时再去创建线程（tast_struch的创建是需要时间的）会花费时间；
- worker（多线程模型）:一个进程多个线程，一个线程一个请求。一个主进程可以生成多个子进程，每个子进程又可以生成多个线程（注意：每个在某一个时刻线程可以响应一个请求，只有这个线程将请求响应结束后该线程才可以继续响应其他的请求哟~）提供服务。当一个子进程生成的线程都不工作了之后，主进程就会想法杀掉该子进程以达到释放空间的目的；
- event（事件模型）:一个线程相应多个请求，（envent-driven，事件驱动，主要目的是为了实现单线程相应多个请求。）基于事件驱动维持多个用户请求；

**http的功能特性**

- 路径别名：alias
- 用户认证：authentication
- 虚拟主机：virtual host
- 反向代理：（如负载均衡）
- 用户站点：当前用户都可以自行打开创建一个自己的站点。
- CGI：Common Gateway Interface等等。

#### 安装httpd

##### 源码编译安装

httpd-2.4.26+apr-1.5.2+apr-util-1.5.4

1. 安装 APR

   - 下载源码包 。（注意版本）下载地址:http://apr.apache.org

   - 安装步骤：首先需要解压tar.gz文件,然后进入解压后的文件夹。

     ```
     ./configure --prefix=/usr/local/share/apr   #--prefix 是指定安装的目录
     make
     make install
     ```

   - 注意:prefix必须一次写正确,否则无法卸载重新指定目录

2. 安装APR-UTIL

  - 下载源码包，下载地址: http://apr.apache.org

  - 安装步骤

    ```
    apt-get install libexpat1-dev #用apt安装 expat的开发库：
    ./configure --prefix=/usr/local/share/apr-util --with-apr=/usr/local/share/apr
    # --prefix 是指定安装的目录,同时需要指定apr的目录。
    make
    make install
    ```

  - 注意:prefix必须一次写正确,否则无法卸载重新指定目录。

3. 安装pcre

   从http://www.pcre.org下载源代码，安装在/usr/local/share/pcre目录下：

   ```
   ./configure --prefix=/usr/local/share/pcre
   make
   make install
   ```

   如果没有安装g++编译环境，执行`apt-get install build-essential`安装。

4. 安装httpd

- 从官网下载源码包，官网：http://httpd.apache.org/

- 安装步骤

  ```
  ./configure --prefix=/opt/servers/httpd --with-apr-utils=/usr/local/share/apr-util --with-apr=/usr/local/share/apr --with-pcre=/usr/local/share/pcre
  ```

  编译过程略有区别，若从git或svn上获取的源代码，需要执行`./buildconf`脚本生成configure文件。我在这个过程以此安装了python（Ubuntu 18.04默认安装python3）和libtool（`apt install libtool-bin`）。

参考：<https://www.jianshu.com/p/f2c4dfbfa5dd>

> RadHat：
>
> sudo yum -y install httpd
>
> Ubuntu：
>
> sudo apt install apache2

修改conf/httpd.conf对应内容，在对应注释下修改。

```
# sudo vim conf/http.conf
 Listen 8088 ## 配置端口
ServerName localhost:80  	## 将原来的‘#ServerName www.example.com:80’替换
```

并且更改logs文件夹权限：

```
# sudo chmod 777 -R /opt/servers/httpd/logs
```

运行

```
# /opt/servers/httpd/bin/httpd
```

查看：<localhost:8088>

##### 安装二进制包

Ubuntu 18.04

```
sudo apt install apache2 apache2-utils
```

##### Docker 安装 Apache

查找Docker Hub上的httpd镜像

```
$ docker search httpd
```

这里我们拉取官方的镜像

```
$ docker pull httpd
```

等待下载完成后，我们就可以在本地镜像列表里查到REPOSITORY为httpd的镜像。

```
$ docker images httpd
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              ee39f68eb241        2 weeks ago         154MB
```

运行容器

```
docker run -p 80:80 -v ~/docker/httpd/htdocs/:/usr/local/apache2/htdocs/ -v ~/docker/httpd/conf/httpd.conf:/usr/local/apache2/conf/httpd.conf -v ~/docker/httpd/logs/:/usr/local/apache2/logs/ --name httpd -d httpd
```

命令说明：

**-p 80:80 :**将容器的80端口映射到主机的80端口

查看容器启动情况

```
$ docker ps
```

通过浏览器访问本地80端口

#### 工作目录功能

根据安装方式不同，目录也是不同：

**使用安装包**（根据不同平台版本，不同软件版本可能会有所不同）：

```
[root@localhost ~]# rpm -ql httpd
/etc/httpd           #配置文件和安装文件根目录     
/etc/httpd/conf  #主配置文件目录
/etc/httpd/conf.d  #子配置文件目录，需要在主配置文件中包含，默认就包含
/etc/httpd/conf.d/README #说明文件
/etc/httpd/conf.d/welcome.conf  #如果用户没有指定，这个文件表示默认的主页文件
/etc/httpd/conf/httpd.conf  #主配置文件
/etc/httpd/conf/magic  #如果mod_mime无法解析，magic是另外一个用来设备文件类型的文件
/etc/httpd/logs  #日志文件目录，指向的是/var/log/httpd的符号链接
/etc/httpd/modules #httpd的模块文件的目录，指向的是/usr/lib64/httpd/modules的符号链接
/etc/httpd/run  #pid文件存放路径，指向的是/var/run/httpd的符号链接
/etc/logrotate.d/httpd #httpd的日志切割轮训脚本，被rsyslog托管
/etc/rc.d/init.d/htcacheclean #清理httpd磁盘换成的一个脚本文件
/etc/rc.d/init.d/httpd   #httpd的服务管理的脚本文件
/etc/sysconfig/htcacheclean  #htcacheclean脚本文件的(选项设置)配置文件
/etc/sysconfig/httpd  #httpd脚本文件的(选项设置)配置文件
/usr/lib64/httpd   #动态链接文件主目录
/usr/lib64/httpd/modules  #httpd的模块文件存放路径，下面的以*.so结尾的都是默认httpd安装后支持的模块
/usr/sbin/apachectl   #调用httpd脚本管理httpd服务的一个集成脚本工具，apachectl
/usr/sbin/htcacheclean   #清理httpd的磁盘缓存的主程序文件
/usr/sbin/httpd  #httpd的主程序文件
/usr/sbin/httpd.event   #event模型的httpd的主程序文件，默认的httpd是prefork
/usr/sbin/httpd.worker  #worker模型的httpd的主程序文件
/usr/sbin/httxt2dbm    #一个数据库管理组件(为 RewriteMap 创建 dbm 文件。)
/usr/sbin/rotatelogs  #httpd自带的日志切割轮训脚本
/usr/sbin/suexec   #执行外部程序前切换用户，这是一个有SUID权限的主程序文件
/usr/share/doc/httpd-2.2.15   #文档路径，下面也是
/usr/share/man/man8/apachectl.8.gz   #man手册
/var/cache/mod_proxy  #负载均衡缓存路径
/var/lib/dav
/var/log/httpd   #日志文件路径
/var/run/httpd  #pid文件路径
/var/www   #站点资源文件跟路径
/var/www/cgi-bin  #cgi程序路径
/var/www/error  #出现错误时相关
/var/www/html   #默认的主页文件路径，可以自己定义主页放在这个目录下
/var/www/icons  #下面都是一些图片之类的(图标相关的)
```

配置目录 `/etc/apache2`:

```
$ tree -L 1 /etc/apache2/
/etc/apache2/
├── apache2.conf
├── conf-available #可用的配置
├── conf-enabled #启用的配置，是conf-available下的软链接
├── envvars	#环境变量 
├── magic	
├── mods-available # 已安装的模块
├── mods-enabled	#已启用的模块，是mods-available下的软链接
├── ports.conf #httpd服务端口信息,修改端口的话，/etc/apache2/ports.conf文件中
|				#修改NameVirtualHost *:80 改为NameVirtualHost x.x.x.x:80
├── sites-available #可用站点信息
└── sites-enabled #已经启用的站点信息，当中的文件是到/etc/apache2/sites-available/ 文件的软连接。
```

配置完apache服务器后，重点就是要指定项目根目录的位置，Ubuntu默认是/var/www。

#### 源码：

```
bin: 该目录用于存放apache常用的命令,比如httpd
cig-bin:该目录存放linux下的常用命令 .sh
conf:存放配置文件httpd.conf，在httpd文件中可以对Apache进行配置。
error:apache用于存放启动或关闭的错误日志。
htdocs:存放站点的文件，如果有多个站点可以通过文件夹来分类。
icons:存放图标
logs:记录Apache的相关日志。
manual:手册
modules:Apached的模块

Apache可以同时监听多个端口。
```

Apache启动时，会读取modules下面的模块，会同时加载多个模块到内存中。是通过管理模块的方式来管理Apache

几个比较重要的文件夹：bin\conf\htdocs\modules

**bin下可执行文件**

```
 ab  压力测试工具：ab [options] [http[s]://]hostname[:port]/path
 apachectl  启动命令，[start|stop|restart|graceful]
 apxs  安装扩展模块的工具
 htcapacheclean  清理磁盘缓冲区命令
 htpasswd  建立和更新基本认证文件
 httpd  控制命令程序，apachectl执行时会调用
 rotatelogs  日志轮询工具，会用cronolog替代
```

  **conf  配置文件目录**

```
httpd.conf  主配置文件
 extra  子配置文件目录
```

**htdocs  默认站点目录**

```
index.html,index.php,index.jsp   首页文件
首页名字在httpd.conf中事先定义好了
DirectoryIndex index.html  
修改好后要重启httpd服务
apachectl -t 或 apachectl graceful(平滑重启)
```

 **logs  日志**        

```
access_log  访问日志
error_log  错误日志
httpd.pid  http进程启动后，会把所有进程的id号写进来
```

**modules模块目录**

具体配置查看[文档](http://httpd.apache.org/docs/2.4/mod/)。

**子配置文件介绍**

| mod_actions         | 基于媒体类型或请求方法，为执行CGI脚本而提供                  |
| ------------------- | ------------------------------------------------------------ |
| mod_alias           | 提供从文件系统的不同部分到文档树的映射和URL重定向            |
| mod_asis            | 发送自己包含HTTP头内容的文件                                 |
| mod_auth_basic      | 使用基本认证                                                 |
| mod_auth_digest     | 使用MD5摘要认证(更安全，但是只有最新的浏览器才支持)          |
| mod_authn_alias     | 基于实际认证支持者创建扩展的认证支持者，并为它起一个别名以便于引用 |
| mod_authn_anon      | 提供匿名用户认证支持                                         |
| mod_authn_dbd       | 使用SQL数据库为认证提供支持                                  |
| mod_authn_dbm       | 使用DBM数据库为认证提供支持                                  |
| mod_authn_default   | 在未正确配置认证模块的情况下简单拒绝一切认证信息             |
| mod_authn_file      | 使用纯文本文件为认证提供支持                                 |
| mod_authnz_ldap     | 允许使用一个LDAP目录存储用户名和密码数据库来执行基本认证和授权 |
| mod_authz_dbm       | 使用DBM数据库文件为组提供授权支持                            |
| mod_authz_default   | 在未正确配置授权支持模块的情况下简单拒绝一切授权请求         |
| mod_authz_groupfile | 使用纯文本文件为组提供授权支持                               |
| mod_authz_host      | 供基于主机名、IP地址、请求特征的访问控制                     |
| mod_authz_owner     | 基于文件的所有者进行授权                                     |
| mod_authz_user      | 基于每个用户提供授权支持                                     |
| mod_autoindex       | 自动对目录中的内容生成列表，类似于"ls"或"dir"命令            |
| mod_cache           | 基于URI键的内容动态缓冲(内存或磁盘)                          |
| mod_cern_meta       | 允许Apache使用CERN httpd元文件，从而可以在发送文件时对头进行修改 |
| mod_cgi             | 在非线程型MPM(prefork)上提供对CGI脚本执行的支持              |
| mod_cgid            | 在线程型MPM(worker)上用一个外部CGI守护进程执行CGI脚本        |
| mod_charset_lite    | 允许对页面进行字符集转换                                     |
| mod_dav             | 允许Apache提供DAV协议支持                                    |
| mod_dav_fs          | 为mod_dav访问服务器上的文件系统提供支持                      |
| mod_dav_lock        | 为mod_dav锁定服务器上的文件提供支持                          |
| mod_dbd             | 管理SQL数据库连接，为需要数据库功能的模块提供支持            |
| mod_deflate         | 压缩发送给客户端的内容                                       |
| mod_dir             | 指定目录索引文件以及为目录提供"尾斜杠"重定向                 |
| mod_disk_cache      | 基于磁盘的缓冲管理器                                         |
| mod_dumpio          | 将所有I/O操作转储到错误日志中                                |
| mod_echo            | 一个很简单的协议演示模块                                     |
| mod_env             | 允许Apache修改或清除传送到CGI脚本和SSI页面的环境变量         |
| mod_example         | 一个很简单的Apache模块API演示模块                            |
| mod_expires         | 允许通过配置文件控制HTTP的"Expires:"和"Cache-Control:"头内容 |
| mod_ext_filter      | 使用外部程序作为过滤器                                       |
| mod_file_cache      | 提供文件描述符缓存支持，从而提高Apache性能                   |
| mod_filter          | 根据上下文实际情况对输出过滤器进行动态配置                   |
| mod_headers         | 允许通过配置文件控制任意的HTTP请求和应答头信息               |
| mod_ident           | 实现RFC1413规定的ident查找                                   |
| mod_imagemap        | 处理服务器端图像映射                                         |
| mod_include         | 实现服务端包含文档(SSI)处理                                  |
| mod_info            | 生成Apache配置情况的Web页面                                  |
| mod_isapi           | 仅限于在Windows平台上实现ISAPI扩展                           |
| mod_ldap            | 为其它LDAP模块提供LDAP连接池和结果缓冲服务                   |
| mod_log_config      | 允许记录日志和定制日志文件格式                               |
| mod_log_forensic    | 实现"对比日志"，即在请求被处理之前和处理完成之后进行两次记录 |
| mod_logio           | 对每个请求的输入/输出字节数以及HTTP头进行日志记录            |
| mod_mem_cache       | 基于内存的缓冲管理器                                         |
| mod_mime            | 根据文件扩展名决定应答的行为(处理器/过滤器)和内容(MIME类型/语言/字符集/编码) |
| mod_mime_magic      | 通过读取部分文件内容自动猜测文件的MIME类型                   |
| mod_negotiation     | 提供内容协商支持                                             |
| mod_nw_ssl          | 仅限于在NetWare平台上实现SSL加密支持                         |
| mod_proxy           | 提供HTTP/1.1的代理/网关功能支持                              |
| mod_proxy_ajp       | mod_proxy的扩展，提供Apache JServ Protocol支持               |
| mod_proxy_balancer  | mod_proxy的扩展，提供负载平衡支持                            |
| mod_proxy_connect   | mod_proxy的扩展，提供对处理HTTP CONNECT方法的支持            |
| mod_proxy_ftp       | mod_proxy的FTP支持模块                                       |
| mod_proxy_http      | mod_proxy的HTTP支持模块                                      |
| mod_rewrite         | 一个基于一定规则的实时重写URL请求的引擎                      |
| mod_setenvif        | 根据客户端请求头字段设置环境变量                             |
| mod_so              | 允许运行时加载DSO模块                                        |
| mod_speling         | 自动纠正URL中的拼写错误                                      |
| mod_ssl             | 使用安全套接字层(SSL)和传输层安全(TLS)协议实现高强度加密传输 |
| mod_status          | 生成描述服务器状态的Web页面                                  |
| mod_suexec          | 使用与调用web服务器的用户不同的用户身份来运行CGI和SSI程序    |
| mod_unique_id       | 为每个请求生成唯一的标识以便跟踪                             |
| mod_userdir         | 允许用户从自己的主目录中提供页面(使用"/~username")           |
| mod_usertrack       | 使用Session跟踪用户(会发送很多Cookie)，以记录用户的点击流    |
| mod_version         | 提供基于版本的配置段支持                                     |
| mod_vhost_alias     | 提供大批量虚拟主机的动态配置支持                             |
| libmysql            | mysql模块                                                    |
| php5apache2_2       | php模块                                                      |

 **httpd-vhost.conf（重要）  //大部分网站都在这里面配置**

```
    NameVirtualHost *:80
        <VirtualHost *:80>  //基于域名的虚拟主机
            ServerAdmin webmaster@dummy-host.example.com
            DocumentRoot "/application/apache2.2.31/docs/dummy-host.example.com"
            ServerName dummy-host.example.com
            ServerAlias www.dummy-host.example.com
            ErrorLog "logs/dummy-host.example.com-error_log"
            CustomLog "logs/dummy-host.example.com-access_log" common
        </VirtualHost>
        <VirtualHost *:80>
            ServerAdmin webmaster@dummy-host2.example.com
            DocumentRoot "/application/apache2.2.31/docs/dummy-host2.example.com"
            ServerName dummy-host2.example.com
            ErrorLog "logs/dummy-host2.example.com-error_log"
            CustomLog "logs/dummy-host2.example.com-access_log" common
        </VirtualHost>
```

   **httpd-mpm.conf（重要）**

```
        <IfModule !mpm_netware_module>  //pid文件
            PidFile "logs/httpd.pid"
        </IfModule>
        <IfModule !mpm_winnt_module>
        <IfModule !mpm_netware_module>
        LockFile "logs/accept.lock"  //锁文件
        </IfModule>
        </IfModule>
        <IfModule mpm_prefork_module>  //prefor模式，另一模式是worker
            StartServers          5
            MinSpareServers       5
            MaxSpareServers      10
            MaxClients          150  //默认并发数150
            MaxRequestsPerChild   0
        </IfModule>
        <IfModule mpm_worker_module>  //worker模式
            StartServers          2
            MaxClients          150
            MinSpareThreads      25
            MaxSpareThreads      75
            ThreadsPerChild      25
            MaxRequestsPerChild   0
        </IfModule>
        <IfModule mpm_beos_module>
            StartThreads            10
            MaxClients              50
            MaxRequestsPerThread 10000
        </IfModule>
        <IfModule mpm_netware_module>
            ThreadStackSize      65536
            StartThreads           250
            MinSpareThreads         25
            MaxSpareThreads        250
            MaxThreads            1000
            MaxRequestsPerChild      0
            MaxMemFree             100
        </IfModule>
        <IfModule mpm_mpmt_os2_module>
            StartServers           2
            MinSpareThreads        5
            MaxSpareThreads       10
            MaxRequestsPerChild    0
        </IfModule>
        <IfModule mpm_winnt_module>
            ThreadsPerChild      150
            MaxRequestsPerChild    0
        </IfModule>
```

 **httpd-default.conf（了解）**       

```
 Timeout 300  //超时时间s
 KeepAlive On  //保持连接状态
 MaxKeepAliveRequests 100  //最大连接数
 KeepAliveTimeout 5  //在同一连接上等待下一个请求的时间
 UseCanonicalName Off  
 AccessFileName .htaccess  //伪静态的语法写在.htaccess
 开发要用，需要给他权限 AllowOverride ALL
 ServerTokens Full  
 ServerSignature On  //隐藏apache版本  防黑客
 HostnameLookups Off
```

#### 基础配置

##### 修改监听的IP地址和接口

在主配置文件中，httpd服务监听IP地址和接口的语句格式为：

> Listen [IP-address:]portnumber [protocol]

此语句的使用有以下几点注意事项：

- IP-address可省略，表示0.0.0.0匹配全部IP；
- 此指令Listen可重复出现多次监听多个IP地址和端口；
- 修改监听的socket后，需重启服务进程方可生效；
- 若限制其必须通过ssl通信时，protocol需定义为https；

##### 使用长连接

长连接指的是tcp链接建立之后，每个资源获取完成后不全断开连接，而是继续等待其他资源请求；但是对于并发访问量较大的服务器，长连接的使用会使得后续某些请求无法得到正常的响应。对于这种情况， 可通过使用较短的长连接超时时长和设置较少的长连接请求数来缓解。
 conf/extre/ httpd-default.conf中配置命令为：

```
keepalive On|off  #是否启动长连接
keepAliveTimeout 15  #长连接超时时间
MaxKeepAliveRequests 100  #最多保持多少个长连接请求
```

##### 定义web目录

在httpd服务的主配置文件中，默认情况下`DocumentRoot "/var/www/html"`定义了默认web站点目录的路径。
 如需自定义默认的目录，需要找如下格式进行添加：

> httpd-2.2:
>  <Directory "/PATH/TO/FILE">
>  Options Indexes FollowSymLinks
>  AllowOverride None
>  Order allow,deny
>  Allow from all
>  </Directory\>

> httpd-2.4:
>  <Directory "/PATH/TO/FILE">
>  Options Indexes FollowSymLinks
>  AllowOverride None
>  Require all granted
>  </Directory\>

- options 包括以下可选参数：

|      参数      |                             说明                             |
| :------------: | :----------------------------------------------------------: |
|    Indexes     | 允许目录浏览，当客户仅指定要访问的目录，但没有指定要访问的文件，且目录下不存在默认文档时，显示该目录中的文件及子目录列表索引 |
|   MultiViews   | 允许内容协商的多重视图，允许返回指定的访问目录下的相关联的文件 |
|      All       | All包含了除MultiViews之外的所有特性，如没有指定options，默认为All |
|    ExecCGI     |                  允许在该目录下执行CGI脚本                   |
| FollowSymLinks |                   允许跟踪符号链接到源文件                   |
|    Includes    |                     允许服务器端包含功能                     |
| IncludesNoExec |           允许服务器端包含功能，但禁止执行CGI脚本            |
|      None      |                      不调用options参数                       |

- AllowOverride
   AllowOverride选项用于定义每个目录下.htaccess文件中的指令类型，但通常设置None。
- Order
   Order选项用于定义缺省的访问权限与Allow和Deny语句的处理顺序。
- Allow/Deny
   Allow和Deny语句可以针对客户机的域名或IP地址进行设置，以决定哪些客户机能够访问服务器。如：Allow from all或者Deny from 172.16.0.0/24等等。
- Require all granted
   此为http-2.4中的允许所有人访问的格式。除此还可以禁止某个IP或域名的访问，如：`Require not ip 1.1.1.1`、`Require not host host.example.com`或者禁止所有人访问`Require all denied`。

#### httpd的访问控制

##### 在Directory中基于IP地址实现访问控制

在http-2.2中基于IP地址的访问控制是利用allow和Deny参数来实现的，如下例子：

```
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from  [IP|NetAddr]  
    Deny from  [IP|NetAddr]  
</Directory>
```

其中NetAddr的格式可类似：172.16、172.16.0.0、172.16.0.0/16、172.16.0.0/255.255.0.0。
 而httpd-2.4中基于Ip地址访问的控制是利用Require参数来实现的，其中Require参数可混合使用，如下例子：

```
<Directory "/data/html">
    AllowOverride none
    Options none
    <RequireAll>
       Require ip [IP|NetAddr]  #允许访问的IP或网段
       Require not ip [IP|NetAddr]  #拒绝访问的Ip或网段
    </RequireAll>
</Directory>
```

此外httpd-2.4版本中还可以利用host名来进行访问控制，如：

```
<Directory "/data/html">
    AllowOverride none
    Options none
    <RequireAll>
      Require host google.com  #只允许来自域名为google.com的主机访问；
       Require not host www.magedu.com #不允许来自域名为www.magedu.com的主机访问；
    </RequireAll>
</Directory>
```

**使用案例**：
禁止主机IP192.168.0.100和109访问相应的主机页面：

```
<VirtualHost 192.168.0.108:80>
        DocumentRoot "/data/html"
        <Directory "/data/html">
                AllowOverride none
                Options none
                <RequireAll>
                        Require all granted  #允许所有主机访问
                        Require not ip 192.168.0.110 192.168.0.100  #禁止匹配的Ip访问
                </RequireAll>
        </Directory>
</VirtualHost>
```

##### 在Directory中基于用户的访问控制

在Directory中支持的认证方式有两种Basic明文认证和digest消息摘要认证，由于并不是所有浏览器都支持摘要认证，因此一般来说用的较多的是明文认证
 首先利用htpasswd命令生成认证的配置文件：

```
[root@localhost ~]# htpasswd -cb /data/userpasswd charlie 123456
Adding password for user charlie
[root@localhost ~]# htpasswd -b /data/userpasswd wch magedu
Adding password for user wch
[root@localhost ~]# cat /data/userpasswd 
charlie:$apr1$1.t1GT7Z$HFMLZT7SR5eF6i51efMo90
wch:$apr1$nzfsSQ4g$qvo8tPvRV5uwnAehOCmr9.
[root@localhost ~]# chcon -R --reference /var/www/ /data/userpasswd  #修改userpasswd文件的安全上下文
```

随后编辑httpd的主配置文件，设置用户认证：

```
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/data/html"  #修改默认的web目录
<Directory "/data/html">
                AllowOverride none
                Options none
                AuthType Basic  #设置认证类型为Basic
                AuthName "welcome to my server."  #设置认证提示
                AuthUserFile "/data/userpasswd"  #指定认证文件的路径
                Require user charlie wch  #指定允许访问的认证用户
</Directory>
```

##### 基于域的用户访问控制

httpd服务除了根据用户做访问控制之外，还能将用户划分为相应的域组，并根据域组来做相应的访问控制。下面为以刚才演示的用户控制为背景做的域组访问控制示列：
 首先创建域组文件：

```
[root@localhost ~]# vim /data/Usergroup
group1:charlie
group2:wch
```

编辑httpd的主配置文件：

```
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/data/html"
<Directory "/data/html">
        AllowOverride none
        Options none
        AuthType Basic
        AuthName "welcome to my server"
        AuthUserFile "/data/userpasswd"
        AuthGroupFile "/data/Usergroup"  #添加域组文件
        Require group group1  #选择允许认证访问的域组
</Directory>
```

#### 虚拟主机VirtualHost

学习了如何在定义httpd的web目录后，大家肯定都会跃跃欲试。但是经历实操之后，大家可能就会去想着创建第二个web目录，然后就发现创建的第二个web目录无法被正常读取访问。此时就需要利用到httpd服务的VirtualHost功能来帮助大家完成这个需求。
Apache虚拟主机就是在一个Apache服务器上配置多个虚拟主机，实现一个服务器提供多站点服务，其实就是访问同一个服务器上的不同目录。
虚拟主机支持三种访问方式：

- 基于IP的方式，需为每个虚拟主机准备至少一个Ip地址，其配置方式如下：

```
<VirtualHost 172.16.100.6:80>
    ServerName www.a.com  
    DocumentRoot "/www/a.com/htdocs"  #虚拟主机的web目录
</VirtualHost>
```

- 基于port的方式，需要为每个虚拟主机使用至少一个独立的port，其配置方式如下：

```
Listen 8080 #在指定其他端口时，需添加监听语句
<VirtualHost 172.16.100.6:8080>  #指定不同的port
    ServerName www.a.com
    DocumentRoot "/www/a.com/htdocs"
</VirtualHost>
```

- 基于FQDN的方式，为每个虚拟主机使用至少一个FQDN，其配置方式如下：

```
NameVirtualHost 172.16.100.6:80 #如果是httpd-2.2，需要在配置文件中添加此句
<VirtualHost 172.16.100.6:80>
    ServerName www.a.com  #指定FQDN
    DocumentRoot "/www/a.com/htdocs"
</VirtualHost>
<VirtualHost 172.16.100.6:80>
    ServerName www.b.net  #指定FQDN
    DocumentRoot "/www/b.net/htdocs"
</VirtualHost>
```

#### httpd.conf配置详解

常用配置指令说明

1. ServerRoot：服务器的基础目录，一般来说它将包含conf/和logs/子目录，其它配置文件的相对路径即基于此目录。默认为安装目录，不需更改。

   语法：`ServerRoot directory-path`

   如：　`ServerRoot "/usr/local/apache-2.2.6"`

   注意，此指令中的路径最后不要加 / 。

2. Listen：指定服务器监听的IP和端口。默认情况下Apache会在所有IP地址上监听。Listen是Apache2.0以后版本必须设置的指令，如果在配置文件中找不到这个指令，服务器将无法启动。

   语法：`Listen [IP-address:]portnumber [protocol]`

   Listen指令指定服务器在那个端口或地址和端口的组合上监听接入请求。如果只指定一个端口，服务器将在所有地址上监听该端口。如果指定了地址和端口的组合，服务器将在指

   定地址的指定端口上监听。可选的protocol参数在大多数情况下并不需要，若未指定该参数，则将为443端口使用默认的https 协议，为其它端口使用http协议。

   使用多个Listen指令可以指定多个不同的监听端口和/或地址端口组合。

   默认为：`Listen 80`

   如果让服务器接受80和8080端口上请求，可以这样设置：

   ```
   Listen 80
   
   Listen 8080
   ```

   如果让服务器在两个确定的地址端口组合上接受请求，可以这样设置：

   ```
   Listen 192.168.2.1:80
   
   Listen 192.168.2.2:8080
   ```

   如果使用IPV6地址，必须用方括号把IPV6地址括起来：

   `Listen [2001:db8::a00:20ff:fea7:ccea]:80`

3. LoadModule：加载特定的DSO模块。Apache默认将已编译的DSO模块存放于4.1目录结构小节中所示的动态加载模块目录中。

   语法：`LoadModule module filename`

   如：`LoadModule rewrite_module modules/mod_rewrite.so`

   如果filename使用相对路径，则路径是相对于ServerRoot所指示的相对路径。

   Apache配置文件默认加载所有已编译的DSO模块，笔者建议只加载如下模块：`authn_file、authn_default、 authz_host、authz_user、authz_default、auth_basic、dir、alias、filter、speling、 log_config、env、vhost_alias、setenvif、mime、negotiation、rewrite、deflate、 expires、headers、cache、file-cache、disk-cache、mem-cache。`

4. User：设置实际提供服务的子进程的用户。为了使用这个指令，服务器必须以root身份启动和初始化。如果你以非root身份启动服务器，子进程将不能够切换至非特权用户，并

   继续以启动服务器的原始用户身份运行。如果确实以root用户启动了服务器，那么父进程将仍然以root身份运行。

   用于运行子进程的用户必须是一个没有特权的用户，这样才能保证子进程无权访问那些不想为外界所知的文件，同样的，该用户亦需没有执行那些不应当被外界执行的程序的权限。强烈建议专门为Apache子进程建立一个单独的用户和组。一些管理员使用nobody用户，但是这并不能总是符合要求，因为可能有其他程序也在使用这个用户。

   例：`User daemon`

   > Unix/Linux中的daemon进程类似于Windows中的后台服务进程，一直在后台运行运行，例如http服务进程nginx，ssh服务进程sshd等。
   > 注意，其英文拼写为daemon而不是deamon。`

5. Group：设置提供服务的Apache子进程运行时的用户组。为了使用这个指令，Apache必须以root初始化启动，否则在切换用户组时会失败，并继续以初始化启动时的用户组运行

   例：`Group daemon`

6. ServerAdmin：设置在所有返回给客户端的错误信息中包含的管理员邮件地址。

   语法：`ServerAdmin email-address|URL`

   如果httpd不能将提供的参数识别为URL，它就会假定它是一个email-address ，并在超连接中用在mailto:后面。推荐使用一个Email地址，因为许多CGI脚本是这样认为的。如果你确实想使用URL，一定要保证指向一个你能够控制的服务器，否则用户将无法确保一定可以和你取得联系。

7. ServerName：设置服务器用于辨识自己的主机名和端口号。

   语法：`ServerName [scheme://]fully-qualified-domain-name[:port]`

   可选的'scheme://'前缀仅在2.2.3以后的版本中可用，用于在代理之后或离线设备上也能正确的检测规范化的服务器URL。

   当没有指定ServerName时，服务器会尝试对IP地址进行反向查询来推断主机名。如果在ServerName中没有指定端口号，服务器会使用接受请求的那个端口。

   为了加强可靠性和可预测性，建议使用ServerName显式的指定一个主机名和端口号。

   如果使用的是基于域名的虚拟主机，在<VirtualHost\>段中的ServerName将是为了匹配这个虚拟主机，在"Host:"请求头中必须出现的主机名。

8. DocumentRoot：设置Web文档根目录。

   语法：`DocumentRoot directory-path`

   在没有使用类似Alias这样的指令的情况下，服务器会将请求中的URL附加到DocumentRoot后面以构成指向文档的路径。

   如果directory-path不是绝对路径，则被假定为是相对于ServerRoot的路径。

   指定DocumentRoot时不应包括最后的"/"。

9. <Directory\>：<Directory\>和</Directory\>用于封装一组指令，使之仅对某个目录及其子目录生效。

   语法：`<Directory Directory-path> ... </Directory>`

   Directory-path可以是一个目录的完整路径，或是包含了Unix shell匹配语法的通配符字符串。在通配符字符串中，"?"匹配任何单个的字符，"*"匹配任何字符序列。也可以使

   用"[]"来确定字符范围。在"~" 字符之后也可以使用正则表达式。

   如果有多个(非正则表达式)<Directory\>配置段符合包含某文档的目录(或其父目录)，那么指令将以短目录优先的规则进行应用，并包含.htaccess文件中的指令。

   正则表达式将在所有普通配置段之后予以考虑。所有的正则表达式将根据它们出现在配置文件中的顺序进行应用。

   <Directory\>指令不可被嵌套使用，也不能出现在<Limit\>或<LimitExcept\>配置段中。

10.  <Files\>：提供基于文件名的访问控制，类似于<Directory\>和<Location\>指令。

    语法：`<Files filename> ... </Files>`

    filename参数应当是一个文件名或是一个包含通配符的字符串，其中"?"匹配任何单个字符，"*"匹配任何字符串序列。在"~"字符之后可以使用正则表达式。

    在此配置段中定义的指令将作用于其基本名称(不是完整的路径)与指定的文件名相符的对象。<Files\>段将根据它们在配置文件中出现的顺序被处理：在<Directory\>段和.htaccess

    文件被处理之后，但在<Location\>段之前。<Files\>能嵌入到<Directory\>段中以限制它们作用的文件系统范围，也可用于.htaccess文件当中，以允许用户在文件层面上控制对它们

    自己文件的访问。

11. <IfModule\>：封装根据指定的模块是否启用而决定是否生效的指令。

    语法：`<IfModule [!]module-file|module-identifier> ... </IfModule>`

    module-file是指编译模块时的文件名，比如mod_rewrite.c　。

    module-identifier是指模块的标识符，比如mod_rewrite　。

    在<IfModule\>配置段中的指令仅当测试结果为真的时候才进行处理，否则所有其间的指令都将被忽略。

12. Options：控制在特定目录中将使用哪些服务器特性

    语法：`Options [+|-]option [[+|-]option] ...`

    option可以为None，不启用任何额外特性，或者下面选项中的一个或多个：

    - All 　除MultiViews之外的所有特性，这是默认设置。

    - ExecCGI 　允许使用mod_cgi执行CGI脚本。

    - FollowSymLinks 　服务器允许在此目录中使用符号连接，如果此配置位于<Location\>配置段中，则会被忽略。

    - Includes 　允许使用mod_include提供的服务器端包含。

    - IncludesNOEXEC 　允许服务器端包含，但禁用"#exec cmd"和"#exec cgi"，但仍可以从ScriptAlias目录使用"#include virtual"虚拟CGI脚本。

    - Indexes 　如果一个映射到目录的URL被请求，而此目录中又没有DirectoryIndex(例如：index.html)，那么服务器会返回由mod_autoindex生成的一个格式化后的目录列表。

    - MultiViews 　允许使用mod_negotiation提供内容协商的"多重视图"(MultiViews)。

    - SymLinksIfOwnerMatch 　服务器仅在符号连接与其目的目录或文件的拥有者具有相同的uid时才使用它。 如果此配置出现在<Location\>配置段中，则将被忽略。

    一般来说，如果一个目录被多次设置了Options ，则最特殊的一个会被完全接受(其它的被忽略)，而各个可选项的设定彼此并不融合。然而，如果所有作用于Options指令的可选项

    前都加有"+" 或"-"符号，此可选项将被合并。所有前面加有"+"号的可选项将强制覆盖当前的可选项设置，而所有前面有"-"号的可选项将强制从当前可选项设置中去除。

13. AllowOverride：确定允许存在于.htaccess文件中的指令类型。

    语法：`AllowOverride All|None|directive-type [directive-type] ...`

    如果此指令被设置为None ，那么.htaccess文件将被完全忽略。事实上，服务器根本不会读取.htaccess文件。

    当此指令设置为All时，所有具有".htaccess"作用域的指令都允许出现在.htaccess文件中。

    directive-type可以是下列各组指令之一：

    - AuthConfig 　允许使用与认证授权相关的指令

    - FileInfo 　允许使用控制文档类型的指令、控制文档元数据的指令、mod_rewrite中的指令、mod_actions中的Action指令

    - Indexes 　允许使用控制目录索引的指令

    - Limit 　允许使用控制主机访问的指令

    Options[=Option,...] 　允许使用控制指定目录功能的指令(Options和XBitHack)。可以在等号后面附加一个逗号分隔的(无空格的)Options选项列表，用来控制允许Options指令使用哪些选项。

    AllowOverride仅在不包含正则表达式的<Directory\>配置段中才是有效的。在<Location\>, <DirectoryMatch\>, <Files\>配置段中都是无效的。

    Order：控制默认的访问状态与Allow和Deny指令生效的顺序。

    Ordering取值范围是以下几种范例之一：

    - Deny,Allow 　Deny指令在Allow指令之前被评估。默认允许所有访问。任何不匹配Deny指令或者匹配Allow指令的客户都被允许访问。

    - Allow,Deny 　Allow指令在Deny指令之前被评估。默认拒绝所有访问。任何不匹配Allow指令或者匹配Deny指令的客户都将被禁止访问。

    - Mutual-failure 　只有出现在Allow列表并且不出现在Deny列表中的主机才被允许访问。这种顺序与"Order Allow,Deny"具有同样效果，不赞成使用。

    关键字只能用逗号分隔，它们之间不能有空格，在所有情况下每个Allow和Deny指令语句都将被评估。

    Allow：控制哪些主机可以访问服务器的该区域。可以根据主机名、IP地址、 IP地址范围或其他环境变量中捕获的客户端请求特性进行控制。

    语法：`Allow from all|host|env=env-variable [host|env=env-variable] ...`

    这个指令的第一个参数总是"from"，随后的参数可以有三种不同形式：如果指定"Allow from all"，则允许所有主机访问，按照下述Deny和Order指令的配置；若要只允许特定的主机或主机群访问服务器，host可以用下面任何一种格式来指定：一个（部分）域名、完整的IP地址、部分IP地址、网络/掩码、网络/nnn无类别域间路由规格；第三种参数格式允许对服务器的访问由环境变量的一个扩展指定，指定"Allow from env=env-variable"时，如果环境变量env-variable存在则访问被允许，使用由mod_setenvif提供的指令，服务器用一种基于客户端请求的弹性方式提供了设置环境变量的能力。因此，这条指令可以用于允许基于像User-Agent(浏览器类型)、Referer或其他 HTTP请求头字段的访问。

    Deny：控制哪些主机被禁止访问服务器的该区域。可以根据主机名、IP地址、 IP地址范围或其他环境变量中捕获的客户端请求特性进行控制。

    语法：`Deny from all|host|env=env-variable [host|env=env-variable] ...`

    此指令的参数设置和Allow指令完全相同。

14. DirectoryIndex：当客户端请求一个目录时寻找的资源列表。

    语法：`DirectoryIndex Local-url [Local-url] ...`

    Local-url(%已解码的)是一个相对于被请求目录的文档的URL(通常是那个目录中的一个文件)。可以指定多个URL，服务器将返回最先找到的那一个，比如：

    ```
    DirectoryIndex index.html index.php
    ```

15. ErrorLog：指定当服务器遇到错误时记录错误日志的文件。

    语法：`ErrorLog file-path|syslog[:facility]`

    如果file-path不是一个以斜杠(/)开头的绝对路径，那么将被认为是一个相对于ServerRoot的相对路径；如果file-path以一个管道符号(|)开头，那么会为它指定一个命令来处理

    错误日志，如 `ErrorLog "|/usr/local/sbin/cronolog /var/log/httpd/%w/errors_log"`　。

    如果系统支持，使用"syslog"替代文件名将通过 syslogd(8)来记载日志。默认将使用系统日志机制local7 ，但您可以用"syslog:facility"语法来覆盖这个设置，其中，facility的取值为syslog(1)中记载的任何一个名字。

16. LogLevel：用于调整记录在错误日志中的信息的详细程度。

    语法：`LogLevel level`

    可以选择下列level，依照重要性降序排列：

    - emerg 　紧急(系统无法使用)

    - alert 　必须立即采取措施

    - crit 　致命情况

    - error 　错误情况

    - warn 　警告情况

    - notice 　一般重要情况

    - info 　 普通信息

    - debug 　调试信息

    当指定了某个级别时，所有级别高于它的信息也会被同时记录。比如，指定 LogLevel info ，则所有notice和warn级别的信息也会被记录。建议至少使用crit级别。

    当错误日志是一个单独分开的正式文件的时候，notice级别的消息总是会被记录下来，而不能被屏蔽。但是，当使用syslog来记录时就没有这个问题。

17. LogFormat：定义访问日志的记录格式。

    语法：`LogFormat format|nickname [nickname]`

    LogFormat指令可以使用两种定义格式中的一种。

    在第一种格式中，指令只带一个参数，以定义后续的TransferLog指令定义的日志格式。另外它也可以通过下述的方法使用nickname来引用某个之前的LogFormat定义的日志格式。

    第二种定义LogFormat指令的格式中，将一个直接的format和一个nickname联系起来。这样在后续的LogFormat或 CustomLog指令中，就不用一再重复整个冗长的格式串。定义别名的LogFormat指令仅仅用来定义一个nickname ，而不做其它任何事情，也就是说，它只是定义了这个别名，它既没有实际应用这个别名，也不是把它设为默认的格式。因此，它不会影响后续的 TransferLog指令。另外，LogFormat不能用一个别名来定义另一个别名。nickname不能包含百分号(%)。

18. CustomLog：设定日志的文件名和格式。

    语法：`CustomLog file|pipe format|nickname [env=[!]environment-variable]`

    第一个参数指定了日志记录的位置，可以使用以下两种方式来设定：

    file 　相对于ServerRoot的日志文件名。
    pipe 　管道符"|"后面紧跟着一个把日志输出当作标准输入的处理程序路径。

    第二个参数指定了写入日志文件的内容。它既可以是由前面的LogFormat指令定义的nickname ，也可以是直接按Apache2.2官方文档中的自定义日志格式小节所描述的规则定义的format字符串。

    第三个参数是可选的，它根据服务器上特定的环境变量是否被设置来决定是否对某一特定的请求进行日志记录。如果这个特定的环境变量被设置(或者在"env=!name"的情况下未被设置)，那么这个请求将被记录。可以使用mod_setenvif和/或mod_rewrite模块来为每个请求设置环境变量。

19. TransferLog：指定日志文件的位置。

    语法：`TransferLog file|pipe`

    本指令除不允许直接定义日志格式或根据条件进行日志记录外，与CustomLog指令有完全相同的参数和功能。实际应用中，日志的格式是由最近的非别名定义的LogFormat指令指定。如果没有定义任何日志格式，则使用通用日志格式。

20. Alias：映射URL到文件系统的特定区域。

    语法：`Alias URL-path file-path|directory-path`

    Alias指令使文档可以被存储在DocumentRoot以外的本地文件系统中。以(%已解码的)url-path路径开头的URL可以被映射到以directory-path开头的本地文件。

    如果对在DocumentRoot之外的某个目录建立了一个Alias ，则可能需要通过<Directory\>段明确的对目标目录设定访问权限。

21. ScriptAlias：映射一个URL到文件系统并视之为CGI脚本目录。

    语法：`ScriptAlias URL-path file-path|directory-path`

    ScriptAlias指令的行为与Alias指令相同，但同时它又标明此目录中含有应该由cgi-script处理器处理的CGI脚本。以URL-path开头的(%已解码的)的URL会被映射到由第二个参数指定的具有完整路径名的本地文件系统中的脚本。

    ScriptSock：在以线程式MPM(worker)运行的Apache中设置用来与CGI守护进程通信的套接字文件名前缀(其后附加父进程 PID组成完整的文件名)。这个套接字将会用启动Apache服务器的父进程用户权限(通常是root)打开。为了维护与CGI脚本通讯的安全性，不允许其他用户拥有写入套接字所在目录的权限是很重要的。

22.  DefaultType：在服务器无法由其他方法确定内容类型时，发送的默认MIME内容类型。

    语法：`DefaultType MIME-type`

    默认：`DefaultType text/plain`

23.  AddType：在给定的文件扩展名与特定的内容类型之间建立映射关系。

    语法：`AddType MIME-type extension [extension] ...`

    MIME-type指明了包含extension扩展名的文件的媒体类型。这个映射关系会添加在所有有效的映射关系上，并覆盖所有相同的extension扩展名映射。

    extension参数是不区分大小的，并且可以带或不带前导点。

24.  ErrorDocument：批示当遇到错误的时候服务器将给客户端什么样的应答。

    语法：`ErrorDocument error-code document`

    - error-code 　服务器返回的错误代码

    - document 　可以由一个斜杠(/)开头来指示一个本地URL(相对于DocumentRoot)，或是提供一个能被客户端解释的完整的URL。此外还能提供一个可以被浏览器显示的消息。比如：

    - ErrorDocument 500http://www.entage.net/err500.html

    - ErrorDocument 404 /errors/bad_urls.html

    - ErrorDocument 403 "Sorry can't allow you access today"

25.  EnableMMAP：指示httpd在递送中如果需要读取一个文件的内容，它是否可以使用内存映射。

    语法：`EnableMMAP On|Off`

    当处理一个需要访问文件中的数据的请求时，比如说当递送一个使用mod_include进行服务器端分析的文件时，如果操作系统支持，Apache将默认使用内存映射。

    这种内存映射有时会带来性能的提高，但在某些情况下，您可能会需要禁用内存映射以避免一些操作系统的问题：

    - 在一些多处理器的系统上，内存映射会减低一些httpd的性能；

    - 在挂载了NFS的DocumentRoot上，若已经将一个文件进行了内存映射，则删除或截断这个文件会造成httpd因为分段故障而崩溃。

    - 在可能遇到这些问题的服务器配置过程中，应当使用下面的命令来禁用内存映射：

26. EnableMMAP Off

    对于挂载了NFS的文件夹，可以单独在<directory\>段中指定禁用内存映射：

    ```
    <Directory "/path-to-nfs-files">
    EnableMMAP Off
    </Directory>
    ```

27. EnableSendfile：控制httpd是否可以使用操作系统内核的sendfile支持来将文件发送到客户端。

    默认情况下，当处理一个请求并不需要访问文件内部的数据时(比如发送一个静态的文件内容)，如果操作系统支持，Apache将使用sendfile将文件内容直接发送到客户端而并不读取文件。

    这个sendfile机制避免了分开的读和写操作以及缓冲区分配，但是在一些平台或者一些文件系统上，最好禁止这个特性来避免一些问题：

    一些平台可能会有编译系统检测不到的有缺陷的sendfile支持，特别是将在其他平台上使用交叉编译得到的二进制文件运行于当前对sendfile支持有缺陷的平台时；

    在Linux上启用IPv6时，使用sendfile将会触发某些网卡上的TCP校验和卸载bug；

    当Linux运行在Itanium处理器上的时候，sendfile可能无法处理大于2GB的文件；

    对于一个通过网络挂载了NFS文件系统的DocumentRoot (比如：NFS或SMB)，内核可能无法可靠的通过自己的缓冲区服务于网络文件。

    如果出现以上情况，你应当禁用sendfile ：

    ```
    EnableSendfile Off
    ```

    针对NFS或SMB，可以单独在<directory\>段中指定禁用：

    ```
    <Directory "/path-to-nfs-files">
    EnableSendfile Off
    </Directory>
    ```