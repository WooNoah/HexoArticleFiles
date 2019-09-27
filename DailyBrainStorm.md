---
title: DailyBrainStorm
date: 2019-08-27 12:59:43
tags:
---

>  写这篇的目的：好多时候没有更新文章，其实并不是没有在做研究，只不过有些小的知识点，并不值得洋洋洒洒的来一篇，故而开了这个tag，每当自己有什么小的研究和小的idea，能够记录下来


#### 2019年08月27日
iOS tableview的headerView和footerView,
在创建的时候设置的frame，系统会根据宽高来添加约束，因此无法通过添加子视图与父视图之间的约束，来做到“子视图撑大父视图”这样。想要修改tableview的header、footer视图，可以在子视图布局完成之后添加回调，重新设置即可。
`self.tableview.tableHeaderView = self.headerView;`
`self.tableview.tableFooterView = self.footerView;`

#### 2019年09月11日
iOS label 默认设置lineBreakMode为省略号在右边
但是如果使用了NSParagraphStyle,则label设置的lineBreakMode就会失效，使用NSMutableParagraphStyle的lineBreakMode代替即可

#### 2019年09月19日
iOS的collectionView, 如果collectionview的frame不足以展示两个cell，则此时cell会居中显示
```
这里以左右滑动为例：collectionview的height为vHeight, cell的height为cHeight。默认collectionview的cell的item间距和item行间距都为0
如果cHeight < vHeight < 2*cHeight, 此时
eg:  vHeight = 100; cHeight = 70;
此时cell居中展示， 上下间距15

如果 2*cHeight < vHeight < 3*cHeight
此时，view能只能展示两个cell，则两个cell分别位于view的top和bottom，剩下的距离为两个cell之间的margin

如果 3*cHeight < vHeight < 4*cHeight
此时，view能展示三个cell，则第一个，第三个分别位于view的顶部和底部，然后中间一个居中展示，(vHeight - 3*cHeight)/2为三个cell之间的间距

如果 4*cHeight < vHeight < 5*cHeight
此时，view能展示四个cell，则第一个，第四个分别位于view的顶部和底部，然后剩下的两个在中间展示，(vHeight - 4*cHeight)/3为三个cell之间的间距

剩下的以此类推
```

#### 2019年09月27日
iOS UIView提供的方法中，有转换坐标系的方法，比如：
```
- (CGPoint)convertPoint:(CGPoint)point toView:(nullable UIView *)view;
- (CGPoint)convertPoint:(CGPoint)point fromView:(nullable UIView *)view;
- (CGRect)convertRect:(CGRect)rect toView:(nullable UIView *)view;
- (CGRect)convertRect:(CGRect)rect fromView:(nullable UIView *)view;
```
项目中用到了`convertRect: toView:`方法
总结一下各个参数的含义。
1. 调用者：转换前的view的父类调用
2. rect：转换前的view的frame
3. toView：转换后的坐标系view
