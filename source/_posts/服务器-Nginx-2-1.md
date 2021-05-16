---
title: nginx 热部署、虚拟主机和日志
date: 2019-08-21 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

#### 热部署

nginx 支持热加载 热部署 ，在不打断用户请求的情况下更新版本

<!--more-->

Nginx 只所以出名，和它内部的精密设计有关。Nginx 采用了高度模块化的设计思路，并且内部的进程主要有两类，master 进程 和 worker 进程。其中 master 进程只有一个，worker 进程可以有多个。

worker 进程才是真正 working 的进程，才是真正处理请求的进程。worker 进程全部都是 master 进程的子进程。worker 进程是以普通用户的身份进行运行的，这样就可以极大增加程序的安全性。就算是万一有一个进程被劫持，那也不会有管理员权限。

nginx 的热部署和其并发模型有着密不可分的关系。说白了，就是因为 master 进程的关系。当通知 ngnix 重读配置文件的时候，master 进程会进行语法错误的判断。如果存在语法错误的话，返回错误，不进行装载；如果配置文件没有语法错误，那么 ngnix 也不会将新的配置调整到所有 worker 中。而是，先不改变已经建立连接的 worker，等待 worker 将所有请求结束之后，将原先在旧的配置下启动的 worker 杀死，然后使用新的配置创建新的 worker。

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

#### 静态资源Web服务器

有两种方法部署，一是将静态资源文件夹移到nginx的`/uar/share/nginx/html`文件夹下；另一个则是使用虚拟主机。

可以通过 nginx 进行虚拟主机的配置，只需要在 `http {}` 中添加一个 `server {}` 模块即可。nginx 虚拟主机的配置，一般分为三种：域名、端口和 ip。

**类型介绍**

1. 基于域名的虚拟主机

   所谓基于域名的虚拟主机，意思就是通过不同的域名区分不同的虚拟主机，基于域名的虚拟主机是企业应用最广的虚拟主机类型，几乎所有对外提供服务的网站使用的都是基于域名的主机，例如www.test1.com www.test2.com等

2. 基于端口的虚拟主机

   同理，所谓基于端口的虚拟主机，意思就是通过不同的端口来区分不同的虚拟主机，此类虚拟主机对应的企业应用主要为公司内部的网站，例如：一些不希望直接对外提供用户访问的网站后台等，访问基于端口的虚拟主机，地址里要带有端口号，例如http://www.test.com:81 http://www.test.com:82等

3. 基于IP的虚拟主机

   同理，所谓基于IP的虚拟主机，意思就是通过不同的IP区分不同的虚拟主机，此类虚拟主机对应的企业应用非常少见，一般不同的业务需要使用多IP的场景都会在负载均衡上进行IP绑定，我不是在web上绑定IP来区分不同的虚拟机。

三种虚拟主机类型均可独立使用，也可以混合使用。

**基于多域名的虚拟主机配置**

基本步骤：修改nginx配置文件配置多域名，重启nginx服务，创建对应的不同站点目录并上传站点文件，也可都使用一个站点目录，通过多域名来访问

```
...
http {
...
server {
	listen 80;
	#server_name 指定虚拟主机的名字。可以指定多个名称，第一个为虚拟主机的名字。
	#可以使用 “ * ” 替代服务器名称的开始或者最后部分。
	server_name www.test1.com;
	location / {
#root 设置请求的根目录，可以用绝对路径或相对路径，如 root html; 会等于 root /usr/local/nginx/html; 。nginx安装目录为/usr/local/nginx/
#而这样设置，当收到一个 domain.cm/index.html 请求时，/usr/local/nginx/html/index.html 文件将会被发送在响应中响应该请求。
		root /usr/share/nginx/html/;
#index 定义将用做索引的文件。文件名称可以包含变量，按照指定的顺序进行文件检查的，最后一个参数可以是绝对路径。
#实际上 domain.cm 请求会被处理成 domain.cm/index.html 。
		index index.html index.htm;
	}
}
server {
	listen 80;
	server_name www.test2.com www.test3.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
server {
	listen 80;
	server_name www.test4.com;
	location / {
		root /data/html/;
		index index.html index.htm;
	}
}
...
}
...
```

**基于多端口的虚拟主机配置**

基本步骤：修改nginx配置文件配置多端口，重启nginx服务，修改安全组规则开放端口，创建对应的不同站点目录并上传站点文件，也可都使用一个站点目录，通过多端口来访问

```
...
http {
...
server {
	listen 81;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
server {
	listen 82;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
server {
	listen 83;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
...
}
...
```

**基于多IP的虚拟主机配置**

 基本步骤：增加网卡获得多ip或者增加辅助ip，修改nginx配置文件配置多ip，重启nginx服务，创建对应的不同站点目录并上传站点文件，也可都使用一个站点目录，通过多ip来访问

