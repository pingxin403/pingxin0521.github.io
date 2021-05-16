---
title: Node.JS 入门
date: 2019-06-07 14:18:59
tags:
 - Node.JS
 - 后端
categories:
 - 后端
 - Node.JS
---


#### 回调

回调是一种异步相当于一个函数。回调函数被调用在完成既定任务。Node大量使用了回调。Node所有的API写的都是支持回调的这样一种方式。例如，一个函数读取一个文件可能开始读取文件，并立即返回控制到执行环境 使得下一个指令可以马上被执行。一旦文件 I/O 完成，它会调用回调函数，同时传递回调函数，该文件作为参数的内容。因此不会有堵塞或等待文件I/O。这使得Node.js高度可扩展，因此可以处理大量的请求，而无需等待任何函数来返回结果。

<!--more-->

**阻塞代码示例**

创建一个 txt 文件：test.txt

```
hanyunpeng0521.github.io
```

创建一个名为test.js 的js文件

```
var fs = require("fs");
var data = fs.readFileSync('test.txt');
console.log(data.toString());
console.log("Program Ended");
```

现在运行 test.js 看到的结果

```bash
$ node test.js
hanyunpeng0521.github.io

Program Ended
```

**非阻塞代码示例**

创建一个 txt 文件：test.txt

```
hanyunpeng0521.github.io
```

创建一个名为test.js 的js文件

```
var fs = require("fs");

fs.readFile('test.txt', function (err, data) {
    if (err) return console.error(err);
    console.log(data.toString());
});
console.log("Program Ended");
```

现在运行 test.js 看到的结果

```
$ node test.js
Program Ended
hanyunpeng0521.github.io

```

#### 事件循环

Node JS是单线程应用程序，但它通过事件和回调的概念，支持并发。NodeJS的每一个API都是异步的，作为一个单独的线程，它使用异步函数调用来维护并发。Node使用观察者模式。Node线程保持一个事件循环，每当任何任务完成后得到结果，它触发通知事件侦听函数来执行相应的事件。

Node.js使用大量事件，这也是为什么Node.js相对于其他类似技术比较快的原因之一。当Node启动其服务器，就可以简单地初始化其变量，声明函数，然后等待事件的发生。

 	虽然事件似乎类似于回调。不同之处在于当回调函数被调用异步函数返回结果，其中的事件处理工作在观察者模式。监听事件函数作为观察者。每当一个事件被解雇，其监听函数开始执行。Node.js有多个内置的事件。 主要扮演者是 EventEmitter，可使用以下语法导入。

```
//import events module
var events = require('events');
//create an eventEmitter object
var eventEmitter = new events.EventEmitter();
```

**示例**

创建一个 js 文件名为 test.js

File: test.js

```
//import events module
var events = require('events');
//create an eventEmitter object
var eventEmitter = new events.EventEmitter();

//create a function connected which is to be executed 
//when 'connection' event occurs
var connected = function connected() {
   console.log('connection succesful.');
  
   // fire the data_received event 
   eventEmitter.emit('data_received.');
}

// bind the connection event with the connected function
eventEmitter.on('connection', connected);
 
// bind the data_received event with the anonymous function
eventEmitter.on('data_received', function(){
   console.log('data received succesfully.');
});

// fire the connection event 
eventEmitter.emit('connection');

console.log("Program Ended.");
```

现在运行test.js看到的结果：

```
node test.js
connection succesful.
Program Ended.
```

#### Node应用程序如何工作

在Node 应用，任何异步函数接受回调作为最后的参数，并回调函数接受错误作为第一个参数。我们再看一下前面的例子。

```
var fs = require("fs");

fs.readFile('test.txt', function (err, data) {
    if (err){
	   console.log(err.stack);
	   return;
	}
    console.log(data.toString());
});
console.log("Program Ended");
```

这里fs.readFile是一个异步函数，其目的是用于读取文件。 如果在文件的读取发生了错误，则err 对象将包含相应的错误，否则data将包含该文件的内容。readFile通过err和data到回调函数后，文件读取操作完成。

#### 事件发射器

EventEmitter类在于事件的模块。它通过通俗易懂的语法如下：

```
//import events module
var events = require('events');
//create an eventEmitter object
var eventEmitter = new events.EventEmitter();
```

当EventEmitter实例出现任何错误，它会发出“error”事件。当新的监听器添加，'newListener'事件被触发，当一个监听器被删除，'removeListener“事件被触发。

 EventEmitter提供多种属性，如：on和emit。on属性用于绑定与该事件的函数，而 emit 用于触发一个事件。

**方法**

