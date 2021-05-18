---
title: "苹果系统美化"
date: 2021-04-21T18:27:43+08:00
lastmod: 2021-04-21T18:27:43+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["macos","hackintonish"]
categories: ["windows"]
author: "王清"
---

每次安装完系统配置环境都是一个极为耗费时间的时期，特此写一篇文章记录一下macOS的常用配置，为后面再次配置少走一些弯路。

# 系统设置

## 常用设置

Bash

```bash
# 取消4位数密码限制 
pwpolicy -clearaccountpolicies

# 更改密码
passwd

# SSD 开启 TRIM 支持
sudo trimforce enable

# APP安装开启任何来源
sudo spctl --master-disable

# 修改主机名
sudo scutil --set HostName xxxxx

# 修改共享名称
sudo scutil --set ComputerName xxxxx

# 安装 xcode 命令行工具
xcode-select --install
```

## 关闭SIP

后面不关闭SIP的话`proxychains-ng`这种代理神器就无法使用。

重启Mac，按住Option键进入启动盘选择模式，按`⌘` +`R`进入Recovery模式。

菜单栏 -> 实用工具（Utilities）-> 终端（Terminal）。

```bash
# 关闭SIP
csrutil disable

# 查看SIP状态
csrutil status

System Integrity Protection status: disabled.(表明关闭成功)
```

# 软件相关

## iTerm2

### Apperance

窗口主题和标签页的停靠位置，这里我设置的左边，iTerm2标签页放在顶部有时候很容易误关！！


![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/a5523b36-eb56-47fd-8f92-e9c811a8d1aa.jpg)


### Profiles

新建一个自己的配置，字体相关设置：


![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/eb94e283-eb45-414a-87d7-2f9658ec8f85.jpg)

iTerm2终端窗口大小、背景透明程度、背景毛玻璃虚化程度：

![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/cfe1b1c7-e523-4a34-859d-80afd469a6da.jpg)

### 配置效果

![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/027daa5c-a4fb-4e12-93d6-945a73eae0b9.jpg)

# 开发相关

## Homebrew

*Homebrew* 是一款自由及开放源代码的软件包管理系统，用以简化macOS系统上的软件安装过程，程序猿开发必备神器。

### 安装

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### brew 加速

brew 和 brew cask 慢或不动的情况也会经常发生，这个时候可以考虑手动更换homebrew国内源来实现加速

**替换现有上游**

```bash
# 替换成国内源：
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
```

**替换Homebrew Bottles源**

- **zsh**用户

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc

source ~/.zshrc

# 再更新一下试试看效果 注意网速 应该可以跑满
brew update
```

- **bash**用户

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile

cd ~
source ~/.bash_profile

# 再更新一下试试看效果 注意网速 应该可以跑满
brew update
```

### brew 重置恢复默认

手残搞错了源也不要紧，依次执行下面命令可以将其重置回来

**重置brew.git**

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

brew update
```

**手动删除shell配置记录**

- **zsh**用户

编辑`~/.zshrc`配置文件，删除之前添加的`export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles`记录

- **bash**用户

编辑`~/.bashrc`配置文件，删除之前添加的`export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles`记录

### brew常用软件

Bash

```bash
# brew 相关权限设置
sudo chown -R $(whoami) /usr/local/share/man/man8
chmod u+w /usr/local/share/man/man8

# ip 命令 查看ip地址很方便
brew install iproute2mac

# 查看主机配置信息
brew install screenfetch

# 查看主机配置信息
brew install neofetch
```

## proxychains

终端命令行下代理神器，可以让指定的命令走设置好的代理，内网渗透、科学上网必备工具：

Bash

```bash
# 安装
brew install proxychains-ng

# 配置文件
vim /usr/local/etc/proxychains.conf
```

将结尾的 `socks4 127.0.0.1 9095`改为

```
socks5 127.0.0.1 1086
```

> 国光的macOS下SSR的默认代理为1086 然后我就习惯了1086端口 这里根据自己的情况来设置

**常用方法**

Bash

```bash
# 代理终端命令
proxychains4 curl www.google.com

