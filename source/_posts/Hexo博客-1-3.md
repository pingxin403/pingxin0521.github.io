---
title: Hexo博客 - 写作工具篇
date: 2019-03-12 16:23:54
tags:
 - 博客
 - Hexo
categories:
 - Hexo博客
---



### 前言

开发环境：Ubuntu 18.04，网络正常情况下
前提：Ubuntu 18.04装有Git和NodeJS
使用工具：NodeJS、Hexo  、Typora、图床

*本系列文章主旨:适合自己的才是最好的*



当然，对于不熟练使用linux的同学，可以选择在Windows上进行写作，然后将`.md`文件上传到linux上进行测试（在Windows上老是出错）。


接下来推荐两个软件进行编写博客时使用的软件：Typora、Firefox浏览器

<!--more-->

**微博图床已有限制，请转到其他图床**

### Typora

[Typora](https://www.typora.io/)是一款超简洁的markdown编辑器，具有如下特点：

1. 完全免费，目前已支持中文
2. 跨平台，支持windows,mac,linux
3. 支持数学公式输入，图片插入
4. 极其简洁，无多余功能
5. 界面所见即所得

#### 安装


**Ubuntu**

```
# or run:
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
# install typora
sudo apt-get install typora
```

**其他**

从官网下载二进制包，根据CentOS的位数进行选择，可以通过`uname -a`查看：

```(shell)
[hyp@promote ~]$ uname -a
Linux promote.cache-dns.local 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

选择对应版本进行下载：

[Typora-linux-x64](https://typora.io/linux/Typora-linux-x64.tar.gz)  

 [Typora-linux-x86](https://typora.io/linux/Typora-linux-ia32.tar.gz)

下载：

```(shell)
[hyp@promote ~]$ wget https://typora.io/linux/Typora-linux-x64.tar.gz
--2019-03-11 10:34:20--  https://typora.io/linux/Typora-linux-x64.tar.gz
正在解析主机 typora.io (typora.io)... 104.25.139.36, 104.25.140.36, 2606:4700:20::6819:8b24, ...
正在连接 typora.io (typora.io)|104.25.139.36|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：69249255 (66M) [application/gzip]
正在保存至: “Typora-linux-x64.tar.gz”

100%[======================================>] 69,249,255  1.87MB/s 用时 31s

2019-03-11 10:34:52 (2.15 MB/s) - 已保存 “Typora-linux-x64.tar.gz” [69249255/69249255])
```

下载完成后进行解压：

```(shell)
[hyp@promote ~]$ sudo tar zxf Typora-linux-x64.tar.gz
[sudo] hyp 的密码：
[hyp@promote ~]$ ls
Typora-linux-x64  Typora-linux-x64.tar.gz  桌面
[hyp@promote ~]$ cd Typora-linux-x64
[hyp@promote Typora-linux-x64]$ ls
blink_image_resources_200_percent.pak  locales
content_resources_200_percent.pak      natives_blob.bin
content_shell.pak                      resources
icudtl.dat                             Typora
libffmpeg.so                           ui_resources_200_percent.pak
libnode.so                             v8_context_snapshot.bin
LICENSE                                version
LICENSES.chromium.html                 views_resources_200_percent.pak
```

其中Typora文件为可运行文件，使用`./Typora`运行会发现出错时，如果是以下情况：

```(shell)
[hyp@promote Typora-linux-x64]$ ./Typora
./Typora: error while loading shared libraries: libXss.so.1: cannot open shared object file: No such file or directory
```

可以尝试安装`libXScrnSaver`。

接着使用使用`./Typora`运行，如果还报错，请参考官网。

远程连接情况下无法运行：

```（shell）
[hyp@promote Typora-linux-x64]$ sudo ./Typora

