---
title: Spring boot 数据访问 
date: 2019-06-30 18:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

在工作中离不开数据库，无论是关系型数据库还是NoSQL，都对我们异常重要，当然他们的工作原理和使用方法不尽相同，如果有一种框架可以使我们对数据库的使用方法变得更加方便的话，那就可以说是在Spring boot环境下的Spring Data框架集合，可以说是开箱即用，下面我们将进一步了解。

版本2.2.2.RELEASE

<!--more-->

第一步，引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!--测试框架-->
        <!-- spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--H2数据库-->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
```

第二步，配置application.yml文件

```yml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:data
    username: root
    password:
```

第三步，直接使用(此时数据库已经配置好，但是是内存数据库，而且并未初始化任何结构和数据)

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = DataApplication.class)
public class AppTest {

    @Autowired
    private DataSource dataSource;

    /**
     * Rigorous Test :-)
     */
    @Test
    public void shouldAnswerWithTrue() throws SQLException {
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
}
```

查看输出：

```
HikariProxyConnection@560858993 wrapping conn0: url=jdbc:h2:mem:data user=ROOT
```

发现使用的是HikariCP数据库连接池,可以从`org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration`内发现其配置DataSource的bean方法（这里使用了懒汉式加载方法，使用静态内部类，只有在使用道内部类时进行加载），可以看到其注释的Conditional，意思就是（1）当HikariDataSource在classpath，就是引入这个类；（2）并没有配置DataSource的示例bean；（3）matchIfMissing=true指定第三个条件非必要条件，条件就是`spring.datasource.type=com.zaxxer.hikari.HikariDataSource`

```java
    @Configuration(
        proxyBeanMethods = false
    )
    @ConditionalOnClass({HikariDataSource.class})
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"},
        havingValue = "com.zaxxer.hikari.HikariDataSource",
        matchIfMissing = true
    )
static class Hikari {
        Hikari() {
        }

        @Bean
        @ConfigurationProperties(
            prefix = "spring.datasource.hikari"
        )
        HikariDataSource dataSource(DataSourceProperties properties) {
            HikariDataSource dataSource = (HikariDataSource)DataSourceConfiguration.createDataSource(properties, HikariDataSource.class);
            if (StringUtils.hasText(properties.getName())) {
                dataSource.setPoolName(properties.getName());
            }

            return dataSource;
        }
    }
```

另外该类中还有其他的数据库连接池：dbcp2、tomcat、自身的datasource(非连接池)。

查看maven依赖

