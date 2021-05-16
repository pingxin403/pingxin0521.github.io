---
title: SpringCloud组件之Spring Cloud Stream
date: 2020-12-18 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

Spring Cloud Stream是一个建立在Spring Boot和Spring Integration之上的框架，有助于创建事件驱动或消息驱动的微服务

官网：https://spring.io/projects/spring-cloud-stream

<!--more-->

### 简介

Springcloud Stream 是一个构建消息驱动微服务的框架。说白了就是操作MQ的，可以屏蔽底层的MQ类型。

应用程序通过inputs 或者 outputs 与SpringcloudStream中binder对象交互。所以我们通过配置来绑定(binding),而与SpringcloudStream 的binder对象负责与消息中间件交互。所以我们只需要了解如何与Stream的binder交互就可以了，屏蔽了与底层MQ交互。

Springcloud Stream为一些MQ提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。

![](https://s3.ax1x.com/2020/12/27/rIm7ct.png)

说明：最底层是消息服务，中间层是绑定层，绑定层和底层的消息服务进行绑定，顶层是消息生产者和消息消费者，顶层可以向绑定层生产消息和和获取消息消费

**为什么引入Stream**

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。类似于Hibernate的作用一样， 我们更多的注重业务开发。Hibernate屏蔽了数据库的差异，可以很好的实现数据库的切换。Stream屏蔽了底层MQ的区别，可以很好的实现切换。目前主流的有ActiveMQ、RocketMQ、RabbitMQ、Kafka，Stream支持的有RabbitMQ和Kafka。

**设计思想**

1. 标准的MQ

   (1)生产者/消费者通过消息媒介传递消息内容

   (2)消息必须走特定的通道Channel

2. 引入Stream

   通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节的隔离。

   ![](https://s3.ax1x.com/2020/12/27/rInhrV.png)

   | 组成            | 说明                                                         |
   | --------------- | ------------------------------------------------------------ |
   | Middleware      | 中间件，目前只支持RabbitMQ和Kafka                            |
   | Binder          | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic，RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
   | @Input          | 注解标识输入通道，通过该输入通道接收到的消息进入应用程序     |
   | @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
   | @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
   | @EnableBinding  | 指信道channel和exchange绑定在一起                            |

3. Stream 的消息通信模式遵循了发布-订阅模式，也就是Topic模式。在RabbitMQ中是Exchange交换机，在Kafka是Topic。

4. 术语

   (1)Binder 绑定器，通过Binder可以很方便的连接中间件，屏蔽差异

   (2)Channel: 通道，是Queue的一种抽象，主要实现存储和转发的媒介，通过Channel对队列进行配置

   (3)Source和Sink 简单的理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接收消息就是输入。

5. 过程可以理解为下图

   ![](https://s3.ax1x.com/2020/12/27/rInoaF.png)

   

### 示例

#### 环境安装

docker安装rabbitMQ

```
docker pull rabbitmq
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672  --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq
```

浏览器打开web管理端：[http://ip:15672](http://ip:15672/)

#### Maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
<!--支持Junit单元测试-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-test-support</artifactId>
    <scope>test</scope>
</dependency>
```

#### 主要概念

微服务架构遵循“智能端点和哑管道”的原则。端点之间的通信由消息中间件（如RabbitMQ或Apache Kafka）驱动。服务通过这些端点或信道发布事件来进行通信。

让我们通过下面这个构建消息驱动服务的基本范例，来看看Spring Cloud Stream框架的一些主要概念。

**服务类**

通过Spring Cloud Stream建立一个简单的应用，从Input通道监听消息然后返回应答到Output通道。

```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class MyLoggerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyLoggerServiceApplication.class, args);
    }

    @StreamListener(Processor.INPUT)
    @SendTo(Processor.OUTPUT)
    public LogMessage enrichLogMessage(LogMessage log) {
        return new LogMessage(String.format("[1]: %s", log.getMessage()));
    }
}
```

注解@EnableBinding声明了这个应用程序绑定了2个通道：INPUT和OUTPUT。这2个通道是在接口Processor中定义的（Spring Cloud Stream默认设置）。所有通道都是配置在一个具体的消息中间件或绑定器中。

让我们来看下这些概念的定义：

- Bindings — 声明输入和输出通道的接口集合。
- Binder — 消息中间件的实现，如Kafka或RabbitMQ
- Channel — 表示消息中间件和应用程序之间的通信管道
- StreamListeners — bean中的消息处理方法，在中间件的MessageConverter特定事件中进行对象序列化/反序列化之后，将在信道上的消息上自动调用消息处理方法。
- Message Schemas — 用于消息的序列化和反序列化，这些模式可以静态读取或者动态加载，支持对象类型的演变。

将消息发布到指定目的地是由发布订阅消息模式传递。发布者将消息分类为主题，每个主题由名称标识。订阅方对一个或多个主题表示兴趣。中间件过滤消息，将感兴趣的主题传递给订阅服务器。订阅方可以分组，消费者组是由组ID标识的一组订户或消费者，其中从主题或主题的分区中的消息以负载均衡的方式递送。

**测试类**

测试类是一个绑定器的实现，允许与通道交互和检查消息。让我们向上面的enrichLogMessage 服务发送一条消息，并检查响应中是否包含文本

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = MyLoggerServiceApplication.class)
@DirtiesContext
public class MyLoggerApplicationIntegrationTest {
    @Autowired
    private Processor pipe;

    @Autowired
    private MessageCollector messageCollector;

    @Test
    public void whenSendMessage_thenResponseShouldUpdateText() {
        pipe.input().send(MessageBuilder.withPayload(new LogMessage("This is my message")).build());

        Object payload = messageCollector.forChannel(pipe.output()).poll().getPayload();

        assertEquals("[1]: This is my message", payload.toString());
    }
}
```

**RabbitMQ配置**

我们需要在工程src/main/resources目录下的application.yml文件里增加RabbitMQ绑定器的配置。

```yaml
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: queue.log.messages
          binder: local_rabbit
          group: logMessageConsumers
        output:
          destination: queue.pretty.log.messages
          binder: local_rabbit
      binders:
        local_rabbit:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
                virtual-host: /
```

input绑定使用名为queue.log.messages的消息交换机，output绑定使用名为queue.pretty.log.messages的消息交换机。所有的绑定都使用名为local_rabbit的绑定器。
请注意，我们不需要预先创建RabbitmQ交换机或队列。运行应用程序时，两个交换机都会自动创建。

#### 自定义通道

在上面的例子里，我们使用Spring Cloud提供的Processor接口，这个接口有一个input通道和一个output通道。

如果我们想创建一些不同，比如说一个input通道和两个output通道，可以新建一个自定义处理器。

```
public interface MyProcessor {
    String INPUT = "myInput";

    @Input
    SubscribableChannel myInput();

    @Output("myOutput")
    MessageChannel anOutput();

    @Output
    MessageChannel anotherOutput();
}
```

**服务类**

Spring将为我们提供这个接口的实现。通道的名称可以通过使用注解来设定，比如@Output(“myOutput”)。如果没有设置的话，Spring将使用方法名来作为通道名称。因此这里有三个通道：myInput， myOutput， anotherOutput。

现在我们可以增加一些路由规则，如果接收到的值小于10则走一个output通道；如果接收到的值大于等于10则走另一个output通道。

```java
@Autowired
private MyProcessor processor;

@StreamListener(MyProcessor.INPUT)
public void routeValues(Integer val) {
    if (val < 10) {
        processor.anOutput().send(message(val));
    } else {
        processor.anotherOutput().send(message(val));
    }
}

private static final <T> Message<T> message(T val) {
    return MessageBuilder.withPayload(val).build();
}
```

**测试类**

发送不同的消息，判断返回值是否是通过不同的通道获得。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = MultipleOutputsServiceApplication.class)
@DirtiesContext
public class MultipleOutputsServiceApplicationIntegrationTest {
    @Autowired
    private MyProcessor pipe;

