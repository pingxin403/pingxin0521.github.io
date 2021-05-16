---
title: Spring 事务
date: 2019-06-22 20:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

#### Spring事务体系

![QoctzR.md.png](https://s2.ax1x.com/2019/12/17/QoctzR.md.png)

Spring事务包含对**分布式事务**和**单机事务**的支持，我们用的比较多的是单机事务，也就是只操作一个数据库的事务。

单机事务，按照用法分，又可以分为**编程式事务模型**（TransactionTemplate）和**声明式事务模型**（@Transactional注解），后者可以理解为 aop + 编程式事务模型。

编程式事务模型里面涉及到很多知识点，比如统一事务模型、事务传播级别、事务隔离级别等

统一事务模型：https://zhuanlan.zhihu.com/p/38772486

声明式事务模型：https://zhuanlan.zhihu.com/p/41864893

#### 局部事务

<!--more-->

理解事务之前，先讲一个你日常生活中最常干的事：取钱。 

比如你去ATM机取1000块钱，大体有两个步骤：首先输入密码金额，银行卡扣掉1000元钱；然后ATM出1000元钱。这两个步骤必须是要么都执行要么都不执行。如果银行卡扣除了1000块但是ATM出钱失败的话，你将会损失1000元；如果银行卡扣钱失败但是ATM却出了1000块，那么银行将损失1000元。所以，如果一个步骤成功另一个步骤失败对双方都不是好事，如果不管哪一个步骤失败了以后，整个取钱过程都能回滚，也就是完全取消所有操作的话，这对双方都是极好的。 

事务就是用来解决类似问题的。事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。 

在企业级应用程序开发中，事务管理必不可少的技术，用来确保数据的完整性和一致性。 

事务有四个特性：ACID

- 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
- 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

![1.jpeg](https://i.loli.net/2019/07/09/5d24031a94aee50249.jpeg)

#### 核心接口

Spring事务管理的实现有许多细节，如果对整个接口框架有个大体了解会非常有利于我们理解事务，下面通过讲解Spring的事务接口来了解Spring实现事务的具体策略。 

1. 事务管理器

   Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 

   Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。此接口的内容如下：

   ```java
   Public interface PlatformTransactionManager()...{  
       // 由TransactionDefinition得到TransactionStatus对象
       TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
       // 提交
       Void commit(TransactionStatus status) throws TransactionException;  
       // 回滚
       Void rollback(TransactionStatus status) throws TransactionException;  
       } 
   ```

   从这里可知具体的具体的事务管理机制对Spring来说是透明的，它并不关心那些，那些是对应各个平台需要关心的，所以Spring事务管理的一个优点就是为不同的事务API提供一致的编程模型，如JTA、JDBC、Hibernate、JPA。下面分别介绍各个平台框架实现事务管理的机制。

2. JDBC事务

   如果应用程序中直接使用JDBC来进行持久化，DataSourceTransactionManager会为你处理事务边界。为了使用DataSourceTransactionManager，你需要使用如下的XML将其装配到应用程序的上下文定义中：

   ```xml
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource" />
       </bean>
   ```

   实际上，DataSourceTransactionManager是通过调用java.sql.Connection来管理事务，而后者是通过DataSource获取到的。通过调用连接的commit()方法来提交事务，同样，事务失败则通过调用rollback()方法进行回滚。

3. Hibernate事务

   如果应用程序的持久化是通过Hibernate实习的，那么你需要使用HibernateTransactionManager。对于Hibernate3，需要在Spring上下文定义中添加如下的`<bean>`声明：

   ```xml
   <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean>
   ```

   sessionFactory属性需要装配一个Hibernate的session工厂，HibernateTransactionManager的实现细节是它将事务管理的职责委托给org.hibernate.Transaction对象，而后者是从Hibernate Session中获取到的。当事务成功完成时，HibernateTransactionManager将会调用Transaction对象的commit()方法，反之，将会调用rollback()方法。

4.  Java持久化API事务（JPA）

   Hibernate多年来一直是事实上的Java持久化标准，但是现在Java持久化API作为真正的Java持久化标准进入大家的视野。如果你计划使用JPA的话，那你需要使用Spring的JpaTransactionManager来处理事务。你需要在Spring中这样配置JpaTransactionManager：

   ```xml
   <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean>
   ```

   JpaTransactionManager只需要装配一个JPA实体管理工厂（javax.persistence.EntityManagerFactory接口的任意实现）。JpaTransactionManager将与由工厂所产生的JPA EntityManager合作来构建事务。

5. Java原生API事务

   如果你没有使用以上所述的事务管理，或者是跨越了多个事务管理源（比如两个或者是多个不同的数据源），你就需要使用JtaTransactionManager：

   ```xml
       <bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
           <property name="transactionManagerName" value="java:/TransactionManager" />
       </bean>
   ```

   JtaTransactionManager将事务管理的责任委托给javax.transaction.UserTransaction和javax.transaction.TransactionManager对象，其中事务成功完成通过UserTransaction.commit()方法提交，事务失败通过UserTransaction.rollback()方法回滚。

#### 基本事务属性

上面讲到的事务管理器接口PlatformTransactionManager通过getTransaction(TransactionDefinition definition)方法来得到事务，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性。 

那么什么是事务属性呢？事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。

![2.jpeg](https://i.loli.net/2019/07/09/5d24031a6679390640.jpeg)

TransactionDefinition接口内容如下：

```java
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
} 
```

我们可以发现TransactionDefinition正好用来定义事务属性，下面详细介绍一下各个事务属性。

![Qoyvj0.png](https://s2.ax1x.com/2019/12/17/Qoyvj0.png)

##### 传播行为

事务的第一个方面是传播行为（propagation behavior）。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring定义了七种传播行为：

| 传播行为                  | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。

**PROPAGATION_NESTED 与PROPAGATION_REQUIRES_NEW的区别:**它们非常类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。使用 PROPAGATION_REQUIRES_NEW时，内层事务与外层事务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。两个事务不是一个真正的嵌套事务。同时它需要JTA事务管理器的支持。

使用PROPAGATION_NESTED时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，它是一个真正的嵌套事务。DataSourceTransactionManager使用savepoint支持PROPAGATION_NESTED时，需要JDBC 3.0以上驱动及1.4以上的JDK版本支持。其它的JTA TrasactionManager实现可能有不同的支持方式。

PROPAGATION_REQUIRES_NEW 启动一个新的, 不依赖于环境的 “内部” 事务. 这个事务将被完全 commited 或 rolled back 而不依赖于外部事务, 它拥有自己的隔离范围, 自己的锁, 等等. 当内部事务开始执行时, 外部事务将被挂起, 内务事务结束时, 外部事务将继续执行。

另一方面, PROPAGATION_NESTED 开始一个 “嵌套的” 事务, 它是已经存在事务的一个真正的子事务. 潜套事务开始执行时, 它将取得一个 savepoint. 如果这个嵌套事务失败, 我们将回滚到此 savepoint. 潜套事务是外部事务的一部分, 只有外部事务结束后它才会被提交。

由此可见, PROPAGATION_REQUIRES_NEW 和 PROPAGATION_NESTED 的最大区别在于, PROPAGATION_REQUIRES_NEW 完全是一个新的事务, 而 PROPAGATION_NESTED 则是外部事务的子事务, 如果外部事务 commit, 嵌套事务也会被 commit, 这个规则同样适用于 roll back.

**PROPAGATION_REQUIRED应该是我们首先的事务传播行为。它能够满足我们大多数的事务需求。**

##### 隔离级别

事务的第二个维度就是隔离级别（isolation level）。隔离级别定义了一个事务可能受其他并发事务影响的程度。 
（1）并发事务引起的问题 
在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务。并发虽然是必须的，但可能会导致一下的问题。

- 脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
- 不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
- 幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

**不可重复读与幻读的区别**

不可重复读的重点是修改: 
同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 
例如：在事务1中，Mary 读取了自己的工资为1000,操作并没有完成

```
    con1 = getConnection();  
    select salary from employee empId ="Mary";  
```

在事务2中，这时财务人员修改了Mary的工资为2000,并提交了事务.

```
    con2 = getConnection();  
    update employee set salary = 2000;  
    con2.commit();  
```

在事务1中，Mary 再次读取自己的工资时，工资变为了2000

```
    //con1  
    select salary from employee empId ="Mary"; 
```

在一个事务中前后两次读取的结果并不一致，导致了不可重复读。

幻读的重点在于新增或者删除： 
同样的条件, 第1次和第2次读出来的记录数不一样 
例如：目前工资为1000的员工有10人。事务1,读取所有工资为1000的员工。

```
    con1 = getConnection();  
    Select * from employee where salary =1000; 
```

共读取10条记录

这时另一个事务向employee表插入了一条员工记录，工资也为1000

```
    con2 = getConnection();  
    Insert into employee(empId,salary) values("Lili",1000);  
    con2.commit();  
```

事务1再次读取所有工资为1000的员工

```
    //con1  
    select * from employee where salary =1000;  
```

共读取到了11条记录，这就产生了幻像读。

从总的结果来看, 似乎不可重复读和幻读都表现为两次读取的结果不一致。但如果你从控制的角度来看, 两者的区别就比较大。 

对于前者, 只需要**锁住满足条件的记录**。 
对于后者, 要**锁住满足条件及其相近的记录**。

**隔离级别**

| 隔离级别                   | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别                                 |
| ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 |
| ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 |
| ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 |
| ISOLATION_SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 |

##### 只读

事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

##### 事务超时

为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

##### 回滚规则

事务五边形的最后一个方面是一组规则，这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的） 

但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

#### 事务状态

上面讲到的调用PlatformTransactionManager接口的getTransaction()的方法得到的是TransactionStatus接口的一个实现，这个接口的内容如下：

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 
```

可以发现这个接口描述的是一些处理事务提供简单的控制事务执行和查询事务状态的方法，在回滚或提交的时候需要应用对应的事务状态。

#### 编程式事务

**编程式和声明式事务的区别**

Spring提供了对编程式事务和声明式事务的支持，编程式事务允许用户在代码中精确定义事务的边界，而声明式事务（基于AOP）有助于用户将操作与事务规则进行解耦。 

简单地说，编程式事务侵入到了业务代码里面，但是提供了更加详细的事务管理；而声明式事务由于基于AOP，所以既能起到事务管理的作用，又可以不影响业务代码的具体实现。

**如何实现编程式事务？**

Spring提供两种方式的编程式事务管理，分别是：使用TransactionTemplate和直接使用PlatformTransactionManager。

1. 使用TransactionTemplate

   采用TransactionTemplate和采用其他Spring模板，如JdbcTempalte和HibernateTemplate是一样的方法。它使用回调方法，把应用程序从处理取得和释放资源中解脱出来。如同其他模板，TransactionTemplate是线程安全的。代码片段：

   ```java
    TransactionTemplate tt = new TransactionTemplate(); // 新建一个TransactionTemplate
       Object result = tt.execute(
           new TransactionCallback(){  
               public Object doTransaction(TransactionStatus status){  
                   updateOperation();  
                   return resultOfUpdateOperation();  
               }  
       }); // 执行execute方法进行事务管理
   ```

   使用TransactionCallback()可以返回一个值。如果使用TransactionCallbackWithoutResult则没有返回值。

2. 使用PlatformTransactionManager

   ```java
      DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(); //定义一个某个框架平台的TransactionManager，如JDBC、Hibernate
       dataSourceTransactionManager.setDataSource(this.getJdbcTemplate().getDataSource()); // 设置数据源
       DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); // 定义事务属性
       transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); // 设置传播行为属性
       TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef); // 获得事务状态
       try {
           // 数据库操作
           dataSourceTransactionManager.commit(status);// 提交
       } catch (Exception e) {
           dataSourceTransactionManager.rollback(status);// 回滚
       }
   ```

#### 声明式事务

 **配置方式**

根据代理机制的不同，总结了五种Spring事务的配置方式，配置文件如下：

1. 每个Bean都有一个代理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 

    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <bean id="userDao" 
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean"> 
           <!-- 配置事务管理器 --> 
           <property name="transactionManager" ref="transactionManager" />    
        <property name="target" ref="userDaoTarget" /> 
         <property name="proxyInterfaces" value="com.bluesky.spring.dao.GeneratorDao" />
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props> 
        </property> 
    </bean> 
</beans>
```

2. 所有Bean共享一个代理基类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 

    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <bean id="transactionBase" 
            class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean" 
            lazy-init="true" abstract="true"> 
        <!-- 配置事务管理器 --> 
        <property name="transactionManager" ref="transactionManager" /> 
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop> 
            </props> 
        </property> 
    </bean>   

    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <bean id="userDao" parent="transactionBase" > 
        <property name="target" ref="userDaoTarget" />  
    </bean>
</beans>
```

3. 使用拦截器

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-2.5.xsd
              http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
   
       <bean id="sessionFactory" 
               class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
           <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
           <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
       </bean> 
   
       <!-- 定义事务管理器（声明式的事务） --> 
       <bean id="transactionManager"
           class="org.springframework.orm.hibernate3.HibernateTransactionManager">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean> 
   
       <bean id="transactionInterceptor" 
           class="org.springframework.transaction.interceptor.TransactionInterceptor"> 
           <property name="transactionManager" ref="transactionManager" /> 
           <!-- 配置事务属性 --> 
           <property name="transactionAttributes"> 
               <props> 
                   <prop key="*">PROPAGATION_REQUIRED</prop> 
               </props> 
           </property> 
       </bean>
   
       <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator"> 
           <property name="beanNames"> 
               <list> 
                   <value>*Dao</value>
               </list> 
           </property> 
           <property name="interceptorNames"> 
               <list> 
                   <value>transactionInterceptor</value> 
               </list> 
           </property> 
       </bean> 
   
       <!-- 配置DAO -->
       <bean id="userDao" class="com.bluesky.spring.dao.UserDaoImpl">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean>
   </beans>
   ```

4. 使用tx标签配置的拦截器

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-2.5.xsd
              http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
              http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
   
       <context:annotation-config />
       <context:component-scan base-package="com.bluesky" />
   
       <bean id="sessionFactory" 
               class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
           <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
           <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
       </bean> 
   
       <!-- 定义事务管理器（声明式的事务） --> 
       <bean id="transactionManager"
           class="org.springframework.orm.hibernate3.HibernateTransactionManager">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean>
   
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
           <tx:attributes>
               <tx:method name="*" propagation="REQUIRED" />
           </tx:attributes>
       </tx:advice>
   
       <aop:config>
           <aop:pointcut id="interceptorPointCuts"
               expression="execution(* com.bluesky.spring.dao.*.*(..))" />
           <aop:advisor advice-ref="txAdvice"
               pointcut-ref="interceptorPointCuts" />       
       </aop:config>     
   </beans>
   ```

5. 全注解

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-2.5.xsd
              http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
              http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
   
       <context:annotation-config />
       <context:component-scan base-package="com.bluesky" />
   
       <tx:annotation-driven transaction-manager="transactionManager"/>
   
       <bean id="sessionFactory" 
               class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
           <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
           <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
       </bean> 
   
       <!-- 定义事务管理器（声明式的事务） --> 
       <bean id="transactionManager"
           class="org.springframework.orm.hibernate3.HibernateTransactionManager">
           <property name="sessionFactory" ref="sessionFactory" />
       </bean>
   
   </beans>
   ```

   此时在DAO上需加上@Transactional注解，如下：

   ```java
   package com.bluesky.spring.dao;
   
   import java.util.List;
   
   import org.hibernate.SessionFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.orm.hibernate3.support.HibernateDaoSupport;
   import org.springframework.stereotype.Component;
   
   import com.bluesky.spring.domain.User;
   
   @Transactional
   @Component("userDao")
   public class UserDaoImpl extends HibernateDaoSupport implements UserDao {
   
       public List<User> listUsers() {
           return this.getSession().createQuery("from User").list();
       }  
   }
   ```

#### 深入

Spring事务管理我相信大家都用得很多，但可能仅仅局限于一个`@Transactional`注解或者在`XML`中配置事务相关的东西。不管怎么说，日常**可能**足够我们去用了。但作为程序员，无论是为了面试还是说更好把控自己写的代码，还是应该得多多了解一下Spring事务的一些细节。

这里我抛出几个问题，看大家能不能瞬间答得上：

- 如果**嵌套调用**含有事务的方法，在Spring事务管理中，这属于哪个知识点？
- 我们使用的框架可能是`Hibernate/JPA`或者是`Mybatis`，都知道的底层是需要一个`session/connection`对象来帮我们执行操作的。要保证事务的完整性，我们需要多组数据库操作要使用**同一个**`session/connection`对象，而我们又知道Spring IOC所管理的对象默认都是**单例**的，这为啥我们在使用的时候不会引发线程安全问题呢？内部Spring到底干了什么？
- 人家所说的BPP又是啥东西？
- Spring事务管理重要接口有哪几个？

我们都知道，Spring事务是Spring AOP的最佳实践之一，所以说AOP入门基础知识(简单配置，使用)是需要先知道的。说到AOP就不能不说AOP底层原理：动态代理设计模式。到这里，对AOP已经有一个基础的认识了。于是我们就可以使用XML/注解方式来配置Spring事务管理。

在IOC学习中，可以知道的是Spring中Bean的生命周期(引出BPP对象)并且IOC所管理的对象默认都是单例的：单例设计模式，单例对象如果有"**状态**"(有成员变量)，那么多线程访问这个单例对象，可能就造成线程不安全。那么何为线程安全？，解决线程安全有很多方式，但其中有一种：让每一个线程都拥有自己的一个变量：ThreadLocal

##### 事务嵌套

参考：https://zhuanlan.zhihu.com/p/38208248

**第一个例子**

在Service层抛出Exception，在Controller层捕获，那如果在Service中有异常，那会事务回滚吗？

```java
// Service方法

@Transactional
public Employee addEmployee() throws Exception {

    Employee employee = new Employee("3y", 23);
    employeeRepository.save(employee);
    // 假设这里出了Exception
    int i = 1 / 0;

    return employee;
}

// Controller调用
@RequestMapping("/add")
public Employee addEmployee() {
    Employee employee = null;
    try {
        employee = employeeService.addEmployee();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return employee;

}

```

可以回滚。

结论：如果是编译时异常不会自动回滚，**如果是运行时异常，那会自动回滚**！

**第二个例子**

我们都知道，带有`@Transactional`注解所包围的方法就能被Spring事务管理起来，那如果我在**当前类下使用一个没有事务的方法去调用一个有事务的方法**，那我们这次调用会怎么样？是否会有事务呢？

用代码来描述一下：

```java
// 没有事务的方法去调用有事务的方法
public Employee addEmployee2Controller() throws Exception {

    return this.addEmployee();
}

@Transactional
public Employee addEmployee() throws Exception {

    employeeRepository.deleteAll();
    Employee employee = new Employee("3y", 23);

    // 模拟异常
    int i = 1 / 0;

    return employee;
}
```

这跟Spring事务的传播机制**没有关系**，下面讲述一下：

- Spring事务管理用的是AOP，AOP底层用的是动态代理。所以如果我们在类或者方法上标注注解`@Transactional`，那么会生成一个**代理对象**。

  ![Qossij.md.png](https://s2.ax1x.com/2019/12/17/Qossij.md.png)

  显然地，我们拿到的是代理(Proxy)对象，调用`addEmployee2Controller()`方法，而`addEmployee2Controller()`方法的逻辑是`target.addEmployee()`，调用回原始对象(target)的`addEmployee()`。所以这次的调用**压根就没有事务存在**，更谈不上说Spring事务传播机制了。

如果是在本类中没有事务的方法来调用标注注解`@Transactional`方法，最后的结论是没有事务的。那如果我将这个标注注解的方法**移到**别的Service对象上，有没有事务？

```java
@Service
public class TestService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Transactional
    public Employee addEmployee() throws Exception {

        employeeRepository.deleteAll();

        Employee employee = new Employee("3y", 23);

        // 模拟异常
        int i = 1 / 0;

        return employee;
    }

}


@Service
public class EmployeeService {

    @Autowired
    private TestService testService;
    // 没有事务的方法去调用别的类有事务的方法
    public Employee addEmployee2Controller() throws Exception {
        return testService.addEmployee();
    }
}
```

事务起作用，因为我们用的是代理对象(Proxy)去调用`addEmployee()`方法，那就当然有事务了。

**spring同一个类中，一个方法调用另外一个注解**

```java
public class A {
    a() {
        b();
    }
    //声明事务
    @Transactional
    b() {
        sql操作
    }
}
```

如果这个时候直接通过调用a()方法，那么在b()方法运行错误的时候，是不会回滚代码的。原因如下：
类A会经过spring 中的AOP生成代理类ProxyA

```java
public class Proxy$A {
     A a = new A();
     
     a() {
         a.a();
     }
     
     b() {
        //开启事务
        startTransaction();
         a.b();
     }
     
}
```

然后在运行的时候，是直接调用代理类A（Proxy$A）中的a()方法，该a()方法直接调用原A类的a()方法，所以不会启动事务，最终导致事务失效。

有以下解决方法：

- 将b()方法抽出来，重新声明一个类，并且该类交由spring管理控制。
- 同时在a()上添加@Transactional注解或者在类上添加。
- 在原A类中的a()方法，改为 **((A)AopContext.currentProxy).b()**

##### Spring事务传播机制

> 如果**嵌套调用**含有事务的方法，在Spring事务管理中，这属于哪个知识点？

在当前**含有事务方法内部调用其他的方法**(无论该方法是否含有事务)，这就属于Spring事务传播机制的知识点范畴了。

Spring事务基于Spring AOP，Spring AOP底层用的动态代理，动态代理有两种方式：

- 基于接口代理(JDK代理)

- - 基于接口代理，凡是类的方法**非public修饰**，或者**用了static关键字**修饰，那这些方法都不能被Spring AOP增强

- 基于CGLib代理(子类代理)

- - 基于子类代理，凡是类的方法**使用了private、static、final修饰**，那这些方法都不能被Spring AOP增强

值得说明的是：那些不能被Spring AOP增强的方法**并不是不能**在事务环境下工作了。只要它们**被外层的事务方法调用了**，由于Spring事务管理的传播级别，内部方法也可以**工作**在外部方法所启动的**事务上下文中**。

##### 多线程问题

> 我们使用的框架可能是`Hibernate/JPA`或者是`Mybatis`，都知道的底层是需要一个`session/connection`对象来帮我们执行操作的。要保证事务的完整性，我们需要**多组数据库操作要使用同一个**`session/connection`对象，而我们又知道Spring IOC所管理的对象默认都是**单例**的，这为啥我们在使用的时候不会引发线程安全问题呢？内部Spring到底干了什么？

回想一下当年我们学Mybaits的时候，是怎么编写Session工具类？

![QosjeO.md.png](https://s2.ax1x.com/2019/12/17/QosjeO.md.png)

没错，用的就是ThreadLocal，同样地，Spring也是用的ThreadLocal。

以下内容来源《精通 Spring4.x》

> 我们知道在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean（如RequestContextHolder、**TransactionSynchronizationManager**、LocaleContextHolder等）中非线程安全状态的“状态性对象”采用ThreadLocal封装，让它们也成为线程安全的“状态性对象”，因此，有状态的Bean就能够以singleton的方式在多线程中工作。

我们可以试着点一下进去TransactionSynchronizationManager中看一下：

![QoypYd.md.png](https://s2.ax1x.com/2019/12/17/QoypYd.md.png)

##### 啥是BPP

参考：https://zhuanlan.zhihu.com/p/38208324

BBP的全称叫做：BeanPostProcessor，一般我们俗称**对象后处理器**

- 简单来说，通过BeanPostProcessor可以对我们的对象进行“**加工处理**”。

Spring管理Bean(或者说Bean的生命周期)也是一个**常考**的知识点，我在秋招也**重新**整理了一下步骤，因为比较重要，所以还是在这里贴一下吧：

1. ResouceLoader加载配置信息

2. BeanDefintionReader解析配置信息，生成一个一个的BeanDefintion

3. BeanDefintion由BeanDefintionRegistry管理起来

4. BeanFactoryPostProcessor对配置信息进行加工(也就是处理配置的信息，一般通过PropertyPlaceholderConfigurer来实现)

5. 实例化Bean

6. 如果该Bean`配置/实现`了InstantiationAwareBean，则调用对应的方法

7. 使用BeanWarpper来完成对象之间的属性配置(依赖)

8. 如果该Bean`配置/实现了`Aware接口，则调用对应的方法

9. 如果该Bean配置了BeanPostProcessor的before方法，则调用

10. 如果该Bean配置了`init-method`或者实现InstantiationBean，则调用对应的方法

11. 如果该Bean配置了BeanPostProcessor的after方法，则调用

12. 将对象放入到HashMap中

13. 最后如果配置了destroy或者DisposableBean的方法，则执行销毁操作

    ![QoykOf.md.png](https://s2.ax1x.com/2019/12/17/QoykOf.md.png)

其中也有关于BPP图片：

![QoyZTg.md.png](https://s2.ax1x.com/2019/12/17/QoyZTg.md.png)



Spring AOP编程底层通过的是动态代理技术，在调用的时候肯定用的是**代理对象**。那么Spring是怎么做的呢？

> 我只需要写一个BPP，在postProcessBeforeInitialization或者postProcessAfterInitialization方法中，对对象进行判断，看他需不需要织入切面逻辑，如果需要，那我就根据这个对象，生成一个代理对象，然后返回这个代理对象，那么最终注入容器的，自然就是代理对象了。

Spring提供了BeanPostProcessor，就是让我们可以对有需要的对象进行“**加工处理**”啊！

##### 认识Spring事务几个重要的接口

Spring事务可以分为两种：

- 编程式事务(通过代码的方式来实现事务)
- 声明式事务(通过配置的方式来实现事务)

编程式事务在Spring实现相对简单一些，而声明式事务因为封装了大量的东西(一般我们使用简单，里头都非常复杂)，所以声明式事务实现要难得多。

在编程式事务中有以下几个重要的了接口：

- TransactionDefinition：定义了Spring兼容的**事务属性**(比如事务隔离级别、事务传播、事务超时、是否只读状态)

- TransactionStatus：代表了事务的具体**运行状态**(获取事务运行状态的信息，也可以通过该接口**间接**回滚事务等操作)

- PlatformTransactionManager：事务管理器接口(定义了一组行为，具体实现交由不同的持久化框架来完成---**类比**JDBC)

  ![QoyMpn.md.png](https://s2.ax1x.com/2019/12/17/QoyMpn.md.png)

在声明式事务中，除了TransactionStatus和PlatformTransactionManager接口，还有几个重要的接口：

- TransactionProxyFactoryBean：生成代理对象
- TransactionInterceptor：实现对象的拦截
- TransactionAttrubute：事务配置的数据

#### 参考

1. [Spring事务配置的五种方式](http://www.blogjava.net/robbie/archive/2009/04/05/264003.html)
2. [Spring中的事务管理实例详解](https://www.jb51.net/article/57589.htm)
3. [一文带你看懂Spring事务！](http://blog.itpub.net/69900354/viewspace-2565243/)
4. [那些年，我们一起追的Spring](https://zhuanlan.zhihu.com/p/41961670)

