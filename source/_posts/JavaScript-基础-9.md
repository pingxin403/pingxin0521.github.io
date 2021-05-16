---
title: JavaScript 常用知识
date: 2019-05-21 11:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

#### var 和 let 和 const 关键字的区别

1. 关于变量提升,var能变量提升，let不能

<!--more-->


```
 // 关于var 如下所示
console.log(a); //输出undefined，此时就是变量提升
var a = 2;  
console.log(a); //2

//相当于下面的代码
var a; //声明且初始化为undefined
console.log(a); //输出undefined
a=2;    //赋值
console.log(a); //2

// 关于let 如下所示
console.log(a); // 报错ReferenceError
let a = 2;
//相当于在第一行先声明a但没有初始化，直到赋值时才初始化

//直接用let声明变量不赋值是会打印undefined，这时候初始化了
let a;
console.log(a);//值为undefined
```

2. 暂时性死区：块级作用域内存在let命令，它所声明的变量就“绑定”这个区域，不再受外部的影响重点内容，简而言之，就是某个代码块有let指令，即使外部有名称相同的变量，该代码块的同名变量与外部的变量也互不干扰。而var不会，如下所示：


```
//let
var a = 123;
if (true) {
 let a="abc";
 console.log(a); //输出abc 
}
console.log(a);  //输出值为123，全局a与局部a互不影响

//var
var a = 123;
if (true) {
 var a="abc";
 console.log(a); //输出abc 
}
console.log(a);  //输出值为abc,全局的已被改变
```

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。例子如下：

```
var tmp=1;
if (true) {
 // TDZ开始
 tmp = 'abc'; // ReferenceError
 console.log(tmp); // ReferenceError

 let tmp; // TDZ结束
 console.log(tmp); // undefined

 tmp = 123;
 console.log(tmp); // 123
}
console.log(tmp); // 
```

3. let声明绑定的代码块内，不能重复声明同一个变量，var可以


```
//a不能重复声明
function sub() {
 let a = 10;
 var a = 1;
}  //报错，Identifier 'a' has already been declared

function sub() {
 let a = 10;
 let a = 1;
}  //同上

function sub() {
 let a = 10;
 {let a = 1;} //此时不在同一个代码块，不会报错
} 

//var可以重复声明，不会报错
function sub() {
 var a = 10;
 var a = 1;
}
```

4. 类似for循环的代码块，let只在代码块内部有效，var在代码块外部也有效

```
//let只在代码块内部有效
for (let i = 0; i < 10; i++) {}
console.log(i); //报错ReferenceError: i is not defined

//var在代码块外部也有效
for (let i = 0; i < 10; i++) {}
console.log(i); //101

let在for循环内特别之处：就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
//只在父作用域
var a = [];
for (let i = 0; i < 10; i++) {
 a[i] = function () {
  console.log(i);
 };
}
a[6](); // 6

//子作用域重新声明
var a = [];
for (let i = 0; i < 10; i++) {
 a[i] = function () {
   let i=3; //重新赋值
   console.log(i);
 };
}
a[6](); // 3 ，取得新的值
```

**let和const**

1. 相同点： 
   - 变量不提升。 
   - 暂时性死区，只能在声明的位置后面使用。 
   - 不可重复声明。 
   - 都属于块级作用域
2. 不同点： 

let声明的变量可以改变。 
 const声明一个只读的常量。一旦声明，常量的值就不能改变，且声明的时候必须初始化赋值。

```
 let a;  //undefined
 const b;//报错，声明的时候必须赋值

let a=1;
 a=2;    //可改变

const b=1;
 b=2;    //报错，不能改变值

//一些自己觉得要注意的点
 let a=null;         //a=null
 a=undefined;    //a=undefined
 a=2;            //a=2
 const a=null;   //a=null,const也可以定义null和undefined
 const b=undefined;   //b=undefined
 b=2;            //报错，不能改变值
```

本质： 

const实际上保证的，并不是变量的不得改动，而是变量指向的那个内存地址所保存的数据不得改动。 

