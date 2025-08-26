+++
title = 'Wrk性能测试工具'
date = 2025-08-26T08:54:31+08:00
draft = false
categories = [ "Go" ]
tags = [ "go", "wrk" ]
+++

**目标**

- wrk 入门
- 使用 wrk 压测已有接口
    写接口测试
    读接口测试

**wrk 安装**

```bash
# Linux
apt install wrk

# MacOS
brew install wrk

# 源码编译安装
git clone https://github.com/wg/wrk.git
# 编译之后得到一个 wrk 可执行文件
cd wrk && make
export {wrk ENV}
```



**命令**

```shell
✗ wrk -t1 -d1s -c2 -s ./scripts/wrk/signup.lua http://localhost:8080/users/signup
Running 1s test @ http://localhost:8080/users/signup
  1 threads and 2 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   109.33ms    7.50ms 130.07ms   88.89%
    Req/Sec    18.44      3.57    20.00     88.89%
  18 requests in 1.01s, 2.27KB read
Requests/sec:     17.84
Transfer/sec:      2.25KB
```

参数介绍：

- `-t`：后面跟着的是线程数量。不要超过本地CPU的核数，比如本地机器是8核就可以设置为4。
- `-d`：后面跟着的是持续时间，比如说 1s 是一秒，也可以是 1m，是一分钟。
- `-c`：后面跟着的是并发数。
- `-s`：后面跟着的是测试的脚本。

通常在测试接口极限的时候一般调整 `-t` 和 `-c` 两个参数。

> wrk 测试最重要的是需要学会编写 `lua 脚本`



