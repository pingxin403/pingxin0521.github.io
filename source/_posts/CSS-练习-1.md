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

### 常用属性

#### cursor

cursor 属性规定要显示的光标的类型（形状）。该属性定义了鼠标指针放在一个元素边界范围内时所用的光标形状（不过 CSS2.1 没有定义由哪个边界确定这个范围）。

默认：`auto`，可继承。

JavaScript 语法：`object.style.cursor="crosshair"`

| 值        | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| *url*     | 需使用的自定义光标的 URL。 注释：请在此列表的末端始终定义一种普通的光标，以防没有由 URL 定义的可用光标。 |
| default   | 默认光标（通常是一个箭头）                                   |
| auto      | 默认。浏览器设置的光标。                                     |
| crosshair | 光标呈现为十字线。                                           |
| pointer   | 光标呈现为指示链接的指针（一只手）                           |
| move      | 此光标指示某对象可被移动。                                   |
| e-resize  | 此光标指示矩形框的边缘可被向右（东）移动。                   |
| ne-resize | 此光标指示矩形框的边缘可被向上及向右移动（北/东）。          |
| nw-resize | 此光标指示矩形框的边缘可被向上及向左移动（北/西）。          |
| n-resize  | 此光标指示矩形框的边缘可被向上（北）移动。                   |
| se-resize | 此光标指示矩形框的边缘可被向下及向右移动（南/东）。          |
| sw-resize | 此光标指示矩形框的边缘可被向下及向左移动（南/西）。          |
| s-resize  | 此光标指示矩形框的边缘可被向下移动（南）。                   |
| w-resize  | 此光标指示矩形框的边缘可被向左移动（西）。                   |
| text      | 此光标指示文本。                                             |
| wait      | 此光标指示程序正忙（通常是一只表或沙漏）。                   |
| help      | 此光标指示可用的帮助（通常是一个问号或一个气球）。           |

示例：

```html
<html>

<body>
<p>请把鼠标移动到单词上，可以看到鼠标指针发生变化：</p>
<span style="cursor:auto">
Auto</span><br />
<span style="cursor:crosshair">
Crosshair</span><br />
<span style="cursor:default">
Default</span><br />
<span style="cursor:pointer">
Pointer</span><br />
<span style="cursor:move">
Move</span><br />
<span style="cursor:e-resize">
e-resize</span><br />
<span style="cursor:ne-resize">
ne-resize</span><br />
<span style="cursor:nw-resize">
nw-resize</span><br />
<span style="cursor:n-resize">
n-resize</span><br />
<span style="cursor:se-resize">
se-resize</span><br />
<span style="cursor:sw-resize">
sw-resize</span><br />
<span style="cursor:s-resize">
s-resize</span><br />
<span style="cursor:w-resize">
w-resize</span><br />
<span style="cursor:text">
text</span><br />
<span style="cursor:wait">
wait</span><br />
<span style="cursor:help">
help</span>
</body>

</html>
```

#### transition

transition 属性是一个简写属性，用于设置四个过渡属性：

- transition-property： 规定设置过渡效果的 CSS 属性的名称。
- transition-duration：规定完成过渡效果需要多少秒或毫秒。
- transition-timing-function：规定速度效果的速度曲线。
- transition-delay：定义过渡效果何时开始。

注：请始终设置 transition-duration 属性，否则时长为 0，就不会产生过渡效果。

默认值：`all 0 ease 0`，不可继承

JavaScript 语法： 	`object.style.transition="width 2s"`

语法：`transition: property duration timing-function delay;`

示例：将鼠标悬停在一个 div 元素上，逐步改变表格的宽度从 100px 到 300px

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title>
<style> 
div
{
	width:100px;
	height:100px;
	background:red;
	transition:width 2s;
	-webkit-transition:width 2s; /* Safari */
}

div:hover
{
	width:300px;
}
</style>
</head>
<body>
<p><b>注意：</b>该实例无法在 Internet Explorer 9 及更早 IE 版本上工作。</p>

<div></div>

<p>鼠标移动到 div 元素上，查看过渡效果。</p>