    @Autowired
    private MessageCollector messageCollector;

    @Test
    public void whenSendMessage_thenResponseIsInAOutput() {
        whenSendMessage(1);
        thenPayloadInChannelIs(pipe.anOutput(), 1);
    }

    @Test
    public void whenSendMessage_thenResponseIsInAnotherOutput() {
        whenSendMessage(11);
        thenPayloadInChannelIs(pipe.anotherOutput(), 11);
    }

    private void whenSendMessage(Integer val) {
        pipe.myInput().send(MessageBuilder.withPayload(val).build());
    }

    private void thenPayloadInChannelIs(MessageChannel channel, Integer expectedValue) {
        Object payload = messageCollector.forChannel(channel).poll().getPayload();
        assertEquals(expectedValue, payload);
    }
}
```

#### 根据条件分派

使用@StreamListener 注释，我们还可以使用自定义的SpEL表达式来过滤用户期望的消息。下面这个例子，我们使用条件调度将消息路由到不同的输出。

```java
@Autowired
private MyProcessor processor;

@StreamListener(
  target = MyProcessor.INPUT, 
  condition = "payload < 10")
public void routeValuesToAnOutput(Integer val) {
    processor.anOutput().send(message(val));
}

@StreamListener(
  target = MyProcessor.INPUT, 
  condition = "payload >= 10")
public void routeValuesToAnotherOutput(Integer val) {
    processor.anotherOutput().send(message(val));
}
```

#### 结合Kafka

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-stream</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-stream-kafka</artifactId>
   </dependency>
   ```

