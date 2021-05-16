---
title: Spring JDBC
date: 2019-05-22 19:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

Spring JDBC 是spring 官方提供的一个持久层框架，对jdbc进行了抽象和封装，消除了重复冗余的jdbc重复性的代码，使操作数据库变的更简单。

<!--more-->

但Spring JDBC本身并不是一个orm框架，与hibernate相比，它需要自己操作sql,手动映射字段关系，在保持灵活性的同时，势必造成了开发效率的降低。如果需要使用完整的orm框架操作数据库，可以使用hibernate或者spring Data Jpa。

Spring JDBC与现在市场上流行的 Hibernate、MyBatis 相比弱了很多，如果我们写一个小型的系统，用 SpringJDBC 还是很灵活的。

Spring JDBC 中的 JdbcTemplate类 在小型开发中还是比较常用的，而 NamedParameterJdbcTemplate、SimpleJdbcTemplate等这些在 3.X版本中就已经标记过了，在以后的版本中也可能会被抛弃或删除。我们本篇就对 JdbcTemplate类 进行讲解。

在使用 JdbcTemplate 小工具时，我们还需要依赖一个连接池，在本篇中选择经典的 C3P0连接池。（在这里我强调一下，数据源的意思是数据的来源，也就是数据库，而连接池是对数据源进行了绑定，它保存着连接数据源的相关信息，所以说数据源和连接池并不是一回事） 

#### 配置 C3P0连接池

c3p0 是现在比较流行的一款开源的 JDBC连接池，它主要用来保存连接数据源的相关信息。数据源这个词理解起来也比较容易，它就是数据的来源，也就是我们常说的数据库。

我们在使用 JdbcTemplate 小工具前必须要配置好一个连接池，也就是必须要获取到一个连接对象。

maven依赖：

```xml
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
 </dependency>
 <dependency>
     <groupId>com.mchange</groupId>
     <artifactId>c3p0</artifactId>
     <version>0.9.5.4</version>
 </dependency>
 <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-jdbc</artifactId>
     <version>${spring.version}</version>
 </dependency>
```

一般情况下，我们在配置连接池时都会将配置信息写入到外部属性文件，本小节只为演示，便不写入外部属性文件中。

1. 配置xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
           <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/learn"/>
           <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
           <property name="user" value="root"/>
           <property name="password" value="970603"/>
       </bean>
   </beans>
   ```

2. 测试

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration({"classpath:applicationContext11.xml"})
   public class Demo11Test {
       @Autowired
       private ApplicationContext context;
       @Test
       public void demo11() throws SQLException {
   
           DataSource dataSource = (DataSource) context.getBean("dataSource");
           Connection connection = dataSource.getConnection();
           System.out.println(connection);
       }
   }
   ```

#### 支持类

1. SqlParameterSource 简介

   可以使用SqlParameterSource实现作为来实现为命名参数设值，默认实现有 ：

   - MapSqlParameterSource实现非常简单，只是封装了java.util.Map；

   - BeanPropertySqlParameterSource封装了一个JavaBean对象，通过JavaBean对象属性来决定命名参数的值。

   - EmptySqlParameterSource 一个空的SqlParameterSource ，常用来占位使用

2. RowMapper简介

   这个接口为了实现sql查询结果和对象间的转换，可以自己实现，也可以使用系统实现，主要实现类有：

   - SingleColumnRowMapper ，sql结果为一个单列的数据，如List<String\> , List<Integer\>,String,Integer等

   - BeanPropertyRowMapper， sql结果匹配到对象 List< XxxVO> , XxxVO

3. JdbcTemplate主要提供以下五类方法：

   - execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句

   - update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句

   - batchUpdate方法用于执行批处理相关语句

   - query方法及queryForXXX方法：用于执行查询相关语句

   - call方法：用于执行存储过程、函数相关语句

#### 插入/修改/删除数据,使用updateXXX方法

**使用Map作为参数**

**API:** `int update(String sql, Map<String, ?> paramMap)`

示例：

