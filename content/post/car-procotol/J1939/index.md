---
title: J1939
subtitle:
date: 2024-06-02T23:05:35+08:00
slug: bce29f6
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

{{ < admonition quote "quote" false >}}
note abstract info tip success question warning failure danger bug example quote
{{ < /admonition >}}

<!--more-->



aaaa


``` C
void rt_cpus_lock_status_restore(struct rt_thread *thread)
{
#if defined(ARCH_MM_MMU) && defined(RT_USING_SMART)
    lwp_aspace_switch(thread);
#endif
    rt_sched_post_ctx_switch(thread);
}
```


