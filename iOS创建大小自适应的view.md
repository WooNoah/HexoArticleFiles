---
title: iOS创建大小自适应的view
date: 2019-04-17 17:00:56
tags:
---


> 开发中免不了会遇到要定制那种宽高自适应的view。这里做下总结

#### 1. 包括的子view不可滚动（即不包含UIScrollView的别的控件）
这种其实比较好处理。
```
#import "TestView.h"
#import "Masonry.h"

@interface TestView()

@property (nonatomic, strong) UIView *view1;

@end

@implementation TestView

- (instancetype)initWithFrame:(CGRect)frame {
if (self = [super initWithFrame:frame]) {
self.backgroundColor = [UIColor redColor];
[self createSubviews];
}
return self;
}

- (void)createSubviews {
self.view1 = [[UIView alloc]init];
self.view1.backgroundColor = [UIColor yellowColor];
[self addSubview:self.view1];
[self.view1 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.top.right.bottom.equalTo(self);
make.size.mas_equalTo(CGSizeMake(150, 20));
}];      
}


//ViewController.m中
- (void)viewDidLoad {
TestView *test = [[TestView alloc]init];
[self.view addSubview:test];
[test mas_makeConstraints:^(MASConstraintMaker *make) {
make.top.equalTo(grayView.mas_bottom).offset(5);
make.centerX.equalTo(self.view);
}];
}
```
可以看到，只需要设置子视图的边距跟父视图相同，然后设置子视图的宽高即可。
但是要注意：*在外部给父视图做约束的时候，`不要设置跟size相关的属性`*
![](https://upload-images.jianshu.io/upload_images/1241385-b3d754b67996dac2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，父视图（红色）和子视图（黄色）大小是一致的。

#### 2.子视图中包括可以滚动的控件（UIScrollView）
比如这个需求：
![](https://upload-images.jianshu.io/upload_images/1241385-29170cc6c90d429c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/1241385-6b0ead550b43a63b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
要弹出一个alert框，然后根据中间文字的长短，来调整整个框的大小！然后给一个最大高度，如果文字超过了这个高度，则为可滚动。

按照上面的思路，给UIScrollview添加约束，UIScrollView的内部放一个UILabel，然后设置label的约束为四边距跟scrollview相同。我们来测试一下：
```
@interface TestView()

@property (nonatomic, strong) UIView *view1; //头部view
@property (nonatomic, strong) UIView *view2; //底部view
@property (nonatomic, strong) UIScrollView *scrollView; 
@property (nonatomic, strong) UILabel *content;

@end

- (void)createSubviews {
self.view1 = [[UIView alloc]init];
self.view1.backgroundColor = [UIColor yellowColor];
[self addSubview:self.view1];
[self.view1 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.top.right.equalTo(self);
make.size.mas_equalTo(CGSizeMake(150, 20));
}];


self.scrollView = [[UIScrollView alloc]init];
self.scrollView.backgroundColor = [UIColor orangeColor];
[self addSubview:self.scrollView];
[self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.right.equalTo(self);
make.top.equalTo(self.view1.mas_bottom);

}];

self.content = [[UILabel alloc]init];
self.content.text = @"按时；的房价阿斯兰的风景按时；大楼附近奥德赛；立方阿萨德管理局ad；管理会计we；了感觉肉而过；阿夫林的快感骄傲的高考哈尔；快给我如果；卡；管理科见到过；拉的积分；阿刘的时间安排温哥华；按时给家里；的设计费；阿刘四大金刚；卡到房管局；利润空间感；为了国家；打了个健康； 光和热涵盖了看法那个卡了人家给旅客进入那个卡了进入高考了人工；的房价阿斯兰的风景按时；大楼附近奥德赛；立";
self.content.numberOfLines = 0;
[self.scrollView addSubview:self.content];

CGSize contentSize = [self.content sizeThatFits:CGSizeMake(150, 9999)];
[self.content mas_makeConstraints:^(MASConstraintMaker *make) {
make.edges.equalTo(self.scrollView);
make.size.mas_equalTo(contentSize);
}];

self.view2 = [[UIView alloc]init];
self.view2.backgroundColor = [UIColor greenColor];
[self addSubview:self.view2];
[self.view2 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.right.bottom.equalTo(self);
make.top.equalTo(self.scrollView.mas_bottom);
make.size.equalTo(self.view1);
}];

}
```
可以看到，已经设置了content的四边距和scrollview一致，然后也给了content一个size。
然后设置scrollview的左边和右边，跟父视图一致。
设置view2的顶部挨着scrollview的底部。
然后view2的底部跟父视图底部一致。
这个思路跟上面[1]()中的，是符合的。
但是结果呢？
![](https://upload-images.jianshu.io/upload_images/1241385-f7c249d3fb1c57c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，只有view1和view2，
*UIScrollview并没有显示在页面上*
可见，这个时候，再使用[1]()的方法就不行了。（如果是我约束设置错了，还请各位大佬批评指正）

所以这里使用了另外一种思路。
> 给scrollview的高度添加约束，然后强引用为一个属性。然后在content文字高度计算完全之后，重新更新此约束，即可修改scrollview的高度。当然，此时子视图的各个控件之间高度的约束就可以忽略了。
```
@property (nonatomic, strong) NSLayoutConstraint *heightConstraint;

//这里设置高度约束。初始值为0
self.heightConstraint = [NSLayoutConstraint constraintWithItem:self.scrollView attribute:(NSLayoutAttributeHeight) relatedBy:(NSLayoutRelationEqual) toItem:nil attribute:(NSLayoutAttributeHeight) multiplier:1 constant:0];
[self addConstraint:self.heightConstraint];

//此处计算完成之后，更新约束值
CGSize contentSize = [self.content sizeThatFits:CGSizeMake(150, 9999)];
[self.content mas_makeConstraints:^(MASConstraintMaker *make) {
make.edges.equalTo(self.scrollView);
make.size.mas_equalTo(contentSize);
}];
self.heightConstraint.constant = contentSize.height;

```
![](https://upload-images.jianshu.io/upload_images/1241385-f8b22a4862d64eb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时，可以看到，就能达到我们要的效果。


然后上面的那种弹框中所说的父视图超过最大高度的时候可滚动，可以在获取到高度的时候给约束添加个判断即可：
```
/**
内容label修改行间距

@param contentStr 要展示的内容字符串
*/
- (void)replaceContentUsingSpecificLineSpacingWithString:(NSString *)contentStr {
NSMutableParagraphStyle *mps = [[NSMutableParagraphStyle alloc]init];
mps.lineSpacing = 6;
mps.lineBreakMode = NSLineBreakByWordWrapping;

NSDictionary *contentAttributes = @{
NSParagraphStyleAttributeName: mps,
NSFontAttributeName: [UIFont systemFontOfSize: 13],
};
NSMutableAttributedString *mas = [[NSMutableAttributedString alloc]initWithString:contentStr attributes:contentAttributes];


self.content.attributedText = mas;
[self layoutIfNeeded];

if (self.content.height < kPxValue(428)) {
LOG(@"短");
self.scrollHeightConstraint.constant = self.content.height;
}else {
LOG(@"长");
self.scrollHeightConstraint.constant = kPxValue(428);
self.contentScrollView.contentSize = CGSizeMake(self.contentScrollView.width,self.content.height);
}

}
```
当然， 这种效果图，还有一种实现方法：`UITextView`
这里有一篇文章可供参考
[iOS:如何优雅的让UITextView根据输入文字实时改变高度](https://www.jianshu.com/p/9e960757de86)
