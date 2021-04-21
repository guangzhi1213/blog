---
title: "Regexp"
date: 2021-04-21T17:51:06+08:00
lastmod: 2021-04-21T17:51:06+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["regex","shell","正则表达式"]
categories: []
author: "王清"
---

# 正则表达式

## 去除代码中的注释

`(?<!:)\/\/.*|\/\*(\s|.)*?\*\/`

可以去除多行注释, 代码片段的注释, 行结尾注释

还有一个学习正则的网站 [regex101](https://regex101.com)
