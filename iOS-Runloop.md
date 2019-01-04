---
title: iOS Runloop
date: 2019-01-04 14:46:01
tags:
---

> 这里记录下iOS中Runloop相关的知识点，以备以后复习总结。

#### 先来说下Runloop相关的概念：

Runloop，顾名思义就是一个线程的循环，在有事件发生的时候处理事件，没事件的时候休眠。

不管iOS还是Android，都有一个Event-Loop系统(iOS->Runloop, Android->Looper)，以此来达到CPU的最优使用。因为本人是iOS开发，所以本篇文章以iOS的Runloop来研究举例。

根据[苹果开发者文档中的解释](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23)，iOS的Runloop对象，有两个类[NSRunloop]()和[CFRunloop]()。而NSRunloop则是在CFRunloop的基础上封装的更高一级的类。有一个词叫做`toll-free bridged`即
```
某些数据类型能够在Core Foundation和Foundation之间互换使用，
可被互换使用的数据类型被称为Toll-Free Bridged类型。
这意味着同一数据类型即可以作为Core Foundation函数的参数，
也可作为接收者向其发送Objective-C消息。
Core Foundation与Foundation之间交换使用数据类型的技术被称为Toll-Free Bridging 。
```
由此可见，两个类基本可以理解为同一个类型，但是CoreFoundation框架更加底层一点。**且暴露出来的接口更多一些，更加方便我们操控runloop对象**
<!--more-->

#### Runloop和线程间的关系
总结一下就是：
iOS中每个线程都会对应一个runloop，这是系统自动为我们生成的，无需我们自己去创建。
获取当前线程的runloop的方法有两种：
```
#NSRunloop
[NSRunloop currentRunloop];

#CFRunloop
CFRunLoopGetCurrent
```
而**主线程的runloop是默认开启的，子线程中的runloop采取懒加载的模式，如果不调用，则不会生成，需要手动开启**


#### Runloop对象的构成：
![](https://upload-images.jianshu.io/upload_images/1241385-d3ad9a7df950d2d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
一个Runloop包含若干个mode, 每个mode中则都包含Source/Timer/Observe, 
每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。
如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。
这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。
```
CFRunLoopSourceRef 是事件产生的地方。
Source有两个版本：`Source0 和 Source1`
• `Source0 只包含了一个回调（函数指针），它并不能主动触发事件`。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
• `Source1 包含了一个 mach_port 和一个回调（函数指针),   能主动唤醒 RunLoop 的线程`

CFRunLoopTimerRef 是基于时间的触发器，它和 NSTimer 是toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

CFRunLoopObserverRef 是观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};

typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```
上面的 Source/Timer/Observer 被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。
***如果一个runloop在启动的时候，内部mode 中一个 item 都没有，则runloop直接退出***

下面看下代码构成：
```
struct __CFRunLoopMode {
CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
CFMutableSetRef _sources0;    // Set
CFMutableSetRef _sources1;    // Set
CFMutableArrayRef _observers; // Array
CFMutableArrayRef _timers;    // Array
...
};

struct __CFRunLoop {
CFMutableSetRef _commonModes;     // Set
CFMutableSetRef _commonModeItems; // Set<Source/Observer/Timer>
CFRunLoopModeRef _currentMode;    // Current Runloop Mode
CFMutableSetRef _modes;           // Set
...
};
```
这里有个概念叫 “CommonModes”：一个 Mode 可以将自己标记为”Common”属性（通过将其 ModeName 添加到 RunLoop 的 “commonModes” 中）。每当 RunLoop 的内容发生变化时，RunLoop 都会自动将 _commonModeItems 里的 Source/Observer/Timer 同步到具有 “Common” 标记的所有Mode里。

应用场景举例：**主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。这两个 Mode 都已经被标记为”Common”属性**。DefaultMode 是 App 平时所处的状态，TrackingRunLoopMode 是追踪 ScrollView 滑动时的状态。`当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。`

