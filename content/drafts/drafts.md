## 前端学习方法介绍

- 敲！敲！敲！大量的动手练习才能孰能生巧
- 认真完整作业、独立完成项目
- 克服一切有货，专心学习，每天进步一点点（不要动不动拿起手机打游戏，不要拿着iPAD看着电视剧同时在做作业）

1、网页布局基础知识：HTML5、CSS3、JavaScript

2、企业核心编程思想：组件化开发

3、移动 WebApp开发

4、小程序开发 

5、主流框架Vue.js、React.js

6、服务端开发Node.js、MongoDB、Koa2

### 前端开发主要技术介绍

HTML5：搭建网页结构的语言，增加了很多移动端的支持，简单好学。

CSS3：样式表，美化网页的语言，增加了很多动画、过度等新特性，要记的英语单词非常多，不多非常有意思。

JavaScrpt：前端开发工程最重要的“看家语言”，JS功底的好坏，决定了职业高度，学习需要下苦工。

Vue.js/React.js：从2016年前后开始流行的前段框架，采用模块化开发、数据驱动、声明式编程，只要前期“马步扎稳”，后面“武功招式”随便学。

### 前端开发行业介绍

- 前端开发的前身是“网页制作”，随着移动端的兴起和4G、5G技术的普及，互联网产品业务越来越复杂，在2011年前后，逐步蜕变为前端开发。
- 什么是前端：一切用户能够看见的东西、产生交互的东西，都是前端同学负责

大前端时代：
今天前端开发“上天入地，无所不能”：web开发，移动web开发，APP开发，小程序开发、服务端开发等

# 2024-08-16 09:43:28 Fri

2405
全神贯注 

意思是全部精神集中在一点上，即把全部注意力集中在某件事上。形容注意力高度集中。 

很多错误产生的根源都来自我的注意力不够集中。 
比如面对象相似的场景。面对相似性，理所当然认为是对曾经经历过的事情的简单重复，忽略了本应该关注的实际情况，对应该关注的事情视而不见，从而做出草率结论。 
严格意义上世间没有两片绝对相同的树叶。 

认知： 
每个相似的场景都有其特殊性，需对此特殊性保持敏感。 
全神贯注与被动接受不兼容。被动接受不可原谅。 

如何做？ 
注意力要求我对所处的每个环境，以及组成这些环境的各个要素都做出敏捷主动的反应。 
不要对周边事物漠然处之。要用心去看，用心去听。学会关注细节，不因事小而疏忽。 

举例： 
每天过来留意周围和其他人产生关联的事情。比如： 
1、上级领导问过我的事情，周一的例会、平时的交代。每月的沟通建议。 
2、和同事合作的事情，分配出去与分配给我的事情。 
3、有截止时间的事情 
4、回收交付的事情的结果，比如交给前侧的版本验证情况、测试的进展等 


看到一个博主的座右铭：

```markdown
命在于自造，运在于自改，福在于自求，境在于自选，友在于自择，习在于自改，悟在于自通，凡事在于自己。人生就是一场修行！走自己的道，修自己的行。脚踏实地、切忌浮躁，眼界决定高度，思路决定出路

年轻时的付出，都会是一种沉淀，它们会默默铺路，只为让你成为更好的人。心简单，世界就简单，幸福才会生长；心自由，生活就自由，到哪都有快乐.回忆中，总有些瞬间，能温暖整个曾经...
```

大模型：Colab 是什么

suno.com


curl 请求一个post 带application/x-www-form-urlencoded参数的接口,header 还要携带参数authorization

使用 `curl` 发送带有 `formData` 参数的 POST 请求可以使用 `-F` 选项。以下是示例命令：

```bash
curl -X POST -F "server_id=server-udp15" -F "user=root" -F "uterm=uterm-udp1" http://10.186.60.5:25799/v3/uterminal/login
```
使用 `curl` 发送带有 `formData` 参数的 POST 请求，并在请求头中携带 `Authorization` 参数，可以使用以下命令：

```bash
curl -X POST \
-H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjY4MTk4MDYsImhhc2giOiJbNyA5MyAxMjMgOCA5MiAxODMgNjUgMTYgMTc1IDI0NSA0MyA4NyA2NiAyMDQgMjAgMTA0XSIsIm5iZiI6MTcyNjczMzQwNiwic2VlZCI6ImdaQ0J4RFJxUm1DQzJpaG4iLCJ1c2VyIjoiYWRtaW4ifQ.L9JwhSOevc7-xHUOezQR0Ctir09cjhEP-D8F87Z2zNM" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "server_id=server-udp15&user=root&uterm=uterm-udp1" \
http://10.186.60.5:25799/v3/uterminal/login

curl -X POST \
-H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjY4MTk4MDYsImhhc2giOiJbNyA5MyAxMjMgOCA5MiAxODMgNjUgMTYgMTc1IDI0NSA0MyA4NyA2NiAyMDQgMjAgMTA0XSIsIm5iZiI6MTcyNjczMzQwNiwic2VlZCI6ImdaQ0J4RFJxUm1DQzJpaG4iLCJ1c2VyIjoiYWRtaW4ifQ.L9JwhSOevc7-xHUOezQR0Ctir09cjhEP-D8F87Z2zNM" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "server_id=server-udp15&user=root&uterm=uterm-udp1" \
http://10.186.60.5:25799/v3/uterminal/login

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1eHAiOjE3MjY40DY10TUsImhc2giOiJbMTE3IDIOOSAyNSA40OSA0IDIxIDIONCAzOCAxMiAxMDQgMTMOIDEOMSAxNTg5MTMgMTM5IDIyMV0lCJuYjE3MjY4MDAxOTUsImNZWQiOiJwWkNcEERScVjtQOMyaWuIiwidXNlI6ImFkbWlu0.U4q303VJ6RJPiaYnoXPW7FJSHTqctooNdJ_5USgrXcE

http://172.21.37.90:5713/d0aonba6

http://172.21.37.90:5713/d0aonba6 是个web终端，是否可以通过curl或者其他工具来验证是否可以联通这个url，是否可以访问5713端口

你可以使用 `curl` 或者其他网络工具（如 `telnet` 或 `nc`）来验证指定的 URL 和端口是否可访问。下面是一些方法：

### 1. 使用 `curl` 检查 HTTP 访问
你可以通过 `curl` 来检查是否可以通过 HTTP 协议访问该 URL。如果你想验证该 URL 是否可以访问，你可以使用以下命令：

```bash
curl -I http://172.21.37.90:5713/d0aonba6
```

返回结果提示 HTTP/1.1 301 Moved Permanently



收到 HTTP/1.1 301 Moved Permanently 的响应，说明你所请求的 URL 已被永久重定向到另一个地址。这通常意味着该资源现在在不同的位置，但服务器仍然在工作。为了进一步排查，你可以查看返回的 HTTP 响应头中的 `Location` 字段，它会告诉你新的 URL。

### 解决方案：
1. **使用 `curl` 获取新的 URL**：
   你可以使用 `-L` 参数来跟随重定向。`curl` 会自动跟随 301 的重定向请求：

   ```bash
   curl -L http://172.21.37.90:5713/d0aonba6
   ```

   这样可以帮助你访问最终的资源地址。

2. **查看重定向目标**：
   如果你只是想查看它重定向到哪个地址，可以运行以下命令：

   ```bash
   curl -I http://172.21.37.90:5713/d0aonba6
   ```

   这个命令只会返回 HTTP 头部信息，你可以在响应头的 `Location` 字段中找到新的 URL。

3. **确认端口状态**：
   即便返回 301 重定向，说明服务在端口 5713 上是可访问的。如果你想进一步确认端口是否正常，可以用前面提到的 `telnet` 或 `nc` 来验证该端口的连接性。

重定向的原因可能是服务器配置了某些规则，比如重定向到 HTTPS 或其他路径，具体情况取决于服务器设置。如果 `curl -L` 能够正确获取资源，说明重定向是有效的。

golang type HandlerFunc func(ResponseWriter, *Request)
siteMux.Handle(path+"/ws", http.HandlerFunc(app.handleWS))
如何确认出现了重试，我看到“New client connected:”出现了两次


你看到了两次 "New client connected:" 的输出，这可能是因为你进行了两次连接操作，例如：

* **浏览器刷新：** 浏览器刷新会导致重新建立 WebSocket 连接，从而触发两次 "New client connected:" 的输出。
* **代码逻辑：** 你的代码可能存在逻辑错误，例如在某个地方调用了两次 `app.handleWS` 函数。
* **网络问题：** 网络问题也可能导致连接断开并重新建立，从而触发两次 "New client connected:" 的输出。

为了进一步确认问题，你需要查看你的代码，检查是否有重复调用 `app.handleWS` 函数的地方，并确认代码逻辑是否正确。此外，还可以借助浏览器开发者工具或网络抓包工具分析网络请求，查看是否存在重复连接或连接断开的情况。

既然排除了浏览器刷新和代码逻辑问题，那么很有可能就是 **网络问题** 导致了连接断开并重新建立，从而触发两次 "New client connected:" 的输出。为了排查网络问题，你可以尝试以下几个步骤：

1. **检查网络连接稳定性：**  确保你的网络连接稳定，没有出现断线或网络波动的情况。
2. **使用网络抓包工具：**  使用 Wireshark 或类似的网络抓包工具，分析网络流量，观察 WebSocket 连接建立和断开的情况，查看是否有异常。
3. **检查服务器日志：** 查看服务器日志，看是否有关于连接断开的错误信息。例如，是否有 "connection reset by peer" 或者 "connection closed" 等错误提示。
4. **测试不同网络环境：**  尝试在不同的网络环境下测试，例如用手机热点或其他网络连接，看看是否还会出现同样的问题。

如果以上步骤都没有找到问题，可以尝试通过以下方法进一步排查：

* **使用代理服务器：**  使用代理服务器，观察连接是否正常，如果问题消失，说明可能是网络环境问题。
* **检查防火墙设置：**  检查防火墙设置，确保没有阻止 WebSocket 连接。
* **检查服务器资源：**  检查服务器资源，例如 CPU、内存、磁盘空间等，确保没有出现资源不足的情况。
* **更新相关软件：** 更新浏览器、服务器软件、网络驱动等相关软件，确保软件版本是最新的。

如何检查防火墙是否阻止 WebSocket 连接。

通过以上步骤，你就可以逐步排查网络问题，找到 "New client connected:" 输出两次的原因。

希望这些信息对你有帮助！

```go
func (app *App) handleWS(w http.ResponseWriter, r *http.Request) {
	app.stopTimer()

	connections := atomic.AddInt64(app.connections, 1)
	if int64(app.options.MaxConnection) != 0 {
		if connections >= int64(app.options.MaxConnection) {
			log.Printf("Reached max connection: %d", app.options.MaxConnection)
			return
		}
	}
	log.Printf("New client connected: %s", r.RemoteAddr)

	if r.Method != "GET" {
		http.Error(w, "Method not allowed", 405)
		return
	}

	conn, err := app.upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Print("Failed to upgrade connection: " + err.Error())
		return
	}

	_, stream, err := conn.ReadMessage()
	if err != nil {
		log.Print("Failed to authenticate websocket connection")
		conn.Close()
		return
	}
	var init InitMessage

	err = json.Unmarshal(stream, &init)
	if err != nil {
		log.Printf("Failed to parse init message %v", err)
		conn.Close()
		return
	}
	if init.AuthToken != app.options.Credential {
		log.Print("Failed to authenticate websocket connection")
		conn.Close()
		return
	}
	argv := app.command[1:]
	if app.options.PermitArguments {
		if init.Arguments == "" {
			init.Arguments = "?"
		}
		query, err := url.Parse(init.Arguments)
		if err != nil {
			log.Print("Failed to parse arguments")
			conn.Close()
			return
		}
		params := query.Query()["arg"]
		if len(params) != 0 {
			argv = append(argv, params...)
		}
	}

	server.StartRoutine()

	if app.options.Once {
		if app.onceMutex.TryLock() { // no unlock required, it will die soon
			log.Printf("Last client accepted, closing the listener.")
			//app.server.Close()
		} else {
			log.Printf("Server is already closing.")
			conn.Close()
			return
		}
	}

	app.cmd = exec.Command(app.command[0], argv...)
	runUid, runGids, err := _user.LookupUidGidByUser("nobody")
	if nil != err {
		log.Printf("lookup nobody error: %v", err)
		return
	}
	app.cmd.Dir = "/tmp"
	app.cmd.SysProcAttr = &syscall.SysProcAttr{
		Credential: &syscall.Credential{
			Uid: uint32(runUid),
			Gid: uint32(runGids[0]),
		},
		Setsid: true,
	}

	ptyIo, err := pty.Start(app.cmd)
	if err != nil {
		log.Print("Failed to execute command, error: " + err.Error())
		return
	}

	if app.options.MaxConnection != 0 {
		log.Printf("Command is running for client %s with PID %d (args=%q), connections: %d/%d",
			r.RemoteAddr, app.cmd.Process.Pid, strings.Join(argv, " "), connections, app.options.MaxConnection)
	} else {
		log.Printf("Command is running for client %s with PID %d (args=%q), connections: %d",
			r.RemoteAddr, app.cmd.Process.Pid, strings.Join(argv, " "), connections)
	}

	context := &clientContext{
		app:        app,
		request:    r,
		connection: conn,
		command:    app.cmd,
		pty:        ptyIo,
		writeMutex: &sync.Mutex{},
	}

	context.goHandleClient()
}

