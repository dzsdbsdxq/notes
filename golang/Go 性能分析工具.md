> 题目序号：689
>
> 题目来源：腾讯
>
> 频次：1

答案：自由

Go 语言为开发者提供了丰富的性能分析 API 和好用的标准工具，这些 API 主要存在于 runtime/pprof、net/http/pprof、runtime/trace 这三个代码包中。回到问题，至于标准工具，主要有 Go tool pprof 和 Go tool trace 这两个。它们可以解析概要文件中的信息。概要文件可以通过 Go test 命令生成。在 Go 语言中，用于分析程序性能的概要文件有三种，分别是 CPU 概要文件、内存概要文件和阻塞概要文件。这些概要文件中的内容都是在某一段时间内，对 Go 程序的相关指标进行多次采样后得到的概要信息，查阅概要文件，内容一般如下：

```go
$ go tool pprof cpuprofile.out
Type: cpu
Time: Nov 9, 2018 at 4:31pm (CST)
Duration: 7.96s, Total samples = 6.88s (86.38%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) 
```
