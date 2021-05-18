---
title: "ChangeClusterIP"
date: 2021-04-25T12:13:17+08:00
lastmod: 2021-04-25T12:13:17+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## 修改kubenetes master ip 地址

https://github.com/kubernetes/kubeadm/issues/338

For those looking for even more clarity, here were my experiences:

1. replace the IP address in all config files in `/etc/kubernetes`

   ```
   oldip=192.168.1.91
   newip=10.20.2.210
   cd /etc/kubernetes
   # see before
   find . -type f | xargs grep $oldip
   # modify files in place
   find . -type f | xargs sed -i "s/$oldip/$newip/"
   # see after
   find . -type f | xargs grep $newip
   ```

2. backing up `/etc/kubernetes/pki`

   ```
   mkdir ~/k8s-old-pki
   cp -Rvf /etc/kubernetes/pki/* ~/k8s-old-pki
   ```

3. identifying certs in `/etc/kubernetes/pki` that have the old IP address as an alt name (this could be cleaned up)

   ```
   cd /etc/kubernetes/pki
   for f in $(find -name "*.crt"); do 
     openssl x509 -in $f -text -noout > $f.txt;
   done
   grep -Rl $oldip .
   for f in $(find -name "*.crt"); do rm $f.txt; done
   ```

4. identify configmap in the `kube-system` namespace that referenced the old IP, edit them:

   ```
   # find all the config map names
   configmaps=$(kubectl -n kube-system get cm -o name | \
     awk '{print $1}' | \
     cut -d '/' -f 2)
   
   # fetch all for filename reference
   dir=$(mktemp -d)
   for cf in $configmaps; do
     kubectl -n kube-system get cm $cf -o yaml > $dir/$cf.yaml
   done
   
   # have grep help you find the files to edit, and where
   grep -Hn $dir/* -e $oldip
   
   # edit those files, in my case, grep only returned these two:
   kubectl -n kube-system edit cm kubeadm-config
   kubectl -n kube-system edit cm kube-proxy
   ```

5. change the IP address (via cli or gui for your distro)

6. delete both the cert and key for each identified by grep in the prior step, regenerate those certs

   > NOTE: prior to recreating the certs via `kubeadm admin phase certs ...`, you'll need to have the new IP address applied

   ```
   rm apiserver.crt apiserver.key
   kubeadm alpha phase certs apiserver
   
   rm etcd/peer.crt etcd/peer.key
   kubeadm alpha phase certs etcd-peer
   ```

7. restart kubelet and docker

   ```
   sudo systemctl restart kubelet
   sudo systemctl restart docker
   ```

8. copy over the new config

   ```
   sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
   ```
