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

###### 修改 commit

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