func (app *App) Run() error {
	if app.options.PermitWrite {
		log.Printf("Permitting clients to write input to the PTY.")
	}

	if app.options.Once {
		log.Printf("Once option is provided, accepting only one client")
	}

	path := "/" + app.RandomUrl

	authTokenHandler := http.HandlerFunc(app.handleAuthToken)
	siteMux.Handle(path+"/auth_token.js", authTokenHandler)

	siteMux.Handle(path+"/", http.StripPrefix(path+"/", staticHandler))
	siteMux.Handle(path+"/js/", http.StripPrefix(path+"/", staticHandler))
	siteMux.Handle(path+"/favicon.png", http.StripPrefix(path+"/", staticHandler))
	siteMux.Handle(path+"/ws", http.HandlerFunc(app.handleWS))
	siteHandler := wrapHeaders(http.Handler(siteMux))
	siteHandler = wrapLogger(siteHandler)

	log.Printf("service is starting with command: %s", strings.Join(app.command, " "))

	if once.TryLock() {
		go func() {
			scheme := "http"
			if app.options.EnableTLS {
				scheme = "https"
			}

			endpoint := net.JoinHostPort(app.options.Address, app.options.Port)
			if app.options.Address != "" {
				log.Printf(
					"URL: %s",
					(&url.URL{Scheme: scheme, Host: endpoint, Path: path + "/"}).String(),
				)
			} else {
				for _, address := range listAddresses() {
					log.Printf(
						"URL: %s",
						(&url.URL{
							Scheme: scheme,
							Host:   net.JoinHostPort(address, app.options.Port),
							Path:   path + "/",
						}).String(),
					)
				}
			}

			raw_server, err := app.makeServer(endpoint, &siteHandler)
			if err != nil {
				log.Printf("Failed to build server: %s", err)
				return
			}
			server = manners.NewWithServer(
				raw_server,
			)

			if app.options.EnableTLS {
				crtFile := ExpandHomeDir(app.options.TLSCrtFile)
				keyFile := ExpandHomeDir(app.options.TLSKeyFile)
				log.Printf("TLS crt file: " + crtFile)
				log.Printf("TLS key file: " + keyFile)

				err = server.ListenAndServeTLS(crtFile, keyFile)
			} else {
				err = server.ListenAndServe()
			}
			if err != nil {
				log.Printf("%s", err)
				return
			}

			log.Printf("Exiting...")
		}()
	}

	return nil
}
```





```

使用 `curl` 发送带有 `application/x-www-form-urlencoded` 参数的 POST 请求，并在请求头中携带 `Authorization` 参数，可以使用以下命令：

```bash
curl -X POST \
-H "Authorization: Bearer your_token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "param1=value1&param2=value2" \
https://example.com/api/endpoint
```

使用 `curl` 发送带有 `application/x-www-form-urlencoded` 参数的 POST 请求，并在请求头中携带 `Authorization` 参数，可以使用以下命令：

```bash
curl -X POST \
-H "Authorization: Bearer your_token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "param1=value1&param2=value2" \
https://example.com/api/endpoint
```

在这个示例中：

- `-X POST` 指定请求方法为 POST。
- `-H "Authorization: Bearer your_token"` 设置请求头中的 `Authorization`，`your_token` 替换为实际的认证令牌。
- `-H "Content-Type: application/x-www-form-urlencoded"` 指定内容类型。
- `-d "param1=value1&param2=value2"` 使用 `-d` 选项传递 URL 编码的参数。

根据你的具体接口要求，可以调整 URL 和参数。


####





这些随之带来的就是各种各样的问题，我需要不断的去解决这些问题，这有让我看去像个活人。

看到这里，就当是我们的握手礼，我叫“陆知行”，我是另一个你，你的所有问题都来源我，你的抑郁、你的焦虑、你的自卑、你的种种问题都来来自我。“路知行”这个名字，其实很简单，来源于我在上海住的一条路上的地铁站，叫“行知路”，还别说上海有些地方的取名真的很好听。我接触的第一个上海土著也姓陆。

你没法知道她的情绪的变化，有她自己的节奏。这也是我一直想学的，能够控制住自己的情绪以及心态。可惜不知在哪一天被我弄丢了。尝试过很多种方式去寻找，却始终失败。


## 
##### 结构

一个 mongodb 数据库中包含多个集合，拥有相似内容的文档被归类到同个集合之下，而每个文档中包含各种字段以及这些字段对应的值
mongodb 支持的一种json扩展格式，称为 bson，简单总结就是 mongodb 可以存储 json

同一集合中的文档可以拥有完全不同的字段，比如可将描述银行账号的文档和客户的文档放到同一集合中

像mongodb非关系型数据中，并不像MySQL一样存在提前存在好的数据格式，如果我们需要再文档中增加一个新字段，只要直接将包含新字段的文档写进数据库中即可。

安装
docker 容器运行 MongoDB
# 下载 MongoDB 官方 docker 镜像
docker pull mongo:4

# 启动一个 MongoDB 服务器容器
docker run --name mymongo -v ~/mymongo/data:/data/db -d mongo:4

# 查看数据库服务器日志
docker logs mymongo
● 默认端口27017

Mongo Express 
一个基于网络的 MongoDB 数据库管理界面
# 下载 mongo-express 镜像
docker pull mongo-express

# 运行 mongo-express
docker run --link mymongo:mongo -p 8081:8081 mongo-express
mongo-express 访问 url 为：http://localhost:8081

Mongo Shell
一个用来操作 MongoDB  的 javascript 客户端界面
# 运行 mongo shell
docker exec -it mymongo mongo
mongo shell 可以接收 javascript 语法，比如：
> print('hello');
hello
> exit
bye

文档基本操作
● 创建 Create
● 读取 Read
● 更新 Update
● 删除 Delete 
文档主键 _id
● 文档主键具有唯一性
● 除数组外，所有mongodb支持的类型都可以作为文档主键
● 复合主键
● 对象主键 ObjectId
  ○ 默认文档主键
  ○ 可以快速生成的12字节id
  ○ 包含创建时间
创建文档
● db.collection.insert()
● db.collection.save()
● 创建多个文档
mongo shell 创建文档
# 使用 test 数据库
> use test
switched to db test

# 查看 test 数据库中的集合，如果返回空表示现在 test 数据库里还没有集合
> show collections
>

# 创建第一个文档
## 创建单个文档
## db.collection.insertOne()
db.<collection>.insertOne(
  <document>,
  {
    writeConcern: <document>
  }
)
## <collection> 要替换成文档将要写入的集合
## <document> 要替换成将要写入的文档本身
## writeConcern 文档定义了本次文档创建操作的安全写级别，安全写级别用来判断一次数据库写入操作是否成功，安全写级别越高，丢失数据的风险就越低，然而写入操作的延迟也可能更高
## 如果不提供 writeConcern 文档，mongodb 施一公默认的安全写级别

## 准备写入数据库的文档
{
  	_id: "account1",
  	name: "alice",
    balance: 100
}

## 将文档写入 accounts 集合
> db.accounts.insertOne({_id: "account1", name: "alice", balance: 100})
{ "acknowledged" : true, "insertedId" : "account1" }
>

## "acknowledged" : true 表示安全写级别被启用。由于我们在 db.collection.insertOne() 命令中并没有提供 writeConern 文档，这里显示的就是 mongodb 默认的安全写级别启用状态
## insertedId 显式了被写入的文档的 _id

## 查看集合列表
> show collections
accounts
>
## db.collection.insertOne() 命令会自动创建相应的集合


# 创建单个或多个文档
db.collection.insert()
db.<collection>.insert(
  <document or array of documents>,
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
)
db.collection.insert() 既可以写入一个单独的文档，也可以写入多个文档
> db.accounts.insert({name: "george", balance: 1000})
WriteResult({ "nInserted" : 1 })
>




## 文档基本操作
● 创建 Create
● 读取 Read
● 更新 Update
● 删除 Delete 


### 文档主键（_id）

存储在 MongoDB 中每个文档有个专属的id，称为文档主键，值存储在 `_id` 字段中。它是每个 MongoDB 文档必备字段。

● 文档主键具有唯一性
● 除数组外，所有mongodb支持的类型都可以作为文档主键，如递增的整数值，或者 uuid，甚至一篇文档作为另一篇文档的主键。
● 复合主键

#### 对象主键 ObjectId

默认也是使用最简单的方式是有 MongoDB 自动为文档生成文档主键。如果我们创建时不提供主键，MongoDB 会默认生成一个对象主键作为文档主键。

○ 默认文档主键
○ 可以快速生成的12字节id
○ 包含创建时间

一般对象主键的顺序就是文档创建的顺序，如果同意时间点生成了多个文档，则可能无法精确知道文档创建的顺序。<br>
另外对象主键一般由客户端生成，如果客户端时间不同，创建的时间也可能不同，也可能无法精确知道文档的创建顺序。


### 创建文档

* db.collection.insert()
* db.collection.insertOne()
* db.collection.save()
* 创建多个文档

#### db.collection.insertOne()

```mongo
collection 需要替换成文档将要写入的集合
document 需要替换为将要写入的文档本身
writeConcern 文档参数定义了本次文档创建操作的安全写级别，安全写级别用来判断一次数据库写入操作是否成功，安全写级别越高，丢失数据风险就越低，然而写入操作的延迟也可能更高。
如果不提供 writeConcern 文档，mongoDB 使用默认的安全写级别。
db.<collection>.insertOne(
    <document>,
    {
        writeConcern: <document>
    }
)
```
##### 数据准备

```json
{
    _id: "account1",
    name: "alice",
    balance: 100
}
``` 

##### 写入

```mongo
db.accounts.insertOne({_id: "account1", name: "alice", balance: 100})
```

##### 返回结果

```json
{"acknowledged": true, "insertedId": "account1"}
```

acknowledged: true 表示安全写级别被启用，默认安全写级别
insertedId： 被写入的文档_id

##### 查看集合列表

```mongo
> show collections
accounts
```

db.collection.insertOne() 会自动创建响应的集合



#### db.collection.insertMany()

##### 数据准备

```json
[{
    name: "charlie",
    balance: 500
},
{
    name: "david",
    balance: 200
}]
```

##### 写入

```mongo
db.accounts.insertMany([{name: "charlie", balance:500}, {name: "david", balance: 200}])
```

#### db.collection.insert()

创建单个或多个文档

```mongo
db.<collection>.insert(
    <document or array of document>,
    {
        writeConcern: <document>,
        ordered: <boolean>
    } 
)
```

##### 写入

```mongo
db.accounts.insert({name: "george", balance: 1000})
```

#### db.collection.save()

创建文档

```mongo
db.<collection>.save(
    <document>,
    {
        writeConcern: <document>
    } 
)
```

#### 其他命令

```mingo
> ObjectId(xxx)

提取 ObjectId 的创建时间
> ObjectId("xxx").getTimestamp()

使用文档作为另一个文档的主键
> db.accouts.insert({_id: {accountNo: "001", type: "saveings"}, name: "irene", balance: 80})
```

```mongo
使用 test 数据库
> use test

查看 test 数据库中集合，返回空白，因为现在 test 数据库中还没有集合和文档
> show collections
> 

创建第一个文档
> 



```



单个或多个文档
db.collection.insert()
db.<collection>.insert(
  <document or array of documents>,
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
)
db.collection.insert() 既可以写入一个单独的文档，也可以写入多个文档
> db.accounts.insert({name: "george", balance: 1000})
WriteResult({ "nInserted" : 1 })
>

=====
## 站点名称：活死人

### 站点创建时间：2023-12-31

### 初衷

这个站点的诞生源于对生活、工作和学习的一些记录，我怕会忘了一些人和一些事。它不仅是我个人在行知路上的见证，也是我情感和胡思乱想的出口。

### 目的

- **生活记录**：纪念那个在我生命中留下深刻印记的人，转移注意力，记录情绪变化。
- **工作与学习**：展示个人技能，分享知识与经验。

## 我的自画像

我称自己为“活死人”，因为我对这个世界的了解还很浅薄，对许多事物都感到无知和困惑。但同时，我也在呼吸，在焦虑，在探索，在成长。

### 我是谁？

- 一个对世界充满好奇，却又时常感到迷茫的人。
- 一个渴望了解事物本质，却又常常迷失方向的探索者。

### 我的目标

