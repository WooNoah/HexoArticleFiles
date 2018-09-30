---
title: iOS数组enumerateXX方法探索
date: 2018-09-30 14:09:40
tags:
---

> 今天看到代码，突然突发奇想，想要探索一个问题：
NSArray的`enumerateObjectsUsingBlock `方法到底是同步的还是异步的，如果我们想要在Block内部修改外部的值，是否要使用`__block`来修饰

这里写了一个简单的demo来验证：
```
- (void)viewDidLoad {
[super viewDidLoad];

self.arr = @[@"1",@"2",@"3",@"4",@"5",@"6",@"7"];
self.count = 0;

[self testMethod];
}

- (void)testMethod {
NSMutableArray *arrM = [NSMutableArray arrayWithCapacity:0];
[self.arr enumerateObjectsUsingBlock:^(NSString *str, NSUInteger idx, BOOL * _Nonnull stop) {
NSLog(@"内部操作 ==>%zd",self.count++);
[arrM addObject:str];
}];

NSLog(@"外部结束：数组=%@，==>%zd",arrM,self.count++);
}
```
LOG如下：
```
2018-09-30 14:02:14.144150+0800 testEnumerate[10209:289132] 内部操作 ==>0
2018-09-30 14:02:14.144277+0800 testEnumerate[10209:289132] 内部操作 ==>1
2018-09-30 14:02:14.144363+0800 testEnumerate[10209:289132] 内部操作 ==>2
2018-09-30 14:02:14.144442+0800 testEnumerate[10209:289132] 内部操作 ==>3
2018-09-30 14:02:14.144517+0800 testEnumerate[10209:289132] 内部操作 ==>4
2018-09-30 14:02:14.144592+0800 testEnumerate[10209:289132] 内部操作 ==>5
2018-09-30 14:02:14.144666+0800 testEnumerate[10209:289132] 内部操作 ==>6
2018-09-30 14:02:14.144783+0800 testEnumerate[10209:289132] 外部结束：数组=(
1,
2,
3,
4,
5,
6,
7
)，==>7
```
#### 可以得出结论：
`enumerateObjectsUsingBlock`方法是同步的，
而且，想要在block内部修改外部的对象，也不需要`__block`修饰
