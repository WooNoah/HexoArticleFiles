---
title: React-Native之路上遇到的问题
date: 2017-08-09 10:54:24
tags:
---

> 最近要研究下React Native，考虑使用RN来写一个安卓端。本篇文章记录一下操作中遇到的各种问题：

### RN基本命令行
`react-native init 项目名` 初始化创建项目
`react-native run-ios/android` 命令行启动app
`react-native --version`    查看RN版本
`npm info react-native`    查看RN远程仓库所有版本信息
`npm update -g react-native-cli`    更新本地RN版本
`npm install --save react-native@版本号` 更新RN到指定版本

<!--more-->
### 插件
[WebstormRN代码提示插件](https://github.com/virtoolswebplayer/ReactNative-LiveTemplate)

### 遇到的问题
**默认安装过Android Studio [virtualbox](https://www.virtualbox.org/)和[Genymotion模拟器](https://www.genymotion.com/download)**
#### 1、Android Studio找不到Genymotion模拟器
[解决方法](http://blog.csdn.net/vcstrong/article/details/40590357)
打开Android Studio首页面，SDK位置：
`configure -> Project Defaults -> Project Structure`

#### 2、unable to load script from assets index.android.bundle on windows

`react-native init xxx`之后，打开Android studio，点击`运行`之后报的这个错。
解决方法：
1. `(in project directory) mkdir android/app/src/main/assets`
2. ```
react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
```
3. `react-native run-android` 该项如果没有安装过JDK则会提示安装，[点击下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，安装之后即可使用。

#### 3. 使用npm导入react库之后，提示command run-ios unrecognized
我的理解： 使用npm导入三方库之后，由于module树中没有刚添加的三方库索引，所以，**需要重新更新一下module树： `npm install`**。

#### 4. 定时器问题
我在学习写demo的时候，是用的ES5的语法来写的，然后demo中需要使用一个定时器，所以我就调用了React提供的库`react-timer-mixin`。但是使用的时候，调用`this.setInterval()`的时候，报错了：
![错误图片](http://upload-images.jianshu.io/upload_images/1241385-d4683ec12fae4528.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后我查了下，说是mixin不支持ES6语法*（但是我用的是ES5的写法，所以我查了半天也不知道哪里的问题，有知道的大神请指教下）*

然后处理方法：（不使用Mixin）***但是在es6中使用时，需铭记在unmount组件时清除（clearTimeout/clearInterval）所有用到的定时器***
```
componentDidMount() {
this.timer = setInterval(function(){
console.log('每隔1s执行一次的方法');
}，1000);

this.timer = setTimeout(() => {
console.log('I do not leak!');
}, 5000);
}

componentWillUnmount() {
clearTimeout(this.timer);
clearTimeInterval(this.timer);
}
```

#### 5. 运行react-native run-ios之后提示xcrun: error: unable to find utility "instruments", not a developer tool or in PATH
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1241385-383bc389b41ca515?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6. 新建RN项目之后，使用Android studio打开安卓项目，遇到unable to load script from assets index.android.bundle问题
解决方法：
`1. cd 到RN项目根目录`
`2. mkdir android/app/src/main/assets`
`3. react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res`
`4. react-native run-android`



#### 7. 新建项目之后，运行`react-native run-ios`一直卡在`build xxx: double-conversion`这里（2018年12月21日，RN version 0.57.8）
新建的React Native 项目，凡是版本号大于0.45的，iOS版本在build的时候会卡在build double conversion 这里。

**找到的网上的解决方法如下：**
==》解决方案在RN中文网有：[http://reactnative.cn/post/4301](http://reactnative.cn/post/4301)

简单来说，0.45后的RN项目，会依赖一些三方库，然而在国内这些库很难下载到，翻墙也很难下到。

操作步骤：

1、查询具体需要的三方库：[https://github.com/facebook/react-native/blob/master/scripts/ios-install-third-party.sh](https://github.com/facebook/react-native/blob/master/scripts/ios-install-third-party.sh)

2、有好人下好了放在百度云盘里了：[https://pan.baidu.com/s/1zvNLcj-TGRDNfzymh2r30A](https://pan.baidu.com/s/1zvNLcj-TGRDNfzymh2r30A)

3、下下来后，把这些压缩文件放到 ~/.rncache（Mac操作：Finder=》前往文件夹=》~/.rncache)

4、然后就可以开始撸码了


--------

还找了一些别的解决方法，如下：

1. 此问题是react-native 0.44版本之后出现的问题。所以我们强制把版本锁定在0.44
新建项目可以执行如下指令
```
react-native init [PROJECT_NAME] --version 0.44.0
```
已经创建过后，那么使用`npm`来重新安装指定版本的react-native即可
> There is an issue with 0.45\. Like [@llanginger](https://github.com/llanginger) said, after downgrading
`npm install react-native@0.44.0` If you are using RN 0.44, you'd better keep it as below:
`"react": "16.0.0-alpha.6", "react-native": "0.44.0",`
works well.

2. 也有说删除`rncache`文件夹，nodeModule，然后重新使用`yarn`安装一遍就可以了
> I solved this problem by deleting .rncache folder within the home directory, deleting node modules and then yarn install, and works like a charm.

https://github.com/facebook/react-native/issues/14368
https://github.com/facebook/react-native/commit/5c53f89dd86160301feee024bce4ce0c89e8c187#commitcomment-23014602