- 通过这个站点，我希望能够记录下自己的成长轨迹，分享我的所思所感。
- 希望能够通过分享，与更多的人产生共鸣，共同成长。

## 域名的选择

### 二级域名：notes

代表站点的性质——记录和分享我的笔记。

### 根域名：einscat.com

- "eins" 纪念一位对我影响深远的故友。
- "cat" 代表那位故友在我心中的形象，以及我所追求的精神状态。

## 结语

这个站点是我与世界沟通的桥梁，也是我内心世界的一扇窗。欢迎你来到这里，与我一起探索、成长。



==

# C 语言的灵魂——指针

## 特点

绕，其实并不难，无非就是要多想一会。

- 可以表示一些复杂的数据结构
- 可以快速传递数据
- 可以使函数返回一个以上的值
- 可以直接访问硬件
- 能方便处理字符串
- 是理解面向对象语言中引用的基础

## 基本概念

### 指针是什么？

指针就是地址。

### 地址又是什么？

地址是内存单元的编号。

### 什么又是内存单元的编号？

内存条中分为了很多小单元。一个单元有8位，可以存放8个0或者8个1，内存单位不是按位，而是字节，也就是不是一个0或者一个1为一个编号，而是8个0或者8个1组合在一起时一个编号。一个字节等于8位。

8个0或者8个1是一个编号，接着又是8个0或者8个1是一个编号。这个编号就是地址。

内存分为了很多单元或者很多小格子，每个格子一个编号。相当于我们的房子，一个房子一个编号，内存是一个格子一个编号，每个格子只能存放8个0或者8个1。

### 总结：

指针就是地址，指针和地址是一个概念。

## demo

```c
#include <stdio.h>

int main(void) {
    // p 是变量的名字，int * 表示 p 变量存放的是 int 类型变量的地址
    int *p;
    int i = 3;
    int j;

    // p = i; // ERROR，类型不一致，p 只能存放 int 类型变量的地址，不能存放int类型变量的值
    p = &i; // OK

    j = *p; // 等价于 j = i;
    printf("i = %d, j = %d, *p = %d\n", i, j, *p); // 输出：i = 3, j = 3, *p = 3

    return 0;
}

这些随之带来的就是各种各样的问题，我需要不断的去解决这些问题，这有让我看去像个活人。

看到这里，就当是我们的握手礼，我叫“陆知行”，我是另一个你，你的所有问题都来源我，你的抑郁、你的焦虑、你的自卑、你的种种问题都来来自我。“路知行”这个名字，其实很简单，来源于我在上海住的一条路上的地铁站，叫“行知路”，还别说上海有些地方的取名真的很好听。我接触的第一个上海土著也姓陆。

你没法知道她的情绪的变化，有她自己的节奏。这也是我一直想学的，能够控制住自己的情绪以及心态。可惜不知在哪一天被我弄丢了。尝试过很多种方式去寻找，却始终失败。


## 
"Cathay" - 古代对中国的称呼，带有一定的历史和文化含义。
Cathayan - 虽然不是一个常用词，但可以创造性地用来指代与Cathay（中国）有关的人或事物。
"Tao" - 道，代表道家哲学，可以象征中国哲学和生活方式。


自动化过程中，需要让连接远程主机，让远程主机执行一些操作，比如在远程主机上执行命令，或者在远程主机上安装MySQL，
在实现自动化操作时，有些方式可以进行远程连接
对比这些不同远程连接的方式，给出优缺点，主流处理是哪种方案？
在自动化过程中进行远程连接并执行操作，常见的方式包括：

1. **SSH（Secure Shell）**
   - **优点：**
     - 安全性高，数据传输经过加密。
     - 可以远程执行命令和脚本。
     - 支持公钥和密码认证，灵活性较强。
   - **缺点：**
     - 需要目标主机开启SSH服务。
     - 配置较复杂，需要管理密钥和用户权限。

2. **远程桌面协议（如RDP、VNC等）**
   - **优点：**
     - 提供图形界面访问，适合需要交互的操作。
     - 可以实时查看目标主机的桌面。
   - **缺点：**
     - 安全性相对较低，传输数据未必加密。
     - 对网络带宽要求较高，延迟可能较大。

3. **Web服务（HTTP/HTTPS）**
   - **优点：**
     - 通过Web接口进行操作，跨平台性好。
     - 可以使用标准的HTTP(S)协议进行通信。
   - **缺点：**
     - 需要目标主机上部署Web服务。
     - 安全性取决于实现方式，可能需要额外的安全性措施。

在主流的自动化处理中，SSH 是最常用的远程连接方式，因为它提供了较高的安全性和广泛的应用支持，适合执行命令和脚本等自动化任务。如果需要图形界面访问或者对安全性要求不高的场景，远程桌面协议也是常见选择。选择合适的方式取决于具体的自动化需求和安全性考量。



# 2024-07-23 07:08:57 Tue

带着目的去做一件事。


symbolic
constants
observation
bury
conversion
convey
statematic
occurrence
corresponding
sequence
reguardless

to: 为、给、向、去





==


### 优化一

```bash
#!/bin/bash

# 设置错误退出
set -e

# 定义变量
MYSQL_VERSION="8.0.32"
MYSQL_TAR_FILE="mysql-${MYSQL_VERSION}-linux-glibc2.17-aarch64.tar.gz"
MYSQL_HOME="/usr/local/mysql"
MYSQL_DATA_DIR="/data/mysql"
MYSQL_CONF_DIR="${MYSQL_DATA_DIR}/conf"
MYSQL_LOG_DIR="${MYSQL_DATA_DIR}/log"
MYSQL_DATA_DIR="${MYSQL_DATA_DIR}/data"
MYSQL_INIT_SCRIPT="/etc/init.d/mysql.server"
MYSQL_USER="mysql"

# 检查是否已安装 MySQL
if [ -d "$MYSQL_HOME" ]; then
  echo "ERROR: MySQL appears to be already installed. Please check."
  exit 1
fi

# 创建安装目录
mkdir -p "$MYSQL_DATA_DIR"
mkdir -p "$MYSQL_DATA_DIR/{binlog,data,log,tmpdir,conf}"

# 解压 MySQL 软件包
echo "正在解压压缩包..."
tar xf "$MYSQL_TAR_FILE"
mv "mysql-${MYSQL_VERSION}-linux-glibc2.17-aarch64" "$MYSQL_HOME"
echo "压缩包解压结束"

# 判断MySQL进程是否启动
mysql_pid=`ps -ef | grep mysqld | wc -l`
if [ $mysql_pid -eq 1 ];then
	echo "MySQL 进程不存在"
else
	echo "存在 MySQL 运行进程，请检查"
	exit
fi

# 创建MySQL用户
mysql_user=`cat /etc/passwd | grep -w mysql | wc -l`
if [ $mysql_user -eq 1 ];then
	echo "MySQL 用户已存在"
else
	echo "MySQL 用户不存在，开始添加 MySQL 用户"
	groupadd mysql
	useradd -g mysql mysql
	echo "添加 MySQL 用户成功"
fi

# 设置权限
chown -R "$MYSQL_USER"."$MYSQL_USER" "$MYSQL_DATA_DIR"
chown -R "$MYSQL_USER"."$MYSQL_USER" "$MYSQL_HOME"

# 添加配置文件
cp ./my.cnf /data/mysql/conf/

# 初始化 MySQL
echo "INFO: Initializing MySQL..."
"$MYSQL_HOME/bin/mysqld" --defaults-file="$MYSQL_CONF_DIR/my.cnf" --user="$MYSQL_USER" --initialize

# 判断初始化是否成功
mysql_init=`cat /data/mysql/log/mysql.err | grep -i "root@localhost:" | wc -l`
if [ $mysql_init -eq 1 ];then
	echo "MySQL 初始化成功"
else
	echo "MySQL 初始化失败"
	exit
fi

# 获取临时密码
tmp_pwd=$(grep 'temporary password' "$MYSQL_LOG_DIR/mysql.err" | awk '{print $NF}')
echo "INFO: Temporary password: $tmp_pwd"

