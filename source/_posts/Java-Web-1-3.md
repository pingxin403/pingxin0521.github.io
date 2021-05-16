---
title: Cookie、Session、token
date: 2019-05-17 08:18:59
tags:
 - Java
 - Web
categories:
 - Java
 - Web
---

会话（Session）跟踪是Web程序中常用的技术，用来**跟踪用户的整个会话**。常用的会话跟踪技术是Cookie与Session。**Cookie通过在客户端记录信息确定用户身份**，**Session通过在服务器端记录信息确定用户身份**。

<!--more-->

**http是一个无状态协议**

什么是无状态呢？就是说这一次请求和上一次请求是没有任何关系的，互不认识的，没有关联的。这种无状态的的好处是快速。坏处是假如我们想要把`www.zhihu.com/login.html`和`www.zhihu.com/index.html`关联起来，必须使用某些手段和工具

**cookie和session**

由于http的无状态性，为了使某个域名下的所有网页能够共享某些数据，session和cookie出现了。客户端访问服务器的流程如下

- 首先，客户端会发送一个http请求到服务器端。

- 服务器端接受客户端请求后，建立一个session，并发送一个http响应到客户端，这个响应头，其中就包含Set-Cookie头部。该头部包含了sessionId。Set-Cookie格式如下:
  `Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]`

- 在客户端发起的第二次请求，假如服务器给了set-Cookie，浏览器会自动在请求头中添加cookie

- 服务器接收请求，分解cookie，验证信息，核对成功后返回response给客户端


**注意**

- cookie只是实现session的其中一种方案。虽然是最常用的，但并不是唯一的方法。禁用cookie后还有其他方法存储，比如放在url中
- 现在大多都是Session + Cookie，但是只用session不用cookie，或是只用cookie，不用session在理论上都可以保持会话状态。可是实际中因为多种原因，一般不会单独使用
- 用session只需要在客户端保存一个id，实际上大量数据都是保存在服务端。如果全部用cookie，数据量大的时候客户端是没有那么多空间的。
- 如果只用cookie不用session，那么账户信息全部保存在客户端，一旦被劫持，全部信息都会泄露。并且客户端数据量变大，网络传输的数据量也会变大

**小结**

**简而言之, session 有如用户信息档案表, 里面包含了用户的认证信息和登录状态等信息. 而 cookie 就是用户通行证**

#### cookies

HTTP cookies，通常称之为“cookie”，已经存在很长时间了，但是仍然没有被充分理解。首要问题是存在许多误解，认为 cookie 是后门程序或病毒，却忽视了其工作原理。第二个问题是，对于 cookie 的操作缺少统一的接口。尽管存在这些问题，cookie 仍旧在 Web 开发中扮演者重要的角色，以至于如果没有出现相应的代替品就消失的话，我们许多喜欢的 Web 应用将变的不可用。

简单地说，cookie 就是浏览器储存在用户电脑上的一小段文本文件。cookie 是纯文本格式，不包含任何可执行的代码。一个 Web 页面或服务器告知浏览器按照一定规范来储存这些信息，并在随后的请求中将这些信息发送至服务器，Web 服务器就可以使用这些信息来识别不同的用户。大多数需要登录的网站在用户验证成功之后都会设置一个 cookie，只要这个 cookie 存在并可以，用户就可以自由浏览这个网站的任意页面。再次说明，cookie 只包含数据，就其本身而言并不有害。

1. 创建 cookie

   Web 服务器通过发送一个称为 `Set-Cookie` 的 HTTP 消息头来创建一个 cookie，`Set-Cookie` 消息头是一个字符串，其格式如下（中括号中的部分是可选的）：

   ```
   Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]
   ```

   消息头的第一部分，value 部分，通常是一个 `name=value` 格式的字符串。事实上，这种格式是原始规范中指定的格式，但是浏览器并不会对 cookie 值按照此格式来验证。实际上，你可以指定一个不含等号的字符串，它同样会被存储。然而，最常用的使用方式是按照 `name=value` 格式来指定 cookie 的值（大多数接口只支持该格式）。

   当存在一个 cookie，并允许设置可选项，该 cookie 的值会在随后的每次请求中被发送至服务器，cookie 的值被存储在名为 Cookie 的 HTTP 消息头中，并且只包含了 cookie 的值，忽略全部设置选项。例如：

   ```
   Cookie: value
   ```

   通过 `Set-Cookie` 指定的可选项只会在浏览器端使用，而不会被发送至服务器端。发送至服务器的 cookie 的值与通过 `Set-Cookie` 指定的值完全一样，不会有进一步的解析或转码操作。如果请求中包含多个 cookie，它们将会被分号和空格分开，例如：

   ```
   Cookie: value1; value2; name1=value1
   ```

   服务器端框架通常包含解析 cookie 的方法，可以通过编程的方式获取 cookie 的值。

