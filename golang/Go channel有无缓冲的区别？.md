无缓冲：一个送信人去你家送信，你不在家他不走，你一定要接下信，他才会走。 

有缓冲：一个送信人去你家送信，扔到你家的信箱转身就走，除非你的信箱满了，他必须等信箱有多余空间才会走。

|          | 无缓冲             | 有缓冲                |
| -------- | ------------------ | --------------------- |
| 创建方式 | make(chan TYPE)    | make(chan TYPE, SIZE) |
| 发送阻塞 | 数据接收前发送阻塞 | 缓冲满时发送阻塞      |
| 接收阻塞 | 数据发送前接收阻塞 | 缓冲空时接收阻塞      |

**非缓冲** `channel`

```
package main

import (
"fmt"
"time"
)

func loop(ch chan int) {
for {
select {
case i := <-ch:
fmt.Println("this  value of unbuffer channel", i)
}
}
}

func main() {
ch := make(chan int)
ch <- 1
go loop(ch)
time.Sleep(1 * time.Millisecond)
}
```

这里会报错 `fatal error: all goroutines are asleep - deadlock!` 就是因为 `ch<-1` 发送了，但是同时没有接收者，所以就发生了阻塞

但如果我们把 `ch <- 1` 放到 `go loop(ch)` 下面，程序就会正常运行

**缓冲** `channel`

```
package main

import (
"fmt"
"time"
)

func loop(ch chan int) {
for {
select {
case i := <-ch:
fmt.Println("this  value of unbuffer channel", i)
}
}
}

func main() {
ch := make(chan int,3)
ch <- 1
ch <- 2
ch <- 3
ch <- 4
go loop(ch)
time.Sleep(1 * time.Millisecond)
}
```

这里也会报 fatal error: all goroutines are asleep - deadlock! ，这是因为 channel 的大小为 3 ，而我们要往里面塞 4 个数据，所以就会阻塞住，解决的办法有两个:

1. 把 channel 长度调大一点
2. 把 channel 的信息发送者 ch <- 1 这些代码移动到 go loop(ch) 下面 ，让 channel 实时消费就不会导致阻塞了