+++
title = '初识RPC'
date = 2020-09-04T08:22:10+08:00
draft = true
categories = [ "gRPC" ]
tags = [ "gRPC", "rpc", "go", "python", "programming" ]
+++

# 1 RPC是什么?

- RPC（Remote Procedure Call）远程过程调用，通俗讲就是一个节点请求另一个节点提供的服务。
- 它强调的是**远程过程调用**，与之对应的是**本地过程调用**，最简单的本地过程调用就是函数调用。

# 2 远程调用中的问题

先从一个最简单的本地函数调用开始，下面是一个最简单的本地过程调用代码：
```go
func add(a, b int) int {
	total := a + b
	return total
}

func main() {
	total := add(1, 2)
	fmt.Println(total)
}
```

```python
def add(a, b):
	total = a + b
	return total

total = add(1, 2)
print(total)
```

上面代码是一个函数调用的不同语言实现，也是一个最简单的本地过程调用，它的调用过程如下：
1. 每个函数调用时会初始化一个栈，在调用之前会将传递的 1 和 2 两个值压入 add 函数所在的栈中；
2. 压入栈之后会有个执行函数过程，分别从栈中取出 a 和 b 的值，也就是 1 和 2；
3. 然后做加法运算，把值给 total，然后将 total 压入函数栈中；
4. 最后 return 时会从栈底的元素弹出，将值赋给变量 total ，函数里面的 total 是个局部变量，函数调用完函数里面的 total 就销毁了。

上面本地函数的调用是在当前节点（机器）上的编译器上完成的。如果将上面调用变成远程调用，过程就不会像上面步骤那么简单了。

## 2.1 思考

如果将 add 函数放在另外一个远程服务器上，然后本地的服务器去调用远程服务器上的这个函数，把结果从另一台服务器返回给本地的服务器，这个过程就是一个远程过程调用。如果要做到这点，会面临什么问题呢？了解了这些问题也就知道了这些问题其实也就是设计RPC要解决的问题。

在远程调用时，我们需要执行的函数体是在远程的机器上的，也就是说，add 函数是在另一个远程机器上执行。这会带来以下几个问题：

- 如何将本地函数放到另一个服务器上去运行
- Call ID
- 序列化和反序列化
- 网络传输

**为什么需要RPC？**

为什么会有RPC？为什么需要将一个服务变成另一个节点提供的服务，而不是去使用类似本地函数一样的调用呢？这其实是分布式系统应用到的最基本的行为。

**为什么需要将一个函数放到另一个服务器上运行？**

比如电商系统有一段下单逻辑，这个逻辑需要扣减库存，但库存服务是在一个独立的系统中，里面有个Reduce扣减库存的函数，原本库存服务写在本地，但是后面发现库存系统非常复杂，就单独做成了一个服务，这个服务也可能放在了另外一台服务器上运行，就这样原来的一个本地函数调用就变成了远程调用。

**不同的服务器之间如何传输？**

当然是网络，只要是跨服务器通信，就一定会涉及到网络。

现成的方案就是做成一个网络服务，可以使用的方案也很多，比如使用各种框架，比如Gin、Beego、HTTP等。

**调用过程**

提到网络就想到协议，又想到在网络调用时有两个端，一个是客户端，一个是服务端，它们都有着各自的作用与职责。客户端与服务端通过json编码协议来传递与解析数据。但json并不是一个高性能的编码协议。

**既然使用网络传输，那么函数的参数如何在网络之间传递呢？**

我们常见的一种发送数据的方式是json。json不仅是一种数据格式，也是一种数据协议，常见的python操作有json.dump()和json.loads()。

简单数据还好，如果是复杂数据呢，每种语言都有自己的数据结构，比如python中的类，Go中的struct，它们又该如何传递呢？

比如：
```go
type Company struct {
	Name    string
	Address string
}

type Employee struct {
	Name    string
	Company Company
}

func TestPrintEmployee(t *testing.T) {
	fmt.Println(Employee{
		Name: "Judy",
		Company: Company{
			Name:    "Ming",
			Address: "Beijing",
		},
	})
}
```

