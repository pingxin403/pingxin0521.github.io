---
title: Node.JS web开发前置知识
date: 2021-04-01 11:18:59
tags:
 - Node.JS
 - 后端
categories:
 - 后端
 - Node.JS
---

web开发前置知识

1. http
2. 编码
3. 加密

<!--more-->

### base64编码

```js
console.log(Buffer.from('Hello').toString('base64'))  // SGVsbG8=
console.log(Buffer.from('SGVsbG8=', 'base64').toString('ascii'))  // hello
```

过程：

1. 用 `string` 新建一个 `Buffer`
2. 将 `Buffer` 解码成指定字符编码的 `string`

主要用到了 Buffer.from 的API，文档如下：

> Buffer.from(string[, encoding])
>
> 1. string <string/> 要编码的字符串
> 2. encoding <string/> string 的字符编码。 默认: 'utf8'
>
> buf.toString([encoding[, start[, end]]])
>
> - encoding <string/> 解码使用的字符编码。默认: 'utf8'
> - start <integer/> 开始解码的字节偏移量。默认: 0
> - end <integer/> 结束解码的字节偏移量（不包含）。 默认: buf.length
> - 返回: <string/>

在`NodeJS`中，`Buffer`是一个全局对象，所以使用时无需单独 `require`。在创建 `Buffer` 时，我们可以通过第二个参数，指明 `string`的编码类型（例如，`base64`）。

`buf.toString`时，也可以指定编码类型。默认为`utf8`。主要编码类型有 `ascii`、`utf8`、`ucs2`、`base64`、`binary`。

### md5和Hmac加密

在nodejs中，可以使用crypto模块来实现各种不同的加密与解密处理，在crypto模块中包含了类似MD5或SHA-1这些散列算法，我们可以通过crypto模块来实现HMAC运算。

HMAC的中文意思是：散列运算消息认证码；运算使用散列算法，以一个密钥和一个消息为输入，生成一个消息摘要作为输出。HMAC运算可以用来验证两段数据是否匹配，以确认该数据没有被篡改。

在crypto模块中，为每一种加密算法定义了一个类。可以使用getCiphers方法来查看nodejs中能够使用的所有加密算法。该方法返回一个数组，包括了nodejS中能够使用的所有散列算法。使用方法如下所示：

```js
const crypto = require('crypto');
console.log(crypto.getCiphers());
```

#### 散列算法

散列(也可以叫哈希)算法，它是用来对一段数据进行验证前，将该数据模糊化，或者也可以为一大段数据提供一个校验码。
在nodejs中，为了使用该散列算法，我们先要使用 createHash方法创建一个hash对象。使用方法如下：

**crypto.createHash(params);**

在如上方法中，需要使用一个参数，其参数值为一个在Node.js中可以使用的算法，比如 'sha1', 'md5', 'sha512' 等等这样的，用于指定需要使用的散列算法，该方法返回被创建的hash对象。

在创建完hash对象后，可以通过使用该对象的update方法创建一个摘要。该方法的使用方式如下：

**hash.update(data, [encoding]);**

在如上面的方法，该方法需要使用两个参数，第一个参数是必选项，该参数值是一个Buffer对象或一个字符串，用于指定摘要内容； 第二个参数 encoding用于指定摘要内容所需使用的编码格式，可以指定为 'utf-8', 'ascii', 或 'binary'. 如果不使用第二个参数，则第一个参数data参数值必须为一个Buffer对象，我们也可以在摘要被输出前使用多次updata方法来添加摘要内容。

在第一步创建了一个hash对象后，第二步就是添加摘要内容，那么第三步我们就是使用hash对象的digest方法来输出摘要内容了；在使用hash对象的digest
方法后，不能再向hash对象中追加摘要内容，也就是说你使用了digest方法作为输出后，你再追加内容也不会执行，因此是不能向hash对象中追加内容。
使用方法如下：

**hash.digest([encoding]);**

该方法有一个参数，该参数是一个可选值，表示的意思是 用于指定输出摘要的编码格式，可指定参数值为 'hex', 'binary', 及 'base64'.如果使用了该参数，那么digest方法返回字符串格式的摘要内容，如果不使用该参数，那么digest方法返回一个是Buffer对象。

#### MD5算法

MD5是计算机领域使用最广泛的散列函数(可以叫哈希算法、摘要算法)，注意是用来确保消息的完整和一致性。

下面我们最主要是以 md5 加密为例来了解下加密算法。
MD5算法有以下特点：

