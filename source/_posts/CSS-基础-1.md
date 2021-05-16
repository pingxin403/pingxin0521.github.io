---
title: CSS 入坑
date: 2019-04-25 12:18:59
tags:
 - 前端
 - CSS
categories:
 - 前端
 - CSS
---

### 概述

- CSS 指层叠样式表 (*C*ascading *S*tyle *S*heets)
- 样式定义*如何显示* HTML 元素
- 样式通常存储在*样式表*中
- 把样式添加到 HTML 4.0 中，是为了*解决内容与表现分离的问题*
- *外部样式表*可以极大提高工作效率
- 外部样式表通常存储在 *CSS 文件*中
- 多个样式定义可*层叠*为一

<!--more-->

#### 优点

##### 样式解决了一个普遍的问题

HTML 标签原本被设计为用于定义文档内容。通过使用 `<h1>、<p>、<table>` 这样的标签，HTML 的初衷是表达“这是标题”、“这是段落”、“这是表格”之类的信息。同时文档布局由浏览器来完成，而不使用任何的格式化标签。

由于两种主要的浏览器（Netscape 和 Internet Explorer）不断地将新的 HTML 标签和属性（比如字体标签和颜色属性）添加到 HTML 规范中，创建文档内容清晰地独立于文档表现层的站点变得越来越困难。

为了解决这个问题，万维网联盟（W3C），这个非营利的标准化联盟，肩负起了 HTML 标准化的使命，并在 HTML 4.0 之外创造出样式（Style）。

所有的主流浏览器均支持层叠样式表。

##### 样式表极大地提高了工作效率

样式表定义如何显示 HTML 元素，就像 HTML 3.2 的字体标签和颜色属性所起的作用那样。样式通常保存在外部的 .css 文件中。通过仅仅编辑一个简单的 CSS 文档，外部样式表使你有能力同时改变站点中所有页面的布局和外观。

由于允许同时控制多重页面的样式和布局，CSS 可以称得上 WEB 设计领域的一个突破。作为网站开发者，你能够为每个 HTML 元素定义样式，并将之应用于你希望的任意多的页面中。如需进行全局的更新，只需简单地改变样式，然后网站中的所有元素均会自动地更新。

##### 多重样式将层叠为一个

样式表允许以多种方式规定样式信息。样式可以规定在单个的 HTML 元素中，在 HTML 页的头元素中，或在一个外部的 CSS 文件中。甚至可以在同一个 HTML 文档内部引用多个外部样式表。

#### 层叠次序

**当同一个 HTML 元素被不止一个样式定义时，会使用哪个样式呢？**

一般而言，所有的样式会根据下面的规则层叠于一个新的虚拟样式表中，其中数字 4 拥有最高的优先权。

1. 浏览器缺省设置
2. 外部样式表
3. 内部样式表（位于 `<head>` 标签内部）
4. 内联样式（在 HTML 元素内部）

因此，内联样式（在 HTML 元素内部）拥有最高的优先权，这意味着它将优先于以下的样式声明：<head> 标签中的样式声明，外部样式表中的样式声明，或者浏览器中的样式声明（缺省值）。

### 语法

 CSS的注释，作用和HTML注释类似，只不过它必须编写在style标签中，或者是css文件中

```html
<style type="text/css">

/*
CSS的注释，作用和HTML注释类似，只不过它必须编写在style标签中，或者是css文件中
*/
</style>
```

CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明。

```
selector {declaration1; declaration2; ... declarationN }
```

选择器通常是您需要改变样式的 HTML 元素。

选择器：通过选择器可以选中页面中指定的元素，并且将声明块中的样式应用到选择器对应的元素上

声明块：声明块紧跟在选择器的后边，使用一对{}括起来，声明块中实际上就是一组一组的名值对结构，这一组一组的名值对我们称为声明，在一个声明块中可以写多个声明，多个声明之间使用`;`隔开，声明的样式名和样式值之间使用`:`来连接

每条声明由一个属性和一个值组成。

属性（property）是您希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开。

```
selector {
property: value;
}
```