</body>
</html>
```

#### transform

CSS3 转换可以可以对元素进行移动、缩放、转动、拉长或拉伸。适用于2D或3D转换的元素

**2D 转换**

- translate()：根据左(X轴)和顶部(Y轴)位置给定的参数，从当前元素位置移动。

- rotate()：在一个给定度数顺时针旋转的元素。负值是允许的，这样是元素逆时针旋转

- scale()：该元素增加或减少的大小，取决于宽度（X轴）和高度（Y轴）的参数

- skew()：包含两个参数值，分别表示X轴和Y轴倾斜的角度，如果第二个参数为空，则默认为0，参数为负表示向相反方向倾斜。

      skewX(<angle>);表示只在X轴(水平方向)倾斜。
      skewY(<angle>);表示只在Y轴(垂直方向)倾斜。

- matrix()：有六个参数，包含旋转，缩放，移动（平移）和倾斜功能。

| 函数                            | 描述                                     |
| ------------------------------- | ---------------------------------------- |
| matrix(*n*,*n*,*n*,*n*,*n*,*n*) | 定义 2D 转换，使用六个值的矩阵。         |
| translate(*x*,*y*)              | 定义 2D 转换，沿着 X 和 Y 轴移动元素。   |
| translateX(*n*)                 | 定义 2D 转换，沿着 X 轴移动元素。        |
| translateY(*n*)                 | 定义 2D 转换，沿着 Y 轴移动元素。        |
| scale(*x*,*y*)                  | 定义 2D 缩放转换，改变元素的宽度和高度。 |
| scaleX(*n*)                     | 定义 2D 缩放转换，改变元素的宽度。       |
| scaleY(*n*)                     | 定义 2D 缩放转换，改变元素的高度。       |
| rotate(*angle*)                 | 定义 2D 旋转，在参数中规定角度。         |
| skew(*x-angle*,*y-angle*)       | 定义 2D 倾斜转换，沿着 X 和 Y 轴。       |
| skewX(*angle*)                  | 定义 2D 倾斜转换，沿着 X 轴。            |
| skewY(*angle*)                  | 定义 2D 倾斜转换，沿着 Y 轴。            |

示例：

```html
<h1>Css3 Transform</h1>
<!-- Rotate-->
<div class="card">
  <div class="box rotate">
    <div class="fill"></div>
  </div>
  <p>rotate(45deg)  </p>
</div>
<div class="card">
  <div class="box rotateX">
    <div class="fill"></div>
  </div>
  <p>rotateX(45deg)</p>
</div>
<div class="card">
  <div class="box rotateY">
    <div class="fill"></div>
  </div>
  <p>rotateY(45deg)</p>
</div>
<div class="card">
  <div class="box rotateZ">
    <div class="fill"></div>
  </div>
  <p>rotateZ(45deg)  </p>
</div>
<!-- scale-->
<div class="card">
  <div class="box scale">
    <div class="fill"></div>
  </div>
  <p>scale(2)</p>
</div>
<div class="card">
  <div class="box scaleX">
    <div class="fill"></div>
  </div>
  <p>scaleX(2)    </p>
</div>
<div class="card">
  <div class="box scaleY">
    <div class="fill"></div>
  </div>
  <p>scaleY(2)    </p>
</div>
<!-- skew-->
<div class="card">
  <div class="box skew">
    <div class="fill"></div>
  </div>
  <p>skew(45deg, 45deg)  </p>
</div>
<div class="card">
  <div class="box skewX">
    <div class="fill"></div>
  </div>
  <p>skewX(45deg)</p>
</div>
<div class="card">
  <div class="box skewY">
    <div class="fill"></div>
  </div>
  <p>skewY(45deg)</p>
</div>
<!-- translate-->
<div class="card">
  <div class="box translate">
    <div class="fill"></div>
  </div>
  <p>translate(45px)  </p>
</div>
<div class="card">
  <div class="box translateX">
    <div class="fill"></div>
  </div>
  <p>translateX(45px)</p>
</div>
<div class="card">
  <div class="box translateY">
    <div class="fill"></div>
  </div>
  <p>translateY(45px)</p>
</div>
<div class="card">
  <div class="box matrix">
    <div class="fill"></div>
  </div>
  <p> matrix(2, 2, 0, 2, 45, 0)</p>
