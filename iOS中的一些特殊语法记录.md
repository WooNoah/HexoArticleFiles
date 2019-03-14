---
title: iOS中的一些特殊语法记录
date: 2019-03-14 17:42:09
tags:
---


> 本篇文章来解释一些iOS代码中的特殊语法、写法，语法糖
#### 1. FBKVOKeyPath
```
#define FBKVOKeyPath(KEYPATH) \
@(((void)(NO && ((void)KEYPATH, NO)), \
({ const char *fbkvokeypath = strchr(#KEYPATH, '.'); NSCAssert(fbkvokeypath, @"Provided key path is invalid."); fbkvokeypath + 1; })))
```
摘录自[KVOController](https://github.com/facebook/KVOController)中`FBKVOController.h`中
官方解释如下：
```
This macro ensures that key path exists at compile time.
Given a real receiver with a key path as you would call it, it verifies at compile time that the key path exists, without calling it.

For example:

FBKVOKeyPath(string.length) => @"length"

Or even the complex case:

FBKVOKeyPath(string.lowercaseString.length) => @"lowercaseString.length".
```
我们可以使用该宏定义获取`string.property`中的`property`属性，而且，默认转换成`NSString`类型
```
1 校验传入的KeyPath是否有编译错误
((void)(NO && ((void)KEYPATH, NO))
// NO && ... 是为了运行时直接返回NO减少操作， 因为有(void)KEYPATH的存在，所以编译时校验了object.property

// 2 将传入的object.property转换为"property"
{ const char *fbkvokeypath = strchr(#KEYPATH, '.'); NSCAssert(fbkvokeypath, @"Provided key path is invalid."); fbkvokeypath + 1; }
// 2.1 #KEYPATH将object.property转为字符串"object.property"
// 2.2 strchr截取".property"
// 2.3 NSCAssert保证fbkvokeypath有效
// 2.4 ".property"+1="property"

// 3 使用@()语法糖将char *转换为NSString类型
@(((void)NO, "property"))
// 因为','操作符是返回后面的值，即string = (@"a", @"b");string的值为@"b"

2.1 附：c语言中，##表示把两个宏参数贴合在一起，而单个#的功能是将其后面的宏参数进行字符串化操作
2.2 strchr()函数定义：
char *strchr( const char *str, int ch ); 
str: 传入的字符串指针
ch：要搜索的字符
return：返回特定字符**第一次**在字符串中的出现的指针，未找到时返回空指针
2.3 NSCAssert和NSAssert区别，见2
2.4 p+1 表示地址，指针p所指向的内存地址的下一个内存地址。

```


#### 2. NSAssert和NSCAssert区别
在苹果的SDK中可以看到这两个都是定义的宏
```
#define NSAssert(condition, desc, ...)  \  
do {                \  
__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS \  
if (!(condition)) {     \  
[[NSAssertionHandler currentHandler] handleFailureInMethod:_cmd \  
object:self file:[NSString stringWithUTF8String:__FILE__] \  
lineNumber:__LINE__ description:(desc), ##__VA_ARGS__]; \  
}               \  
__PRAGMA_POP_NO_EXTRA_ARG_WARNINGS \  
} while(0)  
#endif 
```
```
#define NSCAssert(condition, desc, ...) \  
do {                \  
__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS \  
if (!(condition)) {     \  
[[NSAssertionHandler currentHandler] handleFailureInFunction:[NSString stringWithUTF8String:__PRETTY_FUNCTION__] \  
file:[NSString stringWithUTF8String:__FILE__] \  
lineNumber:__LINE__ description:(desc), ##__VA_ARGS__]; \  
}               \  
__PRAGMA_POP_NO_EXTRA_ARG_WARNINGS \  
} while(0)  
#endif 
```
都是使用断言来保证条件的正确性
但是要注意一点：
`NSAssert`在实现中，强引用了`self`当前对象，这里有可能会引起循环引用

#### 3. iOS枚举中直接设置值为字母
一般情况，我们写枚举like this：
```
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {  
UIViewAnimationTransitionNone,//默认从0开始  
UIViewAnimationTransitionFlipFromLeft,  
UIViewAnimationTransitionFlipFromRight,  
UIViewAnimationTransitionCurlUp,  
UIViewAnimationTransitionCurlDown,  
};  

typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {  
UIViewAutoresizingNone                 = 0,  
UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,  
UIViewAutoresizingFlexibleWidth        = 1 << 1,  
UIViewAutoresizingFlexibleRightMargin  = 1 << 2,  
UIViewAutoresizingFlexibleTopMargin    = 1 << 3,  
UIViewAutoresizingFlexibleHeight       = 1 << 4,  
UIViewAutoresizingFlexibleBottomMargin = 1 << 5  
}; 
```
但是，还会有这样的写法
比如：
```
typedef NS_ENUM(NSInteger, Test)
{
TestA = 0,
TestB = 1,
TestC = 4,
TestD = G,
};
```
这种情况，就是**使用了"G"的**`ASCII`，赋值给了`TestD`
#### 4. [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)中的**RAC宏定义**
先贴个连接：https://www.jianshu.com/p/7086e090069d


[参考资料1：KVOController详解](https://www.jianshu.com/p/8deccb9c8398)
[参考资料2：NSAssert,NSCassert](https://blog.csdn.net/likendsl/article/details/36631401)
[参考资料3：断言(NSAssert)的使用](https://www.jianshu.com/p/6e444981ab45)
[参考资料4：iOS之枚举用法](https://www.jianshu.com/p/740233c2d638)
[参考资料5：Hello, 宏定义魔法世界](https://www.jianshu.com/p/4a1531bac39f)1
