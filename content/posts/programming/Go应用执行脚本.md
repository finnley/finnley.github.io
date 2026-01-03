+++
title = 'Go应用执行脚本失败问题分析'
date = 2025-11-25T16:12:15+08:00
draft = true
categories = [ "Programming" ]
tags = [ "go", "programming"]
+++

# 问题叙述

使用Go编写了一个执行Shell脚本的程序，Go应用与Shell脚本所在目录如下：

```
workspace
├── backend
│   ├── app
│   │   ├── main.go
└── scripts
    └── vms
        ├── .env
        ├── config.json
        ├── setup.sh
```

ecmp 为Go应用，vms 为所要执行的脚本文件所在目录，在运行Go应用时提示类似下面错误：
```
failed to execute script: exit status 1
/workspace/scripts/vms/setup.sh: line 4: 
.env: No such file or directory
jq: error: Could not open file ./config.json: No such file or directory
......
```

# 问题分析

这是一个非常经典的问题。

虽然我的脚本文件（setup.sh）、配置文件（.env、config.json）都在同一个目录下，但是当我通过 Go 程序调用脚本时，脚本执行的“`当前工作目录`”（Current Working Directory, CWD）`默认是 Go 编译好的二进制程序所在的目录`，而不是`脚本所在的目录`。

因此，当脚本里写了 source .env 或读取 ./config.json 时，它实际上是`在 Go 程序运行的目录下寻找这些文件`，而不是在 workspace/scripts/vms/ 下寻找，所以报 "No such file or directory"。

**核心原因分析**

1. Go 程序的视角：假设你的 Go 程序运行在 /App 目录。
2. 脚本的视角：你的脚本位于 /App/scripts/vms/setup.sh。
3. 执行过程：
    - Go 执行命令时，默认会在 /App 下启动 shell。
    - 脚本里写了 source .env 或 cat ./config.json。
    - Shell 会尝试在 /App/.env 寻找文件，而不是 /App/scripts/vms/.env。
    - 结果：找不到文件，抛出 No such file or directory。


解决方案
可以通过以下方式解决：一是修改 Go 代码

方案：在 Go 代码中指定 cmd.Dir (推荐)
这是最优雅的解决方法。在构建 exec.Command 对象后，显式地将 Dir 属性设置为脚本所在的目录。这样脚本执行时，就会自动以该目录为基准寻找文件。

```go
package main

import (
    "fmt"
    "os/exec"
    "path/filepath"
    "os"
)

func runScript(scriptPath string) error {
    // 1. 获取脚本的绝对路径
    absPath, err := filepath.Abs(scriptPath)
    if err != nil {
        return err
    }

    // 2. 获取脚本所在的目录 (例如 /Users/xxx/.../vms/)
    scriptDir := filepath.Dir(absPath)

    // 3. 创建命令
    cmd := exec.Command("/bin/bash", absPath)

    // 4. 【关键步骤】设置命令执行的工作目录
    cmd.Dir = scriptDir 

    // 5. 设置标准输出以便调试
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    // 可选：确保环境变量（如 PATH）被继承，以便找到 jq/yq
    // 如果你需要特定路径的工具，可以在这里追加
    cmd.Env = os.Environ() 

    fmt.Printf("Executing script in directory: %s\n", scriptDir)
    
    return cmd.Run()
}
```

补充检查：环境变量 (PATH) 问题
虽然你的错误日志显示是“找不到文件”，但你提到依赖 jq 和 yq。在某些环境（特别是 macOS 或使用 Homebrew 安装工具时），Go 程序启动的子 Shell 可能无法继承完整的 $PATH。

如果修复了目录问题后，提示 jq: command not found，请检查 Go 代码中是否正确传递了环境变量：

```go
// 确保当前进程的环境变量（包含 PATH）传递给了子进程
cmd.Env = os.Environ()
```

或者手动追加 Homebrew 路径（针对 macOS）：

```go
env := os.Environ()
// 假如 jq 安装在 /opt/homebrew/bin
env = append(env, "PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin")
cmd.Env = env
```

# 总结

- 错误现象：脚本能运行，但找不到脚本同级目录下的文件。
- 根本原因：Go exec 默认在 Go 二进制文件所在目录运行，而非脚本所在目录。
- 最佳实践：在 Go 代码中设置 cmd.Dir = filepath.Dir(scriptPath)。