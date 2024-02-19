> 题目来源：成都景合

解析：大布丁

channel 的使用方法如下：
1、初始化：使用 make() 函数, channel 的go 语言关键字为 chan

```go
    var c chan int = make(chan int) //初始化无缓冲的全局 chan int
    var c chan int = make(chan int, 3) //初始化有缓冲的全局 chan int，缓冲区大小为 3 个 int 类型
```

2、无缓冲的 channel 用于同步处理消息，可用于同步会话链接等

3、有缓冲的 channel 可用于异步或同步处理消息，可以作用于消息队列中。