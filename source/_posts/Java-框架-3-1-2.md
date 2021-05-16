---
title: 任务框架--Quartz (二)
date: 2019-05-21 12:19:59
tags:
 - Java
 - 框架
 - 任务调度
categories:
 - Java
 - 框架
---

### Quartz 线程视图

在Quartz中，有两类线程，Scheduler调度线程和任务执行线程，其中任务执行线程通常使用一个线程池维护一组线程。

<!--more-->

![UTOOLS1576331585151.png](https://i.loli.net/2019/12/14/ox1irB5IT83FduO.png)

Scheduler调度线程主要有两个：执行常规调度的线程，和执行misfiredtrigger的线程。常规调度线程轮询存储的所有trigger，如果有需要触发的trigger，即到达了下一次触发的时间，则从任务执行线程池获取一个空闲线程，执行与该trigger关联的任务。Misfire线程是扫描所有的trigger，查看是否有misfiredtrigger，如果有的话根据misfire的策略分别处理(**fire now** OR **wait for the next fire**)。

Scheduler 使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高运行效率。

**QuartzSchedulerThread 主调度线程**

![UTOOLS1576332252675.png](https://i.loli.net/2019/12/14/xfaudz1hIPDi95A.png)

### Job Stores

Quartz中的trigger和job需要存储下来才能被使用，Quartz中有两种存储方式：RAMJobStore,JobStoreSupport。

其中RAMJobStore是将trigger和job存储在内存中，而JobStoreSupport是基于jdbc将trigger和job存储到数据库中。RAMJobStore的存取速度非常快，但是由于其在系统被停止后所有的数据都会丢失，所以在集群应用中，必须使用JobStoreSupport。

#### RAMJobStore

RAMJobStore是使用最简单的JobStore，它也是性能最高的（在CPU时间方面）。RAMJobStore以其明显的方式获取其名称：它将其所有数据保存在RAM中。这就是为什么它是闪电般快的，也是为什么这么简单的配置。缺点是当您的应用程序结束（或崩溃）时，所有调度信息都将丢失 。 这意味着RAMJobStore无法履行作业和triggers上的“非易失性”设置。对于某些应用程序，这是可以接受的 。 甚至是所需的行为，但对于其他应用程序，这可能是灾难性的。

要使用RAMJobStore（并假设您使用的是StdSchedulerFactory），只需将类名称org.quartz.simpl.RAMJobStore指定为用于配置的JobStore类属性：

配置Quartz以使用RAMJobStore

```
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

没有其他设置。

#### JDBC JobStore

JDBCJobStore也被恰当地命名 -  它通过JDBC将其所有数据保存在数据库中。因此，配置比RAMJobStore要复杂一点，而且也不是那么快。但是，性能下降并不是很糟糕，特别是如果您在主键上构建具有索引的数据库表。在相当现代的一套具有体面的LAN（在调度程序和数据库之间）的机器上，检索和更新触发triggers的时间通常将小于10毫秒。

JDBCJobStore几乎与任何数据库一起使用，已被广泛应用于Oracle，PostgreSQL，MySQL，MS   SQLServer，HSQLDB和DB2。要使用JDBCJobStore，必须首先创建一组数据库表以供Quartz使用。

创建表后，在配置和启动JDBCJobStore之前，您还有一个重要的决定。您需要确定应用程序需要哪种类型的事务。如果您不需要将调度命令（例如添加和删除triggers）绑定到其他事务，那么可以通过使用JobStoreTX作为JobStore 来管理事务（这是最常见的选择）。

如果需要Quartz与其他事务（即J2EE应用程序服务器）一起工作，那么应该使用JobStoreCMT - 在这种情况下，Quartz将让应用程序服务器容器管理事务。

最后一个难题是设置一个DataSource，JDBCJobStore可以从中获取与数据库的连接。DataSources在Quartz属性中使用几种不同的方法之一进行定义。一种方法是让Quartz创建和管理DataSource本身  -  通过提供数据库的所有连接信息。另一种方法是让Quartz使用由Quartz正在运行的应用程序服务器管理的DataSource

要使用JDBCJobStore（并假定您使用的是StdSchedulerFactory），首先需要将Quartz配置的JobStore类属性设置为org.quartz.impl.jdbcjobstore.JobStoreTX或org.quartz.impl.jdbcjobstore.JobStoreCMT  - 具体取决于根据上述几段的解释，您所做的选择。

**配置Quartz以使用JobStoreTx**

```
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
```

接下来，您需要为JobStore选择一个DriverDelegate才能使用。DriverDelegate负责执行特定数据库可能需要的任何JDBC工作。StdJDBCDelegate是一个使用“vanilla”JDBC代码（和SQL语句）来执行其工作的委托。如果没有为您的数据库专门制作另一个代理，请尝试使用此委托 - 我们仅为数据库制作了特定于数据库的代理，我们使用StdJDBCDelegate（似乎最多！）发现了问题。

其他代理可以在“org.quartz.impl.jdbcjobstore”包或其子包中找到。其他代理包括DB2v6Delegate（用于DB2版本6及更早版本），HSQLDBDelegate（用于HSQLDB），MSSQLDelegate（用于Microsoft SQLServer），PostgreSQLDelegate（用于PostgreSQL）），WeblogicDelegate（用于使用Weblogic创建的JDBC驱动程序）

选择委托后，将其类名设置为JDBCJobStore的委托使用。

配置JDBCJobStore以使用DriverDelegate

```
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
```

接下来，您需要通知JobStore您正在使用的表前缀（如上所述）。

使用表前缀配置JDBCJobStore

```
org.quartz.jobStore.tablePrefix = QRTZ_
```

最后，您需要设置JobStore应该使用哪个DataSource。命名的DataSource也必须在Quartz属性中定义。在这种情况下，我们指定Quartz应该使用DataSource名称“myDS”（在配置属性中的其他位置定义）。

使用要使用的DataSource的名称配置JDBCJobStore

```
org.quartz.jobStore.dataSource = myDS
```

> 如果您的计划程序正忙（即几乎总是执行与线程池大小相同的job数量），那么您应该将DataSource中的连接数设置为线程池+ 2的大小。

> 可以将“org.quartz.jobStore.useProperties”配置参数设置为“true”（默认为false），以指示JDBCJobStore将JobDataMaps中的所有值都作为字符串，因此可以作为名称  -  值对存储而不是在BLOB列中以其序列化形式存储更多复杂的对象。从长远来看，这是更安全的，因为您避免了将非String类序列化为BLOB的类版本问题。

#### TerracottaJobStore

TerracottaJobStore提供了一种缩放和鲁棒性的手段，而不使用数据库。这意味着您的数据库可以免受Quartz的负载，可以将其所有资源保存为应用程序的其余部分。

TerracottaJobStore可以运行群集或非群集，并且在任一情况下，为应用程序重新启动之间持续的作业数据提供存储介质，因为数据存储在Terracotta服务器中。它的性能比通过JDBCJobStore使用数据库要好得多（约一个数量级更好），但比RAMJobStore要慢。

要使用TerracottaJobStore（并且假设您使用的是StdSchedulerFactory），只需将类名称org.quartz.jobStore.class  =  org.terracotta.quartz.TerracottaJobStore指定为用于配置石英的JobStore类属性，并添加一个额外的行配置来指定Terracotta服务器的位置：

配置Quartz以使用TerracottaJobStore

```
org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore
org.quartz.jobStore.tcConfigUrl = localhost:9510
```

### Cron 表达式

熟悉linux的可能会接触过。

[quartz/Cron/Crontab表达式在线生成工具](http://www.bejson.com/othertools/cron/)

cron的表达式被用来配置CronTrigger实例。 Cron表达式由7个部分组成，各部分用空格隔开，例如`0 0 12 ? * WED（每星期三下午12:00 执行）`， Cron表达式的7个部分从左到右代表的含义如下 `Seconds Minutes Hours Day-of-Month Month Day-of-Week Year`， 其中Year是可选的

**1.Seconds**
 秒：数字0－59
**2. Minutes**
 分：数字0－59
**3. Hours**
 时 ：数字0-23
**4. Day-of-Month**
 月中的几号 ：可以用数字1-31 中的任一一个值，但要注意一些特别的月份
**5.Month**
 一年中的几月：可以用0-11 或用字符串  “JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV and DEC” 表示
**6.Day-of-Week**
每周：数字1-7（1 ＝ 星期日），或用字符口串“SUN, MON, TUE, WED, THU, FRI and SAT”

| 字段                       | 允许值                                   | 允许的特殊字符                 |
| -------------------------- | ---------------------------------------- | ------------------------------ |
| `秒（Seconds）`            | `0~59的整数`                             | `, - * /    四个字符`          |
| `分（*Minutes*）`          | `0~59的整数`                             | `, - * /    四个字符`          |
| `小时（*Hours*）`          | `0~23的整数`                             | `, - * /    四个字符`          |
| `日期（*DayofMonth*）`     | `1~31的整数（但是你需要考虑你月的天数）` | `,- * ? / L W C     八个字符`  |
| `月份（*Month*）`          | `1~12的整数或者 JAN-DEC`                 | `, - * /    四个字符`          |
| `星期（*DayofWeek*）`      | `1~7的整数或者 SUN-SAT （1=SUN）`        | `, - * ? / L C #     八个字符` |
| `年(可选，留空)（*Year*）` | `1970~2099`                              | `, - * /    四个字符`          |

**Cron中的符号**

每一个域都使用数字，但还可以出现如下特殊字符，它们的含义是：

（1）\*：表示匹配该域的任意值。假如在Minutes域使用\*, 即表示每分钟都会触发事件。

（2）?：只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： `13 13 15 20 * ?`, 其中最后一位只能用？，而不能使用\*，如果使用\*表示不管星期几都会触发，实际上并不是这样。

（3）-：表示范围。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次 

（4）/：表示起始时间开始触发，然后每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次. 

（5）,：表示列出枚举值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。 

（6）L：表示最后，只能出现在DayofWeek和DayofMonth域。如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。 

（7）W:表示有效工作日(周一到周五),只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份 。

（8）LW:这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。 

（9）#:用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。 

**范例**：

```
*/5 * * * * ?  每隔5秒执行一次
0 */1 * * * ?  每隔1分钟执行一次
0 0 23 * * ?  每天23点执行一次
0 0 1 * * ?  每天凌晨1点执行一次：
0 0 1 1 * ?  每月1号凌晨1点执行一次
0 0 23 L * ?  每月最后一天23点执行一次
0 0 1 ? * L  每周星期天凌晨1点实行一次
0 26,29,33 * * * ?  在26分、29分、33分执行一次
0 0 0,13,18,21 * * ? 每天的0点、13点、18点、21点都执行一次

0/20 * * * * ?   表示每20秒 调整任务
0 0 2 1 * ?   表示在每月的1日的凌晨2点调整任务
0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业
0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作
0 0 10,14,16 * * ?   每天上午10点，下午2点，4点 
0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时 
0 0 12 ? * WED    表示每个星期三中午12点 
0 0 12 * * ?   每天中午12点触发 
0 15 10 ? * *    每天上午10:15触发 
0 15 10 * * ?     每天上午10:15触发 
0 15 10 * * ? *    每天上午10:15触发 
0 15 10 * * ? 2005    2005年的每天上午10:15触发 
0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发 
0 0/5 14 * * ?    在每天下午2点到下午2:55期间的每5分钟触发 
0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 
0 0-5 14 * * ?    在每天下午2点到下午2:05期间的每1分钟触发 
0 10,44 14 ? 3 WED    每年三月的星期三的下午2:10和2:44触发 
0 15 10 ? * MON-FRI    周一至周五的上午10:15触发 
0 15 10 15 * ?    每月15日上午10:15触发 
0 15 10 L * ?    每月最后一日的上午10:15触发 
0 15 10 ? * 6L    每月的最后一个星期五上午10:15触发 
0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期五上午10:15触发 
0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发 
```

#### 配置文件

```properties
#===============================================================        
#配置文件不是必须的，Quartz对配置项都是有默认值的，当需要自定义的时候，
#可以在classpath路径下放一个quartz.properties文件，Quartz的StdSchedulerFactory
#在启动时会自动加载该配置文件。
#===============================================================    
 
 
#===============================================================        
#配置主调度程序的属性        
#===============================================================    
org.quartz.scheduler.instanceName = DefaultQuartzScheduler
org.quartz.scheduler.rmi.export = false
org.quartz.scheduler.rmi.proxy = false
org.quartz.scheduler.wrapJobExecutionInUserTransaction = false
#当检查某个Trigger应该触发时，默认每次只Acquire一个Trigger，（为什么要有Acquire的过程呢？是为了防止多线程访问的情况下，
#同一个Trigger被不同的线程多次触发）。尤其是使用JDBC JobStore时，一次Acquire就是一个update语句，尽可能一次性的多获取
#几个Trigger，一起触发，当定时器数量非常大的时候，这是个非常有效的优化。当定时器数量比较少时，触发不是极为频繁时，
#这个优化的意义就不大了。
org.quartz.scheduler.batchTriggerAcquisitionMaxCount=50
 
#===============================================================        
#配置线程池的属性
#===============================================================          
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
#线程池里的线程数，默认值是10，当执行任务会并发执行多个耗时任务时，要根据业务特点选择线程池的大小。
org.quartz.threadPool.threadCount = 50
org.quartz.threadPool.threadPriority = 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true
 
#===============================================================        
#配置JobStore的属性
#===============================================================          
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

### 示例

#### 动态添加、修改和删除定时任务

```java
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;


/**
 * @author hyp
 * Project name is FrameWork
 * Include in com.hyp.learn
 * hyp create at 2019/5/31
 **/
public class QuartzManager {
    private static SchedulerFactory gSchedulerFactory = new StdSchedulerFactory();
    private static String JOB_GROUP_NAME = "EXTJWEB_JOBGROUP_NAME";
    private static String TRIGGER_GROUP_NAME = "EXTJWEB_TRIGGERGROUP_NAME";

    /**
     * @param jobName 任务名
     * @param cls     任务
     * @param time    时间设置，参考quartz说明文档
     * @Description: 添加一个定时任务，使用默认的任务组名，触发器名，触发器组名
     */
    @SuppressWarnings("unchecked")
    public static void addJob(String jobName, Class cls, String time) {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            JobDetail jobDetail = JobBuilder.newJob(cls)
                    .withIdentity(jobName, JOB_GROUP_NAME)
                    .build();

            // 触发器
            CronTrigger trigger = (CronTrigger) TriggerBuilder
                    .newTrigger()
                    .withIdentity(jobName, TRIGGER_GROUP_NAME)
                    .startNow()
                    .withSchedule(CronScheduleBuilder.cronSchedule(time))
                    .build();// 触发器名,触发器组

            sched.scheduleJob(jobDetail, trigger);
            // 启动
            if (!sched.isShutdown()) {
                sched.start();
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @param jobName          任务名
     * @param jobGroupName     任务组名
     * @param triggerName      触发器名
     * @param triggerGroupName 触发器组名
     * @param jobClass         任务
     * @param time             时间设置，参考quartz说明文档
     * @Description: 添加一个定时任务
     */
    @SuppressWarnings("unchecked")
    public static void addJob(String jobName, String jobGroupName, Class jobClass,
                              String triggerName, String triggerGroupName,
                              String time) {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            JobDetail jobDetail = JobBuilder
                    .newJob(jobClass)
                    .withIdentity(jobName, jobGroupName)
                    .build();// 任务名，任务组，任务执行类
            // 触发器
            CronTrigger trigger = TriggerBuilder
                    .newTrigger()
                    .withIdentity(triggerName, triggerGroupName)
                    .startNow()
                    .withSchedule(CronScheduleBuilder.cronSchedule(time))
                    .build();// 触发器名,触发器组

            sched.scheduleJob(jobDetail, trigger);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @param jobName
     * @param time
     * @Description: 修改一个任务的触发时间(使用默认的任务组名 ， 触发器名 ， 触发器组名)
     */
    public static void modifyJobTime(String jobName, String time) {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            CronTrigger trigger = (CronTrigger) sched.getTrigger(new TriggerKey(jobName, TRIGGER_GROUP_NAME));
            if (trigger == null) {
                return;
            }
            String oldTime = trigger.getCronExpression();
            if (!oldTime.equalsIgnoreCase(time)) {
                JobDetail jobDetail = sched.getJobDetail(new JobKey(jobName, JOB_GROUP_NAME));
                Class objJobClass = jobDetail.getJobClass();
                removeJob(jobName);
                addJob(jobName, objJobClass, time);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @param triggerName
     * @param triggerGroupName
     * @param time
     * @Description: 修改一个任务的触发时间
     */
    public static void modifyJobTime(String triggerName,
                                     String triggerGroupName, String time) {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            TriggerKey triggerKey = TriggerKey.triggerKey(triggerName, triggerGroupName);
            CronTrigger trigger = (CronTrigger) sched.getTrigger(triggerKey);
            if (trigger == null) {
                return;
            }
            String oldTime = trigger.getCronExpression();
            if (!oldTime.equalsIgnoreCase(time)) {
                // 触发器
                TriggerBuilder<Trigger> triggerBuilder = TriggerBuilder.newTrigger();
                // 触发器名,触发器组
                triggerBuilder.withIdentity(triggerName, triggerGroupName);
                triggerBuilder.startNow();
                // 触发器时间设定
                triggerBuilder.withSchedule(CronScheduleBuilder.cronSchedule(time));
                // 创建Trigger对象
                trigger = (CronTrigger) triggerBuilder.build();
                // 方式一 ：修改一个任务的触发时间
                sched.rescheduleJob(triggerKey, trigger);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @param jobName
     * @Description: 移除一个任务(使用默认的任务组名 ， 触发器名 ， 触发器组名)
     */
    public static void removeJob(String jobName) {

        removeJob(jobName,JOB_GROUP_NAME,jobName,TRIGGER_GROUP_NAME);
    }

    /**
     * @param jobName
     * @param jobGroupName
     * @param triggerName
     * @param triggerGroupName
     * @Description: 移除一个任务
     */
    public static void removeJob(String jobName, String jobGroupName,
                                 String triggerName, String triggerGroupName) {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            sched.pauseTrigger(new TriggerKey(triggerName, triggerGroupName));// 停止触发器
            sched.unscheduleJob(new TriggerKey(triggerName, triggerGroupName));// 移除触发器
            sched.deleteJob(new JobKey(jobName, jobGroupName));// 删除任务
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @Description:启动所有定时任务
     */
    public static void startJobs() {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            sched.start();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * @Description:关闭所有定时任务
     */
    public static void shutdownJobs() {
        try {
            Scheduler sched = gSchedulerFactory.getScheduler();
            if (!sched.isShutdown()) {
                sched.shutdown();
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

HelloJob类：

```java
public class HelloJob implements Job {
    private static Logger log = LoggerFactory.getLogger(HelloJob.class);

    public HelloJob() {
    }

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        log.error("Hello Job执行时间: " + printTime);
    }
}
```

测试类：

```java
public class QuartzManagerTest {

    public static String JOB_NAME = "动态任务调度";
    public static String TRIGGER_NAME = "动态任务触发器";


    @Test
    public void job() {
        try {
            System.out.println("【系统启动】开始(每1秒输出一次)...");
            QuartzManager.addJob(JOB_NAME, HelloJob.class, "0/1 * * * * ?");

            Thread.sleep(5000);
            System.out.println("【修改时间】开始(每5秒输出一次)...");
            QuartzManager.modifyJobTime(JOB_NAME, "0/5 * * * * ?");

            Thread.sleep(6000);
            System.out.println("【移除定时】开始...");
            QuartzManager.removeJob(JOB_NAME);
            System.out.println("【移除定时】成功");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

#### 利用 SpringBoot + Quartz 搭建的界面化的 Demo

本工程所用到的技术或工具

- Spring Boot
- Mybatis
- Quartz
- PageHelper
- VueJS
- ElementUI
- MySql数据库

代码参考：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-6-quartz>

#### Quartz集群架构

一个Quartz集群中的每个节点是一个独立的Quartz应用，它又管理着其他的节点。这就意味着你必须对每个节点分别启动或停止。Quartz集群中，独立的Quartz节点并不与另一其的节点或是管理节点通信，而是通过相同的数据库表来感知到另一Quartz应用的。

![7.png](https://i.loli.net/2019/05/31/5cf1320aea75383686.png)

**数据库准备**

因为Quzrtz集群依赖于数据库，所以必须先创建数据库表，数据表示官方提供的，我用的是quartz2.3.0版本，有11张表，如下：

![3.png](https://i.loli.net/2019/05/31/5cf124766747476408.png)

**表信息介绍**

- qrtz_blob_triggers : 以Blob 类型存储的触发器。

- qrtz_calendars存储Quartz的Calendar信息

- qrtz_cron_triggers存储CronTrigger，包括Cron表达式和时区信息

- qrtz_fired_triggers存储与已触发的Trigger相关的状态信息，以及相联Job的执行信息

- qrtz_job_details存储每一个已配置的Job的详细信息

- qrtz_locks存储程序的悲观锁的信息

- qrtz_paused_trigger_grps存储已暂停的Trigger组的信息

- qrtz_scheduler_state存储少量的有关Scheduler的状态信息，和别的Scheduler实例

- qrtz_simple_triggers存储简单的Trigger，包括重复次数、间隔、以及已触的次数

- qrtz_simprop_triggers   存储CalendarIntervalTrigger和DailyTimeIntervalTrigger两种类型的触发器

- qrtz_triggers存储已配置的Trigger的信息 


**qrtz_locks就是Quartz集群实现同步机制的行锁表,包括以下几个锁：CALENDAR_ACCESS 、JOB_ACCESS、MISFIRE_ACCESS 、STATE_ACCESS 、TRIGGER_ACCESS**

**从故障实例中恢复Job**

当一个Sheduler实例在执行某个Job时失败了，有可能由另一正常工作的Scheduler实例接过这个Job重新运行。要实现这种行为，配置给JobDetail对象的Job可恢复属性必须设置为true（job.setRequestsRecovery(true)）。如果可恢复属性被设置为false(默认为false)，当某个Scheduler在运行该job失败时，它将不会重新运行；而是由另一个Scheduler实例在下一次触发时间触发。Scheduler实例出现故障后多快能被侦测到取决于每个Scheduler的检入间隔。

**项目代码下载：https://github.com/feixiameiruhua/my-quartz-cluster.git** 

![8.png](https://i.loli.net/2019/05/31/5cf132a63668515818.png)

quartz.properties文件

```properties
# 固定前缀org.quartz
# 主要分为scheduler、threadPool、jobStore、plugin等部分
#
#
#调度器实例编号自动生成
org.quartz.scheduler.instanceId = AUTO

#调度器实例名称
org.quartz.scheduler.instanceName = DefaultQuartzScheduler
org.quartz.scheduler.rmi.export = false
org.quartz.scheduler.rmi.proxy = false
org.quartz.scheduler.wrapJobExecutionInUserTransaction = false

# 实例化ThreadPool时，使用的线程类为SimpleThreadPool
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool

# threadCount和threadPriority将以setter的形式注入ThreadPool实例
# 并发个数
org.quartz.threadPool.threadCount = 5
# 优先级
org.quartz.threadPool.threadPriority = 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true

org.quartz.jobStore.misfireThreshold = 5000

# 默认存储在内存中
#org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore

#持久化
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX

#开启分布式部署
org.quartz.jobStore.isClustered = true

org.quartz.jobStore.tablePrefix = QRTZ_

org.quartz.jobStore.dataSource = qzDS

org.quartz.dataSource.qzDS.driver = com.mysql.jdbc.Driver

org.quartz.dataSource.qzDS.URL = jdbc:mysql://localhost:3306/quartz_test?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true

org.quartz.dataSource.qzDS.user = root

org.quartz.dataSource.qzDS.password = 123456

org.quartz.dataSource.qzDS.maxConnections = 10
```

**配置文件说明**

`org.quartz.jobStore.isClustered = true`

在集群中的每一个实例都必须有一个唯一的"instance  id" ("org.quartz.scheduler.instanceId" 属性), 默认为AUTO就可以。还要有相同的"scheduler  instance name"  ("org.quartz.scheduler.instanceName")，也就是说集群中的每一个实例都必须使用相同的quartz.properties  配置文件。

**调度任务代码**

```java
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.ResourceBundle;

public class SchedulerExecJob implements Job {

    private static Logger logger = LoggerFactory.getLogger(SchedulerExecJob.class);

    @Override
    public void execute(JobExecutionContext context)
            throws JobExecutionException {
        String jobName = context.getJobDetail().getKey().getName();
        switch (jobName) {
            /*每5s执行一次*/
            case "quartz_test1":
                System.err.println(getAddress()+" "+getDate()+"====>quartz_test1<====");
                break;
            /*每5s执行一次*/
            case "quartz_test2":
                System.err.println(getAddress()+" "+getDate()+"====>quartz_test2<====");
                break;
            /*每5s执行一次*/
            case "quartz_test3":
                System.err.println(getAddress()+" "+getDate()+"====>quartz_test3<====");
                break;
            default:
                System.err.println(getAddress()+" "+getDate()+"====>other task<====");
                break;
        }
    }

    public static String getDate(){
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
    }

    public static String getAddress(){
        return "http://localhost:"+ResourceBundle.getBundle("application").getString("server.port");
    }
}
```

项目中有三个任务，我用不同端口（三个服务同时启动，端口不一样,分别为：9099(ServerA)，9098(ServerB),9097(ServerC)）在本机启动同一个项目。当开启ServerA时，三个任务都在9099端口执行；开启ServerB后，有部分任务分配过来；开启ServerC，集群全部启动的情况下,会有任务交错执行或者任务在同一台机器上执行的效果

**注意事项**

Quartz实际并不关心你是在相同还是不同的机器上运行节点。当集群放置在不同的机器上时，称之为水平集群。节点跑在同一台机器上时，称之为垂直集群。对于垂直集群，存在着单点故障的问题。这对高可用性的应用来说是无法接受的，因为一旦机器崩溃了，所有的节点也就被终止了。对于水平集群，存在着时间同步问题。

节点用时间戳来通知其他实例它自己的最后检入时间。假如节点的时钟被设置为将来的时间，那么运行中的Scheduler将再也意识不到那个结点已经宕掉了。另一方面，如果某个节点的时钟被设置为过去的时间，也许另一节点就会认定那个节点已宕掉并试图接过它的Job重运行。最简单的同步计算机时钟的方式是使用某一个Internet时间服务器(Internet  Time Server ITS)。

**节点争抢Job问题**

因为Quartz使用了一个随机的负载均衡算法， Job以随机的方式由不同的实例执行。Quartz官网上提到当前，还不存在一个方法来指派(钉住) 一个 Job 到集群中特定的节点。

### 参考

1. [定时任务框架Quartz](https://blog.csdn.net/noaman_wgs/article/details/80984873)
2. [Spring Boot集成持久化Quartz定时任务管理和界面展示](https://blog.csdn.net/u012907049/article/details/73801122)