> 题目序号：542
> 题目来源：腾讯
> 频次：

答案：村雨

`context.Context`类型是在 Go 1.7 版本引入到标准库的，上下文Context主要用来在goroutine之间传递**截止日期**、**停止信号**等上下文信息，并且它是**并发安全**的，可以控制多个goroutine，因此它可以很方便的用于**并发控制**和**超时控制**，标准库中的一些代码包也引入了Context参数，比如os/exec包、net包、database/sql包，等等。

**Context介绍：**
Context类型的应用比较广的，如http后台服务，多个客户端或者请求会导致启动多个goroutine来提供服务，通过Context，我们可以很方便的实现请求数据的共享，比如token值，超时时间等，可以让系统避免额外的资源消耗。
**Context类型：**
Context类型是一个接口类型，定义了4个方法：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

`Deadline()` ：获取设置的截止日期，到截止日期时，Context会自动发起取消请求，ok为false表示没有设置截止日期；
`Done()`：返回一个只读通道chan，在当前工作完成、超时或者context被取消后关闭；
`Err()`：返回Context结束原因，取消时返回Canceled 错误，超时返回 DeadlineExceeded 错误。
`Value`：获取 key 对应的 value值，可以用它来传递额外的信息和信号。

**Context 衍生**
Context值是可以繁衍的，也就是可以通过一个Context值产生任意个子值，这些子值携带了父值的属性和数据，也可以响应通过其父值传达的信号。

Context根节点是一个已经在context包中预定义好的Context值，是全局唯一的，它既不可以被撤销，也不能携带任何数据，可以通过调用`context.Background`函数获取到它。

context包提供了4个用于繁衍Context值的函数：

`WithCancel`：基于parent context 产生一个可撤销（cancel）的子context
`WithDeadline`：产生可以定时撤销的子context，达到截止日期后，context会收到cancel通知。
`WithTimeout`：与 WithDeadline 类似，产生可以定时撤销的子context
`WithValue`：产生携带额外数据的子context
**总结**
Context类型是一个可以实现多 goroutine 并发控制的同步工具。Context类型主要分为三种，即：根Context、可撤销的Context和携带数据的Context。根Context和衍生的Context构成一颗Context树。需要注意的是，携带数据的Context不能被撤销，可撤销的Context无法携带数据。

Context比sync.WaitGroup更加灵活，在使用WaitGroup时，我们需要确定执行子任务的 goroutine 数量，如果不知道这个数量，使用WaitGroup就有风险了，采用Context就很容易解决了。