---
title: "KernelModuleManage"
date: 2021-04-25T10:55:30+08:00
lastmod: 2021-04-25T10:55:30+08:00
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

## Linux内核模块化设计

### Linux 内核设计：单内核、模块化 ( 动态装载和卸载)

(1)Linux：单内核设计，但充分借鉴了微内核体系的设计的优点；为内核引入了模块化机制；

(2) 内核的组成部分：

kernel：内核核心，一般为bzImage格式，通常位于/boot目录，名称为vmlinuz-VERSION-release；

当系统启动之后该文件就不在使用，因为已经加载到内存，放置/boot下方便管理

kernel object：内核模块，一般放置于/lib/modules/VERSION-release/

内核模块与内核核心版本一定要严格匹配；

### 内核模块：编译选择模式

[ ]：N，不变异此部分

[M]：Module ，以模块化编译，可以临时装载，占用磁盘空间，不占用内核空间

[*]：Y，编译进内核核心，可以直接调用

### ramdisk：辅助性文件，并非必须，取决于内核是否能直接驱动rootfs所在的设备

ramdisk：一个简装版的根文件系统，可提供的驱动如下：

目标设备驱动，例如SCSI设备的驱动；

逻辑设备驱动，例如LVM设备的驱动；

文件系统，例如xfs文件系统；

### 内核模块信息获取和管理命令

#### ldd：打印二进制应用程序所依赖的库文件

print shared library dependencies

**格式：ldd [OPTION]... FILE...**

显示：1) 所依赖库文件名称 => 所依赖 库文件路径  (对应内存载入符号链接映射指向)

2) 整个系统调用库的入口

linux-vdso.so.1 => (0x00007fff293fe000)

/lib64/ld-linux-x86-64.so.2 (0x00007f0228073000)

```shell
[root@91fun ~]# ldd /bin/ls
	linux-vdso.so.1 =>  (0x00007ffee4fcb000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007ff2b3689000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007ff2b3484000)
	libacl.so.1 => /lib64/libacl.so.1 (0x00007ff2b327b000)
	libc.so.6 => /lib64/libc.so.6 (0x00007ff2b2ead000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007ff2b2c4b000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007ff2b2a47000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff2b38b0000)
	libattr.so.1 => /lib64/libattr.so.1 (0x00007ff2b2842000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007ff2b2626000)
```

#### uname ：内核信息获取

print system information**

**格式：uname [OPTION]…**

uname  -a：显示内核所有信息

uname  -v：内核的编译版本号

uname -r：内核的release发行号

uname -n：主机名

```shell
[root@91fun ~]# uname -a
Linux 91fun.club 3.10.0-1062.4.1.el7.x86_64 #1 SMP Fri Oct 18 17:15:30 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[root@91fun ~]# uname -r
3.10.0-1062.4.1.el7.x86_64
[root@91fun ~]# uname -v
#1 SMP Fri Oct 18 17:15:30 UTC 2019
[root@91fun ~]# uname -n
91fun.club
```

#### lsmod：列出内核模块

显示的内核来自于/proc/modules

模块名，大小，被引用的次数、被什么引用

```shell
[root@91fun ~]# lsmod
Module                  Size  Used by
unix_diag              12601  0
af_packet_diag         12611  0
netlink_diag           12669  0
sha512_ssse3           42080  0
sha512_generic         13131  1 sha512_ssse3
drbg                   30186  1
ansi_cprng             12989  0
authenc                17776  0
esp4                   17247  0
xfrm4_mode_transport    12631  0
udp_diag               12801  0
l2tp_ppp               22567  0
l2tp_netlink           17851  1 l2tp_ppp
l2tp_core              34884  2 l2tp_ppp,l2tp_netlink
pppoe                  17920  0
pppox                  13342  2 l2tp_ppp,pppoe
ppp_generic            33041  3 l2tp_ppp,pppoe,pppox
slhc                   13450  1 ppp_generic
xt_multiport           12798  1
xt_policy              12582  3
dummy                  12960  0
xt_nat                 12681  5
veth                   13458  0
tcp_diag               12591  0
inet_diag              18949  2 tcp_diag,udp_diag
xt_conntrack           12760  6
ipt_MASQUERADE         12678  8
nf_nat_masquerade_ipv4    13430  1 ipt_MASQUERADE
nf_conntrack_netlink    36354  0
nfnetlink              14519  2 nf_conntrack_netlink
xt_addrtype            12676  2
iptable_filter         12810  2
iptable_nat            12875  2
nf_conntrack_ipv4      15053  7
nf_defrag_ipv4         12729  1 nf_conntrack_ipv4
nf_nat_ipv4            14115  1 iptable_nat
nf_nat                 26583  3 nf_nat_ipv4,xt_nat,nf_nat_masquerade_ipv4
nf_conntrack          139224  6 nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_ipv4
libcrc32c              12644  2 nf_nat,nf_conntrack
br_netfilter           22256  0
bridge                151336  1 br_netfilter
stp                    12976  1 bridge
llc                    14552  2 stp,bridge
overlay                91659  3
sb_edac                32114  0
iosf_mbi               15582  0
crc32_pclmul           13133  0
ghash_clmulni_intel    13273  0
aesni_intel           189456  0
ppdev                  17671  0
lrw                    13286  1 aesni_intel
gf128mul               15139  1 lrw
glue_helper            13990  1 aesni_intel
ablk_helper            13597  1 aesni_intel
cryptd                 21190  3 ghash_clmulni_intel,aesni_intel,ablk_helper
pcspkr                 12718  0
joydev                 17389  0
virtio_balloon         18015  0
parport_pc             28205  0
i2c_piix4              22401  0
parport                46395  2 ppdev,parport_pc
ip_tables              27126  2 iptable_filter,iptable_nat
ext4                  584153  1
mbcache                14958  1 ext4
jbd2                  107478  1 ext4
virtio_net             28063  0
virtio_console         28076  0
virtio_blk             18323  2
ata_generic            12923  0
pata_acpi              13053  0
cirrus                 24171  1
drm_kms_helper        186531  1 cirrus
syscopyarea            12529  1 drm_kms_helper
sysfillrect            12701  1 drm_kms_helper
sysimgblt              12640  1 drm_kms_helper
fb_sys_fops            12703  1 drm_kms_helper
ttm                    96673  1 cirrus
drm                   456166  4 ttm,drm_kms_helper,cirrus
ata_piix               35052  0
libata                243133  3 pata_acpi,ata_generic,ata_piix
crct10dif_pclmul       14307  0
crct10dif_common       12595  1 crct10dif_pclmul
crc32c_intel           22094  1
floppy                 69432  0
serio_raw              13434  0
virtio_pci             22985  0
virtio_ring            22746  5 virtio_blk,virtio_net,virtio_pci,virtio_balloon,virtio_console
virtio                 14959  5 virtio_blk,virtio_net,virtio_pci,virtio_balloon,virtio_console
drm_panel_orientation_quirks    17180  1 drm
```