</div>
<h4>Perspective : 100</h4>
<div class="perspective-100">
  <div class="card">
    <div class="box rotateX">
      <div class="fill"></div>
    </div>
    <p>rotateX(90deg)</p>
  </div>
  <div class="card">
    <div class="box rotateY">
      <div class="fill"></div>
    </div>
    <p>rotateY(45deg)</p>
  </div>
</div>
<h4>Perspective : 200</h4>
<div class="perspective-200">
  <div class="card">
    <div class="box rotateX">
      <div class="fill"></div>
    </div>
    <p>rotateX(90deg)</p>
  </div>
  <div class="card">
    <div class="box rotateY">
      <div class="fill"></div>
    </div>
    <p>rotateY(45deg)</p>
  </div>
</div>
<!-- transform origin-->
<h2>Transform origin</h2>
<div class="card">
  <div class="box rotate">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100% 0 0  <br/>rotate(45deg)</p>
</div>
<div class="card">
  <div class="box rotate">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0<br/>rotate(45deg)</p>
</div>
<div class="card perspective-200">
  <div class="box rotateX">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0<br/>rotateX(45deg)</p>
</div>
<div class="card perspective-200">
  <div class="box rotateX">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100% 0 0<br/>rotateX(45deg)</p>
</div>
<div class="card perspective-200">
  <div class="box rotateY">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0 <br/>rotateY(45deg)</p>
</div>
<div class="card perspective-200">
  <div class="box rotateY">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100%  0 0<br/>rotateY(45deg)</p>
</div>
<div class="card">
  <div class="box scale">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100%  0 0<br/>scale(2)</p>
</div>
<div class="card">
  <div class="box scale">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0<br/>scale(2)</p>
</div>
<div class="card">
  <div class="box scaleX">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100%  0 0<br/>scaleX(2)</p>
</div>
<div class="card">
  <div class="box scaleX">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0<br/>scaleX(2)</p>
</div>
<div class="card">
  <div class="box scaleY">
    <div class="fill to-100-0-0"></div>
  </div>
  <p>transform-origin : 100%  0 0<br/>scaleY(2)</p>
</div>
<div class="card">
  <div class="box scaleY">
    <div class="fill to-0-100-0"></div>
  </div>
  <p>transform-origin : 0 100%  0<br/>scaleY(2)</p>
</div>
```

CSS：

```css
*, *:after, *:before {
  box-sizing: border-box;
}

body {
  background: #F5F3F4;
  margin: 0;
  padding: 10px;
  font-family: 'Open Sans', sans-serif;
  text-align: center;
}

h1 {
  color: #4c4c4c;
  font-weight: 600;
  border-bottom: 1px solid #ccc;
}

h2, h4 {
  font-weight: 400;
  color: #4d4d4d;
}

.card {
  display: inline-block;
  margin: 10px;
  background: #fff;
  padding: 15px;
  min-width: 200px;
  box-shadow: 0 3px 5px #ddd;
  color: #555;
}
.card .box {
  width: 100px;
  height: 100px;
  margin: auto;
  background: #ddd;
  cursor: pointer;
  box-shadow: 0 0 5px #ccc inset;
}
.card .box .fill {
  width: 100px;
  height: 100px;
  position: relative;
  background: #03A9F4;
  opacity: .5;
  box-shadow: 0 0 5px #ccc;
  -webkit-transition: 0.3s;
  transition: 0.3s;
}
.card p {
  margin: 25px 0 0;
}

.rotate:hover .fill {
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
}

.rotateX:hover .fill {
  -webkit-transform: rotateX(45deg);
  transform: rotateX(45deg);
}

.rotateY:hover .fill {
  -webkit-transform: rotateY(45deg);
  transform: rotateY(45deg);
}

.rotateZ:hover .fill {
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
}

.scale:hover .fill {
  -webkit-transform: scale(2, 2);
  transform: scale(2, 2);
}

.scaleX:hover .fill {
  -webkit-transform: scaleX(2);
  transform: scaleX(2);
}

.scaleY:hover .fill {
  -webkit-transform: scaleY(2);
  transform: scaleY(2);
}

