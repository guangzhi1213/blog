---
title: "Gfw"
date: 2021-04-25T10:54:58+08:00
lastmod: 2021-04-25T10:54:58+08:00
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

# 服务器翻墙指南

原先用的阿里云香港的服务器，用brook做了一个代理，用了一阵挺好用的，但是好景不长这两天给下通知不让上了

## ss版

archlinux 安装包: `sudo pacman -S shadowsocks-libev simple-obfs`

ubuntu 安装包 `sudo apt install shadowsocks-libev simple-obfs`

也挺好，花钱买服务，但是这个服务并不好 买的第一天就卡在那不动了， 只能苦笑，后来参考别人的配置，
发现这个服务器是都是一个ip上的，然而我还买了香港跟日本的两个账号，然并卵，两个都不能用，不知道是
我本地运营商封的，还是国内都不行， 还是说说别的吧

### ss配置文件

现在ss和ssr，通常来说ssr是比较靠谱的吧？就是没看见ssr的配置文件，先用ss吧

```json
服务器地址：example.ssr.com
服务器端口：ssr_port
加密方式：aes-256-cfb
密码：password
```

### 在ubuntu服务器上下载ss client 

```shell
pip install shadowsocks
apt-get install shadowsocks-libev
```

然后就多了一个 shadowsocks-local 命令，ok一切正常

`shadowsocks-local -c config.json`

卧槽竟然上去了，看来是不想让我太折腾了， 好了， 就这样用吧，

```json
 {
     "server":"example.ssr.com",
     "server_port":ssr_server_port,
     "local_address": "127.0.0.1",
     "local_port":1080,
     "password":"ssr_server_password",
     "timeout":300,
     "method":"aes-256-cfb",
     "fast_open": false,
     "workers": 1
 }
```

* 服务方式配置

```shell
cat /etc/shadowsocks-libev/config.json 
{
    "server":"127.0.0.1",
    "server_port":8388,
    "local_port":1080,
    "password":"NiamrejIt9",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```


我只是单传的下载东西时用，使用时就设置一个环境变量就行

## ssr版

2019年6月发现普通的ss模式,已经不能用了, 需要混淆

然后搜了一下 可以用 `pacman -S shadowsocks-libev simple-obfs` 开混淆模式

### 命令行启动

`ss-local -c config.json --plugin obfs-local --plugin-opts "obfs=http;obfs-host=www.bing.com"`

### 服务方式启动

* ubuntu服务器中

```shell
cat /etc/shadowsocks-libev/config-obfs.json
{
    "server":"jaa.relaybit.top",
    "server_port":12030,
    "local_port":1080,
    "password":"0f718f",
    "timeout":60,
    "method":"aes-256-cfb",
    "mode":"tcp_and_udp",
    "fast_open":true,
    "plugin":"obfs-local",
    "plugin_opts":"obfs=http;obfs-host=bing.com"
}
```

`sudo systemctl status shadowsocks-libev-local@config-obfs`

archlinux 应该也是类似的

## socks5协议转成http协议代理

如果需要转换http代理,需要安装 `privoxy` 然后就可以了

安装包 privoxy

### 配置

```shell
listen-address 0.0.0.0:8118
forward-socks5 / 127.0.0.1:1080 .
```

## 谷歌浏览器翻墙

### 桌面版

桌面版有两种方法翻墙, 一种是在terminal中打开浏览器, 一种是使用浏览器插件

#### terminal版

```shell
export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"
terminal: google-chrome-stable
```

#### 插件版

下载 switchomega, 如果在谷歌应用商店中下载,需要翻墙, 所以得回到上一种方式安装插件,或者更改系统代理设置

设置rules `https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`

