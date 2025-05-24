---
title: 执行 xmake 和系统命令
subtitle:  使 tlc/m 脚本文件可以直接或间接执行系统命令。
date: 2025-02-11T21:59:55+08:00
slug: 6996d4f
draft: false
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
categories:
  - Simulink
tags:
  - xmake Simulink
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromFeed: false
hiddenFromRelated: false
summary:
resources:
toc: true
math: false
lightgallery: false
---

{{< admonition quote "quote" false >}}
xmake 作为一个构建工具，可以支持 `arm-gcc` 编译过程。其也可以用于 Simulink 模型生成 C源码之后，进行整体工程代码编译，以及输出 `.hex` 文件。
{{< /admonition >}}

<!--more-->

1. 在 `tlc` 文件中，使用 `FEVAL(...)` 函数，执行本地 `.m` 脚本文件。
``` tlc
FEVAL("runBuildBat")
```

2. 在 `runBuild.m` 脚本文件中，执行 `build.bat` 脚本。
需要注意的是，返回值（out）必须包含，否则会执行失败。具体原因还有待考察。

```
function out = runBuildBat()
    system("build.bat");
    
    out = 0;
end
```

3. 在 tlc 同级目录，创建 `build.bat` 文件，执行一个写入动作。

``` 
# build.bat
echo "hello" > hello.txt
```

4. 可以在本地看到一个 `hello.txt` 文件，其中内容为 "hello"。
