+++
title = 'MySQL基础知识回顾'
date = 2024-06-25T00:16:19+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 基础概念

### 库表

部署的MySQ服务也称为MySQL实例（Instance），实例中存在多个库（DataBase），每个库中有多个表（Table），每个表中又有很多字段（Column）

比如要创建一个 ”学校“ 的信息库，首先需要启动一个MySQL实例
在实例里面会创建很多库，比如“老师”库、“学生”库。
在“学生”库中可能有很多数据表，比如“学生基本信息表”、“学生分数表”等。
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
    score int default null comment '学生分数'
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
  `score` int DEFAULT NULL COMMENT '学生分数',
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

**14. SQL子查询:**

* 准备数据前查看当前库表

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
5 rows in set (0.00 sec)

mysql> use student_info
Database changed
mysql> show tables;
+------------------------+
| Tables_in_student_info |
+------------------------+
| score_info             |
+------------------------+
1 row in set (0.00 sec)
mysql> drop table score_info;
Query OK, 0 rows affected (0.02 sec)
```

* 创建学生信息表

```sql
create table student_info (
    id int primary key auto_increment comment '主键',
    stu_id int default null comment '学生ID',
    stu_name varchar(30) default null comment '学生姓名',
    class varchar(30) default null comment '学生班级'
)engine=innodb default charset=utf8mb4;
```

* 创建学生分数表

```sql
create table score_info(
    id int primary key auto_increment comment '主键', 
    stu_id int default null comment '学生ID',
    score int default null comment '学生分数'
)engine=innodb charset=utf8mb4;
```

* 写入测试数据

```sql
insert into student_info(stu_id, stu_name, class) values(1, 'a', '一班'), (2, 'b', '一班'), (3, 'c', '二班'), (4, 'd', '二班'), (5, 'e', '二班');
insert into score_info(stu_id, score) values(1, 88), (2, 90), (3, 82), (4, 92), (6, 85);
```

* 查看数据

```sql
mysql> select * from student_info;
+----+--------+----------+--------+
| id | stu_id | stu_name | class  |
+----+--------+----------+--------+
|  1 |      1 | a        | 一班   |
|  2 |      2 | b        | 一班   |
|  3 |      3 | c        | 二班   |
|  4 |      4 | d        | 二班   |
|  5 |      5 | e        | 二班   |
+----+--------+----------+--------+
5 rows in set (0.00 sec)

