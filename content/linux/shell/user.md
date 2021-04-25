---
title: "User"
date: 2021-04-25T11:25:17+08:00
lastmod: 2021-04-25T11:25:17+08:00
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

## 新建一个管理员用户

```shell
export JFUSER="wangqing"
export JFUSERPUB='ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQPgrvrY/GZzUpL247IK7nYjVaeJS9PeuIgYEgo1O0uNYfKGte0BufsaBXJ/GUkGkuoEclv15Cz8ly8p+1tZ3TMWqZ53oNKfCvLl3/z4g7M3BhUzGLg8+vLsvXoshSp6Wo8GvhNr2p3z0rDuS9DZg7/q835ENYmy4nFoH3vTPIDCurzBYIOW6WwjYrYOzkc11SkCeXnSWB9RoTyTLlTL8lmrfuF05KrwKkpifabqnwxtOGA9VHPQ7/g+OzhkZYfQJ0dJLIq/LmsbgZMSQv6teXjjTN1P50CGWGr7HzDxgNoCkA4DuKcYPM6MPZBAarlcVElWXD7LhPfBQ2ndZaJSuP JF-005@Haier-PC'

adduser -g wheel ${JFUSER}

usermod -G wheel ${JFUSER}

passwd ${JFUSER}

echo -n "jfinfo88" | passwd --stdin ${JFUSER}

su - ${JFUSER} -c "mkdir /home/${JFUSER}/.ssh"

su - ${JFUSER} -c "chmod 700 /home/${JFUSER}/.ssh"

echo -n "${JFUSERPUB}" > /home/${JFUSER}/.ssh/authorized_keys

chown ${JFUSER}:wheel /home/${JFUSER}/.ssh/authorized_keys

#su - ${JFUSER} -c "echo -n "${JFUSERPUB}" > /home/${JFUSER}/.ssh/authorized_keys"

chmod 600 /home/${JFUSER}/.ssh/authorized_keys
```

### 私人公钥

```shell
wangwei_pubkey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCeRXYKlctaWEOTTwPIbDoFWRhmi5/ytnxhazpMVf/o36rRMdyNoKsNZDRzmrT/J5X4fS28DG68pN+wRj/lnyQu1xzohWD8hgdYGkHhph4XjfIf+hZyrW5m+iDZ1HQGBQirtKVgpEpP1ivDMLxEsBjgbNtay7ggTCq2jpOTKXbYmw== wangwei@wangwei-PC
```

## 新建一个普通账号,如nginx用户

