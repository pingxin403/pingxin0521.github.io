---
title: Hexo博客-入门篇
date: 2019-03-11 14:50:22
tags:
 - 博客
 - Hexo
categories:
 - Hexo博客  
---

### 前言

开发环境：Ubuntu 18.04，网络正常情况下 （想使用Windows或者CentOS 7搭建的同学也可以参考）
前提：电脑装有Git和NodeJS
使用工具：NodeJS、Hexo

*本系列文章主旨:适合自己的才是最好的*

当然，对于不熟练使用Linux的同学，可以选择在Windows上进行写作，然后将`.md`文件上传到linux上进行测试（在Windows上老是出错）。

<!--more-->

### Git安装

```(shell)
//CentOS
sudo yum install git
//或
//Ubuntu
sudo apt install git
```

[git的使用](http://iissnan.com/progit/)

用于后续将博客上传到github上，（注：使用git前需要[配置](http://iissnan.com/progit/html/zh/ch1_5.html)）。

基本配置：

```(shell)
[hyp@promote ~]$ git config --global user.name "pingxin"
[hyp@promote ~]$ git config --global user.email "m13839441583@163.com"
```



### nodeJS安装

​	安装方法多种多样，有以下几种：

1. 去官网下载和自己系统匹配的已编译文件（Linux Binary，.tar.xz)

   [nodeJS英文网址](https://nodejs.org/en/download/)
   [nodeJS中文网址](http://nodejs.cn/download/)  

通过  `uname -a`  命令查看到我的Linux系统位数是64位（备注：x86_64表示64位系统， i686 i386表示32位系统）
```(shell)
[hyp@promote ~]$ sudo wget 	https://nodejs.org/dist/v10.15.2/node-v10.15.2-linux-x64.tar.xz
[hyp@promote ~]$ sudo tar -xf node-v10.15.2-linux-x64.tar.xz -C /usr/local/
[hyp@promote ~]$ cd /usr/local
[hyp@promote local]$ sudo mv node-v10.15.2-linux-x64 nodejs
[hyp@promote local]$ sudo ln -s /usr/local/bin/node /usr/bin/node
[hyp@promote local]$ sudo ln -s /usr/local/lib/node /usr/lib/node
[hyp@promote local]$ sudo ln -s /usr/local/bin/npm /usr/bin/npm
[hyp@promote local]$ sudo ln -s /usr/local/bin/node-waf /usr/bin/node-waf
[hyp@promote local]$ node -v
	v10.15.2
```

2. 使用源码安装（不推荐）  

3. CentOS 7下使用yum安装（使用epel源）
```(shell)
sudo yum install epel-release
sudo yum clean all
sudo yum makecache
yum  install nodejs  -y
sudo npm config set registry https://registry.npm.taobao.org #配置国内源
```
4. ubuntu 18.04安装
```(shell)
sudo apt install nodejs npm
sudo npm config set registry https://registry.npm.taobao.org #配置国内源
```


### 安装Hexo

[Hexo官网](https://hexo.io)

Hexo 是一个快速、简洁且高效的博客框架

Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页

安装：

```(shell)
npm install -g hexo-cli  #安装客户端
npm install -g hexo hexo-server  #安装服务端,用于本地测试
```

创建链接（**如果使用源码安装的nodejs**）：

```shell
[hyp@promote ~]$ sudo ln -s /usr/local/nodejs/lib/node_modules/hexo-cli/bin/hexo /usr/bin/hexo
[hyp@promote ~]$ ll /usr/bin/hexo
lrwxrwxrwx. 1 root root 52 3月   2 10:33 /usr/bin/hexo -> /usr/local/nodejs/lib/node_modules/hexo-cli/bin/hexo
```

使用`hexo version`查看是否成功



### 创建博客项目


- 确定项目目录
```(shell)
folder="/web/blog"
```

- 创建用户：
```
sudo useradd bloger -d ${floder}
sudo passwd bloger
```

- 初始化该目录,目录必须为空目录，该目录不得创建
```(shell)
sudo hexo init $folder
```


- 完成后cd到新建文件夹目录
```(shell)
su - bloger
cd $folder
```

- 安装本地npm库依赖：
```(shell)
npm install
```

项目结构如下：

```(html)
.
├── _config.yml  #网站的 配置 信息，可以在此配置大部分的参数
├── package.json	#应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除
├── scaffolds	#模版 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。
|				#Hexo的模板是指在新建的markdown文件中默认填充的内容。
├── source	#资源文件夹是存放用户资源的地方。
|	|		#除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。
|	|		#Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。
|   ├── _drafts
|   └── _posts
└── themes #主题文件夹。Hexo 会根据主题来生成静态页面
```

### 本地测试

1. 生成静态文件

   在`$folder`目录下，使用`sudo hexo generate`

   ![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nq8lv82j30m80a7aa1.jpg)

2. 开启本地服务

   使用`sudo hexo server`

   ![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nq8ifehj30m6027q2q.jpg)

   支持从本地使用[http://localhost:4000](http://localhost:4000)进行查看

3. 验证

   开启centos的4000端口，使其可以从真机进行访问

   > 开启防火墙端口
   >
   > 1. firewalld
   >
   >    ```shell
       [hyp@promote ~]$ sudo systemctl restart firewalld.service
       [hyp@promote ~]$ sudo firewall-cmd --zone=public --add-port=4000/tcp --permanent
       [hyp@promote ~]$ sudo systemctl restart firewalld.service
       [hyp@promote ~]$ sudo firewall-cmd --list-ports
       4000/tcp
       ```
   >
   > 2. iptables
   >
   >    直接编辑/etc/sysconfig/iptables文
   >
   >    1.编辑/etc/sysconfig/iptables文件：vi /etc/sysconfig/iptables
   >     加入内容并保存：-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 4000 -j ACCEPT
   >     2.重启服务：/etc/init.d/iptables restart
   >     3.查看端口是否开放：/sbin/iptables -L -n | grep 4000

   测试结果如下图：

   ![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nq8yg29j31ef0slgop.jpg)


   后面就会讲到如何配置，最后的项目可以从这里获得：[https://github.com/hanyunpeng0521/blog](https://github.com/hanyunpeng0521/blog)
