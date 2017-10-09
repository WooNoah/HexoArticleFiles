---
title: iOS接入微信支付问题
date: 2017-10-09 14:00:01
tags:
---


>  首先记录下接入时间： 2017年09月13日

#### 过程：
由于项目中要接入微信支付，所以我就先打开了[微信支付开发文档](https://pay.weixin.qq.com/wiki/doc/api/index.html), 毕竟知己知彼，百战不殆啊。大概熟悉了一下流程之后，我们就可以开始[下载demo](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=11_1)了 **我这里接入的是APP支付，只有iOS和Android两个版本的，其他的H5支付和公众号支付，均是由后台接入！**

#### 问题：
##### 1.  我下载了iOSdemo之后,使用xcode要打开项目，发现报错，12个！

![12error](http://upload-images.jianshu.io/upload_images/1241385-5acc5d93af6ec5af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决方法：
<!--more-->
**Build Phases -> Link Binary With Libraries 中添加 `CFNetwork.framework`和`libc++.tbd`**
##### 2. ` libc++abi.dylib: terminating with uncaugt exception of type .... 开头`
解决方法：
**在Build Settings 中Linking模块下的Other Linker Flags 中添加-ObjC。,注意大小写**


#### 后续
PM说要以H5方式接入，所以支付操作改为前端加签交互。
流程如下：

![接入流程](http://upload-images.jianshu.io/upload_images/1241385-9fa8c7295ad875af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![具体步骤描述](http://upload-images.jianshu.io/upload_images/1241385-d59e32081bfb1a74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 关于APP SDK支付
```
    NSString *urlString   = @"http://wxpay.weixin.qq.com/pub_v2/app/app_pay.php?plat=ios";
        //解析服务端返回json数据
        NSError *error;
        //加载一个NSURL对象
        NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:urlString]];
        //将请求的url数据放到NSData对象中
        NSData *response = [NSURLConnection sendSynchronousRequest:request returningResponse:nil error:nil];
        if ( response != nil) {
            NSMutableDictionary *dict = NULL;
            //IOS5自带解析类NSJSONSerialization从response中解析出数据放到字典中
            dict = [NSJSONSerialization JSONObjectWithData:response options:NSJSONReadingMutableLeaves error:&error];

            NSLog(@"url:%@",urlString);
            if(dict != nil){
                NSMutableString *retcode = [dict objectForKey:@"retcode"];
                if (retcode.intValue == 0){
                    NSMutableString *stamp  = [dict objectForKey:@"timestamp"];

                    //调起微信支付
                    PayReq* req             = [[[PayReq alloc] init]autorelease];
                    req.partnerId           = [dict objectForKey:@"partnerid"];
                    req.prepayId            = [dict objectForKey:@"prepayid"];
                    req.nonceStr            = [dict objectForKey:@"noncestr"];
                    req.timeStamp           = stamp.intValue;
                    req.package             = [dict objectForKey:@"package"];
                    req.sign                = [dict objectForKey:@"sign"];
                    BOOL success = [WXApi sendReq:req];
                    if(!success){
                          NSLog(@"调微信失败");
                    }
                    //日志输出
                    NSLog(@"appid=%@\npartid=%@\nprepayid=%@\nnoncestr=%@\ntimestamp=%ld\npackage=%@\nsign=%@",[dict objectForKey:@"appid"],req.partnerId,req.prepayId,req.nonceStr,(long)req.timeStamp,req.package,req.sign );
                    return @"";
                }else{
                    return [dict objectForKey:@"retmsg"];
                }
            }else{
                return @"服务器返回错误，未获取到json对象";
            }
        }else{
            return @"服务器返回错误";
        }
```
大致流程如上， 我们只需要把参数封装成`PayReq`对象，然后调用微信SDK中的`sendReq`方法即可
```
/*! @brief 发送请求到微信，等待微信返回onResp
 *
 * 函数调用后，会切换到微信的界面。第三方应用程序等待微信返回onResp。微信在异步处理完成后一定会调用onResp。支持以下类型
 * SendAuthReq、SendMessageToWXReq、PayReq等。
 * @param req 具体的发送请求，在调用函数后，请自己释放。
 * @return 成功返回YES，失败返回NO。
 */
+(BOOL) sendReq:(BaseReq*)req;
```
详细的操作，请查看上文中demo中的具体配置。
