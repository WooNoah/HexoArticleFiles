---
title: iOS对象等同性
date: 2017-03-08 11:07:58
tags:
---

#### 前言

最近看了一道题  
`NSString *s1 = @"Hello world"; NSString *s2 = @"Hello world";请问 s1 == s2的返回值是YES还是NO？`,
相信很多童鞋的答案都是NO，可能大家认为s1、s2两个对象的地址不同，但是事实真的如此么？

为此特意写了一个demo来印证下：
<!--more-->

```

#importint main(int argc, const char * argv[]) {

@autoreleasepool {

NSString *str1 = @"abc";

NSString *str2 = [NSString stringWithFormat:@"%@",@"abc"];

NSString *str3 = [[NSString alloc]initWithString:str1];

NSString *str4 = [NSString stringWithString:str1];

NSString *str8 = [[NSString alloc] initWithString:@"abc"];

NSString *str9 = [NSString stringWithString:@"abc"];

NSString *str5 = [str1 copy];

NSString *str6 = [str1 mutableCopy];

NSString *str7 = [NSString stringWithFormat:@"%@",@"abc"];

NSLog(@"str1: %p",str1);

NSLog(@"str2: %p",str2);

NSLog(@"str3: %p",str3);

NSLog(@"str4: %p",str4);

NSLog(@"str5: %p",str5);

NSLog(@"str6: %p",str6);

NSLog(@"str7: %p",str7);

NSLog(@"str8: %p",str8);

NSLog(@"str9: %p",str9);

NSLog(@"===============");

NSLog(@"str1 == str2  \t\t%@",(str1 == str2) ? @"YES" : @"NO");

NSLog(@"str1 Equal str2  \t%@",[str1 isEqualToString:str2] ? @"YES" : @"NO");

NSLog(@"str1 == str3  \t\t%@",(str1 == str3) ? @"YES" : @"NO");

NSLog(@"str1 == str4  \t\t%@",(str1 == str4) ? @"YES" : @"NO");

NSLog(@"str3 == str4  \t\t%@",(str4 == str3) ? @"YES" : @"NO");

NSLog(@"str2 == str6  \t\t%@",(str2 == str6) ? @"YES" : @"NO");

NSLog(@"str2 Equal str6  \t%@",[str2 isEqualToString:str6] ? @"YES" : @"NO");

NSLog(@"str2 == str7  \t\t%@",(str7 == str2) ? @"YES" : @"NO");

NSLog(@"%d",str2 == str7);

}

return 0;

}

//LOGCAT如下

2017-02-28 13:27:45.326 testEqual[6456:440040] str1: 0x100001048

2017-02-28 13:27:46.022 testEqual[6456:440040] str2: 0x63626135

2017-02-28 13:27:46.765 testEqual[6456:440040] str3: 0x100001048

2017-02-28 13:27:47.284 testEqual[6456:440040] str4: 0x100001048

2017-02-28 13:27:47.732 testEqual[6456:440040] str5: 0x100001048

2017-02-28 13:27:48.835 testEqual[6456:440040] str6: 0x100102370

2017-02-28 13:27:49.690 testEqual[6456:440040] str7: 0x63626135

2017-02-28 13:28:05.204 testEqual[6456:440040] str8: 0x100001048

2017-02-28 13:28:16.683 testEqual[6456:440040] str9: 0x100001048

2017-02-28 13:29:17.041 testEqual[6456:440040] ===============

2017-02-28 13:29:18.178 testEqual[6456:440040] str1 == str2  NO

2017-02-28 13:29:21.152 testEqual[6456:440040] str1 Equal str2  YES

2017-02-28 13:31:01.274 testEqual[6456:440040] str1 == str3  YES

2017-02-28 13:31:01.274 testEqual[6456:440040] str1 == str4  YES

2017-02-28 13:31:02.918 testEqual[6456:440040] str3 == str4  YES

2017-02-28 13:31:02.918 testEqual[6456:440040] str2 == str6  NO

2017-02-28 13:31:02.918 testEqual[6456:440040] str2 Equal str6  YES

2017-02-28 13:31:02.918 testEqual[6456:440040] str2 == str7  YES

2017-02-28 13:31:02.919 testEqual[6456:440040] 1

```

#### 由此可以得出结论

*Tips:*

1. str1 是 直接使用*字面量语法*赋值的变量。所以，跟str8,str9是一样的，编译器都会执行同样的代码来生成对象。

2. str3和str4,则都是根据一个**已知的字符串变量**来生成对象。*其实就是比上面Tip1多了一个步骤*、所以，`str3和str8，str4和str9地址是相同的`

3. str2和str7`[NSString stringWithFormat:@"%@",@"abc"]`都是格式化生成，地址也是相同的

扩展：`NSString *str10 = [NSString stringWithFormat:@"%@",str1];`同理，如果新加一个str10，那它的地址应该跟str2,str7也是一样的

4. str5`[str1 copy]`和str6`[str1 mutableCopy]`

这两个涉及到深浅拷贝，

str5是str1的浅拷贝，指针都指向相同的地址

str6是str1的深拷贝，*等于重新创建了一个新的对象*所以地址是不同的

#### 因此

如果上面的问题换种问法，比如：

`NSString *s1 = @"hello";

NSString *s2 = [[NSString alloc] initWithString:s1];

请问s1==s2的返回值？`

或者

`NSString *s1 = [NSString stringWithFormat:@"hello"];

NSString *s2 = [NSString stringWithFormat:@"hello"];

请问s1==s2的返回值？`

答案也都是**YES**!

#### PS：

单说NSString，系统为我们提供了一个`isEqualToString:`方法，所以一般情况下来说，我们是不会使用`==`来判断两个NSString对象是否相等的。

