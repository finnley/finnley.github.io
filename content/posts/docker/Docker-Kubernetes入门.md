+++
title = 'Docker-Kubernetes入门'
date = 2020-07-26T10:38:15+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## Docker Swarm Mode Architecture

![](https://images.notes.xuepincat.com/docker/k8s/1.png)

Docker Swarm 分为 Manager 节点和 Worker 节点，Manager 节点主要是负责管理，位置 Cluster 的状态，并且对外提供接口，可以通过 Manager 节点部署 Application, Service 和 Stacks

## Kubernetes Architecture

![](https://images.notes.xuepincat.com/docker/k8s/2.png)

K8S 中类似 Manager 节点称为 Master 节点，Worker 节点称为 Node 节点，Master 节点对外会提供一些接口，可通过这些接口对集群进行一些操作，比如查看整个 Cluster 集群的整体状态，或者将 APP 通过 API 部署到集群中

#### Master

![](https://images.notes.xuepincat.com/docker/k8s/3.png)

Master 可以算是一个 k8s 集群的大脑，它有个 API Server,它主要是暴露给外界访问的，用户可以使用 CLI 或者 UI 通过 API Server 与集群进行交互

Scheduler 是一个调度模块，通过API下达一个命令比如部署应用，这个应用可能需要两个容器，这两个容器运行在哪个节点上，Scheduler 就会通过一些调度算法来决定容器运行在哪个节点

Controller 主要做一些控制款里，比如容器要做一些负载均衡，做一些扩展，就可以通过 Controller 来控制

etcd 作为底层的分布式 key-vaue Store 存储，主要存储 k8s 集群的状态和配置

#### Node

![](https://images.notes.xuepincat.com/docker/k8s/4.png)

Node 里面有个非常重要的概念叫做 Pod, Pod 在 k8s 里面属于可以调度的最小单位，Pod 指的是具有相同的 Namespace 的一些 Container 组合, `具有相同的 Namespace` 是指所有的 Namespace,比如 User Namesapce, Network Namespace,尤其指 Network Namespace,这个容器可能有一个，也可能有多个，如果是多个，它们共享一个 Network Namespace

最下面的几个模块，比如 Docker

kubelet: Node 节点是受 Master 节点控制的，那么 Master 节点是如何控制 Node 节点的呢？比如要在 Node 节点上创建 Pod，以及如何创建 Pod 里面的 Container 和 Volume ，就可以通过 kubelet 来操作

kube-proxy: k8s 总也有 service 的概念，如果 service 想要暴露一些端口给外界访问，就需要通过 kube-proxy 做端口的代理和转发，k8s 中 service 也可以做负载均衡，service 服务的发现和负载均衡也是通过 kube-proxy 实现的

Fluentd: 主要负责日志的采集存储以及查询

## 使用 minikube 搭建单节点集群

1. 本地 安装 minikube

```
brew install minikube
```

2. 安装 kubectl

```
brew instlall kubectl
```

or

```
brew install kubernetes-cli
```

3. 查看版本

```
minikube version
```

```
kubectl version
```

![](https://images.notes.xuepincat.com/docker/k8s/5.png)

4. 创建单节点 k8s 集群

```
minikube start
```

![](https://images.notes.xuepincat.com/docker/k8s/6.png)

5. kubectl context

kubectl 是一个 client 的 CLI,如要要连接 k8s 集群，k8s 集群提供了 API Server,CLI 如果要去连接 API Server,就必须要要有一些基本的配置信息，比如API地址，端口以及一些验证信息，我们可以将这些配置信息称为 context

可以通过下面命令查看当前 config 基本信息

```
kubectl config view
```

![](https://images.notes.xuepincat.com/docker/k8s/8.png)

可以看到 minikube 创建的 k8s 集群的 server 的地址和端口以及认证信息,以及 context 的名字叫 minikube 

可以通过下面命令查看 context

```
kubectl config get-contexts
```

![](https://images.notes.xuepincat.com/docker/k8s/9.png)

可以看到有条叫 minikube 的 context, 还有一条是之前使用的记录，比如有两个集群，这里就可以有两个 context,本地可以使用同一个命令去使用不同的 context 去连接不同的 k8s 集群

6. 查看 cluster 状态

```
kubectl cluster-info
```

![](https://images.notes.xuepincat.com/docker/k8s/10.png)

7. minikube ssh

```
minikube ssh
```

![](https://images.notes.xuepincat.com/docker/k8s/11.png)

可以通过上面的命令进入到通过 minikube 创建的 VirtualBox 的虚机里面，在虚机里面可以执行一些命令，比如 `docker version`

## Pod

Pod 是 K8S 中最小调度单位,k8s中不对容器直接操作，因为 Pod 是最小单位

Pod 和 Container 有什么关系呢？一个 Pod 中可以包含一个或者多个 Container

Pod 又有着什么特性呢？

一个 Pod 共享一个 Namespace,这个 Namespace 包含了用户，网络，存储等等，假如一个 Pod 中有两个容器，那么这两个容器共享一个 Network Namespace,它们俩之间的通信可以通过 localhost 进行通信，就像在本地一个 Namespace 中启动了两个进程，这个两个进程如果进行网络通信，可以直接 localhost

1. 创建一个名为 pod_nginx.yml 文件

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

该文件定义了一个 pod 的 yml，里面是 k8s API 格式，通过该文件可以创建一个 k8s 资源（resource），资源的类型是 `Pod`, metadata 里面记录的是 pod 的名称和 label

spec 里面定义了 Pod 中的容器，可以包含多个容器，上面只定义了一个 Container,这个 Container 的名字叫 nginx, image使用的是 nginx, 端口通过 ContainerPort 指定 nginx 暴露的端口为 80

2. 根据 yml 文件创建 resource

```
kubectl create -f pod_nginx.yml
```

![](https://images.notes.xuepincat.com/docker/k8s/12.png)

3. 查看

```
kubectl get pods
```

![](https://images.notes.xuepincat.com/docker/k8s/13.png)

可以看到有一个 nginx 的 pod 在后台运行，状态是 `Running`, READY 是 `1/1` 表示服务已经启动了

nginx 是一个容器，首先一个 Pod，这个 Pod 里面跑了一个 Container,那么这个 Container 又是在哪里呢？

可以通过下面命令显示 Pod 的详细信息

```
kubectl get pods -o wide
```

![](https://images.notes.xuepincat.com/docker/k8s/14.png)

可以看到有 IP ，有 Node，是运行在 minikube 节点上的,IP 是 172.17.0.3，这个地址其实就是容器的地址

4. 进入容器

一种传统的方式是进入到 minikube 中，然后通过 docker 命令对容器进行操作

```
minikube ssh
```

可通过 `docker ps` 看到一些运行中的 container，可以复制 Container 的ID，然后使用下面命令运行 shell 命令

```
docker exec -it 38f7338627c2 sh
```

![](https://images.notes.xuepincat.com/docker/k8s/15.png)

查看网络

退出后查看下网络,此时是在 minikube 创建的虚机里面

```
docker network ls
```

![](https://images.notes.xuepincat.com/docker/k8s/16.png)

```
docker network inspect bridge
```

![](https://images.notes.xuepincat.com/docker/k8s/17.png)

里面可以看到有一个 endpoint, 名字是 `k8s_POD_nginx_default_ed73f69f-49a2-4f50-b55d-d7626af6a9f9_0`, 这就可以表示创建的 nginx 容器是连接到了 bridge 网络上面的，IP 地址就是上面使用 `kubectl get pods -o wide` 看到的IP地址

除了上面的方法外，还可以通过下面的方式访问容器

```
kubectl exec -it nginx sh
```

![](https://images.notes.xuepincat.com/docker/k8s/18.png)

其中 nginx 是 pod 的名称,因为 nginx 的 pod 只有一个容器，就可以使用 `kubectl exec -it [pod name]` 交互式的进入到 nginx 这个 Pod 里面的 `k8s_POD_nginx_default_ed73f69f-49a2-4f50-b55d-d7626af6a9f9_0` 这个容器

如果一个 pod 中有多个容器呢？

使用 `kubectl exec -it nginx sh` 默认进入的是第一个容器

如果想进入第二个容器，可以通过 `kubectl exec -h` 命令帮助，可以看到 kubectl 支持的一些参数，比如 `-c`,就可以通过 `-c` 指定进入到哪个容器

另外还有 `describe` 命令，可以具体描述一个resource，k8s 中 一个 pod 就是一个 resource,比如下面可以 describe nginx 这个 pod

```
kubectl describe pods nginx
```

![](https://images.notes.xuepincat.com/docker/k8s/19.png)

这样就可以打印出 nginx 这个 pod 的详细信息

5. 网络

上面的图中可以看到启动的nginx容器连接在 monikube 中的 docker 的 bridge 的网络上面，IP 地址是 `172.17.0.3`,当前处于 minikube 的虚机的外面，如果想要访问该地址需要先进入到虚机中

```
minikube ssh
```

看下能否ping通该地址

```
ping 172.17.0.3
```

发现是可 ping 通的，接着使用 curl 命令

```
curl 172.17.0.3
```

![](https://images.notes.xuepincat.com/docker/k8s/20.png)

直接获取到了 NGINX 的 欢迎页面，现在在 minikube 中是可以访问到 nginx 的服务的，但是启动nginx服务的初中并不是让自己访问的，而是让外界可以访问到，下面打算将 nginx 的服务映射到外部接口上

使用 `ip a` 查看下minikube 的地址是 `192.168.64.2`,使用 `exit` 退出 minikube 后这个地址在外面是可以访问的，可以在外面 Ping 下

```
ping 192.168.64.2
```

下面将NGINX服务映射到上面的地址上，将本地8080端口映射到pod中的容器的nginx80端口

```
kubectl port-forward nginx 8080:80
```

![](https://images.notes.xuepincat.com/docker/k8s/22.png)

此时打开浏览器，输入 `127.0.0.1:8080` 回车后就会看到下面页面

![](https://images.notes.xuepincat.com/docker/k8s/23.png)

6. 删除 Pod

```
kubectl delete -f pod_nginx.yml
```

![](https://images.notes.xuepincat.com/docker/k8s/24.png)

## Pod 扩展

* ReplicationController

创建名为 `rc_nginx.yml` 的文件

```
apiVersion: v1
kind: ReplicationController 
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

k8s 中可以使用 ReplicaSet 类型的资源可以创建具有一定数量的比如 replicas 为 3 的，即3个nginx的Pod

```
kubectl create -f rc_nginx.yml
```

创建完后可以通过下面命令查看 ReplicationController

```
kubectl get rc
```

![](https://images.notes.xuepincat.com/docker/k8s/25.png)

通过下面命令查看下 pod

```
kubectl get pods
```

![](https://images.notes.xuepincat.com/docker/k8s/26.png)

可以看到创建了 3 个 pod,这三个 pod 就是一个横向的扩展

现在有3个pod正在运行，如果将其中的一个 pod 删除 

```
kubectl delete pods nginx-bjlpc
```

![](https://images.notes.xuepincat.com/docker/k8s/27.png)

会看到删除了一个pod的同事会再次为我们创建一个 pod，也就是通过 `ReplicationController` 创建 Pod,它会维持一个数目，如果 pod 异常退出或者删除，它会重新创建一个pod来填补这个空白

如果想要扩展可通过 `scale` 参数,

```
kubectl scale rc nginx --replicas=2
```

![](https://images.notes.xuepincat.com/docker/k8s/28.png)

已经变成两个了

可以 scale 成 4个

```
kubectl scale rc nginx --replicas=4
```

![](https://images.notes.xuepincat.com/docker/k8s/29.png)

现在已经有了4个pod，每个pod容器都会监听80端口，通过下面命令查看

```
kubectl get pods -o wide
```

![](http://images.notes.xuepincat.com/docker/k8s/30.png)

4个pod的container都运行在minikube的节点行，并且有4个IP地址，它们都会监听80端口

最后删除 rc_nginx.yml

```
kubectl delete -f rc_nginx.yml
```

![](https://images.notes.xuepincat.com/docker/k8s/31.png)

* ReplicaSet

创建名为 `rs_nginx.yml` 的文件

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

执行下面命令创建

```
kubectl create -f rs_nginx.yml
```

![](https://images.notes.xuepincat.com/docker/k8s/32.png)

使用 scale 进行扩展

```
kubectl scale rs nginx --replicas=2
```

![](https://images.notes.xuepincat.com/docker/k8s/33.png)

删除

```
kubectl delete -f rs_nginx.yml
```

## Deployment

1. 创建一个名为 `deployment_nginx.yml` 文件

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.2
        ports:
        - containerPort: 80
```

2. 创建

```
kubectl create -f deployment_nginx.yml
```

![](https://images.notes.xuepincat.com/docker/k8s/34.png)

可以看到有个叫 nginx-deployment 的 deployments, ReplicaSet 会在 nginx-deployment 的后面加一串数字

查看下 Pod,会发现又会在后面添加一串数字

```
kubectl get pods
```

![](https://images.notes.xuepincat.com/docker/k8s/35.png)

3. 常用命令

* 查看 deployment 详细信息

```
kubectl get deployment -o wide
```

![](https://images.notes.xuepincat.com/docker/k8s/36.png)

* 升级 deployment

```
kubectl set image deployment nginx-deployment nginx=nginx:1.13
```

然后再使用下面命令查看下是否升级成功

```
kubectl get deployment -o wide
```

![](https://images.notes.xuepincat.com/docker/k8s/37.png)

上面图中可以看到已经升级成功

再通过下面命令查看下 rs

```
kubectl get rs
```

![](https://images.notes.xuepincat.com/docker/k8s/38.png)

会看到旧的已经被撤销了

另外可以通过下面命令查看 history

```
kubectl rollout history deployment nginx-deployment
```

可以看到整个 rollout 的历史,图中可以看到有1和2，其实还可以会到1的版本

```
kubectl rollout undo deployment nginx-deployment
```

![](https://images.notes.xuepincat.com/docker/k8s/40.png)

图中已经会滚到 nignx:1.12 的版本了

接着再查看下 history

```
kubectl rollout history deployment nginx-deployment
```

![](https://images.notes.xuepincat.com/docker/k8s/41.png)

会看到历史版本已经增加到了3

现在还是回到 nginx：1.13 的版本

```
kubectl set image deployment nginx-deployment nginx=nginx:1.13
```

上面的操作可以很轻松的更新部署和回滚历史版本

* 将 deployment 暴露给外部访问

```
kubectl expose deployment nginx-deployment --type=NodePort
```

```
kubectl get svc
```

![](https://images.notes.xuepincat.com/docker/k8s/42.png)

看到第二行，它表示它监听了 minikube 的 32754 端口，minikube 的IP地址是 `192.168.64.2`,端口是 `32754`,这个地址值这个集群的地址或者说是k8s集群的Node地址

```
curl 192.168.64.2:32754
```

![](https://images.notes.xuepincat.com/docker/k8s/43.png)