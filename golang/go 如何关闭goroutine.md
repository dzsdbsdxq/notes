> 题目来源：小米

答案：fly

1. 关闭 channel

第一种方法，就是借助 channel 的 close 机制来完成对 goroutine 的精确控制。
在 Go 语言的 channel 中，channel 接受数据有两种方法：
msg := <-ch
msg, ok := <-ch
这两种方式对应着不同的 runtime 方法，我们可以利用其第二个参数进行判别，当关闭 channel 时，就根据其返回结果跳出。

2. 定期轮询 channel

第二种方法，是更为精细的方法，其结合了第一种方法和类似信号量的处理方式。
而 goroutine 的关闭是不知道什么时候发生的，因此在 Go 语言中会利用 for-loop 结合 select 关键字进行监听，再进行完毕相关的业务处理后，再调用 close 方法正式关闭 channel。
若程序逻辑比较简单结构化，也可以不调用 close 方法，因为 goroutine 会自然结束，也就不需要手动关闭了。

3. 使用 context

可以借助 Go 语言的上下文（context）来做 goroutine 的控制和关闭。
在 context 中，我们可以借助 ctx.Done 获取一个只读的 channel，类型为结构体。可用于识别当前 channel 是否已经被关闭，其原因可能是到期，也可能是被取消了。
因此 context 对于跨 goroutine 控制有自己的灵活之处，可以调用 context.WithTimeout 来根据时间控制，也可以自己主动地调用 cancel 方法来手动关闭。