```java
Map<String, Object> paramMap = new HashMap<>();
paramMap.put("id", UUID.randomUUID().toString());
paramMap.put("name", "小明");
paramMap.put("age", 33);
paramMap.put("homeAddress", "乐山");
paramMap.put("birthday", new Date());
template.update( 
     "insert into student(id,name,age,home_address,birthday) values (:id,:name,:age,:homeAddress,:birthday)",
     paramMap
);
```

**使用BeanPropertySqlParameterSource作为参数**

**API:**  `int update(String sql, SqlParameterSource paramSource)`

使用 BeanPropertySqlParameterSource作为参数

```java
public class StudentDTO{
    private String id;
    private String name;
    private String homeAddress;

     //getter,setter
}


```

使用：

```java
StudentDTO dto=new StudentDTO();//这个DTO为传入数据
dto.setId(UUID.randomUUID().toString());
dto.setName("小红");
dto.setHomeAddress("成都");
//------------------------------
template.update("insert into student(id,name,home_address) values (:id,:name,:homeAddress)",
                new BeanPropertySqlParameterSource(dto));
```

**使用MapSqlParameterSource 作为参数**

**API:** `int update(String sql, SqlParameterSource paramSource)`

使用 MapSqlParameterSource 作为参数

```java
MapSqlParameterSource mapSqlParameterSource = new MapSqlParameterSource()
        .addValue("id", UUID.randomUUID().toString())
        .addValue("name", "小王")
        .addValue("homeAddress", "美国");
template.update("insert into student(id,name,home_address) values    
                   (:id,:name,:homeAddress)",mapSqlParameterSource);
                
或者
Map<String, Object> paramMap = new HashMap<>();
paramMap.put("id", UUID.randomUUID().toString());
paramMap.put("name", "小明");
paramMap.put("homeAddress", "乐山");
//---------------
MapSqlParameterSource mapSqlParameterSource = new MapSqlParameterSource(paramMap);
```

#### 查询

**返回单行单列数据**

**API:**    `public < T > T queryForObject(String sql, Map<String, ?> paramMap, Class<T> requiredType)`

**API:**    `public < T > T queryForObject(String sql, SqlParameterSource paramSource, Class<T> requiredType)`

示例(注意EmptySqlParameterSource的使用)：

```java
Integer count = template.queryForObject(
                "select count(*) from student", new HashMap<>(), Integer.class);
String name = template.queryForObject( "select name from student where home_address  limit 1 ", EmptySqlParameterSource.INSTANCE, String.class); 
```

**返回 (多行)单列 数据**

**API:**    `public < T> List< T> queryForList(String sql, Map<String, ?> paramMap, Class< T > elementType)`

**API:**  `public < T> List< T> queryForList(String sql, SqlParameterSource paramSource, Class< T> elementType)` 

示例：

```java
List< String> namelist = template.queryForList("select name from student", new HashMap<>(), String.class);
```

**返回单行数据**

**API:**    `public < T> T queryForObject(String sql, Map< String, ?> paramMap, RowMapper< T>rowMapper)`

**API:**    `public < T> T queryForObject(String sql, SqlParameterSource paramSource, RowMapper< T> rowMapper)` 

示例：

```java
Student  stu = template.queryForObject(
                "select * from student limit 1", new HashMap<>(), new BeanPropertyRowMapper<Student>(Student.class));
//BeanPropertyRowMapper会把下划线转化为驼峰属性
//结果对象可比实际返回字段多或者少
```

注意：这两个API也可以使用SingleColumnRowMapper返回单行单列数据 

```java
String name = template.queryForObject(
                "select name from student limit 1", EmptySqlParameterSource.INSTANCE, new SingleColumnRowMapper<>(String.class));
```

**返回Map形式的单行数据**

**API:**    `public Map< String, Object> queryForMap(String sql, Map< String, ?> paramMap)`

**API:** `public  Map< String, Object> queryForMap(String sql, SqlParameterSource paramSource)` 

示例：

```java
 Map< String, Object> studentMap = template.queryForMap("select * from student limit 1", new HashMap<>());
```

