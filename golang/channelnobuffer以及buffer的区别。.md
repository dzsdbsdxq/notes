> 题目序号：6337
>
> 题目来源：畅天游
>
> 频次：1

答案：（yacoding）

（1）无缓冲的通道保证进行发送和接收的 goroutine 会在同一时间进行数据交换；有缓冲的通道没有这种保证。

（2）声明无缓冲 channel 的方式是不指定缓冲大小的：

```go
package main

import (
	"sync"
	"time"
)

func main() {
	c := make(chan string)

	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		c <- `foo`
	}()

	go func() {
		defer wg.Done()

		time.Sleep(time.Second * 1)
		println(`Message: `+ <-c)
	}()

	wg.Wait()
}
```

解释：第一个协程会在发送消息`foo`时阻塞，原因是接收者还没有就绪。

> 如果缓冲大小设置为 0 或者不设置，channel 为无缓冲类型，通信成功的前提是发送者和接收者都处于就绪状态。

（3）为通道增加一个有限大小的存储空间形成带缓冲通道：

```go
package main

import "fmt"

func main() {

    // 创建一个3个元素缓冲大小的整型通道
    ch := make(chan int, 3)

    // 查看当前通道的大小
    fmt.Println(len(ch))

    // 发送3个整型元素到通道
    ch <- 1
    ch <- 2
    ch <- 3

    // 查看当前通道的大小
    fmt.Println(len(ch))
}
```

解释：输出的结果为 0  3