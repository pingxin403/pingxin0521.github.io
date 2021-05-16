---
title: CSS 再入深坑
date: 2019-04-29 12:18:59
tags:
 - 前端
 - CSS
categories:
 - 前端
 - CSS
---

### 层级

z-index属性指定了元素与元素之间的z轴上的顺序，而z轴决定元素之间发生覆盖的层叠关系。

<!--more-->

**属性值**

1. 默认auto

为什么元素添加定位属性(不包括static)后会覆盖普通元素？

元素添加定位属性(不包括static)后，z-index默认是auto，在层叠水平上相当于z-index为0，但是不会产生层叠上下文，但是会比普通元素没有z-index的层级要高。

2.  数值

z-index支持正负值

3. inherit

继承父元素的z-index值

**层叠上下文**

层叠上下文是html中的一个概念，当一个元素元素含有层叠上下文的时候，那么此元素就更靠近我们的眼睛（假如我们看一堵墙的时候，只能看到墙，后面的东西我们看不到，那这堵墙就相当于含有层叠上下文，更靠近我们，后面的东西可以看做是普通元素）。

第一种方法，页面根元素(html页面根元素就是html)天生具有层叠上下文，我们称它为“根层叠上下文”。

字体在背景上就是以页面根元素为层叠上下文为基础进行覆盖的，字体是inline元素，而background是层叠顺序最低的元素，遵循层叠顺序。

第二种方法，定位元素的z-index为数值的层叠上下文。

```
div{position:absolute; z-index:1;}
```

第三种方法，其他css的属性。

这些属性可以看做z-index为auto

- z-index值不为auto的flex项（父元素display:flex|inline-flex）。
-   元素的opacity值不是1。
-   元素的transform值不是none。
-   元素的mix-blend-mode值不是normal。
-   元素的filter值不是none。
-   元素的isolation值不是isolate。
-   position:fixed声明。
-  元素的-webkit-overflow-scrolling为touch。

**层叠水平**

在层叠上下文中往往包含很多元素，这么多元素由层叠水平来决定z轴上的先后顺序，不至于相互打架，一起相争。

**层叠顺序**

层叠顺序是指不同元素相互层叠，规定先后顺序的一套规则。而层叠上下文和层叠水平都是概念。

