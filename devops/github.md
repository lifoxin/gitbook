### [git基本操作](https://git-scm.com/book/zh/v2)
- git checkout  功能是本地所有修改的。没有提交的，都返回到原来的状态
- git reset --hard HASH功能是返回到某个节点，不保留修改。
- git reflog  查看所有修改
- git branch -v 如果需要查看每一个分支的最后一次提交
- git tag v1.4-lw  创建简单标签
- git tag -a v1.4 -m 'my version 1.4'   创建一个附注标签
- git tag -a v1.2 9fceb02  给过去提交内容打标签
- git push origin v1.5    推送远程仓某个标签
- git push origin --tags  推送远程仓所有标签
- git show v1.4 查看标签内容
- git tag 列出标签
### 第三方合并
- git checkout -b experiment  第三方合并
- git commit -a -m "xx"
- git checkout master
- git merge experiment
#### 变基1
- git checkout experiment   变基的用法
- git rebase master
- git checkout master
- git merge experiment
### 变基2
- git rebase --onto master server client   master变基合并server下的client
- git checkout master
- git merge client                         master合并client
### 变基3
- git rebase master server                省去你先切换到 server 分支，再对其执行变基,合并server
- git checkout master
- git merge server
- git branch -d client     删除分支
- git branch -d server

### 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [远程branch]:[本地branch]

### 上传本地指定分支到远程仓库
$ git push [remote] [本地branch]:[远程branch]