1. 压缩性： 任意长度的数据，算出的MD5值长度都是固定的。
2.  容易计算：从原数据算出MD5值很容易。
3.  抗修改性：对原数据进行任何改动，哪怕只修改一个字节，所得到的MD5值都有很大的区别。
4. 强抗碰撞：已知原数据和其MD5值，想找到一个具有相同的MD5值的伪数据是非常困难的。

MD5的作用是让大容量信息在用数字签名软件签署私人秘钥前被压缩成一种保密的格式(就是把任意长度的字符串变换成一定长的十六进制数字串)。

```js
const crypto = require('crypto');

const str = 'abc';

// 创建一个hash对象
const md5 = crypto.createHash('md5');

// 往hash对象中添加摘要内容
md5.update(str);

// 使用 digest 方法输出摘要内容，不使用编码格式的参数 其输出的是一个Buffer对象
// console.log(md5.digest()); 
// 输出 <Buffer 90 01 50 98 3c d2 4f b0 d6 96 3f 7d 28 e1 7f 72>

// 使用编码格式的参数，输出的是一个字符串格式的摘要内容
console.log(md5.digest('hex')); // 输出 900150983cd24fb0d6963f7d28e17f72
```

只对md5加密的缺点：

通过上面对md5加密后确实比明文好很多，至少很多人直接使用肉眼看到的并记不住，也不知道密码多少，但是只对md5加密也存在缺点，如上代码使用console.log打印两次后，加密后的代码是一样，也就是说 相同的明文密码，加密后，输出两次，md5的值也是一样的。 所以这样也是不安全的。

**密码加盐：**

什么意思呢？就是在密码的特定位置上插入特定的字符串后，再对修改后的字符串进行md5加密，这样做的好处是每次调用代码的时候，插入的字符串不一样，会导致最后的md5值会不一样的。代码如下：

```js
const crypto = require('crypto');

var saltPasswordFunc = function(password, salt) {

  // 密码加盐
  const saltPassword = password + ':' + salt;
  console.log('原始密码：%s', password); 
  console.log('加盐后的密码：%s', saltPassword);


  const md5 = crypto.createHash('md5');
  const result = md5.update(saltPassword).digest('hex');
  console.log('加盐密码后的md5的值为：%s', result);

};

const password = '123456';

saltPasswordFunc(password, 'abc');
/*
 输出结果为：
 原始密码：123456
 加盐后的密码：123456:abc
 加盐密码后的md5的值为：51011af1892f59e74baf61f3d4389092
*/

saltPasswordFunc(password, 'def');
/*
 输出结果为：
 原始密码：123456
 加盐后的密码：123456:def
 加盐密码后的md5的值为：5091d17d6b08ba9a95ccef51f598b249
*/
```

#### HMAC算法

HMAC算法是将散列算法与一个密钥结合在一起，以阻止对签名完整性破坏，其实就是类似于上面的提到的md5密码中加盐道理是类似的。
使用HMAC算法前，我们使用createHmac方法创建一个hmac对象，创建方法如下所示：

**crypto.createHmac(params, key);**

该方法中使用两个参数，第一个参数含义是在Node.js中使用的算法，比如'sha1', 'md5', 'sha256', 'sha512'等等，该方法返回的是hmac对象。
key参数值为一个字符串，用于指定一个PEM格式的密钥。

在创建完成hmac对象后，我们也是一样使用一个update方法来创建一个摘要，该方法使用如下所示：

**hmac.update(data);**

在update方法中，使用一个参数，其参数值为一个Buffer对象或一个字符串，用于指定摘要内容。也是一样可以在被输出之前使用多次update方法来添加摘要内容。

最后一步就是 使用hmac对象的digest方法来输出摘要内容了；在使用hmac对象的digest方法后，不能再向hmac对象中追加摘要内容，也就是说你使用了digest方法作为输出后，因此是不能向hmac对象中追加内容。使用方法如下：

**hmac.digest([encoding]);**

该方法有一个参数，该参数是一个可选值，表示的意思是 用于指定输出摘要的编码格式，可指定参数值为 'hex', 'binary', 及 'base64'.
如果使用了该参数，那么digest方法返回字符串格式的摘要内容，如果不使用该参数，那么digest方法返回一个是Buffer对象。

```js
const crypto = require('crypto');

// 创建一个hmac对象
const hmac = crypto.createHmac('md5', 'abc');

// 往hmac对象中添加摘要内容
const up = hmac.update('123456');

// 使用 digest 方法输出摘要内容

const result = up.digest('hex'); 

console.log(result); // 8c7498982f41b93eb0ce8216b48ba21d
```

###  多进程

我们都知道 Node.js 是以单线程的模式运行的，但它使用的是事件驱动来处理并发，这样有助于我们在多核 cpu 的系统上创建多个子进程，从而提高性能。

