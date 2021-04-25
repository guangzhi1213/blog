---
title: "OperatingSystem"
date: 2021-04-25T11:00:49+08:00
lastmod: 2021-04-25T11:00:49+08:00
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

**一.计算机的基本组成**

  **1.CPU****：**运算器、控制器、寄存器、缓存等组成

  早期采用Poll轮询机制,每隔一定时间询问设备，浪费cpu资源

  后来采用Interput中断机制，硬件通知机制，外围设备通过不断中断来和CPU核心设备交互

​     但中断过多会导致系统性能下降，如网卡访问量过大

运算器、控制器：核心部件

寄存器、缓存：加速部件，为了提高CPU的性能

**CPU架构**：

x86: 32位架构

32位：32条路，每条1/0两个选择 ，有2^32种方法选址，最多只能使用大约4G内存

x64(amd64) ：最早64位CPU是AMD公司发布，后来Intel。向老版本兼容，新指令

m68000：摩托罗拉CPU

arm系列：arm公司只提供架构图，不生产cpu，

各厂商买回去自己生产或者二次开发功耗低，手机端cpu

ultrasparc ： Solaris系统

power：功耗体积大，精简指令集。AIX

powerpc：简称ppc，简装版power，用于早期的Apple的CPU架构

MIPS：

alpha：HP-UX系统

​     交叉编译：cross compile，在一个架构上编译适用另一个硬件架构的Applaction的方法

  **2.** **存储器：**内存，RAM(Random Access Memory)是编址单元

  **3.****Input：**下指令，提供数据等；

  **4.****Output：**输出数据加工的结果；

  **5.****主板**

  北桥：高速总线控制器，，一般接CPU和内存

 南桥：I/O设备控制

 

 

 

**二.操作系统发展史**

 ENTARC：第一台计算机

 **批处理系统**：job1$job2$jiob3$......

 **多任务**：multi tasks  --->Bell，MIT，GE三个组织=Multics

CPU：slice机制，切换任务运算

Memory：分段机制；虚拟地址空间

 **贝尔实验室的ken Thompson --->在PDP-7上开发**

DEC：PDP-11，VAX(VMS)流行，贵 --->Ken在PDP-7上开发

--->1969:Unics对立 = Unix --->Unix：1971.norff

--->1972 Bell实验室有十台使用unix

--->B语言-Dennis Ritch-C语言 ，两人用c语言改写了unix

 从汇编到c可移植性增强，但是在当时的计算机性能差方面问题冒险

--->联合发表在《美国计算机通信》：1974年，第一次公之于众

--->1979年：System V7 比较流行

--->1978年，SCO包装发行unix

--->1988: Microsoft ,XENIX

--->Berkrlry ：Ken 任教伯克利大学

**Bill Joy.组织BSRG。1977年发布BSD（Berkrlry System Distribution）**

--->1980年，DARPA，在BSD系列的unix上研究tcp/ip

--->在版权官司十年unix逐渐落末

 **1981.Microsoft，Bill Gates**

SCP ：QDOS（Quick and Dirty Operating System）

DOS 2.0，性能价格比CP/M更好

Windows（支持图形化）

  windows nt（new technology）

1990.一直在unix上编写DOS

 **SUN公司**:Bill Joy  workstation工作站

 **Apple：**XEROX施乐公司： PARK实验室（star产品：图像化界面） 

---->Bill Gates"盗窃"后开发出最早的windows

 **1985：Richard Stallman**

GUN： GUN is Not Unix

 GLP：General Public License

 FSF：Free Software Foundation  free:freedom自由的

软件方面：X-Window: GPL

gcc: gnu c complier

vi: visual interface

... ...        

 **Andrew： Minix，早起4000+行代码**    

System V Unix <---> BSD

--->1990：BSD --->Jolitz 将BSD移植到x86

 **1991年8月：Linux Torvalds宣布成立Linux;准守GPL协定**

--->基于Larry Wall作者 diff编写补丁和patch打补丁工具运用，协同开发

 完整的OS：Kernel+Application -> GUN／Linux

狭义的OS：Kernel    

 

 

**三.操作系统在硬件上的实现**

\1. Syscall系统调用接口(硬件上内核层接口)

\2. libcall库调用（将通用功能在系统接口再封装成模块方便统一功能调用）

\3. 操作系统：隐藏底层硬件复杂性，差异性Sysacall--->再封装--->libcall

POSIX: Portable Operating System Interface 可移植操作系统接口标准

API: Application Program Interface 程序员面对的编程接口

ABI: Application Binary Interface 程序应用者面对运行程序的接口

编程接口兼容不等于二进制接口兼容
