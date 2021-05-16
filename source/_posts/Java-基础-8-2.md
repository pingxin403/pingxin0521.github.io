---
title: java 数据库连接池
date: 2019-05-15 21:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---

数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个。

<!--more-->

#### 为什么要使用连接池

 数据库连接是一种关键的有限的昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。  一个数据库连接对象均对应一个物理数据库连接，每次操作都打开一个物理连接，使用完都关闭连接，这样造成系统的 性能低下。 数据库连接池的解决方案是在应用程序启动时建立足够的数据库连接，并讲这些连接组成一个连接池(简单说：在一个“池”里放了好多半成品的数据库联接对象)，由应用程序动态地对池中的连接进行申请、使用和释放。对于多于连接池中连接数的并发请求，应该在请求队列中排队等待。并且应用程序可以根据池中连接的使用率，动态增加或减少池中的连接数。 连接池技术尽可能多地重用了消耗内存地资源，大大节省了内存，提高了服务器地服务效率，能够支持更多的客户服务。通过使用连接池，将大大提高程序运行效率，同时，我们可以通过其自身的管理机制来监视数据库连接的数量、使用情况等。 

#### 传统的连接机制与数据库连接池的运行机制区别

##### 不使用连接池流程:

下面以访问MySQL为例，执行一个SQL命令，如果不使用连接池，需要经过哪些流程。

