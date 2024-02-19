一个 Goroutine 是一个函数或方法执行同时旁边其他任何够程采用了特殊的 Goroutine线程。Goroutine 线程比标准线程更轻量级，大多数 Golang 程序同时使用数千个 Goroutine。

要创建 Goroutine，请`go`在函数声明之前添加关键字。 

```go
go f(x, y, z) 
```

您可以通过向 Goroutine 发送一个信号通道来停止它。Goroutines 只能在被告知检查时响应信号，因此您需要在逻辑位置（例如`for`循环顶部）包含检查。 

```go
package main 
func main() { 
quit := make(chan bool)
go func() { 
for { 
select { 
case <-quit:
return
default: 
// … 
}
}
}() 
// … 
quit <- true 
} 
```