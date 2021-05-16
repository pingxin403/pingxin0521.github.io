---
title: Spring boot 健康检查、审计、统计和监控
date: 2019-06-30 17:37:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

### Spring Boot Actuator

Spring Boot Actuator可以帮助你监控和管理Spring Boot应用，比如健康检查、审计、统计和HTTP追踪等。所有的这些特性可以通过JMX或者HTTP endpoints来获得。

<!--more-->

Actuator同时还可以与外部应用监控系统整合，比如 [Prometheus](https://prometheus.io/), [Graphite](https://graphiteapp.org/), [DataDog](https://www.datadoghq.com/), [Influx](https://www.influxdata.com/), [Wavefront](https://www.wavefront.com/), [New Relic](https://newrelic.com/)等。这些系统提供了非常好的仪表盘、图标、分析和告警等功能，使得你可以通过统一的接口轻松的监控和管理你的应用。

Actuator使用[Micrometer](http://micrometer.io/)来整合上面提到的外部应用监控系统。这使得只要通过非常小的配置就可以集成任何应用监控系统。

为了保证 actuator 暴露的监控接口的安全性，需要添加安全控制的依赖`spring-boot-start-security`依赖，访问应用监控端点时，都需要输入验证信息。Security 依赖，可以选择不加，不进行安全管理，但不建议这么做。

**依赖**

```xml
 <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

#### Actuator 的 REST 接口

Actuator 监控分成两类：原生端点和用户自定义端点；自定义端点主要是指扩展性，用户可以根据自己的实际应用，定义一些比较关心的指标，在运行期进行监控。

原生端点是在应用程序里提供众多 Web 接口，通过它们了解应用程序运行时的内部状况。原生端点又可以分成三类：

- 应用配置类：可以查看应用在运行期的静态信息：例如自动配置信息、加载的 springbean 信息、yml 文件配置信息、环境信息、请求映射信息；
- 度量指标类：主要是运行期的动态信息，例如堆栈、请求连、一些健康指标、metrics 信息等；
- 操作控制类：主要是指 shutdown,用户可以发送一个请求将应用的监控功能关闭。

Actuator 提供了 13 个接口，具体如下表所示。

| HTTP 方法 | 路径            | 描述                                                         |
| :-------- | :-------------- | :----------------------------------------------------------- |
| GET       | /auditevents    | 显示应用暴露的审计事件 (比如认证进入、订单失败)              |
| GET       | /beans          | 描述应用程序上下文里全部的 Bean，以及它们的关系              |
| GET       | /conditions     | 就是 1.0 的 /autoconfig ，提供一份自动配置生效的条件情况，记录哪些自动配置条件通过了，哪些没通过 |
| GET       | /configprops    | 描述配置属性(包含默认值)如何注入Bean                         |
| GET       | /env            | 获取全部环境属性                                             |
| GET       | /env/{name}     | 根据名称获取特定的环境属性值                                 |
| GET       | /flyway         | 提供一份 Flyway 数据库迁移信息                               |
| GET       | /liquidbase     | 显示Liquibase 数据库迁移的纤细信息                           |
| GET       | /health         | 报告应用程序的健康指标，这些值由 HealthIndicator 的实现类提供 |
| GET       | /heapdump       | dump 一份应用的 JVM 堆信息                                   |
| GET       | /httptrace      | 显示HTTP足迹，最近100个HTTP request/repsponse                |
| GET       | /info           | 获取应用程序的定制信息，这些信息由info打头的属性提供         |
| GET       | /logfile        | 返回log file中的内容(如果 logging.file 或者 logging.path 被设置) |
| GET       | /loggers        | 显示和修改配置的loggers                                      |
| GET       | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数         |
| GET       | /metrics/{name} | 报告指定名称的应用程序度量值                                 |
| GET       | /scheduledtasks | 展示应用中的定时任务信息                                     |
| GET       | /sessions       | 如果我们使用了 Spring Session 展示应用中的 HTTP sessions 信息 |
| POST      | /shutdown       | 关闭应用程序，要求endpoints.shutdown.enabled设置为true       |
| GET       | /mappings       | 描述全部的 URI路径，以及它们和控制器(包含Actuator端点)的映射关系 |
| GET       | /threaddump     | 获取线程活动的快照                                           |

#### 使用Actuator Endpoints来监控应用

Actuator创建了所谓的**endpoint**来暴露HTTP或者JMX来监控和管理应用。

举个例子，有一个叫`/health`的endpoint，提供了关于应用健康的基础信息。`/metrics`endpoints展示了几个有用的度量信息，比如JVM内存使用情况、系统CPU使用情况、打开的文件等等。`/loggers`endpoint展示了应用的日志和可以让你在运行时改变日志等级。

**值得注意的是，每一给actuator endpoint可以被显式的打开和关闭。此外，这些endpoints也需要通过HTTP或者JMX暴露出来，使得它们能被远程进入。**

让我们运行应用并且尝试进入默认通过HTTP暴露的打开状态的actuator endpoints。之后，我们将学习如何打开更多的endpoints并且通过HTTP暴露它们。

在应用的根目录下打开命令行工具运行以下命令：

```
mvn spring-boot:run
```

应用默认使用`8080`端口运行。一旦这个应用启动了，你可以通过http://localhost:8080/actuator来展示所有通过HTTP暴露的endpoints。

```json
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"info":{"href":"http://localhost:8080/actuator/info","templated":false}}}
```

打开http://localhost:8080/actuator/health，则会显示如下内容:

```json
{"status":"UP"}
```

状态将是`UP`只要应用是健康的。如果应用不健康将会显示`DOWN`,比如与仪表盘的连接异常或者缺水磁盘空间等。

`info`endpoint(http://localhost:8080/actuator/info)展示了关于应用的一般信息，这些信息从编译文件比如`META-INF/build-info.properties`或者Git文件比如`git.properties`或者任何环境的property中获取。

**默认，只有`health`和`info`通过HTTP暴露了出来**。这也是为什么`/actuator`页面只展示了`health`和`info`endpoints。我们将学习如何暴露其他的endpoint。首先，让我们看看其他的endpoints是什么。

以下是一些非常有用的actuator endpoints列表。你可以在[official documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)上面看到完整的列表。

| Endpoint ID    | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| auditevents    | 显示应用暴露的审计事件 (比如认证进入、订单失败)              |
| info           | 显示应用的基本信息                                           |
| health         | 显示应用的健康状态                                           |
| metrics        | 显示应用多样的度量信息                                       |
| loggers        | 显示和修改配置的loggers                                      |
| logfile        | 返回log file中的内容(如果logging.file或者logging.path被设置) |
| httptrace      | 显示HTTP足迹，最近100个HTTP request/repsponse                |
| env            | 显示当前的环境特性                                           |
| flyway         | 显示数据库迁移路径的详细信息                                 |
| liquidbase     | 显示Liquibase 数据库迁移的纤细信息                           |
| shutdown       | 让你逐步关闭应用                                             |
| mappings       | 显示所有的@RequestMapping路径                                |
| scheduledtasks | 显示应用中的调度任务                                         |
| threaddump     | 执行一个线程dump                                             |
| heapdump       | 返回一个GZip压缩的JVM堆dump                                  |

#### 打开和关闭Actuator Endpoints

默认，上述所有的endpints都是打开的，除了`shutdown` endpoint。

你可以通过设置`management.endpoint..enabled to true or false`(`id`是endpoint的id)来决定打开还是关闭一个actuator endpoint。

举个例子，要想打开`shutdown` endpoint，增加以下内容在你的`application.properties`文件中：

```xml
management.endpoint.shutdown.enabled=true
```

#### 暴露Actuator Endpoints

默认，所有的actuator endpoint通过JMX被暴露，而通过HTTP暴露的只有`health`和`info`。

以下是你可以通过应用的properties可以通过HTTP和JMX暴露的actuator endpoint。

- 通过HTTP暴露Actuator endpoints。

  ```properties
  # Use "*" to expose all endpoints, or a comma-separated list to expose selected ones
  management.endpoints.web.exposure.include=health,info 
  management.endpoints.web.exposure.exclude=
  #启用接口关闭 Spring Boot
  management.endpoint.shutdown.enabled=true
  ```

- 通过JMX暴露Actuator endpoints。

  ```properties
  # Use "*" to expose all endpoints, or a comma-separated list to expose selected ones
  management.endpoints.jmx.exposure.include=*
  management.endpoints.jmx.exposure.exclude=
  ```

- 通过设置`management.endpoints.web.exposure.include`为`*`，我们可以在http://localhost:8080/actuator页面看到如下内容。

```
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"auditevents":{"href":"http://localhost:8080/actuator/auditevents","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"conditions":{"href":"http://localhost:8080/actuator/conditions","templated":false},"configprops":{"href":"http://localhost:8080/actuator/configprops","templated":false},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"info":{"href":"http://localhost:8080/actuator/info","templated":false},"loggers":{"href":"http://localhost:8080/actuator/loggers","templated":false},"loggers-name":{"href":"http://localhost:8080/actuator/loggers/{name}","templated":true},"heapdump":{"href":"http://localhost:8080/actuator/heapdump","templated":false},"threaddump":{"href":"http://localhost:8080/actuator/threaddump","templated":false},"prometheus":{"href":"http://localhost:8080/actuator/prometheus","templated":false},"metrics-requiredMetricName":{"href":"http://localhost:8080/actuator/metrics/{requiredMetricName}","templated":true},"metrics":{"href":"http://localhost:8080/actuator/metrics","templated":false},"scheduledtasks":{"href":"http://localhost:8080/actuator/scheduledtasks","templated":false},"httptrace":{"href":"http://localhost:8080/actuator/httptrace","templated":false},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}
```

#### 解析常用的actuator endpoint

**/health endpoint**

`health` endpoint通过合并几个健康指数检查应用的健康情况。

Spring Boot Actuator有几个预定义的健康指标比如[`DataSourceHealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.html), [`DiskSpaceHealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.html), [`MongoHealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/mongo/MongoHealthIndicator.html), [`RedisHealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/redis/RedisHealthIndicator.html), [`CassandraHealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.html)等。它使用这些健康指标作为健康检查的一部分。

