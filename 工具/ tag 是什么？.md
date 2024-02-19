tag ，指向一次 commit 的 id ，通常用来给分支做一个标记。

> 大多数情况下，我们会将每个 Release 版本打一个分支。例如 SkyWalking 的 Tag 是 https://github.com/apache/incubator-skywalking/tags 。

- 打标签 ：`git tag -a v1.01 -m ""Release version 1.01""` 。
- 提交标签到远程仓库 ：`git push origin --tags` 。
- 查看标签 ：`git tag` 。
- 查看某两次 tag 之间的 commit ：`git log --pretty=oneline tagA..tagB` 。
- 查看某次 tag 之后的 commit ：`git log --pretty=oneline tagA..` 。