---
title: OAuth 2.0 完成分布式认证系统
date: 2019-11-07 17:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

#### 分布式认证

随着软件环境和需求的变化 ,软件的架构由单体结构演变为分布式架构,具有分布式架构的系统叫分布式系统,分布式系统的运行通常依赖网络,它将单体结构的系统分为若干服务,服务之间通过网络交互来完成用户的业务处理,当前流行的微服务架构就是分布式系统架构,如下图:

<!--more-->

![QqvmAx.png](https://s2.ax1x.com/2019/12/19/QqvmAx.png)



分布式系统具体如下基本特点:

1. 分布性:每个部分都可以独立部署,服务之间交互通过网络进行通信,比如:订单服务、商品服务。
2. 伸缩性:每个部分都可以集群方式部署,并可针对部分结点进行硬件及软件扩容,具有一定的伸缩能力。
3. 共享性:每个部分都可以作为共享资源对外提供服务,多个部分可能有操作共享资源的情况。
4. 开放性:每个部分根据需求都可以对外发布共享资源的访问接口,并可允许第三方系统访问。

**分布式认证需求**

分布式系统的每个服务都会有认证、授权的需求,如果每个服务都实现一套认证授权逻辑会非常冗余,考虑分布式系统共享性的特点,需要由独立的认证服务处理系统认证授权的请求;考虑分布式系统开放性的特点,不仅对系统内部服务提供认证,对第三方系统也要提供认证。分布式认证的需求总结如下:

1. 统一认证授权

   提供独立的认证服务,统一处理认证授权。
   无论是不同类型的用户,还是不同种类的客户端(web端,H5、APP),均采用一致的认证、权限、会话机制,实现统一认证授权。

   要实现统一则认证方式必须可扩展,支持各种认证需求,比如:用户名密码认证、短信验证码、二维码、人脸识别等认证方式,并可以非常灵活的切换。

2. 应用接入认证

   应提供扩展和开放能力,提供安全的系统对接机制,并可开放部分API给接入第三方使用,一方应用(内部系统服务)和三方应用(第三方应用)均采用统一机制接入。

##### 分布式认证方案

1. 基于session的认证方式

   在分布式的环境下,基于session的认证会出现一个问题,每个应用服务都需要在session中存储用户身份信息,通过负载均衡将本地的请求分配到另一个应用服务需要将session信息带过去,否则会重新认证。

   ![QqvNUP.png](https://s2.ax1x.com/2019/12/19/QqvNUP.png)

   这个时候,通常的做法有下面几种:

   - Session复制:多台应用服务器之间同步session,使session保持一致,对外透明。
   - Session黏贴:当用户访问集群中某台服务器后,强制指定后续所有请求均落到此机器上。
   - Session集中存储:将Session存入分布式缓存中,所有服务器应用实例统一从分布式缓存中存取Session。

   总体来讲,基于session认证的认证方式,可以更好的在服务端对会话进行控制,且安全性较高。但是,session机制方式基于cookie,在复杂多样的移动客户端上不能有效的使用,并且无法跨域,另外随着系统的扩展需提高session的复制、黏贴及存储的容错性。

2. 基于token的认证方式

   基于token的认证方式,服务端不用存储认证数据,易维护扩展性强, 客户端可以把token 存在任意地方,并且可以实现web和app统一认证机制。其缺点也很明显,token由于自包含信息,因此一般数据量较大,而且每次请求都需要传递,因此比较占带宽。另外,token的签名验签操作也会给cpu带来额外的处理负担。

   ![QLSorQ.png](https://s2.ax1x.com/2019/12/19/QLSorQ.png)

基于token的认证方式,它的优点是:

1. 适合统一认证的机制,客户端、一方应用、三方应用都遵循一致的认证机制。
2. token认证方式对第三方应用接入更适合,因为它更开放,可使用当前有流行的开放协议Oauth2.0、JWT等。
3. 一般情况服务端无需存储会话信息,减轻了服务端的压力。

分布式系统认证技术方案见下图:

![QLp1it.png](https://s2.ax1x.com/2019/12/19/QLp1it.png)

流程描述:

1. 用户通过接入方(应用)登录,接入方采取OAuth2.0方式在统一认证服务(UAA)中认证。
2. 认证服务(UAA)调用验证该用户的身份是否合法,并获取用户权限信息。
3. 认证服务(UAA)获取接入方权限信息,并验证接入方是否合法。
4. 若登录用户以及接入方都合法,认证服务生成jwt令牌返回给接入方,其中jwt中包含了用户权限及接入方权限。
5. 后续,接入方携带jwt令牌对API网关内的微服务资源进行访问。
6. API网关对令牌解析、并验证接入方的权限是否能够访问本次请求的微服务。
7. 如果接入方的权限没问题,API网关将原请求header中附加解析后的明文Token,并将请求转发至微服务。
8. 微服务收到请求,明文token中包含登录用户的身份和权限信息。因此后续微服务自己可以干两件事:1,用户授权拦截(看当前用户是否有权访问该资源)2,将用户信息存储进当前线程上下文(有利于后续业务逻辑随时获取当前用户信息)

流程所涉及到UAA服务、API网关这三个组件职责如下:

1. 统一认证服务(UAA)

   它承载了OAuth2.0接入方认证、登入用户的认证、授权以及生成令牌的职责,完成实际的用户认证、授权功能。

2. API网关

   作为系统的唯一入口,API网关为接入方提供定制的API集合,它可能还具有其它职责,如身份验证、监控、负载均衡、缓存等。API网关方式的核心要点是,所有的接入方和消费端都通过统一的网关接入微服务,在网关层处理所有的非业务功能。

### OAuth2.0

OAuth是一个关于授权的开放网络标准，OAuth2是其2.0版本。

它规定了四种操作流程（授权模式）来确保安全

应用场景有第三方应用的接入、微服务鉴权互信、接入第三方平台、第一方密码登录等

> Java王国中Spring Security也对OAuth2标准进行了实现。

<!--more-->

用于REST/APIS的代理授权框架（delegatedauthorizetion framework）,基于令牌token的授权，在无需暴露用户密码的情况下，使应用能够对用户数据有效访问权限，充分解耦认证和授权，实际上是标准的安全架构，支持多种应用场景，服务器端WebApp,浏览器单页面SPA，无线原生App,服务器与服务器之间访问。像仆从钥匙，给应用授权优先的访问权限，代表用户访问用户数据。OAoth是系统之间代理授权协议

**优点**

1. 易实现
2. 安全，服务端不接触用户密码，服务单更容易集中保护。
3. 广泛传播被持续采用
4. 短寿命和封装的token
5. 资源服务器和授权服务器解耦
6. 集中授权简化客户端
7. HTTP/JSON友好易于请求和传递token
8. 考虑多种客户端架构场景
9. 客户可以具有不同的信任级别

**缺点**

1. 协议框架太宽泛，造成各种实现的兼容性和互操作性
2. 与OAuth1.0不兼容
3. OAuth 2.0 不是一个认证协议，OAuth2.0本身并不能告诉你任何用户信息

**应用场景,解决方案**

1. 开放间系统授权

   - 社交联合登陆

   - 开放API平台

2. 现代微服务安全

   - .单页面浏览器APP
   - 无线原生APP
   - 服务端WebApp
   - 微服务和原生API调用

3. 企业内部认证授权(IAM,SSO)

**架构角色**

1. 授权服务 Authorization Service
    客户应用成功认证并获得授权之后，向客户应用颁发访问令牌。

2. 资源服务 Resource Service
    一个web服务或者web应用，保存用户受保护的数据

3. 客户端应用 Client Application
    通常是一个浏览器或者手机app，它需要用户受保护的数据

4. 资源拥有者 owner
    数据拥有者，想把数据分享给他人使用

客户端应用需要访问资源服务,但是没有认证,此时客户端去授权服务获取认证令牌,拿到令牌后交给资源服务器,资源服务器拿到令牌后也去授权服务比较一次,如果是对的,就认证通过.

**OAuth2.0语术概念**

1. 客户凭证 Client Credentials
   客户的clientId和密码用户认证客户

2. 令牌 Tokens
   授权服务器在接收到客户请求后颁发的资源服务器

   **令牌类型**

   - 授权码 (Authorization Code Token)  仅用于授权码类型，用于交换获取访问令牌和刷新令牌
   - 刷新令牌 (Refresh Token) 用于去授权服务器获取一个新的token
   - 访问令牌 (Access Token) 代表用户直接访问受保护的资源服务器
   - Bearer Token 不管谁拿到都可以访问资源
   - Proof of Prosession Token   可以校验Client是否对Token有明确的权限

3. 作用域
   客户请求访问令牌时，有资源拥有者额外指定的细分权限

**名词定义**

1. Third-party application：第三方应用程序

2. HTTP service：HTTP服务提供商，简称"服务提供商"
3. Resource Owner：资源所有者
4. User Agent：用户代理
5. Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。
6. Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

知道了上面这些名词，就不难理解，OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

**OAuth的思路**

OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

#### 运行流程

OAuth 2.0的运行流程如下图，摘自RFC 6749。

![QLFlgx.png](https://s2.ax1x.com/2019/12/19/QLFlgx.png)

1. 用户打开客户端以后，客户端要求用户给予授权。
2. 用户同意给予客户端授权。
3. 客户端使用上一步获得的授权，向认证服务器申请令牌。
4. 认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
5. 客户端使用令牌，向资源服务器申请获取资源。
6. 资源服务器确认令牌无误，同意向客户端开放资源。

不难看出来，上面六个步骤之中，B是关键，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。

**示例**

下边分析一个 Oauth2认证的例子,通过例子去理解OAuth2.0协议的认证流程,本例子是黑马程序员网站使用微信认证的过程,这个过程的简要描述如下:

用户借助微信认证登录黑马程序员网站,用户就不用单独在黑马程序员注册用户,怎么样算认证成功吗?黑马程序员网站需要成功从微信获取用户的身份信息则认为用户认证成功,那如何从微信获取用户的身份信息?用户信息的拥有者是用户本人,微信需要经过用户的同意方可为黑马程序员网站生成令牌,黑马程序员网站拿此令牌方可从微信获取用户的信息。

1. 客户端请求第三方授权

   用户进入黑马程序的登录页面,点击微信的图标以微信账号登录系统,用户是自己在微信里信息的资源拥有者。

2. 资源拥有者同意给客户端授权

   资源拥有者扫描二维码表示资源拥有者同意给客户端授权,微信会对资源拥有者的身份进行验证, 验证通过后,微信会询问用户是否给授权黑马程序员访问自己的微信数据,用户点击“确认登录”表示同意授权,微信认证服务器会颁发一个授权码,并重定向到黑马程序员的网站。

3. 客户端获取到授权码,请求认证服务器申请令牌
   此过程用户看不到,客户端应用程序请求认证服务器,请求携带授权码。

4. 认证服务器向客户端响应令牌
   微信认证服务器验证了客户端请求的授权码,如果合法则给客户端颁发令牌,令牌是客户端访问资源的通行证。此交互过程用户看不到,当客户端拿到令牌后,用户在黑马程序员看到已经登录成功。

5. 客户端请求资源服务器的资源
   客户端携带令牌访问资源服务器的资源。
   黑马程序员网站携带令牌请求访问微信服务器获取用户的基本信息。

6. 资源服务器返回受保护资源
   资源服务器校验令牌的合法性,如果合法则向用户响应资源信息内容

以上认证授权详细的执行流程如下:

![QL9Rc8.png](https://s2.ax1x.com/2019/12/19/QL9Rc8.png)

下面一一讲解客户端获取授权的四种模式。

#### 客户端的授权模式

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

**授权码模式**

授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![QLFTqU.png](https://s2.ax1x.com/2019/12/19/QLFTqU.png)

它的步骤如下：

1. 用户访问客户端，后者将前者导向认证服务器。
2. 用户选择是否给予客户端授权。
3. 假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
4. 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
5. 认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

下面是上面这些步骤所需要的参数。

A步骤中，客户端申请认证的URI，包含以下参数：

- response_type：表示授权类型，必选项，此处的值固定为"code"
- client_id：表示客户端的ID，必选项
- redirect_uri：表示重定向URI，可选项
- scope：表示申请的权限范围，可选项
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子。

```http
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

C步骤中，服务器回应客户端的URI，包含以下参数：

- code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子。

```http
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
          &state=xyz
```

D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

- grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
- code：表示上一步获得的授权码，必选项。
- redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
- client_id：表示客户端ID，必选项。

下面是一个例子。

```http
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

E步骤中，认证服务器发送的HTTP回复，包含以下参数：

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

下面是一个例子。

```http
     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }
```

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。此外，HTTP头信息中明确指定不得缓存。

特点:

- 通过前端渠道客户获取授权码
- 通过后端渠道,客户使用authorization code 交换access token 或refresh token
- 假定资源拥有者和客户在不同的设备上
- 最安全的流程,因为令牌不会传递经过User-Agent

总结: **授权码模式本质上是客户端通过用户名密码发起获取授权码请求,服务端根据回调地址返回授权码,客户端根据授权码访问资源服务器,资源服务器根据授权码拿到授权服务器给的access token返回给客户端,客户端就可以带着这个access token访问资源服务器上的有效资源.**

**简化模式**

简化模式（implicit grant type）不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。

![QLkGzq.png](https://s2.ax1x.com/2019/12/19/QLkGzq.png)

它的步骤如下：

1. 客户端将用户导向认证服务器。
2. 用户决定是否给于客户端授权。
3. 假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。
4. 浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。
5. 资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。
6. 浏览器执行上一步获得的脚本，提取出令牌。
7. 浏览器将令牌发给客户端。

下面是上面这些步骤所需要的参数。

A步骤中，客户端发出的HTTP请求，包含以下参数：

- response_type：表示授权类型，此处的值固定为"token"，必选项。
- client_id：表示客户端的ID，必选项。
- redirect_uri：表示重定向的URI，可选项。
- scope：表示权限范围，可选项。
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子。

```http
GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com
```

C步骤中，认证服务器回应客户端的URI，包含以下参数：

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子。

```http
HTTP/1.1 302 Found
     Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
               &state=xyz&token_type=example&expires_in=3600

```

在上面的例子中，认证服务器用HTTP头信息的Location栏，指定浏览器重定向的网址。注意，在这个网址的Hash部分包含了令牌。

根据上面的D步骤，下一步浏览器会访问Location指定的网址，但是Hash部分不会发送。接下来的E步骤，服务提供商的资源服务器发送过来的代码，会提取出Hash中的令牌。

**密码模式**

密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![QLk6Qx.png](https://s2.ax1x.com/2019/12/19/QLk6Qx.png)

它的步骤如下：

1. 用户向客户端提供用户名和密码。
2. 客户端将用户名和密码发给认证服务器，向后者请求令牌。
3. 认证服务器确认无误后，向客户端提供访问令牌。

B步骤中，客户端发出的HTTP请求，包含以下参数：

- grant_type：表示授权类型，此处的值固定为"password"，必选项。
- username：表示用户名，必选项。
- password：表示用户的密码，必选项。
- scope：表示权限范围，可选项。

下面是一个例子。

```http
POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=password&username=johndoe&password=A3ddj3w
```

C步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。

```http
HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }
```

整个过程中，客户端不得保存用户的密码。

**客户端模式**

客户端模式（Client Credentials Grant）指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。

![QLkxYj.png](https://s2.ax1x.com/2019/12/19/QLkxYj.png)

它的步骤如下：

1. 客户端向认证服务器进行身份认证，并要求一个访问令牌。
2. 认证服务器确认无误后，向客户端提供访问令牌。

A步骤中，客户端发出的HTTP请求，包含以下参数：

- grant*type：表示授权类型，此处的值固定为"client*credentials"，必选项。
- scope：表示权限范围，可选项。

```http
POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=client_credentials

```

认证服务器必须以某种方式，验证客户端身份。

B步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。

```http
HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "example_parameter":"example_value"
     }

```

##### 更新令牌

如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。

客户端发出更新令牌的HTTP请求，包含以下参数：

- grant_type：表示使用的授权模式，此处的值固定为"refresh_token"，必选项。
- refresh_token：表示早前收到的更新令牌，必选项。
- scope：表示申请的授权范围，不可以超出上一次申请的范围，如果省略该参数，则表示与上一次一致。

下面是一个例子。

```http
POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA

```

例子：<https://github.com/hanyunpeng0521/learn-sercurity/tree/master/distributed-security/distributed-security-uaa>

#### JWT

当资源服务和授权服务不在一起时资源服务使用RemoteTokenServices 远程请求授权服务验证token,如果访问量较大将会影响系统的性能 。

解决上边问题:

令牌采用JWT格式即可解决上边的问题,用户认证通过会得到一个JWT令牌,JWT令牌中已经包括了用户相关的信息,客户端只需要携带JWT访问资源服务,资源服务根据事先约定的算法自行完成令牌校验,无需每次都请求认证服务完成授权。

**什么是JWT?**

JSON Web Token(JWT)是一个开放的行业标准(RFC 7519),它定义了一种简介的、自包含的协议格式,用于在通信双方传递json对象,传递的信息经过数字签名可以被验证和信任。JWT可以使用HMAC算法或使用RSA的公钥/私钥对来签名,防止被篡改。

官网:https://jwt.io/
标准: https://tools.ietf.org/html/rfc7519

**JWT令牌的优点:**

1. jwt基于json,非常方便解析。
2. 可以在令牌中自定义丰富的内容,易扩展。
3. 通过非对称加密算法及数字签名技术,JWT防止篡改,安全性高。
4. 资源服务使用JWT可不依赖认证服务即可完成授权

**缺点:**

1. JWT令牌较长,占存储空间比较大。
2. JWT的最大缺点是服务器不保存会话状态，所以在使用期间不可能取消令牌或更改令牌的权限。也就是说，一旦JWT签发，在有效期内将会一直有效。
3. JWT本身包含认证信息，因此一旦信息泄露，任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不宜设置太长。对于某些重要操作，用户在使用时应该每次都进行进行身份验证。
4. 为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传输。

**JWT令牌结构**

JWT令牌由三部分组成,每部分中间使用点(.)分隔,比如:xxxxx.yyyyy.zzzzz

1. Header
   头部包括令牌的类型(即JWT)及使用的哈希算法(如HMAC SHA256或RSA)

   一个例子如下:

   下边是Header部分的内容

   ```
   {
   "alg": "HS256",
   "typ": "JWT"
   }
   ```

   将上边的内容使用Base64Url编码,得到一个字符串就是JWT令牌的第一部分。

2. Payload

   第二部分是负载,内容也是一个json对象,它是存放有效信息的地方,它可以存放jwt提供的现成字段,比如:iss(签发者),exp(过期时间戳), sub(面向的用户)等,也可自定义字段。

   此部分不建议存放敏感信息,因为此部分可以解码还原原始内容。

   最后将第二部分负载使用Base64Url编码,得到一个字符串就是JWT令牌的第二部分。

   一个例子:

   ```
   {
   "sub": "1234567890",
   "name": "456",
   "admin": true
   }
   ```

3. Signature

   第三部分是签名,此部分用于防止jwt内容被篡改。

   这个部分使用base64url将前两部分进行编码,编码后使用点(.)连接组成字符串,最后使用header中声明签名算法进行签名。

   ```
   HMACSHA256(
   base64UrlEncode(header) + "." +
   base64UrlEncode(payload),
   secret)
   ```

   base64UrlEncode(header):jwt令牌的第一部分。base64UrlEncode(payload):jwt令牌的第二部分。secret:签名所使用的密钥。

**配置JWT令牌服务**

在uaa中配置jwt令牌服务,即可实现生成jwt格式的令牌。

1. TokenConfig

   ```java
   @Configuration
   public class TokenConfig {
   private String SIGNING_KEY = "uaa123";
   @Bean
   public TokenStore tokenStore() {
   return new JwtTokenStore(accessTokenConverter());
   }
   @Bean
   public JwtAccessTokenConverter accessTokenConverter() {
   JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
   converter.setSigningKey(SIGNING_KEY); //对称秘钥,资源服务器使用该秘钥来验证
   return converter;
   }
   }
   ```

2. 定义JWT令牌服务

   ```java
   @Autowired
   private JwtAccessTokenConverter accessTokenConverter;
   @Bean
   public AuthorizationServerTokenServices tokenService() {
       DefaultTokenServices service=new DefaultTokenServices();
       service.setClientDetailsService(clientDetailsService);
       service.setSupportRefreshToken(true);
       service.setTokenStore(tokenStore);
       TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
       tokenEnhancerChain.setTokenEnhancers(Arrays.asList(accessTokenConverter));
       service.setTokenEnhancer(tokenEnhancerChain);
       service.setAccessTokenValiditySeconds(7200); // 令牌默认有效期2小时
       service.setRefreshTokenValiditySeconds(259200); // 刷新令牌默认有效期3天
       return service;
   }
   ```

**校验jwt令牌**

资源服务需要和授权服务拥有一致的签字、令牌服务等:

1. 将授权服务中的TokenConfig类拷贝到order资源服务中
2. 屏蔽资源 服务原来的令牌服务类

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResouceServerConfig extends ResourceServerConfigurerAdapter {
    public static final String RESOURCE_ID = "res1";


    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/**").access("#oauth2.hasScope('ROLE_ADMIN')")
                .and().csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }

//    @Override
//    public void configure(ResourceServerSecurityConfigurer resources) {
//        resources.resourceId(RESOURCE_ID)
//                .tokenServices(tokenService())
//                .stateless(true);
//    }

//    // 资源服务令牌解析服务
//    @Bean
//    public ResourceServerTokenServices tokenService() {
////使用远程服务请求授权服务器校验token,必须指定校验token 的url、client_id,client_secret
//        RemoteTokenServices service = new RemoteTokenServices();
//        service.setCheckTokenEndpointUrl("http://localhost:53020/uaa/oauth/check_token");
//        service.setClientId("c1");
//        service.setClientSecret("secret");
//        return service;
//    }

    @Autowired
    private TokenStore tokenStore;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.resourceId(RESOURCE_ID)
                .tokenStore(tokenStore)
                .stateless(true);
    }

}
```

例子：<https://github.com/hanyunpeng0521/learn-sercurity/tree/master/distributed-security>

#### Spring Security 实现分布式系统授权

![QXeHoV.png](https://s2.ax1x.com/2019/12/20/QXeHoV.png)

1. UAA认证服务负责认证授权。
2. 所有请求经过 网关到达微服务
3. 网关负责鉴权客户端以及请求转发
4. 网关将token解析后传给微服务,微服务进行授权。

**注册中心**

所有微服务的请求都经过网关,网关从注册中心读取微服务的地址,将请求转发至微服务。注册中心采用Eureka

**网关**

网关整合 OAuth2.0 有两种思路,一种是认证服务器生成jwt令牌, 所有请求统一在网关层验证,判断权限等操作;另一种是由各资源服务处理,网关只做请求转发。

我们选用第一种。我们把API网关作为OAuth2.0的资源服务器角色,实现接入客户端权限拦截、令牌解析并转发当前登录用户信息(jsonToken)给微服务,这样下游微服务就不需要关心令牌格式解析以及OAuth2.0相关机制了。

API网关在认证授权体系里主要负责两件事:

1. 作为OAuth2.0的资源服务器角色,实现接入方权限拦截。
2. 令牌解析并转发当前登录用户信息(明文token)给微服务

微服务拿到明文token(明文token中包含登录用户的身份和权限信息)后也需要做两件事:

1. 用户授权拦截(看当前用户是否有权访问该资源)
2. 将用户信息存储进当前线程上下文(有利于后续业务逻辑随时获取当前用户信息)

例子：<https://github.com/hanyunpeng0521/learn-sercurity/tree/master/distributed-security>



黑马程序员，文档和官方文档，链接: https://pan.baidu.com/s/1TFDT6UL0BtJAYBslq-tRJQ 提取码: k7d3 

### 参考

1. [OAuth2.0深入理解](https://www.jianshu.com/p/7d253afb617f)