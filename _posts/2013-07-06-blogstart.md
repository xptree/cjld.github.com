---
layout: post
title: 博客建成
description: ""
category: 
tags: [jekyll, github]
---

忙活了两天，总算吧blog建好了。

**粗体测试**

*斜体测试*

*xieti*


以前的blog是在csdn上写的，[传送门](http://blog.csdn.net/jasonzhu8)。在这种地方写blog难免被各路大神鄙视缺少GEEK范*虽然我也从来不是个GEEK*，不过出于可定制性的考虑，而且自己还有一点写网页的基础，正好自己不怎么懂后台和数据库，jekyll本质上又是动态生成静态页面，也就不需要操心数据库和后台了，攻击什么的也毫无压力，于是乎就选了在github上用jekyll搭建。

顺便还学了下git、markdown和liquid，不错不错。

搭建的过程中出了挺多蛋疼的问题，多亏lyp的帮忙：

* markdown的默认引擎在有中文无序列表的时候会抽风，得换个内核，也就是在`_config.yml`里加一句`markdown: rdiscount`，rdiscount默认没有安装，还得装一下。
* jekyll在解析的时候只支持UTF-8，vim在win下的默认编码不是UTF-8，这个问题跪了好久，最后在`_vimrc`里加了这么一句`set fencs=utf8,gbk,gb2312,gb18030,cp936`，暂时一切正常，（不能直接`set enc=utf8`,要不然各种乱码）。
* 还得在jekyll内核里面改改，才能支持好utf8，可以看这篇文章[Jekyll遭遇编码问题](http://log.medcl.net/item/2012/04/jekyll-encounter-encoding-problems/)。
* 还是有时候莫名其妙的抽风，不生成`_site`目录，把git回滚几个版本又正常了...

重新捏了下首页。

改了下css，更加适应中文显示：

* li的`line-height`改为24px
* 默认中文标题字体为黑体，默认中文正文字体为宋体

终于明白jekyll的post前为什么要加日期了，因为方便页面字典序排序，同时也出现了一个问题就是，同一天发布的文章有可能后发布的反而在前面...囧
