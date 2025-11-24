+++
title = '当你的项目多得像一盘散沙，是时候请出 Monorepo 这位“收纳大师”了！'
date = 2025-08-30T18:52:25+08:00
draft = false
categories = [ "PM" ]
tags = [ "pm", "monorepo"]
+++

## 一、你的“开发桌面”是不是也乱成了一锅粥？

想象一下这个场景：

你的桌面上摊着好几个项目——用 Go 写的后端、用 Python 跑的 AI 模型、iOS 和 Android 两个 App、一个 React 前端，外加微信小程序和公众号...

每个项目都是一个独立的文件夹（Git 仓库），每次想做个涉及前后端的修改，你都得像个DJ一样，在好几个窗口之间疯狂“切碟”。时间久了，不仅心累，还容易出错。

如果你对这个场景感同身受，那么恭喜你，你遇到了一个“甜蜜的烦恼”，而 **Monorepo** 就是帮你解决这个烦恼的“收纳魔法”。本文将带你从零开始，使用 `Nx` + `pnpm` 这对黄金搭档，亲手打造一个能容纳 Go、React 等多语言项目的 Monorepo 工作区。

## 二、Monorepo：管理“项目动物园”的神器

把所有项目都放进一个仓库（Monorepo），听起来有点疯狂，但它带来的好处会让你大呼“真香”：

1.  **上帝视角，一目了然**
    再也不用在几十个浏览器标签页或IDE窗口间反复横跳了。所有代码都在一个地方，整个技术版图清清楚楚，管理起来心情都变好了。

2.  **代码共享，如丝般顺滑**

      * **前后端“心灵相通”**：后端 Go 定义了一个数据结构（DTO），前端 React 想用？在 Monorepo 里，你只需创建一个 `libs/types` 共享库，前后端直接引用就行。告别了过去需要发 npm 包或 Go module，然后两边再小心翼翼地更新版本的繁琐流程。
      * **跨平台“组件复用”**：未来小程序和 App 想共享一套工具函数或业务逻辑？在 Monorepo 架构下，这种共享是天生的优势，而非后期的“技术改造”。

3.  **提交与重构，指哪打哪**

      * **原子化提交 (Atomic Commits)**：想象你改了一个后端 API，需要同时更新 Go 服务端、React 前端和 Android App。在 Monorepo 里，这一切可以在一个 commit 中完成！这意味着你的代码库在任何一个历史节点都是**一致且可运行的**，彻底告别了“A 仓库更新了，B 仓库还没跟上，项目挂了”的尴尬。
      * **“地毯式”重构**：想给某个核心函数改个名？在 IDE 里直接全局搜索替换，从后端到前端，一键搞定，安全又高效。

4.  **CI/CD，精准而高效**
    你可以配置一套统一的构建部署流水线。更酷的是，借助 `Nx` 这样的智能工具，当你只改了 React 前端的代码时，CI/CD 会**只构建和部署前端应用**，后端、App 等项目纹丝不动，大大节省了时间和计算资源。

## 三、我们的“神兵利器”：Nx + PNPM

  - **Nx**: 我们的“项目总管家”，它能理解项目之间的依赖关系，实现智能的构建、测试和缓存。
  - **PNPM**: 高效的包管理器，它的 `workspace` 特性是实现 Monorepo 的天然盟友。

**给总管家发“上岗证”**
在动手之前，先在你的电脑上全局安装 Nx。这就像在管理仓库前先安装 Git 一样。

```shell
pnpm add --global nx
```

## 四、实战演练：从零到一，打造你的多语言 Monorepo

### 第1步：开辟鸿蒙，创建工作空间

我们将使用 `create-nx-workspace` 脚手架来初始化项目。它很聪明，会自动检测到你安装了 pnpm。

```shell
# `xuepingmall` 是你的项目名，可以随心所欲地替换
# `--preset=apps` 表示创建一个空的、适合添加多个独立应用的 workspace
# `--pm=pnpm` 明确指定包管理器
npx create-nx-workspace@latest xuepingmall --preset=apps --pm=pnpm
```

整个过程行云流水：

完成后，进入 `xuepingmall` 目录，你的“地基”就已经打好了：

```shell
xuepingmall/
├── node_modules/   # 依赖库存放地
├── nx.json         # Nx 的核心配置文件
├── package.json    # 项目的“身份证”
├── pnpm-lock.yaml  # 锁定依赖版本
└── README.md
```

**温馨提示**：一些旧教程可能会说此时会自动创建 `apps` 目录和 `pnpm-workspace.yaml` 文件。别担心，新版本下我们自己动手，丰衣足食！

