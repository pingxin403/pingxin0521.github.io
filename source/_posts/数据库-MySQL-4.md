---
title: MySQL SQL 函数（五）
date: 2019-05-12 14:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---
### 常用函数

MySQL中常用函数很多，其实不要死记硬背，记住常用的几个，其他查看即可。

<!--more-->

#### 数学函数

使用格式：一般用于插入、修改语句中，直接 函数（参数） 即可，把返回结果用于插入、修改。

- RAND（）：随机数生成。区别在于，RAND()返回的数是完全随机的，而RAND(x)在x相同时返回的值相同
- ROUND(X,Y):得到X的Y位四舍五入小数。
- LOG（x，y）:得到以x为底，y的对数。
- SQRT（x）：得到x的平方根。
- MOD(x,y)：x对y求余。
- CEIL(x)、CEILING(x)：向上取整。
- FLOOR(x)：向下取整。
- ROUND(x)：返回离x最近的整数，也就是对x进行四舍五入处理
- SIGN(x)返回x的符号，-1为负数，0不变，1为正数
- POW(x,y)、POWER(x,y)：幂运算，求x的y次方幂。

应用实例：
```
insert into test values (RAND(0),ROUND(4.5462,3),LOG(2,8),SQRT(9));
select * from test;
```
结果：
```
0,4.54600000000000000000000000000,3.0000000000000000000000000000000,3.0000000000000000000000000000000
```
#### 聚集函数

使用格式：聚集函数一般是配合GROUP BY语句使用的，也可以用于统计整表、整列。

- AVG() ：	返回某列的平均值
- COUNT()：	返回某列/某组/整表的行数（即记录数）
- MAX() 	：返回某列的最大值
- MIN() 	：返回某列的最小值
- SUM() 	：返回某个列之和

样例：
```
select COUNT（*）  from test GROUP BY id;
select AVG(grade) from student;
select MAX(grade) from student;
select MIN(grade) from student;
select SUM(grade) from student;
```
#### 字符串函数

字符串函数的使用格式：
```
select 函数(参数) from 表;
```

- CONCAT(str1,str2,...) ：返回结果为连接参数产生的字符串。如有任何一个参数为NULL ，则返回值为 NULL。

- ASCII(str)：返回字符串str 的最左字符的数值。假如str为空字符串，则返回值为 0 。假如str 为NULL，则返回值为 NULL。 ASCII()用于带有从 0到255的数值的字符。

- BIN(N)：返回N的二进制值的字符串。

- CHAR_LENGTH(str)：返回值为字符串str 的长度，长度的单位为字符。

- CONV(str,from_base,to_base)：不同进制的转换。返回str字符串由from_base进制转化为 to_base 进制的数字串表示。

- ELT(N,str1,str2,str3,...)：返回第一个参数后面的第N个参数。返回若N = 1，则返回值为  str1 ，若N = 2，则返回值为 str2 ，以此类推。

- FIELD(str,str1,str2,str3,...)：返回str1, str2, str3,……列表中的str 的个数。在找不到str 的情况下，返回值为 0 。

- FIND_IN_SET(str,strlist)：假如字符串str 在字符串列表strlist 中， 则返回其在列表中的位置。一个字符串列表就是一个由一些被‘,’符号分开的自链组成的字符串。

- FORMAT(X,D)：将number X设置为格式 '#,###,###.##', 以四舍五入的方式保留到小数点后D位, 而返回结果为一个字符串。

- INSERT(str,pos,len,newstr)：字符串 str的pos到len长的位置被newstr替换。

- INSTR(str,substr)：返回字符串 str 中子字符串substr的第一个出现位置。

- POSITION(substr IN str)：返回子串substr在字符串str第一个出现的位置，如果substr不是在str里面，返回0。

- LOCATE(substr,str,pos)返回子串substr在字符串str第一个出现的位置，从位置pos开始。如果substr不是在str里面，返回0。

- LEFT(str,len)：返回字符串str 从左起len长的子串。

- RIGHT(str,len)：从字符串str 开始，返回最右len 字符。

- LENGTH(str)：返回值为字符串str 的长度，单位为字节。

- LOWER(str)：把str全部变为小写。

- REPEAT(str,count)：str重复count次而成的新字符串。

- REPLACE(str,from_str,to_str)：把str中的from_str内容替换为to_str.

- REVERSE(str)：把str倒序。