mysql> select * from score_info;
+----+--------+-------+
| id | stu_id | score |
+----+--------+-------+
|  1 |      1 |    88 |
|  2 |      2 |    90 |
|  3 |      3 |    82 |
|  4 |      4 |    92 |
|  5 |      6 |    85 |
+----+--------+-------+
5 rows in set (0.01 sec)
```

* 单行子查询

    查询学生名字是a的学生分数。先在学生信息表中查询名字是 a 的学生ID，以查询出的结果为子查询，再到学生分数表中查询学生ID对应的分数。
    ```sql
    mysql> select * from score_info where stu_id = (select stu_id from student_info where stu_name = 'a');
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    +----+--------+-------+
    1 row in set (0.01 sec)
    ```

* 多行子查询

    查询学生是一班的学生分数。先在学生信息表中查询班级是一班的学生ID，以查询出的结果为子查询，再到学生分数表中查询学生ID对应的分数。

    ```sql
    mysql> select * from score_info where stu_id in (select stu_id from student_info where class = '一班');
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    |  2 |      2 |    90 |
    +----+--------+-------+
    2 rows in set (0.00 sec)
    ```

* FROM 子句子查询

    比如对某个字段做处理，形成一个新的表。

    ```sql
    mysql> select stu_id, score-1 as score from score_info;
    +--------+-------+
    | stu_id | score |
    +--------+-------+
    |      1 |    87 |
    |      2 |    89 |
    |      3 |    81 |
    |      4 |    91 |
    |      6 |    84 |
    +--------+-------+
    5 rows in set (0.01 sec)

    mysql> select stu_id, score from (select stu_id, score-1 as score from score_info) as score_tmp where score > 90;
    +--------+-------+
    | stu_id | score |
    +--------+-------+
    |      4 |    91 |
    +--------+-------+
    1 row in set (0.01 sec)

    mysql>
    ```

* 多行子查询

    查询学生是一班的学生分数。先在学生信息表中查询班级是一班的学生ID，以查询出的结果为子查询，再到学生分数表中查询学生ID对应的分数。

    ```sql
    mysql> select * from score_info where stu_id in (select stu_id from student_info where class = '一班');
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    |  2 |      2 |    90 |
    +----+--------+-------+
    2 rows in set (0.00 sec)
    ```

**15. 关联查询:**

* 内连接

    通过关联查询合并两个表的结果。

    ```sql
    mysql> select * from student_info a inner join score_info b on a.stu_id = b.stu_id;
    +----+--------+----------+--------+----+--------+-------+
    | id | stu_id | stu_name | class  | id | stu_id | score |
    +----+--------+----------+--------+----+--------+-------+
    |  1 |      1 | a        | 一班   |  1 |      1 |    88 |
    |  2 |      2 | b        | 一班   |  2 |      2 |    90 |
    |  3 |      3 | c        | 二班   |  3 |      3 |    82 |
    |  4 |      4 | d        | 二班   |  4 |      4 |    92 |
    +----+--------+----------+--------+----+--------+-------+
    4 rows in set (0.02 sec)

    mysql> select * from student_info;
    +----+--------+----------+--------+
    | id | stu_id | stu_name | class  |
    +----+--------+----------+--------+
    |  1 |      1 | a        | 一班   |
    |  2 |      2 | b        | 一班   |
    |  3 |      3 | c        | 二班   |
    |  4 |      4 | d        | 二班   |
    |  5 |      5 | e        | 二班   |
    +----+--------+----------+--------+
    5 rows in set (0.00 sec)

    mysql> select * from score_info;
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    |  2 |      2 |    90 |
    |  3 |      3 |    82 |
    |  4 |      4 |    92 |
    |  5 |      6 |    85 |
    +----+--------+-------+
    5 rows in set (0.00 sec)

    mysql>
    ```

    student_info 表中有个stu_id 为 5的记录没有被查询出来，score_info 里面有个 stu_id 为 6 的记录没有被查询出来。内连接查询的两张表中关联字段相等的记录。

    另外可以展示指定关联的某些字段，比如：

    ```sql
    mysql> select a.stu_id, a.stu_name, b.score from student_info a inner join score_info b on a.stu_id = b.stu_id;
    +--------+----------+-------+
    | stu_id | stu_name | score |
    +--------+----------+-------+
    |      1 | a        |    88 |
    |      2 | b        |    90 |
    |      3 | c        |    82 |
    |      4 | d        |    92 |
    +--------+----------+-------+
    4 rows in set (0.00 sec)
    ```

    带条件查询：

    ```sql
    mysql> select a.stu_id, a.stu_name, b.score from student_info a inner join score_info b on a.stu_id = b.stu_id where a.class='一
    班';
    +--------+----------+-------+
    | stu_id | stu_name | score |
    +--------+----------+-------+
    |      1 | a        |    88 |
    |      2 | b        |    90 |
    +--------+----------+-------+
    2 rows in set (0.01 sec)
    ```

* 左连接

    通过关联查询合并两个表的结果。

    ```sql
    mysql> select * from student_info;
    +----+--------+----------+--------+
    | id | stu_id | stu_name | class  |
    +----+--------+----------+--------+
    |  1 |      1 | a        | 一班   |
    |  2 |      2 | b        | 一班   |
    |  3 |      3 | c        | 二班   |
    |  4 |      4 | d        | 二班   |
    |  5 |      5 | e        | 二班   |
    +----+--------+----------+--------+
    5 rows in set (0.00 sec)

    mysql> select * from score_info;
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    |  2 |      2 |    90 |
    |  3 |      3 |    82 |
    |  4 |      4 |    92 |
    |  5 |      6 |    85 |
    +----+--------+-------+
    5 rows in set (0.00 sec)

    mysql> select * from student_info a left join score_info b on a.stu_id = b.stu_id;
    +----+--------+----------+--------+------+--------+-------+
    | id | stu_id | stu_name | class  | id   | stu_id | score |
    +----+--------+----------+--------+------+--------+-------+
    |  1 |      1 | a        | 一班   |    1 |      1 |    88 |
    |  2 |      2 | b        | 一班   |    2 |      2 |    90 |
    |  3 |      3 | c        | 二班   |    3 |      3 |    82 |
    |  4 |      4 | d        | 二班   |    4 |      4 |    92 |
    |  5 |      5 | e        | 二班   | NULL |   NULL |  NULL |
    +----+--------+----------+--------+------+--------+-------+
    5 rows in set (0.01 sec)

    mysql>
    ```

    发现不仅展示 student_info 表中 stu_id 相等的记录，还显式了 stu_id 为 5 的记录，但是 score_info 表中并没有 stu_id 为 5 的记录，所以值为 NULL。

    所以，left join 的特点会展示左表的数据，右表只会展示关联字段的数据

* 右连接

    通过关联查询合并两个表的结果。

    ```sql
    mysql> select * from student_info;
    +----+--------+----------+--------+
    | id | stu_id | stu_name | class  |
    +----+--------+----------+--------+
    |  1 |      1 | a        | 一班   |
    |  2 |      2 | b        | 一班   |
    |  3 |      3 | c        | 二班   |
    |  4 |      4 | d        | 二班   |
    |  5 |      5 | e        | 二班   |
    +----+--------+----------+--------+
    5 rows in set (0.00 sec)

    mysql> select * from score_info;
    +----+--------+-------+
    | id | stu_id | score |
    +----+--------+-------+
    |  1 |      1 |    88 |
    |  2 |      2 |    90 |
    |  3 |      3 |    82 |
    |  4 |      4 |    92 |
    |  5 |      6 |    85 |
    +----+--------+-------+
    5 rows in set (0.00 sec)

    mysql> select * from student_info a right join score_info b on a.stu_id = b.stu_id;
    +------+--------+----------+--------+----+--------+-------+
    | id   | stu_id | stu_name | class  | id | stu_id | score |
    +------+--------+----------+--------+----+--------+-------+
    |    1 |      1 | a        | 一班   |  1 |      1 |    88 |
    |    2 |      2 | b        | 一班   |  2 |      2 |    90 |
    |    3 |      3 | c        | 二班   |  3 |      3 |    82 |
    |    4 |      4 | d        | 二班   |  4 |      4 |    92 |
    | NULL |   NULL | NULL     | NULL   |  5 |      6 |    85 |
    +------+--------+----------+--------+----+--------+-------+
    5 rows in set (0.01 sec)

    mysql>
    ```

    right join 的特点会展示右表的数据，左表只展示关联字段的数据。

**15. 聚合查询:**

* 准备数据前查看当前库表

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
5 rows in set (0.00 sec)

mysql> use student_info
Database changed
mysql> show tables;
+------------------------+
| Tables_in_student_info |
+------------------------+
| score_info             |
+------------------------+
1 row in set (0.00 sec)
```

