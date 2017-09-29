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

## Git 切换到某一个 commit 版本
如果使用 `git reset --hard HEAD^`, 导致当前workspace 更改, 可以使用 
```
// 先找到commit id
git reflog
// 切换到那个版本的workspace
git reset --hard 3628164
``` 

## git 相关操作
做merge、pull之前一定要先把自己的更改备份，可以使用以下方法
```
git checkout -b tianqi05/deploy-binary-1.2.3
git reset --hard origin/release/1.2.3
git diff >> a.diff
git apply a.diff
```
### git blame
`git blame meituan/server/clouds/common/macutils.py`

### rebase操作做合并
```
git checkout experiment
git rebase master
```

假设master 和 experiment的公共祖先节点是 a， 以上两句是将 experiment 分支首先 git diff 保存从 a 开始的 变化，然后重置到 a， 执行master从a开始的变化，最后apply diff。

```
git checkout master
git merge experiment
```
切回主分支，执行合并操作，将开发分支合并到主分支，此时，就不用处理合并冲突了，因为此时的 experiment是在master的前面。

pull默认是fetch和merge加起来操作，所以一般先使用 fetch 比较好
```
git pull = git fetch + git merge
git pull origin master
git pull team1 dev
```

默认推送当前名字叫 master 的本地分支到 origin/master
```
git push origin master
git push origin tracking:upstream
```
## git rebase -i & squash 重演合并多个commit信息成为一个commit信息
```
git rebase -i 2323123
```

### git track
本地分支（该分支成为 tracking branch）跟踪远程分支（该分支被称为 upstream branch）
git clone 会自动在在本地将本地的 master 分支设置跟踪 origin/master
跟踪方法：
1. 首先拉取远程分支信息
`git fetch`
2. 新建一个新的本地分支跟踪一个远程分支
`git checkout -b localbranchname origin/dev`
3. 修改已存在本地分支或者设置已存在本地分支的跟踪分支(upstream)
`git branch --set-upstream-to=origin/branch currentbranch`
4. 删除远程分支
`git push origin --delete branchname`
5. 展示分支详细信息
`git branch -vv`

### git diff 支持通配符
```
git diff --cached origin
git diff --*.m
```
git apply -v changes.diff 失败
可以patch成功的，失败的手动改
git apply --reject --whitespace=fix mypatch.patch

### 本地编辑代码直接推送到服务器上调试（ssh & rsync）
sync.sh用来做文件同步的，可以方便本地修改代码同步到远程服务器，然后在远程服务器上调试
```bash
#!/bin/bash
action=${1:-"upload"}

declare -A hosts
hosts=(

# 网络测试环境
["yf-cloud-netctr03"]="10.4.225.98"
#["yf-mos-test-net09"]="10.4.253.23"
#["yf-mos-corp-host47"]="10.4.250.37"

)


if [[ $action == "upload" ]]; then
    prompt=""
    for host in "${!hosts[@]}"; do
        prompt+="$host "
    done
    prompt="The hosts you are going to deploy are:\033[32m $prompt \033[0m"
    echo -e $prompt
    read -p "Continue (y/n)" answer
    if [[ ! $answer =~ ^(y|Y)$ ]]; then
        exit 1
    fi
    for host in "${!hosts[@]}"; do
        echo $host ${hosts["$host"]}
        rsync -av -e 'ssh -p 44392' --exclude '*.pyc'  --exclude 'contrib' --exclude 'workspace/*' --exclude '.git/*' --exclude 'backup'  . tianqi05@${hosts["$host"]}:/data/cloud
        #rsync -av -e 'ssh -p 44392' --exclude '*.pyc' ./contrib/glance sankuai@${hosts["$host"]}:/opt/cloud/contrib/
    done
elif [[ $action == "download" ]]; then
    ::
fi

```

## 代码风格检查
这个可以不用设置，要不然 git commit 的时候对于不严格的风格会失败。在 `.git/hooks/` 目录下加入`pre-commit`文件内容如下，需要事先安装好 flake8 和 pep8 两个包。
```bash
#!/bin/bash
function python_style_check() {
    #check python code in a git repo
    root=`git rev-parse --show-toplevel`
    exit=0
    for file in `git diff --name-only --cached --diff-filter=ACMRTUXB HEAD|grep '\.py$'`; do
        flake8 --config=$root/.flake8 $root/$file
        if [ $? -ne 0 ]; then
            let exit=1
        fi
    done
    exit $exit
}
python_style_check
```
不设置的话，直接去掉这个文件即可。