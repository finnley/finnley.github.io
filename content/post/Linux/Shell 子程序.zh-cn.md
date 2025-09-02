+++
title = 'Shell 子程序'
date = 2024-01-21T18:38:14+08:00
draft = false
categories = [ "Linux" ]
+++

## 1 神秘空间

许多玄幻小说中都有类似的设定：主角拥有一方外人不知的“神秘空间”。

在这空间内，他就是主宰，可以收纳各种宝物。但也存在着限制：它默认无法收纳活物，除非是与主角心意相通或毫无敌意的人，方可被引入其中。

这个设定与我们今天要探讨的 Shell “子程序”惊人地相似。**实际上，我们可以把每一个 Shell 程序（进程），都看作一个独立的“空间”。**


## 2 变量 `name` 的奇妙之旅

《鸟哥的Linux私房菜》中提到一条规则：

> 若变量需要在其他子程序执行，则需要以 `export` 来使变量变成环境变量，如：`export VariableName`。

这里的“`子程序`”究竟是什么？为了理解它，最好的方式莫过于亲手实践。让我们通过一个简单的变量 `name` 来揭开它的✨神秘面纱✨。

**第1步：初探当前空间**

我们直接输出变量 `name`，由于尚未定义，输出为空。这好比我们的主空间里空空如也。

```shell
# echo $name

```

**第2步：在主空间放入“私有宝物”**

我们设置变量 `name` 的值为 `test`。

```shell
# name=test
# echo $name		 
test	
```

现在，我们的主空间里有了一个名为 `name` 的“私有宝物”。

**第3步：进入一个新的“神秘空间”**

我们输入 `bash` 命令。这个操作等同于在当前的 Shell 空间内，打开了一个全新的、独立的子空间（子程序）。

```shell
# bash
#
```

你会发现终端看似没有变化，但我们已经身处一个新的环境中了。

**第4步：在新空间中寻找宝物**
我们再次输出变量 `name`，却发现值为空。

```shell
# echo $name		

```

这正是“神秘空间”的第一条规则：**子空间默认无法访问父空间的“私有宝物”**。刚才定义的变量 `name` 是父空间的私有财产，子空间对此一无所知。

刚才我明明已经设置了 name 的值为 test，并且也输出了，这就出现了为什么在输入了 `bash` 后再输出反而没有值的现象。

**第5步：回归主空间**
输入 `exit`，我们便从子空间退出，回到了父空间（退出 “子程序”回到父程序）。

```shell
# exit
exit	
```	

再次查看 `name`，它的值 `test` 依然存在。

**第6步：将“私有宝物”变为“公共信物”**
我们使用 `export` 命令。这个命令的作用，就是将一个“私有宝物”标记为“公共信物”（环境变量），所有从当前空间衍生的新空间都能看到它。

```shell
# export name
```

**第7步：带着信物，再探新空间**
我们再次输入 `bash` 进入一个新的子空间（再次进入“子程序”中）。

```shell
# bash
#
```

**第8步：奇迹发生**
在新空间中输出变量 `name`，这次成功地得到了 `test`！

```shell		
# echo $name
test
```

这便是 `export` 的魔力：它打破了空间的壁垒，让变量得以被继承。

**第9步：旅程结束**
最后，我们再次 `exit` 退出子空间。

```shell
# exit
exit
```

完整过程如下：

![](/images/linux/shell/10.png)

## **3. 到底什么是子程序？**

现在我们可以给出清晰的定义了：

> **在当前的 shell 中去启用另一个新的 shell，新的 shell 就是子程序了**。

一般情况下，像上面的例子展示的那样，父程序中的自定义变量是无法在子程序中使用的，但是可以通过 `export` 将 `变量转成环境变量` 后，就能够在子程序中使用了。 

当我们登录 Linux 系统后，会获得一个默认的 Shell（例如 Bash），它拥有一个唯一的进程ID（PID），它是一个独立程序，它会有一个独有的程序识别码，也就是 PID，这就是我们的“主空间”或“父程序”。之后我们执行的许多命令，都会衍生出新的进程，这些就是“子程序”。


为了便于理解画了一个图:

![](/images/linux/shell/20.png)

如上图所示，我们在原本的 bash 中执行另一个 bash ，结果操作的环境接口会跑到第二个 bash 去（就是子程序），那原本的 bash 就会暂停 （睡着了，就是 sleep）。<br> 
整个指令运行的环境是实线的部分！若要回到原本的 bash 去，就只有将第二个 bash 结束掉 （下发 exit 或 logout）才行。

这个主从空间的关系遵循两条铁律：

* 子程序仅会继承父程序的环境变量，子程序不会继承父程序的自定义变量！在原本 bash 的自定义变量在进入了子程序后就会消失不见，直到离开子程序并回到原本的父程序后，这个变量才会又出现。
* 当子程序执行完后，在子程序内定义的各项变量或动作将会结束而不会传回到父程序中。

为解决“子程序无法使用父程序的自定义变量”这一问题，就得借助 `export` 。<br>
`export` 的作用就是分享自己的变量设置给后来调用的文件或其他程序，如果仅下达 `export` 而没有接变量，此时就会将所有的环境变量展示出来。

## 4 练习

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

## 5 小结

总结下，通过 `source` 就可以解决上面的问题。 `如果想要能够输出脚本内定义的变量，可以通过 source 方式来执行脚本`。



