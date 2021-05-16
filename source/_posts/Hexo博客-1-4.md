---
title: Hexo博客（Ubuntu）-写作部署篇
date: 2019-03-13 12:23:58
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

​	本篇我们将学习将博客部署到github和coding，假设你已经学会了如何在CentOS上对网站进行基本配置和运行、学会了如何在Windows上进行编写markdown文档，然后我们进入下一环节：远程部署。

​	我们运行的服务只能在我们电脑所处的局域网内被进行访问，无法通过因特网获取，这样不能称之为个人博客，所以，我们将其部署当公开环境。

<!--more-->

### 生成文件

使用 Hexo 生成静态文件快速而且简单，在第一篇文章中就已经使用过，这里进行详细描写。

```(shell)
$ hexo generate
```

#### 监视文件变动

Hexo 能够监视文件变动并立即重新生成静态文件，在生成时会比对文件的 SHA1 checksum，只有变动的文件才会写入。

```(shell)
$ hexo generate --watch
```

#### 完成后部署



您可执行下列的其中一个命令，让 Hexo 在生成完毕后自动部署网站，两个命令的作用是相同的。

```(shell)
$ hexo generate --deploy
$ hexo deploy --generate
```

> 简写
>
> 上面两个命令可以简写为
> $ hexo g -d
> $ hexo d -g

### 服务器
#### [hexo-server](https://github.com/hexojs/hexo-server)

Hexo 3.0 把服务器独立成了个别模块，您必须先安装 [hexo-server](https://github.com/hexojs/hexo-server) 才能使用。

```(shell)
$ npm install hexo-server --save
```

安装完成后，输入以下命令以启动服务器，您的网站会在 `http://localhost:4000` 下启动。在服务器启动期间，Hexo 会监视文件变动并自动更新，您无须重启服务器（可能会遇到防火墙阻隔端口问题，请参考[第一篇](../Hexo博客-1)进行设置）。

```(shell)
$ hexo server
```

如果您想要更改端口，或是在执行时遇到了 `EADDRINUSE` 错误，可以在执行时使用 `-p` 选项指定其他端口，如下：

```(shell)
$ hexo server -p 5000
```

#### 静态模式

在静态模式下，服务器只处理 `public` 文件夹内的文件，而不会处理文件变动，在执行时，您应该先自行执行 `hexo generate`，此模式通常用于生产环境（production mode）下。

```(shell)
$ hexo server -s
```

#### 自定义 IP

服务器默认运行在 `0.0.0.0`，您可以覆盖默认的 IP 设置，如下：

```(shell)
$ hexo server -i 192.168.1.1
```

指定这个参数后，您就只能通过该IP才能访问站点。例如，对于一台使用无线网络的笔记本电脑，除了指向本机的`127.0.0.1`外，通常还有一个`192.168.*.*`的局域网IP，如果像上面那样使用`-i`参数，就不能用`127.0.0.1`来访问站点了。对于有公网IP的主机，如果您指定一个局域网IP作为`-i`参数的值，那么就无法通过公网来访问站点。

### 部署到github

​	[gitHub](https://github.com/)是一个面向开源及私有软件项目的托管平台，因为只支持git 作为唯一的版本库格式进行托管，故名gitHub。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olqhou4j30o00jejrp.jpg)

​	如果没有账号，首先在github官网上注册（Sign up），注册成功后登陆（Sign in）（[教程](https://blog.csdn.net/rj597306518/article/details/71307757)，第四步跳过）。有账号的同学直接输入邮箱和密码登录（Sign in）即可（不知道大家有没有看第一篇文章给出的[git教程](http://iissnan.com/progit/)）。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olpsw53j30hx0dogli.jpg)

​	这里假设你已经学会了如何在github创建管理仓库，接下来，请创建一个固定格式的仓库（Create a new repository）：`<用户名>.github.io`,用户名必须和你的用户名一模一样。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olqm5pxj30lx0h874i.jpg)

​	创建完成后，进入仓库（repository），记好该网址URL，我的是`https://github.com/hanyunpeng0521/hanyunpeng0521.github.io`，以及分支（Branch）：`master`，也可以使用别的分支，需要在Hexo配置文件中说明。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olqs3vtj30u90jk3yt.jpg)

如果是使用git类型的部署，需要安装`hexo-deployer-git`。

```(shell)
npm install hexo-deployer-git --save
```

假设你已经学会了将Windows上的markdown文章上传到CentOS上，我们现在更改配置文件`_config.yml`。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olq1ugpj30m306w0so.jpg)

