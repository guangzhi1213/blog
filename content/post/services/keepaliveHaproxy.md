---
title: "KeepaliveHaproxy"
date: 2021-04-25T11:08:33+08:00
lastmod: 2021-04-25T11:08:33+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# haproxy

lvs / haproxy

硬件负载均衡 F5, Big-IP;

软件的负载均衡， 又分为两种实现方式， 1. 基于操作系统的软负载实现(lvs) 2. 基于第三方应用的软负载实现(haproxy)

HAproxy: 基于 4层TCP, 7层HTTP应用的负载均衡软件， 可实现 tcp 和 http 应用的负载均衡解决方案， 不仅可以作为web服务器的负载均衡， 也可以作为mysql数据库的负载均衡，

最高可维护 4w-5w 个并发连接， 单位时间内处理的最大请求数为 2w 个， 最大数据处理能力 10Gbit/s 

支持多于 8 种负载均衡算法，同时也支持回话保持

支持虚拟主机功能，实现web负载均衡更加灵活

v1.3 版本后， 支持连接拒绝， 全透明代理等功能

监控界面

ACL 支持， 能给使用者带来很大方便

* 四层和七层负载均衡器

4层就是ISO模型中的4层， 也称为4层交换机， 通过分析ip层和tcp/udp层的流量实现基于ip+port的负载均衡， 基于4层的负载均衡器有lvs，f5等

以常见的tcp应用为例， 负载均衡器在收到第一个来自客户端的SYN请求时， 会通过设定的负载均衡算法选择一个最佳的后端服务器， 同时将报文中目标ip地址修改为后端服务器ip， 然后直接转发给该后端服务器， 这样一个负载均衡请求就完成了。 从这个过程看来一个tcp连接是客户端于服务器直接连接的， 负载均衡器只不过是完成了一个类似路由器的转发动作， 在某些负载均衡器中为保证后端服务器返回的报文可以正确传送给负载均衡器， 在转发报文时可能还会对报文源地址修改。 

客户端 --tcp连接--> 4层负载均衡器(修改报文头目标地址，修改源地址-按需) --转发连接--> RealServer

7层位于OSI的最高层， 即应用层， 支持多种协议， HTTP,FTP,SMTP 等， 根据报文内容， 再配合负载均衡算法来选择后端服务器， 称为"内容交换器", 对于web服务器7层负载均衡器不仅可以根据 "ip+port" 进行负载分流， 还可以根据访问的url，域名，浏览器类别，语言等决定负载均衡的策略

以tcp应用为例， 由于负载均衡器需要获得报文的内容， 因此只能先代替后端服务器和客户端建立连接， 接着才能收到客户端发送过来的报文内容， 然后再根据该报文中特定字段加上负载均衡器中设置的负载均衡算法来决定最终选择的内容服务器， 

客户端 --独立tcp连接--> 7层负载均衡器(代理) --独立的tcp连接--> RealServer

* HAProxy 与 LVS 的区别

1. 两者都是软件负载均衡应用， 但是lvs是基于操作系统实现的软负载，而haproxy是基于第三方应用实现的软负载均衡
1. lvs是基于4层的ip负载均衡技术， 而haproxy是基于4层和7层技术可提供tcp和http应用的负载均衡综合解决方案
1. lvs工作在iso模型的第4层， 因此其状态监测功能单一， 而haproxy在状态监测方面功能强大， 可支持端口，url，脚本等多种状态监测方式
1. haproxy虽然功能强大， 但是整体性能低于4层模式的lvs负载均衡器，而lvs有接近于硬件产品的网络吞吐和连接负载能力

## haproxy

