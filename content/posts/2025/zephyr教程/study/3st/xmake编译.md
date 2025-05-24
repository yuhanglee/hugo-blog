---
title: zephyr教程-xmake编译
subtitle:
date: 2025-02-12T22:21:33+08:00
slug: 4f8dc97
draft: true
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
  - draft
tags:
  - draft
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromFeed: false
hiddenFromRelated: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
---

{{< admonition quote "quote" false >}}
note abstract info tip success question warning failure danger bug example quote
{{< /admonition >}}

<!--more-->

可以在 ubuntu 中编译出 libkernel2.a 

``` sh
west build app -b MBH86001  -d build/MBH86001 && arm-none-eabi-ar -d build/MBH86001/app/libapp.a main.c.obj
cd libs && arm-none-eabi-ar -M < merge2.mri
```

将 libs 中的 libkernel2.a 下载到windowns中，并执行 `xmake` 可以得到 `build/xxx.hex && build/xxx.bin`。将 `xxx.hex` 下载到开发板中，程序顺利执行。

