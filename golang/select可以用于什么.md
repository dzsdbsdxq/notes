> 题目序号：（3242）
>
> 题目来源：腾讯
>
> 频次：1

## 答案1：（One）

Go 的通道有两种操作方式，一种是带 range 子句的 for 语句，另一种则是 select 语句，它是专门为了操作通道而存在的。这里主要介绍 select 的用法。

**select的语法**

select 语句的语法如下：

```go
select {
    case <-ch1 :
       statement(s)   
    case ch2 <- 1 :
       statement(s)
    …
    default : /* 可选 */
       statement(s)
}
```

这里要注意：

- 每个 case 都必须是一个通信。
  由于 select 语句是专为通道设计的，所以每个 case 表达式中都只能包含操作通道的表达式，比如接收表达式。
- 如果有多个 case 都可以运行，select 会随机公平地选出一个执行，其他不会执行。
- 如果多个 case 都不能运行，若有 default 子句，则执行该语句，反之，select 将阻塞，直到某个 case 可以运行。
- 所有 channel 表达式都会被求值。

用一个简单示例看一下：

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    // 准备好几个通道。
    intChannels := [5]chan int{
        make(chan int, 1),
        make(chan int, 1),
        make(chan int, 1),
        make(chan int, 1),
        make(chan int, 1)
    }
    // 随机选择一个通道，并向它发送元素值。
    index := rand.Intn(5)
    fmt.Printf("The index: %d
", index)
    intChannels[index] <- index
    // 哪一个通道中有可取的元素值，哪个对应的分支就会被执行。
    select {
        case <-intChannels[0]:
            fmt.Println("The first candidate case is selected.")
        case <-intChannels[1]:
            fmt.Println("The second candidate case is selected.")
        case elem := <-intChannels[2]:
            fmt.Printf("The third candidate case is selected. The element is %d.
", elem)
        default:
            fmt.Println("No candidate case is selected!")
    }
}
```

准备了5个通道，放到一个数组里，并用0-4的随机数作为数组的索引，向通道发送元素。用一个包含了三个候选分支的 select 语句，分别尝试从三个通道中接收元素值，哪一个通道中有值，哪一个对应的候选分支就会被执行。
执行结果如下：

```delphi
The index: 1
The second candidate case is selected.
```

多次执行的话，会随机输出不同的字符串，如果随机值不是0、1、2，则会执行 default 语句。

**select死锁**

select 使用不当会发生死锁。

- 如果通道没有数据发送，但 select 中有存在接收通道数据的语句，将发生死锁。

```go
package main
    func main() {  
        ch := make(chan string)
        select {
            case <-ch:
        }
    }
```

报错如下：

```go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/workspace/src/test.go:5 +0x52
exit status 2
```

可以添加 default 语句来避免产生死锁。

- 空 select{}

```go
package main
    func main() {  
        select {}
    }
```

报错如下：

```lua
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:
main.main()
	/workspace/src/test.go:3 +0x20
exit status 2
```

**select和for结合使用**

select 语句只能对其中的每一个case表达式各求值一次。所以，如果想连续或定时地操作其中的通道的话，就需要通过在for语句中嵌入select语句的方式实现。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(time.Second)
    for {
        select {
            case t := <-tick:
                fmt.Println(t)
                break
            }
        }
        fmt.Println("end")
    }
```

打印结果如下：

```yaml
2021-10-10 23:40:59.664804 +0800 CST m=+1.001254136
2021-10-10 23:41:00.665263 +0800 CST m=+2.001696651
2021-10-10 23:41:01.665595 +0800 CST m=+3.002013571
2021-10-10 23:41:02.665293 +0800 CST m=+4.001699053
2021-10-10 23:41:03.665308 +0800 CST m=+5.001702570
2021-10-10 23:41:04.666859 +0800 CST m=+6.003244115
2021-10-10 23:41:05.665595 +0800 CST m=+7.001972958
……
```

你会发现 break 只跳出了 select，无法跳出for。
解决办法有两种：

- 使用 goto 跳出循环

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(time.Second)
    for {
        select {
            case t := <-tick:
                fmt.Println(t)
                //跳到指定位置
                goto END
            }
        }
END:
        fmt.Println("end")
    }
```

- 使用标签

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(time.Second)
//这是标签
FOREND:
    for {
        select {
            case t := <-tick:
                fmt.Println(t)
                //跳出FOREND标签
                break ForEnd
            }
        }
END:
        fmt.Println("end")
    }
```

**select实现超时机制**

主要使用的 `time.After` 实现超时控制。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)
    quit := make(chan bool)

    go func() {
        for {
            select {
            case num := <-ch:  //如果有数据，下面打印。但是有可能ch一直没数据
                fmt.Println("num = ", num)
            case <-time.After(3 * time.Second): //上面的ch如果一直没数据会阻塞，那么select也会检测其他case条件，检测到后3秒超时
                fmt.Println("超时")
                quit <- true  //写入
            }
        }

    }()

    for i := 0; i < 5; i++ {
        ch <- i
        time.Sleep(time.Second)
    }
    <-quit //这里暂时阻塞，直到可读
    fmt.Println("程序结束")
}
```

执行后，可以观察到：依次打印出0-4，几秒过后打印出“超时”和“程序结束”，打印结果如下：

```go
num =  0
num =  1
num =  2
num =  3
num =  4
超时
程序结束
```

#### 