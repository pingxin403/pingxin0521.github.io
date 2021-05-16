---
title: Spring Cloud 组件之Hystrix (五)
date: 2019-11-1 18:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

全部代码参考：<https://github.com/hanyunpeng0521/spring-cloud-learn/tree/master/part4_hystrix>

### 容错处理-基于Hystrix

在微服务架构中，我们将系统拆分成了一个个的服务单元，各单元间通过服务注册与订阅的方式互相依赖。由于每个单元都在不同的进程中运行，依赖通过远程调用的方式执行，这样就有可能因为网络原因或是依赖服务自身问题出现调用故障或延迟，而这些问题会直接导致调用方的对外服务也出现延迟，若此时调用方的请求不断增加，最后就会出现因等待出现故障的依赖方响应而形成任务积压，最终导致自身服务的瘫痪。

<!--more-->

举个例子，在一个电商网站中，我们可能会将系统拆分成，用户、订单、库存、积分、评论等一系列的服务单元。用户创建一个订单的时候，在调用订单服务创建订单的时候，会向库存服务来请求出货（判断是否有足够库存来出货）。此时若库存服务因网络原因无法被访问到，导致创建订单服务的线程进入等待库存申请服务的响应，在漫长的等待之后用户会因为请求库存失败而得到创建订单失败的结果。如果在高并发情况之下，因这些等待线程在等待库存服务的响应而未能释放，使得后续到来的创建订单请求被阻塞，最终导致订单服务也不可用。

在微服务架构中，存在着那么多的服务单元，若一个单元出现故障，就会因依赖关系形成故障蔓延，最终导致整个系统的瘫痪，这样的架构相较传统架构就更加的不稳定。为了解决这样的问题，因此产生了断路器模式。

#### 什么是断路器

断路器模式源于Martin Fowler的[Circuit Breaker](http://martinfowler.com/bliki/CircuitBreaker.html)一文。“断路器”本身是一种开关装置，用于在电路上保护线路过载，当线路中有电器发生短路时，“断路器”能够及时的切断故障电路，防止发生过载、发热、甚至起火等严重后果。

在分布式架构中，断路器模式的作用也是类似的，当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。

在springcloud中断路器组件就是Hystrix。Hystrix也是Netflix套件的一部分。他的功能是，当对某个服务的调用在一定的时间内（默认10s，由metrics.rollingStats.timeInMilliseconds配置），有超过一定次数（默认20次，由circuitBreaker.requestVolumeThreshold参数配置）并且失败率超过一定值（默认50%，由circuitBreaker.errorThresholdPercentage配置），该服务的断路器会打开。返回一个由开发者设定的fallback

![1JVjp9.png](https://s2.ax1x.com/2020/02/01/1JVjp9.png)

fallback可以是另一个由Hystrix保护的服务调用，也可以是固定的值。fallback也可以设计成链式调用，先执行某些逻辑，再返回fallback。

#### Netflix Hystrix

在Spring Cloud中使用了[Hystrix](https://github.com/Netflix/Hystrix) 来实现断路器的功能。Hystrix是Netflix开源的微服务框架套件之一，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。

下面我们来看看如何使用Hystrix。基于上面的示例进行修改，

在消费者的主类`MovieApplication`中使用`@EnableHystrix`注解开启断路器功能,如果使用Feign，配置`@EnableFeignClients`后不需要再加上`@EnableHystrix`。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

1. 针对普通的方法，只需加上HystrixCommand注解及定义回退方法即可，例如：

   ```java
   @HystrixCommand(fallbackMethod = "findByIdFallback")
       @GetMapping(value = "/user/{id}")
       public User findById(@PathVariable Long id) {
           return restTemplate.getForObject("http://user-service/" + id, User.class);
       }
   
       public User findByIdFallback(Long id){
           User user = new User();
           user.setId(-1L);
           user.setUsername("Default User");
   
           return user;
       }
   ```

2. Feign使用Hystrix,针对Feign，它是以接口形式工作的，好在Spring Cloud已默认为Feign整合了Hystrix，不过默认是关闭的，需要手动在配置文件中开启：

   ```yml
   feign:
     hystrix:
       enabled: true
   ```

   然后直接在FeignClient注解中加入fallback属性即可，例如下面所示：

   ```java
   @FeignClient(name = "user-service", fallback = FeignClientFallback.class)
   public interface UserFeignClient {
       @GetMapping(value = "/{id}")
       User findById(@PathVariable("id") Long id);
   }
   
   @Component
   class FeignClientFallback implements UserFeignClient{
       @Override
       public User findById(Long id) {
           User user = new User();
           user.setId(-1L);
           user.setUsername("Default User");
   
           return user;
       }
   }
   ```

   如果想要在Feign发生回退时能够留下日志供查看回退原因，那么可以使用FallbackFactory

   ```java
   @FeignClient(name = "user-service", fallbackFactory = FeignClientFallbackFactory.class)
   public interface UserFeignClient {
       @GetMapping(value = "/{id}")
       User findById(@PathVariable("id") Long id);
   }
   
   @Component
   class FeignClientFallbackFactory implements FallbackFactory<UserFeignClient> {
       private static final Logger LOGGER =
               LoggerFactory.getLogger(FeignClientFallbackFactory.class);
   
       @Override
       public UserFeignClient create(Throwable cause) {
           return new UserFeignClient() {
               @Override
               public User findById(Long id) {
                   /*
                    * 日志最好放在各个fallback中，而不要直接放在create方法中
                    * 否则在引用启动时，就会打印该日志
                    */
                   FeignClientFallbackFactory.LOGGER.info("sorry, fallback. reason was: ", cause);
                   User user = new User();
                   user.setId(-1L);
                   user.setUsername("Default Username");
   
                   return user;
               }
           };
       }
   }
   ```