```
~ # cat /etc/haproxy/haproxy.cfg                                            root@localhost
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global # 全局配置段，属于进程级别的配置，通常和操作系统配置有关
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
    # 日志配置段，使用rsyslog中local2的日志设备， 日志记录等级可以设置为err,warning,info,debug等四种可选
    # log 127.0.0.1 local0 info #使用rsyslog中local2的日志设备,记录等级为info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    # 设定每个haproxy进程可接受的最大并发数， 此选项等同于linux命令行选项 `ulimit -n`
    user        haproxy
    # 运行haproxy进程的用户，可使用UID
    group       haproxy
    # 运行haproxy进程的组，可使用GID
    daemon
    # 设置haproxy进程进入后台，推荐的运行模式
    # nbproc 1 # 设置启动时可创建的进程数， 此参数要求运行模式设为daemon, 默认只启动一个进程， 根据经验，建议设置为小于服务器的cpu内核数， 创建多个进程，能够减少每个进程的任务队列，但是过多的进程可能会导致进程崩溃
    # pidfile /usr/local/haproxy/haproxy.pid #设置pid文件，启动时需要user访问权限

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults # 默认参数的配置部分， 在此部分配置的参数默认会被引用到下面的frontend, backend, listen部分中， 一些公用的配置，只需在defaults段添加一次即可， 如果在frontend,backend,listen中也配置了对应的参数，会覆盖defaults配置参数
    mode                    http
    # mode可设置为tcp,http,health
    # tcp 在此模式下客户端与服务端建立一个全双工的连接， 不会对7层报文做任何检查， 默认为tcp模式， 经常用于 ssl,ssh,smtp等应用
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    # 设置连接后端服务器失败重试次数， 连接失败的次数如果超过后标记为不可用， 可在后面配置
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    # 连接到一台服务器的最长等待时间，默认ms，也可使用其他单位做后缀
    timeout client          1m
    # 设置连接客户端发送数据最长等待时间， 默认ms, 也可以使用其它单位后缀
    timeout server          1m
    # 设置服务器端回应客户端数据发送的最长等待时间默认单位是s, 也可以使用其它单位后缀
    timeout http-keep-alive 10s
    timeout check           10s
    # 设置对后端服务器检测的超时时间， 单位ms, 也可使用其它单位后缀
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:5000  # 用于设置接受用户请求段前端虚拟节点， 在v1.3版本后才添加进来的， 同时引入的还有backend段， 可以根据acl规则直接指定要使用的后端backend
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js
    # acl 规则可以实现智能的负载均衡, 工作在7层模型下
    # 功能： 1. 通过设置acl规则， 检查客户端请求是否合法， 如果符合acl规则要求，那么将放行,不符合则直接中断请求 2. 符合acl规则要求的请求将被提交到后端的服务器集群， 进而实现基于acl规则的负载均衡
    # acl 自定义的acl名称 acl方法 -i [匹配的路径或文件]
    # acl方法: 定义实现acl的方法， 常用的方法有 hdr_reg(host),hdr_dom(host),hdr_beg(host),url_sub,url_dir,path_beg,path_end 等
    # -i: 表示忽略大小写, 后面需要跟上匹配的路径或者文件或者正则表达式
    use_backend static          if url_static
    default_backend             app
    # acl规则例子
    # acl www_policy    hdr_reg(host) -i    ^(www.z.cn|z.cn)
    # acl bbs_policy    hdr_dom(host) -i    bbs.z.cn
    # acl url_policy    url_sub -i          buy_sid=
    # use_backend       server_www          if www_policy
    # use_backend       server_app          if url_policy
    # use_backend       server_bbs          if bbs_policy
    # default_backend   server_cache

    # acl另一个例子
    # acl url_static    path_end            .gif .png .jpg .css .js
    # acl host_www      hdr_beg(host) -i    www
    # acl host_static   hdr_beg(host) -i    img. video. download. ftp.
    # use_backend static    if host_static || host_www url_static
    # use_backend www       if host_www
    # default_backend       server_cache

# 也可以设置bind
# frontend main
#     bind *:5000
#     option httplog # 默认不记录http请求，此选项开启记录httplog
#     option forwardfor # 如果后端服务器需要获得客户端的真实ip， 需要配置此段，其实就是增加了一个 "X-Forwarded-For" 字段
#     option httpclose # 表示在客户端和服务器端完成一次连接后主动关闭此tcp连接， 可以优化性能
#     log global # 表示使用全局配置，默认配置
#     default_backend static # 指定默认的后端服务器池， 也就是指定一组后端真实服务器， 这些真实服务器组在backend段定义，static 静态资源组

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static # 用于设置集群后端服务集群， 也就是调价一组真是服务器，以处理前端用户的请求， 添加的真是服务器类似于lvs中的realserver节点
    balance     roundrobin
    server      static 127.0.0.1:4331 check

# backend 段配置
# backend static
#     mode http
#     option redispatch # 用于cookie 保持的环境中， 在默认情况下， haproxy会将其请求的后端服务器的serverID插入cookie中， 以保证会话持久性， 如果后端服务器出现故障，客户端cookie不会刷新，这就出现问题， 此时如果设置此参数， 将会客户端的请求强制重定向到另一个健康的后端服务器上
#     option abortonclose # 设置此参数， 在服务器负载很高的情况下，自动结束当前队列中处理时间比较长的连接
#     balance roundrobin # roundrobin: 基于权重进行轮询调度的算法, static-rr: 基于权重进行轮询的调度算法， 此算法为静态方法， 在运行时调整其服务器权重不会生效, source: 基于请求源ip的算法， 此算法先对请求的源ip进行散列运算， 然后将结果与后端服务器的权重总数相除后转发至某个匹配的服务器端，可以将同一个客户端ip的请求调度到同一台后端服务器, leastconn, 会将新的请求转发到具有最少连接数目的后端服务器， 在回话时间较长的场景推荐使用此算法， 如数据库负载均衡器， 此算法不适合回话较短的环境中，如基于http的应用, uri: 此算法会对部分或整个url进行散列运算， 再经过与服务器的总权重数相除，最后转发到某台匹配的后端服务器上， hdr: 此算法根据http头进行转发， 如果指定的http头名称不存在，则使用roundrobin算法进行策略转发
#     cookie SERVERID # 表示允许向cookie插入SERVERID， 每台服务器的SERVERID可以在下面的server关键字中使用cookie关键字定义，
#     option httpchk GET /index.php # 表示启用http服务状态监测功能， option httpchk <method> <uri> <version> / (version http version)
#     server <name> <address>[:port] [parm*] # server web1 127.0.0.1:8080 cookie server1 weight 6 check inter 2000 rise 2 fall 3
#     



#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 127.0.0.1:5001 check
    server  app2 127.0.0.1:5002 check
    server  app3 127.0.0.1:5003 check
    server  app4 127.0.0.1:5004 check
# listen段设置， 是frontend和backend部分的结合体， 在v1.3以前会在此设置， 为保证兼容性，以后版本也保留了此配置， 保留其一即可
```