比如上面有一个本地打印任务，现在需要将打印的工作放到另一台服务器上，意味需要将本地内存对象struct放到远程服务器上去，这能直接传递吗？肯定不行。

一种可行的方式就是将struct序列化成json进行传输，本质上在网络上传输的都是二进制对象，然后远程服务器器需要将传输过来的二进制对象反解。

为了实现远程调用，可能还会想到一下方案：       
客户端：
	1. 建立连接 tcp/http
	2. 将employee对象序列化成json字符串 - 序列化
	3. 发送json字符串 - 调用成功后实际接收到的结果也是一个二进制数据
	4. 等待服务器返回结果，将返回的数据解析成自己的对象 - 反序列化

服务端：
	1. 监听网络端口80
	2. 读取数据 - 二进制的json数据
	3. 对数据进行反序列化
	4. 处理业务逻辑
	5. 将处理结果序列化成json二进制数据
	6. 将数据返回

至于协议，是选择TCP还是HTTP呢？如果使用HTTP协议，它有一个问题，就是一次性，一旦对方返回了结果，就会断开连接，但是有HTTP2.0 长连接。

![](/images/programming/rpc/40.png)


**如何将内存中的对象编程网络中对象进行传输**

二进制

## 2.1 如何将本地函数放到另一个服务器上去运行


**如何调用？**

这一定会牵涉到网络。因为只要跨服务器，就一定会涉及到网络传输。我们能想到最简单的方式就是做成一个web服务，比如gin、flask、net/http等等。这就涉及到下面需要解决的问题-网络问题。

如果是这样，还需要解决一些问题，就是函数的参数如何传入？有些函数的参数是简单的数值、字符串，但也有些复杂的参数，比如对象等

想到最常用的传递比如json/xml，以及grpc使用的protobuf。于是这就涉及到下面序列化与反序列化的问题。

网络调用还会涉及到两个端，客户端和服务端，客户端将数据传出到服务端，服务端负责解析数据。




## 2.1 Call ID

**问题1：如果 Server A 向 Server B 发送一个网络请求来调用 add1 函数，那远程服务器 Server B 怎么知道调用的是 add1 函数，而不是 add2 或其他函数呢？**

![](/images/programming/rpc/10.png)

比如现在将 add1 函数放到另外一台服务器上，可以看成是本地的一个进程想要调用远程的一个服务器上的 add1 函数，远程机器上可能还不止一个函数，可能有 add2、add3······

现实生活中是否有类似这样的例子呢？

![](/images/programming/rpc/20.png)

就像一个客户到商店买东西，店家老板怎么知道客户要买什么呢？很简单，只需要客户将自己的需求说出来，把要买的东西告诉老板，是要烟还是酒，如果客户不说老板怎么知道呢？

<center><img src="/images/programming/rpc/30.gif"/></center>

回到最初的问题，要解决Server B如何知道Server A调用的是 add1 而不是其他函数？这就像商店买东西一样，需要让购买者告诉商家，也就是需要 Server A 告诉 Server B 调用的是 add1。

比如：可以给每个函数一个 ID（当然不一定是ID，只要能唯一区别出来都行），当想调用 add1 的时候，告诉远程服务器调用的是 ID 为 1 的函数，远程服务器就知道调用的是 add1 函数并将结果返回。也就是服务双方需要沟通好调用的是哪个函数，在调用时就能够准确定位了。

在本地调用中，函数体是直接通过函数指针来指定的，我们调用add，编译器就自动帮我们调用它相应的函数指针。但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的（不在同一个服务器），就好比修仙宇宙中两个人都不在同一个修仙大陆中一样。

所以，为解决这个问题，RPC的设计就是所有的函数都必须有自己的一个ID，这个ID在所有进程中都是唯一确定的。做远程过程调用时必须附上这个ID，然后还需要在两端分别维护一个函数与ID（ {函数 <--> Call ID}）的对应表。两者的表不一定需要完全相同，但**相同的函数对应的Call ID必须相同**。当一端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给另一个服务器，另一端也通过查表来确定需要调用的函数，然后执行相应函数的代码。

