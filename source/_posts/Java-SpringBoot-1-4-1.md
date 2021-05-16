---
title: Spring boot 数据访问 Redis和缓存
date: 2019-06-30 19:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

在 Spring 的生态中，我们使用 [Spring Data Redis](https://spring.io/projects/spring-data-redis) 来实现对 Redis 的数据访问。

可能这个时候，会有胖友会有疑惑，市面上已经有 Redis、Redisson、Lettuce 等优秀的 Java Redis 工具库，为什么还要有 Spring Data Redis 呢？学不动了，头都要秃了！不要慌，我们先来看一张图：

![QzsSWq.png](https://s2.ax1x.com/2019/12/22/QzsSWq.png)

- 对于下层，Spring Data Redis 提供了统一的操作模板（后文中，我们会看到是 RedisTemplate 类），封装了 Jedis、Lettuce 的 API 操作，访问 Redis 数据。所以，**实际上，Spring Data Redis 内置真正访问的实际是 Jedis、Lettuce 等 API 操作**。
- 对于上层，开发者学习如何使用 Spring Data Redis 即可，而无需关心 Jedis、Lettuce 的 API 操作。甚至，未来如果我们想将 Redis 访问从 Jedis 迁移成 Lettuce 来，无需做任何的变动。😈 相信很多胖友，在选择 Java Redis 工具库，也是有过烦恼的。
- 目前，Spring Data Redis 暂时只支持 Jedis、Lettuce 的内部封装，而 Redisson 是由 [redisson-spring-data](https://github.com/redisson/redisson/tree/master/redisson-spring-data) 来提供。

Spring Data Redis默认使用Lettuce框架

Lettuce 是一个可伸缩线程安全的 Redis 客户端，多个线程可以共享同一个 RedisConnection，它利用优秀 netty NIO 框架来高效地管理多个连接。

#### 示例

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

**配置文件**

```properties
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0
```

**添加 cache 的配置类**

```java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport{
    
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }
}
```

注意我们使用了注解：`@EnableCaching`来开启缓存。

**测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestRedis {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void test() throws Exception {
        stringRedisTemplate.opsForValue().set("aaa", "111");
        Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
    }
    
    @Test
    public void testObj() throws Exception {
        User user=new User("aa@126.com", "aa", "aa123456", "aa","123");
        ValueOperations<String, User> operations=redisTemplate.opsForValue();
        operations.set("com.neox", user);
        operations.set("com.neo.f", user,1, TimeUnit.SECONDS);
        Thread.sleep(1000);
        boolean exists=redisTemplate.hasKey("com.neo.f");
        if(exists){
            System.out.println("exists is true");
        }else{
            System.out.println("exists is false");
        }
    }
}
```

以上都是手动使用的方式，如何在查找数据库的时候自动使用缓存呢，看下面；

**自动根据方法生成缓存**

```java
@RestController
public class UserController {

    @RequestMapping("/getUser")
    @Cacheable(value="user-key")
    public User getUser() {
        User user=new User("aa@126.com", "aa", "aa123456", "aa","123");
        System.out.println("若下面没出现“无缓存的时候调用”字样且能打印出数据表示测试成功");
        return user;
    }
}
```

其中 value 的值就是缓存到 Redis 中的 key

#### 自动配置

查看自动配置类,位置`org.springframework.boot.autoconfigure.data.redis`

1. RedisAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass(RedisOperations.class)
   //属性类
   @EnableConfigurationProperties(RedisProperties.class)
   //导入Lettuce和Jedis客户端配置
   @Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
   public class RedisAutoConfiguration {
   
       //当没有自定义redisTemplate时注入，依赖redisConnectionFactory
   	@Bean
   	@ConditionalOnMissingBean(name = "redisTemplate")
   	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
   			throws UnknownHostException {
   		RedisTemplate<Object, Object> template = new RedisTemplate<>();
   		template.setConnectionFactory(redisConnectionFactory);
   		return template;
   	}
   
         //当没有自定义stringRedisTemplate时注入，依赖redisConnectionFactory
   	@Bean
   	@ConditionalOnMissingBean
   	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
   			throws UnknownHostException {
   		StringRedisTemplate template = new StringRedisTemplate();
   		template.setConnectionFactory(redisConnectionFactory);
   		return template;
       }
   }
   
   ```

2. RedisProperties配置和默认属性

   ```java
   @ConfigurationProperties(prefix = "spring.redis")
   public class RedisProperties {
   
   	/**
   	 * Database index used by the connection factory.
   	 */
   	private int database = 0;
   
   	/**
   	 * Connection URL. Overrides host, port, and password. User is ignored. Example:
   	 * redis://user:password@example.com:6379
   	 */
   	private String url;
   
   	/**
   	 * Redis server host.
   	 */
   	private String host = "localhost";
   
   	/**
   	 * Login password of the redis server.
   	 */
   	private String password;
   
   	/**
   	 * Redis server port.
   	 */
   	private int port = 6379;
   
   ```

3. LettuceConnectionConfiguration,注入ClientResources和LettuceConnectionFactory，JedisConnectionConfiguration也是注入一个JedisConnectionFactory

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass(RedisClient.class)
   class LettuceConnectionConfiguration extends RedisConnectionConfiguration {
   
   	LettuceConnectionConfiguration(RedisProperties properties,
   			ObjectProvider<RedisSentinelConfiguration> sentinelConfigurationProvider,
   			ObjectProvider<RedisClusterConfiguration> clusterConfigurationProvider) {
   		super(properties, sentinelConfigurationProvider, clusterConfigurationProvider);
   	}
   
   	@Bean(destroyMethod = "shutdown")
   	@ConditionalOnMissingBean(ClientResources.class)
   	DefaultClientResources lettuceClientResources() {
   		return DefaultClientResources.create();
   	}
   
   	@Bean
   	@ConditionalOnMissingBean(RedisConnectionFactory.class)
   	LettuceConnectionFactory redisConnectionFactory(
   			ObjectProvider<LettuceClientConfigurationBuilderCustomizer> builderCustomizers,
   			ClientResources clientResources) throws UnknownHostException {
   		LettuceClientConfiguration clientConfig = getLettuceClientConfiguration(builderCustomizers, clientResources,
   				getProperties().getLettuce().getPool());
   		return createLettuceConnectionFactory(clientConfig);
   	}
   //....
   }
   ```

4. RedisTemplate

   ```java
   public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
   //序列化器
   	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer keySerializer = null;
   	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer valueSerializer = null;
   	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashKeySerializer = null;
   	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashValueSerializer = null;
   	private RedisSerializer<String> stringSerializer = RedisSerializer.string();
   
   	//lua脚本执行器
       	private @Nullable ScriptExecutor<K> scriptExecutor;
   //操作执行者
      	private final ValueOperations<K, V> valueOps = new DefaultValueOperations<>(this);
   	private final ListOperations<K, V> listOps = new DefaultListOperations<>(this);
   	private final SetOperations<K, V> setOps = new DefaultSetOperations<>(this);
   	private final StreamOperations<K, ?, ?> streamOps = new DefaultStreamOperations<>(this, new ObjectHashMapper());
   	private final ZSetOperations<K, V> zSetOps = new DefaultZSetOperations<>(this);
   	private final GeoOperations<K, V> geoOps = new DefaultGeoOperations<>(this);
   	private final HyperLogLogOperations<K, V> hllOps = new DefaultHyperLogLogOperations<>(this);
   	private final ClusterOperations<K, V> clusterOps = new DefaultClusterOperations<>(this);
       
       @Override
   	public void afterPropertiesSet() {
           //...
       if (defaultSerializer == null) {
   //默认序列化器
                   defaultSerializer = new JdkSerializationRedisSerializer(
                           classLoader != null ? classLoader : this.getClass().getClassLoader());
               }
           //...
       }
   
   }
   ```


### 缓存

#### JSR107

JSR是Java Specification Requests 的缩写 ，Java规范请求，故名思议提交Java规范，大家一同遵守这个规范的话，会让大家‘沟通’起来更加轻松， JSR-107呢就是关于如何使用缓存的规范。

写过缓存应用的童鞋都会知道，使用缓存大概有以下步骤

1. 向电脑申请一块空间作为缓存

2. 为缓存定义你自己的数据结构

3. 向缓存中写数据

4. 从缓存中读数据

5. 不再使用缓存时，清空你锁申请的内存空间

大概这么多吧，当然里面还有很多细节性的东西，比如过期设置呀，分布式设置呀，是不是要持久化呀，是不是要支持事务呀，要不要加锁呀.......

JSR-107呢就是对缓存常用的操作做了一个抽象，然后给出一个API接口，不同的缓存产品只要实现了这些接口就可以了。使用缓存的用户能也只要调用这些接口就能得到不同产品的缓存服务，而不用悲催的来来回回学习不同缓存的API，更加悲催的是API还没有看明白，某技术就已经黄了。

javaCaching定义了五个核心接口：CacheProvider，CacheManager，Cache，Entry（记录） 和 Expiry。，

1. CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可 以在运行期访问多个CachingProvider。
2. CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache 存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
3. Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个 CacheManager所拥有。
4. Entry是一个存储在Cache中的key-value对。
5. Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期 的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。
   

![lNhqfO.png](https://s2.ax1x.com/2020/01/03/lNhqfO.png)

使用javaCache需要依赖的包是：

```xml
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
```

#### Spring缓存抽象

Spring定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术。并支持使用JCache（JSR-107）注解简化开发。

Cache接口为缓存的组件规范定义，包含缓存的各种操作集合。

Cache接口下Spring提供了各种xxCache的实现；如RedisCache、EnCacheCache、ConcurrentMapCache。

![lN5JG8.png](https://s2.ax1x.com/2020/01/03/lN5JG8.png)

##### 几个重要概念&缓存注解

![lNhvXd.png](https://s2.ax1x.com/2020/01/03/lNhvXd.png)

![lN4S0I.png](https://s2.ax1x.com/2020/01/03/lN4S0I.png)

**@Cacheable**

@Cacheable可以标记在一个方法上，也可以标记在一个类上。当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。对于一个支持缓存的方法，Spring会在其被调用后将其返回值缓存起来，以保证下次利用同样的参数来执行该方法时可以直接从缓存中获取结果，而不需要再次执行该方法。Spring在缓存方法的返回值时是以键值对进行缓存的，值就是方法的返回结果，至于键的话，Spring又支持两种策略，默认策略和自定义策略，这个稍后会进行说明。需要注意的是当一个支持缓存的方法在对象内部被调用时是不会触发缓存功能的。@Cacheable可以指定三个属性，value(Cache名称)、key(自定义key)和condition。

1. value属性是必须指定的，其表示当前方法的返回值是会被缓存在哪个Cache上的，对应Cache的名称。其可以是一个Cache也可以是多个Cache，当需要指定多个Cache时其是一个数组。

2. key属性是用来指定Spring缓存方法的返回结果时对应的key的。该属性支持SpringEL表达式。当我们没有指定该属性时，Spring将使用默认策略生成key。默认使用参数作为key

   如果要动态拼接key也可以，key="#root.methodName+'['+#参数属性名+']'"会被拼接为方法名【参数】

   condition自定义策略是指我们可以通过Spring的EL表达式来指定我们的key。这里的EL表达式可以使用方法参数及它们对应的属性。使用方法参数时我们可以直接使用“#参数名”或者“#p参数index”。

   除了上述使用方法参数作为key之外，Spring还为我们提供了一个root对象可以用来生成key。通过该root对象我们可以获取到以下信息。

   ![lN59KJ.png](https://s2.ax1x.com/2020/01/03/lN59KJ.png)

3. keyGenerator：key的生成器，可以自己指定key生成器的组件id

   key和keyGenerator：二选一使用

4. condition属性指定发生的条件 

   有的时候我们可能并不希望缓存一个方法所有的返回结果。通过condition属性可以实现这一功能。condition属性默认为空，表示将缓存所有的调用情形。其值是通过SpringEL表达式来指定的，当为true时表示进行缓存处理；当为false时表示不进行缓存处理，即每次调用该方法时该方法都会执行一次。如下示例表示只有当user的id为偶数时才会进行缓存。

   ```
   @Cacheable(value={“users”}, key=”#user.id”, condition=”#user.id%2==0”)
   ```

5. cacheManager/cacheResolver指定缓存管理器，因为缓存管理器里面的cache有些是存在ConcurrentHashMap里面的，有些事存在Redis里面的。

   ![lN5VPK.png](https://s2.ax1x.com/2020/01/03/lN5VPK.png)

6. unless：否定缓存，指定条件为true，则不缓存，与condition相反

7. sync：是否使用异步模式，不能同时和unless使用

**@CachePut**

在支持Spring Cache的环境下，对于使用@Cacheable标注的方法，Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中。@CachePut也可以声明一个方法支持缓存功能。与@Cacheable不同的是使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。

**@CacheEvict**

@CacheEvict是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。@CacheEvict可以指定的属性有value、key、condition、allEntries和beforeInvocation。其中value、key和condition的语义与@Cacheable对应的属性类似。即value表示清除操作是发生在哪些Cache上的（对应Cache的名称）；key表示需要清除的是哪个key，如未指定则会使用默认策略生成的key；condition表示清除操作发生的条件。

下面我们来介绍一下新出现的两个属性allEntries和beforeInvocation。

- allEntries是boolean类型，表示是否需要清除缓存中的所有元素。默认为false，表示不需要。当指定了allEntries为true时，Spring Cache将忽略指定的key。有的时候我们需要Cache一下清除所有的元素，这比一个一个清除元素更有效率。

- 清除操作默认是在对应方法成功执行之后触发的，即方法如果因为抛出异常而未能成功返回时也不会触发清除操作。使用beforeInvocation可以改变触发清除操作的时间，当我们指定该属性值为true时，Spring会在调用该方法之前清除缓存中的指定元素。

**@Caching**

@Caching注解可以让我们在一个方法或者类上同时指定多个Spring Cache相关的注解。其拥有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict。

```java
@Caching(cacheable = @Cacheable(“users”), evict = { @CacheEvict(“cache2”),

         @CacheEvict(value = “cache3”, allEntries = true) })

   public User find(Integer id) {

      returnnull;

   }
```

**使用自定义注解**

 Spring允许我们在配置可缓存的方法时使用自定义的注解，前提是自定义的注解上必须使用对应的注解进行标注。如我们有如下这么一个使用@Cacheable进行标注的自定义注解。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Cacheable(value=”users”)
public @interface MyCacheable {
}
```

那么在我们需要缓存的方法上使用@MyCacheable进行标注也可以达到同样的效果。

```
@MyCacheable
   public User findById(Integer id) {
      System.out.println(“find user by id: “ + id);
      User user = new User();
      user.setId(id);
      user.setName(“Name” + id);
      return user;
   }
```

#### 示例

```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.200</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```

配置

```yml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:data
    username: root
    password:
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
mybatis:
  #  mapper-locations: mapper/*.xml
  type-aliases-package: com.hyp.learn.cache.entity
  configuration:
    #自动驼峰命名规则( camel case )映射
    map-underscore-to-camel-case: true

server:
  servlet:
    context-path: /cache
```

sql文件

```sql
-- 部门表
create table if not exists tbl_dept
(
  id        integer primary key auto_increment,
  dept_name varchar(50)
);

-- 员工表
create table if not exists tbl_employee
(
  id        integer primary key auto_increment,
  last_name varchar(50),
  email     varchar(50),
  gender    varchar(5),
  d_id      integer,
  CONSTRAINT fk_e_e FOREIGN KEY (d_id) REFERENCES tbl_dept (id)

);

-- 添加data



insert into tbl_dept
values (1, '开发部门'),
       (2, '销售部门'),
       (3, '售后部门');
insert into tbl_employee
values (1, 'lisa', '123@123.com', '0', 1),
       (2, 'tom', '123@456.com', '1', 2),
       (3, 'lisa2', '123@123.com', '0', 3),
       (4, 'lisa3', '123@123.com', '1', 1),
       (5, 'lisa4', '123@123.com', '0', 2),
       (6, 'lisa5', '123@123.com', '0', 3),
       (7, 'lisa6', '123@123.com', '1', 1),
       (8, 'lisa7', '123@123.com', '1', 2),
       (9, 'lisa8', '123@123.com', '0', 3),
       (10, 'lisa9', '123@123.com', '1', 1),
       (11, 'lisa10', '123@123.com', '0', 2),
       (12, 'lisa11', '123@123.com', '1', 3),
       (13, 'lisa12', '123@123.com', '0', 1);

```

实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Alias("Employee")
public class Employee implements Serializable {

    private Integer id;
    private String lastName;
    private String email;
    private String gender;
}
```

dao类

```java
@Mapper
public interface EmployeeMapper {


    @Select("select * from tbl_employee where id = #{id}")
    public Employee getEmpById(Integer id);

    @Insert("Insert into tbl_employee(lastName,email,gender,d_id) Values(#{lastName},#{email},#{gender},#{dId}) ")
    public Long addEmp(Employee employee);

    @Update("Update tbl_employee set lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{dId}) where id=#{id}")
    public boolean updateEmp(Employee employee);

    @Delete("delete tbl_employee where id =#{id}")
    public boolean deleteEmpById(Integer id);
        @Select("select * from tbl_employee where last_name like #{name} limit 0,1")
    public Employee getEmpByName(String name);
}

```

service类

```java
@Service
//缓存公共配置
@CacheConfig(cacheNames = "px_employee")
public class EmployeeService {
    @Autowired
    private EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存，以后再要相同的调用参数，直接从缓存中取出，不用调用方法
     * 先调用缓存，再可能执行方法，因此不能在Cacheable使用#result
     * 1.cacheName/value：指定cache组件的名称，可以指定单/多个：
     * key：要按SpEL表达式去写，
     */
//    @Cacheable(cacheNames = "px_employee",  keyGenerator = "myKeyGenerator",condition = "#a0>1", unless = "#result == null")
    @Cacheable(cacheNames = "px_employee")
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "员工");
        return employeeMapper.getEmpById(id);
    }

    /**
     * @CachePut:及调用方法，有更新缓存数据
     * 先调用目标方法，再更新缓存
     key要和Cacheable中的key相同才能更新同一缓存
     * @param employee
     * @return
     */
    @CachePut(cacheNames = "px_employee",key = "#result.id")
    public Employee update(Employee employee){

        System.out.println("update" + employee + "员工");

        employeeMapper.updateEmp(employee);
         return employee;
    }
       /**
     *清除操作默认是在对应方法成功执行之后触发的
     * @param id
     */
    @CacheEvict(cacheNames = "px_employee",beforeInvocation = true)
    public void delete(Integer id)
    {
        System.out.println("删除" + id + "员工");
        employeeMapper.deleteEmpById(id);
    }
    
        @Caching(
            cacheable = {
                    @Cacheable(cacheNames = "px_employee", key = "#name")
            },
            put = {
                    @CachePut(cacheNames = "px_employee", key = "#result.id"),
                    @CachePut(cacheNames = "px_employee", key = "#result.email")
            }
    )
    public Employee getEmpByName(String name) {
        return employeeMapper.getEmpByName(name);
    }
}
```

controller类

```java
@RestController
@RequestMapping("/emp")
public class EmploeeController {
    @Autowired
    private EmployeeService employeeService;

     @GetMapping("/{id}")
    public Employee getEmp(@PathVariable("id") Integer id) {
        return employeeService.getEmp(id);
    }

    @PostMapping("/")
    public Employee update(@RequestBody Employee employee) {
        employeeService.update(employee);
        return employee;
    }

    @GetMapping("/del/{id}")
    public String delete(@PathVariable Integer id) {
        employeeService.delete(id);
        return "删除成功";
    }


    @GetMapping("/name/{name}")
    public Employee getEmpByName(@PathVariable("name") String name) {
        return employeeService.getEmpByName(name);
    }
}
```

运行类上添加`@EnableCaching`注解 

```java
@SpringBootApplication
@MapperScan("com.hyp.learn.cache.dao")
@EnableCaching
public class CacheApplication {


    public static void main(String[] args) {

        // devtools：是spring boot的一个热部署工具
        //设置 spring.devtools.restart.enabled 属性为false，可以关闭该特性.
        //System.setProperty("spring.devtools.restart.enabled","false");

        // 启动Sprign Boot
        ConfigurableApplicationContext ctx = SpringApplication.run(CacheApplication.class);

        System.out.println("Let's inspect the beans provided by Spring Boot:");

        String[] beanNames = ctx.getBeanDefinitionNames();
        Arrays.sort(beanNames);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }

    //自定义KeyGenerator
    @Bean(name = "myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+Arrays.asList(params).toString() +"]";
            }
        };
    }
}

