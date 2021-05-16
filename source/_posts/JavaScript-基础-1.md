---
title: JavaScript 入门和变量
date: 2019-04-30 12:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

### 概述

JavaScript 是一门编程语言，可为网站添加交互功能。（例如：游戏、动态样式，动画，以及在按下按钮或收到表单数据时做出的响应，等）

<!--more-->

JavaScript（缩写：JS）是一门完备的 动态编程语言。当应用于 HTML 文档时，可为网站提供动态交互特性。由布兰登·艾克（ Brendan Eich，Mozilla 项目、Mozilla 基金会和 Mozilla 公司的联合创始人）发明。

JavaScript 的应用场合极其广泛。简单到幻灯片、照片库、浮动布局和响应按钮点击。复杂到游戏、2D 和 3D 动画、大型数据库驱动程序，等等。

JavaScript 相当简洁，却非常灵活。开发者们基于 JavaScript 核心编写了大量实用工具，可以使 开发工作事半功倍。其中包括：

- 浏览器应用程序接口（API）—— 浏览器内置的 API 提供了丰富的功能，比如：动态创建 HTML 和设置 CSS 样式、从用户的摄像头采集处理视频流、生成3D 图像与音频样本，等等。
- 第三方 API —— 让开发者可以在自己的站点中整合其它内容提供者（Twitter、Facebook 等）提供的功能。
- 第三方框架和库 —— 用来快速构建网站和应用。

##### JavaScript和ECMAScript的关系

ECMAScript是一种由Ecma国际前身为欧洲计算机制造商协会,英文名称是European Computer ManufacturersAssociation，制定的标准。

JavaScript是由公司开发而成的，公司开发而成的一定是有一些问题，不便于其他的公司拓展和使用。所以欧洲的这个ECMA的组织，牵头制定JavaScript的标准，取名为ECMAScript。

简单来说ECMAScript不是一门语言，而是一个标准。符合这个标准的比较常见的有：JavaScript、Action Script（Flash中用的语言）。就是说，你JavaScript学完了，Flash中的程序也会写了。

ECMAScript在2015年6月，发布了ECMAScript 6版本，语言的能力更强。但是，浏览器的厂商不能那么快的去追上这个标准。

##### 介绍

js是一款运行在客户端的网页编程语言。

组成部分

-   ecmascript   js标准

-   dom        通过js操作网页元素

-   bom        通过api操作浏览器



#### Hello World

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<!--JS代码需要编写到script标签中-->
		<script type="text/javascript">
			
			/*
			 * 控制浏览器弹出一个警告框
			 * alert("哥，你真帅啊！！");
			 */
			
			/*
			 * 让计算机在页面中输出一个内容
			 * document.write()可以向body中输出一个内容
			 * document.write("看我出不出来~~~");
			 */
			
			/*
			 * 向控制台输出一个内容
			 * console.log()的作用是向控制台输出一个内容
			 * console.log("你猜我在哪出来呢？");
			 */
			
			alert("哥，你真帅啊！！");
			document.write("看我出不出来~~~");
			
			console.log("你猜我在哪出来呢？");

		</script>
	</head>
	<body>
	</body>
</html>
```

#### 使用方法

```javascript
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		
		<!--
			可以将js代码编写到外部js文件中，然后通过script标签引入
			写到外部文件中可以在不同的页面中同时引用，也可以利用到浏览器的缓存机制
			推荐使用的方式
		-->
		<!--
			script标签一旦用于引入外部文件了，就不能在编写代码了，即使编写了浏览器也会忽略
			如果需要则可以在创建一个新的script标签用于编写内部代码
		-->
		<script type="text/javascript" src="js/script.js"></script>
		<script type="text/javascript">
			alert("我是内部的JS代码");
		</script>
		
		<!--
			可以将js代码编写到script标签	
		<script type="text/javascript">
			
			alert("我是script标签中的代码！！");
			
		</script>
		-->
	</head>
	<body>
		
		<!--
			可以将js代码编写到标签的onclick属性中
			当我们点击按钮时，js代码才会执行
			
			虽然可以写在标签的属性中，但是他们属于结构与行为耦合，不方便维护，不推荐使用
		-->
		<button onclick="alert('讨厌，你点我干嘛~~');">点我一下</button>
		
		<!--
			可以将js代码写在超链接的href属性中，这样当点击超链接时，会执行js代码
		-->
		<a href="javascript:alert('让你点你就点！！');">你也点我一下</a>
		<a href="javascript:;">你也点我一下</a>
		
	</body>
</html>

