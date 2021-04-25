---
title: "Linux下多网卡绑定bond及模式介绍"
date: 2021-04-25T11:03:20+08:00
lastmod: 2021-04-25T11:03:20+08:00
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

# Linux下多网卡绑定bond及模式介绍

 2019-07-04 [原文](https://www.bbsmax.com/link/S0U1UTNCMzB6TA==)

【介绍】

网卡bond一般主要用于网络吞吐量很大，以及对于网络稳定性要求较高的场景。

主要是通过将多个物理网卡绑定到一个逻辑网卡上，实现了本地网卡的冗余，带宽扩容以及负载均衡。

Linux下一共有七种网卡bond方式，实现以上某个或某几个具体功能。

最常见的三种模式是**bond0**，**bond1**，**bond6**.

【**bond0**】

**平衡轮循环策略**，有自动备援，不过需要"Switch"支援及设定。

**balance-rr**（Round-robin policy）

方式：

传输数据包的顺序是依次传输（即：第一个包走eth0，第二个包就走eth1……，一直到所有的数据包传输完成）。

优点：

提供负载均衡和容错能力。

缺点：

同一个链接或者会话的数据包从不通的接口发出的话，中间会经过不同的链路，在客户端可能会出现**数据包无法有序到达**的情况，而无序到达的数据包将会被要求重新发送，网络**吞吐量反而会下降**。

【**bond1**】

**主-备份策略**

**active-backup**（Active -backup policy）

方式：

只有一个设备处于活动状态，一个宕掉之后另一个马上切换为主设备。

**mac地址为外部可见**，从外面看，bond的mac地址是唯一的，switch不会发生混乱。

优点：

提高了网络连接的可靠性。

缺点：

此模式**只提供容错能力**，**资源利用性较低**，只有一个接口处于active状态，在有N个网络接口bond的状态下，利用率只有1/N。

【bond2】

**平衡策略**

**balance-xor**（XOR policy）

方式：

基于特性的Hash算法传输数据包。

缺省的策略为：(源MAC地址 XOR 目标MAC地址) % slave数量。 # XRO为异或运算，值不同时结果为1，相同为0

可以通过**xmit_hash_policy**选项设置传输策略。

特点：

提供负载均衡和容错能力。

【bond3】

广播策略

broadcast

方式：

在每个slave接口上传输每一个数据包。

特点：

提供容错能力。

【bond4】

IEEE 802.3ad 动态链接聚合

**802.3ad**（ IEEE 802.3ad Dynamic link aggregation）

方式：

创建一个聚合组，**共享同样的速率和双工设定**。

根据802.3ad规范将多个slave工作在同一个激活的聚合体下。外出流量的slave选举基于传输Hash策略，同样，此策略也可以通过xmit_hash_policy选项进行修改。

注意：

**并不是所有的传输策略都是802.3ad所适应的**。

条件：

\1. ethtool支持获取每个slave的速率和双工设定。

\2. switch支持IEEE 802.3ad Dynamic link aggregation（大多数交换机需要设定才支持）

【bond5】

适配器传输负载均衡

**balance-tlb**（Adaptive transmit load balancing）

方式：

在每个slave上根据当前的负载（依据速度）分配外出流量，接收时使用当前轮到的slave。

如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

条件：

ethtool支持获取每个slave的速率。

特点：

不需要任何特别的switch(交换机)支持的通道bonding。

【**bond6**】

**适配器适应性负载均衡**

**balance-alb**（Adaptive load balancing）

方式：

此模式**包含了bond5的balance-tlb**，同时增加了针对IPV4流量的**接收负载均衡**。（receive load balance， rlb）

接收负载均衡是通过**ARP协商**实现的。

bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

来自服务器端的接收流量也会被均衡。

当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。

当ARP应答从对端到达时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。

使用ARP协商进行负载均衡的一个问题是：

每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。

这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。

当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。

接收的负载被顺序地分布（round robin）在bond中最高速的slave上当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。

下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答不会被switch(交换机)阻截。

条件：

\1. ethtool支持获取每个slave的速率

\2. 底层驱动支持设置某个设备的硬件地址

特点：

总是有一个slave（curr_active_slave）**使用bond的硬件地址**，同时每个bond里面的slave都有一个**唯一的硬件地址**。

如果curr_active_slave出了故障，则它的硬件地址会被**重新选举**产生的slave接管。

与bond0最大的区别在于，bond0的多张网卡里面的**流量几乎是相同**的，但是bond6里面的流量是先占满eth0，再占满eth1……依次

【网卡绑定】

我们假定前条件：

2个物理网口eth0，eth1

绑定后的虚拟口为bond0

服务器IP为10.10.10.1

配置文件：

\1. vi /etc/sysconfig/network-scripts/ifcfg-bond0

DEVICE=bond0BOOTPROTO=noneONBOOT=yesIPADDR=10.10.10.1NETMASK=255.255.255.0NETWORK=192.168.0.0

\2. vi /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0BOOTPROTO=noneMASTER=bond0SLAVE=yes

\3. vi /etc/sysconfig/network-scripts/ifcfg-eth1

DEVICE=eth1BOOTPROTO=noneMASTER=bond0SLAVE=yes

修改modprobe相关设定文件，并加载bonding模块：

\1. vi /etc/modprobe.d/bonding.conf

alias bond0 bondingoptions bonding mode=0 miimon=200

\2. 加载模块

modprobe bonding

\3. 确认是否加载成功

[root@slb ~]# **lsmod | grep bonding**

bonding 100065 0

\4. 重启网络

[root@slb ~]# **/etc/init.d/network restart**

[root@slb ~]# **cat /proc/net/bonding/bond0**

Ethernet Channel Bonding Driver: v3.5.0 (November 4, 2008)

Bonding Mode: fault-tolerance (active-backup)

Primary Slave: None

Currently Active Slave: eth0

……

[root@slb ~]# **ifconfig |grep HWaddr**

bond0 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74

eth0 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74

eth1 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74

以上信息可以确认：

a. 现在的**bonding模式**是active-backup

b. 现在**Active的网口**是eth0

c. bond0, eth1的物理地址和处于active状态下的eth0的物理地址相同，这样是为了避免上位交换机发生混乱。

可以随意拔掉一根网线或者在交换机上shutdown一个网口，查看网络是否依旧联通。

\5. 系统启动自动绑定并增加默认网关（**可选**）

[root@slb ~]# **vi /etc/rc.d/rc.local**

ifenslave bond0 eth0 eth1

route add default gw 10.10.10.1

【多网卡绑定】

上面只是两个网卡绑定一个bond0的情况，如果我们要设置多个bond口，就不能这样做了。

·/etc/modprobe.d/bonding.conf·的修改可以如下：

\1. 多个bond的模式一样的情况

alias bond0 bondingalias bond1 bondingoptions bonding max_bonds=2 miimon=200 mode=1

\2. 多个bond的模式不一样的情况

alias bond0 bondingoptions bond0 miimon=100 mode=1install bond1 /sbin/modprobe bonding -o bond1 miimon=200 mode=0 install bond2 /sbin/modprobe bonding -o bond2 miimon=100 mode=1
