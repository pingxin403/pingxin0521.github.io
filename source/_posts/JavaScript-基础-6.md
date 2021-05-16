---
title: JavaScript 流程控制
date: 2019-05-01 06:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

### 流程控制

我们的程序是由一条一条语句构成的，语句是按照自上向下的顺序一条一条执行的，在JS中可以使用{}来为语句进行分组,同一个{}中的语句我们称为是一组语句，它们要么都执行，要么都不执行，一个{}中的语句我们也称为叫一个代码块，在代码块的后边就不用再编写;了

<!--more-->

```javascript
/*
* 
* 	JS中的代码块，只具有分组的的作用，没有其他的用途
* 		代码块内容的内容，在外部是完全可见的
*/
{
var a = 10;	
alert("hello");
console.log("你好");
document.write("语句");
}

console.log("a = "+a);
```

流程控制：就是程序代码执行顺序，通过规定的语句让程序代码有条件的按照一定的方式执行

程序的三种基本结构

**顺序结构**： 从上到下，从左到右执行的顺序，就叫做顺序结构，程序默认就是由上到下顺序执行的

**分支结构**：根据不同的情况，执行对应代码

**循环结构**：重复做一件事情

#### 分支结构

##### if语句

```
if (/* 条件表达式 */) {
  // 执行语句
}
```

**if --- else 语句**

```
if (/* 条件表达式 */) {
// 成立执行语句
} else {
  // 否则执行语句
}
    
 //举例:
 //获取两个数字中的最大值
var num1=100;
var num2=20;
if(num1>num2){
    console.log(num1);
}else{
    console.log(num2);
}

// 判断一个数是偶数还是奇数
var n = 10;
if(n % 2 == 0){
    console.log('偶数');
}else{
    console.log('奇数');
}

//判断是否成年了
var age = parseInt(prompt("请您输入年龄"));
if(age >= 18){
    console.log('恭喜您,您成年了!!');
}else{
    console.log('不好意思,您还未成年,快回家去写作业');
}


```

##### if  ---  else if 语句

```
if (/* 条件1 */){
  // 成立执行语句
} else if (/* 条件2 */){
  // 成立执行语句
} else if (/* 条件3 */){
  // 成立执行语句
} else {
  // 最后默认执行语句
}
           
 //举例:
    /*
    * 例子:
    * 获取考试的分数,如果成绩是在90(含)分以上的,则显示级别:A
    * 如果成绩是大于等于80的则:B
    * 如果成绩是大于等于70的则:C
    * 如果成绩是大于等于60的则:D
    * 如果成绩是小于60的则:E
    *
    * */
    var score = parseInt(prompt("请输入您的成绩:(0-100之间)"));
    if (!isNaN(score) && score <= 100) {
        if (score >= 90) {
            console.log("您的成绩为A");
        } else if (score >= 80) {
            console.log("您的成绩为B");
        } else if (score >= 70) {
            console.log("您的成绩为C");
        } else if (score >= 60) {
            console.log("您的成绩为D");
        } else {
            console.log("您的成绩为E");
        }
    } else {
        console.log("您输入的成绩有误!");
    }


```

案例 判断一个年份是闰年还是平年

```
判断一个年份是闰年还是平年
闰年：能被4整除，但不能被100整除的年份 或者 能被400整除的年份
    var n = 2016;
    if(n % 4 == 0){
        if(n % 100 != 0){
            console.log('闰年');
        }else if(n % 400 == 0){
            console.log('闰年');
        }else{
            console.log('平年');
        }
    }else{
        console.log('平年');
    }
```

##### switch语句

语法格式:

```

   switch (表达式) {
        case 常量1:  
            语句;
            break;
        case 常量2:
            语句;
            break;
            …
        case 常量n:
            语句;
            break;
        default:
            语句;
            break;
    }
/*

- 执行过程:
- 获取表达式的值,和值1比较,相同则执行代码1,遇到break跳出整个语句,结束
- 如果和值1不匹配,则和值2比较,相同则执行代码2,遇到break跳出整个语句,结束
- 如果和值2不匹配,则和值3比较,相同则执行代码3,遇到break跳出整个语句,结束
- 如果和值3不匹配,则和值4比较,相同则执行代码4,遇到break跳出整个语句,结束
- 如果和之前的所有的值都不一样,则直接执行代码5,结束
  */
```

break可以省略，如果省略，代码会继续执行下一个case，

switch 语句在比较值时使用的是全等操作符, 因此不会发生类型转换（例如，字符串'10' 不等于数值 10）

案例：

```
/* *

- 判断这个人的成绩的级别:
- 如果是A,则提示,分数在90分以上
- 如果是B,则提示,分数在80分以上
- 如果是C,则提示,分数在70分以上
- 如果是D,则提示,分数在60分以上
- 否则提示,不及格
- */
  varjiBie = "B";
  switch (jiBie){
    case "A" : 
        console.log("分数在90分以上的");
        break;
    case "B" : 
        console.log("分数在80分以上的");
        break;
    case "C" : 
        console.log("分数在70分以上的");
        break;
    case "D" : 
        console.log("分数在60分以上的");
        break;
    default :
        console.log("不及格");
  }
```

