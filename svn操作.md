---
title: svn操作
date: 2018-10-31 15:23:45
tags:
---


> 这里记录一下一些平时用到的svn指令操作

#### 1. svn log
展示修正的log信息或者路径
```
eg:
svn log ProjectName/Info.plist
就可以看到info.plist文件的修改历史
```
![](https://upload-images.jianshu.io/upload_images/1241385-253b5c8609605808.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2. svn diff
查看两个文件之间的不同
- 如果直接使用svn diff，则展示当前工作区和服务器端的不同。
- 如果想要显示特定两个版本的不同，可以用如下写法：
```
//以下就是展示7504版本和7487版本之间 info.plist文件的变化
svn diff -r 7504:7487 ProjectName/Info.plist
```

<!--more-->

#### 3. svn add 
> Put files and directories under version control, scheduling
them for addition to repository.  They will be added in next commit.
usage: add PATH...

添加文件或者文件夹到SVN

#### 4. svn checkout 
> Check out a working copy from a repository.
usage: checkout URL[@REV]... [PATH]


#### 5. svn commit
> Send changes from your working copy to the repository.
usage: commit [PATH...]

--------------------- 
#### 6. svn revert
> Restore pristine working copy state (undo local changes).
usage: revert PATH...

回滚到以前的版本

#### 7. svn update
> Bring changes from the repository into the working copy.
usage: update [PATH...]

#### 8. svn merge
> Merge changes into a working copy.
usage: 1. merge SOURCE[@REV] [TARGET_WCPATH]
(the 'complete' merge)
2. merge [-c M[,N...] | -r N:M ...] SOURCE[@REV] [TARGET_WCPATH]
(the 'cherry-pick' merge)
3. merge SOURCE1[@REV1] SOURCE2[@REV2] [TARGET_WCPATH]
(the '2-URL' merge)

> notes:

revert命令顾名思义就是对修改过的东西进行回滚操作。一般有2种情况发生时需要用到回滚的操作：

1，修改过的东西没有提交(commit)

这种情况下revert会取消之前的修改

`用法：#svn revert [-R] xxx_file_dir`

如果需要回滚的是一个目录则加上-R（递归）可选参数

2，改动的东西并且提交了

这种情况下，用svn merge命令来进行回滚。

步骤如下：
```
1）执行#svn update命令保证工作区文件是最新的，比如最新版本号是20

2）然后找出要回滚的确切版本号：

执行svn log xxx_file_dir

假设根据svn log日志查出要回滚的版本号是10，如果想要更详细的了解情况，可以使用svn diff -r 20:10 [xxx_file_dir]
3)回滚到版本号10：
执行svn merge -r 20:10 xxx_file_dir

4)提交回滚：
svn commit -m "注释..." 
提交后版本变成了29
完毕
```
--------------------- 
#### 9. svn export
> Create an unversioned copy of a tree.
usage: 1. export [-r REV] URL[@PEGREV] [PATH]
2. export [-r REV] PATH1[@PEGREV] [PATH2]

导出一个不包含版本控制的文件
#### 9. svn upgrade
> Upgrade the metadata storage format for a working copy.
usage: upgrade [WCPATH...]
在我们把服务器上的svn 版本号升级以后，在之前的文件下再执行svn命令时，会提示需要执行svn upgrade命令把当前的代码由低版本的svn 上迁移到高版本的svn 上去。 
直接执行svn upgrade命令就会把所有的代码按照最新的svn 版本重新更新一遍。之后你操作所有的svn 命令都会正常运行。

#### 10. svn ls
```
svn列出所有branches下的分支

$ svn ls svn://192.168.0.178/yewu/branches

或者显示详细内容：

svn ls svn://192.168.0.178/yewu/branches --verbose
```


#### 参考
https://blog.csdn.net/u014100559/article/details/50539232 
