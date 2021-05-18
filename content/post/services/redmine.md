---
title: "Redmine"
date: 2021-04-25T11:18:26+08:00
lastmod: 2021-04-25T11:18:26+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## 安装中出现的问题

### gem install 问题

* 修改 镜像为国内源, `gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/` 执行 `gem sources -l` 查看当前 source ,确保只有国内源
* `gem install rails`
* 如果是 rail 项目， 修改 Gemfile 或者 `bundle config mirror.https://rubygems.org https://gems.ruby-china.org` 
* ruby 源代码下载镜像 `cache.ruby-lang.org`
* `sed -i -E 's!https?://cache.ruby-lang.org/pub/ruby!https://ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db` 修改 RVM ，改用本站作为下载源，提高安装速度
* 编译安装 `Imagemagick`
  
  # 安装依赖包 zlib-devel libtool-ltdl-devel
  ./configure --prefix=/usr/local/ImageMagick/ --enable-lzw --with-modules --with-quantum-depth=8?--enable-shared --disable-openmp



REDMINE是用RUBY开发的基于WEB的项目管理软件，提供项目管理、WIKI、新闻台等功能，可对接版本管理系统GIT、SVN、CVS等获取版本变更信息。通过WEB 形式把成员、任务、文档、讨论以及各种形式的资源组织在一起，推动项目的进度

## 

### 安装指导

- [官方指导](http://www.redmine.org/projects/redmine/wiki/Guide)
- [readmine 安装](http://www.cnblogs.com/wzy5223/p/5339563.html) : 未来战士博客园
- [前端使用apache mod_fcgid模式](http://www.cnblogs.com/wzy5223/p/5335549.html): 修改前段展示方式，正式需要这样弄
