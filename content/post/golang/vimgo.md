---
title: "Vim-go配置"
date: 2021-04-21T17:30:33+08:00
lastmod: 2021-04-21T17:30:33+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["golang","go","vim"]
categories: ["Go","linux"]
author: "王清"
---

## golang vim

### vim-go

### vim-youcompleteme

```
sudo apt install vim-youcompleteme

vim-addon-manager install youcompleteme

The ubuntu version does not support Java so you may want the latest version depending on your language of choise, so alternativly;
```

```
cd ~/.vim/bundle

git clone --depth=1 https://github.com/Valloric/YouCompleteMe.git

cd YouCompleteMe

git submodule update --init --recursive

./install.py --all
```
