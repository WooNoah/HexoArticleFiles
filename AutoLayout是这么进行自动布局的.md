---
title: AutoLayout是这么进行自动布局的
date: 2019-05-07 19:19:15
tags:
---


AutoLayout底层实现为Cassowary算法

Cassowary是以高效的界面线性方程求解算法被提出来的，他解决的是界面的线性规划问题，而线性规划问题的解法是simplex算法

iOS12之前 AutoLayout存在的性能问题在
[WWDC 2018 202 session](https://developer.apple.com/videos/play/wwdc2018/220/)中介绍

------------------
#### Auto Layout 的生命周期

Auto Layout 不只有布局算法 Cassowary，还包含了布局在运行时的生命周期等一整套布局引擎系统，用来统一管理布局的创建、更新和销毁。了解 Auto Layout 的生命周期，是理解它的性能相关话题的基础。这样，在遇到问题，特别是性能问题时，我们才能从根儿上找到原因，从而避免或改进类似的问题。

这一整套布局引擎系统叫作 Layout Engine ，是 Auto Layout 的核心，主导着整个界面布局。

每个视图在得到自己的布局之前，Layout Engine 会将视图、约束、优先级、固定大小通过计算转换成最终的大小和位置。在 Layout Engine 里，每当约束发生变化，就会触发 Deffered Layout Pass，完成后进入监听约束变化的状态。当再次监听到约束变化，即进入下一轮循环中。整个过程如下图所示：

![Layout Engine 界面布局过程](https://upload-images.jianshu.io/upload_images/1241385-650ea7eff36ddbc6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中， Constraints Change 表示的就是约束变化，添加、删除视图时会触发约束变化。Activating 或 Deactivating，设置 Constant 或 Priority 时也会触发约束变化。Layout Engine 在碰到约束变化后会重新计算布局，获取到布局后调用 superview.setNeedLayout()，然后进入 Deferred Layout Pass。

Deferred Layout Pass 的主要作用是做容错处理。如果有些视图在更新约束时没有确定或缺失布局声明的话，会先在这里做容错处理。

接下来，Layout Engine 会从上到下调用 layoutSubviews() ，通过 Cassowary 算法计算各个子视图的位置，算出来后将子视图的 frame 从 Layout Engine 里拷贝出来。

在这之后的处理，就和手写布局的绘制、渲染过程一样了。

------------------

#### 我们使用AutoLayout的方式有以下两种：

1. 使用iOS提供的基于AutoLayout封装的`UIStackView`
其实他更加类似于前端的`flexbox`布局思路，但是**他有一个要求，就是必须是iOS9之后才可以使用**


2. 另外一个，就是我们直接基于AutoLayout封装成库
使用DSL(Domain Specific Language)语言来处理页面布局的方式
https://github.com/WooNoah/STMAssembleView (forked from [ming1016/STMAssembleView](https://github.com/ming1016/STMAssembleView))

#### 注意：
1. 使用AutoLayout, 要多使用UIView的`Compression Resistance Priority`和`Hugging Priority`，
可以参考[这个demo](https://github.com/WooNoah/showAutoLayout)

2. ##### AutoLayout的一些基本概念

*   利用约束来控制视图的大小和位置，系统会在运行时通过设置的约束计算得到frame再绘制屏幕
*   两个属性Content Compression Resistance（排挤，值越高越固定）和Content Hugging（拥抱）,Masonry代码如下

```source-objc
//content hugging 为1000
[view setContentHuggingPriority:UILayoutPriorityRequired
forAxis:UILayoutConstraintAxisHorizontal];

//content compression 为250
[view setContentCompressionResistancePriority:UILayoutPriorityDefaultLow
forAxis:UILayoutConstraintAxisHorizontal];
```

*   multipler属性表示约束值为约束对象的百分比，在Masonry里有对应的multipliedBy函数

```source-objc
//宽度为superView宽度的20%
make.width.equalTo(superView.mas_width).multipliedBy(0.2);
```

*   AutoLayout下UILabel设置多行计算需要设置preferredMaxLayoutWidth

```source-objc
label.preferredMaxWidth = [UIScreen mainScreen].bounds.size.width - margin - padding;
```

*   preferredMaxLayoutWidth用来制定最大的宽，一般用在多行的UILabel中
*   systemLayoutSizeFittingSize方法能够获得view的高度
*   iOS7有两个很有用的属性，topLayoutGuide和bottomLayoutGuide，这个两个主要是方便获取UINavigationController和UITabBarController的头部视图区域和底部视图区域。

```source-objc
//Masonry直接支持这个属性
make.top.equalTo(self.mas_topLayoutGuide);
```

##### 3.[](https://github.com/ming1016/study/wiki/Masonry#autolayout%E5%85%B3%E4%BA%8E%E6%9B%B4%E6%96%B0%E7%9A%84%E5%87%A0%E4%B8%AA%E6%96%B9%E6%B3%95%E7%9A%84%E5%8C%BA%E5%88%AB)AutoLayout关于更新的几个方法的区别

*   setNeedsLayout：告知页面需要更新，但是不会立刻开始更新。执行后会立刻调用layoutSubviews。
*   layoutIfNeeded：告知页面布局立刻更新。所以一般都会和setNeedsLayout一起使用。如果希望立刻生成新的frame需要调用此方法，利用这点一般布局动画可以在更新布局后直接使用这个方法让动画生效。
*   layoutSubviews：系统重写布局
*   setNeedsUpdateConstraints：告知需要更新约束，但是不会立刻开始
*   updateConstraintsIfNeeded：告知立刻更新约束
*   updateConstraints：系统更新约束



#### 总结
```
针对Auto Layout的生命周期，我是这么理解的：
Auto Layout拥有一套Layout Engine引擎，由它来主导页面的布局。
App启动后，主线程的Run Loop会一直处于监听状态，
当约束发生变化后会触发Deffered Layout Pass（延迟布局传递），
在里面做容错处理（约束丢失等情况）并把view标识为dirty状态，然后Run Loop再次进入监听阶段。
当下一次刷新屏幕动作来临（或者是调用layoutIfNeeded）时，
Layout Engine 会从上到下调用 layoutSubviews() ，通过 Cassowary算法计算各个子视图的位置，
算出来后将子视图的frame从Layout Engine拷贝出来，接下来的过程就跟手写frame是一样的了。
```

把循环这部分总结了一下：
*触发约束变化 —> Layout Engine就需要重新计算布局，会先获取到当前的布局，调用SuperView.SetNeedLayout() —> Deffered Layout Pass进行监听 —> Layout Engine 从上到下调用LayoutSubViews()，通过Cassowary算法计算各个子视图的位置，算出来后将子视图的Frame从Layout Engine里拷贝出来 —> 和手写布局的绘制、渲染一样。*


参考：
[https://ming1016.github.io/2015/11/03/deeply-analyse-autolayout/](https://ming1016.github.io/2015/11/03/deeply-analyse-autolayout/)
[https://github.com/forkingdog/fdstackview](https://github.com/forkingdog/fdstackview)
[https://github.com/facebook/yoga](https://github.com/facebook/yoga)
[https://github.com/ming1016/study/wiki/Masonry](https://github.com/ming1016/study/wiki/Masonry)
