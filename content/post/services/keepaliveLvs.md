---
title: "KeepaliveLvs"
date: 2021-04-25T11:09:20+08:00
lastmod: 2021-04-25T11:09:20+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# lvs

## lvs集群

负载均衡层: lvs

服务器群组层: nginx,tomcat

共享存储层: mysql,nfs

lvs 的ip负载均衡技术通过ipvs模块实现， 作用: 安装在director server， 同时虚拟出一个ip地址， 用户必须通过这个虚拟ip访问服务， 称为vip， 

### ipvs的负载均衡机制有三种， NAT, TUN, DR

NAT : 当用户请求到达调度器后， 调度器将请求报文的目标地址(VIP) 改写成选定的 Real Server 地址， 同时报文的目标端口也改成选定的 Real Server 的相应端口， 最后将报文发送到真正的Real Server; 当Real Server 返回数据给用户时， 需要再次将经过负载调度器将报文的源地址和源端口改成虚拟IP的地址和相应的端口， 然后发送给用户

TUN : 通过ip隧道技术实现虚拟服务器， 它的连接调度和管理与 nat方式一样， 只是它的报文转发方法不同， 调度器通过ip隧道技术将用户的请求转发到某个realserver， 而这个Realserver将直接相应用户的请求，不再经过前端调度器。 并且Realserver的地域位置没有要求， 可以在不同的机房， 在此方式中调度器只处理用户报文请求，不响应， 集群的吞吐量大大提高

DR : 通过直接路由技术实现虚拟服务器， 它的连接调度和管理与NAT和TUN一样， 但它的报文转发方式有所不同， DR通过改写请求报文的mac地址， 将请求发送到Realserver， 而Realserver将直接相应用户， 避免了TUN方式ip隧道的开销， 性能最好， 但是 Director Server 和 Real Server 都有一块网卡连在同一个物理网段上

### 负载调度算法

一共有8种调度算法，常用的有: 轮询,加权轮询,最少连接,加权最少连接; 另外四种为: 基于局部性的最少连接，带复制的基于局限性的最少连接， 目标地址散列, 源地址散列

性能: 百兆网卡 吞吐量可高达 10Gbit/s, 千兆网卡 10Gbit/s

支持负载: TCP: HTTP,HTTPS,FTP,SMTP,POP3,IMAP4,PROXY,LDAP,SSMTP等, UDP: DNS,NTP,ICP,视频,音频流播放协议等

`yum -y install ipvsadm`

## keepalived lvs 集群

地址规划;

|节点类型|ip地址规划|主机名|类型|
| -------------------- | :----------- | :------------: | ------: |
| 主Director Server | eth0: 192.168.8.31 | DR1 | 公共ip |
| | eth1: 10.0.10.1 | priv1 | 私有ip |
| | eth0:0 :192.168.8.30 | none | 虚拟ip |
| 备用 Director Server | eth0: 192.168.8.32 | DR2 | 公共ip |
| | eth1: 10.0.10.2 | priv2 | 私有ip |
| Real Server1 | eth0: 192.168.8.33 | rs1 | 公共ip |
| | lo:0 : 192.168.8.30 | none | 虚拟ip |
| Real Server2 | eth0: 192.168.8.34 | rs2 | 公共ip |
| | lo:0 : 192.168.8.30 | none | 虚拟ip |


### keepalived Director

```
# keepalived director master conf 
# 主director节点
 cat keepalived.conf.sample                                              root@localhost
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.8.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance HALVS_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    #nopreempt
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.8.30
    }
}

virtual_server 192.168.8.30 80 {
    delay_loop 6
    lb_algo rr 
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    # sorry_server 192.168.8.34 80 
    
    real_server 192.168.8.33 80 {
        weight 3
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.8.34 80 {
        weight 3
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

```
将 state MASTER 改为 state BACKUP
将 priority 100 改为 priority 80
从节点比主节点优先级要低
```

### Real Server 配置

因为用户请求到达 Realserver 后，直接返回给用户，不再经过Director Server， 因此，就需要在每个Realserver节点添加虚拟的VIP地址， 这样才能返回给用户

```
vim /etc/init.d/lvsrs
#!/bin/bash
VIP=192.168.8.30
/sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
/sbin/route add -host $VIP dev lo:0

echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
sysctl -p
#end
```

写成服务室脚本

```
~$ vim /etc/init.d/lvsrs
#!/bin/bash
VIP=192.168.8.30
./etc/rc.d/init.d/functions
case "$1" in
    start)
        echo "Start LVS of Real Server"
        /sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
        /sbin/route add -host $VIP dev lo:0

        echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
        echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
        ;;
    stop)
        /sbin/ifconfig lo:0 down
        echo "Close LVS Director server"
        echo "0" > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "0" > /proc/sys/net/ipv4/conf/lo/arp_announce
        echo "0" > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo "0" > /proc/sys/net/ipv4/conf/all/arp_announce
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
esac
~$ chmod 755 /etc/init.d/lvsrs
~$ service lvsrs start
~$ /etc/init.d/Keepalived start
~$ /etc/init.d/lvsrs start
```
