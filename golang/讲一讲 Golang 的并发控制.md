> 题目序号：169
>
> 题目来源：哔哩哔哩
>
> 频次：1

**答案1：**（小小）

开发 go 程序的时候，时常需要使用 goroutine 并发处理任务，有时候这些 goroutine 是相互独立的，需要保证并发的数据安全性，也有的时候，goroutine 之间要进行同步与通信，主 goroutine 需要控制它所属的子goroutine。前者并发数据安全依赖锁机制和原子操作，包括互斥锁 `sync.Mutex`，读写锁 `sync.RWMutex`，原子操作`sync/atomic`等，后者涉及到并发行为控制，并发行为控制有三种常见的方式，分别是 `WaitGroup`，`Channel`，`Context`。

1. **数据安全性控制（data race）**

- 互斥锁 sync.Mutex
- 读写锁 sync.RWMutex
- 原子操作 sync/atomic

2. **并发 gorutine 行为控制**

- sync.WaitGroup（等待所有 goroutine 全部结束）
- channel（可以用有缓冲队列控制 goroutine 的数量，可以用 channel+select 实现 goroutine 间的消息通知）
- Context（web 场景中，树状的调用结构下，各 goroutine 间消息同步（退出、超时等），父 context 通知所有子 context）

**WaitGroup**

WaitGroup 位于 Sync 包中，

```go
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```

`Add`方法，用来设置 WaitGroup 的计数值；
`Done`方法，用来将 WaitGroup 的计数值减 1，其实就是调用`Add(-1)`;
`Wait`方法，调用这个方法的 goroutine 会一直阻塞，直到 WaitGroup 的计数值变为 0。

```go
func main() {
  var wg sync.WaitGroup

  wg.Add(2) //添加需要完成的工作量2

  go func() {
    wg.Done() //完成工作量1
    fmt.Println("goroutine 1 完成工作！")
  }()

  go func() {
    wg.Done() //完成工作量1
    fmt.Println("goroutine 2 完成工作！")
  }()

  wg.Wait() //等待工作量2均完成
  fmt.Println("所有的goroutine均已完成工作！")
}

输出:
//goroutine 2 完成工作！
//goroutine 1 完成工作！
//所有的goroutine均已完成工作！

```

WaitGroup这种并发控制方式尤其适用于：某任务需要多 goroutine 协同工作，每个 goroutine 只能做该任务的一部分，只有全部的 goroutine 都完成，任务才算是完成。因此，WaitGroup同名字的含义一样，是一种等待的方式。在使用 WaitGroup 的时候，需要注意以下几点：
不重用 WaitGroup 。虽然 WaitGroup 可以在计数值恢复到零值的时候被重用，但如果在 WaitGroup 的计数器还没有恢复到零值的时候被重用，会导致`panic`。所以不如新建一个 WaitGroup 也不会有很大的资源开销。
保证所有的 Add 方法调用都在 Wait 之前。
不传递负数给 Add 方法，只通过 Done 来给计数值减 1。
不做多余的 Done 方法调用，保证 Add 的技术值和 Done 方法调用的数量是一样的。
不遗漏 Done 方法的调用，否则回导致 Wait hang 住无法返回。

**Channel**

Channel，可以理解为管道，它的主要功能点是：

- 队列存储数据
- 阻塞和唤醒goroutine

Channel 是 Go 中的一个核心类型。channel 是 goroutine 之间主要的通讯方式，一般会和 select 搭配使用。它可以解决实际的业务中，经常需主动的通知某一个 goroutine 结束的问题。比如我们开启一个后台监控 goroutine，当不再需要监控时，就应该通知这个监控 goroutine close，不然它会一直空转，造成泄漏。针对这种情况，就采用 `channel+select`的方法对 goroutine 进行控制。

面试过程中，也可以讲一讲 CSP：
Channel 是 go CSP 模型的关键，Go 语言实现了 CSP 的部分理论，goroutine 对应 CSP 中并发执行的实体，channel 就对应着 CSP 模型 中的 channel。CSP 描述这样一种并发模型：多个 Process 使用一个 Channel 进行通信, 这个 Channel 连结的 Process 通常是匿名的，消息传递通常是同步的（有别于 Actor Model）。

**Context**

Context 通常被译作上下文，它是一个比较抽象的概念。在讨论链式调用技术时也经常会提到上下文。一般理解为程序单元的一个运行状态、现场、快照，而翻译中上下又很好地诠释了其本质，上下则是存在上下层的传递，上会把内容传递给下。在 Go 语言中，程序单元也就指的是 Goroutine。每个 Goroutine 在执行之前，都要先知道程序当前的执行状态，通常将这些执行状态封装在一个 Context 变量中，传递给要执行的 Goroutine 中。上下文则几乎已经成为传递与请求同生存周期变量的标准方法。在网络编程下，当接收到一个网络请求 Request，在处理这个 Request 的 Goroutine 中，可能需要在当前 Goroutine 继续开启多个新的 Goroutine 来获取数据与逻辑处理（例如访问数据库、RPC 服务等），即一个请求 Request，会需要多个 Goroutine 中处理。而这些Goroutine 可能需要共享 Request 的一些信息；同时当 Request 被取消或者超时的时候，所有从这个 Request 创建的所有 Goroutine 也应该被结束。