* 创建学生份数关联表

```sql
create table student_score (
    id int primary key auto_increment comment '主键',
    stu_id int default null comment '学生ID',
    class varchar(30) default null comment '学生班级',
    score int default null comment '学生分数'
)engine=innodb default charset=utf8mb4;
```


* 写入测试数据

```sql
insert into student_score(stu_id, class, score) values(1, '一班', 88), (2, '一班', 90), (3, '二班', 82), (4, '二班', 92), (5, '二班', 85);
```

* 查看数据

```sql
mysql> select * from student_score;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

* 聚合函数

    行数统计：
    ```sql
    mysql> select count(*) from student_score;
    +----------+
    | count(*) |
    +----------+
    |        5 |
    +----------+
    1 row in set (0.01 sec)
    ```

    平均值：
    ```sql
    mysql> select avg(score) from student_score;
    +------------+
    | avg(score) |
    +------------+
    |    87.4000 |
    +------------+
    1 row in set (0.00 sec)
    ```

    求和：
    ```sql
    mysql> select sum(score) from student_score;
    +------------+
    | sum(score) |
    +------------+
    |        437 |
    +------------+
    1 row in set (0.00 sec)
    ```

    最大值：
    ```sql
    mysql> select max(score) from student_score;
    +------------+
    | max(score) |
    +------------+
    |         92 |
    +------------+
    1 row in set (0.01 sec)
    ```

    最小值：
    ```sql
    mysql> select min(score) from student_score;
    +------------+
    | min(score) |
    +------------+
    |         82 |
    +------------+
    1 row in set (0.01 sec)
    ```

**16. 分组:**

    求每个班的平均分。以class来分组，再查询每个分组的平均分：
    ```sql
    mysql> select class, avg(score) from student_score group by class;
    +--------+------------+
    | class  | avg(score) |
    +--------+------------+
    | 一班   |    89.0000 |
    | 二班   |    86.3333 |
    +--------+------------+
    2 rows in set (0.00 sec)
    ```

    分组，求每个班的最高分。
    ```sql
    mysql> select class, max(score) from student_score group by class;
    +--------+------------+
    | class  | max(score) |
    +--------+------------+
    | 一班   |         90 |
    | 二班   |         92 |
    +--------+------------+
    2 rows in set (0.01 sec)
    ```

    分组，求每个班的人数：
    ```sql
    mysql> select class, count(*) from student_score group by class;
    +--------+----------+
    | class  | count(*) |
    +--------+----------+
    | 一班   |        2 |
    | 二班   |        3 |
    +--------+----------+
    2 rows in set (0.01 sec)
    ```

    按班级分开限制每个班的学号：
    ```sql
    mysql> select class, group_concat(stu_id) from student_score group by class;
    +--------+----------------------+
    | class  | group_concat(stu_id) |
    +--------+----------------------+
    | 一班   | 1,2                  |
    | 二班   | 3,4,5                |
    +--------+----------------------+
    2 rows in set (0.00 sec)
    ```

    分组过滤，比如分组之后过滤再求和，查看人数大于2的班级，having 过滤的是分组之后的记录，如果使用where 就会报错：
    ```sql
    mysql> select class, count(*) as stu_count from student_score group by class having stu_count > 2;
    +--------+-----------+
    | class  | stu_count |
    +--------+-----------+
    | 二班   |         3 |
    +--------+-----------+
    1 row in set (0.00 sec)
    ```

**16. 模糊查询**


* 创建学生姓名表

```sql
create table student_name (
    id int primary key auto_increment comment '主键',
    stu_id int default null comment '学生ID',
    stu_name varchar(30) default null comment '学生姓名'
)engine=innodb default charset=utf8mb4;
```


* 写入测试数据

```sql
insert into student_name(stu_id, stu_name) values(1, 'abc'), (2, 'def'), (3, 'ghi');
```

* 匹配关键字符

查询学生姓名以 ‘a’ 开头的学生

```sql
mysql> select * from student_name where stu_name like 'a%';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
+----+--------+----------+
1 row in set (0.01 sec)
```


查询学生姓名最后一个以 ‘c’ 结尾的的学生

```sql
mysql> select * from student_name where stu_name like '%c';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
+----+--------+----------+
1 row in set (0.01 sec)
```

查询学生姓名中间是 ‘b’ 的的学生:
```sql
mysql> select * from student_name where stu_name like '%b%';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
+----+--------+----------+
1 row in set (0.00 sec)
```

* 匹配单个字符

```sql
mysql> select * from student_name where stu_name like 'ab_';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
+----+--------+----------+
1 row in set (0.01 sec)
```

* 匹配所包含的任意一个字符

查询学生姓名中包含a或者d任意一个字符的学生

```sql
mysql> select * from student_name where stu_name regexp '[ad]';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
|  2 |      2 | def      |
+----+--------+----------+
2 rows in set (0.01 sec)
```

上面还有另外一种写法：
```sql
mysql> select * from student_name where stu_name regexp 'a|d';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
|  2 |      2 | def      |
+----+--------+----------+
2 rows in set (0.01 sec)
```

匹配包含英语字母的学生：
```sql
mysql> select * from student_name where stu_name regexp '[a-z]';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
|  2 |      2 | def      |
|  3 |      3 | ghi      |
+----+--------+----------+
3 rows in set (0.01 sec)
```

匹配包含数字的学生：
```sql
mysql> select * from student_name where stu_id regexp '[0-9]';
+----+--------+----------+
| id | stu_id | stu_name |
+----+--------+----------+
|  1 |      1 | abc      |
|  2 |      2 | def      |
|  3 |      3 | ghi      |
+----+--------+----------+
3 rows in set (0.01 sec)
```


`like` 表示模糊匹配

`%` 表示匹配任意字符，字符数量可以是0个或多个

`not like` 表示不匹配

`_` 表示匹配单个字符


`regexp` 表示匹配任意字符


**17. 排序**

使用的下面数据表

```sql
mysql> select * from student_score;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

