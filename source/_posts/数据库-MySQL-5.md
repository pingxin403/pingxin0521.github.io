---
title: MySQL SQL 存储过程（六）
date: 2019-05-12 15:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---
### 前言
SQL语句需要先编译然后执行，而存储过程（Stored Procedure）是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。

<!--more-->

存储过程是可编程的函数，在数据库中创建并保存，可以由SQL语句和控制结构组成。当想要在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的。数据库中的存储过程可以看做是对编程中面向对象方法的模拟，它允许控制数据的访问方式。

存储过程是存储在数据库目录中的一段声明性SQL语句，优点有：

1. 通常存储过程有助于提高应用程序的性能
2. 存储过程有助于减少应用程序和数据库服务器之间的流量
3. 存储的程序对任何应用程序都是可重用的和透明的
4. 存储的程序是安全的

**需要配置变量**

```
set global log_bin_trust_function_creators=1;
```


### 存储过程的创建

#### 语法
```
CREATE PROCEDURE sp_name ([proc_parameter[,...]]) [characteristic ...] routine_body
CREATE PROCEDURE  过程名([[IN|OUT|INOUT] 参数名 数据类型[,[IN|OUT|INOUT] 参数名 数据类型…]]) [特性 ...] 过程体
```
其中，sp_name参数是存储过程的名称；proc_parameter表示存储过程的参数列表； characteristic参数指定存储过程的特性；routine_body参数是SQL代码的内容，可以用BEGIN…END来标志SQL代码的开始和结束。

proc_parameter中的每个参数由3部分组成。这3部分分别是输入输出类型、参数名称和参数类型。其形式如下：
```
[ IN | OUT | INOUT ] param_name type(param_size)
```
其中，IN表示输入参数；OUT表示输出参数； INOUT表示既可以是输入，也可以是输出； param_name参数是存储过程的参数名称；type参数指定存储过程的参数类型，该类型可以是MySQL数据库的任意数据类型。

characteristic参数有多个取值。其取值说明如下：

- LANGUAGE SQL：说明routine_body部分是由SQL语言的语句组成，这也是数据库系统默认的语言。

- [NOT] DETERMINISTIC：指明存储过程的执行结果是否是确定的。DETERMINISTIC表示结果是确定的。每次执行存储过程时，相同的输入会得到相同的输出。NOT DETERMINISTIC表示结果是非确定的，相同的输入可能得到不同的输出。默认情况下，结果是非确定的。

- { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }：指明子程序使用SQL语句的限制。CONTAINS SQL表示子程序包含SQL语句，但不包含读或写数据的语句；NO SQL表示子程序中不包含SQL语句；READS SQL DATA表示子程序中包含读数据的语句；MODIFIES SQL DATA表示子程序中包含写数据的语句。默认情况下，系统会指定为CONTAINS SQL。

- SQL SECURITY { DEFINER | INVOKER }：指明谁有权限来执行。DEFINER表示只有定义者自己才能够执行；INVOKER表示调用者可以执行。默认情况下，系统指定的权限是DEFINER。

- COMMENT 'string'：注释信息。

技巧：创建存储过程时，系统默认指定CONTAINS SQL，表示存储过程中使用了SQL语句。但是，如果存储过程中没有使用SQL语句，最好设置为NO SQL。而且，存储过程中最好在COMMENT部分对存储过程进行简单的注释，以便以后在阅读存储过程的代码时更加方便。

下面创建一个名为num_from_employee的存储过程。代码如下：

```
-- ----------------------------
-- Procedure structure for `proc_adder`
-- ----------------------------
DROP PROCEDURE IF EXISTS `proc_adder`;
DELIMITER //
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_adder`(IN a int, IN b int, OUT sum int)
BEGIN
    #Routine body goes here...

    DECLARE c int;
    if a is null then set a = 0;
    end if;

    if b is null then set b = 0;
    end if;

    set sum  = a + b;
END//
DELIMITER ;

