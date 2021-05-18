---
title: "Bios"
date: 2021-04-21T18:21:08+08:00
lastmod: 2021-04-21T18:21:08+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---


# 刷新bios有风险， 需要先备份，先备份，先备份

完整备份， 用的 afuwingui 直接save， 就备份好了

最好也单独记录一下 mac 地址( )，和 uuid(百度: windows查看uuid，一条简单的命令就行) 和 sn号(可以用dmieditwingui直接查看)

[设备唯一标识方法，如何在windows上获取设备唯一标识](http://www.vonwei.com/post/UniqueDeviceIDforWindows.html)

1. 获取主板uuid: `wmic csproduct get UUID`  linux: `dmidecode -s system-uuid`

## bios降级 降级后丢失 sn uuid mac

修改 sn 和 uuid 用的是 DmiEditWinGui, 修改sn在 system information 和 base board 中填写自己备份的sn。

修改 mac 用 afuwin(ami主板，AFUWIN Flasher 3.05.04)

使用uefitool.exe 将官方 `.cap` 文件导出为 `.rom` 文件, 将 ged 单独导出，修改mac地址，然后用uefitool把 .rom 文件 或者 .cap 文件中的ged替换掉，刷进bios， 

俗话说的好，吃一堑长一智。 两天就是在作死