(Typora:76898): Gtk-WARNING **: 10:38:51.131: cannot open display:
```

请在GUI环境下运行，可以打开对应的markdown文件进行编写：

![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nzuazp5j30yo0kq3zd.jpg)

进一步操作，将该文件夹加入`path`变量：

```(shell)
vim ~/.bashrc
#添加 export PATH=$PATH:<Typora位置>
```

更多请参考[官网](http://support.typora.io/Typora-on-Linux/)。

#### 使用

Typora的使用手册可以参考[此处](https://blog.csdn.net/SIMBA1949/article/details/79001226)，基本上和markdown语法想通，但是该编辑器还支持快捷键，可以更方便编写md文档。

**快捷键**

- 无序列表：输入-之后输入空格
- 有序列表：输入数字+“.”之后输入空格
- 任务列表：-[空格]空格 文字
- 标题：ctrl+数字
- 表格：ctrl+t
- 生成目录：[TOC]按回车
- 选中一整行：ctrl+l
- 选中单词：ctrl+d
- 选中相同格式的文字：ctrl+e
- 跳转到文章开头：ctrl+home
- 跳转到文章结尾：ctrl+end
- 搜索：ctrl+f
- 替换：ctrl+h
- 引用：输入>之后输入空格
- 代码块：ctrl+alt+f
- 加粗：ctrl+b
- 倾斜：ctrl+i
- 下划线：ctrl+u
- 删除线：alt+shift+5
- 插入图片：直接拖动到指定位置即可或者ctrl+shift+i
- 插入链接：ctrl+k

> 也可以选择使用atom

### 图床

接下来介绍的是图床工具，但是我找了半天，也没找到在CentOS上使用的图床，所以选择使用网页版的[图床](https://sm.ms/)。

![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nzur2ktj30h00mt429.jpg)

在编辑md文档时，插入图片、截图说明时，使用本地资源加载会很麻烦，推荐将图片上传到网上，生成链接，然后在md文档需要使用的地方引用链接，Typora中图片可以自动预览，上传后如下图所示，markdown可以使用html格式链接，也可以使用markdown格式链接，如果想调整大小，请使用html格式链接。


![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13nzur2ktj30h00mt429.jpg)



此处可以使用以下链接测试：

```(html)
![](https://ws3.sinaimg.cn/large/005BYqpgly1g10wx2rmztj30ji0d0di4.jpg)
<img src="https://ws3.sinaimg.cn/large/005BYqpgly1g10wx2rmztj30ji0d0di4.jpg" hight=200 width=300/>
```


#### 微博图床失效

这也意味着新浪免费图床时代已经结束了，建议尽量大家还是更换别的图床或者本地化吧。

将下面代码加在位置:`<主题文件夹>/layout/header.ejs`中。
```
<meta name="referrer" content="no-referrer" />
```
这样所有页面都会以no-referrer这样方法加载了，然后微博服务器就不知道是你的网站引用了他的图片了，也就没有外链限制的问题了。（暂时这么解决了，不知道微博会不会后续更加严格的限制外链。

#### Utool 插件

下载[utools](https://www.u.tools)，安装里面的图床插件

------

### 图片压缩

图片太大的话会影响用户浏览的感觉，适当的对图片进行压缩，可以提高网页加载的流畅性。

这里是使用一个叫做[tinypng](https://tinypng.com/)的网站。可以无损压缩.jpg和.png图片，上传压缩完毕后下载即可，快速便捷。

![](https://ws1.sinaimg.cn/mw690/006KyevZgy1g13o386tmjj311w0iradi.jpg)



*本系列文章主旨：Linux上进行部署测试，Windows上进行博文编写*

### 基本写作

​	你可以执行下列命令来创建一篇新文章（也可以按照模板自行编写，然后添加到../sources/<布局>/文件夹下）。
```
$ hexo new [layout] <title>
```
​	您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 \_config.yml 中的 default_layout 参数来指定默认布局。

#### 布局（Layout）

​	Hexo 有三种默认布局：post、page 和 draft，它们分别对应不同的路径，而您自定义的其他布局和 post 相同，都将储存到 source/\_posts 文件夹。

|布局|	路径|
| ---| ---|
|post|	source/_posts|
|page	|source|
|draft	|source/_drafts|

不要处理我的文章
如果你不想你的文章被处理，你可以将 Front-Matter 中的layout: 设为 false

> ##错误：FATAL can not read a block mapping entry; a multiline key may not be an implicit
> ##检查_config.yml内容，配置文件：\_config.yml 中 # Site #URL 属性设置后面的：需要有空格再填写内容.

#### 文件名称

​	Hexo 默认以标题做为文件名称，但您可编辑 new_post_name 参数来改变默认的文件名称，举例来说，设为 :year-:month-:day-:title.md 可让您更方便的通过日期来管理文章。

| 变量     | 描述                               |
| -------- | ---------------------------------- |
| :title   | 标题（小写，空格将会被替换为短杠） |
| :year    | 建立的年份，比如， 2015            |
| :month   | 建立的月份（有前导零），比如， 04  |
| :i_month | 建立的月份（无前导零），比如， 4   |
| :day     | 建立的日期（有前导零），比如， 07  |
| :i_day   | 建立的日期（无前导零），比如， 7   |

#### 草稿

​	刚刚提到了 Hexo 的一种特殊布局：draft，这种布局在建立时会被保存到 source/\_drafts文件夹，您可通过 publish 命令将草稿移动到 source/\_posts 文件夹，该命令的使用方式与 new 十分类似，您也可在命令中指定 layout 来指定布局。
```
$ hexo publish [layout] <title>
```
草稿默认不会显示在页面中，您可在执行时加上 --draft 参数，或是把 render_drafts 参数设为 true 来预览草稿。

#### 模版（Scaffold）

​	在新建文章时，Hexo 会根据 scaffolds 文件夹内相对应的文件来建立文件，例如：
```
$ hexo new photo "My Gallery"
```
在执行这行指令时，Hexo 会尝试在 scaffolds 文件夹中寻找 photo.md，并根据其内容建立文章，以下是您可以在模版中使用的变量：

| 变量   | 描述         |
| ------ | ------------ |
| layout | 布局         |
| title  | 标题         |
| date   | 文件建立日期 |

#### Front-matter

Front-matter 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```(markdown)
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数         | 描述                 | 默认值       |
| ------------ | -------------------- | ------------ |
| `layout`     | 布局                 |              |
| `title`      | 标题                 |              |
| `date`       | 建立日期             | 文件建立日期 |
| `updated`    | 更新日期             | 文件更新日期 |
| `comments`   | 开启文章的评论功能   | true         |
| `tags`       | 标签（不适用于分页） |              |
| `categories` | 分类（不适用于分页） |              |
| `permalink`  | 覆盖文章网址         |              |

#### 分类和标签

​	只有文章支持分类和标签，您可以在 Front-matter 中设置。在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 Foo, Bar 不等于 Bar, Foo；而标签没有顺序和层次。

```(markdown)
categories:
- Diary
tags:
- PS3
- Games
```

**分类方法的分歧**
	如果您有过使用WordPress的经验，就很容易误解Hexo的分类方式。WordPress支持对一篇文章设置多个分类，而且这些分类可以是同级的，也可以是父子分类。但是Hexo不支持指定多个同级分类。下面的指定方法：

```(markdown)
categories:
- Diary
- Life
```

​	会使分类Life成为Diary的子分类，而不是并列分类。因此，有必要为您的文章选择尽可能准确的分类

#### JSON Front-matter

​	除了 YAML 外，你也可以使用 JSON 来编写 Front-matter，只要将 --- 代换成 ;;; 即可。

```(json)
;;;
"title": "Hello World",
"date": "2013/7/13 20:46:25"
;;;
```

### 标签插件（Tag Plugins）

标签插件和 Front-matter 中的标签不同，它们是用于在文章中快速插入特定内容的插件。
​	参考：https://hexo.io/zh-cn/docs/tag-plugins

可以不用记忆，使用Typora编辑器可以快捷添加。

#### 引用块

在文章中插入引言，可包含作者、来源和标题。

- 别号： quote

```(markdown)
{% blockquote [author[, source]] [link] [source_link_title] %}content{% endblockquote %}
```

- 代码块
  在文章中插入代码。

  ```(markdown)
  {% codeblock [title] [lang:language] [url] [link text] %}
  code snippet
  {% endcodeblock %}
  ```

- 反引号代码块
  另一种形式的代码块，不同的是它使用三个反引号来包裹。
```(markdown)
(```)
[language] [title] [url] [link text] code snippet
​(```)
```

- Pull Quote
  在文章中插入 Pull quote。
```(markdown)
{% pullquote [class] %}
content
{% endpullquote %}
```

- jsFiddle
  在文章中嵌入 jsFiddle。
```(markdown)
{% jsfiddle shorttag [tabs] [skin] [width] [height] %}
```

- Image
  在文章中插入指定大小的图片。
```(markdown)
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
```

- 引用文章
  引用其他文章的链接。
```(markdown)
{% post_path slug %}
{% post_link slug [title] %}
```

- 引用资源
  引用文章的资源。
```(markdown)
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

### 资源文件夹

**资源**（Asset）代表 `source` 文件夹中除了文章以外的所有文件，例如图片、CSS、JS 文件等。比方说，如果你的Hexo项目中只有少量图片，那最简单的方法就是将它们放在 `source/images` 文件夹中。然后通过类似于  `![](/images/image.jpg)`  的方法访问它们。

#### 文章资源文件夹

​	对于那些想要更有规律地提供图片和其他资源以及想要将他们的资源分布在各个文章上的人来说，Hexo也提供了更组织化的方式来管理资源。这个稍微有些复杂但是管理资源非常方便的功能可以通过将 `_config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。

```(yaml)
post_asset_folder: true
```

​	当资源文件管理功能打开后，Hexo将会在你每一次通过 `hexo new [layout] <title>` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个 markdown 文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，你可以通过相对路径来引用它们，这样你就得到了一个更简单而且方便得多的工作流。

#### 相对路径引用的标签插件

​	通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确。在Hexo 2时代，社区创建了很多插件来解决这个问题。但是，随着Hexo 3 的发布，许多新的标签插件被加入到了核心代码中。这使得你可以更简单地在文章中引用你的资源。

```(markdown)
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

​	比如说：当你打开文章资源文件夹功能后，你把一个 `example.jpg`图片放在了你的资源文件夹中，如果通过使用相对路径的常规 markdown 语法 `![](/example.jpg)` ，它将 *不会* 出现在首页上。（但是它会在文章中按你期待的方式工作）

正确的引用图片方式是使用下列的标签插件而不是 markdown ：

```(markdown)
{% asset_img example.jpg This is an example image %}
```

通过这种方式，图片将会同时出现在文章和主页以及归档页中。

#### 插入本地图片

本地图片统一放在source/images文件夹中
例如: source/images/test.jpg

```(markdown)
![](/images/test.jpg)
<img src="/images/test.jpg">
```

#### 插入网络图片

```(markdown)
![](网络图片地址)
<img src="网络图片地址">
```

#### 插入音视频

推荐将音视频上传到YouTube、BiliBili、网易云音乐等视频网站，然后通过视频网站提供的分享链接代码，来将视频插入到文章里，这样可以更好的节约博客的空间，和减少出错的概率

#### 插入网址

```(markdown)
[平心官网](https://hanyunpeng.github.io)
<a href="https://hanyunpeng.github.io">平心官网</a>
```

### 数据文件

​	有时您可能需要在主题中使用某些资料，而这些资料并不在文章内，并且是需要重复使用的，那么您可以考虑使用 Hexo 3.0 新增的「数据文件」功能。此功能会载入 `source/_data` 内的 YAML 或 JSON 文件，如此一来您便能在网站中复用这些文件了。

举例来说，在 `source/_data` 文件夹中新建 `menu.yml` 文件：

```(yaml)
Home: /
Gallery: /gallery/
Archives: /archives/
```

您就能在模板中使用这些资料：

```(markdown)
<% for (var link in site.data.menu) { %>
  <a href="<%= site.data.menu[link] %>"> <%= link %> </a>
<% } %>
```

渲染结果如下 :

```(html)
<a href="/"> Home </a>
<a href="/gallery/"> Gallery </a>
<a href="/archives/"> Archives </a>
```

### 延伸

- [主题](https://hexo.io/zh-cn/docs/themes)
- [所有变量](https://hexo.io/zh-cn/docs/variables)
- [辅助函数](https://hexo.io/zh-cn/docs/helpers)
- [国际化](https://hexo.io/zh-cn/docs/internationalization)
- [插件](https://hexo.io/zh-cn/docs/plugins)
