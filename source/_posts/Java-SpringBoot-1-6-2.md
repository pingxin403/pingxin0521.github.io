---
title: Spring Boot 消息队列 Kafka 入门
date: 2020-5-29 18:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

Kafka 是一种高吞吐量的分布式发布订阅消息系统，她有如下特性：

- 通过 O(1) 的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
- 高吞吐量：即使是非常普通的硬件kafka也可以支持每秒数十万的消息。
- 支持通过 Kafka 服务器和消费机集群来分区消息。

<!--more-->

在 Spring 生态中，提供了 [Spring-Kafka](https://spring.io/projects/spring-kafka) 项目，让我们更简便的使用 Kafka 。其官网介绍如下：

> The Spring for Apache Kafka (spring-kafka) project applies core Spring concepts to the development of Kafka-based messaging solutions.
> Spring for Apache Kafka (spring-kafka) 项目将 Spring 核心概念应用于基于 Kafka 的消息传递解决方案的开发。
>
> It provides a "template" as a high-level abstraction for sending messages.
> 它提供了一个“模板”作为发送消息的高级抽象。
>
> It also provides support for Message-driven POJOs with @KafkaListener annotations and a "listener container".
> 它还通过 @KafkaListener 注解和“侦听器容器(listener container)”为消息驱动的 POJO 提供支持。
>
> These libraries promote the use of dependency injection and declarative.
> 这些库促进了依赖注入和声明的使用。
>
> In all of these cases, you will see similarities to the JMS support in the Spring Framework and RabbitMQ support in Spring AMQP.
> 在所有这些用例中，你将看到 Spring Framework 中的 JMS 支持，以及和 Spring AMQP 中的 RabbitMQ 支持的相似之处。

- 注意，Spring-Kafka 是基于 [Spring Message](https://github.com/spring-projects/spring-framework/tree/master/spring-messaging/src/main/java/org/springframework/messaging) 来实现 Kafka 的发送端和接收端。

#### 快速入门

1. pom依赖

   ```xml
    <!-- 引入 Spring-Kafka 依赖 -->
           <!-- 已经内置 kafka-clients 依赖，所以无需重复引入 -->
           <dependency>
               <groupId>org.springframework.kafka</groupId>
               <artifactId>spring-kafka</artifactId>
               <version>2.3.4.RELEASE</version>
           </dependency>
   
           <!-- 实现对 JSON 的自动化配置 -->
           <!-- 因为，Kafka 对复杂对象的 Message 序列化时，我们会使用到 JSON -->
           <!--
               同时，spring-boot-starter-json 引入了 spring-boot-starter ，而 spring-boot-starter 又引入了 spring-boot-autoconfigure 。
               spring-boot-autoconfigure 实现了 Spring-Kafka 的自动化配置
            -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-json</artifactId>
           </dependency>
   
           <!-- 方便等会写单元测试 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
   ```

   

2. 应用配置文件

   ```yml
   spring:
     # Kafka 配置项，对应 KafkaProperties 配置类
     kafka:
       bootstrap-servers: 127.0.0.1:9092 # 指定 Kafka Broker 地址，可以设置多个，以逗号分隔
       # Kafka Producer 配置项
       producer:
         acks: 1 # 0-不应答。1-leader 应答。all-所有 leader 和 follower 应答。
         retries: 3 # 发送失败时，重试发送的次数
         key-serializer: org.apache.kafka.common.serialization.StringSerializer # 消息的 key 的序列化
         value-serializer: org.springframework.kafka.support.serializer.JsonSerializer # 消息的 value 的序列化
       # Kafka Consumer 配置项
       consumer:
         auto-offset-reset: earliest # 设置消费者分组最初的消费进度为 earliest 。可参考博客 https://blog.csdn.net/lishuangzhe7047/article/details/74530417 理解
         key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
         value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
         properties:
           spring:
             json:
               trusted:
                 packages: com.hyp.learn.kafka.message
       # Kafka Consumer Listener 监听器配置
       listener:
         missing-topics-fatal: false # 消费监听接口监听的主题不存在时，默认会报错。所以通过设置为 false ，解决报错
   
   logging:
     level:
       org:
         springframework:
           kafka: ERROR # spring-kafka INFO 日志太多了，所以我们限制只打印 ERROR 级别
         apache:
           kafka: ERROR # kafka INFO 日志太多了，所以我们限制只打印 ERROR 级别
   ```

   - 在 `spring.kafka` 配置项，设置 Kafka 的配置，对应 `org.springframework.boot.autoconfigure.kafka.KafkaProperties` 配置类
   - Spring Boot 提供的 `org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration` 自动化配置类，实现 Kafka 的自动配置，创建相应的 Producer 和 Consumer 。
   - `spring.kafka.bootstrap-servers` 配置项，设置 Kafka Broker 地址。如果多个，使用逗号分隔。
   - `spring.kafka.producer`配置项，一看就知道是 Kafka Producer 所独有。
     - `value-serializer` 配置，我们使用了 Spring-Kafka 提供的 [JsonSerializer](https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer/JsonSerializer.java) 序列化类，因为稍后我们要使用 JSON 的方式，序列化复杂的 Message 消息。
     - 其它配置，一般默认即可。
   - `spring.kafka.consumer`配置项，一看就知道是 Kafka Consumer 所独有。
     - `value-serializer` 配置，我们使用了 Spring-Kafka 提供的 [JsonDeserializer](https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer/JsonDeserializer.java) 反序列化类，因为稍后我们要使用 JSON 的方式，反序列化复杂的 Message 消息。
     - `properties.spring.json.trusted.packages` 配置，配置信任 `com.hyp.learn.kafka.message` 包下的 Message 类们。因为 JsonDeserializer 在反序列化消息时，考虑到安全性，只反序列化成信任的 Message 类

3. messge

   ```java
   public class OrderMessage implements Serializable {
       public static final String TOPIC = "Demo_Test";
   
       private Integer id;
       private String name;
   
   }
   ```

4. Producer

   ```java
   @Component
   public class OrderProducer {
       @Resource
       private KafkaTemplate<Object, Object> kafkaTemplate;
   
       public SendResult syncSend(Integer id) throws ExecutionException, InterruptedException {
           // 创建 Demo01Message 消息
           OrderMessage message = new OrderMessage();
           message.setId(id);
           message.setName(UUID.randomUUID().toString());
           // 同步发送消息
           return kafkaTemplate.send(OrderMessage.TOPIC, message).get();
       }
   
       public ListenableFuture<SendResult<Object, Object>> asyncSend(Integer id) {
           // 创建 Demo01Message 消息
           OrderMessage message = new OrderMessage();
           message.setId(id);
           message.setName(UUID.randomUUID().toString());
           
           // 异步发送消息
           return kafkaTemplate.send(OrderMessage.TOPIC, message);
       }
   }
   ```

   - `#asyncSend(...)` 方法，**异步**发送消息。在方法内部，会调用 `KafkaTemplate#send(topic, data)` 方法，**异步**发送消息，返回 Spring [ListenableFuture](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/util/concurrent/ListenableFuture.java) 对象，一个可以通过监听执行结果的 Future 增强。
   - `#syncSend(...)` 方法，**同步**发送消息。在方法内部，也是调用 `KafkaTemplate#send(topic, data)` 方法，**异步**发送消息。不过，因为我们后面调用了 ListenableFuture 对象的 `#get()` 方法，阻塞等待发送结果，从而实现同步的效果。
   - 暂时未提供 **oneway** 发送消息的方式。因为需要配置 Producer 的 `acks = 0` ，才可以使用这种发送方式。
   - 在序列化时，我们使用了 JsonSerializer 序列化 Message 消息对象，它会在 Kafka 消息 [Headers](https://kafka.apache.org/0110/javadoc/index.html?org/apache/kafka/common/header/Headers.html) 的 `__TypeId__` 上，值为 Message 消息对应的**类全名**。
   - 在反序列化时，我们使用了 JsonDeserializer 序列化出 Message 消息对象，它会根据 Kafka 消息 [Headers](https://kafka.apache.org/0110/javadoc/index.html?org/apache/kafka/common/header/Headers.html) 的 `__TypeId__` 的值，反序列化消息内容成该 Message 对象。

5. consumer

   ```java
   @Component
   public class OrderConsumer {
       private Logger logger = LoggerFactory.getLogger(getClass());
   
       @KafkaListener(topics = OrderMessage.TOPIC,
               groupId = "order-consumer-group-" + OrderMessage.TOPIC)
       public void onMessage(OrderMessage message) {
           logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
       }
   
       @KafkaListener(topics = OrderMessage.TOPIC,
               groupId = "order-A-consumer-group-" + OrderMessage.TOPIC)
       public void onMessage02(OrderMessage record) {
           logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), record);
       }
   }
   ```

   > 集群消费（Clustering）：集群消费模式下，相同 Consumer Group 的每个 Consumer 实例平均分摊消息。

   通过**集群消费**的机制，我们可以实现针对相同 Topic ，不同消费者分组实现各自的业务逻辑。例如说：用户注册成功时，发送一条 Topic 为 `"USER_REGISTER"` 的消息。然后，不同模块使用不同的消费者分组，订阅该 Topic ，实现各自的拓展逻辑：

   - 积分模块：判断如果是手机注册，给用户增加 20 积分。
   - 优惠劵模块：因为是新用户，所以发放新用户专享优惠劵。
   - 站内信模块：因为是新用户，所以发送新用户的欢迎语的站内信。
   - ... 等等

   这样，我们就可以将注册成功后的业务拓展逻辑，实现业务上的解耦，未来也更加容易拓展。同时，也提高了注册接口的性能，避免用户需要等待业务拓展逻辑执行完成后，才响应注册成功。

6. 测试

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest(classes = App.class)
   class OrderProducerTest {
   
       private Logger logger = LoggerFactory.getLogger(getClass());
   
       @Autowired
       private OrderProducer producer;
   
       @Test
       public void syncSend() throws ExecutionException, InterruptedException {
           int id = (int) (System.currentTimeMillis() / 1000);
           SendResult result = producer.syncSend(id);
           logger.info("[testSyncSend][发送编号：[{}] 发送结果：[{}]]", id, result);
   
           // 阻塞等待，保证消费
           new CountDownLatch(1).await();
       }
   
       @Test
       void asyncSend() throws InterruptedException {
           int id = (int) (System.currentTimeMillis() / 1000);
           producer.asyncSend(id).addCallback(new ListenableFutureCallback<SendResult<Object, Object>>() {
   
               @Override
               public void onFailure(Throwable e) {
                   logger.info("[testASyncSend][发送编号：[{}] 发送异常]]", id, e);
               }
   
               @Override
               public void onSuccess(SendResult<Object, Object> result) {
                   logger.info("[testASyncSend][发送编号：[{}] 发送成功，结果为：[{}]]", id, result);
               }
   
           });
   
           // 阻塞等待，保证消费
           new CountDownLatch(1).await();
       }
   }
   ```

7. 运行：先运行主应用，在运行测试方法

KafkaListener：设置每个 Kafka 消费者 Consumer 的消息监听器的配置。

```java
/**
 * 监听的 Topic 数组
 */
String[] topics() default {};
/**
 * 监听的 Topic 表达式
 */
String topicPattern() default "";
/**
 * @TopicPartition 注解的数组。每个 @TopicPartition 注解，可配置监听的 Topic、队列、消费的开始位置
 */
TopicPartition[] topicPartitions() default {};

/**
 * 消费者分组
 */
String groupId() default "";

/**
 * 使用消费异常处理器 KafkaListenerErrorHandler 的 Bean 名字
 */
String errorHandler() default "";

/**
 * 自定义消费者监听器的并发数，使用线程池处理消费
 */
String concurrency() default "";

/**
 * 是否自动启动监听器。默认情况下，为 true 自动启动。
 */
String autoStartup() default "";

/**
 * Kafka Consumer 拓展属性。
 */
String[] properties() default {};
```

#### 批量发送消息

Kafka 提供的批量发送消息，它提供了一个 [RecordAccumulator](http://people.apache.org/~nehanarkhede/kafka-0.9-producer-javadoc/doc/org/apache/kafka/clients/producer/internals/RecordAccumulator.html) 消息收集器，将发送给相同 Topic 的相同 Partition 分区的消息们，“**偷偷**”收集在一起，当满足条件时候，一次性批量发送提交给 Kafka Broker 。如下是三个条件，满足**任一**即会批量发送：

- 【数量】`batch-size` ：超过收集的消息数量的最大条数。
- 【空间】`buffer-memory` ：超过收集的消息占用的最大内存。
- 【时间】`linger.ms` ：超过收集的时间的最大等待时长，单位：毫秒。

```yml
spring:
  # Kafka 配置项，对应 KafkaProperties 配置类
  kafka:
    batch-size: 16384 # 每次批量发送消息的最大数量
    buffer-memory: 33554432 # 每次批量发送消息的最大内存
    properties:
    	linger:
    		ms: 30000 # 批处理延迟时间上限。这里配置为 30 * 1000 ms 过后，不管是否消息数量是否到达 batch-size 或者消息大小到达 buffer-memory 后，都直接发送一次请求
```

其他内容与上面相同，测试

```java
@Test
    public void testASyncSend() throws InterruptedException {
        logger.info("[testASyncSend][开始执行]");

        for (int i = 0; i < 3; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.asyncSend(id).addCallback(new ListenableFutureCallback<SendResult<Object, Object>>() {

                @Override
                public void onFailure(Throwable e) {
                    logger.info("[testASyncSend][发送编号：[{}] 发送异常]]", id, e);
                }

                @Override
                public void onSuccess(SendResult<Object, Object> result) {
                    logger.info("[testASyncSend][发送编号：[{}] 发送成功，结果为：[{}]]", id, result);
                }

            });

            // 故意每条消息之间，隔离 10 秒
            Thread.sleep(10 * 1000L);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }
//# 30 秒后，满足批量消息的最大等待时长，所以 3 条消息被 Producer 批量发送。
//# 因此我们配置的是 acks=1 ，需要等待发送成功后，才会回调 ListenableFutureCallback 的方法。
```

#### 批量消费消息

新增配置：

```yml
spring:
  # Kafka 配置项，对应 KafkaProperties 配置类
  kafka:
    consumer:
      fetch-max-wait: 10000 # poll 一次拉取的阻塞的最大时长，单位：毫秒。这里指的是阻塞拉取需要满足至少 fetch-min-size 大小的消息
      fetch-min-size: 10 # poll 一次消息拉取的最小数据量，单位：字节
      max-poll-records: 100 # poll 一次消息拉取的最大数量
    # Kafka Consumer Listener 监听器配置
    listener:
      type: BATCH # 监听器类型，默认为 SINGLE ，只监听单条消息。这里我们配置 BATCH ，监听多条消息，批量消费
```

#### 定时消息

**Kafka 并未提供定时消息的功能，需要我们自行拓展**。

可以考虑基于 MySQL 存储定时消息，Job 扫描到达时间的定时消息，发送给 Kafka 。

#### 消费重试

Spring-Kafka 提供**消费重试**的机制。在消息**消费失败**的时候，Spring-Kafka 会通过**消费重试**机制，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。

当然，Spring-Kafka 并不会无限重新投递消息给 Consumer 重新消费，而是在默认情况下，达到 N 次重试次数时，Consumer 还是消费失败时，该消息就会进入到**死信队列**。

> 死信队列用于处理无法被正常消费的消息。当一条消息初次消费失败，Spring-Kafka 会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，Spring-Kafka 不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。
>
> Spring-Kafka 将这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。后续，我们可以通过对死信队列中的消息进行重发，来使得消费者实例再次进行消费。

而在 Kafka 中，消费重试和死信队列，是由 Spring-Kafka 所封装提供的。

每条消息的失败重试，是可以配置一定的间隔时间。具体，我们在示例的代码中，来进行具体的解释。

1. KafkaConfiguration

   ```java
   @Configuration
   public class KafkaConfiguration {
   
       @Bean
       @Primary
       public ErrorHandler kafkaErrorHandler(KafkaTemplate<?, ?> template) {
           // <1> 创建 DeadLetterPublishingRecoverer 对象
           ConsumerRecordRecoverer recoverer = new DeadLetterPublishingRecoverer(template);
           // <2> 创建 FixedBackOff 对象
           BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
           // <3> 创建 SeekToCurrentErrorHandler 对象
           return new SeekToCurrentErrorHandler(recoverer, backOff);
       }
   
   }
   ```

   - pring-Kafka 的消费重试功能，通过实现自定义的

      

     SeekToCurrentErrorHandler

      

     ，在 Consumer 消费消息异常的时候，进行拦截处理：

     - 在重试小于最大次数时，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。
     - 在重试到达最大次数时，Consumer 还是消费失败时，该消息就会发送到死信队列。例如说，本小节我们测试的 Topic 是 `"DEMO_04"` ，则其对应的死信队列的 Topic 就是 `"DEMO_04.DLT"` ，即在原有 Topic 加上 `.DLT` 后缀，就是其死信队列的 Topic 。

   - `<1>` 处，创建 [DeadLetterPublishingRecoverer](https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/listener/DeadLetterPublishingRecoverer.java) 对象，它负责实现，在重试到达最大次数时，Consumer 还是消费失败时，该消息就会发送到死信队列。

   - `<2>` 处，创建 [FixedBackOff](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/util/backoff/FixedBackOff.java) 对象。这里，我们配置了重试 3 次，每次固定间隔 30 秒。当然，胖友可以选择 [BackOff](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/util/backoff/BackOff.java) 的另一个子类 [ExponentialBackOff](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/util/backoff/ExponentialBackOff.java) 实现，提供[指数递增的间隔时间](http://note.huangz.me/algorithm/arithmetic/exponential-backoff.html)。

   - `<3>` 处，创建 SeekToCurrentErrorHandler 对象，负责处理异常，串联整个消费重试的整个过程。

2. 批量消费失败的消费重试处理

   ```java
   @Bean
   @Primary
   public BatchErrorHandler kafkaBatchErrorHandler() {
       // 创建 SeekToCurrentBatchErrorHandler 对象
       SeekToCurrentBatchErrorHandler batchErrorHandler = new SeekToCurrentBatchErrorHandler();
       // 创建 FixedBackOff 对象
       BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
       batchErrorHandler.setBackOff(backOff);
       // 返回
       return batchErrorHandler;
   }
   ```

#### 广播消费

在上述的示例中，我们看到的都是使用集群消费。而在一些场景下，我们需要使用**广播消费**。

> 广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

- 不过 Kafka 并不直接提供**内置的**广播消费的功能！！！此时，我们只能退而求其次，每个 Consumer **独有**一个 Consumer Group ，从而保证都能接收到全量的消息。

例如说，在应用中，缓存了数据字典等配置表在内存中，可以通过 Kafka 广播消费，实现每个应用节点都消费消息，刷新本地内存的缓存。

又例如说，我们基于 WebSocket 实现了 IM 聊天，在我们给用户主动发送消息时，因为我们不知道用户连接的是哪个提供 WebSocket 的应用，所以可以通过 Kafka 广播消费，每个应用判断当前用户是否是和自己提供的 WebSocket 服务连接，如果是，则推送消息给用户。

修改了配置项 `spring.kafka.consumer.auto-offset-reset=latest`。因为在广播订阅下，我们一般情况下，无需消费历史的消息，而是从订阅的 Topic 的队列的尾部开始消费即可，所以配置为 `latest` 。