---
title: Spring Cloud组件之Feign--声明式REST调用 (四)
date: 2019-11-1 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part3_feign>

### 声明式REST调用 - 基于Feign

[Feign](https://github.com/OpenFeign/feign)是一个声明式的Web Service客户端，它使得编写Web Serivce客户端变得更加简单。我们只需要使用Feign来创建一个接口并用注解来配置它既可完成。它具备可插拔的注解支持，包括Feign注解和JAX-RS注解。Feign也支持可插拔的编码器和解码器。Spring Cloud为Feign增加了对Spring MVC注解的支持，还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。

<!--more-->

**只需要你创建一个接口,然后在上面添加注解即可.**

使用Ribbon+RestTemplate时,利用RestTemplate对http请求的封装处理,形成了一套模板化的调用方法.但是在实际开发中,由于对服务依赖的调用可能不止一处,**往往一个接口会被多处调用,所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用.**所以,Feign在此基础上做了进一步封装,由他来帮助我们定义和实现依赖服务接口的定义.在Feign的实现下,**我们只需要创建一个接口并使用注解的方式来配置它(以前是Dao接口上标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)**,即可完成对服务提供方的接口绑定,简化了使用Spring cloud Ribbon时,自动封装服务调用客户端的开发量

![1QVbyF.png](https://s2.ax1x.com/2020/01/29/1QVbyF.png)

可以看到openfeign中包括ribbon和hystrix，因此后面使用时不需要再次引入

#### Feign解决了什么问题？

**封装了Http调用流程，更适合面向接口化的变成习惯**
 在服务调用的场景中，我们经常调用基于Http协议的服务，而我们经常使用到的框架可能有HttpURLConnection、Apache HttpComponnets、OkHttp3 、Netty等等，这些框架在基于自身的专注点提供了自身特性。而从角色划分上来看，他们的职能是一致的提供Http调用服务。具体流程如下：

![1JEt8s.png](https://s2.ax1x.com/2020/02/01/1JEt8s.png)

#### Feign是如何设计的？

![1JEwrV.png](https://s2.ax1x.com/2020/02/01/1JEwrV.png)

1. PHASE 1. 基于面向接口的动态代理方式生成实现类

   在使用feign 时，会定义对应的接口类，在接口类上使用Http相关的注解，标识HTTP请求参数信息,如下所示：

   

   ```java
   interface GitHub {
     @RequestLine("GET /repos/{owner}/{repo}/contributors")
     List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);
   }
   
   public static class Contributor {
     String login;
     int contributions;
   }
   
   public class MyApp {
     public static void main(String... args) {
       GitHub github = Feign.builder()
                            .decoder(new GsonDecoder())
                            .target(GitHub.class, "https://api.github.com");
     
       // Fetch and print a list of the contributors to this library.
       List<Contributor> contributors = github.contributors("OpenFeign", "feign");
       for (Contributor contributor : contributors) {
         System.out.println(contributor.login + " (" + contributor.contributions + ")");
       }
     }
   }
   ```

   在Feign 底层，通过基于面向接口的动态代理方式生成实现类，将请求调用委托到动态代理实现类，基本原理如下所示：

   ![1JE6PJ.png](https://s2.ax1x.com/2020/02/01/1JE6PJ.png)

   ```java
   public class ReflectiveFeign extends Feign{
     ///省略部分代码
     @Override
     public <T> T newInstance(Target<T> target) {
       //根据接口类和Contract协议解析方式，解析接口类上的方法和注解，转换成内部的MethodHandler处理方式
       Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
       Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
       List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();
   
       for (Method method : target.type().getMethods()) {
         if (method.getDeclaringClass() == Object.class) {
           continue;
         } else if(Util.isDefault(method)) {
           DefaultMethodHandler handler = new DefaultMethodHandler(method);
           defaultMethodHandlers.add(handler);
           methodToHandler.put(method, handler);
         } else {
           methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
         }
       }
       InvocationHandler handler = factory.create(target, methodToHandler);
       // 基于Proxy.newProxyInstance 为接口类创建动态实现，将所有的请求转换给InvocationHandler 处理。
       T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class<?>[]{target.type()}, handler);
   
       for(DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
         defaultMethodHandler.bindTo(proxy);
       }
       return proxy;
     }
     //省略部分代码
   ```

2. PHASE 2. 根据Contract协议规则，解析接口类的注解信息，解析成内部表现：

   ![1JE7PH.png](https://s2.ax1x.com/2020/02/01/1JE7PH.png)

   Feign 定义了转换协议，定义如下：

   ```java
   /**
    * Defines what annotations and values are valid on interfaces.
    */
   public interface Contract {
   
     /**
      * Called to parse the methods in the class that are linked to HTTP requests.
      * 传入接口定义，解析成相应的方法内部元数据表示
      * @param targetType {@link feign.Target#type() type} of the Feign interface.
      */
     // TODO: break this and correct spelling at some point
     List<MethodMetadata> parseAndValidatateMetadata(Class<?> targetType);
   }
   ```

   默认Contract 实现：

   Feign 默认有一套自己的协议规范，规定了一些注解，可以映射成对应的Http请求，如官方的一个例子：

   ```java
   public interface GitHub {
     
     @RequestLine("GET /repos/{owner}/{repo}/contributors")
     List<Contributor> getContributors(@Param("owner") String owner, @Param("repo") String repository);
     
     class Contributor {
       String login;
       int contributions;
     }
   }
   
   ```

   上述的例子中，尝试调用GitHub.getContributors("foo","myrepo")的的时候，会转换成如下的HTTP请求：

   ```http
   GET /repos/foo/myrepo/contributors
   HOST XXXX.XXX.XXX
   ```

   Feign 默认的协议规范

   | 注解           | 接口Target     | 使用说明                                                     |
   | -------------- | -------------- | ------------------------------------------------------------ |
   | `@RequestLine` | 方法上         | 定义HttpMethod 和 UriTemplate. UriTemplate 中使用`{}` 包裹的表达式，可以通过在方法参数上使用@Param 自动注入 |
   | `@Param`       | 方法参数       | 定义模板变量，模板变量的值可以使用名称的方式使用模板注入解析 |
   | `@Headers`     | 类上或者方法上 | 定义头部模板变量，使用@Param 注解提供参数值的注入。如果该注解添加在接口类上，则所有的请求都会携带对应的Header信息；如果在方法上，则只会添加到对应的方法请求上 |
   | `@QueryMap`    | 方法上         | 定义一个键值对或者 pojo，参数值将会被转换成URL上的 query 字符串上 |
   | `@HeaderMap`   | 方法上         | 定义一个HeaderMap, 与 UrlTemplate 和HeaderTemplate 类型，可以使用@Param 注解提供参数值 |

   具体FeignContract 是如何解析的，不在本文的介绍范围内，详情请参考代码：
    [https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/Contract.java](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FOpenFeign%2Ffeign%2Fblob%2Fmaster%2Fcore%2Fsrc%2Fmain%2Fjava%2Ffeign%2FContract.java)

   **基于Spring MVC的协议规范SpringMvcContract:**

   当前Spring Cloud 微服务解决方案中，为了降低学习成本，采用了Spring MVC的部分注解来完成 请求协议解析，也就是说 ，写客户端请求接口和像写服务端代码一样：客户端和服务端可以通过SDK的方式进行约定，客户端只需要引入服务端发布的SDK API，就可以使用面向接口的编码方式对接服务：

   ![1JV1Q1.png](https://s2.ax1x.com/2020/02/01/1JV1Q1.png)

   至于Spring Cloud 是如何实现 协议解析的，可参考代码：
    [https://github.com/spring-cloud/spring-cloud-openfeign/blob/master/spring-cloud-openfeign-core/src/main/java/org/springframework/cloud/openfeign/support/SpringMvcContract.java](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fspring-cloud%2Fspring-cloud-openfeign%2Fblob%2Fmaster%2Fspring-cloud-openfeign-core%2Fsrc%2Fmain%2Fjava%2Forg%2Fspringframework%2Fcloud%2Fopenfeign%2Fsupport%2FSpringMvcContract.java)

3. PHASE 3. 基于 RequestBean，动态生成Request

   根据传入的Bean对象和注解信息，从中提取出相应的值，来构造Http Request 对象：

   ![1JV2Fg.png](https://s2.ax1x.com/2020/02/01/1JV2Fg.png)

4. PHASE 4. 使用Encoder 将Bean转换成 Http报文正文（消息解析和转码逻辑）

   Feign 最终会将请求转换成Http 消息发送出去，传入的请求对象最终会解析成消息体，如下所示：

   ![1JV4ln.png](https://s2.ax1x.com/2020/02/01/1JV4ln.png)

   在接口定义上Feign做的比较简单，抽象出了Encoder 和decoder 接口：

   ```java
   public interface Encoder {
     /** Type literal for {@code Map<String, ?>}, indicating the object to encode is a form. */
     Type MAP_STRING_WILDCARD = Util.MAP_STRING_WILDCARD;
   
     /**
      * Converts objects to an appropriate representation in the template.
      *  将实体对象转换成Http请求的消息正文中
      * @param object   what to encode as the request body.
      * @param bodyType the type the object should be encoded as. {@link #MAP_STRING_WILDCARD}
      *                 indicates form encoding.
      * @param template the request template to populate.
      * @throws EncodeException when encoding failed due to a checked exception.
      */
     void encode(Object object, Type bodyType, RequestTemplate template) throws EncodeException;
   
     /**
      * Default implementation of {@code Encoder}.
      */
     class Default implements Encoder {
   
       @Override
       public void encode(Object object, Type bodyType, RequestTemplate template) {
         if (bodyType == String.class) {
           template.body(object.toString());
         } else if (bodyType == byte[].class) {
           template.body((byte[]) object, null);
         } else if (object != null) {
           throw new EncodeException(
               format("%s is not a type supported by this encoder.", object.getClass()));
         }
       }
     }
   }
   public interface Decoder {
   
     /**
      * Decodes an http response into an object corresponding to its {@link
      * java.lang.reflect.Method#getGenericReturnType() generic return type}. If you need to wrap
      * exceptions, please do so via {@link DecodeException}.
      *  从Response 中提取Http消息正文，通过接口类声明的返回类型，消息自动装配
      * @param response the response to decode 
      * @param type     {@link java.lang.reflect.Method#getGenericReturnType() generic return type} of
      *                 the method corresponding to this {@code response}.
      * @return instance of {@code type}
      * @throws IOException     will be propagated safely to the caller.
      * @throws DecodeException when decoding failed due to a checked exception besides IOException.
      * @throws FeignException  when decoding succeeds, but conveys the operation failed.
      */
     Object decode(Response response, Type type) throws IOException, DecodeException, FeignException;
   
     /** Default implementation of {@code Decoder}. */
     public class Default extends StringDecoder {
   
       @Override
       public Object decode(Response response, Type type) throws IOException {
         if (response.status() == 404) return Util.emptyValueOf(type);
         if (response.body() == null) return null;
         if (byte[].class.equals(type)) {
           return Util.toByteArray(response.body().asInputStream());
         }
         return super.decode(response, type);
       }
     }
   }
   
   ```

   目前Feign 有以下实现：

   | Encoder/ Decoder 实现                        | 说明                                                         |
   | :------------------------------------------- | :----------------------------------------------------------- |
   | JacksonEncoder，JacksonDecoder               | 基于 Jackson 格式的持久化转换协议                            |
   | GsonEncoder，GsonDecoder                     | 基于Google GSON 格式的持久化转换协议                         |
   | SaxEncoder，SaxDecoder                       | 基于XML 格式的Sax 库持久化转换协议                           |
   | JAXBEncoder，JAXBDecoder                     | 基于XML 格式的JAXB 库持久化转换协议                          |
   | ResponseEntityEncoder，ResponseEntityDecoder | Spring MVC 基于 ResponseEntity< T > 返回格式的转换协议       |
   | SpringEncoder，SpringDecoder                 | 基于Spring MVC HttpMessageConverters 一套机制实现的转换协议 ,应用于Spring Cloud 体系中 |

5. PHASE 5. 拦截器负责对请求和返回进行装饰处理

   在请求转换的过程中，Feign 抽象出来了拦截器接口，用于用户自定义对请求的操作：

   ```java
   public interface RequestInterceptor {
   
     /**
      * 可以在构造RequestTemplate 请求时，增加或者修改Header, Method, Body 等信息
      * Called for every request. Add data using methods on the supplied {@link RequestTemplate}.
      */
     void apply(RequestTemplate template);
   }
   ```

   比如，如果希望Http消息传递过程中被压缩，可以定义一个请求拦截器：

   ```java
   public class FeignAcceptGzipEncodingInterceptor extends BaseRequestInterceptor {
   
       /**
        * Creates new instance of {@link FeignAcceptGzipEncodingInterceptor}.
        *
        * @param properties the encoding properties
        */
       protected FeignAcceptGzipEncodingInterceptor(FeignClientEncodingProperties properties) {
           super(properties);
       }
   
       /**
        * {@inheritDoc}
        */
       @Override
       public void apply(RequestTemplate template) {
           //  在Header 头部添加相应的数据信息
           addHeader(template, HttpEncoding.ACCEPT_ENCODING_HEADER, HttpEncoding.GZIP_ENCODING,
                   HttpEncoding.DEFLATE_ENCODING);
       }
   }
   ```

6. PHASE 6. 日志记录

   在发送和接收请求的时候，Feign定义了统一的日志门面来输出日志信息 , 并且将日志的输出定义了四个等级：

   | 级别    | 说明                                                         |
   | :------ | :----------------------------------------------------------- |
   | NONE    | 不做任何记录                                                 |
   | BASIC   | 只记录输出Http 方法名称、请求URL、返回状态码和执行时间       |
   | HEADERS | 记录输出Http 方法名称、请求URL、返回状态码和执行时间 和 Header 信息 |
   | FULL    | 记录Request 和Response的Header，Body和一些请求元数据         |

   ```java
   public abstract class Logger {
   
     protected static String methodTag(String configKey) {
       return new StringBuilder().append('[').append(configKey.substring(0, configKey.indexOf('(')))
           .append("] ").toString();
     }
   
     /**
      * Override to log requests and responses using your own implementation. Messages will be http
      * request and response text.
      *
      * @param configKey value of {@link Feign#configKey(Class, java.lang.reflect.Method)}
      * @param format    {@link java.util.Formatter format string}
      * @param args      arguments applied to {@code format}
      */
     protected abstract void log(String configKey, String format, Object... args);
   
     protected void logRequest(String configKey, Level logLevel, Request request) {
       log(configKey, "---> %s %s HTTP/1.1", request.method(), request.url());
       if (logLevel.ordinal() >= Level.HEADERS.ordinal()) {
   
         for (String field : request.headers().keySet()) {
           for (String value : valuesOrEmpty(request.headers(), field)) {
             log(configKey, "%s: %s", field, value);
           }
         }
   
         int bodyLength = 0;
         if (request.body() != null) {
           bodyLength = request.body().length;
           if (logLevel.ordinal() >= Level.FULL.ordinal()) {
             String
                 bodyText =
                 request.charset() != null ? new String(request.body(), request.charset()) : null;
             log(configKey, ""); // CRLF
             log(configKey, "%s", bodyText != null ? bodyText : "Binary data");
           }
         }
         log(configKey, "---> END HTTP (%s-byte body)", bodyLength);
       }
     }
   
     protected void logRetry(String configKey, Level logLevel) {
       log(configKey, "---> RETRYING");
     }
   
     protected Response logAndRebufferResponse(String configKey, Level logLevel, Response response,
                                               long elapsedTime) throws IOException {
       String reason = response.reason() != null && logLevel.compareTo(Level.NONE) > 0 ?
           " " + response.reason() : "";
       int status = response.status();
       log(configKey, "<--- HTTP/1.1 %s%s (%sms)", status, reason, elapsedTime);
       if (logLevel.ordinal() >= Level.HEADERS.ordinal()) {
   
         for (String field : response.headers().keySet()) {
           for (String value : valuesOrEmpty(response.headers(), field)) {
             log(configKey, "%s: %s", field, value);
           }
         }
   
         int bodyLength = 0;
         if (response.body() != null && !(status == 204 || status == 205)) {
           // HTTP 204 No Content "...response MUST NOT include a message-body"
           // HTTP 205 Reset Content "...response MUST NOT include an entity"
           if (logLevel.ordinal() >= Level.FULL.ordinal()) {
             log(configKey, ""); // CRLF
           }
           byte[] bodyData = Util.toByteArray(response.body().asInputStream());
           bodyLength = bodyData.length;
           if (logLevel.ordinal() >= Level.FULL.ordinal() && bodyLength > 0) {
             log(configKey, "%s", decodeOrDefault(bodyData, UTF_8, "Binary data"));
           }
           log(configKey, "<--- END HTTP (%s-byte body)", bodyLength);
           return response.toBuilder().body(bodyData).build();
         } else {
           log(configKey, "<--- END HTTP (%s-byte body)", bodyLength);
         }
       }
       return response;
     }
   
     protected IOException logIOException(String configKey, Level logLevel, IOException ioe, long elapsedTime) {
       log(configKey, "<--- ERROR %s: %s (%sms)", ioe.getClass().getSimpleName(), ioe.getMessage(),
           elapsedTime);
       if (logLevel.ordinal() >= Level.FULL.ordinal()) {
         StringWriter sw = new StringWriter();
         ioe.printStackTrace(new PrintWriter(sw));
         log(configKey, sw.toString());
         log(configKey, "<--- END ERROR");
       }
       return ioe;
     }
   ```

7. PHASE 7 . 基于重试器发送HTTP请求

   Feign 内置了一个重试器，当HTTP请求出现IO异常时，Feign会有一个最大尝试次数发送请求，以下是Feign核心
    代码逻辑：

   

   ```java
   final class SynchronousMethodHandler implements MethodHandler {
   
     // 省略部分代码
   
     @Override
     public Object invoke(Object[] argv) throws Throwable {
      //根据输入参数，构造Http 请求。
       RequestTemplate template = buildTemplateFromArgs.create(argv);
       // 克隆出一份重试器
       Retryer retryer = this.retryer.clone();
       // 尝试最大次数，如果中间有结果，直接返回
       while (true) {
         try {
           return executeAndDecode(template);
         } catch (RetryableException e) {
           retryer.continueOrPropagate(e);
           if (logLevel != Logger.Level.NONE) {
             logger.logRetry(metadata.configKey(), logLevel);
           }
           continue;
         }
       }
     }
   ```

   重试器有如下几个控制参数：

   | 重试参数    | 说明                                                         | 默认值 |
   | :---------- | :----------------------------------------------------------- | :----- |
   | period      | 初始重试时间间隔，当请求失败后，重试器将会暂停 初始时间间隔(线程 sleep 的方式)后再开始，避免强刷请求，浪费性能 | 100ms  |
   | maxPeriod   | 当请求连续失败时，重试的时间间隔将按照：`long interval = (long) (period * Math.pow(1.5, attempt - 1));` 计算，按照等比例方式延长，但是最大间隔时间为 maxPeriod, 设置此值能够避免 重试次数过多的情况下执行周期太长 | 1000ms |
   | maxAttempts | 最大重试次数                                                 | 5      |

   具体的代码实现可参考：
    [https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/Retryer.java](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FOpenFeign%2Ffeign%2Fblob%2Fmaster%2Fcore%2Fsrc%2Fmain%2Fjava%2Ffeign%2FRetryer.java)

8. PHASE 8. 发送Http请求

   Feign 真正发送HTTP请求是委托给 `feign.Client` 来做的：

   

   ```java
   public interface Client {
   
     /**
      * Executes a request against its {@link Request#url() url} and returns a response.
      *  执行Http请求，并返回Response
      * @param request safe to replay.
      * @param options options to apply to this request.
      * @return connected response, {@link Response.Body} is absent or unread.
      * @throws IOException on a network error connecting to {@link Request#url()}.
      */
     Response execute(Request request, Options options) throws IOException;
     }
   ```

   Feign 默认底层通过JDK 的 `java.net.HttpURLConnection` 实现了`feign.Client`接口类,在每次发送请求的时候，都会创建新的HttpURLConnection 链接，这也就是为什么默认情况下Feign的性能很差的原因。可以通过拓展该接口，使用Apache HttpClient 或者OkHttp3等基于连接池的高性能Http客户端，我们项目内部使用的就是OkHttp3作为Http 客户端。

   如下是Feign 的默认实现，供参考：

   

   ```java
   public static class Default implements Client {
   
       private final SSLSocketFactory sslContextFactory;
       private final HostnameVerifier hostnameVerifier;
   
       /**
        * Null parameters imply platform defaults.
        */
       public Default(SSLSocketFactory sslContextFactory, HostnameVerifier hostnameVerifier) {
         this.sslContextFactory = sslContextFactory;
         this.hostnameVerifier = hostnameVerifier;
       }
   
       @Override
       public Response execute(Request request, Options options) throws IOException {
         HttpURLConnection connection = convertAndSend(request, options);
         return convertResponse(connection).toBuilder().request(request).build();
       }
   
       HttpURLConnection convertAndSend(Request request, Options options) throws IOException {
         final HttpURLConnection
             connection =
             (HttpURLConnection) new URL(request.url()).openConnection();
         if (connection instanceof HttpsURLConnection) {
           HttpsURLConnection sslCon = (HttpsURLConnection) connection;
           if (sslContextFactory != null) {
             sslCon.setSSLSocketFactory(sslContextFactory);
           }
           if (hostnameVerifier != null) {
             sslCon.setHostnameVerifier(hostnameVerifier);
           }
         }
         connection.setConnectTimeout(options.connectTimeoutMillis());
         connection.setReadTimeout(options.readTimeoutMillis());
         connection.setAllowUserInteraction(false);
         connection.setInstanceFollowRedirects(true);
         connection.setRequestMethod(request.method());
   
         Collection<String> contentEncodingValues = request.headers().get(CONTENT_ENCODING);
         boolean
             gzipEncodedRequest =
             contentEncodingValues != null && contentEncodingValues.contains(ENCODING_GZIP);
         boolean
             deflateEncodedRequest =
             contentEncodingValues != null && contentEncodingValues.contains(ENCODING_DEFLATE);
   
         boolean hasAcceptHeader = false;
         Integer contentLength = null;
         for (String field : request.headers().keySet()) {
           if (field.equalsIgnoreCase("Accept")) {
             hasAcceptHeader = true;
           }
           for (String value : request.headers().get(field)) {
             if (field.equals(CONTENT_LENGTH)) {
               if (!gzipEncodedRequest && !deflateEncodedRequest) {
                 contentLength = Integer.valueOf(value);
                 connection.addRequestProperty(field, value);
               }
             } else {
               connection.addRequestProperty(field, value);
             }
           }
         }
         // Some servers choke on the default accept string.
         if (!hasAcceptHeader) {
           connection.addRequestProperty("Accept", "*/*");
         }
   
         if (request.body() != null) {
           if (contentLength != null) {
             connection.setFixedLengthStreamingMode(contentLength);
           } else {
             connection.setChunkedStreamingMode(8196);
           }
           connection.setDoOutput(true);
           OutputStream out = connection.getOutputStream();
           if (gzipEncodedRequest) {
             out = new GZIPOutputStream(out);
           } else if (deflateEncodedRequest) {
             out = new DeflaterOutputStream(out);
           }
           try {
             out.write(request.body());
           } finally {
             try {
               out.close();
             } catch (IOException suppressed) { // NOPMD
             }
           }
         }
         return connection;
       }
   
       Response convertResponse(HttpURLConnection connection) throws IOException {
         int status = connection.getResponseCode();
         String reason = connection.getResponseMessage();
   
         if (status < 0) {
           throw new IOException(format("Invalid status(%s) executing %s %s", status,
               connection.getRequestMethod(), connection.getURL()));
         }
   
         Map<String, Collection<String>> headers = new LinkedHashMap<String, Collection<String>>();
         for (Map.Entry<String, List<String>> field : connection.getHeaderFields().entrySet()) {
           // response message
           if (field.getKey() != null) {
             headers.put(field.getKey(), field.getValue());
           }
         }
   
         Integer length = connection.getContentLength();
         if (length == -1) {
           length = null;
         }
         InputStream stream;
         if (status >= 400) {
           stream = connection.getErrorStream();
         } else {
           stream = connection.getInputStream();
         }
         return Response.builder()
                 .status(status)
                 .reason(reason)
                 .headers(headers)
                 .body(stream, length)
                 .build();
       }
     }
   ```

#### Feign 的性能怎么样？

Feign 整体框架非常小巧，在处理请求转换和消息解析的过程中，基本上没什么时间消耗。真正影响性能的，是处理Http请求的环节。

 如上所述，由于默认情况下，Feign采用的是JDK的`HttpURLConnection`,所以整体性能并不高，刚开始接触Spring Cloud 的同学，如果没注意这些细节，可能会对Spring Cloud 有很大的偏见。

#### 示例

下面，通过一个例子来展现Feign如何方便的声明对上述user-service服务的定义和调用。

创建一个Spring Boot工程，配置`pom.xml`，将上述的配置中的ribbon依赖替换成feign的依赖即可，具体如下：

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        

        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>


        <!--eureke client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!-- actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- 热启动，热部署依赖包，为了调试方便，加入此包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

在应用主类中通过`@EnableFeignClients`注解开启Feign功能，具体如下：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients // 要使用Feign，需要加上此注解
public class MovieServiceSnApplication {

    public static void main(String[] args) {
        SpringApplication.run(MovieServiceSnApplication.class, args);
    }
}
```

定义服务的接口：

```java
@FeignClient(name = "user-service")
// 如果不使用服务发现，则可以如下直接指定请求的URL
//@FeignClient(name = "user-service", url = "http://localhost:8000/")
public interface UserFeignClient {
    @GetMapping(value = "/{id}")
    User findById(@PathVariable("id") Long id);
}

```

通过Spring MVC的注解来配置compute-service服务下的具体实现。

在web层中调用上面定义UserFeignClient接口

```java
@RestController
public class MovieController {

    @Autowired
    private UserFeignClient userFeignClient;

    @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
    public User findById(@PathVariable Long id) {
        return userFeignClient.findById(id);
    }

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/user-instance")
    public List<ServiceInstance> showInfo() {
        return discoveryClient.getInstances("user-service");
    }
}
```

配置文件不变

启动该应用，访问几次：<http://127.0.0.1:8010/user/1>

再观察日志，可以得到之前使用Ribbon时一样的结果，对服务提供方实现了均衡负载。

完整示例可参考:<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part3_feign>