```

js/script.js

```
alert("我是外部JS文件中的代码");
```

加载顺序：从上到下。

### 变量

JavaScript 借鉴了 Java 的大部分语法，但同时也受到 *Awk，Perl* 和 *Python*的影响。 

JavaScript 是**区分大小写**的，并使用 **Unicode** 字符集。举个例子，可以将单词 Früh （在德语中意思是“早”）用作变量名。

```
var Fruh = "foobar";
```

但是，由于 JavaScript 是**大小写敏感的**，因此变量 `fruh` 和 `Fruh` 则是两个不同的变量。

在 JavaScript 中，指令被称为语句  Statement，并**用分号（;）进行分隔**。

如果一条语句独占一行的话，那么分号是可以省略的。但如果一行中有多条语句，那么这些语句必须以分号分开。 ECMAScript 规定了在语句的末尾自动插入分号（ASI）。虽然不是必需的，但是在一条语句的末尾加上分号是一个很好的习惯。这个习惯可以大大减少代码中产生 bug 的可能性。

JS中会忽略多个空格和换行，所以我们可以利用空格和换行对代码进行格式化

 Javascript 源码从左往右被扫描并转换成一系列由 token 、控制字符、行终止符、注释和空白字符组成的输入元素。空白字符指的是空格、制表符和换行符等。

#### 注释

**Javascript 注释**的语法和 C++ 或许多其他语言类似：

```
// 单行注释

/* 这是一个更长的,
   多行注释
*/

/* 然而, 你不能, /* 嵌套注释 */ 语法错误 */
```

在代码执行过程中，注释将被自动跳过（不执行）。

#### 声明

JavaScript有三种声明方式。

- var：声明一个变量，可选初始化一个值。
- let：声明一个块作用域的局部变量，可选初始化一个值。
- const：声明一个块作用域的只读常量。 声明对象不能更改:`Object.freeze(obj);`。
- **Object.freeze()** 方法可以**冻结**一个对象。一个被冻结的对象再也不能被修改；冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结一个对象后该对象的原型也不能被修改。`freeze()` 返回和传入的参数相同的对象。

#### 变量

在应用程序中，使用变量来作为值的符号名。变量的名字又叫做标识符，其需要遵守一定的规则。

一个 JavaScript 标识符必须以字母、下划线（_）或者美元符号（$）开头；后续的字符也可以是数字（0-9）。因为 JavaScript 语言是区分大小写的，所以字母可以是从“A”到“Z”的大写字母和从“a”到“z”的小写字母。

命名一个标识符时需要遵守如下的规则：

1. 标识符中可以含有`字母、数字、_、$`

2. 标识符不能以数字开头

3. 标识符不能是ES中的关键字或保留字

4. 标识符一般都采用驼峰命名法
   - 首字母小写，每个单词的开头字母大写，其余字母小写：`helloWorld xxxYyyZzz`

你可以使用大部分 ISO 8859-1 或 Unicode 编码的字符作标识符。JS底层保存标识符时实际上是采用的Unicode编码，所以理论上讲，所有的utf-8中含有的内容都可以作为标识符

合法的标识符示例：`Number_hits`，`temp99`，`$credit` 和 `_name`。

你可以用以下三种方式声明变量：

- 使用关键词 `var` 。例如 `var x = 42`。这个语法可以用来声明局部变量和全局变量。
- 直接赋值。例如`x = 42`。在函数外使用这种形式赋值，会产生一个全局变量。在严格模式下会产生错误。因此你不应该使用这种方式来声明变量。
- 使用关键词 `let` 。例如 `let y = 13`。这个语法可以用来声明块作用域的局部变量。

用 `var` 或 `let` 语句声明的变量，如果没有赋初始值，则其值为 `undefined` 。

如果访问一个未声明的变量会导致抛出一个引用错误（ReferenceError）异常：

```javascript
var a;
console.log("The value of a is " + a); // a 的值是 undefined

console.log("The value of b is " + b);// b 的值是 undefined 
var b;

console.log("The value of c is " + c); // 未捕获的引用错误： c 未被定义

let x;
console.log("The value of x is " + x); // x 的值是 undefined

console.log("The value of y is " + y);// 未捕获的引用错误： y 未被定义
let y;
```

你可以使用 `undefined` 来判断一个变量是否已赋值。在以下的代码中，变量`input`未被赋值，因此 `if` 条件语句的求值结果是 `true` 。

```
var input;
if(input === undefined){
  doThis();
} else {
  doThat();
}
```

`undefined` 值在布尔类型环境中会被当作 `false` 。例如，下面的代码将会执行函数 `myFunction`，因为数组 `myArray` 中的元素未被赋值：

```js
var myArray = [];
if (!myArray[0])   myFunction();
```

数值类型环境中 `undefined` 值会被转换为 `NaN`。

```
var a;
a + 2;    // 计算为 NaN
```

当你对一个 `null` 变量求值时，空值 `null` 在数值类型环境中会被当作0来对待，而布尔类型环境中会被当作 `false`。例如：

```
var n = null;
console.log(n * 32); // 在控制台中会显示 0
```

示例：

```
<!DOCTYPE html>
<html>

<head>
	<meta charset="UTF-8">
	<title></title>
	<script type="text/javascript">
		/*
		 * 字面量，都是一些不可改变的值
		 * 		比如 ：1 2 3 4 5 
		 * 		字面量都是可以直接使用，但是我们一般都不会直接使用字面量
		 * 
		 * 变量    变量可以用来保存字面量，而且变量的值是可以任意改变的
		 * 		变量更加方便我们使用，所以在开发中都是通过变量去保存一个字面量，
		 * 		而很少直接使用字面量
		 * 		可以通过变量对字面量进行描述
		 */
		//声明变量
		//在js中使用var关键字来声明一个变量
		var a;
		//为变量赋值
		a = 123;
		a = 456;
		a = 123124223423424;
		//声明和赋值同时进行
		var b = 789;
		var c = 0;
		var age = 80;
		console.log(age);
	</script>
