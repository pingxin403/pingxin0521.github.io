---
title: MySQL 补充知识
date: 2019-06-18 13:08:59
tags:
 - 数据库
 - MySQL
categories:
 - 数据库
 - MySQL
---

#### CURRENT_TIMESTAMP

timestamp有两个属性，分别是CURRENT_TIMESTAMP 和ON UPDATE CURRENT_TIMESTAMP两种，使用情况分别如下：

<!--more-->

1. CURRENT_TIMESTAMP

当要向数据库执行insert操作时，如果有个timestamp字段属性设为CURRENT_TIMESTAMP，则无论这个字段有没有set值都插入当前系统时间  

2. ON UPDATE CURRENT_TIMESTAMP

当执行update操作是，并且字段有ON UPDATE CURRENT_TIMESTAMP属性。则字段无论值有没有变化，它的值也会跟着更新为当前UPDATE操作时的时间。

```
`gmt_create` datetime NULL default CURRENT_TIMESTAMP comment '创建时间',
`gmt_modified` datetime NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP comment '修改时间',
```
当执行update操作是，并且字段有ON UPDATE CURRENT_TIMESTAMP属性。则字段无论值有没有变化，它的值也会跟着更新为当前UPDATE操作时的时间。

如果设置时间的长度，后面的CURRENT_TIMESTAMP也需要指定长度：
```
`gmt_create` datetime(3) NULL default CURRENT_TIMESTAMP(3) comment '创建时间',
`gmt_modified` datetime(3) NULL default CURRENT_TIMESTAMP(3) on update CURRENT_TIMESTAMP(3) comment '修改时间',
```

1. datetime(3)类型的默认值为CURRENT_TIMESTAMP(3)而不是CURRENT_TIMESTAMP().切记!!!
2. mysql5.5及之前版本只支持timestamp类型设置默认值为CURRENT_TIMESTAMP,不支持datetime类型默认值设置为CURRENT_TIMESTAMP

#### TINYINT

unsigned   既为非负数，用此类型可以增加数据长度!

例如如果    tinyint最大是127，那    tinyint    unsigned    最大   就可以到    127 * 2

unsigned 属性只针对整型，而binary属性只用于char 和varchar。

