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

简单的说 Node.js 就是运行在服务端的 JavaScript。

Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。

Node.js 是一个基于 **Chrome V8 引擎**的 **JavaScript 运行环境**。Node.js 使用了一个**事件驱动、非阻塞式 I/O** 的模型，使其轻量又高效。

![2.jpeg](https://i.loli.net/2019/08/24/UEAexu7iaZjMYQk.jpg)

[中文文档](http://nodejs.cn/api/)

<!--more-->

Node.js还提供了各种丰富的JavaScript模块库，它极大简化了使用Node.js来扩展Web应用程序的研究与开发。

Node.js = 运行环境+ JavaScript库

#### 简介

##### Node.js特性

- Node.js库的异步和事件驱动的API全部都是异步就是非阻塞。它主要是指基于Node.js的服务器不会等待API返回的数据。服务器移动到下一个API调用，Node.js发生的事件通知机制后有助于服务器获得从之前的API调用的响应。
- 非常快的内置谷歌Chrome的V8 JavaScript引擎，Node.js库代码执行是非常快的。
- 单线程但高度可扩展 - Node.js使用具有循环事件单线程模型。事件机制有助于服务器在一个非阻塞的方式响应并使得服务器高度可扩展，而不是创建线程限制来处理请求的传统服务器。Node.js使用单线程的程序，但可以提供比传统的服务器(比如Apache HTTP服务器)的请求服务数量要大得多。
- 没有缓冲 - Node.js的应用从来不使用缓冲任何数据。这些应用只是输出数据在块中。
- 许可证协议 - Node.js 在 [MIT 协议](https://raw.githubusercontent.com/joyent/node/v0.12.0/LICENSE) 下发布

##### 在哪里可以使用Node.js？

以下是Node.js证明自己完美的技术的合作伙伴的领域。

-  			I/O 密集型应用程序
-  			数据流应用
-  			数据密集型实时应用(DIRT)
-  			JSON API的应用程序
-  			单页面应用

##### 在哪些地方不要使用Node.js？

不建议使用Node.js的就是针对**CPU密集型应用**。

##### Node服务器和java服务起的对比

**运行时环境**

我们众所周知Java具有一个称作JRE的运行时环境来使得java程序能够顺利运行。JRE有一个称为JVM的虚拟机。JVM有许多组件，如垃圾回收器（GC），即时（JIT）编译器，解释器，类装载器，线程管理器，异常处理器，用于在不同时间执行不同的任务。JRE还有一系列的库来帮助运行时的Java程序。

我们为什么要突然牵扯到JRE运行时环境呢，其实正是为了与Node作比较，Node不是一种语言，也不是框架，更不是工具，它是运行JavaScript应用程序的运行时环境。Node.js有一个称为JavaScript Virtual Machine的虚拟机。它为基于JavaScript的应用程序生成机器代码，以便在不同的平台上启用它。这个虚拟机就是Google的V8引擎，也有主要组件，如JIT和GC，分别用于执行任务，运行时编译，和内存管理。

![1.jpeg](https://i.loli.net/2019/08/24/8QBq9hFiOGjeMHE.jpg)

**发展潜力**

判断Java和Node的发展潜力可能要从其背后的生态社区和支持库上切入，然而以Java为核心的传统体系自然比不上Node这样的新势力，简而言之，Java成熟而庞大，Node迅捷而活跃。Java其功能性和实用性自然不必多说，但是Java包含了大量的样品代码，扰乱了程序猿所想表达的意图，就不如Java三大框架之一的spring，程序猿在使用spring的时候servlet,数据持久，以及构成系统的底层的东西，spring框架已经封装好会帮助你处理这一切，我们只需要专注于写业务层代码就足以。但是在Spring中，子系统一个接一个，哪怕你犯最微小的错误，它都会用让你崩溃的异常来惩罚你。可能紧接着你就会看到巨大的异常信息。里面包含着一个一个你根本不知道的封装好的方法，Spring做了许多工作来实现代码的功能。这种级别的抽象显然需要大量的逻辑，长长的异常信息不一定是坏事，它指出了一个症状：这需要多少内存和性能上的额外开销？spring是怎么执行的？框架需要解析方法名、猜测程序员的意图、构建类似于抽象语法树的东西、生成SQL等等。这些事情的额外开销有多大？所以说使用Java掩盖复杂性并不会因此简单化，只会让系统更复杂。Java严格的类型检查使得Java帮你避免许多类型的bug，因为不好的代码无法通过编译。Java的强类型的缺点就是太多样板代码。程序员要不断进行类型转换，程序员要花掉更多时间写精确的代码，使用更多的样板代码，以图早期发现错误并改正。

而Node.js恰恰相反。线程会导致更复杂化的系统。所以Node.js采用轻量级，单线程的系统，利用了js的匿名函数进行异步回调，你只需要简单的使用匿名函数，也就是闭包。不需要搜索正确的抽象接口，只需要写下业务代码，没有任何冗余。这就是使用Node.js的最大好处，不过异步回调自然也出现一个急需解决的问题：回调陷阱。

在Node.js中，我们不断嵌套回调函数的同时，很容易就陷入回调函数的陷阱中，每层嵌套都会让代码更复杂，使得错误处理和结果处理更困难。一个相关的问题就是js语言不会帮助程序员恰当地表达异步执行。其实有些库会使用Promise来简化异步操作，但是看起来我们把问题简单化了，但是事实上代码层面更复杂化了，Promise用了许多样板代码，掩盖了程序员的真实意图。后来Node.js支持ES5与ES6，可以采用async/await函数重写回调函数。还是同样的异步结构，但使用了正常的循环结构来书写。错误和结果处理的位置也很自然，代码更易于理解，更容易编写，而且也可以很容易地理解程序员的意图。回调陷阱并不是用掩盖复杂性的方式解决的。相反，语言和范式的改变解决了回调陷阱的问题，同时还解决了过多样板代码的问题。有了async函数，代码就更漂亮了。简单化的解决方法，将Node.js的缺点转化为了优点。

但是JavaScript的类型很松散。而且在你书写代码的时候不会进行报错，许多类型不需要定义，通常也不需要用类型转换。因此代码更清晰易读，但存在漏掉编码错误的风险，只有在编译的时候才会去检查你语法以及逻辑是否存在问题，所以在Node.js中，为了更好的调试BUG，Node支持将程序分成不同的模块，因为有模块的存在，将错误发生的范围缩小到某个范围内，使得Node.js模块更容易测试。

**包管理**

Java最重要的问题之一就是没有统一的包管理系统，可能有人会和我说Maven.但是无论是用途、易用性还是功能上，Maven与Node.js的包管理系统相比简直是天壤之别。npm 是 Node.js 官方提供的包管理工具，他已经成了 Node.js 包的标准发布平台，用于 Node.js 包的发布、传播、依赖控制。npm 提供了命令行工具，使你可以方便地下载、安装、升级、删除包，也可以让你作为开发者发布并维护包。最好的地方是npm代码库不仅供Node.js使用，也可以让前端工程师使用。所有的前端JavaScript库都以npm包的形式存在。许多前端工具如Webpack都是用Node.js编写的。

**性能**

Java使用HotSpot这个超级虚拟机，它采用了多字节编译策略。它会检测经常执行的代码，一段代码执行次数越多，就会应用越多的优化。因此HotSpot性能相对来说更快。Node底层选择用c++和v8引擎来实现的，Node.js的事件驱动机制，这意味着要面对大规模的http请求，Node.js是凭借事件驱动来完成的，性能部分是不用担心的，并且很出色。而且，由于V8引擎的改进，Node.js的每次发布都会带来巨大的性能提升。

虽然Node对高并发应用有着极高的性能，但是Node.js也有着自己的缺点：

- Node不适合CPU密集型应用，因为CPU密集型应用如果有长时间的运算，不如大循环，将会导致CPU时间片不能释放，使得后续的IO操作全部暂停。
- 而且Node只支持单核CPU，不能充分利用CPU资源.
  可靠性低，一旦代码某个环节崩溃，将会导致整个系统都崩溃，原因就在于Node是使用单进程。
  Node的开源组件库质量参差不齐，更新快，而且不向下兼容。
- 其实Node.js作为后端能实现几乎所有应用，只是我们选择的时候考虑更多的是这个应用选择Node.js是不是最适合的。

#### 模块化和组件化

组件化与模块化已经深入体现到软件开发当中，也是为了让开发者更好的去解决软件上的**高耦合、低内聚、无重用**的3大代码问题。很多人在实践过程中也存在不少疑惑（很多时候不知道选择用组件还是模块，或者有时候根本分不清自己这得是组件还是模块。云里雾里~~）。因此为了解决这些疑惑帮自己统一了组件化与模块化的使用方式与概念、定位。

##### 组件化

就是"基础库"或者“基础组件"，意思是把代码重复的部分提炼出一个个组件供给功能使用。

- 使用：Dialog，各种自定义的UI控件、能在项目或者不同项目重复应用的代码等等。

- 目的：复用，解耦。
- 依赖：组件之间低依赖，比较独立。
- 架构定位：纵向分层（位于架构底层，被其他层所依赖）。

##### 模块化 

就是"业务框架"或者“业务模块"，也可以理解为“框架”，意思是把功能进行划分，将同一类型的代码整合在一起，所以模块的功能相对复杂，但都同属于一个业务。

- 使用：按照项目功能需求划分成不同类型的业务框架（例如：注册、登录、外卖、直播.....）
- 目的：隔离/封装 （高内聚）。
- 依赖：模块之间有依赖的关系，可通过路由器进行模块之间的耦合问题。
- 架构定位：横向分块（位于架构业务框架层）。

其实组件相当于库，把一些能在项目里或者不同类型项目中可复用的代码进行工具性的封装。

而模块相应于业务逻辑模块，把同一类型项目里的功能逻辑进行进行需求性的封装。

**更多参考**：<http://outofmemory.cn/html/front-end-project-component>

#### 安装

##### 包安装

环境：Ubuntu 18.04

1. 安装

```bash
$ sudo apt-get install nodejs
$ sudo apt-get install npm
```

2. 更改镜像源

   ```bash
   #得到原本的镜像地址
   $ npm get registry 
   > https://registry.npmjs.org/
   #设成淘宝的
   $ npm config set registry http://registry.npm.taobao.org/
   ```

3. 升级

   ```bash
   #升级npm
   $ sudo npm install npm -g
   #升级node.js，n为版本号，
   #n latest(升级node.js到最新版)  or $ n stable（升级node.js到最新稳定版）
   #n后面也可以跟随版本号比如：$ n v0.10.26 或者 $ n 0.10.26
   $ sudo npm install -g n
   $ sudo n latest
   
   ```

4. 选装cnpm

   因为npm安装插件是从国外服务器下载，受网络影响大，可能出现异常，如果npm的服务器在中国就好了，所以我们乐于分享的淘宝团队干了这事。！来自官网：“这是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10分钟 一次以保证尽量与官方服务同步。”；

   **官方网址：http://npm.taobao.org**

   安装：命令提示符执行`npm install cnpm -g --registry=https://registry.npm.taobao.org`；  注意：安装完后最好查看其版本号cnpm -v或关闭命令提示符重新打开，安装完直接使用有可能会出现错误；

   注：cnpm跟npm用法完全一致，只是在执行命令时将npm改为cnpm（以下操作将以cnpm代替npm）

5. 全局安装与本地安装

   npm 的包安装分为本地安装（local）、全局安装（global）两种，从敲的命令行来看，差别只是有没有-g而已，比如我们使用 npm 命令安装常用的 Node.js web框架模块 express:

   ```
   $ npm install express          # 本地安装
   $ npm install express -g       # 全局安装
   ```

##### 源码安装

 以下部分我们将介绍在 Ubuntu Linux 下使用源码安装 Node.js 。 其他的 Linux 系统，如 Centos 等类似如下安装步骤。 

 在 [中文官网](http://nodejs.cn/)上获取 Node.js 源码：

使用 **./configure** 创建编译文件，并按照：

```
$ cd node
$ ./configure  --prefix=/opt/node
$ make
$ sudo make install
```

查看 node 版本：

```
$ node -v
```

#### 创建第一个应用

如果我们使用PHP来编写后端的代码时，需要Apache 或者 Nginx 的HTTP 服务器，并配上 mod_php5 模块和php-cgi。

 从这个角度看，整个"接收 HTTP 请求并提供 Web 页面"的需求根本不需 要 PHP 来处理。

 不过对 Node.js 来说，概念完全不一样了。使用 Node.js 时，我们不仅仅 在实现一个应用，同时还实现了整个 HTTP 服务器。事实上，我们的 Web 应用以及对应的 Web 服务器基本上是一样的。 

在我们创建 Node.js 第一个 "Hello, World!" 应用前，让我们先了解下 Node.js 应用是由哪几部分组成的：

1. **引入 required 模块：**我们可以使用 **require** 指令来载入 Node.js 模块。
2. **创建服务器：**服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
3. **接收请求与响应请求** 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。

步骤一、引入 required 模块

我们使用 **require** 指令来载入 http 模块，并将实例化的 HTTP 赋值给变量 http，实例如下: 

```
var http = require("http");
```

步骤二、创建服务器

接下来我们使用 http.createServer() 方法创建服务器，并使用 listen 方法绑定 8888 端口。 函数通过 request, response 参数来接收和响应数据。

实例如下，在你项目的根目录下创建一个叫 server.js 的文件，并写入以下代码：

```
var http = require('http');

http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```

以上代码我们完成了一个可以工作的 HTTP 服务器。

使用 **node** 命令执行以上的代码：

```
node server.js
Server running at http://127.0.0.1:8888/
```

 接下来，打开浏览器访问 http://127.0.0.1:8888/，你会看到一个写着 "Hello World"的网页。  

**分析Node.js 的 HTTP 服务器：**

-  第一行请求（require）Node.js 自带的  http  模块，并且把它赋值给  http 变量。
-   接下来我们调用 http 模块提供的函数：  createServer  。这个函数会返回 一个对象，这个对象有一个叫做  listen  的方法，这个方法有一个数值参数， 指定这个 HTTP 服务器监听的端口号。 

#### NPM 包管理器使用介绍

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。 
-  允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。 
-  允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

 由于新版的nodejs已经集成了npm，所以之前npm也一并安装好了。同样可以通过输入 **"npm -v"** 来测试是否成功安装。出现版本提示表示安装成功

##### 使用 npm 命令安装模块

npm 安装 Node.js 模块语法格式如下：

```
$ npm install <Module Name>
```

以下实例，我们使用 npm 命令安装常用的 Node.js web框架模块 **express**:

```
$ npm install express
```

 安装好之后，express 包就放在了工程目录下的 node_modules 目录中，因此在代码中只需要通过 **require('express')** 的方式就好，无需指定第三方包路径。 

```
var express = require('express');
```

##### 全局安装与本地安装

 npm 的包安装分为本地安装（local）、全局安装（global）两种，从敲的命令行来看，差别只是有没有-g而已，比如 

```
npm install express          # 本地安装
npm install express -g   # 全局安装
```

如果出现以下错误：

```
npm err! Error: connect ECONNREFUSED 127.0.0.1:8087 
```

解决办法为：

```
$ npm config set proxy null
```

**本地安装**

1. 将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。 

2. 可以通过 require() 来引入本地安装的包。 

 **全局安装**

1. 将安装包放在 /usr/local 下或者你 node 的安装目录。

2. 可以直接在命令行里使用。

如果你希望具备两者功能，则需要在两个地方安装它或使用 **npm link**。

接下来我们使用全局方式安装 express

```
$ npm install express -g
```

你可以使用以下命令来查看所有全局安装的模块：

```
$ npm list -g
```

如果要查看某个模块的版本号，可以使用命令如下：

```
$ npm list grunt
```

##### 使用 package.json

package.json 位于模块的目录下，用于定义包的属性。接下来让我们来看下 express 包的 package.json 文件，位于 node_modules/express/package.json 内容

**Package.json 属性说明**

- **name** - 包名。
- **version** - 包的版本号。
- **description** - 包的描述。
- **homepage** - 包的官网 url 。
- **author** - 包的作者姓名。
- **contributors** - 包的其他贡献者姓名。
- **dependencies** - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。
- **repository** - 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。
- **main** - main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。
- **keywords** - 关键字

##### 卸载模块

 我们可以使用以下命令来卸载 Node.js 模块。

```
$ npm uninstall express
```

卸载后，你可以到 /node_modules/ 目录下查看包是否还存在，或者使用以下命令查看：

```
$ npm ls
```

##### 更新模块

我们可以使用以下命令更新模块：

```
$ npm update express
```

##### 搜索模块

 使用以下来搜索模块：

```
$ npm search express
```

##### 创建模块

创建模块，package.json 文件是必不可少的。我们可以使用 NPM 生成 package.json 文件，生成的文件包含了基本的结果。

```
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (node_modules) runoob                   # 模块名
version: (1.0.0) 
description: Node.js 测试模块(www.runoob.com)  # 描述
entry point: (index.js) 
test command: make test
git repository: https://github.com/runoob/runoob.git  # Github 地址
keywords: 
author: 
license: (ISC) 
About to write to ……/node_modules/package.json:      # 生成地址

{
  "name": "runoob",
  "version": "1.0.0",
  "description": "Node.js 测试模块(www.runoob.com)",
  ……
}


Is this ok? (yes) yes
```

以上的信息，你需要根据你自己的情况输入。在最后输入 "yes" 后会生成 package.json 文件。

  接下来我们可以使用以下命令在 npm 资源库中注册用户（使用邮箱注册）：

```
$ npm adduser
Username: mcmohd
Password:
Email: (this IS public) mcmohd@gmail.com
```

接下来我们就用以下命令来发布模块：

```
$ npm publish
```

如果你以上的步骤都操作正确，你就可以跟其他模块一样使用 npm 来安装。

##### 版本号

 使用NPM下载和发布代码时都会接触到版本号。NPM使用语义版本号来管理代码，这里简单介绍一下。 

 语义版本号分为X.Y.Z三位，分别代表主版本号、次版本号和补丁版本号。当代码变更时，版本号按以下原则更新。 

-  如果只是修复bug，需要更新Z位。
-   如果是新增了功能，但是向下兼容，需要更新Y位。
-   如果有大变动，向下不兼容，需要更新X位。

 版本号有了这个保证后，在申明第三方包依赖时，除了可依赖于一个固定版本号外，还可依赖于某个范围的版本号。例如"argv": "0.0.x"表示依赖于0.0.x系列的最新版argv。

NPM支持的所有版本号范围指定方式可以查看[官方文档](https://npmjs.org/doc/files/package.json.html#dependencies)。 

##### NPM 常用命令

除了可以在[npmjs.org/doc/](https://npmjs.org/doc/)查看官方文档外，这里再介绍一些NPM常用命令。 

 NPM提供了很多命令，例如install和publish，使用npm help可查看所有命令。 

- NPM提供了很多命令，例如`install`和`publish`，使用`npm help`可查看所有命令。
- 使用`npm help <command>`可查看某条命令的详细帮助，例如`npm help install`。
- 在`package.json`所在目录下使用`npm install . -g`可先在本地安装当前命令行程序，可用于发布前的本地测试。
- 使用`npm update <package>`可以把当前目录下`node_modules`子目录里边的对应模块更新至最新版本。
- 使用`npm update <package> -g`可以把全局安装的对应命令行程序更新至最新版。
- 使用`npm cache clear`可以清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人。
- 使用`npm unpublish <package>@<version>`可以撤销发布自己发布过的某个版本代码。

##### 使用淘宝 NPM 镜像

大家都知道国内直接使用 npm 的官方镜像是非常慢的，这里推荐使用淘宝 NPM 镜像。

淘宝 NPM 镜像是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10分钟 一次以保证尽量与官方服务同步。

你可以使用淘宝定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm:

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

这样就可以使用 cnpm 命令来安装模块了：

```
$ cnpm install [name]
```

更多信息可以查阅：http://npm.taobao.org/。

如果你遇到了使用 npm 安 装node_modules 总是提示报错：报错: **npm resource busy or locked.....**。

可以先删除以前安装的 node_modules :

```
$ npm cache clean
$ npm install
```

#### REPL

REPL代表读取评估和演示打印循环，它就像 Window 下的控制台的计算机环境，或 Unix/Linux 系统的 Shell命令输入响应输出。 Node.js或Node 捆绑了一个REPL环境。可执行以下任务。

-  			**读取**- 读取用户的输入，解析输入的JavaScript数据结构并存储在内存
-  			**计算**- 采取并评估计算数据结构
-  			**打印**- 打印结果
-  			**循环** - 循环上面的命令，直到用户按Ctrl-C两次终止

 Node 的REPL 与 Node.js 的实验代码非常有用，用于调试JavaScript代码。

**特点**

REPL可以通过简单地在shell/控制台运行node不带任何参数来启动。

```
$ node
```

1. 简单的表达式
2. 使用变量
3. 多行表达
4. 下划线变量

**命令**

- ctrl + c - 终止当前命令

- ctrl + c twice - 终止 Node REPL

- ctrl + d - 终止 Node REPL

- Up/Down Keys - 查看命令历史记录和修改以前的命令

- tab Keys - 当前命令列表

- .help - 列出所有命令

- .break - 退出多行表达

- .clear - 从多行表达式退出

- .save - 当前 Node REPL会话保存到一个文件

- .load - 加载文件的内容到当前Node REPL会话

