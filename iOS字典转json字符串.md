---
title: iOS字典转json字符串
date: 2018-08-23 10:54:38
tags:
---

```
NSData *contentJSONData = [NSJSONSerialization dataWithJSONObject:contentDic options:(NSJSONWritingPrettyPrinted) error:nil];
NSString *contentJSONStr = [[NSString alloc] initWithData:contentJSONData encoding:NSUTF8StringEncoding];
contentJSONStr = [XLUtil htmlEntityDecode:contentJSONStr];

//将 &lt 等类似的字符转化为HTML中的“<”等
+(NSString *)htmlEntityDecode:(NSString *)string
{
string = [string stringByReplacingOccurrencesOfString:@"&quot;" withString:@"\""];
string = [string stringByReplacingOccurrencesOfString:@"&apos;" withString:@"'"];
string = [string stringByReplacingOccurrencesOfString:@"&lt;" withString:@"<"];
string = [string stringByReplacingOccurrencesOfString:@"&gt;" withString:@">"];
string = [string stringByReplacingOccurrencesOfString:@"&amp;" withString:@"&"];
// Do this last so that, e.g. @"&amp;lt;" goes to @"&lt;" not @"<"
return string;
}

```