```
...
http {
...
server {
	listen 10.0.0.7:80;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
server {
	listen 10.0.0.8:80;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
server {
	listen 10.0.0.9:80;
	server_name www.test.com;
	location / {
		root /usr/share/nginx/html/;
		index index.html index.htm;
	}
}
...
}
...
```

增加辅助ip的方法:

1. 临时性增加辅助ip：

   方法一：`ifconfig eth0:1 10.0.0.8/24 up`

   方法二：ip addr

   ```
   ip addr add 10.0.0.9/24 dev eth0#（使用ip addr能查看）
   
   ip addr add 10.0.0.9/24 label eth0:2 dev eth0 #（使用ifconfig和ipaddr都能查看，推荐使用）
   ```

2. 永久增加辅助ip

   ```bash
   cd /etc/sysconfig/network-scripts/    #进入到网卡配置文件的目录
   
   cp ifcfg-eth0 ifcfg-eth0:1                #拷贝配置文件并重命名
   
   vim ifcfg-eth0:1                        #编辑配置文件
   
   /etc/init.d/network restart            #重启网络服务
   ```

#### 日志

Nginx日志主要分为两种：访问日志和错误日志。日志开关在Nginx配置文件（/etc/nginx/nginx.conf）中设置，两种日志都可以选择性关闭，默认都是打开的。

**访问日志**

访问日志主要记录客户端访问Nginx的每一个请求，格式可以自定义。通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。Nginx中访问日志相关指令主要有两条：

1. log_format

   log_format用来设置日志格式，也就是日志文件中每条日志的格式，具体如下：
   `log_format name(格式名称) type(格式样式)`

   举例说明如下：

   ```
   log_format main '$server_name $remote_addr - $remote_user [$time_local] "$request" '
   				'$status $uptream_status $body_bytes_sent "$http_referer" '
   				'"$http_user_agent" "$http_x_forwarded_for" '
   				'$ssl_protocol $ssl_cipher $upstream_addr $request_time $upstream_response_time';
   ```

   每个样式的含义如下：

   - $server_name：虚拟主机名称。
   - $remote_addr：远程客户端的IP地址。
   - -：空白，用一个“-”占位符替代，历史原因导致还存在。
   - $remote_user：远程客户端用户名称，用于记录浏览者进行身份验证时提供的名字，如登录百度的用户名scq2099yt，如果没有登录就是空白。
   - [$time_local]：访问的时间与时区，比如18/Jul/2012:17:00:01 +0800，时间信息最后的"+0800"表示服务器所处时区位于UTC之后的8小时。
   - $request：请求的URI和HTTP协议，这是整个PV日志记录中最有用的信息，记录服务器收到一个什么样的请求
   - $status：记录请求返回的http状态码，比如成功是200。
   - $uptream_status：upstream状态，比如成功是200.
   - $upstream_addr:后端服务器的IP地址
   - $upstream_status：后端服务器返回的HTTP状态码
     	在server{}中添加
       	add_header backendIP $upstream_addr;
       	add_header backendCode $upstream_status;
   - $body_bytes_sent：发送给客户端的文件主体内容的大小，比如899，可以将日志每条记录中的这个值累加起来以粗略估计服务器吞吐量。
   - $http_referer：记录从哪个页面链接访问过来的。 
   - $http_user_agent：客户端浏览器信息
   - $http_x_forwarded_for：客户端的真实ip，通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
   - $ssl_protocol：SSL协议版本，比如TLSv1。
   - $ssl_cipher：交换数据中的算法，比如RC4-SHA。 
   - $upstream_addr：upstream的地址，即真正提供服务的主机地址。 
   - $request_time：整个请求的总时间。 
   - $upstream_response_time：请求过程中，upstream的响应时间。

   访问日志中一个典型的记录如下：

   ```
   192.168.1.102 - scq2099yt [18/Mar/2013:23:30:42 +0800] "GET /stats/awstats.pl?config=scq2099yt HTTP/1.1" 200 899 "http://192.168.1.1/pv/" "Mozilla/4.0 (compatible; MSIE 6.0; Windows XXX; Maxthon)"
   ```

   需要注意的是：log_format配置必须放在http内，否则会出现如下警告信息：

   ```
   nginx: [warn] the "log_format" directive may be used only on "http" level in /etc/nginx/nginx.conf:97
   ```

2. access_log

   日志级别： `debug > info > notice > warn > error > crit > alert > emerg`

   access_log指令用来指定日志文件的存放路径（包含日志文件名）、格式和缓存大小，具体如下：

   ```
   access_log path(存放路径) [format(自定义日志格式名称) [buffer=size | off]]
   
   语法格式:   access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
                        access_log off;
   默认值   : access_log logs/access.log combined;
   作用域   : http, server, location, if in location, limit_except
   ```

   举例说明如下：

   ```
   access_log logs/access.log main;
   access_log /spool/logs/nginx-access.log compression buffer=32k;
   ```

   如果想关闭日志，可以如下：

   ```
   access_log off;
   ```

   能够使用access_log指令的字段包括：http、server、location。

   需要注意的是：Nginx进程设置的用户和组必须对日志路径有创建文件的权限，否则，会报错。

   Nginx支持为每个location指定强大的日志记录。同样的连接可以在同一时间输出到不止一个的日志中。

