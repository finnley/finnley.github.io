+++
title = 'Shell 子程序'
date = 2024-01-21T18:38:14+08:00
draft = false
categories = [ "Linux" ]
+++

## Shell 子程序

《鸟哥的Linux私房菜》中有一条关于 Shell 变量的设置规则：

> 若变量需要在其他子程序执行，则需要以 `export` 来使变量变成环境变量：`export VariableName`

有两点引起我的疑问：一个是 Shell 中的“子程序” 子程序是什么？

### 如何将设置的变量用在下一个 shell 程序中？

先跟着书中案例引出“子程序”：

```shell
# 输出变量 name，因为事先没有设置，所以输出为空
# echo $name

# 设置变量 name，值为 test
# name=test		
# 再次输出变量，变量 name 值为 test	
# echo $name		 
test		
# 进入到所谓的子程序中		
# bash
# 在子程序中输出变量 name，值为空，刚才明明已经设置了，并且也可以输出，那这里为什么没有输出呢？
# echo $name		

# 此时仍然在子程序，exit 退出子程序
# exit
exit		
# 回到父程序中导出变量		
# export name	
# 再次进入所谓的子程序中	
# bash		
# 此是处理子程序中，再次输出变量，name 的值为 test		
# echo $name
test
# 退出子程序
# exit
exit
```

*什么是子程序呢？*

就是在当前的 shell 中去启用另一个新的 shell，新的 shell 就是子程序了。<br>
一般情况下，像上面的例子展示的那样，父程序中的自定义变量是无法在子程序中使用的，但是可以通过 “export” 将变量转成环境变量后，就能够在子程序中使用了。


当我们登录到 Linux 之后，会自动获取一个 Shell，比如 CentOS7 中默认的 Shell 是 Bash，这个 Bash 就是一个独立的程序，它会有一个独有的程序识别码，也就是 PID。
接下来在 Bash 下下发的所有指令都是由这个 Bash 衍生出来的，而那些下发的指令就被称为 “子程序”，而这个独立的 Bash 也就是“父程序”了。

画一个图方便理解:

![](/img/shell/10.png)

如上图所示，我们在原本的 bash 中执行另一个 bash ，结果操作的环境接口会跑到第二个 bash 去（就是子程序），那原本的 bash 就会在暂停的情况 （睡着了，就是 sleep）。<br> 
整个指令运行的环境是实线的部分！若要回到原本的 bash 去，就只有将第二个 bash 结束掉 （下达 exit 或 logout） 才行。

* 子程序仅会继承父程序的环境变量，子程序不会继承父程序的自定义变量！在原本 bash 的自定义变量在进入了子程序后就会消失不见，直到离开子程序并回到原本的父程序后，这个变量才会又出现。
* 当子程序执行完后，在子程序内定义的各项变量或动作将会结束而不会传回到父程序中。

为解决“子程序无法使用父程序的自定义变量”这一问题，就得借助 `export` 。<br>
`export` 的作用就是分享自己的变量设置给后来调用的文件或其他程序，如果仅下达 `export` 而没有接变量，此时就会将所有的环境变量展示出来。

#### 案例

场景一：

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

这是什么原因呢？<br>

这是因为脚本执行和手动输出变量两个操作是不在同一个 Shell 进程中进行的<br>
我在脚本中定义了变量并执行，然后在另一个 Shell 窗口或会话中尝试手动输出变量，此时是无法获取到脚本执行的进程中定义的变量的。因为每个 Shell 窗口或会话都是独立的进程，它们有自己的环境变量。

如何验证是两个不同的 Shell 进程呢？<br>

可以在 setup.sh 文件最后一行增加用于输出当前 Shell 进程的 PID 的命令:

```shell
echo $$
```

`$` 本身就是个变量，表示的是“当前这个 Shell 的线程号”，亦即是所谓的 PID （Process ID）”

执行完 setup.sh 文件后在当前窗口中执行 `echo $$`，观察两次输出的 PID 是否相同，如果不相同就表示是不同的 Shell 进程，完整过程如下：

```shell
# cat setup.sh
export SERVICE_INSTALL_MYSQL=true
export SERVICE_INSTALL_REDIS=true
export SERVICE_INSTALL_CONSUL=true

echo $$
# ./setup.sh
1703
# echo $$
1568
#
```

结果发现两次输出的 PID 是不同的。

如果想要能够输出脚本内定义的变量，如何实现呢？

## 脚本的执行

在解决上面提出的问题前，这里先介绍下脚本的几种执行方式

#### 直接执行

* 路径方式执行（不论是绝对路径还是相对路径）
* 利用 bash （或者 sh） 方式执行

这两种直接执行方式执行脚本都会使用一个新的 bash 环境来执行脚本内的指令（新的 shell 进程）。<br>
也就是说使用这种执行方式时，其实 script 是在子程序的 bash 内执行的！这里就隐藏了一个概念：“当子程序完成后，在子程序内的各项变量或动作将会结束而不会传回到父程序中”！

例：

有脚本 `showname.sh` 内容如下：

```shell
#!/bin/bash

read -p "Please input your firstname: " firstname # 提示使用者输入
read -p "Please input your lastname: " lastname # 提示使用者输入
echo -e "\nYour full name is ${firstname} ${lastname}" # 屏幕输出
```

执行过程如下：

```shell
# 此前并未定义两个变量，故输出为空
# echo ${firstname} ${lastname}

# 执行脚本，使用两个变量接收用户输入的值，然后输出两个变量的值
# sh showname.sh
Please input your firstname: Li
Please input your lastname: Lei
Your full name is Li Lei
# 脚本执行完后再次输出两个变量，结果仍然为空
# echo ${firstname} ${lastname}

#
```

上面例子证明了子程序中的变量在结束后时不会回传到父程序的。

为便于理解仍然借用上面的图来解释：

![](/img/shell/10.png)

当使用直接执行的方式来处理时，系统会给予一支新的 bash 来让我们来执行 showname.sh 里面的指令，因此两个 firstname, lastname 其实是在上图中子程序 bash 内执行的。<br>
当 showname.sh 执行完毕后，子程序 bash 内的所有数据便被移除，回到父程序，在父程序中输出两个变量就看看不到内容了。


#### 通过 source 执行

还是上面的 `showname.sh` 脚本，完整执行过程如下：

```shell
# 执行脚本，并使用两个变量接受用户输入，然后输出两个变量值
# source showname.sh
Please input your firstname: Li
Please input your lastname: Lei

Your full name is Li Lei
# 脚本执行完后在父程序中输出脚本中定义的变量，会看到有输出
# echo ${firstname} ${lastname}
Li Lei
#
```

这是因为通过 `source` 方式执行会让 showname.sh 在父程序中执行，因此各项动作都会在原本的 bash 内生效，这也是有时候在不退出系统时往 `~/.bashrc` 写入数据后设置生效时，需要使用 `source ~/.bashrc`

source 方式执行如下图：

![](/img/shell/20.png)

总结下，通过 source 就可以解决上面的问题 “如果想要能够输出脚本内定义的变量，可以通过 source 方式来执行脚本”。

