---
title: JavaScript 事件
date: 2019-05-05 11:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

#### 事件流

事件冒泡和事件捕获分别由微软和网景公司提出，这两个概念是为了解决页面中事件流（事件发生顺序）的问题。

```html
<div id="outer">
    <p id="inner">Click me!</p>
</div>
```

上面的代码当中一个div元素当中有一个p子元素，如果两个元素都有一个click的处理函数，那么我们怎么才能知道哪一个函数会首先被触发呢？

<!--more-->

为了解决这个问题微软和网景提出了两种几乎完全相反的概念。

1. 事件冒泡

   微软提出了名为事件冒泡的事件流。事件冒泡可以形象地比喻为把一颗石头投入水中，泡泡会一直从水底冒出水面。也就是说，事件会从最内层的元素开始发生，一直向上传播，直到document对象。

   因此上面的例子在事件冒泡的概念下发生click事件的顺序应该是p -> div -> body -> html -> document

2. 事件捕获

   网景提出另一种事件流名为事件捕获与事件冒泡相反，事件会从最外层开始发生，直到最具体的元素。

   上面的例子在事件捕获的概念下发生click事件的顺序应该是document -> html -> body -> div -> p

3. W3C事件阶段(event phase)：

   当一个DOM事件被触发的时候，他并不是只在它的起源对象上触发一次，而是会经历三个不同的阶段。简而言之：事件一开始从文档的根节点流向目标对象(捕获阶段)，然后在目标对向上被触发(目标阶段)，之后再回溯到文档的根节点(冒泡阶段):

   ![2.png](https://i.loli.net/2019/06/03/5cf48320914c851528.png)

   

**事件捕获阶段(Capture Phase)**

事件从文档的根节点出发，随着DOM树的结构向事件的目标节点流去。途中经过各个层次的DOM节点，并在各节点上触发捕获事件，直到到达时间的目标节点。捕获阶段的主要任务是简历传播路径，在冒泡阶段，时间会通过这个路径回溯到文档根节点。

例如，通过下面的这个函数来给节点设置监听，可以通过将;设置成true来为事件的捕获阶段添加监听回调函数。

```
element.removeEventListener(&ltevent-name>, <callback>, <use-capture>);
```

而，在实际应用中，我们并没有太多使用捕获阶段监听的用例，但是通过在捕获阶段对事件的处理，我们可以阻止类似click事件在某个特定元素上被触发。如下：

```
var form=document.querySeletor('form');
form.addEventListener('click',function(e){
  e.stopPropagation();
  },true);
```

如果你对这种用法不是很了解的话，建议设置为false或者undefined，从而在冒泡阶段对事件进行监听，这也是常用的方法。

**目标阶段(Target Phase)**

当事件到达目标节点时，事件就进入了目标阶段。事件在目标节点上被触发，然后逆向回流，知道传播到最外层的文档节点。

对于多层嵌套的节点，鼠标和指针事件经常会被定位到最里层的元素上。假设，你在一个div元素上设置了click的监听函数，而用户点击在了这个div元素内部的p元素上，那么p元素就是这个时间的目标元素。事件冒泡让我们可以在这个div或者更上层的元素上监听click事件，并且时间传播过程中触发回调函数。

**冒泡阶段(Bubble Phase)**

事件在目标事件上触发后，并不在这个元素上终止。它会随着DOM树一层层向上冒泡，直到到达最外层的根节点，一直向上传播，直到document对象。也就是说，同一事件会一次在目标节点的父节点，父节点的父节点...直到最外层的节点上触发。

绝大多数事件是会冒泡的，但并非所有的。

<iframe scrolling="no" src="https://codepen.io/jingwhale/embed/dojdVG/?height=265&amp;theme-id=dark&amp;default-tab=js,result&amp;embed-version=2" allowtransparency="true" allowfullscreen="true" style="width: 100%;" height="265" frameborder="no">See
 the Pen &lt;a 
href='http://codepen.io/jingwhale/pen/dojdVG/'&gt;dojdVG&lt;/a&gt; by 
jingwhale (&lt;a 
href='http://codepen.io/jingwhale'&gt;@jingwhale&lt;/a&gt;) on &lt;a 
href='http://codepen.io'&gt;CodePen&lt;/a&gt;.
</iframe>

1. 事件冒泡(fasle/不写)：当触发一个节点的事件是，会从当前节点开始，依次触发其祖先节点的同类型事件，直到DOM根节点。
2. 事件捕获(true)：当初发一个节点的事件时，会从DOM根节点开始，依次触发其祖先节点的同类型事件，直到当前节点自身。
3. 什么时候事件冒泡？什么时候事件捕获？
   - 当使用addEventListener绑定事件，第三个参数传为true时表示事件捕获；
   -  除此之外的所有事件绑定均为事件冒泡。

4. 阻止事件冒泡：
   - IE10之前，e.cancelBubble = true;
   -  IE10之后，e.stopPropagation();

5. 阻止默认事件：
   - IE10之前：e.returnValue = false;
   -  IE10之后：e.preventDefault();

#### 事件类型

事件类型有：

- UI（用户界面）事件，用户与页面上元素交互时触发 ；
- 焦点事件：当元素获得或失去焦点时触发 ； 
- 文本事件：当在文档中输入文本时触发；
- 键盘事件：当用户通过键盘在页面上执行操作时触发；
- 鼠标事件：当用户通过鼠标在页面上执行操作时触发；
- 滚轮事件：当使用鼠标滚轮（或类似设备）时触发。

它们之间是继承的关系，如下图：

![1.jpg](https://i.loli.net/2019/06/03/5cf484328704383957.jpg)

1. 

![2.jpg](https://i.loli.net/2019/06/03/5cf484344106512094.jpg)

常用：window、image。例如设置默认的图片：

```
<img src="photo" alt="photo.jpg" onerror="this.src='defualt.jpg'">
```

2. ![3.jpg](https://i.loli.net/2019/06/03/5cf4849e093b469256.jpg)

3. ![4.jpg](https://i.loli.net/2019/06/03/5cf4849edafa690055.jpg)
4. ![5.jpg](https://i.loli.net/2019/06/03/5cf484b0cb68b62504.jpg)



5. ![6.jpg](https://i.loli.net/2019/06/03/5cf484b29dffd87649.jpg)

   ```
   MouseEvent 顺序
   从元素A上方移过
   -mousemove-> mouseover (A) ->mouseenter (A)-> mousemove (A) ->mouseout (A) ->mouseleave (A)
   点击元素
   -mousedown-> [mousemove]-> mouseup->click
   ```

6. ![7.jpg](https://i.loli.net/2019/06/03/5cf484ed820bc90860.jpg)

7. 键盘事件：

   -  keydown：键盘按下时触发

   -  keypress：键盘按下并抬起的瞬间触发。
   - keyup：键盘抬起触发 

   [注意事项]

   - 执行顺序：keydown keypress keyup

   - keypress只能捕获数字，字母，符号键，而不能捕获功能键。

   - 长按时循环执行keydown--keypress

   - 有keydown，并不一定有keyup，当长按时焦点失去，将不再触发keyup

   - keypress区分大小写，keydown，kewup不区分。

**事件因子**

当触发一个事件时，该事件将向事件所调用的函数中，默认传入一个参数，

这个参数就是一个事件因子，包含了该事件的各种详细信息。

```


 document.onkeydown=function(e){
 console.log(e);
 }
document.onkeydown=function(){
console.log(window.event);
}

//兼容浏览器的写法：
document.onkeydown=function(e){
e==e||Window.event;
var Code=e.keyCode||e.which||e.charCode;
if(code==13){
//回车
}
}
```

#### 事件绑定模型

**DOM0事件模型**

绑定注意事项：

使用window.onload加载完成后进行绑定。

```
window.onload =function(){//事件}
```

放在body后面进行绑定。

```
//body内容
<body>
  <button onclick="func()">内联模型绑定</button>
  <button id="btn1">哈哈哈哈</button>
  <button id="btn2">DOM2模型绑定</button>
  <button id="btn3">取消DOM2</button>
</body>
```

1. 内联模型（行内绑定）：将函数名直接作为html标签中属性的属性值。

```
<button onclick="func()">内联模型绑定</button>
```

 缺点：不符合w3c中关于内容与 行为分离的基本规范。


2. 脚本模型（动态绑定）：通过在JS中选中某个节点，然后给节点添加onclick属性。

```
document.getElementById("btn1")=function(){}
```

 优点：符合w3c中关于内容与行为分离的基本规范，实现html与js的分离。

 缺点：同一个节点只能添加一次同类型事件，如果添加多次，最后一个生效。

```
document.getElementById("btn1").onclick=function(){
  alert(1234);  
}
document.getElementById("btn1").onclick=function(){
  alert(234);  
}//重复的只能出现最近的一次
```

3. DOM0共有缺点：通过DOM0绑定的事件，一旦绑定将无法取消。

```javascript
//document.onclick = fn1;
document.onclick = null;//覆盖
```

**DOM2事件模型**

1. 添加DOM2事件绑定：
   - IE8之前，使用.attachEvent("onclick",函数);
   - IE8之后，使用.addEventListener("click",函数，true/false); 参数三：false（默认）表示事件冒泡，传入true表示事件捕获。
   - 兼容所有浏览器的处理方式：

```javascript
 var btn=document.getElementById("btn1");
 if(btn.attachEvent){
 btn.attachEvent("onclick",func1);//事件，事件需要执行的函数IE8可以
 }else{
 btn.attachEventListener("click",func1);
 }
 //bind(btn,'click',func1)

 //定义一个函数，用来为指定元素绑定响应函数
			/*
			 * addEventListener()中的this，是绑定事件的对象
			 * attachEvent()中的this，是window
			 *  需要统一两个方法this
			 */
			/*
			 * 参数：
			 * 	obj 要绑定事件的对象
			 * 	eventStr 事件的字符串(不要on)
			 *  callback 回调函数
			 */
 function bind(obj , eventStr , callback){
				if(obj.addEventListener){
					//大部分浏览器兼容的方式
					obj.addEventListener(eventStr , callback , true);
				}else{
					/*
					 * this是谁由调用方式决定
					 * callback.call(obj)
					 */
					//IE8及以下
					obj.attachEvent("on"+eventStr , function(){
						//在匿名函数中调用回调函数
						callback.call(obj);
					});
				}
			}
```

2. DOM2绑定的优点：

   - 同一个节点，可以使用DOM2绑定多个同类型事件。

   - 使用DOM2绑定的事件，可以有专门的函数进行取消。

3. 取消事件绑定：

   - 使用attachEvent绑定，要用detachevent取消。

   - 使用attachEventListener绑定，要用removeEventListenter取消

     ```
     document.getElementById("btn3").onclick=function(){//不能取消匿名函数
       if(btn.detachEvent){
         btn.detachEvent("onclick",func1);
       }else{
         btn.removeEventListener("click",func1);
       }
         alert("取消DOM2");
     }
     ```

 注意：如果DOM2绑定的事件，需要取消，则绑定事件时，回调函数必须是函数名，  而不能是匿名函数，因为取消事件时，取消传入函数名进行取消。

**委派**

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<script type="text/javascript">
			
			window.onload = function(){
				
				var u1 = document.getElementById("u1");
				
				//点击按钮以后添加超链接
				var btn01 = document.getElementById("btn01");
				btn01.onclick = function(){
					//创建一个li
					var li = document.createElement("li");
					li.innerHTML = "<a href='javascript:;' class='link'>新建的超链接</a>";
					
					//将li添加到ul中
					u1.appendChild(li);
				};
				
				
				/*
				 * 为每一个超链接都绑定一个单击响应函数
				 * 这里我们为每一个超链接都绑定了一个单击响应函数，这种操作比较麻烦，
				 * 	而且这些操作只能为已有的超链接设置事件，而新添加的超链接必须重新绑定
				 */
				//获取所有的a
				var allA = document.getElementsByTagName("a");
				//遍历
				/*for(var i=0 ; i<allA.length ; i++){
					allA[i].onclick = function(){
						alert("我是a的单击响应函数！！！");
					};
				}*/
				
				/*
				 * 我们希望，只绑定一次事件，即可应用到多个的元素上，即使元素是后添加的
				 * 我们可以尝试将其绑定给元素的共同的祖先元素
				 * 
				 * 事件的委派
				 * 	- 指将事件统一绑定给元素的共同的祖先元素，这样当后代元素上的事件触发时，会一直冒泡到祖先元素
				 * 		从而通过祖先元素的响应函数来处理事件。
				 *  - 事件委派是利用了冒泡，通过委派可以减少事件绑定的次数，提高程序的性能
				 */
				
				//为ul绑定一个单击响应函数
				u1.onclick = function(event){
					event = event || window.event;
					
					/*
					 * target
					 * 	- event中的target表示的触发事件的对象
					 */
					//alert(event.target);
					
					
					//如果触发事件的对象是我们期望的元素，则执行否则不执行
					if(event.target.className == "link"){
						alert("我是ul的单击响应函数");
					}
					
				};
				
			};
			
		</script>
	</head>
	<body>
		<button id="btn01">添加超链接</button>
		
		<ul id="u1" style="background-color: #bfa;">
			<li>
				<p>我是p元素</p>
			</li>
			<li><a href="javascript:;" class="link">超链接一</a></li>
			<li><a href="javascript:;" class="link">超链接二</a></li>
			<li><a href="javascript:;" class="link">超链接三</a></li>
		</ul>
		
	</body>
</html>

```

#### 拖拽

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
				position: absolute;
			}
			
			#box2{
				width: 100px;
				height: 100px;
				background-color: yellow;
				position: absolute;
				
				left: 200px;
				top: 200px;
			}
			
		</style>
		
		<script type="text/javascript">
			
			window.onload = function(){
				/*
				 * 拖拽box1元素
				 *  - 拖拽的流程
				 * 		1.当鼠标在被拖拽元素上按下时，开始拖拽  onmousedown
				 * 		2.当鼠标移动时被拖拽元素跟随鼠标移动 onmousemove
				 * 		3.当鼠标松开时，被拖拽元素固定在当前位置	onmouseup
				 */
				
				//获取box1
				var box1 = document.getElementById("box1");
				var box2 = document.getElementById("box2");
				var img1 = document.getElementById("img1");
				
				//开启box1的拖拽
				drag(box1);
				//开启box2的
				drag(box2);
				
				drag(img1);
				
				
				
				
			};
			
			/*
			 * 提取一个专门用来设置拖拽的函数
			 * 参数：开启拖拽的元素
			 */
			function drag(obj){
				//当鼠标在被拖拽元素上按下时，开始拖拽  onmousedown
				obj.onmousedown = function(event){
					
					//设置box1捕获所有鼠标按下的事件
					/*
					 * setCapture()
					 * 	- 只有IE支持，但是在火狐中调用时不会报错，
					 * 		而如果使用chrome调用，会报错
					 */
					/*if(box1.setCapture){
						box1.setCapture();
					}*/
					obj.setCapture && obj.setCapture();
					
					
					event = event || window.event;
					//div的偏移量 鼠标.clentX - 元素.offsetLeft
					//div的偏移量 鼠标.clentY - 元素.offsetTop
					var ol = event.clientX - obj.offsetLeft;
					var ot = event.clientY - obj.offsetTop;
					
					
					//为document绑定一个onmousemove事件
					document.onmousemove = function(event){
						event = event || window.event;
						//当鼠标移动时被拖拽元素跟随鼠标移动 onmousemove
						//获取鼠标的坐标
						var left = event.clientX - ol;
						var top = event.clientY - ot;
						
						//修改box1的位置
						obj.style.left = left+"px";
						obj.style.top = top+"px";
						
					};
					
					//为document绑定一个鼠标松开事件
					document.onmouseup = function(){
						//当鼠标松开时，被拖拽元素固定在当前位置	onmouseup
						//取消document的onmousemove事件
						document.onmousemove = null;
						//取消document的onmouseup事件
						document.onmouseup = null;
						//当鼠标松开时，取消对事件的捕获
						obj.releaseCapture && obj.releaseCapture();
					};
					
					/*
					 * 当我们拖拽一个网页中的内容时，浏览器会默认去搜索引擎中搜索内容，
					 * 	此时会导致拖拽功能的异常，这个是浏览器提供的默认行为，
					 * 	如果不希望发生这个行为，则可以通过return false来取消默认行为
					 * 
					 * 但是这招对IE8不起作用
					 */
					return false;
					
				};
			}
			
			
		</script>
	</head>
	<body>
		
		我是一段文字
		
		<div id="box1"></div>
		
		<div id="box2"></div>
		
		<img src="https://i.loli.net/2019/06/03/5cf48320914c851528.png" id="img1" style="position: absolute;"/>
	</body>
</html>

```

#### 滚动

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
				
				
				//获取id为box1的div
				var box1 = document.getElementById("box1");
				
				//为box1绑定一个鼠标滚轮滚动的事件
				/*
				 * onmousewheel鼠标滚轮滚动的事件，会在滚轮滚动时触发，
				 * 	但是火狐不支持该属性
				 * 
				 * 在火狐中需要使用 DOMMouseScroll 来绑定滚动事件
				 * 	注意该事件需要通过addEventListener()函数来绑定
				 */
				
				
				box1.onmousewheel = function(event){
					
					event = event || window.event;
					
					
					//event.wheelDelta 可以获取鼠标滚轮滚动的方向
					//向上滚 120   向下滚 -120
					//wheelDelta这个值我们不看大小，只看正负
					
					//alert(event.wheelDelta);
					
					//wheelDelta这个属性火狐中不支持
					//在火狐中使用event.detail来获取滚动的方向
					//向上滚 -3  向下滚 3
					//alert(event.detail);
					
					
					/*
					 * 当鼠标滚轮向下滚动时，box1变长
					 * 	当滚轮向上滚动时，box1变短
					 */
					//判断鼠标滚轮滚动的方向
					if(event.wheelDelta > 0 || event.detail < 0){
						//向上滚，box1变短
						box1.style.height = box1.clientHeight - 10 + "px";
						
					}else{
						//向下滚，box1变长
						box1.style.height = box1.clientHeight + 10 + "px";
					}
					
					/*
					 * 使用addEventListener()方法绑定响应函数，取消默认行为时不能使用return false
					 * 需要使用event来取消默认行为event.preventDefault();
					 * 但是IE8不支持event.preventDefault();这个玩意，如果直接调用会报错
					 */
					event.preventDefault && event.preventDefault();
					
					
					/*
					 * 当滚轮滚动时，如果浏览器有滚动条，滚动条会随之滚动，
					 * 这是浏览器的默认行为，如果不希望发生，则可以取消默认行为
					 */
					return false;
					
					
					
					
				};
				
				//为火狐绑定滚轮事件
				bind(box1,"DOMMouseScroll",box1.onmousewheel);
				
				
			};
			
			
			function bind(obj , eventStr , callback){
				if(obj.addEventListener){
					//大部分浏览器兼容的方式
					obj.addEventListener(eventStr , callback , false);
				}else{
					/*
					 * this是谁由调用方式决定
					 * callback.call(obj)
					 */
					//IE8及以下
					obj.attachEvent("on"+eventStr , function(){
						//在匿名函数中调用回调函数
						callback.call(obj);
					});
				}
			}
			
		</script>
	</head>
	<body style="height: 2000px;">
		
		<div id="box1"></div>
		
	</body>
</html>

```

#### 键盘事件

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script type="text/javascript">
			
			window.onload = function(){
				
				/*
				 * 键盘事件：
				 * 	onkeydown
				 * 		- 按键被按下
				 * 		- 对于onkeydown来说如果一直按着某个按键不松手，则事件会一直触发
				 * 		- 当onkeydown连续触发时，第一次和第二次之间会间隔稍微长一点，其他的会非常的快
				 * 			这种设计是为了防止误操作的发生。
				 * 	onkeyup
				 * 		- 按键被松开
				 * 
				 *  键盘事件一般都会绑定给一些可以获取到焦点的对象或者是document
				 */
				
				document.onkeydown = function(event){
					event = event || window.event;
					
					/*
					 * 可以通过keyCode来获取按键的编码
					 * 	通过它可以判断哪个按键被按下
					 * 除了keyCode，事件对象中还提供了几个属性
					 * 	altKey
					 * 	ctrlKey
					 * 	shiftKey
					 * 		- 这个三个用来判断alt ctrl 和 shift是否被按下
					 * 			如果按下则返回true，否则返回false
					 */
					
					//console.log(event.keyCode);
					
					//判断一个y是否被按下
					//判断y和ctrl是否同时被按下
					if(event.keyCode === 89 && event.ctrlKey){
						console.log("ctrl和y都被按下了");
					}
					
					
				};
				
				/*document.onkeyup = function(){
					console.log("按键松开了");
				};*/
				
				//获取input
				var input = document.getElementsByTagName("input")[0];
				
				input.onkeydown = function(event){
					
					event = event || window.event;
					
					//console.log(event.keyCode);
					//数字 48 - 57
					//使文本框中不能输入数字
					if(event.keyCode >= 48 && event.keyCode <= 57){
						//在文本框中输入内容，属于onkeydown的默认行为
						//如果在onkeydown中取消了默认行为，则输入的内容，不会出现在文本框中
						return false;
					}
					
					
				};
			};
			
			
		</script>
	</head>
	<body>
		
		<input type="text" />
		
	</body>
</html>
```

**移动div**

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
				position: absolute;
			}
			
			
		</style>
		
		<script type="text/javascript">
			
			//使div可以根据不同的方向键向不同的方向移动
			/*
			 * 按左键，div向左移
			 * 按右键，div向右移
			 * 。。。
			 */
			window.onload = function(){
				
				//为document绑定一个按键按下的事件
				document.onkeydown = function(event){
					event = event || window.event;
					
					//定义一个变量，来表示移动的速度
					var speed = 10;
					
					//当用户按了ctrl以后，速度加快
					if(event.ctrlKey){
						speed = 500;
					}
					
					/*
					 * 37 左
					 * 38 上
					 * 39 右
					 * 40 下
					 */
					switch(event.keyCode){
						case 37:
							//alert("向左"); left值减小
							box1.style.left = box1.offsetLeft - speed + "px";
							break;
						case 39:
							//alert("向右");
							box1.style.left = box1.offsetLeft + speed + "px";
							break;
						case 38:
							//alert("向上");
							box1.style.top = box1.offsetTop - speed + "px";
							break;
						case 40:
							//alert("向下");
							box1.style.top = box1.offsetTop + speed + "px";
							break;
					}
					
				};
				
			};
			
			
		</script>
	</head>
	<body>
		<div id="box1"></div>
	</body>
</html>

```











参考：

1. [深入浅出js事件](https://www.cnblogs.com/jingwhale/p/4656869.html)
2. [深入理解JS的事件绑定、事件流模型](https://www.jb51.net/article/139997.htm)
3. [js事件详解](https://www.cnblogs.com/BetterMyself/p/5651703.html) 
4. [js 事件的取消](https://blog.csdn.net/qq_35187942/article/details/85956726)