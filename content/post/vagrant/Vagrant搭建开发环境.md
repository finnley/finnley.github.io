+++
title = 'Vagrant搭建开发环境'
date = 2019-05-02T11:01:55+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++


## 准备

* 本地环境 `Ubuntu18.04 LTS`
* 镜像 `Ubuntu18.04.box`

## 步骤

### 下载镜像

1. 比如我需要下载 `ubuntu/bionic64` ，可以打开官网 [ubuntu/bionic64](https://app.vagrantup.com/ubuntu/boxes/bionic64) 进行下载，图中的 `v20190501.0.0` 就是可以下载的版本号

![vagrant](/images/vagrant/48.png)


2. 点击上图进入版本详情页 [v20190501.0.0](https://app.vagrantup.com/ubuntu/boxes/bionic64/versions/20190501.0.0)

3. 拼接URL 

  `https://app.vagrantup.com/centos/boxes/7/versions/1804.02` + `/providers/` + `供应商名字/` + `.box`

公式：下载链接 = 产品版本链接 + 供应商英文意思 + 要下载的供应商名称（如virtualbox）+'.box'

如上：生成的下载链接为：

`https://app.vagrantup.com/ubuntu/boxes/bionic64/versions/20190501.0.0/providers/virtualbox.box`


```
sudo wget https://app.vagrantup.com/ubuntu/boxes/bionic64/versions/20190501.0.0/providers/virtualbox.box
```

### 初始化系统

1. 将下载的镜像放到指定目录中，我存放的目录是 `/mnt/f/vagrant/box`

![vagrant](/images/vagrant/49.png)

2. 查看是否已经将镜像添加到vagrant中

```
vagrant box list
```

![vagrant](/images/vagrant/50.png)

此时镜像列表中并没有发现 `bionic` 的镜像

3. 添加镜像

命令：

```
vagrant box add <自己命名的名称> <镜像名称>
```

比如：

```
vagrant box add bionic bionic-server-cloudimg-amd64-vagrant.box
```

添加完之后再次查看是否已经成功添加到vagrant

```
vagrant box list
```

![vagrant](/images/vagrant/51.png)

4. 创建一个名为 `bionic` 的目录

```
cd ～/vagrant/
sudo mkdir bionic
sudo chmod 777 bionic/
```

5. 初始化

命令： 

```
vagrant init <镜像名称>
```

比如：

```
cd bionic/
vagrant init bionic
```

![vagrant](/images/vagrant/52.png)

此时会在 `bionic` 目录中看到生成了一个 `Vagrantfile` 的配置文件

6. 打开 `Vagrantfile` 文件，会看到 `config.vm.box = "bionic"` ， 其中 `bionic` 就是在上面初始化的时候也就是 `init` 后面输入的 `bionic` ,如果初始化的时候 `init` 后面没有接受参数后面可以在这个配置文件中改变这个值。

7. 启动虚拟机

```
vagrant up
```

如图，一个虚拟机新建并启动完成。

![vagrant](/images/vagrant/53.png)


## 环境配置

此时虚拟机已经新建完成，进入虚拟机

```
vagrant ssh
```

![vagrant](/images/vagrant/54.png)

现在已经进入到了虚拟机内部，当前系统是 `Ubuntu 18.04`。

### 替换源

1. 首先备份源

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

![vagrant](/images/vagrant/55.png)

2. 编辑源

```
sudo vim /etc/apt/sources.list
```

使用 `:.,$d` 删除里面内容替换为下面的阿里源,保存退出

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

3. 更新源

```
sudo apt-get update
```

## 安装软件

### 安装Nginx

1. 查看系统是否已经存在nginx

```
sudo apt-cache search nginx
```

2. 在/etc/apt目录下下载nginx_signing.key

```
wget http://nginx.org/keys/nginx_signing.key 
sudo apt-key add nginx_signing.key
```

![vagrant](/images/vagrant/56.png)

3. 对于Ubuntu，将codename替换为 Ubuntu 分发代号，并将以下代码附加到 `/etc/apt/sources.list` 文件末尾，然后保存退出

```
deb http://nginx.org/packages/ubuntu/ codename nginx 
deb-src http://nginx.org/packages/ubuntu/ codename nginx
```

例如：

```
deb http://nginx.org/packages/ubuntu/ bionic nginx 
deb-src http://nginx.org/packages/ubuntu/ bionic nginx
```

![vagrant](/images/vagrant/57.png)

4. 更新系统

```
sudo apt-get update
```

5. 安装

```
sudo apt-get install -y nginx
```

![vagrant](/images/vagrant/58.png)

6. 查看是否安装成功

```
nginx -v
```

![vagrant](/images/vagrant/59.png)

7. 查看nginx状态

```
sudo systemctl status nginx
```

8. 开启nginx服务并再次查看状态

```
sudo systemctl start nginx
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/60.png)
</div>

9. 测试

```
curl -I 'http://127.0.0.1'
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/61.png)
</div>

10. 可以使用下面命令将nginx服务关闭

```
sudo /etc/init.d/nginx stop
```

或者使用 `ps -ef | grep nginx` 查看nginx状态

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/62.png)
</div>


### 安装MaraiaDB

1. 打开官网 `https://downloads.mariadb.org/mariadb/repositories/#mirror=limestone` ， 找对对应的版本，我这里选择的是 `Ubuntu 18.04` 的版本 `MariaDB 10.3`

2. 打开 `/etc/apt/sources.list` 文件，在末尾添加下面内容

