---
layout: spage
title: 零散
header: Sand
group: navigation
tagline: 一些零散的、不能成为一篇文章的小trick
---
***

    **g++Trick**

    CPLUS_INCLUDE_PATH : include文件夹 可-I"dir"
    LIBRARY_PATH : lib文件夹 可-L"dir"
    -Dxxxx : 相当于define xxxx
    -U : 相当于undef

***

    **makefileTrick**

    $@ : 目的文件名
    $< : 依赖列表的第一个文件
    $^ : 整个依赖列表
    $(xx) : 宏定义

***

    **stickcd**

    1xpWZLteR4uOryFOOZlyAPfwCaUbRo5AkHoJuNx4cDbAqdFyaFxeO7L73shWMEleYTQNSSlm14AFLSCMWljySl3N3P0OOoNpzQQxNsMtP0W5hIwdDuaH*oUF1qeWUNXSdYURPOOPRUYdjqy5FQcp1GWn3Mg*Li4TtIkBf8e9hEoNzaCrVCjSpYuhreSH70G93.wvMLLMOVlqw19K5GSftAk21OtFGd*OoHmD0YsR1a8jJzV8oT9uyhRC.nbQG7*uojgdbaabdgksVdoalzAN8aFj

***

    **VimTrick**

    1234G 跳转到1234行
    N% 跳转到N%的位置
    fx Fx 跳转同行x字符
    nN 前后重复查找
    C-o C-l 跳回
    gf 打开光标处文件
    cw 删掉当前单词并进入插入模式
    <C-q> mswin可视块
    :mks  创建会话文件，用来还原会话
    pwd 显示工作目录 cd 改变工作目录 lcd 改变当前工作目录
    expand("%") 返回文件名 还有很多其他用法
    exe 运行后面的字符串命令
    :%s/fuck/fuckly/gc
    yaw 复制当前单词
    :on[ly] 仅仅保留当前分割窗口
    <C-q> I 块插入mswin下
    Netrw 一些用法
      :Exp <alt-E> <alt-F3> 打开
      v 垂直分割打开
      d 新建目录
      c 设置目录为当前目录
      % 创建新文件并打开
      

***

    **cssTrick**

    . 选择class
    # 选择id
    .sandpost>pre:nth-child(4n+2)
    .sandpost>pre 选出父亲为sandpost
    :nth-child(4n+2) 选出第4n+2个儿子（n for 0~x）
    :not(p) 不包含选择器p
    [attribute^=value] 开头
    [attribute$=value] 结尾
    [attribute*=value] 包含
    box-shadow : x y dl color

***
