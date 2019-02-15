---
layout: post
title: git 操作及原理
date: 2017-03-12 17:02:29
tags: Tool
---
最近全面整理下工作中遇到的git问题，了解了下git工作的原理。
<!-- more -->
### TODO概念解析

工作区，暂存区，远端

一些blob，tree，commit对象的内部概念

.git文件夹下的内容

git merge origin/other\_branch时有时会自动merge
可以故意commit一个有冲突的再merge就可以显示merge的全部内容了

WIP：在进行中，避免被merge

参考书籍[http://iissnan.com/progit/](http://iissnan.com/progit/)

#### HEAD

可以看到`.git/HEAD`中内容

当在master分支时，内容为`ref: refs/heads/master`

当在master分支时，内容为`ref: refs/heads/master`

### 配置

```bash
# 列出所有config
git config --list

git config --global user.name 'Eva'

git config --global user.email 'eva@163.com'

# 换行符转为crlf
git config --global core.autocrlf ture

# 配置git last为显示上次提交信息，git last -p
git config --global alias.last 'log -1 HEAD'
```

### 生成私钥，添加在setting里

```bash
# git clone时ssh的方式，这样就免去了每次提交输密码

# 查看私钥
cd ~/.ssh

# 生成私钥
ssh-keygen -t rsa -C "zhangchu@xiaomi.com"
```

### 新建repository

```
cd d/workspace

mkdir demo

git init

git remote add origin 'https://github.com/EvaLotus/EvaLotus.github.io'

# 第一次需要-u
git push -u origin/master
```

### commit

```
# 查看修改新增了哪些文件
git status

# 提交之前查看下修改了啥，不会列出新增的文件
git diff

# 具体文件修改了啥
git diff a.txt


# 丢弃所有修改
git checkout -- .

# 只是添加到暂存区下管理，被管理
git add file.txt

git add . //新增和修改的文件都会有，删除不会有，所有文件添加到暂存区

git add -u //已经被添加的文件的修改

git add -A //是上面两个功能的合集（git add --all的缩写）

git add -f//强制add，把gitignore里的也会添加

# 删除某文件
git rm
git rm --cached # 从tracked变成untracked
git rm -f # 必须是tracked的才能使用，直接删除文件

//暂存

git stash

git stash pop//恢复暂存

git status//查看哪些没提交的

git commit -m "my 这里是注释"

git commit -am "这里是注释"//多个提交时用-a
```

### 回退

```
git log//先查看log

# reset
git reset --hard HEAD~~几个~就是回退几个版本，此处是本地回退

git reset –hard HEAD^ //回退到上个版本

git reset -hard HEAD~100//回退到上100个版本

# revert

git push -f origin master就是强制push到远端
```

### Branch

```
git branch -a //查看所有分支（包括远端和本地）
# 查看本地分支
git branch

# 查看所有远端分支
git branch -r

# 查看本地已merged的分支
git branch --merged

# 查看本地未merged的分支
git branch --no-merged

# 新建分支，但是不切换到此分支
git branch mybranch

# 删除本地分支，可删除多个
git branch -d mybranch1 mybranch2

# 强制删除本地未merge分支
git branch -D mybranch

# 删除远端分支
git push origin :mybranch
# 或
git push origin --delete mybranch
```

#### 关于删除远端分支注意：

删除远端分支后`另一个用户`并不能获取到分支被删除了,`git branch -r`也依然能获取到被删除的分支

这说明，remotes/origin/\* 这些远程跟踪分支，仅仅是远程分支的一个缓存，并且，不能通过git fetch 命令获取到分支删除的更新

可以通过`git remote show origin`来查看，会列出所有分支状态

```
branchA tracked
refs/remotes/origin/deletedBranchA stale (use 'git remote prune' to remove)
refs/remotes/origin/deletedBranchB stale (use 'git remote prune' to remove)
```

被删除的分支`refs/remotes/origin/deletedBranch`状态是`stale`\(陈旧的\)

后面有提示

```
git remote prune origin
# 或
git fetch -p
```

### Stash

```
# 暂存
git stash

# 列出所有暂存
git stash list

# 列出上次暂存
git stash show

# 恢复暂存，删除list中的
git stash pop stash@{0}

# 只恢复暂存，list中不删除
git stash apply stash@{0}
```

### checkout

```
# 在本地新建一个分支test并切换到此分支
git checkout -b test

# 切换已有的远端分支或本地分支
git checkout test

# 放弃工作区修改的某文件
git checkout -- a.js

# 放弃所有工作区文件的修改，未commit的，不影响未add的
git checkout -- .
```

### merge

```
git checkout dev

git merge origin/master //把master merge到 dev

git push origin test //在远程仓库也新建一个分支test

# cherry-pick
git cherry-pick commitId

# 和rebase的区别是？rebase的commit history会更干净一点
git rabase
```

### Reflog和log

```
# 只列出commit的log
git log

# 列出所有操作，包括pull，checkout
git reflog

# 查看是谁修改的
git blame file
```

git的HEAD是当前活跃分支游标

全局问题

git log

退出git log在英文状态下按Q

使用小乌龟的log来查看历史

```
git fetch //fetch和pull的区别，fetch不会自动merge更安全点

git log //在英文状态下按Q退出

git rm

origin算是仓库名，可以命名为其他的

git pull origin



关于git remote对应的都是仓库

git的origin指向的是本地的代码库托管在github的版本

git remote -v

git remote存在的远端分支 origin

git remote add 就是说可以创建多个远程仓库

git remote rm repositoryName删除远程仓库

git remote rename eva origin 重命名远程仓库

git remote add https://github.com/EvaLotus/test.git 在本地仓库添加一个远程仓库，并将master跟踪到远程分支



git push origin mybranch:master把我的分支push到远端哪个分支上

git push origin mybranch:staging
```

gitlab网站上发出merge request

```
# 带*的表示当前分支
# 回退到merge前
git reset --merge
```

* sourceTree请使用英文版的，不然太难用啦！！

#### gitlab中的权限管理

只有owner在setting里可以邀请member，给member设置master的权限，否则是protected状态

#### svn和git的区别

* SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。

* Git是分布式版本控制系统，那么它就没有中央服务器，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

### 基于 Git Flow 的开发流程

#### Git Flow 分支模型

* master 分支 =&gt; 正式环境（最为稳定功能最为完整）
* test 分支（release分支） =&gt; 测试环境（发布定期要上线的功能）
* dev 分支 =&gt; 团队协作的发开环境（功能最新最全的分支）
* hotfix 分支 =&gt; 修复线上代码的 bug
* feature分支 =&gt; 某个功能点正在开发阶段
* #### 开发阶段
* 开发 on your branch

* gitlab上发起merge request

* code review，accept merge request，delete branch

#### 测试阶段

1. 完成新功能后，dev 分支提交到 test 分支（release分支），进行测试
2. 有bug：创建hotfix 分支（修复后合并到 test 分支）

#### 发布上线

1. release 分支合并进 master 和 develop
2. 上线
3. 线上环境小 bug：创建 hotfix 分支进行修改，大bug：版本回滚

#### 分支命名

参考：

* feature——按照功能点（而不是需求）命名；
* test\(release\)——用发布时间命名，可以加上适当的前缀；
* hotfix——GitLab 的 issue 编号或 bug 性质等。

#### Commit Message格式

star多的项目都有完善的文档体系和高覆盖的测试用例

#### type

* feat：新功能（feature）

* fix：修补bug

* docs：文档（documentation）

* style： 格式（不影响代码运行的变动）

* refactor：重构（即不是新增功能，也不是修改bug的代码变动）

* test：增加测试

* chore：构建过程或辅助工具的变动

#### Scope

用来说明本次Commit影响的范围，即简要说明修改会涉及的部分。这个本来是选填项，但从AngularJS实际项目中可以看出基本上也成了必填项了。

用来简要描述本次改动，概述就好了，因为后面还会在Body里给出具体信息。并且最好遵循下面三条:

#### Subject

* 以动词开头，使用第一人称现在时，比如change，而不是changed或changes

* 首字母不要大写

* 结尾不用句号\(.\)

#### Body

&lt;body&gt;里的内容是对上面subject里内容的展开，在此做更加详尽的描述，内容里应该包含修改动机和修改前后的对比。

#### Revert

此外如果需要撤销之前的Commit，那么本次Commit Message中必须以revert：开头，后面紧跟前面描述的Header部分，格式不变。并且，Body部分的格式也是固定的，必须要记录撤销前Commit的SHA值。

git pull = git fetch +git merge

在git merge时会自动生成个merge的时间节点在commit记录里

#### rebase和merge

两者都是用来合并分支，细节处理上有些不一样

\[rebase\]\([https://mp.weixin.qq.com/s?\_\_biz=MzAwNDYwNzU2MQ==∣=400938481&idx=1&sn=f4d92674ebf00c0a208936e6467c3da1&scene=21\#wechat\_redirect\](https://mp.weixin.qq.com/s?__biz=MzAwNDYwNzU2MQ==&mid=400938481&idx=1&sn=f4d92674ebf00c0a208936e6467c3da1&scene=21#wechat_redirect%29\)
### git迁移

```
git remote set-url origin git@v9.git.n.xiaomi.com:infosec/midun_ui_web.git
git remote set-url --push origin git@v9.git.n.xiaomi.com:infosec/midun_ui_web.git
```

### git hook自动部署

ssh git@gitlab.com -T

#### git迁移

#!/bin/bash
git checkout master
git pull origin master --all

for branch in `git branch -a | grep remotes | grep -v HEAD | grep -v master `; do
git branch --track ${branch#remotes/origin/} $branch
done

export new_repo=$1
git remote set-url origin $new_repo
git remote set-url --push origin $new_repo
git remote -v
git push -u origin --all
git push -u origin --tags

mind map

![](/assets/git.png)
