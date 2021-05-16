---
title: MySQL SQL 数据查询（三）--实际例子
date: 2019-05-06 13:18:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

运行环境：MySQL 8.0

### 示例一

<!--more-->

参考：

1. <https://blog.csdn.net/flycat296/article/details/63681089>
2. <https://www.jianshu.com/p/e6a25cfbd330>
3. <https://www.jianshu.com/p/476b52ee4f1b>

**要求：**

1. 不仅仅在本数据示例上完成查询要求，需要考虑其普适性
2. 在可用性的前提下，尽可能提升查询性能。

**表名和字段**

1. 学生表 
   Student(s_id,s_name,s_birth,s_sex) –学生编号,学生姓名, 出生年月,学生性别 
2. 课程表 
   Course(c_id,c_name,t_id) – –课程编号, 课程名称, 教师编号 
3. 教师表 
   Teacher(t_id,t_name) –教师编号,教师姓名 
4. 成绩表 
   Score(s_id,c_id,s_score) –学生编号,课程编号,分数

**测试数据**

```mysql
create database if not exists test;
use test;
-- 建表
-- 学生表
CREATE TABLE IF NOT EXISTS `Student`(
    `s_id` VARCHAR(20),
    `s_name` VARCHAR(20) NOT NULL DEFAULT '',
    `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
    `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
    PRIMARY KEY(`s_id`)
);
-- 课程表
CREATE TABLE IF NOT EXISTS  `Course`(
    `c_id`  VARCHAR(20),
    `c_name` VARCHAR(20) NOT NULL DEFAULT '',
    `t_id` VARCHAR(20) NOT NULL,
    PRIMARY KEY(`c_id`)
);
-- 教师表
CREATE TABLE IF NOT EXISTS  `Teacher`(
    `t_id` VARCHAR(20),
    `t_name` VARCHAR(20) NOT NULL DEFAULT '',
    PRIMARY KEY(`t_id`)
);
-- 成绩表
CREATE TABLE IF NOT EXISTS  `Score`(
    `s_id` VARCHAR(20),
    `c_id`  VARCHAR(20),
    `s_score` INT(3),
    PRIMARY KEY(`s_id`,`c_id`)
);
-- 插入学生表测试数据
insert into Student values('01' , '赵雷' , '1990-01-01' , '男'),('02' , '钱电' , '1990-12-21' , '男'),('03' , '孙风' , '1990-05-20' , '男'),('04' , '李云' , '1990-08-06' , '男'),('05' , '周梅' , '1991-12-01' , '女'),('06' , '吴兰' , '1992-03-01' , '女'),('07' , '郑竹' , '1989-07-01' , '女'),('08' , '王菊' , '1990-01-20' , '女');
-- 课程表测试数据
insert into Course values('01' , '语文' , '02'),('02' , '数学' , '01'),('03' , '英语' , '03');
 
-- 教师表测试数据
insert into Teacher values('01' , '张三'),('02' , '李四'),('03' , '王五');
 
-- 成绩表测试数据
insert into Score values('01' , '01' , 80),('01' , '02' , 90),('01' , '03' , 99),('02' , '01' , 70),('02' , '02' , 60),('02' , '03' , 80),('03' , '01' , 80),('03' , '02' , 80),('03' , '03' , 80),('04' , '01' , 50),('04' , '02' , 30),('04' , '03' , 20),('05' , '01' , 76),('05' , '02' , 87),('06' , '01' , 31),('06' , '03' , 34),('07' , '02' , 89),('07' , '03' , 98);
```

#### 练习题和sql语句

