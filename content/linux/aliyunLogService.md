---
title: "AliyunLogService"
date: 2021-04-21T17:46:09+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]
---

# 阿里云 logservice 服务

## 常见查询语句

`* AND host = "%s" and status in [200 399) | select remote_addr as ip,count(1) as pv group by ip order by pv desc limit %d`