2. cookie 编码

   对于 cookie 的值进行编码一直都存在一些困惑。普遍认为 cookie 的值必须经过 URL 编码，但其实这是一个谬论，尽管通常都这么做。原始规范中明确指出只有三个字符必须进行编码：**分号、逗号和空格**，规范中还提到可以进行 URL 编码，但并不是必须，在 RFC 中没有提及任何编码。然而，几乎所有的实现都对 cookie 的值进行了一系列的 URL 编码。对于 `name=value` 格式，通常会对 `name` 和 `value` 分别进行编码，而不对等号 `=` 进行编码操作。

3. 过期时间选项

   紧跟 cookie 值后面的每个选项都以分号和空格分开，每个选择都指定了 cookie 在什么情况下应该被发送至服务器。第一个选项是过期时间（expires），指定了 cookie 何时不会再被发送至服务器，随后浏览器将删除该 cookie。该选项的值是一个 `Wdy, DD-Mon-YYYY HH:MM:SS GMT` 日期格式的值，例如：

   ```
   Set-Cookie: name=Nicholas; expires=Sat, 02 May 2009 23:38:25 GMT
   ```

   没有设置 `expires` 选项时，cookie 的生命周期仅限于当前会话中，关闭浏览器意味着这次会话的结束，所以会话 cookie 仅存在于浏览器打开状态之下。这就是为什么为什么当你登录一个 Web 应用时经常会看到一个复选框，询问你是否记住登录信息：如果你勾选了复选框，那么一个 `expires` 选项会被附加到登录 cookie 中。如果 `expires` 设置了一个过去的时间点，那么这个 cookie 会被立即删掉。

4. domain 选项

   下一个选项是 `domain`，指定了 cookie 将要被发送至哪个或哪些域中。默认情况下，`domain` 会被设置为创建该 cookie 的页面所在的域名，所以当给相同域名发送请求时该 cookie 会被发送至服务器。例如，本博中 cookie 的默认值将是 `bubkoo.com`。`domain` 选项可用来扩充 cookie 可发送域的数量，例如：

   ```
   Set-Cookie: name=Nicholas; domain=nczonline.net
   ```

   像 Yahoo! 这种大型网站，都会有许多 name.yahoo.com 形式的站点（例如：my.yahoo.com, finance.yahoo.com 等等）。将一个 cookie 的 `domain` 选项设置为 `yahoo.com`，就可以将该 cookie 的值发送至所有这些站点。浏览器会把 `domain` 的值与请求的域名做一个尾部比较（即从字符串的尾部开始比较），并将匹配的 cookie 发送至服务器。

   `domain` 选项的值必须是发送 `Set-Cookie` 消息头的主机名的一部分，例如我不能在 google.com 上设置一个 cookie，因为这会产生安全问题。不合法的 `domain` 选择将直接被忽略。

5. path 选项

   另一个控制 `Cookie` 消息头发送时机的选项是 `path` 选项，和 `domain` 选项类似，`path` 选项指定了请求的资源 URL 中必须存在指定的路径时，才会发送`Cookie` 消息头。这个比较通常是将 `path` 选项的值与请求的 URL 从头开始逐字符比较完成的。如果字符匹配，则发送 `Cookie` 消息头，例如：

   ```
   Set-Cookie:name=Nicholas;path=/blog
   ```

   在这个例子中，`path` 选项值会与 `/blog`，`/blogrool` 等等相匹配；任何以 `/blog` 开头的选项都是合法的。需要注意的是，只有在 `domain` 选项核实完毕之后才会对 `path` 属性进行比较。`path` 属性的默认值是发送 `Set-Cookie` 消息头所对应的 URL 中的 `path` 部分。

