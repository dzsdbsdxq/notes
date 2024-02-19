## 概念

`Go`标准库提供了`WaitGroup`原语,  可以用它来等待一批 Goroutine 结束

## 底层数据结构

```go
// A WaitGroup must not be copied after first use.
type WaitGroup struct {
 noCopy noCopy
 state1 [3]uint32
}
```

其中 `noCopy` 是 golang 源码中检测禁止拷贝的技术。如果程序中有 WaitGroup 的赋值行为，使用 `go vet` 检查程序时，就会发现有报错。但需要注意的是，noCopy 不会影响程序正常的编译和运行。

`state1`主要是存储着状态和信号量，状态维护了 2 个计数器，一个是请求计数器counter ，另外一个是等待计数器waiter（已调用 `WaitGroup.Wait` 的 goroutine 的个数）

当数组的首地址是处于一个`8`字节对齐的位置上时，那么就将这个数组的前`8`个字节作为`64`位值使用表示状态，后`4`个字节作为`32`位值表示信号量(`semaphore`)；同理如果首地址没有处于`8`字节对齐的位置上时，那么就将前`4`个字节作为`semaphore`，后`8`个字节作为`64`位数值。

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220522104433409.png)

## 使用方法

在WaitGroup里主要有3个方法：

- `WaitGroup.Add()`：可以添加或减少请求的goroutine数量，*`Add(n)` 将会导致 `counter += n`*
- `WaitGroup.Done()`：相当于Add(-1)，`Done()` 将导致 `counter -=1`，请求计数器counter为0 时通过信号量调用`runtime_Semrelease`唤醒waiter线程
- `WaitGroup.Wait()`：会将 `waiter++`，同时通过信号量调用 `runtime_Semacquire(semap)`阻塞当前 goroutine

```go
func main() {
    var wg sync.WaitGroup
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            println("hello")
        }()
    }

    wg.Wait()
}
```

