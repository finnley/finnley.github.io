+++
title = 'MySQL入门'
date = 2024-04-07T07:58:38+08:00
draft = false
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



$ tee -a $HOME/.bashrc <<'EOF'
# Go envs
export GOVERSION=go1.18.3 # Go 版本设置
export GO_INSTALL_DIR=$HOME/go # Go 安装目录
export GOROOT=$GO_INSTALL_DIR/$GOVERSION # GOROOT 设置
export GOPATH=$WORKSPACE/golang # GOPATH 设置
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH # 将 Go 语言自带的和通过 go install 安装的二进制文件加入到 PATH 路径中
export GO111MODULE="on" # 开启 Go moudles 特性
export GOPROXY=https://goproxy.cn,direct # 安装 Go 模块时，代理服务器设置
export GOPRIVATE=
export GOSUMDB=off # 关闭校验 Go 依赖包的哈希值
EOF

$ cd $IAM_ROOT
$ tee ca-csr.json << EOF
{
  "CN": "iam-ca",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "marmotedu",
      "OU": "iam"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
EOF

$ cd $IAM_ROOT
$ tee ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "iam": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
EOF

$ sudo tee -a /etc/hosts <<EOF
127.0.0.1 iam.api.marmotedu.com
127.0.0.1 iam.authz.marmotedu.com
EOF

$ cd $IAM_ROOT
$ source scripts/install/environment.sh
$ tee iam-apiserver-csr.json <<EOF
{
  "CN": "iam-apiserver",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "marmotedu",
      "OU": "iam-apiserver"
    }
  ],
  "hosts": [
    "127.0.0.1",
    "localhost",
    "iam.api.marmotedu.com"
  ]
}
EOF

$ cfssl gencert -ca=${IAM_CONFIG_DIR}/cert/ca.pem \
  -ca-key=${IAM_CONFIG_DIR}/cert/ca-key.pem \
  -config=${IAM_CONFIG_DIR}/cert/ca-config.json \
  -profile=iam iam-apiserver-csr.json | cfssljson -bare iam-apiserver
$ sudo mv iam-apiserver*pem ${IAM_CONFIG_DIR}/cert # 将生成的证书和私钥文件拷贝到配置文件目录