![l35d8s.png](https://s2.ax1x.com/2019/12/31/l35d8s.png)

就可以发现以下几点：

1. 默认引入HikariCP数据库连接池，支持上面的现象
2. 引入spring-jdbc和spring-tx，支持数据库事务
3. 使用logback日志，装饰器模式

再观察一下`org.springframework.boot.autoconfigure.jdbc`下的相关类

1. DataSourceAutoConfiguration，

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
   //配置属性解析类
   @EnableConfigurationProperties(DataSourceProperties.class)
   @Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
   public class DataSourceAutoConfiguration {
   
       //嵌入的数据库属性配置，import注解
       //EmbeddedDataSourceConfiguration--》DataSourceProperties
       //
       //数据库属性类中发现有使用到EmbeddedDatabaseConnection，枚举类型
   	@Configuration(proxyBeanMethods = false)
   	@Conditional(EmbeddedDatabaseCondition.class)
   	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
   	@Import(EmbeddedDataSourceConfiguration.class)
   	protected static class EmbeddedDatabaseConfiguration {
   
   	}
   
       //导入数据源
   	@Configuration(proxyBeanMethods = false)
   	@Conditional(PooledDataSourceCondition.class)
   	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
   	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
   			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
   			DataSourceJmxConfiguration.class })
   	protected static class PooledDataSourceConfiguration {
   
   	}
       //。。。。。
   }
   ```

2. EmbeddedDataSourceConfiguration会对嵌入式数据库进初始化一个EmbeddedDatabase，本身继承BeanClassLoaderAware，会在类加载完毕后进行执行

   ```java
   @Configuration(proxyBeanMethods = false)
   @EnableConfigurationProperties(DataSourceProperties.class)
   public class EmbeddedDataSourceConfiguration implements BeanClassLoaderAware {
   
   	private ClassLoader classLoader;
   
   	@Override
   	public void setBeanClassLoader(ClassLoader classLoader) {
   		this.classLoader = classLoader;
   	}
   	
   	@Bean(destroyMethod = "shutdown")
   	public EmbeddedDatabase dataSource(DataSourceProperties properties) {
   		return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseConnection.get(this.classLoader).getType())
   				.setName(properties.determineDatabaseName()).build();
   	}
   
   }
   ```

3. EmbeddedDatabaseConnection嵌入式数据库的连接参数，从DataSourceProperties得知，默认用户名sa,默认数据库名testdb，默认密码为空字符串。默认配置起作用情况为当引入h2、derby、hsqldb的jar时。

   ```java
   //支持的嵌入式数据库的默认连接参数
   public enum EmbeddedDatabaseConnection {
       NONE((EmbeddedDatabaseType)null, (String)null, (String)null),
       H2(EmbeddedDatabaseType.H2, "org.h2.Driver", "jdbc:h2:mem:%s;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE"),
       DERBY(EmbeddedDatabaseType.DERBY, "org.apache.derby.jdbc.EmbeddedDriver", "jdbc:derby:memory:%s;create=true"),
       HSQL(EmbeddedDatabaseType.HSQL, "org.hsqldb.jdbcDriver", "jdbc:hsqldb:mem:%s");
       //。。。
   }
   ```

   测试一下：

   把上面示例的配置文件注释掉，再重新运行程序，输出：

   ```
   HikariProxyConnection@2046652309 wrapping conn0: url=jdbc:h2:mem:testdb user=SA
   ```

4. DataSourceProperties可以从里面看出datasource配置的所有情况，每个属性都对应`spring.datasource`的配置项，其中schema属性和data属性，可以在程序运行时指定执行的sql文件。先运行schema再运行data

   ```yml
   #示例
   #默认执行文件为resources文件夹下的名为schema-${platform}.sql文件或者schema.sql，对应的数据data-${platform}.sql文件或者data.sql
   #platform是 spring.datasource.platform 的值，比如，可以将它设置为数据库的供应商名称（ hsqldb , h2 , oracle , mysql , postgresql 等），默认为all
   #能通过设置spring.datasource.continue-on-error的值来控制是否继续。
   spring:
     datasource:
       driver-class-name: org.h2.Driver
       url: jdbc:h2:mem:data
       username: root
       password:
       platform: all
       #如果给的sql文件名称符合默认要求，则不用指定
       schema: 
       	- schema-all.sql
       	- schema.sql
       #指定运行schema文件时的用户名和密码
       schema-username: root
       schema-password: root
       #如果给的sql文件名称符合默认要求，则不用指定
       data: 
       	- data-all.sql
       	- data.sql
       #指定运行schema文件时的用户名和密码
       data-username: hyp
       data-password: 123456
   ```

   注意，会在每次运行程序是都会执行，所以如果只执行一次的话，记得执行完毕后取消设置

   可以在`org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder.addDefaultScripts()`方法附加看到其赋值，

5. DataSourceInitializer上面指定sql文件的使用方法会在这里定义,其实就是执行runScripts方法

   ```java
   //查找方法
   private List<Resource> getScripts(String propertyName, List<String> resources, String fallback) {
   		if (resources != null) {
   			return getResources(propertyName, resources, true);
   		}
   		String platform = this.properties.getPlatform();
   		List<String> fallbackResources = new ArrayList<>();
   		fallbackResources.add("classpath*:" + fallback + "-" + platform + ".sql");
   		fallbackResources.add("classpath*:" + fallback + ".sql");
   		return getResources(propertyName, fallbackResources, false);
   	}
   
   
   //schema运行建表文件
   boolean createSchema() {
   		List<Resource> scripts = getScripts("spring.datasource.schema", this.properties.getSchema(), "schema");
   		if (!scripts.isEmpty()) {
   			if (!isEnabled()) {
   				logger.debug("Initialization disabled (not running DDL scripts)");
   				return false;
   			}
   			String username = this.properties.getSchemaUsername();
   			String password = this.properties.getSchemaPassword();
               //执行
   			runScripts(scripts, username, password);
   		}
   		return !scripts.isEmpty();
   	}
   
   //data运行数据初始化文件
   	void initSchema() {
   		List<Resource> scripts = getScripts("spring.datasource.data", this.properties.getData(), "data");
   		if (!scripts.isEmpty()) {
   			if (!isEnabled()) {
   				logger.debug("Initialization disabled (not running data scripts)");
   				return;
   			}
   			String username = this.properties.getDataUsername();
   			String password = this.properties.getDataPassword();
               //执行
   			runScripts(scripts, username, password);
   		}
   	}
   
   ```

6. DataSourceInitializerInvoker从名字得知是DataSourceInitializer的使用类,内部实现监听器和InitializingBean接口

   ```java
   class DataSourceInitializerInvoker implements ApplicationListener<DataSourceSchemaCreatedEvent>, InitializingBean {
   	private DataSourceInitializer dataSourceInitializer;
       private boolean initialized;
   
       //在初始化bean的时候都会执行该方法。先调用afterPropertieSet()方法，然后再调用init-method中指定的方法
   	@Override
   	public void afterPropertiesSet() {
   		DataSourceInitializer initializer = getDataSourceInitializer();
   		if (initializer != null) {
               //执行创建sql
   			boolean schemaCreated = this.dataSourceInitializer.createSchema();
   			if (schemaCreated) {
   				initialize(initializer);
   			}
   		}
   	}
       
       private void initialize(DataSourceInitializer initializer) {
   		try {
   			this.applicationContext.publishEvent(new DataSourceSchemaCreatedEvent(initializer.getDataSource()));
               
   			// The listener might not be registered yet, so don't rely on it.
                       //如果未初始化
   			if (!this.initialized) {
                               //执行初始化数据sql
   				this.dataSourceInitializer.initSchema();
   				this.initialized = true;
   			}
   		}
   		catch (IllegalStateException ex) {
   			logger.warn(LogMessage.format("Could not send event to complete DataSource initialization (%s)",
   					ex.getMessage()));
   		}
   	}
       
       //运行在收到DataSourceSchemaCreatedEvent事件之后
       @Override
   	public void onApplicationEvent(DataSourceSchemaCreatedEvent event) {
   		// NOTE the event can happen more than once and
   		// the event datasource is not used here
   		DataSourceInitializer initializer = getDataSourceInitializer();
           //如果未初始化
   		if (!this.initialized && initializer != null) {
              //执行初始化数据sql
   			initializer.initSchema();
   			this.initialized = true;
   		}
   	}
       
   }
   ```

7. JdbcTemplateAutoConfiguration

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
   //只有一个DataSource
   @ConditionalOnSingleCandidate(DataSource.class)
   //在DataSourceAutoConfiguration配置完成后再执行JdbcTemplateAutoConfiguration
   @AutoConfigureAfter(DataSourceAutoConfiguration.class)
   //相关配置类
   @EnableConfigurationProperties(JdbcProperties.class)
   //引入jdbcTemplate相关配置，
   @Import({ JdbcTemplate相关配置Configuration.class, NamedParameterJdbcTemplateConfiguration.class })
   public class JdbcTemplateAutoConfiguration {
   
   }
   
   
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnMissingBean(JdbcOperations.class)
   class JdbcTemplateConfiguration {
   
   	@Bean
   	@Primary
   	JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
   		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
   		JdbcProperties.Template template = properties.getTemplate();
   		jdbcTemplate.setFetchSize(template.getFetchSize());
   		jdbcTemplate.setMaxRows(template.getMaxRows());
   		if (template.getQueryTimeout() != null) {
   			jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
   		}
   		return jdbcTemplate;
   	}
   
   }
   
   //NamedParameterJdbcTemplate支持具名参数
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnSingleCandidate(JdbcTemplate.class)
   @ConditionalOnMissingBean(NamedParameterJdbcOperations.class)
   class NamedParameterJdbcTemplateConfiguration {
   
   	@Bean
   	@Primary
   	NamedParameterJdbcTemplate namedParameterJdbcTemplate(JdbcTemplate jdbcTemplate) {
   		return new NamedParameterJdbcTemplate(jdbcTemplate);
   	}
   
   }
   ```