* 正序

按照分数正序排列：

```sql
mysql> select * from student_score order by score;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  5 |      5 | 二班   |    85 |
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

* 倒序

按照分数降序排列：
```sql
mysql> select * from student_score order by score desc;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  4 |      4 | 二班   |    92 |
|  2 |      2 | 一班   |    90 |
|  1 |      1 | 一班   |    88 |
|  5 |      5 | 二班   |    85 |
|  3 |      3 | 二班   |    82 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

* 多字段排序，比如先按照班级排序，再按照分数排序

```sql
mysql> select * from student_score order by class, score;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
|  3 |      3 | 二班   |    82 |
|  5 |      5 | 二班   |    85 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

**18. 分页**

* 表数据

```sql
mysql> select * from student_score;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

* 查询表前N条记录

```sql
mysql> select * from student_score limit 2;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  1 |      1 | 一班   |    88 |
|  2 |      2 | 一班   |    90 |
+----+--------+--------+-------+
```

* 分页

从行2开始查询2条记录：

```sql
mysql> select * from student_score limit 2, 2;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
2 rows in set (0.00 sec)
```

发现是从表的第三条记录开始展示的，因为表的记录行号是从0开始的。

* 排序之后查询，比如查询班级的前三名

```sql
mysql> select * from student_score order by score desc limit 3;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  4 |      4 | 二班   |    92 |
|  2 |      2 | 一班   |    90 |
|  1 |      1 | 一班   |    88 |
+----+--------+--------+-------+
3 rows in set (0.01 sec)
```

