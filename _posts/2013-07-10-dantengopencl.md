---
layout: post
title: "OpenCL学习"
description: ""
category: 
tags: [openCL]
---
`OpenCL` [wiki](http://zh.wikipedia.org/wiki/OpenCL),简单来说是一个用于并行计算设备的标准api，一开始觉得CUDA和OpenCL是竞争关系，后面稍微了解了以后才发现他们之间是包容关系。

几乎所有平台都有实现openCL，intel,AMD,Nvidia,CUDA是Nvidia在openCL上又进行了封装和一些针对N卡优化的SDK，相比之openCL更容易上手，不得不吐槽openCL令人蛋碎的纯C用法。

本文参考自 <http://blog.csdn.net/leonwei/article/category/1410041> ，作者写的很不错，让我能快速入门。

买回来一本书结果发现讲的是openCL1.1的，2011年年底就已经出1.2标准了...国内资料的时效性还是不够啊，在网上找到了些比较权威的学习资料：

* 官方reference<http://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/>
* openCL1.2 overview<http://www.khronos.org/assets/uploads/developers/library/overview/opencl-overview.pdf>
* 官方入门教材 <http://developer.amd.com/wordpress/media/2012/10/opencl-1.2.pdf>

##OpenCL架构
笔者简单总结了下OpenCL的架构和一个简单的加法器实现，有错误还望指出:

1.  __Paltform__
    平台，相当于不同厂商的OpenCL实现，比如intel实现了，AMD实现了并且我又同时装了两者的openCL sdk，那我就会有两者的平台。

    获得所有Platform函数：

        cl_int clGetPlatformIDs(
            cl_uint num_entries,
            cl_platform_id *platforms,
            cl_uint *num_platforms)
        
    在OpenCL里面为了获取一个数组元素，都和这个结构差不多

    * `num_entries` 指名`platforms`指针指向的空间有多大，防止越界
    * `platforms` 用来存放列表的指针,需要你在外部申请好空间
    * `num_platforms` 返回有多少个platforms
    * 错误码 OpenCL里的所有函数几乎都有错误返回值，具体错误返回值代表的意思可以到`CL/cl.h`头文件查看。错误返回值一般是函数的返回值或者是最后一个参数。

    假设`num_entries<num_platforms`，那么他只会吧前`num_entries`个platform存起来，举例：

        //get platform numbers
        err = clGetPlatformIDs(0, 0, &num);
 
        //get all platforms
        vector<cl_platform_id> platforms(num);
        err = clGetPlatformIDs(num, &platforms[0], &num);

2.  __Device__
    设备，也就是用来运算的实体

    获取所有设备的函数：

        cl_int clGetDeviceIDs (
            cl_platform_id  platform ,
            cl_device_type  device_type ,
            cl_uint  num_entries ,
            cl_device_id  *devices ,
            cl_uint  *num_devices )
        
    * `platform` 选择一个平台
    * `device_type` 你需要的设备类型，有如下几种
      * `CL_DEVICE_TYPE_CPU`
      * `CL_DEVICE_TYPE_GPU` 
      * `CL_DEVICE_TYPE_ACCELERATOR`
      * `CL_DEVICE_TYPE_CUSTOM`
      * `CL_DEVICE_TYPE_DEFAULT`
      * `CL_DEVICE_TYPE_ALL`
    * 后面这些参数用来获取列表

    获取设备信息：`clGetDeviceInfo` ，具体用法戳[这里](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clGetDeviceInfo.html)

    举例：

        //get device num
        err=clGetDeviceIDs(platforms[DID],CL_DEVICE_TYPE_ALL,0,0,&num);
        vector<cl_device_id> did(num);
        //get all device
        err=clGetDeviceIDs(platforms[DID],CL_DEVICE_TYPE_ALL,num,&did[0],&num);
        cout<<"number of device: "<<num<<endl;
        clGetDeviceInfo(did[0],
            CL_DEVICE_TYPE,
            infoSize,
            info,
            &infoSize);
        cout<<*((cl_device_type*)info)<<endl;

3.  __context__
    上下文，用来沟通设备平台和计算任务。
  
    创建上下文：

        cl_context clCreateContextFromType (
            cl_context_properties   *properties,
            cl_device_type  device_type,
            void  (*pfn_notify) (
              const char *errinfo,
              const void  *private_info,
              size_t  cb,
              void  *user_data),
            void  *user_data,
            cl_int  *errcode_ret)
        
    * `properties` 上下文的属性，用来指定平台
      >openCL中的设置序列一般都是「property 种类」及「property 內容」成对出现，并以 0 做为结束。
    * `device_type` 选择用平台中的那些设备来创建上下文
    * `pfn_notify` 错误回调函数
    * `user_data` 用来传给回调函数的参数
    * 错误码

    举例：

        //set property with certain platform
        cl_context_properties prop[] = { CL_CONTEXT_PLATFORM, reinterpret_cast<cl_context_properties>(platforms[DID]), 0 };
        cl_context context = clCreateContextFromType(prop, CL_DEVICE_TYPE_ALL, NULL, NULL, &err);

4.  __commandQueue__
    用来给device发送指令

        cl_command_queue clCreateCommandQueue(
            cl_context context,
            cl_device_id device,
            cl_command_queue_properties properties,
            cl_int *errcode_ret)

    * `context` 指定上下文
    * `device` 指定设备
    * `properties` commandQueue设置，[详见](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clCreateCommandQueue.html)
    * 错误码

    举例：

        cl_command_queue cqueue = clCreateCommandQueue(context, did[0], 0, &err);

5.  __program__
    在上下文中加载代码，分两个步骤：

    * 加载
     
      [clCreateProgramWithSource](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clCreateProgramWithSource.html)

    * 编译

      [clBuildProgram](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clBuildProgram.html)

    参数设置什么的太多了，不一一详解了，举例:

        const char *src=getAllFile("kernal1.cl");
        cl_program program = clCreateProgramWithSource(context, 1, &src, 0, 0);
        err = clBuildProgram(program, 0, 0, 0, 0, 0);

    这里的`kernal1.cl`使用特殊的语言写的，感觉几乎就是c语言，~~只不过不能调用函数~~不过还是有一些限制的。

        //kernal1.cl
        __kernel void adder(__global const float* a, __global const float* b, __global float* result)
        {
          int idx = get_global_id(0);
          result[idx] = a[idx] +b[idx];
        }
       

6.  __kernel__
    相当于一次函数调用，从`program`里找一个函数调用。

        cl_kernel adder = clCreateKernel(program, "adder", &err);

7.  __Buffer__
    相当于供函数调用的参数。

    创建buffer：

        cl_mem clCreateBuffer (
            cl_context context,
            cl_mem_flags flags,
            size_t size,
            void *host_ptr,
            cl_int *errcode_ret)

    * `context` 选择上下文
    * `flags` 用来决定这个buffer的属性，比如只读，只写，是否需要初始化等等，[详见](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clCreateBuffer.html)
    * `size` `host_ptr` 用于提供初始化的数组
    * 错误码

    设置参数：

        cl_int clSetKernelArg (
            cl_kernel kernel,
            cl_uint arg_index,
            size_t arg_size,
            const void *arg_value)
    
    举例：
    
        cl_mem cl_a = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR, sizeof(cl_float) * DATA_SIZE, &a[0], NULL);
        cl_mem cl_b = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR, sizeof(cl_float) * DATA_SIZE, &b[0], NULL);
        cl_mem cl_res = clCreateBuffer(context, CL_MEM_WRITE_ONLY, sizeof(cl_float) * DATA_SIZE, NULL, NULL);
        clSetKernelArg(adder, 0, sizeof(cl_mem), &cl_a);
        clSetKernelArg(adder, 1, sizeof(cl_mem), &cl_b);
        clSetKernelArg(adder, 2, sizeof(cl_mem), &cl_res);
        