.skew:hover .fill {
  -webkit-transform: skew(45deg, 45deg);
  transform: skew(45deg, 45deg);
}

.skewX:hover .fill {
  -webkit-transform: skewX(45deg);
  transform: skewX(45deg);
}

.skewY:hover .fill {
  -webkit-transform: skewY(45deg);
  transform: skewY(45deg);
}

.translate:hover .fill {
  -webkit-transform: translate(45px, 1em);
  transform: translate(45px, 1em);
}

.translateX:hover .fill {
  -webkit-transform: translateX(45px);
  transform: translateX(45px);
}

.translateY:hover .fill {
  -webkit-transform: translateY(45px);
  transform: translateY(45px);
}

.matrix:hover .fill {
  -webkit-transform: matrix(2, 2, 0, 2, 45, 0);
  transform: matrix(2, 2, 0, 2, 45, 0);
}

.perspective-100 .box {
  -webkit-perspective: 100px;
  perspective: 100px;
}

.perspective-200 .box {
  -webkit-perspective: 200px;
  perspective: 200px;
}

.to-100-0-0 {
  -webkit-transform-origin: 100% 0 0;
          transform-origin: 100% 0 0;
}

.to-0-100-0 {
  -webkit-transform-origin: 0 100% 0;
          transform-origin: 0 100% 0;
}
```



#### transform-origin

transform-Origin属性允许您更改转换元素的位置。 2D转换元素可以改变元素的X和Y轴。 3D转换元素，还可以更改元素的Z轴。 

默认值：`50% 50% 0`

语法：

```
 transform-origin: x-axis y-axis z-axis;
```

| 值     | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| x-axis | 定义视图被置于 X 轴的何处。可能的值： 	 	left 	center 	right 	*length* 	*%* |
| y-axis | 定义视图被置于 Y 轴的何处。可能的值： 	 	top 	center 	bottom 	*length* 	*%* |
| z-axis | 定义视图被置于 Z 轴的何处。可能的值： 	 	*length*      |

```
<!DOCTYPE html>
<html>
<head>
<style>
#div1
{
position: relative;
height: 200px;
width: 200px;
margin: 50px;
padding:10px;
border: 1px solid black;
}

#div2
{
padding:50px;
position: absolute;
border: 1px solid black;
background-color: red;
transform: rotate(45deg);
transform-origin:20% 40%;
-ms-transform: rotate(45deg); /* IE 9 */
-ms-transform-origin:20% 40%; /* IE 9 */
-webkit-transform: rotate(45deg); /* Safari and Chrome */
-webkit-transform-origin:20% 40%; /* Safari and Chrome */
-moz-transform: rotate(45deg); /* Firefox */
-moz-transform-origin:20% 40%; /* Firefox */
-o-transform: rotate(45deg); /* Opera */
-o-transform-origin:20% 40%; /* Opera */
}
</style>
<script>
function changeRot(value)
{
document.getElementById('div2').style.transform="rotate(" + value + "deg)";
document.getElementById('div2').style.msTransform="rotate(" + value + "deg)";
document.getElementById('div2').style.webkitTransform="rotate(" + value + "deg)";
document.getElementById('div2').style.MozTransform="rotate(" + value + "deg)";
document.getElementById('div2').style.OTransform="rotate(" + value + "deg)";
document.getElementById('persp').innerHTML=value + "deg";
}

function changeOrg()
{
var x=document.getElementById('ox').value;
var y=document.getElementById('oy').value;
document.getElementById('div2').style.transformOrigin=x + '% ' + y + '%';
document.getElementById('div2').style.msTransformOrigin=x + '% ' + y + '%';
document.getElementById('div2').style.webkitTransformOrigin=x + '% ' + y + '%';
document.getElementById('div2').style.MozTransformOrigin=x + '% ' + y + '%';
document.getElementById('div2').style.OTransformOrigin=x + '% ' + y + '%';
document.getElementById('origin').innerHTML=x + "% " + y + "%";            
}
</script>
</head>

<body>

<p>Rotate the red div element, try changing its X-axis and Y-axis:</p>

<div id="div1">
  <div id="div2">HELLO</div>
</div>