## haproxy 维护

```
Usage : haproxy [-f <cfgfile>]* [ -vdVD ] [ -n <maxconn> ] [ -N <maxpconn> ]
        [ -p <pidfile> ] [ -m <max megs> ] [ -C <dir> ]
        -v displays version ; -vv shows known build options.
        -d enters debug mode ; -db only disables background mode.
        -dM[<byte>] poisons memory with <byte> (defaults to 0x50)
        -V enters verbose mode (disables quiet mode)
        -D goes daemon ; -C changes to <dir> before loading files.
        -q quiet mode : don't display messages
        -c check mode : only check config files and exit
        -n sets the maximum total # of connections (2000)
        -m limits the usable amount of memory (in MB)
        -N sets the default, per-proxy maximum # of connections (2000)
        -L set local peer name (default to hostname)
        -p writes pids of all children to this file
        -de disables epoll() usage even when available
        -dp disables poll() usage even when available
        -dS disables splice usage (broken on old kernels)
        -dV disables SSL verify on servers side
        -sf/-st [pid ]* finishes/terminates old pids. Must be last arguments.
```

启动haproxy `/usr/local/haproxy/sbin/haproxy -f /usr/loca/haproxy/conf/haproxy.cfg`

关闭haproxy `killall -9 haproxy`

平滑重启haproxy ```/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/conf/haproxy.cfg -st `cat /usr/local/haproxy/logs/haproxy.pid` ```


```
# keepalived master
global_defs {
   notification_email {
        acassen@firewall.loc
        failover@firewall.loc
        sysadmin@firewall.loc   
        }  
        
        notification_email_from Alexandre.Cassen@firewall.loc
        smtp_server 192.168.200.1
        smtp_connect_timeout 30 
        router_id HAProxy_DEVEL
    }

    vrrp_script check_haproxy {
        script "killall -0 haproxy"     #设置探测HAProxy服务运行状态的方式，这里的“killall
        #-0 haproxy”仅仅是检测HAProxy服务状态
         interval 2
          weight 21    
    }

    vrrp_instance HAProxy_HA {
        state BACKUP                        #在haproxy-server和backup-haproxy上均配置为BACKUP
        
        interface eth0 
        virtual_router_id 80
        priority 100
        advert_int 2 
        nopreempt   #不抢占模式，只在优先级高的机器上设置即可，优先级低的机器可不设置 
        authentication {
                auth_type PASS 
                auth_pass 1111   
        }
        notify_master "/etc/keepalived/mail_notify.py master "    notify_backup "/etc/keepalived/mail_notify.py backup" 
        notify_fault "/etc/keepalived/mail_notify.py falut"
        track_script {    
            check_haproxy    
        }    
        virtual_ipaddress {
                192.168.66.10/24 dev eth0       #HAProxy的对外服务IP，即VIP    }
        }

```

```
# sendmail.py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import sysreload(sys)
from email.MIMEText 
import MIMETextimport smtplibimport MySQLdb
sys.setdefaultencoding('utf-8')
import socket, fcntl, struct
def send_mail(to_list,sub,content):
    mail_host="smtp.163.com"    #设置验证服务器，这里以163.com为例 
    mail_user="username"        #设置验证用户名
    mail_pass="xxxxxx"          #设置验证口令
    mail_postfix="163.com"      #设置邮箱的后缀
    me=mail_user+"<"+mail_user+"@"+mail_postfix+">"
    msg = MIMEText(content)
    msg['Subject'] = sub
    msg['From'] = me
    msg['To'] = to_list
    try:
        s = smtplib.SMTP()
        s.connect(mail_host)
        s.login(mail_user,mail_pass)
        s.sendmail(me, to_list, msg.as_string())
        s.close()
        return True
    except Exception, e:
        print str(e)
        return False
    
def get_local_ip(ifname = 'eth0'):
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    inet = fcntl.ioctl(s.fileno(), 0x8915, struct.pack('256s', ifname[:15]))
    ret = socket.inet_ntoa(inet[20:24])
    return ret

if sys.argv[1]!="master" and sys.argv[1]!="backup" and sys.argv[1]!="fault":
    sys.exit()
else:
    notify_type = sys.argv[1]

if __name__ == '__main__':
    strcontent = get_local_ip()+ " " +notify_type+状态被激活，确认HAProxy服务运行状态！"

    #下面这段是设置接收报警信息的邮件地址列表，可设置多个
        mailto_list = ['xxxxxx@163.com', xxxxxx@qq.com']
        for mailto in mailto_list:
            send_mail(mailto, "HAProxy状态切换报警", strcontent.encode('utf-8'))
```