6. secure 选项
   最后一个选项是 `secure`。不像其它选项，该选项只是一个标记而没有值。只有当一个请求通过 SSL 或 HTTPS 创建时，包含 `secure` 选项的 cookie 才能被发送至服务器。这种 cookie 的内容具有很高的价值，如果以纯文本形式传递很有可能被篡改，例如：

   ```
   Set-Cookie: name=Nicholas; secure
   ```

   事实上，机密且敏感的信息绝不应该在 cookie 中存储或传输，因为 cookie 的整个机制原本都是不安全的。默认情况下，在 HTTPS 链接上传输的 cookie 都会被自动添加上 `secure` 选项。

**Cookie 的维护和生命周期**

在一个 cookie 中可以指定任意数量的选项，并且这些选项可以是任意顺序，例如：

```
Set-Cookie:name=Nicholas; domain=nczonline.net; path=/blog
```

这个 cookie 有四个标识符：cookie 的 `name`，`domain`，`path`，`secure` 标记。要想改变这个 cookie 的值，需要发送另一个具有相同 cookie `name`，`domain`，`path` 的 `Set-Cookie` 消息头。例如：

```
Set-Cookie: name=Greg; domain=nczonline.net; path=/blog
```

这将覆盖原来 cookie 的值。但是，修改 cookie 选项的任意一项都将创建一个完全不同的新 cookie，例如：

```
Set-Cookie: name=Nicholas; domain=nczonline.net; path=/
```

这个消息头返回之后，会同时存在两个名为 “name” 的不同的 cookie。如果你访问 `www.nczonline.net/blog` 下的一个页面，以下的消息头将被包含进来：

```
Cookie: name=Greg; name=Nicholas
```

在这个消息头中存在了两个名为 “name” 的 cookie，`path` 值越详细则 cookie 越靠前。 按照 `domain-path-secure` 的顺序，设置越详细的 cookie 在字符串中越靠前。假设我在 `ww.nczonline.net/blog` 下用默认选项创建了另一个 cookie：

```
Set-Cookie: name=Mike
```

那么返回的消息头现在则变为：

```
Cookie: name=Mike; name=Greg; name=Nicholas
```

以 “Mike” 作为值的 cookie 使用了域名（`www.nczonline.net`）作为其 `domain` 值并且以全路径（`/blog`）作为其 `path` 值，则它较其它两个 cookie 更加详细。

**使用失效日期**

当 cookie 创建时指定了失效日期，这个失效日期则关联了以 `name-domain-path-secure` 为标识的 cookie。要改变一个 cookie 的失效日期，你必须指定同样的组合。当改变一个 cookie 的值时，你不必每次都设置失效日期，因为它不是 cookie 标识信息的组成部分。例如：

```
Set-Cookie:name=Mike;expires=Sat,03 May 2025 17:44:22 GMT
```

现在已经设置了 cookie 的失效日期，所以下次我想要改变 cookie 的值时，我只需要使用它的名字：

```
Set-Cookie:name=Matt
```

cookie 的失效日期并没有改变，因为 cookie 的标识符是相同的。实际上，只有你手工的改变 cookie 的失效日期，否则其失效日期不会改变。这意味着在同一个会话中，一个会话 cookie 可以变成一个持久化 cookie（一个可以在多个会话中存在的），反之则不可。为了要将一个持久化 cookie 变为一个会话 cookie，你必须删除这个持久化 cookie，这只要设置它的失效日期为过去某个时间之后再创建一个同名的会话 cookie 就可以实现。

需要记得的是失效日期是以浏览器运行的电脑上的系统时间为基准进行核实的。没有任何办法来来验证这个系统时间是否和服务器的时间同步，所以当服务器时间和浏览器所处系统时间存在差异时这样的设置会出现错误。

**cookie 自动删除**

cookie 会被浏览器自动删除，通常存在以下几种原因：

