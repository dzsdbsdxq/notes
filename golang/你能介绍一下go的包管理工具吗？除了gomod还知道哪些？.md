> 题目序号：（5327）
>
> 题目来源：美团
>
> 频次：1

答案1：（Kjj）

**1、GOPATH**

	对于外部依赖的管理，在go 1.5之前go没有像java使用maven来管理依赖包、包版本；而是直接使用GOPATH来管理外部依赖包。

​	go允许import不同代码库的代码，例如github.com, k8s.io, golang.org等等；对于需要import的代码，可以使用go get命令取下来放到GOPATH对应的目录中去。例如go get github.com/globalsign/mgo，会下载到$GOPATH/src/github.com/globalsign/mgo中去，当其他项目在import github.com/globalsign/mgo的时候也就能找到对应的代码了

​	缺陷：

- 能拉取源码的平台很有限，绝大多数依赖的是 github.com
- 不能区分版本，以至于令开发者以最后一项包名作为版本划分
- 依赖 列表/关系 无法持久化到本地，需要找出所有依赖包然后一个个 go get
- 只能依赖本地全局仓库（GOPATH/GOROOT），无法将库放置于局部仓库（$PROJECT_HOME/vendor）

**2、vendor,godep,govendor,glide**

​	**vendor**

​	为了解决上述问题，go在1.5版本引入了vendor属性(默认关闭，需要设置go环境变量GO15VENDOREXPERIMENT=1)，并在1.6版本中默认开启了vendor属性。

​	简单来说，vendor属性就是让go编译时，优先从项目源码树根目录下的vendor目录查找代码(可以理解为切了一次GOPATH)，如果vendor中有，则不再去GOPATH中去查找。但是vendor目录又带来了一些新的问题：

- vendor目录中依赖包没有版本信息。这样依赖包脱离了版本管理，对于升级、问题追溯，会有点困难。
- 如何方便的得到本项目依赖了哪些包，并方便的将其拷贝到vendor目录下？ Manual is fxxk.

社区为了解决这些(工程)问题，在vendor基础上开发了多个管理工具，比较常用的有[godep](https://github.com/tools/godep), [govendor](https://github.com/kardianos/govendor), [glide](https://github.com/Masterminds/glide)。go官方也在开发官方[dep](https://github.com/golang/dep)，目前还是Alpha状态。

**godep,govendor,glide**

​	这里比较杂乱，gopher们可以看一下原文：https://www.cnblogs.com/jpfss/p/11806832.html

**3、go mod**

​	**go mod的定义**

​	Go.mod是Golang1.11版本新引入的官方包管理工具用于解决之前没有地方记录依赖包具体版本的问题，方便依赖包的管理。

​	Go.mod其实就是一个Modules，关于Modules的官方定义为：

​	Modules是相关Go包的集合，是源代码交换和版本控制的单元。go命令直接支持使用Modules，包括记录和解析对其他模块的依赖性。Modules替换旧的基于GOPATH的方法，来指定使用哪些源文件。

​	Modules和传统的GOPATH不同，不需要包含例如src，bin这样的子目录，一个源代码目录甚至是空目录都可以作为Modules，只要其中包含有go.mod文件。

​	**go.mod的使用**

1.首先将go的版本升级为1.11以上

2.设置GO111MODULE

GO111MODULE
	GO111MODULE有三个值：off, on和auto（默认值）。

- ​	GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
- GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
- GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
  - 当前目录在GOPATH/src之外且该目录包含go.mod文件
  - 当前文件在包含go.mod文件的目录下面。