#### 循环结构

在javascript中，循环语句有三种，while、do..while、for循环。

#####  while语句

基本语法：

```
// 当循环条件为true时，执行循环体，
// 当循环条件为false时，结束循环。
while (循环条件) {
  //循环体
}
```

案例1：计算1-100之间所有数的和

```
// 初始化变量
var i = 1;
var sum = 0;
while (i <= 100) {   // 判断条件
  sum += i;  // 循环体
  i++;  // 自增
}
console.log(sum);
```

案例2：打印100以内7的倍数

```
var i = 1;
while(i < 100){
    if(i % 7 == 0){
        console.log(i);
    }
    i++;
}
```

案例3：求账号和密码是否一致

```
  var userName = prompt("请您输入账号");
  var userPass = prompt("请您输入密码");
  while (userName != 'admin' || userPass != '123456') {
     userName = prompt("请您输入账号");
     userPass = prompt("请您输入密码");
    }

  alert("登录成功!");
```

案例4：

```
/*
	* 	从键盘输入小明的期末成绩:
	*	当成绩为100时，'奖励一辆BMW'
	*	当成绩为[80-99]时，'奖励一台iphone15s'
	*	当成绩为[60-80]时，'奖励一本参考书'
	*	其他时，什么奖励也没有
	*/

/*
	* prompt()可以弹出一个提示框，该提示框中会带有一个文本框，
	* 	用户可以在文本框中输入一段内容，该函数需要一个字符串作为参数，
	* 	该字符串将会作为提示框的提示文字
	* 
	* 用户输入的内容将会作为函数的返回值返回，可以定义一个变量来接收该内容
	*/
//将prompt放入到一个循环中
while(true){
	//score就是小明的期末成绩
	var score = prompt("请输入小明的期末成绩(0-100):");
	//判断用户输入的值是否合法
	if(score >= 0 && score <= 100){
		//满足该条件则证明用户的输入合法，退出循环
		break;
	}
	
	alert("请输入有效的分数！！！");
}



//判断值是否合法
if(score > 100 || score < 0 || isNaN(score)){
	alert("拉出去毙了~~~");
}else{
	//根据score的值来决定给小明什么奖励
	if(score == 100){
		//奖励一台宝马
		alert("宝马，拿去~~~");
	}else if(score >= 80){
		//奖励一个手机
		alert("手机，拿去玩~~~");
	}else if(score >= 60){
		//奖励一本参考书
		alert("参考书，拿去看~~~");
	}else{
		alert("棍子一根~~");
	}
}
```

**do...while语句**

do..while循环 和 while循环 非常像，二者经常可以相互替代，

但是do..while的特点是不管条件成不成立，都会先执行一次。

执行过程 : 先执行一次循环体 , 判断条件是否成立 , 不成立则跳出循环 , 成立则执行循环体 , 然后再判断条件是否成立......

```
do {
  // 循环体;
} while (循环条件);
```

案例：计算1-100的和

```
// 初始化变量
var i = 0;
var sum = 1;
do {
  sum += i;//循环体
  i++;//自增
} while (i <= 100);//循环条件
```

**for语句**

while和do...while一般用来解决无法确认次数的循环。for循环一般在循环次数确定的时候比较方便

for循环语法：

```
// for循环的表达式之间用的是分号分隔的
for (初始化表达式1; 判断表达式2; 自增表达式3) {
  // 循环体4
}
```

执行顺序：1243  ----  243   -----243(直到循环条件变成false)

1. 初始化表达式
2. 判断表达式
3. 自增表达式
4. 循环体

```
//求1-100之间所有偶数的和
var s = 0;
for(var i = 1;i <= 100; i++){
    if(i % 2 == 0){
        s += i;
    }
}
console.log(s);

//打印9*9乘法表
var str = '';
for (var i = 1; i <= 9; i++) {
  for (var j = i; j <=9; j++) {
    str += i + ' * ' + j + ' = ' + i * j + '\t';
  }
  str += '\n';
}
console.log(str);
```

水仙花数：

```
/*
	* 水仙花数是指一个3位数，它的每个位上的数字的3 次幂之和等于它本身。
	（例如：1^3 + 5^3 + 3^3 = 153）,请打印所有的水仙花数。
	*/

//打印所有的三位数
for(var i=100 ; i<1000 ; i++){
	
	//获取i的百位 十位 个位的数字
	//获取百位数字
	var bai = parseInt(i/100);
	
	//获取十位的数字
	var shi = parseInt((i-bai*100)/10);
	
	//获取个位数字
	var ge = i % 10;
	
	//判断i是否是水仙花数
	if(bai*bai*bai + shi*shi*shi + ge*ge*ge == i){
		console.log(i);
	}	
}
```

判断质数：

