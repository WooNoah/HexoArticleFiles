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


#### 2019年10月17日
影响iOS viewController dealloc的方法小结
1. VC中的block强引用了self
```
eg: AFN的请求
在回调中引用了self
```
2. dispatch_after中强引用self
```
如下面的代码中这样，不引用self，则会直接销毁
- (void)normalDispatchAfter {
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"5s later");
    });
}

2019-10-18 16:49:56.568192+0800 GCDStrongReferenceDemo[15628:3535241] ~~~~~~~~~~~~~~~~~~~~SecViewController - dealloc
2019-10-18 16:50:00.070705+0800 GCDStrongReferenceDemo[15628:3535241] 5s later
但是，after中的代码由于是添加到Runloop中了，所以5s之后，仍然会调用


```
同样的，以前怀疑的dispatch_group/notify等方法也不会强引用self
```
    __weak typeof (self) weakSelf = self;
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_queue_create("com.demo.GCDStrongReference", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_group_async(group, queue, ^{
        dispatch_group_enter(group);
//        NSLog(@"~~~~~~~~~~~~~~~~~~~~1111111111 enter");
        [weakSelf logWithInfo:@"11111111  enter"];
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
//            NSLog(@"~~~~~~~~~~~~~~~~~~~~1111111111 leave");
            [weakSelf logWithInfo:@"11111111  leave"];
            dispatch_group_leave(group);
        });
    });
    
    dispatch_group_async(group, queue, ^{
        dispatch_group_enter(group);
//        NSLog(@"~~~~~~~~~~~~~~~~~~~~222222222222");
        [weakSelf logWithInfo:@"22222222222222"];
        dispatch_group_leave(group);
    });
    
    dispatch_group_notify(group, queue, ^{
//        NSLog(@"~~~~~~~~~~~~~~~~~~~~notify");
        [weakSelf logWithInfo:@"notify"];
    });
    
    打印结果也是会直接dealloc
    2019-10-18 16:58:27.441916+0800 GCDStrongReferenceDemo[15858:3547133] ~~~~~~~~~~~~~~~~~~~~SecViewController - 11111111  enter
    2019-10-18 16:58:27.441926+0800 GCDStrongReferenceDemo[15858:3547358] ~~~~~~~~~~~~~~~~~~~~SecViewController - 22222222222222
    2019-10-18 16:58:28.901496+0800 GCDStrongReferenceDemo[15858:3546983] ~~~~~~~~~~~~~~~~~~~~SecViewController - dealloc
```
> 总结：如果在block中改为弱引用，则可以避免 __weak typeof(self) weakSelf = self;

#### 2019年10月30日 setContentHuggingPriority为抗拉伸优先级
以前只是按照字面意思，以为是`拉伸优先级`，结果是`抗拉伸优先级`:
即： 优先级越高，越不容易被拉伸，优先级越低，则越容易被拉伸
同理，`setContentCompressionResistancePriority`为抗压缩优先级
即：优先级越高，越不容易被压缩，优先级越低，越容易被压缩。
默认情况下: HuggingPriority == 250,  CompressionResistancePriority == 750
