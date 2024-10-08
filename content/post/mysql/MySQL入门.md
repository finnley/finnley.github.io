+++
title = 'MySQL入门'
date = 2024-04-07T07:58:38+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## MySQL 语句分类

* DDL（Data Definition Languages）：数据定义语言，这些语句定义了不同的数据库对象。如数据段、数据库、表、列、索引等数据库对象。常用的语句关键字有create、drop、alter 等。

* DML（Data Manipulation Language）：数据操纵语句，主要用来对数据库记录进行增、删、改、查操作，并检查数据库完整性。关键字有 insert、delete、update、select等。

* DCL（Data Control Language）：数据控制语句，用于控制不同数据段直接的访问许可和访问级别的语句，这些语句定义了数据库、表、字段、用户的访问权限和安全级别。关键字有 grant、revoke等。

### DDL语句

对数据库内部对象的创建、修改、删除等操作的语句，与DML的区别在于DML是对表内部操作，而不涉及表的定义、结构的修改。

这里添加个表格，来区分DDL和DML区别

1、创建数据库

对数据库的学习，所有数据存储在数据库中，所以第一个命令就是创建数据库

```sql
create database test1;
```

2、查看数据库

```
show databases;
```

3、选择操作的数据库

```sql
use test1;
```

4、查看数据库下存在哪些表

```sql
show tables;
```

5、删除数据库

```sql
drop database test1;
```

6、创建表

举例：创建一个名为 emp 的表，表中包括 ename（姓名）、hiredate（雇佣日期）、 sal（薪水）和 deptno（部门编号）4个字段。

```sql
create table emp(ename varchar(100), hiredate date, sal decimal(10,2), deptno int(2));
```

7、查看表

```sql
desc emp;
或者
show create table emp \G;
```

却别是 下面的语句查看的信息更全面。比如可以看到字符集和引擎。
\G 是让信息竖向排列，可以展示内容较长的信息。

8、删除表

```sql
delete table emp;
```

9、修改表

主要是修改表结构

```sql

```

## 索引组织表（Index Organized Table）

* 索引组织表不是一种“组织表”，而是由索引“组织起来的”表，即这个这个表数据都是由索引组成的
* InnoDB中，表都是根据 `主键` 顺序组织存放的

### 索引（Index）

* 索引是数据库中对某一列或多列的值进行预排序的数据结构

预排序就是数据在存入表的时候就按照某个值的顺序进行排列

* 索引可以理解为数据的“目录”

####


### 主键（Primary Key）

* InngoDB 存储引擎表中，每张表都有一个主键
* InnoDB 中，主键是一个特殊的索引字段
* 若表中有一个非空唯一索引（Unique Not Null），即为主键
* 若有多个非空唯一索引，选择第一个定义的索引
* 若无，InnoDB 自动创建一个6字节的指针，作为主键


#### 案例

* 下面SQL建表语句中，哪一列是主键？(d)

```sql
CREATE TABLE Z (
  a INT NOT NULL,
  b INT NULL,
  c INT NOT NULL,
  d INT NOT NULL,
  UNIQUE KEY (b),
  UNIQUE KEY (d),
  UNIQUE KEY (c)
);
```

建库：

```sql
CREATE DATABASE `d1` CHARCTER SET 'utf8mb' COLLATE 'utf8mb4_general_ci';
use d1;
desc Z;
```

### B+树“B”是什么？

