---
title: JavaScript AJAX
date: 2019-05-07 11:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

### [AJAX](http://jquery.cuishifeng.cn/jQuery.Ajax.html)

AJAX不是JavaScript的规范，它只是一个哥们“发明”的缩写：Asynchronous JavaScript and XML，意思就是用JavaScript执行异步网络请求。

如果仔细观察一个Form的提交，你就会发现，一旦用户点击“Submit”按钮，表单开始提交，浏览器就会刷新页面，然后在新页面里告诉你操作是成功了还是失败了。如果不幸由于网络太慢或者其他原因，就会得到一个404页面。

<!--more-->

这就是Web的运作原理：一次HTTP请求对应一个页面。

如果要让用户留在当前页面中，同时发出新的HTTP请求，就必须用JavaScript发送这个新请求，接收到数据后，再用JavaScript更新页面，这样一来，用户就感觉自己仍然停留在当前页面，但是数据却可以不断地更新。

![](https://i.loli.net/2019/06/03/5cf49626e9b1e75410.png)

最早大规模使用AJAX的就是Gmail，Gmail的页面在首次加载后，剩下的所有数据都依赖于AJAX来更新。

用JavaScript写一个完整的AJAX代码并不复杂，但是需要注意：AJAX请求是异步执行的，也就是说，要通过回调函数获得响应。

在现代浏览器上写AJAX主要依靠`XMLHttpRequest`对象：

```
function success(text) {
    var textarea = document.getElementById('test-response-text');
    textarea.value = text;
}

function fail(code) {
    var textarea = document.getElementById('test-response-text');
    textarea.value = 'Error code: ' + code;
}

var request = new XMLHttpRequest(); // 新建XMLHttpRequest对象

request.onreadystatechange = function () { // 状态发生变化时，函数被回调
    if (request.readyState === 4) { // 成功完成
        // 判断响应结果:
        if (request.status === 200) {
            // 成功，通过responseText拿到响应的文本:
            return success(request.responseText);
        } else {
            // 失败，根据响应码判断失败原因:
            return fail(request.status);
        }
    } else {
        // HTTP请求还在继续...
    }
}

// 发送请求:
request.open('GET', '/api/categories');
request.send();

alert('请求已发送，请等待响应...');
```

对于低版本的IE，需要换一个`ActiveXObject`对象：

```
function success(text) {
    var textarea = document.getElementById('test-ie-response-text');
    textarea.value = text;
}

function fail(code) {
    var textarea = document.getElementById('test-ie-response-text');
    textarea.value = 'Error code: ' + code;
}

var request = new ActiveXObject('Microsoft.XMLHTTP'); // 新建Microsoft.XMLHTTP对象

request.onreadystatechange = function () { // 状态发生变化时，函数被回调
    if (request.readyState === 4) { // 成功完成
        // 判断响应结果:
        if (request.status === 200) {
            // 成功，通过responseText拿到响应的文本:
            return success(request.responseText);
        } else {
            // 失败，根据响应码判断失败原因:
            return fail(request.status);
        }
    } else {
        // HTTP请求还在继续...
    }
}

// 发送请求:
request.open('GET', '/api/categories');
request.send();

alert('请求已发送，请等待响应...');

```

如果你想把标准写法和IE写法混在一起，可以这么写：

```
var request;
if (window.XMLHttpRequest) {
    request = new XMLHttpRequest();
} else {
    request = new ActiveXObject('Microsoft.XMLHTTP');
}
```

通过检测`window`对象是否有`XMLHttpRequest`属性来确定浏览器是否支持标准的`XMLHttpRequest`。注意，*不要*根据浏览器的`navigator.userAgent`来检测浏览器是否支持某个JavaScript特性，一是因为这个字符串本身可以伪造，二是通过IE版本判断JavaScript特性将非常复杂。



Ajax的核心是XMLHttpRequest对象(XHR)。XHR为向服务器发送请求和解析服务器响应提供了接口。能够以异步方式从服务器获取新数据。

 **XHR的主要方法有：**

```
1. void open(String method,String url,Boolen async)   
   用于创建请求    
   参数：
       method： 请求方式（字符串类型），如：POST、GET、DELETE...
       url：    要请求的地址（字符串类型）
       async：  是否异步（布尔类型）
 
2. void send(String body)
    用于发送请求
    参数：
        body： 要发送的数据（字符串类型） 
3. void setRequestHeader(String header,String value)
    用于设置请求头
    参数：
        header： 请求头的key（字符串类型）
        vlaue：  请求头的value（字符串类型）
 
4. String getAllResponseHeaders()
    获取所有响应头
    返回值：
        响应头数据（字符串类型）
 
5. String getResponseHeader(String header)
    获取响应头中指定header的值
    参数：
        header： 响应头的key（字符串类型）
    返回值：
        响应头中指定的header对应的值
 
6. void abort()
    终止请求
```

 **XHR的主要属性有：**

1. **Number readyState** 状态值（整数），可以确定请求/响应过程的当前活动阶段

-  0：未初始化。未调用open()方法
-  1：启动。已经调用open()方法，未调用send()方法
-  2：发送。已经调用send()方法，未接收到响应
-  3：接收。已经接收到部分数据
-  4：完成。已经接收到全部数据，可以在客户端使用

2.  **Function onreadystatechange** 当readyState的值改变时自动触发执行其对应的函数（回调函数）

3. **String responseText** 作为响应主体被返回的文本（字符串类型）

4. **XmlDocument responseXML** 服务器返回的数据（Xml对象）

5. **Number states** 状态码（整数），如：200、404…

6.  **String statesText** 状态文本（字符串），如：OK、NotFound…

##### get请求

GET用于向服务器查询某些信息：

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

    <h1>XMLHttpRequest - Ajax请求</h1>
    <input type="button" onclick="XmlGetRequest();" value="Get发送请求" />

    <script src="/statics/jquery-1.12.4.js"></script>
    <script type="text/javascript">

        function GetXHR(){
            var xhr = null;
            if(XMLHttpRequest){
                xhr = new XMLHttpRequest();
            }else{
                xhr = new ActiveXObject("Microsoft.XMLHTTP");
            }
            return xhr;
        }
        function XmlGetRequest(){
            var xhr = GetXHR();
            // 定义回调函数
            xhr.onreadystatechange = function(){
                if(xhr.readyState == 4){
                    // 已经接收到全部响应数据，执行以下操作
                    var data = xhr.responseText;
                    console.log(data);
                }
            };
            // 指定连接方式和地址----文件方式
            xhr.open('get', "/test/", true);
            // 发送请求
            xhr.send();
        }
    </script>
</body>
</html>
```

##### post请求

POST请求用于向服务器发送应该被保存的数据。POST请求的主体可以包含非常多的数据，而且格式不限。

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>POST</title>
</head>
<body>

    <h1>XMLHttpRequest - Ajax请求</h1>
    <input type="button" onclick="XmlPostRequest();" value="Post发送请求" />

    <script src="/statics/jquery-1.12.4.js"></script>
    <script type="text/javascript">

        function GetXHR(){
            var xhr = null;
            if(XMLHttpRequest){
                xhr = new XMLHttpRequest();
            }else{
                xhr = new ActiveXObject("Microsoft.XMLHTTP");
            }
            return xhr;
        }
        function XmlPostRequest(){
            var xhr = GetXHR();
            // 定义回调函数
            xhr.onreadystatechange = function(){
                if(xhr.readyState == 4){
                    // 已经接收到全部响应数据，执行以下操作
                    var data = xhr.responseText;
                    console.log(data);
                }
            };
            // 指定连接方式和地址----文件方式
            xhr.open('POST', "/test/", true);
            // 设置请求头
            xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset-UTF-8');
            // 发送请求
            xhr.send('n1=1;n2=2;');
        }
    </script>
</body>
</html>
```

#### JQuery Ajax

 jQuery 提供多个与 AJAX 有关的方法。

 通过 jQuery AJAX 方法，您能够使用 HTTP Get 和 HTTP Post 从远程服务器上请求文本、HTML、XML 或 JSON – 同时您能够把这些外部数据直接载入网页的被选元素中。

-  jQuery 不是生产者，而是大自然搬运工。
-  jQuery Ajax本质 XMLHttpRequest 或 ActiveXObject

 注：2.+版本不再支持IE9以下的浏览器

```
1. jQuery.get(...)
       所有参数：
              url: 待载入页面的URL地址
             data: 待发送 Key/value 参数。
          success: 载入成功时回调函数。
         dataType: 返回内容格式，xml, json,  script, text, html

2.jQuery.post(...)
       所有参数：
              url: 待载入页面的URL地址
             data: 待发送 Key/value 参数
          success: 载入成功时回调函数
         dataType: 返回内容格式，xml, json,  script, text, html

3.jQuery.getJSON(...)
       所有参数：
              url: 待载入页面的URL地址
             data: 待发送 Key/value 参数。
          success: 载入成功时回调函数。

4.jQuery.getScript(...)
       所有参数：
              url: 待载入页面的URL地址
             data: 待发送 Key/value 参数。
          success: 载入成功时回调函数。

5.jQuery.ajax(...)
       部分参数：
              url：请求地址
             type：请求方式，GET、POST（1.9.0之后用method）
          headers：请求头
             data：要发送的数据
      contentType：即将发送信息至服务器的内容编码类型(默认: "application/x-www-form-urlencoded; charset=UTF-8")
            async：是否异步
          timeout：设置请求超时时间（毫秒）
       beforeSend：发送请求前执行的函数(全局)
         complete：完成之后执行的回调函数(全局)
          success：成功之后执行的回调函数(全局)
            error：失败之后执行的回调函数(全局)
          accepts：通过请求头发送给服务器，告诉服务器当前客户端课接受的数据类型
         dataType：将服务器端返回的数据转换成指定类型
            "xml": 将服务器端返回的内容转换成xml格式
           "text": 将服务器端返回的内容转换成普通文本格式
           "html": 将服务器端返回的内容转换成普通文本格式，在插入DOM中时，如果包含JavaScript标签，则会尝试去执行。
         "script": 尝试将返回值当作JavaScript去执行，然后再将服务器端返回的内容转换成普通文本格式
           "json": 将服务器端返回的内容转换成相应的JavaScript对象
          "jsonp": JSONP 格式使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数
```



##### $.ajax()方法

jquery中的ajax方法

**1.url**: 
要求为String类型的参数，（默认为当前页地址）发送请求的地址。

**2.type**: 
要求为String类型的参数，请求方式（post或get）默认为get。注意其他http请求方法，例如put和delete也可以使用，但仅部分浏览器支持。

**3.timeout**: 
要求为Number类型的参数，设置请求超时时间（毫秒）。此设置将覆盖$.ajaxSetup()方法的全局设置。

**4.async**: 
要求为Boolean类型的参数，默认设置为true，所有请求均为异步请求。如果需要发送同步请求，请将此选项设置为false。注意，同步请求将锁住浏览器，用户其他操作必须等待请求完成才可以执行。

**5.cache**: 
要求为Boolean类型的参数，默认为true（当dataType为script时，默认为false），设置为false将不会从浏览器缓存中加载请求信息。

**6.data**: 
要求为Object或String类型的参数，发送到服务器的数据。如果已经不是字符串，将自动转换为字符串格式。get请求中将附加在url后。防止这种自动转换，可以查看　　processData选项。对象必须为key/value格式，例如{foo1:"bar1",foo2:"bar2"}转换为&foo1=bar1&foo2=bar2。如果是数组，JQuery将自动为不同值对应同一个名称。例如{foo:["bar1","bar2"]}转换为&foo=bar1&foo=bar2。

**7. dataType**: 
要求为String类型的参数，预期服务器返回的数据类型。如果不指定，JQuery将自动根据http包mime信息返回responseXML或responseText，并作为回调函数参数传递。可用的类型如下：
xml：返回XML文档，可用JQuery处理。
html：返回纯文本HTML信息；包含的script标签会在插入DOM时执行。
script：返回纯文本JavaScript代码。不会自动缓存结果。除非设置了cache参数。注意在远程请求时（不在同一个域下），所有post请求都将转为get请求。
json：返回JSON数据。
jsonp：JSONP格式。使用SONP形式调用函数时，例如myurl?callback=?，JQuery将自动替换后一个“?”为正确的函数名，以执行回调函数。
text：返回纯文本字符串。

**8.beforeSend**：
要求为Function类型的参数，发送请求前可以修改XMLHttpRequest对象的函数，例如添加自定义HTTP头。在beforeSend中如果返回false可以取消本次ajax请求。XMLHttpRequest对象是惟一的参数。

```
  function(XMLHttpRequest){
               this;   //调用本次ajax请求时传递的options参数
            }
```

**9.complete**：
要求为Function类型的参数，请求完成后调用的回调函数（请求成功或失败时均调用）。参数：XMLHttpRequest对象和一个描述成功请求类型的字符串。
     

```
     function(XMLHttpRequest, textStatus){
             this;    //调用本次ajax请求时传递的options参数
          }
```

**10.success**：要求为Function类型的参数，请求成功后调用的回调函数，有两个参数。
         (1)由服务器返回，并根据dataType参数进行处理后的数据。
         (2)描述状态的字符串。

```
   function(data, textStatus){
            //data可能是xmlDoc、jsonObj、html、text等等
            this;  //调用本次ajax请求时传递的options参数
         }
```

**11.error**:
要求为Function类型的参数，请求失败时被调用的函数。该函数有3个参数，即XMLHttpRequest对象、错误信息、捕获的错误对象(可选)。ajax事件函数如下：

```
       function(XMLHttpRequest, textStatus, errorThrown){
          //通常情况下textStatus和errorThrown只有其中一个包含信息
          this;   //调用本次ajax请求时传递的options参数
       }
```

**12.contentType**：
要求为String类型的参数，当发送信息至服务器时，内容编码类型默认为"application/x-www-form-urlencoded"。该默认值适合大多数应用场合。

**13.dataFilter**：
要求为Function类型的参数，给Ajax返回的原始数据进行预处理的函数。提供data和type两个参数。data是Ajax返回的原始数据，type是调用jQuery.ajax时提供的dataType参数。函数返回的值将由jQuery进一步处理。

```
         function(data, type){
                //返回处理后的数据
                return data;
            }
```

**14.dataFilter**：
要求为Function类型的参数，给Ajax返回的原始数据进行预处理的函数。提供data和type两个参数。data是Ajax返回的原始数据，type是调用jQuery.ajax时提供的dataType参数。函数返回的值将由jQuery进一步处理。

```
            function(data, type){
                //返回处理后的数据
                return data;
            }
```

**15.global**：
要求为Boolean类型的参数，默认为true。表示是否触发全局ajax事件。设置为false将不会触发全局ajax事件，ajaxStart或ajaxStop可用于控制各种ajax事件。

**16.ifModified**：
要求为Boolean类型的参数，默认为false。仅在服务器数据改变时获取新数据。服务器数据改变判断的依据是Last-Modified头信息。默认值是false，即忽略头信息。

**17.jsonp**：
要求为String类型的参数，在一个jsonp请求中重写回调函数的名字。该值用来替代在"callback=?"这种GET或POST请求中URL参数里的"callback"部分，例如{jsonp:'onJsonPLoad'}会导致将"onJsonPLoad=?"传给服务器。

**18.username**：
要求为String类型的参数，用于响应HTTP访问认证请求的用户名。

**19.password**：
要求为String类型的参数，用于响应HTTP访问认证请求的密码。

**20.processData**：
要求为Boolean类型的参数，默认为true。默认情况下，发送的数据将被转换为对象（从技术角度来讲并非字符串）以配合默认内容类型"application/x-www-form-urlencoded"。如果要发送DOM树信息或者其他不希望转换的信息，请设置为false。

**21.scriptCharset**：
要求为String类型的参数，只有当请求时dataType为"jsonp"或者"script"，并且type是GET时才会用于强制修改字符集(charset)。通常在本地和远程的内容编码不同时使用。

```
$(function(){
    $('#send').click(function(){
         $.ajax({
             type: "GET",
             url: "test.json",
             data: {username:$("#username").val(), content:$("#content").val()},
             dataType: "json",
             success: function(data){
                         $('#resText').empty();   //清空resText里面的所有内容
                         var html = ''; 
                         $.each(data, function(commentIndex, comment){
                               html += '<div class="comment"><h6>' + comment['username']
                                         + ':</h6><p class="para"' + comment['content']
                                         + '</p></div>';
                         });
                         $('#resText').html(html);
                      }
         });
    });
});
```



#### 安全限制

上面代码的URL使用的是相对路径。如果你把它改为`'http://www.sina.com.cn/'`，再运行，肯定报错。在Chrome的控制台里，还可以看到错误信息。

这是因为浏览器的同源策略导致的。默认情况下，JavaScript在发送AJAX请求时，URL的域名必须和当前页面完全一致。

完全一致的意思是，域名要相同（`www.example.com`和`example.com`不同），协议要相同（`http`和`https`不同），端口号要相同（默认是`:80`端口，它和`:8080`就不同）。有的浏览器口子松一点，允许端口不同，大多数浏览器都会严格遵守这个限制。

那是不是用JavaScript无法请求外域（就是其他网站）的URL了呢？方法还是有的，大概有这么几种：

一是通过Flash插件发送HTTP请求，这种方式可以绕过浏览器的安全限制，但必须安装Flash，并且跟Flash交互。不过Flash用起来麻烦，而且现在用得也越来越少了。

二是通过在同源域名下架设一个代理服务器来转发，JavaScript负责把请求发送到代理服务器：

```
'/proxy?url=http://www.sina.com.cn'
```

代理服务器再把结果返回，这样就遵守了浏览器的同源策略。这种方式麻烦之处在于需要服务器端额外做开发。

第三种方式称为JSONP，它有个限制，只能用GET请求，并且要求返回JavaScript。这种方式跨域实际上是利用了浏览器允许跨域引用JavaScript资源：

```
<html>
<head>
    <script src="http://example.com/abc.js"></script>
    ...
</head>
<body>
...
</body>
</html>
```

JSONP通常以函数调用的形式返回，例如，返回JavaScript内容如下：

```
foo('data');
```

这样一来，我们如果在页面中先准备好`foo()`函数，然后给页面动态加一个`<script>`节点，相当于动态读取外域的JavaScript资源，最后就等着接收回调了。

以163的股票查询URL为例，对于URL：<http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice>，你将得到如下返回：

```
refreshPrice({"0000001":{"code": "0000001", ... });
```

因此我们需要首先在页面中准备好回调函数：

```
function refreshPrice(data) {
    var p = document.getElementById('test-jsonp');
    p.innerHTML = '当前价格：' +
        data['0000001'].name +': ' + 
        data['0000001'].price + '；' +
        data['1399001'].name + ': ' +
        data['1399001'].price;
}
```

最后用`getPrice()`函数触发：

```
function getPrice() {
    var
        js = document.createElement('script'),
        head = document.getElementsByTagName('head')[0];
    js.src = 'http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice';
    head.appendChild(js);
}
```

就完成了跨域加载数据。

#### CORS

如果浏览器支持HTML5，那么就可以一劳永逸地使用新的跨域策略：CORS了。

CORS全称Cross-Origin Resource Sharing，是HTML5规范定义的如何跨域访问资源。

了解CORS前，我们先搞明白概念：

Origin表示本域，也就是浏览器当前页面的域。当JavaScript向外域（如sina.com）发起请求后，浏览器收到响应后，首先检查`Access-Control-Allow-Origin`是否包含本域，如果是，则此次跨域请求成功，如果不是，则请求失败，JavaScript将无法获取到响应的任何数据。



假设本域是`my.com`，外域是`sina.com`，只要响应头`Access-Control-Allow-Origin`为`http://my.com`，或者是`*`，本次请求就可以成功。

可见，跨域能否成功，取决于对方服务器是否愿意给你设置一个正确的`Access-Control-Allow-Origin`，决定权始终在对方手中。

上面这种跨域请求，称之为“简单请求”。简单请求包括GET、HEAD和POST（POST的Content-Type类型 仅限`application/x-www-form-urlencoded`、`multipart/form-data`和`text/plain`），并且不能出现任何自定义头（例如，`X-Custom: 12345`），通常能满足90%的需求。

无论你是否需要用JavaScript通过CORS跨域请求资源，你都要了解CORS的原理。最新的浏览器全面支持HTML5。在引用外域资源时，除了JavaScript和CSS外，都要验证CORS。例如，当你引用了某个第三方CDN上的字体文件时：

```
/* CSS */
@font-face {
  font-family: 'FontAwesome';
  src: url('http://cdn.com/fonts/fontawesome.ttf') format('truetype');
}

```

如果该CDN服务商未正确设置`Access-Control-Allow-Origin`，那么浏览器无法加载字体资源。

对于PUT、DELETE以及其他类型如`application/json`的POST请求，在发送AJAX请求之前，浏览器会先发送一个`OPTIONS`请求（称为preflighted请求）到这个URL上，询问目标服务器是否接受：

```
OPTIONS /path/to/resource HTTP/1.1
Host: bar.com
Origin: http://my.com
Access-Control-Request-Method: POST
```

服务器必须响应并明确指出允许的Method：

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://my.com
Access-Control-Allow-Methods: POST, GET, PUT, OPTIONS
Access-Control-Max-Age: 86400
```

浏览器确认服务器响应的`Access-Control-Allow-Methods`头确实包含将要发送的AJAX请求的Method，才会继续发送AJAX，否则，抛出一个错误。

由于以`POST`、`PUT`方式传送JSON格式的数据在REST中很常见，所以要跨域正确处理`POST`和`PUT`请求，服务器端必须正确响应`OPTIONS`请求。

需要深入了解CORS的童鞋请移步[W3C文档](http://www.w3.org/TR/cors/)。

参考：[史上最全ajax全套讲解](https://blog.csdn.net/u013766533/article/details/52314519)