---
title: 打造多彩的Xcode console系统
date: 2018-11-30 17:02:59
tags:
---



一、 下载日志、log分级模块

1.安装**[CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)**
```
pod 'CocoaLumberjack'
```
2. 做出相关配置**[Get started using Lumberjack](https://raw.githubusercontent.com/CocoaLumberjack/CocoaLumberjack/master/Documentation/GettingStarted.md)**
```
//PrefixHeader.pch文件中
#define LOG_LEVEL_DEF ddLogLevel
#import <CocoaLumberjack/CocoaLumberjack.h>
#ifdef DEBUG
static DDLogLevel __unused ddLogLevel = DDLogLevelAll;
#else
static DDLogLevel __unused ddLogLevel = DDLogLevelOff;
#endif

//AppDelegate.m中 didFinishLaunchingWithOptions
[DDLog addLogger:[DDOSLogger sharedInstance]]; // Uses os_log

//    日志写入
//    DDFileLogger *fileLogger = [[DDFileLogger alloc] init]; // File Logger
//    fileLogger.rollingFrequency = 60 * 60 * 24; // 24 hour rolling
//    fileLogger.logFileManager.maximumNumberOfLogFiles = 7;
//    [DDLog addLogger:fileLogger];
[[DDTTYLogger sharedInstance] setColorsEnabled:YES];

DDLogVerbose(@"Verbose");
DDLogDebug(@"Debug");
DDLogInfo(@"Info");
DDLogWarn(@"Warn");
DDLogError(@"Error");

```

二、下载xcode插件，让xcode支持console栏的多颜色
**[XcodeColors](https://github.com/robbiehanson/XcodeColors)**

但是Xcode8之后苹果禁止了使用自定义插件。所以目前导入之后，console栏无法显示出彩色。

这里找到一个解决方法（没有尝试过）先记录下：
https://blog.csdn.net/kcetry/article/details/79339712

首先我们知道，从Xcode8开始，Xcode屏蔽了第三方插件，导致插件无法使用。 
解决方法是使用证书对Xcode进行签名(Github上[XVim2](https://github.com/XVimProject/XVim2)的指引)，再安装[XcodeColors](https://github.com/robbiehanson/XcodeColors)。 
需要注意的是[第二步](https://github.com/XVimProject/XVim2/blob/master/SIGNING_Xcode.md)
![](https://upload-images.jianshu.io/upload_images/1241385-e50931e2fc7715f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
假如你已经有开发者证书，并且用第二个证书签名，那么第三步的命令就是 
`sudo codesign -f -s WV87GNFZXS /Applications/Xcode.app`
安装XcodeColors时，别忘了执行 
`./update_compat.sh` 
最后还要添加环境变量 
Product->Scheme->Edit Scheme->Run->Environment Variables 
![](https://upload-images.jianshu.io/upload_images/1241385-f6725206bbad19c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