每个子进程总是带有三个流对象：child.stdin, child.stdout 和child.stderr。他们可能会共享父进程的 stdio 流，或者也可以是独立的被导流的流对象。

Node 提供了 child_process 模块来创建子进程，方法有：

- **exec** - child_process.exec 使用子进程执行命令，缓存子进程的输出，并将子进程的输出以回调函数参数的形式返回。
- **spawn** - child_process.spawn 使用指定的命令行参数创建新进程。
- **fork** - child_process.fork 是 spawn()的特殊形式，用于在子进程中运行的模块，如 fork('./son.js') 相当于 spawn('node', ['./son.js']) 。与spawn方法不同的是，fork会在父进程与子进程之间，建立一个通信管道，用于进程之间的通信。

##### exec() 方法

child_process.exec 使用子进程执行命令，缓存子进程的输出，并将子进程的输出以回调函数参数的形式返回。

语法如下所示：

```
child_process.exec(command[, options], callback)
```

**参数**

参数说明如下：

**command：** 字符串， 将要运行的命令，参数使用空格隔开

**options ：对象，可以是：**

- cwd ，字符串，子进程的当前工作目录
- env，对象 环境变量键值对
- encoding ，字符串，字符编码（默认： 'utf8'）
- shell ，字符串，将要执行命令的 Shell（默认: 在 UNIX 中为`/bin/sh`， 在 Windows 中为`cmd.exe`， Shell 应当能识别 `-c`开关在 UNIX 中，或 `/s /c` 在 Windows 中。 在Windows 中，命令行解析应当能兼容`cmd.exe`）
- timeout，数字，超时时间（默认： 0）
- maxBuffer，数字， 在 stdout 或 stderr 中允许存在的最大缓冲（二进制），如果超出那么子进程将会被杀死 （默认: 200*1024）
- killSignal ，字符串，结束信号（默认：'SIGTERM'）
- uid，数字，设置用户进程的 ID
- gid，数字，设置进程组的 ID

**callback ：**回调函数，包含三个参数error, stdout 和 stderr。

exec() 方法返回最大的缓冲区，并等待进程结束，一次性返回缓冲区的内容。

让我们创建两个 js 文件 support.js 和 master.js。

```js
//support.js
console.log("进程 " + process.argv[2] + " 执行。" );
//master.js 文件代码：
const fs = require('fs');
const child_process = require('child_process');
 
for(var i=0; i<3; i++) {
    var workerProcess = child_process.exec('node support.js '+i, function (error, stdout, stderr) {
        if (error) {
            console.log(error.stack);
            console.log('Error code: '+error.code);
            console.log('Signal received: '+error.signal);
        }
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
    });
 
    workerProcess.on('exit', function (code) {
        console.log('子进程已退出，退出码 '+code);
    });
}
```

执行以上代码，输出结果为：

```
$ node master.js 
子进程已退出，退出码 0
stdout: 进程 1 执行。

stderr: 
子进程已退出，退出码 0
stdout: 进程 0 执行。

stderr: 
子进程已退出，退出码 0
stdout: 进程 2 执行。

stderr: 
```

##### spawn() 方法

child_process.spawn 使用指定的命令行参数创建新进程，语法格式如下：

```
child_process.spawn(command[, args][, options])
```

**参数**

参数说明如下：

**command：** 将要运行的命令

**args：** Array 字符串参数数组

**options Object**

- cwd String 子进程的当前工作目录
- env Object 环境变量键值对
- stdio Array|String 子进程的 stdio 配置
- detached Boolean 这个子进程将会变成进程组的领导
- uid Number 设置用户进程的 ID
- gid Number 设置进程组的 ID

spawn() 方法返回流 (stdout & stderr)，在进程返回大量数据时使用。进程一旦开始执行时 spawn() 就开始接收响应。

```js
//support.js
console.log("进程 " + process.argv[2] + " 执行。" );
//master.js 文件代码：
const fs = require('fs');
const child_process = require('child_process');
 
for(var i=0; i<3; i++) {
   var workerProcess = child_process.spawn('node', ['support.js', i]);
 
   workerProcess.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
   });
 
   workerProcess.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
   });
 
   workerProcess.on('close', function (code) {
      console.log('子进程已退出，退出码 '+code);
   });
}
```

执行以上代码，输出结果为：

```
$ node master.js stdout: 进程 0 执行。

子进程已退出，退出码 0
stdout: 进程 1 执行。

子进程已退出，退出码 0
stdout: 进程 2 执行。

子进程已退出，退出码 0
```

##### fork 方法