**有时你需要一个 Timer，在两个 Mode 中都能得到回调，一种办法就是将这个 Timer 分别加入这两个 Mode。还有一种方式，就是将 Timer 加入到顶层的 RunLoop 的 “commonModeItems” 中。”commonModeItems” 被 RunLoop 自动更新到所有具有”Common”属性的 Mode 里去。**

**Runloop中我们常用的几种mode如下：**
| mode名称 | 功能及调用时机 |
| :-: | :-: | 
|kCFRunLoopDefaultMode| App 平时所处的状态 |
| UITrackingRunLoopMode | 追踪 ScrollView 滑动时的状态，或者Mac OS用户正在执行的拖拽操作 |
| NSRunLoopCommonModes | 即上面代码中_commonModeItems属性，当Runloop内容发生变化的时候，会自动同步此集合中的元素到_commonModes中的各个mode中 |


#### Runloop内部执行流程：
[苹果官方文档]()中这么解释：
![](https://upload-images.jianshu.io/upload_images/1241385-46b59d649b8c6963.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
ibireme博客中的图：
![图摘自ibireme博客](https://upload-images.jianshu.io/upload_images/1241385-1e91e54da8977f7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 一些相关问题
##### 1. UI布局刷新和runloop的关系？
当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

##### 2. GCD中asyncBlock和runloop的关系？？
当调用 dispatch_async(dispatch_get_main_queue(), block) 时，**libDispatch 会向主线程的 RunLoop 发送消息，RunLoop会被唤醒**，并从消息中取得这个 block，并在回调 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__() 里执行这个 block。但这个逻辑仅限于 dispatch 到主线程，dispatch 到其他线程仍然是由 libDispatch 处理的。

##### 3. runloop中各个mode之间关系？存在一个和多个有什么区别？mode中source、timer、observe之间关系？
一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

##### 4. mode中timer和NSTimer关系？
CFRunLoopTimerRef 是基于时间的触发器，它和 NSTimer 是toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

##### 5. 怎么往runloop的source中添加事件？此种方法添加的事件和正常方式添加的有什么区别？
1) 使用`performSelect`等`NSObject`的方法
```
performSelectorOnMainThread: adds the target and the selector to a special input source called
performSelector input source. 
The run loop of the main thread dequeues that input source and handles the method call one by one, 
as part of its event processing loop.
```
可以看到，`performSelect`方法，是添加到当前runloop的`input Source`中，在下一个runloop处理周期中，执行事件。
`NSObject`中的`performSelect`方法，都是**NSObject**自己持有了当前线程的runloop对象，然后去操作它。




##### 6. runloop有很多中mode，如果以一种mode跑runloop，当想切换到另外一种mode的时候怎么切换呢？ 主线程怎么在几种mode间切换的呢？
**使用CFRunloop**
`CFRunLoopStop(CFRunLoopGetMain());`

##### 7. AutoreleasePool
App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。

第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

##### 8. 事件响应

苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 __IOHIDEventSystemClientQueueCallback()。

当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考[这里](http://iphonedevwiki.net/index.php/IOHIDFamily)。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，之后在回调 __IOHIDEventSystemClientQueueCallback() 内触发的 Source0，Source0 再触发 _UIApplicationHandleEventQueue()，进行应用内部的分发。

_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

![](https://upload-images.jianshu.io/upload_images/1241385-8c84f0a32272a535.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 9. 手势识别
当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。

苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。

当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。


[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
[Runloop](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runloop.html)
[https://juejin.im/entry/587c2c4ab123db005df459a1](https://juejin.im/entry/587c2c4ab123db005df459a1)
(https://stackoverflow.com/questions/12091212/understanding-nsrunloop/26357265)
[Understanding NSRunLoop](https://stackoverflow.com/questions/12091212/understanding-nsrunloop)
[What's the relationship between UI animation and the main runloop](https://stackoverflow.com/questions/23013072/whats-the-relationship-between-ui-animation-and-the-main-runloop)
[Inter-Process Communication](https://nshipster.com/inter-process-communication/)
[掘金iOS底层原理探究-Runloop](https://juejin.im/post/5af590c5f265da0b7964f1c2)
[github runloop](https://github.com/ming1016/study/wiki/CFRunLoop)
