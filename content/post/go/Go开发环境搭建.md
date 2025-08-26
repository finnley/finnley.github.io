+++
title = 'Go开发环境搭建'
date = 2020-03-10T10:07:39+08:00
draft = false
categories = [ "Go" ]
tags = [ "go", "golang", "环境搭建" ]
+++

## 安装与配置

**下载**

- [ALL releases](https://go.dev/dl/)
- [Go 安装包下载](https://studygolang.com/dl)

**安装**

```bash
sudo wget https://golang.google.cn/dl/go1.14.12.darwin-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.14.12.darwin-amd64.tar.gz
```

参考：[Go installation](https://go.dev/doc/install)


**配置**

- **`GOROOT`**: Go安装路径，比如 `/usr/local/go`。
- **`GOPATH`**: Go工作目录，包含 `src`、`pkg`、`bin` 三个文件夹，比如 `/Users/{UserName}/golang`。安装目录通常存放自己开发的代码或者第三方依赖库。
- **`GOPROXY`**: 下载依赖库时的镜像代理，可以是公司内部自建镜像。
- **`GOPRIVATE`**: 指向自己的私有库，比如自己公司的私有库。
- **`PATH`**: 该目录下的二进制文件可以在任意目录下直接运行。

> 注意：
> 
> 	1、不要把 `GOPATH` 设置成安装目录。
> 
> 	2、别忘了将 `GOPATH/bin` 加入到 `PATH`，后续通过 `go install` 安装的工具都存放于此目录，如果没有加入环境变量则无法运行。	

例如我自己的 `zshrc` 文件内容配置如下:
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

## 多版本安装

Example for Go v1.23.3:

```bash
go install golang.org/dl/go1.23.3@latest
go1.23.3 download
```

参考：[Installing multiple Go versions](https://go.dev/doc/manage-install)

安装完之后会在用户目录下的sdk下看到go1.23的源码目录。


