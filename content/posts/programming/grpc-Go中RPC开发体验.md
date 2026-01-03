+++
title = 'Go语言RPC开发体验'
date = 2020-09-05T08:22:10+08:00
draft = true
categories = [ "gRPC" ]
tags = [ "gRPC", "rpc", "go", "python", "programming" ]
+++

服务端RPC开发三步：
1. 实例化Server
2. 将接口注册到服务
3. 启动服务

思路：
服务端是提供服务的，所以要实现一个服务。服务端创造一个服务通常叫实例化，将是一个从无到有的过程，是一个由想法到具体概念的过程；
RPC是远程过程调用，将本地方法放到远程服务器上，像调用本地方法一样调用远程服务器上的方法 -> 所以服务端需要暴露方法供外部调用
方法可能有很多，如果将一个一个方法分别注册到服务上非常不方便 -> 所以可以将众多方法绑定到struct上从而变成接口，最后将接口注册到服务

**server/helloworld.go**

```go
package main

import (
	"net"
	"net/rpc"
)

type HelloService struct{}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "hello " + request
	return nil
}

func main() {
	_ = rpc.RegisterName("HelloService", &HelloService{})

	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		panic("监听端口失败")
	}
	conn, err := listener.Accept()
	if err != nil {
		panic("建立连接失败")
	}

	rpc.ServeConn(conn)
}
```

服务端想提供一个打招呼的服务，取名为 “HelloService”，故为其设计一个结构:
```go
type HelloService struct{}
```

就像我想设计一个机器人，为其设计了一个框架，它有头、手、身体和脚。只是“HelloService”这个服务现在还很抽象，什么都没有，很模糊，所以这个结构里什么都没有。


我设计的这个机器人能做什么呢？可以行走、说话。同样类似，我这个“HelloService”能做什么呢？它能打招呼，所以我为其绑定一个动作，在Go中叫“方法”：
```go
type HelloService struct{}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "hello " + request
	return nil
}
```

如忘记指针可见另一篇笔记：《初识 C Pointer》。

方法接收两个参数，注意第二个参数是string类型的指针变量，返回值是个error。现在想返回打招呼的内容，是个字符串，但是方法却是error，类型不一样呀，怎么办呢？幸好第二个参数是指针类型，这就好办了，可以修改指针呀，所以这里返回值是通过修改reply指针变量实现的。

到这里一个可以打招呼的“助手服务”这一新的概念已经创造出来了，它有自己的结构（只不过除了会打招呼其他都不会，啥都没有），接下来考虑的就是如何使用呢？我们不能只顾输入，还得输出，创造出来的东西没有不能用那它没有价值，于是我们想要启动一个服务，让它运作起来。

我需要将其注册为一个服务，类似的我要运作一家公司，我需要到市场管理处注册它，办理营业执照，表示它是合法的，之后才能正常运作。

```go
_ = rpc.RegisterName("HelloService", &HelloService{})
```

办理的“公司”我为其取名为“HelloService”，第二个参数是个接口，就像市场管理部门文件规定的那样，我公司要符合哪些规定，配备了哪些设施，尽管设置、文件、选址这些细节可能不一样，但是你必须要要具备这些通用资格，所以它是一个接口，而不是具体的细节，所以这里传入的 `HelloService` 这个结构体对象。

为什么是取地址呢？我总不能注册公司的把整个公司搬到市场管理部门里面吧，我只要告诉我公司的地址在哪里就行了，这样市场部门就能根据地址找到我们公司了，所以传入的是个地址，多方便呀。

注册好了之后不能只有我知道呀，我还要搞个开业酬宾，放个爆竹宣传一下，我在某某街道开了家公司，让别人都知道过来办理业务，于是我需要对外宣传暴露端口：
```go
listener, err := net.Listen("tcp", ":1234")
if err != nil {
	panic("监听端口失败")
}
```

接下来就是敞开大门，开门迎客，提供服务：
```go
conn, err := listener.Accept()
if err != nil {
	panic("建立连接失败")
}
rpc.ServeConn(conn)
```

**注意**
这里服务端只处理一次连接，当客户端调用完后服务端也就处理结束了，就像目前公司暂没有提供售后一样。

到这里服务端完成，下来就是编写客户端的代码了。


**client.go**
```go
package main

import (
	"fmt"
	"log"
	"net/rpc"
)

func main() {
	client, err := rpc.Dial("tcp", "localhost:1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}

	var reply string
	err = client.Call("HelloService.Hello", "world", &reply)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(reply)
}
```

现在我作为客户，我遇到了一些问题，我需要寻找这样一个公司为我解决这些问题。我事先了解过有这样一个公司，预留了它的电脑号码，于是一天拨打电话过去咨询：
```go
client, err := rpc.Dial("tcp", "localhost:1234")
if err != nil {
	log.Fatal("dialing:", err)
}
```
就这样一条我与公司的连接建立起来了。

