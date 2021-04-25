---
title: "Filesystem"
date: 2021-04-25T10:54:40+08:00
lastmod: 2021-04-25T10:54:40+08:00
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

**文件系统层标准**

   **FHS****：文件系统层级结构标准****(Filesystem Hierarchy Standard)**

/bin：所有用户可用的基本命令程序文件;

/sbin：供系统管理使用的工具程序;

/boot：启动相关文件，引导加载器必须用到的各静态文件：kernel, initramfs(initrd), grub等;

/dev：存储特殊文件或设备文件；字符设备(线性设备)、块设备(随机设备);

/etc：系统程序的配置文件，只能为静态;

/home：普通用户的家目录集中位置，用户名同名子目录，/home/USERNAME;

/root：管理员的家目录;

/lib：为系统启动或根文件系统上的应用程序提供共享库，以及为内核提供内核模块;

  libc.so.*：动态链接的C库;

  ld*：运行时链接器/加载器;

  modules：用于存储内核模块的目录;

/lib64：64位系统特有的存放64位共享库的路径;

/media：便携式设备挂载点;

/mnt：其它文件系统的临时挂载点;

/opt：附加应用程序的安装位置；可选路径;

/srv：当前主机为服务提供的数据;

/tmp：用于存储临时文件的目录；可供所用户执行写入操作；**有特殊权限**;

/usr：usr Hierarchy，全局共享的只读数据路径;

bin, sbin：用户和管理员命令程序

lib, lib64：共享库文件

include：C程序头文件;

share：命令手册页和自带文档等架构特有的文件的存储位置;

X11R6：X-Window程序的安装位置;

src：程序源码文件的存储位置;

​      local：Local hierarchy，本地层级目录;

让系统管理员安装本地应用程序；也通常用于安装第三方程序;

/var：/var Hierarchy，存储常发生变化的数据的目录;

cache    Application cache data 应用缓存数据

lib    Variable state information易变的状态数据

local    Variable data for /usr/loca l可变化 /usr/local下的数据

lock    Lock files 锁文件

log    Log files and directories 日志文件和目录

opt    Variable data for /opt 可变化/opt下的数据

run    Data relevant to running processes 运行进程相关的数据

spool    Application spool data应用队列信息，如例行性计划，邮箱服务器等数据

tmp    Temporary files preserved between system reboots系统重启保存的临时文件    

/proc：基于内存的虚拟文件系统，存储内核及进程其相关信息；内核参数

例如net.ipv4.ip_forward, 虚拟为net/ipv4/ip_forward, 存储于/proc/sys/

  其完整路径为/proc/sys/net/ipv4/ip_forward;

/sys：sysfs虚拟文件系统提供了一种比proc更为理想的访问内核数据的途径；

为管理Linux设备提供一种统一模型的的接口;

硬件设备相关属性映射文件