set @b=5;
call proc_adder(2,@b,@s);
select @s as sum;
```

说明：MySQL中默认的语句结束符为分号（;）。存储过程中的SQL语句需要分号来 结束。为了避免冲突，首先用"DELIMITER //"将MySQL的结束符设置为//。最后再用"DELIMITER ;"来将结束符恢复成分号。这与创建触发器和函数时是一样的。

#### 调用存储过程
要调用存储过程，可以使用以下SQL命令：
```
CALL sp_name(param01,param02,..);
```
#### 删除
存储过程在创建之后，被保存在服务器上以供使用，直至被删除。删除命令（类似于第 21章所介绍的语句）从服务器中删除存储过程。为删除刚创建的存储过程，可使用以下语句：
```
DROP PROCEDURE sp_name;
```
这条语句删除刚创建的存储过程。请注意没有使用后面的 ()，只给出存储过程名。
仅当存在时删除 :如果指定的过程不存在，则 DROP PROCEDURE将产生一个错误。当过程存在想删除它时（如果过程不存在也不产生错误）可使用

```
 DROP PROCEDURE IF EXISTS sp_name；
```
#### 检查存储过程
为显示用来创建一个存储过程的 CREATE语句，使用SHOW CREATE PROCEDURE语句：
```
show create procedure sp_name;
```
为了获得包括何时、由谁创建等详细信息的存储过程列表，使用 `SHOW PROCEDURE STATUS`。
注意：限制过程状态结果，`SHOW PROCEDURE STATUS`列出所有存储过程。为限制其输出，可使用 LIKE指定一个过滤模式，例如：

```
SHOW PROCEDURE STATUS like 'sp_name';
```

#### 变量
如果希望MySQL执行批量插入的操作，那么至少要有一个计数器来计算当前插入的是第几次。

这里的变量是用在存储过程中的SQL语句中的，变量的作用范围在BEGIN .... END 中。

没有DEFAULT子句，初始值为NULL。

```
DECLARE name,address VARCHAR;  -- 发现了吗，SQL中一般都喜欢先定义变量再定义类型，与Java是相反的。
DECLARE age INT DEFAULT 20; -- 指定默认值。若没有DEFAULT子句，初始值为NULL。
```
为变量赋值
```
SET name = 'jay';  -- 为name变量设置值
DECLARE var1,var2,var3 INT;
SET var1 = 10,var2 = 20;  -- 其实为了简化记忆其语法，可以分开来写
-- SET var1 = 10;
-- SET var2 = 20;
SET var3 = var1 + var2;
```
示例
```
DROP PROCEDURE IF EXISTS contStById;
DELIMITER //  -- 定义存储过程结束符号为//
CREATE PROCEDURE contStById(IN sid INT(11),OUT result INT(11)) -- 定义输入变量
BEGIN
    DECLARE sCount INT;
    SELECT COUNT(*) INTO sCount FROM t_student WHERE id = sid;
    SET result = sCount; -- 用变量为输出结果设值
END // -- 结束符要加
DELIMITER ;  -- 重新定义存储过程结束符为分号

CALL contStById(1,@result);
SELECT @result;
```
显然，在存储过程中的变量，可以直接与输出变量进行相应的计算。本例直接把sCount这个变量的值赋值到输出中。

一个变量有自己的范围(作用域)，它用来定义它的生命周期。 如果在存储过程中声明一个变量，那么当达到存储过程的END语句时，它将超出范围，因此在其它代码块中无法访问。

如果您在BEGIN END块内声明一个变量，那么如果达到END，它将超出范围。 可以在不同的作用域中声明具有相同名称的两个或多个变量，因为变量仅在自己的作用域中有效。 但是，在不同范围内声明具有相同名称的变量不是很好的编程习惯。
以`@`符号开头的变量是会话变量。直到会话结束前它可用和可访问。

#### 流程控制

**CASE语句：**

1. 简单函数

   ```mysql
   CASE case_value
       WHEN when_value THEN statement_list
       [WHEN when_value THEN statement_list] ...
       [ELSE statement_list]
   END CASE
   
   -- 示例
   case `gender`
   when 1 then '男'
   when 2 then '女'
   else '未知'
   end
   ```

2. case搜索函数

   ```mysql
   CASE
       WHEN search_condition THEN statement_list
       [WHEN search_condition THEN statement_list] ...
       [ELSE statement_list]
   END CASE
   
   -- 示例
   case 
   when gender = 1 then '男'
   when gender = 2 then '女'
   else '未知' 
   end
   ```

在存储过程中使用：

```mysql
-- ----------------------------
-- Procedure structure for `proc_case`
-- ----------------------------
DROP PROCEDURE IF EXISTS `proc_case`;
DELIMITER //
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_case`(IN type int)
BEGIN
    #Routine body goes here...
    DECLARE c varchar(500);
    CASE type
    WHEN 0 THEN
        set c = 'param is 0';
    WHEN 1 THEN
        set c = 'param is 1';
    ELSE
        set c = 'param is others, not 0 or 1';
    END CASE;
    select c;
END //
DELIMITER ;
```

