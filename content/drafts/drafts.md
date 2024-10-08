# 2024-08-16 09:43:28 Fri

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