---
title: iOS使用CG框架绘制特定形状的view
date: 2017-04-05 15:07:58
tags:
---


####	前言
最近，新项目中，有些相应的需求，要在特殊形状的view中展示数据，然后里边还有些直线，虚线的结合，考虑到使用图片的话不是很好适配，因此这里研究总结了下，使用代码自己来实现相应的需求。

####	开始
*	先来看下实现的效果图
![](https://upload-images.jianshu.io/upload_images/1241385-fbe5d479e9fb91ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*	然后，我们来开始实现它！
<!--more-->
首先要了解： `drawRect:`方法的调用时机：
它是在`init`和`viewDidLoad`方法执行之后，才开始调用的，因此，我们可以在`init`方法中设置相应的参数：
```
@implementation EnsurePaybackFrontView
{
    //虚线距view顶部的距离
    CGFloat _frontHeight;
    //中间两个小半圆的半径
    CGFloat _radii;
    //顶部view的圆角半径
    CGFloat _topRadius;
}

- (instancetype)initWithFrame:(CGRect)frame {
    if (self == [super initWithFrame:frame]) {

        _frontHeight = 64.f;
        _radii = 8.f;
        _topRadius = 4.f;

        self.backgroundColor = [UIColor whiteColor];
    }
    return self;
}

//然后，在drawRect方法中 开始绘制图形
- (void)drawRect:(CGRect)rect {
    // Drawing code
    CGFloat viewWidth = rect.size.width;
    CGFloat viewHieght = rect.size.height;

    //获取绘图上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    //绘制整体背景
    CGContextMoveToPoint(ctx, _topRadius, 0);
    CGContextAddLineToPoint(ctx, viewWidth - _topRadius, 0);
    CGContextAddArc(ctx, viewWidth - _topRadius, _topRadius, _topRadius, 3/2*M_PI, 0, 0);
    CGContextAddLineToPoint(ctx, viewWidth, _frontHeight - _radii);
    CGContextAddArc(ctx, viewWidth, _frontHeight, _radii, 1.5 * M_PI, -M_PI_2, 1);
    CGContextAddLineToPoint(ctx, viewWidth, viewHieght);
    CGContextAddLineToPoint(ctx, 0, viewHieght);
    CGContextAddLineToPoint(ctx, 0, _frontHeight + _radii);
    CGContextAddArc(ctx, 0, _frontHeight, _radii, -M_PI_2, 3 * M_PI_2, 1);
    CGContextAddLineToPoint(ctx, 0, _topRadius);
    CGContextAddArc(ctx, _topRadius, _topRadius, _topRadius, M_PI, M_PI * 1.5, 0);
    CGContextSetFillColorWithColor(ctx, [UIColor yellowColor].CGColor);
    CGContextFillPath(ctx);

    //绘制左边半圆
    CGContextMoveToPoint(ctx, 0, _frontHeight - _radii);
    CGContextAddArc(ctx, 0, _frontHeight, _radii, 3 * M_PI_2, - M_PI_2, 0);
//    CGContextAddLineToPoint(ctx, 0, _frontHeight - _radii);
    CGContextClosePath(ctx);
    CGContextSetFillColorWithColor(ctx, [UIColor colorWithRed:32/255.0 green:170/255.0 blue:251/255.0 alpha:1.0].CGColor);
//    CGContextSetFillColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextFillPath(ctx);
}
```

**在效果图中可以看到，我这里只实现了左边一个半圆的绘制，右边的没有实现，不过实现起来也是同样的道理，这里不再过多赘述。**
*当然，如果整个图形外围形状是那种很规则的，也可以通过直接设置整个view的背景色来实现。*
*不过由于这里上部分存在圆弧部分，所以如果设置背景色，那么弧形外的部分也是需要，重新绘制的，相比下，绘制半圆还是简单一点的，所以，仁者见仁智者见智吧。*

*	外部调用
```
    EnsurePaybackFrontView *test = [[EnsurePaybackFrontView alloc]initWithFrame:CGRectMake(15, 168, 290, 400)];
    test.backgroundColor = [UIColor clearColor];
    UIView *dottedLine = [[UIView alloc]initWithFrame:CGRectMake(8, 64, 290 - 16, 1)];
    //绘制虚线
    [self drawDashLine:dottedLine lineLength:8 lineSpacing:4 lineColor:[UIColor blackColor]];
    [test addSubview:dottedLine];

    [self.view addSubview:test];
```
*	绘制虚线，由于考虑到后期可能会在view上按需求添加额外的多条，所以没有写在内部`drawRect`中，而是写在了外部方便调用
```
/**
 * @param   lineView    需要绘制成虚线的view
 * @param   lineLength  虚线的宽度
 * @param   lineSpacing  虚线的间距
 * @param   lineColor   虚线的颜色
 */
- (void)drawDashLine:(UIView *)lineView lineLength:(int)lineLength lineSpacing:(int)lineSpacing lineColor:(UIColor *)lineColor
{
    CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    [shapeLayer setBounds:lineView.bounds];
    [shapeLayer setPosition:CGPointMake(CGRectGetWidth(lineView.frame) / 2, CGRectGetHeight(lineView.frame))];
    [shapeLayer setFillColor:[UIColor clearColor].CGColor];
    //  设置虚线颜色为blackColor
    [shapeLayer setStrokeColor:lineColor.CGColor];
    //  设置虚线宽度
    [shapeLayer setLineWidth:CGRectGetHeight(lineView.frame)];
    [shapeLayer setLineJoin:kCALineJoinRound];
    //  设置线宽，线间距
    [shapeLayer setLineDashPattern:[NSArray arrayWithObjects:[NSNumber numberWithInt:lineLength], [NSNumber numberWithInt:lineSpacing], nil]];

    //  设置路径
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, NULL, 0, 0);
    CGPathAddLineToPoint(path, NULL,CGRectGetWidth(lineView.frame), 0);
    [shapeLayer setPath:path];
    CGPathRelease(path);
    //  把绘制好的虚线添加上来
    [lineView.layer addSublayer:shapeLayer];
}
```
如果需要查看demo，[Github链接在这里](https://github.com/WooNoah/DrawSpecificShapeView)，如果感觉对您有帮助，麻烦给个赞~谢谢