A.五种基本数据类型（Number,String,Boolean,Undefined,Null）：值就保存在变量指向的那个内存地址，等同于常量。不能改变值。 

B. 复杂数据类型（Object：数组、对象）：该类型变量名不指向数据，而是指向数据所在的地址，const只保证变量名指向的地址不变，并不保证改地址的数据不变，因此可以对该地址的属性值进行修改，但是不能改变地址指向。

```
const a=[];
a.push("Hello"); //可执行，改地址的属性值可以修改
a.length=0;   //可执行，同上
a=["Tom"];   //报错，不能改变地址指向

const b ={};
b.prop=123;   //为b添加一个属性，可以成功
b.prop    //123
b={};    //将b指向另外一个地址，就会报错

如果真的想将对象冻结，应该使用Object.freeze方法。
const b=Object.freeze({});
// 常规模式时，下面一行不起作用,b.prop为undefined
// 严格模式时，该行会报错
b.prop = 123;
```

**推荐**

默认使用const，当需要重新绑定变量时使用let，在ES6不应使用var

#### 箭头函数

lambda表达式（箭头函数）据说是定义函数最简洁的方法，语法上几乎没有冗余成分了。因为JS弱类型的特点，JS中的lambda表达式要比C#和Java中的更简洁（少了参数类型声明）

`arg => returnVal`语法是创建函数最简洁的方式，定义了一个形参为`arg`，返回值为`returnVal`的function

其它语法如下表：

| 语法                                     | 等价代码                                                     | 含义                                                 |
| ---------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| `x => f(x)`                              | `function(x) {     return f(x); }`                           | `y=f(x)`                                             |
| `(x, y)=>x + y;`                         | `function(x, y) {     return x + y; }`                       | `y=f(x,y)=x+y`                                       |
| `(x, y)=>{g(x); g(y); return h(x, y);};` | `function(x, y) {     g(x);     g(y);     return h(x, y); }` | `          g(x), g(y) y=f(x,y)==============h(x,y) ` |
| `()=>({});`                              | `function() {     return {}; }`                              | `y={} `                                              |

P.S.第三列的“含义”指的是数学函数含义，lambda表达式本来就是数学家弄出来的

```
// 简单例子，简化匿名函数的定义
var arr = [1, 3, 21, 12];
console.log(arr.map(x => 2 * x));   // [2, 6, 42, 24]
console.log(arr.sort((a, b) => a - b)); // [1, 3, 12, 21]
arr.forEach((item, index, arr) => {
    if (index %2 == 0) {
        console.log(item);
    }
    if (index == arr.length - 1) {
        console.log(`last item is ${item}`);
    }
});
```

复杂示例：

```
// 复杂例子
var app = {
    cache: {},
    ajax: function(url, callback) {
        var self = this;
        function req(url) {
            var res = `data from ${url}`;
            console.log(`ajax request ${url}`);
            // cache res
            self.cache[url] = res;
            return res;
        }
        var data = req(url);
        callback(data);
    }
}
app.ajax('http://www.xxx.xx', function(data) {
    console.log(`receive: ${data}`);
});
console.log(app.cache);

// 用箭头函数改写
var es6App = {
    cache: {},
    ajax(url, callback) {
        var req = url => {
            var res = `data from ${url}`;
            console.log(`ajax request ${url}`);
            // cache res
            this.cache[url] = res;
            return res;
        }
        var data = req(url);
        callback(data);
    }
}
es6App.ajax('http://www.q.xx', function(data) {
    console.log(`receive: ${data}`);
});
console.log(es6App.cache);
```

**注意**

1. 参数列表与返回值的语法

   1个参数时，左边直接写参数名，0个或者多个参数时，参数列表要用`()`包裹起来

   函数体只有1条语句时，右边值自动成为函数返回值，函数体不止1条语句时，函数体需要用`{}`包裹起来，并且需要手动`return`

   P.S.当然，可能很容易想到不分青红皂白，把`() => {}`作为箭头函数的起手式，但*不建议这样做*，因为下一条说了`{`是有歧义的，可能会带来麻烦

