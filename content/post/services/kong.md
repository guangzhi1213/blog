---
title: "Kong"
date: 2021-04-25T11:09:43+08:00
lastmod: 2021-04-25T11:09:43+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---


# kong api 网关使用

帮助文档:  
1. [API网关kong学习笔记](http://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/09/30/integrate-kubernetes-with-kong.html) 个人认为写的还是比较好的,也介绍了kubernetes中怎么安装使用
2. 因为安装的时候不太开心 所以决定翻译一下官方文档

## install

kong 安装的时候选择了删除 openresty 进行安装, 因为在自己虚拟机上安装的时候 用了原本的 openresty 结果安装的时候纠结了半天, 如果你不是跟我一样容易纠结的人,请看  
[kong 源码安装](https://docs.konghq.com/install/source/)

## install konga dashboard

使用 konga dash 界面来管理 kong  
项目地址: https://github.com/pantsel/konga  

第一次启动直接创建 admin 的用户名和密码

## kong in five minutes

可以简单理解 route 是 pc 到 kong 的配置, service 是才 kong 到后端服务器的配置,及自己服务的配置

### 启动 kong

执行数据迁移, 其实就是创建一些 kong 的表结构  
`kong migrations bootstrap [-c /path/to/kong.conf]`

启动   
`kong start [-c /etc/kong/kong.conf]`

如果你是自己在 openresty 上自己安装的 可以在这选择自己的配置  

### 确认 kong 已经启动成功

启动后会看到 `Kong started`  
默认监听端口:
- :8000 在其上侦听来自客户端的传入HTTP流量，并将其转发到您的上游服务。
- :8443Kong监听传入的HTTPS流量。此端口具有与端口类似的行为:8000，但它仅需要HTTPS流量。可以通过配置文件禁用此端口。
- :8001在其上的管理员API用于配置监听香港。
- :8444 Admin API侦听HTTPS流量的位置。

### 停止 kong, 重新加载 kong

跟 nginx 一样:  
停止: `kong stop`  
重载: `kong reload`  

## 服务配置

### 使用Admin API添加服务

- 添加服务
- 添加路由

我就直接使用 dash 创建了

#### 添加指向一个名字叫做 example-service 并且指向 mockbin 的服务

mockbin 请求什么返回什么的网站 mock 数据用

`curl -i -X POST --url http://localhost:8001/services/ --data 'name=example-service' --data 'url=http://mockbin.org'`

返回一个蕾丝的响应, 并且把 url 中的元素解析成 host, proto, port, 

```html
HTTP/1.1 201 Created
Content-Type: application/json
Connection: keep-alive

{
   "host":"mockbin.org",
   "created_at":1519130509,
   "connect_timeout":60000,
   "id":"92956672-f5ea-4e9a-b096-667bf55bc40c",
   "protocol":"http",
   "name":"example-service",
   "read_timeout":60000,
   "port":80,
   "path":null,
   "updated_at":1519130509,
   "retries":5,
   "write_timeout":60000
}
```

### 添加服务路由

对名字为 example-service 的服务添加路由:

`curl -i -X POST --url http://localhost:8001/services/example-service/routes --data 'hosts[]=example.com'`  
host 是一个列表结构,应该是可以一次添加多个域名, dashboard 中可以直接写上多个域名, cli 中应该也是可以的

返回类似的东西:

```html
HTTP/1.1 201 Created
Content-Type: application/json
Connection: keep-alive

{
   "created_at":1519131139,
   "strip_path":true,
   "hosts":[
      "example.com"
   ],
   "preserve_host":false,
   "regex_priority":0,
   "updated_at":1519131139,
   "paths":null,
   "service":{
      "id":"79d7ee6e-9fc7-4b95-aa3b-61d2e17e7516"
   },
   "methods":null,
   "protocols":[
      "http",
      "https"
   ],
   "id":"f9ce2ed7-c06e-4e16-bd5d-3a82daef3f9d"
}
```

### 通过Kong转发您的请求

curl -X 指定请求方法, -i 现实请求头, --url 指定请求url, --header 添加请求头  
`curl -i -X GET --url http://localhost:8000/ --header 'Host: example.com'`  
`curl -i -X GET --url http://localhost:8000/request\?foo\=bar\&foo\=baz --header 'Host: example.com'`

## plugin 暂时跳过

主要介绍了一个 auth 库, 每次请求都需要携带认证密码

## 增加消费者

需要添加了 auth 库的, 跳过

## 配置文件参考

[https://docs.konghq.com/1.1.x/configuration/](https://docs.konghq.com/1.1.x/configuration/) longlong 长

配置加载

模版文件: `/etc/kong/kong.conf.default` 使用时复制一份到 `/etc/kong/kong.conf`  
kong 还可能会加载 `/etc/kong/kong.conf` `/etc/kong.conf` 位置的配置

使用 `-c or --conf` 手动指定配置文件

跟 nginx 一样,kong 也提供了检查配置文件的 `kong check [/etc/kong/kong.conf]` 检查配置, 默认 `/etc/kong/kong.conf` 的配置文件

调试模式  
`kong start -c [/etc/kong/kong.conf] --vv`

谷歌翻译翻译的挺准的 也太长, 不翻译了, 过

介绍了 如何自定义配置文件, 在自己的 nginx 上配置 kong, 一些kong 及 nginx的配置选项, 数据库的配置, 

## CLI参考

kong的命令行参考

`kong check`

`kong config`

`kong health`

`kong migrations`

`kong prepare`

`kong quit`

`kong reload`

`kong restart`

`kong start`

`kong stop`

`kong version`

## 代理参考

## Admin API参考

介绍了 8001 admin端口的一些返回接口


## 群集参考

