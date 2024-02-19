[![Git 常用命令](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/INFvAceGtwlUJYm.png)](http://static.iocoder.cn/3054153c1904dfd5f3b9d3fbc6bf2375)

<center>Git 常用命令</center>

- `git init`：创建 Git 库。

- `git status` ：查看当前仓库的状态。

- `git show` ：# 显示某次提交的内容 git show $id

- `git diff` ：查看本次修改与上次修改的内容的区别。

- `git add <file>` ：把现在所要添加的文件放到暂存区中。

- `git log -p <file>` ：查看每次详细修改内容的 diff 。

- `git rm <file>` ：从版本库中删除文件。

- `git reset <file>` ：从暂存区恢复到工作文件。

- `git reset HEAD^` ：恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改` 。

  > HEAD 本身是一個游标，它通常會指向某一个本地端分支或是其它 commit，所以你也可以把 HEAD 当做是目前所在的分支（current branch）。 可参见 [《Git 中 HEAD 是什么东西》](https://juejin.im/entry/59a38c5d6fb9a0248e5cc884) 。

- `git commit` ：把 Git add 到暂存区的内容提交到代码区中。

- `git clone` ：从远程仓库拷贝代码到本地。

- `git branch`：查看当前的分支名称。

- `git branch -r` ：查看远程分支。

- `git checkout` ：切换分支。

- `git merge <branch>` ：将 branch 分支合并到当前分支。

- `git stash`：暂存。

- `git stash pop` ：恢复最近一次的暂存。

- `git pull`：抓取远程仓库所有分支更新并合并到本地。

- `git push origin master` ：将本地主分支推到远程主分支。

[![整体流程](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/INFvAceGtwlUJYm.png)](http://static.iocoder.cn/033f8d41d6f67a01a7cfefa6b9aa4cf4)

<center>Git 整体流程</center>