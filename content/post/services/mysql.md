---
title: "Mysql"
date: 2021-04-25T11:11:06+08:00
lastmod: 2021-04-25T11:11:06+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# 数据库

## mysql doc

- 安装
  - [MariaDB - Setting up MariaDB Repositories - MariaDB](https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos7-ppc64le--centos7&mirror=icm&version=10.4) mariadb 官方安装帮助文档
  - [mysql list](http://repo.mysql.com/) repo list
  - [ftp日本下载源](http://ftp.jaist.ac.jp/pub/mysql/Downloads/)   虽然是日本, 但是网速很快, 各种版本的安装包,安装源
  - [数据管理 - 爱情守望者](https://www.waitsun.com/topics/code/数据管理) navicat mac 下载

- 规范
  - [社区规范](https://learnku.com/articles/25148) 表啊, 引擎啊,sql啊 什么的规范

## mysql,mariadb

关系型数据库

Mysql常用SQL语句集锦,from [juejin.im](https://juejin.im/post/584e7b298d6d81005456eb53)

### mysql 高可用

主从复制, 95.000% SLA

```
mysql 复制即一台mysql服务器从另一台mysql服务器上复制日志，然后解析成日志并应用到自身， mysql复制是单向、异步复制： 主服务器首先将更新写入二进制日志文件， 并维护一个索引以跟踪日志的循环，这些日志可以发送到从服务器进行更新， 当一个从服务器连接到主服务器时， 它从主服务器日志中读取上一次成功更新的位置，然后从服务器开始接收从上一次完成更新后发生的所有更新， 所有更新完成等待主服务器通知新的更新.

mysql 复制支持链式复制，也就是说从服务器还可以当做主服务器链接从服务器

所有表的更新必须在主服务器上完成

mysql复制优点：
1. 增加mysql应用健壮性， 当主服务器出现问题时，可以切换到从服务器继续提供服务
2. 可以将mysql读写操作分离， 可以降低mysql运行负载
3. 网络环境好，业务量不是很大同步数据很快

mysql复制方式：
1. 基于语句的复制, 把主服务器上执行的sql语句在从服务器上再执行一遍， 当发现没法准确复制时，自动选择基于行的复制
2. 基于行的复制, 把改变的行复制过去
3. 混合类型, 默认采用的方式， 当sql语句无法准确完成时， 采用行的复制

mysql复制的原理：
1. 完成复制过程主要由三个线程实现， 一个IO线程在主服务器端， 从服务器上一个IO线程和一个sql线程
2. 首先从服务器上的io线程连接到主服务器，然后请求指定日志文件的指定位置或者从最开始的日志位置之后的内容
3. 主服务器在接收到来自从服务器的io线程以后通过自身的io线程根据请求读取指定日志位置之后的日志信息， 并返回给从服务器io线程，返回信息包括(日志包含的信息， 此次返回信息在主服务器端对应的二进制日志文件的名称以及二进制日志中的位置)
4. 从服务器io线程接收到信息后将获取到的日志内容写入到从服务器的中继日志文件， 并且将读取到的主服务器的二进制日志文件名称和位置写入到master-info文件中
5. 从服务器的sql线程在检测到中继日志新增内容后， 解析成sql语句，然后在自身执行这些sql

最简单的主从复制，主节点挂掉手动切换从服务器为主节点
配合Keepalive实现故障自动转移
互为主备，双主互备模型， 正常情况下仅对db主进行数据读取写操作，db从进行复制操作； 当出现异常时keepalived将vip和mysql服务切换到db从， keepalived设置为非抢占模式，因为切换代价太大,
```

```
mysql 复制时
1. 同一时刻只能有一个主服务器进行写操作
2. 一个主服务器可以有多个从服务器
3. 无论主服务器还是从服务器id必须唯一
4. 从服务器可以将信息链式传递给其它从服务器
```


```
mysql自身提供的数据复制技术，

```
MMM高可用, 99.000% SLA


```
monitor, agent, contral三个脚本完成
1. 双master节点， 互为主从复制，需要5个ip地址， 两个master各有一个不变的物理ip， 另外还有两个只读ip和一个可写ip， vip如何切换取决于节点可用性
2. 在以上基础上再增加slave节点可实现双主多从模型， monitor会监控所有节点可用性保证服务连续性和高可用性
3. 如果活动的master节点发生故障，会自动将后端的多个slave节点转向备用的master节点继续进行同步复制，

读写分离分为两种：
1. 从程序层面处理， 读写使用不同的sqlip
2. 从软件层面处理， 进行路由, 相当于代理将不同的请求根据请求内容分发至不同节点, 从而实现读写分离和负载均衡
```

MHA, 类似，都加入了一个监控管理节点,

```
要略优于mmm
```

Heartbeat/SAN, 99.990% SLA

Hearbeat/DRBD, 99.900% SLA

Mysql Cluster 高可用集群

##