- 会话 cooke (Session cookie) 在会话结束时（浏览器关闭）会被删除
- 持久化 cookie（Persistent cookie）在到达失效日期时会被删除
- 如果浏览器中的 cookie 数量达到限制，那么 cookie 会被删除以为新建的 cookie 创建空间。

对于自动删除来说，Cookie 管理显得十分重要，因为这些删除都是无意识的。

**Cookie 限制条件**

cookie 存在许多限制条件，来阻止 cookie 滥用并保护浏览器和服务器免受一些负面影响。有两种 cookie 限制条件：cookie 的属性和 cookie 的总大小。原始规范中限定每个域名下不超过 20 个 cookie，早期的浏览器都遵循该规范，并且在 IE7 中有更近一步的提升。在微软的一次更新中，他们在 IE7 中增加 cookie 的限制数量到 50 个，与此同时 Opera 限定 cookie 数量为 30 个，Safari 和 Chrome 对与每个域名下的 cookie 个数没有限制。

发向服务器的所有 cookie 的最大数量（空间）仍旧维持原始规范中所指出的：4KB。所有超出该限制的 cookie 都会被截掉并且不会发送至服务器。

**Subcookies**

鉴于 cookie 的数量存在限制，开发者提出 subcookies 的观点来增加 cookie 的存储量。Subcookies 是存储在一个 cookie 值中的一些 `name-value` 对，通常与以下格式类似：

```
name=a=b&c=d&e=f&g=h
```

这种方式允许在单个 cookie 中保存多个 `name-value` 对，而不会超出浏览器 cookie 数量的限制。通过这种方式创建 cookie 的负面影响是，需要自定义解析方式来提取这些值，相比较而言 cookie 的格式会更为简单。服务器端框架已开始支持 subcookies 的存储。

**JavaScript 中的 cookie**

在 JavaScript 中通过 `document.cookie` 属性，你可以创建、维护和删除 cookie。创建 cookie 时该属性等同于 `Set-Cookie` 消息头，而在读取 cookie 时则等同于 `Cookie` 消息头。在创建一个 cookie 时，你需要使用和 `Set-Cookie` 期望格式相同的字符串：

```
document.cookie="name=Nicholas;domain=nczonline.net;path=/";
```

设置 `document.cookie` 属性的值并不会删除存储在页面中的所有 cookie。它只简单的创建或修改字符串中指定的 cookie。下次发送一个请求到服务器时，通过 `document.cookie` 设置的 cookie 会和其它通过 `Set-Cookie` 消息头设置的 cookie 一并发送至服务器。这些 cookie 并没有什么明确的不同之处。

要使用 JavaScript 提取 cookie 的值，只需要从 `document.cookie` 中读取即可。返回的字符串与 `Cookie` 消息头中的字符串格式相同，所以多个 cookie 会被分号和字符串分割。例如：

```
name1=Greg; name2=Nicholas
```

鉴于此，你需要手工解析这个 cookie 字符串来提取真实的 cookie 数据。

通过访问 `document.cookie` 返回的 cookie 遵循发向服务器的 cookie 一样的访问规则。要通过 JavaScript 访问 cookie，该页面和 cookie 必须在相同的域中，有相同的 `path`，有相同的安全级别。

注意：一旦 cookie 通过 JavaScript 设置后便不能提取它的选项，所以你将不能知道 `domain`，`path`，`expires` 日期或 `secure` 标记。

**cookie确切的说分为两大类**：会话cookie和持久化cookie。

**会话cookie**是存放在客户端浏览器的内存中，他的生命周期和浏览器是一致的，当浏览器关闭会话cookie也就消失了

**持久化cookie**是存放在客户端硬盘中，持久化cookie的生命周期是我们在设置cookie时候设置的那个保存时间，session的信息是通过sessionid获取的，而sessionid是存放在会话cookie当中的，当浏览器关闭的时候会话cookie消失，所以sessionid也就消失了，但是session的信息还存在服务器端，只是查不到所谓的session但它并不是不存在。所以session在服务器关闭的时候，或者是sessio过期，又或者调用了invalidate()，再或者是session中的某一条数据消失调用session.removeAttribute()方法，session在通过调用session.getsession来创建的。

#### Session

