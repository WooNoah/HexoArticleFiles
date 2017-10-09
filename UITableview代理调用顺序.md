---
title: UITableview代理调用顺序
date: 2017-04-13 16:07:58
tags:
---

> 最近在自定义tableviewCell的时候，遇到了一些问题，这里简单记录一下。

####	Situations
首先我自定义TableviewCell, 在cell上设置了一些textfield, 然后在Controller中设置一个textfield实例变量来持有相对应的cell上的输入框。然后，我想在`viewWillAppear`中给对应的textfield设置placeholder，然后，我发现，***并没有什么卵用!!!***

然后我仔细查看了一下，发现：`在viewWillAppear的时候，self.nameTextfield = nil`。说明此时：赋值操作还没有执行，
<!--more-->

然后我就查看了下Tableview代理的调用方法
```
2017-04-13 16:19:11.520  view will appear
2017-04-13 16:19:11.566  cell for row at index path
2017-04-13 16:19:11.578  cell for row at index path
2017-04-13 16:19:11.580  cell for row at index path
2017-04-13 16:19:11.583  cell for row at index path
2017-04-13 16:19:11.584  viewForHeaderInSection
```
这里是tableview有四个cell，所以`cellForRowAtIndex方法调用了4次`
另外！
**viewWillAppear方法是在cell初始化之前的，最后才是tableviewHeader或者Footer初始化**

然后，我以为这是由于是自定义cell，所以调用顺序受到了影响，然后我使用iOS系统的cell,发现：**跟上边自定义的调用顺序是一样的！**

####	Solutions
这里我说一下我自己的愚见：
1.	我们可以给Controller的textfield使用懒加载，然后不持有cell上输入框对象，改为 `[cell addSubview: self.controllerTextfield];` 这么一来，也避免了在复用的时候cellTextfield上文字位置错乱的问题。
2.	我们可以在`viewDidAppear`方法中才去相对应的操作
```
2017-04-13 16:40:30.184  view will appear
2017-04-13 16:40:30.249  cell for row at index path
2017-04-13 16:40:30.266  cell for row at index path
2017-04-13 16:40:30.267  cell for row at index path
2017-04-13 16:40:30.268  cell for row at index path
2017-04-13 16:40:30.269  viewForHeaderInSection
2017-04-13 16:40:30.277  view did appear
```
由上面的顺序可见，`viewDiDAppear`调用的时候，cell已经初始化了，持有操作已经进行过了，那么，我们也可以进行相对应的操作了


***抛砖引玉，如果大家有别的想法，还请多多指教！***
