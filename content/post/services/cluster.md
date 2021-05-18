---
title: "Cluster"
date: 2021-04-25T11:07:00+08:00
lastmod: 2021-04-25T11:07:00+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

[TOC]

集群: 集群是一个协同工作的服务集合， 用来提供比单一服务更稳定更高效更具可扩展性的服务平台。

一般由两个或多个服务器组成， 集群节点间可以相互通信， 通信方式: 1. 基于RS232的心跳监控 2. 用一块单独的网卡来跑心跳； 其中一台机器宕机后会自动踢出，以保证服务可靠性

一个集群系统必须拥有共享的数据存储， 因为集群对外提供的服务是一致的。

# 集群的分类

1. 高可用集群
1. 负载均衡集群
1. 分布式计算集群

## keepalived

功能: 依靠vrrp协议实现高可用功能， 可以根据第3-5层的交换机制检测每个服务节点的状态

VRRP: 虚拟路由器冗余协议， 它是一种主备模式的协议，通过vrrp可以在网络发生故障时透明的进行设备切换而不影响主机间的数据通信， 可以将多台物理路由器设备虚拟成一个虚拟路由器，通过虚拟ip提供服务，同一时间只有一台物理设备对外提供服务， 

keepalived工作在tcp/ip 的第三四五层 网络层， 传输层， 应用层

在网络层有四个重要的协议， 互联网协议(ip), 互联网控制报文协议(ICMP), 地址转换协议(ARP), 以及反向地址转换协议(RARP), 最长见的工作方式是icmp

在传输层提供两个主要的协议， 传输控制协议(TCP), 用户数据协议(UDP)

在应用层可以运行FTP,TELNET,SMTP,DNS 等各种不同的类型的高层协议， 用户可以自定义keepalived的工作方式

keepalived的体系结构从整体上分为用户空间(User Space) 和 内核空间(Kernel Space), 

![keepalived架构图](../files/softwaredesign.png)

