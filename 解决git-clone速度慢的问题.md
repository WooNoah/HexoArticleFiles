---
title: 解决git clone速度慢的问题
date: 2018-11-20 14:30:23
tags:
---

> 有时候从GitHub上拉取仓库，或者cocoapods更新本地仓库的时候，会遇到网速极慢的情况，本来我以为是国外服务器的原因导致了传递速度减慢，然后朋友说他以前遇到过这个问题，并给了解决方案，这里自己记录一下，以备以后再次遇到能够快速解决
以下是他的原文：

[解决git clone速度慢的问题](https://masterking.github.io/2018/10/26/%E8%A7%A3%E5%86%B3git-clone%E9%80%9F%E5%BA%A6%E6%85%A2%E7%9A%84%E9%97%AE%E9%A2%98/)

之前一直以为是公司对网络的封锁,导致git clone速度慢,后来网上查找这个问题的时候得知不能怪公司,是因为咱们在这个局域网内…
扯多了,回到正题,解决问题,由于我的是Mac电脑,我说的只针对Mac电脑

##### 1. 进入终端命令行模式,输入
```
sudo vim /etc/hosts
```

##### 2. 输入i进入编辑模式,移动到最后一行准备输入

##### 3\. 用浏览器访问 [http://tool.chinaz.com](http://tool.chinaz.com/) 使用` IP查询`工具获得github.com和github.global.ssl.fastly.net的ip地址

##### 4\. 回到第2步中按如下格式输入:

```
192.30.253.112 github.com
151.101.44.249 github.global.ssl.fastly.net
```

##### 5\. 按 esc 键,然后输入 :wq 保存文件并退出vim编辑模式,到此hosts文件修改结束

##### 6\. 更新DNS缓存,输入")6\. 更新DNS缓存,输入

```
sudo dscacheutil -flushcache
```

