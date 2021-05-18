---
title: "Saltstack"
date: 2021-04-21T18:25:07+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]
---

# saltstack

## salt doc

- [salt示例](https://www.tutorialspoint.com/saltstack/index.htm)
- [salt官方文档](https://docs.saltstack.com/en/latest/ref/states/all/salt.states.ssh_auth.html)
- [salt module](https://github.com/saltstack-formulas)

## saltstack 安装

### uninstall in macos

> /opt/salt/bin/salt-call

It looks like you installed it using `.pkg` file: https://repo.saltstack.com/osx/

You can verify by running some commands:

```
❯❯❯❯ pkgutil --pkgs | grep salt
com.saltstack.salt

❯❯❯❯ pkgutil --pkg-info com.saltstack.salt
package-id: com.saltstack.salt
version: 2016.11.3
volume: /
location: /
install-time: 1504874150
```

List installed files:

```
❯❯❯❯ pkgutil --files com.saltstack.salt
Library
Library/LaunchDaemons
Library/LaunchDaemons/com.saltstack.salt.api.plist
Library/LaunchDaemons/com.saltstack.salt.master.plist
Library/LaunchDaemons/com.saltstack.salt.minion.plist
Library/LaunchDaemons/com.saltstack.salt.syndic.plist
etc
etc/salt
etc/salt/master.dist
etc/salt/minion.dist
opt
opt/salt
opt/salt/bin
...
```

Stop services:

```
❯❯❯❯ sudo launchctl unload -w /Library/LaunchDaemons/com.saltstack.salt.minion.plist
```

Do the same for `api`, `master`, `syndic` if they are running.

Remove files first:

```
❯❯❯❯ cd /

❯❯❯❯ pkgutil --only-files --files com.saltstack.salt | grep -v opt
Library/LaunchDaemons/com.saltstack.salt.api.plist
Library/LaunchDaemons/com.saltstack.salt.master.plist
Library/LaunchDaemons/com.saltstack.salt.minion.plist
Library/LaunchDaemons/com.saltstack.salt.syndic.plist
etc/salt/master.dist
etc/salt/minion.dist

❯❯❯❯ pkgutil --only-files --files com.saltstack.salt | grep -v opt | tr '\n' '\0' | xargs -0 sudo rm -f
```

then directories:

```
❯❯❯❯ pkgutil --only-dirs --files com.saltstack.salt | grep -v opt
Library
Library/LaunchDaemons
etc
etc/salt

❯❯❯❯ sudo rm -fr etc/salt

❯❯❯❯ sudo rm -fr opt/salt
```

And finally, remove the receipt:

```
❯❯❯❯ sudo pkgutil --forget com.saltstack.salt
Forgot package 'com.saltstack.salt' on '/'.
```

## master迁移

**今天遇到了删除主节点然后重建的需求。。mmp**

新机器按照文档都可以顺利配置成功， 但是旧机器原先已经在salt里边了，需要删除相应的pki密钥

就是说 需要minion节点删除 `rm -rf /etc/salt/pki/minion/minion_master.pub`

然后在主节点上 添加其密钥 `salt-key -a xxxxx`，测试 `salt '*' test.ping` 就能成功了

## minion迁移

停止 `salt-minion` 

在 `master` 上删除认证信息 `salt-key -d xxx`, 

在 `minion` 节点上删除 `/etc/salt/minion.d/*` 和 `/etc/salt/pki/*`, 然后重启

在 `master` 节点上添加认证 `salt-key -a xxx`