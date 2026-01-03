+++
title = 'TDD测试驱动开发'
date = 2025-11-16T22:04:31+08:00
draft = true
categories = [ "Programming" ]
tags = [ "programming", "go" ]
+++

# 1 测试驱动开发

![](/images/programming/TDD/image-26.png)

在掌握了撰写单元测试和集成测试之后，我们现在从

测试开始出发，也就是尝试 TDD。

TDD：测试驱动开发。大明简洁版定义：先写测试、再写实现。
- 通过撰写测试，理清楚接口该如何定义，体会用户使用起来是否合适。
- 通过撰写测试用例，理清楚整个功能要考虑的主流程、异常流程。

TDD 专注于某个功能的实现。
PS：非正统理论，正统理论太难用。

# 2 测试套件

## 框架

```go
type ArticleTestSuite struct {
	suite.Suite
}

func (s *ArticleTestSuite) TestABC() {
	s.T().Log("hello，这是测试套件")
}

func TestArticle(t *testing.T) {
	suite.Run(t, &ArticleTestSuite{})
}
```

上面代码叫测试套件，是单元测试的一种组织方式，运行 TestArticle 可运行全部的测试套件，运行类似 TestABC 的方法即可运行独立的测试方法。

为什么需要测试套件呢？因为要在这里为好几个接口写集成测试，有些公共的部分想复用，就可以使用测试套件来服用，非常方便。

# 3 定义接口

## 3.1 需求：新文章

根据我们的需求分析，想到前端的输入应该就是两个字段：

• 标题。
• 内容：也就是帖子的主要内容。

后续需求可能会有别的字段，但是目前来说就是这两个了。

## 3.2 HTTP 接口

启用一个新的 ArticleHandler，并且注册第一个路由。

```go
func (h *ArticleHandler) RegisterRoutes(server *gin.Engine) {
	g := server.Group("/articles")
	{
		g.POST("/edit", h.Edit)
	}
}
```

## 3.3 测试用例结构体

在这里，我们遵循之前测试的规范，定义了测试用例的结构体。其中：
- Article 是为了方便我们构造请求引入的。
- Result[int64] 是一个泛型声明，为了简化对比响应的代码而引入的。

所以我们可以先把结构体定义出来：
```go
...

func TestArticle(t *testing.T) {
	suite.Run(t, &ArticleTestSuite{})
}

type Article struct {
	Id      int64
	Title   string `json:"title"`
	Content string `json:"content"`
}
```

这实际也是预期中的输入，故：
```go
func (s *ArticleTestSuite) TestEdit() {
	t := s.T()
	testCases := []struct {
		name string

		// 预期中的输入
		art Article
	}{
		
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			
		})
	}
}
...

func TestArticle(t *testing.T) {
	suite.Run(t, &ArticleTestSuite{})
}
```

预期中的输出是什么呢？我们希望返回的输出就是一个Result，故：
```go
func (s *ArticleTestSuite) TestEdit() {
	t := s.T()
	testCases := []struct {
		name string

		// 预期中的输入
		art Article
        // HTTP 响应码
		wantCode int
		// 我希望 HTTP 响应，带上帖子的 ID
		wantRes Result[int64]
	}{
		
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			
		})
	}
}
...

func TestArticle(t *testing.T) {
	suite.Run(t, &ArticleTestSuite{})
}

type Article struct {
	Id      int64
	Title   string `json:"title"`
	Content string `json:"content"`
}

type Result[T any] struct {
	Code int    `json:"code"`
	Msg  string `json:"msg"`
	Data T      `json:"data"`
}
```

集成测试还需要考了开始的数据准备与数据的验证：
```go
func (s *ArticleTestSuite) TestEdit() {
	t := s.T()
	testCases := []struct {
		name string

        // 集成测试准备数据
		before func(t *testing.T)
		// 集成测试验证数据
		after func(t *testing.T)

		// 预期中的输入
		art Article
        // HTTP 响应码
		wantCode int
		// 我希望 HTTP 响应，带上帖子的 ID
		wantRes Result[int64]
	}{
		
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			
		})
	}
}
```

## 3.4 执行测试用例的代码

```go
type ArticleTestSuite struct {
	suite.Suite
	server *gin.Engine
	db     *gorm.DB
}

...

func (s *ArticleTestSuite) TestEdit() {
	t := s.T()
	testCases := []struct {
		name string

		// 集成测试准备数据
		before func(t *testing.T)
		// 集成测试验证数据
		after func(t *testing.T)

		// 预期中的输入
		art Article

		// HTTP 响应码
		wantCode int
		// 我希望 HTTP 响应，带上帖子的 ID
		wantRes Result[int64]
	}{
		
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			// 构造请求
			// 执行
			// 验证结果

			tc.before(t)
			reqBody, err := json.Marshal(tc.art)
			assert.NoError(t, err)
			req, err := http.NewRequest(http.MethodPost,
				"/articles/edit", bytes.NewBuffer(reqBody))
			require.NoError(t, err)
			// 数据是 JSON 格式
			req.Header.Set("Content-Type", "application/json")
			// 这里你就可以继续使用 req

			resp := httptest.NewRecorder()
			// 这就是 HTTP 请求进去 GIN 框架的入口。
			// 当你这样调用的时候，GIN 就会处理这个请求
			// 响应写回到 resp 里
			s.server.ServeHTTP(resp, req)

			assert.Equal(t, tc.wantCode, resp.Code)
			if resp.Code != 200 {
				return
			}
			var webRes Result[int64]
			err = json.NewDecoder(resp.Body).Decode(&webRes)
			require.NoError(t, err)
			assert.Equal(t, tc.wantRes, webRes)
			tc.after(t)
		})
	}
}
```

由于可能不止一个地方会使用到server，所以将其放入结构体中，现在是如何初始化server呢？
```go
type ArticleTestSuite struct {
	suite.Suite
	server *gin.Engine
	db     *gorm.DB
}

func (s *ArticleTestSuite) SetupSuite() {
	s.server = gin.Default()
	s.server.Use(func(ctx *gin.Context) {
		ctx.Set("claims", &ijwt.UserClaims{
			Uid: 123,
		})
	})
	s.db = startup.InitTestDB()
	artHdl := startup.InitArticleHandler()
	// 注册好了路由
	artHdl.RegisterRoutes(s.server)
}
```

可以通过实现 SetupSuite() 这个方法来初始化一些前置操作，在所有测试执行之前，初始化一些内容。

## 3.5 接口实现：新建帖子

第一个测试用例：新建一个帖子，并且保存成功。对于 before 来说，啥也不用做。对于 after 来说，必须确保已经插入了数据，并且，还要仔细断言插入的数据。

注意这个时候我们已经定义了一个最简单的数据库模型。

PS：在 TDD 之下，整个过程都是迭代的，所以你不需要担心表结构会不会发生变更，接口有没有变化等问题。