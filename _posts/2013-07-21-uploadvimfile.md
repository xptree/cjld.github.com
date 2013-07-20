---
layout: post
title: "vim配置文件打包"
description: ""
category: 
tags: [Vim]
---
随着自己vim功能的增加，插件越来越多，决定把自己的vimfiles文件打包一个，并且还希望vimfiles不需要依赖外部文件，比如`ctags`也一并打包进去。

在github上创建了个新的repo，这时候发现个碍眼的东西，`_vimrc`在`vimfiles`文件夹的外面，`vimfiles`里面的脚本文件vim不是也会自动运行一遍么？于是我把`_vimrc`改名为`vimrc.vim`放到了`vimfiles\plugin`里面，发现同样也奏效了，于是乎就直接删掉原来的vimrc，用`vimfiles\plugin`里面的代替，这样就可以做到vimfile和外部文件无关，下次再重装vim，一个`git clone`，自己熟悉的vim就回来了~

[github repo](https://github.com/cjld/vimfile)