3. open_log_file_cache

   使用open_log_file_cache来设置日志文件缓存(默认是off)。

   - max:设置缓存中的最大文件描述符数量，如果缓存被占满，采用LRU算法将描述符关闭。
   - inactive:设置存活时间，默认是10s
   - min_uses:设置在inactive时间段内，日志文件最少使用多少次后，该日志文件描述符记入缓存中，默认是1次
   - valid:设置检查频率，默认60s
   - off：禁用缓存

   ```
   语法格式:   open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
                        open_log_file_cache off;
   默认值:     open_log_file_cache off;
   作用域:     http, server, location
   ```

   实例一

   ```
   open_log_file_cache max=1000 inactive=20s valid=1m min_uses=2;
   ```

**错误日志**

错误日志主要记录客户端访问Nginx出错时的日志，格式不支持自定义。通过错误日志，你可以得到系统某个服务或server的性能瓶颈等。因此，将日志好好利用，你可以得到很多有价值的信息。错误日志由指令error_log来指定，具体格式如下：

```
error_log path(存放路径) level(日志等级)
```

path含义同access_log，level表示日志等级，具体如下：

```
[ debug | info | notice | warn | error | crit ]
```

从左至右，日志详细程度逐级递减，即debug最详细，crit最少。

举例说明如下：

```
error_log logs/error.log info;
```

需要注意的是：error_log off并不能关闭错误日志，而是会将错误日志记录到一个文件名为off的文件中。
正确的关闭错误日志记录功能的方法如下：

```
error_log /dev/null;
```

上面表示将存储日志的路径设置为“垃圾桶”。

**日志分割/备份**

```bash
#!/bin/bash
#name:backup_nginx_log.sh
#desc:把当前日志按日期备份，重新生成第二天的日志文件
#date:2016-09-18
 
DATE=`date +%Y%m%d`
NGINX_PID=`cat /var/run/nginx.pid`
#如果当前Nginx没有运行就退出
if [ "$?" != 0 ]
then
    exit 1;
fi
 
#nginx 日志所在的路径
LOG_PATH='/var/log/nginx/'
LOG_NAME='access.log'
#避免重复运行而造成覆盖
if [ -f ${LOG_PATH}${LOG_NAME}$DATE ];then
	echo "日志已存在"
	exit 1
fi
mv ${LOG_PATH}${LOG_NAME} ${LOG_PATH}${LOG_NAME}$DATE
 
#删除7天前旧的备份文件
function deloldbak()
{
    olddate=`date +"%Y%m%d" -d "-$1 day"`
    if [ -e "${LOG_PATH}${LOG_NAME}$olddate" ]
    then
        rm -f ${LOG_PATH}${LOG_NAME}$olddate
        echo "${LOG_PATH}${LOG_NAME}$olddate del OK" 
    fi
}
 
#重载nginx配置，重新生成nginx日志文件
kill -USR1 $NGINX_PID
 
if [ "$?" == 0 ]
then
    deloldbak 7
    exit 0;
fi
```

**nginx日志调试技巧**

1. 设置 Nginx 仅记录来自于你的 IP 的错误

   当你设置日志级别成 debug，如果你在调试一个在线的高流量网站的话，你的错误日志可能会记录每个请求的很多消息，这样会变得毫无意义。

   在`events{...}`中配置如下内容，可以使 Nginx 记录仅仅来自于你的 IP 的错误日志。

   ```
   events {
           debug_connection 1.2.3.4;
   }
   ```

2. 调试 nginx rewrite 规则

   调试rewrite规则时，如果规则写错只会看见一个404页面，可以在配置文件中开启nginx rewrite日志，进行调试。

   ```
   server {
           error_log    /var/logs/nginx/example.com.error.log;
           rewrite_log on;
   }
   ```

   `rewrite_log on;` 开启后，它将发送所有的 rewrite 相关的日志信息到 error_log 文件中，使用 [notice] 级别。随后就可以在error_log 查看rewrite信息了。

3. 使用location记录指定URL的日志

   ```
   server {
           error_log    /var/logs/nginx/example.com.error.log;
           location /static/ { 
           error_log /var/logs/nginx/static-error.log debug; 
       }         
   }
   ```

   配置以上配置后，/static/ 相关的日志会被单独记录在static-error.log文件中。

   nginx日志共三个参数
   access_log: 定义日志的路径及格式。
   log_format: 定义日志的模板。
   open_log_file_cache: 定义日志文件缓存。

   proxy_set_header X-Forwarded-For ：如果后端Web服务器上的程序需要获取用户IP，从该Header头获取。`proxy_set_header X-Forwarded-For $remote_addr;`

**常用例子**

