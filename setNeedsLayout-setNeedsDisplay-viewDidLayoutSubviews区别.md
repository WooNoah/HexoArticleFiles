---
title: setNeedsLayout setNeedsDisplay viewDidLayoutSubviews区别
date: 2019-04-02 15:39:48
tags:
---

> 对当前了解的知识点做下总结

#### iOS Layout机制相关方法

1. 修改view的frame
```
- (void)sizeToFit
- (CGSize)sizeThatFits:(CGSize)size
```
------------
2. 告诉系统来刷新布局
```
- (void)layoutIfNeeded
- (void)setNeedsLayout
//布局控件的方法
- (void)layoutSubviews
```
------------
3. 告诉系统来绘制
```
- (void)setNeedsDisplay
//绘制方法
- (void)drawRect
```
------------
<!--more-->
#### layoutSubviews在以下情况下会被调用：

1、init初始化不会触发layoutSubviews

   但是是用initWithFrame 进行初始化时，当rect的值不为CGRectZero时,也会触发
   
   2、addSubview会触发layoutSubviews
   
   3、设置view的Frame会触发layoutSubviews，当然前提是frame的值设置前后发生了变化
   
   4、滚动一个UIScrollView会触发layoutSubviews
   
   5、旋转Screen会触发父UIView上的layoutSubviews事件
   
   6、改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件
   
   在苹果的官方文档中强调:
   ```
   You should override this method only if the autoresizing behaviors 
   of the subviews do not offer the behavior you want.
   ```
   layoutSubviews, 当我们在某个类的内部调整子视图位置时，需要调用。
   
   反过来的意思就是说：如果你想要在外部设置subviews的位置，就不要重写。
   
   刷新子对象布局
   
   -layoutSubviews方法：这个方法，默认没有做任何事情，需要子类进行重写
   -setNeedsLayout方法： 标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，但layoutSubviews一定会被调用
   -layoutIfNeeded方法：如果，有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）
   
   如果要立即刷新，要先调用[view setNeedsLayout]，把标记设为需要布局，然后马上调用[view layoutIfNeeded]，实现布局
   
   ----------------
   ##### 注意！
   > setNeedsLayout官方文档中这么解释：
   
   ![](https://upload-images.jianshu.io/upload_images/1241385-242ca1cabd6d7654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   **可以看到划红线部分，此方法是做一个“此视图需要刷新”的标记，然后在下一个刷新周期，刷新布局，如果给一个view1添加上标记`[view1 setNeedsLayout]`,则view1的子类也会调用`layoutSubviews`方法**
   > 所以，如果我们想要刷新一个视图view1的布局，则要给**他的父类发送消息`- setNeedsLayout`**
   
   如果想要在下一个更新周期到来之前刷新，则可以调用`- layoutIfNeeded`**立刻刷新**
   PS, 在视图第一次显示之前，标记总是“需要刷新”的，可以直接调用[view layoutIfNeeded]
   
   ------------------
   #### 重绘
   
   -drawRect:(CGRect)rect方法：重写此方法，执行重绘任务
   -setNeedsDisplay方法：标记为需要重绘，异步调用drawRect
   -setNeedsDisplayInRect:(CGRect)invalidRect方法：标记为需要局部重绘
   
   sizeToFit会自动调用sizeThatFits方法；
   
   sizeToFit不应该在子类中被重写，应该重写sizeThatFits
   
   sizeThatFits传入的参数是receiver当前的size，返回一个适合的size
   
   sizeToFit可以被手动直接调用
   
   sizeToFit和sizeThatFits方法都没有递归，对subviews也不负责，只负责自己
   
   ------------
   
   layoutSubviews对subviews重新布局
   
   layoutSubviews方法调用先于drawRect
   
   setNeedsLayout在receiver标上一个需要被重新布局的标记，在系统runloop的下一个周期自动调用layoutSubviews
   
   layoutIfNeeded方法如其名，UIKit会判断该receiver是否需要layout.根据Apple官方文档,layoutIfNeeded方法应该是这样的
   
   layoutIfNeeded遍历的不是superview链，应该是subviews链
   
   drawRect是对receiver的重绘，能获得context
   
   setNeedDisplay在receiver标上一个需要被重新绘图的标记，在下一个draw周期自动重绘，iphone device的刷新频率是60hz，也就是1/60秒后重绘 
   
   ![](https://upload-images.jianshu.io/upload_images/1241385-5980512176f7ef8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   
   
   [参考link](http://www.jianshu.com/p/eb2c4bb4e3f1)
   
   --------------
   #### 至于viewDidLayoutSubviews
   先来看看苹果的官方文档：
   ![](https://upload-images.jianshu.io/upload_images/1241385-d93ad7989d2234fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   可以看出来，此方式是UIViewController的对象方法,
   总结一句：
   `当viewController的View布局完成全部的子view的时候会被调用，然后就是这个方法在controller的的子视图的position和size被改变的时候被调用。`
   
   [https://www.appcoda.com.tw/view-controller-lifecycle/](https://www.appcoda.com.tw/view-controller-lifecycle/)
   [https://stackoverflow.com/questions/40737164/whats-exactly-viewdidlayoutsubviews](https://stackoverflow.com/questions/40737164/whats-exactly-viewdidlayoutsubviews)