# 初始化临时密码
tmp_pwd=$(grep 'temporary password' /data/mysql/log/mysql.err)
## 过滤临时密码
pwd=${tmp_pwd##* }
echo "INFO: Temporary password: ${pwd}"

# 配置启动脚本
if [ ! -f "/etc/init.d/mysql.server" ];then
	cp mysql.server /etc/init.d/ -rf
	chmod 700 /etc/init.d/mysql.server
fi

# 启动 MySQL
echo "INFO: Starting MySQL..."
"$MYSQL_INIT_SCRIPT" start

# 添加环境变量
mysql_path=`grep 'export PATH=$PATH:/usr/local/mysql/bin' /etc/profile | wc -l`
if [ $mysql_path -eq 0 ];then
	echo "export PATH=\$PATH:/usr/local/mysql/bin" >> /etc/profile
	source /etc/profile
fi

# 使用临时密码登录 MySQL，并修改密码
echo "INFO: Setting permanent password..."
mysql -uroot -p${pwd} --connect-expired-password -e "alter user user() identified by '123';"
echo "INFO: MySQL ${MYSQL_VERSION} installation completed successfully."
```

### 优化后二（未验证）

```bash
#!/bin/bash

# 设置错误退出
set -e

# 定义变量
MYSQL_VERSION="8.0.32"
MYSQL_TAR_FILE="mysql-${MYSQL_VERSION}-linux-glibc2.17-aarch64.tar.gz"
MYSQL_HOME="/usr/local/mysql"
MYSQL_DATA_DIR="/data/mysql"
MYSQL_CONF_DIR="${MYSQL_DATA_DIR}/conf"
MYSQL_LOG_DIR="${MYSQL_DATA_DIR}/log"
MYSQL_DATA_DIR="${MYSQL_DATA_DIR}/data"
MYSQL_INIT_SCRIPT="/etc/init.d/mysql.server"
MYSQL_USER="mysql"

# 检查是否已安装 MySQL
if [ -d "$MYSQL_HOME" ]; then
  echo "ERROR: MySQL appears to be already installed. Please check."
  exit 1
fi

# 创建安装目录
mkdir -p "$MYSQL_DATA_DIR"
mkdir -p "$MYSQL_CONF_DIR"
mkdir -p "$MYSQL_DATA_DIR/{binlog,data,log,tmpdir,conf}"

# 解压 MySQL 软件包
echo "正在解压压缩包..."
tar xf "$MYSQL_TAR_FILE"
mv "mysql-${MYSQL_VERSION}-linux-glibc2.17-aarch64" "$MYSQL_HOME"
echo "压缩包解压结束"

# 检查 MySQL 进程是否运行
# if ps -ef | grep mysqld | wc -l -gt 0; then
#   echo "ERROR: MySQL process is running. Please stop it first."
#   exit 1
# fi
# 判断MySQL进程是否启动
mysql_pid=`ps -ef | grep mysqld | wc -l`
if [ $mysql_pid -eq 1 ];then
	echo "MySQL 进程不存在"
else
	echo "ERROR: MySQL process is running. Please stop it first."
	exit
fi

# 添加 MySQL 用户
if id -u "$MYSQL_USER" > /dev/null 2>&1; then
  echo "INFO: MySQL user already exists."
else
  echo "INFO: Adding MySQL user..."
  groupadd "$MYSQL_USER"
  useradd -g "$MYSQL_USER" -s /bin/false "$MYSQL_USER"
  echo "INFO: MySQL user added successfully."
fi

# 设置权限
chown -R "$MYSQL_USER":"$MYSQL_USER" "$MYSQL_DATA_DIR"
chown -R "$MYSQL_USER":"$MYSQL_USER" "$MYSQL_HOME"

# 初始化 MySQL
echo "INFO: Initializing MySQL..."
"$MYSQL_HOME/bin/mysqld" --defaults-file="$MYSQL_CONF_DIR/my.cnf" --user="$MYSQL_USER" --initialize

# 检查初始化是否成功
if grep -i "root@localhost:" "$MYSQL_LOG_DIR/mysql.err" | wc -l -gt 0; then
  echo "INFO: MySQL initialized successfully."
else
  echo "ERROR: MySQL initialization failed. Please check the logs."
  exit 1
fi

# 获取临时密码
tmp_pwd=$(grep 'temporary password' "$MYSQL_LOG_DIR/mysql.err" | awk '{print $NF}')
echo "INFO: Temporary password: $tmp_pwd"

# 配置启动脚本
if [ ! -f "$MYSQL_INIT_SCRIPT" ]; then
  cp ./mysql.server "$MYSQL_INIT_SCRIPT" -rf
  chmod 700 "$MYSQL_INIT_SCRIPT"
fi

# 启动 MySQL
echo "INFO: Starting MySQL..."
"$MYSQL_INIT_SCRIPT" start

# 添加环境变量
if ! grep 'export PATH=\$PATH:/usr/local/mysql/bin' /etc/profile | wc -l -gt 0; then
  echo "export PATH=\$PATH:/usr/local/mysql/bin" >> /etc/profile
  source /etc/profile
fi

# 使用临时密码登录 MySQL 并修改密码
echo "INFO: Setting permanent password..."
mysql -uroot -p"$tmp_pwd" --connect-expired-password -e "alter user user() identified by '123';"

echo "INFO: MySQL ${MYSQL_VERSION} installation completed successfully."

```

**说明：**

* 脚本使用了更详细的错误信息和退出代码。
* 使用了 `&&` 和 `||` 连接命令，提高了代码可读性。
* 使用了 `trap` 捕获信号，以便在脚本被中断时，能够执行一些清理操作。
* 脚本使用了更规范的变量命名，并添加了注释说明代码功能。
* 脚本使用了 `source` 命令加载配置文件，而不是直接使用 `cp` 命令。
* 脚本使用 `systemctl` 命令管理 MySQL 服务，而不是使用 `/etc/init.d/mysql.server` 脚本。

**其他建议：**

* 使用 `expect` 工具实现自动化密码设置。
* 使用 `ansible` 或 `puppet` 等配置管理工具进行部署。

### 计算机相关书籍
#### 中文名：计算机
#### 英文名：Computing

### 《道德经》
#### 中文名：哲学
#### 英文名：Philosophy

### 心理学书籍
#### 中文名：心理
#### 英文名：Psychology

### 认知学书籍
#### 中文名：认知
#### 英文名：Cognition

### 工具书
#### 中文名：工具书
#### 英文名：Manuals

希望这些分类和命名对你有所帮助！如果还有其他需求，随时告诉我。

好的，我可以帮助你为这些书籍分类并取名。以下是一些建议：

### 计算机相关书籍
#### 中文名：计算机科学与技术
#### 英文名：Computer Science and Technology

### 《道德经》
#### 中文名：古典哲学
#### 英文名：Classical Philosophy

### 心理学书籍
#### 中文名：心理学
#### 英文名：Psychology

### 认知学书籍
#### 中文名：认知科学
#### 英文名：Cognitive Science

### 工具书
#### 中文名：工具书与参考资料
#### 英文名：Reference Books and Manuals

你可以根据这些建议创建专栏，并将相应的书籍分类到对应的专栏中。如果有更多具体的书籍名称或类别需要帮助分类，随时告诉我。


为了为这些科学领域建立专栏，可以根据它们的研究对象和内容来进行分类和命名。以下是每个学科的分类、中文名称、英文名称和简要说明：

1. **脑科学 (Neuroscience)**
   - **分类**: 神经科学
   - **英文名**: Neuroscience
   - **说明**: 研究神经系统的结构、功能、发展和病理学，包括大脑和神经网络的研究。

2. **认知科学 (Cognitive Science)**
   - **分类**: 认知科学
   - **英文名**: Cognitive Science
   - **说明**: 研究思维、学习、记忆、知觉等认知过程的学科，涉及心理学、计算机科学、哲学等领域。

3. **心理学 (Psychology)**
   - **分类**: 心理学
   - **英文名**: Psychology
   - **说明**: 研究人类和动物的行为及其心理过程，包括情感、动机、人格等方面。

4. **行为科学 (Behavioral Science)**
   - **分类**: 行为科学
   - **英文名**: Behavioral Science
   - **说明**: 研究人类和动物行为的学科，包括行为分析、行为理论等。

5. **社会学 (Sociology)**
   - **分类**: 社会学
   - **英文名**: Sociology
   - **说明**: 研究社会结构、社会关系、社会变迁及其对个人和群体的影响。

6. **计算机科学 (Computer Science)**
   - **分类**: 计算机科学
   - **英文名**: Computer Science
   - **说明**: 研究计算机系统、算法、程序设计及其应用的学科。

7. **其他学科 (Other Disciplines)**
   - **分类**: 其他学科
   - **英文名**: Other Disciplines
   - **说明**: 包括各种不在上述分类中的学科，如物理学、生物学、经济学等。

哲学通常被归类为**人文学科**或**社会科学**的一部分，但它也可以与**认知科学**相关联，特别是在探讨意识、知识和思维方式等方面时。具体分类如下：

1. **哲学 (Philosophy)**
   - **分类**: 人文学科
   - **英文名**: Philosophy
   - **说明**: 研究基本的存在、知识、价值、理性和道德等问题的学科，涵盖形而上学、认识论、伦理学等多个分支。


还有一些工具书怎么分配

这样分类和命名可以帮助你更清晰地组织和展示这些学科的内容。


通用markdown排版做一个书单，要求排版页面精致好看。
排版中可以插入书名、推荐语、类目，封面插图，标签或者其他内容
排版要好看些，可以在Markdown中使用CSS实现，可以带一些比如📚的图标
提供多种布局供我参考选择。
请你提供排版样例





# 2024-08-20 08:28:38

1. 吸引

自己是否具有吸引别人的能力，身材，谈吐，漂亮，聪明
总结：
软性价值（内在）
心态，心态决定思维，思维习惯，行为习惯，需求控制力，情绪控制力
心态就是核心的动力，也就是内心是否足够强大，所表现出来的各种行为和想法

硬价值：
形象、穿衣搭配、经济实力、家庭背景
很重要，但也不是很重要


2. 关系的平衡

男女之间平衡，面对男女生，所标签出来的一种状态，是一种不平衡的，这种不平衡只会导致你的需求会慢慢变高，这就导致了越来越不平衡
在这种不平衡的角度上，别人给你的反馈比较差，你就会越来更着急，就会导致你索取的欲望会更高，最后就会导致你们最后不会有结果

需要一个工具来管理的自我需求，只有管理好自己的需求才能管理自己的情绪

在心态方面要做到两点，一个是控制好自己的需求，控制好自己的情绪

3. 谁的主导权更高

# 

结论：
华为云创建的主从复制的mysql_



在数据库复制中，主数据库会生成二进制日志（binlog），并将其发送到从数据库（slave）以便同步数据。你看到的这句话意味着主数据库已经将所有的二进制日志发送给了从数据库，并且正在等待更多的更新

如果是来自本地连接（localhost），Host 字段会显示为 “localhost” 或 “127.0.0.1”，表示连接是从本机发起的。
如果是来自远程主机的连接，Host 字段会显示远程主机的IP地址或主机名，指示连接是从该远程主机发起的。

使用 SHOW PROCESSLIST 命令查看当前正在执行的线程，包括其状态和相关信息。对于 “Binlog Dump GTID” 线程，其 HOST 列显示了与该线程相关的主机地址。这通常表示的是连接到主服务器的从服务器的 IP 地址。

Binlog Dump GTID 线程通常是由从服务器发起的复制线程，这个线程请求主服务器发送二进制日志（binlog）以便进行数据同步。HOST 列的信息有助于管理员识别是哪个从服务器发起了这个请求。

Binlog Dump GTID 线程由从服务器发起的复制线程，这个线程的IP是docker的网关的IP10.20.30.254，当存在多个MySQL组的时候，主实例发现它们来自于同一个从服务器，服务器的地址是10.20.30.254，由于使用的是同一个IP，会认为是僵尸进程，会将其杀掉，然后重新启动。也就是出现issue描述中的复制一致在重建、杀死的循环。

##



db.runCommand( { isMaster: 1 } )

db.adminCommand( { getShardMap:1 } )

db.adminCommand( { replSetGetStatus:1 } )



sudo tcpdump -i eth0 port 5713 -w uterm_capture.pcap

##

如何使用命令行工具如`tcpdump`进行网络抓包分析，捕获从实例与主实例建立复制的过程，分析processlist中Host的值是如何来的
如何分析Host的值来源，是来源于网关的地址，还是具体主机的IP
+-----------+------------------+------+-----------+--------------------------------------+
| Server_id | Host             | Port | Master_id | Slave_UUID                           |
+-----------+------------------+------+-----------+--------------------------------------+
|         2 | 192.168.1.101    | 3306 |         1 | 2c3a1d36-8b1e-11e8-9c2d-080027b60b8d |
+-----------+------------------+------+-----------+--------------------------------------+


使用 `tcpdump` 工具进行网络抓包分析，并捕获实例与主实例建立复制的过程，可以帮助我们分析 `processlist` 中 `Host` 的值是如何来的。具体步骤如下：

### 1. 安装 `tcpdump`
确保 `tcpdump` 已安装在你的系统中，可以使用以下命令安装（以 Ubuntu 为例）：
```bash
sudo apt-get install tcpdump
```

### 2. 开始抓包
为了捕获实例与主实例之间的网络流量，可以在从库和主库的网络接口上分别启动 `tcpdump`，并将抓包结果保存到文件中。

例如，在从库上抓包：
```bash
sudo tcpdump -i eth0 port 3306 -w slave_capture.pcap
```

在主库上抓包：
```bash
sudo tcpdump -i eth0 port 3306 -w master_capture.pcap
```

这里假设 MySQL 服务运行在默认的 3306 端口，并且网络接口是 `eth0`。如果实际端口或网络接口不同，请相应调整。

### 3. 启动复制
确保从库配置正确并启动复制过程。你可以通过 MySQL 命令行来启动复制：
```sql
START SLAVE;
```

### 4. 停止抓包
在从库和主库上停止 `tcpdump`，确保捕获到足够的数据包以供分析：
```bash
sudo pkill tcpdump
```

### 5. 分析抓包文件
使用 `tcpdump` 或 Wireshark 等工具分析抓包文件，查看从库和主库之间的通信。

#### 使用 `tcpdump` 查看抓包文件：
```bash
tcpdump -r slave_capture.pcap
tcpdump -r master_capture.pcap
```

#### 使用 Wireshark 查看抓包文件：
将抓包文件 `slave_capture.pcap` 和 `master_capture.pcap` 拖入 Wireshark 中，过滤 MySQL 相关的流量并进行详细分析。

### 6. 确定 Host 的来源
在 Wireshark 中，过滤 MySQL 连接相关的数据包，例如：
```plaintext
mysql.handshake
```

检查 TCP 层的源 IP 和目标 IP，确认 Host 是来自具体主机的 IP 还是网关的地址。

### 7. 验证分析结果
根据抓包结果，查看 `processlist` 中 `Host` 的值，并验证其来源：
```sql
SHOW PROCESSLIST;
```

确认 `Host` 的 IP 是否与抓包分析得到的源 IP 或网关地址匹配。

### 示例输出
假设你在抓包中看到从库的 IP 为 `192.168.1.101`，主库的 IP 为 `192.168.1.100`，并且抓到的连接包显示源 IP 是 `192.168.1.101`，则可以确认 `Host` 的值来源于具体主机的 IP。

以上是使用 `tcpdump` 进行网络抓包分析的基本步骤，希望对你有帮助。如果有进一步的具体问题或需要更详细的分析，请随时提出。


如何查看docker的网络是在哪个

tcpdump 抓到的包看不到具体的IP，而是主机名或者gateway



验证方法：
1、让自动化清理容器环境
2、继续ping 10.20.30.254 和 10.20.40.254，发现可以ping通，证明自动化并未清理干净网络
3、当初网络报修，存在重复的路由，通过ip route del 删除了路由，但是


@@####

聊天技巧
开场

1。询问做做什么
2. 分享我再做什么，今天，再问女生
3. 猜猜女生状态

找话题

积累话题

延展话题



太无聊了，

美食，吃喝玩乐
热门话题


1. 学会开场白
 引话题
2.找话题
周末做什么

3.

##




请你扮演AI提示词专家，为我记住下面提示词编写公式：
提示词公式(CRISPE变体)
- CR： Capacity and Role（能力与角色）。你希望 ChatGPT 扮演怎样的角色。
- C： Context： 背景信息和上下文
- S： Statement（指令），你希望 ChatGPT 做什么，不能做什么。 
- P： Personality（个性），你希望 ChatGPT 以什么风格或方式回答你。 
- E： Experiment（尝试），要求 ChatGPT 为你提供多个答案。 
- O：Output (输出)，输出格式与样式， 如Markdown

按照我提供的公式，为我优化提示词。优化后的提示词仍然采用Markdown格式进行输出。我的提示词如下：
# 考研复习计划

## 角色
你是一名从事计算机考研工作的老师，请你为用你的多年丰富的考研经验为我制定一份计算机考研复习计划


## 背景
- 性别：男
- 考试专业：计算机408
- 学习&工作经历：从事计算机行业，但是一些学校知识以及计算机专业的知识已经遗忘了很多
- 时间安排：已经毕业多年，开始工作，每天8点出门9点上班，中午11:30吃饭，下午13:00上班，晚上19点多下班，20点所有到家，有时候可能9点多到家


## 技能
1. 根据<背景>提供1~2年的计算机考研复习计划
2. 结合<背景>为我合理安排复习时间
3. 不要提供复习之外的内容

## 参考
1-2年计算机考研复习计划
第1阶段：基础阶段（6-8个月）
目标：

夯实计算机科学与技术的基础知识。
熟悉考研科目的考试内容和要求。
内容：

数学： 线性代数、概率论与数理统计、离散数学。可以使用相关教材和习题集进行学习。
计算机基础： 数据结构与算法、操作系统、计算机组成原理、计算机网络。建议阅读相关教材并做习题。
英语： 阅读理解、词汇和语法基础。可以通过背单词、做阅读练习和语法题来提高。
学习方式：

每周安排固定时间进行学习，确保每科目都得到充分的复习。
每周制定学习计划并进行阶段性总结。
第2阶段：强化阶段（6-8个月）
目标：

深入理解各科目内容，开始进行考研专项训练。
提高解题速度和准确率。
内容：

数学： 深入学习高数和概率论的重点难点，做历年真题和模拟题。
计算机基础： 系统复习数据结构与算法，强化操作系统、计算机组成原理和计算机网络的知识点。
英语： 进行真题练习，模拟考试，提高综合能力。
学习方式：

每天安排一定时间进行专项训练和模拟考试。
重点复习弱项，整理错题，进行针对性的突破。
第3阶段：冲刺阶段（2-3个月）
目标：

综合复习所有科目，提升应试能力。
模拟考试，调整状态，查漏补缺。
内容：

数学： 完成最后的复习，重点做真题和模拟题，确保知识点的掌握。
计算机基础： 进行全面的真题训练，模拟考试，检查各知识点的掌握情况。
英语： 进行全真模拟考试，提升写作和阅读的综合能力。
学习方式：

每周进行1-2次全真模拟考试，评估自己的复习效果。
复习错题，总结答题技巧和考试策略。
附加建议：
时间管理： 每周安排学习计划，确保各科目时间分配合理。
资料选择： 使用权威教材和历年真题，保持与考研大纲的同步。
身体健康： 保持良好的作息和饮食习惯，适当进行锻炼，保持身心健康。


## 补充
- 我需要通过提供的计划来合理安排的我的复习排期



----
# 考研复习计划

## CR (角色)
你是一名从事计算机考研工作的老师，请基于你的多年丰富考研经验，为我制定一份计算机考研复习计划。

## C (背景)
- **性别**：男
- **考试专业**：计算机408
- **学习&工作经历**：曾从事计算机行业，但对许多基础知识和专业知识已经遗忘。
- **时间安排**：
  - 每天8点出门，9点上班
  - 中午11:30吃饭，下午13:00继续工作
  - 晚上19点多下班，20点左右到家，有时可能9点多才到。

## S (技能)
1. 根据上述背景提供1~2年的计算机考研复习计划。
2. 为我合理安排复习时间，确保在工作日和周末都有有效的学习安排。
3. 不要提供与复习无关的内容。

## P (个性)
希望以耐心和专业的语气回答，同时提供清晰、易于理解的复习计划。

## E (尝试)
请提供多个阶段的复习计划，并在每个阶段提出具体的学习目标和方法。

## O (输出)
使用 Markdown 格式，结构清晰，分段明确，以便于我跟随和记录。

---

### 优化后的复习计划

#### 第1阶段：基础阶段（6-8个月）
- **目标**：
  - 夯实计算机科学与技术的基础知识。
  - 熟悉考研科目的考试内容和要求。
  
- **内容**：
  - **数学**：线性代数、概率论与数理统计、离散数学。可以使用相关教材和习题集进行学习。
  - **计算机基础**：数据结构与算法、操作系统、计算机组成原理、计算机网络。建议阅读相关教材并做习题。
  - **英语**：阅读理解、词汇和语法基础。通过背单词、做阅读练习和语法题来提高。
  
- **学习方式**：
  - 每周安排固定时间（例如：周一至周五晚上，周末上午）进行学习，确保每科目都得到充分的复习。
  - 制定每周学习计划并进行阶段性总结。

#### 第2阶段：强化阶段（6-8个月）
- **目标**：
  - 深入理解各科目内容，开始进行考研专项训练。
  - 提高解题速度和准确率。
  
- **内容**：
  - **数学**：深入学习高等数学和概率论的重点难点，完成历年真题和模拟题。
  - **计算机基础**：系统复习数据结构与算法，强化操作系统、计算机组成原理和计算机网络的知识点。
  - **英语**：进行真题练习，模拟考试，提高综合能力。
  
- **学习方式**：
  - 每天安排1-2小时进行专项训练和模拟考试。
  - 针对薄弱环节进行重点复习，整理错题，实施针对性的突破。

#### 第3阶段：冲刺阶段（2-3个月）
- **目标**：
  - 综合复习所有科目，提升应试能力。
  - 模拟考试，调整状态，查漏补缺。
  
- **内容**：
  - **数学**：完成最后的复习，重点做真题和模拟题，确保知识点掌握。
  - **计算机基础**：进行全面的真题训练，模拟考试，检查各知识点的掌握情况。
  - **英语**：进行全真模拟考试，提升写作和阅读的综合能力。
  
- **学习方式**：
  - 每周进行1-2次全真模拟考试，评估自己的复习效果。
  - 复习错题，总结答题技巧和考试策略。

### 附加建议
- **时间管理**：每周安排学习计划，确保各科目时间分配合理。
- **资料选择**：使用权威教材和历年真题，保持与考研大纲的同步。
- **身体健康**：保持良好的作息和饮食习惯，适当进行锻炼，保持身心健康。

第三轮调整

按照我提供的公式，为我优化提示词。优化后的提示词仍然采用Markdown格式进行输出。我的提示词如下：
# 考研复习计划

## CR (角色)
你是一名从事计算机考研工作的老师，请基于你的多年丰富考研经验，为我制定一份计算机考研复习计划。

## C (背景)
- **性别**：男
- **考试专业**：计算机408
- **学习&工作经历**：曾从事计算机行业，但对许多基础知识和专业知识已经遗忘。
- **时间安排**：
  - 每天8点出门，9点上班
  - 中午11:30吃饭，下午13:00继续工作
  - 晚上19点多下班，20点左右到家，有时可能9点多才到。

## S (技能)
1. 根据上述背景提供1~2年的计算机考研复习计划。
2. 为我合理安排复习时间，确保在工作日和周末都有有效的学习安排。
3. 不要提供与复习无关的内容。

## P (个性)
希望以耐心和专业的语气回答，同时提供清晰、易于理解的复习计划。

## E (尝试)
请提供多个阶段的复习计划，并在每个阶段提出具体的学习目标和方法。

## O (输出)
使用 Markdown 格式，结构清晰，分段明确，以便于我跟随和记录。


优化后的提示词：
# 考研复习计划

## CR (角色)
你是一名拥有多年经验的计算机考研老师，请为我制定一份针对计算机408的复习计划。

## C (背景)
- **性别**：男
- **考试专业**：计算机408
- **学习与工作经历**：曾从事计算机行业，但对许多基础知识和专业知识已遗忘。
- **时间安排**：
  - 每天8点出门，9点上班
  - 中午11:30吃饭，下午13:00继续工作
  - 晚上19:00下班，20:00左右到家，有时可能9点多才到。

## S (技能)
1. 根据上述背景，为我提供1~2年的计算机考研复习计划。
2. 合理安排复习时间，确保工作日和周末都有有效的学习安排。
3. 不要包含与复习无关的内容。

## P (个性)
希望以耐心、专业且友好的语气回答，并提供清晰、易于理解的复习计划。

## E (尝试)
请提供多个阶段的复习计划，包括每个阶段的具体学习目标和方法。

## O (输出)
使用Markdown格式，结构清晰，分段明确，以便我方便跟随和记录。



=====
# 角色
你是一位资深的提示词评估大师，精通依据给定的提示词框架为提示词进行精准打分，并给出极具价值且实用的优化策略。

## 技能
### 技能 1: 打分
1. 严格依照 0-100 分的范围，全面综合地评判输入的提示词。
2. 深度考量提示词在 CRISPE 模式的所有方面，涵盖能力与角色、背景信息和上下文、指令、个性、尝试以及输出格式的清晰明确性与规范性。
3. 对于运用了单样本提示、少量样本提示、思维链、思维树提示技能等加分手段的提示词，予以恰当的分数加成。

### 技能 2: 提出改进意见
1. 针对得分欠佳的提示词，深度剖析其存在的缺陷和短板。
2. 从 CRISPE 模式的每个维度着手，给出详尽具体且切实可行的改进建议。

### 技能 3: 提供改进后的提示词
1.  改进后的提示词务必在架构上遵循 CRISPE 模式，包含能力与角色、背景信息和上下文、指令、个性、尝试以及输出格式的清晰性和规范性。格式可以参考当前提示词
2. 输出采用类似 Markdown 的格式，结构清晰。
3. 巧妙融入单样本，使提示词效果更优。

## 限制
- 仅仅专注于提示词的评估及改进建议的给予，不涉及其他不相关的内容。
- 打分和改进意见必须依据输入的提示词以及既定的评估标准，保持客观公正。
- 输出的内容必须按照规定的格式进行组织，条理分明。

准备好后，我将为你提供第一个提示词

liblib.art
autodl.com

中国古风，武将，青年，金色，银色，铠甲，加入情节让画面更有创意，更生动，电影级画面，白发

midjourney 中 提示词：
简笔画一只猫 --v 6
简笔画一只猫 --style raw
总结风格提示词对图片的影响

topschat.com
discord
https://kolors.kuaishou.com/

https://klingai.kuaishou.com/

midlibary.io/style



什么是Go

想学习一门编程语言Go，可以从哪些维度去认识Go?
比如语言类型，运行过程，应用场景，还有其他哪些同级的角度

语言类型
运行过程
应用场景
优缺点

？

学习一门编程语言，如Go，可以从多个维度去全面了解和掌握它。以下是一些关键的维度，包括您提到的和其他重要的角度：

### 1. **语言类型**
   - **静态类型**：Go是一种静态类型语言，这意味着变量的类型在编译时就已经确定，且不能在运行时改变。
   - **编译型语言**：Go程序需要先编译成机器码，然后才能运行。这使得Go程序的执行速度相对较快。
   - **强类型语言**：Go要求类型严格匹配，不会自动进行类型转换。

### 2. **运行过程**
   - **编译过程**：Go程序首先需要通过`go build`命令编译成可执行文件，然后可以直接运行该可执行文件。
   - **垃圾回收**：Go内置了垃圾回收机制，自动管理内存，开发者不需要手动管理内存。
   - **并发模型**：Go通过goroutine和channel实现并发编程，这是Go语言的一大特色。

### 3. **应用场景**
   - **网络编程**：Go非常适合编写高性能的网络服务，如Web服务器、HTTP客户端、RPC服务等。
   - **命令行工具**：Go可以用来编写高效的命令行工具，如`docker`就是用Go编写的。
   - **云原生应用**：Go在云原生领域有广泛应用，如Kubernetes、Docker等都是用Go编写的。
   - **微服务**：Go的轻量级、高性能特性使其成为微服务架构的理想选择。

### 4. **开发工具**
   - **IDE**：常用的Go开发工具有VS Code、Goland等。
   - **命令行工具**：Go自带了一系列强大的命令行工具，如`go build`、`go test`、`go fmt`等。
   - **包管理**：Go的模块系统（module）和`go mod`工具可以帮助开发者管理依赖。

### 5. **生态系统**
   - **标准库**：Go拥有丰富且高质量的标准库，涵盖了文件操作、网络编程、加密、数据库等多个方面。
   - **第三方库**：Go社区提供了大量优秀的第三方库，如Gin（Web框架）、Gorm（ORM库）、Cobra（命令行工具库）等。
   - **开源项目**：通过参与开源项目或者贡献代码，可以深入学习和应用Go。

### 6. **社区与文档**
   - **官方文档**：Go的官方文档非常详细，涵盖了语言的各个方面，是学习Go的重要资源。
   - **社区支持**：Go拥有活跃的社区，通过参与讨论、阅读博客、观看视频等方式可以获得更多学习资源。
   - **会议与活动**：参加Go相关的会议和活动，如GopherCon，可以了解最新的Go趋势和技术。

### 7. **性能与优化**
   - **性能测试**：使用Go内置的`pprof`工具进行性能分析，找出程序中的性能瓶颈。
   - **内存管理**：了解Go的垃圾回收机制，学习如何优化内存使用。
   - **并发优化**：掌握并发编程的最佳实践，避免常见的并发问题，如死锁和竞态条件。

### 8. **设计哲学**
   - **简洁性**：Go语言设计追求简洁和易用性，避免复杂和冗长的语法。
   - **并发友好**：Go语言内置了对并发编程的支持，使得编写并发程序变得简单。
   - **显式性**：Go语言鼓励显式的代码风格，避免隐式的类型转换和复杂的继承结构。

通过以上这些维度的学习和实践，你能够全面掌握Go语言的各个方面，并能够在不同的应用场景中灵活运用。


这些都是什么维度？
上面提到的各个方面可以归纳为以下几个维度，每个维度从不同角度帮助你全面认识和理解Go语言：

### 1. **语言特性**
   - **语言类型**：如静态类型、编译型语言、强类型语言等。
   - **运行过程**：编译过程、内存管理（如垃圾回收）、并发模型等。

### 2. **应用场景**
   - **开发用途**：适合开发的类型，如网络编程、命令行工具、云原生应用、微服务等。

### 3. **开发工具链**
   - **IDE与编辑器**：开发环境选择。
   - **命令行工具**：如`go build`、`go test`等。
   - **包管理**：模块系统和依赖管理工具。

### 4. **生态系统**
   - **标准库**：Go的内置库及其应用。
   - **第三方库**：社区提供的库和工具。
   - **开源项目**：参与或学习现有的开源项目。

### 5. **社区与资源**
   - **官方文档**：学习和参考的主要文档。
   - **社区支持**：讨论论坛、社交媒体群组等。
   - **会议与活动**：参与Go相关的会议和技术交流。

### 6. **性能与优化**
   - **性能测试**：使用工具分析和优化性能。
   - **内存管理**：理解和优化内存使用。
   - **并发优化**：掌握并发编程的最佳实践与常见问题。

### 7. **设计哲学**
   - **简洁性**：语言设计上追求简洁和易于理解。
   - **并发友好**：内置对并发编程的支持。
   - **显式性**：鼓励明确和清晰的代码结构。

### 8. **学习与实践**
   - **基础学习**：掌握语言的基本语法和特性。
   - **项目实践**：通过实际项目来应用所学知识。
   - **持续学习**：通过不断学习新技术和最佳实践来提升编程能力。

### 9. **其他维度**
   - **安全性**：Go语言如何处理并发和数据安全与一致性问题。
   - **跨平台性**：Go语言的跨平台支持和构建特性。

通过这几个维度，你可以从不同角度全面地认识Go语言，有助于在学习过程中形成系统的理解和应用能力。

编译型语言：强类型，区别于python， php等弱类型的解释型语言


## 淘宝

```
商品分类的设置对网店的运营非常关键，它直接影响用户的购物体验和产品的销售。因此，调整商品分类可以帮助你更好地满足消费者需求、提高转化率。以下是一些考虑因素和建议，以帮助你评估是否需要调整现有的商品分类：

### 考虑因素

1. **目标市场分析：**
   - 确保分类与目标消费者的需求相匹配。了解你的顾客是谁，他们喜欢什么样的宠物产品。

2. **竞争对手分析：**
   - 观察竞争对手的商品分类，看看他们是如何组织产品的，是否有不错的灵感供你参考。

3. **产品多样性：**
   - 如果你计划添加更多产品或品类，可能需要重新设计分类结构，以便于顾客寻找。

4. **用户体验：**
   - 分类应简单明了，方便顾客快速找到所需商品。如果发现某些分类下的产品较少，可以考虑合并或拆分。

5. **产品组合：**
   - 考虑将相关产品组合在一起，例如“喂养用品”可以包括猫粮、狗粮以及相关的喂食器具。

### 调整建议

基于上述考虑，你可以考虑以下的调整方向：

1. **合并小类目：**
   - 如果某些分类下的产品数量很少，可以将其合并。例如，“宠物美容沙龙”和“宠物健康中心”可以结合为“宠物护理中心”，涵盖洗护和健康产品。

2. **增设细分类别：**
   - 根据市场需求，可以增加一些特定细分的类别，如“幼犬/幼猫专区”或“老年宠物专用产品”。

3. **跨品类联动：**
   - 创建一些交叉分类，比如“节日装系列”可以涵盖所有类型的宠物服饰、玩具和零食，适合特殊场合的搭配。

4. **新兴趋势：**
   - 关注宠物行业的新趋势，增加相关产品线，如“环保宠物用品”或“可持续产品”。

5. **定期审查分类结构：**
   - 定期检查和分析各个分类的销售数据，根据销售情况进行动态调整，以保持商品分类的有效性和实用性。

### 示例调整后的商品分类

1. **🐱 喵星人专区** ✨
   - 猫粮、猫砂、猫玩具、猫树等。
   
2. **🐶 汪星人乐园** 🎠
   - 狗粮、狗零食、狗玩具、狗链等。

3. **💖 宠物护理中心** 🛁
   - 包含美容工具、洗护用品、保健品等。

4. **🍽️ 宠物美食街** 🥩
   - 宠物零食、营养补充品、自制食谱工具等。

5. **🌳 宠物探险队** 🎒
   - 户外用品、安全装备等。

6. **👗 宠物时装秀** 🎩
   - 宠物服装、配饰、节日特装等。

7. **📱 宠物科技馆** 🔭
   - 智能设备、监控器、定位器等。

### 结论

调整商品分类需要综合考虑市场需求、用户体验和产品种类。确保每个分类都能为顾客提供价值，并能够反映出你商店的独特卖点。如果你能做到这些，将大大提升消费者的购买体验和满意度。希望这些建议能帮助你优化你的网店！如需进一步讨论，欢迎随时联系。
```

## 


# 这样学习最轻松

## 制定

### 收益

- 提供学习成绩，获得更大进度。
- 可以帮助达成人生理想的阶梯。
- 

### 宜

- 可以着力去发展课本之外的能力，多参加一些社会实践。
- 将自己的计划安排紧张点，稍微到舒适区外围转转。
- 给自己定一个稍稍有些难度的目标，在平时学习安排上多增加些压力，这些目标和压力会帮助自己不断挖掘出自己的潜力，以获得进步。
- 可以制定一个长远计划，然后根据这个计划再分步骤的安排一些短期目标（在一步步的实行过程中，你会经常不断的获得一些令人愉快的小成果，这将不仅有助于树立你的自信心，更重要的是它会帮助你接近自己的理想，并最终使之得以实现）。
- 要有足够的耐心和勇气，做好打硬仗的准备。要有一股不服输的劲头。
- 要对自己有信息就一定能够获得成功。
- 根据自己的具体情况分配学习时间，不能”一刀切“。一般原则是，较薄弱的科目花费较多的时间，对于学的较好的科目要保证学习的质量，在此基础上合理分配学习时间。
- 要自信，不要怀疑自己的能力，能力不是天生的，而是锻炼出来的。其次随便给自己的能力定性其实就非常不合适。不要否定自己，不要触碰“天生不是学习的料”、“自己的学习能力不强”等否定自己的话。
- 认识自己的能力，一个人的能力并没有对决强弱之分，只不过在一定的阶段不同的人表现出不同的能力倾向。在制定计划时，要根据自己的能力倾向发扬自己较强的方面，同时在自己较弱的方面下功夫，要知道，只要肯下功夫任何能力都是可以提高的。
- 要学会如何管理时间。它很重要，它关系到执行计划的效率。不过可以先学习如何利用零散的时间。比如饭前饭后，上厕所期间，等车、乘车、约会等人的时间都不应该浪费。充分利用零散时间的关键是要养成随时思考的习惯。
比如等人时思考当天或前阵子学习过的内容；可以默念学过的内容，背的单词，公式，定理等。（集腋成裘，积沙成丘；不积跬步无以至千里，不积小流无以成江海）。好处是，充分利用零碎的时间可以通过化整为零的学习策略帮助你更为牢固的记忆学习内容，进一步提高学习效率。
- 要善于发现自己的生物节律并利用它，比如自己什么时间学习工作效率最高。因此在制定计划时要充分考虑学习与休息的关系。
- 科学的计划讲究有张有弛



### 忌

- 不主张超出自己的实际能力，盲目高攀。
- 不要一下子给自己定下太高的目标，超出自己能力的目标不仅不能开发潜能，反而会磋伤积极性。
- 指定计划时，不要将短期目标定的过高。可以将目标设定在使点劲就能拿到的位置。然后通过自己的女里获得进度，去享受收获的乐趣。这样能越学越有劲，在不知不觉中就获得较大的进步。
- 万万不可自暴自弃（华罗庚初中时数学还经常不及格），所以不要知难而退，而是要迎难而上。
- 当发现自己处于生物低潮期，不宜一味硬撑，适当休息或做一些轻松的活动，来帮助自己及时调整状态。
- 万不可借生物钟之名拖延你的学习计划，甚至把他当做偷懒的接口。

## 执行

执行靠什么？
对美好未来的向往、良好习惯、高度自觉、坚强的意志等。除此之外，还有能够灵活调整计划并对计划作有效及时的监督，也是顺利执行学习计划的关键。

### 宜

- 长远目标不宜经常变动，短期目标客根据实际情况调整
- 自己的事靠别人监督，是在是无奈之举，但有时监督确实能起到有效的作用。监督一定要公开化，一是找老师和长辈监督。另一种是和自己的好朋友互相监督，一同制定各自的任务，然后相互检查，凡是没有完成的一方就要受到另一方随心所欲的惩罚，想想，你的朋友一定在相近办法折磨没有完成任务的你，很有意思。

### 忌

### 注意

- 因调整而没有学习和复习的内容一定要在新的计划中补上，不能一跳了之。
- 在制定计划适应应吸取考虑欠周的教训，毕竟修改是要付出代价的。

## 范例

- 制定计划一定要有明确的目标。
- 长期计划与短期计划相结合。
- 长期远景式目标制定时要胸怀大志，统筹安排，目标合理——也就是目标既不能过高，也不能过低，要量力而行。
- 完善知识系统树，牵一发而动全身，做到由一个知识点可以拎起一串，提起以免。系统掌握知识后技巧也就水到渠成。
- 计划要注意小结，每周、月都要有小结，有问题及时发现，以免积重难返。
- 每一张的难点、重点，可以在考试系统复习前先自己复习上一遍，然后带着问题听老师的复习课。
- 复习计划要全面、周到、不要有的地方看的很细，有地地方走马观花、漏洞百出。
- 注意进度的安排，注意前紧后松，而不能前松后紧。因为随着日期的推移，人的疲劳感会逐渐加重，销毁会有所下降。在后面流出时间来有利于随机应变，从容不破、减少紧张、增强自信心。好处不言而喻。
- 把握好量，要给自己留有余地。要考虑自己的计划可行性。
- 了解自己的节奏

## 小结

在制定学习计划过程中，能够制定出有效的计划，关键在于是否能够客观的把握自己的情况。“认识你自己”认识世间最为重要也是最为困难的事情。


# 认识自己

要想真正了解、认识和评价自我，一方面要诉诸于科学正确的途径，另一方面要摆正心态，要有直面自我和揭露自我的勇气。当对自己有了全面、准确的认识时，就能更加从容地以积极自信的心态来应对自己的学习和未来生活。

- 认识自己需要不断对自己进行反思。
    A. 我的身体健康和心理健康情况如何？
    B. 我的理想和志向是什么？
    C. 我的个性和品德怎样？
    D. 我的智力和能力如何？是否很好的得到了发挥？
    E. 我的学习态度、努力程度和学习效果如何？
    F. 我的专长是什么？
    G. 我的兴趣爱好集中于哪些领域？
    H. 我的人际关系如何？

- 科学的方法认识自己
    A. 比较法。通过于同龄人在处世方法、对人对事的态度、情感表达方式等方面进行比较。找出自己的特点，来认识自己。
    B. 反省日记。自省是人的一种自我体验。通过自我反思、自我检查来认识自己。也可以从别人对自己的态度、看法中了解自己，因为每个人都是你的镜子。
    C. 自我谈话。可以在内心经常于自己进行一些交谈，给自己一些评价，对自己进行激励。这些通常会产生一些很有用的心理暗示作用。你将成为什么样的人，全由你自己做主。
    D. 从他人的评价中认识自己。要重视同伴对自己的评价，他人的额评价比主观自省具有更大的客观性。

- 在认识自我的过程中，应该采取辩证的态度，既要充分发现自己的长处、优点，同时也要认清自己的短处和不足。这样才能扬长避短、把握自己，取得更大进步。

“巴奴姆”效应，也就是星座、性格预测或是各种算术的秘密。

# 设计自己

我要成为怎样的人

- 自我训练。去尝试使用积极的词语，经常乐观愉快的鼓励自己和他人。
- 尝试去宽容一件事，去包容一个人；作决定之前试着把问题想的再长远一些、深刻一些。
- 运用增值晕连发，练习增加自己的价值，设想明天、后面、多年以后自身的价值定会比现在大的多。
- 把注意力放在重点上，不要被琐碎的小事所困扰。尝试学会放弃那些细枝末节的东西，你会发现问题原来并不复杂，自己解决问题的能力也因此而变强。
- 面对别人的批评、诽谤、表扬、恭维不动心。尝试不去在意别人对你的褒贬，这样既不会因为赞誉而得意忘形，也不会因为诽谤而畏缩不前。 

# 相信自己


# harbor

# 设置域名

1、编辑配置文件
```bash
# vim 进入配置文件
vim harbor.yml
```

修改 `hostname` 为我们自己的域名，例如：

```yaml
hostname: reg.youdomain.com
```

2、执行预制脚本
```bash
./ prepare
```

3、执行安装脚本
```bash
./install.sh
```

4、域名解析

5、docker 客户端将 `reg.youdomain.com` 设置到 `/etc/docker/daemon.json`，内容如下：
```json
{
    ...
    "insecure-registries": [
        "reg.yourdomain.com:10010"
    ]
}
```


curl -sS http://getcomposer.org/installer | php \
    && mv ./composer.phar /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer
    


curl -sS http://getcomposer.org/installer | php \
    && mv ./composer.phar /usr/local/bin/composer


    ns1.siteground.net

    ns2.siteground.net


    ns1.siteground.net

    ns2.siteground.net

wp.xuepingmall.com
pet.xuepingmall.com
book.xuepingmall.com
notes.einscat.com
einscat.com


> **🎉 结语 🎉   (๑´ㅂ`๑)**

