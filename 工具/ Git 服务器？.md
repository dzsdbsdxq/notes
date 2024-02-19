Git 服务器的选择，实际上是比较多的。

- 公有服务方案

  - Github
  - Gitee

- 私有化部署方案

  - GitLab

  - Gogs

  - Bitbucket

    > 注意，Gitlab 和 Bitbucket 也提供公有服务的方案。

一般情况下，大多数公司使用 GitLab 作为 Git 服务器。

> GitLab是一个利用 [Ruby on Rails](http://www.oschina.net/p/ruby+on+rails) 开发的开源应用程序，实现一个自托管的[Git](http://www.oschina.net/p/git)项目仓库，可通过Web界面进行访问公开的或者私人项目。
>
> 它拥有与[Github](http://www.oschina.net/p/github)类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

- 不过因为 GitLb 使用 Ruby on Rails 实现，所以占用的系统资源会比较多。