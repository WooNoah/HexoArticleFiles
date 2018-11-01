---
title: iOS异步变同步
date: 2018-07-18 13:15:36
tags:
---

```
__block BOOL result = NO;
//异步线程中操作是否完成
__block BOOL inThreadOperationComplete = NO;
[[UNUserNotificationCenter currentNotificationCenter] getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
if (settings.authorizationStatus == UNAuthorizationStatusDenied) {
result = NO;
}else if (settings.authorizationStatus == UNAuthorizationStatusNotDetermined) {
result = NO;
}else if (settings.authorizationStatus == UNAuthorizationStatusAuthorized) {
result = YES;
}else {
result = NO;
}
inThreadOperationComplete = YES;
}];

while (!inThreadOperationComplete) {
[NSThread sleepForTimeInterval:0];
}
return result;
```


另外一种写法：***使用信号量***
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

--------------------------begin-----------------
--------------------------async begin-----------------
--------------------------async complete-----------------
--------------------------complete-----------------
```
