---
title: "Security"
date: 2021-04-21T17:58:49+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]
---

## 主要介绍一些安全方面的知识

[https://github.com/SecWiki/sec-chart.git](https://github.com/SecWiki/sec-chart.git)

被入侵后 可能更改sshd服务, 基本上服务器就算是不安全的了,还是重装

### 哪里是安全的重点?

当你在关注安全的时候，不应该关注重点，因为每个地方都是重点。很多大的漏洞，不仅仅来源于软件,这两年常常发现是硬件出了问题。每个人都会犯错误，写出Bug，因此要去解决安全问题，要去做多层(layer) 的安全防护。即使一层出了问题，其他层可能可以解决。安全确实是很严重的问题，每个人都必须关注它。

### Kernel 的性能发展?

A:当硬件发展变缓，确实软件行业的压力是很大的。我们不能再只以来硬件的高速发展，内部在不断优化性能和项目。

### 有什么可以教育其余的开源项目的吗?例如

#### Kubernetes?

A:不要去伤害你的用户! ! !要向你的开发者证明你是一个stable platform (稳定的平台)。Kubernetes 是一个非常年轻的项目，需要建立一个拥有同样文化的社区，以同样的目标去努力。

#### 对于众多不同的domian (企业、游戏、用户端...)有不同的统治性的底层平台，你怎么看?

A:我不在意不同平台谁赢了，我在意我做的技术是最好的。在很多行业，Being First 比Being Best要重要,但是我宇仝不在音_汶不是我的业趣所在.我杀塑在无关干草



- [安全脉搏](https://www.secpulse.com/)
- [网络安全空间https://fofa.so/](https://fofa.so/) 搜索网站是否有漏洞? 其实看着厉害, 但是看不懂
- [全球共计实时地图](https://cybermap.kaspersky.com/) 好看, 看到谁在攻击谁
- [neusoft安全管理](https://isecurity.neusoft.com/book/83.html) 安全加固checklist,666, 漏洞预警, 法律法规,内网安全
- [freebuf 安全](https://www.freebuf.com/) 漏洞纰漏, web安全