child_process.fork 是 spawn() 方法的特殊形式，用于创建进程，语法格式如下：

```
child_process.fork(modulePath[, args][, options])
```

**参数**

参数说明如下：

**modulePath**： String，将要在子进程中运行的模块

**args**： Array 字符串参数数组

**options**：Object

- cwd String 子进程的当前工作目录
- env Object 环境变量键值对
- execPath String 创建子进程的可执行文件
- execArgv Array 子进程的可执行文件的字符串参数数组（默认： process.execArgv）
- silent Boolean 如果为`true`，子进程的`stdin`，`stdout`和`stderr`将会被关联至父进程，否则，它们将会从父进程中继承。（默认为：`false`）
- uid Number 设置用户进程的 ID
- gid Number 设置进程组的 ID

返回的对象除了拥有ChildProcess实例的所有方法，还有一个内建的通信信道。

```js
//support.js
console.log("进程 " + process.argv[2] + " 执行。" );
//master.js 文件代码：
const fs = require('fs');
const child_process = require('child_process');
 
for(var i=0; i<3; i++) {
   var worker_process = child_process.fork("support.js", [i]);    
 
   worker_process.on('close', function (code) {
      console.log('子进程已退出，退出码 ' + code);
   });
}
```

执行以上代码，输出结果为：

```
$ node master.js 
进程 0 执行。
子进程已退出，退出码 0
进程 1 执行。
子进程已退出，退出码 0
进程 2 执行。
子进程已退出，退出码 0
```

#### 连接数据库

使用了淘宝定制的 cnpm 命令进行安装：

```
$ cnpm install mysql
```

在以下实例中根据你的实际配置修改数据库用户名、及密码及数据库名：

```js
// 引入 mysql 数据库连接依赖
const mysql = require("mysql");

// 创建 mysql 连接池并配置参数
const pool = mysql.createPool({
    host: "127.0.0.1",    // 主机地址
    port: 3306,                 // 端口
    user: "root",               // 数据库访问账号
    password: "920619",         // 数据库访问密码
    database: "test",           // 要访问的数据库
    charset: "UTF8_GENERAL_CI", // 字符编码 ( 必须大写 )
    typeCast: true,             // 是否把结果值转换为原生的 javascript 类型
    supportBigNumbers: true,    // 处理大数字 (bigint, decimal), 需要开启 ( 结合 bigNumberStrings 使用 )
    bigNumberStrings: true,     // 大数字 (bigint, decimal) 值转换为javascript字符对象串
    multipleStatements: false,  // 允许每个mysql语句有多条查询, 未防止sql注入不开启
    //connectTimeout: 5000,     // 数据库连接超时时间, 默认无超时
});
pool.connectionLimit = 10;      // 连接池中可以存放的最大连接数量
pool.waitForConnections = true; // 连接使用量超负荷是否等待, false 会报错
pool.queueLimit = 0;            // 每个连接可操作的 列数 上限, 0 为没有上限

// 对外暴漏从连接池中获取数据库连接的方法
module.exports = function () {
    return new Promise((resolve, reject) => {
        pool.getConnection((err, conn) => {
            if (err) {
                console.log("数据库连接获取失败");
            } else {
                resolve(conn);
            }
        });
    });
};
```

连接参数

| 参数               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| host               | 主机地址 （默认：localhost）                                 |
| user               | 用户名                                                       |
| password           | 密码                                                         |
| port               | 端口号 （默认：3306）                                        |
| database           | 数据库名                                                     |
| charset            | 连接字符集（默认：'UTF8_GENERAL_CI'，注意字符集的字母都要大写） |
| localAddress       | 此IP用于TCP连接（可选）                                      |
| socketPath         | 连接到unix域路径，当使用 host 和 port 时会被忽略             |
| timezone           | 时区（默认：'local'）                                        |
| connectTimeout     | 连接超时（默认：不限制；单位：毫秒）                         |
| stringifyObjects   | 是否序列化对象                                               |
| typeCast           | 是否将列值转化为本地JavaScript类型值 （默认：true）          |
| queryFormat        | 自定义query语句格式化方法                                    |
| supportBigNumbers  | 数据库支持bigint或decimal类型列时，需要设此option为true （默认：false） |
| bigNumberStrings   | supportBigNumbers和bigNumberStrings启用 强制bigint或decimal列以JavaScript字符串类型返回（默认：false） |
| dateStrings        | 强制timestamp,datetime,data类型以字符串类型返回，而不是JavaScript Date类型（默认：false） |
| debug              | 开启调试（默认：false）                                      |
| multipleStatements | 是否许一个query中有多个MySQL语句 （默认：false）             |
| flags              | 用于修改连接标志                                             |
| ssl                | 使用ssl参数（与crypto.createCredenitals参数格式一至）或一个包含ssl配置文件名称的字符串，目前只捆绑Amazon RDS的配置文件 |