# 全局代理Bash shell
proxychains4 -q /bin/bash

# 全局代理zsh shell
proxychains4 -q /bin/zsh
```

## Vim

macOS自带vim命令，这里主要就是开启macOS vim下自带的一个配色：

```bash
# 查看 vim 版本
vim --version

# 这样查看版本号也可以
ls /usr/share/vim/

# 查看自带的配色方案  vim80 当前vim 的版本号（macOS Mojave）
ls /usr/share/vim/vim80/colors
```

自带了如下的配色：

```bash
default.vim   elflord.vim   koehler.vim   pablo.vim     shine.vim     zellner.vim
blue.vim      delek.vim     evening.vim   morning.vim   peachpuff.vim slate.vim
darkblue.vim  desert.vim    industry.vim  murphy.vim    ron.vim       torte.vim
```

配色可以自己一个个尝试一下，下面新建一个vim的配置文件：

```bash
vim ~/.vimrc
```

内容如下：

```bash
set nu                " 显示行号
colorscheme desert    " 颜色显示方案
syntax on             " 打开语法高亮
```

配置修改为完，输入`zsh`命令生效配置

## Oh My Zsh

macOS下默认的命令行shell环境为Bash，同时macOS也自带zsh shell环境，这个Oh My Zsh 是一个zsh美化增强插件，可以让自己的shell颜值更逼格。

### 安装

```bash
# 可选命令：全局代理(速度更快)
proxychains4 -q /bin/bash

# curl安装
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 主题

```bash
#修改配置文件
vim ~/.zshrc
```

修改此行（等于号 后面填写自己的主题）：

```bash
ZSH_THEME=robbyrussell
```

Oh My Zsh已经内置了很多主题了：

```bash
# 查看自带的主题
ls -l ~/.oh-my-zsh/themes
```

国光以前也比较喜欢花里胡哨的折腾这些，后来发现画船听雨大佬的zsh主题就是默认的=，= 后面我就也没有再折腾这些花里胡哨的主题了。

接下来主要就是一些相关插的配置了。

### autojump

目录切换神器，大大提高工作效率。

```bash
# macOS 安装
brew install autojump
```

在 `~/.zshrc` 中配置

```
plugins=(其他的插件 autojump)
```

输入`zsh`命令生效配置后即可正常使用`j`命令，下面是简单的演示效果：

```bash
# 第一次 cd 进入某个目录
➜  ~ cd Documents/Hexo
➜  Hexo cd ~

# 后面就可以直接通过 j 命令跳转到那个目录
➜  ~ j hexo
/Users/sec/Documents/Hexo
```

### autosuggestions

终端下自动提示接下来可能要输入的命令，这个实际使用效率还是比较高的

```bash
# 拷贝到 plugins 目录下
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

在 `~/.zshrc` 中配置：

```bash
plugins=(其他的插件 zsh-autosuggestions)
```

输入`zsh`命令生效配置

### zsh-syntax-highlighting

命令输入正确会绿色高亮显示，可以有效地检测命令语法是否正确

```bash
# 拷贝到 plugins 目录下
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在 `~/.zshrc` 中配置：

```bash
plugins=(其他的插件 zsh-syntax-highlighting)
```

输入`zsh`命令生效配置

## Git

### 配置邮箱和用户名

Github 会根据这个邮箱显示对应的commit记录的头像

```bash
# 配置邮箱 
git config --global user.email "xxxxx@xxx.com"

# 配置用户名
git config --global user.name "国光"
```

下图中间两个commit记录没有头像的原因就是没有配置上述邮箱的问题造成的：



![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/a7d2fd31-e811-4c1a-8a96-5c4058c47a53.jpg)



### 配置socks5代理

在天朝 使用 git clone 的正确姿势，配置这个代理的前提是自己得有一个可以访问国外的Socks5代理（比较敏感的话题 这里就不多说了）

