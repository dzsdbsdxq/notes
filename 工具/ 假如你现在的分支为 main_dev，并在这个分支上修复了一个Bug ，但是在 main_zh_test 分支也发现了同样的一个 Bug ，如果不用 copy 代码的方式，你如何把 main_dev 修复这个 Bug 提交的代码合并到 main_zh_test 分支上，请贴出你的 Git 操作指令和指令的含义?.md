> 假设合并时没有冲突

- 1、在 `main_dev` 分支上，通过 gitlog 命令，使用 bugid 搜索提交的 commit id 。
- 2、使用 `git checkout main_zh_test` 命令，切换到 `main_zh_test` 分支。
- 3、使用 `git cherry-pick commitid` 将对 Bug 的修改批量移植到该分支上。
- 4、`git commit` ，提交到本地。
- 5、`git push` ，推送到远程仓库。