---
title: "Archlinux Install Guide."
date: 2021-04-25T10:44:04+08:00
lastmod: 2021-04-25T10:44:04+08:00
draft: true
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

# archlinux 系统安装

> 参考

1. [archlinux 官方安装说明](https://wiki.archlinux.org/index.php/Installation_guide)

2. [archlinux 中文论坛](http://bbs.archlinuxcn.org/)

* Archlinux 的优点

1. 滚动升级, 再也无需因为发行版的更新和重装系统
1. 软件包很新, 第一时间尝试新的软件,比如最新的内核
1. pacman 能够很完美的处理软件包的依赖问题, 再也不会因为ibus而导致gnome启动不了
1. 详细的wiki, 基本能够找到你想要的, 如果你的因为ok

* Archlinux 的缺点

1. pacman 会滚动失败, 还没遇见过
1. 安装有一定的难度
1. 软件包可能存在bug

## 安装前准备工作

### 简介

引导方式使用 `ueif+gpt` 或 `grub+mbr`, 分区方案 采用 `512M boot分区 + 2G swap分区 + 50G home分区 + 100G 根分区`

`ls /sys/firmware/efi/efivars` 查看是否为uefi
### 制作引导盘

1. 下载系统镜像, [阿里云镜像源](http://mirrors.aliyun.com/archlinux/iso/latest/)
1. 制作u盘引导工具, rufus 或者 powerios

### pe进入系统联网更新源

1. 修改源为国内源 `nano /etc/pacman.d/mirrorlist`
1. 添加国内源 `Server http://mirrors.aliyun.com/archlinux/$repo/os/$basearch`
1. 更新数据库 `pacman -Syy`

### 连接无线网

`# wpa_supplicant -i wlp4s0 -c <(wpa_passphrase PDCN xiangpiaopiao)` -B 后台执行

```shell
有线
# systemctl start dhcpcd # 连接
# # systemctl enable dhcpcd 以自动连接
无线连接：
# pacman -S iw wpa_supplicant dialog
# wifi-menu	# 连接
ADSL 宽带连接：
# pacman -S rp-pppoe
# pppoe-setup # 配置
# systemctl start adsl # 连接
# # systemctl enable adsl 以自动连接
```

### 分区

```shell
查看磁盘空间
#lsblk
u盘占据sda，电脑硬盘为 sdb
磁盘分区： 必须有 / , 内存不足开启swap
使用 cfdisk /dev/sdb
一次创建 sdb1:512M boot分区，sdb2:2G swap ,sdb3: 50G / ,sdb4:100G /home 分区
mbr 只能创建4个主分区， 也可以创建扩展分区，然后多个分区方案。
修改分区类型 
# boot分区 选择enable boot 可开机启动，
# swap分区 修改类型为 linux swap
# 其它的默认
```

### 创建分区类型

```shell
启动分区不能为xfs类型，我创建为 ext4，可以直接引导启动
# mkfs.ext4 /dev/sdb1
其余分区格式化为xfs类型,遇见原有分区有文件系统时， 选yes 覆盖即可
# mkfs.xfs /dev/sdb3
# mkfs.xfs /dev/sdb4
开启swap
# mkswap /dev/sdb2
# swapon /dev/sdb2
```

### 挂载分区

```shell
想要在硬盘安装操作系统，首先要按照文件系统的格式 挂载各分区；
挂载点： 可以按照官方 /mnt ，也可以自己选择，由于只需要一次，按照官方挂载在/mnt

# mount /dev/sdb3 /mnt
# mkdir /mnt/{boot,home}
# mount /dev/sdb1 /mnt/boot
# mount /dev/sdb4 /mnt/home
```

## 安装系统

`pacstrap -i /mnt base base-devel linux linux-firmware`

### /etc/fstab

```shell
# genfstab -U <根目录挂载点> >> <根目录挂载点>/etc/fstab
按照示例
#genfstab -U /mnt /etc/fstab
genfstab -U /mnt >> /mnt/etc/fstab
检查文件是否正确
```

### 系统配置

`# arch-chroot /mnt /bin/bash`

### 生成locale文件

Locale 决定了软件使用的语言、书写习惯和字符集。

```shell
# nano /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
使用这三项 
然后执行 locale-gen
设置系统locale，这样log 就会显示成中文
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### locale UTF-8 丢失

有时候使用某些软件是如 pgsql， 需要用到一些语言编码环境变量，否则启动不成功

```shell
localedef -v -c -i en_US -f UTF-8 en_US.UTF-8
```

### 修改键盘布局

```shell
# nano /etc/vconsole.conf
修改了一次 键盘乱了，以后就不再修改了
```

### 设置主机名 archer_wang

```shell
# echo archer_wang > /etc/hostname
添加到对应的hosts
# nano /etc/hosts
127.0.1.1	archer_wang.localdomain	archer_wang
```

### 设置root密码

```shell
echo "mypassword" | passwd --stdin root
```

### 硬件时间设置

```shell
（推荐）UTC 时间：
# hwclock --systohc --utc
本地时间：
# hwclock --systohc --localtime
注意：使用本地时间可能会引起某些不可修复的bug。
```

### 安装grub 引导程序

```shell
# pacman -S grub os-prober
# grub-install --recheck /dev/<目标磁盘>
# grub-mkconfig -o /boot/grub/grub.cfg
```

## 安装完成重启

```shell
# exit 
# umount -R /mnt
# reboot
```

## 重启后什么都没有

20200810 重新安装了一次， 结果发现安装完什么也没有

启动网络： systemctl start systemd-networkd
resovle: systemctl start systemd-resolved

[https://wiki.archlinux.org/index.php/Systemd-networkd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/Systemd-networkd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

极简dhcp配置

`/etc/systemd/network/20-wired.network`:

```
[Match]
Name=enp1s0

[Network]
DHCP=ipv4
```

## 系统备份

`tar cvpjf backup.tar.bz2 --exclude-from=excl /`

## 系统清理

* 清理安装包缓存 `pacman -Scc`
* 清理孤立的安装包 `pacman -Rns $(pacman -Qtdq)`
* 清理日志 `journalctl --vacuum-size=50M` 或者 `journalctl --vacuum-time=1w`
