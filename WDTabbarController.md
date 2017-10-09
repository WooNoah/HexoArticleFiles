---
title: WDTabbarController
date: 2017-03-31 15:07:58
tags:
---


#### 前言
最近改页面，要实现一个用户分步完善信息的页面，印象中以前在别的应用中见过类似的设计，美团或者是什么的，本来想着在网上找找类似的，改下就行了，然后，找了半天，发现并没有类似的，没办法，只能自己来了
#### 实现
先上效果图吧，如果您看符合您的需求，那么您可以参考下，如果不符合，但是也比较感兴趣，也可以帮忙给我瞅瞅有没有BUG，谢谢
![这里写图片描述](http://img.blog.csdn.net/20170331145731116?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3d3d3d3d3d3d3d3ZGk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<!--more-->

**有几点特殊的需求**
1.	用户没有完善前面的信息，那么后边的就不能点，只能查看用户完善过的后一个页面的情况
2.	用户完善信息之后，可以回到以前的页面，然后重新修改提交信息

#####	使用中也有几点要求
1.	子类必须继承自`WDTabbarSubController`
2.	子类frame问题，目前版本只能将`导航栏高度`和`页面上半部分`高度减去，（如果有大神有别的好的思路，欢迎指点）

废话不多说了，直接[Github](https://github.com/WooNoah/WDTabbarController)吧,
***如果您感觉对您有帮助，麻烦给你star,谢谢***