![1.png](https://i.loli.net/2019/05/18/5ce009fce520645102.png)

不使用数据库连接池的步骤：

1. TCP建立连接的三次握手
2. MySQL认证的三次握手
3. 真正的SQL执行
4. MySQL的关闭
5. TCP的四次握手关闭

可以看到，为了执行一条SQL，却多了非常多我们不关心的网络交互。

**优点**：

1. 实现简单

**缺点**：

1. 网络IO较多
2. 数据库的负载较高
3. 响应时间较长及QPS较低
4. 应用频繁的创建连接和关闭连接，导致临时对象较多，GC频繁
5. 在关闭连接后，会出现大量TIME_WAIT 的TCP状态（在2个MSL之后关闭）

##### 使用连接池流程

![2.png](https://i.loli.net/2019/05/18/5ce009fd2ea2349796.png)

使用数据库连接池的步骤：

第一次访问的时候，需要建立连接。 但是之后的访问，均会**复用**之前创建的连接，直接执行SQL语句。

**优点：**

1. 较少了网络开销
2. 系统的性能会有一个实质的提升
3. 没了麻烦的TIME_WAIT状态

#### 工作原理

连接池的工作原理主要由三部分组成，分别为

1. 连接池的建立。一般在系统初始化时，连接池会根据系统配置建立，并在池中创建了几个连接对象，以便使用时能从连接池中获取。连接池中的连接不能随意创建和关闭，这样避免了连接随意建立和关闭造成的系统开销。Java中提供了很多容器类可以方便的构建连接池，例如Vector、Stack等。

2. 连接池的管理。连接池管理策略是连接池机制的核心，连接池内连接的分配和释放对系统的性能有很大的影响。其管理策略是：

   当客户请求数据库连接时，首先查看连接池中是否有空闲连接，如果存在空闲连接，则将连接分配给客户使用；如果没有空闲连接，则查看当前所开的连接数是否已经达到最大连接数，如果没达到就重新创建一个连接给请求的客户；如果达到就按设定的最大等待时间进行等待，如果超出最大等待时间，则抛出异常给客户。

   当客户释放数据库连接时，先判断该连接的引用次数是否超过了规定值，如果超过就从连接池中删除该连接，否则保留为其他客户服务。

   该策略保证了数据库连接的有效复用，避免频繁的建立、释放连接所带来的系统资源开销。

3. 连接池的释放。当应用程序退出时，关闭连接池中所有的连接，释放连接池相关的资源，该过程正好与创建相反。

#### 连接池主要参数

主要介绍一个主要参数的概念，不同的连接池的属性可能不一样。

1. **maxActive**  连接池支持的最大连接数，这里取值为20，表示同时最多有20个数据库连接。一般把maxActive设置成可能的并发量就行了设 0 为没有限制。

2. **maxIdle** 连接池中最多可空闲maxIdle个连接 ，这里取值为20，表示即使没有数据库连接时依然可以保持20空闲的连接，而不被清除，随时处于待命状态。设 0 为没有限制。

3. **minIdle** 连接池中最小空闲连接数，当连接数少于此值时，连接池会创建连接来补充到该值的数量

4. **initialSize** 初始化连接数目 

5. **maxWait** 连接池中连接用完时,新的请求等待时间,毫秒，这里取值-1，表示无限等待，直到超时为止，也可取值9000，表示9秒后超时。超过时间会出错误信息

6. **removeAbandoned**  是否清除已经超过“removeAbandonedTimout”设置的无效连接。如果值为“true”则超过“removeAbandonedTimout”设置的无效连接将会被清除。设置此属性可以从那些没有合适关闭连接的程序中恢复数据库的连接。

7. **removeAbandonedTimeout** 活动连接的最大空闲时间,单位为秒 超过此时间的连接会被释放到连接池中,*针对未被close的活动连接*

8. **minEvictableIdleTimeMillis** 连接池中连接可空闲的时间,单位为毫秒 *针对连接池中的连接对象*

9. **timeBetweenEvictionRunsMillis** / **minEvictableIdleTimeMillis**  每timeBetweenEvictionRunsMillis毫秒秒检查一次连接池中空闲的连接,把空闲时间超过minEvictableIdleTimeMillis毫秒的连接断开,直到连接池中的连接数到minIdle为止

#### 需要注意的点

1. 并发问题

   为了使连接管理服务具有最大的通用性，必须考虑多线程环境，即并发问题。这个问题相对比较好解决，因为各个语言自身提供了对并发管理的支持像java,c#等等，使用synchronized(java)lock(C#)关键字即可确保线程是同步的。使用方法可以参考，相关文献。

2. 事务处理

   我们知道，事务具有原子性，此时要求对数据库的操作符合“ALL-OR-NOTHING”原则,即对于一组SQL语句要么全做，要么全不做。 
   我们知道当2个线程共用一个连接Connection对象，而且各自都有自己的事务要处理时候，对于连接池是一个很头疼的问题，因为即使Connection类提供了相应的事务支持，可是我们仍然不能确定那个数据库操作是对应那个事务的，这是由于我们有２个线程都在进行事务操作而引起的。为此我们可以使用每一个事务独占一个连接来实现，虽然这种方法有点浪费连接池资源但是可以大大降低事务管理的复杂性。 

3. 连接池的分配与释放

   连接池的分配与释放，对系统的性能有很大的影响。合理的分配与释放，可以提高连接的复用度，从而降低建立新连接的开销，同时还可以加快用户的访问速度。 

   对于连接的管理可使用一个List。即把已经创建的连接都放入List中去统一管理。每当用户请求一个连接时，系统检查这个List中有没有可以分配的连接。如果有就把那个最合适的连接分配给他（如何能找到最合适的连接文章将在关键议题中指出）；如果没有就抛出一个异常给用户，List中连接是否可以被分配由一个线程来专门管理捎后我会介绍这个线程的具体实现。

4. 连接池的配置与维护

   连接池中到底应该放置多少连接，才能使系统的性能最佳？系统可采取设置最小连接数（minConnection）和最大连接数（maxConnection）等参数来控制连接池中的连接。比方说，最小连接数是系统启动时连接池所创建的连接数。如果创建过多，则系统启动就慢，但创建后系统的响应速度会很快；如果创建过少，则系统启动的很快，响应起来却慢。这样，可以在开发时，设置较小的最小连接数，开发起来会快，而在系统实际使用时设置较大的，因为这样对访问客户来说速度会快些。最大连接数是连接池中允许连接的最大数目，具体设置多少，要看系统的访问量，可通过软件需求上得到。 

   如何确保连接池中的最小连接数呢？有动态和静态两种策略。动态即每隔一定时间就对连接池进行检测，如果发现连接数量小于最小连接数，则补充相应数量的新连接,以保证连接池的正常运转。静态是发现空闲连接不够时再去检查。

#### 常用数据库连接池

1. [c3p0](https://github.com/swaldman/c3p0)

   C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate，Spring等。单线程，性能较差，适用于小型系统，代码600KB左右。**配置文件类型：properties文件、xml文件**

   c3p0与dbcp区别：dbcp没有自动回收空闲连接的功能；c3p0有自动回收空闲连接功能

2. [dbcp](http://commons.apache.org/proper/commons-dbcp/)

   DBCP（DataBase Connection Pool）数据库连接池，是Java数据库连接池的一种，由Apache开发，通过数据库连接池，可以让程序自动管理数据库连接的释放和断开。单独使用dbcp需要3个包：common-dbcp.jar,common-pool.jar,common-collections.jar，预先将数据库连接放在内存中，应用程序需要建立数据库连接时直接到连接池中申请一个就行，用完再放回。单线程，并发量低，性能不好，适用于小型系统。**配置文件类型：properties文件、xml文件**

3. [Tomcat Jdbc Pool](http://tomcat.apache.org/tomcat-8.5-doc/jdbc-pool.html)

   Tomcat在7.0以前都是使用common-dbcp做为连接池组件，但是dbcp是单线程，为保证线程安全会锁整个连接池，性能较差，dbcp有超过60个类，也相对复杂。Tomcat从7.0开始引入了新增连接池模块叫做Tomcat jdbc pool，基于Tomcat JULI，使用Tomcat日志框架，完全兼容dbcp，通过异步方式获取连接，支持高并发应用环境，超级简单核心文件只有8个，支持JMX，支持XA Connection。

4. [Druid](https://github.com/alibaba/druid)

   阿里出品，淘宝和支付宝专用数据库连接池，但它不仅仅是一个数据库连接池，它还包含一个ProxyDriver，一系列内置的JDBC组件库，一个SQL  Parser。支持所有JDBC兼容的数据库，包括Oracle、MySql、Derby、Postgresql、SQL Server、H2等等。

   DruidDataSource大部分属性都是参考DBCP的，如果你原来就是使用DBCP，迁移是十分方便的。

   **配置文件类型：xml文件,properties文件**

   Druid针对Oracle和MySql做了特别优化，比如Oracle的PS Cache内存占用优化，MySql的ping检测优化。

   Druid提供了MySql、Oracle、Postgresql、SQL-92的SQL的完整支持，这是一个手写的高性能SQL Parser，支持Visitor模式，使得分析SQL的抽象语法树很方便。

   简单SQL语句用时10微秒以内，复杂SQL用时30微秒。

   通过Druid提供的SQL Parser可以在JDBC层拦截SQL做相应处理，比如说分库分表、审计等。Druid防御SQL注入攻击的WallFilter就是通过Druid的SQL Parser分析语义实现的。

5. [HikariCP](https://github.com/brettwooldridge/HikariCP)

   HiKariCP是数据库连接池的一个后起之秀，号称性能最好，可以完美地PK掉其他连接池。**配置文件类型：properties文件、yaml文件**

#### 使用

数据库learn的tb_user表:

```
CREATE TABLE 
IF NOT EXISTS `tb_user` (
`id` bigint(12) UNSIGNED NOT NULL AUTO_INCREMENT,
`name` varchar(128) NOT  NULL UNIQUE,
`passwd` varchar(255) NOT  NULL COMMENT '密码',
`gmt_create` datetime(3) NOT  NULL default CURRENT_TIMESTAMP(3) comment '创建时间',
`gmt_modified` datetime(3) NOT  NULL default CURRENT_TIMESTAMP(3) on update CURRENT_TIMESTAMP(3) comment '修改时间',
PRIMARY KEY (`id`) 
)COMMENT="用户表";
```

数据库为MySQL 8.0.16 ，使用DBUtils框架，其中C3p0、Druid、HikariCP都是使用到slf4j日志框架，Maven依赖：

```
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.7</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.26</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.26</version>
    <scope>compile</scope>
</dependency>
```

测试类DBMain：

```
public class DBMain {
    public static void main(String[] args) {
        //保存数据
        List<User> l = null;
        //运行
        QueryRunner queryRunner = null;
        //连接
        Connection con = null;

        try {
            queryRunner = new QueryRunner();
            //从连接池获取连接
            con = C3P0Utils.getConnection();
            //查询数据，使用Bean链表处理类来处理返回数据
            l = queryRunner.query(con, "select * FROM tb_user", new BeanListHandler<User>(User.class) {
                @Override
                public List<User> handle(ResultSet rs) throws SQLException {
                    List<User> users = new ArrayList<>();
                    //如果有返回数据
                    while (rs.next()) {
                        User u = new User(rs.getLong("id"),
                                rs.getTimestamp("gmt_create"), rs.getTimestamp("gmt_modified"),
                                rs.getString("name"), rs.getString("passwd"));

                        users.add(u);
                        
                    }
                    return users;
                }
            });
            if (null != l) {
                for (User user : l) {
                    System.out.println(user);
                }
            } else {
                System.out.println("数据为空");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if (null != con) {
                try {
                    con.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

数据库java model类：

```
public class Domain {
    private Long id;
    private Date gmtCreate;
    private Date gmtModified;

    public Domain() {
    }

    public Domain(Long id, Date gmtCreate, Date gmtModified) {
        this.id = id;
        this.gmtCreate = gmtCreate;
        this.gmtModified = gmtModified;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Date getGmtCreate() {
        return gmtCreate;
    }

    public void setGmtCreate(Date gmtCreate) {
        this.gmtCreate = gmtCreate;
    }

    public Date getGmtModified() {
        return gmtModified;
    }

    public void setGmtModified(Date gmtModified) {
        this.gmtModified = gmtModified;
    }

    @Override
    public String toString() {
        return "Domain{" +
                "id=" + id +
                ", gmtCreate=" + gmtCreate +
                ", gmtModified=" + gmtModified +
                '}';
    }
}

```

User类

```
public class User extends Domain implements Serializable {
    private String name;
    private String passwd;

    public User() {
        super();
    }

    public User(String name, String passwd) {
        this.name = name;
        this.passwd = passwd;
    }

    public User(Long id, Date gmtCreate, Date gmtModified, String name, String passwd) {
        super(id, gmtCreate, gmtModified);
        this.name = name;
        this.passwd = passwd;
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


    @Override
    public String toString() {
        return "User{" +super.toString()+ ",name='" + name + '\'' +
                ", passwd='" + passwd + '\'' +
                '}';
    }

}
```



### DBCP

maven依赖：

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.6.0</version>
</dependency>

```

#### 配置

| 参数                          | 默认值         | 说明                                                         |
| ----------------------------- | -------------- | ------------------------------------------------------------ |
| username                      | \              | 传递给JDBC驱动的用于建立连接的用户名                         |
| password                      | \              | 传递给JDBC驱动的用于建立连接的密码                           |
| url                           | \              | 传递给JDBC驱动的用于建立连接的URL                            |
| driverClassName               | \              | 使用的JDBC驱动的完整有效的[Java](http://lib.csdn.net/base/javaee) 类名 |
| initialSize                   | 0              | 初始化连接:连接池启动时创建的初始化连接数量,1.2版本后支持    |
| maxActive                     | 8              | 最大活动连接:连接池在同一时间能够分配的最大活动连接的数量, 如果设置为非正数则表示不限制 |
| maxIdle                       | 8              | 最大空闲连接:连接池中容许保持空闲状态的最大连接数量,超过的空闲连接将被释放, 如果设置为负数表示不限制 |
| minIdle                       | 0              | 最小空闲连接:连接池中容许保持空闲状态的最小连接数量,低于这个数量将创建新的连接, 如果设置为0则不创建 |
| maxWait                       | 无限           | 最大等待时间:当没有可用连接时,连接池等待连接被归还的最大时间(以毫秒计数)超过时间则抛出异常,如果设置为-1表示无限等待 |
| testOnReturn                  | false          | 是否在归还到池中前进行检验                                   |
| testWhileIdle                 | false          | 连接是否被空闲连接回收器(如果有)进行检验.如果检测失败, 则连接将被从池中去除.设置为true后如果要生效,validationQuery参数必须设置为非空字符串 |
| minEvictableIdleTimeMillis    | 1000 * 60 * 30 | 连接在池中保持空闲而不被空闲连接回收器线程 (如果有)回收的最小时间值，单位毫秒 |
| numTestsPerEvictionRun        | 3              | 在每次空闲连接回收器线程(如果有)运行时检查的连接数量         |
| timeBetweenEvictionRunsMillis | -1             | 在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位.  如果设置为非正数,则不运行空闲连接回收器线程 |
| validationQuery               | null           | SQL查询,用来验证从连接池取出的连接,在将连接返回给调用者之前.如果指定, 则查询必须是一个SQL SELECT并且必须返回至少一行记录 |
| testOnBorrow                  | true           | 是否在从池中取出连接前进行检验,如果检验失败, 则从池中去除连接并尝试取出另一个。借出连接时不要测试，否则很影响性能。一定要配置，因为它的默认值是true。false表示每次从连接池中取出连接时，不需要执行validationQuery= "SELECT 1" 中的SQL进行测试。若配置为true,对性能有非常大的影响，性能会下降7-10倍。 |
| removeAbandoned               |                | 连接泄漏回收参数，当可用连接数少于3个时才执行                |
| removeAbandonedTimeout        |                | 连接泄漏回收参数，180秒，泄露的连接可以被删除的超时值        |

 **注意事项**

1. maxIdle值与maxActive值应配置的接近
    当连接数超过maxIdle值后，刚刚使用完的连接（刚刚空闲下来）会立即被销毁。而不是想要的空闲M秒后再销毁起一个缓冲作用。若maxIdle与maxActive相差较大，在高负载的系统中会导致频繁的创建、销毁连接，连接数在maxIdle与maxActive间快速频繁波动，这不是想要的。高负载系统的maxIdle值可以设置为与maxActive相同或设置为-1(-1表示不限制)，让连接数量在minIdle与maxIdle间缓冲慢速波动。

2. timeBetweenEvictionRunsMillis建议设置值
   minIdle要与timeBetweenEvictionRunsMillis配合使用才有用,单独使用minIdle不会起作用。

3. initialSize="5"，会在tomcat一启动时，创建5条连接，效果很理想。但同时我们还配置了minIdle="10"，也就是说，最少要保持10条连接，那现在只有5条连接，哪什么时候再创建少的5条连接呢？

   - 等业务压力上来了， DBCP就会创建新的连接

   - 配置timeBetweenEvictionRunsMillis=“时间”,DBCP会启用独立的工作线程定时检查，补上少的5条连接。销毁多余的连接也是同理。

#### 连接销毁的逻辑

 DBCP的连接数会在initialSize - minIdle - maxIdle - maxActive  之间变化。变化的逻辑描述如下：

默认未配置initialSize(默认值是0)和timeBetweenEvictionRunsMillis参数时，刚启动tomcat时，连接数是0。当应用有一个并发访问数据库时DBCP创建一个连接。目前连接数量还未达到minIdle，但DBCP也不自动创建新连接已使数量达到minIdle数量（没有一个独立的工作线程来检查和创建）。随着应用并发访问数据库的增多，连接数也增多，但都与minIdle值无关，很快minIdle被超越，minIdle值一点用都没有。直到连接的数量达到maxIdle值，这时的连接都是只增不减的。 再继续发展，连接数再增多并超过maxIdle时，使用完的连接（刚刚空闲下来的）会立即关闭，总体连接的数量稳定在maxIdle但不会超过maxIdle。
但活动连接（在使用中的连接）可能数量上瞬间超过maxIdle，但永远不会超过maxActive。这时如果应用业务压力小了，访问数据库的并发少了，连接数也不会减少（没有一个独立的线程来检查和销毁），将保持在maxIdle的数量。

默认未配置initialSize(默认值是0)，但配置了timeBetweenEvictionRunsMillis=“30000”（30秒）参数时，刚启动tomcat时，连接数是0。马上应用有一个并发访问数据库时DBCP创建一个连接。目前连接数量还未达到minIdle，每30秒DBCP的工作线程检查连接数是否少于minIdle数量，若少于就创建新连接直到达到minIdle数量。

随着应用并发访问数据库的增多，连接数也增多，直到达到maxIdle值。这期间每30秒DBCP的工作线程检查连接是否空闲了30分钟，若是就销毁。但此时是业务的高峰期，是不会有长达30分钟的空闲连接的，工作线程查了也是白查，但它在工作。到这里连接数量一直是呈现增长的趋势。

当连接数再增多超过maxIdle时，使用完的连接(刚刚空闲下来)会立即关闭，总体连接的数量稳定在maxIdle。停止了增长的趋势。但活动连接（在使用中的连接）可能数量上瞬间超过maxIdle，但永远不会超过maxActive。
 这时如果应用业务压力小了，访问数据库的并发少了，每30秒DBCP的工作线程检查连接(默认每次查3条)是否空闲达到30分钟(这是默认值)，若连接空闲达到30分钟，就销毁连接。这时连接数减少了，呈下降趋势，将从maxIdle走向minIdle。当小于minIdle值时，则DBCP创建新连接已使数量稳定在minIdle，并进行着新老更替。

#### 使用

配置文件：

```
#at https://commons.apache.org/proper/commons-dbcp/configuration.html
username=root
password=******
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/learn?useSSL=false&serverTimezone=GMT-8
initialSize=4
#最大连接数量
maxTotal=50

#<!-- 最大空闲连接 -->
maxIdle=20

#<!-- 最小空闲连接 -->
minIdle=5

#<!-- 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒 -->
maxWait=600


#JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;]
#注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。
connectionProperties=useUnicode=true;characterEncoding=UTF8

#指定由连接池所创建的连接的自动提交（auto-commit）状态。
defaultAutoCommit=true

#driver default 指定由连接池所创建的连接的只读（read-only）状态。
#如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix）
defaultReadOnly=

#driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
#可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
defaultTransactionIsolation=REPEATABLE_READ
```

DBCPUtils.java

```
public class DBCPUtils {
   private static DBCPUtils INSTANCE;
    /**
     * 在java中，编写数据库连接池需实现java.sql.DataSource接口，每一种数据库连接池都是DataSource接口的实现
     * DBCP连接池就是java.sql.DataSource接口的一个具体实现
     */
    private static DataSource ds = null;
    private static ThreadLocal<Connection> tl = null;

    private DBCPUtils() {
        try {

            //加载dbcpconfig.properties配置文件
            InputStream in = DBCPUtils.class.getClassLoader().getResourceAsStream("dbconfig/dbcp.properties");
            Properties prop = new Properties();
            prop.load(in);
            //创建数据源
            ds = BasicDataSourceFactory.createDataSource(prop);
            ds.setLogWriter(new PrintWriter(System.out));

            tl = new ThreadLocal<>();

        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }

    }

    /**
     * 单例模式，使用synchronize来保护线程安全
     *
     * @return DBCPUtils
     */
    public static DBCPUtils getInstance() {
        if (null == INSTANCE) {
            synchronized (DBCPUtils.class) {
                INSTANCE = new DBCPUtils();
            }
        }

        return INSTANCE;
    }

    public static synchronized Connection getConnection() throws SQLException {
        Connection conn = DBCPUtils.tl.get();
        if(conn==null){
            conn = DBCPUtils.ds.getConnection();
            DBCPUtils.tl.set(conn);
        }
        return conn;
    }

    //开启事务
    public static void startTransaction() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null)
            conn.setAutoCommit(false); //开启事务
    }
    //回滚事务
    public static void rollback() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null)
            conn.rollback();
    }
    //提交事务并释放资源
    public static void commitAndRelease() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null){
            conn.commit();
            conn.close();
            DBCPUtils.getInstance().tl.remove();
        }
    }
    //关闭资源方法：conn, rs, stat
}
```

和使用JDBC一样，通过工具类获取connection对象，获取到connection对象以后，正常使用就ok了；
注意不要导错包：

```
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
```

### C3p0

Maven依赖：

```
<dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.4</version>
    </dependency>
```

#### 配置

```
 <c3p0-config>
  <default-config>
 <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
 <property name="acquireIncrement">3</property>
 
 <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
 <property name="acquireRetryAttempts">30</property>
 
 <!--两次连接中间隔时间，单位毫秒。Default: 1000 -->
 <property name="acquireRetryDelay">1000</property>
 
 <!--连接关闭时默认将所有未提交的操作回滚。Default: false -->
 <property name="autoCommitOnClose">false</property>
 
 <!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么
  属性preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试
  使用。Default: null-->
 <property name="automaticTestTable">Test</property>
 
 <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效
  保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试
  获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
 <property name="breakAfterAcquireFailure">false</property>
 
 <!--当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出
  SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->  
 <property name="checkoutTimeout">100</property>
 
 <!--通过实现ConnectionTester或QueryConnectionTester的类来测试连接。类名需制定全路径。
  Default: com.mchange.v2.c3p0.impl.DefaultConnectionTester-->
 <property name="connectionTesterClassName"></property>
 
 <!--指定c3p0 libraries的路径，如果（通常都是这样）在本地即可获得那么无需设置，默认null即可
  Default: null-->
 <property name="factoryClassLocation">null</property>
 
 <!--Strongly disrecommended. Setting this to true may lead to subtle and bizarre bugs.  
  （文档原文）作者强烈建议不使用的一个属性-->  
 <property name="forceIgnoreUnresolvedTransactions">false</property>
 
 <!--每60秒检查所有连接池中的空闲连接。Default: 0 -->  
 <property name="idleConnectionTestPeriod">60</property>
 
 <!--初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->  
 <property name="initialPoolSize">3</property>
 
 <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
 <property name="maxIdleTime">60</property>
 
 <!--连接池中保留的最大连接数。Default: 15 -->
 <property name="maxPoolSize">15</property>
 
 <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements
  属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。
  如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
 <property name="maxStatements">100</property>
 
 <!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0  -->
 <property name="maxStatementsPerConnection"></property>
 
 <!--c3p0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能
  通过多线程实现多个操作同时被执行。Default: 3-->  
 <property name="numHelperThreads">3</property>
 
 <!--当用户调用getConnection()时使root用户成为去获取连接的用户。主要用于连接池连接非c3p0
  的数据源时。Default: null-->  
 <property name="overrideDefaultUser">root</property>
 
 <!--与overrideDefaultUser参数对应使用的一个参数。Default: null-->
 <property name="overrideDefaultPassword">password</property>
 
 <!--密码。Default: null-->  
 <property name="password"></property>
 
 <!--定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个一显著提高测试速度。注意：
  测试的表必须在初始数据源的时候就存在。Default: null-->
 <property name="preferredTestQuery">select id from test where id=1</property>
 
 <!--用户修改系统配置参数执行前最多等待300秒。Default: 300 -->  
 <property name="propertyCycle">300</property>
 
 <!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的
  时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable
  等方法来提升连接测试的性能。Default: false -->
 <property name="testConnectionOnCheckout">false</property>
 
 <!--如果设为true那么在取得连接的同时将校验连接的有效性。Default: false -->
 <property name="testConnectionOnCheckin">true</property>
 
 <!--用户名。Default: null-->
 <property name="user">root</property>
 
 <!--早期的c3p0版本对JDBC接口采用动态反射代理。在早期版本用途广泛的情况下这个参数
  允许用户恢复到动态反射代理以解决不稳定的故障。最新的非反射代理更快并且已经开始
  广泛的被使用，所以这个参数未必有用。现在原先的动态反射与新的非反射代理同时受到
  支持，但今后可能的版本可能不支持动态反射代理。Default: false-->
 <property name="usesTraditionalReflectiveProxies">false</property>
 
    <property name="automaticTestTable">con_test</property>
    <property name="checkoutTimeout">30000</property>
    <property name="idleConnectionTestPeriod">30</property>
    <property name="initialPoolSize">10</property>
    <property name="maxIdleTime">30</property>
    <property name="maxPoolSize">25</property>
    <property name="minPoolSize">10</property>
    <property name="maxStatements">0</property>
    <user-overrides user="swaldman">
    </user-overrides>
  </default-config>
  <named-config name="dumbTestConfig">
    <property name="maxStatements">200</property>
    <user-overrides user="poop">
      <property name="maxStatements">300</property>
    </user-overrides>
   </named-config>
</c3p0-config> 
```



#### 使用

配置文件c3p0-config.xml,位于资源文件夹下：

```
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <!--
    C3P0的缺省(默认)配置，
    如果在代码中“ComboPooledDataSource ds = new ComboPooledDataSource();”这样写就表示使用的是C3P0的缺省(默认)配置信息来创建数据源
    -->
    <default-config>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/learn</property>
        <property name="user">root</property>
        <property name="password">970603</property>

        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">5</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">20</property>
    </default-config>

    <!--
    C3P0的命名配置，
    如果在代码中“ComboPooledDataSource ds = new ComboPooledDataSource("MySQL");”这样写就表示使用的是name是MySQL的配置信息来创建数据源
    -->
    <named-config name="MySQL">
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/learn</property>
        <property name="user">root</property>
        <property name="password">970603</property>

        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">10</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">20</property>
    </named-config>

</c3p0-config>
```

C3p0Utils.java文件：

```
public class C3P0Utils {
 private static C3P0Utils INSTANCE=null;
    private static ComboPooledDataSource ds=null;
    private static ThreadLocal<Connection> tl = null;

    private C3P0Utils()
    {
        try{
            ds=new ComboPooledDataSource();
            //通过读取C3P0的xml配置文件创建数据源，C3P0的xml配置文件c3p0-config.xml必须放在src目录下
            //ds = new ComboPooledDataSource();//使用C3P0的默认配置来创建数据源
//            ds = new ComboPooledDataSource("MySQL");//使用C3P0的命名配置来创建数据源
            ds.setLogWriter(new PrintWriter(System.out));
            tl = new ThreadLocal<>();

        }catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }


    public static C3P0Utils getInstance()
    {
        if (null==INSTANCE)
        {
            synchronized(C3P0Utils.class) {
                INSTANCE = new C3P0Utils();
            }
        }
        return INSTANCE;
    }

 

    public static synchronized Connection getConnection() throws SQLException {
        Connection conn = C3P0Utils.tl.get();
        if(conn==null){
            conn = C3P0Utils.ds.getConnection();
            C3P0Utils.tl.set(conn);
        }
        return conn;
    }

    //开启事务
    public static void startTransaction() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null)
            conn.setAutoCommit(false); //开启事务
    }
    //回滚事务
    public static void rollback() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null)
            conn.rollback();
    }
    //提交事务并释放资源
    public static void commitAndRelease() throws SQLException{
        Connection conn = getConnection();
        if(conn!=null){
            conn.commit();
            conn.close();
            tl.remove();
        }
    }
    //关闭资源方法：conn, rs, stat

}
```

### Druid

Maven依赖：

```
 <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
    </dependency>
```

#### 配置

| 配置                                      | 缺省值             | 说明                                                         |
| ----------------------------------------- | ------------------ | ------------------------------------------------------------ |
| name                                      |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是："DataSource-"  + System.identityHashCode(this). 另外配置此属性至少在1.0.5版本中是不起作用的，强行设置name会出错。[详情-点此处](http://blog.csdn.net/lanmo555/article/details/41248763)。 |
| url                                       |                    | 连接数据库的url，不同数据库不一样。例如： mysql : jdbc:mysql://10.20.153.104:3306/druid2  oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                                  |                    | 连接数据库的用户名                                           |
| password                                  |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。[详细看这里](https://github.com/alibaba/druid/wiki/使用ConfigFilter) |
| driverClassName                           | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName |
| initialSize                               | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                                 | 8                  | 最大连接池数量                                               |
| maxIdle                                   | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                                   |                    | 最小连接池数量                                               |
| maxWait                                   |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements                    | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxPoolPreparedStatementPerConnectionSize | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery                           |                    | 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。 |
| validationQueryTimeout                    |                    | 单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
| testOnBorrow                              | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                              | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testWhileIdle                             | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| keepAlive                                 | false （1.0.28）   | 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作。 |
| timeBetweenEvictionRunsMillis             | 1分钟（1.0.14）    | 有两个含义： 1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun                    | 30分钟（1.0.14）   | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis                |                    | 连接保持空闲而不被驱逐的最小时间                             |
| connectionInitSqls                        |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter                           | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                                   |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat 日志用的filter:log4j 防御sql注入的filter:wall |
| proxyFilters                              |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |

#### 使用

配置文件druid.properties：

```
#Druid at https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8
druid.username=root
druid.password=970603
druid.driverClassName=com.mysql.cj.jdbc.Driver
druid.url=jdbc:mysql://127.0.0.1:3306/learn?useSSL=false&serverTimezone=GMT-8
druid.initialSize=4
#最大连接数量
druid.maxTotal=50

#<!-- 最大空闲连接 -->
druid.maxIdle=20

#<!-- 最小空闲连接 -->
druid.minIdle=5

#<!-- 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒 -->
druid.maxWait=600


#JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;]
#注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。
connectionProperties=useUnicode=true;characterEncoding=UTF8

#指定由连接池所创建的连接的自动提交（auto-commit）状态。
druid.defaultAutoCommit=true

#driver default 指定由连接池所创建的连接的只读（read-only）状态。
#如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix）
druid.defaultReadOnly=

#driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
#可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
druid.defaultTransactionIsolation=REPEATABLE_READ

#Druid配置

#是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。
#poolPreparedStatements=true

#要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。
# 在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
#maxPoolPreparedStatementPerConnectionSize=100
```

DruidUtils文件：

```
public class DruidUtils {
    private static DruidUtils Instance = new DruidUtils();

    public static DruidUtils getInstance() {
        if (null == Instance) {
            synchronized (DruidUtils.class) {
                Instance = new DruidUtils();
            }
        }
        return Instance;
    }

    private DruidDataSource ds = null;

    private DruidUtils() {
        Properties properties = new Properties();
        try {
            properties.load(ClassLoader.getSystemResourceAsStream("druid.properties"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        ds = new DruidDataSource();
        ds.configFromPropety(properties);
        System.out.println(ds);
    }

    /**
     * @return Connection
     * @throws SQLException
     * @Method: getConnection
     * @Description: 从数据源中获取数据库连接
     */
    public static Connection getConnection() throws SQLException {
        return DruidUtils.getInstance().ds.getConnection();
    }
}

```

### HikariCP

Maven依赖：

```
  <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>3.3.1</version>
    </dependency>
    
```

#### 配置

必要属性：

| 属性名              | 特性                                                         |
| ------------------- | ------------------------------------------------------------ |
| dataSourceClassName | 这是JDBC驱动程序提供的DataSource类的名称。 请参阅特定JDBC驱动程序的文档以获取此类名，或参见下表。 注意不支持XA数据源。 XA需要像bitronix这样的真实事务管理器。 请注意，如果您使用jdbcUrl进行“old-school”基于DriverManager的JDBC驱动程序配置，则不需要此属性。 默认值：无 |
| jdbcUrl             | 此属性指示HikariCP使用“基于DriverManager的”配置。 我们认为基于DataSource的配置（上面）由于各种原因（见下文）而优越，但对于许多部署而言，几乎没有显着差异。将此属性与“旧”驱动程序一起使用时，您可能还需要设置driverClassName属性，但首先不使用它。 请注意，如果使用此属性，您仍可以使用DataSource属性来配置驱动程序，实际上建议使用URL本身中指定的驱动程序参数。 默认值：无 |
| username            | 此属性设置从基础驱动程序获取Connections时使用的默认身份验证用户名。 请注意，对于DataSources，它通过在基础DataSource上调用DataSource.getConnection（* username *，password）以非常确定的方式工作。 但是，对于基于驱动程序的配置，每个驱动程序都不同。 在基于驱动程序的情况下，HikariCP将使用此username属性在传递给驱动程序的DriverManager.getConnection（jdbcUrl，props）调用的属性中设置用户属性。 如果这不是您所需要的，请完全跳过此方法并调用addDataSourceProperty（“username”，...），例如。 默认值：无 |
| password            | 此属性设置从基础驱动程序获取Connections时使用的默认验证密码。 请注意，对于DataSources，它通过在基础DataSource上调用DataSource.getConnection（username，* password *）以非常确定的方式工作。 但是，对于基于驱动程序的配置，每个驱动程序都不同。 在基于驱动程序的情况下，HikariCP将使用此密码属性在传递给驱动程序的DriverManager.getConnection（jdbcUrl，props）调用的属性中设置密码属性。 如果这不是您所需要的，请完全跳过此方法并调用addDataSourceProperty（“pass”，...），例如。 默认值：无 |

常用属性

| 属性                | 特性                                                         |
| ------------------- | ------------------------------------------------------------ |
| autoCommit          | 控制从池返回的连接的默认自动提交行为。 它是一个布尔值。 默认值：true |
| connectionTimeout   | 控制客户机(即您)等待来自池的连接的最大毫秒数。如果在没有连接可用的情况下超过此时间，将抛出SQLException异常。最低可接受的连接超时是250毫秒。默认值:30000(30秒) |
| idleTimeout         | 控制允许连接在池中空闲的最长时间。 此设置仅在minimumIdle定义为小于maximumPoolSize时适用。 一旦池达到minimumIdle连接，空闲连接将不会退出。 连接是否空闲退出的最大变化为+30秒，平均变化为+15秒。 在此超时之前，连接永远不会被空闲。 值为0表示永远不会从池中删除空闲连接。 允许的最小值为10000毫秒（10秒）。 默认值：600000（10分钟） |
| maxLifetime         | 控制池中连接的最长生命周期。 使用中的连接永远不会退役，只有当它关闭时才会被删除。 在逐个连接的基础上，应用轻微的负衰减以避免池中的大量灭绝。 我们强烈建议设置此值，它应比任何数据库或基础结构强加的连接时间限制短几秒。 值0表示没有最大生命周期（无限生命周期），当然主题是idleTimeout设置。 默认值：1800000（30分钟） |
| connectionTestQuery | 如果您的驱动程序支持JDBC4，我们强烈建议您不要设置此属性。 这适用于不支持JDBC4 Connection.isValid（）API的“遗留”驱动程序。 这是在从池中给出连接之前执行的查询，以验证与数据库的连接是否仍然存在。 再次尝试运行没有此属性的池，如果您的驱动程序不符合JDBC4，HikariCP将记录错误以通知您。 默认值：无 |
| minimumIdle         | 控制HikariCP尝试在池中维护的最小空闲连接数。 如果空闲连接低于此值并且池中的总连接数小于maximumPoolSize，则HikariCP将尽最大努力快速有效地添加其他连接。 但是，为了获得最高性能和对峰值需求的响应，我们建议不要设置此值，而是允许HikariCP充当固定大小的连接池。 默认值：与maximumPoolSize相同 |
| maximumPoolSize     | 控制允许池到达的最大大小，包括空闲和正在使用的连接。 基本上，此值将确定数据库后端的最大实际连接数。 对此的合理值最好由您的执行环境决定。 当池达到此大小且没有空闲连接可用时，对getConnection（）的调用将在超时之前阻塞最多connectionTimeout毫秒。  默认值：10 |
| metricRegistry      | 仅可通过编程配置或IoC容器获得。 此属性允许您指定池使用的Codahale / Dropwizard MetricRegistry的实例来记录各种度量标准。 有关详细信息，请参阅度量维基页面。 默认值：无 |
| healthCheckRegistry | 仅可通过编程配置或IoC容器获得。 此属性允许您指定池使用的Codahale / Dropwizard HealthCheckRegistry的实例来报告当前的健康信息。 有关详细信息，请参阅运行状况检查维基页面。 默认值：无 |
| poolName            | 此属性表示连接池的用户定义名称，主要显示在日志记录和JMX管理控制台中，以标识池和池配置。 默认值：*auto-generated* |

不常用属性

| 属性                      | 特性                                                         |
| ------------------------- | ------------------------------------------------------------ |
| initializationFailTimeout | 如果池无法成功初始化连接，则此属性控制池是否“快速失败”。 任何正数都被认为是尝试获取初始连接的毫秒数; 在此期间，应用程序线程将被阻止。 如果在此超时发生之前无法获取连接，则将引发异常。 在connectionTimeout期间之后应用此超时。 如果值为零（0），HikariCP将尝试获取并验证连接。 如果获得连接但验证失败，则将引发异常并且池未启动。 但是，如果无法获得连接，则池将启动，但稍后获取连接的努力可能会失败。 小于零的值将绕过任何初始连接尝试，并且池将在尝试在后台获取连接时立即启动。 因此，稍后获得连接的努力可能失败。 默认值：1 |
| isolateInternalQueries    | 此属性确定HikariCP是否在其自己的事务中隔离内部池查询，例如连接活动测试。 由于这些通常是只读查询，因此很少需要将它们封装在自己的事务中。 此属性仅在禁用autoCommit时适用。 默认值：false |
| allowPoolSuspension       | 此属性控制是否可以通过JMX挂起和恢复池。 这对某些故障转移自动化方案很有用。 当池暂停时，对getConnection（）的调用不会超时，并将一直保持到池恢复为止。 默认值：false |
| readOnly                  | 此属性控制默认情况下从池中获取的Connections是否处于只读模式。 请注意，某些数据库不支持只读模式的概念，而其他数据库在Connection设置为只读时提供查询优化。 您是否需要此属性在很大程度上取决于您的应用程序和数据库。 默认值：false |
| registerMbeans            | 此属性控制是否注册JMX管理Bean（“MBean”）。 默认值：false     |
| catalog                   | 此属性设置支持目录概念的数据库的缺省目录。 如果未指定此属性，则使用JDBC驱动程序定义的缺省目录。 默认值：驱动程序默认 |
| connectionInitSql         | 此属性设置一个SQL语句，该语句将在每次创建新连接之后执行，然后再将其添加到池中。 如果此SQL无效或引发异常，则将其视为连接失败，并将遵循标准重试逻辑。 默认值：无 |
| driverClassName           | HikariCP将尝试仅基于jdbcUrl通过DriverManager解析驱动程序，但对于某些较旧的驱动程序，还必须指定driverClassName。 除非您收到明显的错误消息，指出未找到驱动程序，否则请忽略此属性。 默认值：无 |
| transactionIsolation      | 此属性控制从池返回的连接的默认事务隔离级别。 如果未指定此属性，则使用JDBC驱动程序定义的缺省事务隔离级别。 如果您具有所有查询通用的特定隔离要求，则仅使用此属性。 此属性的值是Connection类的常量名称，例如TRANSACTION_READ_COMMITTED，TRANSACTION_REPEATABLE_READ等。默认值：驱动程序默认值 |
| validationTimeout         | 此属性控制连接测试活动的最长时间。 该值必须小于connectionTimeout。 最低可接受的验证超时为250毫秒。 默认值：5000 |
| leakDetectionThreshold    | 此属性控制在记录消息之前连接可以离开池的时间量，指示可能的连接泄漏。 值为0表示禁用泄漏检测。 启用泄漏检测的最低可接受值是2000（2秒）。 默认值：0 |
| schema                    | 此属性仅可通过编程配置或IoC容器获得。 此属性允许您直接设置要由池包装的DataSource实例，而不是让HikariCP通过反射构造它。 这在一些依赖注入框架中很有用。 指定此属性后，将忽略dataSourceClassName属性和所有特定于DataSource的属性。 默认值：无 |
| dataSource                | 此属性设置支持模式概念的数据库的默认模式。 如果未指定此属性，则使用JDBC驱动程序定义的默认架构。 默认值：驱动程序默认 |
| threadFactory             | 此属性仅可通过编程配置或IoC容器获得。 此属性允许您设置将用于创建池使用的所有线程的java.util.concurrent.ThreadFactory的实例。 在某些受限执行环境中需要它，其中线程只能通过应用程序容器提供的ThreadFactory创建。 默认值：无 |
| scheduledExecutor         | 此属性仅可通过编程配置或IoC容器获得。 此属性允许您设置将用于各种内部计划任务的java.util.concurrent.ScheduledExecutorService的实例。 如果为HikariCP提供ScheduledThreadPoolExecutor实例，建议使用setRemoveOnCancelPolicy（true）。 默认值：无 |

#### 使用

属性文件HikariCP.properties

```
#HikariCP配置

username=root
password=970603
jdbcUrl=jdbc:mysql://127.0.0.1:3306/learn?useSSL=false&serverTimezone=GMT-8
dataSource.cachePrepStmts=true
dataSource.prepStmtCacheSize=250
dataSource.prepStmtCacheSqlLimit=2048
dataSource.useServerPrepStmts=true
dataSource.useLocalSessionState=true
dataSource.rewriteBatchedStatements=true
dataSource.cacheResultSetMetadata=true
dataSource.cacheServerConfiguration=true
dataSource.elideSetAutoCommits=true
dataSource.maintainTimeStats=false
```

HCPUtils文件：

```
public class HCPUtils {
    private static HCPUtils INSTANCE=null;

    private HikariDataSource dataSource=null;

    private HCPUtils()
    {
        Properties properties=new Properties();
        try {
            properties.load(ClassLoader.getSystemResourceAsStream("dbconfig/HikariCP.properties") );
        } catch (IOException e) {
            e.printStackTrace();
        }
        HikariConfig config=new HikariConfig(properties);
        dataSource=new HikariDataSource(config);
    }
    public static HCPUtils getINSTANCE()
    {
        if (null==INSTANCE)
        {
            INSTANCE=new HCPUtils();
        }
        return INSTANCE;
    }

    public static Connection getConnection() throws SQLException {
        return HCPUtils.getINSTANCE().dataSource.getConnection();
    }


}

```

### 自定义数据库连接池

编写连接池需要实现 `javax.sql.DataSource` 接口。`DateSource` 接口中定义了 `2` 个重载 `getConnection` 的方法 ：

    Connection getConnection（） ；
    
    Connection getConnection(String username,String password) ;
实现 DateSource 接口，并实现连接池功能的步骤：

```
a、在DateSource构造器中批量创建与数据库的连接，并把创建的连接加入线程安全的集合中 ；

b、实现getConnection方法，让getConnection方法每次调用时，从集合中取出一个连接返回给用户 ；

c、当用户使用完Connection以后，调用Connection.close（）方法时，Collection对象保证将自己返回到集合中，而不是把conn返给数据库 ；

其中Collection保证将自己返回给集合中是此处编程的难点！！
```

在实现 Connection getConnection（） 的时候，有个问题，就是我们不能简单的从LinkedList 里面获取 Connection 返回，而要是确保 linkedList 里面将按个Connection 对象移除了，否则会造成连接冲突；

还有一个问题，就是用户在用完 Connection 的时候，会习惯调用 close（）方法，这样就导致了连接放给了数据库，而不是返给了我们连接池；

**`close（）`方法加强**

现在就有一个问题，就是如何保证调用 `Connection的close（）`方法，是将连接还给连接池而不是数据库 ？

这就涉及到，我们对一个类功能进行加强的问题；在java里面，通常有三种方法；

1. 写一个子类，覆写父类的方法

   这个做法有个致命的缺点：就是必须获得父类的所有信息，向我们这里，继承是基本不可能的！

   因为，真正返回给我们的连接是，mysql的驱动类生成的连接，这个类里面，封装了太多的信息：比如：连接的数据库地址、数据库名字、用户名、连接密码。。这些信息 ；

   我们要真的想继承它，我们就要在子类中得到这些信息，可惜人家都说private字段，子类也拿不着；子类拿不着，那么子类就连不上数据库。。

   除非你去看去mysql的驱动源码，自己把信息拿出来，写在你创建的子类里面，但这跟重写mysql驱动没哈区别了，工程量吓人！

   因此，这里放弃这种做法 ；

2. 用包装模式

   写法：

   - 定义一个类实现与被增强类相同的接口
   - 在类中定义一个变量，记住被增强对象
   - 定义一个构造器，参数接受一个被增强对象的类 
   - 覆盖我们向增强的方法 ；
   - 对于不想增强的方法，直接调用被增强对象的方法 ；

   这样就完美的增强了我们想要的方法；但是也有缺点，就是接口中的每个方法，都要我们去调用被增强对象的方法 ；

   接口中的方法少还行，一旦很多，这种代码，我们就要写很多次，，，，，很恶心！

   因此，应该考虑动态代理 ；       

3. 动态代理（aop编程，面向切面编程）

   主要是利用拦截；（暂时，我也不会，以后会跟，最多20天以内更新。写于2018年6月12日22:07:53）

#### 实现

配置文件db.properties：

```
mysql.user=root
mysql.psw=970603
mysql.driver=com.mysql.cj.jdbc.Driver
mysql.url=jdbc:mysql://127.0.0.1:3306/learn?useSSL=false&serverTimezone=GMT-8
mysql.jdbcPoolInitSize=4
mysql.initialSize=4

#最大连接数量
mysql.maxActive=50

#<!-- 最大空闲连接 -->
mysql.maxIdle=20

#<!-- 最小空闲连接 -->
mysql.minIdle=5

#<!-- 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒 -->
mysql.maxWait=60000
#JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;]
#注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。
mysql.connectionProperties=useUnicode=true;characterEncoding=UTF8

#指定由连接池所创建的连接的自动提交（auto-commit）状态。
mysql.defaultAutoCommit=true

#driver default 指定由连接池所创建的连接的只读（read-only）状态。
#如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix）
mysql.defaultReadOnly=

#driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
#可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
mysql.defaultTransactionIsolation=REPEATABLE_READ
```

JDBCPool文件：

```
public class JDBCPool implements DataSource {
	//线程安全的队列
    private static ConcurrentLinkedQueue<Connection> connections=new ConcurrentLinkedQueue<Connection>();

    private Logger L=Logger.getLogger("JDBCPool");
    private static String username;
    private static  String psw;
    private static  String url;
    private static String driver;

    static {
        Properties properties=new Properties();
        InputStream inputStream=ClassLoader.getSystemResourceAsStream("dbconfig/db.properties");
        try {
            try {
                properties.load(inputStream);
                username=properties.getProperty("mysql.user");
                psw=properties.getProperty("mysql.psw");
                url=properties.getProperty("mysql.url");
                driver=properties.getProperty("mysql.driver");
                int jdbcPoolInitSize=Integer.parseInt(properties.getProperty("mysql.jdbcPoolInitSize"));
                Class.forName(driver);//加载驱动
                for (int i = 0; i < jdbcPoolInitSize; i++) {
                    Connection connection= null;
                    try {
                        connection = DriverManager.getConnection(url,username,psw);
                        connection.setAutoCommit(false);//设置非自动提交事务
                        connection.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);//设置事务级别
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
                    connections.add(connection);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            Class.forName(driver);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
   
    @Override
    public Connection getConnection() throws SQLException {
        if (connections.size()>0)
        {
            final Connection connection=connections.remove();
            //L.info("数据库大小为：" + connections.size());
            //设置动态代理，当connection调用close方法时，就会将connection放回链表中
            return (Connection) Proxy.newProxyInstance(JDBCPool.class.getClassLoader(),
                    connection.getClass().getInterfaces(), new InvocationHandler() {
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {//使用反射对connection的方法进行操作
                            if (!method.getName().equals("close"))
                            {
                                return method.invoke(connection,args);
                            }else {
                                connections.add(connection);
                                //L.info(connection + "已经还给数据库连接池");
                                //L.info("数据库大小为：" + connections.size());
                            }
                            return null;
                        }
                    });
        }else {
            throw  new RuntimeException("数据库忙");
        }
    }

   
    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }

    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    public void setLogWriter(PrintWriter out) throws SQLException {
        
    }

    public void setLoginTimeout(int seconds) throws SQLException {

    }

    public int getLoginTimeout() throws SQLException {
        return 1000;
    }

    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return Logger.getLogger(JDBCPool.class.getName());
    }
}
```

JDBCUtils文件:

```
public class JDBCUtils {

    private static JDBCUtils INSTANCE;
    private JDBCPool jdbcPool;
    private JDBCUtils()
    {
        jdbcPool=new JDBCPool();
    }
    public static  JDBCUtils getInstance()
    {
        if (null==INSTANCE)
        {
            INSTANCE=new JDBCUtils();
        }
        return INSTANCE;
    }


    public static Connection getConnection() throws SQLException {

        return JDBCUtils.getInstance().jdbcPool.getConnection();
    }
}
```