**IF语句：**

mysql 的 if 既可以作为表达式用，也可以在存储过程中作为流程控制语句使用

1. if 表达式

   ```
   IF(expr1,expr2,expr3)
   ```

   如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL), 则 IF() 的返回值为 expr2；否则返回值则为 expr3。IF() 的返回值为数字值或字符串值，具体情况视其所在语境而定。

   ```csharp
   select if(sva=1,"男","女") as ssva from taname where id = '111'
   ```

   作为表达式的 IF 也可以使用 CASE WHEN 来实现：

   ```csharp
   select CASE sva WHEN 1 THEN "男" ELSE "女" END as ssva from taname where id = '1';
   ```

   在第一个方案的返回结果中，value=compare-valu。而第二个方案的返回结果是第一种情况的真实结果。如果没有匹配的结果值，则返回结果为 ELSE 后的结果，如果没有 ELSE部分，则返回值为 NULL。

   ```php
   select CASE 1 WHEN 1 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END as testCol
   ```

   将输出one

2. IF ELSE 作为流程控制语句使用

   if 实现条件判断，满足不通条件执行不通的操作，这个我们只要学习编程的都知道 if 的作用了，下面我们来看下 mysql 存储过程中的 if 是如何使用的吧

   ```mysql
   IF search_condition THEN 
       statement_list
   [ELSEIF search_condition THEN]
       statement_list ...
   [ELSE statement_list]
    END IF
   ```

   与 PHP 中的 IF 语句类似，当 IF 中条件search_condition 成立时，执行 THEN 后的 statement_list 语句，否则判断 ELSEIF中的条件，成立则执行其后的 statement_list 语句，否则继续判断其他分支。当所有分支的条件均不成立时，执行 ELSE 分支。search_condition 是一个条件表达式，可以由 " = 、 < 、<= 、 > 、 >= 、!= " 等条件运算符组成， 并且可以使用 AND 、 OR 、 NOT、 对多个表达式进行组合。

   ```mysql
   -- ----------------------------
   -- Procedure structure for `proc_if`
   -- ----------------------------
   DROP PROCEDURE IF EXISTS `proc_if`;
   DELIMITER //
   CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_if`(IN type int)
   BEGIN
       #Routine body goes here...
       DECLARE c varchar(500);
       IF type = 0 THEN
           set c = 'param is 0';
       ELSEIF type = 1 THEN
           set c = 'param is 1';
       ELSE
           set c = 'param is others, not 0 or 1';
       END IF;
       select c;
   END //
   DELIMITER ;
   ```

   注意：IF作为一条语句，在END IF后需要加上分号“;”以表示语句结束，其他语句如CASE、LOOP等也是相同的。

**循环while语句：**

**思考：**while循环是否只能使用在存储过程或者存储函数之中，不能直接在查询语句中使用？

循环一般在存储过程和存储函数中使用。

创建一个带条件判断的循环过程与REPEAT不同的是，WHILE在语句执行时，先对指定的条件进行判断，如果为真，则执行循环内的语句，否则退出循环

语法格式
```
[while_lable:] WHILE expr_condition DO
Statement_list
END WHILE [while_lable]
```
参数说明

While_lable，为WHILE语句的标注名称
Expr_condition，为进行判断的表达式，如果表达式为真，WHILE语句内的语句，或语句群就被执行，直至expr_condition为假，退出循环

```
-- ----------------------------
-- Procedure structure for `proc_while`
-- ----------------------------
DROP PROCEDURE IF EXISTS `proc_while`;
DELIMITER //
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_while`(IN n int)
BEGIN
    #Routine body goes here...
    DECLARE i int;
    DECLARE s int;
    SET i = 0;
    SET s = 0;
    WHILE i <= n DO
        set s = s + i;
        set i = i + 1;
    END WHILE;
    SELECT s;
END //
DELIMITER ;
```
**LOOP语句:**

