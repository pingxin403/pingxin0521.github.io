---
title: Egg.js -- 为企业级框架和应用而生
date: 2021-05-15 18:18:59
tags:
 - Node.JS
 - 后端
categories:
 - 后端
 - Node.JS
---

## Egg.js

**[Egg.js](https://eggjs.org/zh-cn/) 为企业级框架和应用而生**，解决koa的封装问题和提供Node异步编程方案

Egg 选择了 Koa 作为其基础框架，在它的模型基础上，进一步对它进行了一些增强。

<!--more-->

#### 原则

- 一个插件只做一件事
- 约定优于配置

#### 特性

- 提供基于 Egg [定制上层框架](https://eggjs.org/zh-cn/advanced/framework.html)的能力
- 高度可扩展的[插件机制](https://eggjs.org/zh-cn/basics/plugin.html)
- 内置[多进程管理](https://eggjs.org/zh-cn/advanced/cluster-client.html)
- 基于 [Koa](http://koajs.com/) 开发，性能优异
- 框架稳定，测试覆盖率高
- [渐进式开发](https://eggjs.org/zh-cn/tutorials/progressive.html)

#### 扩展

在基于 Egg 的框架或者应用中，我们可以通过定义 `app/extend/{application,context,request,response}.js` 来扩展 Koa 中对应的四个对象的原型，通过这个功能，我们可以快速的增加更多的辅助方法，例如我们在 `app/extend/context.js` 中写入下列代码：

```js
// app/extend/context.js
module.exports = {
  get isIOS() {
    const iosReg = /iphone|ipad|ipod/i;
    return iosReg.test(this.get('user-agent'));
  },
};
```

在 Controller 中，我们就可以使用到刚才定义的这个便捷属性了：

```js
// app/controller/home.js
exports.handler = ctx => {
  ctx.body = ctx.isIOS
    ? 'Your operating system is iOS.'
    : 'Your operating system is not iOS.';
};
```

更多关于扩展的内容，请查看[扩展](https://eggjs.org/zh-cn/basics/extend.html)章节。

#### 插件

众所周知，在 Express 和 Koa 中，经常会引入许许多多的中间件来提供各种各样的功能，例如引入 [koa-session](https://github.com/koajs/session) 提供 Session 的支持，引入 [koa-bodyparser](https://github.com/koajs/bodyparser) 来解析请求 body。而 Egg 提供了一个更加强大的插件机制，让这些独立领域的功能模块可以更加容易编写。

一个插件可以包含

- extend：扩展基础对象的上下文，提供各种工具类、属性。
- middleware：增加一个或多个中间件，提供请求的前置、后置处理逻辑。
- config：配置各个环境下插件自身的默认配置项。

一个独立领域下的插件实现，可以在代码维护性非常高的情况下实现非常完善的功能，而插件也支持配置各个环境下的默认（最佳）配置，让我们使用插件的时候几乎可以不需要修改配置项。

[egg-security](https://github.com/eggjs/egg-security) 插件就是一个典型的例子。

#### 版本

- egg.js 1.x ==> koa 1.x
- egg.js 2.x ==> koa 2.x

### 快速入门

> 环境： node >=8.x LTS

#### 快速初始化

使用脚手架

```bash
$ mkdir egg-example && cd egg-example
$ npm init egg --type=simple
$ npm i
```

启动项目:

```bash
$ npm run dev
$ open http://localhost:7001
```

效果：

![image.png](https://i.loli.net/2021/05/16/3cq57lYoRxmXhnu.png)



### 基础知识

#### 目录结构

简单了解下目录约定规范。

```bash
egg-project
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
|   ├── router.js
│   ├── controller
│   |   └── home.js
│   ├── service (可选)
│   |   └── user.js
│   ├── middleware (可选)
│   |   └── response_time.js
│   ├── schedule (可选)
│   |   └── my_task.js
│   ├── public (可选)
│   |   └── reset.css
│   ├── view (可选)
│   |   └── home.tpl
│   └── extend (可选)
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config
|   ├── plugin.js
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

如上，由框架约定的目录：

- `app/router.js` 用于配置 URL 路由规则，具体参见 [Router](https://eggjs.org/zh-cn/basics/router.html)。
- `app/controller/**` 用于解析用户的输入，处理后返回相应的结果，具体参见 [Controller](https://eggjs.org/zh-cn/basics/controller.html)。
- `app/service/**` 用于编写业务逻辑层，可选，建议使用，具体参见 [Service](https://eggjs.org/zh-cn/basics/service.html)。
- `app/middleware/**` 用于编写中间件，可选，具体参见 [Middleware](https://eggjs.org/zh-cn/basics/middleware.html)。
- `app/public/**` 用于放置静态资源，可选，具体参见内置插件 [egg-static](https://github.com/eggjs/egg-static)。
- `app/extend/**` 用于框架的扩展，可选，具体参见[框架扩展](https://eggjs.org/zh-cn/basics/extend.html)。
- `config/config.{env}.js` 用于编写配置文件，具体参见[配置](https://eggjs.org/zh-cn/basics/config.html)。
- `config/plugin.js` 用于配置需要加载的插件，具体参见[插件](https://eggjs.org/zh-cn/basics/plugin.html)。
- `test/**` 用于单元测试，具体参见[单元测试](https://eggjs.org/zh-cn/core/unittest.html)。
- `app.js` 和 `agent.js` 用于自定义启动时的初始化工作，可选，具体参见[启动自定义](https://eggjs.org/zh-cn/basics/app-start.html)。关于`agent.js`的作用参见[Agent机制](https://eggjs.org/zh-cn/core/cluster-and-ipc.html#agent-机制)。

由内置插件约定的目录：

- `app/public/**` 用于放置静态资源，可选，具体参见内置插件 [egg-static](https://github.com/eggjs/egg-static)。
- `app/schedule/**` 用于定时任务，可选，具体参见[定时任务](https://eggjs.org/zh-cn/basics/schedule.html)。

**若需自定义自己的目录规范，参见 [Loader API](https://eggjs.org/zh-cn/advanced/loader.html)**

- `app/view/**` 用于放置模板文件，可选，由模板插件约定，具体参见[模板渲染](https://eggjs.org/zh-cn/core/view.html)。
- `app/model/**` 用于放置领域模型，可选，由领域类相关插件约定，如 [egg-sequelize](https://github.com/eggjs/egg-sequelize)。

#### 内置基础对象

1. Application 是全局应用对象，在一个应用中，只会实例化一个，它继承自 [Koa.Application](http://koajs.com/#application)，在它上面我们可以挂载一些全局的方法和对象，可以轻松的在插件或者应用中[扩展 Application 对象](https://eggjs.org/zh-cn/basics/extend.html#Application)。

2. Context 是一个**请求级别的对象**，继承自 [Koa.Context](http://koajs.com/#context)。在每一次收到用户请求时，框架会实例化一个 Context 对象，这个对象封装了这次用户请求的信息，并提供了许多便捷的方法来获取请求参数或者设置响应信息。框架会将所有的 Service 挂载到 Context 实例上，一些插件也会将一些其他的方法和对象挂载到它上面（[egg-sequelize](https://github.com/eggjs/egg-sequelize) 会将所有的 model 挂载在 Context 上）。

3. Request 是一个**请求级别的对象**，继承自 [Koa.Request](http://koajs.com/#request)。封装了 Node.js 原生的 HTTP Request 对象，提供了一系列辅助方法获取 HTTP 请求常用参数。

4. Response 是一个**请求级别的对象**，继承自 [Koa.Response](http://koajs.com/#response)。封装了 Node.js 原生的 HTTP Response 对象，提供了一系列辅助方法设置 HTTP 响应。

5. Controller: 框架提供了一个 Controller 基类，并推荐所有的 Controller 都继承于该基类实现。这个 Controller 基类有下列属性：

   - `ctx` - 当前请求的 Context 实例。
   - `app` - 应用的 Application 实例。
   - `config` - 应用的[配置](https://eggjs.org/zh-cn/basics/config.html)。
   - `service` - 应用所有的 [service](https://eggjs.org/zh-cn/basics/service.html)。
   - `logger` - 为当前 controller 封装的 logger 对象。

6. Service: 框架提供了一个 Service 基类，并推荐所有的 Service 都继承于该基类实现。

   Service 基类的属性和 Controller 基类属性一致，访问方式也类似

7. Helper 用来提供一些实用的 utility 函数。它的作用在于我们可以将一些常用的动作抽离在 helper.js 里面成为一个独立的函数，这样可以用 JavaScript 来写复杂的逻辑，避免逻辑分散各处，同时可以更好的编写测试用例。

   Helper 自身是一个类，有和 Controller 基类一样的属性，它也会在每次请求时进行实例化，因此 Helper 上的所有函数也能获取到当前请求相关的上下文信息。

8. Config: 所有框架、插件和应用级别的配置都可以通过 Config 对象获取到，关于框架的配置，可以详细阅读 [Config 配置](https://eggjs.org/zh-cn/basics/config.html)章节。

9. Logger: 框架内置了功能强大的[日志功能](https://eggjs.org/zh-cn/core/logger.html)，可以非常方便的打印各种级别的日志到对应的日志文件中

10. Subscription: 订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务

具体对象获取位置参考:<https://eggjs.org/zh-cn/basics/objects.html>

#### 运行环境

一个 Web 应用本身应该是无状态的，并拥有根据运行环境设置自身的能力。

框架有两种方式指定运行环境：

1. 通过 `config/env` 文件指定，该文件的内容就是运行环境，如 `prod`。一般通过构建工具来生成这个文件。

   ```
   // config/env
   prod
   ```

2. 通过 `EGG_SERVER_ENV` 环境变量指定运行环境更加方便，比如在生产环境启动应用：

   ```
   EGG_SERVER_ENV=prod npm start
   ```

框架提供了变量 `app.config.env` 来表示应用当前的运行环境。

**NODE_ENV与EGG_SERVER_ENV区别：**

| NODE_ENV   | EGG_SERVER_ENV | 说明         |
| ---------- | -------------- | ------------ |
|            | local          | 本地开发环境 |
| test       | unittest       | 单元测试     |
| production | prod           | 生产环境     |

例如，当 `NODE_ENV` 为 `production` 而 `EGG_SERVER_ENV` 未指定时，框架会将 `EGG_SERVER_ENV` 设置成 `prod`。

**自定义环境**

常规开发流程可能不仅仅只有以上几种环境，Egg 支持自定义环境来适应自己的开发流程。

比如，要为开发流程增加集成测试环境 SIT。将 `EGG_SERVER_ENV` 设置成 `sit`（并建议设置 `NODE_ENV = production`），启动时会加载 `config/config.sit.js`，运行环境变量 `app.config.env` 会被设置成 `sit`。

**与 Koa 的区别**

在 Koa 中我们通过 `app.env` 来进行环境判断，`app.env` 默认的值是 `process.env.NODE_ENV`。但是在 Egg（和基于 Egg 的框架）中，配置统一都放置在 `app.config` 上，所以我们需要通过 `app.config.env` 来区分环境，`app.env` 不再使用。

#### Config 配置

框架提供了强大且可扩展的配置功能，可以自动合并应用、插件、框架的配置，按顺序覆盖，且可以根据环境维护不同的配置。合并后的配置可直接从 `app.config` 获取。

使用代码管理配置，在代码中添加多个环境的配置，在启动时传入当前环境的参数即可。但无法全局配置，必须修改代码。

##### 多环境配置

框架支持根据环境来加载配置，定义多个环境的配置文件：

```
config
|- config.default.js
|- config.prod.js
|- config.unittest.js
`- config.local.js
```

`config.default.js` 为默认的配置文件，所有环境都会加载这个配置文件，一般也会作为开发环境的默认配置文件。

当指定 env 时会同时加载默认配置和对应的配置(具名配置)文件，具名配置和默认配置将合并(使用[extend2](https://www.npmjs.com/package/extend2)深拷贝)成最终配置，具名配置项会覆盖默认配置文件的同名配置。如 `prod` 环境会加载 `config.prod.js` 和 `config.default.js` 文件，`config.prod.js` 会覆盖 `config.default.js` 的同名配置。

##### 配置写法

配置文件返回的是一个 object 对象，可以覆盖框架的一些配置，应用也可以将自己业务的配置放到这里方便管理。

```js
// 配置 logger 文件的目录，logger 默认配置由框架提供
module.exports = {
  logger: {
    dir: '/home/admin/logs/demoapp',
  },
};
```

配置文件也可以简化的写成 `exports.key = value` 形式

```js
exports.keys = 'my-cookie-secret-key';
exports.logger = {
  level: 'DEBUG',
};
```

配置文件也可以返回一个 function，可以接受 appInfo 参数

```js
// 将 logger 目录放到代码目录下
const path = require('path');
module.exports = appInfo => {
  return {
    logger: {
      dir: path.join(appInfo.baseDir, 'logs'),
    },
  };
};
```

内置的 appInfo 有

| appInfo | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| pkg     | package.json                                                 |
| name    | 应用名，同 pkg.name                                          |
| baseDir | 应用代码的目录                                               |
| HOME    | 用户目录，如 admin 账户为 /home/admin                        |
| root    | 应用根目录，只有在 local 和 unittest 环境下为 baseDir，其他都为 HOME。 |

`appInfo.root` 是一个优雅的适配，比如在服务器环境我们会使用 `/home/admin/logs` 作为日志目录，而本地开发时又不想污染用户目录，这样的适配就很好解决这个问题。

请根据具体场合选择合适的写法，但请确保没有写出以下代码：

```js
// config/config.default.js
exports.someKeys = 'abc';
module.exports = appInfo => {
  const config = {};
  config.keys = '123456';
  return config;
};
```

##### 配置加载顺序

应用、插件、框架都可以定义这些配置，而且目录结构都是一致的，但存在优先级（应用 > 框架 > 插件），相对于此运行环境的优先级会更高。

比如在 prod 环境加载一个配置的加载顺序如下，后加载的会覆盖前面的同名配置。

```
-> 插件 config.default.js
-> 框架 config.default.js
-> 应用 config.default.js
-> 插件 config.prod.js
-> 框架 config.prod.js
-> 应用 config.prod.js
```

**注意：插件之间也会有加载顺序，但大致顺序类似，具体逻辑可[查看加载器](https://eggjs.org/zh-cn/advanced/loader.html)。**

##### 配置结果

框架在启动时会把合并后的最终配置 dump 到 `run/application_config.json`（worker 进程）和 `run/agent_config.json`（agent 进程）中，可以用来分析问题。

配置文件中会隐藏一些字段，主要包括两类:

- 如密码、密钥等安全字段，这里可以通过 `config.dump.ignore` 配置，必须是 [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) 类型，查看[默认配置](https://github.com/eggjs/egg/blob/master/config/config.default.js)。
- 如函数、Buffer 等类型，`JSON.stringify` 后的内容特别大

还会生成 `run/application_config_meta.json`（worker 进程）和 `run/agent_config_meta.json`（agent 进程）文件，用来排查属性的来源，如

```json
{
  "logger": {
    "dir": "/path/to/config/config.default.js"
  }
}
```

#### 中间件（Middleware）

Egg 是基于 Koa 实现的，所以 Egg 的中间件形式和 Koa 的中间件形式是一样的，都是基于洋葱圈模型。每次我们编写一个中间件，就相当于在洋葱外面包了一层。

##### 写法

我们先来通过编写一个简单的 gzip 中间件，来看看中间件的写法。

```js
// app/middleware/gzip.js
const isJSON = require('koa-is-json');
const zlib = require('zlib');

async function gzip(ctx, next) {
  await next();

  // 后续中间件执行完成后将响应体转换成 gzip
  let body = ctx.body;
  if (!body) return;
  if (isJSON(body)) body = JSON.stringify(body);

  // 设置 gzip body，修正响应头
  const stream = zlib.createGzip();
  stream.end(body);
  ctx.body = stream;
  ctx.set('Content-Encoding', 'gzip');
}
```

可以看到，框架的中间件和 Koa 的中间件写法是一模一样的，所以任何 Koa 的中间件都可以直接被框架使用。

##### 配置

一般来说中间件也会有自己的配置。在框架中，一个完整的中间件是包含了配置处理的。我们约定一个中间件是一个放置在 `app/middleware` 目录下的单独文件，它需要 exports 一个普通的 function，接受两个参数：

- options: 中间件的配置项，框架会将 `app.config[${middlewareName}]` 传递进来。
- app: 当前应用 Application 的实例。

我们将上面的 gzip 中间件做一个简单的优化，让它支持指定只有当 body 大于配置的 threshold 时才进行 gzip 压缩，我们要在 `app/middleware` 目录下新建一个文件 `gzip.js`

```js
// app/middleware/gzip.js
const isJSON = require('koa-is-json');
const zlib = require('zlib');

module.exports = options => {
  return async function gzip(ctx, next) {
    await next();

    // 后续中间件执行完成后将响应体转换成 gzip
    let body = ctx.body;
    if (!body) return;

    // 支持 options.threshold
    if (options.threshold && ctx.length < options.threshold) return;

    if (isJSON(body)) body = JSON.stringify(body);

    // 设置 gzip body，修正响应头
    const stream = zlib.createGzip();
    stream.end(body);
    ctx.body = stream;
    ctx.set('Content-Encoding', 'gzip');
  };
};
```

##### 使用中间件

中间件编写完成后，我们还需要手动挂载，支持以下方式：

1. 在应用中使用中间件。在 `config.default.js` 中加入下面的配置就完成了中间件的开启和配置：

   ```js
   module.exports = {
     // 配置需要的中间件，数组顺序即为中间件的加载顺序
     middleware: [ 'gzip' ],
   
     // 配置 gzip 中间件的配置
     gzip: {
       threshold: 1024, // 小于 1k 的响应体不压缩
     },
   };
   ```

   该配置最终将在启动时合并到 `app.config.appMiddleware`。

2. 在框架和插件中使用中间件。框架和插件不支持在 `config.default.js` 中匹配 `middleware`，需要通过以下方式：

   ```js
   // app.js
   module.exports = app => {
     // 在中间件最前面统计请求时间
     app.config.coreMiddleware.unshift('report');
   };
   
   // app/middleware/report.js
   module.exports = () => {
     return async function (ctx, next) {
       const startTime = Date.now();
       await next();
       // 上报请求时间
       reportTime(Date.now() - startTime);
     }
   };
   ```

   应用层定义的中间件（`app.config.appMiddleware`）和框架默认中间件（`app.config.coreMiddleware`）都会被加载器加载，并挂载到 `app.middleware` 上。

3. router 中使用中间件。以上两种方式配置的中间件是全局的，会处理每一次请求。 如果你只想针对单个路由生效，可以直接在 `app/router.js` 中实例化和挂载，如下：

   ```js
   module.exports = app => {
     const gzip = app.middleware.gzip({ threshold: 1024 });
     app.router.get('/needgzip', gzip, app.controller.handler);
   };
   ```

##### 框架默认中间件

除了应用层加载中间件之外，框架自身和其他的插件也会加载许多中间件。所有的这些自带中间件的配置项都通过在配置中修改中间件同名配置项进行修改，例如[框架自带的中间件](https://github.com/eggjs/egg/tree/master/app/middleware)中有一个 bodyParser 中间件（框架的加载器会将文件名中的各种分隔符都修改成驼峰形式的变量名），我们想要修改 bodyParser 的配置，只需要在 `config/config.default.js` 中编写

```js
module.exports = {
  bodyParser: {
    jsonLimit: '10mb',
  },
};
```

**注意：框架和插件加载的中间件会在应用层配置的中间件之前，框架默认中间件不能被应用层中间件覆盖，如果应用层有自定义同名中间件，在启动时会报错。**

##### 使用 Koa 的中间件

在框架里面可以非常容易的引入 Koa 中间件生态。

以 [koa-compress](https://github.com/koajs/compress) 为例，在 Koa 中使用时：

```js
const koa = require('koa');
const compress = require('koa-compress');

const app = koa();

const options = { threshold: 2048 };
app.use(compress(options));
```

我们按照框架的规范来在应用中加载这个 Koa 的中间件：

```js
// app/middleware/compress.js
// koa-compress 暴露的接口(`(options) => middleware`)和框架对中间件要求一致
module.exports = require('koa-compress');
// config/config.default.js
module.exports = {
  middleware: [ 'compress' ],
  compress: {
    threshold: 2048,
  },
};
```

如果使用到的 Koa 中间件不符合入参规范，则可以自行处理下：

```js
// config/config.default.js
module.exports = {
  webpack: {
    compiler: {},
    others: {},
  },
};

// app/middleware/webpack.js
const webpackMiddleware = require('some-koa-middleware');

module.exports = (options, app) => {
  return webpackMiddleware(options.compiler, options.others);
}
```

##### 通用配置

无论是应用层加载的中间件还是框架自带中间件，都支持几个通用的配置项：

- enable：控制中间件是否开启。

- match：设置只有符合某些规则的请求才会经过这个中间件。

- ignore：设置符合某些规则的请求不经过这个中间件。

  1. 字符串：当参数为字符串类型时，配置的是一个 url 的路径前缀，所有以配置的字符串作为前缀的 url 都会匹配上。 当然，你也可以直接使用字符串数组。
  2. 正则：当参数为正则时，直接匹配满足正则验证的 url 的路径。
  3. 函数：当参数为一个函数时，会将请求上下文传递给这个函数，最终取函数返回的结果（true/false）来判断是否匹配。

  有关更多的 match 和 ignore 配置情况，详见 [egg-path-matching](https://github.com/eggjs/egg-path-matching).

#### 路由（Router）

Router 主要用来描述请求 URL 和具体承担执行动作的 Controller 的对应关系， 框架约定了 `app/router.js` 文件用于统一所有路由规则。

通过统一的配置，我们可以避免路由规则逻辑散落在多个地方，从而出现未知的冲突，集中在一起我们可以更方便的来查看全局的路由规则。

##### 如何定义 Router

- `app/router.js` 里面定义 URL 路由规则

  ```js
  // app/router.js
  module.exports = app => {
    const { router, controller } = app;
    router.get('/user/:id', controller.user.info);
  };
  ```

- `app/controller` 目录下面实现 Controller

  ```js
  // app/controller/user.js
  class UserController extends Controller {
    async info() {
      const { ctx } = this;
      ctx.body = {
        name: `hello ${ctx.params.id}`,
      };
    }
  }
  ```

这样就完成了一个最简单的 Router 定义，当用户执行 `GET /user/123`，`user.js` 这个里面的 info 方法就会执行。

##### Router 详细定义说明

下面是路由的完整定义，参数可以根据场景的不同，自由选择：

```js
router.verb('path-match', app.controller.action);
router.verb('router-name', 'path-match', app.controller.action);
router.verb('path-match', middleware1, ..., middlewareN, app.controller.action);
router.verb('router-name', 'path-match', middleware1, ..., middlewareN, app.controller.action);
```

路由完整定义主要包括5个主要部分:

-  verb - 用户触发动作，支持 get，post 等所有 HTTP 方法，后面会通过示例详细说明。
  - router.head - HEAD
  - router.options - OPTIONS
  - router.get - GET
  - router.put - PUT
  - router.post - POST
  - router.patch - PATCH
  - router.delete - DELETE
  - router.del - 由于 delete 是一个保留字，所以提供了一个 delete 方法的别名。
  - router.redirect - 可以对 URL 进行重定向处理，比如我们最经常使用的可以把用户访问的根目录路由到某个主页。
- router-name 给路由设定一个别名，可以通过 Helper 提供的辅助函数 pathFor 和 urlFor 来生成 URL。(可选)
- path-match - 路由 URL 路径。
- middleware1 - 在 Router 里面可以配置多个 Middleware。(可选)
- controller - 指定路由映射到的具体的 controller 上，controller 可以有两种写法：
  - app.controller.user.fetch - 直接指定一个具体的 controller
  - 'user.fetch' - 可以简写为字符串形式

**注意事项**

- 在 Router 定义中， 可以支持多个 Middleware 串联执行
- Controller 必须定义在 `app/controller` 目录中。
- 一个文件里面也可以包含多个 Controller 定义，在定义路由的时候，可以通过 `${fileName}.${functionName}` 的方式指定对应的 Controller。
- Controller 支持子目录，在定义路由的时候，可以通过 `${directoryName}.${fileName}.${functionName}` 的方式指定对应的 Controller。

下面是一些路由定义的方式：

```js
// app/router.js
module.exports = app => {
  const { router, controller } = app;
  router.get('/home', controller.home);
  router.get('/user/:id', controller.user.page);
  router.post('/admin', isAdmin, controller.admin);
  router.post('/user', isLoginUser, hasAdminPermission, controller.user.create);
  router.post('/api/v1/comments', controller.v1.comments.create); // app/controller/v1/comments.js
};
```

##### RESTful 风格的 URL 定义

如果想通过 RESTful 的方式来定义路由， 提供了 `app.router.resources('routerName', 'pathMatch', controller)` 快速在一个路径上生成 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 路由结构。

```js
// app/router.js
module.exports = app => {
  const { router, controller } = app;
  router.resources('posts', '/api/posts', controller.posts);
  router.resources('users', '/api/v1/users', controller.v1.users); // app/controller/v1/users.js
};
```

上面代码就在 `/posts` 路径上部署了一组 CRUD 路径结构，对应的 Controller 为 `app/controller/posts.js` 接下来， 你只需要在 `posts.js` 里面实现对应的函数就可以了。

| Method | Path            | Route Name | Controller.Action             |
| ------ | --------------- | ---------- | ----------------------------- |
| GET    | /posts          | posts      | app.controllers.posts.index   |
| GET    | /posts/new      | new_post   | app.controllers.posts.new     |
| GET    | /posts/:id      | post       | app.controllers.posts.show    |
| GET    | /posts/:id/edit | edit_post  | app.controllers.posts.edit    |
| POST   | /posts          | posts      | app.controllers.posts.create  |
| PUT    | /posts/:id      | post       | app.controllers.posts.update  |
| DELETE | /posts/:id      | post       | app.controllers.posts.destroy |

```js
// app/controller/posts.js
exports.index = async () => {};

exports.new = async () => {};

exports.create = async () => {};

exports.show = async () => {};

exports.edit = async () => {};

exports.update = async () => {};

exports.destroy = async () => {};
```

如果我们不需要其中的某几个方法，可以不用在 `posts.js` 里面实现，这样对应 URL 路径也不会注册到 Router。

更多示例参考：<https://eggjs.org/zh-cn/basics/router.html#router-%E5%AE%9E%E6%88%98>

##### 太多路由映射？

若确实有需求，可以如下拆分：

```
// app/router.js
module.exports = app => {
  require('./router/news')(app);
  require('./router/admin')(app);
};

// app/router/news.js
module.exports = app => {
  app.router.get('/news/list', app.controller.news.list);
  app.router.get('/news/detail', app.controller.news.detail);
};

// app/router/admin.js
module.exports = app => {
  app.router.get('/admin/user', app.controller.admin.user);
  app.router.get('/admin/log', app.controller.admin.log);
};
```

也可直接使用 [egg-router-plus](https://github.com/eggjs/egg-router-plus)。

#### 控制器（Controller）

Controller 负责**解析用户的输入，处理后返回相应的结果**，例如

- 在 [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) 接口中，Controller 接受用户的参数，从数据库中查找内容返回给用户或者将用户的请求更新到数据库中。
- 在 HTML 页面请求中，Controller 根据用户访问不同的 URL，渲染不同的模板得到 HTML 返回给用户。
- 在代理服务器中，Controller 将用户的请求转发到其他服务器上，并将其他服务器的处理结果返回给用户

框架推荐 Controller 层主要对用户的请求参数进行处理（校验、转换），然后调用对应的 [service](https://eggjs.org/zh-cn/basics/service.html) 方法处理业务，得到业务结果后封装并返回：

1. 获取用户通过 HTTP 传递过来的请求参数。
2. 校验、组装参数。
3. 调用 Service 进行业务处理，必要时处理转换 Service 的返回结果，让它适应用户的需求。
4. 通过 HTTP 将结果响应给用户。

##### 如何编写 Controller

所有的 Controller 文件都必须放在 `app/controller` 目录下，可以支持多级目录，访问的时候可以通过目录名级联访问。Controller 支持多种形式进行编写，可以根据不同的项目场景和开发习惯来选择。

1. Controller 类（推荐）

   通过定义 Controller 类的方式来编写代码：

   ```js
   // app/controller/post.js
   const Controller = require('egg').Controller;
   class PostController extends Controller {
     async create() {
       const { ctx, service } = this;
       const createRule = {
         title: { type: 'string' },
         content: { type: 'string' },
       };
       // 校验参数
       ctx.validate(createRule);
       // 组装参数
       const author = ctx.session.userId;
       const req = Object.assign(ctx.request.body, { author });
       // 调用 Service 进行业务处理
       const res = await service.post.create(req);
       // 设置响应内容和响应状态码
       ctx.body = { id: res.id };
       ctx.status = 201;
     }
   }
   module.exports = PostController;
   ```

   我们通过上面的代码定义了一个 `PostController` 的类，类里面的每一个方法都可以作为一个 Controller 在 Router 中引用到，我们可以从 `app.controller` 根据文件名和方法名定位到它。

   ```js
   // app/router.js
   module.exports = app => {
     const { router, controller } = app;
     router.post('createPost', '/api/posts', controller.post.create);
   }
   ```

   定义的 Controller 类，会在每一个请求访问到 server 时实例化一个全新的对象，而项目中的 Controller 类继承于 `egg.Controller`，会有下面几个属性挂在 `this` 上。

   - `this.ctx`: 当前请求的上下文 Context 对象的实例，通过它我们可以拿到框架封装好的处理当前请求的各种便捷属性和方法。
   - `this.app`: 当前应用 Application 对象的实例，通过它我们可以拿到框架提供的全局对象和方法。
   - `this.service`：应用定义的 Service，通过它我们可以访问到抽象出的业务层，等价于 `this.ctx.service` 。
   - `this.config`：应用运行时的配置项。
   - `this.logger`：logger 对象，上面有四个方法（`debug`，`info`，`warn`，`error`），分别代表打印四个不同级别的日志，使用方法和效果与 context logger 中介绍的一样，但是通过这个 logger 对象记录的日志，在日志前面会加上打印该日志的文件路径，以便快速定位日志打印位置。

2. 自定义 Controller 基类

   按照类的方式编写 Controller，不仅可以让我们更好的对 Controller 层代码进行抽象（例如将一些统一的处理抽象成一些私有方法），还可以通过自定义 Controller 基类的方式封装应用中常用的方法。

   ```js
   // app/core/base_controller.js
   const { Controller } = require('egg');
   class BaseController extends Controller {
     get user() {
       return this.ctx.session.user;
     }
   
     success(data) {
       this.ctx.body = {
         success: true,
         data,
       };
     }
   
     notFound(msg) {
       msg = msg || 'not found';
       this.ctx.throw(404, msg);
     }
   }
   module.exports = BaseController;
   ```

   此时在编写应用的 Controller 时，可以继承 BaseController，直接使用基类上的方法

##### 调用 Service

我们并不想在 Controller 中实现太多业务逻辑，所以提供了一个 Service 层进行业务逻辑的封装，这不仅能提高代码的复用性，同时可以让我们的业务逻辑更好测试。

在 Controller 中可以调用任何一个 Service 上的任何方法，同时 Service 是懒加载的，只有当访问到它的时候框架才会去实例化它。

```js
class PostController extends Controller {
  async create() {
    const ctx = this.ctx;
    const author = ctx.session.userId;
    const req = Object.assign(ctx.request.body, { author });
    // 调用 service 进行业务处理
    const res = await ctx.service.post.create(req);
    ctx.body = { id: res.id };
    ctx.status = 201;
  }
}
```

更多配置参考:<https://eggjs.org/zh-cn/basics/controller.html>

#### 服务（Service）

简单来说，Service 就是在复杂业务场景下用于做业务逻辑封装的一个抽象层，提供这个抽象有以下几个好处：

- 保持 Controller 中的逻辑更加简洁。
- 保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
- 将逻辑和展现分离，更容易编写测试用例。

**使用场景**

- 复杂数据的处理，比如要展现的信息需要从数据库获取，还要经过一定的规则计算，才能返回用户显示。或者计算完成后，更新到数据库。
- 第三方服务的调用，比如 GitHub 信息获取等。

##### 定义 Service

```js
// app/service/user.js
const Service = require('egg').Service;

class UserService extends Service {
  async find(uid) {
    const user = await this.ctx.db.query('select * from user where uid = ?', uid);
    return user;
  }
}

module.exports = UserService;
```

##### 属性

每一次用户请求，框架都会实例化对应的 Service 实例，由于它继承于 `egg.Service`，故拥有下列属性方便我们进行开发：

- `this.ctx`: 当前请求的上下文 Context 对象的实例，通过它我们可以拿到框架封装好的处理当前请求的各种便捷属性和方法。
- `this.app`: 当前应用 Application 对象的实例，通过它我们可以拿到框架提供的全局对象和方法。
- `this.service`：应用定义的 Service，通过它我们可以访问到其他业务层，等价于 `this.ctx.service` 。
- `this.config`：应用运行时的配置项。
- `this.logger`：logger 对象，上面有四个方法（`debug`，`info`，`warn`，`error`），分别代表打印四个不同级别的日志，使用方法和效果与 context logger 中介绍的一样，但是通过这个 logger 对象记录的日志，在日志前面会加上打印该日志的文件路径，以便快速定位日志打印位置。

为了可以获取用户请求的链路，我们在 Service 初始化中，注入了请求上下文, 用户在方法中可以直接通过 `this.ctx` 来获取上下文相关信息。有了 ctx 我们可以拿到框架给我们封装的各种便捷属性和方法。比如我们可以用：

- `this.ctx.curl` 发起网络调用。
- `this.ctx.service.otherService` 调用其他 Service。
- `this.ctx.db` 发起数据库调用等， db 可能是其他插件提前挂载到 app 上的模块。

##### 注意事项

- Service 文件必须放在 `app/service` 目录，可以支持多级目录，访问的时候可以通过目录名级联访问。

  ```
  app/service/biz/user.js => ctx.service.biz.user
  app/service/sync_user.js => ctx.service.syncUser
  app/service/HackerNews.js => ctx.service.hackerNews
  ```

- 一个 Service 文件只能包含一个类， 这个类需要通过 `module.exports` 的方式返回。

- Service 需要通过 Class 的方式定义，父类必须是 `egg.Service`。

- Service 不是单例，是 **请求级别** 的对象，框架在每次请求中首次访问 `ctx.service.xx` 时延迟实例化，所以 Service 中可以通过 this.ctx 获取到当前请求的上下文。

#### 插件

一个插件其实就是一个『迷你的应用』，和应用（app）几乎一样：

- 它包含了 Service、中间件、配置、框架扩展等等。
- 它没有独立的 Router 和 Controller。
- 它没有 `plugin.js`，只能声明跟其他插件的依赖，而**不能决定**其他插件的开启与否。

##### 使用插件

插件一般通过 npm 模块的方式进行复用：

```
$ npm i egg-mysql --save
```

然后需要在应用或框架的 `config/plugin.js` 中声明：

```js
// config/plugin.js
// 使用 mysql 插件
exports.mysql = {
  enable: true,
  package: 'egg-mysql',
};
```

就可以直接使用插件提供的功能：

```js
app.mysql.query(sql, values);
```

##### 参数介绍

`plugin.js` 中的每个配置项支持：

- `{Boolean} enable` - 是否开启此插件，默认为 true
- `{String} package` - `npm` 模块名称，通过 `npm` 模块形式引入插件
- `{String} path` - 插件绝对路径，跟 package 配置互斥。如应用内部抽了一个插件，但还没达到开源发布独立 `npm` 的阶段，或者是应用自己覆盖了框架的一些插件，参见[渐进式开发](https://eggjs.org/zh-cn/tutorials/progressive.html)。
- `{Array} env` - 只有在指定运行环境才能开启，会覆盖插件自身 `package.json` 中的配置

在上层框架内部内置的插件，应用在使用时就不用配置 package 或者 path，只需要指定 enable 与否：

```js
// 对于内置插件，可以用下面的简洁方式开启或关闭
exports.onerror = false;
```

##### 插件配置

插件一般会包含自己的默认配置，应用开发者可以在 `config.default.js` 覆盖对应的配置：

```js
// config/config.default.js
exports.mysql = {
  client: {
    host: 'mysql.com',
    port: '3306',
    user: 'test_user',
    password: 'test_password',
    database: 'test',
  },
};
```

##### 插件列表

- 框架默认内置了企业级应用常用的插件：
  - [onerror](https://github.com/eggjs/egg-onerror) 统一异常处理
  - [Session](https://github.com/eggjs/egg-session) Session 实现
  - [i18n](https://github.com/eggjs/egg-i18n) 多语言
  - [watcher](https://github.com/eggjs/egg-watcher) 文件和文件夹监控
  - [multipart](https://github.com/eggjs/egg-multipart) 文件流式上传
  - [security](https://github.com/eggjs/egg-security) 安全
  - [development](https://github.com/eggjs/egg-development) 开发环境配置
  - [logrotator](https://github.com/eggjs/egg-logrotator) 日志切分
  - [schedule](https://github.com/eggjs/egg-schedule) 定时任务
  - [static](https://github.com/eggjs/egg-static) 静态服务器
  - [jsonp](https://github.com/eggjs/egg-jsonp) jsonp 支持
  - [view](https://github.com/eggjs/egg-view) 模板引擎
- 更多社区的插件可以 GitHub 搜索 [egg-plugin](https://github.com/topics/egg-plugin)。

##### 插件开发

你可以直接使用 [egg-boilerplate-plugin](https://github.com/eggjs/egg-boilerplate-plugin) 脚手架来快速上手。

```bash
$ mkdir egg-hello && cd egg-hello
$ npm init egg --type=plugin
$ npm i
$ npm test
```

**目录结构**

一个插件其实就是一个『迷你的应用』，下面展示的是一个插件的目录结构，和应用（app）几乎一样。

```
. egg-hello
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
│   ├── extend (可选)
│   |   ├── helper.js (可选)
│   |   ├── request.js (可选)
│   |   ├── response.js (可选)
│   |   ├── context.js (可选)
│   |   ├── application.js (可选)
│   |   └── agent.js (可选)
│   ├── service (可选)
│   └── middleware (可选)
│       └── mw.js
├── config
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    └── middleware
        └── mw.test.js
```

那区别在哪儿呢？

1. 插件没有独立的 router 和 controller。这主要出于几点考虑：

   - 路由一般和应用强绑定的，不具备通用性。
   - 一个应用可能依赖很多个插件，如果插件支持路由可能导致路由冲突。
   - 如果确实有统一路由的需求，可以考虑在插件里通过中间件来实现。

2. 插件需要在 `package.json` 中的 `eggPlugin` 节点指定插件特有的信息：

   - `{String} name` - 插件名（必须配置），具有唯一性，配置依赖关系时会指定依赖插件的 name。

   - `{Array} dependencies` - 当前插件强依赖的插件列表（如果依赖的插件没找到，应用启动失败）。

   - `{Array} optionalDependencies` - 当前插件的可选依赖插件列表（如果依赖的插件未开启，只会 warning，不会影响应用启动）。

   - `{Array} env` - 只有在指定运行环境才能开启，具体有哪些环境可以参考[运行环境](https://eggjs.org/zh-cn/basics/env.html)。此配置是可选的，一般情况下都不需要配置。

     ```
     {
       "name": "egg-rpc",
       "eggPlugin": {
         "name": "rpc",
         "dependencies": [ "registry" ],
         "optionalDependencies": [ "vip" ],
         "env": [ "local", "test", "unittest", "prod" ]
       }
     }
     ```

3. 插件没有 `plugin.js`：

   - `eggPlugin.dependencies` 只是用于声明依赖关系，而不是引入插件或开启插件。
   - 如果期望统一管理多个插件的开启和配置，可以在[上层框架](https://eggjs.org/zh-cn/advanced/framework.html)处理。

**插件能做什么？**

1. 扩展内置对象的接口:<https://eggjs.org/zh-cn/basics/extend.html>
2. 插入自定义中间件
3. 在应用启动时做一些初始化工作
4. 设置定时任务

将 [egg-mysql](https://github.com/eggjs/egg-mysql) 作为自定义的实例。

##### 插件的寻址规则

框架在加载插件的时候，遵循下面的寻址规则：

- 如果配置了 path，直接按照 path 加载。
- 没有 path 根据 package 名去查找，查找的顺序依次是：
  1. 应用根目录下的 `node_modules`
  2. 应用依赖框架路径下的 `node_modules`
  3. 当前路径下的 `node_modules` （主要是兼容单元测试场景）

#### 定时任务

虽然我们通过框架开发的 HTTP Server 是请求响应模型的，但是仍然还会有许多场景需要执行一些定时任务，例如：

1. 定时上报应用状态。
2. 定时从远程接口更新本地缓存。
3. 定时进行文件切割、临时文件删除。

##### 编写定时任务

所有的定时任务都统一存放在 `app/schedule` 目录下，每一个文件都是一个独立的定时任务，可以配置定时任务的属性和要执行的方法。

```js
//app/schedule/update_cache.js
const Subscription = require('egg').Subscription;

class UpdateCache extends Subscription {
  // 通过 schedule 属性来设置定时任务的执行间隔等配置
  static get schedule() {
    return {
      interval: '1m', // 1 分钟间隔
      type: 'all', // 指定所有的 worker 都需要执行
    };
  }

  // subscribe 是真正定时任务执行时被运行的函数
  async subscribe() {
    const res = await this.ctx.curl('http://www.api.com/cache', {
      dataType: 'json',
    });
    this.ctx.app.cache = res.data;
  }
}

module.exports = UpdateCache;
```

还可以简写为

```js
module.exports = {
  schedule: {
    interval: '1m', // 1 分钟间隔
    type: 'all', // 指定所有的 worker 都需要执行
  },
  async task(ctx) {
    const res = await ctx.curl('http://www.api.com/cache', {
      dataType: 'json',
    });
    ctx.app.cache = res.data;
  },
};
```

##### 定时方式

1. interval: 
   - 数字类型，单位为毫秒数，例如 `5000`。
   - 字符类型，会通过 [ms](https://github.com/zeit/ms) 转换成毫秒数，例如 `5s`。
2. cron: cron 表达式通过 [cron-parser](https://github.com/harrisiirak/cron-parser) 进行解析。

##### 类型

框架提供的定时任务默认支持两种类型，worker 和 all。worker 和 all 都支持上面的两种定时方式，只是当到执行时机时，会执行定时任务的 worker 不同：

- `worker` 类型：每台机器上只有一个 worker 会执行这个定时任务，每次执行定时任务的 worker 的选择是随机的。
- `all` 类型：每台机器上的每个 worker 都会执行这个定时任务。

##### 其他参数

除了刚才介绍到的几个参数之外，定时任务还支持这些参数：

- `cronOptions`: 配置 cron 的时区等，参见 [cron-parser](https://github.com/harrisiirak/cron-parser#options) 文档
- `immediate`：配置了该参数为 true 时，这个定时任务会在应用启动并 ready 后立刻执行一次这个定时任务。
- `disable`：配置该参数为 true 时，这个定时任务不会被启动。
- `env`：数组，仅在指定的环境下才启动该定时任务。

##### 执行日志

执行日志会输出到 `${appInfo.root}/logs/{app_name}/egg-schedule.log`，默认不会输出到控制台，可以通过 `config.customLogger.scheduleLogger` 来自定义。

```
// config/config.default.js
config.customLogger = {
  scheduleLogger: {
    // consoleLevel: 'NONE',
    // file: path.join(appInfo.root, 'logs', appInfo.name, 'egg-schedule.log'),
  },
};
```

##### 手动执行定时任务

我们可以通过 `app.runSchedule(schedulePath)` 来运行一个定时任务。`app.runSchedule` 接受一个定时任务文件路径（`app/schedule` 目录下的相对路径或者完整的绝对路径），执行对应的定时任务，返回一个 Promise。

```js
module.exports = app => {
  app.beforeStart(async () => {
    // 保证应用启动监听端口前数据已经准备好了
    // 后续数据的更新由定时任务自动触发
    await app.runSchedule('update_cache');
  });
};
```

#### 启动自定义

我们常常需要在应用启动期间进行一些初始化工作，等初始化完成后应用才可以启动成功，并开始对外提供服务。

框架提供了统一的入口文件（`app.js`）进行启动过程自定义，这个文件返回一个 Boot 类，我们可以通过定义 Boot 类中的生命周期方法来执行启动应用过程中的初始化工作。

框架提供了这些 生命周期函数供开发人员处理：

- 配置文件即将加载，这是最后动态修改配置的时机（`configWillLoad`）
- 配置文件加载完成（`configDidLoad`）
- 文件加载完成（`didLoad`）
- 插件启动完毕（`willReady`）
- worker 准备就绪（`didReady`）
- 应用启动完成（`serverDidReady`）
- 应用即将关闭（`beforeClose`）

我们可以在 `app.js` 中定义这个 Boot 类

```js
// app.js
class AppBootHook {
  constructor(app) {
    this.app = app;
  }

  configWillLoad() {
    // 此时 config 文件已经被读取并合并，但是还并未生效
    // 这是应用层修改配置的最后时机
    // 注意：此函数只支持同步调用

    // 例如：参数中的密码是加密的，在此处进行解密
    this.app.config.mysql.password = decrypt(this.app.config.mysql.password);
    // 例如：插入一个中间件到框架的 coreMiddleware 之间
    const statusIdx = this.app.config.coreMiddleware.indexOf('status');
    this.app.config.coreMiddleware.splice(statusIdx + 1, 0, 'limit');
  }

  async didLoad() {
    // 所有的配置已经加载完毕
    // 可以用来加载应用自定义的文件，启动自定义的服务

    // 例如：创建自定义应用的示例
    this.app.queue = new Queue(this.app.config.queue);
    await this.app.queue.init();

    // 例如：加载自定义的目录
    this.app.loader.loadToContext(path.join(__dirname, 'app/tasks'), 'tasks', {
      fieldClass: 'tasksClasses',
    });
  }

  async willReady() {
    // 所有的插件都已启动完毕，但是应用整体还未 ready
    // 可以做一些数据初始化等操作，这些操作成功才会启动应用

    // 例如：从数据库加载数据到内存缓存
    this.app.cacheData = await this.app.model.query(QUERY_CACHE_SQL);
  }

  async didReady() {
    // 应用已经启动完毕

    const ctx = await this.app.createAnonymousContext();
    await ctx.service.Biz.request();
  }

  async serverDidReady() {
    // http / https server 已启动，开始接受外部请求
    // 此时可以从 app.server 拿到 server 的实例

    this.app.server.on('timeout', socket => {
      // handle socket timeout
    });
  }
}

module.exports = AppBootHook;
```

**注意：在自定义生命周期函数中不建议做太耗时的操作，框架会有启动的超时检测。**

### 核心功能

#### 本地开发

 [egg-bin](https://github.com/eggjs/egg-bin) 模块（只在本地开发和单元测试使用）

调试参考:<https://eggjs.org/zh-cn/core/development.html#%E8%B0%83%E8%AF%95>

#### 单元测试

测试脚本文件统一按 `${filename}.test.js` 命名，必须以 `.test.js` 作为文件后缀,且放置在`test` 目录

统一使用 egg-bin 来运行测试脚本， 自动将内置的 [Mocha](https://mochajs.org/)、[co-mocha](https://github.com/blakeembrey/co-mocha)、[power-assert](https://github.com/power-assert-js/power-assert)，[nyc](https://github.com/istanbuljs/nyc) 等模块组合引入到测试脚本中

测试流程参考:<https://eggjs.org/zh-cn/core/unittest.html#%E5%87%86%E5%A4%87%E6%B5%8B%E8%AF%95>

#### 应用部署

框架内置了 [egg-cluster](https://github.com/eggjs/egg-cluster) 来启动 [Master 进程](https://eggjs.org/zh-cn/core/cluster-and-ipc.html#master)，Master 有足够的稳定性，不再需要使用 [pm2](https://github.com/Unitech/pm2) 等进程守护模块。

同时，框架也提供了 [egg-scripts](https://github.com/eggjs/egg-scripts) 来支持线上环境的运行和停止。

我们还需要对服务进行性能监控，内存泄露分析，故障排除等。

业界常用的有：

- [Node.js 性能平台（alinode）](https://www.aliyun.com/product/nodejs)
- [NSolid](https://nodesource.com/products/nsolid/)

#### 日志

框架内置了强大的企业级日志支持，由 [egg-logger](https://github.com/eggjs/egg-logger) 模块提供。

主要特性：

- 日志分级
- 统一错误日志，所有 logger 中使用 `.error()` 打印的 `ERROR` 级别日志都会打印到统一的错误日志文件中，便于追踪
- 启动日志和运行日志分离
- 自定义日志
- 多进程日志
- 自动切割日志
- 高性能

##### 日志路径

- 所有日志文件默认都放在 `${appInfo.root}/logs/${appInfo.name}` 路径下，例如 `/home/admin/logs/example-app`。
- 在本地开发环境 (env: local) 和单元测试环境 (env: unittest)，为了避免冲突以及集中管理，日志会打印在项目目录下的 logs 目录，例如 `/path/to/example-app/logs/example-app`。

如果想自定义日志路径：

```js
// config/config.${env}.js
exports.logger = {
  dir: '/path/to/your/custom/log/dir',
};
```

##### 日志分类

框架内置了几种日志，分别在不同的场景下使用：

- appLogger `${appInfo.name}-web.log`，例如 `example-app-web.log`，应用相关日志，供应用开发者使用的日志。我们在绝大数情况下都在使用它。
- coreLogger `egg-web.log` 框架内核、插件日志。
- errorLogger `common-error.log` 实际一般不会直接使用它，任何 logger 的 `.error()` 调用输出的日志都会重定向到这里，重点通过查看此日志定位异常。
- agentLogger `egg-agent.log` agent 进程日志，框架和使用到 agent 进程执行任务的插件会打印一些日志到这里。

如果想自定义以上日志文件名称，可以在 config 文件中覆盖默认值：

```js
// config/config.${env}.js
module.exports = appInfo => {
  return {
    logger: {
      appLogName: `${appInfo.name}-web.log`,
      coreLogName: 'egg-web.log',
      agentLogName: 'egg-agent.log',
      errorLogName: 'common-error.log',
    },
  };
};
```

##### 日志切割

框架对日志切割的支持由 [egg-logrotator](https://github.com/eggjs/egg-logrotator) 插件提供。默认日志切割方式按天切割

#### Cookie 与 Session

参考<https://eggjs.org/zh-cn/core/cookie-and-session.html>

#### **多进程模型和进程间通讯**

我们知道 JavaScript 代码是运行在单线程上的，换句话说一个 Node.js 进程只能运行在一个 CPU 上。那么如果用 Node.js 来做 Web Server，就无法享受到多核运算的好处。作为企业级的解决方案，我们要解决的一个问题就是:

> 如何榨干服务器资源，利用上多核 CPU 的并发优势？

而 Node.js 官方提供的解决方案是 [Cluster 模块](https://nodejs.org/api/cluster.html)，其中包含一段简介：

> 单个 Node.js 实例在单线程环境下运行。为了更好地利用多核环境，用户有时希望启动一批 Node.js 进程用于加载。
>
> 集群化模块使得你很方便地创建子进程，以便于在服务端口之间共享。

##### Cluster 是什么呢？

简单的说，

- 在服务器上同时启动多个进程。
- 每个进程里都跑的是同一份源代码（好比把以前一个进程的工作分给多个进程去做）。
- 更神奇的是，这些进程可以同时监听一个端口（具体原理推荐阅读 @DavidCai1993 这篇 [Cluster 实现原理](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)）。

其中：

- 负责启动其他进程的叫做 Master 进程，他好比是个『包工头』，不做具体的工作，只负责启动其他进程。
- 其他被启动的叫 Worker 进程，顾名思义就是干活的『工人』。它们接收请求，对外提供服务。
- Worker 进程的数量一般根据服务器的 CPU 核数来定，这样就可以完美利用多核资源。

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

作为企业级的解决方案，要考虑的东西还有很多。

- Worker 进程异常退出以后该如何处理？
- 多个 Worker 进程之间如何共享资源？
- 多个 Worker 进程之间如何调度？

Agent 好比是 Master 给其他 Worker 请的一个『秘书』，它不对外提供服务，只给 App Worker 打工，专门处理一些公共事务,进程模型：

```
                +--------+          +-------+
                | Master |<-------->| Agent |
                +--------+          +-------+
                ^   ^    ^
               /    |     \
             /      |       \
           /        |         \
         v          v          v
+----------+   +----------+   +----------+
| Worker 1 |   | Worker 2 |   | Worker 3 |
+----------+   +----------+   +----------+
```

框架的启动时序如下：

```js
+---------+           +---------+          +---------+
|  Master |           |  Agent  |          |  Worker |
+---------+           +----+----+          +----+----+
     |      fork agent     |                    |
     +-------------------->|                    |
     |      agent ready    |                    |
     |<--------------------+                    |
     |                     |     fork worker    |
     +----------------------------------------->|
     |     worker ready    |                    |
     |<-----------------------------------------+
     |      Egg ready      |                    |
     +-------------------->|                    |
     |      Egg ready      |                    |
     +----------------------------------------->|
```

1. Master 启动后先 fork Agent 进程
2. Agent 初始化成功后，通过 IPC 通道通知 Master
3. Master 再 fork 多个 App Worker
4. App Worker 初始化成功，通知 Master
5. 所有的进程初始化成功后，Master 通知 Agent 和 Worker 应用启动成功

另外，关于 Agent Worker 还有几点需要注意的是：

1. 由于 App Worker 依赖于 Agent，所以必须等 Agent 初始化完成后才能 fork App Worker
2. Agent 虽然是 App Worker 的『小秘』，但是业务相关的工作不应该放到 Agent 上去做，不然把她累垮了就不好了
3. 由于 Agent 的特殊定位，**我们应该保证它相对稳定**。当它发生未捕获异常，框架不会像 App Worker 一样让他退出重启，而是记录异常日志、报警等待人工处理
4. Agent 和普通 App Worker 挂载的 API 不完全一样，如何识别差异可查看[框架文档](https://eggjs.org/zh-cn/advanced/framework.html)

| 类型   | 进程数量            | 作用                         | 稳定性 | 是否运行业务代码 |
| ------ | ------------------- | ---------------------------- | ------ | ---------------- |
| Master | 1                   | 进程管理，进程间消息转发     | 非常高 | 否               |
| Agent  | 1                   | 后台运行工作（长连接客户端） | 高     | 少量             |
| Worker | 一般设置为 CPU 核数 | 执行业务代码                 | 一般   | 是               |

进程间通信:<https://eggjs.org/zh-cn/core/cluster-and-ipc.html#%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E8%AE%AFipc>

#### View 模板渲染

框架内置 [egg-view](https://github.com/eggjs/egg-view) 作为模板解决方案，并支持多模板渲染，每个模板引擎都以插件的方式引入，但保持渲染的 API 一致

1. 引入 view 插件

   ```bash
   $ npm i egg-view-nunjucks --save
   ```

2. 启用插件

   ```js
   // config/plugin.js
   exports.nunjucks = {
     enable: true,
     package: 'egg-view-nunjucks',
   };
   ```

3. 配置插件, `config.view` 通用配置

   - root {String} : 模板文件的根目录，为绝对路径，默认为 `${baseDir}/app/view`。支持配置多个目录，以 `,` 分割，会从多个目录查找文件。
   - cache {Boolean}:模板路径缓存，默认开启。框架会根据 root 配置的目录依次查找，如果匹配则会缓存文件路径，下次渲染相同路径时不会重新查找。
   - mapping 和 defaultViewEngine: 每个模板在注册时都会指定一个模板名（viewEngineName），在使用时需要根据后缀来匹配模板名，比如指定 `.nj` 后缀的文件使用 Nunjucks 进行渲染。

##### 渲染页面

框架在 Context 上提供了 3 个接口，返回值均为 Promise:

- `render(name, locals)` 渲染模板文件, 并赋值给 ctx.body
- `renderView(name, locals)` 渲染模板文件, 仅返回不赋值
- `renderString(tpl, locals)` 渲染模板字符串, 仅返回不赋值

##### Locals

在渲染页面的过程中，我们通常需要一个变量来收集需要传递给模板的变量，在框架里面，我们提供了 `app.locals` 和 `ctx.locals`。

- `app.locals` 为全局的，一般在 `app.js` 里面配置全局变量。
- `ctx.locals` 为单次请求的，会合并 `app.locals`。
- 可以直接赋值对象，框架在对应的 setter 里面会自动 merge。

但在实际业务开发中，controller 中一般不会直接使用这 2 个对象，直接使用 `ctx.render(name, data)` 即可：

- 框架会自动把 `data` 合并到 `ctx.locals`。
- 框架会自动注入 `ctx`, `request`, `helper` 方便使用。

#### 异常处理

**为了保证异常可追踪，必须保证所有抛出的异常都是 Error 类型，因为只有 Error 类型才会带上堆栈信息，定位到问题。**

框架通过 [onerror](https://github.com/eggjs/egg-onerror) 插件提供了统一的错误处理机制。对一个请求的所有处理方法（Middleware、Controller、Service）中抛出的任何异常都会被它捕获，并自动根据请求想要获取的类型返回不同类型的错误（基于 [Content Negotiation](https://tools.ietf.org/html/rfc7231#section-5.3.2)）。

| 请求需求的格式 | 环境             | errorPageUrl 是否配置 | 返回内容                                             |
| -------------- | ---------------- | --------------------- | ---------------------------------------------------- |
| HTML & TEXT    | local & unittest | -                     | onerror 自带的错误页面，展示详细的错误信息           |
| HTML & TEXT    | 其他             | 是                    | 重定向到 errorPageUrl                                |
| HTML & TEXT    | 其他             | 否                    | onerror 自带的没有错误信息的简单错误页（不推荐）     |
| JSON & JSONP   | local & unittest | -                     | JSON 对象或对应的 JSONP 格式响应，带详细的错误信息   |
| JSON & JSONP   | 其他             | -                     | JSON 对象或对应的 JSONP 格式响应，不带详细的错误信息 |

##### 自定义统一异常处理

框架自带的 onerror 插件支持自定义配置错误处理方法，可以覆盖默认的错误处理方法。

```js
// config/config.default.js
module.exports = {
  onerror: {
    all(err, ctx) {
      // 在此处定义针对所有响应类型的错误处理方法
      // 注意，定义了 config.all 之后，其他错误处理方法不会再生效
      ctx.body = 'error';
      ctx.status = 500;
    },
    html(err, ctx) {
      // html hander
      ctx.body = '<h3>error</h3>';
      ctx.status = 500;
    },
    json(err, ctx) {
      // json hander
      ctx.body = { message: 'error' };
      ctx.status = 500;
    },
    jsonp(err, ctx) {
      // 一般来说，不需要特殊针对 jsonp 进行错误定义，jsonp 的错误处理会自动调用 json 错误处理，并包装成 jsonp 的响应格式
    },
  },
};
```

#### **安全(很重要)**

参考:<https://eggjs.org/zh-cn/core/security.html>

#### 国际化（I18n）

由 [egg-i18n](https://github.com/eggjs/egg-i18n) 插件提供。

默认语言是 `en-US`。假设我们想修改默认语言为简体中文：

```js
// config/config.default.js
exports.i18n = {
  defaultLocale: 'zh-CN',
};
```

##### 切换语言

我们可以通过下面几种方式修改应用的当前语言（修改后会记录到 `locale` 这个 Cookie），下次请求直接用设定好的语言。

优先级从高到低：

1. query: `/?locale=en-US`
2. cookie: `locale=zh-TW`
3. header: `Accept-Language: zh-CN,zh;q=0.5`

##### 编写 I18n 多语言文件

多种语言的配置是独立的，统一存放在 `config/locale/*.js` 下。

```
- config/locale/
  - en-US.js
  - zh-CN.js
  - zh-TW.js
```

不仅对于应用目录生效，在框架，插件的 `config/locale` 目录下同样生效。

##### 获取多语言文本

我们可以使用 `__` (Alias: `gettext`) 函数获取 locale 文件夹下面的多语言文本。

**注意: `__` 是两个下划线**

以上面配置过的多语言为例：

```
ctx.__('Email')
// zh-CN => 邮箱
// en-US => Email
```

如果文本中含有 `%s`，`%j` 等 format 函数，可以按照 [`util.format()`](https://nodejs.org/api/util.html#util_util_format_format_args) 类似的方式调用：

```js
// config/locale/zh-CN.js
module.exports = {
  'Welcome back, %s!': '欢迎回来，%s!',
};

ctx.__('Welcome back, %s!', 'Shawn');
// zh-CN => 欢迎回来，Shawn!
// en-US => Welcome back, Shawn!
```

同时支持数组下标占位符方式，例如：

```js
// config/locale/zh-CN.js
module.exports = {
  'Hello {0}! My name is {1}.': '你好 {0}! 我的名字叫 {1}。',
};

ctx.__('Hello {0}! My name is {1}.', ['foo', 'bar'])
// zh-CN => 你好 foo！我的名字叫 bar。
// en-US => Hello foo! My name is bar.
```

### 插件教程

1. [passport鉴权](https://eggjs.org/zh-cn/tutorials/passport.html)
2. [typescript](https://eggjs.org/zh-cn/tutorials/typescript.html)
3. [前置代理模式](https://eggjs.org/zh-cn/tutorials/proxy.html)



### 参考

1. [egg.js 官网](https://eggjs.org/)
2. [awesome-egg](https://github.com/eggjs/awesome-egg)
3. [eggjs 专栏](https://zhuanlan.zhihu.com/eggjs)
4. [官方示例](https://github.com/eggjs/examples)