Rotate:
<input type="range" min="-360" max="360" value="45" onchange="changeRot(this.value)" />
transform: rotateY:(<span id="persp">45deg</span>);
<br><br>
X-axis:<input type="range" min="-100" max="200" value="20" onchange="changeOrg()" id="ox" /><br>
Y-axis:<input type="range" min="-100" max="200" value="40" onchange="changeOrg()" id="oy" />
transform-origin: <span id="origin">20% 40%</span>;
 
</body>
</html>
```

#### 边框

**border-radius**

border-radius 属性是一个简写属性，用于设置四个 `border-*-radius` 属性。

默认值： 	0
继承性： 	no
JavaScript 语法： 	`object.style.borderRadius="5px"`

语法：

```
border-radius: 1-4 length|% / 1-4 length|%;
```

按此顺序设置每个 radii 的四个值。如果省略 bottom-left，则与 top-right 相同。如果省略 bottom-right，则与 top-left 相同。如果省略 top-right，则与 top-left 相同。

```html
<!DOCTYPE html>
<html>
<head>
<style> 
div
{
text-align:center;
border:2px solid #a1a1a1;
padding:10px 40px; 
background:#dddddd;
width:350px;
border-radius:25px;
-moz-border-radius:25px; /* 老的 Firefox */
}
</style>
</head>
<body>

<div>border-radius 属性允许您向元素添加圆角。</div>

</body>
</html>
```

**box-shadow**

box-shadow 用于向方框添加阴影

语法：

```
box-shadow: h-shadow v-shadow blur spread color inset;
```

box-shadow 向框添加一个或多个阴影。该属性是由逗号分隔的阴影列表，每个阴影由 2-4 个长度值、可选的颜色值以及可选的 inset 关键词来规定。省略长度的值是 0。

| 值         | 描述                                     |
| ---------- | ---------------------------------------- |
| *h-shadow* | 必需。水平阴影的位置。允许负值。         |
| *v-shadow* | 必需。垂直阴影的位置。允许负值。         |
| *blur*     | 可选。模糊距离。                         |
| *spread*   | 可选。阴影的尺寸。                       |
| *color*    | 可选。阴影的颜色。请参阅 CSS 颜色值。    |
| inset      | 可选。将外部阴影 (outset) 改为内部阴影。 |

```html
<!DOCTYPE html>
<html>
<head>
<style> 
div
{
width:300px;
height:100px;
background-color:#ff9900;
-moz-box-shadow: 10px 10px 5px #888888; /* 老的 Firefox */
box-shadow: 10px 10px 5px #888888;
}
</style>
</head>
<body>

<div></div>

</body>
</html>
```

**border-image**

border-image 属性是一个简写属性，用于设置以下属性：

- border-image-source
- border-image-slice
- border-image-width
- border-image-outset
- border-image-repeat

默认值： 	`none 100% 1 0 stretch`
继承性： 	no
JavaScript 语法： 	`object.style.borderImage="url(border.png) 30 30 round"`

| 值                    | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| *border-image-source* | 用在边框的图片的路径。                                       |
| *border-image-slice*  | 图片边框向内偏移。                                           |
| *border-image-width*  | 图片边框的宽度。                                             |
| *border-image-outset* | 边框图像区域超出边框的量。                                   |
| *border-image-repeat* | 图像边框是否应平铺(repeated)、铺满(rounded)或拉伸(stretched)。 |

####  justify-content 

justify-content 用于设置或检索弹性盒子元素在主轴（横轴）方向上的对齐方式。

**提示：**使用 align-content 属性对齐交叉轴上的各项（垂直）。

| 默认值：          | flex-start                                                   |
| ----------------- | ------------------------------------------------------------ |
| 继承：            | 否                                                           |
| 可动画化：        | 否。请参阅 [*可动画化（animatable）*](https://www.runoob.com/cssref/css-animatable.html)。 |
| 版本：            | CSS3                                                         |
| JavaScript 语法： | *object*.style.justifyContent="space-between"                |

语法：

```
 justify-content: flex-start|flex-end|center|space-between|space-around|initial|inherit;
