+++
title = 'MySQL安装'
date = 2024-04-30T07:28:19+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## Bundle 方式安装

### 下载

1、进入官网 https://www.mysql.com/ ，选择 【DWONLOADS】
![](/img/mysql/10.png)

2、在页面下方选择 “MySQL Community(GPL) Downloads” 社区版下载
![](/img/mysql/20.png)

3、点击【MySQL Community Server】
![](/img/mysql/30.png)

4、点击【Archives】选项卡，选择想要下载的版本，然后复制下载链接进行下载，如：
![](/img/mysql/40.png)

```bash
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar
```

### 解压

```bash
tar -xvf mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar
```

解压后得到以下文件：
```bash
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

安装时有两个包最重要，分别是 `mysql-community-server` 和 `mysql-community-client`，在安装这两个之前先安装下 `common` 包，安装顺序也很重要。

1、安装 common 包
```bash
rpm -ivh mysql-community-common-5.7.44-1.el7.x86_64.rpm
```

2、安装 libs 包，不是 libs-compat 包
```bash
# rpm -ivh mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) >= 5.7.9 is needed by mysql-community-libs-5.7.44-1.el7.x86_64
	mariadb-libs is obsoleted by mysql-community-libs-5.7.44-1.el7.x86_64
```

出现 `mariadb-libs is obsoleted by mysql-community-libs-5.7.42-1.el7.x86_64` 说明本地可能存在 `mariadb` ，需要先将 `mariadb` 删除：
```bash
# rpm -qa | grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
# rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
#
```

`--nodeps` 表示不检查依赖。

删除完后重新安装 libs：
```bash
# rpm -ivh mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) >= 5.7.9 is needed by mysql-community-libs-5.7.44-1.el7.x86_64
```

又出现了一个错误，这是因为存在签名校验，可以选择不校验签名，强制安装：
```bash
# rpm -ivh --force --nodeps mysql-community-libs-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-libs-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-libs-5.7.44-1.el7################################# [100%]
```

3、安装 MySQL Client
```bash
# rpm -ivh mysql-community-client-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-client-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-client-5.7.44-1.e################################# [100%]
```

4、安装 MySQL Server
```bash
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

页面提示缺少 libaio、perl 和 net-tools 包，一般提示缺什么包就安装什么包，视情况而定。
```bash
# yum install -y perl net-tools libaio
```

重新安装：
```bash
# rpm -ivh mysql-community-server-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-server-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
error: Failed dependencies:
	mysql-community-common(x86-64) = 5.7.44-1.el7 is needed by mysql-community-server-5.7.44-1.el7.x86_64
```

还是打包秘钥问题，强制安装：
```bash
# rpm -ivh --force --nodeps mysql-community-server-5.7.44-1.el7.x86_64.rpm
warning: mysql-community-server-5.7.44-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-server-5.7.44-1.e################################# [100%]
```

5、验证
```bash
# mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```

安装成功，上面提示并不是安装失败，而是 MySQL Server 还没有启动，无法通过 MySQL Client 连接。

### 启动

1、MySQL 默认配置路径是 `/etc/my.cnf`，打开 `/etc/my.cnf` 文件，在 [mysqld] 下添加一行 `skip-grant-tables`

这句话作用是跳过校验。原因是 MySQL5.7 在启动时会设置一个默认的 root 密码，我们必须查看 log 文件才能知道 root 密码，添加这样一行就是将所有的校验都去掉。为的就是可以直接进入 MySQL，进去之后再修改密码。
![](/img/mysql/50.png)

生产环境切记不可如此，因为开放的这段时间容易被攻击。

2、启动
```bash
# systemctl start mysqld.service
```

3、连接
```bash
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

此时不会经过任何校验（输入用户名和密码）就可连接上了 MySQL。


4、设置 root 密码
```sql
update mysql.user set authentication_string=password('123123') where user='root';
flush privileges;
```

5、退出客户端，重启MySQL。

重启之前需要将之前 `/etc/my.cnf` 配置文件中的 `skip-grant-tables` 去掉，然后再重启。
```bash
# systemctl restart mysqld.service
```

重启之后再尝试通过 `mysql` 关键字直接连接 MySQL，发现已经无法连接，此时则需要通过密码连接。
```bash
# mysql -u root -p123123
```
![](/img/mysql/60.png)


此时下发查询SQL会发现无法查询：
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

7、允许外部连接。

比如使用 Navicat 连接 MySQL 时会发现连接不上，这是因为 CentOS 系统存在防火墙，可以进入 CentOS 设置防火墙开放 3306 端口：
```shell
systemctl status firewalld
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

如果开放了端口还是无法连接 MySQL：
![](/img/mysql/90.png)

