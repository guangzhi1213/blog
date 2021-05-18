---
title: "Problem"
date: 2021-04-25T10:44:55+08:00
lastmod: 2021-04-25T10:44:55+08:00
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

## yaourt

>One or more PGP signatures could not be verified!

`gpg --recv-keys B550E09EA0E98066`

## 可能会遇到的问题

### locale UTF-8 丢失

在 pgsql 安装后,初始化数据库时会遇到这个问题

```shell
localedef -v -c -i en_US -f UTF-8 en_US.UTF-8
```

### cpu 100%

小新air13 安装manjaro时选择nonfree模式, 就不会导致cpu100%问题了.

### PGP could not be verified

yaourt 安装时报 `One or more PGP signatures could not be verified!`

使用命令 `gpg --recv-keys B550E09EA0E98066`

### 软件安装失败

1. 验证失败
    1. /etc/pacman.conf --> SigLevel = Never

### 图形界面问题

```shell
安装完图形工具后 不能进入图形， systemctl enable gdm
启动gdm就可以了，实现重启进入图形界面
gnome 好丑

卸载gnome   : pacman -Rsc xxxx
卸载所有的关系到的依赖
```

桌面环境安装 manjaro 发行版, 不用考虑那么多了

### 雷柏v500 键盘问题

```shell
ctrl alt 键 不能用， 解决了吧算是， 百度的
`github.com/kinglongmee/rapoo-keyboard-driver`
克隆下来以后 make ， make install，
make报错， no target 。。。。
是因为 缺少了 uname -r -headers 包， 编译 需要 内核-headers ？ 哎  不懂啊， pacman -S linux49-headers
编译通过， 重新变异了内核， 然后每次升级内核都需要重新编译，
再然后是开机需要重新加载驱动程序，把install-rapoo.sh 添加到 rc.local 中，如 /home/manjo/.rapoo/install.sh
```

## 安装完成后的优化

### 安装ssh服务端

openssh 默认禁止root用户登陆

```shell
添加普通用户manjoc
useradd manjoc -r
visudo >> 添加manjoc
```

### oh-my-zsh 安装

切换到zsh

`chsh manjoc -s /bin/zsh`

安装 oh-my-zsh, 不会配置zsh插件 略过

```shell
# curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

```

zsh 主题我用 `ys`

`~/.zshrc` 中 `ZSH_THEME="ys"`

### 输入法

自行百度吧

### 添加archlinuxcn源

```shell
# 在 /etc/pacman.conf 末尾添加
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

添加完后需要安装key

`sudo pacman -S archlinuxcn-keyring`

### 添加yaourt安装工具

`sudo pacman -S yaourt`

### 经常用到的工具软件

`pacman -S bind-tools lsof vim zsh wget curl tcpdump go sysstat dstat iotop iftop htop mariadb`

### mariadb数据库安装

安装完成后需要初始化

`mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`

第一次启动后进行安全加固

```shell
systemctl start mariadb
mysql_secure_installation
systemctl restart mariadb
```

升级了mysql后需要执行这条命令:

`mysql_upgrade -u root -p`

```shell
To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:

'./bin/mysqladmin' -u root password 'new-password'
'./bin/mysqladmin' -u root -h manjo-pc password 'new-password'

Alternatively you can run:
'./bin/mysql_secure_installation'

which will also give you the option of removing the test
databases and anonymous user created by default. This is
strongly recommended for production servers.

See the MariaDB Knowledgebase at http://mariadb.com/kb or the
MySQL manual for more instructions.

You can start the MariaDB daemon with:
cd '.' ; ./bin/mysqld_safe --datadir='./data'

You can test the MariaDB daemon with mysql-test-run.pl
cd './mysql-test' ; perl mysql-test-run.pl

Please report any problems at http://mariadb.org/jira

The latest information about MariaDB is available at http://mariadb.org/.
You can find additional information about the MySQL part at:
http://dev.mysql.com
Consider joining MariaDB's strong and vibrant community:
https://mariadb.org/get-involved/
```

### 设置静态ip

```shell
# Allow users of this group to interact with dhcpcd via the control socket.
#controlgroup wheel
interface ens33
static ip_address=192.168.20.24/16
static routers=192.168.20.1
static domain_name_servers=8.8.8.8
# 设置完后 systemctl enable dhcpcd
```
