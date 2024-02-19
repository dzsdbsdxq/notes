> 题目序号：3416
> 题目来源：百度
> 频次：    1

## 答案：豪豪

+ goroutine
+ channel
+ sync.Pool

goroutine

> `goroutine`: 协程应该可以看作时Go语言的一个特色，一个goroutine只有几kb，通过go关键字就能够直接调用
> 进而在协程调度切换方面也只占用非常少的资源，且 goroutine 的控制属于用户态。

channel

> `channel`: channel设计的核心思想就时同过通信共享内存，通过合理使用阻塞节省CPU资源。在不同的场景使用两种不同
>
> + 无缓冲chan：同步控制。
> + 有缓冲chan：异步控制。

sync.Pool

>`sync.Pool`: sync.Pool 设计的初衷是减少GC压力，举个例子：在一个业务中可能需要频繁的New（）一个结构体 并使用json 反射到该结构体的字段上。
>但是频繁的new 结构体 虽然有GC回收，但是GC回收期间也需要STW,因此使用sync.Pool能够提高程序对内存的复用，进而提高性能。