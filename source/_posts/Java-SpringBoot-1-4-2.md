---
title: Spring boot 数据访问 MongoDB
date: 2019-07-12 19:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

MongoDB 是最早热门非关系数据库的之一，使用也比较普遍，一般会用做离线数据分析来使用，放到内网的居多。由于很多公司使用了云服务，服务器默认都开放了外网地址，导致前一阵子大批 MongoDB 因配置漏洞被攻击，数据被删，引起了人们的注意，感兴趣的可以看看这篇文章：[场屠戮MongoDB的盛宴反思：超33000个数据库遭遇入侵勒索](http://www.freebuf.com/articles/database/125127.html)，同时也说明了很多公司生产中大量使用mongodb。

<!--more-->

正式开始之前，第一步就是需要安装Mongo的环境了，因为环境的安装和我们spring的主题没有太大的关系，因此我们选择最简单的使用姿势：直接用docker来安装mongo来使用

```bash
$ docker pull mongo
$ docker volume create mongodb_data
$ docker images mongo
#运行
$ docker run --name mongo  -p 27017:27017 -v mongodb_data:/data/db -d mongo
$ docker exec -it mongo mongo admin
# 创建一个名为 admin，密码为 123456 的用户。
>  db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'}]});
# 尝试使用上面创建的用户信息进行连接。
> db.auth('admin', '123456')
```

命令说明：

**-p 27017:27017 :**将容器的27017(第一个) 端口映射到主机的27017 端口

**-v mongodb_data:/data/db :**将主机中mongodb_data(第一个)挂载到容器的/data/db，作为mongo数据存储目录

查看容器启动情况

```
$ docker ps
```

使用mongo镜像执行mongo 命令连接到刚启动的容器,主机IP为172.17.0.1，查看docker0网卡的IP地址

```
$ docker run -it mongo mongo --host 172.17.0.1
```

#### 示例

官方文档：<https://docs.spring.io/spring-data/mongodb/docs/2.2.4.RELEASE/reference/html/#reference>

1. pom依赖

   ```xml
          <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-mongodb</artifactId>
           </dependency>
   ```

2. 配置文件

   配置文件如下，主要就是连接mongo的url

   ```html
   spring.data.mongodb.uri=mongodb://admin:123456@localhost:27017/test?authSource=admin&authMechanism=SCRAM-SHA-1
   ```

   通过上面的实例，也知道格式如下:

   ```
   mongodb://用户名:密码@host:port/dbNmae?参数
   ```

   - 当没有用户名和密码时，可以省略掉中间的 `root:root@`；

   - 当需要认证时，请格外注意

     - mongodb新版的验证方式改成了

       ```
       SCRAM-SHA-1
       ```

       ，所以参数中一定一定一定得加上

       - `?authSource=admin&authMechanism=SCRAM-SHA-1`

     - 如果将mongodb的验证方式改成了`MONGODB-CR`, 则上面的可以不需要

3.  测试使用

   写一个简单的测试类，看下mongodb是否连接成功，是否可以正常操作

   ```java
   @Slf4j
   @Component
   public class MongoTemplateHelper {
   
       @Getter
       @Setter
       private MongoTemplate mongoTemplate;
   
       public MongoTemplateHelper(MongoTemplate mongoTemplate) {
           this.mongoTemplate = mongoTemplate;
       }
   
   
   
       /**
        * 保存记录
        *
        * @param params
        * @param collectionName
        */
       public void saveRecord(Map<String, Object> params, String collectionName) {
           mongoTemplate.save(params, collectionName);
       }
   
       /**
        * 精确查询方式
        *
        * @param query
        * @param collectionName
        */
       public void queryRecord(Map<String, Object> query, String collectionName) {
           Criteria criteria = null;
           for (Map.Entry<String, Object> entry : query.entrySet()) {
               if (criteria == null) {
                   criteria = Criteria.where(entry.getKey()).is(entry.getValue());
               } else {
                   criteria.and(entry.getKey()).is(entry.getValue());
               }
           }
   
           Query q = new Query(criteria);
           Map result = mongoTemplate.findOne(q, Map.class, collectionName);
           log.info("{}", result);
       }
   }
   ```

   上面提供了两个方法，新增和查询，简单的使用姿势如

   ```java
   @SpringBootApplication
   public class Application {
   
       private static final String COLLECTION_NAME = "personal_info";
   
   
       public Application(MongoTemplateHelper mongoTemplateHelper) {
           Map<String, Object> records = new HashMap<>(4);
           records.put("name", "平心Blog");
           records.put("github", "hanyunpeng0521.github.io");
           records.put("time", LocalDateTime.now());
   
           mongoTemplateHelper.saveRecord(records, COLLECTION_NAME);
   
           Map<String, Object> query = new HashMap<>(4);
           query.put("name", "平心Blog");
           mongoTemplateHelper.queryRecord(query, COLLECTION_NAME);
       }
   
       public static void main(String[] args) {
           SpringApplication.run(Application.class);
       }
   
   }
   ```

   然后开始执行，查看输出

   ```
   2020-01-24 23:22:56.122  INFO 12779 --- [localhost:27017] org.mongodb.driver.cluster               : Monitor thread successfully connected to server with description ServerDescription{address=localhost:27017, type=STANDALONE, state=CONNECTED, ok=true, version=ServerVersion{versionList=[4, 2, 2]}, minWireVersion=0, maxWireVersion=8, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=30, roundTripTimeNanos=7702851}
   
   ```

4. 增删改查

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class UserInfo {
       @Id
       private Long id;
   
       private String username;
   
       private String password;
   
   }
   public interface UserRepository extends MongoRepository<UserInfo,Long> {
   }
   
   @RestController
   @RequestMapping("/user/")
   public class UserController {
       @Autowired
       private UserRepository userRepository;
   
       @GetMapping("save")
       public String save() {
           UserInfo userInfo = new UserInfo(System.currentTimeMillis(), "用户" + System.currentTimeMillis(), "123");
           userRepository.save(userInfo);
           return "success";
       }
   
   
       @GetMapping("getUserList")
       public List<UserInfo> getUserList() {
           List<UserInfo> userInfoList = userRepository.findAll();
           return userInfoList;
       }
   
       @GetMapping("delete")
       public String delete(Long id) {
           userRepository.deleteById(id);
           return "success";
       }
   
       @GetMapping("update")
       public String update(Long id, String username, String password) {
           UserInfo userInfo = new UserInfo(id, username, password);
           userRepository.save(userInfo);
           return "success";
       }
   }
   ```

   到这里就结束了，可以启动项目访问http://localhost:8888/user/save创建几条数据。

   然后访问http://localhost:8888/user/getUserList可以查看刚才创建的数据

   修改和删除这里就不做测试了，在方法上有对应的测试访问地址。

这里做一个简单的总结，通过整合几种数据库，包含关系型数据mysql，文件式数据库mongodb，甚至说elasticsearch等等其实步骤都大致如下：

1. 加入对应依赖
2. 配置文件配置对应数据库信息
3. 数据操作层继承想要的repository

#### 自动配置

自动配置类位于包：org.springframework.boot.autoconfigure.mongo

1. MongoAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   //如果存在MongoClient
   @ConditionalOnClass(MongoClient.class)
   //配置属性类
   @EnableConfigurationProperties(MongoProperties.class)
   //如果没有该类型MongoDbFactory的bean
   @ConditionalOnMissingBean(type = "org.springframework.data.mongodb.MongoDbFactory")
   public class MongoAutoConfiguration {
   
       //如果没有该类的bean，创建MongoClient
   	@Bean
   	@ConditionalOnMissingBean(type = { "com.mongodb.MongoClient", "com.mongodb.client.MongoClient" })
   	public MongoClient mongo(MongoProperties properties, ObjectProvider<MongoClientOptions> options,
   			Environment environment) {
   		return new MongoClientFactory(properties, environment).createMongoClient(options.getIfAvailable());
   	}
   
   }
   ```

2. MongoProperties,可配置的属性

   ```java
   @ConfigurationProperties(prefix = "spring.data.mongodb")
   public class MongoProperties {
   
   	/**
   	 * Default port used when the configured port is {@code null}.
   	 */
   	public static final int DEFAULT_PORT = 27017;
   
   	/**
   	 * Default URI used when the configured URI is {@code null}.
   	 */
   	public static final String DEFAULT_URI = "mongodb://localhost/test";
   
   	/**
   	 * Mongo server host. Cannot be set with URI.
   	 */
   	private String host;
   
   	/**
   	 * Mongo server port. Cannot be set with URI.
   	 */
   	private Integer port = null;
   
   	/**
   	 * Mongo database URI. Cannot be set with host, port and credentials.
   	 */
   	private String uri;
   
   	/**
   	 * Database name.
   	 */
   	private String database;
   
   	/**
   	 * Authentication database name.
   	 */
   	private String authenticationDatabase;
   
   	/**
   	 * GridFS database name.
   	 */
   	private String gridFsDatabase;
   
   	/**
   	 * Login user of the mongo server. Cannot be set with URI.
   	 */
   
   	private String username;
   
   	/**
   	 * Login password of the mongo server. Cannot be set with URI.
   	 */
   	private char[] password;
   
   	/**
   	 * Fully qualified name of the FieldNamingStrategy to use.
   	 */
   	private Class<?> fieldNamingStrategy;
   
   	/**
   	 * Whether to enable auto-index creation.
   	 */
   	private Boolean autoIndexCreation;
   	
   	}
   ```

3. MongoDB工厂类：org.springframework.data.mongodb.MongoDbFactory

   ```
   public interface MongoDbFactory extends CodecRegistryProvider, MongoSessionProvider {
       MongoDatabase getDb() throws DataAccessException;
   
       MongoDatabase getDb(String var1) throws DataAccessException;
   
       PersistenceExceptionTranslator getExceptionTranslator();
   
       /** @deprecated */
       @Deprecated
       DB getLegacyDb();
   
       default CodecRegistry getCodecRegistry() {
           return this.getDb().getCodecRegistry();
       }
   
       ClientSession getSession(ClientSessionOptions var1);
   
       default MongoDbFactory withSession(ClientSessionOptions options) {
           return this.withSession(this.getSession(options));
       }
   
       MongoDbFactory withSession(ClientSession var1);
   
       default boolean isTransactionActive() {
           return false;
       }
   }
   ```

4. MongoReactiveAutoConfiguration:Reactive模式的自动配置

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ MongoClient.class, Flux.class })
   //配置类
   @EnableConfigurationProperties(MongoProperties.class)
   public class MongoReactiveAutoConfiguration {
   	//MongoClient客户端连接
   	@Bean
   	@ConditionalOnMissingBean
   	public MongoClient reactiveStreamsMongoClient(MongoProperties properties, Environment environment,
   			ObjectProvider<MongoClientSettingsBuilderCustomizer> builderCustomizers,
   			ObjectProvider<MongoClientSettings> settings) {
   		ReactiveMongoClientFactory factory = new ReactiveMongoClientFactory(properties, environment,
   				builderCustomizers.orderedStream().collect(Collectors.toList()));
   		return factory.createMongoClient(settings.getIfAvailable());
   	}
   
       //Netty驱动
   	@Configuration(proxyBeanMethods = false)
   	@ConditionalOnClass({ SocketChannel.class, NioEventLoopGroup.class })
   	static class NettyDriverConfiguration {
   
   		@Bean
   		@Order(Ordered.HIGHEST_PRECEDENCE)
   		NettyDriverMongoClientSettingsBuilderCustomizer nettyDriverCustomizer(
   				ObjectProvider<MongoClientSettings> settings) {
   			return new NettyDriverMongoClientSettingsBuilderCustomizer(settings);
   		}
   
   	}
   
   	/**
   	 * {@link MongoClientSettingsBuilderCustomizer} to apply Mongo client settings.
   	 使用Netty构建异步MongoDB
   	 */
   	private static final class NettyDriverMongoClientSettingsBuilderCustomizer
   			implements MongoClientSettingsBuilderCustomizer, DisposableBean {
   
   		private final ObjectProvider<MongoClientSettings> settings;
   
   		private volatile EventLoopGroup eventLoopGroup;
   
   		private NettyDriverMongoClientSettingsBuilderCustomizer(ObjectProvider<MongoClientSettings> settings) {
   			this.settings = settings;
   		}
   
   		@Override
   		public void customize(Builder builder) {
   			if (!isStreamFactoryFactoryDefined(this.settings.getIfAvailable())) {
   				NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
   				this.eventLoopGroup = eventLoopGroup;
   				builder.streamFactoryFactory(
   						NettyStreamFactoryFactory.builder().eventLoopGroup(eventLoopGroup).build());
   			}
   		}
   
   		@Override
   		public void destroy() {
   			EventLoopGroup eventLoopGroup = this.eventLoopGroup;
   			if (eventLoopGroup != null) {
   				eventLoopGroup.shutdownGracefully().awaitUninterruptibly();
   				this.eventLoopGroup = null;
   			}
   		}
   
   		private boolean isStreamFactoryFactoryDefined(MongoClientSettings settings) {
   			return settings != null && settings.getStreamFactoryFactory() != null;
   		}
   
   	}
   
   }
   ```

