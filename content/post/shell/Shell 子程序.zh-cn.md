+++
title = 'Shell 子程序'
date = 2024-01-21T18:38:14+08:00
draft = false
categories = [ "Linux", "Shell" ]
tags = [ "linux", "shell" ]
+++

在《鸟哥的Linux私房菜》中看到关于 Shell 变量的设置规则，有这样一个规则：

> 若该变量需要在其他子程序执行，则需要以 `export` 来使变量变成环境变量：

```shell
export VariableName
```

一时间不明白什么是子程序，shell 中怎么还有子程序这一说法吗？

直到继续往后看，看到类似下面一个例子：

* 如何将设置的变量用在下一个 shell 程序中？

```shell
# echo $name		// 输出变量 name
                    // 因为没有设置，所以输出为空
# name=test			// 设置变量 name，值为 test
# echo $name		// 再次输出变量
test				// 变量值为 test
# bash				// 进入到所谓的子程序中
# echo $name		// 输出变量 name
                    // 值为空，刚才明明已经设置了，并且也可以输出，为什么呢？
# exit				// 子程序，退出子程序
exit				
# export name		// 导出变量
# bash				// 再次进入所谓的子程序中
# echo $name		// 再次输出变量
test				// 可以输出变量 name 的值为 test
# exit				// 退出子程序
exit
```

总结下什么是子程序？
就是在当前的 shell 中去启用另一个新的 shell，新的 shell 就是子程序了。
一般状态下，想上面的例子展示的那样，父程序中的自定义变量是无法在子程序中使用的，但是可以通过 export 将变量转成环境变量后，就能够在子程序中使用了。


于是在我实际使用中遇到了下面的场景

1、在一个名为 setup.sh 的 shell 脚本中定义了一些变量：

```shell
export SERVICE_INSTALL_MYSQL=true
export SERVICE_INSTALL_REDIS=true
export SERVICE_INSTALL_CONSUL=true
```

2、执行 setup.sh

3、执行后输出脚本变量，发现是空的

完整流程如下：

```shell
# cat setup.sh                      // 查看 setup.sh 文件内容
export SERVICE_INSTALL_MYSQL=true
export SERVICE_INSTALL_REDIS=true
export SERVICE_INSTALL_CONSUL=true
# chmod +x setup.sh                 // 赋予文件可执行权限
# ./setup.sh                        // 执行脚本
# echo ${SERVICE_INSTALL_MYSQL}     // 输出脚本内定义的变量
                                    // 输出为空
#
```

这是什么原因呢？

脚本和手动输出不在同一个 Shell 进程中：如果我在脚本中定义了变量，然后在执行脚本后在另一个 Shell 窗口或会话中尝试手动输出变量，那么是无法获取到脚本内定义的变量的。
因为每个 Shell 窗口或会话都是独立的进程，它们有自己的环境变量。
