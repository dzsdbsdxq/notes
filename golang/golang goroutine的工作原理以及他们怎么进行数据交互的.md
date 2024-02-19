> 题目序号：（1497）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（jimyag）

**1.goroutine创建流程是什么样子的？**

在调用go func()的时候，会调用runtime.newproc来创建一个goroutine，这个goroutine会新建一个自己的栈空间，同时在G的sched中维护栈地址与程序计数器这些信息（**备注**：这些数据在goroutine被调度的时候会被用到。准确的说该goroutine在放弃cpu之后，下一次在重新获取cpu的时候，这些信息会被重新加载到cpu的寄存器中。）

创建好的这个goroutine会被放到，它所对应的内核线程M所使用的上下文P中的runqueue中。等待调度器来决定何时取出该goroutine并执行，通常调度是按时间顺序被调度的，这个队列是一个先进先出的队列。

**2.新建的这些goroutine是如何被调度的呢？**

goroutine在创建好了之后，调度器会决定何时执行这个goroutine，这个过程就叫做调度。

新建好的goroutine，最开始都会存储在某一个线程M，所关联的上下文P的runqueue中，但是在后续的调度中，有些goroutine因为调用了**runtime.gosched，**会被放到全局队列中。

**线程M的选择过程，按照下面的顺序执行：**

> 1.从M对应的P中的runqueue中取出goroutine，来执行，没有的话，执行2。
> 2.从全局队列里面尝试取出一个goroutine来执行，有的话，执行！没有的话，执行3。
> 3.从其他的线程M的P中，偷出一些goroutine来执行，偷失败了，执行4。（**备注**：这里偷的话，一偷就偷一半，使用的算法叫做work stealing。）
> 4.线程M发现无事可做，就去休息了，也就是线程的sleep，它等待被唤醒。

**4.运行中的goroutine是怎么停止的呢？一旦被停止了的话，那排队在它后面的goutinue该怎么办？**

讲完了goroutine的调度之后，我们便要考虑一个问题，正在被执行的goroutine何时停止，停止了之后会发生什么？而挂在M对应的P后面的runqueue中的goroutine该怎么办？

**情况1: runtime·park**

当调用了runtime·park函数之后，goroutine会被设置成waiting状态，线程M会放弃它自身关联的上下文P，而系统会分配一个新的线程M1来接管这个上下文P，（**备注**：当然这里面的M1也有可能是本来就创建好的，处于闲置状态中的）。

原来的线程M0则会与上下文断开连接，M0因为无事可做，就去sleep了，等待下次被唤醒。如下图所示：

channel的读写操作，定时器中，网络poll等都有可能park goroutine。



![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/v2-0ade14231f6a41c21cab7dd1c644376d_720w.jpg)

**情况2: runtime·gosched**

调用runtime·gosched函数也可以让当前goroutine放弃cpu，这种情况下会将goroutine设置称runnable，放置到全局队列中。备注：这个也就是为什么全局变量的queue里面会有goroutine的原因。

**5.goroutine被唤醒之后，会做什么？**

goroutine处于waiting状态的话，在调用runtime·ready函数之后，会被唤醒，唤醒的goroutine会被重新放到，M对应的上下文所对应的runqueue中，等待被调度。

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/v2-921d13efe59224291d32ef3bf68a92ac_720w.jpg)



goroutine的通信

实现控制并发的方式，大致可分成以下三类：
-  全局共享变量
-  channel通信
-  Context包

全局共享变量

这是最简单的实现控制并发的方式，实现步骤是：
1. 声明一个全局变量；
2. 所有子goroutine共享这个变量，并不断轮询这个变量检查是否有更新；
3. 在主进程中变更该全局变量；
4. 子goroutine检测到全局变量更新，执行相应的逻辑。

示例如下：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    running := true
    f := func() {
        for running {
            fmt.Println("sub proc running...")
            time.Sleep(1 * time.Second)
        }
        fmt.Println("sub proc exit")
    }
    go f()
    go f()
    go f()
    time.Sleep(2 * time.Second)
    running = false
    time.Sleep(3 * time.Second)
    fmt.Println("main proc exit")
}
```

**全局变量的优势是简单方便，不需要过多繁杂的操作，通过一个变量就可以控制所有子goroutine的开始和结束；缺点是功能有限，由于架构所致，该全局变量只能是多读一写，否则会出现数据同步问题，当然也可以通过给全局变量加锁来解决这个问题，但那就增加了复杂度，另外这种方式不适合用于子goroutine间的通信，因为全局变量可以传递的信息很小；还有就是主进程无法等待所有子goroutine退出，因为这种方式只能是单向通知，所以这种方法只适用于非常简单的逻辑且并发量不太大的场景，一旦逻辑稍微复杂一点，这种方法就有点捉襟见肘。**



channel通信

另一种更为通用且灵活的实现控制并发的方式是使用channel进行通信。

首先，我们先来了解下什么是golang中的channel：Channel是Go中的一个核心类型，你可以把它看成一个管道，通过它并发核心单元就可以发送或者接收数据进行通讯(communication)。

要想理解 channel 要先知道 CSP 模型：



> CSP 是 Communicating Sequential Process 的简称，中文可以叫做通信顺序进程，是一种并发编程模型，由 Tony Hoare 于 1977 年提出。简单来说，CSP 模型由并发执行的实体（线程或者进程）所组成，实体之间通过发送消息进行通信，这里发送消息时使用的就是通道，或者叫 channel。CSP 模型的关键是关注 channel，而不关注发送消息的实体。Go 语言实现了 CSP 部分理论，goroutine 对应 CSP 中并发执行的实体，channel 也就对应着 CSP 中的 channel。
> 也就是说，CSP 描述这样一种并发模型：多个Process 使用一个 Channel 进行通信, 这个 Channel 连结的 Process 通常是匿名的，消息传递通常是同步的（有别于 Actor Model）。



先来看示例代码：

```go
package main
import (
    "fmt"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)
