---
title: vim ls find grep sed指令
date: 2018-09-05 18:01:27
tags:
---

> 先来说下意图：以前在做项目的时候，在网上找了好多demo去了解参考，然后时间长了，没有整理，整个桌面就显得格外的杂乱。于是，就起了一个整理下桌面的念头。

> 然后这些文件之间，我发现了一些共性：
比如说iOS中导航栏相关的demo，基本都会包含有“Navigation”or“Nav”关键字，
遂，就有了【使用vim指令，查找包含“Nav”关键字的文件或者文件夹】这个思路。也就是本文诞生的理由。

这里研究了`ls` `find` `grep` `sed`三个指令。

<!--more-->
#### 0. 正则
这里有一个前提，正则。显示、查找的时候，可以根据正则来匹配相应的文件，内容。

```
?,*,+,\d,\w 都是等价字符
?等价于匹配长度{0,1}  //0或者1位
*等价于匹配长度{0,}   //随机位数
+等价于匹配长度{1,}   //一位以上
\d等价于[0-9]        //0到9范围内的字符串
\D等价于[^0-9]      //任何不在0到9范围内的字符串
\w等价于[A-Za-z_0-9]
\W等价于[^A-Za-z_0-9]。
常用运算符与表达式：
^ 开始
（） 域段
[] 包含,默认是一个字符长度
[^] 不包含,默认是一个字符长度
{n,m} 匹配长度 
. 任何单个字符(\. 字符点)
| 或
\ 转义
$ 结尾
[A-Z] 26个大写字母
[a-z] 26个小写字母
[0-9] 0至9数字
[A-Za-z0-9] 26个大写字母、26个小写字母和0至9数字
， 分割
　　　　　　　　　　　　　　　　　　　　　　　　
分割语法：
[A,H,T,W] 包含A或H或T或W字母
[a,h,t,w] 包含a或h或t或w字母
[0,3,6,8] 包含0或3或6或8数字
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
语法与释义：
基础语法 "^([]{})([]{})([]{})$"
正则字符串 = "开始（[包含内容]{长度}）（[包含内容]{长度}）（[包含内容]{长度}）
```
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
#### 1. ls
```
-1  -- single column output
-A  -- list all except . and ..
-B  -- print octal escapes for control characters
-C  -- list entries in columns sorted vertically
-F  -- append file type indicators
-H  -- follow symlinks on the command line
-L  -- list referenced file for sym link
-P  -- do not follow symlinks
-R  -- list subdirectories recursively
-S  -- sort by size
-T  -- show complete time information
-a  -- list entries starting with .
-b  -- as -B, but use C escape codes whenever possible
-c  -- status change time
-d  -- list directory entries instead of contents
-f  -- output is not sorted
-g  -- long listing but without owner information
-h  -- print sizes in human readable form
-i  -- print file inode numbers
-k  -- print sizes of 1k
-l  -- long listing
-m  -- comma separated
-n  -- numeric uid, gid
-o  -- display file flags
-p  -- append file type indicators for directory
-q  -- hide control chars
-r  -- reverse sort order
-s  -- display size of each file in blocks
-t  -- sort by modification time
-u  -- access time
-w  -- print raw characters
-x  -- sort horizontally
```
这里搜索包含“Nav”字符的文件，使用`ls *Nav*` （*）为正则里的`*等价于匹配长度{0,},也就是所谓的通配符`.
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
##### 这里扩展一个知识点
```
# ls -la
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
total 8
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
drwxr-xr-x    2 root    root         4096 Nov 6 00:04 .
drwxr-x---   14 root     root         4096 Nov 6 00:04 ..
-rw-r--r--    1 root     root         1668 Oct 3 2007 anaconda-ks.cfg
drwxr-xr-x    2 root     root         4096 Nov 6 00:04 aa
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```

显示的文件详细信息分别代表什么呢？
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
1）total 8 代表当前目录下文件大小的总和为8K（每个目录的大小都按4K算）
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　

