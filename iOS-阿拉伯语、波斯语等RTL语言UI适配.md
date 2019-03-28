---
title: iOS 阿拉伯语、波斯语等RTL语言UI适配
date: 2019-03-27 20:37:21
tags:
---


阿拉伯语言是从右到左的方向，然后可以使用约束来适配，
都知道有left、right 和 leading、trailing两组约束属性
```
UIButton *codeButton1 = [[UIButton alloc] init];
codeButton1.translatesAutoresizingMaskIntoConstraints = NO;
[codeButton1 setTitle:@"leadingTrailingButton" forState:UIControlStateNormal];
[codeButton1 setTitleColor:[UIColor orangeColor] forState:UIControlStateNormal];
codeButton1.backgroundColor = [UIColor grayColor];
[self.view addSubview:codeButton1];


//添加约束和父视图leading偏差30
NSLayoutConstraint *codeButton1LeftConstraint =
[NSLayoutConstraint constraintWithItem:codeButton1
attribute:NSLayoutAttributeLeading
relatedBy:NSLayoutRelationEqual
toItem:self.view
attribute:NSLayoutAttributeLeading
multiplier:1.f
constant:30.f];

NSLayoutConstraint *codeButton1TopConstraint =
[NSLayoutConstraint constraintWithItem:codeButton1
attribute:NSLayoutAttributeTop
relatedBy:NSLayoutRelationEqual
toItem:codeButton  //codeButton是另一个在codeButton1上方的按钮
attribute:NSLayoutAttributeBottom
multiplier:1.f
constant:100.f];
NSLayoutConstraint *codeButton1WidthConstraint =
[NSLayoutConstraint constraintWithItem:codeButton1
attribute:NSLayoutAttributeWidth
relatedBy:NSLayoutRelationEqual
toItem:nil
attribute:NSLayoutAttributeNotAnAttribute //这里设置的是一个定值宽度的约束，所以使用NSLayoutAttributeNotAnAttribute
multiplier:1.f
constant:200.f];  
NSLayoutConstraint *codeButton1HeightConstraint =
[NSLayoutConstraint constraintWithItem:codeButton1
attribute:NSLayoutAttributeHeight
relatedBy:NSLayoutRelationEqual
toItem:nil
attribute:NSLayoutAttributeNotAnAttribute
multiplier:1.f
constant:44.f];

//把约束添加给父视图
[self.view addConstraints:@[codeButton1LeftConstraint,codeButton1TopConstraint]];
```

如果我们使用leading/trailing，此时会根据UI的方向来改变对左右的定义，
而如果我们使用left/right,则没有什么变化，（左边一直是左边，右边一直是右边）

这里可以手动使用代码来设置UIView的方向：
```
[UIView appearance].semanticContentAttribute = UISemanticContentAttributeForceRightToLeft;
```


而semanticContentAttribute有以下几种属性
```
typedef NS_ENUM(NSInteger, UISemanticContentAttribute) {
UISemanticContentAttributeUnspecified = 0,
UISemanticContentAttributePlayback, // for playback controls such as Play/RW/FF buttons and playhead scrubbers
UISemanticContentAttributeSpatial, // for controls that result in some sort of directional change in the UI, e.g. a segmented control for text alignment or a D-pad in a game
UISemanticContentAttributeForceLeftToRight,
UISemanticContentAttributeForceRightToLeft
} NS_ENUM_AVAILABLE_IOS(9_0);
```

我们可以通过下面两个方法获取UI布局的方向
```
// This method returns the layout direction implied by the provided semantic content attribute relative to the application-wide layout direction (as returned by UIApplication.sharedApplication.userInterfaceLayoutDirection).
+ (UIUserInterfaceLayoutDirection)userInterfaceLayoutDirectionForSemanticContentAttribute:(UISemanticContentAttribute)attribute NS_AVAILABLE_IOS(9_0);

// This method returns the layout direction implied by the provided semantic content attribute relative to the provided layout direction. For example, when provided a layout direction of RightToLeft and a semantic content attribute of Playback, this method returns LeftToRight. Layout and drawing code can use this method to determine how to arrange elements, but might find it easier to query the container view’s effectiveUserInterfaceLayoutDirection property instead.
+ (UIUserInterfaceLayoutDirection)userInterfaceLayoutDirectionForSemanticContentAttribute:(UISemanticContentAttribute)semanticContentAttribute relativeToLayoutDirection:(UIUserInterfaceLayoutDirection)layoutDirection NS_AVAILABLE_IOS(10_0);
```


#### 参考
[https://www.jianshu.com/p/34b5a8d9a77e](https://www.jianshu.com/p/34b5a8d9a77e)
[https://www.jianshu.com/p/d556a42bb392](https://www.jianshu.com/p/d556a42bb392)

[https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/SupportingRight-To-LeftLanguages/SupportingRight-To-LeftLanguages.html#//apple_ref/doc/uid/10000171i-CH17-SW1](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/SupportingRight-To-LeftLanguages/SupportingRight-To-LeftLanguages.html%23//apple_ref/doc/uid/10000171i-CH17-SW1)

|
