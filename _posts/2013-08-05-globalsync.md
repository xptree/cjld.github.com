---
layout: post
title: 在openCL中实现全局同步
tags: [openCL]
---

全局同步是一个至关重要的功能，可是限于硬件，openCL内部并没有现成的全局同步函数，在网上搜了一阵子无果后，遂开始自己琢磨如何实现全局同步。

要实现全局同步并不难，难就难在如何高效的实现全局同步，这里我用了这样一种评测方法：


    kernel void main(volatile global int *a, global int *flags) {
      int i=get_global_id(0);
      
      FOR(j,0,500000) {
        int temp=a[i]+a[(i+1)&(N-1)]+a[(i+2)&(N-1)];
        globalSync(flags);
        a[i]=temp;
        globalSync(flags);
      }
    }

  - 总进程数2048，一开始a数组全赋值为1。
  - 同步成功的判定是最后a数组中的值全部一样，若中途同步失败，极大概率导致最后结果不一样
  - 测时间来评定全局同步方法的优劣
  - 注：不加同步函数时运行时间为220ms

很快，一个最暴力的想法就有了。

  - 申请一个全局计数器flags，初始化为0
  - 每次运行到同步函数的时候，用原子函数+1
  - 用循环不停的检测flags是否等于总进程数

  代码如下：

      void globalSync1(volatile global int *s) {
        atomic_inc(s);
        while (*s!=get_global_size(0));
        if (get_global_id(0)==0) atomic_sub(s,get_global_size(0));
      }

  测试结果：

  - 工作组大小 1024 耗时 3020ms
  - 工作组大小 64 耗时 3021ms
  - 工作组大小 32 耗时 10000+ms

  效率不高，好歹能用，而且代码短。不过这里有个小陷阱，就是volatile限定符，如果不加volatile限定符，会导致在重复访问相同地址时，会直接返回之前寄存器保存的变量，加volatile限定符相当于告诉他这个变量在其他进程会被更改。


方法一效率瓶颈在于原子函数，减少原子函数的调用次数是优化的重点。于是乎，通过利用barrier可以想到方法二：

  - 仍然是计数器flags清零
  - 每个工作组选出一个代表元，比如`get_local_id(0)==0`
  - 如果该进程不是代表元，用barrier拦住
  - 如果该进程为代表元，计数器+1，用循环不停的检测flags是否等于总工作组数

  代码：

      void globalSync2(volatile global int *s) {
        if (get_local_id(0)==0) {
          atomic_inc(s);
          while (*s!=get_num_groups(0));
        };
        barrier(CLK_GLOBAL_MEM_FENCE | CLK_LOCAL_MEM_FENCE);
        if (get_global_id(0)==0) atomic_sub(s,get_num_groups(0));
      }
  
  测试结果：

  - 工作组大小 1024 耗时 1020ms
  - 工作组大小 64 耗时 1024ms
  - 工作组大小 32 耗时 10000+ms

  有了明显的改善，但是工作组比较小的时候用这个函数来同步任然是不可取的。

想到这里我就没继续深究了，又胡乱google了一通，找到这么一篇英文文章:<http://industrybestpractice.blogspot.com/2012/07/global-synchronisation-in-opencl.html>

文中作者介绍了三个方法，前两个和我想的一样，作者基于第二个方法想出了第三个方法，不需要用到原子函数的方法：

  - 仍然是每个工作组找出个代表元
  - 定义第一个工作组为代表工作组，这个工作组负责管理每个工作组的代表元

  这样就形成了3层结构，一层一层往下管理，不得不说作者的思路很巧妙，代码：

      void globalSync3(volatile global int *s) {
        const int lid=get_local_id(0);
        const int gid=get_group_id(0);
        if (!lid) s[gid]=1;
        if (!gid) {
          if (lid<get_num_groups(0))
            while (!s[lid]);
          barrier(CLK_GLOBAL_MEM_FENCE | CLK_LOCAL_MEM_FENCE);
          s[lid]=0;
        }
        while (!lid && s[gid]);
        barrier(CLK_GLOBAL_MEM_FENCE | CLK_LOCAL_MEM_FENCE);
      }
  
  测试结果：

  - 工作组大小 1024 耗时 1405ms
  - 工作组大小 64 耗时 1377ms
  - 工作组大小 32 耗时 10000+ms

  虽然方法很巧妙，却不如第2个方法来的简单粗暴，而且看了代码的读者很轻易的可以发现，第3个方法有不少缺点：

  - 需要一个标记数组
  - 工作组大小必须大于工作组个数
  - 速度稍慢

综上所述，第2个方法也许我会一直用下去。

测试代码可在 <https://github.com/cjld/opencl1/tree/master/globalSync> 下载。

欢迎拍砖
