---
title: Spring boot 消息队列(使用RabbitMQ)
date: 2019-12-10 08:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

在大多应用中,我们系统之间需要进行异步通信,即异步消息。

消息服务中两个重要概念： 消息代理（message broker）和目的地（destination） 当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目 的地。

<!--more-->

消息队列主要有两种形式的目的地

1. 队列（queue）：点对点消息通信（point-to-point）
2. 主题（topic）：发布（publish）/订阅（subscribe）消息通信

功能

1. 异步通信

   ![lNTcz8.png](https://s2.ax1x.com/2020/01/03/lNTcz8.png)

2. 应用解耦

   ![lNT2QS.png](https://s2.ax1x.com/2020/01/03/lNT2QS.png)

3. 流量削峰

   ![lNTRsg.png](https://s2.ax1x.com/2020/01/03/lNTRsg.png)

点对点式： 

- 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容， 消息读取后被移出队列 
- 消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

发布订阅式：

- 发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么 就会在消息到达时同时收到消息

**JMS**（Java Message Service）JAVA消息服务：

- 基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

**AMQP**（Advanced Message Queuing Protocol）

- 高级消息队列协议，也是一个消息代理的规范，兼容JMS
- RabbitMQ是AMQP的实现

扩展阅读：既然有高级的消息协议，必然有简单的协议，STOMP（Simple (or Streaming) Text Orientated Messaging Protocol），也就是简单消息文本协议，猛击[这里](http://stomp.github.io/)

![lNTWLQ.png](https://s2.ax1x.com/2020/01/03/lNTWLQ.png)

**MSMQ**

这里附带介绍一下MSMQ。.NET开发者接触最多的可能还是这个消息队列，我知道有两个以.NET作为主要开发语言的公司都选择MSMQ来开发公共框架如ESB、日志组件等。

如果你有.NET下MSMQ（微软消息队列）开发和使用经验，一定不会对队列常用术语陌生。对比一下，对后面RabbitMQ的学习和理解非常有帮助。

![1AT8Nd.png](https://s2.ax1x.com/2020/01/22/1AT8Nd.png)

**Spring支持**

-  spring-jms提供了对JMS的支持
- spring-rabbit提供了对AMQP的支持
- 需要ConnectionFactory的实现来连接消息代理
- 提供JmsTemplate、RabbitTemplate来发送消息
- @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发 布的消息
- @EnableJms、@EnableRabbit开启支持

RabbitMQ相关参考:<>

#### RabbitMQ整合spring-boot

1. 引入 spring-boot-starter-amqp

   ```xml
           <!--rabbitmq-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-amqp</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   ```

2. application.yml配置

3. 测试RabbitMQ
   1. AmqpAdmin：管理组件
   2. RabbitTemplate：消息发送处理组件

4. 按照下面配置消息队列

![lNTHzT.png](https://s2.ax1x.com/2020/01/03/lNTHzT.png)

application.properties

```properties
spring.rabbitmq.host=192.168.43.196
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
#虚拟host 可以不设置,使用server默认host
spring.rabbitmq.virtual-host=my_vhost
```

**RabbitmqApplication**

```java
/**
 * 自动配置
 *  1、RabbitAutoConfiguration
 *  2、有自动配置了连接工厂ConnectionFactory；
 *  3、RabbitProperties 封装了 RabbitMQ的配置
 *  4、 RabbitTemplate ：给RabbitMQ发送和接受消息；
 *  5、 AmqpAdmin ： RabbitMQ系统管理功能组件;
 *  	AmqpAdmin：创建和删除 Queue，Exchange，Binding
 *  6、@EnableRabbit +  @RabbitListener 监听消息队列的内容
 *
 */
@EnableRabbit  //开启基于注解的RabbitMQ模式
@SpringBootApplication
public class RabbitmqApplication {

    public static void main(String[] args) {
        SpringApplication.run(RabbitmqApplication.class, args);
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Book {
    private String name;
    private String author;
}

```

RabbitmqApplicationTests

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {RabbitmqApplication.class})
public class AppTest {
    @Autowired
    RabbitTemplate rabbitTemplate;

    @Autowired
    AmqpAdmin amqpAdmin;

    @Test
    public void createExchange() {

//		amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
//		System.out.println("创建完成");

//		amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
        //创建绑定规则

//		amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));

        //amqpAdmin.de
    }

    //单播（点对点）
    @Test
    public void contextLoads() {
        //Message需要自己构造一个；定义消息体内容和消息头
        //rabbitTemplate.send(exchage,routeKey,message);

        //object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq
        //rabbitTemplate.convertAndSend(exchage,routKey,object);
        Map<String, String> map = new HashMap();
        map.put("a", "zs");
        map.put("b", "ls");
        //对象被默认序列化以后发送出去
        rabbitTemplate.convertAndSend("exchange.direct", "hyp.new", new Book("西游记", "吴承恩").toString());
    }

    //接收数据
    @Test
    public void receive() {

        Object o = rabbitTemplate.receiveAndConvert("hyp.new");
//        class java.util.HashMap
//        {a=zs, b=ls}
        System.out.println(o.getClass());
        System.out.println(o);
    }


    //广播
    @Test
    public void sendMsg() {
        rabbitTemplate.convertAndSend("exchange.fanout", "", new Book("mnmn", "kjkj").toString());
    }
}
```

#### Spring Boot自动配置

参看：org.springframework.boot.autoconfigure.amqp下的配置包

1. RabbitAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ RabbitTemplate.class, Channel.class })
   //相关配置所在类
   @EnableConfigurationProperties(RabbitProperties.class)
   //实际配置类
   @Import(RabbitAnnotationDrivenConfiguration.class)
   public class RabbitAutoConfiguration {
   
       //连接Factory
   	@Configuration(proxyBeanMethods = false)
   	@ConditionalOnMissingBean(ConnectionFactory.class)
   	protected static class RabbitConnectionFactoryCreator {
   		//缓存连接工厂
   		@Bean
   		public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties properties,
   				ObjectProvider<ConnectionNameStrategy> connectionNameStrategy) throws Exception {
   //...
   			return factory;
   		}
   
   		private RabbitConnectionFactoryBean getRabbitConnectionFactoryBean(RabbitProperties properties)
   //...
   			return factory;
   		}
   
   	}
   
   	@Configuration(proxyBeanMethods = false)
   	@Import(RabbitConnectionFactoryCreator.class)
   	protected static class RabbitTemplateConfiguration {
   
   		@Bean
   		@ConditionalOnSingleCandidate(ConnectionFactory.class)
   		@ConditionalOnMissingBean(RabbitOperations.class)
   		public RabbitTemplate rabbitTemplate(RabbitProperties properties,
   				ObjectProvider<MessageConverter> messageConverter,
   				ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers,
   				ConnectionFactory connectionFactory) {
   //...
   			return template;
   		}
   
   		private boolean determineMandatoryFlag(RabbitProperties properties) {
   			Boolean mandatory = properties.getTemplate().getMandatory();
   			return (mandatory != null) ? mandatory : properties.isPublisherReturns();
   		}
   //rabbitmq配置
   		@Bean
   		@ConditionalOnSingleCandidate(ConnectionFactory.class)
   		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic", matchIfMissing = true)
   		@ConditionalOnMissingBean
   		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
   			return new RabbitAdmin(connectionFactory);
   		}
   
   	}
   //操作器
   	@Configuration(proxyBeanMethods = false)
   	@ConditionalOnClass(RabbitMessagingTemplate.class)
   	@ConditionalOnMissingBean(RabbitMessagingTemplate.class)
   //模板配置类
   	@Import(RabbitTemplateConfiguration.class)
   	protected static class MessagingTemplateConfiguration {
   
   		@Bean
   		@ConditionalOnSingleCandidate(RabbitTemplate.class)
   		public RabbitMessagingTemplate rabbitMessagingTemplate(RabbitTemplate rabbitTemplate) {
   			return new RabbitMessagingTemplate(rabbitTemplate);
   		}
   
   	}
   
   }
   ```

2. RabbitProperties

   ```java
   @ConfigurationProperties(prefix = "spring.rabbitmq")
   public class RabbitProperties {
   
   	/**
   	 * RabbitMQ host.
   	 */
   	private String host = "localhost";
   
   	/**
   	 * RabbitMQ port.
   	 */
   	private int port = 5672;
   
   	/**
   	 * Login user to authenticate to the broker.
   	 */
   	private String username = "guest";
   
   	/**
   	 * Login to authenticate against the broker.
   	 */
   	private String password = "guest";
       //...
   	}
   ```

3. RabbitAnnotationDrivenConfiguration:Spring AMQP注释驱动端点的配置

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass(EnableRabbit.class)
   class RabbitAnnotationDrivenConfiguration {
   
   	private final ObjectProvider<MessageConverter> messageConverter;
   
   	private final ObjectProvider<MessageRecoverer> messageRecoverer;
   
   	private final ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers;
   
   	private final RabbitProperties properties;
   
   	RabbitAnnotationDrivenConfiguration(ObjectProvider<MessageConverter> messageConverter,
   			ObjectProvider<MessageRecoverer> messageRecoverer,
   			ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers, RabbitProperties properties) {
   		this.messageConverter = messageConverter;
   		this.messageRecoverer = messageRecoverer;
   		this.retryTemplateCustomizers = retryTemplateCustomizers;
   		this.properties = properties;
   	}
   
   	@Bean
   	@ConditionalOnMissingBean
   	SimpleRabbitListenerContainerFactoryConfigurer simpleRabbitListenerContainerFactoryConfigurer() {
   		SimpleRabbitListenerContainerFactoryConfigurer configurer = new SimpleRabbitListenerContainerFactoryConfigurer();
   		configurer.setMessageConverter(this.messageConverter.getIfUnique());
   		configurer.setMessageRecoverer(this.messageRecoverer.getIfUnique());
   		configurer.setRetryTemplateCustomizers(
   				this.retryTemplateCustomizers.orderedStream().collect(Collectors.toList()));
   		configurer.setRabbitProperties(this.properties);
   		return configurer;
   	}
   
   	@Bean(name = "rabbitListenerContainerFactory")
   	@ConditionalOnMissingBean(name = "rabbitListenerContainerFactory")
   	@ConditionalOnProperty(prefix = "spring.rabbitmq.listener", name = "type", havingValue = "simple",
   			matchIfMissing = true)
   	SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory(
   			SimpleRabbitListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
   		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
   		configurer.configure(factory, connectionFactory);
   		return factory;
   	}
   
   	@Bean
   	@ConditionalOnMissingBean
   	DirectRabbitListenerContainerFactoryConfigurer directRabbitListenerContainerFactoryConfigurer() {
   		DirectRabbitListenerContainerFactoryConfigurer configurer = new DirectRabbitListenerContainerFactoryConfigurer();
   		configurer.setMessageConverter(this.messageConverter.getIfUnique());
   		configurer.setMessageRecoverer(this.messageRecoverer.getIfUnique());
   		configurer.setRetryTemplateCustomizers(
   				this.retryTemplateCustomizers.orderedStream().collect(Collectors.toList()));
   		configurer.setRabbitProperties(this.properties);
   		return configurer;
   	}
   
   	@Bean(name = "rabbitListenerContainerFactory")
   	@ConditionalOnMissingBean(name = "rabbitListenerContainerFactory")
   	@ConditionalOnProperty(prefix = "spring.rabbitmq.listener", name = "type", havingValue = "direct")
   	DirectRabbitListenerContainerFactory directRabbitListenerContainerFactory(
   			DirectRabbitListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
   		DirectRabbitListenerContainerFactory factory = new DirectRabbitListenerContainerFactory();
   		configurer.configure(factory, connectionFactory);
   		return factory;
   	}
   
   	@Configuration(proxyBeanMethods = false)
   	@EnableRabbit
   	@ConditionalOnMissingBean(name = RabbitListenerConfigUtils.RABBIT_LISTENER_ANNOTATION_PROCESSOR_BEAN_NAME)
   	static class EnableRabbitConfiguration {
   
   	}
   
   }
   ```




### 参考

1. [Springboot 整合RabbitMq ，用心看完这一篇就够了](https://blog.csdn.net/qq_35387940/article/details/100514134)
2. [RabbitMQ实战](https://blog.csdn.net/u013871100/category_9278769.html)
3. https://www.rabbitmq.com/tutorials/tutorial-one-java.html