#### modinfo命令：显示指定的模块的详细信息

**格式：odinfo [-F field] [-k kernel] [modulename|filename...]**

-k kernel：多内核并存时若要查询另外一个kernel上的模块信息

-F field： 仅显示指定字段的信息；

-n：显示文件路径；

通过读取/lib/modules/#######/*文件的原数据来显示相关信息

```shell
[root@91fun ~]# ls /lib/modules/3.10.0-1062.4.1.el7.x86_64/
build          modules.alias.bin    modules.dep      modules.modesetting  modules.symbols      vdso
extra          modules.block        modules.dep.bin  modules.networking   modules.symbols.bin  weak-updates
kernel         modules.builtin      modules.devname  modules.order        source
modules.alias  modules.builtin.bin  modules.drm      modules.softdep      updates
```

显示内容：文件名、协议、描述、作者、别名、适用于RHEL版本号、依赖的模块、签名单位、签名、加密算法

```shell
[root@91fun ~]# modinfo ext4
filename:       /lib/modules/3.10.0-1062.4.1.el7.x86_64/kernel/fs/ext4/ext4.ko.xz
license:        GPL
description:    Fourth Extended Filesystem
author:         Remy Card, Stephen Tweedie, Andrew Morton, Andreas Dilger, Theodore Ts'o and others
alias:          fs-ext4
alias:          ext3
alias:          fs-ext3
alias:          ext2
alias:          fs-ext2
retpoline:      Y
rhelversion:    7.7
srcversion:     3FC0F2CFC3F9938AE9C8339
depends:        mbcache,jbd2
intree:         Y
vermagic:       3.10.0-1062.4.1.el7.x86_64 SMP mod_unload modversions
signer:         CentOS Linux kernel signing key
sig_key:        60:48:F2:5B:83:1E:C4:47:02:00:E2:36:02:C5:CA:83:1D:18:CF:8F
sig_hashalgo:   sha256
```

#### modprobe：实现模块的装载和卸载，同时会挂载依赖的模块

**格式：modprobe [-r] module_name**

模块的动态装载：modprobe module_name

动态卸载：modprobe -r module_name

 注意：默认被装载的模块不要随意卸载

```shell
[root@91fun ~]# lsmod |grep btrfs
[root@91fun ~]# modprobe btrfs
[root@91fun ~]# lsmod |grep btrfs
btrfs                1074009  0
raid6_pq              102527  1 btrfs
xor                    21411  1 btrfs
```

#### depmod：- Generate modules.dep and map files

内核模块依赖关系文件和系统信息映射文件的生成工具；

#### insmod,rmmod：模块的装载和卸载，不能自动解决模块依赖关系

insmod [filename] [module options...]

filename：模块文件的文件路径；

rmmod [module_name]

### ramdisk文件的管理

#### mkinitrd (CentOS 5)：为当前使用中的内核重新制作ramdisk文件

`mkinitrd [OPTION...] [<initrd-image>] <kernel-version>`

--with=<module>：除了默认的模块之外需要装载至initramfs中的模块；

--preload=<module>：initramfs所提供的模块需要预先装载的模块；

示例： ~]# mkinitrd /boot/initramfs-$(uname -r).img  $(uname -r)