```

| 值            | 描述                                                         | 测试                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| flex-start    | 默认值。项目位于容器的开头。                                 | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=flex-start) |
| flex-end      | 项目位于容器的结尾。                                         | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=flex-end) |
| center        | 项目位于容器的中心。                                         | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=center) |
| space-between | 项目位于各行之间留有空白的容器内。                           | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=space-between) |
| space-around  | 项目位于各行之前、之间、之后都留有空白的容器内。             | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=space-around) |
| initial       | 设置该属性为它的默认值。请参阅 [*initial*](https://www.runoob.com/cssref/css-initial.html)。 | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=initial) |
| inherit       | 从父元素继承该属性。请参阅 [*inherit*](https://www.runoob.com/cssref/css-inherit.html)。 |                                                              |

### align

#### direction

direction 属性规定文本的方向 / 书写方向。

该属性指定了块的基本书写方向，以及针对 Unicode 双向算法的嵌入和覆盖方向。不支持双向文本的用户代理可以忽略这个属性。

默认值： 	ltr
继承性： 	yes
JavaScript 语法： 	`object.style.direction="rtl"`

| 值      | 描述                                      |
| ------- | ----------------------------------------- |
| ltr     | 默认。文本方向从左到右。                  |
| rtl     | 文本方向从右到左。                        |
| inherit | 规定应该从父元素继承 direction 属性的值。 |

#### text-align

text-align 属性规定元素中的文本的水平对齐方式。

该属性通过指定行框与哪个点对齐，从而设置块级元素内文本的水平对齐方式。通过允许用户代理调整行内容中字母和字之间的间隔，可以支持值 justify；不同用户代理可能会得到不同的结果。

默认值： 	如果 direction 属性是 ltr，则默认值是 left；如果 direction 是 rtl，则为 right。
继承性： 	yes
JavaScript 语法： 	`object.style.textAlign="right"`

| 值      | 描述                                       |
| ------- | ------------------------------------------ |
| left    | 把文本排列到左边。默认值：由浏览器决定。   |
| right   | 把文本排列到右边。                         |
| center  | 把文本排列到中间。                         |
| justify | 实现两端对齐文本效果。                     |
| inherit | 规定应该从父元素继承 text-align 属性的值。 |

**值 justify**

值 justify 可以使文本的两端都对齐。在两端对齐文本中，文本行的左右两端都放在父元素的内边界上。然后，调整单词和字母间的间隔，使各行的长度恰好相等。您也许已经注意到了，两端对齐文本在打印领域很常见。不过在 CSS 中，还需要多做些考虑。

#### vertical-align

vertical-align 属性设置元素的垂直对齐方式。

该属性定义行内元素的基线相对于该元素所在行的基线的垂直对齐。允许指定负长度值和百分比值。这会使元素降低而不是升高。在表单元格中，这个属性会设置单元格框中的单元格内容的对齐方式。

```
vertical-align：baseline | sub | super | top | text-top | middle | bottom | text-bottom | <percentage> | <length>
```



| 值          | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| baseline    | 默认。元素放置在父元素的基线上。                             |
| sub         | 垂直对齐文本的下标。                                         |
| super       | 垂直对齐文本的上标                                           |
| top         | 把元素的顶端与行中最高元素的顶端对齐                         |
| text-top    | 把元素的顶端与父元素字体的顶端对齐                           |
| middle      | 把此元素放置在父元素的中部。                                 |
| bottom      | 把元素的顶端与行中最低的元素的顶端对齐。                     |
| text-bottom | 把元素的底端与父元素字体的底端对齐。                         |
| length      |                                                              |
| %           | 使用 "line-height" 属性的百分比值来排列此元素。允许使用负值。 |
| inherit     | 规定应该从父元素继承 vertical-align 属性的值。               |



#### align-items

align-items 属性定义flex子项在flex容器的当前行的侧轴（纵轴）方向上的对齐方式。

**提示：**使用每个弹性对象元素的 align-self 属性可重写 align-items 属性。

```
 align-items: stretch|center|flex-start|flex-end|baseline|initial|inherit;
