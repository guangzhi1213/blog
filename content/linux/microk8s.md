---
title: "Microk8s"
date: 2021-04-21T18:02:24+08:00
draft: false
tags: ["linux","sre","ops"]
categories: ["linux"]
---

# microk8s

## install

### ubuntu20.04lts

```bash
sudo ufw allow in on cni0 && sudo ufw allow out on cni0
sudo ufw default allow routed
```

## command

## component

### microk8s ctr

containerd 是 microk8s 默认的容器技术。而没有使用docker。

我们创建的映像是Docker已知的。但是，Kubernetes不了解新生成的映像。这是因为您的本地Docker守护程序不属于MicroK8s Kubernetes集群。我们可以从本地Docker守护程序导出构建的映像，然后将其“注入”到MicroK8s映像缓存中，如下所示：

```bash
docker save mynginx > myimage.tar
microk8s ctr image import myimage.tar
```

### microk8s dashboard

If RBAC is not enabled access the dashboard using the default token retrieved with:

token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

In an RBAC enabled setup (microk8s enable RBAC) you need to create a user with restricted
permissions as shown in:
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md