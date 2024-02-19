> 题目序号：(1630)
> 题目来源：腾讯
> 频次：1

**答案1：**（cxiang）

**benchmark** 
Go 语言标准库内置的 testing 测试框架提供了基准测试(benchmark)的能力，能让我们很容易地对某一段代码进行性能测试。

**pprodf**

pprof 是用于可视化和分析性能分析数据的工具，pprof 包含两部分：

- 编译到程序中的 `runtime/pprof` 包
- 性能剖析工具 `go tool pprof`