---
title: Node.JS 深入
date: 2019-06-30 18:18:59
tags:
 - Node.JS
 - 后端
categories:
 - 后端
 - Node.JS
---

#### Node.js 常用工具

util 是一个Node.js 核心模块，提供常用函数的集合，用于弥补核心JavaScript 的功能 过于精简的不足。

<!--more-->

##### util.inherits

**util.inherits(constructor, superConstructor)** 是一个实现对象间原型继承的函数。

JavaScript 的面向对象特性是基于原型的，与常见的基于类的不同。JavaScript 没有提供对象继承的语言级别特性，而是通过原型复制来实现的。

在这里我们只介绍 util.inherits 的用法，示例如下：

```js
let util = require('util');
function Base() {
        this.name = 'name';
        this.base = 1995;
        this.sayHello = function() {
                console.log('hello ' + this.name);
        }
}

Base.prototype.showName = function() {
        console.log(this.name);
}

function sub() {
        this.name = 'sub';
}

util.inherits(sub, Base);

let baseObj = new Base();
console.log(baseObj);
baseObj.showName();

let subObj = new sub();
console.log(subObj);
console.log(subObj.name);
subObj.showName();
```

我们定义了一个基础对象 Base 和一个继承自 Base 的 Sub，Base 有三个在构造函数内定义的属性和一个原型中定义的函数，通过util.inherits 实现继承。运行结果如下：

```
Base { name: 'name', base: 1995, sayHello: [Function] }
name
sub { name: 'sub' }
sub
sub
```

**注意：**Sub 仅仅继承了Base 在原型中定义的函数，而构造函数内部创造的 base 属 性和 sayHello 函数都没有被 Sub 继承。

##### util.inspect

**util.inspect(object,[showHidden],[depth],[colors])** 是一个将任意对象转换 为字符串的方法，通常用于调试和错误输出。它至少接受一个参数 object，即要转换的对象。

showHidden 是一个可选参数，如果值为 true，将会输出更多隐藏信息。

depth 表示最大递归的层数，如果对象很复杂，你可以指定层数以控制输出信息的多 少。如果不指定depth，默认会递归2层，指定为 null 表示将不限递归层数完整遍历对象。 如果color 值为 true，输出格式将会以ANSI 颜色编码，通常用于在终端显示更漂亮 的效果。

特别要指出的是，util.inspect 并不会简单地直接把对象转换为字符串，即使该对 象定义了 toString 方法也不会调用。

```
var util = require('util'); 
function Person() { 
    this.name = 'byvoid'; 
    this.toString = function() { 
    return this.name; 
    }; 
} 
var obj = new Person(); 
console.log(util.inspect(obj)); 
console.log(util.inspect(obj, true)); 
```

运行结果是：

```
Person { name: 'byvoid', toString: [Function] }
Person {
  name: 'byvoid',
  toString:
   { [Function]
     [length]: 0,
     [name]: '',
     [arguments]: null,
     [caller]: null,
     [prototype]: { [constructor]: [Circular] } } }
```

