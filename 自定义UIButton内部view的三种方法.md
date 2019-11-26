---
title: 自定义UIButton内部view的三种方法
date: 2019-11-26 12:17:52
tags:
---

有三种方式可以自定义UIButton内部的imageView、titleLabel的位置
 1. 使用系统提供的方法：
    setImageEdgeInsets
    setTitleEdgeInsets
 
 2. 在layoutSubviews方法中。手动修改imageView、titleLabel控件的frame
    如下 part ①中的代码
 
 3. 想要使用Masonry自己处理约束的话，那就得先设置为不让系统自动处理约束
    self.imageView.translatesAutoresizingMaskIntoConstraints = NO;
    self.titleLabel.translatesAutoresizingMaskIntoConstraints = NO;
    然后使用masonry添加约束即可
    
    但是遇到一个问题此处会有一个问题：
    imageView默认是居中的
    如果只添加设置约束方法，但是不添加任何约束
     [self.imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        
     }];
    此时则是图片原始大小，然后居中显示
![](https://upload-images.jianshu.io/upload_images/1241385-88522492ac3e00e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
而且可以看到，虽然我们设置了不让系统处理约束，但是`此处ImageView仍然还是有约束的`
而且，我试着删除系统的约束`[self.imageView removeConstraints:self.imageView.constraints]`之后重新设置，但是**并没有什么效果**
但是想要通过
        make.size.mas_equalTo(self.imageView.image.size);
    调整约束的话，是无效的。
![](https://upload-images.jianshu.io/upload_images/1241385-0e666057d21cadc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **只能重新修改已有的约束来达到效果**
`此处是最优解`
```
    [self.imageView removeConstraints:self.imageView.constraints];
    [self.imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        //1 这是系统给的四个约束，想要调整，需要给原有的约束添加新的值
        make.centerX.equalTo(self);
        make.centerY.equalTo(self).offset(-10);
        make.width.equalTo(@(self.imageView.image.size.width));
        make.height.equalTo(@(self.imageView.image.size.height));
        
        //2
//        make.top.equalTo(self).offset(10);
//        make.bottom.equalTo(self).offset(-60);
//        make.left.equalTo(self).offset(20);
//        make.right.equalTo(self).offset(-50);
                
//          //这里通过设置size是无效的，只能设置width/height
//        make.size.mas_equalTo(self.imageView.image.size);
    }];
```
![](https://upload-images.jianshu.io/upload_images/1241385-e9e51145b364a07c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
也可以调整上下左右的位置来调整达到类似的效果
```
[self.imageView removeConstraints:self.imageView.constraints];
    [self.imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        //1 这是系统给的四个约束，想要调整，需要给原有的约束添加新的值
//        make.centerX.equalTo(self);
//        make.centerY.equalTo(self).offset(-10);
//        make.width.equalTo(@(self.imageView.image.size.width));
//        make.height.equalTo(@(self.imageView.image.size.height));
        
        //2
        make.top.equalTo(self).offset(10);
        make.bottom.equalTo(self).offset(-60);
        make.left.equalTo(self).offset(20);
        make.right.equalTo(self).offset(-50);
                
//          //这里通过设置size是无效的，只能设置width/height
//        make.size.mas_equalTo(self.imageView.image.size);
    }];
```
![](https://upload-images.jianshu.io/upload_images/1241385-d220b6746f3dfb1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



[demo地址](https://github.com/WooNoah/CustomUIButton)
