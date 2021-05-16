---
title: CSS 入深坑
date: 2019-04-25 12:18:59
tags:
 - 前端
 - CSS
categories:
 - 前端
 - CSS
---

### 样式

#### 颜色

HTML 颜色由红色、绿色、蓝色混合而成。

HTML 颜色由一个十六进制符号来定义，这个符号由红色、绿色和蓝色的值组成（RGB）。 

每种颜色的最小值是0（十六进制：#00）。最大值是255（十六进制：#FF）。

<!--more-->

三位数表示法为：#RGB，转换为6位数表示为：#RRGGBB。

RGBA 的意思是（Red-Green-Blue-Alpha）它是在 RGB 上扩展包括了 **“alpha”** 通道，运行对颜色值设置透明度。

相对于使用 rgb(255,255,0)，使用 rgba(255,255,0,0.5) 可以实现设置颜色透明度的功能，这里的 0.5 表示透明度，范围 0~1，0 表示全透明。

background-color 只设定了背景颜色,而background附带了no repeat属性,并且background还能设置图片，背景，定位等。 color:为不同元素设置文本颜色。

通常我们为了省略一个 0.。

141个颜色名称是在HTML和CSS颜色规范定义的（17标准颜色，再加124）。具体参考[这里](https://www.runoob.com/html/html-colornames.html)。

设置方式：

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style>
		body{
			
			display: flex;
			flex-wrap: wrap;
			
		}
 
		.div1{
 
			/* 1.HEX */
 
			background-color: #ccc;
			/* background-color: gray; */
 
			
		}
		.div2{
			/* 2.RGB */
			/* R：
			红色值。正整数 | 百分数
			G：
			绿色值。正整数 | 百分数
			B：
			蓝色值。正整数 | 百分数 */
			background-color: rgb(128, 128, 128);
 
		}
 
		.div3{
			/* 3.HSL */
 
			/* H：
			Hue(色调)。0(或360)表示红色，120表示绿色，240表示蓝色，也可取其他数值来指定颜色。取值为：0 - 360
			S：
			Saturation(饱和度)。取值为：0.0% - 100.0%
			L：
			Lightness(亮度)。取值为：0.0% - 100.0% */
			background-color: hsl(330, 20%, 50%);
		}
		.div4{
			/* 4.RGBA */
		/* 	A：
			Alpha透明度。取值0~1之间。 */
			
 
			background-color: rgb(128, 128, 128,0.5);
		}
		.div5{
			/* 5.HSLA */
			
			background-color: hsl(330, 20%, 50%,0.5) ;
 
		}
 
		.div6{
			/* 6.transparent  用来指定全透明色彩。*/			
			/* color: transparent; */
			border: 1px solid transparent;
			background: transparent;
			/* opacity: 0; */
			
		}
 
		.div7{
			/* 7.currentColor*/
			border: 1px solid;
			color: red;						
		}
	</style>
</head>
<body>
 
	<div class="div1" style="width: 200px;height: 200px">div1   HEX   #ccc</div>
	<div class="div2" style="width: 200px;height: 200px">div2   RGB</div>
	<div class="div3" style="width: 200px;height: 200px">div3   HSL</div>
	<div class="div4" style="width: 200px;height: 200px">div4   RGBA  透明度</div>
	<div class="div5" style="width: 200px;height: 200px">div5   HSLA  透明度</div>
	<div class="div6" style="width: 200px;height: 200px">div6   transparent 背景色全透明</div>
	<div class="div7" style="width: 200px;height: 200px">div7   currentColor</div>
	
</body>
</html>
```

#### 单位

1. px：绝对单位，页面按精确像素展示
2. em：相对单位，基准点为父节点字体的大小，如果自身定义了font-size按自身来计算（浏览器默认字体是16px），整个页面内1em不是一个固定的值。
3. rem：相对单位，可理解为”root em”, 相对根节点html的字体大小来计算，CSS3新加属性，chrome/firefox/IE9+支持。
4. vw：viewpoint width，视窗宽度，1vw等于视窗宽度的1%。
5. vh：viewpoint height，视窗高度，1vh等于视窗高度的1%。
6. vmin：vw和vh中较小的那个。
7. vmax：vw和vh中较大的那个。
8. %:百分比
9. in:寸
10. cm:厘米
11. mm:毫米
12. pt:point，大约1/72寸
13. pc:pica，大约6pt，1/6寸
14. ex：取当前作用效果的字体的x的高度，在无法确定x高度的情况下以0.5em计算(IE11及以下均不支持，firefox/chrome/safari/opera/ios safari/android browser4.4+等均需属性加么有前缀)
15. ch:以节点所使用字体中的“0”字符为基准，找不到时为0.5em(ie10+,chrome31+,safair7.1+,opera26+,ios safari 7.1+,android browser4.4+支持)

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			/*
			 * 长度单位
			 * 		像素 px
			 * 			- 像素是我们在网页中使用的最多的一个单位，
			 * 				一个像素就相当于我们屏幕中的一个小点，
			 * 				我们的屏幕实际上就是由这些像素点构成的
			 * 				但是这些像素点，是不能直接看见。
			 * 			- 不同显示器一个像素的大小也不相同，
			 * 				显示效果越好越清晰，像素就越小，反之像素越大。
			 * 
			 * 		百分比 %
			 * 			- 也可以将单位设置为一个百分比的形式，
			 * 				这样浏览器将会根据其父元素的样式来计算该值
			 * 			- 使用百分比的好处是，当父元素的属性值发生变化时，
			 * 				子元素也会按照比例发生改变
			 * 			- 在我们创建一个自适应的页面时，经常使用百分比作为单位
			 * 
			 * 		em
			 * 			- em和百分比类似，它是相对于当前元素的字体大小来计算的
			 * 			- 1em = 1font-size
			 * 			- 使用em时，当字体大小发生改变时，em也会随之改变
			 * 			- 当设置字体相关的样式时，经常会使用em
			 
			 * 			height 	设置元素高度。 	
                        max-height 	设置元素的最大高度。 	
                        max-width 	设置元素的最大宽度。 	
                        min-height 	设置元素的最小高度。 	
                        min-width 	设置元素的最小宽度。 	
                        width 	设置元素的宽度。
			 */
			.box{
				width: 300px;
				height: 300px;
				background-color: red;
			}
			
			.box1{
				font-size: 20px;
				width: 2em;
				height: 50%;
				background-color: yellow;
			}
			
		</style>
	</head>
	<body>
		
		<div class="box">
			
			<div class="box1"></div>
			
		</div>
		
	</body>
</html>
```

#### 字体

[字体库](https://www.webfont.com/)

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			
			.p1{
				/*设置字体颜色,使用color来设置文字的颜色*/
				color: red;
				/*
				 * 设置文字的大小,浏览器中一般默认的文字大小都是16px
				 	font-size设置的并不是文字本身的大小，
				 		在页面中，每个文字都是处在一个看不见的框中的
				 		我们设置的font-size实际上是设置格的高度，并不是字体的大小
				 		一般情况下文字都要比这个格要小一些，也有时会比格大，
				 		根据字体的不同，显示效果也不能	
				 	
				 * */
				font-size: 30px;
				
				/*
				 * 通过font-family可以指定文字的字体
				 * 	当采用某种字体时，如果浏览器支持则使用该字体，
				 * 		如果字体不支持，则使用默认字体
				 * 该样式可以同时指定多个字体，多个字体之间使用,分开
				 * 	当采用多个字体时，浏览器会优先使用前边的字体，
				 * 		如果前边没有在尝试下一个
				 */
				/*font-family: arial , 微软雅黑;*/
				
				/*
				 * 浏览器使用的字体默认就是计算机中的字体，
				 * 	如果计算机中有，则使用，如果没有就不用
				 * 
				 * 在开发中，如果字体太奇怪，用的太少了，尽量不要使用，
				 * 	有可能用户的电脑没有，就不能达到想要的效果。
				 */
				/*font-family: 华文彩云 , arial , 微软雅黑;*/
				
				font-family: "curlz mt";
				
			}
						.p1{
				color: red;
				font-size: 30px;
				font-family: "微软雅黑";
				/*
				 * font-style可以用来设置文字的斜体
				 * 	- 可选值：
				 * 		normal，默认值，文字正常显示
				 * 		italic 文字会以斜体显示
				 * 		oblique 文字会以倾斜的效果显示
				 *		inherit 规定应该从父元素继承字体样式。
				 * 	- 大部分浏览器都不会对倾斜和斜体做区分，
				 * 		也就是说我们设置italic和oblique它们的效果往往是一样的
				 *  - 一般我们只会使用italic
				 */
				font-style: italic;
				
				/*
				 * font-weight可以用来设置文本的加粗效果：
				 * 		可选值：
				 * 			normal，默认值，文字正常显示
				 * 			bold，文字加粗显示
				 * 			bolder 	定义更粗的字符。
				 *			lighter 	定义更细的字符。
				 *         inherit 	规定应该从父元素继承字体的粗细。
				 * 	该样式也可以指定100-900之间的9个值，
				 * 		但是由于用户的计算机往往没有这么多级别的字体，所以达到我们想要的效果
				 * 		也就是200有可能比100粗，300有可能比200粗，但是也可能是一样的
				 *		400 等同于 normal，而 700 等同于 bold。
				 */
				font-weight: bold;
				
				/*
				 * font-variant可以用来设置小型大写字母
				 *font-variant 属性设置小型大写字母的字体显示文本，
				 *这意味着所有的小写字母均会被转换为大写，
				 *但是所有使用小型大写字体的字母与其余文本相比，其字体尺寸更小。
				 * 		可选值：
				 * 			normal，默认值，文字正常显示
				 * 			small-caps 文本以小型大写字母显示
				 * 			inherit 规定应该从父元素继承。
				 * 小型大写字母：
				 * 		将所有的字母都以大写形式显示，但是小写字母的大写，
				 * 			要比大写字母的大小小一些。
				 */
				font-variant: small-caps ;
				

			}
			
			.p2{
				/*设置一个文字大小*/
				font-size: 50px;
				/*设置一个字体*/
				font-family: 华文彩云;
				/*设置文字斜体*/
				font-style: italic;
				/*设置文字的加粗*/
				font-weight: bold;
				/*设置一个小型大写字母*/
				font-variant: small-caps;
			}
			
			.p3{
				/*
				 * 在CSS中还为我们提供了一个样式叫font，
				 * 	使用该样式可以同时设置字体相关的所有样式,
				 * 	可以将字体的样式的值，统一写在font样式中，不同的值之间使用空格隔开
				 * 使用font设置字体样式时，斜体 加粗 小大字母，没有顺序要求，甚至可写可不写，
				 * 	如果不写则使用默认值，但是要求文字的大小和字体必须写，
				 *而且字体必须是最后一个样式
				 * 	大小必须是倒数第二个样式
				 * 可以按顺序设置如下属性：
                    font-style
                    font-variant
                    font-weight
                    font-size/line-height
                    font-family
				 * 实际上使用简写属性也会有一个比较好的性能
				 */
				font: small-caps bold italic 60px "微软雅黑";
			}
			
			
		</style>
	</head>
	<body>
		
		<p class="p1">
			我是一个p标签，ABCDEFGabcdefg
		</p>
			<p class="p3">我是一段文字，ABCDEFGabcdefg</p>
		
		<p class="p1">我是一段文字，ABCDEFGabcdefg</p>
		
		<p class="p2">我是一段文字，ABCDEFGabcdefg</p>
			<!-- 
			在网页中将字体分成5大类：
				serif（衬线字体）
				sans-serif（非衬线字体）
				monospace （等宽字体）
				cursive （草书字体）
				fantasy （虚幻字体）
			可以将字体设置为这些大的分类,当设置为大的分类以后，
				浏览器会自动选择指定的字体并应用样式
			一般会将字体的大分类，指定为font-family中的最后一个字体	
		-->
		<p style="font-size: 50px; font-family: serif;">衬线字体：我是一段文字，ABCDEFGabcdefg</p>
		<p style="font-size: 50px; font-family: sans-serif;">非衬线字体：我是一段文字，ABCDEFGabcdefg</p>
		<p style="font-size: 50px; font-family: monospace;">等宽字体：我是一段文字，IHABCDEFGabcdefg</p>
		<p style="font-size: 50px; font-family: cursive;">草书字体：我是一段文字，ABCDEFGabcdefg</p>
		<p style="font-size: 50px; font-family: fantasy;">虚幻字体：我是一段文字，ABCDEFGabcdefg</p>
	</body>
</html>

```

#### 行间距

```
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			/*
			 * 在CSS并没有为我们提供一个直接设置行间距的方式，
			 * 	我们只能通过设置行高来间接的设置行间距，行高越大行间距越大
			 * 使用line-height来设置行高 
			 * 	行高类似于我们上学单线本，单线本是一行一行，线与线之间的距离就是行高，
			 * 	网页中的文字实际上也是写在一个看不见的线中的，而文字会默认在行高中垂直居中显示
			 * 
			 * 行间距 = 行高 - 字体大小
			 */
			.p1{
				font-size: 20px;
				/*
					line-height:设置以百分比计的行高
					normal 	默认。设置合理的行间距。
                    number 	设置数字，此数字会与当前的字体尺寸相乘来设置行间距。
                    length 	设置固定的行间距。
                    % 	基于当前字体尺寸的百分比行间距。
                    inherit 	规定应该从父元素继承 line-height 属性的值。
				*/
				/*
				 * 通过设置line-height可以间接的设置行高，
				 * 	可以接收的值：
				 * 		1.直接就收一个大小
				 * 		2.可以指定一个百分数，则会相对于字体去计算行高
				 * 		3.可以直接传一个数值，则行高会设置字体大小相应的倍数
				 */
				/*line-height: 200%;*/
				
				line-height: 2;
			}
			
			.box{
				width: 200px;
				height: 200px;
				background-color: #bfa;
				/*
				 * 对于单行文本来说，可以将行高设置为和父元素的高度一致，
				 * 	这样可以是单行文本在父元素中垂直居中
				 */
				line-height: 200px;
			}
			
			.p2{
				
				
				/*
				 * 在font中也可以指定行高
				 * 	在字体大小后可以添加/行高，来指定行高，该值是可选的，如果不指定则会使用默认值
				 */
				font: 30px "微软雅黑";
				line-height: 50px;
			}
			
		</style>
	</head>

	<body>
		
		<p class="p2">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>
		
		<div class="box">
			
			<a href="#">我是一个超链接</a>
			
		</div>
		

		<p class="p1">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>

	</body>

</html>
```

#### 文本样式

```
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			.p1 {
				/*
				 * text-transform可以用来设置文本的大小写
				 * 	可选值：
				 * 		none 默认值，该怎么显示就怎么显示，不做任何处理
				 * 		capitalize 单词的首字母大写，通过空格来识别单词
				 * 		uppercase 所有的字母都大写
				 * 		lowercase 所有的字母都小写
				 */
				text-transform: lowercase;
			}
			
			.p2 {
				/*
				 * text-decoration可以用来设置文本的修饰
				 * 		可选值：
				 * 			none：默认值，不添加任何修饰，正常显示
				 * 			underline 为文本添加下划线
				 * 			overline 为文本添加上划线
				 * 			line-through 为文本添加删除线
				 */
				text-decoration: line-through;
			}
			
			a {
				/*超链接会默认添加下划线，也就是超链接的text-decoration的默认值是underline
				 	如果需要去除超链接的下划线则需要将该样式设置为none
				 * */
				text-decoration: none;
			}
			
			.p3 {
				/**
				 * letter-spacing可以指定字符间距
				 */
				/*letter-spacing: 10px;*/
				
				/*
				 * word-spacing可以设置单词之间的距离
				 * 	实际上就是设置词与词之间空格的大小
				 */
				word-spacing: 120px;
			}
			
			
			.p4{
				/*
				 * text-align用于设置文本的对齐方式
				 * 	可选值：
				 * 		left 默认值，文本靠左对齐
				 * 		right ， 文本靠右对齐
				 * 		center ， 文本居中对齐
				 * 		justify ， 两端对齐
				 * 				- 通过调整文本之间的空格的大小，来达到一个两端对齐的目的
				 */
				text-align: justify ;
			}
			
			.p5{
				
				font-size: 20px;
				
				/*
				 * text-indent用来设置首行缩进
				 * 	当给它指定一个正值时，会自动向右侧缩进指定的像素
				 * 	如果为它指定一个负值，则会向左移动指定的像素,
				 * 		通过这种方式可以将一些不想显示的文字隐藏起来
				 *  这个值一般都会使用em作为单位
				 */
				text-indent: -99999px;
			}
			
		</style>
	</head>

	<body>
		
		<p class="p5">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>
		
		<h1 class="p4">我是一个h1</h1>
		
		<p class="p4">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>

		<p class="p4">
			“We should start back,” Gared urged as the woods began to grow dark around them. “The wildlings are dead.” “Do the dead frighten you?” Ser Waymar Royce asked with just the hint of a smile. Gared did not rise to the bait. He was an old man, past fifty, and he had seen the lordlings come and go. “Dead is dead,” he said. “We have no business with the dead.” “Are they dead?” Royce asked softly. “What proof have we?” “Will saw them,” Gared said. “If he says they are dead, that’s proof enough for me.” Will had known they would drag him into the quarrel sooner or later. He wished it had been later rather than sooner. “My mother told me that dead men sing no songs,” he put in. “My wet nurse said the same thing, Will,” Royce replied. “Never believe anything you hear at a woman’s tit. There are things to be learned even from the dead.” His voice echoed, too loud in the twilit forest. “We have a long ride before us,” Gared pointed out. “Eight days, maybe nine. And night is falling.” Ser Waymar Royce glanced at the sky with disinterest. “It does that every day about this time. Are you unmanned by the dark, Gared?” Will could see the tightness around Gared’s mouth, the barely suppressed anger in his eyes under the thick black hood of his cloak. Gared had spent forty years in the Night’s Watch, man and boy, and he was not accustomed to being made light of. Yet it was more than that. Under the wounded pride, Will could sense something else in the older man. You could taste it; a nervous tension that came perilous close to fear. Will shared his unease. He had been four years on the Wall. The first time he had been sent beyond, all the old stories had come rushing back, and his bowels had turned to water. He had laughed about it afterward. He was a veteran of a hundred rangings by now, and the endless dark wilderness that the southron called the haunted forest had no more terrors for him.
		</p>

		<p class="p3">
			在我的后园，可以看见墙外有两株树，一株是枣树，还有一株也是枣树。 这上面的夜的天空，奇怪而高，我生平没有见过这样奇怪而高的天空。他仿佛要离开人间而去，使人们仰面不再看见。然而现在却非常之蓝，闪闪地䀹着几十个星星的眼，冷眼。他的口角上现出微笑，似乎自以为大有深意，而将繁霜洒在我的园里的野花草上。 我不知道那些花草真叫什么名字，人们叫他们什么名字。我记得有一种开过极细小的粉红花，现在还开着，但是更极细小了，她在冷的夜气中，瑟缩地做梦，梦见春的到来，梦见秋的到来，梦见瘦的诗人将眼泪擦在她最末的花瓣上，告诉她秋虽然来，冬虽然来，而此后接着还是春，蝴蝶乱飞，蜜蜂都唱起春词来了。她于是一笑，虽然颜色冻得红惨惨地，仍然瑟缩着。 枣树，他们简直落尽了叶子。先前，还有一两个孩子来打他们，别人打剩的枣子，现在是一个也不剩了，连叶子也落尽了。他知道小粉红花的梦，秋后要有春；他也知道落叶的梦，春后还是秋。他简直落尽叶子，单剩干子，然而脱了当初满树是果实和叶子时候的弧形，欠伸得很舒服。但是，有几枝还低亚着，护定他从打枣的竿梢所得的皮伤，而最直最长的几枝，却已默默地铁似的直刺着奇怪而高的天空，使天空闪闪地鬼䀹眼；直刺着天空中圆满的月亮，使月亮窘得发白。 鬼䀹眼的天空越加非常之蓝，不安了，仿佛想离去人间，避开枣树，只将月亮剩下。然而月亮也暗暗地躲到东边去了。而一无所有的干子，却仍然默默地铁似的直刺着奇怪而高的天空，一意要制他的死命，不管他各式各样地䀹着许多蛊惑的眼睛。 哇的一声，夜游的恶鸟飞过了。 我忽而听到夜半的笑声，吃吃地，似乎不愿意惊动睡着的人，然而四围的空气都应和着笑。夜半，没有别的人，我即刻听出这声音就在我嘴里，我也即刻被这笑声所驱逐，回进自己的房。灯火的带子也即刻被我旋高了。 后窗的玻璃上丁丁地响，还有许多小飞虫乱撞。不多久，几个进来了，许是从窗纸的破孔进来的。他们一进来，又在玻璃的灯罩上撞得丁丁地响。一个从上面撞进去了，他于是遇到火，而且我以为这火是真的。两三个却休息在灯的纸罩上喘气。那罩是昨晚新换的罩，雪白的纸，折出波浪纹的叠痕，一角还画出一枝猩红色的栀子。 猩红的栀子开花时，枣树又要做小粉红花的梦，青葱地弯成弧形了……我又听到夜半的笑声；我赶紧砍断我的心绪，看那老在白纸罩上的小青虫，头大尾小，向日葵子似的，只有半粒小麦那么大，遍身的颜色苍翠得可爱，可怜。 我打一个呵欠，点起一支纸烟，喷出烟来，对着灯默默地敬奠这些苍翠精致的英雄们。 一九二四年九月十五日。
		</p>

		<p class="p3">
			“We should start back,” Gared urged as the woods began to grow dark around them. “The wildlings are dead.” “Do the dead frighten you?” Ser Waymar Royce asked with just the hint of a smile. Gared did not rise to the bait. He was an old man, past fifty, and he had seen the lordlings come and go. “Dead is dead,” he said. “We have no business with the dead.” “Are they dead?” Royce asked softly. “What proof have we?” “Will saw them,” Gared said. “If he says they are dead, that’s proof enough for me.” Will had known they would drag him into the quarrel sooner or later. He wished it had been later rather than sooner. “My mother told me that dead men sing no songs,” he put in. “My wet nurse said the same thing, Will,” Royce replied. “Never believe anything you hear at a woman’s tit. There are things to be learned even from the dead.” His voice echoed, too loud in the twilit forest. “We have a long ride before us,” Gared pointed out. “Eight days, maybe nine. And night is falling.” Ser Waymar Royce glanced at the sky with disinterest. “It does that every day about this time. Are you unmanned by the dark, Gared?” Will could see the tightness around Gared’s mouth, the barely suppressed anger in his eyes under the thick black hood of his cloak. Gared had spent forty years in the Night’s Watch, man and boy, and he was not accustomed to being made light of. Yet it was more than that. Under the wounded pride, Will could sense something else in the older man. You could taste it; a nervous tension that came perilous close to fear. Will shared his unease. He had been four years on the Wall. The first time he had been sent beyond, all the old stories had come rushing back, and his bowels had turned to water. He had laughed about it afterward. He was a veteran of a hundred rangings by now, and the endless dark wilderness that the southron called the haunted forest had no more terrors for him.
		</p>

		<a href="#">我是超链接</a>

		<p class="p2">
			“We should start back,” Gared urged as the woods began to grow dark around them. “The wildlings are dead.” “Do the dead frighten you?” Ser Waymar Royce asked with just the hint of a smile. Gared did not rise to the bait. He was an old man, past fifty, and he had seen the lordlings come and go. “Dead is dead,” he said. “We have no business with the dead.” “Are they dead?” Royce asked softly. “What proof have we?” “Will saw them,” Gared said. “If he says they are dead, that’s proof enough for me.” Will had known they would drag him into the quarrel sooner or later. He wished it had been later rather than sooner. “My mother told me that dead men sing no songs,” he put in. “My wet nurse said the same thing, Will,” Royce replied. “Never believe anything you hear at a woman’s tit. There are things to be learned even from the dead.” His voice echoed, too loud in the twilit forest. “We have a long ride before us,” Gared pointed out. “Eight days, maybe nine. And night is falling.” Ser Waymar Royce glanced at the sky with disinterest. “It does that every day about this time. Are you unmanned by the dark, Gared?” Will could see the tightness around Gared’s mouth, the barely suppressed anger in his eyes under the thick black hood of his cloak. Gared had spent forty years in the Night’s Watch, man and boy, and he was not accustomed to being made light of. Yet it was more than that. Under the wounded pride, Will could sense something else in the older man. You could taste it; a nervous tension that came perilous close to fear. Will shared his unease. He had been four years on the Wall. The first time he had been sent beyond, all the old stories had come rushing back, and his bowels had turned to water. He had laughed about it afterward. He was a veteran of a hundred rangings by now, and the endless dark wilderness that the southron called the haunted forest had no more terrors for him.
		</p>

		<p class="p1">
			“We should start back,” Gared urged as the woods began to grow dark around them. “The wildlings are dead.” “Do the dead frighten you?” Ser Waymar Royce asked with just the hint of a smile. Gared did not rise to the bait. He was an old man, past fifty, and he had seen the lordlings come and go. “Dead is dead,” he said. “We have no business with the dead.” “Are they dead?” Royce asked softly. “What proof have we?” “Will saw them,” Gared said. “If he says they are dead, that’s proof enough for me.” Will had known they would drag him into the quarrel sooner or later. He wished it had been later rather than sooner. “My mother told me that dead men sing no songs,” he put in. “My wet nurse said the same thing, Will,” Royce replied. “Never believe anything you hear at a woman’s tit. There are things to be learned even from the dead.” His voice echoed, too loud in the twilit forest. “We have a long ride before us,” Gared pointed out. “Eight days, maybe nine. And night is falling.” Ser Waymar Royce glanced at the sky with disinterest. “It does that every day about this time. Are you unmanned by the dark, Gared?” Will could see the tightness around Gared’s mouth, the barely suppressed anger in his eyes under the thick black hood of his cloak. Gared had spent forty years in the Night’s Watch, man and boy, and he was not accustomed to being made light of. Yet it was more than that. Under the wounded pride, Will could sense something else in the older man. You could taste it; a nervous tension that came perilous close to fear. Will shared his unease. He had been four years on the Wall. The first time he had been sent beyond, all the old stories had come rushing back, and his bowels had turned to water. He had laughed about it afterward. He was a veteran of a hundred rangings by now, and the endless dark wilderness that the southron called the haunted forest had no more terrors for him.
		</p>
	</body>

</html>
```