```



| 值         | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| stretch    | 默认值。元素被拉伸以适应容器。 如果指定侧轴大小的属性值为'auto'，则其值会使项目的边距盒的尺寸尽可能接近所在行的尺寸，但同时会遵照'min/max-width/height'属性的限制。 |
| center     | 元素位于容器的中心。 弹性盒子元素在该行的侧轴（纵轴）上居中放置。（如果该行的尺寸小于弹性盒子元素的尺寸，则会向两个方向溢出相同的长度）。 |
| flex-start | 元素位于容器的开头。   弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。 |
| flex-end   | 元素位于容器的结尾。 弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界。 |
| baseline   | 元素位于容器的基线上。 如弹性盒子元素的行内轴与侧轴为同一条，则该值与'flex-start'等效。其它情况下，该值将参与基线对齐。 |
| initial    | 设置该属性为它的默认值。请参阅 [*initial*](https://www.runoob.com/cssref/css-initial.html)。 |
| inherit    | 从父元素继承该属性。请参阅 [*inherit*](https://www.runoob.com/cssref/css-inherit.html)。 |

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>“居中对齐弹性盒的各项 <div> 元素”</title>
<style>
#main {
    width: 220px;
    height: 300px;
    border: 1px solid black;
    display: -webkit-flex; /* Safari */
    -webkit-align-items: center; /* Safari 7.0+ */
    display: flex;
    align-items: center;
}

#main div {
   -webkit-flex: 1; /* Safari 6.1+ */
   flex: 1;
}
</style>
</head>
<body>

<div id="main">
  <div style="background-color:coral;">RED</div>
  <div style="background-color:lightblue;">BLUE</div>
  <div style="background-color:lightgreen;">Green div with more content.</div>
</div>

<p><b>注意:</b> Internet Explorer 10 及更早版本浏览器不支持 align-items 属性。</p>
<p><b>注意:</b> Safari 7.0 及更新版本通过 -webkit-align-items 属性支持该属性。</p>

</body>
</html>
```

#### align-content

align-content 属性在弹性容器内的各项没有占用交叉轴上所有可用的空间时对齐容器内的各项（垂直）。

**提示：**使用 justify-content 属性对齐主轴上的各项（水平）。

**注意：**容器内必须有多行的项目，该属性才能渲染出效果。

```
 align-content: stretch|center|flex-start|flex-end|space-between|space-around|initial|inherit;
```

| 值            | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| stretch       | 默认值。元素被拉伸以适应容器。 各行将会伸展以占用剩余的空间。如果剩余的空间是负数，该值等效于'flex-start'。在其它情况下，剩余空间被所有行平分，以扩大它们的侧轴尺寸。 |
| center        | 元素位于容器的中心。  各行向弹性盒容器的中间位置堆叠。各行两两紧靠住同时在弹性盒容器中居中对齐，保持弹性盒容器的侧轴起始内容边界和第一行之间的距离与该容器的侧轴结束内容边界与第最后一行之间的距离相等。（如果剩下的空间是负数，则各行会向两个方向溢出的相等距离。） |
| flex-start    | 元素位于容器的开头。 各行向弹性盒容器的起始位置堆叠。弹性盒容器中第一行的侧轴起始边界紧靠住该弹性盒容器的侧轴起始边界，之后的每一行都紧靠住前面一行。 |
| flex-end      | 元素位于容器的结尾。 各行向弹性盒容器的结束位置堆叠。弹性盒容器中最后一行的侧轴起结束界紧靠住该弹性盒容器的侧轴结束边界，之后的每一行都紧靠住前面一行。 |
| space-between | 元素位于各行之间留有空白的容器内。 各行在弹性盒容器中平均分布。如果剩余的空间是负数或弹性盒容器中只有一行，该值等效于'flex-start'。在其它情况下，第一行的侧轴起始边界紧靠住弹性盒容器的侧轴起始内容边界，最后一行的侧轴结束边界紧靠住弹性盒容器的侧轴结束内容边界，剩余的行则按一定方式在弹性盒窗口中排列，以保持两两之间的空间相等。 |
| space-around  | 元素位于各行之前、之间、之后都留有空白的容器内。 各行在弹性盒容器中平均分布，两端保留子元素与子元素之间间距大小的一半。如果剩余的空间是负数或弹性盒容器中只有一行，该值等效于'center'。在其它情况下，各行会按一定方式在弹性盒容器中排列，以保持两两之间的空间相等，同时第一行前面及最后一行后面的空间是其他空间的一半。 |
| initial       | 设置该属性为它的默认值。请参阅 [*initial*](https://www.runoob.com/cssref/css-initial.html)。 |
| inherit       | 从父元素继承该属性。请参阅 [*inherit*](https://www.runoob.com/cssref/css-inherit.html)。 |

示例：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>“对齐弹性盒的 <div> 元素的各项”</title>
<style>
#main {
    width: 70px;
    height: 300px;
    border: 1px solid #c3c3c3;
    display: -webkit-flex;
    display: flex;
    -webkit-flex-wrap: wrap;
    flex-wrap: wrap;
    -webkit-align-content: center;
    align-content: center;
}

#main div {
    width: 70px;
    height: 70px;
}
</style>
</head>
<body>

