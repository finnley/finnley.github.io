+++
title = 'MySQL基础知识回顾'
date = 2024-06-25T00:16:19+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 基础概念

### 库表

部署的MySQ服务也称为MySQL实例（Instance），实例中存在多个库（DataBase），每个库中有多个表（Table），每个表中又有很多字段（Column）

比如要创建一个 ”学校“ 的信息库，首先需要启动一个MySQL实例
在实例里面会创建很多库，比如“老师”库、“学生”库。
在“学生”库中可能有很多数据表，比如“学生基本信息表”、“学生份数表”等。
在“学生基本信息表”中有很多自爱单，比如“姓名”、“性别”等字段。

### SQL 分类

* DDL：数据定义语言 
* DML：数据操纵语言
* DCL：数据控制语言


## 库创建与使用

**1. 创建学生信息库:**

```sql
mysql> create database student_info;
Query OK, 1 row affected (0.02 sec)
```

**2. 查看库:**

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema | 
| student_info       |
| sys                |
+--------------------+
```

* `information_schme`：提供了数据库的元数据，比如数据库名、表名、索引等信息。
* `mysql`：用于存储数据库用户权限、数据库统计信息，如索引等。
* `performance_schema`：用于收集数据库性能数据信息，方便我们分析问题，如哪个SQL执行最多，耗时最长等。
* `sys`：它的所有数据来自 `performance_schema` 库，主要是方便我们快速了解数据库的运行情况。

**3. 使用库:**

```sql
mysql> use student_info
Database changed
```

## 表创建与使用

**1. 建库学生分数表:**

```sql
create table score_info(
    id int primary key auto_increment comment '主键', 
    name varchar(11) default null comment '学生姓名',
    score int default null comment '学生份数'
)engine=innodb charset=utf8mb4;
```

**2. 查看当前库下有哪些表:**

```sql
mysql> show tables;
+------------------------+
| Tables_in_student_info |
+------------------------+
| score_info             |
+------------------------+
1 row in set (0.02 sec)
```

**3. 查看表结构:**

```sql
mysql> show create table score_info;
+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table      | Create Table                                                                                                                                                                                                                                                                          |
+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| score_info | CREATE TABLE `score_info` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(11) DEFAULT NULL COMMENT '学生姓名',
  `score` int DEFAULT NULL COMMENT '学生份数',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci           |
+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```


还可以：

```sql
mysql> desc score_info;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int         | NO   | PRI | NULL    | auto_increment |
| name  | varchar(11) | YES  |     | NULL    |                |
| score | int         | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)
```

## 新增表字段

**1. 在“学生信息表”中增加班级字段:**

```sql
mysql> alter table score_info add column class varchar(10) default null comment '班级';
```

查看表结构：
```sql
mysql> desc score_info;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int         | NO   | PRI | NULL    | auto_increment |
| name  | varchar(11) | YES  |     | NULL    |                |
| score | int         | YES  |     | NULL    |                |
| class | varchar(10) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```


使用这种方式都是在表的最后增加的字段。

**2. 在“学生信息表”中指定位置后面添加字段:**

在“学生信息表”中增加“课程”字段：

```sql
mysql> alter table score_info add column course varchar(10) default null comment '课程' after name;
```

该SQL表示在 score_info 表中的 name 字段后面添加 course 字段。

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     |  NULL    |                |
| class  | varchar(10) | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

**3. 在“学生信息表”中最前面字段:**

在表的最前面增加一个“学号”字段：

```sql
mysql> alter table score_info add column stu_id int default null comment '学号' first;
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| stu_id | int         | YES  |     | NULL    |                |
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | varchar(10) | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
6 rows in set (0.01 sec)
```

**4. 在“学生信息表”中同时添加多个字段:**

```sql
mysql> alter table score_info add column sex varchar(10) default null comment '性别', add column age int default null comment '年龄';
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| stu_id | int         | YES  |     | NULL    |                |
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | varchar(10) | YES  |     | NULL    |                |
| sex    | varchar(10) | YES  |     | NULL    |                |
| age    | int         | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
8 rows in set (0.01 sec)
```

## 删除表字段

**1. 在“学生信息表”中删除字段:**

```sql
mysql> alter table score_info drop column stu_id;
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | varchar(10) | YES  |     | NULL    |                |
| sex    | varchar(10) | YES  |     | NULL    |                |
| age    | int         | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
7 rows in set (0.01 sec)
```

**2. 在“学生信息表”中同时删除多个字段:**

```sql
mysql> alter table score_info drop column sex, drop column age;
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | varchar(10) | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

