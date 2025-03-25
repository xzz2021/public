#### git 常用命令及其用法

```bash
# 设置用户名
git config --global user.name "Your Name"
# 设置邮箱
git config --global user.email "you@example.com"
# 查看配置信息
git config --list
# 初始化一个新的 Git 仓库
git init
# 克隆一个远程仓库
git clone <repository_url>

```

##### 基本操作

```bash
# 查看仓库当前状态
git status
# 添加文件到暂存区
git add <file>
# 添加所有文件到暂存区
git add .
# 提交暂存区的文件
git commit -m "commit message"
# 修改最后一次提交的信息
git commit --amend -m "new commit message"
# 移除文件  并记录此次操作
git rm <file>
# 移除文件，但保留工作区的文件 Git停止跟踪此文件
git rm --cached <file>
```

##### 分支操作

```bash
# 查看所有分支
git branch
# 创建新分支
git branch <branch_name>
# 切换到指定分支
git checkout <branch_name>
# 创建并切换到新分支
git checkout -b <branch_name>
# 合并分支
git merge <branch_name>
# 删除分支
git branch -d <branch_name>
```

##### 查看历史及文件差异

```bash
# 查看提交历史  退出q
git log
# 查看简洁的提交历史
git log --oneline
# 查看某个文件的提交历史
git log <file>
# 查看提交的差异
git diff <commit1> <commit2>
# 查看工作目录和暂存区的差异
git diff
# 查看暂存区和最后一次提交的差异
git diff --cached
```

##### 远程仓库

```bash
# 查看远程仓库
git remote -v
# 添加远程仓库
git remote add <name> <url>
# 删除远程仓库
git remote rm <name>
# 从远程仓库拉取代码
git pull <remote> <branch>
# 推送代码到远程仓库
git push <remote> <branch>
# 推送所有分支到远程仓库
git push --all <remote>
```

##### 标签

```bash
# 创建标签
git tag <tag_name>
# 创建带注释的标签
git tag -a <tag_name> -m "message"
# 查看所有标签
git tag
# 推送标签到远程仓库
git push <remote> <tag_name>
```

##### 撤销操作

```bash
# 撤销工作目录的修改 也就是未暂存时修改的内容
git checkout -- <file>
# 重置暂存区的文件 撤销已add到暂存区的指定文件
git reset HEAD <file>
# 回滚到某个提交
git log --oneline
git reset --hard <commit>
# 撤销git的最近一次commit, 但保留所有更改在工作目录中
git reset --soft HEAD~1
# 撤销git的最近一次commit, 并删除所有更改
git reset --hard HEAD~1

```

```bash
# 图形化界面
gitk
```

#####  修改 commit

```bash
# 替换提交的信息
git commit --amend -m "新的提交消息"
# 移除暂存区的文件
git rm <file>
# 重新提交
git commit --amend
# HEAD~3 表示最近的3次提交 pick edit
git rebase -i HEAD~n
```

### 代码管理

1. merge合并远程分支代码, `git fetch origin dev` 拉取远程代码到本地; `git merge origin/dev`进行合并

   ```bash
   # 查询当前远程的版本
    git remote -v
   # 查看分支的跟踪状态
    git branch -v
   # 创建本地分支并跟踪远程分支
   git checkout -b <本地分支名> <远程仓库名>/<远程分支名>
   # 将现有本地分支与远程分支关联
   git branch --set-upstream-to=<远程仓库名>/<远程分支名> <本地分支名>
   # 获取最新代码到本地(本地当前分支为[branch]，获取的远端的分支为[origin/branch])
    git fetch origin master  [示例1：获取远端的origin/master分支]
    git fetch origin dev [示例2：获取远端的origin/dev分支]
   # 查看版本差异
    git log -p master..origin/master [示例1：查看本地master与远端origin/master的版本差异]
    git log -p dev..origin/dev   [示例2：查看本地dev与远端origin/dev的版本差异]
   # 合并最新代码到本地分支
    git merge origin/master  [示例1：合并远端分支origin/master到当前分支]
    git merge origin/dev [示例2：合并远端分支origin/dev到当前分支]
   ```

2. 已被git追踪的目录.gitignore添加忽略会无效, 需要执行`git rm -r --cached 目录名`

