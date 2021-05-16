---
title: JavaScript BOM
date: 2019-05-04 16:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

由于JavaScript的出现就是为了能在浏览器中运行，所以，浏览器自然是JavaScript开发者必须要关注的。

<!--more-->

目前主流的浏览器分这么几种：

- IE 6~11：国内用得最多的IE浏览器，历来对W3C标准支持差。从IE10开始支持ES6标准；
- Chrome：Google出品的基于Webkit内核浏览器，内置了非常强悍的JavaScript引擎——V8。由于Chrome一经安装就时刻保持自升级，所以不用管它的版本，最新版早就支持ES6了；
- Safari：Apple的Mac系统自带的基于Webkit内核的浏览器，从OS X 10.7 Lion自带的6.1版本开始支持ES6，目前最新的OS X 10.11 El Capitan自带的Safari版本是9.x，早已支持ES6；
- Firefox：Mozilla自己研制的Gecko内核和JavaScript引擎OdinMonkey。早期的Firefox按版本发布，后来终于聪明地学习Chrome的做法进行自升级，时刻保持最新；
- 移动设备上目前iOS和Android两大阵营分别主要使用Apple的Safari和Google的Chrome，由于两者都是Webkit核心，结果HTML5首先在手机上全面普及（桌面绝对是Microsoft拖了后腿），对JavaScript的标准支持也很好，最新版本均支持ES6。

其他浏览器如Opera等由于市场份额太小就被自动忽略了。

另外还要注意识别各种国产浏览器，如某某安全浏览器，某某旋风浏览器，它们只是做了一个壳，其核心调用的是IE，也有号称同时支持IE和Webkit的“双核”浏览器。

不同的浏览器对JavaScript支持的差异主要是，有些API的接口不一样，比如AJAX，File接口。对于ES6标准，不同的浏览器对各个特性支持也不一样。

在编写JavaScript的时候，就要充分考虑到浏览器的差异，尽量让同一份JavaScript代码能运行在不同的浏览器中。

### 浏览器对象

JavaScript可以获取浏览器提供的很多对象，并进行操作。

#### window

`window`对象不但充当全局作用域，而且表示浏览器窗口。

`window`对象有`innerWidth`和`innerHeight`属性，可以获取浏览器窗口的内部宽度和高度。内部宽高是指除去菜单栏、工具栏、边框等占位元素后，用于显示网页的净宽高。

兼容性：IE<=8不支持。

对应的，还有一个`outerWidth`和`outerHeight`属性，可以获取浏览器窗口的整个宽高。

#### navigator

`navigator`对象表示浏览器的信息，最常用的属性包括：

- navigator.appName：浏览器名称；
- navigator.appVersion：浏览器版本；
- navigator.language：浏览器设置的语言；
- navigator.platform：操作系统类型；
- navigator.userAgent：浏览器设定的`User-Agent`字符串。

```javascript
var ua = navigator.userAgent;
			
			console.log(ua);
			
			if(/firefox/i.test(ua)){
				alert("你是火狐！！！");
			}else if(/chrome/i.test(ua)){
				alert("你是Chrome");
			}else if(/msie/i.test(ua)){
				alert("你是IE浏览器~~~");
			}else if("ActiveXObject" in window){
				alert("你是IE11，枪毙了你~~~");
			}
/*
			 * 如果通过UserAgent不能判断，还可以通过一些浏览器中特有的对象，来判断浏览器的信息
			 * 比如：ActiveXObject
			 */
			if("ActiveXObject" in window){
				alert("你是IE，我已经抓住你了~~~");
			}else{
				alert("你不是IE~~~");
			}
			
			alert("ActiveXObject" in window);
```

*请注意*，`navigator`的信息可以很容易地被用户修改，所以JavaScript读取的值不一定是正确的。很多初学者为了针对不同浏览器编写不同的代码，喜欢用`if`判断浏览器版本，例如：

```
var width;
if (getIEVersion(navigator.userAgent) < 9) {
    width = document.body.clientWidth;
} else {
    width = window.innerWidth;
}
```

