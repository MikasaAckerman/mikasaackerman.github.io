---
layout: post
title: "报错：NSLayoutConstraintNumberExceedsLimit"
date: 2017-11-16 17:27:11.000000000 +09:00
tags: 回梦游仙
---

更新Xcode9后，自己的程序在模拟器iOS10 11都没问题 但是iOS9 有个页面提示`This NSLayoutConstraint is being configured with a constant that exceeds internal limits. A smaller value will be substituted, but this problem should be fixed. Break on void _NSLayoutConstraintNumberExceedsLimit() to debug. This will be logged only once. This may break in the future. `然后界面就卡死就进不去了。自己也按照上文描述的加了断点也不行。

![](/assets/images/2017/NSLayoutConstraintNumberExceedsLimit1.png)

在一番不知所措之后，各种查搜，发现Stack Overflow有篇文章说的是不加括号，于是乎就试了一下，果然有提示，按照提示顺藤摸瓜找到了出现错误的类，然后查看代码发现了问题。

![](/assets/images/2017/NSLayoutConstraintNumberExceedsLimit2.png)

![](/assets/images/2017/NSLayoutConstraintNumberExceedsLimit3.png)

该界面有个`textView.contentSize.height`，因为自己不能像算label的高度那样算好textView的高度只能用这个方法了，就报错了，没办法只能改成了iOS10才用textView。

后来经过测试，发现iOS9.0 9.1不行，但是9.3可以（模拟器没有9.2），最后改成了9.3及之后用textView，其他时候用Label。


p.s.记得之前有一次iOS9模拟器打印出类似上面的东西也是让我加断点去调试，打了之后感觉完全没有有用信息，而且完全不影响工程，就忽略了。。下次再遇到需提示自己随时截图。


