---
title: Git常用命令
date: 2016-06-23 21:01:21
categories: Git
tags: Git命令
---
这篇博客介绍和记录了一些Git常用命令
<!-- more -->
## Git安装
**Windows:** [msysgit](https://git-for-windows.github.io/)
**Linux:** `sudo apt-get install git`
**Mac:** 安装`Xcode`

----------

## Git配置
**配置用户**
```bash
# 全局配置
$ git config --global user.name "UserName"
$ git config --global user.email "email@example.com"

# 针对某个仓库配置
$ git config user.name "UserName"
$ git config user.email "email@example.com"
```
**配置颜色**
```bash
# 显示彩色
$ git config --global color.ui true
```
**配置别名**
```bash
# 单一别名
$ git config --global alias.st status

$ git config --global alias.co checkout

$ git config --global alias.ci commit

$ git config --global alias.br branch

# 组合别名
$ git config --global alias.unstage 'reset HEAD'

$ git config --global alias.last 'log -1'

$ git config --global alias.cob 'checkout -b'

$ git config --global alias.psm 'push origin master'

$ git config --global alias.plm 'pull origin master'

$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```
**配置编辑器**
```bash
$ git confog --global core.editor "<editor-name>"
```
----------

## 创建版本库
**操作文件**
```bash
# 创建目录
$ mk <dir-name>

# 进入目录
$ cd <dir-name>

# 创建文件
$ touch <file>
```
**初始化Git仓库**
```bash
$ git init
```
**添加文件到仓库**
```bash
# 添加单个文件
$ git add <file>

# 添加所有文件
$ git add .
```
**提交文件到仓库**
```bash
$ git commit -m "wrote a message"
```
**查看仓库状态**
```bash
$ git status
```

----------


## 对比文件
**工作区和暂存区**
```bash
$ git diff <file>
```
**工作区和版本库**
```bash
$ git diff HEAD -- <file>
```
**暂存区和版本库**
```bash
$ git diff --staged <file>
```
**提交比较**
```bash
# commit2与commit1对比

# 比较两次提交区别
$ git diff <commit1-id> <commit2-id>

# 比较两次提交某个文件区别
$ git diff <commit1-id> <commit2-id> <file>
```
**分支比较**
分支比较时，只比较版本库中的内容，也就是比较不同分支最后一次commit的内容。
```bash
# branch2与branch1对比

# 比较两个分支当前内容
$ git diff <branch1-name>..<branch2-name>

# 如果branch1是branch2的直接父分支，则branch2与branch1的分歧点commit对比
# 如果branch1不是branch2的直接父分支，则追溯到两个分支的共同父分支，然后branch2与该父分支的分歧点commit对比
$ git diff <branch1-name>...<branch2-name>

```

----------


## 版本回滚
**查看提交历史版本号**
```bash
# 回滚前
$ git log
# 精简信息
$ git log --pretty=oneline
# 回滚后
$ git relog
```
回滚前，`git log`命令可以查看所有的提交历史，但回滚后，回滚到的`commit`只有的提交`git log`命令无法查看到，这时需要使用`git relog`命令。
**版本回滚**
```bash
# 回滚到上个版本
$ git reset --hard HEAD^

# 回滚到上上个版本
$ git reset --hard HEAD^^

# 回滚到上100个版本
$ git reset --hard HEAD~100

# 回滚到某个版本
$ git reset --hard <Commit Id>
```

----------


## 撤销修改
**撤销文件修改**
```bash
$ git checkout -- <file>
```
该命令可以将工作区的修改全部撤销，这里有两种情况：
1. 文件修改后还未添加到暂存区，撤销修改就回到和版本库中文件相同的状态
2. 文件添加到了暂存区，又修改了文件，撤销修改就回到了回到了添加到暂存区后的状态
总之，就是让文件回到最近一次`git commit`或者`git add`时的状态

**清空暂存区文件**
```bash
$ git reset HEAD <file>
```
**应用场景**
撤销修改有三种应用场景：

```bash
# 场景一：工作区文件进行了修改，想直接丢弃工作区的修改
$ git checkout -- <file>
```

```bash
# 场景二：工作区文件进行了修改，并添加到了暂存区，想丢弃修改
$ git reset HEAD <file>
$ git checkout -- <file>
```

```bash
# 场景三：工作区文件进行了修改，添加到了暂存区，并提交到了版本库，想丢弃修改，则需要进行版本回滚
$ git reset --hard HEAD^
```

----------


## 删除文件
**删除工作区文件**
直接在文件管理器中删除文件，或者执行命令：
```bash
$ rm <file>
```

**删除版本库文件**
删除工作区文件后，执行命令：
```bash
$ git rm <file>
$ git commit -m "wrote a message"
```
**撤销删除文件**
```bash
$ git checkout -- <file>
```
删除也可作为一种对文件的修改，所以可以使用撤销修改的方法撤销删除，撤销删除文件会丢失最后一次提交后做出的修改

----------

## 远程仓库

远程分支
指的是远程仓库上的分支，也就是指服务器上仓库的分支

远程跟踪分支
远程跟踪分支是远程分支状态的引用，是不能移动的本地引用，有任何的网络操作时，会自动移动，像是远程分支在本地的快照，形式为`<repository-name/branch-name>`，比如`origin/master`,当关联远程库和本地库时,会自动在本地创建远程仓库的快照，并创建远程跟踪分支。Git操作远程仓库时并不是直接远程操作，而是通过远程仓库在本地的快照，操作远程跟踪分支来实现与远程仓库的互动。

跟踪分支
跟踪分支也叫做上游分支，指的是与远程跟踪相关联的本地分支。当从远程跟踪分支检出时，会自动创建跟踪分支，跟踪分支可以直接使用`git pull`执行更新

**创建SSH Key**
```bash
ssh-keygen -t rsa -C "email@example.com"
```
该命令会在用户目录下生成`.ssh`文件夹，里面有密钥`id_rsa`和公钥`id_rsa.pub`两个文件，将公钥中的值配置到GitHub中


**查看远程库信息**
```bash
$ git remote

# 详细信息
$ git remote -v
```

**克隆远程库**

```bash
$ git clone git@github.com:username/repo.git
```
克隆远程库到本地库，Git自动完成本地库对远程库的跟踪，并在本地创建跟踪分支`master`与远程跟踪分支`origin/master`自动关联，可以直接推送更新或者拉取更新

**添加远程库**
首先在GitHub上创建与本地同名的仓库，关联远程库和本地库：
```bash
# origin为远程库名，可以自定义
$ git remote add origin git@github.com:username/repo.git
```

**创建远程分支**
```bash
$ git push origin <branch-local-name>:<branch-remote-name>
```

**关联分支**
```bash
# 关联当前分支与远程跟踪分支
$ git branch -u origin/branch-name

# 关联某个分支与远程跟踪分支
$ git branch --set-upstream branch-name origin/branch-name
```

**查看跟踪分支**
```bash
$ git branch -vv
```
该命令主要是用来查看本地分支是否关联了远程分支。

**抓取**
```bash
# 抓取某个远程库数据到本地
$ git fetch <repository-name>

# 抓取所有远程库
$ git fetch --all
```
更新远程库有而本地没有的数据，也就是更新远程库在本地的快照

**拉取**
```
# 不推荐使用
$ git pull

## 等同于
$ git fetch
$ git merge
```

**推送分支**
```bash
# 当远程仓库为空，本地有master分支,创建远程master分支
$ git push -u origin master
# 相当于要创建远程分支，等同如下命令
$ git push -u origin master:master

# 当远程仓库默认创建master分支
$ git branch -u origin/master # 关联
#或者
$ git branch --set-upstream master origin/master
$ git pull # 拉取
$ git push origin master # 推送
# 或使用别名
$ git psm

```

**检出远程分支**
如果远程仓库`origin`有一个`test`分支而本地没有
```bash
# 检出远程分支，合并到当前dev分支
$ git fetch origin # 更新远程库在本地的快照
$ git merge origin/test # 合并本地的远程跟踪分支

# 检出并在本地创建test跟踪分支
$ git checkout -b test origin/test
# 相当于下面两条命令
$ git branch -b dev origin/dev
$ git checkout dev
# 或者简写
$ git checkout --track origin/test
```

**删除远程分支**
```bash
$ git push origin --delete  <branch-remote-name>
# 或者
$ git push origin :<branch-remote-name>
```

----------


## 分支管理
**查看本地分支**
```bash
$ git branch
```
**查看分支合并图**
```branch
$ git log --graph --pretty=oneline --abbrev-commit

# 或者使用别名
$ git lg
```
**创建本地分支**
```bash
# 创建并切换分支
$ git checkout -b dev

# 相当于下面两条命令
$ git branch dev
$ git checkout dev
```
**切换本地分支**
```bash
$ git checkout dev
```

**merge合并分支**
```bash
# 合并dev分支到当前分支
# 在可能的情况下使用Fast forward模式
$ git merge dev

# 禁用Fast forward模式
$ git merge --no-ff -m "wrote a message" dev

# 合并分支有冲突时,解决完冲突需要重新添加并提交
$ git merge dev
$ git add .
$ git commit -m "wrote a message"

# 合并分支时，将dev分支commit历史合并到新的commit
$ git merge --squash dev
$ git add .
$ git commit -m "wrote a message"
```

使用`git merge`命令合并时：

- `dev`分支所有提交历史会被保存
- 新建`dev`分支后，如果`master`未进行提交，合并分支时，默认会使用`Fast forward`模式，即将`master`指针直接指向`dev`分支最新的提交，删除分支后，分支信息会丢失，看不出来进行过合并
- 新建`dev`分支后，当`master`分支和`dev`分支分别进行了提交时，合并时可能会产生冲突，这时会采用非`Fast forward`模式进行合并，需要进行一次新的提交，删除`dev`分支后，分支信息不会丢失，仍可看出进行过合并
- 将`dev`分支合并到`master`后，两个分支的提交历史会按照时间顺序合并在一起
- 可以使用`git merge --squash`合并`dev`分支的提交历史

**变基合并分支**
```bash
# 步骤一，变基，并修改冲突
$ git rebase master
# 步骤二，添加修改
$ git add .
# 步骤三，继续变基
$ git rebase --continue
...
# 重复步骤二、三
...
# 步骤四，切换分支
$ git checkout master
# 步骤五，快速合并
$ git merge dev
```
使用变基合并分支时需要注意：

- 变基的实质是丢弃现有的提交内容，将`master`分支作为基底进行合并，合并过程中，从`dev`分支首次提交开始，与`master`分支的最新提交进行冲突合并，合并后作为最新的提交，然后用`dev`分支的下一个提交与该最新提交进行冲突合并，循环执行该过程，变基结束后，`dev`分支其实就是合并后要得到的`master`分支
- `dev`分支变基后，`master`分支的提交历史会合并到`dev`分支的提交历史，并且`dev`分支的提交作为新提交
- `master`分支合并变基后的`dev`分支使用的是`Fast forward`模式
- 采用变基合并分支，可以使`master`分支的提交历史保持整洁，且保持线性
- 如果`dev`分支有对应的远程分支，则不适用变基，因为变基会更改提交历史和提交内容，协作时会产生冲突

**删除分支**
```bash
# 直接删除
$ git branch -d dev

# 强制删除
$ git branch -D dev
```
当两个分支内容不同时，如果未进行合并，则删除分支时需要强制删除；如果两个分支内容相同，即使未合并，也可以直接删除。


----------


## 暂存工作现场
**暂存**
```bash
# 可以多次暂存
# 最后stash的id为stash@{0}
# 依次类推
$ git stash
```
**查看**
```bash
$ git stash lis
```
**还原**
```bash
# 还原并删除stash内容
$ git stash pop

# 还原但不删除stash内容
$ git stash apply

# 还原某个stash
$ git stash apply stash@{index}

# 需另外删除stash，默认删除最近的stash
$ git stash drop

# 删除某个stash
$ git stash drop stash@{index}

# 删除所有stash
$ git stash clear
```

**应用场景**
当前正在`dev`分支开发时，工作未完成不能提交，这是需要切换回`master`分支新建`bug`分支修复bug，这是需要暂存工作现场，保持工作区clean，然后切换到`master`分支进行后续操作，结束之后切换回`dev`分支，恢复工作现场然后继续未完成的开发。暂存工作现场时需要注意：

- 当前分支工作区不是clean时，不可切换分支，但是可以创建分支
- 当前分支暂存工作现场后，其他分支也可以使用该`stash`。但不应该这么做
- 在当前分支可以删除其他分支的`stash`，但需要使用`git stasu drop`命令，`git stash pop`并不会删除`stash`。同样不应该这么做
- 暂存工作现场只暂存工作区的内容，而不暂存暂存区的内容


----------


## 开发策略
**分支策略**
开发过程中，常用的分支策略如下：
1. 要保持`master`分支的稳定，仅用来发布新版本，不在该分支上开发
2. 创建`dev`分支，该分支是不稳定的，当需要发布新版本时，将`dev`分支合并到`master`分支，在`master`分支上发布新版本
3. 每个人都基于`dev`分支创建新的分支进行开发，不时的向`dev`分支合并

**协作策略**
多人协作的工作模式通常如下：
1. 首先，使用命令`git pull`拉取更新合并
2. 如果有冲突，则解决冲突，并在本地提交
3. 如果没有冲突，或者冲突解决掉后，再用`git push`推送更新到分支

----------


## 标签管理
标签本质上就是指针，指向某一次的commit，与分支很像，但分支可以移动而标签不可以。通常在基于`dev`分支进行开发后，当需要发布新版本时，将不同分支合并到`master`分支，打上标签，作为版本的标记，然后基于`dev`分支继续进行后续的开发。
**创建标签**
```bash
# 默认给最新commit打标签
$ git tag <tag-name>

# 给某个commit打标签
$ git tag <tag-name> <commit-id>

# 创建带说明的标签
$ git tag -a <tag-name> -m "wrote a message" <commit-id>
```
**删除标签**
```bash
$ git tag -d <tag-name>
```
**推送标签到远程分支**
```bash
# 推送某个标签
$ git push origin <tag-name>

# 推送全部标签
$ git push origin --tags
```
**从远程分支删除标签**
```bash
# 先从本地删除
$ git tag -d <tag-name>
# 再从远程分支删除
$ git push orign :refs/tags/<tag-name>
```
**查看所有标签**
```bash
$ git tag
```
**查看标签信息**
```bash
$ git show <tag-name>
```
**切换标签**
```bash
# 标签就是一个指针，指向某次commit，与分支很像，使用checkout切换
$ git checkout <tag-name>
```

----------


## 忽略文件
在工作区根目录下创建`.gitignore`文件，将要忽略的文件添加到该文件中，然后将该文件提交到Git，Git会自动忽略这些文件。常用的配置文件在这里：[gitignore](https://github.com/github/gitignore)

**强制添加**
```bash
$ git add -f <file>
```
**检查忽略规则**
```bash
$ git check-ignore -v <file>
```

----------


## Git工作原理
工作区
: Working Directory，指的是加入版本控制的文件夹

版本库
: Repository，指的是工作区下的隐藏目录`.git`。里面包含了`暂存区`,Git自动创建的第一个分支`master`，和指向该分支的指针`HEAD`

暂存区
: stage/index，存在于版本库中，用于存放`git add`命令的提交

![](http://o90kgdwwt.bkt.clouddn.com/git1.png)

**工作过程**
当向版本库中添加文件时，执行过程如下:
1. 用`git add`命令将文件添加进去，实际上是将文件的修改添加到了`暂存区`
2. 用`git commit`命令将文件提交，实际上是将文件的修改提交到了当前分支


----------

## 参考资料
[Git教程-廖雪峰](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
[Git权威指南](http://www.worldhello.net/gotgit/)
[Git Community Book中文版](http://gitbook.liuhui998.com/index.html)
[Git-Recipes](https://github.com/geeeeeeeeek/git-recipes/wiki)