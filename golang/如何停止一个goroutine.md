> 题目序号：545
> 题目来源：早安科技
> 频次：

__一、使用channel进行控制__

Go语言有一个著名的设计哲学：Do not communicate by sharing memory; instead, share memory by communicating.——通过通信共享内存，而不是通过共享内存来进行通信。Go语言中实现goroutine之间通信的机制就是channel。因此我们可以使用channel来给goroutine发送消息来变更goroutine的行为。下面是使用channel控制的几种方式。

1.1 for-range结构

for-rang从channel上接收值，直到channel关闭，该结构在Go并发编程中很常用，这对于从单一通道上获取数据去执行某些任务是十分方便的。示例如下

```go
package main

import (
  "fmt"
  "sync"
)

var wg sync.WaitGroup

func worker(ch chan int) {
  defer wg.Done()
  for value := range ch {
    fmt.Println(value) // do something
  }
}

func main() {
  ch := make(chan int)

  wg.Add(1)
  go worker(ch)

  for i := 0; i < 3; i++ {
    ch <- i
  }

  close(ch)
  wg.Wait()
```

1.2 for-select结构

当channel比较多时，for-range结构借不是很方便了。Go语言提供了另外一种和channel相关的语法: select。select能够让goroutine在多个通信操作上等待(可以理解为监听多个channel)。由于这个特性，for-select结构在Go并发编程中使用的频率很高。我在使用Go的开发中，这是我用的最多的一种组合形式:

```go
for {
    select {
    }
}
```

for-select的使用十分灵活，这里我举两个例子

1.2.1 指定一个退出通道

对于for-select结构，一般我会定义一个特定的退出通道，用于接收退出的信号，如quit。退出通道的使用也分两情况，下面看两个示例。

向退出通道发送退出信号

```go
package main

import (
  "fmt"
  "sync"
  "time"
)

var wg sync.WaitGroup

func worker(in, quit <-chan int) {
  defer wg.Done()
  for {
    select {
    case <-quit:
      fmt.Println("收到退出信号")
      return // 必须return，否则goroutine是不会结束的
    case v := <-in:
      fmt.Println(v)
    }
  }
}

func main() {
  quit := make(chan int) // 退出通道
  in := make(chan int)

  wg.Add(1)
  go worker(in, quit)

  for i := 0; i < 3; i++ {
    in <- i
    time.Sleep(1 * time.Second)
  }
  quit <- 1  // 向quit通道发送退出信号
  wg.Wait()
}
```

关闭退出通道
上面这个例子中，如果启动了100个groutine，那么我们就需要向quit通道中发送100次数据，这就很麻烦。怎么办呢？很简单，关闭channel，这样所有监听quit channel的goroutine就都会收到关闭信号。上面的代码只要做一个很小的替换就能工作：

```go
// wg.Add(1)
wg.Add(100)  //前提是你真的有100个goroutine

// quit <- 1 替换为下面的代码
close(quit)
```

1.2.2 多个channel都关闭才能退出

上面讲了定义一个特定的退出通道的方法。这里再讲另一个场景，如果select上监听了多个通道，需要所有的通道都关闭后才能结束goroutine， 这种要如何处理呢？

这里就利用select的一个特性，select不会在nil的通道上进行等待，因此将channel赋值为nil即可。此外，还需要利用channel的ok值。

```go
package main

import (
  "fmt"
  "sync"
  "time"
)

var wg sync.WaitGroup

func worker(in1, in2 <-chan int) {
  defer wg.Done()
  for {
    select {
    case v, ok := <-in1:
      if !ok {
        fmt.Println("收到退出信号")
        in1 = nil
      }
      // do something
      fmt.Println(v)
    case v, ok := <-in2:
      if !ok {
        fmt.Println("收到退出信号")
        in2 = nil
      }
      // do something
      fmt.Println(v)
    }

      // select已经结束，我们需要判断两个通道的状态
    // 都为nil则结束当前goroutine
    if in1 == nil && in2 == nil {
      return
    }
  }
}

func main() {
  in1 := make(chan int) // 退出通道，接收
  in2 := make(chan int)

  wg.Add(2)

  go worker(in1, in2)
  go worker(in2, in2)

  for i := 0; i < 3; i++ {
    in1 <- i
    time.Sleep(1 * time.Second)
    in2 <- i
  }

  close(in1)
  close(in2)
  wg.Wait()
}
```

__二、使用context包__

context包是官方提供的一个用于控制多个goroutine写作的包，篇幅受限，这里只举一个例子，这个例子说明了2个问题：

使用context的cancel信号，可以终止goroutine的运行
context是可以向下传递的

```go
package  main

import (
  "context"
  "fmt"
  "sync"
)

var wg sync.WaitGroup

func gen(ctx context.Context) <-chan int {
  // 创建子context
  subCtx, _ := context.WithCancel(ctx)
  go sub(subCtx)  // 这里使用ctx，也能给goroutine通知

  dst := make(chan int)
  n := 1
  go func() {
    defer wg.Done()
    for {
      select {
      case <-ctx.Done():
        fmt.Println("end")
        return // return，防止goroutine泄露
      case dst <- n:
        n++
      }
    }
  }()
  return dst
}

func sub(ctx context.Context) {
  defer wg.Done()

  for {
    select {
    case <-ctx.Done():
      fmt.Println("end too")
      return // returning not to leak the goroutine
    default:
      fmt.Println("test")
    }
  }
}

func main() {
  wg.Add(2)

  ctx, cancel := context.WithCancel(context.Background())
  for n := range gen(ctx) {
    fmt.Println(n)
    if n == 5 {
      break
    }
  }
  
  cancel()
  wg.Wait()
}
```

__三、总结__

在Go语言的并发编程中，goroutine的启动十分方便，但是goroutine的管理是需要自己去编程实现的。尤其是在多个goroutine协作时，更需要小心谨慎处理，否则程序会有意想不到的bug。

本文主要描述了如何实现从外部主动关闭goroutine的2种方式：

- channel
- context

主动关闭goroutine除了实现特定功能外，还能提升程序性能。goroutine由于某种原因阻塞，不能继续运行，此时程序应该干预，将goroutine结束，而不是让他一直阻塞，如果此类goroutine很多，会耗费更多的资源。因此，有效的管理goroutine是十分有必要的。