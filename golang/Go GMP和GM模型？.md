什么才是一个好的调度器？

能在适当的时机将合适的协程分配到合适的位置，保证公平和效率。

Go采用了GMP模型（对两级线程模型的改进实现），使它能够更加灵活地进行线程之间的调度。

### GMP模型

GMP是Go运行时调度层面的实现，包含4个重要结构，分别是G、M、P、Sched

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220419120403666.png)

**G（Goroutine）**：代表Go 协程Goroutine，存储了 Goroutine 的执行栈信息、Goroutine 状态以及 Goroutine 的任务函数等。**G的数量无限制，理论上只受内存的影响**，创建一个 G 的初始栈大小为2-4K，配置一般的机器也能简简单单开启数十万个 Goroutine ，而且Go语言在 G 退出的时候还会把 G 清理之后放到 P 本地或者全局的闲置列表 gFree 中以便复用。

**M（Machine）**： Go 对操作系统线程（OS thread）的封装，可以看作操作系统内核线程，想要在 CPU 上执行代码必须有线程，通过系统调用 clone 创建。M在绑定有效的 P 后，进入一个调度循环，而调度循环的机制大致是从 P 的本地运行队列以及全局队列中获取 G，切换到 G 的执行栈上并执行 G 的函数，调用 goexit 做清理工作并回到 M，如此反复。M 并不保留 G 状态，这是 G 可以跨 M 调度的基础。**M的数量有限制，默认数量限制是 10000**，可以通过 debug.SetMaxThreads() 方法进行设置，如果有M空闲，那么就会回收或者睡眠。

**P（Processor）：虚拟处理器，M执行G所需要的资源和上下文，只有将 P 和 M 绑定，才能让 P 的 runq 中的 G 真正运行起来。P 的数量决定了系统内最大可并行的 G 的数量，**P的数量受本机的CPU核数影响，可通过环境变量$GOMAXPROCS或在runtime.GOMAXPROCS()来设置，默认为CPU核心数。

**Sched：调度器结构**，它维护有存储M和G的全局队列，以及调度器的一些状态信息

|          | G                      | M                                                          | P                                                       |
| -------- | ---------------------- | ---------------------------------------------------------- | ------------------------------------------------------- |
| 数量限制 | 无限制，受机器内存影响 | 有限制，默认最多10000                                      | 有限制，最多GOMAXPROCS个                                |
| 创建时机 | go func                | 当没有足够的M来关联P并运行其中的可运行的G时会请求创建新的M | 在确定了P的最大数量n后，运行时系统会根据这个数量创建个P |

**核心数据结构:**

```go
//src/runtime/runtime2.go
type g struct {
goid    int64 // 唯一的goroutine的ID
sched gobuf // goroutine切换时，用于保存g的上下文
stack stack // 栈
gopc        // pc of go statement that created this goroutine
startpc    uintptr // pc of goroutine function
...
}

type p struct {
lock mutex
id          int32
status      uint32 // one of pidle/prunning/...

// Queue of runnable goroutines. Accessed without lock.
runqhead uint32 // 本地队列队头
runqtail uint32 // 本地队列队尾
runq     [256]guintptr // 本地队列，大小256的数组，数组往往会被都读入到缓存中，对缓存友好，效率较高
runnext guintptr // 下一个优先执行的goroutine（一定是最后生产出来的)，为了实现局部性原理，runnext中的G永远会被最先调度执行
... 
}

type m struct {
g0            *g     
// 每个M都有一个自己的G0，不指向任何可执行的函数，在调度或系统调用时，M会切换到G0，使用G0的栈空间来调度
curg          *g    
// 当前正在执行的G
... 
}

type schedt struct {
...
runq     gQueue // 全局队列，链表（长度无限制）
runqsize int32  // 全局队列长度
...
}
```

GMP模型的实现算是Go调度器的一大进步，但调度器仍然有一个令人头疼的问题，那就是不支持抢占式调度，这导致一旦某个 G 中出现死循环的代码逻辑，那么 G 将永久占用分配给它的 P 和 M，而位于同一个 P 中的其他 G 将得不到调度，出现“饿死”的情况。

当只有一个 P（GOMAXPROCS=1）时，整个 Go 程序中的其他 G 都将“饿死”。于是在 Go 1.2 版本中实现了基于协作的“抢占式”调度，在Go 1.14 版本中实现了基于信号的“抢占式”调度。

### GM模型

Go早期是GM模型，没有P组件

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220412194120600.png)

**GM调度存在的问题：    **

1.  **全局队列的锁竞争**，当 M 从全局队列中添加或者获取 G 的时候，都需要获取队列锁，导致激烈的锁竞争
2.  **M 转移 G 增加额外开销**，当 M1 在执行 G1 的时候， M1 创建了 G2，为了继续执行 G1，需要把 G2 保存到全局队列中，无法保证G2是被M1处理。因为 M1 原本就保存了 G2 的信息，所以 G2 最好是在 M1 上执行，这样的话也不需要转移G到全局队列和线程上下文切换
3.  **线程使用效率不能最大化**，没有**work-stealing** 和**hand-off** 机制

计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决，为了解决这一的问题 go 从 1.1 版本引入P，在运行时系统的时候加入 P 对象，让 P 去管理这个 G 对象，M 想要运行 G，必须绑定 P，才能运行 P 所管理 的 G