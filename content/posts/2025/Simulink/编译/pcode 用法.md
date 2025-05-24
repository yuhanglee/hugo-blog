---
title: pcode 用法
subtitle:
date: 2025-03-11T22:29:25+08:00
slug: df7af41
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
tags:
	- pcode
	- 编译
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
note abstract info tip success question warning failure danger bug example quote
{{< /admonition >}}

<!--more-->



## pcode

Mathworks 官方链接
[pcode](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#responsive_offcanvas)

pcode 是matlab 中对 `.m` 源码文件进行混淆加密的一种方式，其于R2006a 版本引入，在 R2022a版本中使用增强的混淆加密方式，加大了破解的难度。


语法可以分为四种，具体可以点击下方链接查看官方介绍。

``` matlab
`[pcode(item)](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#btbo5ft-4)`
`[pcode(item,"-R2022a")](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#d126e1046741)`
`[pcode(item,"-R2007b")](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#d126e1046758)`
`[pcode(item1,item2,...,itemN)](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#btbo5ft-6)`
`[pcode(___,"-inplace")](https://ww2.mathworks.cn/help/matlab/ref/pcode.html#btbo5ft-7)`
```


下面针对每个语法，由实例入手，讲解实际使用中需要注意的事项：

> 单个文件创建 P 代码文件

最简单的加密方式，就是针对单个文件进行混淆。可以使用 `pcode xxx.m` 即可在当前目录下，生成 `xxx.p` 文件


> 指定软件版本，使用更加复杂的混淆算法加密

可以增加一个 -Rxxxx 的matlab版本，已指定加密和P文件使用的最低软件版本。  其可以指定为 R2007 之后的任意版本。
``` matlab
pcode xxx.m -R2022a

which xxx
% xxx.p
```


> 从指定目录或者多个文件，一次生成多个P代码文件

生成多目录或多文件的方式，就是在 pcode 的参数中，一次添加多个目录或多个文件，支持通配符语法。

当前的目录结构如下：
```
+---V001
|       compileAll.m
|       funcNames.m
|       slblocks.m
|       sl_customization.m
|       test.m
|
\---V002
        compileAll.m
        funcNames.m
        slblocks.m
        sl_customization.m
        test.m
```

在当前目录下，对 V001 V002 两个文件夹中的 `.m` 文件，一次全部转为 `.p` 文件。

```
% 获取所有的目录
ans = {fullfile(pwd, 'V001'), fullfile(pwd, 'V002')};
% 转换多个目录，但是转换的 .p 文件全部位于 pwd 中
pcode(ans{1}, ans{2});
```

转换完成之后的文件树，在 pwd 路径下，多了几个 .p 文件

```
|   compileAll.p
|   funcNames.p
|   slblocks.p
|   sl_customization.p
|   test.p
|
+---V001
|       compileAll.m
|       funcNames.m
|       slblocks.m
|       sl_customization.m
|       test.m
|
\---V002
        compileAll.m
        funcNames.m
        slblocks.m
        sl_customization.m
        test.m
```

> 在指定目录下，直接生成 .p 文件

上述情况中，已经可以对多个目录，一次全部执行转换过程。但是转换的结果会存放在一起，失去了之前的文件树结构。`-inplace` 参数，就是将转换结果，还在放在转换目录下。


```
% 获取所有的目录
ans = {fullfile(pwd, 'V001'), fullfile(pwd, 'V002')};
% 转换多个目录。增加了 `-inplace` 参数，转换的 .p 文件全部位于原来的 .m 同级目录中
pcode(ans{1}, ans{2}, '-inplace');
```

转换完成的结果如下：

``` matlab
+---V001
|       compileAll.m
|       compileAll.p
|       funcNames.m
|       funcNames.p
|       slblocks.m
|       slblocks.p
|       sl_customization.m
|       sl_customization.p
|       test.m
|       test.p
|
\---V002
        compileAll.m
        compileAll.p
        funcNames.m
        funcNames.p
        slblocks.m
        slblocks.p
        sl_customization.m
        sl_customization.p
        test.m
        test.p
```
