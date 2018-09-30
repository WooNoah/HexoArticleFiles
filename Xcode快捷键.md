---
title: Xcode快捷键
date: 2018-08-17 12:00:39
tags: iOS开发
---

> 记录一下xcode的快捷键，以供开发的时候提高效率

#### 左上角区域
从左到右依次为：
![](https://upload-images.jianshu.io/upload_images/1241385-5b1f1c9de4bff5d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

|解释|项目文件索引useful|版本控制管理|符号导航|全局搜索替换useful|问题索引|测试索引|调试板块useful|断点useful|报告|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|快捷键|`cmd+1`|`cmd+2`|`cmd+3`|`cmd+4`|`cmd+5`|`cmd+6`|`cmd+7` | `cmd+8` | `cmd+9` |
`<useful>`标记的为常用的几项
另外：
搜索快捷键：`cmd+shift+f`

<!--more-->
#### 右上角区域
| 图标 | ![](https://upload-images.jianshu.io/upload_images/1241385-59fa01552386cf01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) | ![](https://upload-images.jianshu.io/upload_images/1241385-a3a57499c243b1b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) | ![](https://upload-images.jianshu.io/upload_images/1241385-9318df99c8551cd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) | ![](https://upload-images.jianshu.io/upload_images/1241385-ba9281dd5835a7cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) | ![](https://upload-images.jianshu.io/upload_images/1241385-f420a2eacffadc19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) |
| :-----:| :-----: | :-----: | :-----: | :-----: | :-----: |
| 解释 | 显示标准编辑框 | 显示辅助编辑框 | 展开/隐藏左侧导航区 | 展开/隐藏下部debug区 | 显示工具 |
| 快捷键 | `cmd+Enter` | `cmd+option+Enter` | `cmd+0` | `cmd+shift+y` | `cmd+option+0` |

#### 程序调试区
断点调试时候的一些操作：
![](https://upload-images.jianshu.io/upload_images/1241385-42cd9b32815e9ae6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 一些按键组合
1. 清空debug栏快捷键：`cmd+k`
![debug栏](https://upload-images.jianshu.io/upload_images/1241385-f75ab89ce70cff41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 使用storyboard或者xib的时候，更新约束： `cmd+shift+=`
3. 调用instruments工具：`cmd+i`
![instruments工具栏](https://upload-images.jianshu.io/upload_images/1241385-51c9042a3e9cd143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4. `cmd+shift+k`
清理项目
5. 深度清理：`cmd+shift+option+k`
![](https://upload-images.jianshu.io/upload_images/1241385-00433f1656f7dc29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此操作会清理所有产品和中间文件
6. 打开指定文件到指定位置`cmd+shift+option+鼠标点击指定文件`
![](https://upload-images.jianshu.io/upload_images/1241385-5999aa54dc813a67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 后边加号，在新的窗口中打开
- 右边加好，在辅助窗口中打开
![](https://upload-images.jianshu.io/upload_images/1241385-53ab18bbde9c8dda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
7. 跳转到当前文件在导航栏中的位置： `shift+cmd+j`
8. 使xcode指定区域取得焦点: `cmd+j`
![](https://upload-images.jianshu.io/upload_images/1241385-e5fc54c364edc8f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
9. `Utilities（cmd+option+0）`层级下的副级按钮：
![](https://upload-images.jianshu.io/upload_images/1241385-4078c5997806d5dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

| 操作 |左侧|右侧|
| :-: | :-: | :-: |
| 解释 | 显示文件检查 | 显示帮助 |
| 快捷键 | `option+cmd+1` | `option+cmd+2` |

10. `cmd+\` : 在当前行添加（删除）断点
`cmd+Y` : 激活（取消激活）全部断点

11. 显示方法调用栈
```
- (void)viewDidLoad {
[super viewDidLoad];

self.arr = @[@"1",@"2",@"3",@"4",@"5",@"6",@"7"];

[self testMethod];
}

- (void)testMethod {
NSLog(@"%@",self.arr);
}
```
如上面代码：
`ctrl+1`调出界面：可以看到**调用该方法的位置**以及**该方法中别的方法的调用栈**
![调用该方法的位置](https://upload-images.jianshu.io/upload_images/1241385-b999841e4051a1b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![该方法中别的方法的调用栈](https://upload-images.jianshu.io/upload_images/1241385-941843fe74101f89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

12. 到指定行号：
![](https://upload-images.jianshu.io/upload_images/1241385-b48a960e0180d398.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
13. `cmd+option+L`跳转到光标所在行



[参考资料](http://www.cocoachina.com/ios/20141225/10761.html)
[参考资料2](https://juejin.im/post/5a3b1bf8f265da432f314788)
