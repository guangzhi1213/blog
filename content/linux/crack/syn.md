---
title: "Syn"
date: 2021-04-25T10:49:37+08:00
lastmod: 2021-04-25T10:49:37+08:00
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

## syn攻击

* 减少发送 sync+ack 包时重试次数, `sysctl -w net.ipv4.tcp_synack_retries=3`

##  SYN类型DDOS攻击预防

* 减少发送 sync+ack 包时重试次数 `sysctl -w net.ipv4.tcp_syncack_retries=3; sysctl -w net.ipv4.tcp_synretries=3`
* SYN cookies 技术 `sysctl -w net.ipv4.tcp_syncookies=1`
* 曾家 backlog 队列 `sysctl -w net.ipv4.tcp_max_syn_backlog=2048`

## Linux 下 其他预防策略

### 策略1 

`sysctl -w net.ipv4.icmp_echo_ignore_all=1`

### 策略2

`iptables -A FORWARD -p tcp -syn -m limit --limit 1/s --limit-burst 5 -j ACCEPT`

`iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT`

`iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT`


