---
title: "使用cobra新建一个gomodule项目"
date: 2021-04-21T16:38:24+08:00
lastmod: 2021-04-21T16:38:24+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["golang","go","cobra"]
categories: ["Go"]
author: "王清"
---

# 创建一个新的Go项目

## 使用cobra初始化项目目录

安装cobra： `go get -u github.com/spf13/cobra/cobra`

命令： `cobra init logipv4geo --pkg-name 91fun.club/manjoc/logipv4geo` 

在当前目录下创建一个文件夹 logipv4geo ， 其中包名叫 `91fun.club/manjoc/logipv4geo`

目录结构入下：

```shell
~]$ tree logipv4geo 
logipv4geo
├── cmd
│   └── root.go
├── LICENSE
└── main.go
```

## 创建 `go mod` 类型的项目

初始化一下项目： `go mod init 91fun.club/manjoc/logipv4geo` ,生成go.mod

```shell
(base) ➜  logipv4geo cat go.mod 
module 91fun.club/manjoc/logipv4geo

go 1.16
```

下载依赖：`go mod tidy`

生成 go.mod:

```shell
module 91fun.club/manjoc/logipv4geo

go 1.16

require (
        github.com/mitchellh/go-homedir v1.1.0
        github.com/spf13/cobra v1.1.3
        github.com/spf13/viper v1.7.1
)
```

同时会更新 `go.sum`。
