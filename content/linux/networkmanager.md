---
title: "Networkmanager"
date: 2021-04-25T10:53:15+08:00
lastmod: 2021-04-25T10:53:15+08:00
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

# Ethernet

## 查看网卡物理设备及系统支持情况

`lspci` : 查看网卡型号

`ls -l /lib/modules/4.9.92-1-MANJARO/kernel/drivers/net/` : linux 内核所有支持的网络设备 以太网设备，虚拟化网络设备， 无线网络等； ethernet 目录下是支持的以太网设备

一般情况下 Broadcom 芯片对应的驱动程序名称类似 tg3.ko,bnx2.ko,bcm57xx; Intel 芯片对应的名称类似 e1000.ko,e10000e.ko; VIA 芯片对应的 via-rhine.ko,via-velecity.ko; Realtek 芯片对应8139cpko,8139too.ko

## 检查网卡驱动是否加载

linux操作系统对硬件的操控是通过驱动程序实现的， linux 内核采用可加载的模块化设计， 只将基本的核心代码模块编译进内核， 同时允许动态地将硬件信息通过驱动程序加载进内核

`lsmod` : 查看系统加载的模块信息， 

`modprobe 模块文件` : 把模块文件加载进内核

modprobe

`-a` : --all, 加载一组匹配的模块

`-r` : --remove, 卸载指定模块， 或者不加参数执行自动清除 

`-l` : --list, 列出所有可用模块

`-n` : --show, 仅显示要执行的操作，不实际执行

`-v` : --verbose, 显示详细信息

`modprobe -av vfat` : 加载vfat模块到内核

`modprobe -n -v vfat` : 显示vfat模块在系统中对应的模块文件

`modprobe -r -v vfat` : 卸载vfat模块

insmod/rmmod 载入/卸载系统模块，与modprobe类似, modprobe 其实是调用了这两个命令，但是modprobe会检查依赖关系

depmod 分析载入模块的相关性， 以供modprobe载入模块到内核时使用

## 编译网卡驱动

rpm/srpm 安装

安装完成后驱动包在 `/lib/modules/4.9.92-1-MANJARO/extramodules/` 目录下, 需要手动复制到 `/lib/modules/4.9.92-1-MANJARO/kernel/drivers/net/` 目录下

使用 `insmod /lib/modules/4.9.92-1-MANJARO/kernel/drivers/net/xxx.ko` 或 `modprobe xxx` 加载模块到内核

设置开机自动加载网卡驱动, 请自行百度,一种方法是开机执行脚本， 一种是写入  `/etc/modprobe.conf` 中


## 配置linux网络

根据linux发型版不同有不同的位置,配置方式也有不同, 用到百度 

centos7 `NetworkManager` 管理软件, 开始使用ip 命令代替 centos6 中 ifconfig 命令, 其实就是 ip ss 家族 和ifconfig route netstat家族

## 配置linux代理转发功能

`echo "1" > /proc/sys/net/ipv4/ip_forward` : 0 表示禁止， 1 表示开启

或者 `/etc/sysctl.conf` 添加一行 `net.ipv4.ip_forward=1` 永久生效，要立即生效使用 `sysctl -p`

## 路由

可以说是gg了

`route -n` : 查看本机路由

`ip route help` ： 查看 ip 家族的使用方法

[linux命令大全](http://man.linuxde.net/) : 不会了就去查把， 不是很难


## manage

**一、****Linux****网络属性配置**

  **1.****Linux主机接入到网络****方式**

IP/NETMASK：实现本地网络通信

路由（网关）：可以进行跨网络通信

DNS服务器地址：基于主机名的通信，Linux可以有三个DNS地址

当第一个地址本身挂了，才会查找其备用地址；若第一个地址无法解析则停止

  **2.****网络属性****配置方式**

​    **(1)****静态指定**

​       1)命令方式

​          ifcfg系列命令：

ifconfig：配置IP，NETMASK

route：配置路由相关信息

netstat：状态及统计数据查看

​          iiproute2系列命令：

ip OBJECT：

addr：地址和掩码；

link：接口

route：路由

ss：状态及统计数据查看

​          CentOS 7：nm(Network Manager)家族

nmcli：命令行工具

nmtui：text window 工具

​     hostname/hostnamectl：主机名配置

​       2) 配置文件：

