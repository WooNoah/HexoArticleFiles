---
title: WKWebview禁止点击文字或空白部分滚动功能
date: 2019-03-20 13:51:58
tags:
---


WKWebview点击文字或者空白区域，如果页面够长，则会发生滚动
两种思路：
#### 1. 在iOS内部修改
> 思路： 1）在WKWebview中寻找是否有相应的属性来修改  //结果并没有
2）捕获到webview的双击事件，然后进行拦截

查看苹果api可以看到，WKWebview 中有一个scrollview，scrollview中有几个手势UIPanGestureRecognizer, UIPinchGestureRecognizer等等，但是并没有UITapGestureRecognizer, 而且，苹果在api中详细的说到：**不能修改scrollview的代理，也不能重写gestureRecognizer属性的getter方法**
```
// Use these accessors to configure the scroll view's built-in gesture recognizers.
// Do not change the gestures' delegates or override the getters for these properties.

// Change `panGestureRecognizer.allowedTouchTypes` to limit scrolling to a particular set of touch types.
@property(nonatomic, readonly) UIPanGestureRecognizer *panGestureRecognizer NS_AVAILABLE_IOS(5_0);
// `pinchGestureRecognizer` will return nil when zooming is disabled.
@property(nullable, nonatomic, readonly) UIPinchGestureRecognizer *pinchGestureRecognizer NS_AVAILABLE_IOS(5_0);
```
所以可以在WKWebveiw的子类scrollview中找tap事件，然后找到之后，删除他
```swift
// iterate over all subviews of the WKWebView's scrollView
for subview in _webView.scrollView.subviews {

// iterate over recognizers of subview
for recognizer in subview.gestureRecognizers ?? [] {

// check the recognizer is  a UITapGestureRecognizer
if recognizer.isKind(of: UITapGestureRecognizer.self) {

// cast the UIGestureRecognizer as UITapGestureRecognizer
let tapRecognizer = recognizer as! UITapGestureRecognizer

// check if it is a 1-finger double-tap
if tapRecognizer.numberOfTapsRequired == 2 && tapRecognizer.numberOfTouchesRequired == 1 {

// remove the recognizer
subview.removeGestureRecognizer(recognizer)
}
}
}
}
```


#### 2. 然后可以考虑JS端修改
> 大概思路是这样： JS中实现双击事件，然后以达到override iOS 系统webview中的双击事件，达到阻拦的效果
```
(function()
{
var agent = navigator.userAgent.toLowerCase(); //检测是否是ios
var iLastTouch = null; //缓存上一次tap的时间
if (agent.indexOf('iphone') >= 0 || agent.indexOf('ipad') >= 0)
{
document.body.addEventListener('touchend', function(event)
{
var iNow = new Date()
.getTime();
iLastTouch = iLastTouch || iNow + 1 /** 第一次时将iLastTouch设为当前时间+1 */ ;
var delta = iNow - iLastTouch;
if (delta < 500 && delta > 0)
{
event.preventDefault();
return false;
}
iLastTouch = iNow;
}, false);
}

})();
```

[http://lydia-blog.logdown.com/posts/7355883-ios-wkwebview-page-double-click-will-move-up](http://lydia-blog.logdown.com/posts/7355883-ios-wkwebview-page-double-click-will-move-up)
[http://www.voidcn.com/article/p-wememcnl-buq.html](http://www.voidcn.com/article/p-wememcnl-buq.html)
