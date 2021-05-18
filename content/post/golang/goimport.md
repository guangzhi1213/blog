---
title: "Go语法 import的用法"
date: 2021-04-21T16:49:50+08:00
lastmod: 2021-04-21T16:49:50+08:00
draft: false
keywords: ["goimport","go","import"]
description: "manjoc'blog"
tags: ["import","golang","go","Go语法"]
categories: ["Go"]
author: "王清"
---

# 引入包时需要注意的地方

## 当你想引入一个模块中的struct时，正确的做法

when i want use this

```go
import (
   "m5/cmd/models"
)

func (m models.Modules) TypeCommand() string {

}
```

remind me this is unresolved type with "models.Modules", you should use this

```go
import (
   "m5/cmd/models"
)

type myModule struct {
    models.Modules
}

func (m myModule) TypeCommand() string {
  // method code here
}
```
