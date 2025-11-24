+++
title = 'YAML中的锚点和别名'
date = 2024-11-18T22:10:39+08:00
draft = false
categories = [ "Yaml" ]
tags = [ "yaml"]
+++

无意间发现yaml提供了一个特别有意思的功能：锚点（&）和别名（*）。可以使用它们来引用 yaml 中一些重复的配置。

yaml 配置如下：
```yaml
defaults: &defaults
  adapter:  postgres
  host:     localhost
 
development:
  database: myapp_development
  <<: *defaults
 
test:
  database: myapp_test
  <<: *defaults
```

等同下面yaml配置：
```yaml
defaults:
  adapter:  postgres
  host:     localhost
 
development:
  database: myapp_development
  adapter:  postgres
  host:     localhost
 
test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```

`&` 用来建立锚点（defaults），`<<` 表示合并到当前数据，`*` 用来引用锚点。


我将这一功能应用到docker-compose文件，省去了大量重复配置，使得 docker-compose 文件更为精炼：

```yaml
x-templates:
  agent_centos8_aarch64: &agent_centos8_aarch64
    image: harbor.einscat.com:10011/library/centos8:aarch64
    privileged: true
    stdin_open: true
    tty: true
    extra_hosts:
      - "docker-registry:192.168.18.20"
    cap_add:
      - ALL
    working_dir: /opt
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw

services:

  vm-1:
    container_name: vm-1
    hostname: "vm-1"
    networks:
      vm_net:
        ipv4_address: 172.20.30.1
      vm_net_2:
        ipv4_address: 172.20.40.1
    ports:
      - "3306:3306"
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:rw"
      - "~/workspace:/root/workspace"
    <<: *agent_centos8_aarch64

  vm-2:
    container_name: vm-2
    hostname: "vm-2"
    networks:
      vm_net:
        ipv4_address: 172.20.30.2
      vm_net_2:
        ipv4_address: 172.20.40.2
    ports:
      - "3307:3306"
      - "6380:6379"
    <<: *agent_centos8_aarch64
```

**参考**

- [YAML 语言教程](https://www.ruanyifeng.com/blog/2016/07/yaml.html?f=tt)