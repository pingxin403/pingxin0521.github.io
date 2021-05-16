---
title: MongoDB入门
date: 2019-05-17 13:18:59
tags:
 - NoSQL
 - MongoDB
 - 数据库
categories:
 - 数据库
 - MongoDB
---

### 常见命令

安装完成后，shell交互式下输入mongo就可以直接无密码登录到数据库。

<!--more-->

#### 数据库连接

1. 开启远程访问：修改配置文件 `/etc/mongod.conf` 中 `bindIp: 127.0.0.1` 为 `bindIp: 0.0.0.0`             
2. 连接数据库：`mongo 数据库地址:端口/数据库名 -u用户名 -p密码`

#### 数据导入导出

1. 导入数据库文件到集合：`mongoimport -d 数据库名 -c 集合名 数据库文件`   
2. 导出数据库到文件：`mongoexport -d 数据库名 -c 集合名 -o 文件名 --type 类型(默认json) -f 字段名(仅csv)`

#### 数据库

在 MongoDB 中，默认的数据库是 test，如果你没有创建任何数据库，那么集合就会保存在 test 数据库中。

1. 查看数据库列表，没有数据则不显示：`show dbs`       

2. 查看当前所在数据库：`db` 或 `db.getName()`         

3. 查看当前数据库状态：`db.stats()`          

4. 没有则创建数据库/切换数据库：`use 数据库名`       

5. 删除当前数据库：`db.dropDatebase()`             

6. 修复当前数据库：`db.repairDatabase()`         

7. 查看当前用户：`show users`

**示例**

```shell
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
> use px
switched to db px
> db
px
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
> db.pingxin.insert({"name":"平心"});
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
px      0.000GB
test    0.000GB
> db.dropDatabase()
{ "dropped" : "px", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
> db
px
> show tables
> db.pingxin.insert({"name":"平心"});
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
px      0.000GB
test    0.000GB
> show tables
pingxin
> db.pingxin.drop();
true
> show tables
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```

> **注意:** 在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。

#### 集合

1. 查看当前数据库集合：`db.getCollectionNames()` 或 `show collections`
2. 创建集合：`db.createCollection("集合名", 选项)`
3. 删除集合：`db.集合名.drop()`           
4. 统计集合数据条数：`db.集合名.count`              
5. 插入数据：`db.集合名.insert({"字段1":"值", "字段2":"值"})`            
6. `for(var i=1; i<=100; i++) db.集合名.insert({"字段1":i,"字段2":"值"})`  *//循环插入100条数据*                   
7.  删除数据：`db.集合名.remove({"字段1":"值", "字段2":"值"})`       *//不加参数为清空当前集合*            
8. 更新数据：`db.集合名.update({"字段1":"值", "字段2":"值"})` 或 `db.集合名.save({"字段1":"值", "字段2":"值"})`           
9. 查询数据：`db.集合名.find()`            *//所有数据*       

10. `db.集合名.find({"字段1": {$gte:10,$lte:30}})`         *//区间10-30(小于lt，大于gt，小于等于lte，大于等于gte)*                  
11. `db.集合名.find({"字段1":"值", "字段2":"值"})`            *//指定数据*           
12. `db.集合名.find({"字段1":"值", "字段2":"值"})`            *//指定数据*            
13. `db.集合名.find().sort({"字段1":1})`                             *//数据按字段升序排列*      
14. `db.集合名.find().sort({"字段1":-1})`                           *//数据按字段降序排列*      
15. `db.集合名.find().limit(30)`                                            *//指定查询数据条数*      
16. `db.集合名.find().skip(2)`                                                *//指定从第n条开始查询*      
17. `db.集合名.find({},{"字段1":1,"字段2":1})`                                                *//指定显示字段(1为显示，0为不显示)*

##### db.createCollection(name, options)

参数说明：

- name: 要创建的集合名称
- options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段   | 类型 | 描述                                                         |
| ------ | ---- | ------------------------------------------------------------ |
| capped | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| size   | 数值 | （可选）为固定集合指定一个最大值（以字节计）。 **如果 capped 为 true，也需要指定该字段。** |
| max    | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。 

在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。

示例：创建一个集合"user"，为字段_id创建索引，最大存储空间是10M，最大文档数量为1000

```
> db.createCollection("user", { capped : true,size : 10485760, max : 1000 } )
```

在 MongoDB 中，可以不用createCollection()方法创建集合，是因为**在插入文档的时候，会自动创建集合**

**示例**

```shell
>db.createCollection("user", { capped : true,size : 10485760, max : 1000 } )
> db.pingxin.insert({"name":"平心"});
WriteResult({ "nInserted" : 1 })
> show tables
pingxin
user
> show collections
pingxin
user
> db.user.drop();
true
> show tables
pingxin
> db.pingxin.find()
{ "_id" : ObjectId("5e2c30647e6edeb89a924ed1"), "name" : "平心" }
> document=({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '平心`博客',
    url: 'hanyunpeng0521.github.io',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
});
> db.col.insert(document);
WriteResult({ "nInserted" : 1 })

```

插入文档你也可以使用 db.col.save(document) 命令。如果不指定 _id 字段 save() 方法类似于 insert() 方法。如果指定 _id 字段，则会更新该 _id 的数据。

3.2 版本后还有以下几种语法可用于插入文档:

- db.collection.insertOne():向指定集合中插入一条文档数据

-  db.collection.insertMany():向指定集合中插入多条文档数据

  ```shell
  #  插入单条数据
  
  > var document = db.collection.insertOne({"a": 3})
  > document
  {
          "acknowledged" : true,
          "insertedId" : ObjectId("571a218011a82a1d94c02333")
  }
  
  #  插入多条数据
  > var res = db.collection.insertMany([{"b": 3}, {'c': 4}])
  > res
  {
          "acknowledged" : true,
          "insertedIds" : [
                  ObjectId("571a22a911a82a1d94c02337"),
                  ObjectId("571a22a911a82a1d94c02338")
          ]
  }
  ```

##### update() 方法

update() 方法用于更新已存在的文档。语法格式如下：

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

**实例**

我们在集合 col 中插入如下数据：

```shell
> document=({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '平心`博客',
    url: 'hanyunpeng0521.github.io',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
});
> db.col.insert(document);
> db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.col.find().pretty()
{
	"_id" : ObjectId("5e2c31417e6edeb89a924ed2"),
	"title" : "MongoDB",
	"description" : "MongoDB 是一个 Nosql 数据库",
	"by" : "平心`博客",
	"url" : "hanyunpeng0521.github.io",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 100
}
> 

```

可以看到标题(title)由原来的 "MongoDB 教程" 更新为了 "MongoDB"。

以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true。

```shell
>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

