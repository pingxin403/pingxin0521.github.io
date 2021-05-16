---
title: Java 常见异常和处理方法(一)
date: 2019-08-07 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

1. java.lang.UnsupportedOperationException: A TupleBackedMap cannot be modified.
    报错类似的代码：
    
    <!--more-->

  ```java
  List<HashMap<String, Object>> newMapList = new ArrayList<>();
  for (int i = 0; i < mapList.size(); i++) {
            Map<String, Object> one = mapList.get(i);
           one.put("something",1);
          //我发现当我往这个one里面添加数据的时候就会报上面的错
  }
  ```

  解决方案：

  ```java
  List<HashMap<String, Object>> newMapList = new ArrayList<>();
  for (int i = 0; i < mapList.size(); i++) {
        Map<String, Object> one = mapList.get(i);
        HashMap<String, Object> newOne = new HashMap<>();
        newOne.putAll(one);
        newOne.put("something",1);
          //重新定义一个新的map,把one放入到newOne里面就好了
  }
  ```

2. java.lang.NullPointerException: null

   下面这样如果是其它情况会返回一个null是不允许的

   ```java
     @Override
       public List<String> moduleIdListByUserId(String userId) {
           List<String> projectIdList = getProjectIdList(userId);
           if (projectIdList.size()>0) {
              return  getModuleIdList(projectIdList);
           }
           return null;
       }
   ```

   应该改为下面这样，首先new一个，即使没有值，也返回

   ```java
     @Override
       public List<String> moduleIdListByUserId(String userId) {
           List<String> moduleIdList =new ArrayList<>();
           List<String> projectIdList = getProjectIdList(userId);
           if (projectIdList.size()>0) {
               moduleIdList= getModuleIdList(projectIdList);
           }
           return moduleIdList;
       }
   
   ```

3. mysql 把一个时间字段置NULL报错，但是结果不影响

   ```
   [Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column ‘information_schema.PROFILING.SEQ’ which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
   ```

4. JPA does not define an IdClass

   由于之前是单一主键，但是换了联合主键之后，就抛出了异常
   解决方案：

   在类的注解里添加@IdClass(xxx.class)

#### [六种异常处理的陋习](http://www.blogjava.net/freeman1984/archive/2007/09/27/148850.html)

你觉得自己是一个Java专家吗？是否肯定自己已经全面掌握了Java的异常处理机制？在下面这段代码中，你能够迅速找出异常处理的六个问题吗？

```java
OutputStreamWriter out = ...
java.sql.Connection conn = ...
try { // ⑸
 Statement stat = conn.createStatement();
 ResultSet rs = stat.executeQuery(
 "select uid, name from user");
while (rs.next())
{
 out.println("ID：" + rs.getString("uid") // ⑹
 "，姓名：" + rs.getString("name"));
}
  conn.close(); // ⑶
  out.close();
}
catch(Exception ex) // ⑵
{
  ex.printStackTrace(); //⑴，⑷
}
```

作为一个Java程序员，你至少应该能够找出两个问题。但是，如果你不能找出全部六个问题，请继续阅读本文。

**反例之一：丢弃异常**

代码：15行-18行。

这段代码捕获了异常却不作任何处理，可以算得上Java编程中的杀手。从问题出现的频繁程度和祸害程度来看，它也许可以和C/C++程序的一个恶名远播的问题相提并论??不检查缓冲区是否已满。如果你看到了这种丢弃（而不是抛出）异常的情况，可以百分之九十九地肯定代码存在问题（在极少数情况下，这段代码有存在的理由，但最好加上完整的注释，以免引起别人误解）。

这段代码的错误在于，异常（几乎）总是意味着某些事情不对劲了，或者说至少发生了某些不寻常的事情，我们不应该对程序发出的求救信号保持沉默和无动于衷。调用一下printStackTrace算不上“处理异常”。不错，调用printStackTrace对调试程序有帮助，但程序调试阶段结束之后，printStackTrace就不应再在异常处理模块中担负主要责任了。

