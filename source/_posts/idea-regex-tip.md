---
title: Idea使用正则表达式替换小技巧
date: 2019-12-05 11:23:57
tags: 
- idea
- tools
categories:
- Tips
description: Idea使用正则表达式替换小技巧
---

日常开发经常会遇到需要复杂的替换。比如最近在优化gradle脚本时，要把之前写的 rootProject.ext.deps['abc'] 写成 "${dpes['abc']}"。项目是复杂项目，这类修改分布在很多build.gradle中。一个一个改实在费力。简单研究了一下idea正则表达式替换的功能，发现非常好用。

查询正则表达式：
```
rootProject\.ext\.deps(.*)?
```
替换内容
```
\"\$\{libs$1\}\"
```

具体正则表达式我就不多说了。这里有是一个正则表达式的功能，使用regex capturing groups来做替换。idea中使用`$1`表达被捕获的内容位置。当然一个正则表达式中也可以有多个capturing groups，然后对应替换`$1`,`$2`... 顺序由左至右捕获的顺序。
上面这个例子中`(.*)?`为一个capturing group


[Idea官方文档](https://www.jetbrains.com/help/idea/tutorial-finding-and-replacing-text-using-regular-expressions.html)

Cheers