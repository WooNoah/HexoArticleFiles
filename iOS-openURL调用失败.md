---
title: iOS openURL调用失败
date: 2018-11-22 17:21:35
tags:
---

> 今天改代码的时候，遇到一个问题，在APP内部跳转到Safari打开某个链接的时候，竟然报错了。然后经过一番试错，终于找到了问题所在！记录一下，以备以后方便查找！

先贴上源代码：
```
/**
* 调用Safari加载URL
*/
- (void)openScheme:(NSString *)urlString {
UIApplication *application = [UIApplication sharedApplication];
NSURL *URL = [NSURL URLWithString:urlString];

if ([application respondsToSelector:@selector(openURL:options:completionHandler:)]) {
[application openURL:URL options:@{} completionHandler:^(BOOL success) {
LOG(@"Open %@: %d",urlString,success);
}];
} else {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
//过期的方法
BOOL success = [application openURL:URL];
#pragma clang diagnostic pop
LOG(@"Open %@: %d",urlString,success);
}
}
```
传入的参数为：`www.gsfxuk.cn/index.html`
从逻辑来看，没什么问题，而且最奇怪的，在另外一个项目中，同样的代码，是没有问题的。
#### 这里罗列一下思考是几个方面：
1. URL问题
打断点看了一下生成的URL，是有值的。`排除！`
引申：
```
以前遇到一个问题：
后台在录入数据的时候，手滑在URL的结尾多加了一个空格，eg: www.baidu.com ,
就因为这个空格，就会造成无法生成正确的URL对象。（NSURL对象为nil）
```
应对此种问题的解决方法：
```
NSString *validStr = [transferStr stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]];
```
***此时得到的`validStr`即为不带空格的字符串！***
2. 新方法存在不兼容问题
根据苹果的习惯，新方法是有版本要求的，但是出了新方法之后，老的方法也是一样可行的，所以我就先调用一下老的方法，排除一下。结果！`不行！`排除此猜想
3. 方法传参问题
接着看一下新方法的入参，即多了的那个`options`
该参数解释如下：引用自[iOS10 新变化之废弃的 openURL](https://juejin.im/entry/57e8c5e28ac247005bd90dcc)
> options目前可传入参数Key在UIApplication头文件只有一个:UIApplicationOpenURLOptionUniversalLinksOnly,其对应的Value为布尔值,默认为False.如该Key对应的Value为True,那么打开所传入的Universal Link时,只允许通过这个Link所代表的iOS应用跳转的方式打开这个链接,否则就会返回success为false,也就是说只有安装了Link所对应的App的情况下才能打开这个Universal Link,而不是通过启动Safari方式打开这个Link的代表的网站.

**一句话总结一下：**
如果传一个空字典的情况（即不传此值，也即默认值），如果传入的scheme在本地能查询的到，调用本地APP，如果本地匹配不到，则调用Safari打开。
如果传入的不是一个空字典（即`@{UIApplicationOpenURLOptionUniversalLinksOnly: @YES}`）,此时当且仅当**URL有效、且本地装了对应APP**的时候，才能打开。(本人猜测是要直接用APP调起某个模块的时候这么写)<本人没遇到过这种情况，所以无法验证，如果说错，请各位大佬批评指正！谢谢！！>

说到scheme，就必须介绍一下苹果的这个应用间跳转的技术了，见[引申1]()

#### 引申
1. scheme [摘自-简书谦言忘语](https://www.jianshu.com/p/0811ccd6a65d)
##### 什么是URL Schemes？
URL Schemes是苹果给出的用来跳转到系统应用或者跳转到别人的应用的一种机制。同时还可以在应用之间传数据。

> 通过对比网页链接来理解 iOS 上的 URL Schemes，应该就容易多了。
URL Schemes 有两个单词：
*URL，我们都很清楚，[http://www.apple.com](http://www.apple.com)
就是个 URL，我们也叫它链接或网址；
*Schemes，表示的是一个 URL 中的一个位置——最初始的位置，即 ://
之前的那段字符。比如 [http://www.apple.com](http://www.apple.com)
这个网址的 Schemes是 **http**。
根据我们上面对 URL Schemes 的使用，我们可以很轻易地理解，在以本地应用为主的 iOS 上，我们可以像定位一个网页一样，用一种特殊的 URL 来定位一个应用甚至应用里某个具体的功能。而定位这个应用的，就应该是这个应用的 URL 的 Schemes 部分，也就是开头儿那部分。

![](https://upload-images.jianshu.io/upload_images/1241385-1d23885f28e34ed8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们在项目的`info`->`URL Types`中设置自己的`URL Schemes`，就等于是在手机中注册了一个域名，然后别的应用或者Safari，WKWebview组件就可以通过访问该scheme已达到调起APP的作用。
以下写出调起APP的代码：
```
NSURL *url = [NSURL URLWithString:@"weixin://"];
[[UIApplication sharedApplication]openURL:url options:@{} completionHandler:^(BOOL success) {
NSLog(@"success:%@",@(success));
}];
```
这么写之后，项目中就会提示*是否要打开微信APP*了
这里也可以理解友盟等第三方登录分享之类的，是在后台配置了应用的独立标识，然后让开发者手动配置在要集成的项目scheme这里，已达到监控统计调起应用，然后再返回的功能。
######但是思考一点：
**如果我们又在手机上安装了一个APP，其中配置了相同的`weixin`scheme，此时会怎么跳转呢？**
> 暂无确定结果！

但是可以确定的是：不同的项目是可以有相同的scheme的。
此时，我们可以在调用openURL方法之前，加一个逻辑完善：
`-canOpenURL`, 即先判断是否能打开，然后再执行！

因此，这里也排除了scheme的配置问题。

4. 因为刚刚思考了scheme的问题，然后想到了类似域名的解释。然后此时灵光一闪：刚刚的入参为`www.gsfxuk.cn/index.html`，并没有`http://`或者`https://`，那么此时传给Safari的时候，它肯定也不知道是个外链啊！！然后我又在别的项目中打印了URL，看到都是包含`http://`或者`https://`的。
至此，问题定位完成！！

#### 解决：
定位好问题之后，就好解决了。我们只需要判断下传入的URL连接是否包含`http://`或`https://`，如果不包含，则给它添加上即可！！
```
/**
* 把string转换成可以URL跳转的类型（指的是带HTTP或者HTTPS）
*/
- (NSString *)convertToURLFormatWithString:(NSString *)str {
if ([str containsString:@"http"] || [str containsString:@"https"]) {
return str;
}else {
NSMutableString *strM = [NSMutableString stringWithString:str];
[strM insertString:@"http://" atIndex:0];
return strM.copy;
}
}
```
#### 参考
[Support Universal Links](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW2)
[Querying URL Schemes with canOpenURL](https://useyourloaf.com/blog/querying-url-schemes-with-canopenurl/)
[uiapplicationopenurloptionuniversallinksonly参数解释](https://developer.apple.com/documentation/uikit/uiapplicationopenurloptionuniversallinksonly?language=objc)
[openURL: deprecated in iOS 10](https://stackoverflow.com/questions/39548010/openurl-deprecated-in-ios-10)
[How can I programmatically determine if a URL is a Universal Link or just a regular web URL?](https://stackoverflow.com/questions/41579374/how-can-i-programmatically-determine-if-a-url-is-a-universal-link-or-just-a-regu)
