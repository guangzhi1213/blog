---
title: "Openresty"
date: 2021-04-21T18:01:47+08:00
draft: false
tags: ["linux","sre","ops","nginx","openresty"]
categories: ["linux"]
---

## limit 模块

```shell
   geo $whiteiplist {
        default 1;
        proxy 100.0.0.0/8;
        183.6.185.234 0;
        183.233.186.71 0;
        1.119.197.112/29 0;
        172.16.0.0/12 0;
        127.0.0.1 0;
    }
    map $http_x_forwarded_for  $clientRealIP {
        "" $remote_addr;
        ~^(?P<firstAddr>[0-9\.]+),?.*$ $firstAddr;
    }
    map $whiteiplist $limit {
        1 $clientRealIP;
        0 "";
    }

    limit_conn_zone $limit zone=address:10m;
    limit_conn   address  10;
    limit_req_zone  $limit zone=request:20m rate=12r/s;
    limit_req   zone=request burst=120 nodelay;
```