```
/*
	* 在页面中接收一个用户输入的数字，并判断该数是否是质数。
	质数：只能被1和它自身整除的数，1不是质数也不是合数，质数必须是大于1的自然数。	
	*/

var num = prompt("请输入一个大于1的整数:");


//判断这个值是否合法
if(num <= 1){
	alert("该值不合法！");
}else{
	
	//创建一个变量来保存当前的数的状态
	//默认当前num是质数
	var flag = true;
	
	//判断num是否是质数
	//获取2-num之间的数
	for(var i=2 ; i<num ; i++){
		//console.log(i);
		//判断num是否能被i整除
		if(num % i == 0){
			//如果num能被i整除，则说明num一定不是质数
			//设置flag为false
			flag = false;
		}
	}
	
	//如果num是质数则输出
	if(flag){
		alert(num + "是质数！！！");
	}else{
		alert("这个不是质数")
	}
	
	
}
```

 **continue和break**

break:立即跳出整个循环，即循环结束，开始执行循环后面的内容（直接跳到大括号）

continue:立即跳出当前循环，继续下一次循环（跳到i++的地方）

案例1：求1-100之间不能被7整除的整数的和（用continue）

```
var s = 0;
for(var i = 0; i < 100; i++){
    if(i % 7 == 0){
        continue;
    }
    s += i;
}

console.log(s);
```

**for in** 

```
for (variable in object) {...}
```

- `variable`

  在每次迭代时，将不同的属性名分配给*变量*。

- `object`

  被迭代枚举其属性的对象。

for...in 循环只遍历可枚举属性。像 Array和 Object使用内置构造函数所创建的对象都会继承自Object.prototype和String.prototype的不可枚举属性，例如 String 的 indexOf()  方法或 Object的toString()方法。循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）。

`for...in` 循环以任意序迭代一个对象的属性。如果一个属性在一次迭代中被修改，在稍后被访问，其在循环中的值是其在稍后时间的值。一个在被访问之前已经被删除的属性将不会在之后被访问。在迭代进行时被添加到对象的属性，可能在之后的迭代被访问，也可能被忽略。

通常，在迭代过程中最好不要在对象上进行添加、修改或者删除属性的操作，除非是对当前正在被访问的属性。这里并不保证是否一个被添加的属性在迭代过程中会被访问到，不保证一个修改后的属性（除非是正在被访问的）会在修改前或者修改后被访问，不保证一个被删除的属性将会在它被删除之前被访问。

示例：

```
var obj = {a:1, b:2, c:3};
    
for (var prop in obj) {
  console.log("obj." + prop + " = " + obj[prop]);
}

// Output:
// "obj.a = 1"
// "obj.b = 2"
// "obj.c = 3"

var triangle = {a: 1, b: 2, c: 3};

function ColoredTriangle() {
  this.color = 'red';
}

ColoredTriangle.prototype = triangle;

var obj = new ColoredTriangle();

for (var prop in obj) {
  if (obj.hasOwnProperty(prop)) {
    console.log(`obj.${prop} = ${obj[prop]}`);
  } 
}

// Output:
// "obj.color = red"
```

**for each...in**

```
for each (variable in object) {
  statement
}
```

- `variable`

  用来遍历属性值的变量，前面的`var`关键字是可选的。该变量是函数的局部变量而不是语句块的局部变量。

- `object`

  属性值会被遍历的对象。

- `statement`

  遍历属性时执行的语句。如果想要执行多条语句，请用[块语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/block)(`{ ... }`) 将多条语句括住。

一些对象的内置属性是无法被遍历到的，包括所有的内置方法，例如String对象的`indexOf`方法。不过，大部分的用户自定义属性都是可遍历的.

```
var sum = 0;
var obj = {prop1: 5, prop2: 13, prop3: 8};

for each (var item in obj) {
  sum += item;
}

print(sum); // 输出"26",也就是5+13+8的值
```

**for...of**

```
for (variable of iterable) {
    //statements
}
```

- `variable`

  在每次迭代中，将不同属性的值分配给变量。

- `iterable`

  被迭代枚举其属性的对象。



for...of语句在可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

```
let iterable = [10, 20, 30];

for (let value of iterable) {
    value += 1;
    console.log(value);
}
// 11
// 21
// 31
```

如果你不想修改语句块中的变量 , 也可以使用[`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)代替`let`。

**for...of与for...in的区别**

无论是for...in还是for...of语句都是迭代一些东西。它们之间的主要区别在于它们的迭代方式。

for...in 语句以原始插入顺序迭代对象的可枚举属性。

for...of 语句遍历可迭代对象定义要迭代的数据。

```
Object.prototype.objCustom = function() {}; 
Array.prototype.arrCustom = function() {};

let iterable = [3, 5, 7];
iterable.foo = 'hello';

for (let i in iterable) {
  console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
}

for (let i in iterable) {
  if (iterable.hasOwnProperty(i)) {
    console.log(i); // logs 0, 1, 2, "foo"
  }
}

for (let i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

