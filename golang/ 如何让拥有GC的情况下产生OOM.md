> 题目序号：（3196）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（One）

**一、内存溢出(out of memory，简称OOM)** 

[内存溢出](https://so.csdn.net/so/search?q=内存溢出&spm=1001.2101.3001.7020)是指程序在申请内存时，没有足够的内存空间供其使用，简单点说就是你要求分配的内存超出了系统能给你的，系统不能满足需求，于是产生溢出出现out of memory异常。

**内存泄露(memory leak)** 

[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)泄露是指程序在申请内存后，无法释放已申请的内存空间，简单点说就是你向系统申请分配内存进行使用(new)，可是使用完了以后却不归还(delete)，结果你申请到的那块内存你自己也不能再访问（也许你把它的地址给弄丢了），而系统也不能再次将它分配给需要的程序。

内存泄露指的是程序运行过程中已不再使用的内存，没有被释放掉，导致这些内存无法被使用，直到程序结束这些内存才被释放的问题。

Go虽然有GC来回收不再使用的堆内存，减轻了开发人员对内存的管理负担，但这并不意味着Go程序不再有内存泄露问题。分配的内存不足以放下数据项序列,称为**内存溢出**。

**二、内存泄漏定位：**

关于Go的内存泄露：10次内存泄露，有9次是goroutine泄露。

**所以：掌握了如何定位和解决goroutine泄露，就掌握了Go内存泄露的大部分场景**。

**2.1: pprof工具:**

pprof是Go的性能分析工具，在程序运行过程中，可以记录程序的运行信息，可以是CPU使用情况、内存使用情况、goroutine运行情况等，当需要性能调优或者定位Bug时候，这些记录的信息是相当重要。

**基本使用:**

使用pprof有多种方式，Go已经现成封装好了1个：`net/http/pprof`，使用简单的几行命令，就可以开启pprof，记录运行信息，并且提供了Web服务，能够通过浏览器和命令行2种方式获取运行数据。

看个demo：

```go
package main
 
import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
)
 
func main() {
    // 开启pprof，监听请求
    ip := "0.0.0.0:6060"
    if err := http.ListenAndServe(ip, nil); err != nil {
        fmt.Printf("start pprof failed on %s
", ip)
    }
```

输入网址`ip:port/debug/pprof/`打开pprof主页，从上到下依次是**5类profile信息**：

1. **block**：goroutine的阻塞信息
2. **goroutine**：所有goroutine的信息，下面的`full goroutine stack dump`是输出所有goroutine的调用栈，是goroutine的debug=2，后面会详细介绍。
3. **heap**：堆内存的信息
4. **mutex**：锁的信息
5. **threadcreate**：线程信息

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/20210704005033409.png)

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/20210704005033409.png)

 

使用命令`go tool pprof url`可以获取指定的profile文件，此命令会发起http请求，然后下载数据到本地之后进入交互式模式，就像gdb一样，可以使用命令查看运行信息，以下是5类请求的方式：

```ruby
# 下载cpu profile，默认从当前开始收集30s的cpu使用情况，需要等待30s
go tool pprof http://localhost:6060/debug/pprof/profile   # 30-second CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=120     # wait 120s
 
# 下载heap profile
go tool pprof http://localhost:6060/debug/pprof/heap      # heap profile
 
# 下载goroutine profile
go tool pprof http://localhost:6060/debug/pprof/goroutine # goroutine profile
 
# 下载block profile
go tool pprof http://localhost:6060/debug/pprof/block     # goroutine blocking profile
 
# 下载mutex profile
go tool pprof http://localhost:6060/debug/pprof/mutex
```

请求之后会展示以下信息： 

1. 下载得到的文件：`/home/ubuntu/pprof/pprof.demo.alloc_objects.alloc_space.inuse_objects.inuse_space.001.pb.gz`，这其中包含了程序名`demo`，profile类型`alloc`已分配的内存，`inuse`代表使用中的内存。
2. `help`可以获取帮助，最先会列出支持的命令，想掌握pprof，要多看看，多尝试。关于命令，本文只会用到3个，我认为也是最常用的：`top`、`list`、`traces`，分别介绍一下。

3. **top**：显示正运行到某个函数goroutine的数量
4. **traces**：显示所有goroutine的调用栈
5. **list**：列出代码详细的信息。

top会列出5个统计数据：

- flat: 本函数占用的内存量。
- flat%: 本函数内存占使用中内存总量的百分比。
- sum%: 前面每一行flat百分比的和，比如第2行虽然的100% 是 100% + 0%。
- cum: 是累计量，加入main函数调用了函数f，函数f占用的内存量，也会记进来。
- cum%: 是累计量占总量的百分比。

怎么发现内存泄露

1. **监控工具**：固定周期对进程的内存占用情况进行采样，数据可视化后，根据内存占用走势（持续上升），很容易发现是否发生内存泄露。
2. **go pprof**：适合没有监控工具的情况，使用Go提供的pprof工具判断是否发生内存泄露

**总结**

**goroutine泄露的本质**

goroutine泄露的本质是channel阻塞，无法继续向下执行，导致此goroutine关联的内存都无法释放，进一步造成内存泄露。

**goroutine泄露的发现和定位**

利用好go pprof获取goroutine profile文件，然后利用3个命令top、traces、list定位内存泄露的原因。

**goroutine泄露的场景**

泄露的场景不仅限于以下两类，但因channel相关的泄露是最多的。

1. channel的读或者写：
   1. 无缓冲channel的阻塞通常是写操作因为没有读而阻塞
   2. 有缓冲的channel因为缓冲区满了，写操作阻塞
   3. 期待从channel读数据，结果没有goroutine写
2. select操作，select里也是channel操作，如果所有case上的操作阻塞，goroutine也无法继续执行。

**编码goroutine泄露的建议**

为避免goroutine泄露造成内存泄露，启动goroutine前要思考清楚：

1. goroutine如何退出？
2. 是否会有阻塞造成无法退出？如果有，那么这个路径是否会创建大量的goroutine？