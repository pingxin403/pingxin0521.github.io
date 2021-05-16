---
title: Html 再进一步
date: 2019-04-25 23:18:59
tags:
 - 前端
 - Html
categories:
 - 前端
 - Html
---

### 常用标签

#### 表格标签 table

<!--more-->

```
<table></table> 用来定义表格，
<tr></tr> 用来定义行，嵌套在 <table></table> 中
<td></td> 用来定义行中的单元格，嵌套在 <tr></tr> 中
<tr></tr> 中只能嵌套<td></td> , 但 <td></td> 相当于一个容器，可以嵌套任意元素
```

表格相关的属性如下：

| 属性名称    | 含义                                                 | 属性取值              |
| ----------- | ---------------------------------------------------- | --------------------- |
| border      | 表格的边框。默认 border="0",即无边框                 | 像素值                |
| cellspacing | 单元格与单元格边框之间的间距。  默认 cellspacing="2" | 像素值                |
| cellpadding | 单元格内容与单元格边框的间距。  默认 cellpadding="1" | 像素值                |
| width       | 表格的宽度                                           | 像素值                |
| height      | 表格的高度                                           | 像素值                |
| align       | 表格在页面中的水平对齐方式                           | left 、center 、right |
| caption     | 标题                                                 | 文本                  |
| colspan     | 从当前列向后 横跨几列, 应用于td、th                  | 数字                  |
| rowspan     | 从当前行向下 竖跨几行, 应用于td、th                  | 数字                  |

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格</title>
</head>
<body>
<table cellspacing="3" cellpadding="2" border="1" align="left">
    <tr>
        <td>第一行第1列</td>
        <td>第一行第2列</td>
        <td>第一行第3列</td>
    </tr>
    <tr>
        <td>第二行第1列</td>
        <td>第二行第2列</td>
        <td>第二行第3列</td>
    </tr>
</table>
</body>
</html>
```

##### 表头标签

```
表头一般位于表格的第一行或者第一列。
表头标签为 <th></th>，在显示的时候默认会显示为加粗的效果
增加表头时，使用 <th></th> 替代对应位置的 <td></td>即可
```

下图即是设置了表头的表格。

![2.png](https://i.loli.net/2019/05/14/5cda7f79ac00451737.png)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表头</title>
</head>
<body>
<table border="1" cellspacing="1" cellpadding="10" align="center">
    <caption>caption标签是啥？标题？</caption>
    <tr>
        <th>属性</th>
        <th>含义</th>
        <th colspan="2">取值</th>
    </tr>
    <tr>
        <th>border</th>
        <td>单元格边框</td>
        <td>像素值，默认0</td>
        <td rowspan="3">rowspan从当前单元格向下跨三行</td>
    </tr>
    <tr>
        <th>cellspacing</th>
        <td>单元格与单元格边框的间距</td>
        <td>像素值，默认2</td>
    </tr>
    <tr>
        <th>cellpadding</th>
        <td>单元格内容与单元格边框的间距</td>
        <td>像素值，默认1</td>
    </tr>
</table>
</body>
</html>
```

##### 表格结构(了解)

使用表格时，可以将表格分为头部、主体、页脚（页脚有兼容问题）

```
<thead></thead> 用来定义头部。必须位于 <table></table> 中，一般包含网页的logo和导航等头部信息。

<tbody></tbody> 用来定义表格的主体，一般包含网页中除头部和底部之外的其他内容。
```