##### save() 方法

save() 方法通过传入的文档来替换已有文档。语法格式如下：

```
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

**参数说明：**

- **document** : 文档数据。
- **writeConcern** :可选，抛出异常的级别。

**实例**

```shell
> db.col.save({ "_id" : ObjectId("5e2c31417e6edeb89a924ed2"), "title" : "MongoDB数据库", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "平心博客", "url" : "hanyunpeng0521.github.io", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

**更多实例**

- 只更新第一条记录：

  `db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );`

- 全部更新：

  `db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );`

- 只添加第一条：

  `db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );`

- 全部添加进去:

  `db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );`

- 全部更新：

  `db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );`

- 只更新第一条记录：

  `db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );`

##### remove() 

remove() 方法的基本语法格式如下所示：

```
db.collection.remove(
   <query>,
   <justOne>
)
```

如果你的 MongoDB 是 2.6 版本以后的，语法格式如下：

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。

remove() 方法已经过时了，现在官方推荐使用 deleteOne() 和 deleteMany() 方法。

如删除集合下全部文档：

```
db.inventory.deleteMany({})
```

删除 status 等于 A 的全部文档：

```
db.inventory.deleteMany({ status : "A" })
```

删除 status 等于 D 的一个文档：

```
db.inventory.deleteOne( { status: "D" } )
```

### 文档

##### 创建

MongoDB 用 insert()或者save()向集合中插入文档

```
db.collection.insert(document)
```

示例：

```
> db.user.find()
> db.user.insert({"name":"user1","age":19})
WriteResult({ "nInserted" : 1 })
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user1", "age" : 19 }
```

插入文档也可以使用 db.collection.save(document) 命令。如果不指定 _id 字段 save() 方法类似于 insert() 方法。如果指定 _id 字段，则会更新该 _id 的数据。save() 方法会在下面 更新文档 里面用范例说明。

MongoDB 用 update() 或者 save() 更新集合中的文档

##### 更新

update() 更新已经存在文档的值

**格式**

```
`db.COLLECTION_NAME.``update``(SELECTIOIN_CRITERIA, UPDATED_DATA)`
```

示例：

```
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user1", "age" : 19 }
> db.user.update({'name':'user1'},{$set:{'name':'user2'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user2", "age" : 19 }
```

save() 方法通过传入的文档来替换已有文档。

**格式**

```
`db.COLLECTION_NAME.save({_id:ObjectId(),NEW_DATA})`
```

##### 删除

MongoDB 用 remove() 删除集合中的文档

**格式**

```
`db.COLLECTION_NAME.remove(DELLETION_CRITTERIA,justOne)`
```

justOne 如果设为 true 或 1，则只删除一个文档。

#### 查询

查询文档可以用 find() 方法查询全部文档，可以用 findOne() 查询第一个文档，当然还可以根据 条件操作符 和 $type操作符 查询满足条件的文档。

MongoDB 用 find() 查询指定集合的全部文档

格式

```
`db.COLLECTION_NAME.find()`
```

如果想要格式化显示查询结果，我们需要用 pretty() 方法

格式如下

```
`db.COLLECTION_NAME.find().pretty()`
```

除了 find() 方法，还有一个 findOne() 方法，它只会返回一个文档

**MongoDB 与 RDBMS Where 语句比较**

| 操作       | 格式         | 范例                                        | RDBMS中的类似语句       |
| :--------- | :----------- | :------------------------------------------ | :---------------------- |
| 等于       | `{:`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{:{$lt:}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`      |
| 小于或等于 | `{:{$lte:}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`     |
| 大于       | `{:{$gt:}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`      |
| 大于或等于 | `{:{$gte:}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`     |
| 不等于     | `{:{$ne:}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50`     |

##### 条件操作符

如果你熟悉sql，那么下表可以更好的理解 MongoDB 条件查询

| 操作       | 格式                     | 范例                                          | RDBMS中的类似语句          |
| ---------- | ------------------------ | --------------------------------------------- | -------------------------- |
| 等于       | `{<key>:<value>`}        | `db.user.find({"name":"liruihuan"}).pretty()` | `where name = 'liruihuan'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.user.find({"age":{$lt:18}}).pretty()`     | `where age < 18`           |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.user.find({"age":{$lte:18}}).pretty()`    | `where age <= 18`          |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.user.find({"age":{$gt:18}}).pretty()`     | `where age > 18`           |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.user.find({"age":{$gte:18}}).pretty()`    | `where age >= 18`          |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.user.find({"age":{$ne:18}}).pretty()`     | `where age != 18`          |

为了更好的理解条件操作符，可以用英文解释一下

```
$gt -- greater than
 
$gte -- gt equal
 
$lt -- less than
 
$lte -- lt equal
 
$ne -- not equal
```

**MongoDB 中的 and 条件**

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，MongoDB 会把这些键作为 and 条件，及常规 SQL 的 AND 条件。

格式

```
`db.collection.find({key1:value1, key2:value2}).pretty()`
```

**MongoDB 中的 or 条件**

MongoDB 中 or 条件用 $or关键字

格式

```
>db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

**MongoDB 中 and 和 or 结合使用**

查询 user 集合中 age 值大于17 且 （name 值为 hyp 或 name 值为 user1） 的文档

```
 db.user.find({"age":{$gt:17},$or:[{"name":"liruihuan"},{"name":"user1"}]}).pretty()
{
       "_id" : ObjectId("58e1d2f0bb1bbc3245fa754b"),
       "name" : "hyp",
       "age" : 18,
       "sex" : "man"
}
{
       "_id" : ObjectId("58e1d2f0bb1bbc3245fa754d"),
       "name" : "user1",
       "age" : 19,
       "sex" : "man"
}
```

此实例类似于 sql 中 where 条件

```
WHERE age > 17  (name='hyp' OR name="user1")
```

##### $type 操作符

基于BSON类型来查询集合中匹配的数据类型，并返回结果。

下表列举了可使用的数据类型

|        **类型**         | **数字** | **备注**         |
| :---------------------: | -------- | ---------------- |
|         Double          | 1        |                  |
|         String          | 2        |                  |
|         Object          | 3        |                  |
|          Array          | 4        |                  |
|       Binary data       | 5        |                  |
|        Undefined        | 6        | 已废弃。         |
|        Object id        | 7        |                  |
|         Boolean         | 8        |                  |
|          Date           | 9        |                  |
|          Null           | 10       |                  |
|   Regular Expression    | 11       |                  |
|       JavaScript        | 13       |                  |
|         Symbol          | 14       |                  |
| JavaScript (with scope) | 15       |                  |
|     32-bit integer      | 16       |                  |
|        Timestamp        | 17       |                  |
|     64-bit integer      | 18       |                  |
|         Min key         | 255      | Query with `-1`. |
|         Max key         | 127      |                  |

如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：

```
db.col.find({"title" : {$type : 2}})
或
db.col.find({"title" : {$type : 'string'}})
```

#### 映射与限制记录

find()查询的结果都是显示了集合中全部的字段，实际应用中，显然是不够用的。那么有没有办法指定特定的字段显示出文档呢？答案是肯定的，MongoDB 中用映射实现这种功能。

MongoDB 中限制字段的显示，可以利用 0 或 1 来设置字段列表。1 用于显示字段，0 用于隐藏字段。

**格式**

```
>db.COLLECTION_NAME.find().limit(NUMBER)
```

示例：

```
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user2", "age" : 19 }
{ "_id" : ObjectId("5ce77f22044e09e008f48b66"), "name" : "user1", "age" : 55 }
> db.user.find({},{"_id":0})
{ "name" : "user2", "age" : 19 }
{ "name" : "user1", "age" : 55 }
```

在执行 `find()` 方法时，`_id` 字段是一直显示的。如果不想显示该字段，则可以设置 "_id":0。

MongoDB 中想要显示或者跳过指定的文档条数，可以利用 limit() 方法和 skip() 方法

limit() 方法接受一个数值类型的参数，其值为想要显示的文档数。

**格式**

```
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

示例：

```
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user2", "age" : 19 }
{ "_id" : ObjectId("5ce77f22044e09e008f48b66"), "name" : "user1", "age" : 55 }
> db.user.find().limit(1)
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user2", "age" : 19 }
```

如果不给 limit() 指定参数会返回全部文档。

skip() 方法接受一个数值类型的参数，其值为想要跳过的文档数。

**格式**

```
`db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)`
```

示例

```
> db.user.find()
{ "_id" : ObjectId("5ce77e66044e09e008f48b65"), "name" : "user2", "age" : 19 }
{ "_id" : ObjectId("5ce77f22044e09e008f48b66"), "name" : "user1", "age" : 55 }
> db.user.find().limit(1).skip(1)
{ "_id" : ObjectId("5ce77f22044e09e008f48b66"), "name" : "user1", "age" : 55 }
```

skip() 方法的默认值是 0 。

#### 排序

在 MongoDB 中使用 sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。

sort()方法基本语法如下所示：

```
>db.COLLECTION_NAME.find().sort({KEY:1})
```

以下实例演示了 col 集合中的数据按字段 likes 的降序排列：

```
>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
{ "title" : "MongoDB数据库" }
{ "title" : "Java 语言" }
```

skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 sort(), 然后是 skip()，最后是显示的 limit()。

### 索引

使用索引可以大大提高文档的查询效率。如果没有索引，会遍历集合中所有文档，才能找到匹配查询语句的文档。这样遍历集合中整个文档的方式是非常耗时的，特别是处理大数据时，耗时几十秒甚至几分钟都是有可能的。

1. 查看集合索引

```
db.collection.getIndexes()
```

2. 查看集合索引大小

```
db.collection.totalIndexSize()
```

3. 删除集合所有索引

```
db.collection.dropIndexes()
```

4. 删除集合指定索引

```
db.collection.dropIndex("索引名称")
```

##### ensureIndex()

MongoDB 中，使用 ensureIndex() 方法创建索引。

**格式**

```
>db.collection.createIndex(keys, options)
```

其中，KEY表示要创建索引的字段名称，1 表示按升序排列字段值。-1 表示按降序排列。

给 user 集合中 name 字段添加索引

```
>db.user.ensureIndex({"name":1})
```

MongoDB 中用 db.collection.getIndexes() 方法查询集合中所有的索引，我们查询一下 user 中所有的索引。

```
> db.user.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "learn.user"
	},
	{
		"v" : 2,
		"key" : {
			"name" : 1
		},
		"name" : "name_1",
		"ns" : "learn.user"
	}
]

```

我们发现 user 中有两个索引，其中索引 "_id_" 是我们创建 user 集合时，MongoDB 自动生成的索引。第二个索引就是我们刚才创建的索引，其中，name 值"name_1"表示索引名称，MongoDB 会自动生成的索引名称。当然，我们也可以自己指定索引的名称。

给 user 集合中 age 字段添加索引，并指定索引名称为 "index_age_esc"。

```
> db.user.ensureIndex({"age":1},{name:"index_age_esc"})
> db.user.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "learn.user"
	},
	{
		"v" : 2,
		"key" : {
			"name" : 1
		},
		"name" : "name_1",
		"ns" : "learn.user"
	},
	{
		"v" : 2,
		"key" : {
			"age" : 1
		},
		"name" : "index_age_esc",
		"ns" : "learn.user"
	}
]

```

指定索引名称用到的 name 参数，只是 ensureIndex() 方法可接收可选参数的其中一个，下表列出了 ensureIndex() 方法可接收的参数

| Parameter          | Type           | Description                                                  |
| ------------------ | -------------- | ------------------------------------------------------------ |
| background         | 布尔值         | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique             | 布尔值         | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name               | 字符串         | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | 布尔值         | 在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | 布尔值         | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | 整型           | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                  | 索引版本       | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights            | 文档(document) | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | 字符串         | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  | 字符串         | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

1. 唯一索引：MongoDB和关系型数据库一样都可以建立唯一索引，重复的键值就不能重新插入了，MongoDB 用 unigue 来确定建立的索引是否为唯一索引，true 表示为唯一索引
2. 复合索引：ensureIndex() 方法中你也可以设置使用多个字段创建索引

##### dropIndex()

MongoDB 用dropIndex() 方法删除索引

**格式**

```
db.COLLECTION_NAME.dropIndex()
```

**注：**dropIndex() 方法可根据指定的索引名称或索引文档删除索引（_id上的默认索引除外）

我们用两种方式删除掉 user 中 name 字段上的索引

```
>db.user.dropIndex("name_1")     #根据索引名称删除索引
>db.user.dropIndex({"name":1})   #根据索引文档删除索引
```

还可以用 dropIndexes() 删除集合中所有索引（_id上的默认索引除外）

```
>db.user.dropIndexs()
```

#### 查询分析

查询分析是查询语句性能分析的重要工具。

MongoDB 中查询分析用 explain() 和 hint() 方法

我们向集合 user 中插入20万条数据，利用 explain() 查询建立索引前后，执行时间的比较，来看看建立索引对查询效率的提高程度。

**第一步**，向 user 中插入20万条数据

```
>db.user.remove({})
>for(var i = 0; i <200000; i++){db.user.insert({"name":"px"+i,"age":18})}
```

**第二步**，删除 user 集合中字段 name 上的索引，然后查询 name = "lrh100000"，利用explain("executionStats")查询此时执行的时间。如果想详细了解，[请访问官网](https://docs.mongodb.com/manual/reference/method/cursor.explain/)。

```
>db.user.dropIndexes()      #删除所有索引
>db.user.find({"name":"px100000"}).explain("executionStats")
```

explain.executionStats.executionTimeMillis：表示查询所用的时间，单位是毫秒。我们可以清楚的看出，没用索引查询用到的时间是 104 毫秒。

**第三步**，给 user 集合中 name 字段添加索引，然后再查询同一个条件，看执行查询所用了多久时间。

```
>db.user.ensureIndex({"name":1})     
>db.user.find({"name":"px100000"}).explain("executionStats")
```

如果用到了索引，explain() 方法会返回 winningPlan，标识用到的索引名称 indexName

我们可以清楚到处，用了索引，执行时间只有 3 毫秒，可以看出，查询效率的提高可不是一星半点。

**第四步**，这一步我们重点看看 hint() 方法的用法。hint() 方法用来强制 MongoDB 使用一个指定的索引。

我们给 user 再添加一个 {"name":1, "age":1}，利用 explain() 方法，看一下用到了哪个索引。

```
>db.user.ensureIndex({"name":1, "age":1})     
>db.user.find({"name":"px100000"}).explain("executionStats")
```

可以看出，此时用到的索引是 "name_1_age_1"，如果我们想用索引 "name_1"，就可以用 hint() 方法指定。

```
>db.user.find({"name":"px100000"}).hint({"name":1}).explain("executionStats")
```

### 用户管理

修改`/etc/mongod.conf`文件如下开启认证：

```undefined
security:
  authorization: enabled
```

然后重启MongoDB服务器：`service mongod restart`

##### 3.0版本以前

在mongodb3.0版本以前中,有一个admin数据库, 牵涉到服务器配置层面的操作,需要先切换到admin数据库，即 use admin , 相当于进入超级用户管理模式,mongo的用户是以数据库为单位来建立的, 每个数据库有自己的管理员.我们在设置用户时,需要先在admin数据库下建立管理员---这个管理员登陆后,相当于超级管理员

```css
命令:db.addUser();
简单参数: db.addUser(用户名,密码,是否只读)
```

注意: 添加用户后,我们再次退出并登陆,发现依然可以直接读数据库，原因是 mongodb服务器启动时, 默认不是需要认证的，要让用户生效需要启动服务器时就指定`--auth`选项。

```bash
# 添加用户
> use admin
> db.addUser('admin','admin',false); # 3.0版本更改为createUser();

# 删除用户
> use test
> db.removeUser(用户名); # 3.0版本更改为dropUser();
```

##### 3.0版本以后

##### 创建管理员

在3.0版本以后,mongodb默认是没有admin这个数据库的,并且创建管理员不再用addUser,而用createUser;

##### 语法说明

```bash
{ user: "<name>",  
  pwd: "<cleartext password>",
  customData: { <any information> }, # 任意的数据,一般是用于描述用户管理员的信息
  roles: [
    { role: "<role>", db: "<database>" } | "<role>", # 如果是role就是直接指定了角色,并作用于当前的数据库
    ...
  ] # roles是必传项,但是可以指定空数组,为空就是不指定任何权限
}
```

Built-In Roles（[内置角色](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.mongodb.org%2Fmanual%2Freference%2Fbuilt-in-roles%2F%23built-in-roles)）：

1. 数据库用户角色：read、readWrite;
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root
    这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
7. 内部角色：__system
    PS：关于每个角色所拥有的操作权限可以点击上面的内置角色链接查看详情。

**操作示例：**

```bash
#创建数据库aaa同时创建用户aaa并作用于当前数据库
use aaa
db.createUser({"user":"aaa","pwd":"000000","roles":["readWrite"]})
```

**官方Example**

```bash
use products # mongoDB的权限设置是以库为单位的,必选要先选择库
db.createUser( 
{ "user" : "accountAdmin01", 
 "pwd": "cleartext password",
 "customData" : { employeeId: 12345 },
 "roles" : [ { role: "clusterAdmin", db: "admin" }, 
             { role: "readAnyDatabase", db: "admin" },
             "readWrite" 
             ] },
{ w: "majority" , wtimeout: 5000 } ) # readWrite 适用于products库,clusterAdmin与readAnyDatabase角色适用于admin库
```

writeConcern文档（[官方说明](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.mongodb.org%2Fmanual%2Freference%2Fwrite-concern%2F)）

- w选项：允许的值分别是 1、0、大于1的值、"majority"、；
- j选项：确保mongod实例写数据到磁盘上的journal（日志），这可以确保mongd以外关闭不会丢失数据。设置true启用。
- wtimeout：指定一个时间限制,以毫秒为单位。wtimeout只适用于w值大于1。

```css
use shop;
db.createUser({
    user:'admin',
    pwd:'zhouzhou123',
    roles:['dbOwner']
})
```

注：只要新加了一个用户,admin数据库就会重新存在;

```css
mongo --host xxx -u admin -p zhouzhou123 --authenticationDatabase shop # 用新创建的用户登录

# 查看当前用户在shop数据库的权限
use shop;
db.runCommand(
  {
    usersInfo:"shopzhouzhou",
    showPrivileges:true
  }
)

# 查看用户信息
db.runCommand({usersInfo:"userName"})

# 创建一个不受访问限制的超级用户
use admin
db.createUser(
  {
    user:"superuser",
    pwd:"pwd",
    roles:["root"]
  }
)
```

##### 认证用户

```php
> use test
> db.auth(用户名,密码); #注意是以库为单位,必须先选择库;
```

##### 删除用户

```bash
# 删除用户
> use test
> db.dropUser('用户名');
```

##### 修改用户密码

```css
> use test
> db.changeUserPassword(用户名, 新密码);

# 修改密码和用户信息
db.runCommand(
  {
    updateUser:"username",
    pwd:"xxx",
    customData:{title:"xxx"}
  }
)
```

### 聚合管道

在讲解聚合管道（Aggregation Pipeline）之前，我们先介绍一下 MongoDB  的聚合功能，聚合操作主要用于对数据的批量处理，往往将记录按条件分组以后，然后再进行一系列操作，例如，求最大值、最小值、平均值，求和等操作。聚合操作还能够对记录进行复杂的操作，主要用于数理统计和数据挖掘。在  MongoDB 中，聚合操作的输入是集合中的文档，输出可以是一个文档，也可以是多条文档。

MongoDB 提供了非常强大的聚合操作，有三种方式：

- 聚合管道（Aggregation Pipeline）
- 单目的聚合操作（Single Purpose Aggregation Operation）
- MapReduce 编程模型

聚合管道是 MongoDB 2.2版本引入的新功能。它由阶段（Stage）组成，文档在一个阶段处理完毕后，聚合管道会把处理结果传到下一个阶段。

**聚合管道功能：**

- 对文档进行过滤，查询出符合条件的文档
- 对文档进行变换，改变文档的输出形式

每个阶段用**阶段操作符（Stage Operators）**定义，在每个阶段操作符中可以用**表达式操作符（Expression Operators）**计算总和、平均值、拼接分割字符串等相关操作，直到每个阶段进行完成，最终返回结果，返回的结果可以直接输出，也可以存储到集合中。

MongoDB 中使用 `db.COLLECTION_NAME.aggregate([{<stage>},...])` 方法来构建和使用聚合管道。先看下官网给的实例，感受一下聚合管道的用法。

![3.png](https://i.loli.net/2019/05/24/5ce787a2e05fc91062.png)

实例中，$match 用于获取 status = "A" 的记录，然后将符合条件的记录送到下一阶段 $group 中进行分组求和计算，最后返回 Results。其中，$match、$group 都是阶段操作符，而阶段 $group 中用到的 $sum 是表达式操作符。

#### 阶段操作符

使用阶段操作符之前，我们先看一下 article 集合中的文档列表，也就是范例中用到的数据。

```
> db.article.find()
{ "_id" : ObjectId("5ce7894d044e09e008f798a7"), "title" : "MongoDB Aggregate", "author" : "liruihuan", "tags" : [ "Mongodb", "Database", "Query" ], "pages" : 5, "time" : ISODate("2017-04-09T11:42:39.736Z") }
{ "_id" : ObjectId("58e1d2f0bb1bbc3245fa7571"), "title" : "MongoDB Index", "author" : "liruihuan", "tags" : [ "Mongodb", "Index", "Query" ], "pages" : 3, "time" : ISODate("2017-04-09T11:43:39.236Z") }
{ "_id" : ObjectId("5ce789bc044e09e008f798a8"), "title" : "MongoDB Query", "author" : "eryueyang", "tags" : [ "Mongodb", "Query" ], "pages" : 8, "time" : ISODate("2017-04-09T11:44:56.276Z") 
```

**1. $project** 

**作用**

修改文档的结构，可以用来重命名、增加或删除文档中的字段。

**范例1**

只返回文档中 title 和 author 字段

```
> db.article.aggregate([{$project:{_id:0, title:1, author:1 }}])
{ "title" : "MongoDB Aggregate", "author" : "liruihuan" }
{ "title" : "MongoDB Index", "author" : "liruihuan" }
{ "title" : "MongoDB Query", "author" : "eryueyang" }
```

因为字段 _id 是默认显示的，这里必须用 _id:0 把字段_id过滤掉。

**范例2**

把文档中 pages 字段的值都增加10。并重命名成 newPages 字段。

```
> db.article.aggregate(
...    [
...       {
...           $project:{
...                _id:0,
...                title:1,
...                author:1,
...                newPages: {$add:["$Pages",10]}
...          }
...       }
...    ]
... )
{ "title" : "MongoDB Aggregate", "author" : "liruihuan", "newPages" : null }
{ "title" : "MongoDB Index", "author" : "liruihuan", "newPages" : null }
{ "title" : "MongoDB Query", "author" : "eryueyang", "newPages" : null }
```

其中，$add 是 加 的意思，是算术类型表达式操作符，具体表达式操作符，下面会讲到。

**2. $match**

**作用**

用于过滤文档。用法类似于 find() 方法中的参数。

**范例**

查询出文档中 pages 字段的值大于等于5的数据。

```
> db.article.aggregate(     [          {               $match: {"pages": {$gte: 5}}          }     ]    ).pretty()
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Database",
		"Query"
	],
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}
{
	"_id" : ObjectId("5ce789bc044e09e008f798a8"),
	"title" : "MongoDB Query",
	"author" : "eryueyang",
	"tags" : [
		"Mongodb",
		"Query"
	],
	"pages" : 8,
	"time" : ISODate("2017-04-09T11:44:56.276Z")
}


```

**注：**

- 在 $match 中不能使用 $where 表达式操作符
- 如果 $match 位于管道的第一个阶段，可以利用索引来提高查询效率
- $match 中使用 $text 操作符的话，只能位于管道的第一阶段
- $match 尽量出现在管道的最前面，过滤出需要的数据，在后续的阶段中可以提高效率。

**3. $group**

**作用**

将集合中的文档进行分组，可用于统计结果。

**范例**

从 article 中得到每个 author 的文章数，并输入 author 和对应的文章数。

```
>db.article.aggregate(
    [
         {
              $group: {_id: "$author", total: {$sum: 1}}
         }
    ]
   )
{ "_id" : "eryueyang", "total" : 1 }
{ "_id" : "liruihuan", "total" : 2 }
  
```

**4. $sort**

**作用**

将集合中的文档进行排序。

**范例**

让集合 article 以 pages 升序排列

```
>db.article.aggregate([{$sort: {"pages": 1}}]).pretty()
{
	"_id" : ObjectId("58e1d2f0bb1bbc3245fa7571"),
	"title" : "MongoDB Index",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Index",
		"Query"
	],
	"pages" : 3,
	"time" : ISODate("2017-04-09T11:43:39.236Z")
}
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Database",
		"Query"
	],
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}
{
	"_id" : ObjectId("5ce789bc044e09e008f798a8"),
	"title" : "MongoDB Query",
	"author" : "eryueyang",
	"tags" : [
		"Mongodb",
		"Query"
	],
	"pages" : 8,
	"time" : ISODate("2017-04-09T11:44:56.276Z")
}

