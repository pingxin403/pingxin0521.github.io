---
title: CSS 盒子模型
date: 2019-04-25 12:18:59
tags:
 - 前端
 - CSS
categories:
 - 前端
 - CSS
---

### 盒子模型

盒子模型是由内容、边框、间隙（padding）、间隔（margin）组成，他们的关系如下图所示：

> 我们目前所学习的知识中，以标准盒子模型为准。

<!--more-->

标准盒子模型：

![2.jpg](https://i.loli.net/2019/05/15/5cdbd83f6935d23213.jpg)

IE盒子模型：

![3.jpg](https://i.loli.net/2019/05/15/5cdbda3f94ff989055.jpg)

盒子实际宽度（高度）=内容（content）+边框（border）+间隙（padding）+间隔（margin）。对于任何一个元素设置width和height控制内容大小，也可以分别设置各自的边框（border）、间隙（padding）、间隔（margin）。灵活设置这些盒子的这些属性，可以实现各自排班效果。

CSS盒模型和IE盒模型的区别：

- 在 **标准盒子模型**中，**width 和 height 指的是内容区域**的宽度和高度。增加内边距、边框和外边距不会影响内容区域的尺寸，但是会增加元素框的总尺寸。
- **IE盒子模型**中，**width 和 height 指的是内容区域+border+padding**的宽度和高度。

注：Android中也有margin和padding的概念，意思是差不多的，如果你会一点Android，应该比较好理解吧。区别在于，Android中没有border这个东西，而且在Android中，margin并不是控件的一部分

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.box1{
				/*
				 * 使用width来设置盒子内容区的宽度
				 * 使用height来设置盒子内容区的高度
				 * 
				 * width和height只是设置的盒子内容区的大小，而不是盒子的整个大小，
				 * 	盒子可见框的大小由内容区，内边距和边框共同决定
				 */
				width: 300px;
				height: 300px;
				
				/*设置背景颜色*/
				background-color: #bfa;
				
				/*
				 * 为元素设置边框
				 * 	要为一个元素设置边框必须指定三个样式
				 * 		border-width:边框的宽度
				 * 		border-color:边框颜色
				 * 		border-style:边框的样式

				 */
				
				/*
				 * 设置边框的宽度
				 */
				/*border-width:10px ;*/
				
				/*
				 	使用border-width可以分别指定四个边框的宽度
				 	如果在border-width指定了四个值，
				 		则四个值会分别设置给 上 右 下 左，按照顺时针的方向设置的
				 		
				 	如果指定三个值，
				 		则三个值会分别设置给	上  左右 下
				 		
				 	如果指定两个值，
				 		则两个值会分别设置给 上下 左右	
				 		
				 	如果指定一个值，则四边全都是该值	
				 	
				 	除了border-width，CSS中还提供了四个border-xxx-width
				 		xxx的值可能是top right bottom left
				 	专门用来设置指定边的宽度	
				 * */
				/*border-width:10px 20px 30px 40px ;*/
				/*border-width:10px 20px 30px ;*/
				/*border-width: 10px 20px ;*/
				border-width: 10px;
				
				/*border-left-width:100px ;*/
				
				
				/*
				 * 设置边框的颜色
				 * 和宽度一样，color也提供四个方向的样式，可以分别指定颜色
				 * border-xxx-color
				 */
				border-color: red;
				/*border-color: red yellow orange blue;*/
				/*border-color: red yellow orange;*/
				/*border-color: red yellow;*/
				
				/*
				 * 设置边框的样式
				 * 	可选值：
				 * 		none，默认值，没有边框
				 * 		solid 实线
				 * 		dotted 点状边框
				 * 		dashed 虚线
				 * 		double 双线
				 * 
				 * style也可以分别指定四个边的边框样式，规则和width一致，
				 * 	同时它也提供border-xxx-style四个样式，来分别设置四个边
				 */
				/*border-style: double;*/
				border-style: solid dotted dashed double; 
			}
			
		</style>
	</head>
	<body>
		<div class="box1"></div>
	</body>
</html>

```

#### border

border是元素的外围，计算元素的宽和高要把border加上特别是特殊网站页面（比如说活动专题页面等）上，这一点是很多新手忽略的地方。border有三个主要属性，color（颜色）、width（粗细）和style（样式）。

1. color主要是指定border的颜色，一共有256的3次方种颜色供我们选择。通常是16进制的值，比如红色是“#FF0000”。
2. width是border粗细程度，可以设置为thin、thick和length，length为具体数值，比如说border:1px #CCC solid;其中1px指的是border的width，默认值是medium，一般浏览器解析为2像素。
3. style属性可以设为`none、hidden、dotted、dashed、solid、double、groove、ridge、inset和outset`等，其中none和hidden是不显示border，hidden可以用来解决边框的冲突问题。对于groove、inset、outset、rigde、border-style，IE会出现兼容问题，使用时一定要注意
4. css中的border是不能有负值出现的，出现的话就相当于把那个属性给取消了。

```
<!DOCTYPE html>
<html>

<head>
	<meta charset="utf-8" />
	<title></title>
	<style type="text/css">
		.box {
			width: 200px;
			height: 200px;
			background-color: #bfa;

			/*设置边框
				 	大部分的浏览器中，边框的宽度和颜色都是有默认值，而边框的样式默认值都是none
				 * */
			/*border-width:10px ;
				border-color: red;
				
				border-style: solid;
                none 	定义无边框。
                hidden 	与 "none" 相同。不过应用于表时除外，对于表，hidden 用于解决边框冲突。
                dotted 	定义点状边框。在大多数浏览器中呈现为实线。
                dashed 	定义虚线。在大多数浏览器中呈现为实线。
                solid 	定义实线。
                double 	定义双线。双线的宽度等于 border-width 的值。
                groove 	定义 3D 凹槽边框。其效果取决于 border-color 的值。
                ridge 	定义 3D 垄状边框。其效果取决于 border-color 的值。
                inset 	定义 3D inset 边框。其效果取决于 border-color 的值。
                outset 	定义 3D outset 边框。其效果取决于 border-color 的值。
                inherit 	规定应该从父元素继承边框样式。
                */
			/*
				 * border
				 * 	- 边框的简写样式，通过它可以同时设置四个边框的样式，宽度，颜色
				 * 	- 而且没有任何的顺序要求
				 * 	- border一指定就是同时指定四个边不能分别指定
				 * 如果不设置其中的某个值，也不会出问题，比如 border:solid #ff0000; 也是允许的。
				 * border-top border-right border-bottom border-left
				 * 	可以单独设置四个边的样式，规则和border一样，只不过它只对一个边生效

				 *border-width: border-top border-right border-bottom border-left;
				 *一次定义上右下左的边框宽度，
				    如果缺少左边距的值，则使用右边距的值。
                    如果缺少下边距的值，则使用上边距的值。
                    如果缺少右边距的值，则使用上边距的值。
				 * 如：border-width:thin medium thick;
					上边框是 10px
					右边框和左边框是中等边框
					下边框是粗边框

					border-width:thin;
					所有 4 个边框都是细边框
					border-color、border-style也可以是如此设置。
				 */
			/*border: red solid 10px   ;*/
			/*border-left: red solid 10px   ;*/

			/*border-top: red solid 10px;
				border-bottom: red solid 10px;
				border-left: red solid 10px;*/
			border: red 10px;
			border-style: double;


		}

		.box1 {

			width: 200px;
			height: 200px;
			background-color: #bfa;
			border-width: thin thick 30px medium;
			border-color: black yellow brown salmon;
			border-style: solid double dotted dashed;
		}

		.box2 {

			width: 200px;
			height: 200px;
			background-color: #bfa;
			border-width: thin thick 20px;
			border-color: black yellow brown;
			border-style: solid double dotted;
		}

		.box3 {

			width: 200px;
			height: 200px;
			background-color: #bfa;
			border-width: thin  20px;
			border-color: black yellow ;
			border-style: solid dotted;
		}
	</style>
</head>

<body>

	<div class="box"></div>
	<div class="box1"></div>
	<div class="box2"></div>
	<div class="box3"></div>

</body>

</html>
```

#### **padding**

padding用于控制content与border之间的距离，同时设置上下左右的padding时，可以这样写`padding:10px 20px 10px 10px;`分别对应上、右、下、左四个方向的padding，逆时针排序，margin属性也可以这样书写。

css中的padding是不能有负值出现的，出现的话就相当于把那个属性给取消了。

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: #bfa;
				/*设置边框*/
				border: 10px red solid;
				
				/*
				 * 内边距（padding），指的是盒子的内容区与盒子边框之间的距离
				  *padding: padding-top padding-right padding-bottom padding-left;
				 *一次定义上右下左的边框宽度，
				    如果缺少左边距的值，则使用右边距的值。
                    如果缺少下边距的值，则使用上边距的值。
                    如果缺少右边距的值，则使用上边距的值。
				 * 	一共有四个方向的内边距，可以通过：
				 * 		padding-top
				 * 		padding-right
				 * 		padding-bottom
				 * 		padding-left
				 * 			来设置四个方向的内边距
				 * 
				 * 内边距会影响盒子的可见框的大小，元素的背景会延伸到内边距,
				 * 	盒子的大小由内容区、内边距和边框共同决定
				 * 	盒子可见框的宽度 = border-left-width + padding-left + width + padding-right + border-right-width
				 *  可见宽的高度 = border-top-width + padding-top + height + padding-bottom + border-bottom-width
				 */
				
				/*设置上内边距*/
				/*padding-top: 100px;*/
				/*设置右内边距*/
				/*padding-right: 100px;
				padding-bottom: 100px;
				padding-left: 100px;*/
				
				/*
				 * 使用padding可以同时设置四个边框的样式，规则和border-width一致
				 */
				/*padding: 100px;*/
				
				/*padding: 100px 200px;*/
				
				/*padding: 100px 200px 300px;*/
				
				padding: 100px 200px 300px 400px;
			}
			
			/*
			 * 创建一个子元素box1占满box2
			 */
			.box2{
				width: 100%;
				height: 100%;
				background-color: yellow;
			}
			
		</style>
	</head>
	<body>
		
		<div class="box1">
			<div class="box2"></div>
		</div>
		
	</body>
</html>

```

#### margin

margin用于控制块与块（可以理解成块级元素）之间的距离。为了便于理解可以把盒子模型想象成一幅画，content是画本身，padding是画与画框的留白，border是画框，margin是画与画之间距离。需要注意的是IE和firefox在处理margin时有一些差别，倘若设定了父元素的高度height值，如果此时子元素的高度超过了父元素的height值，二则显示结果完全不同，IE浏览器会自动扩大，而firefox（火狐浏览器）就不会，这一点是需要注意的。

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
		/*
				 * 外边距指的是当前盒子与其他盒子之间的距离，
				 * 	他不会影响可见框的大小，而是会影响到盒子的位置。
				 * margin:margin-top margin-right margin-bottom margin-left

					如果缺少左外边距的值，则使用右外边距的值。
					如果缺少下外边距的值，则使用上外边距的值。
					如果缺少右外边距的值，则使用上外边距的值。

				 * 盒子有四个方向的外边距：
				 * 	margin-top
				 * 	margin-right
				 * 	margin-bottom
				 * 	margin-left
				 * 
				 * 由于页面中的元素都是靠左靠上摆放的，
				 * 	所以注意当我们设置上和左外边距时，会导致盒子自身的位置发生改变，
				 * 	而如果是设置右和下外边距会改变其他盒子的位置
				 */
				/*
				 * 设置box1的上外边距，盒子上边框和其他的盒子的距离
				 */
				/*margin-top: 100px;*/
				
				/*
				 * 左外边距
				 */
				/*margin-left: 100px;*/
				
				/*设置右和下外边距*/
				/*margin-right: 100px;
				margin-bottom: 100px;*/
				
				/*
				 * 外边距也可以指定为一个负值，
				 * 	如果外边距设置的是负值，则元素会向反方向移动
				 */
				/*margin-left: -150px;
				margin-top: -100px;
				margin-bottom: -100px;*/
				/*margin-bottom: -100px;*/
				
				/*
				 * margin还可以设置为auto，auto一般只设置给水平方向的margin
				 * 	如果只指定，左外边距或右外边距的margin为auto则会将外边距设置为最大值
				 * 	垂直方向外边距如果设置为auto，则外边距默认就是0
				 * 
				 * 如果将left和right同时设置为auto，则会将两侧的外边距设置为相同的值，
				 * 	就可以使元素自动在父元素中居中，所以我们经常将左右外边距设置为auto
				 * 	以使子元素在父元素中水平居中
				 * 
				 */
				
				/*margin-left: auto;
				margin-right: auto;*/
				
				/*
				 * 外边距同样可以使用简写属性 margin，可以同时设置四个方向的外边距,
				 * 	规则和padding一样
				 */
			
			.box1{
				width: 100px;
				height: 100px;
				background-color: red;
				/*
				 * 为上边的元素设置一个下外边距
				 */
				margin-bottom: 100px;
			}
			
			/*
			 * 垂直外边距的重叠
			 * 	- 在网页中相邻的垂直方向的外边距会发生外边距的重叠
			 * 		所谓的外边距重叠指兄弟元素之间的相邻外边距会取最大值而不是取和
			 * 		如果父子元素的垂直外边距相邻了，则子元素的外边距会设置给父元素
			 */
			
			.box2{
				width: 100px;
				height: 100px;
				background-color: green;
				/**
				 * 为下边的元素设置一个上外边距
				 */
				margin-top: 100px;
			}
			
			.box3{
				width: 200px;
				height: 100px;
				background-color: yellow;
				
				/*为box3设置一个上边框*/
				/*border-top: 1px red solid;*/
				/*padding-top: 1px;*/
				
				padding-top: 100px;
			}
			
			.box4{
				width: 100px;
				height: 100px;
				background-color: yellowgreen;
				/*
				 * 为子元素设置一个上外边距，是子元素的位置下移
				 */
				/*margin-top: 100px;*/
			}
			
		</style>
	</head>
	<body>
		

		
		<div class="box1"></div>
		<div class="box2"></div>
		<div class="box3">
			<div class="box4"></div>
		</div>
	</body>
</html>
```

### 内联元素的盒子

1. 内联元素不能设置width和height；
2. 设置水平内边距,内联元素可以设置水平方向的内边距：padding-left，padding-right；
3. 垂直方向内边距，内联元素可以设置垂直方向内边距，但是不会影响页面的布局；
4. 为元素设置边框,内联元素可以设置边框，但是垂直的边框不会影响到页面的布局；
5. 水平外边距内联元素支持水平方向的外边距；
6. 内联元素不支持垂直外边距；
7. 为右边的元素设置一个左外边距，水平方向的相邻外边距不会重叠，而是求和。

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			span{
				background-color: #bfa;
			}
			
			.box1{
				width: 100px;
				height: 100px;
				background-color: red;
			}
			
			.s1{
				/*
				 	内容区、内边距 、边框 、外边距
				 * */
				
				/*
				 * 内联元素不能设置width和height
				 */
				/*width: 200px;
				height: 200px;*/
				
				/*
				 * 设置水平内边距,内联元素可以设置水平方向的内边距
				 */
				padding-left: 100px ;
				padding-right: 100px ;
				
				/*
				 * 垂直方向内边距，内联元素可以设置垂直方向内边距，但是不会影响页面的布局
				 */
				/*padding-top: 50px;
				padding-bottom: 50px;*/
				
				/*
				 * 为元素设置边框,
				 * 	内联元素可以设置边框，但是垂直的边框不会影响到页面的布局
				 */
				border: 1px blue solid;
				
				/*
				 * 水平外边距
				 * 	内联元素支持水平方向的外边距
				 */
				margin-left:100px ;
				margin-right: 100px;
				
				/*
				 * 内联元素不支持垂直外边距
				 */
				/*margin-top: 200px;
				margin-bottom: 200px;*/
				
			}
			
			.s2{
				/*
				 * 为右边的元素设置一个左外边距
				 * 水平方向的相邻外边距不会重叠，而是求和
				 */
				margin-left: 100px;
			}
			
			
		</style>
	</head>
	<body>
		<span class="s1">我是一个span</span>
		<span class="s2">我是一个span</span>
		<span>我是一个span</span>
		<span>我是一个span</span>
		
		<div class="box1"></div>
	</body>
</html>
```

#### 内联元素的修改

##### display

虽然我们不能为内联元素width、height、margin-top和margin-bottom，但是我们可以通过修改display来修改元素的性质，使其或者这种能力。

display的可选项

```
 block：设置元素为块元素
 inline：设置元素为行内元素
 inline-block：设置元素为行内块元素（即它是行内元素，不占行，但是能对其设置）
 none：隐藏元素（元素将在页面中完全消失）
```

##### visibility

visibility属性主要用于元素是否可见。

> 和display不同，使用visibility隐藏一个元素，隐藏后其在文档中所占的位置会依然保持，不会被其他元素覆盖。

可选项

```
- visible，默认值，不会对溢出内容做处理，元素会在父元素以外的位置显示
- hidden, 溢出的内容，会被修剪，不会显示
- scroll, 会为父元素添加滚动条，通过拖动滚动条来查看完整内容 该属性不论内容是否溢出，都会添加水平和垂直双方向的滚动条
- auto，会根据需求自动添加滚动条，需要水平就添加水平需要垂直就添加垂直都不需要就都不加

```

示例：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>display和visibility</title>
    <style>
        a{
            /*可以设置成块元素。*/
            display: block;
            width: 50px;
            height: 50px;
            background-color:red;
        }
        div{
            /*这里可以把块元素转变成行内块元素
            <img>标签是行内块元素。即可以设置宽高，但是又不独占一行。
            */
            display: inline;
            width: 50px;
            height: 100px;
            background-color:blue;
        }
        /*display中的none和visibility中的hidden
        虽然都有隐藏元素的功能，但是，none就是直接隐藏，
        连位置也消失了，而hidden虽然隐藏了，但是位置却保留着。*/
    </style>
</head>
<body>
            <a href="https://www.baidu.com">百度</a>
            <div>我是一个块</div>
</body>
</html>
```

##### overflow

当相关标签里面的内容超出了样式的宽度 和高度是，就会发生一些奇怪的事情，浏 览器会让内容溢出盒子。

> 可以通过overflow来控制内容溢出的情况。

可选值：

```
 visible：默认值
 scroll：添加滚动条
 auto：根据需要添加滚动条
 hidden：隐藏超出盒子的内容
```

给父元素设置：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>溢出</title>
    <style>
        .box1{
            width: 100px;
            height: 200px;
            background-color: red;
            /*自动设置滚动条，根据情况来定*/
            overflow: auto;
            /*添加滚动条，横向和纵向都有*/
            /*overflow: scroll;*/
        }
    </style>
</head>
<body>
    <div class="box1">
        <p>
            1.形散神聚:"形散"既指题材广泛、写法多样，又指结构自由、不拘一格;"神聚"既指中心集中，又指有贯穿全文的线索。散文写人写事都只是表面现象，从根本上说写的是情感体验。情感体验就是"不散的神"，而人与事则是"散"的可有可无、可多可少的"形"。
            2."形散"主要是说散文取材十分广泛自由，不受时间和空间的限制;表现手法不拘一格:可以叙述事件的发展，可以描写人物形象，可以托物抒情，可以发表议论，而且作者可以根据内容需要自由调整、随意变化。"神不散"主要是从散文的立意方面说的，即散文所要表达的主题必须明确而集中，无论散文的内容多么广泛，表现手法多么灵活，无不为更好的表达主题服务。
            3.意境深邃:注重表现作者的生活感受，抒情性强，情感真挚。
            作者借助想象与联想，由此及彼，由浅入深，由实而虚的依次写来，可以融情于景、寄情于事、寓情于物、托物言志，表达作者的真情实感，实现物我的统一，展现出更深远的思想，使读者领会更深的道理。
            4.语言优美:所谓优美，就是指散文的语言清新明丽(也美丽)，生动活泼，富于音乐感，行文如涓涓流水，叮咚有声，如娓娓而谈，情真意切。所谓凝练，是说散文的语言简洁质朴，自然流畅，寥寥数语就可以描绘出生动的形象，勾勒出动人的场景，显示出深远的意境。散文力求写景如在眼前，写情沁人心脾。
        </p>
    </div>
</body>
</html>
```



### 默认样式

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 浏览器为了在页面中没有样式时，也可以有一个比较好的显示效果，
			 * 	所以为很多的元素都设置了一些默认的margin和padding，
			 * 	而它的这些默认样式，正常情况下我们是不需要使用的。
			 * 
			 * 所以我们往往在编写样式之前需要将浏览器中的默认的margin和padding统统的去掉
			 * 	
			 */
			
			/*
			 * 清除浏览器的默认样式
			 */
			
			*{
				margin: 0;
				padding: 0;
			}
		
			
			.box1{
				width: 100px;
				height: 100px;
				background-color: #bfa;
			}
			
			p{
				background-color: yellow;
			}
			
			
		</style>
	</head>
	<body>
		<div class="box1"></div>
		
		<p>我是一个段落</p>
		<p>我是一个段落</p>
		<p>我是一个段落</p>
		
		<ul>
			<li>无序列表</li>
			<li>无序列表</li>
			<li>无序列表</li>
			<li>无序列表</li>
		</ul>
	</body>
</html>
```

### 文档流

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>
	<body>
		
		<!-- 
			文档流
				文档流处在网页的最底层，它表示的是一个页面中的位置，
				 我们所创建的元素默认都处在文档流中
		文档流指的是元素排版布局过程中，元素会默认自动从左往右，从上往下的流式排列方式。
		并最终窗体自上而下分成一行行，并在每行中从左至右的顺序排放元素。
				 
			元素在文档流中的特点
				块元素
					1.块元素在文档流中会独占一行，块元素会自上向下排列。
					2.块元素在文档流中默认宽度是父元素的100%
					3.块元素在文档流中的高度默认被内容撑开
				内联元素
					1.内联元素在文档流中只占自身的大小，会默认从左向右排列，
						如果一行中不足以容纳所有的内联元素，则换到下一行，
						继续自左向右。
					2.在文档流中，内联元素的宽度和高度默认都被内容撑开	
		-->
		
		<!-- 
			当元素的宽度的值为auto时，此时指定内边距不会影响可见框的大小，
				而是会自动修改宽度，以适应内边距
		-->
		<div style="background-color: #bfa;">
			<div style="height: 50px;"></div>
		</div>
		<div style="width: 100px; height: 100px; background-color: #ff0;"></div>
		
		
		<span style="background-color: yellowgreen;">我是一个span</span>
		<span style="background-color: yellowgreen;">我是一个span</span>
		<span style="background-color: yellowgreen;">我是一个span</span>
		<span style="background-color: yellowgreen;">我是一个span</span>
		<span style="background-color: yellowgreen;">我是一个span</span>
		<span style="background-color: yellowgreen;">我是一个span</span>
	</body>
</html>
```

### 浮动

语法：`float:left | right`
 特点:

- 给一个元素设置了浮动后，该元素不占位置（脱离标准流）
- 给一个元素设置了浮动，会影响该元素后面的元素
- 浮动可以让块级元素在一行上显示
- 浮动可是实现模式转换（span 设置浮动可以设置宽高）

浮动作用:

- 浮动最初的作用：  解决文字图片环绕效果。（文字不会浮动的影响）
- 制作网页导航（浮动+列表实现导航）
- 进行网页布局 （div+CSS(盒子模型+浮动+定位)）

注意：
 让块级元素在一行上显示就使用浮动
 如果一个元素要实现模式转换，就使用display

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: red;
				/*
				 * 块元素在文档流中默认垂直排列，所以这个三个div自上至下依次排开，
				 * 	如果希望块元素在页面中水平排列，可以使块元素脱离文档流
				 * 使用float来使元素浮动，从而脱离文档流
				 * 	可选值：
				 * 		none，默认值，元素默认在文档流中排列
				 * 		left，元素会立即脱离文档流，向页面的左侧浮动
				 * 		right，元素会立即脱离文档流，向页面的右侧浮动
				 * 
				 * 当为一个元素设置浮动以后（float属性是一个非none的值），
				 * 	元素会立即脱离文档流，元素脱离文档流以后，它下边的元素会立即向上移动
				 * 	元素浮动以后，会尽量向页面的左上或这是右上漂浮，
				 * 	直到遇到父元素的边框或者其他的浮动元素
				 * 	如果浮动元素上边是一个没有浮动的块元素，则浮动元素不会超过块元素
				 * 	浮动的元素不会超过他上边的兄弟元素，最多最多一边齐
				 */
				float: left;
			}
			
			.box2{
				width: 600px;
				height: 200px;
				background-color: yellow;
				
				float: left;
			}
			
			.box3{
				width: 200px;
				height: 200px;
				background-color: green;
				
				float: right;
			}
					/*
				 * 浮动的元素不会盖住文字，文字会自动环绕在浮动元素的周围，
				 * 	所以我们可以通过浮动来设置文字环绕图片的效果
				 */ 
			.p1{
				background-color: yellow;
			}
			.box4{
				/*
				 * 在文档流中，子元素的宽度默认占父元素的全部
				 */
				background-color: #bfa;
				
				/*
				 * 当元素设置浮动以后，会完全脱离文档流.
				 * 	块元素脱离文档流以后，高度和宽度都被内容撑开
				 */
				/*float: left;*/
			}
			
			.s1{
				/*
				 * 开启span的浮动
				 * 	内联元素脱离文档流以后会变成块元素
				 */
				float: left;
				width: 100px;
				height: 100px;
				background-color: yellow;
			}
			
		</style>
	</head>
	<body>
		
		<div class="box1"></div>
		<div class="box2"></div>
		<div class="box3"></div>
				<p class="p1">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。
这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。
我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。
枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。
鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。
哇的一声，夜游的恶鸟飞过了。
		</p>
			<div class="box4">a</div>
		
		<span class="s1">hello</span>
	</body>
</html>

```

#### 清除浮动

什么时候需要清除浮动：

- 父元素（内容父盒子）没有设置高度（必须条件）
- 该父元素中的所有子元素都设置了浮动（必须条件）

当父容器没有设置高度，里面的盒子没有设置浮动的情况下会将父容器的高度撑开。一旦父容器中的盒子设置浮动，脱离标准文档流，父容器立马没有高度，下面的盒子会跑到浮动的盒子下面。出现这种情况，我们需要清除浮动

**清除浮动的方式**

1. 给父容器设置高度
2. 通过设置clear:left | right  | both
3. 给父容器设置 overflow:hidden
    如果父元素中没有定位的元素，或者定位的元素没有超出父元素的范围，那么可以使用overflow：hidden.
4. 通过伪元素

```
 .clearfix:after{
       content:"";
       height:0; line-height:0;
       visibily:hidden;
       clear:both;
       display:block;
  }
.clearfix{
   zoom:1    　　/*为了兼容IE浏览器 */
 }
```

##### 示例

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		
		<style type="text/css">
			
			.box1{
				width: 100px;
				height: 100px;
				background-color: yellow;
				/*
				 * 设置box1向左浮动
				 */
				float: left;
			}
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellowgreen;
				/*
				 * 由于受到box1浮动的影响，box2整体向上移动了100px
				 * 我们有时希望清除掉其他元素浮动对当前元素产生的影响，这时可以使用clear来完成功能
				 * clear可以用来清除其他浮动元素对当前元素的影响
				 * 可选值：
				 * 		none，默认值，不清除浮动
				 * 		left，清除左侧浮动元素对当前元素的影响
				 * 		right，清除右侧浮动元素对当前元素的影响
				 * 		both，清除两侧浮动元素对当前元素的影响
				 * 				清除对他影响最大的那个元素的浮动
				 */
				
				/*
				 * 清除box1浮动对box2产生的影响
				 * 清除浮动以后，元素会回到其他元素浮动之前的位置
				 */
				/* clear: left; */
				/* float: left; */
			}
			.box3{
				width: 300px;
				height: 300px;
				background-color: skyblue;
				
				clear: both;
			}
			
		</style>
		
	</head>
	<body>
		
		<div class="box1"></div>
		
		<div class="box2"></div>
		
		<div class="box3"></div>
		
	</body>
</html>
```



### 简单布局

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*清除默认样式*/
			*{
				margin: 0;
				padding: 0;
			}
			
			/*设置头部div*/
			.header{
				/*设置一个宽度*/
				width: 1000px;
				/*设置一个高度*/
				height: 120px;
				/*设置一个背景颜色*/
				background-color: yellowgreen;
				/*设置居中*/
				margin: 0 auto;
			}
			
			/*设置一个content*/
			.content{
				/*设置一个宽度*/
				width: 1000px;
				/*设置一个高度*/
				height: 400px;
				/*设置一个背景颜色*/
				background-color: orange;
				/*居中*/
				margin: 10px auto;
			}
			
			/*设置content中小div的样式*/
			.left{
				width: 200px;
				height: 100%;
				background-color: skyblue;
				/*向左浮动*/
				float: left;
			}
			
			.center{
				width: 580px;
				height: 100%;
				background-color: yellow;
				/*向左浮动*/
				float: left;
				/*设置水平外边距*/
				margin: 0 10px;
			}
			
			.right{
				width: 200px;
				height: 100%;
				background-color: pink;
				/*向左浮动*/
				float: left;
			}
			
			
			
			/*设置一个footer*/
			.footer{
				/*设置一个宽度*/
				width: 1000px;
				/*设置一个高度*/
				height: 120px;
				/*设置一个背景颜色*/
				background-color: silver;
				/*居中*/
				margin: 0 auto;
			}
			
		</style>
	</head>
	<body>
		<!-- 头部div -->
		<div class="header"></div>
		
		<!-- 主体内容div -->
		<div class="content">
			
			<!-- 左侧div -->
			<div class="left"></div>
			<!-- 中间div -->
			<div class="center"></div>
			<!-- 右侧div -->
			<div class="right"></div>
			
		</div>
		
		<!-- 底部信息div -->
		<div class="footer"></div>
		
	</body>
</html>
```

### 高度塌陷

在文档流中，一个块级元素如果没有设置height，其height是由子元素撑开的。也就是子元素多高，父元素就多高。
但是为子元素设置浮动后，子元素会完全脱离文档流，此时会导致子元素无法撑起父元素的高度，导致父元素的高度塌陷。
由于父元素的高度塌陷了，则父元素下的所有元素都会向上移动，这样将会导致页面布局混乱，也就是所谓的高度塌陷。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title></title>
    <style type="text/css">
        .par {
            border: 1px solid green;
        } 
        .sub {
            width: 20%;
            height: 50px;
            float: left;
            border: 1px solid red;
        }
        .only {
            width: 30%;
            height: 60px;
            background: #999;
        }
    </style>
</head>
<body>
    <div class="par">
        <div class="sub div1"></div>
        <div class="sub div2"></div>
        <div class="sub div3"></div>
    </div>
    <div class="only"></div>
</body>
</html>
```

当sub这个类选择器设置了float: left的时候，可以看出来，好像它们都挤到一块去了，而class="par"的这个外部div的height很明显是等于0的，它里面的div是浮动排列的，而class="only"的这个div可以看出来是“跑到”它上面的元素的位置去的，就好像它上面的元素都并不存在似的

**BFC是什么？**

根据W3C的标准，在页面中元素都一个隐含的属性叫做Block Formatting Context简称BFC，该属性可以设置打开或者关闭，默认是关闭的，触发了BFC就可以消除（闭合）浮动。当开启元素的BFC以后，元素将会具有如下的特性：

1. 父元素的垂直外边距不会和子元素重叠
2. 开启BFC的元素不会被浮动元素所覆盖
3. 开启BFC的元素可以包含浮动的子元素

触发BFC的情况：

- float的值不为none的时候，使用这种方式开启，虽然可以撑开父元素，但是会导致父元素的宽度丢失，而且使用这种方式也会导致下边的元素上移，不能解决问题
- overflow的值为auto，scroll或者hidden
- display的值为table-cell，table-caption，inline-block，可以解决问题，但是会导致宽度丢失，不推荐使用这种方式
- position的值不是relative和static

推荐方式：将overflow设置为hidden是副作用最小的开启BFC的方式。

**避免高度塌陷的方法**

1. 给父级div定义高度
    原理：给父级DIV定义固定高度，能解决父级div无法获取高度的问题。
    优点：代码简洁
    缺点：高度固定，适合高度固定不变的页面，不适合响应式网站。（不推荐使用）

2. 使用空元素，如`<div class="clear"></div>` (.clear{clear:both})
    原理：添加空的div标签，利用css的clear:both属性清除浮动，让父级div能够获取高度。
    优点：浏览器兼容性与支持都是很好的。
    缺点：多了很多空div标签，如果页面中浮动模块多的话，就会出现很多的空div标签，对于代码的维护与开发有很大的干扰。（不推荐使用）

3. 父级div定义 display:table
    原理：将div属性强制变成表格
    优点：暂时不知
    缺点：会产生新的未知问题。（不推荐使用）

4. 父元素设置 overflow：hidden、auto；
    原理：这个方法的关键在于触发了BFC。在IE6中还需要触发 hasLayout（zoom：1）
    优点：代码简介，不存在结构和语义化问题
    缺点：无法显示需要溢出的元素（亦不太推荐使用）

5. 让父元素本身也浮动

   ```
   .par {
     border: 1px solid green;
     float: left;
   } 
   ```

   缺点：使得跟父元素相邻的元素的布局受到影响，不推荐使用

6. 使用CSS的after伪元素，推荐

   ```
   .clearfix:after {
     content: ".";         /*生成内容作为最后一个元素，至于content里面是什么没有影响*/
     display: block;       /*使得生成的元素以块级元素显示，占满剩余空间*/
     height: 0;            /*避免生成的内容破坏原有空间的高度*/
     clear: both;          /*闭合浮动*/
     visibility: hidden;   /*使得生成内容不可见，并允许可能生成内容盖住的内容进行点击和交互*/
   }
   .clearfix {
     zoom : 1;             /*触发IE6/7的haslayout*/
   }
   ```

#### 示例：

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>导航条</title>
		<style type="text/css">
			
			/*
			 * 清除默认样式
			 */
			*{
				margin: 0;
				padding: 0;
			}
			
			/*
			 * 设置ul
			 */
			.nav{
				/*去除项目符号*/
				list-style: none;
				/*为ul设置一个背景颜色*/
				background-color: #6495ED;
				/*设置一个宽度*/ 
				/*
				 * 在IE6中，如果为元素指定了一个宽度，则会默认开启hasLayout
				 */
				
				max-width: 1000px;
				/*设置元素居中*/
				margin: 50px auto;
				/*解决高度塌陷*/
				overflow: hidden;
			}
			
			/**
			 * 设置li
			 */
			.nav li{
				/*设置li向左浮动*/
				float: left;
				width: 12.5%;
			}
			
			.nav a{
				/*将a转换为块元素*/
				display: block;
				/*为a指定一个宽度*/
				width: 100%;
				/*设置文字居中*/
				text-align: center;
				/*设置一个上下内边距*/
				padding: 5px 0;
				/*去除下划线*/
				text-decoration: none;
				/*设置字体颜色*/
				color: white;
				/*设置加粗*/
				font-weight: bold;
			}
			
			/*
			 * 设置a的鼠标移入的效果
			 */
			.nav a:hover{
				background-color: #fff;
				color: #000;
			}
			
			
		</style>
	</head>
	<body>
		
		<!-- 创建导航条的结构 -->
		<ul class="nav">
			<li><a href="#">首页</a></li>
			<li><a href="#">新闻</a></li>
			<li><a href="#">联系</a></li>
			<li><a href="#">关于</a></li>
			<li><a href="#">首页</a></li>
			<li><a href="#">新闻</a></li>
			<li><a href="#">联系</a></li>
			<li><a href="#">关于</a></li>
		</ul>
		
	</body>
</html>
```

### 定位

定位体现的是元素之间的层级关系（盒子压盒子的效果），通过定位也可以移动元素

css定位主要有四种，静态定位、相对定位、绝对定位和固定定位。其中静态定位这个是元素的默认定位方式，不能使用top，bottom，left，right和z-index属性，其它三种定位可以使用以上几个属性。我们这里主要介绍后边的这三个定位。

#### 相对定位

如果想为元素设置层模型中的相对定位，需要设置position:relative;，它还是会占用该元素在文档中初始的页面空间，通过left、right、top、bottom属性确定元素在正常文档流中的偏移位置，然后**相对于以前的位置移动**，移动的方向和幅度由left、right、top、bottom属性确定。使用相对定位的元素不管它是否进行移动，元素仍要占据它原来的位置。移动元素会导致它覆盖其他的框。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: red;
			}
			
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellow;
				/*
				 * 定位：
				 * 	- 定位指的就是将指定的元素摆放到页面的任意位置
				 * 		通过定位可以任意的摆放元素
				 * 	- 通过position属性来设置元素的定位
				 * 		-可选值：
				 * 			static：默认值，元素没有开启定位
				 * 			relative：开启元素的相对定位
				 * 			absolute：开启元素的绝对定位
				 * 			fixed：开启元素的固定定位（也是绝对定位的一种）
				 */
				
				/*
				 * 当元素的position属性设置为relative时，则开启了元素的相对定位
				 * 	1.当开启了元素的相对定位以后，而不设置偏移量时，元素不会发生任何变化
				 *  2.相对定位是相对于元素在文档流中原来的位置进行定位
				 * 	3.相对定位的元素不会脱离文档流
				 * 	4.相对定位会使元素提升一个层级
				 * 	5.相对定位不会改变元素的性质，块还是块，内联还是内联
				 */
				position: relative;
				
				
				/*
				 * 当开启了元素的定位（position属性值是一个非static的值）时，
				 * 	可以通过left right top bottom四个属性来设置元素的偏移量
				 * 	left：元素相对于其定位位置的左侧偏移量
				 * 	right：元素相对于其定位位置的右侧偏移量
				 * 	top：元素相对于其定位位置的上边的偏移量
				 * 	bottom：元素相对于其定位位置下边的偏移量
				 * 
				 * 通常偏移量只需要使用两个就可以对一个元素进行定位，
				 * 	一般选择水平方向的一个偏移量和垂直方向的偏移量来为一个元素进行定位
				 */
				
				/*left: 100px;
				top: 200px;*/
				
				left: 200px;
				
			}
			
			.box3{
				width: 200px;
				height: 200px;
				background-color: yellowgreen;
				
			}
			
			.s1{
				position: relative;
				width: 200px;
				height: 200px;
				background-color: yellow;
			}
			
		</style>
		
	</head>
	<body>
		
		<div class="box1"></div>
		<div class="box2"></div>
		<div class="box3"></div>
		
		<span class="s1">我是一个span</span>


		
	</body>
</html>
```

#### 绝对定位

相对于已定位的最近的祖先元素，如果没有已定位的最近的祖先元素，那么它的位置就相对于最初的包含块（如body）。绝对定位的框可以从它的包含块向上、右、下、左移动。
绝对定位的框脱离普通流，所以它可以覆盖页面上的其他元素，可以通过设置Ｚ-Iindex属性来控制这些框的堆放次序。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: red;
			}
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellow;
				/*
				 * 当position属性值设置为absolute时，则开启了元素的绝对定位
				 * 
				 * 绝对定位：
				 * 	1.开启绝对定位，会使元素脱离文档流
				 * 	2.开启绝对定位以后，如果不设置偏移量，则元素的位置不会发生变化
				 * 	3.绝对定位是相对于离他最近的开启了定位的祖先元素进行定位的（一般情况，开启了子元素的绝对定位都会同时开启父元素的相对定位）
				 * 		如果所有的祖先元素都没有开启定位，则会相对于浏览器窗口进行定位
				 * 	4.绝对定位会使元素提升一个层级
				 * 	5.绝对定位会改变元素的性质，
				 * 		内联元素变成块元素，
				 * 		块元素的宽度和高度默认都被内容撑开
				 */
				position: absolute;
				
				/*left: 100px;
				top: 100px;*/
				
			}
			.box3{
				width: 300px;
				height: 300px;
				background-color: yellowgreen;
			}
			
			.box4{
				width: 300px;
				height: 300px;
				background-color: orange;
				/*开启box4的相对定位*/
				/*position: relative;*/
			}
			
			.s1{
				width: 100px;
				height: 100px;
				background-color: yellow;
				
				/*开启绝对定位*/
				position: absolute;
			}
			
		</style>
		
	</head>
	<body>
		
		<div class="box1"></div>
		<div class="box5">
			<div class="box4">
				<div class="box2"></div>
			</div>
		</div>
		
		<div class="box3"></div>
		
		<span class="s1">我是一个span</span>
	</body>