[keepalived官方安装文档](http://www.keepalived.org/doc/installing_keepalived.html)

内核空间出于最底层， 包括IPVS和NETLINK两个模块

官方已经给出了几个配置示例

```
cat /usr/local/etc/keepalived/samples/keepalived.conf.sample 
! Configuration File for keepalived

global_defs { # 全局配置标识符，在此区域内的全都是全局配置选项
   notification_email { # 报警邮件地址，可以设置多个，每行一个，本机需要开启sendmail服务
     acassen
   }
   notification_email_from Alexandre.Cassen@firewall.loc # 设置邮件发送地址
   smtp_server 192.168.200.1 # 邮件smtp服务器地址
   smtp_connect_timeout 30 # 连接smtp服务器的超时时间
   router_id LVS_DEVEL # keepalived服务器的一个标识，可以显示在邮件主题中
}

# keepalived的vrrpd配置
# vrrpd配置可以分成同步组配置和vrrp实例配置
# vrrp同步组， 在多个vrrp实例的环境中，每个vrrp实例所对应的网络环境会有所不同，假设一个实例处于网段A, 另一个实例处于网段B, 如果vrrpd只配置了A网段的检测，那么当B网段主机出现问题时，vrrpd会认为自身处于正常状态，不会进行主备切换， vrrp同步组用来解决此问题， 将所有vrrp实例都加入同步组，这样任何一个实例出现问题，都会进行主备切换
# 

# vrrp实例配置以vrrp_instance 作为标识， 
vrrp_instance VI_1 { # VI_1 实例名称
    # state MASTER/BACKUP keepalived 角色
    interface eth0 # 监听的网络接口
    virtual_router_id 50 # 虚拟路由器标识， 一个数字， 同一个vrrp实例使用唯一的标识， 即在同一个vrrp_instance 下， MASTER和BACKUP必须保持一致
    nopreempt
    priority 100 # 定义节点优先级， 数字越大优先级越高， 在同一个instance下，master必须大于backup
    advert_int 1 # 用于设定master和backup主机之间同步检查的时间间隔， 单位s
    # mcast_src_ip  设置发送多播包的地址， 如果不设置将使用绑定的网卡所对应的地址
    # garp_master_delay 10 用于设定在切换到master状态后延时进行Gratuitous arp请求的时间
    # track_interface { # 用于设定额外的网络监听端口，其中任何一个网络接口出现问题， keepalived都会进入FAULT状态
    #    eth0
    #    eth1
    #}
    #authentication {
    #    auth_type PASS # 有 PASS 和 AH 两种类型
    #    auth_pass PASSWORD # 不长于8位, 超过后自动截取前8位
    #}
    # 
    virtual_ipaddress { # 用于设置虚拟ip(VIP), 也叫做漂移ip地址。 可以设置多个虚拟ip地址， 当keepalived切换到master之后，这些ip地址会自动添加到系统，切换到backup之后，会自动删除， "ip address add", 
        192.168.200.11
        192.168.200.12
        192.168.200.13
        # 192.168.16.189/24 dev eth1 会自动执行 "ip addr add 192.168.16.189/24 dev eth1
    }
    #virtual_route { # 用来设置在切换到master时添加的路由
    #   # src <IPADDR> [to] <IPADDR>/<MASK> via|gw <IPADDR> dev <STRING> scope <SCOPE> 
    #    src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
    #    192.168.110.0/24 via 192.168.200.254 dev eth1
    #    192.168.111.0/24 dev eth2
    #    192.168.112.0/24 via 192.168.100.254
    #    192.168.113.0/24 via 192.168.100.252 or 192.168.100.253
    #}
    # nopreempt 设置高可用集群是不抢占功能， 即 当主节点宕机，备用节点会进行接管，主节点再次正常启动后一般不会自动接管服务，切换时有代价的， 也跟利于稳定性和实时性，直到备用节点出现问题后才会进行切换，在使用不抢占功能时， 只能在state为BACKUP节点设置，而且这个节点的优先级必须高于其他节点？？？？
    # preemtp_delay 200 设置抢占的延时时间, 用于抢占模式， 因为有时系统重启后服务需要一段时间后才能完全启动起来，但是网络连接很快
}


# lvs段的配置以 "virtual_server" 作为开始标识， 分为real_server 段和健康监测段
virtual_server 10.10.10.2 1358 { # 设置虚拟服务器的ip， 后面跟ip地址和端口，
    delay_loop 6 # 设置健康检查的时间间隔，单位s
    lb_algo rr  # 设置负载调度算法， 可用 rr,wrr, wlc,lblc,sh,dh 常用  rr,wlc
    lb_kind NAT # 设置lvs实现负载均衡的机制，有 NAR, TUN, DR三种模式
    persistence_timeout 50 # 回话保持时间, 当用户无操作后多长时间会重新调度，否则一直调度到同一个后端server
    protocol TCP # 指定转发协议类型有tcp, udp 可选
    
    # ha_supend 节点状态从master到backup切换时，暂不启用realserver的健康检查
    # virtualhost 在通过 HTTP_GET/SSL_GET 做健康检查时， 指定的web服务器的虚拟主机名称
    
    sorry_server 192.168.200.200 1358 # 备用节点，当realserver全部宕机后启用

    real_server 192.168.200.2 1358 { # realserver 的ip地址和端口
        weight 1 # 设置realserver的权重
        # inhibit_on_failure 表示在检测到realserver失效后， 把它的weight设置为0，而不是从ipvs删除
        # notify_up <string> | <QUOTED-STRING> 通知脚本
        # 健康监测段, HTTP_GET, SSL_GET,TCP_CHECK, SMTP_CHECK, MISC_CHECK
        HTTP_GET {
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d # 检查后的摘要信息， 可以通过 genhash 命令后工具取得， genhash -s 61.135.169.121 -p 80 -u /index.html
              # status_code 200 # 指定http检查返回正常状态码的类型
            }
            # bindto 192.168.12.80 通过此地址发送请求对服务器进行检查
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
        #TCP_CHECK {
        #    connect_port 80 # 端口
        #   connect_timeout 3 # 超时时间
        #    nb_get_retry 3 # 重试次数
        #    delay_before_retry 3 # 重试间隔
        #}
    }
}
```

```
vrrp同步组配置
cat /usr/local/etc/keepalived/samples/keepalived.conf.vrrp.sync 
! Configuration File for keepalived


# G1 包括1-9个实例，G2 包括3-4 两个实例， 
vrrp_sync_group G1 {
  group {
    VI_1
    VI_2
    VI_5
    VI_6
    VI_7
    VI_8
    VI_9
  }
  notify_backup "/usr/local/bin/vrrp.back arg1 arg2"
  notify_master "/usr/local/bin/vrrp.mast arg1 arg2" # 当keepalived进入master状态时，要执行的脚本，并且允许传入参数，可以是状态报警脚本或是服务管理脚本
  notify_fault "/usr/local/bin/vrrp.fault arg1 arg2"
  #nofify_stop 当keepalived终止时需要执行的脚本
}

vrrp_sync_group G2 {
  group {
    VI_3
    VI_4
  }
}

vrrp_instance VI_1 {
    interface eth0
    state MASTER
    virtual_router_id 51
    priority 100
    virtual_ipaddress {
        192.168.200.18/25
    }
}

vrrp_instance VI_2 {
    interface eth1
    state MASTER
    virtual_router_id 52
    priority 100
    virtual_ipaddress {
        192.168.201.18/26
    }
}

vrrp_instance VI_3 {
    interface eth0
    virtual_router_id 53
    priority 100
    virtual_ipaddress {
        192.168.200.19/27
    }
}

vrrp_instance VI_4 {
    interface eth1
    virtual_router_id 54
    priority 100
    virtual_ipaddress {
        192.168.201.19/28
    }
}

vrrp_instance VI_5 {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 100
    virtual_ipaddress {
        192.168.200.20/27
    }
}


vrrp_instance VI_6 {
    state MASTER
    interface eth0
    virtual_router_id 56
    priority 100
    virtual_ipaddress {
        192.168.200.21/27
    }
}

vrrp_instance VI_7 {
    state MASTER
    interface eth0
    virtual_router_id 57
    priority 100
    virtual_ipaddress {
        192.168.200.22/27
    }
}

vrrp_instance VI_8 {
    state MASTER
    interface eth0
    virtual_router_id 58
    priority 100
    virtual_ipaddress {
        192.168.200.23/27
    }
}

vrrp_instance VI_9 {
    state MASTER
    interface eth0
    virtual_router_id 59
    priority 100
    virtual_ipaddress {
        192.168.200.24/27
    }
}

```


## lvs配置

```
cat /usr/local/etc/keepalived/samples/keepalived.conf.vrrp.lvs_syncd
```

```
! Configuration File for keepalived
vrrp_sync_group VG1 {
  VI_2
  VI_3
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    lvs_sync_daemon_interface eth1
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass grr02
    }
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}

vrrp_instance VI_2 {
    interface eth0
    virtual_router_id 52
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.200.19
        192.168.200.20
        192.168.200.21
    }
}

vrrp_instance VI_3 {
    interface eth1
    virtual_router_id 53
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.201.19
        192.168.201.20
        192.168.201.21
    }
}

virtual_server 192.168.200.19 80 {
    delay_loop 20
    lb_algo rr
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
        }
    }
}
```

