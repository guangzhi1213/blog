---
title: "Redis"
date: 2021-04-25T11:17:15+08:00
lastmod: 2021-04-25T11:17:15+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## redis 安装

1. 下载  

    wget -S "http://download.redis.io/releases/redis-4.0.0.tar.gz"

2. 安装  

    $/usr/local/redis-4.0.0
    	make && make install
    	$执行make install 后src下面的六个文件会复制到 /usr/local/bin下 即可执行

3. 

    * make命令执行完成后，会在src目录下生成6个可执行文件，分别是redis-server、redis-cli、redis-benchmark、redis-check-aof、redis-check-dump、redis-sentinel

    <pre>
    redis-server is the Redis Server itself.（Redis服务器本身）
    redis-sentinel is the Redis Sentinel executable (monitoring and failover).（Redis集群的管理工具）
    redis-cli is the command line interface utility to talk with Redis.（与Redis进行交互的命令行客户端）
    redis-benchmark is used to check Redis performances.（Redis性能测试工具）
    redis-check-aof and redis-check-dump are useful in the rare event of corrupted data files.（AOF文件修复工具和RDB文件检查工具
    </pre>

4. 启动、关闭  

  * utils/install_server.sh 集群安装脚本
  * 启动  

  		utils目录下
  		install_server.sh 可以安装redis服务
  		其中
  		redis_init_script 服务脚本，可以复制到/etc/init.d/
  * 关闭  

  		redis-cli shutdown

## redis sentinel

- [obtainal sentinel](https://redis.io/topics/sentinel#obtaining-sentinel) redis哨兵配置

## redis 从节点

## redis 集群

### redis.conf

<pre>
port 7000
cluster-enabled yes #开启实例的集群模式
cluster-config-file nodes.conf # 设定了保存节点配置文件的路径
cluster-node-timeout 5000
appendonly yes
</pre>

### 集群演示


<pre>
# 创建redis目录
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
</pre>
---

同一个 ID ， 从而在集群中保持一个独一无二（unique）的名字  

<pre>
$ redis-trib.rb 在 src 下，选项 `–replicas 1` 表示我们希望为集群中的每个主节点创建一个从节点
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
</pre>

* 执行这个脚本需要提供 ruby 2.3 以上的版本，自带的不够， 关键的是要执行 `gem install redis` 安装 ruby 和 redis 的连接的东东，
	
	<pre>
	# 替换为 ruby 国内源 
	gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
	ruby -version
	# 升级 ruby 版本
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
	curl -sSL https://get.rvm.io | bash -s stable
	source /etc/profile.d/rvm.sh
	mkdir ~/.rvm/user/db
	echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db
	rvm install 2.3.0
	rvm use 2.3.0 --default
	# ruby on rails
	gem install rails
	bundle update
	gem install redis
	</pre>

* 剩下的待补充，公司也没有用
	* [comman](https://redis.io/topics/cluster-tutorial)
	* [Specification](https://redis.io/topics/cluster-spec)
	* [唯品会redis集群搭建经验](http://mdba.cn/2016/06/08/%E5%88%86%E4%BA%AB%EF%BC%9A%E5%94%AF%E5%93%81%E4%BC%9A%E5%A4%A7%E8%A7%84%E6%A8%A1-redis-cluster-%E7%9A%84%E7%94%9F%E4%BA%A7%E5%AE%9E%E8%B7%B5/)

## redis 文档

- sentinel哨兵

  - [obtainal sentinel](https://redis.io/topics/sentinel#obtaining-sentinel) redis哨兵配置 官方文档
  - [redis命令行参考](http://redisdoc.com/)
- cluster
  - [redis集群-官网翻译](http://www.redis.cn/topics/cluster-tutorial.html)
- replication
  - [redis复制](https://www.cnblogs.com/peter1018/p/9766237.html)

