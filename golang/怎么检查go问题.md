> **题目序号：**935
>
> **题目来源：**好未来
>
> **频次：**1

 **答案1：**（趁醉独饮痛）

**golangci-lint：**

golangci-lint 是一个集成工具，它集成了很多静态代码分析工具（静态代码分析是不会运行代码的），我们通过配置这个工具，便可灵活启用需要的代码规范检查。

**安装命令：**

```plain
go get github.com/golangci/golangci-lint/cmd/golangci-lint
```

**自带的 vet**

vet 是 go 中自带的**静态分析工具**，可以让我们检查出 package 或者源码文件中一些隐含的错误。

```go
//分析某个文件
go vet test/main.go

//分析某个包
go vet test/*.go
go vet test/...
```

**benchmark 基准测试**

**profile**

[官方文章](https://go.dev/blog/pprof)

profile就是定时采样，收集cpu，内存等信息，进而给出性能优化指导。

可以通过编译后文件，查看火焰图，查看函数执行时长、内存分析，CPU执行时长

**详情可见-->**[极客兔兔](https://geektutu.com/post/hpg-pprof.html)