---
title: command教程
date: 2022-10-24 17:39:20
tags:
---



# 命令行教程



工作流，终端复用，dotfile配置，还有远程服务器





## 2.工作流



ctrl+c是打断程序

ctrl+z是暂停

ctrl+\是结束程序 





**使用&表示程序后台执行**



使用jobs可以查看当前执行的进程状态，

使用bg %1，回复倍暂停的jobs

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666049269001666604925973.png)





产生式挂起进程使用nohup

  





## 2.tmux

一般使用screen，不过也学习一下

三大核心，session，windows，panel

- 会话

   

  \- 每个会话都是一个独立的工作区，其中包含一个或多个窗口

  - `tmux` 开始一个新的会话
  - `tmux new -s NAME` 以指定名称开始一个新的会话
  - `tmux ls` 列出当前所有会话
  - 在 `tmux` 中输入 `<C-b> d` ，将当前会话分离
  - `tmux a` 重新连接最后一个会话。您也可以通过 `-t` 来指定具体的会话

windows==tab（浏览器的窗口）





- 会话

   

  \- 每个会话都是一个独立的工作区，其中包含一个或多个窗口

  - `tmux` 开始一个新的会话
  - `tmux new -s NAME` 以指定名称开始一个新的会话
  - `tmux ls` 列出当前所有会话
  - 在 `tmux` 中输入 `<C-b> d` ，将当前会话分离
  - `tmux a` 重新连接最后一个会话。您也可以通过 `-t` 来指定具体的会话



ctrl+a （n，是下一个，p是之前一个tmux窗口





## 3.重命名

alias把长命令进行缩短

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666058858981666605885498.png)



alias gc=“git clone”





如何写入重命名，关闭终端，重命名就结束了



**直接写入到dotfiles**





## 4.符号链接

就是和快捷方式一样

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666064699001666606468924.png)





## 5.ssh

这个已经熟练了

、



还有设置别名，在ssh里面进行配置，就不用一个刚刚输入IP了





## 6.homework



### 6.1任务控制

> 1. 我们可以使用类似 `ps aux | grep` 这样的命令来获取任务的 pid ，然后您可以基于pid 来结束这些进程。但我们其实有更好的方法来做这件事。在终端中执行 `sleep 10000` 这个任务。然后用 `Ctrl-Z` 将其切换到后台并使用 `bg`来继续允许它。现在，使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 来查找 pid 并使用 [`pkill`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 结束进程而不需要手动输入pid。(提示：: 使用 `-af` 标记)。



这一题使用的pger==ps aux|grep python

pgrep python





使用prep sleep可以得到

```
sleep 10000
Ctrl-Z
bg

pgrep sleep 

pkill  -af sleep

```







> 1. 如果您希望某个进程结束后再开始另外一个进程， 应该如何实现呢？在这个练习中，我们使用 `sleep 60 &` 作为先执行的程序。一种方法是使用 [`wait`](http://man7.org/linux/man-pages/man1/wait.1p.html) 命令。尝试启动这个休眠命令，然后待其结束后再执行 `ls` 命令。
>
>    但是，如果我们在不同的 bash 会话中进行操作，则上述方法就不起作用了。因为 `wait` 只能对子进程起作用。之前我们没有提过的一个特性是，`kill` 命令成功退出时其状态码为 0 ，其他状态则是非0。`kill -0` 则不会发送信号，但是会在进程不存在时返回一个不为0的状态码。请编写一个 bash 函数 `pidwait` ，它接受一个 pid 作为输入参数，然后一直等待直到该进程结束。您需要使用 `sleep` 来避免浪费 CPU 性能。

