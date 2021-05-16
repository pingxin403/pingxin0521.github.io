---
title: Spring boot 日志框架、异步任务、定时任务、邮件服务
date: 2019-06-30 17:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

### 日志

SpringBoot 日志配置 默认采用slf4j门面+LogBack日志作为日志输出！日志可参考：<https://hanyunpeng0521.github.io/categories/Java/%E6%97%A5%E5%BF%97/>

建议如果不是必要，继续使用默认的框架，因为支持比较好。

```xml
<!--默认的starter中保含logging -->
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring‐boot‐starter</artifactId>
</dependency>

<!-- 即-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring‐boot‐starter‐logging</artifactId>
</dependency>
```

底层依赖关系：

![](https://i.bmp.ovh/imgs/2019/12/6d2397a0b9e6077f.png)

总结:

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录

2. SpringBoot也把其他的日志都替换成了slf4j;

3. 中间替换包

   ```java
   @SuppressWarnings("rawtypes")
   public abstract class LogFactory {
   static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J =
   "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";
   static LogFactory logFactory = new SLF4JLogFactory();
   ```

   ![](https://i.bmp.ovh/imgs/2019/12/f531f3f4a6bdc32a.png)

4. 如果我们要引入其他框架?一定要把这个框架的默认日志依赖移除掉?

   Spring框架用的是commons-logging;

   ```xml
   <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring‐core</artifactId>
       <exclusions>
           <exclusion>
               <groupId>commons‐logging</groupId>
               <artifactId>commons‐logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

   **SpringBoot能自动适配所有的日志,而且底层使用slf4j+logback的方式记录日志,引入其他框架的时候,只需要把这个框架依赖的日志框架排除掉即可;**

使用时使用下面方法初始化

```java
//依赖
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

//类内
private final static Logger logger = LoggerFactory.getLogger(LogTest.class);

```

#### 配置

**默认配置**

SpringBoot默认帮我们配置好了日志;

```
//实现可以参考：org.springframework.boot.logging包下内容
```

**控制台输出级别**

在application.properties文件中配置

如果你的终端支持ANSI，设置彩色输出会让日志更具可读性。通过在`application.properties`中设置`spring.output.ansi.enabled`参数来支持。

- NEVER：禁用ANSI-colored输出（默认项）
- DETECT：会检查终端是否支持ANSI，是的话就采用彩色输出（推荐项）
- ALWAYS：总是使用ANSI-colored格式输出，若终端不支持的时候，会有很多干扰信息，不推荐使用

```properties
#多彩输出
spring.output.ansi.enabled=detect
#日志级别
logging.level.root=info
#所有包下面都以debug级别输出
logging.level.*=info
```

**默认输出格式**

可以通过 logging.pattern.console = 进行配置

**文件输出**

springboot默认会将日志输出到控制台，线上查看日志时会很不方便，一般我们都是输出到文件。

需要在application.properties配置

```properties
#日志输出路径问价 优先输出 logging.file
logging.file=C:/Users/tizzy/Desktop/img/my.log
 
#设置目录，会在该目录下创建spring.log文件，并写入日志内容，
logging.path=C:/Users/tizzy/Desktop/img/
 
#日志大小 默认10MB会截断，重新输出到下一个文件中，默认级别为：ERROR、WARN、INFO
logging.file.max-size=10MB
```

logging.file 和 logging.path 同时设置时候会优先使用logging.file 作为日志输出。

**自定义日志配置**

日志服务在ApplicationContext 创建之前就被初始化了，并不是采用Spring的配置文件进行控制。

那我们来如何进行自定义配置日志呢。

springboot为我们提供了一个规则，按照规则组织配置文件名，就可以被正确加载，SpringBoot就不使用他默认配置的了：

![](https://i.bmp.ovh/imgs/2019/12/294c0be083ed4da9.png)

logback.xml:直接就被日志框架识别了;

logback-spring.xml:日志框架就不直接加载日志的配置项,由SpringBoot解析日志配置,可以使用SpringBoot的高级Profile功能

```xml
<springProfile name="staging">
 <!‐‐ configuration to be enabled when the "staging" profile is active ‐‐>
 可以指定某段配置只在某个环境下生效
</springProfile>
```

如果使用logback.xml作为日志配置文件,还要使用profile功能,会有以下错误
`no applicable action for [springProfile]`

因此推荐使用`*-spring.*`配置文件进行配置

- Logback：`logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, `logback.groovy`

  默认，不需要引入额外依赖，示例logback-spring.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>
  
  	<!-- 文件输出格式 -->
  	<property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n" />
  	<!-- test文件路径 -->
  	<property name="TEST_FILE_PATH" value="/logs/test.log" />
  	<!-- pro文件路径 -->
  	<property name="PRO_FILE_PATH" value="/logs/prod.log" />
  
  	<!-- 开发环境 -->
  	<springProfile name="dev">
  		<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
  			<encoder>
  				<pattern>%date [%thread] %-5level %logger{80} || %msg%n</pattern>
  			</encoder>
  		</appender>
  		<logger name="cn.timebusker.util" level="debug" />
  		<root level="info">
  			<appender-ref ref="CONSOLE" />
  		</root>
  	</springProfile>
  
  	<!-- 测试环境 -->
  	<springProfile name="test">
  		<!-- 每天产生一个文件 -->
  		<appender name="TEST-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
  			<!-- 文件路径 -->
  			<file>${TEST_FILE_PATH}</file>
  			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
  				<!-- 文件名称 -->
  				<fileNamePattern>${TEST_FILE_PATH}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
  				<!-- 文件最大保存历史数量 -->
  				<MaxHistory>100</MaxHistory>
  			</rollingPolicy>
  			<layout class="ch.qos.logback.classic.PatternLayout">
  				<pattern>${PATTERN}</pattern>
  			</layout>
  		</appender>
  		<root level="info">
  			<appender-ref ref="TEST-FILE" />
  		</root>
  	</springProfile>
  
  	<!-- 生产环境 -->
  	<springProfile name="prod">
  		<appender name="PROD_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
  			<file>${PRO_FILE_PATH}</file>
  			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
  				<fileNamePattern>${PRO_FILE_PATH}/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
  				<MaxHistory>100</MaxHistory>
  			</rollingPolicy>
  			<layout class="ch.qos.logback.classic.PatternLayout">
  				<pattern>${PATTERN}</pattern>
  			</layout>
  		</appender>
  		<root level="warn">
  			<appender-ref ref="PROD_FILE" />
  		</root>
  	</springProfile>
  </configuration>
  
  ```

- Log4j：`log4j-spring.properties`, `log4j-spring.xml`, `log4j.properties`, `log4j.xml`

  ```xml
  <dependency>
  
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
      <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
  </exclusions>
  </dependency>
  <dependency>
  
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j</artifactId>
  </dependency>
  ```

- Log4j2：`log4j2-spring.xml`, `log4j2.xml`

  ```xml
  <dependency>
  
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
      <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
  </exclusions>
  </dependency>
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
  </dependency>
  ```

- JDK (Java Util Logging)：`logging.properties`

  ```xml
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
      <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
  </exclusions>
  </dependency>
  
  ```



### 异步处理

异步调用是相对于同步调用而言的，同步调用是指程序按预定顺序一步步执行，每一步必须等到上一步执行完后才能执行，异步调用则无需等待上一步程序执行完即可执行。

**如何实现异步调用？**

多线程，这是很多人第一眼想到的关键词，没错，多线程就是一种实现异步调用的方式。

在非spring目项目中我们要实现异步调用的就是使用多线程方式，可以自己实现Runable接口或者集成Thread类，或者使用jdk1.5以上提供了的Executors线程池。

可以参考Servlet 3.0 的异步请求处理

在 SpringBoot 中通过线程池来异步执行任务的两种方法：

1. 通过 Spring 自带的 `@EnableAsync` 和 `@Async` 两个注解实现异步执行任务功能
2. 通过自定义的方式

在通过 `@EnableAsync` 和 `@Async` 两个注解实现异步执行任务中会进一步分析 `@Async` 的局限性，自定义 `@Async` 注解的线程池，以及异常的处理。

**`@Async` 的局限性**

1. 只能作用于 *public* 方法上
2. 方法不能自己调自己，也就是说不能迭代调用

**示例**

1. 配置类：

   ```java
   /**
    * 利用@EnableAsync注解开启异步任务支持
    */
   @Configuration
   @EnableAsync
   public class TaskExecutorConfig implements AsyncConfigurer {
   
   	/**
   	 * Set the ThreadPoolExecutor's core pool size.
   	 */
   	private static final int CORE_POOL_SIZE = 5;
   
   	/**
   	 * Set the ThreadPoolExecutor's maximum pool size.
   	 */
   	private static final int MAX_POOL_SIZE = 20;
   
   	/**
   	 * Set the capacity for the ThreadPoolExecutor's BlockingQueue.
   	 */
   	private static final int QUEUE_CAPACITY = 10;
   
   	/**
   	 * 通过重写getAsyncExecutor方法，制定默认的任务执行由该方法产生
   	 * 
   	 * 配置类实现AsyncConfigurer接口并重写getAsyncExcutor方法，并返回一个ThreadPoolTaskExevutor
   	 * 这样我们就获得了一个基于线程池的TaskExecutor
   	 */
   	@Override
   	public Executor getAsyncExecutor() {
   		ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
   		taskExecutor.setCorePoolSize(CORE_POOL_SIZE);// 线程池维护线程的最少数量
   		taskExecutor.setMaxPoolSize(MAX_POOL_SIZE);// 线程池维护线程的最大数量
   		taskExecutor.setQueueCapacity(QUEUE_CAPACITY);// 线程池所使用的缓冲队列
   		taskExecutor.initialize();
   		return taskExecutor;
   	}
   
   	@Override
   	public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
   		return new CustomAsyncExceptionHandler();
   	}
   
   	/**
   	 * 对于没有返回值的 带有 `@Async` 注解的方法的异常处理
   	 */
   	private class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {
   
   		private Logger logger = LoggerFactory.getLogger(this.getClass());
   
   		@Override
   		public void handleUncaughtException (Throwable throwable, Method method, Object... obj) {
   			logger.info("Exception message - {}", throwable.getMessage());
   			logger.info("Method name - {}", method.getName());
   			for (Object param : obj) {
   				logger.info("Parameter value - {}", param);
   			}
   		}
   	}
   
   	/**
   	 * 自定义任务执行器：在定义了多个任务执行器的情况下，可以使用@Async("getMineAsync")来设定
   	 * 
   	 * @return
   	 */
       @Bean(name = "selfConfigThreadPool")
   	public Executor getMineAsync() {
   		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
   		executor.setCorePoolSize(CORE_POOL_SIZE - 4);
   		executor.setMaxPoolSize(MAX_POOL_SIZE - 10);
   		executor.setQueueCapacity(QUEUE_CAPACITY - 5);
   		executor.setThreadNamePrefix("mineAsync-");
   		// rejection-policy：当pool已经达到max size的时候，如何处理新任务
   		// CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
   		executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
   		executor.initialize();
   		return executor;
   	}
   
   }
   ```

2. Controller层业务代码

   ```java
   import com.hyp.learn.async.service.ArithmeticService;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.cglib.core.Constants;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RequestMethod;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.concurrent.ExecutionException;
   import java.util.concurrent.Future;
   
   @RestController
   @RequestMapping(value = "/api/async", produces = "application/json;charset=utf-8")
   public class AsyncController implements Constants {
   
       private final static Logger logger = LoggerFactory.getLogger(AsyncController.class);
   
       @Autowired
       private ArithmeticService arithmeticService;
   
       @RequestMapping(value = {"/asyncData"}, method = RequestMethod.GET)
       public void asyncData() {
           long start = System.currentTimeMillis();
           long sync = 0L;
           try {
               logger.info("--------------------------------------------\n");
               logger.info("每个任务执行的时间是：" + ArithmeticService.DoTime + "（毫秒）");
   
               arithmeticService.subByVoid();
   
               logger.info("--------------------------------------------\n");
           } catch (Exception e) {
               e.printStackTrace();
           }
           long end = System.currentTimeMillis();
           logger.info("\t........请求响应时间为：" + (end - start) + "（毫秒）");
   
       }
   
       @RequestMapping(value = {"/asyncGetData"}, method = RequestMethod.GET)
       public Long asyncBackData() throws ExecutionException, InterruptedException {
           long start = System.currentTimeMillis();
           Future<Long> task=null;
           try {
               logger.info("--------------------------------------------\n");
               logger.info("每个任务执行的时间是：" + ArithmeticService.DoTime + "（毫秒）");
   
               task = arithmeticService.subByAsync();
               
               logger.info("--------------------------------------------\n");
           } catch (Exception e) {
               e.printStackTrace();
           }
           long end = System.currentTimeMillis();
           logger.info("\t........请求响应时间为：" + (end - start) + "（毫秒）");
           return task.get();
       }
   
       @RequestMapping(value = {"/syncData"}, method = RequestMethod.GET)
       public void syncData() {
           long start = System.currentTimeMillis();
           long sync = 0L;
           try {
               logger.info("--------------------------------------------\n");
               logger.info("每个任务执行的时间是：" + ArithmeticService.DoTime + "（毫秒）");
               sync = arithmeticService.subBySync();
               logger.info("同步任务执行的时间是：" + sync + "（毫秒）");
   
               logger.info("--------------------------------------------\n");
           } catch (Exception e) {
               e.printStackTrace();
           }
           long end = System.currentTimeMillis();
           logger.info("\t........请求响应时间为：" + (end - start) + "（毫秒）");
       }
   
       /**
        * 自定义实现线程异步
        */
       @RequestMapping(value = {"/mine", "/m*"}, method = RequestMethod.GET)
       public void mineAsync() {
           for (int i = 0; i < 100; i++) {
               try {
                   arithmeticService.doMineAsync(i);
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

3. ThreadPoolTaskExecutor

   ```java
   public class ThreadPoolTaskExecutor extends ExecutorConfigurationSupport
   		implements AsyncListenableTaskExecutor, SchedulingTaskExecutor {
   
   	private final Object poolSizeMonitor = new Object();
   
   	private int corePoolSize = 1;
   
   	private int maxPoolSize = Integer.MAX_VALUE;
   
   	private int keepAliveSeconds = 60;
   
   	private int queueCapacity = Integer.MAX_VALUE;
   
   	private boolean allowCoreThreadTimeOut = false;
   
   	@Nullable
   	private TaskDecorator taskDecorator;
   
   	@Nullable
   	private ThreadPoolExecutor threadPoolExecutor;
   
   	// Runnable decorator to user-level FutureTask, if different
   	private final Map<Runnable, Object> decoratedTaskMap =
   			new ConcurrentReferenceHashMap<>(16, ConcurrentReferenceHashMap.ReferenceType.WEAK);
   
   
   	/**
   	 * Set the ThreadPoolExecutor's core pool size.
   	 * Default is 1.
   	 * <p><b>This setting can be modified at runtime, for example through JMX.</b>
   	 */
   	public void setCorePoolSize(int corePoolSize) {
   		synchronized (this.poolSizeMonitor) {
   			this.corePoolSize = corePoolSize;
   			if (this.threadPoolExecutor != null) {
   				this.threadPoolExecutor.setCorePoolSize(corePoolSize);
   			}
   		}
   	}
   
   	/**
   	 * Return the ThreadPoolExecutor's core pool size.
   	 */
   	public int getCorePoolSize() {
   		synchronized (this.poolSizeMonitor) {
   			return this.corePoolSize;
   		}
   	}
   
   	/**
   	 * Set the ThreadPoolExecutor's maximum pool size.
   	 * Default is {@code Integer.MAX_VALUE}.
   	 * <p><b>This setting can be modified at runtime, for example through JMX.</b>
   	 */
   	public void setMaxPoolSize(int maxPoolSize) {
   		synchronized (this.poolSizeMonitor) {
   			this.maxPoolSize = maxPoolSize;
   			if (this.threadPoolExecutor != null) {
   				this.threadPoolExecutor.setMaximumPoolSize(maxPoolSize);
   			}
   		}
   	}
   
   	/**
   	 * Return the ThreadPoolExecutor's maximum pool size.
   	 */
   	public int getMaxPoolSize() {
   		synchronized (this.poolSizeMonitor) {
   			return this.maxPoolSize;
   		}
   	}
   
   	/**
   	 * Set the ThreadPoolExecutor's keep-alive seconds.
   	 * Default is 60.
   	 * <p><b>This setting can be modified at runtime, for example through JMX.</b>
   	 */
   	public void setKeepAliveSeconds(int keepAliveSeconds) {
   		synchronized (this.poolSizeMonitor) {
   			this.keepAliveSeconds = keepAliveSeconds;
   			if (this.threadPoolExecutor != null) {
   				this.threadPoolExecutor.setKeepAliveTime(keepAliveSeconds, TimeUnit.SECONDS);
   			}
   		}
   	}
   
   	/**
   	 * Return the ThreadPoolExecutor's keep-alive seconds.
   	 */
   	public int getKeepAliveSeconds() {
   		synchronized (this.poolSizeMonitor) {
   			return this.keepAliveSeconds;
   		}
   	}
   
   	/**
   	 * Set the capacity for the ThreadPoolExecutor's BlockingQueue.
   	 * Default is {@code Integer.MAX_VALUE}.
   	 * <p>Any positive value will lead to a LinkedBlockingQueue instance;
   	 * any other value will lead to a SynchronousQueue instance.
   	 * @see java.util.concurrent.LinkedBlockingQueue
   	 * @see java.util.concurrent.SynchronousQueue
   	 */
   	public void setQueueCapacity(int queueCapacity) {
   		this.queueCapacity = queueCapacity;
   	}
   
   	/**
   	 * Specify whether to allow core threads to time out. This enables dynamic
   	 * growing and shrinking even in combination with a non-zero queue (since
   	 * the max pool size will only grow once the queue is full).
   	 * <p>Default is "false".
   	 * @see java.util.concurrent.ThreadPoolExecutor#allowCoreThreadTimeOut(boolean)
   	 */
   	public void setAllowCoreThreadTimeOut(boolean allowCoreThreadTimeOut) {
   		this.allowCoreThreadTimeOut = allowCoreThreadTimeOut;
   	}
   
   	/**
   	 * Specify a custom {@link TaskDecorator} to be applied to any {@link Runnable}
   	 * about to be executed.
   	 * <p>Note that such a decorator is not necessarily being applied to the
   	 * user-supplied {@code Runnable}/{@code Callable} but rather to the actual
   	 * execution callback (which may be a wrapper around the user-supplied task).
   	 * <p>The primary use case is to set some execution context around the task's
   	 * invocation, or to provide some monitoring/statistics for task execution.
   	 * @since 4.3
   	 */
   	public void setTaskDecorator(TaskDecorator taskDecorator) {
   		this.taskDecorator = taskDecorator;
   	}
   
   
   	/**
   	 * Note: This method exposes an {@link ExecutorService} to its base class
   	 * but stores the actual {@link ThreadPoolExecutor} handle internally.
   	 * Do not override this method for replacing the executor, rather just for
   	 * decorating its {@code ExecutorService} handle or storing custom state.
   	 */
   	@Override
   	protected ExecutorService initializeExecutor(
   			ThreadFactory threadFactory, RejectedExecutionHandler rejectedExecutionHandler) {
   
   		BlockingQueue<Runnable> queue = createQueue(this.queueCapacity);
   
   		ThreadPoolExecutor executor;
   		if (this.taskDecorator != null) {
   			executor = new ThreadPoolExecutor(
   					this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
   					queue, threadFactory, rejectedExecutionHandler) {
   				@Override
   				public void execute(Runnable command) {
   					Runnable decorated = taskDecorator.decorate(command);
   					if (decorated != command) {
   						decoratedTaskMap.put(decorated, command);
   					}
   					super.execute(decorated);
   				}
   			};
   		}
   		else {
   			executor = new ThreadPoolExecutor(
   					this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
   					queue, threadFactory, rejectedExecutionHandler);
   
   		}
   
   		if (this.allowCoreThreadTimeOut) {
   			executor.allowCoreThreadTimeOut(true);
   		}
   
   		this.threadPoolExecutor = executor;
   		return executor;
   	}
   
   	/**
   	 * Create the BlockingQueue to use for the ThreadPoolExecutor.
   	 * <p>A LinkedBlockingQueue instance will be created for a positive
   	 * capacity value; a SynchronousQueue else.
   	 * @param queueCapacity the specified queue capacity
   	 * @return the BlockingQueue instance
   	 * @see java.util.concurrent.LinkedBlockingQueue
   	 * @see java.util.concurrent.SynchronousQueue
   	 */
   	protected BlockingQueue<Runnable> createQueue(int queueCapacity) {
   		if (queueCapacity > 0) {
   			return new LinkedBlockingQueue<>(queueCapacity);
   		}
   		else {
   			return new SynchronousQueue<>();
   		}
   	}
   
   	/**
   	 * Return the underlying ThreadPoolExecutor for native access.
   	 * @return the underlying ThreadPoolExecutor (never {@code null})
   	 * @throws IllegalStateException if the ThreadPoolTaskExecutor hasn't been initialized yet
   	 */
   	public ThreadPoolExecutor getThreadPoolExecutor() throws IllegalStateException {
   		Assert.state(this.threadPoolExecutor != null, "ThreadPoolTaskExecutor not initialized");
   		return this.threadPoolExecutor;
   	}
   
   	/**
   	 * Return the current pool size.
   	 * @see java.util.concurrent.ThreadPoolExecutor#getPoolSize()
   	 */
   	public int getPoolSize() {
   		if (this.threadPoolExecutor == null) {
   			// Not initialized yet: assume core pool size.
   			return this.corePoolSize;
   		}
   		return this.threadPoolExecutor.getPoolSize();
   	}
   
   	/**
   	 * Return the number of currently active threads.
   	 * @see java.util.concurrent.ThreadPoolExecutor#getActiveCount()
   	 */
   	public int getActiveCount() {
   		if (this.threadPoolExecutor == null) {
   			// Not initialized yet: assume no active threads.
   			return 0;
   		}
   		return this.threadPoolExecutor.getActiveCount();
   	}
   
   
   	@Override
   	public void execute(Runnable task) {
   		Executor executor = getThreadPoolExecutor();
   		try {
   			executor.execute(task);
   		}
   		catch (RejectedExecutionException ex) {
   			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
   		}
   	}
   
   	@Override
   	public void execute(Runnable task, long startTimeout) {
   		execute(task);
   	}
   
   	@Override
   	public Future<?> submit(Runnable task) {
   		ExecutorService executor = getThreadPoolExecutor();
   		try {
   			return executor.submit(task);
   		}
   		catch (RejectedExecutionException ex) {
   			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
   		}
   	}
   
   	@Override
   	public <T> Future<T> submit(Callable<T> task) {
   		ExecutorService executor = getThreadPoolExecutor();
   		try {
   			return executor.submit(task);
   		}
   		catch (RejectedExecutionException ex) {
   			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
   		}
   	}
   
   	@Override
   	public ListenableFuture<?> submitListenable(Runnable task) {
   		ExecutorService executor = getThreadPoolExecutor();
   		try {
   			ListenableFutureTask<Object> future = new ListenableFutureTask<>(task, null);
   			executor.execute(future);
   			return future;
   		}
   		catch (RejectedExecutionException ex) {
   			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
   		}
   	}
   
   	@Override
   	public <T> ListenableFuture<T> submitListenable(Callable<T> task) {
   		ExecutorService executor = getThreadPoolExecutor();
   		try {
   			ListenableFutureTask<T> future = new ListenableFutureTask<>(task);
   			executor.execute(future);
   			return future;
   		}
   		catch (RejectedExecutionException ex) {
   			throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
   		}
   	}
   
   	@Override
   	protected void cancelRemainingTask(Runnable task) {
   		super.cancelRemainingTask(task);
   		// Cancel associated user-level Future handle as well
   		Object original = this.decoratedTaskMap.get(task);
   		if (original instanceof Future) {
   			((Future<?>) original).cancel(true);
   		}
   	}
   
   }
   
   ```
   
4. 异步任务类代码：

   ```java
   /**
    * 操作计算 service 类：简单实现有关异步和同步两种计算方式的性能比较
    */
   @Component
   public class ArithmeticService {
   
   	private final static Logger logger = LoggerFactory.getLogger(ArithmeticService.class);
   
   	public static final int DoTime = 5000;
   
   	/**
   	 * 异步任务 只需要在所需实现异步的方法上加上@Async注解， 并通过Future<T>来接受异步方法的处理结果
   	 * 通过@Async注解表明该方法是个异步方法，如果注解在类级别，则表明该类所有的方法都是异步方法
   	 * 
   	 * @return
   	 */
   	@Async
   	public Future<Long> subByAsync() throws Exception {
   		long start = System.currentTimeMillis();
   		long sum = 0;
   		Thread.sleep(DoTime);
   		long end = System.currentTimeMillis();
   		sum = end - start;
   		logger.info("\t 完成任务一");
   		return new AsyncResult<>(sum);
   	}
   
   	/**
   	 * 仅使用异步注解的方式实现异步方法
   	 * 
   	 * @return
   	 */
   	@Async
   	public void subByVoid() throws Exception {
   		long start = System.currentTimeMillis();
   		long sum = 0;
   		Thread.sleep(DoTime);
   		long end = System.currentTimeMillis();
   		sum = end - start;
   		logger.info("\t 完成任务二   ");
   		logger.info("注解任务执行的时间是： " + sum + "（毫秒）");
   	}
   
   	/**
   	 * 使用同步计算的方式--同步调用
   	 * 
   	 * @return
   	 */
   	public long subBySync() throws Exception {
   		long start = System.currentTimeMillis();
   		long sum = 0;
   		Thread.sleep(DoTime);
   		long end = System.currentTimeMillis();
   		sum = end - start;
   		logger.info("\t 完成任务三   ");
   		return sum;
   	}
       
   //在方法级别上自定义执行线程池
   	@Async("selfConfigThreadPool")
   	public void doMineAsync(int i) throws Exception {
   		System.out.println("------\t:" + i);
   		Thread.sleep(1000);
   	}
   }
   ```

**异步执行过程出现异常**

- 当带有 `@Async` 注解的方法返回类型是 *Future* 对象，异常的处理非常简单，调用 *Future.get()* 将会抛出异常，在外面进行 `try...catch` 即可。
- 如果带有 `@Async` 注解的方法返回类型是 *void*, 那么如何处理异常呢？
- 解决起来也很简单：在实现 AsyncConfigurer 接口时，同时重写 getAsyncExecutor 和 getAsyncUncaughtExceptionHandler 两个方法，比如上面的 CustomAsyncExceptionHandler 处理类

出现异常如下：

```java
   @Async
    public void asyncExection() {
        logger.info("Execute method asynchronously, thread name = {}", Thread.currentThread().getName());
        throw new RuntimeException("异常");
    }
```

异常输出

```
2019-12-21 23:09:51.958  INFO 30026 --- [lTaskExecutor-1] c.h.l.async.service.ArithmeticService    : Execute method asynchronously, thread name = ThreadPoolTaskExecutor-1
2019-12-21 23:09:51.960  INFO 30026 --- [lTaskExecutor-1] ecutorConfig$CustomAsyncExceptionHandler : Exception message - 异常
2019-12-21 23:09:51.961  INFO 30026 --- [lTaskExecutor-1] ecutorConfig$CustomAsyncExceptionHandler : Method name - asyncExection
```

#### 自定义

1. 新增配置类 ThreadConfig，通过 `@Bean` 来配置单个或者多个线程池。

   ThreadConfig 配置类，定义了两个线程池，一个用来发送邮件，一个用来处理心跳服务

   ```java
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.LinkedBlockingQueue;
   import java.util.concurrent.ThreadPoolExecutor;
   import java.util.concurrent.TimeUnit;
   
   @Configuration
   public class ThreadConfig {
   
       /**
        * 邮件服务
        * @return
        */
       @Bean("sendMailExecutorService")
       public ExecutorService sendMailExecutorService() {
           return new ThreadPoolExecutor(2, 2,
                   60L, TimeUnit.MILLISECONDS,
                   new LinkedBlockingQueue<>());
       }
   
       /**
        * 心跳服务
        * @return
        */
       @Bean("heartbeatExecutorService")
       public ExecutorService heartbeatExecutorService() {
           return new ThreadPoolExecutor(1, 1,
                   60L, TimeUnit.MILLISECONDS,
                   new LinkedBlockingQueue<>());
       }
   }
   ```

2. 通过 `@Qualifier("sendMailExecutorService")` 和 `@Autowired` 注入，ThreadService

   ```java
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.stereotype.Service;
   
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Future;
   import java.util.concurrent.TimeUnit;
   
   @Service
   public class ThreadService {
   
       private Logger logger = LoggerFactory.getLogger(this.getClass());
   
       @Qualifier("sendMailExecutorService")
       @Autowired
       private ExecutorService sendMailExecutorService;
   
       @Qualifier("heartbeatExecutorService")
       @Autowired
       private ExecutorService heartbeatExecutorService;
   
   
       public void heartbeatService() {
           heartbeatExecutorService.submit(() -> {
   
               // TODO 负责心跳有关的工作
               logger.info("Execute heartbeatService asynchronously, thread name = {}", Thread.currentThread().getName());
   
           });
       }
   
       public Future<Boolean> sendMailService() {
           return sendMailExecutorService.submit(() -> {
   
               logger.info("Execute sendMailService asynchronously, thread name = {}", Thread.currentThread().getName());
   
               // 休息1秒，模拟邮件发送过程
               TimeUnit.SECONDS.sleep(1);
               return true;
           });
       }
   }
   ```

3. 注意 ThreadConfig 中配置了多个线程池，由于 ExecutorService 类型相同，因此需要加上 Bean 的名称以及在注入的时候需要加上 `@Qualifier`

4. 通过 ThreadController 调用服务，具体如下。ThreadController 中 `/api/asyncThread/heartbeat` api 执行不需要返回，`/api/asyncThread/sendMail` api 需要返回结果

   ```java
   import com.ckjava.test.service.ThreadService;
   import com.ckjava.xutils.Constants;
   import com.ckjava.xutils.http.HttpResponse;
   import io.swagger.annotations.Api;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   @RequestMapping(value = "/api/asyncThread", produces = "application/json;charset=utf-8")
   public class ThreadController implements Constants {
   
       @Autowired
       private ThreadService threadService;
   
       /**
        * 请求，立即返回，但是不是具体的执行结果，具体的任务在线程池中慢慢的执行
        */
       @GetMapping("/heartbeat")
       public HttpResponse<String> asyncData() {
           threadService.heartbeatService();
           return HttpResponse.getReturn(null, HTTPCODE.code_200, STATUS.SUCCESS);
       }
   
       /**
        * 请求，执行完毕后再返回具体的结果，具体的任务在线程池中执行
        */
       @GetMapping("/sendMail")
       public HttpResponse<Boolean> asyncGetData() throws Exception {
           return HttpResponse.getReturn(threadService.sendMailService().get(), HTTPCODE.code_200, STATUS.SUCCESS);
       }
   
   }
   ```

**结论**

- 通过上面两种方式比较，可以发现 Spring 自带的 `@EnableAsync` 和 `@Async` 两个注解本质上也是基于 Java 自身的 Executor 线程池的，这种方式比较简单
- 自定义的方式可以带来更大的灵活性，以及可控性

#### 原理分析

Spring默认给我们创建了applicationTaskExecutor的ExecutorService的线程池。通过源码分析，Spring boot的starter已经给我们设置了默认的执行器

1. org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration

   ```java
   //实际使用的org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor，
   //而不是juc的线程池
   @ConditionalOnClass(ThreadPoolTaskExecutor.class)
   @Configuration(proxyBeanMethods = false)
   //配置类所在位置
   @EnableConfigurationProperties(TaskExecutionProperties.class)
   public class TaskExecutionAutoConfiguration {
   
   	/**
   	 * Bean name of the application {@link TaskExecutor}.
   	 */
   	public static final String APPLICATION_TASK_EXECUTOR_BEAN_NAME = "applicationTaskExecutor";
   
   	@Bean
   	@ConditionalOnMissingBean
   	public TaskExecutorBuilder taskExecutorBuilder(TaskExecutionProperties properties,
   			ObjectProvider<TaskExecutorCustomizer> taskExecutorCustomizers,
   			ObjectProvider<TaskDecorator> taskDecorator) {
   		TaskExecutionProperties.Pool pool = properties.getPool();
   		TaskExecutorBuilder builder = new TaskExecutorBuilder();
   		builder = builder.queueCapacity(pool.getQueueCapacity());
   		builder = builder.corePoolSize(pool.getCoreSize());
   		builder = builder.maxPoolSize(pool.getMaxSize());
   		builder = builder.allowCoreThreadTimeOut(pool.isAllowCoreThreadTimeout());
   		builder = builder.keepAlive(pool.getKeepAlive());
   		Shutdown shutdown = properties.getShutdown();
   		builder = builder.awaitTermination(shutdown.isAwaitTermination());
   		builder = builder.awaitTerminationPeriod(shutdown.getAwaitTerminationPeriod());
   		builder = builder.threadNamePrefix(properties.getThreadNamePrefix());
   		builder = builder.customizers(taskExecutorCustomizers.orderedStream()::iterator);
   		builder = builder.taskDecorator(taskDecorator.getIfUnique());
   		return builder;
   	}
   
   	@Lazy
   	@Bean(name = { APPLICATION_TASK_EXECUTOR_BEAN_NAME,
   			AsyncAnnotationBeanPostProcessor.DEFAULT_TASK_EXECUTOR_BEAN_NAME })
   	@ConditionalOnMissingBean(Executor.class)
   	public ThreadPoolTaskExecutor applicationTaskExecutor(TaskExecutorBuilder builder) {
   		return builder.build();
   	}
   
   }
   ```

2. TaskExecutionProperties：根据Spring boot的配置定律，我们可以通过配置来定义异步任务的参数

   ```java
   @ConfigurationProperties("spring.task.execution")
   public class TaskExecutionProperties {
   
   	private final Pool pool = new Pool();
   
   	private final Shutdown shutdown = new Shutdown();
   
   	/**
   	 * Prefix to use for the names of newly created threads.
   	 */
   	private String threadNamePrefix = "task-";
   
   //...
   
   	public static class Pool {
   
   		/**
   		 * Queue capacity. An unbounded capacity does not increase the pool and therefore
   		 * ignores the "max-size" property.
   		 */
   		private int queueCapacity = Integer.MAX_VALUE;
   
   		/**
   		 * Core number of threads.
   		 */
   		private int coreSize = 8;
   
   		/**
   		 * Maximum allowed number of threads. If tasks are filling up the queue, the pool
   		 * can expand up to that size to accommodate the load. Ignored if the queue is
   		 * unbounded.
   		 */
   		private int maxSize = Integer.MAX_VALUE;
   
   		/**
   		 * Whether core threads are allowed to time out. This enables dynamic growing and
   		 * shrinking of the pool.
   		 */
   		private boolean allowCoreThreadTimeout = true;
   
   		/**
   		 * Time limit for which threads may remain idle before being terminated.
   		 */
   		private Duration keepAlive = Duration.ofSeconds(60);
   //...
   
   	}
   
   	public static class Shutdown {
   
   		/**
   		 * Whether the executor should wait for scheduled tasks to complete on shutdown.
   		 */
   		private boolean awaitTermination;
   
   		/**
   		 * Maximum time the executor should wait for remaining tasks to complete.
   		 */
   		private Duration awaitTerminationPeriod;
   	//...
   
   	}
   
   }
   ```

   省略get set方法，spring boot的配置以spring.task.execution开头，参数的设置参考如上源码的属性设置。各位可以自行尝试，当然因为Spring bean的定义方式，我们可以复写bean来达到自定义的目的

   比如：

   ```java
   @Configuration
   @EnableAsync
   public class TaskAsyncConfig {
    
       @Bean
       public Executor initExecutor(){
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           //定制线程名称，还可以定制线程group
           executor.setThreadFactory(new ThreadFactory() {
               private final AtomicInteger threadNumber = new AtomicInteger(1);
    
               @Override
               public Thread newThread(Runnable r) {
                   Thread t = new Thread(Thread.currentThread().getThreadGroup(), r,
                           "async-task-" + threadNumber.getAndIncrement(),
                           0);
                   return t;
               }
           });
           executor.setCorePoolSize(10);
           executor.setMaxPoolSize(20);
           executor.setKeepAliveSeconds(5);
           executor.setQueueCapacity(100);
   //        executor.setRejectedExecutionHandler(null);
           return executor;
       }
   }
   ```

##### Spring 异步任务执行过程分析

执行异步任务使用Spring CGLib动态代理AOP实现，动态代理后使用AsyncExecutionInterceptor来处理异步逻辑，执行submit方法，默认的taskExecutor使用BeanFactory中获取。 

默认使用SimpleAsyncUncaughtExceptionHandler处理异步异常。下面我们来试试

```java
@EnableAsync
@Service
public class TaskService {
 
    @Async
    public String doTask(){
        System.out.println(Thread.currentThread().getThreadGroup() + "-------" + Thread.currentThread().getName());
        throw new RuntimeException(" I`m a demo test exception-----------------");
    }
}
```

默认会打印logger.error("Unexpected exception occurred invoking async method: " + method, ex);日志 

```java
public class SimpleAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
 
	private static final Log logger = LogFactory.getLog(SimpleAsyncUncaughtExceptionHandler.class);
 
 
	@Override
	public void handleUncaughtException(Throwable ex, Method method, Object... params) {
		if (logger.isErrorEnabled()) {
			logger.error("Unexpected exception occurred invoking async method: " + method, ex);
		}
	}
 
}
```

### 定时任务

在我们开发项目过程中，经常需要定时任务来帮助我们来做一些内容， Spring Boot 默认已经帮我们实行了，只需要添加相应的注解就可以实现

定时任务实现的几种方式：

1. Timer：这是java自带的java.util.Timer类，这个类允许你调度一个java.util.TimerTask任务。使用这种方式可以让你的程序按照某一个频度执行，但不能在指定时间运行。一般用的较少且不推荐。

   ```java
   public class TestTimer {
       public static void main(String[] args) {
           TimerTask timerTask = new TimerTask() {
               @Override
               public void run() {
                   System.out.println("task  run:"+ new Date());
               }
           };
           Timer timer = new Timer();
           //安排指定的任务在指定的时间开始进行重复的固定延迟执行。这里是每3秒执行一次
           timer.schedule(timerTask,10,3000);
       }
   }
   ```

2. ScheduledExecutorService：也jdk自带的一个类；是基于线程池设计的定时任务类,每个调度任务都会分配到线程池中的一个线程去执行,也就是说,任务是并发执行,互不影响。

   ```java
   public class TestScheduledExecutorService {
       public static void main(String[] args) {
           ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
           // 参数：1、任务体 2、首次执行的延时时间
           //      3、任务执行间隔 4、间隔时间单位
           service.scheduleAtFixedRate(()->System.out.println("task ScheduledExecutorService "+new Date()), 0, 3, TimeUnit.SECONDS);
       }
   }
   ```

3. Spring Task：Spring3.0以后自带的task，可以将它看成一个轻量级的Quartz，而且使用起来比Quartz简许多。

   示例参考下面

4. Quartz：这是一个功能比较强大的的调度器，可以让你的程序在指定时间执行，也可以按照某一个频度执行，配置起来稍显复杂。

#### 示例

1. pom文件

   pom 包里面只需要引入 Spring Boot Starter 包即可

   ```xml
   <dependencies>
   	<dependency>
   		<groupId>org.springframework.boot</groupId>
   		<artifactId>spring-boot-starter</artifactId>
   	</dependency>
   	<dependency>
   		<groupId>org.springframework.boot</groupId>
   		<artifactId>spring-boot-starter-test</artifactId>
   		<scope>test</scope>
   	</dependency>
   </dependencies>
   ```

2. 启动类启用定时

   在启动类上面加上`@EnableScheduling`即可开启定时

   ```java
   @SpringBootApplication
   @EnableScheduling
   public class Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Application.class, args);
   	}
   }
   ```

3. 多线程执行

   在传统的Spring项目中，我们可以在xml配置文件添加task的配置，而在SpringBoot项目中一般使用config配置类的方式添加配置，所以新建一个AsyncConfig类

   ```java
   @Configuration
   @EnableAsync
   public class AsyncConfig {
        /*
       此处成员变量应该使用@Value从配置中读取
        */
       private int corePoolSize = 10;
       private int maxPoolSize = 200;
       private int queueCapacity = 10;
       @Bean
       public Executor taskExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           executor.setCorePoolSize(corePoolSize);
           executor.setMaxPoolSize(maxPoolSize);
           executor.setQueueCapacity(queueCapacity);
           executor.initialize();
           return executor;
       }
   }
   ```

4. 创建定时任务实现类

   ```java
   
   @Component
   @EnableScheduling
   //@Async
   public class AlarmTask {
   
       /**默认是fixedDelay 上一次执行完毕时间后执行下一轮*/
       @Scheduled(cron = "0/5 * * * * *")
       public void run() throws InterruptedException {
           Thread.sleep(6000);
           System.out.println(Thread.currentThread().getName()+"=====>>>>>使用cron  {}"+(System.currentTimeMillis()/1000));
       }
   
       /**fixedRate:上一次开始执行时间点之后5秒再执行*/
       @Scheduled(fixedRate = 5000)
       public void run1() throws InterruptedException {
           Thread.sleep(6000);
           System.out.println(Thread.currentThread().getName()+"=====>>>>>使用fixedRate  {}"+(System.currentTimeMillis()/1000));
       }
   
       /**fixedDelay:上一次执行完毕时间点之后5秒再执行*/
       @Scheduled(fixedDelay = 5000)
       public void run2() throws InterruptedException {
           Thread.sleep(7000);
           System.out.println(Thread.currentThread().getName()+"=====>>>>>使用fixedDelay  {}"+(System.currentTimeMillis()/1000));
       }
   
       /**第一次延迟2秒后执行，之后按fixedDelay的规则每5秒执行一次*/
       @Scheduled(initialDelay = 2000, fixedDelay = 5000)
       public void run3(){
           System.out.println(Thread.currentThread().getName()+"=====>>>>>使用initialDelay  {}"+(System.currentTimeMillis()/1000));
       }
   
   ```

   结果如下：

   ```
   taskExecutor-3=====>>>>>使用initialDelay  {}1579772434
   taskExecutor-1=====>>>>>使用fixedRate  {}1579772438
   taskExecutor-7=====>>>>>使用initialDelay  {}1579772439
   taskExecutor-2=====>>>>>使用fixedDelay  {}1579772439
   taskExecutor-4=====>>>>>使用cron  {}1579772441
   taskExecutor-5=====>>>>>使用fixedRate  {}1579772443
   taskExecutor-3=====>>>>>使用initialDelay  {}1579772444
   taskExecutor-6=====>>>>>使用fixedDelay  {}1579772444
   taskExecutor-8=====>>>>>使用cron  {}1579772446
   ```

#### 参数说明

`@Scheduled` 参数可以接受两种定时的设置，一种是我们常用的`cron="*/6 * * * * ?"`,一种是 `fixedRate = 6000`，两种都表示每隔六秒打印一下内容。

**fixedRate 说明**

- `@Scheduled(fixedRate = 6000)` ：上一次开始执行时间点之后6秒再执行
- `@Scheduled(fixedDelay = 6000)` ：上一次执行完毕时间点之后6秒再执行
- `@Scheduled(initialDelay=1000, fixedRate=6000)` ：第一次延迟1秒后执行，之后按 fixedRate 的规则每6秒执行一次

#### 源码分析

- 扫描Task

  熟悉Spring的人都知道BeanPostProcessor这个回调接口，Spring框架扫描所有被@Scheduled注解的方法就是通过实现这个回调接口来实现的。

  具体实现类为ScheduledAnnotationBeanPostProcessor，在ScheduledAnnotationBeanPostProcessor的postProcessAfterInitialization会从bean中解析出有@Scheduled注解的方法，然后调用processScheduled处理这些方法。

  ```java
  public Object postProcessAfterInitialization(final Object bean, String beanName) {
          Class<?> targetClass = AopUtils.getTargetClass(bean);
          if (!this.nonAnnotatedClasses.contains(targetClass)) {
                          //扫描bean内带有Scheduled注解的方法
              Map<Method, Set<Scheduled>> annotatedMethods = MethodIntrospector.selectMethods(targetClass,
                      new MethodIntrospector.MetadataLookup<Set<Scheduled>>() {
                          @Override
                          public Set<Scheduled> inspect(Method method) {
                              Set<Scheduled> scheduledMethods =
                                      AnnotatedElementUtils.getMergedRepeatableAnnotations(method, Scheduled.class, Schedules.class);
                              return (!scheduledMethods.isEmpty() ? scheduledMethods : null);
                          }
                      });
              if (annotatedMethods.isEmpty()) {
                                  //如果这个class没有注解的方法，缓存下来，因为一个class可能有多个bean
                  this.nonAnnotatedClasses.add(targetClass);
                  if (logger.isTraceEnabled()) {
                      logger.trace("No @Scheduled annotations found on bean class: " + bean.getClass());
                  }
              }
              else {
                  // Non-empty set of methods
                  for (Map.Entry<Method, Set<Scheduled>> entry : annotatedMethods.entrySet()) {
                      Method method = entry.getKey();
                      for (Scheduled scheduled : entry.getValue()) {
                                                  //处理这些有Scheduled的方法
                          processScheduled(scheduled, method, bean);
                      }
                  }
                  if (logger.isDebugEnabled()) {
                      logger.debug(annotatedMethods.size() + " @Scheduled methods processed on bean '" + beanName +
                              "': " + annotatedMethods);
                  }
              }
          }
          return bean;
      }
  
  ```

- **注册Task**

  processScheduled方法会处理这个方法以及它对应的注解生成Task，然后把这些Task注册到`ScheduledTaskRegistrar`，`ScheduledTaskRegistrar`负责Task的生命周期。

  ```java
  protected void processScheduled(Scheduled scheduled, Method method, Object bean) {
          try {
              Assert.isTrue(method.getParameterTypes().length == 0,
                      "Only no-arg methods may be annotated with @Scheduled");
  
              Method invocableMethod = AopUtils.selectInvocableMethod(method, bean.getClass());
              Runnable runnable = new ScheduledMethodRunnable(bean, invocableMethod);
              boolean processedSchedule = false;
              String errorMessage =
                      "Exactly one of the 'cron', 'fixedDelay(String)', or 'fixedRate(String)' attributes is required";
  
              Set<ScheduledTask> tasks = new LinkedHashSet<ScheduledTask>(4);
  
              // Determine initial delay
              long initialDelay = scheduled.initialDelay();
              String initialDelayString = scheduled.initialDelayString();
              if (StringUtils.hasText(initialDelayString)) {
                  Assert.isTrue(initialDelay < 0, "Specify 'initialDelay' or 'initialDelayString', not both");
                  if (this.embeddedValueResolver != null) {
                      initialDelayString = this.embeddedValueResolver.resolveStringValue(initialDelayString);
                  }
                  try {
                      initialDelay = Long.parseLong(initialDelayString);
                  }
                  catch (NumberFormatException ex) {
                      throw new IllegalArgumentException(
                              "Invalid initialDelayString value \"" + initialDelayString + "\" - cannot parse into integer");
                  }
              }
  
              // Check cron expression
              String cron = scheduled.cron();
              if (StringUtils.hasText(cron)) {
                  Assert.isTrue(initialDelay == -1, "'initialDelay' not supported for cron triggers");
                  processedSchedule = true;
                  String zone = scheduled.zone();
                  if (this.embeddedValueResolver != null) {
                      cron = this.embeddedValueResolver.resolveStringValue(cron);
                      zone = this.embeddedValueResolver.resolveStringValue(zone);
                  }
                  TimeZone timeZone;
                  if (StringUtils.hasText(zone)) {
                      timeZone = StringUtils.parseTimeZoneString(zone);
                  }
                  else {
                      timeZone = TimeZone.getDefault();
                  }
                  tasks.add(this.registrar.scheduleCronTask(new CronTask(runnable, new CronTrigger(cron, timeZone))));
              }
  
              // At this point we don't need to differentiate between initial delay set or not anymore
              if (initialDelay < 0) {
                  initialDelay = 0;
              }
  
              // Check fixed delay
              long fixedDelay = scheduled.fixedDelay();
              if (fixedDelay >= 0) {
                  Assert.isTrue(!processedSchedule, errorMessage);
                  processedSchedule = true;
                  tasks.add(this.registrar.scheduleFixedDelayTask(new IntervalTask(runnable, fixedDelay, initialDelay)));
              }
              String fixedDelayString = scheduled.fixedDelayString();
              if (StringUtils.hasText(fixedDelayString)) {
                  Assert.isTrue(!processedSchedule, errorMessage);
                  processedSchedule = true;
                  if (this.embeddedValueResolver != null) {
                      fixedDelayString = this.embeddedValueResolver.resolveStringValue(fixedDelayString);
                  }
                  try {
                      fixedDelay = Long.parseLong(fixedDelayString);
                  }
                  catch (NumberFormatException ex) {
                      throw new IllegalArgumentException(
                              "Invalid fixedDelayString value \"" + fixedDelayString + "\" - cannot parse into integer");
                  }
                  tasks.add(this.registrar.scheduleFixedDelayTask(new IntervalTask(runnable, fixedDelay, initialDelay)));
              }
  
              // Check fixed rate
              long fixedRate = scheduled.fixedRate();
              if (fixedRate >= 0) {
                  Assert.isTrue(!processedSchedule, errorMessage);
                  processedSchedule = true;
                  tasks.add(this.registrar.scheduleFixedRateTask(new IntervalTask(runnable, fixedRate, initialDelay)));
              }
              String fixedRateString = scheduled.fixedRateString();
              if (StringUtils.hasText(fixedRateString)) {
                  Assert.isTrue(!processedSchedule, errorMessage);
                  processedSchedule = true;
                  if (this.embeddedValueResolver != null) {
                      fixedRateString = this.embeddedValueResolver.resolveStringValue(fixedRateString);
                  }
                  try {
                      fixedRate = Long.parseLong(fixedRateString);
                  }
                  catch (NumberFormatException ex) {
                      throw new IllegalArgumentException(
                              "Invalid fixedRateString value \"" + fixedRateString + "\" - cannot parse into integer");
                  }
                  tasks.add(this.registrar.scheduleFixedRateTask(new IntervalTask(runnable, fixedRate, initialDelay)));
              }
  
              // Check whether we had any attribute set
              Assert.isTrue(processedSchedule, errorMessage);
  
              // Finally register the scheduled tasks
              synchronized (this.scheduledTasks) {
                  Set<ScheduledTask> registeredTasks = this.scheduledTasks.get(bean);
                  if (registeredTasks == null) {
                      registeredTasks = new LinkedHashSet<ScheduledTask>(4);
                      this.scheduledTasks.put(bean, registeredTasks);
                  }
                  registeredTasks.addAll(tasks);
              }
          }
          catch (IllegalArgumentException ex) {
              throw new IllegalStateException(
                      "Encountered invalid @Scheduled method '" + method.getName() + "': " + ex.getMessage());
          }
      }
  
  ```

- 运行Task

  任务注册到ScheduledTaskRegistrar之后，我们就要运行它们。触发操作会在下面2个回调方法内触发。
  afterSingletonsInstantiated和名字一样，会等所有Singleton类型的bean实例化后触发。
  onApplicationEvent会等ApplicationContext完成refresh后被触发。

  ```java
  @Override
      public void afterSingletonsInstantiated() {
          if (this.applicationContext == null) {
              // Not running in an ApplicationContext -> register tasks early...
              finishRegistration();
          }
      }
  
      @Override
      public void onApplicationEvent(ContextRefreshedEvent event) {
          if (event.getApplicationContext() == this.applicationContext) {
              // Running in an ApplicationContext -> register tasks this late...
              // giving other ContextRefreshedEvent listeners a chance to perform
              // their work at the same time (e.g. Spring Batch's job registration).
              finishRegistration();
          }
      }
  
  ```

  在finishRegistration中先会对ScheduledTaskRegistrar进行初始化，ScheduledTaskRegistrar设置我们容器中的线程池，如果没有配置这个默认线程池，ScheduledTaskRegistrar内部也会创建一个。初始化完成之后，调用ScheduledTaskRegistrar的afterPropertiesSet方法运行注册的任务。

  ```java
  private void finishRegistration() {
          if (this.scheduler != null) {
              this.registrar.setScheduler(this.scheduler);
          }
  
          if (this.beanFactory instanceof ListableBeanFactory) {
              Map<String, SchedulingConfigurer> configurers =
                      ((ListableBeanFactory) this.beanFactory).getBeansOfType(SchedulingConfigurer.class);
              for (SchedulingConfigurer configurer : configurers.values()) {
                  configurer.configureTasks(this.registrar);
              }
          }
  
          if (this.registrar.hasTasks() && this.registrar.getScheduler() == null) {
              Assert.state(this.beanFactory != null, "BeanFactory must be set to find scheduler by type");
              try {
                  // Search for TaskScheduler bean...
                  this.registrar.setTaskScheduler(this.beanFactory.getBean(TaskScheduler.class));
              }
              catch (NoUniqueBeanDefinitionException ex) {
                  try {
                      this.registrar.setTaskScheduler(
                              this.beanFactory.getBean(DEFAULT_TASK_SCHEDULER_BEAN_NAME, TaskScheduler.class));
                  }
                  catch (NoSuchBeanDefinitionException ex2) {
                      if (logger.isInfoEnabled()) {
                          logger.info("More than one TaskScheduler bean exists within the context, and " +
                                  "none is named 'taskScheduler'. Mark one of them as primary or name it 'taskScheduler' " +
                                  "(possibly as an alias); or implement the SchedulingConfigurer interface and call " +
                                  "ScheduledTaskRegistrar#setScheduler explicitly within the configureTasks() callback: " +
                                  ex.getBeanNamesFound());
                      }
                  }
              }
              catch (NoSuchBeanDefinitionException ex) {
                  logger.debug("Could not find default TaskScheduler bean", ex);
                  // Search for ScheduledExecutorService bean next...
                  try {
                      this.registrar.setScheduler(this.beanFactory.getBean(ScheduledExecutorService.class));
                  }
                  catch (NoUniqueBeanDefinitionException ex2) {
                      try {
                          this.registrar.setScheduler(
                                  this.beanFactory.getBean(DEFAULT_TASK_SCHEDULER_BEAN_NAME, ScheduledExecutorService.class));
                      }
                      catch (NoSuchBeanDefinitionException ex3) {
                          if (logger.isInfoEnabled()) {
                              logger.info("More than one ScheduledExecutorService bean exists within the context, and " +
                                      "none is named 'taskScheduler'. Mark one of them as primary or name it 'taskScheduler' " +
                                      "(possibly as an alias); or implement the SchedulingConfigurer interface and call " +
                                      "ScheduledTaskRegistrar#setScheduler explicitly within the configureTasks() callback: " +
                                      ex2.getBeanNamesFound());
                          }
                      }
                  }
                  catch (NoSuchBeanDefinitionException ex2) {
                      logger.debug("Could not find default ScheduledExecutorService bean", ex2);
                      // Giving up -> falling back to default scheduler within the registrar...
                      logger.info("No TaskScheduler/ScheduledExecutorService bean found for scheduled processing");
                  }
              }
          }
  
          this.registrar.afterPropertiesSet();
      }
  
  ```

- ScheduledTaskRegistrar
  `ScheduledTaskRegistrar`内部维护一个线程池以及我们添加的任务，负责任务的启动以及停止工作。

  ```java
  private TaskScheduler taskScheduler;
  
      private ScheduledExecutorService localExecutor;
  
      private List<TriggerTask> triggerTasks;
  
      private List<CronTask> cronTasks;
  
      private List<IntervalTask> fixedRateTasks;
  
      private List<IntervalTask> fixedDelayTasks;
  
      private final Map<Task, ScheduledTask> unresolvedTasks = new HashMap<Task, ScheduledTask>(16);
  
      private final Set<ScheduledTask> scheduledTasks = new LinkedHashSet<ScheduledTask>(16);
  
  ```

- **Task注册**

  任务注册通过对应的`add`方法，随便举个例子如下

  ```java
  public void addFixedRateTask(IntervalTask task) {
              if (this.fixedRateTasks == null) {
                  this.fixedRateTasks = new ArrayList<IntervalTask>();
              }
              this.fixedRateTasks.add(task);
          }
  
  ```

- **Task执行**

  在`ScheduledAnnotationBeanPostProcessor`中我们通过`ScheduledTaskRegistrar`的`afterPropertiesSet`启动任务的执行。

  ```java
  public void afterPropertiesSet() {
          scheduleTasks();
      }
  
  ```

  `afterPropertiesSet`内调用了`scheduleTasks`方法

  ```java
      protected void scheduleTasks() {
              if (this.taskScheduler == null) {
                  this.localExecutor = Executors.newSingleThreadScheduledExecutor();
                  this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
              }
              if (this.triggerTasks != null) {
                  for (TriggerTask task : this.triggerTasks) {
                      addScheduledTask(scheduleTriggerTask(task));
                  }
              }
              if (this.cronTasks != null) {
                  for (CronTask task : this.cronTasks) {
                      addScheduledTask(scheduleCronTask(task));
                  }
              }
              if (this.fixedRateTasks != null) {
                  for (IntervalTask task : this.fixedRateTasks) {
                      addScheduledTask(scheduleFixedRateTask(task));
                  }
              }
              if (this.fixedDelayTasks != null) {
                  for (IntervalTask task : this.fixedDelayTasks) {
                      addScheduledTask(scheduleFixedDelayTask(task));
                  }
              }
          }
  
  ```

  在scheduleTasks方法内，会通过每种任务的schedule方法执行任务，schedule方法会返回一个ScheduledTask对象，ScheduledTask主要用来保存任务的Future，这个Future主要是用来取消任务执行。然后addScheduledTask把ScheduledTask保存起来。

  随便举一个任务的schedule方法的例子

  ```java
  public ScheduledTask scheduleFixedDelayTask(IntervalTask task) {
              ScheduledTask scheduledTask = this.unresolvedTasks.remove(task);
              boolean newTask = false;
              if (scheduledTask == null) {
                  scheduledTask = new ScheduledTask();
                  newTask = true;
              }
              if (this.taskScheduler != null) {
                  if (task.getInitialDelay() > 0) {
                      Date startTime = new Date(System.currentTimeMillis() + task.getInitialDelay());
                      scheduledTask.future =
                              this.taskScheduler.scheduleWithFixedDelay(task.getRunnable(), startTime, task.getInterval());
                  }
                  else {
                      scheduledTask.future =
                              this.taskScheduler.scheduleWithFixedDelay(task.getRunnable(), task.getInterval());
                  }
              }
              else {
                  addFixedDelayTask(task);
                  this.unresolvedTasks.put(task, scheduledTask);
              }
              return (newTask ? scheduledTask : null);
          }
  
  ```

- **Task销毁**

  `ScheduledTaskRegistrar`实现了`DisposableBean`接口，在这个`bean`被销毁的时候，会触发取消任务执行。

  ```java
  public void destroy() {
              for (ScheduledTask task : this.scheduledTasks) {
                  task.cancel();
              }
              if (this.localExecutor != null) {
                  this.localExecutor.shutdownNow();
              }
          }
  
  ```

  上面调用了`ScheduledTask`的`cancel`方法取消任务，我们来看下它的实现。

  ```java
  public final class ScheduledTask {
      
          volatile ScheduledFuture<?> future;
      
      
          ScheduledTask() {
          }
      
      
          /**
           * Trigger cancellation of this scheduled task.
           */
          public void cancel() {
              ScheduledFuture<?> future = this.future;
              if (future != null) {
                  future.cancel(true);
              }
          }
      
      }
  
  ```

  通过保存任务的`future`来对任务进行取消

#### 整合Quartz

- 添加依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-quartz</artifactId>
  </dependency>
  ```

- 在resources目录下创建schema文件夹，并将mysql的脚本文件放到该目录下，如果使用的不是mysql数据库，可以直接在Quartz官网下载(下载文件docs/dbTables文件内)，下载地址:http://d2zwv9pap9ylyd.cloudfront.net/quartz-2.2.3-distribution.tar.gz

  schema文件下下tables_mysql.sql内容如下：

  ```sql
  DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
  DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
  DROP TABLE IF EXISTS QRTZ_LOCKS;
  DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_TRIGGERS;
  DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;
  DROP TABLE IF EXISTS QRTZ_CALENDARS;
  
  
  CREATE TABLE QRTZ_JOB_DETAILS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      JOB_NAME  VARCHAR(200) NOT NULL,
      JOB_GROUP VARCHAR(200) NOT NULL,
      DESCRIPTION VARCHAR(250) NULL,
      JOB_CLASS_NAME   VARCHAR(250) NOT NULL,
      IS_DURABLE VARCHAR(1) NOT NULL,
      IS_NONCONCURRENT VARCHAR(1) NOT NULL,
      IS_UPDATE_DATA VARCHAR(1) NOT NULL,
      REQUESTS_RECOVERY VARCHAR(1) NOT NULL,
      JOB_DATA BLOB NULL,
      PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
  );
  
  CREATE TABLE QRTZ_TRIGGERS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      JOB_NAME  VARCHAR(200) NOT NULL,
      JOB_GROUP VARCHAR(200) NOT NULL,
      DESCRIPTION VARCHAR(250) NULL,
      NEXT_FIRE_TIME BIGINT(13) NULL,
      PREV_FIRE_TIME BIGINT(13) NULL,
      PRIORITY INTEGER NULL,
      TRIGGER_STATE VARCHAR(16) NOT NULL,
      TRIGGER_TYPE VARCHAR(8) NOT NULL,
      START_TIME BIGINT(13) NOT NULL,
      END_TIME BIGINT(13) NULL,
      CALENDAR_NAME VARCHAR(200) NULL,
      MISFIRE_INSTR SMALLINT(2) NULL,
      JOB_DATA BLOB NULL,
      PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
      FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
          REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP)
  );
  
  CREATE TABLE QRTZ_SIMPLE_TRIGGERS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      REPEAT_COUNT BIGINT(7) NOT NULL,
      REPEAT_INTERVAL BIGINT(12) NOT NULL,
      TIMES_TRIGGERED BIGINT(10) NOT NULL,
      PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
      FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
          REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
  );
  
  CREATE TABLE QRTZ_CRON_TRIGGERS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      CRON_EXPRESSION VARCHAR(200) NOT NULL,
      TIME_ZONE_ID VARCHAR(80),
      PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
      FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
          REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
  );
  
  CREATE TABLE QRTZ_SIMPROP_TRIGGERS
    (          
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      STR_PROP_1 VARCHAR(512) NULL,
      STR_PROP_2 VARCHAR(512) NULL,
      STR_PROP_3 VARCHAR(512) NULL,
      INT_PROP_1 INT NULL,
      INT_PROP_2 INT NULL,
      LONG_PROP_1 BIGINT NULL,
      LONG_PROP_2 BIGINT NULL,
      DEC_PROP_1 NUMERIC(13,4) NULL,
      DEC_PROP_2 NUMERIC(13,4) NULL,
      BOOL_PROP_1 VARCHAR(1) NULL,
      BOOL_PROP_2 VARCHAR(1) NULL,
      PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
      FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP) 
      REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
  );
  
  CREATE TABLE QRTZ_BLOB_TRIGGERS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      BLOB_DATA BLOB NULL,
      PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
      FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
          REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
  );
  
  CREATE TABLE QRTZ_CALENDARS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      CALENDAR_NAME  VARCHAR(200) NOT NULL,
      CALENDAR BLOB NOT NULL,
      PRIMARY KEY (SCHED_NAME,CALENDAR_NAME)
  );
  
  CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      TRIGGER_GROUP  VARCHAR(200) NOT NULL, 
      PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP)
  );
  
  CREATE TABLE QRTZ_FIRED_TRIGGERS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      ENTRY_ID VARCHAR(95) NOT NULL,
      TRIGGER_NAME VARCHAR(200) NOT NULL,
      TRIGGER_GROUP VARCHAR(200) NOT NULL,
      INSTANCE_NAME VARCHAR(200) NOT NULL,
      FIRED_TIME BIGINT(13) NOT NULL,
      SCHED_TIME BIGINT(13) NOT NULL,
      PRIORITY INTEGER NOT NULL,
      STATE VARCHAR(16) NOT NULL,
      JOB_NAME VARCHAR(200) NULL,
      JOB_GROUP VARCHAR(200) NULL,
      IS_NONCONCURRENT VARCHAR(1) NULL,
      REQUESTS_RECOVERY VARCHAR(1) NULL,
      PRIMARY KEY (SCHED_NAME,ENTRY_ID)
  );
  
  CREATE TABLE QRTZ_SCHEDULER_STATE
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      INSTANCE_NAME VARCHAR(200) NOT NULL,
      LAST_CHECKIN_TIME BIGINT(13) NOT NULL,
      CHECKIN_INTERVAL BIGINT(13) NOT NULL,
      PRIMARY KEY (SCHED_NAME,INSTANCE_NAME)
  );
  
  CREATE TABLE QRTZ_LOCKS
    (
      SCHED_NAME VARCHAR(120) NOT NULL,
      LOCK_NAME  VARCHAR(40) NOT NULL, 
      PRIMARY KEY (SCHED_NAME,LOCK_NAME)
  );
  
  
  commit;
  ```

  

-  配置quartz:在`application.yml`中配置quartz。相关配置的作用已经写在注解上。

  ```yml
  # spring的datasource等配置未贴出
  spring:
    quartz:
        # 将任务等保存化到数据库
        job-store-type: jdbc
        # 程序结束时会等待quartz相关的内容结束
        wait-for-jobs-to-complete-on-shutdown: true
        # QuartzScheduler启动时更新己存在的Job,这样就不用每次修改targetObject后删除qrtz_job_details表对应记录
        overwrite-existing-jobs: true
        jdbc:
        # 每次启动重新初始化数据库中Quartz相关的表，如果不需要每次启动服务都重新创建表，下面两项可以不配置，事先在数据库中创建好Quartz相关的表
        	initialize-schema: always
        	schema: classpath:schema/tables_mysql.sql
        # 这里居然是个map，搞得智能提示都没有，佛了
        properties:
          org:
            quartz:
            	# scheduler相关
              scheduler:
                # scheduler的实例名
                instanceName: scheduler
                instanceId: AUTO
              # 持久化相关
              jobStore:
                class: org.quartz.impl.jdbcjobstore.JobStoreTX
                driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
                # 表示数据库中相关表是QRTZ_开头的
                tablePrefix: QRTZ_
                useProperties: false
              # 线程池相关
              threadPool:
                class: org.quartz.simpl.SimpleThreadPool
                # 线程数
                threadCount: 10
                # 线程优先级
                threadPriority: 5
                threadsInheritContextClassLoaderOfInitializingThread: true
  ```

- 创建任务类TestQuartz，该类主要是继承了QuartzJobBean

  ```java
  public class TestQuartz extends QuartzJobBean {
      /**
       * 执行定时任务
       * @param jobExecutionContext
       * @throws JobExecutionException
       */
      @Override
      protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
          // 任务的具体逻辑
          System.out.println("quartz task "+new Date());
      }
  }
  ```

- 创建配置类QuartzConfig

  ```java
  @Configuration
  public class QuartzConfig {
      @Bean
      public JobDetail teatQuartzDetail(){
          return JobBuilder.newJob(TestQuartz.class).withIdentity("testQuartz").storeDurably().build();
      }
      @Bean
      public Trigger testQuartzTrigger(){
          SimpleScheduleBuilder scheduleBuilder = SimpleScheduleBuilder.simpleSchedule()
                  .withIntervalInSeconds(10)  //设置时间周期单位秒
                  .repeatForever();
          return TriggerBuilder.newTrigger().forJob(teatQuartzDetail())
                  .withIdentity("testQuartz")
                  .withSchedule(scheduleBuilder)
                  .build();
      }
  }
  ```

更多示例参考:<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-6-quartz>

### 邮件服务

发送邮件应该是网站的必备功能之一，什么注册验证，忘记密码或者是给用户发送营销信息。最早期的时候我们会使用 JavaMail 相关 api 来写发送邮件的相关代码，后来 Spring 推出了 JavaMailSender 更加简化了邮件发送的过程，在之后 Spring Boot 对此进行了封装就有了现在的 `spring-boot-starter-mail` ,本章文章的介绍主要来自于此包。

1. pom 包配置

   pom 包里面添加 `spring-boot-starter-mail` 包引用

   ```xml
   <dependencies>
   	<dependency> 
   	    <groupId>org.springframework.boot</groupId>
   	    <artifactId>spring-boot-starter-mail</artifactId>
   	</dependency> 
   </dependencies>
   ```

2. 在 application.properties 中添加邮箱配置

   ```properties
   #网易邮箱服务器地址，qq为smtp.qq.com
   spring.mail.host=smtp.163.com 
   #用户名
   spring.mail.username=xxx@163.com 
   #密码，也可以是授权码，具体设置参考
   spring.mail.password=xxyyooo    
   spring.mail.default-encoding=UTF-8
   
   mail.fromMail.addr=xxx@oo.com  //以谁来发送邮件
   ```

   ![1ZNMLT.png](https://s2.ax1x.com/2020/01/24/1ZNMLT.png)

   ![1ZNYWR.png](https://s2.ax1x.com/2020/01/24/1ZNYWR.png)

3. 编写 mailService，这里只提出实现类。

   ```java
   @Component
   public class MailServiceImpl implements MailService{
   
       private final Logger logger = LoggerFactory.getLogger(this.getClass());
   
       @Autowired
       private JavaMailSender mailSender;
   
       @Value("${mail.fromMail.addr}")
       private String from;
   
       @Override
       public void sendSimpleMail(String to, String subject, String content) {
           SimpleMailMessage message = new SimpleMailMessage();
           message.setFrom(from);
           //抄送自己，否则报554
           message.setCc(from);
           message.setTo(to);
           message.setSubject(subject);
           message.setText(content);
   
           try {
               mailSender.send(message);
               logger.info("简单邮件已经发送。");
           } catch (Exception e) {
               logger.error("发送简单邮件时发生异常！", e);
           }
   
       }
   }
   ```

4. 编写 test 类进行测试

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class MailServiceTest {
   
       @Autowired
       private MailService MailService;
   
       @Test
       public void testSimpleMail() throws Exception {
           MailService.sendSimpleMail("ityouknow@126.com","test simple mail"," hello this is simple mail");
       }
   }
   ```

5. 发送 html 格式邮件

   ```java
   public void sendHtmlMail(String to, String subject, String content) {
       MimeMessage message = mailSender.createMimeMessage();
   
       try {
           //true表示需要创建一个multipart message
           MimeMessageHelper helper = new MimeMessageHelper(message, true);
           helper.setFrom(from);
           //添加抄送，否则报554
           helper.setCc(from);
           helper.setTo(to);
           helper.setSubject(subject);
           helper.setText(content, true);
   
           mailSender.send(message);
           logger.info("html邮件发送成功");
       } catch (MessagingException e) {
           logger.error("发送html邮件时发生异常！", e);
       }
   }
   ```

   在测试类中构建 html 内容，测试发送

   ```java
   @Test
   public void testHtmlMail() throws Exception {
       String content="<html>\n" +
               "<body>\n" +
               "    <h3>hello world ! 这是一封Html邮件!</h3>\n" +
               "</body>\n" +
               "</html>";
       MailService.sendHtmlMail("ityouknow@126.com","test simple mail",content);
   }
   ```

6. 发送带附件的邮件

   ```java
   //在 MailService 添加 sendAttachmentsMail 方法.
   public void sendAttachmentsMail(String to, String subject, String content, String filePath){
       MimeMessage message = mailSender.createMimeMessage();
   
       try {
           MimeMessageHelper helper = new MimeMessageHelper(message, true);
           helper.setFrom(from);
           helper.setTo(to);
           helper.setSubject(subject);
           helper.setText(content, true);
   
           FileSystemResource file = new FileSystemResource(new File(filePath));
           String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
           helper.addAttachment(fileName, file);
   
           mailSender.send(message);
           logger.info("带附件的邮件已经发送。");
       } catch (MessagingException e) {
           logger.error("发送带附件的邮件时发生异常！", e);
       }
   }
   ```

   > 添加多个附件可以使用多条 `helper.addAttachment(fileName, file)`

   在测试类中添加测试方法

   ```java
   @Test
   public void sendAttachmentsMail() {
       String filePath="e:\\tmp\\application.log";
       mailService.sendAttachmentsMail("ityouknow@126.com", "主题：带附件的邮件", "有附件，请查收！", filePath);
   }
   ```

7. 发送带静态资源的邮件

   邮件中的静态资源一般就是指图片，在 MailService 添加 sendAttachmentsMail 方法.

   ```java
   public void sendInlineResourceMail(String to, String subject, String content, String rscPath, String rscId){
       MimeMessage message = mailSender.createMimeMessage();
   
       try {
           MimeMessageHelper helper = new MimeMessageHelper(message, true);
           helper.setFrom(from);
           helper.setTo(to);
           helper.setSubject(subject);
           helper.setText(content, true);
   
           FileSystemResource res = new FileSystemResource(new File(rscPath));
           helper.addInline(rscId, res);
   
           mailSender.send(message);
           logger.info("嵌入静态资源的邮件已经发送。");
       } catch (MessagingException e) {
           logger.error("发送嵌入静态资源的邮件时发生异常！", e);
       }
   }
   ```

   在测试类中添加测试方法

   ```java
   @Test
   public void sendInlineResourceMail() {
       String rscId = "neo006";
       String content="<html><body>这是有图片的邮件：<img src=\'cid:" + rscId + "\' ></body></html>";
       String imgPath = "C:\\Users\\summer\\Pictures\\favicon.png";
   
       mailService.sendInlineResourceMail("ityouknow@126.com", "主题：这是有图片的邮件", content, imgPath, rscId);
   }
   ```

   到此所有的邮件发送服务已经完成了。

#### 自动配置

自动配置位于org.springframework.boot.autoconfigure.mail下

1. MailSenderAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ MimeMessage.class, MimeType.class, MailSender.class })
   //如果MailSender的bean则不生效
   @ConditionalOnMissingBean(MailSender.class)
   @Conditional(MailSenderCondition.class)
   //属性配置类
   @EnableConfigurationProperties(MailProperties.class)
   //导入配置Jndi和普通的MailSender配置
   @Import({ MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class })
   public class MailSenderAutoConfiguration {
   
   	/**
   	两种方式配置
   	 * Condition to trigger the creation of a {@link MailSender}. This kicks in if either
   	 * the host or jndi name property is set.
   	 */
   	static class MailSenderCondition extends AnyNestedCondition {
   
   		MailSenderCondition() {
   			super(ConfigurationPhase.PARSE_CONFIGURATION);
   		}
   
   		@ConditionalOnProperty(prefix = "spring.mail", name = "host")
   		static class HostProperty {
   
   		}
   
   		@ConditionalOnProperty(prefix = "spring.mail", name = "jndi-name")
   		static class JndiNameProperty {
   
   		}
   
   	}
   
   }
   ```

2. MailProperties邮件支持的配置属性

   ```java
   @ConfigurationProperties(prefix = "spring.mail")
   public class MailProperties {
   
   	private static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;
   
   	/**
   	 * SMTP server host. For instance, `smtp.example.com`.
   	 */
   	private String host;
   
   	/**
   	 * SMTP server port.
   	 */
   	private Integer port;
   
   	/**
   	 * Login user of the SMTP server.
   	 */
   	private String username;
   
   	/**
   	 * Login password of the SMTP server.
   	 */
   	private String password;
   
   	/**
   	 * Protocol used by the SMTP server.
   	 */
   	private String protocol = "smtp";
   
   	/**
   	 * Default MimeMessage encoding.
   	 */
   	private Charset defaultEncoding = DEFAULT_CHARSET;
   
   	/**
   	 * Additional JavaMail Session properties.
   	 */
   	private Map<String, String> properties = new HashMap<>();
   }
   ```

3. MailSenderPropertiesConfiguration，这里不对MailSenderJndiConfiguration作说明，其实际作用跟MailSenderPropertiesConfiguration相同

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnProperty(prefix = "spring.mail", name = "host")
   class MailSenderPropertiesConfiguration {
   //注入的bean
   	@Bean
   	@ConditionalOnMissingBean(JavaMailSender.class)
   	JavaMailSenderImpl mailSender(MailProperties properties) {
   		JavaMailSenderImpl sender = new JavaMailSenderImpl();
   		applyProperties(properties, sender);
   		return sender;
   	}
   //配置属性生效
   	private void applyProperties(MailProperties properties, JavaMailSenderImpl sender) {
   		sender.setHost(properties.getHost());
   		if (properties.getPort() != null) {
   			sender.setPort(properties.getPort());
   		}
   		sender.setUsername(properties.getUsername());
   		sender.setPassword(properties.getPassword());
   		sender.setProtocol(properties.getProtocol());
   		if (properties.getDefaultEncoding() != null) {
   			sender.setDefaultEncoding(properties.getDefaultEncoding().name());
   		}
   		if (!properties.getProperties().isEmpty()) {
   			sender.setJavaMailProperties(asProperties(properties.getProperties()));
   		}
   	}
   
   	private Properties asProperties(Map<String, String> source) {
   		Properties properties = new Properties();
   		properties.putAll(source);
   		return properties;
   	}
   
   }
   
   ```

4. MailSenderValidatorAutoConfiguration自动配置，用于测试邮件服务

   ```java
   @Configuration(proxyBeanMethods = false)
   @AutoConfigureAfter(MailSenderAutoConfiguration.class)
   //当配置此属性时生效
   @ConditionalOnProperty(prefix = "spring.mail", value = "test-connection")
   //当只有一个实例bean时
   @ConditionalOnSingleCandidate(JavaMailSenderImpl.class)
   public class MailSenderValidatorAutoConfiguration {
   
   	private final JavaMailSenderImpl mailSender;
   
   	public MailSenderValidatorAutoConfiguration(JavaMailSenderImpl mailSender) {
   		this.mailSender = mailSender;
   	}
   
   	@PostConstruct
   	public void validateConnection() {
   		try {
   			this.mailSender.testConnection();
   		}
   		catch (MessagingException ex) {
   			throw new IllegalStateException("Mail server is not available", ex);
   		}
   	}
   
   }
   ```

   

#### 邮件系统

上面发送邮件的基础服务就这些了，但是如果我们要做成一个邮件系统的话还需要考虑以下几个问题：

我们会经常收到这样的邮件：

```
尊敬的neo用户：
              
          恭喜您注册成为xxx网的用户,，同时感谢您对xxx的关注与支持并欢迎您使用xx的产品与服务。
          ...
```

其中只有 neo 这个用户名在变化，其它邮件内容均不变，如果每次发送邮件都需要手动拼接的话会不够优雅，并且每次模板的修改都需要改动代码的话也很不方便，因此对于这类邮件需求，都建议做成邮件模板来处理。模板的本质很简单，就是在模板中替换变化的参数，转换为 html 字符串即可，这里以`thymeleaf`为例来演示。

1. pom 中导入 thymeleaf 的包

   ```xml
   <dependency>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>
   ```

2. 在 resorces/templates 下创建 emailTemplate.html

   ```xml
   <!DOCTYPE html>
   <html lang="zh" xmlns:th="http://www.thymeleaf.org">
       <head>
           <meta charset="UTF-8"/>
           <title>Title</title>
       </head>
       <body>
           您好,这是验证邮件,请点击下面的链接完成验证,<br/>
           <a href="#" th:href="@{ http://www.ityouknow.com/neo/{id}(id=${id}) }">激活账号</a>
       </body>
   </html>
   ```

3. 解析模板并发送

   ```java
   @Test
   public void sendTemplateMail() {
       //创建邮件正文
       Context context = new Context();
       context.setVariable("id", "006");
       String emailContent = templateEngine.process("emailTemplate", context);
   
       mailService.sendHtmlMail("ityouknow@126.com","主题：这是模板邮件",emailContent);
   }
   ```

**发送失败**

因为各种原因，总会有邮件发送失败的情况，比如：邮件发送过于频繁、网络异常等。在出现这种情况的时候，我们一般会考虑重新重试发送邮件，会分为以下几个步骤来实现：

- 1、接收到发送邮件请求，首先记录请求并且入库。
- 2、调用邮件发送接口发送邮件，并且将发送结果记录入库。
- 3、启动定时系统扫描时间段内，未发送成功并且重试次数小于3次的邮件，进行再次发送

**异步发送**

很多时候邮件发送并不是我们主业务必须关注的结果，比如通知类、提醒类的业务可以允许延时或者失败。这个时候可以采用异步的方式来发送邮件，加快主交易执行速度，在实际项目中可以采用MQ发送邮件相关参数，监听到消息队列之后启动发送邮件。



### 参考

1. [SpringBoot | SpringBoot 是如何实现日志的？](https://www.jianshu.com/p/a27de9b7d297)
2. [springBoot异步执行方法@Async](https://www.jianshu.com/p/2d4b89c7a3f1)
3. [SpringBoot 中异步执行任务的 2 种方式](http://toulezu.com/2019/08/07/Spring-Boot-EnableAsync-Async-ExecutorService-usage-practice/)
4. [Spring boot异步任务原理分析](https://blog.csdn.net/fenglllle/article/details/91398384)
5. [深入理解spring boot异步调用方式@Async](https://www.jb51.net/article/118562.htm)