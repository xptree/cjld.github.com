---
layout: post
title: openCL同步函数
tags: [openCL]
---

在并行程序里，做到同步并且效率不会下降是一件很麻烦的事情，openCL里有几个用来同步的函数，简单学习下。

笔者为新手，文章都是学习笔记，有些地方靠自己猜测，有错误还希望指出~

##barrier##

    void barrier (cl_mem_fence_flags flags)

* 只能同步同一个工作组内部的函数。
* cl\_mem\_fence\_flags:
  * CLK\_LOCAL\_MEM\_FENCE
  * CLK\_GLOBAL\_MEM\_FENCE
* `CLK_LOCAL_MEM_FENCE` 和 `CLK_GLOBAL_MEM_FENCE` 的区别

  这里用到的memfence技术，并不是真真意义上的同步，而是变量读写上保证同步，大体可以这样理解，在memfence保证了在barrier之前的“写”操作在barrier之后的“读”操作之前，不过好像也不完全是（reference上看不大明白）。

  总之，要用local变量的时候就用`barrier(CLK_LOCAL_MEM_FENCE)`,要用global变量是就用`barrier(CLK_GLOBAL_MEM_FENCE)`，保险起见还可以用`barrier(CLK_LOCAL_MEM_FENCE | CLK_GLOBAL_MEM_FENCE)`。

  另外openCL里也有`void mem_fence(cl_mem_fence_flags flags)`，用法感觉和barrier差不多，区别在于，barrier相当于一个全工作组的mem fence，只有工作组里所有进程都过了mem fence，大家才会一起继续执行。

##关于工作组##

* 每个group的local空间很小，比如gt750m是`49152 byte`，只够开10000多个int
* local 工作组大小必须是 global 大小的约数，而且local也不能太大，我的是1024

##异步复制函数##

感觉没啥用呢...手写复制难道会慢很多么，又不是硬盘IO操作。

另外最近找到了个大大翻译的中文reference，地址<http://github.com/downloads/walkthetalk/contextstudy/opencl-spec-zh-beta2.pdf> ，[这里](https://niqingliang2003.wordpress.com/)是他的blog。
