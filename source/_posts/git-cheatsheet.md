---
title: git-cheatsheet
date: 2016-09-02 11:49:51
updated: 2016-09-02 11:49:51
tags:
- git
categories:
---

![GIT](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)
- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库
<!-- more -->

### 新建代码库
```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

### 配置
Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。
```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```
### 增加/删除文件
```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```
### 代码提交
```
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

### 分支
```
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

### 标签
```
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```
### 查看信息

```
	
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

### 远程同步
```
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all

```
### 撤销

```
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 恢复暂存区的所有的php文件到工作区
git checkout *.php

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

### 其他
```
#生成一个可供发布的压缩包
git archive

```

---

### Git Basics
- git init Initialize a repository
- git status Show status of working tree
- git add file.txt Start tracking file.txt
- git add main.txt Stage modified file main.txt
- git diff Show what's changed but not yet staged
- git commit Commit changes
- git commit -a Stage files and commit
- git mv main.txt file.txt Rename main.txt to file.txt
- git fetch develop Pull data from remote 'develop' without merging
- git pull origin develop Fetch and merge branch 'develop' from origin
- git clone url Create local copy of remote repository at 'url'

### Branching
- git branch Show current branches
- git push origin master Push master branch to origin server
- git branch -v Show last commit on all branches
- git checkout master Switch to branch 'master'
- git branch feature1 Create new branch called 'feature1'
- git checkout -b feature2 Create branch 'feature2' and switch to it
- git branch -d mybranch Delete branch 'mybranch'
- git branch --merged Show branches already merged into current branch
- git branch --no-merged Show branches not yet merged into current branch
- git branch -D fix Force delete branch 'fix' that is not yet merged
- git push origin feature1 Push local branch 'feature1' to origin
- git push staging develop:master Push develop branch to remote staging master
- git checkout -b fix1 origin/fix1 Create local branch 'fix1' based off origin branch
- git checkout --track origin/fix2 Create tracking branch 'fix2' based off origin
- git push origin :fix2 Delete remote branch 'fix2' from origin

```
### Merging / Rebasing
- git mergetool Use graphical merge tool
- git commit Finalize merge after resolving conflicts
- git merge feature1 Merge branch 'feature1' with current branch
- git add file.txt Mark file.txt as resolved after merge
- git rebase develop Rebase changes made on current branch over develop
- git rebase master develop Rebase master onto develop without checking it out
- git rebase --onto master 1a 1b Rebase master onto branch 1b made from branch 1a
### Remotes
- git remote Show remote servers you have configured
- git remote -v Show remote servers with URL displayed
- git remote add myurl url Add remote server 'url' with shortname 'myurl'
- git remote rename server1 server2 Rename remote 'server1' to 'server2'
- git remote rm server1 Remove remote 'server1'
- git remote show origin Show info about remote origin
### Commit Logs
- git log Show commit logs
- git log -p -2 Show last two commits with diffs
- git log --stat Show commit logs with stats
- git log --pretty=oneline Show commit logs one per line
- git log --graph Show commit logs with ascii graph
- git log --since=1.week Show commit log for the last week
- git blame -L 10,15 file.rb Show prev commits for each lines 10-15 of file.rb
### Undo / Change History
- git rm --cached main.txt Remove main.txt from staging but keep in working
- git commit --amend Replace last commit with whats in staging
- git checkout -- file.txt Discard changes to file.txt
- git reset HEAD file.txt Unstage file.txt
- git commit --amend Modify last commit message
- git rebase -i HEAD~3 Make changes to the last 3 commits
### Using Tags
- git tag Show available tags
- git tag -a v3.0 Create annotated tag 'v3.0'
- git show v3.0 Show info for tag v3.0
- git tag -s v3.0 Create signed tag v3.0
- git tag v2.1-lw Create lightweight tag v2.1
- git tag -v v3.0 Verify signed tag v3.0
- git tag -a v2.2 8feb Tag previous commit '8feb' as v2.2
- git push origin v2.2 Push tag v2.2 to origin
- git push origin --tags Push all local tags to origin
### Using Stashes
- git stash Stash changes without committing
- git stash list Show stores stashes
- git stash apply Reapply most recent stash
- git stash apply stash@2 Reapply stash 2
- git stash apply --index Reapply stashed changes along with staged changes
- git stash drop stash@{2} Drop stash 2
- git stash pop Apply most recent stash and drop from stack
- git stash branch mybranch Create branch 'mybranch' from stash
- git stash clear Delete all stashes
- git diff --staged Show what's staged but not yet committed
- git diff --check Check for whitespace errors before committing

### Using Bisect
- git bisect start Start binary search of commits to find bad commit
- git bisect bad Mark current commit as broken during bisect
- git bisect good v2.2 Mark v2.2 as last known good commit during bisect
- git bisect good Mark current commit as good during bisect
- git bisect reset Reset HEAD when finished with bisect
- git bisect run test.sh Run 'test.sh' on each commit during bisect

[参考](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)
[参考](https://www.shortcutfoo.com/app/dojos/git/cheatsheet)