func consumer(stop <-chan bool) {
    for {
        select {
        case <-stop:
            fmt.Println("exit sub goroutine")
            return
        default:
            fmt.Println("running...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}
func main() {
        stop := make(chan bool)
        var wg sync.WaitGroup
        // Spawn example consumers
        for i := 0; i < 3; i++ {
            wg.Add(1)
            go func(stop <-chan bool) {
                defer wg.Done()
                consumer(stop)
            }(stop)
        }
        waitForSignal()
        close(stop)
        fmt.Println("stopping all jobs!")
        wg.Wait()
}
func waitForSignal() {
    sigs := make(chan os.Signal)
    signal.Notify(sigs, os.Interrupt)
    signal.Notify(sigs, syscall.SIGTERM)
    <-sigs
}
```

这里可以实现优雅等待所有子goroutine完全结束之后主进程才结束退出，借助了标准库sync里的Waitgroup，这是一种控制并发的方式，可以实现对多goroutine的等待，官方文档是这样描述的：



> A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls Add to set the number of goroutines to wait for.
> Then each of the goroutines runs and calls Done when finished. At the same time, Wait can be used to block until all goroutines have finished.

简单来讲，它的源码里实现了一个类似计数器的结构，记录每一个在它那里注册过的协程，然后每一个协程完成任务之后需要到它那里注销，然后在主进程那里可以等待直至所有协程完成任务退出。
使用步骤：

1. 创建一个Waitgroup的实例wg；
2. 在每个goroutine启动的时候，调用wg.Add(1)注册；
3. 在每个goroutine完成任务后退出之前，调用wg.Done()注销。
4. 在等待所有goroutine的地方调用wg.Wait()阻塞进程，知道所有goroutine都完成任务调用wg.Done()注销之后，Wait()方法会返回。

Context

Context的创建和调用关系是层层递进的，也就是我们通常所说的链式调用，类似数据结构里的树，从根节点开始，每一次调用就衍生一个叶子节点。首先，生成根节点，使用context.Background方法生成，而后可以进行链式调用使用context包里的各类方法，context包里的所有方法：

```go
package main

import (
    "context"
    "crypto/md5"
    "fmt"
    "io/ioutil"
    "net/http"
    "sync"
    "time"
)

type favContextKey string

func main() {
    wg := &sync.WaitGroup{}
    values := []string{"https://www.baidu.com/", "https://www.zhihu.com/"}
    ctx, cancel := context.WithCancel(context.Background())

    for _, url := range values {
        wg.Add(1)
        subCtx := context.WithValue(ctx, favContextKey("url"), url)
        go reqURL(subCtx, wg)
    }

    go func() {
        time.Sleep(time.Second * 3)
        cancel()
    }()

    wg.Wait()
    fmt.Println("exit main goroutine")
}

func reqURL(ctx context.Context, wg *sync.WaitGroup) {
    defer wg.Done()
    url, _ := ctx.Value(favContextKey("url")).(string)
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("stop getting url:%s
", url)
            return
        default:
            r, err := http.Get(url)
            if r.StatusCode == http.StatusOK && err == nil {
                body, _ := ioutil.ReadAll(r.Body)
                subCtx := context.WithValue(ctx, favContextKey("resp"), fmt.Sprintf("%s%x", url, md5.Sum(body)))
                wg.Add(1)
                go showResp(subCtx, wg)
            }
            r.Body.Close()
            //启动子goroutine是为了不阻塞当前goroutine，这里在实际场景中可以去执行其他逻辑，这里为了方便直接sleep一秒
            // doSometing()
            time.Sleep(time.Second * 1)
        }
    }
}

func showResp(ctx context.Context, wg *sync.WaitGroup) {
    defer wg.Done()
    for {
        select {
        case <-ctx.Done():
            fmt.Println("stop showing resp")
            return
        default:
            //子goroutine里一般会处理一些IO任务，如读写数据库或者rpc调用，这里为了方便直接把数据打印
            fmt.Println("printing ", ctx.Value(favContextKey("resp")))
            time.Sleep(time.Second * 1)
        }
    }
}
```

前面我们说过Context就是设计用来解决那种多个goroutine处理一个Request且这多个goroutine需要共享Request的一些信息的场景，以上是一个简单模拟上述过程的demo。

首先调用context.Background()生成根节点，然后调用withCancel方法，传入根节点，得到新的子Context以及根节点的cancel方法（通知所有子节点结束运行），这里要注意：该方法也返回了一个Context，这是一个新的子节点，与初始传入的根节点不是同一个实例了，但是每一个子节点里会保存从最初的根节点到本节点的链路信息 ，才能实现链式。

程序的reqURL方法接收一个url，然后通过http请求该url获得response，然后在当前goroutine里再启动一个子groutine把response打印出来，然后从ReqURL开始Context树往下衍生叶子节点（每一个链式调用新产生的ctx）,中间每个ctx都可以通过WithValue方式传值（实现通信），而每一个子goroutine都能通过Value方法从父goroutine取值，实现协程间的通信，每个子ctx可以调用Done方法检测是否有父节点调用cancel方法通知子节点退出运行，根节点的cancel调用会沿着链路通知到每一个子节点，因此实现了强并发控制。

[深入golang之---goroutine并发控制与通信 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/36907022)

参考[Go并发(二)：goroutine的实现原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/82740001)