丢弃异常的情形非常普遍。打开JDK的ThreadDeath类的文档，可以看到下面这段说明：“特别地，虽然出现ThreadDeath是一种‘正常的情形’，但ThreadDeath类是Error而不是Exception的子类，因为许多应用会捕获所有的Exception然后丢弃它不再理睬。”这段话的意思是，虽然ThreadDeath代表的是一种普通的问题，但鉴于许多应用会试图捕获所有异常然后不予以适当的处理，所以JDK把ThreadDeath定义成了Error的子类，因为Error类代表的是一般的应用不应该去捕获的严重问题。可见，丢弃异常这一坏习惯是如此常见，它甚至已经影响到了Java本身的设计。

那么，应该怎样改正呢？主要有四个选择：

1、处理异常。针对该异常采取一些行动，例如修正问题、提醒某个人或进行其他一些处理，要根据具体的情形确定应该采取的动作。再次说明，调用printStackTrace算不上已经“处理好了异常”。

2、重新抛出异常。处理异常的代码在分析异常之后，认为自己不能处理它，重新抛出异常也不失为一种选择。

3、把该异常转换成另一种异常。大多数情况下，这是指把一个低级的异常转换成应用级的异常（其含义更容易被用户了解的异常）。

4、不要捕获异常。

结论一：既然捕获了异常，就要对它进行适当的处理。不要捕获异常之后又把它丢弃，不予理睬。

**反例之二：不指定具体的异常**

代码：15行。

许多时候人们会被这样一种“美妙的”想法吸引：用一个catch语句捕获所有的异常。最常见的情形就是使用catch(Exception ex)语句。但实际上，在绝大多数情况下，这种做法不值得提倡。为什么呢？

要理解其原因，我们必须回顾一下catch语句的用途。catch语句表示我们预期会出现某种异常，而且希望能够处理该异常。异常类的作用就是告诉Java编译器我们想要处理的是哪一种异常。由于绝大多数异常都直接或间接从java.lang.Exception派生，catch(Exception ex)就相当于说我们想要处理几乎所有的异常。

再来看看前面的代码例子。我们真正想要捕获的异常是什么呢？最明显的一个是SQLException，这是JDBC操作中常见的异常。另一个可能的异常是IOException，因为它要操作OutputStreamWriter。显然，在同一个catch块中处理这两种截然不同的异常是不合适的。如果用两个catch块分别捕获SQLException和IOException就要好多了。这就是说，catch语句应当尽量指定具体的异常类型，而不应该指定涵盖范围太广的Exception类。

另一方面，除了这两个特定的异常，还有其他许多异常也可能出现。例如，如果由于某种原因，executeQuery返回了null，该怎么办？答案是让它们继续抛出，即不必捕获也不必处理。实际上，我们不能也不应该去捕获可能出现的所有异常，程序的其他地方还有捕获异常的机会??直至最后由JVM处理。

结论二：在catch语句中尽可能指定具体的异常类型，必要时使用多个catch。不要试图处理所有可能出现的异常。

**反例之三：占用资源不释放**

代码：3行-14行。

异常改变了程序正常的执行流程。这个道理虽然简单，却常常被人们忽视。如果程序用到了文件、Socket、JDBC连接之类的资源，即使遇到了异常，也要正确释放占用的资源。为此，Java提供了一个简化这类操作的关键词finally。

finally是样好东西：不管是否出现了异常，Finally保证在try/catch/finally块结束之前，执行清理任务的代码总是有机会执行。遗憾的是有些人却不习惯使用finally。

当然，编写finally块应当多加小心，特别是要注意在finally块之内抛出的异常??这是执行清理任务的最后机会，尽量不要再有难以处理的错误。

结论三：保证所有资源都被正确释放。充分运用finally关键词。

**反例之四：不说明异常的详细信息**　　

代码：3行-18行。

仔细观察这段代码：如果循环内部出现了异常，会发生什么事情？我们可以得到足够的信息判断循环内部出错的原因吗？不能。我们只能知道当前正在处理的类发生了某种错误，但却不能获得任何信息判断导致当前错误的原因。

printStackTrace的堆栈跟踪功能显示出程序运行到当前类的执行流程，但只提供了一些最基本的信息，未能说明实际导致错误的原因，同时也不易解读。

因此，在出现异常时，最好能够提供一些文字信息，例如当前正在执行的类、方法和其他状态信息，包括以一种更适合阅读的方式整理和组织printStackTrace提供的信息。

结论四：在异常处理模块中提供适量的错误原因信息，组织错误信息使其易于理解和阅读。

