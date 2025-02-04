---
title: RTT 启动顺序
subtitle: 启动顺序
date: 2024-06-04T20:50:07+08:00
slug: b234ee6
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

{{< admonition abstract "启动" >}}
启动是系统的第一个动作。
{{< /admonition >}}




<!--more-->


系统在完成堆桟初始化之后，就可以调用 `rtthread_startup` 函数，完成操作系统启动动作。

`rtthread_startup` 函数内主要操作：
- 关闭中断
- 初始化必要的片上驱动
- 初始化系统定时器
- 初始化系统调动器
- 初始化信号，如果使用了的话
- 创建第一个应用线程
- 创建定时器线程
- 创建空闲线程
- 加锁CPU锁

最后启动调度器，在调度器内启动第一个线程(一般为`main`线程)，并且永远都不会返回。

{{< admonition info >}}
为啥不会返回呢？因为完成调度之后，就跳转到第一个线程上执行，且后续会在各个线程中跳转，无法返回调度器函数。
{{< /admonition >}}

``` C
void rt_system_scheduler_start(void)
{
    struct rt_thread *to_thread;
    rt_ubase_t highest_ready_priority;

    /**
     * legacy rt_cpus_lock. some bsp codes still use it as for it's critical
     * region. Since scheduler is never touching this, here we just release it
     * on the entry.
     */
     // 主动释放掉 `cpus` 锁，防止前面使用过忘记释放。
    rt_hw_spin_unlock(&_cpus_lock);

    /* ISR will corrupt the coherency of running frame */
    rt_hw_local_irq_disable();

    /**
     * for the accessing of the scheduler context. Noted that we don't have
     * current_thread at this point
     */
     // 加锁调度器。
     // 我们还没有完成调度器初始化，所以无法使用多核调度，也就不能让其他核心使用调度器。
    _fast_spin_lock(&_mp_scheduler_lock);

    /* get the thread scheduling to */
    // 获取当前就绪列表中优先级最高的任务
    // 任务在执行 `rt_thread_startup` 之后，就已经加入就绪列表中。
    // 一般在启动调度器之前，只会初始化个别系统任务。如：主任务、空闲任务、软件定时器任务。
    to_thread = _scheduler_get_highest_priority_thread(&highest_ready_priority);
    // 最高任务不能为空，这个很好解释，如果将要跳转的先成为空，那应该向哪里跳转呢。
    RT_ASSERT(to_thread);

    /* to_thread is picked to running on current core, so remove it from ready queue */
    // todo:
    _sched_remove_thread_locked(to_thread);

    /* dedigate current core to `to_thread` */
    // 重新设置第一个线程的核心为启动核，当然启动核心也非必须是ID为0的核心
    // 需要考虑一个问题，除了主线程在调度器启动之后立即执行，其余线程，如空闲线程，软件定时器线程是否永远被启动核绑定并执行？
    RT_SCHED_CTX(to_thread).oncpu = rt_hw_cpu_id();
    RT_SCHED_CTX(to_thread).stat = RT_THREAD_RUNNING;

    LOG_D("[cpu#%d] switch to priority#%d thread:%.*s(sp:0x%08x)",
          rt_hw_cpu_id(), RT_SCHED_PRIV(to_thread).current_priority,
          RT_NAME_MAX, to_thread->parent.name, to_thread->sp);

    _fast_spin_unlock(&_mp_scheduler_lock);

    /* switch to new thread */
    // 切换到第一个任务，需要了解的是，第一次切换，不需要 from 线程，也就是说，只需要知道跳转地址就行。
    rt_hw_context_switch_to((rt_ubase_t)&to_thread->sp, to_thread);

    /* never come back */
    // 永远不会执行到这里
    // 建议增加一行代码 
    // RT_DEBUG(0);
    // 这样在启动失败之后，可以非常迅速定位到问题
}

```

{{< admonition info >}}
软件定时器会直接绑定到启动核心，但是空闲任务是每个核心都会执行，所以不需要特别关心。
{{< /admonition >}}



## 切换上下文
提到任务切换，跨不过去的一道坎就是上下文切换。把操作系统比作整个人体的话，系统定时器就是心脏，调度器就是大脑，决定下一个可执行哪个任务，而上下文切换就是记忆体，来保存上个动作和恢复当前动作。

每个处理器对待现场的处理都是不一样的。例如 `ARM` 处理器需要使用汇编语句一次保存通用寄存器和状态寄存器。而 `TriCore` 核心有自己的上下文保存区域，简称 `CSA(Content Savee Area)`。

### CSA
`CSA` 是 `TriCore` 核心的一个特点，在某一个指定的区域按照既定的规则来存储上下文片段，每个片段可以直接将现场保存，可以认为是"快照"，比较容易保存和恢复。每个 `CSA` 占用空间为 `64字节`。实际占用空间可以通过设置 `CPU_FCX` 寄存器初始化第一个 `CSA` 元素。

TODO: CSA 教程


### 切换上下文


上面提到了每个 `CSA` 块都可以保存一个现场，其要执行的操作就是从 `空闲链表` 中申请一块，存储现场之后，在放到上一个被使用的 `CSA` 块的 `NEXT` 位置，该位置需要按照规定的公式进行计算得出。同样，恢复现场的时候，需要将当前 `CSA` 块清空之后，再释放到 `空闲链表` 中，并设置 `FCX` 寄存器的值。


```C
/* 获取当前线程的 CSA LinkWord */
current_upper_csa = LINKWORD_TO_ADDRESS(__mfcr(CPU_PCXI)); /* 保存当前线程的 CSA 上下文 */
*((unsigned long *) from) = current_upper_csa[0];
/* 将 to 线程的 CSA 地址赋值给当前线程的上层上下文的 LinkWord ，用于 TriCore 自动切换线程。*/
current_upper_csa[0] = *((unsigned long *) to);
```