session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构(也可能就是使用散列表)来保存信息。但程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否包含了一个session标识 —— 称为session id。

如果已经包含一个session id则说明以前已经为此客户创建过session，服务器就按照session id把这个session检索出来使用(如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的session对象，但用户人为地在请求的URL后面附加上一个JSESSION的参数)。

如果客户请求不包含session id，则为此客户创建一个session并且生成一个与此session相关联的session id，这个session id将在本次响应中返回给客户端保存。

> cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。 

**Cookie和Session、SessionID的关系**

sessionid 是一个会话的 key，浏览器第一次访问服务器会在服务器端生成一个 session，有一个 sessionID 和它对应，并返回给浏览器，这个 sessionID 会被保存在浏览器的会话 cookie 中。tomcat 生成的 sessionID 叫做 jsessionID。

sessionID 在访问 tomcat 服务器 HttpServletRequest 的 getSession(true) 的时候创建，tomcat 的 ManagerBase 类提供创建sessionID 的方法：随机数+时间+jvmid。Tomcat 的 StandardManager 类将 session 存储在内存中，也可以持久化到 file，数据库，memcache，redis等。

客户端只保存 sessionID 到 cookie 中，而不会保存 session。session 不会因为浏览器的关闭而删除，只能通过程序调用 HttpSession.invalidate() 或超时才能销毁。

1. session 的 id 是从哪里来的，sessionID 是如何使用的？

   当客户端第一次请求 session 对象时候，服务器会为客户端创建一个 session，并将通过特殊算法算出一个 session 的 ID，用来标识该 session 对象。

2. session 存放在哪里？

   服务器端的内存中。不过 session 可以通过特殊的方式做持久化管理（memcache，redis）。

3. session在下列情况下被删除

   - 程序调用HttpSession.invalidate()
   - 距离上一次收到客户端发送的session id时间间隔超过了session的最大有效时间
   - 服务器进程被停止

   注意：

   - 客户端只保存 sessionID 到 cookie 中，而不会保存 session。
   - 关闭浏览器只会使存储在客户端浏览器内存中的session cookie失效，不会使服务器端的session对象失效，同样也不会使已经保存到硬盘上的持久化cookie消失。

4. session cookie和session对象的生命周期是一样的吗

    当用户关闭了浏览器虽然session cookie已经消失，但session对象仍然保存在服务器端。

5. Session 在何时创建呢？

    一个常见的错误是以为session在有客户端访问时就被创建，然而事实是直到某server端程序(如Servlet)调用HttpServletRequest.getSession(true)这样的语句时才会被创建。

   在创建了Session的同时，服务器会为该Session生成唯一的Session id，而这个Session id在随后的请求中会被用来重新获得已经创建的Session；在Session被创建之后，就可以调用Session相关的方法往Session中增加内容了，而这些内容只会保存在服务器中，发到客户端的只有Session id；当客户端再次发送请求的时候，会将这个Session id带上，服务器接受到请求之后就会依据Session id找到相应的Session，从而再次使用之。

6. 打开两个浏览器窗口访问应用程序会使用同一个session还是不同的session？

   通常session cookie是不能跨窗口使用的，当你新开了一个浏览器窗口进入相同页面时，系统会赋予你一个新的session id，这样我们信息共享的目的就达不到了。

   此时我们可以先把session id保存在persistent cookie中(通过设置session的最大有效时间)，然后在新窗口中读出来，就可以得到上一个窗口的session id了，这样通过session cookie和persistent cookie的结合我们就可以实现了跨窗口的会话跟踪。

**过程(服务端session + 客户端 sessionId)**

1. 用户向服务器发送用户名和密码

2. 服务器验证通过后,在当前对话(session)里面保存相关数据,比如用户角色, 登陆时间等;

3. 服务器向用户返回一个`session_id`, 写入用户的`cookie`

4. 用户随后的每一次请求, 都会通过`cookie`, 将`session_id`传回服务器

5. 服务端收到 `session_id`, 找到前期保存的数据, 由此得知用户的身份

**客户端用cookie保存了sessionID时**