```bash
# 我这里习惯用1086端口 具体根据自己的配置来灵活设置
git config --global http.proxy 'socks5://127.0.0.1:1086'
git config --global https.proxy 'socks5://127.0.0.1:1086'

# 取消git代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看 git 配置
cat ~/.gitconfig

# 或者这样也可以查看git的配置
git config -l
```

## Python

macOS自带Python 2.7的版本，而且没有安装pip，得自己再折腾完善一下。

### pip2

macOS里面Python自带easy_install，用来安装pip2非常方便

```bash
# 安装pip2
sudo easy_install pip

# 查看pip的版本
pip --version
```

### Python3 & pip3

使用brew命令安装python3与pip3，一条龙服务，安装起来很方便

```bash
# 安装Python3相关
brew install python

# 查看Python版本
python3 --version

# 查看pip的版本
pip3 --version
```

### ipython3

Python要被淘汰掉了，ipython这种辅助工具包的话 安装一个ipython3的 基本上可以满足日常的调试需求了

```bash
# 安装ipython3
pip3 install ipython
```

## VS Code

VS Code 用的不多，所以就没有折腾太多插件，只是做一个基本的插件安装。

**Chinese (Simplified) Language Pack for Visual Studio Code**：官方中文插件

**VSCode Great Icons**：文件夹图标主题

**filesize**：左下角显示文件大小

**ASCIIDecorator**：常用来生成一些酷炫的logo

**Vibrancy**：VS Code毛玻璃效果插件

**Terminal**：命令行爱好者必备插件

**Beautify**：代码格式美化插件

**Excel Viewer**：CSV 表格内容显示插件

**Diff**：比较2个文件的区别

**kite**：Python 语句补全插件 自动语义提示效率很高（得配合Kite这个APP来使用）

上一个默认代码配色的效果图：


![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/3752ccf0-330a-4adf-8ddf-4c77d3c0aa45.jpg)

## Atom

第一次启动Atom可能会出现如下的报错：

```
The package `spell-check` cannot load the system dictionary for `zh-CN`. See the settings for ways of changing the languages used, resolving missing dictionaries, or hiding this warning.
```

这个是Atom拼写检查插件报错，解决方法很简单 `Packages / Spell Check` 去掉勾选 **Use Locales**

### 字体

国光习惯编辑器使用的是`IBMPlexMono`字体

谷歌字体详情链接：https://fonts.google.com/specimen/IBM+Plex+Mono

`Settings`-`Editor`-`Font Family`中Atom默认的字体为：

```
Menlo, Consolas, DejaVu Sans Mono, monospace
```

国光安装第三方字体手设置了为：

```
IBMPlexMono-MediumItalic
```

> 实际上 macOS自带的字体 Monaco 也是很不错的

### 标题栏

```
Settings`-`Core`-`Title Bar`，国光这里为了颜值将其设置为了`hidden
```

这样就把Atom的标题栏隐藏起来了，整体看上去更加清爽一点。

### 插件

Atom插件安装慢的话 可以手动使用`apm`配合`proxychains4`命令来安装，安装成功后重启，这样总的下来速度会快很多：

\####activate-power-mode

只勾选`Auto Toggle`、`Particles - Enabled`、`Enable`，其他全部取消勾选。

在`Particles Colours`下面选择`Colour at the cursor`表示烟花效果的颜色是光标所在的颜色

这样在编码的时候就会直接触发烟花效果了，而且不会有烦人的屏幕震动和码字速度统计。

#### 其他插件

**kite**：Python 语句补全插件 自动语义提示效率很高 （得配合Kite这个APP来使用）

**atom-beautify**：atom 代码格式美化插件

**atom-clock**：底部状态栏时钟插件

**platformio-ide-terminal**：Atom必备终端命令行插件 颜值也很高

**minimap**：编辑器右侧代码小地图

**minimap-find-and-replace**：代码小地图查找和替换高亮

**highlight-selected**：鼠标高亮选择

**minimap-highlight-selected**：代码小地图中鼠标选择高亮，依赖 **highlight-selected**插件

**busy-signal**：和 linter插件配套使用