```

**5. $limit**

**作用**

限制返回的文档数量

**范例**

返回集合 article 中前两条文档

```
> db.article.aggregate([{$limit: 2}]).pretty()
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Database",
		"Query"
	],
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}
{
	"_id" : ObjectId("58e1d2f0bb1bbc3245fa7571"),
	"title" : "MongoDB Index",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Index",
		"Query"
	],
	"pages" : 3,
	"time" : ISODate("2017-04-09T11:43:39.236Z")
}

```

**6. $skip**

**作用**

跳过指定数量的文档，并返回余下的文档。

**范例**

跳过集合 article 中一条文档，输出剩下的文档

```
> db.article.aggregate([{$skip: 1}]).pretty()
{
	"_id" : ObjectId("58e1d2f0bb1bbc3245fa7571"),
	"title" : "MongoDB Index",
	"author" : "liruihuan",
	"tags" : [
		"Mongodb",
		"Index",
		"Query"
	],
	"pages" : 3,
	"time" : ISODate("2017-04-09T11:43:39.236Z")
}
{
	"_id" : ObjectId("5ce789bc044e09e008f798a8"),
	"title" : "MongoDB Query",
	"author" : "eryueyang",
	"tags" : [
		"Mongodb",
		"Query"
	],
	"pages" : 8,
	"time" : ISODate("2017-04-09T11:44:56.276Z")
}