LOOP循环语句，用来重复执行某些语句
与IF和CASE语句相比，LOOP只是创建一个循环操作的过程，并不进行条件判断
LOOP内的语句一直重复执行，知道跳出循环语句
语法：
```
[loop_label:] LOOP
Statement_list
END LOOP [loop_label]
```
Loop_label，表示LOOP语句的标注名称，该参数可以省略
Statement，表示需要循环执行的语句
```
DECLARE id INT DEFAULT 0;
qdd_loop:LOOP
SET id=id+1;
IF id>=10 THEN LEAVE add_loop;
END IF;
END LOOP add_loop;
```
循环执行了id加1的操作,当id值小于10时，循环重复执行，当id值大于或者等于10时，使用LEAVE语句退出循环

**LEAVE语句**

用于退出任何被标注的流程控制结构
语法格式
```
LEAVE label
```
参数说明

label，表示循环的标志
通常情况下，LEAVE语句与BEGIN……END、循环语句一起使用

**ITERATE语句**

ITERATE，意思是再次循环,用于将执行顺序转到语句段的开头处
语法格式
```
ITERATE lable
```
参数说明

Lable，表示循环的标志
注意，ITERATE语句只可以出现在，LOOP、REPEAT和WHILE语句中

演示ITERATE语句，在LOOP语句内的使用
```
CREATE PROCEDURE doiterate()
BEGIN
DECLARE p1 INT DEFAULT 0;
My_loop:LOOP
SET p1=p1+1;
IF p1<10 THEN ITERATE my_loop;
ELSEIF p1>20 THEN LEAVE my_loop;
END IF;
SELECT ‘p1 is between 10 and 20’;
END LOOP my_loop;
END
```
P1的初始值为0，如果，p1的值小于10时，重复执行p1加1的操作，当p1大于或等于10，并且小于20时，打印消息p1 is between 10 and 20，当p1大于20时，退出循环

**REPEAT语句**

用于创建一个带有条件判断的循环过程
每次语句执行完毕之后，会对条件表达式进行判断，如果表达式为真，则循环结束，否则，重复执行循环中的语句
语法格式
```
[repeat_lable:] REPEAT
Statement_list
UNTIL expr_condition
END REPEAT [repeat_lable]
```
参数说明

Repeat_lable，为REPEAT语句的标注名称，该参数是可选的
REPEAT语句内的语句，或语句群被重复，直至expr_condition为真

使用REPEAT语句，执行循环过程
```
DECLARE id INT DEFAULT 0;
REPEAT
SET id=id+1;
UNTIL id>=10;
END REPEAT;
```

#### 游标

mysql 游标还有它的一些特性：
1. 服务器端

  有些数据库服务器可以同时运行服务器端和客户端游标。服务器游标在数据库中管理，而客户端游标可以在数据库之外的应用程序（程序语言等）中生成，控制。MySQL 只支持服务器端游标，也就是说，所有的游标必须和存储过程一样，事先写在数据库中，在各语言中只能进行请求调用。
2. 只读

  游标可以是可读和可写的。只读游标可以从数据库中读取数据，而可写游标可以更新由游标指向的数据。MySQL 只支持只读游标，也就是说，只能进行数据查询。
3. 敏感

  游标可以是敏感的，也可以是不敏感的。敏感游标引用数据库中的实际数据，而不敏感的游标指向在创建游标时建立的数据临时副本。MySQL 只支持敏感游标，因为是只读的，所以不用考虑破坏数据，因此直接对数据进行操作也没问题。
4. 只向前

  高级的游标实现可以向后和向前遍历数据集，跳过记录，完成大量其它的导航任务。目前 MySQL 游标只是向前的，意味着只能向前遍历数据集。此外，MySQL 游标一次只能向前移动一条记录，不能跳过。

使用游标需要遵循下面步骤：

1. 首先用DECLARE语句声明一个游标
```
DECLARE cursor_name CURSOR FOR SELECT_statement;  
```
 上面这条语句就对我们执行的select语句返回的记录指定了一个游标   