##运行程序

运行程序分两步，创建运行指令和拷贝运行结果，貌似程序真正的执行是在拷贝运行结果的时候？

创建运行指令:

    clEnqueueNDRangeKernel(
        cl_command_queue queue，
        cl_kernel kernel，
        cl_uint work_dims，
        const size_t *global_work_offset,
        const size_t *global_work_size, 
        const size_t *local_work_size,
        cl_uint num_events,
        const cl_event *wait_list,
        cl_event *event)
    
* `queue` `kernel` 指定队列和内核
* `work_dims` 设定参数的维度，注意，这个参数是用来传递给不同进程的参数，同样也是接下来三个数组的大小，kernel里可以通过`get_work_dim`来获得。
* `global_work_offset` 每一维初始下标
* `global_work_size` 每一维结束下标+1
* `local_work_size` 暂时不太清楚什么用，好像目的是吧不同进程捆绑起来便于沟通？
* `num_events` `wait_list` 用来设定在这个命令之前需要完成那些命令
* `event` 返回当前事件

拷贝运行结果：

    cl_int clEnqueueReadBuffer (
        cl_command_queue command_queue,
        cl_mem buffer,
        cl_bool blocking_read,
        size_t offset,
        size_t cb,
        void *ptr,
        cl_uint num_events_in_wait_list,
        const cl_event *event_wait_list,
        cl_event *event)
    
貌似有些和同步异步有关的东西，还不太明白，具体[戳这里](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/clEnqueueReadBuffer.html)。

举例：

    size_t work_size=DATA_SIZE-1;
    cout<<"begin run...."<<endl;
    long long tt=clock();
    err = clEnqueueNDRangeKernel(cqueue, adder, 1, 0, &work_size, 0, 0, 0, 0);
    cout<<"err:"<<err<<' '<<work_size<<' '<<clock()-tt<<endl;
    
    tt=clock();
    vector<float> res(DATA_SIZE);
    err = clEnqueueReadBuffer(cqueue, cl_res, CL_TRUE, 0, sizeof(float) * DATA_SIZE, &res[0], 0, 0, 0);

这样一个简易的并行加法器就完成了， [完整代码](https://github.com/cjld/opencl1)

简单的利用这个加法器对我的CPU(i5-2410M)以及GPU(AMD HD 6470M)的性能做了一下测试：

一共500个进程，每个进程做10000000次浮点加法，结果分别如下：

* AMD平台的GPU(AMD HD 6470M) device __260ms__ 相当于192GHz
* AMD平台的CPU(i5-2410M) device __530ms__ 相当于94GHz
* intel平台的CPU(i5-2410M) device __136ms__ 相当于367Ghz

这不科学啊...我为了防止他偷偷帮我加优化，还特地改成了fibonacci数列求解，难不成你都智能到自动矩阵乘法优化了？还有显卡跑的还没cpu快你干嘛吃的...

另外我已经彻底成为A卡黑了，在win8下驱动装了一年都没装好，而且回win7用opencl还会莫名其妙的蓝屏。
