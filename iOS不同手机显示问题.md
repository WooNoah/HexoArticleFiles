---
title: iOS不同手机显示问题
date: 2018-12-10 15:58:54
tags:
---


今天遇到一个问题：设置一个view的高度，在不同手机下显示不一样：直接贴代码：
```
- (UIView *)line1 {
if (!_line1) {
_line1 = [[UIView alloc]init];
}
return _line1;
}

- (UIView *)line3 {
if (!_line3) {
_line3 = [[UIView alloc]init];
}
return _line3;
}


-(void)setLines{

/**
----------line1--------------
|line2
----------line3--------------
**/

[self addSubview:self.line1];
[self addSubview:self.line3];

[self.line1 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.top.right.equalTo(self);
make.height.equalTo(@(0.5));
}];

[self.line3 mas_makeConstraints:^(MASConstraintMaker *make) {
make.left.right.equalTo(self);
make.bottom.equalTo(self).offset(-0.25);
make.height.equalTo(self.line1);
}];
}

```
代码很简单，一目了然！
两条线按理说应该是一样的高度的。但是！！！
下面来看下结果！
```
//设置为0.33 6s：上下0.5px  max：上下都为0.33
//设置为0.5  6s：上下都为0.5px, max：上0.67，下0.33
//设置为0.44  6s：上下0.5px max：上下0.33
//设置为0.66  6s：上0.5px 下1px max：上下都为0.66px
//设置为0.66  6s：上下都为1px max：上下都为1px
```
我猜这个是跟屏幕的分辨率有关系，待以后研究下再来补充。
这里取折中方案，***先设置为0.33***


