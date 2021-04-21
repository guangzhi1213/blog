---
title: "Php"
date: 2021-04-21T18:26:08+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: true
reward: true
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: true
  options: ""

sequenceDiagrams: 
  enable: true
  options: ""

---

## php 安装

服务器上安装, 使用华为mirror源

`remi` 帮助文档: [https://rpms.remirepo.net/wizard/](https://rpms.remirepo.net/wizard/)

### centos7

#### 1.1 epel repository

`yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm`

### 1.2 remi repository

```
yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

### 1.3 yum-utils

```
yum install yum-utils
```

### 1.4 install php	

```
yum  install php72
```

### 1.5 install php addtional package

```
yum  install php72-php-xxx

yum install php72-runtime php72-php-pecl-igbinary php72-php-pecl-xdebug php72-php-imap php72-php-pdo php72-php-ldap php72-php-pecl-zip php72-php-mysqlnd php72-php-gd php72-php-pecl-inotify php72-php-cli php72-php-process php72-php-pecl-mysql php72-php-bcmath php72-php-fpm php72-php-xml php72-php-devel php72-php-json php72-php-pecl-redis php72-php-snmp php72-php-common php72-php-pear php72-php-mbstring 
```

### 1.6 system

`systemctl restart php72-php-fpm`

`cat /etc/opt/remi/php71/php.ini`

### 1.7 更换成华为 mirror

```
sed 's!^metalink=!#metalink=!g' -i /etc/yum.repos.d/remi*.repo
sed 's!^mirrorlist=!#mirrorlist=!g' -i /etc/yum.repos.d/remi*.repo
sed 's!^#baseurl=!baseurl=!g' -i /etc/yum.repos.d/remi*.repo
sed '/^baseurl=/s!https\?://[^/]*/\(remi/\)\?!http://mirrors.huaweicloud.com/remi/!g' -i /etc/yum.repos.d/remi*.repo
```

or

```
sed -e 's!^metalink=!#metalink=!g' \
-e 's!^mirrorlist=!#mirrorlist=!g' \
-e 's!^#baseurl=!baseurl=!g' \
-e '/^baseurl=/s!https\?://[^/]*/\(remi/\)\?\(.*\)!http://mirrors.huaweicloud.com/remi/\2!g;' \
-i /etc/yum.repos.d/remi*.repo;
```
