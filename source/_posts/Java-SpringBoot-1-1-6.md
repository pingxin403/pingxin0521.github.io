---
title: Spring boot Webflux
date: 2019-07-10 10:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

我们知道传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的，在Servlet3.1之后才有了异步非阻塞的支持。

而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如tomcat 7、jetty 9、Netty、Undertow及**支持Servlet3.1的容器**上，因此它的运行环境的可选择行要比传统web框架多的多。

<!--more-->

#### Reactor 框架

在 Java 生态中，提供响应式编程的框架主要有 [Reactor](https://projectreactor.io/)、[RxJava](https://github.com/ReactiveX/RxJava)、[JDK9 Flow API](https://community.oracle.com/docs/DOC-1006738) 。

Reactor 有两个非常重要的基本概念：

- [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) ，表示的是包含 0 到 N 个元素的异步序列。当消息通知产生时，订阅者([Subscriber](https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Subscriber.html))中对应的方法 `#onNext(t)`, `#onComplete(t)` 和 `#onError(t)` 会被调用。
- [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) 表示的是包含 0 或者 1 个元素的异步序列。该序列中同样可以包含与 Flux 相同的三种类型的消息通知。
- 同时，Flux 和 Mono 之间可以进行转换。例如：
  - 对一个 Flux 序列进行计数操作，得到的结果是一个 `Mono` 对象。
  - 把两个 Mono 序列合并在一起，得到的是一个 Flux 对象。

其实，可以先暂时简单把 Mono 理解成 Object ，Flux 理解成 List

#### Spring WebFlux

Spring Framework 5 提供了一个新的 [`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux) 模块。该模块包含了：

- 对响应式支持的 HTTP 和 WebSocket 客户端。
- 对响应式支持的 Web 服务器，包括 Rest API、HTML 浏览器、WebSocket 等交互方式。

根据官方的说法，webflux主要在如下两方面体现出独有的优势：

1. 非阻塞式

   其实在servlet3.1提供了非阻塞的API，WebFlux提供了一种比其更完美的解决方案。使用非阻塞的方式可以利用较小的线程或硬件资源来处理并发进而提高其可伸缩性

2. 函数式编程端点

   老生常谈的编程方式了，Spring5必须让你使用java8，那么函数式编程就是java8重要的特点之一，而WebFlux支持函数式编程来定义路由端点处理请求。

在服务端方面，WebFlux 提供了 2 种编程模型（翻译成使用方式，可能更易懂）：

- 方式一，**基于 Annotated Controller 方式实现**：基于 `@Controller` 和 SpringMVC 使用的其它注解。😈 也就是说，我们大体上可以像使用 SpringMVC 的方式，使用 WebFlux 。
- 方式二，**基于函数式编程方式**：函数式，Java 8 lambda 表达式风格的路由和处理。😈 可能有点晦涩，晚点我们看了示例就会明白。

**SpringMVC与SpringWebFlux**

我们先来看官网的一张图：

![QzmJ0J.png](https://s2.ax1x.com/2019/12/22/QzmJ0J.png)

它们都可以用注解式编程模型，都可以运行在tomcat，jetty，undertow等servlet容器当中。但是SpringMVC采用命令式编程方式，代码一句一句的执行，这样更有利于理解与调试，而WebFlux则是基于异步响应式编程，对于初次接触的码农们来说会不习惯。对于这两种框架官方给出的建议是：

1. 如果原先使用用SpringMVC好好的话，则没必要迁移。因为命令式编程是编写、理解和调试代码的最简单方法。因为老项目的类库与代码都是基于阻塞式的。
2. 如果你的团队打算使用非阻塞式web框架，WebFlux确实是一个可考虑的技术路线，而且它支持类似于SpringMvc的Annotation的方式实现编程模式，也可以在微服务架构中让WebMvc与WebFlux共用Controller，切换使用的成本相当小
3. 在SpringMVC项目里如果需要调用远程服务的话，你不妨考虑一下使用WebClient，而且方法的返回值可以考虑使用Reactive Type类型的，当每个调用的延迟时间越长，或者调用之间的相互依赖程度越高，其好处就越大

官网明确指出，SpringWebFlux并不是让你的程序运行的更快(相对于SpringMVC来说)，而是在有限的资源下提高系统的伸缩性，因此当你对响应式编程非常熟练的情况下并将其应用于新的系统中，还是值得考虑的，否则还是老老实实的使用WebMVC吧

下图显示了服务端的技术栈，左侧是 [`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/master/spring-mvc) 模块中传统的、基于 Servlet 的 Spring MVC ，右侧是 `spring-webflux` 模块中的响应式技术栈。

![QzmbAs.png](https://s2.ax1x.com/2019/12/22/QzmbAs.png)

仔细看第一层的两个框框，分别是上面提到的 WebFlux 的两种编程模型。表达的是 SpringMVC 不支持 Router Functions 方式，而 WebFlux 支持

#### 示例

使用上个文章的示例进行更改：

<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-4-WebFlux>

在 pom.xml 文件中，引入相关依赖。

```xml
       <!-- 实现对 Spring WebFlux 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```

- 引入 reactor-core 依赖，使用 Reactor 作为 WebFlux 的响应式框架的基础。
- 引入 spring-boot-starter-reactor-netty 依赖，使用 Netty 构建 WebFlux 的 Web 服务器。其中 RxNetty 库，是基于 Reactor 的响应式框架的基础之上，提供出 Netty 的响应式 API 。

当然，我们除了使用可以使用其它作为 WebFlux 的 Web 服务器，如下表格：

| Server name           | Server API used                                              | Reactive Streams support                                     |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Netty                 | Netty API                                                    | [Reactor Netty](https://github.com/reactor/reactor-netty)    |
| Undertow              | Undertow API                                                 | spring-web: Undertow to Reactive Streams bridge              |
| Tomcat                | Servlet 3.1 non-blocking I/O; Tomcat API to read and write ByteBuffers vs byte[] | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Jetty                 | Servlet 3.1 non-blocking I/O; Jetty API to write ByteBuffers vs byte[] | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Servlet 3.1 container | Servlet 3.1 non-blocking I/O                                 | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |

**配置支持WebFlux**

```
@EnableWebFlux
//在配置类或者应用类上添加
```

**基于 Annotated Controller 方式实现**

```java

@RestController
@RequestMapping("/users")
public class UserController {


    /**
     * 查询用户列表
     *
     * @return 用户列表
     */
    @GetMapping("/list")
    public Flux<UserVO> list() {
        // 查询列表
        List<UserVO> result = new ArrayList<>();
        result.add(new UserVO().setId(1).setName("yudaoyuanma"));
        result.add(new UserVO().setId(2).setName("woshiyutou"));
        result.add(new UserVO().setId(3).setName("chifanshuijiao"));
        // 返回列表
        return Flux.fromIterable(result);
    }

    /**
     * 获得指定用户编号的用户
     *
     * @param id 用户编号
     * @return 用户
     */
    @GetMapping("/get")
    public Mono<UserVO> get(@RequestParam("id") Integer id) {
        // 查询用户
        UserVO user = new UserVO().setId(id).setName("username:" + id);
        // 返回
        return Mono.just(user);
    }

    /**
     * 添加用户
     *
     * @param addDTO 添加用户信息 DTO
     * @return 添加成功的用户编号
     */
    @PostMapping("/add")
    public Mono<Integer> add(@RequestBody Publisher<UserAddDTO> addDTO) {
        // 插入用户记录，返回编号
        Integer returnId = 1;
        // 返回用户编号
        return Mono.just(returnId);
    }

    /**
     * 更新指定用户编号的用户
     *
     * @param updateDTO 更新用户信息 DTO
     * @return 是否修改成功
     */
    @PostMapping("/update")
    public Mono<Boolean> update(@RequestBody Publisher<UserUpdateDTO> updateDTO) {
        // 更新用户记录
        Boolean success = true;
        // 返回更新是否成功
        return Mono.just(success);
    }

    /**
     * 删除指定用户编号的用户
     *
     * @param id 用户编号
     * @return 是否删除成功
     */
    @PostMapping("/delete") // URL 修改成 /delete ，RequestMethod 改成 DELETE
    public Mono<Boolean> delete(@RequestParam("id") Integer id) {
        // 删除用户记录
        Boolean success = false;
        // 返回是否更新成功
        return Mono.just(success);
    }

}

```

- 在类和方法上，我们添加了 `@Controller` 和 SpringMVC 在使用的 `@GetMapping` 和 `PostMapping` 等注解，提供 API 接口，这个和我们在使用 SpringMVC 是一模一样的。

- `#list()` 方法，我们最终调用 `Flux#fromIterable(Iterable it)` 方法，将 List 包装成 Flux 对象返回。

- `#get(Integer id)` 方法，我们最终调用 `Mono#just(T data)` 方法，将 UserVO 包装成 Mono 对象返回。

- `#add(Publisher addDTO)` 方法，参数为 [Publisher](https://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Publisher.html) 类型，泛型为 UserAddDTO 类型，并且添加了 `@RequestBody` 注解，从 request 的 Body 中读取参数。注意，此时提交参数需要使用 `"application/json"` 等 Content-Type 内容类型。

- `#add(...)` 方法，也可以使用 `application/x-www-form-urlencoded` 或 `multipart/form-data` 这两个 Content-Type 内容类型，通过 request 的 Form Data 或 Multipart Data 传递参数。

  ```java
  // UserController.java
  
  /**
   * 添加用户
   *
   * @param addDTO 添加用户信息 DTO
   * @return 添加成功的用户编号
   */
  @PostMapping("/add2")
  public Mono<Integer> add(Mono<UserAddDTO> addDTO) {
      // 插入用户记录，返回编号
      Integer returnId = UUID.randomUUID().hashCode();
      // 返回用户编号
      return Mono.just(returnId);
  }
  ```

**基于函数式编程方式**

```java
import com.hyp.learn.wf.vo.UserVO;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;
import org.springframework.web.reactive.function.server.*;
import reactor.core.publisher.Mono;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

/**
 * @author hyp
 * Project name is spring-boot-learn
 * Include in com.hyp.learn.wf.config
 * hyp create at 19-12-22
 **/
@Configuration
public class UserRouter {

    @Bean
    public RouterFunction<ServerResponse> userListRouterFunction() {
        return route(GET("/users2/list"),
                new HandlerFunction<ServerResponse>() {

                    @Override
                    public Mono<ServerResponse> handle(ServerRequest request) {
                        // 查询列表
                        List<UserVO> result = new ArrayList<>();
                        result.add(new UserVO().setId(1).setName("yudaoyuanma"));
                        result.add(new UserVO().setId(2).setName("woshiyutou"));
                        result.add(new UserVO().setId(3).setName("chifanshuijiao"));
                        // 返回列表
                        return ok().bodyValue(result);
                    }

                });
    }

    @Bean
    public RouterFunction<ServerResponse> userGetRouterFunction() {
        return route(GET("/users2/get"),
                new HandlerFunction<ServerResponse>() {

                    @Override
                    public Mono<ServerResponse> handle(ServerRequest request) {
                        // 获得编号
                        Integer id = request.queryParam("id")
                                .map(s -> StringUtils.isEmpty(s) ? null : Integer.valueOf(s)).get();
                        // 查询用户
                        UserVO user = new UserVO().setId(id).setName(UUID.randomUUID().toString());
                        // 返回列表
                        return ok().bodyValue(user);
                    }

                });
    }

    @Bean
    public RouterFunction<ServerResponse> demoRouterFunction() {
        return route(GET("/users2/demo"), request -> ok().bodyValue("demo"));
    }

}
```

- 在类上，添加 `@Configuration` 注解，保证该类中的 Bean 们，都被扫描到。
- 在每个方法中，我们都通弄 [`RouterFunctions#route(RequestPredicate predicate, HandlerFunction handlerFunction)`](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RouterFunctions.java) 方法，定义了一条路由。
  - 第一个参数 `predicate` 参数，是 [RequestPredicate](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RequestPredicate.java) 类型，请求谓语，用于匹配请求。可以通过 [RequestPredicates](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RequestPredicates.java) 来构建各种条件。
  - 第二个参数 `handlerFunction` 参数，是 [RouterFunction](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/main/java/org/springframework/web/reactive/function/server/RouterFunction.java) 类型，处理器函数。
- 每个方法定义的路由，胖友自己看下代码，一眼能看的明白。一般来说，采用第三个方法的写法，更加简洁。

推荐**基于 Annotated Controller 方式实现**的编程方式，更符合我们现在的开发习惯，学习成本也相对低一些。同时，和 API 接口文档工具 Swagger 也更容易集成。

#### 测试接口

**集成测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = WebFluxApplication.class)
@AutoConfigureWebFlux
@AutoConfigureWebTestClient
public class UserControllerTest {

    @Autowired
    private WebTestClient webClient;

    @Test
    public void testList() {
        webClient.get().uri("/users/list")
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("[\n" +
                "    {\n" +
                "        \"id\": 1,\n" +
                "        \"username\": \"yudaoyuanma\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 2,\n" +
                "        \"username\": \"woshiyutou\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 3,\n" +
                "        \"username\": \"chifanshuijiao\"\n" +
                "    }\n" +
                "]"); // 响应结果
    }

    @Test
    public void testGet() {
        // 获得指定用户编号的用户
        webClient.get().uri("/users/get?id=1")
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("{\n" +
                "    \"id\": 1,\n" +
                "    \"username\": \"username:1\"\n" +
                "}"); // 响应结果
    }

    @Test
    public void testGet2() {
        // 获得指定用户编号的用户
        webClient.get().uri("/users/v2/get?id=1")
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("{\n" +
                "    \"id\": 1,\n" +
                "    \"username\": \"test\"\n" +
                "}"); // 响应结果
    }

    @Test
    public void testAdd() {
        Map<String, Object> params = new HashMap<>();
        params.put("username", "yudaoyuanma");
        params.put("password", "nicai");
        // 添加用户
        webClient.post().uri("/users/add")
                .bodyValue(params)
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("1"); // 响应结果。因为没有提供 content 的比较，所以只好使用 json 来比较。竟然能通过
    }

    @Test
    public void testAdd2() { // 发送文件的测试，可以参考 https://dev.to/shavz/sending-multipart-form-data-using-spring-webtestclient-2gb7 文章
        BodyInserters.FormInserter<String> formData = // Form Data 数据，需要这么拼凑
                BodyInserters.fromFormData("username", "yudaoyuanma")
                        .with("password", "nicai");
        // 添加用户
        webClient.post().uri("/users/add2")
                .body(formData)
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("1"); // 响应结果。因为没有提供 content 的比较，所以只好使用 json 来比较。竟然能通过
    }


    @Test
    public void testUpdate() {
        Map<String, Object> params = new HashMap<>();
        params.put("id", 1);
        params.put("username", "yudaoyuanma");
        // 修改用户
        webClient.post().uri("/users/update")
                .bodyValue(params)
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody(Boolean.class) // 期望返回值类型是 Boolean
                .consumeWith((Consumer<EntityExchangeResult<Boolean>>) result -> // 通过消费结果，判断符合是 true 。
                        Assert.assertTrue("返回结果需要为 true", result.getResponseBody()));
    }

    @Test
    public void testDelete() {
        // 删除用户
        webClient.post().uri("/users/delete?id=1")
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody(Boolean.class) // 期望返回值类型是 Boolean
                .isEqualTo(true); // 这样更加简洁一些
//                .consumeWith((Consumer<EntityExchangeResult<Boolean>>) result -> // 通过消费结果，判断符合是 true 。
//                        Assert.assertTrue("返回结果需要为 true", result.getResponseBody()));
    }

}
```

- 在类上，我们添加了 `@AutoConfigureWebTestClient` 注解，用于自动化配置我们稍后注入的 WebTestClient Bean 对象 `webClient` 。在后续的测试中，我们会看到都是通过 `webClient` 调用后端 API 接口。而每一次调用后端 API 接口，都会执行**真正的后端逻辑**。因此，整个逻辑，走的是**集成测试**，会启动一个**真实**的 Spring 环境。
- 每次 API 接口的请求，都通过 [RequestHeadersSpec](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.RequestHeadersSpec.html) 来构建。构建完成后，通过 `RequestHeadersSpec#exchange()` 方法来执行请求，返回 [ResponseSpec](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.ResponseSpec.html) 结果。
  - WebTestClient 的 `#get()`、`#head()`、`#delete()`、`#options()` 方法，返回的是 [RequestHeadersUriSpec](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.RequestHeadersUriSpec.html) 对象。
  - WebTestClient 的 `#post()`、`#put()`、`#delete()`、`#patch()` 方法，返回的是 [RequestBodyUriSpec](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.RequestBodyUriSpec.html) 对象。
  - RequestHeadersUriSpec 和 RequestBodyUriSpec 都继承了 RequestHeadersSpec 接口。
- 执行完请求后，通过调用 [RequestBodyUriSpec](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.RequestBodyUriSpec.html) 的各种断言方法，添加对结果的预期，相当于做断言。如果不符合预期，则会抛出异常，测试不通过。

**单元测试**

```java

@Service
public class UserService {

    public UserVO get(Integer id) {
        return new UserVO().setId(id).setUsername("test");
    }

}

//在 UserController 类中，增加 GET /users/v2/get 接口，获得指定用户编号的用户。代码如下：
// UserController.java

@Autowired
private UserService userService;

/**
 * 获得指定用户编号的用户
 *
 * @param id 用户编号
 * @return 用户
 */
@GetMapping("/v2/get")
public Mono<UserVO> get2(@RequestParam("id") Integer id) {
    // 查询用户
    UserVO user = userService.get(id);
    // 返回
    return Mono.just(user);
}

//创建 UserControllerTest2 测试类，我们来测试一下简单的 UserController 的新增的这个 API 操作。代码如下：

// UserControllerTest2.java

@RunWith(SpringRunner.class)
@WebFluxTest(UserController.class)
public class UserControllerTest2 {

    @Autowired
    private WebTestClient webClient;

    @MockBean
    private UserService userService;

    @Test
    public void testGet2() throws Exception {
        // Mock UserService 的 get 方法
        System.out.println("before mock:" + userService.get(1)); // <1.1>
        Mockito.when(userService.get(1)).thenReturn(
                new UserVO().setId(1).setUsername("username:1")); // <1.2>
        System.out.println("after mock:" + userService.get(1)); // <1.3>

        // 查询用户列表
        webClient.get().uri("/users/v2/get?id=1")
                .exchange() // 执行请求
                .expectStatus().isOk() // 响应状态码 200
                .expectBody().json("{\n" +
                "    \"id\": 1,\n" +
                "    \"username\": \"username:1\"\n" +
                "}"); // 响应结果
    }

}
```

- 在类上添加 @WebFluxTest 注解，并且传入的是 UserController 类，表示我们要对 UserController 进行单元测试。
- 同时，`@WebFluxTest` 注解，是包含了 `@UserController` 的组合注解，所以它会自动化配置我们稍后注入的 WebTestClient Bean 对象 `mvc` 。在后续的测试中，我们会看到都是通过 `webClient` 调用后端 API 接口。**但是**！每一次调用后端 API 接口，并**不会**执行**真正的后端逻辑**，而是走的 Mock 逻辑。也就是说，整个逻辑，走的是**单元测试**，**只**会启动一个 **Mock** 的 Spring 环境。

#### 全局统一返回

上章对spring MVC情况下的全局统一返回进行处理，这里根据那个代码进行修改

<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-4-WebFlux>

采用 `ControllerAdvice` + `@ExceptionHandler` 注解的方式，可以很方便的实现 WebFlux 的全局异常处理。不过这种方案存在一个弊端，不支持 WebFlux 的基于函数式编程方式。不过考虑到，绝大多数情况下，我们并不会采用基于函数式编程方式，所以这种方案还是没问题的。

如果真的需要支持 WebFlux 的基于函数式编程方式，可以看看 [《Handling Errors in Spring WebFlux》](https://www.baeldung.com/spring-webflux-errors) 文章，通过继承 `org.springframework.boot.autoconfigure.web.reactive.error.AbstractErrorWebExceptionHandler` 抽象类，实现自定义的全局异常处理器。

### WebFilter 过滤器

在 WebFlux 中，我们可以通过实现 WebFilter 接口，过滤 WebFlux 处理请求的过程，自定义前置和处理的逻辑。该接口代码如下：

```
public interface WebFilter {

	Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);

}
```

- 因为 [WebFilterChain](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/server/WebFilterChain.java) 的 `#filter(ServerWebExchange exchange)` 方法，返回的是 `Mono` 对象，所以可以进行各种 Reactor 的操作。咳咳咳，当然需要胖友比较了解 Reactor 的使用，我们才能实现出的 WebFilter ，否则会觉得挺难用的。
- 另外，WebFilterChain 是由多个 WebFilter 过滤器组成的链，其默认的实现为 [DefaultWebFilterChain](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/server/handler/DefaultWebFilterChain.java) 。
- 总体来说，从形态上和我们在 Servlet 看到的 [FilterChain](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/FilterChain.java) 和 [Filter](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/Filter.java) 是比较相似的，只是因为结合了 Reactor 响应式编程，所以编写时，差异蛮大的。





### 参考

1. [Spring Boot 响应式 WebFlux 入门](http://www.iocoder.cn/Spring-Boot/WebFlux/?self)