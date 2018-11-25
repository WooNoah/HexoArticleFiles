---
title: 换电脑后怎么继续维护以前hexo+Github创建的个人网站
date: 2017-09-30 15:28:02
tags:
---

-----------------
#### 2018年11月25日补充：
今天电脑升级了系统，然后hexo不知道怎么回事没有了。
等于又在自己电脑上重新安装了一遍hexo

##### 先说一下途中遇到的一些问题：
> 在安装完hexo之后，我从github上拉取了以前的hexoBaseConfig文件夹

本来以为直接把文章md源文件拖进来，然后主题相关文件夹拖到theme下即可。但是，在执行到`hexo s`之后，点击打开本地调试网页，发现报错说`找不到文件夹路径`，
1. 我先考虑可能是端口冲突了
```
lsof -i tcp:4000
```
查询了一波。。发现只有node占用了这个端口。而且把本地虚拟调试网页关闭了之后，该指令并不能找到什么了。**排除此猜想**
2. 那只能是hexo路径的问题了
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Hexo #网站标题
subtitle:   #网站副标题
description:  #网站描述
author: John Doe  #作者
language:    #语言
timezone:    #网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com   #你的站点Url
root: /                       #站点的根目录
permalink: :year/:month/:day/:title/   #文章的 永久链接 格式   
permalink_defaults:    #永久链接中各部分的默认值

# Directory   
source_dir: source   #资源文件夹，这个文件夹用来存放内容
public_dir: public     #公共文件夹，这个文件夹用于存放生成的站点文件。
tag_dir: tags         # 标签文件夹     
archive_dir: archives    #归档文件夹
category_dir: categories      #分类文件夹
code_dir: downloads/code     #Include code 文件夹
i18n_dir: :lang                #国际化（i18n）文件夹
skip_render:                #跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。    

# Writing
new_post_name: :title.md # 新文章的文件名称
default_layout: post     #预设布局
titlecase: false # 把标题转换为 title case
external_link: true # 在新标签中打开链接
filename_case: 0     #把文件名称转换为 (1) 小写或 (2) 大写
render_drafts: false  #是否显示草稿
post_asset_folder: false  #是否启动 Asset 文件夹
relative_link: false      #把链接改为与根目录的相对位址    
future: true                #显示未来的文章
highlight:                    #内容中代码块的设置    
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:          #分类别名
tag_map:            #标签别名

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD         #日期格式
time_format: HH:mm:ss        #时间格式    

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10    #分页数量
pagination_dir: page  

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape   #主题名称

# Deployment
## Docs: https://hexo.io/docs/deployment.html
#  部署部分的设置
deploy:     
  type:  #类型，常用的git
