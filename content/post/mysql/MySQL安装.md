+++
title = 'MySQL安装'
date = 2024-04-30T07:28:19+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## Bundle 方式安装

### 下载

1、进入官网 https://www.mysql.com/ ，选择 【DWONLOADS】：

![](/img/mysql/10.png)

2、在页面下方选择 “MySQL Community(GPL) Downloads” 社区版下载：

![](/img/mysql/20.png)

3、点击【MySQL Community Server】 ：

![](/img/mysql/30.png)

4、点击【Archives】选项卡，选择想要下载的版本，然后复制下载链接进行下载：

比如：

![](/img/mysql/40.png)

```
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar
```

### 解压

```
tar -xvf mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar
```

解压后得到以下文件：

```
mysql-community-client-5.7.44-1.el7.x86_64.rpm
mysql-community-common-5.7.44-1.el7.x86_64.rpm
mysql-community-devel-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.44-1.el7.x86_64.rpm
mysql-community-libs-5.7.44-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.44-1.el7.x86_64.rpm
mysql-community-server-5.7.44-1.el7.x86_64.rpm
mysql-community-test-5.7.44-1.el7.x86_64.rpm
```

### 安装

安装时有两个包最重要，一个是 `mysql-community-server` 和 `mysql-community-client`，在安装这两个之前先安装下 `common` 包，安装顺序也很重。

1、安装 common 包

```
# rpm -ivh mysql-community-common-5.7.44-1.el7.x86_64.rpm
```

2、安装 libs，不是 libs-compat

```
# rpm -ivh mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) >= 5.7.9 is needed by mysql-community-libs-5.7.44-1.el7.x86_64
	mariadb-libs is obsoleted by mysql-community-libs-5.7.44-1.el7.x86_64
```

出现 `mariadb-libs is obsoleted by mysql-community-libs-5.7.42-1.el7.x86_64` 说明本地可能存在 `mariadb` ，需要先将 `mariadb` 删除。

```
# rpm -qa | grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
# rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
#
```

`--nodeps` 表示不检查依赖。

重新安装 libs：

```
# rpm -ivh mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) >= 5.7.9 is needed by mysql-community-libs-5.7.44-1.el7.x86_64
```

又出现了一个错误，这是因为存在签名校验，可以选择不校验签名，强制安装：

```
# rpm -ivh --force --nodeps mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-libs-5.7.44-1.el7################################# [100%]
```

3、安装 MySQL Client

```
# rpm -ivh mysql-community-client-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-client-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-client-5.7.44-1.e################################# [100%]
```

4、安装 MySQL Server

```
# rpm -ivh mysql-community-server-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-server-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	/usr/bin/perl is needed by mysql-community-server-5.7.44-1.el7.x86_64
	libaio.so.1()(64bit) is needed by mysql-community-server-5.7.44-1.el7.x86_64
	libaio.so.1(LIBAIO_0.1)(64bit) is needed by mysql-community-server-5.7.44-1.el7.x86_64
	libaio.so.1(LIBAIO_0.4)(64bit) is needed by mysql-community-server-5.7.44-1.el7.x86_64
	mysql-community-common(x86-64) = 5.7.44-1.el7 is needed by mysql-community-server-5.7.44-1.el7.x86_64
	perl(Getopt::Long) is needed by mysql-community-server-5.7.44-1.el7.x86_64
	perl(strict) is needed by mysql-community-server-5.7.44-1.el7.x86_64
```

页面提示缺少 libaio、perl 和 net-tools 包，提示缺什么包就安装什么包，视情况而定。

```
# yum install -y perl net-tools libaio
```

重新安装：

```
# rpm -ivh mysql-community-server-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-server-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) = 5.7.44-1.el7 is needed by mysql-community-server-5.7.44-1.el7.x86_64
```

还是打包秘钥问题，可以强制安装：

```
# rpm -ivh --force --nodeps mysql-community-server-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-server-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-server-5.7.44-1.e################################# [100%]
```

5、验证

```
# mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```

可以正常使用，上面提示并不是安装失败，而是 server 还没有启动，无法通过client连接。

### 启动

1、MySQL 默认配置路径是 `/etc/my.cnf`，打开 `/etc/my.cnf` 文件，在 [mysqld] 下添加一行 `skip-grant-tables`

这句话作用是跳过校验。原因是 MySQL5.7 在启动时会设置一个默认的 root 密码，我们必须查看log才能知道 root 密码，添加这样一行就是将所有的校验都去掉。为的就是可以直接进入 mysql，进去之后再修改密码。

![](/img/mysql/50.png)

生产环境切记不可如此。因为开放的这段时间容易被攻击。

2、启动

```
# systemctl start mysqld.service
```

3、连接

```
[root@localhost ~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.44 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

此时不会经过任何校验就可连接上 MySQL


4、设置 root 密码

```sql
update mysql.user set authentication_string=password('123123') where user='root';
flush privileges;
```

5、退出客户端，重启MySQL

重启之前需要将之前 `/etc/my.cnf` 配置文件中的 `skip-grant-tables` 去掉，然后再重启。

```
# systemctl restart mysqld.service
```

重启之后再尝试通过 `mysql` 关键字直接连接 MySQL，发现已经无法连接，需要通过密码连接。

```
# mysql -u root -p123123
```

![](/img/mysql/60.png)


此时下发查询SQL会发现无法查询，如下图:

![](/img/mysql/70.png)

6、设置密码策略以及密码长度（仅应用于开发学习）

```sql
set global validate_password_policy=LOW;
```

意思是将密码的复杂程度设置低一些。

```sql
set global validate_password_length=4;
```

意思是将密码的长度设置低，以便学习开发时设置较短的密码长度。

接着重新设置下密码：

```sql
set password=password('123123');
```

密码设置完后再下发查询SQL，发现可以正常使用了：

![](/img/mysql/80.png)

7、允许外部连接

比如使用 Navicat 连接 MySQL 时会发现连接不上，这是因为 centos 是有防火墙的，可以进入Centos 设置防火墙开放3306端口，操作如下：

```shell
systemctl status firewalld
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

如果开放了端口还是无法连接MySQL，还需要登录进 myqsl，开启允许远程登录，操作如下：

```sql
grant all privileges on *.* to 'root'@'%' identified by '123123' with grant option;
```

意思是使用root用户和123123这个密码可以远程登录到MySQL里面。