![1.png](https://i.loli.net/2019/05/16/5cdd73cc2590726936.png)

**z-index的层叠**

(1) 不发生嵌套

没有层叠上下文，则根据DOM顺序，后面的覆盖前面的。

在有层叠上下文的时候，根据z-index的数值大小，谁大谁在上面。

(2) 发生嵌套

每个层叠上下文相当于每一个独立的个体，内部的元素无论z-index为多大，对最顶部同级的层叠上下文元素毫无影响。

子级无论z-index为何值，都不会对父级元素层级造成影响。

为了方便理解，你可以把同一层级的层叠上下文元素看做是两个不同的人，不同的人有不同的高度，你可以比较两个人的高度，但是你不能说手长身高就更高一点吧。

层叠上下文内部的其他元素的覆盖关系就是以父级为根层叠上下文，重新按照上面的规则进行覆盖。

**z-index使用**

(1) z-index的技巧

*1**.***定位元素z-index:auto可以看成z-index:0

例如：定位元素（不包括static）的层级要比普通元素层级要高。

*2.*z-index不为auto会产生层叠上下文

例如：两个同级div都设置relative和z-index为数值，这两个div就会产生层叠上下文，内部元素永远不会影响到父级div的覆盖关系。

*3.*z-index层叠顺序终止与父级层叠上下文

(2) z-index值最好不要超过9

z-index在使用过程中，为避免一个项目多人协作产生互相覆盖的问题，所以在使用z-index的时候，不要让z-index大于9，防止同事设置弹出框或其他层级要求很高的无法覆盖。

(3) 获取最高层级

我们在用到一些插件什么的时候，有时插件有一个很高的层级，我们自定义的元素需要覆盖在其之上，这时我们可以通过js获取body内最高层级，让我们自定义的元素z-index等于最高层级的z-index+1。

(4) 利用z-index隐藏

在模拟button和input.submit的时候，我们可以设置样式如下：

```
button{
position:absolute;
z-index:-1;
}
label{
background:#999;
padding:8px 16px;
line-height:30px;
color:#fff;
}
<button id="btn"></button>
<label for='btn'></label>
```

上面父级无定位z-index设置，自然不会创建层叠上下文，此时层叠上下文为根层叠上下文html，则button是以为层叠上下文根元素html来进行覆盖，由于label为inline元素，button层级为z-index负值，根据层叠顺序我们可以明白button在body上面，而label位于button之上。

示例：

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<link rel="stylesheet" type="text/css" href="#">
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: red;
				position: relative;
				
				z-index: 2;
				
				opacity: 0.5;
				filter: alpha(opacity=50);
			}
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellow;
				/*开启绝对定位*/
				position: absolute;
				/*设置偏移量*/				
				top: 100px;
				left: 100px;
				/*
				 * 如果定位元素的层级是一样，则下边的元素会盖住上边的
				 * 通过z-index属性可以用来设置元素的层级
				 * 可以为z-index指定一个正整数作为值，该值将会作为当前元素的层级
				 * 	层级越高，越优先显示
				 * 
				 * 对于没有开启定位的元素不能使用z-index
				 */
				z-index: 25;
				
				opacity: 0.5;
				filter: alpha(opacity=50);
				
			}
			.box3{
				width: 200px;
				height: 200px;
				background-color: yellowgreen;
				/*position: relative;
				z-index: 3;*/
				position: absolute;
				top: 200px;
				left: 200px;
				z-index: 30;
				
				/*
				 * 设置元素的透明背景
				 * opacity可以用来设置元素背景的透明，
				 * 	它需要一个0-1之间的值
				 * 		0 表示完全透明
				 * 		1 表示完全不透明
				 * 		0.5 表示半透明
				 */
				opacity: 0.5;
				
				/*
				 * opacity属性在IE8及以下的浏览器中不支持
				 * IE8及以下的浏览器需要使用如下属性代替
				 * 	alpha(opacity=透明度)
				 * 透明度，需要一个0-100之间的值
				 * 	0 表示完全透明
				 * 	100 表示完全不透明
				 * 	50 半透明
				 * 
				 * 这种方式支持IE6，但是这种效果在IE Tester中无法测试
				 */
				filter: alpha(opacity=50);
			}
			
			.box4{
				width: 200px;
				height: 200px;
				background-color: orange;
				/*开启相对定位*/
				position: relative;
				/*
				 * 父元素的层级再高，也不会盖住子元素
				 */
				z-index: 20;
				
				top: 500px;
			}
			
			.box5{
				width: 100px;
				height: 100px;
				background-color: skyblue;
				/*开启绝对定位*/
				position: absolute;
				z-index: 10;
			
			}
			
		</style>
		
	</head>
	<body style="height: 5000px;">
		
		<div  class="box1"></div>
		<div class="box2"></div>
		<div class="box3"></div>
		<div class="box4">
			<div class="box5"></div>
		</div>
	</body>
</html>

```

### 背景

![2.png](https://i.loli.net/2019/05/16/5cdd7512437d710738.png)

#### 背景颜色

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <style>
            div{
                width: 300px;
                height: 300px;
                border: 10px solid red;

                /*背景颜色三种表示方法:颜色的单词,rgb,16进制*/
                background-color:green;
                background-color: rgb(0,0,0);
                background-color:#008800;
            }
        </style>
    </head>
    <body>

        <div>

        </div>
    </body>
</html>
```

#### 背景图片

背景图片有：位置  重复性   

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.box1{
				width: 1024px;
				height: 724px;
				margin: 0 auto;
				/*设置背景样式*/
				background-color: #bfa;
				/*
				 * 使用background-image来设置背景图片
				 * 	- 语法：background-image:url(相对路径);
				 * 
				 * 	- 如果背景图片大于元素，默认会显示图片的左上角
				 * 	- 如果背景图片和元素一样大，则会将背景图片全部显示
				 * 	- 如果背景图片小于元素大小，则会默认将背景图片平铺以充满元素
				 * 
				 * 可以同时为一个元素指定背景颜色和背景图片，
				 * 	这样背景颜色将会作为背景图片的底色
				 * 	一般情况下设置背景图片时都会同时指定一个背景颜色
				 */
				background-image:url(img/1.png);
				
				/*
				 * background-repeat用于设置背景图片的重复方式
				 * 	可选值：
				 * 		repeat，默认值，背景图片会双方向重复（平铺）
				 * 		no-repeat ，背景图片不会重复，有多大就显示多大
				 * 		repeat-x， 背景图片沿水平方向重复
				 * 		repeat-y，背景图片沿垂直方向重复
				 */
				background-repeat: no-repeat;
				/*
				 * 背景图片默认是贴着元素的左上角显示
				 * 通过background-position可以调整背景图片在元素中的位置
				 * 可选值：
				 * 		该属性可以使用 top right left bottom center中的两个值
				 * 			来指定一个背景图片的位置
				 * 			top left 左上
				 * 			bottom right 右下
				 * 			如果只给出一个值，则第二个值默认是center
				 * 
				 * 		也可以直接指定两个偏移量，
				 * 			第一个值是水平偏移量
				 * 				- 如果指定的是一个正值，则图片会向右移动指定的像素
				 * 				- 如果指定的是一个负值，则图片会向左移动指定的像素
				 * 			第二个是垂直偏移量	
				 * 				- 如果指定的是一个正值，则图片会向下移动指定的像素
				 * 				- 如果指定的是一个负值，则图片会向上移动指定的像素
				 * 		
				 */
				/*background-position: -80px -40px;*/
					/*
				 * background-attachment用来设置背景图片是否随页面一起滚动
				 * 		可选值：
				 * 			scroll，默认值，背景图片随着窗口滚动
				 * 			fixed，背景图片会固定在某一位置，不随页面滚动
				 * 
				 * 不随窗口滚动的图片，我们一般都是设置给body，而不设置给其他元素
				 */
				 /*
				 * 当背景图片的background-attachment设置为fixed时，
				 * 	背景图片的定位永远相对于浏览器的窗口
				 */
				background-attachment: fixed;
				
			   /*
				 * background
				 * 	- 通过该属性可以同时设置所有背景相关的样式
				 * 	- 没有顺序的要求，谁在前睡在后都行
				 * 		也没有数量的要求，不写的样式就使用默认值
				 background: #bfa url(img/3.png) center center no-repeat fixed;

				 */
			}
			
			
		</style>
		
		<!--<link rel="stylesheet" type="text/css" href="css/style.css"/>-->
	</head>
	<body>
		
		<div class="box1"></div>
		
	</body>