## 2.3 序列化和反序列化

就好比寄信，寄件方需要将写好的信使用信封装起来，这就类似数据的序列化。收方收到信后会拆开信封取出里面的信件，这就相当于数据的反序列化。

```go
package main

import "fmt"

func Add(a, b int) int {
	total := a + b
	return total
}

func main() {
	total := Add(1, 2)
	fmt.Println(total)
}
```

```python
def add(a, b):
	total = a + b
	return total

print(add(1, 2))
```

我们常见的一种发送数据的方式是json。json不仅是一种数据格式，也是一种数据协议，常见的操作有json.dump()和json.loads()。
json.dump()本质上就是一种序列化。json.loads()就是将传输过来的数据进行反序列化。

对简单的数据如何传输很好理解，如果是复杂的数据结构呢？又该如何传递呢？

比如上面有一个 Add 函数，现在想把Add函数变成一个远程的函数调用，也就意味着需要把Add函数放到远程服务器上去运行，分布式系统也存在类似的场景：


**问题2：将一个函数放到远程服务器上来调用，调用过程中客户端怎么把参数值传给远程的函数呢？**

在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用C++，客户端用Java或者Python）。

这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要经过序列化反序列化的过程。

## 2.4 网络传输

现在想把add函数放到另一台服务器上去调用，这势必会设计到网络。也就是一定会走网络请求。紧接着会想到如何将函数中的1和传递到远程的服务器上去。

远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把Call ID和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分RPC框架都使用TCP协议，但其实UDP也可以，而gRPC干脆就用了HTTP2。Java的Netty也属于这层的东西。

解决了上面三个机制，就能实现RPC了，具体过程如下：
client端解决的问题：

1. 将这个调用映射为Call ID。这里假设用最简单的字符串当Call ID的方法
2. 将Call ID，a和b序列化。可以直接将它们的值以二进制形式打包
3. 把2中得到的数据包发送给ServerAddr，这需要使用网络传输层
4. 等待服务器返回结果
4. 如果服务器调用成功，那么就将结果反序列化，并赋给total

server端解决的问题

1. 在本地维护一个Call ID到函数指针的映射call_id_map，可以用dict完成
2. 等待请求，包括多线程的并发处理能力
3. 得到一个请求后，将其数据包反序列化，得到Call ID
4. 通过在call_id_map中查找，得到相应的函数指针
5. 将a和rb反序列化后，在本地调用add函数，得到结果
6. 将结果序列化后通过网络返回给Client

在上面的整个流程中，估计有部分同学看到了熟悉的计算机网络的流程和web服务器的定义。
所以要实现一个RPC框架，其实只需要按以上流程实现就基本完成了。
其中：
Call ID映射可以直接使用函数字符串，也可以使用整数ID。映射表一般就是一个哈希表。
序列化反序列化可以自己写，也可以使用Protobuf或者FlatBuffers之类的。
网络传输库可以自己写socket，或者用asio，ZeroMQ，Netty之类。
实际上真正的开发过程中，除了上面的基本功能以外还需要更多的细节：网络错误、流量控制、超时和重试等。
最后提一个问题： 如何将远程的这些过程写出本地函数调用的感觉来？



## 分析

```go
package _0_rpc

import (
	"fmt"
	"testing"
)

func Add(a, b int) int {
	return a + b
}

func TestAdd(t *testing.T) {
	fmt.Println(Add(1, 2))
}

```

现在要把Add函数编程一个远程的函数调用，这意味着需要把Add函数放到另一台服务器上运行。

# 使用http server实现rpc

## Go版本1.0

**服务端**

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strconv"
)