#### dracut(CentOS 6/7 ,兼容5)

low-level tool for generating an initramfs image**

`dracut [OPTION...] [<image> [<kernel version>]]`

示例： 

`~]# dracut /boot/initramfs-$(uname -r).img $(uname -r)`

### 四 内核信息输出伪文件系统

#### /proc：内核状态和统计信息的输出接口；还提供一个配置接口，/proc/sys

**参数：**

只读：信息输出；例如/proc/#/*,进程相关信息

```shell
[root@91fun ~]# ls /proc/
1      11316  2     273    41    570   846   9885       diskstats    keys        net            sysvipc
10     11319  20    279    43    571   863   9890       dma          key-users   pagetypeinfo   timer_list
1031   13     21    288    44    578   8982  9907       driver       kmsg        partitions     timer_stats
10924  14     22    289    45    579   9     9953       execdomains  kpagecount  sched_debug    tty
10926  15     23    29837  46    59    9098  acpi       fb           kpageflags  schedstat      uptime
10927  16     24    30     4793  6     9109  buddyinfo  filesystems  loadavg     scsi           version
11     17     2420  30000  4849  6516  9120  bus        fs           locks       self           vmallocinfo
11071  17616  256   31     507   6555  9125  cgroups    interrupts   mdstat      slabinfo       vmstat
11072  17621  2588  32     534   7     9142  cmdline    iomem        meminfo     softirqs       zoneinfo
11073  17639  262   33     535   7561  94    consoles   ioports      misc        stat
11281  18     264   367    541   777   9472  cpuinfo    irq          modules     swaps
11287  19     265   390    555   8     9478  crypto     kallsyms     mounts      sys
11293  19428  267   4      5611  843   9868  devices    kcore        mtrr        sysrq-trigger
```

可写：可接受用户指定一个“新值”来实现对内核某功能或特性的配置；/proc/sys/

格式： /proc/sys: net/ipv4/ip_forward 相当于 net.ipv4.ip_forward

```shell
[root@91fun ~]# ll /proc/sys
total 0
dr-xr-xr-x 1 root root 0 Mar  9 00:55 abi
dr-xr-xr-x 1 root root 0 Nov 13 10:52 crypto
dr-xr-xr-x 1 root root 0 Mar  9 00:55 debug
dr-xr-xr-x 1 root root 0 Mar  9 00:55 dev
dr-xr-xr-x 1 root root 0 Nov 13 18:52 fs
dr-xr-xr-x 1 root root 0 Nov 13 18:52 kernel
dr-xr-xr-x 1 root root 0 Nov 13 18:52 net
dr-xr-xr-x 1 root root 0 Mar  9 00:55 user
dr-xr-xr-x 1 root root 0 Nov 13 18:52 vm
```

#### 修改参数方式

**1) sysctl命令**

专用于查看或设定/proc/sys目录下参数的值； sysctl [options] [variable[=value]]

查看：# sysctl -a；# sysctl variable        

修改其值：# sysctl -w variable=value

**2) 文件系统命令（cat, echo)**

查看：# cat /proc/sys/PATH/TO/SOME_KERNEL_FILE

设定：# echo "VALUE"  > /proc/sys/PATH/TO/SOME_KERNEL_FILE

**3) 配置文件：/etc/sysctl.conf, /etc/sysctl.d/\*.conf**

立即生效的方式：sysctl -p [/PATH/TO/CONFIG_FILE]

#### 重要的内核参数：

net.ipv4.ip_forward：核心转发；

vm.drop_caches：

kernel.hostname：主机名；

net.ipv4.icmp_echo_ignore_all：忽略所有ping操作；

  
#### /sys目录：Kernel 2.6版本后引入

sys文件系统：输出内核识别出的各硬件设备的相关属性信息，也有内核对硬件特性的可设置参数；

对此些参数的修改，即可定制硬件设备工作特性；

```shell
[root@91fun ~]# ll /sys
total 0
drwxr-xr-x   2 root root 0 Nov 13 18:52 block
drwxr-xr-x  33 root root 0 Nov 13 18:52 bus
drwxr-xr-x  51 root root 0 Nov 13 18:52 class
drwxr-xr-x   4 root root 0 Nov 13 18:52 dev
drwxr-xr-x  14 root root 0 Nov 13 18:52 devices
drwxr-xr-x   6 root root 0 Nov 13 18:52 firmware
drwxr-xr-x   7 root root 0 Nov 13 18:52 fs
drwxr-xr-x   2 root root 0 Nov 13 18:52 hypervisor
drwxr-xr-x  10 root root 0 Nov 13 18:52 kernel
drwxr-xr-x 161 root root 0 Nov 13 18:52 module
drwxr-xr-x   2 root root 0 Mar  9 00:55 power
```

udev：通过读取/sys目录下的硬件设备信息按需为各硬件设备创建设备文件；

udev是用户空间程序；专用工具：devadmin, hotplug；

udev为设备创建设备文件时，会读取其事先定义好的规则文件

一般在/etc/udev/rules.d/目录下，以及/usr/lib/udev/rules.d/目录下
