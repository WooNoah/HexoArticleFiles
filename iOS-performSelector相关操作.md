---
title: iOS performSelector相关操作
date: 2019-07-17 13:59:26
tags:
---

> 今天遇到一个问题，先来给各位看官看一下：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];
NSLog(@"hello world 1");
});

- (void)backGroundThread{
NSLog(@"—hello world2—");
}
```
**问：这段代码会打印什么？**

*可能很多人会开始猜了。这里先卖个关子，带着你的疑问继续看下去吧。*

#### 首先，总体来看一下苹果提供的`performSelector`系列的API
```
@interface NSObject (NSThreadPerformAdditions)

- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
// equivalent to the first method with kCFRunLoopCommonModes

- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
// equivalent to the first method with kCFRunLoopCommonModes
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

/****************     Delayed perform     ******************/

@interface NSObject (NSDelayedPerforming)

- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray<NSRunLoopMode> *)modes;
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay;
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(nullable id)anArgument;
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget;

@end

```
可以看到，这几个是苹果提供的常用的多线程事件相关操作。用这几个方法，我们可以在子线程处理某些方法、拉回到主线程做一些操作，在当前线程延迟多久做一些操作等等。
`performSelectorInBackground `这个就不说了。

今天主要来研究下带有`waitUntilDone `参数的方法，即`performSelectorOnMainThread: withObject: waitUntilDone:`和`performSelector: onThread: withObject: waitUntilDone:`两个方法：
先来看下官方是怎么解释waitUntilDone的wait这个参数的：
```
wait
A Boolean that specifies whether the current thread blocks until after the specified selector is performed on the receiver on the main thread. 
Specify YES to block this thread; otherwise, specify NO to have this method return immediately.
If the current thread is also the main thread, and you specify YES for this parameter, the message is delivered and processed immediately.
```
> 可以看到，这个参数是个bool类型，如果为YES，则会阻塞当前线程直到指定的方法执行完成才返回。
如果为NO, 则会立即返回。

#### 这时如果把上面例子中的代码改为这样：
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:YES];
NSLog(@"hello world 1");
});

- (void)backGroundThread{
NSLog(@"—hello world2—");
}
```
此时结果是什么？
相信大家都能看出来，肯定是 `先打印hello world 2，然后再打印hello  world1的`.
然后回归到最上面的问题。
结果是什么呢？
好奇的小伙伴们可以项目中跑一下看看：
其实结果是：**`只有hello world 1`**
```
2019-07-17 14:24:38.417458+0800 RunloopDemo[4298:394797] hello world 1
```
这是为什么呢？为什么performSelector要执行的方法没有走呢？
这个瞎猜是没用的， 我们只能去查苹果是怎么解释`performSelector:onThread:withObject:waitUntilDone:`的了：
```
## Discussion

You can use this method to deliver messages to other threads in your application. 
The message in this case is a method of the current object that you want to execute on the target thread. 

This method queues the message on the run loop of the target thread using the default run loop modes—that is, 
the modes associated with the [NSRunLoopCommonModes]  constant. 
As part of its normal run loop processing, the target thread dequeues the message (assuming it is running in one of the default run loop modes) and invokes the desired method.

You cannot cancel messages queued using this method. If you want the option of canceling a message on the current thread, 
you must use either the [performSelector:withObject:afterDelay:] or [performSelector:withObject:afterDelay:inModes:] method.

### Special Considerations

This method registers with the runloop of its current context, and depends on that runloop being run on a regular basis to perform correctly. 
One common context where you might call this method and end up registering with a runloop that is not automatically run on a regular basis is when being invoked by a dispatch queue. 
If you need this type of functionality when running on a dispatch queue, you should use [dispatch_after] and related methods to get the behavior you want.

```
可以看到：
此方法是为了把当前线程的对象传递给别的线程的，此方法会被加入到目标线程的runloop队列中，该runloop使用默认mode--NSRunLoopCommonModes。当该线程的runloop执行的时候，它会以此出队列，然后执行想要执行的方法。
由此可以得出结论，`此方法是依赖于线程的runloop的`。
而上面例子中，我们使用了`dispatch_async`创建了一个子线程，我们知道，`子线程的runloop是默认不启动的`,因此，我们添加对应的方法即可。
```
- (void)viewDidLoad {
[super viewDidLoad];
// Do any additional setup after loading the view.

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

NSLog(@"hello world 1");

NSRunLoop *currentRunLoop = [NSRunLoop currentRunLoop];
[currentRunLoop run];
});
}

- (void)backGroundThread{
NSLog(@"—hello world2—");
}
```
果然，此时再打印，结果就正常了：
```
2019-07-17 14:42:18.784324+0800 RunloopDemo[4477:428656] hello world 1
2019-07-17 14:42:18.784650+0800 RunloopDemo[4477:428656] —hello world2—
```
但是，经过一系列的操作，我发现
```
NSRunLoop *currentRunLoop = [NSRunLoop currentRunLoop];
[currentRunLoop run];
```
两行代码放的位置，也是有讲究的。
![](https://upload-images.jianshu.io/upload_images/1241385-bb5ae8163c7f9d24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一共1、2、3三种情况，结果又不尽相同。
1. 假如放在1️⃣的位置，结果如何呢？又变成`只有hello world 1了`
```
2019-07-17 14:47:44.859178+0800 RunloopDemo[4549:445631] hello world 1
```
2. 2️⃣的位置呢？`只有hello world 2了`
```
2019-07-17 14:48:29.820267+0800 RunloopDemo[4566:448005] —hello world2—
```
3. 位置3️⃣，结果就是上面那样，`是正常的`。

是不是很奇怪呢？
因此再来看下`[[NSRunLoop currentRunLoop] run]`方法苹果是怎么解释的了。
```
### Discussion

If no input sources or timers are attached to the run loop, this method exits immediately; otherwise, it runs the receiver in the `NSDefaultRunLoopMode`by repeatedly invoking [runMode:beforeDate:].
In other words, this method effectively begins an infinite loop that processes data from the run loop’s input sources and timers. 

Manually removing all known input sources and timers from the run loop is not a guarantee that the run loop will exit. 
macOS can install and remove additional input sources as needed to process requests targeted at the receiver’s thread. 
Those sources could therefore prevent the run loop from exiting. 

If you want the run loop to terminate, you shouldn't use this method. 
Instead, use one of the other run methods and also check other arbitrary conditions of your own, in a loop. A simple example would be:


BOOL shouldKeepRunning = YES; // global
NSRunLoop *theRL = [NSRunLoop currentRunLoop];
while (shouldKeepRunning && [theRL runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]]);

where `shouldKeepRunning` is set to `NO`somewhere else in the program.

```
同样可以看到，RunLoop的run方法，在有输入源或者定时器的情况下，是重复的调用`runMode:beforeDate:`方法，换句话说，他是一个无限的循环，而且手动移除输入源和定时器并不能保证runloop会退出。所以苹果建议我们使用`runMode:beforeDate:`，并且给了下面一个标准的写法。
所以，我们上面的代码就可以改为：
```
NSRunLoop *currentRunLoop = [NSRunLoop currentRunLoop];
//[currentRunLoop run];
[currentRunLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
```
此时，放置在2️⃣、3️⃣位置都是正常运行的了。
```
那位置1️⃣呢？仔细想想也是可以理解的，
因为run方法中说了，如果输入源或定时器都没有的情况下，runloop是直接退出的。
在位置1️⃣的时候开启runloop，此时并没有输入源加入, 所以此时runloop直接就退出了。
（performSelector:onThread:withObject:waitUntilDone: 方法会把方法作为输入源添加到runloop中),
所以2️⃣、3️⃣位置的时候，就不会出现此问题。
```

#### 那再来看下拉回到主线程的方法
```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
```
因为主线程的Runloop是默认开启的，所以不需要我们来处理。
把最上方中方法改为`performSelectorOnMainThread`，然后`waitUntilDone`参数为`YES`的时候，结果如下：
```
2019-07-17 15:05:58.550527+0800 RunloopDemo[4714:483346] —hello world2—
2019-07-17 15:05:58.550725+0800 RunloopDemo[4714:483400] hello world 1
```
`waitUntilDone`参数为`NO`的时候，结果如下：
```
2019-07-17 15:06:52.652589+0800 RunloopDemo[4736:485901] hello world 1
2019-07-17 15:06:52.661368+0800 RunloopDemo[4736:485839] —hello world2—
```
可以看出，此参数大致可以按“同步”、“异步”的方式来理解。

#### 因此，可以得出结论！！！！
> 在方法`- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait`中， 执行方法的时候，是把输入源添加到对应线程的RunLoop中的，但是RunLoop此时并没有启动，所以方法不用调用。话句话说：**`方法的调用顺序取决于RunLoop的启动时机`**，参照`- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;`方法的结果，我们可以知道，RunLoop的启动时机应该是在当前线程调用方法作用域的最后位置。

最终代码：
```
- (void)viewDidLoad {
[super viewDidLoad];
// Do any additional setup after loading the view.

//    [self threadInfo:@"UI THREAD"];
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
//        [self threadInfo:@"dispatch async"];

//        [self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];
[self performSelectorOnMainThread:@selector(backGroundThread) withObject:nil waitUntilDone:NO];
//        [self performSelectorInBackground:@selector(backGroundThread) withObject:nil];
//        [self performSelector:@selector(backGroundThread) withObject:nil afterDelay:2 inModes:@[NSDefaultRunLoopMode]];
//        [self performSelector:@selector(backGroundThread) withObject:nil afterDelay:(2)];

NSLog(@"hello world 1");

NSRunLoop *currentRunLoop = [NSRunLoop currentRunLoop];
//[currentRunLoop run];
[currentRunLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
});
}

- (void)backGroundThread{
NSLog(@"—hello world2—");
}
```




#### 举一反三
苹果提供的别的API, 诸如
```
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray<NSRunLoopMode> *)modes;
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay;
```
假如他们也是在子线程中调用的话，我们同样也是需要手动开启runloop的。

#### 以上。END