知道自动注入JdbcTemplate，再做个小测试

1. 修改配置文件

   ```yml
   spring:
     datasource:
       driver-class-name: org.h2.Driver
       url: jdbc:h2:mem:data
       username: root
       password:
       schema: classpath:db/schema.sql
       data: classpath:db/data.sql
   ```

2. sql文件

   ```sql
   -- schema.sql
   
   create TABLE if not exists tb_user
   (
     id       integer primary key auto_increment,
     username varchar(50),
     birthday date,
     sex      varchar(5),
     address  varchar(100)
   );
   
   
   -- data.sql
   -- 添加data
   
   insert into tb_user
   values (1, '张三', '2000-2-5', '男', '地球'),
          (2, '小红', '2001-2-22', '女', '火星');
   
   ```

3. 修改测试方法

   ```
   
       @Autowired
       private JdbcTemplate jdbcTemplate;
       @Test
       public void testQuery() {
           List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from tb_user");
           list.forEach(System.out::println);
       }
   ```

4. 运行结果

   ```
   {ID=1, USERNAME=张三, BIRTHDAY=2000-02-05, SEX=男, ADDRESS=地球}
   {ID=2, USERNAME=小红, BIRTHDAY=2001-02-22, SEX=女, ADDRESS=火星}
   ```

我们这里使用的是h2数据库内存模式，所以会在程序结束运行后删除数据库，因此不用在运行后去除schema和data设置

#### 整合Druid数据库连接池

[Druid Spring Boot Starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter) 用于帮助你在Spring Boot项目中轻松集成Druid数据库连接池和监控。

加入依赖

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

修改配置文件

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
```

在第一个测试方法中间打上一个断点，再次debug运行第一个测试

![l8iFBT.png](https://s2.ax1x.com/2019/12/31/l8iFBT.png)

##### 监控

引入依赖

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>com.github.drtrang</groupId>
            <artifactId>druid-spring-boot2-autoconfigure</artifactId>
            <version>1.1.10</version>
        </dependency>
```

