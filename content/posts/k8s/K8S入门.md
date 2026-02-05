+++
title = 'K8S入门'
date = 2026-01-21T17:18:11+08:00 
draft = true
categories = [ "Kubernetes" ]
tags = [ "k8s", "kubernetes" ]
+++

## 为什么需要Kubernetes

### 传统容器编排的痛点

容器技术虽然解决了应用与技术设施异构的问题，让应用可以做到一次构建，多次部署，但在复杂的微服务场景，单靠容器技术还不够，它仍然有些问题没有解决：
- 集成和编排微服务模块
- 提供按需自动扩、缩容能力
- 故障自愈
- 集群内通信

### Kubernetes能解决的问题

- 按需垂直扩容，新的服务器（node）能够轻易增加或删除
- 按需水平扩容，容器实例能够轻松扩容、缩容
- 副本控制器，不用担心副本状态
- 服务发现和路由
- 自动部署和回滚，如果应用状态错误，可实现自动回滚

### 什么场景需要使用Kubernetes

- 微服务架构应用
- CI/CD
- 边缘计算
- 要求快速部署、快速测试、快速验证的需求
- 降低硬件资源成本，提高使用率

### 什么场景不需要使用Kubernetes

- 轻量级单体应用
- 无高并发需求



---

## 架构与核心概念

### 主控制节点组件

主控制节点组件对集群做出全局决策（比如调度），以及检测和响应集群事件（例如当不满足部署的replicas字段时，启动新的pod）。

主控制节点组件可以在集群中的任何节点上运行。然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有的主控制节点组件，并且不会在此计算机上运行用户容器。

**apiserver**
主节点上负责提供 Kubernetes API 服务的组件，它是 K8s控制面的前端组件。

**etcd**
它是兼具一致性和高可用性的键值数据，可以作为保存K8s所有集群数据的后台数据。

**kube-scheduler**
主节点上的组件，该组件件事那些新创建的未指定运行节点的Pod，并选择节点让Pod在上面运行。

调度决策考虑的因素包括单个Pod和Pod集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载见的干扰和最后时限。

它是个调度器，当平台请求要求work node创建一个Pod，scheduler会通知apiserver创建一个请求，调度器会考虑这个请求会落在哪个work node上，需要申请多少资源，开发哪些端口。

**kube-controller-manager**
在主节点上运行控制器的组件，从逻辑上，每个控制器都是一个单独的进程，但为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。这些控制器包括：

1. 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
2. 副本控制器（Replication Controller）：负责为系统中的每个副本控制器对象维护正确数量的Pod
3. 终端控制器（Endpoints Controller）：填充终端（Endpoints）对象（即加入Service和Pod）
4. 服务账户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认账户和API访问令牌

### 从节点组件

从节点组件在每个节点上运行，维护运行的Pod并提供K8s运行环境
**Kubelet**
一个在集群中每个节点上运行的代理。它保证容器都运行在Pod中。

kubelet 接收一组通过各类机制提供给它的PodSpecs，确保这些PodSpecs中描述的容器处于运行状态且健康。kubelet不会管理不是由K8s创建的容器。

**kube-proxy**
它是集群中每个节点上运行的网络代理，实现Kubernetes Service概念的一部分。
kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

**容器运行时（Container Runtime）**
容器运行环境是负责运行容器的软件。
Kubernetes 支持多个容器运行环境: Docker、containerd、cri-o、rktlet 以及任何实现 Kubernetes CRI (容器运行环境接口)。

### 插件

• DNS
尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该有集群 DNS，因为很多示例都需要 DNS 服务。

• Web 界面（仪表盘）
Dashboard 是 Kubernetes 集群的通用的、基于 Web 的用户界面。它使用户可以管理集群中运行的应用程序以及集群本身并进行故障排除。

• 容器资源监控
容器资源监控将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面。

• 集群层面日志
集群层面日志机制负责将容器的日志数据保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口。

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