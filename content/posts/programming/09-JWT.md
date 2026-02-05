+++
title = '第九章 JWT'
date = 2025-11-04T08:47:41+08:00
draft = true
categories = [ "Programming" ]
tags = [ "go" ]
+++

## 1 简介

前面我们实现的登录功能，是直接依赖于 Gin 的Session 插件达成的。

我们还可以考虑使用 JWT 来实现登录功能。JWT（JSON Web Token）是很常用的一种机制，主要用于身份认证，也就是登录。它的基本原理就是通过加密生成一个 token，而后客户端每次访问的时候都带上这个 token。

![](/images/programming/90.png)

它由三部分组成：
- Header：头部，JWT 的元数据，也就是描述这个token 本身的数据，一个 JSON 对象。
- Payload：负载，数据内容，一个 JSON 对象。
- Signature：签名，根据 header 和 token 生成。签名用来验证前两个有没有被篡改

JWT原始API：
https://github.com/golang-jwt/jwt/

### 1.1 优缺点

和 Session 相比，优缺点如下：

优点：
- 不依赖于第三方存储。
- 适合在分布式环境下使用。
- 提高性能（因为没有 Redis 访问之类的）。

缺点：
- 对加密依赖非常大，比 Session 容易泄密。
- 最好不要在 JWT 里面放置敏感信息。

## 2 使用 

在登录过程中，使用JWT与使用Cookie、Session方式一样分为两步，：
- JWT 加密和解密数据（登录时放数据）。
- 登录校验（校验时验证数据）。

## 3 代码

### 3.1 登录

```go
func (u *UserHandler) Login(ctx *gin.Context) {
	type LoginReq struct {
		Email    string `json:"email"`
		Password string `json:"password"`
	}

	var req LoginReq
	if err := ctx.Bind(&req); err != nil {
		return
	}

	user, err := u.svc.Login(ctx, req.Email, req.Password)
	if err == service.ErrInvalidUserOrPassword {
		ctx.String(http.StatusOK, "邮箱或密码错误")
		return
	}
	if err != nil {
		ctx.String(http.StatusOK, "系统错误")
		return
	}

	// 使用JWT设置登录态
	{
		claims := UserClaims{
			Uid: user.Id,
		}
		token := jwt.NewWithClaims(jwt.SigningMethodHS512, claims)
		tokenStr, err := token.SignedString([]byte(config.JwtSignKey))
		if err != nil {
			ctx.String(http.StatusInternalServerError, "系统错误")
			return
		}
		// token 放在 Header 中返回给前端
		ctx.Header("x-jwt-token", tokenStr)

		fmt.Println("tokenStr: ", tokenStr)
		fmt.Println("user: ", user)
	}

	ctx.String(http.StatusOK, "登录成功")
}
```

### 3.2 校验

```go
func (l *LoginJWTMiddlewareBuilder) Build() gin.HandlerFunc {
	// 用go的方式编码解码为二进制
	gob.Register(time.Now())

	return func(ctx *gin.Context) {
		// 不需要校验的接口
		for _, path := range l.paths {
			if ctx.Request.URL.Path == path {
				return
			}
		}

		// 登录校验
		{
			tokenHeader := ctx.GetHeader("Authorization")
			// 未携带表示未登录
			if tokenHeader == "" {
				ctx.AbortWithStatus(http.StatusUnauthorized)
				return
			}

			// Bearer xx
			segs := strings.Split(tokenHeader, " ")
			if len(segs) != 2 {
				// 没登录，有人瞎搞，乱传
				ctx.AbortWithStatus(http.StatusUnauthorized)
				return
			}

			tokenStr := segs[1]
			// 这里已经将 claims 拿到了
			claims := &web.UserClaims{}
			token, err := jwt.ParseWithClaims(tokenStr, claims, func(token *jwt.Token) (interface{}, error) {
				return []byte(config.JwtSignKey), nil
			})
			if err != nil {
				// 没登录
				ctx.AbortWithStatus(http.StatusUnauthorized)
				return
			}
			// err 为 nil，token 肯定不为 nil
			// 最后一个条件可加可不加
			if token == nil || !token.Valid || claims.Uid == 0 {
				// 没登录
				ctx.AbortWithStatus(http.StatusUnauthorized)
				return
			}

			// 这里为了方便在接口中获取用户信息，无需重复解析token
			ctx.Set("claims", claims)
			//ctx.Set("userId", claims.Uid)
		}
	}
}

```