`==`和`isEqualToString:`有什么区别呢？

由上面的例子也可以看到，

`str1 == str2  NO

str1 Equal str2  YES`

`isEqualToString:`应该是只比较了两个对象的值，不比较地址

而`==`则会比较两个对象的地址和值是否都相等

毕竟判断两个对象是否相等的条件是：

`当且仅当其“指针值（pointer value）”完全相等时，这两个对象才相等`.


--------------------
#### 2018年11月26日延伸：
#### 一、NSString对象`copy`和`strong`修饰词的区别
```
//先声明两个变量
@property (nonatomic, strong) NSString *strongStr;
@property (nonatomic, copy) NSString *cpyStr;
```
然后这里分两种情况
1. 不可变str的情况
```
- (void)inmutableTest {
NSString *string = [NSString stringWithFormat:@"abc"];
self.strongStr = string;
self.cpyStr = string;
NSLog(@"string:%@, 对象地址:%p, 指针地址：%p", string,string,&string);
NSLog(@"strongStr:%@, 对象地址:%p, 指针地址：%p,", self.strongStr,_strongStr,&_strongStr);
NSLog(@"cpyStr:%@, 对象地址:%p, 指针地址：%p", self.cpyStr,_cpyStr,&_cpyStr);

//改变string的值
string = @"123";

NSLog(@"string:%@, 对象地址:%p, 指针地址：%p", string,string,&string);
NSLog(@"strongStr:%@, 对象地址:%p, 指针地址：%p,", self.strongStr,_strongStr,&_strongStr);
NSLog(@"cpyStr:%@, 对象地址:%p, 指针地址：%p", self.cpyStr,_cpyStr,&_cpyStr);
}
//结果：
//string:abc, 对象地址:0xe9f9b161ce4bbd88, 指针地址：0x7ffee1306e48
//strongStr:abc, 对象地址:0xe9f9b161ce4bbd88, 指针地址：0x7fcc7b42a660,
//cpyStr:abc, 对象地址:0xe9f9b161ce4bbd88, 指针地址：0x7fcc7b42a668
//string:123, 对象地址:0x10e8fa1a0, 指针地址：0x7ffee1306e48
//strongStr:abc, 对象地址:0xe9f9b161ce4bbd88, 指针地址：0x7fcc7b42a660,
//cpyStr:abc, 对象地址:0xe9f9b161ce4bbd88, 指针地址：0x7fcc7b42a668

```
可以看到，在string为NSString(不可变)的情况下，修改string的值意味着***改变指针的指向***
此时，***`copy`和`strong`修饰的新字符串值都不会变！因为他们的指针指向的地址都还是以前的地址！！***

2. 可变Str的情况
```
- (void)mutableTest {
NSMutableString *string= [[NSMutableString alloc]initWithString:@"abc"];
self.strongStr = string;
self.cpyStr = string;
NSLog(@"string:%@, 对象地址:%p, 指针地址：%p", string,string,&string);
NSLog(@"strongStr:%@, 对象地址:%p, 指针地址：%p,", self.strongStr,_strongStr,&_strongStr);
NSLog(@"cpyStr:%@, 对象地址:%p, 指针地址：%p", self.cpyStr,_cpyStr,&_cpyStr);

//改变string的值
[string appendFormat:@"%@",@"123"];

NSLog(@"string:%@, 对象地址:%p, 指针地址：%p", string,string,&string);
NSLog(@"strongStr:%@, 对象地址:%p, 指针地址：%p,", self.strongStr,_strongStr,&_strongStr);
NSLog(@"cpyStr:%@, 对象地址:%p, 指针地址：%p", self.cpyStr,_cpyStr,&_cpyStr);
}
//结果：
//string:abc, 对象地址:0x60000094ffc0, 指针地址：0x7ffee2898758
//strongStr:abc, 对象地址:0x60000094ffc0, 指针地址：0x7f998c70d520,
//cpyStr:abc, 对象地址:0x9b661481bbd71a70, 指针地址：0x7f998c70d528
//string:abc123, 对象地址:0x60000094ffc0, 指针地址：0x7ffee2898758
//strongStr:abc123, 对象地址:0x60000094ffc0, 指针地址：0x7f998c70d520,
//cpyStr:abc, 对象地址:0x9b661481bbd71a70, 指针地址：0x7f998c70d528
```
此结果就能明显看到区别了，
**`strong`修饰的字符串，会跟随源字符串的变化而变化！`copy`修饰的并不会！！**

***这也可以解释：为什么NSString的变量要用`copy`来修饰***
***生成NSString对象的时候，就是不想在后续的过程中改变他，如果使用`strong`来修饰，则途中修改了原值，strong修饰的变量也会跟着变化，这显然有悖于初衷***

####  二、有关原文中stringWithFormat生成对象地址和别的方法生成对象地址不同的问题：
> 由上文可以看到，`stringWithFormat`和`initWithFormat`两方法生成的NSString对象地址是相同的。其他的任何方法生成的对象都不同！因为`stringWithFormat`方法内部也是调用了`initWithFormat`方法。

![](https://upload-images.jianshu.io/upload_images/1241385-0d0638d2c13f1a32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原因是这样：
`stringWithFormat`方法会创建一个新的对象，开辟新的内存空间，遵循引用计数原则，创建释放等等。
`stringWithString`等方法，其实并没有创建一个新的对象，苹果对于这种字面量字符串，一般都是存储在`常量区`，`stringWithFormat`等方法只是添加一个该常量的引用而已，**并不会开辟新的内存空间**

这也就能解释为什么这两种方式创建的字符串，内存地址不同了