客户端用cookie保存了sessionID，当我们请求服务器的时候，会把这个sessionID一起发给服务器，服务器会到内存中搜索对应的sessionID，如果找到了对应的 sessionID，说明我们处于登录状态，有相应的权限；如果没有找到对应的sessionID，这说明：要么是我们把浏览器关掉了（后面会说明为什么），要么session超时了（没有请求服务器超过20分钟），session被服务器清除了，则服务器会给你分配一个新的sessionID。你得重新登录并把这个新的sessionID保存在cookie中。

在没有把浏览器关掉的时候（这个时候假如已经把sessionID保存在cookie中了）这个sessionID会一直保存在浏览器中，每次请求的时候都会把这个sessionID提交到服务器，所以服务器认为我们是登录的；当然，如果太长时间没有请求服务器，服务器会认为我们已经所以把浏览器关掉了，这个时候服务器会把该sessionID从内存中清除掉，这个时候如果我们再去请求服务器，sessionID已经不存在了，所以服务器并没有在内存中找到对应的 sessionID，所以会再产生一个新的sessionID，这个时候一般我们又要再登录一次。

**客户端没有用cookie保存sessionID时**

这个时候如果我们请求服务器，因为没有提交sessionID上来，服务器会认为你是一个全新的请求，服务器会给你分配一个新的sessionID，这就是 为什么我们每次打开一个新的浏览器的时候（无论之前我们有没有登录过）都会产生一个新的sessionID（或者是会让我们重新登录）。

当我们一旦把浏览器关掉后，再打开浏览器再请求该页面，它会让我们登录，这是为什么？我们明明已经登录了，而且还没有超时，sessionID肯定还在服务器上的，为什么现在我们又要再一次登录呢？这是因为我们关掉浏览再请求的时候，我们提交的信息没有把刚才的sessionID一起提交到服务器，所以服务器不知道我们是同一个人，所以这时服务器又为我们分配一个新的sessionID，打个比方：浏览器就好像一个要去银行开户的人，而服务器就好比银行， 这个要去银行开户的人这个时候显然没有帐号（sessionID），所以到银行后，银行工作人员问有没有帐号，他说没有，这个时候银行就会为他开通一个帐号（sessionID）。所以可以这么说，每次打开一个新的浏览器去请求的一个页面的时候，服务器都会认为，这是一个新的请求，他为你分配一个新的sessionID。

**存在的问题:扩展性不好**

单机当然没问题， 如果是服务器集群， 或者是跨域的服务导向架构， 这就要求session数据共享，每台服务器都能够读取session。

举例来说， A网站和B网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？这个问题就是如何实现单点登录的问题

1. Nginx ip_hash 策略，服务端使用 Nginx 代理，每个请求按访问 IP 的 hash 分配，这样来自同一 IP 固定访问一个后台服务器，避免了在服务器 A 创建 Session，第二次分发到服务器 B 的现象。

2. Session复制：任何一个服务器上的 Session 发生改变（增删改），该节点会把这个 Session 的所有内容序列化，然后广播给所有其它节点。

3. 共享Session：将Session Id 集中存储到一个地方，所有的机器都来访问这个地方的数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败；

另一种方案是服务器索性不保存session数据了，所有数据就保存在客户端，每次请求都发回服务器。这种方案就是接下来要介绍的基于Token的验证;

#### token

token 也称作令牌，由uid+time+sign[+固定参数]
token 的认证方式类似于**临时的证书签名**, 并且是一种服务端无状态的认证方式, 非常适合于 REST API 的场景. 所谓无状态就是服务端并不会保存身份认证相关的数据。

**组成**

- uid: 用户唯一身份标识
- time: 当前时间的时间戳
- sign: 签名, 使用 hash/encrypt 压缩成定长的十六进制字符串，以防止第三方恶意拼接
- 固定参数(可选): 将一些常用的固定参数加入到 token 中是为了避免重复查库

**存放**

token在客户端一般存放于localStorage，cookie，或sessionStorage中。在服务器一般存于数据库中

**token认证流程**

token 的认证流程与cookie很相似

- 用户登录，成功后服务器返回Token给客户端。
- 客户端收到数据后保存在客户端
- 客户端再次访问服务器，将token放入headers中
- 服务器端采用filter过滤器校验。校验成功则返回请求数据，校验失败则返回错误码

