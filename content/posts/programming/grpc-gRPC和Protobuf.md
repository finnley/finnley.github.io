+++
title = 'gRPC和Protobuf'
date = 2020-09-06T08:22:10+08:00
draft = true
categories = [ "gRPC" ]
tags = [ "gRPC", "rpc", "go", "python", "programming" ]
+++

**注意**
- Protobuf 不是 gRPC的一部分，而是gRPC使用了Protobuf协议。

# gRPC

gRPC 是Google发布的一个`高性能`、`开源`和通用的 `RPC框架`，底层基于 `HTTP2.0` 设计，并没有基于TCP，但也不用担心基于的HTTP协议造成的协议问题，因为HTTP2.0相对于HTTP1.0改进了很多，无论是性能还是长连接都有很大改进。可以认为HTTP2.0对TCP做了一层封装，但也没有损失多少TCP的性能问题。

gRPC除了使用HTTP2.0的高性能之外，还采用高压缩比的Protobuf协议。不仅压缩比高，序列化与反序列化也比常用JSON协议高很多。

另外gRPC还支持多种语言，目前提供 C、Java 和 Go 语言版本，分别是：grpc、grpc-java、grpc-go。 其中 C 版本支持 C、C++、Node.js、Python、Ruby、Objective-C、PHP 和 C# 支持。

[grpc](https://github.com/grpc/grpc)

# Protobuf

java中的dubbo dubbo/rmi/hessian messagepack 如果你懂了协议完全有能力自己去实现一个协议
- 习惯用 Json、XML 数据存储格式的你们，相信大多都没听过Protocol Buffer
- Protocol Buffer 其实 是 Google出品的一种轻量 & 高效的结构化数据存储格式，性能比 Json、XML 真的强！太！多！
- protobuf经历了protobuf2和protobuf3，pb3比pb2简化了很多，目前主流的版本是pb3

在考虑一个数据协议的时候通常会考虑以下几点：
- 数据压缩如何
- 序列化与反序列化效果如何

![](/images/grpc/10.png)

# GRPC调用过程

![](/images/grpc/20.png)

## Protobuf管理工具buf

[buf](https://buf.build/docs/cli/installation/)

**proto的难点**

```shell
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative user.proto
```

命令会使用到许多插件，无论书写、记忆、管理，都不是易事。

- 插件难管理。
- protoc命令本身不好记忆，使用时每次都要输入一堆参数
- 各种文件目录，package定位简直要命

所以直接使用protoc比较难，因此我们还可以考虑使用buf来管理。

**buf使用**

```yaml

```

在项目的顶级目录之下，有一个 buf.gen.yaml 文件，里面定义了 buf 怎么帮我们管理和编译 protobuf。

- go_package_prefix: 避免了每次写 go package 都要写老长一段的问题。
- plugins: 这是比较关键的配置，这里指定了两个插件，并且在两个插件里面分别指定了 out 和 opt。
- Go 语言插件
- gRPC 插件

Buf 还有其他很多配置项，需要使用的时候可以查阅文档。

最终在顶级目录下运行我封装好的 `make grpc` 或直接执行 `buf generate webook/api/proto`

