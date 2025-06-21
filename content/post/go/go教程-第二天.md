---
title: go教程-第二天
subtitle:
date: 2023-12-27T21:41:43+08:00
draft: false
author:
  name: yuhang
  link: 
  email: jeekwe@126.com
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - go教程
categories:
  - 教程
  - go教程
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRss: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: true
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

我们可以创建属于自己的模块，也叫做"造轮子"。
<!--more-->

软件开发过程中，可能要使用各种模块来完成一些小功能或者编写小工具，此时就需要有一个简单，易懂，方便的外部模块来实现特定功能，防止因为功能多，造成代码臃肿。


# 创建Go模块

> 模块内，只需要实现特定功能，尽量简洁。如果需要完成不属于该模块的功能，则需要调用其他模块。


首先，需要创建一个Go模块。在模块内，可以为一组离散且有用的函数组成一个或多个相关模块包。例如，可以创建一个包，其中包含具有执行财务分析功能的包，其他编写和财务相关应用程序的开发者就可以在该模块基础之上完成对应功能。


## 创建绿色模块

在模块源码路径中初始化当前目录。
```shell
> go mod init greetings
go: creating new go.mod: module greetings
```

在该目录下，新建文件，并命名为 `greetings.go`，并将下面的代码粘贴到文件内。
```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

至此，模块内代码已经完成。该模块定义了一个函数 `Hello`，被外部调用时，输出 `Hi <name>. Welcome!`。


由以上的函数，可以得出几个结论：
> 1. 函数关键字为 func

> 2. 和C语言不一样的是，参数先写变量名称，后跟参数类型

> 3. 返回类型写到最后

> 4. 语句不以分号结束，按照行结束

> 5. `message` 未定义类型，是弱类型语言



# 调用模块

除了定义的模块外，还需要可以由外部模块，调用该模块内函数或变量，以完成我们最初的功能。



```shell
admin@DESKTOP-NQU1HUV E:\go\dayStudy\2day\hello
$ go mod tidy
go: found greetings in greetings v0.0.0-00010101000000-000000000000

admin@DESKTOP-NQU1HUV E:\go\dayStudy\2day\hello
$ go run .
Hi, Gladys. Welcome!
```

