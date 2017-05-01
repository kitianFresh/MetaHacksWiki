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

## 解决在只能使用 HTTP 协议时候自己电脑上git push每次都要输入用户名密码问题
由于实验楼git服务器使用的是http协议而不是git或者ssh协议（希望实验楼能提供ssh协议的支持吧，这7样就可以把自己的公钥上传了）导致每一次本地git push都要输入用户名和密码；可以使用git config来配置；由于这是实验楼的一个项目，所以使用local就行了；命令如下：

```
git config --local credential.helper cache (这个默认密码只保存15分钟的)
git config --local credential.helper 'cache --timeout=3600' (可以添加时间参数修改保存时间)
git config --local credential.helper store (长期存储密码)
```