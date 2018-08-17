---
title: Emmet快速编写html
date: 2017-12-15 12:58:30
tags:
---

#### 前面我们通过《超高速前端开发工具——Emmet》简单介绍了Emmet以及在编辑器中的安装，今天我们再来看用Emmet超高速的编写HTML代码。
缩写是Emmet工具的核心，这些特殊的表达式通过Emmet解析并转化为结构化的代码块，而语法用CSS选择器，HTML标签和一些Emmet特定的代码生成，所以对于任何一个前端开发人员都可以很容易地掌握和使用。
你可以使用标签名称为div、p等生成HTML标签，Emmet没有一组预定义可用的标记名称，您可以编写任何单词并把它转化为一个标签，例如：


`div → <div></div>, footer → <footer></footer>`


#### Emmet代码虽然没有特定的标签，但编写时还是有一定的规则与技巧，下面我们就看一看：
生成HTML文件的初始结构：
之前我们会用软件直接新建一个HTML文档，初始结构就生成了，但有些编辑器是不带这个功能的，手动输入是件痛苦的事，有了Emmet一切变得如此简单。生成HTML4（过渡）结构初始文档只需输入“html:4t”，HTML4（严格）结构初始文档只需输入“html:4s”，将生成标准的HTML4（严格）标准结构：

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">  
<html lang="en">  
<head>  
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">  
    <title>Document</title>  
</head>  
<body>  

</body>  
</html>  
```
<!--more-->
而HTML5就更简单省事了，像HTML4的输入格式“html:5”，更狠的是HTML5只需输入“!”,就可以生成HTML5文档的初始结构：

```
<!doctype html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Document</title>  
</head>  
<body>  

</body>  
</html>  
```

##### ***1. 父子关系：>，使用>操作符在内部相互嵌套的标签：***


`div>ul>li`


将生成：

```
<div>  
    <ul>  
        <li></li>  
    </ul>  
</div>  
```

##### ***2. 兄弟关系：+，使用+操作符将标签处于同一个层级：***


`div+p+footer`


将生成：

```
<div></div>  
<p></p>  
<footer></footer>  
```

生成兄弟关系时，像ul dl这样的列表标签，使用+操作符将生成一个标准的列表结构：


`ul+`  


将生成：

```
<ul>  
    <li></li>  
</ul>  
```



`dl+`  


将生成：

```
<dl>  
    <dt></dt>  
    <dd></dd>  
</dl>  
```

##### ***3. 上级关系：^，使用^操作符使标签与前一标签的父级处于相同的级别：***


`div+div>p>span+em^bq`


将生成：

```
<div>      
    <p><span></span><em></em></p>      
    <blockquote></blockquote>      
</div>  
```

***使用两^操作符就与前一标签的爷爷级是相同级别，依此类推：***


`div+div>p>span+em^^bq`


将生成：

```
<div></div>  
<div>  
    <p><span></span><em></em></p>  
</div>  
<blockquote></blockquote>  
```

##### ***4. 乘法：*，使用*操作符可以输出多个标签：***


`div>ul>li*5`


将生成：

```
<div>  
    <ul>  
        <li></li>  
        <li></li>  
        <li></li>  
        <li></li>  
        <li></li>  
    </ul>  
</div>  
```

##### ***5. 分组：()，用()操作符进行分组，使编写的代码结构更加清晰、明了，一组标签就相当一个元素：***


`div>(header>ul>li*2>a)+footer>p`


将生成：

```
<div>  
    <header>  
        <ul>  
            <li><a href=""></a></li>  
            <li><a href=""></a></li>  
        </ul>  
    </header>  
    <footer>  
        <p></p>  
    </footer>  
</div>  
```

你可以使用多个()，并使用乘法*操作符：


`(div>dl>(dt+dd)*3)+footer>p`


将生成：

```
<div>  
    <dl>  
        <dt></dt>  
        <dd></dd>  
        <dt></dt>  
        <dd></dd>      
        <dt></dt>  
        <dd></dd>  
    </dl>  
</div>  
<footer>  
    <p></p>  
</footer>  
```

***CSS选择器，给标签指定id和class选择器，只需在标签的后面直接添加，但必需以.或#开头：***


`div#header+div.page+div#footer.class1.class2.class3`


将生成：

```
<div id="header"></div>  
<div class="page"></div>  
<div id="footer" class="class1 class2 class3"></div>  
```

Emmet默认的标签是div，所以我们在写带有CSS选择器的div标签时，可以省去div，你可以试试。
自定义属性：[]（英文下的中括号），使用[]操作符给标签添加自定义属性：

```
td[title="Hello world!" colspan=3]  


将生成：


<td title="Hello world!" colspan="3"></td>  
```

可以把你喜欢的一些属性放在[]内，如果不指定属性值，代码将生成不带属性值的HTML默认标签：


td[colspan title]  


将生成：


<td colspan="" title=""></td>  


属性值必需使用单引号或双引号，不然就会出现你可能想到的效果。
##### ***6. 项目编号：$，使用*可以重复的标签，$可以使标签生成有序列表，输出的值为数字：***


`ul>li.item$*5`


将生成：