1. main格式

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                       '$upstream_addr $upstream_response_time $request_time ';
access_log  logs/access.log  main;
```

2. json格式

```
log_format logstash_json '{"@timestamp":"$time_iso8601",'
       '"host": "$server_addr",'
       '"client": "$remote_addr",'
       '"size": $body_bytes_sent,'
       '"responsetime": $request_time,'
       '"domain": "$host",'
       '"url":"$request_uri",'
       '"referer": "$http_referer",'
       '"agent": "$http_user_agent",'
       '"status":"$status",'
       '"x_forwarded_for":"$http_x_forwarded_for"}';
```

解释：
`$uri`请求中的当前URI(不带请求参数，参数位于`$args`)，不同于浏览器传递的`$request_uri`的值，它可以通过内部重定向，或者使用index指令进行修改。不包括协议和主机名，例如/foo/bar.html。
`$request_uri` 这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看`$uri`更改或重写URI。
也就是说：`$request_uri`是原始请求URL，`$uri`则是经过nginx处理请求后剔除参数的URL,所以会将汉字表现为union。
坑点：
使用`$uri` 可以在nginx对URL进行更改或重写，但是用于日志输出可以使用`$request_uri`代替，如无特殊业务需求，完全可以替换。

3. 压缩格式

日志中增加了压缩的信息。

```
http {
    log_format compression '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';

    server {
        gzip on;
        access_log /spool/logs/nginx-access.log compression;
        ...
    }
}
```

4. upstream格式

增加upstream消耗的时间。

```
http {
    log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                             '"$request" $status $body_bytes_sent '
                             '"$http_referer" "$http_user_agent"'
                             'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';

    server {
        access_log /spool/logs/nginx-access.log upstream_time;
        ...
    }
}
```

5. 统计status 出现的次数

   ```
   awk '{print $9}' access.log | sort | uniq -c | sort -rn
   ```

6. 显示返回302状态码的URL。

   ```
   awk '($9 ~ /302/)' access.log | awk '{print $7}' | sort | uniq -c | sort -rn
   ```

#### 日志分析工具

##### [ngxtop](https://github.com/lebinh/ngxtop)

ngxtop 不仅可以对以前的日志进行排序、整理还可以实时监控 Nginx 日志。

ngxtop 是 Python 编写的，所以安装需要 Python2 或者 Python3 的环境，大家需要自行先配置好 Python 的环境。使用 pip 来安装 ngxtop ：

```
yum -y install epel-release
yum install python-pip -y
#或
apt-get install python-pip


pip install --upgrade pip
#然后
pip install ngxtop

```

参数：

```
print|top|avg|sum
ngxtop info 显示日志格式信息
-l <file>或--access-log <file> 设置日志路径
-f <format>或--log-format <format> 设置日志格式，默认格式combined，另外一种较常用格式为common
--no-follow 处理以前的日志，实时日志不做处理
-t <seconds> 或 --interval <seconds> 刷新频率，默认2秒
-g <var>或 --group-by <var> 按变量分组，默认显示 request_path
-w <var>或 --having <expr> 筛选 [default: 1]
-o <var>或 --order-by <var> 输出的排序方式，默认: 访问数
-n <number>或 --limit <number> 显示top多条，默认前top 10条
-a <exp> ...或 --a <exp> ... 对输出字段做处理，可选 sum, avg, min, max
-v或 --verbose 详细输出
-d或 --debug debug模式，输出每行及记录
-h或 --help 显示帮助详细
--version 显示版本信息
-c <file>或 --config <file> 指定nginx配置文件，自动分析日志格式
-i <filter-expression>或 --filter <filter-expression> 满足表达式的过滤将被处理
-p <filter-expression>或 --pre-filter <filter-expression> in-filter expression to check in pre-parsing phase.

```

另外一些变量可以在分析时用到，名字含义同日志格式里的设置：

```
remote_addr、remote_user、time_local、request、request_path、status、body_bytes_sent、http_referer、http_user_agent
```

![image.png](https://i.loli.net/2019/10/14/n3WOgUP8dwSC1Zp.png)



使用实例：

1. 实时监控日志

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log
   ```

   虽然直接执行 ngxtop 会自动搜索 nginx.conf ，但是直接解析里面默认虚拟主机的，建议直接指定日志文件。可以指定上 -n限定条数，也可以指定上 -g http_user_agent 按 useragent 查看。