</head>
<body>
</body>
</html>
```

#### 变量的作用域

在函数之外声明的变量，叫做*全局*变量，因为它可被当前文档中的任何其他代码所访问。在函数内部声明的变量，叫做*局部*变量，因为它只能在当前函数的内部访问。

ECMAScript 6 之前的 JavaScript 没有 语句块 作用域；相反，语句块中声明的变量将成为语句块所在函数（或全局作用域）的局部变量。

例如，如下的代码将在控制台输出 5，因为 `x` 的作用域是声明了 `x` 的那个函数（或全局范围），而不是 `if` 语句块。

```
if (true) {
  var x = 5;
}
console.log(x); // 5
```

如果使用 ECMAScript 6 中的 `let` 声明，上述行为将发生变化。

```
if (true) {
  let y = 5;
}
console.log(y); // ReferenceError: y 没有被声明
```

#### 变量提升

JavaScript 变量的另一个不同寻常的地方是，你可以先使用变量稍后再声明变量而不会引发异常。这一概念称为变量提升；JavaScript 变量感觉上是被“提升”或移到了函数或语句的最前面。但是，提升后的变量将返回 undefined 值。因此在使用或引用某个变量之后进行声明和初始化操作，这个被提升的变量仍将返回 undefined 值。

```
/**
 * 例子1
 */
console.log(x === undefined); // true
var x = 3;


/**
 * 例子2
 */
// will return a value of undefined
var myvar = "my value";

(function() {
  console.log(myvar); // undefined
  var myvar = "local value";
})();
```

上面的例子，也可写作：

```
/**
 * 例子1
 */
var x;
console.log(x === undefined); // true
x = 3;
 
/**
 * 例子2
 */
var myvar = "my value";
 
(function() {
  var myvar;
  console.log(myvar); // undefined
  myvar = "local value";
})();
```

由于存在变量提升，一个函数中所有的`var`语句应尽可能地放在接近函数顶部的地方。这个习惯将大大提升代码的清晰度。

在 ECMAScript 6 中，let（const）将**不会提升**变量到代码块的顶部。因此，在变量声明之前引用这个变量，将抛出引用错误（ReferenceError）。这个变量将从代码块一开始的时候就处在一个“暂时性死区”，直到这个变量被声明为止。

```
console.log(x); // ReferenceError
let x = 3;
```

#### 函数提升

对于函数来说，只有函数声明会被提升到顶部，而函数表达式不会被提升。

```
/* 函数声明 */

foo(); // "bar"

function foo() {
  console.log("bar");
}


/* 函数表达式 */

baz(); // 类型错误：baz 不是一个函数

var baz = function() {
  console.log("bar2");
};
```

#### 全局变量

实际上，全局变量是*全局对象*的属性。在网页中，缺省的全局对象是 `window` ，所以你可以用形如 `window.`*variable* 的语法来设置和访问全局变量。

因此，你可以通过指定 window 或 frame 的名字，在当前 window 或 frame 访问另一个 window 或 frame 中声明的变量。例如，在文档里声明一个叫 `phoneNumber` 的变量，那么你就可以在子框架里使用 `parent.phoneNumber` 的方式来引用它。

#### 常量

你可以用关键字 `const` 创建一个只读的常量。常量标识符的命名规则和变量相同：必须以字母、下划线（_）或美元符号（$）开头并可以包含有字母、数字或下划线。

```
const PI = 3.14;
```

常量不可以通过重新赋值改变其值，也不可以在代码运行时重新声明。它必须被初始化为某个值。

常量的作用域规则与 `let` 块级作用域变量相同。若省略`const`关键字，则该标识符将被视为变量。

在同一作用域中，不能使用与变量名或函数名相同的名字来命名常量。例如：

```
// 这会造成错误
function f() {};
const f = 5;

// 这也会造成错误
function f() {
  const g = 5;
  var g;

  //语句
}
```

然而，对象属性被赋值为常量是不受保护的，所以下面的语句执行时不会产生错误。

```
const MY_OBJECT = {"key": "value"};
MY_OBJECT.key = "otherValue"
```

同样的，数组的被定义为常量也是不受保护的，所以下面的语句执行时也不会产生错误。

```
const MY_ARRAY = ['HTML','CSS'];
MY_ARRAY.push('JAVASCRIPT');
console.log(MY_ARRAY); //logs ['HTML','CSS','JAVASCRIPT'];
```

### 数据结构和类型

#### 字面量

字面量是由语法表达式定义的常量；或，通过由一定字词组成的语词表达式定义的常量

在JavaScript中，你可以使用各种字面量。这些字面量是脚本中按字面意思给出的固定的值，而不是变量。（译注：字面量是常量，其值是固定的，而且在程序脚本运行中不可更改，比如false，3.1415，thisIsStringOfHelloworld ，invokedFunction: myFunction("myArgument")。介绍以下类型的字面量：

- 数组字面量(Array literals)
- 布尔字面量(Boolean literals)
- 浮点数字面量(Floating-point literals)
- 整数(Integers)
- 对象字面量(Object literals)
- RegExp literals
- 字符串字面量(String literals)

##### 数组字面量 (Array literals)

数组字面值是一个封闭在方括号对([])中的包含有零个或多个表达式的列表，其中每个表达式代表数组的一个元素。当你使用数组字面值创建一个数组时，该数组将会以指定的值作为其元素进行初始化，而其长度被设定为元素的个数。

下面的示例用3个元素生成数组`coffees`，它的长度是3。

```
var coffees = ["French Roast", "Colombian", "Kona"];

