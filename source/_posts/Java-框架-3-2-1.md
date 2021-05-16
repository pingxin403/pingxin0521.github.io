---
title: 分布式任务调度框架--XXL-JOB
date: 2020-07-18 17:18:59
tags:
 - Java
 - 框架
 - 任务调度
 - 分布式
categories:
 - Java
 - 框架
---

XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

官方地址中文版：[http://www.xuxueli.com/xxl-job](http://www.xuxueli.com/xxl-job)

目前已有多家公司接入xxl-job，包括比较知名的大众点评，京东，优信二手车，北京尚德，360金融 (360)，联想集团 (联想)，易信 (网易)等等....

<!--more-->

Quartz作为开源作业调度中的佼佼者，是作业调度的首选。集群环境中Quartz采用API的方式对任务进行管理，

Quartz存在以下问题：

1. 问题一：调用API的的方式操作任务，不人性化；
2. 问题二：需要持久化业务QuartzJobBean到底层数据表中，系统侵入性相当严重。
3. 问题三：调度逻辑和QuartzJobBean耦合在同一个项目中，这将导致一个问题，在调度任务数量逐渐增多，同时调度任务逻辑逐渐加重的情况加，此时调度系统的性能将大大受限于业务；
4. 问题四：quartz底层以“抢占式”获取DB锁并由抢占成功节点负责运行任务，会导致节点负载悬殊非常大；而XXL-JOB通过执行器实现“协同分配式”运行任务，充分发挥集群优势，负载各节点均衡。

#### 快速入门

1. 下载项目源码

   ```bash
   git clone http://gitee.com/xuxueli0323/xxl-job.git
   ```

   按照maven格式将源码导入IDE, 使用maven进行编译即可，源码结构如下：

   ```
   xxl-job-admin：调度中心
   xxl-job-core：公共依赖
   xxl-job-executor-samples：执行器Sample示例（选择合适的版本执行器，可直接使用，也可以参考其并将现有项目改造成执行器）
       ：xxl-job-executor-sample-springboot：Springboot版本，通过Springboot管理执行器，推荐这种方式；
       ：xxl-job-executor-sample-spring：Spring版本，通过Spring容器管理执行器，比较通用；
       ：xxl-job-executor-sample-frameless：无框架版本；
       ：xxl-job-executor-sample-jfinal：JFinal版本，通过JFinal管理执行器；
       ：xxl-job-executor-sample-nutz：Nutz版本，通过Nutz管理执行器；
       ：xxl-job-executor-sample-jboot：jboot版本，通过jboot管理执行器；
   ```

2. 初始化“调度数据库”

   调度数据库初始化SQL脚本” 位置为:

   ```
   /xxl-job/doc/db/tables_xxl_job.sql
   ```

   调度中心支持集群部署，集群情况下各节点务必连接同一个mysql实例;

   如果mysql做主从,调度中心集群节点务必强制走主库;

3. 配置部署“调度中心”

   调度中心项目：xxl-job-admin
   作用：统一管理任务调度平台上调度任务，负责触发调度执行，并且提供任务管理平台。

   - 步骤一：调度中心配置：

     调度中心配置文件地址,修改mysql信息：

     ```
     /xxl-job/xxl-job-admin/src/main/resources/application.properties
     ```

   - 步骤二：部署项目：如果已经正确进行上述配置，可将项目编译打包部署。调度中心访问地址：http://localhost:8080/xxl-job-admin (该地址执行器将会使用到，作为回调地址)

     默认登录账号 “admin/123456”

   - 步骤三：调度中心集群（可选）：调度中心支持集群部署，提升调度系统容灾和可用性。

     调度中心集群部署时，几点要求和建议：

     - DB配置保持一致；
     - 集群机器时钟保持一致（单机集群忽视）；
     - 建议：推荐通过nginx为调度中心集群做负载均衡，分配域名。调度中心访问、执行器回调配置、调用API服务等操作均通过该域名进行。

   其他：Docker 镜像方式搭建调度中心：

   ```bash
   docker pull xuxueli/xxl-job-admin:{指定版本}
   docker run -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin  -d xuxueli/xxl-job-admin:{指定版本}
   
   /**
   * 如需自定义 mysql 等配置，可通过 "-e PARAMS" 指定，参数格式 PARAMS="--key=value  --key2=value2" ；
   * 配置项参考文件：/xxl-job/xxl-job-admin/src/main/resources/application.properties
   * 如需自定义 JVM内存参数 等配置，可通过 "-e JAVA_OPTS" 指定，参数格式 JAVA_OPTS="-Xmx512m" ；
   */
   docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin  -d xuxueli/xxl-job-admin:{指定版本}
   ```

4. 配置部署“执行器项目”

   “执行器”项目：xxl-job-executor-sample-springboot (提供多种版本执行器供选择，现以 springboot 版本为例，可直接使用，也可以参考其并将现有项目改造成执行器)

   作用：负责接收“调度中心”的调度并执行；可直接部署执行器，也可以将执行器集成到现有业务项目中。

   - 步骤一：maven依赖

     ```xml
     <!-- http://repo1.maven.org/maven2/com/xuxueli/xxl-job-core/ -->
     <dependency>
         <groupId>com.xuxueli</groupId>
         <artifactId>xxl-job-core</artifactId>
         <version>${最新稳定版本}</version>
     </dependency>
     ```

   - 步骤二：执行器配置

     执行器配置，配置文件地址：

     ```
     /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/resources/application.properties
     ```

   - 步骤三：执行器组件配置

     执行器组件，配置文件地址：

     ```
     /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/java/com/xxl/job/executor/core/config/XxlJobConfig.java
     ```

   - 步骤四：部署执行器项目

     如果已经正确进行上述配置，可将执行器项目编译打部署，系统提供多种执行器Sample示例项目，选择其中一个即可，各自的部署方式如下。

     ```
     xxl-job-executor-sample-springboot：项目编译打包成springboot类型的可执行JAR包，命令启动即可；
     xxl-job-executor-sample-spring：项目编译打包成WAR包，并部署到tomcat中。
     xxl-job-executor-sample-jfinal：同上
     xxl-job-executor-sample-nutz：同上
     xxl-job-executor-sample-jboot：同上
     ```

     至此“执行器”项目已经部署结束。

5. 步骤五：执行器集群（可选）

   执行器支持集群部署，提升调度系统可用性，同时提升任务处理能力。

   执行器集群部署时，几点要求和建议：

   - 执行器回调地址（xxl.job.admin.addresses）需要保持一致；执行器根据该配置进行执行器自动注册等操作。
   - 同一个执行器集群内AppName（xxl.job.executor.appname）需要保持一致；调度中心根据该配置动态发现不同集群的在线执行器列表。

#### 任务详解

##### 配置属性详细说明：

```
- 执行器：任务的绑定的执行器，任务触发调度时将会自动发现注册成功的执行器, 实现任务自动发现功能; 另一方面也可以方便的进行任务分组。每个任务必须绑定一个执行器, 可在 "执行器管理" 进行设置;
- 任务描述：任务的描述信息，便于任务管理；
- 路由策略：当执行器集群部署时，提供丰富的路由策略，包括；
    FIRST（第一个）：固定选择第一个机器；
    LAST（最后一个）：固定选择最后一个机器；
    ROUND（轮询）：；
    RANDOM（随机）：随机选择在线的机器；
    CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。
    LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；
    LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的机器优先被选举；
    FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；
    BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；
    SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有机器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；
- Cron：触发任务执行的Cron表达式；
- 运行模式：
    BEAN模式：任务以JobHandler方式维护在执行器端；需要结合 "JobHandler" 属性匹配执行器中任务；
    GLUE模式(Java)：任务以源码方式维护在调度中心；该模式的任务实际上是一段继承自IJobHandler的Java类代码并 "groovy" 源码方式维护，它在执行器项目中运行，可使用@Resource/@Autowire注入执行器里中的其他服务；
    GLUE模式(Shell)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "shell" 脚本；
    GLUE模式(Python)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "python" 脚本；
    GLUE模式(PHP)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "php" 脚本；
    GLUE模式(NodeJS)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "nodejs" 脚本；
    GLUE模式(PowerShell)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "PowerShell" 脚本；
- JobHandler：运行模式为 "BEAN模式" 时生效，对应执行器中新开发的JobHandler类“@JobHandler”注解自定义的value值；
- 阻塞处理策略：调度过于密集执行器来不及处理时的处理策略；
    单机串行（默认）：调度请求进入单机执行器后，调度请求进入FIFO队列并以串行方式运行；
    丢弃后续调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，本次请求将会被丢弃并标记为失败；
    覆盖之前调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，将会终止运行中的调度任务并清空队列，然后运行本地调度任务；
- 子任务：每个任务都拥有一个唯一的任务ID(任务ID可以从任务列表获取)，当本任务执行结束并且执行成功时，将会触发子任务ID所对应的任务的一次主动调度。
- 任务超时时间：支持自定义任务超时时间，任务运行超时将会主动中断任务；
- 失败重试次数；支持自定义任务失败重试次数，当任务失败时将会按照预设的失败重试次数主动进行重试；
- 报警邮件：任务调度失败时邮件通知的邮箱地址，支持配置多邮箱地址，配置多个邮箱地址时用逗号分隔；
- 负责人：任务的负责人；
- 执行参数：任务执行所需的参数；
```

1. BEAN模式（类形式）

   Bean模式任务，支持基于类的开发方式，每个任务对应一个Java类。

   - 优点：不限制项目环境，兼容性好。即使是无框架项目，如main方法直接启动的项目也可以提供支持，可以参考示例项目 “xxl-job-executor-sample-frameless”；
   - 缺点：
     - 每个任务需要占用一个Java类，造成类的浪费；
     - 不支持自动扫描任务并注入到执行器容器，需要手动注入。

2. BEAN模式（方法形式）

   Bean模式任务，支持基于方法的开发方式，每个任务对应一个方法。

   - 优点：
     - 每个任务只需要开发一个方法，并添加”[@XxlJob](https://github.com/XxlJob)”注解即可，更加方便、快速。
     - 支持自动扫描任务并注入到执行器容器。
   - 缺点：要求Spring容器环境；

   > 基于方法开发的任务，底层会生成JobHandler代理，和基于类的方式一样，任务也会以JobHandler的形式存在于执行器任务容器中。

3. GLUE模式(Java)

   任务以源码方式维护在调度中心，支持通过Web IDE在线更新，实时编译和生效，因此不需要指定JobHandler。

4. GLUE模式(Shell/Python/NodeJS/PHP/PowerShell)

#### 总体设计

**“调度数据库”配置**

XXL-JOB调度模块基于自研调度组件并支持集群部署，调度数据库表说明如下：

```
- xxl_job_lock：任务调度锁表；
- xxl_job_group：执行器信息表，维护任务执行器信息；
- xxl_job_info：调度扩展信息表： 用于保存XXL-JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等；
- xxl_job_log：调度日志表： 用于保存XXL-JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
- xxl_job_log_report：调度日志报表：用户存储XXL-JOB任务调度日志的报表，调度中心报表功能页面会用到；
- xxl_job_logglue：任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；
- xxl_job_registry：执行器注册表，维护在线的执行器和调度中心机器地址信息；
- xxl_job_user：系统用户表；
```

##### 架构设计

将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。

将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

**系统组成**

- 调度模块（调度中心）：
  负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；
  支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。
- 执行模块（执行器）：
  负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；
  接收“调度中心”的执行请求、终止请求和日志请求等。

![BIQQbD.png](https://s1.ax1x.com/2020/11/07/BIQQbD.png)

**过期处理策略**

任务调度错过触发时间时的处理策略：

- 可能原因：服务重启；调度线程被阻塞，线程被耗尽；上次调度持续阻塞，下次调度被错过；
- 处理策略：
  - 过期超5s：本次忽略，当前时间开始计算下次触发时间
  - 过期5s内：立即触发一次，当前时间开始计算下次触发时间

**日志回调服务**

调度模块的“调度中心”作为Web服务部署时，一方面承担调度中心功能，另一方面也为执行器提供API服务。

调度中心提供的”日志回调服务API服务”代码位置如下：

```
xxl-job-admin#com.xxl.job.admin.controller.JobApiController.callback
```

“执行器”在接收到任务执行请求后，执行任务，在执行结束之后会将执行结果回调通知“调度中心

**调度日志**

调度中心每次进行任务调度，都会记录一条任务日志，任务日志主要包括以下三部分内容：

- 任务信息：包括“执行器地址”、“JobHandler”和“执行参数”等属性，点击任务ID按钮可查看，根据这些参数，可以精确的定位任务执行的具体机器和任务代码；
- 调度信息：包括“调度时间”、“调度结果”和“调度日志”等，根据这些参数，可以了解“调度中心”发起调度请求时具体情况。
- 执行信息：包括“执行时间”、“执行结果”和“执行日志”等，根据这些参数，可以了解在“执行器”端任务执行的具体情况；

调度日志，针对单次调度，属性说明如下：

- 执行器地址：任务执行的机器地址；
- JobHandler：Bean模式表示任务执行的JobHandler名称；
- 任务参数：任务执行的入参；
- 调度时间：调度中心，发起调度的时间；
- 调度结果：调度中心，发起调度的结果，SUCCESS或FAIL；
- 调度备注：调度中心，发起调度的备注信息，如地址心跳检测日志等；
- 执行时间：执行器，任务执行结束后回调的时间；
- 执行结果：执行器，任务执行的结果，SUCCESS或FAIL；
- 执行备注：执行器，任务执行的备注信息，如异常日志等；
- 执行日志：任务执行过程中，业务代码中打印的完整执行日志

