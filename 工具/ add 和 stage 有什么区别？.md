在回答这个问题之前需要先了解 Git 仓库的三个组成部分：

- 工作区(Working Directory)：在 Git 管理下的正常目录都算是工作区，我们平时的编辑工作都是在工作区完成。
- 暂存区(Stage)：临时区域。里面存放将要提交文件的快照。
- 历史记录区(History)：`git commit` 后的记录区。

然后，是这三个区的转换关系以及转换所使用的命令：[![三个区的转换关系](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/V6YD1eLShRgxmNr.png)](http://static.iocoder.cn/2135c3721109d120d0bde7f2ac0a1faa)

<center>三个区的转换关系</center>

再然后，我们就可以来说一下 `git add` 和 `git stage` 了。

- **其实，他们两是同义的**，所以，惊不惊喜，意不意外？这个问题竟然是个陷阱…引入 `git stage` 的原因其实比较有趣：是因为要跟 `svn add` 区分，两者的功能是完全不一样的，svn add 是将某个文件加入版本控制，而 git add 则是把某个文件加入暂存区。
- 因为在 Git 出来之前大家用 SVN 比较多，所以为了避免误导，Git 引入了 `git stage` ，然后把 `git diff --staged` 做为 `git diff --cached` 的相同命令。基于这个原因，我们建议使用 `git stage` 以及 `git diff --staged` 。