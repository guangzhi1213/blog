---
title: "Diskmanager"
date: 2021-04-25T10:52:36+08:00
lastmod: 2021-04-25T10:52:36+08:00
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

**一、磁盘管理命令**

**1.du：评估文件系统的磁盘使用量(显示美国而目录的大小)**

格式：du [OPTION]... [FILE]...

-s: sumary 列出总量

-h: human-readable  G/M格式显示出来

**2.df：报告文件系统分区的使用情况**

格式：df [OPTION]... [FILE]...

-l：仅显示本地文件的相关信息；

-h：human-readable

-i：显示inode的使用状态而非blocks

**3.fdisk：创建查看分区**

**(1)查看磁盘的分区信息****：**

fdisk -l [-u] [device...]：列出指定磁盘设备上的分区情况；

**(2)管理分区**

fdisk提供了一个交互式接口来管理分区，它有许多子命令，分别用于不同的管理功能；所有的操作均在内存中完成，没有直接同步到磁盘；直到使用w命令保存至磁盘上；

n：创建新分区

d：删除已有分区

t：修改分区类型

l：查看所有已经ID

w：保存并退出

q：不保存并退出

m：查看帮助信息

p：显示现有分区信息

**注意**：在已经分区并且已经挂载其中某个分区的磁盘设备上创建的新分区，内核可能在创建完成后无法直接识别；

**4.通知内核强制重读磁盘分区表：**

查看：cat /proc/partitions

CentOS 5：partprobe [device]

CentOS 6,7：partx, kpartx

partx -a [device]

kpartx -af [device]

**5.ext系列文件系统专用管理工具**

**(1) mke2fs 创建文件系统**

mke2fs [OPTIONS] device

-t {ext2|ext3|ext4}：指明要创建的文件系统类型

mkfs.ext4 = mkfs -t ext4 = mke2fs -t ext4

-b {1024|2048|4096}：指明文件系统的块大小；

-L LABEL：指明卷标；

-j：创建有日志功能的文件系统ext3；

mke2fs -j = mke2fs -t ext3 = mkfs -t ext3 = mkfs.ext3

-i #：bytes-per-inode，指明inode与字节的比率；即每多少字节创建一个Indode;

-N #：直接指明要给此文件系统创建的inode的数量；

-m #：指定预留的空间，百分比；

-O [^]FEATURE：以指定的特性创建目标文件系统；

**(2)****e2label：卷标的查看与设定** 

查看：e2label device

设定：e2label device LABEL

**(3)tune2fs：查看或修改ext系列文件系统的某些属性，块大小创建后不可修改**

**adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems；**

tune2fs [OPTIONS] device

-l：查看超级块的内容；

修改指定文件系统的属性：

-j：ext2 --> ext3；

-L LABEL：修改卷标；

-m #：调整预留空间百分比；

-O [^]FEATHER：开启或关闭某种特性；

-o [^]mount_options：开启或关闭某种默认挂载选项  acl ^acl

**(4)dumpe2fs命令：显示ext系列文件系统的属性信息**

dumpe2fs [-h] device

**(5)e2fsck :检测一个ext2/ext3/ext4文件系统的Linux**

e2fsck [OPTIONS] device

-y：对所有问题自动回答为yes;

-f：即使文件系统处于clean状态，也要强制进行检测；

**6.fsck：check and repair a Linux file system 检查修复Linux文件系统**

因进程意外中止或系统崩溃等 原因导致定稿操作非正常终止时，可能会造成文件损坏；

此时，应该检测并修复文件系统； 建议，离线进行；

-t fstype：指明文件系统类型；

fsck -t ext4 = fsck.ext4

-a：无须交互而自动修复所有错误；

-r：交互式修复；

**6.blkid：查看文件系统UUID、卷标、安全类型、格式等相关信息**

blkid device

blkid -L LABEL：根据LABEL定位设备

blkid -U UUID：根据UUID定位设备

**二、swap文件系统**

**1.Linux上的交换分区必须使用独立的文件系统；且文件系统的System ID必须为82；**

**2.mkswap；创建swap设备**

mkswap [OPTIONS] device

-L LABEL：指明卷标

-f：强制

**3.交换分区的启用和禁用**

启用：swapon

swapon [OPTION] [DEVICE]

-a：定义在/etc/fstab文件中的所有swap设备；

禁用：swapoff

swapoff DEVICE

**三、挂载和卸载**

**1.挂载：根文件系统外通过关联至根文件系统上的某个目录来实现访问**

挂载点：mount_point，用于作为另一个文件系统的访问入口；

(1) 事先存在；

(2) 应该使用未被或不会被其它进程使用到的目录；

(3) 挂载点下原有的文件将会被隐藏；

**2.mount命令**

**(1)mount：查看挂载分区**

\# mount

\# cat /etc/mtab

\# cat /proc/mounts

**(2)mount [-nrw] [-t vfstype] [-o options] device dir** 

-r：readonly，只读挂载；

-w：read and write, 读写挂载；

-n：默认情况下，设备挂载或卸载的操作会同步更新至/etc/mtab文件中；-n用于禁止此特性；

-t vfstype：指明要挂载的设备上的文件系统的类型；多数情况下可省略，

此时mount会通过blkid来判断要挂载的设备的文件系统类型；

-L LABEL：挂载时以卷标的方式指明设备； mount -L LABEL dir

-U UUID：挂载时以UUID的方式指明设备；mount -U UUID dir

-o options：挂载选项

    sync/async：同步/异步操作；

    atime/noatime：文件或目录在被访问时是否更新其访问时间戳；

    diratime/nodiratime：目录在被访问时是否更新其访问时间戳；

    remount：重新挂载；

    acl：支持使用facl功能；

`mount -o acl device dir`

`tune2fs -o acl device`

ro：只读

rw：读写

dev/nodev：此设备上是否允许创建设备文件；

exec/noexec：是否允许运行此设备上的程序文件；

auto/noauto：

user/nouser：是否允许普通用户挂载此文件系统；

suid/nosuid：是否允许程序文件上的suid和sgid特殊权限生效；        

   defaults：Use default options: rw, suid, dev, exec, auto, nouser, async, and relatime.

**【使用技巧】可以实现将目录绑定至另一个目录上，作为其临时访问入口；**

**mount --bind 源目录 目标目录**

**3.设定除根文件系统以外的其它文件系统能够开机时自动挂载：/etc/fstab文件**

每行定义一个要挂载的文件系统及相关属性：

(1) 要挂载的设备：

设备文件；

LABEL

UUID

伪文件系统：如sysfs, proc, tmpfs等

(2) 挂载点

swap类型的设备的挂载点为swap；

(3) 文件系统类型；

(4) 挂载选项

defaults：使用默认挂载选项；

如果要同时指明多个挂载选项，彼此间以事情分隔；

defaults,acl,noatime,noexec

(5) 转储频率

0：从不备份；

1：每天备份；

2：每隔一天备份；

(6) 自检次序

0：不自检；

1：首先自检，通常只能是根文件系统可用1；

2：次级自检

...

mount -a：可自动挂载定义在此文件中的所支持自动挂载的设备；

**4.fuser****、****lsof**

正在被进程访问到的挂载点无法被卸载，查看被哪个或哪些进程所占用：

**# lsof MOUNT_POINT**

**# fuser -v MOUNT_POINT**

终止所有正在访问某挂载点的进程**：# fuser -km MOUNT_POINT**