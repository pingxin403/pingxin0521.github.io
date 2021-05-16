---
title: 定时任务调度框架 概述
date: 2019-4-21 12:19:59
tags:
 - Java
 - 框架
 - 任务调度
categories:
 - Java
 - 框架
---

> 任务调度是指基于给定时间点，给定时间间隔或者给定执行次数自动执行任务。

**业务场景：**

定时执行的任务（单次、有规律的多次、无规律的多次），单机和集群版

**需求和痛点：**

1. 集群部署服务时，如何确保任务不被重复执行？---最急迫
2. 如何监控、告警等；
3. 高可用、无单点故障；
4. 优秀的并行处理能力、分片能力；

<!--more-->

### 单机实现方案

下面简要介绍四种任务调度的 Java 实现：

- Timer（不推荐）
- ScheduledExecutor
- spring注解`@Scheduled`
- 开源工具包 Quartz

#### Timer

使用 Timer 实现任务调度的核心类是 Timer 和 TimerTask。其中 Timer 负责设定 TimerTask 的起始与间隔执行时间。使用者只需要创建一个 TimerTask 的继承类，实现自己的 run 方法，然后将其丢给 Timer 去执行即可。

Timer 的设计核心是一个 TaskList 和一个 TaskThread。Timer 将接收到的任务丢到自己的 TaskList 中，TaskList 按照 Task 的最初执行时间进行排序。TimerThread 在创建 Timer 时会启动成为一个守护线程。这个线程会轮询所有任务，找到一个最近要执行的任务，然后休眠，当到达最近要执行任务的开始时间点，TimerThread 被唤醒并执行该任务。之后 TimerThread 更新最近一个要执行的任务，继续休眠。

```java
import java.util.Timer;
 import java.util.TimerTask;

 public class TimerTest extends TimerTask {

 private String jobName = "";

 public TimerTest(String jobName) {
 super();
 this.jobName = jobName;
 }

 @Override
 public void run() {
 System.out.println("execute " + jobName);
 }

 public static void main(String[] args) {
 Timer timer = new Timer();
 long delay1 = 1 * 1000;
 long period1 = 1000;
 // 从现在开始 1 秒钟之后，每隔 1 秒钟执行一次 job1
 timer.schedule(new TimerTest("job1"), delay1, period1);
 long delay2 = 2 * 1000;
 long period2 = 2000;
 // 从现在开始 2 秒钟之后，每隔 2 秒钟执行一次 job2
 timer.schedule(new TimerTest("job2"), delay2, period2);
 }
 }
 Output:
 execute job1
 execute job1
 execute job2
 execute job1
 execute job1
 execute job2
```

**优点**：JDK本身就自带该工具类，无需第三方依赖，只需实现TimerTask类即可使用Timer进行调度配置，使用起来简单方便。

**缺点：**Timer中所有的任务都一个TaskThread线程来调度和执行，任务的执行方式是串行的，如果前一个任务发生延迟或异常会影响到后续任务的执行。

#### ScheduledExecutor