## 修改表字段类型 

**1. 将“学生信息表”表字段class字段类型修改为int类型:**

```sql
mysql> alter table score_info modify column class int default null comment '班级';
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(11) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | int         | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

**2. 将“学生信息表”表字段name字段类型长度:**

```sql
mysql> alter table score_info modify column name varchar(20) default null comment '学生姓名';
```

查看表结构：

```sql
mysql> desc score_info;
+--------+-------------+------+-----+---------+----------------+
| Field  | Type        | Null | Key | Default | Extra          |
+--------+-------------+------+-----+---------+----------------+
| id     | int         | NO   | PRI | NULL    | auto_increment |
| name   | varchar(20) | YES  |     | NULL    |                |
| course | varchar(10) | YES  |     | NULL    |                |
| score  | int         | YES  |     | NULL    |                |
| class  | int         | YES  |     | NULL    |                |
+--------+-------------+------+-----+---------+----------------+
5 rows in set (0.01 sec)
```

## 修改表字段名称

**1. 将“学生信息表”表字段名称:**

```sql
mysql> alter table score_info change name stu_name varchar(20) default null comment '学生姓名';
```

查看表结构：

```sql
mysql> desc score_info;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int         | NO   | PRI | NULL    | auto_increment |
| stu_name | varchar(20) | YES  |     | NULL    |                |
| course   | varchar(10) | YES  |     | NULL    |                |
| score    | int         | YES  |     | NULL    |                |
| class    | int         | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
5 rows in set (0.01 sec)
```

## 删表

```sql
mysql> drop table score_info;
Query OK, 0 rows affected (0.03 sec)
```

## 增删改查

**1. 新建 score_info 表:**

```
create table score_info(
    id int primary key auto_increment comment '主键', 
    name varchar(11) default null comment '学生姓名',
    score int default null comment '学生分数'
)engine=innodb charset=utf8mb4;
```

**2. 写入单行数据:**

```sql
mysql> insert into score_info values (1, 'a', 88);
```

注意：int 类型的值不需要使用引号， char 或者 varchar 类型的值需要使用引号。

**3. 写入多行数据:**

```sql
mysql> insert into score_info values (2, 'b', 90), (3, 'c', 86);
```

**4. 写入指定字段数据:**

```sql
mysql> insert into score_info(name, score) values ('d', 60);
```

**5. 全表查询:**

```sql
mysql> select * from score_info;
+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | a    |    88 |
|  2 | b    |    90 |
|  3 | c    |    86 |
|  4 | d    |    60 |
+----+------+-------+
4 rows in set (0.00 sec)
```

**6. 条件查询:**

查询 id 值为 2 的记录

```sql
mysql> select * from score_info where id = 2;
+----+------+-------+
| id | name | score |
+----+------+-------+
|  2 | b    |    90 |
+----+------+-------+
1 row in set (0.01 sec)
```

**7. 指定字段查询:**

查询 name 的记录

```sql
mysql> select name from score_info where id = 2;
+------+
| name |
+------+
| b    |
+------+
1 row in set (0.00 sec)
```

**8. 按条件修改数据:**

把 score_info 表中 id 为 1 的记录的 name 值修改为 'aa';

```sql
mysql> update score_info set name = 'aa' where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from score_info;
+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | aa   |    88 |
|  2 | b    |    90 |
|  3 | c    |    86 |
|  4 | d    |    60 |
+----+------+-------+
4 rows in set (0.00 sec)
``` 

**9. 全表修改数据:**

```sql
mysql> update score_info set name = 'aa';
Query OK, 3 rows affected (0.01 sec)
Rows matched: 4  Changed: 3  Warnings: 0

mysql> select * from score_info;
+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | aa   |    88 |
|  2 | aa   |    90 |
|  3 | aa   |    86 |
|  4 | aa   |    60 |
+----+------+-------+
4 rows in set (0.00 sec)
``` 

**10. 按条件删除数据:**

```sql
mysql> delete from score_info where id = 4;
Query OK, 1 row affected (0.00 sec)