```

**7. $unwind**

**作用**

将文档中数组类型的字段拆分成多条，每条文档包含数组中的一个值。

**范例**

把集合 article 中 title="MongoDB Aggregate" 的 tags 字段拆分

```
>db.article.aggregate(
    [
         {
              $match: {"title": "MongoDB Aggregate"}
         },
         {
              $unwind: "$tags"
         }
    ]
   ).pretty()
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : "Mongodb",
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : "Database",
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}
{
	"_id" : ObjectId("5ce7894d044e09e008f798a7"),
	"title" : "MongoDB Aggregate",
	"author" : "liruihuan",
	"tags" : "Query",
	"pages" : 5,
	"time" : ISODate("2017-04-09T11:42:39.736Z")
}

```

**注：**

- $unwind 参数数组字段为空或不存在时，待处理的文档将会被忽略，该文档将不会有任何输出
- $unwind 参数不是一个数组类型时，将会抛出异常
- $unwind 所作的修改，只用于输出，不能改变原文档

#### 表达式操作符

 表达式操作符有很多操作类型，其中最常用的有布尔管道聚合操作、集合操作、比较聚合操作、算术聚合操作、字符串聚合操作、数组聚合操作、日期聚合操作、条件聚合操作、数据类型聚合操作等。每种类型都有很多用法，这里就不一一举例了。

##### 布尔管道聚合操作（Boolean Aggregation Operators）

| 名称   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `$and` | Returns `true` only when *all* its expressions evaluate to `true`. Accepts any number of argument expressions. |
| `$or`  | Returns `true` when *any* of its expressions evaluates to `true`. Accepts any number of argument expressions. |
| `$not` | Returns the boolean value that is the opposite of its argument expression. Accepts a single argument expression. |

假如有一个集合 mycol

```
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 }
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
```

确定 qty 是否大于250或者小于200

```
db.mycol.aggregate(
   [
     {
       $project:
          {
            item: 1,
            result: { $or: [ { $gt: [ "$qty", 250 ] }, { $lt: [ "$qty", 200 ] } ] }
          }
     }
   ]
)
```

##### 集合操作（Set Operators）

用于集合操作，求集合的并集、交集、差集运算。

| 名称               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `$setEquals`       | Returns `true` if the input sets have the same distinct elements. Accepts two or more argument expressions. |
| `$setIntersection` | Returns a set with elements that appear in *all* of the input sets. Accepts any number of argument expressions. |
| `$setUnion`        | Returns a set with elements that appear in *any* of the input sets. Accepts any number of argument expressions. |
| `$setDifference`   | Returns a set with elements that appear in the first set but not in  the second set; i.e. performs a relative complement of the second set  relative to the first. Accepts exactly two argument expressions. |
| `$setIsSubset`     | Returns `true` if all elements  of the first set appear in the second set, including when the first set  equals the second set; i.e. not a strict subset. Accepts exactly two  argument expressions. |
| `$anyElementTrue`  | Returns `true` if *any* elements of a set evaluate to `true`; otherwise, returns `false`. Accepts a single argument expression. |
| `$allElementsTrue` | Returns `true` if *no* element of a set evaluates to `false`, otherwise, returns `false`. Accepts a single argument expression. |

**范例**

假如有一个集合 mycol

```
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] }
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] }
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] }
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] }
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] }
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] }
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] }
{ "_id" : 8, "A" : [ ], "B" : [ ] }
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
```

求出集合 mycol 中 A 和 B 的交集

```
db.mycol.aggregate(
   [
     { $project: { A:1, B: 1, allValues: { $setUnion: [ "$A", "$B" ] }, _id: 0 } }
   ]
)
{ "A": [ "red", "blue" ], "B": [ "red", "blue" ], "allValues": [ "blue", "red" ] }
{ "A": [ "red", "blue" ], "B": [ "blue", "red", "blue" ], "allValues": [ "blue", "red" ] }
{ "A": [ "red", "blue" ], "B": [ "red", "blue", "green" ], "allValues": [ "blue", "red", "green" ] }
{ "A": [ "red", "blue" ], "B": [ "green", "red" ], "allValues": [ "blue", "red", "green" ] }
{ "A": [ "red", "blue" ], "B": [ ], "allValues": [ "blue", "red" ] }
{ "A": [ "red", "blue" ], "B": [ [ "red" ], [ "blue" ] ], "allValues": [ "blue", "red", [ "red" ], [ "blue" ] ] }
{ "A": [ "red", "blue" ], "B": [ [ "red", "blue" ] ], "allValues": [ "blue", "red", [ "red", "blue" ] ] }
{ "A": [ ], "B": [ ], "allValues": [ ] }
{ "A": [ ], "B": [ "red" ], "allValues": [ "red" ] }
```

##### 比较聚合操作（Comparison Aggregation Operators）

| 名称   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `$cmp` | Returns: `0` if the two values are equivalent, `1` if the first value is greater than the second, and `-1` if the first value is less than the second. |
| `$eq`  | Returns `true` if the values are equivalent.                 |
| `$gt`  | Returns `true` if the first value is greater than the second. |
| `$gte` | Returns `true` if the first value is greater than or equal to the second. |
| `$lt`  | Returns `true` if the first value is less than the second.   |
| `$lte` | Returns `true` if the first value is less than or equal to the second. |
| `$ne`  | Returns `true` if the values are *not* equivalent.           |

##### 算术聚合操作（Arithmetic Aggregation Operators）

| 名称        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| `$abs`      | Returns the absolute value of a number.                      |
| `$add`      | Adds numbers to return the sum, or adds numbers and a date to return  a new date. If adding numbers and a date, treats the numbers as  milliseconds. Accepts any number of argument expressions, but at most,  one expression can resolve to a date. |
| `$ceil`     | Returns the smallest integer greater than or equal to the specified number. |
| `$divide`   | Returns the result of dividing the first number by the second. Accepts two argument expressions. |
| `$exp`      | Raises *e* to the specified exponent.                        |
| `$floor`    | Returns the largest integer less than or equal to the specified number. |
| `$ln`       | Calculates the natural log of a number.                      |
| `$log`      | Calculates the log of a number in the specified base.        |
| `$log10`    | Calculates the log base 10 of a number.                      |
| `$mod`      | Returns the remainder of the first number divided by the second. Accepts two argument expressions. |
| `$multiply` | Multiplies numbers to return the product. Accepts any number of argument expressions. |
| `$pow`      | Raises a number to the specified exponent.                   |
| `$sqrt`     | Calculates the square root.                                  |
| `$subtract` | Returns the result of subtracting the second value from the first.  If the two values are numbers, return the difference. If the two values  are dates, return the difference in milliseconds. If the two values are a  date and a number in milliseconds, return the resulting date. Accepts  two argument expressions. If the two values are a date and a number,  specify the date argument first as it is not meaningful to subtract a  date from a number. |
| `$trunc`    | Truncates a number to its integer.                           |

**范例**

假如有一个集合 mycol

```
{ _id: 1, start: 5, end: 8 }
{ _id: 2, start: 4, end: 4 }
{ _id: 3, start: 9, end: 7 }
{ _id: 4, start: 6, end: 7 }
```

求集合 mycol 中 start 减去 end 的绝对值

```
db.mycol.aggregate([
   {
     $project: { delta: { $abs: { $subtract: [ "$start", "$end" ] } } }
   }
])
{ "_id" : 1, "delta" : 3 }
{ "_id" : 2, "delta" : 0 }
{ "_id" : 3, "delta" : 2 }
{ "_id" : 4, "delta" : 1 }
```

##### 字符串聚合操作（String Aggregation Operators）

| 名称            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `$concat`       | Concatenates any number of strings.                          |
| `$indexOfBytes` | Searches a string for an occurence of a substring and returns the  UTF-8 byte index of the first occurence. If the substring is not found,  returns `-1`. |
| `$indexOfCP`    | Searches a string for an occurence of a substring and returns the  UTF-8 code point index of the first occurence. If the substring is not  found, returns `-1`. |
| `$split`        | Splits a string into substrings based on a delimiter. Returns an  array of substrings. If the delimiter is not found within the string,  returns an array containing the original string. |
| `$strLenBytes`  | Returns the number of UTF-8 encoded bytes in a string.       |
| `$strLenCP`     | Returns the number of UTF-8 code points in a string.         |
| `$strcasecmp`   | Performs case-insensitive string comparison and returns: `0` if two strings are equivalent, `1` if the first string is greater than the second, and `-1` if the first string is less than the second. |
| `$substr`       | Deprecated. Use `$substrBytes` or `$substrCP`.               |
| `$substrBytes`  | Returns the substring of a string. Starts with the character at the  specified UTF-8 byte index (zero-based) in the string and continues for  the specified number of bytes. |
| `$substrCP`     | Returns the substring of a string. Starts with the character at the  specified UTF-8 code point (CP) index (zero-based) in the string and  continues for the number of code points specified. |
| `$toLower`      | Converts a string to lowercase. Accepts a single argument expression. |
| `$toUpper`      | Converts a string to uppercase. Accepts a single argument expression. |

**范例**

假如有一个集合 mycol

```
{ "_id" : 1, "city" : "Berkeley, CA", "qty" : 648 }
{ "_id" : 2, "city" : "Bend, OR", "qty" : 491 }
{ "_id" : 3, "city" : "Kensington, CA", "qty" : 233 }
{ "_id" : 4, "city" : "Eugene, OR", "qty" : 842 }
{ "_id" : 5, "city" : "Reno, NV", "qty" : 655 }
{ "_id" : 6, "city" : "Portland, OR", "qty" : 408 }
{ "_id" : 7, "city" : "Sacramento, CA", "qty" : 574 }
```

以 ',' 分割集合 mycol 中字符串city的值，用 $unwind 拆分成多个文档，匹配出城市名称只有两个字母的城市，并求和各个城市中 qty 的值，最后以降序排序。

```
db.mycol.aggregate([
  { $project : { city_state : { $split: ["$city", ", "] }, qty : 1 } },
  { $unwind : "$city_state" },
  { $match : { city_state : /[A-Z]{2}/ } },
  { $group : { _id: { "state" : "$city_state" }, total_qty : { "$sum" : "$qty" } } },
  { $sort : { total_qty : -1 } }
])
{ "_id" : { "state" : "OR" }, "total_qty" : 1741 }
{ "_id" : { "state" : "CA" }, "total_qty" : 1455 }
{ "_id" : { "state" : "NV" }, "total_qty" : 655 }
```

##### 数组聚合操作（Array Aggregation Operators） 

| 名称            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `$arrayElemAt`  | Returns the element at the specified array index.            |
| `$concatArrays` | Concatenates arrays to return the concatenated array.        |
| `$filter`       | Selects a subset of the array to return an array with only the elements that match the filter condition. |
| `$indexOfArray` | Searches an array for an occurence of a specified value and returns  the array index of the first occurence. If the substring is not found,  returns `-1`. |
| `$isArray`      | Determines if the operand is an array. Returns a boolean.    |
| `$range`        | Outputs an array containing a sequence of integers according to user-defined inputs. |
| `$reverseArray` | Returns an array with the elements in reverse order.         |
| `$reduce`       | Applies an expression to each element in an array and combines them into a single value. |
| `$size`         | Returns the number of elements in the array. Accepts a single expression as argument. |
| `$slice`        | Returns a subset of an array.                                |
| `$zip`          | Merge two lists together.                                    |
| `$in`           | Returns a boolean indicating whether a specified value is in an array. |

**范例**

假如有一个集合 mycol

```
{ "_id" : 1, "name" : "dave123", favorites: [ "chocolate", "cake", "butter", "apples" ] }
{ "_id" : 2, "name" : "li", favorites: [ "apples", "pudding", "pie" ] }
{ "_id" : 3, "name" : "ahn", favorites: [ "pears", "pecans", "chocolate", "cherries" ] }
{ "_id" : 4, "name" : "ty", favorites: [ "ice cream" ] }
```

求出集合 mycol 中 favorites 的第一项和最后一项

```
db.mycol.aggregate([
   {
     $project:
      {
         name: 1,
         first: { $arrayElemAt: [ "$favorites", 0 ] },
         last: { $arrayElemAt: [ "$favorites", -1 ] }
      }
   }
])
{ "_id" : 1, "name" : "dave123", "first" : "chocolate", "last" : "apples" }
{ "_id" : 2, "name" : "li", "first" : "apples", "last" : "pie" }
{ "_id" : 3, "name" : "ahn", "first" : "pears", "last" : "cherries" }
{ "_id" : 4, "name" : "ty", "first" : "ice cream", "last" : "ice cream" }
```

##### 日期聚合操作（Date Aggregation Operators）

| 名称            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `$dayOfYear`    | Returns the day of the year for a date as a number between 1 and 366 (leap year). |
| `$dayOfMonth`   | Returns the day of the month for a date as a number between 1 and 31. |
| `$dayOfWeek`    | Returns the day of the week for a date as a number between 1 (Sunday) and 7 (Saturday). |
| `$year`         | Returns the year for a date as a number (e.g. 2014).         |
| `$month`        | Returns the month for a date as a number between 1 (January) and 12 (December). |
| `$week`         | Returns the week number for a date as a number between 0 (the  partial week that precedes the first Sunday of the year) and 53 (leap  year). |
| `$hour`         | Returns the hour for a date as a number between 0 and 23.    |
| `$minute`       | Returns the minute for a date as a number between 0 and 59.  |
| `$second`       | Returns the seconds for a date as a number between 0 and 60 (leap seconds). |
| `$millisecond`  | Returns the milliseconds of a date as a number between 0 and 999. |
| `$dateToString` | Returns the date as a formatted string.                      |
| `$isoDayOfWeek` | Returns the weekday number in ISO 8601 format, ranging from `1` (for Monday) to `7` (for Sunday). |
| `$isoWeek`      | Returns the week number in ISO 8601 format, ranging from `1` to `53`. Week numbers start at `1` with the week (Monday through Sunday) that contains the year’s first Thursday. |
| `$isoWeekYear`  | Returns the year number in ISO 8601 format. The year starts with the  Monday of week 1 (ISO 8601) and ends with the Sunday of the last week  (ISO 8601). |

**范例**

假如有一个集合 mycol

```
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2017-01-01T08:15:39.736Z") }
```

得到集合 mycol 中 date 字段的相关日期值

```
db.mycol.aggregate(
   [
     {
       $project:
         {
           year: { $year: "$date" },
           month: { $month: "$date" },
           day: { $dayOfMonth: "$date" },
           hour: { $hour: "$date" },
           minutes: { $minute: "$date" },
           seconds: { $second: "$date" },
           milliseconds: { $millisecond: "$date" },
           dayOfYear: { $dayOfYear: "$date" },
           dayOfWeek: { $dayOfWeek: "$date" },
           week: { $week: "$date" }
         }
     }
   ]
)
{
  "_id" : 1,
  "year" : 2017,
  "month" : 1,
  "day" : 1,
  "hour" : 8,
  "minutes" : 15,
  "seconds" : 39,
  "milliseconds" : 736,
  "dayOfYear" : 1,
  "dayOfWeek" : 1,
  "week" : 0
}
```

##### 条件聚合操作（Conditional Aggregation Operators）

| 名称      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| `$cond`   | A ternary operator that evaluates one expression, and depending on  the result, returns the value of one of the other two expressions.  Accepts either three expressions in an ordered list or three named  parameters. |
| `$ifNull` | Returns either the non-null result of the first expression or the  result of the second expression if the first expression results in a  null result. Null result encompasses instances of undefined values or  missing fields. Accepts two expressions as arguments. The result of the  second expression can be null. |
| `$switch` | Evaluates a series of case expressions. When it finds an expression which evaluates to `true`, `$switch` executes a specified expression and breaks out of the control flow. |

**范例**

假如有一个集合 mycol

```
{ "_id" : 1, "item" : "abc1", qty: 300 }
{ "_id" : 2, "item" : "abc2", qty: 200 }
{ "_id" : 3, "item" : "xyz1", qty: 250 }
```

如果集合 mycol 中 qty 字段值大于等于250，则返回30，否则返回20

```
db.mycol.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
               }
           }
      }
   ]
)
{ "_id" : 1, "item" : "abc1", "discount" : 30 }
{ "_id" : 2, "item" : "abc2", "discount" : 20 }
{ "_id" : 3, "item" : "xyz1", "discount" : 30 }
```

##### 数据类型聚合操作（Data Type Aggregation Operators）

| 名称    | 说明                                    |
| ------- | --------------------------------------- |
| `$type` | Return the BSON data type of the field. |

**范例**

假如有一个集合 mycol

```
{ _id: 0, a : 8 }
{ _id: 1, a : [ 41.63, 88.19 ] }
{ _id: 2, a : { a : "apple", b : "banana", c: "carrot" } }
{ _id: 3, a :  "caribou" }
{ _id: 4, a : NumberLong(71) }
{ _id: 5 }
```

获取文档中 a 字段的数据类型

```
db.mycol.aggregate([{
    $project: {
       a : { $type: "$a" }
    }
}])
{ _id: 0, "a" : "double" }
{ _id: 1, "a" : "array" }
{ _id: 2, "a" : "object" }
{ _id: 3, "a" : "string" }
{ _id: 4, "a" : "long" }
{ _id: 5, "a" : "missing" }
```

#### 聚合管道优化

默认情况下，整个集合作为聚合管道的输入，为了提高处理数据的效率，可以使用一下策略：

- 将 $match 和 $sort 放到管道的前面，可以给集合建立索引，来提高处理数据的效率。
- 可以用 $match、$limit、$skip 对文档进行提前过滤，以减少后续处理文档的数量。

当聚合管道执行命令时，MongoDB 也会对各个阶段自动进行优化，主要包括以下几个情况：

1. $sort + $match 顺序优化

如果 $match 出现在 $sort 之后，优化器会自动把 $match 放到 $sort 前面

2. $skip + $limit 顺序优化

如果 $skip 在 $limit 之后，优化器会把 $limit 移动到 $skip 的前面，移动后 $limit的值等于原来的值加上 $skip 的值。

例如：移动前：{$skip: 10, $limit: 5}，移动后：{$limit: 15, $skip: 10}

#### 使用限制

对聚合管道的限制主要是对 返回结果大小 和 内存 的限制。

**返回结果大小**

聚合结果返回的是一个文档，不能超过 16M，从 MongoDB 2.6版本以后，返回的结果可以是一个游标或者存储到集合中，返回的结果不受 16M 的限制。

**内存**

聚合管道的每个阶段最多只能用 100M 的内存，如果超过100M，会报错，如果需要处理大数据，可以使用 allowDiskUse 选项，存储到磁盘上。

#### 单目的聚合操作

单目的聚合命令，常用的：count()、distinct()，与聚合管道相比，单目的聚合操作更简单，使用非常频繁。先通过 distinct() 看一下工作流程

![4.png](https://i.loli.net/2019/05/24/5ce78d9b994b339614.png)

distinct() 的作用是去重。而 count() 是求文档的个数。

下面用 count() 方法举例说明一下

**范例**

求出集合 article 中 time 值大于 2017-04-09 的文档个数

```
>db.article.count( { time: { $gt: new Date('04/09/2017') } } )
```

这个语句等价于

```
db.article.find( { time: { $gt: new Date('04/09/2017') } } ).count()
```

