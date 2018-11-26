---
title: iOS代码约束处理&&Masonry源码分析
date: 2018-05-11 15:05:47
tags:
---

###iOS约束问题解决方法

####1.NSLayoutConstraint类方法
#####通过VFL语句
VFL语句的写法总结：

```
如果是横向约束：
H: 开头
纵向：
V：开头
（假设要添加约束的对象是currentView）
如果想要与父类添加约束，
则：添加一个"|"
H:|
如果想要添加于某控件(leftView)之间的约束：直接写[该控件名称]
H:[leftView]
因此：
如果想要水平方向的，左边离父视图距离为5，写法为：
H:|-5-[currentView]

右边同理：
水平方向，左边距离父视图5，右边距离rightView为10
H:|-5-[currentView]-10-[rightView]
水平方向，左边距离leftView5像素，右边距离父视图为10
H:[leftView]-5-[currentView]-10-|

竖直方向同理
H(orizontal)  改为V(ertical)即可
```
#####通过类方法constraintWithItem
```
@param view1 指定需要添加约束的视图一
@param attr1 指定视图一需要约束的属性
@param relation 指定视图一和视图二添加约束的关系
@param view2 指定视图一依赖关系的视图二；可为nil
@param attr2 指定视图一所依赖的视图二的属性，若view2=nil，该属性设置 NSLayoutAttributeNotAnAttribute
@param multiplier 系数   情况一和二为亲测
情况一：设置A视图的高度 = A视图高度 * multiplier + constant；此时才会起作用；
情况二：设置A视图和其他视图的关系或 toItem=nil，multiplier设置不等于0即可，若等于0会crash；
@param c 常量
@return 返回生成的约束对象
+(instancetype)constraintWithItem:(id)view1 attribute:(NSLayoutAttribute)attr1 relatedBy:(NSLayoutRelation)relation toItem:(nullable id)view2 attribute:(NSLayoutAttribute)attr2 multiplier:(CGFloat)multiplier constant:(CGFloat)c;

eg:
[NSLayoutConstraint constraintWithItem:headerImageView attribute:(NSLayoutAttributeLeft) relatedBy:(NSLayoutRelationEqual) toItem:self.headerView attribute:(NSLayoutAttributeLeft) multiplier:1.0 constant:0];
```
------------------------

###2.第三方库[Masonry](https://github.com/SnapKit/Masonry)
今天看了一下Masonry的处理：
总结一下：
Masonry说起来就是在NSLayoutConstraint类方法的上层做了些封装，
但是最重要的一点是：***它支持链式语法***
```
它给UIView和UIViewController都添加了category，
添加了一些属性，类型为MASViewAttribute，（类似override了类的上下左右等属性），
就是由于这一点，才支持的链式语法：


使用MASConstraintsMaker类
```
我们以`mas_left`属性为例来看一下他的具体实现
```
- (MASViewAttribute *)mas_left {
return [[MASViewAttribute alloc] initWithView:self layoutAttribute:NSLayoutAttributeLeft];
}
```
我们可以看到：**该类处理之后，仍然返回了一个MASViewAttribute类型的对象**,也正是由于这一点，所以才可以实现：
`make.left.right.height.width.equalTo(self.view).offset(spaceX); ` ***前面这些属性***的链式写法