2. 有歧义的字符

   `{`是*唯一*1个有歧义的字符，所以返回对象字面量时需要用`()`包裹，否则会被当作块语句解析

   例如：

   ```
   var f1 = () => {};
   f1();   // 返回undefined
   // 等价于
   // var f1 = function() {};
   
   var f2 = () => ({});
   f2();   // 返回空对象{}
   // 等价于
   // var f2 = function() {return {};};
   ```

3. 关于this

   箭头函数会*从外围作用域继承this*，为了避免`that = this`，需要遵守：*除了对象上的直接函数属性值用function语法外，其它函数都用箭头函数*

   这个规则很容易理解，示例如下：

   ```
   // 场景1
   function MyType() {}
   MyType.prototype.fn = function() {/*定义箭头函数*/};  // 箭头函数中this指向MyType类型实例
   
   // 场景2
   var obj = {};
   obj.fn = function() {/*定义箭头函数*/};   // 箭头函数中this指向obj
   ```

   区别在于`function`关键字定义的函数属性中，该函数的`this`指向这个函数属性所属的对象（匿名函数的`this`指向global对象或者`undefined`）。说白了，`function`能定义一个新`this`，而箭头函数不能，它只能从外层借一个`this`。所以，需要新`this`出现的时候用`function`定义函数，想沿用外层的`this`时就用箭头函数

4. 关于arguments对象

   箭头函数没有arguments对象，因为标准鼓励使用默认参数、可变参数、参数解构

   例如：

   ```
   // 一般函数
   (function(a) {console.log(`a = ${a}, arguments[0] = ${arguments[0]}`)})(1);
   // log print: a = 1, arguments[0] = 1
   
   // 与上面等价的箭头函数
   (a => console.log(`a = ${a}, arguments[0] = ${arguments[0]}`))(1);
   // log print: Uncaught ReferenceError: arguments is not defined
   ```

   这与函数匿名不匿名无关，规则就是箭头函数中无法访问arguments对象（undefined），如下：

   ```
   // 非匿名函数
   var f = a => console.log(`a = ${a}, arguments[0] = ${arguments[0]}`);
   f(2);
   // log print: Uncaught ReferenceError: arguments is not defined
   ```

**不应使用箭头函数的情况**

1. 对象方法

```
var cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}
```

调用`cat.jumps()`后，`cat.lives`并不会减1。这是因为this就没有被绑定到这个对象上，而是从父级作用域继承了`this`.

2. 具有动态context的回调函数

当context需要动态改变的时候，箭头函数通常不是正确的选择。比如在一个事件句柄中：

```
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');
});
```

如果我们点击按钮，我们会得到一个TypeError。这是因为this不是绑定到按钮，而是绑定到了它的父级。

3. 当使用箭头函数使你的代码可读性降低（难以知道这段代码将发生什么）的时候

**何时使用箭头函数**？

箭头函数最能发挥功力的场景就是需要this绑定到context，而非函数本身的时候。我还尤其喜欢在map或者reduce方法中使用箭头函数，这使得程序可读性更强。

#### 函数参数

**函数默认参数**

1. 方法一

   ```
   
   function example(a,b){ 
     var a = arguments[0] ? arguments[0] : 1;//设置参数a的默认值为1 
     var b = arguments[1] ? arguments[1] : 2;//设置参数b的默认值为2 
     return a+b; 
   } 
   //或
   
   function example(){ 
     var a = arguments[0] ? arguments[0] : 1;//设置第一个参数的默认值为1 
     var b = arguments[1] ? arguments[1] : 2;//设置第二个参数的默认值为2 
     return a+b; 
   } 
   ```

2. 方法二

   ```
   
   function example(name,age){ 
     name=name||'貂蝉'; 
     age=age||21; 
     alert('你好！我是'+name+'，今年'+age+'岁。'); 
   } 
   //这种写法的缺点在于：
   //如果参数y赋值了，但是对应的布尔值为false，则该赋值不起作用。如果在调用函数的时候，传入的y参数是一个空字符串，那么y就会被修改为默认值。
   //避免这个问题，需要先判断一下：
   //1.通过判断值是否等于undefined，
   //2.判断arguments.length是否为1.
   
   //或
   
   function example(name,age){ 
     if(!name){name='貂蝉';} 
     if(!age){age=21;} 
     alert('你好！我是'+name+'，今年'+age+'岁。'); 
   } 
   
   ```