举个例子，如果你的应用使用`Redis`，`RedisHealthindicator`将被当作检查的一部分。如果使用`MongoDB`，那么`MongoHealthIndicator`将被当作检查的一部分。

你也可以关闭特定的健康检查指标，比如在prpperties中使用如下命令：

```xml
management.health.mongo.enabled=false
```

默认，所有的这些健康指标被当作健康检查的一部分。

1. 显示详细的健康信息

   `health` endpoint只展示了简单的`UP`和`DOWN`状态。为了获得健康检查中所有指标的详细信息，你可以通过在`application.yaml`中增加如下内容：

   ```xml
   management:
     endpoint:
       health:
         show-details: always
   ```

   一旦你打开上述开关，你在`/health`中可以看到如下详细内容：

   ```json
   {"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":250790436864,"free":27172782080,"threshold":10485760}}}}
   ```

   `health` endpoint现在包含了`DiskSpaceHealthIndicator`。

   如果你的应用包含database(比如MySQL)，`health` endpoint将显示如下内容：

   ```json
   {
      "status":"UP",
      "details":{
         "db":{
            "status":"UP",
            "details":{
               "database":"MySQL",
               "hello":1
            }
         },
         "diskSpace":{
            "status":"UP",
            "details":{
               "total":250790436864,
               "free":100330897408,
               "threshold":10485760
            }
         }
      }
   }
   ```

   如果你的MySQL server没有启起来，状态将会变成`DOWN`：

   ```json
   {
      "status":"DOWN",
      "details":{
         "db":{
            "status":"DOWN",
            "details":{
               "error":"org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30006ms."
            }
         },
         "diskSpace":{
            "status":"UP",
            "details":{
               "total":250790436864,
               "free":100324585472,
               "threshold":10485760
            }
         }
      }
   }
   ```

