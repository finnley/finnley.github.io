+++
title = 'MongoDB基本操作'
date = 2024-05-12T08:22:05+08:00
draft = true
categories = [ "MongoDB" ]
+++

## MongoDB 初识

### 是什么？

* 文档数据库
* NoSQL类型数据库（非关系型数据库）
* C++编写的开源数据库系统

### 特点

* 面向文档。以 json 格式存储数据。
* 高性能。数据存储与内存，查询快，性能高。
* 高可用。自带高可用方案，如副本集、分片集群。
* 丰富的查询语句
* 支持多种主流开发语言。如java, golang
* 可添加索引加快检索速度。


### 架构

* 单实例
* 副本集
* 分片


## 安装部署

学习之前需要学会MongoDB的安装部署。工欲善其事必先利其器。

下载：
```bash
cd /opt
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.27.tgz
```

解压：
```bash
# tar zxvf mongodb-linux-x86_64-rhel70-5.0.27.tgz
```

创建数据目录和日志目录：
```bash
# mkdir -p /data/mongodb/{db,log}
```

添加环境变量：
```bash
# export PATH=$PATH:/opt/mongodb-linux-x86_64-rhel70-5.0.27/bin/
```

启动：
```bash
# mongod --dbpath /data/mongodb/db --logpath /data/mongodb/log/mongod.log --fork --bind_ip 0.0.0.0
about to fork child process, waiting until server is ready for connections.
forked process: 9912
child process started successfully, parent exiting
```

fork：表示以后台daemon方式运行 <br> 
bind_ip：表示对外服务IP，0.0.0.0 表示任何IP都可以连接Mongo服务 <br> 

验证：
```bash
# ps -ef | grep mongo
```

登录：
```bash
mongo
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```


## 术语概念

* database

数据库，类比 MySQL 中的 Database，保存的是有组织的数据的容器，一个mongodb可以创建多个数据库，每个数据库都有自己的集合和权限，不同的数据库存储在不同的文件里。 

* collection

集合，类比就是 MySQL 中的表，集合存在于数据库（database）中，mongodb 集合于关系型数据表不同的是，mongodb 在同一个集合中可以插入不同格式和类型的数据，这在关系型数据库中就不可以。<br> 

* document
文档，类比 MySQL中的行记录，mongodb 写入文档之前不需要设置字段（类似MySQL的表结构），而关系型数据库就需要事先定义好表结构。<br> 

* field

数据字段，类比MySQL中的列字段。

* index

索引，类比MySQL中的索引。

* primary key

主键，类比MySQL中的主键。

## 连接

### 无密码连接方式连接 mongo

直接输入 `mongo`


### 有密码方式连接mongo

先试用 `mongo` 连接：
```bash
# mongo
...
---
> use admin
switched to db admin
>
```

创建用户：
```mongo
> db.createUser({user:"root",pwd:"123",roles:[{role:"root",db:"admin"}]})
Successfully added user: {
	"user" : "root",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}
>
```

通过 role 指定 mongodb 的内置角色，也可以指定自定义角色。
role 为 root 表示超级权限。

退出并关闭已有MongoDB服务：
```bash
kill [mongo process id]
```

重启启动MongoDB服务：
```bash
# mongod --dbpath /data/mongodb/db --logpath /data/mongodb/log/mongod.log --fork --auth --bind_ip 0.0.0.0
about to fork child process, waiting until server is ready for connections.
forked process: 10492
child process started successfully, parent exiting
```

较之以前启动方式多了个 `--auth` 参数开启验证模式。

