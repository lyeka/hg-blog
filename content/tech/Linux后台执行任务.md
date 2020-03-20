---
title: "Linux后台执行任务"
date: 2020-03-10
keywords: ["后台执行"]
description: "Linux后台执行任务的几种方式"
tags: ["linux"]
categories: ["linux"]
isCJKLanguage: true
---

# Linux后台执行任务

有时候执行某些脚本/任务耗时比较长，不想一直使其占据终端或者因为终端关闭而受影响（例如网络导致从

服务器断开等），需要将其放入后台执行。

当用户注销（logout）或者网络断开时，终端会收到 HUP（hangup）信号从而关闭其所有子进程。因此，我们的解决办法就有两种途径：要么让进程忽略 HUP 信号，要么让进程运行在新的会话里从而成为不属于此终端的子进程。

这时候可选下面其中方式让其在后台执行：

- nohup
- &
- setsid



## nohup

用法： 只需要在命令前加上nohup即可

标准输出和标准错误却省的话会自动重定向输出的nohup.out文件。可加入&将其放入后台执行，并加上`> xxx.log  2>&1`在命令后面重定向输出

```bash
# nohup <command> & > <filename> 2>&1
nohup ping www.baidu.com & > out.log 2>&1
```





## &

用法：在命令后加入&即可

标准输出和标准错误缺省的话还是会输出到终端，终端关闭的话后台任务也是会终止的，故一般搭配nohup使用比较方便

```bash
# <command> &
ping www.baidu.com &
```





## setsid

用法：在命令前加入setsid即可

原理：将程序/任务在一个新的session中执行，即不属于当前终端的子进程，就不会受到当前HUP的影响



## 管理后台任务

使用nohug/& 的话会将任务放入后台任务队列，下面介绍如何管理它们



### 查看

`jobs` 加上-l还可以看其任务的进程号



### 将其前台执行

`fg <job id>`



### 将其后台执行

`bg <job id>`



### 终止

如果当前终端未关闭的话，使用jobs可以查看当前的后台任务队列。这时候可以使用fg将其调到前台执行，在执行对应的终止操作，或者jobs -l拿到进程号，再执行kill等操作。

如果终端已经关闭，再次使用jobs是看不到后台的任务队列的，这时候只可以使用ps等找到对应的进程号进行终止操作了。



## 重定向输出

0， 1， 2分别代表着标准输入，标准输出，标准错误三个文件描述符

在命令后使用 `> <filename> ` 等同于 `1 > <filename>`，即将标准输出重定向输出到对应的文件

而 2>&1 代表将标准错误重定向到标准输出

所以`<command> > out.log 2>&1`代表将标命令输出和错误输出都重定向打印到out.log文件



参考

- https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/index.html#nohup
- https://blog.csdn.net/qq_31821675/article/details/78246808
- https://blog.csdn.net/u011630575/article/details/52151995