鉴于 Timer 的上述缺陷，Java 5 推出了基于线程池设计的 ScheduledThreadPoolExecutor。其设计思想是，每一个被调度的任务都会由线程池中一个线程去执行，因此任务是并发执行的，相互之间不会受到干扰。需要注意的是，只有当任务的执行时间到来时，ScheduledThreadPoolExecutor才会真正启动一个线程，其余时间 ScheduledThreadPoolExecutor都是在轮询任务的状态。

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledExecutorTest implements Runnable {
    private String jobName = "";

    public ScheduledExecutorTest(String jobName) {
        super();
        this.jobName = jobName;
    }

    @Override
    public void run() {
        System.out.println("execute " + jobName);
    }

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(10);

        long initialDelay1 = 1;
        long period1 = 1;
        // 从现在开始1秒钟之后，每隔1秒钟执行一次job1
        service.scheduleAtFixedRate(
                new ScheduledExecutorTest("job1"), initialDelay1,
                period1, TimeUnit.SECONDS);

        long initialDelay2 = 1;
        long delay2 = 1;
        // 从现在开始2秒钟之后，每隔2秒钟执行一次job2
        service.scheduleWithFixedDelay(
                new ScheduledExecutorTest("job2"), initialDelay2,
                delay2, TimeUnit.SECONDS);
    }
}
Output:
execute job1
execute job1
execute job2
execute job1
execute job1
execute job2
```

ScheduleAtFixedRate 和 ScheduleWithFixedDelay。ScheduleAtFixedRate 每次执行时间为上一次任务开始起向后推一个时间间隔，即每次执行时间为 :`initialDelay, initialDelay+period, initialDelay+2*period,…`；ScheduleWithFixedDelay 每次执行时间为上一次任务结束起向后推一个时间间隔，即每次执行时间为：`initialDelay, initialDelay+executeTime+delay, initialDelay+2*executeTime+2*delay`。由此可见，ScheduleAtFixedRate 是基于固定时间间隔进行任务调度，ScheduleWithFixedDelay 取决于每次任务执行的时间长短，是基于不固定时间间隔进行任务调度。

#### Spring的@Scheduled注解

`@Scheduled` 注解用于标注这个方法是一个定时任务的方法，有多种配置可选。cron支持cron表达式，指定任务在特定时间执行；fixedRate以特定频率执行任务；fixedRateString以string的形式配置执行频率。**默认为串行执行**

1. 先在启动类上添加 @EnableScheduling 注解开始支持。
2. 然后在需要执行的`public void`方法上添加`@Scheduled`,并且只能指定一种配置方法

**参数说明**

- initial-delay : 表示第一次运行前需要延迟的时间，单位是毫秒
- fixed-delay : 表示从上一个任务完成到下一个任务开始的间隔, 单位是毫秒。
- fixed-rate : 表示从上一个任务开始到下一个任务开始的间隔, 单位是毫秒。(如果上一个任务执行超时，则可能是上一个任务执行完成后立即启动下一个任务)
- cron : cron 表达式。(定时执行，如果上一次任务执行超时而导致某个定时间隔不能执行，则会顺延下一个定时间隔时间。下一个任务和上一个任务的间隔时间不固定)

**并行调度实现**：

要在 Spring 中实现并行调度，可以新建一个配置类来实现 `SchedulingConfigurer` 接口，也可以使用 `@Async` 注解。下面分别进行介绍。

1. 实现 `SchedulingConfigurer`

   ```java
   import java.util.concurrent.Executor;
   import java.util.concurrent.Executors;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.scheduling.annotation.EnableScheduling;
   import org.springframework.scheduling.annotation.SchedulingConfigurer;
   import org.springframework.scheduling.config.ScheduledTaskRegistrar;
   
   @Configuration
   @EnableScheduling
   public class ScheduleConfig implements SchedulingConfigurer {
   
       @Override
       public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
           taskRegistrar.setScheduler(taskExecutor());
       }
   
       @Bean(destroyMethod = "shutdown")
       public Executor taskExecutor() {
           return Executors.newScheduledThreadPool(10);
       }
       
   }
   ```

2. 使用 `@Async`

   启动类添加 @EnableAsync 开启异步方法支持。然后在方法上添加 @Async 注解。

   使用默认的 @Async 配置，线程池开启的线程挺多的。我们可以通过实现 AsyncConfigurer 接口来进行灵活的设置。

   ```java
   import java.lang.reflect.Method;
   import java.util.concurrent.Executor;
   import java.util.concurrent.ThreadFactory;
   import java.util.concurrent.atomic.AtomicInteger;
   
   import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.scheduling.annotation.AsyncConfigurer;
   import org.springframework.scheduling.annotation.EnableAsync;
   import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
   import org.springframework.util.StringUtils;
   
   import lombok.extern.slf4j.Slf4j;
   
   @Configuration
   @Slf4j
   public class AsyncConfig implements AsyncConfigurer {
   
       // 自定义线程池
       @Override
       public Executor getAsyncExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           executor.setCorePoolSize(10);
           executor.setMaxPoolSize(20);
           executor.setQueueCapacity(100);
           // 设置线程名称前缀（前缀+线程序号作为最终的线程名称）
           executor.setThreadNamePrefix("AsyncExecT-");
           // 设置拒绝策略
           //executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
           executor.initialize(); //如果不初始化，导致找到不到执行器
           //ExecutorService executor = Executors.newFixedThreadPool(10, new MyNamedThreadFactory("PHS"));
           return executor;
       }
   }
   ```

#### Spring Quartz Scheduler

Spring Quartz是在Spring框架中使用Quartz工具来实现任务调度的方式，在Spring下对Quartz可以方便的完成任务调度需要的配置。

Quartz是一个相对上述两种调度工具更为复杂的任务调度系统，使用Trigger, Job 和JobDetail对象来实现对各种类型任务的调度。Spring也需要配置好这些对象，才能使用Quartz的任务调度功能。

Spring提供了一个JobDetailFactoryBean用于配置JobDetail，在JobDetail中包含了所有运行job (ExampleJob)需要的信息。

1. 引入maven依赖

   ```xml
   <!-- quartz -->
   		<dependency>
   			<groupId>org.quartz-scheduler</groupId>
   			<artifactId>quartz</artifactId>
   			<version>2.2.1</version>
   		</dependency>
   		<dependency>
   			<groupId>org.quartz-scheduler</groupId>
   			<artifactId>quartz-jobs</artifactId>
   			<version>2.2.1</version>
   		</dependency>
   
   ```

2. 任务调度类

   ```java
   public class MyJob implements Job {
   	publicvoid execute(JobExecutionContext context) throws JobExecutionException {
   		System.out.println("quartz MyJob date:" + new Date().getTime());
   	}
   }
   ```

3. 启动任务类

   ```java
   //1.创建Scheduler的工厂
         SchedulerFactory sf = new StdSchedulerFactory();
   //2.从工厂中获取调度器实例
         Scheduler scheduler = sf.getScheduler();
    
    
   //3.创建JobDetail
         JobDetail jb = JobBuilder.newJob(MyJob.class)
                 .withDescription("this is a ram job") //job的描述
                 .withIdentity("ramJob", "ramGroup") //job 的name和group
                 .build();
    
   //任务运行的时间，SimpleSchedle类型触发器有效
   longtime=  System.currentTimeMillis() + 3*1000L; //3秒后启动任务
         Date statTime = new Date(time);
    
   //4.创建Trigger
   //使用SimpleScheduleBuilder或者CronScheduleBuilder
         Trigger t = TriggerBuilder.newTrigger()
                     .withDescription("")
                     .withIdentity("ramTrigger", "ramTriggerGroup")
   //.withSchedule(SimpleScheduleBuilder.simpleSchedule())
                     .startAt(statTime)  //默认当前时间启动
                     .withSchedule(CronScheduleBuilder.cronSchedule("0/2 * * * * ?")) //两秒执行一次
                     .build();
    
   //5.注册任务和定时器
   scheduler.scheduleJob(jb, t);
    
   //6.启动调度器
   scheduler.start();
   ```

### 分布式

单机版任务定时执行会在分布式环境下出现重复执行的情况，而且分布式环境下需要更多的运维数据和程序执行情况，因此需要对任务的执行情况进行记录和调控。

#### [ShedLock](https://github.com/lukas-krecan/ShedLock)

与Spring的@Scheduled注解结合，使用AOP注解的方式，可使用MongoDB、JDBC-DB、Redis或Zookeeper等来实现分布式锁，具体采用哪种方式，由使用者决定；**它仅仅是一个分布式锁，并不是调度程序**；

```java
//示例：与Spring的原生注解 @Scheduled配合使用
//在启动类添加注解
@EnableSchedulerLock(defaultLockAtMostFor = "10m")

