+++
title = 'Vagrant高级应用'
date = 2018-12-21T23:35:47+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++

## 转发端口配置

1. 进入之前创建的目录lanmp,打开virtualbox，查看网络配置

![vagrant](/images/vagrant/19.png)

2. 现在重启虚拟机

```
vagrant reload
```

3. 然后再去查看虚拟机网络

![vagrant](/images/vagrant/20.png)

发现之前设置网络设置里面的端口转发消失了。这是因为vagrant每次重启的时候都会按照自己的配置文件进行设置。

4. 现在编辑目录下的Vagrantfile文件

```
sudo vim Vagrantfile
```

5. 在 `config.vm.box = "ubuntu1404"` 下面添加下面端口转发的配置

```
config.vm.network "forwarded_port", guest: 80, host: 8888
config.vm.network "forwarded_port", guest: 8888, host: 8889
```

![vagrant](/images/vagrant/21.png)

6. `vagrant reload` 重启虚拟机

![vagrant](/images/vagrant/22.png)

配置的端口转发就又出来了，现在打开浏览器进行测试。

![vagrant](/images/vagrant/23.png)

## 配置共享目录

1. 编辑目录下的 `Vagrantfile` 文件

```
sudo vim Vagrantfile
```

2. 继续在刚才添加的下面继续添加下面这句话

```
config.vm.synced_folder "/mnt/f/www/", "/home/www", :nfs => true
```

![vagrant](/images/vagrant/24.png)

3. 重启 `vagrant reload`, 在重启后会出现下面错误页面

![vagrant](/images/vagrant/25.png)

因为我的本机是 `Ubuntu` ，部分 `Linux` 需要安装对应 `package` 才能支持（以 Ubuntu 为例）：

```
sudo apt-get install nfs-kernel-server nfs-common 
```

虚拟机中需要执行下面命令进行安装 ,但是会提示已经安装过了，无伤大雅

```
sudo apt-get install nfs-common
```

4. 然后再去执行 `vagrant reload`，然是会提示下面错误

![vagrant](/images/vagrant/26.png)

这是因为我们之前配置了共享文件夹，共享文件夹只能和私有网络进行配合使用，所以我们这里将共享文件夹配置注释掉，然后再次重启

5. 继续编辑配置文件，配置私有网络

```
config.vm.network "private_network" ,ip: "192.168.199.101"
```

![vagrant](/images/vagrant/27.png)

6. 然后再次重启虚拟机

![vagrant](/images/vagrant/28.png)

此时进入虚拟机输入 `ll /home/www` 此时就会出现了`www`目录了，并且里面有很多我们主机文件

## 网络配置

下面是刚才设置的网络

![vagrant](/images/vagrant/29.png)

下面就直接可以使用配置的IP再浏览器中访问

![vagrant](/images/vagrant/30.png)

配置共有网络 不能与自己本地的IP重复，如我本机IP192.168.1.100，则配置的共有网络可以是192.168.1.101

```
config.vm.network "public_network" ,ip: "192.168.1.101"
```

![vagrant](/images/vagrant/31.png)

然后重启

中途会让你选择网络链接方式，我输入的是1，最后退爆出下面错误

![vagrant](/images/vagrant/32.png)

这是因为我们之前配置了共享文件夹，共享文件夹只能和私有网络进行配合使用，所以我们这里讲共享文件夹配置注释掉，然后再次重启

![vagrant](/images/vagrant/33.png)

![vagrant](/images/vagrant/34.png)

此时IP共享文件夹仍然存在，但是里面文件不见了

下面仍然改为私有网络，共享文件夹

![vagrant](/images/vagrant/35.png)

## 修改虚拟机名称

编辑　目录下的Vagrantfile文件

```
sudo vim Vagrantfile
```

找到下面这一行代码并将注释去掉

```
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
```

修改后如下面所示

```
   config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
    vb.name = "ubuntu1404"
   end
```

然后重启虚拟机

```
vagrant reload
```

此时发现虚拟机的名称已经改变了，再也不是之前的长长的名称了

![vagrant](/images/vagrant/42.png)

## 修改主机名称

我们进入到虚拟机内主机中发现终端会显示下面`vagrant@vagrant-ubuntu-trusty-64:~$`,而 `@`后面的就是主机的名称

编辑　目录下的Vagrantfile文件

```
sudo vim Vagrantfile
```

在之前配置的网络配置下面添加下面一句话

```
config.vm.hostname = "vmubuntu"
```

![vagrant](/images/vagrant/43.png)

然后重启虚拟机

```
vagrant reload
```

重启完成后再次进入虚拟机,在终端可以看到主机名已经改变了

![vagrant](/images/vagrant/44.png)

## 修改虚拟机内存和CPU

我们可以使用下面命令查看虚拟机内存

```
free -m
```

![vagrant](/images/vagrant/45.png)

我们可以使用下面命令查看虚拟机CPU

```
top
```

修改虚拟机内存和实在刚才配置虚拟机名称的地方进行修改

```
vb.memory = "1024"
vb.cpus = 2
```

![vagrant](/images/vagrant/46.png)

然后再重启虚拟机


## 优化nginx配置

打开nginx配置文件 `nginx.conf`

```
sudo vim /etc/nginx/nginx.conf
```

找到 `sendfile on;`  将其修改为 `sendfile off;` 然后重启nginx服务，这样就可以将本地写的代码很快的同步到虚拟机中，而Apache2默认已经进行设置了。


## 打包分发

打包分发是将配置的lanmp进行打包，最后打包成.box结尾的镜像，然后分发是将打包好的镜像给别人使用。

打包命令

```
#打包
vagrant package --output xxx.box

#分发
vagrant package --output xxx.box --base 虚拟机名称
```

在分发过成功可能会遇到一个问题，就是如果box升级了怎么办，比如说我们在虚拟机中又安装了一个软件redis, 对于老用户可以使用Vagrantfile来更新，对于新用户可以直接使用将新打包的box文件新建虚拟机

打包之前需要将虚拟机关闭

```
#关机
vagrant halt

#打包 如果不输入任何参数 它会生成一个package.box的文件
vagrant package
```

```
#打包 输入参数的情况
vagrant package --output lanmp.box
```

![vagrant](/images/vagrant/47.png)

通过Vagrantfile更新

编辑　目录下的Vagrantfile文件

```
sudo vim Vagrantfile
```

找到下面这一段

```
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
```

打开中注释，它有一个实例，是安装Apache2的，我们增加一个安装redis的

```
   config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
      apt-get install -y redis-server
   SHELL
```

然后重启,这样它就会执行方才写安装命令

```
vagrant reload --provision
```


对于新用户完全可以进入到虚拟机自己去安装redis,然后再去重新打包分发

现在如果我拿到别人分发给我的box,我如何去使用呢

```
vagrant box add lanmp lanmp.box
vagrant box list
vagrant init lanmp
vagrant up
```

注意：

如果在启动过程中需要将之前设置的网络配置进行关闭。
或者再重新配置网络 再后面添加一句话 , auto_config: true

```
config.vm.network "private_network" ,ip: "192.168.199.102", auto_config: true
```