2. 创建一个自定义的健康指标

   你可以通过实现`HealthIndicator`接口来自定义一个健康指标，或者继承`AbstractHealthIndicator`类。

   ```java
   import org.springframework.boot.actuate.health.AbstractHealthIndicator;
   import org.springframework.boot.actuate.health.Health;
   import org.springframework.stereotype.Component;
   
   @Component
   public class CustomHealthIndicator extends AbstractHealthIndicator {
   
       @Override
       protected void doHealthCheck(Health.Builder builder) throws Exception {
           // Use the builder to build the health status details that should be reported.
           // If you throw an exception, the status will be DOWN with the exception message.
           
           builder.up()
                   .withDetail("app", "Alive and Kicking")
                   .withDetail("error", "Nothing! I'm good.");
       }
   }
   ```

   一旦你增加上面的健康指标到你的应用中去后，`health` endpoint将展示如下细节:

   ```json
   {
      "status":"UP",
      "details":{
         "custom":{
            "status":"UP",
            "details":{
               "app":"Alive and Kicking",
               "error":"Nothing! I'm good."
            }
         },
         "diskSpace":{
            "status":"UP",
            "details":{
               "total":250790436864,
               "free":97949245440,
               "threshold":10485760
            }
         }
      }
   }
   ```

**/metrics endpoint**