mysql> select * from score_info;
+----+------+-------+
| id | name | score |
+----+------+-------+
|  1 | aa   |    88 |
|  2 | aa   |    90 |
|  3 | aa   |    86 |
+----+------+-------+
3 rows in set (0.00 sec)
```  

**11. 删除全表数据:**

```sql
mysql> delete from score_info;
Query OK, 3 rows affected (0.01 sec)

mysql> select * from score_info;
Empty set (0.01 sec)
```  

注意：删除表数据时一定要添加where条件，否则删除的就是所有数据

如果要清空表所有数据，可以使用下面语句，因为 binlog 在行模式下使用delete方式删除会记录每一行的删除，会导致binlog非常大。
使用 truncate 就只会记录一条 truncate 语句

```sql
mysql> truncate score_info;
Query OK, 0 rows affected (0.07 sec) 
```

**13. 过滤表数据:**

先准备测试数据：

```sql
mysql> insert into score_info values (1, 'a', 88), (2, 'b', 90), (3, 'c', 86), (4, 'd', 60);
```

* 等值匹配

    查询 name 名字为 'a' 的学生信息：

    ```sql
    mysql> select * from score_info where name = 'a';
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  1 | a    |    88 |
    +----+------+-------+
    1 row in set (0.00 sec)
    ```

* 不等于

    查询 name 名字不为 'a' 的学生信息：

    ```sql
    mysql> select * from score_info where name <> 'a';
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  2 | b    |    90 |
    |  3 | c    |    86 |
    |  4 | d    |    60 |
    +----+------+-------+
    3 rows in set (0.01 sec)
    ```

    或者下面的写法：

    ```sql
    mysql> select * from score_info where name != 'a';
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  2 | b    |    90 |
    |  3 | c    |    86 |
    |  4 | d    |    60 |
    +----+------+-------+
    3 rows in set (0.01 sec)
    ```

* 小于

    查询分数小于 80 的学生信息

    ```sql
    mysql> select * from score_info where score < 80;
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  4 | d    |    60 |
    +----+------+-------+
    1 row in set (0.01 sec)
    ```

    如果是小于等于则将 ‘<’ 替换为 ‘<=’。

* 大于

    查询分数大于 80 的学生信息

    ```sql
    mysql> select * from score_info where score > 80;
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  1 | a    |    88 |
    |  2 | b    |    90 |
    |  3 | c    |    86 |
    +----+------+-------+
    3 rows in set (0.01 sec)
    ```

    如果是大于等于则将 ‘>’ 替换为 ‘>=’。

* 范围查询

    查询分数在 80~90 之间的学生信息

    ```sql
    mysql> select * from score_info where score between 80 and 90;
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  1 | a    |    88 |
    |  2 | b    |    90 |
    |  3 | c    |    86 |
    +----+------+-------+
    3 rows in set (0.01 sec)
    ```
* 多条件同时满足

    查询分数大于等于88分，并且名字是 'a' 的学生信息。

    ```sql
    mysql> select * from score_info where score >= 88 and name = 'a';
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  1 | a    |    88 |
    +----+------+-------+
    1 row in set (0.00 sec)
    ```

* 多条件匹配任意一个

    查询分数大于88或者分数小于等于60分的学生信息。

    ```sql
    mysql> select * from score_info where score > 88 or score <= 60;
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  2 | b    |    90 |
    |  4 | d    |    60 |
    +----+------+-------+
    2 rows in set (0.01 sec)
    ```

* 查询多个值

    查询名字是 a 和 b 的学生信息。

    ```sql
    mysql> select * from score_info where name in ('a', 'b');
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  1 | a    |    88 |
    |  2 | b    |    90 |
    +----+------+-------+
    2 rows in set (0.00 sec)
    ```

    与之相反的就是 `not in`

    ```sql
    mysql> select * from score_info where name not in ('a', 'b');
    +----+------+-------+
    | id | name | score |
    +----+------+-------+
    |  3 | c    |    86 |
    |  4 | d    |    60 |
    +----+------+-------+
    2 rows in set (0.01 sec)
    ```