---
title: iOS启动优化
date: 2019-04-15 11:07:49
tags:
---


> 先说结论： 启动时间 = pre-main()时间 + post-main()时间
#### App启动过程
```
①解析Info.plist 
加载相关信息，例如闪屏
沙箱建立、权限检查
②Mach-O加载 
如果是胖二进制文件，寻找合适当前CPU架构的部分
加载所有依赖的Mach-O文件（递归调用Mach-O加载的方法）
定位内部、外部指针引用，例如字符串、函数等
执行声明为__attribute__((constructor))的C函数
加载类扩展（Category）中的方法
C++静态对象加载、调用ObjC的 +load 函数
③程序执行 
调用main()
调用UIApplicationMain()
调用applicationWillFinishLaunching
```

#### 一、Pre-Main阶段
pre-main阶段的定义为APP开始启动到系统调用main函数这一段时间，这个阶段又可以分为这几个步骤

##### 1.Dylib Loading

##### 2.Rebase/Binding Symbols
fix-ups adjust pointers within an image (rebasing) and set pointers that point to symbols outside the image (binding). To speed up rebase/binding time you need fewer pointer fix-ups. Apps with large numbers of Objective-C classes, selectors and categories can add 800ms to launch times (large is 20,000). If your app uses C++ code use less virtual functions. Using Swift Structs is also generally faster.

##### 3.ObjC Runtime Setup

##### 4.Initializers
`+load`是在main函数之前就加载了，
而`+initialize`则是在该类第一次接到消息的时候才会调用，
所以，要尽量推后代码的加载时机。
但是使用`+initialize`会有一个问题，也是该方法的调用机制造成的：
在使用子类的时候，会先调用父类的`+initialize`方法。
所以，这里可以配合`dispatch_once`方法来实现类似`+load`的功能

这一步可以做的优化有：
```
①使用 +initialize 来替代 +load
②不要使用 atribute((constructor)) 将方法显式标记为初始化器，而是让初始化方法调用时才执行。
比如使用 dispatch_once(),pthread_once() 或 std::once()。也就是在第一次使用时才初始化，推迟了一部分工作耗时。
也尽量不要用到C++的静态对象。
③还有一个结论就是swift，编译起来要比OC要快
```

做下总结:
![](https://upload-images.jianshu.io/upload_images/1241385-7eaba2071dca293c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Xcode为我们提供的测试方法：
在`Edit Scheme`->`Arguments`->`Environment`中添加`DYLD_PRINT_STATISTICS`，值为`1`
![](https://upload-images.jianshu.io/upload_images/1241385-ca040a890ff8c180.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次运行，即可看到各项的结果（皆为毫秒）
![](https://upload-images.jianshu.io/upload_images/1241385-25f5f72bed2a87f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 二、Post-Main()阶段
这个阶段就是程序加载了main函数之后，到页面展示出来之间的这段时间。
这个时间可以这么计算：
`AppDelegate`的`willFinishLaunchingWithOptions`到`applicationDidBecomeActive`之间的时间，即为post-main()时间
```
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions {
NSLog(@"post-main start:%@",[self getMMTime]);
return YES;
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
NSLog(@"post-main stop:%@",[self getMMTime]);
}

-(NSString *)getMMTime{
NSTimeInterval a = [[NSDate date] timeIntervalSince1970] * 1000; // *1000 是精确到毫秒，不乘就是精确到秒
return [NSString stringWithFormat:@"%.0f", a];
}
```
得出结果为毫秒，相减即得结果。

##### 优化思路：
![](https://upload-images.jianshu.io/upload_images/1241385-e58c1a5b75e5e11d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### Reference
##### 中文文章
[WWDC之优化App启动速度](https://www.jianshu.com/p/cf95d020e1b2)
[iOS 程序 main函数之前发生什么](https://www.jianshu.com/p/5efe327ac7ea)

##### 英文文章
[iOS App Launch time analysis and optimizations](https://medium.com/@avijeet.dutta13/ios-app-launch-time-analysis-and-optimization-a219ee81447c)
[Improving Your iOS App’s Launch Time](https://techblog.izotope.com/2018/03/08/improving-your-ios-apps-launch-time/)
[iOS app launch time measurement](https://stackoverflow.com/questions/35929530/ios-app-launch-time-measurement)
[Xcode & Instruments: Measuring Launch time, CPU Usage, Memory Leaks, Energy Impact and Frame Rate](https://medium.com/@phillfarrugia/xcode-instruments-measuring-launch-time-cpu-usage-memory-leaks-energy-impact-and-frame-rate-1caf8905079f)

##### Apple Documentation
[The App Life Cycle](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/TheAppLifeCycle/TheAppLifeCycle.html)
[Overview of Dynamic Libraries](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
[Executing Mach-O Files](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/executing_files.html#//apple_ref/doc/uid/TP40001829)



