**多个goroutine并发执行时，每一个goroutine抢到处理器的时间点不一致，gorouine的执行本身不能保证顺序。**即代码中先写的gorouine并不能保证先执行

思路：使用channel进行通信通知，用channel去传递信息，从而控制并发执行顺序

```sql
package main

import (
"fmt"
"sync"
"time"
)

var wg sync.WaitGroup

func main() {
ch1 := make(chan struct{}, 1)
ch2 := make(chan struct{}, 1)
ch3 := make(chan struct{}, 1)
ch1 <- struct{}{}
wg.Add(3)
start := time.Now().Unix()
go print("gorouine1", ch1, ch2)
go print("gorouine2", ch2, ch3)
go print("gorouine3", ch3, ch1)
wg.Wait()
end := time.Now().Unix()
fmt.Printf("duration:%d
", end-start)
}

func print(gorouine string, inputchan chan struct{}, outchan chan struct{}) {
// 模拟内部操作耗时
time.Sleep(1 * time.Second)
select {
case <-inputchan:
fmt.Printf("%s
", gorouine)
outchan <- struct{}{}
}
wg.Done()
}
```

- 输出：

```
gorouine1
gorouine2
gorouine3
duration:1
```