更多说明可参见：https://github.com/mysqljs/mysql

执行简单的操作测试

```js
// 引入工具类
let mysqlUtil = require("./mysql-util");

// 声明一个同步方法, 这里要求有 SQL 语句的基础
let ceshi = async function () {
    // 获取连接, es6 新特性 awit 不能少
    let conn = await mysqlUtil();
    // 准备一个防 SQL 注入的 SQL 语句, 并准备参数
    let sqlStr = "INSERT INTO user (UserName, UserSex) value (?, ?)";
    let sqlParam = ["姓名", 1];
    // 执行 SQL 语句并返回结果
    conn.query(sqlStr, sqlParam, function (err, ret) {
        console.log(ret);
　　　　　conn.release();　
    });
};

// 调用方法
ceshi();
```

执行带事务的操作测试

```js
// 引入工具类
let mysqlUtil = require("./mysql-util");

// 声明一个同步方法, 这里要求有 SQL 语句的基础
// 在 beginTransaction 和 commit 之间可以执行多次 query 方法
let ceshi = async function () {
    // 获取连接
    let conn = await mysqlUtil();
    // 开启事物
    await new Promise((resolve, reject) => {
        conn.beginTransaction(err => {
            if (err) {
                reject(err);
            } else {
                resolve();
            }
        });
    });
    // 执行第一个 SQL 语句
    let result1 = await new Promise((resolve, reject) => {
        let sqlStr = "INSERT INTO user (UserName, UserSex) values (?, ?)";
        let sqlParam = ["姓名2", 0];
        conn.query(sqlStr, sqlParam, function (err, ret) {
            if (err) {
                // 回滚之前的数据库操作, 直至碰到 beginTransaction
                return conn.rollback(() => {
                    resolve(err);
                });
            }
            resolve(ret);
        });
    });
    console.log(result1);
    // 执行第二个 SQL 语句
    let result2 = await new Promise((resolve, reject) => {
        let sqlStr = "SELECT * FROM user";
        //let sqlStr = "SELEC * FROM user"; // 错误的 SQL 语句
        conn.query(sqlStr, function (err, ret) {
            if (err) {
                // 回滚之前的数据库操作, 直至碰到 beginTransaction
                return conn.rollback(() => {
                    resolve(err);
                });
            }
            resolve(ret);
        });
    });
    console.log(result2);
    // 关闭事务
    await new Promise((resolve, reject) => {
        conn.commit(err => {
            if (err) {
                reject(err);
            } else {
                resolve();
            }
        });
    });
　　 conn.release();　　
};

// 调用测试方法
ceshi();
```

#### 连接 MongoDB

使用了[淘宝定制的 cnpm 命令](https://www.runoob.com/nodejs/nodejs-npm.html#taobaonpm)进行安装：

```
$ cnpm install mongodb -g
```

操作

```js
 // 引入模块
var mongoose=require('mongoose');

// 连接数据库
mongoose.connect('mongodb://localhost:27017/users')

// 得到数据库连接句柄
var db=mongoose.connection;

//通过 数据库连接句柄，监听mongoose数据库成功的事件
db.on('open',function(err){
    if(err){
        console.log('数据库连接失败');
        throw err;
    }
    console.log('数据库连接成功')
    })
    
    //定义表数据结构
    var userModel=new mongoose.Schema({
        id:Number,
        nickname:String,
        mobile:String,
        password:String
    
    },{
        versionKey:false //去除： - -v
    })
    
    // 将表的数据结构和表关联起来
    // var productModel=mongoose.model('anyname',表的数据结构，表名)
    var userModel=mongoose.model("userList",userModel,"userList");
    
    
    userList=[
        {id:0,nickname:"pwl",mobile:"15556930270",password:"123456"},
        {id:1,nickname:"ws",mobile:"15556931933",password:"123456"},
        {id:2,nickname:"yl",mobile:"15556930268",password:"123456"}
    
    ]
    
    // 添加数据（添加完数据可以在隐藏起来）
     userModel.insertMany(userList,function(err,result){
        if(err){
            console.log("数据添加失败");
            throw err;
        }
        console.log("数据添加成功:",result);
     })
    
    // 删除数据
     userModel.remove({},function(err){
      if(err){
          console.log('删除数据失败');
          throw err;
      }
      console.log("删除数据成功");
     })
    
    //导出数据
    module.exports={
        userModel:userModel
    }
```
