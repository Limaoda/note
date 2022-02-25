## git常用操作和一些协作流程

```shell
# 在终端中输出帮助文档日志
git -h #输出所有git命令的基本介绍日志
# 例子
git add -h #输出git add的所有使用帮助日志
git commit -h #输出git commit的所有使用帮助日志

# 打开git官方帮助文档
git --help
# 例子
git add --help 
git commit --help


# 将工作区内容添加至暂存区
git add <文件路径(相对.git文件)>
# 例子
git add ./hello.py

# 将暂存区内容提交至本地仓库
git commit -m"注释"

# 本地仓库关联远程仓库
git remote add <远程主机名> <远程仓库url>
# 例子
git remote add origin git@e.coding.net:huaixin/java_study/web_demo.git

# 移除远程仓库关联
git remote rm <远程主机名>
# 例子
git remote rm origin

# 将本地仓库内容推送至远程仓库
git push <远程主机名> <本地分支名> <远程仓库分支名>
# 例子
git push origin master:remote-master #将本地的master分支内容推送到远程主机origin的远程仓库remote-master分支上
git push origin :remote-master #删除远程主机origin的远程仓库remote-master分支（不指定本地分支而指定远程分支相当于推送一条空的本地分支至远程remote-master分支），作用与git push origin --delete remote-master相同
git push origin master #将本地仓库master分支内容推送至远程主机origin的远程仓库master分支（没有指定远程分支，自动找到远程仓库中与本地同名的分支进行推送，如远程不存在同名分支，则自动创建并进行推送）
git push #仅限于当前分支只有一条远程分支的情况下
git push --all origin #将本地所有分支推送到远程主机origin的远程仓库所有分支上（远程不存在同名分支则自动创建）
git push origin master -f 或
git push origin master --force # 将本地仓库master分支强制推送至远程主机origin的远程仓库master分支上（包括本地分支比远程分支版本低的情况），！谨慎操作
git push --set-upstream(-u) origin master #将本地分支master默认关联至远程主机origin的远程仓库master分支，以后在本地master分支可直接git push将其推送至远程

# 将远程仓库中与当前本地分支同名的分支拉下来并进行合并，同等于git fetch + git merge FETCH_HEAD,用法同git push
git pull <远程主机名> <远程分支名> <本地分支名>

# 切换本地分支
git checkout <本地分支名>
# 例子
git checkout dev #假设当前位于本地master分支上，则切换当前分支为dev

# 删除远程分支
git push origin --delete <remoteBranchName>
# 例子
git push origin --delete master

# 删除本地分支 -D 
git branch -D <localBranchName>
# 例子
git branch -D master

# 查看本地分支
git branch

# 创建新的本地分支
git branch <本地分支名>
# 例子
git branch dev #创建名为dev的本地分支

# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 重命名分支
git branch -m <oldName> <newName>
# 例子
git branch -m master mymaster

# 查看当前本地分支历史提交日志
git log

# 查看提交操作日志(当回滚到历史版本后想回滚到未来时可用)
git reflog

# 版本回滚
git reset --hard <commitId> #commitId可从git log和git reflog的输出日志中获取

# 分支合并
git merge <合并分支>
# 例子
git merge dev # 假设当前分支为master，将dev分支的所有内容对比master并合并到master分支中

# 合并回滚
git merge --abort #当合并后有冲突时可使用该命令回到合并前（需保证合并前的当前分支和合并目标分支都有进行commit）

# 将当前未进行commit的所有文件暂存起来
git stash 
# 将当前未进行commit的所有文件从暂存区中还原出来
git stash pop

# 变基
git rebase <合并分支> <主分支> 
# git rebase和git merge都可以用来做分支合并操作，git rebase的本质是去找到两个分支的祖先节点（公共节点），然后将源分支（被合并的分支）在祖先节点之后的提交移动到目标分支的最前面（此时源分支的提交历史受到影响），不会创建一个新的commit（保证分支树线性进行），当遇到冲突时需要先git rebase --continue再git commit，如果源分支是一条公共分支（多人开发维护的分支），那么rebase需要慎用
# git merge是创建一个新的commit节点，此时目标分支改变（新加了源分支的提交），源分支不变，优点是保留了提交历史和分支结构，缺点是每次合并会多一次commit节点，污染了提交历史，造成分支合并树混乱。
```