配置文件

```yml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:data
    username: root
    password:
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      max-wait: 30000
      query-timeout: 10
      validation-query: SELECT 1
      use-global-data-source-stat: true
      # 默认开启，当前已开启
      stat:
        log-slow-sql: true
        slow-sql-millis: 1000
      # 默认关闭，需手动开启，当前已开启
      slf4j:
        enabled: true
        data-source-log-enabled: false
        connection-log-enabled: false
        statement-log-enabled: false
        result-set-log-enabled: false
      # 默认关闭，需手动开启，当前已开启
      wall:
        enabled: true
        log-violation: true
        throw-exception: false
        config:
          delete-where-none-check: true
      # 默认关闭，需手动开启，当前已关闭
      config:
        enabled: false
      # 默认关闭，需手动开启，当前已关闭
      web-stat:
        enabled: false
      # 默认关闭，需手动开启，当前已关闭
      aop-stat:
        enabled: false
      # 默认关闭，需手动开启，当前已关闭
      stat-view-servlet:
        enabled: false
  transaction:
    rollback-on-commit-failure: true
  mvc:
    servlet:
      load-on-startup: 1
  aop:
    auto: true
    proxy-target-class: true
  http:
    encoding:
      enabled: true
      force: true
      charset: utf-8
    converters:
      preferred-json-mapper: jackson
  jackson:
    default-property-inclusion: non_null
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    serialization:
      indent_output: true
      write_null_map_values: true
      write_dates_as_timestamps: false
    deserialization:
      fail_on_unknown_properties: false
    parser:
      allow_unquoted_control_chars: true
      allow_single_quotes: true
logging:
  level:
    root: info
    org.springframework: info
    org.springframework.jdbc: debug
    org.springframework.boot: info
    org.mybatis: debug
    com.alibaba.druid: debug
    com.github.trang: debug
    executableSql: debug
server:
  servlet:
    context-path: /data
```

配置servlet和filter

```java
@Configuration
public class DruidConfig {
    //配置监控
    //1.配置管理后台的servlet
    @Bean
    public ServletRegistrationBean<StatViewServlet> statViewServlet() {

        ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(
                new StatViewServlet(), "/druid/*"
        );

        //配置初始化参数
        //com.alibaba.druid.support.http.StatViewServlet
//        com.alibaba.druid.support.http.ResourceServlet
        Map<String, String> data = new HashMap<>();
        data.put("loginUsername", "admin");
        data.put("loginPassword", "123456");
        registrationBean.setInitParameters(data);


        return registrationBean;
    }

    //2.配置一个监控的filter
    @Bean
    public FilterRegistrationBean<WebStatFilter> webStatFilter() {
        FilterRegistrationBean<WebStatFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new WebStatFilter());

        //配置初始化参数
//        com.alibaba.druid.support.http.WebStatFilter
        Map<String, String> data = new HashMap<>();
        data.put("exclusions", "*.js,*.css,/druid/*");

        bean.setInitParameters(data);

        bean.setUrlPatterns(Collections.singletonList("/*"));
        return bean;
    }
}
```

创建一个Controller

```java
@RestController
@RequestMapping("/main")
public class MainController {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @GetMapping("/query")
    public List query() {
        List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from tb_user");
        list.forEach(System.out::println);
        return list;
    }
}
```

运行程序，访问<http://127.0.0.1:8080/data/druid/login.html>,输入用户名admin，密码123456

