---
title: "Fanq"
date: 2021-04-21T18:21:52+08:00
lastmod: 2021-04-21T18:21:52+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# 全球最大那啥网站上一下真的要动一动筋骨

原先用的阿里云香港的服务器，用brook做了一个代理，用了一阵挺好用的，但是好景不长这两天给下通知不让上了

现在转战另一个 影子服务器

也挺好，花钱买服务，但是这个服务并不好 买的第一天就卡在那不动了， 只能苦笑，后来参考别人的配置，
发现这个服务器是都是一个ip上的，然而我还买了香港跟日本的两个账号，然并卵，两个都不能用，不知道是
我本地运营商封的，还是国内都不行， 还是说说别的吧

现在ss和ssr，通常来说ssr是比较靠谱的吧？就是没看见ssr的配置文件，先用ss吧

```json
* 服务器地址：example.ssr.com
*
* * 服务器端口：ssr_port
*
* * 加密方式：aes-256-cfb
*
* * 密码：password
```

在ubuntu服务器上下载ss client , 

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

我只是单传的下载东西时用，使用时就设置一个环境变量就行

2019年6月发现普通的ss模式,已经不能用了, 需要混淆

然后搜了一下 可以用 ` pacman -S shadowsocks-libev simple-obfs` 开混淆模式

`ss-local -c config.json --plugin obfs-local --plugin-opts "obfs=http;obfs-host=www.bing.com"`

其它同理不变


```shell
export http_proxy="socks5://127.0.0.1:1080"
export http_proxy="socks5://127.0.0.1:1080"
curl ip.cn -L
$当前 IP: 165.227.59.161 来自: 美国 

```

ok, 还是蛮快的

20190512

如果需要转换http代理,需要安装 `privoxy` 然后就可以了

谷歌浏览器翻墙

linux: 

```shell
export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"
terminal: google-chrome-stable
下载 switchomega

设置rules https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