</html>

```

备注：相对于已相对定位的祖先元素对框进行绝对定位，在大多数现代浏览器中都可以实现的很好。

#### 固定定位

相对于浏览器窗口，其余的特点类似于绝对定位。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		
		<style type="text/css">
			
			.box1{
				width: 200px;
				height: 200px;
				background-color: red;
			}
			.box2{
				width: 200px;
				height: 200px;
				background-color: yellow;
				/*
				 * 当元素的position属性设置fixed时，则开启了元素的固定定位
				 * 	固定定位也是一种绝对定位，它的大部分特点都和绝对定位一样
				 * 不同的是：
				 * 		固定定位永远都会相对于浏览器窗口进行定位
				 * 		固定定位会固定在浏览器窗口某个位置，不会随滚动条滚动
				 * 
				 * IE6不支持固定定位
				 */
				position: fixed;
				
				left: 0px;
				top: 0px;
				
			}
			.box3{
				width: 200px;
				height: 200px;
				background-color: yellowgreen;
			}
			
		</style>
		
	</head>
	<body style="height: 5000px;">
		
		<div class="box1"></div>
		<div class="box4" style="width: 300px; height: 300px; background-color: orange; position: relative;">
			<div class="box2"></div>
		</div>
		
		<div class="box3"></div>
	</body>
</html>

```

