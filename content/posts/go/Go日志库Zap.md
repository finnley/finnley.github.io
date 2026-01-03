+++
title = 'Go日志库Zap'
date = 2025-03-10T10:07:39+08:00
draft = true
categories = [ "Go" ]
tags = [ "go", "golang", "环境搭建" ]
+++

# 1 安装

```shell
go get -u go.uber.org/zap
```

参考：[uber-go/zap](https://github.com/uber-go/zap)

# 2 使用

```go
logger, _ := zap.NewProduction()
defer logger.Sync() // flushes buffer, if any
sugar := logger.Sugar()
sugar.Infow("failed to fetch URL",
  // Structured context as loosely typed key-value pairs.
  "url", url,
  "attempt", 3,
  "backoff", time.Second,
)
sugar.Infof("Failed to fetch URL: %s", url)
```

Zap提供了两种类型的日志记录器:
- Sugared Logger
- Logger。

在性能很好但不是很关键的上下文中，使用SugaredLogger。它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。

在每一微秒和每一次内存分配都很重要的上下文中，使用Logger。它甚至比SugaredLogger更快，内存分配次数也更少，但它只支持强类型的结构化日志记录

总结：
- 图方便使用 Sugared Logger，但性能不如 Logger 。
- 要求性能使用 Logger，但它不如Sugared Logger方便。

Logger 使用如下：
```go
func TestLogger(t *testing.T) {
	// 2. Logger 性能高
	logger, _ := zap.NewProduction()
	defer logger.Sync() // flushes buffer, if any

	url := "https://notes.einscat.com"
	logger.Info("failed to fetch URL",
		zap.String("url", url),
		zap.Int("nums", 3),
	)
}
```
使用起来麻烦，但性能高，不会使用到反射机制，使用 Sugared Logger 会使用到反射机制。

# 3 输出日志到文件

```go
func NewLogger() (*zap.Logger, error) {
	cfg := zap.NewProductionConfig()
	cfg.OutputPaths = []string{
		"./myproject.log",
		"stderr",
		"stdout",
	}
	return cfg.Build()
}

func TestLog2File(t *testing.T) {
	logger, err := NewLogger()
	if err != nil {
		panic(err)
	}
	su := logger.Sugar()
	defer su.Sync()

	url := "https://notes.einscat.com"
	su.Info("failed to fetch URL",
		zap.String("url", url),
		zap.Int("attempt", 3),
		zap.Duration("backoff", time.Second),
	)
}
```
