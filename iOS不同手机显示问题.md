---
title: iOS不同手机显示问题
date: 2018-12-10 15:58:54
tags:
---


今天遇到一个问题：设置一个view的高度，在不同手机下显示不一样：直接贴代码：
```
- (UIView *)line1 {
if (!_line1) {
_line1 = [[UIView alloc]init];
}
return _line1;
}

- (UIView *)line3 {
if (!_line3) {
_line3 = [[UIView alloc]init];
}
return _line3;
}


-(void)setLines{

/**
----------line1--------------
|line2
----------line3--------------
**/

[self addSubview:self.line1];
[self addSubview:self.line3];

[self.line1 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.top.right.equalTo(self);
make.height.equalTo(@(0.5));
}];

[self.line3 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.right.equalTo(self);
make.bottom.equalTo(self).offset(-0.25);
make.height.equalTo(self.line1);
}];
}

```
代码很简单，一目了然！
两条线按理说应该是一样的高度的。但是！！！
下面来看下结果！
```
//设置为0.33 6s：上下0.5px  max：上下都为0.33
//设置为0.5  6s：上下都为0.5px, max：上0.67，下0.33
//设置为0.44  6s：上下0.5px max：上下0.33
//设置为0.66  6s：上0.5px 下1px max：上下都为0.66px
//设置为0.66  6s：上下都为1px max：上下都为1px
```
我猜这个是跟屏幕的分辨率有关系，待以后研究下再来补充。
这里取折中方案，***先设置为0.33***


-------------------------------
### 2019年01月02日 更新
> 根据参考文章中的知识点总结如下：

##### 知识点1、iPhone中屏幕分辨率
目前分为三种，@1x(iPhone 4之前的设备), @2x(iPhone 4 ~ iPhone XR中非Plus设备), @3x(iPhone X, XS XS Max, 和 Plus设备).
那么，可以得出结论，不同设备上，显示宽度为1个点占用的像素是不同的
（@1x上为1px，@2x上为2px, @3x上为3px）
也就是说，1px的宽度在不同scale的设备上是不同的（@1x的宽度为1个点，@2x的为0.5个点，@3x的为0.33个点）
因此就可以根据设备来分别设置1px的宽度
`#define SINGLE_LINE_WIDTH    (1 / [UIScreen mainScreen].scale)`

##### 知识点2、iPhone的渲染机制
为了获得良好的视觉效果，绘图系统通常都会采用一个叫“antialiasing(反锯齿)”的技术，iOS也不例外,
显示屏幕有很多小的显示单元组成，可以接单的理解为一个单元就代表一个像素。如果要画一条黑线，条线刚好落在了一列或者一行显示显示单元之内，将会渲染出标准的一个像素的黑线。
`但如果线落在了两个行或列的中间时，那么会得到一条“失真”的线，其实是两个像素宽的灰线。`如下图所示:
![](https://upload-images.jianshu.io/upload_images/1241385-37a3749a6c6e3375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
官方解释如上:
规定：奇数像素宽度的线在渲染的时候将会表现为柔和的宽度扩展到向上的整数宽度的线，除非你手动的调整线的位置，使线刚好落在一行或列的显示单元内。如何对齐呢？**我们可以给渲染位置做下偏移！**
如果当前的线是“失真”的情况：使之偏移宽度的一半，即可渲染在正确的标准位置（通俗来讲就是没有渲染在两个点中间）
同样的提供一个宏来做计算偏移量
`#define SINGLE_LINE_ADJUST_OFFSET   ((1 / [UIScreen mainScreen].scale) / 2)`
这么一来，就可以绘制标准的线了

***当然，我们实际开发中，判断当前位置是否是正好跨在两个点之间渲染的，是一个比较难的问题，我们只需要保证我们需要的线是准确的1px就可以了（即只使用上面的宏即可）***



#### 参考：
https://blog.csdn.net/hherima/article/details/46679157
