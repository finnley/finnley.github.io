+++
title = 'go依赖发布'
date = 2026-01-16T10:11:05+08:00
draft = true
categories = [ "Go" ]
tags = [ "go", "golang"]
+++

在 Go 语言中，**“发布”其实就是打标签（Tag）**，而**“使用”就是 `go get`**。Go Modules 依赖 Git 的标签来进行版本管理。

以下是完整的发布与使用流程，分为 **公开仓库** 和 **私有仓库** 两种情况（通常公司的通用库都是私有的）。

---

### 第一阶段：发布 (在 `ab-go` 仓库操作)

#### 1. 确认 `go.mod` 模块名

首先确保你的 `ab-go` 根目录下的 `go.mod` 文件的模块名（module name）与你的 GitHub 地址完全一致。

打开 `ab-go/go.mod`，它应该长这样：

```go
// 必须与 GitHub 地址一致，否则别人拉不下来
module github.com/your-username/ab-go 

go 1.20

```

#### 2. 提交代码并推送

确保所有代码都已提交并推送到 `main` 分支。

```bash
git add .
git commit -m "feat: add ginx/validator"
git push origin main

```

#### 3. 打标签 (发布版本)

Go Modules 遵循语义化版本 (SemVer)。我们需要打一个 `v` 开头的标签。

```bash
# 打上 v0.0.1 标签
git tag v0.0.1

# 将标签推送到 GitHub
git push origin v0.0.1

```

*此时，你的库已经在 GitHub 上“发布”了 v0.0.1 版本。*

---

### 第二阶段：使用 (在 `user-api` 业务项目操作)

#### 场景 A：如果 `ab-go` 是【公开】仓库

这是最简单的情况。

1. **下载依赖**：
在你的 `user-api` 项目根目录下执行：
```bash
go get github.com/your-username/ab-go@v0.0.1

```


2. **在代码中使用**：
```go
import "github.com/your-username/ab-go/ginx/validator"

```



---

#### 场景 B：如果 `ab-go` 是【私有】仓库 (重点)

绝大多数通用库都是私有的，直接 `go get` 会报错（403 Forbidden 或找不到包），因为 Go 默认走公共代理（proxy.golang.org），代理无法访问你的私有仓库。

你需要配置 Go 环境变量：

1. **配置 GOPRIVATE**：
告诉 Go：“这个域名下的仓库是私有的，别走代理，直接去 GitHub 拉。”
```bash
# 将你的用户名替换进去
go env -w GOPRIVATE=github.com/your-username

```


*如果公司有多个私有库，可以写 `github.com/your-company/**`
2. **配置 Git 鉴权**：
Go 底层是调用 `git` 命令来下载代码的。你需要确保你的终端能通过 Git 访问该私有仓库。
* **方式 1 (推荐SSH)**：配置好 SSH Key，确保 `git clone git@github.com...` 能成功。
还需要配置 Git 强制走 SSH（因为 Go 默认用 HTTPS）：
```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"

```


* **方式 2 (HTTPS + Token)**：确保你电脑记住了 GitHub 密码或 Token。


3. **下载依赖**：
现在再执行：
```bash
go get github.com/your-username/ab-go@v0.0.1

```



---

### 第三阶段：本地联调 (高级技巧)

在开发过程中，你可能发现 `ab-go` 有个 Bug，需要修改。

* **痛苦的路径**：改 `ab-go` -> 提交 -> 打标签 v0.0.2 -> 业务项目 `go get @v0.0.2` -> 发现还有 Bug -> 重复...
* **优雅的路径**：使用 `replace` 指令。

假设你的目录结构是这样的：

```text
workspace/
├── ab-go/      (通用库)
└── user-api/   (业务项目)

```

你想在 `user-api` 里直接使用本地的 `ab-go` 进行调试：

1. 在 `user-api/go.mod` 文件末尾添加：
```go
// 告诉 Go：别去网上下了，直接用我隔壁文件夹里的代码
replace github.com/your-username/ab-go => ../ab-go

```


2. 现在你在本地修改 `../ab-go` 里的代码，`user-api` 重启后会立即生效，无需推送，无需打标签。
3. **调试完成后**：
* 去 `ab-go` 提交代码，打新标签 `v0.0.2`。
* 回到 `user-api`，**删除** `replace` 行。
* 执行 `go get github.com/your-username/ab-go@v0.0.2` 升级到正式版。



### 总结

1. **发布** = `git tag v0.0.1` + `git push origin v0.0.1`。
2. **私有库** = 设置 `GOPRIVATE`。
3. **日常开发** = 使用 `go.mod` 的 `replace` 指令进行本地联调。