`metrics` endpoint展示了你可以追踪的所有的度量。

```json
{
    "names": [
        "jvm.memory.max",
        "http.server.requests",
        "process.files.max",
        ...
        "tomcat.threads.busy",
        "process.start.time",
        "tomcat.servlet.error"
    ]
}
```

想要获得每个度量的详细信息，你需要传递度量的名称到URL中，像

[http://localhost:8080/actuator/metrics/{MetricName}](http://localhost:8080/actuator/metrics/{MetricName)

举个例子，获得`systems.cpu.usage`的详细信息，使用以下URLhttp://localhost:8080/actuator/metrics/system.cpu.usage。它将显示如下内容:

```json
{
    "name": "system.cpu.usage",
    "measurements": [
    {
        "statistic": "VALUE",
        "value": 0
    }
    ],
"availableTags": []
}
```

**/loggers endpoint**

`loggers` endpoint，可以通过访问http://localhost:8080/actuator/loggers来进入。它展示了应用中可配置的loggers的列表和相关的日志等级。

你同样能够使用[http://localhost:8080/actuator/loggers/{name}](http://localhost:8080/actuator/loggers/{name)来展示特定logger的细节。

举个例子，为了获得`root` logger的细节，你可以使用http://localhost:8080/actuator/loggers/root：

```json
{
   "configuredLevel":"INFO",
   "effectiveLevel":"INFO"
}
```

在运行时改变日志等级：

`loggers` endpoint也允许你在运行时改变应用的日志等级。

举个例子，为了改变`root` logger的等级为`DEBUG` ，发送一个`POST`请求到http://localhost:8080/actuator/loggers/root，加入如下参数

```json
{
   "configuredLevel": "DEBUG"
}
```

这个功能对于线上问题的排查非常有用。

同时，你可以通过传递`null`值给`configuredLevel`来重置日志等级。

**/info endpoint**

`info` endpoint展示了应用的基本信息。它通过`META-INF/build-info.properties`来获得编译信息，通过`git.properties`来获得Git信息。它同时可以展示任何其他信息，只要这个环境property中含有`info`key。

你可以增加properties到`application.yaml`中，比如：

```kotlin
# INFO ENDPOINT CONFIGURATION
info:
  app:
    name: @project.name@
    description: @project.description@
    version: @project.version@
    encoding: @project.build.sourceEncoding@
    java:
      version: @java.version@
```

注意，我使用了Spring Boot的[Automatic property expansion](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-automatic-expansion) 特征来扩展来自maven工程的properties。

一旦你增加上面的properties，`info` endpoint将展示如下信息：

```json
{
    "app": {
    "name": "actuator",
    "description": "Demo project for Spring Boot",
    "version": "0.0.1-SNAPSHOT",
    "encoding": "UTF-8",
    "java": {
        "version": "1.8.0_161"
        }
    }
}
```

#### 使用Spring Security来保证Actuator Endpoints安全

Actuator endpoints是敏感的，必须保障进入是被授权的。如果Spring Security是包含在你的应用中，那么endpoint是通过HTTP认证被保护起来的。

如果没有， 你可以增加以下以来到你的应用中去：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

接下去让我们看一下如何覆写spring security配置，并且定义你自己的进入规则。

下面的例子展示了一个简单的spring securiy配置。它使用叫做[`EndPointRequest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/autoconfigure/security/servlet/EndpointRequest.html)

的`ReqeustMatcher`工厂模式来配置Actuator endpoints进入规则。

```java
import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
import org.springframework.boot.actuate.context.ShutdownEndpoint;
import org.springframework.boot.autoconfigure.security.servlet.PathRequest;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class ActuatorSecurityConfig extends WebSecurityConfigurerAdapter {

    /*
        This spring security configuration does the following

        1. Restrict access to the Shutdown endpoint to the ACTUATOR_ADMIN role.
        2. Allow access to all other actuator endpoints.
        3. Allow access to static resources.
        4. Allow access to the home page (/).
        5. All other requests need to be authenticated.
        5. Enable http basic authentication to make the configuration complete.
           You are free to use any other form of authentication.
     */

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                    .requestMatchers(EndpointRequest.to(ShutdownEndpoint.class))
                        .hasRole("ACTUATOR_ADMIN")
                    .requestMatchers(EndpointRequest.toAnyEndpoint())
                        .permitAll()
                    .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
                        .permitAll()
                    .antMatchers("/")
                        .permitAll()
                    .antMatchers("/**")
                        .authenticated()
                .and()
                .httpBasic();
    }
}
```

为了能够测试以上的配置，你可以在`application.yaml`中增加spring security用户。

```bash
# Spring Security Default user name and password
spring:
  security:
    user:
      name: actuator
      password: actuator
      roles: ACTUATOR_ADMIN
```

#### Prometheus&Grafana

Prometheus是一个开源的监控系统，起源于[SoundCloud](https://soundcloud.com/)。它由以下几个核心组件构成：

- 数据爬虫：根据配置的时间定期的通过HTTP抓去metrics数据。
- [time-series](https://en.wikipedia.org/wiki/Time_series) 数据库：存储所有的metrics数据。
- 简单的用户交互接口：可视化、查询和监控所有的metrics

Grafana使你能够把来自不同数据源比如Elasticsearch, Prometheus, Graphite, influxDB等多样的数据以绚丽的图标展示出来。

它也能基于你的metrics数据发出告警。当一个告警状态改变时，它能通知你通过email，slack或者其他途径。

值得注意的是，Prometheus仪表盘也有简单的图标。但是Grafana的图表表现的更好。这也是为什么，在这篇文章中，我们将整合Grafana和Pormetheus来可视化metrics数据。

Spring Boot使用[Micrometer](http://micrometer.io/)，一个应用metrics组件，将actuator metrics整合到外部监控系统中。

它支持很多种监控系统，比如Netflix Atalas, AWS Cloudwatch, Datadog, InfluxData, SignalFx, Graphite, Wavefront和Prometheus等。

为了整合Prometheus，你需要增加`micrometer-registry-prometheus`依赖：

```xml
<!-- Micrometer Prometheus registry  -->
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

一旦你增加上述的依赖，Spring Boot会自动配置一个[`PrometheusMeterRegistry`](https://github.com/micrometer-metrics/micrometer/blob/master/implementations/micrometer-registry-prometheus/src/main/java/io/micrometer/prometheus/PrometheusMeterRegistry.java)和[`CollectorRegistry`](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/CollectorRegistry.java)来收集和输出格式化的metrics数据，使得Prometheus服务器可以爬取。

所有应用的metrics数据是根据一个叫`/prometheus`的endpoint来设置是否可用。Prometheus服务器可以周期性的爬取这个endpoint来获取metrics数据。

##### 解析Spring Boot Actuator的/prometheus Endpoint

首先，你可以通过actuator endpoint-discovery页面(http://localhost:8080/actuator)来看一下`prometheus` endpoint。

```
"prometheus": {
"href": "http://127.0.0.1:8080/actuator/prometheus",
"templated": false
}
```

`prometheus` endpoint暴露了格式化的metrics数据给Prometheus服务器。你可以通过`prometheus` endpoint(http://localhost:8080/actuator/prometheus)看到被暴露的metrics数据:

```
# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for  the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{area="nonheap",id="Code Cache",} 9830400.0
jvm_memory_committed_bytes{area="nonheap",id="Metaspace",} 4.3032576E7
jvm_memory_committed_bytes{area="nonheap",id="Compressed Class Space",} 6070272.0
jvm_memory_committed_bytes{area="heap",id="PS Eden Space",} 2.63192576E8
jvm_memory_committed_bytes{area="heap",id="PS Survivor Space",} 1.2058624E7
jvm_memory_committed_bytes{area="heap",id="PS Old Gen",} 1.96608E8
# HELP logback_events_total Number of error level events that made it to the logs
# TYPE logback_events_total counter
logback_events_total{level="error",} 0.0
logback_events_total{level="warn",} 0.0
logback_events_total{level="info",} 42.0
logback_events_total{level="debug",} 0.0
logback_events_total{level="trace",} 0.0
...
```

##### 使用Docker下载和运行Prometheus

```
$ docker pull prom/prometheus
```

接下来，我们需要配置Prometheus来抓取Spring Boot Actuator的`/prometheus` endpoint中的metrics数据。

创建一个`prometheus.yml`的文件，填入以下内容：

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['127.0.0.1:9090']
  - job_name: 'spring-actuator'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
    - targets: ['HOST_IP:8080']
```

在Prometheus文档中，上面的配置文件是[basic configuration file](https://prometheus.io/docs/prometheus/latest/getting_started/#configuring-prometheus-to-monitor-itself)的扩展。

上面中比较重要的配置项是`spring-actuator` job中的`scrape_configs`选项。

`metrics_path`是Actuator中`prometheus` endpoint中的路径。`targes`包含了Spring Boot应用的`HOST`和`PORT`。

请确保替换`HOST_IP`为你Spring Boot应用运行的电脑的IP地址。值得注意的是，`localhost`将不起作用，因为我们将从docker container中连接HOST机器。你必须设置网络IP地址。

最后，让我们在Docker中运行Prometheus。使用以下命令来启动一个Prometheus服务器。

```shell
$ docker run -d --name=prometheus -p 9090:9090 -v <PATH_TO_prometheus.yml_FILE>:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml

```

请确保替换为你在上面创建的Prometheus配置文件的保存的路径。

你可以通过访问[http://localhost:9090](http://localhost:9090/)访问Prometheus仪表盘。你可以通过Prometheus查询表达式来查询metrics。

下面是一些例子：

- 系统CPU使用

  ![lUlTqP.png](https://s2.ax1x.com/2020/01/03/lUlTqP.png)

- API的延迟响应

  ![lUlzMn.png](https://s2.ax1x.com/2020/01/03/lUlzMn.png)

你可以从Prometheus官方文档中学习更多的 [`Prometheus Query Expressions`](https://prometheus.io/docs/introduction/first_steps/#using-the-expression-browser)。

##### 使用Docker下载和运行Grafana

```
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

你可以访问[http://localhost:3000](http://localhost:3000/)，并且使用默认的账户名(admin)密码(admin)来登录Grafana。

通过以下几步导入Prometheus中的metrics数据并且在Grafana上可视化。

1. 在Grafana上增加Prometheus数据源

   ![lU1AG4.png](https://s2.ax1x.com/2020/01/03/lU1AG4.png)

2. 建立一个仪表盘图表

   ![lU1QIO.png](https://s2.ax1x.com/2020/01/03/lU1QIO.png)

3. 添加一个Prometheus查询

   ![lU1tsI.png](https://s2.ax1x.com/2020/01/03/lU1tsI.png)

4. 默认的可视化

   ![lU1ddf.png](https://s2.ax1x.com/2020/01/03/lU1ddf.png)

### Spring Boot Admin

Spring Boot Actuator 提供了对单个 Spring Boot 的监控，信息包含：应用状态、内存、线程、堆栈等等，比较全面的监控了 Spring Boot 应用的整个生命周期。

但是这样监控也有一些问题：第一，所有的监控都需要调用固定的接口来查看，如果全面查看应用状态需要调用很多接口，并且接口返回的 Json 信息不方便运营人员理解；第二，如果 Spring Boot 应用集群非常大，每个应用都需要调用不同的接口来查看监控信息，操作非常繁琐低效。在这样的背景下，就诞生了另外一个开源软件：**Spring Boot Admin**。

Spring Boot Admin 是一个管理和监控 Spring Boot 应用程序的开源软件。每个应用都认为是一个客户端，通过 HTTP 或者使用 Eureka 注册到 admin server 中进行展示，Spring Boot Admin UI 部分使用 VueJs 将数据展示在前端。

#### 监控单体应用

##### Admin Server 端

**项目依赖**

```xml
<dependencies>
  <dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

**配置文件**

```
server.port=8000
```

服务端设置端口为：8000。

**启动类**

```java
@Configuration
@EnableAutoConfiguration
@EnableAdminServer
public class AdminServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(AdminServerApplication.class, args);
  }
}
```

完成上面三步之后，启动服务端，浏览器访问`http://localhost:8000`可以看到以下界面：

![1mvZ6S.png](https://s2.ax1x.com/2020/01/26/1mvZ6S.png)

##### Admin Client 端

**项目依赖**

```xml
<dependencies>
    <dependency>
      <groupId>de.codecentric</groupId>
      <artifactId>spring-boot-admin-starter-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**配置文件**

```properties
server.port=8001
spring.application.name=Admin Client
spring.boot.admin.client.url=http://localhost:8000  
management.endpoints.web.exposure.include=*
```

- `spring.boot.admin.client.url` 配置 Admin Server 的地址
- `management.endpoints.web.exposure.include=*` 打开客户端 Actuator 的监控。

**启动类**

```java
@SpringBootApplication
public class AdminClientApplication {
  public static void main(String[] args) {
    SpringApplication.run(AdminClientApplication.class, args);
  }
}
```

配置完成之后，启动 Client 端，Admin 服务端会自动检查到客户端的变化，并展示其应用

![1mvbng.png](https://s2.ax1x.com/2020/01/26/1mvbng.png)

页面会展示被监控的服务列表，点击详项目名称会进入此应用的详细监控信息。

#### 监控微服务

如果我们使用的是单个 Spring Boot 应用，就需要在每一个被监控的应用中配置 Admin Server 的地址信息；如果应用都注册在 Eureka 中就不需要再对每个应用进行配置，Spring Boot Admin 会自动从注册中心抓取应用的相关信息。

如果我们使用了 Spring Cloud 的服务发现功能，就不需要在单独添加 Admin Client 客户端，仅仅需要 Spring Boot Server ,其它内容会自动进行配置。

接下来我们以 Eureka 作为服务发现的示例来进行演示，实际上也可以使用 Consul 或者 Zookeeper。

1. 服务端和客户端添加 spring-cloud-starter-eureka 到包依赖中

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 启动类添加注解

   ```java
   @Configuration
   @EnableAutoConfiguration
   @EnableDiscoveryClient
   @EnableAdminServer
   public class SpringBootAdminApplication {
       public static void main(String[] args) {
           SpringApplication.run(SpringBootAdminApplication.class, args);
       }
   
       @Configuration
       public static class SecurityPermitAllConfig extends WebSecurityConfigurerAdapter {
           @Override
           protected void configure(HttpSecurity http) throws Exception {
               http.authorizeRequests().anyRequest().permitAll()  
                   .and().csrf().disable();
           }
       }
   }
   ```

   使用类 SecurityPermitAllConfig 关闭了安全验证。

3. 在客户端中配置服务发现的地址

   ```properties
   eureka:   
     instance:
       leaseRenewalIntervalInSeconds: 10
       health-check-url-path: /actuator/health
       metadata-map:
         startup: ${random.int}    #needed to trigger info and endpoint update after restart
     client:
       registryFetchIntervalSeconds: 5
       serviceUrl:
         defaultZone: ${EUREKA_SERVICE_URL:http://localhost:8761}/eureka/
   
   management:
     endpoints:
       web:
         exposure:
           include: "*"  
     endpoint:
       health:
         show-details: ALWAYS
   ```

   Spring Cloud 提供了示例代码可以参考这里：[spring-boot-admin-sample-eureka](https://github.com/codecentric/spring-boot-admin/tree/master/spring-boot-admin-samples/spring-boot-admin-sample-eureka/)

   重启启动服务端和客户端之后，访问服务端的相关地址就可以看到监控页面了。

### 参考

1. [Spring Boot Actuator: Production-ready features](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)
2. [对没有监控的微服务Say No！](http://mp.163.com/v2/article/detail/D7SQCHGT0511FQO9.html)
3. [Spring Boot Actuator 使用](https://www.jianshu.com/p/af9738634a21)