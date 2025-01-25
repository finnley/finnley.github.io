+++
title = 'Go开发环境搭建'
date = 2020-03-10T10:07:39+08:00
draft = false
categories = [ "Go" ]
tags = [ "go", "golang" ]
+++

## 安装与配置

**下载地址**

- [ALL releases](https://go.dev/dl/)
- [Go 安装包下载](https://studygolang.com/dl)

**下载与安装**

参考：[Go installation](https://go.dev/doc/install)

```bash
sudo wget https://golang.google.cn/dl/go1.14.12.darwin-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.14.12.darwin-amd64.tar.gz
```

**环境配置**

- **`GOROOT`**: Go安装路径，比如 `/usr/local/go`。
- **`GOPATH`**: Go工作目录，包含 `src`、`pkg`、`bin` 三个文件夹，比如 `/Users/{UserName}/golang`，通常存放自己开发的代码或者第三方依赖库。
- **`GOPROXY`**: 下载依赖库时的镜像代理，可以是公司内部自建镜像。
- **`GOPRIVATE`**: 指向自己的私有库，比如自己公司的私有库。
- **`PATH`**: 该目录下的二进制文件可以在任意目录下直接运行。

> 注意：
> 
> 	1、不要把 `GOPATH` 设置成安装目录。
> 
> 	2、别忘了将 `GOPATH/bin` 加入到 `PATH`，后续通过 `go install` 安装的工具都存放于此目录，如果没有加入环境变量则无法运行。	

例如我自己的 `zshrc` 文件内容大体如下，可能略有少许不同:
```bash
# golang
export GOROOT=/usr/local/go1.14.12
export GOPATH=$HOME/workspace/golang
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
export GOPROXY=goproxy.cn,goproxy.io,direct
# export GOPRIVATE=xxx.cloud
# export GOINSECURE=xxx.cloud
```

**验证**

```bash
~ go version
go version go1.14.12 darwin/arm64
```

另外可以通过查看环境配置来验证是否安装成功。

```bash
go env
```

**补充**

还可以通过下面方式设置代理和开启 Go Module:
```bash
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GO111MODULE=on
```

**卸载**
```bash
sudo rm -rf /usr/local/go
```

## Go多版本安装

参考：[Installing multiple Go versions](https://go.dev/doc/manage-install)

Example for Go v1.23.3:

```bash
go install golang.org/dl/go1.23.3@latest
go1.23.3 download
```

安装完之后会在用户目录下的sdk下看到go1.23的源码目录。


## protobuf安装

参考：[mac 安装protobuf步骤](https://zhuanlan.zhihu.com/p/60471892)

1.从github上下载protobuf3，这里的版本是 [Protocol Buffers v3.13.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.13.0)

有很多语言版本的，mac下选择第一个:
![](/images/protobuf/1.jpg)

2.下载下来后解压压缩包，并进入目录

```bash
cd /usr/local
sudo tar -zxvf protobuf-all-3.13.0.tar.gz
cd protobuf-3.13.0/
```

3.设置编译目录

```bash
sudo ./configure --prefix=/usr/local/protobuf
```

4.安装

```bash
sudo make
sudo make install
```

5.配置环境变量，找到用户目录/Users/$USER 的 .zsh 或者 .bash_profile 文件并编辑，添加下面两行

```bash
export PROTOBUF=/usr/local/protobuf 
export PATH=$PROTOBUF/bin:$PATH
```

6.`source` 一下使文件生效

```bash
source ~/.zshrc
```

或者

```bash
source .bash_profile
```

7.测试安装结果

```bash
protoc --version
```

![](/images/protobuf/2.png)

8.另外还需要下载 Go 的依赖包

```go
go get github.com/golang/protobuf/protoc-gen-go
```

9.命令
```bash
protoc -I . helloworld.proto --go_out=plugins=grpc:.
```

**protoc**: 表示将 protocal buffer 文件生成对应语言的源码，比如可以生成 python、java、go等语言的源码，这里使用 `--go_out` 生成的 go 对应的源码，`--pyhon_out` 就会生成 python 对应的源码
**-I**: 表示后面的 `helloworld.proto` 文件在哪个路径之下，`I` 是 `Input` 的首字母，这里 `-I` 后面接的是 `.` 表示在当前目录之下，也就是 `helloworld.proto` 在当前目录之下
**--go_out**： 表示生成的是 golang 的源码
**plugins**：有些人可能希望 protocal buffer 变得更为强大，protoc 提供了插件的机制，允许我们在 proto 文件中在原来的基础上自定义一些代码逻辑，不建议初学者自己添加内容
**plugins=grpc**：表示除了生成基本的proto源码之外，再运行grpc的插件，插件的源码就是上面的 `protoc-gen-go`，这个插件可以完成 grpc 所需的内容，所以 `protoc` 是用来生成 `protocal buffer` 的，`protoc-gen-go` 是用来生成 `grpc` 的内容的，使用下面命令可以生成 protocal buffer 文件，但是没有 grpc 相关内容
```go
protoc -I . helloworld.proto --go_out=:.
```