2. 日志分析

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log --no-follow
   ```

   可以加一下参数进行详细分析，下面几个例子

   按rquest_path且是404的前10请求：

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log --no-follow top request_path --filter 'status == 404'
   ```

   按总bytes sent最高的前10：

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log --no-follow --order-by 'avg(bytes_sent) * count'
   ```

   按remote address进行排序前10：

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log --no-follow --group-by remote_addr
   ```

   显示400或更高返回状态码的且只显示request、status、http_referer这三列信息：

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech --no-follow -i 'status >= 400' print request status http_referer
   ```

   显示bytes_sent平均值且状态码为200且request_path以vpser开始的前10：

   ```
   ngxtop -l /home/wwwlogs/www.imydl.tech.log --no-follow avg bytes_sent --filter 'status == 200 and request_path.startswith("vpser")'
   ```

大家可以组合前面的命令进行日志的实时监控和日志排查整理，相信ngxtop会给大家带来一些管理上的方便。

##### [GoAccess](https://www.goaccess.cc)

需求：及时得到线上用户nginx访问日志分析统计结果！

![image.png](https://i.loli.net/2019/10/14/OMJwGRABdbnjNrz.png)

**具体安装：**

1. 编译安装

```shell
$ yum install glib2 glib2-devel GeoIP-devel  ncurses-devel zlib zlib-devel -y
$ wget http://tar.goaccess.io/goaccess-1.2.tar.gz
$ tar -xzvf goaccess-1.2.tar.gz
$ cd goaccess-1.2/
$ ./configure --prefix=/opt/goaccess --enable-utf8 --enable-geoip=legacy
$ make && make install
$ ln -s /opt/goaccess/bin/goaccess /usr/bin/goaccess
```

2. 在各主流 Linux 发行版上安装

   ```shell
   #Debian/Ubuntu
   
   sudo apt-get install goaccess
   #或 官方 GoAccess Debian/Ubuntu 仓库
   $ echo "deb http://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
   $ wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
   $ sudo apt-get update
   $ sudo apt-get install goaccess
   
   #如需支持 on-disk(Trusty+ or Wheezy+), 请运行：sudo apt-get install goaccess-tcb。
   #官方库已经支持通过 https 获取 .deb 格式包文件，您可能需要安装 apt-transport-https。
   
   #Fedora/CentOS
   $ sudo yum install goaccess
   
   ```

   更多参考[官网](https://www.goaccess.cc/?mod=download)

3. docker安装

   在 Docker 容器中运行 GoAccess 之前，请先在 /srv/goaccess/data 目录下创建配置文件。 您可以自行从头开始或者使用 [config/goaccess.conf](https://raw.githubusercontent.com/allinurl/goaccess/master/config/goaccess.conf) 作为起点并根据需要进行修改。

   一份最小化的支持实时 HTML 报告的适用于 Docker 容器的配置文件至少需要设置以下这些选项：`log-format`, `log-file`, `output`, `real-time-html` 以及 `ws-url`。

   配置文件准备好以后，请从 Github 上克隆源码仓库到本地：

   ```
   $ git clone https://github.com/allinurl/goaccess.git goaccess && cd $_
   ```

   接着请按照如下步骤创建并运行镜像：

   ```
   docker build . -t allinurl/goaccess
   docker run --restart=always -d -p 7890:7890 \
     -v "/srv/goaccess/data:/srv/data"         \
     -v "/srv/goaccess/html:/srv/report"       \
     -v "/var/log/apache2:/srv/logs"           \
     --name=goaccess allinurl/goaccess
   ```

   **注意:** 可能您需要替换 `/var/log/apache2` 为您自己的 Web 服务器的访问日志。

   如果一切顺利，一份安装报告将会出现在 `/srv/goaccess/html/` 目录下。

   如果在构建镜像之后修改了配置文件，是不需要重新构建的。简单的重启容器即可：

   ```
   docker restart goaccess
   ```

**用法：**

参考[官方文档](https://www.goaccess.cc/?mod=man)

使用nginx2goaccess.sh脚本将nginx日志格式格式化为goaccess能识别的日志格式，

```bash
#!/bin/bash
#
# Convert from this:
#   http://nginx.org/en/docs/http/ngx_http_log_module.html
# To this:
#   https://goaccess.io/man
#
# Conversion table:
#   $time_local         %d:%t %^
#   $host               %v
#   $http_host          %v
#   $remote_addr        %h
#   $request_time       %T
#   $request_method     %m
#   $request_uri        %U
#   $server_protocol    %H
#   $request            %r
#   $status             %s
#   $body_bytes_sent    %b
#   $bytes_sent         %b
#   $http_referer       %R
#   $http_user_agent    %u
#
# Samples:
#
# log_format combined '$remote_addr - $remote_user [$time_local] '
# '"$request" $status $body_bytes_sent '
# '"$http_referer" "$http_user_agent"';
#   ./nginx2goaccess.sh '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"'
#
# log_format compression '$remote_addr - $remote_user [$time_local] '
# '"$request" $status $bytes_sent '
# '"$http_referer" "$http_user_agent" "$gzip_ratio"';
#   ./nginx2goaccess.sh '$remote_addr - $remote_user [$time_local] "$request" $status $bytes_sent "$http_referer" "$http_user_agent" "$gzip_ratio"'
#
# log_format main
# '$remote_addr\t$time_local\t$host\t$request\t$http_referer\t$http_x_mobile_group\t'
# 'Local:\t$status\t$body_bytes_sent\t$request_time\t'
# 'Proxy:\t$upstream_cache_status\t$upstream_status\t$upstream_response_length\t$upstream_response_time\t'
# 'Agent:\t$http_user_agent\t'
# 'Fwd:\t$http_x_forwarded_for';
#   ./nginx2goaccess.sh '$remote_addr\t$time_local\t$host\t$request\t$http_referer\t$http_x_mobile_group\tLocal:\t$status\t$body_bytes_sent\t$request_time\tProxy:\t$upstream_cache_status\t$upstream_status\t$upstream_response_length\t$upstream_response_time\tAgent:\t$http_user_agent\tFwd:\t$http_x_forwarded_for'
#
# log_format main
# '${time_local}\t${remote_addr}\t${host}\t${request_method}\t${request_uri}\t${server_protocol}\t'
# '${http_referer}\t${http_x_mobile_group}\t'
# 'Local:\t${status}\t*${connection}\t${body_bytes_sent}\t${request_time}\t'
# 'Proxy:\t${upstream_status}\t${upstream_cache_status}\t'
# '${upstream_response_length}\t${upstream_response_time}\t${uri}${log_args}\t'
# 'Agent:\t${http_user_agent}\t'
# 'Fwd:\t${http_x_forwarded_for}';
#   ./nginx2goaccess.sh '${time_local}\t${remote_addr}\t${host}\t${request_method}\t${request_uri}\t${server_protocol}\t${http_referer}\t${http_x_mobile_group}\tLocal:\t${status}\t*${connection}\t${body_bytes_sent}\t${request_time}\tProxy:\t${upstream_status}\t${upstream_cache_status}\t${upstream_response_length}\t${upstream_response_time}\t${uri}${log_args}\tAgent:\t${http_user_agent}\tFwd:\t${http_x_forwarded_for}'
#
# Author: Rogério Carvalho Schneider <stockrt@gmail.com>

