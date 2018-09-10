---
title: iOS UIScrollView Bug
date: 2018-09-10 15:28:45
tags:
---

> 这里记录一下，开发中遇到的一些问题：以便以后查阅

#### 1. `contentOffset`问题
```
先来说下环境：xcode 9.4.1 模拟器：iPhone SE 11.4
我们先在一个view中放置一个scrollview，
然后设置他的偏移量：self.scv.contentOffset = CGPointMake(-50, 0);
然后点击一个别的页面，比如UITabbarController中别的item，
再次回到带有scrollview的页面的时候，scrollview会把偏移量重置
```
现象如图：
![示例GIF](https://upload-images.jianshu.io/upload_images/1241385-d44a470354f3e70d.gif?imageMogr2/auto-orient/strip)

[1.代码](https://gitee.com/WooNoah/UIScrollviewBugDemo.git)不在GitHub上传了，传在[码云仓库](https://gitee.com/WooNoah/UIScrollviewBugDemo.git)