**返回多行数据**

**API:**     `public < T> List< T> query(String sql, Map< String, ?> paramMap, RowMapper< T> rowMapper)` 

**API:**     `public < T> List< T> query(String sql, SqlParameterSource paramSource, RowMapper< T> rowMapper)` 

**API:**    `public < T> List< T> query(String sql, RowMapper< T> rowMapper)` 

示例：

```java
List< Student> studentList = template.query(
                "select * from student",  
                new BeanPropertyRowMapper<>(Student.class)
);        
```

同理，也可以使用SingleColumnRowMapper返回单行列表List< String>,List< Integer>等

**返回多行数据(Map)**

**API:**    `public List< Map< String, Object>> queryForList(String sql, Map< String, ?> paramMap)` 

**API:**        `public List< Map< String, Object>> queryForList(String sql, SqlParameterSource paramSource)`

示例：

```java
List<Map<String, Object>> mapList = template.queryForList(
                "select * from student", new HashMap<>());
```

**总结**

1. 开发中尽量使用NamedParameterJdbcTemplate代替JdbcTemplate，如果想使用JdbcTemplate，也可以通过NamedParameterJdbcTemplate#getJdbcOperations()获取
2. 不建议使用查询结构为Map的API

#### 模板类

Spring JDBC 提供了模板类对数据库简化对数据库的操作，其中JdbcTemplate是最常用的，如果需要使用命名参数可以使用NamedParameterJdbcTemplate，SimpleJdbcTemplate在3.1版本已经标记过时，在我使用的4.3版本中，已经被删除。

**JdbcTemplate 入门示例**

JdbcTemplate使用很简单，注入一个数据源就可以使用了,这里使用上面的配置

xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/learn"/>
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="user" value="root"/>
        <property name="password" value="970603"/>
    </bean>

    <bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:applicationContext11.xml"})
public class Demo11Test {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void demo11() throws SQLException {
        String sql = "select * from tb_user";
        List<Map<String, Object>> users = jdbcTemplate.queryForList(sql);
        System.out.println(users);

    }

}
```

对于参数赋值，可以采用占位符的方式

```java
 String sql = "select * from tb_user where id=?";
 List<Map<String, Object>> users = jdbcTemplate.queryForList(sql,1L);
 List<Map<String, Object>> users1 = jdbcTemplate.queryForList(sql, new Object[] {1L});
```

**mapper映射**

Spring JDBC 通过mapper接口把resultSet对象中的数据映射为java对象，例如上述例子中返回 `List<Map<String, Object>>`，其实使用的是`ColumnMapRowMapper`的mapper实现。我们自己可以通过实现`RowMapper`接口的方式自定义从resultSet到java对象的映射关系。

mysql用户表：

```mysql
CREATE TABLE `tb_user` (
  `id` bigint(12) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(128) NOT NULL,
  `passwd` varchar(255) NOT NULL COMMENT '密码',
  `gmt_create` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
  `gmt_modified` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表'  
```

User实体类:

```java
public class User implements Serializable{
    private BigDecimal id;
    private String name;
    private String passwd;
    private Timestamp create;
    private Timestamp modified;

    public User() {
    }

    public User(BigDecimal id, String name, String passwd, Timestamp create, Timestamp modified) {
        this.id = id;
        this.name = name;
        this.passwd = passwd;
        this.create = create;
        this.modified = modified;
    }

    public BigDecimal getId() {
        return id;
    }

    public void setId(BigDecimal id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPasswd() {
        return passwd;
    }

    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

    public Timestamp getCreate() {
        return create;
    }

    public void setCreate(Timestamp create) {
        this.create = create;
    }

    public Timestamp getModified() {
        return modified;
    }

    public void setModified(Timestamp modified) {
        this.modified = modified;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", passwd='" + passwd + '\'' +
                ", create=" + create +
                ", modified=" + modified +
                '}';
    }
}
```

自定义mapper映射

```java
@Test
    public void simpleMapperTest(){
        String sql = "select * from tb_user";
        List<User> users = jdbcTemplate.query(sql, new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setId(rs.getObject("id") == null ? null : rs.getBigDecimal("id"));
                user.setName(rs.getString("name"));
                user.setPasswd(rs.getString("passwd"));
                user.setCreate(rs.getTimestamp("gmt_create"));
                user.setModified(rs.getTimestamp("gmt_modified"));
                return user;
            }
        });
        System.out.println(users);
    }