var a=[3];

console.log(a.length); // 1

console.log(a[0]); // 3
```

#### 数据类型

最新的 ECMAScript 标准定义了7种数据类型：

- 六种基本数据类型:
  - 布尔值（Boolean），有2个值分别是：true 和 false.
  - null ， 一个表明 null 值的特殊关键字。 JavaScript 是大小写敏感的，因此 null 与 Null、NULL或变体完全不同。
  - undefined ，和 null 一样是一个特殊的关键字，undefined 表示变量未定义时的属性。
  -  数字（Number），整数或浮点数，例如： 42 或者 3.14159
  - 字符串（String），字符串是一串表示文本值的字符序列，例如："Howdy" 。
  - 代表（Symbol） ( 在 ECMAScript 6 中新添加的类型).。一种实例是唯一且不可改变的数据类型。
- 以及对象（Object）。

虽然这些数据类型相对来说比较少，但是通过他们你可以在程序中开发有用的功能。对象（Objects）和函数（functions）是这门语言的另外两个基本元素。你可以把对象当作存放值的一个命名容器，然后将函数当作你的程序能够执行的步骤。

示例：

```javascript
/*
* 数据类型指的就是字面量的类型
*  在JS中一共有六种数据类型
* 		String 字符串
* 		Number 数值
* 		Boolean 布尔值
* 		Null 空值
* 		Undefined 未定义
* 		Object 对象
* 
* 其中String Number Boolean Null Undefined属于基本数据类型
* 	而Object属于引用数据类型
*/

/*
* String字符串
* 	- 在JS中字符串需要使用引号引起来
* 	- 使用双引号或单引号都可以，但是不要混着用
* 	- 引号不能嵌套，双引号不能放双引号，单引号不能放单引号
*/
var str = 'hello';

str = '我说:"今天天气真不错！"';

/*
在字符串中我们可以使用\作为转义字符，
	当表示一些特殊符号时可以使用\进行转义
	
	\" 表示 "
	\' 表示 '
	\n 表示换行
	\t 制表符
	\\ 表示\
* */
str = "我说:\"今天\t天气真不错！\"";
str = "\\\\\\";
//输出字面量 字符串str
//alert("str");
//输出变量str
//alert(str);
var str2 = "hello";
str2 = "你好";
str2 = 3;
//alert("hello，你好");
//console.log("我就是不出来~~~");

/*
	* 在JS中所有的数值都是Number类型，
	* 	包括整数和浮点数（小数）
	* 
	* JS中可以表示的数字的最大值
	* 	Number.MAX_VALUE
	* 		1.7976931348623157e+308
	* 
	* 	Number.MIN_VALUE 大于0的最小值
	* 		5e-324
	* 
	*  如果使用Number表示的数字超过了最大值，则会返回一个
	* 		Infinity 表示正无穷
	* 		-Infinity 表示负无穷
	* 		使用typeof检查Infinity也会返回number
	*  NaN 是一个特殊的数字，表示Not A Number
	* 		使用typeof检查一个NaN也会返回number
	*/
//数字123
var a = 123;
//字符串123
var b = "123";
/*
	可以使用一个运算符 typeof
		来检查一个变量的类型
	语法：typeof 变量	
	检查字符串时，会返回string
	检查数值时，会返回number
	* */
//console.log(typeof b);

a = -Number.MAX_VALUE * Number.MAX_VALUE;

a = "abc" * "bcd";

a = NaN;

//console.log(typeof a);

a = Number.MIN_VALUE;

//console.log(a);

/*
	* 在JS中整数的运算基本可以保证精确
	*/
var c = 1865789 + 7654321;

/*
	* 如果使用JS进行浮点运算，可能得到一个不精确的结果
	* 	所以千万不要使用JS进行对精确度要求比较高的运算	
	*/
var c = 0.1 + 0.2;

console.log(c);

/*
	* Boolean 布尔值
	* 	布尔值只有两个，主要用来做逻辑判断
	* 	true
	* 		- 表示真
	* 	false
	* 		- 表示假
	* 
	* 使用typeof检查一个布尔值时，会返回boolean
	*/

var bool = false;

console.log(typeof bool);
console.log(bool);

/*
	* Null（空值）类型的值只有一个，就是null
	* 	null这个值专门用来表示一个为空的对象
	* 	使用typeof检查一个null值时，会返回object
	* 
	* Undefined（未定义）类型的值只有一个，就undefind
	* 	当声明一个变量，但是并不给变量赋值时，它的值就是undefined
	* 	使用typeof检查一个undefined时也会返回undefined
	*/
