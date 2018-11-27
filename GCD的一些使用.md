---
title: GCD的一些使用
date: 2018-11-27 17:48:32
tags:
---

> 本片文字意在记录gcd的一些用法

#### 1. dispatch_semaphore
> 相关方法：
dispatch_semaphore_create()   //创建信号量，设置能同时执行的最大线程数
dispatch_semaphore_wait()      //信号量-1
dispatch_semaphore_signal()  //信号量+1

> dispatch_semaphore_wait方法：
Decrement the counting semaphore. If the resulting value is less than zero, this function waits for a signal to occur before returning.
信号量减1，如果结果小于0，会阻塞当前线程等待新信号的出现。
如果为DISPATCH_TIME_FOREVER，则会一直等待
> dispatch_semaphore_signal方法：
Increment the counting semaphore. If the previous value was less than zero, this function wakes a thread currently waiting in [dispatch_semaphore_wait]()
信号量加1，如果之前的信号量小于0，则会唤醒dispatch_semaphore_wait方法中等待着的线程

<!--more-->

使用的方法：**先降后升**
先创建信号量，设置能同时执行的最大线程数。如果为1，则同时可执行的线程就是1个，等进入到耗时操作1的时候，先让信号量降低，然后处理完成之后，再调用signal方法使信号量升高，在别的耗时操作中，也是一样的逻辑。
那么，在耗时操作2开始的时候，由于耗时操作1中信号量为0，且等待时间为`DISPATCH_TIME_FOREVER`，**此时则会一直等待**。

