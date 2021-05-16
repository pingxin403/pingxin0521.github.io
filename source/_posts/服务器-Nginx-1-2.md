---
title: nginx 配置详解
date: 2019-08-20 12:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

### Nginx工作原理

Nginx由内核和模块组成，完成工作是通过查找配置文件将客户端请求映射到一个location block(location是用于URL匹配的命令)，location配置的命令会启动不同模块完成工作。

<!--more-->

Nginx模块分为核心模块，基础模块和第三方模块。

> **核心模块：**HTTP模块、EVENT模块(事件)、MAIL模块。
> **基础模块：**HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块、HTTP Rewrite模块。
> **第三方模块：**HTTP Upstream Request Hash模块、Notice模块、HTTP Access Key模块。

![uLbW6A.png](https://s2.ax1x.com/2019/10/12/uLbW6A.png)


#### Nginx配置文件结构

如果你下载好啦，你的安装文件，不妨打开conf文件夹的nginx.conf文件，Nginx服务器的基础配置，默认的配置也存放在此。

在nginx.conf的注释符号位#

nginx文件的结构，这个对刚入门的同学，可以多看两眼。

默认的config 

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

nginx文件结构

```
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

![image.png](https://i.loli.net/2019/10/14/IDPv5bH6oOxRMed.png)

1. 全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. server块：配置虚拟主机的相关参数，一个http中可以有多个server。
5. location块：配置请求的路由，以及各种页面的处理情况。

下面给大家上一个配置文件，作为理解，同时也配入我搭建的一台测试机中，给大家示例。 

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

    upstream mysvr {   #上游服务
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

上面是nginx的基本配置，需要注意的有以下几点：

1. 惊群现象：一个网路连接到来，多个睡眠的进程被同事叫醒，但只有一个进程能获得链接，这样会影响系统性能；

2. 每个指令必须有分号结束。

3. 内置变量

   ```
   $uri:	当前请求的uri，不带参
   $request_uri:	请求的uri，带完整参数
   $host:	http请求报⽂中host⾸部,如果没有则以处理此请求的虚拟主机的主机名代替
   $hostname:	nginx服务运⾏在主机的主机名
   $remote_addr:	客户端IP
   $remote_port:	客户端端⼝
   $remote_user:	使⽤⽤户认证时客户端⽤户输⼊的⽤户名
   $request_filename:	⽤户请求中的URI经过本地root或alias转换后映射的本地⽂件路径
   $request_method:	请求⽅法,	GET	POST	PUT
   $server_addr:	服务器地址
   $server_name:	服务器名称
   $server_port:	服务器端⼝
   $server_protocol:	服务器向客户端发送响应时的协议,	如http/1.1	http/1.0
   $scheme:在请求中使⽤scheme,	如http://xxx.com中的http
   $http_HEADER:	匹配请求报⽂中指定的HEADER
   $http_host:	匹配请求报⽂中的host⾸
   $document_root:	当前请求映射到的root配置
   ```

   

#### Nginx配置文件

![uLbRld.png](https://s2.ax1x.com/2019/10/12/uLbRld.png)

配置文件主要由四部分组成：main(全区设置)，server(主机配置)，upstream(负载均衡服务器设置)，和location(URL匹配特定位置设置)。

1. 全局变量

   ```nginx
   #Nginx的worker进程运行用户以及用户组
   #user  nobody nobody;
   #Nginx开启的进程数
   worker_processes  1;
   #worker_processes auto;
   #以下参数指定了哪个cpu分配给哪个进程，一般来说不用特殊指定。如果一定要设的话，用0和1指定分配方式.
   #这样设就是给1-4个进程分配单独的核来运行，出现第5个进程是就是随机分配了。eg:
   #worker_processes 4     #4核CPU 
   #worker_cpu_affinity 0001 0010 0100 1000;
   nets    
   #定义全局错误日志定义类型，[debug|info|notice|warn|crit]
   #error_log  logs/error.log  info;
   #指定进程ID存储文件位置
   #pid        logs/nginx.pid;
   #一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致。
   #vim /etc/security/limits.conf
   #  *                soft    nproc          65535
   #  *                hard    nproc          65535
   #  *                soft    nofile         65535
   #  *                hard    nofile         65535
   worker_rlimit_nofile 65535;
   ```

2. 事件配置

   ```java
   events {
       #use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
       use epoll;
       #每个进程可以处理的最大连接数，理论上每台nginx服务器的最大连接数为worker_processes*worker_connections。理论值：worker_rlimit_nofile/worker_processes
       #注意：最大客户数也由系统的可用socket连接数限制（~ 64K），所以设置不切实际的高没什么好处
       worker_connections  65535;    
       #worker工作方式：串行（一定程度降低负载，但服务器吞吐量大时，关闭使用并行方式）
       #multi_accept on; 
   }
   ```

3. http参数

   ```nginx
      #文件扩展名与文件类型映射表
       include mime.types;
       #默认文件类型
       default_type application/octet-stream;
    
   #日志相关定义
       #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
       #                  '$status $body_bytes_sent "$http_referer" '
       #                  '"$http_user_agent" "$http_x_forwarded_for"';
       #定义日志的格式。后面定义要输出的内容。
       #1.$remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；
       #2.$remote_user ：用来记录客户端用户名称；
       #3.$time_local ：用来记录访问时间与时区；
       #4.$request  ：用来记录请求的url与http协议；
       #5.$status ：用来记录请求状态； 
       #6.$body_bytes_sent ：记录发送给客户端文件主体内容大小；
       #7.$http_referer ：用来记录从那个页面链接访问过来的；
       #8.$http_user_agent ：记录客户端浏览器的相关信息
       #连接日志的路径，指定的日志格式放在最后。
       #access_log  logs/access.log  main;
       #只记录更为严重的错误日志，减少IO压力
       error_log logs/error.log crit;
       #关闭日志
       #access_log  off;
    
       #默认编码
       #charset utf-8;
       #服务器名字的hash表大小
       server_names_hash_bucket_size 128;
       #客户端请求单个文件的最大字节数
       client_max_body_size 8m;
       #指定来自客户端请求头的hearerbuffer大小
       client_header_buffer_size 32k;
       #指定客户端请求中较大的消息头的缓存最大数量和大小。
       large_client_header_buffers 4 64k;
       #开启高效传输模式。
       sendfile on;
       #防止网络阻塞
       tcp_nopush on;
       tcp_nodelay on;    
       #客户端连接超时时间，单位是秒
       keepalive_timeout 60;
       #客户端请求头读取超时时间
       client_header_timeout 10;
       #设置客户端请求主体读取超时时间
       client_body_timeout 10;
       #响应客户端超时时间
       send_timeout 10;
    
   #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。
       fastcgi_connect_timeout 300;
       fastcgi_send_timeout 300;
       fastcgi_read_timeout 300;
       fastcgi_buffer_size 64k;
       fastcgi_buffers 4 64k;
       fastcgi_busy_buffers_size 128k;
       fastcgi_temp_file_write_size 128k;
    
   #gzip模块设置
       #开启gzip压缩输出
       gzip on; 
       #最小压缩文件大小
       gzip_min_length 1k; 
       #压缩缓冲区
       gzip_buffers 4 16k;
       #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
       gzip_http_version 1.0;
       #压缩等级 1-9 等级越高，压缩效果越好，节约宽带，但CPU消耗大
       gzip_comp_level 2;
       #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
       gzip_types text/plain application/x-javascript text/css application/xml;
       #前端缓存服务器缓存经过压缩的页面
       gzip_vary on;
   ```

4. 虚拟主机基本设置

   ```nginx
   #虚拟主机定义
       server {
           #监听端口
           listen       80;
           #访问域名
           server_name  localhost;
           #编码格式，若网页格式与此不同，将被自动转码
           #charset koi8-r;
           #虚拟主机访问日志定义
           #access_log  logs/host.access.log  main;
           #对URL进行匹配
           location / {
               #访问路径，可相对也可绝对路径
               root   html;
               #首页文件。以下按顺序匹配
               index  index.html index.htm;
           }
    
   #错误信息返回页面
           #error_page  404              /404.html;
           # redirect server error pages to the static page /50x.html
           #
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   html;
           }
    
   #访问URL以.php结尾则自动转交给127.0.0.1
           # proxy the PHP scripts to Apache listening on 127.0.0.1:80
           #
           #location ~ \.php$ {
           #    proxy_pass   http://127.0.0.1;
           #}
   #php脚本请求全部转发给FastCGI处理
           # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
           #
           #location ~ \.php$ {
           #    root           html;
           #    fastcgi_pass   127.0.0.1:9000;
           #    fastcgi_index  index.php;
           #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
           #    include        fastcgi_params;
           #}
    
   #禁止访问.ht页面 （需ngx_http_access_module模块）
           # deny access to .htaccess files, if Apache's document root
           # concurs with nginx's one
           #
           #location ~ /\.ht {
           #    deny  all;
           #}
       }
   #HTTPS虚拟主机定义
       # HTTPS server
       #
       #server {
       #    listen       443 ssl;
       #    server_name  localhost;
       #    ssl_certificate      cert.pem;
       #    ssl_certificate_key  cert.key;
       #    ssl_session_cache    shared:SSL:1m;
       #    ssl_session_timeout  5m;
       #    ssl_ciphers  HIGH:!aNULL:!MD5;
       #    ssl_prefer_server_ciphers  on;
       #    location / {
       #        root   html;
       #        index  index.html index.htm;
       #    }
       #}
   #vue配置
       server {
           listen       80;
           server_name  jcsd-cdn-monitor.jdcloud.com;
    
           #charset koi8-r;
    
           #access_log  logs/host.access.log  main;
    
           root /root/dist;
    
           location / {
               try_files $uri $uri/ /index.html;
           }
    
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   html;
           }
       }
   
   
   ```

5. Nignx状态监控

   ```nginx
   #Nginx运行状态，StubStatus模块获取Nginx自启动的工作状态（编译时要开启对应功能）
   location /NginxStatus {
   	#启用StubStatus的工作访问状态    
   	stub_status    on;
       #指定StubStaus模块的访问日志文件 可off
   	access_log    logs/Nginxstatus.log;
     	#Nginx认证机制（需Apache的htpasswd命令生成）
   	#auth_basic    "NginxStatus";
   	#用来认证的密码文件
   	#auth_basic_user_file    ../htpasswd;    
   }
   ```

   访问：[http://IP/NginxStatus](http://ip/NginxStatus)(测试就不加密码验证相关)

   ![img](https://upload-images.jianshu.io/upload_images/1996162-f583bc37409ffd00.png?imageMogr2/auto-orient/strip|imageView2/2/w/316/format/webp)

   ```
   active connections – 活跃的连接数量
   server accepts handled requests — 总共处理了3个连接 , 成功创建3次握手, 总共处理了1个请求
   reading — 读取客户端的连接数.
   writing — 响应数据到客户端的数量
   waiting — 开启 keep-alive 的情况下,这个值等于 active – (reading+writing), 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接.
   ```

6. 反向代理

   ```nginx
   #以下配置追加在HTTP的全局变量中
    
   #启动代理缓存功能
   proxy_buffering on;
   #nginx跟后端服务器连接超时时间(代理连接超时)
   proxy_connect_timeout      5;
   #后端服务器数据回传时间(代理发送超时)
   proxy_send_timeout         5;
   #连接成功后，后端服务器响应时间(代理接收超时)
   proxy_read_timeout         60;
   #设置代理服务器（nginx）保存用户头信息的缓冲区大小
   proxy_buffer_size          16k;
   #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
   proxy_buffers              4 32k;
   #高负荷下缓冲大小（proxy_buffers*2）
   proxy_busy_buffers_size    64k;
   #设定缓存文件夹大小，大于这个值，将从upstream服务器传
   proxy_temp_file_write_size 64k;
   #反向代理缓存目录
   proxy_cache_path /data/proxy/cache levels=1:2 keys_zone=cache_one:500m inactive=1d max_size=1g;
   #levels=1:2 设置目录深度，第一层目录是1个字符，第2层是2个字符
   #keys_zone:设置web缓存名称和内存缓存空间大小
   #inactive:自动清除缓存文件时间。
   #max_size:硬盘空间最大可使用值。
   #指定临时缓存文件的存储路径(必须在同一分区)
   proxy_temp_path /data/proxy/temp;
    
   #服务配置
   server {
       #侦听的80端口
       listen       80;
       server_name  localhost;
       location / {
           #反向代理缓存设置命令(proxy_cache zone|off,默认关闭所以要设置)
           proxy_cache cache_one;
           #对不同的状态码缓存不同时间
           proxy_cache_valid 200 304 12h;
           #设置以什么样参数获取缓存文件名
           proxy_cache_key $host$uri$is_args$args;
           #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr; 
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
           #代理设置
           proxy_pass   http://IP; 
           #文件过期时间控制
           expires    1d;
       }
       #配置手动清楚缓存(实现此功能需第三方模块 ngx_cache_purge)
       #http://www.123.com/2017/0316/17.html访问
       #http://www.123.com/purge/2017/0316/17.html清楚URL缓存
       location ~ /purge(/.*) {
           allow    127.0.0.1;
           deny    all;
           proxy_cache_purge    cache_one    $host$1$is_args$args;
       }
       #设置扩展名以.jsp、.php、.jspx结尾的动态应用程序不做缓存
       location ~.*\.(jsp|php|jspx)?$ { 
           proxy_set_header Host $host; 
           proxy_set_header X-Real-IP $remote_addr; 
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
           proxy_pass http://IP;
       }
   ```

7. 负载均衡

   ```nginx
   #负载均衡服务器池
   upstream my_server_pool {
       #调度算法
       #1.轮循（默认）（weight轮循权值）
       #2.ip_hash：根据每个请求访问IP的hash结果分配。（会话保持）
       #3.fair:根据后端服务器响应时间最短请求。（upstream_fair模块）
       #4.url_hash:根据访问的url的hash结果分配。（需hash软件包）
       #参数：
       #down：表示不参与负载均衡
       #backup:备份服务器
       #max_fails:允许最大请求错误次数
       #fail_timeout:请求失败后暂停服务时间。
       server 192.168.1.109:80 weight=1 max_fails=2 fail_timeout=30;
       server 192.168.1.108:80 weight=2 max_fails=2 fail_timeout=30;
   }
   #负载均衡调用
   server {
       ...
       location / {
       proxy_pass http://my_server_pool;
       }
   }
   ```

8. URL重写

   ```nginx
   #根据不同的浏览器URL重写
   if($http_user_agent ~ Firefox){
   rewrite ^(.*)$  /firefox/$1 break; 
   }
   if($http_user_agent ~ MSIE){
   rewrite ^(.*)$  /msie/$1 break; 
   }
    
   #实现域名跳转
   location / {
       rewrite ^/(.*)$ https://web8.example.com$1 permanent;
   }
   ```

9. IP限制

   ```nginx
   #限制IP访问
   location / {
       deny 192.168.0.2；
       allow 192.168.0.0/24;
       allow 192.168.1.1;
       deny all;
   }
   ```

10. expires缓存功能

    作用: 设置expires减少不必要的http请求

    将静态资源(css.js.img..)等缓存到客户端,减轻服务器压力

    ```
    #步骤1：在虚拟主机server中增加下属代码即可
    location ~ \.(gif|jpg|jpeg|png|bmp|ico|js|css)$ {
        root /php/wwwroot/web1;
        expires 1d;  #1d （天）  1h（时）  1m（分）  1s（秒）
    }
    #步骤2：重启nginx
    nginx -s reload
    #步骤3：创建测试文件查看是否设置成功
    echo 'alert(1)' > /php/wwwroot/web1/test2.js
    echo  \
    '<script src=./test2.js></script>' > \
    /php/wwwroot/web1/test2.html
    ```

    linux设置电脑时间

    ```bash
    date -s '2018/7/13 15:11:25'
    clock -w #将时间写到磁盘中
    ```

    为什么开启expires速度快

    ```u
    为什么304比200快？
    明确304和200都有发送http请求，但是304会进行检测，如果未修改则不响应数据，而是从浏览器返回
    为什么expire比304快？
    因为静态资源设置了expire直接从浏览器获取不发送http请求
    ```

    ![image.png](https://i.loli.net/2019/10/15/hyiEjr9fPqXkTzu.png)

11. 防盗链

    一个网站上会有很多的图片，如果你不希望其他网站直接用你的图片地址访问自己的图片，或者希望对图片有版权保护。再或者不希望被第三方调用造成服务器的负载以及消耗比较多的流量问题，那么防盗链就是你必须要做的

    **防盗链配置**

    在Nginx中配置防盗链其实很简单，

    语法: `valid_referers none | blocked | server_names | string ...;`

    默认值: —

    上下文: server, location 

    “Referer”请求头为指定值时，内嵌变量$invalid_referer被设置为空字符串，否则这个变量会被置成“1”。查找匹配

    时不区分大小写，其中none表示缺少referer请求头、blocked表示请求头存在，但是它的值被防火墙或者代理服务器删除、server_names表示referer请求头包含指定的虚拟主机名

    ```nginx
    location ~ .*.(gif|jpg|ico|png|css|svg|js)$ {
        valid_referers none blocked 192.168.20.130;
        if ($invalid_referer) {
    　　    return 404;
        }
    　　    root static;
    }
    　　
    ```

    需要注意的是伪造一个有效的“Referer”请求头是相当容易的，因此这个模块的预期目的不在于彻底地阻止这些非法请求，而

    是为了阻止由正常浏览器发出的大规模此类请求。还有一点需要注意，即使正常浏览器发送的合法请求，也可能没有“Referer”请求头。 

12. 跨域访问

    什么叫跨域呢？如果两个节点的协议、域名、端口、子域名不同，那么进行的操作都是跨域的，浏览器为了安全问题都是限制跨域访问，所以跨域其实是浏览器本身的限制。 
    
    ```nginx
    修改proxy_demo.conf配置
    server{
        listen 80;
        server_name localhost;
        location / {
            proxy_pass http://192.168.20.130:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
            add_header 'Access-Control-Allow-Origin' '*'; // #允许来自所有的访问地址
            add_header 'Access-Control-Allow-Methods' 'GET,PUT,POST,DELETE,OPTIONS'; //支持的请求方式
            add_header 'Access-Control-Allow-Header' 'Content-Type,*'; //支持的媒体类型
    }
        location ~ .*\.(gif|jpg|ico|png|css|svg|js)$ {
       root static;
        }
    }   
    ```
    

#### 优化

1. 优化写磁盘操作：

    我们知道，每次*Nginx*访问完一个文件之后，*Linux*系统将会对它的“*Access*”，即访问时间，进行修改，    在一个高并发的访问中，这对磁盘写操作是很大的，因此要关闭这个功能。

   ```
   /dev/sdb1         /data        ext3      defaults                  0 0
   #修改为如下配置
   /dev/sdb1         /data        ext3      defaults,noatime,nodiratime  0 0
   ```

   然后重新启动系统。如果不能重启系统，那么可以使用*remount*选项来重新挂载：

   ```
   mount -o defaults,noatime, nodiratime -o remount /dev/sdb1 /sdb
   ```

2. 内核参数优化

   - fs.file-max = 999999：这个参数表示进程（比如一个worker进程）可以同时打开的最大句柄数，这个参数直线限制最大并发连接数，需根据实际情况配置。
   - net.ipv4.tcp_max_tw_buckets = 6000 ：这个参数表示操作系统允许TIME_WAIT套接字数量的最大值，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。该参数默认为180000，过多的TIME_WAIT套接字会使Web服务器变慢。注：主动关闭连接的服务端会产生TIME_WAIT状态的连接
   - net.ipv4.ip_local_port_range = 1024 65000  ：允许系统打开的端口范围。
   - net.ipv4.tcp_tw_recycle = 1 ：启用timewait快速回收。
   - net.ipv4.tcp_tw_reuse = 1 ：开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接。这对于服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接。
   - net.ipv4.tcp_keepalive_time = 30：这个参数表示当keepalive启用时，TCP发送keepalive消息的频度。默认是2小时，若将其设置的小一些，可以更快地清理无效的连接。
   - net.ipv4.tcp_syncookies = 1 ：开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理。
   - net.core.somaxconn = 40960  ：web 应用中 listen 函数的 backlog 默认会给我们内核参数的。
   - net.core.somaxconn  ：限制到128，而nginx定义的NGX_LISTEN_BACKLOG 默认为511，所以有必要调整这个值。注：对于一个TCP连接，Server与Client需要通过三次握手来建立网络连接.当三次握手成功后,我们可以看到端口的状态由LISTEN转变为ESTABLISHED,接着这条链路上就可以开始传送数据了.每一个处于监听(Listen)状态的端口,都有自己的监听队列.监听队列的长度与如somaxconn参数和使用该端口的程序中listen()函数有关。somaxconn定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128，对于一个经常处理新连接的高负载 web服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。大的侦听队列对防止拒绝服务 DoS 攻击也会有所帮助。
   - net.core.netdev_max_backlog = 262144  ：每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
   - net.ipv4.tcp_max_syn_backlog = 262144 ：这个参数标示TCP三次握手建立阶段接受SYN请求队列的最大长度，默认为1024，将其设置得大一些可以使出现Nginx繁忙来不及accept新连接的情况时，Linux不至于丢失客户端发起的连接请求。
   - net.ipv4.tcp_rmem = 10240 87380 12582912 ：这个参数定义了TCP接受缓存（用于TCP接受滑动窗口）的最小值、默认值、最大值。
   - net.ipv4.tcp_wmem = 10240 87380 12582912：这个参数定义了TCP发送缓存（用于TCP发送滑动窗口）的最小值、默认值、最大值。
   - net.core.rmem_default = 6291456：这个参数表示内核套接字接受缓存区默认的大小。
   - net.core.wmem_default = 6291456：这个参数表示内核套接字发送缓存区默认的大小。
   - net.core.rmem_max = 12582912：这个参数表示内核套接字接受缓存区的最大大小。
   - net.core.wmem_max = 12582912：这个参数表示内核套接字发送缓存区的最大大小。
   - net.ipv4.tcp_syncookies = 1：该参数与性能无关，用于解决TCP的SYN攻击。

   下面贴一个完整的内核优化设置：

   ```properties
   fs.file-max = 999999
   net.ipv4.ip_forward = 0
   net.ipv4.conf.default.rp_filter = 1
   net.ipv4.conf.default.accept_source_route = 0
   kernel.sysrq = 0
   kernel.core_uses_pid = 1
   net.ipv4.tcp_syncookies = 1
   kernel.msgmnb = 65536
   kernel.msgmax = 65536
   kernel.shmmax = 68719476736
   kernel.shmall = 4294967296
   net.ipv4.tcp_max_tw_buckets = 6000
   net.ipv4.tcp_sack = 1
   net.ipv4.tcp_window_scaling = 1
   net.ipv4.tcp_rmem = 10240 87380 12582912
   net.ipv4.tcp_wmem = 10240 87380 12582912
   net.core.wmem_default = 8388608
   net.core.rmem_default = 8388608
   net.core.rmem_max = 16777216
   net.core.wmem_max = 16777216
   net.core.netdev_max_backlog = 262144
   net.core.somaxconn = 40960
   net.ipv4.tcp_max_orphans = 3276800
   net.ipv4.tcp_max_syn_backlog = 262144
   net.ipv4.tcp_timestamps = 0
   net.ipv4.tcp_synack_retries = 1
   net.ipv4.tcp_syn_retries = 1
   net.ipv4.tcp_tw_recycle = 1
   net.ipv4.tcp_tw_reuse = 1
   net.ipv4.tcp_mem = 94500000 915000000 927000000
   net.ipv4.tcp_fin_timeout = 1
   net.ipv4.tcp_keepalive_time = 30
   net.ipv4.ip_local_port_range = 1024 65000
   ```

   执行sysctl  -p使内核修改生效。

3. 关于系统连接数的优化

   linux 默认值 open files为1024。查看当前系统值：

   ```
   # ulimit -n
   1024
   ```

   说明server只允许同时打开1024个文件。

   使用ulimit -a 可以查看当前系统的所有限制值，使用ulimit -n 可以查看当前的最大打开文件数。

   新装的linux 默认只有1024 ，当作负载较大的服务器时，很容易遇到error: too many open files。因此，需要将其改大，在/etc/security/limits.conf最后增加：

   ```
   *               soft    nofile           65535
   *               hard   nofile           65535
   *               soft    noproc         65535
   *               hard   noproc         65535
   ```

   重启系统后生效

4. 关于nginx优化：

   - 关闭访问日志

   - 使用epoll

   - nginx配置优化

     ```
     worker_connections 65535
     keepalive_timeout 60
     client_header_buffer_size 4k #通过getconf PAGESIZE获取页面大小
     worker_rlimit_nofile 65535
     ```

5. 内存优化

   使用google开发的google-perftools优化nginx的内存分配效率和速度，帮助在高并发的情况下控制内存的使用。 

   TCMalloc在内存的分配上效率和速度要比malloc高得多。但是nginx的内存占用其实是很少的，一个进程占用的内存大概只有12M左右，所有google-perftools对nginx的优化效果可能不太明显。

   下载并安装google-perftools

   ```bash
   # 从github上下载perftools工具包
   # https://github.com/gperftools/gperftools
    cd /usr/local/src/
    wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.7/gperftools-2.7.tar.gz
   # 注意：
   # 1、如果是32位系统，在configure gperftools的时候，可以不添加--enable-frame-pointers参数。
   # 2、如果是64位系统，需要先安装libunwind，再在configure gperftools的时候，添加--enable-frame-pointers参数。
   tar zxvf gperftools-2.7.tar.gz
   cd gperftools-2.7
   ./configure --enable-frame-pointers --enable-libunwind --with-tcmalloc-pagesize=32
   make && make install
   ```

   使nginx支持google-perftools

   ```bash
   # 重新编译ngx，添加--with-google_perftools_module
   # 简单演示参数添加方法
   ./configure  --prefix=/opt/server/nginx --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-google_perftools_module
   make && make install
   ```

   配置nginx对google-perftools的支持

   ```bash
   # 添加线程目录
   mkdir /tmp/tcmalloc
   chmod 0777 /tmp/tcmalloc
   
   # 修改nginx.conf，在pid行下面添加如下信息
   google_perftools_profiles /tmp/tcmalloc;
   
   # 重启nginx
    /opt/server/nginx/sbin/nginx -s reload
   
   # 验证是否支持
   lsof -n |grep tcmalloc
   nginx 1414 nobody 12w REG 252,1 0 1452011 /tmp/tcmalloc.1414
   nginx 1415 nobody 14w REG 252,1 0 1452010 /tmp/tcmalloc.1415
   # 备注：tcmalloc的记录文件数量与nginx.conf中worker_processes的值有关
   ```

   可能会遇到的问题

   ```bash
   # 启动nginx，发现缺少libprofiler.so.0动态库的支持
   /opt/openresty/nginx/sbin/nginx 
   /opt/openresty/nginx/sbin/nginx: error while loading shared libraries: libprofiler.so.0: cannot open shared object file: No such file or directory
   # 查找系统下是否有libprofiler.so.0动态库
   whereis libprofiler.so.0
   libprofiler.so: /usr/local/lib/libprofiler.so.0 /usr/local/lib/libprofiler.so
   # 由于不在nginx程序查找的目录下，所以需要创建软链接
   # 评估缺少的动态库支持，可通过ldd /opt/openresty/nginx/sbin/nginx的方式，查看"not found"部分。
   ln -s /usr/local/lib/libprofiler.so.0.4.18 /lib/libprofiler.so.0
   ```

   

### 参考

1. [Nginx 一个高性能的HTTP和反向代理服务器](https://www.cnblogs.com/wcwnina/p/9946747.html)
2. [Nginx 安装与部署配置以及Nginx和uWSGI开机自启](https://www.cnblogs.com/wcwnina/p/8728430.html)
3. [基于Linux和Nginx的Django部署](https://www.cnblogs.com/wcwnina/p/9906081.html)
4. [Nginx 相关介绍(Nginx是什么?能干嘛?)](https://www.cnblogs.com/wcwnina/p/8728391.html)
5. [Nginx 配置 HTTPS 服务器](https://aotu.io/notes/2016/08/16/nginx-https/index.html)