![l8Avq0.png](https://s2.ax1x.com/2020/01/01/l8Avq0.png)

这样就算完成了监控功能，更多配置参考官网

#### 整合Mybatis

mybatis是一个强大的持久化框架

引入依赖

```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

```

![l8Exld.png](https://s2.ax1x.com/2020/01/01/l8Exld.png)

会自动导入mybatis、mybatis-spring、spring-boot-starter-jdbc、mybatis-spring-boot-autoconfigure

按照惯例，我们先来看看MybatisAutoConfiguration等自动配置类

```java
@Configuration
//需要引入的类
@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class})
@ConditionalOnSingleCandidate(DataSource.class)
//配置类MybatisProperties
@EnableConfigurationProperties({MybatisProperties.class})
//需要在这些配置类完成之后执行
@AutoConfigureAfter({DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class})
public class MybatisAutoConfiguration implements InitializingBean {

}
```

MybatisAutoConfiguration会在DataSourceAutoConfiguration和MybatisLanguageDriverAutoConfiguration配置完成后执行，DataSourceAutoConfiguration上面已经分析过了，MybatisLanguageDriverAutoConfiguration则是为了执行Mybatis的脚本语言提供的驱动注入，内部有freemarker、velocity、thymeleaf框架的驱动。

MybatisAutoConfiguration默认会进行SqlSessionFactory和SqlSessionTemplate的初始化，另外支持下面方式指定指定mapper文件

```yml
mybatis:
  mapper-locations: classpath:mapping/*Mapper.xml
```

##### 示例

在上面的示例中基础上进行更改

配置文件增加

```yml
mybatis:
  mapper-locations: mapper/*.xml
  type-aliases-package: com.hyp.learn.data.entity
  configuration:
    #自动驼峰命名规则( camel case )映射
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl
    log-prefix: executableSql.
```

数据库文件

```sql
-- db/schema.sql

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

-- db/data.sql

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

增加对应额实体类

```java
//com.hyp.learn.data.entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Employee implements Serializable {

    private Integer id;
    private String lastName;
    private String email;
    private String gender;
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Department implements Serializable {

    private Integer id;
    private String departmentName;
    private List<Employee> emps;
    
}
```

增加dao接口

```java
//com.hyp.learn.data.dao
public interface DepartmentMapper {

    public Department getDeptById(Integer id);

    public Department getDeptByIdStep(Integer id);
}

public interface EmployeeMapper {

    //返回一条记录的map；key就是列名，值就是对应的值
    public Map<String, Object> getEmpByIdReturnMap(Integer id);

    public List<Employee> getEmpsByLastNameLike(String lastName);

    public Employee getEmpByMap(Map<String, Object> map);

    public Employee getEmpByIdAndLastName(@Param("id") Integer id, @Param("lastName") String lastName);
        
    public List<Employee> getEmpsByDeptId(Integer id);


    public Employee getEmpById(Integer id);

    public Long addEmp(Employee employee);

    public boolean updateEmp(Employee employee);

    public boolean deleteEmpById(Integer id);
}
```

对应的mapper文件

```xml
<!-- DepartmentMapper.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hyp.learn.data.dao.DepartmentMapper">


    <!--public Department getDeptById(Integer id);  -->
    <select id="getDeptById" resultType="Department">
        select id, dept_name departmentName
        from tbl_dept
        where id = #{id}
    </select>


    <!-- collection：分段查询 -->
    <resultMap type="Department" id="MyDeptStep">
        <id column="id" property="id"/>
        <id column="dept_name" property="departmentName"/>
        <collection property="emps"
                    select="com.hyp.learn.data.dao.EmployeeMapper.getEmpsByDeptId"
                    column="{deptId=id}" fetchType="lazy"/>
    </resultMap>
    <!-- public Department getDeptByIdStep(Integer id); -->
    <select id="getDeptByIdStep" resultMap="MyDeptStep">
        select id, dept_name
        from tbl_dept
        where id = #{id}
    </select>

</mapper>


<!-- EmployeeMapper.xml-->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hyp.learn.data.dao.EmployeeMapper">

    <!--public Map<String, Object> getEmpByIdReturnMap(Integer id);  -->
    <select id="getEmpByIdReturnMap" resultType="map">
        select *
        from tbl_employee
        where id = #{id}
    </select>

    <!-- public List<Employee> getEmpsByLastNameLike(String lastName); -->
    <!--resultType：如果返回的是一个集合，要写集合中元素的类型  -->
    <select id="getEmpsByLastNameLike" resultType="Employee">
        select *
        from tbl_employee
        where last_name like #{lastName}
    </select>

    <select id="getEmpsByDeptId" resultType="Employee">
        select *
        from tbl_employee
        where d_id = #{deptId}
    </select>

    <!-- public Employee getEmpByMap(Map<String, Object> map); -->
    <select id="getEmpByMap" resultType="Employee">
        select *
        from ${tableName}
        where id = ${id}
          and last_name = #{lastName}
    </select>

    <!--  public Employee getEmpByIdAndLastName(Integer id,String lastName);-->
    <select id="getEmpByIdAndLastName" resultType="Employee">
        select *
        from tbl_employee
        where id = #{id}
          and last_name = #{lastName}
    </select>

    <select id="getEmpById" resultType="Employee">
        select *
        from tbl_employee
        where id = #{id}
    </select>


    <insert id="addEmp" parameterType="Employee"
            useGeneratedKeys="true" keyProperty="id" databaseId="h2">
        insert into tbl_employee(last_name, email, gender)
        values (#{lastName}, #{email}, #{gender})
    </insert>


    <!-- public void updateEmp(Employee employee);  -->
    <update id="updateEmp">
        update tbl_employee
        set last_name=#{lastName},
            email=#{email},
            gender=#{gender}
        where id = #{id}
    </update>

    <!-- public void deleteEmpById(Integer id); -->
    <delete id="deleteEmpById">
        delete
        from tbl_employee
        where id = #{id}
    </delete>
</mapper>
```

##### 多数据源

说起多数据源，一般都来解决那些问题呢，主从模式或者业务比较复杂需要连接不同的分库来支持业务。我们遇到的情况是后者，网上找了很多，大都是根据 Jpa 来做多数据源解决方案，要不就是老的 Spring 多数据源解决方案，还有的是利用 Aop 动态切换，感觉有点小复杂，其实我只是想找一个简单的多数据支持而已，折腾了两个小时整理出来，供大家参考。

以 Mybatis Xml 版本为例，给大家展示如何如何配置多数据源。

**sql文件**

```mysql
-- ----------------------------
-- Table structure for `users`
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `userName` varchar(32) DEFAULT NULL COMMENT '用户名',
  `passWord` varchar(32) DEFAULT NULL COMMENT '密码',
  `user_sex` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=28 DEFAULT CHARSET=utf8;
```

**配置文件**

```properties
mybatis.config-location=classpath:mybatis/mybatis-config.xml

spring.datasource.test1.jdbc-url=jdbc:mysql://localhost:3306/test1?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.test1.username=root
spring.datasource.test1.password=root
spring.datasource.test1.driver-class-name=com.mysql.cj.jdbc.Driver

spring.datasource.test2.jdbc-url=jdbc:mysql://localhost:3306/test2?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.test2.username=root
spring.datasource.test2.password=root
spring.datasource.test2.driver-class-name=com.mysql.cj.jdbc.Driver
```

一个 test1 库和一个 test2 库，其中 test1 位主库，在使用的过程中必须指定主库，不然会报错。

**数据源配置**

```java
@Configuration
@MapperScan(basePackages = "com.neo.mapper.test1", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
public class DataSource1Config {

    @Bean(name = "test1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.test1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "test1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

最关键的地方就是这块了，一层一层注入,首先创建 DataSource，然后创建 SqlSessionFactory 再创建事务，最后包装到 SqlSessionTemplate 中。其中需要指定分库的 mapper 文件地址，以及分库dao层代码

```
@MapperScan(basePackages = "com.neo.mapper.test1", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
```

这块的注解就是指明了扫描 dao 层，并且给 dao 层注入指定的 SqlSessionTemplate。所有`@Bean`都需要按照命名指定正确。

**dao 层和 xml层**

dao 层和 xml 需要按照库来分在不同的目录，比如：test1 库 dao 层在 `com.neo.mapper.test1` 包下，test2 库在`com.neo.mapper.test2`

```
public interface User1Mapper {
	List<UserEntity> getAll();
	UserEntity getOne(Long id);
	void insert(UserEntity user);
	void update(UserEntity user);
	void delete(Long id);
}
```

xml 层

```xml
<mapper namespace="com.neo.mapper.test1.User1Mapper" >
    <resultMap id="BaseResultMap" type="com.neo.model.User" >
        <id column="id" property="id" jdbcType="BIGINT" />
        <result column="userName" property="userName" jdbcType="VARCHAR" />
        <result column="passWord" property="passWord" jdbcType="VARCHAR" />
        <result column="user_sex" property="userSex" javaType="com.neo.enums.UserSexEnum"/>
        <result column="nick_name" property="nickName" jdbcType="VARCHAR" />
    </resultMap>
    
    <sql id="Base_Column_List" >
        id, userName, passWord, user_sex, nick_name
    </sql>

    <select id="getAll" resultMap="BaseResultMap"  >
       SELECT 
       <include refid="Base_Column_List" />
     FROM users
    </select>

    <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
        SELECT 
       <include refid="Base_Column_List" />
     FROM users
     WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="com.neo.model.User" >
       INSERT INTO 
          users
          (userName,passWord,user_sex) 
        VALUES
          (#{userName}, #{passWord}, #{userSex})
    </insert>
    
    <update id="update" parameterType="com.neo.model.User" >
       UPDATE 
          users 
       SET 
        <if test="userName != null">userName = #{userName},</if>
        <if test="passWord != null">passWord = #{passWord},</if>
        nick_name = #{nickName}
       WHERE 
          id = #{id}
    </update>
    
    <delete id="delete" parameterType="java.lang.Long" >
       DELETE FROM
           users 
       WHERE 
           id =#{id}
    </delete>

</mapper>
```

**测试**

测试可以使用 SpringBootTest,也可以放到 Controller中，这里只贴 Controller 层的使用

```java
@RestController
public class UserController {

    @Autowired
    private User1Mapper user1Mapper;

	@Autowired
	private User2Mapper user2Mapper;
	
	@RequestMapping("/getUsers")
	public List<UserEntity> getUsers() {
		List<UserEntity> users=user1Mapper.getAll();
		return users;
	}
	
    @RequestMapping("/getUser")
    public UserEntity getUser(Long id) {
    	UserEntity user=user2Mapper.getOne(id);
        return user;
    }
    
    @RequestMapping("/add")
    public void save(UserEntity user) {
        user2Mapper.insert(user);
    }
    
    @RequestMapping(value="update")
    public void update(UserEntity user) {
        user2Mapper.update(user);
    }
    
    @RequestMapping(value="/delete/{id}")
    public void delete(@PathVariable("id") Long id) {
        user1Mapper.delete(id);
    }
    
}
```

Mybatis 注解版本配置多数据源和 Xml 版本基本一致



#### 整合Spring data JPA

Spring Data 项目的目的是为了简化构建基于 Spring 框架应用的数据访问技术,包括非关系数据库、Map-Reduce 框架、云数据服务等等;另外也包含对关系数据库的访问支持。

SpringData为我们提供使用统一的API来对数据访问层进行操作;这主要是Spring DataCommons项目来实现的。Spring Data Commons让我们在使用关系型或者非关系型数据访问技术时都基于Spring提供的统一标准,标准包含了CRUD(创建、获取、更新、删除)、查询、排序和分页的相关操作。

统一的Repository接口：

```
Repository<T, ID extends Serializable>:统一接口
RevisionRepository<T, ID extends Serializable, N extends Number & Comparable<N>>:基于乐观锁机制
CrudRepository<T, ID extends Serializable>:基本CRUD操作
PagingAndSortingRepository<T, ID extends Serializable>:基本CRUD及分页
```

提供数据访问模板类：如:MongoTemplate、RedisTemplate等

JPA与Spring Data

- JpaRepository基本功能：编写接口继承JpaRepository既有crud及分页等基本功能
- 定义符合规范的方法命名：在接口中只需要声明符合规范的方法,即拥有对应的功能
- @Query自定义查询,定制查询SQL
- Specifications查询(Spring Data JPA支持JPA2.0的Criteria查询)

![l8n8V1.png](https://s2.ax1x.com/2020/01/01/l8n8V1.png)

引入依赖

```xml
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```

![l8usOJ.png](https://s2.ax1x.com/2020/01/01/l8usOJ.png)

可以看到默认使用的Hibernate来实现JPA标准

1. 查看jpa的自动配置类，位于`org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration`

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnBean(DataSource.class)
   //当引入JpaRepository类时
   @ConditionalOnClass(JpaRepository.class)
   //不存在下列bean
   @ConditionalOnMissingBean({ JpaRepositoryFactoryBean.class, JpaRepositoryConfigExtension.class })
   //具有下面的配置，非必须
   @ConditionalOnProperty(prefix = "spring.data.jpa.repositories", name = "enabled", havingValue = "true",
   		matchIfMissing = true)
   //引入Jpa的注册者
   @Import(JpaRepositoriesRegistrar.class)
   //在着HibernateJpa和TaskExecution配置完成之后
   @AutoConfigureAfter({ HibernateJpaAutoConfiguration.class, TaskExecutionAutoConfiguration.class })
   public class JpaRepositoriesAutoConfiguration {
   	@Bean
   	@Conditional(BootstrapExecutorCondition.class)
   	public EntityManagerFactoryBuilderCustomizer entityManagerFactoryBootstrapExecutorCustomizer(
   			Map<String, AsyncTaskExecutor> taskExecutors) {
   		return (builder) -> {
   			AsyncTaskExecutor bootstrapExecutor = determineBootstrapExecutor(taskExecutors);
   			if (bootstrapExecutor != null) {
   				builder.setBootstrapExecutor(bootstrapExecutor);
   			}
   		};
   	}
   
   	private AsyncTaskExecutor determineBootstrapExecutor(Map<String, AsyncTaskExecutor> taskExecutors) {
   		if (taskExecutors.size() == 1) {
   			return taskExecutors.values().iterator().next();
   		}
   		return taskExecutors.get(TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME);
   	}
   
   	private static final class BootstrapExecutorCondition extends AnyNestedCondition {
   
   		BootstrapExecutorCondition() {
   			super(ConfigurationPhase.REGISTER_BEAN);
   		}
   
   		@ConditionalOnProperty(prefix = "spring.data.jpa.repositories", name = "bootstrap-mode",
   				havingValue = "deferred", matchIfMissing = false)
   		static class DeferredBootstrapMode {
   
   		}
   
   		@ConditionalOnProperty(prefix = "spring.data.jpa.repositories", name = "bootstrap-mode", havingValue = "lazy",
   				matchIfMissing = false)
   		static class LazyBootstrapMode {
   
   		}
   
   	}
   
   }
   ```

2. HibernateJpaAutoConfiguration，Hibernate的相关注入

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class, SessionImplementor.class })
   @EnableConfigurationProperties(JpaProperties.class)
   @AutoConfigureAfter({ DataSourceAutoConfiguration.class })
   @Import(HibernateJpaConfiguration.class)
   public class HibernateJpaAutoConfiguration {
   
   }
   //实际配置管理类
   @Configuration(proxyBeanMethods = false)
   @EnableConfigurationProperties(HibernateProperties.class)
   @ConditionalOnSingleCandidate(DataSource.class)
   class HibernateJpaConfiguration extends JpaBaseConfiguration {
       
   }
   //配置属性类
   @ConfigurationProperties("spring.jpa.hibernate")
   public class HibernateProperties {
   }
   ```

3. TaskExecutionAutoConfiguration,线程池自动配置

   ```java
   @ConditionalOnClass(ThreadPoolTaskExecutor.class)
   @Configuration(proxyBeanMethods = false)
   @EnableConfigurationProperties(TaskExecutionProperties.class)
   public class TaskExecutionAutoConfiguration {
   
   }
   
   //相关属性类
   @ConfigurationProperties("spring.task.execution")
   public class TaskExecutionProperties {
   }
   ```

4. JpaRepositoriesRegistrar实际自动注入Jpa Repositories的类，内部声明带有@EnableJpaRepositories注解的静态内部类，自动支持Jpa，不需要再配置

   ```java
   class JpaRepositoriesRegistrar extends AbstractRepositoryConfigurationSourceSupport {
   //。。。。
   
   	@EnableJpaRepositories
   	private static class EnableJpaRepositoriesConfiguration {
   
   	}
   
   }
   ```

5. `org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration`为Hibernate JPA的ORM自动配置类

   ```java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class, SessionImplementor.class })
   //Jpa 相关属性
   @EnableConfigurationProperties(JpaProperties.class)
   @AutoConfigureAfter({ DataSourceAutoConfiguration.class })
   @Import(HibernateJpaConfiguration.class)
   public class HibernateJpaAutoConfiguration {
   
   }
   ```

6. JpaProperties为Jpa 相关属性

   ```java
   @ConfigurationProperties(prefix = "spring.jpa")
   public class JpaProperties {
   }
   ```

7. HibernateJpaConfiguration为Hibernate的配置类

   ```java
   @Configuration(proxyBeanMethods = false)
   //Jpa下的Hibernate配置
   @EnableConfigurationProperties(HibernateProperties.class)
   @ConditionalOnSingleCandidate(DataSource.class)
   class HibernateJpaConfiguration extends JpaBaseConfiguration {
   
   }
   ```

8. HibernateProperties为Hibernate配置

   ```
   @ConfigurationProperties("spring.jpa.hibernate")
   public class HibernateProperties {
   
   }
   ```

**示例**

配置文件

```yml
spring:
  jpa:
    hibernate:
      #根据实体类配置生成对数据库的修改
      ddl-auto: update
      naming:
        #数据库下划线《--》实体类驼峰
        physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
    #显示sql语句    
    show-sql: true

  jackson:
    #json驼峰转下划线
    property-naming-strategy: CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES
```

实体类

```java
@Data
//声明为实体类
@Entity
//配置表的属性，默认表名为实体类的名称单词首字符小写
@Table(name = "tb_user")
//@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public class User {
    @Id
    //自增主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column()
    private String lastName;

    @Column
    private String email;
}

```

数据访问接口

```java
//PagingAndSortingRepository和CrudRepository,<实体类，主键类型>
public interface UserReposity extends JpaRepository<User,Integer> {
    //按照特定名称的方法声明可以自动生成代理方法
    public User findFirstByEmail(String email);
}

```

测试类

```java
@RestController
public class UserController {
    @Autowired
    private UserReposity userReposity;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){
        Optional<User> user = userReposity.findById(id);
        return user.orElse(null);
    }

    @PostMapping("/user")
    public User insertUser(@RequestBody  User user){


        return userReposity.save(user);
    }

}

```

运行进行请求即可

#### 多数据源的支持

**同源数据库的多源支持**

日常项目中因为使用的分布式开发模式，不同的服务有不同的数据源，常常需要在一个项目中使用多个数据源，因此需要配置 Spring Boot Jpa 对多数据源的使用，一般分一下为三步：

- 1 配置多数据源
- 2 不同源的实体类放入不同包路径
- 3 声明不同的包路径下使用不同的数据源、事务支持

**异构数据库多源支持**

比如我们的项目中，即需要对 mysql 的支持，也需要对 Mongodb 的查询等。

实体类声明`@Entity` 关系型数据库支持类型、声明`@Document` 为 Mongodb 支持类型，不同的数据源使用不同的实体就可以了

```java
interface PersonRepository extends Repository<Person, Long> {
 …
}

@Entity
public class Person {
  …
}

interface UserRepository extends Repository<User, Long> {
 …
}

@Document
public class User {
  …
}
```

但是，如果 User 用户既使用 Mysql 也使用 Mongodb 呢，也可以做混合使用

```java
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}

@Entity
@Document
public class Person {
  …
}
```

也可以通过对不同的包路径进行声明，比如 A 包路径下使用 mysql,B 包路径下使用 MongoDB

```java
@EnableJpaRepositories(basePackages = "com.neo.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.neo.repositories.mongo")
interface Configuration { }
```

