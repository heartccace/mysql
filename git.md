###    GIT

git config --global



项目的版本是一个提交对象，项目的快照是一个树对象

echo 'test content' | git hash-object -w --stdin 将test content写入git数据库

git hash-object -w  文件路径 将文件纳入git数据库

git cat-file -p hash 查看hash对应的内容

git cat-file -t hash 查看hash对应类型



git ls-files -s 查看暂存区

git update-index --add --cacheinfo 100644 hash 文件名 (首次将文件加入到缓存区)



git write-tree 生成树对象，及版本对象（将所有缓存区对象生成一个快照）

git read-tree --prefix=bak 树hash 将第一棵树对象加到第二棵树对象后面生成新的树对象



提交对象

echo ‘commit’ | git  commit-tree hash

echo ‘commit’ | git  commit-tree hash -p 父hash 未提交对象指定父提交对象



git add ./ 将当前目录所有文件加入版本库，再从版本库加载到暂存区

git rm 文件名删除文件/需要运行git commit 提交操作

git mv 源文件 目标文件 修改源文件名到目标文件名 

已经被跟踪的文件可以直接使用git  -a -m“ss”直接提交

git log --oneline

git log --oneline --decorate --graph --all 查看项目分叉历史

git config --global alias.别名 “log --oneline --decorate --graph --all” 配置别名

git diff 查看未提交的暂存

git reflog 查看修改head的所有完整操作

git log 查看当前指针所对应的所有一条线上策操作

### 分支指向最新提交的一次提交：

git branch 分支名 创建分支（并不会切到创建的分支）

git checkout 分支名

git branch -d 分支名  删除分支 （首先要切到别的分支上）

git branch -v 查看每个分支的最后一次提交

git branch name commitHash 新建一个分支，并使分支指向提交对象

在切换分支时，有未暂存的提交



### 分支合并

首先切到主分支，使用git merge 分支名 主分支合并指定分支

快进合并：



### git 存储

git stash命令将未完成的修改保存到一个栈上，而你可以在任何时候重新应用到这个改动 git stash apply



git stash list 查看存储

git stash applay 恢复缓存

git stash pop 应用缓存并删除存储

git stash drop stash@{0} 删除存储

### 撤销与重置

工作区

​		撤回在工作区的修改：git checkout --文件名称

暂存区

​		撤回在暂存区的修改：git reset HEAD <文件名>

版本库

​		1、重新修改提交注释

​		撤回版本库的修改：git commit --amend 



git reset --soft head 将当前的head指针指向当前分支，放弃之前的提交

git reset --soft hash 回到指定hash位置



### git远程仓库

git remote add 别名 url

git push 别名 分支名 推送到远程仓库

git fetch 别名 将修改同步到远程跟踪分支上

git clone **.git

更新分支：





git push -u 别名 分支名称 绑定本地分支与远端分支，可以实现git push 直接提交

