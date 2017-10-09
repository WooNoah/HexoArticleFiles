---
title: iOS导航栏pop的时候出现短暂蓝色
date: 2017-03-14 10:30:35
tags:
---

#### 起因
今天看老的项目，突然发现一个问题：
> 从页面①push到页面②，pop回页面①的时候，发现导航栏上面有那么一闪而逝的蓝色。


#### 解决
<!--more-->
然后我就开始考虑这个问题产生的原因：
1. 首先，我检查了页面①的导航栏设置。
我在该页面是把导航栏设置为透明的
```
- (void)viewWillAppear:(BOOL)animated {
[super viewWillAppear:animated];
[self.navigationController.navigationBar setBackgroundImage:[[UIImage alloc] init] forBarMetrics:UIBarMetricsDefault];
[self.navigationController.navigationBar setShadowImage:[[UIImage alloc] init]];
}

- (void)viewWillDisappear:(BOOL)animated {
[super viewWillDisappear:animated];
[self.navigationController.navigationBar setBackgroundImage:[UIImage imageNamed:@"默认导航栏图片"] forBarMetrics:UIBarMetricsDefault];

}
```

然后，我就想着，要不就在`viewWillAppear`中把导航栏隐藏下，等`viewWillDisappear`中再把导航栏显示出来。大不了就重新添加些label,button来模拟一个导航栏出来。
然后我就试了下，发现，并没有什么卵用！

--------------

- 接着，我发现同一个属于页面①下面的子页面，进去再pop出来的时候，并没有那个蓝色。
然后，我就到页面②中去寻找问题。
经过对比，我发现，页面②跟同层级的别的Controller相比，少了些代码。
***原来，我在别的Controller的viewWillAppear中都手动设置了导航栏的！***

*引人深思：*
这个项目是以前写的，当时没有设置基类，所以才导致有些页面忘记设置导航栏属性。
***tip：***
> 以后各位再构建新的工程的时候，多使用基类，方便省事！！！

原来问题在这里！接着我就在页面②的`viewWillAppear`中添加了相关代码。问题解决！

--------------

- 发散思维（说白了就是好奇心）驱使下，
我在页面②`viewWillAppear`中设置过后，重新在`页面①中设置导航栏hidden`，
发现，***也是不行的***

--------------

- 我想到了我在网上查询的别人的问题以及解决方法[解决 iOS View Controller Push/Pop 时的黑影](https://imtx.me/archives/1933.html)

他的说法，`如果这个 ViewController 是在 TabBarViewController 的 NavigationController 上 Push/Pop 的，那么只需要把 TabBarViewController 的 View 设置一下白色背景就可以了`, 我找到TabbarController，设置背景色，`没卵用！！！`

--------------

- 同样的，我也搜到了别的说是根windows没有设置背景色的，然后，`仍然没用`。

--------------

- 如果有大神能看到，请给我讲解下这个问题！谢谢！！

--------------
#### PS
说道上面的文章，他有说道一个问题，也是我遇到的
```
iOS 的 UITableViewCell 有一个很严重的问题，是 7.x 某个版本以后引起的，如果给 detailTextLabel.text 设置 nil 或者 ""，再设置具体的 text 后有时会显示不出来，但 Tap 一下能出来。具体的解决办法是，不要设置 nil 或 ""，设置 " " （中间有空格）。真是让人吐血的 Workaround。
```
当时的设计是这样的：
![这里写图片描述](http://img.blog.csdn.net/20170314151804156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3d3d3d3d3d3d3d3ZGk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后设置头像那里，我想着偷下懒: 设置cell的`detailTextLabel为@“”`，再添加一个ImageView，然后添加约束，就可以了。

然后，在查看UI的时候，发现头像"飞了"，然后我各种调试，才发现是detailTextLabel不显示，因此，我也使用这种比较拙略的解决方法：
```
cell.detailTextLabel.text = @" ";
[cell addSubview:self.userIconImgView];
//这里是Masonry代码
[self.userIconImgView mas_makeConstraints:^(MASConstraintMaker *make) {
	make.centerY.equalTo(cell);
	make.right.equalTo(cell.extraLabel.mas_right);
    make.size.mas_equalTo(CGSizeMake(60, 60));
}];
```