3. 方法三

   ```
   
   function example(setting){ 
     var defaultSetting={ 
       name:'小红', 
       age:'30', 
       sex:'女', 
       phone:'100866', 
       QQ:'100866', 
       birthday:'1949.10.01'
     }; 
     $.extend(defaultSetting,settings); 
     var message='姓名：'+defaultSetting.name 
     +'，性别：'+defaultSetting.sex 
     +'，年龄：'+defaultSetting.age 
     +'，电话：'+defaultSetting.phone 
     +'，QQ：'+defaultSetting.QQ 
     +'，生日：'+defaultSetting.birthday 
     +'。'; 
     alert(message); 
   } 
   
   ```

4. 方法四：es6

   ```
   function log(x,y='world'){
       console.log(x,y);
   }
   log('hello'); //hello world
   ```

   与解构赋值默认值结合使用

   ```
   function m1({x=0,y=0} = {}){
       return [x,y];
   }
   function m2({x,y} = {x:0,y:0}){
       return [x,y];
   }
   m1({x:3});//[3,0]
   m2({x:3});//[3,undefined]
   m1({});//[0,0]
   m2({});//[undefined,undefined]
   ```

   通常情况下，定义了默认值的参数应该是函数的尾参数。因为这样比较容易看出，到底省略了哪些参数，如果非尾部的参数设置默认值，实际上这个参数是无法省略的。
   如果有默认值的参数都不是尾参数，这时，无法只省略该参数而不省略其后的参数，除非显示输入undefined。如果传入undefined，那么就会触发默认值，但是null没有这个效果。
   函数的length属性
   如果函数指定了默认值后，函数的length属性就不会包含有默认值的参数。这是因为length属性的含义是，该函数预期传入的参数个数，某个参数指定默认值之后，预期传入的参数个数就不包括
   这个参数了，同理，rest参数也不会计入length属性。
   函数参数默认值的类型
   （1）变量
   如果函数参数的默认值是一个变量，则该变量所处的作用域和其他变量的作用域规则相同，即是先前函数的作用域，然后再是全局作用域。
   （2）函数

   如果函数A的参数默认值是函数B，那么由于函数的作用域是其声明的时候所在的作用域，函数B的作用域就在全局作用域而不是函数A的作用域。

#### 模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`｀`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`
 
// 多行字符串，保证原格式
`In JavaScript this is
 not legal.`
 
console.log(`string text line 1
string text line 2`);
 
// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

上面代码中的模板字符串，都是用反引号表示。如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。

```
let greeting = `\`Yo\` World!`;
```

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

```
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

模板字符串中嵌入变量，需要将变量名写在`${}`之中。

```
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 传统写法为
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```

大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。

```
let x = 1;
let y = 2;
 
`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"
 
`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"
 
let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字符串之中还能调用函数。

```
function fn() {
  return "Hello World";
}
 