2. 定义Kafka Streams

   ```java
   public interface GreetingsStreams {
       String INPUT = "greetings-in";
       String OUTPUT = "greetings-out";
       @Input(INPUT)
       SubscribableChannel inboundGreetings();
       @Output(OUTPUT)
       MessageChannel outboundGreetings();
   }
   ```

   为了使我们的应用程序能够与Kafka通信，我们需要定义一个出站流来将消息写入Kafka主题，并使用入站流来读取来自Kafka主题的消息。

   Spring Cloud提供了一种方便的方法：可以简单地创建为每个流定义单独的方法接口。

   inboundGreetings()方法定义要从Kafka读取的入站流，并且outboundGreetings()方法定义要写入Kafka的出站流。

   在运行时，Spring将创建一个基于Java代理的GreetingsStreams接口实现，可以在代码中的任何位置注入Spring Bean来访问我们的两个流。

3. 配置Spring Cloud Stream

   我们的下一步是将Spring Cloud Stream绑定到GreetingsStreams接口中的流。这可以通过使用以下代码创建一个StreamsConfig类来完成：

   ```java
   @EnableBinding(GreetingsStreams.class)
   public class StreamsConfig {
   }
   ```

4. Kafka的配置属性

   ```yaml
   spring:
     cloud:
       stream:
         kafka:
           binder:
             brokers: localhost:9092
         bindings:
           greetings-in:
             destination: greetings
             contentType: application/json
           greetings-out:
             destination: greetings
             contentType: application/json
   ```

   上面的配置属性配置了要连接的Kafka服务器的地址，以及我们在代码中用于入站和出站流的Kafka主题。他们都必须使用相同的Kafka主题greetings！contentType属性告诉Spring Cloud Stream在流中发送/接收我们的消息对象String。

5. 创建消息对象

   Greetings使用下面的代码创建一个简单的类，代表我们读取的消息对象并写入greetingsKafka主题：

   ```java
   @Getter
   @Setter
   @ToString
   @Builder
   public class Greetings {
       private long timestamp;
       private String message;
   
       public Greetings(long timestamp, String message) {
           this.timestamp = timestamp;
           this.message = message;
       }
   
       public Greetings() {
   
       }
   }
   ```

6. 创建一个服务层来写入Kafka

   让我们使用下面的代码创建GreetingsService类，该代码将Greetings对象写入greetingsKafka主题：

   ```java
   @Service
   @Slf4j
   public class GreetingsService {
       private final GreetingsStreams greetingsStreams;
   
       public GreetingsService(GreetingsStreams greetingsStreams) {
           this.greetingsStreams = greetingsStreams;
       }
   
       public void sendGreeting(final Greetings greetings) {
           log.info("Sending greetings {}", greetings);
           MessageChannel messageChannel = greetingsStreams.outboundGreetings();
           messageChannel.send(MessageBuilder
                   .withPayload(greetings)
                   .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON)
                   .build());
       }
   }
   ```

   在sendGreeting()方法中，我们使用注入的GreetingsStream对象来发送由Greetings对象表示的消息。

7. 创建REST API

   现在我们将创建一个REST API端点，它将触发使用GreetingsServiceSpring Bean 向Kafka发送消息：

   ```java
   @RestController
   public class GreetingsController {
       private final GreetingsService greetingsService;
       public GreetingsController(GreetingsService greetingsService) {
           this.greetingsService = greetingsService;
       }
       @GetMapping("/greetings")
       @ResponseStatus(HttpStatus.ACCEPTED)
       public void greetings(@RequestParam("message") String message) {
           Greetings greetings = Greetings.builder()
                   .message(message)
                   .timestamp(System.currentTimeMillis())
                   .build();
           greetingsService.sendGreeting(greetings);
       }
   }
   ```

   greetings()方法定义了一个HTTP GET /greetings端点，该端点接受请求参数message并将其传递给GreetingsService的sendGreeting()方法

8. 监听Kafka主题

   ```java
   @Component
   @Slf4j
   public class GreetingsListener {
       @StreamListener(GreetingsStreams.INPUT)
       public void handleGreetings(@Payload Greetings greetings) {
           log.info("Received greetings: {}", greetings);
       }
   }
   ```

   GreetingsListener有一个方法handleGreetings()，Spring Cloud Stream将监听Kafka主题Greetings上的每个新消息对象greetings。这要归功于为handleGreetings()方法配置的注释@StreamListener。

9. 运行应用程序

   ```
   mvn spring-boot:run
   ```

   应用程序运行后，在浏览器中转到 http://localhost:8080/greetings?message=hello并检查控制台:

   ```
   (timestamp=1531643278270, message=hello)
   ```

   如果需要输入用户名和密码，用户名是user，密码在控制台using generated security password可以找到，或者使用元注解排除SecurityAutoConfiguration.class

   ```
   @SpringBootApplication(exclude = {SecurityAutoConfiguration.class })
   ```