# Params
log_format="$1"

# Usage
if [[ -z "$log_format" ]]; then
    echo "Usage: $0 '<log_format>'"
    exit 1
fi

# Variables map
conversion_table="time_local,%d:%t_%^
host,%v
http_host,%v
remote_addr,%h
request_time,%T
request_method,%m
request_uri,%U
server_protocol,%H
request,%r
status,%s
body_bytes_sent,%b
bytes_sent,%b
http_referer,%R
http_user_agent,%u"

# Conversion
for item in $conversion_table; do
    nginx_var=${item%%,*}
    goaccess_var=${item##*,}
    goaccess_var=${goaccess_var//_/ }
    log_format=${log_format//\$\{$nginx_var\}/$goaccess_var}
    log_format=${log_format//\$$nginx_var/$goaccess_var}
done
log_format=$(echo "$log_format" | sed 's/${[a-z_]*}/%^/g')
log_format=$(echo "$log_format" | sed 's/$[a-z_]*/%^/g')

# Config output
echo "
- Generated goaccess config:
time-format %T
date-format %d/%b/%Y
log_format $log_format
"

# EOF
```

使用如下方式获取日志格式：

```
sh nginx2goaccess.sh '<log_format>'　　　　　　　　　#log_format为你nginx.conf中配置的日志格式
　　　
　#如：
sh nginx2goaccess.sh '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"'
```

会得到的结果，在goaccess-1.3/config下面创建一个nginxlog.conf：

```
#- Generated goaccess config:
time-format %T
date-format %d/%b/%Y
log_format %h - %^ [%d:%t %^] "%r" %s %b "%R" "%u" "%^"

```

配置见证奇迹的时刻，生成统计页面

```
goaccess -f /var/log/nginx/access.log -p /opt/goaccess/etc/nginxlog.conf -o /usr/share/nginx/html/report.html
#或生成中文的方法
LANG="zh_CN.UTF-8" bash -c "goaccess -f /var/log/nginx/access.log -p /opt/goaccess/etc/nginxlog.conf -o /usr/share/nginx/html/report.html"



#html可视化文件的实时更新方法
nohup goaccess -f /var/log/nginx/access.log -p /opt/goaccess/etc/nginxlog.conf -o /usr/share/nginx/html/report.html --real-time-html --ws-url=report.xxx.com &
```

GoAccess还为实时过滤和解析提供了极大的灵活性。例如，自goaccess启动以来通过监视日志来快速诊断问题：

```
tail -f access.log | goaccess - 
```

```
日志格式符号解释
%x 与时间格式和日期格式变量匹配的日期和时间字段。当给出时间戳而不是日期和时间在两个单独的变量中时使用。
%t 时间字段匹配时间格式变量。
%d 与日期格式变量匹配的日期字段。
%v 服务器名称根据规范名称设置（服务器块或虚拟主机）。
%e 这是HTTP身份验证确定的请求文档的人的用户标识。
%h host（客户端IP地址，IPv4或IPv6）
%r 来自客户端的请求行。这需要围绕请求的特定分隔符（单引号，双引号等）可解析。否则，使用特殊的格式说明符，如组合%m，%U，%q和%H解析各个字段。
注意：使用或者%r获得完整的请求OR %m，%U，%q并%H形成你的要求，不要同时使用。
%m 请求方法。
%U 请求的URL路径。
注意：如果查询字符串在%U，则无需使用%q。但是，如果URL路径不包含任何查询字符串，则可以使用%q并将查询字符串附加到请求中。
%q 查询字符串。
%H 请求协议。
%s 服务器发送回客户端的状态代码。
%b 返回给客户端的对象大小。
%R “Referer”HTTP请求标头。
%u 用户代理HTTP请求标头。
%D 服务请求所需的时间，以微秒为单位。
%T 服务请求所需的时间，以毫秒为单位，分辨率为毫秒。
%L 服务请求所用的时间，以毫秒为单位的十进制数。
%^ 忽略此字段。
%~向前移动日志字符串，直到找到非空格（！isspace）char。
~h X-Forwarded-For（XFF）字段中的主机（客户端IP地址，IPv4或IPv6）。

选项解释
　　　-f 指定nginx日志文件
　　　-p 指定日志格式文件
　　　-o 输出到指定html文件
　　　--real-time-html 实时刷新
　　　--ws-url 绑定一个域名
```

配置nginx访问刚刚生成的report.html页面

```
$ vim /etc/nginx/conf.d/goaccess.conf
server{
	listen 8080;
	server_name localhost;

	location /report.html {
            alias /usr/share/nginx/html/report.html
        } 
}

```

重启nginx

在浏览器就可以访问 http://127.0.0.1:8080/report.html

#### Nginx解决跨域

很多人都会遇到 Nginx 跨域的问题, 而我遇到的问题是：

- 客户端在 [www.a.com](http://www.a.com/)
- 服务端在 [www.b.com](http://www.b.com/)
- Nginx 在 [www.c.com](http://www.c.com/)

此时需要对 Nginx 进行跨域配置才可以访问 `www.c.com` 的获取客户端 `www.a.com` 来请求 `www.b.com` 的数据, Nginx 配置如下(重要部分)：

```nginx
location / {
    add_header 'Access-Control-Allow-Origin' $http_origin;
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
    if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
    }
    root   html;
    proxy_pass http://xxx:8000/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_connect_timeout 5;
}
```

别忘了把配置中 `proxy_pass` 对应的 `http://xxx:8000/` 地址换成你的服务地址, 当然了, 你的客户端请求的地址不可以是你的服务地址, 而是 Nginx 的地址, 这样就可以达到解决跨域的问题。

#### nginx不转发http header问题解决

1. 默认的情况下nginx引用header变量时不能使用带下划线的变量。要解决这样的问题只能单独配置[underscores_in_headers](http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers) on；
2. 默认的情况下会忽略掉带下划线的变量。要解决这个需要配置[ignore_invalid_headers](http://nginx.org/en/docs/http/ngx_http_core_module.html#ignore_invalid_headers) off。
3. 当然最好的方法就是不使用带下划线的header变量

#### Https配置

有关 SSL 的介绍可以参阅维基百科的[传输层安全协议](https://zh.wikipedia.org/wiki/傳輸層安全協議)和阮一峰先生的 [《SSL/TLS协议运行机制的概述》](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)。

SSL 证书主要有两个功能：**加密**和**身份证明**，通常需要购买，也有免费的，通过第三方 SSL 证书机构颁发，常见可靠的第三方 SSL 证书颁发机构有下面几个：

![image.png](https://i.loli.net/2019/10/15/oZiVID2xc13az9s.png)

[StartCom](https://www.startssl.com/) 机构上的 SSL 证书有以下几种：

- 企业级别：EV(Extended Validation)、OV(Organization Validation)
- 个人级别：IV(Identity Validation)、DV（Domain Validation）

其中 EV、OV、IV 需要付费

免费的证书安全认证级别一般比较低，不显示单位名称，不能证明网站的真实身份，仅起到加密传输信息的作用，适合个人网站或非电商网站。由于此类只验证域名所有权的低端 SSL 证书已经被国外各种欺诈网站滥用，因此强烈推荐部署验证单位信息并显示单位名称的 OV SSL 证书或申请最高信任级别的、显示绿色地址栏、直接在地址栏显示单位名称的 EV SSL 证书，就好像 [StarCom](https://www.startssl.com/) 的地址栏一样

更多关于购买 SSL 证书的介绍：[SSL 证书服务，大家用哪家的？](https://www.zhihu.com/question/19578422)、[DV免费SSL证书](http://freessl.wosign.com/freessl)

**使用 OpenSSL 生成 SSL Key 和 CSR 文件**

配置 HTTPS 要用到私钥 example.key 文件和 example.crt 证书文件，申请证书文件的时候要用到 example.csr 文件，`OpenSSL` 命令可以生成 example.key 文件和 example.csr 证书文件。

- CSR：Cerificate Signing Request，证书签署请求文件，里面包含申请者的 DN（Distinguished Name，标识名）和公钥信息，**在第三方证书颁发机构签署证书的时候需要提供**。证书颁发机构拿到 CSR 后使用其根证书私钥对证书进行加密并生成 CRT 证书文件，里面包含证书加密信息以及申请者的 DN 及公钥信息
- Key：证书申请者私钥文件，和证书里面的公钥配对使用，在 HTTPS 『握手』通讯过程需要使用私钥去解密客戶端发來的经过证书公钥加密的随机数信息，是 HTTPS 加密通讯过程非常重要的文件，**在配置 HTTPS 的時候要用到**

使用 `OpenSSl`命令可以在系统当前目录生成 **example.key** 和 **example.csr** 文件：

```bash
openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=ShenZhen/L=ShenZhen/O=Example Inc./OU=Web Security/CN=example.com"
#创建CA证书
openssl req -new -x509 -key example_com.key -out ca.crt -days 3650  -subj "/C=CN/ST=ShenZhen/L=ShenZhen/O=Example Inc./OU=Web Security/CN=example.com"
#创建自当前日期起有效期为期十年的服务器证书server.crt
openssl x509 -req -days 3650 -in example_com.csr -CA ca.crt -CAkey example_com.key -CAcreateserial -out example_com.crt
```

下面是上述第一条命令相关字段含义：

- C：Country ，单位所在国家，为两位数的国家缩写，如： CN 就是中国
- ST 字段： State/Province ，单位所在州或省
- L 字段： Locality ，单位所在城市 / 或县区
- O 字段： Organization ，此网站的单位名称;
- OU 字段： Organization Unit，下属部门名称;也常常用于显示其他证书相关信息，如证书类型，证书产品名称或身份验证类型或验证内容等;
- CN 字段： Common Name ，网站的域名;

生成 csr 文件后，提供给 CA 机构，签署成功后，就会得到一個 **example.crt** 证书文件，SSL 证书文件获得后，就可以在 Nginx 配置文件里配置 HTTPS 了。

```bash
$ ls
ca.crt ca.srl example_com.crt example_com.csr example_com.key
```

其中,`example_com.crt`和`example_com.key`就是你的nginx需要的证书文件.

**配置 HTTPS**

要开启 HTTPS 服务，在配置文件信息块(server block)，必须使用监听命令 `listen` 的 ssl 参数和定义服务器证书文件和私钥文件，如下所示：

```nginx
server {
    #ssl参数
    listen              443 ssl;
    server_name         example.com;
    #证书文件
    ssl_certificate     /opt/server/nginx/conf/example_com.crt;#配置证书位置
    #私钥文件
    ssl_certificate_key /opt/server/nginx/conf/example_com.key;#配置秘钥位置
    #ssl_client_certificate ca.crt;#双向认证
    #ssl_verify_client on; #双向认证
    
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    #...
    ssl_session_timeout  5m;
    ssl_prefer_server_ciphers   on;
}
```

证书文件会作为公用实体發送到每台连接到服务器的客戶端，私钥文件作为安全实体，**应该被存放在具有一定权限限制的目录文件，并保证 Nginx 主进程有存取权限**。

私钥文件也有可能会和证书文件同放在一個文件中，如下面情況：

```nginx
ssl_certificate     /opt/server/nginx/conf/example_com.cert;
ssl_certificate_key /opt/server/nginx/conf/example_com.cert;
```

这种情況下，证书文件的的读取权限也应该加以限制，仅管证书和私钥存放在同一个文件里，但是只有证书会被发送到客戶端

命令 `ssl_protocols` 和 `ssl_ciphers` 可以用来限制连接只包含 SSL/TLS 的加強版本和算法，默认值如下：

```nginx
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers HIGH:!aNULL:!MD5;
```

由于这两个命令的默认值已经好几次发生了改变，因此不建议显性定义，除非有需要额外定义的值，如定义 D-H 算法：

```nginx
#使用DH文件
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
#定义算法
ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
#...
```

**优化**

```nginx
worker_processes auto;

http {

    #配置共享会话缓存大小，视站点访问情况设定
    ssl_session_cache   shared:SSL:10m;
    #配置会话超时时间
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         example.com;
        
        #设置长连接
        keepalive_timeout   70;
        
        #HSTS策略
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
        
        #证书文件
        ssl_certificate     /opt/server/nginx/conf/example_com.crt;
        #私钥文件
        ssl_certificate_key /opt/server/nginx/conf/example_com.key; 
        
        #优先采取服务器算法
        ssl_prefer_server_ciphers on;
        #使用DH文件
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        #定义算法
        ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
        #减少点击劫持
        add_header X-Frame-Options DENY;
        #禁止服务器自动解析资源类型
        add_header X-Content-Type-Options nosniff;
        #防XSS攻擊
        add_header X-Xss-Protection 1;
        #...
```

**HTTP/HTTPS混合服务器配置**

```nginx
server {
    listen              80;
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     /opt/server/nginx/conf/example_com.crt;
    ssl_certificate_key /opt/server/nginx/conf/example_com.key;
    #...
}
```