func main() {
	// 使用最简单的方式来传递参数：http://127.0.0.1:8000/add?a=1&b=2
	// 返回格式：json。{"data": 3}
	// 解决了3个问题：1.Call ID（URL），2.数据传输协议 3.网络传输协议（HTTP）
	http.HandleFunc("/add", func(w http.ResponseWriter, r *http.Request) {
		// 前端参数解析
		_ = r.ParseForm()
		fmt.Println("path: ", r.URL.Path)
		a, _ := strconv.Atoi(r.Form["a"][0])
		b, _ := strconv.Atoi(r.Form["b"][0])
		w.Header().Set("Content-Type", "application/json") // 设置返回类型
		jData, _ := json.Marshal(map[string]int{
			"data": a + b,
		})
		_, _ = w.Write(jData)
	})

	http.ListenAndServe(":8080", nil)
}

```

**客户端**

由于使用的HTTP协议与URL方式传递参数，所以可以直接在浏览器中输入URL：

![](/images/programming/rpc/50.png)

这样就可以访问一个远程的服务了，但这样仍然举例rpc很远，还存在许多问题。于是还可以通过下面方式来实现客户端的功能“

```go
package main

import (
	"fmt"

	"github.com/kirinlabs/HttpRequest"
)

func main() {
	// 这个库仅用于演示，不做生产使用
	req := HttpRequest.NewRequest()
	res, _ := req.Get("http://localhost:8000/add?a=1&b=2")
	body, _ := res.Body()
	fmt.Println(string(body)) // {"data":3}
}

```

如果客户端和服务端像上面这样，没有什么意义，为了向RPC靠齐，实现远程调用，该怎么做到像本地一样调用呢？

所以要继续封装.

## Go版本2.0

**客户端**

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/kirinlabs/HttpRequest"
)

type ResponseData struct {
	Data int `json:"data"`
}

func Add(a, b int) int {
	// 这个库仅用于演示，不做生产使用
	req := HttpRequest.NewRequest()
	res, _ := req.Get(fmt.Sprintf("http://localhost:8000/%s?a=%d&b=%d", "add", a, b))
	body, _ := res.Body()
	rspData := ResponseData{}
	_ = json.Unmarshal(body, &rspData)
	return rspData.Data
}

func main() {
	fmt.Println(Add(1, 2)) // 3
}
```

现在做了一次简单的封装，看上去有点就像本地函数一样调用的影子了，但仍是存在问题。


## Python1.0版本

首先一点需要明确：远程调用一定会发起一个网络请求，一定会有个网络连接(tcp/udp)，一定会暴露一个端口
  a. 把远程的函数变成一个http请求

**服务端**

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlparse, parse_qsl
import json

host = ('', 8003)


class AddHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        parsed_url = urlparse(self.path)  # 解析请求路径
        qs = dict(parse_qsl(parsed_url.query))  # 解析请求参数
        a = int(qs.get('a', 0))
        b = int(qs.get('b', 0))
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({
            "result": a + b,
        }).encode("utf-8"))


if __name__ == '__main__':
    server = HTTPServer(host, AddHandler)
    print("启动服务器")
    server.serve_forever()
```

**客户端**

```python
# requests
import requests

rsp = requests.get("http://127.0.0.1:8003?a=1&b=2")
print(rsp.text)
```

上面是本质上就是简单的HTTP调用，这种调用方式需要我们知道每个函数调用的url地址和参数，以及参会数据如何解析，我们想要的是值，而不是json格式的数据，因为这并不像本地调用函数的返回结果，于是还可以继续封装改进，能够将函数调用像本地函数调用一样。

## Python2.0版本

**客户端**
```python
# requests
import json

import requests

class Client:
    def __init__(self, url):
        self.url = url

    def add(self, a, b):
        rsp = requests.get(f"{self.url}?a={a}&b={b}")
        return json.loads(rsp.text).get("result", 0)