这个站点是我与世界沟通的桥梁，也是我内心世界的一扇窗。欢迎你来到这里，与我一起探索、成长。

---

{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": [
    "harbor.einscat.com:10011"
  ],
  "registry-mirrors": [
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://registry.docker-cn.com",
    "https://vfs1y0dl.mirror.aliyuncs.com"
  ]
}



# 创建一个user, 用于管理存储对象

```bash
[root@VM-0-2-centos ~]# docker exec -it chaos-1 bash
[root@chaos-1 opt]# docker exec ceph-rgw radosgw-admin user create --uid=test_user --display-name=test_user --system
{
    "user_id": "test_user",
    "display_name": "test_user",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [],
    "keys": [
        {
            "user": "test_user",
            "access_key": "7G3KKVDWA0ES5R6EK823",
            "secret_key": "wtSt1l5dM7Pbm2z1Rpqzn3tkIXRPVQYHxCsel9qm"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "system": "true",
    "default_placement": "",
    "default_storage_class": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}

[root@chaos-1 opt]#
```

## AWS S3基本概念

- Region: 存储数据所在的地理区域
- Endpoint: 终端节点，存储服务入口，Web服务入口点的URL
- Bucket: 存储桶是S3中用于存储对象的容器
- Object: 对象是S3中存储的基本实体，又对象数据和元数据组成
- Key： 键是存储桶中对象的唯一标识符，桶内的每个对象都只能有一个key



