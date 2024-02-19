> **题目序号：**(120)
>
> **题目来源：**米哈游
> **频次：**1

**答案1：**（阿纪、）+

[参考文章](https://studygolang.com/articles/10631?fr=sidebar)

golang控制并发有三种经典的方式,一种是通过**channel**通知实现并发控制 一种是**WaitGroup**,另外一种就是**Context**。

1. 使用最基本通过channel通知实现并发控制
   无缓冲通道:
    无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送 goroutine 和接收 goroutine 同时准备好，才可以完成发送和接收操作。
    从上面无缓冲的通道定义来看，发送 goroutine 和接收 gouroutine 必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相对应的操作准备好为止。这种无缓冲的通道我们也称之为同步通道。
    例子:

   ```go
    func main() {
        ch := make(chan struct{})
        go func() {
            ch <- struct{}{}
        }()
        fmt.Println(<-ch)
    }
   ```

   **解释**
   当主 goroutine 运行到 <-ch 接受 channel 的值的时候，如果该 channel 中没有数据，就会一直阻塞等待，直到有值。 这样就可以简单实现并发控制。

2. 通过sync包中的WaitGroup实现并发控制
   在 sync 包中，提供了 WaitGroup ，它会等待它收集的所有 goroutine 任务全部完成，在主 goroutine 中 Add(delta int) 索要等待goroutine 的数量。
   在每一个 goroutine 完成后 Done() 表示这一个goroutine 已经完成，当所有的 goroutine 都完成后，在主 goroutine 中 WaitGroup 返回返回。

   例子：

   ```go
   fun main(){
       var wg sync.WaitGroup
       var urls = []string{
           "http://www.golang.org/",
           "http://www.sensetime.com/",
           "http://www.baidu.com/",
       }
       for _, url := range urls {
           wg.Add(1)
           go func(url string) {
               defer wg.Done()
               http.Get(url)
           }(url)
       }
       wg.Wait()
   }
   ```

3. 在Go 1.7 以后引进的强大的Context上下文，实现并发控制
   在一些简单场景下使用 channel 和 WaitGroup 已经足够了，但是当面临一些复杂多变的网络并发场景下 channel 和 WaitGroup 显得有些力不从心了。
   比如一个网络请求 Request，每个 Request 都需要开启一个 goroutine 做一些事情，这些 goroutine 又可能会开启其他的 goroutine，比如数据库和RPC服务。
   所以我们需要一种可以跟踪 goroutine 的方案，才可以达到控制他们的目的，这就是Go语言为我们提供的 Context，称之为上下文非常贴切，它就是goroutine 的上下文。
   它是包括一个程序的运行环境、现场和快照等。每个程序要运行时，都需要知道当前程序的运行状态，通常Go 将这些封装在一个 Context 里，再将它传给要执行的 goroutine 。
   context 包主要是用来处理多个 goroutine 之间共享数据，及多个 goroutine 的管理。
   context包方法:
       Done() 返回一个只能接受数据的channel类型，当该context关闭或者超时时间到了的时候，该channel就会有一个取消信号
       Err() 在Done() 之后，返回context 取消的原因。
       Deadline() 设置该context cancel的时间点
       Value() 方法允许 Context 对象携带request作用域的数据，该数据必须是线程安全的。
   例子:

   ```go
   func childFunc(cont context.Context, num *int) {
       ctx, _ := context.WithCancel(cont)
       for {
           select {
           case <-ctx.Done():
               fmt.Println("child Done : ", ctx.Err())
               return
           }
       }
   }
   
   func main() {
       gen := func(ctx context.Context) <-chan int {
           dst := make(chan int)
           n := 1
           go func() {
               for {
                   select {
                   case <-ctx.Done():
                       fmt.Println("parent Done : ", ctx.Err())
                       return // returning not to leak the goroutine
                   case dst <- n:
                       n++
                       go childFunc(ctx, &n)
                   }
               }
           }()
           return dst
       }
   
       ctx, cancel := context.WithCancel(context.Background())
       for n := range gen(ctx) {
           fmt.Println(n)
           if n >= 5 {
               break
           }
       }
       cancel()
       time.Sleep(5 * time.Second)
   }
   ```

   在上面的例子中，主要描述的是通过一个channel实现一个为循环次数为5的循环，在每一个循环中产生一个goroutine，每一个goroutine中都传入context，在每个goroutine中通过传入ctx创建一个子Context,并且通过select一直监控该Context的运行情况，当在父Context退出的时候，代码中并没有明显调用子Context的Cancel函数，但是分析结果，子Context还是被正确合理的关闭了，这是因为，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅的解决了goroutine启动后不可控的问题。