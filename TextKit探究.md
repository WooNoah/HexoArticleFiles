---
title: TextKit探究
date: 2018-10-23 11:14:45
tags:
---

> 最近优化代码，遇到了一个问题，这里记录一下：
> 类似一个搜索框页面，根据搜索的内容调接口请求，若接口返回数据为空的时候，需要显示一个无数据的view,但是view中要包含用户刚刚textfield键入的内容。也就是说view的高度宽度要自适应这种情况

先来看看效果图：
![](https://upload-images.jianshu.io/upload_images/1241385-812d7acdba73ef12.gif?imageMogr2/auto-orient/strip)

#### 1. 本来思路:
以前都是中文模式，label显示的内容都比较短，可以一行显示完全，所以实现方式就比较多了，可以使用3个label，分别显示前、中、后三段内容，然后分别配置各个label的属性。
**但是！**，后来需求变动，要添加英文模式，这里就能看的，英文模式下，要显示的内容就大大加长了。（也就是说一行不能完全显示）
这里就要***使内容高度宽度自适应了***
这也就是将要提到的[思路2]()
<!--more-->

#### 2. 思路2，使用UITextview来实现
因为UITextview已经默认将内容拼合起来， 而且它存在一个`attributedText`属性
```
@property(null_resettable,copy) NSAttributedString *attributedText NS_AVAILABLE_IOS(6_0);
```
所以，我们可以根据`range`或者根据正则来配置中间用户键入的文字属性。
自定义view配置各个控件就不贴了。
这里主要就是一个宽度的计算问题：
```
- (void)layoutSubviews {
[super layoutSubviews];

//textView的最合适的size
CGSize appropriateSize = [self.searchResult sizeThatFits:CGSizeMake(self.bounds.size.width - 5 - 45, CGFLOAT_MAX)];
[self updateSubviewConstraintsWithTextViewSize:appropriateSize];
}

此处的约束方法是，这个view最宽为整个屏幕宽度的0.8，
内部如果宽度小于整个屏幕的0.8倍，则显示原始大小，若大于0.8倍。则宽度为0.8*ScreenWidth.
然后高度自适应

//所以外部约束如下：
self.frontView = [[combineSearchResultView alloc]init];
[self addSubview:self.frontView];
[self.frontView mas_makeConstraints:^(MASConstraintMaker *make) {
make.centerX.equalTo(self);
make.top.equalTo(self).offset(54);
make.width.lessThanOrEqualTo(self).multipliedBy(0.8);
}];
//内部约束如下：
/**
* 更新子视图约束
*/
- (void)updateSubviewConstraintsWithTextViewSize:(CGSize)tvSize {
{...}
//此处是textView的约束
[self.searchResult mas_remakeConstraints:^(MASConstraintMaker *make) {
make.left.equalTo(self.icon.mas_right).offset(5);
make.top.equalTo(self.icon);
make.width.equalTo(@(tvSize.width));
make.height.equalTo(@(tvSize.height));
make.right.equalTo(self);
}];
{...}
}
```
还有一点要注意：
我发现一般来说UITextview都会有上下左右四个方向的边距（大约为8px），若想要修改UITextview中内容的四个边距，一般来说我们都会设置`contentInset`属性，但是，在textview中好像并不太好使，看了下文档，里边有一个属性
```
// Inset the text container's layout area within the text view's content area
@property(nonatomic, assign) UIEdgeInsets textContainerInset NS_AVAILABLE_IOS(7_0);
```
可以看到，此属性就是修改textview文本内容的containerView的内边距的
设置一下即可： 
```
self.searchResult.textContainerInset = UIEdgeInsetsMake(0, 0, 0, 0);
```
**注：若textview的约束没设置正确，设置textContainerInset也有可能会显示不正常**

以上就能达到上图那样的效果

#### 3.引申，既然是显示文字的控件，那我们就来看下他们的底层实现：TextKit
`TextKit`是苹果提供的基于`Core Text`的渲染带文字的诸如`UILaebl`,`UITextField`,`UITextView`等控件的高级API。
![](https://upload-images.jianshu.io/upload_images/1241385-d1bd39daf36b60ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先介绍一下，它包含的三大主要的类：
`NSTextStorage、NSLayoutManager、NSTextContainer`，正如苹果官方推荐的MVC模式一样，这三个类也分别代表了不同的部分：
![](https://upload-images.jianshu.io/upload_images/1241385-5c6d193b12738373.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到：
1. ##### NSTextStorage
相当于MVC中的model，用来提供要展示的数据。
而且NSTextStorage是NSMutableAttributedString的子类，所以配置的时候，可以添加一些文字需要展示的属性。
平时我们在使用的时候，可以直接使用此类的实例，但是，如果需要自定义的时候，要继承于他，此时，我们要额外实现四个方法：
```
/* Note for subclassing NSTextStorage: NSTextStorage is a semi-abstract subclass of NSMutableAttributedString.
It implements change management (beginEditing/endEditing), 
verification of attributes, delegate handling, and layout management notification. 
The one aspect it does not implement is the actual attributed string storage 
--- this is left up to the subclassers, which need to override the two NSMutableAttributedString primitives 
in addition to two NSAttributedString primitives:

- (NSString *)string;
- (NSDictionary *)attributesAtIndex:(NSUInteger)location effectiveRange:(NSRangePointer)range;

- (void)replaceCharactersInRange:(NSRange)range withString:(NSString *)str;
- (void)setAttributes:(NSDictionary *)attrs range:(NSRange)range;

These primitives should perform the change then call edited:range:changeInLength: to get everything else to happen.
*/

NS_CLASS_AVAILABLE(10_0, 7_0) @interface NSTextStorage : NSMutableAttributedString
```
可以看到，我们要额外实现：四个方法，来配置string，然后NSTextStorage会调用`edited:range:changeInLength`方法
```
/**************************** Edit management ****************************/

/*
Notifies and records a recent change.  
If there are no outstanding -beginEditing calls, 
this method calls -processEditing to trigger post-editing processes.  
This method has to be called by the primitives after changes are made if subclassed and overridden.  
editedRange is the range in the original string (before the edit).
*/
- (void)edited:(NSTextStorageEditActions)editedMask range:(NSRange)editedRange changeInLength:(NSInteger)delta;

/* 
Sends out -textStorage:willProcessEditing, fixes the attributes, 
sends out -textStorage:didProcessEditing, and notifies the layout 
managers of change with the -processEditingForTextStorage:edited:range:changeInLength:invalidatedRange: method.  
Invoked from -edited:range:changeInLength: or -endEditing.
*/
- (void)processEditing;
```
可以看到，上面的方法会记录改变的信息，也就是说**string的每一次改变, NSTextStorage都会调用此方法。**然后 *edited:range:changeInLength:*方法也会自动调用*beginEdiging*方法，传递文字变化状态，
（也可以使用*beginEditing/endEditing*方法包裹住要处理的数据，在endEditing方法调用之后，会自动调用*processEditing*传递变化。）
然后*processEditing*，会调用**NSLayoutManager**的*textStorage:edited:range:changeInLength:invalidatedRange:*方法把textStorage传过来的字符串绘制出图形


2. ##### NSLayoutManager
**NSLayoutManager**则相当于MVC中的Controller.用来协调model的数据和展示view(UITextView,UITextfield,UILabel etc.)之间的关系。

这里有一点需要注意：
***如果我们自己要绘制文字的时候，需要先绘制背景，然后再绘制文字。***
常用的方法有几大类：

1).  生成类：可以生成图形和布局
```
/************************ Causing glyph generation and layout ************************/

// These methods allow clients to specify exactly the portions of the document for which they wish to have glyphs or layout.  This is particularly important if non-contiguous layout is enabled.  The layout manager still reserves the right to perform glyph generation or layout for larger ranges.  If non-contiguous layout is not enabled, then the range in question will always effectively be extended to start at the beginning of the text.
- (void)ensureGlyphsForCharacterRange:(NSRange)charRange;
- (void)ensureGlyphsForGlyphRange:(NSRange)glyphRange;
- (void)ensureLayoutForCharacterRange:(NSRange)charRange;
- (void)ensureLayoutForGlyphRange:(NSRange)glyphRange;
- (void)ensureLayoutForTextContainer:(NSTextContainer *)container;
- (void)ensureLayoutForBoundingRect:(CGRect)bounds inTextContainer:(NSTextContainer *)container;
```
2). 获取类：获取图形、布局的信息
```
/************************ Get layout information ************************/
// Returns the container in which the given glyph is laid and (optionally) by reference the whole range of glyphs that are in that container.  This will cause glyph generation and layout for the line fragment containing the specified glyph, or if non-contiguous layout is not enabled, up to and including that line fragment; if non-contiguous layout is not enabled and effectiveGlyphRange is non-NULL, this will additionally cause glyph generation and layout for the entire text container containing the specified glyph.
- (nullable NSTextContainer *)textContainerForGlyphAtIndex:(NSUInteger)glyphIndex effectiveRange:(nullable NSRangePointer)effectiveGlyphRange;
- (nullable NSTextContainer *)textContainerForGlyphAtIndex:(NSUInteger)glyphIndex effectiveRange:(nullable NSRangePointer)effectiveGlyphRange


/************************ Get glyphs and glyph properties ************************/
// Returns the total number of glyphs.  If non-contiguous layout is not enabled, this will force generation of glyphs for all characters.
@property (readonly, NS_NONATOMIC_IOSONLY) NSUInteger numberOfGlyphs;

// If non-contiguous layout is not enabled, these will cause generation of all glyphs up to and including glyphIndex.  The first CGGlyphAtIndex variant returns kCGFontIndexInvalid if the requested index is out of the range (0, numberOfGlyphs), and optionally returns a flag indicating whether the requested index is in range.  The second CGGlyphAtIndex variant raises a NSRangeError if the requested index is out of range.
- (CGGlyph)CGGlyphAtIndex:(NSUInteger)glyphIndex isValidIndex:(nullable BOOL *)isValidIndex NS_AVAILABLE(10_11,7_0);
- (CGGlyph)CGGlyphAtIndex:(NSUInteger)glyphIndex NS_AVAILABLE(10_11,7_0);
- (BOOL)isValidGlyphIndex:(NSUInteger)glyphIndex API_AVAILABLE(macosx(10.0), ios(7.0), watchos(2.0), tvos(9.0));

// If non-contiguous layout is not enabled, this will cause generation of all glyphs up to and including glyphIndex.  It will return the glyph property associated with the glyph at the specified index.
- (NSGlyphProperty)propertyForGlyphAtIndex:(NSUInteger)glyphIndex NS_AVAILABLE(10_5, 7_0);

// If non-contiguous layout is not enabled, this will cause generation of all glyphs up to and including glyphIndex.  It will return the character index for the first character associated with the glyph at the specified index.
- (NSUInteger)characterIndexForGlyphAtIndex:(NSUInteger)glyphIndex;

// If non-contiguous layout is not enabled, this will cause generation of all glyphs up to and including those associated with the specified character.  It will return the glyph index for the first glyph associated with the character at the specified index.
- (NSUInteger)glyphIndexForCharacterAtIndex:(NSUInteger)charIndex;
```
3).  绘图类, 按照图形布局信息，绘制文字
``` 
/************************ Drawing support ************************/
/* These methods are primitives for drawing.  
You can override these to perform additional drawing, or to replace text drawing entirely, but not to change layout.  
You can call them if you want, but focus must already be locked on the destination view or image.  
-drawBackgroundForGlyphRange:atPoint: draws the background color and selection and marked range aspects of the text display, 
along with block decoration such as table backgrounds and borders.  
-drawGlyphsForGlyphRange:atPoint: draws the actual glyphs, including attachments, as well as any underlines or strikethroughs.  
In either case all of the specified glyphs must lie in a single container.
*/
- (void)drawBackgroundForGlyphRange:(NSRange)glyphsToShow atPoint:(CGPoint)origin;
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(CGPoint)origin;
```


