> **题目序号：**（298）
>
> **题目来源：**
>
> **频次：**1

**答案1：**（树枝）+

**CSP并发模型：**

Go实现了两种并发模式。第一种：多线程共享内存。第二种：通过通信来共享内存（CSP）

CSP并发模型是Go语言特有的并发模型，也是Go语言官方所推荐的并发模型。

Go的CSP并发模型，是由Go语言中的`goroutine`与`channel`共同来实现的。

- goroutine：Go语言中使用关键字`go`来创建goroutine。将关键字`go`放到需要调用的函数前，在相同地址空间调用运行这个函数，该函数在执行的时候会创建一个独立的线程去执行，这个线程就是Go语言中的goroutine。
- channel：Go语言中goroutine之间的通信机制

**线程模型：**

1. 一对一模型（1:1）

   将一个用户级线程映射到一个内核线程，每一个线程由内核调度器独立调度，线程之间互不影响

   优点：在多核处理器的条件下，实现了真正的并行。

   缺点：为每一个用户级线程建立一个内核线程，开销大，浪费资源。

2. 多对一模型（M:1）

   将多个用户级线程映射到一个内核线程。

   优点：线程上下文切换发生在用户空间。

   缺点：只有一个处理器被应用，在多处理环境下是不可以被接受的，实现了并发，不能解决并行问题。

3. 多对多模型（M:N）

   多个用户级线程运行在多个内核线程上，这使得大部分的线程上下文切换都发生在用户空间，而多个内核线程又能充分利用处理器资源

**goroutine的调度机制：**

goroutine采用多对多的线程模型，goroutine的并发调度采用的是MPG调度模型

- M： Machine的简称， 在linux平台上是用clone系统调用创建的，其与用linux pthread库创建出来的线程本质上是一样的，都是利用系统调用创建出来的OS线程实体。M的作用就是执行G中包装的并发任务。Go运行时系统中的调度器的主要职责就是将G公平合理的安排到多个M上去执行。其属于OS资源，可创建的数量上也受限了OS，通常情况下G的数量都多于活跃的M的。 
- P：Processor的简称，处理器，它的主要用途就是用来执行goroutine的，它维护了一个goroutine队列，即runqueue。Processor是让咱们从N:1调度到M:N调度的重要部分。 
- G：Goroutine的简称，用go关键字加函数调用的代码就是创建了一个G对象，是对一个要并发执行的任务的封装，也可以称作用户态线程。属于用户级资源，对OS透明，具备轻量级，可以大量创建，上下文切换成本低等特点。 
- Processor的数量是在启动时被设置为环境变量GOMAXPROCS的值，或者经过运行时调用函数GOMAXPROCS()进行设置。Processor数量固定意味着任意时刻只有GOMAXPROCS个线程在运行go代码。 

**在单核处理器的场景下：**

所有的goroutine运行在同一个M物理线程下，每个M维护一个P，任何时刻P中只有一个goroutine，其余的goroutine都在runQueue中等待。

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/1649083632444.png)

上图讲的是两个内核线程的状况。一个M会对应一个内核线程，一个M也会链接一个上下文P，一个上下文P至关于一个“处理器”，一个上下文链接一个或者多个Goroutine。P(Processor)的数量是在启动时被设置为环境变量GOMAXPROCS的值，或者经过运行时调用函数`runtime.GOMAXPROCS()`进行设置。Processor数量固定意味着任意时刻只有固定数量的线程在运行go代码。Goroutine中就是咱们要执行并发的代码。图中P正在执行的`Goroutine`为蓝色的；处于待执行状态的`Goroutine`为灰色的，灰色的`Goroutine`造成了一个队列`runqueues` 

**当系统调用syscall:**

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/1649083816748.png)

上图左图所示，M0中的G0执行了syscall，而后就建立了一个M1(也有可能自己就存在，没建立)，（转向右图）而后M0丢弃了P，等待syscall的返回值，M1接受了P，将·继续执行`Goroutine`队列中的其余`Goroutine`。

当系统调用syscall结束后，M0会“偷”一个上下文，若是不成功，M0就把它的Gouroutine G0放到一个全局的runqueue中，而后本身放到线程池或者转入休眠状态。全局runqueue是各个P在运行完本身的本地的Goroutine runqueue后用来拉取新goroutine的地方。P也会周期性的检查这个全局runqueue上的goroutine，不然，全局runqueue上的goroutines可能得不到执行而饿死。

**均衡的分配工做:**

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/1649083880474.png)

 每一个P中的`Goroutine`不一样致使他们运行的效率和时间也不一样，在一个有不少P和M的环境中，不能让一个P跑完自身的`Goroutine`就没事可作了，由于或许其余的P有很长的`goroutine`队列要跑，空闲的P会从其他的P中获取一半的`Goroutine`