由于我事先了解过他们家公司的提供的服务，也实现准备好了材料，于是在在咨询服务的时候我直接和公司讲我要办理某个具体的服务“HelloService.Hello”，并将准备的材料“world”发送给了他们，公司那边办理好了，会连带材料一并给我寄过来。
```go
var reply string
err = client.Call("HelloService.Hello", "world", &reply)
if err != nil {
	log.Fatal(err)
}
```

我收到公司寄过来的材料后检查下是否办理好了：
```go
fmt.Println(reply)
```

注意不能这样写：
```go
var reply *string
err = client.Call("HelloService.Hello", "world", reply)
```
因为你定义了一个*string指针变量，刚开始是个nil，然后你把一个它传给过去，这就像你告诉公司你有这么材料，然后你却和公司说你寄过去了，这不就出问题了，公司实际没有收到呢寄过去的材料，也就是你不能将一个只是构造出来的还没有地址的变量传过去。

我们可以改成下面方式：
```go
var reply *string = new(string)
err = client.Call("HelloService.Hello", "world", reply)
```
new 表示我在内存中分配了一块空间，并把这块空间的地址赋值给了reply这个指针变量，这样传过去的就是一个真实存在的材料了，而不是你告诉公司只是一个口头说你准备好了材料但实际并不存在得材料。

在办理过程中不管是公司还是个人，都会产生一些感想与体验，这些有些可能是问题，可能是需要我们进行改进的点，比如：
1. 每次调用都需要客户端传入指定的服务，比如“HelloService.Hello”，这就要求客户端必须知道提供的服务与方法，而rpc是要像本地函数一样调用远程方法，类似这样的调法“client.Hello("world", reply)”



想想公司目前提供只有中国境内的汉语用户，现在来了一个法国人，他来办理业务，怎么办呢？在RPC中，这就是序列化和反序列化要解决的问题，只有解决了这个问题，也就解决了RPC跨语言调用的问题。

这里我们需要上面写的Go语言的RPC序列化协议是什么？能否替换成常见的序列化协议，就像公司是汉语业务，客户是法语业务，但双方都懂英语这一通用语言，那业务就不成问题了。

Go 中的RPC使用的自身特有的Gob编码，如果调用方是Python语言，那将无法直接提供服务，这就严重限制了跨语言调用的能力。


# 替换rpc序列化协议为json

```go
package main

import (
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct{}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "hello " + request
	return nil
}

func main() {
	// 1.实例化一个server
	// 2.注册
	// 3.启动

	_ = rpc.RegisterName("HelloService", &HelloService{})

	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		panic("监听端口失败")
	}

	for {
		conn, err := listener.Accept()
		if err != nil {
			panic("建立连接失败")
		}

		go rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	}
}
```

想要替换序列化协议，就不再使用 `rpc.ServeConn(conn)`，而是使用支持编解码的 `rpc.ServeCodec(jsonrpc.NewServerCodec(conn))`，服务端也只需要修改这一句即可，传入的参数只要是支持的编解码都可以，这里使用的就是json来进行编解码。

客户端修改如下：
```go
package main

import (
	"fmt"
	"log"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

func main() {
	conn, err := net.Dial("tcp", "localhost:1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}

	var reply string
	client := rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))
	err = client.Call("HelloService.Hello", "world", &reply)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(reply)
}

```

现在解决了公司和客户的语言问题，如果使用Python客户端来调用呢？
```python
import json
import socket

request = {
    "id": 0,
    "method": "HelloService.Hello",
    "params": ["Tom"],

}

client = socket.create_connection(("localhost", 1234))
client.sendall(json.dumps(request).encode())

# 获取服务端返回的数据
rsp = client.recv(1024)
rsp = json.loads(rsp.decode())

print(rsp["result"])

```

现在就实现了跨语言的调用。

# 替换rpc传输协议为http

为了使用更方便，现在想修改tcp请求为http请求，让服务变得更通用。

**server.go**
```go
package main

import (
	"io"
	"net/http"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct{}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "hello " + request
	return nil
}

func main() {
	_ = rpc.RegisterName("HelloService", &HelloService{})
	http.HandleFunc("/jsonrpc", func(w http.ResponseWriter, r *http.Request) {
		var conn io.ReadWriteCloser = struct {
			io.Writer
			io.ReadCloser
		}{
			ReadCloser: r.Body,
			Writer:     w,
		}
		rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	})

	http.ListenAndServe(":1234", nil)
}
```

**client.python**
```python
import requests

request = {
    "id": 0,
    "method": "HelloService.Hello",
    "params": ["Judy"],

}

rsp = requests.post("http://localhost:1234/jsonrpc", json=request)
print(rsp.text)

```