### 3.3 小结

**步骤**

1、首先在Login接口中，登录成功后生成JWT Token。

- 在JWT Token中写入所需的数据。
- 把JWT Token写入HTTP Responder Header x-jwt-token中返回给前端。

2、改造跨域中间件，允许前端访问 x-jwt-token。

3、之后前端的所有访问，除去特定接口，之后前端访问的所有接口都会携带这个Token。

4、接入JWT登录校验的Middleware。

- 读取JWT Token
- 验证JWT Token是否合法。

![](/images/programming/jwt/10.png)

## 4 JWT Token携带数据

## 5 混用JWT和Session

JWT限制了我们不能使用敏感数据，那么你真有类似需求的时候，就可以考虑将数据放在“Session”里面。

基本的思路：在JWT里面存储你的 userId，然后用 userId 来组成 key，比如说 user.info:123 这种 key，然后用这个 key 去 Redis 里面取数据，也可以考虑使用本地缓存数据。

误区：用了JWT就不能使用Session，用了Session就不能使用JET。

## 6 保护系统

在功能完成之后，现在要进一步考虑保护我们的系统。一般来说，你要考虑两方面的事情：

- 正常用户会不会搞崩你的系统？
- 如果有人攻击你的系统，系统能撑住吗？

对于中小型公司来说第一条不会是问题，对于大公司来说两条都考虑。

### 6.1 系统可能存在的漏洞

现在我们的系统最明显的漏洞就是：
- 任何人都可以注册。
- 任何人都可以登录。

![](/images/programming/security/10.png)

也就是说，万一有一个人用shell脚本拼命给你发注册请求、登录请求，系统负载就会很高，而且这两个请求都会查询数据库，也就是说数据库负载也很高。
注册和登录是两个非常关键的功能，注册会影响新用户接入，登录会影响已有的用户，所以一定要保护好它们。这两个也都会涉及到数据库，恶意使用使用会沿着链路将数据库打爆，如果数据库还提供给其他业务使用，其他业务也会受到影响。

### 6.2 解决方案——限流

这个时候，我们可以考虑，能不能限制住每个人发送的请求数量？又或者限制住系统处理的请求数量？这就是限流。
![](/images/programming/security/20.png)

限流是最常见的保护系统的办法。限流有很多算法，但是都大同小异。这里使用的最简单的方式——限制每一个用户每秒最多发送固定数量的请求。
为什么这么设计？因为我们不是攻击者，我们作为普通人使用登录注册都不会很频繁，正常人通常只注册一次，几天登录一次，所以正常用户都不会频繁发送请求，所以才如此设计——限制每一个用户每秒最多发送固定数量的请求。

**1、思考**

这一设计会引起一些问题的思考：

- 我怎么认定谁是谁？尤其是在登录和注册这个接口里，用户都还没登录成功，我都不知道他是谁，这一问题表示我们该用什么来标识用户到底是否是恶意用户。
- 即便识别出用户，我怎么确定我限流的阈值应该是多少？每秒100还是每秒200个请求？

**2、限流对象**

![](/images/programming/security/30.png)

第一个问题的答案是：用 IP。也就是说我们的限流针对的是 IP。IP 虽然并不能实际意义上代表一个人，但这已经是我们比较好的选择了。

更好的选择是用 MAC 地址或者设备标识符之类的，比如说 CPU 序列号，但是在 Web 端很少用。

APP 端就可以考虑用设备序列号。当然，在使用 IP 的情况下，我们可能会误把不同的人看成是同一个人。但是只要我们限制的阈值不是很小，就不会有问题。

## 限流阈值

![](/images/programming/130.png)

限流阈值应该是多少？

理论上来说，这应该是通过压测来得到的（面试回答这个）。比如说你压测整个系统，发现最多只能撑住每秒 1000 个请求，那么阈值就是 1000。
而我们是针对个人，搞不了压测。所以可以凭借经验来设置，比如说我们正常人手速，一秒钟撑死一个请求，那么就算我们考虑到共享 IP 之类的问题，给个每秒 100 也已经足够了。

## 被限流的请求怎么办？

![](/images/programming/140.png)

如果我每秒处理 100 个请求，那第 101 个请求过来怎么办？

显然，只能拒绝了，也就是返回错误。这个错误，不同公司有不同的规范。如果你自己决策的话，可以返回什么服务器繁忙之类的信息。

## 参考

https://learnku.com/articles/17883?order_by=vote_count&