### 泄露原因

- Goroutine 内进行channel/mutex 等读写操作被一直阻塞。
- Goroutine 内的业务逻辑进入死循环，资源一直无法释放。
- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待

### 泄露场景

如果输出的 goroutines 数量是在不断增加的，就说明存在泄漏

**nil channel**

channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞。

```go
func main() {
fmt.Println("before goroutines: ", runtime.NumGoroutine())
block1()
time.Sleep(time.Second * 1)
fmt.Println("after goroutines: ", runtime.NumGoroutine())
}

func block1() {
var ch chan int
for i := 0; i < 10; i++ {
go func() {
<-ch
}()
}
}
```

输出结果：

```
before goroutines:  1
after goroutines:  11
```

**发送不接收**

channel 发送数量 超过 channel接收数量，就会造成阻塞

```go
func block2() {
ch := make(chan int)
for i := 0; i < 10; i++ {
go func() {
ch <- 1
}()
}
}
```

**接收不发送**

channel 接收数量 超过 channel发送数量，也会造成阻塞

```go
func block3() {
ch := make(chan int)
for i := 0; i < 10; i++ {
go func() {
<-ch
}()
}
}
```

**http request body未关闭**

`resp.Body.Close()` 未被调用时，goroutine不会退出

```go
func requestWithNoClose() {
_, err := http.Get("https://www.baidu.com")
if err != nil {
fmt.Println("error occurred while fetching page, error: %s", err.Error())
}
}

func requestWithClose() {
resp, err := http.Get("https://www.baidu.com")
if err != nil {
fmt.Println("error occurred while fetching page, error: %s", err.Error())
return
}
defer resp.Body.Close()
}

func block4() {
for i := 0; i < 10; i++ {
wg.Add(1)
go func() {
defer wg.Done()
requestWithNoClose()
}()
}
}

var wg = sync.WaitGroup{}

func main() {
block4()
wg.Wait()
}
```

一般发起http请求时，需要确保关闭body

```go
defer resp.Body.Close()
```

**互斥锁忘记解锁**

第一个协程获取 `sync.Mutex` 加锁了，但是他可能在处理业务逻辑，又或是忘记 `Unlock` 了。

因此导致后面的协程想加锁，却因锁未释放被阻塞了

```go
func block5() {
var mutex sync.Mutex
for i := 0; i < 10; i++ {
go func() {
mutex.Lock()
}()
}
}
```

**sync.WaitGroup使用不当**

由于 `wg.Add` 的数量与 `wg.Done` 数量并不匹配，因此在调用 `wg.Wait` 方法后一直阻塞等待

```go
func block6() {
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
go func() {
wg.Add(2)
wg.Done()
wg.Wait()
}()
}
}
```

### 如何排查

单个函数：调用 `runtime.NumGoroutine` 方法来打印 执行代码前后Goroutine 的运行数量，进行前后比较，就能知道有没有泄露了。

生产/测试环境：使用`PProf`实时监测Goroutine的数量