---
title: hexo+github搭建博客心历路程
date: 2017-03-09 10:30:35
tags:
---


#### 前言
今天突然回想起来以前在GitHub上创建的一个仓库，本来是想着以后写博客的时候都把数据同步上去，但是，今天尝试了一次，发现步骤稍显复杂，
1. 我先把项目clone到了本地
2. 创建了一个新的文件夹，在新的文件夹中，我新建了一个HTML文件
3. 修改了项目中的`index.html`文件，新建一个`a`标签，地址链接到***该目录下新建文件夹中的html文件上。***
4. 运行，发现一堆乱码（原来，我使用的是markdown语法，然后写在了html文件中。。实力脑残了一波。。）
5. 因此，我就修改了index.html中a标签链接的地址（本来为xxx.html，然后改成了xxx.md）
6. 然后，还是一堆乱码（等我测试了解了一圈之后，重新刷新，其实已经可以看见markdown语法已经有点效果了，但是，还是显示的不太好。）
<!--more-->
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1241385-b8190a99b67e8b4a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**对策**
然后，我在网上找了一些方法，可以直接在同一时间上传内容，然后修改首页index上的索引。`也就是那种用户可以专注于写博客，不用再费心于首页的样式，GitHub上传同步上的问题了`
想想是不是很开心呢？

这么以来，
#### 大致来说，就有两种方法实现：
##### jekyll+Github
##### Hexo+Github
那么这两种都有什么特点呢？
1. Jekyll  
采用ruby实现，所以速度上没有JS快
可以直接使用markdown来编辑文本，
但是据说不支持本地预览（据说，是据说各位。。放下手中的西瓜刀）


2. Hexo
采用Javascript实现，
可以直接使用markdown来编辑文本，
配置到GitHub之前，可以先在本地预览

安装方面来说：其实我感觉对于Mac用户来说，几乎都是一样的，
Mac自带的有ruby，所以jekyll可以直接安装。
而Hexo的话，用户需要先自己安装Node.js, 对用我这个安装的有brew的人来说，一行代码的事情：`brew install node`

*另外一点*，hexo对多平台多设备支持的没有jekyll好，至于解决方法，我后文再说吧