下面这行代码的作用是将 h1 元素内的文字颜色定义为红色，同时将字体大小设置为 14 像素。

在这个例子中，h1 是选择器，color 和 font-size 是属性，red 和 14px 是值。

```
h1 {
color:red;
font-size:14px;
}
```

![1.gif](https://i.loli.net/2019/05/15/5cdb9aa77e9d351641.gif)

> 提示：请使用花括号来包围声明。

#### 值的不同写法和单位

除了英文单词 red，我们还可以使用十六进制的颜色值 #ff0000：

```
p { color: #ff0000; }
```

为了节约字节，我们可以使用 CSS 的缩写形式：

```
p { color: #f00; }
```

我们还可以通过两种方法使用 RGB 值：

```
p { color: rgb(255,0,0); }
p { color: rgb(100%,0%,0%); }
```

请注意，当使用 RGB 百分比时，即使当值为 0 时也要写百分比符号。但是在其他的情况下就不需要这么做了。比如说，当尺寸为 0 像素时，0 之后不需要使用 px 单位，因为 0 就是 0，无论单位是什么。

提示：如果值为若干单词，则要给值加引号：

```
p {font-family: "sans serif";}
```

#### 多重声明：

提示：如果要定义不止一个声明，则需要用分号将每个声明分开。下面的例子展示出如何定义一个红色文字的居中段落。最后一条规则是不需要加分号的，因为分号在英语中是一个分隔符号，不是结束符号。然而，大多数有经验的设计师会在每条声明的末尾都加上分号，这么做的好处是，当你从现有的规则中增减声明时，会尽可能地减少出错的可能性。就像这样：

```
p {text-align:center; color:red;}	
```

你应该在每行只描述一个属性，这样可以增强样式定义的可读性，就像这样：

```
p {
  text-align: center;
  color: black;
  font-family: arial;
}
```

#### 空格和大小写

大多数样式表包含不止一条规则，而大多数规则包含不止一个声明。多重声明和空格的使用使得样式表更容易被编辑：

```
body {
  color: #000;
  background: #fff;
  margin: 0;
  padding: 0;
  font-family: Georgia, Palatino, serif;
  }
```

是否包含空格不会影响 CSS 在浏览器的工作效果，同样，与 XHTML 不同，CSS 对大小写不敏感。不过存在一个例外：如果涉及到与 HTML 文档一起工作的话，class 和 id 名称对大小写是敏感的。



### 引用CSS

引入层叠样式表的方法包括：

1. 行内式：也称内联式，直接在标记的style属性中设定CSS样式。（最不常用）

   ```
   <span style="font-size:14px;">
       <div id="demo" 
       style="position:absoulte;left: 20px;top:50px;width:300px;
       height: 50px;border: solid 1px red;padding-left: 20px;
       color:gray;font-size: 20px;line-height: 50px;text-align: center;">
       我是行内式写法
       </div>
   </span>
   ```

2. 嵌入式：将CSS样式集中写在网页的`<head></head>`标签对的`<style></style>`标签对中；

   对于用CSS比较少的网页，可以先这样写。后面若是增加很多，还是写在外部CSS文件中比较好。

   例如我这段当鼠标移上时放大图片的代码。

   ```
   <!DOCTYPE html>
   <html>
    <head>
           <meta charset="UTF-8">
           <title>图片放大</title>
           <style type="text/css">
               #div1 {
                   width: 200px;
                   height: 200px;
                   margin: 50px auto;
               }
               #div1 img {
                   cursor: pointer;
                   transition: all 0.6s;
               }
               #div1 img:hover {
                   transform: scale(1.4);
               }
           </style>
       </head>
   <body>
       <div id="div1">
           <img src="http://jingyile.gotoip2.com/wp-content/uploads/2018/05/33cb7ef344e3f9b5-1.jpg" />
       </div>
   </body>
   </html>
   ```
   
3. 链接式：跟第4个的导入式都称外部式或者外联式，使用link引用外部CSS文件

   格式：（一般写在head标签内，写在 body里面好像也可以...）

   ```
   <link rel="stylesheet" href="Style.css" type="text/css" />
   ```

   其中的rel="stylesheet"  type="text/css" 是固定写法不可修改！

   这应该是使用最广泛的也是最常见的一种写法了。

   使用link来引用外部的css的优势

   - 有利于SEO，使用此方法引用外部css文件，将使得html页面的源代码少很多比起直接加入css样式，因为搜索引擎蜘蛛爬行网页的时候是不爬行css文件的，这样使得html源代码很少，使得蜘蛛爬行更快，处理更少，增大了此网页的权重，有利于排名。
   - 节约html使得浏览器下载网页时候分开线程了，就像加载一个页面同时有两条线在打开一个页面道理，使得网页打开格外快。（用户浏览此网页的时候html源代码和css文件同时下载，使得更加快速）
   - 修改网页的样式方便，只需修改css样式即可修改网页的美工样式，如果在网站项目中此方法，因整站应用了共用的css基本样式，这样修改整站风格样式根据快捷方便

4. 导入式：使用`@import`引用外部CSS文件

   ```
   <style type="text/css">
   @import url(Style.css);
   </style>
   ```

   这种方式会在整个页面加载完成时才引入css文件，很明显的问题就是页面的"裸奔"现象，这个当然从交互和体验的角度考虑是绝对不能接受的

**四种CSS引入方式的优先级**

1. 就近原则

2. 理论上：行内>内嵌>链接>导入

3. 实际上：内嵌、链接、导入在同一个文件头部，谁离相应的代码近，谁的优先级高

### 块元素和内联元素

**块元素**：

- 用作web页面中的主要构建模块，每个块元素都单独显示，就好像前后都有换行，特立独行。
- 块级元素会独占一行，**默认情况下宽度自动填满其父元素宽度** 
- 宽度(width)、高度(height)、内边距(padding)和外边距(margin)都可控制。

**内联元素**：

- 用来标记小段内容，会显示在所在的段落中，随波逐流。
- 内联元素(inline)不会独占一行，相邻的内联元素会排在同一行。**其宽度随内容的变化而变化**
- 宽度(width)、高度(height)、内边距的top/bottom(padding-top/padding-bottom)和外边距的top/bottom(margin-top/margin-bottom)都不可改变（即左右可变）。

设计页面时，一般先从块元素开始，然后在完善页面时加入内联元素。

通过CSS样式可以控制元素的显示，主要有以下三类

- display:block  -- 显示为块级元素
- display:inline  -- 显示为内联元素
- dipslay:inline-block -- 显示为内联块元素，表现为同行显示并可修改宽高内外边距等属性。

注意：内联元素内可以放块元素，内部的块元素会撑大外部的内联标签，可以通过放块元素来控制内联元素的高度。

**内联块状元素inline-block：**

简单来说就是将对象呈现为inline对象，但是对象的内容作为block对象呈现（可以设置宽高和margin值）。之后的内联对象会被排列在同一内联。比如我们可以给一个link（a元素）inline-block属性值，使其既具有block的宽度高度特性又具有inline的同行特性。

#### 区别

区别主要是三个方面:一是排列方式，二是宽高边距设置，三是默认宽度。

(1) 块级元素会独占一行，而内联元素和内联块元素则会在一行内显示。

(2) 块级元素和内联块元素可以设置 width、height 属性，而内联元素设置无效。

(3) 块级元素的 width 默认为 100%，而内联元素则是根据其自身的内容或子元素来决定其宽度

(4) 块元素主要用来做页面中的布局，内联元素主要用来选中文本设置样式，一般情况下只使用块元素去包含内联元素，而不会使用内联元素去包含一个块元素



**常见内联元素**：

   ```
a – 锚点 
abbr – 缩写 
b – 粗体(不推荐) 
big – 大字体 
br – 换行 
cite – 引用 
code – 计算机代码(在引用源码的时候需要) 
em – 强调 
font – 字体设定(不推荐) 
i – 斜体 
img – 图片 
input – 输入框 
kbd – 定义键盘文本 
label – 表格标签 
q – 短引用 
span – 常用内联容器，定义文本内区块 
strong – 粗体强调 
textarea – 多行文本输入框 
   ```

**块级元素**：

```
address – 地址 
blockquote – 块引用 
dir – 目录列表 
div – 常用块级容易，也是CSS layout的主要标签 
dl – 定义列表 
fieldset – form控制组 
form – 交互表单 
h1 – h6 标题 
hr – 水平分隔线 
menu – 菜单列表 
ol – 有序表单 
p – 段落 
pre – 格式化文本 
table – 表格 
ul – 无序列表 
li

```

### 选择器的分组

要使用css对HTML页面中的元素实现一对一，一对多或者多对一的控制，这就需要用到CSS选择器。HTML页面中的元素就是通过CSS选择器进行控制的。

严格来讲，选择器的种类可以分为三种：标签名选择器、类选择器和ID选择器。

而所谓的后代选择器和群组选择器只不过是对前三种选择器的扩展应用。而在标签内写入style=""的方式，应该是CSS的一种引入方式，而不是选择器，因为根本就没有用到选择器。

而一般人们将上面这几种方式结合在一起，所以就有了5种或6种选择器了。

元素之间的关系

- 父元素：直接包含子元素的元素
- 子元素：直接被父元素包含的元素
- 祖先元素：直接或间接包含后代元素的元素，父元素也是祖先元素
- 后代元素：直接或间接被祖先元素包含的元素，子元素也是后代元素
- 兄弟元素：拥有相同父元素的元素叫做兄弟元素

通过[游戏](http://flukeout.github.io/)学习CSS，练习三到四次。

常用选择器：

![](https://i.loli.net/2019/05/15/5cdbb333c854b84172.png)



```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>常用选择器</title>
		<style type="text/css">

			/*为页面中的所有的p元素，设置一个字体颜色为红色*/
			/*
			 * 元素选择器
			 * 	作用：通过元素选择器可以选则页面中的所有指定元素
			 *  语法：标签名 {}
			 */
			
			/*p{
				color: red;
			}
			
			h1{
				color: red;
			}*/
			
			/*
			 * id选择器
			 * 	- 通过元素的id属性值选中唯一的一个元素
			 *  - 语法：
			 * 		#id属性值 {}
			 */
			/*#p1{
				font-size: 20px;
			}*/
			
			/*
			 * 类选择器
			 * 	- 通过元素的class属性值选中一组元素
			 *  - 语法：
			 * 		.class属性值{}
			 */
			/*.p2{
				color: red;
			}
			
			.hello{
				font-size: 50px;
			}*/
			
			/*
			 * 为id为p1的元素，class为p2的元素，还有h1,同时设置一个背景颜色为黄色
			 */
			
			/*
			 * 选择器分组(并集选择器)
			 * 	- 通过选择器分组可以同时选中多个选择器对应的元素
			 * 	- 语法：选择器1,选择器2,选择器N{}
			 */
			/*#p1 , .p2 , h1{
				background-color: yellow;
			}*/
			
			/*
			 * 通配选择器
			 * 	- 他可以用来选中页面中的所有的元素
			 * 	语法：*{}
			 */
			
			/**{
				color: red;
			}*/
			
			/*
			 * 为拥有class p3 span元素设置一个背景颜色为黄色
			 * 
			 * 复合选择器（交集选择器）
			 * 	- 作用：
			 * 		- 可以选中同时满足多个选择器的元素
			 *  - 语法：
			 * 		- 选择器1选择器2选择器N{}
			 */
			span.p3{
				background-color: yellow;
			}
			
			/*
			 * 对于id选择器来说，不建议使用复合选择器
			 * p#p1{
				background-color: red;
			}*/	
		</style>
	</head>
	<body>
		<h1>悯农</h1>
		<p>锄禾日当午</p>
		<p>锄禾日当午</p>
		<p id="p1">锄禾日当午</p>
		
		<!-- 
			我们可以为元素设置class属性，
				class属性和id属性类似，只不过class属性可以重复
				拥有相同class属性值的元素，我们说他们是一组元素
				可以同时为一个元素设置多个class属性值，多个值之间使用空格隔开
		-->
		<p class="p2 hello">锄禾日当午</p>
		<p class="p2">锄禾日当午</p>
		<p class="p2">锄禾日当午</p>
		
		<p>锄禾日当午</p>
		<p>锄禾日当午</p>
		<p>锄禾日当午</p>
		
		<p class="p3">锄禾日当午</p>
		<span class="p3">汗滴禾下土</span>
		
	</body>
</html>
```

##### 伪类

就其定义而论，“伪类用于向某些选择器添加特殊的效果”，说实话是很难看懂他到底是干嘛的，但我们可以从它的使用方面感受他带来的便利，在开始讲使用之前我们要先了解一下常用的伪类：

1. :link 应用于未被访问过的链接；

2. :hover 应用于鼠标悬停到的元素；

3. :active 应用于被激活的元素；

4. :visited 应用于被访问过的链接，与:link互斥。

5. :focus 应用于拥有键盘输入焦点的元素

6. :first-child 选择某个元素的第一个子元素；

7. :last-child 选择某个元素的最后一个子元素；

8. :nth-child() 选择某个元素的一个或多个特定的子元素；

9. :empty 选择的元素里面没有任何内容。

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			/*
			 * 伪类专门用来表示元素的一种的特殊的状态，
			 * 	比如：访问过的超链接，比如普通的超链接，比如获取焦点的文本框
			 * 当我们需要为处在这些特殊状态的元素设置样式时，就可以使用伪类
			 */
			/*
			 * 涉及到a的伪类一共有四个：
			 * 	:link
			 *  :visited
			 * 	:hover
			 * 	:active
			 * 而这四个选择器的优先级是一样的。
			 */
						
			/*
			 * 为没访问过的链接设置一个颜色为绿色
			 * 	:link
			 * 		- 表示普通的链接（没访问过的链接）
			 */
			a:link{
				color: yellowgreen;
			}
			
			/*
			 * 为访问过的链接设置一个颜色为红色
			 * 	:visited
			 * 		- 表示访问过的链接
			 * 
			 * 浏览器是通过历史记录来判断一个链接是否访问过,
			 * 	由于涉及到用户的隐私问题，所以使用visited伪类只能设置字体的颜色
			 * 
			 */
			a:visited{
				color: red;
			}
			
			/*
			 * :hover伪类表示鼠标移入的状态
			 */
			a:hover{
				color: skyblue;
			}
			
			/*
			 * :active表示的是超链接被点击的状态
			 */
			a:active{
				color: black;
			}
			
			/*
			 * :hover和:active也可以为其他元素设置
			 * IE6中，不支持对超链接以外的元素设置:hover和:active
			 */
			/*p:hover{
				background-color: yellow;
			}
			
			p:active{
				background-color: orange;
			}*/
			
			/*
			 * 文本框获取焦点以后，修改背景颜色为黄色
			 */
			input:focus{
				background-color: yellow;
			}
			
			/**
			 * 为p标签中选中的内容使用样式
			 * 	可以使用::selection为类
			 * 	注意：这个伪类在火狐中需要采用另一种方式编写::-moz-selection
			 */
			
			/**
			 * 兼容火狐的
			 */
			p::-moz-selection{
				background-color: orange;
			}
			
			/**
			 * 兼容大部分浏览器的
			 */
			p::selection{
				background-color: orange;
			}
			
		</style>
	</head>
	<body>
		
		
		<a href="http://www.baidu.com">访问过的链接</a>
		<br /><br />
		<a href="http://www.baidu123456.com">没访问过的链接</a>
		
		<p>我是一个段落</p>
		
		<!-- 使用input可以创建一个文本输入框 -->
		<input type="text" />
		
	</body>
</html>

```

**否定伪类**

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 为所有的p元素设置一个背景颜色为黄色，除了class值为hello的
			 * 
			 * 否定伪类：
			 * 	作用：可以从已选中的元素中剔除出某些元素
			 * 	语法：
			 * 		:not(选择器)
			 */
			p:not(.hello){
				background-color: yellow;
			}
			
		</style>
	</head>
	<body>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p class="hello">我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
	</body>
</html>
```

##### 伪元素

伪元素和伪类的定义刚好相反，“伪元素用于向某些选择器设置特殊效果”，伪元素主要有以下几类：

1. :first-letter 选择元素文本的第一个字（母）。

2. :first-line 选择元素文本的第一行。

3. :before 在元素内容的最前面添加新内容。

4. :after 在元素内容的最后面添加新内容。

```
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			/*
			 * 使用伪元素来表示元素中的一些特殊的位置
			 */
			/*
			 * 为p中第一个字符来设置一个特殊的样式
			 */
			
			/*p:first-letter {
				color: red;
				font-size: 20px;
			}*/
			/*
			 * 为p中的第一行设置一个背景颜色为黄色
			 */
			
			/*p:first-line {
				background-color: yellow;
			}*/
			
			/*
			 * :before表示元素最前边的部分
			 * 	一般before都需要结合content这个样式一起使用，
			 * 	通过content可以向before或after的位置添加一些内容
			 * 
			 * :after表示元素的最后边的部分
			 */
			p:before{
				content: "我会出现在整个段落的最前边";
				color: red;
			}
			
			p:after{
				content: "我会出现在整个段落的最后边";
				color: orange;
			}
		</style>
	</head>

	<body>
		<p>
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>
	</body>

</html>
```

##### 属性选择器

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 为所有具有title属性的p元素，设置一个背景颜色为黄色
			 * 属性选择器
			 * 	- 作用：可以根据元素中的属性或属性值来选取指定元素
			 * 	- 语法：
			 * 		[属性名] 选取含有指定属性的元素
			 * 		[属性名="属性值"] 选取含有指定属性值的元素
			 * 		[属性名^="属性值"] 选取属性值以指定内容开头的元素
			 * 		[属性名$="属性值"] 选取属性值以指定内容结尾的元素
			 * 		[属性名*="属性值"] 选取属性值以包含指定内容的元素
			 */
			/*p[title]{
				background-color: yellow;
			}*/
			
			/*
			 * 为title属性值是hello的元素设置一个背景颜色为黄色
			 */
			/*p[title="hello"]{
				background-color: yellow;
			}*/
			
			/*
			 * 为title属性值以ab开头的元素设置一个背景颜色为黄色
			 */
			/*p[title^="ab"]{
				background-color: yellow;
			}*/
			
			/*
			 * 为title属性值以c结尾的元素设置一个背景颜色
			 */
			/*p[title$="c"]{
				background-color: yellow;
			}*/
			
			p[title*="c"]{
				background-color: yellow;
			}
			
			
		</style>
	</head>
	<body>
		
		<!--
			title属性，这个属性可以给任何标签指定
				当鼠标移入到元素上时，元素中的title属性的值将会作为提示文字显示
		-->
		<p title="hello">我是一个段落</p>
		<p>我是一个段落</p>
		<p title="hello">我是一个段落</p>
		<p title="abbc">我是一个段落</p>
		<p title="abccd">我是一个段落</p>
		<p title="abc">我是一个段落</p>
		
	</body>
</html>
```

##### 子元素选择器

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			/*
			 * 为第一个p标签设置一个背景颜色为黄色
			 * 	:first-child 可以选中第一个子元素
			 *  :last-child 可以选中最后一个子元素
			 */
			/*body > p:first-child{
				background-color: yellow;
			}*/
			
			/*p:last-child{
				background-color: yellow;
			}*/
			
			/*
			 * :nth-child 可以选中任意位置的子元素
			 * 		该选择器后边可以指定一个参数，指定要选中第几个子元素
			 * 		even 表示偶数位置的子元素
			 * 		odd 表示奇数位置的子元素
			 * 		
			 */
			/*p:nth-child(odd){
				background-color: yellow;
			}*/
			
			/*
			 * :first-of-type
			 * :last-of-type
			 * :nth-of-type
			 * 		和:first-child这些非常的类似，
			 * 		只不过child，是在所有的子元素中排列
			 * 		而type，是在当前类型的子元素中排列
			 */
			/*p:first-of-type{
				background-color: yellow;
			}*/
			p:last-of-type{
				background-color: yellow;
			}
		</style>
	</head>
	<body>
		<span>我是span</span>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<span>hello</span>
		
		<!--<div>
			<p>我是DIV中的p标签</p>
		</div>-->
		
	</body>
</html>

```

##### 兄弟元素选择器

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 为span后的一个p元素设置一个背景颜色为黄色
			 * 后一个兄弟元素选择器
			 * 	作用：可以选中一个元素后紧挨着的指定的兄弟元素
			 * 	语法：前一个 + 后一个
			 * 
			 */
			
			/*span + p{
				background-color: yellow;
			}*/
			
			/*
			 * 选中后边的所有兄弟元素
			 * 	语法：前一个 ~ 后边所有	
			 */
			span ~ p{
				background-color: yellow;
			}
			
		</style>
	</head>
	<body>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<span>我是一个span</span>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
		<p>我是一个p标签</p>
	</body>
</html>
```

#### 样式的继承

所谓CSS的继承是指被包在内部的标签将拥有外部标签的样式性质。继承特性最典型的应用通常发挥在整个网页的样式预设，需要指定为其它样式的部份设定在个别元素里即可。这项特性可以给网页设计者提供更理想的发挥空间。但同时继承也有很多规则，应用的时候容易让人迷惑

在CSS中，继承是一种非常自然的行为，我们甚至不需要考虑是否能够这样去做，但是继承也有其局限性。

首先，有些属性是不能继承的。这没有任何原因，只是因为它就是这么设置的。举个例子来说：border属性，大家都知道，border属性是用来设置元素的边框的，它就没有继承性。如下图所示，如果继承了边框属性，那么文档看起来就会很奇怪，除非采取另外的措施关掉边框的继承属性。

多数边框类属性，比如象Padding（补白），Margin（边界），背景和边框的属性都是不能继承的

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			body{
				font-size: 30px;
			}
		</style>
	</head>
	<body>
		<!-- 
			像儿子可以继承父亲的遗产一样，在CSS中，祖先元素上的样式，也会被他的后代元素所继承,
			利用继承，可以将一些基本的样式设置给祖先元素，这样所有的后代元素将会自动继承这些样式。
			
			但是并不是所有的样式都会被子元素所继承，比如：背景相关的样式都不会被继承 边框相关的样式 定位相关的
		-->
		
		<div style="background-color: yellow;">
			<p>
				我是p标签中的文字
				<span>我是span中的文字</span>
			</p>
		</div>
		
		
		<span>我是p元素外的span</span>
		
	</body>
</html>
```

**权重**

样式表中的特殊性描述了不同规则的相对权重，它的基本规则是：

- 统计选择符中的ID属性个数。 
- 统计选择符中的CLASS属性个数。 
- 统计选择符中的HTML标记名格式。 
- 最后，按正确的顺序写出三个数字，不要加空格或逗号，得到一个三位数。( 注意，你需要将数字转换成一个以三个数字结尾的更大的数)。相应于选择符的最终数字列表可以很容易确定较高数字特性凌驾于较低数字的。

以下是一个按特性分类的选择符的列表：

- H1 {color:blue;}                        特性值为：1
- P EM {color:purple;}                    特性值为：2
- .apple {red;}                           特性值为：10 
- P.bright {color:yellow;}                  特性值为：11
- P.bright EM.dark {color:brown;}           特性值为：22
- #id316 {color:yellow}                   特性值为：100

从上表我们可以看出#id316具有更高的特殊性，因而它有更高的权重。当有多个规则都能应用于同一个元素时，权重越高的样式将被优先采用。

有的时候我们为同一个元素设置了不同的CSS样式代码，那么元素会启用哪一个CSS样式呢?我们来看一下面的代码：

```
p{color:red;}
.first{color:green;}
<p class="first">三年级时，我还是一个<span>胆小如鼠</span>的小女孩。</p>
```

p和.first都匹配到了p这个标签上，那么会显示哪种颜色呢？green是正确的颜色，那么为什么呢？

是因为浏览器是根据权值来判断使用哪种css样式的，权值高的就使用哪种css样式。例如下面的代码：

```
p{color:red;} /*权值为1*/
p span{color:green;} /*权值为1+1=2*/
.warning{color:white;} /*权值为10*/
p span.warning{color:purple;} /*权值为1+1+10=12*/
#footer .note p{color:yellow;} /*权值为100+10+1=111*/
```

注意：还有一个权值比较特殊--继承也有权值但很低，有的文献提出它只有0.1，所以可以理解为继承的权值最低。

**层叠**

层叠就是在html文件中对于同一个元素可以有多个css样式存在，当有相同权重的样式存在时，会根据这些css样式的前后顺序来决定，处于最后面的css样式会被应用。

如下面代码:

```
p{color:red;}
p{color:green;}
<p class="first">三年级时，我还是一个<span>胆小如鼠</span>的小女孩。</p>
```

最后 p 中的文本会设置为green，这个层叠很好理解，理解为后面的样式会覆盖前面的样式。所以前面的css样式优先级就不难理解了：

内联样式表（标签内部）> 嵌入样式表（当前文件中）> 外部样式表（外部文件中）。

我们在做网页代码的时，有些特殊的情况需要为某些样式设置具有最高权值，怎么办？这时候我们可以使用!important来解决。
如下代码：

```
p{color:red!important;}
p{color:green;}
<p class="first">三年级时，我还是一个<span>胆小如鼠</span>的小女孩。</p>
```

这时 p 段落中的文本会显示的red红色。

> 注意：!important要写在分号的前面

这里注意当网页制作者不设置css样式时，浏览器会按照自己的一套样式来显示网页。并且用户也可以在浏览器中设置自己习惯的样式，比如有的用户习惯把字号设置为大一些，使其查看网页的文本更加清楚。这时注意样式优先级为：浏览器默认的样式 < 网页制作者样式 < 用户自己设置的样式，但记住!important优先级样式是个例外，权值高于用户自己设置的样式。

#### 选择器的优先级

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.p1{
				background-color: yellow;
			}
			
			p{
				background-color: red;
			}
			
			
			
			/*
			 * 当使用不同的选择器，选中同一个元素时并且设置相同的样式时，
			 * 	这时样式之间产生了冲突，最终到底采用哪个选择器定义的样式，由选择器的优先级（权重）决定
			 *  优先级高的优先显示。
			 * 
			 * 优先级的规则
			 * 		内联样式 ， 优先级  1000
			 * 		id选择器，优先级   100
			 * 		类和伪类， 优先级   10
			 * 		元素选择器，优先级 1 
			 * 		通配* ，    优先级 0
			 * 		继承的样式，没有优先级
			 * 
			 * 当选择器中包含多种选择器时，需要将多种选择器的优先级相加然后在比较，
			 * 	但是注意，选择器优先级计算不会超过他的最大的数量级，如果选择器的优先级一样，
			 * 	则使用靠后的样式。
			 * 
			 *  并集选择器的优先级是单独计算
			 * 	div , p , #p1 , .hello{}	
			 * 
			 *  可以在样式的最后，添加一个!important，则此时该样式将会获得一个最高的优先级，
			 * 	将会优先于所有的样式显示甚至超过内联样式，但是在开发中尽量避免使用!important
			 * 
			 */
			
			*{
				font-size: 50px;
			}
			
			p{
				font-size: 30px;
			}
			
			#p2{
				background-color: yellowgreen;
			}
			
			p#p2{
				background-color: red;
			}
			
			
			.p3{
				color: green;
			}
			
			.p1{
				color: yellow;
				background-color: greenyellow !important;
			}
			
			
			
		</style>
	</head>
	<body>
		
		<p class="p1 p3" id="p2" style="background-color: orange;">我是一个段落
			<span>我是p标签中的span</span>
		</p>
		
	</body>
</html>

```