```
<ul>  
    <li class="item1"></li>  
    <li class="item2"></li>  
    <li class="item3"></li>  
    <li class="item4"></li>  
    <li class="item5"></li>  
</ul>  
```

除了生成无序列表，其它的标签也是一样：


`h$[title=item$]{Header $}*3`


将生成：

```
<h1 title="item1">Header 1</h1>  
<h2 title="item2">Header 2</h2>  
<h3 title="item3">Header 3</h3>  
```

***你也可以使用多个$操作符用0（零）来分填充数字：***



`ul>li.item$$$*5`


将生成：

```
<ul>  
    <li class="item001"></li>  
    <li class="item002"></li>  
    <li class="item003"></li>  
    <li class="item004"></li>  
    <li class="item005"></li>  
</ul>  
```

##### ***7. 更改列表的起始数字与顺序，看下面的代码就一目了然：数字的倒序，只需在*前添加@-：***


`ul>li.item$@-*5`  


将生成：

```
<ul>  
    <li class="item5"></li>  
    <li class="item4"></li>  
    <li class="item3"></li>  
    <li class="item2"></li>  
    <li class="item1"></li>  
</ul>  
```

***起始数字，在*前添加@起始数字：***


`ul>li.item$@3*5`  


将生成：

```
<ul>  
    <li class="item3"></li>  
    <li class="item4"></li>  
    <li class="item5"></li>  
    <li class="item6"></li>  
    <li class="item7"></li>  
</ul>  
```

而从起始数字为3的列表倒序，只需把上面的Emmet代码item后面的数字写成@-3*5。
##### ***8. 文本：{}，使用花括号来添加文本元素：***


`a{Click me}`  


将生成：


`<a href="">Click me</a>`


注意：当{}作为单独的一个操作符使用时，a{click}和a>{click}将生成相同的标签，但当使用了多个，或用了其它操作符时将会生成不同的标签：

```
<!-- a{click}+b{here} -->  
<a href="">click</a><b>here</b>  

<!-- a>{click}+b{here} -->  
<a href="">click<b>here</b></a>  
```

***当元素用+连接时，文本{}编写正确后，并不能改变标签的层级关系：***


`p>{Click }+a{here}+{ to continue}`  


将生成：


`<p>Click <a href="">here</a> to continue</p>`


对于一些特殊的标签类型，比如：a、img、link、input等带有属性值的标签，在编写Emmet插件时已经为我们编写好了基本的结构。
比如a标签，在编辑中输入a，即可解析成一个基本的a标签：


`<a href=""></a>`


a标签有几个属性值，在编写Emmet代码时可以添加这些值来覆盖默认的属性值：


`a[href="http://salonglong.com" title="远方的雪山" class="logo"]{远方的雪山}`


将生成：


`<a href="http://salonglong.com" title="远方的雪山" class="logo">远方的雪山</a>`  


因为属性值是不可以缩写的，所以看起来编写带属性值是相似的。
在许多情况下，你可以跳过输入标签名称和Emmet代码，得到HTML代码，例如下面的例子：

```
<div>  
    .item   
</div>  

<span>.item</span>  

<ul class="nav">  
    .item   
</ul>  
```

将生成：

```
<div>  
    <div class="item"></div>  
</div>  

<span><span class="item"></span></span>  

<ul class="nav">  
    <li class="item"></li>  
</ul>  
```

从上面的例子中我们可以看到，插件会根据id或class所在的父级标签生成相应的标签，这种写法也是遵循HTML的编写规则，通过下面的代码你会更明白：

```
.wrap>.content                           div.wrap>div.content   
em>.info                                     em>span.info   
ul>.item*3                                   ul>li.item*3   
table>#row$*4>[colspan=2]       table>tr#row$*4>td[colspan=2]  


上面对应的四组代码最终生成的代码对应为：


<div class="wrap">  
    <div class="content"></div>  
</div>  

<em><span class="info"></span></em>  

<ul>  
    <li class="item"></li>  
    <li class="item"></li>  
    <li class="item"></li>  
</ul>  

<table>  
    <tr id="row1">  
        <td colspan="2"></td>  
    </tr>  
    <tr id="row2">  
        <td colspan="2"></td>  
    </tr>  
    <tr id="row3">  
        <td colspan="2"></td>  
    </tr>  
    <tr id="row4">  
        <td colspan="2"></td>  
    </tr>  
</table>  
```

我们应该了解到，CSS选择器在块级元素中默认的HTML标签为div，在内联级元素中为span，而对于HTML一些特殊的标签：ul li、table tr td，将会生成对应的内部标签。
这篇文章介绍了HTML的基本标签在Emmet下的写法，萨龙龙把它当做学习笔记，可以随时查看，如果对你有帮助就更好。
Emmet编写代码的格式最重要的就是不能有空格，如果有空格将不会完全解析和生成HTML代码。
Emmet最核心的就是缩写，而它也不属于一门语言，编写时也不需要有很强的可读性，最重要的就是能高速的生成HTML代码。只要你多练习，多看下官方的英文文档，很快你将掌握Emmet的编写方式，同时也将大大提高你的前端开发速度。

[原文链接](http://salonglong.com/emmet-html.html)

