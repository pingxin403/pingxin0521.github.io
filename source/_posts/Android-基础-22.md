---
title: Android  常见错误
date: 2019-06-03 18:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

#### android.database.sqlite.SQLiteException: no such table出现原因与解决办法

<!--more-->

最近在编写SQLite，新增一个表，写完逻辑，感觉自己萌萌哒~~~

一运行，魅族式闪退，卧槽。。。

一看：android.database.sqlite.SQLiteException: no such table（或者是no column named）

分析：

1. 语法错了？表名写错了？

     仔细研究，发现这些都没有问题。

2. 难道是没有重新编译？

     于是吓得我赶紧把debug上去的apk删除了，再重新debug一次进去。
     
3. 创建表出错

     执行db.execSQL一次只能运行一条SQL语句，因此如果有多张表，需要多次运行分开创建。

#### android.content.res.Resources$NotFoundException的解决方案

错误原因是：**由于我的数据获取到的是int类型的，而我却用setText方法传给了TextView，将类型转换为String就好了。**

错误代码：

```
tv02.setText(obj.YZMC);1
```

正确代码：

```
tv02.setText("" + obj.YZMC);1
```

记录分享一下，希望能对同样遇到这个问题的朋友们一些帮助~