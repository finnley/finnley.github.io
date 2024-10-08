+++
title = 'Shell 子程序'
date = 2024-01-21T18:38:14+08:00
draft = false
categories = [ "Linux" ]
+++

# 1 神秘空间

很多玄幻小说里面都有这样类似的设定：男主角有一个别人都不知道的金手指——神秘空间。

男主角可以在这个神秘空间内随心所欲。在这个空间内，他就是主宰。神秘空间甚至还可以收纳外界物品，但也存在着限制，无法将人收进来，除非是对自己非常信任的身边人，对自己没有敌意，才能将他们收进神秘空间。

那这一神秘空间与这篇笔记的 “Shell 子程序” 有什么联系吗？

# 2 主角登场

《鸟哥的Linux私房菜》中有一条关于 `Shell 变量` 的设置规则：

> 若变量需要在其他子程序执行，则需要以 `export` 来使变量变成环境变量，如：`export VariableName`

有个点引起我的疑问：Shell 中的 `子程序` 是什么？下面有请我们的主角✨闪亮✨登场👏🏻👏🏻：

1、输出变量 name，由于事先没有设置，所以输出为空。
```shell
# echo $name

```

2、设置变量 name，值为 test。
```shell
# name=test
#
```

3、再次输出变量 name ，值为 test。
```shell
# echo $name		 
test
```

4、输入 `bash`（进入到所谓的“子程序”中）
```shell
# bash
#
```
发现在回车后终端并没有任何变化。

5、输出变量 name，值为空。
```shell
# echo $name		

```
刚才我明明已经设置了 name 的值为 test，并且也输出了，为什么在输入了 `bash` 后再输出反而没有值了呢？

6、承接【5】，输入 `exit` （退出 “子程序”回到父程序）
```shell
# exit
exit	
```	

7、使用 `export` 导出变量 name。
```shell
# export name
```

8、再次输入 `bash`，（再次进入“子程序”中）。
```shell
# bash
#
```
	
9、再次输出变量 name，这次输出值为 test
```shell		
# echo $name
test
```

10、最后再次退出。
```shell
# exit
exit
```

完整过程如下：

![](/images/linux/shell/10.png)


# 什么是子程序

> **在当前的 shell 中去启用另一个新的 shell，新的 shell 就是子程序了**。

一般情况下，像上面的例子展示的那样，父程序中的自定义变量是无法在子程序中使用的，但是可以通过 `export` 将 `变量转成环境变量` 后，就能够在子程序中使用了。 

当我们登录到 Linux 之后，会自动获取一个 Shell，比如 CentOS 7 中默认的 Shell 是 Bash，这个 Bash 就是一个独立的程序，它会有一个独有的程序识别码，也就是 PID。
接下来在 Bash 下下发的所有指令都是由这个 Bash 衍生出来的，而那些下发的指令就被称为 “子程序”，而这个独立的 Bash 也就是“父程序”了。

为了便于理解画了一个图:

![](/images/linux/shell/20.png)

如上图所示，我们在原本的 bash 中执行另一个 bash ，结果操作的环境接口会跑到第二个 bash 去（就是子程序），那原本的 bash 就会暂停 （睡着了，就是 sleep）。<br> 
整个指令运行的环境是实线的部分！若要回到原本的 bash 去，就只有将第二个 bash 结束掉 （下发 exit 或 logout）才行。

* 子程序仅会继承父程序的环境变量，子程序不会继承父程序的自定义变量！在原本 bash 的自定义变量在进入了子程序后就会消失不见，直到离开子程序并回到原本的父程序后，这个变量才会又出现。
* 当子程序执行完后，在子程序内定义的各项变量或动作将会结束而不会传回到父程序中。

为解决“子程序无法使用父程序的自定义变量”这一问题，就得借助 `export` 。<br>
`export` 的作用就是分享自己的变量设置给后来调用的文件或其他程序，如果仅下达 `export` 而没有接变量，此时就会将所有的环境变量展示出来。

# 3 练习

1、在一个名为 setup.sh 的 shell 脚本中定义了一些变量，使用 `cat` 命令来查看 setup.sh 文件内容：
```shell
# cat setup.sh
export SERVICE_INSTALL_MYSQL=true
export SERVICE_INSTALL_REDIS=true
export SERVICE_INSTALL_CONSUL=true
```

2、赋予文件可执行权限并执行 setup.sh 脚本
```shell
# chmod +x setup.sh 
# ./setup.sh 
```

3、输出脚本内定义的变量，发现为空
```shell
# echo ${SERVICE_INSTALL_MYSQL}
                                    // 输出为空
#
```

完成过程如下：
![](/images/linux/shell/30.png)

**这是什么原因呢？**

这是因为“脚本执行”和“手动输出变量”两个操作是不在同一个 Shell 进程中进行的。我在脚本中定义了变量并使用 `export` 导出变量，然后在另一个 Shell 窗口或会话中输出变量，此时是无法获取到脚本执行的进程中定义的变量的。因为每个 Shell 窗口或会话都是独立的进程，它们有自己的环境变量。

**如何验证是脚本执行和输出变量两个是不同的 Shell 进程呢？**

可以在 setup.sh 文件最后一行增加用于输出当前 Shell 进程的 PID 的命令：
```shell
echo $$
```

- `$` 本身就是个变量，表示的是 `当前这个 Shell 的线程号`，亦即是所谓的 PID （Process ID）”

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

结果发现两次输出的 PID 是不同的，说明它们是两个不同的 Shell。

如果想要能够输出脚本内定义的变量，如何实现呢？

在解决上面提出的问题前，这里先介绍下脚本的几种执行方式

**1、直接执行**

* 路径方式执行（不论是绝对路径还是相对路径）
* 利用 bash （或者 sh） 方式执行

这两种直接执行方式执行脚本都会使用一个新的 bash 环境来执行脚本内的指令（新的 shell 进程）。

也就是说使用这两种执行方式时，其实 script 是在子程序的 bash 内执行的！这里就隐藏了一个概念：“`当子程序完成后，在子程序内的各项变量或动作将会结束而不会传回到父程序中`”！

例：有脚本 `showname.sh` 内容如下：
```shell
#!/bin/bash

read -p "Please input your firstname: " firstname # 提示使用者输入
read -p "Please input your lastname: " lastname # 提示使用者输入
echo -e "\nYour full name is ${firstname} ${lastname}" # 屏幕输出
```

完整执行过程如下：
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

![](/images/linux/shell/40.png)

> 上面例子证明了子程序中的变量在结束后时不会回传到父程序的。

为便于理解仍然借用上面的图来解释：

![](/images/linux/shell/20.png)

当使用直接执行的方式来处理时，系统会给予一支新的 bash 来让我们来执行 showname.sh 里面的指令，因此两个 firstname, lastname 其实是在上图中子程序 bash 内执行的。

当 showname.sh 执行完毕后，子程序 bash 内的所有数据便被移除，回到父程序，在父程序中输出两个变量就看不到内容了。

**2、通过 source 执行**

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

![](/images/linux/shell/50.png)

这是因为通过 `source` 方式执行会让 showname.sh 在父程序中执行，因此各项动作都会在原本的 bash 内生效，这也是有时候在不退出系统时往 `~/.bashrc` 写入数据后设置生效时需要执行下 `source ~/.bashrc`。

source 方式执行流程如下图：

![](/images/linux/shell/60.png)

# 小结

总结下，通过 `source` 就可以解决上面的问题。 `如果想要能够输出脚本内定义的变量，可以通过 source 方式来执行脚本`。