- RPAD(str,len,padstr)：把str用padstr填充到len长。

- SUBSTRING(str,pos) , SUBSTRING(str FROM pos)：字符串str返回一个子字符串，起始于位置 pos。

- SUBSTRING(str,pos,len) , SUBSTRING(str FROM pos FOR len)：字符串str返回一个长度同len字符相同的子字符串，起始于位置 pos。

- TRIM([{BOTH | LEADING | TRAILING} [remstr] FROM] str) ，TRIM(remstr FROM str)：从str中删除remstr。

- UPPER(str)：把str转为大写。

#### 日期和时间函数

- NOW():返回该条语句【注意，是该条语句，而不是该函数。区别在于下面的sysdate（）函数】运行时的具体日期时间。
```
eg:select NOW();

re：2016-09-12 21:25:06
```
- sysdate()： 日期时间函数跟 now() 类似，不同之处在于：now() 在语句执行开始时值就得到了， sysdate() 在函数执行时动态得到值。
```
mysql> select now(), sleep(3), now();

+---------------------+----------+---------------------+
| now()               | sleep(3) | now()               |
+---------------------+----------+---------------------+
| 2008-08-08 22:28:21 |        0 | 2008-08-08 22:28:21 |
+---------------------+----------+---------------------+

mysql> select sysdate(), sleep(3), sysdate();

+---------------------+----------+---------------------+
| sysdate()           | sleep(3) | sysdate()           |
+---------------------+----------+---------------------+
| 2008-08-08 22:28:41 |        0 | 2008-08-08 22:28:44 |
+---------------------+----------+---------------------+
```
可以看到，虽然中途 sleep 3 秒，但 now() 函数两次的时间值是相同的； sysdate() 函数两次得到的时间值相差 3 秒。

- curdate()：获取当前日期。

- curtime()：获得当前时间（time）函数。

- 时间选取函数：
选取日期时间的各个部分：日期、时间、年、季度、月、日、小时、分钟、秒、微秒
```
set @dt = '2018-09-10 07:15:30.123456';

select date(@dt);        -- 2008-09-10
select time(@dt);        -- 07:15:30.123456
select year(@dt);        -- 2008
select quarter(@dt);     -- 3
select month(@dt);       -- 9
select week(@dt);        -- 36
select day(@dt);         -- 10
select hour(@dt);        -- 7
select minute(@dt);      -- 15
select second(@dt);      -- 30
select microsecond(@dt); -- 123456

dayname(), monthname()： 返回星期和月份名称。
select dayname(@dt);     -- Friday
select monthname(@dt);   -- August：
```
- WEEKDAY(date) ：返回date的星期索引(0=星期一，1=星期二, ……6= 星期天)。

- DAYOFMONTH(date) ：返回date的月份中日期，在1到31范围内。

- DAYOFYEAR(date) ：返回date在一年中的日数, 在1到366范围内。

- MONTH(date) ：返回date的月份，范围1到12。

- DAYNAME(date) ：返回date的星期名字，比如：Friday。

- MONTHNAME(date) ：返回date的月份名字。

- QUARTER(date) ：返回date一年中的季度，范围1到4。

#### 加密解密函数

- ENCODE(,)   DECODE(,)：加密解密字符串。该函数有两个参数：被加密或解密的字符串和作为加密或解密基础的密钥。Encode结果是一个二进制字符串，以BLOB类型存储。
```
加密 INSERT INTO users (username, password) VALUES ('joe', ENCODE('guessme', 'abracadabra'));
解密 SELECT DECODE(password, 'abracadabra') FROM users WHERE username='joe';
```

- MD5()：计算字符串的MD5校验和（128位）。

- SHA()：计算字符串的SHA校验和（160位）

#### 格式化函数

- FORMAT(x,y) ：把x格式化以逗号分隔开的数字序列，y是结果的小数位数。同时，结果会以 , 每3位数进行分割，像金额表达一样。

- DATE_FORMAT(date,fmt) 和TIME_FORMAT(time,fmt)函数可以用来格式化日期和时间值：这俩函数接受日期或者时间值和一个指定结果格式的格式化字符串。这个格式化字符串包含特殊的符号。不同的符号表示不同的内容，字符串就是一系列自选的符号组成，这样日期或时间就会按照字符串表示的格式表达出来。