`foo ${fn()} bar`
// foo Hello World bar
```

如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的`toString`方法。

如果模板字符串中的变量没有声明，将报错。

```
// 变量place没有声明
let msg = `Hello, ${place}`;
// 报错
```

由于模板字符串的大括号内部，就是执行 JavaScript 代码，因此如果大括号内部是一个字符串，将会原样输出。

```
`Hello ${'World'}`
// "Hello World"
```

- 模板占位符中的代码可以是任意 JavaScript 表达式，所以函数调用、算数运算等这些都可以作为占位符使用，你甚至可以在一个模板字符串中嵌套另一个，我称之为模板套构（template inception）。
- 如果这两个值都不是字符串，可以按照常规将其转换为字符串。例如：如果 action 是一个对象，将会调用它的.toString() 方法将其转换为字符串值。
- 如果你需要在模板字符串中书写反撇号，你必须使用反斜杠将其转义：`\``等价于"`"。
- 同样地，如果你需要在模板字符串中引入字符 $ 和{。无论你要实现什么样的目标，你都需要用反斜杠转义每一个字符：`\$`和`\{`。

#### 标签模板

 紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串

```
{
	//alert(123);
	alert`1234`
}

{
	let obj = {
		name:"张三",
		age:"男"
	}
	function fn(a,b,c){
		console.log(a);//第一个参数是一个数组，
		//该数组的成员是模板字符串中那些没有变量替换的部分
		// (3) ["login ", "admin", "", raw: Array(3)]
		return `${a}===>${b}===>${c}`;
	}
	console.info(fn`login ${obj.name}admin${obj.age}`);//login ,admin,===>张三===>男
}
```

用户输入过滤：<https://github.com/cure53/DOMPurify>

```
function sanitize(strings,...values){
  const dirty= strings.reduce((prev,curr,i)=>`${prev}${curr}${values[i]||''}`,'')
  return DOMPurify.sanitize(dirty);
}

let msg=sanitize`${inputmsg}`;
```

#### [JS中对URL进行转码与解码](https://www.cnblogs.com/lvmylife/p/7595036.html)

**1. escape 和 unescape**

escape()不能直接用于URL编码，它的真正作用是返回一个字符的Unicode编码值。

采用unicode字符集对指定的字符串除0-255以外进行编码。所有的空格符、标点符号、特殊字符以及更多有联系非ASCII字符都将被转化成%xx格式的字符编码（xx等于该字符在字符集表里面的编码的16进制数字）。比如，空格符对应的编码是%20。
escape不编码字符有69个：*，+，-，.，/，@，_，0-9，a-z，A-Z。

escape()函数用于js对字符串进行编码**，**不常用。

```
　　var url = "http://localhost:8080/pro?a=1&b=张三&c=aaa";　　escape(url)  -->   http%3A//localhost%3A8080/pro%3Fa%3D1%26b%3D%u5F20%u4E09%26c%3Daaa    
```

**2. encodeURI 和 decodeURI**

把URI字符串采用UTF-8编码格式转化成escape各式的字符串。
encodeURI不编码字符有82个：!，#，$，&，'，(，)，*，+，,，-，.，/，:，;，=，?，@，_，~，0-9，a-z，A-Z

 encodeURI()用于整个url编码

```
　　var url = "http://localhost:8080/pro?a=1&b=张三&c=aaa";　　encodeURI(url)  -->   http://localhost:8080/pro?a=1&b=%E5%BC%A0%E4%B8%89&c=aaa 
```

**3. encodeURIComponent 和 decodeURIComponent**

与encodeURI()的区别是，它用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。

因此，"; / ? : @ & = + $ , #"，这些在encodeURI()中不被编码的符号，在encodeURIComponent()中统统会被编码。至于具体的编码方法，两者是一样。把URI字符串采用UTF-8编码格式转化成escape格式的字符串。

encodeURIComponent() 用于参数的传递，参数包含特殊字符可能会造成间断。

例1

```
var url = "http://localhost:8080/pro?a=1&b=张三&c=aaa";
　　encodeURIComponent(url) --> http%3A%2F%2Flocalhost%3A8080%2Fpro%3Fa%3D1%26b%3D%E5%BC%A0%E4%B8%89%26c%3Daaa
```

例2

```
var url = "http://localhost:8080/pp?a=1&b="+ paramUrl,

　　var paramUrl = "http://localhost:8080/aa?a=1&b=2&c=3";
　　应该使用encodeURIComponent()进行转码　　
　　encodeURIComponent(paramUrl) --> http://localhost:8080/pp?a=1&b=http%3A%2F%2Flocalhost%3A8080%2Faa%3Fa%3D1%26b%3D2%23%26c%3D3
```

文章参考: 

http://www.cnblogs.com/douJiangYouTiao888/p/6473874.html

https://segmentfault.com/a/1190000010735689