2. 其次需要使用OPEN语句来打开上面你定义的游标
```
OPEN cursor_name;  
```
3. 接下来你可以用FETCH语句来获得下一行数据，并且游标也将移动到对应的记录上(这个就类似java里面的那个iterator)。
```
FETCH cursor_name INTO variable list;  
```
4. 然后最后当我们所需要进行的操作都结束后我们要把游标释放掉。
```
    CLOSE cursor_name;  
```
在使用游标时需要注意的是，使用定义一个针对NOT FOUND的条件处理函数(condition handler)来避免出现“no data to fetch”这样的错误，条件处理函数就是当某种条件产生时所执行的代码，这里但我们游标指到记录的末尾时，便达到NOT FOUND这样条件，这个时候我们希望继续进行后面的操作，所以我们会在下面的代码中看到一个CONTINUE。

下面的游标使用演示,这个代码纯粹演示如何使用，在这里没有其他任何意义:)
```
drop procedure IF EXISTS test_proc;
delimiter //
create procedure test_proc()
begin
	-- 声明一个标志done， 用来判断游标是否遍历完成
	DECLARE done INT DEFAULT 0;

	-- 声明一个变量，用来存放从游标中提取的数据
	-- 特别注意这里的名字不能与由游标中使用的列明相同，否则得到的数据都是NULL
	DECLARE tname varchar(50) DEFAULT NULL;
	DECLARE tpass varchar(50) DEFAULT NULL;

	-- 声明游标对应的 SQL 语句
	DECLARE cur CURSOR FOR
		select `name`,`passwd` from `ims`.`tb_user`;

	-- 在游标循环到最后会将 done 设置为 1
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

	-- 执行查询
	open cur;
	-- 遍历游标每一行
	REPEAT
		-- 把一行的信息存放在对应的变量中
		FETCH cur INTO tname, tpass;
		if not done then
			-- 这里就可以使用 tname， tpass 对应的信息了
			select tname, tpass;
		end if;
 	UNTIL done END REPEAT;
	CLOSE cur;
end
//
delimiter ;

-- 执行存储过程
call test_proc();
```

需要注意的是变量的声明、游标的声明和HANDLER声明的顺序不能搞错，必须是先声明变量，再申明游标，最后声明HANDLER。上述存储过程的例子中只使用了一个游标，那么如果要使用两个或者更多游标怎么办，其实很简单，可以这么说，一个怎么用两个就是怎么用的。

这里需要注意的是，在遍历第二个游标前使用了set done = 0，因为当第一个游标遍历玩后其值被handler设置为1了，如果不用set把它设置为 0 ，那么第二个游标就不会遍历了。当然好习惯是在每个打开游标的操作前都用该语句，确保游标能真正遍历。当然还可以使用begin语句块嵌套的方式来处理多个游标
```
drop procedure IF EXISTS test_proc_2;
delimiter //
create procedure test_proc_2()
begin
	DECLARE done INT DEFAULT 0;
	DECLARE tname varchar(50) DEFAULT NULL;
	DECLARE tpass varchar(50) DEFAULT NULL;

	DECLARE cur_1 CURSOR FOR
		select name, password from netingcn_proc_test;

	DECLARE cur_2 CURSOR FOR
		select id, name from netingcn_proc_test;

	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

	open cur_1;
	REPEAT
		FETCH cur_1 INTO tname, tpass;
		if not done then
			select tname, tpass;
		end if;
 	UNTIL done END REPEAT;
	CLOSE cur_1;

	begin
		DECLARE done INT DEFAULT 0;
		DECLARE tid int(11) DEFAULT 0;
		DECLARE tname varchar(50) DEFAULT NULL;

		DECLARE cur_2 CURSOR FOR
			select id, name from netingcn_proc_test;

		DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

		open cur_2;
		REPEAT
			FETCH cur_2 INTO tid, tname;
			if not done then
				select tid, tname;
			end if;
	 	UNTIL done END REPEAT;
		CLOSE cur_2;
	end;
end
//
delimiter ;

call test_proc_2();
```

#### java使用函数和存储过程

java使用函数

修改mysql语句的结束符为//,定义一个函数，完成字符串拼接
```
delimiter //
create function hello( s char(20) ) returns char(50)
return concat('hello，',s,'!');
//
delimiter ;
select hello('world');
```
使用
```
/**
 * 演示Java调用Mysql的函数
 * @author AdminTC
 */
public class TestJavaCallMysqlFunc {
    public static void main(String[] args) throws Exception{
        String  sql = "{? = call hello(?)}";

        Connection conn = JdbcUtil.getConnection();
        CallableStatement cstmt = conn.prepareCall(sql);

        //第一个输出的?设置类型
        cstmt.registerOutParameter(1,Types.VARCHAR);

        //第二个输入的?设置值
        cstmt.setString(2,"hyp");

        //调用函数
        cstmt.execute();

        //接收返回的值
        String value = cstmt.getString(1);

        //显示
        System.out.println(value);

        JdbcUtil.close(cstmt);
        JdbcUtil.close(conn);
    }
}
```


