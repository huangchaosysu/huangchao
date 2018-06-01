---
layout: single
permalink: /git/18/
title: "分支管理策略"
excerpt: "分支管理策略"
sidebar:
  nav: "gitnav"
# toc: true
---

通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

下面我们实战一下--no-ff方式的git merge：

首先，仍然创建并切换dev分支：
```
$ git checkout -b dev
Switched to a new branch 'dev'
```
修改readme.txt文件，并提交一个新的commit：
```
$ git add readme.txt 
$ git commit -m "add merge"
[dev f52c633] add merge
 1 file changed, 1 insertion(+)
```
现在，我们切换回master：
```
$ git checkout master
Switched to branch 'master'
```
准备合并dev分支，请注意--no-ff参数，表示禁用Fast forward：
```
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```
因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去。

合并后，我们用git log看看分支历史：
```
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master) merge with no-ff
|\  
| * f52c633 (dev) add merge
|/  
*   cf810e4 conflict fixed
...
```
可以看到，不使用Fast forward模式，merge后就像这样：  
![](/assets/images/gitimg/30)

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

所以，团队合作的分支看起来就像这样  
![](/assets/images/gitimg/31)

批注： 关于分支的管理策略，给大家推荐git-flow这个工具, 不过也可以不用这个工具，但是按照其工作逻辑来自己管理，git-flow这个工具只是简化了操作。其分支管理策略如下:

![](/assets/images/gitimg/32)

* Master分支存放所有正式发布的版本，可以作为项目历史版本记录分支
* Develop分支为主开发分支
* Feature/xx分支为新功能分支，feature分支都是基于develop创建的，开发完成后会合并到develop分支上
* Release分支基于最新develop分支创建，当新功能足够发布一个新版本(或者接近新版本发布的截止日期)，从develop分支创建一个release分支作为新版本的起点，开发完成后合并到master并打上版本号，同时也合并到develop，更新最新开发分支
* Hotfix分支基于Master分支创建，对线上版本的bug进行修复，完成后直接合并到master分支和develop分支