**反例之五：过于庞大的try块**

代码：3行-14行。

经常可以看到有人把大量的代码放入单个try块，实际上这不是好习惯。这种现象之所以常见，原因就在于有些人图省事，不愿花时间分析一大块代码中哪几行代码会抛出异常、异常的具体类型是什么。把大量的语句装入单个巨大的try块就象是出门旅游时把所有日常用品塞入一个大箱子，虽然东西是带上了，但要找出来可不容易。

一些新手常常把大量的代码放入单个try块，然后再在catch语句中声明Exception，而不是分离各个可能出现异常的段落并分别捕获其异常。这种做法为分析程序抛出异常的原因带来了困难，因为一大段代码中有太多的地方可能抛出Exception。

结论五：尽量减小try块的体积。

**反例之六：输出数据不完整**

代码：7行-11行。

不完整的数据是Java程序的隐形杀手。仔细观察这段代码，考虑一下如果循环的中间抛出了异常，会发生什么事情。循环的执行当然是要被打断的，其次，catch块会执行??就这些，再也没有其他动作了。已经输出的数据怎么办？使用这些数据的人或设备将收到一份不完整的（因而也是错误的）数据，却得不到任何有关这份数据是否完整的提示。对于有些系统来说，数据不完整可能比系统停止运行带来更大的损失。

较为理想的处置办法是向输出设备写一些信息，声明数据的不完整性；另一种可能有效的办法是，先缓冲要输出的数据，准备好全部数据之后再一次性输出。

结论六：全面考虑可能出现的异常以及这些异常对执行流程的影响。

**改写后的代码**

根据上面的讨论，下面给出改写后的代码。也许有人会说它稍微有点?嗦，但是它有了比较完备的异常处理机制。

```
OutputStreamWriter out = ...
java.sql.Connection conn = ...
try {
　Statement stat = conn.createStatement();
　ResultSet rs = stat.executeQuery(
　　"select uid, name from user");
　while (rs.next())
　{
　　out.println("ID：" + rs.getString("uid") + "，姓名: " + rs.getString("name"));
　}
}
catch(SQLException sqlex)
{
　out.println("警告：数据不完整");
　throw new ApplicationException("读取数据时出现SQL错误", sqlex);
}
catch(IOException ioex)
{
　throw new ApplicationException("写入数据时出现IO错误", ioex);
}
finally
{
　if (conn != null) {
　　try {
　　　conn.close();
　　}
　　catch(SQLException sqlex2)
　　{
　　　System.err(this.getClass().getName() + ".mymethod - 不能关闭数据库连接: " + sqlex2.toString());
　　}
　}

　if (out != null) {
　　try {
　　　out.close();
　　}
　　catch(IOException ioex2)
　　{
　　　System.err(this.getClass().getName() + ".mymethod - 不能关闭输出文件" + ioex2.toString());
　　}
　}
}
```

本文的结论不是放之四海皆准的教条，有时常识和经验才是最好的老师。如果你对自己的做法没有百分之百的信心，务必加上详细、全面的注释。

另一方面，不要笑话这些错误，不妨问问你自己是否真地彻底摆脱了这些坏习惯。即使最有经验的程序员偶尔也会误入歧途，原因很简单，因为它们确确实实带来了“方便”。所有这些反例都可以看作Java编程世界的恶魔，它们美丽动人，无孔不入，时刻诱惑着你。也许有人会认为这些都属于鸡皮蒜毛的小事，不足挂齿，但请记住：勿以恶小而为之，勿以善小而不为。

#### 解决springboot配置@ControllerAdvice不能捕获NoHandlerFoundException问题 

使用springboot开发一个RESTful API服务，配置了@ControllerAdvice，其它类型异常都能正常捕获，就是不能捕获NoHandlerFoundException，安装以往使用springmvc的经验，需要设置DispatcherServlet.throwExceptionIfNoHandlerFound，NoHandlerFoundException就会被DispatcherSevlet抛出，并被@ControllerAdvice捕获处理。想来springboot中自然也是可以的。

网上一搜发现，只需设置spring.mvc.throw-exception-if-no-handler-found=true即可。设置后依然无效！
再次搜索，还需要设置spring.resources.add-mappings=false，问题解决！

