---
layout: post
title: "解决windows GPU计算时间限制"
description: ""
category: 
tags: [openCL]
---
windows如果发现单次GPU计算时间过长，会把这次计算任务直接掐掉钟来，目的是为了防止GPU出现死锁而无暇顾及桌面图形化界面，在win8以前是超过5s掐掉，win8是超过2s掐掉，症状是出现错误-5。

这种机制被称为是 `TDR`，可以禁用掉，详见[这里](http://msdn.microsoft.com/en-us/library/windows/hardware/ff569918.aspx)

* 直接禁用TDR:
  
      KeyPath   : HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\GraphicsDrivers
      KeyValue  : TdrLevel
      ValueType : REG_DWORD
      ValueData : TdrLevelOff (0) - Detection disabled 
       TdrLevelBugcheck (1) - Bug check on detected timeout, for example, no recovery.
       TdrLevelRecoverVGA (2) - Recover to VGA (not implemented).
       TdrLevelRecover (3) - Recover on timeout. This is the default value.

* 设置延时：

      KeyPath   : HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\GraphicsDrivers
      KeyValue  : TdrDelay
      ValueType : REG_DWORD
      ValueData : Number of seconds to delay. 2 seconds is the default value.