* offset

```sql
mysql> select * from student_score limit 2 offset 1;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  2 |      2 | 一班   |    90 |
|  3 |      3 | 二班   |    82 |
+----+--------+--------+-------+
2 rows in set (0.00 sec)

mysql> select * from student_score limit 2 offset 2;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
2 rows in set (0.00 sec)

mysql>
```

offset 表示要跳过的行

**19. 组合**

* 表数据

查询二班的所有学生分数：
```sql
mysql> select * from student_score where class = '二班';
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
+----+--------+--------+-------+
3 rows in set (0.00 sec)

mysql>
```

再查询分数大于90分的学生：
```sql
mysql> select * from student_score where score >= 90;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  2 |      2 | 一班   |    90 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
2 rows in set (0.01 sec)
```

现在想要组合这两个查询结果：
```sql
select * from student_score where class = '二班'
union
select * from student_score where score >= 90;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
|  2 |      2 | 一班   |    90 |
+----+--------+--------+-------+
4 rows in set (0.00 sec)
```

两个表中都有一条 id 为 的记录，发现 union 默认会对结果去重，如果不去重，可以使用 union all:
```sql
select * from student_score where class = '二班'
union all
select * from student_score where score >= 90;
+----+--------+--------+-------+
| id | stu_id | class  | score |
+----+--------+--------+-------+
|  3 |      3 | 二班   |    82 |
|  4 |      4 | 二班   |    92 |
|  5 |      5 | 二班   |    85 |
|  2 |      2 | 一班   |    90 |
|  4 |      4 | 二班   |    92 |
+----+--------+--------+-------+
5 rows in set (0.00 sec)
```

## 小练习

* 创建学生信息库 student

```sql
create database student;
use student
```

* 在 student 库中创建学生基本信息表 student_info

```sql
create table student_info (
    id int primary key auto_increment comment '主键',
    stu_num int default null comment '学号',
    stu_name varchar(10) default null comment '学生姓名',
    stu_class varchar(10) default null comment '学生班级'
)engine=innodb default charset=utf8mb4;
```

```sql
insert into student_info(stu_num, stu_name, stu_class) values(1101, '周星驰', '高三一班'), (1102, '张学友', '高三一班'), (1201, '王小波', '高三二班'), (1202, '金刚狼', '高三二班');
```

表数据

```sql
mysql> select * from student_info;
+----+---------+-----------+--------------+
| id | stu_num | stu_name  | stu_class    |
+----+---------+-----------+--------------+
|  1 |    1101 | 周星驰    | 高三一班     |
|  2 |    1102 | 张学友    | 高三一班     |
|  3 |    1201 | 王小波    | 高三二班     |
|  4 |    1202 | 金刚狼    | 高三二班     |
+----+---------+-----------+--------------+
4 rows in set (0.00 sec)
```

* 创建一张学生分数表，表名为 student_score

