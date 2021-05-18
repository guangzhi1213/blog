---
title: "Pureftp"
date: 2021-04-25T11:16:49+08:00
lastmod: 2021-04-25T11:16:49+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## 安装

```conf
.configure \
–prefix=/usr/local/pureftpd \ //pureftpd安装目录
–with-everything \ //安装几乎所有的功能，包括altlog、cookies、throttling、ratios、ftpwho、upload script、virtual users（puredb）、quotas、virtual hosts、directory aliases、external authentication、Bonjour、privilege separation本次安装只使用这个选项。
--with-cookie \ //当用户登录时显示指定的横幅
--with-diraliases \ //支持目录别名，用快捷方式代cd命令
--with-extauth \ //编译支持扩展验证的模块,大多数用户不使用这个选项
--with-ftpwho \ //支持pure-ftpwho命令,启用这个功能需要更多的额外内存
--with-language=english \ //修改服务器语言，默认是英文，如果你要做修改，请翻译‘src/messages_en.h’文件
--with-ldap \   //LADP目录支持，需要安装openldap
--with-minimal \ //FTP最小安装，最基本的功能
--with-mysql \ //MySQL支持，如果MySQL安装在自定义目录上，你需要使用命令—with-mysql=/usr/local/mysq这类
--with-nonroot \   //不需要root用户就可以启动服务
```
