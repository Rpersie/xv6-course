6.828 2014 第一课: O/S 概述
=

## 概述

* 6.828 目标
  * 通过设计和实现一个小型的OS来详细的理解操作系统
  * 构建系统的实践经验("Applying 6.033")

* 应用程序想从OS获取什么?
  * 将硬件抽象为便携的接口
  * 多路复用硬件，使得多个程序可以同时运行
  * 隔离应用程序所包含的错误(Isolate applications to contain bugs)
  * 允许将计算机资源在多个应用程序间共享

* 什么是OS?
  * 例如: OSX, Windows, Linux
  * 微观上: 一个硬件管理的库
  * 宏观上: 物理机器 -> abstract one w/ better properties

* 组织结构: 分层描述
   硬件: CPU, mem, disk
   内核: [各种各样的服务]
   用户空间: 应用程序, 例如: vi 和 gcc
  * 我们非常关心各层的接口和内核内部的结构

* 通常OS的内核都提供哪些服务?
  * 进程
  * 内存
  * 文件内容
  * 目录和文件名
  * 安全
  * 还有很多其他的,如: 用户, 进程间通信, 网络, 时间, 终端

* OS抽象起来像什么?
  * 应用程序只能通过系统调用来看到OS
  * 例如, 从 UNIX / Linux中:

```
  fd = open("out", 1);
  write(fd, "hello\n", 6);
  pid = fork();
```

* 为什么OS的设计和实现这么难但是却很有意义?
  * 环境的差异: weird h/w, 无调试器
  * 必须高效 (thus low-level?)
	...但是又要抽象和便携 (thus high-level?)
  * 强大 (thus many features?)
	...但是又要简单(thus a few composable building blocks?)
  * features interact: `fd = open(); ...; fork()`
  * 性能交互(behaviors interact): CPU priority vs memory allocator.
  * 开放性问题: 安全问题, 多核问题

* 你将会很高兴学到关于操作系统的知识 如果你...
  * 想致力于解决上面的问题
  * 想知道关于底层是怎么工作的
  * 要建立一个高性能的系统
  * 需要诊断bugs或安全问题

## 课程结构

* 课程主页: http://pdos.lcs.mit.edu/6.828

* 课程
  * OS的基本思想
  * xv6-一个传统O/S的扩展检查(extended inspection of xv6, a traditional O/S)
  * 几个最近的话题
  * 重新加深对xv6设计的理解

* 实验: JOS, 一个小的外核风格的x86 OS
  * 你要构建JOS, 还有5个实验以及你选择的最后一个实验
  * 内核接口: 暴露出硬件, 但是进行了保护 -- 不是抽象!
  * 底层库: fork, exec, pipe, ...
  * 应用程序: 文件系统, shell, ..
  * 开发环境: gcc, qemu
  * 实验1已经过时了
  * 通过make grade来进行实验评分

* 代码审查

* 两个测验: 一个在课堂上, 一个在这星期结束。

## Shell和系统调用

* 6.828是一个主要是关于系统调用接口的设计和实现的OS，我们先来看看程序如何使用这些接口,例如:
  Unix shell.
* shell是Unix的一种命令行形式的GUI接口
  * 通常要处理登陆会话，运行其它进程
  * 你可以看下6.033关于shell的一些使用: http://web.mit.edu/6.033/www/assignments/handson-unix.html
  * shell同样也是一个脚本编程语言
  * 你可以看一些简单的shell操作的例子,他们如何使用不同OS抽象接口,并将这些抽象的接口组织在一起,如果你不熟悉shell的话可以看下这个论文[Unix paper](../readings/ritchie78unix.pdf)

* [Simplified xv6 sh.c](../homework/sh.c)
  * [chapter 0 of xv6 book](../xv6/book-rev8.pdf)
  * 基本框架: 分析并执行命令 (比如: ls, ls | wc, ls > out)
  * Shell是使用系统调用实现的 (比如: read, write, fork, exec, wait)
    一些规定: 返回-1表示错误,
    错误码存在 <code>errno</code> 全局变量中,
    <code>perror</code> 根据<code>errno</code>打印出对应错误描述信息.
  * 许多系统调用都通过libc库封装成例程(比如: fgets vs read)

<!-- 
  Demo:
  - open sh.c in emacs
  - look at main()
  - look at runcmd()
  - look at fgets()
  - man 3 fgets()
  -->
  
* 跟踪系统调用 $ ls
    * 在OSX系统中: sudo dtruss ./a.out  (a.out是sh.c编译出来可执行程序)
    * 在Linux系统中: strace ./a.out

<!--
  - compile sh.c
  - run ./a.out
  - strace ./a.out
  -->

  * fork()做了什么?
    拷贝用户内存(user memory)
    拷贝进程在内核中的状态(比如: user id)
    子进程获取到一个不同的PID
    子进程的状态中包含了父进程的PID
    返回两次，分别是不同的值
  * 父进程和子进程有可能并发运行(例如: 可能同时运行在不同的处理器上).
  * wait()做了什么?
	等待任何子进程退出
  如果在父进程调用wait等待之前子进程退出了？

<!--
    - strace /bin/sh
    - study output:
    read()
    write()
    stat()
    etc.
	-->

  * 什么是文件描述符? (0, 1, 2, etc. in read/write)
  [echo.c](l-overview/echo.c)

  * 什么是i/o重定向?
  [echo.c](l-overview/redirect.c)
    在sh.c中是怎么实现重定向">"的

  * 什么是管道?  (ls | wc )
  [pipe1.c](l-overview/pipe1.c)
  [pipe2.c](l-overview/pipe2.c)
    在sh.c中是怎么实现管道的?

* 家庭作业 [shell](../homework/xv6-shell.html) 