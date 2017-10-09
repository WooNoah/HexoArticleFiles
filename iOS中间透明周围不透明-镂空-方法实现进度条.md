---
title: iOS中间透明周围不透明-镂空-方法实现进度条
date: 2017-04-11 15:07:58
tags:
---

####	前言
最近由于要给APP更新UI，然后UIDesigner为了"尝鲜",给我一些新的界面设计，所以我这里也不得不再来造些另外的轮子。如果对您有帮助，麻烦[Github](https://github.com/WooNoah/WDProgressBar)给个star

这里先来看下今天要造的轮子
![效果图](http://upload-images.jianshu.io/upload_images/1241385-79ae79f01f0b864f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

没错，就是上边的进度条。

####	实现
<!--more-->

*	先说下思路（***采用的是使用MASK遮罩，然后改变底层OrangeColorLayer长度，以达到进度条长度变化的思路***）
		1. 首先，我们创建一个新的类，`继承自UIView`,然后，我们采用`Core Graphics`框架和`UIBezierPath类`来实现中间的进度条样式。先来分析下进度条，可以看出来，进度条内部，有两种颜色：`橙色的TintColor`，`DarkGray的进度达不到的进度条颜色`，然后，`进度条背景色lightDarkColor`,`和最上层圆形中的小圆点`。
		2. 我这里的进度条，实现发方法是***采用4层layer***来实现：最底层，绘制`DarkGray颜色的进度条`，再往上一层，设置有颜色的一层layer,**通过改变该层layer的宽度，已达到有颜色进度条的长度变化**，再上层，是一个`中间进度条轮廓透明，周围是lightGrayColor背景色的遮罩View`,最上层，是几个进度条内部小圆的layer。***采用的是使用MASK遮罩，然后改变底层OrangeColorLayer长度，以达到进度条长度变化的思路***

*	这里代码
		1.先看下暴露出来的API接口

		@interface RRMyMaterialProgressBar : UIView
		//一共项数
		//@property (nonatomic,assign) CGFloat totalStepsCounts;

		@property (nonatomic,strong) CAShapeLayer *firstArc;

		@property (nonatomic,strong) CAShapeLayer *secondArc;

		@property (nonatomic,strong) CAShapeLayer *thirdArc;

		@property (nonatomic,strong) CAShapeLayer *forthArc;

		//当前进度
		@property (nonatomic,assign) CGFloat currentPercentage;
		//渲染颜色
		//@property (nonatomic,strong) UIColor *progressBarTintColor;

		- (instancetype)initWithFrame:(CGRect)frame totalStepsCounts:(CGFloat)steps progressBarTintColor:(UIColor *)tintClr;

		@end

#### PS：
这里是有一些问题的，最开始的时候，我是在`drawRect`方法中实现的，然后把总共的圆形个数，和进度条的渲染颜色都作为了外部属性，也是能实现的，但是：**有些问题**：考虑下我们的使用场景：一般我们都是在外层VC的`viewWillAppear或者viewDidLoad`方法中设置这个progressBar的进度，但是！如果在`drawRect`方法中实现的话，此时，内部的各个layer都还没有初始化，**全部都是nil**，如果在`viewDidAppear`方法中设置的话，会有一个BUG：刚进到页面内的时候，会看到进度条的进度是100%的，（因为默认创建ColorLayer的时候是设置的整个Frame的）,然后，调用外层`viewDidAppear`方法设置进度条的进度Layer长度，此时，进度条才会又从头开始显示。这么一来，用户体验就不是太好了。

所以，我又优化了一下：progressBarView 初始化的时候，就来绘制这些内层layer,以此来保证外部VC调用`viewWillAppear或者viewDidLoad`方法的时候，内层progressBar内的各个实例都已经ALLOC,并且可以设置他们。(所以，以前的`totalStepsCounts和progressBarTintColor`都可以设置为私有变量),然后我们直接在初始化方法中传入参数以赋值。

```
#import "RRMyMaterialProgressBar.h"

@interface RRMyMaterialProgressBar()

@property (nonatomic,strong) CALayer *backColorLayer;

@property (nonatomic,strong) CAShapeLayer *middleDarkArcLayer;

@property (nonatomic,strong) CAShapeLayer *foregroundMaskLayer;


//渲染颜色
@property (nonatomic,strong) UIColor *progressBarTintColor;

//上一次的比例
@property (nonatomic,assign) CGFloat lastPercentage;

//一共项数
@property (nonatomic,assign) CGFloat totalStepsCounts;

@end

@implementation RRMyMaterialProgressBar

- (instancetype)initWithFrame:(CGRect)frame totalStepsCounts:(CGFloat)steps progressBarTintColor:(UIColor *)tintClr{
    if (self == [super initWithFrame:frame]) {
        self.progressBarTintColor = tintClr;
        self.totalStepsCounts = steps;
        [self setupSubLayersWithFrame:frame];
    }
    return self;
}


- (void)setupSubLayersWithFrame:(CGRect)rect {
    self.layer.backgroundColor = [UIColor redColor].CGColor;

    //左右两侧的距离
    CGFloat padding = 10.f;
    //外圆的半径
    CGFloat arcRadius = 7.f;
    //中间线的线宽
    CGFloat middleLineWidth = 1.f;
    //内层圆按钮半径
    CGFloat innerArcRadius = 4.f;
    CGFloat viewWidth = rect.size.width;
    CGFloat viewHeight = rect.size.height;


    double arcSinValue = middleLineWidth/arcRadius;
    double radian = asin(arcSinValue);

    double tanValue = tan(radian);

    CGFloat arcTanLength = middleLineWidth/tanValue;

    CGFloat middleLineDistance = (viewWidth - 2 * padding - arcRadius * 2 - (self.totalStepsCounts * 2 - 2) * arcTanLength)/(self.totalStepsCounts - 1);


    self.middleDarkArcLayer = [CAShapeLayer layer];
    UIBezierPath *middleDrakColorPath = [UIBezierPath bezierPath];
    [middleDrakColorPath moveToPoint:CGPointMake(padding, viewHeight/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius, viewHeight/2) radius:arcRadius startAngle:M_PI endAngle:2*M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius +arcTanLength + middleLineDistance, viewHeight/2 - middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius +arcTanLength * 2 + middleLineDistance, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:2 * M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius + 3 * arcTanLength + middleLineDistance * 2, viewHeight/2 - middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius + 4 * arcTanLength + middleLineDistance * 2, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:2 * M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius + 5 * arcTanLength + middleLineDistance * 3, viewHeight/2 - middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius + 6 * arcTanLength + middleLineDistance * 3, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:3 * M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius + 5 * arcTanLength + middleLineDistance * 2, viewHeight/2 + middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius + 4 * arcTanLength + middleLineDistance * 2, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius + 3 * arcTanLength + middleLineDistance, viewHeight/2 + middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius + 2 * arcTanLength + middleLineDistance, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];
    [middleDrakColorPath addLineToPoint:CGPointMake(padding + arcRadius + arcTanLength, viewHeight/2 + middleLineWidth/2)];
    [middleDrakColorPath addArcWithCenter:CGPointMake(padding + arcRadius, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];
    [middleDrakColorPath closePath];

    self.middleDarkArcLayer.fillColor = RGB(203, 204, 205).CGColor;
    self.middleDarkArcLayer.path = middleDrakColorPath.CGPath;
    [self.layer addSublayer:self.middleDarkArcLayer];


    //MARK: 颜色层layer
    self.backColorLayer = [CALayer layer];
    self.backColorLayer.frame = CGRectMake(0, 0, viewWidth, viewHeight);
    self.backColorLayer.backgroundColor = self.progressBarTintColor.CGColor;
    self.backColorLayer.position = CGPointMake(0, viewHeight/2);
    self.backColorLayer.anchorPoint = CGPointMake(0, 0.5);
    [self.layer addSublayer:self.backColorLayer];


    self.foregroundMaskLayer = [CAShapeLayer layer];
    //MARK: 使用UIBezierPath 绘制遮挡VIEW的路径
    UIBezierPath *outerPath = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(0,0, rect.size.width, rect.size.height) cornerRadius:rect.size.height/2];

    UIBezierPath *colorPath = [UIBezierPath bezierPath];
    [colorPath moveToPoint:CGPointMake(padding, viewHeight/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius, viewHeight/2) radius:arcRadius startAngle:M_PI endAngle:2*M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius +arcTanLength + middleLineDistance, viewHeight/2 - middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius +arcTanLength * 2 + middleLineDistance, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:2 * M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius + 3 * arcTanLength + middleLineDistance * 2, viewHeight/2 - middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius + 4 * arcTanLength + middleLineDistance * 2, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:2 * M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius + 5 * arcTanLength + middleLineDistance * 3, viewHeight/2 - middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius + 6 * arcTanLength + middleLineDistance * 3, viewHeight/2) radius:arcRadius startAngle:M_PI + radian endAngle:3 * M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius + 5 * arcTanLength + middleLineDistance * 2, viewHeight/2 + middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius + 4 * arcTanLength + middleLineDistance * 2, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius + 3 * arcTanLength + middleLineDistance, viewHeight/2 + middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius + 2 * arcTanLength + middleLineDistance, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];
    [colorPath addLineToPoint:CGPointMake(padding + arcRadius + arcTanLength, viewHeight/2 + middleLineWidth/2)];
    [colorPath addArcWithCenter:CGPointMake(padding + arcRadius, viewHeight/2) radius:arcRadius startAngle:radian endAngle:M_PI - radian clockwise:YES];

    [outerPath appendPath:colorPath];
    [outerPath setUsesEvenOddFillRule:YES];


    self.foregroundMaskLayer.path = outerPath.CGPath;
    self.foregroundMaskLayer.fillRule = kCAFillRuleEvenOdd;
    self.foregroundMaskLayer.fillColor = RGB(221, 221, 221).CGColor;
    [self.layer addSublayer:self.foregroundMaskLayer];

    //MARK: 第一个内部圆
    self.firstArc = [CAShapeLayer layer];
    UIBezierPath *arcPath1 = [UIBezierPath bezierPath];
    [arcPath1 addArcWithCenter:CGPointMake(padding + arcRadius, viewHeight/2) radius:innerArcRadius startAngle:0 endAngle:2 * M_PI clockwise:1];
    self.firstArc.path = arcPath1.CGPath;
    self.firstArc.fillColor = RGB(221, 221, 221).CGColor;
    [self.layer addSublayer:self.firstArc];


    //MARK: 第二个内部圆
    self.secondArc = [CAShapeLayer layer];
    UIBezierPath *arcPath2 = [UIBezierPath bezierPath];
    [arcPath2 addArcWithCenter:CGPointMake(padding + arcRadius + arcTanLength + middleLineDistance + arcTanLength, viewHeight/2) radius:innerArcRadius startAngle:0 endAngle:2 * M_PI clockwise:1];
    self.secondArc.path = arcPath2.CGPath;
    self.secondArc.fillColor = RGB(221, 221, 221).CGColor;
    [self.layer addSublayer:self.secondArc];


    //MARK: 第三个内部圆
    self.thirdArc = [CAShapeLayer layer];
    UIBezierPath *arcPath3 = [UIBezierPath bezierPath];
    [arcPath3 addArcWithCenter:CGPointMake(padding + arcRadius + arcTanLength + middleLineDistance + arcTanLength + arcTanLength + middleLineDistance + arcTanLength, viewHeight/2) radius:innerArcRadius startAngle:0 endAngle:2 * M_PI clockwise:1];
    self.thirdArc.path = arcPath3.CGPath;
    self.thirdArc.fillColor = RGB(221, 221, 221).CGColor;
    [self.layer addSublayer:self.thirdArc];


    //MARK: 第四个内部圆
    self.forthArc = [CAShapeLayer layer];
    UIBezierPath *arcPath4 = [UIBezierPath bezierPath];
    [arcPath4 addArcWithCenter:CGPointMake(padding + arcRadius + arcTanLength + middleLineDistance + arcTanLength + arcTanLength + middleLineDistance + arcTanLength + arcTanLength + middleLineDistance + arcTanLength, viewHeight/2) radius:innerArcRadius startAngle:0 endAngle:2 * M_PI clockwise:1];
    self.forthArc.path = arcPath4.CGPath;
    self.forthArc.fillColor = RGB(221, 221, 221).CGColor;
    [self.layer addSublayer:self.forthArc];
}


- (void)setCurrentPercentage:(CGFloat)currentPercentage {
    _currentPercentage = currentPercentage;

    CABasicAnimation *animate = [CABasicAnimation animationWithKeyPath:@"transform.scale.x"];
    animate.fromValue = self.lastPercentage == 0 ? @(0) : @(self.lastPercentage);        
    animate.toValue = @(currentPercentage);
    animate.repeatCount = 1;
    animate.autoreverses = NO;
    animate.duration = 1.f;
    animate.removedOnCompletion = NO;
    animate.fillMode = kCAFillModeForwards;
    [self.backColorLayer addAnimation:animate forKey:@"progressAnimation"];

    self.lastPercentage = currentPercentage;

}

```

####	另外
整个进度条并不是圆形是环形的，如果有UI需求，我们可以通过设置API接口中的四个shapeLayer的`fillColor`值来改变内部小圆的颜色的
```
self.forthArc.fillColor = [UIColor yourWantedColor].CGColor;
```

这里有些[参考资料](http://blog.csdn.net/zhz459880251/article/details/50035631)
都是[关于view中间镂空周围不是空的](https://segmentfault.com/q/1010000005705732)