```
# MariaDB 10.3 repository list - created 2019-05-02 08:16 UTC
# http://downloads.mariadb.org/mariadb/repositories/
deb [arch=amd64,arm64,ppc64el] http://mirror.lstn.net/mariadb/repo/10.3/ubuntu bionic main
deb-src http://mirror.lstn.net/mariadb/repo/10.3/ubuntu bionic main
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/63.png)
</div>

3. 执行下面内容

```
sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mirror.lstn.net/mariadb/repo/10.3/ubuntu bionic main'
```

4. 更新并安装

```
sudo apt-get update
sudo apt install mariadb-server
```

期间需要两次设置数据库密码

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/83.png)
</div>

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/84.png)
</div>

5. 安装安装

```
mysql_secure_installation
```

```
vagrant@ubuntu-bionic:~$ mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

6. 测试

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/64.png)
</div>

### 安装PHP7

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install -y php7.3 php7.3-json php7.3-xml php7.3-fpm php7.3-cli  php7.3-gd php7.3-mysql php7.3-curl php7.3-pgsql php7.3-mbstring php7.3-cgi php7.3-dev
```

安装 `php7.3-dev` 是用来使用 `phpize` 的

查看版本

```
php -v 
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/65.png)
</div>


## 高级配置

### 端口转发

官网 `https://www.vagrantup.com/docs/networking/forwarded_ports.html`

1. 打开目录下的配置文件

```
sudo vim Vagrantfile 
```

在 `config.vm.box = "bionic"` 这句话下面添加下面端口转发指令

```
config.vm.network "forwarded_port", guest: 80, host: 8080
```

表示我本地浏览器可以通过 `8080` 端口访问虚拟机 `80` 端口

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/66.png)
</div>

2. 重启虚拟机

```
vagrant reload
```

3. 测试

在本地浏览中输入 `127.0.0.1:8080`

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/67.png)
</div>

### 配置共享目录（挂载）

官网 `https://www.vagrantup.com/docs/synced-folders/nfs.html`

1. 打开目录下的配置文件

```
sudo vim Vagrantfile 
```

在 `config.vm.network "forwarded_port", guest: 80, host: 8080` 这句话下面添加下面共享目录指令

```
 config.vm.synced_folder "/mnt/f/www/", "/home/www", :nfs => true
```

表示将我本机的 `/mnf/f/www/` 目录映射到虚拟机内的 `/home/www` 目录中

进入虚拟机查看 `/home/www` 目录是否存在

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/68.png)
</div>


2. 重启

```
vagrant reload
```

3. 在 ` config.vm.synced_folder "/mnt/f/www/", "/home/www", :nfs => true` 添加私有网络配置, 因为设置 `nfs` 的时候一定要设置一个静态IP， 如果配置中将 `:nfs => true` 去掉的话，就不需要设置静态IP了

```
config.vm.network "private_network" ,ip: "192.168.43.43"
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/69.png)
</div>

4. 重启

```
vagrant reload
```

5. 使用 `vagrant ssh` 进入虚拟机，此时会看到 `/home/www` 目录已经存在了

6. 在虚拟机中可以使用 `ifconfig` 查看网络配置

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/70.png)
</div>

7. 此时就可以使用IP来访问

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/71.png)
</div>

### 修改虚拟机名称（虚拟机里面主机的名称）

打开Virtual,在虚拟机列表中显示的就是虚拟机名称

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/72.png)
</div>

这里名称太长了，所以需要修改，这里修改为 `bionic`

打开配置文件找到 `config.vm.provider "virtualbox" do |vb|` 这一段，将注释去掉

修改前：

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/73.png)
</div>

修改后：

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/74.png)
</div>

重启

```
vagrant reload
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/75.png)
</div>

### 修改虚拟机主机名称

进入虚拟机

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/76.png)
</div>

如图所示 `ubuntu-bionic` 就是主机名

只需要在配置文件中添加 ` config.vm.hostname = "vmubuntu"` ，然后重启

### 配置虚拟机内容和CPU

使用 `free -m` 查看内存,使用 `top` 命令，然后输入 `1` 查看CPU， 输入 `q` 退出

配置虚拟机内存和CPU是在添加名称的地方进行修改的

```
vb.memory = "1024"
vb.cpus = 2
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/77.png)
</div>

然后重启 `vagrant reload`


### 优化Nginx

官网 `https://www.vagrantup.com/docs/synced-folders/virtualbox.html`

在nginx配置中添加 `sendfile off;` , 但是在 `Apache` 中已经默认关了

```
sudo vim nginx.conf 
```

找到 `sendfile        on;` , 将其设置为 `off`

重启nginx服务

```
sudo /etc/init.d/nginx restart
```

## 打包分发

1. 打包之前需要将虚拟机关闭

```
vagrant halt
```

2. 打包

```
vagrant package
```

如果没有输入参数它会自动生成一个 `package.box` 的文件

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/78.png)
</div>

如果带参数

```
vagrant package --output ubuntu1804.box
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/79.png)
</div>

3. 可以通过U盘或者网盘进行拷贝分发，这里介绍通过配置文件进行分发

比如这里我需要在虚拟机中安装redis

打开配置文件，找到下面配置块

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/80.png)
</div>

修改如下：

```
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
   config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
    sudo apt-get install -y redis-server
   SHELL
```

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/81.png)
</div>

重启,这样就会执行安装redis的命令

```
vagrant reload --provision
```

4. 进入虚拟机使用 `ps -ef | grep redis` 查看是否安装

<div align=center>
![vagrant](http://images.ornnth.com/hexo/vagrant/82.png)
</div>


5. 在打包的时候需要将网络配置全部关闭或者在设置了网络配置的情况下添加一个 `， auto_config: true` 这个参数