<div id="main">
  <div style="background-color:coral;"></div>
  <div style="background-color:lightblue;"></div>
  <div style="background-color:pink;"></div>
</div>

<p><b>注意:</b> Internet Explorer 10 及更早版本浏览器不支持 align-content 属性。</p>
<p><b>注意:</b> Safari 7.0 及更新版本通过 -webkit-align-content 属性支持该属性。</p>

</body>
</html>
```

###  display

display 属性规定元素应该生成的框的类型。

| 值                   | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `none`               | `此元素不会被显示。`                                         |
| `block`              | `此元素将显示为块级元素，此元素前后会带有换行符。`           |
| `inline`             | `默认。此元素会被显示为内联元素，元素前后没有换行符。`       |
| `inline-block`       | `行内块元素。（CSS2.1 新增的值）`                            |
| `list-item`          | `此元素会作为列表显示。`                                     |
| `run-in`             | `此元素会根据上下文作为块级元素或内联元素显示。`             |
| `compact`            | `CSS 中有值 compact，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。` |
| `marker`             | `CSS 中有值 marker，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。` |
| `table`              | `此元素会作为块级表格来显示（类似 <table>），表格前后带有换行符。` |
| `inline-table`       | `此元素会作为内联表格来显示（类似 <table>），表格前后没有换行符。` |
| `table-row-group`    | `此元素会作为一个或多个行的分组来显示（类似 <tbody>）。`     |
| `table-header-group` | `此元素会作为一个或多个行的分组来显示（类似 <thead>）。`     |
| `table-footer-group` | `此元素会作为一个或多个行的分组来显示（类似 <tfoot>）。`     |
| `table-row`          | `此元素会作为一个表格行显示（类似 <tr>）。`                  |
| `table-column-group` | `此元素会作为一个或多个列的分组来显示（类似 <colgroup>）。`  |
| `table-column`       | `此元素会作为一个单元格列显示（类似 <col>）`                 |
| `table-cell`         | `此元素会作为一个表格单元格显示（类似 <td> 和 <th>）`        |
| `table-caption`      | `此元素会作为一个表格标题显示（类似 <caption>）`             |
| `inherit`            | `规定应该从父元素继承 display 属性的值。`                    |

#### display:flex 布局

display:flex 是一种布局方式。它即可以应用于容器中，也可以应用于行内元素。是W3C提出的一种新的方案，可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持。

Flex是Flexible Box的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。设为Flex布局以后，子元素的float、clear和vertical-align属性将失效。

采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

![3.png](https://i.loli.net/2019/05/17/5cde0f9b69ad264362.png)

以下6个属性设置在容器上： 

- flex-direction　　容器内项目的排列方向(默认横向排列)　　
- flex-wrap　　容器内项目换行方式
- flex-flow　　以上两个属性的简写方式
- justify-content　　项目在主轴上的对齐方式
- align-items　　项目在交叉轴上如何对齐
- align-content　　定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。





容器中项目的属性：

- `order　　项目的排列顺序。数值越小，排列越靠前，默认为0。`
- `flex-grow　　项目的放大比例，默认为0，即如果存在剩余空间，也不放大。`
- `flex-shrink　　项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。`
- `flex-basis　　在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。`
- `flex　　是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。`
- `align-self　　允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。`