**JDBC调用存储过程: CallableStatement**

在Java里面调用存储过程，写法那是相当的固定：
```
Class.forName(....
Connection conn = DriverManager.getConnection(....
/**
*p是要调用的存储过程的名字，存储过程的4个参数，用4个？号占位符代替
*其余地方写法固定
*/
CallableStatement cstmt = conn.prepareCall("{call p(?,?,?,?)}");
/**
*告诉JDBC，这些个参数，哪些是输出参数，输出参数的类型用java.sql.Types来指定
*下面的意思是，第3个？和第4个？是输出参数，类型是INTEGER的
*Types后面具体写什么类型，得看你的存储过程参数怎么定义的
*/
cstmt.registerOutParameter(3, Types.INTEGER);
cstmt.registerOutParameter(4, Types.INTEGER);
/**
*在我这里第1个？和第2个？是输入参数，第3个是输出参数，第4个既输入又输出
*下面是设置他们的值,第一个设为3，第二个设为4，第4个设置为5
*没设第3个，因为它是输出参数
*/
cstmt.setInt(1, 3);
cstmt.setInt(2, 4);
cstmt.setInt(4, 5);
//执行
cstmt.execute();
//把第3个参数的值当成int类型拿出来
int three = cstmt.getInt(3);
System.out.println(three);
//把第4个参数的值当成int类型拿出来
int four = cstmt.getInt(4);
System.out.println(four);
//用完别忘给人家关了，后开的先关
cstmt.close();
conn.close();
```
**java获取存储过程返回的结果集**

1. 创建一张表，插入数据
```
CREATE TABLE TEST_TABLE(
       ID NUMBER(16),
       NAME VARCHAR2(20)
);

insert into TEST_TABLE (ID, NAME) values ('1', '张三');
insert into TEST_TABLE (ID, NAME) values ('2', '李四');
```
2. 创建存储过程
```
--包头
CREATE OR REPLACE PACKAGE PKG_TEST IS
  TYPE CURSOR_TYPE IS REF CURSOR;--用于接收结果的游标
  PROCEDURE P_TEST(O_CURSOR OUT CURSOR_TYPE);--存储过程
END PKG_TEST;
--包体
CREATE OR REPLACE PACKAGE BODY PKG_TEST IS
  PROCEDURE P_TEST(O_CURSOR OUT CURSOR_TYPE) IS--输出的是包头中定义的游标
  BEGIN
    OPEN O_CURSOR FOR
      SELECT ID, NAME FROM TEST_TABLE;--将查询结果放入游标中
  END P_TEST;
END PKG_TEST;
```
3. 编写java代码
```
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

import oracle.jdbc.OracleCallableStatement;

public class Test {
 public static void main(String[] args) {
  Connection conn = null;
  CallableStatement proc = null;
  ResultSet rs = null;
  try {
   Class.forName("oracle.jdbc.driver.OracleDriver");
   conn = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:orcl", "scott", "tiger");

   proc = conn.prepareCall("{ call PKG_TEST.P_TEST(?) }");//存储过程包.名
   proc.registerOutParameter(1, oracle.jdbc.OracleTypes.CURSOR);//需要注册输出的参数
   proc.execute();//执行存储过程

   rs = ((OracleCallableStatement) proc).getCursor(1);//获取结果集
   while (rs.next()) {
    System.out.println("ID:" + rs.getInt("ID") + " NAME:" + rs.getString("NAME"));
   }
  } catch (ClassNotFoundException e) {
   e.printStackTrace();
  } catch (SQLException e) {
   e.printStackTrace();
  } finally {
   try {
    if (rs != null) rs.close();
    if (conn != null && !conn.isClosed()) conn.close();
    rs = null;
    conn = null;
   } catch (SQLException e) {
    e.printStackTrace();
   }
  }
 }
}
```
