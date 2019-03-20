---
title: Git操作
date: 2019-03-20 16:32:15
tags:
---


#### Git操作
----------------
##### 常规操作
1.  git init      
2.  git add
3.  git rm
3.  git commit -m
4.  git commit -am 相当于 `git add + git commit -m`
4.  git push -u origin master
5.  git pull origin master --allow-unrelated-histories
6.  git status
7.  git log  `查看提交过的版本`
8.  git reflog `在回退以后又想再次回到之前的版本`
8.  git reset    `1文件从暂存区回退到工作区 2版本回退 `
9.  git config --list
10. git checkout -- <file name>  `1可以丢弃工作区的修改2可以切换分支`
11.  git clone
12.  git branch
13.  git checkout -b <分支名>
14.  git reset 切换到
15.  git merge --no-ff -m "添加的注释" <分支名>    不使用fast-forward合并分支
16.  git branch -d <分支名>   删除分支


-------------
1.  git tag -a <tag名> -m "tag释义" <commit id>   给指定commit id添加tag
2.  git tag -d  删除指定tag

----------------
2018年08月06日 更新
`git clone` 的时候，在后边添加`--depth`有什么用？
>  最近在学习别人家代码的时候，需要`clone`代码的，这个大家都知道，但是在clone的时候，我遇到了这个问题：

```
Cloning into 'xxx'...
remote: Counting objects: 1180, done.
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```
经过一番百度，别人说这是限制了传输时候的速度，
所以，我又来了一番操作：
```
//设置传输传输流速度为500M
git config -global http.postBuffer 524288000
```
有些时候是好使的，然而，有些时候，`The same error` :(
然后，我又百度上一番操作，
发现了别人另一个思路：`只clone最后一次commit`，**也就是今天所说的--depth**
`depth`顾名思义，就是深度的意思，`深度克隆！`
`1`的意思，就是`即表示只克隆最近一次commit`

------------------
2019年03月20日 更新：
这里测试了在GitHub创建分支 然后在本地分支内创建新文件，然后**只提交到远程分支中** （也就是说新提交的文件在远程主分支中看不到）

> 想法如下： 先在本地分支中，把此分支和远程分支创建连接： 此例中为newb分支 `git branch --set-upstream-to=origin/newb` 对应的取消连接命令如下： `git branch --unset-upstream xxx` xxx为分支名

然后上传文件，测试中使用的是`2.html`文件

```
git push origin aaa:bbb

```

其中：aaa为本地分支，bbb为远程分支 所以此处命令为： `git push origin newb:newb` 此时，就能看到2.html文件只在newb分支中展示，master分支中没有 *不知道不设置分支连接有没有影响*

另一个测试文件`1.txt`，上传时的命令为： `git push origin newb`此时并没有指定本地分支和远程分支

刚刚测试了文件`3.md` 此时取消了连接 即执行了命令：

```
git branch --unset-upstream newb
然后新建3.md文件
接着提交到本地仓库，然后提交远程仓库
git push origin newb:newb

```

此时发现： 主分支中仍然没有此文件。 说明：
**只要执行push操作的时候指定本地和远程分支，分支的关联与否都没关系！！**
#### [[link]](https://github.com/WooNoah/ReactNativeDemo/tree/newb)