var a = null;

var b = undefined;

console.log(typeof b);
```



#### 数据类型的转换

JavaScript是一种动态类型语言(dynamically typed language)。这意味着你在声明变量时可以不必指定数据类型，而数据类型会在代码执行时会根据需要自动转换。因此，你可以按照如下方式来定义变量：

```
var answer = 42;
```

然后，你还可以给同一个变量赋予一个字符串值，例如：

```
answer = "Thanks for all the fish...";
```

因为 JavaScript 是动态类型的，这种赋值方式并不会提示出错。

在包含的数字和字符串的表达式中使用加法运算符（+），JavaScript 会把数字转换成字符串。例如，观察以下语句：

```
x = "The answer is " + 42 // "The answer is 42"
y = 42+3 + " is the answer"+3+6 // "45 is the answer36"
```

在涉及其它运算符（译注：如下面的减号'-'）时，JavaScript语言不会把数字变为字符串。例如（译注：第一例是数学运算，第二例是字符串运算）：

```
"37" - 7 // 30
"37" + 7 // "377"
```

##### 字符串转换为数字

有一些方法可以将内存中表示一个数字的字符串转换为对应的数字：parseInt()和parseFloat()

` parseInt `方法只能返回整数，所以使用它会丢失小数部分。另外，调用 parseInt 时最好总是带上进制(radix) 参数，这个参数用于指定使用哪一种进制。

将字符串转换为数字的另一种方法是使用一元**加法运算符**。

```
"1.1" + "1.1" = "1.11.1"
(+"1.1") + (+"1.1") = 2.2   
// 注意：加入括号为清楚起见，不是必需的。
```

示例：

```javascript

/*
* 强制类型转换
* 	- 指将一个数据类型强制转换为其他的数据类型
* 	- 类型转换主要指，将其他的数据类型，转换为
* 		String Number Boolean
* 		
*/

/*
* 将其他的数据类型转换为String
* 	方式一：
* 		- 调用被转换数据类型的toString([进制])方法
* 		- 该方法不会影响到原变量，它会将转换的结果返回
* 		- 但是注意：null和undefined这两个值没有toString()方法，
* 			如果调用他们的方法，会报错
* 
*  方式二：
* 		- 调用String()函数，并将被转换的数据作为参数传递给函数
* 		- 使用String()函数做强制类型转换时，
* 			对于Number和Boolean实际上就是调用的toString()方法
* 			但是对于null和undefined，就不会调用toString()方法
* 				它会将 null 直接转换为 "null"
* 				将 undefined 直接转换为 "undefined"
* 
*/

var a = 123;			
//调用a的toString()方法
//调用xxx的yyy()方法，就是xxx.yyy()
a = a.toString();

a = true;
a = a.toString();

a = null;
//a = a.toString(); //报错			
a = undefined;
//a = a.toString(); //报错						
a = 123;			
//调用String()函数，来将a转换为字符串
a = String(a);			
a = null;
a = String(a);			
a = undefined;
a = String(a);			
console.log(typeof a);
console.log(a);		


/*
	* 将其他的数据类型转换为Number
	* 	 转换方式一：
	* 		使用Number()函数
	* 			- 字符串 --> 数字
	* 				1.如果是纯数字的字符串，则直接将其转换为数字
	* 				2.如果字符串中有非数字的内容，则转换为NaN
	* 				3.如果字符串是一个空串或者是一个全是空格的字符串，则转换为0
	* 			- 布尔 --> 数字
	* 				true 转成 1
	* 				false 转成 0
	* 
	* 			- null --> 数字     0
	* 
	* 			- undefined --> 数字 NaN
	* 
	* 转换方式二：
	* 		- 这种方式专门用来对付字符串
	* 		- parseInt() 把一个字符串转换为一个整数
	* 		- parseFloat() 把一个字符串转换为一个浮点数
	*/

var a = "123";

//调用Number()函数来将a转换为Number类型
a = Number(a);

a = false;
a = Number(a);

a = null;
a = Number(a);

a = undefined;
a = Number(a);

a = "123567a567px";
//调用parseInt()函数将a转换为Number
/*
	* parseInt()可以将一个字符串中的有效的整数内容去出来，
	* 	然后转换为Number
	*/
a = parseInt(a);

/*
	* parseFloat()作用和parseInt()类似，不同的是它可以获得有效的小数
	*/
a = "123.456.789px";
a = parseFloat(a);

/*
	* 如果对非String使用parseInt()或parseFloat()
	* 	它会先将其转换为String然后在操作
	*/
a = true;
a = parseInt(a);

a = 198.23;
a = parseInt(a);

console.log(typeof a);
console.log(a);

/*
	* 将其他的数据类型转换为Boolean
	* 	- 使用Boolean()函数
	* 		- 数字 ---> 布尔
	* 			- 除了0和NaN，其余的都是true
	* 
	* 		- 字符串 ---> 布尔
	* 			- 除了空串，其余的都是true
	* 
	* 		- null和undefined都会转换为false
	* 
	* 		- 对象也会转换为true
	* 		
	*/

var a = 123; //true
a = -123; //true
a = 0; //false
a = Infinity; //true
a = NaN; //false

