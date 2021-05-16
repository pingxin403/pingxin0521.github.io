---
title: 微服务追踪 组件之SkyWalking  (八)
date: 2020-5-27 12:18:59
tags:
 - Java 
 - 框架
 - Spring Cloud
categories:
 - Java
 - Spring Cloud
---

随着微服务架构的流行，一些微服务架构下的问题也会越来越突出，比如一个请求会涉及多个服务，而服务本身可能也会依赖其他服务，整个请求路径就构成了一个网状的调用链，而在整个调用链中一旦某个节点发生异常，整个调用链的稳定性就会受到影响，所以会深深的感受到 “银弹” 这个词是不存在的，每种架构都有其优缺点 。

![texIQU.png](https://s1.ax1x.com/2020/05/28/texIQU.png)

<!--more-->

面对以上情况， 我们就需要一些可以帮助理解系统行为、用于分析性能问题的工具，以便发生故障的时候，能够快速定位和解决问题，这时候 APM（应用性能管理）工具就该闪亮登场了。

目前主要的一些 APM 工具有: Cat、Zipkin、Pinpoint、SkyWalking，这里主要介绍 [SkyWalking](https://github.com/apache/skywalking) ，它是一款优秀的国产 APM 工具，包括了分布式追踪、性能指标分析、应用和服务依赖分析等。

### [Skywalking](https://skywalking.apache.org/)

下面是 SkyWalking 6.x 的架构图：

![tm9iTJ.png](https://s1.ax1x.com/2020/05/28/tm9iTJ.png)

> **说明：** SkyWalking 的核心是数据分析和度量结果的存储平台，通过 HTTP 或 gRPC 方式向 SkyWalking Collecter 提交分析和度量数据，SkyWalking Collecter 对数据进行分析和聚合，存储到 Elasticsearch、H2、MySQL、TiDB 等其一即可，最后我们可以通过 SkyWalking UI 的可视化界面对最终的结果进行查看。Skywalking 支持从多个来源和多种格式收集数据：多种语言的 Skywalking Agent 、Zipkin v1/v2 、Istio 勘测、Envoy 度量等数据格式。
>
> 整体架构看似模块有点多，但在实际上还是比较清晰的，主要就是通过收集各种格式的数据进行存储，然后展示。所以搭建 Skywalking 服务我们需要关注的是 SkyWalking Collecter、SkyWalking UI 和 存储设备，SkyWalking Collecter、SkyWalking UI 官方下载安装包内已包含，最终我们只需考虑存储设备即可。

整体架构包含如下三个组成部分:
1. 探针(agent)负责进行数据的收集,包含了Tracing和Metrics的数据,agent会被安装到服务所在的服务器上,以方便数据的获取。
2. 可观测性分析平台OAP(Observability Analysis Platform),接收探针发送的数据,并在内存中使用分析引擎(Analysis Core)进行数据的整合运算,然后将数据存储到对应的存储介质上,比如Elasticsearch、MySQL数据库、H2数据库等。同时OAP还使用查询引擎(Query Core)提供HTTP查询接口。
3. Skywalking提供单独的UI进行数据的查看,此时UI会调用OAP提供的接口,获取对应的数据然后进行展示。

#### Skywalking主要概念

使用如下案例来进行Skywalking主要概念的介绍,Skywalking主要概念包含:

- 服务 (Service)
- 端点 (Endpoint)
- 实例 (Instance)

#### 环境搭建

> 具体的安装步骤可以在Skywalking的官方github上找到:
> https://github.com/apache/skywalking/blob/master/docs/en/setup/README.md

参考资料：

链接: https://pan.baidu.com/s/11ls6BJFP7AZGu_52LvxRmw 提取码: iqsp

##### 直接安装

> apache-skywalking-apm-6.5.0.tar.gz ---Skywalking最新的安装包

默认使用H2存储数据

```bash
#切换到skywalking目录
cd /usr/local/skywalking
#解压压缩包
tar -zxvf apache-skywalking-apm-6.4.0.tar.gz
./bin/startup.sh
```

执行startup.bat之后会启动如下两个服务：

- Skywalking-Collector：追踪信息收集器，通过 gRPC/Http 收集客户端的采集信息 ，Http默认端口 12800，gRPC默认端口 11800。
- Skywalking-Webapp：管理平台页面 默认端口 8080，登录信息 admin/admin

##### 配置使用ES存储

1. 创建目录

   ```
   mkdir /usr/local/skywalking
   ```

2. 将资源目录中的elasticsearch和skywalking安装包上传到虚拟机/usr/local/skywalking目录下。

   > elasticsearch-6.4.0.tar.gz ---elasticsearch 6.4的安装包,Skywalking对es版本号有一定要求,最
   > 好使用6.3.2以上版本,如果是7.x版本需要额外进行配置。
   > apache-skywalking-apm-6.5.0.tar.gz ---Skywalking最新的安装包

3. 首先安装elasticsearch,将压缩包解压。

   ```
   tar -zxvf ./elasticsearch-6.4.0.tar.gz
   ```

   修改 Linux系统的限制配置,将文件创建数修改为65536个。
   1. 修改系统中允许应用最多创建多少文件等的限制权限。Linux默认来说,一般限制应用最多
   创建的文件是65535个。但是ES至少需要65536的文件创建数的权限。
   2. 修改系统中允许用户启动的进程开启多少个线程。默认的Linux限制root用户开启的进程可
   以开启任意数量的线程,其他用户开启的进程可以开启1024个线程。必须修改限制数为
   4096+。因为ES至少需要4096的线程池预备。

   ```
   vi /etc/security/limits.conf
   #新增如下内容在limits.conf文件中
   es soft nofile 65536
   es hard nofile 65536
   es soft nproc 4096
   es hard nproc 4096
   ```

   修改系统控制权限, ElasticSearch需要开辟一个65536字节以上空间的虚拟内存。Linux默认不允许任何用户和应用程序直接开辟这么大的虚拟内存。

   ```
   vi /etc/sysctl.conf
   #新增如下内容在sysctl.conf文件中,当前用户拥有的内存权限大小
   vm.max_map_count=262144
   #让系统控制权限配置生效
   sysctl -p
   ```

   建一个用户, 用于ElasticSearch启动。
   ES在5.x版本之后,强制要求在linux中不能使用root用户启动ES进程。所以必须使用其他用户启动ES进程才可以。

   ```bash
   #创建用户
   useradd
   es
   #修改上述用户的密码
   passwd es
   #修改elasicsearch目录的拥有者
   chown -R
   es elasticsearch-6.4.0
   ```

   使用 es用户启动elasticsearch

   ```bash
   #切换用户
   su es
   #到ElasticSearch的bin目录下
   cd bin/
   #后台启动
   ./elasticsearch -d
   ```

   默认 ElasticSearch是不支持跨域访问的,所以在不修改配置文件的情况下我们只能从虚拟机内部进行访问测试ElasticSearch是否安装成功,使用curl命令访问9200端口:

   ```
   curl http://localhost:9200
   ```

4. 安装Skywalking,分为两个步骤:

   - 安装 Backend后端服务
   - 安装 UI

   首先切回到root用户,切换到目录下,解压Skywalking压缩包。

   ```bash
   #切换到root用户
   su root
   #切换到skywalking目录
   cd /usr/local/skywalking
   #解压压缩包
   tar -zxvf apache-skywalking-apm-6.4.0.tar.gz
   ```

   修改 Skywalking存储的数据源配置:

   ```
   cd apache-skywalking-apm-bin
   vi config/application.yml
   ```

   我们可以看到默认配置中,使用了 H2作为数据源。我们将其全部注释。

   ```yml
   #h2:
   # driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
   # url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
   # user: ${SW_STORAGE_H2_USER:sa}
   #metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
   #mysql:
   #metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
   ```

   将 ElasticSearch对应的配置取消注释

   ```yml
   storage:elasticsearch:
   nameSpace: ${SW_NAMESPACE:""}
   clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
   protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
   trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
   trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
   user: ${SW_ES_USER:""}
   password: ${SW_ES_PASSWORD:""}
   indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
   indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
   # Those data TTL settings will override the same settings in core module.
   recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
   otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
   monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is
   month
   #
   # Batch process setting, refer to
   https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-
   bulk-processor.html
   bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every
   1000 requests
   flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10
   seconds whatever the number of requests
   concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of
   concurrent requests
   metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
   segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
   ```

   默认使用了 localhost下的ES,所以我们可以不做任何处理,直接进行使用。启动OAP程序:

   ```
   bin/oapService.sh
   ```

   这样安装 Backend后端服务就已经完毕了,接下来我们安装UI。先来看一下UI的配置文件:

   ```bash
   cat webapp/webapp.yml
   
   #默认启动端口
   server:
   port: 8080
   collector:
   path: /graphql
   ribbon:
   ReadTimeout: 10000
   #OAP服务,如果有多个用逗号隔开
   listOfServers: 127.0.0.1:12800
   ```

   目前的默认配置不用修改就可以使用,启动 UI程序:

   ```
   ./bin/webappService.sh
   ```

   然后我们就可以通过浏览器访问 Skywalking的可视化页面了,访问地址:<http://IP地址:8080>,如果
   出现下面的图,就代表安装成功了。

   > ./bin/startup.sh可以同时启动backend和ui,后续可以执行该文件进行重启。

##### 配置信息

1. 收集器相关配置：支持 http/gRPC收集

   ```yml
   core:
     default:
       restHost: ${SW_CORE_REST_HOST:0.0.0.0}
       restPort: ${SW_CORE_REST_PORT:12800}
       restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
       gRPCHost: ${SW_CORE_GRPC_HOST:0.0.0.0}
       gRPCPort: ${SW_CORE_GRPC_PORT:11800}
       downsampling:
       - Hour
       - Day
       - Month
       # Set a timeout on metric data. After the timeout has expired, the metric data will automatically be deleted.
       recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:90} # Unit is minute
       minuteMetricsDataTTL: ${SW_CORE_MINUTE_METRIC_DATA_TTL:90} # Unit is minute
       hourMetricsDataTTL: ${SW_CORE_HOUR_METRIC_DATA_TTL:36} # Unit is hour
       dayMetricsDataTTL: ${SW_CORE_DAY_METRIC_DATA_TTL:45} # Unit is day
       monthMetricsDataTTL: ${SW_CORE_MONTH_METRIC_DATA_TTL:18} # Unit is month
   
   ```

2. 收集信息存储：支持h2、Mysql和 ES

   ```yml
   storage:
     h2:
       driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
       url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
       user: ${SW_STORAGE_H2_USER:sa}
   #  elasticsearch:
   #    # nameSpace: ${SW_NAMESPACE:""}
   #    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
   #    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
   #    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
   #    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
   #    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
   #    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
   #    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
   #    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
   
   ```

3. 可采集信息：jvm运行相关信息，zipkin追踪日志等。

   ```yml
   receiver-register:
     default:
   receiver-trace:
     default:
       bufferPath: ${SW_RECEIVER_BUFFER_PATH:../trace-buffer/}  # Path to trace buffer files, suggest to use absolute path
       bufferOffsetMaxFileSize: ${SW_RECEIVER_BUFFER_OFFSET_MAX_FILE_SIZE:100} # Unit is MB
       bufferDataMaxFileSize: ${SW_RECEIVER_BUFFER_DATA_MAX_FILE_SIZE:500} # Unit is MB
       bufferFileCleanWhenRestart: ${SW_RECEIVER_BUFFER_FILE_CLEAN_WHEN_RESTART:false}
       sampleRate: ${SW_TRACE_SAMPLE_RATE:10000} # The sample rate precision is 1/10000. 10000 means 100% sample in default.
   receiver-jvm:
     default:
   #service-mesh:
   #  default:
   #    bufferPath: ${SW_SERVICE_MESH_BUFFER_PATH:../mesh-buffer/}  # Path to trace buffer files, suggest to use absolute path
   #    bufferOffsetMaxFileSize: ${SW_SERVICE_MESH_OFFSET_MAX_FILE_SIZE:100} # Unit is MB
   #    bufferDataMaxFileSize: ${SW_SERVICE_MESH_BUFFER_DATA_MAX_FILE_SIZE:500} # Unit is MB
   #    bufferFileCleanWhenRestart: ${SW_SERVICE_MESH_BUFFER_FILE_CLEAN_WHEN_RESTART:false}
   #istio-telemetry:
   #  default:
   #receiver_zipkin:
   #  default:
   #    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
   #    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
   #    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
   
   ```

##### RocketBot

1. 仪表盘

   打开RocketBot默认会出现仪表盘页面:

   ![tmEe5F.png](https://s1.ax1x.com/2020/05/28/tmEe5F.png)

   仪表盘页面分为两大块:

   - 服务仪表盘,展示服务的调用情况
   - 数据库仪表盘,展示数据库的响应时间等数据

   选中服务仪表盘,有四个维度的统计数据可以进行查看:

   - 全局,查看全局接口的调用,包括全局响应时长的百分比,最慢的端点,服务的吞吐量等
   - 服务,显示服务的响应时长、 SLA、吞吐量等信息
   - 端点,显示端点的响应时长、 SLA、吞吐量等信息
   - 实例,显示实例的响应时长、 SLA、吞吐量等信息,还可以查看实例的JVM的GC信息、CPU信息、
     内存信息

2. 拓扑图

   Skywalking提供拓扑图,直观的查看服务之间的调用关系:

   ![tmEJUO.png](https://s1.ax1x.com/2020/05/28/tmEJUO.png)

   User 代表用户应用,目前案例中其实是浏览器
   图中Skywalking_boot应用被User调用,同时显示它是一个Spring MVC的应用。后续案例中会出现多个应用调用,使用拓扑图就能清楚的分析其调用关系了。

3. 追踪

   在Skywalking中,每一次用户发起一条请求,就可以视为一条追踪数据,每条追踪数据携带有一个ID值。追踪数据在追踪页面中可以进行查询:

   ![tmEBrt.png](https://s1.ax1x.com/2020/05/28/tmEBrt.png)

   左侧是追踪列表,也可以通过上方的追踪 ID来进行查询。点击追踪列表某一条记录之后,右侧会显示出
   此条追踪的详细信息。有三种显示效果:

   - 列表
   - 树结构
   - 表格

   可以很好的展现此条追踪的调用链情况而链路上每个节点,可以通过左键点击节点查看详细信息:

   ![tmE2GQ.png](https://s1.ax1x.com/2020/05/28/tmE2GQ.png)

   当前的接口是 HTTP的GET请求,相对比较简单,后续的示例中出现异常情况或者数据库访问,可以打印
   出异常信息、堆栈甚至详细的SQL语句。

4. 告警
   Skywalking中的告警功能相对比较简单,在达到告警阈值之后会生成一条告警记录,在告警页面上进行展示。

#### 基础

agent探针可以让我们不修改代码的情况下,对java应用上使用到的组件进行动态监控,获取运行数据发送到OAP上进行统计和存储。agent探针在java中是使用java agent技术实现的,不需要更改任何代码,java agent会通过虚拟机(VM)接口来在运行期更改代码。

Agent探针支持 JDK 1.6 - 12的版本,Agent探针所有的文件在Skywalking的agent文件夹下。文件目录如下;

```
+-- agent
    +-- activations
        apm-toolkit-log4j-1.x-activation.jar
        apm-toolkit-log4j-2.x-activation.jar
        apm-toolkit-logback-1.x-activation.jar
        ...
    //配置文件
    +-- config
   		agent.config
    //组件的所有插件
    +-- plugins
        apm-dubbo-plugin.jar
        apm-feign-default-http-9.x.jar
        apm-httpClient-4.x-plugin.jar
        .....
    //可选插件
    +-- optional-plugins
        apm-gson-2.x-plugin.jar
        .....
    +-- bootstrap-plugins
        jdk-http-plugin.jar
        .....
    +-- logs
    skywalking-agent.jar
```

部分插件在使用上会影响整体的性能或者由于版权问题放置于可选插件包中,不会直接加载,如果需要使用,将可选插件中的jar包拷贝到plugins包下。

由于没有修改 agent探针中的应用名,所以默认显示的是Your_ApplicationName。我们修改下应用名称,让他显示的更加正确。编辑agent配置文件:

```
cd /usr/local/skywalking/apache-skywalking-apm-bin/agent/config
vi agent.config
```

我们在配置中找到这么一行:
```
# The service name in UI

agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
```

这里的配置含义是可以读取到 SW_AGENT_NAME配置属性,如果该配置没有指定,那么默认名称Your_ApplicationName。

1. tomcat：更改catalina.sh，在文件顶部添加:

   ```
   CATALINA_OPTS="$CATALINA_OPTS -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar"; export CATALINA_OPTS
   ```

2. Spring Boot：使用命令启动 spring boot项目:

   ```
   java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent_boot/skywalking-agent.jar -Dserver.port=8082 -jar skywalking_springboot.jar &
   ```

   访问 <http://IP:8082/sayBoot>

#### 常用插件

##### 配置覆盖

Skywalking支持的几种配置方式:

- 系统配置(System properties)

  使用 `skywalking.+` 配置文件中的配置名作为系统配置项来进行覆盖

  ```
  -Dskywalking.agent.service_name=skywalking_mysql
  ```

- 探针配置( Agent options)

  ```
  -javaagent:/path/to/skywalking-agent.jar=[option1]=[value1],[option2]=[value2]
  #举例
  -javaagent:/path/to/skywalking-agent.jar=agent.service_name=skywalking_mysql
  -javaagent:/path/to/skywalking-agent.jar=agent.ignore_suffix='.jpg,.jpeg'
  ```

- 系统环境变量( System environment variables)

  由于agent.service_name配置项如下所示:

  ```
  # The service name in UI
  agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
  ```

  可以在环境变量中设置 SW_AGENT_NAME的值来指定服务名。

覆盖优先级：探针配置 > 系统配置 >系统环境变量 > 配置文件中的值

##### 过滤指定的端点

在开发过程中,有一些端点(接口)并不需要去进行监控,比如Swagger相关的端点。这个时候我们就可以使用Skywalking提供的过滤插件来进行过滤。在skywalking_plugins中编写两个接口进行测试:

![tmVFRH.png](https://s1.ax1x.com/2020/05/29/tmVFRH.png)

将agent中的 /agent/optional -plugins/apm-trace-ignore-plugin-6.4.0.jar 插件拷贝到plugins目录中。

启动skywalking_plugins应用,等待启动成功：

```bash
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar -Dskywalking.agent.service_name=skywalking_plugins -Dskywalking.trace.ignore_path=/exclude jar skywalking_plugins.jar &
```

> 这里添加 `-Dskywalking.trace.ignore_path=/exclude`参数来标识需要过滤哪些请求,支持 `AntPath` 表达式:
> `/path/*` , `/path/**` , `/path/?`
>
> - ? 匹配任何单字符
>
> * `*`匹配0或者任意数量的字符
> * `**` 匹配0或者更多的目录

#### 告警功能

Skywalking每隔一段时间根据收集到的链路追踪的数据和配置的告警规则(如服务响应时间、服务响应时间百分比)等,判断如果达到阈值则发送相应的告警信息。发送告警信息是通过调用webhook接口完成,具体的webhook接口可以使用者自行定义,从而开发者可以在指定的webhook接口中编写各种告警方式,比如邮件、短信等。告警的信息也可以在RocketBot中查看到。

以下是默认的告警规则配置,位于skywalking安装目录下的config文件夹下 alarm -settings.yml 文件中:

```yml
# Sample alarm rules.
rules:
  # Rule unique name, must be ended with `_rule`.
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 3
    silence-period: 5
    message: Response time of service {name} is more than 1000ms in 3 minutes of last 10 minutes.
  service_sla_rule:
    # Metrics value need to be long, double or int
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    # The length of time to evaluate the metrics
    period: 10
    # How many times after the metrics match the condition, will trigger alarm
    count: 2
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 3
    message: Successful rate of service {name} is lower than 80% in 2 minutes of last 10 minutes
  service_p90_sla_rule:
    # Metrics value need to be long, double or int
    metrics-name: service_p90
    op: ">"
    threshold: 1000
    period: 10
    count: 3
    silence-period: 5
    message: 90% response time of service {name} is more than 1000ms in 3 minutes of last 10 minutes
  service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 5
    message: Response time of service instance {name} is more than 1000ms in 2 minutes of last 10 minutes
#  Active endpoint related metrics alarm will cost more memory than service and service instance metrics alarm.
#  Because the number of endpoint is much more than service and instance.
#
#  endpoint_avg_rule:
#    metrics-name: endpoint_avg
#    op: ">"
#    threshold: 1000
#    period: 10
#    count: 2
#    silence-period: 5
#    message: Response time of endpoint {name} is more than 1000ms in 2 minutes of last 10 minutes

webhooks:
#  - http://127.0.0.1/notify/
#  - http://127.0.0.1/go-wechat/

```

以上文件定义了默认的 4种规则:
1. 最近3分钟内服务的平均响应时间超过1秒
2. 最近2分钟服务成功率低于80%
3. 最近3分钟90%服务响应时间超过1秒
4. 最近2分钟内服务实例的平均响应时间超过1秒

规则中的参数属性如下:

![tmVcex.png](https://s1.ax1x.com/2020/05/29/tmVcex.png)

webhooks可以配置告警产生时的调用地址。

修改告警规则配置文件,将webhook地址修改为:

```
webhooks:
  - http://127.0.0.1:8089/webhook
```

启动skywalking_alarm应用,等待启动成功。

```
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin/agent/skywalking-agent.jar -Dskywalking.agent.service_name=skywalking_alarm -jar skywalking_alarm.jar
```

不停调用接口,接口地址为: <http://IP:8089/timeout>

查看告警信息接口: <http://IP:8089/show>

### 参考

1. [OpenTracing:开放式分布式追踪规范](https://www.jianshu.com/p/d2b11c079af0)