但这样既可能判断不准确，也很难维护代码。正确的方法是充分利用JavaScript对不存在属性返回`undefined`的特性，直接用短路运算符`||`计算：

```
var width = window.innerWidth || document.body.clientWidth;
```

#### screen

`screen`对象表示屏幕的信息，常用的属性有：

- screen.width：屏幕宽度，以像素为单位；
- screen.height：屏幕高度，以像素为单位；
- screen.colorDepth：返回颜色位数，如8、16、24。

#### location

`location`对象表示当前页面的URL信息。例如，一个完整的URL：

```
http://www.example.com:8080/path/index.html?a=1&b=2#TOP
```

可以用`location.href`获取。要获得URL各个部分的值，可以这么写：

```
location.protocol; // 'http'
location.host; // 'www.example.com'
location.port; // '8080'
location.pathname; // '/path/index.html'
location.search; // '?a=1&b=2'
location.hash; // 'TOP'
```

要加载一个新页面，可以调用`location.assign()`。如果要重新加载当前页面，调用`location.reload()`方法非常方便。

#### document

`document`对象表示当前页面。由于HTML在浏览器中以DOM形式表示为树形结构，`document`对象就是整个DOM树的根节点。

`document`的`title`属性是从HTML文档中的`<title>xxx</title>`读取的，但是可以动态改变

```
document.title = '努力学习JavaScript!';
```

请观察浏览器窗口标题的变化。

要查找DOM树的某个节点，需要从`document`对象开始查找。最常用的查找是根据ID和Tag Name。

我们先准备HTML数据：

```
<dl id="drink-menu" style="border:solid 1px #ccc;padding:6px;">
    <dt>摩卡</dt>
    <dd>热摩卡咖啡</dd>
    <dt>酸奶</dt>
    <dd>北京老酸奶</dd>
    <dt>果汁</dt>
    <dd>鲜榨苹果汁</dd>
</dl>
```

用`document`对象提供的`getElementById()`和`getElementsByTagName()`可以按ID获得一个DOM节点和按Tag名称获得一组DOM节点：

```
var menu = document.getElementById('drink-menu');
var drinks = document.getElementsByTagName('dt');
var i, s, menu, drinks;

menu = document.getElementById('drink-menu');
menu.tagName; // 'DL'

drinks = document.getElementsByTagName('dt');
s = '提供的饮料有:';
for (i=0; i<drinks.length; i++) {
    s = s + drinks[i].innerHTML + ',';
}
console.log(s);
```

`document`对象还有一个`cookie`属性，可以获取当前页面的Cookie。

Cookie是由服务器发送的key-value标示符。因为HTTP协议是无状态的，但是服务器要区分到底是哪个用户发过来的请求，就可以用Cookie来区分。当一个用户成功登录后，服务器发送一个Cookie给浏览器，例如`user=ABC123XYZ(加密的字符串)...`，此后，浏览器访问该网站时，会在请求头附上这个Cookie，服务器根据Cookie即可区分出用户。

Cookie还可以存储网站的一些设置，例如，页面显示的语言等等。

JavaScript可以通过`document.cookie`读取到当前页面的Cookie。

由于JavaScript能读取到页面的Cookie，而用户的登录信息通常也存在Cookie中，这就造成了巨大的安全隐患，这是因为在HTML页面中引入第三方的JavaScript代码是允许的：

```
<!-- 当前页面在wwwexample.com -->
<html>
    <head>
        <script src="http://www.foo.com/jquery.js"></script>
    </head>
    ...
</html>
```

如果引入的第三方的JavaScript中存在恶意代码，则`www.foo.com`网站将直接获取到`www.example.com`网站的用户登录信息。

为了解决这个问题，服务器在设置Cookie时可以使用`httpOnly`，设定了`httpOnly`的Cookie将不能被JavaScript读取。这个行为由浏览器实现，主流浏览器均支持`httpOnly`选项，IE从IE6 SP1开始支持。

为了确保安全，服务器端在设置Cookie时，应该始终坚持使用`httpOnly`。

#### history

