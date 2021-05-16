---
title: 关于
date: 2019-04-20 14:09:40
type: "about"
comments: false
---
### 个人介绍

现就职：**三七互娱**–Java游戏平台开发工程师

本科：**江苏大学网络工程**

- Tel：（+86）13839441583
- E-mail：[m13839441583@163.com](mailto:m13839441583@163.com)
- Age: 21岁
- 博客地址：
  - [https://epxq36.coding-pages.com](https://epxq36.coding-pages.com/):部署在coding
  - https://pingxin0521.gitee.io/px_blog/:部署在gitee

#### 专业技能

1. 熟练掌握Java体系，熟悉并发开发知识，熟悉JVM原理和优化，熟悉Spring、Spring MVC、MyBatis、Hibernate、Spring Data 系列框架和Spring boot等应用框架的使用和原理，了解spring cloud微服务框架
2. 熟悉MySQL、Redis、MongoDB数据库的使用、运维和优化知识，了解其原理知识
3. 熟练掌握linux运维技术及Shell脚本开发;了解容器技术docker、k8s；了解负载均衡、反向代理和高可用等技术；熟悉微服务架构
4. 了解常见中间件技术使用和原理（Netty、Nginx、Jetty、Tomcat、Redis和Kafka）
5. 熟悉maven、gradle、git等常用开发工具的使用，熟练掌握uml对系统建模
6. 了解前端开发（js、html、css）和微信小程序开发，以及前后端交互相关原理和前后端分离相关技术

搞过ubuntu 18.04做个人电脑，搞过蜗牛星际做NAS，搞过Android手机的root来玩，也做过酒店服务员、婚礼布场，商品促销员等等

### 已部署项目

演示使用，不能保证性能

1. [company-frame](http://47.98.40.244:8021/): admin/666666

2. [spring-boot-shiro-frame](http://47.98.40.244:8022/): root/123456

3. blog

   : 无密码

   - [管理界面](http://47.98.40.244:8024/admin)： pingxin/111111

4. [云收藏](http://47.98.40.244:8025/): pingxin/pingxin123

5. 实验平台考核系统（

   毕设

   ）

   - 后端部署地址：http://47.98.40.244:8023/
   - 前端部署地址：http://47.98.40.244:8080/

### 项目

#### 学习笔记

博客对应位置查看：[https://hanyunpeng0521.github.io](https://hanyunpeng0521.github.io/)

java 基础学习：java特性和java web基础和原理

项目位置：[JavaLearn](https://github.com/hanyunpeng0521/JavaLearn)

**java框架学习**：mybatis、quartz、spring、spring MVC 、搜索引擎等，因为时间原因，很早的笔记没有保留下来

项目位置：[java-framework](https://github.com/hanyunpeng0521/java-framework)

**Spring Boot笔记**：主要是Spring boot跟其他框架的联合使用以及其本身的自动配置原理

项目位置：[spring-boot-learn](https://github.com/hanyunpeng0521/spring-boot-learn)

**安全框架笔记**：主要是Spring Security和Shiro的学习和一些比较简单的项目。

项目位置：[learn-sercurity](https://github.com/hanyunpeng0521/learn-sercurity)

#### 设备管理系统

Springboot+web+微信小程序设计实现

课设项目：[equipment-management](https://github.com/hanyunpeng0521/equipment-management)

主要角色：设备运维管理人员、维修人员、物业人员

主要对象：设备

对象设备的生命周期：新建–》安装–》保养/维修–》停用

##### 功能

1. 设备类，包括该类设备信息，包括常用名称、功能、场景。
2. 安装位置，安装地点、环境信息等
3. 设备组，选择设备类组，填写设备信息，如厂商名、购价、使用年限等，绑定默认保养计划
4. 设备，新建设备时，选择设备组，填写设备代码；安装设备时，选择安装位置、安装时间
5. 保养：保养计划，为每种设备组进行设置保养计划，可以针对不同安装位置的设备进行设置。保养计划也就是物业人员每隔多长时间进行检查。
6. 维修：由物业人员、管理人员提出维修任务，或者到达年限的设备会发送维修任务，维修人员收到后进行维修检查操作。
7. 停用：到达年限或者不可再用的设备，由管理人员设置不可用。

##### 环境

开发环境：Ubuntu 18.04+JDK 1.8+Maven 3.6+IntelliJ Idea 2018.3

运行环境：Ubuntu 18.04+JDK 1.8+Tomcat 9

使用技术:

- Spring boot 2.2.2.REALSE：快速开发、开箱即用
- Mybatis 3.6：持久化框架
- Mybatis plus：简化数据库访问
- Spring MVC 5：MVC框架
- zxing：二维码生成
- lombok:注解开发
- swagger ui：Web API文档生成

数据库：H2(测试)+MySQL 8.0

#### 问卷系统

课程设计项目，只完成SQL设计和数据库访问层设计：[Questionnaire](https://github.com/hanyunpeng0521/Questionnaire)

#### floor-drain

floor-drain是一款Web防护工具,是一款恶意web请求防范器，使用spring boot+Redis/Map进行实现

**流程**

[![8qQdiQ.png](https://s1.ax1x.com/2020/03/24/8qQdiQ.png)](https://s1.ax1x.com/2020/03/24/8qQdiQ.png)

**知识点**

- 理解Spring boot自定义starter
- 缓存理论（不涉及持久化）
- 高并发

项目位置：[ floor-drain-spring-boot-starter](https://github.com/hanyunpeng0521/floor-drain-spring-boot-starter)

#### Spring Boot API Project Seed

Spring Boot API Project Seed 是一个基于**Spring Boot & MyBatis**的种子项目，用于快速构建中小型API、RESTful API项目，该种子项目已经有过多个真实项目的实践，稳定、简单、快速，使我们摆脱那些重复劳动，专注于业务代码的编写，减少加班。下面是一个简单的使用演示，看如何基于本项目在短短几十秒钟内实现一套简单的API，并运行提供服务。

**框架介绍**

- Spring Boot 2.2.4
- lombok
- Mybatis+Mybatis plus
- MySQL+Redis 自定义缓存注解
- FastJson
- shiro+Jwt
- knife4j+swagger 2：Api文档
  文档访问位置：http://x.x.x.x:8080/doc.html

**特征&提供**

- 面向前后端分离项目
- 最佳实践的项目结构、配置文件、精简的POM
- 统一响应结果封装及**生成工具**
- 完整的统一异常处理
- 简单的接口签名认证
- 常用基础方法抽象封装
- 使用Druid Spring Boot Starter 集成Druid数据库连接池与监控
- 使用FastJsonHttpMessageConverter，提高JSON序列化速度
- 集成MyBatis、Mybatis plus，实现单表业务零SQL
- 提供代码生成器根据表名生成对应的Model、Mapper、MapperXML、Service、ServiceImpl、Controller等基础代码，其中Controller模板默认提供POST和RESTful两套，根据需求在`strategy.setRestControllerStyle(true)`方法中自己选择，默认使用POST模板。代码模板可根据实际项目的需求来扩展，由于每个公司业务都不太一样，所以只提供了一些比较基础、通用的模板，**主要是提供一个思路**来减少重复代码的编写，我在实际项目的使用中，其实根据公司业务的抽象编写了大量的模板。另外，使用模板也有助于保持团队代码风格的统一
- 自定义enum转换器，数据库存储为int，controller返回为String
- 自定义log注解，使用`com.company.project.business.annotation.BussinessLog`注解在controller方法，会自动生成http访问日志，如果选择save，则会保存道数据库
- 自定义RedisCache注解，简化缓存配置
- 简单的权限框架，基于shiro+JWT
- 另有彩蛋，待你探索

默认master为token认证

1. token分支：使用token认证,面向前后端分离和APP应用
2. session分支：使用Session认证，面向网页应用

位置: https://github.com/hanyunpeng0521/spring-boot-api-project-seed

#### 博客系统

开发环境：Intellij IDEA+MySQL+H2+Jetty

开发技术：java web+SSM+Spring boot+Thymeleaf+Semantic UI

感想体会：用实践克服经验不足，用知识加强自身能力，熟练使用不同技术和框架，对java web开发流程有了更深的认识，熟练项目分析与设计。

核心功能：博客的发布；检索文章；注册、登陆；

上述是[blog-sb-jpa](https://github.com/hanyunpeng0521/blog/tree/master/blog-sb-jpa)

项目地址: [blog](https://github.com/hanyunpeng0521/blog/)

包括：

1. [blog-servlet](https://github.com/hanyunpeng0521/blog/tree/master/blog-servlet):使用Servlet和Jsp开发,功能弱爆了
2. [blog-sb-jpa](https://github.com/hanyunpeng0521/blog/tree/master/blog-sb-jpa):使用Spring Boot+jpa进行开发，有所改进，就是功能太少了
3. [blog-sb-mb](https://github.com/hanyunpeng0521/blog/tree/master/blog-sb-mb):使用Spring boot+Mybatis,最新版,good good study

#### 购物商城项目

开发环境：Intellij IDEA+MySQL +Tomcat+Redis

开发框架：Spring Boot+SSM+Shiro+MySQL

项目地址：https://github.com/hanyunpeng0521/mall

- online-book-mall:在线图书商城，使用Spring boot+Mybatis
- shopping-sharing-jdbc：使用sharing-jdbc实现的分库分表的商品系统

感想体会：自顶向下设计网站架构，划分业务功能，熟练掌握框架的使用，并通过测试对应用进行纠错和改正；对用户信息进行保护，并将用户重要数据进行权限控制。

核心功能：注册、登陆、浏览商品、购物车、查询商品、退出 等模块

#### 仿云收藏网站

项目原型来自：https://github.com/cloudfavorites/favorites-web

项目地址：[cloud-collections-web](https://github.com/hanyunpeng0521/cloud-collections-web)

在一定基础上使用了前端代码，重写了后端和大部分前端代码，换JPA为Mybatis，简化了后端代码逻辑。

云收藏是一个使用 Spring Boot 构建的开源网站，可以让用户在线随时随地收藏的一个网站，在网站上分类整理收藏的网站或者文章，可以作为稍后阅读的一个临时存放。作为一个开放开源的软件，可以让用户从浏览器将收藏夹内容导入到云收藏，也支持随时将云收藏收集的文章导出去做备份。

核心功能点：

- 收藏、分类、检索文章
- 导出、导出（包活从浏览器中）
- 可以点赞、分享、讨论
- 注册、登录、个人账户
- 临时收藏、查看别人收藏
- 其它…

**项目使用技术**

- Vue
- Bootstrap
- jQuery
- Thymeleaf
- Mybatis
- Spring Boot Mail
- WebJars
- Mysql
- Tomcat
- Redis

#### Safety-Drill

基于spring boot 2.2.4、shiro、session/jwt、MySQL、redis、swagger2、mybatis 、thymeleaf/freemarker，权限控制的方式为 RBAC。代码通熟易懂，可以用做练手项目。

位置：https://github.com/hanyunpeng0521/safety-drill

包括：

- [spring-boot-shiro-frame](https://github.com/hanyunpeng0521/safety-drill/tree/master/spring-boot-shiro-frame): 基于Session进行会话控制，使用FreeMarker模板引擎
- [company-frame](https://github.com/hanyunpeng0521/safety-drill/tree/master/company-frame): 基于JWT进行会话，使用thymeleaf模板引擎

#### 实验平台考核子系统

使用远程虚拟实验技术进行实现，使用B/S软件架构，利用监控模块对浏览器控件的变化信息进行记录，去除记录过程的噪音数据，同时将数据保存在服务器端，可以在浏览器选择将数据进行导出和导入，同时将数据进行编解码或者转换成人类或者计算机能够看懂的语言。记录文件可以在浏览器端使用，选择记录文件后可以选择播放或者一帧一帧播放。

1. 在浏览器操作控件，系统记录控件的变化信息。学生可以选择自动记录操作步骤，或者手动点击关键步骤进行记录；
2. 记录的内容可以生成记录文件，保存到指定路径，并可以下载到本地；
3. 教师可以通过浏览器选择记录文件进行回放查看；分为自动回放和手动回放，手动回放可以点击选择上一帧/下一帧播放（即上一组/下一组记录数据）；
4. 教师可通过教师端界面查看教学班中没有提交记录文件的学生名单；
5. 教师可以对记录文件进行打分，保存到数据库，并可以对分数进行修改。可以导出成绩单到本地；
6. 教师可以上传记录文件到服务器，也可以将服务器端的记录文件下载到本地。

**使用技术**

1. 服务器：使用Java语言进行实现，同时使用Spring Boot 框架来进行功能的完成
2. 数据：使用MySQL关系型数据库保存结构化数据，使用非关系型数据库Redis和MongoDB保存其他数据
3. 浏览器：使用前后端分离的架构模式，前端使用Vue框架和Element组件库联合开发，保证前端界面的简洁美观和高性能。
4. 运维：使用容器化技术，将其在生产阶段进行打包为容器，在经过测试之后，使用测试通过的容器进行部署，保证服务的隔离性和独立性。依照具体情况使用监控组件对服务的运行状况进行数据收集和危险报警。

**技术栈**

- Java 8
- Spring Boot
- MySQL
- Redis
- MongoDB
- Fastjson
- JWT
- Shiro
- Mybatis
- Druid数据库连接池
- Swagger
- Vue
- Element

##### **项目位置**

- 码云地址：https://gitee.com/pingxin0521/experiment-platform
- 前端代码地址：https://gitee.com/pingxin0521/experiment-platform-vue
- 后端部署地址：http://47.98.40.244:8023/
- 前端部署地址：http://47.98.40.244:8080/

#### 微服务商城项目

TODOhttps://github.com/hanyunpeng0521/PX-Cloud)