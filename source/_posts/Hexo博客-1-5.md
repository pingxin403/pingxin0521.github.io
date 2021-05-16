---
title: Hexo博客-优化篇
date: 2019-03-13 16:24:37
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

​	我们通过上篇的不懈努力，终于将博客部署到外网上，并且使用了免费的域名来访问我们的博客，这样我们就可以通过三种方式来访问我们的博客，分别是github、coding以及no-ip免费域名。

​	但是，仔细看看，发现我们的博客貌似有点丑，这篇我们就来对我们的博客进行美化吧。通过查阅Hexo官网得知，我们可以对项目进行添加插件和主题，插件用于增加附加功能，主题用来美化博客。

​	在 Hexo 中有两份主要的配置文件，其名称都是\_config.yml。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

<!--more-->

​	为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件。

### 主题

创建 Hexo 主题非常容易，您只要在 `themes` 文件夹内，新增一个任意名称的文件夹，并修改 站点配置文件内的 `theme` 设定，即可切换主题。一个主题可能会有以下的结构：

```(shell)
.
├── _config.yml
├── languages
├── layout
├── scripts
└── source
```

**_config.yml**

主题的配置文件。修改时会自动更新，无需重启服务器。

**languages**

语言文件夹。请参见 第三章国际化 (i18n)。

**layout**