//添加注解到任务
import net.javacrumbs.shedlock.core.SchedulerLock;

@Scheduled(cron = "0 */15 * * * *")//每15分钟运行一次
@SchedulerLock(name = "scheduledTaskName", lockAtMostFor = "14m", lockAtLeastFor = "14m")
public void scheduledTask() {
   // do something
}
//lockAtMostFor : 表示最多锁定14分钟，主要用于防止执行任务的节点挂掉（即使这个节点挂掉，在14分钟后，锁也被释放），一般将其设置为明显大于任务的最大执行时长；如果任务运行时间超过该值（即任务14分钟没有执行完），则该任务可能被重复执行！
//lockAtLeastFor : 至少锁定时长，确保在15分钟内，该任务不会运行超过1次；
```

**JdbcTemplate**

1. 创建数据库表

   ```sql
   # MySQL, MariaDB
   CREATE TABLE shedlock(name VARCHAR(64) NOT NULL, lock_until TIMESTAMP(3) NOT NULL,
       locked_at TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3), locked_by VARCHAR(255) NOT NULL, PRIMARY KEY (name));
   
   # Postgres
   CREATE TABLE shedlock(name VARCHAR(64) NOT NULL, lock_until TIMESTAMP NOT NULL,
       locked_at TIMESTAMP NOT NULL, locked_by VARCHAR(255) NOT NULL, PRIMARY KEY (name));
   
   # Oracle
   CREATE TABLE shedlock(name VARCHAR(64) NOT NULL, lock_until TIMESTAMP(3) NOT NULL,
       locked_at TIMESTAMP(3) NOT NULL, locked_by VARCHAR(255) NOT NULL, PRIMARY KEY (name));
   
   # MS SQL
   CREATE TABLE shedlock(name VARCHAR(64) NOT NULL, lock_until datetime2 NOT NULL,
       locked_at datetime2 NOT NULL, locked_by VARCHAR(255) NOT NULL, PRIMARY KEY (name));
   
   # DB2
   CREATE TABLE shedlock(name VARCHAR(64) NOT NULL PRIMARY KEY, lock_until TIMESTAMP NOT NULL,
       locked_at TIMESTAMP NOT NULL, locked_by VARCHAR(255) NOT NULL);
   ```

2. 添加依赖

   ```xml
   <dependency>
       <groupId>net.javacrumbs.shedlock</groupId>
       <artifactId>shedlock-provider-jdbc-template</artifactId>
       <version>4.15.1</version>
   </dependency>
   ```

3. 配置

   ```java
   import net.javacrumbs.shedlock.provider.jdbctemplate.JdbcTemplateLockProvider;
   
   ...
   @Bean
   public LockProvider lockProvider(DataSource dataSource) {
               return new JdbcTemplateLockProvider(
                   JdbcTemplateLockProvider.Configuration.builder()
                   .withJdbcTemplate(new JdbcTemplate(dataSource))
                   .usingDbTime() // Works on Postgres, MySQL, MariaDb, MS SQL, Oracle, DB2, HSQL and H2
                   .build()
               );
   }
   ```

可以简单解决多实例重复运行任务的问题，同时需要使用中间件来共享任务执行的结果，但是本质上是一个分布式锁，而不是任务调度框架。

#### Quartz in Jdbc

Java事实上的定时任务标准。但Quartz关注点在于定时任务而非数据，并无一套根据数据处理而定制化的流程。虽然Quartz可以基于数据库实现作业的高可用，但缺少分布式并行调度的功能

独立的Quratz节点之间是不需要通信的，不同节点之间是通过数据库表来感知另一个应用，只有使用持久的JobStore才能完成Quartz集群。如果某一个节点失效，那么Job会在其他节点上执行。

保证只在一台机器上触发：数据库悲观锁；一旦某一节点线程获取了该锁，那么Job就会在这台机器上被执行，其他节点进行锁等待；

**缺点**

1. 不适合大量的短任务 & 不适合过多节点部署；
2. 解决了高可用的问题，并没有解决任务分片的问题，存在单机处理的极限（即：不能实现水平扩展）。
3. 需要把任务信息持久化到业务数据表，和业务有耦合
4. 调度逻辑和执行逻辑并存于同一个项目中，在机器性能固定的情况下，业务和调度之间不可避免地会相互影响。
5. quartz集群模式下，是通过数据库独占锁来唯一获取任务，任务执行并没有实现完善的负载均衡机制。

#### [Elastic-Job](http://shardingsphere.apache.org/elasticjob/index_zh.html)

[ElasticJob](https://shardingsphere.apache.org/elasticjob/current/cn/overview/) 是一个分布式调度解决方案，由 2 个相互独立的子项目 ElasticJob-Lite 和 ElasticJob-Cloud 组成。

ElasticJob-Lite 定位为轻量级无中心化解决方案，使用jar的形式提供分布式任务的协调服务；
ElasticJob-Cloud 使用 Mesos 的解决方案，额外提供资源治理、应用分发以及进程隔离等服务。

ElasticJob 的各个产品使用统一的作业 API，开发者仅需要一次开发，即可随意部署。

Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务；Elastic-Job-Cloud采用自研Mesos Framework的解决方案，额外提供资源治理、应用分发以及进程隔离等功能；

Elastic-Job-Lite并没有宿主程序，而是基于部署作业框架的程序在到达相应时间点时各自触发调度。它的开发也比较简单，引用Jar包实现一些方法即可，最后编译成Jar包运行。Elastic-Job-Lite的分布式部署全靠ZooKeeper来同步状态和原数据。实现高可用的任务只需将分片总数设置为1，并把开发的Jar包部署于多个服务器上执行，任务将会以1主N从的方式执行。一旦本次执行任务的服务器崩溃，其他执行任务的服务器将会在下次作业启动时选择一个替补执行。如果开启了失效转移，那么功能效果更好，可以保证在本次作业执行时崩溃，备机之一立即启动替补执行。

Elastic-Job-Lite的任务分片也是通过ZooKeeper来实现，Elastic-Job并不直接提供数据处理的功能，框架只会将分片项分配至各个运行中的作业服务器，开发者需要自行处理分片项与真实数据的对应关系。框架也预置了一些分片策略：平均分配算法策略，作业名哈希值奇偶数算法策略，轮转分片策略。同时也提供了自定义分片策略的接口。

另外Elastic-Job-Lite还提供了一个任务监控和管理界面：Elastic-Job-Lite-Console。它和Elastic-Job-Lite是两个完全不关联的应用程序，使用ZooKeeper来交换数据，管理人员可以通过这个界面查看、监控和管理Elastic-Job-Lite的任务，必要的时候还能手动触发任务

**功能列表**

- 分布式调度协调
- 弹性扩容缩容
- 失效转移
- 错过执行作业重触发
- 作业分片一致性，保证同一分片在分布式环境中仅一个执行实例
- 自诊断并修复分布式不稳定造成的问题
- 支持并行调度
- 支持作业生命周期操作
- 丰富的作业类型
- Spring整合以及命名空间提供
- 运维平台

**优缺点**

优点：

- 基于成熟的定时任务作业框架Quartz cron表达式执行定时任务；
- 支持任务分片：可以拆分任务，分别由不同节点执行；
- 官网文档齐全，全中文；
- 弹性扩容缩容：运行中的作业服务器崩溃，或新增N台作业服务器，作业框架将在下次作业执行前重新分片，不影响当前作业执行；
- 任务监控和管理界面；

缺点：

- 部署复杂

#### [XXL-JOB](https://www.xuxueli.com/xxl-job/)

[XXL-Job官网](https://github.com/xuxueli/xxl-job)是大众点评员工徐雪里于2015年发布的分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。

将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

![B5aVzQ.png](https://s1.ax1x.com/2020/11/07/B5aVzQ.png)

**系统组成**

- 调度模块（调度中心）：

  负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；

  支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。

- 执行模块（执行器）：

  负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；

  接收“调度中心”的执行请求、终止请求和日志请求等。