</html>
```

### 表格

在HTML中，Table节点由于其层层嵌套的节点结构，一度名声很臭，且一度被呼吁用DIV+CSS取而代之。但在实际项目开发中，一碰到规整的数据显示，不知不觉又会用起它。可见其生命力之顽强。

#### 表格边框

如需在 CSS 中设置表格边框，请使用 border 属性。

```
table, th, td
  {
  border: 1px solid blue;
  }
```

请注意，上例中的表格具有双线条边框。这是由于 table、th 以及 td 元素都有独立的边框。

如果需要把表格显示为单线条边框，请使用 border-collapse 属性。

#### 折叠边框

border-collapse 属性设置是否将表格边框折叠为单一边框：

```
table
  {
  border-collapse:collapse;
  }

table,th, td
  {
  border: 1px solid black;
  }
```

#### 表格宽度和高度

通过 width 和 height 属性定义表格的宽度和高度。

```
table
  {
  width:100%;
  }

th
  {
  height:50px;
  }
```

#### 表格文本对齐

ext-align 和 vertical-align 属性设置表格中文本的对齐方式。

text-align 属性设置水平对齐方式，比如左对齐、右对齐或者居中：

```
td
  {
  text-align:right;
  }
```

vertical-align 属性设置垂直对齐方式，比如顶部对齐、底部对齐或居中对齐：

```
td
  {
  height:50px;
  vertical-align:bottom;
  }
```

#### 表格内边距

如需控制表格中内容与边框的距离，请为 td 和 th 元素设置 padding 属性：

```
td
  {
  padding:15px;
  }
```

#### 表格颜色

下面的例子设置边框的颜色，以及 th 元素的文本和背景颜色：

```
table, td, th
  {
  border:1px solid green;
  }

th
  {
  background-color:green;
  color:white;
  }

```

#### CSS Table 属性

| 属性                                                         | 描述                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| [border-collapse](http://www.w3school.com.cn/cssref/pr_tab_border-collapse.asp) | 设置是否把表格边框合并为单一的边框。 |
| [border-spacing](http://www.w3school.com.cn/cssref/pr_tab_border-spacing.asp) | 设置分隔单元格边框的距离。           |
| [caption-side](http://www.w3school.com.cn/cssref/pr_tab_caption-side.asp) | 设置表格标题的位置。                 |
| [empty-cells](http://www.w3school.com.cn/cssref/pr_tab_empty-cells.asp) | 设置是否显示表格中的空单元格。       |
| [table-layout](http://www.w3school.com.cn/cssref/pr_tab_table-layout.asp) | 设置显示单元、行和列的算法。         |

示例

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 设置表格的宽度
			 */
			table{
				width: 300px;
				/*居中*/
				margin: 0 auto;
				/*边框*/
				/*border:1px solid black;*/
				/*
				 * table和td边框之间默认有一个距离
				 * 	通过border-spacing属性可以设置这个距离
				 */
				/*border-spacing:0px ;*/
				
				/*
				 * border-collapse可以用来设置表格的边框合并
				 * 如果设置了边框合并，则border-spacing自动失效
				 */
				border-collapse: collapse;
				/*设置背景样式*/
				/*background-color: #bfa;*/
			}
			
			/*
			 * 设置边框
			 */
			td , th{
				border: 1px solid black;
			}
			
			/*
			 * 设置隔行变色
			 */
			tr:nth-child(even){
				background-color: #bfa;
			}
			
			/*
			 * 鼠标移入到tr以后，改变颜色
			 */
			tr:hover{
				background-color: #ff0;
			}
			
			
		</style>
	</head>
	<body>
		<!--
			table是一个块元素
		-->
		
		<table>
			<tr>
				<!--
					可以使用th标签来表示表头中的内容，
						它的用法和td一样，不同的是它会有一些默认效果
				-->
				<th>学号</th>
				<th>姓名</th>
				<th>性别</th>
				<th>住址</th>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
			<tr>
				<td>1</td>
				<td>孙悟空</td>
				<td>男</td>
				<td>花果山</td>
			</tr>
			<tr>
				<td>2</td>
				<td>猪八戒</td>
				<td>男</td>
				<td>高老庄</td>
			</tr>
			<tr>
				<td>3</td>
				<td>沙和尚</td>
				<td>男</td>
				<td>流沙河</td>
			</tr>
			<tr>
				<td>4</td>
				<td>唐僧</td>
				<td>男</td>
				<td>女儿国</td>
			</tr>
		</table>
		
	</body>
</html>
```

