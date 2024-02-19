> 题目来源：腾讯

答案：树枝

**首先都介绍什么是pprof**

pprof是golang自带的性能分析工具，可以查看应用的运行状态，分析程序CPU，内存，goroutine等的使用情况，可以生成类似火焰图、堆栈图，内存分析图等。 

在golang中针对不用使用场景，提供了两种方式开启pprof分析性能

1. runtime/pprof：采集非server程序的运行数据进行分析
2. net/http/pprof：采集http server的运行数据进行分词

**简单介绍pprof的作用**

1. cpu分析，按照一定的频率监听cpu寄存器使用情况。确定Cpu周期花费时间的跟踪位置。
2. 内存分析，在应用程序进行堆分配时记录堆栈跟踪，用于监视当前和历史内存使用情况，以及检查内存泄漏。
3. 阻塞分析，记录 goroutine 阻塞等待同步（包括定时器通道）的位置
4. 互斥锁分析，报告互斥锁的竞争情况

**pprof的使用**

分析过程大体可以分为两步：1.导出数据。2.分析数据

~~~ go
package main

import (
    // 在引用中加上
    "net/http"
    _ "net/http/pprof"
)

func main () {
    // 通过协程开启pprof数据采集
    go func(){
        _ = http.ListenAndServe("0.0.0.0:8090", nil)
    }()
    // 业务代码doSomeThing
    todoSomething()
}

~~~

 打开浏览器，访问http://127.0.0.1:8090/debug/pprof/ 

~~~ go
//其会自动下载数据到本地，然后供你分析
go tool pprof http://127.0.0.1:6060/debug/pprof/profile?seconds=60

~~~

**2.分析数据**

```go
输入 web 命令，可在弹出的浏览器窗口看到cpu占用情况;
输入 pdf 命令，会生成一张pdf文件;
输入 top10，会显示前10 最消耗cpu的程序片断;
```

