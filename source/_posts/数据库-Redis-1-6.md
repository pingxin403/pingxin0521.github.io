---
title: Redis 应用场景示例
date: 2020-2-14 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### Redis共享Session原理及示例

#### Redis共享session的作用

- 微服务自身可以保持无状态，应用实例数量的多少不会影响用户登录状态；
- 可实现单点登录的踢出功能，如可以让上次异地登录的用户下线；
- session在多个服务或服务器间共享，实现多站点单点登录(参考SSO原理)

<!--more-->

这只是一个解决方案。

#### Redis缓存session原理简述

其工作原理，可简单用图描述(假设服务A运行有有个多个实例)：

![1vmo59.png](https://s2.ax1x.com/2020/02/14/1vmo59.png)

#### Springboot-session结合Redis示例

1. 添加maven依赖

   ```
   <dependency>  
   	<groupId>org.springframework.boot</groupId>  
   	<artifactId>spring-boot-starter-redis</artifactId>
   	<version>1.4.0.RELEASE</version>
   </dependency>  
   <dependency>  
   	<groupId>org.springframework.session</groupId>  
   	<artifactId>spring-session-data-redis</artifactId>  
   </dependency>
   <dependency>
   	<groupId>org.springframework.session</groupId>
   	<artifactId>spring-session</artifactId>
   	<version>1.3.3.RELEASE</version>
   </dependency>
   ```

2. 添加redis配置（application.properties）

   ```properties
   spring.redis.database=1  
   spring.redis.host=localhost
   spring.redis.password=xxxxx
   spring.redis.port=6379
   
   # server.port=8080
   server.port=8081
   
   ```

3. 添加配置类

   ```java
   @Configuration  
   @EnableRedisHttpSession  
   public class RedisSessionConfig {  
   }
   ```

4. 添加controller

   ```java
   @RestController  
   @RequestMapping(value = "/")
   public class Hello {  
       @RequestMapping(value = "/set", method = RequestMethod.GET)  
       public Map<String, Object> firstResp (HttpServletRequest request){           
           request.getSession().setAttribute("testKey", "testValue");
   
           Map<String, Object> map = new HashMap<>(); 
           map.put("testKey", "testValue");  
           return map;
       }
     
       @RequestMapping(value = "/query", method = RequestMethod.GET)  
       public Object sessions (HttpServletRequest request){
           Map<String, Object> map = new HashMap<>();
           map.put("sessionId", request.getSession().getId());
           map.put("testKey", request.getSession().getAttribute("testKey"));
           return map;
       }
   } 
   ```

5. 浏览器访问测试

   - 启动两个springboot程序，分别监听8080，8081端口

   - 浏览器分别访问`http://127.0.0.1:8080/set`, `127.0.0.1:8081/query`接口,效果如下：

     ![1vnKGn.png](https://s2.ax1x.com/2020/02/14/1vnKGn.png)

     可以看出两个独立的应用已经共享了同一个session。




