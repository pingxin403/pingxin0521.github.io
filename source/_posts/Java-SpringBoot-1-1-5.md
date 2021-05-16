---
title: Spring boot 整合WebSocket
date: 2019-07-18 10:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

#### 前言

WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

<!--more-->

在 WebSocket API 中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

现在，很多网站为了实现推送技术，所用的技术都是 Ajax 轮询。轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

HTML5 定义的 WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

![8jeQMt.png](https://s1.ax1x.com/2020/03/25/8jeQMt.png)

浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过 TCP 连接直接交换数据。

当你获取 Web Socket 连接后，你可以通过 **send()** 方法来向服务器发送数据，并通过 **onmessage** 事件来接收服务器返回的数据。

以下 API 用于创建 WebSocket 对象。

```
var Socket = new WebSocket(url, [protocol] );
```

以上代码中的第一个参数 url, 指定连接的 URL。第二个参数 protocol 是可选的，指定了可接受的子协议。

Websocket 使用 ws 或 wss 的统一资源标志符，类似于 HTTPS，其中 wss 表示在 TLS 之上的 Websocket。如：

```
ws://example.com/wsapi
wss://secure.example.com/
```

Websocket 使用和 HTTP 相同的 TCP 端口，可以绕过大多数防火墙的限制。默认情况下，Websocket 协议使用 80 端口；运行在 TLS 之上时，默认使用 443 端口。

**一个典型的Websocket握手请求如下：**

客户端请求

```
GET / HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Host: example.com
Origin: http://example.com
Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
Sec-WebSocket-Version: 13
```

服务器回应

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=
Sec-WebSocket-Location: ws://example.com/
```

-  Connection 必须设置 Upgrade，表示客户端希望连接升级。
-  Upgrade 字段必须设置 Websocket，表示希望升级到 Websocket 协议。
-  Sec-WebSocket-Key 是随机的字符串，服务器端会用这些数据来构造出一个 SHA-1 的信息摘要。把 “Sec-WebSocket-Key” 加上一个特殊字符串 “258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算 SHA-1 摘要，之后进行 BASE-64 编码，将结果做为 “Sec-WebSocket-Accept” 头的值，返回给客户端。如此操作，可以尽量避免普通 HTTP 请求被误认为 Websocket 协议。
-  Sec-WebSocket-Version 表示支持的 Websocket 版本。RFC6455 要求使用的版本是 13，之前草案的版本均应当弃用。
-  Origin 字段是可选的，通常用来表示在浏览器中发起此 Websocket 连接所在的页面，类似于 Referer。但是，与 Referer 不同的是，Origin 只包含了协议和主机名称。
-  其他一些定义在 HTTP 协议中的字段，如 Cookie 等，也可以在 Websocket 中使用。

在服务器方面，网上都有不同对websocket支持的服务器：

- php - http://code.google.com/p/phpwebsocket/
- jetty - http://jetty.codehaus.org/jetty/（版本7开始支持websocket）
- netty - http://www.jboss.org/netty
- ruby - http://github.com/gimite/web-socket-ruby
- Kaazing - https://web.archive.org/web/20100923224709/http://www.kaazing.org/confluence/display/KAAZING/Home
- Tomcat - [http://tomcat.apache.org/（7.0.27支持websocket，建议用tomcat8，7.0.27中的接口已经过时）](http://tomcat.apache.org/)
- WebLogic - [http://www.oracle.com/us/products/middleware/cloud-app-foundation/weblogic/overview/index.html（12.1.2開始支持）](http://www.oracle.com/us/products/middleware/cloud-app-foundation/weblogic/overview/index.html)
- node.js - https://github.com/Worlize/WebSocket-Node
- node.js - [http://socket.io](http://socket.io/)
- nginx - http://nginx.com/
- mojolicious - http://mojolicio.us/
- python - https://github.com/abourget/gevent-socketio
- Django - https://github.com/stephenmcd/django-socketio
- erlang - https://github.com/ninenines/cowboy.git

**WebSocket 属性**

以下是 WebSocket 对象的属性。假定我们使用了以上代码创建了 Socket 对象：

| 属性                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| Socket.readyState     | 只读属性 **readyState** 表示连接状态，可以是以下值：0 - 表示连接尚未建立。1 - 表示连接已建立，可以进行通信。2 - 表示连接正在进行关闭。3 - 表示连接已经关闭或者连接不能打开。 |
| Socket.bufferedAmount | 只读属性 **bufferedAmount** 已被 send() 放入正在队列中等待传输，但是还没有发出的 UTF-8 文本字节数。 |

**WebSocket 事件**

以下是 WebSocket 对象的相关事件。假定我们使用了以上代码创建了 Socket 对象：

| 事件    | 事件处理程序     | 描述                       |
| :------ | :--------------- | :------------------------- |
| open    | Socket.onopen    | 连接建立时触发             |
| message | Socket.onmessage | 客户端接收服务端数据时触发 |
| error   | Socket.onerror   | 通信发生错误时触发         |
| close   | Socket.onclose   | 连接关闭时触发             |

**WebSocket 方法**

以下是 WebSocket 对象的相关方法。假定我们使用了以上代码创建了 Socket 对象：

| 方法           | 描述             |
| :------------- | :--------------- |
| Socket.send()  | 使用连接发送数据 |
| Socket.close() | 关闭连接         |

##### WebSocket 实例

WebSocket 协议本质上是一个基于 TCP 的协议。

为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息"Upgrade: WebSocket"表明这是一个申请协议升级的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。

1. 服务端

   在执行以上程序前，我们需要创建一个支持 WebSocket 的服务。从 [pywebsocket](https://github.com/google/pywebsocket) 下载 mod_pywebsocket ,或者使用 git 命令下载：

   ```
   git clone https://github.com/google/pywebsocket.git
   ```

   mod_pywebsocket 需要 python 环境支持

   mod_pywebsocket 是一个 Apache HTTP 的 Web Socket扩展，安装步骤如下：

   - 解压下载的文件。

   - 进入 **pywebsocket** 目录。

   - 执行命令：

     ```
     $ python setup.py build
     $ sudo python setup.py install
     ```

   - 查看文档说明:

     ```
     $ pydoc mod_pywebsocket
     ```

2. 开启服务

   在 **pywebsocket/mod_pywebsocket** 目录下执行以下命令：

   ```
   $ sudo python standalone.py -p 9998 -w ../example/
   ```

   以上命令会开启一个端口号为 9998 的服务，使用 -w 来设置处理程序 echo_wsh.py 所在的目录。

3. 客户端index.html

   ```html
   <!DOCTYPE HTML>
   <html>
      <head>
      <meta charset="utf-8">
      <title>WebSocket </title>
       
         <script type="text/javascript">
            function WebSocketTest()
            {
               if ("WebSocket" in window)
               {
                  alert("您的浏览器支持 WebSocket!");
                  
                  // 打开一个 web socket
                  var ws = new WebSocket("ws://localhost:9998/echo");
                   
                  ws.onopen = function()
                  {
                     // Web Socket 已连接上，使用 send() 方法发送数据
                     ws.send("发送数据");
                     alert("数据发送中...");
                  };
                   
                  ws.onmessage = function (evt) 
                  { 
                     var received_msg = evt.data;
                     alert("数据已接收...");
                  };
                   
                  ws.onclose = function()
                  { 
                     // 关闭 websocket
                     alert("连接已关闭..."); 
                  };
               }
               
               else
               {
                  // 浏览器不支持 WebSocket
                  alert("您的浏览器不支持 WebSocket!");
               }
            }
         </script>
           
      </head>
      <body>
      
         <div id="sse">
            <a href="javascript:WebSocketTest()">运行 WebSocket</a>
         </div>
         
      </body>
   </html>
   ```

![8j1hVI.png](https://s1.ax1x.com/2020/03/25/8j1hVI.png)

##### websocket跟 socket 的区别

软件通信有七层结构，下三层结构偏向与数据通信，上三层更偏向于数据处理，中间的传输层则是连接上三层与下三层之间的桥梁，每一层都做不同的工作，上层协议依赖与下层协议。基于这个通信结构的概念。

Socket 其实并不是一个协议，是应用层与 TCP/IP 协议族通信的中间软件抽象层，它是一组接口。当两台主机通信时，让 Socket 去组织数据，以符合指定的协议。TCP 连接则更依靠于底层的 IP 协议，IP 协议的连接则依赖于链路层等更低层次。

WebSocket 则是一个典型的应用层协议。

总的来说：Socket 是传输控制层协议，WebSocket 是应用层协议。

#### WebSocket 实现

1. maven依赖

   ```xml
       <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
   ```

2. 配置文件

   ```properties
   spring.application.name=websocket
   server.port=8888
   ```

3. socket管理器

   ```java
   
   /**
    * socket管理器
    */
   @Slf4j
   public class SocketManager {
       private static ConcurrentHashMap<String, WebSocketSession> manager = new ConcurrentHashMap<String, WebSocketSession>();
   
       public static void add(String key, WebSocketSession webSocketSession) {
           log.info("新添加webSocket连接 {} ", key);
           manager.put(key, webSocketSession);
       }
   
       public static void remove(String key) {
           log.info("移除webSocket连接 {} ", key);
           manager.remove(key);
       }
   
       public static WebSocketSession get(String key) {
           log.info("获取webSocket连接 {}", key);
           return manager.get(key);
       }
   
   
   }
   ```

4. 服务端和客户端在进行握手挥手时会被执行

   ```java
   @Component
   @Slf4j
   public class WebSocketDecoratorFactory implements WebSocketHandlerDecoratorFactory {
       @Override
       public WebSocketHandler decorate(WebSocketHandler handler) {
           return new WebSocketHandlerDecorator(handler) {
               @Override
               public void afterConnectionEstablished(WebSocketSession session) throws Exception {
                   log.info("有人连接啦  sessionId = {}", session.getId());
                   Principal principal = session.getPrincipal();
                   if (principal != null) {
                       log.info("key = {} 存入", principal.getName());
                       // 身份校验成功，缓存socket连接
                       SocketManager.add(principal.getName(), session);
                   }
   
   
                   super.afterConnectionEstablished(session);
               }
   
               @Override
               public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
                   log.info("有人退出连接啦  sessionId = {}", session.getId());
                   Principal principal = session.getPrincipal();
                   if (principal != null) {
                       // 身份校验成功，移除socket连接
                       SocketManager.remove(principal.getName());
                   }
                   super.afterConnectionClosed(session, closeStatus);
               }
           };
       }
   }
   ```

   注意：以上session变量，需要注意两点，一个是getId(),一个是getPrincipal().getName()
   getId() ： 返回的是唯一的会话标识符。

   getPrincipal() : 经过身份验证，返回Principal实例，未经过身份验证，返回null

   Principal: 委托人的抽象概念,可以是公司id，名字，用户唯一识别token等

   当你按上面代码使用，你会发现getPrincipal()返回null，为什么？这边还需要重写一个DefaultHandshakeHandler

5. 我们可以通过请求信息，比如token、或者session判用户是否可以连接，这样就能够防范非法用户

   ```java
   @Slf4j
   @Component
   public class PrincipalHandshakeHandler extends DefaultHandshakeHandler {
       @Override
       protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {
           /**
            * 这边可以按你的需求，如何获取唯一的值，既unicode
            * 得到的值，会在监听处理连接的属性中，既WebSocketSession.getPrincipal().getName()
            * 也可以自己实现Principal()
            */
           if (request instanceof ServletServerHttpRequest) {
               ServletServerHttpRequest servletServerHttpRequest = (ServletServerHttpRequest) request;
               HttpServletRequest httpRequest = servletServerHttpRequest.getServletRequest();
               /**
                * 这边就获取你最熟悉的陌生人,携带参数，你可以cookie，请求头，或者url携带，这边我采用url携带
                */
               final String token = httpRequest.getParameter("token");
               if (StringUtils.isEmpty(token)) {
                   return null;
               }
               return new Principal() {
                   @Override
                   public String getName() {
                       return token;
                   }
               };
           }
           return null;
       }
   }
   
   ```

6. WebSocketConfig配置

   ```java
   /**
    * WebSocketConfig配置
    */
   @Configuration
   @EnableWebSocketMessageBroker
   public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
   
       @Autowired
       private WebSocketDecoratorFactory webSocketDecoratorFactory;
       @Autowired
       private PrincipalHandshakeHandler principalHandshakeHandler;
   
       @Override
       public void registerStompEndpoints(StompEndpointRegistry registry) {
           /**
            * myUrl表示 你前端到时要对应url映射
            */
           registry.addEndpoint("/myUrl")
                   .setAllowedOrigins("*")
                   .setHandshakeHandler(principalHandshakeHandler)
                   .withSockJS();
       }
   
       @Override
       public void configureMessageBroker(MessageBrokerRegistry registry) {
           /**
            * queue 点对点
            * topic 广播
            * user 点对点前缀
            */
           registry.enableSimpleBroker("/queue", "/topic");
           registry.setUserDestinationPrefix("/user");
       }
   
       @Override
       public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
           registration.addDecoratorFactory(webSocketDecoratorFactory);
       }
   }
   ```

7. 控制器

   ```java
   
   @RestController
   @Slf4j
   public class TestController {
   
       @Autowired
       private SimpMessagingTemplate template;
   
       /**
        * 服务器指定用户进行推送,需要前端开通 var socket = new SockJS(host+'/myUrl' + '?token=1234');
        */
       @RequestMapping("/sendUser")
       public void sendUser(String token) {
           log.info("token = {} ,对其发送您好", token);
           WebSocketSession webSocketSession = SocketManager.get(token);
           if (webSocketSession != null) {
               /**
                * 主要防止broken pipe
                */
               template.convertAndSendToUser(token, "/queue/sendUser", "您好");
           }
   
       }
   
       /**
        * 广播，服务器主动推给连接的客户端
        */
       @RequestMapping("/sendTopic")
       public void sendTopic() {
           template.convertAndSend("/topic/sendTopic", "大家晚上好");
   
       }
   
       /**
        * 客户端发消息，服务端接收
        *
        * @param message
        */
       // 相当于RequestMapping
       @MessageMapping("/sendServer")
       public void sendServer(String message) {
           log.info("message:{}", message);
       }
   
       /**
        * 客户端发消息，大家都接收，相当于直播说话
        *
        * @param message
        * @return
        */
       @MessageMapping("/sendAllUser")
       @SendTo("/topic/sendTopic")
       public String sendAllUser(String message) {
           // 也可以采用template方式
           return message;
       }
   
       /**
        * 点对点用户聊天，这边需要注意，由于前端传过来json数据，所以使用@RequestBody
        * 这边需要前端开通var socket = new SockJS(host+'/myUrl' + '?token=4567');   token为指定name
        * @param map
        */
       @MessageMapping("/sendMyUser")
       public void sendMyUser(@RequestBody Map<String, String> map) {
           log.info("map = {}", map);
           WebSocketSession webSocketSession = SocketManager.get(map.get("name"));
           if (webSocketSession != null) {
               log.info("sessionId = {}", webSocketSession.getId());
               template.convertAndSendToUser(map.get("name"), "/queue/sendUser", map.get("message"));
           }
       }
   
   
   }
   
   ```

8. 页面

   ```java
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="UTF-8" />
   <title>Spring Boot WebSocket+广播式</title>
   </head>
   <body>
       <noscript>
           <h2 style="color:#ff0000">貌似你的浏览器不支持websocket</h2>
       </noscript>
       <div>
           <div>
               <button id="connect" onclick="connect()">连接</button>
               <button id="disconnect"  onclick="disconnect();">断开连接</button>
           </div>
           <div id="conversationDiv">
               <label>输入你的名字</label> <input type="text" id="name" />
               <br>
               <label>输入消息</label> <input type="text" id="messgae" />
               <button id="send" onclick="send();">发送</button>
               <p id="response"></p>
           </div>
       </div>
       <script src="https://cdn.bootcss.com/sockjs-client/1.1.4/sockjs.min.js"></script>
       <script src="https://cdn.bootcss.com/stomp.js/2.3.3/stomp.min.js"></script>
       <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
       <script type="text/javascript">
           var stompClient = null;
           //gateway网关的地址
           var host="http://127.0.0.1:8888";
           function setConnected(connected) {
               document.getElementById('connect').disabled = connected;
               document.getElementById('disconnect').disabled = !connected;
               document.getElementById('conversationDiv').style.visibility = connected ? 'visible' : 'hidden';
               $('#response').html();
           }
           // SendUser ***********************************************
           function connect() {
               //地址+端点路径，构建websocket链接地址,注意，对应config配置里的addEndpoint
               var socket = new SockJS(host+'/myUrl' + '?token=4567');
               stompClient = Stomp.over(socket);
               stompClient.connect({}, function(frame) {
                   setConnected(true);
                   console.log('Connected:' + frame);
                   //监听的路径以及回调
                   stompClient.subscribe('/user/queue/sendUser', function(response) {
                       showResponse(response.body);
                   });
               });
           }
           /*
           function connect() {
               //地址+端点路径，构建websocket链接地址,注意，对应config配置里的addEndpoint
               var socket = new SockJS(host+'/myUrl');
               stompClient = Stomp.over(socket);
               stompClient.connect({}, function(frame) {
                   setConnected(true);
                   console.log('Connected:' + frame);
                   //监听的路径以及回调
                   stompClient.subscribe('/topic/sendTopic', function(response) {
                       showResponse(response.body);
                   });
               });
           }*/
           function disconnect() {
               if (stompClient != null) {
                   stompClient.disconnect();
               }
               setConnected(false);
               console.log("Disconnected");
           }
           function send() {
               var name = $('#name').val();
               var message = $('#messgae').val();
               /*//发送消息的路径,由客户端发送消息到服务端 
               stompClient.send("/sendServer", {}, message);
               */
               /*// 发送给所有广播sendTopic的人,客户端发消息，大家都接收，相当于直播说话 注：连接需开启 /topic/sendTopic
               stompClient.send("/sendAllUser", {}, message);
               */
               /* 这边需要注意，需要启动不同的前端html进行测试，需要改不同token ，例如 token=1234，token=4567
               * 然后可以通过写入name 为token  进行指定用户发送
               */
               stompClient.send("/sendMyUser", {}, JSON.stringify({name:name,message:message}));
           }
           function showResponse(message) {
               var response = $('#response');
               response.html(message);
           }
       </script>
   </body>
   </html>
   ```

   