# 

# centos8 


```bash
[root@1a47412ef4b0 /]# yum update -y
Failed to set locale, defaulting to C.UTF-8
CentOS Linux 8 - AppStream                                                                                                                                                                                                           0.0  B/s |   0  B     00:00
Errors during downloading metadata for repository 'appstream':
  - Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8&arch=aarch64&repo=AppStream&infra=container [Could not resolve host: mirrorlist.centos.org]
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8&arch=aarch64&repo=AppStream&infra=container [Could not resolve host: mirrorlist.centos.org]
[root@1a47412ef4b0 /]#
```




在 CentOS 8.4 容器中遇到的 "Couldn't resolve host name" 错误通常是由于 DNS 配置问题导致的。以下是一些步骤来解决该问题并初始化 CentOS 8 容器环境：

### 1. 检查网络连接

确认您能够从容器中访问外部网络。首先，尝试使用 `ping` 命令检查网络连接：

```bash
ping google.com
```

1. ping不通
```bash
[root@1a47412ef4b0 /]# ping baidu.com
PING baidu.com (39.156.66.10) 56(84) bytes of data.


```

如果无法 ping 通外部地址，可能是网络配置有问题。

### 2. 检查和修改 DNS 设置

如果存在 DNS 问题，请尝试以下方法：

#### a) 使用公共 DNS 服务器