//调用Boolean()函数来将a转换为布尔值
a = Boolean(a);

a = " ";

a = Boolean(a);

a = null; //false
a = Boolean(a);

a = undefined; //false
a = Boolean(a);

console.log(typeof a);
console.log(a);
```

#### 运算符

```javascript
/*
	* 运算符也叫操作符
	* 	通过运算符可以对一个或多个值进行运算,并获取运算结果
	* 	比如：typeof就是运算符，可以来获得一个值的类型
	* 		它会将该值的类型以字符串的形式返回
	* 		number string boolean undefined object
	* 
	* 	算数运算符
	* 		当对非Number类型的值进行运算时，会将这些值转换为Number然后在运算
	* 			任何值和NaN做运算都得NaN
	* 
	* 		+
	* 			+可以对两个值进行加法运算，并将结果返回
	* 			 如果对两个字符串进行加法运算，则会做拼串
	* 				会将两个字符串拼接为一个字符串，并返回
	* 			任何的值和字符串做加法运算，都会先转换为字符串，然后再和字符串做拼串的操作
	* 		-
	* 			- 可以对两个值进行减法运算，并将结果返回
	* 
	* 		*
	* 			* 可以对两个值进行乘法运算
	* 		/
	* 			/ 可以对两个值进行除法运算
	* 		%
	* 			% 取模运算（取余数）
	*/
var a = 123;

var result = typeof a;

//console.log(typeof result);

result = a + 1;

result = 456 + 789;

result = true + 1;

result = true + false;

result = 2 + null;

result = 2 + NaN;

result = "你好" + "大帅哥";

var str = "锄禾日当午，" +
			"汗滴禾下土，" +
			"谁知盘中餐，" +
			"粒粒皆辛苦";
			
			
result = 123 + "1";

result = true + "hello";

//任何值和字符串相加都会转换为字符串，并做拼串操作
/*
	* 我们可以利用这一特点，来将一个任意的数据类型转换为String
	* 	我们只需要为任意的数据类型 + 一个 "" 即可将其转换为String
	* 	这是一种隐式的类型转换，由浏览器自动完成，实际上它也是调用String()函数
	*/
var c = 123;

c = c + "";

//c = null;

//c = c + "";


//console.log(result);
//console.log(typeof c);
//console.log("c = "+c);

result = 1 + 2 + "3"; //33

result = "1" + 2 + 3; //123

result = 100 - 5;

result = 100 - true;

result = 100 - "1";

result = 2 * 2;

result = 2 * "8";

result = 2 * undefined;

result = 2 * null;

result = 4 / 2;

result = 3 / 2;

/*
	* 任何值做- * /运算时都会自动转换为Number
	* 	我们可以利用这一特点做隐式的类型转换
	* 		可以通过为一个值 -0 *1 /1来将其转换为Number
	* 		原理和Number()函数一样，使用起来更加简单
	*/

var d = "123";

//console.log("result = "+result);

d = d - 0;

/*console.log(typeof d);
console.log(d);*/

result = 9 % 3;
result = 9 % 4;
result = 9 % 5;

console.log("result = "+result);
```

**一元运算符**

```javascript
/*
	* 一元运算符，只需要一个操作数
	* 	+ 正号
	* 		- 正号不会对数字产生任何影响
	* 	- 负号
	* 		- 负号可以对数字进行负号的取反
	* 
	* 	- 对于非Number类型的值，
	* 		它会将先转换为Number，然后在运算
	* 		可以对一个其他的数据类型使用+,来将其转换为number
	* 		它的原理和Number()函数一样
	*/

var a = 123;

a = -a;

a = true;

a = "18";

a = +a;

/*console.log("a = "+a);
console.log(typeof a);*/

var result = 1 + +"2" + 3;

console.log("result = "+result);
var a ="a = "+ 1 + +"2" + 3;
a = +"2";
console.log(typeof a);
console.log(a);
```

**自增自减**

```javascript
/*
	* 自增 ++
	* 	 - 通过自增可以使变量在自身的基础上增加1
	* 	 - 对于一个变量自增以后，原变量的值会立即自增1
	* 	 - 自增分成两种：后++(a++) 和 前++(++a)	
	* 		无论是a++ 还是 ++a，都会立即使原变量的值自增1
	* 			不同的是a++ 和 ++a的值不同
	* 		a++的值等于原变量的值（自增前的值）
	* 		++a的值等于新值 （自增后的值）
	* 
	* 自减 --
	* 	- 通过自减可以使变量在自身的基础上减1
	* 	- 自减分成两种：后--(a--) 和 前--(--a)
	* 		无论是a-- 还是 --a 都会立即使原变量的值自减1
	* 			不同的是a-- 和 --a的值不同
	* 				a-- 是变量的原值 （自减前的值）
	* 				--a 是变量的新值 （自减以后的值）
	* 			
	* 	
	*/
var n1=10;
var n2=20;

var n = n1++; //n1 = 11  n1++ = 10

console.log('n='+n);  // 10
console.log('n1='+n1); //11

n = ++n1 //n1 = 12  ++n1 =12
console.log('n='+n); //12
console.log('n1='+n1); //12

