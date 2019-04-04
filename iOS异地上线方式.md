---
title: iOS异地上线方式
date: 2019-04-04 11:44:29
tags:
---


> iOS异地上线是什么意思，都有哪些情况？

#### 异地上线，大概就是外包给一个公司开发项目，然后在**拿不到源码的情况下**，我们使用自己公司的苹果开发者账号来上线

大致上分为几种思路：

##### 1. 苹果开发者账号密码给外包公司
这种方法。。。。emmmmmm，自行体会。。。

##### 2. 使用苹果公司提供的授权方式
[苹果开发者](https://developer.apple.com)登录账号之后，有个member栏目，
**然后把别的个人账户添加到要上线的公司账户的组织下面**，
此时在xcode中账户设置里面输入个人账户，应该就可以正常上传应用了。

##### 3. 分享p12和mobileProvision文件（只是思路，不确定可不可行）
###### 分两步：1.先创建p12证书
先在`钥匙串`->`证书助手`->`从证书机构请求证书`
![](https://upload-images.jianshu.io/upload_images/1241385-57765779ad8e6d35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
请求完成之后，下载，然后就可以在钥匙串中找到了
然后在想要生成p12文件的证书上，`右键导出` 
![](https://upload-images.jianshu.io/upload_images/1241385-f2a5e47b3ec6fa2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后输入要保存的名字，以及路径
![](https://upload-images.jianshu.io/upload_images/1241385-2366f7d401e09eb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到文件后缀名为`p12`类型

###### 2.然后创建profile provision(配置文件)文件
在[苹果开发者](https://developer.apple.com)的`证书管理`里边，新建配置文件
（开发和发布的要分开）

创建完成这两个文件之后，就可以把他们发给开发人员了。
只需要双击文件之后，这些配置就会被存到`钥匙串`中。
然后在Xcode中就可以选择`Profile Provisioning`文件中对应的team了（我的猜想）

[https://support.mobincube.com/hc/en-us/articles/200511933-How-to-get-the-p12-file-and-provisioning-profile-for-publishing-an-app-on-App-Store](https://support.mobincube.com/hc/en-us/articles/200511933-How-to-get-the-p12-file-and-provisioning-profile-for-publishing-an-app-on-App-Store)

##### 4. IPA包重签名
[GitHub上有第三方工具 ios-app-signer](https://github.com/WooNoah/ios-app-signer)
他是一个Mac APP，下载之后运行，就可以看到以下界面
![](https://upload-images.jianshu.io/upload_images/1241385-b8cafc92a9e92c0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**这里要注意几点：**
**前三项是必选的！！！**
***选对`签名证书`(Signing Certificate) 和 `配置文件`(Profile Provisioning)***
**`配置文件`(Profile Provisioning)**可能需要先按照上面[3.2]()的步骤生成一个，然后再选择
