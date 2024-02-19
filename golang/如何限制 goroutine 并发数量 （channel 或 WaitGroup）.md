> 题目序号：6744
> 题目来源：360
> 频次：6

答案：苏木

限制 goroutine 并发数量有两种办法：

* 使用channel通道
* WaitGroup

**chanel 实现 goroutine 并发数量限制**  
在每次执行 go 语句之前向通道写入值，直到通道满之后就阻塞了，从而实现对 goroutine 数量限制。

```go
func elegance(){
    v := <-ch
    fmt.Println("the ch value receive",v)
}

func main(){
    ch = make(chan int,5)
    for i:=0;i<10;i++{
        ch <- i
        fmt.Println("the ch value send",ch)
        go elegance()
        fmt.Println("the result i",i)
    }
}
```

但尽管这样还是会有新的问题出现：因为在所有 goroutine 执行完成之前， main 函数可能就退出了，导致一些 goroutine 被强制结束。

这时候我们就需要使用 `sync.WaitGroup`，来实现对 goroutine 的结束过程可控。

**WaitGroup 实现 goroutine 并发数量限制** 

WatiGroup是sync包中的一个struct类型，用来收集需要等待执行完成的goroutine。下面是它的定义：

```go
type WaitGroup struct {
        // Has unexported fields.
}
    A WaitGroup waits for a collection of goroutines to finish. The main
    goroutine calls Add to set the number of goroutines to wait for. Then each
    of the goroutines runs and calls Done when finished. At the same time, Wait
    can be used to block until all goroutines have finished.

    A WaitGroup must not be copied after first use.


func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```

它有3个方法：

* `Add()`：每次激活想要被等待完成的goroutine之前，先调用Add()，用来设置或添加要等待完成的goroutine数量。
  例如Add(2)或者两次调用Add(1)都会设置等待计数器的值为2，表示要等待2个goroutine完成
* `Done()`：每次需要等待的goroutine在真正完成之前，应该调用该方法来人为表示goroutine完成了，该方法会对等待计数器减1
* `Wait()`：在等待计数器减为0之前，Wait()会一直阻塞当前的goroutine

也就是说，Add()用来增加要等待的goroutine的数量，Done()用来表示goroutine已经完成了，减少一次计数器，Wait()用来等待所有需要等待的goroutine完成。

通过实例可以很容易理解。

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended
", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

上面激活了3个 `goroutine`，每次激活 goroutine 之前，都先调用 `Add()` 方法增加一个需要等待的 `goroutine 计数`。每个 goroutine 都运行 `process()` 函数，这个函数在执行完成时需要调用 `Done()` 方法来表示 goroutine 的结束。激活3个 goroutine 后， `main goroutine` 会执行到 `Wait()`，由于每个激活的 goroutine 运行的 process() 都需要睡眠2秒，所以 main goroutine 在 Wait() 这里会阻塞一段时间(大约2秒)，当所有 goroutine 都完成后，等待计数器减为0，Wait() 将不再阻塞，于是 main goroutine 得以执行后面的 `Println()`。

还有一点需要特别注意的是 process() 中使用指针类型的 `*sync.WaitGroup` 作为参数，才能使3个 goroutine 共享一个 `wg`。

**参考资料**

通过 channel 限制 goroutine 并发数量 
https://blog.csdn.net/weixin_42445362/article/details/124607185

WaitGroup 用法说明 
https://www.cnblogs.com/f-ck-need-u/p/10004787.html