**过程**

1. 用户通过用户名和密码发送请求
2. 程序验证
3. 程序返回一个签名的token给客户端
4. 客户端储存token, 并且每次用每次发送请求
5. 服务端验证Token并返回数据

**token可以抵抗csrf，cookie+session不行**

假如用户正在登陆银行网页，同时登陆了攻击者的网页，并且银行网页未对csrf攻击进行防护。攻击者就可以在网页放一个表单，该表单提交src为`http://www.bank.com/api/transfer`，body为`count=1000&to=Tom`。倘若是session+cookie，用户打开网页的时候就已经转给Tom1000元了.因为form 发起的 POST 请求并不受到浏览器同源策略的限制，因此可以任意地使用其他域的 Cookie 向其他域发送 POST 请求，形成 CSRF 攻击。在post请求的瞬间，cookie会被浏览器自动添加到请求头中。但token不同，token是开发者为了防范csrf而特别设计的令牌，浏览器不会自动添加到headers里，攻击者也无法访问用户的token，所以提交的表单无法通过服务器过滤，也就无法形成攻击。

**分布式情况下的session和token**

我们已经知道session时有状态的，一般存于服务器内存或硬盘中，当服务器采用分布式或集群时，session就会面对负载均衡问题。

- 负载均衡多服务器的情况，不好确认当前用户是否登录，因为多服务器不共享session。这个问题也可以将session存在一个服务器中来解决，但是就不能完全达到负载均衡的效果。当今的几种[解决session负载均衡](http://blog.51cto.com/zhibeiwang/1965018)的方法。

而token是无状态的，token字符串里就保存了所有的用户信息

- 客户端登陆传递信息给服务端，服务端收到后把用户信息加密（token）传给客户端，客户端将token存放于localStroage等容器中。客户端每次访问都传递token，服务端解密token，就知道这个用户是谁了。通过cpu加解密，服务端就不需要存储session占用存储空间，就很好的解决负载均衡多服务器的问题了。 这个方法叫做[JWT(Json Web Token)](https://huanqiang.wang/2017/12/28/JWT 介绍/)

#### 总结

- session存储于服务器，可以理解为一个状态列表，拥有一个唯一识别符号sessionId，通常存放于cookie中。服务器收到cookie后解析出sessionId，再去session列表中查找，才能找到相应session。依赖cookie
- cookie类似一个令牌，装有sessionId，存储在客户端，浏览器通常会自动添加。
- token也类似一个令牌，无状态，用户信息都被加密到token中，服务器收到token后解密就可知道是哪个用户。需要开发者手动添加。
- jwt只是一个跨域认证的方案

### 参考

1. [Cookie、Session、Token那点事儿](https://www.jianshu.com/p/bd1be47a16c1)
3. [cookie,token验证的区别](https://www.jianshu.com/p/c33f5777c2eb)
4. [有了cookie为什么需要session](https://segmentfault.com/q/1010000016504003)
5. [CSRF Token的设计是否有其必要性](https://segmentfault.com/q/1010000000713614)
6. [cookie,token,session三者的问题和解决方案](https://junyiseo.com/php/757.html)
7. [负载均衡集群中的session解决方案](http://blog.51cto.com/zhibeiwang/1965018)
8. [JWT介绍](https://huanqiang.wang/2017/12/28/JWT 介绍/)
9. [Json Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
10. [Cookie和Session的使用和区别](https://www.jianshu.com/p/9a561b36e9f3)
11. [彻底理解cookie,session,token](https://www.liangzl.com/get-article-detail-16019.html)
12. [JSON Web Token 入门教程](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2018%2F07%2Fjson_web_token-tutorial.html)
13. [Cookie、Session、Token那点事儿（原创）](https://www.jianshu.com/p/bd1be47a16c1)
14. [3种web会话管理的方式](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Flyzg%2Fp%2F6067766.html)
15. [你真的了解 Cookie 和 Session 吗](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5cd9037ee51d456e5c5babca)
16. [不要用JWT替代session管理（上）：全面了解Token,JWT,OAuth,SAML,SSO](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F38942172)