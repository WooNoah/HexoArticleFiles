---
title: SVN-solve-conflicts
date: 2017-03-20 11:07:58
tags:
---

#### 原因
由于上周五改了代码，但是下班的时候忘记提交SVN了，然后周末赶着上线，所以在家里加班，**改了代码，换了图片**，整理完成后，打包提交了。

然后，今天到公司，想要同步下，执行了下`svn update`，然后，冲突就出现了！

因此：出现了两种类型的冲突。

1.    tree conflict
	说明两次提交，修改了目录结构，包括文件或者文件所在目录的改名、删除、移动。
	**因为两次代码实现不同**，`第二次添加了category`，所以才造成了这个问题。

1.    text conflict
	1. *由于我在项目中添加了运行自增脚本*，所以，项目的build是不一样的。
	1. 两次代码实现方式不同。
	2. 同一个位置，图片不同。

#### 解决
<!--more-->

1.	tree conflict
	update的时候直接添加了。

1.	text conflict
	- 代码这个不用说了，也是直接更新了，如果出现同一个文件，同一个位置，改动不同，则会出现：
	```
	<<<<<< confliceFileName
	content1
	==========
	content2
	>>>>>>>>
	```

		留下需要的内容，删除不需要的，`解决`！

	- 版本号的问题
		xcode使用的是plist类型文件，不过没有别的，本质上也是xml, `open in source code`,解决方法同上！
	PS：
	删除之后，使用指令`svn resolved [file]`来标记冲突解决。

	- 这里要特别说下图片的问题了！
		默认情况下，同名图片，更新的时候，***后来更行的是会直接替换原图***,所以会出现相应的英文提示。然后执行上面所说的`svn resolved`指令的时候，会出现一些问题(以名为btn@2x的图片为例)：
		```
		svn resolved 项目名/图片资源文件夹名字/btn@2x
		然后，svn会报错!
		E200009: '项目名/图片文件夹名/btn@2x.png': a peg revision is not allowed here
		```
		解决：
		在图片的后边加个`@`即可：
		`svn resolved 项目名/图片文件夹名/btn@2x.png@`
		```
		From the SVN book (emphasis added):

		The perceptive reader is probably wondering at this point whether the peg revision syntax
causes problems for working copy paths or URLs that actually have at signs in them.
After all, how does svn know whether news@11 is the name of a directory in my tree
or just a syntax for “revision 11 of news”? Thankfully,
while svn will always assume the latter, there is a trivial workaround.
You need only append an at sign to the end of the path,
such as news@11@. svn cares only about the last at sign in the argument,
and it is not considered illegal to omit a literal peg revision specifier after that at sign.
This workaround even applies to paths that end in an at sign—you would use filename@@
to talk about a file named filename@.
		```
		[也可以参考stackoverflow](http://stackoverflow.com/questions/757435/how-to-escape-characters-in-subversion-managed-file-names)