2）drwxr-xr-x 第一个字符有3种情况：“-”表示普通文件，“d”代表目录，“l”代表连接文件，“b”代表设备文件。
后面的9个字符每3个为一组，分别代表文件所有者、文件所有者所在用户组、其它用户对文件拥有的权限。每组中3个字符分别代表读、写、执行的权限，若没有其中的任何一个权限则用“-”表示。执行的权限有两个字符可选“x”代表可执行，“s”代表套接口文件。
　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　
![](https://upload-images.jianshu.io/upload_images/1241385-d460dd2452ab1699.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
紧接着的数字2代表 “aa”这个目录下的目录文件数目（这个数目=隐藏目录数目+普通目录数目）。我们进入“aa”目录用命令 ls –al （为了看到隐藏文件我们加上-a这个参数）
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```
❗️一个`.`表示当前路径，两个`..`表示父路径。
所以，上面的第3行中的4代表当前目录中有子目录4个，即`.`,`..`,`anaconda-ks.cfg`和`aa`
第4行中的14代表这个目录的上一层目录中有14个子目录。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
再接下来的root代表这个文件（目录）的属主为 用户root
再接下来的root代表这个文件（目录）所属的用户组为 组root
4096 代表文件的大小（字节数），目录的大小总是为4096字节。
Nov 6 00:04 代表文件（目录）的修改时间。
aa代表文件（目录）在名字。
```
3）文件名颜色的含义
```
默认色代表普通文件。 例：install.log
绿色代表可执行文件。 例：rc.news
红色代表tar包文件。    例：vim-7.1.tar.bz2
蓝色代表目录文件。    例：aa
水红代表图象文件。    例：Sunset.jpg
青色代表链接文件。    例：rc4.d   （此类文件相当于快捷方式）
黄色代表设备文件。    例：fd0
```
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
4）几个比较常用的参数。
```
-t 按最后修改时间排序。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
-S 按文件大小排序。（大写的S）
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
-r 排序时按倒序。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
-h 显示文件大小时增加可读性 （例：1K 234M 2G）
```
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
如果这个aa是个普通文件，2就代表这个文件有2个别名（这个文件被人创建了一个硬链接文件）
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
#### 2. find
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```
find命令原理：从指定的起始目录开始，递归地搜索其各个子目录，查找满足寻找条件的文件，并可以对其进行相关的操作。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
格式：find [查找目录] [参数] [匹配模型]  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
多参数格式：find [查找目录] [参数] [匹配模型] [参数] [匹配模型]  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
例如：
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
1、find . -name "*.sh"           
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在当前目录（及子目录）下找以sh结尾的文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
2、find . -perm 755               
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在当前目录（及子目录）下找属性为755的文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
3、find . -user root                  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在当前目录（及子目录）下找属主为root的文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
4、find /var -mtime -5           
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在/var下找更改时间在5天以内的文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
5、find /var -mtime +3          
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在/var下找更改时间在3天以前的文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
6、find /etc -type l                
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在/etc下查找文件类型为|的链接文件。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
7、find . -size +1000000c    
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找在当前目录（及子目录）下查找文件大小大于1M的文件，1M是1000000个字节。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
8、find . -perm 700 |xargs chmod 777         
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找出当前目录（及子目录）下所有权限为700的文件，并把其权限重设为777。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
9、find . -type f |xargs ls -l                         
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
查找出文件并查看其详细信息。
```
所以使用`find`查找的指令为：
`find ./ -iname "nav"`在当前目录下，不区分大小写查找以nav字符为name的文件和文件夹
`find ./ -iname "nav*"`在当前目录下，不区分大小写查找以nav字符开头的文件和文件夹
`find ./ -iname "*nav*"`在当前目录下，不区分大小写查找name中包含nav的文件和文件夹
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
#### 3. grep
> grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。
```
-a 不要忽略二进制数据。
-A<显示列数> 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-b 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-c 计算符合范本样式的列数。
-C<显示列数>或-<显示列数>  除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-d<进行动作> 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
-e<范本样式> 指定字符串作为查找文件内容的范本样式。
-E 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
-f<范本文件> 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
-F 将范本样式视为固定字符串的列表。
-G 将范本样式视为普通的表示法来使用。
-h 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
-H 在显示符合范本样式的那一列之前，标示该列的文件名称。
-i 忽略字符大小写的差别。
-l 列出文件内容符合指定的范本样式的文件名称。
-L 列出文件内容不符合指定的范本样式的文件名称。
-n 在显示符合范本样式的那一列之前，标示出该列的编号。
-q 不显示任何信息。
-R/-r 此参数的效果和指定“-d recurse”参数相同。
-s 不显示错误信息。
-v 反转查找。
-[w](http://man.linuxde.net/w "w命令") 只显示全字符合的列。
-x 只显示全列符合的列。
-y 此参数效果跟“-i”相同。
-o 只输出文件中匹配到的部分。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```
上面是`grep`命令的一些配置参数，
但是一些常用的就一个`-r`即`递归子文件夹`, `-n`即`显示行号`
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
一些用法：
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
1. 在文件中搜索一个单词，命令会返回一个包含“match_pattern”的文本行：
```
grep match_pattern file_name
grep "match_pattern" file_name
```
2. 在多个文件中查找：
```
grep "match_pattern" file_1 file_2 file_3 ...
```
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
所以，如果想要找到`某个文件夹下的子文件中是否存在某个字符`，可以用
`grep -r -n "Nav" ./` 即在当前目录以及当前目录的子文件夹中，寻找Nav字符串，并显示行号
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
这里做一个demo
```
桌面上新建一个text.txt，内容如下：
➜  Desktop cat text.txt
sfasdfsadfsad signal sdf
saljflskjdlafjsignalsinal
sdjflkasjdlfkjsa
```
然后执行：
```
➜  Desktop grep "signal" ./text.txt
sfasdfsadfsad signal sdf
saljflskjdlafjsignalsinal
```
可以看到，`grep`能匹配到文件内的对应字符
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
#### 4. sed
既然能大规模找到相应的字符了，那肯定会有一个需求：大规模替换：这里就需要用到一个命令`sed`
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```
**命令格式**
sed [options] '[command](http://man.linuxde.net/command "command命令")' [file](http://man.linuxde.net/file "file命令")(s)
sed [options] -f scriptfile file(s)
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
### 选项
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
-e<script>或--expression=<script>：以选项中的指定的script来处理输入的文本文件；
-f<script文件>或--file=<script文件>：以选项中指定的script文件来处理输入的文本文件；
-h或--[help](http://man.linuxde.net/help "help命令")：显示帮助；
-n或--quiet或——silent：仅显示script处理后的结果；
-V或--version：显示版本信息。</pre>
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
### sed命令
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
a\ 在当前行下面插入文本。
i\ 在当前行上面插入文本。
c\ 把选定的行改为新的文本。
d 删除，删除选择的行。
D 删除模板块的第一行。
s 替换指定字符
h 拷贝模板块的内容到内存中的缓冲区。
H 追加模板块的内容到内存中的缓冲区。
g 获得内存缓冲区的内容，并替代当前模板块中的文本。
G 获得内存缓冲区的内容，并追加到当前模板块文本的后面。
l 列表不能打印字符的清单。
n 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。
N 追加下一个输入行到模板块后面并在二者间嵌入一个新行，改变当前行号码。
p 打印模板块的行。
P(大写) 打印模板块的第一行。
q 退出Sed。
b lable 分支到脚本中带有标记的地方，如果分支不存在则分支到脚本的末尾。
r file 从file中读行。
t label if分支，从最后一行开始，条件一旦满足或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾。
T label 错误分支，从最后一行开始，一旦发生错误或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾。
[w](http://man.linuxde.net/w "w命令") file 写并追加模板块到file末尾。  
W file 写并追加模板块的第一行到file末尾。  
! 表示后面的命令对所有没有被选定的行发生作用。  
= 打印当前行号码。  
# 把注释扩展到下一个换行符以前。  </pre>
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
### sed替换标记
g 表示行内全面替换。  
p 表示打印行。  
w 表示把行写入一个文件。  
x 表示互换模板块中的文本和缓冲区中的文本。  
y 表示把一个字符翻译为另外的字符（但是不用于正则表达式）
\1 子串匹配标记
& 已匹配字符串标记
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
```
### 替换指令格式
```
sed 's/book/books/' file
```
**-n选项**和**p命令**一起使用表示只打印那些发生替换的行：
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
`sed -n 's/[test](http://man.linuxde.net/test "test命令")/TEST/p' file`
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
直接编辑文件**选项-i**，会匹配file文件中每一行的第一个book替换为books：
```
sed -i 's/book/books/g' file
```
所以根据上面的`grep`指令，我们可以使用`sed`来批量替换：
`sed -i "s/原始字符/替换字符/g" grep -r -l  "原始字符" 要替换的文件夹路径`

##### 另外：
```
-i extensionEdit files in-place, saving backups with the specified extension. 
If a zero-length extension is given, no backup will be saved. 
It is not recommended to give a zero-length extension when in-place editing files, 
as you risk corruption or partial content in situations where disk space is exhausted, etc.

翻译：就地替换文件，根据提供的扩展名保存源文件备份。如果不提供扩展名，则不备份。
建议替换操作时提供文件备份的扩展名，因为如果恰巧磁盘耗尽的话，你将冒着原文件被损坏的风险。
```



所以，如果我们不需要备份的话，可以这样

`sed -i “” “s/string_old/string_new/g” grep -rl string_old ./`

或者要备份原文件

`sed -i “.bak” “s/string_old/string_new/g” grep -rl string_old ./`
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
#### 参考
[find指令](https://www.cnblogs.com/lanchang/p/6597372.html)
[ls资料](https://askubuntu.com/questions/225621/how-to-search-for-all-the-files-starting-with-the-name-abc-in-a-directory)
[find和grep](https://superuser.com/questions/345101/bash-is-there-a-way-to-search-for-a-particular-string-in-a-directory-of-files)
[replace](https://www.linux.com/learn/vim-tips-basics-search-and-replace)
[find&regular](https://www.cnblogs.com/jiangzhaowei/p/5451173.html)