**linter-ui-default**：和 linter插件配套使用，依赖**intentions**, **busy-signal**

**linter**：代码法检测

**intentions**：和 linter插件配套使用

**ide-python**：Python 代码检测

**one-vibrancy**：Atom毛玻璃透明效果

![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/29117fa6-7733-40e0-b3d1-f70caecc5f40.jpg)

## Sublime Text3

### 激活

编辑hosts文件：

```bash
sudo vim /etc/hosts
```

插入如下内容：

```bash
# Sublime Text3 3211
127.0.0.1 www.sublimetext.com
127.0.0.1 license.sublimehq.com
```

修改完Hosts文件后即可直接使用网络上网友分享的license激活：

```bash
----- BEGIN LICENSE -----
Member J2TeaM
Single User License
EA7E-1011316
D7DA350E 1B8B0760 972F8B60 F3E64036
B9B4E234 F356F38F 0AD1E3B7 0E9C5FAD
FA0A2ABE 25F65BD8 D51458E5 3923CE80
87428428 79079A01 AA69F319 A1AF29A4
A684C2DC 0B1583D4 19CBD290 217618CD
5653E0A0 BACE3948 BB2EE45E 422D2C87
DD9AF44B 99C49590 D2DBDEE1 75860FD2
8C8BB2AD B2ECE5A4 EFC08AF2 25A9B864
------ END LICENSE ------
```

该license亲测适用于Sublime Text3 3.2.2,Build 3211版本

### 插件

首先得安装Package Control，然后才可以愉快地安装插件，安装方法很简单`菜单栏`-`Tools`-`Install Package Control`，如果安装卡顿的话 开一个全局代理的话速度会快很多，这里不再赘述。安装成功后可以在`菜单栏`-`Sublime Text`-`Preferences`-`Package Control`-输入`Install Package`，等待左下角进度加载完即可安装任意插件。

> 吐槽一下 安装插件这么常用的功能 为什么要做的这么复杂 这让小白该怎么办 =,=