![](https://i.loli.net/2019/07/15/5d2c38a32848990620.png)

**类方法**

![](https://i.loli.net/2019/07/15/5d2c38a31848060679.png)

**事件**

![](https://i.loli.net/2019/07/15/5d2c38a340ffb89203.png)

示例

File: test.js

```
var events = require('events');
var eventEmitter = new events.EventEmitter();

//listener #1
var listner1 = function listner1() {
   console.log('listner1 executed.');
}

//listener #2
var listner2 = function listner2() {
  console.log('listner2 executed.');
}

// bind the connection event with the listner1 function
eventEmitter.addListener('connection', listner1);

// bind the connection event with the listner2 function
eventEmitter.on('connection', listner2);

var eventListeners = require('events').EventEmitter.listenerCount(eventEmitter,'connection');
console.log(eventListeners + " Listner(s) listening to connection event");

// fire the connection event 
eventEmitter.emit('connection');

// remove the binding of listner1 function
eventEmitter.removeListener('connection', listner1);
console.log("Listner1 will not listen now.");

// fire the connection event 
eventEmitter.emit('connection');

eventListeners = require('events').EventEmitter.listenerCount(eventEmitter,'connection');
console.log(eventListeners + " Listner(s) listening to connection event");

console.log("Program Ended.");
```

现在运行test.js看到的结果：

```
$ node test.js
2 Listner(s) listening to connection event
listner1 executed.
listner2 executed.
Listner1 will not listen now.
listner2 executed.
1 Listner(s) listening to connection event
Program Ended.
```

#### 缓冲模块

缓冲器模块可以被用来创建缓冲区和SlowBuffer类。缓冲模块可以使用以下语法导入。 

```
var buffer = require("buffer")
```

**Buffer 类**

Buffer类是一个全局类，可以在应用程序，无需导入缓冲模块进行访问。缓冲区是一种整数数组并对应于原始存储器V8堆以外分配。 缓冲区不能调整大小。

![](https://i.loli.net/2019/07/15/5d2c3a81e06bd61002.png)

示例

File: test.js

```
//create a buffer
var buffer = new Buffer(26);
console.log("buffer length: " + buffer.length);

//write to buffer
var data = "YiiBai.com";
buffer.write(data);
console.log(data + ": " + data.length + " characters, " + Buffer.byteLength(data, 'utf8') + " bytes");

//slicing a buffer
var buffer1 = buffer.slice(0,14);
console.log("buffer1 length: " + buffer1.length);
console.log("buffer1 content: " + buffer1.toString());

//modify buffer by indexes
for (var i = 0 ; i < 26 ; i++) {
  buffer[i] = i + 97; // 97 is ASCII a
}
console.log("buffer content: " + buffer.toString('ascii'));

var buffer2 = new Buffer(4);

buffer2[0] = 0x3;
buffer2[1] = 0x4;
buffer2[2] = 0x23;
buffer2[3] = 0x42;

//reading from buffer
console.log(buffer2.readUInt16BE(0));
console.log(buffer2.readUInt16LE(0));
console.log(buffer2.readUInt16BE(1));
console.log(buffer2.readUInt16LE(1));
console.log(buffer2.readUInt16BE(2));
console.log(buffer2.readUInt16LE(2));


var buffer3 = new Buffer(4);
buffer3.writeUInt16BE(0xdead, 0);
buffer3.writeUInt16BE(0xbeef, 2);

console.log(buffer3);

buffer3.writeUInt16LE(0xdead, 0);
buffer3.writeUInt16LE(0xbeef, 2);

console.log(buffer3);
//convert to a JSON Object
var json = buffer3.toJSON();
console.log("JSON Representation : ");
console.log(json);

//Get a buffer from JSON Object
var buffer6 = new Buffer(json);
console.log(buffer6);

//copy a buffer
var buffer4 = new Buffer(26);
buffer.copy(buffer4);
console.log("buffer4 content: " + buffer4.toString());

//concatenate a buffer
var buffer5 = Buffer.concat([buffer,buffer4]);
console.log("buffer5 content: " + buffer5.toString());
```

现在运行test.js看到的结果：

```
$ node test.js
buffer length: 26
YiiBai.com: 18 characters, 18 bytes
buffer1 length: 14
buffer1 content: YiiBai
buffer content: abcdefghijklmnopqrstuvwxyz
772
1027
1059
8964
9026
16931
<Buffer de ad be ef>
<Buffer ad de ef be>
buffer4 content: abcdefghijklmnopqrstuvwxyz
buffer5 content: abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz

```

#### 数据流

数据流是从源数据读取或写入数据到目标对象以。在节点中，有四种类型的流：

-  			**Readable** - 数据流，其是用于读操作
-  			**Writable** - 数据流，用在写操作
-  			**Duplex** - 数据流，其可以用于读取和写入操作
-  			**Transform** - 双相类型流，输出基于输入进行计算

 	每种类型的流是EventEmitter，并且一次引发几个事件。例如，一些常用的事件是：

-  			**data** - 当有数据可读取时触发此事件
-  			**end** - 当没有更多的数据读取时触发此事件
-  			**error** - 当有错误或接收数据写入时触发此事件
-  			**finish** - 当所有数据已刷新到底层系统时触发此事件

**从数据流中读取**

创建一个txt文件名为test.txt

```
hanyunpeng0521.github.io
```

创建一个文件 test.js:

```
var fs = require("fs");
var data = '';
//create a readable stream
var readerStream = fs.createReadStream('test.txt');

//set the encoding to be utf8. 
readerStream.setEncoding('UTF8');

//handle stream events
readerStream.on('data', function(chunk) {
   data += chunk;
});

readerStream.on('end',function(){
   console.log(data);
});

readerStream.on('error', function(err){
   console.log(err.stack);
});
console.log("Program Ended");
```

现在运行 test.js 看到的结果如下：

```bash
$ node test04.js 
Program Ended
pingxin0521.github.io


```

**写入到流**

更新文件 test.js 内容

```
var fs = require("fs");
var data = 'Yiibai.Com';
//create a writable stream
var writerStream = fs.createWriteStream('test1.txt');

//write the data to stream
//set the encoding to be utf8. 
writerStream.write(data,'UTF8');

//mark the end of file
writerStream.end();

//handle stream events
writerStream.on('finish', function() {
    console.log("Write completed.");
});

writerStream.on('error', function(err){
   console.log(err.stack);
});
console.log("Program Ended");
```

现在运行 test.js 看到的结果：

```
$ node test.js
Program Ended
Write completed.
```

**管道流**

管道是一种机制，一个流的输出连接到另一个流(作为另外一个流的输入)。它通常用来从一个流中获取数据，并通过该流输出到另一个流。管道没有对操作限制。考虑上面的例子中，在这里我们使用readerStream 读取test.txt的内容，并使用 writerStream 写入 test1.txt。现在，我们将用管道来简化操作，或者从一个文件中读取并写入到另一个文件。

更新 test.js 文件的内容

```
var fs = require("fs");

//create a readable stream
var readerStream = fs.createReadStream('test.txt');

//create a writable stream
var writerStream = fs.createWriteStream('test2.txt');

//pipe the read and write operations
//read test.txt and write data to test2.txt
readerStream.pipe(writerStream);

console.log("Program Ended");
```

现在运行test.js看到的结果

```
$ node test.js
Program Ended
```

打开文件 test2.txt

```
pingxin0521.github.io
```

#### Node.js模块系统

为了让Node.js的文件可以相互调用，Node.js提供了一个简单的模块系统。

模块是Node.js 应用程序的基本组成部分，文件和模块是一一对应的。换言之，一个 Node.js 文件就是一个模块，这个文件可能是JavaScript 代码、JSON 或者编译过的C/C++ 扩展。

##### 创建模块

在 Node.js 中，创建一个模块非常简单，如下我们创建一个 **main.js** 文件，代码如下:

```
var hello = require('./hello');
hello.world();
```

以上实例中，代码 require('./hello') 引入了当前目录下的 hello.js 文件（./ 为当前目录，node.js 默认后缀为 js）。

Node.js 提供了 exports 和 require 两个对象，其中 exports 是模块公开的接口，require 用于从外部获取一个模块的接口，即所获取模块的 exports 对象。

接下来我们就来创建 hello.js 文件，代码如下：

```
exports.world = function() {
  console.log('Hello World');
}
```

在以上示例中，hello.js 通过 exports 对象把 world 作为模块的访问接口，在 main.js 中通过 require('./hello') 加载这个模块，然后就可以直接访 问 hello.js 中 exports 对象的成员函数了。

有时候我们只是想把一个对象封装到模块中，格式如下：

```
module.exports = function() {
  // ...
}
```

例如:

```
//hello.js 
function Hello() { 
    var name; 
    this.setName = function(thyName) { 
        name = thyName; 
    }; 
    this.sayHello = function() { 
        console.log('Hello ' + name); 
    }; 
}; 
module.exports = Hello;
```

这样就可以直接获得这个对象了：

```
//main.js 
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('pingxin'); 
hello.sayHello(); 
```

模块接口的唯一变化是使用 module.exports = Hello 代替了exports.world = function(){}。 在外部引用该模块时，其接口对象就是要输出的 Hello 对象本身，而不是原先的 exports。

##### 服务端的模块放在哪里

也许你已经注意到，我们已经在代码中使用了模块了。像这样：

```
var http = require("http");
...
http.createServer(...);
```

Node.js 中自带了一个叫做 **http** 的模块，我们在我们的代码中请求它并把返回值赋给一个本地变量。

这把我们的本地变量变成了一个拥有所有 http 模块所提供的公共方法的对象。

Node.js 的 require 方法中的文件查找策略如下：

由于 Node.js 中存在 4 类模块（原生模块和3种文件模块），尽管 require 方法极其简单，但是内部的加载却是十分复杂的，其加载优先级也各自不同。如下图所示：

![1.jpg](https://i.loli.net/2019/07/31/5d415e6ab3bd688510.jpg)

**从文件模块缓存中加载**

尽管原生模块与文件模块的优先级不同，但是都会优先从文件模块的缓存中加载已经存在的模块。

**从原生模块加载**

原生模块的优先级仅次于文件模块缓存的优先级。require 方法在解析文件名之后，优先检查模块是否在原生模块列表中。以http模块为例，尽管在目录下存在一个 http/http.js/http.node/http.json 文件，require("http") 都不会从这些文件中加载，而是从原生模块中加载。

原生模块也有一个缓存区，同样也是优先从缓存区加载。如果缓存区没有被加载过，则调用原生模块的加载方式进行加载和执行。

**从文件加载**

当文件模块缓存中不存在，而且不是原生模块的时候，Node.js 会解析 require 方法传入的参数，并从文件系统中加载实际的文件，加载过程中的包装和编译细节在前一节中已经介绍过，这里我们将详细描述查找文件模块的过程，其中，也有一些细节值得知晓。

require方法接受以下几种参数的传递：

- http、fs、path等，原生模块。
- ./mod或../mod，相对路径的文件模块。
- /pathtomodule/mod，绝对路径的文件模块。
- mod，非原生模块的文件模块。

在路径 Y 下执行 require(X) 语句执行顺序：

```
1. 如果 X 是内置模块
   a. 返回内置模块
   b. 停止执行
2. 如果 X 以 '/' 开头
   a. 设置 Y 为文件根路径
3. 如果 X 以 './' 或 '/' or '../' 开头
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
4. LOAD_NODE_MODULES(X, dirname(Y))
5. 抛出异常 "not found"

LOAD_AS_FILE(X)
1. 如果 X 是一个文件, 将 X 作为 JavaScript 文本载入并停止执行。
2. 如果 X.js 是一个文件, 将 X.js 作为 JavaScript 文本载入并停止执行。
3. 如果 X.json 是一个文件, 解析 X.json 为 JavaScript 对象并停止执行。
4. 如果 X.node 是一个文件, 将 X.node 作为二进制插件载入并停止执行。

LOAD_INDEX(X)
1. 如果 X/index.js 是一个文件,  将 X/index.js 作为 JavaScript 文本载入并停止执行。
2. 如果 X/index.json 是一个文件, 解析 X/index.json 为 JavaScript 对象并停止执行。
3. 如果 X/index.node 是一个文件,  将 X/index.node 作为二进制插件载入并停止执行。

LOAD_AS_DIRECTORY(X)
1. 如果 X/package.json 是一个文件,
   a. 解析 X/package.json, 并查找 "main" 字段。
   b. let M = X + (json main 字段)
   c. LOAD_AS_FILE(M)
   d. LOAD_INDEX(M)
2. LOAD_INDEX(X)

LOAD_NODE_MODULES(X, START)
1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   b. DIR = path join(PARTS[0 .. I] + "node_modules")
   c. DIRS = DIRS + DIR
   d. let I = I - 1
5. return DIRS
```

> **exports 和 module.exports 的使用**
>
> 如果要对外暴露属性或方法，就用 **exports** 就行，要暴露对象(类似class，包含了很多属性和方法)，就用 **module.exports**。

#### Node.js 函数

在JavaScript中，一个函数可以作为另一个函数的参数。我们可以先定义一个函数，然后传递，也可以在传递参数的地方直接定义函数。

Node.js中函数的使用与Javascript类似，举例来说，你可以这样做：

```
function say(word) {
  console.log(word);
}

function execute(someFunction, value) {
  someFunction(value);
}

execute(say, "Hello");
```

以上代码中，我们把 say 函数作为execute函数的第一个变量进行了传递。这里传递的不是 say 的返回值，而是 say 本身！

这样一来， say 就变成了execute 中的本地变量 someFunction ，execute可以通过调用 someFunction() （带括号的形式）来使用 say 函数。

当然，因为 say 有一个变量， execute 在调用 someFunction 时可以传递这样一个变量。

**匿名函数**

我们可以把一个函数作为变量传递。但是我们不一定要绕这个"先定义，再传递"的圈子，我们可以直接在另一个函数的括号中定义和传递这个函数：

```
function execute(someFunction, value) {
  someFunction(value);
}

execute(function(word){ console.log(word) }, "Hello");
```

我们在 execute 接受第一个参数的地方直接定义了我们准备传递给 execute 的函数。

用这种方式，我们甚至不用给这个函数起名字，这也是为什么它被叫做匿名函数 。

**函数传递是如何让HTTP服务器工作的**

带着这些知识，我们再来看看我们简约而不简单的HTTP服务器：

```
var http = require("http");

http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello World");
  response.end();
}).listen(8888);
```

现在它看上去应该清晰了很多：我们向 createServer 函数传递了一个匿名函数。

用这样的代码也可以达到同样的目的：

```
var http = require("http");

function onRequest(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello World");
  response.end();
}

http.createServer(onRequest).listen(8888);
```

使用浏览器打开<http://127.0.0.1:8888>，可以看到“Hello World”

#### Node.js 路由

我们要为路由提供请求的 URL 和其他需要的 GET 及 POST 参数，随后路由需要根据这些数据来执行相应的代码。

因此，我们需要查看 HTTP 请求，从中提取出请求的 URL 以及 GET/POST 参数。这一功能应当属于路由还是服务器（甚至作为一个模块自身的功能）确实值得探讨，但这里暂定其为我们的HTTP服务器的功能。

我们需要的所有数据都会包含在 request 对象中，该对象作为 onRequest() 回调函数的第一个参数传递。但是为了解析这些数据，我们需要额外的 Node.JS 模块，它们分别是 url 和 querystring 模块。

```
                   url.parse(string).query
                                           |
           url.parse(string).pathname      |
                       |                   |
                       |                   |
                     ------ -------------------
http://localhost:8888/start?foo=bar&hello=world
                                ---       -----
                                 |          |
                                 |          |
              querystring.parse(queryString)["foo"]    |
                                            |
                         querystring.parse(queryString)["hello"]
```

当然我们也可以用 querystring 模块来解析 POST 请求体中的参数，稍后会有演示。

现在我们来给 onRequest() 函数加上一些逻辑，用来找出浏览器请求的 URL 路径：

```
//server.js 
var http = require("http");
var url = require("url");
 
function start() {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }
 
  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}
 
exports.start = start;
```

好了，我们的应用现在可以通过请求的 URL 路径来区别不同请求了--这使我们得以使用路由（还未完成）来将请求以 URL 路径为基准映射到处理程序上。

在我们所要构建的应用中，这意味着来自 /start 和 /upload 的请求可以使用不同的代码来处理。稍后我们将看到这些内容是如何整合到一起的。

现在我们可以来编写路由了，建立一个名为 **router.js** 的文件，添加以下内容：

```js
function route(pathname) {
  console.log("About to route a request for " + pathname);
}
 
exports.route = route;
```

如你所见，这段代码什么也没干，不过对于现在来说这是应该的。在添加更多的逻辑以前，我们先来看看如何把路由和服务器整合起来。

我们的服务器应当知道路由的存在并加以有效利用。我们当然可以通过硬编码的方式将这一依赖项绑定到服务器上，但是其它语言的编程经验告诉我们这会是一件非常痛苦的事，因此我们将使用依赖注入的方式较松散地添加路由模块。

首先，我们来扩展一下服务器的 start() 函数，以便将路由函数作为参数传递过去，**server.js** 文件代码如下

```js
var http = require("http");
var url = require("url");
 
function start(route) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
 
    route(pathname);
 
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }
 
  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}
 
exports.start = start;
```

同时，我们会相应扩展 index.js，使得路由函数可以被注入到服务器中：

```js
var server = require("./server");
var router = require("./router");
 
server.start(router.route);
```

在这里，我们传递的函数依旧什么也没做。

如果现在启动应用（node index.js，始终记得这个命令行），随后请求一个URL，你将会看到应用输出相应的信息，这表明我们的HTTP服务器已经在使用路由模块了，并会将请求的路径传递给路由：

```bash
$ node index.js
Server has started.
```

以上输出已经去掉了比较烦人的 /favicon.ico 请求相关的部分。

浏览器访问 <http://127.0.0.1:8888/>

#### Node.js 全局对象

JavaScript 中有一个特殊的对象，称为全局对象（Global Object），它及其所有属性都可以在程序的任何地方访问，即全局变量。

在浏览器 JavaScript 中，通常 window 是全局对象， 而 Node.js 中的全局对象是 global，所有全局变量（除了 global 本身以外）都是 global 对象的属性。

在 Node.js 我们可以直接访问到 global 的属性，而不需要在应用中包含它。

##### 全局对象与全局变量

global 最根本的作用是作为全局变量的宿主。按照 ECMAScript 的定义，满足以下条 件的变量是全局变量：

- 在最外层定义的变量；
- 全局对象的属性；
- 隐式定义的变量（未定义直接赋值的变量）。

当你定义一个全局变量时，这个变量同时也会成为全局对象的属性，反之亦然。需要注 意的是，在 Node.js 中你不可能在最外层定义变量，因为所有用户代码都是属于当前模块的， 而模块本身不是最外层上下文。

**注意：** 最好不要使用 var 定义变量以避免引入全局变量，因为全局变量会污染命名空间，提高代码的耦合风险。

1. __filename 表示当前正在执行的脚本的文件名。它将输出文件所在位置的绝对路径，且和命令行参数所指定的文件名不一定相同。 如果在模块中，返回的值是模块文件的路径。

   创建文件 main.js ，代码如下所示：

   ```
   // 输出全局变量 __filename 的值
   console.log( __filename );
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   ```

2. __dirname 表示当前执行脚本所在的目录。

   创建文件 main.js ，代码如下所示：

   ```
   // 输出全局变量 __dirname 的值
   console.log( __dirname );
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   ```

3. setTimeout(cb, ms) 全局函数在指定的毫秒(ms)数后执行指定函数(cb)。：setTimeout() 只执行一次指定函数。

   返回一个代表定时器的句柄值。

   创建文件 main.js ，代码如下所示：

   ```
   function printHello(){
      console.log( "Hello, World!");
   }
   // 两秒后执行以上函数
   setTimeout(printHello, 2000);
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   Hello, World!
   ```

4. clearTimeout( t ) 全局函数用于停止一个之前通过 setTimeout() 创建的定时器。 参数 t 是通过 setTimeout() 函数创建的定时器。

   创建文件 main.js ，代码如下所示：

   ```
   function printHello(){
      console.log( "Hello, World!");
   }
   // 两秒后执行以上函数
   var t = setTimeout(printHello, 2000);
   
   // 清除定时器
   clearTimeout(t);
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   ```

5. setInterval(cb, ms) 全局函数在指定的毫秒(ms)数后执行指定函数(cb)。

   返回一个代表定时器的句柄值。可以使用 clearInterval(t) 函数来清除定时器。

   setInterval() 方法会不停地调用函数，直到 clearInterval() 被调用或窗口被关闭。

   创建文件 main.js ，代码如下所示：

   ```
   function printHello(){
      console.log( "Hello, World!");
   }
   // 两秒后执行以上函数
   setInterval(printHello, 2000);
   //var timer1 = setInterval("yourFunction1",时间);
   //clearInterval(timer1);
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   ```

   Hello, World! Hello, World! Hello, World! Hello, World! Hello, World! ……

   以上程序每隔两秒就会输出一次"Hello, World!"，且会永久执行下去，直到你按下 **ctrl + c** 按钮。

6. console 用于提供控制台标准输出，它是由 Internet Explorer 的 JScript 引擎提供的调试工具，后来逐渐成为浏览器的实施标准。

   Node.js 沿用了这个标准，提供与习惯行为一致的 console 对象，用于向标准输出流（stdout）或标准错误流（stderr）输出字符。

   以下为 console 对象的方法:

   | 序号 | 方法 & 描述                                                  |
   | :--- | :----------------------------------------------------------- |
   | 1    | `console.log([data][, ...]) 向标准输出流打印字符并以换行符结束。该方法接收若干 个参数，如果只有一个参数，则输出这个参数的字符串形式。如果有多个参数，则 以类似于C 语言 printf() 命令的格式输出。` |
   | 2    | `console.info([data][, ...]) 该命令的作用是返回信息性消息，这个命令与console.log差别并不大，除了在chrome中只会输出文字外，其余的会显示一个蓝色的惊叹号。` |
   | 3    | `console.error([data][, ...]) 输出错误消息的。控制台在出现错误时会显示是红色的叉子。` |
   | 4    | `console.warn([data][, ...]) 输出警告消息。控制台出现有黄色的惊叹号。` |
   | 5    | **console.dir(obj[, options])** 用来对一个对象进行检查（inspect），并以易于阅读和打印的格式显示。 |
   | 6    | **console.time(label)** 输出时间，表示计时开始。             |
   | 7    | **console.timeEnd(label)** 结束时间，表示计时结束。          |
   | 8    | **console.trace(message[, ...])** 当前执行的代码在堆栈中的调用路径，这个测试函数运行很有帮助，只要给想测试的函数里面加入 console.trace 就行了。 |
   | 9    | `console.assert(value[, message][, ...]) 用于判断某个表达式或变量是否为真，接收两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为false，才会输出第二个参数，否则不会有任何结果。` |

   console.log()：向标准输出流打印字符并以换行符结束。

   console.log 接收若干 个参数，如果只有一个参数，则输出这个参数的字符串形式。如果有多个参数，则 以类似于C 语言 printf() 命令的格式输出。

   第一个参数是一个字符串，如果没有 参数，只打印一个换行。

   ```
   console.log('Hello world'); 
   console.log('byvoid%diovyb'); 
   console.log('byvoid%diovyb', 1991); 
   ```

   运行结果为：

   ```
   Hello world 
   byvoid%diovyb 
   byvoid1991iovyb 
   ```

   - console.error()：与console.log() 用法相同，只是向标准错误流输出。
   - console.trace()：向标准错误流输出当前的调用栈。

   ```
   console.trace();
   ```

   运行结果为：

   ```
   Trace: 
   at Object.<anonymous> (/home/byvoid/consoletrace.js:1:71) 
   at Module._compile (module.js:441:26) 
   at Object..js (module.js:459:10) 
   at Module.load (module.js:348:31) 
   at Function._load (module.js:308:12) 
   at Array.0 (module.js:479:10) 
   at EventEmitter._tickCallback (node.js:192:40)
   ```

   创建文件 main.js ，代码如下所示：

   ```
   console.info("程序开始执行：");
   
   var counter = 10;
   console.log("计数: %d", counter);
   
   console.time("获取数据");
   //
   // 执行一些代码
   // 
   console.timeEnd('获取数据');
   
   console.info("程序执行完毕。")
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   程序开始执行：
   计数: 10
   获取数据: 0ms
   程序执行完毕
   ```

7. process 是一个全局变量，即 global 对象的属性。

   它用于描述当前Node.js 进程状态的对象，提供了一个与操作系统的简单接口。通常在你写本地命令行程序的时候，少不了要 和它打交道。下面将会介绍 process 对象的一些最常用的成员方法。

   | 序号 | 事件 & 描述                                                  |
   | :--- | :----------------------------------------------------------- |
   | 1    | **exit** 当进程准备退出时触发。                              |
   | 2    | **beforeExit** 当 node 清空事件循环，并且没有其他安排时触发这个事件。通常来说，当没有进程安排时 node 退出，但是 'beforeExit' 的监听器可以异步调用，这样 node 就会继续执行。 |
   | 3    | **uncaughtException** 当一个异常冒泡回到事件循环，触发这个事件。如果给异常添加了监视器，默认的操作（打印堆栈跟踪信息并退出）就不会发生。 |
   | 4    | **Signal 事件** 当进程接收到信号时就触发。信号列表详见标准的 POSIX 信号名，如 SIGINT、SIGUSR1 等。 |

   创建文件 main.js ，代码如下所示：

   ```
   process.on('exit', function(code) {
   
     // 以下代码永远不会执行
     setTimeout(function() {
       console.log("该代码不会执行");
     }, 0);
     
     console.log('退出码为:', code);
   });
   console.log("程序执行结束");
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   程序执行结束
   退出码为: 0
   ```

   退出状态码

   | 状态码 | 名称 & 描述                                                  |
   | :----- | :----------------------------------------------------------- |
   | 1      | **Uncaught Fatal Exception** 有未捕获异常，并且没有被域或 uncaughtException 处理函数处理。 |
   | 2      | **Unused** 保留                                              |
   | 3      | **Internal JavaScript Parse Error** JavaScript的源码启动 Node 进程时引起解析错误。非常罕见，仅会在开发 Node 时才会有。 |
   | 4      | **Internal JavaScript Evaluation Failure** JavaScript 的源码启动 Node 进程，评估时返回函数失败。非常罕见，仅会在开发 Node 时才会有。 |
   | 5      | **Fatal Error** V8 里致命的不可恢复的错误。通常会打印到 stderr ，内容为： FATAL ERROR |
   | 6      | **Non-function Internal Exception Handler** 未捕获异常，内部异常处理函数不知为何设置为on-function，并且不能被调用。 |
   | 7      | **Internal Exception Handler Run-Time Failure** 未捕获的异常， 并且异常处理函数处理时自己抛出了异常。例如，如果 process.on('uncaughtException') 或 domain.on('error') 抛出了异常。 |
   | 8      | **Unused** 保留                                              |
   | 9      | **Invalid Argument** 可能是给了未知的参数，或者给的参数没有值。 |
   | 10     | **Internal JavaScript Run-Time Failure** JavaScript的源码启动 Node 进程时抛出错误，非常罕见，仅会在开发 Node 时才会有。 |
   | 12     | **Invalid Debug Argument**  设置了参数--debug 和/或 --debug-brk，但是选择了错误端口。 |
   | 128    | **Signal Exits** 如果 Node 接收到致命信号，比如SIGKILL 或 SIGHUP，那么退出代码就是128 加信号代码。这是标准的 Unix 做法，退出信号代码放在高位。 |

9. Process 提供了很多有用的属性，便于我们更好的控制系统的交互：

   | 序号. | 属性 & 描述                                                  |
   | :---- | :----------------------------------------------------------- |
   | 1     | **stdout** 标准输出流。                                      |
   | 2     | **stderr** 标准错误流。                                      |
   | 3     | **stdin** 标准输入流。                                       |
   | 4     | **argv** argv 属性返回一个数组，由命令行执行脚本时的各个参数组成。它的第一个成员总是node，第二个成员是脚本文件名，其余成员是脚本文件的参数。 |
   | 5     | **execPath** 返回执行当前脚本的 Node 二进制文件的绝对路径。  |
   | 6     | **execArgv** 返回一个数组，成员是命令行下执行脚本时，在Node可执行文件与脚本文件之间的命令行参数。 |
   | 7     | **env** 返回一个对象，成员为当前 shell 的环境变量            |
   | 8     | **exitCode** 进程退出时的代码，如果进程优通过 process.exit() 退出，不需要指定退出码。 |
   | 9     | **version** Node 的版本，比如v0.10.18。                      |
   | 10    | **versions** 一个属性，包含了 node 的版本和依赖.             |
   | 11    | **config** 一个包含用来编译当前 node 执行文件的 javascript 配置选项的对象。它与运行 ./configure 脚本生成的 "config.gypi" 文件相同。 |
   | 12    | **pid** 当前进程的进程号。                                   |
   | 13    | **title** 进程名，默认值为"node"，可以自定义该值。           |
   | 14    | **arch** 当前 CPU 的架构：'arm'、'ia32' 或者 'x64'。         |
   | 15    | **platform** 运行程序所在的平台系统 'darwin', 'freebsd', 'linux', 'sunos' 或 'win32' |
   | 16    | **mainModule** require.main 的备选方法。不同点，如果主模块在运行时改变，require.main可能会继续返回老的模块。可以认为，这两者引用了同一个模块。 |

   创建文件 main.js ，代码如下所示：

   ```
   // 输出到终端
   process.stdout.write("Hello World!" + "\n");
   
   // 通过参数读取
   process.argv.forEach(function(val, index, array) {
      console.log(index + ': ' + val);
   });
   
   // 获取执行路径
   console.log(process.execPath);
   
   
   // 平台信息
   console.log(process.platform);
   ```

   执行 main.js 文件，代码如下所示:

   ```
   $ node main.js
   Hello World!
   0: node
   1: /web/www/node/main.js
   /usr/local/node/0.10.36/bin/node
   darwin
   ```

   Process 提供了很多有用的方法，便于我们更好的控制系统的交互：

   | 序号 | 方法 & 描述                                                  |
   | :--- | :----------------------------------------------------------- |
   | 1    | **abort()** 这将导致 node 触发 abort 事件。会让 node 退出并生成一个核心文件。 |
   | 2    | **chdir(directory)** 改变当前工作进程的目录，如果操作失败抛出异常。 |
   | 3    | **cwd()** 返回当前进程的工作目录                             |
   | 4    | **exit([code])** 使用指定的 code 结束进程。如果忽略，将会使用 code 0。 |
   | 5    | **getgid()** 获取进程的群组标识（参见 getgid(2)）。获取到得时群组的数字 id，而不是名字。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 6    | **setgid(id)** 设置进程的群组标识（参见 setgid(2)）。可以接收数字 ID 或者群组名。如果指定了群组名，会阻塞等待解析为数字 ID 。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 7    | **getuid()** 获取进程的用户标识(参见 getuid(2))。这是数字的用户 id，不是用户名。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 8    | **setuid(id)** 设置进程的用户标识（参见setuid(2)）。接收数字 ID或字符串名字。果指定了群组名，会阻塞等待解析为数字 ID 。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 9    | **getgroups()** 返回进程的群组 iD 数组。POSIX 系统没有保证一定有，但是 node.js 保证有。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 10   | **setgroups(groups)** 设置进程的群组 ID。这是授权操作，所以你需要有 root 权限，或者有 CAP_SETGID 能力。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 11   | **initgroups(user, extra_group)** 读取 /etc/group ，并初始化群组访问列表，使用成员所在的所有群组。这是授权操作，所以你需要有 root 权限，或者有 CAP_SETGID 能力。 注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。 |
   | 12   | **kill(pid[, signal])** 发送信号给进程. pid 是进程id，并且 signal 是发送的信号的字符串描述。信号名是字符串，比如 'SIGINT' 或 'SIGHUP'。如果忽略，信号会是 'SIGTERM'。 |
   | 13   | **memoryUsage()** 返回一个对象，描述了 Node 进程所用的内存状况，单位为字节。 |
   | 14   | **nextTick(callback)** 一旦当前事件循环结束，调用回调函数。  |
   | 15   | **umask([mask])** 设置或读取进程文件的掩码。子进程从父进程继承掩码。如果mask 参数有效，返回旧的掩码。否则，返回当前掩码。 |
   | 16   | **uptime()** 返回 Node 已经运行的秒数。                      |
   | 17   | **hrtime()** 返回当前进程的高分辨时间，形式为 [seconds, nanoseconds]数组。它是相对于过去的任意事件。该值与日期无关，因此不受时钟漂移的影响。主要用途是可以通过精确的时间间隔，来衡量程序的性能。 你可以将之前的结果传递给当前的 process.hrtime() ，会返回两者间的时间差，用来基准和测量时间间隔。 |

   创建文件 main.js ，代码如下所示：

   ```js
   // 输出当前目录
   console.log('当前目录: ' + process.cwd());
   
   // 输出当前版本
   console.log('当前版本: ' + process.version);
   
   // 输出内存使用情况
   console.log(process.memoryUsage());
   ```

   执行 main.js 文件，代码如下所示:

   ```shell
   $ node main.js
   当前目录: /home/hyp/programer/vscode/node
   当前版本: v10.16.0
   { rss: 32219136,
     heapTotal: 7061504,
     heapUsed: 4516040,
     external: 8272 }
   ```
   
   