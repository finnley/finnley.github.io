+++
title = '使用vagrant安装CentOS7'
date = 2019-05-15T23:01:27+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++


# 方法一

此种方法需要联网下载 `centos` 的 `vagrant` 盒子，真的很慢

```
mkdir centos7
cd centos7
vagrant init centos/7
vagrant up
```

# 方法二

提前下载好 `centos/7` 的镜像，所以这里先进行添加镜像，我这里的镜像(`CentOS-7-x86_64-Vagrant-1902_01.VirtualBox.box`)已经下载好了

## 添加镜像

```
vagrant box add centos/7 CentOS-7-x86_64-Vagrant-1902_01.VirtualBox.box
```

![](https://images.notes.xuepincat.com/vagrant/114.png)

## 查看

```
vagrant box list
```

![](https://images.notes.xuepincat.com/vagrant/115.png)

## 创建目录

```
mkdir centos7
cd centos7
```

## 初始化

```
vagrant init centos/7
```

## 启动

```
vagrant up
```

## 连接

```
vagrant ssh
```

# 练习

接下来使用 `virualbox` 和 `vagrant` 搭建带有 `docker` 的环境

## 创建目录

首先创建一个名为 `dev-centos7` 的目录

```
mkdir centos7-dev
cd centos7-dev
```

## setup.sh

进入 `centos7-dev` 目录，创建一个 `setup.sh` 的 `shell` 文件，该脚本在 `vagrant up` 启动后自动执行，内容如下：

```
#/bin/sh

# install some tools
sudo yum install -y git vim wget gcc glibc-static telnet bridge-utils

# install docker
# curl -fsSL get.docker.com -o get-docker.sh
# sh get-docker.sh

# 太慢了，我改成了下面使用 daocloud 加速
#curl -fsSL https://get.daocloud.io/docker/ -o get-docker.sh
#sh get-docker.sh
# curl -sSL https://get.daocloud.io/docker | sh
#rm -rf get-docker.sh

# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
#sudo service docker start

# start docker service
sudo groupadd docker
sudo usermod -aG docker vagrant
sudo systemctl start docker
sudo systemctl enable docker

# 配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vfs1y0dl.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# install docker compose
curl -L https://get.daocloud.io/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

* `shell` 脚本中在 `centos` 的系统中安装一些常用的软件，如 `vim`, `git`, `wget` 等；
* `shell` 脚本中安装 `docker`，为了防止 `docker` 下载过慢，使用了阿里软件源；
* `shell` 脚本安装 `docker compose`

## Vagrantfile

在执行 `vagrant init centos/7` 后目录中会出现 `Vagrantfile` 的文件，在生成的文件基础上修改，修改后的内容如下：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "centos7-dev",
        :eth1 => "192.168.20.21",
        :mem => "1024",
        :cpu => "1",
        :port => "2021"
    },
    {
        :name => "centos7-testing",
        :eth1 => "192.168.20.22",
        :mem => "1024",
        :cpu => "1",
        :port => "2022"
    },
    {
        :name => "centos7-staging",
        :eth1 => "192.168.20.23",
        :mem => "1024",
        :cpu => "1",
        :port => "2023"
    }
]

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.network "forwarded_port", guest: 80, host: opts[:port]
        config.vm.provider "vmware_fusion" do |v|
          v.vmx["memsize"] = opts[:mem]
          v.vmx["numvcpus"] = opts[:cpu]
        end

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end

        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.synced_folder "~/Code", "/home/vagrant/Code"
  config.vm.synced_folder "../labs", "/home/vagrant/labs"
  config.vm.provision "shell", privileged: true, path: "./setup.sh"

end
```

* 添加内容：

```
boxes = [
    {
        :name => "centos7-dev",
        :eth1 => "192.168.20.21",
        :mem => "1024",
        :cpu => "1",
        :port => "2021"
    }
]
```

* 它表示创建的主机的名称是 `centos7-dev`，分配的 `IP` 是 `192.168.20.21`，设置的内存为 `1024M` 也就是 `1G`, CPU 个数是 1 个

```
config.vm.network "forwarded_port", guest: 80, host: opts[:port]
```

* 上面这句话表示将虚拟机的 `80` 端口映射到了本地的 `2020` 端口上

```
config.vm.synced_folder "../Code", "/home/vagrant/Code"
```

* 上面这句话表示将本地的 `~/Code` 目录映射到了虚拟机中的 `/home/vagrant/Code` 目录中

## 启动 

```
vagrant up
```

## 连接

```
vagrant status
vagrant ssh centos7-dev
```