---
layout: post
title: "openCL C++ 封装api学习"
description: ""
category: 
tags: [openCL]
---

openCL同样也有自己的c++包装器，因为本质还是基于原来的c实现的，只不过用了一些c++的特性来简化编程，所以只需要一个单独的头文件就可以了。

单独需要的头文件为 `cl.hpp`，我将不同版本的`cl.hpp`和官方的reference放到[这里](https://github.com/cjld/opencl1/tree/master/opencl_c%2B%2B)了，使用的时候直接将`#include <CL/cl.h>`改为`#include <CL/cl.hpp>`即可。

另外由于CUDA暂时不支持1.2，得先用1.1的`cl.hpp`。

有了c++的包装以后opencl的结构显得清晰多了，下面一一列举几个模块的用法：

1.  错误信息获取
    
    经过c++包装过的错误信息可以很方便的获取了，使用的是c++的try-throw-catch机制。
    之前还从来没听说过这种玩意，稍微Google了下，用法大概如下：

        try {
          ...          
          throw x;
          ...
        } catch(A a) {
          ...
        }

    假设throw出来的x类型正好为A的话，就会将x赋值给a，然后执行catch代码块里的东西，在过程里的throw同样有效。

    想要在openCL C++ wrapper里使用，就必须定义宏：`#define __CL_ENABLE_EXCEPTIONS`，然后使用如下代码抓取错误：

        try {

          ...

        } catch (cl::Error err) {
          cerr<<"ERROR : "<<err.what()<<'('<<err.err()<<')'<<endl;
        }

2.  Platform获取

        vector<cl::Platform> platformList;
        cl::Platform::get(&platformList);

3.  Platform info

        void printPlatformInfo(vector<cl::Platform> &list) {
          cout<<endl<<"Platform size : "<<list.size()<<endl;
          const char name[][20]={"PROFILE","VERSION","NAME","VENDOR","EXTENSIONS"};
          for (size_t i=0; i<list.size(); i++) {
            cout<<endl;
            for (size_t j=0; j<=4; j++) {
              string info;
              list[i].getInfo((cl_platform_info)(CL_PLATFORM_PROFILE+j),&info);
              cout<<name[j]<<" : "<<info<<endl;
            }
          }
        }

4.  Device获取

        vector<cl::Device> deviceList;
        platformList[0].getDevices(CL_DEVICE_TYPE_ALL,&deviceList);

5.  Device info

        void printDeviceInfo(vector<cl::Device> &list) {
          cout<<endl<<"Device size : "<<list.size()<<endl;
          for (size_t i=0; i<list.size(); i++) {
            cout<<endl;
            string info;
            list[i].getInfo(CL_DEVICE_NAME,&info);
            cout<<"name : "<<info<<endl;
            cl_uint size;
            list[i].getInfo(CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS,&size);
            cout<<"max dim number : "<< list[i].getInfo<CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS>() <<endl;
            vector<size_t> sizes=list[i].getInfo<CL_DEVICE_MAX_WORK_ITEM_SIZES>();
            cout<<"max work item size : ";
            for (size_t j=0;j<sizes.size();j++)
              cout<<sizes[j]<<" ";
            cout<<endl;
         //   cout<<"max sub device : "<<CL_DEVICE_PARTITION_MAX_SUB_DEVICES<<endl;
          }
        }

    这里有个比较有意思的地方，就是getInfo，这里用了模板，因为这个函数的返回值随着枚举类的不同返回值也是不一样的，当然，以前的用法也是允许的。

    另外最近才得知，原来大部分显卡都最多支持3维，如我的GT750m，输出`CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS`，得出就是3。

7.  SubContext

    openCL1.2的特性，目的可以将一部分计算资源腾出来供ui操作或者其他。

        cl_int createSubDevices(
            const cl_device_partition_property_ext * properties,
            VECTOR_CLASS<Device>* devices);

    用法和`clCreateSubDevices`类似，详见[这里](http://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/clCreateSubDevices.html);

6.  Context

        cl::Context context(deviceList);

    简单明了，多么舒畅，虽然context也有其他创建方法，但都不如这个简洁，这一步可以省去参数`cl_context_properties`。

7.  CommandQueue

        cl::CommandQueue cmdQueue(context,deviceList[0]);

8.  Program & Build

        cl::Program::Sources src(
            1,
            make_pair(getAllFile("kernal1.cpp"),0)
        );

        cl::Program program(context,src);
        program.build(deviceList);

9.  Kernel & Buffer

        cl::Buffer bufferA(context, CL_MEM_READ_ONLY, LIST_SIZE * sizeof(int));
        cl::Buffer bufferB(context, CL_MEM_READ_ONLY, LIST_SIZE * sizeof(int));
        cl::Buffer bufferC(context, CL_MEM_WRITE_ONLY, LIST_SIZE * sizeof(int));

        cl::Kernel kernel(program, "adder");
        kernel.setArg(0, bufferA);
        kernel.setArg(1, bufferB);
        kernel.setArg(2, bufferC);

10. write Buffer & set Kernel Dim

        int A[LIST_SIZE];
        for (int i=0;i<LIST_SIZE;i++) A[i]=i;
        cmdQueue.enqueueWriteBuffer(bufferA, CL_TRUE, 0, LIST_SIZE * sizeof(int), A);
        cmdQueue.enqueueWriteBuffer(bufferB, CL_TRUE, 0, LIST_SIZE * sizeof(int), A);

        cmdQueue.enqueueNDRangeKernel(kernel, cl::NDRange(), cl::NDRange(LIST_SIZE), cl::NDRange());

    这里的`NDRange`比较有意思，他有4个构造函数，分别是带0~3个参数，以此代表维度，`cl::NDRange(5,6,7)`意思就是3维，每个维度大小分别为5,6,7。

11. read Buffer

        cmdQueue.enqueueReadBuffer(bufferC, CL_TRUE, 0, LIST_SIZE * sizeof(int), A);

总的来说还是挺清晰明了的呢,openCL C++ wrapper 比原来的代码结构清晰多了，另外`cl.hpp`里面的代码可读性也很高，有时候不需要查手册直接翻源文件就好了。
