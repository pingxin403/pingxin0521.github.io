---
title: Hexo博客-写作理论篇
date: 2019-03-11 16:50:22
tags:
 - 博客
 - Hexo
categories:
 - Hexo博客
---

### 前言

开发环境：Ubuntu 18.04，网络正常情况下
前提：电脑装有Git和NodeJS
使用工具：NodeJS、Hexo

*本系列文章主旨:适合自己的才是最好的*

<!--more-->

### 语法

该项目的主要文件有这几种：配置文件`.yml`、信息文件`.json`、文章文件`.md`。

上文讲到项目结构如下：

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
└── themes #主题文件夹。Hexo 会根据主题来生成静态页面，下一章再讲
```

#### `.yml`语法

​	每一个 YAML 文件都是从一个列表开始，列表中的每一项都是一个键值对, 通常它们被称为一个 “哈希” 或 “字典”， 所以, 我们需要知道如何在 YAML 中编写列表和字典。

下面立刻展示YAML最基本，最常用的一些使用格式：
 首先YAML中允许表示三种格式，分别是常量值，对象和数组
 例如：

```(yaml)
#即表示url属性值；
url: http://www.wolfcode.cn
#即表示server.host属性的值；
server:
    host: http://www.wolfcode.cn
#数组，即表示server为[a,b,c]
server:
    - 120.168.117.21
    - 120.168.117.22
    - 120.168.117.23
#常量
pi: 3.14   #定义一个数值3.14
hasChild: true  #定义一个boolean值
name: '你好YAML'   #定义一个字符串
```

**注释**

和properties相同，使用#作为注释，YAML中只有行注释。

**基本格式要求**

1. YAML大小写敏感；
2. 使用缩进代表层级关系；
3. 缩进只能使用空格，不能使用TAB，不要求空格个数，只需要相同层级左对齐（一般2个或4个空格）

**对象**

使用冒号代表，格式为key: value。冒号后面要加一个空格：

```(yaml)
key: value
```

可以使用缩进表示层级关系；

```(yaml)
key:
    child-key: value
    child-key2: value2
```

YAML中还支持流式(flow)语法表示对象，比如上面例子可以写为：

```(yaml)
key: {child-key: value, child-key2: value2}
```

较为复杂的对象格式，可以使用问号加一个空格代表一个复杂的key，配合一个冒号加一个空格代表一个value：

```(yaml)
?  
    - complexkey1
    - complexkey2
:
    - complexvalue1
    - complexvalue2
```

意思即对象的属性是一个数组[complexkey1,complexkey2]，对应的值也是一个数组[complexvalue1,complexvalue2]

**数组**

使用一个短横线加一个空格代表一个数组项：

```(yaml)
hobby:
    - Java
    - LOL
```

当然也可以有这样的写法：

```(yaml)
-
    - Java
    - LOL
```

可以简单理解为：[[Java,LOL]]
 一个相对复杂的例子：

```(yaml)
companies:
    -
        id: 1
        name: company1
        price: 200W
    -
        id: 2
        name: company2
        price: 500W
```

意思是companies属性是一个数组，每一个数组元素又是由id,name,price三个属性构成；
 数组也可以使用流式(flow)的方式表示：

```(yaml)
companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
```

**常量**

YAML中提供了多种常量结构，包括：整数，浮点数，字符串，NULL，日期，布尔，时间。下面使用一个例子来快速了解常量的基本使用：

```(yaml)
boolean:
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以
float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
null:
    nodeName: 'node'
    parent: ~  #使用~表示null
string:
    - 哈哈
    - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    #字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime:
    -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