再来看一下equalTo的实现：
```
- (MASConstraint * (^)(id))equalTo {
return ^id(id attribute) {
return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
};
}

- (MASConstraint * (^)(id))mas_equalTo {
return ^id(id attribute) {
return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
};
}

- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation { MASMethodNotImplemented(); }
```
我们可以看到`.equalTo`和`.mas_equalTo`底层实现都是一致的
```
/**
*    Sets the constraint relation to given NSLayoutRelation
*  returns a block which accepts one of the following:
*    MASViewAttribute, UIView, NSValue, NSArray
*  see readme for more details.
*/
- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation;
```
这个方法加上上面的.equalTo,都是通过Block传入数据，加一起，等于变向实现了苹果原生方法：
`-constraintWithItem`，
param1,需要添加约束的视图：category传入self
param2. 需要的约束属性： 前面mas_xxx实现
param3. 视图一和视图二的约束关系： NSLayoutRelationEqual
param4. 视图一依赖的视图： 通过block传入
param5. 被依赖视图的属性：block传入，可以是MASViewAttribute, UIView, NSValue, NSArray中的一种
param6. 系数： 
```
通过该方法传入
- (MASConstraint * (^)(CGFloat multiplier))multipliedBy { MASMethodNotImplemented(); }
```
param7. 数值：
```
通过该方法传入
- (MASConstraint * (^)(CGFloat))offset {
return ^id(CGFloat offset){
self.offset = offset;
return self;
};
}
```
*由此可以看出，苹果原生NSLayoutAttribute约束方法需要的参数，都通过点语法都给拼接完成*

-------------

插一句 `.left`和`mas_left`实现也是一致的
```
#define MAS_ATTR_FORWARD(attr)  \
- (MASViewAttribute *)attr {    \
return [self mas_##attr];   \
}

@interface MAS_VIEW (MASShorthandAdditions)
@property (nonatomic, strong, readonly) MASViewAttribute *left;

@implementation MAS_VIEW (MASShorthandAdditions)
MAS_ATTR_FORWARD(top);
MAS_ATTR_FORWARD(left);

```
可以看出来，`.left`底层还是 调用的是`mas_left`方法

-----------------

然后在category中调用的时候，需要有一个统一的入口类，这里MASConstraintMaker类应运而生
```
@interface MASConstraintMaker : NSObject

/**
*    The following properties return a new MASViewConstraint
*  with the first item set to the makers associated view and the appropriate MASViewAttribute
*/
@property (nonatomic, strong, readonly) MASConstraint *left;
@property (nonatomic, strong, readonly) MASConstraint *top;
@property (nonatomic, strong, readonly) MASConstraint *right;
@property (nonatomic, strong, readonly) MASConstraint *bottom;
@property (nonatomic, strong, readonly) MASConstraint *leading;
@property (nonatomic, strong, readonly) MASConstraint *trailing;
@property (nonatomic, strong, readonly) MASConstraint *width;
@property (nonatomic, strong, readonly) MASConstraint *height;
@property (nonatomic, strong, readonly) MASConstraint *centerX;
@property (nonatomic, strong, readonly) MASConstraint *centerY;
@property (nonatomic, strong, readonly) MASConstraint *baseline;

@property (nonatomic, strong, readonly) MASConstraint *firstBaseline;
@property (nonatomic, strong, readonly) MASConstraint *lastBaseline;

#if TARGET_OS_IPHONE || TARGET_OS_TV

@property (nonatomic, strong, readonly) MASConstraint *leftMargin;
@property (nonatomic, strong, readonly) MASConstraint *rightMargin;
@property (nonatomic, strong, readonly) MASConstraint *topMargin;
@property (nonatomic, strong, readonly) MASConstraint *bottomMargin;
@property (nonatomic, strong, readonly) MASConstraint *leadingMargin;
@property (nonatomic, strong, readonly) MASConstraint *trailingMargin;
@property (nonatomic, strong, readonly) MASConstraint *centerXWithinMargins;
@property (nonatomic, strong, readonly) MASConstraint *centerYWithinMargins;

#endif
```
所以，在创建方法中
```
/**
*  Creates a MASConstraintMaker with the callee view.
*  Any constraints defined are added to the view or the appropriate superview once the block has finished executing
*
*  @param block scope within which you can build up the constraints which you wish to apply to the view.
*
*  @return Array of created MASConstraints
*/
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block;
```
传入一个maker对象，我们就可以操作类的属性了。
```
self.view2 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.equalTo(self.view1).offset(5);
}
```