很奇怪，为什么禁用了资源映射后，问题就解决了呢？

研究DispatcherServlet源码发现，NoHandlerFoundException异常能否被抛出，关键在如下代码：

```
mappedHandler = getHandler(processedRequest);
if (mappedHandler == null || mappedHandler.getHandler() == null) {
	noHandlerFound(processedRequest, response);
	return;
}
```

只有第一句代码找不到对应该请求的处理器时，才会进入下面的noHandler方法去抛出NoHandlerFoundException异常。
通过测试发现，springboot的WebMvcAutoConfiguration会默认配置如下资源映射：

> /**映射到/static（或/public、/resources、/META-INF/resources） /webjars/** 映射到classpath:/META-INF/resources/webjars/ /**/favicon.ico映射favicon.ico文件.

这下就明白了，即使你的地址错误，仍然会匹配到/**这个静态资源映射地址，就不会进入noHandlerFound方法，自然不会抛出NoHandlerFoundException了。
所以，我们需要的就是改掉默认的静态资源映射访问路径就可以了。
配置如下属性，NoHandlerFoundException异常就能被@ControllerAdvice捕获了

```
spring.mvc.throw-exception-if-no-handler-found=true
spring.mvc.static-path-pattern=/statics/**
```

附上@ControllerAdvice代码：

```
/**
 * ResponseEntityExceptionHandler预提供了一个处理Spring常见异常的Exceptionhandler
 * @author wangyongjun
 * @date 2018/4/19.
 */
@RestControllerAdvice
public class GlobalControllerExceptionHandler extends ResponseEntityExceptionHandler {

  /**
   * 覆盖handleExceptionInternal这个汇总处理方法，将响应数据替换为我们的{@link ApiErrorResponse}即可
   * @param ex
   * @param body
   * @param headers
   * @param status
   * @param request
   * @return
   */
  @Override
  protected ResponseEntity<Object> handleExceptionInternal(Exception ex, Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {

    return new ResponseEntity<>(new ApiErrorResponse().setCode(String.valueOf(status.value())).setMsg(ex.getMessage()), status);
  }

  /**
   * 400错误，bad request
   * @param ex
   * @return
   */
  @ExceptionHandler(value = { IllegalArgumentException.class,Exception400.class})
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public ApiErrorResponse badRequestException(Exception ex) {
    return ApiErrorResponse.error400().setMsg(ex.getMessage());
  }

  /**
   * 401错误，未授权
   * @param ex
   * @return
   */
  @ExceptionHandler(value = { Exception401.class})
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public ApiErrorResponse unauthorizedException(Exception ex) {
    return ApiErrorResponse.error401().setMsg(ex.getMessage());
  }

  /**
   * 403错误，权限不足
   * @param ex
   * @return
   */
  @ExceptionHandler(value = { Exception403.class})
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public ApiErrorResponse forbiddenException(Exception ex) {
    return ApiErrorResponse.error403().setMsg(ex.getMessage());
  }

  /**
   * 404错误，处理器不存在异常，资源不存在异常
   * @param ex
   * @return
   */
  @ExceptionHandler(value = { Exception404.class })
  @ResponseStatus(HttpStatus.NOT_FOUND)
  public ApiErrorResponse noHandlerFoundException(Exception ex) {
    return ApiErrorResponse.error404().setMsg(ex.getMessage());
  }

  /**
   * 500异常
   * @param ex
   * @return
   */
  @ExceptionHandler(value = {Exception500.class,Exception.class })
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  public ApiErrorResponse exception(Exception ex) {

    return ApiErrorResponse.error500().setMsg(ex.getMessage());
  }
  
}
```

我使用的是@RestControllerAdvice，异常处理的返回都会被解析为json，不做RESTful API的使用@ControllerAdvice即可。
spring预提供了一个ResponseEntityExceptionHandler，能处理大部分spring自己定义的异常，我们只需要覆盖handleExceptionInternal这个汇总处理方法，将返回的ResponseEntity的body部分替换为我们自己定义的ApiErrorResponse即可。
当然，你也可以自由地配置对其它异常的处理。

#### SpringBoot使用Swagger2出现Unable to infer base url

在使用SpringBoot中配置Swagger2的时候，出现

> Unable to infer base url. This is common when using dynamic servlet registration or when the API is behind an API Gateway. The base url is the root of where all the swagger resources are served. For e.g. if the api is available at http://example.org/api/v2/api-docs then the base url is http://example.org/api/. Please enter the location manually:

原因:

1. 需要在SpringBoot的启动Application前面加上 @EnableSwagger2注解；

2. 可能是由于使用了Spring Security 影响：尝试使用以下Spring Security配置解决：

   ```
   import org.springframework.context.annotation.Configuration;
   import org.springframework.security.config.annotation.web.builders.HttpSecurity;
   import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
    
