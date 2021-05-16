---
title: JDBC 详解
date: 2019-04-07 10:18:59
tags:
 - Java
 - 数据库
categories:
 - Java
 - 基础
---
### 概述

**什么是JDBC**

JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序

JDBC制定了统一访问各类关系数据库的标准接口，为各个数据库厂商提供了标准接口的实现。
<!--more-->

JDBC需要连接驱动，驱动是两个设备要进行通信，满足一定通信数据格式，数据格式由设备提供商规定，设备提供商为设备提供驱动软件，通过软件可以与该设备进行通信。

JDBC规范将驱动程序归结为以下几类（选自Core Java Volume Ⅱ——Advanced Features）：

- 第一类驱动程序将JDBC翻译成ODBC，然后使用一个ODBC驱动程序与数据库进行通信。
- 第二类驱动程序是由部分Java程序和部分本地代码组成的，用于与数据库的客户端API进行通信。
- 第三类驱动程序是纯Java客户端类库，它使用一种与具体数据库无关的协议将数据库请求发送给服务器构件，然后该构件再将数据库请求翻译成数据库相关的协议。
- 第四类驱动程序是纯Java类库，它将JDBC请求直接翻译成数据库相关的协议。

**数据库驱动**

Java提供访问数据库规范称为JDBC，而生产厂商提供规范的实现类称为驱动。

我们安装好数据库之后，我们的应用程序也是不能直接使用数据库的，必须要通过相应的数据库驱动程序，通过驱动程序去和数据库打交道。其实也就是数据库厂商的JDBC接口实现，即对Connection等接口的实现类的jar文件。