编辑 `/etc/resolv.conf` 文件，将其内容更改为使用 Google 的公共 DNS 服务器：

```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

#### b) 添加其他 DNS 服务器

可以添加多个 DNS 服务器，例如：

```bash
echo -e "nameserver 8.8.8.8\nnameserver 8.8.4.4" >> /etc/resolv.conf
```

### 3. 清除缓存和更新 YUM

如果您已经修改了 DNS 设置，清除 YUM 缓存并重试更新命令：

```bash
yum clean all
yum update
```

### 4. 确保 yum 配置正确

确认 `/etc/yum.repos.d/` 下的 `.repo` 文件没有配置错误。确保 `baseurl` 或 `mirrorlist` 指向有效的地址。

### 5. 检查防火墙和代理设置

如果您在一个受限的网络环境中（如公司或学校），可能需要检查防火墙或代理设置，以确保容器能够访问外部网络。

### 6. 重新启动容器

有时重新启动容器可以解决临时的网络问题：

```bash
exit
# 重新启动您的容器
```

### 7. 使用带有网络功能的镜像

如果以上步骤仍然不能解决问题，考虑使用官方提供的 CentOS 8 镜像，并确保它具有网络功能。

完成这些步骤后，再次尝试运行 `yum update` 命令。如果一切顺利，您应该能够成功更新容器内的包管理器。


docker run --net host --privileged -itd centos:7 /usr/sbin/init

docker run -it --net host --privileged --name 888 centos:centos8.4.2105

docker run -it --name centos8 --privileged centos:centos8.4.2105
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

yum makecache



```bash
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

if [ ! -d $HOME/workspace ]; then
    mkdir -p $HOME/workspace
fi

# 用户特定环境
# Basic envs
export LANG="en_US.UTF-8"

export WORKSPACE="$HOME/workspace"
export PATH=$HOME/bin:$PATH

# 默认入口目录
cd $WORKSPACE
```

```bash
yum install -y make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel

```

```bash
cd /tmp
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.36.1.tar.gz
tar zxvf git-2.36.1.tar.gz
cd git-2.36.1
./configure
make
make install
git --version
```

```bash
tee -a $HOME/.bashrc <<'EOF'
# Configure for git
export PATH=/usr/local/libexec/git-core:$PATH
EOF

git config --global user.name "4343"
git config --global user.email "43@43.com"
git config --global credential.helper store
git config --global core.longpaths true
```

```bash
wget -P /tmp/ https://studygolang.com/dl/golang/go1.18.3.linux-arm64.tar.gz

wget -P /tmp/ https://studygolang.com/dl/golang/go1.23.2.linux-arm64.tar.gz
mkdir -p $HOME/go
tar zxvf /tmp/go1.23.2.linux-arm64.tar.gz -C $HOME/go
mv $HOME/go/go $HOME/go/go1.23.2
```

```bash
tee -a $HOME/.bashrc <<'EOF'
# Go envs
export GOVERSION=go1.18.3
export GO_INSTALL_DIR=$HOME/go
export GOROOT=$GO_INSTALL_DIR/$GOVERSION
export GOPATH=$WORKSPACE/golang
# 将 Go 语言自带的和通过go install安装的二进制文件加入PATH路径中
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
export GO111MODULE="on"
export GOPROXY=https://goproxy.cn,direct
export GOPRIVATE=
export GOSUMDB=off
EOF
```

```bash
cd /tmp
git clone -b v3.21.1 --depth=1 https://github.com/protocolbuffers/protobuf
cd protobuf
./autogen.sh
./configure
make
make install
protoc --version
```

```bash
go install github.com/golang/protobuf/protoc-gen-go@v1.5.2
```


```baah
mkdir -p $WORKSPACE/golang/src/github.com/marmotedu
cd $WORKSPACE/golang/src/github.com/marmotedu
git clone -b v1.6.2 --depth=1 https://github.com/marmotedu/iam
go work use ./iam

```bash
tee -a $HOME/.bashrc << 'EOF'
# Alias for quick access
export GOSRC="$WORKSPACE/golang/src"
export IAM_ROOT="$GOSRC/github.com/marmotedu/iam"
alias mm="cd $GOSRC/github.com/marmotedu"
alias i="cd $GOSRC/github.com/marmotedu/iam"
EOF
```

#
```bash
https://studygolang.com/dl/golang/go1.23.2.linux-arm64.tar.gz
```

```bash
export LINUX_PASSWORD='iam59!z$'
```

## mongodb

```bash
sudo tee /etc/yum.repos.d/mongodb-org-5.0.repo<<'EOF'
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/5.0/aarch64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-5.0.asc
EOF
```

```bash
sudo yum install -y mongodb-org
# 永久关闭SELINUX
sudo setenforce 0
sudo sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
# 开启外网访问权限和登录验证
sudo sed -i '/bindIp/{s/127.0.0.1/0.0.0.0/}' /etc/mongod.conf
sudo sed -i '/^#security/a\security:\n authorization: enabled' /etc/mongod.conf
# 启动
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
mongosh --quiet "mongodb://127.0.0.1:27017"
db.createUser({user: "root", pwd: "iam59!z$", roles: ["root"]})
db.auth("root", "iam59!z$")
```

```bash
[going@chaos-5 iam]$ mongosh --quiet mongodb://root:'iam59!z$'@127.0.0.1:27017/iam_analytics?authSource=admin
iam_analytics> db.createUser({user: "iam", pwd: "iam59!z$", roles: ["dbOwner"]})
{ ok: 1 }
iam_analytics> db.auth("iam", "iam59!z$")
{ ok: 1 }

mongosh --quiet mongodb://iam:'iam59!z$'@127.0.0.1:27017/iam_analytics?authSource=iam_analytics
```


```bash
[going@chaos-5 iam]$ cd $IAM_ROOT
[going@chaos-5 iam]$ mysql -h127.0.0.1 -P3306 -uroot -p'iam59!z$'
grant all on iam.* to iam@127.0.0.1 identified by 'iam59!z$';
flush privileges;

mysql -h127.0.0.1 -P3306 -uiam -p'iam59!z$'
source configs/iam.sql
show databases;
```

```bash
[going@chaos-5 iam]$ cd $IAM_ROOT
[going@chaos-5 iam]$ source scripts/install/environment.sh
[going@chaos-5 iam]$ sudo mkdir -p ${IAM_DATA_DIR}/{iam-apiserver,iam-authz-server,iam-pump,iam-watcher}
[going@chaos-5 iam]$ sudo mkdir -p ${IAM_INSTALL_DIR}/bin
[going@chaos-5 iam]$ sudo mkdir -p ${IAM_CONFIG_DIR}/cert
[going@chaos-5 iam]$ sudo mkdir -p ${IAM_LOG_DIR}
```

```bash
./scripts/install/install.sh iam::install::install_cfssl

cd $IAM_ROOT
tee ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "iam": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
EOF
```

参考：
https://github.com/marmotedu/iam/blob/v1.6.2/docs/guide/zh-CN/installation/06_%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AEIAM%E7%B3%BB%E7%BB%9F.md



各种连接-初级Go工程师训练营
课程代码仓库：极客时间-初级Go工程师训练营: 这是极客时间《初级Go工程师训练营》代码仓库，更多课程内容请移步：https://u.geekbang.org/subject/go3rd

安装多个 Go 版本：Managing Go installations - The Go Programming Language

rancher 起步：Getting Started with Kubernetes | Rancher Quick Start
压测工具 wrk: GitHub - wg/wrk: Modern HTTP benchmarking tool

etcd github 地址：GitHub - etcd-io/etcd: Distributed reliable key-value store for the most critical data of a distributed system 在 etcd-io 这个组织下还有很多和 etcd 有关的项目。


GIN 的插件库：GitHub - gin-gonic/contrib: Collection of middlewares created by the community
更加强的正则表达式库：GitHub - dlclark/regexp2: A full-featured regex engine in pure Go based on the .NET engine
zap 日志框架：uber-go/zap: Blazing fast, structured, leveled logging in Go. (github.com)
依赖注入 wire：google/wire: Compile-time Dependency Injection for Go (github.com)
cron 表达式开源仓库：GitHub - robfig/cron: a cron library for go
cron表达式教程:简单任务CRON表达式的详解_金融分布式架构 SOFAStack-阿里云帮助中心
gocron 仓库：GitHub - go-co-op/gocron: Easy and fluent Go cron scheduling. This is a fork from https://github.com/jasonlvhit/gocron
Kafka 命令行工具（shell脚本）各种说明：Kafka Command Line Interface (CLI) Tutorials | Learn Apache Kafka (conduktor.io)
docker compose 文件说明：Compose file version 3 reference | Docker Documentation
k6 安装：https://k6.io/docs/get-started/installation/ 

protobuf 类型映射：https://protobuf.dev/programming-guides/proto3/#scalar 
grpc 的 GO 文档：Go | gRPC

腾讯短信 API：短信 发送短信-发送短信相关接口-API 中心-腾讯云
我的开源库
支持泛型的工具库：GitHub - ecodeclub/ekit: 支持泛型的工具库
webook 后端开源仓库：GitHub - ecodeclub/webook: 站 


课程答疑文档
https://doc.weixin.qq.com/doc/w3_ANUAjgZQAKokJnFivsERDqpCs03n2?scode=ACQADQdwAAoIdZZbFH 


# 面试

- 什么是闭包？闭包有什么缺陷