| **说明符** | **说明**                                                     |
| ---------- | ------------------------------------------------------------ |
| %a         | 工作日的缩写名称  (Sun..Sat)                                 |
| %b         | 月份的缩写名称 (Jan..Dec)                                    |
| %c         | 月份，数字形式(0..12)                                        |
| %D         | 带有英语后缀的该月日期 (0th, 1st, 2nd, 3rd, …)               |
| %d         | 该月日期, 数字形式 (00..31)                                  |
| %e         | 该月日期, 数字形式(0..31)                                    |
| %f         | 微秒 (000000..999999)                                        |
| %H         | 小时(00..23)                                                 |
| %h         | 小时(01..12)                                                 |
| %I         | 小时 (01..12)                                                |
| %i         | 分钟,数字形式 (00..59)                                       |
| %j         | 一年中的天数 (001..366)                                      |
| %k         | 小时 (0..23)                                                 |
| %l         | 小时 (1..12)                                                 |
| %M         | 月份名称 (January..December)                                 |
| %m         | 月份, 数字形式 (00..12)                                      |
| %p         | 上午（AM）或下午（ PM）                                      |
| %r         | 时间 , 12小时制 (小时hh:分钟mm:秒数ss 后加 AM或PM)           |
| %S         | 秒 (00..59)                                                  |
| %s         | 秒 (00..59)                                                  |
| %T         | 时间 , 24小时制 (小时hh:分钟mm:秒数ss)                       |
| %U         | 周 (00..53), 其中周日为每周的第一天                          |
| %u         | 周 (00..53), 其中周一为每周的第一天                          |
| %V         | 周 (01..53), 其中周日为每周的第一天 ; 和 %X同时使用          |
| %v         | 周 (01..53), 其中周一为每周的第一天 ; 和 %x同时使用          |
| %W         | 工作日名称 (周日..周六)                                      |
| %w         | 一周中的每日 (0=周日..6=周六)                                |
| %X         | 该周的年份，其中周日为每周的第一天, 数字形式,4位数;和%V同时使用 |
| %x         | 该周的年份，其中周一为每周的第一天, 数字形式,4位数;和%v同时使用 |
| %Y         | 年份, 数字形式,4位数                                         |
| %y         | 年份, 数字形式 (2位数)                                       |
| %%         | ‘%’文字字符                                                  |

#### 类型转换函数

CAST() 和CONVERT() 函数可用来获取一个类型的值，并产生另一个类型的值。

格式：CAST(xxx  AS   类型)  ,   CONVERT(xxx,类型)

类型 可以是以下值其中的 一个：
```
二进制 : BINARY    
字符型,可带参数 : CHAR()     
日期 : DATE     
时间: TIME     
日 期时间型 : DATETIME     
浮点数 : DECIMAL      
整数 : SIGNED     
无符号整数 : UNSIGNED
```
#### 系统信息函数

使用格式：select 函数();
- VERSION()：返回数据库版本号。
- CONNECTION_ID():返回数据库的连接次数.
- DATABASE()、SCHEMA():返回当前数据库名
- USER()、SYSTEM_USER()、SESSION_USER()，CURRENT_USER()、CURRENT_USER:返回当前用户名
- CHARSET(str)：返回字符串str的字符集（编码格式）
- LAST_INSERT_ID() ：返回最近生成的AUTO_INCREMENT值.

#### 排名函数

如下表：

```mysql
CREATE TABLE employee(salary int,time_length int);
INSERT INTO employee VALUES(1,10),(2,9),(3,4),(3,8),(5,7),(6,1),(7,3),(8,2);
```

1. row_number
   row_number在排名时序号连续不重复，即使遇到表中的两个3时亦如此

   ```mysql
   select row_number() OVER(order by e.salary desc) as row_num , e.salary from employee e
   
   +-----------+----------+
   |   row_num |   salary |
   |-----------+----------|
   |         1 |        8 |
   |         2 |        7 |
   |         3 |        6 |
   |         4 |        5 |
   |         5 |        3 |
   |         6 |        3 |
   |         7 |        2 |
   |         8 |        1 |
   +-----------+----------+
   
   ```

   注意：在使用row_number实现分页时需要特别注意一点，over子句中的order by 要与Sql排序记录中的order by 保持一致，否则得到的序号可能不是连续的

   ```mysql
   select row_number() OVER(order by e.salary desc) as row_num , e.salary
   from employee e
   order by e.time_length
   
   +-----------+----------+
   |   row_num |   salary |
   |-----------+----------|
   |         3 |        6 |
   |         1 |        8 |
   |         2 |        7 |
   |         5 |        3 |
   |         4 |        5 |
   |         6 |        3 |
   |         7 |        2 |
   |         8 |        1 |
   +-----------+----------+
   
   ```