n = n2--;// n2=19 n2--=20
console.log('n='+n); //20
console.log('n2='+n2); //19

n = --n2; //n2=18 --n2 = 18
console.log('n='+n); //18
console.log('n2='+n2); //18
```

**逻辑运算符**

```javascript
/*
	* JS中为我们提供了三种逻辑运算符
	* ! 非
	* 	- !可以用来对一个值进行非运算
	* 	- 所谓非运算就是值对一个布尔值进行取反操作，
	* 		true变false，false变true
	* 	- 如果对一个值进行两次取反，它不会变化
	* 	- 如果对非布尔值进行元素，则会将其转换为布尔值，然后再取反
	* 		所以我们可以利用该特点，来将一个其他的数据类型转换为布尔值
	* 		可以为一个任意数据类型取两次反，来将其转换为布尔值，
	* 		原理和Boolean()函数一样
	* 
	* && 与
	* 	- &&可以对符号两侧的值进行与运算并返回结果
	* 	- 运算规则
	* 		- 两个值中只要有一个值为false就返回false，
	* 			只有两个值都为true时，才会返回true
	* 		- JS中的“与”属于短路的与，
	* 			如果第一个值为false，则不会看第二个值
	* 
	* || 或
	* 	- ||可以对符号两侧的值进行或运算并返回结果
	* 	- 运算规则：
	* 		- 两个值中只要有一个true，就返回true
	* 			如果两个值都为false，才返回false
	*		- JS中的“或”属于短路的或
	* 			如果第一个值为true，则不会检查第二个值
	*/

//如果两个值都是true则返回true
var result = true && true;

//只要有一个false，就返回false
result = true && false;
result = false && true;
result = false && false;

//console.log("result = "+result);

//第一个值为true，会检查第二个值
//true && alert("看我出不出来！！");

//第一个值为false，不会检查第二个值
//false && alert("看我出不出来！！");

//两个都是false，则返回false
result = false || false;

//只有有一个true，就返回true
result = true || false;
result = false || true ;
result = true || true ;

//console.log("result = "+result);

//第一个值为false，则会检查第二个值
//false || alert("123");

//第一个值为true，则不再检查第二个值
//true || alert("123");



var a = false;

//对a进行非运算
a = !a;

//console.log("a = "+a);

var b = 10;
b = !!b;

//console.log("b = "+b);
//console.log(typeof b);

/*
	* && || 非布尔值的情况
	* 	- 对于非布尔值进行与或运算时，
	* 		会先将其转换为布尔值，然后再运算，并且返回原值
	* 	- 与运算：
	* 		- 如果第一个值为true，则必然返回第二个值
	* 		- 如果第一个值为false，则直接返回第一个值
	* 
	* 	- 或运算
	* 		- 如果第一个值为true，则直接返回第一个值
	* 		- 如果第一个值为false，则返回第二个值
	* 
	*/

//true && true
//与运算：如果两个值都为true，则返回后边的
var result = 5 && 6;


//与运算：如果两个值中有false，则返回靠前的false
//false && true
result = 0 && 2;
result = 2 && 0;
//false &&　false
result = NaN && 0;
result = 0 && NaN;


//true || true
//如果第一个值为true，则直接返回第一个值
result = 2 || 1;
result = 2 || NaN;
result = 2 || 0;

//如果第一个值为false，则直接返回第二个值
result = NaN || 1;
result = NaN || 0;

result = "" || "hello";

result = -1 || "你好";
console.log("result = "+result);
```

**赋值运算符**

```javascript
/*
* =
* 	可以将符号右侧的值赋值给符号左侧的变量
* += 
* 	a += 5 等价于 a = a + 5
* -=
* 	a -= 5 等价于 a = a - 5
* *=
* 	a *= 5 等价于 a = a * 5
* /=
* 	a /= 5 等价于 a = a / 5
* %=
* 	a %= 5 等价于 a = a % 5
* 	
*/
var a = 10;

//a = a + 5;
//a += 5;

//a -= 5;

//a *= 5;

// a = a%3;
a %= 3;

console.log("a = "+a);
```

**关系运算符**

```javascript
/*
* 通过关系运算符可以比较两个值之间的大小关系，
* 	如果关系成立它会返回true，如果关系不成立则返回false
* 
* > 大于号
* 	- 判断符号左侧的值是否大于右侧的值
* 	- 如果关系成立，返回true，如果关系不成立则返回false
* 
* >= 大于等于
* 	- 判断符号左侧的值是否大于或等于右侧的值
* 
* < 小于号
* <= 小于等于
* 
* 非数值的情况
* 	- 对于非数值进行比较时，会将其转换为数字然后在比较
* 	- 如果符号两侧的值都是字符串时，不会将其转换为数字进行比较
* 		而会分别比较字符串中字符的Unicode编码
*/

var result = 5 > 10;//false

result = 5 > 4; //true

result = 5 > 5; //false

result = 5 >= 5; //true

result = 5 >= 4; //true

result = 5 < 4; //false

result = 4 <= 4; //true

//console.log("result = "+result);

