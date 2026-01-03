+++
title = '第九章 JWT'
date = 2025-11-04T08:47:41+08:00
draft = true
categories = [ "Programming" ]
tags = [ "go", "programming"]
+++

# 1 JWT简介

前面我们实现的登录功能，是直接依赖于 Gin 的Session 插件达成的。

我们还可以考虑使用 JWT 来实现登录功能。JWT（JSON Web Token）是很常用的一种机制，主要用于身份认证，也就是登录。它的基本原理就是通过加密生成一个 token，而后客户端每次访问的时候都带上这个 token。

![](/images/programming/90.png)

它由三部分组成：
- Header：头部，JWT 的元数据，也就是描述这个token 本身的数据，一个 JSON 对象。
- Payload：负载，数据内容，一个 JSON 对象。
- Signature：签名，根据 header 和 token 生成。签名用来验证前两个有没有被篡改

JWT原始API：
https://github.com/golang-jwt/jwt/

# 2 JWT使用 

在登录过程中，使用 JWT 也是两步：
- JWT 加密和解密数据。
- 登录校验。

# 3 代码

**登录**

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

**校验**

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

# 4 使用 JWT 的优缺点

和 Session 比起来

优点：
- 不依赖于第三方存储。
- 适合在分布式环境下使用。
- 提高性能（因为没有 Redis 访问之类的）。

缺点：
- 对加密依赖非常大，比 Session 容易泄密。
- 最好不要在 JWT 里面放置敏感信息。

# 5 混用 JWT 和 Session 机制

前面 JWT 限制了我们不能使用敏感数据，那么你真有类似需求的时候，就可以考虑将数据放在“Session”里面。

基本的思路就是：你在 JWT 里面存储你的 userId，然后用 userId 来组成 key，比如说 user.info:123 这种 key，然后用这个 key 去 Redis 里面取数据，也可以考虑使用本地缓存数据。