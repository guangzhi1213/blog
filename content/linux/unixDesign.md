---
title: "UnixDesign"
date: 2021-04-25T10:59:14+08:00
lastmod: 2021-04-25T10:59:14+08:00
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

1.Unix开发基于Multics分时操作系统

2.NIH（Not invented here，非我发明）

3.GPL：GUN公共授权协议，适用于软件的法律协议。开源

4.Unix哲学：

1）小即是美：易理解、维护、低消耗系统资源、易于其他工具结合

2）让每一个程序制作好一件事

3）尽快建立原型（prototyping）：”第三个系统”概念

4）舍高效而取可移植性

5）使用纯文本文件来存储数据：二进制严格禁止

6）充分利用软件的杠杆效应:借用代码模块;将一切自动化

7）使用shell脚本来提高杠杆效应和可移植性

8）避免强制性的用户界面

9）让每一个程序都成为过滤器

5.Unix信条

1）允许用户制定环境：程序应该只是解决问题的机制，而不是限定标准

2）尽量使操作系统内核小而轻巧

3）使用小写字母并保持简短

4）保护树木：在线存储

5）沉默是金：在需要提供出错信息时候，unix命令不提示

6）并行思考：大多数的任务能分解成更小的子任务，并行运行----对称处理(SMP)设计

7）各部分值大于整体：可集合小程序代替大程序，灵活实用

8）寻找90%的解决方案：完成90%会更有效节省成本，完美很难

9）更坏就是更好：包容

10）层次化思考：目录结构

6.MIPS度量法：每分钟执行上万条指令，衡量CPU性能的流行方法

7.IDE（interactive development environment）互动式开发环境

8.微优化（micro-optimizations）：prof和其他攻击来定位使用的最频繁的子程序优化unix下的C语言

9.Unix中常用工具和功能上说明----每个命令其实就是一个工具功能：

1）awk：对以字段组织的文本进行操作

2）expand：将制表符转换成空格

3）wc：计算文件行数、字数和字节数

4）sed：非互动形式的文本编辑器

5）roff：综合性文本格式化和排版设置工具

6）tset：比较两个字符串是否相同；检查文件的模态，了解它们是否可写

\10. Unix将数据存储为文本形式，然后使用不同的面象文本的小型工具来对数据处理

\11. Shell脚本由一个或多个语句组成，通过调用本地程序、解释程序和其他的shell脚本来执行任务。将每条命令加载到内存执行，间接调用这些产程序。

​    Shell脚本集成他人的努力成果满座自己的目标。

​    Shell解释性语言，思考---编辑---测试

​    内核里不能使用shell脚本。

\12. Unix哲学的优势之一就是它很重视数量众多的小命令，shell脚本是一种将他们统一在一起成为一个强大整体媒介

13.CUI：一种与应用程序进行交互的模式，位于系统最高级命令解释器之上。一旦在命令解释器调用一个程序，那么知道退出之前你都无法再与命令解释器进行交互。实际效果就是你完全被这个应用程序的用户界面牵扯，直到退出之后才能获得自由。

\14. Unix特点：简洁性、正确性、一致性、完整性

15.VMS：闭源专有操作系统，DEC公司。基本信念：用户害怕计算机

16.VMS系统确实是Unix的对立面。

VMS通常只给用户提供单一化的解决路径，Unix会提供十几个甚至更多的解决方法;

VMS系统喜欢采用有着多个选项、规模宏大的单一化程序来满足众多用户需求，Unix小即是美，每一个都执行单一功能且只有为数不多的选项；

VMS最初采用汇编语言和BLISS-32，与底层的硬件结构高度相关，Unix采用C语言，并可移植到许多CPU架构

17.MS-DOS：为公众设计的操作系统，易于使用。简洁有效的命令语言。没有提供真正的多任务功能，不管命令行输入多少条命令，它一次只能执行一条

18.Windows：设计思想更易于新手使用

\19. Windows的图形用户界面与底层操作系统紧密集成在一起；Linux中的X Windows System与Windows却有着本质的区别：它只不过是运行在操作上的应用程序

20.几乎所有的Perl(Practical and Report Language 实用型摘录与报告语言)程序都能充当过滤器：非常善于和其他软件交互、晦涩难用、可扩展性、开源脚本工具。