3. ##### NSTextContainer
该类是辅助类，辅助view生成盛放NSLayoutManager类生成的图形。
如：
*NSTextview*中的
```
// Get the text container for the text view
@property(nonatomic,readonly) NSTextContainer *textContainer NS_AVAILABLE_IOS(7_0);
```
该类可以设置以下属性：
*size*：展示区域的大小
*exclusionPaths*:排除展示区域的路径，是一个内部盛放UIBezierPath对象的数组
*lineBreakMode*: 换行模式
*lineFragmentPadding*：线段边距

以及提供了一个方法，来设置以上提及的一些属性
```
/**************************** Line fragments ****************************/

/* Returns the bounds of a line fragment rect inside the receiver for proposedRect.  This is the intersection of proposedRect and the receiver's bounding rect defined by -size property.  
The regions defined by -exclusionPaths property are excluded from the return value.  charIndex is the character location inside the text storage for the line fragment being processed.  
It is possible that proposedRect can be divided into multiple line fragments due to exclusion paths.  
In that case, remainingRect returns the remainder that can be passed in as the proposed rect for the next iteration.  
baseWritingDirection determines the direction of advancement for line fragments inside a visual horizontal line.  
The values passed into the method are either NSWritingDirectionLeftToRight or NSWritingDirectionRightToLeft.  This method can be overridden by subclasses for further text container region customization.
*/
- (CGRect)lineFragmentRectForProposedRect:(CGRect)proposedRect atIndex:(NSUInteger)characterIndex writingDirection:(NSWritingDirection)baseWritingDirection remainingRect:(nullable CGRect *)remainingRect NS_AVAILABLE(10_11, 7_0);
```
**我们可以通过创建NSTextContainer的子类，然后重写该方法，来定义那些文字展示在不规则图形中的情况**
![](https://upload-images.jianshu.io/upload_images/1241385-0a3a4812bcf6e724.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总结一下；整体的绘制流程如下：
![](https://upload-images.jianshu.io/upload_images/1241385-c34ef654b8a28822.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 有以下几种应用场景：
1. 语法高亮

1. 类似markdown编辑器。

2. 多列展示文章

3. 图文混排

4. label上文字点击




[Using Text Kit to Draw and Manage Text](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/CustomTextProcessing/CustomTextProcessing.html#//apple_ref/doc/uid/TP40009542-CH4-SW1)
[TextKit Best Practices WWDC 2018](https://developer.apple.com/videos/play/wwdc2018/221/)
[TextKit Best Practices WWDC 2018中文图解](https://juejin.im/post/5b27451a51882574eb597e04)
[Bullet list on iOS with TextKit](https://blog.vinhis.me/2017/04/16/bullet-list-on-ios-with-textkit.html)
[Getting to Know TextKit](https://www.objc.io/issues/5-ios7/getting-to-know-textkit/)
[String Rendering](https://www.objc.io/issues/9-strings/string-rendering/)
[How to fit text in a circle in UILabel](https://stackoverflow.com/questions/22178773/how-to-fit-text-in-a-circle-in-uilabel)