```

**调用存储过程**

对于数据库数据的更新操作，可以直接调用update接口，如果是存储过程，可以通过execute完成调用。

```mysql
DROP PROCEDURE IF EXISTS test;
CREATE PROCEDURE test (IN userId BIGINT ( 20 ),OUT userName VARCHAR ( 200 ) ) 
BEGIN
SET userName = ( SELECT `name` FROM tb_user WHERE id = userId );
END;
```

测试：

```java
@Test
    public void proTest() {
        String proc = "{call test(?,?)}";
        String userName = jdbcTemplate.execute(new CallableStatementCreator() {
            public CallableStatement createCallableStatement(Connection con) throws SQLException {
                String proc = "{call test(?,?)}";
                CallableStatement cs = con.prepareCall(proc);
                cs.setLong(1, 1L);// 设置输入参数的值 索引从1开始
                cs.registerOutParameter(2, Types.VARCHAR);// 设置输出参数的类型
                return cs;
            }
        }, new CallableStatementCallback<String>() {
            public String doInCallableStatement(CallableStatement cs) throws SQLException, DataAccessException {
                cs.execute();
                return cs.getString(2);// 返回输出参数
            }
        });
        System.out.println(userName);
    }
```

**NamedParameterJdbcTemplate**

NamedParameterJdbcTemplate的使用基本上和JdbcTemplate类似，只不过参数的赋值方式由占位符变成了命名参数，命名参数优势在于，如果一个相同的参数出现了多次，只需要进行一次赋值即可。

创建NamedParameterJdbcTemplate对象的两种方式

```java
// 方式1
namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(jdbcTemplate);
// 方式2
namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
```

测试：

```
@Test
  public void namedParameterJdbcTemplateTest() {
    String sql = "select * from user where id =:id";
    Map<String, Object> parameters = new HashMap<String, Object>();
    parameters.put("id", 1L);
    List<Map<String, Object>> users = namedParameterJdbcTemplate.queryForList(sql, parameters);
    System.out.println(users);
  }
```

**batchUpdate**

对于大数据量的数据更新，可以采用batchUpdate接口

```
@Test
  public void batchUpdateTest() {
    String sql = "insert into user (user_name,password) VALUES (?, ?)";
    List<User> users = Lists.newArrayList();
    for (int i = 0; i <= 10; i++) {
      User user = new User();
      user.setUserName("xiaoming");
      user.setPassword("123456");
      users.add(user);
    }
    jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps, int i) throws SQLException {
        User user = users.get(i);
        int count = 0;
        ps.setString(++count, user.getUserName());// 索引从1开始
        ps.setString(++count, user.getPassword());
      }

      @Override
      public int getBatchSize() {
        return users.size();
      }
    });
  }
```

**注意事项**

关于queryForObject,如果没有查询到，或者查询到多个，都会抛异常，看一下源码

```java
  // 未查询到，或者查询到多个，都会抛异常
  public static <T> T requiredSingleResult(Collection<T> results) throws IncorrectResultSizeDataAccessException {
    int size = (results != null ? results.size() : 0);
    if (size == 0) {
      throw new EmptyResultDataAccessException(1);
    }
    if (results.size() > 1) {
      throw new IncorrectResultSizeDataAccessException(1, size);
    }
    return results.iterator().next();
  }
```

queryForMap最终走的也是queryForObject方法，因此，使用时也要注意

在自定义mapper获取ResultSet的值时，获取基本类型数据会有默认值，解决办法如下

```java
  Long id =  rs.getLong("id");//如果id为null,会默认0
  Long id1 =  rs.getObject("id") == null ? null : rs.getLong("id");//先判断是否为null
```

