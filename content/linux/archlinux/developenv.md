---
title: "Archlinux Developenv Guide."
date: 2021-04-25T10:46:29+08:00
lastmod: 2021-04-25T10:46:29+08:00
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

## 软件安装

yaourt 安装时报 `One or more PGP signatures could not be verified!`

使用命令 `gpg --recv-keys B550E09EA0E98066`

dns使用错误是也会出错, 保险的是使用谷歌dns.

更新archlinux-keyring失败时.

pacman -U /var/cache/pacman/pkg/archlinux-keyring-20200422-1-any.pkg.tar.zst

## 安装

## vim + golang

功能:   自动补全，代码提示，语法检查

`vim-go + vim-youcompleteme + vimrc + zsh + oh-my-zsh`

这几个加起来，嗯，可以正常用了，可惜现在的云厂商都没有这个发行版的镜像，自己也不知道怎么安装这个，是不是可以写个 dockerfile , 然后在云主机 docker 环境下，貌似可以的，有空试试

### golang

```shell
# step 1
$ wget -S https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz
$ tar xf go1.11.2.linux-amd64.tar.gz -C /usr/local

# step 2
$ cat /etc/profile.d/go.sh
GOROOT=/usr/local/go
GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

# step 3
配置git
env GIT_TERMINAL_PROMPT=1 go get code.xxx.org/adarch/kitedemo
否则不提示输入用户名

# 发现一个可笑的事情我使用ubuntu将这个环境变量配置到profile.d 目录下，
# 竟然没有生效， 作为一个合格的运维，我骗自己说重启一下就好了
```

 美化，梅花

```shell
sudo pacman -S git zsh vim-youcompleteme
* vim-youcompleteme 应该是在 archlinux-cn 源中
```

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

`ZSH_THEME="ys"`

然后配置 vimrc

```shell
git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
```

vim-go 直接安装就好了, vim version 8.1

`git clone https://github.com/fatih/vim-go.git ~/.vim/pack/plugins/start/vim-go`

然后安装 vim-go 需要的 binary, maybe 这需要翻墙，忘记了

```shell
vim 中执行， 命令行中 vim + 回车
:GoInstallBinaries
```

小新air13 安装manjaro时选择nonfree模式, 就不会导致cpu100%问题了.


这就ok了，贼鸡好用, 感谢大婶


