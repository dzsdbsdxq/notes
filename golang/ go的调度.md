> 题目序号：（3239）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（One）

如何调度实现的机制？

- G是Goroutine的缩写，在这里就是Goroutine的控制结构，是对Goroutine的抽象。其中包括执行的函数指令及参数；G保存的任务对象；线程上下文切换，现场保护和现场恢复需要的寄存器(SP、IP)等信息。**p本地队列中的G是环形队列**

- M：thread表示操作系统线程也可以成为内核线程，由操作系统调度以及管理，调度器最多可以创建 10000 个线程，在 Go 语言中使用 runtime.m 结构表示。**M的组织形式是链表**

- P：processor逻辑处理器，但不代表真正的CPU的数量，真正决定并发程度的是P，初始化的时候一般会去读取GOMAXPROCS对应的值，如果没有显示设置，则会读取默认值**，所以当P有任务时需要创建或者唤醒一个系统线程(M)来执行它队列里的任务。所以P/M需要进行绑定，构成一个执行单元。此时的p是在堆上**

- **(2)调度器的设计策略**

  **复用线程**：避免频繁的创建、销毁线程，而是对线程的复用。

  1）work stealing机制

   当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。

  2）hand off机制

   当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执行。

  **利用并行**：`GOMAXPROCS`设置P的数量，最多有`GOMAXPROCS`个线程分布在多个CPU上同时运行。`GOMAXPROCS`也限制了并发的程度，比如`GOMAXPROCS = 核数/2`，则最多利用了一半的CPU核进行并行。

  **抢占**：在coroutine中要等待一个协程主动让出CPU才执行下一个协程，在Go中，一个goroutine最多占用CPU 10ms，防止其他goroutine被饿死，这就是goroutine不同于coroutine的一个地方。

  **全局G队列**：在新的调度器中依然有全局G队列，但功能已经被弱化了，当M执行work stealing从其他P偷不到G时，它可以从全局G队列获取G。

  ![image-20220417213654487](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417213654487.png)

  从上图我们可以分析出几个结论：

   1、我们通过 go func()来创建一个goroutine；

   2、有两个存储G的队列，一个是局部调度器P的本地队列、一个是全局G队列。新创建的G会先保存在P的本地队列中，如果P的本地队列已经满了就会保存在全局的队列中；

   3、G只能运行在M中，一个M必须持有一个P，M与P是1：1的关系。M会从P的本地队列弹出一个可执行状态的G来执行，如果P的本地队列为空，就会想其他的MP组合偷取一个可执行的G来执行；

   4、一个M调度G执行的过程是一个循环机制；

   5、当M执行某一个G时候如果发生了syscall或则其余阻塞操作，M会阻塞，如果当前有一些G在执行，runtime会把这个线程M从P中摘除(detach)，然后再创建一个新的操作系统的线程(如果有空闲的线程可用就复用空闲线程)来服务于这个P；

   6、当M系统调用结束时候，这个G会尝试获取一个空闲的P执行，并放入到这个P的本地队列。如果获取不到P，那么这个线程M变成休眠状态， 加入到空闲线程中，然后这个G会被放入全局队列中。