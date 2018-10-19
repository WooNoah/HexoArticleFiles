---
title: iOS上传文件参数解析(HTTP header Content-Disposition)
date: 2018-10-19 16:41:47
tags:
---

#### **AFNetworking**中提供的图片上传方法：
```
[_sessionManager POST:uploadUrl parameters:dict constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
if (imageData) {
[formData appendPartWithFileData:imageData name:<#name#> fileName:<#fileName#> mimeType:@"jpg"];
}
} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {
suc(task,responseObject);
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
fail(task,error);
}];
```
可以看到里边有几个字段：
```
/**
Appends the HTTP header `Content-Disposition: file; filename=#{filename}; name=#{name}"` and `Content-Type: #{mimeType}`, followed by the encoded file data and the multipart form boundary.

@param data The data to be encoded and appended to the form data.
@param name The name to be associated with the specified data. This parameter must not be `nil`.
@param fileName The filename to be associated with the specified data. This parameter must not be `nil`.
@param mimeType The MIME type of the specified data. (For example, the MIME type for a JPEG image is image/jpeg.) For a list of valid MIME types, see http://www.iana.org/assignments/media-types/. This parameter must not be `nil`.
*/
- (void)appendPartWithFileData:(NSData *)data
name:(NSString *)name
fileName:(NSString *)fileName
mimeType:(NSString *)mimeType
{
NSParameterAssert(name);
NSParameterAssert(fileName);
NSParameterAssert(mimeType);

NSMutableDictionary *mutableHeaders = [NSMutableDictionary dictionary];
[mutableHeaders setValue:[NSString stringWithFormat:@"form-data; name=\"%@\"; filename=\"%@\"", name, fileName] forKey:@"Content-Disposition"];
[mutableHeaders setValue:mimeType forKey:@"Content-Type"];

[self appendPartWithHeaders:mutableHeaders body:data];
}
```
#### 而 Appends the HTTP header `Content-Disposition: file; filename=#{filename}; name=#{name}"` and `Content-Type: #{mimeType}`, followed by the encoded file data and the multipart form boundary.又是什么呢？

![](https://upload-images.jianshu.io/upload_images/1241385-1bc25da5f14c1aa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/1241385-dbf99a79c8e8edcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看出来：
#### name：就是前后端约定好的描述图片的字段，（比如传入avatar，那就是代表该图片是用户头像，后台在取的时候，也需要根据这个name来取出）
#### fileName: 上传的文件名称


https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition
https://tools.ietf.org/html/rfc6266
https://blog.csdn.net/sinat_38364990/article/details/70867357
http://blog.csdn.net/wwlhsgs/article/details/45075327
http://www.cnblogs.com/brucejia/archive/2012/12/24/2831060.html
