> 题目序号：（3438）
>
> 题目来源：百度
>
> 频次：1

答案1：（Kjj）

**1、go sum的意义**

	为了确保一致性构建，Go引入了go.mod文件来标记每个依赖包的版本，在构建过程中go命令会下载go.mod中的依赖包，下载的依赖包会缓存在本地，以便下次构建。 考虑到下载的依赖包有可能是被黑客恶意篡改的，以及缓存在本地的依赖包也有被篡改的可能，单单一个go.mod文件并不能保证一致性构建。

​	为了解决Go module的这一安全隐患，Go开发团队在引入go.mod的同时也引入了go.sum文件，用于记录每个依赖包的哈希值，在构建时，如果本地的依赖包hash值与go.sum文件中记录得不一致，则会拒绝构建。

**2、go sum的内容**

​	go.sum 的每一行都是一个条目，格式为：

```go
<module> <version>/go.mod <hash>
```

或者：

```go
<module> <version> <hash>
<module> <version>/go.mod <hash>
```

​	其中module是依赖的路径，version是依赖的版本号，hash是以h1:开头的字符串，表示生成checksum的算法是第一版的hash算法（sha256），彼此之间由空格分开。

​	比如，某个go.sum文件中记录了github.com/google/uuid 这个依赖包的`v1.1.1`版本的哈希值：

```go
github.com/google/uuid v1.1.1 h1:Gkbcsh/GbpXz7lPftLA3P6TYMwjCLYm83jiFQZF/3gY=  
github.com/google/uuid v1.1.1/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=
```

​	在Go module机制下，我们需要同时使用依赖包的名称和版本才可以准确的描述一个依赖，为了方便叙述，下面我们使用依赖包版本来指代依赖包名称和版本。

​	正常情况下，每个依赖包版本会包含两条记录，第一条记录为该依赖包版本整体（所有文件）的哈希值，第二条记录仅表示该依赖包版本中go.mod文件的哈希值，如果该依赖包版本没有go.mod文件，则只有第一条记录（第一种格式）。如上面的例子中（第二种格式），v1.1.1表示该依赖包版本整体，而v1.1.1/go.mod表示该依赖包版本中go.mod文件。