   @Configuration
   class SecurityConfig extends WebSecurityConfigurerAdapter {
    
       private static final String[] AUTH_WHITELIST = {
    
               // -- swagger ui
               "/swagger-resources/**",
               "/swagger-ui.html",
               "/v2/api-docs",
               "/webjars/**"
       };
       
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests()
                   .antMatchers(AUTH_WHITELIST).permitAll()
                   .antMatchers("/**/*").denyAll();
       }
   }
   ```

3. 配置在修改后再次刷新需要清空浏览器缓存，否则配置可能无法生效。如果使用chrome内核的浏览器，可以长按刷新键清空缓存访问

4. 但是在我的项目中，发现是我使用Spring的ResponseBodyAdvice全局返回再处理的一个类，本意是为所有返回JSON数据统一添加“状态=succuess”等信息，没想到该实现影响了Swagger的使用，会导致swagger返回的JSON数据格式和期望的不一致，故swagger报错。解决方法在该接口实现类上面的@ControllerAdvice 注解限制接口的扫描包即可避免。

   ```
   @ControllerAdvice(basePackages = "com.xxx.controller")
   ```

5. 在排查该问题时，假设你的swagger-ui访问路径是http://localhost:8080/swagger-ui.html，可以先直接访问 http://localhost:8080/v2/api-docs，查看swagger是否正确获取到了JSON格式的数据，且JSON格式数据是否为类似以下格式，如果不对则可能是因为别的返回预处理接口对数据进行了处理，导致swagger无法获取到正确的数据。

   ```
   {
   	"swagger": "2.0",
   	"info": {
   		"description": "My project for swagger practice",
   		"version": "v1.1",
   		"title": "swagger-demo",
   		"license": {
   			"url": "http://www.apache.org/licenses/LICENSE-2.0"
   		}
   	},
   	"host": "http://localhost:8080",
   	"basePath": "/"
   }
   ```


#### java.io.IOException: Broken pipe

![image.png](https://i.loli.net/2021/04/01/EtWrbYF7i29pAko.png)

注：读懂下面这句话，首先要熟悉TCP 四次挥手

![image.png](https://i.loli.net/2021/04/01/zdjBYfeFpIMR3GD.png)

总结 Broken Pipe：

 **这个异常是客户端读取超时关闭了连接,这时候服务器端再向客户端已经断开的连接写数据时就发生了broken pipe异常！**

### java tcp/ip异常

1.  java.net.SocketTimeoutException

   这 个异 常比较常见，socket 超时。一般有 2 个地方会抛出这个，一个是 connect 的 时 候 ， 这 个 超 时 参 数 由connect(SocketAddress endpoint,int timeout) 中的后者来决定，还有就是 setSoTimeout(int timeout)，这个是设定读取的超时时间。它们设置成 0 均表示无限大。

2. java.net.BindException:Address already in use: JVM_Bind

   该 异 常 发 生 在 服 务 器 端 进 行 new ServerSocket(port) 或者 socket.bind(SocketAddress bindpoint)操作时。

   原因:与 port 一样的一个端口已经被启动，并进行监听。此时用 netstat –an 命令，可以看到一个 Listending 状态的端口。只需要找一个没有被占用的端口就能解决这个问题。

3. java.net.ConnectException: Connection refused: connect

   该异常发生在客户端进行 new Socket(ip, port)或者 socket.connect(address,timeout)操作时，原 因:指定 ip 地址的机器不能找到（也就是说从当前机器不存在到指定 ip 路由），或者是该 ip 存在，但找不到指定的端口进行监听。应该首先检查客户端的 ip 和 port是否写错了，假如正确则从客户端 ping 一下服务器看是否能 ping 通，假如能 ping 通（服务服务器端把 ping 禁掉则需要另外的办法），则 看在服务器端的监听指定端口的程序是否启动。

4. java.net.SocketException: Socket is closed 

   该异常在客户端和服务器均可能发生。异常的原因是己方主动关闭了连接后（调用了 Socket 的 close 方法）再对网络连接进行读写操作。

5. java.net.SocketException: Connection reset 或者Connect reset by peer:Socket write error

   该异常在客户端和服务器端均有可能发生，引起该异常的原因有两个，第一个就是假如一端的 Socket 被关闭（或主动关闭或者因为异常退出而引起的关闭）， 另一端仍发送数据，发送的第一个数据包引发该异常(Connect reset by peer)。另一个是一端退出，但退出时并未关闭该连接，另 一 端 假 如 在 从 连 接 中 读 数 据 则 抛 出 该 异 常（Connection reset）。简单的说就是在连接断开后的读和写操作引起的。

   还有一种情况，如果一端发送RST数据包中断了TCP连接，另外一端也会出现这个异常，如果是tomcat，异常如下：

   org.apache.catalina.connector.ClientAbortException: java.io.IOException: Connection reset by peer

    阿里的tcp方式的健康检查为了提高性能，省去挥手交互，直接发送一个RST来终断连接，就会导致服务器端出现这个异常；

   对于服务器，一般的原因可以认为：

   a) 服务器的并发连接数超过了其承载量，服务器会将其中一些连接主动 Down 掉.

   b) 在数据传输的过程中，浏览器或者接收客户端关闭了，而服务端还在向客户端发送数据。

6. java.net.SocketException: Too many open files

   原因: 操作系统的中打开文件的最大句柄数受限所致，常常发生在很多个并发用户访问服务器的时候。因为为了执行每个用户的应用服务器都要加载很多文件(new 一个socket 就需要一个文件句柄)，这就会导致打开文件的句柄的缺乏。

   解决方式：

   a) 尽量把类打成 jar 包，因为一个 jar 包只消耗一个文件句柄，如果不打包，一个类就消耗一个文件句柄。

   b) java 的 GC 不能关闭网络连接打开的文件句柄，如果没有执行 close()则文件句柄将一直存在，而不能被关闭。

   也可以考虑设置 socket 的最大打开 数来控制这个问题。对操作系统做相关的设置，增加最大文件句柄数量。

   ulimit -a 可以查看系统目前资源限制，ulimit -n 10240 则可以修改，这个修改只对当前窗口有效。

7. java.net.SocketException: Broken pipe

   该异常在客户端和服务器均有可能发生。在抛出SocketExcepton:Connect reset by peer:Socket write error 后，假如再继续写数据则抛出该异常。前两个异常的解决方法是首先确保程序退出前关闭所有的网络连接，其次是要检测对方的关闭连接操作，发现对方 关闭连接后自己也要关闭该连接。

   对于 4 和 5 这两种情况的异常，需要特别注意连接的维护。在短连接情况下还好，如果是长连接情况，对于连接状态的维护不当，则非常容易出现异常。基本上对长连接需要做的就是：

   a) 检测对方的主动断连（对方调用了 Socket 的 close 方法）。因为对方主动断连，另一方如果在进行读操作，则此时的返回值是-1。所以一旦检测到对方断连，则主动关闭己方的连接（调用 Socket 的 close 方法）。

   b) 检测对方的宕机、异常退出及网络不通,一般做法都是心跳检测。双方周期性的发送数据给对方，同时也从对方接收“心跳数据”，如果连续几个周期都没有收到 对方心跳，则可以判断对方或者宕机或者异常退出或者网络不通，此时也需要主动关闭己方连接；如果是客户端可在延迟一定时间后重新发起连接。虽然 Socket 有一个keep alive 选项来维护连接，如果用该选项，一般需要两个小时才能发现对方的宕机、异常退出及网络不通。

8. Cannot assign requested address    

   a) 端口号被占用,导致地址无法绑定：

   java.net.BindException: Cannot assign requested address: bind：是由于IP地址变化导致的；

   b) 服务器网络配置异常：

   /etc/hosts  中配置的地址错误;

   c) 还有一种情况是执行ipconfig 发现没有环路地址，这是因为环路地址配置文件丢失了