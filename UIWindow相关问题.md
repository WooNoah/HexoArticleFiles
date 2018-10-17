---
title: UIWindow相关问题
date: 2018-10-17 16:22:46
tags:
---

> 这里记录一些关于UIWindow相关的问题

1. #### [UIApplication shareApplication].keyWindow上面添加子控件无法响应
项目中有些类似的页面，需要封装起来，但是在封装的过程中，遇到了这个问题，
总结一下大致流程：
1. 在一个页面A中，需要弹出一个带有多个按钮以供选择的页面B，（类似以前写的地址选择器）,页面B中又包含了一个封装了collectionView的页面C，跳转逻辑就是这样。
2. 但是以前写地址选择器的时候，子视图上的控件都是`直接添加到新建的view上`，但是现在说的情况是添加到`[[[UIApplication sharedApplication] delegate] window] `或者`[UIApplication sharedApplication].keyWindow`上，情况不太一样
<!--more-->
1. 先来说添加到自定义view上的情况：
![](https://upload-images.jianshu.io/upload_images/1241385-e8bb40d55d5a0bca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现方法1： 底部放一个遮罩view，用来调整底部背景色，然后主要显示的view放在一个frontView上，子视图也全放在frontView上
以上面图示为例，层次图如下：
![层次图](https://upload-images.jianshu.io/upload_images/1241385-a5383c535e54436d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码：
```
//底部背景view
UIImageView *backView = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, UIScreenSize.width, UIScreenSize.height)];
backView.userInteractionEnabled = YES;
backView.backgroundColor = [UIColor blackColor];
backView.alpha = .7;
[self addSubview:backView];

UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(dismissKeyboard)];
[backView addGestureRecognizer:tap];

UIView *frontView = [[UIView alloc]init];
frontView.backgroundColor = [UIColor whiteColor];
frontView.layer.cornerRadius = 10.f;
frontView.layer.masksToBounds = YES;
[self addSubview:frontView];

[frontView mas_makeConstraints:^(MASConstraintMaker *make) {
make.centerX.equalTo(self);
make.centerY.equalTo(self).offset(-heiMultiply(0.1));
make.size.mas_equalTo(CGSizeMake(widMultiply(.7), widMultiply(.7)*0.78));
}];

UILabel *titleLbl = [[UILabel alloc]init];
CAGradientLayer *gradientLayer =  [CAGradientLayer layer];
gradientLayer.frame = CGRectMake(0, 0, widMultiply(.9), 50);
gradientLayer.startPoint = CGPointMake(0, 0);
gradientLayer.endPoint = CGPointMake(1, 0);
gradientLayer.locations = @[@(0.1),@(1.0)];
[gradientLayer setColors:@[(id)[RGB(49, 206, 252) CGColor], (id)[RGB(30, 175, 252) CGColor]]];
[titleLbl.layer addSublayer:gradientLayer];

titleLbl.text = @"请输入还款金额";
titleLbl.textAlignment = NSTextAlignmentCenter;
titleLbl.font = [UIFont boldSystemFontOfSize:15];
titleLbl.userInteractionEnabled = YES;
titleLbl.textColor = [UIColor whiteColor];
[frontView addSubview:titleLbl];

[titleLbl mas_makeConstraints:^(MASConstraintMaker *make) {
make.top.equalTo(frontView);
make.left.equalTo(frontView);
make.right.equalTo(frontView);
make.height.mas_equalTo(50);
}];

self.closeBtn = [UIButton buttonWithType:UIButtonTypeCustom];
[self.closeBtn setImage:[UIImage imageNamed:@"x"] forState:UIControlStateNormal];
self.closeBtn.adjustsImageWhenHighlighted = NO;
[self.closeBtn addTarget:self action:@selector(dismissAction) forControlEvents:UIControlEventTouchUpInside];
[titleLbl addSubview:self.closeBtn];

[self.closeBtn mas_makeConstraints:^(MASConstraintMaker *make) {
make.centerY.equalTo(titleLbl);
make.right.equalTo(titleLbl).offset(-15);
make.size.mas_equalTo(CGSizeMake(14, 14));
}];

self.confirmButton = [[UIButton alloc]init];
CAGradientLayer *buttonGradientLayer =  [CAGradientLayer layer];
buttonGradientLayer.frame = CGRectMake(0, 0, widMultiply(.9) - 30, 50);
buttonGradientLayer.startPoint = CGPointMake(0, 0);
buttonGradientLayer.endPoint = CGPointMake(1, 0);
buttonGradientLayer.locations = @[@(0.1),@(1.0)];
[buttonGradientLayer setColors:@[(id)[RGB(46, 229, 253) CGColor],(id)[RGB(41, 195, 252) CGColor]]];
[self.confirmButton.layer addSublayer:buttonGradientLayer];

[self.confirmButton setTitle:@"确认" forState:UIControlStateNormal];
self.confirmButton.titleLabel.font = [UIFont systemFontOfSize:15];
[frontView addSubview:self.confirmButton];

[self.confirmButton mas_makeConstraints:^(MASConstraintMaker *make) {
make.bottom.equalTo(frontView);
make.left.equalTo(frontView);
make.centerX.equalTo(frontView);
make.height.mas_equalTo(50);
}];

self.repayAmount = [[UITextField alloc]init];
self.repayAmount.layer.borderWidth = 0.5f;
self.repayAmount.layer.borderColor = RGB(207, 207, 207).CGColor;
self.repayAmount.keyboardType = UIKeyboardTypeDecimalPad;
[frontView addSubview:self.repayAmount];

[self.repayAmount mas_makeConstraints:^(MASConstraintMaker *make) {
make.center.equalTo(frontView);
make.width.equalTo(frontView).multipliedBy(0.66);  // 350 / 530
make.height.equalTo(frontView).multipliedBy(0.22);   // 90 / 410
}];
```
可以看到，如果有导航栏的时候，因为view是属于NavigationController的，所以`view是无法遮盖住导航栏的`
但是，此时，`底部背景view的手势是可以响应的`。
2. 再来另外一种实现方式
内部控件摆放方式都差不多，看个人需要自行摆放，
主要的差距在，`第一个方法是放在了自定义view，self上了，第二种，就是直接放在主window上`
先看效果图：
![](https://upload-images.jianshu.io/upload_images/1241385-3df55ff4feee1110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，`此时导航栏已经被遮挡着了`。
![层次图](https://upload-images.jianshu.io/upload_images/1241385-e1367e4ba8b5dcb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码：
```
self.gestureView = [[UIView alloc]initWithFrame:[UIScreen mainScreen].bounds];
self.gestureView.backgroundColor = [UIColor colorWithWhite:0.4 alpha:1];
self.gestureView.alpha = 0;
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(tapAction)];
[self.gestureView addGestureRecognizer:tap];
[[[[UIApplication sharedApplication] delegate] window] addSubview:self.gestureView];

self.btn = [UIButton buttonWithType:UIButtonTypeCustom];
self.btn.backgroundColor = [UIColor orangeColor];
self.btn.frame = CGRectMake(100, 100, 100, 44);
[self.btn setTitle:@"customButton" forState:UIControlStateNormal];
[self.btn addTarget:self action:@selector(btnClick) forControlEvents:UIControlEventTouchUpInside];
[[[[UIApplication sharedApplication] delegate] window] addSubview:self.btn];


self.two = [[TestViewTwo alloc]init];
self.two.backgroundColor = [UIColor greenColor];
[self.gestureView addSubview:self.two];
[self.two mas_makeConstraints:^(MASConstraintMaker *make) {
make.top.equalTo(@(190));
make.centerX.equalTo(self.gestureView);
make.left.equalTo(self.gestureView).offset(50);
make.height.equalTo(self.two.mas_width);
}];

[self show];
```
外部调用如下：
```
- (IBAction)customAction:(id)sender {
TestView *test = [[TestView alloc]initWithFrame:[UIScreen mainScreen].bounds];
}


- (IBAction)repayAction:(id)sender {
self.alertView = [[EnsureRepayAlertView alloc]initWithFrame:self.view.bounds];
[self.view addSubview:self.alertView];
[UIView animateWithDuration:.3 animations:^{
self.alertView.alpha = 1;
}];
}
```
此时，方法二的实现方式，你会发现：`内部添加的手势无法响应了！`
发现这个问题也是很糟心了，我一度怀疑事件传递出现了问题，然后想要使用UIWindow的`[- sendEvent:]`方法来强制添加事件，但是又一次寻找bug的过程中， 我突然想到了打印一下**类的声明周期方法**
然后，我就打印了`dealloc`方法，然后我竟然发现，`该方法被调用了！`
然后，顺理成章的，我以为引用计数为0然后被销毁了，所以在外部强引用了此view，
```
@interface ViewController ()
@property (strong, nonatomic) TestView *test;
@end

- (IBAction)customAction:(id)sender {
//TestView *test = [[TestView alloc]initWithFrame:[UIScreen mainScreen].bounds];  
//也就是此处test对象实现为viewcontroller的属性， 变为：
self.test = [[TestView alloc]initWithFrame:[UIScreen mainScreen].bounds];
}
```
此时再次运行，发现果然手势可以被响应了！
是不是很开心？然后迫不及待的要上传SVN了？
NONONO！
其实，这里并不能完全没问题，
比如我们是实际效果图为：
![](https://upload-images.jianshu.io/upload_images/1241385-a395a368df68c812.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
刚刚我们只是测试了外部一个view，即类似上面frontView层级的view,然后内部view存在collectionItem的点击事件，所以我使用代理回传了点击事件。常规写法如下：
```
@protocol SQRegulatorContentViewDelegate <NSObject>

- (void)didClickRegulatorButton:(SQRegulatorButton *)sender;

@end

@interface SQRegulatorContentView : UIView

- (instancetype)initWithRegulators:(NSArray *)regulators;

@property (nonatomic, copy) NSArray *regulators;

@property (nonatomic, assign) CGFloat selfHeight;

@property (nonatomic, weak) id<SQRegulatorContentViewDelegate> delegate;

@end
```
然后继续断点调试，发现`SQRegulatorContentView `的delegate属性为nil，猜想，内部子view也调用了dealloc方法，打印如下：
![](https://upload-images.jianshu.io/upload_images/1241385-f759788322e2c2a9.gif?imageMogr2/auto-orient/strip)
可以看到：在view创建的时候，竟然执行了dealloc方法。这里网上查到一个类似的问题：也是
[把AlertView添加到window上然后无法显示的问题](https://blog.csdn.net/lnking1992/article/details/80354773)
但是改完了之后，还是无法避免上述的问题，此问题暂时不知道解决办法，只能是一层的时候，还可以使用这种方式，如果子view中还添加了别的子view,那么就会出现所述的调用dealloc的问题



参考：
https://blog.csdn.net/u010850094/article/details/51577624
https://www.baidu.com/s?wd=%5Buiapplication%20shareApplication%5D.keywindow%20%E5%AD%90%E8%A7%86%E5%9B%BE%20%E6%89%8B%E5%8A%BF%E6%97%A0%E6%B3%95%E5%93%8D%E5%BA%94