![3.png](https://i.loli.net/2019/05/14/5cda7fb9990f651391.png)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>thead+tbody</title>
</head>
<body>
<table cellspacing="2" cellpadding="10" align="center" border="1">
    <thead>
    <tr>
        <th>属性名称</th>
        <th>含义</th>
        <th>取值</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>colspan</td>
        <td>向右横跨几列</td>
        <td>数值</td>
    </tr>
    <tr>
        <td>rowspan</td>
        <td>向下竖跨几行</td>
        <td>数值</td>
    </tr>

    </tbody>
</table>
</body>
</html>
```

##### 表格标题标签 caption

```
<caption></caption> 标签用来定义标题的标签
必须写在 <table></table> 中，和 <thead></thead>平级
```

相关代码可以参考  `表头标签` 的代码。

##### 单元格合并

```
适用于 <td></td>、<th></th>
colspan 跨列合并（水平合并）
rowspan 跨行合并（垂直合并）
```

相关代码可以参考  `表头标签` 的代码。

#### 表单标签

html 中一个完整的表单通常由 表单元素、提示信息、表单域（即多个表单的父容器）三部分组成。

![4.png](https://i.loli.net/2019/05/14/5cda81175fe0c15107.png)

##### imput 输入标签

```
<input/> 为单标签（自闭合标签）

type 是其基本属性，用来控制输入的类型
```

> input 、br、hr 、img、 base 都是单标签

> 

| 属性      | 取值     | 含义                                   |
| --------- | -------- | -------------------------------------- |
| type      | text     | **单行**文本输入框（不换行的）         |
| type      | password | 密码输入框                             |
| type      | radio    | 单选框（配合name 可以实现单选效果）    |
| type      | checkbox | 复选框                                 |
| type      | button   | 普通按钮                               |
| type      | submit   | 提交按钮                               |
| type      | reset    | 重置按钮                               |
| type      | image    | 图像形式的提交按钮                     |
| type      | file     | 文件域, 点击之后打开文件选择器         |
| name      | 任意文本 | 控件名称 , name 相同则表示是同一组数据 |
| value     | 任意文本 | 默认文本值                             |
| size      | 正整数   | 显示大小                               |
| checked   | checked  | 默认是否被选中                         |
| maxlength | 正整数   | 控制输入的最大字符数量                 |

多个 radio 使用相同的 name ，则表示这是一组数据，这样可以实现单选效果。如果不加 name 多个 radio 可同时被选中

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>input标签</title>
</head>
<body>
    用户名：<input type="text" maxlength="15"/>
    <br/>

    <!--次数密码中间的空格使用了是全角输入法，全角输入法中，空格占一个汉字的大小-->
    密　码：<input type="password"/>
    <br/>

    <!--使用name 限定了一组内容，从而实现单选-->
    性　别：
    <input type="radio" name="sex" checked="checked"/> 男
    <input type="radio" name="sex"/> 女
    <br/>

    爱　好：
    <input type="checkbox" name="hobby"/> 看电影
    <input type="checkbox" name="hobby"/> 读书
    <br/>

    <input type="button" value="普通按钮"/>
    <br/>
    <input type="submit" value="提交按钮"/>
    <br/>
    <input type="reset" value="重置按钮"/>
    <br/>
     <input type="image" 
    src="https://i.loli.net/2019/05/14/5cda83ea711a141951.jpeg"/>   
     <br/>

    请选择文件：<input type="file"/>

</body>
</html>
```

##### label 标签(理解)

- label 标签为 input 标签定义标注/标签
- 用来绑定一个表单元素，当点击 label 标签的时候，被绑定的 表单元素就会获取焦点
- 通过 for 属性，可以绑定 label 和 input ; 或者直接用lable 标签将input 包裹起来

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>label的使用</title>
</head>
<body>
    <!--label 中直接包裹 input,可以实现绑定-->
    <label>点击此处文本，用户名输入框会获取焦点 <br> 用户名：<input type="text"/></label>
    <br/>

    <hr/>
    <!--使用 label 的 for 属性绑定input-->
    <label for="#two">点击此处文本，密码输入框会获取焦点</label>
    <br/>
    用户名：<input type="text"/>
    <br/>
    密　码：<input type="text" id="#two"/>
</body>
</html>
```

##### textarea 文本域标签

`<textarea></textarea>`用来做大量文本的输入，支持多行

有 cols 、rows 属性。cols 限制每行中所输入的文本字数，rows 限制最大行数。（这两个属性通常不被使用，更多使用会使用CSS样式做相关控制）

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>textarea标签</title>
</head>
<body>
    请输入评论内容：
    <br/>
    <textarea ></textarea>
</body>
</html>

```

##### 下拉菜单 <select\></select\>

```
<select></select> 用来定义下拉菜单

<option></option> 用来定义下拉菜单选项

<select></select> 中至少包含一对 <option></option>
```

在 option 中定义了属性 selected="selected" 之后，表示该项被默认选中

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>select标签</title>
</head>
<body>
    设置家乡
    <select >
        <option>选择省份</option>
        <option>山东</option>
        <option>内蒙古</option>
        <option>黑龙江</option>
        <option>山西</option>
    </select>
    <select>
        <option>济南</option>
        <option selected="selected">临沂</option>
        <option>聊城</option>
        <option>莱芜</option>
        <option>青岛</option>
    </select>
</body>
</html>

```

##### 表单域 <form\></form\>

该标签用来定义表单域，以实现用户信息的收集和传递，form 中的内容通常都会被提交到服务器。
基本语法格式： 

```
 <form action="url地址" method="提交方式" name="表单名称">
     ...各种表单控件...
 </form>
```

常用属性有action、method、name 

- action : 指定接收并处理表单信息的服务器url地址
- method : 表单数据的提交方式。分为 post / get
- name : 指定表单名称，用来区分页面中的多个表单

每个表单都应该有自己的表单域
使用form 包裹之后点击提交按钮才有提交的动作

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>form表单域</title>
</head>
<body>
    <!--使用 form 包裹之后，提交按钮和图片按钮点击时就有效果了-->
    <form action="https://www.baidu.com" method="post" name="userInfo">
        用户名：<input type="text" maxlength="15"/>
        <br/>

        <!--次数密码中间的空格使用了是全角输入法，全角输入法中，空格占一个汉字的大小-->
        密　码：<input type="password"/>
        <br/>

        <!--使用name 限定了一组内容，从而实现单选-->
        性　别：
        <input type="radio" name="sex" checked="checked"/> 男
        <input type="radio" name="sex"/> 女
        <br/>

        爱　好：
        <input type="checkbox" name="hobby"/> 看电影
        <input type="checkbox" name="hobby"/> 读书
        <br/>

        <input type="button" value="普通按钮"/>
        <br/>
        <input type="submit" value="提交按钮"/>
        <br/>
        <input type="reset" value="重置按钮"/>
        <br/>
        <input type="image" src="../image/imgButton.png"/>
        <br/>

        请选择文件：<input type="file"/>

    </form>
</body>
</html>

```

### HTML 框架

通过使用框架，你可以在同一个浏览器窗口中显示不止一个页面。

**iframe语法:**

```
<iframe src="URL"></iframe>
```

该URL指向不同的网页。

height 和 width 属性用来定义iframe标签的高度与宽度。属性默认以像素为单位, 但是你可以指定其按比例显示 (如："80%")。

frameborder 属性用于定义iframe表示是否显示边框。设置属性值为 "0" 移除iframe的边框

iframe可以显示一个目标链接的页面

```
<iframe src="demo_iframe.htm" name="iframe_a"></iframe>
<p><a href="http://www.runoob.com" target="iframe_a">RUNOOB.COM</a></p>
```

### HTML5新标签及新特性

HTML5 是 W3C 与 WHATWG 合作的结果,WHATWG 指 Web Hypertext Application Technology Working Group。

WHATWG 致力于 web 表单和应用程序，而 W3C 专注于 XHTML 2.0。在 2006 年，双方决定进行合作，来创建一个新版本的 HTML。

HTML5 中的一些有趣的新特性：

- 用于绘画的 canvas 元素
- 用于媒介回放的 video 和 audio 元素
- 对本地离线存储的更好的支持
- 新的特殊内容元素，比如 article、footer、header、nav、section
- 新的表单控件，比如 calendar、date、time、email、url、search

最新版本的 Safari、Chrome、Firefox 以及 Opera 支持某些 HTML5 特性。Internet Explorer 9 将支持某些 HTML5 特性。

IE9 以下版本浏览器兼容HTML5的方法，使用本站的静态资源的html5shiv包：

```

<!--[if lt IE 9]>
    <script src="http://cdn.static.runoob.com/libs/html5shiv/3.7/html5shiv.min.js"></script>
<![endif]-->
```

载入后，初始化新标签的CSS：

```
/*html5*/
article,aside,dialog,footer,header,section,nav,figure,menu{display:block}
```



#### 文档类型设定

HTML , 对应早期的 HTML4.0.1, 其基本结构如下：

```
 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
 <html lang="en">
     <head>
         <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
         <title>Document</title>
     </head>
     <body>
     </body>
 </html>
```

XHTML ，其基本结构如下：

```
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
          "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
  <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
  <head>
      <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
      <title>Document</title>
  </head>
  <body>
  
  </body>
  </html>
```

HTML5 ，其基本格式如下

```
  <!doctype html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport"
            content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>Document</title>
  </head>
  <body>
  
  </body>
  </html>

```

在webStorm 中，如果想查看上述三种类型的基本格式，可以按如下步骤：new > file > 输入文件名为 xxx.html > 分别输入 html:4s  / html:xt / html:5 然后回车即可
如果想查看某个页面使用了两种 HTML，可以 右击，然后选择查看网页源码 ，然后查看<!Doctype > 中的信息即可

> **注意：**对于中文网页需要使用 **<meta charset="utf-8">** 声明编码，否则会出现乱码。

#### HTML5 的改进

- 新元素
- 新属性
- 完全支持 CSS3 
- Video 和 Audio
- 2D/3D 制图
- 本地存储
- 本地 SQL 数据
- Web 应用

#### HTML5 使用 CSS3

- 新选择器
- 新属性
- 动画
- 2D/3D 转换
- 圆角
- 阴影效果
- 可下载的字体

#### 已移除元素

以下的 HTML 4.01 元素在HTML5中已经被删除:

- `<acronym>` 
- `<applet>` 
- `<basefont>` 
- `<big>` 
- `<center>` 
- `<dir>` 
- `<font>` 
- `<frame>` 
- `<frameset>` 
- `<noframes>` 
- `<strike>` 

#### 字符设定

- XHTML、HTML中设置字符集时使用：
  `<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">`
- HTML5 中设置字符集时使用：
  `<meta charset="UTF-8">`

#### 常用新标签

[w3school 中文网站](http://w3school.com.cn/)



| 标签     | 作用                                                 |
| -------- | ---------------------------------------------------- |
| header   | 定义文档的页眉                                       |
| nav      | 定义导航链接部分                                     |
| footer   | 定义文档或者节的页脚/底部                            |
| article  | 定义文章                                             |
| section  | 定义文档中的节（section/段落）                       |
| aside    | 定义其所处内容之外的内容/侧边                        |
| datalist | 定义选项列表，与input 配合使用该标签，两者通过id关联 |
| fieldset | 可将表单内的相关元素打包/分组, 与legend 搭配使用     |

##### datalist 示例

datalist 中包裹 option
datalist 与 input 关联后，input 就具备的 select 的效果，同时具有了输入联想功能。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>inputlist</title>
</head>
<body>
    <input type="text" value="请输入姓名" list="names">
    <datalist id="names">
        <option>张三</option>
        <option>李四</option>
        <option>王五</option>
    </datalist>

</body>
</html>
```

##### fieldset 示例

fieldset 默认宽度满屏

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>fieldset</title>
</head>
<body>
    <fieldset>
        <legend>用户登录</legend>
        用户名:<input type="text"/>
        <br/>
        密　码:<input type="password"/>
    </fieldset>
</body>
</html>
```

#### 新增的input type属性值

这些新增的类型，更加细化的限定了输入内容，如果输入格式不对，在提交的时候会自动给出相应提示
更多新增type 参考 w3school

| 类型     | 示例                      | 含义                 |
| -------- | ------------------------- | -------------------- |
| email    | `<input type="email">`    | 输入邮箱格式         |
| tel      | `<input type="tel">`      | 输入手机号           |
| url      | `<input type="url">`      | 输入url              |
| number   | `<input type="number">`   | 输入数字             |
| search   | `<input type="search">`   | 搜索框（体现语义化） |
| range    | `<input type="range">`    | 自由拖动的滑块       |
| time     | `<input type="time">`     | 输入小时 分钟        |
| date     | `<input type="date">`     | 输入年 月 日         |
| datetime | `<input type="datetime">` | 输入 日期 时间       |
| month    | `<input type="month">`    | 输入月年             |
| week     | `<input type="week">`     | 输入星期 年          |
| color    | `<input type="color">`    | 调出调色板           |

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>newInputType</title>
</head>
<body>
    <form action="http://www.baidu.com">
        email：<input type="email"/>
        <br/>
        tel：<input type="tel"/>
        <br/>
        url：<input type="url"/>
        <br/>
        number：<input type="number"/>
        <br/>
        search：<input type="search"/>
        <br/>
        range：<input type="range"/>
        <br/>
        time：<input type="time"/>
        <br/>
        date：<input type="date"/>
        <br/>
        datetime：<input type="datetime-local"/>
        <br/>
        month：<input type="month"/>
        <br/>
        week：<input type="week"/>
        <br/>
        color：<input type="color"/>
        <br/>
        <input type="submit"/>
    </form>
</body>
</html>
```

#### 新增的input 属性

| 属性         | 示例                                              | 含义                                                         |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------ |
| placeholder  | `<input type="text" placeholder="请输入用户名"/>` | 提示文本，参考 android TextView 的 hintText                  |
| autofocus    | `<input type="text" autofocus>`                   | 自动获取焦点                                                 |
| multiple     | `<input type="file" multiple>`                    | 多文件上传                                                   |
| autocomplete | `<input type="text">`                             | 自动完成，取值 on、 off。 on表示记录已经输入的值供下次自动完成。 此外，必须有提交按钮，必须设置name 属性，然后该属性才会生效 |
| required     | `<input type="text" required>`                    | 必填项                                                       |
| accesskey    | `<input type="text" accesskey="s">`               | 定义让控件获取焦点的快捷键，快捷键采用`alt + 字母`的形式     |

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>newInputAttr</title>
</head>
<body>
    <form action="form.html">

        <input type="text" placeholder="请输入用户名"/>
        <br/>
        <br/>
        <!--可以简化为 <input type="text" autofocus/>  -->
        <input type="text" autofocus="autofocus" placeholder="自动获取焦点"/>
        <br/>
        <br/>
        <input type="file" multiple/>
        <br/>
        <br/>
        <input type="text" autocomplete name="identify" placeholder="下次自动补足输入内容"/>
        <br/>
        <br/>
        <!--在火狐浏览器中，如果没有输入内容，点击输入框外部区域，边框会变红-->
        <input type="text" required placeholder="这是必填项"/>
        <br/>
        <br/>
        <input type="text" accesskey="s" placeholder="获取焦点的快捷键为 alt+s"/>
        <br/>
        <br/>
        <input type="submit"/>
    </form>
</body>
</html>
```

#### 综合案例

效果图

![6.png](https://i.loli.net/2019/05/14/5cda8a7d727f183751.png)

代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>test1</title>
</head>
<body>
    <form action="">
        <fieldset>
            <legend>学生档案</legend>
            <label>姓　　名：<input type="text" placeholder="请输入学生姓名"/></label> <br/>
            <label>手 机 号：<input type="tel" placeholder="请输入学生手机号"/></label><br/>
            <label>邮　　箱：<input type="email"/></label><br/>
            <label>所属学院：<input type="text" list="academy"/></label>
            <datalist id="academy">
                <option >JAVA学院</option>
                <option >前端学院</option>
                <option >PHP学院</option>
            </datalist><br/>
            <label>出生日期：<input type="date"/></label><br/>
            <label>语文成绩：<input type="number" max="100" min="0" value="0"/></label><br/>
            <label>数学成绩: <meter max="100" min="0" low="59" value="10"></meter></label><br/>
            <label>英语成绩: <meter max="100" min="0" low="59" value="90"></meter></label><br/>
            <input type="submit"><br/>
            <input type="reset">

        </fieldset>
    </form>
</body>
</html>

```

### 多媒体标签

![](https://i.loli.net/2019/05/14/5cdae1894339298667.png)

##### （1）embed

用来插入各种多媒体，格式可以是 Midi、Wav、 AIFF 、AU 、Mp3等

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>embed</title>
</head>
<body>
<embed src="http://player.video.iqiyi.com/44cb2ab93ef163fea5a206e52da7c390/0/0/v_19rqyv6lfo.swf-albumId=1268727400-tvId=1268727400-isPurchase=0-cnId=3"
       allowFullScreen="true" quality="high" width="480" height="350" align="middle"
       allowScriptAccess="always" type="application/x-shockwave-flash"></embed>
</body>
</html>
```

> 上面示例代码中，embed 节点中的内容是直接从 爱奇艺 网站下复制的。做法是：`找到一个视频 > 分享 > 点击向下的箭头（即 更多）> 复制 html 代码` 即可

##### (2) audio

html5 通过 `<audio></audio>` 标签实现音频播放

audio 开始和结束标签之间可以嵌入文本，当浏览器不支持该标签时，该文本可以作为提示语。

audio 在不同浏览器中的兼容情况如下：

![7.png](https://i.loli.net/2019/05/14/5cda8df6ca83187076.png)

常用属性如下：

![](https://i.loli.net/2019/05/15/5cdb98ae7206b26340.png)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>audio</title>
</head>
<body>
    <!--使用方式1-->
    <audio src="../assets/audio/皇后大道东.mp3" autoplay="autoplay" controls="controls" loop="loop">
        提示语：您的浏览器不支持audio标签
    </audio>

    <br/>

    <!--使用方式2: 通过 source 定义三种音频格式，从而达到不同浏览器上都能播放的情况-->
    <audio loop controls preload="auto">
        <source src="../assets/audio/皇后大道东.mp3">
        <source src="../assets/audio/皇后大道东.ogg">
        <source src="../assets/audio/皇后大道东.wav">
        提示语：您的浏览器不支持audio标签
    </audio>
    
 
</body>
</html>

```

> 注意，如果 歌曲比较大，那么加载的过程会比较长！！！但是，只要设置了 autoplay ，加载完成后必然会自动播放

##### video

- html5中使用`<video></video>` 来实现视频的播放
- video 目前支持三种视频格式：ogg、mp4、webM
- video 在各浏览器上的兼容情况如下：

![8.png](https://i.loli.net/2019/05/14/5cda8df98178d65254.png)

常用属性如下：

![](https://i.loli.net/2019/05/15/5cdb9853c221237388.png)

```
<video width="320" height="240" controls>
<source src="movie.mp4" type="video/mp4">
<source src="movie.ogg" type="video/ogg">
<source src="movie.webm" type="video/webm">
<object data="movie.mp4" width="320" height="240">
<embed src="movie.swf" width="320" height="240">
</object>
</video>


```

### Canvas

HTML5 的 canvas 元素使用 JavaScript 在网页上绘制图像。

画布是一个矩形区域，您可以控制其每一像素。

canvas 拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。

创建：

向 HTML5 页面添加 canvas 元素。

规定元素的 id、宽度和高度：

```
<canvas id="myCanvas" width="200" height="100"></canvas>
```

通过 JavaScript 来绘制：

```html
<script type="text/javascript">
var c=document.getElementById("myCanvas");//使用 id 来寻找 canvas 元素
var cxt=c.getContext("2d");//创建 context 对象,内建的 HTML5 对象，
    //拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。
cxt.fillStyle="#FF0000";
cxt.fillRect(0,0,150,75);
</script>
```





