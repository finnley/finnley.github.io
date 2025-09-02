+++
title = 'K8S入门'
date = 2023-10-16T22:52:25+08:00
draft = false
categories = [ "Kubernetes" ]
tags = [ "k8s", "kubernetes" ]
+++

## 文档

**参考**
- [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [](https://kubernetes.io/docs/home/)

## 介绍

Kubernetes 是一个开源的 `容器编排平台`，简称K8s。

换句话来说，是管容器的。举例来说：
- 学生是容器，那么老师就是 K8s。
- 你是容器，那么你老板就是 K8s。

既然是管理容器的，所以 K8s 既可以管理 Docker 产生的容器，也可以管理不是 Docker 产生的容器。

**为什么在不同的操作系统上都可以运行同样的容器呢**
可以理解为容器运行在统一的API之上，不同的操作系统为这个API提供了实现。容器有个规范，较 `OCI`。

### 基本概念

![](/images/k8s/10.png)

最开始学习的时候，你只需要记住几个基本的东西就可以：
- Pod：实例。
- Service：逻辑上的服务，可以认为这个是你业务上 XXX 服务的直接映射。
- Deployment：管理 Pod 的东西。

Pod 和 Service 最简单的理解方式：
- 假如说你有一个 Web 应用，部署了三个实例，那么就是一个 Web Service，对应了三个 Pod。
- 一个 Pod 里面很跑很多容器。比如同时可以在一个 Pod 里面部署 MySQL 和 Redis。
- Service 是一个逻辑上的服务，比如一个商城，这就是一个服务，也比如是一个 MySQL 服务，一个Redis服务，甚至可以理解为是一台服务器。

Deployment 最好的理解方式：
- 你跟运维说要保证我的 Web 有三个实例，少了运维就重启一个，多了运维就删除一个，运维就是那个 Deployment。就像你妈，你热了她就给你退掉一件衣服，你冷了她就给你加上一件衣服。

### 安装

- Docker (支持K8s)
- kubectl
- Rancher（备选）

## 配置

### Deployment

#### apiVersion

K8s 简单理解就是一个配置驱动的，或者元数据驱动，或者声明式的框架。怎么理解这句话？

我们用户只负责提供各种配置，然后 K8s 根据配置来执行一些动作。所以问题就来了，K8s 怎么知道该怎么解读这个配置呢？
答案就是利用 apiVersion 来确定怎么解读。

#### spec（specification）

也就是 Deployment 的规格说明，规格说明你就直接理解为说明书。

- replicas：值为 3，也就是我这个 Deployment 有三个副本，实际上就是三个 Pod。
- selector：筛选器，就是在 K8s 的一大堆 Pod 里面，可以让k8s知道哪些是我管理的那三个。比如他管理者上面有标签的是app，值是  micro-book ，只要带上这个标签就都我管，多了删，少了加。
- template：我该怎么创建每个 Pod，或者说每个Pod 长什么样。有了 template（模板），我就可以照猫画虎直接创建出来了。

#### selector

筛选器，它在 K8s 里面被用来筛选所需资源。你后面会经常和它打交道。

比如说在右图中，我们配置 selector 使用的是matchLabels。也就是说，Deployment 按照标签（label）来筛选它需要的资源——Pod。

它需要的是含有 app=webook 这个标签的 Pod。常用的除了 matchLabels 还有 matchExpressions，即根据表达式来筛选。如果 apiVersion 是你自定义的，你也可以设计自己的 selector，不过很复杂。

#### template

template 在 kind 不同的时候对应的内容也不同。
在 Deployment 里面，template 是创建 Pod 的模
板。
template 里面的内容也有很多。你再次看到了
metadata、labels 这些。
而在 template 的 spec 里面，要指定 containers。
从这里你也可以看出来，Pod 里面跑着的就是容器，而
且可以跑多个容器。

#### image

image 就是镜像，显然这里我们用的是 Docker 构建的
镜像。
注意，Docker 是实现 OCI 标准的，所以即便现在 K8s
声称不支持 Docker，但这个镜像依旧可以用

### Service

此时，只有 Deployment 你是没办法从外面访问的，你需要
将 Pod 封装为一个逻辑上的服务，即 Service。
右边 spec 里面有两个字段要注意：
• type：这里选择的是负载均衡，意思是这个 Service 在逻辑
上还要负责负载均衡的问题。
• 给谁负载均衡？给 selector 里面筛选出来的 Pod 做负载均衡。
• ports：端口，你可以配置很多个，比如还可以额外配置一
个 HTTPS 的。
• name：就是你这个端口的名字，一般用来指示用途，比如右边
的 http 意思是这个端口用来接收 HTTP 请求。你可以随便写。
• port：外部访问的端口。
• protocol：这个端口监听什么协议。
• targetPort：我这边转发请求的时候，应该转发到 Pod 上的哪
个端口。