2. rank

   rank函数会把要求排序的值相同的归为一组且每组序号一样，排序不会连续

   ```mysql
   select rank() OVER(order by e.salary desc) as row_num , e.salary from employee e
   
   +-----------+----------+
   |   row_num |   salary |
   |-----------+----------|
   |         1 |        8 |
   |         2 |        7 |
   |         3 |        6 |
   |         4 |        5 |
   |         5 |        3 |
   |         5 |        3 |
   |         7 |        2 |
   |         8 |        1 |
   +-----------+----------+
   
   ```

3. dense_rank

   dense_rank排序是连续的，也会把相同的值分为一组且每组排序号一样

   ```mysql
   select dense_rank() OVER(order by e.salary desc) as row_num , e.salary from employee e
   +-----------+----------+
   |   row_num |   salary |
   |-----------+----------|
   |         1 |        8 |
   |         2 |        7 |
   |         3 |        6 |
   |         4 |        5 |
   |         5 |        3 |
   |         5 |        3 |
   |         6 |        2 |
   |         7 |        1 |
   +-----------+----------+
   
   ```

4. NTILE

   NTILE(group_num)将所有记录分成group_num个组，每组序号一样

   ```mysql
   select NTILE(2) OVER(order by e.salary desc) as row_num , e.salary from employee e
   +-----------+----------+
   |   row_num |   salary |
   |-----------+----------|
   |         1 |        8 |
   |         1 |        7 |
   |         1 |        6 |
   |         1 |        5 |
   |         2 |        3 |
   |         2 |        3 |
   |         2 |        2 |
   |         2 |        1 |
   +-----------+----------+
   ```

示例：

```mysql
create table students(
	id int(4)  auto_increment primary key,
	name varchar(50) not null, 
	score int(4) not null
	);
	
insert into students(name,score) values('curry', 100),
	('klay', 99),
	('KD', 100), 
	('green', 90), 
	('James', 99), 
	('AD', 96);
	
	
select id, name, rank() over(order by score desc) as r from students;
select id, name, DENSE_RANK() OVER(order by score desc) as dense_r from students;
select id, name, row_number() OVER(order by score desc) as row_r from students;

select id, name,score, rank() over(order by score desc) as r,
	DENSE_RANK() OVER(order by score desc) as dense_r,
	row_number() OVER(order by score desc) as row_r
from students;

-- 需要注意的一点是as后的别名，千万不要与前面的函数名重名，否则会报错。


```



### 自定义函数UDF

函数存储着一系列sql语句，调用函数就是一次性执行这些语句。所以函数可以降低语句重复。【但注意的是函数注重返回值，不注重执行过程，所以一些语句无法执行。所以函数并不是单纯的sql语句集合。】

UDF可以实现的功能不止于此,UDF有两个关键点,一个是参数,一个是返回值,UDF可以没有参数,但UDF必须有且只有一个返回值

函数与存储过程的区别：函数只会返回一个值，不允许返回一个结果集。函数强调返回值，所以函数不允许返回多个值的情况，即使是查询语句。

**创建**

```
create function 函数名([参数列表]) returns 数据类型
begin
 sql语句;
 return 值;
end;
```
参数列表的格式是：  变量名 数据类型

示例：
```
-- 最简单的仅有一条sql的函数
create function myselect2() returns int return 666;
select myselect2(); -- 调用函数

CREATE FUNCTION simpleFun() RETURNS VARCHAR(20) RETURN "Hello World!";

select simpleFun();

--
create function myselect3() returns int
begin
    declare c int;
    select id from class where cname="python" into c;
    return c;
end;
select myselect3();
-- 带传参的函数
create function myselect5(name varchar(15)) returns int
begin
    declare c int;
    select id from class where cname=name into c;
    return c;
end;
select myselect5("python");
```

报错：

```
(1418, 'This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)')

因为CREATE PROCEDURE, CREATE FUNCTION, ALTER PROCEDURE,ALTER FUNCTION,CALL, DROP PROCEDURE, DROP FUNCTION等语句都会被写进二进制日志,然后在从服务器上执行。但是，一个执行更新的不确定子程序(存储过程、函数、触发器)是不可重复的，在从服务器上执行(相对与主服务器是重复执行)可能会造成恢复的数据与原始数据不同，从服务器不同于主服务器的情况。

```