```

参考：[YAML快速入门](https://www.jianshu.com/p/97222440cd08)

#### `.json`语法

JSON 语法是 JavaScript 对象表示法语法的子集。

- 数据在名称/值对中
- 数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

JSON 数据的书写格式是：名称/值对。

名称/值对包括字段名称（在双引号中），后面写一个冒号，然后是值：

```(json)
"firstName" : "John"
```

这很容易理解，等价于这条 JavaScript 语句：

```(javascript)
firstName = "John"
```

JSON 值可以是：

- 数字（整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组（在方括号中）
- 对象（在花括号中）
- null

JSON 对象在花括号中书写：

对象可以包含多个名称/值对：

```(json)
{ "firstName":"John" , "lastName":"Doe" }
```

这一点也容易理解，与这条 JavaScript 语句等价：

```(javascript)
firstName = "John"
lastName = "Doe"
```

JSON 数组在方括号中书写：

数组可包含多个对象：

```(json)
{
"employees": [
{ "firstName":"John" , "lastName":"Doe" },
{ "firstName":"Anna" , "lastName":"Smith" },
{ "firstName":"Peter" , "lastName":"Jones" }
]
}
```

在上面的例子中，对象 "employees" 是包含三个对象的数组。每个对象代表一条关于某人（有姓和名）的记录。

JSON 文件类型

- JSON 文件的文件类型是 ".json"
- JSON 文本的 MIME 类型是 "application/json"

#### `.md`语法

**什么是 Markdown**

> Markdown 是一种轻量级标记语言，创始人为约翰·格鲁伯（John Gruber）。它允许人们「使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档 」—— 维基百科>Markdown具有一系列衍生版本，用于扩展Markdown的功能（如表格、脚注、内嵌HTML等等），这些功能原初的Markdown尚不具备，它们能让Markdown转换成更多的格式，例如LaTeX，Docbook。Markdown增强版中比较有名的有Markdown Extra、MultiMarkdown、 Maruku等。这些衍生版本要么基于工具，如Pandoc；要么基于网站，如GitHub和Wikipedia，在语法上基本兼容，但在一些语法和渲染效果上有改动。如果你看不懂以上对 Markdown 的定义，那也无所谓。约翰·格鲁伯自己对Markdown的描述的重点也在于标记。

**标题**

在想要设置为标题的文字前面加#来表示
 一个#是一级标题，二个#是二级标题，以此类推。支持六级标题。

注：标准语法一般在#后跟个空格再写文字。

示例：

```(markdown)
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```

效果如下：

![](https://ws1.sinaimg.cn/large/006KyevZgy1g0xrpkjlg8j30ox0b174m.jpg)

**字体**

- 加粗

要加粗的文字左右分别用两个*号包起来

- 斜体

要倾斜的文字左右分别用一个*号包起来

- 斜体加粗

要倾斜和加粗的文字左右分别用三个*号包起来

- 删除线

要加删除线的文字左右分别用两个~~号包起来

示例：

```(markdown)
**这是加粗的文字**
*这是倾斜的文字*`
***这是斜体加粗的文字***
~~这是加删除线的文字~~
```

效果如下：

**这是加粗的文字**
 *这是倾斜的文字*
 **这是斜体加粗的文字**
 ~~这是加删除线的文字~~

------

**引用**

在引用的文字前加>即可。引用也可以嵌套，如加两个>>三个>>>
 n个...
 貌似可以一直加下去，但没神马卵用

示例：
```(markdown)
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容
```
效果如下：

> 这是引用的内容
>
> > 这是引用的内容
> >
> > > > > > > > > > 这是引用的内容

**分割线**

三个或者三个以上的 - 或者 * 都可以。

示例：

```(markdown)
---
----
***
```

效果如下：
 可以看到，显示效果是一样的。

------

------

------

------

**图片**

语法：

```(html)
![图片alt](图片地址 ''图片title'')

图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
也可以使用html进行显示
```

示例：

```(markdown)
![老婆](https://ws1.sinaimg.cn/large/006KyevZgy1g0wh93i8yrj3052052wem.jpg)
```

效果如下：

