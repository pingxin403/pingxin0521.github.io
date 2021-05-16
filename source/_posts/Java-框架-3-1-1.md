---
title: 任务框架--Quartz (一)
date: 2019-05-21 12:18:59
tags:
 - Java
 - 框架
 - 任务调度
categories:
 - Java
 - 框架
---

Quartz是OpenSymphony开源组织在Job scheduling领域的开源项目,它可以与J2EE与J2SE应用程序相结合也可以单独使用。Quartz可以用来创建简单或为运行十个，百个，甚至是好几万个Jobs这样复杂的日程序表。Jobs可以做成标准的Java组件或 EJBs。

Quartz是一个任务日程管理系统，一个在预先确定（被纳入日程）的时间到达时，负责执行（或者通知）其他软件组件的系统。

[Quartz快速入门指南](https://www.w3cschool.cn/quartz_doc/quartz_doc-2put2clm.html)

<!--more-->

类似于java.util.Timer。但是相较于Timer， Quartz增加了很多功能：

- 持久性作业 - 就是保持调度定时的状态;
- 作业管理 - 对调度作业进行有效的管理;

Quartz用一个小Java库发布文件（.jar文件），这个库文件包含了所有Quartz核心功能。这些功能的主要接口(API)是Scheduler接口。它提供了简单的操作，例如：将任务纳入日程或者从日程中取消，开始/停止/暂停日程进度。

![3.png](https://i.loli.net/2019/05/31/5cf11bbc74b9699000.png)

特点：

- Quartz 是一个完全由 Java 编写的开源作业调度框架，为在 Java 应用程序中进行作业调度提供了简单却强大的机制。
- Quartz 可以与 J2EE 与 J2SE 应用程序相结合也可以单独使用。
- Quartz 允许程序开发人员根据时间的间隔来调度作业。
- Quartz 实现了作业和触发器的多对多的关系，还能把多个作业与不同的触发器关联。

框架图：

![1.png](https://i.loli.net/2019/05/31/5cf11ccd7d95316520.png)

- Scheduler：quartz的运行容器，Trigger和JobDetail可以注册到scheduler中，两者在Scheduler中拥有各自的组及名称(组及名称是Scheduler查找定位容器中某一对象的依据)，Trigger和JobDetail的组合名称都必须唯一，但是两者的组和名称可以相同，因为它们是不同类，一个job可以对应多个trigger，但是一个trigger只能对应一个job
- Job：就是要执行的任务，该接口只有一个方法execute（JobExecutionContext jobExecutionContext)方法，Job运行时的信息保存在JobDataMap中
- JobDetail ：Job的描述类，job执行时的依据此对象的信息反射实例化出Job的具体执行对象。
- Trigger：触发器，存放Job执行的时间策略。用于定义任务调度时间规则。主要包括四类：SimpleTriger、CronTrigger、DateIntervalTrigger和NthIncludeDayTrigger，目前常用的是前两种
- JobStore： 存储作业和调度期间的状态
- JobBuilder：定义和创建JobDetail实例的接口
- TriggerBuilder： 定义和创建Trigger实例的接口
- Calendar：指定排除的时间点（如排除法定节假日）

Scheduler的生命期，从SchedulerFactory创建它时开始，到Scheduler调用shutdown()方法时结束；Scheduler被创建后，可以增加、删除和列举Job和Trigger，以及执行其它与调度相关的操作（如暂停Trigger）。但是，Scheduler只有在调用start()方法后，才会真正地触发trigger（即执行job）

这些接口的关系图如下：

![UTOOLS1576331550118.png](https://i.loli.net/2019/12/14/zeWo2ctRbdmV4Tp.png)

Maven依赖：

```xml
 	<dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
      </dependency>
            <!--日志框架-->
      <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.26</version>
        
      </dependency>
      <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-jdk14</artifactId>
        <version>1.7.26</version>
      </dependency>

      <!--quartz任务框架-->
      <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz-jobs</artifactId>
        <version>2.3.1</version>
      </dependency>
      <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz</artifactId>
        <version>2.3.1</version>
        <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
      </dependency>


```

**使用示例**

HelloJob类

```java
//新建一个能够打印任意内容的Job
public class HelloJob implements Job {
    private static Logger log = LoggerFactory.getLogger(HelloJob.class);  
    public HelloJob() { }  
    public void execute(JobExecutionContext context) throws JobExecutionException {
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        log.error("Hello Job执行时间: " + printTime);
    }
}  
```

测试

```java
//创建Schedule，执行任务
public class MyScheduler {
    private static Logger _log = LoggerFactory.getLogger(HelloJob.class);

    public static void main(String[] args) throws SchedulerException, InterruptedException {
        // 1. 创建 SchedulerFactory
        SchedulerFactory factory = new StdSchedulerFactory();
        // 2. 从工厂中获取调度器实例
        Scheduler scheduler = factory.getScheduler();

        // 3. 引进作业程序,创建JobDetail实例，并与HelloJob类绑定(Job执行内容)
        JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)
                .withDescription("this is a ram job") //job的描述
                .withIdentity("jobTest", "jobTestGrip") //job 的name和group
                .build();

        long time = System.currentTimeMillis() + 3 * 1000L; //3秒后启动任务
        Date statTime = new Date(time);

        // 4. 创建Trigger
        //使用SimpleScheduleBuilder或者CronScheduleBuilder
        Trigger trigger = TriggerBuilder.newTrigger()
                .withDescription("this is a cronTrigger")
                .withIdentity("jobTrigger", "jobTriggerGroup")
                // .startNow()//立即生效
//                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
//                        .withIntervalInSeconds(5)//每隔5s执行一次
//                        .repeatForever())//一直执行
                .startAt(statTime)  //默认当前时间启动
                .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")) //5秒执行一次
                .build();

        // 5. 注册任务和定时器
        scheduler.scheduleJob(jobDetail, trigger);


        System.out.println("--------scheduler start ! ------------");

        // 6. 启动 调度器
        scheduler.start();
        _log.info("启动时间 ： " + new Date());

        //睡眠
        TimeUnit.MINUTES.sleep(1);
        scheduler.shutdown();
        System.out.println("--------scheduler shutdown ! ------------");
    }
}

```

当Job的一个trigger被触发时，execute（）方法由调度程序的一个工作线程调用。传递给execute()方法的JobExecutionContext对象向作业实例提供有关其“运行时”job的一个trigger被触发后（稍后会讲到），execute()方法会被scheduler的一个工作线程调用；传递给execute()方法的JobExecutionContext对象中保存着该job运行时的一些信息 ，执行job的scheduler的引用，触发job的trigger的引用，JobDetail对象引用，以及一些其它信息。

JobDetail对象是在将job加入scheduler时，由客户端程序（你的程序）创建的。它包含job的各种属性设置，以及用于存储job实例状态信息的JobDataMap。

Trigger用于触发Job的执行。当你准备调度一个job时，你创建一个Trigger的实例，然后设置调度相关的属性。Trigger也有一个相关联的JobDataMap，用于给Job传递一些触发相关的参数。Quartz自带了各种不同类型的Trigger，最常用的主要是SimpleTrigger和CronTrigger。

SimpleTrigger主要用于一次性执行的Job（只在某个特定的时间点执行一次），或者Job在特定的时间点执行，重复执行N次，每次执行间隔T个时间单位。CronTrigger在基于日历的调度上非常有用，如“每个星期五的正午”，或者“每月的第十天的上午10:15”等。

为什么既有Job，又有Trigger呢？很多任务调度器并不区分Job和Trigger。有些调度器只是简单地通过一个执行时间和一些job标识符来定义一个Job；其它的一些调度器将Quartz的Job和Trigger对象合二为一。在开发Quartz的时候，我们认为将调度和要调度的任务分离是合理的。在我们看来，这可以带来很多好处。

例如，Job被创建后，可以保存在Scheduler中，与Trigger是独立的，同一个Job可以有多个Trigger；这种松耦合的另一个好处是，当与Scheduler中的Job关联的trigger都过期时，可以配置Job稍后被重新调度，而不用重新定义Job；还有，可以修改或者替换Trigger，而不用重新定义与之关联的Job。

将Job和Trigger注册到Scheduler时，可以为它们设置key，配置其身份属性。Job和Trigger的key（JobKey和TriggerKey）可以用于将Job和Trigger放到不同的分组（group）里，然后基于分组进行操作。同一个分组下的Job或Trigger的名称必须唯一，即一个Job或Trigger的key由名称（name）和分组（group）组成。

### job

Job 是一个接口，只有一个方法  `void execute(JobExecutionContext context)`，开发者实现接口来定义任务。`JobExecutionContext` 类提供了调度上下文的各种信息。Job 运行时的信息保存在 `JobDataMap` 实例中。例如：

```java
public class HelloJob implements Job {
    private static Logger log = LoggerFactory.getLogger(HelloJob.class);  
    public HelloJob() { }  
    public void execute(JobExecutionContext context) throws JobExecutionException {
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        log.error("Hello Job执行时间: " + printTime);
    }
} 
```

##### Job状态与并发

关于job的状态数据（即JobDataMap）和并发性，还有一些地方需要注意。在job类上可以加入一些注解，这些注解会影响job的状态和并发性。

- @DisallowConcurrentExecution：将该注解加到job类上，告诉Quartz不要并发地执行同一个job定义（这里指特定的job类）的多个实例。该限制是针对JobDetail的，而不是job类的。但是我们认为（在设计Quartz的时候）应该将该注解放在job类上，因为job类的改变经常会导致其行为发生变化。

- @PersistJobDataAfterExecution：将该注解加在job类上，告诉Quartz在成功执行了job类的execute方法后（没有发生任何异常），更新JobDetail中JobDataMap的数据，使得该job（即JobDetail）在下一次执行的时候，JobDataMap中是更新后的数据，而不是更新前的旧数据。和   @DisallowConcurrentExecution注解一样，尽管注解是加在job类上的，但其限制作用是针对job实例的，而不是job类的。由job类来承载注解，是因为job类的内容经常会影响其行为状态（比如，job类的execute方法需要显式地“理解”其”状态“）。

如果你使用了@PersistJobDataAfterExecution注解，强烈建议你同时使用@DisallowConcurrentExecution注解，因为当同一个job（JobDetail）的两个实例被并发执行时，由于竞争，JobDataMap中存储的数据很可能是不确定的。

#### JobDetailImpl 类 / JobDetail 接口

`JobDetailImpl`类实现了`JobDetail`接口，用来描述一个 job，定义了job所有属性及其 `get/set` 方法。下面是 job 内部的主要属性：

| 属性名        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| class         | 必须是job实现类（比如`JobImpl`），用来绑定一个具体`job`      |
| name          | job 名称。如果未指定，会自动分配一个唯一名称。所有job都必须拥有一个唯一`name`，如果两个 job 的`name`重复，则只有最前面的 job 能被调度 |
| group         | job 所属的组名                                               |
| description   | job描述                                                      |
| durability    | 是否持久化。如果job设置为非持久，当没有活跃的`trigger`与之关联的时候，job 会自动从`scheduler`中删除。也就是说，非持久`job`的生命期是由`trigger`的存在与否决定的 |
| shouldRecover | 是否可恢复。如果 job 设置为可恢复，一旦 job 执行时`scheduler`发生`hard shutdown`（比如进程崩溃或关机），当`scheduler`重启后，该`job`会被重新执行 |
| jobDataMap    | 除了上面常规属性外，用户可以把任意`kv`数据存入`jobDataMap`，实现 job 属性的无限制扩展，执行 job 时可以使用这些属性数据。此属性的类型是`JobDataMap`，实现了`Serializable`接口，可做跨平台的序列化传输 |

通过JobDetail对象，可以给job实例配置的其它属性有：

- Durability：如果一个job是非持久的，当没有活跃的trigger与之关联的时候，会被自动地从scheduler中删除。也就是说，非持久的job的生命期是由trigger的存在与否决定的；
- RequestsRecovery：如果一个job是可恢复的，并且在其执行的时候，scheduler发生硬关闭（hard   shutdown)（比如运行的进程崩溃了，或者关机了），则当scheduler重新启动的时候，该job会被重新执行。此时，该job的JobExecutionContext.isRecovering()  返回true。

#### JobDataMap

JobDataMap中可以包含不限量的（序列化的）数据对象，在job实例执行的时候，可以使用其中的数据；JobDataMap是Java Map接口的一个实现，额外增加了一些便于存取基本类型的数据的方法。

JobDataMap实现了JDK的Map接口，可以以Key-Value的形式存储数据。JobDetail、Trigger都可以使用JobDataMap来设置一些参数或信息，Job执行execute()方法的时候，JobExecutionContext可以获取到JobExecutionContext中的信息

将job加入到scheduler之前，在构建JobDetail时，可以将数据放入JobDataMap，如下示例：

```java
// define the job and tie it to our DumbJob class
  JobDetail job = newJob(DumbJob.class)
      .withIdentity("myJob", "group1") // name "myJob", group "group1"
      .usingJobData("jobSays", "Hello World!")
      .usingJobData("myFloatValue", 3.141f)
      .build();
```

在job的执行过程中，可以从JobDataMap中取出数据，如下示例：

```java
public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
      throws JobExecutionException
    {
      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getJobDetail().getJobDataMap();

      String jobSays = dataMap.getString("jobSays");
      float myFloatValue = dataMap.getFloat("myFloatValue");

      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
  }
```

如果你在job类中，为JobDataMap中存储的数据的key增加set方法（如在上面示例中，增加setJobSays(String 
val)方法），那么Quartz的默认JobFactory实现在job被实例化的时候会自动调用这些set方法，这样你就不需要在execute()方法中显式地从map中取数据了。

在Job执行时，JobExecutionContext中的JobDataMap为我们提供了很多的便利。它是JobDetail中的JobDataMap和Trigger中的JobDataMap的并集，但是如果存在相同的数据，则后者会覆盖前者的值。

```java
 public class DumbJob implements Job {

        public DumbJob() {
        }

        public void execute(JobExecutionContext context)
          throws JobExecutionException
        {
            JobKey key = context.getJobDetail().getKey();

            JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

            String jobSays = dataMap.getString("jobSays");
            float myFloatValue = dataMap.getFloat("myFloatValue");
            ArrayList state = (ArrayList)dataMap.get("myStateData");
            state.add(new Date());

            System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
        }
    }
```

如果你希望使用JobFactory实现数据的自动“注入”，则示例代码为：

```java
  public class DumbJob implements Job {


    String jobSays;
    float myFloatValue;
    ArrayList state;

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
      throws JobExecutionException
    {
      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

      state.add(new Date());

      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }

    public void setJobSays(String jobSays) {
      this.jobSays = jobSays;
    }

    public void setMyFloatValue(float myFloatValue) {
      myFloatValue = myFloatValue;
    }

    public void setState(ArrayList state) {
      state = state;
    }

  }
```

你也许发现，整体上看代码更多了，但是execute()方法中的代码更简洁了。而且，虽然代码更多了，但如果你的IDE可以自动生成setter方法，你就不需要写代码调用相应的方法从JobDataMap中获取数据了，所以你实际需要编写的代码更少了。

##### JobExecutionException

最后，是关于Job.execute(..)方法的一些额外细节。execute方法中仅允许抛出一种类型的异常（包括RuntimeExceptions），即JobExecutionException。因此，你应该将execute方法中的所有内容都放到一个”try-catch”块中。你也应该花点时间看看JobExecutionException的文档，因为你的job可以使用该异常告诉scheduler，你希望如何来处理发生的异常。

**其他**

JobDetail绑定指定的Job，每次Scheduler调度执行一个Job的时候，首先会拿到对应的Job，然后创建该Job实例，再去执行Job中的execute()的内容，**任务执行结束后**，关联的Job对象实例会被释放，且会被JVM GC清除。

为什么设计成JobDetail + Job，不直接使用Job

> JobDetail定义的是任务数据，而真正的执行逻辑是在Job中。
> 这是因为任务是有可能并发执行，如果Scheduler直接使用Job，就会存在对同一个Job实例并发访问的问题。而JobDetail & Job 方式，Sheduler每次执行，都会根据JobDetail创建一个新的Job实例，这样就可以规避并发访问的问题。

通过这些介绍，所以Job实例实现时，需要注意以下几点：

- 它必须具有一个无参构造函数
- 它不应该有静态数据类型。因为每次job实例执行完之后便被回收，而静态成员变量因为依附于类存在，并不能被回收，如果静态变量的值被某个类修改，它的值就没法被保护

### Trigger

Trigger是一个类，描述触发Job执行的时间触发规则。主要有  `SimpleTrigger`  和  `CronTrigger`  这两个子类。当仅需触发一次或者以固定时间间隔周期执行，`SimpleTrigger`是最适合的选择；而`CronTrigger`则可以通过`Cron`表达式定义出各种复杂时间规则的调度方案：如每早晨9:00执行，周一、周三、周五下午5:00执行等；

以下是 trigger 的属性：

| 属性名             | 属性类型          | 说明                                                         |
| ------------------ | ----------------- | ------------------------------------------------------------ |
| name               | 所有trigger通用   | trigger名称                                                  |
| group              | 所有trigger通用   | trigger所属的组名                                            |
| description        | 所有trigger通用   | trigger描述                                                  |
| calendarName       | 所有trigger通用   | 日历名称，指定使用哪个Calendar类，经常用来从trigger的调度计划中排除某些时间段 |
| misfireInstruction | 所有trigger通用   | 错过job（未在指定时间执行的job）的处理策略，默认为MISFIRE_INSTRUCTION_SMART_POLICY。详见这篇[blog](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fspbdev%2Farticle%2Fdetails%2F41679477)[^Quartz misfire](https://link.jianshu.com?t=%5BSpbDev%5D(http%3A%2F%2Fmy.csdn.net%2FSpbDev)%EF%BC%9A%5BQuartz%E7%9A%84misfire%5D(http%3A%2F%2Fblog.csdn.net%2Fspbdev%2Farticle%2Fdetails%2F41679477)) |
| priority           | 所有trigger通用   | 优先级，默认为5。当多个trigger同时触发job时，线程池可能不够用，此时根据优先级来决定谁先触发 |
| jobDataMap         | 所有trigger通用   | 同job的jobDataMap。假如job和trigger的jobDataMap有同名key，通过getMergedJobDataMap()获取的jobDataMap，将以trigger的为准 |
| startTime          | 所有trigger通用   | 触发开始时间，默认为当前时间。决定什么时间开始触发job        |
| endTime            | 所有trigger通用   | 触发结束时间。决定什么时间停止触发job                        |
| nextFireTime       | SimpleTrigger私有 | 下一次触发job的时间                                          |
| previousFireTime   | SimpleTrigger私有 | 上一次触发job的时间                                          |
| repeatCount        | SimpleTrigger私有 | 需触发的总次数                                               |
| timesTriggered     | SimpleTrigger私有 | 已经触发过的次数                                             |
| repeatInterval     | SimpleTrigger私有 | 触发间隔时间                                                 |

#### Simple Trigger

SimpleTrigger可以满足的调度需求是：在具体的时间点执行一次，或者在具体的时间点执行，并且以指定的间隔重复执行若干次。比如，你有一个trigger，你可以设置它在2015年1月13日的上午11:23:54准时触发，或者在这个时间点触发，并且每隔2秒触发一次，一共重复5次。

根据描述，你可能已经发现了，SimpleTrigger的属性包括：**开始时间、结束时间、重复次数以及重复的间隔**。这些属性的含义与你所期望的是一致的，只是关于结束时间有一些地方需要注意。

重复次数，可以是0、正整数，以及常量SimpleTrigger.REPEAT_INDEFINITELY。重复的间隔，必须是0，或者long型的正数，表示毫秒。注意，如果重复间隔为0，trigger将会以重复次数并发执行(或者以scheduler可以处理的近似并发数)。

如果你还不熟悉DateBuilder，了解后你会发现使用它可以非常方便地构造基于开始时间(或终止时间)的调度策略。

endTime属性的值会覆盖设置重复次数的属性值；比如，你可以创建一个trigger，在终止时间之前每隔10秒执行一次，你不需要去计算在开始时间和终止时间之间的重复次数，只需要设置终止时间并将重复次数设置为REPEAT_INDEFINITELY(当然，你也可以将重复次数设置为一个很大的值，并保证该值比trigger在终止时间之前实际触发的次数要大即可)。

下面的程序就实现了程序运行5s后开始执行Job，执行Job 5s后结束执行：

```java
Date startDate = new Date();
startDate.setTime(startDate.getTime() + 5000);

 Date endDate = new Date();
 endDate.setTime(startDate.getTime() + 5000);

        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
                .usingJobData("trigger1", "这是jobDetail1的trigger")
                .startNow()//立即生效
                .startAt(startDate)
                .endAt(endDate)
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(1)//每隔1s执行一次
                .repeatForever()).build();//一直执行

```

##### SimpleTrigger Misfire策略

SimpleTrigger有几个misfire相关的策略，告诉quartz当misfire发生的时候应该如何处理。这些策略以常量的形式在SimpleTrigger中定义(JavaDoc中介绍了它们的功能)。这些策略包括：

SimpleTrigger的Misfire策略常量：

```
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_FIRE_NOW
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT
```

如果使用smart policy，SimpleTrigger会根据实例的配置及状态，在所有MISFIRE策略中动态选择一种Misfire策略。SimpleTrigger.updateAfterMisfire()的JavaDoc中解释了该动态行为的具体细节。

在使用SimpleTrigger构造trigger时，misfire策略作为基本调度(simple schedule)的一部分进行配置(通过SimpleSchedulerBuilder设置)：

```java
    trigger = newTrigger()
        .withIdentity("trigger7", "group1")
        .withSchedule(simpleSchedule()
            .withIntervalInMinutes(5)
            .repeatForever()
            .withMisfireHandlingInstructionNextWithExistingCount())
        .build();
```

#### CronTrigger

CronTrigger通常比Simple Trigger更有用，如果您需要基于日历的概念而不是按照SimpleTrigger的精确指定间隔进行重新启动的作业启动计划。

使用CronTrigger，您可以指定号时间表，例如“每周五中午”或“每个工作日和上午9:30”，甚至“每周一至周五上午9:00至10点之间每5分钟”和1月份的星期五“。

即使如此，和SimpleTrigger一样，CronTrigger有一个startTime，它指定何时生效，以及一个（可选的）endTime，用于指定何时停止计划。

建立一个触发器，每隔一分钟，每天上午8点至下午5点之间：

```java
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
    .forJob("myJob", "group1")
    .build();
```

CroTrigger是基于Cron表达式的，先了解下Cron表达式：
由7个子表达式组成字符串的，格式如下：

> [秒] [分] [小时] [日] [月] [周] [年]

更多查看下一篇文章

##### CronTrigger Misfire说明

以下说明可以用于通知Quartz当CronTrigger发生失火时应该做什么。这些指令定义为CronTrigger本身的常量（包括描述其行为的JavaDoc）。说明包括：

CronTrigger的Misfire指令常数

```
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_DO_NOTHING
MISFIRE_INSTRUCTION_FIRE_NOW
```

所有触发器还具有可用的Trigger.MISFIRE_INSTRUCTION_SMART_POLICY指令，并且该指令也是所有触发器类型的默认值。“智能策略”指令由CronTrigger解释为MISFIRE_INSTRUCTION_FIRE_NOW。CronTrigger.updateAfterMisfire（）方法的JavaDoc解释了此行为的确切细节。

在构建CronTriggers时，您可以将misfire指令指定为简单计划的一部分（通过CronSchedulerBuilder）：

```java
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?")
        .withMisfireHandlingInstructionFireAndProceed())
    .forJob("myJob", "group1")
    .build();
```

### Calendar

`org.quartz.Calendar`和 `java.util.Calendar`不同，它是一些日历特定时间点的集合（可以简单地将`org.quartz.Calendar`看作`java.util.Calendar`的集合——`java.util.Calendar`代表一个日历时间点，无特殊说明后面的`Calendar`即指`org.quartz.Calendar`）。一个Trigger可以和多个Calendar关联，以便排除或包含某些时间点。假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在`Trigger`触发机制的基础上使用Calendar进行定点排除。

任何实现了Calendar接口的可序列化对象都可以作为Calendar对象，Calendar接口如下：

```java
package org.quartz;

public interface Calendar {

  public boolean isTimeIncluded(long timeStamp);

  public long getNextIncludedTime(long timeStamp);

}
```

注意到这些方法的参数类型为long。你也许猜到了，他们就是**毫秒单位**的时间戳。即Calendar排除时间段的单位可以精确到毫秒。你也许对“排除一整天”的Calendar比较感兴趣。Quartz提供的org.quartz.impl.HolidayCalendar类可以很方便地实现。

Calendar必须先实例化，然后通过addCalendar()方法注册到scheduler。如果使用HolidayCalendar，实例化后，需要调用addExcludedDate(Date  date)方法从调度计划中排除时间段。以下示例是将同一个Calendar实例用于多个trigger：

```java
HolidayCalendar cal = new HolidayCalendar();
cal.addExcludedDate( someDate );
cal.addExcludedDate( someOtherDate );

sched.addCalendar("myHolidays", cal, false);


Trigger t = newTrigger()
    .withIdentity("myTrigger")
    .forJob("myJob")
    .withSchedule(dailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
    .modifiedByCalendar("myHolidays") // but not on holidays
    .build();

// .. schedule job with trigger

Trigger t2 = newTrigger()
    .withIdentity("myTrigger2")
    .forJob("myJob2")
    .withSchedule(dailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
    .modifiedByCalendar("myHolidays") // but not on holidays
    .build();

// .. schedule job with trigger2
```

上面的代码创建了两个触发器，每个触发器都计划每天触发。然而，在日历所排除的期间内发生的任何触发都将被跳过。

### Scheduler

调度器，代表一个**Quartz**的独立运行容器，好比一个『大管家』，这个大管家应该可以接受 `Job`， 然后按照各种`Trigger`去运行，**Trigger**和**JobDetail**可以注册到Scheduler中，两者在Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据，**Trigger的组及名称必须唯一，JobDetail的组和名称也必须唯一**（但可以和Trigger的组和名称相同，因为它们是不同类型的）。Scheduler定义了多个接口方法，允许外部通过组及名称访问和控制容器中Trigger和JobDetail。

![2.png](https://i.loli.net/2019/05/31/5cf11e6022cf996089.png)

Scheduler 可以将 Trigger 绑定到某一 JobDetail 中，这样当 Trigger 触发时，对应的 Job 就被执行。可以通过 SchedulerFactory创建一个 Scheduler 实例。Scheduler 拥有一个 SchedulerContext，它类似于 ServletContext，保存着 Scheduler 上下文信息，Job 和 Trigger 都可以访问 SchedulerContext 内的信息。SchedulerContext 内部通过一个 Map，以键值对的方式维护这些上下文数据，SchedulerContext 为保存和获取数据提供了多个 put() 和 getXxx() 的方法。可以通过`Scheduler# getContext()`获取对应的`SchedulerContext`实例；

![6.png](https://i.loli.net/2019/05/31/5cf12714b80d419557.png)

JobExecutionContext中包含了Quartz运行时的环境以及Job本身的详细数据信息。

当Schedule调度执行一个Job的时候，就会将JobExecutionContext传递给该Job的execute()中，Job就可以通过JobExecutionContext对象获取信息。

### Listeners

Listeners是用于根据调度程序中发生的事件执行操作。TriggerListeners接收到与触发器（trigger）相关的事件，JobListeners 接收与jobs相关的事件。

与触发相关的事件包括：触发器触发，触发失灵，触发完成（触发器关闭的作业完成）。

org.quartz.TriggerListener接口

```java
public interface TriggerListener {

    public String getName();

    public void triggerFired(Trigger trigger, JobExecutionContext context);

    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

    public void triggerMisfired(Trigger trigger);

    public void triggerComplete(Trigger trigger, JobExecutionContext context,
            int triggerInstructionCode);
}
```

job相关事件包括：job即将执行的通知，以及job完成执行时的通知。

org.quartz.JobListener接口

```java
public interface JobListener {

    public String getName();

    public void jobToBeExecuted(JobExecutionContext context);

    public void jobExecutionVetoed(JobExecutionContext context);

    public void jobWasExecuted(JobExecutionContext context,
            JobExecutionException jobException);

}
```

##### 使用自己的Listeners

要创建一个listener，只需创建一个实现org.quartz.TriggerListener和/或org.quartz.JobListener接口的对象。然后，listener在运行时会向调度程序注册，并且必须给出一个名称（或者，他们必须通过他们的getName（）方法来宣传自己的名字）。

为了方便起见，实现这些接口，也可以扩展JobListenerSupport类或TriggerListenerSupport类，并且只需覆盖需要的方法

listener与调度程序的ListenerManager一起注册，并配有描述listener希望接收事件的job/触发器的Matcher。

```
Listener在运行时间内与调度程序一起注册，并且不与jobs和触发器一起存储在JobStore中。这是因为听众通常是与应用程序的集成点。因此，每次运行应用程序时，都需要重新注册该调度程序。
```

**添加对特定job感兴趣的JobListener：**

```
scheduler.getListenerManager().addJobListener(myJobListener, jobKeyEquals(jobKey("myJobName", "myJobGroup")));
```

添加对两个特定组的所有job感兴趣的JobListener：

```
scheduler.getListenerManager().addJobListener(myJobListener, or(jobGroupEquals("myJobGroup"), jobGroupEquals("yourGroup")));
```

添加对所有job感兴趣的JobListener：

```
scheduler.getListenerManager().addJobListener(myJobListener, allJobs());
```

注册TriggerListeners的工作原理相同。

#### SchedulerListeners

SchedulerListeners非常类似于TriggerListeners和JobListeners，除了它们在Scheduler本身中接收到事件的通知  不一定与特定触发器（trigger）或job相关的事件。

与计划程序相关的事件包括：添加job/触发器，删除job/触发器，调度程序中的严重错误，关闭调度程序的通知等。

org.quartz.SchedulerListener接口

```java
public interface SchedulerListener {

    public void jobScheduled(Trigger trigger);

    public void jobUnscheduled(String triggerName, String triggerGroup);

    public void triggerFinalized(Trigger trigger);

    public void triggersPaused(String triggerName, String triggerGroup);

    public void triggersResumed(String triggerName, String triggerGroup);

    public void jobsPaused(String jobName, String jobGroup);

    public void jobsResumed(String jobName, String jobGroup);

    public void schedulerError(String msg, SchedulerException cause);

    public void schedulerStarted();

    public void schedulerInStandbyMode();

    public void schedulerShutdown();

    public void schedulingDataCleared();
}
```

SchedulerListeners注册到调度程序的ListenerManager。SchedulerListeners几乎可以实现任何实现org.quartz.SchedulerListener接口的对象。

添加SchedulerListener：

```
scheduler.getListenerManager().addSchedulerListener(mySchedListener);
```

删除SchedulerListener：

```
scheduler.getListenerManager().removeSchedulerListener(mySchedListener);
```

### 参考

1. [定时任务框架Quartz](https://blog.csdn.net/noaman_wgs/article/details/80984873)