RedHat及相关发行版：/etc/sysconfig/network-scripts/ifcfg-NETCARD_NAME

 DNS服务器指定配置文件：/etc/resolv.conf

本地主机名配置文件：/etc/sysconfig/network

​     注：命令配置能及时生效，但时关闭当前进程之后配置失效，为一次性配置方式

​    通过配置文件配置网络属性，无法立即生效，需要重启服务、重新加载配置文件或者重启进程

​    **(2)****动态分配：依赖于本地网络中有DHCP服务**

  DHCP：Dynamic Host Configure Procotol， 动态主机配置协议，此时不能固定IP地址

  **3.****网络接口命名**

​    **(1)****传统命名**

  以太网：eth#，例如eth0, eth1, ...

  PPP网络：ppp#， 例如，ppp0, ppp1, ...

​    **(2)****可预测命名方案****(****CentOS** **7)**

   支持多种不同的命名机制，根据Fireware, 拓扑结构等信息自动配置

1) Firmware或BIOS为主板上集成的设备提供的索引信息可用，则根据此索引进行命名，如eno1,eno2, ...

2) Firmware或BIOS为PCI-E扩展槽所提供的索引信息可用，且可预测，则根据此索引进行命名，如ens1, ens2, ...

3) 如果硬件接口的物理位置信息可用，则根据此信息命名，如enp2s0, ...

4) 如果用户显式定义，也可根据MAC地址命名，例如eno16777736(十六进制MAC), ...

5)上述均不可用，则仍使用传统方式命名；

​    **(3)****命名格式的组成**

  en：ethernet，表示因特网网卡接口

  wl：wlan，表示无线网网卡接口

  ww：wwan,Wireless Wide Area Network，表示无线广域网网卡

​    **(4)****名称类型：**

  o<index>：集成设备的设备索引号；

  s<slot>：扩展槽的索引号；

  x<MAC>：基于MAC地址的命名；

  p<bus>s<slot>：基于总线及槽的拓扑结构进行命名；

**二、****ifcfg****系列****:****fconfig, route, netstat**

**1.****ifconfig：****配置查看网络接口，默认不能显示第二地址，只能显示主地址**

**指明标签****(****接口别名****)****就能够显示了**

**(1)****ifconfig** **[INTERFACE]** **默认只会显示激活状态的网卡信息**

**# ifconfig**  **-a：显示所有接口，包括inactive****非激活****状态接口；**

注意：CentOS 6和CentOS 7显示结果有所不同

**CentOS 7****：**

**显示含义解析：**

eno1677736：网卡接口名称：

flags：标志位，UP表示网卡启用激活状态

mtu：maximum transmission unit，网卡最大传输单元为1500字节

inet：IPv4地址；  netmask：子网掩码； broadcast：广播地址

inet6：IPv6地址

HWaddr ：以太网地址，对应于CentOS 6中的HWaddr硬件地址

txqueuelen 1000 (Ethernet)：以太网传输队列长度

RX packets 7526 bytes 631299 (616.5 KiB)：此次网卡激活后接搜到的报文数量，总大小

RX errors :接收时错误的个数；dropped:丢包个数；overruns:溢出个数； frame:帧

TX packets 162 bytes 18461 (18.0 KiB)：传输的报文数量

TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0：传输的错误、丢包、溢出、帧