这是因为MySQL未开启允许远程登录，需要进入 MySQL 进行如下操作：
```sql
grant all privileges on *.* to 'root'@'%' identified by '123123' with grant option;
```

意思是使用 “root” 用户和 “123123” 这个密码可以远程登录到MySQL里面。然后再尝试连接就可以成功连接了。
![](/img/mysql/100.png)


## Compressed TAR Archive 方式

1、下载
```bash
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.32-linux-glibc2.17-aarch64.tar.gz
```

2、解压
```bash
tar zxvf mysql-8.0.32-linux-glibc2.17-aarch64.tar.gz
```

3、移动
```bash
mv mysql-8.0.32-linux-glibc2.17-aarch64 /opt/mysql
```

4、添加mysql用户组
```bash
groupadd mysql
```

5、建立系统账号
```bash
useradd -r -g mysql -s /bin/false mysql
```

`-g` 表示指定用户组
`/bin/false` 表示该不能登录

6、创建mysql目录
```bash
mkdir -p /data/mysql
mkdir -p /data/mysql/{binlog,data,log,tmpdir,conf}
```

`binlog` 存放二进制日志目录
`data` 存在数据目录
`log` 存在错误日志、慢日志目录
`tmpdir` 临时文件存放目录
`conf` 存放配置文件目录

7、修改属主
```bash
chown -R mysql.mysql /data/mysql
chown -R mysql.mysql /usr/local/mysql
```

8、添加配置文件
```bash
vim /data/mysql/conf/my.cnf
```

配置内容如下：
https://www.yuque.com/u8058753/pwvn3w/zedon838xflq5xg2?singleDoc# 《/data/mysql/conf/my.cnf》

9、初始化bing获取临时密码
```bash
/opt/mysql/bin/mysqld --defaults-file=/data/mysql/conf/my.cnf --user=mysql --initialize
```

`--user=mysql` 表示MySQL进程运行时使用的用户

回车后则开始进行初始化，目的有两个：

* 创建默认库，可以到 /data/mysql/data 目录下查看
* 创建一个随机过期的的密码的超级用户，并将这个密码和该用户记录到错误日志中

10、查看创建的超级用户和密码
```bash
# grep password /data/mysql/log/mysql.err
2024-06-23T07:47:08.980554Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qwD7j*cFl.1u
#
```

11、启动MySQL

首先配置启动脚本：
```bash
cp /opt/mysql/support-files/mysql.server /etc/init.d/
```

编辑启动脚本：
```bash
vim /etc/init.d/mysql.server
```

找到 `/datadir` 并配置：
```
datadir=/data/mysql/data
confdir=/data/mysql/conf
```

找到 `/mysqld_safe`，找到 `$bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &` 这行，修改后如下：
```bash
$bindir/mysqld_safe --defaults-file=$confdir/my.cnf --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &
```

找到 `datadir=\*`，找到 `--datadir=*)  datadir=`echo "$arg" | sed -e 's/^[^=]*=//'`` 这样，修改后如下：
```shell
--datadir=*)  datadir="/data/mysql/data"
```

启动 MySQL：
```bash
# /etc/init.d/mysql.server start
Starting MySQL.... SUCCESS!
#
```

查看MySQL是否启动成功：
```bash
# ps -ef | grep mysql
root       292     1  0 08:00 pts/1    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --defaults-file=/data/mysql/conf/my.cnf --datadir=/data/mysql/data --pid-file=/data/mysql/data/chaos-1.pid
mysql     1544   292  6 08:00 pts/1    00:00:02 /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/conf/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/log/mysql.err --open-files-limit=28192 --pid-file=/data/mysql/data/chaos-1.pid --socket=/tmp/mysql.sock --port=3306
root      1601     6  0 08:01 pts/1    00:00:00 grep --color=auto mysql
```

12、设置环境变量
```bash
vim /etc/profile
```

在最后添加下面配置：
```bash
MYSQL_HOME=/usr/local/mysql
PATH=$PATH:$MYSQL_HOME/bin
export PATH MYSQL_HOME
```

保存退出后执行 `source /etc/profile`。然后查看 MySQL版本：
```bash
# mysql --version
mysql  Ver 8.0.32 for Linux on aarch64 (MySQL Community Server - GPL)
```

13、登录 MySQL 并修改密码
```bash
#  grep password /data/mysql/log/mysql.err
2024-06-23T07:47:08.980554Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qwD7j*cFl.1u
[root@chaos-1 data]# mysql -uroot -p'qwD7j*cFl.1u'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.32

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql>
```

这是临时的超级用户，登录后并不能正常工作，需要先修改密码：
```bash
mysql> alter user user() identified by 'finnley';
Query OK, 0 rows affected (0.01 sec)

mysql>
```

