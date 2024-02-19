PR 和 MR 的全称分别是 pull request 和 merge request。

🚀 解释它们两者的区别之前，我们需要先了解一下 Code Review，因为 PR 和 MR 的引入正是为了进行 Code Review。

- Code Review 是指在开发过程中，对代码的系统性检查。通常的目的是查找系统缺陷，保证代码质量和提高开发者自身水平。 Code Review 是轻量级代码评审，相对于正式代码评审，轻量级代码评审所需要的各种成本要明显低的多，如果流程正确，它可以起到更加积极的效果。
- 进行 Code Review 的原因：
  - 提高代码质量
  - 及早发现潜在缺陷与 BUG ，降低事故成本。
  - 促进团队内部知识共享，提高团队整体水平
  - 评审过程对于评审人员来说，也是一种思路重构的过程，帮助更多的人理解系统。

🚀 然后我们需要了解下 fork 和 branch ，因为这是 PR 和 MR 各自所属的协作流程。

- **fork** 是 git 上的一个协作流程。通俗来说就是把别人的仓库备份到自己仓库，修修改改，然后再把修改的东西提交给对方审核，对方同意后，就可以实现帮别人改代码的小目标了。fork 包含了两个流程：
  - fork 并更新某个仓库

[![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/a87e005d714e9fd3b8da23c3b20c4fff)](http://static.iocoder.cn/6227935e023affd4e0a339ccd197c8a6)

- 同步 fork

[![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/a87e005d714e9fd3b8da23c3b20c4fff)](http://static.iocoder.cn/a87e005d714e9fd3b8da23c3b20c4fff)

- 和 fork 不同，**branch** 并不涉及其他的仓库，操作都在当前仓库完成。

[![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/a87e005d714e9fd3b8da23c3b20c4fff)](http://static.iocoder.cn/7200ee4a67dad8b11387fab7abef9f76)

考察关键点：

- Code review；
- PR 和 MR 所属流程的细节。

回答关键点：

- 回答这个问题的时候不要单单只说它们的区别。而是要从 PR 和 MR 产生的原因，分析它们所属的流程，然后再得出两者的区别。