##### 常规操作如下：
```
/**
* 常规dispatch_semaphore使用方法（多个网络请求，限制最大并发数量）
*/
- (void)dispatchSemaphoreTest1 {
dispatch_queue_t queue = dispatch_queue_create(0, 0);
dispatch_semaphore_t sema = dispatch_semaphore_create(2);

NSLog(@"--------------------------dispatch begin-----------------");
dispatch_async(queue, ^{
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success1-----------------");
dispatch_semaphore_signal(sema);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail1-----------------");
dispatch_semaphore_signal(sema);
}];

});

NSLog(@"--------------------------dispatch async complete1-----------------");
});

dispatch_async(queue, ^{
//Thread4: EXC_BAD_INSTRUCTION(code=EXC_I386_INVOP,subcode=0x0)
//https://stackoverflow.com/questions/24337791/exc-bad-instruction-code-exc-i386-invop-subcode-0x0-on-dispatch-semaphore-dis
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(8 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success2-----------------");
dispatch_semaphore_signal(sema);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail2-----------------");
dispatch_semaphore_signal(sema);
}];
});
NSLog(@"--------------------------dispatch async complete2-----------------");
});

dispatch_async(queue, ^{
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success3-----------------");
dispatch_semaphore_signal(sema);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail3-----------------");
dispatch_semaphore_signal(sema);
}];
NSLog(@"--------------------------dispatch async complete3-----------------");
});

NSLog(@"--------------------------dispatch complete-----------------");
}
```
运行之后，竟然崩溃了！！
报错信息：
```
EXC_BAD_INSTRUCTION(code=EXC_I386_INVOP,subcode=0x0)
```
查询了[stackoverflow](https://stackoverflow.com/)之后，发现[此篇文章](https://stackoverflow.com/questions/24337791/exc-bad-instruction-code-exc-i386-invop-subcode-0x0-on-dispatch-semaphore-dis)
此问题是使用了`dispatch_group_enter`,`dispatch_group_leave`和`dispatch_group_wait`三个方法的途中，遇到的这个问题，他的写法如下：
```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);
dispatch_group_t group = dispatch_group_create();

for (...) {
if (...) {
dispatch_group_enter(group);
dispatch_async(queue, ^{

[self webserviceCall:url onCompletion:^{
dispatch_group_leave(group);
}];
});
}
}

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
dispatch_group_wait(group, dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)));
dispatch_sync(queue, ^{
// call completion handler passed in by caller
});
});
```
其中回答说：
![](https://upload-images.jianshu.io/upload_images/1241385-17175c3346c2a442.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是在使用的过程中，调用已经释放了的group才导致的这个问题。

受此文章启发，我考虑到上面的写法是否也是由于这个原因呢？
然后我把`dispatch_semaphore_t`作为当前类的属性，强引用一下，发现解决了这个问题。
```
//只需要在前面创建semaphore和queue的时候，强引用一下，然后再后续的调用中，都修改为self.xxx即可
self.sema = dispatch_semaphore_create(2);
self.queue = dispatch_queue_create(0, 0);
```
-------------------


##### 异步变同步
下面的实现是一种取巧的方法，达到了同步的效果，
```
- (void)testDispatchSemaphore {
NSLog(@"--------------------------begin-----------------");
dispatch_semaphore_t sema = dispatch_semaphore_create(0);
dispatch_queue_t que = dispatch_get_global_queue(0, 0);

dispatch_async(que, ^{
NSLog(@"--------------------------async begin-----------------");
[NSThread sleepForTimeInterval:3];
dispatch_semaphore_signal(sema);
NSLog(@"--------------------------async complete-----------------");
});


dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
NSLog(@"--------------------------complete-----------------");

}

//log如下：
--------------------------begin-----------------
--------------------------async begin-----------------
--------------------------async complete-----------------
--------------------------complete-----------------
```


#### 2. 多个异步操作结束之后，统一操作
> 用到的方法：
dispatch_group_enter()
dispatch_group_leave()
dispatch_group_notify()

注意：**这里enter和leave要成对出现**
```
- (void)dispatchGroupTest {
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("com.fxeyeQueue.www", DISPATCH_QUEUE_CONCURRENT);

dispatch_group_enter(group);
dispatch_group_enter(group);
dispatch_group_async(group, queue, ^{

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{


NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success1-----------------");
dispatch_group_leave(group);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail1-----------------");
dispatch_group_leave(group);
}];

[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success2-----------------");
dispatch_group_leave(group);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail2-----------------");
dispatch_group_leave(group);
}];
});

});

dispatch_group_notify(group, queue, ^{
NSLog(@"--------------------------notify-----------------");
});

NSLog(@"--------------------------complete-----------------");

}
//log如下：
14:08:58.014812+0800 testOperation[6098:474448] --------------------------complete-----------------
14:09:01.035264+0800 testOperation[6098:474448] --------------------------fail2-----------------
14:09:01.035430+0800 testOperation[6098:474448] --------------------------fail1-----------------
14:09:01.035564+0800 testOperation[6098:474778] --------------------------notify-----------------

```
也有另外一种写法：
```
把两个请求分别放到两个异步group操作中，即：
- (void)dispatchGroupTest {
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("com.fxeyeQueue.www", DISPATCH_QUEUE_CONCURRENT);

dispatch_group_async(group, queue, ^{

NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
dispatch_group_enter(group);
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success1-----------------");
dispatch_group_leave(group);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail1-----------------");
dispatch_group_leave(group);
}];
});

dispatch_group_async(group, queue, ^{

NSString *imgUrl = @"https://www.baidu.com/img/bd_logo1.png";
dispatch_group_enter(group);
[[AFHTTPSessionManager manager] GET:imgUrl parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
NSLog(@"--------------------------success2-----------------");
dispatch_group_leave(group);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
NSLog(@"--------------------------fail2-----------------");
dispatch_group_leave(group);
}];
});

dispatch_group_notify(group, queue, ^{
NSLog(@"--------------------------notify-----------------");
});

NSLog(@"--------------------------complete-----------------");

}
```
从log上看，也是可以达到效果的，
但是！此种写法在时机使用中，遇到了一个问题，会造成整个APP卡死。。
然后[Bugly]()报错说`max operation count reached`，思来想去也没找到问题。还请知道解决方法的大佬给予明示！thx~
-----------------------
今天又接着Google，终于找到了问题所在。受[此文](https://www.jianshu.com/p/29152d90cef7)启发。
项目中代码是在WKWebview的`didFinishNavigation`方法中调用请求方法。
然后
```
/**
* 两个请求结束之后，统一操作
*/
- (void)groupRequest {
dispatch_group_t group = dispatch_group_create();
self.group = group;
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

dispatch_group_enter(self.group);
dispatch_group_enter(self.group);
dispatch_group_async(self.group, queue, ^{
[某网络请求方法 {
dispatch_group_leave(self.group);
}];
});

dispatch_group_async(self.group, queue, ^{
[某网络请求方法 {
dispatch_group_leave(self.group);
}];
});

dispatch_group_notify(self.group, queue, ^{
//获取到之后，刷新页面
[self reloadViewWithModelData];
});
}
```
由于webview的加载过程不可控，所以可能存在**多次调用此方法的情况**，然后就会创建很多的group，造成Bugly报出的[max operation count reached](https://www.google.com.hk/search?safe=strict&source=hp&ei=hZH7W_6WFMvovgSs3arQBw&q=dispatch+group+max+operation+count+reacthed+&btnK=Google+%E6%90%9C%E7%B4%A2&oq=dispatch+group+max+operation+count+reacthed+&gs_l=psy-ab.3...2814.32604..33494...5.0..2.452.10919.0j14j22j8j1......0....1..gws-wiz.....0..0j0i12j0i10j0i30j0i8i30j33i160.XAbprWZQ1Wo)问题，因此***懒加载group***理论上就能解决此问题。上面代码改为：
```
- (void)groupRequest {
if (self.group == nil) {
dispatch_group_t group = dispatch_group_create();
self.group = group;
}
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
......
}
```

#### 附文中代码的[Demo](https://github.com/WooNoah/GCDDemo)