//console.log(1 > true); //false
//console.log(1 >= true); //true
//console.log(1 > "0"); //true
//console.log(10 > null); //true
//任何值和NaN做任何比较都是false
//console.log(10 <= "hello"); //false
//console.log(true > false); //true

//console.log("1" < "5"); //true
//console.log("11" < "5"); //true

//比较两个字符串时，比较的是字符串的字符编码
//console.log("a" < "b");//true
//比较字符编码时是一位一位进行比较
//如果两位一样，则比较下一位，所以借用它来对英文进行排序
//console.log("abc" < "bcd");//true
//比较中文时没有意义
//console.log("戒" > "我"); //true

//如果比较的两个字符串型的数字，可能会得到不可预期的结果
//注意：在比较两个字符串型的数字时，一定一定一定要转型
console.log("11123123123123123123" < +"5"); //true
```

**相等运算符**

```javascript
/*
	* 相等运算符用来比较两个值是否相等，
	* 	如果相等会返回true，否则返回false
	* 
	* 使用 == 来做相等运算
	* 	- 当使用==来比较两个值时，如果值的类型不同，
	* 		则会自动进行类型转换，将其转换为相同的类型
	* 		然后在比较
	* 不相等
	* 	 不相等用来判断两个值是否不相等，如果不相等返回true，否则返回false
	* 	- 使用 != 来做不相等运算
	* 	- 不相等也会对变量进行自动的类型转换，如果转换后相等它也会返回false
	* 
	* 		
	*  ===
	* 		全等
	* 		- 用来判断两个值是否全等，它和相等类似，不同的是它不会做自动的类型转换
	* 			如果两个值的类型不同，直接返回false
	* 	!==
	* 		不全等
	* 		- 用来判断两个值是否不全等，和不等类似，不同的是它不会做自动的类型转换
	* 			如果两个值的类型不同，直接返回true
	*/

//console.log(1 == 1); //true

var a = 10;

//console.log(a == 4); //false

//console.log("1" == 1); //true

//console.log(true == "1"); //true

//console.log(null == 0); //false

/*
	* undefined 衍生自 null
	* 	所以这两个值做相等判断时，会返回true
	*/
//console.log(undefined == null);

/*
	* NaN不和任何值相等，包括他本身
	*/
//console.log(NaN == NaN); //false

var b = NaN;

//判断b的值是否是NaN
//console.log(b == NaN);
/*
	* 可以通过isNaN()函数来判断一个值是否是NaN
	* 	如果该值是NaN则返回true，否则返回false
	*/
//console.log(isNaN(b));

//console.log(10 != 5); //true
//console.log(10 != 10); //false
//console.log("abcd" != "abcd"); //false
//console.log("1" != 1);//false

//console.log("123" === 123);//false
//console.log(null === undefined);//false

console.log(1 !== "1"); //true
```

**条件运算符**

```
/*
* 条件运算符也叫三元运算符
* 	语法：
* 		条件表达式?语句1:语句2;
* 	- 执行的流程：
* 		条件运算符在执行时，首先对条件表达式进行求值，
* 			如果该值为true，则执行语句1，并返回执行结果
* 			如果该值为false，则执行语句2，并返回执行结果
* 		如果条件的表达式的求值结果是一个非布尔值，
* 			会将其转换为布尔值然后在运算
*/

//false?alert("语句1"):alert("语句2");

var a = 300;
var b = 143;
var c = 50;

//a > b ? alert("a大"):alert("b大");

//获取a和b中的最大值
//var max = a > b ? a : b;
//获取a b c 中的大值
//max = max > c ? max : c;

//这种写法不推荐使用，不方便阅读
var max = a > b ? (a > c ? a :c) : (b > c ? b : c);

//console.log("max = "+max);

//"hello"?alert("语句1"):alert("语句2");
```

**运算符优先级**

JavaScript中的运算符优先级是一套规则。该规则在计算表达式时控制运算符执行的顺序。具有较高优先级的运算符先于较低优先级的运算符执行。例如，乘法的执行先于加法。

下表按从最高到最低的优先级列出JavaScript运算符。具有相同优先级的运算符按从左至右的顺序求值。

| 运算符                             | 描述                                         |
| ---------------------------------- | -------------------------------------------- |
| . [] ()                            | 字段访问、数组下标、函数调用以及表达式分组   |
| ++ -- - ~ ! delete new typeof void | 一元运算符、返回数据类型、对象创建、未定义值 |
| * / %                              | 乘法、除法、取模                             |
| + - +                              | 加法、减法、字符串连接                       |
| << >> >>>                          | 移位                                         |
| < <= > >= instanceof               | 小于、小于等于、大于、大于等于、instanceof   |
| == != === !==                      | 等于、不等于、严格相等、非严格相等           |
| &                                  | 按位与                                       |
| ^                                  | 按位异或                                     |
| \|                                 | 按位或                                       |
| &&                                 | 逻辑与                                       |
| \|\|                               | 逻辑或                                       |
| ?:                                 | 条件                                         |
| = oP=                              | 赋值、运算赋值                               |
| ,                                  | 多重求值                                     |

圆括号可用来改变运算符优先级所决定的求值顺序。这意味着圆括号中的表达式应在其用于表达式的其余部分之前全部被求值。

