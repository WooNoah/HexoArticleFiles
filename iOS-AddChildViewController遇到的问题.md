---
title: iOS AddChildViewController遇到的问题
date: 2019-01-10 16:08:22
tags:
---


> 今天在项目中检查循环引用问题的时候遇到了此问题，查询了别的页面，发现打印log并不一样。所以查阅了资料，这里记录一下。

很多时候，我们都会遇到，在一个viewController中，添加别的controller，已达到特殊的转场效果，或者为了用户能在一个页面看到并和多个页面的内容交互的效果。
因此，苹果给我们提供了这个概念，和实现方法：`addChildViewController`

这里有一个使用场景，
1. 页面初始化的时候，在viewDidLoad中添加了子控制器，此时先加载缓存数据（也就是页面会先展示）
2. 然后展示的同时，重新调用接口，拉取数据。
3. 得到数据之后，重新刷新页面
相信大部分人也都是这么实现的。

因此，demo中是这么写的：
> A(ViewController)页面中有一个按钮，跳转到B(TestViewController)页面, 然后B页面中有两个按钮，第一个按钮用来第一次添加childViewController->C页面(SecViewController), 第二个页面，用来模拟网络请求得到结果之后，重新刷新布局的操作。

这里有一个上述操作打印的log图
![](https://upload-images.jianshu.io/upload_images/1241385-7533db896cf0aabb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`可以看到中间很醒目的dealloc！此处就造成了我的疑惑`
然后来看下containerViewController中是怎么实现的：
```
//TestViewController.m
- (void)configureWithData:(NSArray *)array{
if (!array.count) return;


NSLog(@"~~~~~~~~~~~~~~~~~~~~%@ ---- %@ 前",NSStringFromClass([self class]), NSStringFromSelector(_cmd));
[self.childViewControllers makeObjectsPerformSelector:@selector(removeFromParentViewController)];
NSLog(@"~~~~~~~~~~~~~~~~~~~~%@ ---- %@ 后",NSStringFromClass([self class]), NSStringFromSelector(_cmd));


[array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
SecViewController *listVC = [[SecViewController alloc] init];
listVC.view.frame = self.view.bounds;
listVC.parentContainerViewController = self;
[self addChildViewController:listVC];
//        [listVC didMoveToParentViewController:self];
}];
}
```
此方法在重新布局UI的时候调用。

可以看到；其中有一句
```
[self.childViewControllers makeObjectsPerformSelector:@selector(removeFromParentViewController)];
```
此处记录一下：
[苹果官方文档](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html)中是这么描述的：
```
### Removing a Child View Controller

To remove a child view controller from your content, remove the parent-child relationship between the view controllers by doing the following:

1.  Call the child’s `[willMoveToParentViewController:](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621381-willmovetoparentviewcontroller)` method with the value `nil`.

2.  Remove any constraints that you configured with the child’s root view.

3.  Remove the child’s root view from your container’s view hierarchy.

4.  Call the child’s `[removeFromParentViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621425-removefromparentviewcontroller)` method to finalize the end of the parent-child relationship.

Removing a child view controller permanently severs the relationship between parent and child. Remove a child view controller only when you no longer need to refer to it. For example, a navigation controller does not remove its current child view controllers when a new one is pushed onto the navigation stack. It removes them only when they are popped off the stack.

```
大致意思就是：
*如果想要移除一个子控制器，需要自控制器调用`willMoveToParentViewController `, 移除子控制器view在父视图中的位置约束，从父容器的层级中移除子控制器，然后调用子控制器的`removeFromParentViewController `。*
**此种操作，就会造成子控制器不再被引用，然后执行了`dealloc`方法**！
*所以，此处就会造成， 在进入一个页面的时候（默认是只在dealloc处打印了，没有在viewDidLoad处打印），先看到了子控制器的dealloc*

### 附 [Demo](https://github.com/WooNoah/AddChildViewControllerD.git)
