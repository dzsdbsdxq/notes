>  **题目序号：**（9，31，98，152，218，486，637，651，694，739，819，863，917，978，937，1216，1674，1865，2298，2708，3285，3303，3602，5117，5164，5443，5678，5654，6065，6370，6551、5526、5045、4823、4552、4369、4348、4423）
>  **题目来源**：深信服，百度，小米，哔哩哔哩，好未来，跟谁学，学而思，网易，腾讯，知乎，高德，字节，新浪，虾皮，Aibee、快手、探探、滴滴
>  **频次：**37

**答案1：**（小小）+

**三个字母的含义**

- `G（Goroutine）`：G 就是我们所说的 Go 语言中的协程 Goroutine 的缩写，相当于操作系统中的进程控制块。其中存着 goroutine 的运行时栈信息，CPU 的一些寄存器的值以及执行的函数指令等。
- `M（Machine）`：代表一个操作系统的主线程，对内核级线程的封装，数量对应真实的 CPU 数。一个 M 直接关联一个 os 内核线程，用于执行 G。M 会优先从关联的 P 的本地队列中直接获取待执行的 G。M 保存了 M 自身使用的栈信息、当前正在 M上执行的 G 信息、与之绑定的 P 信息。
- `P（Processor）`：Processor 代表了 M 所需的上下文环境，代表 M 运行 G 所需要的资源。是处理用户级代码逻辑的处理器，可以将其看作一个局部调度器使 go 代码在一个线程上跑。当 P 有任务时，就需要创建或者唤醒一个系统线程来执行它队列里的任务，所以 P 和 M 是相互绑定的。总的来说，P 可以根据实际情况开启协程去工作，它包含了运行 goroutine 的资源，如果线程想运行 goroutine，必须先获取 P，P 中还包含了可运行的 G 队列。

**源码**

1. **G**

```go
type g struct {
  stack       stack   		// 描述真实的栈内存，包括上下界

  m              *m     	// 当前的 m
  sched          gobuf   	// goroutine 切换时，用于保存 g 的上下文      
  param          unsafe.Pointer // 用于传递参数，睡眠时其他 goroutine 可以设置 param，唤醒时该goroutine可以获取
  atomicstatus   uint32
  stackLock      uint32 
  goid           int64  	// goroutine 的 ID
  waitsince      int64 		// g 被阻塞的大体时间
  lockedm        *m     	// G 被锁定只在这个 m 上运行
}
```

其中 sched 比较重要，该字段保存了 goroutine 的上下文。goroutine 切换的时候不同于线程有 OS 来负责这部分数据，而是由一个 gobuf 结构体来保存，gobuf 的结构如下：

```go
type gobuf struct {
    sp   uintptr
    pc   uintptr
    g    guintptr
    ctxt unsafe.Pointer
    ret  sys.Uintreg
    lr   uintptr
    bp   uintptr // for GOEXPERIMENT=framepointer
}
```

这里可以看出该结构体保存了当前的栈指针，计数器，还有 g 自身，这里记录自身 g 的指针的目的是为了**能快速的访问到 goroutine 中的信息**。

2. **M**

```go
type m struct {
    g0      *g     				// 带有调度栈的goroutine

    gsignal       *g         	// 处理信号的goroutine
    tls           [6]uintptr 	// thread-local storage
    mstartfn      func()
    curg          *g       		// 当前运行的goroutine
    caughtsig     guintptr 
    p             puintptr 		// 关联p和执行的go代码
    nextp         puintptr
    id            int32
    mallocing     int32 		// 状态

    spinning      bool 			// m是否out of work
    blocked       bool 			// m是否被阻塞
    inwb          bool 			// m是否在执行写屏蔽

    printlock     int8
    incgo         bool
    fastrand      uint32
    ncgocall      uint64      	// cgo调用的总数
    ncgo          int32       	// 当前cgo调用的数目
    park          note
    alllink       *m 			// 用于链接allm
    schedlink     muintptr
    mcache        *mcache 		// 当前m的内存缓存
    lockedg       *g 			// 锁定g在当前m上执行，而不会切换到其他m
    createstack   [32]uintptr 	// thread创建的栈
}
```

结构体 M 中，有两个重要的字段：

-  curg：代表结构体M当前绑定的结构体 G 。
-  g0 ：是带有调度栈的 goroutine，普通的 goroutine 的栈是在**堆上**分配的可增长的栈，但是 g0 的栈是 **M 对应的线程**的栈。与调度相关的代码，会先切换到该 goroutine 的栈中再执行。

3. **P**

```go
type p struct {
    lock mutex

    id          int32
    status      uint32 		// 状态，可以为pidle/prunning/...
    link        puintptr
    schedtick   uint32     // 每调度一次加1
    syscalltick uint32     // 每一次系统调用加1
    sysmontick  sysmontick 
    m           muintptr   // 回链到关联的m
    mcache      *mcache
    racectx     uintptr

    goidcache    uint64 	// goroutine的ID的缓存
    goidcacheend uint64

    // 可运行的goroutine的队列
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr

    runnext guintptr 		// 下一个运行的g

    sudogcache []*sudog
    sudogbuf   [128]*sudog

    palloc persistentAlloc // per-P to avoid mutex

    pad [sys.CacheLineSize]byte
}
```

- P 的个数就是 GOMAXPROCS（最大256），启动时固定的，一般不修改；GOMAXPOCS 默认值是当前电脑的核心数，单核CPU就只能设置为1，如果设置>1，在 GOMAXPOCS 函数中也会被修改为1。
- M 的个数和P 的个数不一定一样多（会有休眠的M或者不需要太多的 M）（M 最大10000）；
- 每一个 P 保存着本地 G 任务队列，也有一个全局 G 任务队列。

**模型介绍**

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/1648716696731-0d4e04b1-da32-4ff8-ac44-f2f74cfb7629-16488810664042.jpeg)
本地队列：存放等待运行的 G，一个本地队列存放的G数量一般不超过 256 个，优先将新创建的 G 放在 P 的本地队列中，如果满了会放在全局队列中。
全局队列：存放等待运行的 G，读写要**加锁**，所以拿取效率在多线程竞争的情况下相比于本地队列来说要低。

**面试回答模板**

首先呢，GMP 这三个字母的含义分别是 Goroutine，Machine，Processor。这个Goroutine，相当于操作系统中的进程控制块。其中存着 goroutine 的运行时栈信息，CPU 的一些寄存器的值以及执行的函数指令等。Machine就是代表了一个操作系统的主线。M 结构体中，保存了 M 自身使用的栈信息、当前正在 M上执行的 G 信息、与之绑定的 P 信息。M 直接关联一个 os 内核线程，用于执行 G。（这里思考一个这个模型的图片回答），这个 M 做的事情就是从关联的 P 的本地队列中直接获取待执行的 G。剩下的 Processor 是代表了 M 所需的上下文环境，代表 M 运行 G 所需要的资源。当 P 有任务时，就需要创建或者唤醒一个系统线程来执行它队列里的任务。在GMP调度模型中，P 的个数就是 GOMAXPROCS，是可以手动设置的，但一般不修改，GOMAXPOCS 默认值是当前电脑的核心数，单核CPU就只能设置为1，如果设置>1，在 GOMAXPOCS 函数中也会被修改为1。总的来说，这个 P 结构体的主要的任务就是可以根据实际情况开启协程去工作。