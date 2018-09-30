---
title: 新集成cocoapod路径问题
date: 2017-03-24 14:07:58
tags:
---


### 前因

今天，在给老的项目“瘦身”的时候，发现了一些旧的，不再使用的第三方库。或者是一些正在使用的，但是目录结构看起来相当复杂的第三方。在整理的时候，看着可以说是相当的烦心。

然后，就想着给项目添加cocoapods支持

### 后果
**开始整理！！**

<!--more-->

-   终端cd到当前项目路径下
-   `pod init` pod初始化
-   然后添加需要使用pod管理的第三方库名，以及相应的版本号（如不知道，可用`pod search xxx`来搜索），整理完成之后，保存退出（`shift+zz`）
-   更新`pod update`

**然后**，就出现了以下的问题
![](https://upload-images.jianshu.io/upload_images/1241385-b93e3783eccf2beb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**The xxx target overrides the 'HEADER_SEARCH_PATHS' build setting defined in ''pods/Target Support Files/Pods-xxxxx.config. This can lead to problems with the Cocoapods installation**

**-Use the '$(inherited)' flag, or**
**-Remove the build settings from the target.**

### 处理

-	选中target
-	在`build Setting`中，分别给`Header search paths`和`other linker flags`中添加`$(inherited)`

### 结束
