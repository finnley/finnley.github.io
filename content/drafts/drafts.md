这些随之带来的就是各种各样的问题，我需要不断的去解决这些问题，这有让我看去像个活人。

看到这里，就当是我们的握手礼，我叫“陆知行”，我是另一个你，你的所有问题都来源我，你的抑郁、你的焦虑、你的自卑、你的种种问题都来来自我。“路知行”这个名字，其实很简单，来源于我在上海住的一条路上的地铁站，叫“行知路”，还别说上海有些地方的取名真的很好听。我接触的第一个上海土著也姓陆。

你没法知道她的情绪的变化，有她自己的节奏。这也是我一直想学的，能够控制住自己的情绪以及心态。可惜不知在哪一天被我弄丢了。尝试过很多种方式去寻找，却始终失败。


## 
##### 结构

一个 mongodb 数据库中包含多个集合，拥有相似内容的文档被归类到同个集合之下，而每个文档中包含各种字段以及这些字段对应的值
mongodb 支持的一种json扩展格式，称为 bson，简单总结就是 mongodb 可以存储 json

同一集合中的文档可以拥有完全不同的字段，比如可将描述银行账号的文档和客户的文档放到同一集合中

像mongodb非关系型数据中，并不像MySQL一样存在提前存在好的数据格式，如果我们需要再文档中增加一个新字段，只要直接将包含新字段的文档写进数据库中即可。

安装
docker 容器运行 MongoDB
# 下载 MongoDB 官方 docker 镜像
docker pull mongo:4

# 启动一个 MongoDB 服务器容器
docker run --name mymongo -v ~/mymongo/data:/data/db -d mongo:4

# 查看数据库服务器日志
docker logs mymongo
● 默认端口27017

Mongo Express 
一个基于网络的 MongoDB 数据库管理界面
# 下载 mongo-express 镜像
docker pull mongo-express

# 运行 mongo-express
docker run --link mymongo:mongo -p 8081:8081 mongo-express
mongo-express 访问 url 为：http://localhost:8081

Mongo Shell
一个用来操作 MongoDB  的 javascript 客户端界面
# 运行 mongo shell
docker exec -it mymongo mongo
mongo shell 可以接收 javascript 语法，比如：
> print('hello');
hello
> exit
bye

文档基本操作
● 创建 Create
● 读取 Read
● 更新 Update
● 删除 Delete 
文档主键 _id
● 文档主键具有唯一性
● 除数组外，所有mongodb支持的类型都可以作为文档主键
● 复合主键
● 对象主键 ObjectId
  ○ 默认文档主键
  ○ 可以快速生成的12字节id
  ○ 包含创建时间
创建文档
● db.collection.insert()
● db.collection.save()
● 创建多个文档
mongo shell 创建文档
# 使用 test 数据库
> use test
switched to db test

# 查看 test 数据库中的集合，如果返回空表示现在 test 数据库里还没有集合
> show collections
>

# 创建第一个文档
## 创建单个文档
## db.collection.insertOne()
db.<collection>.insertOne(
  <document>,
  {
    writeConcern: <document>
  }
)
## <collection> 要替换成文档将要写入的集合
## <document> 要替换成将要写入的文档本身
## writeConcern 文档定义了本次文档创建操作的安全写级别，安全写级别用来判断一次数据库写入操作是否成功，安全写级别越高，丢失数据的风险就越低，然而写入操作的延迟也可能更高
## 如果不提供 writeConcern 文档，mongodb 施一公默认的安全写级别

## 准备写入数据库的文档
{
  	_id: "account1",
  	name: "alice",
    balance: 100
}

## 将文档写入 accounts 集合
> db.accounts.insertOne({_id: "account1", name: "alice", balance: 100})
{ "acknowledged" : true, "insertedId" : "account1" }
>

## "acknowledged" : true 表示安全写级别被启用。由于我们在 db.collection.insertOne() 命令中并没有提供 writeConern 文档，这里显示的就是 mongodb 默认的安全写级别启用状态
## insertedId 显式了被写入的文档的 _id

## 查看集合列表
> show collections
accounts
>
## db.collection.insertOne() 命令会自动创建相应的集合