client = Client("http://127.0.0.1:8003")
print(client.add(1, 2)) # 3
print(client.add(1, 3)) # 4
```

至此我们实现一个DEMP级别rpc封装。


**小结**

class Client 相当于一个代理，我们调用时不想知道过多的细节，只想像本地一样调用，这些细节包括发送的服务器url是什么
，call id，序列化与反序列化问题、传输协议。
代理也可以成为Proxy，在rpc中成为Stub存根。

# RPC开发的要素分析

![](/images/programming/rpc/60.png)

rpc开发的四大要素
RPC技术在架构设计上有四部分组成，分别是：客户端、客户端存根、服务端、服务端存根。
**客户端(Client)：**服务调用发起方，也称为服务消费者。
**客户端存根(Client  Stub)：**该程序运行在客户端所在的计算机机器上，主要用来存储要调用的服务器的地址，另外，该程序还负责将客户端请求远端服务器程序的数据信息打包成数据包，通过网络发送给服务端Stub程序；其次，还要接收服务端Stub程序发送的调用结果数据包，并解析返回给客户端。
**服务端(Server)：**远端的计算机机器上运行的程序，其中有客户端要调用的方法。
**服务端存根(Server Stub)：**接收客户Stub程序通过网络发送的请求消息数据包，并调用服务端中真正的程序功能方法，完成功能调用；其次，将服务端执行调用的结果进行数据处理打包发送给客户端Stub程序。
了解完了RPC技术的组成结构我们来看一下具体是如何实现客户端到服务端的调用的。实际上，如果我们想要在网络中的任意两台计算机上实现远程调用过程，要解决很多问题，比如：
两台物理机器在网络中要建立稳定可靠的通信连接。
两台服务器的通信协议的定义问题，即两台服务器上的程序如何识别对方的请求和返回结果。也就是说两台计算机必须都能够识别对方发来的信息，并且能够识别出其中的请求含义和返回含义，然后才能进行处理。这其实就是通信协议所要完成的工作。
在上述图中，通过1-10的步骤图解的形式，说明了RPC每一步的调用过程。具体描述为：
1、客户端想要发起一个远程过程调用，首先通过调用本地客户端Stub程序的方式调用想要使用的功能方法名；
2、客户端Stub程序接收到了客户端的功能调用请求，将客户端请求调用的方法名，携带的参数等信息做序列化操作，并打包成数据包。
3、客户端Stub查找到远程服务器程序的IP地址，调用Socket通信协议，通过网络发送给服务端。
4、服务端Stub程序接收到客户端发送的数据包信息，并通过约定好的协议将数据进行反序列化，得到请求的方法名和请求参数等信息。
5、服务端Stub程序准备相关数据，调用本地Server对应的功能方法进行，并传入相应的参数，进行业务处理。
6、服务端程序根据已有业务逻辑执行调用过程，待业务执行结束，将执行结果返回给服务端Stub程序。
7、服务端Stub程序**将程序调用结果按照约定的协议进行序列化，**并通过网络发送回客户端Stub程序。
8、客户端Stub程序接收到服务端Stub发送的返回数据，**对数据进行反序列化操作，**并将调用返回的数据传递给客户端请求发起者。
9、客户端请求发起者得到调用结果，整个RPC调用过程结束。
rpc需要使用到的术语
通过上文一系列的文字描述和讲解，我们已经了解了RPC的由来和RPC整个调用过程。我们可以看到RPC是一系列操作的集合，其中涉及到很多对数据的操作，以及网络通信。因此，我们对RPC中涉及到的技术做一个总结和分析：
1、动态代理技术： 上文中我们提到的Client Stub和Sever Stub程序，在具体的编码和开发实践过程中，都是使用动态代理技术自动生成的一段程序。
2、序列化和反序列化：  在RPC调用的过程中，我们可以看到数据需要在一台机器上传输到另外一台机器上。在互联网上，所有的数据都是以字节的形式进行传输的。而我们在编程的过程中，往往都是使用数据对象，因此想要在网络上将数据对象和相关变量进行传输，就需要对数据对象做序列化和反序列化的操作。
**序列化：**把对象转换为字节序列的过程称为对象的序列化，也就是编码的过程。
**反序列化：**把字节序列恢复为对象的过程称为对象的反序列化，也就是解码的过程。
我们常见的Json,XML等相关框架都可以对数据做序列化和反序列化编解码操作。后面我们要学习的Protobuf协议，这也是一种数据编解码的协议，在RPC框架中使用的更广泛。

