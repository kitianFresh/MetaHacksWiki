---
title: "git"
date: 2017-04-23 21:22
---

## repo fork notes

 - [fork-a-repo](https://help.github.com/articles/fork-a-repo/)
 - [syncing-a-fork](https://help.github.com/articles/syncing-a-fork/)
 - [creating-a-pull-request](https://help.github.com/articles/creating-a-pull-request/)

## Git出现merge或者pull与本地工作区状态冲突
```
Updating c5ba2bc..ad63656
error: Your local changes to the following files would be overwritten by merge:
    BloodTestReportOCR/static/index.html
    BloodTestReportOCR/view.py
Please, commit your changes or stash them before you can merge.
Aborting
```

方案1： git stash，他会把当前工作区的保存到一个Git栈中,当需要取出来的时候，git stash pop会从Git栈中读取最近一次保存的内容，恢复工作区的相关内容
```
git stash
git merge

git stash list： 所有保存的git栈内的备份
git stash pop： 弹出栈顶内容
git stash clear： 清空Git栈
```
方案2：放弃本地修改就可以了
```
git reset --hard
git pull
```