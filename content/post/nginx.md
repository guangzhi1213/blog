---
title: "Nginx"
date: 2021-04-25T11:12:48+08:00
lastmod: 2021-04-25T11:12:48+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## nginx

- 日志
  - [goaccess解析nginx日志](https://goaccess.io/man#custom-log)
- [collected 日志收集](https://collectd.org/documentation/manpages/collectd.conf.5.shtml)
  
- kong
  - [konga gui](https://pantsel.github.io/konga/)
  - [添加消费者](https://docs.konghq.com/0.14.x/getting-started/adding-consumers/) 官方文档
- 限流
  - [nginx限流的几种方式](http://blog.51cto.com/zhweizhi/2063157)
  - 

- 安全
  - [nginx mod security](https://ngx.hk/2017/09/05/nginx%e7%bc%96%e8%af%91%e5%ae%89%e8%a3%85modsecurity%e4%bd%9c%e4%b8%ba%e9%ab%98%e6%80%a7%e8%83%bdwaf.html) modsecurity 模块安装

### 在nginx端开放处理设置

```nginx
# Wide-open CORS config for nginx
#
location / {
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     }
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     }
}
```
