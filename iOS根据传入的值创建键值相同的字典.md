---
title: iOS根据传入的值创建键值相同的字典
date: 2019-07-12 12:13:10
tags:
---

```
/* This macro is a helper for making view dictionaries for +constraintsWithVisualFormat:options:metrics:views:.  
NSDictionaryOfVariableBindings(v1, v2, v3) is equivalent to [NSDictionary dictionaryWithObjectsAndKeys:v1, @"v1", v2, @"v2", v3, @"v3", nil];
*/
#define NSDictionaryOfVariableBindings(...) _NSDictionaryOfVariableBindings(@"" # __VA_ARGS__, __VA_ARGS__, nil)
UIKIT_EXTERN  NSDictionary *_NSDictionaryOfVariableBindings(NSString *commaSeparatedKeysString, __nullable id firstValue, ...) NS_AVAILABLE_IOS(6_0);
```
摘自[Apple Sample](https://developer.apple.com/library/archive/samplecode/ViewTransitions/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007411)
