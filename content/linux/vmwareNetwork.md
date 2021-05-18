---
title: "VmwareNetwork"
date: 2021-04-25T11:04:28+08:00
lastmod: 2021-04-25T11:04:28+08:00
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

# 网关制作

## 因为换了个地方租然后以前在自己电脑上装了几个虚拟机作为kubernetes的练习材料，这边的路由器ip又还了， 又不想换虚拟机的ip， 怕出现什么奇怪的问题， 正好趁这个机会学习一下制作网关

- k8s-1: 192.168.1.51/24, gateway: 192.168.1.54
- k8s-2: 192.168.1.52/24, gateway: 192.168.1.54
- k8s-3: 192.168.1.53/24, gateway: 192.168.1.54
- archlinux-routing:
  - ens32: 172.16.1.102/16, gateway: 路由器设备地址
  - ens34: 192.168.1.54/24, 不配置网关

其实就是又起了一个虚拟机，狠狠的在吃我的内存， 是以前练习用的一个archlinux系统， 里边装了docker 然后就把我耽误了一上午， 下午才弄好， docker装好后，会加几条iptables规则， 我看着好像跟我的没什么关系， 就没有管， 最后不行了才狠心删了docker， 应为本来也不怎么用

然后重启了iptables规则就干净了， `sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o ens34 -j MASQUERADE` 再加上自己的规则， 就好用了

总的来说就是在 vmware 上创建了一个新的虚拟网络设备 vmnet2 host only 模式， dhcp 改成192.168.1.x/24 与原先的网络一样的状态， 然后将三台 k8s 设备的网卡改成 vmnet2 这个， 这样据说可以 ping 通 `192.168.1.1` 物理机 vmnet2 的ip地址， 没成功， 但是主机可以用scrt连上这几个设备， 这几个设备也是可以互相连通的， 但是现在还不能上网， 并且网关还是 `192.168.1.1` 

然后再打开另一个准备当作网关的虚拟机， `sudo echo 1 > /proc/sys/net/ipv4/ip_forward` 需要切换到root用户执行， 就算是用sudo也不行， 作用就是开通内核转发， 就是可以让网卡一 和网卡二 连通ing，  加入开机启动项， 

`sudo vim /usr/lib/systemd/system/rc-local.service`

填入一下内容：

```shell
[Unit]
Description="/etc/rc.local Compatibility" 
#Wants=sshdgenkeys.service
#After=sshdgenkeys.service
#After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardInput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

创建 `/etc/rc.local` 文件

```shell
sudo touch /etc/rc.local
sudo chmod +x /etc/rc.local
```

将 `echo 1 > /proc/sys/net/ipv4/ip_forward` 填入其中， 加入开机启动项中 `sudo systemctl enable rc-local.service`

再设置iptables 进行流量转发就行 `sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o ens34 -j MASQUERADE`

保存iptables规则 `iptables-save > /etc/iptables/iptables.rules`

修改k8s那三台虚拟机的默认网关到 当作网关的虚拟机的内网网卡ip地址
