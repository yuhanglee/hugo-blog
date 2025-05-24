---
title: xmake-第一天
subtitle:
date: 2024-01-02T21:43:58+08:00
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
  - xmake
categories:
  - xmake
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

<!--more-->


[xmake官方教程](https://xmake.io/#/zh-cn/getting_started)


## 1. 安装 xmake

在 `ubuntu 24.04` 中，可以使用以下几种方法安装 `xmake` 。
#### 1. 使用 curl
``` sh
curl -fsSL https://xmake.io/shget.text | bash

# 可以根据需求选择版本
curl -fsSL https://xmake.io/shget.text | bash -s dev
curl -fsSL https://xmake.io/shget.text | bash -s v2.7.7
```

#### 2. 使用 wget
``` sh
wget https://xmake.io/shget.text -O - | bash
```

#### 3. 使用 `PowerShell`
``` powershell
Invoke-Expression (Invoke-Webrequest 'https://xmake.io/psget.text' -UseBasicParsing).Content

# 可以根据需求选择版本
Invoke-Expression (Invoke-Webrequest 'https://xmake.io/psget.text' -UseBasicParsing).Content dev
Invoke-Expression (Invoke-Webrequest 'https://xmake.io/psget.text' -UseBasicParsing).Content v2.7.7
```

根据所使用的系统平台不同，还可以分为 `Windows`、`Linux`、`MacOS` 三种特有的方式。

### Windows
#### 1. 直接安装 exe 程序
a. 从 [Releases](https://github.com/xmake-io/xmake/releases) 上下载windows安装包
b. 运行安装程序 xmake-[version].[win32|win64].exe


#### 2. 使用 scoop
``` powershell
scoop install xmake
```

#### 3. 使用 winget
```
winget install xmake
```

#### 4. 使用 pacman
```
# 64 bits
pacman -Sy mingw-w64-x86_64-xmake

# 32 bits
pacman -Sy mingw-w64-i686-xmake
```


### MacOS

#### 1.  使用 brew
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install xmake
```

#### 2. 使用安装包
a. 从 [Releases](https://github.com/xmake-io/xmake/releases) 上下载pkg安装包
b. 双击运行

### Arch Linux
```
sudo pacman -Sy xmake
```


### Alpine Linux
```
sudo apk add xmake
```


### Ubuntu 
#### 1. 使用 apt 安装
``` sh
sudo add-apt-repository ppa:xmake-io/xmake
sudo apt update
sudo apt install xmake
```

其他平台安装方式，可以参考官方文档 [安装 - xmake](https://xmake.io/#/zh-cn/guide/installation?id=ubuntu)

### 源码方式安装
{{ admonition note 注意 }}
一般不建议使用 `root` 权限安装 `xmake` 或者使用 `xmake`。
{{ /admonition }}

```
git clone --recursive https://github.com/xmake-io/xmake.git
cd ./xmake
./configure
make
./scripts/get.sh __local__ __install_only__
source ~/.xmake/profile
```


### 更新升级

```
xmake update 2.7.1
```




