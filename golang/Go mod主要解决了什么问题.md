> 题目序号：（3438）
>
> 题目来源：百度
>
> 频次：1

答案1：（Kjj）

- **项目不在需要放到$GOPATH/src目录下**
- **依赖包的版本控制**
  依赖包的版本交由go.mod文件控制。在go.mod用require语句指定包和版本 ，go命令会根据指定的路径和版本下载包，指定版本时可以用latest，这样它会自动下载指定包的最新版本；如果，go.mod中没有指定，go命令会自动下载代码中的依赖的最新版本。
- **依赖包中的地址失效问题的解决**
  在go快速发展的过程中，由于一些依赖包地址变更而导致无法下载问题。以前的做法：
  1.修改源码，用新路径替换import的地址
  2.git clone 或 go get 新包后，copy到$GOPATH/src里旧的路径下
  但无论什么方法，都不便于维护，特别是多人协同开发时。
  而在gomod模式下，只需要在go.mod文件里用 replace 替换包，例如
  replace golang.org/x/text => github.com/golang/text latest
  go会用 github.com/golang/text 替代golang.org/x/text，原理就是下载github.com/golang/text 的最新版本到 $GOPATH/pkg/mod/golang.org/x/text下。
- **在项目中执行go get命令可以下载依赖包，并且还可以指定下载的版本**
  运行go get -u将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
  运行go get -u=patch将会升级到最新的修订版本
  运行go get package@version将会升级到指定的版本号version