登录：
```bash
# mongo -uroot -p
MongoDB shell version v5.0.27
Enter password:
...
---
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
 

### Database 常见操作

* 创建和使用 database

```mongo
> use testdb
switched to db testdb
>
```

* 查看database

```mongo
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```

发现并没有刚刚创建的 `testdb` 库，如果要展示需要往 `testdb` 库中写入一些数据。

```mongo
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> use testdb
switched to db testdb
> db.test.insert({"num":1})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
>
```

* 删除 database

```mongo
> use testdb
switched to db testdb
> db.dropDatabase()
{ "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```


### mongodb 用户管理

* 创建管理用户并赋权

```mongo
use admin
db.createUser({user:"root",pwd:"123",roles:[{role:"root",db:"admin"}]})
```

与上面的使用授权登录时创建用户一致。


* 创建普通用户并赋权

```mongo
> use testdb
switched to db testdb
> db.createUser({user:"jack", pwd:"123",roles:[{role:"readWrite",db:"testdb"}]})
Successfully added user: {
	"user" : "jack",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "testdb"
		}
	]
}
>
```

readWrite: 具有读写权限。


* 查看用户

```mongo
> use admin
switched to db admin
> show users
{
	"_id" : "admin.root",
	"userId" : UUID("82f3f3ab-145a-4048-a2f0-e45b826da3ad"),
	"user" : "root",
	"db" : "admin",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
>
```

上面命令可以看到 admin 库下的用户，查看testdb库下的用户：
```mongo
> use testdb
switched to db testdb
> show users
{
	"_id" : "testdb.jack",
	"userId" : UUID("285ee755-fb97-4b06-a2f4-58f62bdd45a4"),
	"user" : "jack",
	"db" : "testdb",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "testdb"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
>
```

还有其他方式查询所有用户：
```mongo
> use admin
switched to db admin
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("82f3f3ab-145a-4048-a2f0-e45b826da3ad"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "0SMXP3duCSBHeyvbrZQnHQ==", "storedKey" : "gRu7euWPdX5hR8LwHMhP0tKbhpg=", "serverKey" : "o8YRFc5/OgvsUhuYYKyRPmaCoAY=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "7fvjM0tG9i+J5JsoZosPs+t7kldSiKvUSDDjAg==", "storedKey" : "BqQrez37A+86GgH+kahkF8WF3gBXhwGgpgRG2ygqjp8=", "serverKey" : "Eq34qWVvwxiOFmHThzQ4Io2QsvgByDCEUzHWBBRWpjU=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
{ "_id" : "testdb.jack", "userId" : UUID("285ee755-fb97-4b06-a2f4-58f62bdd45a4"), "user" : "jack", "db" : "testdb", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "e1R2K9zwRG7h0pCexsumMQ==", "storedKey" : "oO5sT+KeI8KH3gWfDhro8eFyVGQ=", "serverKey" : "8i17FU9vh+YOuBOPSEK0nk06Eco=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "sVjsyNXvz9anCOMOOpYg71D4BCOtAMwdIRBwIQ==", "storedKey" : "RET9ENt5Qfojrwq7YtieAReosgBXSeJUvKSKNOERPj4=", "serverKey" : "BQVDVzy0V2YRqWkYE1dfI5s/UoJMkxFyvgJjNpScfjE=" } }, "roles" : [ { "role" : "readWrite", "db" : "testdb" } ] }
>
```

* 删除用户

```mongo
> use testdb
switched to db testdb
> db.dropUser("jack")
true
> use admin
switched to db admin
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("82f3f3ab-145a-4048-a2f0-e45b826da3ad"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "0SMXP3duCSBHeyvbrZQnHQ==", "storedKey" : "gRu7euWPdX5hR8LwHMhP0tKbhpg=", "serverKey" : "o8YRFc5/OgvsUhuYYKyRPmaCoAY=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "7fvjM0tG9i+J5JsoZosPs+t7kldSiKvUSDDjAg==", "storedKey" : "BqQrez37A+86GgH+kahkF8WF3gBXhwGgpgRG2ygqjp8=", "serverKey" : "Eq34qWVvwxiOFmHThzQ4Io2QsvgByDCEUzHWBBRWpjU=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
>
```


* 常用内置角色

● root 超级角色
● readWrite 对某个非系统库的读写权限
● read 对某个非系统库的读权限
● readWriteAnyDatabase 对所有数据的读写权限
● readAnyDatabase 对所有数据的读权限


## CRUD 常见操作

* 创建集合
 
```mongo
> use testdb
switched to db testdb
> db.createCollection("animals")
{ "ok" : 1 }
>
```


* 查看库表集合

```mongo
> show tables
animals
> show collections
animals
>
```


* 删除集合

```mongo
>  db.animals.drop()
true
> show tables
>
```


* 插入文档

```mongo
>  db.students.save({name:"zhangsan"})
WriteResult({ "nInserted" : 1 })
> db.students.insert({name:"lisi"})
WriteResult({ "nInserted" : 1 })
> db.students.insert({name:"wangwu",age:20})
WriteResult({ "nInserted" : 1 })
>

```

* 插入多行

```mongo
> db.students.insertMany([{name:"zhao", age:16},{name:"qian", age:14}, {name:"sun", age:28}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("666d36f8171c254b653426a0"),
		ObjectId("666d36f8171c254b653426a1"),
		ObjectId("666d36f8171c254b653426a2")
	]
}
>
```

* 循环插入多行

```mongo
> for (var i=1; i<=100; i++) db.books.insert({num:i, name:'a'})
WriteResult({ "nInserted" : 1 })
>
```


* 查询文档

```mongo
> use testdb
switched to db testdb
> db.students.find()
{ "_id" : ObjectId("666d35fe171c254b6534269d"), "name" : "zhagnsan" }
{ "_id" : ObjectId("666d361e171c254b6534269e"), "name" : "lisi" }
{ "_id" : ObjectId("666d366d171c254b6534269f"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("666d36f8171c254b653426a0"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("666d36f8171c254b653426a1"), "name" : "qian", "age" : 14 }
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
>
> db.books.find()
{ "_id" : ObjectId("666d3738171c254b653426a3"), "num" : 1, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a4"), "num" : 2, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a5"), "num" : 3, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a6"), "num" : 4, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a7"), "num" : 5, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a8"), "num" : 6, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426a9"), "num" : 7, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426aa"), "num" : 8, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426ab"), "num" : 9, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426ac"), "num" : 10, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426ad"), "num" : 11, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426ae"), "num" : 12, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426af"), "num" : 13, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b0"), "num" : 14, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b1"), "num" : 15, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b2"), "num" : 16, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b3"), "num" : 17, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b4"), "num" : 18, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b5"), "num" : 19, "name" : "a" }
{ "_id" : ObjectId("666d3738171c254b653426b6"), "num" : 20, "name" : "a" }
Type "it" for more
>
```

输入 `it` 分页展示。


* 带条件查询

```mongo
> db.students.find({name:"lisi"})
{ "_id" : ObjectId("666d361e171c254b6534269e"), "name" : "lisi" }
>
```


* 更新文档

```mongo
> db.students.find({name:"zhangsan"}) 
{ "_id" : ObjectId("666d35fe171c254b6534269d"), "name" : "zhangsan" }
>
> db.students.update({name:"zhangsan"}, {$set:{name:"zhangsansan"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.students.find({name:"zhangsansan"})
>
{ "_id" : ObjectId("666d35fe171c254b6534269d"), "name" : "zhangsansan" }
>
```
db.one.update({name:"zhangsan"},{$set:{name:"zhangsansan"}})

* 删除文档

按条件删除：
> db.students.remove({name:"zhangsansan"})
WriteResult({ "nRemoved" : 1 })
> db.students.find({name:"zhangsansan"})
>
```

* 删除集合所有数据
```mongo
> db.books.remove({})
WriteResult({ "nRemoved" : 100 })
> db.books.find()
>
```

* 条件操作符&条件查询

查询 age 大于 20 的学生：
```mongo
>  db.students.find({age: {$gt: 20}})
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
>
```

查询 age 小于 20 的学生：
```mongo
>  db.students.find({age: {$gt: 20}})
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
> db.students.find({age: {$lt: 20}})
{ "_id" : ObjectId("666d36f8171c254b653426a0"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("666d36f8171c254b653426a1"), "name" : "qian", "age" : 14 }
>
```

多条件查询：
```mongo
> db.students.find({$and:[{name:"sun"},{age:28}]})
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
> db.students.find({$or:[{name:"sun"},{name:"qian"}]})
{ "_id" : ObjectId("666d36f8171c254b653426a1"), "name" : "qian", "age" : 14 }
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
>
```

* 对结果排序

1 表示升序，-1 表示降序：
```mongo
> db.students.find().sort({"age": 1})
{ "_id" : ObjectId("666d361e171c254b6534269e"), "name" : "lisi" }
{ "_id" : ObjectId("666d36f8171c254b653426a1"), "name" : "qian", "age" : 14 }
{ "_id" : ObjectId("666d36f8171c254b653426a0"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("666d366d171c254b6534269f"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
>
```

对带条件的结果排列：
```mongo
> db.students.find({age: {$gt: 16}}).sort({"age": 1})
{ "_id" : ObjectId("666d366d171c254b6534269f"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
>
```

降序：
```mongo
> db.students.find().sort({"age": -1})
{ "_id" : ObjectId("666d36f8171c254b653426a2"), "name" : "sun", "age" : 28 }
{ "_id" : ObjectId("666d366d171c254b6534269f"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("666d36f8171c254b653426a0"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("666d36f8171c254b653426a1"), "name" : "qian", "age" : 14 }
{ "_id" : ObjectId("666d361e171c254b6534269e"), "name" : "lisi" }
>
```

* 求和

统计集合文档总数：
```mongo
> db.students.count()
5
> db.students.find({age: {$gt: 20}}).count()
1
>
```

* 字段求和

先准备一些测试数据
```mongo
> db.students.insertMany([{name:"jack", class: 1, score:89},{name:"tom", class:1, score:92},{name:"peter", class:2,score:94},{name:"ben", class:2,score:88}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("666d3e65171c254b65342707"),
		ObjectId("666d3e65171c254b65342708"),
		ObjectId("666d3e65171c254b65342709"),
		ObjectId("666d3e65171c254b6534270a")
	]
}
>
```

查询每个班级的总分：
```mongo
> db.students.aggregate([{'$group':{_id:'$class', 'sum_score':{'$sum': '$score'}}}])
{ "_id" : 2, "sum_score" : 182 }
{ "_id" : null, "sum_score" : 0 }
{ "_id" : 1, "sum_score" : 181 }
>
```

aggregate 使用：

```mongo
> db.collection_name.aggregate([{
    '$group': {
        '_id': '$group_field',
        'avg_xxx': {
            '$avg': '$avg_field'
        }
    }
}])
```

group_field：分组字段 <br>
avg_xxx：聚合之后的字段名 <br>
$avg：表示求平均值
$avg_field：表示要聚合的字段

求平均值：
```mongo
> db.students.aggregate([{'$group': {_id: '$class', 'avg_score': {'$avg': '$score'}}}])
{ "_id" : 2, "avg_score" : 91 }
{ "_id" : 1, "avg_score" : 90.5 }
{ "_id" : null, "avg_score" : null }
>
```

分组后最大值：
```mongo
> db.students.aggregate([{'$group': {_id: '$class', 'max_score': {'$max': '$score'}}}])
{ "_id" : 2, "max_score" : 94 }
{ "_id" : 1, "max_score" : 92 }
{ "_id" : null, "max_score" : null }
>
```

分组后最小值：
```mongo
> db.students.aggregate([{'$group': {_id: '$class', 'min_score': {'$min': '$score'}}}])
{ "_id" : 2, "min_score" : 88 }
{ "_id" : 1, "min_score" : 89 }
{ "_id" : null, "min_score" : null }
>
```

## 索引

如果数据量很大，使用索引会有效提高数据的检索速度。

MongoDB 存储的是文档，MongoDB 是存储文档的非关系型数据库

准备数据：
```mongo
> for (var i=1; i <= 10000; i++) db.books.insert({num: i, name: 'a'})
```

查询前10条记录：
```mongo
> db.books.find().limit(10)
```

添加索引：
```mongo
> db.books.createIndex({"num":1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
```

num: 表示要添加的字段名
1: 表示升序索引，如果创建降序索引则设置值为-1

查询集合索引：
```mongo
> db.books.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"num" : 1
		},
		"name" : "num_1"
	}
]
>
```

删除索引：
```mongo
> db.books.dropIndex("num_1")
{ "nIndexesWas" : 2, "ok" : 1 }
> db.books.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
>
```

### 索引类型

* 单键索引
* 组合索引

```mongo
> db.books.createIndex({"num":1, "name": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
> db.books.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"num" : 1,
			"name" : 1
		},
		"name" : "num_1_name_1"
	}
]
>
```

* 多值索引(数组索引)

准备测试集合和测试数据：
```mongo
> db.fruits.insert({name: "apple", "tag": ["a", "b", "c"]})
```

创建索引：
```mongo
> db.fruits.createIndex({"tag": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

查询数据库是如果tag的值是 "a", "b", "c" 里面的就会使用到索引，如：
```mongo
> db.fruits.find({"tag": "b"})
{ "_id" : ObjectId("666e52e872940df15c43e056"), "name" : "apple", "tag" : [ "a", "b", "c" ] }
>
```

* 全文索引

```mongo
> db.fruits.createIndex({"name": text})
```

一个集合最多创建一个全文索引。

* 哈希索引

```mongo
> db.fruits.createIndex({name: "hashed"})
```

* 地理位置索引

```mongo
> db.location.insertMany([{loc:[9, 9]}, {loc:[11, 11]}, {loc: [100, 100]}])
> db.location.createIndex({"loc": "2d"})
> db.location.find({loc: {$geoWithin: {$box: [[10, 10], [12, 12]]}}})
```

## 查询计划

查看 MongoDB 的查询语句是否按希望的执行，如是否使用索引，就可以通过查询计划查看。

使用方式就是在查询语句之后使用 `explain` 关键字。

```mongo
> for (var i=1; i <= 10000; i++) db.books2.insert({num: i, name: 'a'})
> db.books2.find({num: 99}).explain()
```

explain 有三种不同的形式：

* queryPlanner 查询优化器选择查询的详细信息，这也是默认的，explain 不指定则默认为 queryPlanner
* executionStat 表示详细查询计划
* allPlaninExecution 最完善的执行计划，也可以使用 true 来代替

以 allPlaninExecution 为例，查看执行计划主要关注一下几个字段：
```mongo
> db.books.find({num: 99}).explain(true)
> db.books2.find({num: 99}).explain(true)
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "testdb.books2",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"num" : {
				"$eq" : 99
			}
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"num" : {
					"$eq" : 99
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
		"executionTimeMillis" : 10,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 10000,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"num" : {
					"$eq" : 99
				}
			},
			"nReturned" : 1,
			"executionTimeMillisEstimate" : 1,
			"works" : 10002,
			"advanced" : 1,
			"needTime" : 10000,
			"needYield" : 0,
			"saveState" : 10,
			"restoreState" : 10,
			"isEOF" : 1,
			"direction" : "forward",
			"docsExamined" : 10000
		},
		"allPlansExecution" : [ ]
	},
	"command" : {
		"find" : "books2",
		"filter" : {
			"num" : 99
		},
		"$db" : "testdb"
	},
	"serverInfo" : {
		"host" : "localhost.localdomain",
		"port" : 27017,
		"version" : "5.0.27",
		"gitVersion" : "49571988f1fea870e803f71a3ef8417173f3fbb1"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
>
```

* nReturned： 表示返回的行数
* executionTimeMillis：表示花费的时间，单位是毫秒
* totalKeysExamined：扫描了多少索引项
* totalDocsExamined：表示扫描了多少文档，通常与 nReturned 比值进行对，如果越大，就表示执行计划非常差，比如上面的扫描了10000个文档才返回1条记录，我们就要考虑添加合适的索引。

executionStages 下 stage 的值 “COLLSCAN” 表示全集合扫描，“INDEXSCAN” 表示索引扫描。从这个执行计划中可以看到这个查询非常慢，需要考虑添加索引来加速查询

```mongo
> db.books2.createIndex({"num": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

再次查看执行计划：
```mongo
> db.books2.find({num: 99}).explain(true)
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "testdb.books2",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"num" : {
				"$eq" : 99
			}
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"num" : 1
				},
				"indexName" : "num_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"num" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"num" : [
						"[99.0, 99.0]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 1,
		"executionTimeMillis" : 1,
		"totalKeysExamined" : 1,
		"totalDocsExamined" : 1,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 1,
			"executionTimeMillisEstimate" : 0,
			"works" : 2,
			"advanced" : 1,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"docsExamined" : 1,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 1,
				"executionTimeMillisEstimate" : 0,
				"works" : 2,
				"advanced" : 1,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"keyPattern" : {
					"num" : 1
				},
				"indexName" : "num_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"num" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"num" : [
						"[99.0, 99.0]"
					]
				},
				"keysExamined" : 1,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0
			}
		},
		"allPlansExecution" : [ ]
	},
	"command" : {
		"find" : "books2",
		"filter" : {
			"num" : 99
		},
		"$db" : "testdb"
	},
	"serverInfo" : {
		"host" : "localhost.localdomain",
		"port" : 27017,
		"version" : "5.0.27",
		"gitVersion" : "49571988f1fea870e803f71a3ef8417173f3fbb1"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
>
```

发现 totalDocsExamined 从之前的10000变成了1，说明扫描的文档数是1；
stage 的值为 FETCH ，表示是根据索引检索指定的文档；
executionTimeMillis 执行时间从之前的10毫秒变成了1毫秒。

```mongo
> db.books2.find({num: 99})
{ "_id" : ObjectId("666e60cbc7fb19d88d1e56fa"), "num" : 99, "name" : "a" }
>
```

## 慢查询

查看慢查询是否开启：
```mongo
> db.getProfilingLevel()
0
>
```

0 表示未开启，1表示已开启。

开启慢查询：
```mongo
> db.setProfilingLevel(1, 100)
{ "was" : 0, "slowms" : 100, "sampleRate" : 1, "ok" : 1 }
> db.getProfilingLevel()
1
>
```

1 表示开启慢查询，100 表示执行时间超过100毫秒的语句就认为属于慢查询，并记录到系统中。

关闭慢查询：
```mongo
> db.setProfilingLevel(0)
```

实践查询慢查询：
```mongo
> for (var i=1; i<= 300000; i++) db.books3.insert({num:i, name: 'c'})
WriteResult({ "nInserted" : 1 })
> db.setProfilingLevel(1, 100)
{ "was" : 1, "slowms" : 100, "sampleRate" : 1, "ok" : 1 }
> db.books3.find({"num": 9999})
{ "_id" : ObjectId("666e6472c7fb19d88d1ea4b6"), "num" : 9999, "name" : "c" }
> db.system.profile.find().limit(1)
{ "op" : "query", "ns" : "testdb.books3", "command" : { "find" : "books3", "filter" : { "num" : 9999 }, "lsid" : { "id" : UUID("ed7f5da4-3e45-42dd-be0a-dda2ff20169c") }, "$db" : "testdb" }, "keysExamined" : 0, "docsExamined" : 300000, "cursorExhausted" : true, "numYield" : 300, "nreturned" : 1, "queryHash" : "35269713", "planCacheKey" : "6BCA4864", "locks" : { "FeatureCompatibilityVersion" : { "acquireCount" : { "r" : NumberLong(302) } }, "Global" : { "acquireCount" : { "r" : NumberLong(302) } }, "Mutex" : { "acquireCount" : { "r" : NumberLong(1) } } }, "flowControl" : {  }, "storage" : { "data" : { "bytesRead" : NumberLong(3467214), "timeReadingMicros" : NumberLong(4505) } }, "responseLength" : 152, "protocol" : "op_msg", "millis" : 243, "planSummary" : "COLLSCAN", "execStats" : { "stage" : "COLLSCAN", "filter" : { "num" : { "$eq" : 9999 } }, "nReturned" : 1, "executionTimeMillisEstimate" : 29, "works" : 300001, "advanced" : 1, "needTime" : 300000, "needYield" : 0, "saveState" : 300, "restoreState" : 300, "isEOF" : 1, "direction" : "forward", "docsExamined" : 300000 }, "ts" : ISODate("2024-06-16T04:10:00.692Z"), "client" : "127.0.0.1", "appName" : "MongoDB Shell", "allUsers" : [ { "user" : "root", "db" : "admin" } ], "user" : "root@admin" }
>
```

command：表示执行的命令
keysExamined：表示索引扫描数
docsExamined：表示扫描文档数，这里扫描了
COLLSCAN：表示全集合扫描
client：表示客户端信息，可以定位到慢查询来源。

## MongoDB 压测

安装java：
```bash
yum install -y java-devel
```

安装 maven：
```bash
wget http://ftp.heanet.ie/mirrors/www.apache.org/dist/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz

wget https://dlcdn.apache.org/maven/maven-3/3.9.7/binaries/apache-maven-3.9.7-bin.tar.gz
```

## MongoDB 副本集

### 架构

![alt text](image-4.png)

![](/img/mongodb/replica/190.png)

常见的架构如上图，Secondary 节点持续复制 Primary 节点的数据，Arbiter 节点负责心跳检测和选举，该节点不接收数据。

### 副本集复制原理

当Primary 节点有修改操作时，会将变更记录在oplog中，有一个线程会持续监听 oplog 的变化，当有变化时会将变化传到 Secondary从节点，然后在从节点回放。

### 故障恢复原理

副本集中节点两两互相发送心跳，当超过5次未收到对方心跳时，就会认为对方失联，如果失联的是主节点，那么从节点会发起选举，选举出新的主节点，如果失联的是从节点，则不会发起新的选举，选举时基于raft一致性算法

### 副本集部署

1、准备3个节点
下载、解压

2、三台机器节点分别创建数据目录
```bash
mkdir -p /data/{mongodb27001/data/configdb, mongodb27001/data/sharddb,mongodb27001/conf,mongodb27001/run,mongodb27001/logs}
```

3、三台机器分别设置环境变量
```bash
export PARTH=$PATH:/opt/mongodb-xxx/bin/
```

4、修改安装配置文件
```bash
vim /data/mongodb27001/conf/mongod.conf
systemLog:
  destination: file # 日志形式
  logAppend: true # 日志追加方式
  path: /data/mongodb27001/logs/mongod.log # 日志目录
storage:
  dbPath: /data/mongodb27001/data
processManagement:
  fork: true # true表示以后台方式运行
  pidFilePath: /data/mongodb27001/run/mongod.pid
net:
  port: 27001
  bindIp: 0.0.0.0 # 所有机器都可以访问MongoDB
replication:
  replSetName: testdb_repl # 副本集名称
```

5、分别启动
```bash
 mongod -f /data/mongodb27001/conf/mongd.conf
```

6、创建集群
第一台机器：
```bash
mongo --port 27001
```

添加集群配置：
```mongo
config={
    _id: 'testdb_repl', 
    members: [
        {_id: 0, host: '192.168.152.30:27001', priority: 1}, 
        {_id: 1, host: '192.168.152.31:27001', priority: 1}, 
        {_id: 2, host: '192.168.152.32:27001', arbiterOnly: true}, 
    ]
}
```

初始化配置：
```mongo
rs.initiate(config)
```

继续回车看到第一个节点就是Primary 节点.


查看集群状态：
```moingo
rs.status()
```

查看同步：
进入Secondary
```mongo
use testdb

rs.secondaryOk()
db.one.Find()

```

进入primary 写入数据
```mongo
db.one.save({name: "abc"})
db.one.find()
```

sed查询
```mongo
db.one.find()
```

## 分片集群

### 使用场景

* 数据量过大，单机备份和恢复耗时长
* 副本集已无法满足高并发写入的性能要求

### 特点

* 对应用透明，应用通过分片方式连接到分片集群，与普通单节点连接一样
* 数据自动平衡，不会出现时间长数据在各分片不均匀情况
* 支持动态扩容，扩容期间MongoDB可继续提供服务，不必停机

### 架构

![](/img/mongodb/sharding/30.png)

Shard 用来存储数据，为了提高高可用和数据一致性，一个分片就是一个副本集，比如图中的 shard1 和shard2，分片之间的数据是不重复的。

Router（mongos）节点与客户端连接，并将操作定向到一个或多个分片中，生产环境可以配置多个 mongos 来实现高可用和负载均衡

Config Server 节点用来存储集群的元数据，包括集群数据集到分片的映射。Router 根据元数据将操作定向到对应的分片中。


### 部署

架构：

1、在三台机器上各启动一个 config server，它们的端口都是27020，这些 config server 会配置为一个副本集。

2、分别在3台机器上启动 27001 和 27002 端口，三台机器上的 27001 端口对应的第一个副本集，作为第一个分片，三台机器上的 27002 端口对应的第二个副本集，作为第二个分片。

三台机器各自执行下面命令，不同的是 bing_ip 需要修改为各自机器的IP：
```bash
mkdir -p /data/mongodb/config
mongod --configsvr --replSet config --dbpath /data/mongodb/config --bind_ip localhost,192.168.152.30 --logpath /data/mongodb/config/mongod.log --port 27020 --fork
```

configsvr: 表示这个节点是配置服务器
replSet: 配置的集合名称
dbpath: 表示配置的是数据目录
bind_ip: 机器IP
logpath: 日志文件路径
fork: 以后台进程方式启动


连接其中一台机器，我连接的第一台机器：
创建 config server 的副本集
```bash
mongo --port 27020
> rs.initiate(
    {
        _id: "config",
        configsvr: true,
        members: [
            {_id: 0, host: "xxx.xxx.xx.30:27020"},
            {_id: 1, host: "xxx.xxx.xx.31:27020"},
            {_id: 2, host: "xxx.xxx.xx.33:27020"}
        ]
    }
)
回车
回车
```

查看副本集状态：
```mongo
> rs.status()
```

关注 members，会有一个 Primary 和两个 Secondary，health 都是1表示集群创建爱你成功

启动mongodb实例，这些用来存储数据的分片mongodb:
在一台机器上部署两个实例，我在第一台机器上执行：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02
mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,192.168.152.30 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,192.168.152.30 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

第二机器创建也不部署两个实例：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02
mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,192.168.152.31 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,192.168.152.31 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

第三机器创建也不部署两个实例：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02
mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,192.168.152.32 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,192.168.152.32 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

shardsvr: 表示该节点为分片节点，分片副本集
replSet: 副本集名称

查看每台机器mongodb启动情况：
每台机器
```bash
ps -ef | grep mongo
```

创建分片副本集：
第一个副本集机器
```bash
mongo --port 27001
> rs.initiate(
    {
        _id: "shardtest01",
        configsvr: true,
        members: [
            {_id: 0, host: "xxx.xxx.xx.30:27001"},
            {_id: 1, host: "xxx.xxx.xx.31:27001"},
            {_id: 2, host: "xxx.xxx.xx.33:27001"}
        ]
    }
)
> rs.status()
```

第二个副本集机器
```bash
mongo --port 27002
> rs.initiate(
    {
        _id: "shardtest02",
        configsvr: true,
        members: [
            {_id: 0, host: "xxx.xxx.xx.30:27002"},
            {_id: 1, host: "xxx.xxx.xx.31:27002"},
            {_id: 2, host: "xxx.xxx.xx.33:27002"}
        ]
    }
)
> rs.status()
```

启动mongos，随便选择一台机器启动mongos，生产环境一般建议在业务机器启动机客户端启动：
```bash
mkdir -p /data/mongodb/mongos
# 配置 config server 集群
mongod --configdb config/192.168.152.30:27020,192.168.152.31:27020,192.168.152.32:27020 --bind_ip localhost,192.168.152.30 --logpath /data/mongodb/mongos/mongos.log --port 27017 --fork
```

bind_ip 为对客户端开放的IP

将分片副本集添加到集群：
```mongo
mongo --port 27017
> sh.addShard("shardtest01/192.168.152.30:27001,192.168.152.31:27001,192.168.152.32:27001")
> sh.addShard("shardtest02/192.168.152.30:27002,192.168.152.31:27002,192.168.152.32:27002")
```

创建分片表：
```mongo
> mongo --port 27017
sh.enableSharding("testdb") # 为数据库启动分片
```

对集合创建分片规则：
```mongo
mongo --port 27017
# 分片字段为_id， 分片方法为哈希
sh.shardCollection("testdb.userinfo", {_id: "hashed"})
> sh.status()
```

注意如果userinfo drop重建就需要重建collection分片规则。

### 分片集群数据分布测试

```mongo
mongo --port 27017 testdb

> for (var i = 1; i <=10; i++) db.userinfo.save({userid: i, username: "aa"})

> db.userinfo.find()
```

查看两个分片的分片情况：
先登录第一个shard：
```mongo
mongo --port 27001 testdb
db.userinfo.find()
```

先登录第二个shard：
```mongo
mongo --port 27002 testdb
db.userinfo.find()
```

结论：
华为云创建的主从复制的mysql_



在数据库复制中，主数据库会生成二进制日志（binlog），并将其发送到从数据库（slave）以便同步数据。你看到的这句话意味着主数据库已经将所有的二进制日志发送给了从数据库，并且正在等待更多的更新

如果是来自本地连接（localhost），Host 字段会显示为 “localhost” 或 “127.0.0.1”，表示连接是从本机发起的。
如果是来自远程主机的连接，Host 字段会显示远程主机的IP地址或主机名，指示连接是从该远程主机发起的。

使用 SHOW PROCESSLIST 命令查看当前正在执行的线程，包括其状态和相关信息。对于 “Binlog Dump GTID” 线程，其 HOST 列显示了与该线程相关的主机地址。这通常表示的是连接到主服务器的从服务器的 IP 地址。

Binlog Dump GTID 线程通常是由从服务器发起的复制线程，这个线程请求主服务器发送二进制日志（binlog）以便进行数据同步。HOST 列的信息有助于管理员识别是哪个从服务器发起了这个请求。

Binlog Dump GTID 线程由从服务器发起的复制线程，这个线程的IP是docker的网关的IP10.20.30.254，当存在多个MySQL组的时候，主实例发现它们来自于同一个从服务器，服务器的地址是10.20.30.254，由于使用的是同一个IP，会认为是僵尸进程，会将其杀掉，然后重新启动。也就是出现issue描述中的复制一致在重建、杀死的循环。