更多详情可以访问 [http://nodejs.cn/api/util.html](http://nodejs.cn/api/util.html) 了解详细内容。

#### Node.js 文件系统

Node.js 提供一组类似 UNIX（POSIX）标准的文件操作API。 Node 导入文件系统模块(fs)语法如下所示：

```
var fs = require("fs")
```

**异步和同步**

Node.js 文件系统（fs 模块）模块中的方法均有异步和同步版本，例如读取文件内容的函数有异步的 fs.readFile() 和同步的 fs.readFileSync()。

异步的方法函数最后一个参数为回调函数，回调函数的第一个参数包含了错误信息(error)。

建议大家使用异步方法，比起同步，异步方法性能更高，速度更快，而且没有阻塞。

创建 input.txt 文件，内容如下：

```
平心地址：https://hanyunpeng0521.github.io
文件读取实例
```

创建 file.js 文件, 代码如下：

```
var fs = require("fs");

// 异步读取
fs.readFile('input.txt', function (err, data) {
   if (err) {
       return console.error(err);
   }
   console.log("异步读取: " + data.toString());
});

// 同步读取
var data = fs.readFileSync('input.txt');
console.log("同步读取: " + data.toString());

console.log("程序执行完毕。");
```

以上代码执行结果如下：

```
$ node file.js 
平心地址：https://hanyunpeng0521.github.io
文件读取实例

程序执行完毕。
平心地址：https://hanyunpeng0521.github.io
文件读取实例
```

接下来，让我们来具体了解下 Node.js 文件系统的方法。

**打开文件**

以下为在异步模式下打开文件的语法格式：

```
fs.open(path, flags[, mode], callback)
```

参数使用说明如下：

- **path** - 文件的路径。
- **flags** - 文件打开的行为。具体值详见下文。
- **mode** - 设置文件模式(权限)，文件创建默认权限为 0666(可读，可写)。
- **callback** - 回调函数，带有两个参数如：callback(err, fd)。

flags 参数可以是以下值：

| Flag | 描述                                                   |
| :--- | :----------------------------------------------------- |
| r    | 以读取模式打开文件。如果文件不存在抛出异常。           |
| r+   | 以读写模式打开文件。如果文件不存在抛出异常。           |
| rs   | 以同步的方式读取文件。                                 |
| rs+  | 以同步的方式读取和写入文件。                           |
| w    | 以写入模式打开文件，如果文件不存在则创建。             |
| wx   | 类似 'w'，但是如果文件路径存在，则文件写入失败。       |
| w+   | 以读写模式打开文件，如果文件不存在则创建。             |
| wx+  | 类似 'w+'， 但是如果文件路径存在，则文件读写失败。     |
| a    | 以追加模式打开文件，如果文件不存在则创建。             |
| ax   | 类似 'a'， 但是如果文件路径存在，则文件追加失败。      |
| a+   | 以读取追加模式打开文件，如果文件不存在则创建。         |
| ax+  | 类似 'a+'， 但是如果文件路径存在，则文件读取追加失败。 |

接下来我们创建 file.js 文件，并打开 input.txt 文件进行读写，代码如下所示：

```
var fs = require("fs");

// 异步打开文件
console.log("准备打开文件！");
fs.open('input.txt', 'r+', function(err, fd) {
   if (err) {
       return console.error(err);
   }
  console.log("文件打开成功！");     
});
```

以上代码执行结果如下：

```
$ node file.js 
准备打开文件！
文件打开成功！
```

**获取文件信息**

以下为通过异步模式获取文件信息的语法格式：

```
fs.stat(path, callback)
```

参数使用说明如下：

- **path** - 文件路径。
- **callback** - 回调函数，带有两个参数如：(err, stats), **stats** 是 fs.Stats 对象。

fs.stat(path)执行后，会将stats类的实例返回给其回调函数。可以通过stats类中的提供方法判断文件的相关属性。例如判断是否为文件：

```
var fs = require('fs');

fs.stat('/Users/liuht/code/itbilu/demo/fs.js', function (err, stats) {
    console.log(stats.isFile());         //true
})
```

stats类中的方法有：

| 方法                      | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| stats.isFile()            | 如果是文件返回 true，否则返回 false。                        |
| stats.isDirectory()       | 如果是目录返回 true，否则返回 false。                        |
| stats.isBlockDevice()     | 如果是块设备返回 true，否则返回 false。                      |
| stats.isCharacterDevice() | 如果是字符设备返回 true，否则返回 false。                    |
| stats.isSymbolicLink()    | 如果是软链接返回 true，否则返回 false。                      |
| stats.isFIFO()            | 如果是FIFO，返回true，否则返回 false。FIFO是UNIX中的一种特殊类型的命令管道。 |
| stats.isSocket()          | 如果是 Socket 返回 true，否则返回 false。                    |

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");

console.log("准备打开文件！");
fs.stat('input.txt', function (err, stats) {
   if (err) {
       return console.error(err);
   }
   console.log(stats);
   console.log("读取文件信息成功！");
   
   // 检测文件类型
   console.log("是否为文件(isFile) ? " + stats.isFile());
   console.log("是否为目录(isDirectory) ? " + stats.isDirectory());    
});
```

以上代码执行结果如下：

```
$ node file.js 
准备打开文件！
{ dev: 16777220,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4096,
  ino: 40333161,
  size: 61,
  blocks: 8,
  atime: Mon Sep 07 2015 17:43:55 GMT+0800 (CST),
  mtime: Mon Sep 07 2015 17:22:35 GMT+0800 (CST),
  ctime: Mon Sep 07 2015 17:22:35 GMT+0800 (CST) }
读取文件信息成功！
是否为文件(isFile) ? true
是否为目录(isDirectory) ? false
```

**写入文件**

以下为异步模式下写入文件的语法格式：

```
fs.writeFile(file, data[, options], callback)
```

writeFile 直接打开文件默认是 **w** 模式，所以如果文件存在，该方法写入的内容会覆盖旧的文件内容。

参数使用说明如下：

- **file** - 文件名或文件描述符。
- **data** - 要写入文件的数据，可以是 String(字符串) 或 Buffer(缓冲) 对象。
- **options** - 该参数是一个对象，包含 {encoding, mode, flag}。默认编码为 utf8, 模式为 0666 ， flag 为 'w'
- **callback** - 回调函数，回调函数只包含错误信息参数(err)，在写入失败时返回。

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");

console.log("准备写入文件");
fs.writeFile('input.txt', '我是通 过fs.writeFile 写入文件的内容',  function(err) {
   if (err) {
       return console.error(err);
   }
   console.log("数据写入成功！");
   console.log("--------我是分割线-------------")
   console.log("读取写入的数据！");
   fs.readFile('input.txt', function (err, data) {
      if (err) {
         return console.error(err);
      }
      console.log("异步读取文件数据: " + data.toString());
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
准备写入文件
数据写入成功！
--------我是分割线-------------
读取写入的数据！
异步读取文件数据: 我是通 过fs.writeFile 写入文件的内容
```

**读取文件**

以下为异步模式下读取文件的语法格式：

```
fs.read(fd, buffer, offset, length, position, callback)
```

该方法使用了文件描述符来读取文件。

参数使用说明如下：

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **buffer** - 数据写入的缓冲区。
- **offset** - 缓冲区写入的写入偏移量。
- **length** - 要从文件中读取的字节数。
- **position** - 文件读取的起始位置，如果 position 的值为 null，则会从当前文件指针的位置读取。
- **callback** - 回调函数，有三个参数err, bytesRead, buffer，err 为错误信息， bytesRead 表示读取的字节数，buffer 为缓冲区对象。

input.txt 文件内容为：

```
平心地址：https://hanyunpeng0521.github.io
```

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");
var buf = new Buffer.alloc(1024);

console.log("准备打开已存在的文件！");
fs.open('input.txt', 'r+', function(err, fd) {
   if (err) {
       return console.error(err);
   }
   console.log("文件打开成功！");
   console.log("准备读取文件：");
   fs.read(fd, buf, 0, buf.length, 0, function(err, bytes){
      if (err){
         console.log(err);
      }
      console.log(bytes + "  字节被读取");
      
      // 仅输出读取的字节
      if(bytes > 0){
         console.log(buf.slice(0, bytes).toString());
      }
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
准备打开已存在的文件！
文件打开成功！
准备读取文件：
66  字节被读取
平心地址：https://hanyunpeng0521.github.io
```

**关闭文件**

以下为异步模式下关闭文件的语法格式：

```
fs.close(fd, callback)
```

该方法使用了文件描述符来读取文件。

参数使用说明如下：

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **callback** - 回调函数，没有参数。

input.txt 文件内容为：

```
平心地址：https://hanyunpeng0521.github.io
```

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");
var buf = new Buffer.alloc(1024);

console.log("准备打开文件！");
fs.open('input.txt', 'r+', function(err, fd) {
   if (err) {
       return console.error(err);
   }
   console.log("文件打开成功！");
   console.log("准备读取文件！");
   fs.read(fd, buf, 0, buf.length, 0, function(err, bytes){
      if (err){
         console.log(err);
      }

      // 仅输出读取的字节
      if(bytes > 0){
         console.log(buf.slice(0, bytes).toString());
      }

      // 关闭文件
      fs.close(fd, function(err){
         if (err){
            console.log(err);
         } 
         console.log("文件关闭成功");
      });
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
准备打开文件！
文件打开成功！
准备读取文件！
平心地址：https://hanyunpeng0521.github.io
文件关闭成功
```

**截取文件**

以下为异步模式下截取文件的语法格式：

```
fs.ftruncate(fd, len, callback)
```

该方法使用了文件描述符来读取文件。

参数使用说明如下：

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **len** - 文件内容截取的长度。
- **callback** - 回调函数，没有参数。

input.txt 文件内容为：

```
平心地址：https://hanyunpeng0521.github.io
```

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");
var buf = new Buffer.alloc(1024);

console.log("准备打开文件！");
fs.open('input.txt', 'r+', function(err, fd) {
   if (err) {
       return console.error(err);
   }
   console.log("文件打开成功！");
   console.log("截取10字节内的文件内容，超出部分将被去除。");
   
   // 截取文件
   fs.ftruncate(fd, 10, function(err){
      if (err){
         console.log(err);
      } 
      console.log("文件截取成功。");
      console.log("读取相同的文件"); 
      fs.read(fd, buf, 0, buf.length, 0, function(err, bytes){
         if (err){
            console.log(err);
         }

         // 仅输出读取的字节
         if(bytes > 0){
            console.log(buf.slice(0, bytes).toString());
         }

         // 关闭文件
         fs.close(fd, function(err){
            if (err){
               console.log(err);
            } 
            console.log("文件关闭成功！");
         });
      });
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
准备打开文件！
文件打开成功！
截取10字节内的文件内容，超出部分将被去除。
文件截取成功。
读取相同的文件
...
文件关闭成功
```

**删除文件**

以下为删除文件的语法格式：

```
fs.unlink(path, callback)
```

参数使用说明如下：

- **path** - 文件路径。
- **callback** - 回调函数，没有参数。

input.txt 文件内容为：

```
平心地址：https://hanyunpeng0521.github.io
```

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");

console.log("准备删除文件！");
fs.unlink('input.txt', function(err) {
   if (err) {
       return console.error(err);
   }
   console.log("文件删除成功！");
});
```

以上代码执行结果如下：

```
$ node file.js 
准备删除文件！
文件删除成功！
```

再去查看 input.txt 文件，发现已经不存在了。

**创建目录**

以下为创建目录的语法格式：

```
fs.mkdir(path[, options], callback)
```

参数使用说明如下：

- **path** - 文件路径。
- options 参数可以是：
  - **recursive** - 是否以递归的方式创建目录，默认为 false。
  - **mode** - 设置目录权限，默认为 0777。
- **callback** - 回调函数，没有参数。

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");
// tmp 目录必须存在
console.log("创建目录 /tmp/test/");
fs.mkdir("/tmp/test/",function(err){
   if (err) {
       return console.error(err);
   }
   console.log("目录创建成功。");
});
```

以上代码执行结果如下：

```
$ node file.js 
创建目录 /tmp/test/
目录创建成功。
```

可以添加 recursive: true 参数，不管创建的目录 /tmp 和 /tmp/a 是否存在：

```
fs.mkdir('/tmp/a/apple', { recursive: true }, (err) => {
  if (err) throw err;
});
```

**读取目录**

以下为读取目录的语法格式：

```
fs.readdir(path, callback)
```

参数使用说明如下：

- **path** - 文件路径。
- **callback** - 回调函数，回调函数带有两个参数err, files，err 为错误信息，files 为 目录下的文件数组列表。

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");

console.log("查看 /tmp 目录");
fs.readdir("/tmp/",function(err, files){
   if (err) {
       return console.error(err);
   }
   files.forEach( function (file){
       console.log( file );
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
查看 /tmp 目录
input.out
output.out
test
test.txt
```

**删除目录**

以下为删除目录的语法格式：

```
fs.rmdir(path, callback)
```

参数使用说明如下：

- **path** - 文件路径。
- **callback** - 回调函数，没有参数。

接下来我们创建 file.js 文件，代码如下所示：

```
var fs = require("fs");
// 执行前创建一个空的 /tmp/test 目录
console.log("准备删除目录 /tmp/test");
fs.rmdir("/tmp/test",function(err){
   if (err) {
       return console.error(err);
   }
   console.log("读取 /tmp 目录");
   fs.readdir("/tmp/",function(err, files){
      if (err) {
          return console.error(err);
      }
      files.forEach( function (file){
          console.log( file );
      });
   });
});
```

以上代码执行结果如下：

```
$ node file.js 
准备删除目录 /tmp/test
读取 /tmp 目录
……
```

以下为 Node.js 文件模块相同的方法列表：

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **fs.rename(oldPath, newPath, callback)** 异步 rename().回调函数没有参数，但可能抛出异常。 |
| 2    | **fs.ftruncate(fd, len, callback)** 异步 ftruncate().回调函数没有参数，但可能抛出异常。 |
| 3    | **fs.ftruncateSync(fd, len)** 同步 ftruncate()               |
| 4    | **fs.truncate(path, len, callback)** 异步 truncate().回调函数没有参数，但可能抛出异常。 |
| 5    | **fs.truncateSync(path, len)** 同步 truncate()               |
| 6    | **fs.chown(path, uid, gid, callback)** 异步 chown().回调函数没有参数，但可能抛出异常。 |
| 7    | **fs.chownSync(path, uid, gid)** 同步 chown()                |
| 8    | **fs.fchown(fd, uid, gid, callback)** 异步 fchown().回调函数没有参数，但可能抛出异常。 |
| 9    | **fs.fchownSync(fd, uid, gid)** 同步 fchown()                |
| 10   | **fs.lchown(path, uid, gid, callback)** 异步 lchown().回调函数没有参数，但可能抛出异常。 |
| 11   | **fs.lchownSync(path, uid, gid)** 同步 lchown()              |
| 12   | **fs.chmod(path, mode, callback)** 异步 chmod().回调函数没有参数，但可能抛出异常。 |
| 13   | **fs.chmodSync(path, mode)** 同步 chmod().                   |
| 14   | **fs.fchmod(fd, mode, callback)** 异步 fchmod().回调函数没有参数，但可能抛出异常。 |
| 15   | **fs.fchmodSync(fd, mode)** 同步 fchmod().                   |
| 16   | **fs.lchmod(path, mode, callback)** 异步 lchmod().回调函数没有参数，但可能抛出异常。Only available on Mac OS X. |
| 17   | **fs.lchmodSync(path, mode)** 同步 lchmod().                 |
| 18   | **fs.stat(path, callback)** 异步 stat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。 |
| 19   | **fs.lstat(path, callback)** 异步 lstat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。 |
| 20   | **fs.fstat(fd, callback)** 异步 fstat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。 |
| 21   | **fs.statSync(path)** 同步 stat(). 返回 fs.Stats 的实例。    |
| 22   | **fs.lstatSync(path)** 同步 lstat(). 返回 fs.Stats 的实例。  |
| 23   | **fs.fstatSync(fd)** 同步 fstat(). 返回 fs.Stats 的实例。    |
| 24   | **fs.link(srcpath, dstpath, callback)** 异步 link().回调函数没有参数，但可能抛出异常。 |
| 25   | **fs.linkSync(srcpath, dstpath)** 同步 link().               |
| 26   | **fs.symlink(srcpath, dstpath[, type], callback)** 异步 symlink().回调函数没有参数，但可能抛出异常。 type 参数可以设置为 'dir', 'file', 或 'junction' (默认为 'file') 。 |
| 27   | **fs.symlinkSync(srcpath, dstpath[, type])** 同步 symlink(). |
| 28   | **fs.readlink(path, callback)** 异步 readlink(). 回调函数有两个参数 err, linkString。 |
| 29   | **fs.realpath(path[, cache], callback)** 异步 realpath(). 回调函数有两个参数 err, resolvedPath。 |
| 30   | **fs.realpathSync(path[, cache])** 同步 realpath()。返回绝对路径。 |
| 31   | **fs.unlink(path, callback)** 异步 unlink().回调函数没有参数，但可能抛出异常。 |
| 32   | **fs.unlinkSync(path)** 同步 unlink().                       |
| 33   | **fs.rmdir(path, callback)** 异步 rmdir().回调函数没有参数，但可能抛出异常。 |
| 34   | **fs.rmdirSync(path)** 同步 rmdir().                         |
| 35   | **fs.mkdir(path[, mode], callback)** S异步 mkdir(2).回调函数没有参数，但可能抛出异常。 访问权限默认为 0777。 |
| 36   | **fs.mkdirSync(path[, mode])** 同步 mkdir().                 |
| 37   | **fs.readdir(path, callback)** 异步 readdir(3). 读取目录的内容。 |
| 38   | **fs.readdirSync(path)** 同步 readdir().返回文件数组列表。   |
| 39   | **fs.close(fd, callback)** 异步 close().回调函数没有参数，但可能抛出异常。 |
| 40   | **fs.closeSync(fd)** 同步 close().                           |
| 41   | **fs.open(path, flags[, mode], callback)** 异步打开文件。    |
| 42   | **fs.openSync(path, flags[, mode])** 同步 version of fs.open(). |
| 43   | **fs.utimes(path, atime, mtime, callback)**                  |
| 44   | **fs.utimesSync(path, atime, mtime)** 修改文件时间戳，文件通过指定的文件路径。 |
| 45   | **fs.futimes(fd, atime, mtime, callback)**                   |
| 46   | **fs.futimesSync(fd, atime, mtime)** 修改文件时间戳，通过文件描述符指定。 |
| 47   | **fs.fsync(fd, callback)** 异步 fsync.回调函数没有参数，但可能抛出异常。 |
| 48   | **fs.fsyncSync(fd)** 同步 fsync.                             |
| 49   | **fs.write(fd, buffer, offset, length[, position], callback)** 将缓冲区内容写入到通过文件描述符指定的文件。 |
| 50   | **fs.write(fd, data[, position[, encoding]], callback)** 通过文件描述符 fd 写入文件内容。 |
| 51   | **fs.writeSync(fd, buffer, offset, length[, position])** 同步版的 fs.write()。 |
| 52   | **fs.writeSync(fd, data[, position[, encoding]])** 同步版的 fs.write(). |
| 53   | **fs.read(fd, buffer, offset, length, position, callback)** 通过文件描述符 fd 读取文件内容。 |
| 54   | **fs.readSync(fd, buffer, offset, length, position)** 同步版的 fs.read. |
| 55   | **fs.readFile(filename[, options], callback)** 异步读取文件内容。 |
| 56   | **fs.readFileSync(filename[, options])**                     |
| 57   | **fs.writeFile(filename, data[, options], callback)** 异步写入文件内容。 |
| 58   | **fs.writeFileSync(filename, data[, options])** 同步版的 fs.writeFile。 |
| 59   | **fs.appendFile(filename, data[, options], callback)** 异步追加文件内容。 |
| 60   | **fs.appendFileSync(filename, data[, options])** The 同步 version of fs.appendFile. |
| 61   | **fs.watchFile(filename[, options], listener)** 查看文件的修改。 |
| 62   | **fs.unwatchFile(filename[, listener])** 停止查看 filename 的修改。 |
| 63   | `fs.watch(filename[, options][, listener]) 查看 filename 的修改，filename 可以是文件或目录。返回 fs.FSWatcher 对象。` |
| 64   | **fs.exists(path, callback)** 检测给定的路径是否存在。       |
| 65   | **fs.existsSync(path)** 同步版的 fs.exists.                  |
| 66   | **fs.access(path[, mode], callback)** 测试指定路径用户权限。 |
| 67   | **fs.accessSync(path[, mode])** 同步版的 fs.access。         |
| 68   | **fs.createReadStream(path[, options])** 返回ReadStream 对象。 |
| 69   | **fs.createWriteStream(path[, options])** 返回 WriteStream 对象。 |
| 70   | **fs.symlink(srcpath, dstpath[, type], callback)** 异步 symlink().回调函数没有参数，但可能抛出异常。 |

更多内容，请查看官网文件模块描述：[File System](http://nodejs.cn/api/fs.html)。

#### Node.js GET/POST请求

在很多场景中，我们的服务器都需要跟用户的浏览器打交道，如表单提交。

表单提交到服务器一般都使用 GET/POST 请求。

##### 获取GET请求内容

由于GET请求直接被嵌入在路径中，URL是完整的请求路径，包括了?后面的部分，因此你可以手动解析后面的内容作为GET请求的参数。

node.js 中 url 模块中的 parse 函数提供了这个功能。

```js
var http = require('http');
var url = require('url');
var util = require('util');
 
http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/plain; charset=utf-8'});
    res.end(util.inspect(url.parse(req.url, true)));
}).listen(3000);
```

在浏览器中访问 <http://localhost:3000/user?name=平心&url=https://hanyunpeng0521.github.io> 然后查看返回结果

##### 获取 URL 的参数

我们可以使用 url.parse 方法来解析 URL 中的参数，代码如下：

```js
var http = require('http');
var url = require('url');
var util = require('util');
 
http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/html; charset=utf8'});
 
    // 解析 url 参数
    var params = url.parse(req.url, true).query;
    res.write("网站名：" + params.name);
    res.write("\n");
    res.write("网站 URL：" + params.url);
    res.end();
 
}).listen(3000);
```

在浏览器中访问 <http://localhost:3000/user?name=平心&url=https://hanyunpeng0521.github.io> 然后查看返回结果

##### 获取 POST 请求内容

POST 请求的内容全部的都在请求体中，http.ServerRequest 并没有一个属性内容为请求体，原因是等待请求体传输可能是一件耗时的工作。

比如上传文件，而很多时候我们可能并不需要理会请求体的内容，恶意的POST请求会大大消耗服务器的资源，所以 node.js 默认是不会解析请求体的，当你需要的时候，需要手动来做。

```js
var http = require('http');
var querystring = require('querystring');
var util = require('util');
 
http.createServer(function(req, res){
    // 定义了一个post变量，用于暂存请求体的信息
    var post = '';     
 
    // 通过req的data事件监听函数，每当接受到请求体的数据，就累加到post变量中
    req.on('data', function(chunk){    
        post += chunk;
    });
 
    // 在end事件触发后，通过querystring.parse将post解析为真正的POST请求格式，然后向客户端返回。
    req.on('end', function(){    
        post = querystring.parse(post);
        res.end(util.inspect(post));
    });
}).listen(3000);
```

以下实例表单通过 POST 提交并输出数据：

```js
var http = require('http');
var querystring = require('querystring');
 
var postHTML = 
  '<html><head><meta charset="utf-8"><title>示例</title></head>' +
  '<body>' +
  '<form method="post">' +
  '网站名： <input name="name"><br>' +
  '网站 URL： <input name="url"><br>' +
  '<input type="submit">' +
  '</form>' +
  '</body></html>';
 
http.createServer(function (req, res) {
  var body = "";
  req.on('data', function (chunk) {
    body += chunk;
  });
  req.on('end', function () {
    // 解析参数
    body = querystring.parse(body);
    // 设置响应头部信息及编码
    res.writeHead(200, {'Content-Type': 'text/html; charset=utf8'});
 
    if(body.name && body.url) { // 输出提交的数据
        res.write("网站名：" + body.name);
        res.write("<br>");
        res.write("网站 URL：" + body.url);
    } else {  // 输出表单
        res.write(postHTML);
    }
    res.end();
  });
}).listen(3000);
```

#### Node.js 工具模块

##### OS 模块

Node.js os 模块提供了一些基本的系统操作函数。我们可以通过以下方式引入该模块：

```
var os = require("os")
```

方法

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **os.tmpdir()** 返回操作系统的默认临时文件夹。               |
| 2    | **os.endianness()** 返回 CPU 的字节序，可能的是 "BE" 或 "LE"。 |
| 3    | **os.hostname()** 返回操作系统的主机名。                     |
| 4    | **os.type()** 返回操作系统名                                 |
| 5    | **os.platform()** 返回编译时的操作系统名                     |
| 6    | **os.arch()** 返回操作系统 CPU 架构，可能的值有 "x64"、"arm" 和 "ia32"。 |
| 7    | **os.release()** 返回操作系统的发行版本。                    |
| 8    | **os.uptime()** 返回操作系统运行的时间，以秒为单位。         |
| 9    | **os.loadavg()** 返回一个包含 1、5、15 分钟平均负载的数组。  |
| 10   | **os.totalmem()** 返回系统内存总量，单位为字节。             |
| 11   | **os.freemem()** 返回操作系统空闲内存量，单位是字节。        |
| 12   | **os.cpus()** 返回一个对象数组，包含所安装的每个 CPU/内核的信息：型号、速度（单位 MHz）、时间（一个包含 user、nice、sys、idle 和 irq 所使用 CPU/内核毫秒数的对象）。 |
| 13   | **os.networkInterfaces()** 获得网络接口列表。                |

属性

| 序号 | 属性 & 描述                               |
| :--- | :---------------------------------------- |
| 1    | **os.EOL** 定义了操作系统的行尾符的常量。 |

创建 main.js 文件，代码如下所示：

```js
var os = require("os");

// CPU 的字节序
console.log('endianness : ' + os.endianness());

// 操作系统名
console.log('type : ' + os.type());

// 操作系统名
console.log('platform : ' + os.platform());

// 系统内存总量
console.log('total memory : ' + os.totalmem() + " bytes.");

// 操作系统空闲内存量
console.log('free memory : ' + os.freemem() + " bytes.");
```

代码执行结果如下：

```
$ node main.js 
endianness : LE
type : Linux
platform : linux
total memory : 8250183680 bytes.
free memory : 339279872 bytes.
```

##### Node.js Path 模块

Node.js path 模块提供了一些用于处理文件路径的小工具，我们可以通过以下方式引入该模块：

```
var path = require("path")
```

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **path.normalize(p)** 规范化路径，注意'..' 和 '.'。          |
| 2    | `path.join([path1][, path2][, ...]) 用于连接路径。该方法的主要用途在于，会正确使用当前系统的路径分隔符，Unix系统是"/"，Windows系统是"\"。` |
| 3    | **path.resolve([from ...], to)** 将 **to** 参数解析为绝对路径，给定的路径的序列是从右往左被处理的，后面每个 path 被依次解析，直到构造完成一个绝对路径。 例如，给定的路径片段的序列为：/foo、/bar、baz，则调用 path.resolve('/foo', '/bar', 'baz') 会返回 /bar/baz。`path.resolve('/foo/bar', './baz'); // 返回: '/foo/bar/baz'  path.resolve('/foo/bar', '/tmp/file/'); // 返回: '/tmp/file'  path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif'); // 如果当前工作目录为 /home/myself/node， // 则返回 '/home/myself/node/wwwroot/static_files/gif/image.gif'` |
| 4    | **path.isAbsolute(path)** 判断参数 **path** 是否是绝对路径。 |
| 5    | **path.relative(from, to)** 用于将绝对路径转为相对路径，返回从 from 到 to 的相对路径（基于当前工作目录）。在 Linux 上：`path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb'); // 返回: '../../impl/bbb'`在 Windows 上：`path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb'); // 返回: '..\\..\\impl\\bbb'` |
| 6    | **path.dirname(p)** 返回路径中代表文件夹的部分，同 Unix 的dirname 命令类似。 |
| 7    | **path.basename(p[, ext])** 返回路径中的最后一部分。同 Unix 命令 bashname 类似。 |
| 8    | **path.extname(p)** 返回路径中文件的后缀名，即路径中最后一个'.'之后的部分。如果一个路径中并不包含'.'或该路径只包含一个'.' 且这个'.'为路径的第一个字符，则此命令返回空字符串。 |
| 9    | **path.parse(pathString)** 返回路径字符串的对象。            |
| 10   | **path.format(pathObject)** 从对象中返回路径字符串，和 path.parse 相反。 |

属性

| 序号 | 属性 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **path.sep** 平台的文件路径分隔符，'\\' 或 '/'。             |
| 2    | **path.delimiter** 平台的分隔符, ; or ':'.                   |
| 3    | **path.posix** 提供上述 path 的方法，不过总是以 posix 兼容的方式交互。 |
| 4    | **path.win32** 提供上述 path 的方法，不过总是以 win32 兼容的方式交互。 |

创建 main.js 文件，代码如下所示：

```
var path = require("path");

// 格式化路径
console.log('normalization : ' + path.normalize('/test/test1//2slashes/1slash/tab/..'));

// 连接路径
console.log('joint path : ' + path.join('/test', 'test1', '2slashes/1slash', 'tab', '..'));

// 转换为绝对路径
console.log('resolve : ' + path.resolve('main.js'));

// 路径中文件的后缀名
console.log('ext name : ' + path.extname('main.js'));
```

##### Net 模块

Node.js Net 模块提供了一些用于底层的网络通信的小工具，包含了创建服务器/客户端的方法，我们可以通过以下方式引入该模块：

```
var net = require("net")
```

**方法**

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | `net.createServer([options][, connectionListener]) 创建一个 TCP 服务器。参数 connectionListener 自动给 'connection' 事件创建监听器。` |
| 2    | **net.connect(options[, connectionListener])** 返回一个新的 'net.Socket'，并连接到指定的地址和端口。 当 socket 建立的时候，将会触发 'connect' 事件。 |
| 3    | **net.createConnection(options[, connectionListener])** 创建一个到端口 port 和 主机 host的 TCP 连接。 host 默认为 'localhost'。 |
| 4    | `net.connect(port[, host][, connectListener]) 创建一个端口为 port 和主机为 host的 TCP 连接 。host 默认为 'localhost'。参数 connectListener 将会作为监听器添加到 'connect' 事件。返回 'net.Socket'。` |
| 5    | `net.createConnection(port[, host][, connectListener]) 创建一个端口为 port 和主机为 host的 TCP 连接 。host 默认为 'localhost'。参数 connectListener 将会作为监听器添加到 'connect' 事件。返回 'net.Socket'。` |
| 6    | **net.connect(path[, connectListener])** 创建连接到 path 的 unix socket 。参数 connectListener 将会作为监听器添加到 'connect' 事件上。返回 'net.Socket'。 |
| 7    | **net.createConnection(path[, connectListener])** 创建连接到 path 的 unix socket 。参数 connectListener 将会作为监听器添加到 'connect' 事件。返回 'net.Socket'。 |
| 8    | **net.isIP(input)** 检测输入的是否为 IP 地址。 IPV4 返回 4， IPV6 返回 6，其他情况返回 0。 |
| 9    | **net.isIPv4(input)** 如果输入的地址为 IPV4， 返回 true，否则返回 false。 |
| 10   | **net.isIPv6(input)** 如果输入的地址为 IPV6， 返回 true，否则返回 false。 |

##### net.Server

net.Server通常用于创建一个 TCP 或本地服务器。

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | `server.listen(port[, host][, backlog][, callback]) 监听指定端口 port 和 主机 host ac连接。 默认情况下 host 接受任何 IPv4 地址(INADDR_ANY)的直接连接。端口 port 为 0 时，则会分配一个随机端口。` |
| 2    | **server.listen(path[, callback])** 通过指定 path 的连接，启动一个本地 socket 服务器。 |
| 3    | **server.listen(handle[, callback])** 通过指定句柄连接。     |
| 4    | **server.listen(options[, callback])** options 的属性：端口 port, 主机 host, 和 backlog, 以及可选参数 callback 函数, 他们在一起调用server.listen(port, [host], [backlog], [callback])。还有，参数 path 可以用来指定 UNIX socket。 |
| 5    | **server.close([callback])** 服务器停止接收新的连接，保持现有连接。这是异步函数，当所有连接结束的时候服务器会关闭，并会触发 'close' 事件。 |
| 6    | **server.address()** 操作系统返回绑定的地址，协议族名和服务器端口。 |
| 7    | **server.unref()** 如果这是事件系统中唯一一个活动的服务器，调用 unref 将允许程序退出。 |
| 8    | **server.ref()** 与 unref 相反，如果这是唯一的服务器，在之前被 unref 了的服务器上调用 ref 将不会让程序退出（默认行为）。如果服务器已经被 ref，则再次调用 ref 并不会产生影响。 |
| 9    | **server.getConnections(callback)** 异步获取服务器当前活跃连接的数量。当 socket 发送给子进程后才有效；回调函数有 2 个参数 err 和 count。 |

**事件**

| 序号 | 事件 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **listening** 当服务器调用 server.listen 绑定后会触发。      |
| 2    | **connection** 当新连接创建后会被触发。socket 是 net.Socket实例。 |
| 3    | **close** 服务器关闭时会触发。注意，如果存在连接，这个事件不会被触发直到所有的连接关闭。 |
| 4    | **error** 发生错误时触发。'close' 事件将被下列事件直接调用。 |

##### net.Socket

net.Socket 对象是 TCP 或 UNIX Socket 的抽象。net.Socket 实例实现了一个双工流接口。 他们可以在用户创建客户端(使用 connect())时使用, 或者由 Node 创建它们，并通过 connection 服务器事件传递给用户。

**事件**

net.Socket 事件有：

| 序号 | 事件 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **lookup** 在解析域名后，但在连接前，触发这个事件。对 UNIX sokcet 不适用。 |
| 2    | **connect** 成功建立 socket 连接时触发。                     |
| 3    | **data** 当接收到数据时触发。                                |
| 4    | **end** 当 socket 另一端发送 FIN 包时，触发该事件。          |
| 5    | **timeout** 当 socket 空闲超时时触发，仅是表明 socket 已经空闲。用户必须手动关闭连接。 |
| 6    | **drain** 当写缓存为空得时候触发。可用来控制上传。           |
| 7    | **error** 错误发生时触发。                                   |
| 8    | **close** 当 socket 完全关闭时触发。参数 had_error 是布尔值，它表示是否因为传输错误导致 socket 关闭。 |

**属性**

net.Socket 提供了很多有用的属性，便于控制 socket 交互：

| 序号 | 属性 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **socket.bufferSize** 该属性显示了要写入缓冲区的字节数。     |
| 2    | **socket.remoteAddress** 远程的 IP 地址字符串，例如：'74.125.127.100' or '2001:4860:a005::68'。 |
| 3    | **socket.remoteFamily** 远程IP协议族字符串，比如 'IPv4' or 'IPv6'。 |
| 4    | **socket.remotePort** 远程端口，数字表示，例如：80 or 21。   |
| 5    | **socket.localAddress** 网络连接绑定的本地接口 远程客户端正在连接的本地 IP 地址，字符串表示。例如，如果你在监听'0.0.0.0'而客户端连接在'192.168.1.1'，这个值就会是 '192.168.1.1'。 |
| 6    | **socket.localPort** 本地端口地址，数字表示。例如：80 or 21。 |
| 7    | **socket.bytesRead** 接收到得字节数。                        |
| 8    | **socket.bytesWritten** 发送的字节数。                       |

**方法**

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **new net.Socket([options])** 构造一个新的 socket 对象。     |
| 2    | `socket.connect(port[, host][, connectListener]) 指定端口 port 和 主机 host，创建 socket 连接 。参数 host 默认为 localhost。通常情况不需要使用 net.createConnection 打开 socket。只有你实现了自己的 socket 时才会用到。` |
| 3    | **socket.connect(path[, connectListener])** 打开指定路径的 unix socket。通常情况不需要使用 net.createConnection 打开 socket。只有你实现了自己的 socket 时才会用到。 |
| 4    | **socket.setEncoding([encoding])** 设置编码                  |
| 5    | `socket.write(data[, encoding][, callback]) 在 socket 上发送数据。第二个参数指定了字符串的编码，默认是 UTF8 编码。` |
| 6    | `socket.end([data][, encoding]) 半关闭 socket。例如，它发送一个 FIN 包。可能服务器仍在发送数据。` |
| 7    | **socket.destroy()** 确保没有 I/O 活动在这个套接字上。只有在错误发生情况下才需要。（处理错误等等）。 |
| 8    | **socket.pause()** 暂停读取数据。就是说，不会再触发 data 事件。对于控制上传非常有用。 |
| 9    | **socket.resume()** 调用 pause() 后想恢复读取数据。          |
| 10   | **socket.setTimeout(timeout[, callback])** socket 闲置时间超过 timeout 毫秒后 ，将 socket 设置为超时。 |
| 11   | **socket.setNoDelay([noDelay])** 禁用纳格（Nagle）算法。默认情况下 TCP 连接使用纳格算法，在发送前他们会缓冲数据。将 noDelay 设置为 true 将会在调用 socket.write() 时立即发送数据。noDelay 默认值为 true。 |
| 12   | `socket.setKeepAlive([enable][, initialDelay]) 禁用/启用长连接功能，并在发送第一个在闲置 socket 上的长连接 probe 之前，可选地设定初始延时。默认为 false。 设定 initialDelay （毫秒），来设定收到的最后一个数据包和第一个长连接probe之间的延时。将 initialDelay 设为0，将会保留默认（或者之前）的值。默认值为0.` |
| 13   | **socket.address()** 操作系统返回绑定的地址，协议族名和服务器端口。返回的对象有 3 个属性，比如{ port: 12346, family: 'IPv4', address: '127.0.0.1' }。 |
| 14   | **socket.unref()** 如果这是事件系统中唯一一个活动的服务器，调用 unref 将允许程序退出。如果服务器已被 unref，则再次调用 unref 并不会产生影响。 |
| 15   | **socket.ref()** 与 unref 相反，如果这是唯一的服务器，在之前被 unref 了的服务器上调用 ref 将不会让程序退出（默认行为）。如果服务器已经被 ref，则再次调用 ref 并不会产生影响。 |

**实例**

创建 server.js 文件，代码如下所示：

```js
var net = require('net');
var server = net.createServer(function(connection) { 
   console.log('client connected');
   connection.on('end', function() {
      console.log('客户端关闭连接');
   });
   connection.write('Hello World!\r\n');
   connection.pipe(connection);
});
server.listen(8080, function() { 
  console.log('server is listening');
});
```

执行以上服务端代码：

```
$ node server.js
server is listening   # 服务已创建并监听 8080 端口
```

新开一个窗口，创建 client.js 文件，代码如下所示：

```
var net = require('net');
var client = net.connect({port: 8080}, function() {
   console.log('连接到服务器！');  
});
client.on('data', function(data) {
   console.log(data.toString());
   client.end();
});
client.on('end', function() { 
   console.log('断开与服务器的连接');
});
```

执行以上客户端的代码：

```
连接到服务器！
Hello World!

断开与服务器的连接
```

##### DNS 模块

Node.js **DNS** 模块用于解析域名。引入 DNS 模块语法格式如下：

```
var dns = require("dns")
```

**方法**

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **dns.lookup(hostname[, options], callback)** 将域名（比如 'runoob.com'）解析为第一条找到的记录 A （IPV4）或 AAAA(IPV6)。参数 options可以是一个对象或整数。如果没有提供 options，IP v4 和 v6 地址都可以。如果 options 是整数，则必须是 4 或 6。 |
| 2    | **dns.lookupService(address, port, callback)** 使用 getnameinfo 解析传入的地址和端口为域名和服务。 |
| 3    | **dns.resolve(hostname[, rrtype], callback)** 将一个域名（如 'runoob.com'）解析为一个 rrtype 指定记录类型的数组。 |
| 4    | **dns.resolve4(hostname, callback)** 和 dns.resolve() 类似, 仅能查询 IPv4 (A 记录）。 addresses IPv4 地址数组 (比如，['74.125.79.104', '74.125.79.105', '74.125.79.106']）。 |
| 5    | **dns.resolve6(hostname, callback)** 和 dns.resolve4() 类似， 仅能查询 IPv6( AAAA 查询） |
| 6    | **dns.resolveMx(hostname, callback)** 和 dns.resolve() 类似, 仅能查询邮件交换(MX 记录)。 |
| 7    | **dns.resolveTxt(hostname, callback)** 和 dns.resolve() 类似, 仅能进行文本查询 (TXT 记录）。 addresses 是 2-d 文本记录数组。(比如，[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]）。 每个子数组包含一条记录的 TXT 块。根据使用情况可以连接在一起，也可单独使用。 |
| 8    | **dns.resolveSrv(hostname, callback)** 和 dns.resolve() 类似, 仅能进行服务记录查询 (SRV 记录）。 addresses 是 hostname可用的 SRV 记录数组。 SRV 记录属性有优先级（priority），权重（weight）, 端口（port）, 和名字（name） (比如，[{'priority': 10, 'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]）。 |
| 9    | **dns.resolveSoa(hostname, callback)** 和 dns.resolve() 类似, 仅能查询权威记录(SOA 记录）。 |
| 10   | **dns.resolveNs(hostname, callback)** 和 dns.resolve() 类似, 仅能进行域名服务器记录查询(NS 记录）。 addresses 是域名服务器记录数组（hostname 可以使用） (比如, ['ns1.example.com', 'ns2.example.com']）。 |
| 11   | **dns.resolveCname(hostname, callback)** 和 dns.resolve() 类似, 仅能进行别名记录查询 (CNAME记录)。addresses 是对 hostname 可用的别名记录数组 (比如，, ['bar.example.com']）。 |
| 12   | **dns.reverse(ip, callback)** 反向解析 IP 地址，指向该 IP 地址的域名数组。 |
| 13   | **dns.getServers()** 返回一个用于当前解析的 IP 地址数组的字符串。 |
| 14   | **dns.setServers(servers)** 指定一组 IP 地址作为解析服务器。 |

**rrtypes**

以下列出了 dns.resolve() 方法中有效的 rrtypes值:

- `'A'` IPV4 地址, 默认
- `'AAAA'` IPV6 地址
- `'MX'` 邮件交换记录
- `'TXT'` text 记录
- `'SRV'` SRV 记录
- `'PTR'` 用来反向 IP 查找
- `'NS'` 域名服务器记录
- `'CNAME'` 别名记录
- `'SOA'` 授权记录的初始值

**错误码**

每次 DNS 查询都可能返回以下错误码：

- `dns.NODATA`: 无数据响应。
- `dns.FORMERR`: 查询格式错误。
- `dns.SERVFAIL`: 常规失败。
- `dns.NOTFOUND`: 没有找到域名。
- `dns.NOTIMP`: 未实现请求的操作。
- `dns.REFUSED`: 拒绝查询。
- `dns.BADQUERY`: 查询格式错误。
- `dns.BADNAME`: 域名格式错误。
- `dns.BADFAMILY`: 地址协议不支持。
- `dns.BADRESP`: 回复格式错误。
- `dns.CONNREFUSED`: 无法连接到 DNS 服务器。
- `dns.TIMEOUT`: 连接 DNS 服务器超时。
- `dns.EOF`: 文件末端。
- `dns.FILE`: 读文件错误。
- `dns.NOMEM`: 内存溢出。
- `dns.DESTRUCTION`: 通道被摧毁。
- `dns.BADSTR`: 字符串格式错误。
- `dns.BADFLAGS`: 非法标识符。
- `dns.NONAME`: 所给主机不是数字。
- `dns.BADHINTS`: 非法HINTS标识符。
- `dns.NOTINITIALIZED`: c c-ares 库尚未初始化。
- `dns.LOADIPHLPAPI`: 加载 iphlpapi.dll 出错。
- `dns.ADDRGETNETWORKPARAMS`: 无法找到 GetNetworkParams 函数。
- `dns.CANCELLED`: 取消 DNS 查询。

**实例**

创建 main.js 文件，代码如下所示：

```
var dns = require('dns');

dns.lookup('www.github.com', function onLookup(err, address, family) {
   console.log('ip 地址:', address);
   dns.reverse(address, function (err, hostnames) {
   if (err) {
      console.log(err.stack);
   }

   console.log('反向解析 ' + address + ': ' + JSON.stringify(hostnames));
});  
});
```

执行以上代码，结果如下所示:

```
address: 192.30.252.130
reverse for 192.30.252.130: ["github.com"]
```

##### Domain 模块

Node.js **Domain(域)** 简化异步代码的异常处理，可以捕捉处理try catch无法捕捉的异常。引入 Domain 模块 语法格式如下：

```
var domain = require("domain")
```

domain模块，把处理多个不同的IO的操作作为一个组。注册事件和回调到domain，当发生一个错误事件或抛出一个错误时，domain对象会被通知，不会丢失上下文环境，也不导致程序错误立即退出，与process.on('uncaughtException')不同。

Domain 模块可分为隐式绑定和显式绑定：

- 隐式绑定: 把在domain上下文中定义的变量，自动绑定到domain对象
- 显式绑定: 把不是在domain上下文中定义的变量，以代码的方式绑定到domain对象

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **domain.run(function)** 在域的上下文运行提供的函数，隐式的绑定了所有的事件分发器，计时器和底层请求。 |
| 2    | **domain.add(emitter)** 显式的增加事件                       |
| 3    | **domain.remove(emitter)** 删除事件。                        |
| 4    | **domain.bind(callback)** 返回的函数是一个对于所提供的回调函数的包装函数。当调用这个返回的函数时，所有被抛出的错误都会被导向到这个域的 error 事件。 |
| 5    | **domain.intercept(callback)** 和 domain.bind(callback) 类似。除了捕捉被抛出的错误外，它还会拦截 Error 对象作为参数传递到这个函数。 |
| 6    | **domain.enter()** 进入一个异步调用的上下文，绑定到domain。  |
| 7    | **domain.exit()** 退出当前的domain，切换到不同的链的异步调用的上下文中。对应domain.enter()。 |
| 8    | **domain.dispose()** 释放一个domain对象，让node进程回收这部分资源。 |
| 9    | **domain.create()** 返回一个domain对象。                     |

**属性**

| 序号 | 属性 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **domain.members** 已加入domain对象的域定时器和事件发射器的数组。 |

**实例**

创建 main.js 文件，代码如下所示：

```
var EventEmitter = require("events").EventEmitter;
var domain = require("domain");

var emitter1 = new EventEmitter();

// 创建域
var domain1 = domain.create();

domain1.on('error', function(err){
   console.log("domain1 处理这个错误 ("+err.message+")");
});

// 显式绑定
domain1.add(emitter1);

emitter1.on('error',function(err){
   console.log("监听器处理此错误 ("+err.message+")");
});

emitter1.emit('error',new Error('通过监听器来处理'));

emitter1.removeAllListeners('error');

emitter1.emit('error',new Error('通过 domain1 处理'));

var domain2 = domain.create();

domain2.on('error', function(err){
   console.log("domain2 处理这个错误 ("+err.message+")");
});

// 隐式绑定
domain2.run(function(){
   var emitter2 = new EventEmitter();
   emitter2.emit('error',new Error('通过 domain2 处理'));   
});


domain1.remove(emitter1);
emitter1.emit('error', new Error('转换为异常，系统将崩溃!'));
```

执行以上代码，结果如下所示:

```
监听器处理此错误 (通过监听器来处理)
domain1 处理这个错误 (通过 domain1 处理)
domain2 处理这个错误 (通过 domain2 处理)

events.js:72
        throw er; // Unhandled 'error' event
              ^
Error: 转换为异常，系统将崩溃!
    at Object.<anonymous> (/www/node/main.js:40:24)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Function.Module.runMain (module.js:497:10)
    at startup (node.js:119:16)
    at node.js:929:3
```