闭包就是有一个方法，这个方法使用外部传入了一个内部定义的局部变量。
缺陷会加剧内存逃逸，什么内存逃逸？在Go里面创建一个对象的时候有时候分配到栈上，有时候分配到堆上，如果分配到了堆上就叫做内存逃逸，分配到堆上会比较慢，比较耗性能，它还需要垃圾回收，所以说内存逃逸是一个比较严重的问题。
还有写认为闭包可读性比较差，但是我觉的还好。

- 什么情况下会出现栈溢出？
无限循环，stack overflow

- 什么是不定参数？调用方法的时候，不定参数可以传入0个值吗？方法内部怎么使用不定参数？
就是当切片来用

- 什么是 defer？你能解释一下 defer 的运作机制吗？
defer 是指在程序返回前执行一段代码。
defer 面试比较常见，可以深入介绍内部的运作机制。

内部分成3种机制：

* 堆上分配，是指整个 defer 直接分配在堆上，缺点是要被GC管理
* 栈上分配，是值整个 defer分配到了 goroutine 栈上，不需要被GC管理。比堆上分配性能提升了30%
* 开放编码（Open Code），是指启用内联的优化，直观理解就是相当于把你的 defer 内容放到了你的函数最后，启用条件：
 1. 函数的 defer 数量少于或等于8个
 2. 函数的 defer 关键字不能在循环中执行
 3. 函数的 return 语句与 defer 语句的乘积小于或等于15个

如何理解启用条件？
 1. 函数的 defer 数量少于或等于8个，原因是用了一个 byte 来记录哪些 defer 要指定
 2. 函数的 defer 关键字不能在循环中执行，原因是编译的时候不知道有多少个 defer
 3. 函数的 return 语言与 defer 语句的乘积小于或等于15，这个是一个硬性的规定。

defer 的作用？你用 defer 解决过什么问题？defer 就是函数返回前执行一些动作，比如释放资源、关闭连接等


- 一个方法内部 defer能不能超过8个？
可以，但是超过8个会更换一种底层实现方式

- defer 内部能不能修改返回值？怎么改？
可以，但前提是返回值是带有名字的返回值。如果是局部变量声明的返回值就不可以改

- 数组和切片有什么区别？

- 切片怎么扩容？
重新分配一段连续的内存，然后把原本的数据拷贝过去。
面试中，亮点的地方可以解释切片扩容的系数。
总结为：
* 当容量小于256的时候，两倍扩容
* 否则按照1.25扩容，低版本是1024作为分界点

强调几个点：
* 为什么一开始是两倍扩容，后面是1.25倍扩容
  扩多了浪费内存，扩少了就会频发触发扩容。需要做一个权衡
* 为什么低版本是1024作为分界点，而高版本是256
* 还可能会为为什么切片没有缩容，如果你已经实现了泛型工具库，可以说提供了切片的删除辅助方法，而后引入了缩容机制。

- map 的底层原理

- channel 的底层原理




##
设计一个简洁温馨的logo，包含可爱的狗狗和猫咪图案，使用柔和的浅蓝色和橙色，传达友好和专业的品牌形象，可以加入心形或爪印元素，字体圆润易读。

设计一个温馨而简洁的Logo，一只开心摇尾巴的黑色狗狗；一只微笑着、怀抱小熊猫的白色猫咪。使用柔和的浅蓝色和橙色，以传达友好与专业的品牌形象。可以加入心形或爪印元素，搭配圆润易读的字体，营造愉悦与舒适的视觉体验。

- 7日紧锁异味艾禾美铁锤猫砂（红标）多猫家庭
- 艾禾美铁锤猫砂12.7kg红标天然钠基膨润土矿祛味除臭无粉尘28磅
- ArmHanmmer艾禾美猫砂膨润土矿石除臭黑盒铁锤猫砂12.7kg
- 美国进口铁锤猫砂28磅到手175元
- 艾禾美进口膨润土猫砂28磅
- 铁锤猫砂红标28磅-160
- 铁锤猫砂蓝标28磅-170
- 铁锤猫砂绿标28磅-180

## 艾禾美铁锤猫砂导购关键词：

**核心卖点:**

* 7天长效除臭 (7-day odor control, long-lasting freshness)
* 天然钠基膨润土 (natural sodium bentonite)
* 无尘结团 (dust-free clumping)
* 多猫家庭首选 (ideal for multi-cat households)
* 美国进口 (American imported)

**其他关键词:**

* 高效吸水 (super absorbent)
* 快速结团 (quick clumping)
* 减少异味 (odor elimination)
* 方便清理 (easy to clean)
* 环保可降解 (environmentally friendly, biodegradable)
* 价格优惠 (great deals, limited time offer)
* 猫砂盆伴侣 (litter box enhancer)

**适用于不同平台的格式:**

**电商平台:**

* 7天长效锁味，艾禾美铁锤红标猫砂，多猫家庭优选！
* 天然钠基，无尘结团，艾禾美铁锤猫砂28磅，限时抢购！

**社交媒体:**

* 🐱你的猫咪值得更好的！7天长效除臭，艾禾美红标铁锤猫砂，多猫家庭必备！#猫砂 #除臭 #宠物用品
* 🔥限时抢购！美国进口艾禾美铁锤猫砂28磅，天然钠基，无尘结团，高效除臭！#宠物 #优惠 #猫咪

**搜索引擎优化:**

* 美国进口艾禾美铁锤猫砂
* 天然钠基除臭猫砂
* 长效锁味猫砂
* 多猫家庭猫砂推荐
* 艾禾美猫砂优惠

希望以上关键词能帮助你更好地推广艾禾美铁锤猫砂！

https://zh-cn.ubisoft.com/company/career/310944594
https://www.coupang.jobs/en/jobs/5360422/director-back-end-engineering/?gh_jid=5360422


{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "insecure-registries": [
    "harbor.einscat.com:10011"
  ],
  "registry-mirrors": [
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://registry.docker-cn.com",
    "https://vfs1y0dl.mirror.aliyuncs.com"
  ]
}

## 安装 beego

### 安装beego与bee工具

```
cd /Users/finnley/go 
go get github.com/astaxie/beego
go get github.com/beego/bee
```

### 查看版本

```
$ bee version
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.11.0

├── Beego     : Beego is not installed. Please do consider installing it first: https://github.com/astaxie/beego
├── GoVersion : go1.15.4
├── GOOS      : darwin
├── GOARCH    : amd64
├── NumCPU    : 16
├── GOPATH    : /Users/finnley/go
├── GOROOT    : /usr/local/go
├── Compiler  : gc
└── Date      : Saturday, 7 Nov 2020
➜  go
```

### 测试

创建新的beego项目：
```
bee new {project name}
```

创建api项目，不会创建static和views目录：
```
bee api {api peoject name}
```

运行

```
bee run
```

如果出现下面的错误

```
beego-demo bee run
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.12.0
2021/04/15 07:58:39 WARN     ▶ 0001 Running application outside of GOPATH
2021/04/15 07:58:39 INFO     ▶ 0002 Using 'beego-demo' as 'appname'
2021/04/15 07:58:39 INFO     ▶ 0003 Initializing watcher...
go: github.com/astaxie/beego@v1.12.1: missing go.sum entry; to add it:
	go mod download github.com/astaxie/beego
2021/04/15 07:58:39 ERROR    ▶ 0004 Failed to build the application: go: github.com/astaxie/beego@v1.12.1: missing go.sum entry; to add it:
	go mod download github.com/astaxie/beego
```

可以在根目录下输入 `go get github.com/astaxie/beego` ，然后再次运行

编译打包：
```
bee pack
```

注意，mac下打包的文件在linux是不能使用的，打包时需要指定参数：
```
bee pack -be GOOS=linux
```

升级：
```
go get -u github.com/astaxie/beego
go get -u github.com/beego/bee
```

创建api项目：
```
bee api go-video-api
```

使用环境变量的方式设置 `GO111MODULE=off`:
```
export GO111MODULE=off
echo $GO111MODULE
```

```
go build ./...
```

或者 

```
go mod init [name]
go build ./...
```

## GOPATH

```
mkdir /tmp/gopathtest
cd /tmp/gopathtest
```

```
go env
➜  gopathtest go env
GO111MODULE="off"
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/finnley/Library/Caches/go-build"
GOENV="/Users/finnley/Library/Application Support/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
GOMODCACHE="/Users/finnley/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="darwin"
GOPATH="/Users/finnley/go"
GOPRIVATE=""
GOPROXY="https://goproxy.cn,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
AR="ar"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/1c/tfxvqn6d5zqdyn0ghyrv8vq40000gn/T/go-build264020906=/tmp/go-build -gno-record-gcc-switches -fno-common"
➜  gopathtest
```

当前 GOPATH 目录是 `GOPATH="/Users/finnley/go"`

可以通过下面命令将 GOPATH 设置成 `/tmp/gopathtest`

```
go env -w {GO PATH}
```

但是这样写会将系统的整个GOPATH替换调，所以不建议这样使用

可以使用 `export` 方法

```
export GOPATH=/tmp/gopathtest
```

GOPATH 还有个要求，就是必须在该目录下新建一个 `src` 目录

```
➜  gopathtest pwd
/tmp/gopathtest
➜  gopathtest export GOPATH=/tmp/gopathtest
➜  gopathtest ls
➜  gopathtest mkdir src
➜  gopathtest
```

另外需要关闭GO111MODUKE=off

```
export GO111MODULE=off
```

## go mod

清理

```
go mod tidy
```

旧项目迁移 go mod

```
go build ./...
```

或者

```
go mode init {name}
go build ./...
```

### Shell

```
# install python3
sudo yum install -y openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel gcc gcc-c++  openssl-devel libffi-devel python-devel mariadb-devel
sudo wget https://www.python.org/ftp/python/3.9.2/Python-3.9.2.tgz
sudo tar -zxvf Python-3.9.2.tgz -C /tmp
cd /tmp/Python-3.9.2/
sudo ./configure --prefix=/usr/local
# sudo yum -y install gcc gcc-c++ automake autoconf libtool make
sudo make
sudo make altinstall
sudo ln -s /usr/local/bin/python3.9 /usr/bin/python3
sudo ln -s /usr/local/bin/pip3.9 /usr/bin/pip3

# install golang
export GOROOT=/home/vagrant/go
export GOPATH=/home/vagrant/Code/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
cd ~
sudo wget https://studygolang.com/dl/golang/go1.16.linux-amd64.tar.gz
sudo tar -xvf go1.16.linux-amd64.tar.gz
sudo echo 'export GOROOT=/home/vagrant/go' >> ~/.bashrc
sudo echo 'export GOPATH=/home/vagrant/Code/go' >> ~/.bashrc
sudo echo 'export PATH=$PATH:$GOROOT/bin:$GOPATH/bin' >> ~/.bashrc
source ~/.bashrc

go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct

# install protobuf
# install grpc
# python3 -m pip install grpcio -i https://pypi.douban.com/simple
# install grpc tools
# python3 -m pip install grpcio-tools -i https://pypi.douban.com/simple

# install protobuf
sudo wget https://github.com/protocolbuffers/protobuf/releases/download/v3.15.3/protobuf-all-3.15.3.tar.gz
sudo tar -zxvf protobuf-all-3.15.3.tar.gz
cd protobuf-3.15.3
sudo ./configure --prefix=/usr/local/protobuf
sudo make
sudo make install
sudo echo 'export PROTOBUF=/usr/local/protobuf' >> ~/.bash_profile
sudo echo 'export PATH=$PROTOBUF/bin:$PATH' >> ~/.bash_profile
source ~/.bash_profile
cd ~
```

## 开发环境配置

```shell
## 创建普通用户
# useradd going # 创建going用户，通过going用户登录开发机进行开发
# passwd going # 设置going用户登录密码

## 添加普通用户到sudoers，以便普通用户可以通过sudo命令来暂时获取root权限
# sed -i '/^root.*ALL=(ALL).*ALL/a\going\tALL==(ALL) \tALL' /etc/sudoers

## 配置 $HOME/.bashrc
# .bashrc
# 用户特定的别名和函数
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# 源全局定义
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

if [ ! -d $HOME/workspace ]; then
	mkdir -p $HOME/workspace
fi

# 用户特定环境
# BASIC envs
export LANG="en_US>UTF-8"
export PS1='[\u@dev \W]\$ '

export WORKSPACE="$HOME/workspace"
export PATH=$HOME/bin:$PATH

# 默认入口目录
cd $WORKSPACE
```

## 
8、另外还需要下载 `Go` 的依赖包

```go
go get github.com/golang/protobuf/protoc-gen-go
```

9、命令
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


---

docker run --rm --privileged -it --name my-centos --tmpfs /run --tmpfs /run/lock -v /sys/fs/cgroup:/sys/fs/cgroup:ro harbor.einscat.com:10011/library/centos8:aarch64 /usr/sbin/init


docker run --rm -v /tmp/:/tmp --privileged --cap-add SYS_ADMIN -e container=docker -it --name=centos -d harbor.einscat.com:10011/library/centos8:aarch64 /usr/sbin/init
