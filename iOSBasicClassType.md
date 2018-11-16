---
title: iOSBasicClassType
date: 2018-08-24 01:13:09
tags:
---


> iOS中有很多类，今天我们来讨论一些基本的类，以及他们的子类,相信`NSString`,`NSArray`,`NSDictionary`每一个iOS开发者都不会陌生，然后我们平时在开发的时候，很多时候也都会用到他们的一个别的子类，`NSMutableXX`，即`可变对象`。

### 说到可变对象和不可变对象，可能少不了另一个知识点copy和mutableCopy了###
1. copy和mutableCopy，深拷贝和浅拷贝
这里粗略的说下结论：

|原始对象|操作过程|拷贝完之后的对象|
|:-:|:-:|:-:|
|不可变对象|copy|浅拷贝，不会生成新对象，只是指针指向同一个对象|
|不可变对象|mutableCopy|深拷贝，会生成新的对象，新对象为可变对象|
|可变对象|copy|浅拷贝，会生成新的对象，新对象为不可变对象|
|可变对象|mutableCopy|深拷贝，会生成新对象，新对象也是可变对象|

2. NSString,NSArray,NSDictionary类簇
其实，这些基本类，是存在一个类簇的概念的，也就是说我们外部调用的看着是同一个接口，但是内部操作的时候，可能会调用其他的子类,
**这里罗列一下在研究的时候看到的具体子类:**
2.1 `NSString`，这里先声明两个变量，`NSString *str声明的不可变string`和`NSMutableString *strM声明的可变string`

|状态|类名|
|:-:|:-:|
|str|__NSCFConstantString|
|[str copy]|__NSCFConstantString|
|strM|__NSCFString|
|[strM copy]|NSTaggedPointerString|
|[strM mutableCopy]|__NSCFString|

2.2 `NSArray`,两个变量`NSArray *arr声明的不可变array`，和`NSMutableArray *arrM声明的可变array`

|状态|类名|
|:-:|:-:|
|arr|__NSArrayI|
|[arr copy]|__NSArrayI|
|arrM|__NSArrayM|
|[arrM copy]|__NSFrozenArrayM 或者 __NSArrayI|
|[arrM mutableCopy]|__NSArrayM|

2.3 `NSDictionary`,两个变量`NSDictionary *dict声明的不可变dictionary`和`NSMutableDictionary *dictM声明的可变dictionary`

|状态|类名|
|:-:|:-:|
|dict|__NSDictionaryI|
|[dict copy]|__NSDictionaryI|
|dictM|__NSDictionaryM|
|[dict copy]|__NSFrozenDictioinaryM|
|[dict mutableCopy]|__NSDictionaryM|


[给个demo](https://github.com/WooNoah/iOSBasicClassType)
