---
title: nginx 常用屏蔽规则
date: 2020-11-16 11:18:59
tags:
 - 服务器
 - Nginx
categories:
 - 服务器
 - Nginx
---

#### 防盗链

防盗链的原理其实很简单，目前比较流行的做法就是通过Referer来进行判断和限制，Referer的解释说明如下：

> HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，服务器基此可以获得一些信息用于处理。——引用自百度百科

简单来说，假如我博客域名是`37.com`，我在nginx中设置，只允许Referer为`*.37.com`的来源请求图片，其它网站来的一律禁止。这里我们需要用到`ngx_http_referer_module`模块和`$invalid_referer`变量，请看下面进一步解释。

<!--more-->

**[ngx_http_referer_module模块](http://nginx.org/en/docs/http/ngx_http_referer_module.html)**

`ngx_http_referer_module`模块用于阻止对“Referer”头字段中具有无效值的请求访问站点。应该记住，使用适当的“Referer”字段值来构造请求非常容易，因此本模块的预期目的不是要彻底阻止此类请求，而是阻止常规浏览器发送的请求的大量流量。还应该考虑到，即使对于有效请求，常规浏览器也可能不发送“Referer”字段。

- **语法：**`valid_referers none | blocked | server_names | string ...;`
- **可用于：**`server,location`

可以看到valid_referers指令中存在一些参数，比如none|blocked，含义如下：

- none：请求标头中缺少“Referer”字段，也就是说Referer为空，浏览器直接访问的时候Referer一般为空。
- blocked： Referer”字段出现在请求标头中，但其值已被防火墙或代理服务器删除; 这些值是不以`“http://”` 或 `“https://”` 开头的字符串;
- server_names： 服务器名称，也就是域名列表。

**$invalid_referer变量**

我们设置valid_referers 指令后，会将其结果传递给一个变量`$invalid_referer`，其值为0或1，可以使用这个指令来实现防盗链功能，如果valid_referers列表中没有包含Referer头的值，$invalid_referer将被设置为1。

**设置防盗链白名单**

白名单就是只允许白名单内的域名访问，其余一律禁止。

```nginx
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico|webp)$ {
    valid_referers none blocked *.37.com *.37wan.com;
    if ($invalid_referer) {
        return 403;
    }
}
```

上面的配置含义是先用location匹配出需要的格式（图片和视频），然后用valid_referers指令设置允许的域名，其它域名没有包含在valid_referers列表中，$invalid_referer变量返回的值为1，最终返回403，禁止访问。以上就是防盗链白名单的设置。

**防盗链黑名单**

黑名单与白名单正好相反，就是只禁止黑名单中的域名请求，其余一律放行，相比白名单，黑名单的限制更加宽松。黑名单的设置方法也差不多。

```nginx
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico|webp)$ {
    valid_referers *.baidu.com;
    if ($invalid_referer = 0) {
        return 403;
    }
}
```

上面的配置中我们用`valid_referers`指令设置黑名单域名`*.baidu.com`，获取到指定的Referer头之后，`$invalid_referer`返回值为0，最终返回403，禁止百度的域名来访问。

#### 防止文件被下载

比如将网站数据库导出到站点根目录进行备份，很有可能也会被别人下载，从而导致数据丢失的风险。以下规则可以防止一些常规的文件被下载，可根据实际情况增减。

```nginx
location ~ \.(zip|rar|sql|bak|gz|7z)$ {
  return 444;
}
```

#### 屏蔽非常见蜘蛛（爬虫）

如果经常分析网站日志你会发现，一些奇怪的UA总是频繁的来访问网站，而这些UA对网站收录毫无意义，反而增加服务器压力，可以直接将其屏蔽。

```nginx
if ($http_user_agent ~* (SemrushBot|python|MJ12bot|AhrefsBot|AhrefsBot|hubspot|opensiteexplorer|leiki|webmeup)) {
     return 444;
}
```

#### 禁止某个目录执行脚本

比如网站上传目录，通常存放的都是静态文件，如果因程序验证不严谨被上传木马程序，导致网站被黑。以下规则请根据自身情况改为您自己的目录，需要禁止的脚本后缀也可以自行添加。

```nginx
#uploads|templets|data 这些目录禁止执行PHP
location ~* ^/(uploads|templets|data)/.*.(php|php5)$ {
    return 444;
}
```

#### 屏蔽某个IP或IP段

如果网站被恶意灌水或CC攻击，可从网站日志中分析特征IP，将其IP或IP段进行屏蔽。

```nginx
#屏蔽192.168.5.23这个IP
deny 192.168.5.23;
#屏蔽192.168.5.* 这个段
deny 192.168.5.0/24;
#允许指定ip
allow   219.232.244.234;
```

