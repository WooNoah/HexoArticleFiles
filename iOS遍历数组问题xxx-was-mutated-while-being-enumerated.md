---
title: iOS遍历数组问题xxx was mutated while being enumerated
date: 2018-11-20 11:18:27
tags:
---


#### 今天遍历数组删除数据的时候，遇到一个问题：
> <__NSArrayM: 0xb550c30> was mutated while being enumerated

这里记录一下。


源代码是这么写的
```
- (NSMutableArray *)removeIncludeExposureDataWithinArray:(NSMutableArray *)arr {
NSMutableArray *resArr = arr.mutableCopy;
for (TraderNewsModel *m in resArr) {
if ([m.categoryName isEqualToString:@"曝光"]) {
[resArr removeObject:m];
}
}
return resArr;
}
```
然后报错信息就是如上，大概意思就是数组在枚举遍历的时候，发生变化了。。。

查阅资料之后发现:
`for in`这种遍历方式是不允许的，而`for (int i; i < count; i++)`和`-enumeratorObjectsUsingBlock`都是允许的。

#### 解决方法：
1. 确保数组在遍历的时候**不被修改**，也就是创建一个tempArrM,来保存修改好的数据即可
2. 使用`for (int i; i < count; i++)`或者`enumeratorObjectXXX`方法来遍历
一般的for循环遍历就不说了，这里写一下使用block修改的方式：
```
/**
* 删除包含“曝光”的字段
*/
- (NSMutableArray *)removeIncludeExposureDataWithinArray:(NSMutableArray *)arr {
NSMutableArray *resArr = arr.mutableCopy;
[resArr enumerateObjectsUsingBlock:^(TraderNewsModel *model, NSUInteger idx, BOOL * _Nonnull stop) {
if ([model.categoryName isEqualToString:@"曝光"]) {
*stop = YES;
[resArr removeObject:model];
}
}];
return resArr;
}
```

[参考资料](https://blog.csdn.net/piaodang1234/article/details/11902541)
