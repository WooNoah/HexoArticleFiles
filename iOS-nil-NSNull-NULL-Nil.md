---
title: 'iOS nil,NSNull,NULL,Nil'
date: 2018-11-17 00:44:10
tags:
---

> 探索一下，往后台传值的过程中，遇到的一些问题： 后台交互的时候，传了一个字典，字典中有可能包含空值

先来看下现象：
```
//情况1，直接为nil
NSString *name = nil;

NSDictionary *dic = @{@"givenName":@"W",@"firstName":@"D",@"allName":name,@"gender":@(55)};
NSLog(@"%@",dic);

//此时，直接报错：
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 
'*** -[__NSPlaceholderDictionary initWithObjects:forKeys:count:]: attempt to insert nil object from objects[2]'


//情况2，传NSNull的实例变量
NSString *name = [NSNull null];

NSDictionary *dic = @{@"givenName":@"W",@"firstName":@"D",@"allName":name,@"age":@(55)};
NSLog(@"%@",dic);

//打印结果如下：
Printing description of dic:
{
allName = "<null>";
firstName = D;
age = 55;
givenName = W;
}
```

所以，看得出来，iOS里的空值还是有区别的，要好好区别，不然可能是会引起崩溃的！

这里来了解一下他们的概念：
![](https://upload-images.jianshu.io/upload_images/1241385-2465ce8ff7b3a439.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`NSNull` 在 Foundation 和其它框架中被广泛的使用，以解决如 `NSArray` 和 `NSDictionary` 之类的集合不能有 `nil` 值的缺陷。你可以将 `NSNull` 理解为有效的将 `NULL` 或者 `nil` 值封装[boxing](https://en.wikipedia.org/wiki/Object_type_(object-oriented_programming)#Boxing)，以达到在集合中使用它们的目的。
由此可见：**NSString中是可以添加为空对象的，即（nil）**
而**NSArray, NSDictionary则不行，为了防止崩溃，则使用NSNull来代替**


https://nshipster.cn/nil/
https://www.jianshu.com/p/6abd21fde286
