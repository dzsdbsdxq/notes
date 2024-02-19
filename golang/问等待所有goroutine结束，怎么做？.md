> **题目序号：**（6609）
>
> **题目来源**：滴滴 
>
> **频次**：1

**答案1：**（peace）

1）用channel进行同步(该方法需要知道goroutine的数量)

```go
func main() {
    ch := make(chan int, 2)

    go func() {
        for i := 0; i < 10; i++ {
            time.Sleep(1 * time.Second)
            fmt.Println("go routine1", i)
        }
        ch <- 1
    }()
    go func() {
        for i := 0; i < 10; i++ {
            time.Sleep(1 * time.Second)
            fmt.Println("go routine2", i)
        }
        ch <- 2
    }()

    // 等待
    for i := 0; i < 2; i++ {
        <-ch
    }

    fmt.Println("main exist")
}
```

2)用sync.WaitGroup

```go
package main

import "fmt"
import "time"
import "sync"

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            time.Sleep(1 * time.Second)
            fmt.Println(i)
        }(i)
    }

    wg.Wait() // 等待

    fmt.Println("main exist")
}
```
