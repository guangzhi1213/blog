---
title: "Nexus"
date: 2021-04-25T11:11:44+08:00
lastmod: 2021-04-25T11:11:44+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## yum 仓库代理

#### 下载地址

nexus2: [https://help.sonatype.com/repomanager2/download](https://help.sonatype.com/repomanager2/download)

nexus3: [https://help.sonatype.com/repomanager3/download](https://help.sonatype.com/repomanager3/download)

#### nexus代理配置

> yum开头为 yum 类型的代理, docker开头为 docker 镜像类型的代理, go开头的为 goproxy 类型的代理
> System -> HTTP 中配置代理地址

yum-host: 自己上传的rpm包, 没有特殊的地方

yum-proxy: 代理的包

    Proxy:
      Remote storage: https://mirrors.aliyun.com/centos/
    Negative Cache:
      Not found cache enabled: 去掉选中

yum-group: 以上两个的合集

yum-kubernetes-proxy:

    Proxy:
      Remote storage: https://mirrors.aliyun.com/kubernetes/
    Negative Cache:
      Not found cache enabled: 去掉选中

yum-kubernetes-group: 可选

yum-docker-proxy: 

    Proxy:
      Remote storage: https://mirrors.aliyun.com/docker-ce/linux/centos/
    Negative Cache:
      Not found cache enabled: 去掉选中

yum-docker-group: 可选

helm-proxy:

    Proxy:
      Remote storage: https://kubernetes-charts.storage.googleapis.com/
    Negative Cache:
      Not found cache enabled: 去掉选中

helm-hosted: 自己上传chart的地方

goproxy:

    Proxy:
      Remote storage: https://mirrors.aliyun.com/goproxy/
    Negative Cache:
      Not found cache enabled: 去掉选中

golang: golang包的group

docker-host: 存放自己创建的镜像的地方

docker-proxy-quay:

    Proxy:
      Remote storage: https://quay.io
      Docker index: Use proxy registry
    Negative Cache:
      Not found cache enabled: 去掉选中

docker-proxy-k8s-gcr:

    Proxy:
      Remote storage: https://k8s.gcr.io
      Docker index: Use proxy registry
    Negative Cache:
      Not found cache enabled: 去掉选中

docker-proxy-gcr:

    Proxy:
      Remote storage: https://gcr.io
      Docker index: Use proxy registry
    Negative Cache:
      Not found cache enabled: 去掉选中

docker-proxy:

    Proxy:
      Remote storage: https://r2nm48l3.mirror.aliyuncs.com
      Docker index: Use Docker Hub
    Negative Cache:
      Not found cache enabled: 去掉选中

docker-group: 以上几个docker镜像仓库的group

    HTTP:
      Create an http connecter: 8082
      Allow anymous docker pull: 选中
    Docker Registry API support:
      Enable docker v1 api: 选中
#### centos配置

nexus-repo:

```repo
[nexus-base]
name=Nexus Yum Repository
baseurl=http://10.0.8.10:8081/repository/yumgroup/$releasever/os/$basearch/
enabled=1
gpgcheck=0

[nexus-updates]
name=Nexus Yum Repository
baseurl=http://10.0.8.10:8081/repository/yumgroup/$releasever/updates/$basearch/
enabled=1
gpgcheck=0

[nexus-extras]
name=Nexus Yum Repository
baseurl=http://10.0.8.10:8081/repository/yumgroup/$releasever/extras/$basearch/
enabled=1
gpgcheck=0
```

docker-repo:

```repo
[docker]
gpgcheck=0
enabled=1
baseurl=http://10.0.8.10:8081/repository/yum-docker-group/$releasever/$basearch/stable
name=docker repo
```

kubernetes-repo:

```repo
[kubernetes]
repo_gpgcheck=0
name=kubernetes
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
enabled=1
baseurl=http://10.0.8.10:8081/repository/yum-kubernetes-group/yum/repos/kubernetes-el$releasever-$basearch
```

salt-repo:

```repo
[saltstack]
gpgcheck=0
enabled=1
baseurl=https://mirrors.aliyun.com/saltstack/yum/redhat/$releasever/$basearch/3000
name=salt-py3-repo-3000
```
