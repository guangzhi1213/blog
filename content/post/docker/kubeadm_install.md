---
title: "Kubeadm_install"
date: 2021-04-25T12:12:56+08:00
lastmod: 2021-04-25T12:12:56+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## kubeadm init

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

自己创建了一个仓库的时候如何使用自己的仓库。

`kubeadm init --kubernetes-version=1.18.0  --apiserver-advertise-address=10.0.8.128   --image-repository 10.0.8.10:8082/google_containers  --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16`


