---
title: "主机扫描"
date: 2021-04-21T17:58:04+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]
---

## 主机扫描

### fping

支持批量扫描

`fping IP1 IP2`

```shell
  options:
  -g  支持主机段的方式： 192.168.1.1 192.168.1.255 OR 192.168.1.0/24
  -f  fping -f filename ; 从文件中读取
  -a  只显示存活的
  -u  哪些没有存活
```

## 端口扫描

### hping

host屏蔽icmp探测，支持tcp数据包组装、分析工具

* 对指定端口发起tcp探测

`hping -p PORT -S`

`-s` 设置tcp模式SYN包

* 伪造来源ip,模拟ddos攻击

`-a IP`: 伪造ip地址

```shell
修改内核参数：临时有效
sysctl -w net.ipv4.icmp_echo_ignore_all=1
```

```shell
测试：
hping -p 61493 -S 192.168.59.61 可以hping通，ping不通了
```

```shell
example1:
hping --scan 1-30,70-90 -S www.target.host

```


## 提权

## 路由检测：

* traceroute

```shell
命令： traceroute www.imooc.com -nT -p80

***图2-1***
发送的udp的报文，默认使用大于3万端口的报文；
ttl： 生成时间是1；
使用tcp协议  -T -p   发送tcp协议  -p指定端口
使用icmp协议 -I   大写I
-n 禁止转换为hostname方式，使用ip显示
使用icmp，udp 方式显示不出来最终结果，但使用80端口的话就可以了，因为80端口肯定是开放的，可能是针对iicmp屏蔽了；
```

* mtr


## 组合工具

目的1批量主机存活扫描 目的2针对主机服务扫描

### nmap：

```shell
扫描类型：
  icmp：-sP                ping扫描
  tcp syn： -sS         tcp半开放扫描
  tcp connect： -sT  tcp全开放扫描
  udp： -sU               udp协议扫描
  nmap -sP 192.168.0.1/24

选择端口范围:
  -p 0-30000；默认扫描前1000
```

### ncat

```shell
-w 设置超时时间
-z  一个输入输出模式
-v  显示命令执行过程

基于tcp协议（默认）
nc -v -z -w2 192.168.59.62 1-65535

基于udp协议
nc -v -u -z -w2 192.168.59.61 1-65535
```