![](https://i.loli.net/2019/04/30/5cc7aa7acf89a.png)

每种数值类型的名称和取值范围

![](https://i.loli.net/2019/04/30/5cc7aa7abf567.png)

各种类型值所需的存储量

![](https://i.loli.net/2019/04/30/5cc7aa7a8db88.png)

mysql提供了五种整型： tinyint、smallint、mediumint、int和bigint。int为integer的缩写。这些类型在可表示的取值范围上是不同的。 整数列可定义为unsigned从而禁用负值；这使列的取值范围为0以上。各种类型的存储量需求也是不同的。取值范围较大的类型所需的存储量较大。

mysql 提供三种浮点类型： float、double和decimal。与整型不同，浮点类型不能是unsigned的，其取值范围也与整型不同，这种不同不仅在于这些类型有最大 值，而且还有最小非零值。最小值提供了相应类型精度的一种度量，这对于记录科学数据来说是非常重要的（当然，也有负的最大和最小值）。

**比较**

tinyint 型的字段如果设置为UNSIGNED类型，只能存储从0到255的整数,不能用来储存负数。
tinyint 型的字段如果不设置UNSIGNED类型,存储-128到127的整数。
1个tinyint型数据只占用一个字节;一个INT型数据占用四个字节。

这看起来似乎差别不大，但是在比较大的表中，字节数的增长是很快的。

tinyint(1)与tinyint(2)的区别可以从下面看出来
```
CREATE TABLE `test` (                                  
          `id` int(11) NOT NULL AUTO_INCREMENT,                
          `str` varchar(255) NOT NULL,                                     
          `state` tinyint(1) unsigned zerofill DEFAULT NULL,   
          `state2` tinyint(2) unsigned zerofill DEFAULT NULL,  
          `state3` tinyint(3) unsigned zerofill DEFAULT NULL,  
          `state4` tinyint(4) unsigned zerofill DEFAULT NULL,  
          PRIMARY KEY (`id`)                                   
        ) ENGINE=MyISAM AUTO_INCREMENT=6 DEFAULT CHARSET=utf8  

insert into test (str,state,state2,state3,state4) values('csdn',4,4,4,4);
select * from test;

```
结果：
```
id   str      state   state2   state3   state4
1    csdn  4         04         004        0004
```


mysql 中int(1)和tinyint(1)中的1只是指定显示长度，并不表示存储长度.
tinyint可以存储1字节, 即unsigned 0~255(signed -127~127)。显示大小不受此限制 (所有整数类型相同),即使设为1,也可以存入和取出大于10的数。括号里的数字,即显示大小对整数来说主要有两个目的, 一是做为编码文档;将 tinyint (1) 放到表定义中会告诉人们只有数字 0 - 9 应该输入, 表没有被设计具有其他值。

另一个目的是, 你可以与属性ZEROFILL联合使用。ZEROFILL将填充小于显示大小的数字,在他们面前补上零。例如 TINYINT (3) 与 ZEROFILL, 插入值4, 你会得到 004。

#### mysql数据类型之用 TINYINT(1) 还是 ENUM( 'true' , 'false')
在以往的经验中，如果遇到需要抉择是否用mysql的enum数据类型时，我基本不用思考的就会放弃 ENUM()并用tinyint取而代之，原因就是我以前接触的哪些场景，均适合用tinyint，也即在第一次选择了tinyint后就再也没认真研究关注过这两个字段类型了，而今天在开发 超凡商标管家 的途中遇到一个商标状态扩展的需求，需要建立一张大量存储0和1的字段的表，因为是大量0和1，所以我犹豫了下，顺便再次查阅下官方文档，其建议如下:

```
TINYINT（1）或ENUM（'真'，'假'）？

     用ENUM存储枚举当存储只有2个值时只占用一个位的宽度，0或1，但会花更多的时间去寻找了枚举查询的开始。

     用TINYINT（1）默认就会占用4个位的宽度（0000）

     因此得出结论：

     比如要存储一个介于0-9之间的值，为了查询获取这个值，建议用TINYINT（1）会更快，

     但如果你是为了大量记录枚举（“真”，“假”），那么用ENUM( 'true' , 'false') 搜索会更快。
```
说起这个ENUM， 经查阅各大技术社区的网络文摘，ENUM确实是mysql里的一个特色字段，印象里模糊记得在以前看到一些比较知名的商城系统如shopnc里面在用它，但也没细究，可能是因为他可以设置字段的区间范围，会让值可以被数据库所控制，有枚举约束的功能（比如，字段只想有0和1，如果用TINYINT（1），结果就可能出现2，那2就是赃数据了）

但ENUM也有一些比较棘手的问题，比如数据迁移的时候，他几乎不可能被其他数据库所支持，如果enum里面是字符串，对于其他数据库来说就更郁闷了，还不能设为tinyint等类型的字段（enum虽然可以存储字符串，但对于内部来说，还是以顺序进行索引，比如'a','b','c'，我们也可以用索引值来获取值select * from tbl_name whre enum = 2，这与select * from tbl_name where enum = 'b'等义）如果你看明白了这两句SQL为什么等义，那么你也就可以了解为什么不主张用enum字段了。

也就是说，假如一个设计不合理的ENUM字段，给程序员带来的就完全是梦魇了，比如一个enum字段的范围是('0','1','2','3','4','5')，而enum的枚举值对应的索引是从1开始的，因此，insert into table (enum)values(1)，插入的并不是1，而是0。

另外假如你在设计好enum的枚举字段范围并使用了一段时间后，再到字段范围中加一个枚举值，并且不是加在最后，那么也就相当于把原来的范围都改变了索引值，也就是当你在查询的时候直接查询值（并加上单引号），将不会使用enum自身隐藏的索引值来获取结果了。

如果是纯数值型，还是建议采用tinyint字段吧，毕竟它也只占一个字节，即使出现赃数据，也可以被接受，不象enum，如果纯数字型范围，更改了索引，你就不知道你查询的值是否正确了）

故建议，如果字段是字符串，并且长度固定，可以尝试用char，如果是数值型，还是用tinyint吧，比较安全稳定，而且即使迁移，也不会出现太多问题。

所以为了最终方便，避免出现一些不必要的问题，我在本次商标状态扩张表中仍然按以往习惯使用tinyint哈。


#### This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its de 错误解决办法

这是我们开启了bin-log, 我们就必须指定我们的函数是否是
1. DETERMINISTIC 不确定的
2. NO SQL 没有SQl语句，当然也不会修改数据
3. READS SQL DATA 只是读取数据，当然也不会修改数据
4. MODIFIES SQL DATA 要修改数据
5. CONTAINS SQL 包含了SQL语句

其中在function里面，只有 DETERMINISTIC, NO SQL 和 READS SQL DATA 被支持。如果我们开启了 bin-log, 我们就必须为我们的function指定一个参数。

在MySQL中创建函数时出现这种错误的解决方法：
```
set global log_bin_trust_function_creators=TRUE;
```

#### Mysql 中获取刚插入的自增长id的值
```
insert into user (username,password) VALUES ('zyl','123');   
//获取刚插入的自增长id的值
select last_insert_id();   
```
在MySQL中，使用auto_increment类型的id字段作为表的主键，并用它作为其他表的外键，形成“主从表结构”，这是数据库设计中常见的用法。但是在具体生成id的时候，我们的操作顺序一般是：先在主表中插入记录，然后获得自动生成的id，以它为基础插入从表的记录。这里面有个困难，就是插入主表记录后，如何获得它对应的id。通常的做法，是通过“select max(id) from tablename”的做法，但是显然这种做法需要考虑并发的情况，需要在事务中对主表加以“X锁“，待获得max(id)的值以后，再解锁。这种做法需要的步骤比较多，有些麻烦，而且并发性也不好。有没有更简单的做法呢?答案之一是通过select LAST_INSERT_ID()这个操作。乍一看，它和select max(id)很象，但实际上它是线程安全的。也就是说它是具体于数据库连接的。下面通过实验说明：
1. 在连接1中向A表插入一条记录，A表包含一个auto_increment类型的字段。
2. 在连接2中向A表再插入一条记录。
3. 结果：在连接1中执行select LAST_INSERT_ID()得到的结果和连接2中执行select LAST_INSERT_ID()的结果是不同的;而在两个连接中执行select max(id)的结果是相同的。

其实在MSSQL中SCOPE_IDENTITY()和IDENT_CURRENT()的区别和这里是类似的。使用SCOPE_IDENTITY()可以获得插入某个IDENTITY字段的当前会话的值，而使用IDENT_CURRENT()会获得在某个IDENTITY字段上插入的最大值，而不区分不同的会话。

注：使用select last_insert_id()时要注意，当一次插入多条记录时，只是获得第一次插入的id值，务必注意!可以试试
```
insert into tb(c1,c2) values (c1value,c2value),(c1value1,c2value2)..。
```
最后有个问题一直没解决：

**使用select last_insert_id()时要注意，当一次插入多条记录时，只是获得第一次插入的id值，务必注意。**

查询自增id的下一个值

```
-- 查询自增id的下一个值
SELECT
	AUTO_INCREMENT
FROM
	INFORMATION_SCHEMA. TABLES
WHERE
	TABLE_NAME = '{table name}'
```


#### 获取某个表中的最大序号数

我们在写数据库程序的时候,经常会需要获取某个表中的最大序号数,
一般情况下获取刚插入的数据的id，使用select max(id) from table 是可以的。
但在多线程情况下，就不行了。

下面介绍三种方法

(1) getGeneratedKeys()方法:
```
Connection conn = ;   
    Serializable ret = null;   
    PreparedStatement state = .;   
    ResultSet rs=null;   
    try {   
        state.executeUpdate();   
        rs = state.getGeneratedKeys();   
        if (rs.next()) {   
            ret = (Serializable) rs.getObject(1);   
        }         
    } catch (SQLException e) {   
    }   
    return ret;
```
2)LAST_INSERT_ID:   

LAST_INSERT_ID 是与table无关的，如果向表a插入数据后，再向表b插入数据，LAST_INSERT_ID会改变。

在多用户交替插入数据的情况下max(id)显然不能用。

这就该使用LAST_INSERT_ID了，因为LAST_INSERT_ID是基于Connection的，只要每个线程都使用独立的Connection对象，LAST_INSERT_ID函数将返回该Connection对AUTO_INCREMENT列最新的insert or update*作生成的第一个record的ID。这个值不能被其它客户端（Connection）影响，保证了你能够找回自己的 ID 而不用担心其它客户端的活动，而且不需要加锁。使用单INSERT语句插入多条记录, LAST_INSERT_ID返回一个列表。

(3)select @@IDENTITY:  

```
String sql="select @@IDENTITY";   
```
@@identity是表示的是最近一次向具有identity属性(即自增列)的表插入数据时对应的自增列的值，是系统定义的全局变量。一般系统定义的全局变量都是以@@开头，用户自定义变量以@开头。比如有个表A，它的自增列是id，当向A表插入一行数据后，如果插入数据后自增列的值自动增加至101，则通过select @@identity得到的值就是101。使用@@identity的前提是在进行insert操作后，执行select @@identity的时候连接没有关闭，否则得到的将是NULL值。  

#### mysql使用ROW_COUNT()返回插入、更新、删除操作影响行数

如果你要测试 FOUND_ROWS() 和 ROW_COUNT() 这两个函数，最好就不要用那些MySQL的图形化管理工具软件了（例如，SQLYog）。因为当你使用些工具软件执行某条SQL语句时，可能实际上并不仅仅是执行了这条SQL，这些软件同时会在后台自己执行一些其他SQL语句。所以有时你可能会发现这两个函数返回的结果和你预期的并不一样。所以呢，最好还是用 cmd 窗口来执行SQL进行测试。

**FOUND_ROWS() 函数**

（1） FOUND_ROWS()函数返回的是上一条 SELECT 语句（或 SHOW语句等）查询结果集的记录数。

注意，是上一条 SELECT 语句（即执行该函数前的最近一条SELECT语句），而不是上一条 SQL 语句；因为上一条SQL语句不一定是 SELECT 语句。

且，像 SELECT ROW_COUNT() 这种语句也是 SELECT 语句，它们的结果集也会被 FOUND_ROWS() 函数查出来。

（2）如果上一条 SELECT 语句查询结果为空，则返回 0。

（3）SHOW XXX（例如，show tables、show databases、show status）语句也会被 FOUND_ROWS() 函数查出来。


**ROW_COUNT() 函数**

（1）FOUND_ROWS()函数返回的是上一条SQL语句，对表数据进行修改操作后影响的记录数。

如果上一条SQL语句不是修改操作语句（INSERT/UPDATE/DELETE 等），而是查询语句（SELECT/SHOW 等）则返回-1。如果是修改操作语句，则返回修改（增/删/该）影响的记录数。

注意，这里是上一条SQL语句（即执行该函数前的上一条SQL语句），和上面有所区别。

（2）如果上一条SQL语句是UPDATE语句，但是UPDATE后所有数据的值并没有改变，则返回 0。

（3）如果上一条SQL语句是建表语句（创建表或临时表），但创建的是空表，则返回 0。

如果是删除表（DROP语句），则返回的还是 0。

（4）如果是创建临时表，但使用的是 AS 关键字直接将查询出来的值赋值给新建的临时表的话（其实就相当于新建了一个空表，紧接着使用了一条INSERT语句而已），则返回插入的记录数。

在Mysql中ROW_COUNT()返回前一个SQL进行UPDATE，DELETE，INSERT操作所影响的行数。
注意在UPDATE中，如果替换前、后值是一样的，ROW_COUNT也会返回0。

存储过程示例示例：

```
begin
insert into test values('','第一条');
if ROW_COUNT()>0 then
insert into test values('','第二条');
end if;
end;
```

如果第一条数据插入成功，再插入第二条数据。

ROW_COUNT()用于判断刚刚的操作是否成功，如果成功则返回>0;否则返回0；

#### Mysql判断记录是否存在

判断记录是否存在的sql，不同的写法，也会有不同的性能。

```
select count(*) from tablename where col = 'col';
```
这种方法性能上有些浪费，没必要把全部记录查出来。

```
select 1 from tablename where col = 'col' limit 1;
```
执行这条sql语句，所影响的行数不是0就是1。

特别解释下limit 1，mysql在找到一条记录后就不会往下继续找了。性能提升很多。

结论：推荐第二种方式。



#### MySQL分页

首先获取分页数据集的数量，如orders_history订单历史表：

```
select count(*) from orders_history;
```

一般的分页查询使用简单的 limit 子句就可以实现。limit 子句声明如下：

```
SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset
```

LIMIT 子句可以被用于指定 SELECT 语句返回的记录数。需注意以下几点：

- 第一个参数指定第一个返回记录行的偏移量
- 第二个参数指定返回记录行的最大数目
- 如果只给定一个参数：它表示返回最大的记录行数目
- 第二个参数为 -1 表示检索从某一个偏移量到记录集的结束所有的记录行
- 初始记录行的偏移量是 0(而不是 1)

下面是一个应用实例：

```
select * from orders_history where type=8 limit 1000,10;
```

该条语句将会从表 orders_history 中查询第1000条数据之后的10条数据，也就是第1001条到第1010条数据。

从查询时间来看，基本可以确定，在查询记录量低于100时，查询时间基本没有差距，随着查询记录量越来越大，所花费的时间也会越来越多。

随着查询偏移的增大，尤其查询偏移大于10万以后，查询时间急剧增加。

**这种分页查询方式会从数据库第一条记录开始扫描，所以越往后，查询速度越慢，而且查询的数据越多，也会拖慢总查询速度。**

##### 使用子查询优化

这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

```
select * from orders_history where type=8 limit 100000,1;

select id from orders_history where type=8 limit 100000,1;

select * from orders_history where type=8 and 
id>=(select id from orders_history where type=8 limit 100000,1) 
limit 100;

select * from orders_history where type=8 limit 100000,100;
```



4条语句的查询时间如下：

- 第1条语句：3674ms
- 第2条语句：1315ms
- 第3条语句：1327ms
- 第4条语句：3710ms

针对上面的查询需要注意：

- 比较第1条语句和第2条语句：使用 select id 代替 select * 速度增加了3倍
- 比较第2条语句和第3条语句：速度相差几十毫秒
- 比较第3条语句和第4条语句：得益于 select id 速度增加，第3条语句查询速度增加了3倍

这种方式相较于原始一般的查询方法，将会增快数倍。

##### 使用 id 限定优化

这种方式假设数据表的id是**连续递增**的，则我们根据查询的页数和查询的记录数可以算出查询的id的范围，可以使用 id between and 来查询：

```
select * from orders_history where type=2 
and id between 1000000 and 1000100 limit 100;
```

查询时间：15ms 12ms 9ms

这种查询方式能够极大地优化查询速度，基本能够在几十毫秒之内完成。限制是只能使用于明确知道id的情况，不过一般建立表的时候，都会添加基本的id字段，这为分页查询带来很多便利。

还可以有另外一种写法：

```
select * from orders_history where id >= 1000001 limit 100;
```

当然还可以使用 in 的方式来进行查询，这种方式经常用在多表关联的时候进行查询，使用其他表查询的id集合，来进行查询：

```
select * from orders_history where id in
(select order_id from trade_2 where goods = 'pen')
limit 100;
```

这种 in 查询的方式要注意：某些 mysql 版本不支持在 in 子句中使用 limit。

##### 使用临时表优化

这种方式已经不属于查询优化，这儿附带提一下。

对于使用 id 限定优化中的问题，需要 id  是连续递增的，但是在一些场景下，比如使用历史表的时候，或者出现过数据缺失问题时，可以考虑使用临时存储的表来记录分页的id，使用分页的id来进行  in 查询。这样能够极大的提高传统的分页查询速度，尤其是数据量上千万的时候。

##### 关于数据表的id说明

一般情况下，在数据库中建立表的时候，强制为每一张表添加 id 递增字段，这样方便查询。

如果像是订单库等数据量非常庞大，一般会进行分库分表。这个时候不建议使用数据库的 id 作为唯一标识，而应该使用分布式的高并发唯一 id 生成器来生成，并在数据表中使用另外的字段来存储这个唯一标识。

使用先使用范围查询定位 id （或者索引），然后再使用索引进行定位数据，能够提高好几倍查询速度。即先 select id，然后再 select *；

### 修改时区

以下记录修改mysql时区的几种方法。

具体：
方法一：通过mysql命令行模式下动态修改

1. 查看mysql当前时间，当前时区

```
> select curtime();   #或select now()也可以
+-----------+
| curtime() |
+-----------+
| 15:18:10  |
+-----------+

> show variables like "%time_zone%";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)
#time_zone说明mysql使用system的时区，system_time_zone说明system使用CST时区
```

2. 修改时区

```
> set global time_zone = '+8:00';  ##修改mysql全局时区为北京时间，即我们所在的东8区
> set time_zone = '+8:00';  ##修改当前会话时区
> flush privileges;  #立即生效
```


方法二：通过修改my.cnf配置文件来修改时区

```
# vim /etc/my.cnf  ##在[mysqld]区域中加上
default-time_zone = '+8:00'

# /etc/init.d/mysqld restart  ##重启mysql使新时区生效
```

方法三：如果不方便重启mysql，又想临时解决时区问题，可以通过php或其他语言在初始化mysql时初始化mysql时区
这里，以php为例，在mysql_connect()下使用mysql_query(“SET time_zone = ‘+8:00′”)。
这样可以在保证你不重启的情况下改变时区。但是mysql的某些系统函数还是不能用如：now()。这句，还是不能理解。

### 数据类型转换

Mysql中提供了两个内置函数提供我们使用分别为:CAST和CONVERT,Mysql 的CAST()和CONVERT() 函数可用来转换或者获取一个我们需要的类型。两者具体的语法如下：

(1)  cast(value as type);

(2) convert(value,type); 

**语法**

说明 value 为要转换原数据类型的值,type目的类型.但可以转换的类型是有限制的。类型可以是以下任何一个：

- 二进制，同带binary前缀的效果 : BINARY    
- 字符型，可带参数 : CHAR()    
- 日期 : DATE     
- 时间: TIME     
- 日期时间型 : DATETIME     
- 浮点数 : DECIMAL      
- 整数 : SIGNED     
- 无符号整数 : UNSIGNED

#### mysql将字符串字段转为数字排序或比大小

mysql里面有个坑就是，有时按照某个字段的大小排序（或是比大小）发现排序有点错乱。后来才发现，是我们想当然地把对字符串字段当成数字并按照其大小排序（或是比大小），结果肯定不会是你想要的结果。

这时候需要把字符串转成数字再排序。

最简单的办法就是在字段后面加上+0,如把'123'转成数字123（以下例子全为亲测）：

**排序**

- 方法一：`ORDER BY '123'+0;`（首推）

- 方法二：`ORDER BY CAST('123' AS SIGNED);`
- 方法三：`ORDER BY CONVERT('123',SIGNED);`

**比较**

```mysql
SELECT '123'+0;  --   结果为123
SELECT '123'+0>127;  --   结果为0
SELECT '123'+0>12;  --   结果为1

SELECT CAST('123' AS SIGNED);    --  结果为123
SELECT CONVERT('123',SIGNED)>127;   --  结果为0
SELECT CONVERT('123',SIGNED)>12;   --  结果为1

SELECT CAST('123' AS SIGNED);  -- 结果为123
SELECT CAST('123' AS SIGNED)>127;  -- 结果为0
SELECT CAST('123' AS SIGNED)>12;   -- 结果为1
```

综合例子：

```mysql
SELECT '123'+0>12 ORDER BY CONVERT('123',SIGNED);  --  结果为1
```

#### SQL_MODE

##### SQL_MODE类型

mysql中的sql_mode就是控制mysql行为模式的一些配置。查看mysql的sql_mode可通过以下语句：

```mysql
select @@sql_mode;

show variables like "sql_mode";

***************************[ 1. row ]***************************
@@sql_mode | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

1. NO_ENGIN_SUBSTITUTION

   其中，NO_ENGIN_SUBSTITUTION表示当创建表时，指定一个不存在的存储引擎时，mysql是报错还是设置成默认存储引擎innodb；经过实验发现：当sql_mode中含有NO_ENGIN_SUBSITUTION这一项时，在创建表指定一个不存在的存储引擎，mysql会报错(substitution意为替换)。反之，则会设置成默认的innodb。

   修改sql_mode，如果想去除sql_mode中某个选项，可用如下语句：

   ```
   set sql_mode = (select replace(@@sql_mode, 'NO_ENGINE_SUBSTITUTION', ''));
   ```

   如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常。

2. NO_AUTO_CREATE_USER

   禁止mysql创建密码为空的用户，如果sql_mode中含有该选项，那么创建mysql用户 时如果不设置密码则会报错。

3. NO_ZERO_IN_DATE

   在严格模式，不接受月或日部分为0的日期。如果使用IGNORE选项，我们为类似的日期插入'0000-00-00'。在非严格模式，可以接受该日期，但会生成警告。

4. NO_ZERO_DATE

   在严格模式，不要将 '0000-00-00'做为合法日期。你仍然可以用IGNORE选项插入零日期。在非严格模式，可以接受该日期，但会生成警告。

5. STRICT_TRANS_TABLES

   在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做任何限制。对于InnoDB表，sql插入执行失败，会报错，全部回滚。

6. STRICT_ALL_TABLES

   对于InnoDB表，和STRICT_TRANS_TABLES模式相同;对于MyISAM表，sql插入执行失败会保存错误发生之前修改的行，忽略剩下的行，并报错。

7. ERROR_FOR_DIVISION_BY_ZERO

   在严格模式，在INSERT或UPDATE过程中，如果被零除(或MOD(X，0))，则产生错误(否则为警告)。如果未给出该模式，被零除时MySQL返回NULL。如果用到INSERT IGNORE或UPDATE IGNORE中，MySQL生成被零除警告，但操作结果为NULL。

8. ONLY_FULL_GROUP_BY

   对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么将认为这个SQL是不合法的，因为列不在GROUP BY语句中。

   因为有only_full_group_by，所以我们要在MySQL中正确的使用group by语句的话，只能是select c1 from t group by c1 (**即只能展示group by的字段，其他均都要报1055的错**)

##### sql_mode模式

首先创建一个表：`create table testmode(name varchar(2),test varchar(2));`

1. ANSI模式：宽松模式，对插入数据进行校验，如果不符合定义类型或长度，对数据类型调整或截断保存，报warning警告。

   先设置模式ANSI：`set @@sql_mode=ANSI;`

   ```mysql
   > select @@sql_mode \G
   ***************************[ 1. row ]***************************
   @@sql_mode | REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ONLY_FULL_GROUP_BY,ANSI
   ```

   插入数据：`insert into testmode values('11111111','2342342424');`

   插入成功，报出2条警告。而且只截取了前面2个字符数据。

2. TRADITIONAL模式：严格模式，当向mysql数据库插入数据时，进行数据的严格校验，保证错误数据不能插入，报error错误。用于事物时，会进行事物的回滚。

   先设置模式TRADITIONAL：`set @@sql_mode=TRADITIONAL;`

   ```mysql
   > select @@sql_mode \G
   ***************************[ 1. row ]***************************
   @@sql_mode | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION
   ```

### 参考

1. [MySQL的sql_mode解析与设置](https://baijiahao.baidu.com/s?id=1637179252298020747&wfr=spider&for=pc)