接着退出，再使用新密码登录：
```bash
[root@chaos-1 data]# mysql -uroot -p
Enter password: [这里输入的是刚才设置的密码]
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql>
```

14、关闭MySQL
```bash
[root@chaos-1 data]# /etc/init.d/mysql.server stop
Shutting down MySQL.. SUCCESS!
[root@chaos-1 data]# ps -ef | grep mysql
root       292     1  0 08:00 pts/1    00:00:00 [mysqld_safe] <defunct>
root      1659     6  0 08:12 pts/1    00:00:00 grep --color=auto mysql
[root@chaos-1 data]#
```

发现 MySQL 进程已经不存在了。

## Shell 脚本部署

手动安装存在的缺点就是安装比较耗时。

### 准备

创建脚本部署目录:
```bash
mkdir -p /data/script/install_mysql8
mkdir -p /data/mysql/conf/my.cnf /data/script/install_mysql8
cp /opt/mysql-8.0.32-linux-glibc2.17-aarch64.tar.gz /data/script/install_mysql8
cp /etc/init.d/mysql.server /data/script/install_mysql8
```

### 编辑安装脚本

```bash
#!/bin/bash

# 解压
if [ -d "/usr/local/mysql" ];then
	echo "/usr/local/mysql 文件夹已存在，请确认是否安装了 MySQL"
	exit
fi

echo "正在解压压缩包..."
tar xf mysql-8.0.32-linux-glibc2.17-aarch64.tar.gz
mv mysql-8.0.32-linux-glibc2.17-aarch64 /usr/local/mysql
echo "压缩包解压结束"

# 创建MySQL目录
if [ -d "/data/mysql/" ];then
	echo "/data/mysql 文件夹已存在，请确认是否安装了 MySQL"
	exit
fi
mkdir -p /data/mysql/{binlog,data,log,tmpdir,conf}

# 判断MySQL进程是否启动
mysql_pid=`ps -ef | grep mysqld | wc -l`
if [ $mysql_pid -eq 1 ];then
	echo "MySQL 进程不存在"
else
	echo "存在 MySQL 运行进程，请检查"
	exit
fi

# 创建MySQL用户
mysql_user=`cat /etc/passwd | grep -w mysql | wc -l`
if [ $mysql_user -eq 1 ];then
	echo "MySQL 用户已存在"
else
	echo "MySQL 用户不存在，开始添加 MySQL 用户"
	groupadd mysql
	useradd -g mysql mysql
	echo "添加 MySQL 用户成功"
fi

# 修改权限
chown -R mysql.mysql /data/mysql/
chown -R mysql.mysql /usr/local/mysql

# 添加配置文件
cp ./my.cnf /data/mysql/conf/

# 初始化
echo "开始初始化"
/usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/conf/my.cnf --user=mysql --initialize

# 判断初始化是否成功
mysql_init=`cat /data/mysql/log/mysql.err | grep -i "root@localhost:" | wc -l`
if [ $mysql_init -eq 1 ];then
	echo "MySQL 初始化成功"
else
	echo "MySQL 初始化失败"
	exit
fi

# 初始化临时密码
tmp_pwd=$(grep 'temporary password' /data/mysql/log/mysql.err)
## 过滤临时密码
pwd=${tmp_pwd##* }
echo "临时密码: ${pwd}"

# 配置启动脚本
if [ ! -f "/etc/init.d/mysql.server" ];then
	cp mysql.server /etc/init.d/ -rf
	chmod 700 /etc/init.d/mysql.server
fi

# 启动MySQL
/etc/init.d/mysql.server start

# 添加环境变量
mysql_path=`grep 'export PATH=$PATH:/usr/local/mysql/bin' /etc/profile | wc -l`
if [ $mysql_path -eq 0 ];then
	echo "export PATH=\$PATH:/usr/local/mysql/bin" >> /etc/profile
	source /etc/profile
fi

## 使用临时密码登录 MySQL，并修改密码
mysql -uroot -p${pwd} --connect-expired-password -e "alter user user() identified by '123';"
echo "MySQL8 安装完成"

```

### 执行
```bash
./install_mysql.sh
正在解压压缩包...
压缩包解压结束
MySQL 进程不存在
MySQL 用户已存在
开始初始化
MySQL 初始化成功
临时密码: 0uozxGqdF_BU
Starting MySQL... SUCCESS!
mysql: [Warning] Using a password on the command line interface can be insecure.
MySQL8 安装完成
```

### 登录

```bash
# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql>
```

脚本仍存在很多优化空间，后续需要对配置脚本优化。

## ChatGPT 提供的方法安装

略

修改 root 密码：
```bash
./bin/mysql -u root mysql
alter user 'root'@'localhost' identified by 'new_password';
flush privileges;
exit;
```