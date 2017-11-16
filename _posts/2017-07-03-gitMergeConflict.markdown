---
layout: post
title: "Git合并后再discard的问题"
date: 2017-07-03 11:01:11.000000000 +09:00
tags: 回梦游仙
---

一直在用的GitHub Desktop，上周代码搞定后，想把自己的master分支的代码合并到当前分支上，但是有冲突，我当时只是想试一下合并，懒得改冲突，就点了Xcode里`Source Control`里的`Discard All Changes`，这时候变回去了，我就顺手commit 同步了，结果是GG，与master合并不了了。。

今天有时间，为了搞明白，重新建立个工程来一遍。名为GitPractice，分支是`master`和`practice1`，先把两个分支的代码搞的不一样。

然后在`practice1`分支上合并`master`的，有冲突，然后点击`Discard All Changes`，如下图——

![](/assets/images/2017/GitPractice1.png)

此时`Update from master`还是亮着的，说明还是可以合并的，但是点击了显示如下图——

![](/assets/images/2017/GitPractice2.png)

我这时候就按着他说的commit 同步了，后来就GG了。

因为不专业，第一次遇到这种情况，不知所措，就上网搜了下，说的是——

> - 错误可能是因为在你以前pull下来的代码没有自动合并导致的.
> - 有2个解决办法:
> - 1.保留你本地的修改
> - git merge --abort
> - git reset --merge
> - 合并后记得一定要提交这个本地的合并
> - 然后在获取线上仓库
> - git pull

> - 2.down下线上代码版本,抛弃本地的修改
> - 不建议这样做,但是如果你本地修改不大,或者自己有一份备份留存,可以直接用线上最新版本覆盖到本地
> - git fetch --all
> - git reset --hard origin/master
> - git fetch

所以自己上面的步骤又来了一遍，让两个分支代码有冲突，然后合并，再discard，就到这里，用上文说的`git reset --merge`

![](/assets/images/2017/GitPractice3.png)

![](/assets/images/2017/GitPractice4.png)

果然冲突又出来了，感觉呼吸到了新鲜空气，因为找到了问题的根源，但是项目的代码就GG了，只能自己重新再改一遍，没办法，吃一堑长一智啦~


/--------------------------------------------------------------------------/

十月补充：不能在其他分支合并master，这里想合并得用git rebase

在master合并其他分支不会有问题~
