---
title: "linux常用命令"
date: 2021-04-25T11:24:12+08:00
lastmod: 2021-04-25T11:24:12+08:00
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

##### 1. 删除过期登陆用户；

```
for user in `w | grep '[0-9][0-9]days' | cut -d " " -f 2` ; do  pkill -kill -t $user  ; done
```

##### 2.获取网卡ip

```
ifconfig eth0 | grep -Po '(?<=addr:).*(?=  Bcast)'
```

##### 3. ip link 使用

```
ip addr add 10.0.10.1/16 brd + dev eno33554984 label eno33554984:1 
```

##### 4. 性能监控工具 top

> ps -ef|grep PID 
>
> 用top -p pid 看看吧
>
> top -u username 查看某个用户的占用资源

##### 5.dig命令

安装：centos,fedora: yum install bind-utils ,ubuntu: dnsutils

  用命令 rpm -qf $(which dig)

##### 6.ping命令

指定源ip：ping -I  | mtr --address

##### 7.arp  绑定，解绑

解绑:

```
arp -a && arp -a |awk -F '(' '{print $2}'|awk -F ')' '{print $1}'|while read ip;do arp -d $ip;done
```

绑定:

```
#!/bin/bash
###huoqu wangguan IPhe MAC
   GWIP=`/sbin/route -n|awk '$1 == "0.0.0.0"'|awk '{print $2}'`
   GWMAC=`/sbin/arp -a $GWIP|awk '{print $4}'`
###bang ding IP ,xiu gai qidong wenjian
   /sbin/arp -s $GWIP $GWMAC
if [ ! `cat /etc/rc.local|grep hostarp` ];then
   echo "/vdncloud/script/hostarp" >> /etc/rc.local
fi
###jiancha queren
/sbin/arp -a|grep PERM
/sbin/route -n|grep $GWIP
```

##### 8. 硬盘监控： smartmontools

`smartctl -t short /dev/sda`

`smartctl -a /dev/sda`

##### 9. lspci

此命令在 pciutils 中

##### 10. 获取ip地址

1. method1

```shell
ifconfig | sed -n '/eth0/{N;s/.*inet addr://;s/ .*//p}'ifconfig | awk '/eth0/{getline;gsub(/addr:/,"",$2);print $2}'ifconfig | awk -F"[ \t:]+" -vdev=eth0 '$2=="Link"{d=$1==dev?1:0}d&&/inet addr:/{print $4}'ifconfig eth0 | sed '/inet addr/!d; s/.*inet addr://; s/ .*//'ifconfig eth0 | awk -F '[: ]+' '/inet addr/{print $4}'ifconfig eth0 | awk '/inet addr/{gsub(/addr:/,"",$2);print $2}'ifconfig eth0 | awk -vRS='inet addr:' 'NR!=1{print $1}'ifconfig eth0 | grep -Po '(?<=inet addr:)\S+'ip r | awk 'NR==1{print $NF}'
```

ifconfig eth0 | grep -Po '(?<=addr:).*(?=  Bcast)'

2. method2

```shell
$ ifconfig eth3
eth3   Link encap:Ethernet HWaddr 08:00:12:34:56:78 
​     inet addr:10.0.222.15 Bcast:10.0.222.255 Mask:255.255.255.0
​     inet6 addr: fe80::a00:12ff:fe34:5678/64 Scope:Link
​     ...
$ ifconfig eth3 | grep -oP "HWaddr \K\S+"    -->  08:00:12:34:56:78
$ ifconfig eth3 | grep -oP "inet addr:\K\S+"    -->  10.0.222.15
$ ifconfig eth3 | grep -oP "inet6 addr: \K\S+"   --> fe80::a00:12ff:fe34:5678/64
$ ifconfig eth3 | grep -oP "inet6 addr: \K[^\/]+"  -->  fe80::a00:12ff:fe34:5678
$ ifconfig eth3 | awk 'match($0,/HWaddr ([^ ]+)/,a){print a[1]}'  -->  08:00:12:34:56:78
$ ifconfig eth3 | awk 'match($0,/inet addr:([^ ]+)/,a){print a[1]}'  -->  10.0.222.15
$ ifconfig eth3 | awk 'match($0,/inet6 addr: ([^ ]+)/,a){print a[1]}'  -->  fe80::a00:12ff:fe34:5678/64
$ ifconfig eth3 | awk 'match($0,/inet6 addr: ([^\/]+)/,a){print a[1]}'  -->  fe80::a00:12ff:fe34:5678
```

##### IFS

The reason that IFS is not being set is that bash isn't seeing that as a separate command... you need to put a line feed or a semicolon after the command in order to terminate it:

```
$ cat /tmp/ifs.sh
LINE="7.6.5.4"
IFS='.'  read -a ARRAY <<< "$LINE"
echo "$IFS"
echo "${ARRAY[@]}"
$ bash /tmp/ifs.sh 
7 6 5 4
but
$ cat /tmp/ifs.sh 
LINE="7.6.5.4"
IFS='.';  read -a ARRAY <<< "$LINE"
echo "$IFS"
echo "${ARRAY[@]}"

$ bash /tmp/ifs.sh 
.
7 6 5 4
```

##### 12. 网络状态监测

nc

nmap

> nmap -p xx,xx -sV -sS -T4 -oN nmap.txt -iL list.txt

##### 13. 比较两个文件不同的地方

```
统计两个文本文件的相同行
grep -Ff file1 file2
统计file2中有，file1中没有的行
grep -vFf file2 file1
```

##### 14. curl 查看自己的公网IP

curl -s ip.cn

##### 15 scrt 护眼模式

色调:85;饱和度:123;亮度:205

1、ssh

2、无状态连接；http

4、生成随机数  30个

​	openssl rand -base64 30		

​	tr -dc A-Za-z0-9_ < /dev/urandom | head -c 30 | xargs

6、

dsa/dss 拥有固定长度 1024bits

rsa  

ecdsa  256 384 521 位

7、

取消 ssh 反解 主机名

编辑/etc/ssh/sshd_config

找到 UseDNS 设为 no

8、

配置私钥公钥

\#: (umask 077;openssl genrsa -out ssl/nginx.key 1024)

![屏幕截图(19).png](file:///private/var/folders/tl/lg1qq8j11ds_wkspmdvhv2ch0000gn/T/WizNote/cb9b06fa-afd5-4817-b8fe-5618b8ad6f58/index_files/0161aab8-62ba-493d-9bae-0e1a9b92250e.png)

9、

路由：

route add default gw 172.18.10.1

ip:

ip addr add 172.18.59.5/16 dev eno16777736

11、

journalctl -xn