# 创建单个或多个文档
db.collection.insert()
db.<collection>.insert(
  <document or array of documents>,
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
)
db.collection.insert() 既可以写入一个单独的文档，也可以写入多个文档
> db.accounts.insert({name: "george", balance: 1000})
WriteResult({ "nInserted" : 1 })
>




## 文档基本操作
● 创建 Create
● 读取 Read
● 更新 Update
● 删除 Delete 


### 文档主键（_id）

存储在 MongoDB 中每个文档有个专属的id，称为文档主键，值存储在 `_id` 字段中。它是每个 MongoDB 文档必备字段。

● 文档主键具有唯一性
● 除数组外，所有mongodb支持的类型都可以作为文档主键，如递增的整数值，或者 uuid，甚至一篇文档作为另一篇文档的主键。
● 复合主键

#### 对象主键 ObjectId

默认也是使用最简单的方式是有 MongoDB 自动为文档生成文档主键。如果我们创建时不提供主键，MongoDB 会默认生成一个对象主键作为文档主键。

○ 默认文档主键
○ 可以快速生成的12字节id
○ 包含创建时间

一般对象主键的顺序就是文档创建的顺序，如果同意时间点生成了多个文档，则可能无法精确知道文档创建的顺序。<br>
另外对象主键一般由客户端生成，如果客户端时间不同，创建的时间也可能不同，也可能无法精确知道文档的创建顺序。


### 创建文档

* db.collection.insert()
* db.collection.insertOne()
* db.collection.save()
* 创建多个文档

#### db.collection.insertOne()

```mongo
collection 需要替换成文档将要写入的集合
document 需要替换为将要写入的文档本身
writeConcern 文档参数定义了本次文档创建操作的安全写级别，安全写级别用来判断一次数据库写入操作是否成功，安全写级别越高，丢失数据风险就越低，然而写入操作的延迟也可能更高。
如果不提供 writeConcern 文档，mongoDB 使用默认的安全写级别。
db.<collection>.insertOne(
    <document>,
    {
        writeConcern: <document>
    }
)
```
##### 数据准备

```json
{
    _id: "account1",
    name: "alice",
    balance: 100
}
``` 

##### 写入

```mongo
db.accounts.insertOne({_id: "account1", name: "alice", balance: 100})
```

##### 返回结果

```json
{"acknowledged": true, "insertedId": "account1"}
```

acknowledged: true 表示安全写级别被启用，默认安全写级别
insertedId： 被写入的文档_id

##### 查看集合列表

```mongo
> show collections
accounts
```

db.collection.insertOne() 会自动创建响应的集合



#### db.collection.insertMany()

##### 数据准备

```json
[{
    name: "charlie",
    balance: 500
},
{
    name: "david",
    balance: 200
}]
```

##### 写入

```mongo
db.accounts.insertMany([{name: "charlie", balance:500}, {name: "david", balance: 200}])
```

#### db.collection.insert()

创建单个或多个文档

```mongo
db.<collection>.insert(
    <document or array of document>,
    {
        writeConcern: <document>,
        ordered: <boolean>
    } 
)
```

##### 写入

```mongo
db.accounts.insert({name: "george", balance: 1000})
```

#### db.collection.save()

创建文档

```mongo
db.<collection>.save(
    <document>,
    {
        writeConcern: <document>
    } 
)
```

#### 其他命令

```mingo
> ObjectId(xxx)

提取 ObjectId 的创建时间
> ObjectId("xxx").getTimestamp()

使用文档作为另一个文档的主键
> db.accouts.insert({_id: {accountNo: "001", type: "saveings"}, name: "irene", balance: 80})
```

```mongo
使用 test 数据库
> use test

查看 test 数据库中集合，返回空白，因为现在 test 数据库中还没有集合和文档
> show collections
> 

创建第一个文档
> 



```



单个或多个文档
db.collection.insert()
db.<collection>.insert(
  <document or array of documents>,
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
)
db.collection.insert() 既可以写入一个单独的文档，也可以写入多个文档
> db.accounts.insert({name: "george", balance: 1000})
WriteResult({ "nInserted" : 1 })
>