网上很多人纠结于这两个的选择，我感觉，还是仁者见仁智者见智吧！
[这里有一个各种静态网页生成器的比较](https://segmentfault.com/a/1190000002476681)

本文采用的是Hexo来配置的。

#### 安装步骤
前提：
1. [Hexo官网Github](https://github.com/hexojs/hexo)
2. Git -> 电脑安装xcode
3. Github客户端 -> `brew cask install github-desktop`
4. Node.js -> `brew install node`
5. 如果以上都做过了，燃石在执行安装hexo过程中会报错：`Failed at the hexo-util@0.6.1 build:highlight script`,那么还是老老实实去[Node.js官网](https://nodejs.org/en/)手动下载安装node.js吧


然后，开始！！
##### 安装Hexo
1. 首先安装hexo  `sudo npm install -g hexo`
2. 然后新建一个文件夹，用来存放以后的博客，当做博客的根目录。*这里默认为桌面*`Blog文件夹`-> `mkdir Blog`
3. `cd`到`Blog`文件夹，然后使用`hexo init`来初始化
4. 初始化完成之后，这个时候其实就已经创建好index.html文件了，此时我们`hexo g (hexo generate)`来生成静态页面。
5. `hexo s (hexo server)`来本地查看页面。
如果成功你会看到一个静态的你的主页。
当然，也会失败，我看别人的，是会有失败的情况的， 那么，`hexo clean`来清理下缓存，重新试下。

##### 新建Github页面
这个应该不用多说：
1. 登录[Github](github.com)网址，然后登陆，`new repository`新建仓库
2. 项目命名为`你github昵称+balabala.github.io`（最好以.io结尾）
3. 然后创建，点击当前仓库的setting选项，拉到倒数第二栏**Github Pages**，修改source栏为***master branch***, 生成网页
然后你会在倒数第二栏的上端看到你的博客的地址，记下来！!后面要用到的！！！


##### 关联GitHub，基础Hexo配置
此时我们在`~/UserMacName/Desktop/Blog/`目录下，然后我们打开该目录下的`_config.yml`文件进行编辑
翻到最下边，然后改成如下
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
type: git
repository: https://github.com/WooNoah/woodi.github.io.git  ###这里填写上面看到的网址.git
branch: master

```
**注意：**
hexo在以前的2.x的版本的时候，type栏写的是github, 升级成3.0版本之后，如果继续写type为github，那么会报错`找不到github`，因此需要先安装`hexo-deployer-git`(vim指令为:`npm install hexo-deployer-git --save`)，然后将github修改为git

然后配置博客相关的信息：
_config.yml上面：
```
# Site
title: Woooodi's Private Zone  ##这个是博客的title
subtitle:
description:
author:  D                   ##你的名字
language: zh-Hans               ##语言
timezone:                       ##时区

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://woonoah.github.io/woodi.github.io/        ###这里！！！非！常！重！要！！ 写上你博客的地址，也就是上面setting栏中，让你记住的那个地址
root: /woodi.github.io                               ###像上面说的，如果你的地址是在子文件夹中，那么root设置下
permalink: :year/:month/:day/:title/
permalink_defaults:
```
当然，更详细的参数配置都可以在[hexo官网上找到](https://hexo.io/zh-cn/docs/configuration.html)


当然，肯定有人是奔着Hexo好看的主题来的，所以，肯定会有人来问，***哪里换主题？***
首先，给大家一个[hexo主题选择的网址](https://github.com/hexojs/hexo/wiki/Themes)，大家各选其好，选一个出来，我使用的是一个名叫[next](https://github.com/iissnan/hexo-theme-next)的主题
下载之：
还是cd到Blog根目录，
`git clone https://github.com/iissnan/hexo-theme-next themes/next`
//新版`NextT地址`
`git clone https://github.com/theme-next/hexo-theme-next themes/next`
然后，你就会看到一个theme文件夹，里边放着一个next主题文件


仍然是_config.yml中，拉到最下边：
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next    ###这里，可以选择你想要换的主题名称。
```

当然， 在next文件夹中，也有一个_config.yml文件，我们叫他`主题配置文件`，而根目录的那个，我们可以叫`hexo配置文件`
大家不要搞混了


然后，大家就可以按照上面的步骤，修改完毕，
```
hexo clean  #先清理，防止有缓存
hexo g
hexo s
```
我们应该就能看到理想的主题了。

如果没有问题，我们就可以使用新的指令
`hexo deploy`来把本地配置部署到GitHub上了。

***笔者，因为没有配置上面url部分，导致多调试了1天***
PS，如果不配置上面的URL
则会显示如下
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1241385-c272b3931c70cfc4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另外，
`Next`模板，存在3中主题组合，
```
Muse         默认就是这个 
Mist     
Pisces         
```
只有Pisces支持
sideBar在左边

#### 一些基本路径
　文章在 source/_posts，编辑器可以用 Sublime，支持 markdown 语法。如果想修改头像可以直接在主题的 _config.yml 文件里面修改，友情链接，之类的都在这里，修改名字在 public/index.html 里修改，开始打理你的博客吧，有什么问题或者建议，都可以提出来，我会继续完善。
　
　#### 另外加一步美化操作：
　[hexo next主题下，自动更换背景图片](https://blog.csdn.net/jinggege0818/article/details/82113120)
　
　打开文档下themes\next\source\css\ _custom\custom.styl文件，这个是Next故意留给用户自己个性化定制一些样式的文件，添加以下代码：
　```
　/**
　url可更换为自己喜欢的图片的地址。
　repeat：是否重复出现
　attachment：定义背景图片随滚动轴的移动方式
　position：设置背景图像的起始位置。
　*/
　//https://source.unsplash.com     此网址能获取到随机的图片
　body {
　background:url(https://source.unsplash.com/random/1600x900);
　background-repeat: no-repeat;
　background-attachment:fixed;
　background-position:50% 50%;
　//这一行是设置图片位置的，宽度等宽，高度占整屏的35%
　//background-size:100% 35%;
　}
　```
　
　
　#### 常用的指令
　```
　hexo new "postName" #新建文章
　hexo new page "pageName" #新建页面
　hexo generate #生成静态页面至public目录
　hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
　hexo deploy #将.deploy目录部署到GitHub
　hexo help  #查看帮助
　hexo version  #查看Hexo的版本
　```
　
　
　#### 附录
　这里在补一个[Hexo主题站](https://hexo.io/themes/)