`history`对象保存了浏览器的历史记录，JavaScript可以调用`history`对象的`back()`或`forward ()`，相当于用户点击了浏览器的“后退”或“前进”按钮。

这个对象属于历史遗留对象，对于现代Web页面来说，由于大量使用AJAX和页面交互，简单粗暴地调用`history.back()`可能会让用户感到非常愤怒。

新手开始设计Web页面时喜欢在登录页登录成功时调用`history.back()`，试图回到登录前的页面。这是一种错误的方法。

JavaScript返回上一页代码区别：

```
window.history.go(-1);  //返回上一页
window.history.back();  //返回上一页
//如果要强行刷新的话就是：window.history.back();location.reload();
window.location.go(-1); //刷新上一页
```

任何情况，你都不应该使用`history`这个对象了。

### console

对于前端开发者来说，在开发过程中需要监控某些表达式或变量的值的时候，用 debugger 会显得过于笨重，取而代之则是会将值输出到控制台上方便调试。最常用的语句就是`console.log(expression)`了。

然而对于作为一个全局对象的`console`对象来说，大多数人了解得还并不全面，当然我也是，经过我的一番学习，现在对于这个能玩转控制台的 JS 对象有了一定的认识，想与大家分享一下。

console 对象除了`console.log()`这一最常被开发者使用的方法之外，还有很多其他的方法。灵活运用这些方法，可以给开发过程增添许多便利。

#### 方法

**console.assert(expression, object[, object...])**

接收至少两个参数，第一个参数的值或返回值为`false`的时候，将会在控制台上输出后续参数的值。例如：

```
console.assert(1 == 1, object); // 无输出，返回 undefined
console.assert(1 == 2, object); // 输出 object
```

**console.count([label])**

输出执行到该行的次数，可选参数 label 可以输出在次数之前，例如：

```
(function() {
  for (var i = 0; i < 5; i++) {
    console.count('count');
  }
})();
// count: 1
// count: 2
// count: 3
// count: 4
// count: 5
```

**console.dir(object)**

将传入对象的属性，包括子对象的属性以列表形式输出，例如：

```
var obj = {
  name: 'classicemi',
  college: 'HUST',
  major: 'ei'
};
console.dir(obj);
```

输出如下：