```
但是，**在查看了Directory之后，发现也并没有什么问题!!!**
然后我尝试着改了`URL`相关的配置，
> 网站存放在子目录
如果您的网站存放在子文件夹中，例如 [http://yoursite.com/subDirectory]()，则请将您的 url 设为 [http://yoursite.com/subDirectory]() 并把 root 设为 /subDirectory/

此处root默认值为/，也就是意味着是跟_config.yml文件同一层级的。也就是说，如果我们博客是放在此层级中的某一个子文件夹中的时候，我们需要把子文件夹路径写上去！

然后，填写完`root`之后，执行`hexo server`命令之后，会显示配置过的root路径上去，这里以`subBlogPath`字段为例，此时会显示为：
```
INFO  Hexo is running at http://localhost:4000/subBlogPath. Press Ctrl+C to stop.
```

> 在经过了一番搜索调试之后，发现，修改url也并没有什么用处。。
我也就彻底理解不了了，我只能理解为以前上传的HexoBaseConfig文件中存在一部分配置问题了。。这里如果有大佬知道问题所在，请不吝赐教！！先说声谢谢了！

#### *虽然不明白，但是还是有解决方法的不是么？*
这也就是更新此文章的意义所在！
正因为以前的配置可能跟现在的存在不同，或者冲突。然后导致了pull下来之后无法使用，那我们换个思路。
1. 我们只要安装好了hexo, 那可以很简单的通过执行`hexo init`指令来重新配置相关的环境。。
2. 配置完成之后，把`_config.yml文件`、`文章md源文件`、`主题文件`拷贝进去，那不就是完成了迁移了嘛？
**所以，只要备份了这些文件即可，别的hexo的相关配置，就交给hexo自己去重新配置吧！**

如此这般，这般如此之后，配备了hexo的Blog文件夹又重新出现在了我的桌面上。。

----------------------------
#### 2017年10月09日补充
由于十一国庆节前，只是有一个上面所说的想法，现在节后回来，总得按照这个实践一下。

  其实上面所说的博客内容，还是`hexo生成的html文件`，如果以前的文件没有保存的话，还是没有任何作用的，因此，重点就在：***保存博客文章的原始文件（也就是我们写博客的时候创建的markdown文件）***，有了这个文件之后，我们就可以使用`hexo g`命令，重新生成html文件了。
今天，我试了2种方案:
1. 统一把整个博客文件夹（包含配置文件，.md文件，以及hexo处理后生成的各种文件夹）备份到GitHub上。但是，试过之后，效果是可以达到的，我发现了一些比较繁琐的一些问题： **hexo自己生成的文件夹，由于hexo clean, hexo g等命令的作用下，会有重新生成或者删除的操作，在git管理下，需要繁琐的git checkout 或者git add。** 所以我又萌生了另一种想法，也就是想法2
2. 其实这个方法就是退而求其次： 我们把hexo的配置文件备份到一个远程仓库，然后另外备份文章原始文件夹。
在更换电脑之后使用的时候，我们只需要，`git clone <#hexo配置文件#>`比如到桌面*blog文件夹*, 然后，删除掉.git隐藏文件（因为这个配置文件是只需要配置一次的，换电脑的时候，也是直接获取一次就可以。不需要重复配置，当然，如果你想要修改主题或者别的配置文件，那么也可以留着。）
然后`git clone <#文章原始md文件#>`到`blog->source->_posts文件夹`下， ***或者直接clone到source文件夹下，然后修改名字为_posts***，这么以来，以后再提交的时候，就只需要使用hexo提交文章，然后`git add <#新添加的文章md文件#>`到远程仓库了，免去了繁琐的修改hexo所生成的文件的操作。


如果在hexo s调试的时候，提示： no layout: index.html。
查看主题文件是否存在：  
如果不存在，
cd到Blog根目录，
`git clone https://github.com/iissnan/hexo-theme-next themes/next`


3. 经过几个小时的实践，方法二也存在一些问题，比如同步备份基础配置的时候，theme主题下的next文件夹不能同步到GitHub上，
多次配置修改hexo参数之后，出现GitHub404错误，所以也可以***仅仅备份文章源文件，然后每次更换的时候，重新装载npm安装hexo，然后下载文章源文件到source/_posts文件夹下***

---------------------
#### 以下是原文章：
#### 首先
**1. hexo的相关配置文件都是本地存放的。**
**2. 同步到GitHub上的文件都是`hexo g`之后生成的（博客根目录下的）public文件夹中的内容。**


#### 结论
##### 前提
如果以前是使用hexo搭建的博客，以前电脑中创建过hexo根目录，
然后中途更换过电脑，以前电脑上的配置文件 在`现电脑上都没有了`

##### 解决方法
***如果想要重新在Github上创建repository，然后继续管理博客***
那么请移步我以前写的[hexo+github搭建博客心历路程](http://www.jianshu.com/p/e4ad163dd963)
***如果想要继续维护以前的博客，那么：***
1. 首先，我们要把以前github上博客的仓库clone下来，通过实践，我们知道，这个clone下来的文件夹就是以前本地博客根目录下的`public`文件夹
2. 我们可以自己本地安装hexo之后，自己创建一个public文件夹，然后把clone下来的文件***全部放进去***
3. 为了以后不再这么麻烦，我们可以直接**在GitHub上备份一下我们的hexo配置文件**，以后再遇到这种情况的时候，就可以直接`clone博客配置文件到本地， clone博客内容文件夹到本地，然后把内容文件夹该名称public放在配置文件夹根目录即可`


