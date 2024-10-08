+++
title = '通过Vagrant搭建多台CentOS服务器'
date = 2020-04-19T21:47:55+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++

![](/images/vagrant/0.jpg)

<!-- more -->

创建一个名为 `docker-network` 的目录

```
mkdir docker-network
```

进入目录编辑， 创建 setup.sh 文件， 键入下面内容

```
#/bin/sh

# install some tools
sudo yum install -y git vim wget gcc glibc-static telnet bridge-utils

# install docker
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh

# start docker service
sudo groupadd docker
sudo usermod -aG docker vagrant
sudo systemctl start docker

rm -rf get-docker.sh
```

上面的脚本很慢，可以使用 daocloud 进行加速,我这里使用的是下面的脚本

```
#/bin/sh

# install some tools
sudo yum install -y git vim gcc glibc-static telnet bridge-utils

# install docker
curl -fsSL https://get.daocloud.io/docker/ -o get-docker.sh
sh get-docker.sh

# start docker service
sudo groupadd docker
sudo usermod -aG docker vagrant
sudo systemctl start docker

rm -rf get-docker.sh
```

`:wq` 保存退出

接着创建并编辑 `Vagrantfile` 文件，键入下面内容

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "docker-node1",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1",
        :port => "8888"
    },
    {
        :name => "docker-node2",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1",
        :port => "9999"
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

  config.vm.synced_folder "./labs", "/home/vagrant/labs"
  config.vm.provision "shell", privileged: true, path: "./setup.sh"

end
```

启动 vagrant

```
vagrant up
```

进入节点，比如进入第一个节点

```
vagrant ssh docker-node1
```

进入后可以运行一些docker命令