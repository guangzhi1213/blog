---
title: "Azkaban"
date: 2021-04-25T11:06:44+08:00
lastmod: 2021-04-25T11:06:44+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

学习资料:

- [三种模式-部署](https://blog.csdn.net/wangpei1949/article/details/79521722)
- [azkaban详细介绍-使用](https://blog.csdn.net/clypm/article/details/79076801)

## 沙雕 azkaban

你知道为什么没人用吗? 因为你 ` 48 azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus` web server 配置的太沙雕了, 

`MinimumFreeMemory` 配置了内存小于6Gb的机器就不调度了???, 扎心扎了两天了, 果然还是最少配置比较好, 可能这个项目就是大公司用的

## azkaban.properties

```yaml
#启用multiple-executor模式
azkaban.use.multiple.executors=true
#在每次分发job时，先过滤出满足条件的executor，然后再做比较筛选
#如最小剩余内存,MinimumFreeMemory,过滤器会检查executor空余内存是否会大于6G，如果不足6G，则web-server不会将任务交由该executor执行。可参考Azkaban Github源码
#如CpuStatus，过滤器会检查executor的cpu占用率是否达到95%，若达到95%，web-server也不会将任务交给该executor执行。可参考Azkaban Github源码。
#参数含义参考官网说明http://azkaban.github.io/azkaban/docs/latest/#configuration
#由于是虚拟机，不需要过滤，只需要比较即可
#azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
#某个任务是否指定了executor id
azkaban.executorselector.comparator.NumberOfAssignedFlowComparator=1
#内存
azkaban.executorselector.comparator.Memory=1
#最后一次被分发
azkaban.executorselector.comparator.LastDispatched=1
#CPU
azkaban.executorselector.comparator.CpuUsage=1
```

## 部署

编译安装

下载一个版本的源码下来, 提供了gradlew命令行, 直接编译就行

 编译: `./gradlew build installDist -x test `, 完成后的包在 `build/distributions/` 目录下

我们部署时 需要 azkaban-web-server 和 azkaban-exec-server, 一个网页页面, 一个执行器

修改配置文件: 1. exec 修改时区为 `Asia/Shanghai`, 2. 指定 webserver, 3. 修改 `azkaban.executorselector.filters` 把memory去掉,否则内存小于6Gb的不会调度到.

然后自己玩吧.