解决办法也有两种，

1. 第一种是在创建子程序(存储过程、函数、触发器)时，声明为DETERMINISTIC或NO SQL与READS SQL DATA中的一个，

   ```
   CREATE DEFINER = CURRENT_USER PROCEDURE `NewProc`()
       DETERMINISTIC
   BEGIN
    #Routine body goes here...
   END;;
   ```

2. 第二种是信任子程序的创建者，禁止创建、修改子程序时对SUPER权限的要求，设置log_bin_trust_routine_creators全局系统变量为1。设置方法有三种:

   ```
   1.在客户端上执行SET GLOBAL log_bin_trust_function_creators = 1;
   2.MySQL启动时，加上–log-bin-trust-function-creators选贤，参数设置为1
   3.在MySQL配置文件my.ini或my.cnf中的[mysqld]段上加log-bin-trust-function-creators=1
   
   ```

**调用**
直接使用函数名()就可以调用【虽然这么说，但返回的是一个结果，sql中不使用select的话任何结果都无法显示出来（所以单纯调用会报错），】
如果想要传入参数可以使用函数名(参数)
调用方式【下面调用的函数都是上面中创建的。】：

```
-- 无参调用
select myselect3();
-- 传参调用
select myselect5("python");
select * from class where id=myselect5("python");
```

**查看**

查看函数创建语句：`show create function 函数名;`

查看所有函数：`show function status [like 'pattern'];`

**修改**

函数的修改只能修改一些如comment的选项，不能修改内部的sql语句和参数列表。
```
alter function 函数名 选项；
```

**删除**

```
drop function 函数名;
```


#### 语法

在函数体中我们可以使用更为复杂的语法,比如复合结构/流程控制/任何SQL语句/定义变量等等。

在函数体中,如果包含多条语句,我们需要把多条语句放到BEGIN...END语句块中
```
DELIMITER //
CREATE FUNCTION IF EXIST deleteById(uid SMALLINT UNSIGNED)
RETURNS VARCHAR(20)
BEGIN
DELETE FROM son WHERE id = uid;
RETURN (SELECT COUNT(id) FROM son);
END//
```
DELIMITER // 意思是修改默认的结束符";"为"//",以后的SQL语句都要以"//"作为结尾

特别说明:UDF中,REURN语句也包含在BEGIN...END中

**自定义函数中定义局部变量语法:**
```
DECLARE var_name[,varname]...date_type [DEFAULT VALUE];
```
简单来说就是:
```
DECLARE 变量1[,变量2,... ]变量类型 [DEFAULT 默认值]
```
这些变量的作用范围是在BEGIN...END程序中,而且定义局部变量语句必须在BEGIN...END的第一行定义

```
DELIMITER //

CREATE FUNCTION addTwoNumber(x SMALLINT UNSIGNED, Y SMALLINT UNSIGNED)
RETURNS SMALLINT
BEGIN
DECLARE a, b SMALLINT UNSIGNED DEFAULT 10;
SET  a = x, b = y;
RETURN a+b;
END//
DELIMITER ;

SELECT addTwoNumber(1,2);

DROP FUNCTION addTwoNumber;
```
上边的代码只是把两个数相加,当然,没有必要这么写,只是说明局部变量的用法,还是要说明下:这些局部变量的作用范围是在BEGIN...END程序中

**为变量赋值语法:**

```
SET parameter_name = value[,parameter_name = value...]

SELECT  INTO parameter_name
```
示例：
```
DECLARE x int;
SELECT COUNT(id) FROM tdb_name INTO x;
RETURN x;
END//
```

**用户变量定义语法:(可以理解成全局变量)**
```
SET @param_name = value

SET @allParam = 100;
SELECT @allParam;
```
上述定义并显示@allParam用户变量,其作用域只为当前用户的客户端有效

**自定义函数中流程控制语句语法:**

存储过程和函数中可以使用流程控制来控制语句的执行。

MySQL中可以使用IF语句、CASE语句、LOOP语句、LEAVE语句、ITERATE语句、REPEAT语句和WHILE语句来进行流程控制。

每个流程中可能包含一个单独语句，或者是使用BEGIN...END构造的复合语句，构造可以被嵌套

参考存储过程章节。