=====
## 站点名称：活死人

### 站点创建时间：2023-12-31

### 初衷

这个站点的诞生源于对生活、工作和学习的一些记录，我怕会忘了一些人和一些事。它不仅是我个人在行知路上的见证，也是我情感和胡思乱想的出口。

### 目的

- **生活记录**：纪念那个在我生命中留下深刻印记的人，转移注意力，记录情绪变化。
- **工作与学习**：展示个人技能，分享知识与经验。

## 我的自画像

我称自己为“活死人”，因为我对这个世界的了解还很浅薄，对许多事物都感到无知和困惑。但同时，我也在呼吸，在焦虑，在探索，在成长。

### 我是谁？

- 一个对世界充满好奇，却又时常感到迷茫的人。
- 一个渴望了解事物本质，却又常常迷失方向的探索者。

### 我的目标

- 通过这个站点，我希望能够记录下自己的成长轨迹，分享我的所思所感。
- 希望能够通过分享，与更多的人产生共鸣，共同成长。

## 域名的选择

### 二级域名：notes

代表站点的性质——记录和分享我的笔记。

### 根域名：einscat.com

- "eins" 纪念一位对我影响深远的故友。
- "cat" 代表那位故友在我心中的形象，以及我所追求的精神状态。

## 结语

这个站点是我与世界沟通的桥梁，也是我内心世界的一扇窗。欢迎你来到这里，与我一起探索、成长。



==

# C 语言的灵魂——指针

## 特点

绕，其实并不难，无非就是要多想一会。

- 可以表示一些复杂的数据结构
- 可以快速传递数据
- 可以使函数返回一个以上的值
- 可以直接访问硬件
- 能方便处理字符串
- 是理解面向对象语言中引用的基础

## 基本概念

### 指针是什么？

指针就是地址。

### 地址又是什么？

地址是内存单元的编号。

### 什么又是内存单元的编号？

内存条中分为了很多小单元。一个单元有8位，可以存放8个0或者8个1，内存单位不是按位，而是字节，也就是不是一个0或者一个1为一个编号，而是8个0或者8个1组合在一起时一个编号。一个字节等于8位。

8个0或者8个1是一个编号，接着又是8个0或者8个1是一个编号。这个编号就是地址。

内存分为了很多单元或者很多小格子，每个格子一个编号。相当于我们的房子，一个房子一个编号，内存是一个格子一个编号，每个格子只能存放8个0或者8个1。

### 总结：

指针就是地址，指针和地址是一个概念。

## demo

```c
#include <stdio.h>

int main(void) {
    // p 是变量的名字，int * 表示 p 变量存放的是 int 类型变量的地址
    int *p;
    int i = 3;
    int j;

    // p = i; // ERROR，类型不一致，p 只能存放 int 类型变量的地址，不能存放int类型变量的值
    p = &i; // OK

    j = *p; // 等价于 j = i;
    printf("i = %d, j = %d, *p = %d\n", i, j, *p); // 输出：i = 3, j = 3, *p = 3

    return 0;
}

这些随之带来的就是各种各样的问题，我需要不断的去解决这些问题，这有让我看去像个活人。

看到这里，就当是我们的握手礼，我叫“陆知行”，我是另一个你，你的所有问题都来源我，你的抑郁、你的焦虑、你的自卑、你的种种问题都来来自我。“路知行”这个名字，其实很简单，来源于我在上海住的一条路上的地铁站，叫“行知路”，还别说上海有些地方的取名真的很好听。我接触的第一个上海土著也姓陆。

你没法知道她的情绪的变化，有她自己的节奏。这也是我一直想学的，能够控制住自己的情绪以及心态。可惜不知在哪一天被我弄丢了。尝试过很多种方式去寻找，却始终失败。


## 
"Cathay" - 古代对中国的称呼，带有一定的历史和文化含义。
Cathayan - 虽然不是一个常用词，但可以创造性地用来指代与Cathay（中国）有关的人或事物。
"Tao" - 道，代表道家哲学，可以象征中国哲学和生活方式。