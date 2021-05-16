---
title: nginx 常用http模块
date: 2019-08-29 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

#### 配置文件

![UTOOLS1572246394408.png](https://i.loli.net/2019/10/28/Av5Wu1XJhqZFYxV.png)

1. 全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

2. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

4. server块：配置虚拟主机的相关参数，一个http中可以有多个server。

5. location块：配置请求的路由，以及各种页面的处理情况。

```nginx
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

#### 指令合并

![UTOOLS1572246508055.png](https://i.loli.net/2019/10/28/ouSwvZxkzag4PAG.png)

![UTOOLS1572246630104.png](https://i.loli.net/2019/10/28/K5dzC6BQIkOnYRU.png)

![UTOOLS1572246889782.png](https://i.loli.net/2019/10/28/e3xXzCT9JIjhiny.png)

#### [listen指令](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)

为IP 设置address和，port对于path服务器将在其上接受UNIX域套接字的设置。无论address和port，或只address或仅port可指定。An address也可以是主机名，例如：

```
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
```

IPv6 addresses (0.7.36) are specified in square brackets:

```
listen [::]:8000;
listen [::1];
```

UNIX域套接字（0.8.21）使用“ `unix:`”前缀指定：

```
listen unix:/var/run/nginx.sock;
```

如果仅address给出，则使用端口80。

如果该指令不存在，则`*:80`如果nginx以超级用户特权运行，则使用该指令，`*:8000` 否则使用。

#### 处理http头部的模块

![UTOOLS1572247635561.png](https://i.loli.net/2019/10/28/4vSlwrdPscNe7VE.png)

接受请求HTTP模块

![UTOOLS1572247869398.png](https://i.loli.net/2019/10/28/dIqoctKuPMUDVT6.png)

#### nginx正则表达式

在`Nginx`中`location`, `server_name`,`rewrite`等模块使用了大量的正则表达式，通过正则表达式可以完整非常强悍的功能，但是这部分对我们阅读源码也产生了非常大的困惑。

`Nginx`中的正则表达式使用了`pcre`格式，并且封装了[`pcre函数库`](http://regexkit.sourceforge.net/Documentation/pcre/pcreapi.html#SEC1)的几个常用函数

<https://www.cnblogs.com/IPYQ/p/7889399.html>

#### server_name指令

![UTOOLS1572249073627.png](https://i.loli.net/2019/10/28/tD8HAplgRL4OKPZ.png)

![UTOOLS1572249266626.png](https://i.loli.net/2019/10/28/Wy2UA18DHoq3Vd9.png)

**匹配顺序**

![image-20191028155701190](/home/hyp/.config/Typora/typora-user-images/image-20191028155701190.png)

#### http请求处理时的11个阶段

```c
typedef enum {
    // 在接收到完整的HTTP头部后处理的HTTP阶段
    NGX_HTTP_POST_READ_PHASE = 0,

    /*在将请求的URI与location表达式匹配前，修改请求的URI（所谓的重定向）是一个独立的HTTP阶段*/
    NGX_HTTP_SERVER_REWRITE_PHASE,

    /*根据请求的URI寻找匹配的location表达式，这个阶段只能由ngx_http_core_module模块实现，不建议其他HTTP模块重新定义这一阶段的行为*/
    NGX_HTTP_FIND_CONFIG_PHASE,

    /*在NGX_HTTP_FIND_CONFIG_PHASE阶段寻找到匹配的location之后再修改请求的URI*/
    NGX_HTTP_REWRITE_PHASE,

    /*这一阶段是用于在rewrite重写URL后，防止错误的nginx.conf配置导致死循环（递归地修改URI），因此，这一阶段仅由ngx_http_core_module模块处理。目前，控制死循环的方式很简单，首先检查rewrite的次数，如果一个请求超过10次重定向,就认为进入了rewrite死循环，这时在NGX_HTTP_POST_REWRITE_PHASE阶段就会向用户返回500，表示服务器内部错误*/
    NGX_HTTP_POST_REWRITE_PHASE,

    /*表示在处理NGX_HTTP_ACCESS_PHASE阶段决定请求的访问权限前，HTTP模块可以介入的处理阶段*/
    NGX_HTTP_PREACCESS_PHASE,

    // 这个阶段用于让HTTP模块判断是否允许这个请求访问Nginx服务器
    NGX_HTTP_ACCESS_PHASE,

    /*在NGX_HTTP_ACCESS_PHASE阶段中，当HTTP模块的handler处理函数返回不允许访问的错误码时（实际就是NGX_HTTP_FORBIDDEN或者NGX_HTTP_UNAUTHORIZED），这里将负责向用户发送拒绝服务的错误响应。因此，这个阶段实际上用于给NGX_HTTP_ACCESS_PHASE阶段收尾*/
    NGX_HTTP_POST_ACCESS_PHASE,

    /*这个阶段完全是为try_files配置项而设立的，当HTTP请求访问静态文件资源时，try_files配置项可以使这个请求顺序地访问多个静态文件资源，如果某一次访问失败，则继续访问try_files中指定的下一个静态资源。这个功能完全是在NGX_HTTP_TRY_FILES_PHASE阶段中实现的*/
    NGX_HTTP_TRY_FILES_PHASE,

    // 用于处理HTTP请求内容的阶段，这是大部分HTTP模块最愿意介入的阶段
    NGX_HTTP_CONTENT_PHASE,

    /*处理完请求后记录日志的阶段。例如，ngx_http_log_module模块就在这个阶段中加入了一个handler处理方法，使得每个HTTP请求处理完毕后会记录access_log访问日志*/
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```

以上阶段中，有些阶段是必备的，有些阶段是可选的，各个阶段可以允许多个HTTP模块同时介入，nginx会按照各个HTTP模块的ctx_index顺序执行这些模块的handler方法。

但是，ngx_http_find_config_phase,nginx_http_post_rewrite_phase,nginx_http_post_access_phase,ngx_http_try_files_phase这四个阶段是不允许HTT模块加入自己的ngx_http_handler_py方法处理用户请求的，他们仅由HTTP框架自身实现。

**处理流程**

![UTOOLS1572250968039.png](https://i.loli.net/2019/10/28/Rdhgq8mGFuXj76B.png)

| 序号 | 阶段           | 指令                             | 备注                     |
| ---- | -------------- | -------------------------------- | ------------------------ |
| 1    | POST_READ      | realip                           | 获取客户端真实IP         |
| 2    | SERVER_REWRITE | rewrite                          |                          |
| 3    | FIND_CONFIG    |                                  |                          |
| 4    | REWRITE        | rewrite                          |                          |
| 5    | POST_REWRITE   |                                  |                          |
| 6    | PRE_ACCESS     | limit_conn, limit_req            |                          |
| 7    | ACCESS         | auth_basic, access, auth_request | auth_basic可以做访问限制 |
| 8    | POST_ACCESS    |                                  |                          |
| 9    | PRE_CONTENT    | try_files                        |                          |
| 10   | CONTENT        | index, autoindex, concat         |                          |
| 11   | LOG            | access_log                       | access_log记录请求日志   |

#### [realip](http://nginx.org/en/docs/http/ngx_http_realip_module.html)模块

realip拿到用户真正IP地址，为后续操作做准备

![UTOOLS1572251584457.png](https://i.loli.net/2019/10/28/UKSoP82VeMzsRrC.png)

![UTOOLS1572252197442.png](https://i.loli.net/2019/10/28/GbNlIReV9rchg3J.png)

**做反向代理时配置**

```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Real-Port $remote_port;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

三个header分别表示：

```
X-Real-IP            客户端ip
X-Real-Port          客户端或上一级端口
X-Forwarded-For      包含了客户端和各级代理ip的完整ip链路
```

其中X-Real-IP是必需的，后两项选填。当只存在一级nginx代理的时候X-Real-IP和X-Forwarded-For是一致的，而当存在多级代理的时候，X-Forwarded-For 就变成了如下形式

```
X-Forwarded-For: 客户端ip， 一级代理ip， 二级代理ip...
```

在获取客户端ip的过程中虽然X-Forwarded-For是选填的，但是个人建议还是保留这，以便出现安全问题的时候，可以根据日志文件回溯来源。

#### [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)模块

rewrite模块即`ngx_http_rewrite_module`模块，主要功能是改写请求URI，是Nginx默认安装的模块。rewrite模块会根据PCRE正则匹配重写URI，然后发起内部跳转再匹配location，或者直接做30x重定向返回客户端。

rewrite模块的指令有break, if, return, rewrite, set等。rewrite指令所执行的顺序如下：

- 首先在server上下文中依照顺序执行rewrite模块指令；
- 如果server中行了rewrite重写，那么以新URI发起内部跳转，直接匹配location，不会再执行server里的rewrite指令，然后
  - 新URI直接匹配location
  - 如果匹配上某个location，那么其中的rewrite模块指令同样依照顺序执行
  - 如果再次导致URI的rewrite，那么再一次进行内部跳转去匹配location，但跳转的总次数不能超过10次



![UTOOLS1572252625331.png](https://i.loli.net/2019/10/28/gNfmzTFG41762rX.png)



示例：

```nginx
//如果UA包含"MSIE"，rewrite请求到/msid/目录下
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}
 
//如果cookie匹配正则，设置变量$id等于正则引用部分
if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}
 
//给某个访问IP返回403
if ( $remote_addr = "202\.38\.78\.85" ){
    return 403;
}
 
//如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
if ($request_method = POST) {
    return 405;
}
 
//如果$slow可以通过set指令设置，则进行限速处理
if ($slow) {
    limit_rate 10k;
}
 
//如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查
if (!-f $request_filename){
    break;
    proxy_pass http://127.0.0.1;
}
 
//如果query string中包含"post=140"，则永久重定向到example.com
if ($args ~ post=140){
    rewrite ^ http://example.com/ permanent;
}
 
//防盗链
location ~* \.(gif|jpg|png|swf|flv)$ {
    valid_referers none blocked www.baidu.com www.ywnds.com;
    if ($invalid_referer) {
        return 404;
    }
}
```

return和error_page的区别：

![UTOOLS1572252740529.png](https://i.loli.net/2019/10/28/vBKEdZ596DXeLUg.png)

**rewrite指令**

> 基本语法： rewrite regex replacement [flag];
> 上下文：server, location, if



![UTOOLS1572253369792.png](https://i.loli.net/2019/10/28/DGjsLgzvAEHeWrT.png)

在server中使用的情况：

```
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403; #没有匹配上，那就返回403咯
    ...
}
```

注意，在server中使用rewrite ，我们使用的flag是last，但是在location中，我们却只能用break：

```
location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}
```

如果在location的rewrite也使用last，便会再次以新的URI重新发起内部重定向，再次进行location匹配，而新的URI中极有可能和旧的URI一样再次匹配到相同location中，这样死循环发生了。当循环到第10次时，Nginx会终止这样无意义的循环，并返回500错误。这点需要特别的注意。

**break指令**

> 基本语法：break;
> 上下文：server, location, if

停止处理任何rewrite的相关指令。如果出现在location里面，那么所有后面的rewrite模块指令都不会再执行，也不发起内部重定向，而是直接用新的URI进一步处理请求。

**if指令**

> 基本语法：if (condition) { ... }
> 上下文：server, location

根据条件condition的真假决定是否加载{...}中的配置，{...}中的配置可以继承外面的配置，也可以对外面已有配置指令进行覆写。

![UTOOLS1572254191299.png](https://i.loli.net/2019/10/28/P8mHSBAu31eitQT.png)

**set指令**

> 基本语法：set $variable value;
> 上下文：server, location, if

这是一个有用的指令，用来定义变量，变量的值可以包含字符串，另外的变量或者是二者结合。

```
set $var = $http_x_forwarded_for;
```

> 注意：在Nginx中，除非特殊说明，大部分地方字符串的不需要引号括住，字符串和变量的拼接也不需要引号

**rewrite_log指令**

> 基本语法：rewrite_log on | off;
> 上下文：http, server, location, if

如果开启 on，那么当发生rewrite时，会产生一个notice级别的日志；否则不会产生任何日志。默认情况下是不产生的，但在调试的时候可以将其置为on。

参考：<https://www.cnblogs.com/minirice/p/8872093.html>

#### find_config阶段

**location 指令**



![UTOOLS1572254489103.png](https://i.loli.net/2019/10/28/PJXmx8dW1KTtlQq.png)

![UTOOLS1572254535517.png](https://i.loli.net/2019/10/28/8voLyDCBZO7mpKw.png)

匹配规则

![UTOOLS1572254588446.png](https://i.loli.net/2019/10/28/nrI37wlGseMF5iJ.png)

#### [limit_conn](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)模块

![UTOOLS1572254879738.png](https://i.loli.net/2019/10/28/kAeln5uW7UD8xBm.png)

**指令**

![UTOOLS1572254947000.png](https://i.loli.net/2019/10/28/zmsvyVXIWcubCJR.png)

![UTOOLS1572255002397.png](https://i.loli.net/2019/10/28/tDTqmxlFIi7jV2G.png)

#### [limit_req](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)模块

![image-20191028173337906](/home/hyp/.config/Typora/typora-user-images/image-20191028173337906.png)

**leaky bucket算法**

![Kca6SS.png](https://s2.ax1x.com/2019/10/28/Kca6SS.png)

**相关指令**

![KcaOm9.png](https://s2.ax1x.com/2019/10/28/KcaOm9.png)

![Kcdmff.png](https://s2.ax1x.com/2019/10/28/Kcdmff.png)

![KcdK1S.png](https://s2.ax1x.com/2019/10/28/KcdK1S.png)

```nginx
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_req_zone $binary_remote_addr zone=one:10m rate=2r/m;

server {
	server_name limit.pingxin.com;
	root /usr/share/nginx/html/;
	error_log /var/log/nginx/myerror.log info;
	location / {
		limit_conn_status 500;
		limit_conn_log_level warn;
		#limit_req zone=one burst=3 nodelay;
		limit_rate 50;
		limit_conn addr 1;
		limit_req zone=one;
	}

}
```

#### [access](http://nginx.org/en/docs/http/ngx_http_access_module.html)模块



![KgNraq.png](https://s2.ax1x.com/2019/10/28/KgNraq.png)

**指令**

![KgNcGT.png](https://s2.ax1x.com/2019/10/28/KgNcGT.png)

#### [auth_basic](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)模块

![KgNhL9.png](https://s2.ax1x.com/2019/10/28/KgNhL9.png)

**指令**

![KgN7i6.png](https://s2.ax1x.com/2019/10/28/KgN7i6.png)

生成密码文件

![KgNzdI.png](https://s2.ax1x.com/2019/10/28/KgNzdI.png)

**示例**

```nginx
server {
	server_name access.pingxin.com;
	error_log error.log debug;
	default_type test/plain;
	location /{
		satisfy any;
		auth_basic "test auth_basic";
		auth_basic_user_file auth.pass;
		deny all;
	}
}

```

#### [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html)模块

![Kg22rt.png](https://s2.ax1x.com/2019/10/28/Kg22rt.png)

配置文件access.conf

```nginx
server {
	server_name access.pingxin.com;
	error_log error.log debug;
	default_type test/plain;
	location /auth_basic {
		satisfy any;
		auth_basic "test auth_basic";
		auth_basic_user_file auth.pass;
		deny all;
	}
	
	location /auth_request {
		auth_request /test_auth;
	}
	location = /test_auth {
		proxy_pass http://127.0.0.1:8090/auth_upstream;
		proxy_pass_request_body off;
		proxy_set_header Content-Length "";
		proxy_set_header X-Original_URI $request_uri;
	}
    
}

```

上游服务配置文件access_upstream.conf

```

server {
    listen 8090;
    location /auth_upstream {
        return 200 'auth sucsess!';
    }
}
```

#### access 阶段的Satisfy指令

![K5PJ3t.png](https://s2.ax1x.com/2019/10/30/K5PJ3t.png)



![K5PqKK.png](https://s2.ax1x.com/2019/10/30/K5PqKK.png)

1. access阶段不会生效，因为return在rewrite阶段，在access阶段之前。
2. 有影响
3. 可以的，Satisfy配置为any，虽然有deny all,但是会执行进一步认证，用户输入用户名密码后，还是有机会访问到文件
4. 可以的，模块内的配置无顺序要求
5. 没有机会，Satisfy配置为all，任一access模块同意就放行。

#### precontent阶段：[try_files](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)模块

![K5Fv9A.png](https://s2.ax1x.com/2019/10/30/K5Fv9A.png)

配置文件tryfiles.conf

```
server {
	server_name tryfiles.pingxin.com;
	root /usr/share/nginx/html;
	default_type text/plain;
	location /first {
		try_files /system/maintenance.html
				$uri $uri/index.html $uri.html
				@lasturl;
	}
	location @lasturl {
		return 200 'lasturl!\n'; 
	}
	location /second {
		try_files $uri $uri/index.html $uri.html =404;
	}
}
```

可以在反向代理中很有用，先在本地进行查找是否有此类文件，如果没有，再在上游服务中进行查找。

#### precontent阶段：[mirror](http://nginx.org/en/docs/http/ngx_http_mirror_module.html)模块

![K5AQMt.png](https://s2.ax1x.com/2019/10/30/K5AQMt.png)

上游服务配置文件mirror_upstream.conf

```
server {
    listen 10020;
    error_log upstream.log;
    location / {
        return 200 'auth sucsess!';
    }
}
```

实际处理配置文件mirror.conf

```
server {
	listen 8001;
	location / {
		 mirror /mirror;
		 mirror_request_body off;
	}
	location = /mirror {
		 internal;
		 proxy_pass http://127.0.0.1:10020$request_uri;
		 proxy_pass_request_body off;
		 proxy_set_header Content-length "";
		 proxy_set_header X-Original-URI $request_uri;
	}
}
```

#### static模块

![K5mYgH.png](https://s2.ax1x.com/2019/10/30/K5mYgH.png)

![K5m55T.png](https://s2.ax1x.com/2019/10/30/K5m55T.png)

配置文件static.conf

```
server {
	server_name static.pingxin.com;
	
	location /root {
		root html;
	}
	location /alias {
		alias html;
	}
	location ~ /root/(\w+\.txt) {
		root html/first/$1;
	}
	location ~ /root/(\w+\.txt) {
		alias html/first/$1;
	}
	location /RealPath {
		alias html/realpath/;
		return 200 '$request_filename;$document_root;$realpath_root\n';
	}
}
```

使用上述路径进行请求，观察error.log文件日志。`/usr/share/nginx/html/`为上层root路径

1. `/root`==>`/usr/share/nginx/html/root`
2. `/root/1.txt`==>`/usr/share/nginx/html/first/1.txt/root/1.txt`
3. `/alias/`==>`/usr/share/nginx/html/index.html`
4. `/alias/1.txt`==>`/usr/share/nginx/html/1.txt`

![K5Q8cF.png](https://s2.ax1x.com/2019/10/30/K5Q8cF.png)

```shell
 mkdir /usr/share/nginx/html/first
 echo "1.txt" >/usr/share/nginx/html/first/1.txt
 ln -s /usr/share/nginx/html/realpath /usr/share/nginx/html/first
```

然后访问`static.pingxin.com/RealPath/1.txt`

```shell
$ curl static.pingxin.com/RealPath/1.txt
/usr/share/nginx/html/realpath//1.txt;/usr/share/nginx/html/realpath/;/usr/share/nginx/html/first
```

![K5lTG6.png](https://s2.ax1x.com/2019/10/30/K5lTG6.png)

1. types 类型名和文件类型做映射，由一个hasp表进行存储，types_hash_bucket_size和types_hash_max_size进行控制
2. default_type没有文件名时，告诉Centent-type的值

![K51Sit.png](https://s2.ax1x.com/2019/10/30/K51Sit.png)

log_not_found置为off时，当找不到文件时不会产生错误日志。

![K510yD.png](https://s2.ax1x.com/2019/10/30/K510yD.png)

![K51ImQ.png](https://s2.ax1x.com/2019/10/30/K51ImQ.png)

1. absolute_redirect：on返回域名，off不返回
2. server_name_in_redirect：on使用server_name定义的域名;off使用host名
3. port_in_redirect:on显示端口；off不使用端口,默认为on

配置文件dirredirect.conf

```
server{
	server_name return.pingxin.com dir.pingxin.com;
	server_name_in_redirect off;
	listen 8088;
	port_in_redirect on;
	absolute_redirect off;
	root html/;
}
```

当absolute_redirect设为off时：

```
$ curl 127.0.0.1:8088/first -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.16.1
Date: Wed, 30 Oct 2019 15:05:47 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
Location: /first/
```

把absolute_redirect注释掉

```shell
$ curl 127.0.0.1:8088/first -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.16.1
Date: Wed, 30 Oct 2019 15:07:32 GMT
Content-Type: text/html
Content-Length: 169
Location: http://127.0.0.1:8088/first/
Connection: keep-alive
$ curl -H 'Host:aaaa' 127.0.0.1:8088/first -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.16.1
Date: Wed, 30 Oct 2019 15:08:34 GMT
Content-Type: text/html
Content-Length: 169
Location: http://aaaa:8088/first/
Connection: keep-alive
```

希望按照server_name来进行重定向，`server_name_in_redirect`置为on

```shell
$ curl -H 'Host:aaaa' 127.0.0.1:8088/first -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.16.1
Date: Wed, 30 Oct 2019 15:10:55 GMT
Content-Type: text/html
Content-Length: 169
Location: http://return.pingxin.com:8088/first/
Connection: keep-alive


```

#### [index](http://nginx.org/en/docs/http/ngx_http_index_module.html)和[auto_index](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)模块

auto_index显示一个目录下的目录结构，index先于auto_index执行

![K5GFsO.png](https://s2.ax1x.com/2019/10/30/K5GFsO.png)

![K5G8eg.png](https://s2.ax1x.com/2019/10/30/K5G8eg.png)

![K5GJoj.png](https://s2.ax1x.com/2019/10/30/K5GJoj.png)

配置文件：autoindex.conf

```nginx
server {
	server_name autoindex.pingxin.com;
	listen 8080;
	location / {
	alias html/;
	autoindex on;
	#index a.html;
	autoindex_exact_size off;
	autoindex_format json;
	autoindex_localtime on;
	}
}
```

此时访问`autoindex.pingxin.com:8080`返回的是首页，如果把`#index a.html;`去掉注释，就可看到返回值变成了json格式,而且是`/usr/share/nginx/html`目录下的文件信息。

```shell
$ curl autoindex.pingxin.com:8080
[
{ "name":"en-US", "type":"directory", "mtime":"Sun, 27 Oct 2019 13:15:43 GMT" },
{ "name":"first", "type":"directory", "mtime":"Wed, 30 Oct 2019 14:52:29 GMT" },
{ "name":"icons", "type":"directory", "mtime":"Mon, 28 Oct 2019 09:44:08 GMT" },
{ "name":"img", "type":"directory", "mtime":"Sun, 27 Oct 2019 13:15:43 GMT" },
{ "name":"realpath", "type":"directory", "mtime":"Wed, 30 Oct 2019 14:52:29 GMT" },
{ "name":"404.html", "type":"file", "mtime":"Thu, 03 Oct 2019 05:12:34 GMT", "size":3650 },
{ "name":"50x.html", "type":"file", "mtime":"Thu, 03 Oct 2019 05:12:34 GMT", "size":3693 },
{ "name":"index.html", "type":"file", "mtime":"Fri, 16 May 2014 15:12:48 GMT", "size":4833 },
{ "name":"nginx-logo.png", "type":"file", "mtime":"Thu, 03 Oct 2019 05:12:34 GMT", "size":368 },
{ "name":"poweredby.png", "type":"file", "mtime":"Thu, 03 Oct 2019 05:12:34 GMT", "size":368 }
]
```

将`autoindex_format`值改为`html`

```shell
$ curl autoindex.pingxin.com:8080
<html>
<head><title>Index of /</title></head>
<body>
<h1>Index of /</h1><hr><pre><a href="../">../</a>
<a href="en-US/">en-US/</a>                                             27-Oct-2019 21:15       -
<a href="first/">first/</a>                                             30-Oct-2019 22:52       -
<a href="icons/">icons/</a>                                             28-Oct-2019 17:44       -
<a href="img/">img/</a>                                               27-Oct-2019 21:15       -
<a href="realpath/">realpath/</a>                                          30-Oct-2019 22:52       -
<a href="404.html">404.html</a>                                           03-Oct-2019 13:12    3650
<a href="50x.html">50x.html</a>                                           03-Oct-2019 13:12    3693
<a href="index.html">index.html</a>                                         16-May-2014 23:12    4833
<a href="nginx-logo.png">nginx-logo.png</a>                                     03-Oct-2019 13:12     368
<a href="poweredby.png">poweredby.png</a>                                      03-Oct-2019 13:12     368
</pre><hr></body>
</html>

```

#### [concat模块](http://tengine.taobao.org/document/http_concat.html)

可以在一次请求中返回多个文件的内容

![MZLvyd.png](https://s2.ax1x.com/2019/11/08/MZLvyd.png)

**指令**

![MZO27t.png](https://s2.ax1x.com/2019/11/08/MZO27t.png)

concat.conf配置文件

```nginx
server{
	server_name concat.pingxin.com;
    concat on;
    root html;
    location /concat {
        concat_max_files 20;
        concat_types text/plain;
        concat_unique on;
        concat_delimiter ':::';
        concat_ignore_file_error on;
    }
}
```

#### [log模块](http://nginx.org/en/docs/http/ngx_http_log_module.html)

![MZzqYt.png](https://s2.ax1x.com/2019/11/08/MZzqYt.png)

![MZzLfP.png](https://s2.ax1x.com/2019/11/08/MZzLfP.png)

![MeSFf0.png](https://s2.ax1x.com/2019/11/08/MeSFf0.png)

![MeS81K.png](https://s2.ax1x.com/2019/11/08/MeS81K.png)

#### Http过滤模块

![MeS0ht.png](https://s2.ax1x.com/2019/11/08/MeS0ht.png)

![MeS27j.png](https://s2.ax1x.com/2019/11/08/MeS27j.png)

#### [sub模块](http://nginx.org/en/docs/http/ngx_http_sub_module.html)

![MeSOE9.png](https://s2.ax1x.com/2019/11/08/MeSOE9.png)

![MepSgK.png](https://s2.ax1x.com/2019/11/08/MepSgK.png)

sub.conf配置文件

```nginx
server{
	server_name sub.pingxin.com;
	location /{
		sub_filter 'Nginx.oRg' '$host/nginx';
		sub_filter 'nginx.cOm' '$host/nginx';
		#sub_filter_once on;
		#sub_filter_once off;
		#sub_filter_last_modified off;
		#sub_filter_last_modified on;	
	}
}
```

#### [addition模块](http://nginx.org/en/docs/http/ngx_http_addition_module.html)

![Mepxaj.png](https://s2.ax1x.com/2019/11/08/Mepxaj.png)

![Me9ALF.png](https://s2.ax1x.com/2019/11/08/Me9ALF.png)

```nginx
server{
	server_name addition.pingxin.com;
	location /{
		add_before_body /before_action;
        add_after_body /after_action;
        addition_types *;
	}
    location /before_action {
        return 200 'new content before \n'; 
    }
    location /after_action {
        return 200 'new content after \n'; 
    }
    loaction /testhost {
        uninitialized_variable_warn on;
        set $foo 'testhost';
        return 200 '$gzip_ratio\n';
    }
}
```