```mysql
-- 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数  

SELECT s.*, a.s_score AS 01_score, b.s_score AS 02_score
FROM `Student` s
	JOIN `Score` a
	ON s.s_id = a.s_id
		AND a.c_id = '01'
	LEFT JOIN `Score` b
	ON s.s_id = b.s_id
		AND b.c_id = '02'
		OR b.c_id = NULL
WHERE a.s_score > b.s_score;


-- 或者
SELECT s.*, a.s_score AS score_01, b.s_score AS score_02
FROM `Student` s, (
		SELECT s_id, s_score
		FROM `Score`
		WHERE c_id = '01'
	) a, (
		SELECT s_id, s_score
		FROM `Score`
		WHERE c_id = '02'
	) b
WHERE a.s_id = b.s_id
	AND a.s_score > b.s_score
	AND s.s_id = a.s_id;

-- 结果：
+--------+----------+------------+---------+------------+------------+
|   s_id | s_name   | s_birth    | s_sex   |   01_score |   02_score |
|--------+----------+------------+---------+------------+------------|
|     02 | 钱电     | 1990-12-21 | 男      |         70 |         60 |
|     04 | 李云     | 1990-08-06 | 男      |         50 |         30 |
+--------+----------+------------+---------+------------+------------+
 
 
-- 2、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

SELECT s.s_name, sc.s_id
	, ROUND(AVG(sc.s_score), 2) AS avg_score
FROM `Student` s
	INNER JOIN `Score` sc ON s.s_id = sc.s_id
GROUP BY sc.s_id
HAVING avg_score >= 60;


-- 结果
+--------+----------+-------------+
|   s_id | s_name   |   avg_score |
|--------+----------+-------------|
|     01 | 赵雷     |       89.67 |
|     02 | 钱电     |       70    |
|     03 | 孙风     |       80    |
|     05 | 周梅     |       81.5  |
|     07 | 郑竹     |       93.5  |
+--------+----------+-------------+

-- 3、查询在 SC 表存在成绩的学生信息
SELECT *
FROM Student
WHERE s_id IN (
	SELECT s_id
	FROM `Score`
	WHERE s_score IS NOT NULL
)

+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     01 | 赵雷     | 1990-01-01 | 男      |
|     02 | 钱电     | 1990-12-21 | 男      |
|     03 | 孙风     | 1990-05-20 | 男      |
|     04 | 李云     | 1990-08-06 | 男      |
|     05 | 周梅     | 1991-12-01 | 女      |
|     06 | 吴兰     | 1992-03-01 | 女      |
|     07 | 郑竹     | 1989-07-01 | 女      |
+--------+----------+------------+---------+

 
-- 4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
        -- (包括有成绩的和无成绩的)
 
SELECT s.s_id, s.s_name
	, ROUND(IFNULL(AVG(sc.s_score), 0), 2) AS avg_score
FROM `Student` s
	LEFT JOIN `Score` sc ON s.s_id = sc.s_id
GROUP BY s.s_id
HAVING avg_score < 60;

-- 结果
+--------+----------+-------------+
|   s_id | s_name   |   avg_score |
|--------+----------+-------------|
|     04 | 李云     |       33.33 |
|     06 | 吴兰     |       32.5  |
|     08 | 王菊     |        0    |
+--------+----------+-------------+

 
-- 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
SELECT s.s_id, s.s_name, COUNT(a.c_id) AS sum_course
	, IFNULL(SUM(a.s_score),0) AS sum_score
FROM `Student` s
	LEFT JOIN `Score` a ON s.s_id = a.s_id
GROUP BY s.s_id, s.s_name;

-- 结果
+--------+----------+--------------+-------------+
|   s_id | s_name   |   sun_course |   sum_score |
|--------+----------+--------------+-------------|
|     01 | 赵雷     |            3 |         269 |
|     02 | 钱电     |            3 |         210 |
|     03 | 孙风     |            3 |         240 |
|     04 | 李云     |            3 |         100 |
|     05 | 周梅     |            2 |         163 |
|     06 | 吴兰     |            2 |          65 |
|     07 | 郑竹     |            2 |         187 |
|     08 | 王菊     |            0 |           0 |
+--------+----------+--------------+-------------+

 
 
-- 6、查询"李"姓老师的数量 
SELECT COUNT(t.t_id)
FROM `Teacher` t
WHERE t.t_name LIKE '李%'; 
 -- 结果
 +-----------------+
|   COUNT(t.t_id) |
|-----------------|
|               1 |
+-----------------+

-- 7、查询学过"张三"老师授课的同学的信息 
SELECT a.*
FROM Student a
WHERE a.s_id IN (
	SELECT s_id
	FROM Score
	WHERE c_id IN (
		SELECT c_id
		FROM Course
		WHERE t_id = (
			SELECT t_id
			FROM Teacher
			WHERE t_name = '张三'
		)
	)
	GROUP BY s_id
);

-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     01 | 赵雷     | 1990-01-01 | 男      |
|     02 | 钱电     | 1990-12-21 | 男      |
|     03 | 孙风     | 1990-05-20 | 男      |
|     04 | 李云     | 1990-08-06 | 男      |
|     05 | 周梅     | 1991-12-01 | 女      |
|     07 | 郑竹     | 1989-07-01 | 女      |
+--------+----------+------------+---------+


-- 8、查询没学过"张三"老师授课的同学的信息 
SELECT a.*
FROM Student a
WHERE a.s_id NOT IN (
	SELECT s_id
	FROM Score
	WHERE c_id in (
		SELECT c_id
		FROM Course
		WHERE t_id = (
			SELECT t_id
			FROM Teacher
			WHERE t_name = '张三'
		)
	)
	GROUP BY s_id
);
-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     06 | 吴兰     | 1992-03-01 | 女      |
|     08 | 王菊     | 1990-01-20 | 女      |
+--------+----------+------------+---------+


-- 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
SELECT s.*
FROM `Student` s, `Score` a, `Score` b
WHERE s.s_id = a.s_id
	AND a.s_id = b.s_id
	AND a.c_id = '01'
	AND b.c_id = '02';
 
-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     01 | 赵雷     | 1990-01-01 | 男      |
|     02 | 钱电     | 1990-12-21 | 男      |
|     03 | 孙风     | 1990-05-20 | 男      |
|     04 | 李云     | 1990-08-06 | 男      |
|     05 | 周梅     | 1991-12-01 | 女      |
+--------+----------+------------+---------+

 
-- 10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
 
SELECT s.*
FROM `Student` s
WHERE s.s_id IN (
		SELECT s_id
		FROM `Score`
		WHERE c_id = '01'
	)
	AND s.s_id NOT IN (
		SELECT s_id
		FROM `Score`
		WHERE c_id = '02'
	);
-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     06 | 吴兰     | 1992-03-01 | 女      |
+--------+----------+------------+---------+


-- 11、查询没有学全所有课程的同学的信息 
SELECT s.*
FROM `Student` s, (
		SELECT COUNT(*) AS sum_cource
		FROM `Course`
	) sc
WHERE s_id IN (
	SELECT s_id
	FROM `Score`
	GROUP BY s_id
	HAVING COUNT(c_id) < sc.sum_cource
);
-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     05 | 周梅     | 1991-12-01 | 女      |
|     06 | 吴兰     | 1992-03-01 | 女      |
|     07 | 郑竹     | 1989-07-01 | 女      |
+--------+----------+------------+---------+



-- 12、查询至少有一门课与学号为"01"的同学所学相同的同学的信息 
SELECT *
FROM `Student`
WHERE s_id IN (
	SELECT DISTINCT a.s_id AS id
	FROM `Score` a
	WHERE a.c_id IN (
		SELECT b.c_id
		FROM `Score` b
		WHERE b.s_id = '01'
	)
);
-- 结果
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     01 | 赵雷     | 1990-01-01 | 男      |
|     02 | 钱电     | 1990-12-21 | 男      |
|     03 | 孙风     | 1990-05-20 | 男      |
|     04 | 李云     | 1990-08-06 | 男      |
|     05 | 周梅     | 1991-12-01 | 女      |
|     06 | 吴兰     | 1992-03-01 | 女      |
|     07 | 郑竹     | 1989-07-01 | 女      |
+--------+----------+------------+---------+

-- 13、查询和"01"号的同学学习的课程完全相同的其他同学的信息 
 
-- 把每个学生的选课id拼接后进行比较,注意拼接顺序 
SELECT *
FROM (
	SELECT t.s_id, GROUP_CONCAT(t.c_id) AS c_ids
	FROM (
		SELECT s_id, c_id
		FROM Score
	) t
	WHERE s_id != '01'
	GROUP BY t.s_id
    order by t.s_id
) co
WHERE co.c_ids = (
	SELECT GROUP_CONCAT(t.c_id) AS c_ids
	FROM (
		SELECT s_id, c_id
		FROM Score
	) t
	WHERE s_id = '01'
	GROUP BY t.s_id
    order by t.s_id
);

-- 结果
+--------+----------+
|   s_id | c_ids    |
|--------+----------|
|     02 | 01,02,03 |
|     03 | 01,02,03 |
|     04 | 01,02,03 |
+--------+----------+


    
-- 14、查询没学过"张三"老师讲授的任一门课程的学生姓名 
SELECT a.s_name
FROM Student a
WHERE a.s_id NOT IN (
	SELECT s_id
	FROM Score
	WHERE c_id = (
		SELECT c_id
		FROM Course
		WHERE t_id = (
			SELECT t_id
			FROM Teacher
			WHERE t_name = '张三'
		)
	)
	GROUP BY s_id
);
-- 结果
+----------+
| s_name   |
|----------|
| 吴兰     |
| 王菊     |
+----------+

 
-- 15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 
SELECT s.s_id, s.s_name, AVG(sc.s_score)
FROM `Student` s, `Score` sc
WHERE s.s_id = sc.s_id
	AND sc.s_score < 60
GROUP BY s.s_id
HAVING COUNT(sc.s_score) >= 2;

-- 结果
+--------+----------+-------------------+
|   s_id | s_name   |   avg(sc.s_score) |
|--------+----------+-------------------|
|     04 | 李云     |           33.3333 |
|     06 | 吴兰     |           32.5    |
+--------+----------+-------------------+

 
-- 16、检索"01"课程分数小于60，按分数降序排列的学生信息
SELECT s.*, a.s_score
FROM `Student` s
	JOIN `Score` a
	ON a.s_id = s.s_id
		AND a.c_id = '01'
		AND a.s_score < 60
ORDER BY a.s_score DESC;
-- 结果
+--------+----------+------------+---------+-----------+
|   s_id | s_name   | s_birth    | s_sex   |   s_score |
|--------+----------+------------+---------+-----------|
|     04 | 李云     | 1990-08-06 | 男      |        50 |
|     06 | 吴兰     | 1992-03-01 | 女      |        31 |
+--------+----------+------------+---------+-----------+


-- 17、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
-- 写死
SELECT a.s_id, a.s_name, IFNULL((
		SELECT s_score
		FROM Score
		WHERE a.s_id = Score.s_id
			AND c_id = 1
	), 0) AS 语文
	, IFNULL((
		SELECT s_score
		FROM Score
		WHERE a.s_id = Score.s_id
			AND c_id = 2
	), 0) AS 数学
	, IFNULL((
		SELECT s_score
		FROM Score
		WHERE a.s_id = Score.s_id
			AND c_id = 3
	), 0) AS 英语
	, IFNULL(round(AVG(s_score), 2), 0) AS avg_score
FROM Student a
	LEFT JOIN Score ON a.s_id = Score.s_id
GROUP BY a.s_id
ORDER BY avg_score DESC

-- 或者
select s_id,
    IFNULL(sum(case when c_id=01 then s_score else null end),0) as score_01,
    IFNULL(sum(case when c_id=02 then s_score else null end),0) as score_02,
    IFNULL(sum(case when c_id=03 then s_score else null end),0) as score_03,
    avg(s_score)
from Score group by s_id
order by avg(s_score) desc;

-- 结果
+--------+----------+--------+--------+--------+-------------+
|   s_id | s_name   |   语文 |   数学 |   英语 |   avg_score |
|--------+----------+--------+--------+--------+-------------|
|     07 | 郑竹     |      0 |     89 |     98 |       93.5  |
|     01 | 赵雷     |     80 |     90 |     99 |       89.67 |
|     05 | 周梅     |     76 |     87 |      0 |       81.5  |
|     03 | 孙风     |     80 |     80 |     80 |       80    |
|     02 | 钱电     |     70 |     60 |     80 |       70    |
|     04 | 李云     |     50 |     30 |     20 |       33.33 |
|     06 | 吴兰     |     31 |      0 |     34 |       32.5  |
|     08 | 王菊     |      0 |      0 |      0 |        0    |
+--------+----------+--------+--------+--------+-------------+

 
-- 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
-- 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
SELECT a.c_id AS "课程ID", a.c_name AS "课程name", MAX(b.s_score) AS "最高分"
	, MIN(b.s_score) AS "最低分"
	, ROUND(AVG(b.s_score), 2) AS "平均分"
	, CONCAT(FLOOR((
		SELECT COUNT(*)
		FROM Score c
		WHERE c.s_score >= 60
			AND b.c_id = c.c_id
	) / COUNT(*) * 100), '%') AS "及格率"
	, CONCAT(FLOOR((
		SELECT COUNT(*)
		FROM Score c
		WHERE c.s_score BETWEEN 70 AND 80
			AND b.c_id = c.c_id
	) / COUNT(*) * 100), '%') AS "中等率"
	, CONCAT(FLOOR((
		SELECT COUNT(*)
		FROM Score c
		WHERE c.s_score BETWEEN 80 AND 90
			AND b.c_id = c.c_id
	) / COUNT(*) * 100), '%') AS "优良率"
	, CONCAT(FLOOR((
		SELECT COUNT(*)
		FROM Score c
		WHERE c.s_score >= 90
			AND b.c_id = c.c_id
	) / COUNT(*) * 100), '%') AS "优秀率"
FROM Course a, Score b
WHERE a.c_id = b.c_id
GROUP BY b.c_id \G

-- 或者
select c.c_id as 课程号, c.c_name as 课程名称, count(*) as 选修人数,
    max(s_score) as 最高分, min(s_score) as 最低分, avg(s_score) as 平均分,
    sum(case when s_score >= 60 then 1 else 0 end)/count(*) as 及格率,
    sum(case when s_score >= 70 and s_score < 80 then 1 else 0 end)/count(*) as 中等率,
    sum(case when s_score >= 80 and s_score < 90 then 1 else 0 end)/count(*) as 优良率,
    sum(case when s_score >= 90 then 1 else 0 end)/count(*) as 优秀率
from Score sc, Course c
where c.c_id = sc.c_id
group by c.c_id
order by count(*) desc, c.c_id asc

-- 结果
***************************[ 1. row ]***************************
课程ID   | 01
课程name | 语文
最高分    | 80
最低分    | 31
平均分    | 64.50
及格率    | 66%
中等率    | 66%
优良率    | 33%
优秀率    | 0%
***************************[ 2. row ]***************************
课程ID   | 02
课程name | 数学
最高分    | 90
最低分    | 30
平均分    | 72.67
及格率    | 83%
中等率    | 16%
优良率    | 66%
优秀率    | 16%
***************************[ 3. row ]***************************
课程ID   | 03
课程name | 英语
最高分    | 99
最低分    | 20
平均分    | 68.50
及格率    | 66%
中等率    | 33%
优良率    | 33%
优秀率    | 33%
    
    
-- 19、按各科成绩进行排序，并显示排名(实现不完全)

SET @rn = 0;

SELECT a.*, @rn := @rn + 1
FROM (
	SELECT a.*, b.s_score
	FROM Course a
		INNER JOIN `Score` b ON a.c_id = b.c_id
	WHERE a.c_id = '01'
	ORDER BY b.c_id, b.s_score DESC
) a;

SELECT a.*
FROM (
	SELECT a.*, b.s_score,rank() over(partition by b.c_id order by b.s_score desc) as rk
	FROM Course a
		INNER JOIN `Score` b ON a.c_id = b.c_id
	WHERE a.c_id = '01'
	ORDER BY b.c_id, b.s_score DESC
) a;

-- 结果
+--------+----------+--------+-----------+--------------+
|   c_id | c_name   |   t_id |   s_score |   @rn:=@rn+1 |
|--------+----------+--------+-----------+--------------|
|     01 | 语文     |     02 |        80 |            1 |
|     01 | 语文     |     02 |        80 |            2 |
|     01 | 语文     |     02 |        76 |            3 |
|     01 | 语文     |     02 |        70 |            4 |
|     01 | 语文     |     02 |        50 |            5 |
|     01 | 语文     |     02 |        31 |            6 |
+--------+----------+--------+-----------+--------------+

-- 按平均成绩进行排序，显示总排名和各科排名，Score 重复时保留名次空缺
select s.*, rank_01, rank_02, rank_03, rank_total
from Student s
left join (select s_id, rank() over(partition by c_id order by s_score desc) as rank_01 from Score where c_id=01) A on s.s_id=A.s_id
left join (select s_id, rank() over(partition by c_id order by s_score desc) as rank_02 from Score where c_id=02) B on s.s_id=B.s_id
left join (select s_id, rank() over(partition by c_id order by s_score desc) as rank_03 from Score where c_id=03) C on s.s_id=C.s_id
left join (select s_id, rank() over(order by avg(s_score) desc) as rank_total from Score group by s_id) D on s.s_id=D.s_id
order by rank_total asc

-- 结果
+------+--------+---------------------+------+---------+---------+---------+------------+
| Sid  | Sname  | Sage                | Ssex | rank_01 | rank_02 | rank_03 | rank_total |
+------+--------+---------------------+------+---------+---------+---------+------------+
| 08   | 王菊   | 1990-01-20 00:00:00 | 女   |    NULL |    NULL |    NULL |       NULL |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |    NULL |       2 |       2 |          1 |
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |       1 |       1 |       1 |          2 |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |       3 |       3 |    NULL |          3 |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |       1 |       4 |       3 |          4 |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |       4 |       5 |       3 |          5 |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |       5 |       6 |       6 |          6 |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |       6 |    NULL |       5 |          7 |
+------+--------+---------------------+------+---------+---------+---------+------------+
8 rows in set (0.00 sec)

-- 按平均成绩进行排序，显示总排名和各科排名，Score 重复时合并名次
select s.*, rank_01, rank_02, rank_03, rank_total
from Student s
left join (select s_id, dense_rank() over(partition by c_id order by s_score desc) as rank_01 from Score where c_id=01) A on s.s_id=A.s_id
left join (select s_id, dense_rank() over(partition by c_id order by s_score desc) as rank_02 from Score where c_id=02) B on s.s_id=B.s_id
left join (select s_id, dense_rank() over(partition by c_id order by s_score desc) as rank_03 from Score where c_id=03) C on s.s_id=C.s_id
left join (select s_id, dense_rank() over(order by avg(s_score) desc) as rank_total from Score group by s_id) D on s.s_id=D.s_id
order by rank_total asc

+------+--------+---------------------+------+---------+---------+---------+------------+
| Sid  | Sname  | Sage                | Ssex | rank_01 | rank_02 | rank_03 | rank_total |
+------+--------+---------------------+------+---------+---------+---------+------------+
| 08   | 王菊   | 1990-01-20 00:00:00 | 女   |    NULL |    NULL |    NULL |       NULL |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |    NULL |       2 |       2 |          1 |
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |       1 |       1 |       1 |          2 |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |       2 |       3 |    NULL |          3 |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |       1 |       4 |       3 |          4 |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |       3 |       5 |       3 |          5 |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |       4 |       6 |       5 |          6 |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |       5 |    NULL |       4 |          7 |
+------+--------+---------------------+------+---------+---------+---------+------------+
8 rows in set (0.00 sec)


-- 20、查询学生的总成绩并进行排名
SET @rn = 0;
select
@rn := @rn + 1 ,a.*, IFNULL(sum(b.s_score),0) as sum_score
from
Student a
left join
Score b ON a.s_id = b.s_id
group by s_id
order by sum_score desc;



-- 结果
+------------------+--------+----------+------------+---------+-------------+
|   @rn := @rn + 1 |   s_id | s_name   | s_birth    | s_sex   |   sum_score |
|------------------+--------+----------+------------+---------+-------------|
|                1 |     01 | 赵雷     | 1990-01-01 | 男      |         269 |
|                3 |     03 | 孙风     | 1990-05-20 | 男      |         240 |
|                2 |     02 | 钱电     | 1990-12-21 | 男      |         210 |
|                7 |     07 | 郑竹     | 1989-07-01 | 女      |         187 |
|                5 |     05 | 周梅     | 1991-12-01 | 女      |         163 |
|                4 |     04 | 李云     | 1990-08-06 | 男      |         100 |
|                6 |     06 | 吴兰     | 1992-03-01 | 女      |          65 |
|                8 |     08 | 王菊     | 1990-01-20 | 女      |           0 |
+------------------+--------+----------+------------+---------+-------------+





-- 21、查询不同老师所教不同课程平均分从高到低显示 
SELECT t.*, c.c_name
	, ROUND(AVG(s.s_score), 2) AS avg_score
FROM `Teacher` t
	JOIN `Course` c ON t.t_id = c.t_id
	LEFT JOIN `Score` s ON s.c_id = c.c_id
GROUP BY c.c_id, c.t_id, t.t_name
ORDER BY avg_score DESC;
-- 结果
+--------+----------+----------+-------------+
|   t_id | t_name   | c_name   |   avg_score |
|--------+----------+----------+-------------|
|     01 | 张三     | 数学     |       72.67 |
|     03 | 王五     | 英语     |       68.5  |
|     02 | 李四     | 语文     |       64.5  |
+--------+----------+----------+-------------+


-- 22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩
 -- 需要改进
SELECT d.*, c.排名, c.s_score, c.c_id
FROM (
	SELECT a.s_id, a.s_score, a.c_id
		, @i := @i + 1 AS 排名
	FROM Score a, (
			SELECT @i := 0
		) s
	WHERE a.c_id = '01'
) c
	LEFT JOIN Student d ON c.s_id = d.s_id
WHERE 排名 BETWEEN 2 AND 3
UNION
SELECT d.*, c.排名, c.s_score, c.c_id
FROM (
	SELECT a.s_id, a.s_score, a.c_id
		, @j := @j + 1 AS 排名
	FROM Score a, (
			SELECT @j := 0
		) s
	WHERE a.c_id = '02'
) c
	LEFT JOIN Student d ON c.s_id = d.s_id
WHERE 排名 BETWEEN 2 AND 3
UNION
SELECT d.*, c.排名, c.s_score, c.c_id
FROM (
	SELECT a.s_id, a.s_score, a.c_id
		, @k := @k + 1 AS 排名
	FROM Score a, (
			SELECT @k := 0
		) s
	WHERE a.c_id = '03'
) c
	LEFT JOIN Student d ON c.s_id = d.s_id
WHERE 排名 BETWEEN 2 AND 3;
 
 -- 结果
 +--------+----------+------------+---------+--------+-----------+--------+
|   s_id | s_name   | s_birth    | s_sex   |   排名 |   s_score |   c_id |
|--------+----------+------------+---------+--------+-----------+--------|
|     02 | 钱电     | 1990-12-21 | 男      |      2 |        70 |     01 |
|     03 | 孙风     | 1990-05-20 | 男      |      3 |        80 |     01 |
|     02 | 钱电     | 1990-12-21 | 男      |      2 |        60 |     02 |
|     03 | 孙风     | 1990-05-20 | 男      |      3 |        80 |     02 |
|     02 | 钱电     | 1990-12-21 | 男      |      2 |        80 |     03 |
|     03 | 孙风     | 1990-05-20 | 男      |      3 |        80 |     03 |
+--------+----------+------------+---------+--------+-----------+--------+

 
-- 23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比
 SELECT DISTINCT f.c_name, a.c_id, b.`85-100`, b.百分比, c.`70-85`
	, c.百分比, d.`60-70`, d.百分比, e.`0-60`, e.百分比
FROM Score a
	LEFT JOIN (
		SELECT c_id, SUM(CASE 
				WHEN s_score > 85
				AND s_score <= 100 THEN 1
				ELSE 0
			END) AS `85-100`
			, ROUND(100 * (SUM(CASE 
				WHEN s_score > 85
				AND s_score <= 100 THEN 1
				ELSE 0
			END) / COUNT(*)), 2) AS 百分比
		FROM Score
		GROUP BY c_id
	) b
	ON a.c_id = b.c_id
	LEFT JOIN (
		SELECT c_id, SUM(CASE 
				WHEN s_score > 70
				AND s_score <= 85 THEN 1
				ELSE 0
			END) AS `70-85`
			, ROUND(100 * (SUM(CASE 
				WHEN s_score > 70
				AND s_score <= 85 THEN 1
				ELSE 0
			END) / COUNT(*)), 2) AS 百分比
		FROM Score
		GROUP BY c_id
	) c
	ON a.c_id = c.c_id
	LEFT JOIN (
		SELECT c_id, SUM(CASE 
				WHEN s_score > 60
				AND s_score <= 70 THEN 1
				ELSE 0
			END) AS `60-70`
			, ROUND(100 * (SUM(CASE 
				WHEN s_score > 60
				AND s_score <= 70 THEN 1
				ELSE 0
			END) / COUNT(*)), 2) AS 百分比
		FROM Score
		GROUP BY c_id
	) d
	ON a.c_id = d.c_id
	LEFT JOIN (
		SELECT c_id, SUM(CASE 
				WHEN s_score >= 0
				AND s_score <= 60 THEN 1
				ELSE 0
			END) AS `0-60`
			, ROUND(100 * (SUM(CASE 
				WHEN s_score >= 0
				AND s_score <= 60 THEN 1
				ELSE 0
			END) / COUNT(*)), 2) AS 百分比
		FROM Score
		GROUP BY c_id
	) e
	ON a.c_id = e.c_id
	LEFT JOIN Course f ON a.c_id = f.c_id
 
 +----------+--------+----------+----------+---------+----------+---------+----------+--------+----------+
| c_name   |   c_id |   85-100 |   百分比 |   70-85 |   百分比 |   60-70 |   百分比 |   0-60 |   百分比 |
|----------+--------+----------+----------+---------+----------+---------+----------+--------+----------|
| 语文     |     01 |        0 |     0    |       3 |    50    |       1 |    16.67 |      2 |    33.33 |
| 数学     |     02 |        3 |    50    |       1 |    16.67 |       0 |     0    |      2 |    33.33 |
| 英语     |     03 |        2 |    33.33 |       2 |    33.33 |       0 |     0    |      2 |    33.33 |
+----------+--------+----------+----------+---------+----------+---------+----------+--------+----------+


-- 24、查询学生平均成绩及其名次 
SET @i = 0;
SET @k =0;

SELECT a.s_id, @i := @i + 1 AS '不保留空缺排名'
	, @k := CASE 
		WHEN @avg_score = a.avg_s THEN @k
		ELSE @i
	END AS '保留空缺排名', @avg_score := avg_s AS '平均分'
FROM (
	SELECT s_id, ROUND(AVG(s_score), 2) AS avg_s
	FROM Score
	GROUP BY s_id
) a, (
		SELECT @avg_score := 0, @i := 0
			, @k := 0
	) b;
	
	
SELECT RANK() OVER (ORDER BY a.avg_score) AS rk, a.*
FROM (
	SELECT sc.s_id, AVG(sc.s_score) AS avg_score
	FROM `Score` sc
	GROUP BY sc.s_id
	ORDER BY avg_score DESC
) a;

+------+--------+-------------+
|   rk |   s_id |   avg_score |
|------+--------+-------------|
|    1 |     06 |     32.5    |
|    2 |     04 |     33.3333 |
|    3 |     02 |     70      |
|    4 |     03 |     80      |
|    5 |     05 |     81.5    |
|    6 |     01 |     89.6667 |
|    7 |     07 |     93.5    |
+------+--------+-------------+

	
-- 25、查询各科成绩前三名的记录

select * from (select *, DENSE_RANK() over(partition by c_id order by s_score desc) as graderank from Score) A 
where A.graderank <= 3


+--------+--------+-----------+-------------+
|   s_id |   c_id |   s_score |   graderank |
|--------+--------+-----------+-------------|
|     01 |     01 |        80 |           1 |
|     03 |     01 |        80 |           1 |
|     05 |     01 |        76 |           2 |
|     02 |     01 |        70 |           3 |
|     01 |     02 |        90 |           1 |
|     07 |     02 |        89 |           2 |
|     05 |     02 |        87 |           3 |
|     01 |     03 |        99 |           1 |
|     07 |     03 |        98 |           2 |
|     02 |     03 |        80 |           3 |
|     03 |     03 |        80 |           3 |
+--------+--------+-----------+-------------+
 
-- 26、查询每门课程被选修的学生数 
 
SELECT sc.c_id, COUNT(*)
FROM `Score` sc
GROUP BY sc.c_id;        
 +--------+------------+
|   c_id |   COUNT(*) |
|--------+------------|
|     01 |          6 |
|     02 |          6 |
|     03 |          6 |
+--------+------------+

 
-- 27、查询出只有两门课程的全部学生的学号和姓名 
  SELECT s.*
FROM `Student` s
WHERE s.s_id IN (
	SELECT sc.s_id
	FROM `Score` sc
	GROUP BY sc.s_id
	HAVING COUNT(*) = 2
);
+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     05 | 周梅     | 1991-12-01 | 女      |
|     06 | 吴兰     | 1992-03-01 | 女      |
|     07 | 郑竹     | 1989-07-01 | 女      |
+--------+----------+------------+---------+

 
-- 28、查询男生、女生人数 

SELECT s.s_sex, COUNT(*)
FROM `Student` s
GROUP BY s.s_sex;

+---------+------------+
| s_sex   |   COUNT(*) |
|---------+------------|
| 男      |          4 |
| 女      |          4 |
+---------+------------+


-- 29、查询名字中含有"风"字的学生信息
 
select * from Student where s_name like '%风%';

+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     03 | 孙风     | 1990-05-20 | 男      |
+--------+----------+------------+---------+
 
-- 30、查询同名同性学生名单，并统计同名人数 
 
SELECT *
FROM `Student` s
GROUP BY s.s_name, s.s_sex
HAVING COUNT(*) > 1;
 +--------+----------+-----------+---------+
| s_id   | s_name   | s_birth   | s_sex   |
|--------+----------+-----------+---------|
+--------+----------+-----------+---------+

 
 
-- 31、查询1990年出生的学生名单
select s_name from Student where s_birth like '1990%';
+----------+
| s_name   |
|----------|
| 赵雷     |
| 钱电     |
| 孙风     |
| 李云     |
| 王菊     |
+----------+

 
-- 32、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列 
SELECT sc.c_id, ROUND(AVG(sc.s_score), 2) AS avg_score
FROM `Score` sc
GROUP BY sc.c_id
ORDER BY avg_score DESC, sc.c_id ASC;
+--------+-------------+
|   c_id |   avg_score |
|--------+-------------|
|     02 |       72.67 |
|     03 |       68.5  |
|     01 |       64.5  |
+--------+-------------+
 
-- 33、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩 
SELECT s.*, a.avg_score
FROM `Student` s, (
		SELECT sc.s_id, ROUND(AVG(sc.s_score), 2) AS avg_score
		FROM `Score` sc
		GROUP BY sc.s_id
	) a
WHERE s.s_id = a.s_id;

+--------+----------+------------+---------+-------------+
|   s_id | s_name   | s_birth    | s_sex   |   avg_score |
|--------+----------+------------+---------+-------------|
|     01 | 赵雷     | 1990-01-01 | 男      |       89.67 |
|     02 | 钱电     | 1990-12-21 | 男      |       70    |
|     03 | 孙风     | 1990-05-20 | 男      |       80    |
|     04 | 李云     | 1990-08-06 | 男      |       33.33 |
|     05 | 周梅     | 1991-12-01 | 女      |       81.5  |
|     06 | 吴兰     | 1992-03-01 | 女      |       32.5  |
|     07 | 郑竹     | 1989-07-01 | 女      |       93.5  |
+--------+----------+------------+---------+-------------+

 
-- 34、查询课程名称为"数学"，且分数低于60的学生姓名和分数 
 
SELECT a.s_name, b.s_score
FROM Score b
	LEFT JOIN Student a ON a.s_id = b.s_id
WHERE b.c_id = (
		SELECT c_id
		FROM Course
		WHERE c_name = '数学'
	)
	AND b.s_score < 60;
	
+----------+-----------+
| s_name   |   s_score |
|----------+-----------|
| 李云     |        30 |
+----------+-----------+
   
-- 35、查询所有学生的课程及分数情况； 
 
 
SELECT a.s_id, a.s_name, SUM(CASE c.c_name
		WHEN '语文' THEN b.s_score
		ELSE 0
	END) AS '语文'
	, SUM(CASE c.c_name
		WHEN '数学' THEN b.s_score
		ELSE 0
	END) AS '数学', SUM(CASE c.c_name
		WHEN '英语' THEN b.s_score
		ELSE 0
	END) AS '英语'
	, SUM(b.s_score) AS '总分'
FROM Student a
	LEFT JOIN Score b ON a.s_id = b.s_id
	LEFT JOIN Course c ON b.c_id = c.c_id
GROUP BY a.s_id, a.s_name

 +--------+----------+--------+--------+--------+--------+
|   s_id | s_name   |   语文 |   数学 |   英语 |   总分 |
|--------+----------+--------+--------+--------+--------|
|     01 | 赵雷     |     80 |     90 |     99 |    269 |
|     02 | 钱电     |     70 |     60 |     80 |    210 |
|     03 | 孙风     |     80 |     80 |     80 |    240 |
|     04 | 李云     |     50 |     30 |     20 |    100 |
|     05 | 周梅     |     76 |     87 |      0 |    163 |
|     06 | 吴兰     |     31 |      0 |     34 |     65 |
|     07 | 郑竹     |      0 |     89 |     98 |    187 |
|     08 | 王菊     |      0 |      0 |      0 | <null> |
+--------+----------+--------+--------+--------+--------+

 
 -- 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数； 
SELECT s.s_name, c.c_name, a.s_score
FROM `Student` s
	JOIN `Score` a ON s.s_id = a.s_id
	JOIN `Course` c ON a.c_id = c.c_id
WHERE s.s_id IN (
	SELECT DISTINCT sc.s_id
	FROM `Score` sc
	WHERE sc.s_score > 70
);

 +----------+----------+-----------+
| s_name   | c_name   |   s_score |
|----------+----------+-----------|
| 周梅     | 语文     |        76 |
| 孙风     | 语文     |        80 |
| 钱电     | 语文     |        70 |
| 赵雷     | 语文     |        80 |
| 郑竹     | 数学     |        89 |
| 周梅     | 数学     |        87 |
| 孙风     | 数学     |        80 |
| 钱电     | 数学     |        60 |
| 赵雷     | 数学     |        90 |
| 郑竹     | 英语     |        98 |
| 孙风     | 英语     |        80 |
| 钱电     | 英语     |        80 |
| 赵雷     | 英语     |        99 |
+----------+----------+-----------+

 
 
-- 37、查询不及格的课程
SELECT a.s_id, a.c_id, b.c_name, a.s_score
FROM Score a
	LEFT JOIN Course b ON a.c_id = b.c_id
WHERE a.s_score < 60
+--------+--------+----------+-----------+
|   s_id |   c_id | c_name   |   s_score |
|--------+--------+----------+-----------|
|     04 |     01 | 语文     |        50 |
|     06 |     01 | 语文     |        31 |
|     04 |     02 | 数学     |        30 |
|     04 |     03 | 英语     |        20 |
|     06 |     03 | 英语     |        34 |
+--------+--------+----------+-----------+
 
--38、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名； 
SELECT a.s_id, b.s_name
FROM Score a
	LEFT JOIN Student b ON a.s_id = b.s_id
WHERE a.c_id = '01'
	AND a.s_score > 80
 
-- 39、求每门课程的学生人数 
SELECT COUNT(*)
FROM Score
GROUP BY c_id;

+------------+
|   count(*) |
|------------|
|          6 |
|          6 |
|          6 |
+------------+


-- 40、查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩
 
        -- 查询老师id   
SELECT c_id
FROM Course c, Teacher d
WHERE c.t_id = d.t_id
	AND d.t_name = '张三'
        -- 查询最高分（可能有相同分数）
SELECT MAX(s_score)
FROM Score
WHERE c_id = '02'
        -- 查询信息
SELECT a.*, b.s_score, b.c_id, c.c_name
FROM Student a
	LEFT JOIN Score b ON a.s_id = b.s_id
	LEFT JOIN Course c ON b.c_id = c.c_id
WHERE b.c_id = (
		SELECT c_id
		FROM Course c, Teacher d
		WHERE c.t_id = d.t_id
			AND d.t_name = '张三'
	)
	AND b.s_score IN (
		SELECT MAX(s_score)
		FROM Score
		WHERE c_id = '02'
	)
+--------+----------+------------+---------+-----------+--------+----------+
|   s_id | s_name   | s_birth    | s_sex   |   s_score |   c_id | c_name   |
|--------+----------+------------+---------+-----------+--------+----------|
|     01 | 赵雷     | 1990-01-01 | 男      |        90 |     02 | 数学     |
+--------+----------+------------+---------+-----------+--------+----------+
      
-- 41、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 
SELECT DISTINCT b.s_id, b.c_id, b.s_score
FROM Score a, Score b
WHERE a.c_id != b.c_id
	AND a.s_score = b.s_score
	
+--------+--------+-----------+
|   s_id |   c_id |   s_score |
|--------+--------+-----------|
|     01 |     01 |        80 |
|     02 |     03 |        80 |
|     03 |     01 |        80 |
|     03 |     02 |        80 |
|     03 |     03 |        80 |
+--------+--------+-----------+

-- 42、查询每门功成绩最好的前两名 
        -- 牛逼的写法
SELECT a.s_id, a.c_id, a.s_score
FROM Score a
WHERE (
	SELECT COUNT(1)
	FROM Score b
	WHERE b.c_id = a.c_id
		AND b.s_score >= a.s_score
) <= 2
ORDER BY a.c_id
+--------+--------+-----------+
|   s_id |   c_id |   s_score |
|--------+--------+-----------|
|     01 |     01 |        80 |
|     03 |     01 |        80 |
|     01 |     02 |        90 |
|     07 |     02 |        89 |
|     01 |     03 |        99 |
|     07 |     03 |        98 |
+--------+--------+-----------+


-- 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列  
SELECT c_id, COUNT(*) AS total
FROM Score
GROUP BY c_id
HAVING total > 5
ORDER BY total, c_id ASC

+--------+---------+
|   c_id |   total |
|--------+---------|
|     01 |       6 |
|     02 |       6 |
|     03 |       6 |
+--------+---------+


-- 44、检索至少选修两门课程的学生学号 
SELECT s_id, COUNT(*) AS sel
FROM Score
GROUP BY s_id
HAVING sel >= 2

+--------+-------+
|   s_id |   sel |
|--------+-------|
|     01 |     3 |
|     02 |     3 |
|     03 |     3 |
|     04 |     3 |
|     05 |     2 |
|     06 |     2 |
|     07 |     2 |
+--------+-------+


-- 45、查询选修了全部课程的学生信息 
SELECT *
FROM Student
WHERE s_id IN (
	SELECT s_id
	FROM Score
	GROUP BY s_id
	HAVING COUNT(*) = (
		SELECT COUNT(*)
		FROM Course
	)
)

+--------+----------+------------+---------+
|   s_id | s_name   | s_birth    | s_sex   |
|--------+----------+------------+---------|
|     01 | 赵雷     | 1990-01-01 | 男      |
|     02 | 钱电     | 1990-12-21 | 男      |
|     03 | 孙风     | 1990-05-20 | 男      |
|     04 | 李云     | 1990-08-06 | 男      |
+--------+----------+------------+---------+

-- 46、查询各学生的年龄
    -- 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
SELECT s_birth
	, DATE_FORMAT(NOW(), '%Y') - DATE_FORMAT(s_birth, '%Y') - CASE 
		WHEN DATE_FORMAT(NOW(), '%m%d') > DATE_FORMAT(s_birth, '%m%d') THEN 0
		ELSE 1
	END AS age
FROM Student;
 
 +------------+-------+
| s_birth    |   age |
|------------+-------|
| 1990-01-01 |    29 |
| 1990-12-21 |    28 |
| 1990-05-20 |    29 |
| 1990-08-06 |    29 |
| 1991-12-01 |    27 |
| 1992-03-01 |    27 |
| 1989-07-01 |    30 |
| 1990-01-20 |    29 |
+------------+-------+

 
-- 47、查询本周过生日的学生
SELECT *
FROM Student
WHERE WEEK(DATE_FORMAT(NOW(), '%Y%m%d')) = WEEK(s_birth)

SELECT *
FROM Student
WHERE YEARWEEK(s_birth) = YEARWEEK(DATE_FORMAT(NOW(), '%Y%m%d'))

SELECT WEEK(DATE_FORMAT(NOW(), '%Y%m%d'))

-- 48、查询下周过生日的学生
SELECT *
FROM Student
WHERE WEEK(DATE_FORMAT(NOW(), '%Y%m%d')) + 1 = WEEK(s_birth)
 
-- 49、查询本月过生日的学生
SELECT *
FROM Student
WHERE MONTH(DATE_FORMAT(NOW(), '%Y%m%d')) = MONTH(s_birth) 
-- 50、查询下月过生日的学生
SELECT *
FROM Student
WHERE MONTH(DATE_FORMAT(NOW(), '%Y%m%d')) + 1 = MONTH(s_birth)
```

#### 遇到问题

**错误1**

(1055, "Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.s.s_id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by")

修改sql_mode，如果想去除sql_mode中某个选项，可用如下语句：

```
set sql_mode = (select replace(@@sql_mode, 'only_full_group_by', ''));
```





### 参考：

1. [MySQL的sql_mode解析与设置](https://baijiahao.baidu.com/s?id=1637179252298020747&wfr=spider&for=pc)