### 第2步：Git 远程地址从 HTTPS 切换到 SSH

如果在 GitHub 推送代码时觉得 HTTPS 方式太慢或需要频繁输入密码，可以切换到更高效的 SSH 方式。

```shell
# 查看你当前的远程地址
git remote -v

# 将 origin 的地址替换成你的 SSH 地址
git remote set-url origin git@github.com:YourUsername/YourRepoName.git
```


**规划“功能分区”**
在根目录下，创建两个核心目录：

```shell
mkdir apps libs
```

  - `apps/`: **“成品展示区”**。这里存放所有可以独立部署的应用，比如你的 React 前端、Go 后端服务、Android App 等。
  - `libs/`: **“共享工具箱”**。这里存放所有可复用的代码库，比如共享的工具函数、公共 UI 组件、通用类型定义等。

**配置 pnpm Workspace**
在项目根目录下，确保你有一个 `pnpm-workspace.yaml` 文件，告诉 pnpm 它的管辖范围。如果没​​有，就创建一个：

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'libs/*'
```

### 第2步：让第一个“居民”——Go 后端服务入驻

**创建 Go 项目的“家”**
由于后端可能采用微服务架构，我们先在 `apps` 里为所有后端服务建一个总目录 `backend-core`。

```shell
mkdir apps/backend-core
cd apps/backend-core

# 初始化 Go Workspace，让多个 Go 微服务能和谐共处
go work init
```

此时会在 `backend-core` 目录下生成一个名为 `go.work` 文件，内容如下：
```shell
go 1.25.0
```

**创建第一个微服务 `user-mgr`**

```shell
# 创建项目目录
mkdir user-mgr
cd user-mgr

# 初始化 Go Module
go mod init xuepingmall.com/user-mgr
```

**将其登记到 Go Workspace**

```shell
# 回到 backend-core 目录
cd ..
go work use ./user-mgr
```

`go.work` 此时内容更新如下：
```shell
go 1.25.0

use ./user-mgr
```

现在，`backend-core` 目录下的 `go.work` 文件记录了 `user-mgr` 的存在，方便统一管理。

**启动一个简单的 Gin 服务**
为了验证，我们在 `user-mgr/main.go` 中写入一个“Hello World”级别的服务：

```go
package main

