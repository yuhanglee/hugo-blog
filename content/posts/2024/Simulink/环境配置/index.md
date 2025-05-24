---
title: Simulink环境配置
subtitle:
date: 2024-07-18T21:29:33+08:00
slug: 356f2d6
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
  - Coder
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



<!--more-->


1. 搭建下面模型

![](Pasted%20image%2020240718213511.png)

![](Pasted%20image%2020240718213520.png)

其中 `Function-Call` 的周期分别为 0.02 和 0.005。
![](Pasted%20image%2020240718213632.png)


2. 配置

我们一般使用手写代码，对于5ms和20ms周期任务，会分为两个周期任务，分别为 task_5ms 和 task_20ms。在计数增加到5/20之后，会执行对应的周期函数。

```C

uint32 in1 = 1;
uint32 out1 = 0;
uint32 in2 = 1;
uint32 out2 = 0;
uint32 count[2] = {0};
uint32 timeout[2] = {5, 20};
uint32 flag[2] = {0};


void task_5ms(void)
{
	out1 += (in1 * 2);
}


void task_20ms(void)
{
	out2 += (in2 * 4);
}

void main(void)
{
	if (flag[0] == 1)
	{
		flag[0] = 0;
		task_5ms();
	}

	if (flag[1]  == 1)
	{
		flag[1] = 0;
		task_20ms();
	}
}

void isr_1ms()
{
	int i = 0;

	for (i = 0; i < 2; i++)
	{
		count[i]++;
		if (count[i] >= timeout[i])
		{
			flag[i] = 1;
		}
	}
}


```

我们需要做的就是让生成代码，尽可能接近手写代码。


- 生成代码和报告

我们首先配置模型，是的每次只生成代码。

配置解算器类型为固定方式，周期为1ms。 这样，模型每隔1ms执行一次。

![](Pasted%20image%2020240718222859.png)


- 生成代码

按下 <brd> ctrl+B </brd> ，可以生成代码。
但是在生成代码之前，需要配置报告。

1) 配置目标

配置目标为 `ert.tlc`，嵌入式代码类型

![](Pasted%20image%2020240718223257.png)


2)  生成报告

创建通用报告，并自动打开报告。

![](Pasted%20image%2020240718223357.png)

完成以上配置之后，按下 `ctrl+B` ，就可以自动生成代码，并完成之后，自动打开报告。



- 报告分析


1) main函数


在 `ert_main.c` 中，出现了 main 函数，这个是我们不需要的。

![](Pasted%20image%2020240718223539.png)


可以在下面模型配置中关闭 `main` 文件生成。

![](Pasted%20image%2020240718223632.png)


2) terminate函数

![](Pasted%20image%2020240718223733.png)

可以在下面模型配置中关闭 terminate 函数。

![](Pasted%20image%2020240718223841.png)


3) 删除注释

注释给每个人的感觉不一样，有些开发者觉得注释可以来说明程序的走向，但是Simulink中的注释不太容易看懂。在最终生成代码中，我一般关闭注释输出。
但是有一个好处，在调试阶段，可以很快的通过注释，来反向确定当前生成代码的模块。

可以在下面模型配置中关闭 注释 选项

![](Pasted%20image%2020240718224209.png)


4) 多周期任务执行

不同任务周期，可以配置多个周期的函数。其中周期越短，后缀数值越小。例如当前两个任务周期为5ms和20ms，则会生成两个函数，分别为 `_step0()` 和 `_step1()` 。 

![](Pasted%20image%2020240718225525.png)

生成代码如下：

![](Pasted%20image%2020240718225729.png)

5) 整理代码

- inline 方式生成代码

![](Pasted%20image%2020240718225755.png)

![](Pasted%20image%2020240718225813.png)

- 不使用全局变量