```
create table student_score (
    id int primary key auto_increment comment '主键',
    stu_num int default null comment '学号',
    stu_course varchar(10) default null comment '学科',
    stu_score int default null comment '分数'
)engine=innodb default charset=utf8mb4;
```

```sql
insert into student_score(stu_num, stu_course, stu_score) values(1101, '语文', 88), (1101, '数学', 90), (1102, '语文', 91), (1102, '数学', 87), (1201, '语文', 89), (1201, '数学', 80), (1202, '语文', 92), (1202, '数学', 89);
```

表数据：
```sql
mysql> select * from student_score;
+----+---------+------------+-----------+
| id | stu_num | stu_course | stu_score |
+----+---------+------------+-----------+
|  1 |    1101 | 语文       |        88 |
|  2 |    1101 | 数学       |        90 |
|  3 |    1102 | 语文       |        91 |
|  4 |    1102 | 数学       |        87 |
|  5 |    1201 | 语文       |        89 |
|  6 |    1201 | 数学       |        80 |
|  7 |    1202 | 语文       |        92 |
|  8 |    1202 | 数学       |        89 |
+----+---------+------------+-----------+
8 rows in set (0.00 sec)
```

* 案例： 查看周星驰的学号

```sql
mysql> select stu_num from student_info where stu_name = '周星驰';
+---------+
| stu_num |
+---------+
|    1101 |
+---------+
1 row in set (0.00 sec)
```

* 案例：查询姓张的学生

```sql
mysql> select * from student_info where stu_name like '张%';
+----+---------+-----------+--------------+
| id | stu_num | stu_name  | stu_class    |
+----+---------+-----------+--------------+
|  2 |    1102 | 张学友    | 高三一班     |
+----+---------+-----------+--------------+
1 row in set (0.00 sec)
```

* 案例：查询一班的人数 

```sql
mysql> select count(*) from student_info where stu_class='高三一班';
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)
```

* 案例：查询周星驰的所有成绩

分析：先通过学生表拿到学生的学号，然后根据学号查询分数

子查询方式：
```sql
mysql> select * from student_score where stu_num = (select stu_num from student_info where stu_name = '周星驰');
+----+---------+------------+-----------+
| id | stu_num | stu_course | stu_score |
+----+---------+------------+-----------+
|  1 |    1101 | 语文       |        88 |
|  2 |    1101 | 数学       |        90 |
+----+---------+------------+-----------+
2 rows in set (0.00 sec)
```

关联查询方式：
```sql
mysql> select * from student_score a inner join student_info b on a.stu_num = b.stu_num where b.stu_name = '周星驰';
+----+---------+------------+-----------+----+---------+-----------+--------------+
| id | stu_num | stu_course | stu_score | id | stu_num | stu_name  | stu_class    |
+----+---------+------------+-----------+----+---------+-----------+--------------+
|  1 |    1101 | 语文       |        88 |  1 |    1101 | 周星驰    | 高三一班     |
|  2 |    1101 | 数学       |        90 |  1 |    1101 | 周星驰    | 高三一班     |
+----+---------+------------+-----------+----+---------+-----------+--------------+
2 rows in set (0.01 sec)
```

* 案例：查询分数大于90的学生姓名和学科

```
mysql>  select b.stu_name, a.stu_course, a.stu_score from student_score a inner join student_info b on a.stu_num = b.stu_num where a.stu_score > 90;
+-----------+------------+-----------+
| stu_name  | stu_course | stu_score |
+-----------+------------+-----------+
| 张学友    | 语文       |        91 |
| 金刚狼    | 语文       |        92 |
+-----------+------------+-----------+
2 rows in set (0.01 sec)
```

* 案例：查看所有人的语文和数学的总分

分析： 先根据学号分组，在查询他们的和

```sql
mysql> select stu_num, sum(stu_score) from student_score group by stu_num;
+---------+----------------+
| stu_num | sum(stu_score) |
+---------+----------------+
|    1101 |            178 |
|    1102 |            178 |
|    1201 |            169 |
|    1202 |            181 |
+---------+----------------+
4 rows in set (0.00 sec)
```

* 案例：查看总分排名前二的学生姓名和分数

