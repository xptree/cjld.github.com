---
layout: post
title: "Google-Code-Prettfy添加代码高亮"
description: ""
category: 
tags: [GoogleCodePrettfy]
---

想给自己的jekyll blog加一下代码高亮,上网搜了下,有这么主流的三种方法:

* Redcarpet
  Redcarpet作为markdown引擎可以使用以下代码来添加代码高亮

      ``` ruby
      require 'rubygems'

      def foo
      puts 'foo'
      end

      #comment
      ```

  详见[这里](http://ztpala.com/2011/10/27/code-highlighting-jekyll/)
* Pygments
  应该是jekyll原生支持的方法，这个[链接](http://zyzhang.github.io/blog/2012/08/31/highlight-with-Jekyll-and-Pygments/)讲的比较详细。

* Google-Code-Prettfy
  通用的方法，Google造福人类，只要嵌入js和css即可。

因为坑爹的windows原因，ruby各种抽风，所以我只能用最后一个办法了，[这篇博文](http://www.lidongkui.com/use-prettify-to-highlight-code)写的不错，利用jquery可以做到只需要改外部而不需要改`_post/`文件夹下就可以实现代码高亮。

效果测试：

    #include <fuckly>
    
    int main() {
      int a=1,b=a*a;
      return 0;
    }

最后因为我在“关于页”和“零散页”滥用了pre标签，导致这些地方也有高亮了，不管了，还挺好看的。

缺点还是有的，加载太慢了，用chrome打开以后要过将近一秒才能让代码高亮。

###__UPDATA__

过了一秒才出现的BUG查出来了，这个[链接](http://www.lidongkui.com/use-prettify-to-highlight-code)使用加载代码高亮效果的代码是：

    $(window).load(function(){
         $("pre").addClass("prettyprint");
         prettyPrint();
    })

因为jekyll使用了disqus，相当于他在等待disqus搞完以后才执行这段代码，因此直接改成这样就好了：

    $(document).ready(function(){
         $("pre").addClass("prettyprint");
         prettyPrint();
    })

这样等待页面静态元素加载完就会执行这段代码，而不是等到disqus加载完。
