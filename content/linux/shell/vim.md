---
title: "Vim"
date: 2021-04-25T11:57:28+08:00
lastmod: 2021-04-25T11:57:28+08:00
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

# vim编辑器

文本编辑器： 纯文本, ASCII text; Unicode 

? 普通用户保存需要root权限的文件方法？

## 模式化的编辑器

### 基本模式

* 编辑模式, 命令模式
* 输入模式
* 末行模式

### 打开文件

<pre>
# vim [option] [file ...]
    +#: 打开文件后，直接让光标出于第#行行首
    +/PATTERN: 打开文件后，直接让光标处于第一个被PATTERN 匹配到的行的行首
</pre>

### 模式转换

* 默认模式： 编辑模式
* 编辑模式 --> 输入模式:
    <pre>
    i: insert, 在光标所在处输入
    a: append, 在光标在处后方输入
    o: 在光标所在处的下方打开一个新行
    I: 在光标所在行的行首输入
    A: 在光标所在行的行位输入
    O: 在光标所在处的上方打开一个新行
    </pre>
* 输入模式 --> 编辑模式
    ESC
* 编辑模式 --> 末行模式
    :
* 末行模式 --> 编辑模式
    ESC

### 关闭文件

* `ZZ`: 保存并退出
* `:q`: 退出
* `q!`: 强制退出，不保存此前的编辑操作
* `wq`: 保存并退出
* `x`: 保存并退出
* `w /PATH/TO/SOMEFILE`: 保存至某文件

### 光标跳转

#### 字符间跳转

* `h`: 
* `j`:
* `k`:
* `l`:

* `#COMMAND`: 跳转至#个数的字符

#### 单词间跳转

* `w`: 下一个单词词首
* `e`: 当前或后一个单词的词尾
* `b`: 当前或前一个单词词首

* `#COMMAND`: 跳转至#个数的单词

#### 行首行尾跳转

* `^`: 跳转至行首的第一个非空白字符
* `0,Home`: 跳转至行首
* `$,End`: 跳转至行尾

#### 行间跳转

* `#G,gg`: 跳转至#行
* `1G,gg`: 跳转至第一行
* `G`: 跳转至最后一行

#### 句间跳转

* `)`:
* `(`:

##### 段间跳转

* `}`:
* `{`:

### 翻屏

* `Ctrl+f`: 向文件尾翻一屏
* `Ctrl+b`: 向文件首部翻一屏
* `Ctrl+d`: 向文件尾部翻半屏
* `Ctrl+u`: 向文件首部翻半屏
* `Enter`: 按行向后翻

### vim的编辑命令

#### 字符编辑

* `x`: 删除光标所在处的字符
* `#x`: 删除光标所在处起始位置的#个字符

* `xp`: 交换光标所在处的字符与其后面的字符的位置

* `D`: 删除当前行从当前位置至行尾部分

#### 替换命令

* `r`：替换光标所在处的字符，只能替换一次
* `R`: 替换光标所在处的字符，保持替换模式，直到ESC退出

#### 删除命令

* `d`: 删除命令, 可结合光标跳转字符, 实现范围删除
* `d$`:
* `d^`:
* `dw`:
* `de`:
* `db`:
* `dd`:

#### 搜索替换

`:s/a1/a2/g` : 将当前行的a1 替换为 a2

`:n1,n2s/a1/a2/g` : 将第n1到n2 行的a1替换为a2

`:g/a1/a2/g` : 将文件中所有的a1替换为a2