![老婆](https://ws1.sinaimg.cn/large/006KyevZgy1g0wh93i8yrj3052052wem.jpg)

**超链接**

语法：

```(markdown)
[超链接名](超链接地址 "超链接title")
title可加可不加
```

示例：

```(markdown)
[百度](http://baidu.com)
```

效果如下：

[百度](http://baidu.com)

注：Markdown本身语法不支持链接在新页面中打开，如果想要在新页面中打开的话可以用html语言的a标签代替。

```(html)
<a href="超链接地址" target="_blank">超链接名</a>
```

示例
<a href="http://baidu.com" target="\_blank">百度</a>

------

**列表**

*无序列表*

语法：
无序列表用 - + * 任何一种都可以

```(html)
- 列表内容
+ 列表内容
* 列表内容

注意：- + * 跟内容之间都要有一个空格
```

效果如下：

- 列表内容

- 列表内容

- 列表内容

- 有序列表


*数字加点*

```(markdown)
1.列表内容
2.列表内容
3.列表内容

注意：序号跟内容之间要有空格
```

效果如下：

1. 列表内容
2. 列表内容
3. 列表内容

*列表嵌套*

上一级和下一级之间敲三个空格即可

- 一级无序列表内容
  - 二级无序列表内容
  - 二级无序列表内容
  - 二级无序列表内容
- 一级无序列表内容
  1. 二级有序列表内容
  2. 二级有序列表内容
  3. 二级有序列表内容

1. 一级有序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
   - 二级无序列表内容
2. 一级有序列表内容
   1. 二级有序列表内容
   2. 二级有序列表内容
   3. 二级有序列表内容

------

**表格**

语法：

```(markdown)
表头|表头|表头
---|:--:|---:
内容|内容|内容
内容|内容|内容

第二行分割表头和内容。
- 有一个就行，为了对齐，多加了几个
文字默认居左
-两边加：表示文字居中
-右边加：表示文字居右
注：原生的语法两边都要用 | 包起来。此处省略
```

示例：

```
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
```

效果如下：

| 姓名 | 技能 | 排行 |
| ---- | ---- | ---- |
| 刘备 | 哭   | 大哥 |
| 关羽 | 打   | 二哥 |
| 张飞 | 骂   | 三弟 |

**代码**

语法：
 单行代码：代码之间分别用一个反引号包起来

```
    `代码内容`
```

代码块：代码之间分别用三个反引号包起来，且两边的反引号单独占一行

```
(```)
  代码...
  代码...
  代码...
(```)
```

> 注：为了防止转译，前后三个反引号处加了小括号，实际是没有的。这里只是用来演示，实际中去掉两边小括号即可。

示例：

单行代码

```(markdown)
`create database hero;`
```

代码块

```(markdown)
(```)
    function fun(){
         echo "这是一句非常牛逼的代码";
    }
    fun();
(```)
```

效果如下：

单行代码

```(markdown)
create database hero;
```

代码块

```(markdown)
function fun(){
  echo "这是一句非常牛逼的代码";
}
fun();
```

**流程图**

```(markdown)
​```flow
st=>start: 开始
op=>operation: My Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
&```
```

效果如下：

![](https://i.loli.net/2019/03/27/5c9afef6146a4.png)  

以上就是该项目所使用的主要文件的语法，其中markdown文件语法经常使用，希望大家多多掌握。

### 配置

您可以在 `_config.yml` 中修改大部分的配置，参考上述YAML语法，`：`与后面的配置之间要有空格。

**网站**

| 参数          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `title`       | 网站标题，更改为自己网站的标题，尽量简短明了                 |
| `subtitle`    | 网站副标题，标题的扩充，不要很长                             |
| `description` | 网站描述，简单描述自己的网站                                 |
| `author`      | 您的名字                                                     |
| `language`    | 网站使用的语言，中国大陆默认为`zh-CN`                        |
| `timezone`    | 网站时区。Hexo 默认使用您电脑的时区。[时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。比如说：`America/New_York`, `Japan`, 和 `UTC` 。 |

> 其中，`description`主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author`参数用于主题显示文章的作者。


**网址**

| 参数                 | 描述                                                         | 默认值                      |
| -------------------- | ------------------------------------------------------------ | --------------------------- |
| `url`                | 网址                                                         |                             |
| `root`               | 网站根目录                                                   |                             |
| `permalink`          | 文章的 [永久链接](https://hexo.io/zh-cn/docs/permalinks)格式 | `:year/:month/:day/:title/` |
| `permalink_defaults` | 永久链接中各部分的默认值                                     |                             |

> 网站存放在子目录
>
> 如果您的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`。

**目录**

一般不改变

| 参数           | 描述                                                         | 默认值           |
| -------------- | ------------------------------------------------------------ | ---------------- |
| `source_dir`   | 资源文件夹，这个文件夹用来存放内容。                         | `source`         |
| `public_dir`   | 公共文件夹，这个文件夹用于存放生成的站点文件。               | `public`         |
| `tag_dir`      | 标签文件夹                                                   | `tags`           |
| `archive_dir`  | 归档文件夹                                                   | `archives`       |
| `category_dir` | 分类文件夹                                                   | `categories`     |
| `code_dir`     | Include code 文件夹                                          | `downloads/code` |
| `i18n_dir`     | 国际化（i18n）文件夹                                         | `:lang`          |
| `skip_render`  | 跳过指定文件的渲染，您可使用 [glob 表达式](https://github.com/isaacs/node-glob)来匹配路径。 |                  |

> 提示
>
> 如果您刚刚开始接触Hexo，通常没有必要修改这一部分的值。

**文章**

| 参数                | 描述                                                         | 默认值    |
| ------------------- | ------------------------------------------------------------ | --------- |
| `new_post_name`     | 新文章的文件名称                                             | :title.md |
| `default_layout`    | 预设布局，                                                   | post      |
| `auto_spacing`      | 在中文和英文之间加入空格                                     | false     |
| `titlecase`         | 把标题转换为 title case                                      | false     |
| `external_link`     | 在新标签中打开链接                                           | true      |
| `filename_case`     | 把文件名称转换为 (1) 小写或 (2) 大写                         | 0         |
| `render_drafts`     | 显示草稿                                                     | false     |
| `post_asset_folder` | 启动 [Asset 文件夹](https://hexo.io/zh-cn/docs/asset-folders) | false     |
| `relative_link`     | 把链接改为与根目录的相对位址                                 | false     |
| `future`            | 显示未来的文章                                               | true      |
| `highlight`         | 代码块的设置                                                 |           |

> 相对地址
>
> 默认情况下，Hexo生成的超链接都是绝对地址。例如，如果您的网站域名为`example.com`,您有一篇文章名为`hello`，那么绝对链接可能像这样：`http://example.com/hello.html`，它是**绝对**于域名的。相对链接像这样：`/hello.html`，也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。通常情况下，建议使用绝对地址。

**分类 & 标签**

| 参数               | 描述     | 默认值          |
| ------------------ | -------- | --------------- |
| `default_category` | 默认分类 | `uncategorized` |
| `category_map`     | 分类别名 |                 |
| `tag_map`          | 标签别名 |                 |

**日期 / 时间格式**

Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

| 参数          | 描述     | 默认值       |
| ------------- | -------- | ------------ |
| `date_format` | 日期格式 | `YYYY-MM-DD` |
| `time_format` | 时间格式 | `H:mm:ss`    |

**分页**

| 参数             | 描述                                | 默认值 |
| ---------------- | ----------------------------------- | ------ |
| `per_page`       | 每页显示的文章量 (0 = 关闭分页功能) | `10`   |
| `pagination_dir` | 分页目录                            | `page` |

**扩展**

| 参数     | 描述                                |
| -------- | ----------------------------------- |
| `theme`  | 当前主题名称。值为`false`时禁用主题 |
| `deploy` | 部署部分的设置，后面会讲到          |

### 命令

使用hexo进行项目操作，需要掌握其常用命令。

1. 命令

命令格式：`hexo <操作> [folder]`

- init
  `$ hexo init [folder]`
  新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

- new
  `$ hexo new [layout] <title>`
  新建一篇文章。如果没有设置 layout 的话，默认使用_config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。

- generate
  `$ hexo generate`
  生成静态文件。

  选项	描述
  -d, --deploy	文件生成后立即部署网站
  -w, --watch	监视文件变动
  该命令可以简写为`$ hexo g`

- publish
  `$ hexo publish [layout]  <filename>`
  发表草稿。

- server
  `$ hexo server`
  启动服务器。默认情况下，访问网址为： http://localhost:4000/。

  选项	描述
  -p, --port	重设端口
  -s, --static	只使用静态文件
  -l, --log	启动日记记录，使用覆盖记录格式

- deploy
  `$ hexo deploy`
  部署网站。

  参数	描述
  -g, --generate	部署之前预先生成静态文件
  该命令可以简写为：

  `$ hexo d`

- render
  `$ hexo render <file1> [file2] ...`
  渲染文件。

  参数	描述
  -o, --output	设置输出路径

- migrate
  `$ hexo migrate <type>`
  从其他博客系统 迁移内容。

- clean
  `$ hexo clean`
  清除缓存文件 (db.json) 和已生成的静态文件 (public)。
  在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

- list
  `$ hexo list <type>`
  列出网站资料。

- version
  `$ hexo version`
  显示 Hexo 版本。

2. 选项

- 安全模式
  `$ hexo --safe`
  在安全模式下，不会载入插件和脚本。当您在安装新插件遭遇问题时，可以尝试以安全模式重新执行。
- 调试模式
  `$ hexo --debug`
  在终端中显示调试信息并记录到 debug.log。当您碰到问题时，可以尝试用调试模式重新执行一次，并 提交调试信息到 GitHub。
- 简洁模式
  `$ hexo --silent`
  隐藏终端信息。
- 自定义配置文件的路径
  `$ hexo --config custom.yml`
  自定义配置文件的路径，执行后将不再使用 _config.yml。
- 显示草稿
  `$ hexo --draft`
  显示 source/_drafts 文件夹中的草稿文章。
- 自定义 CWD
  `$ hexo --cwd /path/to/cwd`
  自定义当前工作目录（Current working directory）的路径