Sublime Text3 国光也就经常编辑文本使用，所以插件安装的不多，下面只列出了一些国光自己用的插件，大家也可以去官网自己搜索想要的插件：[https://packagecontrol.io](https://packagecontrol.io/)

**ChineseLocalizations**：Sublime Text中文汉化插件

**A File Icon**：文件图标插件，提高颜值

**kite**：Python 语句补全插件 自动语义提示效率很高 （得配合Kite这个APP来使用）

## Gitbook

首先得安装`npm`命令，[官网](https://nodejs.org/zh-cn/)直接下载 pkg 安装包 安装即可：

```bash
# 安装Gitbook
➜  ~ sudo npm install -g gitbook-cli

# 验证是否安装成功
➜  ~ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

### 基本命令

下面是Gitbook常用的一些命令：

```bash
# Gitbook 目录结构初始化
gitbook init

# 本地预览
gitbook serve

# Gitbook指定端口启动Web服务
gitbook serve --port 8000

# 生成静态html
gitbook build

# 生成pdf文件（得安装配置好 https://calibre-ebook.com/download）
gitbook pdf
```

### 文章结构

**README.md 与 SUMMARY.md编写**

- README.md

一本Gitbook的简介

```
# Gitbook 使用入门


> GitBook 是一个基于 Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书。

本书将简单介绍如何安装、编写、生成、发布一本在线图书。
```

- SUMMARY.md

一本书的目录结构

```
# Summary

* [Introduction](README.md)
* [基本安装](howtouse/README.md)
   * [Node.js安装](howtouse/nodejsinstall.md)
   * [Gitbook安装](howtouse/gitbookinstall.md)
   * [Gitbook命令行速览](howtouse/gitbookcli.md)
* [图书项目结构](book/README.md)
   * [README.md 与 SUMMARY编写](book/file.md)
   * [目录初始化](book/prjinit.md)
* [图书输出](output/README.md)
   * [输出为静态网站](output/outfile.md)
   * [输出PDF](output/pdfandebook.md)
* [发布](publish/README.md)
   * [发布到Github Pages](publish/gitpages.md)
* [结束](end/README.md)
```

### 插件相关

**基本插件安装**

下面演示一下 `侧边栏宽度可调节`插件的安装，后面就不再赘述了。

项目地址：https://github.com/yoshidax/gitbook-plugin-splitter

在Gitbook项目的根目录下建立`book.json`文件，内容如下：

```json
{
    "plugins": ["splitter"]
}
```

然后执行

```
gitbook install
```

即可完成插件的安装

![img](https://image.3001.net/images/20200115/15790595345170.jpg)

**gitbook-plugin-search-plus**

简介：支持中文搜索

项目地址：https://github.com/lwdgit/gitbook-plugin-search-plus

```json
{
    plugins: ["-lunr", "-search", "search-plus"]
}
```

**gitbook-plugin-anchor-navigation-ex**

简介：文章目录TOC导航以及返回顶部

项目地址：https://github.com/zq99299/gitbook-plugin-anchor-navigation-ex

```json
{
  "plugins": ["anchor-navigation-ex"]
}
```

因为该插件会自动生成自己的目录层级序号，国光本人喜欢自己来分层，所以这里需要配置一下关闭这个功能：

```json
"pluginsConfig": {
  "anchor-navigation-ex": {
      "showLevel": false
  }
}
```

**toggle-chapters**

简介：Gitbook的左侧目录折叠

项目地址：https://github.com/poojan/gitbook-plugin-toggle-chapters

```json
{
    "plugins": ["toggle-chapters"]
}
```

### book.json

经过上面的简单配置后，我的book.json具体内容如下：

```json
{
  "title": "Python学习记录",
    "author": "国光",
    "language": "zh-hans",
    "links": {
        "sidebar": {
          "个人博客": "https://www.sqlsec.com"
        }
      },
    "plugins": ["splitter","-lunr", "-search", "search-plus","anchor-navigation-ex","toggle-chapters"],
    "pluginsConfig": {
        "anchor-navigation-ex": {
            "showLevel": false
        }
    }
}
```

# 安全相关

macOS下也有很多原生的渗透测试工具，安装配置起来比Windows都要方便很多。

## binwalk

文件分析工具，旨在协助研究人员对文件进行分析，提取及逆向工程，CTF比赛中常用来解隐写相关的题目。

```bash
# macOS安装binwalk
➜  ~ brew install binwalk

# 查看版本信息
➜  ~ binwalk

Binwalk v2.2.0
Craig Heffner, ReFirmLabs
https://github.com/ReFirmLabs/binwalk

Usage: binwalk [OPTIONS] [FILE1] [FILE2] [FILE3] ...
```

## burpsuite

Web安全抓包改包必备神器，具体配置可以参考我的这篇文章：[macOS下如何优雅的使用Burp Suite](https://www.sqlsec.com/2019/11/macbp.html)



![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/9fba59d8-c066-4f0e-af50-43619d7e995d.jpg)



## dirsearch

Web目录扫描神器，自带的字典很强大，逐渐替代了御剑等一些工具。

```bash
# 进入安全工具的目录下（国光习惯将安全工具放在 Documents 目录下）
➜ cd /Users/sec/Documents/Sec

# git clone 下载最新版本的 dirsearch
➜ git clone https://github.com/maurosoria/dirsearch.git
Cloning into 'dirsearch'...
remote: Enumerating objects: 1744, done.
remote: Total 1744 (delta 0), reused 0 (delta 0), pack-reused 1744
Receiving objects: 100% (1744/1744), 17.73 MiB | 1.26 MiB/s, done.
Resolving deltas: 100% (1015/1015), done.

# 创建 dirsearch 的快捷方式
➜ ln -s /Users/sec/Documents/Sec/dirsearch/dirsearch.py /usr/local/bin/dirsearch

# 刷新 zsh
➜ zsh

# 发起一个基本扫描
➜ dirsearch -u https://www.baidu.com -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: Applications | HTTP method: get | Threads: 10 | Wordlist size: 6104

Error Log: /Users/sec/Documents/Sec/dirsearch/logs/errors-19-12-26_22-09-24.log

Target: https://www.baidu.com

[22:09:24] Starting:
[22:09:24] 302 -  231B  - /.bash_history.php  ->  http://www.baidu.com/forbiddenip/forbidden.html
[22:09:24] 302 -  231B  - /.env.php  ->  http://www.baidu.com/forbiddenip/forbidden.html
[22:09:24] 302 -  231B  - /.env.sample.php  ->  
...
```

## hashcat

密码破解神器，支持GPU破解

```bash
# macOS安装hashcat
➜  ~ brew install hashcat

# 查看版本信息
➜  ~ hashcat --version
v5.1.0
```

## masscan

世界上最快的端口扫描器，一般和nmap配合使用 可以极大地提高效率

```bash
# macOS安装masscan
➜  ~ brew install masscan

# 查看版本信息
➜  ~ masscan --version

Masscan version 1.0.4 ( https://github.com/robertdavidgraham/masscan )
Compiled on: Aug 21 2018 02:07:01
Compiler: gcc 4.2.1 Compatible Apple LLVM 10.0.0 (clang-1000.10.43.1)
OS: Apple
CPU: unknown (64 bits)
GIT version: unknown
```

## metasploit

黑客TOP10工具之一，漏洞攻击框架，必备神器之一，macOS下配置可以参考我的这篇文章[macOS下优雅的使用Metasploit](https://www.sqlsec.com/2019/11/macmsf.html)

```bash
# 查看版本信息
➜  ~ msfconsole -V
Framework Version: 5.0.66-dev-75dc82f764050ff800dc2bf0482c31aae693edd4

➜  ~ msfconsole
 ______________________________________________________________________________
|                                                                              |
|                          3Kom SuperHack II Logon                             |
|______________________________________________________________________________|
|                                                                              |
|                                                                              |
|                                                                              |
|                 User Name:          [   security    ]                        |
|                                                                              |
|                 Password:           [               ]                        |
|                                                                              |
|                                                                              |
|                                                                              |
|                                   [ OK ]                                     |
|______________________________________________________________________________|
|                                                                              |
|                                                       https://metasploit.com |
|______________________________________________________________________________|


       =[ metasploit v5.0.66-dev-75dc82f764050ff800dc2bf0482c31aae693edd4]
+ -- --=[ 1956 exploits - 1091 auxiliary - 336 post       ]
+ -- --=[ 558 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

msf5 >
```

## nmap

端口扫描必备神器

```bash
# macOS安装nmap
➜  ~ brew install nmap

# 查看版本信息
➜  ~ nmap -v
Starting Nmap 7.80 ( https://nmap.org ) at 2019-12-26 21:07 CST
Read data files from: /usr/local/bin/../share/nmap
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.03 seconds
```

## sqlmap

Web安全必备工具，SQL注入神器

```bash
# 下载可能会卡 请自行备好代理
➜  ~ brew install sqlmap
```



![img](https://image.3001.net/images/20191226/15773682044024.jpg)



如果安装卡这里的话，通过`proxychains4`也没有解决的话 ，国光我这里是这样临时解决的：

编辑 zsh 配置文件

```bash
vim .zshrc
```

添加如下内容：

```bash
# socks5 proxy
export ALL_PROXY=socks5://127.0.0.1:1086
```

配置修改为完，输入`zsh`命令生效配置

## wpscan

WordPress 漏洞扫描器，收录了历届WordPress漏洞，可以辅助渗透测试一些WordPress相关的站点

```bash
# 部署可能会卡 请自行备好代理
➜  ~ brew install wpscan

# 查看 wpscan 版本信息
➜  ~ wpscan --version
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.4
          Sponsored by Sucuri - https://sucuri.net
      @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

Current version: 2.9.4
Last database update: 2019-12-26
```


来源: 国光
文章作者: 国光
文章链接: https://www.sqlsec.com/2019/12/macos.html
咳咳又想白嫖文章？本文章著作权归作者所有，任何形式的转载都请注明出处。
