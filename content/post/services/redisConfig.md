---
title: "RedisConfig"
date: 2021-04-25T11:17:38+08:00
lastmod: 2021-04-25T11:17:38+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---


[redis中文文档](http://www.redis.cn/documentation.html)  
[redis官方文档](https://redis.io/documentation)

## 调整系统参数

### 调整overcommit_memory参数

`/proc/sys/vm/overcommit_memory` 内存分配策略，可以是 0, 1, 2  

0，表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。  
1，表示内核允许分配所有的物理内存，而不管当前的内存状态如何。  
2，表示内核允许分配超过所有物理内存和交换空间总和的内存  

> Redis在dump数据的时候，会fork出一个子进程，理论上child进程所占用的内存和parent是一样的，比如parent占用的内存为 8G，这个时候也要同样分配8G的内存给child, 如果内存无法负担，往往会造成redis服务器的down机或者IO负载过高，效率下降。所以这里比较优化的内存分配策略应该设置为 1(表示内核允许分配所有的物理内存，而不管当前的内存状态如何)。

**Background save may fail under low memory condition**

`echo 1 > /proc/sys/vm/overcommit_memory`

`echo "vm.overcommit_memory=1" >> /etc/sysctl.conf && sysctl -p` 

**The TCP backlog setting of 511 cannot be enforced** 

`echo 511 > /proc/sys/net/core/somaxconn`

`echo "echo 511 > /proc/sys/net/core/somaxconn" > /etc/rc.local`

## 调整 redis 配置文件参数


daemonize yes #是否作为守护进程运行

pidfile redis.pid #如以后台进程运行，则需指定一个pid，默认为/var/run/redis.pid

save 900 1 #当有一条Keys数据被改变是，900秒刷新到disk一次

save 300 10 #当有10条Keys数据被改变时，300秒刷新到disk一次

save 60 10000 #当有1w条keys数据被改变时，60秒刷新到disk一次

rdbcompression yes #当dump .rdb数据库的时候是否压缩数据对象

rdbchecksum yes #存储和加载rdb文件时校验

dbfilename dump.rdb #本地数据库文件名，默认值为dump.rdb

stop-writes-on-bgsave-error yes #后台存储错误停止写。

dir /var/lib/redis/ #本地数据库存放路径，默认值为 ./

## redis 配置文件 Replication 

slaveof <masterip> <masterport> #当本机为从服务时，设置主服务的IP及端口

masterauth <master-password> #当本机为从服务时，设置主服务的连接密码

requirepass foobared #连接密码， 服务端

##  redis-benchmark 压力测试工具

redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20
redis-benchmark -t set -n 1000000 -r 100000000
redis-benchmark -t ping,set,get -n 100000 –csv
redis-benchmark -r 10000 -n 10000 lpush mylist 