import (
	"net/http"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong from user-mgr!",
		})
	})
	r.Run(":8081") // 监听 8081 端口
}
```

### 第3步：给 Go 项目在 Monorepo 里“上户口”

现在 Go 项目已经存在了，但我们的“总管家” Nx 还不知道。我们需要为它创建一个“身份证”——`project.json` 文件。

在 `apps/backend-core/user-mgr` 目录下新建 `project.json`：

```json
{
  "name": "user-mgr",
  "$schema": "../../../node_modules/nx/schemas/project-schema.json",
  "root": "apps/backend-core/user-mgr",
  "projectType": "application",
  "tags": ["scope:backend", "lang:go"],
  "targets": {
    "build": {
      "executor": "nx:run-commands",
      "options": {
        "command": "cd {projectRoot} && go build -o ../../dist/user-mgr ."
      },
      "outputs": ["{workspaceRoot}/dist/user-mgr"]
    },
    "serve": {
      "executor": "nx:run-commands",
      "options": {
        "command": "cd {projectRoot} && go run ."
      }
    },
    "test": {
      "executor": "nx:run-commands",
      "options": {
        "command": "cd {projectRoot} && go test ./..."
      }
    }
  }
}
```

这份“身份证”告诉了 Nx 关于 `user-mgr` 的一切：它的名字、位置、以及如何**构建(build)**、\*\*运行(serve)**和**测试(test)\*\*它。`outputs` 字段尤其重要，它告诉 Nx 构建产物的位置，以便启用缓存。

**验明正身**
运行 `nx show projects`，看看 Nx 是否已经认出了我们的新成员：

```shell
➜  xuepingmall git:(main) ✗ nx show projects
user-mgr
```

成功！或者，用更酷的方式，打开项目依赖图看看：

```shell
nx graph
```

### 第4步：启动项目，感受 Monorepo 的魔力



**点火，启动！**
回到项目根目录，用 Nx 的命令来启动我们的 Go 服务：

```shell
nx serve user-mgr
```

看到 Gin 服务的启动日志了吗？恭喜，你的第一个非 JS 项目已成功融入 Monorepo！

### 第5步：如法炮制，添加 React 前端项目

**手动创建 Vite + React 项目**
在 `apps` 目录下，使用 `pnpm create` 命令：

```shell
cd apps
# 根据提示操作，项目名称输入 web-frontend，选择 React 和 TypeScript
pnpm create vite@latest
```

操作完成后，`apps` 目录下会多出一个 `web-frontend` 文件夹。

**为前端项目“上户口”**
同样，在 `apps/web-frontend` 目录下新建 `project.json` 文件：

```json
{
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "name": "web-frontend",
  "sourceRoot": "apps/web-frontend/src",
  "projectType": "application",
  "tags": ["scope:frontend", "framework:react"],
  "targets": {
    "serve": {
      "executor": "nx:run-commands",
      "options": {
        "command": "pnpm dev",
        "cwd": "apps/web-frontend"
      }
    },
    "build": {
      "executor": "nx:run-commands",
      "options": {
        "command": "pnpm build",
        "cwd": "apps/web-frontend"
      },
      "outputs": ["{workspaceRoot}/apps/web-frontend/dist"]
    },
    "preview": {
      "executor": "nx:run-commands",
      "options": {
        "command": "pnpm preview",
        "cwd": "apps/web-frontend"
      }
    },
    "lint": {
      "executor": "nx:run-commands",
      "options": {
        "command": "pnpm lint",
        "cwd": "apps/web-frontend"
      }
    }
  }
}
```

这里的配置和 Go 项目类似，只是把 `targets` 里的命令换成了 `pnpm dev`、`pnpm build` 等前端专用命令。

**统一安装所有依赖**
回到 Monorepo 的根目录，运行 `pnpm install`。pnpm 会自动扫描所有 `apps/*` 和 `libs/*` 下的 `package.json`，并将所有依赖项安装到根目录的 `node_modules` 中。

这里切记如果目录过深，pnpm-workspace.yaml 需要配置到项目目录，否则前端无法执行下面命令。

```shell
# 确保在 xuepingmall/ 根目录下
pnpm install
```

**用 Nx 统一指挥前端项目**
现在，你可以像指挥官一样，在根目录用同样简洁的 Nx 命令来操作前端项目了：

```shell
# 启动开发服务器
nx serve web-frontend

# 构建生产包
nx build web-frontend

# 检查代码格式
nx lint web-frontend
```

**为前端项目安装依赖**

这个操作需要在 Monorepo 的根目录下执行，而不是进入到前端项目的子目录中。

具体步骤如下：

假设您要为名为 web-frontend 的前端项目安装 axios 这个库：

1、打开终端，并确保您位于 Monorepo 的根目录（即 xuepingmall/ 目录下）。

执行以下命令：
```shell
pnpm add axios --filter web-frontend
```

- pnpm add <package-name>: 这是 pnpm 添加依赖的标准命令。
- --filter <project-name>: 这是关键部分。它告诉 pnpm，这个命令只针对名为 web-frontend 的这个项目生效。项目名称是在其 package.json 文件里的 "name" 字段定义的。

如果您想安装一个开发依赖（devDependency），可以添加 -D 标志：

```shell
pnpm add typescript -D --filter web-frontend
```

为什么这样做？

在 pnpm 工作空间（workspace）中：

- 集中管理：所有项目的依赖最终都会被安装到根目录的 node_modules 文件夹中，并通过符号链接（symlinks）的方式供各个项目使用，这极大地节省了磁盘空间。
- 精确操作：使用 --filter 标志可以确保依赖被正确地添加到 apps/web-frontend/package.json 文件中，而不是错误地加到根目录的 package.json 或其他项目中。
- 维护一致性：执行命令后，pnpm 会自动更新根目录下的 pnpm-lock.yaml 文件，以保证整个 Monorepo 中所有依赖版本的一致性和锁定。

-----

## 附录

### 将已有的 Git 仓库导入 Monorepo

如果你想把一个已存在的独立项目（比如 `Observatory`）迁移到 Monorepo 的 `apps/backend-core` 目录下，`git subtree` 是个不错的选择。

```shell
# 1. (可选) 添加一个远程别名，方便操作
git remote add backend_repo_local /path/to/your/existing/Observatory

# 2. 使用 subtree 将对方的 main 分支添加到指定前缀目录下
git subtree add --prefix=apps/backend-core backend_repo_local main

# 3. (可选) 移除远程别名
git remote remove backend_repo_local
```

恭喜你！你已经成功掌握了搭建和管理一个多语言 Monorepo 的核心技能。现在，去把你那盘散沙般的项目都收纳进这个强大的工作区吧！

### 安装 `gh`

需要提前安装 `gh`，否则会出现问题。

```shell
brew install gh

sudo apt install gh -y
```