```

运行测试缓存是否生效

#### 缓存原理

自动配置类位置：org.springframework.boot.autoconfigure.cache

1. CacheAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   //生效条件
   @ConditionalOnClass(CacheManager.class)
   @ConditionalOnBean(CacheAspectSupport.class)
   @ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
   //配置属性类
   @EnableConfigurationProperties(CacheProperties.class)
   //在这些自动配置完成之后进行配置
   @AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
   		HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
   //实际配置的类
   @Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })
   public class CacheAutoConfiguration {
   
   	@Bean
   	@ConditionalOnMissingBean
   	public CacheManagerCustomizers cacheManagerCustomizers(ObjectProvider<CacheManagerCustomizer<?>> customizers) {
   		return new CacheManagerCustomizers(customizers.orderedStream().collect(Collectors.toList()));
   	}
   
   	@Bean
   	public CacheManagerValidator cacheAutoConfigurationValidator(CacheProperties cacheProperties,
   			ObjectProvider<CacheManager> cacheManager) {
   		return new CacheManagerValidator(cacheProperties, cacheManager);
   	}
   
   	@ConditionalOnClass(LocalContainerEntityManagerFactoryBean.class)
   	@ConditionalOnBean(AbstractEntityManagerFactoryBean.class)
   	static class CacheManagerEntityManagerFactoryDependsOnPostProcessor
   			extends EntityManagerFactoryDependsOnPostProcessor {
   
   		CacheManagerEntityManagerFactoryDependsOnPostProcessor() {
   			super("cacheManager");
   		}
   
   	}
       
       /**
        
   	 * {@link ImportSelector} to add {@link CacheType} configuration classes.
   	 */
   	static class CacheConfigurationImportSelector implements ImportSelector {
   
   		@Override
   		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
   			CacheType[] types = CacheType.values();
   			String[] imports = new String[types.length];
   			for (int i = 0; i < types.length; i++) {
   				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
   			}
   			return imports;
   		}
   
   	}
       
   }
   ```

