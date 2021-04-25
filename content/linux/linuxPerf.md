---
title: "LinuxPerf"
date: 2021-04-25T10:56:01+08:00
lastmod: 2021-04-25T10:56:01+08:00
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

## linux 性能分析

from [linux performance](http://www.brendangregg.com/linuxperf.html)



### 系统硬件资源

cpu

内存

磁盘I/O性能

网络

### 操作系统资源

系统安装时，磁盘分区，交换分区内存的分配

内核参数优化, web服务器: net.ipv4.ip_local_port_range, net.ipv4.tcp_tw_reuse, net.core.somaxconn 等网络参数

文件系统优化, xfs, ext4,



![](/images/linux_performance_tools.png)

## 一、 observability tools 性能观测工具

![](/images/linux_performance_observability_tools.png)

## 二、benchmarking tools 性能测评工具

![](/images/linux_performance_benchmark_tools.png)

##  三、 tuning tools 性能调优工具

![](/images/linux_performance_tuning_tools.png)

## 四、 observability sar sar工具

![](/images/linux_performance_observability_sar.png)