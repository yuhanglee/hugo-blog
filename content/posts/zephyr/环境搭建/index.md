---
title: zephyr 环境搭建
subtitle:
date: 2024-11-09T16:38:11+08:00
slug: 022c49c
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
hiddenFromRss: false
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
如果我看得更远，那是因为我站在巨人的肩膀上。 --- 牛顿
{{< /admonition >}}

<!--more-->



研究和修改官方示例，是一个不错的选择。


# [zephyrproject-rtos/example-application：示例树外应用程序，也是一个模块](https://github.com/zephyrproject-rtos/example-application?tab=readme-ov-file)

1. windows 搭建过程，可以参考官方教程或者参考下方博客教程。
[Zephyr 开发环境搭建与配置 - 创世~ - 博客园](https://www.cnblogs.com/ZyArdu/p/17841516.html)


```ad-note
Python 的虚拟环境可以使用 workon 来进行配置。
可以参考 [python虚拟环境管理之以workon切换虚拟环境](https://www.yisu.com/ask/32010920.html)
```

2.  定义自己的开发环境

可以在官方示例上做一些修改，暂时满足自己的 "hello world" 测试。  具体目录的说明和各个文件语法，都需要查看官方文档来具体说明。

> 初始化工作环境

新建 workspace 工作目录，并初始化 workspace 为 west 工程。
``` shell
mkdir workspace
cd workspace
# 初始化 workspace 为 west 工程，并拉取 example-application 远程仓库
west init -m https://github.com/zephyrproject-rtos/example-application --mr main my-workspace
# 更新软件。
cd my-workspace
west update
```


> 构建和运行

要构建应用程序，则需要运行以下命令:

``` shell
cd example-apolication
west build -p auto -b custom_plank -d build_custom_plank app
dir build_custom_plank | find ".elf"
```

编译完成后，可以搜索一下生成的 `elf` 文件。

``` shell
> dir build_custom_plank | find ".elf"
2024/11/09  15:19           709,440 zephyr.elf
2024/11/09  15:18           709,996 zephyr_pre0.elf
```

如果看过 zephyr 官方入门文档的话，可以直接略过下方对于命令行参数的说明：

`-p` :  选择编译前清除模式，分别为 `auto`、`always`、`never`
`-b` :  选择 board 型号，所支持的型号，可以使用命令 `west boards` 来获取
`-d` :  选择编译文件输出目录



