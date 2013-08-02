---
layout: post
title: 超简易openCL api
tags: [openCL]
---

虽说openCL有个简单的c++封装器，不过还是很麻烦呢，一大坨代码，能向CUDA一样一句话搞定多好，遂半夜睡不着开始写超简易openCL api。

成果图：

    #include "ldCLtemplate.h"
    #include <ctime>

    #define N 2048
    #define FOR(i,l,r) for (int i=(l);i<=(r);i++)

    int a[N],b[N],c[N];

    int main() {
      FOR(i,0,N-1) a[i]=b[i]=i;

      LDCL ldcl("kernal.cl","adder",cl::NDRange(N));
      ldcl.setArg(a, N*sizeof(a[0]), CL_MEM_READ_ONLY);
      ldcl.setArg(b, N*sizeof(b[0]), CL_MEM_READ_ONLY);
      ldcl.setArg(c, N*sizeof(b[0]), CL_MEM_WRITE_ONLY, LDCL_NEED_READ);
      ldcl.run();
    }

还是要5行呢...，本来setArg那个地方想用个template来着，搞半天都没编译成功，遂放弃。

文档什么也懒得写了......，反正我自己用的说，[github repo](https://github.com/cjld/opencl1/tree/master/ld_opencl_template)

今天还学了下原子函数的用法，原子函数相当于一个全局阻塞的函数，不过速度不咋滴。

对这样一个kernal做了测试：

    #define FOR(i,l,r) for (int i=(l);i<=(r);i++)

    kernel void adder(constant int* a, constant int* b, global int* result)
    {
      int idx = get_global_id(0);
      int x=1,y=2;
      FOR(i,1,100000) {
        atomic_add(result,result[1]);
        atomic_add(result+1,result[0]);
        atomic_add(result,result[1]);
        atomic_add(result+1,result[0]);
        atomic_add(result,result[1]);
        atomic_add(result+1,result[0]);
        atomic_add(result,result[1]);
        atomic_add(result+1,result[0]);
        atomic_add(result,result[1]);
        atomic_add(result+1,result[0]);
      }
    }

复制这么多份目的是为了不让for循环本身成为主要的复杂度。

测试发现，和工作组大小有非常大的关系：

- 工作组大小：1024，耗时：3067ms
- 工作组大小：512，耗时：3037ms
- 工作组大小：256，耗时：2968ms
- 工作组大小：128，耗时：2943ms
- 工作组大小：64，耗时：2886ms
- 工作组大小：32，耗时：5120ms
- 工作组大小：16，耗时：7737ms
- 工作组大小：8，耗时：9095ms
- 工作组大小：4，耗时：11587ms
- 工作组大小：2，耗时：12355ms
- 工作组大小：1，耗时：18766ms

工作组大小为64是最优值，这是为什么？貌似我的 CL\_DEVICE\_MAX\_WORK\_ITEM\_SIZES 的最后一维是64，而且低于64耗时快速攀升，不解，有空上stackoverflow去问问，总之以后尽量吧工作组开大和避免使用过多的原子函数就好了吧。