![1.png](https://i.loli.net/2019/04/16/5cb55e9e9ccb9.png)

JDBC是接口，驱动是接口的实现，没有驱动将无法完成数据库连接，从而不能操作数据库！每个数据库厂商都需要提供自己的驱动，用来连接自己公司的数据库，也就是说驱动一般都由数据库生成厂商提供。

### 常用接口

提供的接口包括：
- JAVA API：提供对JDBC的管理链接；
- JAVA Driver API：支持JDBC管理到驱动器连接。
- DriverManager：这个类管理数据库驱动程序的列表，查看加载的驱动是否符合JAVA Driver API的规范。
- Connection：与数据库中的所有的通信是通过唯一的连接对象。
- Statement：把创建的SQL对象，转而存储到数据库当中。
- ResultSet：它是一个迭代器，用于检索查询数据。

#### Driver接口

Driver接口由数据库厂家提供，作为java开发人员，只需要使用Driver接口就可以了。在编程中要连接数据库，必须先装载特定厂商的数据库驱动程序，不同的数据库有不同的装载方法。如：
```
装载MySql驱动：Class.forName("com.mysql.jdbc.Driver");
装载Oracle驱动：Class.forName("oracle.jdbc.driver.OracleDriver");
```
#### Connection接口

Connection与特定数据库的连接（会话），在连接上下文中执行sql语句并返回结果。DriverManager.getConnection(url, user, password)方法建立在JDBC URL中定义的数据库Connection连接上。
```
连接MySql数据库：Connection conn = DriverManager.getConnection("jdbc:mysql://host:port/database", "user", "password");
连接Oracle数据库：Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@host:port:database", "user", "password");
连接SqlServer数据库：Connection conn = DriverManager.getConnection("jdbc:microsoft:sqlserver://host:port; DatabaseName=database", "user", "password");
```
常用方法：
```
createStatement()：创建向数据库发送sql的statement对象。
prepareStatement(sql) ：创建向数据库发送预编译sql的PrepareSatement对象。
prepareCall(sql)：创建执行存储过程的callableStatement对象。
setAutoCommit(boolean autoCommit)：设置事务是否自动提交。
commit() ：在链接上提交事务。
rollback() ：在此链接上回滚事务。
```
#### Statement接口

用于执行静态SQL语句并返回它所生成结果的对象。

三种Statement类：
```
Statement：由createStatement创建，用于发送简单的SQL语句（不带参数）。
PreparedStatement ：继承自Statement接口，由preparedStatement创建，用于发送含有一个或多个参数的SQL语句。PreparedStatement对象比Statement对象的效率更高，并且可以防止SQL注入，所以我们一般都使用PreparedStatement。
CallableStatement：继承自PreparedStatement接口，由方法prepareCall创建，用于调用存储过程。
```
常用Statement方法：
```
execute(String sql):运行语句，返回是否有结果集
executeQuery(String sql)：运行select语句，返回ResultSet结果集。
executeUpdate(String sql)：运行insert/update/delete操作，返回更新的行数。
addBatch(String sql) ：把多条sql语句放到一个批处理中。
executeBatch()：向数据库发送一批sql语句执行。
```
#### ResultSet接口

ResultSet提供检索不同类型字段的方法，常用的有：
```
getString(int index)、getString(String columnName)：获得在数据库里是varchar、char等类型的数据对象。
getFloat(int index)、getFloat(String columnName)：获得在数据库里是Float类型的数据对象。
getDate(int index)、getDate(String columnName)：获得在数据库里是Date类型的数据。
getBoolean(int index)、getBoolean(String columnName)：获得在数据库里是Boolean类型的数据。
getObject(int index)、getObject(String columnName)：获取在数据库里任意类型的数据。
```
ResultSet还提供了对结果集进行滚动的方法：
```
next()：移动到下一行
Previous()：移动到前一行
absolute(int row)：移动到指定行
beforeFirst()：移动resultSet的最前面。
afterLast() ：移动到resultSet的最后面。
```

使用后依次关闭对象及连接：ResultSet → Statement → Connection

### JDBC数据类型
在使用Java程序中的数据之前，JDBC会将Java数据类型转换成相对应的JDBC数据类型。它们之间有一个默认的对应关系，能够保证在不同的数据库实现和驱动之间的一致性。

下表列出了它们的对应关系：

![](https://i.loli.net/2019/04/16/5cb574469b2c8.png)

**DECIMAL对应BigDecimal**

SQL和Java对于空值（null）有不同的处理方式。在使用SQL处理一些Java中的空值时，我们最好能够遵循一些最佳实践，比如避免使用原始类型（primitive type），因为它们默认值不能为空而是会依据类型设定默认值，比如int型默认值为0、布尔型（boolean）默认值为false，等等。

与此相反，我们建议使用这些原始类型的包装类。类似这样的情况，类ResultSet的wasNull()方法就会非常有用。

#### util.Date与sql.Date

Java中有两个Date类，一个是java.util.Date通常情况下用它获取当前时间或构造时间，另一个是java.sql.Date是针对SQL语句使用的，它只包含日期而没有时间部分。两个类型的时间可以相互转化。

**util.Date转sql.Date** 

```
Date utilDate = new Date();//util.Date
System.out.println("utilDate : " + utilDate);
//util.Date转sql.Date
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
System.out.println("sqlDate : " + sqlDate);
java.sql.Timestamp sqlDate = new java.sql.Timestamp(utilDate.getTime());//uilt date转sql date
System.out.println("sqlDate : " + sqlDate);
```

java.sql包下给出三个与数据库相关的日期时间类型: 

- Date：表示日期，只有年月日，没有时分秒。会丢失时间； 

- Time：表示时间，只有时分秒，没有年月日。会丢失日期； 

- Timestamp：表示时间戳，有年月日时分秒，以及毫秒。

**sql.Date转util.Date**

```
System.out.println("*********util.Date转sql.Date*********");
Date utilDate = new Date();//util.Date
System.out.println("utilDate : " + utilDate);
Timestamp sqlDate = new Timestamp(utilDate.getTime());//util.Date转sql.Date
System.out.println("sqlDate : " + sqlDate);

System.out.println("*********sql.Date转util.Date*********");
System.out.println("sqlDate : " + sqlDate);
Date date = new Date(sqlDate.getTime());//sql.Date转util.Date
/*
java.util.Date date = new java.util.Date(sqlDate.getTime());
*/
System.out.println("utilDate : " + date);
```

**同时util.Date和sql.Date都可以用SimpleDateFormat格式化** 

```
Date utilDate = new Date();//uilt.Date
System.out.println("utilDate : " + utilDate);

SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
System.out.println("format : " + format.format(utilDate));

System.out.println("**********************************************");

Timestamp sqlDate = new Timestamp(utilDate.getTime());//uilt.Date转sql.Date
System.out.println("sqlDate : " + sqlDate);
System.out.println("format : " + format.format(sqlDate));
```

### 使用JDBC的步骤

加载JDBC驱动程序 → 建立数据库连接Connection → 创建执行SQL的语句Statement → 处理执行结果ResultSet → 释放资源

**常用数据库JDBC URL格式：**

1. sqLite
 
驱动程序包名：`sqlitejdbc-v056.jar`
驱动程序类名: `org.sqlite.JDBC`
JDBC URL: `jdbc:sqlite:c:\*.db`
默认端口 无
 
2. Microsoft SQL Server

  **Microsoft SQL Server JDBC Driver （一般用来连接 SQLServer 2000）**
  驱动程序包名：`msbase.jar mssqlserver.jar msutil.jar`
  驱动程序类名: `com.microsoft.jdbc.sqlserver.SQLServerDriver`
  JDBC URL: `jdbc:microsoft:sqlserver://<server_name>:<port>`
  默认端口1433，如果服务器使用默认端口则port可以省略
  **Microsoft SQL Server 2005 JDBC Driver**
  驱动程序包名：`sqljdbc.jar`
  驱动程序类名: `com.microsoft.sqlserver.jdbc.SQLServerDriver`
  JDBC URL:` jdbc:sqlserver://<server_name>:<port>`
  默认端口1433，如果服务器使用默认端口则port可以省略

3. Oracle
  Oracle Thin JDBC Driver
  驱动程序包名：`ojdbc14.jar`
  驱动程序类名: `oracle.jdbc.driver.OracleDriver`
  JDBC URL:
  `jdbc:oracle:thin:@//<host>:<port>/ServiceName`
  或
  `jdbc:oracle:thin:@<host>:<port>:<SID>`

4. IBM DB2
  **IBM DB2 Universal Driver Type 4**
  驱动程序包名：`db2jcc.jar db2jcc_license_cu.jar`
  驱动程序类名: `com.ibm.db2.jcc.DB2Driver`
  JDBC URL: `jdbc:db2://<host>[:<port>]/<database_name>`
  **IBM DB2 Universal Driver Type 2**
  驱动程序包名：`db2jcc.jar db2jcc_license_cu.jar`
  驱动程序类名:` com.ibm.db2.jcc.DB2Driver`
  JDBC URL: `jdbc:db2:<database_name>`

5. H2 

  ```
  driverClassName=org.h2.Driver
  ```

  在代码中访问数据库  `JDBC_URL = "jdbc:h2:~/test";`

  内存模式访问: `JDBC_URL = "jdbc:h2:mem:test";`

  远程模式连接:

  ```
  jdbc:h2:tcp://<server>[:<port>]/[<path>]<databaseName>
  jdbc:h2:tcp://localhost/~/test 使用用户主目录
  jdbc:h2:tcp://localhost//data/test 使用绝对路径
  ```

6. MySQL
    MySQL Connector/J Driver
    驱动程序包名：`mysql-connector-java-x.x.xx-bin.jar`
    驱动程序类名:` com.mysql.jdbc.Driver`
    JDBC URL:` jdbc:mysql://<host>:<port>/<database_name>`
    默认端口3306，如果服务器使用默认端口则port可以省略
    MySQL Connector/J Driver 允许在URL中添加额外的连接属性
```html
jdbc:mysql://<host>:<port>/<database_name>?property1=value1&property2=value2
```
注意： 需要操作记录为了避免乱码应该加上属性 useUnicode=true&characterEncoding=utf8 ，比如
```html
jdbc:mysql://192.168.177.129:3306/report?useUnicode=true&characterEncoding=utf8
```


#### 注册驱动 (只做一次)

方式一：
```java
Class.forName(“com.MySQL.jdbc.Driver”);
```
推荐这种方式，不会对具体的驱动类产生依赖。

方式二：
```java
DriverManager.registerDriver(com.mysql.jdbc.Driver);
```
会造成DriverManager中产生两个一样的驱动，并会对具体的驱动类产生依赖。

#### 建立连接
```java
Connection conn = DriverManager.getConnection(url, user, password);
```
URL用于标识数据库的位置，通过URL地址告诉JDBC程序连接哪个数据库，URL的写法为：

![1.png](https://i.loli.net/2019/04/16/5cb56002c88dc.png)

如果URL中给出用户名和密码，就不用再填写。


**URL常用参数：**

![](https://i.loli.net/2019/04/16/5cb5622c33257.png)


#### 创建执行SQL语句的statement

```java
//Statement  
String id = "5";
String sql = "delete from table where id=" +  id;
Statement st = conn.createStatement();  
st.executeQuery(sql);  
//存在sql注入的危险
//如果用户传入的id为“5 or 1=1”，那么将删除表中的所有记录

//PreparedStatement 有效的防止sql注入(SQL语句在程序运行前已经进行了预编译,当运行时动态地把参数传给PreprareStatement时，即使参数里有敏感字符如 or '1=1'也数据库会作为一个参数一个字段的属性值来处理而不会作为一个SQL指令)
String sql = “insert into user (name,pwd) values(?,?)”;  
PreparedStatement ps = conn.preparedStatement(sql);  
ps.setString(1, “col_value”);  //占位符顺序从1开始
ps.setString(2, “123456”); //也可以使用setObject
ps.executeQuery();
```
**Statement、 PreparedStatement 、CallableStatement 区别和联系**
1. Statement、PreparedStatement和CallableStatement都是接口(interface)。
2. Statement继承自Wrapper、PreparedStatement继承自Statement、CallableStatement继承自PreparedStatement。
3. Statement接口提供了执行语句和获取结果的基本方法；
    PreparedStatement接口添加了处理 IN 参数的方法；
    CallableStatement接口添加了处理 OUT 参数的方法。
4. - Statement：普通的不带参的查询SQL；支持批量更新,批量删除;
    - PreparedStatement：可变参数的SQL,编译一次,执行多次,效率高;安全性好，有效防止Sql注入等问题;支持批量更新,批量删除;
    - CallableStatement：继承自PreparedStatement,支持带参数的SQL操作;支持调用存储过程,提供了对输出和输入/输出参数(INOUT)的支持;

5. Statement每次执行sql语句，数据库都要执行sql语句的编译 ，最好用于仅执行一次查询并返回结果的情形时，效率高于PreparedStatement。

**PreparedStatement**是预编译的，使用PreparedStatement有几个好处

1. 在执行可变参数的一条SQL时，PreparedStatement比Statement的效率高，因为DBMS预编译一条SQL当然会比多次编译一条SQL的效率要高。
2. 安全性好，有效防止Sql注入等问题。
3.  对于多次重复执行的语句，使用PreparedStament效率会更高一点，并且在这种情况下也比较适合使用batch；
4.  代码的可读性和可维护性。

**Statement常见方法**

- executeQuery：返回结果集(ResultSet),数据查询。

- executeUpdate: 执行给定SQL语句,该语句可能为 INSERT、UPDATE 或 DELETE 语句， 或者不返回任何内容的SQL语句（如 SQL DDL 语句）,数据更新。

- execute: 可用于执行任何SQL语句，返回一个boolean值， 表明执行该SQL语句是否返回了ResultSet。如果执行后第一个结果是ResultSet，则返回true，否则返回false。

示例：
```java
//Statement用法:  
String sql = "select seq_orderdetailid.nextval as test dual";  
Statement stat1=conn.createStatement();  
ResultSet rs1 = stat1.executeQuery(sql);  
if ( rs1.next() ) {  
    id = rs1.getLong(1);  
}  

//INOUT参数使用：  
CallableStatement cstmt = conn.prepareCall("{call revise_total(?)}");  
cstmt.setByte(1, 25);  
cstmt.registerOutParameter(1, java.sql.Types.TINYINT);  
cstmt.executeUpdate();  
byte x = cstmt.getByte(1);  

//Statement的Batch使用:  
Statement stmt  = conn.createStatement();  
String sql = null;  
for(int i =0;i<20;i++){  
    sql = "insert into test(id,name)values("+i+","+i+"_name)";  
    stmt.addBatch(sql);  
}  
stmt.executeBatch();  

//PreparedStatement的Batch使用:  
PreparedStatement pstmt  = con.prepareStatement("UPDATE EMPLOYEES  SET SALARY = ? WHERE ID =?");  
for(int i =0;i<length;i++){  
    pstmt.setBigDecimal(1, param1[i]);  
    pstmt.setInt(2, param2[i]);  
    pstmt.addBatch();  
}  
pstmt.executeBatch();  

//PreparedStatement用法:  
PreparedStatement pstmt  = con.prepareStatement("UPDATE EMPLOYEES  SET SALARY = ? WHERE ID =?");  
pstmt.setBigDecimal(1, 153.00);  
pstmt.setInt(2, 1102);  
pstmt. executeUpdate()
```

**execute与executeUpdate的区别**

- 相同点：都可以执行增加，删除，修改

- 不同1：

  execute可以执行查询语句,然后通过getResultSet，把结果集取出来;executeUpdate不能执行查询语句

- 不同2:

  execute返回boolean类型，true表示执行的是查询语句，false表示执行的是insert,delete,update等等;

  executeUpdate返回的是int，表示有多少条数据受到了影响

示例：

```java

// 相同点：都可以执行增加，删除，修改

s.execute(sqlInsert);
s.execute(sqlDelete);
s.execute(sqlUpdate);
s.executeUpdate(sqlInsert);
s.executeUpdate(sqlDelete);
s.executeUpdate(sqlUpdate);

// 不同1：execute可以执行查询语句
// 然后通过getResultSet，把结果集取出来
String sqlSelect = "select * from hero";

s.execute(sqlSelect);
ResultSet rs = s.getResultSet();
while (rs.next()) {
    System.out.println(rs.getInt("id"));
}

// executeUpdate不能执行查询语句
// s.executeUpdate(sqlSelect);

// 不同2:
// execute返回boolean类型，true表示执行的是查询语句，false表示执行的是insert,delete,update等等
boolean isSelect = s.execute(sqlSelect);
System.out.println(isSelect);

// executeUpdate返回的是int，表示有多少条数据受到了影响
String sqlUpdate = "update Hero set hp = 300 where id < 100";
int number = s.executeUpdate(sqlUpdate);
System.out.println(number);
```

#### 处理执行结果(ResultSet)

对数据库的查询操作，一般需要返回查询结果，在程序中，JDBC为我们提供了ResultSet接口来专门处理查询结果集。

Statement通过以下方法执行一个查询操作：
```java
ResultSet executeQuery(String sql) throws SQLException
```
单词Query就是查询的意思。函数的返回类型是ResultSet，实际上查询的数据并不在ResultSet里面，依然是在数据库里，ResultSet中的next()方法类似于一个指针，指向查询的结果，然后不断遍历。所以这就要求连接不能断开。

ResultSet接口常用方法：
```java
boolean next()     //遍历时，判断是否有下一个结果
int getInt(String columnLabel)
int getInt(int columnIndex)
Date getDate(String columnLabel)
Date getDate(int columnIndex)
String getString(String columnLabel)
String getString(int columnIndex)
```

示例：
```java
ResultSet rs = ps.executeQuery();  
 While(rs.next()){  
     rs.getString(“col_name”);  
     rs.getInt(1);  
     //…
}
```
#### 释放资源
```java
//数据库连接（Connection）非常耗资源，尽量晚创建，尽量早的释放
 //都要加try catch 以防前面关闭出错，后面的就不执行了
try {
    if (rs != null) {
        rs.close();
    }
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    try {
        if (st != null) {
            st.close();
        }
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        try {
            if (conn != null) {
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
### 批处理

实际开发中需要向数据库发送多条SQL语句，这时，如果逐条执行SQL语句，效率会很低，因此可以使用JDBC提供的批处理机制。Statement和PreparedStatemen都实现了批处理。

JDBC实现批处理有两种方式：**statement和preparedstatement**

**批量更新API**

- addBatch(String sql)：Statement类的方法, 可以将多条sql语句添加Statement对象的SQL语句列表中
- addBatch()：PreparedStatement类的方法, 可以将多条预编译的sql语句添加到PreparedStatement对象的SQL语句列表中
- executeBatch()：把Statement对象或PreparedStatement对象语句列表中的所有SQL语句发送给数据库进行处理
- clearBatch()：清空当前SQL语句列表

**使用Statement完成批处理**

范例：

- 使用Statement对象添加要批量执行SQL语句，如下：

  ```java
  Statement.addBatch(sql1);
   Statement.addBatch(sql2);
   Statement.addBatch(sql3);
  ```

- 执行批处理SQL语句：`Statement.executeBatch();`

- 清除批处理命令：`Statement.clearBatch();`

采用Statement.addBatch(sql)方式实现批处理的优缺点

优点：可以向数据库发送多条不同的ＳＱＬ语句。

缺点：SQL语句没有预编译；当向数据库发送多条语句相同，但仅参数不同的SQL语句时，需重复写上很多条SQL语句。

**使用PreparedStatement完成批处理**

范例:

```java
 @Test
    public void testJdbcBatchHandleByPrepareStatement(){
        long starttime = System.currentTimeMillis();
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        
        try{
            conn = JdbcUtils.getConnection();
            String sql = "insert into testbatch(id,name) values(?,?)";
            st = conn.prepareStatement(sql);
            for(int i=1;i<1000008;i++){  //i=1000  2000
                st.setInt(1, i);
                st.setString(2, "aa" + i);
                st.addBatch();
                if(i%1000==0){
                    st.executeBatch();
                    st.clearBatch();//避免内存溢出异常
                }
            }
            st.executeBatch();
        }catch (Exception e) {
            e.printStackTrace();
        }finally{
            JdbcUtils.release(conn, st, rs);
        }
        long endtime = System.currentTimeMillis();
        System.out.println("程序花费时间：" + (endtime-starttime)/1000 + "秒！！");
    }
```

采用PreparedStatement.addBatch()方式实现批处理的优缺点

优点：发送的是预编译后的SQL语句，执行效率高。

缺点：只能应用在SQL语句相同，但参数不同的批处理中。因此此种形式的批处理经常用于在同一个表中批量插入数据，或批量更新表的数据。

### 事务

事务的ACID特性：

满足ACID特性的操作，我们可以说它是一个事物。

- 原子性：该操作是最小逻辑单元整体，已经不可分隔。
- 一致性：要么所有都执行，要么所有都不执行。
- 隔离性：多个事务相互隔离，互不影响。
- 持久性：事物的执行结果永久生效。

在JDBC中可以调用Connection对象的setAutoCommit(false)这个接口，将commit()之前的所有操作都看成是一个事物。同时，如果事务执行过程中发生异常，可以调用rollback()接口进行回滚到事务开始之前的状态。

![1 (1).png](https://i.loli.net/2019/04/16/5cb56ee7f409f.png)

首先得清楚什么时候使用事务。

当你需要一次执行多条SQL语句时，可以使用事务。通俗一点说，就是，如果这几条SQL语句全部执行成功，则才对数据库进行一次更新，如果有一条SQL语句执行失败，则这几条SQL语句全部不进行执行，这个时候需要用到事务。

 其次才是事务的具体使用。

1. 获取对数据库的连接
2. 设置事务不自动提交（默认情况是自动提交的）
   `conn.setAutoCommit(false);  ` 其中conn是第一步获取的随数据库的连接对象。
3. 把想要一次性提交的几个sql语句用事务进行提交

```java
   Statement stmt = null;
   stmt = conn.createStatement();
   stmt.executeUpdate(sql1);
   stmt.executeUpdate(Sql2);
//   ...
   conn.commit();   //使用commit提交事务
```

4. 捕获异常，进行数据的回滚（回滚一般写在catch块中）

```java
  catch（Exception e）
  {
      ...
      try
      {
         conn.rollback();
      } catch(Exception e)
      {...}
   }
```

5. 把事务再改成自动提交（默认状态）

```java
 conn.setAutoCommit(true);
```

##### 事务隔离级别

**事务的并发读问题**

- 脏读：读取到另外一个事务未提交数据（不允许出来的事）；
- 不可重复读：两次读取不一致；
- 幻读（虚读）：读到另一事务已提交数据。

**并发事务问题**

因为并发事务导致的问题大致有5类，其中两类是更新问题三类是读问题。

- 脏读（dirty read）：读到另一个事务的未提交新数据，即读取到了脏数据；

  例子：A向B转账，A执行了转账语句，但A还没有提交事务，B读取数据，发现自己账户钱变多了！B跟A说，我已经收到钱了。A回滚事务【rollback】，等B再查看账户的钱时，发现钱并没有多。

  下面的三种个隔离级别都可以防止：

  - Serializable【TRANSACTION_SERIALIZABLE】
  - Repeatable read【TRANSACTION_REPEATABLE_READ】
  - Read committed【TRANSACTION_READ_COMMITTED】

- 不可重复读（unrepeatable）：对同一记录的两次读取不一致，因为另一事务对该记录做了修改；

- 幻读（虚读）（phantom read）：对同一张表的两次查询不一致，因为另一事务插入了一条记录。只有TRANSACTION_SERIALIZABLE隔离级别才能防止产生幻读。

**四大隔离级别**

 4个等级的事务隔离级别，在相同的数据环境下，使用相同的输入，执行相同的工作，根据不同的隔离级别，可以导致不同的结果。不同事务隔离级别能够解决的数据并发问题的能力是不同的。

1. SERIALIZABLE(串行化)

   - 不会出现任何并发问题，因为它是对同一数据的访问是串行的，非并发访问的；

   - 性能最差

2. REPEATABLE READ(可重复读)（MySQL）

   - 防止脏读和不可重复读，不能处理幻读

   - 性能比SERIALIZABLE好

3. READ COMMITTED(读已提交数据)（Oracle）

   - 防止脏读，不能处理不可重复读和幻读；

   - 性能比REPEATABLE READ好

4. READ UNCOMMITTED(读未提交数据)

   - 可能出现任何事物并发问题，什么都不处理。

   - 性能最好

**MySQL隔离级别**

MySQL的默认隔离级别为Repeatable read,可以通过下面语句查看：

```mysql
SELECT @@`TX_ISOLATION`;
```

也可以通过下面语句来设置当前连接的隔离级别：

```mysql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ ;
-- [4选1]
```

**JDBC设置隔离级别**

con.setTransactionIsolation(int level) :参数可选值如下：

- Connection.TRANSACTION_READ_UNCOMMITTED；
- Connection.TRANSACTION_READ_COMMITTED；
- Connection.TRANSACTION_REPEATABLE_READ；
- Connection.TRANSACTION_READ_SERIALIZABLE。

**解答各种疑问**

1. 回滚的目的是什么呢？

目的是使得sql1，sql2。。。等操作要么全部执行成功，要么全部执行不成功，这也是为什     么把这几个sql语句当成一个事务来处理的目的。

2. 回滚从哪里开始回滚，我如何控制回滚的起始点。

其实是可以设置存储点的

```java
 Savepoint piont = conn.setSavepoint();
 conn.rollback(point);
```

如果你没有设置存储点，他会回滚到你设置禁止事务自动提交的时候，因为你是先设置禁止自动提交的，再进行executeUpdate（sql）的，所以他会回滚到你的所有执行的这几个sql语句前的状态。

### 其他操作

**获取自增长id**

在Statement通过execute或者executeUpdate执行完插入语句后，MySQL会为新插入的数据分配一个自增长id，(前提是这个表的id设置为了自增长,在Mysql创建表的时候，AUTO_INCREMENT就表示自增长)
 ```sql
CREATE TABLE hero (
  id int(11) AUTO_INCREMENT,
  ...
}
 ```

但是无论是execute还是executeUpdate都不会返回这个自增长id是多少。需要通过StatementgetGeneratedKeys获取该id

PreparedStatement构造函数后面加了个Statement.RETURN_GENERATED_KEYS参数，以确保会返回自增长ID。 通常情况下不需要加这个，有的时候需要加，所以先加上，保险一些

 ```java
PreparedStatement ps = c.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)

//...

// 在执行完插入语句后，MySQL会为新插入的数据分配一个自增长id
// JDBC通过getGeneratedKeys获取该id
ResultSet rs = ps.getGeneratedKeys();
 ```
**获取表的元数据**
元数据概念：
和数据库服务器相关的数据，比如数据库版本，有哪些表，表有哪些字段，字段类型是什么等等。

```java
// 查看数据库层面的元数据
// 即数据库服务器版本，驱动版本，都有哪些数据库等等

DatabaseMetaData dbmd = c.getMetaData();

// 获取数据库服务器产品名称
System.out.println("数据库产品名称:\t"+dbmd.getDatabaseProductName());
// 获取数据库服务器产品版本号
System.out.println("数据库产品版本:\t"+dbmd.getDatabaseProductVersion());
// 获取数据库服务器用作类别和表名之间的分隔符 如test.user
System.out.println("数据库和表分隔符:\t"+dbmd.getCatalogSeparator());
// 获取驱动版本
System.out.println("驱动版本:\t"+dbmd.getDriverVersion());

System.out.println("可用的数据库列表：");
// 获取数据库名称
ResultSet rs = dbmd.getCatalogs();

while (rs.next()) {
    System.out.println("数据库名称:\t"+rs.getString(1));
}
```

**SQL注入问题**

假设有登录案例SQL语句如下**:**

```sql
SELECT * FROM 用户表 WHERE NAME = 用户输入的用户名 AND PASSWORD = 用户输的密码;
```

此时，当用户输入正确的账号与密码后，查询到了信息则让用户登录。但是当用户输入的账号为XXX 密码为：XXX’  OR ‘a’=’a时，则真正执行的代码变为：

```sql
SELECT * FROM 用户表 WHERE NAME = ‘XXX’ AND PASSWORD =’ XXX’  OR ’a’=’a’;
```

此时，上述查询语句时永远可以查询出结果的。那么用户就直接登录成功了，显然我们不希望看到这样的结果，这便是SQL注入问题。

为此，我们**使用PreparedStatement来解决对应的问题**。

#### JDBC版本变动

MySQL 版本和 mysql-connector-java 版本对应关系如下，MySQL官方也是推荐使用 mysql-connector-java-8.X.jar 去连接 MySQL 8.0 的版本

| Connector/J version | Driver Type | JDBC version       | MySQL Server version | Status                                    |
| ------------------- | ----------- | ------------------ | -------------------- | ----------------------------------------- |
| 5.1                 | 4           | 3.0, 4.0, 4.1, 4.2 | 5.6*, 5.7*, 8.0*     | General availability                      |
| 8.0                 | 4           | 4.2                | 5.6, 5.7, 8.0        | General availability. Recommended version |

高版本MySQL（5.7,5.8）的JDBC连接新问题：

```
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>8.0.11</version>
 </dependency>
```

然后遇到了驱动问题、SSL安全访问的问题和时区问题。

```
"com.mysql.jdbc.Driver" --》com.mysql.cj.jdbc.Driver
```

这个是安全连接的问题，在sql连接字符串中，添加关于不使用SSL访问数据数据库的说明：useSSL=false。

这个是时区问题，在访问字符串中添加关于时区的设置说明：serverTimezone=UTC。

如：

```
jdbc:mysql://localhost:3306/dbname?characterEncoding=utf-8&useSSL=true&serverTimezone=GMT
```

**mysql-connector-java升级到8.0之后的一些兼容问题**

基本还是兼容的，但是有一些不兼容的地方，目前已经笔者知道的有2点：

1. 如果数据库表自增主键id是bigint类型，原来版本执行后返回的id是Long类型，现在改为了BigInteger类型，如果是使用mybatis基本没问题；如果是自定义的JDBC框架就要格外注意看处理是否有问题，类型是否存在不匹配导致问题。

2. 日期类型的字段处理可能存在问题，如表中字段为TIMESTAMP，之前查询返回能够返回毫秒值，升级后不再返回。如果在查询时有使用日期转换函数转换为String，并且对毫秒值进行了处理，那么升级后会报错，需要去掉对毫秒值得处理。如果查询返回直接映射为Date就没问题。

### DBUtils

DBUtils是apache下的一个小巧的JDBC轻量级封装的工具包，其最核心的特性是结果集的封装，可以直接将查询出来的结果集封装成JavaBean，这就为我们做了最枯燥乏味、最容易出错的一大部分工作。

maven配置：

```xml
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.7</version>
</dependency>
```

#### 为什么需要Dbutils ？

在使用Dbutils 之前，我们Dao层使用的技术是JDBC，那么分析一下JDBC的弊端：
（1）数据库链接对象、sql语句操作对象，封装结果集对象，这三大对象会重复定义
（2）封装数据的代码重复，而且操作复杂，代码量大
（3）释放资源的代码重复
结果：（1）程序员在开发的时候，有大量的重复劳动。（2）开发的周期长，效率低

#### 核心类QueryRunner

- 带有Connection的
  - Int  update(Connection conn, String sql, Object param);执行更新带一个占位符的sql
  - Int  update(Connection conn, String sql, Object…  param);执行更新带多个占位符的sql
  - Int[]  batch(Connection conn, String sql, Object[][] params) 批处理
  - T  query(Connection conn ,String sql, ResultSetHandler<T> rsh, Object... params) 查询方法
- 不带有Connection的
  - Int  update( String sql, Object param); 执行更新带一个占位符的sql
  - Int  update( String sql, Object…  param);执行更新带多个占位符的sql
  - Int[]  batch( String sql, Object[][] params); 批处理

注意： 如果调用DbUtils组件的操作数据库方法，没有传入连接对象，那么在实例化QueryRunner对象的时候需要传入数据源对象：

```java
    QueryRunner qr = new QueryRunner(DataSource ds);
```

#### 更新操作（包括delete 、insert、 update）

1. 删除

   ```java
   import org.apache.commons.dbutils.QueryRunner;
   import utils.jdbcUtil;

   import java.sql.*;
   
   /**
    * Created by pc on 2017/9/9.
    */
   public class DbUtils {
       public static void main(String [] args){
               try {
                   String sql = " delete from student where id =? ";
                   Connection conn = jdbcUtil.getConnection();
                   QueryRunner queryRunner= new QueryRunner();
                   queryRunner.update(conn,sql,7);
                   jdbcUtil.close(conn,null,null);
               }catch (Exception e){
                   throw new RuntimeException(e);
               }
   
           }
       }
   ```

2. 插入

   ```java
   import org.apache.commons.dbutils.QueryRunner;
   import utils.jdbcUtil;
   
   import java.sql.*;
   
   public class DbUtils {
       public static void main(String [] args){
               try {
                   String sql = "INSERT INTO student (NAME,PASSWORD) VALUE (?,?)";
                   Connection conn = jdbcUtil.getConnection();
                   QueryRunner queryRunner= new QueryRunner();
                   queryRunner.update(conn,sql,"丹丹",123456);
                   jdbcUtil.close(conn,null,null);
               }catch (Exception e){
                   throw new RuntimeException(e);
               }
           }
             public static void main2(String [] args){
               try {
                   String sql = "INSERT INTO student (NAME,PASSWORD) VALUE (?,?)";
                   Connection conn = jdbcUtil.getConnection();
                   QueryRunner queryRunner= new QueryRunner();
                   queryRunner.batch(conn,sql,new Object[][]{{"李彦斌",666666},{"胡歌",888888}});
                   jdbcUtil.close(conn,null,null);
               }catch (Exception e){
                   throw new RuntimeException(e);
               }
           }
       }
   ```

3. 更新

   与新建使用一样，只是SQL语句不一样

4. 查询

   对于结果集的处理有以下几个结果集处理器：

   1. BenaHandler //把单行结果集的数据封装成javaBean对象，返回值是ResultSetHandler

      ```java
        ResultSetHandler <javaBean类型> rsh = new BeanHandler<javaBean类型>(javaBean.class);
      ```

        本方法多用于在 处理把单行结果集封装成JavaBean对象。（对象时通过反射完成创建的）

   2. BeanListHandler

      ```java
        List<javaBean类型> list = <List<javaBean类型>> new BeanListHandler<javaBean类型>(javaBean.class);
      ```

        本方法多用于把多行结果集封装成对象，并且把对象添加到集合中，新版本中可能不需要进行类型的转换，
        的到集合可以通过foreach循环来进行遍历。

   3. MapHandler

      ```java
        Map <String,Object> map = new MapHandler();
      ```

        本方法是用来吧单行结果集封装到一个Map中其中map的键是表中的列名称，值对应表的列值。 

   4. MapListHandler

      ```java
        List<Map<String,Object>> listmap = new MapListHandler();
      ```

        本方法是用来多行结果集的处理，把每行的结果封装成一个map，最后把所有的，安排都装刀片一个集合中
        返回值是一个集合，但是集合中存放的是map，

   5. ColumnHandler

      ```java
      5. List<Object> nameList = new ColumnHandler();
      ```

        	本方法是用来出来单列，单行 或者多行的数据

   6. ScalarHandler
        本方法是用于处理单行单列的数据，多用于聚合函数的查询，但是以一个点需要注意，就是当聚合函数是涉及到数字类型的时候，一定要注意返回值类型的转换。有的人会选用Integer，long等类型，这些严格来说都是不合法的，例如，long类型最大只能容纳20的阶乘，21的阶乘就会包异常，所以我们要选用Number(这个是所有数据类型)的父类，并且对外提供的有Number.intValue(),和Number.LongValue(),等方法。

      ```java
      public void addProduct() throws SQLException{
              Connection conn=JDBCUtils.getConn();
              QueryRunner qr=new QueryRunner();
              String sql="insert into product(pid,pname) values(?,?)";
              Object[] obj={"1234567","iphoneXS"};
              qr.update(conn, sql,obj);
              DbUtils.closeQuietly(conn);
          }
          public void deleteProduct() throws SQLException{
              Connection conn=JDBCUtils.getConn();
              QueryRunner qr=new QueryRunner();
              String sql="delete from product where pname=? ";
              Object[] obj={"qwdqw"};
              qr.update(conn,sql, obj);
              DbUtils.closeQuietly(conn);
          }
          public void editProduct() throws SQLException{
              Connection conn=JDBCUtils.getConn();
              QueryRunner qr=new QueryRunner();
              String sql="update product set pname=? where pid=?";
              Object[] obj={"vivoX10","1234567"};
              qr.update(conn, sql,obj);
              DbUtils.closeQuietly(conn);
          }
          //ArrayHandler
          //将结果集中的第一行数据封装到Object[]中
          public void select1() throws SQLException{
              Connection conn=JDBCUtils.getConn();
              QueryRunner qr=new QueryRunner();
              String sql="select * from product";
              Object[] obj=qr.query(conn,sql, new ArrayHandler());
              for(Object o:obj){
                  System.out.println(o);
              }
          }
          //ArrayListHandler
          //将结果集中的每一行都封装到Object[]中，然后将每一个Object数组封装到一个List集合中
          public void select2() throws SQLException{
              Connection conn=JDBCUtils.getConn();
              QueryRunner qr=new QueryRunner();
              String sql="select * from product";
              List<Object[]> list=qr.query(conn,sql, new ArrayListHandler());
              for(Object[] obj:list){
                  for(Object o:obj){
                      System.out.println(o+"\t");
                  }
                  System.out.println();
              }
          }
          //BeanHandler
          //将结果集中的第一条记录封装到指定的JavaBean中
          public void select3() throws SQLException{
              QueryRunner qr=new QueryRunner();
              Connection conn=JDBCUtils.getConn();
              String sql="select * from product";
              Product product=qr.query(conn,sql, new BeanHandler<Product>(Product.class));
              System.out.println(product);
              DbUtils.closeQuietly(conn);
          }
          //BeanListHandler
              //将结果集中的每一条记录封装到指定的JavaBean中再将每一个JavaBean封装到List集合中
              public void select4() throws SQLException{
                  QueryRunner qr=new QueryRunner();
                  Connection conn=JDBCUtils.getConn();
                  String sql="select * from product";
                  List<Product> list=qr.query(conn,sql, new BeanListHandler<Product>(Product.class));
                  for(Product p:list){
                      System.out.println(p);
                  }
                  DbUtils.closeQuietly(conn);
              }
              //ColumnListHandler
              //将结果集中的指定列封装到List集合中
              public void select5() throws SQLException{
                  QueryRunner qr=new QueryRunner();
                  Connection conn=JDBCUtils.getConn();
                  String sql="select * from product";
                  List<String> list=qr.query(conn,sql, new ColumnListHandler<String>("pname"));
                  for(String s:list){
                      System.out.println(s);
                  }
                  DbUtils.closeQuietly(conn);
              }
              //ScalarHandler
              public void select6() throws SQLException{
                  QueryRunner qr=new QueryRunner();
                  Connection conn=JDBCUtils.getConn();
                  String sql="select count(*) from product";
                  Long count=qr.query(conn,sql, new ScalarHandler<Long>());
                  System.out.println(count);
                  DbUtils.closeQuietly(conn);
              }
              
              public void select7() throws SQLException{
                  QueryRunner qr=new QueryRunner();
                  Connection conn=JDBCUtils.getConn();
                  String sql="select * from product";
                  Map<String,Object> map=qr.query(conn, sql, new MapHandler());
                  Set<Map.Entry<String,Object>> set=map.entrySet();
                  for(Map.Entry<String,Object> entry:set){
                      System.out.println(entry.getKey()+"..."+entry.getValue());
                  }
                  DbUtils.closeQuietly(conn);
              }
              public void select8() throws SQLException{
                  QueryRunner qr=new QueryRunner(DBUtils.getDataSource());
                  //Connection conn=JDBCUtils.getConn();
                  String sql="select * from product";
                  List<Map<String,Object>> list=qr.query(sql, new MapListHandler());
                  for(Map<String,Object> map:list){
                      Set<Map.Entry<String,Object>> set=map.entrySet();
                      for(Map.Entry<String,Object> entry:set){
                          System.out.println(entry.getKey()+"..."+entry.getValue());
                      }
                  }
                  
                  //DbUtils.closeQuietly(conn);
              }
      }
      ```

更多参考：

[DbUtils学习—-DbUtils类](http://blog.csdn.net/x_iya/article/details/77244232)
[DBUtils学习—-QueryRunner类](http://blog.csdn.net/x_iya/article/details/77370161)
[DBUtils学习—-ResultSetHandler接口与实现](http://blog.csdn.net/x_iya/article/details/77185105)
[DBUtils学习—-RowProcessor接口与实现](http://blog.csdn.net/x_iya/article/details/77302878)
[DBUtils学习—-BeanProcessor类](http://blog.csdn.net/x_iya/article/details/77428087)
[DBUtils学习—-QueryLoader类](http://blog.csdn.net/x_iya/article/details/77407860)

### 面试

1. PreparedStatement的缺点是什么，怎么解决这个问题？

   PreparedStatement的一个缺点是，**我们不能直接用它来执行in条件语句**；需要执行IN条件语句的话，下面有一些解决方案：

   - 分别进行单条查询——这样做性能很差，不推荐。
   - 使用存储过程——这取决于数据库的实现，不是所有数据库都支持。
   - 动态生成PreparedStatement——这是个好办法，但是不能享受PreparedStatement的缓存带来的好处了。
   - 在PreparedStatement查询中使用NULL值——如果你知道输入变量的最大个数的话，这是个不错的办法，扩展一下还可以支持无限参数。

2. JDBC的DriverManager是用来做什么的？

   - JDBC的DriverManager是一个**工厂类**，我们**通过它来创建数据库连接。** 
   - 当JDBC的Driver类被加载进来时，它会自己注册到DriverManager类里面
   - 然后我们会把数据库配置信息传成DriverManager.getConnection()方法**，DriverManager会使用注册到它里面的驱动来获取数据库连接，并返回给调用的程序**。

3. 有哪些不同的ResultSet？

   根据创建Statement时输入参数的不同，会对应不同类型的ResultSet。如果你看下Connection的方法，你会发现createStatement和prepareStatement方法重载了，以支持不同的ResultSet和并发类型。

   一共有三种ResultSet对象。

   - ResultSet.TYPE_FORWARD_ONLY：这是默认的类型，它的游标只能往下移。
   - ResultSet.TYPE_SCROLL_INSENSITIVE：游标可以上下移动，一旦它创建后，数据库里的数据再发生修改，对它来说是透明的。
   - ResultSet.TYPE_SCROLL_SENSITIVE：游标可以上下移动，如果生成后数据库还发生了修改操作，它是能够感知到的。

   ResultSet有两种并发类型。

   - ResultSet.CONCUR_READ_ONLY:ResultSet是只读的，这是默认类型。
   - ResultSet.CONCUR_UPDATABLE:我们可以使用ResultSet的更新方法来更新里面的数据。 

4. JDBC的DataSource是什么，有什么好处

   DataSource即数据源，它是定义在javax.sql中的一个接口，跟DriverManager相比，它的功能要更强大**。我们可以用它来创建数据库连接，当然驱动的实现类会实际去完成这个工作。除了能创建连接外，它还提供了如下的特性：**

   - **缓存PreparedStatement以便更快的执行**
   - **可以设置连接超时时间**
   - **提供日志记录的功能**
   - **ResultSet大小的最大阈值设置**
   - **通过JNDI的支持，可以为servlet容器提供连接池的功能**

5. 如何通过JDBC的DataSource和Apache Tomcat的JNDI来创建连接池？

   Tomcat服务器也给我们提供了连接池，内部其实就是DBCP

   步骤：

   1. 在META-INF目录下配置context.xml文件【文件内容可以在tomcat默认页面的 JNDI Resources下Configure Tomcat's Resource Factory找到】
   2. 导入Mysql或oracle开发包到tomcat的lib目录下
   3. 初始化JNDI->获取JNDI容器->检索以XXX为名字在JNDI容器存放的连接池

   context.xml文件的配置：

   ```xml
   <Context>
   
     <Resource name="jdbc/EmployeeDB"
               auth="Container"
               type="javax.sql.DataSource"
               
               username="root"
               password="root"
               driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://localhost:3306/zhongfucheng"
               maxActive="8"
               maxIdle="4"/>
   </Context>
   ```

   java

   ```java
           try {
   
               //初始化JNDI容器
               Context initCtx = new InitialContext();
   
               //获取到JNDI容器
               Context envCtx = (Context) initCtx.lookup("java:comp/env");
   
               //扫描以jdbc/EmployeeDB名字绑定在JNDI容器下的连接池
               DataSource ds = (DataSource)
                       envCtx.lookup("jdbc/EmployeeDB");
   
               Connection conn = ds.getConnection();
               System.out.println(conn);
   
           } 
   ```

6. 常见的JDBC异常有哪些？

   有以下这些：

   - java.sql.SQLException——这是JDBC异常的基类。
   - java.sql.BatchUpdateException——当批处理操作执行失败的时候可能会抛出这个异常。这取决于具体的JDBC驱动的实现，它也可能直接抛出基类异常java.sql.SQLException。
   - java.sql.SQLWarning——SQL操作出现的警告信息。
   - java.sql.DataTruncation——字段值由于某些非正常原因被截断了（不是因为超过对应字段类型的长度限制）。

7. JDBC中存在哪些不同类型的锁？

   从广义上讲，有两种锁机制来防止多个用户同时操作引起的数据损坏。

   -  **乐观锁——只有当更新数据的时候才会锁定记录**。
   - **悲观锁——从查询到更新和提交整个过程都会对数据记录进行加锁。**

8. java.util.Date和java.sql.Date有什么区别？

   java.util.Date包含日期和时间，而java.sql.Date只包含日期信息，而没有具体的时间信息。**如果你想把时间信息存储在数据库里，可以考虑使用Timestamp或者DateTime字段**

9. SQLWarning是什么，在程序中如何获取SQLWarning？

   SQLWarning是SQLException的子类，**通过Connection, Statement, Result的getWarnings方法都可以获取到它**。 SQLWarning不会中断查询语句的执行，只是用来提示用户存在相关的警告信息。

10. 如果java.sql.SQLException: No suitable driver found该怎么办？

    如果你的SQL URL串格式不正确的话，就会抛出这样的异常。不管是使用DriverManager还是JNDI数据源来创建连接都有可能抛出这种异常。

11. JDBC的RowSet是什么，有哪些不同的RowSet？

    RowSet用于存储查询的数据结果，和ResultSet相比，**它更具灵活性。RowSet继承自ResultSet，因此ResultSet能干的，它们也能，而ResultSet做不到的，它们还是可以**。RowSet接口定义在javax.sql包里。

    RowSet提供的额外的特性有：

    - 提供了Java Bean的功能，可以通过settter和getter方法来设置和获取属性。RowSet使用了JavaBean的事件驱动模型，它可以给注册的组件发送事件通知，比如游标的移动，行的增删改，以及RowSet内容的修改等。
    - RowSet对象默认是**可滚动，可更新的，因此如果数据库系统不支持ResultSet实现类似的功能，可以使用RowSet来实现**。

    RowSet分为两大类：

    - A. **连接型RowSet——这类对象与数据库进行连接**，和ResultSet很类似。JDBC接口只提供了一种连接型RowSet，javax.sql.rowset.JdbcRowSet，它的标准实现是com.sun.rowset.JdbcRowSetImpl。
    - B. **离线型RowSet——这类对象不需要和数据库进行连接，因此它们更轻量级，更容易序列化。它们适用于在网络间传递数据**。
      - 有四种不同的离线型RowSet的实现。
        - CachedRowSet——可以通过他们获取连接，执行查询并读取ResultSet的数据到RowSet里。我们可以在离线时对数据进行维护和更新，然后重新连接到数据库里，并回写改动的数据。
        - WebRowSet继承自CachedRowSet——他可以读写XML文档。
        - JoinRowSet继承自WebRowSet——它不用连接数据库就可以执行SQL的join操作。
        - FilteredRowSet继承自WebRowSet——我们可以用它来设置过滤规则，这样只有选中的数据才可见。

12. 什么是JDBC的最佳实践？

    - 数据库资源是非常昂贵的，**用完了应该尽快关闭它**。Connection, Statement, ResultSet等JDBC对象都有close方法，调用它就好了。
    -  **养成在代码中显式关闭掉ResultSet，Statement，Connection的习惯**，如果你用的是连接池的话，连接用完后会放回池里，但是没有关闭的ResultSet和Statement就会造成资源泄漏了。
    - 在finally块中关闭资源，保证即便出了异常也能正常关闭。
    -  **大量类似的查询应当使用批处理完成**。
    - **尽量使用PreparedStatement而不是Statement，以避免SQL注入，同时还能通过预编译和缓存机制提升执行的效率。**
    - 如果你要将大量数据读入到ResultSet中，**应该合理的设置fetchSize以便提升性能**。
    - 你用的数据库可能没有支持所有的隔离级别，用之前先仔细确认下。
    - 数据库隔离级别越高性能越差，确保你的数据库连接设置的隔离级别是最优的。
    - 如果在WEB程序中创建数据库连接，最好通过JNDI使用JDBC的数据源，这样可以对连接进行重用。
    - **如果你需要长时间对ResultSet进行操作的话，尽量使用离线的RowSet。**

### 参考

1. [JDBC常见面试题](https://segmentfault.com/a/1190000013312766)