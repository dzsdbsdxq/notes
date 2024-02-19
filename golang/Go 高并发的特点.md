> 题目序号：6748
> 题目来源：360
> 频次：2

答案：苏木

Go 语言天生支持高并发，得益于 go 关键字开辟了协程的调用。

```go
func main()  {
   go add(1,1)  // 开辟了协程
   go add(2,2)  // 开辟了协程
   time.Sleep(time.Second)
}
func add(a int,b int)  {
   result := a+b
   fmt.Println(result)
}
```

>`协程`：
>英文叫作 `Coroutine`，协程是一种用户态的轻量级线程。协程拥有自己的上下文和栈，协程调度切换的时候，将寄存器上下文和栈保存下来，在切回来的时候，又恢复之前保存的上下文和栈。在 go 语言中的协程实体就是 `goroutine`。

**Go 的并发**

Go 语言参考 Communicating Sequential Process（CSP） 模型设计了 goroutine。同时， Go 在 CSP 的 channel 基础中又加入了缓存机制，可以使得当前的任务暂存起来（CSP 中的 channel 是立刻执行），等待执行进程准备好了，即接收到 channel，再逐个按顺序执行。

1) `goroutine：` go 执行并发的实体，它底层是使用协程（coroutine）实现并发，`coroutine` 是一种运行在用户态的用户线程，go 底层选择使用 coroutine 的出发点是因为，它具有以下特点：

   * 用户空间 避免了内核态和用户态的切换导致的成本
   * 可以由语言和框架层进行调度
   * 更小的栈空间允许创建大量的实例
     2) `channel：` 被单独创建并且可以在进程之间传递，它的通信模式类似于 boss-worker 模式的，一个实体通过将消息发送到 channel 中，然后又监听这个 channel 的实体处理，两个实体之间是匿名的，这个就实现实体中间的解耦，在实现原理上其实是一个阻塞的消息队列。
     3) `调度器：` goroutine 中提供了调度器，在调度器加入了steal working 算法 ，goroutine 是可以被异步抢占，因此没有函数调用的进程不再对调度器造成死锁或造成垃圾回收的大幅变慢。并且 go 对网络IO库进行了封装，屏蔽了复杂的细节，对外提供统一的语法关键字支持，简化了并发程序编写的成本。

**参考资料**

Go语言的并发编程 https://www.csdn.net/tags/NtDaYg0sMDMwNjAtYmxvZwO0O0OO0O0O.html

https://blog.csdn.net/big_white_py/article/details/111465167