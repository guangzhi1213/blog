---
title: "ProcessAndThread"
date: 2021-04-25T11:01:46+08:00
lastmod: 2021-04-25T11:01:46+08:00
draft: false
keywords: []
description: ""
tags: ["linux","sre","ops"]
categories: ["linux"]
author: "王清"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

## Archive for the ‘Linux’ Category



## [了解Linux的进程与线程](https://timyang.net/linux/linux-process/)

Sunday, May 3rd, 2009 by Tim | **[11 Comments](https://timyang.net/linux/linux-process/#comments)**
Filed under: [Linux](https://timyang.net/category/linux/) | Tags: [Linux](https://timyang.net/tag/linux/), [process](https://timyang.net/tag/process/), [thread](https://timyang.net/tag/thread/)

上周碰到部署在真实服务器上某个应用[CPU占用过高](http://twitter.com/xmpp/statuses/1635132615)的问题，虽然经过tuning, 问题貌似已经解决，但我对tuning的方式只是基于大胆的假设并最终生效了。我更希望更多的求证一下程序背后CPU及OS kernel当时的运作机制。所以我读了一些[Linux内核设计与实现](http://www.douban.com/subject/1446021/)及其他一些相关资料，对Linux process的机制与切换有了更多一些体会。本文尽可能条理一点，但由于牵涉点较多，同时自己可能觉得某些点有记录的价值，因此文字可能会零散。

- 进程状态

Linux进程的状态比较容易理解，值得注意的是 UNINTERRUPTIBLE 及 ZOMBIE

TASK_RUNNING
TASK_INTERRUPTIBLE
TASK_UNINTERRUPTIBLE 此时进程不接收信号，这就是为什么有时候kill一个繁忙的进程没有响应。
TASK_ZOMBIE 我们经常 kill -9 pid 之后运行ps会发现被kill的进程仍然存在，状态为 zombie。zombie的进程实际上已经结束，占用的资源也已经释放，仅由于kernel的相关进程描述符还未释放。
TASK_STOPPED

- Kernel space and user space

Kernel space是供内核，设备驱动运行的内存区域。user space是供普通应用程序运行的区域。每一个进程都运行在自己的虚拟内存区域，不能访问其他进程的内存空间。普通进程不能访问kernel space, 只能通过系统调用来间接进行。当系统内存比较紧张时，非当前运行进程user space可能会被swap到磁盘。

使用命令 pmap -x <pid> 可以查看进程的内存占用信息； lsof -a -p <pid> 可以查看一个进程打开的文件信息。ps -Lf <pid> 可以查看进程的线程数。

另外procfs也是一个分析进程结构的好地方。procfs是一个虚拟的文件系统，它把系统中正在运行的进程都显现在/proc/<pid>目录下。

- 进程创建

进程创建通常调用fork实现。创建后子进程和父进程指向同一内存区域，仅当子进程有write发生时候，才会把改动的区域copy到子进程新的地址空间，这就是copy-on-write技术，它极大的提高了创建进程的速度。

- Linux的线程实现

Linux线程是通过进程来实现。Linux kernel为进程创建提供一个clone()系统调用，clone的参数包括如 CLONE_VM, CLONE_FILES, CLONE_SIGHAND 等。通过clone()的参数，新创建的进程，也称为LWP(Lightweight process)与父进程共享内存空间，文件句柄，信号处理等，从而达到创建线程相同的目的。

Linux 2.6的线程库叫NPTL(Native POSIX Thread Library)。POSIX thread(pthread)是一个编程规范，通过此规范开发的多线程程序具有良好的跨平台特性。尽管是基于进程的实现，但新版的NPTL创建线程的效率非常高。一些测试显示，基于NPTL的内核创建10万个线程只需要2秒，而没有NPTL支持的内核则需要长达15分钟。

在Linux 2.6之前，Linux kernel并没有真正的thread支持，一些thread library都是在clone()基础上的一些基于user space的封装，因此通常在信号处理、进程调度(每个进程需要一个额外的调度线程)及多线程之间同步共享资源等方面存在一定问题。为了解决这些问题，当年IBM曾经开发一套NGPT(Next Generation POSIX Threads), 效率比 LinuxThreads有明显改进，但由于NPTL的推出，NGPT也完成了相关的历史使命并停止了开发。

NPTL的实现是在kernel增加了futex(fast userspace mutex)支持用于处理线程之间的sleep与wake。futex是一种高效的对共享资源互斥访问的算法。kernel在里面起仲裁作用，但通常都由进程自行完成。

NPTL是一个1×1的线程模型，即一个线程对于一个操作系统的调度进程，优点是非常简单。而其他一些操作系统比如Solaris则是MxN的，M对应创建的线程数，N对应操作系统可以运行的实体。(N<M)，优点是线程切换快，但实现稍复杂。

- 信号

进程接收信号有两种：同步和异步。同步信号比如SEGILL(非法访问), SIGSEGV(segmentation fault)等。发生此类信号之后，系统会立即转到内核陷阱处理程序，因此同步信号也称为陷阱。异步信号如kill, lwp_kill, sigsend等调用产生的都是，异步信号也称为中断。

kill <pid> 调用的是 SIGTERM, 此信号可以被捕获和忽略。

kill -9 <pid> 调用的是 SIGKILL, 杀掉进程，不能被捕获和忽略。

SIGHUP是在终端被断开时候调用，如果信号没有被处理，进程会终止。这就是为什么突然断网刚通过远程终端启动的进程都终止的原因。防止的方法是在启动的命令前加上 nohup 命令来忽略 SIGHUP信号。如 nohup ./startup.sh &

很多应用程序通常捕获SIGHUP用来实现一些自定义特性，比如通过控制台传递信号让正在运行的程序重新加载配置文件，避免重启带来的停止服务的副作用。可惜的是，在JAVA中没法直接使用这一功能，SUN JVM没有官方的signal支持，尽管它已经可以实现，详情可参看[Singals and Java](http://jeremymanson.blogspot.com/2007/06/signals-and-java.html).

另外有个有趣的现象是 zombie 状态的进程 kill/kill -9 都没有任何作用，这是由于进程本身已经不存在，所以没有相应的进程来处理signal, zombie状态的进程只是kernel中的进程描述符及相关数据结构没有释放，但进程实体已经不存在了。![img](http://img.zemanta.com/pixy.gif?x-id=facfcc8c-3046-8e07-aa6b-9e048a5b3124)

关于僵尸进程，也可参看下酷壳上的这篇[Linux 的僵尸(zombie)进程](http://cocre.com/?p=656)，从程序的角度解释了相关原理。