![](https://i.loli.net/2019/05/30/5cef6f6cdb1f218460.png)

**console.error(object[, object...])**

用于输出错误信息，用法和常见的`console.log`一样，不同点在于输出内容会标记为错误的样式，便于分辨。输出结果：

![](https://i.loli.net/2019/05/30/5cef6fd9d6d5612188.png)

**console.group**

这是个有趣的方法，它能够让控制台输出的语句产生不同的层级嵌套关系，每一个`console.group()`会增加一层嵌套，相反要减少一层嵌套可以使用`console.groupEnd()`方法。语言表述比较无力，看代码：

```
console.log('这是第一层');
console.group();
console.log('这是第二层');
console.log('依然第二层');
console.group();
console.log('第三层了');
console.groupEnd();
console.log('回到第二层');
console.groupEnd();
console.log('回到第一层');
```

输出如下：

![](https://i.loli.net/2019/05/30/5cef701ca5be956048.png)

和`console.group()`相似的方法是`console.groupCollapsed()`作用相同，不同点是嵌套的输出内容是折叠状态，在有大段内容输出的时候使用这个方法可以使输出版面不至于太长。。。吧

**console.info(object[, object...])**

此方法与之前说到的`console.error`一样，用于输出信息，没有什么特别之处。

```
console.info('info'); // 输出 info
```

**console.table()**

可将传入的对象，或数组以表格形式输出，相比传统树形输出，这种输出方案更适合内部元素排列整齐的对象或数组，不然可能会出现很多的 undefined。

```
var obj = {
  foo: {
    name: 'foo',
    age: '33'
  },
  bar: {
    name: 'bar',
    age: '45'
  }
};

var arr = [
  ['foo', '33'],
  ['bar', '45']
];

console.table(obj);
console.table(arr);
```

输出结果：

![](https://i.loli.net/2019/05/30/5cef707b880f642231.png)

**console.log(object[, object...])**

这个不用多说，这个应该是开发者最常用的吧，也不知道是谁规定的。。。

```
console.log('log'); // 输出 log
```

**console.profile([profileLabel])**

这是个挺高大上的东西，可用于性能分析。在 JS 开发中，我们常常要评估段代码或是某个函数的性能。在函数中手动打印时间固然可以，但显得不够灵活而且有误差。借助控制台以及`console.profile()`方法我们可以很方便地监控运行性能。

例如下面这段代码：

```
function parent() {
  for (var i = 0; i < 10000; i++) {
    childA()
  }
}

function childA(j) {
  for (var i = 0; i < j; i++) {}
}

console.profile('性能分析');
parent();
console.profileEnd('性能分析');
```

然后我们可以在 Profiles 面板下看到上述代码运行过程中的消耗时间。页面加载过程的性能优化是前端开发的一个重要部分，使用控制台的 profiles 面板可以监控所有 JS 的运行情况使用方法就像录像机一样，控制台会把运行过程录制下来。

**console.time(name)**
计时器，可以将成对的`console.time()`和`console.timeEnd()`之间代码的运行时间输出到控制台上，`name`参数可作为标签名。

```
console.time('计时器');
for (var i = 0; i < 100; i++) {
  for (var j = 0; j < 100; j++) {
  for (var k = 0; k < 100; k++) {}
  }
}
console.timeEnd('计时器');
```

**console.trace()**

`console.trace()`用来追踪函数的调用过程。在大型项目尤其是框架开发中，函数的调用轨迹可以十分复杂，`console.trace()`方法可以将函数的被调用过程清楚地输出到控制台上。

```
function tracer(a) {
  console.trace();
  return a;
}

function foo(a) {
  return bar(a);
}

function bar(a) {
  return tracer(a);
}

var a = foo('tracer');
```

**console.warn(object[, object...])**

输出参数的内容，作为警告提示。

```
console.warn('warn'); // 输出 warn
```

**占位符**

`console`对象上的五个直接输出方法，`console.log()`,`console.warn()`,`console.error()`,`console.exception()`(等同于`console.error()`)和`console.info()`，都可以使用占位符。支持的占位符有四种，分别是字符(`%s`)、整数(`%d` 或 `%i`)、浮点数(`%f`)和对象(`%o`)。

```
console.log('%s是%d年%d月%d日', '今天', 2014, 4, 15);
console.log('圆周率是%f', 3.14159);

var obj = {
  name: 'hyp'
}
console.log('%o', obj);
```

还有一种特殊的标示符`%c`，对输出的文字可以附加特殊的样式，当进行大型项目开发的时候，代码中可能有很多其他开发者添加的控制台语句。开发者对自己的输出定制特别的样式就可以方便自己在眼花缭乱的输出结果中一眼看到自己需要的内容。想象力丰富的童鞋也可以做出有创意的输出信息，比如常见的招聘信息和个人介绍啥的。

```
console.log('%cMy name is hyp.', 'color: #fff; background: #f40; font-size: 24px;');
```

`%c`标示符可以用各种 CSS 语句来为输出添加样式，再随便举个栗子，`background`属性的`url()`中添加图片路径就可以实现图片的输出了。

#### 定时调用

在javascritp中，有两个关于定时器的专用函数，分别为：

1. 倒计定时器：`timename=setTimeout("function();",delaytime);`

2. 循环定时器：`timename=setInterval("function();",delaytime);`

第一个参数“function()”是定时器触发时要执行的动作，可以是一个函数，也可以是几个函数，函数间用“；”隔开即可。

比如要弹出两个警告窗口，便可将“function();”换成 “alert('第一个警告窗口!');alert('第二个警告窗口!');”；

而第二个参数“delaytime”则是间隔的时间，以毫秒为单位，即填写“5000”，就表示5秒钟。 　　

倒计时定时器是在指定时间到达后触发事件，而循环定时器就是在间隔时间到来时反复触发事件，

两者的区别在于：前者只是作用一次，而后者则不停地作用。

比如你打开一个页面后，想间隔几秒自动跳转到另一个页面，则你就需要采用倒计定时器“setTimeout("function();",delaytime)”  ，而如果想将某一句话设置成一个一个字的出现， 则需要用到循环定时器“setInterval("function();",delaytime)”   。获取表单的焦点，则用到document.activeElement.id。利用if来判断document.activeElement.id和表单的ID是否相同。  比如：

```
if ("mid" == document.activeElement.id) {alert();}
```

"mid"便是表单对应的ID。

定时器：

用以指定在一段特定的时间后执行某段程序。

JS中定时执行,setTimeout和setInterval的区别,以及l解除方法

setTimeout(Expression,DelayTime),在DelayTime过后,将执行一次Expression,setTimeout 运用在延迟一段时间，再进行某项操作。 setTimeout("function",time) 设置一个超时对象

setInterval(expression,delayTime),每个DelayTime,都将执行Expression.常常可用于刷新表达式. setInterval("function",time) 设置一个超时对象

SetInterval为自动重复，setTimeout不会重复。

clearTimeout(对象) 清除已设置的setTimeout对象 clearInterval(对象) 清除已设置的setInterval对象



```
/*
* setInterval()
* 	- 定时调用
* 	- 可以将一个函数，每隔一段时间执行一次
* 	- 参数：
* 		1.回调函数，该函数会每隔一段时间被调用一次
* 		2.每次调用间隔的时间，单位是毫秒
* 
* 	- 返回值：
* 		返回一个Number类型的数据
* 		这个数字用来作为定时器的唯一标识
*/
var num = 1;

var timer = setInterval(function(){

count.innerHTML = num++;

if(num == 11){
//关闭定时器
clearInterval(timer);
}

},1000);
```

示例：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script type="text/javascript">
			
			window.onload = function(){
				
				/*
				 * 使图片可以自动切换
				 */
				
				//获取img标签
				var img1 = document.getElementById("img1");
				
				//创建一个数组来保存图片的路径
				var imgArr = ["img/1.jpg","img/2.jpg","img/3.jpg","img/4.jpg","img/5.jpg"];
				
				//创建一个变量，用来保存当前图片的索引
				var index = 0;
				
				//定义一个变量，用来保存定时器的标识
				var timer;
				
				//为btn01绑定一个单击响应函数
				var btn01 = document.getElementById("btn01");
				btn01.onclick = function(){
					
					/*
					 * 目前，我们每点击一次按钮，就会开启一个定时器，
					 * 	点击多次就会开启多个定时器，这就导致图片的切换速度过快，
					 * 	并且我们只能关闭最后一次开启的定时器
					 */
					
					//在开启定时器之前，需要将当前元素上的其他定时器关闭
					clearInterval(timer);
					
					/*
					 * 开启一个定时器，来自动切换图片
					 */
					timer = setInterval(function(){
						//使索引自增
						index++;
						//判断索引是否超过最大索引
						/*if(index >= imgArr.length){
							//则将index设置为0
							index = 0;
						}*/
						index %= imgArr.length;
						//修改img1的src属性
						img1.src = imgArr[index];
						
					},1000);
				};
				
				//为btn02绑定一个单击响应函数
				var btn02 = document.getElementById("btn02");
				btn02.onclick = function(){
					//点击按钮以后，停止图片的自动切换，关闭定时器
					/*
					 * clearInterval()可以接收任意参数，
					 * 	如果参数是一个有效的定时器的标识，则停止对应的定时器
					 * 	如果参数不是一个有效的标识，则什么也不做
					 */
					clearInterval(timer);
					
				};
				
				
			};
			
		</script>
	</head>
	<body>
		
		<img id="img1" src="img/1.jpg"/>
		<br /><br />
		<button id="btn01">开始</button>
		<button id="btn02">停止</button>
	</body>
</html>
```

#### 延时调用

```
/*
* 延时调用，
* 	延时调用一个函数不马上执行，而是隔一段时间以后在执行，而且只会执行一次
* 
* 延时调用和定时调用的区别，定时调用会执行多次，而延时调用只会执行一次
* 
* 延时调用和定时调用实际上是可以互相代替的，在开发中可以根据自己需要去选择
*/
var timer = setTimeout(function(){
console.log(num++);
},3000);

//使用clearTimeout()来关闭一个延时调用
clearTimeout(timer);
```

示例：

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
			
			#box1{
				width: 100px;
				height: 100px;
				background-color: red;
				position: absolute;
				left: 0;
			}
			
			#box2{
				width: 100px;
				height: 100px;
				background-color: yellow;
				position: absolute;
				left: 0;
				top: 200px;
			}
			
		</style>
		<script type="text/javascript" src="js/tools.js"></script>
		<script type="text/javascript">
			
			window.onload = function(){
				
				//获取box1
				var box1 = document.getElementById("box1");
				//获取btn01
				var btn01 = document.getElementById("btn01");
				
				//获取btn02
				var btn02 = document.getElementById("btn02");
				
				
				//点击按钮以后，使box1向右移动（left值增大）
				btn01.onclick = function(){
					move(box1 ,"left", 800 , 20);
				};
				
				
				//点击按钮以后，使box1向左移动（left值减小）
				btn02.onclick = function(){
					move(box1 ,"left", 0 , 10);
				};
				
				
				//获取btn03
				var btn03 = document.getElementById("btn03");
				btn03.onclick = function(){
					move(box2 , "left",800 , 10);
				};
				
				//测试按钮
				var btn04 = document.getElementById("btn04");
				btn04.onclick = function(){
					//move(box2 ,"width", 800 , 10);
					//move(box2 ,"top", 800 , 10);
					//move(box2 ,"height", 800 , 10);
					move(box2 , "width" , 800 , 10 , function(){
						move(box2 , "height" , 400 , 10 , function(){
							move(box2 , "top" , 0 , 10 , function(){
								move(box2 , "width" , 100 , 10 , function(){
							
								});
							});
						});
					});
				};
			};
			
			//定义一个变量，用来保存定时器的标识
			/*
			 * 目前我们的定时器的标识由全局变量timer保存，
			 * 	所有的执行正在执行的定时器都在这个变量中保存
			 */
			//var timer;
//尝试创建一个可以执行简单动画的函数
/*
 * 参数：
 * 	obj:要执行动画的对象
 * 	attr:要执行动画的样式，比如：left top width height
 * 	target:执行动画的目标位置
 * 	speed:移动的速度(正数向右移动，负数向左移动)
 *  callback:回调函数，这个函数将会在动画执行完毕以后执行
 */
function move(obj, attr, target, speed, callback) {
	//关闭上一个定时器
	clearInterval(obj.timer);

	//获取元素目前的位置
	var current = parseInt(getStyle(obj, attr));

	//判断速度的正负值
	//如果从0 向 800移动，则speed为正
	//如果从800向0移动，则speed为负
	if(current > target) {
		//此时速度应为负值
		speed = -speed;
	}

	//开启一个定时器，用来执行动画效果
	//向执行动画的对象中添加一个timer属性，用来保存它自己的定时器的标识
	obj.timer = setInterval(function() {

		//获取box1的原来的left值
		var oldValue = parseInt(getStyle(obj, attr));

		//在旧值的基础上增加
		var newValue = oldValue + speed;

		//判断newValue是否大于800
		//从800 向 0移动
		//向左移动时，需要判断newValue是否小于target
		//向右移动时，需要判断newValue是否大于target
		if((speed < 0 && newValue < target) || (speed > 0 && newValue > target)) {
			newValue = target;
		}

		//将新值设置给box1
		obj.style[attr] = newValue + "px";

		//当元素移动到0px时，使其停止执行动画
		if(newValue == target) {
			//达到目标，关闭定时器
			clearInterval(obj.timer);
			//动画执行完毕，调用回调函数
			callback && callback();
		}

	}, 30);
}

/*
 * 定义一个函数，用来获取指定元素的当前的样式
 * 参数：
 * 		obj 要获取样式的元素
 * 		name 要获取的样式名
 */
function getStyle(obj, name) {

	if(window.getComputedStyle) {
		//正常浏览器的方式，具有getComputedStyle()方法
		return getComputedStyle(obj, null)[name];
	} else {
		//IE8的方式，没有getComputedStyle()方法
		return obj.currentStyle[name];
	}

}				
			
		</script>
	</head>
	<body>
		
		<button id="btn01">点击按钮以后box1向右移动</button>
		<button id="btn02">点击按钮以后box1向左移动</button>
		<button id="btn03">点击按钮以后box2向右移动</button>
		<button id="btn04">测试按钮</button>
		
		<br /><br />
		
		<div id="box1"></div>
		<div id="box2"></div>
		
		<div style="width: 0; height: 1000px; border-left:1px black solid; position: absolute; left: 800px;top:0;"></div>
		
	</body>
</html>

```

轮播图

```
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
			
			/*
			 * 设置outer的样式
			 */
			#outer{
				/*设置宽和高*/
				width: 520px;
				height: 333px;
				/*居中*/
				margin: 50px auto;
				/*设置背景颜色*/
				background-color: greenyellow;
				/*设置padding*/
				padding: 10px 0;
				/*开启相对定位*/
				position: relative;
				/*裁剪溢出的内容*/
				overflow: hidden;
			}
			
			/*设置imgList*/
			#imgList{
				/*去除项目符号*/
				list-style: none;
				/*设置ul的宽度*/
				/*width: 2600px;*/
				/*开启绝对定位*/
				position: absolute;
				/*设置偏移量*/
				/*
				 * 每向左移动520px，就会显示到下一张图片
				 */
				left: 0px;
			}
			
			/*设置图片中的li*/
			#imgList li{
				/*设置浮动*/
				float: left;
				/*设置左右外边距*/
				margin: 0 10px;
			}
			
			/*设置导航按钮*/
			#navDiv{
				/*开启绝对定位*/
				position: absolute;
				/*设置位置*/
				bottom: 15px;
				/*设置left值
				 	outer宽度  520
				 	navDiv宽度 25*5 = 125
				 		520 - 125 = 395/2 = 197.5
				 * */
				/*left: 197px;*/
			}
			
			#navDiv a{
				/*设置超链接浮动*/
				float: left;
				/*设置超链接的宽和高*/
				width: 15px;
				height: 15px;
				/*设置背景颜色*/
				background-color: red;
				/*设置左右外边距*/
				margin: 0 5px;
				/*设置透明*/
				opacity: 0.5;
				/*兼容IE8透明*/
				filter: alpha(opacity=50);
			}
			
			/*设置鼠标移入的效果*/
			#navDiv a:hover{
				background-color: black;
			}
		</style>
		
		<!--引用工具-->
		<script type="text/javascript" src="js/tools.js"></script>
		<script type="text/javascript">
			window.onload = function(){
				//获取imgList
				var imgList = document.getElementById("imgList");
				//获取页面中所有的img标签
				var imgArr = document.getElementsByTagName("img");
				//设置imgList的宽度
				imgList.style.width = 520*imgArr.length+"px";
				
				
				/*设置导航按钮居中*/
				//获取navDiv
				var navDiv = document.getElementById("navDiv");
				//获取outer
				var outer = document.getElementById("outer");
				//设置navDiv的left值
				navDiv.style.left = (outer.offsetWidth - navDiv.offsetWidth)/2 + "px";
				
				//默认显示图片的索引
				var index = 0;
				//获取所有的a
				var allA = document.getElementsByTagName("a");
				//设置默认选中的效果
				allA[index].style.backgroundColor = "black";
				
				/*
				 	点击超链接切换到指定的图片
				 		点击第一个超链接，显示第一个图片
				 		点击第二个超链接，显示第二个图片
				 * */
				
				//为所有的超链接都绑定单击响应函数
				for(var i=0; i<allA.length ; i++){
					
					//为每一个超链接都添加一个num属性
					allA[i].num = i;
					
					//为超链接绑定单击响应函数
					allA[i].onclick = function(){
						
						//关闭自动切换的定时器
						clearInterval(timer);
						//获取点击超链接的索引,并将其设置为index
						index = this.num;
						
						//切换图片
						/*
						 * 第一张  0 0
						 * 第二张  1 -520
						 * 第三张  2 -1040
						 */
						//imgList.style.left = -520*index + "px";
						//设置选中的a
						setA();
						
						//使用move函数来切换图片
						move(imgList , "left" , -520*index , 20 , function(){
							//动画执行完毕，开启自动切换
							autoChange();
						});
						
					};
				}
				
				
				//开启自动切换图片
				autoChange();
				
				
				//创建一个方法用来设置选中的a
				function setA(){
					
					//判断当前索引是否是最后一张图片
					if(index >= imgArr.length - 1){
						//则将index设置为0
						index = 0;
						
						//此时显示的最后一张图片，而最后一张图片和第一张是一摸一样
						//通过CSS将最后一张切换成第一张
						imgList.style.left = 0;
					}
					
					//遍历所有a，并将它们的背景颜色设置为红色
					for(var i=0 ; i<allA.length ; i++){
						allA[i].style.backgroundColor = "";
					}
					
					//将选中的a设置为黑色
					allA[index].style.backgroundColor = "black";
				};
				
				//定义一个自动切换的定时器的标识
				var timer;
				//创建一个函数，用来开启自动切换图片
				function autoChange(){
					
					//开启一个定时器，用来定时去切换图片
					timer = setInterval(function(){
						
						//使索引自增
						index++;
						
						//判断index的值
						index %= imgArr.length;
						
						//执行动画，切换图片
						move(imgList , "left" , -520*index , 20 , function(){
							//修改导航按钮
							setA();
						});
						
					},3000);
					
				}
				
				
			};
			
		</script>
	</head>
	<body>
		<!-- 创建一个外部的div，来作为大的容器 -->
		<div id="outer">
			<!-- 创建一个ul，用于放置图片 -->
			<ul id="imgList">
				<li><img src="img/1.jpg"/></li>
				<li><img src="img/2.jpg"/></li>
				<li><img src="img/3.jpg"/></li>
				<li><img src="img/4.jpg"/></li>
				<li><img src="img/5.jpg"/></li>
				<li><img src="img/1.jpg"/></li>
			</ul>
			<!--创建导航按钮-->
			<div id="navDiv">
				<a href="javascript:;"></a>
				<a href="javascript:;"></a>
				<a href="javascript:;"></a>
				<a href="javascript:;"></a>
				<a href="javascript:;"></a>
			</div>
		</div>
	</body>
</html>

```

#### 类操作

```
//定义一个函数，用来向一个元素中添加指定的class属性值
/*
 * 参数:
 * 	obj 要添加class属性的元素
 *  cn 要添加的class值
 * 	
 */
function addClass(obj, cn) {

	//检查obj中是否含有cn
	if(!hasClass(obj, cn)) {
		obj.className += " " + cn;
	}

}

/*
 * 判断一个元素中是否含有指定的class属性值
 * 	如果有该class，则返回true，没有则返回false
 * 	
 */
function hasClass(obj, cn) {

	//判断obj中有没有cn class
	//创建一个正则表达式
	//var reg = /\bb2\b/;
	var reg = new RegExp("\\b" + cn + "\\b");

	return reg.test(obj.className);

}

/*
 * 删除一个元素中的指定的class属性
 */
function removeClass(obj, cn) {
	//创建一个正则表达式
	var reg = new RegExp("\\b" + cn + "\\b");

	//删除class
	obj.className = obj.className.replace(reg, "");

}

/*
 * toggleClass可以用来切换一个类
 * 	如果元素中具有该类，则删除
 * 	如果元素中没有该类，则添加
 */
function toggleClass(obj, cn) {

	//判断obj中是否含有cn
	if(hasClass(obj, cn)) {
		//有，则删除
		removeClass(obj, cn);
	} else {
		//没有，则添加
		addClass(obj, cn);
	}

}
```

