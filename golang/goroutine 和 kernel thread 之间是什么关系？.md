> 题目序号：(2960)
>
> 题目来源：pingCAP
>
> 频次：1

答案：咸鱼没有早餐

在进程被划分为更小的线程后，线程成为了**最小的调度单元**，也是在 CPU 上**执行的最小单元**

操作系统将内存空间划分为**内核空间**和**用户空间**，

由于**用户级线程**一般使用线程库来模拟线程且**对操作系统保持透明**，因此对操作系统而言只能调度内核空间中的**内核级线程**

内核级线程 **kernel thread** 简称 **KSE**

由此可以将内核级线程视作==用户级线程上 CPU 运行的**机会**==

![image-20220417134648439](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417134648439.png)

对于传统的内核级线程和用户线程，有**一对一**、**一对多**和**多对多**三种模型

对于同一进程内的用户级线程切换，不需要切换上下文也无需额外开销

对于不同进程内的用户级线程切换，要切换进程的上下文，开销较大

![image-20220417135413988](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417135413988.png)

在 go 中使用 goroutine，而 goroutine 是 go 中实现的**用户级线程**

因此一般来看 go 中的线程调度应该如左图所示

在 go 中使用 GMP 模型，其中 P(processor) 专门管理一个 goroutine 的队列，实际情况如右图所示

![image-20220417141424408](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417141424408-16502031148431.png)

总结：goroutine 依靠 kernel thread 执行