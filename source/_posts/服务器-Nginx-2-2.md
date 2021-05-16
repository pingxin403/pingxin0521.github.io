---
title: nginx 正向代理、反向代理、负载均衡和web缓存
date: 2019-08-22 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

### nginx代理

nginx的正向代理，只能代理http、tcp等，不能代理https请求。有很多人不是很理解具体什么是nginx的正向代理、什么是反向代理。下面结合自己的使用做的一个简介：

1）正向代理：

   所谓正向代理就是内网服务器主动要去请求外网的地址或服务，所进行的一种行为。内网服务---访问--->外网

2）反向代理：

  所谓反向代理就是外网要访问内网服务而进行的一种行为。 外网----请求--->内网服务

<!--more-->

#### 正向代理和反向代理的区别

正向代理和反向代理的区别我在知乎上找到两张图可以帮助我们很好的理解：

![image.png](https://i.loli.net/2019/10/15/DK75VNZB6E1Rywv.png)



**正向代理:客户端 <一> 代理 一>服务端**

正向代理简单地打个租房的比方:

A(客户端)想租C(服务端)的房子,但是A(客户端)并不认识C(服务端)租不到。
B(代理)认识C(服务端)能租这个房子所以你找了B(代理)帮忙租到了这个房子。

这个过程中C(服务端)不认识A(客户端)只认识B(代理)
C(服务端)并不知道A(客户端)租了房子，只知道房子租给了B(代理)。

**反向代理:客户端 一>代理 <一> 服务端**

反向代理也用一个租房的例子:

A(客户端)想租一个房子,B(代理)就把这个房子租给了他。
这时候实际上C(服务端)才是房东。
B(代理)是中介把这个房子租给了A(客户端)。

这个过程中A(客户端)并不知道这个房子到底谁才是房东
他都有可能认为这个房子就是B(代理)的

由上的例子和图我们可以知道正向代理和反向代理的区别在于代理的对象不一样,正向代理的代理对象是客户端,反向代理的代理对象是服务端。

![image.png](https://i.loli.net/2019/10/15/O59W7Vz1CySKgiM.png)



- 正向代理: 对于客户端,我的请求到达Nginx,Nginx把我的请求分配到外部服务器,隐藏了服务端的身份
- 反向代理: 服务端向外部客户端提供服务,但是任务是由Nginx下发的,不知道客户端是谁

```
location / {
 proxy_pass http://localhost:8000;       # 设定请求跳转后的地址,可以使用 hostname 或 IP:Port 形式
 proxy_set_header X-Real-IP $remote_addr;# 后端请求携带原始请求的真实 IP 地址
```

proxy_pass:设置被代理服务器的地址和被映射的URI,地址可以使用主机名或IP加端口号的形式

```
location /html/ {
1. proxy_pass http://proxy.com;
2. proxy_pass http://proxy.com/;
}
```

假设访问的url是 `http://domain.com/html/test.js`

对于1来说 proxy.com 后面没有"/",表示"/html/" 请求(包括自己)后续的路径及其参数等关键字都由`http://a.com/`来处理,代理后变成了`http://proxy.com/html/test.js`

对 2来所 proxy.com 后面有"/",表示"/html/" 请求后续的路径及其参数等关键字都由 `http://a.com/`来处理,代理后变成了`http://proxy.com/test.js`

**一个请求的生命周期**

```
server {
    listen 8000;
    location / {
    proxy_pass http://192.168.31.129:8001;
        }
}
```

这个server服务监听`0.0.0.0:8000`端口监听表示本机素有ip的8000端口,如果有客户端请求到我们本机的8000端口,则开始匹配, `/`表示匹配所有的路径,对8000端口无条件转发到`http://192.168.31.129:8001;` ps:我们的服务器需要绑定`http://192.168.31.129:8001;`

#### 两种代理配置方式

1. 正向代理：

   nginx同样可以实现代理上网的功能，在conf.d文件夹中新建文件proxy.conf(文件名随意)，内容如下：

   ```nginx
   server {
           access_log /var/log/nginx/access.log;
       #8090这个端口，就是用来正向代理的。客户端发送到这个端口的请求，都会被代理出去。
           listen 8090;
           location / {
           #resolver 定义域名解析。本实验好像没有用到这个配置。改成一个不存在的ip都不影响。
                   resolver 8.8.8.8;
                   proxy_pass $scheme://$http_host$request_uri;
                   proxy_buffers   256 4k;
                   proxy_max_temp_file_size 0k;
           }
   }
   ```
   
   nginx实现代理上网，有三个关键点必须注意，其余的配置跟普通的nginx一样
   
   1. 增加dns解析resolver
   
   2. 增加无server_name名的server
   
   3. proxy_pass指令
   
   client端：
   
   一次代理，直接在shell执行：
   
   ```
   # export http_proxy=http://192.168.1.9:8090
   ```
   
   永久使用：
   
   ```
   # vim .bashrc
   export http_proxy=http://192.168.1.9:8090
   # source .bashrc
   ```
   
   注：取消代理`unset http_proxy`

2. 反向代理

   nginx反向代理的指令不需要新增额外的模块，默认自带proxy_pass指令，只需要修改配置文件就可以实现反向代理。

   配置前的准备工作，后端跑apache服务的ip和端口，也就是说可以通过http://ip:port能访问到你的网站。

   然后就可以新建一个proxy.conf,加入如下内容，记得修改ip和域名为你的ip和域名。

   在配置文件夹下的`conf.d`文件夹下创建`proxy.conf`

   ```nginx
   ## Basic reverse proxy server ##
   ## tomcat and httpd
   upstream ippools  {
       server 172.17.0.4:8080; #Apache
       server 172.17.0.5:8080; #Apache
   }
   
   server {
       listen 80;
       server_name  localhost;
   	root   html;
       index  index.html index.htm index.php;
   
       ## send request back to apache ##
       location / {
           proxy_pass  $scheme://ippools;
   
           #Proxy Settings
           proxy_redirect     off;
           proxy_set_header   Host             $host;
           proxy_set_header   X-Real-IP        $remote_addr;
           proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
           proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
           proxy_max_temp_file_size 0;
           proxy_connect_timeout      90;
           proxy_send_timeout         90;
           proxy_read_timeout         90;
           #该指令开启从后端被代理服务器的响应内容缓冲.
           proxy_buffering on；
           #该指令设置缓冲区大小,从被代理的后端服务器取得的响应内容,会先读取放置到这里
           #小的响应header通常位于这部分响应内容里边.
           #默认来说,该缓冲区大小等于指令proxy_buffers所设置的;但是,你可以把它设置得更小.
           proxy_buffer_size          4k;
           #该指令设置缓冲区的大小和数量,从被代理的后端服务器取得的响应内容,会放置到这里. 默认情况下,一个缓冲区的大小等于页面大小,可能是4K也可能是8K,这取决于平台
           proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
           proxy_temp_file_write_size 64k;
      }
   }
   ```
   

代理服务器站在客户端那边就是正向代理，代理服务器站在原始服务器那边就是反向代理,Nginx通过`proxy_pass`可以设置代理服务。

#### Nginx代理服务的配置说明

当代理遇到状态码为404时，我们把404页面导向百度。

```
error_page 404 https://www.baidu.com; #错误页
```

如果我们想让他起作用，我们必须配合着下面的配置一起使用

```
proxy_intercept_errors on;    #如果被代理服务器返回的状态码为400或者大于400，设置的error_page配置起作用。默认为off。
```

如果我们的代理只允许接受get，post请求方法的一种

```
proxy_method get;    #支持客户端的请求方法。post/get；
```

设置支持的http协议版本

```
proxy_http_version 1.0 ; #Nginx服务器提供代理服务的http协议版本1.0，1.1，默认设置为1.0版本
```

如果你的nginx服务器给2台web服务器做代理，负载均衡算法采用轮询，那么当你的一台机器web程序iis关闭，也就是说web不能访问，那么nginx服务器分发请求还是会给这台不能访问的web服务器，如果这里的响应连接时间过长，就会导致客户端的页面一直在等待响应，对用户来说体验就打打折扣，这里我们怎么避免这样的情况发生呢。这里我配张图来说明下问题。

![image.png](https://i.loli.net/2019/10/15/KVrFoa7TJ982zB1.png)

如果负载均衡中其中web2发生这样的情况，nginx首先会去web1请求，但是nginx在配置不当的情况下会继续分发请求道web2，然后等待web2响应，直到我们的响应时间超时，才会把请求重新分发给web1，这里的响应时间如果过长，用户等待的时间就会越长。

下面的配置是解决方案之一。

```
proxy_connect_timeout 1;   #nginx服务器与被代理的服务器建立连接的超时时间，默认60秒
proxy_read_timeout 1; #nginx服务器想被代理服务器组发出read请求后，等待响应的超时间，默认为60秒。
proxy_send_timeout 1; #nginx服务器想被代理服务器组发出write请求后，等待响应的超时间，默认为60秒。
proxy_ignore_client_abort on;  #客户端断网时，nginx服务器是否终端对被代理服务器的请求。默认为off。
```

如果使用upstream指令配置啦一组服务器作为被代理服务器，服务器中的访问算法遵循配置的负载均衡规则，同时可以使用该指令配置在发生哪些异常情况时，将请求顺次交由下一组服务器处理。

```
proxy_next_upstream timeout;  #反向代理upstream中设置的服务器组，出现故障时，被代理服务器返回的状态值。
error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off
```

- error：建立连接或向被代理的服务器发送请求或读取响应信息时服务器发生错误。
- timeout：建立连接，想被代理服务器发送请求或读取响应信息时服务器发生超时。
- invalid_header:被代理服务器返回的响应头异常。
- off:无法将请求分发给被代理的服务器。
- http_400，....:被代理服务器返回的状态码为400，500，502，等。

如果你想通过http获取客户的真是ip而不是获取代理服务器的ip地址，那么要做如下的设置。

```
proxy_set_header Host $host; #只要用户在浏览器中访问的域名绑定了 VIP VIP 下面有RS；则就用$host ；host是访问URL中的域名和端口  www.taobao.com:80

proxy_set_header X-Real-IP $remote_addr;  #把源IP 【$remote_addr,建立HTTP连接header里面的信息】赋值给X-Real-IP;这样在代码中 $X-Real-IP来获取 源IP

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;#在nginx 作为代理服务器时，设置的IP列表，会把经过的机器ip，代理机器ip都记录下来，用 【，】隔开；代码中用 echo $x-forwarded-for |awk -F, '{print $1}' 来作为源IP
```

**HTTP 请求头中的 X-Forwarded-For**

通过名字就知道，X-Forwarded-For 是一个 HTTP 扩展头部。HTTP/1.1（RFC 2616）协议并没有对它的定义，它最开始是由 Squid 这个缓存代理软件引入，用来表示 HTTP 请求端真实 IP。如今它已经成为事实上的标准，被各大 HTTP 代理、负载均衡等转发服务广泛使用，并被写入 [RFC 7239](http://tools.ietf.org/html/rfc7239)（Forwarded HTTP Extension）标准之中。

X-Forwarded-For 请求头格式非常简单，就这样：

```shell
X-Forwarded-For: client, proxy1, proxy2
```

可以看到，XFF 的内容由「英文逗号 + 空格」隔开的多个部分组成，最开始的是离服务端最远的设备 IP，然后是每一级代理设备的 IP。

如果一个 HTTP 请求到达服务器之前，经过了三个代理 Proxy1、Proxy2、Proxy3，IP 分别为 IP1、IP2、IP3，用户真实 IP 为 IP0，那么按照 XFF 标准，服务端最终会收到以下信息：

```shell
X-Forwarded-For: IP0, IP1, IP2
```

Proxy3 直连服务器，它会给 XFF 追加 IP2，表示它是在帮 Proxy2 转发请求。列表中并没有 IP3，IP3 可以在服务端通过 Remote Address 字段获得。我们知道 HTTP 连接基于 TCP 连接，HTTP 协议中没有 IP 的概念，Remote Address 来自 TCP 连接，表示与服务端建立 TCP 连接的设备 IP，在这个例子里就是 IP3。

Remote Address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接，更不会有后面的 HTTP 请求。不同语言获取 Remote Address 的方式不一样，例如 php 是 `$_SERVER["REMOTE_ADDR"]`，Node.js 是 `req.connection.remoteAddress`，但原理都一样。

1. 直接对外提供服务的 Web 应用，在进行与安全有关的操作时，只能通过 Remote Address 获取 IP，不能相信任何请求头；
2. 使用 Nginx 等 Web Server 进行反向代理的 Web 应用，在配置正确的前提下，要用 `X-Forwarded-For` 最后一节 或 `X-Real-IP` 来获取 IP（因为 Remote Address 得到的是 Nginx 所在服务器的内网 IP）；同时还应该禁止 Web 应用直接对外提供服务；
3. 在与安全无关的场景，例如通过 IP 显示所在地天气，可以从 `X-Forwarded-For` 靠前的位置获取 IP，但是需要校验 IP 格式合法性；

PS：网上有些文章建议这样配置 Nginx，其实并不合理：

```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $remote_addr;
```

这样配置之后，安全性确实提高了，但是也导致请求到达 Nginx 之前的所有代理信息都被抹掉，无法为真正使用代理的用户提供更好的服务。还是应该弄明白这中间的原理，具体场景具体分析。

**关于代理配置的配置文件部分示例**

```nginx
	include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat ' $remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。
    proxy_connect_timeout 1;   #nginx服务器与被代理的服务器建立连接的超时时间，默认60秒
    proxy_read_timeout 1; #nginx服务器想被代理服务器组发出read请求后，等待响应的超时间，默认为60秒。
    proxy_send_timeout 1; #nginx服务器想被代理服务器组发出write请求后，等待响应的超时间，默认为60秒。
    proxy_http_version 1.0 ; #Nginx服务器提供代理服务的http协议版本1.0，1.1，默认设置为1.0版本。
    #proxy_method get;    #支持客户端的请求方法。post/get；
    proxy_ignore_client_abort on;  #客户端断网时，nginx服务器是否终端对被代理服务器的请求。默认为off。
    proxy_ignore_headers "Expires" "Set-Cookie";  #Nginx服务器不处理设置的http相应投中的头域，这里空格隔开可以设置多个。
    proxy_intercept_errors on;    #如果被代理服务器返回的状态码为400或者大于400，设置的error_page配置起作用。默认为off。
    proxy_headers_hash_max_size 1024; #存放http报文头的哈希表容量上限，默认为512个字符。
    proxy_headers_hash_bucket_size 128; #nginx服务器申请存放http报文头的哈希表容量大小。默认为64个字符。
    proxy_next_upstream timeout;  #反向代理upstream中设置的服务器组，出现故障时，被代理服务器返回的状态值。error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off
    #proxy_ssl_session_reuse on; 默认为on，如果我们在错误日志中发现“SSL3_GET_FINSHED:digest check failed”的情况时，可以将该指令设置为off。
```

### 负载均衡

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。扩展策略，就天马行空，只有你想不到的没有他做不到的啦，你可以参照所有的负载均衡算法，给他一一找出来做下实现。

#### 负载均衡算法

1. 源地址哈希法：根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。

2. 轮询法：将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

3. 随机法：通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。

4. 加权轮询法：不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。

5. 加权随机法：与加权轮询法一样，加权随机法也根据后端机器的配置，系统的负载分配不同的权重。不同的是，它是按照权重随机请求后端服务器，而非顺序。

6. 最小连接数法：由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。


上3个图，理解这三种负载均衡算法的实现

![image.png](https://i.loli.net/2019/10/15/Z42exq5GAQFTYBl.png)

Ip hash算法，对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。 

![image.png](https://i.loli.net/2019/10/15/spbCTzYkwDNqGhu.png)

**示例**

```nginx
//test-yii2.conf
upstream guwenjie_http {
        server **.***.***.***:9503 weight=1;
        server **.***.***.***:8811 weight=2;
}
server
     {
        listen 80;
        #listen [::]:80 default_server ipv6only=on;
        server_name test1.freephp.top;
        index  index.php index.html   index.htm ;
        root   /home/wwwroot/workspace/public/static;

        #error_page   404   /404.html;

        # Deny access to PHP files in specific directory
        #location ~ /(wp-content|uploads|wp-includes|images)/.*\.php$ { deny all; }

        include enable-php.conf;
		
		location / {
             if (!-e $request_filename){
                #proxy_pass http://127.0.0.1:8855;
                proxy_pass http://guwenjie_http;
             }
         }
         
        location /nginx_status
        {
            stub_status on;
            access_log   off;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }

        location ~ /.well-known {
            allow all;
        }

        location ~ /\.
        {
            deny all;
        }
```

我们使用 nginx 中的 upstream模块 来实现nginx将跨越单机的限制，完成网络数据的接收、处理和转发。我们主要使用提到的转发功能进行调度分发。

我定义的 upstream 模块名称是 guwenjie_http （最好定义一个有意义的，这个就很不好 _），我配置了两个IP端口，到时候nginx分发的视乎就往这两个服务器上分发。

先来对 upstream 进行一个说明吧

```nginx
//举例，以下IP，端口无效
 upstream test{ 
      server 11.22.333.11:6666 weight=1; 
      server 11.22.333.22:8888 down; 
      server 11.22.333.33:8888 backup;
      server 11.22.333.44:5555 weight=2; 
}
//down 表示单前的server临时不參与负载.
//weight 默觉得1.weight越大，负载的权重就越大
//backup： 其他全部的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻
```

后面的 weight=1，weight=2 是表示权重的意思，数字越大，权重越高，在该例中 8811 这个端口权重就是 8855 的两倍，比如三次请求，大概就是两次分发给 8811 一次分发给 8855 ，其实这个是不需要写的，upstream 模块默认就是轮询法，每个ip分发一次，设置权重（加权轮询法）的意义上面已经解释过了。

1、热备：如果你有2台服务器，当一台服务器发生事故时，才启用第二台服务器给提供服务。服务器处理请求的顺序：AAAAAA突然A挂啦，BBBBBBBBBBBBBB.....

```
upstream mysvr { 
      server 127.0.0.1:7878; 
      server 192.168.10.121:3333 backup;  #热备     
    }
```

2、轮询：nginx默认就是轮询其权重都默认为1，服务器处理请求的顺序：ABABABABAB....

```
upstream mysvr { 
      server 127.0.0.1:7878;
      server 192.168.10.121:3333;       
    }
```

3、加权轮询：跟据配置的权重的大小而分发给不同服务器不同数量的请求。如果不设置，则默认为1。下面服务器的请求顺序为：ABBABBABBABBABB....

```
 upstream mysvr { 
      server 127.0.0.1:7878 weight=1;
      server 192.168.10.121:3333 weight=2;
      }
```

4、ip_hash:nginx会让相同的客户端ip请求相同的服务器。

```
upstream mysvr { 
      server 127.0.0.1:7878; 
      server 192.168.10.121:3333;
      ip_hash;
    }
```

5、fair（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。与weight分配策略类似。

```
upstream backserver {
    server server1;
    server server2;
    fair;
}
```

6、url_hash（第三方）按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

```
upstream backserver {
    server squid1:3128;
    server squid2:3128;
    hash $request_uri;
    hash_method crc32;
}
```

7、如果你对上面4种均衡算法不是很理解，那么麻烦您去看下我上一篇配的图片，可能会更加容易理解点。

到这里你是不是感觉nginx的负载均衡配置特别简单与强大，那么还没完，咱们继续哈，这里扯下蛋。

关于nginx负载均衡配置的几个状态参数讲解。

- down，表示当前的server暂时不参与负载均衡。
- backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
- weight 默认为1.weight越大，负载的权重就越大。
- max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
- fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

```
 upstream mysvr { 
      server 127.0.0.1:7878 weight=2 max_fails=2 fail_timeout=2;
      server 192.168.10.121:3333 weight=1 max_fails=2 fail_timeout=1;    
    }
```

到这里应该可以说nginx的内置负载均衡算法已经没有货啦。如果你像跟多更深入的了解nginx的负载均衡算法，nginx官方提供一些插件大家可以了解下。

配置实例：

```nginx
#user  nobody;
worker_processes  4;
events {
    # 最大并发数
    worker_connections  1024;
}
http{
    # 待选服务器列表
    upstream myproject{
        # ip_hash指令，将同一用户引入同一服务器。
        ip_hash;
        server 125.219.42.4 fail_timeout=60s;
        server 172.31.2.183;
        }

    server{
                # 监听端口
                listen 80;
                # 根目录下
                location / {
                    # 选择哪个服务器列表
                    proxy_pass http://myproject;
                }

            }
}
```

#### 负载均衡的session共享

如果网站是存放在一台机器上，是不存在session共享这个问题的，因为所有的会话数据都在这一台机器上。但是，现在的网站大部分都是需要做负载均衡的，即需要把用户的请求分发到不同机器，当然这时会话ID在客户端是不存在问题的，但是服务端会出现取不到session数据的情况。如下图：

![image.png](https://i.loli.net/2019/10/15/HACtzigNW9X36GM.png)

在该架构中，采用Nginx做负载均衡，两个Tomcat做后端服务器，假设当客户端第一次请求时，Nginx将其分发到了Tomcat1，这时候Tomcat1会产生sessionID返回给客户端，并同时保存在自己的内存中；当客户端第二次请求时，Nginx将其分发到了Tomcat2，这时便无法取到session。从而就会重新生成session，返回给客户端，并保存在自己的内存中。两台Tomcat中保存的同一个用户的session不同，这便是session的一致性问题。

**解决的方案**

1. 不使用session，使用cookie

   session是存放在服务器端的，cookie是存放在客户端的，我们可以把用户访问页面产生的session放到cookie里面，就是以cookie为中转站。你访问web服务器A，产生了session然后把它放到cookie里面，当你的请求被分配到B服务器时，服务器B先判断服务器有没有这个session，如果没有，再去看看客户端的cookie里面有没有这个session，如果也没有，说明session真的不存，如果cookie里面有，就把cookie里面的sessoin同步到服务器B，这样就可以实现session的同步了。

   说明：这种方法实现起来简单，方便，也不会加大数据库的负担，但是如果客户端把cookie禁掉了的话，那么session就无从同步了，这样会给网站带来损失；cookie的安全性不高，虽然它已经加了密，但是还是可以伪造的，所以这种方式也是不推荐的。

2.  session存在数据库mysql

   session保存在数据库中，是把session表和其他的数据表存放在一起，那么当用户只要登录后随便操作了些什么就要去数据库验证一下session的状态，这样无疑加重了mysql数据库的压力；如果数据库也做了集群的话，那么也就是说每个数据库集群的节点都得保存这个session表，而且要保证每个集群的节点中数据库的session表的数据保持一致，实时同步

   说明：session保持在数据库，加重了数据库的IO，增大数据库的压力和负担，从而影响数据库的读写性能，而且mysql集群的话也不利于session的实时同步

3. session存在缓存memcache或者redis中

   memcache可以做分布式，php配置文件中设置存储方式为memcache，这样php自己会建立一个session集群，将session数据存储在memcache中。

   说明：这种方式来同步session，不会加大数据库的负担，而且安全性比用cookie保存session大大的提高，把session放到内存里面，比从文件中读取要快很多。但是memcache把内存分成很多种规格的存储块，有块就有大小，这种方式也就决定了，memcache不能完全利用内存，会产生内存碎片，如果存储块不足，还会产生内存溢出。

4. ip_hash技术，nginx中可以配置，当某个ip下的客户端请求指定（固定，因为根据IP地址计算出一个hash值，根据hash值来判断分配给那台服务器，从而每次该ip请求都分配到指定的服务器）的服务器，这样就可以保证有状态请求的状态的完整性，不至于出现状态丢失的情况，一下是nginx的配置，可以参考一下：

   ```nginx
   upstream nginx.example.com  
       {   
                server 192.168.1.2:80;   
                server 192.168.1.3:80;  
                ip_hash;  
       }  
       server  
       {  
                listen 80;  
                location /  
                {  
                        proxy_pass  
                       http://nginx.example.com;  
                }  
    }
   ```

   注意：ip_hash这个方案确实可以保证带有状态的请求的完整性，但是它有一个很大的缺陷，那就是ip_hash方案必须保证Nginx是最前端的服务器（接受真实的ip），如果nginx不是最前端的服务器，还存在中间件（中间服务器什么的），那么nginx获取的ip地址就不是真实的ip地址，那么这个ip_hash就没有任何意义

5. 还有其他的一些方法，本人暂时还不太清楚，有待继续的学习（url_hash不知道可以不可以？是否添加一台共享数据的服务器，把状态等一些公共的数据都保持到这台服务器上，从而集群的所有服务器都从共享服务器上边获取状态进行验证？？待求证）

为了解决这个问题，首当其冲，我会想到，将Tomcat1中的session复制到Tomcat2中即可，当然是可以的，但是不方便，因为这里只有两台服务器，而当后台服务器增多时，会很麻烦。从而，便有了如下的解决方法：

![image.png](https://i.loli.net/2019/10/15/mU1kfblEJQzxZ9I.png)

即，将session分离出来，每个服务器都是从该session服务器（集群）中获取。这样以来，新增加的服务器也只需从session集群中获取。

（session集群可以通过memcached或redis来实现）

##### 使用nginx和tomcat配置负载均衡和session共享

<https://blog.csdn.net/u012383839/article/details/79801105>

### Nginx动静分离

必须依赖服务器生存的我们称为动。不需要依赖容器的比如css/js或者图片等，这类就叫静

**静态资源的类型** 

在Nginx的conf目录下，有一个mime.types文件

输入命令 ：`cat ../conf/mime.types`

```
types {
text/html html htm shtml;
text/css css;
text/xml xml;
image/gif gif;
image/jpeg jpeg jpg;
application/javascript js;
application/atom+xml atom;
application/rss+xml rss;
text/mathml mml;
text/plain txt;
text/vnd.sun.j2me.app-descriptor jad;
text/vnd.wap.wml wml;
text/x-component htc;
image/png png;
image/svg+xml svg svgz;
image/tiff tif tiff;
image/vnd.wap.wbmp wbmp;
image/webp webp;
image/x-icon ico;
image/x-jng jng;
image/x-ms-bmp bmp;
application/font-woff woff;
application/java-archive jar war ear;
application/json json;
application/mac-binhex40 hqx;
application/msword doc;
application/pdf pdf;
application/postscript ps eps ai;
application/rtf rtf;
application/vnd.apple.mpegurl m3u8;
application/vnd.google-earth.kml+xml kml;
application/vnd.google-earth.kmz kmz;
application/vnd.ms-excel xls;
application/vnd.ms-fontobject eot;
application/vnd.ms-powerpoint ppt;
application/vnd.oasis.opendocument.graphics odg;
application/vnd.oasis.opendocument.presentation odp;
application/vnd.oasis.opendocument.spreadsheet ods;
application/vnd.oasis.opendocument.text odt;
application/vnd.openxmlformats-officedocument.presentationml.presentation
pptx;
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
xlsx;
application/vnd.openxmlformats-officedocument.wordprocessingml.document
docx;
application/vnd.wap.wmlc wmlc;
application/x-7z-compressed 7z;
application/x-cocoa cco;
application/x-java-archive-diff jardiff;
application/x-java-jnlp-file jnlp;
application/x-makeself run;
application/x-perl pl pm;
application/x-pilot prc pdb;
application/x-rar-compressed rar;
application/x-redhat-package-manager rpm;
application/x-sea sea;
application/x-shockwave-flash swf;
application/x-stuffit sit;
application/x-tcl tcl tk;
application/x-x509-ca-cert der pem crt;
application/x-xpinstall xpi;
application/xhtml+xml xhtml;
application/xspf+xml xspf;
application/zip zip;
application/octet-stream bin exe dll;
application/octet-stream deb;
application/octet-stream dmg;
application/octet-stream iso img;
application/octet-stream msi msp msm;
audio/midi mid midi kar;
audio/mpeg mp3;
audio/ogg ogg;
audio/x-m4a m4a;
audio/x-realaudio ra;
video/3gpp 3gpp 3gp;
video/mp2t ts;
video/mp4 mp4;
video/mpeg mpeg mpg;
video/quicktime mov;
video/webm webm;
video/x-flv flv;
video/x-m4v m4v;
video/x-mng mng;
video/x-ms-asf asx asf;
video/x-ms-wmv wmv;
video/x-msvideo avi;
}
```

用户访问一个网站，然后从服务器端获取相应的资源通过浏览器进行解析渲染最后展示给用户，而服务端可以返回

各种类型的内容，比如xml、jpg、png、gif、flflash、MP4、html、css等等，那么浏览器就是根据mime-type来决定用什么形式来展示的

服务器返回的资源给到浏览器时，会把媒体类型告知浏览器，这个告知的标识就是Content-Type，比如Content-Type:text/html。 

**演示代码**

```nginx
location ~ .*\.(js|css|png|svg|ico|jpg)$ {
    valid_referers none blocked 192.168.20.130 www.gupaoedu.com;
    if ($invalid_referer) {
        return 404;
    }
    root static-resource;
    expires 1d;
}    
```

**动静分离的好处**

- 第一个，Nginx本身就是一个高性能的静态web服务器；

- 第二个，其实静态文件有一个特点就是基本上变化不大，所以动静分离以后我们可以对静态文件进行缓存、或者压缩提高网站性能 

### web缓存

当一个客户端请求web服务器, 请求的内容可以从以下几个地方获取：服务器、浏览器缓存中或缓存服务器中。这取决于服务器端输出的页面信息

浏览器缓存将文件保存在客户端，好的缓存策略可以减少对网络带宽的占用，可以提高访问速度，提高用户的体验，还可以减轻服务器的负担nginx缓存配置 

Nginx的Web缓存服务主要由proxy_cache相关指令集和fastcgi_cache相关指令集构成，前者用于反向代理时，对后端内容源服务器进行缓存，后者主要用于对FastCGI的动态程序进行缓存。两者的功能基本上一样。

最新的Nginx版本，proxy_cache和fastcgi_cache已经比较完善，加上第三方的ngx_cache_purge模块（用于清除指定URL的缓存），已经可以完全取代Squid。

**proxy_cache相关指令集**

1. proxy_cache指令 语法: `proxy_cache zone_name ;`
   该指令用于设置哪个缓存区将被使用,zone_name的值为proxy_cache_path指令创建的缓存区的名称.

2. proxy_cache_path指令, 语法 

   ```
   proxy_cache_path path [levels=number]
   keys_zone=zone_name:zone_size[inactive=time] [max_size=size];
   ```

   该指令用于设置缓存文件的存放路径.例:

   ```
   proxy_cache_path /data0/proxy_cache_dir levels=1:2 keys_zone=cache_one:500m
   inactive=1d max_size=30g ;
   ```

   path 存放目录
   levels 指定该缓存空间有两层hash目录,第一层目录为1个字母,第二层目录为2个字母,保存的文件名会类似/data0/proxy_cache_dir/c/29/XXXXXX ;
   keys_zone参数用来为这个缓存区起名.
   500m 指内存缓存空间大小为500MB
   inactive的1d指如果缓存数据在1天内没有被访问,将被删除
   max_size的30g是指硬盘缓存空间为30G

3. proxy_cache_methods 指令 

   语法:`proxy_cache_methods[GET HEAD POST];`
   该指令用于设置缓存哪些HTTP方法,默认缓存HTTP GET/HEAD方法,不缓存HTTP POST 方法

4. proxy_cache_min_uses指令 

   语法:`proxy_cache_min_uses the_number`
   该指令用于设置缓存的最小使用次数,默认值为1

5. proxy_cache_valid指令 

   语法: `proxy_cache_valid reply_code [reply_code...] time ;`
   该指令用于对不同返回状态码的URL设置不同的缓存时间.
   例:

   ```
   proxy_cache_valid 200 302 10m ;
   proxy_cache_valid 404 1m ;
   ```

   设置200,302状态的URL缓存10分钟,404状态的URL缓存1分钟.

6. proxy_cache_key指令 

   语法: `proxy_cache_key line ;`
   该指令用来设置Web缓存的Key值,Nginx根据Key值md5哈希存储缓存.一般根据$host(域名),$request_uri(请求的路径)等变量组合成proxy_cache_key .

**proxy_cache完整示例**

Nginx配置文件(nginx.conf)对扩展名为gif,jpg,jpeg,png,bmp,swf,js,css的图片,flash，javascript , css文件开启Web缓存,其他文件不缓存.

```nginx
http{
  proxy_temp_path /data0/proxy_temp_path ;
  #设置Web缓存区名称为cache_one,内存缓存空间大小为500M,自动清除超过1天没有被  

#访问的缓存数据,硬盘缓存空间大小为30G
  proxy_cache_path /data0/proxy_cache_path levels=1:2

keys_zone=cache_one:200m inactive=1d max_size=30g ;
    
  server{
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$
    {
     #使用Web缓存区cache_one
     proxy_cache cache_one ;
     #对不同HTTP状态码缓存设置不同的缓存时间
     proxy_cache_valid 200 304 12h ;
     proxy_cache_valid 301 302 1m ;
     proxy_cache_valid any 1m ;
     #设置Web缓存的Key值,Nginx根据Key值md5哈希存储缓存,这里根据"域名,URI,
     #参数"组合成Key
     proxy_cache_key $host$uri$is_args$args;
    }

   #用于清除缓存,假设一个URL为http://my.domain.com/test.gif,通过访问
   #http://my.domain.com/purge/test.gif可以清除该URL的缓存
    location ~ /purge(/.*)
    {
     #设置只允许指定的IP或IP段才可以清除URL缓存
     allow 127.0.0.1 ;
     allow 192.168.0.0/16 ;
     deny all ;
     proxy_cache_purge cache_one $host$1$is_args$args ;
    }
  }
}

```

**fastcgi_cache相关指令集**

1. fastcgi_cache指令
   语法:`fastcgi_cache zone_name;`
   该指令用于设置哪个缓存区将被使用,zone_name的值为fastcgi_cache_path指令创建的缓存区名称.

2. fastcgi_cache_path指令
   语法:`fastcgi_cache_path path [levels=number] keys_zone=zone_name:zone_size [inactive=time] [max_size=size];`
   该指令用于设置缓存文件的存放路径,
   例:

   ```
   fastcgi_cache_path /data0/fastcgi_cache_dir levels=1:2
   keys_zone=cache_one:500m inactive=1d max_size=30g ;
   ```

   注意这个指令只能在http标签内配置,levels指定该缓存空间有两层hash目录,第一层目录为1个字母,第二层为2个字母,保存的文件名会类似/data0/fastcgi_cache_dir/c/29/XXXX;

   keys_zone参数用来为这个缓存区起名,
   500m指内存缓存空间大小为500MB;
   inactive的1d指如果缓存数据在1天内没有被访问,将被删除;
   max_size的30g是指硬盘缓存空间为30GB

3. fastcgi_cache_methods指令
   语法:`fastcgi_cache_methods [GET HEAD POST] ;`
   该指令用于设置缓存哪些HTTP方法,默认缓存HTTP GET/HEAD 方法,不缓存HTTP POST方法

4. fastcgi_cache_min_uses指令
   语法:`fastcgi_cache_min_uses the_number;`
   该指令用于设置缓存的最小使用次数,默认值为1.

5. fastcgi_cache_valid指令
   `fastcgi_cache_valid reply_code [reply_code...] time;`
   该指令用于对不同返回状态码的URL设置不同的缓存时间.

   ```
   fastcgi_cache_valid 200 302 10m ;
   fastcgi_cache_valid 404 1m ;
   ```

   设置200,302状态的URL缓存10分钟,404状态的URL缓存1分钟.
   如果不指定状态码,直接指定缓存时间,则只有200,301,302状态的URL缓存5分钟.

6. fastcgi_cache_key指令
   语法:`fastcgi_cache_key line ;`
   该指令用来设置Web缓存的Key值,Nginx根据Key值md5哈希存储缓存.一般根据FastCGI服务器的地址和端口,$request_uri(请求的路径)等变量组合成fastcgi_cache_key。

**fastcgi_cache完整示例**

Nginx配置文件(nginx.conf)对扩展名为gif,jpg,jpeg,png,bmp,swf,js,css的图片,Flash,JavaScript,CSS文件开启Web缓存,其他文件不缓存.

```nginx
http{
 #fastcgi_temp_path和fastcgi_cache_path指定的路径必须在同一分区
  fastcgi_temp_path /data0/fastcgi_temp_path ;
 #设置Web缓存区名称为cache_one,内存缓存空间大小为500MB,自动清除超过1天没有被

 #访问的缓存数据,硬盘缓存空间大小为30G
  fastcgi_cache_path /data0/fastcgi_cache_path levels=1:2

keys_zone=cache_one:200m inactive=1d max_size=30g ;

  server{
    location ~ .*\.(php|php5)$
    {
     #使用Web缓存区cache_one
     fastcgi_cache cache_one ;
     #对不同的HTTP状态码缓存设置不同的缓存时间
     fastcgi_cache_valid 200 10m ;
     fastcgi_cache_valid 301 302 1h ;
     fastcgi_cache_valid an 1m ;
     #设置Web缓存的key值,Nginx根据key值md5哈希存储缓存,这里根据"FastCGI服务  

   #器的IP,端口,请求的URI"组合成Key。
     fastcgi_cache_key 127.0.0.1:9000$requet_uri ;
     #FastCGI服务器
     fastcgi_pass 127.0.0.1:9000 ;
     fastcgi_index index.php ;
     include fcgi.conf ;
    }
  }
}
```





#### 参考

1. [HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)
2. [Nginx代理功能与负载均衡详解](https://www.cnblogs.com/knowledgesea/p/5199046.html)
3. [Nginx负载均衡高可用](http://www.uml.org.cn/zjjs/201808214.asp)
4. [nginx和keepalived实现nginx高可用](https://blog.csdn.net/u012453843/article/details/69668663)
5. [nginx:负载均衡的session共享](https://www.cnblogs.com/zengguowang/p/8261695.html)
6. [Nginx的session一致性问题](https://blog.csdn.net/qq_31246691/article/details/81517186)