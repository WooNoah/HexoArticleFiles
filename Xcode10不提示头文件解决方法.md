---
title: Xcode10不提示头文件解决方法
date: 2018-09-30 11:06:19
tags:
---

Xcode10 新增了一个构建系统起名“New Build System”（新构建系统），在Xcode10正式发布会变成了Xcode的默认Build System，旧的构建系统称为 legacy build system （传统构建系统), `在使用新的构建系统时, 导入头文件时 xcode 会出现闪退或者不提示的情况`, 最终发现可能是因为 xcode10 默认了 新构建系统导致的 , 但为什么会导致原因尚不清楚.

具体的解决方法是将默认的构建系统还是设置为传统的构建系统, 具体的设置路径如下图
##### 1.点击`File->Project Setting`

![](https://upload-images.jianshu.io/upload_images/1241385-e7b2093917fe1282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 2.Build System中选中`Legacy Build System`
![](https://upload-images.jianshu.io/upload_images/1241385-071dfbbc1d473b05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[参考资料](https://www.cnblogs.com/canfixme/p/9704095.html)