#### 高度塌陷补充

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.box1{
				width: 300px;
				height: 300px;
				background-color: #bfa;
			}
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellow;
				
				/*
				 * 子元素和父元素相邻的垂直外边距会发生重叠，子元素的外边距会传递给父元素
				 * 使用空的table标签可以隔离父子元素的外边距，阻止外边距的重叠
				 */
				margin-top: 100px;
			}
			
			
			
			
			.box3{
				border: 10px red solid;
			}
			
			.box4{
				width: 100px;
				height: 100px;
				background-color: yellowgreen;
				float: left;
			}
			
			/**
			 * 解决父子元素的外边距重叠
			 */
			/*.box1:before{
				content: "";*/
				/*
				 * display:table可以将一个元素设置为表格显示
				 */
			/*	display: table;
			}
			*/
			
			/**
			 * 解决父元素高度塌陷
			 */
			/*.clearfix:after{
				content: "";
				display: block;
				clear: both;
			}*/
			
			/*
			 * 经过修改后的clearfix是一个多功能的
			 * 	既可以解决高度塌陷，又可以确保父元素和子元素的垂直外边距不会重叠
			 */
			.clearfix:before,
			.clearfix:after{
				content: "";
				display: table;
				clear: both;
			}
			
			.clearfix{
				zoom: 1;
			}
		</style>
	</head>
	<body>
		
		<div class="box3 clearfix">
			<div class="box4"></div>
		</div>
		
		<div class="box1 clearfix">
			<div class="box2"></div>
		</div>
		
	</body>
</html>

```

### CSS Hack

在我们制作页面时CSS hack由于不同的浏览器，比如Internet Explorer,Mozilla Firefox等，对CSS的解析认识不一样，因此会导致生成的页面效果不一样，得不到我们所需要的页面效果。 这个时候我们就需要针对不同的浏览器去写不同的CSS，让它能够同时兼容不同的浏览器，能在不同的浏览器中也能得到我们想要的页面效果。

CSS Hack大致有3种表现形式，属性级Hack、选择器Hack以及IE条件Hack

注意：尽可能减少对CSS Hack的使用。

原理：
由于不同的浏览器对CSS的支持及解析结果不一样，还由于CSS中的优先级的关系。我们就可以根据这个来针对不同的浏览器来写不同的CSS。

1. `属性级Hack：比如IE6能识别下划线“_”和星号“*”，IE7能识别星号“*”，但不能识别下划线”_ ”，而firefox两个都不能认识。`

2. `选择符级Hack：比如IE6能识别*html .class{}，IE7能识别*+html .class{}或者*:first-child+html .class{}。`

3. `IE条件注释Hack：IE条件注释是微软IE5开始就提供的一种非标准逻辑语句。比如针对所有IE：&lt;!-[if IE]&gt;&lt;!-您的代码-&gt;&lt;![endif]&gt;，针对IE6及以下版本：&lt;!-[if it IE 7]&gt;&lt;!-您的代码-&gt;&lt;![endif]-&gt;，这类Hack不仅对CSS生效，对写在判断语句里面的所有代码都会生`效。

PS：条件注释只有在IE浏览器下才能执行，这个代码在非IE浏览下被当做注释视而不见。可以通过IE条件注释载入不同的CSS、JS、HTML和服务器代码等。

具体参考：[这里](https://www.html.cn/book/css/hack/index.htm)