[![wKioL1Z_1EXAM6DsAALeJp49SKc092.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1EXAM6DsAALeJp49SKc092.jpg)](http://s2.51cto.com/wyfs02/M00/78/94/wKioL1Z_1EXAM6DsAALeJp49SKc092.jpg)

**CentOS 6****：**

eh0：网卡接口，其表现形式和CentOS 7有很大区别

HWaddr 00:0C:29:46:14:98硬件地址

[![wKiom1Z_1C_hkRGaAAQ_IjoN_e4586.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1C_hkRGaAAQ_IjoN_e4586.jpg)](http://s2.51cto.com/wyfs02/M00/78/95/wKiom1Z_1C_hkRGaAAQ_IjoN_e4586.jpg)

**(2)****ifconfig** **[-v]** **interface** **[aftype]** **options | address …** **更改网卡****IPv4****地址**

立即送往内核中的TCP/IP协议栈，并生效，远程连接修改，原来的地址会没有导致掉线

**#** **ifconfig** **I****NTER****FACE**  **IP/MASK** 

**#** **ifconfig**  **I****NTER****FACE IP**  **netmask**  **NETMASK** **:****用****netmask****关键字**

options： ifconfig  INTERFACE  OPTIONS

[-]promisc：混杂模式，-表示关闭混杂模式，直接加表示进入混杂模式

… …

管理IPv6地址：add|del addr/prefixlen

[![wKioL1Z_1Eqj8GNrAAJZ_FO5Lfk941.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1Eqj8GNrAAJZ_FO5Lfk941.jpg)](http://s3.51cto.com/wyfs02/M01/78/94/wKioL1Z_1Eqj8GNrAAJZ_FO5Lfk941.jpg)

**(3)****启用****/****关闭网卡**

**1)#** **ifconfig**  **I****NTER****FACE** **up|down**

[![wKioL1Z_1EvwIsRpAALR8i3tdyE169.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1EvwIsRpAALR8i3tdyE169.jpg)](http://s3.51cto.com/wyfs02/M01/78/94/wKioL1Z_1EvwIsRpAALR8i3tdyE169.jpg)

**2)****ifup/ifdown命令：**

注意：此命令是通过配置文件/etc/sysconfig/network-scripts/ifcfg-IFACE来识别接口并完成配置；

**(4)****删除指定接口网卡** **的地址：**

\# ifconfig INTERFACE 0

[![wKiom1Z_1DajQBbJAAQ5Y1RK78Y028.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1DajQBbJAAQ5Y1RK78Y028.jpg)](http://s5.51cto.com/wyfs02/M01/78/95/wKiom1Z_1DajQBbJAAQ5Y1RK78Y028.jpg)

 

**2.****route命令：路由查看及管理**

路由条目类型(三种)：

主机路由：目标地址为单个IP；

网络路由：目标地址为IP网络；

默认路由：目标为任意网络，0.0.0.0/0.0.0.0

**(1)****查看：**

**# route -n**

-n： 表示以数字形式显示信息，不反向解析地址和端口号

若有很多路由信息的时候，反向解析为主机名和端口名会占用很多资源开销

**显示解析：**

Destination：目标地址

Gateway：下一跳网管地址

0.0.0.0：表示本地主机的网络地址，自己的主机就在网络上无需网关，直连路由，

Genmask：目标网络的掩码地址

Flags：路由条目的标志

​       U (route is up)：up，表示启用状态

​       H (target is a host)：目标地址是一个主机地址

​       G (use gateway)：使用一个网关

​       R (reinstate route for dynamic routing)：为路由恢复动态路径选择

​       D (dynamically installed by daemon or redirect)

​       M (modified from routing daemon or redirect)

​       A (installed by addrconf)

​       C (cache entry)

​       ! (reject route)

​       G：表示是一个网关，但不一定是目标网关，只有目标地址是0.0.0.0的才是默认网关

Metric：度量值，表示到达这个网络中间要进过的开销

Ref：Number of references to this route. (Not used in the Linux kernel.)

Use：Count of lookups for the route

Iface：通过本主机的哪块网卡接口对发送数据

[![wKiom1Z_1Daz2NsFAAENz43jGM4839.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1Daz2NsFAAENz43jGM4839.jpg)](http://s3.51cto.com/wyfs02/M02/78/95/wKiom1Z_1Daz2NsFAAENz43jGM4839.jpg)

**(2)****添加：**

**route add [-net|-host] target [netmask Nm] [gw** **GW]**  **[dev] If]**

-net|-host ：网络路由| 主机路由，默认路由为网络路由

target [netmask Nm] ：目标地址，可以用简写子网掩码格式，也可以用关键字netmask完整格式

[gw GW] ：gw为关键字，GW表示真正的下一跳地址

下一跳必须与自己的某块网卡在同一网段内，且存在

[dev] If]：进由哪块网卡，可以省略，能自动判断

[![wKioL1Z_1FCiVk7PAAHRJWD3fIs059.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1FCiVk7PAAHRJWD3fIs059.jpg)](http://s2.51cto.com/wyfs02/M02/78/94/wKioL1Z_1FCiVk7PAAHRJWD3fIs059.jpg)

示例：

route add -net 10.0.0.0/8 gw 192.168.10.1 dev eth1

 oute add -net 0.0.0.0/0.0.0.0 gw 192.168.10.1 === route add default gw 192.168.10.1       

**(3)****删除：**

**route del [-net|-host] target [gw Gw] [netmask Nm] [[dev] If]**

示例： route del -net 10.0.0.0/8 gw 192.168.10.1

route del default

[![wKiom1Z_1DiDMgxkAAFFTHqEha4012.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1DiDMgxkAAFFTHqEha4012.jpg)](http://s3.51cto.com/wyfs02/M00/78/95/wKiom1Z_1DiDMgxkAAFFTHqEha4012.jpg)

 
**3.****netstat命令：****查看网络****状态及统计数据**

Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships

显示网络连接、路由表、接口连接、伪装连接和多播成员关系

**(1)****显示路由表：****#** **netstat -rn**

-r：显示内核路由表

-n：以数字形式显示信息，不反向解析地址

[![wKioL1Z_1FKxNNQuAAEjE32sk4M562.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1FKxNNQuAAEjE32sk4M562.jpg)](http://s1.51cto.com/wyfs02/M02/78/94/wKioL1Z_1FKxNNQuAAEjE32sk4M562.jpg)

**(2)****显示网络连接****信息：****# netstat  OPTIONS(****常用组合：****-tan, -uan, -tnl, -unl, -tunlp****)**

-t,--tcp：TCP协议的相关连接，连接均有其状态；FSM（Finate State Machine）；

通信开始之前，要建立一个虚链路；通信完成后还要拆除链接

-u,--udp：UDP相关的连接；无连接的协议；直接发送数据报文

-w：raw socket裸套接字相关的连接

-l：处于监听状态的连接

-a：所有状态的连接

-n：以数字格式显示IP和Port；

-e：扩展格式

-p：显示相关的进程及PID；

[![wKioL1Z_1FOjmMrwAAI6ASy56S0303.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1FOjmMrwAAI6ASy56S0303.jpg)](http://s3.51cto.com/wyfs02/M00/78/94/wKioL1Z_1FOjmMrwAAI6ASy56S0303.jpg)

tcp状态：LISTEN、ESTABLISEHD、FIN_WAIT_1等待状态、FIN_WAIT_2、SYN_SENT、SYN_RECV、CLOSED

**注意：****传输** **层协议****区别****(TCP|UDP)**

tcp：面向连接的协议；通信开始之前，要建立一个虚链路；通信完成后还要拆除连接；

udp：无连接的协议；直接发送数据报文；

**(3)****显示接口的统计数据：**

netstat {--interfaces|-I|-i} [iface][--all|-a] [--extend|-e] [--verbose|-v] [--program|-p] [--numeric|-n]

所有接口：netstat -I

[![wKiom1Z_1DuR634wAADy61_fMGo861.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1DuR634wAADy61_fMGo861.jpg)](http://s5.51cto.com/wyfs02/M02/78/95/wKiom1Z_1DuR634wAADy61_fMGo861.jpg)

指定接口：netstat -I<IFace>，注意中间不能有空格

[![wKiom1Z_1DziQtf9AADMu0a4oic030.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1DziQtf9AADMu0a4oic030.jpg)](http://s4.51cto.com/wyfs02/M01/78/96/wKiom1Z_1DziQtf9AADMu0a4oic030.jpg)

 

**4.****配置主机名****hostname****/hostnamectl****命令：**

**(1)hostname**

  查看：hostname

  配置：hostname HOSTNAME,当前系统有效，重启后无效；

​     **(2)****hostnamectl命令（CentOS 7）：****该命令会直接修改配置文件生效**

​    hostnamectl status：显示当前主机名信息；

​    hostnamectl  HOSTNAME：设定主机名，永久有效；

[![wKioL1Z_1FeSl2o8AAOlrksWinw147.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1FeSl2o8AAOlrksWinw147.jpg)](http://s1.51cto.com/wyfs02/M01/78/94/wKioL1Z_1FeSl2o8AAOlrksWinw147.jpg)

 

 

 

 

 

**三、****ip****route****系列：****ip****、****ss**

​    iproute2系列和内核关系紧密，直接放置到内核生效，其版本号和内核的版本号会保持一致

[![wKiom1Z_1EHhbqjFAAN79T6cb_Q510.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1EHhbqjFAAN79T6cb_Q510.jpg)](http://s3.51cto.com/wyfs02/M02/78/96/wKiom1Z_1EHhbqjFAAN79T6cb_Q510.jpg)

   Advanced IP routing and network device configuration tools  ：提供网络工具

[![wKioL1Z_1FuiRy31AAG2FNiPLpY659.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1FuiRy31AAG2FNiPLpY659.jpg)](http://s1.51cto.com/wyfs02/M02/78/94/wKioL1Z_1FuiRy31AAG2FNiPLpY659.jpg)

  **1.****ip命令：show / manipulate routing, devices, policy routing and tunnels** **策略路由、隧道、路由、设备**

​     ip  [ OPTIONS ]  OBJECT  { COMMAND | help }

 OBJECT := { link | addr | route | netns }

**注意： OBJECT可简写，各OBJECT的子命令也可简写；**

​    **(1)****ip** **link****：****network device configuration****,****网络设备配置**

​        **1)****ip link set** **：****change device attributes****,****修改设备属性**

dev NAME (default)：指明要管理的设备，默认配置，dev关键字可省略；

up和down：启用，禁用

multicast on或multicast off：启用或禁用多播功能；

name NAME：重命名接口

mtu NUMBER：设置MTU的大小，默认为1500；

netns PID：ns为namespace，用于将接口移动到指定的网络名称空间；

**实例：**

   修改eth1名称，注意在修改前要先停用

[root@localhost ~]# ip link set eth1 down

[root@localhost ~]# ip link set eth1 name ethtest

[root@localhost ~]# ip link show

[![wKioL1Z_1F2iUxAmAALC6DBJVds568.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1F2iUxAmAALC6DBJVds568.jpg)](http://s2.51cto.com/wyfs02/M00/78/94/wKioL1Z_1F2iUxAmAALC6DBJVds568.jpg)

​        **2)****ip link show****/list** **：****display device attributes****，显示设备属性**

[![wKiom1Z_1Eago-vgAAIgO2zlnGI138.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1Eago-vgAAIgO2zlnGI138.jpg)](http://s4.51cto.com/wyfs02/M01/78/96/wKiom1Z_1Eago-vgAAIgO2zlnGI138.jpg)

​        **3)****ip link help****：****显示简要使用帮助；**

[![wKioL1Z_1GDwEQWlAAKZ1QWjzx0829.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1GDwEQWlAAKZ1QWjzx0829.jpg)](http://s4.51cto.com/wyfs02/M00/78/94/wKioL1Z_1GDwEQWlAAKZ1QWjzx0829.jpg)

​    **(2)****ip** **netns****：****manage network namespaces.****管理网络名称空间**

​    ip netns list：列出所有的netns

​    ip netns add NAME：创建指定的netns

​    ip netns del NAME：删除指定的netns

​    ip netns  exec NAME COMMAND：在指定的netns中运行命令

​    **(3)i****p address****：** **protocol address management.**  **协议地址管理**

​        **1)****ip address add** **：****add new protocol address****，增加新的协议地址**

 **ip addr add**  **I****NTER****F****ACE****ADDR**  **dev**  **I****NTER****FACE**

[label NAME]：为额外添加的地址指明接口别名；

[broadcast ADDRESS]：广播地址；会根据IP和NETMASK自动计算得到；

[scope SCOPE_VALUE]：范围变量

global：全局可用；

link：接口可用；

host：仅本机可用；

[![wKioL1Z_1GWyFRjyAAXyyaYKrqA439.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1GWyFRjyAAXyyaYKrqA439.jpg)](http://s5.51cto.com/wyfs02/M01/78/94/wKioL1Z_1GWyFRjyAAXyyaYKrqA439.jpg)

​        **2)i****p** **address** **delete** **：****delete protocol address****，删** **除协议地址**

​      \# ip addr delete INTERFACEADDR dev IFACE

[![wKiom1Z_1E7hK-NcAAIWigD2mYU460.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1E7hK-NcAAIWigD2mYU460.jpg)](http://s5.51cto.com/wyfs02/M00/78/96/wKiom1Z_1E7hK-NcAAIWigD2mYU460.jpg)

​        **3)****ip** **address show****：****look at protocol addresses****，查看协议地址**

\# ip addr  list [IFACE]：显示接口的地址；

[![wKiom1Z_1FKjLboAAAUxaaT6PcI811.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1FKjLboAAAUxaaT6PcI811.jpg)](http://s5.51cto.com/wyfs02/M02/78/96/wKiom1Z_1FKjLboAAAUxaaT6PcI811.jpg)

​        **4)****ip address flush****：****flush protocol addresses****，删** **除指定接口上所有的的地址**

\# ip addr flush dev IFACE

[![wKioL1Z_1G2gpyTqAAMpfVxVtuA119.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1G2gpyTqAAMpfVxVtuA119.jpg)](http://s5.51cto.com/wyfs02/M01/78/94/wKioL1Z_1G2gpyTqAAMpfVxVtuA119.jpg)

​    **(4)****ip route** **：****outing table management****，管理路由表**

​        **1)**ip route add - add new route增加

​    ip route change - change route修改

​    ip route replace - change or add new one修改或者增加

​    ip route  add TYPE PREFIX via GW [dev IFACE] [src SOURCE_IP]

示例：

\# ip route add 192.168.0.0/24 via 10.0.0.1 dev eth1 src 10.0.20.100

\# ip route add default via GW                        

​        **2)**ip route delete - delete route

ip route del TYPE PRIFIX

​        **3)**ip route show - list routes

  **4)**ip route flush - flush routing tables

  **5)**ip route get - get a single route

  ip route get TYPE PRIFIX

 

  **2.****ss命令：****查看网络****状态及统计数据**  **ss [options] [ FILTER ]**

​     **(1)[OPTION]****：**

-t：TCP协议的相关连接

-u：UDP相关的连接

-w：raw socket相关的连接

-l：监听状态的连接

-a：所有状态的连接

-n：数字格式

-p：相关的程序及其PID

-e：扩展格式信息

-m：内存用量

-o：计时器信息

[![wKioL1Z_1G-wjX0eAAKt6nXWGXg208.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1G-wjX0eAAKt6nXWGXg208.jpg)](http://s4.51cto.com/wyfs02/M02/78/94/wKioL1Z_1G-wjX0eAAKt6nXWGXg208.jpg)

​     **(2)****FILTER := [ state TCP-STATE ] [ EXPRESSION ]** **状态过滤功能**

**可以过滤端口、状态等信息来查看**

EXPRESSION：

​            dport =

​             sport =

示例：'( dport = :22 or sport = :22)'

   **~]# ss  -tan  '( dport = :22 or sport = :22 )'**

[![wKiom1Z_1Faz9c33AADpD2K_lKI744.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1Faz9c33AADpD2K_lKI744.jpg)](http://s2.51cto.com/wyfs02/M00/78/96/wKiom1Z_1Faz9c33AADpD2K_lKI744.jpg)

   **~]# ss -tan state ESTABLISHED**

[![wKioL1Z_1HDRDN1gAACh2yEESSc410.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1HDRDN1gAACh2yEESSc410.jpg)](http://s2.51cto.com/wyfs02/M00/78/94/wKioL1Z_1HDRDN1gAACh2yEESSc410.jpg)

 

 

 

 

**四、****nmcli命令：**

 **nmcli [ OPTIONS ]** **OBJECT** **{ COMMAND | help }**

   **(1)****device****：** **show and manage network interfaces****显示管理网络接口**

  COMMAND := { status | show | connect | disconnect | delete | wifi | wimax }

[![wKiom1Z_1FfTwaqsAAC5vuiTPw4563.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1FfTwaqsAAC5vuiTPw4563.jpg)](http://s2.51cto.com/wyfs02/M01/78/96/wKiom1Z_1FfTwaqsAAC5vuiTPw4563.jpg)

   **(2)****connection****：****start, stop, and manage network connections****，**

  COMMAND := { show | up | down | add | edit | modify | delete | reload | load }

   **(3)****modify [ id | uuid | path ]  [+|-]. **

如何修改IP地址等属性：

\# nmcli conn modify IFACE [+|-]setting.property value

ipv4.address

ipv4.gateway

ipv4.dns1

ipv4.method

manual

 

 

**五、****配置文件：**

  **1.****IP/NETMASK/GW/DNS等属性的配置文件：/etc/sysconfig/network-scripts/ifcfg-IFACE**

​         ifcfg-IFACE：实际接口名称；

[![wKioL1Z_1HHyVEz4AAItPiTfo34649.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1HHyVEz4AAItPiTfo34649.jpg)](http://s4.51cto.com/wyfs02/M01/78/94/wKioL1Z_1HHyVEz4AAItPiTfo34649.jpg)

​    **(1)vim****编辑配置文件**

配置文件/etc/sysconfig/network-scripts/ifcfg-IFACE通过大量参数来定义接口的属性，可直接修改

**1)****fcfg-IFACE配置文件参数：**

DEVICE：此配置文件对应的设备的名称；

ONBOOT：在系统引导过程中，是否激活此接口；

UUID：此设备的惟一标识；

IPV6INIT：是否初始化IPv6；

BOOTPROTO：激活此接口时使用什么协议来配置接口属性，常用的有dhcp、bootp、static、none；

TYPE：接口类型，常见的有Ethernet, Bridge；

DNS1：第一DNS服务器指向；

DNS2：备用DNS服务器指向；

DOMAIN：DNS搜索域；

IPADDR： IP地址；

NETMASK：子网掩码；CentOS 7支持使用PREFIX以长度方式指明子网掩码；

GATEWAY：默认网关；

USERCTL：是否允许普通用户控制此设备；

PEERDNS：如果BOOTPROTO的值为“dhcp”，是否允许dhcp server分配的dns服务器指向覆盖本地手动指定的DNS服务器指向；默认为允许；

HWADDR：设备的MAC地址；

NM_CONTROLLED：是否使用NetworkManager服务来控制接口

在CentOS 6上networkManager不完善，集群、虚拟化桥接在此网络服务下无法使用

网络服务有两种：network、NetworkManager

**2)****管理网络服务：**

CentOS 6: service SERVICE {start|stop|restart|status}

CentOS 7：systemctl {start|stop|restart|status} SERVICE[.service]

   配置文件修改之后，如果要生效，需要重启网络服务；

CentOS 6：# service network restart

CentOS 7：# systemctl restart network.service

​    **(2)****专用的命令的进行修改**

 CentOS 6：system-config-network (setup)

**# setup**

[![wKiom1Z_1Fnji2pGAAFrhLpjZOA240.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1Fnji2pGAAFrhLpjZOA240.jpg)](http://s4.51cto.com/wyfs02/M02/78/96/wKiom1Z_1Fnji2pGAAFrhLpjZOA240.jpg)

**#****system-config-network**

[![wKiom1Z_1FvAk3MkAADqd3Z6YqI434.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKiom1Z_1FvAk3MkAADqd3Z6YqI434.jpg)](http://s5.51cto.com/wyfs02/M01/78/96/wKiom1Z_1FvAk3MkAADqd3Z6YqI434.jpg)

 **CentOS 7: nmtui**

[![wKioL1Z_1HSgi7iTAACJoQoZlHk483.jpg](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/070fe68d-c8f3-47ab-b9c4-494e985927eb/index_files/wKioL1Z_1HSgi7iTAACJoQoZlHk483.jpg)](http://s5.51cto.com/wyfs02/M02/78/94/wKioL1Z_1HSgi7iTAACJoQoZlHk483.jpg)

  **2.****路由的相关配置文件：/etc/sysconfig/network-scripts/route-IFACE**

​      用到非默认网关路由：/etc/sysconfig/network-scripts/route-IFACE

支持两种配置方式，但不可混用；

(1) 每行一个路由条目： TARGET via GW

(2) 每三行一个路由条目：

ADDRESS#=TARGET

NETMASK#=MASK

GATEWAY#=NEXTHOP

  **3.****给接口配置多个地址：**

​    **(1)ip addr** **add INTERFACEADDR dev** **I****NTER****FACE** **label LABELNAME**

​    **(****2****)ifconfig IFACE_LABEL IPADDR/NETMASK**

IFACE_LABEL： eth0:0, eth0:1, ...

​    **(****3****)为别名添加配置文件；**

DEVICE=IFACE_LABEL

BOOTPROTO：网上别名不支持动态获取地址；

static, none

  **4.hostname****配置文件：/etc/sysconfig/network**

 命令：HOSTNAME=<HOSTNAME>

注意：此方法的设置不会立即生效； 重读配置文件或者重启系统后后会一直有效；

  **5.****配置DNS服务器指向：**

​     配置文件：/etc/resolv.conf，添加 nameserver  DNS_SERVER_IP

  **6.****/etc/hosts 别名，名称解析，事先生效，先查看此文件**