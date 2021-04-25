---
title: "Dotfile"
date: 2021-04-25T10:35:21+08:00
lastmod: 2021-04-25T10:35:21+08:00
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

# 常用工具的本地配置

## git

`~/.gitconfig`

```shell
[user]
        email = xiuxia.zg@gmail.com
        name = manjoc
[http]
        proxy = sock5://127.0.0.1:1080
[https]
        proxy = sock5://127.0.0.1:1080
```

## ssh

`~/.ssh/config`

```shell
Host manjoc@91fun.club
    HostName 91fun.club
    User manjoc
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
Host ubuntu2004-office
    HostName 192.168.200.128
    User manjoc
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519
Host archlinux
    HostName 172.18.128.1
    Port 22
    User root
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519
```

## 