布局文件夹。用于存放主题的模板文件，决定了网站内容的呈现方式，Hexo 内建 [Swig](http://paularmstrong.github.com/swig/) 模板引擎，您可以另外安装插件来获得 [EJS](https://github.com/hexojs/hexo-renderer-ejs)、[Haml](https://github.com/hexojs/hexo-renderer-haml) 或 [Jade](https://github.com/hexojs/hexo-renderer-jade) 支持，Hexo 根据模板文件的扩展名来决定所使用的模板引擎，例如：

```(shell)
layout.ejs   - 使用 EJS
layout.swig  - 使用 Swig
```

您可参考 第四章模板 以获得更多信息。

**scripts**

脚本文件夹。在启动时，Hexo 会载入此文件夹内的 JavaScript 文件，请参见 第六章插件以获得更多信息。

**source**

资源文件夹，除了模板以外的 Asset，例如 CSS、JavaScript 文件等，都应该放在这个文件夹中。文件或文件夹开头名称为 `_`（下划线线）或隐藏的文件会被忽略。

如果文件可以被渲染的话，会经过解析然后储存到 `public` 文件夹，否则会直接拷贝到 `public` 文件夹。

### 使用Material X主题

​	[Material X主题](https://xaoxuu.com/wiki/material-x/)是一个简约的卡片式Hexo博客主题。

​	Hexo 安装主题的方式非常简单，只需要将主题文件拷贝至站点目录的 `themes` 目录下， 然后修改下配置文件即可。具体到 Material X 来说，安装步骤如下。

**说明**

- 使用卡片设计元素以及交互动效。
- 高度可定制化的侧边栏小部件。
- 使用 fontawesome 5.6.3 免费版图标。
- 支持MobShare分享，自定义分享按钮的icon、图片。
- 支持3种评论系统：Disqus、来必力和Valine评论。（以及基于Valine的 Volantis ）
- 提供主题CDN，也可自定义CDN。
- 方便更换主题色、自定义字体、边距、圆角、阴影等视觉效果。
- 支持APlayer播放器，可以播放网易云、QQ音乐、虾米、酷狗平台以及其它服务器的音乐。
- 针对推荐文章插件进行了布局优化。
- 支持不蒜子阅读统计、百度分析、Google分析。
- 支持渲染MathJax数学公式，优化了渲染效果。
- 针对移动端布局进行了大量优化。
- 使用 import 字段方便导入各种标签到主题中。

#### 下载主题

​	有两种方式下载，一种是使用git下载最新版，另一种是下载稳定版本，一般来说，软件的稳定性要比多功能更重要，这里我们列出这两个方法。

**使用git**

在终端窗口下，定位到 Hexo 站点目录下。使用 `Git` checkout 代码：

```(shell)
$ cd your-hexo-site
$ git clone https://github.com/xaoxuu/hexo-theme-material-x themes/material-x
```

如果你熟悉 [Git](http://git-scm.com/)， 建议你使用 克隆最新版本 的方式，之后的更新可以通过 `git pull` 来快速更新， 而不用再次下载压缩包替换。

然后安装必要的依赖包

```
npm i -S hexo-generator-search hexo-generator-json-content hexo-renderer-less
```

#### 启用主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 **站点配置文件**， 找到 `theme` 字段，并将其值更改为 `material-x`。

启用 material-x 主题

```(yaml)
theme: material-x
```

到此，material-x 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 `hexo clean` 来清除 Hexo 的缓存。

#### 验证主题

首先启动 Hexo 本地站点，并开启调试模式（即加上 `--debug`），整个命令是 `hexo s --debug`。 在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：

```(shell)
INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
```

此时linux 上Hexo服务已经打开，如果防火墙未打开请参考，第一篇最后一章。

在主机上通过浏览器查看：`服务器主机ip:4000`。查看文末，如此便成功部署material-x。

#### 站点配置

**站点配置**

1. 多语言支持

   在博客根目录的配置文件中：

   ```yaml
   language:
   - zh-CN
   - en
   - zh-HK
   - zh-TW
   ```

   博客将按照给定的优先级显示语言，目前支持的语言只有这4种

2. 站点信息

   在博客根目录的配置文件中：

   ```yml
   # 网站标题
   title: my blog
   
   # 网站图标
   favicon: https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicon.ico
   
   # 作者头像，会出现在文章标题下方，不同于侧边栏的大头像
   avatar: https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/avatar/avatar.png
   ```

3. Import

   **字段：import** 可以在无需修改主题文件的情况下在head和body中添加各种标签。

   目前支持4个字段，`meta`和`link`对应head中的``和``标签。`script`可以在body末尾导入js文件。（如果还有什么需求可以在下方留言）

   例如在博客根目录的配置文件中：

   ```yml
   # 全局导入
   import:
     meta:
     - "<meta name='theme-color' content='#FFFFFF'>"
     - "<meta name='msapplication-TileColor' content='#1BC3FB'>"
     - "<meta name='msapplication-config' content='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicons/browserconfig.xml'>"
     link:
     - "<link rel='shortcut icon' type='image/x-icon' href='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicon.ico'>"
     - "<link rel='icon' type='image/x-icon' sizes='32x32' href='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicons/favicon-32x32.png'>"
     - "<link rel='apple-touch-icon' type='image/png' sizes='180x180' href='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicons/apple-touch-icon.png'>"
     - "<link rel='mask-icon' color='#1BC3FB' href='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicons/safari-pinned-tab.svg'>"
     - "<link rel='manifest' href='https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/favicon/favicons/site.webmanifest'>"
     # script:
   ```

更多请见Hexo官方文档：[Hexo配置](https://hexo.io/zh-cn/docs/configuration)

#### 主题配置

本节的配置信息写在主题的config.yml文件中

**页面默认布局**

这里的“布局”是指放置什么模块、顺序如何，具体的宽高等细节请在less中自定义。页面配置的Front-matter中也可以配置一部分布局。

```yml
layout:
  # 文章列表（主页、自定义的列表）布局
  posts:
    # 列表中每一篇文章的meta信息
    meta: [title, author, date, categories, top]
    # 列表类页面的侧边栏
    sidebar: [author, list, grid, category, tagcloud]
  # 文章页面布局
  article:
    # 文章页面主体元素，你也可以在页面的Front-matter中设置
    body: [article, comments]
    # 默认的meta信息，文章中没有配置则按照这里的配置来显示，设置为false则不显示
    # 其中，title只在header中有效，music和thumbnail无需在这里设置，文章中有则显示
    # 如果tags放置在meta.header中，那么在post列表中不显示（因为卡片下方已经有了）
    header: [title, author, date, categories, counter, top]
    footer: [updated, tags, share]
    # 文章页面的侧边栏
    sidebar: [author, toc, grid, category, tagcloud, list, related_posts]
  # 其他的页面布局暂时等于文章列表
```

**设置封面**

```yml
# page的封面
cover:
  scheme: search    # 后期将会提供多种封面方案
  # height: half      # full（默认值）: 首页封面占据整个第一屏幕，其他页面占半个屏幕高度， half: 所有页面都封面都只占半个屏幕高度
  title: "xaoxuu"
  # logo: assets/logo.png    # logo和title只显示一个，若同时设置，则只显示logo
  # search_placeholder: '搜索'
  # 主页封面菜单
  features:
    - name: 博文
      icon: fas fa-rss
      url: /
    - name: 项目
      icon: fas fa-code-branch
      url: projects/
    - name: 归档
      icon: fas fa-archive
      url: blog/archives/
      rel: nofollow
    - name: 关于
      icon: fas fa-info-circle
      url: about/
      rel: nofollow
```

**设置导航栏**

```yml
# 桌面端导航栏菜单
menu_desktop:
  - name: Cocoa Dev
    icon: fab fa-apple
    url: blog/categories/dev/cocoa/
  - name: Dev
    icon: fas fa-laptop-code
    url: blog/categories/dev/
  - name: Life
    icon: fas fa-coffee
    url: blog/categories/life/
    rel: nofollow

# 移动端导航菜单（从右上角的按钮点击展开）
menu_mobile:
  - name: 近期文章
    icon: fas fa-clock
    url: /
  - name: 文章归档
    icon: fas fa-archive
    url: blog/archives/
    rel: nofollow
  - name: 开源项目
    icon: fas fa-code-branch
    url: projects/
  - name: 歌单分享
    icon: fas fa-compact-disc
    url: music/
    rel: nofollow
  - name: 我的友链
    icon: fas fa-link
    url: friends/
    rel: nofollow
  - name: 主题文档
    icon: fas fa-book
    url: https://xaoxuu.com/wiki/material-x/
    rel: nofollow
  - name: 关于小站
    icon: fas fa-info-circle
    url: about/
    rel: nofollow
```

其中 icon 是 `fontawesome` 图标名，你要显示什么图标，去 [fontawesome.com](https://fontawesome.com/icons?d=gallery&m=free) 找免费版的就可以了。

**设置侧边栏**

1. 字段：widgets

   侧边栏小部件显示什么、显示多少个、顺序如何完全由博主给定的顺序决定。
   自定显示的方法详见【主题配置 -> 页面默认布局】以及【页面配置 -> 显示侧边栏】

   通用字段：

   如无特殊说明，所有的小部件都至少支持下面这些通用字段：

   | 字段   | 含义                     | 是否必须 | 值类型               | 默认值 |
   | :----- | :----------------------- | :------- | :------------------- | :----- |
   | widget | 表示这个小部件是什么类型 | 是       | 详见【widget枚举值】 | plain  |
   | enable | 是否启用                 | 否       | Bool                 | true   |
   | icon   | 左上角的图标             | 否       | String               | ‘’     |
   | title  | 标题                     | 否       | String               | ‘’     |
   | more   | 右上角的按钮             | 否       | 详见【more】         | -      |

   **字段：widget** 代表小部件的类型，取值有以下几种：

   | 取值     | 含义           |
   | :------- | :------------- |
   | author   | 博主信息小部件 |
   | category | 文章分类小部件 |
   | tagcloud | 标签云小部件   |
   | toc      | 文章目录小部件 |
   | music    | 音乐小部件     |
   | plain    | 纯文本小部件   |
   | list     | 列表小部件     |

   **字段：more** 小部件右上角的按钮

   | 字段    | 含义              | 是否必须 |
   | :------ | :---------------- | :------- |
   | icon    | 按钮图标          | 是       |
   | url     | 按钮的链接        | 是       |
   | animate | 按钮hover时的动画 | 否       |
   | target  | a标签的target属性 | 否       |
   | rel     | a标签的rel属性    | 否       |

   **字段：animate** 取值有以下几种：

   | 取值     | 含义           |
   | :------- | :------------- |
   | rotate90 | 顺时针旋转90度 |

2. 博主信息小部件

   博主信息小部件不支持上述的通用字段中的`icon`、`title`和`more`，以下只显示博主信息小部件特有字段：

   | 字段       | 含义     | 是否必须 | 值类型      | 默认值 |
   | :--------- | :------- | :------- | :---------- | :----- |
   | avatar     | 头像     | 否       | String      | ‘’     |
   | title      | 标题     | 否       | String      | ‘’     |
   | body       | 正文     | 否       | String      | ‘’     |
   | social     | 社交信息 | 否       | Bool        | false  |
   | jinrishici | 今日诗词 | 否       | Bool,String | false  |

   在侧边栏`author`小部件中找到或者新增`jinrishici`字段。

   | 取值   | 含义                               |
   | :----- | :--------------------------------- |
   | false  | 不加载，不显示                     |
   | true   | 显示，当请求失败时显示config.title |
   | String | 显示，当请求失败时显示String       |

   ```yml
   # 侧边栏小部件
   widgets:
     - widget: author
       avatar: https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/avatar/avatar.png
       jinrishici: 柳暗花明又一村
       social: true
   ```

3. 纯文本小部件

   | 字段   | 含义                         | 是否必须 | 值类型        | 默认值 |
   | :----- | :--------------------------- | :------- | :------------ | :----- |
   | widget | 表示这个小部件是什么类型     | 否       | 必须是’plain’ | plain  |
   | body   | 正文内容，支持markdown语法。 | 否       | markdown      | ‘’     |

4. 列表小部件

   | 字段   | 含义                     | 是否必须 | 值类型       | 默认值 |
   | :----- | :----------------------- | :------- | :----------- | :----- |
   | widget | 表示这个小部件是什么类型 | 是       | 必须是’list’ | plain  |
   | rows   | 每一行元素               | 否       | 详见【rows】 | -      |

   **rows：**

   | 字段   | 含义              | 是否必须           | 值类型 | 默认值 |
   | :----- | :---------------- | :----------------- | :----- | :----- |
   | name   | 名称              | 是                 | String | ‘’     |
   | img    | 方形图片          | 否，不设置时是空白 | String | ‘’     |
   | avatar | 圆形图片          | 否，不设置时是空白 | String | ‘’     |
   | icon   | 图标              | 否，不设置时是空白 | String | ‘’     |
   | url    | 链接              | 是                 | String | ‘’     |
   | desc   | 描述文字          | 否                 | String | ‘’     |
   | target | a标签的target属性 | 否                 | String | ‘’     |
   | rel    | a标签的rel属性    | 否                 | String | ‘’     |

   注意：一行的img、avatar和icon字段如果提供了多个，最多只会显示一个。

5. 文章分类小部件

   | 字段   | 含义                     | 是否必须 | 值类型           | 默认值 |
   | :----- | :----------------------- | :------- | :--------------- | :----- |
   | widget | 表示这个小部件是什么类型 | 是       | 必须是’category’ | plain  |

6. 标签云小部件

   | 字段   | 含义                     | 是否必须 | 值类型           | 默认值 |
   | :----- | :----------------------- | :------- | :--------------- | :----- |
   | widget | 表示这个小部件是什么类型 | 是       | 必须是’tagcloud’ | plain  |

7. 音乐小部件

   | 字段   | 含义                     | 是否必须 | 值类型        | 默认值 |
   | :----- | :----------------------- | :------- | :------------ | :----- |
   | widget | 表示这个小部件是什么类型 | 是       | 必须是’music’ | plain  |

8. TOC目录小部件

   没有特有字段，并且不支持通用字段中的`more`，因为其右上角的按钮具有特殊作用。

   ```yml
   layout:
     # 文章列表（主页、自定义的列表布局）布局
     posts:
       # 每一篇文章的meta信息
       meta: [title, author, date, categories, top]
       # 侧边栏
       sidebar: [author, grid, category, tagcloud, list]
     # 文章页面布局
     article:
       # 主体元素，你也可以在页面的Front-matter中设置
       body: [article, comments]
       # 默认的meta信息，文章中没有配置则按照这里的配置来显示，设置为false则不显示
       # 其中，title只在header中有效，music和thumbnail无需在这里设置，文章中有则显示
       # 如果tags放置在meta.header中，那么在post列表中不显示（因为卡片下方已经有了）
       header: [title, author, date, categories, top]
       footer: [updated, tags, share]
       # 侧边栏（文章）
       sidebar: [toc, related_posts]
     # 其他的页面布局暂时等于文章列表
   
   
   # 侧边栏小部件配置
   sidebar:
     - widget: author
       avatar: https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/avatar/avatar.png
       social: true
     - widget: toc
     - widget: grid
       icon: fas fa-map-signs
       title: 站内导航
       rows:
         - name: 近期文章
           icon: fas fa-clock
           url: /
         - name: 文章归档
           icon: fas fa-archive
           url: blog/archives/
           rel: nofollow
         - name: 项目Wiki
           icon: fas fa-landmark
           url: wiki/
         # - name: 歌单分享
         #   icon: fas fa-compact-disc
         #   url: music/
         #   rel: nofollow
         - name: 我的友链
           icon: fas fa-link
           url: friends/
           rel: nofollow
         - name: 主题文档
           icon: fas fa-book
           url: https://xaoxuu.com/wiki/material-x/
           rel: nofollow
         - name: 关于小站
           icon: fas fa-info-circle
           url: about/
           rel: nofollow
     - widget: category
       more:
         icon: fas fa-expand-arrows-alt
         url: blog/categories/
         rel: nofollow
     - widget: tagcloud
       icon: fas fa-tags
       more:
         icon: fas fa-expand-arrows-alt
         url: blog/tags/
         rel: nofollow
   ```

**社交信息**

```yml
# 页脚社交信息
social:
  - icon: fas fa-envelope
    url: mailto:me@xaoxuu.com
  - icon: fab fa-github
    url: https://github.com/xaoxuu
  - icon: fas fa-music
    url: https://music.163.com/#/user/home?id=63035382
```

这些社交按钮也会同时出现在侧边栏头像下方，可以在侧边栏的配置中设置不显示。

**分享按钮**

```yml
# 分享按钮
# 当id为qrcode时需要安装插件  npm i -S hexo-helper-qrcode
share:
  - id: qq
    name: QQ好友
    img: https://cdn.jsdelivr.net/gh/xaoxuu/assets@19.1.9/logo/128/qq.png
  - id: qzone
    name: QQ空间
    img: https://cdn.jsdelivr.net/gh/xaoxuu/assets@19.1.9/logo/128/qzone.png
  # - id: qrcode
  #   name: 微信
  #   img: https://cdn.jsdelivr.net/gh/xaoxuu/assets@19.1.9/logo/128/wechat.png
  - id: weibo
    name: 微博
    img: https://cdn.jsdelivr.net/gh/xaoxuu/assets@19.1.9/logo/128/weibo.png
  # - id: qrcode
  #   name: QRcode
  #   img: https://cdn.jsdelivr.net/gh/xaoxuu/assets@19.1.9/logo/128/qrcode.png

```

目前二维码分享在部分浏览器中无法使用。

**设置页脚**

```yml
footer: '这是页脚文字，支持**markdown**语法。'
```

#### 页面配置

本节的配置信息写在页面的Front-matter中

**布局模板**

| 取值     | 含义                             |
| :------- | :------------------------------- |
| page     | 独立页面                         |
| post     | 文章页面                         |
| category | 分类页面                         |
| tag      | 标签页面                         |
| links    | 友链页面                         |
| list     | 列表页面（类似于首页的文章列表） |

**Front-matter**

Front-matter 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量。【详见官方文档】

| 字段        | 含义                 | 值类型        | 默认值        |
| :---------- | :------------------- | :------------ | :------------ |
| layout      | 布局模版             | String        | -             |
| title       | 标题                 | String        | -             |
| date        | 创建时间             | Date          | 文件创建时间  |
| updated     | 更新日期             | Date          | 文件修改时间  |
| permalink   | 覆盖文章网址         | String        | -             |
| music       | 内部音乐控件         | 详见【music】 | -             |
| keywords    | 页面关键词           | String        | -             |
| description | 页面描述、摘要       | String        | -             |
| author      | 作者                 | String        | config.author |
| author_url  | 作者链接             | String        | config.url    |
| avatar      | 作者头像             | String        | config.avatar |
| cover       | 是否显示封面         | Bool          | true          |
| meta        | 文章或页面的meta信息 | Bool, Array   | theme.meta    |
| sidebar     | 页面侧边栏           | Bool, Array   | theme.sidebar |
| body        | 页面主体元素         | Array         | theme.body    |

layout=post时特有的字段：

| 字段          | 含义         | 值类型        | 默认值 |
| :------------ | :----------- | :------------ | :----- |
| categories    | 分类         | String, Array | -      |
| tag           | 标签         | String, Array | -      |
| toc           | 是否生成目录 | Bool          | true   |
| popular_posts | 显示相关文章 | Bool          | true   |
| mathjax       | 是否渲染公式 | Bool, String  | false  |
| top           | 是否置顶     | Bool          | false  |
| thumbnail     | 缩略图       | String        | false  |
| icons         | 图标         | Array         | []     |

icons字段设置的图标将会在归档页面文章标题后面显示，目的是标记出需要特别注意的文章，例如：

```markdown
---
icons: [fas fa-fire accent]
---
可以通过 red/blue/green/yellow/orange/theme/accent 来设置图标的颜色
theme为主题色: @theme_main
accent为链接高亮颜色: @color_text_highlight
```

可以设置多个图标：

```
---
icons: [fas fa-star yellow, fas fa-fire accent]
---
```

layout=links时特有的字段：

| 字段  | 含义 | 值类型        | 默认值 |
| :---- | :--- | :------------ | :----- |
| links | 友链 | 详见【links】 | -      |

> 更多请见Hexo官方文档：[Front-matter](https://hexo.io/zh-cn/docs/front-matter)

**独立页面**

1. 关于页面,创建`source/about/index.md`，内容为：

   ```
   ---
   layout: page
   title: 关于
   body: [article, grid, comments]
   valine:
     placeholder: 有什么想对我说的呢？
   sidebar: false
   ---
   ```

2. 分类页面,创建`source/categories/index.md`，内容为：

   ```
   ---
   layout: category
   index: true
   title: 所有分类
   ---
   ```

3. 标签页面,创建`source/tags/index.md`，内容为：

   ```
   ---
   layout: tag
   index: true
   title: 所有标签
   ---
   ```

4. 列表页面,创建`source/mylist/index.md`，内容为：

   ```
   ---
   layout: list
   type: mylist
   index: true
   ---
   ```

   结果就是筛选出所有文章中`type: mylist`的文章。

5. 友链页面,创建`source/friends/index.md`，内容为：

   ```yml
   ---
   layout: links     # 必须
   title: 我的朋友们   # 可选，这是友链页的标题
   links:
     - group: 技术大佬
       icon: fas fa-user-tie
       items:
       - name:     # 博客名
         avatar:   # 头像链接
         url:      # 博客链接
         backgroundColor: '#3E74C9' # 卡片背景颜色
         textColor: '#fff'  # 卡片文字颜色
         tags:     # 标签
         - 标签1
         - 标签2
     - group: 分组2
       icon: fas fa-user-tie
       items:
       - name:     # 博客名
         avatar:   # 头像链接
         url:      # 博客链接
         backgroundColor: '#3E74C9' # 卡片背景颜色
         textColor: '#fff'  # 卡片文字颜色
         tags:     # 标签
         - 标签1
         - 标签2
   ---
   
   这里可以写友链页面下方的文字备注，例如自己的友链规范、示例等。
   ```

6. 404页面,创建`source/404.md`，内容为：

   ```yml
   ---
   layout: page
   title: 404 Not Found
   body: [article, comments]
   meta:
     header: false
     footer: false
   sidebar: false
   valine:
     path: /404.html
     placeholder: 请留言告诉我您要访问哪个页面找不到了
   ---
   
   # <center>**404 Not Found**</center>
   
   <br>
   
   <center>**很抱歉，您访问的页面不存在**</center>
   <center>可能是输入地址有误或该地址已被删除</center>
   
   <br>
   <br>
   ```

**页面元素排列**

默认是文章+评论：

```
---
body: [article, comments]
---
```

如果你想把相关文章卡片显示在评论前，可以这样写：

```
---
body: [article, related_posts, comments]
---
```

**文章属性**

1. 文章置顶

   在Front-matter中设置以下值：

   ```
   top: true
   ```

   如果想自定义置顶标签的文字，可以直接设置为字符串，例如：

   ```
   top: 近期更新
   ```

2. 文章分类,多个分类有两种关系，一种是层级（等同于文件夹），一种是并列（等同于标签）。

   多级分类

   ```
   categories: [分类A, 分类B]
   ```

   或者

   ```
   categories:
     - 分类A
     - 分类B
   ```

   并列分类

   ```
   categories:
     - [分类A]
     - [分类B]
   ```

   多级+并列分类

   ```
   categories:
     - [分类A, 分类B]
     - [分类C, 分类D]
   ```

3. 文章BGM

   标题右边显示迷你音乐播放器，支持的字段有：`server`、`type`、`id`，使用APlayer播放器

   | 字段   | 含义                         | 是否必须 | 值类型               | 默认值      |
   | :----- | :--------------------------- | :------- | :------------------- | :---------- |
   | color  | 音乐控件的主题色             | 否       | HEX颜色值字符串      | -           |
   | volume | 默认音量                     | 否       | 0~1之间的浮点数      | 0.7         |
   | mode   | 播放模式                     | 否       | 枚举，详见【mode】   | circulation |
   | server | 服务器                       | 是       | 枚举，详见【server】 | -           |
   | type   | 表明id值是单曲、专辑还是歌单 | 是       | 枚举，详见【type】   | -           |
   | id     | 列表id                       | 是       | String               | ‘’          |

   **mode：**

   | 取值        | 含义     |
   | :---------- | :------- |
   | random      | 随机     |
   | single      | 单曲     |
   | order       | 列表顺序 |
   | circulation | 列表循环 |

   **server：**

   | 取值    | 含义   |
   | :------ | :----- |
   | netease | 网易云 |
   | tencent | QQ音乐 |
   | xiami   | 虾米   |
   | kugou   | 酷狗   |

   **type：**

   | 取值     | 含义     |
   | :------- | :------- |
   | song     | 单曲     |
   | album    | 专辑     |
   | playlist | 播放列表 |

   值从哪里获取请自行探索，有疑问可以去[APlayer](https://aplayer.js.org/)官网留言，我只保证主题能够使用这项服务，不附带第三方服务的使用教程。

   示例：

   ```
   ---
   music:
     enable: true      # true（文章内和文章列表都显示） internal（只在文章内显示）
     server: netease   # netease（网易云音乐）tencent（QQ音乐） xiami（虾米） kugou（酷狗）
     type: song        # song （单曲） album （专辑） playlist （歌单） search （搜索）
     id: 26664345      # 歌曲/专辑/歌单 ID
   ---
   ```

4. 文章摘要

   第一优先级是寻找正文中的`<!--more-->`，其前面的为摘要，可以显示在文章列表中，后面的是正文。如果没有，则寻找Front-matter中的description字段，以其为摘要。如果还没有，就没有摘要。

   如果使用`<!-- more -->`来实现摘要，则`<!-- more -->`前后一定要有空行，不然可能导致显示错位。

   ```
   这是摘要
   
   <!-- more -->
   
   这是正文
   ```

5. 是否开启渲染MathJax

   | 取值     | 含义                               |
   | :------- | :--------------------------------- |
   | false    | 不渲染，默认值                     |
   | true     | 渲染                               |
   | internal | 只在文章内部渲染，文章列表中不渲染 |

   ```
   ---
   mathjax: true
   ---
   ```

   > 如果公式仍无法正确渲染可以阅读@MicDZ大神的这篇文章：[《在material-x上使用KaTeX》](https://www.micdz.cn/article/katex-on-material-x/)。

6. 显示meta标签

   文章顶部和底部的日期、分类、更新日期、标签、分享等属于meta标签，顶部的为`header`，底部的为`footer`。

   在Front-matter找到或者新增`meta`：

   ```yml
   ---
   # 默认的meta信息，文章中没有配置则按照这里的配置来显示，设置为false则不显示
   # 其中，title只在header中有效，music和thumbnail无需在这里设置，文章中有则显示
   # 如果tags放置在meta.header中，那么在post列表中不显示（因为卡片下方已经有了）
   meta:
     header: [title, author, date, categories, counter, top]
     footer: [updated, tags, share]
   ---
   ```

   像404、关于页面就可以完全隐藏：

   ```
   ---
   meta:
     header: false
     footer: false
   ---
   ```

7. 设置文章作者

   由于支持多作者共同维护一个博客，所以可以设置单独一篇文章的作者：

   ```
   ---
   author:
     name: 作者
     avatar: https://img.vim-cn.com/a1/d53c11fb5d4fd69529bc805d385fe818feb3f6.png
     url: https://baidu.com
   ---
   ```

**是否要显示封面**

如果某个页面不需要封面，可以这样写：

```
---
cover: false
---
```

**引入外部文章**

利用`permalink`，搭配自定义的文章作者信息，你可以在文章列表中显示外部文章或者网址，例如：

```
---
layout:     post
date:       2017-07-05
title:      [转]如何搭建基于Hexo的独立博客
categories: [Dev, Hexo]
tags:
  - Hexo
author:
  name: xaoxuu
  avatar: https://cdn.jsdelivr.net/gh/xaoxuu/assets@master/avatar/avatar.png
  url: https://xaoxuu.com
permalink: https://xaoxuu.com/blog/2017-07-05-hexo-blog/
---

![](https://img.vim-cn.com/d9/a9af7dc49fc0af8ca3e6dd2450a2f7095a87db.png)

<!--more-->
```

**显示侧边栏**

通过自由设置边栏卡片来删减对应页面的冗余信息，提高有价值的信息在页面中的权重。

如果某个页面不需要侧边栏，可以这样写：

```
---
sidebar: []
---
```

某个页面想指定显示某几个侧边栏，就这样写:

```
---
sidebar: [grid, toc, tags] # 放置任何你想要显示的侧边栏部件
---
```



**使用CDN**

对于大部分将博客deploy到GitHub的用户来说，直接加载本地资源速度比较慢，可以使用jsdelivr为开源项目提供的CDN服务。

开启方法,在博客根目录的配置文件中：

```
use_cdn: true
```

如果你需要对样式进行DIY，可以只关闭style文件的CDN。

```yml
info:
  name: Material X
  docs: https://xaoxuu.com/wiki/material-x/
  cdn: # 要使用CDN，请在根目录的config文件中写上 use_cdn: true
    css:
      # style: https://cdn.jsdelivr.net/gh/xaoxuu/cdn-material-x@19.8/css/style.css
    js:
      app: https://cdn.jsdelivr.net/gh/xaoxuu/cdn-material-x@19.8/js/app.js
      search: https://cdn.jsdelivr.net/gh/xaoxuu/cdn-material-x@19.8/js/search.js
      volantis: https://cdn.jsdelivr.net/gh/xaoxuu/volantis@1.0.5/js/volantis.min.js

```

自定义CDN,如果你把对应的文件上传到自己的CDN服务器，可以把对应的链接改为自己的CDN链接。

**Fancybox图片预览**

将需要放大预览的图片用<fancybox\></fancybox\>包含起来。例如这个图是不能点开的：

例如这个图是不能点开的：
![img](https://img.vim-cn.com/52/a54815c02ce232f11f54b2c547c1337828833c.png)
而这个图是可以点开的：


[![img](https://img.vim-cn.com/52/a54815c02ce232f11f54b2c547c1337828833c.png)](https://img.vim-cn.com/52/a54815c02ce232f11f54b2c547c1337828833c.png)

源码：

```
例如这个图是不能点开的：
![](https://img.vim-cn.com/52/a54815c02ce232f11f54b2c547c1337828833c.png)
而这个图是可以点开的：
<fancybox>
![](https://img.vim-cn.com/52/a54815c02ce232f11f54b2c547c1337828833c.png)
</fancybox>
```

一行显示多图,将多个图片同时放在一对<fancybox\></fancybox\>中即可：

```
<fancybox>
![](https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/wiki/material-x/349a2633dbfb842ea62ff9d810ca9b4a8dbb33.png)
![](https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/wiki/material-x/9ae699a4d7c8a7fa5a3007fc37e0b61b5b55bd.png)
![](https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/wiki/material-x/d3f01cd009b11025b453c697173855649d01a0.png)
![](https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/wiki/material-x/7a9f2a9360f6c8a4da9e00f5e1e8500ddbb223.png)
![](https://cdn.jsdelivr.net/gh/xaoxuu/assets@18.12.27/wiki/material-x/fffd5955330a5d55c55401539da4bac77d5438.png)
</fancybox>
```

**Valine评论系统**官网： https://valine.js.org

在博客根目录的配置文件中：

```
leancloud:
  app_id: 你的appId
  app_key: 你的appKey
```

在主题的配置文件中：

```yml
valine:
  enable: true # 如果你想用Valine评论系统，请设置enable为true
  volantis: true # 是否启用volantis版本（禁止匿名，增加若干贴吧、QQ表情）
  # 还需要在根目录配置文件中添加下面这三行内容
  # leancloud:
  #   app_id: 你的appId
  #   app_key: 你的appKey
  guest_info: nick,mail,link #valine comment header info
  placeholder: 快来评论吧~ # valine comment input placeholder(like: Please leave your footprints )
  avatar: mp # gravatar style https://valine.js.org/avatar
  pageSize: 20 # comment list page size
  verify: false # valine verify code (true/false)
  notify: false # valine mail notify (true/false)
  lang: zh-cn
  highlight: false
```

其中，placeholder支持在Front-matter中设置。

在文章的Front-matter中：

```
---
valine:
  placeholder: 你觉得xxx怎么样呢？
---
```

也可以通过设置valine.path实现多个页面共用一个评论框。

在文章的Front-matter中：

```
---
valine:
  path: /wiki/material-x/
---
```

**分析与统计**

默认支持[不蒜子](http://busuanzi.ibruce.info/)的访问统计，可以添加百度统计和Google Analytics。

1. 百度统计

   | 源文件          | 相关字段              | 功能          |
   | :-------------- | :-------------------- | :------------ |
   | `./_config.yml` | `baidu_analytics_key` | 百度统计的key |

2. Google Analytics

   | 源文件          | 相关字段               | 功能                  |
   | :-------------- | :--------------------- | :-------------------- |
   | `./_config.yml` | `google_analytics_key` | Google Analytics的key |

3. CNZZ统计

   请参考ZYMIN网友的这篇教程：[《hexo+ejs+material x 添加CNZZ统计代码》](https://zymin.cn/arcticle/hexo+ejs+material.html)

4. 字数统计和阅读时长、网站运行时间

   请参考TRHX网友的这篇教程：[《Hexo 博客主题个性化》](https://itrhx.com/2018/08/27/A04-Hexo-blog-topic-personalization/)

**添加卡通人物**

我在逛别人博客的时候偶然发现右下角居然有一个萌萌的卡通人物，还能根据你鼠标位置摇头，瞬间被吸引到了，赶紧也给自己博客添加一个吧！[点击此处](https://github.com/EYHN/hexo-helper-live2d)进入该项目地址

输入如下命令获取 live2d ：

```
$ npm install --save hexo-helper-live2d
```

输入以下命令，下载相应的模型，将 packagename 更换成模型名称即可，更多模型选择请[点击此处](https://github.com/xiazeyu/live2d-widget-models)，各个模型的预览请[访问原作者的博客](https://huaji8.top/post/live2d-plugin-2.0/)

```
$ npm install packagename
```

打开站点目录下的 _config.yml 文件，添加如下代码：

```yml
live2d:
  enable: true
  scriptFrom: local
  model:
    use: live2d-widget-model-hijiki #模型选择
  display:
    position: right  #模型位置
    width: 150       #模型宽度
    height: 300      #模型高度
  mobile:
    show: false      #是否在手机端显示
```

**添加背景动态彩带效果**

样式一是鼠标点击后彩带自动更换样式，样式二是飘动的彩带,实现方法：在 \themes\material-x\layout\layout.ejs 文件的body前面添加如下代码：

```html
<!-- 样式一（鼠标点击更换样式） -->
<script src="https://g.joyinshare.com/hc/ribbon.min.js" type="text/javascript"></script>

<!-- 样式二（飘动的彩带） -->
<script src="https://g.joyinshare.com/hc/piao.js" type="text/javascript"></script>
```

**使用 Hexo-Git-Backup 插件备份**

由于 Hexo 博客是静态托管的，所有的原始数据都保存在本地，如果哪一天电脑坏了，或者是误删了本地数据，那就是叫天天不应叫地地不灵了，此时定时备份就显得比较重要了，常见的备份方法有：打包数据保存到U盘、云盘或者其他地方，但是早就有大神开发了备份插件：[hexo-git-backup](https://github.com/coneycode/hexo-git-backup) ，只需要一个命令就可以将所有数据包括主题文件备份到 github 了

安装

```
$ npm install hexo-git-backup --save
```

到 Hexo 博客根目录的 `_config.yml` 配置文件里添加以下配置

```yml
backup:
  type: git
  theme: material-x
  message: Back up my blog
  repository:
    github: git@github.com:hanyunpeng0521/hanyunpeng0521.github.io.git,backup
    coding: git@git.dev.tencent.com:pingxin0521/pingxin0521.coding.me.git,backup

```

参数解释：

- theme：你要备份的主题名称
- message：自定义提交信息
- repository：仓库名，注意仓库地址后面要添加一个分支名，比如我就创建了一个 backup 分支

最后使用以下命令备份你的博客：

```
$ hexo backup
```

或者使用以下简写命令也可以：

```
$ hexo b
```