```sql
mysql> select b.stu_name, a.stu_num, sum(a.stu_score) as score from student_score a inner join student_info b on a.stu_num = b.stu_num group by a.stu_num order by score desc limit 2;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'student.b.stu_name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
mysql> show global variables like 'sql_mode';
+---------------+-----------------------------------------------------------------------------------------------------------------------+
| Variable_name | Value                                                                                                                 |
+---------------+-----------------------------------------------------------------------------------------------------------------------+
| sql_mode      | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION |
+---------------+-----------------------------------------------------------------------------------------------------------------------+
1 row in set (0.02 sec)

mysql>
```

将 ONLY_FULL_GROUP_BY 移除掉：
```sql
set global sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```

退出MySQL重新 use student，再执行查询语句：
```sql
mysql> select b.stu_name, a.stu_num, sum(a.stu_score) as score from student_score a inner join student_info b on a.stu_num = b.stu_num group by a.stu_num order by score desc limit 2;
+-----------+---------+-------+
| stu_name  | stu_num | score |
+-----------+---------+-------+
| 金刚狼    |    1202 |   181 |
| 周星驰    |    1101 |   178 |
+-----------+---------+-------+
2 rows in set (0.01 sec)
```

* 案例：求各学科的平均分

```sql
mysql> select stu_course, avg(stu_score) from student_score group by stu_course;
+------------+----------------+
| stu_course | avg(stu_score) |
+------------+----------------+
| 语文       |        90.0000 |
| 数学       |        86.5000 |
+------------+----------------+
2 rows in set (0.01 sec)
```

## ChatGPT 练习

* 充当客户端

prompt：

```
请你充当MySQL8.0的客户端，执行的命令返回形式请按照实际在MySQL客户端执行的返回结果，不需要添加其他多余的问题，明白回复收到。
```

充当SQL考官：

prompt：
```
请你来帮忙联系一下MySQL的语句，每次给我出一道题，让我写出SQL语句，你回答告诉我正确还是错误，如果错误，告诉我正确答案，然后再出一道新的
```

* 出SQL笔试题

prompt：

```
出一道MYSQL笔试题，同一张表，包括多个问题
```

## 字段类型

* CHAR 和 VARCHAR

char 用来保存固定长度的字符，使用时后面通常会声明固定的长度，范围是0~255。
比如char(10)表示最多可以容纳10个字符

varchar 是可变长度，范围是0~65535。

通过实例验证两种类型区别。

```
create database test1;
use test1
create table a(one char(10), two varchar(10));

# 写入超过10个字符的数据
insert into a (one) values ('abcabcabcabc');

insert into a (two) values ('abcabcabcabc');

# 写入少于10个字符的数据
insert into a (one) values ('abc');

insert into a (two) values ('abc');

# 写入值携带空格的区别
insert into a values ('a  ', 'a  ');
select concat(one, '|'), concat(two, '|') from a;

也就是在写入数据时char不会保留空格
```

* TEXT 和 BLOB

都是保存纯文本数据类型，区别是TEXT保存字符数据， BLOB 保存二进制数据，并且BLOG有个特点，就是可以将图片转换成二进制保存到数据库中，但通常不建议这么使用，会影响数据库性能，在使用临时表查询结果中，如果有TEXT 和 BLOB会使服务器导致磁盘上的表二不是内存中的表，因为内存存储引擎不支持这些数据类型，这会导致检索速度非常慢。

* ENUM 和 SET 

枚举数据类型

写入字段是创建表时所枚举出来的值

```
create table b (name varchar(10), gender enum('男', '女'));
insert into b select 'a', '男';
insert into b select 'b', '女';

insert into b select 'c', '其他';
```

ENUM 类型每个枚举值都有一个索引号，比如第一个指”男“的索引号是1，第二个值的索引号是2，空值索引值为0：
```sql
select * from b where gender = 1;
```

由于索引号的存在，官方也不建议适应数字所谓枚举值，因为这样可能会混淆枚举值和索引号。

SET 类型允许将一个或多个枚举值放在一起：
```
create table c(one set('a', 'b', 'd', 'd'));
insert into c values('a,b,c'), ('a,a', 'b,d,b');
select * from c;

# 写入枚举值之外的
insert into c values ('a,b,e');
```

### 如何选择

类型        应用场景
CHAR        固定长度的字符串
VARCHAR     可变长度的字符串
TEXT        长文本
BLOG        二进制形式的长文本
ENUM        枚举场景，并且只使用一个枚举值
SET         枚举场景，使用多个枚举值