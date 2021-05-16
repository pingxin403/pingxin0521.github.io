---
title: JavaScript DOM
date: 2019-05-05 10:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

### DOM

<!--more-->

HTML DOM 定义了访问和操作 HTML 文档的标准方法。

DOM 将 HTML 文档表达为树结构。DOM 是 Document Object Model（文档对象模型）的缩写。

![](https://i.loli.net/2019/06/02/5cf3c319858f859519.png)

DOM 是 W3C（万维网联盟）的标准。

DOM 定义了访问 HTML 和 XML 文档的标准：

> “W3C 文档对象模型 （DOM） 是中立于平台和语言的接口，它允许程序和脚本动态地访问和更新文档的内容、结构和样式。”

W3C DOM 标准被分为 3 个不同的部分：

- 核心 DOM - 针对任何结构化文档的标准模型
- XML DOM - 针对 XML 文档的标准模型。XML DOM 定义了所有 XML 元素的*对象*和*属性*，以及访问它们的*方法*。
- HTML DOM - 针对 HTML 文档的标准模型

HTML DOM 是：

- HTML 的标准对象模型
- HTML 的标准编程接口
- W3C 标准

HTML DOM 定义了所有 HTML 元素的*对象*和*属性*，以及访问它们的*方法*。

*换言之，HTML DOM 是关于如何获取、修改、添加或删除 HTML 元素的标准。*

#### DOM 节点

根据 W3C 的 HTML DOM 标准，HTML 文档中的所有内容都是节点：

- 整个文档是一个文档节点
- 每个 HTML 元素是元素节点
- HTML 元素内的文本是文本节点
- 每个 HTML 属性是属性节点
- 注释是注释节点

HTML DOM 将 HTML 文档视作树结构。这种结构被称为*节点树*：

![](https://i.loli.net/2019/06/02/5cf3c319858f859519.png)

通过 HTML DOM，树中的所有节点均可通过 JavaScript 进行访问。所有 HTML 元素（节点）均可被修改，也可以创建或删除节点。

**节点父、子和同胞**

节点树中的节点彼此拥有层级关系。

父（parent）、子（child）和同胞（sibling）等术语用于描述这些关系。父节点拥有子节点。同级的子节点被称为同胞（兄弟或姐妹）。

- 在节点树中，顶端节点被称为根（root）
- 每个节点都有父节点、除了根（它没有父节点）
- 一个节点可拥有任意数量的子
- 同胞是拥有相同父节点的节点

下面的图片展示了节点树的一部分，以及节点之间的关系：

![](https://i.loli.net/2019/06/02/5cf3c5978b2f699088.png)

```
<html>
  <head>
    <title>DOM 教程</title>
  </head>
  <body>
    <h1>DOM 第一课</h1>
    <p>Hello world!</p>
  </body>
</html>

```

从上面的 HTML 中：

- `<html> 节点没有父节点；它是根节点`
- `<head> 和 <body> 的父节点是 <html> 节点`
- `文本节点 "Hello world!" 的父节点是 <p> 节点`

并且

- `<html> 节点拥有两个子节点：<head> 和 <body>`
- `<head> 节点拥有一个子节点：<title> 节点`
- `<title> 节点也拥有一个子节点：文本节点 "DOM 教程"`
- `<h1> 和 <p> 节点是同胞节点，同时也是 <body> 的子节点`

并且：

- `<head> 元素是 <html> 元素的首个子节点`

- `<body> 元素是 <html> 元素的最后一个子节点`

- `<h1> 元素是 <body> 元素的首个子节点`

- <p> 元素是 <body> 元素的最后一个子节点

#### DOM 方法

方法是我们可以在节点（HTML 元素）上执行的动作。

**编程接口**

可通过 JavaScript （以及其他编程语言）对 HTML DOM 进行访问。

所有 HTML 元素被定义为对象，而编程接口则是对象方法和对象属性。

方法是您能够执行的动作（比如添加或修改元素）。

属性是您能够获取或设置的值（比如节点的名称或内容）。

**getElementById() 方法**

getElementById() 方法返回带有指定 ID 的元素：

```
var element=document.getElementById("intro");
```

#### 属性

一些常用的 HTML DOM 方法：

- getElementById(id) - 获取带有指定 id 的节点（元素）
- appendChild(node) - 插入新的子节点（元素）
- removeChild(node) - 删除子节点（元素）

一些常用的 HTML DOM 属性：

- innerHTML - 节点（元素）的文本值
- parentNode - 节点（元素）的父节点
- childNodes - 节点（元素）的子节点
- attributes - 节点（元素）的属性节点



**nodeName** 属性规定节点的名称。

- nodeName 是只读的
- 元素节点的 nodeName 与标签名相同
- 属性节点的 nodeName 与属性名相同
- 文本节点的 nodeName 始终是 #text
- 文档节点的 nodeName 始终是 #document

nodeName 始终包含 HTML 元素的大写字母标签名。



**nodeValue** 属性规定节点的值。

- 元素节点的 nodeValue 是 undefined 或 null
- 文本节点的 nodeValue 是文本本身
- 属性节点的 nodeValue 是属性值

```
<html>
<body>

<p id="intro">Hello World!</p>

<script type="text/javascript">
x=document.getElementById("intro");
document.write(x.firstChild.nodeValue);
</script>

</body>
</html>
```

**nodeType** 属性返回节点的类型。nodeType 是只读的。

比较重要的节点类型有：

| 元素类型 | NodeType |
| -------- | -------- |
| 元素     | 1        |
| 属性     | 2        |
| 文本     | 3        |
| 注释     | 8        |
| 文档     | 9        |

#### 对象方法

| 方法                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| getElementById()         | 返回带有指定 ID 的元素。                                     |
| getElementsByTagName()   | 返回包含带有指定标签名称的所有元素的节点列表（集合/节点数组）。 |
| getElementsByClassName() | 返回包含带有指定类名的所有元素的节点列表。                   |
| appendChild()            | 把新的子节点添加到指定节点。                                 |
| removeChild()            | 删除子节点。                                                 |
| replaceChild()           | 替换子节点。                                                 |
| insertBefore()           | 在指定的子节点前面插入新的子节点。                           |
| createAttribute()        | 创建属性节点。                                               |
| createElement()          | 创建元素节点。                                               |
| createTextNode()         | 创建文本节点。                                               |
| getAttribute()           | 返回指定的属性值。                                           |
| setAttribute()           | 把指定属性设置或修改为指定的值。                             |

 **访问 HTML 元素**

您能够以不同的方式来访问 HTML 元素：

- 通过使用 getElementById() 方法
- 通过使用 getElementsByTagName() 方法
- 通过使用 getElementsByClassName() 方法

```
node.getElementById("id");
node.getElementsByTagName("tagname");
document.getElementsByClassName("intro");
```

#### 修改

修改 HTML DOM 意味着许多不同的方面：

- 改变 HTML 内容
- 改变 CSS 样式
- 改变 HTML 属性
- 创建新的 HTML 元素
- 删除已有的 HTML 元素
- 改变事件（处理程序）

**创建 HTML 内容**

改变元素内容的最简答的方法是使用 innerHTML 属性

```
<html>
<body>

<p id="p1">Hello World!</p>

<script>
document.getElementById("p1").innerHTML="New text!";
</script>

</body>
</html>
```

**改变 HTML 样式**

通过 HTML DOM，您能够访问 HTML 元素的样式对象。

```
<html>

<body>
<p id="p2">Hello world!</p>

<script>
document.getElementById("p2").style.color="blue";
</script>

</body>
</html>
```

**创建新的 HTML 元素**

如需向 HTML DOM 添加新元素，您首先必须创建该元素（元素节点），然后把它追加到已有的元素上。

```
<div id="d1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
var para=document.createElement("p");
var node=document.createTextNode("This is new.");
para.appendChild(node);

var element=document.getElementById("d1");
element.appendChild(para);
</script>
```

**使用事件**

HTML DOM 允许您在事件发生时执行代码。

当 HTML 元素”有事情发生“时，浏览器就会生成事件：

- 在元素上点击
- 加载页面
- 改变输入字段

```
<html>
<body>

<input type="button" onclick="document.body.style.backgroundColor='lavender';"
value="Change background color" />
<script>
function ChangeBackground()
{
document.body.style.backgroundColor="lavender";
}
</script>

<input type="button" onclick="ChangeBackground()"
value="Change background color" />

<p id="p1">Hello world!</p>

<script>
function ChangeText()
{
document.getElementById("p1").innerHTML="New text!";
}
</script>

<input type="button" onclick="ChangeText()" value="Change text">

</body>
</html>
```

**创建新的 HTML 元素 - appendChild()**

如需向 HTML DOM 添加新元素，您首先必须创建该元素，然后把它追加到已有的元素上。

```
<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
var para=document.createElement("p");
var node=document.createTextNode("This is new.");
para.appendChild(node);

var element=document.getElementById("div1");
element.appendChild(para);
</script>
```

**创建新的 HTML 元素 - insertBefore()**

上一个例子中的 appendChild() 方法，将新元素作为父元素的最后一个子元素进行添加。

如果不希望如此，您可以使用 insertBefore() 方法：

```
<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
var para=document.createElement("p");
var node=document.createTextNode("This is new.");
para.appendChild(node);

var element=document.getElementById("div1");
var child=document.getElementById("p1");
element.insertBefore(para,child);
</script>
```

**删除已有的 HTML 元素**

如需删除 HTML 元素，您必须清楚该元素的父元素：

```
<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>
<script>
var parent=document.getElementById("div1");
var child=document.getElementById("p1");
parent.removeChild(child);
</script>
```

**替换 HTML 元素**

如需替换 HTML DOM 中的元素，请使用 replaceChild() 方法：

```
<div id="div1">
<p id="p1">This is a paragraph.</p>
<p id="p2">This is another paragraph.</p>
</div>

<script>
var para=document.createElement("p");
var node=document.createTextNode("This is new.");
para.appendChild(node);

var parent=document.getElementById("div1");
var child=document.getElementById("p1");
parent.replaceChild(para,child);
</script>
```

#### 事件

HTML DOM 允许 JavaScript 对 HTML 事件作出反应。。

**对事件作出反应**

当事件发生时，可以执行 JavaScript，比如当用户点击一个 HTML 元素时。

如需在用户点击某个元素时执行代码，请把 JavaScript 代码添加到 HTML 事件属性中：

```
onclick=JavaScript
```

HTML 事件的例子：

- 当用户点击鼠标时
- 当网页已加载时
- 当图片已加载时
- 当鼠标移动到元素上时
- 当输入字段被改变时
- 当 HTML 表单被提交时
- 当用户触发按键时

```
<!DOCTYPE html>
<html>
<body>
<h1 onclick="this.innerHTML='hello!'">请点击这段文本!</h1>
</body>
</html>
```

**HTML 事件属性**

如需向 HTML 元素分配事件，您可以使用事件属性。

向 button 元素分配一个 onclick 事件：

```
<!DOCTYPE html>
<html>
<body>

<p>点击按钮来执行 <b>displayDate()</b> 函数。</p>

<button onclick="displayDate()">试一试</button>

<script>
function displayDate()
{
document.getElementById("demo").innerHTML=Date();
}
</script>

<p id="demo"></p>

</body>
</html>

```

在上面的例子中，当点击按钮时，会执行名为 displayDate 的函数。

**使用 HTML DOM 来分配事件**

HTML DOM 允许您使用 JavaScript 向 HTML 元素分配事件：

为 button 元素分配 onclick 事件：

```
<!DOCTYPE html>
<html>
<head>
</head>
<body>

<p>点击按钮来执行 <b>displayDate()</b> 函数。</p>

<button id="myBtn">试一试</button>

<script>
document.getElementById("myBtn").onclick=function(){displayDate()};
function displayDate()
{
document.getElementById("demo").innerHTML=Date();
}
</script>

<p id="demo"></p>

</body>
</html>

```

在上面的例子中，名为 displayDate 的函数被分配给了 id=myButn" 的 HTML 元素。

当按钮被点击时，将执行函数。

**onload 和 onunload 事件**

当用户进入或离开页面时，会触发 onload 和 onunload 事件。

onload 事件可用于检查访客的浏览器类型和版本，以便基于这些信息来加载不同版本的网页。

onload 和 onunload 事件可用于处理 cookies。

```
<!DOCTYPE html>
<html>
<body onload="checkCookies()">

<script>
function checkCookies()
{
if (navigator.cookieEnabled==true)
	{
	alert("Cookies are enabled")
	}
else
	{
	alert("Cookies are not enabled")
	}
}
</script>

<p>弹出的提示框会告诉你浏览器是否已启用 cookie。</p>
</body>
</html>

```

**onchange 事件**

onchange 事件常用于输入字段的验证。

下面的例子展示了如何使用 onchange。当用户改变输入字段的内容时，将调用 upperCase() 函数。

```
<!DOCTYPE html>
<html>
<head>
<script>
function myFunction()
{
var x=document.getElementById("fname");
x.value=x.value.toUpperCase();
}
</script>
</head>
<body>

请输入你的英文名：<input type="text" id="fname" onchange="myFunction()">
<p>当你离开输入框时，被触发的函数会把你输入的文本转换为大写字母。</p>

</body>
</html>
```

**onmouseover 和 onmouseout 事件**

onmouseover 和 onmouseout 事件可用于在鼠标指针移动到或离开元素时触发函数。

```
<!DOCTYPE html>
<html>
<body>

<div 
onmouseover="mOver(this)" 
onmouseout="mOut(this)" 
style="background-color:#D94A38;width:200px;height:50px;padding-top:25px;text-align:center;">
Mouse Over Me
</div>

<script>
function mOver(obj)
{
obj.innerHTML="谢谢你"
}

function mOut(obj)
{
obj.innerHTML="把鼠标指针移动到上面"
}
</script>

</body>
</html>
```

**onmousedown、onmouseup 以及 onclick 事件**

onmousedown、onmouseup 以及 onclick 事件是鼠标点击的全部过程。首先当某个鼠标按钮被点击时，触发 onmousedown 事件，然后，当鼠标按钮被松开时，会触发 onmouseup 事件，最后，当鼠标点击完成时，触发 onclick 事件。

```
<!DOCTYPE html>
<html>
<body>

<div 
onmousedown="mDown(this)" 
onmouseup="mUp(this)" 
style="background-color:#D94A38;width:200px;height:50px;padding-top:25px;text-align:center;">
点击这里
</div>

<script>
function mDown(obj)
{
obj.style.backgroundColor="#1ec5e5";
obj.innerHTML="松开鼠标"
}

function mUp(obj)
{
obj.style.backgroundColor="#D94A38";
obj.innerHTML="谢谢你"
}
</script>

</body>
</html>
```

#### 导航

通过 HTML DOM，您能够使用节点关系在节点树中导航。

getElementsByTagName() 方法返回节点列表。节点列表是一个节点数组。

HTML DOM 节点列表长度

length 属性定义节点列表中节点的数量。

您能够使用三个节点属性：parentNode、firstChild 以及 lastChild ，在文档结构中进行导航。

这里有两个特殊的属性，可以访问全部文档：

- document.documentElement - 全部文档
- document.body - 文档的主体

除了 innerHTML 属性，您也可以使用 childNodes 和 nodeValue 属性来获取元素的内容。

#### 样式

更改CSS

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			#box1{
				width: 100px;
				height: 100px;
				background-color: red;
			}
			
		</style>
		
		<script type="text/javascript">
			
			window.onload = function(){
				
				/*
				 * 点击按钮以后，修改box1的大小
				 */
				//获取box1
				var box1 = document.getElementById("box1");
				//为按钮绑定单击响应函数
				var btn01 = document.getElementById("btn01");
				btn01.onclick = function(){
					
					//修改box1的宽度
					/*
					 * 通过JS修改元素的样式：
					 * 	语法：元素.style.样式名 = 样式值
					 * 
					 * 注意：如果CSS的样式名中含有-，
					 * 		这种名称在JS中是不合法的比如background-color
					 * 		需要将这种样式名修改为驼峰命名法，
					 * 		去掉-，然后将-后的字母大写
					 * 
					 * 我们通过style属性设置的样式都是内联样式，
					 * 	而内联样式有较高的优先级，所以通过JS修改的样式往往会立即显示
					 * 
					 * 但是如果在样式中写了!important，则此时样式会有最高的优先级，
					 * 	即使通过JS也不能覆盖该样式，此时将会导致JS修改样式失效
					 * 	所以尽量不要为样式添加!important
					 * 
					 * 
					 * 
					 */
					box1.style.width = "300px";
					box1.style.height = "300px";
					box1.style.backgroundColor = "yellow";
					
				};
				
				
				//点击按钮2以后，读取元素的样式
				var btn02 = document.getElementById("btn02");
				btn02.onclick = function(){
					//读取box1的样式
					/*
					 * 	语法：元素.style.样式名
					 * 
					 * 通过style属性设置和读取的都是内联样式
					 * 	无法读取样式表中的样式
					 */
					//alert(box1.style.height);
					alert(box1.style.width);
				};
			};
			
			
		</script>
	</head>
	<body>
		
		<button id="btn01">点我一下</button>
		<button id="btn02">点我一下2</button>
		
		<br /><br />
		
		<div id="box1"></div>
		
	</body>
</html>

```

读取CSS样式

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			#box1{
				width: 100px;
				height: 100px;
				background-color: yellow;
			}
			
		</style>
		
		<script type="text/javascript">
			
			window.onload = function(){
				
				//点击按钮以后读取box1的样式
				var box1 = document.getElementById("box1");
				var btn01 = document.getElementById("btn01");
				btn01.onclick = function(){
					//读取box1的宽度
					//alert(box1.style.width);
					
					/*
					 * 获取元素的当前显示的样式
					 * 	语法：元素.currentStyle.样式名
					 * 它可以用来读取当前元素正在显示的样式
					 * 	如果当前元素没有设置该样式，则获取它的默认值
					 * 
					 * currentStyle只有IE浏览器支持，其他的浏览器都不支持
					 */
					
					//alert(box1.currentStyle.width);
					//box1.currentStyle.width = "200px";
					//alert(box1.currentStyle.backgroundColor);
					
					/*
					 * 在其他浏览器中可以使用
					 * 		getComputedStyle()这个方法来获取元素当前的样式
					 * 		这个方法是window的方法，可以直接使用
					 * 需要两个参数
					 * 		第一个：要获取样式的元素
					 * 		第二个：可以传递一个伪元素，一般都传null
					 * 
					 * 该方法会返回一个对象，对象中封装了当前元素对应的样式
					 * 	可以通过对象.样式名来读取样式
					 * 	如果获取的样式没有设置，则会获取到真实的值，而不是默认值
					 * 	比如：没有设置width，它不会获取到auto，而是一个长度
					 * 
					 * 但是该方法不支持IE8及以下的浏览器
					 * 
					 * 通过currentStyle和getComputedStyle()读取到的样式都是只读的，
					 * 	不能修改，如果要修改必须通过style属性
					 */
					//var obj = getComputedStyle(box1,null);
					
					/*alert(getComputedStyle(box1,null).width);*/
					//正常浏览器的方式
					//alert(getComputedStyle(box1,null).backgroundColor);
					
					//IE8的方式
					//alert(box1.currentStyle.backgroundColor);
					
					//alert(getStyle(box1,"width"));
					
					var w = getStyle(box1,"width");
					alert(w);
					
					
				};
				
			};
			
			/*
			 * 定义一个函数，用来获取指定元素的当前的样式
			 * 参数：
			 * 		obj 要获取样式的元素
			 * 		name 要获取的样式名
			 */
			
			function getStyle(obj , name){
				
				if(window.getComputedStyle){
					//正常浏览器的方式，具有getComputedStyle()方法
					return getComputedStyle(obj , null)[name];
				}else{
					//IE8的方式，没有getComputedStyle()方法
					return obj.currentStyle[name];
				}
				
				//return window.getComputedStyle?getComputedStyle(obj , null)[name]:obj.currentStyle[name];
				
			}
			
			
		</script>
	</head>
	<body>
		<button id="btn01">点我一下</button>
		<br /><br />
		<div id="box1" ></div>
	</body>
</html>

```



### DOM

由于HTML文档被浏览器解析后就是一棵DOM树，要改变HTML的结构，就需要通过JavaScript来操作DOM。

始终记住DOM是一个树形结构。操作一个DOM节点实际上就是这么几个操作：

- 更新：更新该DOM节点的内容，相当于更新了该DOM节点表示的HTML的内容；
- 遍历：遍历该DOM节点下的子节点，以便进行进一步操作；
- 添加：在该DOM节点下新增一个子节点，相当于动态增加了一个HTML节点；
- 删除：将该节点从HTML中删除，相当于删掉了该DOM节点的内容以及它包含的所有子节点。

#### 查询

在操作一个DOM节点前，我们需要通过各种方式先拿到这个DOM节点。最常用的方法是`document.getElementById()`和`document.getElementsByTagName()`，以及CSS选择器`document.getElementsByClassName()`。

1. 根据id属性的值获取元素，返回来的是一个元素对象

```
document.getElementById("id属性的值");
```

2. 根据标签名字获取元素，返回来的是一个伪数组，里面保存了多个DOM对象

```
document.getElementsByTagName("标签名字");
```

下面的几个有的浏览器不支持

3. 根据name属性的值获取元素，返回来的是一个伪数组，里面保存了多个DOM对象

```
document.getElementsByName("name属性的值");
```

4. 根据类样式的名字获取元素，返回来的是一个伪数组，里面保存了多个DOM对象

```
document.getElementsByClassName("类样式的名字");
```

5. 根据选择器获取元素，返回来的是一个元素对象

```
document.querySelector("选择器的名字");
```

6. 根据选择器获取元素，返回来的是一个伪数组，里面保存了多个DOM对象

```
document.querySelectorAll("选择器的名字");
```



由于ID在HTML文档中是唯一的，所以`document.getElementById()`可以直接定位唯一的一个DOM节点。`document.getElementsByTagName()`和`document.getElementsByClassName()`总是返回一组DOM节点。要精确地选择DOM，可以先定位父节点，再从父节点开始选择，以缩小范围。

例如：

```
// 返回ID为'test'的节点：
var test = document.getElementById('test');

// 先定位ID为'test-table'的节点，再返回其内部所有tr节点：
var trs = document.getElementById('test-table').getElementsByTagName('tr');

// 先定位ID为'test-div'的节点，再返回其内部所有class包含red的节点：
var reds = document.getElementById('test-div').getElementsByClassName('red');

// 获取节点test下的所有直属子节点:
var cs = test.children;

// 获取节点test下第一个、最后一个子节点：
var first = test.firstElementChild;
var last = test.lastElementChild;
```

第二种方法是使用`querySelector()`和`querySelectorAll()`，需要了解selector语法，然后使用条件来获取节点，更加方便：

```
// 通过querySelector获取ID为q1的节点：
var q1 = document.querySelector('#q1');

// 通过querySelectorAll获取q1节点内的符合条件的所有节点：
var ps = q1.querySelectorAll('div.highlighted > p');

```

注意：低版本的IE<8不支持`querySelector`和`querySelectorAll`。IE8仅有限支持。

严格地讲，我们这里的DOM节点是指`Element`，但是DOM节点实际上是`Node`，在HTML中，`Node`包括`Element`、`Comment`、`CDATA_SECTION`等很多种，以及根节点`Document`类型，但是，绝大多数时候我们只关心`Element`，也就是实际控制页面结构的`Node`，其他类型的`Node`忽略即可。根节点`Document`已经自动绑定为全局变量`document`。

#### 更新DOM

拿到一个DOM节点后，我们可以对它进行更新。

可以直接修改节点的文本，方法有两种：

一种是修改`innerHTML`属性，这个方式非常强大，不但可以修改一个DOM节点的文本内容，还可以直接通过HTML片段修改DOM节点内部的子树：

```
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本为abc:
p.innerHTML = 'ABC'; // <p id="p-id">ABC</p>
// 设置HTML:
p.innerHTML = 'ABC <span style="color:red">RED</span> XYZ';
// <p>...</p>的内部结构已修改
```

用`innerHTML`时要注意，是否需要写入HTML。如果写入的字符串是通过网络拿到了，要注意对字符编码来避免XSS攻击。

第二种是修改`innerText`或`textContent`属性，这样可以自动对字符串进行HTML编码，保证无法设置任何HTML标签：

```
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本:
p.innerText = '<script>alert("Hi")</script>';
// HTML被自动编码，无法设置一个<script>节点:
// <p id="p-id">&lt;script&gt;alert("Hi")&lt;/script&gt;</p>
```

两者的区别在于读取属性时，`innerText`不返回隐藏元素的文本，而`textContent`返回所有文本。另外注意IE<9不支持`textContent`。

修改CSS也是经常需要的操作。DOM节点的`style`属性对应所有的CSS，可以直接获取或设置。因为CSS允许`font-size`这样的名称，但它并非JavaScript有效的属性名，所以需要在JavaScript中改写为驼峰式命名`fontSize`：

```
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置CSS:
p.style.color = '#ff0000';
p.style.fontSize = '20px';
p.style.paddingTop = '2em';
```

#### 插入DOM

当我们获得了某个DOM节点，想在这个DOM节点内插入新的DOM，应该如何做？

如果这个DOM节点是空的，例如，`<div></div>`，那么，直接使用`innerHTML = '<span>child</span>'`就可以修改DOM节点的内容，相当于“插入”了新的DOM节点。

如果这个DOM节点不是空的，那就不能这么做，因为`innerHTML`会直接替换掉原来的所有子节点。

有两个办法可以插入新的节点。一个是使用`appendChild`，把一个子节点添加到父节点的最后一个子节点。例如：

```
<!-- HTML结构 -->
<p id="js">JavaScript</p>
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
</div>
```

把`<p id="js">JavaScript</p>`添加到`<div id="list">`的最后一项：

```
var
    js = document.getElementById('js'),
    list = document.getElementById('list');
list.appendChild(js);
```

现在，HTML结构变成了这样：

```
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
    <p id="js">JavaScript</p>
</div>
```

因为我们插入的`js`节点已经存在于当前的文档树，因此这个节点首先会从原先的位置删除，再插入到新的位置。

更多的时候我们会从零创建一个新的节点，然后插入到指定位置：

```
var
    list = document.getElementById('list'),
    haskell = document.createElement('p');
haskell.id = 'haskell';
haskell.innerText = 'Haskell';
list.appendChild(haskell);
```

这样我们就动态添加了一个新的节点：

```
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
    <p id="haskell">Haskell</p>
</div>
```

动态创建一个节点然后添加到DOM树中，可以实现很多功能。举个例子，下面的代码动态创建了一个`<style>`节点，然后把它添加到`<head>`节点的末尾，这样就动态地给文档添加了新的CSS定义：

```
var d = document.createElement('style');
d.setAttribute('type', 'text/css');
d.innerHTML = 'p { color: red }';
document.getElementsByTagName('head')[0].appendChild(d);
```

可以在Chrome的控制台执行上述代码，观察页面样式的变化。

##### insertBefore

如果我们要把子节点插入到指定的位置怎么办？可以使用`parentElement.insertBefore(newElement, referenceElement);`，子节点会插入到`referenceElement`之前。

还是以上面的HTML为例，假定我们要把`Haskell`插入到`Python`之前：

```
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
</div>
```

可以这么写：

```
var
    list = document.getElementById('list'),
    ref = document.getElementById('python'),
    haskell = document.createElement('p');
haskell.id = 'haskell';
haskell.innerText = 'Haskell';
list.insertBefore(haskell, ref);
```

新的HTML结构如下：

```
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="haskell">Haskell</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
</div>
```

可见，使用`insertBefore`重点是要拿到一个“参考子节点”的引用。很多时候，需要循环一个父节点的所有子节点，可以通过迭代`children`属性实现：

```
var
    i, c,
    list = document.getElementById('list');
for (i = 0; i < list.children.length; i++) {
    c = list.children[i]; // 拿到第i个子节点
}
```

#### 删除DOM

删除一个DOM节点就比插入要容易得多。

要删除一个节点，首先要获得该节点本身以及它的父节点，然后，调用父节点的`removeChild`把自己删掉：

```
// 拿到待删除节点:
var self = document.getElementById('to-be-removed');
// 拿到父节点:
var parent = self.parentElement;
// 删除:
var removed = parent.removeChild(self);
removed === self; // true
```

注意到删除后的节点虽然不在文档树中了，但其实它还在内存中，可以随时再次被添加到别的位置。

当你遍历一个父节点的子节点并进行删除操作时，要注意，`children`属性是一个只读属性，并且它在子节点变化时会实时更新。

例如，对于如下HTML结构：

```
<div id="parent">
    <p>First</p>
    <p>Second</p>
</div>
```

当我们用如下代码删除子节点时：

```
var parent = document.getElementById('parent');
parent.removeChild(parent.children[0]);
parent.removeChild(parent.children[1]); // <-- 浏览器报错
```

浏览器报错：`parent.children[1]`不是一个有效的节点。原因就在于，当`<p>First</p>`节点被删除后，`parent.children`的节点数量已经从2变为了1，索引`[1]`已经不存在了。

因此，删除多个节点时，要注意`children`属性时刻都在变化。

#### 示例：

观看图片

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			*{
				margin: 0;
				padding: 0;
			}
			
			#outer{
				width: 500px;
				margin: 50px auto;
				padding: 10px;
				background-color: greenyellow;
				/*设置文本居中*/
				text-align: center;
			}
		</style>
		
		<script type="text/javascript">
			
			window.onload = function(){
				
				/*
				 * 点击按钮切换图片
				 */
				
				//获取两个按钮
				var prev = document.getElementById("prev");
				var next = document.getElementById("next");
				
				/*
				 * 要切换图片就是要修改img标签的src属性
				 */
				
				//获取img标签
				var img = document.getElementsByTagName("img")[0];
				
				//创建一个数组，用来保存图片的路径
				var imgArr = ["img/1.jpg" , "img/2.jpg" , "img/3.jpg" , "img/4.jpg" ,"img/5.jpg"];
				
				//创建一个变量，来保存当前正在显示的图片的索引
				var index = 0;
				
				//获取id为info的p元素
				var info = document.getElementById("info");
				//设置提示文字
				info.innerHTML = "一共 "+imgArr.length+" 张图片，当前第 "+(index+1)+" 张";
				
				
				//分别为两个按钮绑定单击响应函数
				prev.onclick = function(){
					
					/*
					 * 切换到上一张，索引自减
					 */
					index--;
					
					//判断index是否小于0
					if(index < 0){
						index = imgArr.length - 1;
					}
					
					img.src = imgArr[index];
					
					//当点击按钮以后，重新设置信息
					info.innerHTML = "一共 "+imgArr.length+" 张图片，当前第 "+(index+1)+" 张";
				};
				
				next.onclick = function(){
					
					/*
					 * 切换到下一张是index自增
					 */
					index++;
					
					if(index > imgArr.length - 1){
						index = 0;
					}
					
					//切换图片就是修改img的src属性
					//要修改一个元素的属性 元素.属性 = 属性值
					img.src = imgArr[index];
					
					//当点击按钮以后，重新设置信息
					info.innerHTML = "一共 "+imgArr.length+" 张图片，当前第 "+(index+1)+" 张";
					
				};
				
				
			};
			
			
		</script>
	</head>
	<body>
		<div id="outer">
			
			<p id="info"></p>
			
			<img src="img/1.jpg" alt="冰棍" />
			
			<button id="prev">上一张</button>
			<button id="next">下一张</button>
			
		</div>
	</body>
</html>

```

增删改

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>Untitled Document</title>
		<link rel="stylesheet" type="text/css" href="style/css.css" />
		<script type="text/javascript">
		
			window.onload = function() {
				
				//创建一个"广州"节点,添加到#city下
				myClick("btn01",function(){
					//创建广州节点 <li>广州</li>
					//创建li元素节点
					/*
					 * document.createElement()
					 * 	可以用于创建一个元素节点对象，
					 * 	它需要一个标签名作为参数，将会根据该标签名创建元素节点对象，
					 * 	并将创建好的对象作为返回值返回
					 */
					var li = document.createElement("li");
					
					//创建广州文本节点
					/*
					 * document.createTextNode()
					 * 	可以用来创建一个文本节点对象
					 *  需要一个文本内容作为参数，将会根据该内容创建文本节点，并将新的节点返回
					 */
					var gzText = document.createTextNode("广州");
					
					//将gzText设置li的子节点
					/*
					 * appendChild()
					 * 	 - 向一个父节点中添加一个新的子节点
					 * 	 - 用法：父节点.appendChild(子节点);
					 */
					li.appendChild(gzText);
					
					//获取id为city的节点
					var city = document.getElementById("city");
					
					//将广州添加到city下
					city.appendChild(li);
					
					
				});
				
				//将"广州"节点插入到#bj前面
				myClick("btn02",function(){
					//创建一个广州
					var li = document.createElement("li");
					var gzText = document.createTextNode("广州");
					li.appendChild(gzText);
					
					//获取id为bj的节点
					var bj = document.getElementById("bj");
					
					//获取city
					var city = document.getElementById("city");
					
					/*
					 * insertBefore()
					 * 	- 可以在指定的子节点前插入新的子节点
					 *  - 语法：
					 * 		父节点.insertBefore(新节点,旧节点);
					 */
					city.insertBefore(li , bj);
					
					
				});
				
				
				//使用"广州"节点替换#bj节点
				myClick("btn03",function(){
					//创建一个广州
					var li = document.createElement("li");
					var gzText = document.createTextNode("广州");
					li.appendChild(gzText);
					
					//获取id为bj的节点
					var bj = document.getElementById("bj");
					
					//获取city
					var city = document.getElementById("city");
					
					/*
					 * replaceChild()
					 * 	- 可以使用指定的子节点替换已有的子节点
					 * 	- 语法：父节点.replaceChild(新节点,旧节点);
					 */
					city.replaceChild(li , bj);
					
					
				});
				
				//删除#bj节点
				myClick("btn04",function(){
					//获取id为bj的节点
					var bj = document.getElementById("bj");
					//获取city
					var city = document.getElementById("city");
					
					/*
					 * removeChild()
					 * 	- 可以删除一个子节点
					 * 	- 语法：父节点.removeChild(子节点);
					 * 		
					 * 		子节点.parentNode.removeChild(子节点);
					 */
					//city.removeChild(bj);
					
					bj.parentNode.removeChild(bj);
				});
				
				
				//读取#city内的HTML代码
				myClick("btn05",function(){
					//获取city
					var city = document.getElementById("city");
					
					alert(city.innerHTML);
				});
				
				//设置#bj内的HTML代码
				myClick("btn06" , function(){
					//获取bj
					var bj = document.getElementById("bj");
					bj.innerHTML = "昌平";
				});
				
				myClick("btn07",function(){
					
					//向city中添加广州
					var city = document.getElementById("city");
					
					/*
					 * 使用innerHTML也可以完成DOM的增删改的相关操作
					 * 一般我们会两种方式结合使用
					 */
					//city.innerHTML += "<li>广州</li>";
					
					//创建一个li
					var li = document.createElement("li");
					//向li中设置文本
					li.innerHTML = "广州";
					//将li添加到city中
					city.appendChild(li);
					
				});
				
				
			};
			
			function myClick(idStr, fun) {
				var btn = document.getElementById(idStr);
				btn.onclick = fun;
			}
			
		
		</script>
		
	</head>
	<body>
		<div id="total">
			<div class="inner">
				<p>
					你喜欢哪个城市?
				</p>

				<ul id="city">
					<li id="bj">北京</li>
					<li>上海</li>
					<li>东京</li>
					<li>首尔</li>
				</ul>
				
			</div>
		</div>
		<div id="btnList">
			<div><button id="btn01">创建一个"广州"节点,添加到#city下</button></div>
			<div><button id="btn02">将"广州"节点插入到#bj前面</button></div>
			<div><button id="btn03">使用"广州"节点替换#bj节点</button></div>
			<div><button id="btn04">删除#bj节点</button></div>
			<div><button id="btn05">读取#city内的HTML代码</button></div>
			<div><button id="btn06">设置#bj内的HTML代码</button></div>
			<div><button id="btn07">创建一个"广州"节点,添加到#city下</button></div>
		</div>
	</body>
</html>
```

### 操作表单

用JavaScript操作表单和操作DOM是类似的，因为表单本身也是DOM树。

不过表单的输入框、下拉框等可以接收用户输入，所以用JavaScript来操作表单，可以获得用户输入的内容，或者对一个输入框设置新的内容。

HTML表单的输入控件主要有以下几种：

- 文本框，对应的`<input type="text">`，用于输入文本；
- 口令框，对应的`<input type="password">`，用于输入口令；
- 单选框，对应的`<input type="radio">`，用于选择一项；
- 复选框，对应的`<input type="checkbox">`，用于选择多项；
- 下拉框，对应的`<select>`，用于选择一项；
- 隐藏文本，对应的`<input type="hidden">`，用户不可见，但表单提交时会把隐藏文本发送到服务器。

#### 获取值

如果我们获得了一个`<input>`节点的引用，就可以直接调用`value`获得对应的用户输入值：

```
// <input type="text" id="email">
var input = document.getElementById('email');
input.value; // '用户输入的值'
```

这种方式可以应用于`text`、`password`、`hidden`以及`select`。但是，对于单选框和复选框，`value`属性返回的永远是HTML预设的值，而我们需要获得的实际是用户是否“勾上了”选项，所以应该用`checked`判断：

```
// <label><input type="radio" name="weekday" id="monday" value="1"> Monday</label>
// <label><input type="radio" name="weekday" id="tuesday" value="2"> Tuesday</label>
var mon = document.getElementById('monday');
var tue = document.getElementById('tuesday');
mon.value; // '1'
tue.value; // '2'
mon.checked; // true或者false
tue.checked; // true或者false
```

#### 设置值

设置值和获取值类似，对于`text`、`password`、`hidden`以及`select`，直接设置`value`就可以：

```
// <input type="text" id="email">
var input = document.getElementById('email');
input.value = 'test@example.com'; // 文本框的内容已更新
```

对于单选框和复选框，设置`checked`为`true`或`false`即可。

示例：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>全选练习</title>
<script type="text/javascript">

	window.onload = function(){
		
		
		//获取四个多选框items
		var items = document.getElementsByName("items");
		//获取全选/全不选的多选框
		var checkedAllBox = document.getElementById("checkedAllBox");
		
		/*
		 * 全选按钮
		 * 	- 点击按钮以后，四个多选框全都被选中
		 */
		
		//1.#checkedAllBtn
		//为id为checkedAllBtn的按钮绑定一个单击响应函数
		var checkedAllBtn = document.getElementById("checkedAllBtn");
		checkedAllBtn.onclick = function(){
			
			
			//遍历items
			for(var i=0 ; i<items.length ; i++){
				
				//通过多选框的checked属性可以来获取或设置多选框的选中状态
				//alert(items[i].checked);
				
				//设置四个多选框变成选中状态
				items[i].checked = true;
			}
			
			//将全选/全不选设置为选中
			checkedAllBox.checked = true;
			
			
		};
		
		/*
		 * 全不选按钮
		 * 	- 点击按钮以后，四个多选框都变成没选中的状态
		 */
		//2.#checkedNoBtn
		//为id为checkedNoBtn的按钮绑定一个单击响应函数
		var checkedNoBtn = document.getElementById("checkedNoBtn");
		checkedNoBtn.onclick = function(){
			
			for(var i=0; i<items.length ; i++){
				//将四个多选框设置为没选中的状态
				items[i].checked = false;
			}
			
			//将全选/全不选设置为不选中
			checkedAllBox.checked = false;
			
		};
		
		/*
		 * 反选按钮
		 * 	- 点击按钮以后，选中的变成没选中，没选中的变成选中
		 */
		//3.#checkedRevBtn
		var checkedRevBtn = document.getElementById("checkedRevBtn");
		checkedRevBtn.onclick = function(){
			
			//将checkedAllBox设置为选中状态
			checkedAllBox.checked = true;
			
			for(var i=0; i<items.length ; i++){
				
				//判断多选框状态
				/*if(items[i].checked){
					//证明多选框已选中，则设置为没选中状态
					items[i].checked = false;
				}else{
					//证明多选框没选中，则设置为选中状态
					items[i].checked = true;
				}*/
				
				items[i].checked = !items[i].checked;
				
				//判断四个多选框是否全选
				//只要有一个没选中则就不是全选
				if(!items[i].checked){
					//一旦进入判断，则证明不是全选状态
					//将checkedAllBox设置为没选中状态
					checkedAllBox.checked = false;
				}
			}
			
			//在反选时也需要判断四个多选框是否全都选中
			
			
			
		};
		
		/*
		 * 提交按钮：
		 * 	- 点击按钮以后，将所有选中的多选框的value属性值弹出
		 */
		//4.#sendBtn
		//为sendBtn绑定单击响应函数
		var sendBtn = document.getElementById("sendBtn");
		sendBtn.onclick = function(){
			//遍历items
			for(var i=0 ; i<items.length ; i++){
				//判断多选框是否选中
				if(items[i].checked){
					alert(items[i].value);
				}
			}
		};
		
		
		//5.#checkedAllBox
		/*
		 * 全选/全不选 多选框
		 * 	- 当它选中时，其余的也选中，当它取消时其余的也取消
		 * 
		 * 在事件的响应函数中，响应函数是给谁绑定的this就是谁
		 */
		//为checkedAllBox绑定单击响应函数
		checkedAllBox.onclick = function(){
			
			//alert(this === checkedAllBox);
			
			//设置多选框的选中状态
			for(var i=0; i <items.length ; i++){
				items[i].checked = this.checked;
			}
			
		};
		
		//6.items
		/*
		 * 如果四个多选框全都选中，则checkedAllBox也应该选中
		 * 如果四个多选框没都选中，则checkedAllBox也不应该选中
		 */
		
		//为四个多选框分别绑定点击响应函数
		for(var i=0 ; i<items.length ; i++){
			items[i].onclick = function(){
				
				//将checkedAllBox设置为选中状态
				checkedAllBox.checked = true;
				
				for(var j=0 ; j<items.length ; j++){
					//判断四个多选框是否全选
					//只要有一个没选中则就不是全选
					if(!items[j].checked){
						//一旦进入判断，则证明不是全选状态
						//将checkedAllBox设置为没选中状态
						checkedAllBox.checked = false;
						//一旦进入判断，则已经得出结果，不用再继续执行循环
						break;
					}
					
				}
				
				
				
			};
		}
		
		
	};
	
</script>
</head>
<body>

	<form method="post" action="">
		你爱好的运动是？<input type="checkbox" id="checkedAllBox" />全选/全不选 
		
		<br />
		<input type="checkbox" name="items" value="足球" />足球
		<input type="checkbox" name="items" value="篮球" />篮球
		<input type="checkbox" name="items" value="羽毛球" />羽毛球
		<input type="checkbox" name="items" value="乒乓球" />乒乓球
		<br />
		<input type="button" id="checkedAllBtn" value="全　选" />
		<input type="button" id="checkedNoBtn" value="全不选" />
		<input type="button" id="checkedRevBtn" value="反　选" />
		<input type="button" id="sendBtn" value="提　交" />
	</form>
</body>
</html>
```

#### HTML5控件

HTML5新增了大量标准控件，常用的包括`date`、`datetime`、`datetime-local`、`color`等，它们都使用`<input>`标签：

```
<input type="date" value="2015-07-01">
<input type="datetime-local" value="2015-07-01T02:03:04">
<input type="color" value="#ff0000">
```

不支持HTML5的浏览器无法识别新的控件，会把它们当做`type="text"`来显示。支持HTML5的浏览器将获得格式化的字符串。例如，`type="date"`类型的`input`的`value`将保证是一个有效的`YYYY-MM-DD`格式的日期，或者空字符串。

#### 提交表单

最后，JavaScript可以以两种方式来处理表单的提交（AJAX方式在后面章节介绍）。

方式一是通过`<form>`元素的`submit()`方法提交一个表单，例如，响应一个`<button>`的`click`事件，在JavaScript代码中提交表单：

```
<!-- HTML -->
<form id="test-form">
    <input type="text" name="test">
    <button type="button" onclick="doSubmitForm()">Submit</button>
</form>

<script>
function doSubmitForm() {
    var form = document.getElementById('test-form');
    // 可以在此修改form的input...
    // 提交form:
    form.submit();
}
</script>
```

这种方式的缺点是扰乱了浏览器对form的正常提交。浏览器默认点击`<button type="submit">`时提交表单，或者用户在最后一个输入框按回车键。因此，第二种方式是响应`<form>`本身的`onsubmit`事件，在提交form时作修改：

```
<!-- HTML -->
<form id="test-form" onsubmit="return checkForm()">
    <input type="text" name="test">
    <button type="submit">Submit</button>
</form>

<script>
function checkForm() {
    var form = document.getElementById('test-form');
    // 可以在此修改form的input...
    // 继续下一步:
    return true;
}
</script>
```

注意要`return true`来告诉浏览器继续提交，如果`return false`，浏览器将不会继续提交form，这种情况通常对应用户输入有误，提示用户错误信息后终止提交form。

在检查和修改`<input>`时，要充分利用`<input type="hidden">`来传递数据。

例如，很多登录表单希望用户输入用户名和口令，但是，安全考虑，提交表单时不传输明文口令，而是口令的MD5。普通JavaScript开发人员会直接修改`<input>`：

```
<!-- HTML -->
<form id="login-form" method="post" onsubmit="return checkForm()">
    <input type="text" id="username" name="username">
    <input type="password" id="password" name="password">
    <button type="submit">Submit</button>
</form>

<script>
function checkForm() {
    var pwd = document.getElementById('password');
    // 把用户输入的明文变为MD5:
    pwd.value = toMD5(pwd.value);
    // 继续下一步:
    return true;
}
</script>
```

这个做法看上去没啥问题，但用户输入了口令提交时，口令框的显示会突然从几个`*`变成32个`*`（因为MD5有32个字符）。

要想不改变用户的输入，可以利用`<input type="hidden">`实现：

```
<!-- HTML -->
<form id="login-form" method="post" onsubmit="return checkForm()">
    <input type="text" id="username" name="username">
    <input type="password" id="input-password">
    <input type="hidden" id="md5-password" name="password">
    <button type="submit">Submit</button>
</form>

<script>
function checkForm() {
    var input_pwd = document.getElementById('input-password');
    var md5_pwd = document.getElementById('md5-password');
    // 把用户输入的明文变为MD5:
    md5_pwd.value = toMD5(input_pwd.value);
    // 继续下一步:
    return true;
}
</script>
```

注意到`id`为`md5-password`的`<input>`标记了`name="password"`，而用户输入的`id`为`input-password`的`<input>`没有`name`属性。没有`name`属性的`<input>`的数据不会被提交。

### 操作文件

在HTML表单中，可以上传文件的唯一控件就是`<input type="file">`。

*注意*：当一个表单包含`<input type="file">`时，表单的`enctype`必须指定为`multipart/form-data`，`method`必须指定为`post`，浏览器才能正确编码并以`multipart/form-data`格式发送表单的数据。

出于安全考虑，浏览器只允许用户点击`<input type="file">`来选择本地文件，用JavaScript对`<input type="file">`的`value`赋值是没有任何效果的。当用户选择了上传某个文件后，JavaScript也无法获得该文件的真实路径

通常，上传的文件都由后台服务器处理，JavaScript可以在提交表单时对文件扩展名做检查，以便防止用户上传无效格式的文件：

```
var f = document.getElementById('test-file-upload');
var filename = f.value; // 'C:\fakepath\test.png'
if (!filename || !(filename.endsWith('.jpg') || filename.endsWith('.png') || filename.endsWith('.gif'))) {
    alert('Can only upload image file.');
    return false;
}
```

#### File API

由于JavaScript对用户上传的文件操作非常有限，尤其是无法读取文件内容，使得很多需要操作文件的网页不得不用Flash这样的第三方插件来实现。

随着HTML5的普及，新增的File API允许JavaScript读取文件内容，获得更多的文件信息。

HTML5的File API提供了`File`和`FileReader`两个主要对象，可以获得文件信息并读取文件。

```
var
    fileInput = document.getElementById('test-image-file'),
    info = document.getElementById('test-file-info'),
    preview = document.getElementById('test-image-preview');
// 监听change事件:
fileInput.addEventListener('change', function () {
    // 清除背景图片:
    preview.style.backgroundImage = '';
    // 检查文件是否选择:
    if (!fileInput.value) {
        info.innerHTML = '没有选择文件';
        return;
    }
    // 获取File引用:
    var file = fileInput.files[0];
    // 获取File信息:
    info.innerHTML = '文件: ' + file.name + '<br>' +
                     '大小: ' + file.size + '<br>' +
                     '修改: ' + file.lastModifiedDate;
    if (file.type !== 'image/jpeg' && file.type !== 'image/png' && file.type !== 'image/gif') {
        alert('不是有效的图片文件!');
        return;
    }
    // 读取文件:
    var reader = new FileReader();
    reader.onload = function(e) {
        var
            data = e.target.result; // 'data:image/jpeg;base64,/9j/4AAQSk...(base64编码)...'            
        preview.style.backgroundImage = 'url(' + data + ')';
    };
    // 以DataURL的形式读取文件:
    reader.readAsDataURL(file);
});
```

上面的代码演示了如何通过HTML5的File API读取文件内容。以DataURL的形式读取到的文件是一个字符串，类似于`data:image/jpeg;base64,/9j/4AAQSk...(base64编码)...`，常用于设置图像。如果需要服务器端处理，把字符串`base64,`后面的字符发送给服务器并用Base64解码就可以得到原始文件的二进制内容。

#### 回调

上面的代码还演示了JavaScript的一个重要的特性就是单线程执行模式。在JavaScript中，浏览器的JavaScript执行引擎在执行JavaScript代码时，总是以单线程模式执行，也就是说，任何时候，JavaScript代码都不可能同时有多于1个线程在执行。

你可能会问，单线程模式执行的JavaScript，如何处理多任务？

在JavaScript中，执行多任务实际上都是异步调用，比如上面的代码：

```
reader.readAsDataURL(file);
```

就会发起一个异步操作来读取文件内容。因为是异步操作，所以我们在JavaScript代码中就不知道什么时候操作结束，因此需要先设置一个回调函数：

```
reader.onload = function(e) {
    // 当文件读取完成后，自动调用此函数:
};
```

当文件读取完成后，JavaScript引擎将自动调用我们设置的回调函数。执行回调函数时，文件已经读取完毕，所以我们可以在回调函数内部安全地获得文件内容