Hexo 提供了快速方便的一键部署功能，让您只需一条命令就能将网站部署到服务器上，在项目目录下使用。

```(shell)
$ hexo deploy --debug
```

上传过程中要求输入github上的用户名和密码，输入后会上传到指定分支（branch）。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olq5iyyj30m607x3yj.jpg)

然后打开给定的repo，此处为：

`https://github.com/hanyunpeng0521/hanyunpeng0521.github.io.git`

打开查看文件是否有变化。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olrl3k2j30yu0j8mxk.jpg)

但是这样就完成了吗，怎么没有显示呢？其实，这只是一个普通的仓库，我们想要打开我们的博客是另外的一个链接：`https://<仓库名>`。

此处是：`https://hanyunpeng0521.github.io/`

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olsm0n9j317b0tsacr.jpg)

​	这样就把我们的博客搭载在github上，但是，github是国外的网站，网速有一定限制，接下来将我们的博客部署到coding上。

### 部署到coding

​	CODING 是一个高效的研发流程与管理平台。秉承 “让开发更简单” 的使命，致力于提升软件研发的管理效率。提供了任务协作、代码管理、GIT/SVN 代码托管、在线编辑器等一系列研发管理和支撑工具。

​	现在coding集成到腾讯云开发者平台，这是[官网](https://dev.tencent.com/user)。注册登录后，和github操作差不多，新建一个项目。首先在coding net 上创建一个仓库，仓库名为: <你的用户名>.coding.me。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olr0sg4j317u0nqdg7.jpg)



更改CentOS 7上的_config.yml文件，和上面类似，修改为以下样子。

repo为`https://git.dev.tencent.com/<用户名>/<仓库名>.git`。

此处为：`https://git.dev.tencent.com/pingxin0521/pingxin0521.coding.me.git`

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olq9dwfj30mt0dsdfv.jpg)

查看项目是否有变化，可以发现有新的commit。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olqwl1kj30o60fdaa9.jpg)

打开项目--》代码查看，发现项目中有新的代码。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olravduj31ar0fkaae.jpg)

然后开启项目--》Pages服务，如下：

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olr575yj30t60fljrm.jpg)

服务开启后就可以访问个人博客，网站链接格式如下：

`http://<项目名>/`

此处为：`http://pingxin0521.coding.me/`

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olrx837j30vr0sptak.jpg)



通过上面的配置，我们将个人博客搭建到github和coding上，可以让别人访问到我们的博客，但是，我们的个人博客链接不是很合适，最好的域名是能够简明并且能够知意。

### 更改域名

鸟哥推荐的[网站](www.noip.com)，可以注册使用免费的DNS域名（有一定限制），我们先注册用户，登陆后新建免费域名，然后绑定到coding上（coding是国内网站，速度较快）。

进入该网站主页，选择上面的管理DNS服务。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olt093vj31bm0r6n4a.jpg)

选择新建hostname。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olqdgeuj31b10dqjrh.jpg)

选择域名后缀，选择free的域名，建议里面有blog的域名，然后补全主机名的前面部分，主机类型选择“DNS Alias”，目标主机写“www.<项目名>”,这里是`www.pingxin0521.coding.me`。

> 注：域名有有效期限，必须在30天内更新，否则会失效。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olrpozoj319c0kdjrv.jpg)

但是仍然不行，需要更改coding的pages服务配置，设置其绑定新域名，填写你申请的域名，绑定后刷新一下，选择自定义域名勾选首选。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olrg2udj31bc0kk0t0.jpg)

此时可以通过新域名进行访问博客，在浏览器输入新域名。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g13olscajsj312l0qb765.jpg)



至此，部署篇完结，希望大家好好探索这些个未知的领域，做更好的自己。
