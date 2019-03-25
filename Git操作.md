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

<!--more-->
-------------
1.  git tag -a <tag名> -m "tag释义" <commit id>   给指定commit id添加tag
2.  git tag -d  删除指定tag

----------------
#### 2018年08月06日 更新
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
#### 2019年03月20日 更新：
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

-------------------
#### 2019年03月25日 更新 
#### 1️⃣`git fetch`
首先，git fetch 有四种基本用法
```
1. git fetch            
→→ 这将更新git remote 中所有的远程repo 所包含分支的最新commit-id, 将其记录到.git/FETCH_HEAD文件中

2. git fetch remote_repo         
→→ 这将更新名称为remote_repo 的远程repo上的所有branch的最新commit-id，将其记录。 

3. git fetch remote_repo remote_branch_name        
→→ 这将这将更新名称为remote_repo 的远程repo上的分支： remote_branch_name

4. git fetch remote_repo remote_branch_name:local_branch_name       
→→ 这将这将更新名称为remote_repo 的远程repo上的分支： remote_branch_name ，
并在本地创建local_branch_name 本地分支保存远端分支的所有数据。
```

git pull 的运行过程：

git pull : 首先，基于本地的FETCH_HEAD记录，比对本地的FETCH_HEAD记录与远程仓库的版本号，然后git fetch 获得当前指向的远程分支的后续版本的数据，然后再利用git merge将其与本地的当前分支合并。


首先确定一个概念：
```
① git  pull  等价于
② git fetch + git merge
```
但是目前来看，发现了一种情况，只能使用步骤②
> 如果本地创建了一个新的分支，但是远端仓库中**并没有此分支**，此时如果想要拉取远端的更新到本地的新分支，此时要怎么做？

##### 我只实验了一种情况：
```
使用git fetch origin aaa:bbb，比如：
git fetch origin master:temp //从远程的origin仓库的master分支下载到本地并新建一个分支temp
此种情况下，不过本地是否有temp分支，执行完命令之后，本地都会存在一个temp分支

然后切换到"要合并到"的分支中，执行
git merge就可以合并

然后合并完成之后，删除新建的分支即可
git branch -d temp

```
##### 但是！！！如果使用git fetch origin aaa:bbb的时候，bbb为一个已经在本地存在的分支
则此时会报错：
```
fatal: Refusing to fetch into current branch refs/heads/bbb of non-bare repository
```
##### 这里有一篇参考文章[git refusing to fetch into current branch](https://stackoverflow.com/questions/2236743/git-refusing-to-fetch-into-current-branch)
##### [真正理解 git fetch, git pull 以及 FETCH_HEAD](https://www.cnblogs.com/ToDoToTry/p/4095626.html)
##### [git fetch 更新远程代码到本地仓库](https://www.cnblogs.com/chenlogin/p/6592228.html)



#### 2️⃣ Git rebase
> Rebase is another way to integrate changes from one branch to another. Rebase compresses all the changes into a single “patch.” Then it integrates the patch onto the target branch. Unlike merging, rebasing flattens the history because it transfers the completed work from one branch to another.

如果我们的提交记录太过复杂，需要整合一下，这个时候，我们可以使用git rebase,来把以前的全部提交，整合成一条新的单一的分支，同时也可以修改以前的commit信息
```
git rebase HEAD~10  //此时就会整合最近十次的提交
```
![rebase前](https://upload-images.jianshu.io/upload_images/1241385-d69d1b3657f1194b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![rebase后](https://upload-images.jianshu.io/upload_images/1241385-b2b635cebdc40339.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**可以看到，本来分开的两个分支，结果合并成为了一个单一分支**

> 以下部分内容摘自[筱湮简书](https://www.jianshu.com/p/098d85a58bf1)


##### 修改最后一次注释

如果你只想修改最后一次注释（就是最新的一次提交），那好办：
`git commit --amend`
出现有注释的界面（你的注释应该显示在第一行）， 输入i进入修改模式，修改好注释后，按Esc键 退出编辑模式，输入:wq保存并退出。ok，修改完成。
例如修改时编辑界面的图：
![图片摘取自别处，侵删](https://upload-images.jianshu.io/upload_images/1241385-e6118455102e7d9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 修改之前的注释
修改之前的某次注释
1. 输入：
`git rebase -i HEAD~2`
最后的数字2指的是显示到倒数第几次 比如这个输入的2就会显示倒数的两次注释（最上面两行）
![图片摘取自别处，侵删](https://upload-images.jianshu.io/upload_images/1241385-5c79a86827c03a3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 你想修改哪条注释 就把哪条注释前面的`pick`换成`edit`。方法就是上面说的编辑方式：`i`---编辑，把`pick`换成`edit`---`Esc`---`:wq`.
3. 然后：（接下来的步骤Terminal会提示）
`git commit --amend`

4. 修改注释，保存并退出后，输入：
`git rebase --continue`
![图片摘取自别处，侵删](https://upload-images.jianshu.io/upload_images/1241385-5a333429e6b103c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实这个原理我的理解就是先版本回退到你想修改的某次版本，然后修改当前的commit注释，然后再回到本地最新的版本

##### 修改之前的某几次注释
修改多次的注释其实步骤和上面的一样，不同点在于：

1. 同上
2. 你可以将多个想修改的commit注释前面的pick换成edit

3. 依次修改你的注释（顺序是从旧到新），Terminal基本都会提示你接下来的操作，每修改一个注释都要重复上面的3和4步，直到修改完你所选择的所有注释


#### 3️⃣ Git tag相关
> 这里有一个场景，如果有别的小伙伴们在别的地方新打了一个tag，那我们如何在本地手动获取到呢？

`git pull --tags`

[https://stackoverflow.com/questions/1204190/does-git-fetch-tags-include-git-fetch](https://stackoverflow.com/questions/1204190/does-git-fetch-tags-include-git-fetch)


