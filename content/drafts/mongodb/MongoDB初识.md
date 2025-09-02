+++
title = 'MongoDB初识'
date = 2024-05-11T08:22:05+08:00
draft = true
categories = [ "MongoDB" ]
tags = [ "mongodb" ]
+++

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
<p style="text-align:center;font-size:1.3em">文档目录结构</p>

- [1 介绍](#1-介绍)
	- [1.1 什么是Meteor?](#11-什么是meteor)
	- [1.2 快速入门](#12-快速入门)
	- [1.3 Meteor资源](#13-meteor资源)
	- [1.4 什么是Meteor指南?](#14-什么是meteor指南)
	- [1.5 指南开发](#15-指南开发)
- [2 代码风格](#2-代码风格)
	- [2.1 风格一致的好处](#21-风格一致的好处)
	- [2.2 JavaScript风格指南](#22-javascript风格指南)
	- [2.3 用ESLint检查你的代码](#23-用eslint检查你的代码)
	- [2.4 Meteor编码风格](#24-meteor编码风格)
- [3 应用程序结构](#3-应用程序结构)
	- [3.1 通用JavaScript](#31-通用javascript)
	- [3.2 文件结构](#32-文件结构)
	- [3.3 默认的文件加载顺序](#33-默认的文件加载顺序)
	- [3.4 拆分成多个应用程序](#34-拆分成多个应用程序)
- [4 迁移到 Meteor 2.4](#4-迁移到-meteor-24)
	- [4.1 从2.3以前的版本迁移?](#41-从23以前的版本迁移)

<!-- markdown-toc end -->

# 1 MongoDB是什么

* 文档数据库
* NoSQL 类型数据库（非关系型数据库）
* C++ 编写的开源数据库系统

## 1.1 特点

* 面向文档。以 json 格式存储数据。
* 高性能。数据存储于内存，查询快，性能高。
* 高可用。自带高可用方案，如副本集、分片集群。
* 丰富的查询语句。
* 支持多种主流开发语言。如Java、Go。
* 可添加索引加快检索速度。

## 1.2 数据结构

```json
{
	name: "Lu",
	age: 23
}
```

## 1.3 架构

* 单实例：一台机器上只运行了一个MongoDB实例，不具备高可用。
* 副本集：一般一主一从，有一个仲裁节点，主节点挂掉之后从节点可以继续为主提供服务。
* 分片：在副本集的基础上将数据分散成多个分片。如数据量较大时可以使用分片。

## 1.4 适用场景

- 游戏
- 物流
- 社交
- 直播
- 物联网
- 电商

## 1.5 案例

- Keep
- 小红书
- 网易
- 京东
- 华为
- 纽约时报
- Elsevier
- FourSquare
- Bosch

## 1.6 技术场景

- 单表数据量上亿
- 数据结构模型多变
- 使用到地里位置功能
- 业务可能出现爆发增长
- 高可用

## 1.7 安装部署

参考：`《MongoDB常见架构搭建》`

# 2 基本概念

**1. database**

数据库，类比 MySQL 中的 Database，是保存有组织数据的容器。
	
- 一个mongodb可以创建多个数据库。
- 每个数据库都有自己的集合和权限，不同的数据库存储在不同的文件里。 

**2. collection**

集合，类比就是 MySQL 中的表。

- 集合存在于数据库（database）中。
- mongodb 在同一个集合中可以插入不同格式和类型的数据，甚至同一集合中不同行数据、字段数据都可以不同，这在关系型数据库中的表就不可以。
- 集合不能包含空字符。
- 集合不能以 `system.` 开头。

**3. document**

文档，类比 MySQL 中的行记录。

- mongodb 写入文档之前不需要设置字段（类似MySQL的表结构），而关系型数据库就需要事先定义好表结构。
- 同一行文档里不能有重复的 key，并且区分大小写。
- 文档中的键值对是有序的。

**4. field**

数据字段，类比MySQL中的列字段。

**5. index**

索引，类比 MySQL 中的索引。
	
- 常用作条件的字段可添加索引大大提高查询速度。

**6. primary key**

主键，类比 MySQL 中的主键。

**小结**

| 术语                   | MongoDB			      | MySQL   				 |
|:----------------------|:-----------------------|:-------------------------|
| 数据库 				 | database               | database                 |
| 集合     		         | collection             | table                    |
| 文档      			 | document               | row                      |
| 字段        			 | field                  | column                   |
| 索引        			 | index                  | index                    |
| 主键       			 | primary key            | primary key              |


# 3 操作

## 1.1 连接

**（一）无密码连接方式连接**

直接输入 `mongo` 即可：
```bash
# mongo
MongoDB shell version v5.0.28
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("df6aa60a-6a68-4ec8-a0d9-7d563804e420") }
MongoDB server version: 5.0.28
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
	https://community.mongodb.com
---
The server generated these startup warnings when booting:
        2024-09-22T23:58:33.139+08:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
        2024-09-22T23:58:33.139+08:00: You are running this process as the root user, which is not recommended
        2024-09-22T23:58:33.139+08:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never' in this binary version
        2024-09-22T23:58:33.139+08:00: /sys/kernel/mm/transparent_hugepage/defrag is 'always'. We suggest setting it to 'never' in this binary version
---
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```

**（二）有密码方式连接**

1、先使用 `mongo` 连接登录：
```bash
# mongo
...
---
> use admin
switched to db admin
>
```

2、创建用户：
```bash
> db.createUser({user: "root", pwd: "123", roles: [{role: "root", db: "admin"}]})
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

- **role**：指定 mongodb 的内置角色，也可以指定自定义角色。role 设置为 `root` 表示超级权限。

3、停止MongoDB服务
```bash
kill [mongo process id]
```

4、重启启动MongoDB服务
```bash
mongod --dbpath /data/mongodb/db --logpath /data/mongodb/log/mongod.log --fork --auth --bind_ip 0.0.0.0
```

较之以前启动方式多了个 `--auth` 参数开启验证模式。

输出如下：
```bash
about to fork child process, waiting until server is ready for connections.
forked process: 2542
child process started successfully, parent exiting
```

5、登录
```bash
# mongo -uroot -p
MongoDB shell version v5.0.28
Enter password:
...
---
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
 
## 1.2 Database操作

**创建和切换database**
```bash
> use testdb
switched to db testdb
>
```

**查看database**
```bash
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```

发现只有3个系统库，没有刚刚创建的 `testdb` 库，如果要展示，则需要往 `testdb` 库中写入一些数据：
```bash
> db.test.insert({age: 10})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
>
```

**删除database**
```bash
> use testdb
switched to db testdb
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
> db.dropDatabase()
{ "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>
```

## 1.3 用户管理

**创建管理用户并赋权**
```bash
use admin
db.createUser({user: "root", pwd: "123", roles: [{role: "root", db: "admin"}]})
```

**创建普通用户并赋权**
```bash
> use testdb
switched to db testdb
> db.createUser({user: "user_lisi", pwd: "123", roles: [{role: "readWrite", db: "testdb"}]})
Successfully added user: {
	"user" : "user_lisi",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "testdb"
		}
	]
}
>
```

- **readWrite**: 具有读写权限。


**查看用户**
```bash
> use admin
switched to db admin
> show users
{
	"_id" : "admin.root",
	"userId" : UUID("b6e24fa0-b869-48cc-87fb-d99a7fb65795"),
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

`show users` 只能查看当前库的用户。上面命令可以看到 admin 库下的用户，查看testdb库下的用户：
```bash
> use testdb
switched to db testdb
> show users
{
	"_id" : "testdb.user_lisi",
	"userId" : UUID("f25bf4c6-23d2-481e-a72d-ad4e9451a8ce"),
	"user" : "user_lisi",
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

其他方式查询所有用户：
```bash
> use admin
switched to db admin
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("b6e24fa0-b869-48cc-87fb-d99a7fb65795"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "kSA5ue4nXTQIoN4SlhiSuw==", "storedKey" : "LSRskkw2XH2j0/ftCOyNmaqa87Q=", "serverKey" : "mWSt9PDxX5NBTddoNRGJP6RoajQ=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "3D5su0krLfZXeKOBZHyJMamQebxzzDM/rOyKBQ==", "storedKey" : "bTTvNbs+XE3N98AxmUyyr4dKGemG9ffMXBLJ5IU9Kx4=", "serverKey" : "gGE2biwK5GiKJ3yzhw6vQfab91/bcPfSYGO/08U5l60=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
{ "_id" : "testdb.user_lisi", "userId" : UUID("f25bf4c6-23d2-481e-a72d-ad4e9451a8ce"), "user" : "user_lisi", "db" : "testdb", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "Fh/ptSG0Bvg1VnoshuBMPA==", "storedKey" : "TK75nBsA+b+R3NZXjmeZvR1t5GA=", "serverKey" : "onMkHZYEpUgj5smf1Z82j+IBeUo=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "1vPMLeSd7+jP7o/ITZIuv25MxBbEYpWmBJE9hw==", "storedKey" : "LZhHbcFZ8YGFLDW3tfpjSTNYGn1Zf6Lqzs/uz9kjmi0=", "serverKey" : "TZJQwsD42SPYW25unkzUDR3DhV4m6Woy6FCdfAYKmig=" } }, "roles" : [ { "role" : "readWrite", "db" : "testdb" } ] }
>
```

**删除用户**

```bash
> use testdb
switched to db testdb
> db.dropUser("user_lisi")
true
> use admin
switched to db admin
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("b6e24fa0-b869-48cc-87fb-d99a7fb65795"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "kSA5ue4nXTQIoN4SlhiSuw==", "storedKey" : "LSRskkw2XH2j0/ftCOyNmaqa87Q=", "serverKey" : "mWSt9PDxX5NBTddoNRGJP6RoajQ=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "3D5su0krLfZXeKOBZHyJMamQebxzzDM/rOyKBQ==", "storedKey" : "bTTvNbs+XE3N98AxmUyyr4dKGemG9ffMXBLJ5IU9Kx4=", "serverKey" : "gGE2biwK5GiKJ3yzhw6vQfab91/bcPfSYGO/08U5l60=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
>
```

**常用内置角色**

- **root**：超级权限角色
- **readWrite**：对某个非系统库的读写权限
- **read**：对某个非系统库的读权限
- **readWriteAnyDatabase**：对所有数据库的读写权限
- **readAnyDatabase**：对所有数据库的读权限

## 1.4 CRUD

**创建集合**

先创建DB并切换至创建的DB，然后再创建集合。
```bash
> use testdb
switched to db testdb
> db.createCollection("animals")
{ "ok" : 1 }
>
```

**查看集合**
```bash
> show tables
animals
> show collections
animals
>
```

**删除集合**

```bash
>  db.animals.drop()
true
> show tables
>
```

**插入文档**

1、插入单个文档
```bash
> db.students.save({name: "zhangsan"})
WriteResult({ "nInserted" : 1 })
> db.students.insert({name: "lisi"})
WriteResult({ "nInserted" : 1 })
>
```

2、插入多个字段
```bash
> db.students.insert({name: "wangwu", age: 20})
WriteResult({ "nInserted" : 1 })
>
```

3、插入多行
```bash
> db.students.insertMany([{name: "zhao", age: 16}, {name: "qian", age: 24}, {name: "sun", aeg: 28}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("66f183d0aa126474784afbdc"),
		ObjectId("66f183d0aa126474784afbdd"),
		ObjectId("66f183d0aa126474784afbde")
	]
}
>
```

如果插入正常，则返回所有数据的id。


4、批量插入多行
```bash
> for (var i = 1; i <= 100; i++) db.books.insert({num: i, name: 'abc'})
WriteResult({ "nInserted" : 1 })
>
```


**查询文档**

1、查询集合 students 所有数据
```bash
> use testdb
switched to db testdb
> db.students.find()
{ "_id" : ObjectId("66f182baaa126474784afbd9"), "name" : "zhangsan" }
{ "_id" : ObjectId("66f182c9aa126474784afbda"), "name" : "lisi" }
{ "_id" : ObjectId("66f18329aa126474784afbdb"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("66f183d0aa126474784afbdc"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
>
```

2、查询批量插入集合books的数据
```bash
> db.books.find()
{ "_id" : ObjectId("66f1847daa126474784afbdf"), "num" : 1, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe0"), "num" : 2, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe1"), "num" : 3, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe2"), "num" : 4, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe3"), "num" : 5, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe4"), "num" : 6, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe5"), "num" : 7, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe6"), "num" : 8, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe7"), "num" : 9, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe8"), "num" : 10, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbe9"), "num" : 11, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbea"), "num" : 12, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbeb"), "num" : 13, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbec"), "num" : 14, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbed"), "num" : 15, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbee"), "num" : 16, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbef"), "num" : 17, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbf0"), "num" : 18, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbf1"), "num" : 19, "name" : "abc" }
{ "_id" : ObjectId("66f1847daa126474784afbf2"), "num" : 20, "name" : "abc" }
Type "it" for more
>
```

我插入了100条，但这里只默认展示了20条，如果展示下一页，可以根据最后的提示输入 `it`


**带条件查询**

查询 students 集合中 name 是 "lisi" 的数据
```bash
> db.students.find({name: "lisi"})
{ "_id" : ObjectId("66f182c9aa126474784afbda"), "name" : "lisi" }
>
```

**更新文档**

将 students 集合中 name 为 “zhangsan” 的记录改为 name 为 “zhangsansan”
```bash
> db.students.find({name: "zhangsan"})
{ "_id" : ObjectId("66f182baaa126474784afbd9"), "name" : "zhangsan" }
>
```

前面是匹配的条件，后面是要设置的值：
```bash
> db.students.update({name: "zhangsan"}, {$set: {name: "zhangsansan"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
>
```

查询验证：
```bash
> db.students.find({name: "zhangsansan"})
{ "_id" : ObjectId("66f182baaa126474784afbd9"), "name" : "zhangsansan" }
>
```

**删除文档**
1、按条件删除，删除集合 students 中 name 为 “zhangsansan” 的文档
```bash
> db.students.remove({name: "zhangsansan"})
WriteResult({ "nRemoved" : 1 })
>
```

2、删除集合所有数据
```bash
> db.books.remove({})
WriteResult({ "nRemoved" : 100 })
>
```

会返回删除的行数。

**条件操作符**

条件查询：

| 术语                   | 条件     		      |
|:----------------------|:-----------------------|
| 大于 				     | &gt                    |
| 小于 				     | &lt                    |
| 大于等于     		      | &gte                   |
| 小于等于     		      | &lte                   |

1、查询集合 students 中 age 大于 20 的学生：
```bash
> db.students.find({age: {$gt: 20}})
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
>
```

2、查询集合 students 中 age 小于 20 的学生
```bash
> db.students.find({age: {$lt: 20}})
{ "_id" : ObjectId("66f183d0aa126474784afbdc"), "name" : "zhao", "age" : 16 }
>
```

3、多条件查询

查询集合 students 中 name 为 “sun” ，age 为 28 的记录：
```bash
> db.students.find({name: "sun"}, {age: 28})
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "age" : 28 }
>
```

查询集合 students 中 name 为 “sun”，或者 name 为 “qian” 的记录：
```bash
> db.students.find({$or: [{name: "sun"}, {name: "qian"}]})
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
>
```

## 1.5 统计处理

**结果排序**

`1` 表示升序，`-1` 表示降序：

1、查询集合 students ，对age进行升级排列
```bash
> db.students.find().sort({"age": 1})
{ "_id" : ObjectId("66f182c9aa126474784afbda"), "name" : "lisi" }
{ "_id" : ObjectId("66f183d0aa126474784afbdc"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("66f18329aa126474784afbdb"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
>
```

2、对带条件的查询结果排列
```bash
> db.students.find({age: {$gt: 16}}).sort({"age": 1})
{ "_id" : ObjectId("66f18329aa126474784afbdb"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
>
```

3、降序
```bash
> db.students.find().sort({"age": -1})
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
{ "_id" : ObjectId("66f18329aa126474784afbdb"), "name" : "wangwu", "age" : 20 }
{ "_id" : ObjectId("66f183d0aa126474784afbdc"), "name" : "zhao", "age" : 16 }
{ "_id" : ObjectId("66f182c9aa126474784afbda"), "name" : "lisi" }
>
```

4、对查询 age 大于 20 的数据进行降序排序
```bash
> db.students.find({age: {$gt: 20}}).sort({"age": -1})
{ "_id" : ObjectId("66f183d0aa126474784afbde"), "name" : "sun", "age" : 28 }
{ "_id" : ObjectId("66f183d0aa126474784afbdd"), "name" : "qian", "age" : 24 }
>
```

**求和**

统计集合文档总数：
```bash
> db.students.count()
5
> db.students.find({age: {$gt: 20}}).count()
2
>
```

**字段求和**

1、提前准备一些测试数据
```bash
> db.students.insertMany([{name: "jack", class: 1, score: 89}, {name: "tom", class: 1, score: 92}, {name: "peter", class: 2, score: 94}, {name: "lee", class: 2, score: 88}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("66f198ded3b492bb3ff26d97"),
		ObjectId("66f198ded3b492bb3ff26d98"),
		ObjectId("66f198ded3b492bb3ff26d99"),
		ObjectId("66f198ded3b492bb3ff26d9a")
	]
}
>
```

2、查询每个班级的总分：
```bash
> db.students.aggregate([{'$group': {_id: '$class', 'sum_score': {'$sum': '$score'}}}])
{ "_id" : 1, "sum_score" : 181 }
{ "_id" : null, "sum_score" : 0 }
{ "_id" : 2, "sum_score" : 182 }
>
```

- `$group`: 分组
- `_id: '$class'`： 表示根据 class 字段进行分组
- `sum_score`：表示求和之后的字段
- `$sum`：表示求和，后面跟求和的字段，这里是对 score 字段求和。

aggregate 使用：

```bash
> db.collection_name.aggregate([{
    '$group': {
        '_id': '$group_field',
        'avg_xxx': {
            '$avg': '$avg_field'
        }
    }
}])
```

- $group_field：分组字段
- avg_xxx：聚合之后的字段名
- $avg：表示求平均值
- $avg_field：表示要聚合的字段

3、求平均值
```bash
> db.students.aggregate([{'$group': {_id: '$class', 'avg_score': {'$avg': '$score'}}}])
{ "_id" : 1, "avg_score" : 90.5 }
{ "_id" : null, "avg_score" : null }
{ "_id" : 2, "avg_score" : 91 }
>
```

4、分组后最大值
```bash
> db.students.aggregate([{'$group': {_id: '$class', 'max_score': {'$max': '$score'}}}])
{ "_id" : 1, "max_score" : 92 }
{ "_id" : 2, "max_score" : 94 }
{ "_id" : null, "max_score" : null }
>
```

5、分组后最小值
```bash
> db.students.aggregate([{'$group': {_id: '$class', 'min_score': {'$min': '$score'}}}])
{ "_id" : 1, "min_score" : 89 }
{ "_id" : 2, "min_score" : 88 }
{ "_id" : null, "min_score" : null }
>
```

## 1.6 索引

**单键索引**

- 单键索引：一个索引包含一个字段

索引就好比字段的目录，可以帮我们提高查找速度，如果数据量很大，使用索引会有效提高数据的检索速度。

1、准备数据
```bash
> for (var i = 1; i <= 10000; i++) db.books.insert({num: i, name: "abc"})
WriteResult({ "nInserted" : 1 })
>
```

2、查询前10条记录
```bash
> db.books.find().limit(10)
{ "_id" : ObjectId("66f1a071ac12297393191993"), "num" : 1, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191994"), "num" : 2, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191995"), "num" : 3, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191996"), "num" : 4, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191997"), "num" : 5, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191998"), "num" : 6, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac12297393191999"), "num" : 7, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac1229739319199a"), "num" : 8, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac1229739319199b"), "num" : 9, "name" : "abc" }
{ "_id" : ObjectId("66f1a071ac1229739319199c"), "num" : 10, "name" : "abc" }
>
```

3、在 num 字段上添加索引
```bash
> db.books.createIndex({"num": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

- num: 表示要添加的字段名
- 1: 表示升序索引，如果创建降序索引则设置值为-1

4、查询集合索引
```bash
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

5、删除索引
```bash
> db.books.dropIndex("num_1")
{ "nIndexesWas" : 2, "ok" : 1 }
>
> db.books.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
>
```

**索引类型**

- 组合索引：一个索引包含多个字段

1、使用 num 和 name 创建组合索引
```bash
> db.books.createIndex({"num": 1, "name": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
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

**多值索引(数组索引)**

1、准备测试集合和测试数据
```bash
> db.fruits.insert({name: "apple", "tag": ["a", "b", "c"]})
WriteResult({ "nInserted" : 1 })
>
```

2、创建索引
```bash
> db.fruits.createIndex({"tag": 1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

3、查询数据库是如果tag的值是 "a", "b", "c" 里面的就会使用到索引，如：
```bash
> db.fruits.find({"tag": "b"})
{ "_id" : ObjectId("66f1a380ac122973931940a3"), "name" : "apple", "tag" : [ "a", "b", "c" ] }
>
```

**全文索引**

在集合 fruits 的name创建了全文索引
```bash
> db.fruits.createIndex({"name": "text"})
{
	"numIndexesBefore" : 2,
	"numIndexesAfter" : 3,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

一个集合最多创建一个全文索引。

**哈希索引**
```bash
> db.fruits.createIndex({name: "hashed"})
{
	"numIndexesBefore" : 3,
	"numIndexesAfter" : 4,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
>
```

**地理位置索引**
```bash
> db.location.insertMany([{loc: [9, 9]}, {loc: [11, 11]}, {loc: [100, 100]}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("66f1a42eac122973931940a4"),
		ObjectId("66f1a42eac122973931940a5"),
		ObjectId("66f1a42eac122973931940a6")
	]
}
> db.location.createIndex({"loc": "2d"})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
> db.location.find({loc: {$geoWithin: {$box: [[10, 10], [12, 12]]}}})
{ "_id" : ObjectId("66f1a42eac122973931940a5"), "loc" : [ 11, 11 ] }
>
```

## 1.7 查询计划

有时候想知道 MongoDB 的查询语句是否按期望的执行，如是否命中索引，就可以通过查询计划查看。

使用方式：在查询语句之后使用 `explain` 关键字。

```bash
> for (var i=1; i <= 10000; i++) db.books.insert({num: i, name: 'abc'})
> db.books.find({num: 99}).explain()
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "testdb.books",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"num" : {
				"$eq" : 99
			}
		},
		"queryHash" : "35269713",
		"planCacheKey" : "941810BA",
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"num" : 1,
					"name" : 1
				},
				"indexName" : "num_1_name_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"num" : [ ],
					"name" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"num" : [
						"[99.0, 99.0]"
					],
					"name" : [
						"[MinKey, MaxKey]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"command" : {
		"find" : "books",
		"filter" : {
			"num" : 99
		},
		"$db" : "testdb"
	},
	"serverInfo" : {
		"host" : "chaos-1",
		"port" : 27017,
		"version" : "5.0.28",
		"gitVersion" : "a8f8b8e1e271f236e761d0138e2418d0a114c941"
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

explain 有三种不同的形式：

**queryPlanner**：查询优化器选择查询的详细信息，这也是默认形式，explain 不指定则默认为 `queryPlanner`

```bash
db.books.find({num: 99}).explain("queryPlanner")
```

**executionStats**：表示详细查询计划

```bash
db.books.find({num: 99}).explain("executionStats")
```

**allPlansExecution**：最完善的执行计划，也可以使用 true 来代替

```bash
db.books.find({num: 99}).explain("allPlansExecution")
db.books.find({num: 99}).explain(true)
```

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

## 1.8 慢查询

1、查看慢查询是否开启
```bash
> db.getProfilingLevel()
0
>
```

0 表示未开启，1表示已开启。

2、开启慢查询
```bash
> db.setProfilingLevel(1, 100)
{ "was" : 0, "slowms" : 100, "sampleRate" : 1, "ok" : 1 }
> db.getProfilingLevel()
1
>
```

1 表示开启慢查询，100 表示执行时间超过100毫秒的语句就认为属于慢查询，并记录到系统中。

3、关闭慢查询
```bash
> db.setProfilingLevel(0)
```

3、练习慢查询
```bash
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

**command**：表示执行的命令。
**keysExamined**：表示索引扫描数。
**docsExamined**：表示扫描文档数，这里扫描了。
**COLLSCAN**：表示全集合扫描。
**client**：表示客户端信息，可以定位到慢查询来源。

