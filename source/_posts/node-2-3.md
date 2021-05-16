---
title: Koa -- 基于Node.js平台的下一代Web开发框架
date: 2021-04-30 18:18:59
tags:
 - Node.JS
 - 后端
categories:
 - 后端
 - Node.JS
---

[Koa](https://koa.bootcss.com/) 是由 Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。 使用 koa 编写 web 应用，可以免除重复繁琐的回调函数嵌套， 并极大地提 升错误处理的效率。koa 不在内核方法中绑定任何中间件， 它仅仅提供了一个轻量优雅的 函数库，使得编写 Web 应用变得得心应手。开发思路和 express 差不多，最大的特点就是 可以避免异步嵌套。

<!--more-->

Koa.js 是一个极其精简的Web框架，只提供一下两种功能：

- HTTP服务
  - 处理HTTP请求request
  - 处理HTTP响应response
- 中间件容器
  - 中间件的加载
  - 中间件的执行

剩下的其他Web服务所需的能力，就根据开发者的需求去自定义开发，留下了很大的灵活空间，提高了Web服务的开发成本

### 入门

#### 知识储备

- Node.js 的基础知识
  - `http` 模块的使用
  - `fs` 模块使用
  - `path` 模块使用
  - `Buffer` 类型
- ES 的基础知识
  - `Promise`
  - `async/await`
- 其他知识
  - `HTTP` 协议
  - `Cookie` 原理

#### 环境准备

- 因为node.js v7.6.0开始完全支持async/await，不需要加flag，所以node.js环境都要7.6.0以上
- node.js环境 版本v7.6以上
  - 直接安装node.js 7.6：node.js官网地址[https://nodejs.org](https://nodejs.org/)
  - nvm管理多版本node.js：可以用nvm 进行node版本进行管理
    - Mac系统安装nvm https://github.com/creationix/nvm#manual-install
    - windows系统安装nvm https://github.com/coreybutler/nvm-windows
    - Ubuntu系统安装nvm https://github.com/creationix/nvm
- npm 版本3.x以上

#### 示例

1. 安装依赖

   ```bash
   npm init
   npm install koa@2.0.0
   ```

   之后修改package.json文件,增加start命令

   ```json
   {
     "name": "koa-server",
     "version": "1.0.0",
     "description": "",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "keywords": [],
     "author": "",
     "license": "ISC",
     "devDependencies": {
       "koa": "^2.13.1"
     }
   }
   ```

2. 创建app.js文件

   ```js
   const Koa = require('koa');
   const app = new Koa();
   
   // Express 写法
   // app.use(function (req, res) {
   //     res.send("hello word")
   // });
   app.use(async ctx => {
     ctx.body = 'Hello，Koa2!';
   });
   
   app.listen(3000);
   console.log('app started at port 3000...');
   ```

3. 运行程序

   ```bash
   npm run start
   ```

   然后浏览器请求<http://127.0.0.1:3000>

   ![](https://z3.ax1x.com/2021/05/09/gY93uV.png)

async/await：

- 可以让异步逻辑用同步写法实现
- 最底层的await返回需要是Promise对象
- 可以通过多层 async function 的同步写法代替传统的callback嵌套

#### 源码结构

```
├── lib
│   ├── application.js
│   ├── context.js
│   ├── request.js
│   └── response.js
└── package.json
```

这个就是 `GitHub` [https://github.com/koajs/koa](https://github.com/koajs/koa/)上开源的koa2源码的源文件结构，核心代码就是lib目录下的四个文件

- application.js 是整个koa2 的入口文件，封装了context，request，response，以及最核心的中间件处理流程。
- context.js 处理应用上下文，里面直接封装部分request.js和response.js的方法
- request.js 处理http请求
- response.js 处理http响应

#### koa2特性

- 只提供封装好http上下文、请求、响应，以及基于`async/await`的中间件容器。
- 利用ES7的`async/await`的来处理传统回调嵌套问题和代替koa@1的generator，但是需要在node.js 7.x的harmony模式下才能支持`async/await`。
- 中间件只支持 `async/await` 封装的，如果要使用koa@1基于generator中间件，需要通过中间件koa-convert封装一下才能使用。

#### 中间件

- koa v1和v2中使用到的中间件的开发和使用
- generator 中间件开发在koa v1和v2中使用
- async await 中间件开发和只能在koa v2中使用

**generator中间件开发**

1. generator中间件开发

   > generator中间件返回的应该是function * () 函数

   ```js
   /* ./middleware/logger-generator.js */
   function log( ctx ) {
       console.log( ctx.method, ctx.header.host + ctx.url )
   }
   
   module.exports = function () {
       return function * ( next ) {
   
           // 执行中间件的操作
           log( this )
   
           if ( next ) {
               yield next
           }
       }
   }
   ```

2. generator中间件在koa@1中的使用

   > generator 中间件在koa v1中可以直接use使用

   ```js
   const koa = require('koa')  // koa v1
   const loggerGenerator  = require('./middleware/logger-generator')
   const app = koa()
   
   app.use(loggerGenerator())
   
   app.use(function *( ) {
       ctx.body = 'hello,Koa2!'
   })
   
   app.listen(3000)
   console.log('the server is starting at port 3000')
   ```

3. generator中间件在koa@2中的使用

   > generator 中间件在koa v2中需要用koa-convert封装一下才能使用

   ```js
   const Koa = require('koa') // koa v2
   //https://github.com/koajs/convert
   const convert = require('koa-convert')
   const loggerGenerator  = require('./middleware/logger-generator')
   const app = new Koa()
   
   app.use(convert(loggerGenerator()))
   
   app.use(( ctx ) => {
       ctx.body = 'hello,Koa2!'
   })
   
   app.listen(3000)
   console.log('the server is starting at port 3000')
   ```

**async中间件开发**

1. async 中间件开发

   ```js
   /* ./middleware/logger-async.js */
   
   function log( ctx ) {
       console.log( ctx.method, ctx.header.host + ctx.url )
   }
   
   module.exports = function () {
     return async function ( ctx, next ) {
       log(ctx);
       await next()
     }
   }
   ```

2. async 中间件在koa@2中使用

   > async 中间件只能在 koa v2中使用

   ```js
   const Koa = require('koa') // koa v2
   const loggerAsync  = require('./middleware/logger-async')
   const app = new Koa()
   
   app.use(loggerAsync())
   
   app.use(( ctx ) => {
       ctx.body = 'hello,Koa2!'
   })
   
   app.listen(3000)
   console.log('the server is starting at port 3000')
   ```

#### middleware

middleware（ 中间件） 始终是Koa中的一个核心概念。在middleware中，中间件由
外而内的相互嵌套，执行完毕之后再由内到外的依次执行回调。

**工作原理**

1. 初始化koa实例后，我们会用`use`方法来加载中间件(middleware)，会有一个数组来存储中间件，use调用顺序会决定中间件的执行顺序。
2. 每个中间件都是一个函数(不是函数将报错)，接收两个参数，**第一个是ctx上下文对象，另一个是next函数**
3. 中间件函数中调用next方法就会去取下一个中间件函数继续执行。每个中间件函数执行完毕后都会返回一个promise对象。(ps:调用next方法并不是表示当前中间件函数执行完毕了，调用next之后仍可以继续执行其他代码)

他的执行顺序就像穿过下图的洋葱：

![](https://z3.ax1x.com/2021/05/09/gYiX2d.png)

洋葱模型可以看出，中间件的在 `await next()` 前后的操作，很像数据结构的一种场景——“栈”，先进后出。同时，又有统一上下文管理操作数据。综上所述，可以总结出一下特性。

- 有统一 `context`
- 操作先进后出
- 有控制先进后出的机制 `next`
- 有提前结束机制

**示例**：

```js
// 导入koa，和koa 1.x不同，在koa2中，我们导入的是一个class，因此用大写的Koa表示:
const Koa = require('koa');

// 创建一个Koa对象表示web app本身:
const app = new Koa();

// 对于任何请求，app将调用该异步函数处理请求：
//ctx是由koa传入的封装了request和response的变量
//next是koa传入的将要处理的下一个异步函数

//0.打印请求方法和路径
const middleware0 = async (ctx,next) =>{
    console.log(`${ctx.request.method} ${ctx.request.url}`); // 打印URL
    await next(); // 调用下一个middleware
    const rt = ctx.response.get('X-Response-Time');
    console.log(`${ctx.method} ${ctx.url} - ${rt}`);
}

//1.记录请求时间
const middleware1 = async (ctx,next)=>{
    const start = new Date();
    console.log('请求前',start.toISOString())
    await next();
    const ms = +new Date() - +start;
    console.log('请求后',new Date().toISOString())
    //设置响应时间
    ctx.set('X-Response-Time', `${ms}ms`);
};

//2.处理请求
const middleware2 = async (ctx,next)=>{
    await next();
    console.log('处理请求中')
    // 设置response的Content-Type:
    ctx.response.type = 'text/html';
    // 设置response的内容:
    ctx.response.body = '<h1>Hello, koa2!</h1>';
};

//注册中间件, 调用app.use()的顺序决定了middleware的顺序
app.use(middleware0)
app.use(middleware1)
app.use(middleware2)

// 在端口3000监听:
app.listen(3000);
console.log('app started at port 3000...');
```

运行后请求<http://127.0.0.1:3000>,命令行打印

```
GET /
请求前 2021-05-09T07:30:54.210Z
处理请求中
请求后 2021-05-09T07:30:54.218Z
GET / - 8ms
GET /favicon.ico
请求前 2021-05-09T07:30:54.384Z
处理请求中
请求后 2021-05-09T07:30:54.386Z
GET /favicon.ico - 2ms
```

### 路由

上下文的请求request对象中url之就是当前访问的路径名称，可以根据ctx.request.url 通过一定的判断或者正则匹配就可以定制出所需要的路由。

#### 自定义路由

目录结构:

```
.
├── app.js
├── package.json
└── view
    ├── 404.html
    ├── index.html
    └── todo.html
```

app.js

```js
const Koa = require('koa')
const fs = require('fs')
const app = new Koa()

/**
 * 用Promise封装异步读取文件方法
 * @param  {string} page html文件名称
 * @return {promise}      
 */
function render( page ) {
  return new Promise(( resolve, reject ) => {
    let viewUrl = `./view/${page}`
    fs.readFile(viewUrl, "binary", ( err, data ) => {
      if ( err ) {
        reject( err )
      } else {
        resolve( data )
      }
    })
  })
}

/**
 * 根据URL获取HTML内容
 * @param  {string} url koa2上下文的url，ctx.url
 * @return {string}     获取HTML文件内容
 */
async function route( url ) {
  let view = '404.html'
  switch ( url ) {
    case '/':
      view = 'index.html'
      break
    case '/index':
      view = 'index.html'
      break
    case '/todo':
      view = 'todo.html'
      break
    case '/404':
      view = '404.html'
      break
    default:
      break
  }
  let html = await render( view )
  return html
}

app.use( async ( ctx ) => {
  let url = ctx.request.url
  let html = await route( url )
  ctx.body = html
})

app.listen(3000)
console.log('[demo] route-simple is starting at port 3000')
```

html

```html
<!-- 404.html -->

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>404</title>
</head>
<body>
    <h1>koa2 demo 404 page</h1>
    <p>this is a 404 page</p>
</body>
</html>

<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
    <h1>koa2 demo index page</h1>
    <p>this is a index page</p>
    <ul>
      <li><a href="/">/</a></li>
      <li><a href="/index">/index</a></li>
      <li><a href="/todo">/todo</a></li>
      <li><a href="/404">/404</a></li>
      <li><a href="/nofund">/nofund</a></li>
    </ul>
</body>
</html>
<!-- todo.html -->

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>todo</title>
</head>
<body>
    <h1>koa2 demo todo page</h1>
    <p>this is a todo page</p>
</body>
</html>
```

访问http://localhost:3000/index ![route-result-01](https://z3.ax1x.com/2021/05/09/gYuEz8.png)

#### [koa-router](http://github.com/koajs/router)中间件

如果依靠`ctx.request.url`去手动处理路由，将会写很多处理代码，这时候就需要对应的路由的中间件对路由进行控制，这里介绍一个比较好用的路由中间件koa-router

1. 安装koa-router中间件

   ```bas
   npm install --save koa-router@7
   ```

2. 使用

   ```js
   const Koa = require('koa')
   const fs = require('fs')
   const app = new Koa()
   
   const Router = require('koa-router')
   
   let home = new Router()
   
   // 子路由1
   home.get('/', async ( ctx )=>{
     let html = `
       <ul>
         <li><a href="/page/helloworld">/page/helloworld</a></li>
         <li><a href="/page/404">/page/404</a></li>
       </ul>
     `
     ctx.body = html
   })
   
   // 子路由2
   let page = new Router()
   page.get('/404', async ( ctx )=>{
     ctx.body = '404 page!'
   }).get('/helloworld', async ( ctx )=>{
     ctx.body = 'helloworld page!'
   })
   
   // 装载所有子路由
   let router = new Router()
   router.use('/', home.routes(), home.allowedMethods())
   router.use('/page', page.routes(), page.allowedMethods())
   
   // 加载路由中间件
   app.use(router.routes()).use(router.allowedMethods())
   
   app.listen(3000, () => {
     console.log('[demo] route-use-middleware is starting at port 3000')
   })
   ```

### 请求数据获取

#### GET请求

在koa中，获取GET请求数据源头是koa中request对象中的query方法或querystring方法，query返回是格式化好的参数对象，querystring返回的是请求字符串，由于ctx对request的API有直接引用的方式，所以获取GET请求数据有两个途径。

- 1.是从上下文中直接获取
  - 请求对象ctx.query，返回如 { a:1, b:2 }
  - 请求字符串 ctx.querystring，返回如 a=1&b=2
- 2.是从上下文的request对象中获取
  - 请求对象ctx.request.query，返回如 { a:1, b:2 }
  - 请求字符串 ctx.request.querystring，返回如 a=1&b=2

示例

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {
  let url = ctx.url
  // 从上下文的request对象中获取
  let request = ctx.request
  let req_query = request.query
  let req_querystring = request.querystring

  // 从上下文中直接获取
  let ctx_query = ctx.query
  let ctx_querystring = ctx.querystring

  ctx.body = {
    url,
    req_query,
    req_querystring,
    ctx_query,
    ctx_querystring
  }
})

app.listen(3000, () => {
  console.log('[demo] request get is starting at port 3000')
})
```

执行后程序后，用chrome访问 http://localhost:3000/page/user?a=1&b=2 会出现以下情况

```json
{"url":"/page/user?a=1&b=2","req_query":{"a":"1","b":"2"},"req_querystring":"a=1&b=2","ctx_query":{"a":"1","b":"2"},"ctx_querystring":"a=1&b=2"}
```

**POST请求**

对于POST请求的处理，koa2没有封装获取参数的方法，需要通过解析上下文context中的原生node.js请求对象req，将POST表单数据解析成query string（例如：`a=1&b=2&c=3`），再将query string 解析成JSON格式（例如：`{"a":"1", "b":"2", "c":"3"}`）

> 注意：ctx.request是context经过封装的请求对象，ctx.req是context提供的node.js原生HTTP请求对象，同理ctx.response是context经过封装的响应对象，ctx.res是context提供的node.js原生HTTP请求对象。
>
> 具体koa2 API文档可见 https://github.com/koajs/koa/blob/master/docs/api/context.md#ctxreq

示例

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {

  if ( ctx.url === '/' && ctx.method === 'GET' ) {
    // 当GET请求时候返回表单页面
    let html = `
      <h1>koa2 request post demo</h1>
      <form method="POST" action="/">
        <p>userName</p>
        <input name="userName" /><br/>
        <p>nickName</p>
        <input name="nickName" /><br/>
        <p>email</p>
        <input name="email" /><br/>
        <button type="submit">submit</button>
      </form>
    `
    ctx.body = html
  } else if ( ctx.url === '/' && ctx.method === 'POST' ) {
    // 当POST请求的时候，解析POST表单里的数据，并显示出来
    let postData = await parsePostData( ctx )
    ctx.body = postData
  } else {
    // 其他请求显示404
    ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
  }
})

// 解析上下文里node原生请求的POST参数
function parsePostData( ctx ) {
  return new Promise((resolve, reject) => {
    try {
      let postdata = "";
      ctx.req.addListener('data', (data) => {
        postdata += data
      })
      ctx.req.addListener("end",function(){
        let parseData = parseQueryStr( postdata )
        resolve( parseData )
      })
    } catch ( err ) {
      reject(err)
    }
  })
}

// 将POST请求参数字符串解析成JSON
function parseQueryStr( queryStr ) {
  let queryData = {}
  let queryStrList = queryStr.split('&')
  console.log( queryStrList )
  for (  let [ index, queryStr ] of queryStrList.entries()  ) {
    let itemList = queryStr.split('=')
    queryData[ itemList[0] ] = decodeURIComponent(itemList[1])
  }
  return queryData
}

app.listen(3000, () => {
  console.log('[demo] request post is starting at port 3000')
})
```

#### [koa-bodyparser](https://github.com/koajs/bodyparser)中间件

对于POST请求的处理，koa-bodyparser中间件可以把koa2上下文的formData数据解析到ctx.request.body中

1. 安装

   ```bash
   npm install --save koa-bodyparser@3
   ```

2. 使用

   ```js
   const Koa = require('koa')
   const app = new Koa()
   const bodyParser = require('koa-bodyparser')
   
   // 使用ctx.body解析中间件
   app.use(bodyParser())
   
   app.use( async ( ctx ) => {
   
     if ( ctx.url === '/' && ctx.method === 'GET' ) {
       // 当GET请求时候返回表单页面
       let html = `
         <h1>koa2 request post demo</h1>
         <form method="POST" action="/">
           <p>userName</p>
           <input name="userName" /><br/>
           <p>nickName</p>
           <input name="nickName" /><br/>
           <p>email</p>
           <input name="email" /><br/>
           <button type="submit">submit</button>
         </form>
       `
       ctx.body = html
     } else if ( ctx.url === '/' && ctx.method === 'POST' ) {
       // 当POST请求的时候，中间件koa-bodyparser解析POST表单里的数据，并显示出来
       let postData = ctx.request.body
       ctx.body = postData
     } else {
       // 其他请求显示404
       ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
     }
   })
   
   app.listen(3000, () => {
     console.log('[demo] request post is starting at port 3000')
   })
   ```

### 静态资源服务器

一个http请求访问web服务静态资源，一般响应结果有三种情况

- 访问文本，例如js，css，png，jpg，gif
- 访问静态目录
- 找不到资源，抛出404错误

#### 原生koa2

1. 目录

   ```
   ├── static # 静态资源目录
   │   ├── css/style.css
   │   ├── image/nodejs.jpg
   │   ├── js/index.js
   │   └── index.html
   ├── util # 工具代码
   │   ├── content.js # 读取请求内容
   │   ├── dir.js # 读取目录内容
   │   ├── file.js # 读取文件内容
   │   ├── mimes.js # 文件类型列表
   │   └── walk.js # 遍历目录内容
   └── index.js # 启动入口文件
   ```

2. `index.js`

   ```js
   const Koa = require('koa')
   const path = require('path')
   const content = require('./util/content')
   const mimes = require('./util/mimes')
   
   const app = new Koa()
   
   // 静态资源目录对于相对入口文件index.js的路径
   const staticPath = './static'
   
   // 解析资源类型
   function parseMime( url ) {
     let extName = path.extname( url )
     extName = extName ?  extName.slice(1) : 'unknown'
     return  mimes[ extName ]
   }
   
   app.use( async ( ctx ) => {
     // 静态资源目录在本地的绝对路径
     let fullStaticPath = path.join(__dirname, staticPath)
   
     // 获取静态资源内容，有可能是文件内容，目录，或404
     let _content = await content( ctx, fullStaticPath )
   
     // 解析请求内容的类型
     let _mime = parseMime( ctx.url )
   
     // 如果有对应的文件类型，就配置上下文的类型
     if ( _mime ) {
       ctx.type = _mime
     }
   
     // 输出静态资源内容
     if ( _mime && _mime.indexOf('image/') >= 0 ) {
       // 如果是图片，则用node原生res，输出二进制数据
       ctx.res.writeHead(200)
       ctx.res.write(_content, 'binary')
       ctx.res.end()
     } else {
       // 其他则输出文本
       ctx.body = _content
     }
   })
   
   app.listen(3000)
   console.log('[demo] static-server is starting at port 3000')
   ```

3. util工具

   ```js
   //util/content.js
   const path = require('path')
   const fs = require('fs')
   
   // 封装读取目录内容方法
   const dir = require('./dir')
   
   // 封装读取文件内容方法
   const file = require('./file')
   
   
   /**
    * 获取静态资源内容
    * @param  {object} ctx koa上下文
    * @param  {string} 静态资源目录在本地的绝对路径
    * @return  {string} 请求获取到的本地内容
    */
   async function content( ctx, fullStaticPath ) {
   
     // 封装请求资源的完绝对径
     let reqPath = path.join(fullStaticPath, ctx.url)
   
     // 判断请求路径是否为存在目录或者文件
     let exist = fs.existsSync( reqPath )
   
     // 返回请求内容， 默认为空
     let content = ''
   
     if( !exist ) {
       //如果请求路径不存在，返回404
       content = '404 Not Found! o(╯□╰)o！'
     } else {
       //判断访问地址是文件夹还是文件
       let stat = fs.statSync( reqPath )
   
       if( stat.isDirectory() ) {
         //如果为目录，则渲读取目录内容
         content = dir( ctx.url, reqPath )
   
       } else {
         // 如果请求为文件，则读取文件内容
         content = await file( reqPath )
       }
     }
   
     return content
   }
   
   module.exports = content
   
   //util/dir.js
   const url = require('url')
   const fs = require('fs')
   const path = require('path')
   
   // 遍历读取目录内容方法
   const walk = require('./walk')
   
   /**
    * 封装目录内容
    * @param  {string} url 当前请求的上下文中的url，即ctx.url
    * @param  {string} reqPath 请求静态资源的完整本地路径
    * @return {string} 返回目录内容，封装成HTML
    */
   function dir ( url, reqPath ) {
   
     // 遍历读取当前目录下的文件、子目录
     let contentList = walk( reqPath )
   
     let html = `<ul>`
     for ( let [ index, item ] of contentList.entries() ) {
       html = `${html}<li><a href="${url === '/' ? '' : url}/${item}">${item}</a>` 
     }
     html = `${html}</ul>`
   
     return html
   }
   
   module.exports = dir
   
   //util/file.js
   const fs = require('fs')
   
   /**
    * 读取文件方法
    * @param  {string} 文件本地的绝对路径
    * @return {string|binary} 
    */
   function file ( filePath ) {
   
    let content = fs.readFileSync(filePath, 'binary' )
    return content
   }
   
   module.exports = file
   
   //util/walk.js
   
   const fs = require('fs')
   const mimes = require('./mimes')
   
   /**
    * 遍历读取目录内容（子目录，文件名）
    * @param  {string} reqPath 请求资源的绝对路径
    * @return {array} 目录内容列表
    */
   function walk( reqPath ){
   
     let files = fs.readdirSync( reqPath );
   
     let dirList = [], fileList = [];
     for( let i=0, len=files.length; i<len; i++ ) {
       let item = files[i];
       let itemArr = item.split("\.");
       let itemMime = ( itemArr.length > 1 ) ? itemArr[ itemArr.length - 1 ] : "undefined";
   
       if( typeof mimes[ itemMime ] === "undefined" ) {
         dirList.push( files[i] );
       } else {
         fileList.push( files[i] );
       }
     }
   
   
     let result = dirList.concat( fileList );
   
     return result;
   };
   
   module.exports = walk;
   
   //util/mmimes.js
   
   let mimes = {
     'css': 'text/css',
     'less': 'text/css',
     'gif': 'image/gif',
     'html': 'text/html',
     'ico': 'image/x-icon',
     'jpeg': 'image/jpeg',
     'jpg': 'image/jpeg',
     'js': 'text/javascript',
     'json': 'application/json',
     'pdf': 'application/pdf',
     'png': 'image/png',
     'svg': 'image/svg+xml',
     'swf': 'application/x-shockwave-flash',
     'tiff': 'image/tiff',
     'txt': 'text/plain',
     'wav': 'audio/x-wav',
     'wma': 'audio/x-ms-wma',
     'wmv': 'video/x-ms-wmv',
     'xml': 'text/xml'
   }
   
   module.exports = mimes
   ```

4. 静态资源

   ```html
   // static/css/style.css
   .h1 {
     padding: 20px;
     color: #222;
     background: #f0f0f0;
   }
   
   // static/js/index.js
   (function( ){
     alert('hello koa2 static server')
     console.log('hello koa2 static server')
   })()
   
   // static/index.html
   
   <!DOCTYPE html>
   <html>
   <head>
       <meta charset="UTF-8">
       <title>index</title>
       <link rel="stylesheet" href="/css/style.css" />
   </head>
   <body>
       <h1 class="h1">koa2 simple static server</h1>
       <img src="/image/nodejs.jpg" />
       <script src="/js/index.js"></script>
   </body>
   </html>
   ```

#### [koa-static](https://github.com/koajs/static)中间件

```js
const Koa = require('koa')
const path = require('path')
const static = require('koa-static')

const app = new Koa()

// 静态资源目录对于相对入口文件index.js的路径
const staticPath = './static'

app.use(static(
  path.join( __dirname,  staticPath)
))


app.use( async ( ctx ) => {
  ctx.body = 'hello world'
})

app.listen(3000, () => {
  console.log('[demo] static-use-middleware is starting at port 3000')
})
```

### 使用cookie

koa提供了从上下文直接读取、写入cookie的方法

- ctx.cookies.get(name, [options]) 读取上下文请求中的cookie
- ctx.cookies.set(name, value, [options]) 在上下文中写入cookie

koa2 中操作的cookies是使用了npm的cookies模块，源码在https://github.com/pillarjs/cookies，所以在读写cookie的使用参数与该模块的使用一致。

**示例：**

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {

  if ( ctx.url === '/index' ) {
    ctx.cookies.set(
      'cid', 
      'hello world',
      {
        domain: 'localhost',  // 写cookie所在的域名
        path: '/index',       // 写cookie所在的路径
        maxAge: 10 * 60 * 1000, // cookie有效时长
        expires: new Date('2021-09-15'),  // cookie失效时间
        httpOnly: false,  // 是否只用于http请求中获取
        overwrite: false  // 是否允许重写
      }
    )
    ctx.body = 'cookie is ok'
  } else {
    ctx.body = 'hello world' 
  }

})

app.listen(3000, () => {
  console.log('[demo] cookie is starting at port 3000')
})
```

访问http://localhost:3000/index

- 可以在控制台的cookie列表中中看到写在页面上的cookie
- 在控制台的console中使用document.cookie可以打印出在页面的所有cookie（需要是httpOnly设置false才能显示）

### 实现session

koa2原生功能只提供了cookie的操作，但是没有提供session操作。session就只用自己实现或者通过第三方中间件实现。在koa2中实现session的方案有一下几种

- 如果session数据量很小，可以直接存在内存中
- 如果session数据量很大，则需要存储介质存放session数据

#### [koa-session](https://github.com/koajs/session)

用于Koa的简单会话中间件。 默认为基于cookie的会话，并支持外部存储。

当给session赋值时，koa-session会将session值加密并设置成cookie,所以不使用额外存储的话，其session大小会有限制

**安装**

```
$ npm install koa-session
```

**示例**

```js
const session = require('koa-session');
const Koa = require('koa');
const app = new Koa();

app.keys = ['some secret hurr'];

const CONFIG = {
  key: 'koa.sess', /** (string) cookie key (default is koa.sess) */
  /** (number || 'session') maxAge in ms (default is 1 days) */
  /** 'session' will result in a cookie that expires when session/browser is closed */
  /** Warning: If a session cookie is stolen, this cookie will never expire */
  maxAge: 86400000,
  autoCommit: true, /** (boolean) automatically commit headers (default true) */
  overwrite: true, /** (boolean) can overwrite or not (default true) */
  httpOnly: true, /** (boolean) httpOnly or not (default true) */
  signed: true, /** (boolean) signed or not (default true) */
  rolling: false, /** (boolean) Force a session identifier cookie to be set on every response. The expiration is reset to the original maxAge, resetting the expiration countdown. (default is false) */
  renew: false, /** (boolean) renew session when session is nearly expired, so we can always keep user logged in. (default is false)*/
  secure: false, /** (boolean) secure cookie*/
  sameSite: null, /** (string) session cookie sameSite options (default null, don't set it) */
};

app.use(session(CONFIG, app));
// or if you prefer all default config, just use => app.use(session(app));

app.use(ctx => {
  // ignore favicon
  if (ctx.path === '/favicon.ico') return;

  let n = ctx.session.views || 0;
  ctx.session.views = ++n;
  ctx.body = n + ' views';
});

app.listen(3000);
console.log('listening on port 3000');
```

#### [koa-generic-session](https://github.com/koajs/generic-session)

用于koa的通用会话中间件，易于与[redis](https://github.com/koajs/koa-redis)或[mongo](https://github.com/freakycue/koa-generic-session-mongo)等自定义存储一起使用，支持defer session getter。

仅在手动设置会话时，此中间件才会设置cookie。每次修改会话时（且仅在修改会话时），它将重置cookie和会话。

您可以使用滚动会话来为每个触摸会话的请求重置cookie和会话。每个请求都可以覆盖保存行为。

> 这里使用redis

安装：

```bash
npm i koa-generic-session koa-redis
```

示例：

```js
const session = require('koa-generic-session');
const redisStore = require('koa-redis');
const koa = require('koa');

const app = new koa();
app.keys = ['keys', 'keykeys'];
app.use(session({
  store: redisStore()
}));

app.use(async (ctx,next) => {
    if(ctx.session.viewCount ==null) {
        ctx.session.viewCount = 0
    }
    ctx.session.viewCount++
    ctx.body = {
        errno:0,
        viewCount: ctx.session.viewCount
    }
})

app.listen(3000);
```

### [Koa-views](https://github.com/queckezz/koa-views)加载模板引擎

[ejs](https://github.com/mde/ejs)

**安装**

```bash
# 安装koa模板使用中间件
npm install --save koa-views

# 安装ejs模板引擎
npm install --save ejs
```

**目录**

```
├── package.json
├── index.js
└── view
    └── index.ejs
```

**./index.js文件**

```js
const Koa = require('koa')
const views = require('koa-views')
const path = require('path')
const app = new Koa()

// 加载模板引擎
app.use(views(path.join(__dirname, './view'), {
  extension: 'ejs'
}))

app.use( async ( ctx ) => {
  let title = 'hello koa2'
  await ctx.render('index', {
    title,
  })
})

app.listen(3000)
```

**./view/index.ejs 模板**

```ejs
<!DOCTYPE html>
<html>
<head>
    <title><%= title %></title>
</head>
<body>
    <h1><%= title %></h1>
    <p>EJS Welcome to <%= title %></p>
</body>
</html>
```

### 文件上传

#### busboy模块

[busboy](https://github.com/mscdex/busboy) 模块是用来解析POST请求，node原生req中的文件流。

1. 安装

   ```bash
   npm install --save busboy
   ```

2. 示例

   ```js
   const inspect = require('util').inspect 
   const path = require('path')
   const fs = require('fs')
   const Busboy = require('busboy')
   
   // req 为node原生请求
   const busboy = new Busboy({ headers: req.headers })
   
   // ...
   
   // 监听文件解析事件
   busboy.on('file', function(fieldname, file, filename, encoding, mimetype) {
     console.log(`File [${fieldname}]: filename: ${filename}`)
   
   
     // 文件保存到特定路径
     file.pipe(fs.createWriteStream('./upload'))
   
     // 开始解析文件流
     file.on('data', function(data) {
       console.log(`File [${fieldname}] got ${data.length} bytes`)
     })
   
     // 解析文件结束
     file.on('end', function() {
       console.log(`File [${fieldname}] Finished`)
     })
   })
   
   // 监听请求中的字段
   busboy.on('field', function(fieldname, val, fieldnameTruncated, valTruncated) {
     console.log(`Field [${fieldname}]: value: ${inspect(val)}`)
   })
   
   // 监听结束事件
   busboy.on('finish', function() {
     console.log('Done parsing form!')
     res.writeHead(303, { Connection: 'close', Location: '/' })
     res.end()
   })
   req.pipe(busboy)
   ```

#### 上传文件

1. 帮助文件

   ```js
   const inspect = require('util').inspect
   const path = require('path')
   const os = require('os')
   const fs = require('fs')
   const Busboy = require('busboy')
   
   /**
    * 同步创建文件目录
    * @param  {string} dirname 目录绝对地址
    * @return {boolean}        创建目录结果
    */
   function mkdirsSync( dirname ) {
     if (fs.existsSync( dirname )) {
       return true
     } else {
       if (mkdirsSync( path.dirname(dirname)) ) {
         fs.mkdirSync( dirname )
         return true
       }
     }
   }
   
   /**
    * 获取上传文件的后缀名
    * @param  {string} fileName 获取上传文件的后缀名
    * @return {string}          文件后缀名
    */
   function getSuffixName( fileName ) {
     let nameList = fileName.split('.')
     return nameList[nameList.length - 1]
   }
   
   /**
    * 上传文件
    * @param  {object} ctx     koa上下文
    * @param  {object} options 文件上传参数 fileType文件类型， path文件存放路径
    * @return {promise}         
    */
   function uploadFile( ctx, options) {
     let req = ctx.req
     let res = ctx.res
     let busboy = new Busboy({headers: req.headers})
   
     // 获取类型
     let fileType = options.fileType || 'common'
     let filePath = path.join( options.path,  fileType)
     let mkdirResult = mkdirsSync( filePath )
   
     return new Promise((resolve, reject) => {
       console.log('文件上传中...')
       let result = { 
         success: false,
         formData: {},
       }
   
       // 解析请求文件事件
       busboy.on('file', function(fieldname, file, filename, encoding, mimetype) {
         let fileName = Math.random().toString(16).substr(2) + '.' + getSuffixName(filename)
         let _uploadFilePath = path.join( filePath, fileName )
         let saveTo = path.join(_uploadFilePath)
   
         // 文件保存到制定路径
         file.pipe(fs.createWriteStream(saveTo))
   
         // 文件写入事件结束
         file.on('end', function() {
           result.success = true
           result.message = '文件上传成功'
   
           console.log('文件上传成功！')
           resolve(result)
         })
       })
   
       // 解析表单中其他字段信息
       busboy.on('field', function(fieldname, val, fieldnameTruncated, valTruncated, encoding, mimetype) {
         console.log('表单字段数据 [' + fieldname + ']: value: ' + inspect(val));
         result.formData[fieldname] = inspect(val);
       });
   
       // 解析结束事件
       busboy.on('finish', function( ) {
         console.log('文件上结束')
         resolve(result)
       })
   
       // 解析错误事件
       busboy.on('error', function(err) {
         console.log('文件上出错')
         reject(result)
       })
   
       req.pipe(busboy)
     })
   
   } 
   
   
   module.exports =  {
     uploadFile
   }
   ```

2. 入口文件

   ```js
   const Koa = require('koa')
   const path = require('path')
   const app = new Koa()
   // const bodyParser = require('koa-bodyparser')
   
   const { uploadFile } = require('./util/upload')
   
   // app.use(bodyParser())
   
   app.use( async ( ctx ) => {
   
     if ( ctx.url === '/' && ctx.method === 'GET' ) {
       // 当GET请求时候返回表单页面
       let html = `
         <h1>koa2 upload demo</h1>
         <form method="POST" action="/upload.json" enctype="multipart/form-data">
           <p>file upload</p>
           <span>picName:</span><input name="picName" type="text" /><br/>
           <input name="file" type="file" /><br/><br/>
           <button type="submit">submit</button>
         </form>
       `
       ctx.body = html
   
     } else if ( ctx.url === '/upload.json' && ctx.method === 'POST' ) {
       // 上传文件请求处理
       let result = { success: false }
       let serverFilePath = path.join( __dirname, 'upload-files' )
   
       // 上传文件事件
       result = await uploadFile( ctx, {
         fileType: 'album', // common or album
         path: serverFilePath
       })
   
       ctx.body = result
     } else {
       // 其他请求显示404
       ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
     }
   })
   
   app.listen(3000, () => {
     console.log('[demo] upload-simple is starting at port 3000')
   })
   ```

3. 运行结果

   ![](https://z3.ax1x.com/2021/05/09/gYIYcj.png)

   ![](https://z3.ax1x.com/2021/05/09/gYIaBq.png)

   ![](https://z3.ax1x.com/2021/05/09/gYI0EV.png)

   ![](https://z3.ax1x.com/2021/05/09/gYIsCF.png)

### mysql模块

安装mysql：https://www.mysql.com/downloads/

安装mysql模块： `npm install --save mysql`

mysql模块是node操作MySQL的引擎，可以在node.js环境下对MySQL数据库进行建表，增、删、改、查等操作。

#### Promise封装mysql模块

1. async-db.js

   ```js
   const mysql = require('mysql')
   
   const pool = mysql.createPool({
     host     :  '127.0.0.1',
     user     :  'root',
     password :  '123456',
     database :  'koa_demo'
   })
   
   let query = function( sql, values ) {
   
     return new Promise(( resolve, reject ) => {
       pool.getConnection(function(err, connection) {
         if (err) {
           reject( err )
         } else {
           connection.query(sql, values, ( err, rows) => {
   
             if ( err ) {
               reject( err )
             } else {
               resolve( rows )
             }
             connection.release()
           })
         }
       })
     })
   
   }
   
   
   module.exports = {
     query
   }
   ```

2. 使用

   ```js
   const { query } = require('./async-db')
   async function selectAllData( ) {
     let sql = 'SELECT * FROM my_table'
     let dataList = await query( sql )
     return dataList
   }
   
   async function getData() {
     let dataList = await selectAllData()
     console.log( dataList )
   }
   
   getData()
   ```

#### [Sequelize](https://www.sequelize.com.cn/)

Sequelize 是一个基于 promise 的 Node.js [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping), 目前支持 [Postgres](https://en.wikipedia.org/wiki/PostgreSQL), [MySQL](https://en.wikipedia.org/wiki/MySQL), [MariaDB](https://en.wikipedia.org/wiki/MariaDB), [SQLite](https://en.wikipedia.org/wiki/SQLite) 以及 [Microsoft SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server). 它具有强大的事务支持, 关联关系, 预读和延迟加载,读取复制等功能。

Sequelize 遵从 [语义版本控制](http://semver.org/)。 支持 Node v10 及更高版本以便使用 ES6 功能。

1. 安装依赖

   ```bash
   npm i sequelize mysql2
   ```

2. 示例

   ```js
   const Sequelize = require('sequelize');
   const config = {
       database: 'test', // 使用哪个数据库
       username: 'root', // 用户名
       password: '123456', // 口令
       host: 'localhost', // 主机名
       port: 3306 // 端口号，MySQL默认3306
   };
   
   
   //创建一个sequelize对象实例
   const sequelize = new Sequelize(config.database, config.username, config.password, {
       host: config.host,
       dialect: 'mysql',
       pool: {
           max: 5,
           min: 0,
           idle: 30000
       },
   });
   
   sequelize.authenticate().then(res => {
       console.log('Connection has been established successfully.');
       //定义模型Pet，告诉Sequelize如何映射数据库表
       const Pet = sequelize.define('pet', {
           id: {
               type: Sequelize.STRING(50),
               primaryKey: true
           },
           name: Sequelize.STRING(100),
           gender: Sequelize.BOOLEAN,
           birth: Sequelize.STRING(10),
           createdAt: Sequelize.BIGINT,
           updatedAt: Sequelize.BIGINT,
           version: Sequelize.BIGINT
       }, {
           timestamps: false
       });
   
       console.log(Pet)
   
       sequelize.close().then((res) => {
           console.log('closed')
       })
   }).catch(error => {
       console.error('Unable to connect to the database:', error);
   })
   ```

   用`sequelize.define()`定义Model时，传入名称`pet`，默认的表名就是`pets`。第二个参数指定列名和数据类型，如果是主键，需要更详细地指定。第三个参数是额外的配置，我们传入`{ timestamps: false }`是为了关闭Sequelize的自动添加timestamp的功能。

   我们把通过`sequelize.define()`返回的`Pet`称为Model，它表示一个数据模型。

   我们把通过`Pet.findAll()`返回的一个或一组对象称为Model实例，每个实例都可以直接通过`JSON.stringify`序列化为JSON字符串。但是它们和普通JSON对象相比，多了一些由Sequelize添加的方法，比如`save()`和`destroy()`。调用这些方法我们可以执行更新或者删除操作。

   所以，使用Sequelize操作数据库的一般步骤就是：

   - 首先，通过某个Model对象的`findAll()`方法获取实例；

   - 如果要更新实例，先对实例属性赋新值，再调用`save()`方法；

   - 如果要删除实例，直接调用`destroy()`方法。

   注意`findAll()`方法可以接收`where`、`order`这些参数，这和将要生成的SQL语句是对应的。

### 跨域

#### jsonp支持

在项目复杂的业务场景，有时候需要在前端跨域获取数据，这时候提供数据的服务就需要提供跨域请求的接口，通常是使用JSONP的方式提供跨域接口。

- JSONP跨域输出的数据是可执行的JavaScript代码
  - ctx输出的类型应该是'text/javascript'
  - ctx输出的内容为可执行的返回数据JavaScript代码字符串
- 需要有回调函数名callbackName，前端获取后会通过动态执行JavaScript代码字符，获取里面的数据

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {


  // 如果jsonp 的请求为GET
  if ( ctx.method === 'GET' && ctx.url.split('?')[0] === '/getData.jsonp') {

    // 获取jsonp的callback
    let callbackName = ctx.query.callback || 'callback'
    let returnData = {
      success: true,
      data: {
        text: 'this is a jsonp api',
        time: new Date().getTime(),
      }
    }

    // jsonp的script字符串
    let jsonpStr = `;${callbackName}(${JSON.stringify(returnData)})`

    // 用text/javascript，让请求支持跨域获取
    ctx.type = 'text/javascript'

    // 输出jsonp字符串
    ctx.body = jsonpStr

  } else {

    ctx.body = 'hello jsonp'

  }
})

app.listen(3000, () => {
  console.log('[demo] jsonp is starting at port 3000')
})
```

#### [koa-safe-jsonp](https://github.com/koajs/koa-safe-jsonp)中间件

安全的jsonp插件

```
$ npm i koa-safe-jsonp
```

示例

```js
const jsonp = require('koa-safe-jsonp');
const Koa = require('Koa');

const app = new Koa();
jsonp(app, {
  callback: '_callback', // default is 'callback'
  limit: 50, // max callback name string length, default is 512
});

app.use(function (ctx) {
  ctx.jsonp = {foo: "bar"};
});

app.listen(3000, () => {
  console.log('[demo] jsonp is starting at port 3000')
})
```

curl test it:

```bash
$ curl 'http://127.0.0.1:3000/foo.json?_callback=fn' -v
详细信息: GET http://127.0.0.1:3000/foo.json?_callback=fn with 0-byte payload
详细信息: received 51-byte response of content type application/javascript; charset=utf-8


StatusCode        : 200
StatusDescription : OK
Content           : /**/ typeof fn === 'function' && fn({"foo":"bar"});
RawContent        : HTTP/1.1 200 OK
                    X-Content-Type-Options: nosniff
                    Connection: keep-alive
                    Keep-Alive: timeout=5
                    Content-Length: 51
                    Content-Type: application/javascript; charset=utf-8
                    Date: Sun, 09 May 2021 15:24:4...
Forms             : {}
Headers           : {[X-Content-Type-Options, nosniff], [Connection, keep-alive], [Keep-Alive, timeout=5], [Content-Len
                    gth, 51]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 51
```

#### CORS跨域

[koa2-cors](https://github.com/zadzbw/koa2-cors)

1. 安装

   ```bash
   npm install koa2-cors
   ```

2. 示例

   ```js
   var koa = require('koa');
   var route = require('koa-route');
   var cors = require('koa2-cors');
   var app = koa();
   
   /**
    * 关键点：
    * 1、如果需要支持 cookies,
    *    Access-Control-Allow-Origin 不能设置为 *,
    *    并且 Access-Control-Allow-Credentials 需要设置为 true
    *    (注意前端请求需要设置 withCredentials = true)
    * 2、当 method = OPTIONS 时, 属于预检(复杂请求), 当为预检时, 可以直接返回空响应体, 对应的 http 状态码为 204
    * 3、通过 Access-Control-Max-Age 可以设置预检结果的缓存, 单位(秒)
    * 4、通过 Access-Control-Allow-Headers 设置需要支持的跨域请求头
    * 5、通过 Access-Control-Allow-Methods 设置需要支持的跨域请求方法
    */
   app.use(cors({
     origin: '*',
     maxAge: 60*60*24,
     allowMethods: ['POST', 'GET', 'OPTIONS', 'DELETE', 'PUT'],
     allowHeaders: ['X-Requested-With', 'User-Agent', 'Referer', 'Content-Type', 'Cache-Control','accesstoken']
   }));
   
   app.use(route.get('/', function() {
     this.body = { msg: 'Hello World!' };
   }));
   
   app.listen(3000);
   ```

### 单元测试

测试是一个项目周期里必不可少的环节，开发者在开发过程中也是无时无刻进行“人工测试”，如果每次修改一点代码，都要牵一发动全身都要手动测试关联接口，这样子是禁锢了生产力。为了解放大部分测试生产力，相关的测试框架应运而生，比较出名的有mocha，karma，jasmine等。虽然框架繁多，但是使用起来都是大同小异。

1. 安装

   ```bash
   npm install --save-dev mocha chai supertest
   ```

   - mocha 模块是测试框架
   - chai 模块是用来进行测试结果断言库，比如一个判断 1 + 1 是否等于 2
   - supertest 模块是http请求测试库，用来请求API接口

2. 目录结构

   ```
   .
   ├── index.js # api文件
   ├── package.json
   └── test # 测试目录
       └── index.test.js # 测试用例
   ```

3. api文件

   ```js
   const Koa = require('koa')
   const app = new Koa()
   
   const server = async ( ctx, next ) => {
     let result = {
       success: true,
       data: null
     }
   
     if ( ctx.method === 'GET' ) { 
       if ( ctx.url === '/getString.json' ) {
         result.data = 'this is string data'
       } else if ( ctx.url === '/getNumber.json' ) {
         result.data = 123456
       } else {
         result.success = false
       }
       ctx.body = result
       next && next()
     } else if ( ctx.method === 'POST' ) {
       if ( ctx.url === '/postData.json' ) {
         result.data = 'ok'
       } else {
         result.success = false
       }
       ctx.body = result
       next && next()
     } else {
       ctx.body = 'hello world'
       next && next()
     }
   }
   
   app.use(server)
   
   module.exports = app
   
   app.listen(3000, () => {
     console.log('[demo] test-unit is starting at port 3000')
   })
   ```

   启动服务后访问接口会看到以下数据

   http://localhost:3000/getString.json

   ```
   {"success":true,"data":"this is string data"}
   ```

4. 测试用例

   ```js
   const supertest = require('supertest')
   const chai = require('chai')
   const app = require('./../index')
   
   const expect = chai.expect
   const request = supertest( app.listen() )
   
   describe( '开始测试demo的GET请求', ( ) => {
     
     it('测试/getString.json请求', ( done ) => {
         request
           .get('/getString.json')
           .expect(200)
           .end(( err, res ) => {
             expect(res.body).to.be.an('object')
             expect(res.body.success).to.be.an('boolean')
             expect(res.body.data).to.be.an('string')
             done()
           })
     })
   
     it('测试/getNumber.json请求', ( done ) => {
         request
           .get('/getNumber.json')
           .expect(200)
           .end(( err, res ) => {
             expect(res.body).to.be.an('object')
             expect(res.body.success).to.be.an('boolean')
             expect(res.body.data).to.be.an('number')
             done()
           })
     })
   })
   
   
   describe( '开始测试demo的POST请求', ( ) => {
     it('测试/postData.json请求', ( done ) => {
         request
           .post('/postData.json')
           .expect(200)
           .end(( err, res ) => {
             expect(res.body).to.be.an('object')
             expect(res.body.success).to.be.an('boolean')
             expect(res.body.data).to.be.an('string')
             done()
           })
     })
   })
   ```

5. 执行测试用例

   ```bash
   # node.js <= 7.5.x
   ./node_modules/.bin/mocha  --harmony
   
   # node.js = 7.6.0
   ./node_modules/.bin/mocha
   ```

   > 注意：
   >
   > 1. 如果是全局安装了mocha，可以直接在当前项目目录下执行 mocha --harmony 命令
   > 2. 如果当前node.js版本低于7.6，由于7.5.x以下还直接不支持async/awiar就需要加上--harmony

   会自动读取执行命令 ./test 目录下的测用例文件 inde.test.js，并执行。

### Log

#### 原生logger

- 修改`app.js`文件

  ```js
  // logger
  app.use(async (ctx, next) => {
    const start = new Date()
    await next()
    const ms = new Date() - start
    console.log(`${ctx.method} ${ctx.url} - ${ms}ms`)
  })
  ```

#### [koa-logger](https://github.com/koajs/logger)

用于koa的开发风格的日志中间件。

```
<-- GET /
--> GET / 200 835ms 746b
<-- GET /
--> GET / 200 960ms 1.9kb
<-- GET /users
--> GET /users 200 357ms 922b
<-- GET /users?page=2
--> GET /users?page=2 200 466ms 4.66kb
```

1. 安装

   ```bash
   $ npm install koa-logger
   ```

2. 示例

   ```js
   const logger = require('koa-logger')
   const Koa = require('koa')
   
   const app = new Koa()
   app.use(logger())
   ```

   扩展

   ```js
   const logger = require('koa-logger')
   const Moment = require('moment')
   
   // middleware
   app.use(logger((str,args) => {
     console.log(Moment().format('YYYY-MM-DD HH:mm:ss') + str)
   }))
   ```

   参数 str是带有ANSI Color的输出字符串，您可以使用其他模块(如strip-ansi)获得纯文本
   参数`args`是一个数组`[format, method, url, status, time, length]`

#### [koa-timer](https://github.com/koajs/timer)

记录时间的中间件

![image.png](https://i.loli.net/2021/05/09/rFYG65N2Bu4Slej.png)

1. 安装

   ```bash
   npm install koa-timer
   ```

2. 使用

   ```js
   const timer = require('koa-timer')
   const Koa = require('koa')
   
   const app = new Koa()
   app.use(timer())
   ```

#### [koa-accesslog](https://github.com/koajs/accesslog)

输出普通日志格式的访问日志到任何流。默认为process.stdout。

1. 安装

   ```ba
   npm i koa-accesslog
   ```

2. 使用

   ```js
   const Koa = require('koa');
   const accesslog = require('koa-accesslog');
   const app = new Koa();
   
   app.use(accesslog());
   ```

   输出

   ```
   127.0.0.1 - - [19/Nov/2014:13:47:37 +0100] "GET / HTTP/1.X" 404 -
   ```

   您可以配置Accesslog来使用任何可写流，例如流的实例。PassThrough如下所示。

   ```js
   const log = new stream.PassThrough();
   app.use(accesslog(log));
   ```

#### [koa-pino-logger](https://github.com/pinojs/koa-pino-logger)

koa-pino-logger是最快的JSON koa logger。

- `koa-bunyan-logger`: 5844 req/sec
- `koa-logger`: 9401 req/sec
- `koa-morgan`: 10881 req/sec
- `koa-pino-logger`: 10534 req/sec
- `koa-pino-logger` (extreme): 11112 req/sec
- koa w/out logger: 14518 req/sec

1. 安装

   ```bash
   npm install --save koa-pino-logger
   ```

2. 示例

   ```js
   //Request logging
   var koa = require('koa')
   var logger = require('koa-pino-logger')
   
   var app = new Koa()
   app.use(logger())
   
   app.use((ctx) => {
     ctx.log.info('something else')
     ctx.body = 'hello world'
   })
   
   app.listen(3000)
   
   
   //Thrown Error logging
   
   var koa = require('koa')
   var logger = require('koa-pino-logger')
   
   var app = new Koa()
   app.silent = true // disable console.errors
   app.use(logger())
   
   app.use((ctx) => {
     ctx.body = 'hello world'
     throw Error('bang!')
   })
   
   app.listen(3000)
   ```

#### [koa-onerror](https://github.com/koajs/onerror)

- 下载`koa-onerror`：`npm i koa-onerror --save`

- 修改`app.js`文件

  ```js
  const onerror = require('koa-onerror')
  
  // error handler
  onerror(app)
  
  app.on('error', (err, ctx) => {
    console.error('server', err, ctx)
  })
  ```

- 修改`app.js`,修改404设置，设置404显示页面

  ```js
  app.use(async (ctx, next) => {
    await next()
    if (ctx.status == 404) {
      ctx.status = 404;
      await ctx.render('404', {})
    }
  })
  ```

#### [log4js](https://github.com/log4js-node/log4js-node)

- 在根目录下新建`logger/`目录
- 在`logger/`目录下新建`logs/`目录，用来存放日志文件
- 在`logger/`目录下新建`log4js.js`文件



1. 安装

   ```bash
   npm install log4js
   ```

2. 示例`log4js.js`

   ```js
   const path = require('path')
   const log4js = require('log4js'); //加载log4js模块
   
   log4js.configure({
     appenders: {
       info: {
         type: "dateFile",
         filename: path.join(__dirname, 'logs', 'info', 'info'),// 您要写入日志文件的路径
         //compress: true, //（默认为false） - 在滚动期间压缩备份文件（备份文件将具有.gz扩展名）
         pattern: "yyyy-MM-dd.log", //（可选，默认为.yyyy-MM-dd） - 用于确定何时滚动日志的模式。格式:.yyyy-MM-dd-hh:mm:ss.log
         encoding: 'utf-8', // default "utf-8"，文件的编码
         // maxLogSize: 10000000, // 文件最大存储空间，当文件内容超过文件存储空间会自动生成一个文件xxx.log.1的序列自增长的文件
         alwaysIncludePattern: true, //（默认为false） - 将模式包含在当前日志文件的名称以及备份中
       },
       error: {// 错误日志
         type: 'dateFile',
         filename: path.join(__dirname, 'logs', 'error', 'error'),
         pattern: 'yyyy-MM-dd.log',
         encoding: 'utf-8', // default "utf-8"，文件的编码
         // maxLogSize: 10000000, // 文件最大存储空间，当文件内容超过文件存储空间会自动生成一个文件xxx.log.1的序列自增长的文件
         alwaysIncludePattern: true
       }
     },
     categories: {
       default: { appenders: ['info'], level: 'info' },
       info: { appenders: ['info'], level: 'info' },
       error: { appenders: ['error'], level: 'error' }
     }
   });
   
   
   /**
    * 错误日志记录方式
    * @param {*} content 日志输出内容
    */
   function logError(content) {
     const log = log4js.getLogger("error");
     log.error(content)
   }
   /**
    * 日志记录方式
    * @param {*} content 日志输出内容
    */
   function logInfo(content) {
     const log = log4js.getLogger("info");
     log.info(content)
   }
   
   module.exports = {
     logError,
     logInfo
   }
   ```

3. 修改app.js文件，捕获全局状态下的error

   ```js
   const log4js = require('./logger/log4js')
   
   app.on('error', (err, ctx) => {
   	log4js.logError(err)
   })
   app.on('error-info', (err, ctx) => {
   	log4js.logInfo(err)
   })
   ```

4. 在`try-catch`中可以使用`ctx.app.emit('error', e, ctx)`抛出错误；或者使用`ctx.app.emit('error-info', e, ctx)`抛出`info`错误

#### [koa-log4](https://github.com/dominhhai/koa-log4js)

`log4js-node`是一款比较好的在`node`环境下对于日志处理的模块

`koa-log4`在`log4js-node`的基础上做了一次包装，是`koa`的一个处理日志的中间件，此模块可以帮助你按照你配置的规则分叉日志消息。

- 在根目录下新建`logger/`目录
- 在`logger/`目录下新建`logs/`目录，用来存放日志文件
- 在`logger/`目录下新建`index.js`文件

1. 安装

   ```bash
   $ npm i --save koa-log4@2
   ```

2. 示例

   ```js
   const path = require('path')
   const log4js = require('koa-log4')
   
   log4js.configure({
     appenders: {
       access: {
         type: 'dateFile',
         // 生成文件的规则
         pattern: '-yyyy-MM-dd.log',
         // 文件名始终以日期区分
         alwaysIncludePattern: true,
         encoding: 'utf-8',
         // 生成文件路径和文件名
         filename: path.join(__dirname, 'logs', 'access')
       },
       application: {
         type: 'dateFile',
         pattern: '-yyyy-MM-dd.log',
         alwaysIncludePattern: true,
         encoding: 'utf-8',
         filename: path.join(__dirname, 'logs', 'application')
       },
       out: {
         type: 'console'
       }
     },
     categories: {
       default: { appenders: ['out'], level: 'info' },
       access: { appenders: ['access'], level: 'info' },
       application: { appenders: ['application'], level: 'WARN' }
     }
   })
   
   // // 记录所有访问级别的日志
   // exports.accessLogger = () => log4js.koaLogger(log4js.getLogger('access'))
   // // 记录所有应用级别的日志
   // exports.logger = log4js.getLogger('application')
   
   module.exports = {
     // 记录所有访问级别的日志
     accessLogger: () => log4js.koaLogger(log4js.getLogger('access')),
     // 记录所有应用级别的日志
     logger: log4js.getLogger('application')
   }
   ```

3. 访问级别的，记录用户的所有请求，作为koa的中间件，直接使用便可。修改app.js文件

   ```js
   const { accessLogger } = require('./logger/logger')
   
   app.use(accessLogger())
   ```

4. 应用级别的日志，可记录全局状态下的error，修改app.js全局捕捉异常

   ```js
   const { logger } = require('./logger/logger')
   
   // 在try-catch错误是无法监听的  
   // 需要手动释放：ctx.app.emit('error', err, ctx)
   // 或者在try-catch中直接logger.error(e)
   // 在需要的代码中放入即可监听
   app.on('error', (err, ctx) => {
     logger.error(err)
   
     // 这里可以优化下，开发环境才记录日志
     // if (process.env.NODE_ENV !== 'development') {
     //   logger.error(err)
     // }
   })
   ```

5. 应用级别的日志，也可记录接口请求当中的错误处理。

   ```js
   const { logger } = require('../logger/logger')
   
   router.get('/test', async ctx => {
     try {
       ddd()
       ctx.body = 'test'
     } catch (e) {
       logger.error(e)
       // 用这种方式手动释放也可以 - app.js文件里面，已经监听了error事件
       // ctx.app.emit('error', e, ctx)
       ctx.body = { code: -1, msg: e.message }
     }
   })
   ```

#### 全局错误捕捉

`app.js`文件中有个`app.on('error', (err, ctx) => { //... })`，
这个只能捕捉到应用的错误。比如在路由，在控制器中出现的错误捕捉不到

可以写一个中间件，用来捕捉全局的错误

修改`app.js`文件，添加捕获全局错误代码

```js
/* 错误捕捉中间件 */
app.use(async(ctx, next) => {
  try {
    // 这里给ctx对象添加了一个error方法
    // 后面的代码中，当执行ctx.error方法时，就会抛出一个异常
    // 这样，在其他的路由或中间件里，代码执行到ctx.error时就会直接跳回到这个的错误捕捉中间件，ctx.error后面的代码就不会再执行了
    ctx.error = (code, message) => {
      if (typeof code === 'string') {
        message = code
        code = 500
      }
      ctx.throw(code || 500, message || '服务器错误')
    }
    await next()
  } catch (e) {
    log4js.logError(e)
    const status = e.status || 500
    ctx.error(status, e.message || '服务器错误')

    // ctx.status = e.status || 500
    // ctx.body = e.message || '服务器错误'
  }
})
```

### WebSocket

websocket的选择了[ws](https://github.com/websockets/ws)，性能好，使用多，能承载的客户端数量也多。

1. 安装ws

   ```bash
   npm install ws --save
   ```

2. ws的基本使用

   ```js
   const WebSocket = require('ws');
   const wss = new WebSocket.Server({ port: 3000 });
   wss.on('connection', function connection(ws) {
     ws.on('message', function incoming(message) {
       console.log('received: %s', message);
     });
     ws.send('something');
   });
   ```

3. 与KOA共享端口

   有些时候不想让服务器开启两个端口，想让websocket和koa共享一个端口，先把ws封装成一个模块再说

    ```js
    //util/ws.js
    module.exports = wss => {
        wss.on('connection', function connection(ws) {
            ws.on('message', function incoming(message) {
                console.log('received: %s', message);
            });
            ws.send('something');
        });
    }
    ```

   接着在app.js中引入ws模块，这里要注意改写一下koa原本的简写方式

   ```js
   // app.js
   const http = require('http');
   const Koa = require('koa');
   const WebSocket = require('ws');
   const app = new Koa();
   const WebSocketApi = require('./util/ws');//引入封装的ws模块
   const server = http.createServer(app.callback())
   const wss = new WebSocket.Server({// 同一个端口监听不同的服务
       server
   });
   WebSocketApi(wss)
   server.listen(3000)
   ```

   

### 参考

1. [awesome-koa](https://github.com/fine)
2. [koa2 进阶学习](https://chenshenhai.github.io/koa2-note/)
3. [Koa.js 设计模式-学习笔记](https://chenshenhai.github.io/koajs-design-note/)
4. [ws 实现聊天室](https://www.liaoxuefeng.com/wiki/1022910821149312/1103332447876608)
5. [Awesome Koa](https://github.com/fineen/awesome-koa)