2. CacheConfigurationImportSelector导入cache的配置bean

   ![lN5WL9.png](https://s2.ax1x.com/2020/01/03/lN5WL9.png)

   可以看到支持nop（无）、simple、JSR107、ehcacha、hazelcast、Infinispan、Couchbase、Redis、Caffeine等缓存的配置方式，默认是SimpleCacheConfiguration配置

3. SimpleCacheConfiguration注册ConcurrentMapCacheManager

   ```
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnMissingBean(CacheManager.class)
   @Conditional(CacheCondition.class)
   class SimpleCacheConfiguration {
   
   	@Bean
   	ConcurrentMapCacheManager cacheManager(CacheProperties cacheProperties,
   			CacheManagerCustomizers cacheManagerCustomizers) {
   		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
   		List<String> cacheNames = cacheProperties.getCacheNames();
   		if (!cacheNames.isEmpty()) {
   			cacheManager.setCacheNames(cacheNames);
   		}
   		return cacheManagerCustomizers.customize(cacheManager);
   	}
   
   }
   ```

4. ConcurrentMapCacheManager可以获取和缓存ConcurrentMapCache的缓存组件，

   ```java
   public class ConcurrentMapCacheManager implements CacheManager, BeanClassLoaderAware {
   //那么程序会先把这个cache加载出来：
   //（1.）判断这个cache是否存在
   //（2.）如果不存在就加锁再次确认一遍
   //（3.）还是不存在就创建ConcurrentMapCache类型的cache，并put到concurrentMap中
       //缓存组件没有就创建
   	@Override
   	@Nullable
   	public Cache getCache(String name) {
   		Cache cache = this.cacheMap.get(name);
   		if (cache == null && this.dynamic) {
   			synchronized (this.cacheMap) {
   				cache = this.cacheMap.get(name);
   				if (cache == null) {
   					cache = createConcurrentMapCache(name);
   					this.cacheMap.put(name, cache);
   				}
   			}
   		}
   		return cache;
   	}
   }
   //创建缓存组件
   	protected Cache createConcurrentMapCache(String name) {
   		SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
   		return new ConcurrentMapCache(name, new ConcurrentHashMap<>(256),
   				isAllowNullValues(), actualSerialization);
   
   	}
   
   ```

5. ConcurrentMapCache，ConcurrentMapCache内部使用ConcurrentHashMap缓存数据，ConcurrentHashMap是线程安全的Map类

   ```java
   	@Override
   	@Nullable
   	public <T> T get(Object key, Callable<T> valueLoader) {
   		return (T) fromStoreValue(this.store.computeIfAbsent(key, k -> {
   			try {
   				return toStoreValue(valueLoader.call());
   			}
   			catch (Throwable ex) {
   				throw new ValueRetrievalException(key, valueLoader, ex);
   			}
   		}));
   	}
   
   ```

6. CacheAspectSupport缓存执行方法

   ```java
   //1. 获取缓存组件
   //2.查看是否有缓存击中，有则返回，否则执行方法，获取需要缓存的值并放入缓存
   @Nullable
   	private Object execute(final CacheOperationInvoker invoker, Method method, CacheOperationContexts contexts) {
   	// Special handling of synchronized invocation
   		if (contexts.isSynchronized()) {
   			CacheOperationContext context = contexts.get(CacheableOperation.class).iterator().next();
   			if (isConditionPassing(context, CacheOperationExpressionEvaluator.NO_RESULT)) {
                   //key生成策略
   				Object key = generateKey(context, CacheOperationExpressionEvaluator.NO_RESULT);
   				Cache cache = context.getCaches().iterator().next();
   				try {
   					return wrapCacheValue(method, cache.get(key, () -> unwrapReturnValue(invokeOperation(invoker))));
   				}
   				catch (Cache.ValueRetrievalException ex) {
   					// The invoker wraps any Throwable in a ThrowableWrapper instance so we
   					// can just make sure that one bubbles up the stack.
   					throw (CacheOperationInvoker.ThrowableWrapper) ex.getCause();
   				}
   			}
   			else {
   				// No caching required, only call the underlying method
   				return invokeOperation(invoker);
   			}
   		}
   
   
   		// Process any early evictions
   		processCacheEvicts(contexts.get(CacheEvictOperation.class), true,
   				CacheOperationExpressionEvaluator.NO_RESULT);
   
   		// Check if we have a cached item matching the conditions
   		Cache.ValueWrapper cacheHit = findCachedItem(contexts.get(CacheableOperation.class));
   
   		// Collect puts from any @Cacheable miss, if no cached item is found
   		List<CachePutRequest> cachePutRequests = new LinkedList<>();
   		if (cacheHit == null) {
   			collectPutRequests(contexts.get(CacheableOperation.class),
   					CacheOperationExpressionEvaluator.NO_RESULT, cachePutRequests);
   		}
   
   		Object cacheValue;
   		Object returnValue;
   
   		if (cacheHit != null && !hasCachePut(contexts)) {
   			// If there are no put requests, just use the cache hit
   			cacheValue = cacheHit.get();
   			returnValue = wrapCacheValue(method, cacheValue);
   		}
   		else {
   			// Invoke the method if we don't have a cache hit
   			returnValue = invokeOperation(invoker);
   			cacheValue = unwrapReturnValue(returnValue);
   		}
   
   		// Collect any explicit @CachePuts
           //结果放入缓存
   		collectPutRequests(contexts.get(CachePutOperation.class), cacheValue, cachePutRequests);
   
   		// Process any collected put requests, either from @CachePut or a @Cacheable miss
   		for (CachePutRequest cachePutRequest : cachePutRequests) {
   			cachePutRequest.apply(cacheValue);
   		}
   
   		// Process any late evictions
   		processCacheEvicts(contexts.get(CacheEvictOperation.class), false, cacheValue);
   
   		return returnValue;
   	}
   ```

#### 集成Redis缓存中间件

查看配置类`org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration`,发现只要引入redis的starter就可以使用该配置，如果还有其他缓存中间件，配置指定`spring.cache.type=redis`

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {

	@Bean
	RedisCacheManager cacheManager(CacheProperties cacheProperties, CacheManagerCustomizers cacheManagerCustomizers,
			ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
			ObjectProvider<RedisCacheManagerBuilderCustomizer> redisCacheManagerBuilderCustomizers,
			RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(
				determineConfiguration(cacheProperties, redisCacheConfiguration, resourceLoader.getClassLoader()));
		List<String> cacheNames = cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		redisCacheManagerBuilderCustomizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
		return cacheManagerCustomizers.customize(builder.build());
	}

	private org.springframework.data.redis.cache.RedisCacheConfiguration determineConfiguration(
			CacheProperties cacheProperties,
			ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
			ClassLoader classLoader) {
		return redisCacheConfiguration.getIfAvailable(() -> createConfiguration(cacheProperties, classLoader));
	}

	private org.springframework.data.redis.cache.RedisCacheConfiguration createConfiguration(
			CacheProperties cacheProperties, ClassLoader classLoader) {
		Redis redisProperties = cacheProperties.getRedis();
		org.springframework.data.redis.cache.RedisCacheConfiguration config = org.springframework.data.redis.cache.RedisCacheConfiguration
				.defaultCacheConfig();
		config = config.serializeValuesWith(
				SerializationPair.fromSerializer(new JdkSerializationRedisSerializer(classLoader)));
		if (redisProperties.getTimeToLive() != null) {
			config = config.entryTtl(redisProperties.getTimeToLive());
		}
		if (redisProperties.getKeyPrefix() != null) {
			config = config.prefixKeysWith(redisProperties.getKeyPrefix());
		}
		if (!redisProperties.isCacheNullValues()) {
			config = config.disableCachingNullValues();
		}
		if (!redisProperties.isUseKeyPrefix()) {
			config = config.disableKeyPrefix();
		}
		return config;
	}

}
```

默认情况下redis会连接`localhost:6379`,直接运行即可

#### 共享 Session

分布式系统中，Session 共享有很多的解决方案，其中托管到缓存中应该是最常用的方案之一，

Spring Session provides an API and implementations for managing a user’s session information.

Spring Session 提供了一套创建和管理 Servlet HttpSession 的方案。Spring Session 提供了集群 Session（Clustered Sessions）功能，默认采用外置的 Redis 来存储 Session 数据，以此来解决 Session 共享的问题。

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.session</groupId>
       <artifactId>spring-session-data-redis</artifactId>
   </dependency>
   ```

2. Session 配置

   ```
   @Configuration
   @EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
   public class SessionConfig {
   }
   ```

   > maxInactiveIntervalInSeconds: 设置 Session 失效时间，使用 Redis Session 之后，原 Spring Boot 的 server.session.timeout 属性不再生效。

3. 测试

   添加测试方法获取 sessionid

   ```
   @RequestMapping("/uid")
   String uid(HttpSession session) {
       UUID uid = (UUID) session.getAttribute("uid");
       if (uid == null) {
           uid = UUID.randomUUID();
       }
       session.setAttribute("uid", uid);
       return session.getId();
   }
   ```

   登录 Redis 输入 `keys '*sessions*'`

   ```
   t<spring:session:sessions:db031986-8ecc-48d6-b471-b137a3ed6bc4
   t(spring:session:expirations:1472976480000
   ```

   其中 1472976480000 为失效时间，意思是这个时间后 Session 失效，`db031986-8ecc-48d6-b471-b137a3ed6bc4` 为 sessionId,登录 http://localhost:8080/uid 发现会一致，就说明 Session 已经在 Redis 里面进行有效的管理了。

4. 如何在两台或者多台中共享 Session

   其实就是按照上面的步骤在另一个项目中再次配置一次，启动后自动就进行了 Session 共享。



#### 参考

1. [负载均衡集群中的session解决方案](https://blog.51cto.com/zhibeiwang/1965018)
2. [Redis的两个典型应用场景](http://emacoo.cn/blog/spring-redis)
3. [SpringBoot应用之分布式会话](https://segmentfault.com/a/1190000004358410)