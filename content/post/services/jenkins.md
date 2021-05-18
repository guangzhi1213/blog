---
title: "Jenkins"
date: 2021-04-25T11:08:08+08:00
lastmod: 2021-04-25T11:08:08+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## jenkins

### 官方文档

- [w3c 翻译](https://www.w3cschool.cn/jenkins/)
- 

### jenkins pipeline

- [pipeline 官方示例](https://www.jenkins.io/doc/book/pipeline/getting-started/)

### jenkins 备份脚本

```shell
#!/bin/bash  
#  Jenkins Configuraitons Directory  
JENKINS_HOME=/root/.jenkins

cd $JENKINS_HOME  
  
#  Add general configurations, job configurations, and user content  
#git add -- *.xml jobs/*/*.xml userContent/* ansible/*  
git add -- *.xml jobs/*/*.xml userContent/*
  
#  only add user configurations if they exist  
if [ -d users ]; then  
    user_configs=`ls users/*/config.xml`  
      
    if [ -n "$user_configs" ]; then  
        git add $user_configs  
    fi  
fi  
  
# mark as deleted anything that's been, well, deleted  
to_remove=`git status | grep "deleted" | awk '{print $3}'`  
  
if [ -n "$to_remove" ]; then  
    git rm --ignore-unmatch $to_remove  
fi  
  
git commit -m "Automated Jenkins commit"  

git push -q -u origin master  
```

git仓库要有权限提交...

### jenkins发布前端项目

早上遇到一个发布vue项目出错的事情， 语法不规范导致错误日志很多， 满屏错误不知道他们在想啥，手动执行还是可以成功的

修改 `CI` 变量为 false 就行，是一个node使用到的变量。